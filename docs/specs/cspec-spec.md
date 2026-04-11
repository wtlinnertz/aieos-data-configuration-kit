# Configuration Specification — Specification

Version: v1.0

The Configuration Specification (CSPEC) defines what configuration a system requires and how it should be managed. It inventories all configuration items with their types, defaults, and validation rules; documents per-environment overrides; identifies secrets and their handling methods; and classifies change impact (restart vs. hot-reload).

The CSPEC is generated after the TDD is frozen in EEK, when the system's configuration requirements are known.

---

## What This Artifact Is Not

- **Not a deployment manifest.** The CSPEC defines configuration structure and rules — not the actual values deployed to a specific environment at a specific time. Deployment manifests are operational artifacts that implement the CSPEC's specifications.
- **Not a secrets management policy.** The CSPEC identifies which configuration items are secrets and specifies their handling method. Organization-wide secrets management policy belongs in principles files.
- **Not a runtime configuration file.** The CSPEC is a governance document describing what configuration exists and how it should be validated. It does not replace `.env` files, config maps, or parameter stores.

---

## Purpose

The CSPEC serves three roles:

1. **Configuration contract** — Defines the complete set of configuration items a system requires, making implicit configuration explicit
2. **Validation standard** — Specifies how each configuration item must be validated (type, range, enum, pattern), enabling automated config verification
3. **Environment coverage map** — Documents per-environment values or inheritance, ensuring every target environment has a defined configuration state

---

## Upstream Dependencies

- Frozen TDD (from EEK) — provides configuration requirements and data models
- Frozen ORD (from EEK) — provides config readiness checks
- Organizational principles (if applicable) — configuration management policy

---

## Required Sections

1. Document Control
2. Configuration Inventory
3. Environment Coverage
4. Secret Identification
5. Validation Rules
6. Change Impact

---

## Content Rules

### Document Control
**Rules**
- CSPEC ID must be present (format: CSPEC-{PROJECT}-{NNN})
- System name must be present
- Owner must be present as a team or role, not an individual
- Version must be present (format: v{N}, e.g., v1, v2)
- Status must be one of: Draft | Validated | Frozen
- TDD reference must be present (the frozen TDD this CSPEC is derived from)

**Failure Examples**
- CSPEC ID absent
- Owner listed as an individual person's name
- TDD reference absent or pointing to an unfrozen TDD

### Configuration Inventory
**Rules**
- Every configuration item must be listed with:
  - A name (the configuration key as it appears in the system)
  - A type (string, integer, boolean, float, duration, URL, enum, or custom type with definition)
  - A default value (or explicit statement that no default exists and the value is required)
  - A description (what this configuration controls)
  - A validation rule reference (cross-reference to §5)
- Configuration items must be sourced from the frozen TDD — items not in the TDD must not appear in the CSPEC
- Each configuration item name must be unique within the CSPEC

**Failure Examples**
- Configuration item listed without type
- Default value absent with no "required, no default" statement
- Configuration items present that are not traceable to the TDD
- Duplicate configuration item names

### Environment Coverage
**Rules**
- Every target environment must be listed (e.g., development, staging, production)
- For each environment, every configuration item must have either:
  - An explicit value defined for that environment, or
  - An explicit inheritance statement (e.g., "inherits from staging" or "uses default")
- No environment may have undefined configuration items — gaps must be explicitly documented with a resolution plan

**Failure Examples**
- Environment listed with only partial configuration coverage
- Configuration item has no value and no inheritance statement for an environment
- "TBD" without a resolution plan and target date

### Secret Identification
**Rules**
- Every configuration item that contains sensitive data must be identified as a secret
- For each secret, the handling method must be specified:
  - Storage method (e.g., "environment variable injected from secrets manager," "encrypted at rest in config store")
  - Access control (who/what can read the secret)
  - Rotation policy (how often the secret is rotated, or "no rotation — static credential" with justification)
- No secret may have a plaintext default value in the CSPEC
- If no configuration items are secrets, this section must explicitly state "No secrets identified"

**Failure Examples**
- Secret identified but handling method absent
- Secret with a plaintext default value (e.g., `default: "my-api-key-12345"`)
- Database connection string not identified as a secret
- Section absent entirely

### Validation Rules
**Rules**
- Every configuration item listed in §2 must have a validation rule
- Each validation rule must specify at least one of:
  - Type check (matches the declared type)
  - Range (for numeric types: min/max bounds)
  - Enum (for enumerated types: the complete set of valid values)
  - Pattern (for string types: a regex or format description, e.g., "valid URL," "IPv4 address")
- Validation rules must be specific enough to be machine-evaluable — "valid value" is not a validation rule

**Failure Examples**
- Configuration item in §2 with no corresponding validation rule in §5
- Validation rule stated as "must be valid" without specifics
- Enum type declared but valid values not listed
- Numeric type with no range specified and no justification for unbounded

### Change Impact
**Rules**
- Every configuration item must be classified as one of:
  - **Hot-reload** — change takes effect without restart
  - **Restart required** — change requires service restart to take effect
  - **Redeployment required** — change requires a new deployment
- The classification must be stated per configuration item, not as a blanket statement
- If the change impact is unknown for any item, it must be documented as a gap with a resolution plan

**Failure Examples**
- Change impact not classified for one or more items
- Blanket statement like "all configuration changes require restart"
- "Unknown" without resolution plan

---

## Format Requirements

- Configuration item names must be exact strings matching the system's configuration keys
- Types must use the type vocabulary defined in §2 rules (string, integer, boolean, float, duration, URL, enum, or custom with definition)
- Environment names must be consistent throughout the document

---

## Completeness Rules

- All six sections must be present and non-empty
- Every configuration item has name, type, default (or required statement), description, and validation rule reference
- Every environment has complete coverage (explicit value or inheritance for every item)
- Every secret has a handling method with no plaintext defaults
- Every item has a change impact classification

---

## Hard Gates

1. **config_inventory** — All configuration items identified with name, type, default (or required statement), description, and validation rule reference; items traceable to TDD; no duplicates
2. **environment_coverage** — Every target environment listed; every configuration item has defined value or explicit inheritance for every environment; no undefined gaps without resolution plan
3. **secret_identification** — All secrets identified with handling method (storage, access control, rotation policy); no plaintext defaults for secrets; section present even if no secrets exist
4. **validation_rules** — Every configuration item has a validation rule (type check, range, enum, or pattern); rules are specific enough to be machine-evaluable
5. **change_impact** — Every configuration item classified as hot-reload, restart required, or redeployment required; no unclassified items without documented resolution plan
