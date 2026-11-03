# MRP Calculation Rules and Demo Scenarios

## Purpose and Scope

This document is the calculation contract for the first version of ProsjektMelk. It defines the deterministic numbers that the application displays and that the three demo scenarios must produce. All names, suppliers, materials, and quantities below are fictional.

The model plans one finished product at a time: **Blueberry Yogurt, 150 g cup**. Its one-level bill of materials (BOM) contains only raw material or packaging; it does not explode a material into further components. The application is decision support: it recommends and explains a purchase need, but never creates or places an order.

## Shared Data Assumptions

| Product | Unit | Material | BOM quantity per product unit | Material unit | Fictional supplier |
|---|---:|---|---:|---|---|
| Blueberry Yogurt, 150 g cup | cup | Blueberry puree | 0.15 | kg | Nordic Berry Ingredients AS |
| Blueberry Yogurt, 150 g cup | cup | Yogurt cup | 1 | cup | PackForm AS |
| Blueberry Yogurt, 150 g cup | cup | Yogurt foil seal (oblat) | 1 | seal | PackForm AS |
| Blueberry Yogurt, 150 g cup | cup | Yogurt lid | 1 | lid | DairyPack AS |

- `production quantity` is the planned number of finished cups for one production date.
- `planning date` is a seeded date-only value for the active demo scenario. Scenario calculations never use the computer clock.
- `opening balance` is historical baseline stock for a material in a scenario. Current physical stock equals opening balance plus posted accepted receipts minus posted physical consumption. Rejected quantities never increase stock, and history does not add stock again.
- `available inventory` for an outstanding production row is current physical stock minus stock allocated to earlier outstanding rows in the selected week's calculation order. Allocation does not itself deduct physical stock. V1 does not subtract manual/external reservations or add open purchase orders.
- Each supplier-material combination has one fixed lead time expressed as a non-negative whole number of calendar days. Delivery history is displayed descriptively; it never changes the fixed lead time or a calculated deadline.
- V1 has no safety stock, minimum order quantity, or order multiple.
- A complete delivery record has an **Order Sent Date** and **Actual Receipt Date**. **Confirmed Delivery Date** is the optional supplier-promised arrival date, for example manually copied from a fictional order confirmation; it is not proof of receipt. These labels map to the existing order-sent, delivery-received, and promised-date fields respectively. V1 has no email integration.
- Selected weeks use ISO-8601 Monday–Sunday boundaries. Every planning, production, order-sent, promised, received, and deadline input must be a real Gregorian date in exact `YYYY-MM-DD` form and remains a plain calendar date; calculations do not convert through local timestamps or daylight-saving rules.
- Production quantities are whole cups from 1 through 1,000,000. A BOM quantity is greater than zero and at most 1,000,000.000; inventory is from zero through 1,000,000,000,000.000. BOM, inventory, need, and recommendation quantities are finite base-10 decimals with at most three fractional digits, and every derived material quantity must remain at or below 1,000,000,000,000.000. Calculations use exact decimal arithmetic, never round intermediate values, and display final quantities with up to three fractional digits and no trailing zeroes.
- V1 canonical unit codes are `cup`, `kg`, `seal`, and `lid`. A BOM material unit is compatible only when its canonical unit code exactly equals the Material's canonical unit code; v1 performs no unit conversion.

## Requirement and Ordering Rules

**MRP-R1 — Per-plan requirement rule**

For every material in the product's one-level BOM:

| Result | Rule | Interpretation |
|---|---|---|
| Gross need | `production quantity × BOM quantity per product unit` | Material required for the planned production run. |
| Net need | `max(gross need − available inventory, 0)` | Quantity still uncovered by usable on-hand inventory. |
| Recommended order quantity | `net need`, only when `net need > 0` | Exact positive shortage to recommend. When net need is zero, show no purchase recommendation. |
| Latest safe order date | `production date − fixed lead time in calendar days` | The buyer must place the order on or before this date for the material to arrive by the production date under the standard lead-time rule. |

