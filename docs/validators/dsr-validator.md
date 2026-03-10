# Data Schema Record — Validator

This validator evaluates a completed Data Schema Record (DSR) against `docs/specs/dsr-spec.md`. It is used in a separate AI session from the one that generated the DSR.

**Validator role:** Judge pass/fail. Do not suggest improvements, redesign content, or offer alternatives. Evaluate only what is explicitly present.

---

## Inputs Required

To run this validator, provide:
1. The completed DSR (full document)
2. `docs/specs/dsr-spec.md` (the spec — use this as the authoritative rules)

Do not use any other document as the source of truth for pass/fail criteria.

---

## Evaluation Procedure

Evaluate each hard gate in order. For each gate, apply the rules exactly as stated in the spec. Do not infer intent. Do not give partial credit. Ambiguity in the artifact is a failure condition — if you cannot determine whether a gate passes from what is explicitly present, the gate fails.

---

## Hard Gate Checks

### Gate 1: schema_inventory

Check:
- Every schema has a schema name (exact identifier used in the system)
- Every schema has a type classification from the enumerated list: database | api_contract | event_schema | config_format (or custom with definition)
- Every schema has a current version number
- Every schema has a description
- Every schema has a field list with field names and types
- Schema names are unique within the DSR
- Schemas are traceable to the TDD (referenced in Document Control)

**Pass:** All schemas have all five attributes; field list present for each; names unique; schemas traceable to TDD.
**Fail:** Any schema missing name, type, version, description, or field list; type not from enumerated list; duplicate names; schemas not traceable to TDD.

---

### Gate 2: compatibility_rules

Check:
- Every schema has backward compatibility requirement stated (Yes or No)
- Every schema has forward compatibility requirement stated (Yes or No)
- Compatibility direction is specified — "compatible" without direction fails
- Version range for compatibility is specified
- If breaking changes are allowed, the approval process is documented

**Pass:** All schemas have both backward and forward compatibility stated; version range specified; breaking change process documented if applicable.
**Fail:** Any schema missing compatibility rules; direction not specified; version range absent; breaking changes allowed without approval process.

---

### Gate 3: migration_plan

Check:
- For each schema change, a migration plan is present with:
  - Source version and target version
  - Ordered migration steps
  - Rollback procedure
  - Data preservation statement
  - Estimated duration or impact window
- For initial schema versions with no changes, the section states "Initial schema version — no migration required"
- No migration plan is "TBD" or empty

**Pass:** Every schema change has complete migration plan; initial versions have explicit statement; rollback present for every migration.
**Fail:** Schema change without migration steps; migration without rollback; no data preservation statement; "TBD" migration plan; initial version without explicit statement.

---

### Gate 4: data_validation

Check:
- Every field listed in §2 (Schema Inventory) has a corresponding validation rule in §5
- Each validation rule specifies at least:
  - Type constraint (matches declared type)
  - Required/optional status
- Additional constraints (min/max, regex, foreign key) are present where applicable
- Rules are specific enough to be machine-evaluable — "must be valid" fails

**Pass:** Every field has validation; rules include type and required/optional at minimum; rules are machine-evaluable.
**Fail:** Any field without validation; validation stated as "valid" without specifics; required/optional status absent.

---

### Gate 5: version_history

Check:
- Every schema version is tracked with version number, date, change description, and author
- The initial version entry is present for each schema
- Entries are in chronological order
- Dates are in ISO 8601 format or equivalent unambiguous format
- Change descriptions are present (not blank or "N/A" for non-initial entries)

**Pass:** Every version tracked; initial entry present; chronological order; dates present; descriptions present.
**Fail:** Version without date; version without change description; initial entry missing; entries out of order.

---

## Output Format

Produce a JSON result in exactly this format:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": {
    "schema_inventory": "PASS | FAIL",
    "compatibility_rules": "PASS | FAIL",
    "migration_plan": "PASS | FAIL",
    "data_validation": "PASS | FAIL",
    "version_history": "PASS | FAIL"
  },
  "blocking_issues": [
    {
      "gate": "<gate_name>",
      "description": "<what specifically failed>",
      "location": "<section or field where the failure is>"
    }
  ],
  "warnings": [
    {
      "description": "<non-blocking observation>",
      "location": "<section>"
    }
  ],
  "completeness_score": "<0-100>"
}
```

**Interpretation rules:**
- Any gate failure → `"status": "FAIL"`
- `blocking_issues` lists exactly the failures — no additional content
- `warnings` are non-blocking; they do not affect status
- `completeness_score` is advisory; it does not override gate results
- If all gates pass, `blocking_issues` is an empty array

---

## Validator Constraints

- Do not suggest how to fix failures
- Do not redesign or improve the DSR
- Do not evaluate writing quality beyond spec requirements
- Evaluate only what is explicitly present in the document
