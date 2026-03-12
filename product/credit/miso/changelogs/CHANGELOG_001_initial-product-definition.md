# CHANGELOG 001 — Miso: Initial Product Definition

**Date**: 2026-03-05
**Layer Affected**: Product
**Author**: Claude Code (assisted)
**Status**: Final — append only

---

## What Changed

Defined **Miso (Credit Scoring Service)** as a new product within the Credit Portfolio.

### Documents Created

| Document | Path |
|----------|------|
| PRODUCT.md | `product/credit/miso/PRODUCT.md` |
| Model Registry CAPABILITY.md | `product/credit/miso/capabilities/model-registry/CAPABILITY.md` |
| Score Router CAPABILITY.md | `product/credit/miso/capabilities/score-router/CAPABILITY.md` |
| Score Evaluator CAPABILITY.md | `product/credit/miso/capabilities/score-evaluator/CAPABILITY.md` |
| Score Mapper CAPABILITY.md | `product/credit/miso/capabilities/score-mapper/CAPABILITY.md` |
| Score Contract CAPABILITY.md | `product/credit/miso/capabilities/score-contract/CAPABILITY.md` |
| Inference Audit Log CAPABILITY.md | `product/credit/miso/capabilities/inference-audit-log/CAPABILITY.md` |

### Documents Updated

| Document | Path | Change |
|----------|------|--------|
| PORTFOLIO_CATALOG.md | `product/PORTFOLIO_CATALOG.md` | Added Miso row under Credit Portfolio |
| Credit PORTFOLIO.md | `product/credit/PORTFOLIO.md` | Added Miso to constituent products, cross-product dependencies, and roadmap |
| Onigiri PRODUCT.md | `product/credit/onigiri/PRODUCT.md` | Added Miso to product boundary (IS NOT responsible for credit scoring) and integration map |

---

## Rationale

The originating request was to add a routing function that sends each loan application to credit scoring based on its pre-configured campaign. Analysis revealed this requires more than a feature within Onigiri's Risk Assessment Engine:

- Scoring models evolve independently of the origination workflow — separate lifecycle → separate product
- Traffic splitting and A/B testing between model versions require runtime configuration management not appropriate inside Onigiri
- Raw model output is confidential and must never appear on the application record — confidentiality enforcement requires an explicit product boundary
- The standardized score contract (stable field names for JMESPath and eligibility criteria in Onigiri) must be pre-configured before model activation — requires a dedicated contract management capability
- Regulatory auditability requires an immutable, append-only record of every inference event — a cross-cutting concern warranting its own capability

---

## Decision Log

| # | Decision | Rationale |
|---|----------|-----------|
| D1 | New product (not a capability of Onigiri) | Distinct lifecycle, confidentiality boundary, and infrastructure responsibility justify product-level separation |
| D2 | Codename: Miso | Japanese food naming convention; Miso is a foundational, enriching ingredient — fitting for a shared scoring service |
| D3 | Portfolio: Credit (not Platform) | Miso is initially scoped to credit origination; if reuse across portfolios emerges, it can be migrated to Platform portfolio |
| D4 | Miso owns model execution | A pure proxy layer cannot enforce confidentiality — owning inference is required to prevent raw score leakage |
| D5 | Inference Audit Log added | Regulatory requirement for auditability and tamper-evidence; immutability enforced at both application and infrastructure levels |
| D6 | raw_output_hash (not raw score) in audit log | Stores tamper-evidence without storing confidential data; SHA-256 hash enables integrity verification without raw score exposure |

---

## Links to Updated Documents

- [Miso PRODUCT.md](../PRODUCT.md)
- [Credit PORTFOLIO.md](../../PORTFOLIO.md)
- [PORTFOLIO_CATALOG.md](../../../PORTFOLIO_CATALOG.md)
- [Onigiri PRODUCT.md](../../onigiri/PRODUCT.md)
