# CHANGELOG 001 — Dashi: Initial Product Definition

**Date**: 2026-03-09
**Layer Affected**: Product + Capability
**Author**: Claude Code (assisted)
**Status**: Final — append only

---

## What Changed

Defined **Dashi (Asset Valuation Service)** as a new product within the Credit Portfolio.

### Documents Created

| Document | Path |
|----------|------|
| PRODUCT.md | `product/credit/turbo-rate/PRODUCT.md` |
| Raw Source Ingestion CAPABILITY.md | `product/credit/turbo-rate/capabilities/raw-source-ingestion/CAPABILITY.md` |
| Source Cleansing Configuration CAPABILITY.md | `product/credit/turbo-rate/capabilities/source-cleansing-configuration/CAPABILITY.md` |
| Vehicle Identity Resolution CAPABILITY.md | `product/credit/turbo-rate/capabilities/vehicle-identity-resolution/CAPABILITY.md` |
| Price Consolidation Engine CAPABILITY.md | `product/credit/turbo-rate/capabilities/price-consolidation-engine/CAPABILITY.md` |
| Rate Management Dashboard CAPABILITY.md | `product/credit/turbo-rate/capabilities/rate-management-dashboard/CAPABILITY.md` |
| Rate Publishing API CAPABILITY.md | `product/credit/turbo-rate/capabilities/rate-publishing-api/CAPABILITY.md` |

### Documents Updated

| Document | Path | Change |
|----------|------|--------|
| PORTFOLIO_CATALOG.md | `product/PORTFOLIO_CATALOG.md` | Added Dashi (TurboRate) row under Credit Portfolio |
| Credit PORTFOLIO.md | `product/credit/PORTFOLIO.md` | Added Dashi to constituent products, cross-product dependencies (Onigiri ↔ Dashi), and strategic roadmap |
| Onigiri PRODUCT.md | `product/credit/onigiri/PRODUCT.md` | Added Dashi to boundary (IS NOT: vehicle market rate management), RECEIVES (canonical rate), SENDS (vehicle identifier), and integration map diagram |

---

## Rationale

The originating request was to define a configurable asset valuation system for vehicle-secured lending in the Credit portfolio. Analysis confirmed this requires a standalone product, not a capability within Onigiri:

- Vehicle market rate management has a distinct lifecycle owner (Risk Management team) separate from the loan origination owner (Credit PO). Different lifecycles → separate product.
- Source data ingestion, cleansing, and consolidation are batch pipeline concerns with infrastructure requirements (immutable raw storage, config versioning, retroactive reprocessing) that are architecturally incompatible with Onigiri's event-driven underwriting state machine.
- The Risk Management team needs no-code tools to manage price sources and react to market events independently of loan campaign operations — a different operational surface from Onigiri's campaign configuration.
- Behavioral pricing (customer-character-based rate adjustments) was explicitly excluded from Dashi scope: Onigiri already has a configurable JMESPath rule engine; extending it for behavioral pricing avoids cross-product coupling. Dashi produces one canonical market rate per vehicle; Onigiri applies all borrower-specific logic.

---

## Decision Log

| # | Decision | Rationale |
|---|----------|-----------|
| D1 | New product (not a capability of Onigiri) | Distinct lifecycle owner, distinct data domain (vehicle market data vs. loan application), distinct infrastructure (batch pipeline vs. underwriting state machine) |
| D2 | Codename: Dashi (user-selected) | Japanese food naming convention; Dashi is the fundamental clear broth extracted from multiple raw sources — literal metaphor for multi-source extraction and distillation |
| D3 | Portfolio: Credit (not Platform) | Primary consumer is Onigiri; business owner is Credit Risk. Rate logic is lending-specific. If consumed cross-portfolio, re-evaluate placement at that time. |
| D4 | TurboRate Vehicle ID (not VIN/license plate) | VINs not universally available in Thai used-car market; plates change on resale. Brand + model + year + grade is the stable identity key. |
| D5 | Immutable raw storage before transformation | Enables retroactive reprocessing and forensic audit: "what was the rate derived from, from which source, on which date?" |
| D6 | Market adjustments are a correction layer (not overwrites) | Adjustments with mandatory expiry sit above the pipeline-computed rate; rates revert automatically after expiry. Prevents permanent manual override of data-driven values. |
| D7 | No behavioral pricing in Dashi | Behavioral adjustments belong in Onigiri's Loan Campaign Configuration. Mixing market data logic with borrower-specific logic would couple Dashi to lending campaign rules. |
| D8 | Rate Management Dashboard handles market corrections only | Risk team reacts to macro market events (not individual customer profiles). Per-customer pricing is Onigiri's domain. |
| D9 | Dashi does not integrate with DaVinci | Dashi manages vehicle model-level market rates, not customer-vehicle ownership records. No customer data accepted or stored. |

---

## Links to Updated Documents

- [Dashi PRODUCT.md](../PRODUCT.md)
- [Credit PORTFOLIO.md](../../PORTFOLIO.md)
- [PORTFOLIO_CATALOG.md](../../../PORTFOLIO_CATALOG.md)
- [Onigiri PRODUCT.md](../../onigiri/PRODUCT.md)
