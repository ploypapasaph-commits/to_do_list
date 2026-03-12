# Feature: Receiver Account Pre-Check

**Capability**: Disbursement Orchestration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-07

---

## User Story

> *"As the Onigiri system, I want to automatically pre-check the receiver's transfer destination via the appropriate API when the bank account input section is completed in the draft step, so that invalid destination issues are surfaced to the user before the application advances."*

---

## Transfer Type Options

The customer selects one of two transfer methods. The pre-check API and validation logic differ per type:

| Transfer Type | Input Required | Pre-Check API | Validates |
|---|---|---|---|
| **Bank Account** | Account number + Bank code | Bank / Core Banking API | Account exists and is active |
| **PromptPay** | PromptPay ID (mobile number or National ID) | PromptPay lookup API | PromptPay ID is registered and active |

The transfer type selection determines which fields are shown in the bank account section and which pre-check API is called.

---

## Job-to-be-Done

The customer selects a transfer method (Bank Account or PromptPay) and fills in the corresponding receiver details during the loan application draft. The moment the section is fully completed, Onigiri triggers a pre-check call to the appropriate API to validate that the destination exists and is eligible to receive funds. This prevents disbursement failure at a later stage by catching invalid destinations early — while the user is still able to correct the information.

---

## State Flow

```
draft
  │
  │  [Bank account section completed — auto-trigger]
  ▼
checking_receiver_account
  │
  ├── [valid]    ──→  draft (section marked valid, user may proceed)
  │
  └── [invalid]  ──→  draft (section flagged, user must correct account info)
```

> The application returns to `draft` in both outcomes. The pre-check result is stored and surfaced inline on the form. The application cannot submit until the pre-check result is `valid`.

---

## Acceptance Criteria

All criteria are pass/fail verifiable.

### Transfer Type Selection

| # | Given | When | Then |
|---|-------|------|------|
| AC-1 | Application in `draft`. | User selects **Bank Account** as transfer type. | Bank account fields (account number, bank code) are shown. PromptPay fields are hidden. Any prior pre-check result is cleared. |
| AC-2 | Application in `draft`. | User selects **PromptPay** as transfer type. | PromptPay ID field is shown. Bank account fields are hidden. Any prior pre-check result is cleared. |
| AC-3 | Application in `draft`. | User changes transfer type after a `valid` pre-check result. | Pre-check result is cleared. Section resets to `pending_check`. |

### Pre-Check Trigger

| # | Given | When | Then |
|---|-------|------|------|
| AC-4 | Application in `draft`. Transfer type = **Bank Account**. All required fields (account number + bank code) are filled. | User completes the last required field. | System triggers Bank Account pre-check API call. Application transitions to `checking_receiver_account`. |
| AC-5 | Application in `draft`. Transfer type = **PromptPay**. PromptPay ID field is filled. | User completes the PromptPay ID field. | System triggers PromptPay lookup API call. Application transitions to `checking_receiver_account`. |

### Pre-Check Result

| # | Given | When | Then |
|---|-------|------|------|
| AC-6 | Application in `checking_receiver_account`. | API responds with destination valid. | Application transitions back to `draft`. Section marked `valid`. Result (account name / PromptPay registered name, reference ID, timestamp) stored in RDS and displayed to user for confirmation. |
| AC-7 | Application in `checking_receiver_account`. | API responds with destination invalid. | Application transitions back to `draft`. Section flagged `invalid` with reason. Result stored in RDS. User must correct and re-trigger. |
| AC-8 | Application in `checking_receiver_account`. | API returns error (timeout, 5xx, network failure). | Application transitions back to `draft`. Section flagged `check_failed`. User notified and may retry. Error stored in RDS. |

### Submission Gate

| # | Given | When | Then |
|---|-------|------|------|
| AC-9 | Application in `draft`. Bank account section flagged `invalid` or `check_failed`. | User edits any field in the section. | Pre-check result cleared. Section resets to `pending_check`. Pre-check re-triggers on section completion. |
| AC-10 | Application in `draft`. Bank account section in `valid` state. | User attempts to submit the application. | Submission allowed. |
| AC-11 | Application in `draft`. Bank account section NOT in `valid` state. | User attempts to submit the application. | Submission blocked. Error: transfer destination must be verified before submission. |
| AC-12 | Application in any state other than `draft`. | Any pre-check trigger occurs. | Request ignored. No state change. |

---

## Edge Cases and Error States

| Case | Behaviour |
|------|-----------|
| **User edits account info while check is in progress** | In-flight API call result is discarded on return. Section resets to `pending_check`. New pre-check triggers when section is completed again. |
| **User switches transfer type while check is in progress** | In-flight API call result is discarded. Section resets to `pending_check` for the newly selected type. |
| **Bank Account API timeout** | Falls under AC-8. Returns to `draft` with `check_failed`. Retry is user-initiated. Maximum timeout: TBD — must be defined with bank API team. |
| **PromptPay ID not registered** | Falls under AC-7 (destination invalid). Reason displayed: "PromptPay ID not found." |
| **Pre-check valid but account closed by disbursement time** | Pre-check at draft does not guarantee validity at disbursement. Core Banking's fund transfer is the definitive check. Pre-check reduces — but does not eliminate — disbursement failure risk. |
| **No fields changed after valid result** | If no field in the section has changed, no re-trigger occurs. Existing `valid` result stands. |
| **API unavailable (prolonged outage)** | Submission blocked until `valid` result obtained. No override; pre-check is mandatory for both transfer types. |

---

## Out of Scope

- **What fields constitute the bank account section.** Field definition is owned by the Smart Form capability.
- **Displaying the pre-check result UI.** Front-end rendering is out of scope for this spec.
- **The fund transfer itself.** This feature covers only the draft-time pre-check. The actual transfer and its callback are covered by `FEATURE_fund-transfer-callback-handler.md`.

---

## Dependencies

| Dependency | Type | Notes |
|---|---|---|
| Bank Account pre-check API contract | External (Bank / Core Banking) | Endpoint, request/response schema, error codes, timeout SLA, and auth mechanism must be defined before advancing to Spec. |
| PromptPay lookup API contract | External (PromptPay / NITMX) | Endpoint, request/response schema, error codes, timeout SLA, and auth mechanism must be defined before advancing to Spec. |
| Smart Form — bank account section field definition | Intra-capability (Smart Form) | The trigger condition "section completed" requires a defined list of required fields per transfer type. Smart Form capability owns this definition. |
| Underwriting Workflow — 4-Phase State Machine | Intra-product | `checking_receiver_account` must be a valid state in the state machine. |

---

## Open Questions

1. **What is the maximum acceptable pre-check latency for each type?** Bank Account API vs PromptPay lookup may have different SLAs. This determines whether each check is synchronous (inline) or asynchronous (polling/webhook).
2. **Can a branch officer override a failed pre-check?** If yes, what role is required and how is the override recorded for audit?
3. ~~**Is pre-check mandatory for all product types?**~~ **Resolved (2026-03-07):** Pre-check is mandatory for both **Bank Account** and **PromptPay** transfer types, with no exceptions. Submission is always blocked until a `valid` result is obtained.
4. **PromptPay API provider?** NITMX (via PromptPay gateway) or routed through Core Banking? Affects auth and schema.

---

## Status

`Concept` — user story and acceptance criteria defined. **Blocked on:**
- Bank account pre-check API contract (not yet defined)
- Smart Form bank account section field list
- `checking_receiver_account` state registration in state machine
- ~~Business decision on override policy (Open Question 3)~~ — **Resolved:** pre-check is mandatory; no override path required.
