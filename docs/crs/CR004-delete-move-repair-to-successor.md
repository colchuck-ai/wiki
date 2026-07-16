# CR004 - Delete, move, and the repair-to-successor link model

Updated C008 - Lifecycle & Retirement: `retire` becomes `delete` (with an optional, resolve-checked replacement), `relocate` becomes `move`, and the two per-operation link-repair behaviors are unified into a single **repair-to-successor** rule. Review candidates are narrowed to the `delete` case, so `move` and `merge` raise none.

## Change

Before: C008 exposed `retire`, which deleted a concept and stripped every inbound link to plain text, and `relocate`, which moved a concept and retargeted every inbound link to the new path — two separate repair behaviors. A hard removal had no way to *redirect* inbound links to a successor even when a genuine one existed, and `retire` always surfaced its referrers as `review_candidates`. There was no operation to combine two existing concepts, so a merge that folded one concept into another and wanted the folded-away concept's links to follow had nowhere to send them.

After: `retire` is renamed `delete` and gains an optional `replacement_concept_id` (refused unless it resolves to an existing concept); `relocate` is renamed `move`; `deprecate` is unchanged. All three repairs follow one rule — inbound links are repaired to the departing concept's **successor**: the new path (`move`), the replacement or merge target (`delete` with a replacement, and the `delete` step of a `merge`), or plain text when there is no successor (`delete` with none). `review_candidates` are surfaced only for `delete` (content actually removed); `move` and a `merge`'s `delete` step produce none, because the content demonstrably survives at a known location. This makes the `merge` composition C003 now offers (`revise` + `C008.delete(from, replacement=into)`) preserve navigability by redirecting the folded-away concept's inbound links to the merge target. C008's Data model, Interfaces, Behavior, Edge cases, Relationships, and Success criteria are updated.

## Rationale

Once [ADR011](../engineering/drs/ADR011-concept-verb-surface.md) introduced `merge` of two existing concepts and `delete` with a replacement, the repair question stopped being "retire or relocate" and became "does the departing concept have a successor to point at?" — which the old two-case split could not express. Keeping strip-to-plain-text for every deletion would have thrown away a valid redirect on a merge, dead-ending readers who followed an inbound link even though the destination was known, working against O005 - Fast discovery exactly where the destination is certain. Collapsing the behaviors into one successor-directed rule closes that gap and a latent one in the old model (a `retire` with a replacement had nowhere to send its links). See [ADR012](../engineering/drs/ADR012-repair-to-successor-link-model.md) for the full option analysis, including why deprecating-the-source-on-merge and surfacing review candidates uniformly were both rejected.

## Affects

- **C008 - Lifecycle & Retirement** — the modified element: `retire`→`delete` (optional replacement), `relocate`→`move`, unified repair-to-successor rule, and review-candidate narrowing. Its Data model, Interfaces, Behavior, Edge cases, Relationships, and Success criteria are updated.
- **O005-R003 - Referential integrity** — served more fully: inbound links now follow a removed concept to its successor wherever one exists, rather than always degrading to plain text.
- **O003-R001 - Deprecation over deletion** — unchanged in intent; `deprecate` and `delete` remain the steward's free, no-default choice, now clearly two verbs.
- **ADR012** — records the decision and alternatives; supersedes ADR009's per-operation repair split. The `delete`/`move` renames follow **ADR011**.
