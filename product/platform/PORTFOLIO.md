# Portfolio: Platform

**Strategic Domain**: Shared Data Infrastructure and Enterprise Master Data
**Business Owner**: CTO / Head of Data Platform
**Status**: Active Investment
**Last Updated**: 2026-03-06

---

## Investment Thesis

The Platform portfolio holds shared infrastructure that all other portfolios depend on. It contains three foundational systems that together represent the complete enterprise data layer — the **Master Data** initiative:

- **DaVinci** is the enterprise Customer Golden Record — the single source of truth for customer identity, consent, and cross-product contact compliance.
- **Genesis** is the enterprise Contract & Collateral Golden Record — the single source of truth for contract and collateral data, covering both new originations (event-driven from Onigiri) and historical data migrated from legacy source systems.
- **Core Banking** is the authoritative financial ledger — the single source of truth for loan account state, balance, payment allocation, interest accrual, and DPD status.

None of these products has a direct customer-facing value proposition. Their value is measured by the cost, risk, and inconsistency they eliminate for other products:
- Without DaVinci: each product maintains its own customer record → PDPA consent cannot be enforced → Thai Debt Collection Act contact limits cannot be tracked across products.
- Without Genesis: contract and collateral data remains fragmented in the origination system → downstream services must query Onigiri directly → portfolio-level reporting is unreliable → migrated historical data has no authoritative home.
- Without Core Banking: each product implements its own balance calculation and DPD logic → inconsistent NPL figures → regulatory exposure → Collections and Loan Servicing operate on stale or contradictory data.
- With all three: one canonical customer identity, one authoritative contract and collateral record, one authoritative financial ledger. Every downstream consumer queries Platform instead of duplicating this logic.

**Why a separate portfolio?** Platform products have fundamentally different investment and governance patterns: they are owned by engineering/data teams (not product teams), they have no direct revenue attribution, and their value is measured as shared infrastructure cost avoidance rather than feature delivery velocity. Conflating them with operational or domain portfolios would obscure this distinction.

