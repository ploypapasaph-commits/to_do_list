# Feature: Matcha Callback Handler

**Capability**: Disbursement Orchestration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-09

---

## User Story

> *"As the Onigiri system, I want to receive a Matcha callback from `pending_document_checking` and route the loan application to the correct next state based on the outcome, so that each verification result is handled deterministically without manual intervention."*

---

## Job-to-be-Done

The loan application is blocked in `pending_document_checking`, waiting for Matcha's external document verification decision. When Matcha sends a callback, Onigiri routes based on `outcome`:

| `outcome` | Meaning | Routing |
|---|---|---|
| `approved` — loan amount changed | Documents verified; disbursement details differ from approval | Transition → `waiting_for_confirmation` (branch must confirm updated details) |
| `approved` — loan amount unchanged | Documents verified; disbursement details identical to approval | Bypass `waiting_for_confirmation`. Transition → `waiting_fund_transfer` |
| `returned` | Documents require revision | Transition → `draft`. Request document upload from branch. |
| `referred` | Documents require senior review | Transition → `pending_approval`. Publish CA work-entry via Raijin. |

All transitions must be atomic and idempotent, guarding against double-transition on duplicate callback delivery.

Any Matcha callback received after the application has left `pending_document_checking` is a no-op — Matcha has no authority over state transitions from that point onward.

---

## Acceptance Criteria

All criteria are pass/fail verifiable.

| # | Given | When | Then |
|---|-------|------|------|
| AC-1 | Application in `pending_document_checking`. `matcha_task_uuid` on record matches callback `taskUuid`. **Loan amount has changed** from the approved amount. | POST `/api/credit-application/verification-callback` with `outcome=approved`, `isReDecision=false`. | Application transitions to `waiting_for_confirmation` in a single PG transaction. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-1b | Application in `pending_document_checking`. `matcha_task_uuid` on record matches callback `taskUuid`. **Loan amount has not changed** from the approved amount. | POST `/api/credit-application/verification-callback` with `outcome=approved`, `isReDecision=false`. | Application bypasses `waiting_for_confirmation` and transitions directly to `waiting_fund_transfer` in a single PG transaction. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-2 | Application in `pending_document_checking`. `matcha_task_uuid` on record matches callback `taskUuid`. | POST with `outcome=returned`, `isReDecision=false`. | Application transitions to `draft` in a single PG transaction. Document upload requested. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-3 | Application in `pending_document_checking`. `matcha_task_uuid` on record matches callback `taskUuid`. | POST with `outcome=referred`, `isReDecision=false`. | Application transitions to `pending_approval` in a single PG transaction. CA work-entry published via Raijin (processId=3, CA role) within the same transaction. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-4 | Application in `pending_document_checking`. Same `taskUuid` + completion event ID has already been processed (duplicate delivery). | Same callback arrives again. | No state change. Response: 200 OK. (Idempotent.) |
| AC-5 | Application in any state other than `pending_document_checking`. | Any Matcha callback arrives. | No state change. Payload stored in MongoDB for audit. Response: 200 OK. |
| AC-6 | Callback `taskUuid` does not match `matcha_task_uuid` stored on the application. | Any Matcha callback arrives. | No state change. No payload stored. Response: 404. |
| AC-7 | Application in `pending_document_checking` with valid `taskUuid`. Onigiri's PG write fails mid-transition. | POST with any valid `outcome`. | State is not partially updated. Response: 5xx. (Allows Matcha's retry logic — 3× exponential backoff — to redeliver.) |

---

## Edge Cases and Error States

| Case | Behaviour |
|------|-----------|
| **Duplicate callback after successful processing** | Matcha retries because its HTTP client timed out, but Onigiri had already committed. Detected by idempotency key (taskUuid + completion event ID) in RDS. No-op, 200 OK. |
| **Loan amount comparison definition** | The loan amount evaluated for the bypass condition is the disbursement amount recorded at the time the application entered `pending_document_checking`. It is compared against the originally approved disbursement amount stored at the `pending_approval` → `create_facility` transition. If the amounts are equal, the bypass applies. The comparison field and precision (e.g., currency unit, rounding) must be defined before this feature advances from Concept → Spec. |
| **`isReDecision=true` on any state** | Matcha re-decisions have no routing effect in this capability. Treat as AC-5: no-op, payload stored, 200 OK. Flag in monitoring. |
| **Malformed payload (missing required fields)** | Return 400. Do not store payload. Do not change state. |

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| Underwriting Workflow — 4-Phase State Machine feature | Intra-product | `waiting_for_confirmation`, `waiting_fund_transfer`, `draft`, and `pending_approval` must all be valid inbound targets from `pending_document_checking`. |
| Matcha — `POST /api/credit-application/verification-callback` contract | External (Matcha product) | Defined in `product/operations/matcha/ARCHITECTURE.md`. This feature is a consumer of that contract. |
| Raijin work-entry publisher | Cross-cutting infrastructure | Required for AC-3: publish CA work-entry (processId=3) when `referred` outcome received. Same mechanism used by existing Underwriting Workflow transitions. |
| Shared Matcha callback router | Intra-product | The physical endpoint receives all Matcha callbacks. The router dispatches by current application state + outcome. All three outcomes from `pending_document_checking` are now owned by this feature. |

---

## Status

`Concept` — user story and acceptance criteria defined. Not yet ready for engineering estimation. Blocked on:
- Loan amount comparison field definition (currency field name, precision, rounding rules)
- Confirmation of the Raijin publisher mechanism for AC-3
- Engineering owner assignment
