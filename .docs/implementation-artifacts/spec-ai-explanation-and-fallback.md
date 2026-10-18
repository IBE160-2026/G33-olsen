---
title: 'Plan AI explanations and keyless fallback'
type: 'feature'
created: '2026-10-18'
status: 'done'
route: 'dispatch'
review_loop_iteration: 1
baseline_commit: 'c9bb9be80d6cb052fea8553b7154d77b458b6748'
context:
  - '.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/brief.md'
  - '.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/calculation-rules.md'
  - '.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/tilbakemelding-product-brief.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The brief promises AI explanations but does not identify the model, how the API key is handled, what facts the model may use, or how the app remains runnable for an assessor without a key.

**Approach:** Define an optional server-side language-model explanation path and a deterministic template fallback. Both paths explain the same application-calculated facts; neither path calculates quantities, dates, shortages, supplier statistics, or priorities.

## Boundaries & Constraints

**Always:** Treat the calculation engine as the only numerical source of truth; send the model only fictional, structured facts already shown by the app; keep the API key server-side in an ignored local environment file; provide a committed example variable name with no secret; validate the returned explanation before display; and fall back on missing configuration, timeout, provider error, malformed output, or unsupported claims.

**Never:** Let the model alter calculations, invent quantities or dates, place orders, receive real company or personal data, expose the API key to browser code, require a paid account for the assessor, or make the core shortage workflow depend on network access.

## Approved Decisions

- Use OpenAI with `gpt-5.6-sol` as the configured model and `OPENAI_MODEL` as the server-side model variable. Current API availability was not established by the official OpenAI documentation search, so an unavailable-model response must use the same safe fallback as any provider failure.
- Produce buyer-facing explanations in English. The user may continue collaborating on the project in Norwegian.
- Show the deterministic standard explanation immediately. When a server-side API key is configured, offer a **Generate AI explanation** action rather than calling the model automatically.

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|---|---|---|---|
| Configured model | Valid calculated alert facts and a usable server-side key | A clearly labelled AI explanation restates the supplied reason, numbers, deadline, risk, and permitted buyer options | Reject output that introduces unsupported numbers, dates, suppliers, or actions; show fallback instead |
| No key | Valid calculated alert facts but no API key | A clearly labelled standard explanation is rendered from deterministic templates | No error blocks the alert or recommendation |
| Provider unavailable | Timeout, rate limit, network failure, or invalid model response | The same alert remains usable and the standard explanation is shown | Log a non-secret diagnostic; never display credentials or raw provider errors |
| Calculation blocked | Required BOM, inventory, production, or lead-time input is invalid | Explanation describes the specific missing input and does not manufacture a recommendation | Model call is skipped |

</frozen-after-approval>

## Code Map

- `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/brief.md` -- promises AI explanations and assessor-runnable v1 behavior; add only a concise policy and link.
- `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/calculation-rules.md` -- numerical source of truth whose facts may be explained but never recomputed by the model.
- `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/tilbakemelding-product-brief.md` -- requires a named model, key location, keyless fallback, and explicit explanation-only responsibility.
- `.gitignore` -- already ignores `.env` and `.env.*` while preserving example files; reuse this policy and add a real non-secret `.env.example`.
- `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/.memlog.md` -- append approved AI-boundary and fallback decisions for downstream PRD work.

## Tasks & Acceptance

**Execution:**
- [x] `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/ai-explanation-plan.md` -- document the approved model, complete server configuration, weekly and per-alert trusted input, result-version binding, closed response schema with concrete limits, deterministic rendering, endpoint cost controls, template fallback, labels, and failure behavior -- provides an enforceable implementation contract requested by the teacher.
- [x] `.env.example` -- add empty `OPENAI_API_KEY` and the approved `OPENAI_MODEL` value only -- gives developers and the assessor a copyable configuration example without a secret.
- [x] `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/brief.md` -- add a concise AI-explanation policy and link to the detailed plan -- keeps the product promise traceable.
- [x] `.docs/planning-artifacts/briefs/brief-G33-Olsen-2026-09-19/.memlog.md` -- record the approved model, language, interaction, key, and fallback decisions -- preserves the rationale.

**Acceptance Criteria:**
- Given an assessor runs the future app without an API key, when an alert is opened, then every calculation and a readable standard explanation remain available.
- Given a configured model receives an explanation request, when the request is built, then it contains only application-calculated fictional facts and permitted options.
- Given model output adds unsupported numerical or operational claims, when it is validated, then it is rejected and replaced by the deterministic fallback.
- Given configuration documentation is inspected, when the key instructions are followed, then the real key stays uncommitted and unavailable to browser code.
- Given an alert changes while generation is in flight, when the older response arrives, then its result-version mismatch prevents it from replacing the current explanation.
- Given repeated generation attempts or an expired request, when endpoint controls run, then duplicate paid calls are rejected and timed-out provider work is aborted.

