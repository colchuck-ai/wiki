# CR006 - Assisted-upkeep traceability: C003 and reciprocal claims

Corrected the O004-R003 (assisted upkeep) requirement-component mapping to include C003 - Integration Authoring — the remediation interface CR003 established but never propagated to the map — and made every mapped component claim the requirement in its own document, so the mapping is bidirectional and no longer disagrees with the components it points at.

## Change

Before: the Requirement-Component Map mapped **O004-R003 - Assisted upkeep** to {C006, C007, C008, C009, C010}. C003 was absent, even though CR003 made `revise` from the steward's direction the concrete assisted-upkeep interface — correcting an existing concept by intent, with no hand-editing. Of the five mapped components, only C009 claimed O004-R003 in its own intro; C006, C007, C008, and C010 did not. C009's claim, moreover, enumerated siblings ("shares assisted upkeep with C006, C007, C008, and C010") that never reciprocated, and it omitted C003 entirely. The map and the components disagreed in both directions: the map named components that were silent, and the one claiming component named a set that didn't match.

After: the row lists **C003, C006, C007, C008, C009, C010**, and each states its share of assisted upkeep in its own intro. C003 is the remediation half — `revise` from steward direction acts on the detect faces' findings with no hand-editing. C006, C007, C009, and C010 are detect faces, each surfacing its signal (drift, staleness, gaps, scope drift) so the steward audits nothing by hand. C008 sits on both sides: it surfaces retirement candidates like the other detect faces and carries out the confirmed removal, deprecation, or relocation by verb. The fragile sibling enumeration is replaced everywhere by one role-based phrase — "the corpus's other detect faces and C003's remediation verbs" — so the claim no longer rots when the satisfying set changes.

## Rationale

O004-R003 is a two-part requirement: "surface what needs the steward's attention *and* carry out the resulting changes by intent, so maintenance requires no hand-auditing *or* hand-editing of the corpus." The detect faces (C006, C007, C008, C009, C010) provide the surfacing half; C003's `revise` and C008's lifecycle verbs provide the carry-out half. The complete satisfying set therefore spans both halves — the current five detect/lifecycle components plus C003. Narrowing the row to only the components that happened to claim it would have been the smaller edit, but it would have stripped the surfacing half of its component owners, leaving "no hand-auditing" traced to nothing.

The gap traces to CR003: it introduced the remediation interface (`revise` from steward direction, filling the O004-R003 cell) but the change never reached the traceability map. This reconciles the artifact with the decision. Replacing C009's enumeration with a role phrase fixes the class of defect rather than the instance — an inline list of co-satisfiers is a second place the mapping lives, and it had already drifted from the real one on the architecture element.

## Affects

- **Wiki Curation Architecture** — the Requirement-Component Map's O004-R003 row gains C003 - Integration Authoring (now C003, C006, C007, C008, C009, C010).
- **C003 - Integration Authoring** — its intro now states O004-R003 (the remediation half of assisted upkeep) as a first-class claim, alongside its existing O001-R001, O004-R001, and O005-R002 claims. The `revise`-from-direction body language (already present) is unchanged.
- **C006 - Source Revalidation**, **C007 - Currency Tracking**, **C009 - Coverage Review**, **C010 - Charter** — each intro now reciprocally claims its detect-face share of O004-R003, via the uniform role-based phrase. C009's prior sibling enumeration is retired.
- **C008 - Lifecycle & Retirement** — its intro now claims O004-R003 on both the surface side (retirement candidates) and the carry-out side (its lifecycle verbs).
- **O004-R003 - Assisted upkeep** — now fully traced to its satisfying components in both directions.
- **CR003** — this propagates the assisted-upkeep interface CR003 recorded into the traceability map it never reached.
