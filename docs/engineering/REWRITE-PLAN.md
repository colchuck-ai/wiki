# Engineering tree re-trace & rewrite — working plan

> **Status: CONFIRMED (2026-07-17). Approach: full clean-room.**
> This is a temporary working doc. Delete it when the rewrite completes
> (Phase 5). It is not part of the intent tree.

## Why this exists

On 2026-07-17 the **product** tree was rewritten (13 grill decisions;
PDR002–PDR004). O-numbers were preserved, but requirements changed wording and
verb, and new requirements/risks were added. The **engineering** tree
(architecture README, components C001–C011, ADRs, CRs) is currently *frozen*
and still traces to the OLD product structure. This plan rebuilds it against the
new product tree.

## Approach (confirmed)

- **Full clean-room.** Re-derive the whole engineering tree — architecture root,
  the component decomposition, and the architectural decisions — from the new
  product requirements. **The component set itself is open**: do not assume
  C001–C011 are the right components. C-numbers may change.
- **Old tree as coverage checklist, not template.** Draft the new architecture
  *independently* from the product tree, then diff it against the old
  engineering tree (components, ADRs) used purely as a checklist, to confirm no
  capability or hard-won decision was dropped. This is the same design-it-twice
  discipline used on the product rewrite — the second design must be allowed to
  diverge, not transcribe.
- **Archive first.** Snapshot the current engineering tree to
  `archive/docs-v2/engineering/` before starting.
- **Archive the CRs, start the log empty.** Fold settled CR content into the new
  components/ADRs; archive CR001–CR008 and CR010. The tree shrinks by having
  fewer decisions to record.
- **Chunkable.** Each ADR and each component spec is independently reviewable
  and committable, so work can pause/resume across sessions.

## Product changes that ripple down (the trace targets)

| Product change (2026-07-17) | Engineering impact |
|---|---|
| **Delegation envelope / manage-by-exception** (PDR003, O004-R004) | New cross-cutting concern: where the envelope lives, who evaluates it, how escalation surfaces. |
| **Decision-provenance + review tier** (PDR003, O002-R002, O009-R002) | New per-item / per-claim data: who vetted, how handled, envelope version. |
| **Charter required + foundational** (PDR004, O008-R001) | Charter is load-bearing for triage/significance/dedup/coverage and required to exist (bootstrap). |
| **Machine-readable-first / dual surface** (O005, O009-R002, O002-R001) | Grounding signal, review tier, recency, link reasons as structured fields; agent-queryable discovery. |
| **Citation resolves to durable form + live locator** (O001-R002/R003) | Two structured references. **Reverses CR001**, revisits ADR007. |
| **Held-aside / non-destructive significance** (O003-R004, O007-R003) | New recoverable, non-integrated state/store. |

## Phases

### Phase 0 — Baseline
- [ ] Archive the whole engineering tree → `archive/docs-v2/engineering/`
      (README, components, ADRs, CRs)
- [ ] Re-read the new product `README.md` + PDR002–004 as the trace target
- [ ] Extract from the old tree the **surviving invariants** that must carry
      over regardless of decomposition (e.g. OKF is external / not Wiki's to
      define; plain-text + git substrate; no shared mutable state; one owner of
      format knowledge). These are checklist items, not a template.

### Phase 1 — Architecture root + architectural decisions
- [ ] Draft the new engineering `README.md`: the system's shape derived from the
      new product tree (pipeline + standing capabilities, or whatever the
      clean-room derivation yields — do not presume the old shape).
- [ ] Author ADRs for the decisions the new product concepts force, plus
      re-affirm surviving old decisions (re-authored, not inherited):
  - [ ] **ADR — Delegation envelope**: where it lives, who evaluates it, how
        escalation surfaces to the steward.
  - [ ] **ADR — Decision-provenance & review-tier representation**: OKF
        conventions / frontmatter / log.
  - [ ] **ADR — Held-aside storage model**: recoverable non-integrated material;
        audit and reversal.
  - [ ] **ADR — Citation two-reference model**: durable resolution target + live
        locator. **Supersedes CR001**, replaces ADR007.
  - [ ] **ADR — Agent-queryable discovery surface** (O005 agent framing).
  - [ ] **ADR — OKF conformance boundary** (re-affirm/redecide the old ADR001
        stance under the new decomposition).
  - [ ] **ADR — Storage substrate** (plain-text + git, no shared mutable state —
        re-affirm).

### Phase 2 — Derive the component decomposition
- [ ] From the requirements alone, decide **what components the system needs** —
      each a deep module behind a small interface at a clean seam. Do not start
      from C001–C011.
- [ ] Diff the derived set against the old C001–C011 as a coverage checklist:
      every old capability must land somewhere in the new set or be consciously
      dropped (`log` what's dropped and why).
- [ ] Produce the Requirement→Component map covering every product requirement
      (O001–O009, all R's), with no orphans in either direction.

### Phase 3 — Component specs (dependency order)
- [ ] Spec each component in the derived set, innermost dependency first
      (format-knowledge owner and storage substrate before the pipeline before
      the standing capabilities).
- [ ] For each: capabilities, data model, behavior (envelope verbs where
      applicable), relationships, and success criteria aligned to the product
      **proxy metrics**.

### Phase 4 — CR cleanup
- [ ] Fold any still-relevant CR content into the new components/ADRs
- [ ] Archive CR001–CR008, CR010 → `archive/docs-v2/`; new CR log starts empty

### Phase 5 — Alignment check & close
- [ ] Run the `/intent` alignment/coherence check across product + engineering
- [ ] Verify every requirement → a component, and every component → a
      requirement (no orphans either direction)
- [ ] Confirm the surviving-invariants checklist from Phase 0 is all satisfied
- [ ] Delete this plan file
