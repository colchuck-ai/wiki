# C002 - Triage

Adjudicates each queued item before it enters the corpus, weighing it against several admission criteria — the charter's declared scope, overlap with existing concepts, and whether it is substantial enough to keep — and capturing the steward's disposition, flagging anything that fails a criterion for judgment rather than admitting it automatically.

C002 is the curation pipeline's **adjudication gate**, sitting between intake (C001 - Ingestion Queue) and authoring (C003 - Integration Authoring). It makes no change to the corpus: it reads the un-triaged backlog and the existing corpus, evaluates three admission criteria, **recommends** a disposition with rationale and evidence, and — once the steward confirms or overrides — records the decision back onto C001's task. It admits nothing automatically (integration is never automatic); it never writes a concept, promotes a snapshot, or merges anything (C003), and never removes existing content (C008). It fulfills the duplication check (O003-R003), the significance bar (O003-R004), and scope-aware triage (O008-R002), and realizes the adjudication step of the product's **Ingest** operation surfaced through the curation Agent Skill.

## Data model

C002 owns **no artifact of its own**. Its output extends the **ingestion task** C001 already owns (the intake→triage interface artifact), materializing the disposition schema C001 reserves for triage. C002 writes only these staging fields; C001 continues to own the task's intake fields and its non-OKF record format.

- **`status`** — advanced from `untriaged` to one of `admitted` / `rejected` / `held`, the values C001's schema defines. Setting it is the only status transition C002 performs, and only through `record_disposition`.
- **`triage`** — the disposition block, appended to the task record:
  - **`findings`** — one per criterion, each a `{ verdict, rationale, evidence }`:
    - **`scope`** — `verdict ∈ { in-scope, out-of-scope, marginal, not-evaluated }`, judged against `C010.get_charter()`; `not-evaluated` when no charter is declared.
    - **`overlap`** — `verdict ∈ { none, overlaps }` with `candidates: [{ concept-id, reason, confidence }]` — the existing concepts the item may duplicate, each a concrete merge target (see Behavior).
    - **`significance`** — `verdict ∈ { substantial, insubstantial }`, the judgment of whether the material is worth keeping.
  - **`recommendation`** — the tool's recommended disposition (`admit` / `reject` / `hold`) with a combined rationale synthesized from the findings. Advisory only.
  - **`decision`** — the steward's confirmed disposition. May equal or override the recommendation; the recorded decision, not the recommendation, is authoritative.
  - **`direction`** — present only on an admitted item, the reconciliation steer handed to C003: `{ mode ∈ { merge, keep-new }, target-concept-id? , note? }`. `target-concept-id` is required when `mode = merge`; `note` carries any distillation hint. (`discard` is expressed as a `rejected` status, not a direction.)
  - **`triaged-at`** — ISO-8601 time the disposition was recorded; **`triaged-by`** — optional free-form note.

**`held`** means adjudicated but not resolved — the item has been looked at and is neither admitted nor discarded (e.g. awaiting material or a later scope call). A held task drops out of the un-triaged backlog but remains a live task the steward can re-triage.

## Interfaces

C002 exposes an in-process face. It reads corpus concepts through C004's parse helper and reads the backlog and charter through C001 and C010; every write it performs is confined to C001's staging task record. It never writes to the corpus.

- **`triage(task_id) → recommendation`** — evaluates all three criteria for one queued task against the current corpus and charter, and returns the recommended disposition with per-criterion findings and overlap candidates. **Read-only** — it records nothing. This is the "surface the work" half of the recommend-and-confirm posture, driven per backlog item by the Ingest operation.
- **`assess_overlap(task) → candidate[]`** — the overlap scan on its own: exact matches by C001's `content-hash` plus a semantic scan of the corpus, returning candidate concepts with a reason and confidence. Exposed so C003 or the steward can re-resolve candidates without re-running the full triage.
- **`record_disposition(task_id, decision, [direction]) → task`** — writes the steward's confirmed disposition and advances `status`. On `admit`, records `direction` (with a `target-concept-id` when merging) for C003. On `reject`, calls `C001.discard(task)`. On `hold`, sets `status = held`. This is the only call that changes a task's status.

## Behavior

- **Recommend, never auto-admit.** `triage` only evaluates and recommends; only `record_disposition` changes a task's status, and only with a steward decision. No path advances a task to `admitted` without the steward confirming. This is the concrete guard that keeps admission from becoming automatic.
- **All three criteria, every item.** Scope, overlap, and significance are always evaluated. A fail on any one does not itself reject the item — it flags the item for the steward's judgment with its rationale. The recommendation weighs the findings together (e.g. out-of-scope or insubstantial → recommend `reject` or `hold`; clean → recommend `admit`).
- **Scope degrades cleanly.** When `get_charter()` returns none, the scope finding is `not-evaluated` and overlap and significance still run — matching C010's optional-charter contract. The absence of a charter never blocks triage.
- **Overlap is exact + semantic.** Exact duplicates are caught by comparing C001's intake `content-hash`; overlap beyond identical bytes is found by an agent scan that reads corpus concepts through C004's parse helper and surfaces semantically-overlapping concepts as candidates. C002 surfaces candidates as concrete targets; it never merges — the steward chooses merge / keep-new / discard, and C003 executes any merge (O003-R003).
- **Significance is advisory.** Insubstantial material is flagged for the steward, never auto-dropped (O003-R004). The steward may admit something the tool judged insubstantial; the recorded decision governs.
- **Recommend/record, don't author.** C002 writes only the staging task's disposition. It never writes a concept, promotes a snapshot, or applies a deprecation — those are C003 and C008. A rejection is executed as `C001.discard`, leaving the corpus untouched.
- **Re-triage is idempotent against current state.** Re-running `triage` on a task re-evaluates against the corpus and charter as they stand now (e.g. after a charter revision or after related concepts changed), yielding a fresh recommendation. Recording a new disposition supersedes the prior one on the same task.

