# How to Use This Kit with AI

This guide explains how to set up AI sessions for each artifact type in the Data & Configuration Kit workflow. Follow the session setup instructions precisely — incorrect session setup is the most common cause of poor artifact quality.

---

## Core Discipline

**One artifact per session.** Do not generate multiple artifacts in the same session.

**Separate generation and validation.** Always validate in a new session. Never ask the AI that generated an artifact to validate it — this produces self-validation bias.

**Include full frozen documents.** Do not summarize upstream artifacts. Provide the complete document.

---

## CSPEC — Generation Session

**Session setup:**
```
Documents to provide:
1. Frozen TDD (full document — especially config requirements section)
2. Frozen ORD (full document — config readiness checks)
3. docs/specs/cspec-spec.md
4. docs/artifacts/cspec-template.md

Prompt:
"Generate a Configuration Specification using the provided inputs.
Follow the prompt in docs/prompts/cspec-prompt.md.
Use the template exactly. Satisfy all hard gates in the spec.
Do not invent configuration items not in the TDD.
Mark any missing information with [MISSING: reason]. Output pure Markdown."
```

**After generation:** Review the CSPEC. Confirm:
- All configuration items from the TDD are present
- Secrets are correctly identified with handling method specified
- Every environment has defined values or explicit inheritance documented
- Validation rules are defined for every configuration item
- Change impact (restart vs. hot-reload) is classified for every item

**Validation session setup:**
```
Documents to provide:
1. The generated CSPEC (full document)
2. docs/specs/cspec-spec.md

Prompt:
"Validate this Configuration Specification against the CSPEC spec.
Use only the spec as the source of truth for pass/fail criteria.
Do not suggest improvements. Judge only what is explicitly present.
Output JSON using the format defined in docs/validators/cspec-validator.md."
```

---

## FFLR — Generation Session

**Session setup:**
```
Documents to provide:
1. Frozen RR (full document — feature flag states at release time)
2. Frozen RP (full document — flag-based exposure strategy)
3. Previous FFLR version (if updating an existing FFLR)
4. docs/specs/fflr-spec.md
5. docs/artifacts/fflr-template.md

Prompt:
"Generate a Feature Flag Lifecycle Record using the provided inputs.
Follow the prompt in docs/prompts/fflr-prompt.md.
Use the template exactly. Satisfy all hard gates in the spec.
Do not invent flags not documented in the RR or RP.
Mark any missing information with [MISSING: reason]. Output pure Markdown."
```

**After generation:** Review the FFLR. Confirm:
- All flags from the RR and RP are listed
- Each flag has a type classification (release/ops/experiment/permission)
- Current state per environment is documented
- Retirement criteria are defined for every flag
- Rollout history captures the initial creation event

**Validation session setup:**
```
Documents to provide:
1. The generated FFLR (full document)
2. docs/specs/fflr-spec.md

Prompt:
"Validate this Feature Flag Lifecycle Record against the FFLR spec.
Use only the spec as the source of truth for pass/fail criteria.
Do not suggest improvements. Judge only what is explicitly present.
Output JSON using the format defined in docs/validators/fflr-validator.md."
```

---

## FFLR — Re-Freeze Session (Living Document Update)

When updating the FFLR for a flag state change, retirement, or periodic review:

**Session setup:**
```
Documents to provide:
1. Current frozen FFLR (the version being updated)
2. Evidence of the change (flag state change record, retirement decision, RHR cycle date)
3. docs/specs/fflr-spec.md
4. docs/artifacts/fflr-template.md

Prompt:
"Update this Feature Flag Lifecycle Record to reflect the following change: {describe the change}.
Follow the prompt in docs/prompts/fflr-prompt.md.
Increment the version. Add the state change to rollout history.
Update the stale flag assessment. Output pure Markdown."
```

**After update:** Validate and re-freeze using the standard FFLR validation session setup.

---

## DSR — Generation Session

**Session setup:**
```
Documents to provide:
1. Frozen TDD (full document — data models and schema definitions)
2. Previous DSR version (if updating for schema evolution)
3. docs/specs/dsr-spec.md
4. docs/artifacts/dsr-template.md

Prompt:
"Generate a Data Schema Record using the provided inputs.
Follow the prompt in docs/prompts/dsr-prompt.md.
Use the template exactly. Satisfy all hard gates in the spec.
Do not invent schemas not in the TDD.
Mark any missing information with [MISSING: reason]. Output pure Markdown."
```

**After generation:** Review the DSR. Confirm:
- All schemas from the TDD are present with current version numbers
- Compatibility rules (backward/forward) are stated for each schema
- Migration plans include rollback procedures
- Validation rules are defined for schema fields
- Version history is populated

**Validation session setup:**
```
Documents to provide:
1. The generated DSR (full document)
2. docs/specs/dsr-spec.md

Prompt:
"Validate this Data Schema Record against the DSR spec.
Use only the spec as the source of truth for pass/fail criteria.
Do not suggest improvements. Judge only what is explicitly present.
Output JSON using the format defined in docs/validators/dsr-validator.md."
```

---

## Troubleshooting

**Validator returns FAIL on multiple gates**
Check that the generation session included all required inputs. Missing inputs are the most common cause of multi-gate failures.

**Configuration items are incomplete**
The TDD may not have documented all configuration requirements explicitly. Review the TDD for implicit config needs (database connections, API keys, feature toggles mentioned in the design) and ensure the CSPEC captures them.

**Feature flags are missing retirement criteria**
Every flag must have a retirement condition. If the RR or RP does not specify retirement plans, the FFLR should still define criteria based on flag type (e.g., release flags retire after full rollout; experiment flags retire after analysis completion).

**Schema compatibility rules are vague**
Compatibility rules must be explicit: "backward compatible" or "forward compatible" with specific version ranges. Do not accept "compatible" without direction specified.
