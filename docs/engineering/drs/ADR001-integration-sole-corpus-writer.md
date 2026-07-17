# ADR001 — Integration is the sole corpus writer

## Status
Accepted (2026-07-17). Engineering clean-room rewrite, Chunk A.

## Context
The concept corpus is the system's one mutable artifact, and many capabilities want
to change it: triage resolves a merge, lifecycle deprecates and relocates, coverage &
scope flag divergence, authoring integrates. The product also demands that all of this
be OKF-conformant (O006-R002), provenance-stamped (O002-R002, PDR003), and reversible
(deprecation over deletion). If each capability edited concept files directly, every
one would need to embed OKF knowledge, agree on field placement, and stamp provenance —
and two capabilities could touch the same concept concurrently. That is textbook shared
mutable state.

## Options
- **Each capability edits concept files** (the obvious decomposition): simple call
  graphs, but format knowledge and provenance logic spread across N writers, and the
  corpus has no single consistency owner.
- **Split writers by field-region** (claims vs metadata): fewer writers, but two owners
  of one file still race, and the region boundary leaks into every caller.
- **One writer, everyone else expresses intent** (chosen): a single Integration
  component exposes intent verbs; all others *drive* it.

## Decision
[C008 Integration](../components/C008-integration.md) is the **only** component that
mutates concept files. It exposes an intent-verb surface — `integrate`, `merge`,
`deprecate`, `supersede`, `relocate`, `repair_links`, `flag` — and owns the edit behind
each. Every other component that wants the corpus changed expresses intent through these
verbs (this is the engineering form of intent-driven authoring / assisted-upkeep-by-intent,
O004-R001/R003).

Two guarantees ride on this:
- **Per-verb atomicity.** Each verb commits the concept-file change and its
  `C005.record` provenance entry together — one commit or none. (This absorbs the old
  merge-atomicity concern, now intra-component rather than a cross-component composition.)
- **Corpus as single source of truth.** Derived views — the index/graph (C009),
  currency (C005), drift status (C004) — are *functions of the corpus* and recompute
  from it. C008 does not orchestrate them, so there is no multi-store write to keep
  consistent.

## Consequences
- **No shared mutable state on the corpus.** The central invariant of the architecture
  falls out of this decision rather than being asserted.
- OKF-conformant editing, citation, grounding, and provenance stamping live in **one**
  place; the grounding check is an internal seam of C008 (settles old ADR015).
- C008 is heavily depended upon — that is acceptable depth (leverage), not a smell; the
  dependency arrows into it are *intent* calls, not writes.
- Callers become pure decision modules (C007, C011, C012 own no corpus state), which
  makes each testable through a single call.
- **Reversal:** costly — every capability's contract assumes intent-not-edit. This is
  why it is recorded here.
