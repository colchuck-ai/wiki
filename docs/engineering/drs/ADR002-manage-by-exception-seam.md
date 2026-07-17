# ADR002 — Manage-by-exception as a single governance seam

## Status
Accepted (2026-07-17). Engineering clean-room rewrite, Chunk A. Realizes PDR003.

## Context
The product mandates *manage by exception* (PDR003, O004-R004): Wiki acts autonomously
within a steward-declared delegation envelope and escalates only genuine exceptions. A
synchronous per-item human gate is explicitly forbidden — it relocates the bottleneck
O004 exists to remove. This act-vs-escalate decision is needed by *every* autonomous
admission and upkeep path: triage (O008-R002, O003-R003/R004), authoring (O009-R001),
retirement (O003-R002), coverage (O007-R002), scope reconciliation (O008-R003). And
every escalation must reach the steward without drowning them — or the "flat effort"
promise of O004 breaks on the writer side.

## Options
- **Each capability embeds envelope logic and its own escalation channel:** no new
  component, but "how we decide act-vs-escalate" and "where escalations go" are
  re-implemented (and drift) across six capabilities, and the steward faces N queues.
- **Fold envelope evaluation into the Charter owner** (both are policy anchors): but the
  envelope *reads* the charter (asymmetric), has a different audience and cadence, and
  is optional where the charter is required — merging them buries that asymmetry.
- **One governance seam** (chosen): a single component owns envelope evaluation and the
  one steward attention surface.

## Decision
[C002 Governance](../components/C002-governance.md) is the single seam for autonomy.
It exposes `evaluate(action) -> act | escalate` — **the only place** act-vs-escalate is
decided — and owns the **one** steward attention surface (`escalate` / `list_attention`
/ `resolve`). Every autonomous capability funnels its consequential actions through
`evaluate`, and every escalation, review candidate, and distillation review surfaces
here and nowhere else. It exposes `envelope_version()` so every disposition can be
stamped for provenance (PDR003).

The attention surface is **composed, not a firehose**: items are deduplicated/rolled up
by subject, so N signals about one concept surface as one item. This keeps steward
effort flat (O004) — the writer-side mirror of the manage-by-exception promise.

The envelope is optional; absent, C002 applies a conservative default (escalate the
consequential, act on the mechanical). Charter and envelope stay **separate**
components (C001 vs C002) — the envelope reads the charter, not the reverse.

## Consequences
- Autonomy is legible and reversible at one seam: one policy evaluator, one audit point,
  one steward inbox — the operational core of PDR003.
- The composed surface absorbs the *product-motivated* part of the old cross-face
  aggregator (dedup/rollup). The elaborate conflict-pairing/routing of old ADR016 is
  **not** imported as architecture; it may return as a C002 Phase-3 refinement if
  steward experience demands (logged in [DECOMPOSITION.md](../DECOMPOSITION.md#consciously-dropped)).
- C002 carries two internal seams (evaluator, attention surface); if the surface grows
  its own weight, splitting it into a dedicated aggregator is the pre-identified path.
