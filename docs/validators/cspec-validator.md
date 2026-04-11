# Configuration Specification — Validator

This validator evaluates a completed Configuration Specification (CSPEC) against `docs/specs/cspec-spec.md`. It is used in a separate AI session from the one that generated the CSPEC.

**Validator role:** Judge pass/fail. Do not suggest improvements, redesign content, or offer alternatives. Evaluate only what is explicitly present.

---

## Inputs Required

To run this validator, provide:
1. The completed CSPEC (full document)
2. `docs/specs/cspec-spec.md` (the spec — use this as the authoritative rules)

Do not use any other document as the source of truth for pass/fail criteria.

---

## Evaluation Procedure

Evaluate each hard gate in order. For each gate, apply the rules exactly as stated in the spec. Do not infer intent. Do not give partial credit. Ambiguity in the artifact is a failure condition — if you cannot determine whether a gate passes from what is explicitly present, the gate fails.

---

## Hard Gate Checks

### Gate 1: config_inventory

Check:
- Every configuration item has a name (exact configuration key)
- Every configuration item has a type (from the enumerated list: string, integer, boolean, float, duration, URL, enum, or custom with definition)
- Every configuration item has a default value or explicit "Required — no default" statement
- Every configuration item has a description
- Every configuration item has a validation rule reference to §5
- Configuration item names are unique within the CSPEC
- Configuration items are traceable to the TDD (referenced in Document Control)

**Pass:** All items have all five fields; names unique; items traceable to TDD.
**Fail:** Any item missing name, type, default/required statement, description, or validation reference; duplicate names; items not traceable to TDD.

---

### Gate 2: environment_coverage

Check:
- Every target environment is listed
- For each environment, every configuration item has either an explicit value or an explicit inheritance statement
- No environment-item combination is undefined without a resolution plan and target date
- Undefined gaps have resolution plans with target dates

**Pass:** Every environment has complete coverage for every item; any gaps have resolution plans.
**Fail:** Any environment-item combination undefined without resolution plan; environment missing; partial coverage without explanation.

---

### Gate 3: secret_identification

Check:
- Every configuration item that contains sensitive data is identified as a secret
- For each secret, a storage method is specified
- For each secret, access control is specified
- For each secret, a rotation policy is specified (or "no rotation — static credential" with justification)
- No secret has a plaintext default value
- If no secrets exist, the section explicitly states "No secrets identified"

**Pass:** All secrets identified with storage, access control, and rotation; no plaintext defaults; section present.
**Fail:** Secret without handling method; plaintext default for a secret; section absent; sensitive item not identified as secret (e.g., database password, API key, connection string with credentials).

---

### Gate 4: validation_rules

Check:
- Every configuration item in §2 has a corresponding validation rule in §5
- Each validation rule specifies at least one of: type check, range, enum values, or pattern
- Validation rules are specific enough to be machine-evaluable — "valid value" or "must be correct" fails
- Enum types have their complete set of valid values listed
- Numeric types have range bounds or explicit justification for unbounded

**Pass:** Every item has a validation rule; rules are specific and machine-evaluable.
**Fail:** Any item missing a validation rule; rule stated as "valid" without specifics; enum without values listed; numeric without range and no justification.

---

### Gate 5: change_impact

Check:
- Every configuration item is classified as one of: hot-reload, restart required, or redeployment required
- Classification is per item, not a blanket statement
- If any item's change impact is unknown, it is documented as a gap with a resolution plan and target date

**Pass:** Every item classified individually; gaps have resolution plans.
**Fail:** Any item without classification; blanket statement; "unknown" without resolution plan.

---

## Output Format

Produce a JSON result in exactly this format:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": {
    "config_inventory": "PASS | FAIL",
    "environment_coverage": "PASS | FAIL",
    "secret_identification": "PASS | FAIL",
    "validation_rules": "PASS | FAIL",
    "change_impact": "PASS | FAIL"
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
- Do not redesign or improve the CSPEC
- Do not evaluate writing quality beyond spec requirements
- Evaluate only what is explicitly present in the document
