---
title: 'Define MRP calculation rules and demo scenarios'
type: 'feature'
created: '2026-10-18'
status: 'done'
route: 'dispatch'
review_loop_iteration: 0
baseline_commit: 'fbaa06a38107de245c0078d506c3074975692be7'
context:
  - '.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/brief.md'
  - '.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/tilbakemelding-product-brief.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The brief describes requirement planning and ordering alerts, but it does not define the rules that make those calculations consistent or testable. The three demo scenarios also lack fixed input data and expected results.

**Approach:** Document a deliberately small, one-level BOM planning model and three hand-checkable fictional scenarios. Link the brief to that source of truth so the later PRD, architecture, implementation, and tests share the same rules.

## Boundaries & Constraints

**Always:** Keep all data fictional; use a one-level BOM (finished product to raw materials or packaging); make every calculation reproducible from shown inputs; show the buyer the basis for every alert; and keep the application as decision support rather than automatic ordering.

**Never:** Add ERP integration, demand forecasting, multi-level BOM explosion, automatic purchase placement, or implementation code as part of this planning change.

## Approved Decisions

- V1 has no safety stock: net need is calculated only from gross need and available inventory.
- The recommended order quantity equals positive net need; minimum order quantities and order multiples are deferred.
- Each supplier-material combination uses one fixed lead time in v1; delivery history is descriptive only and does not change the deadline rule.
- All three worked scenarios use a compact blueberry-yogurt example with one raw material, one cup, one foil seal (oblat), and one lid. The user approved the foil-seal addition during PRD creation; its inventory covers all three scenarios so the previously approved shortage quantities and deadlines remain unchanged.

## I/O & Edge-Case Matrix

**Historical specification note — 2026-11-02:** The original planning intent below is preserved. Later user-approved stock receipt/consumption requirements are defined by current PRD FR-25/FR-26 and calculation-contract MRP-R8–MRP-R10. The table's missing-lead-time wording is corrected to the already-approved partial-result rule; it is not permission to suppress a valid quantity recommendation.

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|---|---|---|---|
| Plan increase | A weekly quantity increases for a product with a defined BOM and inventory | Gross need, net need, recommendation, and latest safe order date are derived from the stated rule | Zero or negative net need creates no purchase recommendation |
| Packaging shortage | Required cups or lids exceed available stock | The affected packaging material, shortage quantity, and buyer action are visible | Missing BOM or stock blocks the affected recommendation; missing/invalid lead time preserves valid quantity and blocks only its deadline, with the calculation-contract message |
| Delayed delivery | A recorded delivery arrives later than its promised or standard lead time | The supplier delay and resulting material risk are explained without changing calculated quantities | A delivery with incomplete dates is excluded from lead-time statistics and marked incomplete |

</frozen-after-approval>

## Code Map

- `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/brief.md` -- approved product direction and current scope; update only to state the agreed calculation principles and link to the detailed rules.
- `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/tilbakemelding-product-brief.md` -- teacher feedback requiring explicit rules, hand-calculated expected outcomes, one-level BOM, and a single lead-time rule.
- `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/.memlog.md` -- records established product decisions; preserve the fictional-data and buyer-decision principles.
- `.docs/implementation-artifacts/spec-mrp-rules-and-scenarios.md` -- this draft specification; do not treat it as an implementation artifact until the four product decisions are approved.

## Tasks & Acceptance

**Execution:**
- [x] `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/calculation-rules.md` -- created the calculation contract, definitions, formulas, data assumptions, and worked scenario tables -- establishes the testable source of truth.
- [x] `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/brief.md` -- added a concise calculation-rules summary and a link to the detailed contract; revised the success criteria to refer to verified scenario outcomes -- makes the brief traceable without overloading it.
- [x] `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/.memlog.md` -- recorded the approved domain decisions and scenario-data choice -- preserves why the scope was selected.

**Acceptance Criteria:**
- Given a product quantity, its one-level BOM, and the current inventory, when the requirement rule is applied, then the documentation shows the gross need and net need with a hand-checkable calculation.
- Given a positive net need and an agreed ordering policy, when a recommendation is produced, then its quantity follows the documented rule exactly.
- Given a planning date and agreed lead-time policy, when a material is needed for a production date, then the latest safe order date is calculated and explained consistently.
- Given each of the three predefined scenarios, when its supplied fictional inputs are used, then the documentation states a material, warning reason, buyer options, and expected numeric result.

## Implementation Notes

- Added a separate calculation contract so the product brief remains concise while later planning and implementation work has a single, hand-checkable source of truth.
- Corrected the delayed-delivery explanation: a repeated three-day delay after the fixed-lead-time arrival date would arrive on 2026-11-13, three days after the 2026-11-10 production date.
- Verified all three scenario calculations and the documented missing-data and incomplete-delivery behavior with Node assertions.

## Spec Change Log

- 2026-11-02: Reconciled the older packaging-validation wording with MRP-R3: missing lead time blocks only a deadline, never a valid recommendation quantity. Current accepted scope includes scenario-local receipts/physical consumption under MRP-R8–MRP-R10 and four release increments; the historical planning-only specification is not the complete current scope. Supplier comparison is Increment 3 and optional AI is Increment 4. Existing named baseline quantities and deadlines remain unchanged.

- 2026-10-18: During PRD creation, the user expanded the compact one-level BOM with Yogurt foil seal (oblat) and approved a descriptive two-supplier Yogurt-lid comparison. This supersedes only the earlier three-line fixture wording; it preserves the approved formulas, validation rules, and three shortage outcomes.

## Review Triage Log

- `medium` — `calculation-rules.md` used an ambiguous "required input" statement while its lead-time row allows a quantity recommendation without a deadline. This could lead the future UI to suppress a valid quantity or display an unsupported deadline; route: patch.
- `medium` — `calculation-rules.md` specified supplier-material lead times but named only the lid supplier. Future supplier selection and comparison would lack a purchase target for the other materials; route: patch.
- `medium` — `calculation-rules.md` did not validate missing production quantity/date, non-positive BOM values, negative inventory or lead time, or incompatible units. Invalid inputs could produce plausible but false quantities or dates; route: patch.
- `medium` — `calculation-rules.md` described individual production dates without stating how a weekly plan shares inventory across multiple runs. Reusing on-hand stock for each run would understate shortages; route: patch.
- `medium` — `calculation-rules.md` calculated a deadline but did not define the buyer-facing urgency when it is today or overdue. The dashboard would not reliably prioritize a time-critical recommendation; route: patch.
- `medium` — `calculation-rules.md` referred to delivery statistics and supplier-delay risk without a deterministic display rule. The delayed-delivery scenario needs an explicit trigger and a minimal historical summary; route: patch.
- `medium` — `calculation-rules.md` lacked a rule for absent production quantity or date. Arithmetic or a deadline could otherwise be presented with undefined inputs; route: patch.
- `medium` — `calculation-rules.md` lacked a rule for negative quantity or lead-time values. Those values can silently create misleading shortages or deadlines; route: patch.
- `medium` — `calculation-rules.md` lacked a rule for a delivery receipt preceding its order-sent date. A negative lead time would corrupt late-delivery statistics; route: patch.

## Design Notes

The detail belongs in a separate calculation contract rather than embedded as a long table in the product brief. This preserves the brief as a readable product document while giving subsequent project artifacts a stable, testable reference.

## Verification

**Manual checks:**
- Recalculate each scenario from the displayed quantities, BOM values, inventory, and dates; expect the documented results to match exactly.
- Confirm that the brief links to the calculation contract and that no rule contradicts the first-version scope.