## Edge cases

- **No charter declared**: scope finding is `not-evaluated`; overlap and significance still run; the recommendation notes scope was not assessed. Consistent with C010 and ADR003.
- **Exact duplicate of existing content** (`content-hash` matches a cited source or another task): surfaced as a high-confidence overlap candidate and recommended accordingly — still surfaced for the steward to merge / keep / discard, never silently discarded.
- **Overlap with several concepts**: all candidates are surfaced, ranked by confidence; the steward picks one merge `target-concept-id`, which is what C003 receives.
- **Marginal scope** (item legitimately spans the boundary): scope verdict `marginal` with rationale; the steward decides, mirroring C010's marginal-case handling. C002 does not resolve the marginal call itself.
- **Capture-failed task** (C001 flagged the origin unreachable, no material held): C002 triages on the submitter's note and `origin` but marks that the material itself is unavailable, so overlap and significance are low-confidence; the typical recommendation is `hold` pending material, or the steward discards.
- **In-scope but insubstantial**: flagged `insubstantial`; the steward may still admit — the recommendation is advisory, not a gate.
- **Recording on an already-resolved task**: a `rejected` task's artifacts are gone (discarded), so recording is refused; an `admitted` or `held` task may be re-triaged and its disposition revised.
- **Corpus changes between admission and authoring**, making the recorded overlap stale: the recorded `direction` is a point-in-time steer C003 acts on; if it has gone stale, the steward re-triages to recompute before authoring.

## Relationships

- **C001 - Ingestion Queue**: reads `list_backlog`, `get_task`, and `durable_ref`; writes the disposition block and advances `status` in the task record per C001's schema; calls `discard` on rejection. The task file is the single artifact carried through intake → triage → authoring.
- **C010 - Charter**: reads `get_charter()` for the scope criterion, adjudicating against the single scope authority at intake timing, and drops the criterion cleanly when no charter exists. C010 owns the charter; C002 is a reader (ADR003).
- **C004 - OKF Conformance**: uses the parse helper to read corpus concepts (frontmatter and body) during the overlap scan; encodes no OKF structural literal itself and writes no concept, so the one-owner invariant (ADR001) holds with C002 present.
- **C003 - Integration Authoring**: consumes an admitted task's disposition and `direction` — authoring the concept and executing the steward-chosen merge against the named `target-concept-id`. C002 decides *whether and what*; C003 does *how*.
- **C005 - Index & Navigation** (future): overlap detection is self-contained in C002 today (a scan via C004). If C005 later offers candidate retrieval from its index/graph, C002 may delegate the candidate search as an optimization without changing this contract — it is not a dependency (ADR004).
- **Boundary**: C002 owns adjudication and disposition capture. It does not intake or durably hold material (C001), author or merge concepts (C003), define scope (C010 owns the charter), define the format (C004), or retire existing concepts (C008). It writes nothing to the corpus.

## Success criteria

- **Nothing auto-admitted**: no path sets `status = admitted` without a steward decision through `record_disposition` — verifiable by driving `triage` and confirming the task's status is unchanged until a disposition is recorded.
- **Three criteria evaluated**: triage of a task with a charter yields scope, overlap, and significance findings; with no charter, overlap and significance are still present and scope is `not-evaluated`.
- **Overlap beyond exact bytes**: a semantically-overlapping but not byte-identical incoming item surfaces the existing concept as a candidate — testable with a paraphrase of an existing concept.
- **Concrete merge target**: when overlap is found and the steward chooses merge, the recorded `direction` names a specific existing `concept-id` C003 can act on.
- **No corpus writes**: `triage` and `record_disposition` leave the corpus bundle byte-for-byte unchanged; every write lands in the staging task record — verifiable by running triage and diffing the corpus.
- **Clean rejection**: recording a `reject` calls `C001.discard` and leaves the corpus untouched.
- **Scope degrades cleanly**: with no charter, triage still runs and recommends on overlap and significance alone.
- **Recommendation is advisory**: the steward can override any recommended disposition, and the recorded `decision` reflects the override, not the recommendation.
- **One-owner invariant preserved**: C002 contains no OKF structural literal (it reads concepts only through C004) — ADR001's grep-able check stays clean with C002 present.

## Notes

- Triage is surfaced through the product's **Ingest** operation (engineering README, Technology Choices): the tool presents the recommendation and its evidence; the steward confirms or overrides. Direct steward operations drive it one backlog item at a time.
- The overlap scan is an agent pass whose cost scales with corpus size — acceptable in the low-volume, single-steward envelope, the same posture C010's reconciliation takes.
- The recommend-and-confirm posture, recording the disposition and downstream reconciliation direction by extending C001's task record, and self-contained (C005-independent) semantic overlap detection are captured in [ADR004](../drs/ADR004-triage-disposition-model.md).

## See Also

### Architectural Decision Records

- [ADR004 - Triage disposition model](../drs/ADR004-triage-disposition-model.md)
