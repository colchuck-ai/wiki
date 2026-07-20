# C001 — Charter

The required, in-corpus scope referent: the corpus's declared purpose and scope
areas, read by every scope / significance / duplication / coverage judgment.

## Capabilities

- **`read` — the scope referent.** Hand any caller the current `Charter`. This
  is the single question "what is this corpus *for*, and what does it cover?"
  answered from one place, so no component embeds its own notion of scope.
- **`scope_areas` — the enumerable partition.** Project the charter's scope
  areas as a list, the unit coverage (O007) and scope-divergence (O008) surveys
  iterate. A convenience over `read().scope`; may be empty (provisional).
- **`declare` — establish/replace.** The steward states the charter (purpose +
  scope areas). Used at bootstrap and for a wholesale restatement.
- **`revise` — sharpen in place.** The steward applies a partial change (edit
  the purpose, add/rename/remove a scope area). The normal cadence: the charter
  is sharpened as the material is understood, not rewritten.

## Data model

C001 owns exactly one artifact: the **charter**, a *policy anchor* (the shared
declared/persistent/in-corpus/revisable pattern, sibling to C002's envelope).

- **Shape:** the shared type `Charter = { purpose, [ScopeArea] }`.
  - `purpose` — a consumer-legible statement of what the corpus is for.
  - `ScopeArea` — a named area (identity + short description) that partitions the
    intended scope. It is the referent unit for coverage and scope divergence.
- **Where it lives:** a single canonical OKF-conformant file at a well-known
  corpus path — plain-text, version-controlled (O006), **machine-readable-first**
  (structured `purpose` / `scope_areas` fields) with a human rendering. It is a
  *policy anchor*, **not a concept**, so it sits outside the concept namespace
  C008 writes.
- **Provisional is legal.** The charter is required to *exist*, not to be
  complete (PDR004): `purpose` may be a placeholder and `scope_areas` may be
  empty or partial. There is no "invalid" or "absent" charter state.

## Behavior

- **`read` always succeeds.** Because the charter is required and seeded at
  corpus init (see bootstrap), it always exists; `read` returns the current
  state — possibly provisional — and has no absent / error mode. Every downstream
  scope judgment can therefore assume a referent unconditionally.
- **`scope_areas`** returns the declared areas; an empty list is a truthful
  signal that scope is not yet partitioned, not a failure.
- **`declare` / `revise`** are **steward-facing only** — declaring scope is
  curation judgment. C001 writes the charter artifact in place and reports the
  change to `C005.record`. No autonomous path writes the charter: C002 and the
  pipeline *read* it, never mutate it.
- **Writer identity (resolved internally).** C001 writes its *own* artifact
  directly, not through C008. Rationale: the sole-writer invariant (ADR001) is
  scoped to the **concept corpus**; policy anchors are written by their owners
  (C001 charter, C002 envelope), which is also why the frozen tree lets C001
  depend on nothing — routing through C008 would invert the foundation.
- **Bootstrap (resolved internally).** A required charter must exist from day
  one *without* a setup wall on a finished scope statement. Resolution: corpus
  initialization **seeds a provisional charter** — the steward's purpose if given
  at init, else a placeholder purpose with empty `scope_areas` — and records the
  genesis via `C005.record`. `read` is thus total from the first commit, and the
  seed is sharpened by ordinary `revise`.
- **Provenance.** `declare` / `revise` emit a `Disposition` to `C005.record`
  (target = the charter, actor = `steward`, stamped `envelope_version`, time,
  detail). The `outcome` is the policy-anchor value `declared` (for `declare`) or
  `revised` (for `revise`) — the shared-type additions that let charter changes
  record through the one provenance log (see Decisions).

## Relationships

- **Depends on:** nothing — the foundation (matches the frozen dependency
  direction: `C001 ◀── C002, C007, C008, C011, C012`).
- **Reports to:** `C005.record` for provenance. This is the append-only sink
  every actor reports to, not a downward structural dependency.
- **Read by:**
  - **C002 Governance** — the envelope's scope rules read the charter.
  - **C007 Triage** — scope, significance, and duplication are all judged against it.
  - **C008 Integration** — does *not* read C001 directly; charter influence
    reaches C008 through `C002.evaluate` (which reads C001). C008's frozen §
    lists no C001 call. (The terse arrow-block still shows `C001 ◀── C008`; that
    is the reconciliation discrepancy noted for Phase 4.)
  - **C011 Lifecycle** — relevance of retirement candidates is judged partly vs scope.
  - **C012 Coverage & Scope** — coverage (scope areas vs index) and scope
    divergence (charter vs corpus) survey against it.

## Decisions

- **PDR004 — charter required and foundational.** Required to exist and be
  revisable (not complete on day one), and load-bearing: an actual input to
  triage, the significance bar, duplication, and coverage — not dead compliance.
- **Charter-less degradation path consciously dropped** (DECOMPOSITION drop #3):
  the old optional-charter mode (triage dropping its scope criterion, coverage
  narrowing) is deliberately removed. There is no "no charter" branch to spec.
- **Internal:** C001 is the writer of its own policy-anchor artifact (not C008);
  a provisional charter is seeded at init so `read` is total.
- **Policy-anchor provenance (steward-ratified).** `declare` / `revise` record to
  `C005.record` with the `outcome` values `declared` / `revised`, added to the shared
  `Disposition.outcome` type so policy-anchor changes travel in the one provenance log
  (rather than a separate history surface). Shared with C002's envelope.

## Success criteria

- **`read` is total.** For every corpus at every point in time it returns a
  `Charter` (possibly provisional) — never absent, never an error. This is the
  guarantee that **scope always has a referent**.
- **The O007 / O008 proxies are well-defined.** Coverage (O007: scope areas with
  adequate coverage) and on-scope focus (O008: scope-divergent concepts) both
  resolve against a present charter, so neither proxy can enter the undefined
  "no referent" state PDR004 exists to prevent.
- **`scope_areas` is enumerable**, letting `C012.coverage` compute per-area
  coverage; significance (C007) and duplication (C007) resolve their
  charter-relative judgments with no missing-input case.
- **Every change is provenance-complete:** each `declare` / `revise` appears in
  `C005.history(charter)`.
- **Consumer-legible & portable:** the charter is plain-text, version-controlled,
  and OKF-conformant, so a consumer can read what the corpus is for and it
  travels with the corpus (O006).
