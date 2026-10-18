# AI Explanation and Keyless Fallback Plan

## Purpose and non-negotiable boundary

ProsjektMelk explains results that its calculation engine has already produced. The engine defined in [MRP Calculation Rules and Demo Scenarios](calculation-rules.md) is the only source of quantities, dates, shortage state, weekly aggregation, supplier statistics, risk state, and buyer options. The model must not calculate, correct, rank, extrapolate, or add any of them.

All explanation data is fictional. Explanations never place an order or change a production plan, inventory value, supplier record, recommendation, deadline, or priority. The application is fully usable without OpenAI: every alert first renders a **Standard explanation** from deterministic server templates and the immutable calculation result.

The deliberately constrained AI path demonstrates optional model-guided presentation inside a closed safety boundary. It is not needed for correctness or usability: deterministic output remains sufficient for every result and fallback path.

## Buyer experience

- Show **Standard explanation** immediately for every valid or blocked result.
- Show **Generate AI explanation** only when the server has validated all three required environment variables, the server-side generation kill switch is enabled, and the calculation result is eligible. Never call OpenAI automatically.
- Keep the standard explanation visible while generation runs. After a valid response, render a separate **AI explanation**. The provider never supplies display-ready prose; it selects a closed explanation plan that the server renders in English from trusted values.
- If configuration, eligibility, endpoint controls, the provider call, or validation fails, keep the alert and standard explanation usable and show only `AI explanation unavailable; showing the standard explanation.` Do not expose the failure category to the browser.
- Present options as choices for the buyer. Never imply that ProsjektMelk has ordered, approved, contacted, expedited, or changed anything.

## Server-only configuration

The committed repository root contains `.env.example` with exactly:

```dotenv
OPENAI_API_KEY=
OPENAI_MODEL=gpt-5.6-sol
OPENAI_REASONING_EFFORT=high
```

Developers copy these names to an ignored local `.env` file and put the real key there. `.gitignore` ignores `.env` and `.env.*` while allowing example files. A key must never be committed, logged, embedded in browser JavaScript, placed in a public environment-variable namespace, sent to the browser, or returned by an endpoint. Only server code may read it and call OpenAI.

Generation is configured only when the trimmed key is non-empty, the trimmed model value is exactly the approved `gpt-5.6-sol`, and `OPENAI_REASONING_EFFORT` is exactly `high`. The server passes the effort separately as `reasoning: { effort: "high" }` to the Responses API; `high` is not part of the model ID. Missing or invalid configuration, including a model unavailable to the account, is a normal fallback condition. The browser may receive one derived boolean, `aiExplanationAvailable`; it receives none of the variables or the reason generation is unavailable.

`gpt-5.6-sol` with `high` reasoning effort is the exact user-selected configuration and must not be silently substituted. Official OpenAI documentation lists the model, Structured Outputs, the Responses API, and `high` effort as supported; access for the project's own API account must still be verified before integration. Until then—and whenever that check later fails—the mandatory standard fallback is used. Higher effort may increase latency and reasoning-token usage, so the endpoint limits below remain mandatory.

## Immutable source data and request construction

The browser sends only `alertId`, the displayed `resultVersion`, and an idempotency key. Before reserving capacity or invoking the provider, the server rejects a displayed version that differs from the current alert version. It then reconstructs the provider request from the same immutable, validated calculation-result snapshot used to render the alert. Client-supplied facts, options, labels, and text are ignored. `resultVersion` is an opaque, immutable revision or digest that changes whenever any input or calculated output used by the explanation changes; a digest is lowercase hexadecimal prefixed with `result-` so it satisfies the safe-ID grammar.

Before a provider call, the server validates the following request rules:

- `contractVersion` is `1`, `language` is `en`, and `scope` is `per-alert` or `weekly`.
- `alertId`, `resultVersion`, every evidence ID, row ID, collection ID, and option ID matches `^[a-z][a-z0-9-]{0,63}$`. All evidence, row, collection, and option IDs are globally unique within the request. Conflicting or duplicate IDs reject the request before the provider call.
- The JSON request is at most 24 KiB and contains at most 128 evidence items, 12 production rows, one collection, and four permitted options. Text values are server-owned fictional names or enum codes, never user-entered free text, and are at most 100 characters.
- Dates use `YYYY-MM-DD`, selected weeks use `YYYY-Www`, numeric values are finite JSON numbers, units come from the calculation result, and arrays use the deterministic ordering below.
- There are no credentials, raw database records, personal data, real-company data, provider errors, or fields not already represented in the buyer-visible result.

The provider request has this closed top-level shape; no additional properties are allowed. The empty arrays below illustrate the property names only and do not form an eligible call for the shown positive-shortage state:

```json
{
  "contractVersion": 1,
  "resultVersion": "result-7f32",
  "language": "en",
  "scope": "per-alert",
  "alertId": "alert-yogurt-cup",
  "alertState": "positive-shortage",
  "reasonCode": "gross-need-exceeds-inventory",
  "evidence": [],
  "productionRows": [],
  "collections": [],
  "riskCode": null,
  "riskBasisRowId": null,
  "permittedOptions": []
}
```

Only these scope/state/reason triples are valid: `per-alert` / `positive-shortage` / `gross-need-exceeds-inventory`; `per-alert` / `no-shortage` / `inventory-covers-gross-need`; and `weekly` / `weekly-positive-shortage` / `weekly-gross-need-exceeds-starting-inventory`. Every other combination is rejected before the provider call. A weekly result with no positive shortage is rendered deterministically and is ineligible for AI generation. `riskCode` is `repeat-delay-could-reach-or-miss`, `historical-supplier-risk`, or `null`; it is an application-calculated value, not a model judgment.

Each evidence item has exactly `id`, `kind`, `value`, and `unit`; `unit` is a string or `null`. `kind` is one of:

`product`, `material`, `material-unit`, `production-date`, `production-quantity`, `bom-quantity`, `gross-need`, `available-inventory`, `net-need`, `recommended-quantity`, `supplier`, `lead-time-days`, `latest-safe-order-date`, `deadline-state`, `overdue-days`, `selected-week`, `weekly-total-gross-need`, `weekly-starting-inventory`, `weekly-recommended-quantity`, `weekly-earliest-shortage-deadline`, `inventory-before`, `covered-quantity`, `sequential-shortage`, `inventory-after`, `valid-delivery-count`, `average-actual-lead-time-days`, `late-delivery-count`, or `greatest-recorded-delay-days`.

`value` is never null. The compact evidence registry below is normative:

| Evidence kind(s) | Exact JSON value | Exact `unit` |
|---|---|---|
| `product`, `material`, `supplier` | Non-blank server-owned string, at most 100 characters | `null` |
| `material-unit` | Non-blank unit code from the calculation result, at most 20 characters | `null` |
| `production-date`, `latest-safe-order-date`, `weekly-earliest-shortage-deadline` | Valid `YYYY-MM-DD` string | `null` |
| `selected-week` | Valid `YYYY-Www` string | `null` |
| `deadline-state` | Exactly `future`, `order-today`, or `overdue` | `null` |
| `production-quantity` | Finite number greater than zero | Product unit from the result |
| `bom-quantity` | Finite number greater than zero | Material unit per product unit, for example `kg/cup` |
| `gross-need`, `available-inventory`, `net-need`, `recommended-quantity`, `weekly-total-gross-need`, `weekly-starting-inventory`, `weekly-recommended-quantity`, `inventory-before`, `covered-quantity`, `sequential-shortage`, `inventory-after` | Finite number zero or greater | Material unit from the result |
| `lead-time-days`, `overdue-days`, `valid-delivery-count`, `late-delivery-count`, `greatest-recorded-delay-days` | Integer zero or greater | `calendar-day` for day values; `delivery` for delivery counts |
| `average-actual-lead-time-days` | Finite number zero or greater, already rounded to one decimal by the calculation engine | `calendar-day` |

