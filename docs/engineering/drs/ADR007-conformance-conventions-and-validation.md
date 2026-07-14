# ADR007 - Conformance as Conventions and Validation

Re-scope **C004** from a mandatory runtime read/write gateway (the "OKF Conformance Adapter" every corpus access routes through) to the single **owner of format knowledge**, realized as (a) OKF authoring conventions encapsulated in the curation Agent Skill and (b) a conformance validator that checks the bundle against OKF §9 and reports violations. Rename the component to **C004 - OKF Conformance**. This **supersedes [ADR001 - Single OKF Conformance Boundary](ADR001-single-okf-conformance-boundary.md)**.

## Context

ADR001 routed every corpus read and write through C004 as a runtime gateway, so no other component held format knowledge and an OKF version bump was absorbed in one place — at the acknowledged cost of a mandatory indirection on every corpus access, a hot path that "must not become a bottleneck."

Two facts undercut that tradeoff. First, OKF conformance (SPEC §9) is deliberately near-empty: every non-reserved `.md` file has a parseable frontmatter block with a non-empty `type`, and `index.md`/`log.md` follow §6/§7 *when present*. Consumers **MUST NOT** reject a bundle for missing optional fields, unknown `type` values, unknown keys, or broken links. The enforcement surface a gateway would guard is essentially one rule, over a format explicitly designed to stay stable and permissive as bundles grow and are partially agent-generated.

Second, the authoring engine is an LLM (the Agent Skill, per the architecture's Technology Choices). Convention encapsulation (O007-R002) — applying the format's structural and naming conventions so no contributor need know them — is naturally a Skill-prompt responsibility, not a runtime data path. A mandatory gateway over a maximally-simple, permissive, stable format is abstraction the format's thinness does not earn.

## Options

- **Mandatory runtime gateway** (ADR001's choice): every read/write routes through C004. Rejected now — the indirection cost is not justified by a one-rule enforcement surface, and the LLM author already encapsulates the conventions the gateway would apply.
- **Format knowledge duplicated per component**: rejected for the same reason ADR001 rejected it — OKF rules spread across components, drift component-by-component, and an OKF bump touches every component.
- **Conventions plus validator, single owner, direct authoring** *(chosen)*: one component owns the conventions the Skill applies and a validator that checks conformance, while components author and read the corpus directly.

## Decision

Adopt **conventions plus validator**. C004 owns the OKF authoring conventions the Agent Skill applies and provides a conformance validator run as the `lint` operation's conformance check (see [ADR010 - Curation Agent Skill and Operations](ADR010-curation-agent-skill.md)). Other components author and read corpus concepts directly following those conventions rather than routing through a runtime gateway.

Format knowledge stays centralized — one component owns the conventions and the validator — and conformance is enforced by validation at well-defined checkpoints (after ingest, and during lint) instead of by structural impossibility. An OKF version bump is still absorbed in one place: the conventions and validator. The Requirement–Component Map is unchanged — O003-R003, O005-R001, O005-R002, O005-R003, and O007-R002 still map to C004; only the mechanism changes, not which component owns them.

## Consequences

- Enabling: removes the hot-path indirection on every corpus access while keeping format knowledge single-owned and centrally enforced.
- Enabling: matches the reality that an LLM author encapsulates conventions, and gives conformance an explicit, testable checkpoint (the validator) rather than an implicit structural guarantee.
- Enabling: an OKF revision remains an isolated change to C004's conventions and validator, so Wiki's non-goal of not defining the format is intact.
- Cost: conformance is now *checked* rather than *impossible to violate* — a concept can be transiently non-conformant between authoring and the next validation run; the design relies on the Agent Skill honoring the conventions and on the validator being run (it is wired into the `lint` operation, ADR010).
- Cost: the principle "components reason about concepts, never files" weakens from a hard structural boundary to a convention the other components follow.
