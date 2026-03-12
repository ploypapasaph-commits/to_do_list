# ITEM: Product Type Publication Authorization

**Status:** Concept
**Capability:** Product Type Configuration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-10
**Last Updated:** 2026-03-10

---

## What & Why
Two-tier approval workflow for product type definitions, reusing the same state machine pattern as Campaign Publication Authorization. Product type definitions are high-impact — they determine what fields, documents, and data flow into the system. This feature ensures CPO + Risk Officer (Tier 1, parallel) and CRO (Tier 2, sequential) review and approve before any product type becomes available for campaign selection. At activation, Onigiri also syncs new document types to Matcha, with a dedicated SYNC_FAILED state for retry handling.

## Acceptance Criteria
- [ ] Submit transitions product type to PENDING_T1 (read-only)
- [ ] CPO and Risk Officer notified simultaneously on submission
- [ ] Either T1 rejector returns product type to DRAFT with feedback
- [ ] Both T1 approvals required before CRO notification (PENDING_T2)
- [ ] CRO approval triggers Matcha document type sync
- [ ] Successful sync transitions to ACTIVE; product type available for campaign selection
- [ ] Failed sync enters SYNC_FAILED; retry available without re-approval
- [ ] Submitter cannot self-approve T1 actions
- [ ] ACTIVE product types are versioned (edit creates new DRAFT)
- [ ] Immutable audit trail for every state transition

## Dependencies
- Underwriting Workflow state machine engine (provides the topology infrastructure)
- Campaign Publication Authorization (mirror feature; same pattern)
- Document Type Registration (Matcha sync triggered at activation)
- RBAC / role system (CPO, Risk Officer, CRO role checks)

## Notes / Open Questions
- Should there be a separate "Withdraw" action for POs to pull back a submitted product type?
- Should the SYNC_FAILED state have a maximum retry count before escalating to engineering?
- Should CRO delegation be supported (acting CRO when CRO is unavailable)?

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_product-type-publication-authorization.md` created or updated in `capabilities/product-type-configuration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
