# ADR008 - Recency materialization and per-directory log scoping

Materialize both a concept's recency (`timestamp`) and its change history (`log.md` entries) as durable, in-corpus artifacts that C007 - Currency Tracking writes on every recorded change — never computed on demand. Scope `log.md` per directory, holding entries only for concepts directly inside it, with no cascade to ancestor directories; compute any corpus-wide or subtree-wide view by scanning on demand instead of persisting one.

> **Superseded in part by [ADR011](ADR011-concept-verb-surface.md):** two things below no longer hold. (1) `record` is no longer a terminal call the writer must remember — it becomes an **internal effect** of each write verb. (2) `history(concept_id)` is redefined from the current-directory scan recorded here to a **bundle-wide filtered scan**, so it stays complete across a relocation; the post-relocation truncation noted in the third Consequence is therefore reversed (annotated inline there). The materialization decision itself — durable in-corpus `timestamp` and per-directory `log.md` with no cascade, aggregates computed on demand — is retained unchanged.

## Context

C007 - Currency Tracking fulfills recorded recency (O002-R001) and change history (O002-R002). ADR006 anticipated this decision when it specced C005 - Index & Navigation: "C007 - Currency Tracking will face the analogous materialize-vs-compute question for recency and change history." Two questions had to be settled before C007's interfaces could be specced.

First, materialize or compute. Unlike C005's cross-link graph or C006's and C010's drift findings — all recomputed fresh on every call and never persisted — the engineering README already commits C007's output to be "materialized as in-corpus artifacts so recency and history travel with the bundle," directly serving O006 portable reuse: a consumer with no Wiki tooling should be able to read a concept's last-changed time and its history straight off the files. This half of the question was effectively pre-decided by that commitment; it is recorded formally here rather than left implicit.

Second, if materialized, at what granularity. OKF §7 permits `log.md` "at any level of the hierarchy" with no requirement that entries be scoped to their directory or cascaded upward — the spec's own example is ambiguous about whether it shows a root-level or subtree-level log. C005 already set a materialize-per-directory precedent for `index.md` (ADR006), but an index entry only ever lists a directory's direct contents, while a change log's purpose (a readable history of what happened) plausibly wants a wider view at higher directories.

## Options

- **Per-directory, no cascade (chosen)**: one `log.md` per directory that has at least one recorded change, holding entries only for concepts directly inside that directory — the same locality ADR006 established for `index.md`. A wider view (a subtree or whole-bundle timeline) is not persisted; it is computed on demand by scanning every `log.md` under the requested scope. A single event writes exactly one directory's `log.md` plus the changed concept's own `timestamp` — never more, regardless of directory depth.
- **Per-directory, cascading to ancestors**: the same per-directory logs, but every entry is also appended to every ancestor directory's `log.md` up to the bundle root, so the root log is always a complete corpus history and any subdirectory's log covers exactly its subtree. Gives an always-current root timeline for free at read time, at the cost of writing as many files as the concept's directory has ancestors on every single event, and keeping all of them in sync — the kind of write fan-out and multi-artifact consistency burden the architecture's "no shared mutable state" principle cautions against.
- **Single bundle-root log only**: no per-directory logs — one `log.md` at the bundle root records every change anywhere in the corpus. Simplest write path (one file, one append), but contradicts the README's "history for each part of the corpus" wording, gives no way to scope a read to one subtree without filtering a single growing file, and abandons the locality precedent C005 already set for the sibling reserved filename.

## Decision

C007 materializes both artifacts on every `record` call: a concept's `timestamp` (frontmatter, written via C004) is set to the moment of the call, and a dated entry (via C004's log-entry helper) is appended to the `log.md` in the concept's own directory — never to any other directory's log. A directory with no recorded change has no `log.md` at all, mirroring C005's "no index.md with nothing to list" precedent. No corpus-wide or subtree-wide history artifact is persisted; `history_all(scope?)` computes that view on demand by scanning every `log.md` under `scope`, the same on-demand-aggregation posture ADR006 set for C005's graph and inbound-link queries.

## Consequences

- A single `record` call touches exactly two things: the changed concept's `timestamp` and one `log.md` file (its own directory's) — never a variable number of files scaling with directory depth, ruling out the cascade option's write fan-out.
- A whole-corpus or subtree timeline costs a bundle-wide (or scope-limited) scan on every read via `history_all` — the same complexity class as C005's `dangling_links` and `graph`, acceptable in the low-volume, single-steward envelope and optimizable later without changing C007's external contract.
- If a concept is later relocated to a different directory (a future C008 operation), its historical entries stay behind in the old directory's `log.md`, since entries are never rewritten or moved. `history(concept_id)`, scoped to the concept's current directory, will not surface pre-relocation entries — a known imprecision recorded in C007's own Edge cases, in the same spirit as C006's git-baseline imprecision note, not solved here. **— Reversed by [ADR011](ADR011-concept-verb-surface.md):** `history(concept_id)` is redefined as a bundle-wide filtered scan rather than a current-directory scan, so pre-relocation entries *do* surface and this truncation no longer holds. The physical entries still stay behind in the old directory's `log.md` (they are never moved); what changed is that the query no longer misses them.
- C007's write path is structurally identical to C005's `reindex` contract: a writer that mutates a concept calls a single method after finishing the mutation, and that method's cost and scope are local to where the concept lives, not to the whole bundle.
- The materialize-vs-compute question ADR006 opened for C007 is now closed the same direction as C005's index half (materialize what's local, compute what's aggregate) — no second precedent needed for future components that face an analogous choice.

## Affected elements

- **C007 - Currency Tracking** — the component this decision defines; backlinks here.
- **C005 - Index & Navigation** — the precedent this decision follows (ADR006); no behavior change, backlinked for symmetry.
- **C003 - Integration Authoring** — takes on the obligation to call C007's `record` after `author`, alongside its existing `reindex` obligation to C005.
- **C008 - Lifecycle & Retirement** — will take on the obligation to call `record` after a deprecation, retirement, or relocation, and will inherit the relocation known-imprecision noted above; to be specced against this contract.
- **C010 - Charter** — `declare_charter` and `revise_charter` route the charter concept's recency and history through C007 like any other concept.
