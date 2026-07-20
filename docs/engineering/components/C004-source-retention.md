# C004 — Source Retention

**Role.** Hold the durable/live source pair for every citation and compute drift — one substrate serving both provenance (O001) and fidelity (O009).

## Capabilities

Three calls, frozen in [INTERFACES.md](../INTERFACES.md) § C004:

- `retain(source) -> Citation` — capture a version-controlled durable snapshot and, when a live origin exists, record its distinct locator. Idempotent per source.
- `resolve(durable_ref) -> SourceContent` — fetch the durable snapshot: the form grounding and drift check *against*. Never returns the live origin.
- `check_drift(Citation) -> DriftStatus` — fetch the `live_locator` and diff it against `durable_ref` → `unchanged | drifted | unreachable`.

Behind these, C004 hides *how sources are held durably* and *how live-vs-snapshot drift is detected*. It is otherwise stateless: the snapshots are the only artifact, and they live in the corpus under version control.

## Data model

Shared types (see [DECOMPOSITION.md](../DECOMPOSITION.md#shared-types)):

- `Citation = { durable_ref, live_locator? }` — `durable_ref` is the resolution target; `live_locator` is present only when the source has a live external origin.
- `DriftStatus = unchanged | drifted | unreachable`.

**Durable snapshots.** A `durable_ref` addresses a plain-text snapshot held **in-corpus, under version control** — the same git-tracked substrate as the concept files, not a separate store. Chosen form: the captured content is written as a plain-text artifact keyed by **content identity** (a content hash of the normalized capture), and `durable_ref` is that stable content-addressed reference. Rationale: (1) content-addressing makes `retain` idempotent for free — identical source content yields the same ref; (2) reusing the git corpus, rather than an external blob store, keeps history portable (O006, ADR004) — a corpus slice carries its own sources — and avoids a second substrate that O001/O009 could fork over (PDR001).

**Live-origin locators.** A `live_locator` is a distinct structured reference to where the source currently lives externally (e.g. a URL). It is used **only** to detect drift. It is never dereferenced by `resolve` and never substituted for `durable_ref` (ADR003). Sources with no live origin (already durable in place) carry no `live_locator`; their drift is trivially `unchanged`.

## Behavior

**`retain(source)` — authoring-time capture.**
- Normalize and snapshot the source content, write it to the durable (git-tracked) store, and return `Citation { durable_ref, live_locator? }`.
- **Idempotent per source.** Because `durable_ref` is content-addressed, retaining the same source content twice yields the same `durable_ref` and does not write a second snapshot. A caller may re-invoke freely; the corpus gains at most one snapshot per distinct captured content.
- Called by **C008 only, at authoring time** — when a claim first cites the source (ADR003). It is *not* called at intake: C006 never calls C004, keeping C004 entirely corpus-side. Consequence (accepted, ADR003): a source that rots between intake and authoring is captured only in its authoring-time state — fidelity is defined relative to what was actually cited.

**`resolve(durable_ref)` — the grounding/drift baseline.**
- Fetch and return the durable snapshot's content. This is what author-time grounding (C008) reads and what `check_drift` compares against.
- **Never** returns or falls back to the live origin. If a `durable_ref` cannot be fetched, that is a corpus-integrity fault (a snapshot that should exist under version control does not) — surfaced as an error, not silently redirected to `live_locator`.

**`check_drift(Citation)` — live-vs-snapshot comparison.**
- If the citation has no `live_locator`, return `unchanged` (nothing external to drift from).
- Otherwise fetch the current content at `live_locator` and diff it against the content of `durable_ref`:
  - fetch succeeds, content matches the snapshot → `unchanged`;
  - fetch succeeds, content differs → `drifted`;
  - **fetch fails** (network error, gone, access denied, timeout) → `unreachable`.
- `unreachable` is a first-class, non-terminal outcome, distinct from `drifted`: the live origin *may* still match, but we cannot confirm it. It is a currency input (C005) and thereby a retirement-review signal (reaching C011 through `C005.currency`), not a corpus fault — the `durable_ref` remains fully resolvable and the claim stays grounded regardless.
- Called by the **C005 currency refresh** (the periodic drift sweep). `check_drift` is a pure read: it computes a status and writes nothing.

## Relationships

Matches [INTERFACES.md § Dependency direction](../INTERFACES.md#dependency-direction-no-cycles) exactly. C004 is a stateless-ish source store; its only artifact is the in-corpus snapshots.

- **`retain`** — called by **C008** (authoring: cite a source during `integrate`/`merge`).
- **`resolve`** — called by **C008** (author-time grounding check).
- **`check_drift`** — called by **C005** (the periodic currency/drift sweep). Its result
  reaches **C011** (retirement review) through `C005.currency`, not by C011 calling C004.
- **C006 never calls C004** — capture is authoring-time, not intake (ADR003).
- C004 depends on nothing below it; it does not call other components (it does not write provenance — its callers report their own dispositions to C005).

## Decisions

- [ADR003](../drs/ADR003-citation-two-reference-model.md) — two-reference model (durable target + distinct live locator) and **authoring-time** capture; supersedes old CR001/ADR007.
- **Drift is computed here, materialized elsewhere.** C004 *computes* `DriftStatus`; C005 *materializes* it as the `drift_status` half of `CurrencyStatus`. C004 does not store currency, and C005 does not diff sources. This split was settled by locality (DECOMPOSITION.md).
- **One substrate, component-internal storage form.** The content-addressed, in-corpus snapshot form is a C004-internal choice, changeable behind this interface without touching callers.

## Success criteria

Tied to the **O001 proxy** — *the proportion of claims carrying a citation that resolves to a durable, version-controlled form*:

- **Every cited claim resolves durably.** For every citation C008 writes, `resolve(durable_ref)` returns the version-controlled snapshot the claim was grounded against — `durable_ref` is never a live locator (O001-R002).
- **Drift is detectable and honest.** Where a `live_locator` exists, `check_drift` distinguishes `unchanged`, `drifted`, and `unreachable`, so currency (C005) and retirement review (C011) act on real signal (O001-R003).
- **One substrate, not two.** The same durable snapshots serve provenance (O001) and fidelity (O009); no caller builds a second source store (PDR001). If a second store ever appears, that is the failure this component exists to prevent.
