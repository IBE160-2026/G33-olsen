---
title: 'Product Requirements Document: ProsjektMelk'
status: final
created: 2026-10-18
updated: 2026-11-03
---

# Product Requirements Document: ProsjektMelk

## 0. Document Purpose

This PRD translates the approved product brief and teacher feedback into stable requirements for UX, architecture, epics, stories, implementation, and assessment. It defines product behavior and release boundaries; the exact calculation, validation, and AI contracts remain authoritative in [MRP Calculation Rules and Demo Scenarios](../../briefs/brief-G33-Olsen-2026-09-19/calculation-rules.md) and [AI Explanation and Keyless Fallback Plan](../../briefs/brief-G33-Olsen-2026-09-19/ai-explanation-plan.md).

## 1. Vision

ProsjektMelk is a browser-based decision-support dashboard for a fictional dairy manufacturer. It helps a procurement buyer understand how a changed weekly production plan affects ingredient and packaging needs, identify shortages before production stops, and see a safe next action without manually calculating requirements.

The buyer first checks On-hand Inventory, Allocated Quantity, and Available Quantity, then evaluates Material need and whether the configured Fixed Lead Time allows a new order to arrive before production starts. Delivery feasibility is a planning assessment, not supplier confirmation. If timely supply cannot be confirmed, the buyer coordinates a plan change with production outside the application.

The product is intentionally transparent and narrow. A deterministic calculation engine produces every quantity, date, supplier statistic, and risk state. Supplier history and the optional AI Explanation help the buyer understand those results but never change them. The buyer remains responsible for purchasing decisions, and the application never places an order.

The first version is a graded course project. Its success depends on a stable, testable core, a traceable AI-assisted development process, and the ability for an assessor to run the application locally without signing in or using paid services or private infrastructure.

### 1.1 Release Decision

Delivery has four gates: verified MRP and keyless Standard Explanations; goods receipts, deviations, physical consumption and linked delivery history; descriptive supplier comparison and risk; then optional AI Explanation. The teacher-defined core chain—change Production Plan, calculate Material need, show shortage, and show Latest Safe Order Date—takes precedence over later stock-posting functions, persistence, extra catalog data, visual polish, supplier comparison, and AI work. The increment boundaries appear in §9.

## 2. Target Users and Journeys

### 2.1 Target Users

- **Primary user — procurement buyer:** needs to identify an affected material, verify the shortage quantity and deadline, assess supplier risk, and choose a response.
- **Secondary stakeholder — production planner:** may view the same dashboard without signing in to understand material constraints, but has no separate role, permissions, or version 1 journey. The procurement buyer enters demo plan changes.

### 2.2 Jobs To Be Done

- When the weekly plan changes, identify which ingredients or packaging materials become short.
- Verify how each requirement, recommendation, and deadline was calculated.
- Understand whether recorded supplier performance introduces delivery risk without confusing that history with the fixed planning rule.
- Identify buyer options before the planned production run is affected.
- Rerun the verified demo from a known state during development and assessment.

### 2.3 Key User Journeys

- **UJ-1 — Recalculate after a plan change:** the buyer opens the dashboard without signing in, selects a production week, inspects On-hand Inventory, Allocated Quantity, and Available Quantity, and changes a planned quantity or date within that week. The buyer sees recalculated material rows, shortages, deadlines, calculation bases, Standard Explanations, and buyer options. When timely supply cannot be confirmed, the buyer uses the affected Material, shortage quantity, and production date to discuss a plan change with production outside the application, then can manually edit the plan and recalculate.
- **UJ-2 — Investigate supplier risk:** the buyer opens a shortage, reviews valid descriptive delivery history, and sees why a late delivery creates risk. Descriptive history never changes MRP formulas or Fixed Lead Time. Recording an actual receipt through UJ-5 changes physical stock and therefore recalculates the current need; history must not be confused with that stock posting.
- **UJ-3 — Request an optional AI explanation:** when server configuration is valid, the buyer explicitly requests an AI explanation and receives only a validated presentation of the current calculated result. Any failure leaves the standard explanation visible.
- **UJ-4 — Restore the assessment baseline:** the buyer selects **Reset demo data**, confirms the destructive reset, and receives the original fictional plans, inventory, and delivery records for the three verified scenarios. Canceling confirmation preserves the current state.
- **UJ-5 — Receive materials and complete production:** the buyer enters actual receipt, total and rejected quantities, and available order-confirmation dates in one form. The application validates and posts accepted stock plus linked history once. The buyer reviews recalculated stock and shortages, then confirms the displayed BOM consumption for a production run, acknowledging any allocation conflict. The completed row remains visible and read-only; remaining rows are recalculated against current stock.

## 3. Glossary