For example, a plan for 1,500 cups needs `1,500 × 0.15 = 225 kg` of blueberry puree. With 180 kg available, net need is `max(225 − 180, 0) = 45 kg`; the recommendation is 45 kg.

## Weekly Planning and Inventory Consumption

**MRP-R2 — Weekly inventory-consumption rule**

For a selected week, the application groups outstanding production-plan rows by material and sums their gross needs. Completed rows remain in history but are excluded from outstanding demand and allocation. It allocates each material's current physical stock in ascending production-date order. If two rows have the same production date, it uses ascending product name and then stable row ID as tie-breakers. A row's covered quantity reduces the inventory available to every later row, so inventory cannot be counted twice. This is planned allocation, not physical consumption. The resulting recommendation and deadline attach only to the scenario's configured supplier-material relationship; alternative comparison suppliers do not enter MRP arithmetic.

The weekly recommended quantity is the sum of all positive sequential shortages for that supplier-material combination; it equals `max(total outstanding weekly gross need − current physical stock, 0)`. If every row with a positive sequential shortage has a valid configured supplier and Fixed Lead Time, the weekly latest safe order date is the earliest deadline among those rows. If any positive-shortage row lacks valid deadline input, the aggregate deadline is `deadline-unavailable`: all valid quantities remain visible, the applicable validation message is shown, and no deadline is inferred from the remaining rows. Existing baseline fixtures have no posted movements or Completed rows, so their answers remain unchanged.

**Current-balance acceptance:** 1,000 opening cups + 480 accepted cups − 500 posted consumed cups = 980 current physical cups. An outstanding 1,200-cup run has a 220-cup shortage. Recalculation never subtracts the Completed run again. The opening balance and movements remain history. Receipt-date eligibility is defined by MRP-R8; this rule does not make future expected receipts available stock or reconstruct historical balances.

## Data Validation and Alert Basis

**MRP-R3 — Calculation validation and partial-result rule**

The application must show the inputs and arithmetic behind every alert: product, production date, production quantity, BOM quantity, gross need, available inventory, net need, supplier-material lead time, and latest safe order date when applicable.

A recommendation is blocked, rather than guessed, when an input required for that result is missing or invalid:

| Missing or invalid input | Blocked output | Message to the buyer |
|---|---|---|
| Production date or production quantity is missing, production date is not a real Gregorian `YYYY-MM-DD` date, or production quantity is zero or negative | Gross and net need, recommendation, and deadline | `Cannot calculate: a valid production date and a positive production quantity are required.` |
| Production quantity is fractional | Gross and net need, recommendation, and deadline | `Cannot calculate: production quantity must be a whole number of cups.` |
| Production quantity, BOM quantity, inventory, or derived material quantity exceeds its product-level maximum | Affected downstream calculation | `Cannot calculate: <field> exceeds the supported maximum for <material>.` |
| BOM quantity is zero or negative | Gross and net need; recommendation | `Cannot calculate: BOM quantity must be positive for <material>.` |
| Material and BOM units are incompatible | Gross and net need; recommendation | `Cannot calculate: incompatible units for <material>.` |
| BOM quantity | Gross and net need; recommendation | `Cannot calculate: BOM quantity is missing for <material>.` |
| Available inventory is missing or negative | Net need and recommendation | `Cannot calculate: available inventory must be zero or greater for <material>.` |
| Fixed supplier-material lead time is missing, negative, or fractional | Latest safe order date only | `Quantity can be calculated, but deadline is unavailable: lead time must be a whole number of days, zero or greater, for <supplier> and <material>.` |
| Planning Date is missing or is not a real Gregorian `YYYY-MM-DD` date | Deadline urgency state only | `Deadline urgency is unavailable: a valid Planning Date is required.` |
| Edited production date is outside the selected ISO week | Save and recalculation for that edit; retain the previous valid date | `Cannot update production date: date must remain within selected week <week>.` |

