# CR012 - Coarse Source Revalidation

Re-scoped **C006 - Source Revalidation** from re-fetch → normalize-as-at-capture → line-diff to a coarse change signal that presents the captured snapshot and the live source for the steward to judge, unifying text and binary handling. Scoped to C006 - Source Revalidation. Captured by [ADR012 - Coarse Source Revalidation](../engineering/drs/ADR012-coarse-source-revalidation.md).

## Change

- **C006 - Source Revalidation**: the three-step re-fetch / normalize-as-at-capture / line-diff behavior is replaced by a coarse comparison of the live re-fetch against the captured snapshot at available fidelity (change / no-change); on a suspected change, the captured snapshot and the live source are presented together for the steward's judgment. The text-vs-binary diff split is removed — the coarse model ADR008 defined for binary assets becomes the general rule for all sources.

Before, C006 normalized the re-fetched source "the same way capture did at intake" and diffed line-by-line — but producer-agnostic intake (C001) owns no single capture transform to reproduce, so the normalization step had no reliable referent, and a line-diff was more precision than the low-volume single-steward envelope needs. After, revalidation is a coarse change signal and "see what actually changed" is preserved by presenting both artifacts rather than by a machine diff.

## Rationale

Removes the normalization-ownership gap between C006 and the producer-agnostic Inbox, and collapses two diff models into one. It matches the single-steward envelope, where the steward reviews any flagged drift anyway. See ADR012 for the decision and the rejected `normalized-by` alternative.

## Affects

- Engineering: C006 - Source Revalidation (behavior). Refines [ADR002 - Retain Captured Source Snapshots](../engineering/drs/ADR002-retain-captured-source-snapshots.md)'s "see what actually changed" consequence, and generalizes [ADR008 - Captured Source Assets](../engineering/drs/ADR008-captured-source-assets.md)'s coarse asset diff to all sources. No requirement or outcome text changes. Captured by [ADR012 - Coarse Source Revalidation](../engineering/drs/ADR012-coarse-source-revalidation.md).
