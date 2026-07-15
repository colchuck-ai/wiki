# CR004 - Captured Source Assets

Allowed non-text originating sources to be retained in their captured binary
form as version-controlled assets, resolving the collision between
captured-source retention (O001-R003) and the plain-text corpus (O005-R001).
Scoped to the O001 and O005 outcomes (product), the architecture README
constraint, and C001 - Inbox (engineering).

## Change

- **O001-R003 (Captured-source retention) and O005-R001 (Plain-text corpus)**:
  no requirement text is reversed. A clarifying distinction is added — curated
  *knowledge* stays plain-text markdown, while a retained *originating source*
  may be a binary asset when the source is inherently non-text. This makes
  O001-R003 satisfiable for non-text sources without weakening O005-R001's
  intent, which is tool-neutrality, not UTF-8-only bytes.
- **Architecture constraint**: the plain-text/version-control constraint is
  reworded from "no component may introduce a binary or tool-proprietary store"
  to "no proprietary or tool-locked store," so a plain binary committed to
  version control is permitted specifically as captured provenance.
- **C001 - Inbox**: the data model gains captured-asset retention — a non-text
  source is stored as a version-controlled asset (e.g. under `raw/assets/`)
  colocated with the raw item and referenced by it; the asset is the immutable
  captured-source snapshot. An edge case covers non-text capture, and the
  explicit-purge path extends to sensitive assets.

Before, the corpus/inbox was constrained plain-text-only and each item's
captured snapshot was its plain-text body, so an inherently binary source could
not be retained in its captured form — O001-R003 was unsatisfiable for non-text
sources and collided with the "no binary store" constraint. After, non-text
sources are retained as version-controlled binary assets; curated concepts stay
plain-text; the constraint forbids only a proprietary or tool-locked store.

## Rationale

O005-R001's real intent is that knowledge require no proprietary tool to read,
not that every byte be UTF-8. A committed PNG or PDF is tool-neutral, diffable at
the file level, and travels in-bundle, so it preserves the O005 portability the
requirement protects while making O001-R003's provenance guarantee hold for the
sources — images, PDFs, data files — that real curation actually encounters. A
text-only fallback would keep only a derivative transcription and lose the
captured form; a hash would show that a source changed but never what it was.

## Affects

- Product: O001-R003 and O005-R001 (clarifying notes distinguishing curated
  knowledge from retained source assets; no requirement reversed). Captured by
  [PDR002 - Provenance Assets Exception](../product/drs/PDR002-provenance-assets-exception.md).
- Engineering: the architecture README constraint bullet, and C001 - Inbox (data
  model and edge case for asset retention). Captured by
  [ADR008 - Captured Source Assets](../engineering/drs/ADR008-captured-source-assets.md).
