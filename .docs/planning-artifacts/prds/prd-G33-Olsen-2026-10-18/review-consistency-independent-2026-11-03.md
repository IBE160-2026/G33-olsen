# Independent Consistency Review — ProsjektMelk

**Review date:** 2026-11-03
**Snapshot:** Current reconciled PRD (`status: final`, updated 2026-11-03), including the clarified MRP-R9 completion inputs and replay/reset-context rules.
**Scope:** Documentation consistency and exact worked-number/date checks. No application implementation or provider integration was tested. Earlier reports are preserved.

## Verdict

No actionable consistency finding remains in the reviewed scope. The approved current-stock workflow, date meanings, receipt/history linkage, outstanding-only planning, and four-increment release order agree across the current brief, PRD, and calculation/AI contracts. Historical specifications clearly identify the later scope and partial-result rules that supersede their earlier wording.

| Severity | Findings |
|---|---:|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 0 |

## Independently checked stock logic

- **Current physical balance:** PRD Glossary/FR-6 and MRP-R2 use opening balance plus posted accepted receipts minus posted consumption. Allocation is a calculation over current stock and outstanding rows, not another physical deduction. Completed rows stay visible and read-only but contribute no remaining demand or allocation.
- **Balance example:** `1,000 + (500 − 20) − 500 = 980` cups. Outstanding demand of 1,200 cups leaves a 220-cup shortage. Historical opening stock and completed demand are not counted again.
- **Full four-material fixture:** An 800-cup run needs 120 kg puree, 800 cups, 800 seals, and 800 lids. Opening stock is 140 kg, 650 cups, 850 seals, and 900 lids. The 500-cup receipt with 20 rejected adds 480 cups, producing 1,130 cups before completion. Completion leaves **20 kg puree, 330 cups, 50 seals, and 100 lids**. No outstanding row remains, so no current shortage deadline or Supplier Risk remains for that run; delivery history is retained.
- **Allocation conflict:** With 1,000 physical cups, an earlier 800-cup outstanding run, and a selected 500-cup run, explicit completion leaves 500 physical cups and a 300-cup shortage for the earlier run. The approved warning/confirmation behavior is preserved; cancellation changes nothing. Physical insufficiency is a separate atomic-posting failure.
- **Completion prerequisites:** Current MRP-R9 explicitly requires quantity/date, BOM/units, and sufficient physical stock. Missing lead time, deadline, supplier history, or provider configuration does not block otherwise valid physical consumption. This is consistent with MRP-R3's partial-result rule.
- **Replay and context:** Equivalent receipt/completion retries resolve the existing posting before new-state validation. Receipt posting and linked Delivery Record creation are atomic. Reset retires the previous posting context, and scenario switching cannot silently change ownership or replay movements into another scenario.

## Independently checked date and history logic

- **Three distinct meanings:** Order Sent Date is when the purchase order was sent; Confirmed Delivery Date is the optional supplier-promised arrival; Actual Receipt Date is physical receipt. The names map to the existing order-sent/promised/delivery-received contract fields without duplicate dates or email integration.
- **Actual versus expected supply:** A new receipt requires a valid Actual Receipt Date on or before the scenario Planning Date. Confirmed Delivery Date can be future-dated and never increases physical stock by itself. This agrees with the explicitly deferred date-effective supply model.
- **Known versus unknown:** Unknown Order Sent Date is an explicit buyer choice. It permits a valid physical receipt but leaves its linked Delivery Record incomplete and excluded from all supplier statistics/risk. Invalid supplied dates are not silently converted to unknown. A known order date after actual receipt, or confirmation before a known order date, blocks the whole posting.
- **Lifecycle dates:** 2026-11-08 to 2026-11-12 is four calendar days. Actual receipt on 2026-11-12 is one day late against confirmation on 2026-11-11. The four-day cup lead time gives the run a 2026-11-08 latest safe order date, four days overdue against this independent fixture's 2026-11-12 Planning Date.
- **Baseline compatibility:** The separate lifecycle fixture is not a fourth UI scenario and does not alter the original three seeded Planning Dates or baseline answers. Baseline MRP-R2 quantities still produce `0 + 15 + 45 = 60 kg` of sequential shortages and the existing earliest deadline.
- **Scenario isolation and shared history:** A physical receipt increases only its owning scenario's stock and creates one linked global descriptive history entry. Shared summaries may change in other scenarios; stock never transfers through the history. Reset restores the complete documented baseline.

## Cross-source checks

- The brief and PRD now agree on four gated increments: planning core; physical receipts/consumption with linked history; supplier comparison/risk; optional AI. Increment 1 remains the minimum submission; Increment 2 has its own lifecycle gate.
- The AI contract uses current stock and only outstanding production in weekly evidence. Its retained `weekly-starting-inventory` wire name is explicitly defined as stock at the current calculation's start, not historical Opening Balance.
- Receipt/completion confirmations and allocation warnings remain deterministic UI. Stock, shared-history, scenario-switch, and reset events have explicit late-response/cache verification requirements; no new AI authority or operational action is introduced.
- The MRP packaging scenario and historical MRP specification preserve valid recommendation quantities when only lead time is missing. The AI specification's historical matrix is expressly qualified by the same current partial-result rule.
- `spec-prioritize-v1-scope.md` preserves the original three-increment decision as history and explicitly points to the current four-increment brief/PRD. Its old increment numbers are not current release instructions.
- Batch tracing, individual posting reversal, production-consumption deviations, date-effective future supply, historical balance reconstruction, and cross-week time-phased planning remain deferred. This review does not demand those capabilities or reinterpret the current-state selected-week demo as a full ERP.

## Evidence and limits

Read the current PRD, brief, calculation contract, AI contract, three historical specifications, and deferred-work document. Independent Python Decimal/date assertions verified the lifecycle material amounts, 980/220 balance example, 300-cup allocation-conflict example, 60 kg sequential shortage, four-day lead time, and one-day lateness. `git diff --check` passed.

These are documentation arithmetic/date assertions, not application tests. Architecture choices, runtime behavior, receipt/consumption transactions, and fake-provider regression tests still need implementation and verification in the corresponding build increments. Project-account access for the configured optional model remains the documented pre-integration check rather than a claim established by this review.
