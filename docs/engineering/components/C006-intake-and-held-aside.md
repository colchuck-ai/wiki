# C006 — Intake & Held-Aside

**Role.** Own *all non-integrated material* — the producer-agnostic raw queue that
holds submissions before triage, and the recoverable held-aside store that holds
material triage rejected at the significance/scope bar. Neither is the corpus; nothing
here is ever a concept, and nothing here is destroyed.

## Capabilities

Behind the frozen surface ([INTERFACES.md § C006](../INTERFACES.md#c006-intake--held-aside-non-integrated-material)),
C006 hides the whole staging + recoverable-quarantine lifecycle:

- `submit(RawMaterial) -> IntakeId` — the one door in. Any producer, any time.
- `next() -> RawMaterial?` — hand the next untriaged item to C007 (`None` when the
  queue is empty).
- `hold_aside(item, reason) -> HeldId` — move an item into the recoverable held-aside
  store with the reason it was rejected. Called by C007.
- `list_held_aside() -> [HeldItem]` — enumerate what is held, for coverage review (C012).
- `restore(HeldId) -> RawMaterial` — return a held item to triage; the reversal path.

The depth is that these five verbs are the *only* way material moves in, out of, or
back into the non-integrated space — so "nothing lost before triage" and "every
rejection reversible" are structural, not conventions a caller must remember.

## Data model

Two stores, both owned exclusively by C006, both **non-integrated** (distinct from the
concept corpus C008 owns — material here has never been distilled into a concept):

- **Raw queue** (pre-triage). Each entry is a `RawMaterial` keyed by the `IntakeId`
  `submit` minted. FIFO-by-default ordering is the internal default (`next` hands back
  the oldest untriaged item); ordering is a component-internal choice, not part of the
  frozen surface. An entry leaves the queue only by being triaged — never by expiry.
- **Held-aside store** (recoverable rejections). Each entry is a `HeldItem` keyed by a
  `HeldId`: the original `RawMaterial`, the `reason` (below-significance / ambiguous- or
  out-of-scope), and enough origin to trace it back (the reason and hold event are the
  auditable record; the durable disposition trail lives in C005). `restore` returns the
  carried `RawMaterial` to the raw queue.

**Internal representation (recorded here).** Both stores are plain-text, in-corpus-repo,
version-controlled staging areas — a raw-queue holding area and a held-aside holding
area — consistent with the portability invariant, but kept **outside** the concept
corpus so no non-integrated item is ever mistaken for a concept. The exact on-disk
layout is a C006-internal choice hidden behind the five verbs; callers see only
`RawMaterial` / `IntakeId` / `HeldId` / `HeldItem`.

## Behavior

- **`submit`** — the single, **producer-agnostic** entry point (O007-R001). It accepts
  material *the moment it arrives* and **never rejects at the door**: no scope,
  significance, duplication, or format judgment happens here — those are triage's job.
  Every submission is durably queued and acknowledged with an `IntakeId`. This is what
  closes O007-RSK001 (lost before intake): once submitted, material cannot silently
  vanish before it is triaged.
- **`next`** — hands the next untriaged `RawMaterial` to C007 when C007 asks. C006 makes
  no disposition itself; it only supplies work.
- **`hold_aside`** — called by C007 when triage's outcome is `hold_aside` (below the
  significance bar, or ambiguous/out-of-scope). C006 moves the item from the raw queue
  into the held-aside store with its `reason`, mints a `HeldId`, and **reports the
  disposition to `C005.record`** (`outcome = held_aside`) so the rejection is auditable
  in the provenance history. This realizes O007-R003 / O003-R004: the bar gates
  *integrate-vs-hold-aside*, not *keep-vs-destroy*.
- **`list_held_aside`** — read by C012 for coverage review; the returned `HeldItem`s are
  the "recoverable held-aside items awaiting review" the O007 proxy counts.
- **`restore`** — returns a held item to the raw queue for re-triage (recording the
  reversal via `C005.record`). This is *deprecation over deletion* at intake: a wrong
  hold-aside call is reversible by moving the item back, never by resurrecting something
  that was destroyed.

**Purge boundary.** *No verb on C006 destroys material.* `submit`/`next` move items
through the queue; `hold_aside`/`restore` move items between the queue and the
held-aside store; `list_held_aside` only reads. Genuine purge — permanently removing a
held item — is a **separate, explicit act**, deliberately *not* one of these five verbs,
and outside the autonomous path (an operator/steward-initiated removal, recorded like
any other disposition). C006's contract is that everything submitted remains recoverable
until such an explicit act; C006 exposes no autonomous deletion.

## Relationships

Matches the frozen dependency direction (`C006 ◀── C007, C012`):

- **Producers → `submit`.** Any producer, human or agent; C006 knows nothing about who.
- **C007 → `next` / `hold_aside`.** Triage pulls work and routes rejects here; C006 is
  driven by C007, and does not call C007.
- **C012 → `list_held_aside`.** Coverage & Scope reads the held-aside listing as one
  input to the O007 coverage picture.
- **C006 → `C005.record`.** C006 *reports* hold-aside and restore dispositions to the
  provenance log; it never writes C005's store directly.
- **C006 does *not* call C004.** Source retention is authoring-time and corpus-side
  (ADR003); a durable snapshot is captured when C008 first cites a source, not at intake.
  C006 holds raw material, not citations.

## Decisions

- **Held-aside is a separate recoverable store, not an in-corpus marker.** Settled at
  the decomposition fork (one non-integrated-material owner). Held material has *never
  been a concept*, so it cannot carry a deprecation marker the way a retired concept does
  (C008/C011) — there is no concept to mark. It therefore lives in its own store, not in
  the corpus. **No ADR:** this follows directly from *deprecation over deletion* plus the
  fact that held-aside material was never integrated; it is not a surprising, hard-to-
  reverse trade-off, so it does not meet the ADR bar.

## Success criteria

Tied to the **O007 proxy** (nothing lost before intake; count of recoverable held-aside
items awaiting review):

- **Nothing relied upon is lost before triage.** Every `submit` durably queues and
  acknowledges its material; the only exit from the raw queue is triage. (O007-R001,
  closes O007-RSK001.)
- **Held-aside is visible for review.** `list_held_aside` gives C012 the complete,
  current set of recoverable rejections, so the O007 proxy count is always derivable.
- **Every rejection is auditable and reversible.** Each `hold_aside` carries its reason,
  is recorded to `C005.record`, and can be reversed by `restore` — a false negative at
  the significance bar is recoverable, never a silent, unrecoverable loss (O007-R003,
  closes O007-RSK003).
- **No autonomous destruction.** No C006 verb removes material; purge is a separate
  explicit act. The held-aside store only grows or is restored-from under autonomy.