A zero or negative result after the net-need formula is displayed as net need `0`; it creates no purchase recommendation. A missing or invalid input is not treated as zero. Crucially, when BOM quantity and inventory are complete and valid, a missing or invalid lead time blocks **only** the latest safe order date: the application still shows the calculated net need and a positive quantity recommendation with its named supplier. A missing or invalid Planning Date does not block a valid quantity or Latest Safe Order Date; it blocks only comparison of that deadline to the Planning Date and therefore produces no `future`, **Order today**, or **Overdue** state. A result with unavailable deadline or urgency state is ineligible for AI generation.

**MRP-R4 — Deadline urgency rule**

For a valid positive recommendation with a valid deadline, compare the deadline with the planning date. A deadline equal to the planning date is displayed as **Order today**. A deadline before the planning date is displayed as **Overdue by _N_ calendar days** and retains the calculated quantity; it does not silently move or remove the recommendation.

## Delivery-History Rule

**MRP-R5 — Delivery validation and statistics rule**

Delivery history is descriptive only. Validate a record in this order: (1) when order-sent or delivery-received date is missing, label it `Incomplete delivery record`; (2) when any supplied date is not a real Gregorian date in exact `YYYY-MM-DD` form, label it `Invalid delivery date`; (3) otherwise, when promised date or delivery-received date precedes order-sent date, label it `Invalid delivery date sequence`; (4) otherwise the record is valid. Invalid and incomplete records are excluded from every delivery statistic and risk assessment.

For a valid complete delivery record, actual lead time is `delivery-received date − order-sent date` in calendar days. A delivery is late when it is received after its promised date; if no promised date was recorded, it is late when its actual lead time exceeds the fixed supplier-material lead time. The delay is the number of calendar days beyond that promised date or standard lead time. If a valid record has no promised date and that supplier-material Fixed Lead Time is missing or invalid, retain the record in valid-delivery count and average actual lead time, but mark late-delivery count, greatest delay, and Supplier Risk unavailable for that supplier-material summary. Show `Late-delivery statistics are unavailable: lead time is required for <supplier> and <material> when promised date is absent.`

Incomplete records are marked `Incomplete delivery record` and excluded from average lead-time and late-delivery statistics. For each supplier-material combination, the descriptive historical summary contains: count of valid complete deliveries, average actual lead time (`sum of actual lead-time days ÷ valid complete delivery count`, rounded decimal-half-up to one decimal), count of late deliveries, and greatest recorded delay in calendar days. If there are no valid complete deliveries, display `No valid delivery history` rather than a statistic. These values do not alter a recommendation, quantity, or latest safe order date.

**MRP-R6 — Supplier-risk rule**

A supplier-delay risk warning appears only when all three conditions hold: (1) that supplier-material combination has at least one valid complete late delivery with a positive delay, (2) the current plan has a positive net need for the same material, and (3) the current Latest Safe Order Date is valid. The warning states the greatest recorded delay and the risk calculation. When `latest safe order date + greatest recorded delay` is on or after the affected production date, state that a repeat delay could reach or miss the production date; otherwise state that the delay is historical supplier risk. If conditions 1 and 2 hold but the current deadline is unavailable, show the delivery summary but no Supplier Risk and append `Supplier Risk is unavailable: a valid current deadline is required to compare historical delay.` The result is AI-ineligible under MRP-R3. This rule is descriptive only.

### Supplier-comparison fixture

Increment 3 compares two fictional suppliers for **Yogurt lid** without scoring, ranking, recommending, or switching supplier automatically. DairyPack AS remains the configured supplier for the worked scenarios with its 3-day Fixed Lead Time. PackForm AS is an alternative lid supplier with a 4-day Fixed Lead Time and these valid complete records:

| Supplier | Material | Order sent | Promised | Received | Actual lead time | Result |
|---|---|---|---|---|---:|---|
| PackForm AS | Yogurt lid | 2026-09-01 | 2026-09-05 | 2026-09-05 | 4 days | On time (`delivery-packform-lid-1`) |
| PackForm AS | Yogurt lid | 2026-09-15 | 2026-09-19 | 2026-09-20 | 5 days | 1 day late (`delivery-packform-lid-2`) |

