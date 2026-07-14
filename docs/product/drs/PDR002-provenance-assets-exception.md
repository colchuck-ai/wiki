# PDR002 - Provenance Assets Exception

The plain-text corpus (O005-R001) admits a bounded exception: non-text source
material may be retained in its **captured binary form** as provenance (an
"asset"). Curated knowledge — the concepts themselves — stays plain-text
markdown; only the retained *originating source* may be a binary asset, and only
when that source is inherently non-text (an image, a PDF, a data file).

## Context

Two requirements collide for a source that is not text. O001-R003
(Captured-source retention) requires retaining "the captured form of each
integrated statement's originating source" so a statement stays verifiable even
if the external source later changes or disappears. O005-R001 (Plain-text
corpus) requires storing knowledge as plain, human-readable files that need no
proprietary tool to read.

For an inherently binary source — a diagram, a scanned report, a spreadsheet —
its *captured form* cannot be plain text. Under a literal reading of O005-R001,
O001-R003 is therefore unsatisfiable for non-text sources: either the source is
not retained in its captured form (failing O001-R003) or it is retained as
bytes that are not plain text (failing O005-R001). The framework must decide
which requirement bends. OKF already anticipates the case — §8 permits mirroring
external material into a `references/` subdirectory as first-class material — so
the format does not forbid it.

## Options

- **Text-only Wiki**: refuse non-text sources, or retain only a text
  transcription of them. Keeps the corpus strictly plain-text, but loses the
  fidelity of the captured form — a transcription is a derivative, not the
  source — so it weakens the O001-R003 provenance guarantee exactly where a
  consumer would most want the original.
- **Hash / fingerprint only**: retain a checksum of the binary source, not the
  bytes. Detects that a source changed, but cannot show *what* it was, and
  leaves nothing to fall back on when the external source disappears.
- **Bounded asset exception** *(chosen)*: retain the binary source as a
  version-controlled asset alongside its text record; keep curated concepts
  plain-text.

## Decision

Adopt the **bounded asset exception**. The distinction that resolves the tension
is that O005-R001's real intent is *no proprietary, tool-locked store* — not *no
bytes that are not UTF-8*. A captured PNG or PDF committed to version control is
tool-neutral, diff-tracked at the file level, and travels inside the bundle, so
it preserves the O005 portability O005-R001 exists to protect. Curated concepts
remain plain-text markdown; only retained source provenance may be binary, and
only for sources that are inherently non-text. This is realized on the
engineering side by [ADR008 - Captured Source Assets](../../engineering/drs/ADR008-captured-source-assets.md).

## Consequences

- Enabling: O001-R003 becomes satisfiable for non-text sources — provenance is
  retained at full fidelity, and revalidation (O001-R002) has the real captured
  artifact to diff against.
- Enabling: the exception rides O005 portability — assets travel in-bundle and
  are version-control-native, so nothing proprietary is introduced.
- Cost: the corpus is no longer strictly UTF-8-only; a consumer must tolerate
  binary assets in the provenance store, and the "plain-text" guarantee now
  reads as "no proprietary/tool-locked store" wherever it is stated.
- Cost: retention of binary assets grows the store monotonically, the same
  retirement-review concern (O006-R002) that already applies to retained text.