- **Production Plan** — Planned finished-product quantity and production date within a selected week.
- **Planning Date** — Seeded date-only evaluation date for one named Demo Scenario; it is never taken from the computer clock during scenario acceptance.
- **Demo Scenario** — Isolated named planning snapshot containing its own Planning Date, Production Plan, inventory, and configured supplier relationships; the fictional supplier-delivery history is shared globally across snapshots.
- **BOM Line** — One direct relationship from a finished product to an ingredient or packaging material and its quantity per product unit.
- **Material** — An ingredient or packaging item consumed by a Production Plan.
- **Usable Inventory** — Non-negative on-hand Material quantity available to cover demand.
- **Opening Balance** — Historical baseline stock for a Material in a scenario, retained separately from subsequent posted movements.
- **On-hand Inventory** — Current usable physical stock in the active scenario: Opening Balance plus posted accepted receipts minus posted physical consumption. Rejected quantities and planned allocations do not change this balance. This is fictional demo stock, not a live warehouse count.
- **Allocated Quantity** — The cumulative portion of current On-hand Inventory assigned to earlier outstanding rows in the FR-6 calculation order. Only covered demand is allocated; unmet demand never allocates nonexistent stock.
- **Available Quantity** — On-hand Inventory minus Allocated Quantity before the current production row. It is never negative and is the Usable Inventory input to that row's Net Need calculation.
- **Gross Need** — Production quantity multiplied by the applicable BOM Line quantity.
- **Net Need** — The positive remainder after subtracting Usable Inventory from Gross Need; never below zero.
- **Recommended Order Quantity** — A positive Net Need. Zero Net Need produces no purchase recommendation.
- **Fixed Lead Time** — Non-negative whole calendar days configured for one supplier–Material combination.
- **Latest Safe Order Date** — Production date minus Fixed Lead Time.
- **Standard Explanation** — Immediate deterministic English text rendered from application-calculated facts.
- **AI Explanation** — Optional English text rendered by the server from a validated, provider-selected plan over the same facts.
- **Order Sent Date** — Date the buyer sent the purchase order; it may be explicitly marked unknown for a new receipt.
- **Confirmed Delivery Date** — Optional date the supplier promises arrival at the dairy, for example in an order-confirmation email. It is the existing contract's `promised date`, not the date the email arrived or the supplier shipped the goods.
- **Actual Receipt Date** — Date goods physically arrived at the dairy; it is the contract's `delivery-received date` and the receipt posting's date. It may differ from Confirmed Delivery Date.
- **Delivery Record** — A supplier–Material history entry using Order Sent Date, optional Confirmed Delivery Date, and Actual Receipt Date. Missing order date makes it incomplete, not evidence of a guessed lead time.
- **Supplier Risk** — Descriptive warning derived from valid Delivery Records and a current positive Net Need; it never changes a quantity or deadline.
- **Demo Data** — Preloaded fictional products, Materials, suppliers, inventory, Production Plans, and Delivery Records.

## 4. Features and Functional Requirements

### 4.1 Demo Data, Planning, and Persistence

**Description:** The dashboard starts from coherent Demo Data, supports the required production-plan interaction, and remains repeatable for assessment. Real company and personal data are forbidden.

#### FR-1: Load the fictional baseline

The system must provide Demo Data sufficient to run the three Blueberry Yogurt acceptance scenarios without manual setup.

**Consequences:**
- The required product, Blueberry puree raw material, Yogurt cup, Yogurt foil seal (oblat), Yogurt lid, inventory, supplier, lead-time, plan, and delivery values are present.
- The dashboard provides a named Demo Scenario selector for `scenario-plan-increase`, `scenario-packaging-shortage`, and `scenario-delayed-delivery`; each loads an isolated planning snapshot and seeded Planning Date, while supplier-delivery history is global and reset activates `scenario-plan-increase`.
- Scenario selection loads an exact initial plan and the buyer performs one acceptance edit: 1,000→1,500 cups for `scenario-plan-increase`, 600→800 for `scenario-packaging-shortage`, and 500→600 for `scenario-delayed-delivery`.
- Demo Scenario acceptance uses the seeded Planning Date rather than the computer clock.
- Version 1 is a current-state demo. Selecting a week filters outstanding production for the FR-6 planning calculation; it does not rewind physical stock, repost historical movements, or promise a date-effective allocation across several weeks. A week view's quantity assessment is not a guarantee against demand outside that selected week. Historical as-of balances and cross-week time-phased supply planning are deferred.
- New goods receipts, deviations, physical stock balances, completed-production state, and consumption history belong only to the active Demo Scenario. Switching scenarios must not copy or replay stock movements into another scenario; returning restores that scenario's own saved state. The exact initial plan applies to a fresh or reset baseline, not every return to a scenario.
- Additional products may be included only after the required scenarios are stable.

#### FR-2: Change a weekly Production Plan

The buyer must be able to select a week and change a planned finished-product quantity and date. Realizes UJ-1.

Quantity and date edits apply only to outstanding production. A **Completed** row remains visible but its production quantity and date are read-only; attempts to edit it must be rejected without changing the plan, posted consumption, or inventory.

**Consequences:**
- A valid change triggers deterministic recalculation.
- Invalid input displays all applicable validation messages and never becomes an implicit zero.
- A valid production date may change only within the selected ISO week. An out-of-week edit is rejected with `Cannot update production date: date must remain within selected week <week>.`, the previous valid date remains stored and visible, and weekly totals do not change.

#### FR-3: Persist local demo changes

Valid Production Plan changes and Delivery Records must persist locally across application restarts.

Scenario-owned receipts, deviations, consumption history, physical balances, and production-completion state must persist under the same conditions. Reloading or switching scenarios must never repost a receipt or deduct consumption again. Supplier-delivery history remains shared descriptive evidence and cannot move inventory between scenarios.

**Consequences:**
- Persistence requires no sign-in, cloud service, or external infrastructure.
- Persisted changes survive page reload, browser close/reopen, and local application-server restart on the same local application instance.
- Persisted state has a version. Missing state loads the baseline; corrupt or incompatible state blocks calculations, explains the problem, and offers **Reset demo data** as a recovery path.
- Version 1 does not synchronize state across machines or users.

#### FR-4: Reset Demo Data

The buyer must be able to restore the complete fictional baseline for repeatable testing and assessment. Realizes UJ-4.

**Consequences:**
- Reset replaces locally changed plans and delivery records only after confirmation.
- Canceling confirmation leaves the active scenario, locally changed plans, inventory, and delivery records unchanged.
- Confirmed reset also clears newly posted scenario receipts, deviations, and consumption history, restores baseline physical inventory and uncompleted production in every scenario, and recomputes baseline allocation and availability. Cancel preserves these records and states as well.
- An in-flight receipt or completion from before confirmed reset cannot post into the restored baseline; scenario switching must not change an action's owning scenario. MRP-R10 governs this context boundary.
- After reset, all three named planning snapshots return to their documented `initialPlan` values and the global supplier history contains all three baseline Yogurt-lid Delivery Records—one DairyPack AS record and two PackForm AS records; `scenario-plan-increase` becomes active.

