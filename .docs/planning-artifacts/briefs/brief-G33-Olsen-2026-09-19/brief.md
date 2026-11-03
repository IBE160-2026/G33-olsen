---
title: Product Brief: ProsjektMelk
status: complete
created: 2026-09-19
updated: 2026-11-03
---

# Product Brief: ProsjektMelk

## Executive Summary

ProsjektMelk is a browser-based decision-support dashboard for a buyer at a fictional dairy manufacturer. It shows whether materials cover a changed weekly production plan, what is missing, and whether fixed supplier lead times allow a new order to arrive before production starts. The required demo uses three Blueberry Yogurt scenarios. The buyer makes purchasing decisions and coordinates any production-plan changes.

## The Problem

Customer orders can change a production plan set a week ahead. The buyer must quickly determine whether ingredients and packaging, especially cups and lids, still cover the plan. Missing materials or uncertain supplier lead times can cause rushed purchases and production stoppages. Overstock and expiry are secondary concerns; the immediate priority is avoiding shortages that prevent a planned run.

## The Solution

The buyer selects a production week and sees on-hand inventory, quantities automatically allocated to earlier production in that week, and the remaining quantity available before each run. The dashboard highlights shortages with the missing material, quantity, affected production date, recommended order quantity, and latest safe order date.

Fixed lead time provides a planning assessment, not confirmation of supplier availability. If the deadline has passed, the buyer must confirm timely supply or discuss a plan change with production. The buyer can manually edit quantity or date within the selected week and see recalculated results. Supplier follow-up adds descriptive delivery comparisons. Every alert has a standard explanation; configured AI explanations are optional.

One receipt form records total and rejected quantities, Order Sent Date (or explicitly unknown), optional Confirmed Delivery Date from the supplier, and Actual Receipt Date. Only accepted, actually received goods increase stock; the promised date is not proof of receipt. Receipt dates must not be after the scenario Planning Date. Completion confirms calculated BOM consumption once and keeps read-only production history. The demo uses current stock, not historical or future date-effective balances.

## Calculation Rules

The one-level bill of materials (BOM) maps a finished product directly to ingredients and packaging. Gross need is production quantity times BOM quantity per product unit. Net need is gross need minus available inventory, floored at zero; a positive net need is the recommended order quantity. The latest safe order date is production date minus fixed supplier-material lead time in calendar days. Safety stock, minimum quantities, order multiples, and multi-level BOMs are excluded. Delivery history never changes quantities or deadlines. Exact rules, validations, and hand-calculated scenarios are in [MRP Calculation Rules and Demo Scenarios](calculation-rules.md).

## AI Explanation Policy

Every alert immediately shows an English standard explanation based on the application's calculated fictional facts. With valid server-side OpenAI configuration, the buyer may request an optional AI explanation. The model only selects approved templates and existing evidence; the server validates and renders the text. AI never calculates or adds facts or actions. Missing configuration or failed validation leaves the standard explanation usable. The selected model, reasoning setting, server-only API key handling, cost controls, and fallback contract are documented in [AI Explanation and Keyless Fallback Plan](ai-explanation-plan.md).

## What Makes This Different

The real alternatives are spreadsheets and manual checks. The application brings on-hand inventory, allocated and available quantities, material requirements, and ordering deadlines into one buyer-focused view. Changes to the production plan automatically update the calculations and show each shortage with its calculation basis and the affected production run. The intended advantage is helping the buyer understand the consequences of a plan change more quickly; this remains a usability hypothesis to check with a user. It answers the practical question: *What will be short, when will it be short, and what are my options?*

## Who This Serves

The primary user is a procurement buyer responsible for securing ingredients and packaging. Success means identifying a material constraint and a safe next action before production is affected. The production planner is a secondary stakeholder who may inspect the same dashboard, with no separate role, permissions, or workflow in the first version. The buyer enters demo plan changes and coordinates with production outside the app.

## Success Criteria

The minimum submission lets a buyer change a production plan, inspect shortages and their calculation basis, and understand quantities, deadlines, and standard explanations without login or an API key. The provider-independent first version adds goods receipt with deviations, confirmed physical consumption, retained history, and supplier comparison. Current stock equals opening balance plus accepted receipts minus posted consumption; completed production is not counted again. Configured AI remains optional.

The three fictional scenarios—plan increase, packaging shortage, and delayed delivery—must match the linked hand-calculated outcomes, showing affected material, cause, quantity, date, and buyer options. The delayed-delivery risk warning is verified in increment 3. A usability check asks a user to identify the shortage, affected production, and next action without manually calculating requirements.

## Scope + Vision

### First Version

The dashboard runs with preloaded fictional data and no login. Each increment starts only after the previous flow and its tests are stable.

Build each increment in small, verifiable steps: check inventory and allocation before dependent shortage calculations, then verify quantities and deadlines against hand-calculated examples before adding supplier follow-up or AI.

1. **Core MRP demo:** reproduce quantities and deadlines in all three verified baseline scenarios with standard explanations and no API key. This is the minimum complete submission; the delayed-delivery risk warning belongs to increment 3.
2. **Goods receipt and physical consumption:** register total receipt and deviations once, add only accepted quantity to the active scenario's stock, and create linked supplier history. Confirm completed production to deduct consumption once, retain read-only history, and recalculate outstanding demand. Warn about allocation conflicts and allow explicit confirmation when physical stock is sufficient. Verify these flows, persistence, and reset before continuing.
3. **Supplier follow-up:** compare descriptive delivery history and reproduce the delayed-delivery risk warning. History never changes formulas or fixed lead times; physical receipts and consumption change stock inputs and therefore calculated needs.
4. **Optional provider-generated explanation:** offer the user-triggered AI explanation when valid server configuration is available. The core workflow and standard explanation must remain fully usable without a key or provider access.

The seed data may represent the planned dairy range, but the three Blueberry Yogurt scenarios are the required acceptance path. Additional products or polish must not delay a stable, tested core flow.

### Outside the First Version

Trend dashboards for production volume, inventory movement, and supplier performance, plus login and account management, are deferred. A signed anonymous technical session protects optional AI requests without introducing a user account.

The first version also excludes manual reservations, incoming-order supply in calculations, email or real ERP/supplier integrations, automatic ordering or rescheduling, advanced forecasting, full production scheduling/capacity optimization, real company data, and enterprise-grade security. Batch tracing, individual posting reversal, production-consumption deviations, and date-effective supply calculations are possible later extensions documented in [Deferred Work](../../../implementation-artifacts/deferred-work.md).

### Vision

Longer term, the product could add demand forecasting, deeper production scheduling, and ERP integration while preserving buyer control over procurement decisions.
