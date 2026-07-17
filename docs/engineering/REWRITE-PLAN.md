# Engineering tree re-trace & rewrite — working plan

> **Status: CONFIRMED (2026-07-17). Approach: full clean-room.**
> This is a temporary working doc. Delete it when the rewrite completes
> (Phase 5). It is not part of the intent tree.
>
> **Chunk A COMPLETE (2026-07-17): Phases 0–2 done.** Outputs committed:
> `README.md` (architecture root), `DECOMPOSITION.md` (12 components + coverage
> diff + conscious drops), `INTERFACES.md` (frozen contracts), `REQUIREMENT-MAP.md`
> (full trace, no orphans), `drs/ADR001–005`, and `components/C001–C012` stubs
> (frozen interface + Phase-3 seed). Old tree archived to
> `archive/docs-v2/engineering/`. **Next: Chunks B…N = Phase 3** — fan out one
> subagent per component to write its behavioral spec from the frozen Chunk-A
> outputs.

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
- **Chunk boundaries (smart-zone-aware).**
  - **Chunk A — one unbroken session:** Phases 0–2 (architecture root + all ADRs
    + decomposition + inter-component interfaces + Requirement→Component map).
    Tightly interdependent design work — do **not** compact mid-way; `/handoff`
    to a fresh session rather than push on degraded. Ends committed.
  - **Chunks B…N — one per component, fresh context, parallel:** Phase 3 specs,
    fanned out to subagents, each reading the frozen Chunk-A outputs.
  - **Chunk final:** Phases 4–5.

## Product changes that ripple down (the trace targets)

| Product change (2026-07-17) | Engineering impact |
|---|---|
| **Delegation envelope / manage-by-exception** (PDR003, O004-R004) | New cross-cutting concern: where the envelope lives, who evaluates it, how escalation surfaces. |
| **Decision-provenance + review tier** (PDR003, O002-R002, O009-R002) | New per-item / per-claim data: who vetted, how handled, envelope version. |
| **Charter required + foundational** (PDR004, O008-R001) | Charter is load-bearing for triage/significance/dedup/coverage and required to exist (bootstrap). |
| **Machine-readable-first / dual surface** (O005, O009-R002, O002-R001) | Grounding signal, review tier, recency, link reasons as structured fields; agent-queryable discovery. |
| **Question-based retrieval** (PDR005, O005-R004) | Retrieval-by-question needs a component owner; mechanism is open (agent-over-corpus vs. built search). Lands under the agent-query ADR. |
| **Citation resolves to durable form + live locator** (O001-R002/R003) | Two structured references. **Reverses CR001**, revisits ADR007. |
| **Held-aside / non-destructive significance** (O003-R004, O007-R003) | New recoverable, non-integrated state/store. |

## Phases

### Phase 0 — Baseline
- [x] Archive the whole engineering tree → `archive/docs-v2/engineering/`
      (README, components, ADRs, CRs)
- [x] Re-read the new product `README.md` + PDR001–005 as the trace target
- [x] Separate two things pulled from the old tree — do **not** conflate them:
  - **True invariants** (product-derived; must survive any decomposition): OKF is
    external / not Wiki's to define; plain-text + git substrate; and the product
    requirements themselves. Fixed constraints on the derivation.
  - **Prior design choices** (old decisions to RE-AFFIRM or REPLACE in Phase 1,
    never inherited as givens): e.g. one owner of format knowledge (ADR001/C004),
    no shared mutable state, the pipeline + standing-capabilities shape. The
    clean-room must *earn* these from the requirements, not assume them.

### Phase 1 — Architecture root + architectural decisions
- [x] Draft the new engineering `README.md`: the system's shape derived from the
      new product tree (pipeline + standing capabilities, or whatever the
      clean-room derivation yields — do not presume the old shape).
