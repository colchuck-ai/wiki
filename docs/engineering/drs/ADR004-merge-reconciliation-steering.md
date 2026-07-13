# ADR004 - Merge Reconciliation Steering

When an incoming item overlaps an existing concept, Wiki surfaces the overlap and has the steward choose among defined reconciliation modes — **supersede**, **fold-in**, or **correct** — before authoring; the tool then executes the chosen mode. Merge is the one integration direction that is never fire-and-forget.

## Context

O007-R001 lists **merge** as a steward direction alongside keep, deprecate, and discard, and the product's "intent in, authoring out" stance (O007) lets the steward trust the tool's authoring — so keep and discard are near-instant, one-word verdicts. But **merge** is under-specified. An item can overlap an existing concept in incompatible ways: it may *replace* the standing statement, *coexist* with it, or *correct* it. These are editorial judgments about what the corpus now asserts, and the tool cannot infer which one the steward means without risking a misrepresentation of shared knowledge.

Two existing requirements bound how a merge must behave: O006-R001 requires that a replaced concept be deprecated with a reason and a pointer to its replacement rather than hard-deleted, and O003-R004 requires inbound links to be resolved before a statement is relocated or removed. A merge that supersedes must honor both. The architecture must decide how the bare "merge" direction becomes corpus content.

## Options

- **Silent auto-merge**: the tool infers how to combine the item with the overlapping concept and authors without asking. Lowest steward effort, but the tool must guess supersede-vs-coexist-vs-correct and can silently overwrite or contradict standing knowledge. This is the one place "trust the authoring" breaks — the decision is editorial, not mechanical, so trusting the tool to make it is trusting it to invent the team's position.
- **Always author as a new concept**: never merge; every item becomes its own concept, cross-linked to near-neighbors. No overwrite risk, but the corpus accretes near-duplicates and O006 high-signal degrades, and any contradiction between the old and new statement is left for the consumer to resolve at read time.
- **Steward-steered reconciliation** *(chosen)*: the tool detects the overlap during triage, surfaces the target concept(s), and asks the steward to choose supersede / fold-in / correct; the tool then executes the chosen mode. Adds exactly one decision per merge, but keeps the editorial judgment with the steward and the mechanics with the tool.

## Decision

Adopt **steward-steered reconciliation**. C002 - Scope Triage detects and surfaces the overlapping concept(s) alongside the merge direction; C003 - Integration Authoring presents three modes and executes the chosen one:

- **Supersede** — author the replacement into the target concept and mark the prior statement deprecated in place, with a reason and a pointer to the replacement (through C008 - Lifecycle & Retirement), inbound links resolved first (through C005 - Index & Navigation).
- **Fold-in** — incorporate the material into the existing concept as a coexisting statement; nothing is deprecated.
- **Correct** — replace an erroneous or stale statement in place (the prior text was wrong, not superseded); the prior text is preserved in change history (through C007 - Currency Tracking).

This localizes the only non-mechanical integration decision to a single explicit steward choice, so "trust the authoring" holds for the mechanics without the tool guessing editorial intent. Keep and discard stay fire-and-forget; merge alone requires the reconciliation choice.

## Consequences

- Enabling: the tool never silently overwrites or contradicts standing knowledge — supersede honors O006-R001 (deprecation with a replacement pointer) and O003-R004 (inbound links resolved before relocation) by construction.
- Enabling: keeps O007's low-effort promise intact on the common path (keep/discard) while spending the steward's attention only where an editorial decision genuinely exists.
- Cost: a merge is a two-step interaction (surface → choose → author), not a one-word verdict; under high overlap volume this would be the steward's main effort. Acceptable under the current low-volume assumption.
- Cost: overlap detection in C002 must be good enough to surface the right target(s); a missed overlap silently falls back to authoring a near-duplicate — an O006 retirement-review concern, not a correctness failure.
