# PDR001 - Claim fidelity as a first-class outcome

We decided to measure whether an authored claim faithfully represents what its cited source supports as its own outcome — **O009 - Faithful representation** — rather than folding fidelity into an existing outcome, and to amend the product spine sentence so "trustworthy" explicitly includes *faithful to its source*.

This decision affects the product statement (`docs/product/README.md`) and adds the new outcome O009. It depends on O001 - Traceable provenance, whose durable source reference (O001-R002) the fidelity checks read against, but does not change it.

## Context

Wiki is a distiller: it refines raw material into cited concepts and stakes its value on knowledge a consumer can trust on sight. Distortion in distillation — a claim that overstates, oversimplifies, or drifts from what its source actually supports — is the distiller's most characteristic failure mode.

The outcome set O001–O008 measured traceability, currency, sustained signal, curation effort, discovery, portability, coverage, and on-scope focus — but nothing measured fidelity. O001 gives a claim a durable, resolvable source reference (O001-R002) and detects when that source later *changes* (O001-R003), yet a claim can be perfectly cited and durable while still misrepresenting what the source says at the moment it is authored. A consumer who cannot trust the distillation routes questions back to the steward — collapsing the core job promise of not routing every question through one person.

## Options

- **Fold fidelity into O001 - Traceable provenance** as an added risk and requirement: keeps the outcome count down, but conflates two independent properties — *can I find the source?* (traceability) and *does the claim match it?* (fidelity). O001's metric (proportion of claims a consumer can trace) would become ambiguous, and the fidelity gap would stay hidden inside a passing outcome.
- **Treat fidelity as a quality gate under O003 - Sustained signal or O004 - Low curation effort**: those measure signal-to-noise and hands-on effort, not correctness of representation. A distorted claim can be substantial, unique, and cheap to author while still being unfaithful, so neither metric would register the failure.
- **Add fidelity as a first-class outcome (O009)** with its own risks and requirements, and amend the spine sentence: raises the outcome count to nine, but gives the distiller's signature failure a dedicated, measurable target and a clean mitigation trace.

## Decision

Add **O009 - Faithful representation** as a first-class outcome under the Knowledge Stewardship job, with risks O009-RSK001 (distortion in distillation) and O009-RSK002 (indistinguishable grounding), and requirements O009-R001 (source-grounded authoring), O009-R002 (legible grounding), and O009-R003 (distillation review). Amend the product spine sentence so the decomposition of "trustworthy" is complete: traceable + faithful + current + complete + portable.

Fidelity is genuinely distinct from traceability: a claim can be traceable but unfaithful, or faithful but untraceable. Keeping them separate lets each be measured and mitigated on its own terms.

O009 introduces **no new source-retention mechanism**. Its checks read against the durable, version-controlled source reference that O001-R002 already guarantees; O009-R001 and O009-R003 depend on it — without a resolvable source there is nothing to check a claim against.

## Consequences

- The distiller's most characteristic failure now has a measurable target and an explicit mitigation trace (authoring check → consumer grounding signal → steward distillation review).
- "Trustworthy" in the product headline is now fully decomposed across outcomes, with no silent gap between "cited" and "correct."
- O009's three requirements now trace to the engineering tree. Source-grounded authoring (O009-R001) and the legible per-claim grounding signal + review tier (O009-R002) are owned by **C008 - Integration**, where the grounding check is an author-time internal seam — a placement settled in [ADR001](../../engineering/drs/ADR001-integration-sole-corpus-writer.md) (sole corpus writer), which absorbed the old standalone grounding-seam decision, with signal placement in [ADR004](../../engineering/drs/ADR004-machine-readable-signal-placement.md). Distillation review (O009-R003) is **shared**: C008 escalates each unsupported claim with the span of the source it was drawn from, and **C002 - Governance** owns the attention surface the steward reviews it on. The full trace is in the [Requirement → Component map](../../engineering/REQUIREMENT-MAP.md). (Originally logged here as an open follow-up — assign owners for the source-grounding check, the consumer-facing grounding signal, and the claim-alongside-source review view — and discharged by that trace.)
- Fidelity checking is heuristic — judging whether a claim is supported by a source is itself fallible — so the outcome is stated as *maximize the proportion*, not as a guarantee.
- Reusing O001-R002 keeps provenance and fidelity on one durable-source substrate; a change to how sources are held durably now affects both outcomes.
