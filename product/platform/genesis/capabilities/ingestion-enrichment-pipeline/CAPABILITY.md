# Capability: Ingestion & Enrichment Pipeline

**Parent Product**: Genesis → [PRODUCT](../../PRODUCT.md)
**Product Owner**: TBD
**Status**: 📝 Draft
**Last Updated**: 2026-03-06

---

## Business Function

This is the Genesis product boundary entry point. The Ingestion & Enrichment Pipeline consumes raw domain events published by upstream systems (e.g. Onigiri), enriches the payload by calling source APIs to produce a complete master record, stores the enriched payload to S3, and publishes an Internal Event to trigger downstream Genesis APIs (Contract and Collateral) to upsert into RDS.

This capability corresponds to feature **F6: Full Payload Adapter** in the Genesis implementation plan.

**Key transformation:** Converts Business Events (e.g. "loan approved") into Data Events (e.g. "master data upsert required") that are safe and complete for the Golden Record store.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Upstream Event Consumer | Concept | Subscribes to RabbitMQ Exchange (Event Trigger A, upstream vhosts). Consumes domain events from Onigiri and other upstream systems. |
| Payload Enrichment | Concept | Calls source APIs to fetch additional fields required by the Genesis RDS schema. Produces a complete, enriched payload. |
| S3 Payload Archive | Concept | Stores the full enriched payload to `s3://turbo-master-data/logs_event/`. Used as the read source for genesis-contract-api and genesis-collateral-api. Also used by Batch Processor for historical replays and audit. |
| Internal Event Publisher | Concept | Publishes an Internal Event to the `ods.internal` exchange (vhost: `ods`) after successful S3 write. Triggers Contract and Collateral capabilities to process. |

---

## Business Rules

| Rule | Description |
|------|-------------|
| Enrich before store | No raw upstream event may be written directly to RDS. Enrichment must complete before the S3 write. |
| S3 write before internal event | The Internal Event to `ods.internal` must only be published after the enriched payload is confirmed written to S3. |
| Idempotency | Re-processing the same upstream event must produce the same S3 payload and the same Internal Event. Duplicate events must not create duplicate records. |
| Enrichment failure handling | If a source API call fails during enrichment, the event must not be silently dropped. Dead-letter queue or retry strategy required (to be defined in ARCHITECTURE.md). |

---

## Open Questions

- What is the full list of upstream events consumed? (Confirm with Onigiri team — event names, payload schema, RabbitMQ vhost and exchange names)
- What source APIs are called during enrichment, and what fields do they provide?
- What is the retry policy for source API failures?
- How is idempotency enforced — by event ID, by surrogate key, or by payload hash?
