# ADR004 — Placement of per-item machine-readable signals

## Status
Accepted (2026-07-17). Engineering clean-room rewrite, Chunk A.

## Context
*Machine-readable-first* (a product operating principle) requires that every
human-legible signal be a machine-readable field first: per-claim grounding signal and
review tier (O009-R002), per-concept recency (O002-R001), deprecation/supersession
markers (O003-R001), reason-annotated cross-links (O005-R002), and a dated change
history carrying decision-provenance (O002-R002). These live at three different
granularities (per-claim, per-concept, per-change) and have different writers and change
cadences. Two constraints pull on where they physically live: the corpus must stay
**portable** (O006 — plain-text, version-controlled, understandable with no tooling), and
[C008 must be the sole writer](ADR001-integration-sole-corpus-writer.md) of concept files.

## Options
- **All signals in a central sidecar/database:** clean write path, but a tooling-less
  reader gets nothing off the files — a direct O006 portability regression (this is what
  a naive reading of "one provenance log" would produce).
- **All signals inline in the concept file:** maximal portability, but drift status and
  cross-concept history would force a concept-file write every time a *source* changes —
  churn, and it fights the sole-writer/atomicity model.
- **Split by what the signal is a property of** (chosen).

## Decision
Signals that are a property of the concept a consumer reads live **inline in the concept
file**, written by C008 (the sole writer) at mutation time: per-claim grounding signal +
review tier, per-concept recency stamp, deprecation/supersession markers, and
reason-annotated cross-links. A consumer (human or agent) opening the file sees them with
no tooling.

The **change history / decision-provenance log** is owned by
[C005 Currency & Provenance](../components/C005-currency-and-provenance.md) as an
**append-only, plain-text, in-corpus, scoped** artifact — scoped (e.g. per-directory) so
a slice of the corpus carries its own history rather than depending on a central store.
This preserves O006 portability for history too.

**Drift status** is materialized by C005 (computed by C004), *not* stored inline —
because it changes when the source changes, independent of the concept, and inlining it
would force spurious concept-file writes.

## Consequences
- The corpus is portable at every layer: current state inline, history co-located,
  both plain-text and version-controlled (O006 holds end-to-end).
- The sole-writer model is preserved: C008 writes inline fields; C005 owns the log;
  neither writes the other's store.
- Recency is stamped inline by C008 (it knows which changes are *meaningful*) and also
  derivable from the log — a deliberate, cheap redundancy serving inline legibility.
- **Reversal:** costly — field placement is baked into the OKF shape and every
  consumer's parse. Hence recorded.