PackForm AS therefore has 2 valid complete lid deliveries, 4.5 days average actual lead time, 1 late delivery, and 1 day greatest recorded delay. The comparison shows these facts beside DairyPack AS facts only. It never changes the scenario's configured supplier, 100-lid recommendation, or 2026-11-07 deadline.

## Worked Demo Scenarios

**MRP-R7 — Named acceptance fixtures**

The three scenarios are isolated, selectable planning snapshots rather than simultaneous states of one plan. The dashboard exposes `scenario-plan-increase`, `scenario-packaging-shortage`, and `scenario-delayed-delivery`. A fresh or reset scenario loads its own Planning Date, initial Production Plan, opening inventory, and configured supplier relationships. Returning to a previously changed scenario restores its saved state without replaying stock movements. Supplier Delivery Records are one global fictional history shared by all scenarios: `delivery-dairypack-lid-1`, `delivery-packform-lid-1`, and `delivery-packform-lid-2` are available in the comparison from every scenario. **Reset demo data** requires confirmation, restores all three planning snapshots and global supplier history, and activates `scenario-plan-increase` at its initial plan. MRP-R10 defines the stock-history boundary.

### 1. Production-plan increase

Planning date: 2026-11-03. Selection loads an `initialPlan` of **1,000 cups** for production on 2026-11-10. The acceptance action changes that row to the `expectedPlan` of **1,500 cups**; the calculation uses the resulting total plan of 1,500 cups.

| Material | BOM per cup | Gross need calculation | Available inventory | Net need calculation | Fixed lead time | Expected result |
|---|---:|---:|---:|---:|---:|---|
| Blueberry puree | 0.15 kg | `1,500 × 0.15 = 225 kg` | 180 kg | `225 − 180 = 45 kg` | 3 days | Recommend 45 kg; latest safe order date **2026-11-07**. |
| Yogurt cup | 1 cup | `1,500 × 1 = 1,500 cups` | 1,600 cups | `max(1,500 − 1,600, 0) = 0` | 4 days | No purchase recommendation. |
| Yogurt foil seal (oblat) | 1 seal | `1,500 × 1 = 1,500 seals` | 1,550 seals | `max(1,500 − 1,550, 0) = 0` | 4 days | No purchase recommendation. |
| Yogurt lid | 1 lid | `1,500 × 1 = 1,500 lids` | 1,700 lids | `max(1,500 − 1,700, 0) = 0` | 3 days | No purchase recommendation. |

**Expected warning:** Blueberry puree requires a purchase recommendation because the increased plan needs 225 kg while only 180 kg is available. The buyer sees the 45 kg shortfall and the 2026-11-07 deadline. Buyer options are to order 45 kg by the deadline, change the plan, or investigate usable inventory; the system does not order automatically.

### 2. Packaging shortage

Planning date: 2026-11-04. Selection loads an `initialPlan` of **600 cups** for production on 2026-11-12. The acceptance action changes that row to the `expectedPlan` of **800 cups**.

| Material | BOM per cup | Gross need calculation | Available inventory | Net need calculation | Fixed lead time | Expected result |
|---|---:|---:|---:|---:|---:|---|
| Blueberry puree | 0.15 kg | `800 × 0.15 = 120 kg` | 140 kg | `max(120 − 140, 0) = 0 kg` | 3 days | No purchase recommendation. |
| Yogurt cup | 1 cup | `800 × 1 = 800 cups` | 650 cups | `800 − 650 = 150 cups` | 4 days | Recommend 150 cups; latest safe order date **2026-11-08**. |
| Yogurt foil seal (oblat) | 1 seal | `800 × 1 = 800 seals` | 850 seals | `max(800 − 850, 0) = 0` | 4 days | No purchase recommendation. |
| Yogurt lid | 1 lid | `800 × 1 = 800 lids` | 900 lids | `max(800 − 900, 0) = 0` | 3 days | No purchase recommendation. |

