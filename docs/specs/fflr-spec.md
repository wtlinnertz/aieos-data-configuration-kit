# Feature Flag Lifecycle Record — Specification

Version: v1.0

The Feature Flag Lifecycle Record (FFLR) tracks feature flags from creation through retirement. It documents flag purpose, type, current state per environment, rollout history, and retirement criteria. The FFLR is a living document that is updated as flags change state and re-frozen periodically.

The FFLR is created during REK when feature flags are used for progressive delivery. It is maintained through the production lifecycle and re-frozen at each RHR cycle or when a flag is retired.

---

## What This Artifact Is Not

- **Not a feature flag platform configuration.** The FFLR documents the governance state of feature flags — not the platform-specific configuration that implements them. Platform configurations are operational artifacts.
- **Not a release plan.** The FFLR tracks flag state over time. The release plan (RP) defines the rollout strategy. The FFLR references the RP but does not replace it.
- **Not a feature specification.** The FFLR documents the flag that controls a feature's exposure, not the feature itself. Feature specifications belong in the PRD or TDD.

---

## Purpose

The FFLR serves three roles:

1. **Flag inventory** — Provides a single source of truth for all active feature flags, preventing "flag sprawl" where flags accumulate without tracking
2. **Lifecycle governance** — Defines retirement criteria and target dates for every flag, ensuring temporary flags do not become permanent
3. **Audit trail** — Records the complete rollout history of each flag, enabling post-incident analysis of flag-related changes

---

## Upstream Dependencies

- Frozen RR (from REK) — provides feature flag states at release time
- Frozen RP (from REK) — provides flag-based exposure strategy
- Previous FFLR version (if updating an existing record)

---

## Required Sections

1. Document Control
2. Flag Inventory
3. State Tracking
4. Retirement Criteria
5. Rollout History
6. Stale Flag Assessment

---

## Content Rules

### Document Control
**Rules**
- FFLR ID must be present (format: FFLR-{PROJECT}-{NNN})
- System name must be present
- Owner must be present as a team or role, not an individual
- Version must be present (format: v{N}, e.g., v1, v2, v3 — incremented at each re-freeze)
- Status must be one of: Draft | Validated | Frozen
- RR reference must be present (the frozen RR that triggered the initial FFLR or the most recent update)
- Assessment date must be present (the date of the most recent stale flag assessment)

**Failure Examples**
- FFLR ID absent
- Version not incremented from previous frozen version
- RR reference absent
- Assessment date absent

### Flag Inventory
**Rules**
- Every active flag must be listed with:
  - A flag key (the exact identifier as it appears in the flag system)
  - A purpose statement (what this flag controls and why it exists)
  - A type classification: one of release | ops | experiment | permission
  - A creation date
  - A creator (team or role)
- Retired flags must remain in the inventory with status "retired" and retirement date
- Each flag key must be unique within the FFLR
- Flag types must use the enumerated values — custom types are not permitted without a principles file defining them

**Failure Examples**
- Flag listed without type classification
- Flag listed without purpose statement
- Type not from the enumerated list (release | ops | experiment | permission)
- Duplicate flag keys

### State Tracking
**Rules**
- For every active flag in §2, the current state must be documented per environment
- State must be one of: enabled | disabled | percentage (with percentage value) | not deployed
- Every target environment must be covered — no environment may have an unknown flag state
- If a flag has different configurations per environment (e.g., enabled in staging, 50% in production), each environment's state must be listed separately

**Failure Examples**
- Flag with no state for one or more environments
- State described qualitatively ("mostly enabled") instead of using enumerated values
- Environment missing from state tracking

### Retirement Criteria
**Rules**
- Every flag must have a defined retirement condition — the specific event or condition that triggers retirement
- Every flag must have a target retirement date
- Retirement conditions must be specific and evaluable (not "when no longer needed")
- Permanent flags (ops or permission type) must have a review date instead of a retirement date, with justification for permanence

**Failure Examples**
- Flag without retirement condition
- Retirement condition stated as "when ready" or "TBD"
- Target date absent
- Permanent flag without justification or review date

### Rollout History
**Rules**
- Every state change for every flag must be logged with:
  - Timestamp (ISO 8601 format or date at minimum)
  - Who made the change (team or role)
  - What changed (from state → to state, per environment)
  - Rationale (why the change was made)
- The initial creation of the flag must be the first entry in its rollout history
- History entries must be in chronological order

**Failure Examples**
- State change without timestamp
- State change without rationale
- Missing initial creation entry
- History entries out of order

### Stale Flag Assessment
**Rules**
- Every flag past its target retirement date must be flagged as stale
- For each stale flag, the assessment must include:
  - How far past the retirement date (in days or weeks)
  - Current state per environment
  - Reason the flag has not been retired
  - Updated target retirement date or escalation action
- If no flags are stale, this section must explicitly state "No stale flags identified as of {assessment date}"
- The assessment date must match the Document Control assessment date

**Failure Examples**
- Flag past retirement date not identified as stale
- Stale flag without reason for delay
- Stale flag without updated target date or escalation
- Section absent entirely

---

## Format Requirements

- Flag keys must be exact strings matching the flag system
- Dates must be in ISO 8601 format (YYYY-MM-DD) or equivalent unambiguous format
- Percentages must be numeric (e.g., 50%, not "half")
- State values must use the enumerated vocabulary: enabled | disabled | percentage | not deployed | retired

---

## Completeness Rules

- All six sections must be present and non-empty
- Every active flag has key, purpose, type, creation date, creator, per-environment state, retirement criteria, and rollout history
- Every stale flag is assessed
- Rollout history captures all state changes with timestamp, who, what, and rationale

---

## Relationship Rules

- The FFLR is a living document — it is updated and re-frozen as flags change state
- Each re-freeze increments the version number
- The previous frozen version is retained as an immutable record
- Re-freeze is required at each RHR cycle, at flag retirement, and at any flag state change
- The FFLR feeds stale flag alerts to RRK — RRK consumes the stale flag assessment section

---

## Hard Gates

1. **flag_inventory** — All active flags listed with flag key, purpose, type (release | ops | experiment | permission), creation date, and creator; retired flags listed with retirement date; no duplicate keys
2. **state_tracking** — Current state per environment documented for every active flag; state uses enumerated values (enabled | disabled | percentage | not deployed); every target environment covered
3. **retirement_criteria** — Every flag has a defined retirement condition and target date; conditions are specific and evaluable; permanent flags have review date with justification
4. **rollout_history** — Every state change logged with timestamp, who, what changed, and rationale; initial creation is first entry; entries in chronological order
5. **stale_flag_assessment** — Flags past retirement date identified as stale; each stale flag has delay reason and updated target date or escalation; assessment date matches Document Control; section present even if no stale flags
