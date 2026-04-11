# Configuration Specification

---

## 1. Document Control

| Field | Value |
|-------|-------|
| CSPEC ID | CSPEC-{PROJECT}-{NNN} |
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

## 2. Configuration Inventory

### {Configuration Item Name}

| Field | Value |
|-------|-------|
| Name | {exact configuration key as it appears in the system} |
| Type | {string / integer / boolean / float / duration / URL / enum / custom} |
| Default | {default value, or "Required — no default"} |
| Description | {what this configuration controls} |
| Validation Rule | See §5: {rule reference} |
| Secret | Yes / No |

*Repeat for each configuration item. All items must be traceable to the frozen TDD.*

---

## 3. Environment Coverage

| Configuration Item | Development | Staging | Production |
|-------------------|-------------|---------|------------|
| {item name} | {value or "inherits from {env}" or "uses default"} | {value or inheritance} | {value or inheritance} |

*Add columns for additional environments as needed. Every item must have a defined value or explicit inheritance for every environment.*

**Undefined Items:**

| Configuration Item | Environment | Resolution Plan | Target Date |
|-------------------|-------------|-----------------|-------------|
| {item name} | {environment} | {plan to resolve} | {YYYY-MM-DD} |

*If no undefined items, write: "All configuration items have defined values or inheritance for all environments."*

---

## 4. Secret Identification

### {Secret Name}

| Field | Value |
|-------|-------|
| Configuration Item | {item name from §2} |
| Storage Method | {e.g., "environment variable injected from secrets manager"} |
| Access Control | {who/what can read this secret} |
| Rotation Policy | {rotation frequency, or "no rotation — static credential" with justification} |

*Repeat for each secret. If no secrets, write: "No secrets identified."*

---

## 5. Validation Rules

### {Configuration Item Name}

| Field | Value |
|-------|-------|
| Type Check | {matches declared type from §2} |
| Range | {min/max bounds for numeric types, or "N/A"} |
| Enum Values | {complete set of valid values for enum types, or "N/A"} |
| Pattern | {regex or format description for string types, or "N/A"} |
| Additional Constraints | {any other validation constraints, or "None"} |

*Repeat for each configuration item in §2.*

---

## 6. Change Impact

| Configuration Item | Change Impact | Notes |
|-------------------|---------------|-------|
| {item name} | Hot-reload / Restart required / Redeployment required | {any additional context} |

**Unclassified Items:**

| Configuration Item | Resolution Plan | Target Date |
|-------------------|-----------------|-------------|
| {item name} | {plan to determine impact} | {YYYY-MM-DD} |

*If no unclassified items, write: "All configuration items have classified change impact."*