**Expected warning:** Yogurt cups are the affected packaging material: 150 cups are short. The buyer sees the calculation and must decide whether to order 150 cups by 2026-11-08, change the production plan, or verify stock. Missing cup BOM or cup inventory blocks the affected quantity and recommendation under MRP-R3. Missing or invalid supplier lead time preserves the valid 150-cup recommendation and blocks only its deadline, using the exact MRP-R3 deadline-unavailable message.

### 3. Delayed supplier delivery

Planning date: 2026-11-03. Selection loads an `initialPlan` of **500 cups** for production on 2026-11-10. The acceptance action changes that row to the `expectedPlan` of **600 cups**. Yogurt lids are supplied by the fictional supplier DairyPack AS with a fixed lead time of 3 calendar days.

| Material | BOM per cup | Gross need calculation | Available inventory | Net need calculation | Fixed lead time | Expected result |
|---|---:|---:|---:|---:|---:|---|
| Blueberry puree | 0.15 kg | `600 × 0.15 = 90 kg` | 110 kg | `max(90 − 110, 0) = 0 kg` | 3 days | No purchase recommendation. |
| Yogurt cup | 1 cup | `600 × 1 = 600 cups` | 700 cups | `max(600 − 700, 0) = 0` | 4 days | No purchase recommendation. |
| Yogurt foil seal (oblat) | 1 seal | `600 × 1 = 600 seals` | 650 seals | `max(600 − 650, 0) = 0` | 4 days | No purchase recommendation. |
| Yogurt lid | 1 lid | `600 × 1 = 600 lids` | 500 lids | `600 − 500 = 100 lids` | 3 days | Recommend 100 lids; latest safe order date **2026-11-07**. |

The recorded DairyPack AS lid delivery (`delivery-dairypack-lid-1`) was ordered on 2026-10-01, promised for 2026-10-04, and received on 2026-10-07. Its actual lead time is `2026-10-07 − 2026-10-01 = 6 days`; it is **3 days late** against both the promised date and the 3-day standard lead time.

**Expected warning:** The buyer sees a 100-lid recommendation and its unchanged 2026-11-07 deadline, plus a supplier-delay risk: a delivery ordered on that deadline is expected on 2026-11-10 under the fixed lead time, but would arrive on 2026-11-13 if it repeated the recorded three-day delay. Buyer options are to contact DairyPack AS, seek an expedited or alternative supply, change the plan, or accept the risk. The historical delay explains the risk only; it does not change the 100-lid quantity or the fixed-lead-time deadline. A record missing either order-sent or delivery-received date is labelled incomplete and excluded from delivery statistics.

## Weekly Sequencing Acceptance Fixture

This focused fixture tests MRP-R2 independently of the three single-row scenarios. For ISO week `2026-W46`, Blueberry puree starts with **75 kg** of Usable Inventory and has a 3-day Fixed Lead Time. The rows deliberately include a same-date tie and must be processed in the order shown: production date, product name, then stable row ID.

| Stable row ID | Production date | Product | Planned cups | Gross need | Inventory before | Covered | Sequential shortage | Inventory after | Row deadline when short |
|---|---|---|---:|---:|---:|---:|---:|---:|---|
| `row-week-a` | 2026-11-10 | Blueberry Yogurt, 150 g cup | 400 | 60 kg | 75 kg | 60 kg | 0 kg | 15 kg | Not applicable |
| `row-week-b` | 2026-11-10 | Blueberry Yogurt, 150 g cup | 200 | 30 kg | 15 kg | 15 kg | 15 kg | 0 kg | 2026-11-07 |
| `row-week-c` | 2026-11-12 | Blueberry Yogurt, 150 g cup | 300 | 45 kg | 0 kg | 0 kg | 45 kg | 0 kg | 2026-11-09 |

The hand-calculated total Gross Need is **135 kg**. The weekly recommendation is `15 + 45 = 60 kg`, which also equals `max(135 − 75, 0) = 60 kg`. The weekly earliest shortage deadline is **2026-11-07**. An automated test must also invalidate one positive-shortage row's lead time and verify that the 60 kg quantity remains visible while the aggregate deadline becomes `deadline-unavailable`; it must not fall through to 2026-11-09.

