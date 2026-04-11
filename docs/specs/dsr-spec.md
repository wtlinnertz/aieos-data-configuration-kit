# Data Schema Record — Specification

Version: v1.0

The Data Schema Record (DSR) documents data schema definitions, evolution rules, and migration requirements. It covers database schemas, API contracts, event schemas, and configuration file formats. The DSR tracks schema versions and provides migration procedures with rollback plans.

The DSR is generated during EEK when the TDD defines data models, and is versioned as schemas evolve over time.

---

## What This Artifact Is Not

- **Not a database migration script.** The DSR documents what schemas exist, how they evolve, and what the migration plan is. The actual migration scripts are implementation artifacts that execute the DSR's specifications.
- **Not an API documentation tool.** The DSR governs schema structure and evolution rules. API documentation (endpoint descriptions, usage examples) belongs in the ORD or external API docs.
- **Not a data dictionary.** The DSR focuses on schema structure, compatibility, and migration. Business-level data definitions and glossaries are separate documentation concerns.

---

## Purpose

The DSR serves three roles:

1. **Schema inventory** — Provides a single source of truth for all data schemas a system manages, with current version numbers
2. **Evolution governance** — Defines compatibility requirements and migration procedures, preventing uncontrolled schema changes
3. **Migration safety net** — Documents migration steps with rollback procedures, ensuring schema changes can be reversed if needed

---

## Upstream Dependencies

- Frozen TDD (from EEK) — provides data models and schema definitions
- Previous DSR version (if updating for schema evolution)

---

## Required Sections

1. Document Control
2. Schema Inventory
3. Compatibility Rules
4. Migration Plan
5. Data Validation
6. Version History

---

## Content Rules

### Document Control
**Rules**
- DSR ID must be present (format: DSR-{PROJECT}-{NNN})
- System name must be present
- Owner must be present as a team or role, not an individual
- Version must be present (format: v{N}, e.g., v1, v2)
- Status must be one of: Draft | Validated | Frozen
- TDD reference must be present (the frozen TDD this DSR is derived from)

**Failure Examples**
- DSR ID absent
- Owner listed as an individual person's name
- TDD reference absent

### Schema Inventory
**Rules**
- Every schema must be listed with:
  - A schema name (the identifier used in the system)
  - A schema type: one of database | api_contract | event_schema | config_format | custom (with definition)
  - A current version number
  - A description (what data this schema governs)
  - A list of fields with their types
- Schema names must be unique within the DSR
- Schemas must be sourced from the frozen TDD — schemas not in the TDD must not appear in the DSR

**Failure Examples**
- Schema listed without version number
- Schema listed without type classification
- Schema fields not listed
- Schemas present that are not traceable to the TDD

### Compatibility Rules
**Rules**
- For each schema, backward and forward compatibility requirements must be stated
- Compatibility must be expressed as one of:
  - **Backward compatible** — new versions can read data written by old versions
  - **Forward compatible** — old versions can read data written by new versions
  - **Full compatible** — both backward and forward compatible
  - **Breaking changes allowed** — with explicit approval process and migration requirement
- If compatibility requirements differ by schema, they must be stated per schema, not as a blanket rule
- The version range for compatibility must be specified (e.g., "backward compatible with v1.x and v2.x")

**Failure Examples**
- Compatibility requirements absent for a schema
- "Compatible" without specifying direction (backward/forward)
- No version range specified
- Breaking changes allowed without approval process

### Migration Plan
**Rules**
- For each schema change (from the previous version or from initial creation), a migration plan must be present
- Each migration plan must include:
  - Source version and target version
  - Migration steps (ordered list of actions)
  - Rollback procedure (how to reverse the migration)
  - Data preservation statement (what happens to existing data during migration)
  - Estimated duration or impact window
- If no schema changes have occurred (initial DSR), this section must state "Initial schema version — no migration required"

**Failure Examples**
- Schema change without migration steps
- Migration steps without rollback procedure
- No data preservation statement
- "TBD" migration plan

### Data Validation
**Rules**
- For each schema field listed in §2, validation rules must be defined
- Each validation rule must specify at least one of:
  - Type constraint (matches the declared type)
  - Required/optional status
  - Constraints (min/max length, value range, regex pattern, foreign key reference)
  - Default value (if the field has one)
- Validation rules must be specific enough to be machine-evaluable

**Failure Examples**
- Schema field with no validation rule
- Validation stated as "must be valid" without specifics
- Required/optional status not stated
- Constraints described qualitatively ("short string")

### Version History
**Rules**
- Every schema version must be tracked with:
  - Version number
  - Date of the version
  - Change description (what changed from the previous version)
  - Author (team or role)
- The initial version must be the first entry
- Entries must be in chronological order
- If the DSR is at its initial version, the first entry must document the initial schema creation

**Failure Examples**
- Version without date
- Version without change description
- Initial version entry missing
- Entries out of chronological order

---

## Format Requirements

- Schema names must be exact strings matching the system's schema identifiers
- Version numbers must follow a consistent format (semver recommended but not required)
- Dates must be in ISO 8601 format (YYYY-MM-DD) or equivalent unambiguous format
- Field types must use a consistent type vocabulary throughout the document

---

## Completeness Rules

- All six sections must be present and non-empty
- Every schema has name, type, current version, description, and field list
- Compatibility rules stated for every schema
- Migration plan present for every schema change (or "initial version" statement)
- Validation rules defined for every schema field
- Version history tracked for every schema version

---

## Relationship Rules

- The DSR is versioned — a new version is required when schemas change
- A frozen DSR may not be edited — changes require a new version via the Schema Evolution Protocol (see playbook)
- Schema migrations must not be executed until the DSR is frozen
- If a schema is deprecated, it remains in the inventory with status "deprecated" and deprecation date

---

## Hard Gates

1. **schema_inventory** — All schemas identified with schema name, type (database | api_contract | event_schema | config_format), current version, description, and field list; names unique; schemas traceable to TDD
2. **compatibility_rules** — Backward and forward compatibility requirements stated per schema; direction specified; version range specified; breaking changes have approval process documented
3. **migration_plan** — Schema changes have migration steps with source/target version, ordered actions, rollback procedure, data preservation statement, and estimated duration; initial version has explicit "no migration required" statement
4. **data_validation** — Validation rules defined for every schema field; rules include type constraint and required/optional status at minimum; rules are specific enough to be machine-evaluable
5. **version_history** — Every schema version tracked with version number, date, change description, and author; initial version entry present; entries in chronological order
