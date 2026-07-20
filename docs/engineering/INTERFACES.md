# Frozen inter-component interfaces

> **Status: frozen at end of Chunk A (Phase 2 step 7).** These are the *surfaces
> other components call* — the contracts that let the Phase 3 component specs be
> written in parallel without colliding. Internal behavior is deliberately absent;
> depth lives behind these surfaces. Changing a signature here is a cross-component
> event, not a local edit.

"Interface" here means everything a caller must know: the call, its invariants,
ordering constraints, and error/edge modes — not just the type signature.

Types in `CamelCase` are shared vocabulary defined in
[DECOMPOSITION.md](DECOMPOSITION.md#shared-types); verbs are `snake_case`.

---

## C001 Charter — the scope referent

| Call | Contract |
|---|---|
| `read() -> Charter` | Current charter (purpose + scope areas). **Always succeeds** — the charter is required, so it always exists; it MAY be provisional/incomplete. |
| `scope_areas() -> [ScopeArea]` | The declared scope areas coverage/scope surveys enumerate. |
| `declare(Charter)` / `revise(CharterPatch)` | Steward-facing. Writes the charter in-corpus, revisable-in-place. Reports provenance to `C005.record`. |

**Invariants.** Required (always readable). Consumer-legible. Read by C002, C007,
C008, C011, C012. C001 depends on **nothing** below it.

---

## C002 Governance — the envelope + the one attention surface

| Call | Contract |
|---|---|
| `evaluate(ProposedAction) -> Decision` | `Decision = act \| escalate(reason)`. Pure decision against the envelope; reads `C001`. **No side effects.** Absent envelope → the conservative default (escalate the consequential; act on the mechanical) — see C002 spec. |
| `escalate(item, reason, evidence) -> AttentionId` | Post to the single steward attention surface. Used for envelope escalations, review candidates (C011/C012), and distillation reviews (C008, `evidence` = source span). |
| `resolve(AttentionId) -> StewardDecision` | The steward's adjudication for an escalated item. Callers block on / poll this to complete a held flow. |
| `envelope_version() -> Version` | Stamped into every disposition (provenance needs it). |
| `list_attention() -> [AttentionItem]` | The steward's single inbox — **composed, not a firehose**: items are deduplicated/rolled up by subject so N signals about one concept surface as one item (keeps steward effort flat, O004). |

**Invariants.** Envelope optional. `evaluate` is the *only* place act-vs-escalate is
decided — no component embeds envelope logic. Every escalation surfaces here, nowhere
else. The attention surface **composes** (subject rollup/dedup) rather than relaying raw
signals. Reads C001; called by every autonomous capability (C007, C008, C011, C012).
*Internal seams (for Phase 3): the envelope evaluator and the composed attention surface
are two internal seams behind this one interface.*

---

## C003 OKF Conformance — single owner of format knowledge

| Call | Contract |
|---|---|
| `apply(content, kind) -> ConformantContent` | Normalize to the external format's structural + naming conventions. Called by **C008 only** (the sole writer). |
| `validate(target) -> ConformanceReport` | `ok \| [violation]` against the **pinned** spec. Called by C008 (pre-write) and by C009/C012 (audit). |
| `spec_version() -> Version` | The pinned external-format version. |

**Invariants.** The **only** component that embeds format rules. Wiki *consumes* the
format; it does not define it. A format version bump is a one-component change.
Stateless (owns no persistent artifact) — deep by behavior, not by data.

---

## C004 Source Retention — the durable/live source pair

| Call | Contract |
|---|---|
| `retain(source) -> Citation` | `Citation = { durable_ref, live_locator? }`. Captures a version-controlled durable snapshot and records the distinct live-origin locator when one exists. **Idempotent** per source. Captured at **authoring time** (first citation), called by C008 — *not* at intake (ADR003), so C004 is entirely corpus-side and C006 never calls it. |
| `resolve(durable_ref) -> SourceContent` | Fetch the durable snapshot — what grounding (C008) and drift check against. Never returns the live origin. |
| `check_drift(Citation) -> DriftStatus` | `unchanged \| drifted \| unreachable`. Compares `live_locator` against `durable_ref`. Called by the currency refresh (C005); its result reaches C011 through `C005.currency`, not by C011 calling C004 directly. |

**Invariants.** `durable_ref` is the resolution target and is **never** substituted by
`live_locator` (ADR003). One substrate serves both provenance (O001) and fidelity
(O009) — callers must not build a second source store.

---

## C005 Currency & Provenance — the append-only history

| Call | Contract |
|---|---|
| `record(Disposition)` | **Append-only.** `Disposition = { target, outcome, actor, envelope_version, time, detail }`. The **single write path** to the provenance log. Called by C008 on every mutation and by C007/C011/C012/C001/C002 on any recordable action. |
| `history(target) -> [ProvenanceEntry]` | Dated change history incl. decision-provenance (O002-R002). |
| `recency(concept) -> Recency` | Last meaningful change (O002-R001), derived from the log + C008's in-file stamp. |
| `currency(concept) -> CurrencyStatus` | `{ recency, drift_status }` — the O002 proxy picture. `drift_status` materialized from `C004.check_drift`. |

**Invariants.** Single writer of the provenance store; callers **report**, never write
it directly (no shared mutable state). The log is append-only, and **plain-text,
in-corpus, and scoped so history travels with the material** (a corpus slice carries its
own history — not a central database) — preserving O006 portability (ADR004). Drift is
*computed* by C004 and *materialized* here (C005 does not itself diff sources).

---

## C006 Intake & Held-Aside — non-integrated material

| Call | Contract |
|---|---|
| `submit(RawMaterial) -> IntakeId` | Producer-agnostic single entry point; accepts material the moment it arrives (O007-R001). Never rejects at the door. |
| `next() -> RawMaterial?` | Hand the next untriaged item to C007. |
| `hold_aside(item, reason) -> HeldId` | Move item to the recoverable held-aside store (below-significance / ambiguous-scope). Called by C007. Reports provenance to C005. |
| `list_held_aside() -> [HeldItem]` | For coverage review (part of the O007 proxy). Read by C012. |
| `restore(HeldId) -> RawMaterial` | Return a held item to triage — the reversal path (deprecation-over-deletion). |

**Invariants.** Owns all non-integrated material (raw queue + held-aside). Nothing is
destroyed here; purge is a separate explicit act. Held-aside is recoverable and audited.

---

## C007 Triage — the disposition decider

| Call | Contract |
|---|---|
| `triage(RawMaterial) -> TriageOutcome` | `integrate \| hold_aside \| merge(into: ConceptId) \| escalate`. Evaluates scope (vs `C001`), duplication (vs `C009` index), and significance (vs `C001`). Within-envelope calls go through `C002.evaluate`; ambiguous ones escalate. (`TriageOutcome` is distinct from the provenance `Disposition` of C005.) |

**Invariants.** Owns **no persistent store** — a pure decision module (charter + item +
corpus → disposition), which makes it testable through this one call. Executes nothing
itself: routes via `C006.hold_aside`, `C008.integrate`/`merge`, `C002.escalate`.

---

## C008 Integration — the SOLE corpus writer (intent-verb surface)

Every corpus mutation in the system goes through exactly one of these verbs. Each is
executed OKF-conformantly (`C003.apply`/`validate`), cites via `C004.retain`, and
reports to `C005.record`.

| Call | Contract |
|---|---|
| `integrate(RawMaterial, intent) -> ConceptId` | Distill into a cited, grounded, cross-linked concept. Runs the author-time grounding check (internal seam) against `C004.resolve`; stamps per-claim grounding signal + review tier; within-envelope admit/hold via `C002.evaluate`, else escalate with source span via `C002.escalate`. |
| `merge(sources, into) -> ConceptId` | Compose overlapping material into one concept; prune redundant citations; preserve provenance. |
| `deprecate(ConceptId, reason, successor?)` | Mark deprecated/superseded in place (never delete). |
| `relocate(ConceptId, new_location)` | Move a concept to a new canonical path (OKF naming). Triggers `repair_links` so inbound links follow — the relocate half of referential integrity (O005-R003/RSK002). |
| `repair_links(ConceptId, successor)` | Retarget inbound links to a successor (drives referential integrity; uses `C009.inbound_links`). |
| `flag(ConceptId, kind, reason)` | Attach a scope-divergence / retirement-candidate marker (machine-readable). |

**Invariants.** The **only** writer of concept files — no other component edits the
corpus. Callers express *intent*; C008 owns the edit. Writes these in-concept fields:
claims, citations (`durable_ref`+`live_locator`), per-claim grounding + review tier,
reason-annotated cross-links, deprecation markers, recency stamp. **Every verb is
atomic: the concept-file change and its `C005.record` provenance entry commit together,
one-commit-or-none.** Derived views (C009 index/graph, C005 currency) recompute from the
corpus as the single source of truth — C008 does not orchestrate them (ADR001).

---

## C009 Discovery & Index — deterministic navigational structure

| Call | Contract |
|---|---|
| `index() -> Index` | The navigable listing over all concepts — machine-readable structure with a human rendering (O005-R001). |
| `neighbors(ConceptId) -> [Link]` | Reason-annotated cross-links for traversal (O005-R002). |
| `inbound_links(ConceptId) -> [Link]` | Who points *at* this concept — for referential integrity before retire/relocate (O005-R003). Read by C011. |
| `dangling() -> [Link]` | Links resolving to nothing (an integrity signal). |

**Invariants.** Read-only over the corpus — it *materializes* structure, it does not
mutate concepts. Machine-readable-first, rendering second. Deterministic (contrast C010).

---

## C010 Question Retrieval — heuristic free-form lookup

| Call | Contract |
|---|---|
| `retrieve(question) -> [RankedConcept]` | Map a question posed in the consumer's own words → the relevant concepts (O005-R004). **Heuristic** — may miss; ranked, not guaranteed. |

**Invariants.** Distinct from C009 because it is heuristic and its **mechanism is open**
(agent-over-corpus vs a built search index — ADR005). Read-only over the corpus.

---

## C011 Lifecycle & Retirement — the mutating maintainer

| Call | Contract |
|---|---|
| `review() -> [RetirementCandidate]` | Periodic + on-demand. Surfaces candidates from `C005.currency` (staleness/drift) and in-corpus supersession markers. |
| (drives) | For each candidate within the envelope (`C002.evaluate`): `C008.deprecate` / `C008.relocate` / `C008.repair_links` (after `C009.inbound_links`). Ambiguous → `C002.escalate`. Owns the referential-integrity-sensitive moves (both **retire and relocate**, O005-R003). |

**Invariants.** Mutates the corpus **only** by driving C008 intent verbs — never edits
files directly. Owns the retirement *decision*, not the write.

---

## C012 Coverage & Scope Review — the read-only surveyor

| Call | Contract |
|---|---|
| `coverage() -> CoverageReport` | Charter scope areas (`C001`) vs `C009.index` + `C006.list_held_aside` + `C009.dangling` (link targets that were authored-as-intended but are absent = known gaps) → thin/absent areas (O007-R002). Mechanism (metric vs agent-scan of the corpus) is open, like C010. |
| `scope() -> [ScopeDivergence]` | Charter (`C001`) vs corpus → drifted-out-of-scope content (O008-R003). |
| (drives) | Escalates via `C002.escalate`; scope flags executed via `C008.flag`. |

**Invariants.** Read-only survey — produces reports and drives C008/C002; never writes
the corpus itself.

---

## Dependency direction (no cycles)

```
C001 Charter  ◀── C002, C007, C008, C011, C012        (foundation; depends on nothing)
C002 Governance ◀── C007, C008, C011, C012            (reads C001)
C003 OKF Conformance ◀── C008, C009, C012             (stateless)
C004 Source Retention ◀── C008, C005                  (stateless-ish source store)
C005 Currency&Provenance ◀── all mutators             (reads C004)
C006 Intake&Held-Aside ◀── C007, C012
C007 Triage → C001, C002, C006, C008, C009
C008 Integration → C002, C003, C004, C005, C009        (SOLE corpus writer)
C009 Discovery&Index → (reads corpus)                  (read-only)
C010 Question Retrieval → (reads corpus)               (read-only)
C011 Lifecycle → C002, C005, C008, C009
C012 Coverage&Scope → C001, C002, C006, C008, C009
```

The corpus is written by C008 and read by C009/C010; every arrow into C008 is an
*intent* call, not a shared write. No component reaches past these surfaces.