An end-to-end week-boundary test must select `2026-W46` (Monday 2026-11-09 through Sunday 2026-11-15), accept edits to either boundary date, reject 2026-11-08 and 2026-11-16 with the exact MRP-R3 message, retain the previous valid date, and leave weekly totals unchanged after rejection.

## Goods Receipt and Physical Consumption Rules

**MRP-R8 — Single goods-receipt posting rule (Increment 2)**

One registration contains a stable receipt ID, owning scenario, Material, supplier, canonical unit, **Actual Receipt Date**, total received quantity, deviation/rejected quantity, **Order Sent Date** or an explicit **Order Sent Date unknown** choice, and optional **Confirmed Delivery Date**. Accepted quantity is `total received − rejected quantity`. A confirmed delivery date is a promise and may be after the Planning Date or Actual Receipt Date; early delivery is not an error. It never adds expected supply to current stock.

Before applying current-state validation, resolve an existing receipt identity: an equivalent replay returns its original result without posting again, while conflicting content is rejected. Current stock limits must not make an already-posted equivalent replay look like a new receipt. Validate genuinely new postings as follows:

1. Require a known scenario, Material, supplier, matching canonical Material unit, a valid Actual Receipt Date, and a valid scenario Planning Date. Both dates are strict Gregorian `YYYY-MM-DD` values. An Actual Receipt Date after the Planning Date is rejected. Do not replace the seeded Planning Date with the computer clock.
2. Total received must be greater than zero and at most 1,000,000,000,000.000; rejected quantity must be zero or greater and no greater than total received. Both use finite exact base-10 values with at most three fractional digits. Accepted quantity and resulting current physical stock must remain within zero through 1,000,000,000,000.000. Invalid units, precision, range, or arithmetic block the complete posting.
3. The buyer must either supply a valid Order Sent Date or explicitly select **Order Sent Date unknown**. An empty field without that explicit choice is not a fabricated date and must be corrected or marked unknown before posting. Known supplied dates must be real Gregorian dates. A known Order Sent Date after Actual Receipt Date, or a supplied Confirmed Delivery Date before a known Order Sent Date, blocks posting. The buyer must correct the supplied value or actively mark Order Sent Date unknown; the application never silently converts an invalid value to unknown. Selecting unknown removes the previously entered Order Sent Date from the posting payload. A malformed optional Confirmed Delivery Date must be corrected or cleared, not silently ignored.
4. A new receipt ID posts accepted stock once in its owning scenario and creates exactly one linked global Delivery Record with the same Material, supplier, and delivery fields. Both effects and their stable one-to-one reference succeed atomically or neither occurs. Same ID and equivalent normalized payload returns the existing result without a stock or statistics change. Reusing the ID with different owning scenario, Material, supplier, dates, unknown-date choice, unit, or quantities rejects the posting without changing either record. Two distinct receipt IDs with equal quantities are independent receipts, not automatically duplicates.

The buyer-facing receipt errors include these exact messages:

| Invalid input | Message |
|---|---|
| Missing or invalid Actual Receipt Date or Planning Date | `Cannot post receipt: valid receipt and Planning Dates are required.` |
| Actual Receipt Date after Planning Date | `Cannot post receipt: receipt date must not be after the scenario Planning Date.` |
| Missing Order Sent Date with unknown not selected | `Cannot post receipt: enter Order Sent Date or select Order Sent Date unknown.` |
| Supplied malformed delivery date | `Cannot post receipt: supplied delivery dates must be valid Gregorian dates.` |
| Known order/receipt/confirmation sequence is contradictory | `Cannot post receipt: delivery dates conflict; correct them or mark Order Sent Date unknown.` |
| Invalid received/deviation quantity, precision, unit, or resulting balance | `Cannot post receipt: valid quantities in the Material unit and supported range are required.` |
| Existing receipt ID with different content | `Cannot post receipt: receipt ID already exists with different content.` |

