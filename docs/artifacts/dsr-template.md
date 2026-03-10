# Data Schema Record

---

## 1. Document Control

| Field | Value |
|-------|-------|
| DSR ID | DSR-{PROJECT}-{NNN} |
| System Name | {system name} |
| Owner | {team or role — not an individual person} |
| Version | v{N} |
| Status | Draft / Validated / Freeze Pending / Frozen |
| TDD Reference | {TDD-{PROJECT}-{NNN}} |
| Governance Model Version | 1.0 |
| Prompt Version | {prompt version} |
| Spec Version | {spec version} |
| Principles Version | {principles file versions} |

---

## 2. Schema Inventory

### {Schema Name}

| Field | Value |
|-------|-------|
| Schema Name | {identifier used in the system} |
| Schema Type | database / api_contract / event_schema / config_format |
| Current Version | {version number} |
| Description | {what data this schema governs} |

**Fields:**

| Field Name | Type | Required | Description |
|-----------|------|----------|-------------|
| {field name} | {type} | Yes / No | {description} |

*Repeat for each schema. All schemas must be traceable to the frozen TDD.*

---

## 3. Compatibility Rules

| Schema Name | Backward Compatible | Forward Compatible | Version Range | Breaking Change Process |
|-------------|--------------------|--------------------|---------------|------------------------|
| {schema name} | Yes / No | Yes / No | {e.g., "v1.x and v2.x"} | {approval process, or "N/A — no breaking changes allowed"} |

*Every schema must have compatibility rules stated.*

---

## 4. Migration Plan

### {Schema Name}: {source version} → {target version}

| Field | Value |
|-------|-------|
| Source Version | {version} |
| Target Version | {version} |
| Estimated Duration | {time estimate or impact window} |
| Data Preservation | {what happens to existing data during migration} |

**Migration Steps:**

1. {step 1}
2. {step 2}
3. {step N}

**Rollback Procedure:**

1. {rollback step 1}
2. {rollback step 2}
3. {rollback step N}

*Repeat for each schema change. If initial version with no changes: "Initial schema version — no migration required."*

---

## 5. Data Validation

### {Schema Name}

| Field Name | Type Constraint | Required | Constraints | Default |
|-----------|----------------|----------|-------------|---------|
| {field name} | {type} | Yes / No | {min/max, regex, foreign key, etc.} | {default value or "None"} |

*Repeat for each schema. Every field from §2 must have validation rules.*

---

## 6. Version History

### {Schema Name}

| Version | Date | Change Description | Author |
|---------|------|--------------------|--------|
| {version} | {YYYY-MM-DD} | {what changed from previous version, or "Initial schema creation"} | {team or role} |

*Repeat for each schema. Initial version must be first entry. Entries in chronological order.*
