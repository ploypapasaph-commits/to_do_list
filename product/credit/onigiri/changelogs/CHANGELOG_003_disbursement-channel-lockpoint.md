# Changelog: Disbursement Channel Lockpoint

**Product**: Onigiri (Loan Origination System)
**Portfolio**: Credit
**Changelog #**: 003
**Layer Affected**: Product / Capability (Underwriting Workflow + Smart Form)
**Date**: 2026-03-09
**Author**: Claude (AI-assisted design session)

---

## What Changed

### PRODUCT.md
- Added **Cross-Capability Integrity Rules** section defining the Application State High-Water Mark (HWM) pattern and the lockpoint summary table
- Documents the cross-capability contract: Underwriting Workflow writes HWM; Smart Form reads HWM to determine field mutability

### capabilities/underwriting-workflow/CAPABILITY.md
- Added **Application High-Water Mark** feature to Feature Inventory
- Added **State High-Water Mark (HWM)** business rule section: definition, monotonic-increase guarantee, HWM order table
- Added **Execution Step Idempotency Guards** business rule section: Create Facility (skip guard) and Create Loan + Disbursement (hard block guard)

### capabilities/smart-form/CAPABILITY.md
- Added **Field Lockpoint Enforcement** feature to Feature Inventory
- Added **Field Lockpoint Groups** business rule section: three groups (`LOAN_TERMS`, `DISBURSEMENT_CHANNEL`, `ALL_FINANCIAL`) with HWM thresholds and rationale
- Added **Read-Only Rendering Rules** business rule section: lock indicator, tooltip, submit availability, server-side enforcement
- Added `lock_group` property to Field Definition Properties table

---

## Rationale

A security analysis identified a double-disbursement exploit path in the existing workflow:

1. Cash path application is disbursed (Create Loan + Disbursement executes)
2. QA returns the application to Draft for document corrections
3. Underwriter changes disbursement channel from Cash to Bank Transfer
4. Application re-enters the workflow, routes through the non-cash path
5. Create Facility executes again (second facility in Core Banking)
6. Create Loan + Disbursement executes again → **double disbursement**

The exploit requires two conditions: (A) the disbursement channel is editable in Draft after Create Facility has executed, and (B) Create Facility and Create Loan + Disbursement can execute again on re-entry.

---

## Decision Log

| Decision | Alternatives Considered | Rationale |
|----------|------------------------|-----------|
| Lock disbursement channel at `Create Facility` HWM | Lock at `Approval`; lock at `Create Loan + Disbursement` | `Create Facility` is the exact moment Core Banking commits to a disbursement type. Earlier locks (Approval) are too conservative for genuine pre-facility corrections. Later locks (post-disbursement) do not prevent the exploit path. |
| HWM stored as a field on the application record | Derive locked state from current workflow state; separate lockpoint event log | Simplest implementation. One field, monotonically increasing. No new events or tables needed. Current workflow state is insufficient because it resets on return to Draft. |
| Create Loan + Disbursement guard is a hard block, not a skip | Skip (like Create Facility guard) | A second disbursement is never safe to silently reuse. It requires explicit supervisor resolution and audit logging. Create Facility is idempotent (same facility, same ID); disbursement is not. |
| Server-side API enforcement of field locks | Client-side only | Client-side locks are UX. Any direct API call bypasses them. Server-side is authoritative. Both layers are required. |
| Remediation = cancel + new application | Allow in-place channel change with supervisor override | A channel change post-facility is a new business decision, not a correction. It requires a new Core Banking facility. A new application correctly models this as a new lifecycle event with its own audit trail. |

---

## Links

- [PRODUCT.md](../PRODUCT.md)
- [Underwriting Workflow CAPABILITY.md](../capabilities/underwriting-workflow/CAPABILITY.md)
- [Smart Form CAPABILITY.md](../capabilities/smart-form/CAPABILITY.md)
