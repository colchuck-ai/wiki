# C002 — Governance

**Role.** The single seam for autonomy: it owns the delegation envelope and the one
composed steward attention surface, and it is the *only* place the act-vs-escalate
decision is made.

## Capabilities

- **`evaluate(ProposedAction) -> Decision`** — the envelope evaluator. Decides
  `act | escalate(reason)` for one proposed autonomous action. Pure, side-effect-free.
- **`escalate(item, reason, evidence) -> AttentionId`** — post an exception to the
  attention surface. The single posting path; makes no act-vs-escalate decision itself.
- **`resolve(AttentionId) -> StewardDecision`** — return the steward's adjudication that
  unblocks a held flow.
- **`envelope_version() -> Version`** — the envelope revision in force, stamped into
  every disposition for provenance.
- **`list_attention() -> [AttentionItem]`** — the steward's single inbox, composed
  (deduplicated/rolled up by subject) rather than a raw signal relay.

See [INTERFACES.md](../INTERFACES.md#c002-governance--the-envelope--the-one-attention-surface)
for the frozen signatures, invariants, and edge modes — this spec does not restate them.

## Data model

**The delegation envelope** (optional policy anchor). A steward-declared, persistent,
in-corpus, revisable-in-place statement of what Wiki may do autonomously — the *mirror*
of the charter's policy-anchor treatment (C001), differing only in that it is **optional**
and the envelope *reads* the charter, never the reverse. It is an ordered set of **rules**;
a rule scopes an action pattern (`ProposedAction.kind`, target class, and a consequence
band — e.g. "mechanical relink," "admit a below-threshold claim," "retire a stale concept")
to a verdict `act | escalate`. First-match wins; anything unmatched falls to the
conservative default (below). The envelope is one document, so a revision is a single
in-corpus edit that bumps `envelope_version` (a monotonic `Version`); the prior version
stays legible in the provenance log. Absent envelope → `envelope_version` returns a
reserved `default` sentinel, so provenance records that the built-in default, not a
declared rule set, governed a disposition.

**The attention surface store.** A set of `AttentionItem`s keyed by **subject** (see
below). Each item carries: its `AttentionId`, the subject, the accumulated escalation
reason(s), the `evidence` folded in from each contributing signal (for distillation
reviews this is the source span, O009-R003), a state (`open | resolved`), and — once
adjudicated — the `StewardDecision`. This is the only store C002 owns; the envelope
lives in the corpus, not here.

## Behavior

Two internal seams sit behind the one interface.

**(1) The envelope evaluator.** `evaluate` is a **pure decision**: `(ProposedAction,
charter via C001.read, envelope) → act | escalate(reason)`. It writes nothing — it posts
nothing, records nothing — so it is fully testable as input→verdict and callers stay in
control of sequencing. It is the **only** place act-vs-escalate is decided; no other
component embeds envelope logic. A declared envelope is applied first-match; with **no
envelope**, the conservative default holds: **escalate the consequential** (anything that
changes what a consumer relies on or is not cheaply reversible — admitting a new claim,
retiring/relocating a concept, a scope change) and **act on the mechanical** (reversible,
consumer-invisible upkeep — relinking, formatting, in-place stamping). The default leans
toward escalation on ambiguity, matching manage-by-exception's safety posture. Because
`evaluate` has no side effects, the caller that receives `escalate` is the one that calls
`escalate(...)`; `envelope_version()` is read by C005 (via the caller) to stamp the
resulting `Disposition`.

**(2) The composed attention surface.** `escalate` folds an incoming exception into the
item for its **subject** rather than appending a raw row, so N signals about one thing
surface as **one** `AttentionItem` — this is the writer-side of O004 (steward effort flat
as volume grows). **Subject = the `ConceptId`** the signal concerns; where there is no
concept yet (an intake/authoring candidate, a thin scope area) the subject is the target
referent (the pending item or the `ScopeArea`). *Rationale:* the steward adjudicates a
thing, not a stream, so keying rollup on the thing under judgment is what keeps the inbox
from being a firehose. `list_attention` returns these composed items. `resolve` returns
the `StewardDecision` for an item; resolving it settles **every** folded signal on that
subject at once (one decision unblocks all flows held on it) — again, one judgment per
subject, not per signal.

**Three escalation sources feed the one surface** (all via `escalate`, all composed the
same way):
1. **Envelope escalations** — an autonomous path (C007/C008/C011/C012) whose `evaluate`
   returned `escalate`.
2. **Review candidates** — items a periodic survey judged to need steward judgment:
   retirement candidates (C011) and coverage/scope-divergence findings (C012).
3. **Distillation reviews** — a claim C008's author-time grounding check could not admit
   within the envelope, carried with its **source span** as `evidence` so the steward can
   confirm the claim represents the source before integration (O009-R003).

There is no fourth channel and no per-component inbox: every exception in the system
surfaces here and nowhere else.

## Relationships

- **Reads C001** (`read`) — the envelope evaluates against charter scope.
- **Called by every autonomous capability** — C007 (triage), C008 (authoring/grounding),
  C011 (retirement), C012 (coverage/scope). They call `evaluate` for their consequential
  actions and `escalate` for the exceptions; they read `envelope_version` to stamp
  dispositions and block on / poll `resolve` to complete a held flow.
- **Depends on nothing below C002** in the [dependency direction](../INTERFACES.md#dependency-direction-no-cycles);
  C002 records its own steward-adjudication actions to C005 (via the standard `record`
  path), it does not write the provenance store directly. Envelope declaration/revision
  records with the policy-anchor `outcome` values `declared` / `revised` (the shared-type
  additions it shares with C001's charter), so envelope changes travel in the one
  provenance log.

## Decisions

Governed by [ADR002](../drs/ADR002-manage-by-exception-seam.md): one governance seam,
one evaluator, one composed inbox. The surface deliberately absorbs **only** the
product-motivated part of the old cross-face aggregator — subject dedup/rollup. The
elaborate conflict-pairing (expand-vs-retire), the face→verb routing table, and "Lint"
are **consciously dropped** ([DECOMPOSITION.md drop #1](../DECOMPOSITION.md#consciously-dropped))
and are **not architecture** now. Conflict-pairing may return as a **Phase-3 C002
refinement** *if steward experience demands it* — behind this same interface, requiring no
cross-component change.

## Success criteria

Tied to **O004** (curation-experience: steward effort stays flat as material volume grows):

- **Flat inbox, not flat input.** As integration/upkeep volume rises, the count of
  distinct `AttentionItem`s tracks the number of distinct *subjects needing judgment*, not
  the number of raw signals — composition holds N signals-about-one-concept to one item.
- **No second decision point.** Act-vs-escalate is exercised only through `evaluate`; a
  grep of the other components finds no embedded envelope logic and no alternate
  escalation channel.
- **Legible autonomy.** Every autonomous disposition carries an `envelope_version` (a real
  version, or the `default` sentinel), so the steward can audit which policy governed each
  action after the fact rather than gating each one before.
- **One decision clears the flow.** Resolving a composed item unblocks all flows held on
  that subject, so steward effort per exception does not scale with how many callers were
  waiting on it.
