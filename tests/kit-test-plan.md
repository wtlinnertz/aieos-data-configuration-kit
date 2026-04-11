# Data & Configuration Kit — Test Plan

This document contains the structural integrity checks and flow scenario tests for the Data & Configuration Kit. These tests verify that the kit is complete, internally consistent, and capable of producing valid artifacts.

---

## Structural Integrity Checks

Structural checks verify that the kit's files are present, properly named, and internally consistent. These checks do not require AI — they are verifiable by inspection.

### S-01: Four-File Completeness

**Check:** Every governed artifact type has exactly four files: spec, template, prompt, validator.

| Artifact Type | Spec | Template | Prompt | Validator |
|---------------|------|----------|--------|-----------|
| CSPEC | docs/specs/cspec-spec.md | docs/artifacts/cspec-template.md | docs/prompts/cspec-prompt.md | docs/validators/cspec-validator.md |
| FFLR | docs/specs/fflr-spec.md | docs/artifacts/fflr-template.md | docs/prompts/fflr-prompt.md | docs/validators/fflr-validator.md |
| DSR | docs/specs/dsr-spec.md | docs/artifacts/dsr-template.md | docs/prompts/dsr-prompt.md | docs/validators/dsr-validator.md |

**Expected result:** All files present.

---

### S-02: Hard Gate Count Alignment

**Check:** Each spec's declared hard gate count matches the validator's gate checks.

| Artifact Type | Spec Hard Gates | Validator Gates |
|---------------|----------------|----------------|
| CSPEC | 5 | 5 |
| FFLR | 5 | 5 |
| DSR | 5 | 5 |

**Expected result:** Counts match for all three artifact types.

---

### S-03: Hard Gate Name Alignment

**Check:** Gate names in specs match gate names in validators (exact string match for JSON output field names).

| Artifact | Spec Gate Names | Validator Gate Names |
|----------|----------------|---------------------|
| CSPEC | config_inventory, environment_coverage, secret_identification, validation_rules, change_impact | config_inventory, environment_coverage, secret_identification, validation_rules, change_impact |
| FFLR | flag_inventory, state_tracking, retirement_criteria, rollout_history, stale_flag_assessment | flag_inventory, state_tracking, retirement_criteria, rollout_history, stale_flag_assessment |
| DSR | schema_inventory, compatibility_rules, migration_plan, data_validation, version_history | schema_inventory, compatibility_rules, migration_plan, data_validation, version_history |

**Expected result:** All gate names match exactly.

---

### S-04: Prompt-to-Spec Reference Integrity

**Check:** Each generation prompt references the correct spec and template. No prompt inlines content rules.

| Prompt | References Spec | References Template | Inlines Rules? |
|--------|----------------|--------------------|----|
| cspec-prompt.md | docs/specs/cspec-spec.md | docs/artifacts/cspec-template.md | No |
| fflr-prompt.md | docs/specs/fflr-spec.md | docs/artifacts/fflr-template.md | No |
| dsr-prompt.md | docs/specs/dsr-spec.md | docs/artifacts/dsr-template.md | No |

**Expected result:** All prompts reference correct spec and template; no inlined rules.

---

### S-05: Validator-to-Spec Reference Integrity

**Check:** Each validator references its spec as the source of truth. Validators do not reference prompts.

| Validator | References Spec | References Prompt? |
|-----------|-----------------|-------------------|
| cspec-validator.md | docs/specs/cspec-spec.md | No |
| fflr-validator.md | docs/specs/fflr-spec.md | No |
| dsr-validator.md | docs/specs/dsr-spec.md | No |

**Expected result:** All validators reference the correct spec; none reference prompts.

---

### S-06: Template Section Alignment

**Check:** Each template's section headings match the required sections listed in the corresponding spec.

| Artifact | Spec Required Sections | Template Sections |
|----------|----------------------|-------------------|
| CSPEC | Document Control, Configuration Inventory, Environment Coverage, Secret Identification, Validation Rules, Change Impact | §1–§6 (all present) |
| FFLR | Document Control, Flag Inventory, State Tracking, Retirement Criteria, Rollout History, Stale Flag Assessment | §1–§6 (all present) |
| DSR | Document Control, Schema Inventory, Compatibility Rules, Migration Plan, Data Validation, Version History | §1–§6 (all present) |

**Expected result:** All template sections match spec required sections.

---

## Flow Scenario Tests

Flow scenarios verify that the kit's artifacts, when produced in order with appropriate inputs, pass validation. These tests require AI execution.

---

### F-00: CSPEC Generation from TDD

**Scenario:** Receive a frozen TDD with configuration requirements. Generate and validate a CSPEC.

**Setup:**
- Provide: a frozen TDD with configuration items (database URL, API keys, feature toggles, log level, port number)
- Provide: a frozen ORD with config readiness checks
- Generate CSPEC using the CSPEC prompt
- Validate using the CSPEC validator

**Expected outcomes:**
- CSPEC: all 5 gates PASS
- All configuration items from TDD present in inventory
- Secrets (database URL, API keys) identified with handling method
- All environments covered
- Every item has a validation rule and change impact classification

**Key gate to verify:** Gate 3 (secret_identification) — confirm database connection strings and API keys are identified as secrets with no plaintext defaults.

---

### F-01: FFLR Generation from Release

**Scenario:** Receive a frozen RR with feature flags used for progressive delivery. Generate, validate, and freeze the FFLR.

**Setup:**
- Provide: a frozen RR with 3 feature flags (1 release, 1 experiment, 1 ops)
- Provide: a frozen RP with flag-based rollout strategy
- Generate FFLR using the FFLR prompt
- Validate using the FFLR validator

**Expected outcomes:**
- FFLR: all 5 gates PASS
- All 3 flags present with correct type classifications
- Per-environment state documented for each flag
- Retirement criteria defined (release and experiment with dates; ops with review date)
- Rollout history includes creation event for each flag

**Key gate to verify:** Gate 3 (retirement_criteria) — confirm the ops flag has a permanence justification and review date rather than a retirement date.

---

### F-02: DSR Generation and Evolution

**Scenario:** Generate an initial DSR from a TDD. Then simulate a schema change and generate a new DSR version with migration plan.

**Setup:**
- Provide: a frozen TDD with 2 database schemas and 1 API contract
- Generate DSR v1 using the DSR prompt
- Validate DSR v1 → freeze
- Simulate: a new field is added to one schema (backward-compatible change)
- Generate DSR v2 with updated TDD
- Validate DSR v2

**Expected outcomes:**
- DSR v1: all 5 gates PASS; migration plan states "Initial schema version — no migration required"
- DSR v2: all 5 gates PASS; migration plan includes steps, rollback, data preservation for the added field; version history shows both v1 and v2 entries
- Compatibility rules maintained across versions

**Key gate to verify:** Gate 3 (migration_plan) — confirm DSR v2 has a complete migration plan with rollback procedure for the schema change.

---

## Notes

- All structural checks (S-01 through S-06) should be verified before running flow scenarios.
- Flow scenarios F-00 through F-02 cover the three primary artifact generation patterns.
- Additional scenarios may be added as new patterns are identified in production use.
