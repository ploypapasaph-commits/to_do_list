# Feature: Fund Transfer Callback Handler

**Capability**: Disbursement Orchestration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-09

---

## User Story

> *"As a loan officer, I want to confirm loan payment to the customer so that the application transitions from `waiting_for_confirmation` through `waiting_create_facility` (system) to `waiting_fund_transfer` and the disbursement can proceed."*
>
> *"As a loan officer, I want to reject the confirmation so that the application transitions from `waiting_for_confirmation` to `rejected` when the disbursement should not proceed."*
>
> *"As the Onigiri system, I want to receive a Core Banking fund transfer `COMPLETE` callback and route the loan application to the correct next state based on the `transferResult` payload (Success or Reject), so that each transfer outcome is handled deterministically without manual intervention."*

---

## State Flow

```
waiting_for_confirmation
        │
        ├── [Loan officer Confirm]  ──→  waiting_create_facility  ──→  waiting_fund_transfer
        │                                  [system auto-advance]              │
        │                                                          [Core Banking POSTs COMPLETE callback]
        │                                                                      │
        │                                                          ├── [transferResult=Success]  ──→  waiting_create_loan_operation
        │                                                          │
        │                                                          └── [transferResult=Reject]   ──→  rejected
        │
        └── [Loan officer Reject]   ──→  rejected
```

---

## Job-to-be-Done

In `waiting_for_confirmation`, a loan officer has two choices: confirm or reject. Confirming advances the application to `waiting_create_facility` — a system state that auto-completes without human action and advances to `waiting_fund_transfer`. Rejecting terminates the application as `rejected`.

Once in `waiting_fund_transfer`, Onigiri awaits an asynchronous `COMPLETE` callback from Core Banking. The callback always carries a `transferResult` field that determines the next state:

| `transferResult` | Meaning | Next State |
|---|---|---|
| `Success` | Funds transferred successfully | `waiting_create_loan_operation` |
| `Reject` | Transfer rejected by Core Banking | `rejected` |

The callback is the sole trigger for these transitions — Onigiri must not advance past `waiting_fund_transfer` without an explicit Core Banking callback.

---

## Acceptance Criteria

All criteria are pass/fail verifiable.

### Confirm Loan Payment Function

| # | Given | When | Then |
|---|-------|------|------|
| AC-1 | Application in `waiting_for_confirmation`. | Loan officer calls `POST /applications/{id}/confirm-loan-payment` with valid authentication. | Application transitions to `waiting_create_facility` in a single PG transaction. Confirmation timestamp and officer ID recorded. System auto-advances to `waiting_fund_transfer` on facility creation completion. Response: 200 OK. |
| AC-2 | Application in `waiting_for_confirmation`. | Same officer or a different officer calls `POST /applications/{id}/confirm-loan-payment` again (duplicate). | No state change. Response: 200 OK. (Idempotent.) |
| AC-3 | Application in any state other than `waiting_for_confirmation`. | Loan officer calls `POST /applications/{id}/confirm-loan-payment`. | No state change. Response: 409 Conflict. |
| AC-4 | Request is missing authentication or caller is not an authorised loan officer role. | `POST /applications/{id}/confirm-loan-payment` is called. | Request rejected. Response: 401/403. No state change. |
| AC-5 | Application ID does not exist. | `POST /applications/{id}/confirm-loan-payment` is called. | Response: 404. No state change. |

### Reject Confirmation Function

| # | Given | When | Then |
|---|-------|------|------|
| AC-5b | Application in `waiting_for_confirmation`. | Loan officer calls `POST /applications/{id}/reject-confirmation` with valid authentication. | Application transitions to `rejected` in a single PG transaction. Rejection timestamp and officer ID recorded. Response: 200 OK. |
| AC-5c | Application in `waiting_for_confirmation`. | Same `POST /applications/{id}/reject-confirmation` is called again (duplicate). | No state change. Response: 200 OK. (Idempotent — application is already `rejected`.) |
| AC-5d | Application in any state other than `waiting_for_confirmation`. | Loan officer calls `POST /applications/{id}/reject-confirmation`. | No state change. Response: 409 Conflict. |
| AC-5e | Request is missing authentication or caller is not an authorised loan officer role. | `POST /applications/{id}/reject-confirmation` is called. | Request rejected. Response: 401/403. No state change. |
| AC-5f | Application ID does not exist. | `POST /applications/{id}/reject-confirmation` is called. | Response: 404. No state change. |

### Fund Transfer Callback

