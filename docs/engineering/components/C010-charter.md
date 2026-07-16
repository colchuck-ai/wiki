# C010 - Charter

Holds the steward's declaration of the corpus's purpose and scope as an optional, persistent anchor; applies a core-purpose revision in place; and periodically reconciles the corpus against the declared scope to surface content that has drifted out of it.

C010 is the corpus's **single scope authority**. It owns one artifact — the charter — and the periodic sweep that measures the existing corpus against it. It realizes the architecture principle that both adjudications of scope resolve against one declaration: per-item triage of incoming material at intake (C002 - Triage) and periodic reconciliation of what is already in the corpus (C010) read the same charter, so scope cannot fork. C010 detects drift; it does not repair it — retiring an out-of-scope concept is the steward's judgment, executed by C008 - Lifecycle & Retirement. The charter is stored as an ordinary OKF concept (`type: charter`) so it travels with the bundle and is read and written only through C004's helpers; C010 encodes no OKF structural knowledge (see [ADR003](../drs/ADR003-charter-as-in-corpus-concept.md)).

## Data model

C010 owns exactly one artifact, the **charter**, and no persisted reconciliation state.

- **Charter concept.** A single OKF concept identified by `type: charter`, authored and parsed through C004 like any other concept. C010 owns only its body semantics; C004 owns the surrounding concept format unchanged.
  - `type: charter` in frontmatter — the discovery key. This is a C010 semantic convention, not a value C004 registers or constrains (C004 tolerates it as any other unknown `type`).
  - `# Purpose` — required prose: what this corpus is for. This is the primary basis for every scope judgment.
  - `# In scope` — optional bulleted boundaries that sharpen what belongs.
  - `# Out of scope` — optional bulleted boundaries that sharpen what does not.
  - Recency and change history are the concept's own, maintained by C007 - Currency Tracking and git; C010 adds no versioning of its own.
- **Drift finding** (transient, not written to the corpus). One per concept reconciliation judges outside the declared scope: `{ concept-id, verdict: out-of-scope, rationale }`. Surfaced to the steward through the Survey operation; never persisted, mirroring C006's detect-don't-repair report.

By convention the charter is authored at the bundle root (`charter.md`, slug via C004), but it is discovered **by type**, so its path is not load-bearing.

## Interfaces

C010 exposes an in-process face. Reads and writes of the charter concept go through C004; the corpus is otherwise untouched (reconciliation is read-only).

- **`declare_charter(purpose, [in_scope], [out_of_scope]) → charter`** — creates the single charter concept via C004's authoring helpers. Refuses if a charter already exists (use `revise_charter`); the charter is optional, so this is only ever called when the steward chooses to declare one.
- **`get_charter() → charter | none`** — returns the parsed charter, or `none` when none is declared. The single read point for the declared scope, consumed by C002 - Triage and C009 - Coverage Review so both adjudicate against the same authority.
- **`revise_charter(changes)`** — edits the purpose or the optional boundaries in place through C004's serialize helpers, preserving the concept's identity and its C007/git history. Used for the core-purpose revision O008-R001 requires.
- **`reconcile() → drift_finding[]`** — the periodic whole-corpus sweep: judges each concept against the current charter and returns those outside its declared scope as candidates, each with a rationale. Read-only with respect to the corpus. Returns empty when no charter is declared.

## Behavior

