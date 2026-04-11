# Feature Flag Lifecycle Record — Validator

This validator evaluates a completed Feature Flag Lifecycle Record (FFLR) against `docs/specs/fflr-spec.md`. It is used in a separate AI session from the one that generated the FFLR.

**Validator role:** Judge pass/fail. Do not suggest improvements, redesign content, or offer alternatives. Evaluate only what is explicitly present.

---

## Inputs Required

To run this validator, provide:
1. The completed FFLR (full document)
2. `docs/specs/fflr-spec.md` (the spec — use this as the authoritative rules)

Do not use any other document as the source of truth for pass/fail criteria.

---

## Evaluation Procedure

Evaluate each hard gate in order. For each gate, apply the rules exactly as stated in the spec. Do not infer intent. Do not give partial credit. Ambiguity in the artifact is a failure condition — if you cannot determine whether a gate passes from what is explicitly present, the gate fails.

---

## Hard Gate Checks

### Gate 1: flag_inventory

Check:
- Every active flag has a flag key (exact identifier)
- Every active flag has a purpose statement
- Every active flag has a type classification from the enumerated list: release | ops | experiment | permission
- Every active flag has a creation date
- Every active flag has a creator (team or role)
- Retired flags are present with status "retired" and retirement date
- Flag keys are unique within the FFLR

**Pass:** All active flags have all five fields; retired flags retained with retirement date; no duplicate keys.
**Fail:** Any active flag missing key, purpose, type, creation date, or creator; type not from enumerated list; duplicate keys; retired flags removed from inventory.

---

### Gate 2: state_tracking

Check:
- Every active flag has current state documented per environment
- State uses enumerated values: enabled | disabled | percentage (with numeric value) | not deployed
- Every target environment is covered — no environment has an unknown state for any active flag
- Qualitative descriptions ("mostly enabled," "partially rolled out") fail

**Pass:** Every active flag has state per environment; all states use enumerated values; all environments covered.
**Fail:** Any flag missing state for an environment; state described qualitatively; environment missing from tracking.

---

### Gate 3: retirement_criteria

Check:
- Every flag has a defined retirement condition
- Every flag has a target retirement date
- Retirement conditions are specific and evaluable (not "when ready," "when no longer needed," or "TBD")
- Permanent flags (ops or permission type) have a review date instead of retirement date, with justification for permanence

**Pass:** All flags have conditions and dates; conditions are evaluable; permanent flags justified.
**Fail:** Any flag without retirement condition; condition stated as "when ready" or "TBD"; permanent flag without justification or review date; target date absent.

---

### Gate 4: rollout_history

Check:
- Every flag has at least one rollout history entry (the creation entry)
- The first entry for each flag is the initial creation event
- Every entry has a timestamp (date or ISO 8601)
- Every entry has who made the change (team or role)
- Every entry has what changed (from/to state, per environment)
- Every entry has a rationale
- Entries are in chronological order

**Pass:** Every flag has creation as first entry; all entries have timestamp, who, what, rationale; chronological order.
**Fail:** Any flag missing creation entry; entry without timestamp, who, what, or rationale; entries out of order.

---

### Gate 5: stale_flag_assessment

Check:
- Every flag past its target retirement date is identified as stale
- Each stale flag has: delay reason, current state per environment, updated target date or escalation action
- The assessment date matches the Document Control assessment date
- If no flags are stale, the section explicitly states "No stale flags identified as of {date}"
- Section is present — not absent or empty

**Pass:** Stale flags identified; each has delay reason and updated target; assessment date matches; section present.
**Fail:** Flag past retirement date not identified as stale; stale flag without delay reason; stale flag without updated target; assessment date mismatch; section absent.

---

## Output Format

Produce a JSON result in exactly this format:

```json
{
  "status": "PASS | FAIL",
  "summary": "<one sentence verdict>",
  "hard_gates": {
    "flag_inventory": "PASS | FAIL",
    "state_tracking": "PASS | FAIL",
    "retirement_criteria": "PASS | FAIL",
    "rollout_history": "PASS | FAIL",
    "stale_flag_assessment": "PASS | FAIL"
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
- Do not redesign or improve the FFLR
- Do not evaluate writing quality beyond spec requirements
- Evaluate only what is explicitly present in the document
