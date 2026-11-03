# PRD Domain Consistency — 2026-11-02

## Verdict

The November PRD changes are consistent with the approved brief and existing calculation and AI contracts within the requested review scope. No blocking domain inconsistency was found. This is a document consistency check, not an implementation or execution test.

## Inventory and allocation

- The glossary and FR-6 define On-hand Inventory as starting stock and Allocated Quantity as stock actually covering earlier rows, not the total of earlier demand. Allocation is capped at starting stock and cannot manufacture negative availability.
- Available Quantity is the inventory input immediately before the current row. FR-8 explicitly prevents starting stock from being reused in every row's Net Need calculation.
- The MRP-R2 fixture has starting stock 75 kg, covered quantities 60, 15, and 0 kg, and sequential shortages 0, 15, and 45 kg. Therefore the PRD's pre-row allocation/availability pairs are correctly 0/75, 60/15, and 75/0 kg. The weekly recommendation remains 60 kg; no arithmetic or acceptance result changes.
- The clarified manual/external-reservation exclusion in §8 preserves automatic within-week allocation. The calculation contract's older word “reservations” is read together with its explicit MRP-R2 sequential-consumption rule, not as an exclusion of that rule.

## Delivery assessment and manual follow-up

- FR-7 correctly qualifies Fixed Lead Time as a planning assumption rather than supplier confirmation. An overdue deadline does not prove expedited delivery impossible and does not alter the recommended quantity.
- UJ-1 and FR-12 keep communication outside the application and changes manual. FR-2's existing selected-week date boundary remains in force. There is no automatic supplier contact, message sending, purchasing, or rescheduling.
- The qualification is a product display requirement; it does not require provider-created prose or expansion of the closed AI template/option schema. Existing server-owned explanations and buyer options remain authoritative.

## Partial results and risk

- FR-9 preserves quantities when lead time is invalid, preserves quantities and valid dates when Planning Date is invalid, and prevents unsupported urgency or AI generation.
- FR-7 preserves all-or-unavailable aggregate deadlines when any shortage row lacks deadline input. SM-7 explicitly retains the 60 kg recommendation in this case.
- FR-17 and the AI contract preserve the valid-current-deadline requirement for Supplier Risk; unavailable risk never invents evidence or a deadline.
- Allocation visibility adds no open-order supply, live stock integration, historical-lead-time substitution, or provider dependency.

## Assumptions

A-1 and A-2 were explicitly confirmed by the user on 2026-11-02. The requirements retain reset confirmation and isolated scenario planning snapshots with global supplier history. No new domain decision is needed for the changes reviewed here.

## Findings

None requiring a correction before UX or architecture within this focused scope.
