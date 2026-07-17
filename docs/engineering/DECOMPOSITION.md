# Component decomposition

How the 12 components were derived, what each owns, and how the derived set was
checked against the archived tree. The frozen call surfaces are in
[INTERFACES.md](INTERFACES.md); the full trace is in
[REQUIREMENT-MAP.md](REQUIREMENT-MAP.md).

## Derivation method (blind-first)

Derived from the frozen product tree **without** consulting the old engineering tree
(anchoring is the main failure mode), using the deep-module lens:

1. **List capabilities** the requirements demand (from `docs/product/README.md` only).
2. **Group by the state/artifact each owns** — what does a cluster exclusively read
   and write? This is the locality cut.
3. **Cut seams** where a small interface hides deep behavior; aim for **one owner per
   artifact**, so "no shared mutable state" either falls out or is flagged.
4. **Cross-check against the material lifecycle** (intake → triage → author → maintain
   → discover) — no stage orphaned.
5. **Only then** diff against the old set as a coverage checklist (see
   [Coverage diff](#coverage-diff-phase-2-step-5)).

Four decomposition forks were put to the steward and resolved to the recommended
shape (sole-writer corpus; one non-integrated-material owner; separate Charter +
Governance; Lifecycle vs Coverage&Scope split). Three were settled by locality
(drift computed in C004 / materialized in C005; Currency&Provenance a real owner;
Question Retrieval its own component).

## The single source of truth

The **concept corpus** — plain-text, OKF-conformant files in version control — is the
one source of truth. [C008 Integration](#c008--integration) is its only writer. Every
other view is a *function of the corpus* (+ the provenance log): the index (C009),
currency (C005), drift status (C004→C005). This is what makes "no shared mutable
state" real rather than aspirational.

## Shared types

Referenced by [INTERFACES.md](INTERFACES.md). Kept deliberately small.

| Type | Shape |
|---|---|
| `Charter` | `{ purpose, [ScopeArea] }` — required; may be provisional |
| `ProposedAction` | `{ kind, target, actor_intent, context }` |
| `Decision` | `act \| escalate(reason)` |
| `AttentionItem` / `AttentionId` | a composed steward-inbox entry / its id |
| `StewardDecision` | the steward's adjudication of an escalated item |
| `ConformanceReport` | `ok \| [violation]` |
| `Citation` | `{ durable_ref, live_locator? }` |
| `DriftStatus` | `unchanged \| drifted \| unreachable` |
| `Disposition` | `{ target, outcome, actor, envelope_version, time, detail }` (provenance) |
| `TriageOutcome` | `integrate \| hold_aside \| merge(into) \| escalate` |
| `CurrencyStatus` | `{ recency, drift_status }` |
| `Link` | `{ target, reason }` |
| `RawMaterial` / `IntakeId` / `HeldId` | submitted material / its ids |
| `ConceptId` | a concept's stable identity |

`actor ∈ { wiki_auto, steward }`; `outcome ∈ { integrated, held_aside, merged,
deprecated, superseded, relocated, flagged }`.

## The components

Each entry: **role**, the **artifact it owns** (or "stateless"), why it is **deep**,
who it **depends on / is driven by**, and the **requirements** it owns. Full specs are
Phase 3 (`components/C0xx-*.md`, not yet written); this is the seed.

### C001 — Charter
- **Owns:** the charter policy anchor (purpose + scope areas), in-corpus, revisable.
- **Deep:** the single scope referent every scope/significance/dedup/coverage judgment
  reads; hides charter representation behind `read`/`scope_areas`.
- **Depends on:** nothing (foundation). **Driven by:** steward (declare/revise).
- **Owns reqs:** O008-R001. **Note:** now **required** (PDR004) — see [drop #4](#consciously-dropped).

### C002 — Governance
- **Owns:** the delegation envelope + the single steward attention surface.
- **Deep:** two internal seams — the **envelope evaluator** (`evaluate → act | escalate`,
  the only place autonomy is decided) and the **composed attention surface** (dedups/rolls
  up escalations by subject so steward effort stays flat). Reads C001.
- **Driven by:** every autonomous capability (C007/C008/C011/C012) funnels through it.
- **Owns reqs:** O004-R004; O009-R003 (attention surface for distillation reviews).
- See [ADR002](drs/ADR002-manage-by-exception-seam.md).

### C003 — OKF Conformance
- **Owns:** nothing persistent — **format knowledge** (stateless, deep by behavior).
- **Deep:** the *single* place OKF structural + naming rules live; `apply`/`validate`
  against a pinned external spec, so a format bump is a one-component change.
- **Driven by:** C008 (apply/validate), C009/C012 (audit).
- **Owns reqs:** O004-R002, O006-R002.

### C004 — Source Retention
- **Owns:** durable source snapshots + distinct live-origin locators; drift computation.
- **Deep:** hides "how sources are held durably" and how live-vs-snapshot drift is
  detected behind `retain`/`resolve`/`check_drift`. One substrate for O001 + O009.
- **Driven by:** C008 (retain at authoring time — *not* intake), C005 (drift refresh),
  C011. **Owns reqs:** O001-R002, O001-R003. See [ADR003](drs/ADR003-citation-two-reference-model.md).

### C005 — Currency & Provenance
- **Owns:** the append-only decision-provenance log (plain-text, in-corpus, scoped —
  history travels with the material), drift-status materialization, per-concept currency.
- **Deep:** everyone *reports* dispositions through `record`; the physical materialization
  (inline vs scoped log) is hidden. Single writer of the provenance store.
- **Driven by:** all mutators (report), consumers of `history`/`recency`/`currency`.
- **Owns reqs:** O002-R001, O002-R002. See [ADR004](drs/ADR004-machine-readable-signal-placement.md).

### C006 — Intake & Held-Aside
- **Owns:** all non-integrated material — the producer-agnostic raw queue **and** the
  recoverable held-aside store.
- **Deep:** hides the staging + recoverable-quarantine lifecycle behind
  `submit`/`hold_aside`/`restore`/`list_held_aside`. Nothing destroyed here.
- **Driven by:** producers (submit), C007 (route/hold), C012 (reads held-aside count).
- **Owns reqs:** O007-R001, O007-R003.

### C007 — Triage
- **Owns:** nothing persistent — the **disposition decision** (stateless, deep by
  judgment): scope (vs C001) + duplication (vs C009) + significance (vs C001).
- **Deep:** a large amount of judgment behind one `triage(item) -> TriageOutcome` call,
  which is also its whole test surface. Executes nothing; routes.
- **Driven by:** C006 (`next`). **Drives:** C006/C008/C002. **Owns reqs:** O003-R003,
  O003-R004, O008-R002.

### C008 — Integration
- **Owns:** the concept corpus — **the sole writer.** Intent-verb surface
  (integrate/merge/deprecate/supersede/relocate/repair_links/flag).
- **Deep:** a lot of behavior (OKF-conformant editing, citation, author-time grounding
  check as an internal seam, cross-link authoring, provenance stamping, atomic commit)
  behind a handful of intent verbs. Every verb commits concept + provenance atomically.
- **Depends on:** C002, C003, C004, C005, C009. **Driven by:** C007, C011, C012 (intent).
- **Owns reqs:** O001-R001, O003-R001, O004-R001, O005-R002 (write), O006-R001/R003,
  O009-R001, O009-R002. See [ADR001](drs/ADR001-integration-sole-corpus-writer.md).

### C009 — Discovery & Index
- **Owns:** the navigable index + reason-annotated cross-link graph (materialized from
  the corpus; read-only over it).
- **Deep:** hides materialize-vs-compute (a Phase-3 C009 choice) behind
  `index`/`neighbors`/`inbound_links`/`dangling`. Machine-readable-first.
- **Driven by:** consumers; C011 (`inbound_links`), C012 (`index`/`dangling`).
- **Owns reqs:** O005-R001, O005-R002, O005-R003.

### C010 — Question Retrieval
- **Owns:** nothing persistent (or a derived index, hidden) — free-form question →
  concepts.
- **Deep:** the whole heuristic mechanism (agent-over-corpus vs built search) is hidden
  behind `retrieve(question)`; open by design. Read-only over the corpus.
- **Owns reqs:** O005-R004. See [ADR005](drs/ADR005-question-retrieval-mechanism.md).

### C011 — Lifecycle & Retirement
- **Owns:** nothing persistent — the **retirement/relocation decision** (stateless, deep).
- **Deep:** surfaces candidates from currency/drift/supersession, then drives C008's
  intent verbs (deprecate/relocate/repair_links) within the envelope. Owns the
  referential-integrity-sensitive moves (retire **and** relocate).
- **Driven by:** schedule/steward. **Drives:** C002, C005, C008, C009.
- **Owns reqs:** O003-R002, O004-R003 (surface half), O005-R003 (repair).

### C012 — Coverage & Scope Review
- **Owns:** nothing persistent — read-only **charter-vs-corpus surveys**.
- **Deep:** `coverage` (charter vs index + held-aside + dangling targets) and `scope`
  (charter vs corpus → drift-out); mechanism open (metric vs agent-scan).
- **Drives:** C002 (escalate), C008 (`flag`). **Owns reqs:** O004-R003 (surface half),
  O007-R002, O008-R003.

## Coverage diff (Phase 2 step 5)

The archived old tree (11 components, 16 ADRs) was read *after* the blind derivation,
purely as a coverage checklist. **Every old capability lands somewhere** in the new
set, with the conscious exceptions below. The blind derivation and the old tree
converged closely (sole-writer verb surface, single conformance owner, grounding as an
internal seam, repair-to-successor) — arrived at independently, which is a good sign.

Old → new (condensed): oldC001 Ingestion → C006 (queue) + C004 (durable holding) +
C007 (content-hash dedup); oldC002 Triage → C007 + C006 (held) + C002 (confirm);
oldC003 Integration Authoring → C008 + C003 + C004; oldC004 OKF → C003; oldC005 Index →
C009; oldC006 Source Revalidation → C004 (compute) + C005 (materialize); oldC007
Currency → C005; oldC008 Lifecycle → C011 (decide) + C008 (execute); oldC009 Coverage →
C012; oldC010 Charter → C001 + C012 (reconcile); oldC011 Curation Operations → see
drop #1.

### Consciously dropped

Logged with reason, per Phase 2 step 5:

1. **Cross-face aggregation richness** (old C011 / old ADR016): conflict-pairing
   (expand-vs-retire), a face→verb routing table, and "Lint" as a named operation. The
   **product** motivates only a *composed/deduplicated* attention surface (O004 flat
   effort), which is absorbed into **C002**. The elaborate aggregator is old-tree-specific
   gold-plating; the clean-room declines to transcribe it. Conflict-pairing may return as
   a **C002 Phase-3 refinement** if steward experience demands — it is not load-bearing
   architecture. *(This is the one place the blind set and the old set genuinely diverge;
   flagged to the steward.)*
2. **"Lint" as a single operation:** its constituents survive (validate → C003, dangling →
   C009, freshness → C009/C005). Only the bundled name is dropped.
3. **Charter-less degradation path** (old ADR003/013): the old system ran without a
   charter (triage dropped its scope criterion, coverage narrowed). PDR004 makes the
   charter **required and foundational**, so this degrade-cleanly path is deliberately
   removed — a direct product consequence, not a regression.
4. **Per-item review of every action:** removed by manage-by-exception (PDR003); the
   steward audits composed exceptions, not each action.

### Old-ADR checklist outcome

Old ADR001 (single conformance) → C003 boundary. ADR004/005/009/010/012/015 → settled
in components (C007/C008/C011). ADR014 atomicity → folded into **ADR001** (per-verb
one-commit-or-none), its cross-component framing obsolete under sole-writer. The
still-live decisions were consolidated into **five** new ADRs (ADR001–005); intake
snapshot-timing → ADR003, history portability/placement → ADR004, index
materialize-vs-compute → deferred to the C009 Phase-3 spec (component-internal, behind
C009's interface).