### 4.2 Deterministic Material Requirements Planning

**Description:** The calculation engine is the sole source of numerical and operational truth. It uses a one-level BOM and fixed supplier–Material lead times.

#### FR-5: Calculate per-plan material demand

For every BOM Line, the system must calculate `Gross Need = production quantity × BOM quantity`, `Net Need = max(Gross Need − Usable Inventory, 0)`, and Recommended Order Quantity equal to positive Net Need.

**Consequences:**
- Production quantity must be a whole number from 1 through 1,000,000 finished cups. A BOM quantity must be greater than zero and no more than 1,000,000.000. Inventory must range from zero through 1,000,000,000,000.000. Every derived material quantity must be no more than 1,000,000,000,000.000. BOM, inventory, need, and recommendation values must be finite base-10 decimals with at most three fractional digits.
- Arithmetic is exact in base 10, never rounds intermediate values, and displays final quantities with up to three fractional digits and no trailing zeros.
- Fractional BOM boundary tests must prove that rounding cannot create or remove a shortage.
- Inputs at and just beyond every product-level maximum must have automated boundary tests.

#### FR-6: Calculate weekly material demand

For a selected ISO-8601 Monday–Sunday week, the system must group outstanding demand by Material and allocate current On-hand Inventory sequentially by production date, product name, and a stable row identifier. Completed rows remain visible but contribute no outstanding demand. This planning allocation does not post physical consumption.

**Consequences:**
- Current physical stock is not counted twice. Opening Balance is historical, not an additional source of supply.
- Before each outstanding row, the displayed Allocated Quantity is the sum of inventory-covered quantities for earlier outstanding rows in the documented order; Available Quantity equals current On-hand Inventory minus that allocation. Allocation never exceeds current stock, and an earlier shortage never becomes an allocation of nonexistent stock. Recalculation derives these values anew; there is no manual reservation workflow.
- The weekly recommendation equals the sum of positive sequential shortages and `max(total outstanding Gross Need − current On-hand Inventory, 0)`.
- Acceptance example: Opening Balance is 1,000 cups, a receipt accepts 480 cups, and completed production consumes 500. Current On-hand Inventory is 980 cups. An outstanding run requiring 1,200 cups has a 220-cup shortage; neither the opening stock nor completed demand is counted again. Without posted movements or Completed rows, existing baseline scenario answers remain unchanged.
- Each recommendation and deadline attaches only to the Demo Scenario's configured supplier–Material relationship; alternative comparison suppliers do not enter MRP arithmetic.
- The hand-calculated multi-row fixture in MRP-R2 must verify sequential inventory consumption, stable same-date tie ordering, non-duplication of inventory, aggregate equality, and earliest-deadline selection.

#### FR-7: Calculate the Latest Safe Order Date

For a positive Net Need with valid supplier data, the system must calculate Latest Safe Order Date as production date minus Fixed Lead Time in calendar days.

**Consequences:**
- A Latest Safe Order Date that equals the Planning Date displays **Order today**.
- A Latest Safe Order Date earlier than the Planning Date displays **Overdue by N calendar days** without changing the Recommended Order Quantity.
- Positive-shortage results with a valid deadline and Planning Date must state that timing is based on Fixed Lead Time and does not confirm supplier availability or delivery. A deadline on or after the Planning Date means a new order can arrive by the production date under that rule; an overdue deadline means normal lead time no longer covers the planned start. The application must not claim that actual delivery is guaranteed or impossible.
- The weekly deadline is the earliest deadline only when every row with a positive sequential shortage has valid configured supplier and lead-time data. If any such row lacks valid deadline input, quantities remain visible but the aggregate deadline is `deadline-unavailable`; the system must not select a deadline from the remaining rows.
- Planning, production, order, delivery, and deadline values are plain calendar dates calculated without local timestamps or daylight-saving conversion.
- Every entered date must be a real Gregorian date in exact `YYYY-MM-DD` form.

#### FR-8: Display the calculation basis

Every result must show product, production date and quantity, BOM quantity, Gross Need, Usable Inventory, Net Need, Material unit, and—when applicable—supplier, Fixed Lead Time, Recommended Order Quantity, and Latest Safe Order Date.

For each valid inventory row, show **On-hand Inventory**, **Allocated Quantity**, and **Available Quantity** with the Material unit and current production context. The Usable Inventory used for row-level Net Need is that row's Available Quantity, not the starting stock again. The MRP-R2 fixture must display allocation/availability pairs of 0/75, 60/15, and 75/0 kg in its documented row order; after a plan edit, labels and values must update with the calculation.

#### FR-9: Handle incomplete calculation input

The system must apply **MRP-R3 — Calculation validation and partial-result rule** from the calculation contract.

**Consequences:**
- Missing, fractional, out-of-range, or otherwise invalid production input blocks affected downstream calculations with the exact MRP-R3 message; production quantities are never rounded to whole cups.
- Material and BOM units are compatible only by exact canonical unit-code equality. Version 1 codes are `cup`, `kg`, `seal`, and `lid`, and version 1 performs no conversions.
- Missing, negative, or fractional Fixed Lead Time blocks only the deadline when quantity and supplier remain valid.
- A missing or invalid Planning Date preserves valid quantities and Latest Safe Order Dates, blocks only deadline urgency, shows `Deadline urgency is unavailable: a valid Planning Date is required.`, and makes the affected result AI-ineligible.
- A positive shortage without a supplier retains its quantity but produces no purchase option.
- All applicable validation messages appear in deterministic order.

### 4.3 Alerts, Standard Explanations, and Buyer Options

**Description:** Results turn into readable decision support without hiding their basis or taking action for the buyer.

