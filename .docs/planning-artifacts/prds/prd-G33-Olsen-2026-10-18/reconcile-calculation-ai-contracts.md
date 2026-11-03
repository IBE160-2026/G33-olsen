# Reconciliation: PRD vs. Calculation and AI Contracts

## Scope and Verdict

This reconciliation compares `prd.md` with:

- `calculation-rules.md`
- `ai-explanation-plan.md`
- `spec-mrp-rules-and-scenarios.md`
- `spec-ai-explanation-and-fallback.md`
- `spec-prioritize-v1-scope.md`

The PRD covers the approved product behavior and correctly delegates implementation-level precision to the two linked contracts. The newly confirmed one-level BOM contains all four required materials—Blueberry puree, Yogurt cup, Yogurt foil seal (oblat), and Yogurt lid—and FR-19 includes the approved two-supplier Yogurt-lid comparison. Two source-level inconsistencies and one acceptance-coverage omission remain; none requires changing the PRD's product direction.

## Covered Requirements

| Contract area | PRD coverage | Result |
|---|---|---|
| One-level, four-material BOM | FR-1 names Blueberry puree, Yogurt cup, Yogurt foil seal (oblat), and Yogurt lid; FR-5 applies the calculation to every BOM line. | Covered |
| Per-plan MRP arithmetic | FR-5 defines Gross Need, non-negative Net Need, and Recommended Order Quantity equal to positive Net Need. | Covered |
| Weekly aggregation | FR-6 requires material grouping, sequential inventory consumption, deterministic date/product/row ordering, no double-counting, and the weekly identity. | Covered |
| Deadline and urgency states | FR-7 covers fixed calendar-day lead time, earliest weekly deadline, **Order today**, and overdue behavior without changing quantity. | Covered |
| Visible calculation basis | FR-8 requires the source inputs and derived facts required by the calculation contract. | Covered |
| Partial and blocked results | FR-9 preserves quantity when only lead time is invalid, preserves shortage without a supplier while withholding the purchase option, and delegates exact validation ordering to the contract. | Covered |
| Standard explanations and buyer control | FR-10–FR-12 require immediate deterministic English explanations, complete applicable options, and no claimed operational action. | Covered |
| Delivery history and supplier risk | FR-13–FR-18 cover recording, validation, exclusion of invalid/incomplete records, exact descriptive statistic categories, lateness rules, the positive-shortage risk gate, and the non-interference boundary. | Covered |
| Supplier comparison fixture | FR-19 fixes the comparison to DairyPack AS and PackForm AS for Yogurt lid, preserves DairyPack as configured supplier, includes PackForm's 4-day lead time and its two delivery outcomes, and forbids ranking, switching, or calculation changes. | Covered |
| Keyless AI fallback | FR-20–FR-24 and NFR-2–NFR-4 keep the deterministic flow fully usable without a key, account access, provider availability, or network success. | Covered |
| Explanation-only AI boundary | FR-21–FR-23 require immutable application facts, closed structured output, server validation/rendering, stale-result rejection, and fallback rather than model-generated calculations or prose. | Covered |
| Approved provider configuration | FR-24 records `gpt-5.6-sol` with separate `high` reasoning effort and delegates operational controls to the AI contract. | Covered |
| Gated v1 scope | §8 preserves the completed scope spec: deterministic MRP first, provider-independent supplier follow-up second, and optional AI third; trends and login remain non-goals. | Covered |

## Omissions Requiring Follow-up

### 1. Supplier-comparison fixture lacks an explicit exact-value acceptance check

FR-19 describes the PackForm fixture inputs, and FR-15 defines the statistics, but the Success Metrics do not explicitly require the comparison to reproduce PackForm's expected output: 2 valid deliveries, 4.5 days average actual lead time, 1 late delivery, and 1 day greatest delay. SM-1 can be read as covering only the three worked MRP scenarios. Before stories/tests are finalized, add an acceptance test derived from `calculation-rules.md` for both supplier columns and confirm that **Reset demo data** restores the comparison records as part of the baseline.

## Conflicts and Ambiguities

### 1. Yogurt-cup fixed lead time is inconsistent inside the calculation contract

The contract states that each supplier–Material combination has one fixed lead time, and PackForm AS is the named Yogurt-cup supplier. Scenario 1 uses 5 calendar days for Yogurt cup, while scenarios 2 and 3 use 4 calendar days for the same supplier–Material combination. The PRD correctly delegates exact fixture values, so it cannot settle this contradiction. Select one fixed value and update the calculation contract and affected expected dates before implementation.

### 2. The frozen MRP spec describes a three-line BOM while the approved contract and PRD use four lines

`spec-mrp-rules-and-scenarios.md` says the compact example has “one ingredient, one cup, and one lid,” omitting the Yogurt foil seal (oblat). The later calculation contract, current PRD, and latest user direction all require four materials. Treat the four-material BOM as the current decision, but formally record that it supersedes the older frozen wording before using the spec as implementation evidence.

## Technical Detail Correctly Left in Source Contracts

The PRD should remain product-focused. The following details are correctly authoritative in the linked contracts rather than duplicated in the PRD:

- Exact BOM quantities, material units, suppliers, inventories, production dates, lead times, arithmetic, validation strings, delivery dates, supplier statistics, and hand-calculated scenario outputs.
- The supplier-risk formula and exact risk wording, including the DairyPack and PackForm fixture rows and expected statistics.
- Exact standard-explanation wording, option-code ordering, clause ordering, evidence kinds, value/unit/nullability rules, weekly row structure, template IDs, and deterministic formatting.
- Request/response schemas; safe-ID, byte, array, row, token, and time limits; strict structured-output behavior; result-version binding; eligibility rules; cache identity; idempotency; atomic reservation; rate/concurrency caps; kill switch; cancellation; logging; and negative-test matrix.
- Server-only environment-variable handling and the rule that unavailable model/account/schema capability always uses the standard fallback without silent model substitution.

## Reconciliation Outcome

The PRD is directionally ready for continued finalization. Resolve the Yogurt-cup lead-time value, formally supersede the old three-line BOM wording, and carry the exact supplier-comparison outputs into acceptance tests; otherwise the calculation and AI commitments are represented without unnecessary technical duplication.
