# C008 — Integration

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The **sole writer** of the concept corpus, via an intent-verb surface.

**Owns.** The concept corpus (the single source of truth). In-concept fields it writes:
claims, citations, per-claim grounding signal + review tier, reason-annotated
cross-links, deprecation/supersession markers, recency stamp.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C008 (`integrate`, `merge`,
`deprecate`, `supersede`, `relocate`, `repair_links`, `flag`). Every verb commits the
concept change and its `C005.record` provenance entry atomically (one commit or none).

**Requirements owned.** O001-R001, O003-R001, O004-R001, O005-R002 (write), O006-R001/R003,
O009-R001, O009-R002. See [REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Calls C003 (apply/validate), C004 (retain/resolve), C005 (record),
C002 (admit/hold/escalate), C009 (inbound_links for repair). Driven by C007, C011, C012
via intent. The author-time grounding check is an **internal seam** of this component.

**Decisions.** [ADR001](../drs/ADR001-integration-sole-corpus-writer.md) (sole writer +
per-verb atomicity + corpus-as-source-of-truth); grounding seam internal (settles old
ADR015).

**Phase-3 spec must add:** each verb's behavior (distillation, merge composition +
citation pruning, supersession, relocation), the grounding-check seam, and success
criteria tied to the O009 proxy (grounding pass rate) and O001 (cited claims).