#### FR-10: Show shortage and no-shortage states

The dashboard must distinguish positive shortage, no shortage, blocked calculation, order-today, and overdue states for each affected Material.

#### FR-11: Show a Standard Explanation

Every valid or blocked result must immediately display a Standard Explanation using the exact validated facts and deterministic English rendering rules. Realizes UJ-1.

#### FR-12: Present buyer-controlled options

For a positive shortage, the system must display the complete applicable option set defined by the AI contract's **Immutable source data and request construction** and **Closed model response** sections.

**Consequences:**
- Options are choices, not recommendations executed by the application.
- The application never claims that an order, approval, contact, expedition, or plan change has occurred.
- Every positive-shortage detail identifies the affected Material, shortage quantity, and production date so the buyer can communicate the constraint outside the app. The buyer can manually change quantity or date within the selected week under FR-2 and recalculate; the system sends no message and performs no automatic rescheduling.

### 4.4 Supplier Follow-up

**Description:** Supplier information is descriptive evidence. It helps explain risk but never feeds back into Fixed Lead Time or MRP arithmetic. Realizes UJ-2.

#### FR-13: Record supplier deliveries

The buyer must be able to record Order Sent Date (or explicitly unknown), optional Confirmed Delivery Date, and Actual Receipt Date for a fictional supplier–Material combination. These labels map respectively to the existing contract's order-sent, promised, and delivery-received fields; no extra duplicate date fields are required.

For new physical receipts, these delivery fields belong to the single goods-receipt workflow in FR-25. Posting must create one linked Delivery Record automatically; the buyer must not enter that arrival again in a separate delivery-history form. Existing seeded historical Delivery Records remain descriptive evidence already accounted for in baseline stock.

**Consequences:**
- Every Delivery Record has a stable local ID. If a repeated save uses the same ID and a byte-equivalent normalized payload, the system returns the existing record without changing it or duplicating statistics. If an existing ID is reused with a different supplier, Material, or dates, the system rejects the save without changing data and displays `Cannot save delivery: record ID already exists with different content.` Version 1 doesn't provide record editing.
- A successful save is atomic and immediately recomputes and displays the affected supplier–Material summary and Supplier Risk.

#### FR-14: Validate Delivery Records

The system must apply **MRP-R5 — Delivery validation and statistics rule**. A Delivery Record is incomplete if its order-sent or delivery-received date is missing. It is invalid if its promised or delivery-received date is before its order-sent date. The system must validate presence before date sequence and exclude incomplete and invalid records from all statistics and risk calculations.

For a new physical receipt, apply MRP-R8 posting validation before saving either stock or the linked history: Actual Receipt Date is mandatory, and every supplied date must be valid. An explicitly unknown Order Sent Date permits an incomplete linked record; it must not conceal another supplied invalid date. Confirmed Delivery Date may be before or after Actual Receipt Date because deliveries may be early or late; it may be in the future relative to Planning Date and never increases stock by itself.

#### FR-15: Summarize valid delivery history

For each supplier–Material combination, the system must show the valid-delivery count, average actual lead time to one decimal place, late-delivery count, and greatest recorded delay. If there are no valid records, the system must display **No valid delivery history**.

**Consequences:**
- Average actual lead time uses decimal round-half-up to one decimal before display or explanation rendering.
- If a valid record has no promised date and its Fixed Lead Time is unavailable, valid count and average actual lead time remain available, but late count, greatest delay, and Supplier Risk are unavailable with the exact MRP-R5 message.

#### FR-16: Explain lateness consistently

A complete delivery is late if it is received after its promised date. If no promised date exists, it is late when actual lead time exceeds Fixed Lead Time.

#### FR-17: Show Supplier Risk only when relevant

Supplier Risk must appear only when the same supplier–Material combination has a positive current Net Need, at least one valid late Delivery Record with positive delay, and a valid current Latest Safe Order Date. When history and need qualify but the current deadline is unavailable, the delivery summary remains visible, Supplier Risk is suppressed, and the Standard Explanation appends `Supplier Risk is unavailable: a valid current deadline is required to compare historical delay.` The result is AI-ineligible.

#### FR-18: Keep supplier history descriptive

Delivery history, Supplier Risk, and supplier comparison must never modify Fixed Lead Time, Recommended Order Quantity, or Latest Safe Order Date.

#### FR-19: Compare available suppliers descriptively

The buyer must be able to compare DairyPack AS and PackForm AS side by side for Yogurt lid using only their validated descriptive history.

**Consequences:**
- DairyPack AS remains the configured scenario supplier with its 3-day Fixed Lead Time and recorded 3-day delay.
- PackForm AS is an alternative lid supplier with a 4-day Fixed Lead Time, one on-time four-day delivery, and one five-day delivery received one day late.
- The view shows no score, ranking, automatic selection, derived recommendation, or change to the configured supplier or calculation.

### 4.5 Optional AI Explanation

**Description:** The provider path demonstrates tightly constrained AI presentation. It is never required for calculation, explanation availability, or assessment. Realizes UJ-3.

#### FR-20: Expose generation only when eligible

The system must show **Generate AI explanation** only when all of the following are present: complete and valid server configuration, the required signed anonymous demo session, the required AI-contract endpoint controls, and an eligible current result. Generation must start only after the buyer selects the control.

#### FR-21: Restrict the model to existing facts

The model must receive only immutable, version-bound fictional facts and permitted buyer options already produced by the application. It must never calculate, correct, rank, extrapolate, mutate data, or take operational action.

#### FR-22: Validate before rendering

The server must accept only `contractVersion: 1` and the closed structured explanation plan defined by the AI contract's **Closed model response** and **Validation, version binding, and rendering** sections. It must verify the evidence bindings and current result version before rendering the plan through server-owned English templates.

