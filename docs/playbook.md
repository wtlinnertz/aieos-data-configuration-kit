# Data & Configuration Kit — Playbook

This playbook defines the end-to-end process for the Data & Configuration Kit (Layer 11). Unlike linear kits, DCK is a cross-cutting kit whose artifacts are triggered at different pipeline points. This playbook documents the trigger conditions, generation process, and freeze protocol for each artifact type.

---

## Artifact Flow

```
Trigger A: TDD frozen (EEK)
         │
         ▼
  Configuration Specification (CSPEC) — generated
         │ Validate → Freeze
         ▼
  Outputs to REK (config validation criteria) and RRK (drift detection)

Trigger B: Feature flags created (REK)
         │
         ▼
  Feature Flag Lifecycle Record (FFLR) — generated
         │ Validate → Freeze
         │ (living document — re-frozen at RHR cycle or flag retirement)
         ▼
  Outputs to RRK (stale flag alerts)

Trigger C: Schema design (EEK/TDD)
         │
         ▼
  Data Schema Record (DSR) — generated
         │ Validate → Freeze
         │ (versioned — new version when schemas evolve)
         ▼
  Outputs to EEK (migration requirements) and REK (schema readiness)
```

The three artifact types are independent of each other. They do not have a sequential dependency — each is triggered by its own upstream event. All three may be active simultaneously for the same initiative.

---

## Upstream Interfaces

### From EEK (Engineering Execution Kit — Layer 4)

**Input:** Frozen TDD (Technical Design Document) — config requirements, data models
**Input:** Frozen ORD (Operational Readiness Document) — config readiness checks

The TDD provides:
- Configuration items the system requires (environment variables, feature toggles, connection strings)
- Data models and schema definitions
- Environment-specific requirements

The ORD provides:
- Config readiness verification requirements
- Deployment configuration checklist

If the TDD does not document configuration requirements or data models, the corresponding DCK artifact (CSPEC or DSR) is not required for that initiative.

See `docs/entry-from-eek.md` for the boundary briefing.

### From REK (Release & Exposure Kit — Layer 5)

**Input:** Frozen RR (Release Record) — feature flag states at release time
**Input:** Frozen RP (Release Plan) — flag-based exposure strategy

The RR provides:
- Feature flags created or modified during the release
- Current flag state per environment at release completion
- Flag-based rollout strategy used

If no feature flags were used in the release, no FFLR is required.

### From RRK (Reliability & Resilience Kit — Layer 6)

**Input:** Config-related Incident Records (IR) — incidents caused by configuration errors
**Input:** Config drift observations from Reliability Health Reports (RHR)

Config-related incidents or drift observations may trigger CSPEC revision via the re-entry protocol.

### From ODK (Operational Diagnostics Kit — Layer 8)

**Input:** Config-related PMR (Post-Mortem Report) findings

PMR findings that identify configuration management gaps may trigger CSPEC revision.

---

## Downstream Interfaces

### To REK (Release & Exposure Kit — Layer 5)

**Output:** Frozen CSPEC — provides configuration validation criteria for release verification

The CSPEC §2 (Configuration Inventory) and §4 (Validation Rules) provide the criteria REK uses to verify configuration is correct before promoting a release.

### To RRK (Reliability & Resilience Kit — Layer 6)

**Output:** CSPEC — config drift detection requirements (what to monitor for configuration consistency)
**Output:** FFLR — stale flag alerts (flags past retirement date that need cleanup)

The CSPEC §5 (Change Impact) informs RRK what configuration changes require restart vs. hot-reload. The FFLR §4 (Retirement Criteria) feeds RRK alerting for stale flags.

---

## CSPEC — Configuration Specification

**Artifact:** Configuration Specification (CSPEC)
**Type:** Generated
**Trigger:** TDD frozen in EEK
**Spec:** `docs/specs/cspec-spec.md` (5 hard gates)
**Template:** `docs/artifacts/cspec-template.md`
**Prompt:** `docs/prompts/cspec-prompt.md`
**Validator:** `docs/validators/cspec-validator.md`

### Purpose

The CSPEC defines what configuration a system requires and how it should be managed. It inventories all configuration items, specifies validation rules and defaults, documents per-environment overrides, identifies secrets, and classifies change impact (restart vs. hot-reload).

### Inputs

- Frozen TDD (from EEK) — config requirements and data models
- Frozen ORD (from EEK) — config readiness checks
- Organizational principles (if applicable)

### Process

