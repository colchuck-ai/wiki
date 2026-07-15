# C005 - Index & Navigation

Maintains the navigable index of what the corpus contains, traverses the knowledge cross-link graph, and surfaces the inbound links to a concept before it is removed or relocated so nothing is left following a dangling reference.

C005 is a **read-and-derive layer** over the corpus C003 writes: it never authors a concept or a cross-link itself. It materializes the OKF §6 index at every directory that holds concepts and computes the cross-link graph and inbound-link lookups on demand, rather than maintaining a second stored graph (see [ADR006](../drs/ADR006-index-materialization-and-graph-computation.md)). It fulfills navigable index (O005-R001) and referential integrity (O005-R003) outright, and shares reasoned cross-links (O005-R002) with C003 - Integration Authoring: C003 authors a link with its reason stated in prose; C005 makes the resulting graph traversable and keeps callers from dead-ending on it. Like C006, C005 detects and reports; it repairs nothing itself.

## Data model

- **Index file (`index.md`)**, one per directory that contains at least one concept, rendered through C004's index-rendering helper (OKF §6: no frontmatter, sections grouping entries under a heading). C005 groups entries into a `# Subdirectories` section (linking child directories that themselves hold concepts) and a `# Concepts` section (each entry: title, bundle-relative link, and the concept's frontmatter `description` when present, per OKF §6's SHOULD). Entries are ordered deterministically (by title) so regenerating with no corpus change produces no diff. A directory with no concepts of its own but with concept-bearing subdirectories still gets an index listing only the `# Subdirectories` section; either section is omitted when it would be empty (never an empty heading).
- **Cross-link graph.** Not a stored artifact — a computed view over the bundle-relative links (C004 §5) found in concept bodies, read through C004's parse helper. Directed, untyped edges (OKF §5.3: the relationship kind lives in prose, not the link); computed fresh on every query, never cached.
- **Inbound-link result** (transient). One entry per concept in the corpus whose body links to the queried target: `{ source-concept-id, target-concept-id }`. Never persisted — it is the answer to a query, not a record.
- **Dangling-link result** (transient). One entry per link whose target does not resolve to an existing concept: `{ source-concept-id, target-path }`. C005 makes no claim about *why* a link is dangling (not-yet-written per OKF §5.3, or orphaned by a removal) — that judgment belongs to the consumer (C009).

Provenance assets under `/references/` (C003's promoted sources) are not concepts and never appear in an index entry or the graph — the graph stays pure knowledge, matching ADR005's provenance model.

## Interfaces

C005 exposes an in-process face. `reindex` is the only corpus mutation; every other call is read-only.

- **`reindex(scope?) → updated_paths[]`** — regenerates the `index.md` for `scope` (a directory) and, when omitted, the whole bundle, from the concepts currently on disk (read through C004). Idempotent: re-running with no intervening corpus change returns an empty `updated_paths[]` and writes nothing. Called by any component that writes, deprecates, retires, or relocates a concept — C003 after authoring, C008 after a lifecycle change — since C005 does not watch the corpus.
- **`inbound_links(concept_id) → link[]`** — every concept in the corpus that links to `concept_id`, computed by scanning the corpus fresh. The concrete mechanism for O005-R003: called before a removal or relocation completes so inbound references can be resolved rather than left dangling.
- **`dangling_links(scope?) → link[]`** — every link in `scope` (default: whole bundle) whose target does not resolve to an existing concept. Consumed by C009 - Coverage Review as one gap signal.
- **`graph(scope?) → edges[]`** — the computed outbound/inbound edge set for `scope`, for traversal use (e.g. a curation Agent Skill surfacing related concepts). Read-only, computed fresh.

## Behavior

- **Materialize the index, compute the graph.** `index.md` is a generated, git-tracked artifact kept in sync by `reindex`; the graph and inbound/dangling-link queries are recomputed on every call with no persisted reverse structure. The split is deliberate — see [ADR006](../drs/ADR006-index-materialization-and-graph-computation.md).
- **Generated, never hand-edited.** `index.md` is C005's output, not an authoring surface — no other component or steward action writes to it directly. Regenerating it is always safe and always idempotent.
- **Write-triggered, not polled.** C005 does not watch the corpus for changes. It relies on writers calling `reindex` after a mutation, consistent with the architecture's "no shared mutable state" principle — C005 holds no background process and no cache that could drift silently.
- **Surface before dead-ending.** `inbound_links` is the check any removal or relocation flow must run first. C005 does not block the caller or repair the links itself — it supplies the information; resolving, retargeting, or knowingly accepting a transitional dangling state is the caller's (C008's) and ultimately the steward's judgment, mirroring the architecture's detect-don't-repair posture (shared with C006, C009, C010).
- **Charter is indexed, not excluded.** The charter concept (C010, `type: charter`) is an ordinary concept for indexing and graph purposes — it appears in `index.md` and in `graph()` like any other concept (ADR003). C005 may present it under a distinct heading rather than mixed into ordinary entries, but this is a presentation choice, not an exclusion; C005 has no candidacy list to exclude it from (unlike C008's retirement candidates or C009's coverage gaps).
- **Tolerate forward references.** A link to a not-yet-written concept is not an error (OKF §5.3) and is not filtered out of `dangling_links` — it is reported like any other unresolved link, since only the consumer's context (not C005) can tell a deliberate forward reference from an orphaned one.

