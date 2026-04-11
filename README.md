# aieos-data-configuration-kit

**Layer 11 of the AIEOS system — Data & Configuration (Cross-Cutting)**

This kit governs how configuration is structured, feature flags are managed through their lifecycle, and data schemas evolve across environments. It is a cross-cutting kit that receives inputs from multiple layers and provides outputs to multiple downstream consumers.

---

## What This Kit Does

Engineering produces systems that depend on configuration, feature flags, and data schemas. But "configuration exists" is not the same as "configuration is governed." This kit governs the gap:

- **Configuration structure** — What configuration does a system require? What are the valid values, defaults, and per-environment overrides?
- **Feature flag lifecycle** — What flags exist, what state are they in per environment, when should they be retired?
- **Data schema evolution** — How do schemas change over time? What are the compatibility requirements and migration procedures?

---

## Artifact Types

This kit produces three governed artifact types:

| Trigger | Artifact | Purpose |
|---------|----------|---------|
| TDD frozen (EEK) | Configuration Specification (CSPEC) | Defines configuration structure, validation rules, defaults, and per-environment overrides |
| Feature flags created (REK) | Feature Flag Lifecycle Record (FFLR) | Tracks flags from creation through retirement with per-environment state |
| Schema design (EEK/TDD) | Data Schema Record (DSR) | Documents schema definitions, evolution rules, and migration requirements |

Each governed artifact type has exactly four governing files: spec, template, prompt, validator.

---

## Quick Start

1. Read `docs/playbook.md` — the complete process definition
2. Read `docs/how-to-use-with-ai.md` — session setup and AI tool guidance
3. See `docs/entry-from-eek.md` — boundary briefing when arriving from the Engineering Execution Kit

---

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
  entry-from-eek.md    # Boundary briefing from EEK
examples/              # Worked examples
tests/
  kit-test-plan.md     # Structural integrity checks and flow scenarios
CLAUDE.md              # AI operating instructions
```

---

## AIEOS Layer

| Layer | Kit | Status |
|-------|-----|--------|
| 4. Engineering Execution | `aieos-engineering-execution-kit` | Built |
| 5. Release & Exposure | `aieos-release-exposure-kit` | Built |
| 6. Reliability & Resilience | `aieos-reliability-resilience-kit` | Built |
| **11. Data & Configuration** | **`aieos-data-configuration-kit`** | **Built** |

See `aieos-governance-foundation/docs/layer-model.md` for the full layer model.