## Implementation Notes

- Added a documentation-level contract only; application code and provider integration remain future implementation work.
- Used the repository's existing `.env` ignore policy and documented empty/example values rather than adding a credential.
- Review loop 1 reverted the first derivation because its free-form output contract could not enforce the approved source-of-truth boundary.

## Spec Change Log

- 2026-10-18: Added the AI explanation implementation contract, linked it from the product brief, and recorded the approved decisions in the brief memory log.
- 2026-10-18 review loop 1: Review found internally inconsistent evidence IDs, unenforceable free-form claim validation, missing weekly facts and result-version binding, undefined schema limits, absent endpoint controls, and incomplete edge-case verification. The execution tasks and acceptance criteria now require a closed response plan referencing unique evidence IDs, deterministic server rendering, a versioned weekly/per-alert payload, concrete limits, stale-response rejection, deduplication/rate/concurrency controls, timeout cancellation, a real `.env.example`, and complete negative cases. Avoids accepting stale, swapped, invented, incomplete, or financially unbounded AI output. KEEP the approved model, English output, user-triggered generation, server-only key, immediate standard explanation, and keyless/provider-failure fallback.

## Review Triage Log

- `maybe-false` — The requested `gpt-5.6-sol` API availability could not be established from current official documentation. The exact account model list would settle whether the AI path works; the documented unavailable-model fallback prevents core-workflow failure, and the model choice remains user-owned; route: defer if still unresolved after re-derivation.
- `medium` — The example placed `shortage-reason` in the response `factIds` even though it was not a request fact, so a literal validator would reject the example; route: bad_spec.
- `medium` — Free-form prose plus allowlisted IDs cannot reliably reject unsupported qualitative claims, so the sole-source-of-truth boundary was not enforceable; route: bad_spec.
- `medium` — Weekly aggregation and sequential inventory-consumption facts from the calculation contract were absent, preventing accurate explanation of weekly recommendations; route: bad_spec.
- `medium` — Requests and responses were not bound to a calculation-result revision, allowing a stale explanation beside updated numbers; route: bad_spec.
- `medium` — The exact response schema and concrete output limits were promised but not defined, leaving incompatible validation implementations; route: bad_spec.
- `low` — The document labelled a code block `.env.example` without adding that file. The smallest direct correction is to add the promised non-secret example file; route: patch, moot during bad-spec loopback.
- `medium` — The generation endpoint had no documented authorization, rate, concurrency, deduplication, or usage controls, exposing paid calls to repetition or abuse; route: bad_spec.
- `low` — Button availability checked only the key rather than complete validated provider configuration, exposing an action known to fail; route: patch, moot during bad-spec loopback.
- `medium` — Verification omitted unknown/missing/duplicate IDs, extra properties, size limits, markup, stale results, binding errors, and unsupported-claim cases needed to enforce the contract; route: bad_spec.
- `medium` — An in-flight response could be attached after the alert facts changed. This is the same missing result-revision contract independently confirmed by the edge-case review; route: bad_spec.
- `medium` — Repeated clicks could create concurrent paid calls and race to replace the displayed explanation; route: bad_spec.
- `medium` — Duplicate fact or option IDs with conflicting values made response references ambiguous; route: bad_spec.
- `medium` — Multiple invalid calculation inputs had no deterministic ordering or complete display rule, so the explanation could hide additional blockers; route: bad_spec.
- `medium` — A positive shortage with no supplier had no blocked/fallback rule, allowing an empty supplier and unusable purchase option; route: bad_spec.
- `medium` — A schema-valid but blank explanation could pass without an explicit non-blank constraint; route: bad_spec.
- `medium` — Referencing required fact IDs did not require their values to appear in the output, so a valid response could omit the promised quantity, deadline, or reason; route: bad_spec.
- `high` — Allowlisted values could be swapped between fact meanings, allowing incorrect arithmetic to pass value-membership validation; route: bad_spec.
- `medium` — Empty option references could omit buyer choices despite the explanation contract promising them; route: bad_spec.
- `medium` — The output contract did not enforce the approved English language; route: bad_spec.
- `low` — Markdown was forbidden in prose but the validator lacked an explicit markup check. Plain-text rendering limits security impact, but confusing output remains; route: patch, moot during bad-spec loopback.
- `medium` — A timed-out provider request was not required to be aborted, allowing hidden resource use and possible charges after fallback; route: bad_spec.
- `medium` — Returning buyer option IDs in model order contradicts the rule that the model cannot rank options; the server must preserve application-owned option order; route: patch.
- `medium` — Allowed clause orders were not enumerated, so semantically confusing permutations could pass; route: patch.
- `medium` — Six option codes existed beside a four-option response cap without deterministic per-state selection; route: patch.
- `medium` — Evidence kinds lacked an exact value-type, unit, nullability, and applicability registry, allowing incompatible renderers; route: patch.
- `medium` — Deterministic rendering referenced nonexistent "existing" formatters without defining locale, precision, unit, plural, and date behavior; route: patch.
- `maybe-false` — Carried: `gpt-5.6-sol` API availability remains unverified, and the configured-valid path depends on account access. An actual account model-list/API check is required to settle it; route: defer.
- `medium` — The plan did not require OpenAI strict structured-output/schema enforcement or a fallback when that capability is unavailable; route: patch.
- `medium` — Same-key retries were told both to reuse pending work and to receive fallback, creating conflicting behavior; route: patch.
- `medium` — Per-user controls left project-wide concurrency and spend unbounded; route: patch with a small global demo ceiling and kill switch.
- `medium` — Cache keys omitted model, contract, template-registry, and option-policy versions, allowing stale plans after policy changes; route: patch.
- `medium` — Weekly rows lacked a final stable row-ID tie-breaker after date and product, so versions and sequences could vary; route: patch.
- `medium` — Weekly supplier-risk aggregation did not identify which shortage deadline governs the singular risk statement; route: patch.
- `low` — With closed templates, the paid AI path adds only constrained presentation selection rather than new facts or decisions. This is intentional safety scope but needs explicit product-value rationale; route: patch.
- `low` — A single browser-safe fallback message did not provide cooldown behavior after rate limits. Authentication recovery is handled by the normal session, but a non-secret retry delay is useful; route: patch.
- `low` — A digest beginning with a digit conflicts with the safe-ID grammar only if it is used raw; the implementation should always prefix it with `result-`; route: patch.
- `medium` — `scope`, `alertState`, and `reasonCode` compatibility was not explicitly validated, allowing the wrong template family; route: patch.
- `false` — Weekly no-shortage results need no AI explanation plan because generation is defined for alerts and a zero-need weekly summary is not an alert. The plan should state this eligibility rule explicitly to prevent ambiguity; route: patch as a direct clarification.
- `medium` — Evidence type/unit incompatibility independently confirms the missing evidence registry; route: patch.
- `medium` — More than four applicable options independently confirms the missing state-and-risk option policy; route: patch.
- `medium` — A weekly positive shortage with unavailable lead time lacked an aggregate deadline-unavailable rule; route: patch.
- `medium` — Source evidence necessarily appears in several required clauses, but the duplicate rule did not state that cross-clause reuse is valid; route: patch.
- `medium` — The endpoint did not reject a stale displayed version before paying for a provider call; route: patch.
- `medium` — Same-key pending retries independently confirm the idempotency/active-call precedence conflict; route: patch.
- `medium` — Non-atomic rate, idempotency, and active-call checks could allow parallel calls through; route: patch.

