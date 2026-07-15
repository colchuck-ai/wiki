# ADR012 - Coarse Source Revalidation

Re-scope C006 - Source Revalidation from *re-fetch → normalize-as-at-capture → line-diff* to a **coarse change signal**: compare the live re-fetch against the captured snapshot at whatever fidelity exists (change / no-change) and, on a suspected change, present the **captured snapshot and the live source together** for the steward to judge. Drop the capture-normalization step and the text-vs-binary diff split.

## Context

C006 (per [ADR010](ADR010-curation-agent-skill.md) and CR008) specified re-fetching the cited source, normalizing it "the same way capture did at intake," then diffing line-by-line against the captured snapshot. Two problems undercut this. Intake is **producer-agnostic** (C001 - Inbox): any human or agent deposits already-captured material, so there is no single Wiki-owned capture transform to reproduce — "the same way capture did" has no referent when producers are heterogeneous, and two producers can normalize the same source differently. And an automated line-diff is more precision than the **low-volume single-steward envelope** ([ADR009](ADR009-low-volume-operating-envelope.md)) needs, since the steward reviews any flagged drift anyway. The design already carried two diff models: [ADR008 - Captured Source Assets](ADR008-captured-source-assets.md) accepts a coarse change/no-change diff for binary assets.

## Options

- **Precise diff with matched normalization** (current, rejected): requires a deterministic capture transform the producer-agnostic Inbox does not own or guarantee.
- **Record the normalization method per item** (a `normalized-by` field): reproducible, but adds per-item metadata and machinery for a signal the steward inspects anyway — complexity the envelope does not earn.
- **Coarse change signal plus present-both** *(chosen)*: detect change coarsely; on a flag, present the captured snapshot and the live source for steward judgment. One model for text and binary.

## Decision

Adopt the **coarse change signal**. C006 re-fetches the cited source and compares it to the captured snapshot at available fidelity (text, derived-text, or byte level) to decide change / no-change. A suspected change is flagged to C007 - Currency Tracking as due-for-reconciliation, and the steward is shown the captured snapshot alongside the live source to judge what changed. There is no normalization-to-match-capture step, and the same coarse model covers text and binary — generalizing ADR008's asset handling to all sources. "See what actually changed" is preserved by presenting both artifacts, not by a machine diff.

## Consequences

- Enabling: removes the producer-agnostic normalization-ownership gap — revalidation no longer assumes a capture transform the Inbox never owned.
- Enabling: one revalidation model for all sources; ADR008's binary coarse diff is no longer a special case but the general rule.
- Enabling: matches the single-steward envelope — the steward, already in the loop, judges the change from both artifacts.
- Cost: no automated line-level "what changed" for text sources; the steward reads the two artifacts to see the difference. This refines [ADR002 - Retain Captured Source Snapshots](ADR002-retain-captured-source-snapshots.md)'s consequence "lets a curator see what actually changed" — still true, but via presented artifacts rather than a machine diff.
- Cost: a coarse comparison can over-flag (cosmetic changes) or under-flag; acceptable because the output is a steward prompt, not an automatic edit.