**Future Platform candidates**: A shared notification service (SMS/push/email), a shared reporting data warehouse, or a workflow engine (if Sensei's task engine becomes generic enough) may be added to this portfolio as the company scales.

---

## Constituent Products

| Product | Codename | Status | PRODUCT.md | Description |
|---------|----------|--------|------------|-------------|
| Customer & Product Master Data | DaVinci (ダヴィンチ) | 📝 Draft | [PRODUCT](davinci/PRODUCT.md) | Enterprise Customer Golden Record. Consent-based visibility (PDPA), event-driven sync, data consolidation engine, resolution workflow, collection contact compliance. |
| Contract & Collateral Master Data | Genesis | 📝 Draft | [PRODUCT](genesis/PRODUCT.md) | Enterprise Contract & Collateral Golden Record. Event-driven ingestion via Worker Adapter (RabbitMQ → S3 → RDS), contract master data, collateral master data, batch migration of legacy data, migration audit trail (MongoDB). Publishes completion events to downstream consumers via ods.trigger. |
| Core Banking | TBD | 📝 Draft | [PRODUCT](core-banking/PRODUCT.md) | Authoritative financial ledger. Loan account lifecycle, payment hierarchy, interest & fee calculation, DPD engine. Single source of truth for all loan account state. |

---

## Portfolio-Level OKRs

| Objective | Key Result | Target |
|-----------|-----------|--------|
| Achieve complete Customer Golden Record coverage | % of active customers with a DaVinci record | 100% by launch |
| Enable real-time data currency | DaVinci event processing latency (p95) | < 30 seconds |
| Enforce PDPA compliance | % of cross-entity data access attempts blocked without valid consent | 100% |
| Resolve data conflicts within SLA | T1 resolutions completed on same day | > 95% |
| Resolve data conflicts within SLA | T3 resolutions completed within 5 business days | > 90% |
| Eliminate contact frequency violations | % of collection contacts that exceed daily limit (per Debt Collection Act) | 0% |
| Achieve complete Contract Golden Record coverage | % of active contracts with a Genesis RDS record | 100% at go-live |
| Achieve complete Collateral Golden Record coverage | % of active collaterals with a Genesis RDS record | 100% at go-live |
| Complete legacy data migration | % of historical contracts and collaterals migrated from source systems to Genesis RDS | 100% before go-live |
| Ensure migration data accuracy | % of migrated records with zero payloadDifference discrepancies | > 99% |

---

## Cross-Product Dependencies

```
Platform Portfolio → All Portfolios

  Core Banking (owned)
  Core Banking ← Onigiri       : CreateFacility command, DisburseLoan command → via API
  Core Banking ← Payment GW    : PaymentReceived event → via event
  Core Banking ← Scheduler     : DailyAccrualTrigger → via scheduled job
  Core Banking → DaVinci        : LoanDisbursed, LoanPaymentReceived, LoanStatusChanged, LoanDPDChanged, LoanClosed events
  Core Banking → Onigiri        : CreateFacilityResult → via API response

  DaVinci (owned)
  DaVinci ← Core Banking  : LoanDisbursed, LoanPaymentReceived, LoanStatusChanged, LoanClosed, LoanDPDChanged events
  DaVinci ← Policy Admin  : PolicyIssued, PremiumPaid, PolicyLapsed, PolicyCancelled, PolicyRenewed events
  DaVinci ← Matcha        : CustomerVerified, CustomerKYCExpired events
  DaVinci ← Onigiri       : ApplicationCreated, ApplicationApproved, CustomerProfileUpdated events
  DaVinci ← Sensei        : ContactRecorded events

  DaVinci → Sensei         : ContactLimitReached, ContactLimitApproaching, ContactWindowClosed events
  DaVinci → Sensei         : customer.resolution_required events (triggers Admin tasks)
  DaVinci → Matcha         : T3 verification task requests (identity evidence)
  DaVinci → Branch Dashboards / Collection Systems / Risk Analytics : REST Query API
  DaVinci → Onigiri        : Customer identity lookup on application creation
  DaVinci → Collections    : LoanDPDChanged, ContactRecorded events (Collections as downstream consumer)

  Genesis (owned)
  Genesis ← Onigiri        : domain events (ApplicationApproved, ContractCreated, CollateralRegistered, etc.)
                             → via RabbitMQ Event Trigger A (upstream vhosts)
                             → consumed by Worker Adapter (Genesis boundary begins here)
  Worker Adapter → S3      : full enriched payload → s3://turbo-master-data/logs_event/
  Worker Adapter → ods     : Internal Event → ods.internal exchange (vhost: ods)
  Genesis → RDS            : upsert Contract + Collateral Golden Records (PostgreSQL)
  Genesis → MongoDB        : migration audit trail only (ops_*_event collections, internal tool)
  Genesis → Downstream     : completion events → ods.trigger (vhost: ods)
                             consumed by Sync Workers (ASTRA, CRM, etc.) owned by downstream teams
```

---

## Strategic Roadmap

| Horizon | Initiative | Owner |
|---------|-----------|-------|
| **Now** | DaVinci — Data Consolidation Engine (field authority, no-data-loss, admin override, self-check) | Platform PO |
| **Now** | DaVinci — Event-Driven Synchronization (Core Banking + Onigiri events) | Platform PO |
| **Now** | DaVinci — Golden Record + Downstream Query API | Platform PO |
| **Now** | Core Banking — PRODUCT.md defined; ARCHITECTURE.md and capability specs to follow | Platform PO |
| **Now** | Genesis — PRODUCT.md defined (Master Data initiative) | Master Data BA |
| **Now** | Genesis — Ingestion & Enrichment Pipeline (Worker Adapter, F6): RabbitMQ → S3 → ods.internal | Genesis Engineering |
| **Now** | Genesis — Contract Master Data: ods.internal → RDS upsert (genesis-contract-api) | Genesis Engineering |
| **Now** | Genesis — Collateral Master Data: ods.internal → RDS upsert (genesis-collateral-api) | Genesis Engineering |
| **Now** | Genesis — Batch Migration & Audit: legacy data migration into RDS + MongoDB audit trail | Genesis Engineering |
| **Next** | DaVinci — PDPA Consent Engine (Consent Registry, visibility filtering API) | Platform PO |
| **Next** | DaVinci — Collection Contact Compliance (Unified Contact Log, Frequency Check API) | Platform PO |
| **Next** | DaVinci — Data Resolution Workflow (ResolutionRequest lifecycle, T1/T2/T3, Sensei integration) | Platform PO |
| **Next** | Core Banking — ARCHITECTURE.md: API contracts, data model, event schema registry | Core Banking PO |
| **Next** | Core Banking — Assign codename | Platform PO |
| **Next** | Genesis — Capability specs (CAPABILITY.md) for all 4 capabilities | Master Data BA |
| **Next** | Genesis — Confirm upstream event contracts with Onigiri team (event names + payload schema) | Master Data BA + Onigiri PO |
| **Next** | Genesis — Confirm ods.trigger event schema with downstream teams (ASTRA, CRM) | Master Data BA |
| **Later** | DaVinci — Policy Admin event domain (insurance product sync) | Platform PO |
| **Later** | Core Banking — Feature decomposition under each capability | Core Banking PO |
| **Later** | Genesis — Feature decomposition under each capability | Genesis Engineering |

---

## Key Risks and Constraints

| Risk | Severity | Mitigation |
|------|----------|-----------|
| DaVinci is a blocking dependency for all other portfolios | Critical | Prioritize Golden Record + Query API as the first deliverable. All other capabilities are additive. |
| Core Banking is a blocking dependency for Collections and Loan Servicing | Critical | Core Banking ARCHITECTURE.md and event schema must be defined before Collections or Loan Servicing integration work begins. |
| Architecture gap: PDPA Consent Engine not yet in ATLAS | High | Identified in `davinci/artifacts/research/atlas_vs_architecture_gap_analysis.md`. Must be specced before launch. |
| Architecture gap: Contact Compliance API contract not specified in ARCHITECTURE.md | High | Sensei depends on this for event subscription. Must be resolved before Sensei Task Engine is built. |
| Core Banking codename not assigned | Medium | A codename is needed before cross-team communication becomes confusing. Assign at next Portfolio review. |
| Collateral Master Data gap — RESOLVED | Closed | Previously flagged as a DaVinci gap. Collateral Master Data is now owned by **Genesis** (Contract & Collateral Master Data product). See [genesis/PRODUCT.md](genesis/PRODUCT.md). |
| Genesis upstream event contracts not yet confirmed with Onigiri | High | The exact event names and payload schemas that Onigiri emits to RabbitMQ (Event Trigger A) must be confirmed before the Worker Adapter can be built. Requires alignment with the Onigiri team. |
| Genesis downstream event schema (ods.trigger) not yet agreed with consumers | High | ASTRA Worker, CRM Worker, and other downstream Sync Workers need the ods.trigger event schema before they can build their projection logic. Must be defined and published by Genesis team. |
| Genesis Worker Adapter source API availability | Medium | The Worker Adapter enriches payloads by calling source APIs. If those APIs are slow or unavailable, enrichment fails. Must define a retry + dead-letter strategy and a Enrichment Failure Rate SLA. |
| `davinci_customer_id` vs. National ID as canonical key — decision unresolved | Medium | The ARCHITECTURE.md does not specify which key is the primary lookup. Must be decided before integration contracts are written. |
| Core Banking capability ownership questions open | Medium | Payment hierarchy config ownership (Core Banking vs. Onigiri Campaign Config), write-off approval flow, and interest calculation ownership are unresolved. Document decisions in ARCHITECTURE.md before feature work begins. |
| Event processing eventual consistency | Low | DaVinci is a read model — eventual consistency is acceptable for all use cases except real-time contact compliance. Contact compliance events must be near-real-time (< 30s p95). |
