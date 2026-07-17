# C002 — Governance

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The single seam for autonomy: envelope evaluation (`act | escalate`) and the
one composed steward attention surface.

**Owns.** The delegation envelope (optional policy anchor) + the steward attention
surface (deduplicated/rolled-up by subject).

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C002 (`evaluate`,
`escalate`, `resolve`, `envelope_version`, `list_attention`). Two internal seams: the
envelope evaluator and the composed attention surface.

**Requirements owned.** O004-R004; O009-R003 (attention surface for distillation
reviews). See [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Reads C001. Called by every autonomous capability (C007, C008,
C011, C012).

**Decisions.** [ADR002](../drs/ADR002-manage-by-exception-seam.md). Absorbs only the
product-motivated part of the old aggregator (dedup/rollup); conflict-pairing/routing
are a possible Phase-3 refinement, not architecture.

**Phase-3 spec must add:** envelope semantics + versioning, the conservative
no-envelope default, the composition/rollup rules, and success criteria tied to O004
(steward effort stays flat as volume grows).
