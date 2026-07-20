# C012 — Coverage & Scope Review

**Role.** The read-only surveyor: periodic charter-vs-corpus surveys that find
**thin/absent scope areas** (coverage) and **content that has drifted out of scope**
(scope), then drive the steward attention surface and the sole writer to act on them.

## Capabilities

Two survey calls plus the intent verbs they drive — see
[INTERFACES.md](../INTERFACES.md) § C012:

- `coverage() -> CoverageReport` — enumerate the charter's scope areas
  (`C001.scope_areas`) and, against the corpus and its inflow, classify each as
  **absent / thin / adequate**, listing the known gaps and the material awaiting
  review (O007-R002).
- `scope() -> [ScopeDivergence]` — judge the corpus against the charter and return
  the concepts that have **drifted out of scope** (O008-R003).
- **Drives** `C002.evaluate` (act-vs-escalate), `C002.escalate` (the steward's
  judgment calls), and `C008.flag` (execute in-envelope scope flags). C012 itself
  writes **nothing** to the corpus.

C012 is the **surface half** of assisted upkeep (O004-R003): it surfaces what needs
attention and expresses intent; C008 carries out the change. It is deep because a
lot of judgment — mapping free-form charter areas to filed concepts, ruling on
on-scope-ness — hides behind these two calls, exactly as C007 hides triage judgment.

## Data model

**Stateless — owns no persistent store.** C012 is a read-only survey module: it
reads inputs at survey time, computes reports, and drives other components. There is
no coverage or scope artifact to build, stamp, invalidate, or keep in sync;
correctness cannot lag the corpus because C012 holds no derived state. The
scope-divergence *marker* that results from a survey lives in the concept file,
written and owned by **C008** (`flag`), not here.

Inputs it reads (never writes):

- **`C001.scope_areas()`** — the declared scope areas, the referent for both surveys.
  Required (PDR004), so a survey always has something to measure against.
- **`C009.index()`** — the concepts that exist, to map onto scope areas.
- **`C009.dangling()`** — link targets authored-as-intended but absent: **known gaps**
  the corpus itself points at.
- **`C006.list_held_aside()`** — recoverable material awaiting review; its count is
  part of the O007 proxy.

