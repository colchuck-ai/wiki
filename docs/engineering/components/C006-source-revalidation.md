# C006 - Source Revalidation

Detects when a cited source has changed since a statement was captured, so a curator can tell which statements no longer match the source they cite and which underlying sources have moved on.

Source Revalidation is a standing capability that compares the live external source against the form Wiki captured. It does not repair drift; it detects and reports it, so the steward can direct reconciliation. It runs as part of the **Lint** operation (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Behavior

The baseline is the captured-source snapshot the Inbox froze on integration (C001 - Inbox) — the immutable `raw/<id>` record, or, for a non-text source, the retained asset ([ADR008 - Captured Source Assets](../drs/ADR008-captured-source-assets.md)). Revalidation:

1. Re-fetches the item's `source` reference.
2. Normalizes the fetched result the same way capture did at intake (for example, web page → markdown), so the comparison is like-for-like.
3. Diffs the normalized live result against the captured snapshot. A non-empty diff is **drift**.

A detected drift is the signal to C007 - Currency Tracking that the concept is due for reconciliation. For binary or asset sources the diff is necessarily coarser — change/no-change at the byte or derived-text level (per ADR008) — than the line-level diff available for text sources.

## Edge cases

- **`source: none` or unknown**: there is no external reference to revalidate against; provenance still rests on the captured snapshot the Inbox retained (C001), and nothing is reported as drift.
- **External source disappeared** (e.g. a 404 or removed page): reported as drift-by-disappearance. The captured snapshot remains the verifiable form of the statement's source (O001-R003), so the statement does not become unverifiable.

## Relationships

- **C001 - Inbox**: reads the captured-source snapshot (or retained asset) as the baseline the live source is diffed against.
- **C007 - Currency Tracking**: a detected drift is the signal that a concept is due for reconciliation.

## Success criteria

- For a cited statement whose external source has changed since capture, revalidation reports the drift against the captured snapshot (O001-R002).
- A source that has disappeared is reported without the statement losing its captured-form verifiability (O001-R003).

## See Also

### Architectural Decision Records

- [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)

### Change Records

- [CR008 - Delivery Form and Standing-Capability Specifications](../../crs/CR008-delivery-form-and-standing-capabilities.md)