Known-date postings produce a Delivery Record evaluated by MRP-R5. Explicitly unknown Order Sent Date does not block accepted stock: retain the linked record as `Incomplete delivery record`, visibly show the missing date, and exclude it from every supplier statistic and Supplier Risk. Do not infer the date from Fixed Lead Time or Actual Receipt Date. Existing valid delivery history continues to contribute normally. The single posting may change descriptive supplier summaries in other scenarios because history is shared; it must never add stock there. Seeded historic deliveries are already reflected in opening stock and must never be reposted as new receipts.

**MRP-R9 — Confirmed production-consumption rule (Increment 2)**

**Complete production** previews the current production quantity, date, and each exact `production quantity × BOM quantity` consumption. Posting requires an outstanding row with a valid quantity and production date, valid BOM quantities and compatible units, and valid sufficient physical stock for every BOM Material. Fixed Lead Time, a purchase deadline, supplier delivery history, and provider configuration are not consumption inputs; their absence does not block an otherwise valid completion. The buyer must confirm; cancel changes nothing. Revalidate current physical stock and production state at confirmation so a stale preview cannot post an obsolete or insufficient-stock calculation.

Resolve an already completed row's posting before validating a new consumption: an equivalent repeat returns the recorded completion without rechecking it as new demand. If quantity, BOM, or stock facts changed since a still-unposted preview, show the refreshed amounts and require confirmation again. Conflicting attempts cannot overwrite an existing posting.

- If any Material lacks sufficient current physical stock, reject the entire completion, identify the Material and physical shortage, and change neither stock, production state, nor consumption history. Check all BOM Materials before making any deduction.
- If physical stock covers all consumption but another outstanding production has an allocation conflict, preview the affected Material, conflicting allocated quantity, affected production, and resulting shortages. The buyer can cancel or explicitly confirm proceeding despite the conflict. An allocation conflict alone does not block completion. Do not automatically edit other production quantities or dates.
- Confirmation atomically records one consumption posting associated with the production's stable row ID, deducts every BOM Material once, and marks the row **Completed**. A repeated completion returns the existing completion without another deduction. A concurrent or stale action must not replace an already posted consumption with different content.
- Preserve posting-time quantity, production date, Material units, and consumed quantities in history. The Completed row remains visible with read-only quantity/date and accessible consumption history; it contributes no outstanding Gross Need, allocation, Net Need, recommendation, or deadline. Recalculate only outstanding rows from the new current physical balance.
- No undo, reopening, or direct edit of posted consumption is provided in v1. Full demo reset is separate. Actual consumption equals the confirmed BOM-derived amount; production waste and actual-versus-planned adjustments, batch tracking, and individual reversals are deferred.

**MRP-R10 — Current-state scenario, history, and reset boundary**

Each scenario owns its Opening Balances, posted receipts/deviations, current physical stock, completed-production state, and consumption history. Scenario switching or restart restores that saved state; it never copies, replays, or deducts movements. Shared supplier Delivery Records are descriptive evidence and cannot transfer physical stock across scenarios. The selected week's outstanding plan is evaluated against the current scenario stock; selecting a week is not a historical/as-of balance reconstruction or an additional source of stock.

A confirmed **Reset demo data** restores all scenario opening balances and initial uncompleted plans, removes newly posted receipts/deviations and consumption, restores the three baseline global Delivery Records, and activates `scenario-plan-increase`. Canceling reset preserves all state. Receipt dates are audit data subject to MRP-R8 eligibility; expected future supplies, retrospective balance reconstruction, and time-phased supply across weeks remain deferred. The original three baseline fixtures remain unchanged when no new stock movements are posted.

A receipt or completion submitted from a retired pre-reset context must not post into the restored baseline. Its owning scenario cannot silently change when the active selector changes. Tests must hold an action pending across scenario switch or reset and verify that it cannot create a movement in the wrong or restored context.

## Full Stock-Lifecycle Acceptance Fixture

