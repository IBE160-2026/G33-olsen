---
title: Product Brief: ProsjektMelk
status: complete
created: 2026-09-19
updated: 2026-10-18
---

# Product Brief: ProsjektMelk

## Executive Summary

The AI-Assisted Dairy Procurement Planner is a browser-based decision-support tool for a fictional dairy manufacturer. It helps procurement planners prevent production stoppages by showing how a weekly production plan affects material availability, supplier risk, and purchase needs. It supports—not replaces—the buyer’s judgment.

It matters now because a production plan set a week ahead can change when customer orders change. The buyer must quickly judge whether ingredients and packaging, especially cups and lids, will still arrive in time. A clearer overview supports safer decisions before a shortage stops production.

## The Problem

Production plans are set about a week in advance but can change when customer orders change. A late plan adjustment can expose a shortage of essential materials—particularly packaging such as cups and lids—and stop production. A buyer may also need to account for many suppliers with different actual lead times, without a clear record of the time from placing an order to receiving the goods.

The cost of the status quo is lost production time, rushed purchasing decisions, and less confidence in whether an order will arrive in time. Overstock and expiry remain relevant for dairy operations, but the immediate priority is avoiding material shortages that prevent a planned run from taking place.

## The Solution

When a planner selects or updates a production week, the buyer receives a clear priority list:

1. Immediate shortage warnings, for example a blueberry-yogurt lid shortage before Thursday.
2. Recommended order quantities and the latest safe order dates.
3. Supplier-delivery comparisons based on recorded order and delivery dates.

The buyer can investigate the underlying information, compare suppliers, and choose an appropriate response. An AI assistant explains each warning or recommendation in plain language. The result is a faster, safer decision process rather than an automated ordering system.

## Calculation Rules

The first version uses a deliberately small, one-level BOM model: a finished product maps directly to ingredients and packaging. Gross need is planned product quantity times the BOM quantity; net need is the positive remainder after usable on-hand inventory. A positive net need is the recommended order quantity. The latest safe order date is the production date minus the fixed lead time for that supplier-material combination. There is no safety stock, minimum order quantity, order multiple, multi-level BOM explosion, or automatic purchase placement. Delivery history describes supplier risk but does not change calculated quantities or deadlines. The complete definitions, validations, and hand-calculated demo inputs are in [MRP Calculation Rules and Demo Scenarios](calculation-rules.md).

## AI Explanation Policy

Every alert immediately shows an English standard explanation rendered from the application's calculated fictional facts. With valid server-side OpenAI configuration, the buyer may explicitly request an AI explanation using `gpt-5.6-sol`; the model selects only a closed plan of approved templates and evidence bindings, while the server validates and renders the text. It never calculates, changes, or adds facts or actions. Missing configuration, stale results, provider failures, or invalid plans leave the alert and standard explanation fully usable. The key handling, versioned contract, endpoint controls, validation, and fallback rules are defined in [AI Explanation and Keyless Fallback Plan](ai-explanation-plan.md).

## What Makes This Different

The real alternatives are disconnected spreadsheets, manual checks, or acting late when a shortage is discovered. This approach brings the production plan, product structure, inventory, supplier delivery history, and packaging risk into one buyer-focused view. It answers the practical question: *What will be short, when will it be short, and what are my options?*

The application is deliberately transparent: it shows the basis for an alert and lets the buyer investigate recommendations or compare suppliers. The AI explains the result, but the buyer retains responsibility for the decision. The advantage is a tighter, more understandable workflow—not an invented technical moat.

## Who This Serves

The primary user is a procurement planner or buyer responsible for securing ingredients and packaging for a dairy manufacturer. They need to understand the consequences of a changing production plan and decide whether to place or adjust an order. Success for them is identifying a material risk and a safe next action before production is affected. A production planner is a secondary user who benefits from seeing material constraints before changing the schedule.

## Success Criteria

The minimum submission is working when a buyer can change a weekly production plan, identify a packaging or ingredient shortage, verify the displayed calculation and latest safe order date, and understand the immediate standard explanation without a login, API key, or manual calculation. The complete first version additionally lets the buyer inspect and record supplier deliveries and, when configured, request an optional AI explanation without making that provider path necessary for the workflow.

For the demo, every predefined fictional scenario must match the verified outcomes in the [MRP Calculation Rules and Demo Scenarios](calculation-rules.md): a production-plan increase, a packaging shortage, and a delayed supplier delivery. Each must show the affected material, the reason for the warning, its hand-checkable numeric result, and relevant buyer options. A usability check asks a user to identify the next action from the dashboard without manually calculating requirements. The project’s mission signal is that procurement risk becomes visible early enough to support an action before a planned production run.

## Scope + Vision

### First Version

The first release is a small, interactive browser dashboard that runs with preloaded fictional data and no login. Its implementation is deliberately gated into three increments; work moves to the next increment only after the earlier increment and its tests are stable.

1. **Core MRP demo:** let the buyer change a weekly production plan, calculate material requirements from the one-level BOM, show shortages and latest safe order dates, and immediately show the approved English standard explanation. It must reproduce the MRP quantities and deadlines in all three demo scenarios; the delayed-delivery warning itself belongs to increment 2. This is the minimum complete submission and takes priority over every later increment.
2. **Supplier follow-up:** compare the descriptive delivery history already defined by the calculation rules, reproduce the delayed-delivery scenario's supplier-risk warning, and let the buyer record a delivery. This increment must not change calculated quantities or fixed-lead-time deadlines.
3. **Optional provider-generated explanation:** offer the user-triggered AI explanation when valid server configuration is available. The core workflow and standard explanation must remain fully usable without a key or provider access.

The seed data may represent the planned dairy range, but the three Blueberry Yogurt scenarios are the required acceptance path. Additional products or polish must not delay a stable, tested core flow.

### Outside the First Version

Descriptive trend dashboards for production volume, inventory movement, and supplier performance are deferred until after the first version. Login and user-account management are also deferred because a single-role fictional-data demo does not need a user identity to demonstrate its core value. The server may still issue a signed anonymous demo session for CSRF protection, AI-call deduplication, rate limits, and cache isolation; this is a technical control, not a user account or login flow.

The first version also excludes real ERP or supplier-system integrations, automatic order placement, advanced demand forecasting, full production scheduling or capacity optimization, real company data, and enterprise-grade security.

### Vision

Within two to three years, the product could expand with demand forecasting, deeper production scheduling, and integration with a real ERP. These phases would improve data quality, support production execution, and connect the application to live operational data while preserving its core goal: helping buyers understand and act on procurement risk.
