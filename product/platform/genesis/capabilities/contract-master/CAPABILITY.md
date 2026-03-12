# Capability: Contract Master Data

**Parent Product**: Genesis → [PRODUCT](../../PRODUCT.md)
**Product Owner**: TBD
**Status**: 📝 Draft
**Last Updated**: 2026-03-06

---

## Business Function

The authoritative store for all contract records — both new contracts originating from Onigiri workflows and historical contracts migrated from legacy source systems. This capability consumes Internal Events from the `ods.internal` exchange, reads the enriched payload from S3, and upserts contract records into the Genesis RDS (PostgreSQL) database.

The Contract Master Data is the single source of truth that downstream systems (ASTRA, CRM, collections, risk analytics) rely on for contract information. They do not query Onigiri directly.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Internal Event Consumer (Contracts) | Concept | Subscribes to `ods.internal` exchange (vhost: `ods`). Filters for contract-relevant Internal Events. |
| S3 Payload Reader | Concept | Reads the enriched payload from the S3 path provided in the Internal Event. |
| Contract Upsert | Concept | Upserts the contract record into RDS (PostgreSQL). New record on first insert; update existing record on subsequent events for the same contract. |
| Completion Event Publisher | Concept | Publishes a completion event to `ods.trigger` (Event Trigger B) after successful RDS upsert, enabling downstream Sync Workers to update their display databases. |

---

## Business Rules

| Rule | Description |
|------|-------------|
| RDS is the source of truth | No downstream system reads contract data directly from Onigiri. All queries go to Genesis RDS. |
| Upsert, not insert-only | If a contract already exists in RDS (matched by contract key), the record is updated. No duplicates. |
| New + migrated data coexist | The RDS schema must support both new contracts (event-driven) and migrated historical contracts (from Batch Migration). There is no distinction in the stored record — both are Golden Records. |
| Completion event after commit | The `ods.trigger` completion event must only be published after the RDS transaction is confirmed committed. |

---

## Open Questions

- What is the canonical contract key used for upsert matching? (Contract ID, Loan ID, or surrogate key?)
- What is the full RDS schema for a contract record? (To be defined in ARCHITECTURE.md)
- Which fields in the contract record are mandatory vs. optional?
- What happens if the S3 payload is missing or corrupted when the Internal Event arrives?
