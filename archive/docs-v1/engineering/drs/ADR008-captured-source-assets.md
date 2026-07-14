# ADR008 - Captured Source Assets

C001 - Inbox retains a non-text captured source as a version-controlled **binary
asset** colocated with the raw item, and the architecture constraint "no
component may introduce a binary or tool-proprietary store" is refined to "no
**proprietary or tool-locked** store" — a plain binary asset (PNG, PDF, data
file) committed to version control is permitted specifically as captured source
provenance. This realizes [PDR002 - Provenance Assets Exception](../../product/drs/PDR002-provenance-assets-exception.md)
on the engineering side.

## Context

PDR002 decided that an inherently non-text source may be retained in its
captured binary form so O001-R003 (Captured-source retention) holds for it. The
engineering picture currently blocks this in two places. C001 - Inbox models
each item as one plain-text file whose frozen body *is* the captured-source
snapshot ([ADR003 - Inbox Storage Layout](ADR003-inbox-storage-layout.md)) — a
text-centric model that cannot hold an image or a PDF. And the architecture
README constraint forbids any binary store outright, which directly contradicts
O001-R003 for non-text sources. The architecture must decide how a binary
captured source is stored without introducing a proprietary or tool-locked
persistence layer.

## Options

- **Keep the text-only body**: store only a text transcription and drop the
  binary. Simplest, but fails PDR002 / O001-R003 — the captured *form* is lost.
- **External or proprietary blob store**: push binaries to an object store or a
  tool-specific attachment system. Removes the binary from the corpus, but
  violates O005 portability and tool-neutrality — the store would not travel
  with the bundle.
- **In-bundle version-controlled asset** *(chosen)*: retain the original bytes
  as a plain file under the inbox store, committed to version control alongside
  the raw item.

## Decision

Adopt **in-bundle version-controlled assets**. When a captured source is
non-text, C001 stores the original bytes as an asset (for example under a
`raw/assets/` path, following OKF §8 `references/`-style colocation) and the
item's text record references it. The asset is the immutable captured-source
snapshot that [C006 - Source Revalidation](../components/C006-source-revalidation.md)
diffs the live source against, and that an integrated statement's source link
can resolve to. The architecture constraint is reworded from "no binary or
tool-proprietary store" to "no proprietary or tool-locked store," so a plain
binary committed to version control is allowed as provenance while a bespoke or
tool-locked persistence layer is still forbidden. The explicit-purge path C001
already defines for sensitive material applies equally to sensitive assets.

## Consequences

- Enabling: captured provenance is retained at full fidelity for non-text
  sources, and revalidation has the real artifact to diff against.
- Enabling: the asset stays version-control-native and travels with the bundle,
  so O005 portability holds and no proprietary store is introduced.
- Cost: the inbox store may now contain binary blobs; monotonic growth is a
  retirement-review concern (O006-R002), as it already is for retained text.
- Cost: revalidation diffing of a binary source is necessarily coarser
  (change/no-change at the byte or derived-text level) than the line-level diff
  available for text.
- Cost: the "plain-text" guarantee must be restated as "no proprietary/tool-locked
  store" wherever it appears, so readers do not mistake it for a UTF-8-only rule.
