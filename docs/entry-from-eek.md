# Entry from Engineering Execution Kit (EEK)

**You are here because:** The TDD is frozen and contains configuration requirements or data model definitions that need governance.

**What you must bring:**

| Artifact | Status Required | Where to Find It |
|----------|----------------|-----------------|
| Technical Design Document (TDD-{PROJECT}-{NNN}) — config requirements section | Frozen | EEK `docs/artifacts/` in the consuming project |
| Operational Readiness Document (ORD-{PROJECT}-{NNN}) — config readiness checks | Frozen | EEK `docs/artifacts/` in the consuming project |

**Which DCK artifact to generate depends on what the TDD contains:**

| TDD Content | DCK Artifact | First Action |
|-------------|-------------|--------------|
| Configuration items (env vars, feature toggles, connection strings) | CSPEC | Generate CSPEC from TDD config requirements |
| Data models, database schemas, API contracts | DSR | Generate DSR from TDD data model section |
| Both | Both CSPEC and DSR | Generate independently; no dependency between them |
| Neither | None | No DCK artifacts required for this initiative |

**Feature flags (FFLR)** are not triggered by the EEK handoff. FFLRs are triggered when feature flags are created during REK. If you know flags will be used, note this for the REK phase but do not create the FFLR yet.

**Where to start:** `docs/playbook.md` — find the section for the artifact type you need (CSPEC or DSR)

**What changes at this boundary:**

- You shift from defining what to build (EEK) to governing how it is configured and how its data structures evolve.
- The TDD defines the system's needs; the DCK artifacts define the management rules around those needs.
- Configuration validation rules from the CSPEC will feed back into REK as release verification criteria.

**Common mistakes at this transition:**

| Mistake | How to Avoid |
|---------|--------------|
| Generating a CSPEC before the TDD is frozen | The TDD defines the configuration requirements. Wait for freeze. |
| Listing configuration items not in the TDD | The CSPEC must not expand scope beyond what the TDD defines. |
| Creating a DSR for trivial data structures | DSRs govern schemas that evolve over time. Ephemeral or trivial structures may not need formal schema governance. |
| Combining CSPEC and DSR into one artifact | They are separate artifact types with separate specs, templates, and validators. Generate independently. |

**If you arrived here without a complete upstream artifact:**

Stop. Return to EEK, complete the TDD configuration requirements section, freeze, and then re-enter DCK. DCK cannot define configuration governance without knowing what the system requires. A pointer to an unfrozen or incomplete TDD does not satisfy the entry condition.

---

*For the full entry flow, see `docs/playbook.md`.*