Every evidence kind appears at most once in a per-alert result or weekly aggregate. Row evidence may repeat a kind across different rows, but each row has exactly one value for each declared slot and an evidence ID may belong to only one row. The same evidence ID may be reused across different response clauses—for example, the material in both reason and recommendation—but may appear only once within one clause. `value` is copied from the calculation result without model-facing display arithmetic.

Presence is state-specific and closed:

- Per-alert `positive-shortage` requires product, material, material-unit, production date and quantity, BOM quantity, gross need, available inventory, positive net need, an equal positive recommended quantity, supplier, lead-time days, latest safe order date, and deadline state. `overdue-days` is required only for `overdue` and forbidden otherwise.
- Per-alert `no-shortage` requires product, material, material-unit, production date and quantity, BOM quantity, gross need, available inventory, and zero net need. Recommendation, supplier, lead time, deadline, overdue, weekly, row, option, and risk evidence is forbidden.
- `weekly-positive-shortage` requires material, material-unit, selected week, weekly total gross need, weekly starting inventory, positive weekly recommended quantity, supplier, lead-time days, weekly earliest deadline, and the complete row evidence described below. Per-alert production, BOM, net-need, recommendation, deadline-state, and overdue evidence is forbidden.
- The four delivery-history evidence kinds are all required exactly when `riskCode` is non-null and all forbidden when it is null.

### Deterministic English formatting

The renderer does not depend on unspecified application formatters. It escapes names as plain text; preserves validated ISO dates and week strings; formats integer values as base-10 integers; formats quantities with up to three decimal places using round-half-up and removes trailing zeroes; formats average actual lead time with exactly one decimal place; and separates a number from its display unit with one space. `calendar-day` renders as `calendar day` or `calendar days` by value, while `delivery` renders as `delivery` or `deliveries`. Deadline states render exactly as `future`, `Order today`, or `Overdue by N calendar days`. All fixed words and option labels come from the English template and option registries in this document.

### Per-alert facts

For a valid per-alert result, include the applicable product, material, production date and quantity, BOM quantity, gross need, usable inventory, net need, material unit, recommendation, fictional supplier, fixed lead time, latest safe order date, deadline state, and overdue days. Include supplier-history evidence and `riskCode` only when the calculation engine has already produced the corresponding risk warning.

### Weekly facts

For a weekly alert, also include selected week, weekly total gross need, starting inventory, aggregated supplier-material recommended quantity, and the earliest deadline among rows with a positive sequential shortage. `productionRows` contains the engine's rows in ascending production date, then ascending product name, then ascending row ID as the final stable tie-breaker. Every row has exactly:

```json
{
  "id": "row-1",
  "productEvidenceId": "product-1",
  "productionDateEvidenceId": "date-1",
  "grossNeedEvidenceId": "gross-1",
  "inventoryBeforeEvidenceId": "inventory-before-1",
  "coveredQuantityEvidenceId": "covered-1",
  "sequentialShortageEvidenceId": "shortage-1",
  "inventoryAfterEvidenceId": "inventory-after-1"
}
```

The row slots require `product`, `production-date`, `gross-need`, `inventory-before`, `covered-quantity`, `sequential-shortage`, and `inventory-after` respectively, and each referenced item may belong to only one row. A single `weekly-sequence` collection lists every row ID in exactly the displayed order. The model can reference that collection but cannot reorder, omit, or edit its rows. This preserves sequential inventory consumption and prevents inventory from being counted twice.

The collection has exactly this shape: `{ "id": "sequence-1", "kind": "weekly-sequence", "rowIds": ["row-1"] }`. `rowIds` contains every and only production-row ID once. Per-alert requests have no collections or production rows.

If any row with a positive sequential shortage has unavailable deadline input, the aggregate deadline is `deadline-unavailable`; the standard explanation shows every valid quantity plus the applicable lead-time message, and the weekly result is ineligible for AI generation. When weekly `riskCode` is non-null, `riskBasisRowId` is required and equals the first positive-shortage row in the ordering above that is affected by that supplier-material risk. It is `null` for per-alert requests and weekly requests without risk.