- **Single authority, two timings.** The charter C010 owns is the one artifact scope is judged against, both at intake (per-item, C002) and periodically over the whole corpus (C010's `reconcile`). C010 exposes it through `get_charter`; it never keeps a second copy.
- **Detect, don't repair.** `reconcile` surfaces out-of-scope concepts as candidates only. Deprecating or removing one is the steward's judgment, executed by C008; revising the charter to include it is the alternative. C010 never mutates a concept it flags. This is the guard for O008-RSK001 (input-driven drift) applied to the existing corpus.
- **Optional, degrades cleanly.** With no charter declared, `get_charter` returns `none`, `reconcile` returns empty, and C002 simply drops its scope criterion while still checking duplication and significance. The system is fully operational without a charter (O008-R001's optionality).
- **Revisable in place, no side effects.** A revision edits the one charter concept; it does not purge or re-triage anything. Its only effect is that subsequent triage and reconciliation adjudicate against the new scope. Because there is no persisted reconciliation result, revision needs no invalidation machinery; the steward re-runs reconciliation when they want the corpus re-measured against the new purpose.
- **Charter is meta.** Being a concept, the charter appears in the index and graph, but `reconcile` never reports the charter itself as out-of-scope, and it is excluded from coverage gaps (C009) and retirement candidacy (C008). It describes the corpus; it is not corpus content to be adjudicated.

## Edge cases

- **No charter declared**: `get_charter → none`; `reconcile → []`; C002 runs without a scope criterion. Declaration is offered but never forced.
- **Multiple `type: charter` concepts** (steward error): a conflict — C010 surfaces it for resolution and adjudicates against **none** until it is resolved, rather than silently choosing one and giving scope two authorities.
- **Charter revised so once-in-scope content is now out of scope**: the next `reconcile` surfaces those concepts as drift candidates; nothing is retired automatically.
- **Purpose empty or placeholder**: a charter with no meaningful `# Purpose` cannot adjudicate scope; C010 surfaces it for the steward rather than emitting confident-but-baseless drift findings. (A content judgment C010 owns — not a C004 structural finding.)
- **Corpus concept that legitimately spans the scope boundary**: `reconcile` surfaces it with its rationale for the steward's judgment; C010 does not decide the marginal case itself.

## Relationships

- **C004 - OKF Conformance**: the charter is an OKF concept read and written only through C004's parse/serialize helpers; C010 relies on C004's tolerance of the unknown `type: charter` and encodes no OKF structural literal. C010 owns the charter's body semantics (Purpose / In scope / Out of scope); C004 owns the concept format.
- **C002 - Triage**: reads `get_charter` to adjudicate incoming material against the declared scope (O008-R002). C010 owns the charter; C002 is a reader. Same authority, intake timing.
- **C009 - Coverage Review**: reads the declared scope as one input to its sourcing agenda (O007-R002), and excludes the charter concept itself from coverage analysis.
- **C008 - Lifecycle & Retirement**: when the steward chooses to act on an out-of-scope drift candidate, C008 executes either mechanism — `deprecate` (marking it in place with a reason) or `delete` (removing it outright, with its inbound links auto-repaired to their successor) — the steward's free choice either way. C010 detects the candidate; C008 executes. C010 excludes the charter concept from retirement candidacy.
- **C007 - Currency Tracking**: the charter concept, like any concept, gets recency and change history from C007; charter revision-in-place inherits its version trail this way.
- **C005 - Index & Navigation**: indexes the charter as a concept; any distinct presentation of it as meta is C005's concern, not C010's.
- **Boundary**: C010 owns the charter artifact, its declaration and in-place revision, and periodic scope reconciliation. It does not triage incoming items (C002), delete or deprecate concepts (C008), fill or source gaps (C009), or define the OKF format (C004).

## Success criteria

- **Single authority**: exactly one charter is resolvable, and C002's per-item triage and C010's reconciliation both adjudicate against the artifact returned by `get_charter` — verifiable by confirming neither holds a second scope copy.
- **Optional operation**: with no charter, `declare_charter` is available, `reconcile` returns empty, and triage still runs its non-scope criteria — the system never requires a charter to function.
- **Revisable in place**: revising the charter preserves the concept's identity and history (git/C007) rather than creating a new artifact — testable by revising and confirming the same concept, with prior versions in history.
- **Detect, don't repair**: `reconcile` never writes to the corpus; its entire output is candidates for the steward — verifiable by running it and confirming the corpus is byte-for-byte unchanged.
- **Format ownership preserved**: the charter is read and written only through C004, and C010 contains no OKF structural literal — the one-owner invariant (ADR001) still greps clean with C010 present.
- **Meta exclusion**: `reconcile` never flags the charter concept itself, and it never appears as a coverage gap or retirement candidate.
- **At most one authority**: multiple `type: charter` concepts produce a surfaced conflict, not a silent pick.

## Notes

- `type: charter` is a C010 semantic convention; C004 neither registers nor constrains `type` values (consistent with C004's data model) and tolerates it as any unknown type.
- Reconciliation is a whole-corpus agent pass; its cost scales with corpus size, acceptable in the low-volume, single-steward envelope. It is surfaced through the product's **Survey** operation, while charter declaration and revision are direct steward operations through the curation Agent Skill.
- The storage form (`type: charter` in-corpus concept), the single-authority-at-two-timings design, and the detect-don't-repair posture are captured in [ADR003](../drs/ADR003-charter-as-in-corpus-concept.md).

## See Also

### Architectural Decision Records

- [ADR003 - Charter as an in-corpus concept and single scope authority](../drs/ADR003-charter-as-in-corpus-concept.md)
