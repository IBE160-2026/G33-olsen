# MRP Calculation Rules and Demo Scenarios

## Purpose and Scope

This document is the calculation contract for the first version of ProsjektMelk. It defines the deterministic numbers that the application displays and that the three demo scenarios must produce. All names, suppliers, materials, and quantities below are fictional.

The model plans one finished product at a time: **Blueberry Yogurt, 150 g cup**. Its one-level bill of materials (BOM) contains only raw material or packaging; it does not explode a material into further components. The application is decision support: it recommends and explains a purchase need, but never creates or places an order.

## Shared Data Assumptions

| Product | Unit | Material | BOM quantity per product unit | Material unit | Fictional supplier |
|---|---:|---|---:|---|---|
| Blueberry Yogurt, 150 g cup | cup | Blueberry puree | 0.15 | kg | Nordic Berry Ingredients AS |
| Blueberry Yogurt, 150 g cup | cup | Yogurt cup | 1 | cup | PackForm AS |
| Blueberry Yogurt, 150 g cup | cup | Yogurt lid | 1 | lid | DairyPack AS |

- `production quantity` is the planned number of finished cups for one production date.
- `available inventory` is the current usable on-hand quantity for that material. V1 does not subtract reservations or add open purchase orders.
- Each supplier-material combination has one fixed lead time expressed in calendar days. Delivery history is displayed descriptively; it never changes the fixed lead time or a calculated deadline.
- V1 has no safety stock, minimum order quantity, or order multiple.
- A complete delivery record has an order-sent date and a delivery-received date. A promised date is used when recorded.

## Requirement and Ordering Rules

For every material in the product's one-level BOM:

| Result | Rule | Interpretation |
|---|---|---|
| Gross need | `production quantity × BOM quantity per product unit` | Material required for the planned production run. |
| Net need | `max(gross need − available inventory, 0)` | Quantity still uncovered by usable on-hand inventory. |
| Recommended order quantity | `net need`, only when `net need > 0` | Exact positive shortage to recommend. When net need is zero, show no purchase recommendation. |
| Latest safe order date | `production date − fixed lead time in calendar days` | The buyer must place the order on or before this date for the material to arrive by the production date under the standard lead-time rule. |

For example, a plan for 1,500 cups needs `1,500 × 0.15 = 225 kg` of blueberry puree. With 180 kg available, net need is `max(225 − 180, 0) = 45 kg`; the recommendation is 45 kg.

## Weekly Planning and Inventory Consumption

For a selected week, the application first groups all production-plan rows by material and sums their gross needs. It then consumes each material's starting available inventory in ascending production-date order. If two rows have the same production date, it uses ascending product name as the stable tie-breaker. A row's covered quantity reduces the inventory available to every later row, so inventory cannot be counted twice.

The weekly recommended quantity is the sum of all positive sequential shortages for that supplier-material combination; it is equivalent to `max(total weekly gross need − starting available inventory, 0)`. If any shortage exists, its latest safe order date is the earliest deadline among the production rows with a positive sequential shortage. This aggregation does not change the per-row arithmetic shown in the worked single-date scenarios.

## Data Validation and Alert Basis

The application must show the inputs and arithmetic behind every alert: product, production date, production quantity, BOM quantity, gross need, available inventory, net need, supplier-material lead time, and latest safe order date when applicable.

A recommendation is blocked, rather than guessed, when an input required for that result is missing or invalid:

| Missing or invalid input | Blocked output | Message to the buyer |
|---|---|---|
| Production date or production quantity is missing, or production quantity is zero or negative | Gross and net need, recommendation, and deadline | `Cannot calculate: production date and a positive production quantity are required.` |
| BOM quantity is zero or negative | Gross and net need; recommendation | `Cannot calculate: BOM quantity must be positive for <material>.` |
| Material and BOM units are incompatible | Gross and net need; recommendation | `Cannot calculate: incompatible units for <material>.` |
| BOM quantity | Gross and net need; recommendation | `Cannot calculate: BOM quantity is missing for <material>.` |
| Available inventory is missing or negative | Net need and recommendation | `Cannot calculate: available inventory must be zero or greater for <material>.` |
| Fixed supplier-material lead time is missing or negative | Latest safe order date only | `Quantity can be calculated, but deadline is unavailable: lead time must be zero or greater for <supplier> and <material>.` |

A zero or negative result after the net-need formula is displayed as net need `0`; it creates no purchase recommendation. A missing or invalid input is not treated as zero. Crucially, when BOM quantity and inventory are complete and valid, a missing or invalid lead time blocks **only** the latest safe order date: the application still shows the calculated net need and a positive quantity recommendation with its named supplier.

For a valid positive recommendation with a valid deadline, compare the deadline with the planning date. A deadline equal to the planning date is displayed as **Order today**. A deadline before the planning date is displayed as **Overdue by _N_ calendar days** and retains the calculated quantity; it does not silently move or remove the recommendation.

## Delivery-History Rule

Delivery history is descriptive only. For a complete delivery record, actual lead time is `delivery-received date − order-sent date` in calendar days. A delivery is late when it is received after its promised date; if no promised date was recorded, it is late when its actual lead time exceeds the fixed supplier-material lead time. The delay is the number of calendar days beyond that promised date or standard lead time. A receipt date before its order-sent date is invalid, is labelled `Invalid delivery date sequence`, and is excluded from every delivery statistic and risk assessment.