**Consequences:**
- Provider-supplied prose never reaches the interface.
- A stale, incomplete, unsupported, or invalid plan is discarded rather than repaired in the browser.

#### FR-23: Preserve the keyless fallback

Missing configuration, unavailable project-account access, timeout, network or provider failure, invalid output, rate limiting, or cancellation must leave the alert and Standard Explanation fully usable.

#### FR-24: Bound provider use

Provider calls must use the approved `gpt-5.6-sol` model with separate `high` reasoning effort and every mandatory control in the AI contract's **Endpoint cost and concurrency controls** section.

### 4.6 Goods Receipt and Physical Consumption

Realizes UJ-5. Receipts and confirmed production extend the planning model with explicit physical stock movements. Planned allocation is not physical consumption. Exact posting rules and the end-to-end stock fixture are defined in MRP-R8–MRP-R10 of the calculation contract.

#### FR-25: Record goods receipt and deviations

The buyer must be able to record a Material, supplier, Actual Receipt Date, total received quantity, and rejected/deviation quantity, plus Order Sent Date or explicit unknown and optional Confirmed Delivery Date. Accepted quantity equals total received quantity minus deviation quantity; only accepted quantity increases On-hand Inventory. Recalculate allocation, availability, shortages, and recommendations after posting. The date information is entered manually; email retrieval or parsing is outside version 1.

**Consequences:**
- A receipt of 500 cups with 20 rejected increases stock by 480 cups; history preserves all three quantities.
- Quantities must be valid in the Material's canonical unit, total receipt must be positive, and deviation must be between zero and total received. Invalid input cannot change stock.
- Apply MRP-R8's exact decimal, unit, identity, and atomic-posting rules. Total, rejected, accepted, and resulting stock quantities must be finite base-10 values with at most three fractional digits and no more than 1,000,000,000,000.000. Reject unsupported precision or overflow without rounding or changing either stock or history. Two genuinely distinct receipt identities can hold identical quantities; a replay of one identity cannot post twice.
- A receipt may be posted only with a valid receipt date on or before the active scenario's valid seeded Planning Date. Reject a later date with **Cannot post receipt: receipt date must not be after the scenario Planning Date.** Reject a missing or invalid receipt or Planning Date with **Cannot post receipt: valid receipt and Planning Dates are required.** Rejection changes neither stock nor linked delivery history. Do not substitute the computer clock or invent a date.
- Acceptance must allow receipt on the Planning Date and an earlier valid date, reject the next calendar day, and reject invalid/missing dates without changing stock or history. Expected future deliveries never increase current stock. Date-effective future supply, historical balance reconstruction, and delivery-by-production-date calculations are deferred; receipt posting is a current-balance event, not a retroactive replay of history.
- The same receipt can affect stock only once. Duplicate actions must not duplicate stock or supplier statistics.
- One registration must post both the accepted stock increase in the active scenario and exactly one linked global Delivery Record. Collect the existing FR-13 delivery fields in this same workflow; the receipt date is the delivery-received date. Reuse Material and supplier rather than asking the buyer to enter them twice. Supplier statistics include the linked record only under the existing FR-14 validation rules; missing or invalid dates must never be invented to make a record eligible.
- An unknown order-sent date may be left empty and must not block an otherwise valid physical receipt. Post accepted stock and retain the linked Delivery Record as **Incomplete delivery record** with the missing date visible; exclude that record from all supplier statistics and Supplier Risk under FR-14/MRP-R5. Do not infer order-sent date from receipt date or Fixed Lead Time. This exception does not waive the required valid receipt date or Planning Date.
- The buyer must explicitly select the unknown state when Order Sent Date is unavailable. A supplied malformed date, Order Sent Date after Actual Receipt Date, or Confirmed Delivery Date before a known Order Sent Date blocks the entire posting with the applicable MRP-R8 validation. Do not automatically discard or mark an invalid supplied date unknown. The buyer can correct it or explicitly mark only the order date unknown before retrying; cancel or failure changes nothing.
- Acceptance must post 500 cups with 20 rejected and no order-sent date: stock increases by 480 once, the incomplete linked record remains inspectable, and valid-delivery count, average lead time, late count, and delay statistics remain unchanged. Existing valid history continues to contribute normally; incomplete evidence alone must not create Supplier Risk.
- The receipt and linked Delivery Record must retain a stable one-to-one reference. Posting both effects is atomic; a failed save changes neither stock nor history. Retrying the same registration returns the same posted result without adding stock or another Delivery Record. Conflicting content under an existing receipt identity is rejected without modifying either effect.
- The linked history is shared descriptive evidence and can update supplier summaries or risk in other scenarios; only the owning scenario receives stock. Acceptance must verify both effects, cancel/failure atomicity, replay protection, and unchanged physical stock in other scenarios.
- Existing historical deliveries are evidence, not new receipts. Their quantities must never be added again to the starting stock.
- A posted receipt changes only its owning active scenario's stock. Acceptance must post a 500-cup receipt with 20 rejected in one scenario, verify its 480-cup increase, switch to another scenario and verify unchanged stock there, then return and verify the original receipt and balance without a second posting.

#### FR-26: Confirm production and record physical consumption

Before posting, **Complete production** must show the consumption calculated from the production quantity and BOM and require buyer confirmation. In this version, confirmed physical consumption equals that displayed calculation; recording production waste or actual-versus-planned consumption differences is deferred.

Apply MRP-R9 for validation and atomic posting. Confirmation must use the current outstanding production quantity, BOM, and physical balances; if those facts change after the preview, refresh the preview and require confirmation again rather than posting stale amounts. No completion occurs automatically merely because a production date passes.