## Design Notes

The fallback is the stable presentation layer, not a degraded calculation path. The model receives unique evidence IDs and returns only a closed explanation plan: ordered clause-template IDs plus bound fact, reason, risk, and option references. It never returns display-ready prose. The server validates uniqueness, required references, value bindings, result version, limits, and allowed template combinations, then renders English sentences from deterministic templates. This preserves an AI-selected explanation structure while preventing invented wording or swapped facts.

The trusted request covers both single-alert facts and weekly aggregation: selected week, ordered production rows, sequential inventory consumption, aggregated supplier-material recommendation, and earliest positive-shortage deadline. Every request and response carries the immutable calculation-result version; stale responses are discarded.

Generation availability requires a valid key and model. A later scope decision removed login from v1, so the future endpoint must use a signed anonymous demo session with same-origin/CSRF protection, accept one active request per alert/result version, deduplicate retries, apply a small per-session rate and usage cap, abort provider work on timeout or disconnect, and never expose whether failure came from credentials or account access.

## Verification

**Manual checks:**
- Trace every AI-input field back to a value defined by the calculation contract and confirm that no prompt asks the model to derive a number.
- Review the no-key, timeout, invalid-output, and blocked-calculation paths and confirm each ends in a usable deterministic explanation.
- Confirm that no real secret value appears in tracked files.
- Confirm the contract tests duplicate or unknown IDs, missing required references, swapped bindings, extra properties, blank output plans, unsupported template combinations, size/array limits, stale versions, repeated requests, timeout abortion, and missing supplier or multiple validation errors.

**Completed:**
- Traced the allowed request fields to the calculation contract's alert basis, ordering rules, validation messages, and delivery-history summary.
- Confirmed the documented no-key, timeout/provider, unavailable-model, malformed-output, unsupported-claim, and blocked-calculation paths retain the standard explanation.
- Searched the changed content for credential-like values; only the variable names, an empty key example, and the approved model name are present.
- Audited the re-derived contract against every review-loop finding, including semantic binding swaps, weekly row completeness, stale results, response limits, complete options, duplicate requests, and timeout cancellation.