Incomplete records are marked `Incomplete delivery record` and excluded from average lead-time and late-delivery statistics. For each supplier-material combination, the descriptive historical summary contains: count of valid complete deliveries, average actual lead time (`sum of actual lead-time days ÷ valid complete delivery count`, displayed to one decimal), count of late deliveries, and greatest recorded delay in calendar days. If there are no valid complete deliveries, display `No valid delivery history` rather than a statistic. These values do not alter a recommendation, quantity, or latest safe order date.

A supplier-delay risk warning appears only when both conditions hold: (1) that supplier-material combination has at least one valid complete late delivery with a positive delay, and (2) the current plan has a positive net need for the same material. The warning states the greatest recorded delay and the risk calculation. When `latest safe order date + greatest recorded delay` is on or after the affected production date, state that a repeat delay could reach or miss the production date; otherwise state that the delay is historical supplier risk. This rule is descriptive only.

## Worked Demo Scenarios

### 1. Production-plan increase

Planning date: 2026-11-03. The Blueberry Yogurt plan increases from 1,000 to **1,500 cups** for production on 2026-11-10. The calculation uses the resulting total plan of 1,500 cups.

| Material | BOM per cup | Gross need calculation | Available inventory | Net need calculation | Fixed lead time | Expected result |
|---|---:|---:|---:|---:|---:|---|
| Blueberry puree | 0.15 kg | `1,500 × 0.15 = 225 kg` | 180 kg | `225 − 180 = 45 kg` | 3 days | Recommend 45 kg; latest safe order date **2026-11-07**. |
| Yogurt cup | 1 cup | `1,500 × 1 = 1,500 cups` | 1,600 cups | `max(1,500 − 1,600, 0) = 0` | 5 days | No purchase recommendation. |
| Yogurt lid | 1 lid | `1,500 × 1 = 1,500 lids` | 1,700 lids | `max(1,500 − 1,700, 0) = 0` | 3 days | No purchase recommendation. |

**Expected warning:** Blueberry puree requires a purchase recommendation because the increased plan needs 225 kg while only 180 kg is available. The buyer sees the 45 kg shortfall and the 2026-11-07 deadline. Buyer options are to order 45 kg by the deadline, change the plan, or investigate usable inventory; the system does not order automatically.

### 2. Packaging shortage

Planning date: 2026-11-04. The plan is **800 cups** for production on 2026-11-12.

| Material | BOM per cup | Gross need calculation | Available inventory | Net need calculation | Fixed lead time | Expected result |
|---|---:|---:|---:|---:|---:|---|
| Blueberry puree | 0.15 kg | `800 × 0.15 = 120 kg` | 140 kg | `max(120 − 140, 0) = 0 kg` | 3 days | No purchase recommendation. |
| Yogurt cup | 1 cup | `800 × 1 = 800 cups` | 650 cups | `800 − 650 = 150 cups` | 4 days | Recommend 150 cups; latest safe order date **2026-11-08**. |
| Yogurt lid | 1 lid | `800 × 1 = 800 lids` | 900 lids | `max(800 − 900, 0) = 0` | 3 days | No purchase recommendation. |

**Expected warning:** Yogurt cups are the affected packaging material: 150 cups are short. The buyer sees the calculation and must decide whether to order 150 cups by 2026-11-08, change the production plan, or verify stock. If the cup BOM, cup inventory, or its supplier lead time is missing, the applicable blocked-calculation message above replaces the unsupported result.

### 3. Delayed supplier delivery

Planning date: 2026-11-03. The plan is **600 cups** for production on 2026-11-10. Yogurt lids are supplied by the fictional supplier DairyPack AS with a fixed lead time of 3 calendar days.

| Material | BOM per cup | Gross need calculation | Available inventory | Net need calculation | Fixed lead time | Expected result |
|---|---:|---:|---:|---:|---:|---|
| Blueberry puree | 0.15 kg | `600 × 0.15 = 90 kg` | 110 kg | `max(90 − 110, 0) = 0 kg` | 3 days | No purchase recommendation. |
| Yogurt cup | 1 cup | `600 × 1 = 600 cups` | 700 cups | `max(600 − 700, 0) = 0` | 4 days | No purchase recommendation. |
| Yogurt lid | 1 lid | `600 × 1 = 600 lids` | 500 lids | `600 − 500 = 100 lids` | 3 days | Recommend 100 lids; latest safe order date **2026-11-07**. |

The recorded DairyPack AS lid delivery was ordered on 2026-10-01, promised for 2026-10-04, and received on 2026-10-07. Its actual lead time is `2026-10-07 − 2026-10-01 = 6 days`; it is **3 days late** against both the promised date and the 3-day standard lead time.

**Expected warning:** The buyer sees a 100-lid recommendation and its unchanged 2026-11-07 deadline, plus a supplier-delay risk: a delivery ordered on that deadline is expected on 2026-11-10 under the fixed lead time, but would arrive on 2026-11-13 if it repeated the recorded three-day delay. Buyer options are to contact DairyPack AS, seek an expedited or alternative supply, change the plan, or accept the risk. The historical delay explains the risk only; it does not change the 100-lid quantity or the fixed-lead-time deadline. A record missing either order-sent or delivery-received date is labelled incomplete and excluded from delivery statistics.

## Hand-Check Checklist

For each scenario, a reviewer can verify that the displayed gross need equals planned cups times BOM quantity, net need is never below zero, a recommendation exists only for positive net need, and the latest safe order date equals production date minus the listed fixed lead time. The warning must name the material, reason, numeric result, and buyer options.
