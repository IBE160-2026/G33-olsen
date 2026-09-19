---
title: Product Brief: ProsjektMelk
status: complete
created: 2026-09-19
updated: 2026-09-19
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

## What Makes This Different

The real alternatives are disconnected spreadsheets, manual checks, or acting late when a shortage is discovered. This approach brings the production plan, product structure, inventory, supplier delivery history, and packaging risk into one buyer-focused view. It answers the practical question: *What will be short, when will it be short, and what are my options?*

The application is deliberately transparent: it shows the basis for an alert and lets the buyer investigate recommendations or compare suppliers. The AI explains the result, but the buyer retains responsibility for the decision. The advantage is a tighter, more understandable workflow—not an invented technical moat.

## Who This Serves

The primary user is a procurement planner or buyer responsible for securing ingredients and packaging for a dairy manufacturer. They need to understand the consequences of a changing production plan and decide whether to place or adjust an order. Success for them is identifying a material risk and a safe next action before production is affected. A production planner is a secondary user who benefits from seeing material constraints before changing the schedule.

## Success Criteria

The product is working when a buyer can change a weekly production plan, identify a packaging or ingredient shortage, investigate a safe order date and supplier history, and understand the AI explanation before production is affected.

For the demo, every predefined fictional scenario must be handled correctly: a production-plan increase, a packaging shortage, and a delayed supplier delivery. Each must show the affected material, the reason for the warning, and relevant buyer options. A usability check asks a user to identify the next action from the dashboard without manually calculating requirements. The project’s mission signal is that procurement risk becomes visible early enough to support an action before a planned production run.

## Scope + Vision

### First Version

The first release is a small, interactive browser dashboard with a simple login. It uses preloaded fictional data for a dairy product range: three yogurt flavours, milk with different fat percentages, butter, and sour cream. The data includes product BOMs, inventory levels, supplier details, lead times, recorded deliveries, and weekly production plans. Simple forms let the user change a plan and record a delivery so the demo remains interactive.

Version one calculates material requirements, flags shortages and ordering deadlines, records supplier delivery performance, provides AI explanations, and shows descriptive trends. The statistics cover production-volume trends by product, inventory movement by material, and supplier performance such as average lead time and late deliveries.

### Outside the First Version

It does not include real ERP or supplier-system integrations, automatic order placement, advanced demand forecasting, full production scheduling or capacity optimization, real company data, or enterprise-grade security.

### Vision

Within two to three years, the product could expand with demand forecasting, deeper production scheduling, and integration with a real ERP. These phases would improve data quality, support production execution, and connect the application to live operational data while preserving its core goal: helping buyers understand and act on procurement risk.
