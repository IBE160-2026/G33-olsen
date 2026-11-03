# Deferred Work

- source_spec: `.docs/planning-artifacts/prds/prd-G33-Olsen-2026-10-18/prd.md#fr-25-record-goods-receipt-and-deviations`
  summary: Consider date-effective delivery and inventory calculations after the required first-version flows are stable.
  evidence: User explicitly deferred more complex delivery-date calculations. V1 posts only actual receipts dated on or before the scenario Planning Date and excludes expected future deliveries from current stock. A later extension could compare expected receipt dates with production dates and calculate availability over time; historical as-of balances and cross-week supply planning would need their own agreed rules and fixtures. This is a possible later feature, not a v1 commitment.

- source_spec: `.docs/planning-artifacts/prds/prd-G33-Olsen-2026-10-18/prd.md#fr-26-confirm-production-and-record-physical-consumption`
  summary: Consider controlled reversal of an individual production-consumption posting after required assignment flows are stable; not a first-version priority.
  evidence: User explicitly deferred this on 2026-11-02, relying on confirmation and operational checking routines for the initial demo. Any future reversal must preserve history and avoid overwriting unrelated later stock movements. Existing Reset demo data restores the complete fictional baseline and is a separate feature.

- source_spec: `.docs/planning-artifacts/prds/prd-G33-Olsen-2026-10-18/prd.md`
  summary: Consider batch tracking only after required assignment flows are complete and stable.
  evidence: User explicitly deferred batch tracking on 2026-11-02. Retaining historical receipt, deviation, and physical-consumption quantities is required independently of batch tracing. Production waste and actual-versus-planned consumption deviations are also deferred under the approved initial completion flow.

- source_spec: `.docs/implementation-artifacts/spec-ai-explanation-and-fallback.md`
  summary: Verify that `gpt-5.6-sol` is available to the project's OpenAI API account before application integration.
  evidence: Official OpenAI documentation confirms general API, Structured Outputs, and `high` reasoning-effort support; an account model-list or API check is still needed to confirm project-account access. The mandatory standard fallback keeps the core workflow usable.
