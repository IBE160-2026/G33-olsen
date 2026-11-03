# Reconciliation: Final Product Brief, 2 November 2026

## Inputs and result

Compared the finalized `brief.md` and its append-only `.memlog.md` with the existing PRD and PRD memory log, including calculation contract MRP-R2 and its weekly sequencing fixture. The latest approved brief refines the existing product rather than changing the arithmetic or adding a reservation/order system. Preserve FR IDs and existing scenario outcomes.

## Required PRD updates

1. **Make inventory visibility explicit (§3, FR-6, FR-8).** Define On-hand Inventory as the starting physical inventory for the selected snapshot/week, Allocated Quantity before a row as inventory already covered by earlier rows in MRP-R2 order, and Available Quantity before a row as starting inventory minus that cumulative allocation. Require all three labels/values to be visible for each material/result. Preserve Usable Inventory as an explicit alias for the row's available inventory, so FR-5 and linked contracts retain their meaning. Do not count unmet earlier demand as allocation of nonexistent stock.
2. **Expose shortage follow-up (§2.2, UJ-1, FR-12).** The user-approved flow is inventory → requirement/shortage → timing assessment → buyer communication outside the application → manual quantity/date edit and recalculation. State that the shortage view includes affected production date, missing material and amount. Existing FR-2 remains the edit capability and retains the selected-week date restriction. No new messaging, workflow role or automatic scheduling feature is required.
3. **Separate planning assessment from supplier confirmation (§3 Fixed Lead Time, FR-7, FR-12).** The deadline and order-today/overdue state use the configured fixed lead time; they do not prove current supplier availability or promise delivery. Require a clear deterministic qualification in the result/standard explanation, especially when the order deadline has passed. The buyer confirms timely supply with the supplier. Do not infer a new ETA or consume supplier history in arithmetic.
4. **Strengthen small-step build gates (§9, DAR-2).** Current increment gates already match the brief. Add that inventory/allocation is checked against the weekly fixture before dependent shortage behavior, and quantities/deadlines are verified before supplier follow-up or AI. This is sequencing and evidence, not an extra product feature or new implementation architecture.
5. **Preserve the usability hypothesis (§1 and SM-2).** The intended advantage over spreadsheets is a unified, automatically recalculated view that makes consequences easier to understand. It is unproven; do not claim measured superiority. Extend SM-2 to identify affected production and distinguish on-hand/allocation/availability, alongside existing shortage, cause, quantity, deadline and next option. Existing user testing can assess understanding without adding a new competitive benchmark.

## Allocation acceptance grounded in the existing fixture

For MRP-R2's 75 kg starting inventory, require cumulative prior allocation / available-before-row values of **0 / 75**, **60 / 15**, and **75 / 0 kg** for rows a, b and c respectively. Row coverage is **60, 15, 0 kg**; row shortages remain **0, 15, 45 kg**. Cumulative allocation never exceeds the 75 kg physical stock even though cumulative gross demand reaches 135 kg. The weekly recommendation remains **60 kg** and the earliest deadline remains **2026-11-07**.

## Conflicts and boundaries

- PRD Non-Goals currently says “reservations” without qualification. Change that to **manual or external reservations** so it does not appear to forbid the newly explicit, already-existing automatic allocation within the selected week's plan.
- No conflict with the existing manual edit rule, no-login role definition, fixed lead-time arithmetic or supplier-history separation was found.
- Keep incoming orders outside supply calculations and keep automatic rescheduling/contact/message sending outside v1.
- Existing A-1 reset confirmation and A-2 scenario isolation remain unratified assumptions in the PRD. The new brief does not independently approve either; resolve them before final status.
- The PRD's exact AI provider/model policy is unchanged by this brief update.

## Memory-log reconciliation

The November decisions should be appended to the PRD memory log as the reason for these updates. Older brief entries about login, trend dashboards, forecasting and a broad dairy catalogue remain historical context superseded or deferred by the current brief; they must not be reintroduced as v1 commitments. Detailed formulas and AI controls remain in the linked authoritative contracts; no new addendum is needed for this input.
