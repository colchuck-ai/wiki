# PDR003 - Reliance Is Steward-Supplied

The reviews that keep the reference complete (O002-R003) and high-signal (O006-R002) operate on **measurable proxies plus the steward's judgment**, not on a system-measured signal of "what the team relies on." Wiki captures no usage or reliance telemetry, so the requirement wording is reframed to name the signals a component can actually compute, and reliance itself is an explicit steward input.

## Context

O002 (Complete coverage) and O006 (High signal) both invoked "what the team actually relies on" and "concepts no longer relied upon." But nothing in the architecture produces a reliance signal: Query is not a first-class, persisted surface and logs no usage, and the corpus is plain-text with no telemetry. C009 - Coverage Review already quietly substituted three proxies (dangling links, declared scaffold scope, agent-proposed gaps) for the missing reliance data; C008 retirement review had no method at all — "no longer relied upon" was asserted and never operationalized. Two requirements therefore promised a capability the system cannot deliver as written.

## Options

- **Measure reliance via query telemetry**: have the Query operation record which concepts are read and cited, and feed coverage and retirement. Powerful, but introduces usage state, cuts against the query non-goal, and is premature under the low-volume single-steward envelope (ADR009) where no reliance-precision pain has appeared.
- **Keep the "relied upon" wording unmeasured**: leave the requirements as they read. Rejected — they overpromise and are unbuildable as stated; a reviewer cannot tell when they are met.
- **Reframe to measurable proxies plus steward judgment** *(chosen)*: coverage draws on declared scope, dangling links, and agent-proposed gaps; retirement draws on staleness, source drift, supersession, and scope-drift; the judgment of what the team relies on is supplied by the steward, not measured by the tool.

## Decision

Adopt the **proxies-plus-steward-judgment** reframe. O002-R003 and O006-R002 are reworded to name the computable signals each review runs on, with reliance as an explicit steward input. C009 already conforms; C008's retirement review is specified on staleness (C007 timestamp age), source drift (C006), supersession, and scope-drift (C010 Survey), presented for the steward's retirement judgment. A query-telemetry reliance signal is recorded here as a deferred future capability, to revisit only if retirement precision becomes a real pain.

## Consequences

- Enabling: the requirements now match what a component can compute — no phantom capability, and a reviewer can tell when each is met.
- Enabling: C008 retirement gains a concrete, buildable basis consistent with C009's proxy approach, so the two periodic reviews rest on the same kind of measurable signal.
- Cost: neither review can claim usage-grounded reliance; a concept that is genuinely unused but on-scope and current will not be auto-flagged for retirement — the steward must judge it.
- Follow-up: a query-telemetry reliance signal is the natural enhancement if retirement precision is ever needed; it is deferred under ADR009 and would be weighed against the query non-goal at that time.
