---
title: 'Prioritize the first-version scope'
type: 'feature'
created: '2026-10-18'
status: 'done'
route: 'oneshot'
review_loop_iteration: 0
context:
  - '.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/tilbakemelding-product-brief.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The product brief presents calculation, deadline alerts, supplier history, delivery registration, AI explanation, trend statistics, and login as one first-version commitment. That breadth risks an incomplete or unstable submission and does not reflect the teacher's recommended implementation order.

**Approach:** Rewrite the first-version scope as three gated increments: first the three verified MRP scenarios, then supplier comparison and delivery registration, and finally optional AI explanation. Move descriptive trend statistics and login outside the first version, while preserving the approved one-level BOM, fixed lead-time rule, fictional data, keyless fallback, and buyer decision-support boundary.

</frozen-after-approval>

## Implementation Notes

- Replaced the flat first-version feature list with three dependency-ordered increments and made the verified MRP scenarios the minimum complete submission.
- Deferred descriptive trends and login while preserving supplier follow-up, optional AI explanation, the approved calculation boundaries, and keyless operation.
- Updated the product-brief memory log so later PRD, architecture, epic, and story work inherits the same priority order.
- Review clarified that increment 1 owns each scenario's MRP result and the standard explanation, while increment 2 owns the delayed-delivery risk warning and increment 3 owns only provider-generated explanation.
- Reconciled the removal of login with the AI endpoint contract by using a signed anonymous demo session for CSRF, deduplication, rate limits, and cache isolation.

## Review Triage Log

- `medium` — Scenario 3 combined an MRP quantity/deadline with a delivery-history warning assigned to a later increment; split its acceptance across increments 1 and 2.
- `medium` — The first draft postponed the mandatory standard explanation until the AI increment; moved deterministic explanation into the core and limited increment 3 to provider-generated output.
- `medium` — Removing login conflicted with authenticated AI endpoint controls; replaced user-account assumptions with a signed anonymous demo session and per-session controls.
- `medium` — The success criteria did not distinguish the minimum submission from the complete first version; added separate, provider-independent completion criteria.
