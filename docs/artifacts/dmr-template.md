# Data Migration Record

## §1 Document Control

| Field | Value |
|-------|-------|
| DMR ID | DMR-{INITIATIVE}-{NNN} |
| Initiative | {full initiative name} |
| Owner | {team or role responsible for migration execution} |
| Version | v1.0 |
| Status | Draft |
| DSR Reference | {DSR artifact ID and version} |
| Governance Model Version | {version from §15 of governance-model.md} |
| Prompt Version | {prompt file version used} |
| Spec Version | v1.0 |
| Principles Version | data-configuration-principles v1.0 |

---

## §2 Migration Inventory

| Migration ID | Source Schema | Target Schema | Data Store | Estimated Volume | Migration Type |
|-------------|-------------|--------------|-----------|-----------------|---------------|
| M-001 | {current schema version} | {target schema version} | {database/store name} | {row count or data size} | Online / Offline / Hybrid |

---

## §3 Migration Strategy

### M-001: {Migration Name}

**Approach:**
1. {step}
2. {step}

**Ordering constraints:** {dependencies on other migrations, or "None"}

**Concurrency:** {can run in parallel with M-XXX, or "Sequential only"}

**Duration estimate:** {expected time}

**Downtime required:** None / Planned window ({duration}) / Emergency-only

*(Repeat for each migration)*

---

## §4 Data Validation Criteria

### M-001: {Migration Name}

| Validation Check | Method | Expected Result | Blocking |
|-----------------|--------|----------------|----------|
| Row count match | COUNT(*) before/after | Equal (±{tolerance}) | Yes |
| Data integrity | {checksum/sample comparison} | Match | Yes |
| Referential integrity | FK constraint validation | No orphans | Yes |
| Application smoke test | {key queries} | {expected results} | Yes |

*(Repeat for each migration)*

---

## §5 Rollback Procedure

| Migration ID | Rollback Strategy | Rollback Steps | Data Loss Risk | Rollback Duration |
|-------------|-------------------|---------------|---------------|-------------------|
| M-001 | Reverse migration / Restore from backup / Forward-fix | 1. {step} 2. {step} | None / Partial / Full | {estimated time} |

---

## §6 Execution Plan

| Phase | Activity | Timing | Owner | Pre-Condition |
|-------|----------|--------|-------|--------------|
| Pre-migration | Backup verification | {date} | {who} | Backup completed and tested |
| Pre-migration | Dry run on staging | {date} | {who} | Staging data representative |
| Migration | Execute M-001 | {date} | {who} | Dry run passed |
| Post-migration | Run validation checks | {date} | {who} | Migration completed |
| Post-migration | Monitor for errors | {duration} | {who} | Validation passed |
