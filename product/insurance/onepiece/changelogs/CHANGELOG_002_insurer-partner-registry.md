# Changelog 002: Insurer Partner Registry & Coverage Year Dimension

> **Date:** 2026-03-05
> **Layer Affected:** Portfolio, Product, Capability
> **Author:** AI Assistant

---

## What Changed

### Portfolio Layer
- Updated OKR: "Multi-insurer coverage" key result changed from "at least 2" to "at least 5 insurer partners at launch" (5 are already active)

### Product Layer
- Added **Insurer Partner Registry** with 5 active partners: วิริยะ (VIR), ชับบ์สามัคคี (CHUBB), แอกซ่า (AXA), เมืองไทย (MTI), อลิอันซ์ อยุธยา (AZA)
- Added **Insurer-Product Matrix** with 17 insurer-product combinations (14 REST API, 3 Manual Input)
- Added **coverage year** as a configuration dimension (1 year vs. > 1 year)
- Documented that > 1 year coverage products are currently branch-only
- Documented that motorcycle products have no insurer partner yet
- Documented that file transfer issuance is not in use with current partners
- Updated integration map with actual insurer names
- Added future plan note: all products to be available in all channels

### Capability Layer (Product Catalog)
- Added coverage year restriction rules (PC-004, PC-006)
- Updated issuance method rule to include coverage year as a dimension (PC-005)

---

## Key Insights

- Coverage year is a first-class dimension alongside insurer, product, and channel
- The online channel currently only supports REST API issuance with 1-year coverage
- Manual input issuance is exclusively วิริยะ (VIR) and exclusively branch
- Channel availability is effectively determined by: product type + coverage year + issuance method

---

## Links

- [OnePiece PRODUCT.md](../PRODUCT.md)
- [Product Catalog CAPABILITY.md](../capabilities/product-catalog/CAPABILITY.md)
- [Insurance PORTFOLIO.md](../../PORTFOLIO.md)
