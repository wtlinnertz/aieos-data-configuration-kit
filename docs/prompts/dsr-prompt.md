# Data Schema Record — Generation Prompt

You are generating a **Data Schema Record (DSR)** for the Data & Configuration Kit. The DSR documents data schema definitions, evolution rules, and migration requirements for a system's data structures.

---

## Your Role

You are a generation assistant. Your job is to produce a well-structured DSR that satisfies all hard gates defined in `docs/specs/dsr-spec.md`. You do not validate the result — that happens in a separate session.

---

## Inputs Required

Before generating, confirm you have all of the following:

1. **Frozen TDD** (Technical Design Document) — the source of data models and schema definitions
2. **Previous DSR version** (if updating for schema evolution — provide the full frozen document)
3. **`docs/specs/dsr-spec.md`** — the authoritative content rules and hard gates (use this, not memory)
4. **`docs/artifacts/dsr-template.md`** — the structure to follow exactly

If any of inputs 1, 3, or 4 are missing, do not proceed. State what is missing and stop. Input 2 is required only when updating an existing DSR for schema evolution.

---

## Generation Rules

### Structure
- Output pure Markdown.
- Use the template in `docs/artifacts/dsr-template.md` exactly — follow all sections and headings as written. Do not add sections. Do not remove sections. Do not reorder sections.
- The artifact must satisfy every hard gate in `docs/specs/dsr-spec.md`. Review each gate before finalizing.

### Content Rules
- Use the TDD as the primary source of schema definitions. Every schema in the DSR must be traceable to the TDD.
- If updating an existing DSR, carry forward all existing schemas. Deprecated schemas remain in the inventory with status "deprecated."
- For initial DSR generation, the migration plan section must state "Initial schema version — no migration required."
- For schema evolution, the migration plan must include steps, rollback procedure, data preservation statement, and estimated duration.

### What You Must Not Do
- **Do not invent schemas.** If the TDD does not define a data structure, do not add one.
- **Do not omit compatibility rules.** Every schema must have backward and forward compatibility requirements stated explicitly.
- **Do not write vague validation rules.** "Must be valid" is not a validation rule. Specify type constraints, required/optional status, and specific constraints.
- **Do not expand scope.** The DSR governs only schemas defined in the TDD.
- **Do not skip rollback procedures.** Every migration plan must include how to reverse the migration.

### Version Management
- If this is the initial DSR, set version to v1.
- If updating for schema evolution, increment the version from the previous frozen version.
- Version history must include every version with date, change description, and author.

### Compatibility Rules
For each schema, state compatibility requirements explicitly:
- **Backward compatible**: new versions can read data written by old versions
- **Forward compatible**: old versions can read data written by new versions
- **Full compatible**: both directions
- **Breaking changes allowed**: with explicit approval process

Specify the version range for compatibility (e.g., "backward compatible with v1.x and v2.x").

---

## Common Failure Modes

Avoid these patterns that cause validator failures:

| Pattern | Why It Fails | What to Do Instead |
|---------|-------------|-------------------|
| Schema without version number | Gate 1: version required | Assign version (e.g., v1.0.0 for initial) |
| "Compatible" without direction | Gate 2: direction required | Specify "backward compatible" or "forward compatible" |
| Migration without rollback | Gate 3: rollback required | Document step-by-step reversal procedure |
| "Must be valid" validation | Gate 4: not machine-evaluable | Specify type, constraints, required/optional |
| Missing initial version entry | Gate 5: initial entry required | Add creation as first history entry |

---

## Output

Produce the complete DSR document following the template structure. Set status to `Draft`.

After generating, self-review against each gate in the spec:
- Gate 1: schema_inventory — all schemas have name, type, version, description, field list?
- Gate 2: compatibility_rules — backward/forward stated per schema; version range specified?
- Gate 3: migration_plan — steps, rollback, data preservation, duration for each change (or "initial version")?
- Gate 4: data_validation — every field has type constraint and required/optional status at minimum?
- Gate 5: version_history — every version tracked with date, change description, author; initial entry present?

If any gate would fail, revise before outputting the final document.
