# Portfolio Catalog

> Master registry of all portfolios and their constituent products.
> This is the single entry point for any cross-product or cross-portfolio work.
> **Read this first before any product-level session.**
>
> Last Updated: 2026-03-09

---

## Portfolio Registry

| Portfolio | Product | Codename | Status | Portfolio Def | Summary |
|-----------|---------|----------|--------|---------------|---------|
| **Credit** | Loan Origination System | Onigiri (おにぎり) | 📝 Draft | [PORTFOLIO](credit/PORTFOLIO.md) | All-in-one loan origination platform. Smart Form intake → fixed-topology underwriting workflow → campaign-driven risk assessment → disbursement. Integrates with Matcha, Wasabi, DaVinci, Sensei, Core Banking, NCB. |
| **Credit** | Credit Scoring Service | Miso (味噌) | 📝 Draft | [PORTFOLIO](credit/PORTFOLIO.md) | Standalone credit scoring service. Campaign-to-model routing, A/B traffic splitting, model inference execution, raw→standardized score mapping, confidentiality enforcement, immutable inference audit log. |
| **Credit** | Asset Valuation Service | Dashi (だし) | 📝 Draft | [PORTFOLIO](credit/PORTFOLIO.md) | Multi-source vehicle price ingestion, configurable data cleansing pipeline, cross-source vehicle identity resolution, price consolidation engine, risk team rate management dashboard, canonical market rate API. Pure market data service — no borrower context. |
| **Operations** | Document Verification Service | Matcha (抹茶) | ✅ Active | [PORTFOLIO](operations/PORTFOLIO.md) | Universal domain-agnostic document verification engine. 4-state task lifecycle, flexible rule configuration, SHA-256 change detection, async car check integration, re-flow deduplication, AI-first routing via Wasabi. |
| **Operations** | AI Document Verification | Wasabi (わさび) | 📝 Draft | [PORTFOLIO](operations/PORTFOLIO.md) | Stateless LLM-based document verification. 4-stage pipeline: quality → type classification → instruction verification → report assembly. Operates outside Matcha; returns results to Onigiri for early warnings and Matcha for routing. |
| **Operations** | Branch Operations Orchestration | Sensei (先生) | 📝 Draft | [PORTFOLIO](operations/PORTFOLIO.md) | Centralized branch worklist and task orchestration. Aggregates work from Sensei playbooks, external service requests (Onigiri, Matcha), and supervisor tasks into a single prioritized queue. Contact compliance via DaVinci events. |
| **Platform** | Customer & Product Master Data | DaVinci (ダヴィンチ) | 📝 Draft | [PORTFOLIO](platform/PORTFOLIO.md) | Enterprise Golden Record. Consent-based visibility (PDPA), event-driven sync, customer data change management, collection contact compliance, data consolidation engine (field-level authority, no-data-loss), data resolution workflow (3-tier). |
| **Platform** | Contract & Collateral Master Data | Genesis | 📝 Draft | [PORTFOLIO](platform/PORTFOLIO.md) | Contract and Collateral Golden Record. Event-driven ingestion via Worker Adapter (RabbitMQ → S3 → RDS), contract master data, collateral master data, batch migration of legacy data, migration audit trail (MongoDB). Publishes completion events to downstream consumers via ods.trigger. |
| **Platform** | Core Banking | TBD | 📝 Draft | [PORTFOLIO](platform/PORTFOLIO.md) | Authoritative financial ledger for all loan accounts. Loan account lifecycle management, configurable payment hierarchy engine, interest & fee calculation, DPD engine. Single source of truth for account balance and delinquency status across all credit products. |
| **Insurance** | Insurance Distribution Platform | OnePiece (ワンピース) | 📝 Draft | [PORTFOLIO](insurance/PORTFOLIO.md) | Multi-channel insurance broker platform. Aggregates quotations from multiple insurers, manages sales applications across branch and online channels, processes payments (cash/QR/2C2P), issues policies via file transfer/API/manual, and brokers post-sale endorsements and cancellations. |

---

## Directory Structure

```
product/
├── PORTFOLIO_CATALOG.md          ← You are here
│
├── credit/
│   ├── PORTFOLIO.md
│   ├── onigiri/
│   │   ├── PRODUCT.md
│   │   ├── ATLAS.md              ← Detailed source of truth (migration period)
│   │   └── capabilities/
│   ├── miso/
│   │   ├── PRODUCT.md
│   │   └── capabilities/
│   └── turbo-rate/
│       ├── PRODUCT.md
│       └── capabilities/
│
├── operations/
│   ├── PORTFOLIO.md
│   ├── matcha/
│   │   ├── PRODUCT.md
│   │   ├── ATLAS.md
│   │   ├── ARCHITECTURE.md
│   │   └── capabilities/
│   ├── wasabi/
│   │   ├── PRODUCT.md
│   │   ├── ATLAS.md
│   │   └── capabilities/
│   └── sensei/
│       ├── PRODUCT.md
│       ├── ATLAS.md
│       └── capabilities/
│
├── insurance/
│   ├── PORTFOLIO.md
│   └── onepiece/
│       ├── PRODUCT.md
│       └── capabilities/
│
└── platform/
    ├── PORTFOLIO.md
    ├── davinci/
    │   ├── PRODUCT.md
    │   ├── ATLAS.md
    │   ├── Architecture.md
    │   └── capabilities/
    ├── genesis/
    │   ├── PRODUCT.md
    │   └── capabilities/
    └── core-banking/
        ├── PRODUCT.md
        └── capabilities/
```

---

## Migration Status

> Products are being migrated from flat ATLAS.md structure to the 4-layer Atomic Architecture.
> During migration, `ATLAS.md` in each product folder remains the detailed source of truth.
> `PRODUCT.md` and `CAPABILITY.md` files are extractions. `FEATURE_<name>.md` files are pending engineering owner assignment.

| Layer | Status |
|-------|--------|
| Portfolio (PORTFOLIO_CATALOG + PORTFOLIO.md) | ✅ Complete |
| Product (PRODUCT.md) | ✅ Complete |
| Capability (CAPABILITY.md) | ✅ Complete (stubs) |
| Feature (FEATURE_*.md) | ⏳ Pending — requires engineering owner assignment |
