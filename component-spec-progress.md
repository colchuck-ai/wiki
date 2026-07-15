# Component Spec Progress

Tracks promotion of the 10 architecture components (C001–C010) from Tier 0 (inline
on `docs/engineering/README.md`) to Tier 1 own-document specs, using the `/intent`
skill. Work proceeds in **dependency order** (foundation → leaves) so every spec can
reference a settled contract rather than an assumed one.

**How to resume (new session):** invoke the `/intent` skill, trace context for the
target component (`references/context-tracing.md` → Component), then run the
Draft → Judge workflow (`references/workflows.md`, solo repo = Draft + Judge only).
Judge = `python3 scripts/lint_intent_tree.py docs/` from the skill root, then the
binary checklist. Update the Status column here when a component's spec lands.

## Dependency graph

"Depends on" = needs the other's *contract* (format, data, or signal). Components
read/write concept files directly on the git substrate — no shared mutable state.

```
Layer 0 (root):  C004
Layer 1:         C001, C010, C005           (depend only on C004)
Layer 2:         C002, C003, C006, C007      (depend on layer 1)
Layer 3:         C008, C009                  (depend on layer 2 + layer 1)
```

## Status

| # | Component | Layer | Depends on | Requirements fulfilled | Status |
|---|---|---|---|---|---|
| C004 | OKF Conformance | 0 | — (vendored OKF spec) | O004-R002, O006-R001, O006-R002 | ✅ Done (+ ADR001) |
| C005 | Index & Navigation | 1 | C004 | O005-R001, O005-R002*, O005-R003 | ⬜ Not started |
| C001 | Ingestion Queue | 1 | C004 | O001-R002, O007-R001 | ⬜ Not started |
| C010 | Charter | 1 | C004 | O008-R001, O008-R003 | ⬜ Not started |
| C002 | Triage | 2 | C004, C010, C005, C001 | O003-R003, O003-R004, O008-R002 | ⬜ Not started |
| C003 | Integration Authoring | 2 | C004, C005, C001, C002 | O001-R001, O004-R001, O005-R002* | ⬜ Not started |
| C006 | Source Revalidation | 2 | C004, C001 | O001-R003 | ⬜ Not started |
| C007 | Currency Tracking | 2 | C004, C003 | O002-R001, O002-R002 | ⬜ Not started |
| C008 | Lifecycle & Retirement | 3 | C004, C005, C006, C007 | O003-R001, O003-R002 | ⬜ Not started |
| C009 | Coverage Review | 3 | C004, C005, C010 | O007-R002 | ⬜ Not started |

\* O005-R002 (Reasoned cross-links) is shared by C003 and C005 per the
Requirement-Component Map.

**Recommended next:** C005 — the most-depended-on component after C004
(C002, C008, C009 all consume its graph / inbound-link contract).

## Notes / open judgments carried forward

- **C004 role**: library + validator (not a write-proxy or store). [decided]
- **C004 validator**: two tiers — OKF §9 errors + Wiki structural-*shape* advisories;
  never semantic content. [decided]
- **Kebab-case slug convention** recorded inline in C004 Notes, not its own ADR —
  promote to ADR if contested.
- **O006-R001** scoped to C004 *validating* the plain-text floor; *storage* is the
  git substrate (O006-R003, no dedicated component).
- Records so far: **ADR001** (single OKF conformance boundary). Add CRs only for
  material changes to existing elements; promotion/elaboration of a Tier 0 component
  into its Tier 1 spec is not itself a CR-worthy change.