This independent automated-test fixture exercises Increment 2; it is not a fourth selectable UI scenario and does not change any of the three named baseline Planning Dates. It uses the same four-line BOM and opening quantities as the packaging-shortage example, with its own seeded Planning Date **2026-11-12**, selected week **2026-W46**, and one outstanding **800-cup** run dated **2026-11-12**. Supplier-history starting counts for PackForm AS / Yogurt cup are zero in this isolated test fixture.

| Material | Opening physical stock | Gross Need for 800 cups | Initial shortage | Physical consumption on completion |
|---|---:|---:|---:|---:|
| Blueberry puree | 140 kg | 120 kg | 0 kg | 120 kg |
| Yogurt cup | 650 cups | 800 cups | 150 cups | 800 cups |
| Yogurt foil seal | 850 seals | 800 seals | 0 seals | 800 seals |
| Yogurt lid | 900 lids | 800 lids | 0 lids | 800 lids |

1. **Before receipt:** previewing completion is blocked by the 150-cup physical shortage. None of the four Material balances, production state, or consumption history changes. The valid 150-cup recommendation remains, with 2026-11-08 deadline under the four-day cup lead time, overdue by four calendar days relative to this fixture's Planning Date.
2. **Post receipt `receipt-lifecycle-cup-1`:** supplier PackForm AS, Yogurt cup, 500 received, 20 rejected, 480 accepted; Order Sent Date 2026-11-08, Confirmed Delivery Date 2026-11-11, Actual Receipt Date 2026-11-12. Current cup stock becomes `650 + 480 = 1,130`. The linked valid Delivery Record has four-day actual lead time and is one day late against confirmation. In this fixture, its summary becomes one valid delivery, 4.0-day average, one late delivery, greatest delay one day. The run's cup shortage becomes zero; no other Material stock changes.
3. **Allocation:** before the only outstanding row, prior allocation is zero and available quantities equal current physical stock. Planning coverage of the 800-cup run does not physically deduct its BOM materials. Preview cancel preserves 140 kg puree, 1,130 cups, 850 seals, and 900 lids.
4. **Confirm completion:** atomically consume 120 kg puree, 800 cups, 800 seals, and 800 lids. Remaining physical balances are **20 kg puree, 330 cups, 50 seals, and 100 lids**. The visible row is **Completed** and read-only. With no outstanding rows, remaining Gross Need, allocation, Net Need, and Recommended Order Quantity are zero; no current shortage deadline or Supplier Risk appears. The one-day-late supplier record stays in history.
5. **Replay, isolation, and reset:** replaying the receipt or completion, reopening the application, or switching away and back must leave these balances and counts unchanged. Other scenarios' stock is unchanged, although global descriptive history may be visible there. Attempts to edit the Completed quantity/date leave all records unchanged. Canceling full reset changes nothing; confirming full reset restores the fixture's opening quantities and uncompleted 800-cup plan and removes its posted receipt, linked Delivery Record, and consumption. Separately, the normal UI reset must still restore all three original named scenarios and their three baseline Delivery Records under MRP-R7.

Additional branches must verify all rejected quantities from 0 through the total (including zero accepted stock), negative/over-total deviations, numeric precision/maxima and result overflow, unknown Order Sent Date with unchanged supplier statistics, malformed/contradictory supplied dates, Actual Receipt Date equal to or before Planning Date versus the next day, same-ID equivalent replay versus different-content rejection, and atomic failure of either half of receipt posting. The allocation-conflict branch uses 1,000 physical cups, an earlier 800-cup run, and a selected 500-cup run: warn and preserve all state on cancel; explicit confirmation leaves 500 physical cups and a 300-cup shortage on the earlier outstanding run. Other BOM Materials must be sufficiently stocked for that branch so it tests allocation conflict rather than physical insufficiency.

## Hand-Check Checklist

For each scenario, a reviewer can verify that the displayed gross need equals planned cups times BOM quantity, net need is never below zero, a recommendation exists only for positive net need, and the latest safe order date equals production date minus the listed fixed lead time. The warning must name the material, reason, numeric result, and buyer options.