`CoverageReport` and `ScopeDivergence` are **C012-local** types (not shared
vocabulary — nothing else consumes them; see
[DECOMPOSITION.md](../DECOMPOSITION.md#shared-types)):

- `CoverageReport` = per-scope-area `{ area, classification: absent|thin|adequate,
  evidence }`, plus the enumerated known gaps (dangling targets) and the count of
  held-aside items awaiting review.
- `ScopeDivergence` = `{ concept, degree: divergent|borderline, evidence }`.

## Behavior

**Chosen mechanism: an agent scanning the corpus, consuming C009/C006's structured
signals as inputs.** The interface (§ C012) leaves this open — metric vs agent-scan,
like C010. This spec **chooses agent-scan** and records why below.

**`coverage()`** — for each `C001.scope_areas()` entry:

1. **Depth** — map the `C009.index()` concepts that address the area (a semantic
   match, not a tag lookup) and judge whether the area is *absent* (no concepts),
   *thin* (present but shallow relative to its declared purpose), or *adequate*.
2. **Known gaps** — attribute `C009.dangling()` targets to areas: a link the corpus
   authored toward a concept that does not yet exist is a gap the corpus names itself.
3. **Inflow** — surface the `C006.list_held_aside()` count as material that may fill
   thin areas once reviewed (part of the O007 proxy).

The report is the coverage signal model. C012 then routes each thin/absent finding
through `C002.evaluate`: because *what the team relies on* is the steward's call
(O007-R002), coverage findings are **escalated via `C002.escalate`**, not acted on —
there is no C012 write path that fills a gap (filling one is authoring, which flows
through intake→triage→C008). Coverage surfaces; it does not author.

**`scope()`** — walk `C009.index()` against the charter (purpose + scope areas) and
classify each concept as on-scope, *borderline*, or *divergent*. This is the scope
signal model. For each divergence, C012 proposes the flag through `C002.evaluate`:

- **within the envelope** → execute `C008.flag(concept, scope_divergent, reason)`.
  C008 owns the write and its atomic provenance entry; the flag is what makes the
  O008 proxy (flagged-and-unresolved) observable.
- **borderline / out of envelope** → `C002.escalate` for the steward (O008-R003).

**Read-only throughout.** C012 produces reports and expresses intent; every corpus
mutation — the scope-divergence marker included — is C008's. The old no-charter
degradation branch does **not** exist: the charter is required (PDR004), so neither
survey has a "scope has no referent" mode to fall back to.

**Why agent-scan.** Both surveys turn on *meaning*, not tokens: whether a concept
addresses a declared area, and whether it belongs at all, are semantic judgments a
fixed metric cannot make unless every concept self-declared its scope area — which
the OKF/charter model does not guarantee. An agent reading plain-text, self-describing
concepts (O006) matches on purpose, and leans on the deterministic structure C009
already exposes (index, dangling) rather than rebuilding it. The corpus is small and
in version control, so scanning it costs little and needs no persistent artifact — the
lowest-commitment mechanism, measurable before investing in a built metric. It is
**swappable with no ripple**: a future scope/coverage metric can move inside this spec
behind the unchanged `coverage`/`scope` signatures, since nothing else depends on which
mechanism is used.

## Relationships

Match the [INTERFACES.md dependency direction](../INTERFACES.md#dependency-direction-no-cycles)
exactly — `C012 Coverage&Scope → C001, C002, C006, C008, C009`:

- **Reads `C001`** — the charter scope areas, the referent for both surveys.
- **Reads `C009`** — `index()` (what exists) and `dangling()` (known gaps).
- **Reads `C006`** — `list_held_aside()`, for the awaiting-review count.
- **Drives `C002`** — `evaluate` to decide act-vs-escalate, `escalate` for the
  steward's judgment calls. No envelope logic is embedded here; C002 is the only
  place act-vs-escalate is decided.
- **Drives `C008`** — `flag` executes in-envelope scope flags. C012 never edits a
  concept file; C008 is the sole writer.

Read-only over the corpus, alongside C009/C010. It shares the O004-R003 **surface
half** with C011 — both surface and drive C008, neither writes directly.

## Decisions

- **Coverage/scope signal set — settled in-component, no ADR.** Coverage = charter
  scope areas vs `C009.index` + `C006.list_held_aside` + `C009.dangling` (authored-but-
  absent targets = known gaps); scope = charter vs corpus → drift-out. Reversible and
  behind the interface, so no cross-component decision record is warranted.
- **Mechanism (agent-scan) — component-internal choice, no ADR.** Left open by the
  interface (like C010); chosen here as agent-scan-over-corpus consuming C009/C006
  signals, changeable without touching any other component.
- **No-charter degradation — obsolete.** PDR004 makes the charter required and
  foundational; the old degrade-cleanly-without-a-charter path (old ADR003/013) is
  deliberately removed, a direct product consequence, not a regression.

## Success criteria

Tied to the two proxies C012 makes observable:

- **O007 — complete coverage.** Every declared scope area is classified each survey,
  so *silent gaps* (O007-RSK002) become visible signals; the count of held-aside items
  awaiting review is surfaced, not buried. Success is that thin/absent areas and the
  material that could fill them are put in front of the steward — not that C012 fills
  them (it cannot; that is authoring).
- **O008 — on-scope focus.** Scope-divergent concepts are flagged (in-envelope) or
  escalated (borderline), so the O008 proxy — *proportion flagged scope-divergent and
  not yet resolved* — is a real, falling number as the steward and C008 resolve flags.
  A flag raised and left unresolved is the metric working, not failing.
- **Honestly heuristic, and safe.** Agent-scan may misjudge an area's depth or a
  concept's scope; that is a designed limitation surfaced for steward judgment
  (O007-R002/O008-R003 both escalate the calls that matter). Because C012 writes
  nothing, a wrong survey degrades no outcome on its own — provenance, currency, and
  integrity are untouched; only what reaches the attention surface is at stake.
