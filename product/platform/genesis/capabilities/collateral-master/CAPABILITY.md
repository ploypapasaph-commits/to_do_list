# Capability: Collateral Master Data

**Parent Product**: Genesis → [PRODUCT](../../PRODUCT.md)
**Product Owner**: TBD
**Status**: 📝 Draft
**Last Updated**: 2026-03-06

---

## Business Function

The authoritative store for all collateral records — both new collaterals registered during Onigiri loan origination and historical collaterals migrated from legacy source systems. This capability consumes Internal Events from the `ods.internal` exchange, reads the enriched payload from S3, and upserts collateral records into the Genesis RDS (PostgreSQL) database.

The Collateral Master Data is the single source of truth for collateral ownership, valuation, and linkage to loan accounts. Downstream systems do not query Onigiri or other origination systems for collateral data.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Internal Event Consumer (Collaterals) | Concept | Subscribes to `ods.internal` exchange (vhost: `ods`). Filters for collateral-relevant Internal Events. |
| S3 Payload Reader | Concept | Reads the enriched payload from the S3 path provided in the Internal Event. |
| Collateral Upsert | Concept | Upserts the collateral record into RDS (PostgreSQL). New record on first insert; update on subsequent events for the same collateral. |
| Completion Event Publisher | Concept | Publishes a completion event to `ods.trigger` (Event Trigger B) after successful RDS upsert, enabling downstream Sync Workers to update their display databases. |

---

## Business Rules

| Rule | Description |
|------|-------------|
| RDS is the source of truth | No downstream system reads collateral data directly from Onigiri or any legacy system. All queries go to Genesis RDS. |
| Upsert, not insert-only | If a collateral already exists in RDS (matched by collateral key), the record is updated. No duplicates. |
| New + migrated data coexist | The RDS schema must support both new collaterals (event-driven) and migrated historical collaterals (from Batch Migration). No distinction in the stored record — both are Golden Records. |
| Collateral may link to multiple loans | A single collateral asset (e.g. a land title) may secure more than one loan. The schema must support this many-to-many relationship. |
| Completion event after commit | The `ods.trigger` completion event must only be published after the RDS transaction is confirmed committed. |

---

## Open Questions

- What is the canonical collateral key used for upsert matching? (Collateral ID, asset registration number, or surrogate key?)
- What is the full RDS schema for a collateral record? (To be defined in ARCHITECTURE.md)
- How is the many-to-many relationship between collaterals and loans modelled in RDS?
- What collateral types are in scope? (Land title, vehicle, etc.)
- What happens if the S3 payload is missing or corrupted when the Internal Event arrives?
