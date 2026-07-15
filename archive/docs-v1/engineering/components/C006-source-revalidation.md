# C006 - Source Revalidation

Detects when a cited source has changed since a statement was captured, so a curator can tell which statements no longer match the source they cite and which underlying sources have moved on.

Source Revalidation is a standing capability that compares the live external source against the form Wiki captured. It does not repair drift; it detects and reports it, so the steward can direct reconciliation. It runs as part of the **Lint** operation (see [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)).

## Behavior

The baseline is the captured-source snapshot the Inbox froze on integration (C001 - Inbox) — the immutable `raw/<id>` record, or, for a non-text source, the retained asset ([ADR008 - Captured Source Assets](../drs/ADR008-captured-source-assets.md)). Revalidation is a **coarse change signal**, not a precise diff (see [ADR012 - Coarse Source Revalidation](../drs/ADR012-coarse-source-revalidation.md)):

1. Re-fetches the item's `source` reference.
2. Compares the live result against the captured snapshot at whatever fidelity exists — text, derived-text, or byte level — to decide **change / no-change**. There is no normalization-to-match-capture step: intake is producer-agnostic (C001), so there is no single capture transform to reproduce, and one coarse model covers text and binary alike (generalizing ADR008's asset handling).
3. On a suspected change, presents the captured snapshot **and** the live source together for the steward to judge what changed.

A suspected change is the signal to C007 - Currency Tracking that the concept is due for reconciliation. "See what actually changed" is preserved by presenting both artifacts, not by a machine diff — appropriate to the low-volume single-steward envelope, where the steward reviews any flag anyway.

## Edge cases

- **`source: none` or unknown**: there is no external reference to revalidate against; provenance still rests on the captured snapshot the Inbox retained (C001), and nothing is reported as drift.
- **External source disappeared** (e.g. a 404 or removed page): reported as drift-by-disappearance. The captured snapshot remains the verifiable form of the statement's source (O001-R003), so the statement does not become unverifiable.
- **Provenance-purged concept**: when the captured source was purged (see [ADR013 - Provenance-Purge Cascade](../drs/ADR013-provenance-purge-cascade.md)), there is no baseline to compare against; the concept is treated as baseline-less — no drift is reported — rather than erroring, and its provenance-purged mark (C008) already signals the loss of source backing.

## Relationships

- **C001 - Inbox**: reads the captured-source snapshot (or retained asset) as the baseline the live source is compared against.
- **C007 - Currency Tracking**: a suspected change is the signal that a concept is due for reconciliation.
- **C008 - Lifecycle & Retirement**: a suspected change is a retirement-candidate signal; a provenance-purged concept has no baseline to revalidate.

## Success criteria

- For a cited statement whose external source has changed since capture, revalidation flags the change against the captured snapshot and presents both artifacts for the steward (O001-R002).
- A source that has disappeared is reported without the statement losing its captured-form verifiability (O001-R003).

## See Also

### Architectural Decision Records

- [ADR012 - Coarse Source Revalidation](../drs/ADR012-coarse-source-revalidation.md)
- [ADR013 - Provenance-Purge Cascade](../drs/ADR013-provenance-purge-cascade.md)
- [ADR008 - Captured Source Assets](../drs/ADR008-captured-source-assets.md)
- [ADR010 - Curation Agent Skill and Operations](../drs/ADR010-curation-agent-skill.md)

### Change Records

- [CR008 - Delivery Form and Standing-Capability Specifications](../../crs/CR008-delivery-form-and-standing-capabilities.md)
- [CR012 - Coarse Source Revalidation](../../crs/CR012-coarse-source-revalidation.md)
- [CR013 - Provenance-Purge Cascade](../../crs/CR013-provenance-purge-cascade.md)
