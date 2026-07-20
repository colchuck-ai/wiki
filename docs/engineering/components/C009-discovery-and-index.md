# C009 — Discovery & Index

**Role.** The **deterministic** navigational structure over the corpus — the index, the
reason-annotated cross-link graph, and the referential-integrity queries derived from
them. Read-only over the corpus; machine-readable-first.

## Capabilities

Tied to the frozen interface ([INTERFACES.md](../INTERFACES.md) § C009 — signatures are
authoritative there; not restated or widened):

- **`index() -> Index`** — the navigable listing over **all** concepts, as machine-readable
  structure with a human rendering (O005-R001).
- **`neighbors(ConceptId) -> [Link]`** — the concept's **outbound** reason-annotated
  cross-links, for traversal (O005-R002).
- **`inbound_links(ConceptId) -> [Link]`** — who points *at* a concept, for referential
  integrity **before** a retire/relocate (O005-R003). Read by C011.
- **`dangling() -> [Link]`** — links resolving to nothing: an integrity signal (feeds C012).

Every result is derived from the corpus at call time; none mutate it.

## Data model

C009 owns **no source-of-truth state** — the corpus (concept files written by C008) is the
single source of truth. What C009 derives from it is two structures, machine-readable
first and rendered second:

- **The navigable index** — a listing over every concept, carrying its `ConceptId`,
  canonical location/name, and the OKF structural grouping. The parseable form is primary
  (an agent enumerates the corpus without opening files); the human rendering is a view of
  the same structure, never a separate artifact.
- **The reason-annotated cross-link graph** — nodes are concepts, edges are
  `Link = { target, reason }` (shared type, [DECOMPOSITION.md](../DECOMPOSITION.md#shared-types)).
  The `reason` is **authored by C008** in the concept file at distillation (O005-R002 write
  side); C009 reads those links and builds the graph and its reverse index. C009 never
  writes a link.

Both structures are stamped with the **corpus revision they were built from** (the as-of
token — see freshness, below). Links are the concepts' own fields; the graph is a *view* of
them, not a second copy of the truth.

## Behavior

**Materialize-vs-compute — decided here (settles old ADR006, component-internal).**
C009 uses a **hybrid: compute-on-read semantics backed by a corpus-revision-keyed cache**,
validated against the live corpus on every call. There is **no stored index that C008 must
keep in sync**: the corpus is version-controlled (O006-R003), so C009 fingerprints the
current corpus revision (e.g. the tree/commit hash), and on each call either serves a cache
built at that exact revision or rescans and rebuilds. The cache is a *performance* copy
keyed on corpus state, never authoritative and never invalidated by a push from C008.

*Rationale.* (1) A stored, separately-maintained index would be shared mutable state that
C008 would have to orchestrate — exactly what ADR001 forbids ("derived views recompute from
the corpus… C008 does not orchestrate them"). Validate-against-corpus removes that coupling.
(2) Pure compute-on-read is correct but rescans the whole corpus per call; the revision key
makes the common case (corpus unchanged) O(1) while keeping every served view provably
current. (3) It mirrors C005's discipline — the expensive derivation is bounded by *change*,
not by *read volume*, and the view carries how fresh it is.

**Freshness / dry-run query.** Because the cache is revalidated against the corpus
fingerprint on every call, a returned view **is current by construction** — its as-of
revision equals corpus HEAD at call time. That as-of revision is carried on the returned
`Index` (and shared by the `neighbors`/`inbound_links`/`dangling` results of the same call),
so a caller can confirm *which* corpus state the view reflects without a separate call — the
frozen 4-call surface is **not** widened. Every call is a pure read (dry-run): it observes
the corpus, never mutates it.

**`index`** enumerates all concepts into the navigable listing (machine-readable + rendered),
so a consumer sees what exists before opening documents.

**`neighbors`** returns a concept's outbound `Link`s with their authored reasons — the
traversal edges, reason included so a consumer knows *why* to follow one.

**`inbound_links`** walks the reverse graph: every concept whose authored links `target` the
given concept. This is the pre-condition C011 reads **before** driving a `C008.deprecate`/
`relocate`, so inbound links are resolved (retargeted to a successor) rather than left
dangling (O005-R003).

**`dangling`** returns every `Link` whose `target` resolves to no live concept. C009 uses
**`C003.validate`** to tell a *malformed* target (violates OKF naming/structure) from a
*well-formed-but-absent* one; both surface as dangling for C012, but the classification keeps
the signal honest. This is the "absence of dangling links" half of the O005 proxy.

**Deterministic — contrast C010.** Given a corpus revision, every call returns the same
result: the index, graph, and integrity queries are exact functions of the concept files.
This is precisely what distinguishes C009 from **C010 Question Retrieval**, which is
heuristic and may miss (ADR005). A consumer who knows the structure navigates C009; one who
does not, asks C010.

## Relationships

Matches [INTERFACES.md → Dependency direction](../INTERFACES.md#dependency-direction-no-cycles)
exactly:

- **Reads:** the **corpus** (concept files + their authored links), written by **C008** — the
  single source of truth. Read-only; C009 **never** mutates concepts and **never** writes
  links.
- **Depends on:** **C003** — `validate` for name/structure classification in `dangling`
  (`C003 ◀── C009`).
- **Driven by:** **C011** (`inbound_links`, before retire/relocate); **C012** (`index` and
  `dangling`, for coverage/integrity surveys); any **consumer** (`index`/`neighbors` to
  browse and traverse).
- **No cycle:** C009's only downward edge is to the stateless C003; it calls no mutator and
  is called only inbound. It provides *reads*, drives nothing.

## Decisions

- **Materialize-vs-compute settled here — no architecture ADR.** Per the
  [DECOMPOSITION.md](../DECOMPOSITION.md) old-ADR checklist, old **ADR006** (index
  materialize-vs-compute) was deferred to this spec as a component-internal choice behind
  C009's frozen interface. Resolved above: **hybrid compute-on-read with a
  corpus-revision-keyed cache, validated per call.** It is behind the interface, reverses
  cheaply (drop the cache → pure compute-on-read), and needs no cross-component decision.
- **No new fork.** The choice touches no other component's contract; C008's non-orchestration
  of derived views (ADR001) is *relied on*, not changed.

## Success criteria

Tied to the **O005 proxy** — *index coverage over all concepts, cross-link density with
stated reasons, and absence of dangling links, exposed both as a rendered view and as
machine-parseable structure*:

- **Coverage:** `index` lists **every** concept in the corpus at the served revision — no
  concept is absent from the listing (O005-R001).
- **Reasoned density:** every edge `neighbors`/`inbound_links` return carries the `reason`
  C008 authored; a link without a stated reason is a conformance violation surfaced via
  `C003.validate`, not a silent edge (O005-R002).
- **Integrity:** `dangling` finds every link resolving to nothing, so referential breaks are
  visible to C012 and resolvable via C011 before they strand a consumer (O005-R003); and
  `inbound_links` is complete, so no inbound link is missed ahead of a retire/relocate.
- **Dual affordance:** every result is machine-parseable structure first with an equivalent
  human rendering — an agent and a teammate reach the same navigation (O005-RSK003).
- **Determinism:** two calls at the same corpus revision return identical results; the as-of
  revision on each view lets a caller confirm it reflects current corpus state.