1. After the TDD is frozen, review its configuration requirements section to identify all configuration items.
2. In a new AI session, provide: frozen TDD + frozen ORD + `docs/specs/cspec-spec.md` + `docs/artifacts/cspec-template.md`. Use the CSPEC prompt (`docs/prompts/cspec-prompt.md`).
3. Review the generated CSPEC. Confirm all configuration items from the TDD are present. Verify secrets are correctly identified. Check that environment overrides cover all target environments.
4. Validate in a separate session using `docs/validators/cspec-validator.md` and the spec.
5. On PASS: human review and freeze.
6. On FAIL: address blocking issues; re-generate or correct; re-validate.

### Freeze Points

- CSPEC must be Frozen before it can be used as config validation criteria by REK
- A frozen CSPEC is immutable; changes require the re-entry protocol

---

## FFLR — Feature Flag Lifecycle Record

**Artifact:** Feature Flag Lifecycle Record (FFLR)
**Type:** Generated (then maintained as living document)
**Trigger:** Feature flags created during REK
**Spec:** `docs/specs/fflr-spec.md` (5 hard gates)
**Template:** `docs/artifacts/fflr-template.md`
**Prompt:** `docs/prompts/fflr-prompt.md`
**Validator:** `docs/validators/fflr-validator.md`

### Purpose

The FFLR tracks feature flags from creation through retirement. It documents flag purpose, type, current state per environment, rollout history, and retirement criteria. Unlike most AIEOS artifacts, the FFLR is a living document that is updated as flags change state and re-frozen periodically.

### Inputs

- Frozen RR (from REK) — feature flag states at release time
- Frozen RP (from REK) — flag-based exposure strategy
- Previous FFLR version (if updating an existing FFLR)

### Process

1. After feature flags are created during a release, collect the flag inventory from the RR and RP.
2. In a new AI session, provide: frozen RR + frozen RP + `docs/specs/fflr-spec.md` + `docs/artifacts/fflr-template.md`. Use the FFLR prompt (`docs/prompts/fflr-prompt.md`).
3. Review the generated FFLR. Confirm all flags from the release are listed. Verify retirement criteria are defined for each flag.
4. Validate in a separate session using `docs/validators/fflr-validator.md` and the spec.
5. On PASS: human review and freeze.
6. On FAIL: address blocking issues; re-generate or correct; re-validate.

### Living Document Protocol

The FFLR is re-frozen when:
- A flag state changes (enabled, disabled, percentage change) — update rollout history, re-validate, re-freeze
- A flag is retired — update state to "retired," record retirement date, re-validate, re-freeze
- At each RHR cycle — review all flags for staleness, update stale flag assessment, re-validate, re-freeze
- A new flag is added to the system — add to inventory, re-validate, re-freeze

Each re-freeze increments the FFLR version (v1, v2, v3, etc.). The previous version remains as an immutable record.

### Freeze Points

- Initial FFLR must be Frozen before it can feed stale flag alerts to RRK
- Each re-freeze produces a new version; the previous version is retained

---

## DSR — Data Schema Record

**Artifact:** Data Schema Record (DSR)
**Type:** Generated
**Trigger:** Schema design during EEK/TDD
**Spec:** `docs/specs/dsr-spec.md` (5 hard gates)
**Template:** `docs/artifacts/dsr-template.md`
**Prompt:** `docs/prompts/dsr-prompt.md`
**Validator:** `docs/validators/dsr-validator.md`

### Purpose

The DSR documents data schema definitions, evolution rules, and migration requirements. It covers database schemas, API contracts, event schemas, and configuration file formats. It tracks schema versions and provides migration procedures with rollback plans.

### Inputs

- Frozen TDD (from EEK) — data models and schema definitions
- Previous DSR version (if updating an existing DSR for schema evolution)

### Process

1. After the TDD is frozen, review its data model section to identify all schemas.
2. In a new AI session, provide: frozen TDD + `docs/specs/dsr-spec.md` + `docs/artifacts/dsr-template.md`. Use the DSR prompt (`docs/prompts/dsr-prompt.md`).
3. Review the generated DSR. Confirm all schemas from the TDD are present. Verify compatibility rules are defined. Check migration plans for schema changes.
4. Validate in a separate session using `docs/validators/dsr-validator.md` and the spec.
5. On PASS: human review and freeze.
6. On FAIL: address blocking issues; re-generate or correct; re-validate.

### Schema Evolution Protocol