- [x] Author an ADR **only** when a decision clears the bar: hard-to-reverse
      **and** surprising-without-context **and** a real trade-off. No migrating
      old ADRs. Decisions the new product concepts are expected to force (write
      each only if it clears the bar):
  - [x] **Delegation envelope**: where it lives, who evaluates it, how escalation
        surfaces to the steward.
  - [x] **Decision-provenance & review-tier representation**: OKF conventions /
        frontmatter / log.
  - [x] **Held-aside storage model**: recoverable non-integrated material; audit
        and reversal.
  - [x] **Citation two-reference model**: durable resolution target + live
        locator. **Supersedes CR001**, replaces ADR007.
  - [x] **Agent-queryable discovery / question-based-retrieval owner**
        (O005-R004; mechanism open).
  - [x] *(only if the derivation actually re-opens them)* OKF conformance
        boundary; storage substrate.
- [x] **End-of-Phase-1 checklist (not a work list):** walk old ADR001–ADR015 —
      for each, did the clean-room face this decision? If yes, is it settled (in
      an ADR or a component)? If we didn't face it, why not? Catches any real
      decision that silently vanished.

### Phase 2 — Derive the component decomposition (blind-first, via `/codebase-design`)
**Do not reopen the old component docs until step 5** — anchoring is the main
failure mode, and blind-first ordering is what makes "component set is open" real.
- [x] 1. List every capability the requirements demand (from the frozen product
      tree only).
- [x] 2. Group capabilities by the **state/artifact each owns** (locality) — what
      data does this cluster exclusively read and write?
- [x] 3. Cut **seams** where a small interface hides deep behavior (depth); aim
      for one owner per artifact, so "no shared mutable state" either falls out
      as absent or is flagged as a finding — not assumed.
- [x] 4. Cross-check completeness against the material **lifecycle** (intake →
      triage → author → maintain → discover) — no stage orphaned.
- [x] 5. *Now* diff the derived set against old C001–C011 as a coverage
      checklist: every old capability lands somewhere new or is consciously
      dropped (`log` what's dropped and why).
- [x] 6. Produce the Requirement→Component map covering every product requirement
      (O001–O009, all R's incl. O005-R004), with no orphans in either direction.
- [x] 7. **Freeze each component's interface** (the small surface others call) —
      this contract is what lets Phase 3 specs run in parallel without colliding.
      Chunk A is not done until interfaces are frozen and committed.

### Phase 3 — Component specs (parallel fan-out)
Enabled by Chunk A freezing the interfaces. One subagent per component; each is
given the frozen decomposition, its component's interface, and the requirements
mapped to it.
- [ ] Fan out one subagent per component in the derived set (independent, since
      interfaces are frozen).
- [ ] Each spec covers: capabilities, data model, behavior (envelope verbs where
      applicable), relationships, and success criteria aligned to the product
      **proxy metrics**.
- [ ] **Reconciliation pass** after fan-out: verify the specs agree at every
      shared interface (a subagent may interpret a contract slightly
      differently) and that no component reached outside its frozen interface.

### Phase 4 — CR cleanup
- [ ] Fold any still-relevant CR content into the new components/ADRs
- [ ] Archive CR001–CR008, CR010 → `archive/docs-v2/`; new CR log starts empty

### Phase 5 — Definition of done & close
"Done" means all six exit criteria hold:
- [ ] 1. **Traceability** — every requirement → a component and every component →
      a requirement, no orphans either direction (run the `/intent` alignment check).
- [ ] 2. **Invariants** — the Phase 0 true-invariants all hold.
- [ ] 3. **Coverage** — every old capability lands somewhere new or is
      consciously logged as dropped (from Phase 2 step 5).
- [ ] 4. **Deep-module bar** (`/codebase-design`) — each component is deep (real
      behavior behind a small interface); no shallow pass-through components; one
      owner per artifact.
- [ ] 5. **Anti-churn / readability** — the whole engineering tree reads in "one
      coffee"; the CR log starts empty; no ADR present that didn't clear the bar.
- [ ] 6. **Independent coherence read** — a fresh perspective (a subagent, or the
      `/intent` judge) reviews the assembled tree; not author-graded.
- [ ] Delete this plan file.
