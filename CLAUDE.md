# CLAUDE.md — Data & Configuration Kit

## What This Repository Is

This is the **Data & Configuration Kit** — an AIEOS cross-cutting kit that governs configuration management, feature flag lifecycle, and data schema evolution across environments. It is Layer 11 of the AIEOS system. It receives inputs from multiple layers (primarily EEK and REK) and provides outputs to REK, RRK, and other downstream consumers. Unlike linear kits, its artifacts are triggered at different pipeline points rather than following a single sequential flow.

## Repository Structure

```
docs/
  principles/          # Organizational configuration and data policy (input material)
  specs/               # Content rules and hard gates per artifact type
  artifacts/           # Templates
  prompts/             # AI generation prompts
  validators/          # Quality gate definitions
  playbook.md          # End-to-end process definition
  index.md             # Documentation entry point
  how-to-adapt.md      # Organizational adoption guidance
  how-to-use-with-ai.md # AI tool usage guide
  governance-model.md  # AIEOS structural rules (reference)
  entry-from-eek.md    # Boundary briefing from Engineering Execution Kit
examples/              # Worked examples
tests/                 # Structural integrity checks
```

## Artifact Types

This kit produces four governed artifact types:

1. **Configuration Specification (CSPEC)** — Defines configuration structure, validation rules, default values, and per-environment overrides. Establishes what configuration a system requires and how it should be managed.
2. **Feature Flag Lifecycle Record (FFLR)** — Tracks feature flags from creation through retirement. Documents flag purpose, current state per environment, rollout history, and retirement criteria. The FFLR is a living document that is re-frozen periodically.
3. **Data Schema Record (DSR)** — Documents data schema definitions, evolution rules, and migration requirements. Covers database schemas, API contracts, event schemas, and configuration file formats.
4. **Data Migration Record (DMR)** — Migration planning, validation criteria, and rollback procedures for schema-driven data migrations. 5 hard gates.

Each artifact type has exactly four governing files: spec, template, prompt, validator.

## Key Rules

- **Specs are the source of truth** — prompts and validators reference specs, never inline rules
- **Validators judge, they do not help** — no suggestions, no redesign
- **Freeze before promote** — upstream artifacts (TDD, ORD, RR) must be frozen before DCK artifact generation
- **Separate generation and validation** — different AI sessions to prevent self-validation bias
- **No scope expansion** — downstream artifacts must not expand scope beyond upstream
- **No inferred information** — mark missing information explicitly, do not fill gaps
- **FFLR is a living document** — re-frozen at each RHR cycle or when a flag is retired
- **DSR is versioned** — when schemas change, issue a new DSR version; do not edit a frozen DSR
- **Governance model sync** — `docs/governance-model.md` is a synchronized copy of `aieos-governance-foundation/governance-model.md` (canonical authority). Do not edit kit copy directly; update `aieos-governance-foundation` first, then sync all kit copies to match exactly. See governance-model.md §15 for versioning and change protocol.
- **Engagement Record** — DCK maintains the Layer 11 section of the project's ER. Add artifact IDs as they freeze. See `docs/playbook.md`.

## Artifact Flow

```
Trigger: TDD frozen (EEK)
  → Configuration Specification (CSPEC) → validate → freeze

Trigger: Feature flags created (REK)
  → Feature Flag Lifecycle Record (FFLR) → validate → freeze
    (living document — re-frozen at RHR cycle or flag retirement)

Trigger: Schema design (EEK/TDD)
  → Data Schema Record (DSR) → validate → freeze
    (versioned — new version when schemas evolve)

Trigger: DSR frozen with schema changes requiring data migration
  → Data Migration Record (DMR) → validate → freeze
```

## Boundary Contracts

- **Upstream (EEK):** Receives frozen TDD (config requirements, data models) and ORD (config readiness checks). See `docs/entry-from-eek.md` for the boundary briefing.
- **Upstream (REK):** Receives RR (feature flag states at release time) and RP (flag-based exposure strategy).
- **Downstream (REK):** CSPEC provides config validation criteria for release verification.
- **Downstream (RRK):** Config drift detection requirements from CSPEC; FFLR stale flag alerts.
- **Upstream (RRK):** Config-related incident data (IR), config drift observations (RHR).
- **Upstream (ODK):** Config-related PMR findings may trigger CSPEC revision.

## File Naming

| Type | Pattern |
|------|---------|
| Spec | `{type}-spec.md` |
| Template | `{type}-template.md` |
| Prompt | `{type}-prompt.md` |
| Validator | `{type}-validator.md` |

## When Working on This Kit

- Read the playbook (`docs/playbook.md`) for the full process definition
- Read the governance model (`docs/governance-model.md`) for structural rules
- Check `docs/how-to-use-with-ai.md` for session setup instructions
- Reference `docs/entry-from-eek.md` when arriving from the Engineering Execution Kit

## Building or Auditing AIEOS Kits

- `aieos-governance-foundation/docs/kit-structure-standard.md` — compliance checklist for building and auditing kits
- `aieos-governance-foundation/docs/philosophy.md` — design rationale for governance model decisions
- `aieos-governance-foundation/docs/layer-model.md` — layer model and kit registry

## Commit Message Style

Follow conventional commits: `docs: <description>`
