# Data Migration Record — Spec

**Artifact type:** Data Migration Record (DMR)
**Kit:** Data & Configuration Kit (DCK), Layer 11
Version: v1.0

---

## Purpose

The Data Migration Record governs the planning, validation, and rollback strategy for data migrations triggered by schema changes. It ensures that data is migrated safely, validated against correctness criteria, and recoverable if the migration fails.

---

## Artifact ID Format

`DMR-{INITIATIVE}-{NNN}`

Example: `DMR-TASKFLOW-001`

---

## Trigger

Generate a DMR when a frozen DSR contains schema changes that require data migration — structural changes to existing data stores, data format transformations, or data movement between systems. If a DSR documents schema changes that are purely additive (new optional fields, new tables with no data population), a DMR is not required.

---

## Inputs

| Input | Source | Required |
|-------|--------|----------|
| Frozen DSR | DCK | Yes |
| Frozen TDD | EEK | Recommended (for behavioral context) |
| Frozen SAD | EEK | Recommended (for data store architecture) |
| Current data profiles | Consuming project | Recommended (volumes, distributions, edge cases) |

---

## Sections

### §1 Document Control

| Field | Value |
|-------|-------|
| DMR ID | `DMR-{INITIATIVE}-{NNN}` |
| Initiative | Full initiative name |
| Owner | Team or role responsible for migration execution |
| Version | Document version |
| Status | Draft / Validated / Frozen |
| DSR Reference | DSR artifact ID and version |
| Governance Model Version | Version of governance model in effect |
| Prompt Version | Version of the prompt used to generate this artifact |
| Spec Version | v1.0 |
| Principles Version | data-configuration-principles v1.0 |

### §2 Migration Inventory

A table listing every data migration required by the DSR schema changes.

| Migration ID | Source Schema | Target Schema | Data Store | Estimated Volume | Migration Type |
|-------------|-------------|--------------|-----------|-----------------|---------------|
| M-{NNN} | {current schema version} | {target schema version} | {database/store name} | {row count or data size} | Online / Offline / Hybrid |

Migration type definitions:
- **Online** — Migration runs while the system serves traffic. Requires dual-read/write or progressive migration.
- **Offline** — Migration requires downtime. System is unavailable during migration window.
- **Hybrid** — Migration starts online (backfill) and completes with a brief offline cutover.

### §3 Migration Strategy

For each migration in §2:

- **Approach** — Detailed migration steps (e.g., add new column → backfill → swap reads → drop old column)
- **Ordering constraints** — Dependencies between migrations (M-002 must complete before M-003)
- **Concurrency** — Which migrations can run in parallel
- **Duration estimate** — Expected migration time based on data volume and approach
- **Downtime required** — None / Planned window / Emergency-only

### §4 Data Validation Criteria

For each migration:

| Validation Check | Method | Expected Result | Blocking |
|-----------------|--------|----------------|----------|
| Row count match | COUNT(*) before/after | Equal (±tolerance) | Yes |
| Data integrity | Checksum or sample comparison | Match | Yes |
| Referential integrity | FK constraint validation | No orphans | Yes |
| Application smoke test | Key queries against migrated data | Expected results | Yes |
| {custom check} | {method} | {expected} | Yes/No |

Every migration must have at least one blocking validation check. Migrations without validation criteria fail the `validation_criteria` gate.

### §5 Rollback Procedure

For each migration:

| Migration ID | Rollback Strategy | Rollback Steps | Data Loss Risk | Rollback Duration |
|-------------|-------------------|---------------|---------------|-------------------|
| M-{NNN} | Reverse migration / Restore from backup / Forward-fix | {numbered steps} | None / Partial / Full | {estimated time} |

Rollback strategy definitions:
- **Reverse migration** — Apply inverse schema change and reverse data transformation. Preferred when transformation is reversible.
- **Restore from backup** — Restore the data store from a pre-migration backup. Requires verified backup.
- **Forward-fix** — Migration cannot be reversed; fix issues by applying a corrective migration. Must document why reversal is impossible.

If rollback is impossible (one-way transformation, external system dependency), this must be documented with explicit risk acceptance.

### §6 Execution Plan

| Phase | Activity | Timing | Owner | Pre-Condition |
|-------|----------|--------|-------|--------------|
| Pre-migration | Backup verification | {date/relative} | {who} | Backup completed and tested |
| Pre-migration | Dry run on staging | {date/relative} | {who} | Staging data representative |
| Migration | Execute migration {M-NNN} | {date/relative} | {who} | Dry run passed |
| Post-migration | Run validation checks | {date/relative} | {who} | Migration completed |
| Post-migration | Monitor for errors | {duration} | {who} | Validation passed |

A dry run on a non-production environment is required for every offline or hybrid migration.

---

## Hard Gates

| # | Gate | Rule |
|---|------|------|
| 1 | `migration_inventory` | Every schema change in the DSR that requires data migration has a corresponding entry in §2 with source/target schema, data store, estimated volume, and migration type. No migration is referenced in §3–§6 that is absent from §2. |
| 2 | `strategy_complete` | Every migration in §2 has a strategy in §3 with approach, ordering constraints, duration estimate, and downtime requirement. No migration lacks a strategy. |
| 3 | `validation_criteria` | Every migration has at least one blocking validation check in §4 with method and expected result. No migration lacks validation criteria. |
| 4 | `rollback_defined` | Every migration has a rollback procedure in §5 with strategy, steps, data loss risk, and duration. Migrations that cannot be reversed have explicit documentation of why and risk acceptance. |
| 5 | `dry_run_required` | Every offline or hybrid migration has a dry run step in §6 on a non-production environment. Online migrations should have a dry run but it is not blocking. |

---

## Scope Boundaries

- DMR scope is bounded by the frozen DSR. Do not include migrations for schema changes not documented in the DSR.
- DMR governs migration planning, not execution monitoring. Execution tracking is an operational concern.
- DMR does not replace the DSR's migration plan section — the DSR §4 defines the high-level migration approach; the DMR provides the detailed execution plan, validation criteria, and rollback procedures.
- If no schema changes require data migration, no DMR is needed. Document this in the ER: "DMR not required — DSR schema changes are purely additive."
