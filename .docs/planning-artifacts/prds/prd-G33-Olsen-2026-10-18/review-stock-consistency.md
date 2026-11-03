# Stock Consistency Review — ProsjektMelk

**Reviewed:** 2026-11-03. Resumed sequentially by the primary assistant after the parallel review was interrupted; document and arithmetic review only.

## Verdict

The prior stock-contract contradictions are resolved in the reconciled documents. No blocking documentation finding remains in this pass. This is not evidence that the application or its planned automated tests exists or passes.

## Resolution of the previous findings

| Previous finding | Current resolution |
|---|---|
| SC-01 — Starting versus current stock | PRD glossary/FR-6 and MRP-R2 use current stock = opening + posted accepted receipts − posted physical consumption. Only outstanding production enters new allocation and need. |
| SC-02 — Receipt/history linkage | FR-13/FR-25 and MRP-R8 specify one scenario-owned receipt with one linked global descriptive record, atomic posting and once-only replay. Unknown order date is explicit and excluded from statistics. |
| SC-03 — Receipt date cutoff | Actual Receipt Date must be valid and no later than the scenario Planning Date. Confirmed Delivery Date is a separate optional promise and can be future. Current-state views do not reconstruct old balances or add expected future supply. |
| SC-04 — Receipt identity and bounds | MRP-R8 defines canonical units, exact three-decimal limits, result bounds, stable identity and conflicting replay. Existing identities are resolved before new-posting stock validation. |
| SC-05 — Release/tests | Increment 2 and SM-9 require the operational gate. Increment 3 owns comparison/risk and Increment 4 optional AI. Existing reset baselines are preserved. |
| SC-06 — Missing lead time | Packaging fixture and older specification now preserve valid quantity while withholding only the unsupported deadline. |

## Full fixture check

The independent operational fixture uses Planning Date and production date 2026-11-12; it is not an extra selectable UI scenario.

- Opening stock: 140 kg puree, 650 cups, 850 seals, 900 lids.
- 800 cups require 120 kg puree and 800 of each packaging component.
- Receipt 500 cups, 20 rejected: 480 accepted and 1,130 physical cups.
- Order 2026-11-08, confirmation 2026-11-11, actual receipt 2026-11-12: four-day actual lead time, one-day delay.
- Completion leaves 20 kg puree, 330 cups, 50 seals, and 100 lids.
- Completed history stays visible and read-only; outstanding demand is zero and consumption is not deducted again.
- The independent balance example 1,000 + 480 − 500 = 980 and outstanding 1,200 − 980 = 220 is consistent.
- The allocation-override example leaves 500 physical cups and a 300-cup shortage for the other 800-cup run.
- Cancel, failure, replay, scenario switching, pre-reset pending actions, and confirmed reset have explicit invariants.

## Final clarifications applied

Completion depends on quantity/date, BOM/units, and physical stock, not purchase lead time/history/provider availability. Equivalent receipt/completion retries are resolved before treating them as new postings. A stale preview requires refreshed confirmation. A pre-reset action cannot post into the restored baseline.

## Limits

Date-effective supply across weeks, actual-consumption deviations, batch tracing, and individual reversals remain outside scope. No actual receipt, inventory posting, provider call, or application test was executed.

Finding counts: critical 0, high 0, medium 0, low 0.
