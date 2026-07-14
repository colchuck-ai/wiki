# CR007 - Low-Volume Single-Steward Operating Envelope

Added a first-class Constraint to the engineering architecture stating that Wiki targets a low-volume, single-steward adjudication envelope, and anchored ADR004's previously-implicit "low-volume assumption" to the new ADR009. Scoped to the architecture README and, by reference, ADR004 - Merge Reconciliation Steering.

## Change

Before, the low-volume assumption existed only implicitly, inside ADR004's cost discussion ("acceptable under the current low-volume assumption"). The architecture README's Constraints section did not state it, so two design commitments — producer-agnostic intake that welcomes automated producers, and a steward-attention model that adjudicates every item and every merge by hand — sat in unstated tension.

After, the architecture README carries an explicit Constraint: Wiki targets a low-volume, single-steward envelope in which the steward's adjudication capacity is the system's backpressure, and machine-volume automated production is out of scope for the current design. ADR004's assumption is anchored to ADR009, which records the decision, its alternatives, and its consequences.

## Rationale

A load-bearing scoping assumption belongs in the architecture's stated constraints, not buried in one decision record's cost analysis. Stated, it can be checked against the producer-agnostic intake commitment and gives a clear signal for when high-volume affordances become necessary. Unstated, it silently breaks the first time an automated producer outruns the steward.

## Affects

- Engineering: the architecture README's Constraints section (new constraint), and ADR004 - Merge Reconciliation Steering (its "low-volume assumption" now references ADR009). No component behavior changes. Captured by [ADR009 - Low-Volume Single-Steward Operating Envelope](../engineering/drs/ADR009-low-volume-operating-envelope.md).