**Consequences:**
- Confirmation deducts the displayed Material quantities and records Material, quantity, and the production reference in persistent consumption history. Canceling changes nothing.
- Completion must occur atomically for the whole production: insufficient physical On-hand Inventory for any required Material blocks completion, identifies the shortage, and leaves all stock and production state unchanged. This physical-stock validation is separate from a conflict with planned allocation.
- When physical stock covers the requested consumption but part of that stock is allocated to other production, show a warning before confirmation identifying the affected Material, conflicting allocated quantity, affected production, and resulting shortages. An allocation conflict must not disable completion: the buyer can cancel or explicitly confirm proceeding despite the conflict. After confirmation, deduct physical consumption once, remove the completed row from outstanding demand, and recompute remaining allocations and shortages using the existing deterministic ordering. Do not silently change other production quantities or dates.
- Acceptance example: physical stock is 1,000 cups, earlier outstanding production needs 800, and the buyer completes another run needing 500. Warn that completion leaves only 500 for the earlier run and creates a 300-cup shortage. Cancel preserves all state; confirmation leaves 500 physical cups, records 500 consumed once, and shows the remaining run's 300-cup shortage. Repeated confirmation must not deduct again.
- A repeated completion must not deduct stock again. Completed production no longer contributes future demand or planned allocation; recalculate the remaining plan against the updated physical balance without double deduction.
- Completed production remains visible in the selected week's plan with the English status **Completed** and its recorded consumption accessible. It is excluded from remaining Gross Need, Net Need, allocation, recommendations, and deadlines; historical consumption must not be presented as a current shortage. Refreshing, switching scenarios, or selecting the week again must not deduct it again. The plan must distinguish completed rows from outstanding rows rather than silently removing them.
- Version 1 provides no undo, reopening, or direct edit of posted production consumption. The confirmed completion flow assumes the buyer follows the production team's checking routines; this is a scope assumption, not a guarantee that human error is impossible. Reversing an individual posting may be considered later and must preserve an audit trail. FR-4's confirmed full-demo reset remains available for restoring fictional baseline data; it is not a reversal of one posting.
- After completion, the production row's quantity and date are read-only under FR-2. Preserve the quantity, date, and Material consumption recorded at posting; later edits to other outstanding rows must not rewrite that history. Acceptance must attempt quantity and date edits on a Completed row, verify rejection and unchanged stock/history, and verify that outstanding rows remain editable.
- Historical receipts, deviations, and consumption remain inspectable after materials have been used. History itself is not available stock; old receipts and consumed goods must never be counted again merely because their records exist.
- Batch tracking and links between material batches and production batches are outside the current required scope and may be considered after the assignment's required flows are stable.

## 5. Cross-Cutting Non-Functional Requirements

- **NFR-1 — Correctness:** Every calculation is deterministic, reproducible, independent of AI, and covered by automated tests against the hand-calculated scenarios.
- **NFR-2 — Ability to run locally:** In a clean, supported environment, the assessor can install, start, reset, and test the application by following the README commands without signing in or using an API key, paid account, or private infrastructure.
- **NFR-3 — Resilience:** Invalid input, repeated actions, absent provider configuration, and provider failures do not crash or block the deterministic workflow.
- **NFR-4 — Security and privacy:** Only fictional data is used. Secrets remain server-side, ignored by Git, absent from browser bundles and logs, and represented only by empty/example values in tracked configuration.
- **NFR-5 — Accessibility, language, and clarity:** The complete version 1 interface uses English and targets WCAG 2.2 AA for the core journeys. All controls have visible semantic labels. The complete core flow works by keyboard in a logical order with visible focus. Status is not communicated by color alone, and dynamic validation and status changes are announced. Normal-text contrast is at least 4.5:1; large-text and UI-component contrast is at least 3:1.
- **NFR-6 — Responsive consistency:** The core dashboard, alert detail, supplier comparison, and reset confirmation must work at 1280 × 720 and 390 × 844 CSS pixels and at 200% browser zoom. Content must remain visible, controls must not overlap, and the primary flow must not require horizontal scrolling.
- **NFR-7 — Maintainability:** Calculation, local persistence, presentation, and provider integration have separate dependency boundaries. UI and provider modules cannot own or override calculation rules. All linting and formatting checks documented in the README must pass for the tracked source files.
- **NFR-8 — Testability:** Central formulas, weekly sequencing, receipt/deviation validation, linked receipt/history posting, production consumption, confirmation/replay behavior, persistence/reset, delivery statistics, Standard Explanations, and provider fallbacks can be tested independently and through the core user flow.

## 6. Delivery and Assessment Requirements

- **DAR-1 — Traceability convention:** Every epic and story must reference its source FR, NFR, or DAR; implementation location; and verifying test or manual check. Record requirements, implementation stories, commits, prompts and session evidence, review findings, corrected AI output, test results, documented corrections, and approved deviations in the repository during the work.
- **DAR-2 — Test isolation:** Automated tests must use deterministic fakes or stubs for the provider path and must make no live or paid OpenAI calls. The documented test strategy must assign central formulas, weekly ordering, validations, linked receipts, physical consumption, persistence/reset, supplier statistics, and provider fallbacks to appropriate unit, integration, or end-to-end coverage. Exercise receipt, completion, scenario-switch, shared-history, and reset events during an active or cached explanation and verify that stale content cannot replace the current result.
- **DAR-3 — UX and architecture evidence:** Before their corresponding implementation, the project must contain a lightweight UX artifact for the priority dashboard and alert-detail view and an architecture artifact describing module boundaries, data flow, persistence ownership, provider isolation, configuration, and dependency direction.
- **DAR-4 — README acceptance:** The README must identify the product, intended user, and supported runtime versions. It must provide exact, copyable commands to install, start, test, lint, and format the application. It must also explain optional environment configuration, keyless behavior, Demo Data, reset, the core-scenario walkthrough, and the project structure, and it must link directly to planning and process artifacts and scenario and seed locations under `.docs/`.
- **DAR-5 — Clean-start rehearsal:** Before delivery, a clean checkout must be installed, started, reset, exercised through the core flow, and tested using only the README instructions. Results and any corrections must be recorded.
- **DAR-6 — Repository hygiene:** `.gitignore` and the final tracked-file audit must exclude secrets, real `.env` files, dependencies, builds, caches, local state/databases, editor files, runtime/debug logs, backups, temporary drafts, and unrelated binaries. Version-controlled project session records under `.logs/<user>/` and their `issues.md` files are required process evidence, not runtime logs. Record each substantive session's task, plan, result, and unresolved project items; mark resolved items without deleting their history.

