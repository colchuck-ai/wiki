# ADR011 - Concept verb surface

Reshape the concept-lifecycle operations into a small verb surface organized by the steward's **intent**, by decoupling two axes the current design fuses: the **source of material** (an admitted task, another concept) and the **lifecycle transition** (create, revise, move, delete, deprecate). Make source a *parameter* of `create`/`revise` rather than a distinct verb; express `merge` as a *composition* of existing primitives; add no named read verbs (reading a plain-text corpus is direct file access, a consumption convenience), redefining only the existing `history` read so it stays complete across a relocation; and make index, recency, log, and link-repair maintenance an **internal effect** of each write verb rather than a step the caller must remember. Supersedes the `keep-new`/`merge`-mode framing of `author` in [ADR005](ADR005-integration-authoring-provenance-and-merge.md); the paired link-repair rule the new verbs need is settled in [ADR012](ADR012-repair-to-successor-link-model.md).

> **Extended by [ADR014](ADR014-merge-composition-integrity.md):** the per-verb "atomic including its effects" guarantee below is refined so that atomicity *composes* — the `merge` composition (`revise` + `delete`) commits as one unit or not at all (one git commit or none), rather than as two independently-atomic verbs that could half-complete. The verb surface and per-verb atomicity recorded here are otherwise unchanged.

## Context

The concept-lifecycle operations grew one component at a time. C003 - Integration Authoring exposes `author(task_id)` with a `direction.mode ∈ { keep-new, merge }`; C008 - Lifecycle & Retirement exposes `deprecate`, `retire`, `relocate`; C005 - Index & Navigation exposes `reindex`; C007 - Currency Tracking exposes `record`. Each verb is named for the component that owns it, and each authoring verb is welded to *where its material comes from*: `keep-new` and `merge` are really "create from an admitted task" and "update from an admitted task."

That fusion of source with transition leaves intent-cells empty. Three gaps follow directly from it:

1. **No revise of an existing concept.** The only content-authoring entry point is `author(task_id)`, which refuses unless an admitted ingestion task exists. A steward who needs to correct a claim, fix wording, or reword a link's reason in a concept already in the corpus has no operation — yet C006 - Source Revalidation ("correcting a drifted claim… executed through C003 (re-authoring)") and C010 - Charter ("the steward corrects…") both already name a remediation path that has no interface behind it.
2. **No merge of two concepts already in the corpus.** `merge` only folds an incoming admitted task into a target. Two near-duplicate concepts discovered later (O003-RSK001's supersession scan, or the steward's eye) have no combine operation, though O003-R003 already names "merge" as a steward choice.
3. **Materialization is correctness-by-convention.** `reindex` and `record` are terminal calls every writer must remember in the right order; a bypass silently drifts the index and recency, documented as "a stewardship gap to catch via Lint" rather than a guarantee.

One smaller edge compounds it: `history(concept_id)` is scoped to a concept's *current* directory, so it silently truncates after a relocation ([ADR008](ADR008-recency-materialization-and-log-scoping.md) accepted this imprecision).

Constraints frame the choices: the corpus stays plain-text and version-control-native; integration is never automatic; the system is low-volume and single-steward. One capability is **explicitly deferred**: authoring a *new* concept from the steward's own knowledge with no external source (source-less `create`) strains O001 - Traceable provenance and O004-R001's "turn raw material into knowledge," and is left to a future PDR — so `create` remains admitted-task-gated. Revising an *existing* concept from the steward's direction is a different matter: the concept already carries provenance, and a steward correction is upkeep (O004-R003), so it stays in scope; introducing a net-new source-backed claim still comes through a task.

## Options

**How to organize the write verbs:**