When schemas change:
1. Do not edit the frozen DSR. The frozen DSR remains immutable.
2. Identify the trigger: new feature requirement, data model refactoring, API version bump, or incident-driven change.
3. Generate a new DSR version (v2, v3, etc.) using the DSR prompt with the updated TDD or schema specification.
4. The new version must include migration steps from the previous version with rollback procedures.
5. Validate and freeze the new DSR version.
6. All downstream consumers reference the new version.

### Freeze Points

- DSR must be Frozen before schema migrations are executed
- A frozen DSR is immutable; changes require a new version via the Schema Evolution Protocol

---

## Freeze Points Summary

| Artifact | Trigger | When Frozen | What It Gates |
|----------|---------|------------|---------------|
| CSPEC | TDD frozen (EEK) | After validation | Config validation criteria for REK; drift detection for RRK |
| FFLR | Flags created (REK) | After validation (re-frozen periodically) | Stale flag alerts for RRK |
| DSR | Schema design (EEK/TDD) | After validation | Schema migration execution; downstream schema consumers |

---

## Session Discipline

Follow these rules in every AI session:

| Rule | Why |
|------|-----|
| One artifact per session | Prevents context contamination across artifacts |
| Separate generation and validation sessions | Prevents self-validation bias |
| Include full frozen upstream artifacts | Do not summarize; the AI needs complete context |
| Include spec and template for generation sessions | The AI generates against the spec, not from memory |
| Include spec and validator for validation sessions | The validator judges against the spec, not against the generated content |

Do not ask the AI that generated an artifact to also validate it in the same session.

---

## Re-Entry Protocol

When a frozen artifact must be corrected:

1. Identify the artifact to be changed (CSPEC, FFLR, or DSR).
2. Identify all downstream artifacts or consumers that reference it.
3. Assess the impact: does the change affect release verification criteria? Does it change drift detection rules? Does it affect schema migration procedures?
4. Issue a new version of the artifact (e.g., CSPEC v2, DSR v2).
5. Re-validate the new version.
6. Re-validate all affected downstream artifacts that reference the changed artifact.
7. Document the change in the new version's document control section with a reference to the original and the reason for the correction.

For the FFLR, which is a living document, routine state updates follow the Living Document Protocol above rather than the full re-entry protocol. The re-entry protocol applies only when the FFLR structure or hard gate fields must be corrected after a freeze.

---

## Amendment Procedure

A frozen artifact may be corrected in place without re-validation when **all** of the following criteria are met:

1. The correction does not affect any field evaluated by a hard gate.
2. The correction does not change scope, decisions, owners, or technical specifications.
3. The correction does not affect any field referenced by a downstream artifact.

**Procedure:** Make the correction and add an Amendment Log entry to the artifact's Document Control section: date, what changed, materiality criterion cited, and who authorized the change. No re-validation is required.

**If there is any ambiguity** about whether a change is non-material, treat it as material and issue a new artifact version. The amendment path must not become a workaround for the version protocol.

---

## Principle File Revision

When a principle file in `docs/principles/` changes, use the change categories defined in `aieos-governance-foundation/docs/principle-file-standard.md`:

| Change Category | Version Bump | Re-Entry Impact |
|----------------|-------------|-----------------|
| **Minor** (clarification only) | `v_.x → v_.x+1` | No re-entry required; already-frozen artifacts remain valid |
| **Significant** (new requirement or tightened constraint) | `v1.x → v1.x+1` | Review artifacts generated after the change against updated principles; already-frozen artifacts are grandfathered |
| **Breaking** (removal or loosening) | `vN.x → vN+1.0` | Requires service owner authorization and documented business justification; re-entry may be warranted |

Every change to a principle file must bump the version field, even minor clarifications.

---

## Maintaining the Engagement Record

The Engagement Record (ER) is a project-level artifact that lives in the consuming project at `docs/engagement/er-{initiative}.md`. It spans all AIEOS layers and is maintained by each kit's operators as work passes through. The ER spec and format are defined in `aieos-governance-foundation/docs/engagement-record-spec.md`.

**DCK maintains the Layer 11 section of the ER.**

### What to Update

| Trigger | ER update |
|---------|-----------|
| CSPEC frozen | Add CSPEC ID to artifact table |
| FFLR frozen (initial or re-freeze) | Add FFLR ID and version to artifact table |
| DSR frozen | Add DSR ID and version to artifact table |
| DSR revised (new version) | Add new DSR version; note revision date |
| FFLR flag retired | Note flag retirement in Layer 11 section |