Each permitted option has exactly `id` and `code`. `code` is one of `order-recommended-quantity`, `change-production-plan`, `verify-usable-inventory`, `contact-supplier`, `seek-expedited-or-alternative-supply`, or `accept-supplier-risk`. The server owns the English text for every code.

The server, not the model, constructs the exact ordered option set. The response `optionIds` must match it byte-for-byte in this order:

| Alert state and risk | Ordered option codes |
|---|---|
| `positive-shortage` or `weekly-positive-shortage`, `riskCode: null` | `order-recommended-quantity`, `change-production-plan`, `verify-usable-inventory` |
| `positive-shortage` or `weekly-positive-shortage`, non-null `riskCode` | `contact-supplier`, `seek-expedited-or-alternative-supply`, `change-production-plan`, `accept-supplier-risk` |
| `no-shortage` | Empty |

## Eligibility and deterministic blocked handling

Skip the model call whenever a required production, BOM, unit, inventory, lead-time, or supplier input is missing or invalid. Preserve the calculation contract's partial-result rule: invalid lead time blocks only the deadline, so the standard explanation still includes any valid quantity recommendation and its supplier, followed by the exact lead-time validation message. A positive shortage without a supplier keeps its calculated quantity but displays `Cannot prepare a purchase option: a supplier is required for <material>.`

Collect all applicable validation messages; never stop after the first. Render them in the calculation-rules table order, then by material name and stable field ID. This makes multiple failures complete and deterministic. A blocked result never manufactures a recommendation, deadline, supplier, option, or risk statement.

## Closed model response

The system instruction says that the model is selecting an English explanation plan, not writing prose or acting as a calculator or purchasing agent. The OpenAI request must use strict structured output against the closed response schema below, with `additionalProperties: false` at every object level, required fields at every level, the documented enums and bounds, and no free-form string field. If the selected model or account cannot enforce that strict schema, treat the provider as unavailable and use the standard fallback; do not downgrade to JSON mode or prose. The provider must return one JSON object with this exact top-level schema and no additional properties:

```json
{
  "contractVersion": 1,
  "resultVersion": "result-7f32",
  "language": "en",
  "clauses": [
    {
      "templateId": "shortage-reason-v1",
      "bindings": [
        { "slot": "material", "sourceId": "material-1" },
        { "slot": "grossNeed", "sourceId": "gross-1" },
        { "slot": "availableInventory", "sourceId": "inventory-1" },
        { "slot": "netNeed", "sourceId": "net-1" }
      ]
    },
    {
      "templateId": "purchase-recommendation-v1",
      "bindings": [
        { "slot": "material", "sourceId": "material-1" },
        { "slot": "recommendedQuantity", "sourceId": "recommendation-1" },
        { "slot": "supplier", "sourceId": "supplier-1" }
      ]
    },
    {
      "templateId": "deadline-date-v1",
      "bindings": [
        { "slot": "latestSafeOrderDate", "sourceId": "deadline-1" }
      ]
    },
    {
      "templateId": "buyer-options-v1",
      "bindings": []
    }
  ],
  "optionIds": ["option-order", "option-plan", "option-stock"]
}
```

The raw response must be at most 4 KiB. `clauses` contains 1–6 unique entries; each contains only `templateId` and `bindings`. `bindings` contains 0–8 entries with unique slots and contains only `slot` and `sourceId`; zero bindings are valid only for `buyer-options-v1`. `optionIds` contains 0–4 unique IDs. Template IDs, source IDs, and option IDs use the 64-character ID grammar; slot names are the exact case-sensitive names in the registry. Empty plans and blank strings are invalid. Because no field accepts free-form prose, markup, HTML, Markdown, code, and unsupported natural-language claims are outside the schema.

The template registry is a closed discriminated union:

| Template ID | Exact slots and required source kind | Applicability |
|---|---|---|
| `shortage-reason-v1` | `material` → `material`; `grossNeed` → `gross-need`; `availableInventory` → `available-inventory`; `netNeed` → `net-need` | Per-alert positive shortage; all four slots required |
| `purchase-recommendation-v1` | `material` → `material`; `recommendedQuantity` → `recommended-quantity`; `supplier` → `supplier` | Per-alert positive shortage; all three required |
| `deadline-date-v1` | `latestSafeOrderDate` → `latest-safe-order-date` | Positive shortage with a future date |
| `deadline-today-v1` | `latestSafeOrderDate` → `latest-safe-order-date`; `deadlineState` → `deadline-state` | Positive shortage whose state is `order-today` |
| `deadline-overdue-v1` | `latestSafeOrderDate` → `latest-safe-order-date`; `overdueDays` → `overdue-days` | Positive shortage whose state is `overdue` |
| `no-shortage-v1` | `material` → `material`; `grossNeed` → `gross-need`; `availableInventory` → `available-inventory`; `netNeed` → `net-need` | Per-alert no shortage; `netNeed` must equal zero |
| `weekly-shortage-v1` | `selectedWeek` → `selected-week`; `material` → `material`; `totalGrossNeed` → `weekly-total-gross-need`; `startingInventory` → `weekly-starting-inventory`; `recommendedQuantity` → `weekly-recommended-quantity`; `earliestDeadline` → `weekly-earliest-shortage-deadline`; `supplier` → `supplier`; `sequence` → the sole `weekly-sequence` collection | Weekly positive shortage; all eight required |
| `supplier-risk-v1` | `supplier` → `supplier`; `validDeliveryCount` → `valid-delivery-count`; `averageActualLeadTimeDays` → `average-actual-lead-time-days`; `lateDeliveryCount` → `late-delivery-count`; `greatestDelayDays` → `greatest-recorded-delay-days`; `riskBasisRow` → the exact `riskBasisRowId` row for weekly scope only | Only when non-null `riskCode` and a positive shortage exist; `riskBasisRow` is required for weekly scope and forbidden for per-alert scope |
| `buyer-options-v1` | no evidence bindings | Positive shortage; response `optionIds` supplies the options |

The server owns clause order as well as option order. The response must match one exact sequence:

| Alert state and risk | Exact clause order |
|---|---|
| `positive-shortage`, `riskCode: null` | `shortage-reason-v1`, `purchase-recommendation-v1`, the one applicable deadline template, `buyer-options-v1` |
| `positive-shortage`, non-null `riskCode` | `shortage-reason-v1`, `purchase-recommendation-v1`, the one applicable deadline template, `supplier-risk-v1`, `buyer-options-v1` |
| `weekly-positive-shortage`, `riskCode: null` | `weekly-shortage-v1`, `buyer-options-v1` |
| `weekly-positive-shortage`, non-null `riskCode` | `weekly-shortage-v1`, `supplier-risk-v1`, `buyer-options-v1` |
| `no-shortage` | `no-shortage-v1` |

Bindings within each clause must follow the slot order shown in the template registry. `optionIds` must exactly equal the applicable server-owned ordered set above, not a permutation or subset. Unknown or repeated templates, reordered clauses or bindings, unsupported combinations, missing clauses, inapplicable risk or deadline templates, and reordered or incomplete options reject the whole plan.

### Deterministic template registry

The registry below is the rendering contract. Bracketed values use the deterministic English formatting contract above; every bracket corresponds to the exact typed slot above.