## 7. Constraints and Guardrails

- The application is decision support, not an autonomous purchasing or production system.
- The calculation and AI contracts linked in §0 govern exact formulas, messages, ordering, schemas, and edge cases.
- The core flow uses no network dependency. Optional provider access is server-only and never silently substitutes another model or reasoning setting.
- A signed anonymous demo session must protect optional provider calls, but version 1 has no sign-in, account management, or user roles.
- Local persistence and Demo Data reset are single-instance demo capabilities, not a multi-user data platform.
- Additional products, visual polish, or optional AI work must not delay stable increments 1 and 2.

## 8. Non-Goals

- Safety stock, minimum order quantities, order multiples, manual or external reservations, open-order supply, or multi-level BOM explosion. Automatic within-week allocation under FR-6 remains in scope.
- Automatic purchase placement, supplier contact, approval, expedition, or production-plan changes.
- Login, account management, role-based access, real company data, or enterprise-grade security.
- Descriptive trend dashboards for production volume, inventory movement, or supplier performance.
- Real ERP or supplier integrations, advanced demand forecasting, full production scheduling, or capacity optimization.
- AI-generated calculations, deadlines, supplier statistics, priorities, options, or display-ready prose.
- Email integration, batch tracing, individual stock-posting reversal, actual-versus-planned production-consumption deviations, and date-effective future-supply or historical-balance reconstruction.

## 9. Release Scope and Order

### Increment 1 — Minimum Complete Submission

- Three isolated named Demo Scenarios with seeded Planning Dates.
- Weekly Production Plan change.
- One-level BOM calculations, weekly sequential inventory consumption, shortages, deadlines, validations, and visible calculation basis.
- Immediate Standard Explanations, buyer options, and complete keyless operation: when provider configuration is absent, the application makes no provider call and hides the generation control without reducing the core result.
- Correct MRP quantities and deadlines for all three acceptance scenarios.

Within Increment 1, verify inventory and allocation before building dependent shortage presentation, then verify quantities and deadlines against the hand-calculated fixtures. Supplier and optional AI work must build on the passing core checks rather than proceed on unverified calculation results.

The mandatory core gate is the tested chain **select Demo Scenario → perform its documented initialPlan→expectedPlan edit → calculate Material need → show shortage → show Latest Safe Order Date and Standard Explanation** for all three arithmetic fixtures. After that gate passes, local persistence and reset complete Increment 1. Increment 2 starts only after both the core gate and persistence/reset tests are stable.

### Increment 2 — Goods Receipt and Physical Consumption

- One goods-receipt registration posts accepted quantity, deviations, and one linked Delivery Record under FR-13/FR-14/FR-25.
- Confirmed production completion posts physical consumption once, preserves read-only Completed history, and recalculates only outstanding demand under FR-26.
- Scenario-owned movements, current balances, and completion states persist and reset under FR-1/FR-3/FR-4. Batch tracing and individual reversal remain deferred.

The gate must verify the FR-25/FR-26 worked examples, missing order-sent date, receipt-date boundaries, quantity/deviation validation, atomic posting, replay/conflict handling, insufficient physical stock, allocation-warning cancel/proceed, Completed-row exclusion and edit rejection, scenario isolation, persistence, and confirmed reset. All Increment 1 baseline fixtures must still pass after reset. Supplier statistics used internally by linked-history validation can be tested here; comparison and risk presentation belong to Increment 3.

### Increment 3 — Supplier Follow-up and Complete Provider-Independent Version 1

- Descriptive supplier history summaries and side-by-side comparison.
- Delayed-delivery Supplier Risk for scenario 3 without changing its reset-baseline 100-lid recommendation or 2026-11-07 deadline.
- Verify FR-15–FR-19 supplier statistics, incomplete-history exclusion, and risk guards, with stock movements continuing to obey Increment 2 rules.

### Increment 4 — Optional Provider Enhancement

- User-triggered AI Explanation with the closed response contract.
- Configured-provider validation, version binding, cost controls, and failure fallback while preserving the already-tested Increment 1 keyless behavior.

Work starts on a later increment only after the previous increment and its tests are stable. Trend dashboards and sign-in remain outside version 1 regardless of available time.

## 10. Success Metrics

### Primary

- **SM-1 — Scenario correctness:** automated tests reproduce 100% of the documented materials, quantities, dates, no-shortage states, and delayed-delivery facts for the three scenarios. Validates FR-5–FR-18.
- **SM-2 — Buyer comprehension:** in a documented usability check, the tester identifies On-hand Inventory, Allocated Quantity, Available Quantity, the affected Material and production date, shortage reason and quantity, deadline state, and one applicable next option without manual calculation. The tester must distinguish a Fixed Lead Time assessment from supplier-confirmed delivery. The intended advantage over spreadsheets is a usability hypothesis, not an established speed improvement; record observed difficulties without claiming a measured comparative benefit unless a comparison is actually performed. Validates FR-7, FR-8, FR-10–FR-12.
- **SM-3 — Clean-start operation:** a clean-machine rehearsal completes installation, startup, Demo Data reset, core-flow execution, and automated tests using only the README instructions. Validates FR-1–FR-4 and NFR-2.

### Secondary

