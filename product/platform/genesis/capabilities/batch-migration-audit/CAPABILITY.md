# Capability: Batch Migration & Audit

**Parent Product**: Genesis → [PRODUCT](../../PRODUCT.md)
**Product Owner**: TBD
**Status**: 📝 Draft
**Last Updated**: 2026-03-06

---

## Business Function

Responsible for migrating existing contract and collateral data from legacy source systems into the Genesis RDS (PostgreSQL) Golden Record store. This is a one-time migration effort (Batch 1), but the infrastructure is designed to support historical replay and ongoing audit.

The capability also maintains the **Migration Audit Trail** — a MongoDB store (`ops_*_event` collections) where the full raw payload and the `payloadDifference` (the diff between source data and the RDS schema) are recorded. This is an **internal tool for the Genesis team only**, used to verify that data mapping is accurate before records are committed to the master database.

Once migration is complete and verified, this capability transitions from active migration mode to a maintenance/replay mode.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Batch 1 Analysis Job | Concept | Reads existing contract and collateral records from legacy source systems. Computes `payloadDifference` between source data and target RDS schema. Writes full raw payload + diff to MongoDB (`ops_*_event` collections) for team review before committing. |
| Legacy Data Upsert | Concept | After team verification of the Audit Trail, commits the migrated records into Genesis RDS. Uses the same upsert logic as the event-driven path (Contract Master and Collateral Master capabilities). |
| Historical Replay | Concept | Uses the S3 archive (`s3://turbo-master-data/logs_event/`) to replay historical events and re-derive Genesis RDS state if needed (e.g. after schema changes or data corrections). |
| Migration Dashboard | Concept | Internal view for the Genesis team showing migration progress: total records, migrated count, error count, `payloadDifference` discrepancy rate. |

---

## Business Rules

| Rule | Description |
|------|-------------|
| Audit before commit | No legacy record may be committed to Genesis RDS without first passing through the Audit Trail (MongoDB `ops_*_event`) and being reviewed by the Genesis team. |
| `payloadDifference` must reach 0% | Migration is not considered complete until the discrepancy rate between source data and RDS schema is verified at 0% (target: > 99% accuracy per PRODUCT.md KPI). |
| Audit Trail is internal only | MongoDB `ops_*_event` collections are not a product — they are a team tool. No downstream consumer may read from this store. |
| No data loss | All source records must be accounted for — either migrated successfully or explicitly flagged as unmigratable with documented reason. |
| Migration does not block event-driven path | Batch Migration runs concurrently with the event-driven ingestion path. The same upsert logic applies; if a migrated record is later updated by a live event, the event-driven update takes precedence. |

---

## Open Questions

- What are the legacy source systems? (Names, database types, access methods)
- What is the total volume of contracts and collaterals to migrate?
- What is the expected `payloadDifference` rate before any data cleaning?
- Who reviews and approves the Audit Trail before the commit step? (Genesis team lead? Data steward?)
- What is the go-live date constraint that defines the migration deadline?
- Is there a rollback plan if post-migration discrepancies are found in production?
