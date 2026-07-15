# C003 - Integration Authoring

Turns a triaged item and the steward's direction into a concept — composing it, attaching its source citation, and cross-linking related concepts with the reason for each stated — and executes the steward-chosen reconciliation when the item merges with an existing concept.

C003 is the curation pipeline's **write stage** and the only component that writes concepts into the corpus. It sits after C002 - Triage: it acts only on a task the steward has admitted, reads that task's disposition and reconciliation `direction` (from C001's record) and its `durable_ref`, and authors the concept **through C004's write face** so no OKF structural knowledge lives here. It realizes the authoring half of the product's **Ingest** operation — the steward supplies direction; C003 performs all composition, citation, linking, and metadata (intent in, work out). It fulfills cited claims (O001-R001) and intent-driven authoring (O004-R001), and shares reasoned cross-links (O005-R002) with C005 - Index & Navigation: C003 authors the link with its reason, C005 makes the graph navigable and keeps it intact.

## Data model

C003 owns **no staging artifact of its own**. Its output is the corpus itself — concepts and the provenance assets they cite — whose format is C004's, not C003's. C003 owns the *composition*, and it settles one path convention ADR002 deferred to it.

- **Authored concept.** An OKF concept C003 composes and writes via C004: frontmatter (a non-empty `type`, a `title`, an authoring-time `timestamp`, and any steward-supplied metadata), a structural markdown body, a `# Citations` block, and cross-links. C003 owns which fields and content go in; C004 owns the surrounding format.
- **Provenance area** (`/references/`, bundle root). The corpus-side location a durable source is promoted into at authoring — settling the promotion boundary C001 and [ADR002](../drs/ADR002-intake-staging-and-durable-source.md) left open. A promoted source is a **git-tracked asset, not a concept**: UTF-8 text for textual sources, or the bounded binary-asset fallback for non-text sources. C004's plain-text-concept `error` rule excludes declared snapshot assets, so a promoted binary source never reads as a conformance failure.
- **Citation.** A numbered `# Citations` block (OKF §8), rendered through C004's citation helper. **Concept-level by default**: a single-source concept's block covers all its claims. **Inline per-claim** only when a concept draws on more than one source (after a merge): individual claims carry numbered markers so a reader can tell which source backs which claim. **Link target depends on how the source was durably held** ([ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md)): an in-place source cites its existing repo path, unchanged; a captured-snapshot source cites the live `origin` URL as the primary link (OKF §8 permits an absolute-URL citation), naming the archived `/references/` asset as a durable fallback copy in the surrounding prose. This keeps the citation pointed at the actual external material and gives C006 - Source Revalidation a live locator to read directly off the concept.
- **Cross-link.** A bundle-relative link (`/…`, C004's link helper) whose **reason is stated in the surrounding prose**, because OKF §5.3 conveys the relationship kind in prose, not in the link itself. C003 emits no link without its reason.

## Interfaces

C003 exposes an in-process face. Every read of a task goes through C001, every OKF read/write goes through C004, and the only corpus writes are the authored concept and its promoted provenance asset.

- **`author(task_id) → concept_id`** — the main entry. Refuses unless the task's recorded `decision` is `admit`. Reads the task and its `durable_ref` (C001) and its reconciliation `direction` (C002's disposition block); promotes the durable source into `/references/`; composes the concept and its `# Citations` via C004; writes confirmed cross-links; and writes the concept file. On `direction.mode = keep-new`, mints a new concept (slug/path via C004); on `merge`, folds into `target-concept-id` in place (see Behavior). Returns the concept's C004-derived id.
- **`propose_links(concept | task_id) → link_candidate[]`** — the cross-link candidate scan: reuses C002's overlap candidates when present, else runs its own agent scan reading corpus concepts through C004, returning each candidate with a suggested reason for the steward to confirm, drop, or edit. **Advisory and read-only** — it writes nothing; `author` only writes links the steward confirmed. Self-contained: no C005 dependency ([ADR004](../drs/ADR004-triage-disposition-model.md) precedent).
- **`promote_source(task) → citation_ref`** — promotes the durable source from C001's staging into `/references/` and returns the citable bundle-relative path. Exposed as a helper; `author` calls it. For an already-version-controlled in-place source, it returns the existing repo path and copies no bytes.

## Behavior

- **Author only what the steward admitted.** No path writes a concept for a task whose recorded `decision` is not `admit`. Integration stays never-automatic: C002 decides *whether and what*, C003 does *how*, one admitted item at a time.
- **Intent in, authoring out.** From the admitted task and the steward's direction and note, C003 performs all composition, citation, linking, and metadata. The steward never hand-edits the corpus (O004-R001).
- **Cite what it writes.** Every authored concept carries a `# Citations` entry resolving to its originating source (O001-R001) — a promoted `/references/` asset, or the in-place repo path for an already-versioned source. A single source covers the whole concept; a merge that fuses more than one source adds inline per-claim markers so provenance stays legible.
- **Promote at authoring, cite by locator.** The durable source C001 held in staging is promoted into `/references/` as a git-tracked asset regardless of citation form — the archived copy always exists as the drift baseline C006 later hashes on demand. The citation link itself follows the source's kind ([ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md)): an in-place source already version-controlled alongside the corpus is cited at its existing path with no copy; a captured-snapshot source is cited by its live `origin` URL, with the archived `/references/` path named alongside it in prose. Because nothing outside the task pointed at the staging path, the one-time relocation of the archived copy is safe (ADR002).
- **Reasoned cross-links.** Links are authored with their reason in prose (OKF §5.3). Candidates are proposed by `propose_links`; the steward confirms; `author` writes only confirmed links. A link target that is not yet written is allowed — OKF §5.3 requires consumers to tolerate not-yet-written knowledge, and inbound-link integrity is C005's, not a block on authoring.
- **Merge preserves identity.** `direction.mode = merge` folds the new material into the existing `target-concept-id` **in place**: it unions the target's citations and cross-links, reconciles the prose per the steward's note, and keeps the target's concept-id and history. No new id is minted, so inbound links stay valid — referential integrity holds by construction, and neither C005 nor C008 is pulled onto the authoring path. `keep-new` instead mints a fresh concept.
- **Write the concept, not the index, log, or recency.** C003 writes the concept and its citations and links. The navigable index (C005) and the recency and change history (C007) are maintained by their owners over the corpus C003 produces. `author` calls `C007.record(concept_id, kind, summary)` as its terminal step — `kind = Creation` for `keep-new`, `kind = Update` for `merge` — so C007 owns the initial `timestamp` entirely; C003 never sets it itself (C007's data model).
- **No OKF structural literal.** Frontmatter, slug/path, citation and link formatting, and concept-id derivation all resolve through C004, so ADR001's one-owner invariant holds with C003 present.

## Edge cases

- **Merge target no longer exists** (deleted or relocated since triage): C003 does not silently recreate it. It surfaces the stale `direction` for the steward to re-triage, mirroring C002's stale-direction handling.
- **`keep-new` slug collides with an existing concept**: C004's slug helper disambiguates deterministically; C003 never overwrites an existing concept.
- **Admitted capture-failed task** (steward admitted despite no held material): C003 authors from the submitter's note and `origin`, cites the `origin` descriptively, and flags provenance as best-effort — there is no durable asset to promote. Consistent with C001's capture-failed record and C002's low-confidence triage of it.
- **Non-text source**: promoted into `/references/` as the version-controlled binary asset fallback and cited by path; C004 excludes declared snapshot assets from concept validation.
- **Cross-link target not yet written**: written as a forward reference; not an error (OKF §5.3). C005 resolves integrity later.
- **Multi-source merge**: `# Citations` unions the sources and individual claims carry inline markers, so a reader can trace each claim to the source it came from.
- **Source drifts after authoring**: not C003's concern — C006 - Source Revalidation detects drift by comparing the citation's live locator against the archived `/references/` asset, hashed on demand rather than against any stored baseline.

## Relationships

- **C002 - Triage**: consumes an admitted task's `triage.decision` and `direction` (`merge` with a `target-concept-id`, or `keep-new`) plus any note, read from C001's task record. C002 decides whether and what to admit; C003 executes how. A `direction` gone stale between admission and authoring is surfaced for re-triage, not acted on blindly.
- **C001 - Ingestion Queue**: reads `get_task` and `durable_ref`; promotes the staged snapshot into `/references/`, settling the promotion boundary C001 deferred. C001 durably holds the source until promotion; C003 performs the relocation and citation. A rejected task never reaches C003 (C001 discards it at triage).
- **C004 - OKF Conformance**: the sole authoring path — serialize frontmatter, derive slug/path and concept-id, render the `# Citations` block and bundle-relative links. C003 encodes no OKF literal and may call `validate` on the authored concept. ADR001's one-owner invariant holds with C003 present.
- **C005 - Index & Navigation**: C003 authors concepts and their reasoned cross-links (O005-R002); C005 maintains the navigable index and inbound-link integrity over that output. No hard dependency — link-candidate discovery is self-contained in C003 today (agent scan through C004, reusing C002's overlap candidates), and can later delegate to C005's index/graph without changing this contract (ADR004 precedent).
- **C007 - Currency Tracking**: `author` calls `record` as its terminal step, so authoring is the first event in a concept's history; recording recency (the `timestamp` value) and the dated change log is C007's alone — C003 emits the concept through C004 but never sets `timestamp` itself.
- **C006 - Source Revalidation**: reads its citation's live `origin` URL (captured-snapshot source) or in-place path (in-place source) to know what to revalidate, and the promoted `/references/` asset as the drift baseline it hashes on demand — both read straight off the authored concept via C004, with no dependency on C001's task record after authoring ([ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md)).
- **C008 - Lifecycle & Retirement**: an ordinary merge preserves the target and marks no supersession, so it never invokes C008. Deprecation and retirement are C008's distinct flow, not triggered by authoring.
- **Boundary**: C003 owns concept authoring, source promotion and citation, reasoned cross-linking, and merge execution. It does not adjudicate admission (C002), intake or durably hold material (C001), define the format (C004), maintain the index or link integrity (C005), track recency or history (C007), revalidate sources (C006), or retire concepts (C008). It writes to the corpus only for steward-admitted items.

## Success criteria

- **Nothing authored without admission**: no path writes a concept for a task whose recorded `decision` is not `admit` — verifiable by calling `author` on a held or rejected task and confirming the corpus is byte-for-byte unchanged.
- **Every concept cited**: each authored concept carries a `# Citations` entry that resolves to its originating source (a `/references/` asset or an in-place path); a concept fused from more than one source carries inline per-claim markers. Testable by authoring and confirming the citation resolves.
- **Durable source promoted and citable**: after `author` of an external source, the source is retrievable from the corpus at the cited path independent of its origin; an already-version-controlled source is cited in place with no bytes copied.
- **Merge preserves referential integrity**: a `merge` keeps the target's concept-id and history and mints no new id, so inbound links to it remain valid — verifiable by merging and confirming the target id and its inbound links are unchanged.
- **Reasoned links**: every cross-link C003 writes has an accompanying prose reason; no link is emitted without one (OKF §5.3).
- **Intent-driven, no hand-editing**: authoring a concept requires only the steward's direction; C003 performs all composition, citation, linking, and metadata (O004-R001).
- **One-owner invariant preserved**: C003 contains no OKF structural literal — frontmatter fields, reserved filenames, link form, or citation/index structure — all resolve through C004, so ADR001's grep check stays clean with C003 present.

## Notes

- The provenance area (`/references/` at the bundle root) and the cite-by-path model — a promoted source is an **asset, not a concept** — settle the corpus-side provenance layout ADR002 deferred to C003, and are captured with the citation-granularity and merge decisions in [ADR005](../drs/ADR005-integration-authoring-provenance-and-merge.md).
- Authoring is the terminal write of the product's **Ingest** operation, surfaced through the curation Agent Skill: the steward directs; the tool authors, cites, and links.
- Link-candidate discovery is a self-contained agent scan through C004, reusing C002's overlap candidates when available — no C005 dependency, consistent with [ADR004](../drs/ADR004-triage-disposition-model.md), and optimizable later by delegating to C005.
- The captured-snapshot citation link form (live `origin` URL, archived asset as fallback) was settled after the fact, once specifying C006 - Source Revalidation exposed the need — see [ADR007](../drs/ADR007-citation-link-form-for-drift-revalidation.md) and [CR001](../../crs/CR001-external-citations-cite-live-origin.md).

## See Also

### Architectural Decision Records

- [ADR005 - Integration authoring, provenance, and merge model](../drs/ADR005-integration-authoring-provenance-and-merge.md)
- [ADR007 - Citation link form for drift revalidation](../drs/ADR007-citation-link-form-for-drift-revalidation.md)

### Change Records

- [CR001 - External citations cite the live origin](../../crs/CR001-external-citations-cite-live-origin.md)
