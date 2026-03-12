# Changelog 003: Payment Model Revision

> **Date:** 2026-03-05
> **Layer Affected:** Product, Capability
> **Author:** AI Assistant

---

## What Changed

### Product Layer (PRODUCT.md)
- Replaced simplified Payment Matrix with detailed Payment Model section
- Documented **current state**: Branch (Cash, Bill Payment QR), Online (2C2P), Online is full-payment only
- Documented **planned state** with 6 granular payment channels:
  - Cash, Bill Payment QR (branch-only)
  - 2C2P QR, 2C2P DPAY (both channels)
  - 2C2P Credit Card, 2C2P IRR (full payment only)
- Added payment channel code registry (CASH, BQR, 2C2P-QR, 2C2P-DPAY, 2C2P-CC, 2C2P-IRR)

### Capability Layer
- **Payment Processing**: Expanded feature inventory from 7 to 10 features (split QR into Bill Payment QR, added individual 2C2P sub-channel features). Split business rules into current state and planned state sections. Updated payment flow diagram.
- **Product Catalog**: Replaced outdated PC-002/PC-003 rules. Added PC-003 (online full-payment only), PC-003a (2C2P CC/IRR full-payment only).

---

## Key Corrections from Previous State
- 2C2P is NOT online-only -- it will be available in branch (planned)
- "QR" was ambiguous -- now split into "Bill Payment QR" (branch physical) and "2C2P QR" (digital)
- Online does NOT offer installment -- full payment only
- 2C2P Credit Card and 2C2P IRR are full-payment-term only (not installment)

---

## Links

- [OnePiece PRODUCT.md](../PRODUCT.md)
- [Payment Processing CAPABILITY.md](../capabilities/payment-processing/CAPABILITY.md)
- [Product Catalog CAPABILITY.md](../capabilities/product-catalog/CAPABILITY.md)
