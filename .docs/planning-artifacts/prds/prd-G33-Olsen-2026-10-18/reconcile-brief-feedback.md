# Reconciliation: Product Brief and Teacher Feedback

## Covered inputs

- The PRD preserves the concrete buyer problem, no-login fictional-data demo, one-level BOM, and non-autonomous purchasing boundary from the brief.
- It incorporates the teacher's requested calculation contract: gross need, net need, recommended quantity, fixed lead time, latest safe order date, validation behavior, and hand-checkable outcomes are authoritative through the linked calculation rules.
- All required material categories are present in FR-1 and all three scenario fixtures: Yogurt lid, Yogurt cup, Yogurt foil seal (oblat), and Blueberry puree as the selected raw-material line. A separate base-yogurt ingredient is not required by the approved fixture.
- Scope is staged in the requested order: core MRP, supplier follow-up, then optional AI; trend dashboards and login are outside v1.
- The AI choice, server-only key handling, deterministic English fallback, cost controls, and rule that AI never calculates are covered by FR-20–FR-24 and the linked AI contract.

## Missing or changed requirements

1. **Automated tests are not explicitly forbidden from calling the live provider.** NFR-8 requires provider paths to be testable, but the PRD should state that automated tests use a deterministic fake/stub and run without an API key or paid calls.
2. **Assessment discoverability is only implicit.** NFR-9 covers traceability, but the teacher specifically asks that README link to the hidden `.docs/` planning artifacts and identify where seed data and scenario fixtures live.
3. **The requested UX evidence is absent.** The feedback recommends sketches of the priority dashboard and alert-detail view. The PRD defines their behavior, but does not require or link a UX artifact for those two views.
4. **Persistence and reset expand Increment 1 beyond the teacher's minimum chain.** They were added later for repeatable assessment and are useful, but the teacher's strict minimum is plan change → material need/shortage → latest safe order date. Persistence/reset should remain supporting requirements and must not delay that tested chain.

## Conflicts

- There is no substantive conflict on materials: the current fixture includes lids, cups, seals, and a raw material. Adding a separate base-yogurt ingredient now would expand the verified BOM and should not be inferred without a new decision.
- There is no conflict on supplier comparison: limiting the first comparison to two fictional Yogurt-lid suppliers is consistent with the teacher's request to control scope, provided the four BOM materials still appear in the MRP results.
- The only scope tension is that persistence/reset is treated as part of the minimum increment even though it was not part of the teacher's stated minimum functionality.

## Recommended PRD correction

Add three explicit assessment requirements: (1) provider tests must use a deterministic fake and make no live or paid API calls; (2) README must link to `.docs/` and name the seed/scenario locations; and (3) the UX deliverable must cover the dashboard priority view and alert detail. Clarify in Increment 1 that the tested MRP chain is the release gate and persistence/reset is supporting work that cannot delay it. Keep Blueberry puree as the approved raw-material line rather than adding another ingredient unless the calculation contract is deliberately revised.
