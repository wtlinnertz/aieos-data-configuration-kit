# How to Adapt This Kit

This kit provides the structure, rules, and prompts for configuration, feature flag, and data schema governance. Adapting it to your organization means filling in the content that is specific to your context — without modifying the governance structure.

---

## What to Adapt

### Organizational Principles

**Directory:** `docs/principles/`

Create principle files that define your organization's policies for:

- **Configuration management**: How does your organization handle configuration across environments? What is the approval process for production config changes? Are there mandatory configuration review requirements?
- **Feature flag policy**: What types of flags does your organization use? What is the maximum lifetime for temporary flags? Who can create, modify, and retire flags?
- **Data schema governance**: What compatibility requirements does your organization enforce? Are breaking schema changes allowed? What is the review process for schema migrations?

### Environment Definitions

The CSPEC template uses environment names as placeholders. Define your organization's standard environments (e.g., development, staging, production, or dev, qa, uat, prod) and document them in your principles files.

### Flag Type Taxonomy

The FFLR spec defines four flag types: release, ops, experiment, permission. If your organization uses a different taxonomy, document it in a principles file and reference it during FFLR generation.

### Schema Compatibility Standards

The DSR spec requires compatibility rules (backward/forward). If your organization has specific compatibility standards (e.g., "all API changes must be backward compatible for 2 major versions"), document them in a principles file.

---

## What Not to Adapt

### Specs

The specs define what makes an artifact valid. Do not soften hard gates to make artifacts easier to produce. If a hard gate is failing consistently, that usually means the artifact is incomplete — not that the gate is wrong.

If you genuinely need to add a hard gate (your organization requires something the spec does not check), add it. Do not remove existing hard gates.

### Validator Logic

Validators evaluate against specs. If a validator is producing unexpected results, check whether the spec accurately captures your requirements — and adjust the spec if needed, not the validator.

### Governance Model

`docs/governance-model.md` is a synchronized copy of the canonical governance model. Do not edit it. If you believe the governance model should change, update `aieos-governance-foundation/governance-model.md` and sync all kit copies.

---

## Adding Artifact Types

If your organization needs additional governed artifacts (e.g., a secrets rotation record, an environment provisioning spec), follow the four-file system:

1. Write the spec first — define the hard gates before writing anything else
2. Write the validator — this forces you to verify the spec is evaluable
3. Write the template — structure only, no content rules
4. Write the prompt — generation behavior, references spec and template

Register the new artifact type in the playbook, index, and CLAUDE.md.

---

## Tool Bindings

This kit is tool-agnostic. Configuration items, feature flags, and schemas are described generically.

When adopting this kit, create bindings documents:

```
docs/bindings/
  config-system-mapping.md       # Maps CSPEC items to your config management system
  feature-flag-mapping.md        # Maps FFLR flags to your feature flag platform
  schema-registry-mapping.md     # Maps DSR schemas to your schema registry or ORM
```

Bindings are not governed artifacts — they have no spec, validator, or prompt. Update them when your tooling changes without touching the governed files.

---

## First-Time Setup Checklist

- [ ] Read `docs/playbook.md` fully before beginning
- [ ] Identify which artifact types you need for your initiative (CSPEC, FFLR, DSR, or a combination)
- [ ] Obtain frozen upstream artifacts (TDD from EEK for CSPEC/DSR; RR from REK for FFLR)
- [ ] Review and populate `docs/principles/` with your organizational policies
- [ ] Generate and freeze the applicable artifacts
- [ ] Confirm downstream consumers (REK, RRK) are aware of the frozen artifacts
