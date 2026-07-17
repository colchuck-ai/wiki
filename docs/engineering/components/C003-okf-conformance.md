# C003 — OKF Conformance

> **Status:** interface frozen (Chunk A). **Full behavioral spec: Phase 3 — pending.**

**Role.** The single owner of format knowledge: apply the external open format's
structural + naming conventions, and validate against the pinned spec.

**Owns.** Nothing persistent — format knowledge (stateless, deep by behavior). Wiki
*consumes* the format; it does not define it.

**Frozen interface.** See [INTERFACES.md](../INTERFACES.md) § C003 (`apply`, `validate`,
`spec_version`).

**Requirements owned.** O004-R002, O006-R002. See
[REQUIREMENT-MAP.md](../REQUIREMENT-MAP.md).

**Key relationships.** Called by C008 (apply/validate before every write) and C009/C012
(audit). No other component embeds format rules — a format version bump is a
one-component change.

**Decisions.** The single-conformance-owner boundary was re-derived from
convention-encapsulation + the external pinned format (settles old ADR001); no separate
ADR — it is a textbook deep-module boundary, documented here.

**Phase-3 spec must add:** the pinned OKF version + upgrade handling, the apply/validate
behavior, and success criteria tied to the O006 proxy (corpus validates against the
pinned spec).
