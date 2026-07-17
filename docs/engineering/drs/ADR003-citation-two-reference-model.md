# ADR003 — Citation two-reference model (durable target + live locator)

## Status
Accepted (2026-07-17). Engineering clean-room rewrite, Chunk A. **Supersedes old CR001
and old ADR007** (which cited the live origin with an archived fallback and no persisted
baseline).

## Context
The product now requires (O001-R002) that a citation *resolve* to a **durable,
version-controlled form** — capturing a snapshot when the source cannot be referenced
durably in place — *and* that, where a live external locator exists, it be carried as a
**distinct** structured reference for drift detection, "never as a substitute for the
durable resolution target." Fidelity (O009) reads against that same durable form
(PDR001): one substrate, two outcomes. This reverses the earlier stance where the live
origin was the primary reference. Two sub-questions follow: what are the two references,
and *when* is the durable snapshot captured.

## Options
*Reference model:* (a) live locator with archived fallback (old ADR007) — rots and
conflates "where to check for change" with "what to check against"; (b) durable
snapshot only — loses drift detection; (c) **both, distinct** (chosen).

*Capture timing:* (a) at intake, for every submission — no rot window, but snapshots
material that may be discarded or sit in held-aside indefinitely; (b) **at authoring,
on first citation** (chosen) — snapshots only what is actually cited, and the snapshot
is exactly the form the claim is grounded against.

## Decision
A citation is `{ durable_ref, live_locator? }`. The `durable_ref` is the
version-controlled resolution target — the only thing grounding (O009) and drift
detection check *against*. The `live_locator`, when a live origin exists, is a distinct
reference used *only* to detect drift; it is never substituted for `durable_ref`.

The durable snapshot is captured at **authoring time**, when a claim first cites the
source (`C008 → C004.retain`). Rationale: O001-R002/O009 concern *cited* claims, and the
claim is faithful to the form cited at authoring — so the snapshot is precisely that
form. This also keeps [C004 Source Retention](../components/C004-source-retention.md)
entirely corpus-side (authoring + post-authoring drift); intake (C006) never calls it,
avoiding the intake↔retention reach-back the old design worked to prevent.

## Consequences
- Traceability (O001) and fidelity (O009) share **one** durable-source substrate; a
  change to how sources are held durably affects both — they must not fork it (PDR001).
- Drift detection (O001-R003) is a clean comparison of `live_locator` against
  `durable_ref`, computed in C004 and materialized as currency in C005.
- **Known limitation:** a source that rots between intake and authoring is captured only
  in its authoring-time state. Accepted — fidelity is defined relative to what was cited,
  and capturing every intake would snapshot discarded material.
