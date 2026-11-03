# Teacher Feedback Follow-up — 2026-11-03

## Source verification

Reviewed the teacher's [feedback dated 2026-10-06](../../briefs/brief-G33-Olsen-2026-09-19/tilbakemelding-product-brief.md). Before preparing the push, fetched `origin` from the project's GitHub repository. Its feedback file and the local copy have the same Git blob ID, `52a8738bf034276844bfb2ef904e2e29f11e7852`. The fetched remote branch was at `599ba70`; no newer feedback was present there. This is a consistency check against the supplied feedback, not a new approval from the teacher.

## Main requested changes

| Teacher's request | Current planning evidence | Status |
|---|---|---|
| Explicit gross/net need, order quantity, lead-time/deadline rules, exclusions and hand-calculated answers | Calculation contract MRP-R1–MRP-R7 and PRD FR-5–FR-9 define exact rules, calendar-day fixed lead times, validation/partial results, units and baseline scenarios. MRP-R8–MRP-R10 add approved physical-stock rules without changing baseline answers. | Covered in planning; application tests remain to be implemented. |
| Model choice, server-side key handling, keyless fallback and AI restricted to explaining computed facts | Linked AI contract, PRD FR-20–FR-24, empty-key `.env.example`, deterministic Standard Explanations, user-triggered optional provider path and cost controls. Model-account access remains a pre-integration check. | Covered in planning; no live provider test claimed. |
| Smaller, prioritized first version; trends deferred and login reconsidered | Brief and PRD §9 preserve the teacher-defined core as Increment 1. Increment 2 is stock posting, Increment 3 supplier comparison/risk, Increment 4 optional AI. Trends, login, batch tracing, individual reversal and date-effective future-supply calculations are outside v1. | Priority and minimum preserved; later stock operations add scope and must not delay the core. |

## Other feedback carried forward

- One-level BOM, fixed lead-time rule, fictional data and no automatic ordering are retained.
- README now links to the hidden planning/process documents and identifies the existing scenario definitions. Runtime commands and executable seed/test locations cannot be claimed before implementation; DAR-4/DAR-5 require them and a clean-start rehearsal before delivery.
- UX, architecture, source-to-story/test traceability, automated checks and session evidence are requirements for the next phases. The completed document reviews do not substitute for those implementation deliverables.
- The repository contains the updated brief and PRD, decision logs, accepted corrections and independent review reports, preserving the requested planning history.

## Scope judgment

The current plan addresses the teacher's main requested adjustments. Physical receipts, deviations and confirmed consumption are a later user-approved extension, not something the teacher explicitly required or separately approved. Keeping them after the verified core and retaining the deferred-work boundaries is essential to the teacher's scope advice. There is no basis here for promising a grade or claiming that the finished application already satisfies the assessment criteria.