- **SM-4 — Keyless continuity:** every calculation, alert, buyer option, and Standard Explanation remains available with no OpenAI key or provider access. Validates FR-11, FR-20–FR-24.
- **SM-5 — Invalid-input containment:** automated tests confirm every documented invalid or incomplete input blocks only the affected outputs and never invents zero values, recommendations, dates, or statistics. Validates FR-9 and FR-14.
- **SM-6 — Supplier-comparison fixture:** automated tests must verify both Yogurt-lid supplier summaries. DairyPack AS must show 1 valid delivery, a 6.0-day average actual lead time, 1 late delivery, and a greatest delay of 3 days. PackForm AS must show 2 valid deliveries, a 4.5-day average actual lead time, 1 late delivery, and a greatest delay of 1 day. DairyPack risk must remain descriptive and must not change the 100-lid recommendation or 2026-11-07 deadline. Demo Data reset must restore all three Delivery Records. Validates FR-4, FR-15, FR-17, and FR-19.
- **SM-7 — Weekly sequencing fixture:** automated tests reproduce the MRP-R2 multi-row fixture: 135 kg total Gross Need, 75 kg starting inventory, allocation/availability pairs of 0/75, 60/15, and 75/0 kg, row shortages of 0, 15, and 45 kg in deterministic order, a 60 kg weekly recommendation, and 2026-11-07 earliest deadline; invalidating one shortage row's lead time preserves 60 kg and makes the aggregate deadline unavailable. Validates FR-6–FR-9.
- **SM-8 — Week-boundary edit:** with `2026-W46` selected, end-to-end tests must verify that the system accepts 2026-11-09 and 2026-11-15; rejects 2026-11-08 and 2026-11-16 with the exact message; retains the previous valid date; and preserves weekly totals after rejection. Validates FR-2, FR-6, and FR-9.
- **SM-9 — Stock lifecycle:** automated tests must pass the complete Increment 2 gate, including the 480-cup accepted receipt, 980-cup current-balance example, allocation-conflict confirmation, single physical deduction, visible read-only Completed history, and unchanged baseline fixtures after reset. Validates FR-1–FR-4, FR-6, FR-13/FR-14, and FR-25/FR-26.

### Counter-Metrics

- **SM-C1 — AI usage is not success:** generation count or AI wording variety must not be optimized; correctness and keyless usability take priority.
- **SM-C2 — Breadth is not success:** additional products, charts, authentication, or integrations do not compensate for unstable core scenarios.
- **SM-C3 — Historical speed is not planning truth:** changing descriptive supplier history alone must never improve or worsen MRP quantities or fixed-lead-time deadlines. A linked receipt can change stock inputs, but its history statistics never override the calculation rules.

## 11. Risks and Mitigations

- **Subtle MRP defects:** formulas or weekly ordering may look plausible while being wrong. Mitigation: pure calculation boundaries, hand-calculated fixtures, and unit and workflow tests.
- **Scope expansion:** optional UI, AI, or catalog work may displace testing. Mitigation: enforce the increment gates in §9.
- **Provider cost or availability:** `high` reasoning can increase latency and token use, and project-account access may be unavailable. Mitigation: explicit invocation, strict caps and timeout, and complete deterministic fallback.
- **Non-repeatable demo state:** persisted edits can obscure expected results. Mitigation: confirmed reset and scenario-level acceptance tests.
- **Documentation drift:** plans may stop matching implementation. Mitigation: update requirements and README with each scoped change and keep small descriptive commits and review evidence.

## 12. Assumptions Index

No unresolved product assumptions remain. On 2026-11-02 the user confirmed A-1 (confirmation before reset) and A-2 (three isolated selectable planning snapshots, global supplier history, and reset activating `scenario-plan-increase`). These decisions are requirements in UJ-4, FR-1, and FR-4 and are recorded in the workspace memory log.

## 13. Open Questions and Deferred Verification

The stock-movement decisions have been reconciled into the brief, this PRD, MRP-R8–MRP-R10, and the AI regression requirements. The resumed review on 2026-11-03 found no remaining blocking documentation finding. The records below preserve earlier decisions; the current validation report replaces the pre-reconciliation verdict. This final planning status is not a claim that the application has been implemented or tested.

- **OQ-1 — Scenario ownership and receipt linkage (resolved 2026-11-02):** Receipts and consumption affect only the active scenario's stock. A single receipt registration also creates one linked global Delivery Record under FR-13/FR-25. Shared descriptive history never adds stock to another scenario; saved-state and full-reset rules remain in FR-1/FR-3/FR-4.
- **OQ-2 — Completion and allocation (resolved 2026-11-02):** Current physical stock is the basis for outstanding production; Opening Balance is historical. FR-6 and MRP-R2 exclude Completed demand and preserve the agreed allocation and availability meanings. FR-1 and MRP-R10 bound the demo to current-state selected-week planning, not historical reconstruction or cross-week time-phased supply. Allocation-conflict confirmation and read-only Completed history are specified in FR-26/MRP-R9.
- **OQ-3 — Contract and release reconciliation (resolved 2026-11-02):** FR-13/FR-25 and MRP-R8 define a single linked receipt/history registration, explicit unknown Order Sent Date, validation of every supplied date, actual receipt on or before Planning Date, numeric bounds, identity conflicts, and atomic/replay behavior. MRP-R9/MRP-R10 define completion and isolation; the full four-material fixture verifies the flow without altering the three baseline scenarios. Section 9 assigns stock operations to Increment 2, supplier comparison/risk to Increment 3, and optional AI to Increment 4. The AI contract adds stock-event/context stale-response checks without expanding AI authority.

Before optional provider integration, the project owner must verify access to the configured model in the project's own API account, as recorded in [Deferred Work](../../../implementation-artifacts/deferred-work.md). If unavailable, the mandatory standard explanation remains usable; the approved provider configuration must not be silently substituted. Framework, database, and deployment decisions belong to the architecture work, not to this PRD.