- `shortage-reason-v1`: `[material] has a shortage because gross need is [grossNeed] and usable inventory is [availableInventory], leaving a net need of [netNeed].`
- `purchase-recommendation-v1`: `The application recommends [recommendedQuantity] of [material] from [supplier].`
- `deadline-date-v1`: `The latest safe order date is [latestSafeOrderDate].`
- `deadline-today-v1`: `The latest safe order date is [latestSafeOrderDate]: order today.`
- `deadline-overdue-v1`: `The latest safe order date is [latestSafeOrderDate], which is overdue by [overdueDays] calendar days.`
- `no-shortage-v1`: `[material] has gross need [grossNeed] and usable inventory [availableInventory], so net need is [netNeed] and no purchase is recommended.`
- `weekly-shortage-v1`: `For [selectedWeek], [material] has total gross need [totalGrossNeed] and starting inventory [startingInventory]. Sequential consumption is: [sequence]. The weekly recommended quantity from [supplier] is [recommendedQuantity], and the earliest positive-shortage deadline is [earliestDeadline].` The `[sequence]` renderer emits every row, in collection order, as `[product] on [productionDate]: gross need [grossNeed], inventory before [inventoryBefore], covered [coveredQuantity], shortage [sequentialShortage], inventory after [inventoryAfter].`
- `supplier-risk-v1`: `[supplier] has [validDeliveryCount] valid complete deliveries with average actual lead time [averageActualLeadTimeDays] days, [lateDeliveryCount] late deliveries, and a greatest recorded delay of [greatestDelayDays] calendar days.` For weekly scope, append `The risk is attached to [product] on [productionDate].`, resolving both values from `[riskBasisRow]`. Append `A repeat delay could reach or miss the production date.` only for `repeat-delay-could-reach-or-miss`; append `This is historical supplier risk and does not change the calculated quantity or deadline.` only for `historical-supplier-risk`.
- `buyer-options-v1`: `Buyer options: [options].` The option renderer emits the complete `optionIds` set in response order using these exact code-to-text mappings: `order-recommended-quantity` → `Order the recommended quantity by the displayed deadline`; `change-production-plan` → `Change the production plan`; `verify-usable-inventory` → `Verify usable inventory`; `contact-supplier` → `Contact the named supplier`; `seek-expedited-or-alternative-supply` → `Seek expedited or alternative supply`; `accept-supplier-risk` → `Accept the displayed supplier risk`.

## Validation, version binding, and rendering

Before display, the server performs all checks below and rejects the entire response on the first failure category:

1. Enforce byte limits, parse exactly one JSON object, apply the closed schema, reject additional properties, enforce string and array bounds, and reject blank or duplicate values.
2. Require exact `contractVersion`, `resultVersion`, and `language: "en"`. If the current alert's version no longer equals the request and response version, discard the response as stale without rendering it.
3. Reject unknown, missing, duplicate, or conflicting request IDs and response references. Every response `sourceId` must resolve to an evidence, collection, or explicitly permitted row ID, and every option ID must resolve to the same immutable request snapshot.
4. For every binding, require the exact slot name and source kind in the registry. This semantic slot check prevents a valid quantity or date from being swapped into a different meaning.
5. Enforce alert-state template combinations, risk and deadline preconditions, the complete required template set, the complete option set, and weekly collection completeness and order.
6. Render only through server-owned English templates. Each template interpolates every required binding in its declared semantic slot; there is no path that displays provider text. Escape interpolated fictional names and render the result as plain text, never trusted HTML.

This guarantees that referenced quantities, dates, reasons, risks, and options actually appear in their correct meanings. A schema-valid but incomplete, stale, swapped, blank, non-English, or unsupported plan falls back rather than being repaired in the browser.

## Endpoint cost and concurrency controls

The future server endpoint must apply all of these controls before calling OpenAI:

- Require a server-issued, signed anonymous demo session plus same-origin/CSRF protection. The session grants access only to the application's preloaded fictional alerts and creates no user account or login flow.
- Accept an idempotency key of 1–64 safe ID characters. Apply one atomic reservation operation covering the demo-session/alert/result tuple state and all shared attempt/concurrency counters before checking or changing them. Precedence inside it is: (1) the same idempotency key joins the existing pending work or reuses its validated result for 10 minutes without consuming another attempt; (2) a different key while work is active receives the standard fallback state and starts no call; (3) only when neither exists may the server check caps, increment counters, and reserve one active slot. This prevents concurrent checks across both identical and different tuples from racing.
- Limit each anonymous demo session to five provider attempts per rolling 10 minutes and 20 per rolling 24 hours. Also allow at most two provider calls concurrently across the project and at most 100 project provider attempts per UTC day. Reserved global slots are released on every completion, cancellation, or failure. Rejected, joined, duplicate, configuration-failed, and stale requests do not consume an attempt. Do not automatically retry provider failures.
- Provide a server-only runtime `aiGenerationEnabled` kill switch checked inside the same reservation transaction. It defaults off outside an explicitly configured demo run and immediately prevents new provider calls when disabled; its value is never browser-controlled.
- Cap the request and response at the sizes and collection limits above and configure at most 500 output tokens.
- Set an eight-second provider deadline. Abort the provider request on timeout or client disconnect with an actual cancellation signal; ignore any late completion and never attach it to an alert.
- Cache only a validated explanation plan. Its identity includes anonymous demo-session scope, alert ID, result version, exact model name, `contractVersion: 1`, `templateRegistryVersion: templates-v1`, and `optionPolicyVersion: options-v1`. Increment the matching version whenever the response contract, template text/bindings, or option sets/order change. Any change to the model or one of those versions is a cache miss. Revalidate the complete current request, response, policy versions, and result version before returning cached or newly generated content.