- **Intent-named verbs with a source parameter (chosen)**: five transitions — `create`, `revise`, `move`, `delete`, `deprecate` — where the source of material (an admitted task, another concept, or — for `revise` of an existing concept — the steward's direction) is a *parameter* of `create`/`revise`, not a separate verb. `revise` in place keeps id, history, and inbound links and recomputes citations exactly as today's `merge` does; it simply accepts more than one source. Every distinct transition maps to exactly one verb, and the empty intent-cells (revise-existing, merge-of-existing) fill themselves.
- **Keep the component-ownership decomposition (status quo)**: `author(keep-new|merge)`, `deprecate`, `retire`, `relocate`. Rejected: it fuses source with transition, so it structurally cannot express "revise from the steward's direction" or "combine two existing concepts" without adding yet more mode flags, and it forces a steward to manufacture an ingestion task to fix a typo — fighting O004 - Low curation effort.

**Whether `merge` is a primitive or a composition:**

- **`merge` as a named composition (chosen)**: `merge(from, into)` ≡ `revise(into, source=from)` then `delete(from, replacement=into)`. Named because O003-RSK003 (redundant capture) is a first-class risk and "combine these two" is a distinct steward intent, but built entirely from primitives — no new authoring or removal machinery. Its inbound-link redirect is settled in [ADR012](ADR012-repair-to-successor-link-model.md).
- **`merge` as a fourth write primitive**: another authoring path to specify, test, and keep coherent with `revise`. Rejected: it *is* revise-then-delete; naming it as sugar keeps the primitive set minimal.

**Whether materialization is caller-orchestrated or verb-internal:**

- **Verb-internal effect (chosen)**: each write verb's contract includes stamping recency, appending the change log, regenerating the affected index, and repairing links — atomically, as part of the verb. `reindex`, `record`, `promote_source`, and `propose_links` stop being steward-facing operations and become internal effects/helpers. The component model underneath is unchanged — C005 still owns the index, C007 still owns the log — but the steward and the Agent Skill never orchestrate the effects, so index/recency drift is eliminated by construction rather than caught by Lint.
- **Keep terminal caller calls (status quo)**: the writer calls `reindex`/`record` last. Rejected: it is exactly the convention that lets a bypass drift the corpus silently; nothing about the low-volume envelope makes the footgun worth keeping.

**Whether to add named read verbs:**

- **No dedicated read verbs; fix only `history`'s completeness (chosen)**: the corpus is plain text, so reading a concept is opening its file and reading the index is opening the materialized `index.md` — the Query consumption operation does that directly, and the reads that actually *compute* (`history`/`history_all`, `inbound_links`, `dangling_links`, `graph`) already live on their owning components. The one substantive read change is redefining the existing `history(concept_id)` as a bundle-wide filtered scan so it stays complete across a `move`, superseding ADR008's per-directory scoping.
- **Name `get`/`list` read verbs over the files (rejected)**: a `get(concept)` and `list(scope)` would wrap "open the file" and "read `index.md`" in component interfaces that add no capability — everything `get` would bundle is already in the concept file except inbound links (already `C005.inbound_links`), and `list` just returns the artifact `reindex` already wrote. Naming them contradicts the architecture's own line that querying is a consumption convenience, not a curation operation. Rejected as surface without capability.

## Decision

Adopt an intent-named concept verb surface. The write primitives are `create`, `revise`, `move`, `delete`, and `deprecate`; `merge` is a named composition of `revise` + `delete`. Material source is a parameter of `create`/`revise`: for `create`, an admitted task (today's `keep-new`); for `revise`, an admitted task (today's `merge`), another concept (feeding `merge`), or the steward's direction (correcting an existing concept, O004-R003). Only source-less `create` — a new concept authored from the steward's own knowledge with no task — is deferred to a future PDR, so `create` stays admitted-task-gated. `revise` folds a source into an existing concept in place — preserving id, history, and inbound links and recomputing citations exactly as ADR005's merge reconciliation specifies. No dedicated read verbs are added: the Query consumption operation reads the plain-text concepts and the materialized index directly, and the reads that compute stay on their components. The one read change is redefining `history(concept_id)` as a bundle-wide filtered scan so it stays complete across a `move`.

Every write verb is atomic including its effects: it stamps recency, appends the change log, regenerates the affected index, and repairs links as part of the verb, through the same component owners as today (C005, C007) and the same C004 write face — `reindex`, `record`, `promote_source`, and `propose_links` are internal, not steward-facing operations. No OKF structural literal moves out of C004, so ADR001's one-owner invariant holds.

This supersedes ADR005's framing of authoring as `author` with a `keep-new`/`merge` mode; ADR005's provenance model (cite-by-path assets under `/references/`), citation granularity (concept-level default, inline per-claim on multi-source), and merge reconciliation (fold in place, preserve id, recompute citations) are retained unchanged and now realized by `create` and `revise`.

## Consequences

- The two intent gaps close: `revise(concept, source=steward-direction)` gives the steward a way to correct or refine an existing concept — the interface C006 and C010 already reference for correcting content — and `revise(concept, source=another-concept)` plus `delete` gives `merge` of two existing concepts (O003-R003, O003-RSK003). Only source-less *creation* remains deferred; correcting an existing, already-provenanced concept does not wait on the PDR.
- Index and recency can no longer drift from a forgotten terminal call — the write verb owns the effect. The isolation the architecture prizes ("no shared mutable state," self-contained components) is preserved *underneath* the facade; what changes is only the surface the steward and Agent Skill see.
- `create` remaining admitted-task-gated means O001 - Traceable provenance and the never-automatic principle are unchanged by this decision; the provenance-straining path is quarantined behind the deferred PDR rather than shipped implicitly.
- `history` becomes always-complete, superseding ADR008's accepted post-relocation truncation; the cost is a bundle-wide log scan per query, acceptable in the low-volume, single-steward envelope and consistent with `history_all` already scanning that way.
- Naming shifts ripple into the component specs: `author` → `create` + `revise`, `relocate` → `move`, `retire` → `delete` (`deprecate` unchanged). These are the concrete edits the paired Change Records carry; this ADR fixes the decision, not the prose.
- The Agent Skill's operation vocabulary (Ingest, Survey, Lint, Query) re-expresses in these verbs: `create`/`revise`/`merge` realize Ingest's write half, `move`/`delete`/`deprecate` are directed from Survey findings, and Query stays a read-only consumption convenience over the files (plus the computed reads `history`, `inbound_links`, `graph`) — not a named curation verb.

## Affected elements

- **C003 - Integration Authoring** — `author(keep-new|merge)` becomes `create` and `revise` (source-parameterized), and hosts the `merge` composition; the source-promotion and citation helpers become internal effects. Backlinks here.
- **C008 - Lifecycle & Retirement** — `retire` → `delete`, `relocate` → `move`, `deprecate` unchanged; link-repair becomes a verb-internal effect per ADR012. Backlinks here.
- **C005 - Index & Navigation** — `reindex` becomes an internal effect of the write verbs; no `list` read verb is added (the materialized `index.md` is read directly). Backlinks here.
- **C007 - Currency Tracking** — `record` becomes an internal effect; `history(concept_id)` is redefined bundle-wide and complete, superseding ADR008's per-directory scoping. Backlinks here.
- **Wiki Curation Architecture** (`docs/engineering/README.md`) — the Agent-Skill operation vocabulary re-expresses in the verb surface and the "materialization is a caller step" framing is removed. Backlinks here.