## Edge cases

- **Directory with subdirectories but no concepts of its own**: `index.md` lists only `# Subdirectories`; no empty `# Concepts` heading is emitted.
- **Directory with neither concepts nor concept-bearing subdirectories**: no `index.md` is generated at all — an index with nothing to list is omitted, not emitted empty.
- **`reindex` called with no corpus change since the last call**: returns `updated_paths: []` and performs no write — the idempotence success criterion.
- **Concept moved or removed without a corresponding `reindex`** (a writer bypassed the contract): the on-disk `index.md` is stale until the next `reindex` covering that path; C005 does not detect this drift proactively — a stewardship gap to catch via Lint/Survey passes that call `reindex`, not a runtime guarantee C005 enforces.
- **Forward reference to a concept not yet written**: appears in `dangling_links`, tolerated per OKF §5.3, not flagged as malformed — C009 is the consumer that judges whether it represents a real gap.
- **Link to a concept later removed by C008**: becomes a genuinely broken link, indistinguishable in `dangling_links`'s output from a forward reference — both are "target does not resolve," and C005 makes no claim about cause.
- **Two concepts in the same directory with identical titles**: both appear as distinct `index.md` entries (distinguished by their linked path, per C004's deterministic slug disambiguation); C005 does not merge or deduplicate by title.

## Relationships

- **C003 - Integration Authoring**: the primary writer C005 tracks. C003 authors concepts and their reasoned cross-links; it calls `reindex` after `author` so new and changed concepts appear in their directory's index. C005 does not participate in C003's own cross-link candidate scan (ADR005) — no dependency runs from C003 to C005.
- **C004 - OKF Conformance**: C005's sole path to corpus content — the index-rendering helper (OKF §6) to serialize `index.md`, the parse helper to read concept frontmatter (for descriptions) and body links (for the graph), and the link-syntax helper to recognize a bundle-relative link. C005 owns index maintenance and link-integrity semantics; C004 owns the surrounding format and validates only `index.md`'s structural shape.
- **C008 - Lifecycle & Retirement**: the one component with a real dependency on C005 (see [ADR006](../drs/ADR006-index-materialization-and-graph-computation.md)). C008 calls `inbound_links` before completing a deprecation, retirement, or relocation, and calls `reindex` after, since the concept's index entry and deprecation marker change.
- **C009 - Coverage Review**: consumes `dangling_links` as one input to its sourcing agenda, alongside the charter's declared scope and agent-proposed gaps.
- **C010 - Charter**: the charter concept is indexed and graphed like any concept; C005 may present it under a distinct heading, but C010 imposes no exclusion requirement on C005 the way it does on C008 and C009.
- **C002 - Triage / C001 - Ingestion Queue**: no relationship. Both stay self-contained (ADR004, ADR005); C005 is not on the intake or triage path.
- **Boundary**: C005 owns index generation and cross-link graph computation, including inbound- and dangling-link reporting. It does not author concepts or links (C003), decide what to retire or relocate (C008), judge scope or gaps (C009, C010), or define OKF structure (C004). It writes only `index.md`; it never repairs a link.

## Success criteria

- **Index completeness**: after `reindex`, every concept on disk appears in its directory's `index.md`; no concept is present in the corpus and absent from its directory's listing.
- **Idempotent regeneration**: running `reindex` twice with no intervening corpus change produces a byte-identical `index.md` both times — the second run's `updated_paths` is empty.
- **OKF-conformant index**: every generated `index.md` carries no frontmatter and parses cleanly through C004 (OKF §6).
- **Inbound-link fidelity**: `inbound_links(id)` returns exactly the concepts whose body currently links to `id` — verifiable by adding a link and confirming it appears, then removing it and confirming it disappears.
- **No hidden mutation**: `inbound_links`, `dangling_links`, and `graph` never write to the corpus; `reindex` writes only `index.md` files, never concept content or links.
- **Charter surfaced**: a declared charter concept is discoverable via `graph()` and appears in its directory's `index.md`, confirming C005 applies no silent exclusion.

## Notes

- Materializing `index.md` (rather than synthesizing it only at query time) and computing the graph and inbound/dangling-link queries on demand (rather than maintaining a persisted reverse index) are the two storage decisions this component turned on; both are captured, with alternatives and consequences, in [ADR006](../drs/ADR006-index-materialization-and-graph-computation.md).
- The specific `index.md` grouping convention (`# Subdirectories` then `# Concepts`, alphabetical within each) is a Wiki choice within OKF §6's freedom, recorded here rather than in a separate ADR — promote it if it becomes contested.
- C005 is the first component with a genuine dependency from another undrafted component (C008) rather than the self-contained-scan pattern C002 and C003 established — relevant context when C008 is specced next.

## See Also

### Architectural Decision Records

- [ADR003 - Charter as an in-corpus concept and single scope authority](../drs/ADR003-charter-as-in-corpus-concept.md)
- [ADR006 - Index materialization and on-demand graph computation](../drs/ADR006-index-materialization-and-graph-computation.md)
