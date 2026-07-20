# C007 — Triage

The disposition decider: scope + duplication + significance, all judged against the
charter, composed into one routing decision. A pure decision module — it owns no store
and executes nothing; it decides and routes.

## Capabilities

Behind the frozen surface ([INTERFACES.md § C007](../INTERFACES.md#c007-triage--the-disposition-decider)),
C007 exposes exactly one call:

- `triage(RawMaterial) -> TriageOutcome`, where
  `TriageOutcome = integrate | hold_aside | merge(into: ConceptId) | escalate`.

That single call hides the whole triage judgment: it reads the charter (C001), looks up
overlap against the corpus structure (C009), weighs scope / duplication / significance,
and routes the item to the one component that will act on it. `TriageOutcome` is the
*routing verb* C007 emits; it is **distinct** from the provenance `Disposition` C005
records (which captures the executed `outcome` after C008/C006 act).

## Data model

**Stateless.** C007 owns no persistent artifact. `triage` is a pure function of three
inputs — the charter (`C001.read`), the item (`RawMaterial` from `C006.next`), and the
corpus structure (`C009` lookup) — to one of four outcomes. It caches nothing, mints no
ids, and holds no queue (the raw queue and held-aside store are C006's). This one call
is therefore its entire test surface: fix a charter and a corpus, feed an item, assert
the outcome and the route taken. Depth here is *judgment*, not data.

## Behavior

`triage` runs three judgments, **each relative to the charter**, then composes them into
a candidate disposition, then puts that candidate to the governor. It never decides
act-vs-escalate itself and never performs the action.

**The three judgments (all charter-relative):**

1. **Scope** (item vs `C001.scope_areas`, O008-R002). Does the item fall within a
   declared scope area? Yields *in-scope* / *out-of-scope* / *ambiguous-scope*. The
   charter is the only referent — C007 embeds no independent notion of scope.
2. **Duplication** (item vs existing concepts via C009, O003-R003). Does the item
   overlap an already-integrated concept? C007 reads C009 as its lookup surface
   (`index` / `neighbors`) to surface candidate concepts, then judges overlap **relative
   to charter scope** (two items can be near-identical in text yet distinct concepts
   under the charter, or vice-versa). Yields *novel* / *overlaps(ConceptId)* /
   *ambiguous-overlap*.
3. **Significance** (item vs charter, O003-R004). Is the material substantial enough,
   *relative to the charter's purpose*, to earn a consumer-facing concept? Yields
   *above-bar* / *below-bar* / *borderline*.

**Overlap-detection approach (component-internal choice).** C007 does **not** build its
own similarity index over the corpus; it queries C009's deterministic structure for
candidate neighbours and applies its own overlap *judgment* on top. Rationale: C009 is
the single owner of corpus-derived navigational structure (one owner per artifact); a
second index in C007 would fork that structure and drift from it. C007 supplies the
scope-relative *judgment* (is this a duplicate worth merging?), C009 supplies the
*candidates*. The scoring heuristic itself (lexical / embedding / agent-over-corpus) is
an open, C007-internal choice behind the one call — mirroring C010's open-mechanism
stance — because it is a tunable heuristic, not a contract.

**Composition into a candidate.** The three judgments fold into one candidate:

- in-scope + novel + above-bar → **integrate**;
- in-scope + overlaps(ConceptId) + above-bar → **merge(into: ConceptId)**;
- below-bar, or unambiguously out-of-scope → **hold_aside** (recoverable, not destroyed);
- *any* ambiguity — ambiguous-scope, ambiguous-overlap, or borderline-significance →
  the candidate is **escalate**.

**Governance gate (act-vs-escalate lives in C002, never here).** C007 forms a
`ProposedAction` for the candidate and calls `C002.evaluate`. This is uniform: every
candidate is put to the governor, because C002 is the *only* place autonomy is decided
(no component embeds envelope logic). C002's response routes the final outcome:

- `evaluate → act` → C007 executes by *intent*: **integrate** / **merge** via
  `C008.integrate` / `C008.merge`; **hold_aside** via `C006.hold_aside(item, reason)`.
  (C002's conservative default acts on the mechanical/reversible — e.g. hold-aside —
  and escalates the consequential when the envelope is absent.)
- `evaluate → escalate(reason)`, or an intrinsically ambiguous candidate → the outcome
  is **escalate**, posted to the single steward attention surface via
  `C002.escalate(item, reason, evidence)`. Out-of-scope ambiguity is escalated for the
  steward, never auto-admitted (O008-R002).

C007 **executes nothing**: `C008` writes the corpus, `C006` moves material to held-aside
(and records that disposition), `C002` carries the escalation. C007 only decides and
routes — it holds **no store and no provenance obligation of its own**. Each triage
outcome is recorded by the component that executes it (`C006` hold-aside, `C008`
integrate/merge, `C002` escalation), so C007 is not a `C005.record` caller — consistent
with its dependency arrow, which lists no C005 edge.

## Relationships

Matches the frozen dependency direction (`C007 → C001, C002, C006, C008, C009`):

- **Reads C001** (`read` / `scope_areas`) — the scope + significance referent.
- **Reads C009** (`index` / `neighbors`) — the overlap-candidate lookup; C007 does not
  mutate or duplicate it.
- **Governs via C002** — `evaluate` (the sole act-vs-escalate decision) and `escalate`
  (the single attention surface). C007 embeds no envelope logic.
- **Routes to C006** (`hold_aside`) and **C008** (`integrate` / `merge`) by intent —
  those components own the store and the write; C007 owns neither.
- **Driven by C006** (`next` hands it the next untriaged item). C007 does not call
  producers and owns no queue.

C007 owns **no store** and performs **no corpus mutation** — every arrow out of it is a
read or an intent call.

## Decisions

- **The disposition set is settled in-component: `integrate | hold_aside | merge |
  escalate`.** These four fall directly out of the material lifecycle (author / hold /
  compose / defer-to-steward) crossed with the manage-by-exception seam; they are not a
  surprising, hard-to-reverse trade-off, so **no ADR** is warranted. This settles the
  old-tree **ADR004** (triage disposition set), whose live content is absorbed here.
- **Internal:** overlap detection reuses C009's structure rather than a private index
  (one-owner-per-artifact); the overlap-scoring heuristic is an open C007-internal choice.

## Success criteria

Tied to the **O003** and **O008** proxies (unresolved-flag proportions the charter makes
well-defined):

- **No blind duplicate is admitted (O003-R003).** When an incoming item overlaps an
  existing concept under charter scope, `triage` yields `merge(into: …)` or, when the
  resolution is ambiguous, `escalate` — never a second `integrate` of the same concept.
  This keeps redundant capture (O003-RSK003) out of the corpus.
- **Below-bar material is held aside, not destroyed (O003-R004).** Insubstantial items
  yield `hold_aside` (recoverable via `C006.restore`), gating
  *integrate-vs-hold-aside*, never *keep-vs-destroy*. Low-value accretion (O003-RSK004)
  is kept out of the consumer-facing corpus without loss.
- **Out-of-scope ambiguity is escalated, not auto-admitted (O008-R002).** Ambiguous-scope
  items yield `escalate` to the steward; only clearly in-scope, above-bar, within-envelope
  items reach `C008.integrate`. This is what keeps the O008 scope-divergence proxy low at
  the front door rather than relying on later reconciliation (C012).
- **Act-vs-escalate is correct because it is C002's.** Because C007 never embeds envelope
  logic, the autonomy boundary is auditable in one place; a governance change re-shapes
  triage's routing with no edit to C007.
