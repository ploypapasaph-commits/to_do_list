# Changelog 001: Initial Product Definition

> **Date:** 2026-03-05
> **Layer Affected:** Portfolio, Product, Capability
> **Author:** AI Assistant

---

## What Changed

### Portfolio Layer
- Created **Insurance** portfolio with `PORTFOLIO.md`
- Added Insurance portfolio and OnePiece product to `PORTFOLIO_CATALOG.md`

### Product Layer
- Created **OnePiece (ワンピース)** -- Insurance Distribution Platform
- Defined product boundary (IS / IS NOT)
- Documented insurance product matrix (6 products across 2 channels)
- Documented payment matrix (2 terms x 3 channels with channel restrictions)
- Documented policy issuance methods (file transfer, REST API, manual)
- Defined integration map: DaVinci (customer data), Matcha (doc verification for installment), insurer partners
- Created capability registry with 6 capabilities

### Capability Layer
- Created 6 capability stubs:
  1. **Product Catalog & Channel Configuration** -- master config for products, channels, payment, issuance
  2. **Quotation & Comparison** -- multi-insurer package retrieval and comparison
  3. **Application Management** -- sales application lifecycle, channel-specific flows
  4. **Payment Processing** -- payment term/channel routing, installment loan approval via Matcha
  5. **Policy Issuance** -- insurer integration dispatcher (file transfer, API, manual)
  6. **Post-Sale Management** -- endorsement/cancellation brokering with insurers

---

## Rationale

- **Single product, not micro-products:** Payment, issuance, and post-sale are capabilities within one product because they share a single lifecycle and have no independent consumers. If a shared payment service is needed later, it can be extracted.
- **Post-Sale as separate capability:** Endorsement and cancellation have a distinct flow (broker relay model) that differs from the sales flow, warranting a dedicated capability.
- **Insurance portfolio vs. adding to existing:** Insurance is a distinct strategic domain with its own business model (brokerage vs. lending), warranting a separate portfolio.

---

## Decisions

| Decision | Alternatives Considered | Why Chosen |
|----------|------------------------|------------|
| Single product (OnePiece) | Micro-products (separate Payment, Issuance products) | No independent lifecycle or consumer for payment/issuance; avoids integration tax |
| New Insurance portfolio | Add to existing portfolio (e.g., Platform) | Insurance brokerage is a distinct business domain with unique value proposition |
| 6 capabilities | Fewer (merge quotation + application) or more (split per channel) | Each capability has a distinct business function and potential owner |

---

## Links

- [PORTFOLIO_CATALOG.md](../../PORTFOLIO_CATALOG.md)
- [Insurance PORTFOLIO.md](../PORTFOLIO.md)
- [OnePiece PRODUCT.md](../PRODUCT.md)
