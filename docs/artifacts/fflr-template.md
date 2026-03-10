# Feature Flag Lifecycle Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| FFLR ID | FFLR-{PROJECT}-{NNN} |
| System Name | {system name} |
| Owner | {team or role — not an individual person} |
| Version | v{N} |
| Status | Draft / Validated / Freeze Pending / Frozen |
| RR Reference | {RR-{PROJECT}-{NNN}} |
| Assessment Date | {YYYY-MM-DD} |
| Governance Model Version | 1.0 |
| Prompt Version | {prompt version} |
| Spec Version | {spec version} |
| Principles Version | {principles file versions} |

---

## 2. Flag Inventory

### {Flag Key}

| Field | Value |
|-------|-------|
| Flag Key | {exact identifier as it appears in the flag system} |
| Purpose | {what this flag controls and why it exists} |
| Type | release / ops / experiment / permission |
| Creation Date | {YYYY-MM-DD} |
| Creator | {team or role} |
| Status | active / retired |
| Retirement Date | {YYYY-MM-DD, if retired} |

*Repeat for each flag. Retired flags remain in the inventory.*

---

## 3. State Tracking

| Flag Key | Development | Staging | Production |
|----------|-------------|---------|------------|
| {flag key} | enabled / disabled / percentage (N%) / not deployed | {state} | {state} |

*Add columns for additional environments as needed. Every active flag must have state for every environment.*

---

## 4. Retirement Criteria

| Flag Key | Retirement Condition | Target Retirement Date | Permanent | Review Date | Justification |
|----------|---------------------|----------------------|-----------|-------------|---------------|
| {flag key} | {specific, evaluable condition that triggers retirement} | {YYYY-MM-DD} | No | — | — |
| {flag key} | — | — | Yes | {YYYY-MM-DD} | {justification for permanence} |

*Every flag must have a retirement condition and target date, OR be marked as permanent with review date and justification.*

---

## 5. Rollout History

### {Flag Key}

| Timestamp | Who | Change | Rationale |
|-----------|-----|--------|-----------|
| {YYYY-MM-DD or ISO 8601} | {team or role} | Created: {initial state per environment} | {why the flag was created} |
| {YYYY-MM-DD or ISO 8601} | {team or role} | {from state} → {to state} in {environment} | {why the change was made} |

*Repeat for each flag. Initial creation must be the first entry. Entries in chronological order.*

---

## 6. Stale Flag Assessment

**Assessment Date:** {YYYY-MM-DD — must match Document Control assessment date}

### Stale Flags

| Flag Key | Days Past Retirement | Current State | Reason for Delay | Updated Target Date |
|----------|---------------------|---------------|-------------------|-------------------|
| {flag key} | {N days} | {current state per environment} | {reason not yet retired} | {YYYY-MM-DD} |

*If no stale flags, write: "No stale flags identified as of {assessment date}."*
