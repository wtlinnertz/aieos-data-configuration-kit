# Data Migration Record — Validator

## Purpose

Evaluate a Data Migration Record against the hard gates defined in `dmr-spec.md`. Produce a PASS/FAIL judgment. Do not help, suggest improvements, or expand scope.

## Inputs

1. The DMR to evaluate
2. `docs/specs/dmr-spec.md` — the authoritative rules
3. The frozen DSR referenced in the DMR (for completeness verification)

## Evaluation Process

Evaluate each hard gate independently. A gate passes or fails — there is no partial credit.

### Gate 1: `migration_inventory`

Check §2 Migration Inventory:
- Cross-reference the frozen DSR: every schema change that requires data migration has an entry
- Each entry has: migration ID, source schema, target schema, data store, estimated volume, migration type
- Migration type is one of: Online, Offline, Hybrid
- No migration referenced in §3–§6 is absent from §2

### Gate 2: `strategy_complete`

Check §3 Migration Strategy:
- Every migration in §2 has a strategy entry in §3
- Each strategy has: approach (numbered steps), ordering constraints, duration estimate, and downtime requirement
- No migration lacks a strategy

### Gate 3: `validation_criteria`

Check §4 Data Validation Criteria:
- Every migration has at least one blocking validation check
- Each check has: method, expected result, and blocking flag
- No migration lacks validation criteria

### Gate 4: `rollback_defined`

Check §5 Rollback Procedure:
- Every migration has a rollback entry
- Each entry has: rollback strategy, rollback steps, data loss risk, and rollback duration
- Migrations using "Forward-fix" strategy have explicit documentation of why reversal is impossible
- No migration lacks a rollback procedure

### Gate 5: `dry_run_required`

Check §6 Execution Plan:
- Every migration with type Offline or Hybrid has a dry run step on a non-production environment
- Online migrations: dry run is recommended but not blocking

## Output Format

```json
{
  "status": "PASS | FAIL",
  "summary": "<one-sentence verdict>",
  "hard_gates": {
    "migration_inventory": "PASS | FAIL",
    "strategy_complete": "PASS | FAIL",
    "validation_criteria": "PASS | FAIL",
    "rollback_defined": "PASS | FAIL",
    "dry_run_required": "PASS | FAIL"
  },
  "blocking_issues": [
    {
      "gate": "<gate_name>",
      "description": "<what is wrong>",
      "location": "<section reference>"
    }
  ],
  "warnings": [
    {
      "description": "<non-blocking observation>",
      "location": "<section reference>"
    }
  ],
  "completeness_score": "<0-100>"
}
```

## Rules

- Do not suggest fixes or improvements
- Do not expand scope beyond what the spec requires
- Do not evaluate quality of writing or presentation
- Evaluate only against the hard gates in the spec
- Verify DMR scope against the frozen DSR — flag migrations that don't correspond to DSR schema changes
