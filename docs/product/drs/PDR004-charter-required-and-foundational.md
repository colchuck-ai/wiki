# PDR004 - Charter required and foundational

We decided to make the **charter required** (rather than optional) and to
**elevate it from a single requirement under O008 to a foundational corpus
concept** the outcome tree presupposes — one that triage, the significance bar,
duplication, and coverage all read against, and that earns a clause in the
product spine.

This decision affects the product statement (`docs/product/README.md`): the
charter moves into a Foundational concepts section; O008-R001 becomes
"charter-anchored scope"; O007 and O008 are marked as predicated on a declared
charter.

## Context

O008-R001 made the charter *optional*. But two outcomes are defined entirely in
terms of a scope: O008 (on-scope focus) and O007 (complete coverage, "within
the intended scope"). The only thing that declares that scope is the charter —
its absence *is* O008-RSK002 (implicit boundary). So an optional charter that
two outcomes depend on means a steward who skips it silently voids two of nine
outcomes, with nothing signalling the gap.

Inferring scope from existing corpus contents was rejected: it makes "intended
scope" a mirror of "whatever arrived," which is exactly O008-RSK001
(input-driven drift) — inference defines the outcome into the failure it exists
to prevent.

Separately, the charter is read by far more than scope-drift: coverage (O007),
the significance bar (O003-R004, "substantial *relative to the charter*"), and
duplication (O003-R003, overlap judged relative to scope) all consult it, and
the delegation envelope's scope rules read it too. Left as a leaf under O008, a
reader would never guess that significance and dedup depend on it.

## Options

- **Keep charter optional** (status quo): lowest setup friction, but O007/O008
  silently evaporate without it and the tree misrepresents the charter's reach.
- **Required but still a leaf under O008**: fixes the referent problem, but
  still hides that significance, dedup, and coverage read the charter.
- **Required and elevated to a foundational concept** (chosen): the corpus has a
  purpose the way it has an identity; the outcomes presuppose it.

## Decision

Make the charter **required** and **foundational**, under two conditions that
keep "required" from becoming dead compliance:

1. **Required to exist and be revisable, not complete on day one.** The charter
   is provisional at first and sharpened as the material is understood — a
   required setup wall on a finished scope statement would punish the honest
   steward still figuring out the scope.
2. **Required and load-bearing.** It must be an actual input to triage (O008),
   the significance bar (O003), duplication (O003), and coverage (O007). A
   required-but-unconsulted charter is theater.

The charter is a **policy anchor** — the same declared/persistent/in-corpus/
revisable pattern as the delegation envelope
([PDR003](PDR003-manage-by-exception.md)) — but the two are distinct artifacts:
different audiences (the charter is consumer-legible identity; the envelope is
operational), different change cadence (identity-rare vs. operationally tuned),
and the envelope *reads* the charter, not the reverse. They differ in that the
charter is required and the envelope optional.

## Consequences

- O007 and O008 always have a referent; their metrics are well-defined.
- The charter earns a spine clause and a Foundational-concepts entry; O008-R001
  becomes "charter-anchored scope."
- Because scope is always declared, scope-aware triage and reconciliation
  (O008-R002/R003) and the significance bar (O003-R004) have a stable basis for
  their judgments.
- A consumer can read the charter to know what the corpus is *for* — feeding the
  transferable-stewardship principle, since the corpus now externalizes the
  purpose the steward would otherwise hold only in their head.