| # | Given | When | Then |
|---|-------|------|------|
| AC-6 | Application in `waiting_fund_transfer`. | Core Banking POSTs `COMPLETE` callback with `transferResult=Success` and a valid `transferReferenceId`. | Application transitions to `waiting_create_loan_operation` in a single PG transaction. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-7 | Application in `waiting_fund_transfer`. | Core Banking POSTs `COMPLETE` callback with `transferResult=Reject`. | Application transitions to `rejected` in a single PG transaction. Callback payload stored in MongoDB. Response: 200 OK. |
| AC-8 | Application in `waiting_fund_transfer`. Same `transferReferenceId` has already been processed (duplicate delivery). | Same callback arrives again. | No state change. Response: 200 OK. (Idempotent, dedup by `transferReferenceId`.) |
| AC-9 | Application in any state other than `waiting_fund_transfer`. | Any Core Banking fund transfer callback arrives. | No state change. Payload stored in MongoDB for audit. Response: 200 OK. |
| AC-10 | Application in `waiting_fund_transfer`. Onigiri's PG write fails mid-transition. | Core Banking POSTs any valid `COMPLETE` callback. | State is not partially updated. Response: 5xx. (Allows Core Banking retry.) |
| AC-11 | Callback payload references an application ID that does not exist in Onigiri. | Any Core Banking fund transfer callback arrives. | No state change. No payload stored. Response: 404. |
| AC-12 | Callback arrives without valid authentication credentials. | Any Core Banking fund transfer callback arrives. | Request rejected. Response: 401. No state change. No payload stored. |
| AC-13 | Callback payload is missing `transferReferenceId` or `transferResult`. | Malformed callback arrives. | Request rejected. Response: 400. No state change. No payload stored. |
| AC-14 | Callback `transferResult` contains an unrecognised value (not `Success` or `Reject`). | Malformed callback arrives. | Request rejected. Response: 400. No state change. No payload stored. Flag in monitoring. |

---

## Edge Cases and Error States

| Case | Behaviour |
|------|-----------|
| **Confirm called while application is not in `waiting_for_confirmation`** | Application may have already been confirmed (idempotent, 200 OK) or is in an incompatible state (409 Conflict). No state change in either case. |
| **Reject called while application is not in `waiting_for_confirmation`** | Application may have already been rejected (idempotent, 200 OK) or is in an incompatible state (409 Conflict). No state change in either case. |
| **Confirm called after application has been rejected** | Application is `rejected`. Not in `waiting_for_confirmation`. Falls under AC-3: 409 Conflict. |
| **Duplicate callback after successful processing** | Core Banking retries after network timeout, but Onigiri had already committed. Detected by `transferReferenceId` idempotency key in RDS. No-op, 200 OK (AC-9). |
| **Core Banking callback after officer rejected** | Officer rejected → `rejected` before the fund transfer was ever initiated. Core Banking should not send a callback, but if one arrives, the application will not be in `waiting_fund_transfer`. Falls under AC-9: no-op, payload stored, 200 OK. |
| **Authentication mechanism** | TBD — candidates are shared secret (HMAC-SHA256 header), mutual TLS, or IP allowlist. Must be resolved before this feature can advance from Concept → Spec. Until defined, AC-12 cannot be implemented. |

---

## Out of Scope

- **What triggers the fund transfer itself.** This feature covers only the callback receipt. Whether Onigiri initiates the fund transfer on entering `waiting_fund_transfer` (by calling Core Banking) is a separate feature not yet specified. See Capability Open Question #1.
- **Operator retry workflow for failed transfers.** This feature stores the failure and raises an alert. The operator's recovery action (manual retry, application state reversion, etc.) is out of scope for this spec.
- **What happens after `waiting_create_loan_operation`.** The next state transition is not yet defined. See Capability Open Question #2.

---

## Open Dependency: Core Banking Fund Transfer Callback API Contract

The Core Banking fund transfer callback API contract **does not yet exist**. This feature creates the requirement. The following must be defined jointly with the Core Banking team before this feature can advance from Concept → Spec:

- Endpoint path and HTTP method for Onigiri to receive the callback
- Request payload schema: minimum required fields including `transferReferenceId`, `applicationId` (or equivalent Onigiri reference), `status` (always `COMPLETE`), `transferResult` (enum: `Success` / `Reject`), and any result-specific detail fields (e.g., reject reason)
- Authentication mechanism (see Edge Cases above)
- Retry policy: how many retries, what backoff, on what HTTP response codes
- SLA: maximum time between fund transfer completion and callback delivery

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| Underwriting Workflow — 4-Phase State Machine feature | Intra-product | `waiting_for_confirmation`, `waiting_create_facility`, `waiting_fund_transfer`, `waiting_create_loan_operation`, and `rejected` must be valid states. `waiting_create_facility` must be registered as a system-auto state with an outbound transition to `waiting_fund_transfer`. |
| Core Banking — fund transfer callback API contract | External (Core Banking product) | **Not yet defined.** This feature creates the requirement. See Open Dependency above. |
| Core Banking PRODUCT.md — SENDS section | Cross-product documentation | Core Banking's product boundary must declare this as an outbound integration. See `product/platform/core-banking/PRODUCT.md`. |

---

## Status

`Concept` — user story and acceptance criteria defined. **Blocked on:**
- Core Banking fund transfer callback API contract (does not yet exist)
- Authentication mechanism selection
- `waiting_create_facility` state registration and auto-advance mechanism definition
- Engineering owner assignment