Endpoint responses never reveal whether a failure came from credentials, account access, model availability, validation, or provider behavior. For demo-session or project rate/cap failures only, the same generic fallback response may include a non-secret integer `retryAfterSeconds`, from 1 through 86,400, calculated as the time until the applicable rolling window or UTC-day cap can admit another attempt; it does not identify which cap fired or disclose provider details.

## Deterministic standard rendering

Standard and AI rendering consume the same immutable result snapshot, option order, clause order, and server-owned sentence templates. The standard path builds the prescribed plan itself; the constrained AI path must return that same closed structure and evidence binding plan before it passes through validation and the shared renderer.

At minimum, deterministic rendering covers:

- positive recommendation: material, gross need, usable inventory, net need/recommended quantity, supplier, deadline state/date, and every permitted buyer option;
- weekly recommendation: selected week, total gross need, starting inventory, every ordered sequential-consumption row, aggregated supplier-material recommendation, earliest positive-shortage deadline, and options;
- no shortage: gross need is covered, net need is zero, and no purchase is recommended;
- deadline unavailable: valid quantity and supplier followed by the exact lead-time validation message, without an invented date;
- all blocked inputs: every exact validation message in deterministic order and no manufactured output;
- supplier risk: only the calculated historical summary and risk category, without changing quantity or deadline.

## Failure and logging policy

Use the standard explanation for missing or invalid configuration, ineligible or stale results, duplicate or rate-limited attempts, timeout, disconnect, network failure, authentication failure, unavailable model, provider error, empty or oversized response, malformed JSON, schema failure, binding failure, unsupported combination, or any other validation failure. The alert, arithmetic, recommendation, and buyer options remain visible.

Logs may contain only a non-secret diagnostic category, local correlation ID, safe model name, elapsed time, result version, and validation failure code. Never log the key, authorization headers, complete provider request or response, raw provider error body, raw database rows, or browser-visible credentials.

## Required verification contract

Future automated contract tests must run and pass for all four matrix paths: configured-valid generation, no key, provider unavailable (including timeout cancellation, unavailable model, and unsupported strict-schema capability), and blocked calculation. Additional negative cases must cover unknown, missing, and duplicate IDs; conflicting request IDs; evidence types, units, nullability, and state presence; extra properties; blank plans; array and byte limits; wrong language or version; a displayed-version mismatch before invocation; stale result replacement; swapped semantic bindings; omitted required templates or values; reordered clauses, bindings, or options; empty/subset options; inapplicable risk/deadline templates; missing supplier; multiple validation errors and their order; weekly row tie ordering, deadline-unavailable aggregation, and earliest-row risk binding; Markdown/HTML-shaped strings; same-key joining, different-key rejection, atomic racing requests, demo-session and project caps, the two-call global ceiling, kill-switch behavior, cache policy-version invalidation, safe retry delays; disconnect and timeout abortion; and late provider completion. Tests must assert that every rejection preserves the standard explanation and that no rejected or stale plan replaces current content.

A source review must additionally confirm that environment access and provider calls exist only in server code, browser bundles contain no key or provider configuration, tracked files contain no real secret, and no provider-supplied text reaches the renderer.
