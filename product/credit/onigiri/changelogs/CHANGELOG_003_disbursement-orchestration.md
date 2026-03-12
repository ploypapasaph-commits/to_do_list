# Changelog 003: Disbursement Orchestration Capability

**Product**: Onigiri (Loan Origination System)
**Portfolio**: Credit
**Changelog #**: 003
**Layer Affected**: Product / Capability / Feature
**Date**: 2026-03-04
**Session Type**: @CAPABILITY addition

---

## Summary

Retires the `next_dummy_state` placeholder introduced in Blueprint-03 (Matcha Architecture) and introduces two concrete business states — `waiting_fund_transfer` and `waiting_create_loan_operation` — along with the new **Disbursement Orchestration** capability that owns them.

This is the first specification of Onigiri's post-document-verification disbursement pipeline. Two callback handlers are defined at the @FEATURE layer: one for Matcha's `approved` outcome, and one for Core Banking's fund transfer confirmation.

---

## What Changed

| # | Layer | Change | Document |
|---|-------|--------|----------|
| 1 | Capability | **New**: Disbursement Orchestration capability folder and CAPABILITY.md created. Contains 3 decision tables, state flow diagram, NFRs, and 4 open questions. | [capabilities/disbursement-orchestration/CAPABILITY.md](../capabilities/disbursement-orchestration/CAPABILITY.md) |
| 2 | Feature | **New**: Matcha Callback Handler (Approved Path) — handles `approved` initial decision → `waiting_fund_transfer`; re-decision routing from `waiting_fund_transfer`. 8 acceptance criteria defined. | [capabilities/disbursement-orchestration/features/FEATURE_matcha-callback-handler.md](../capabilities/disbursement-orchestration/features/FEATURE_matcha-callback-handler.md) |
| 3 | Feature | **New**: Fund Transfer Callback Handler — handles Core Banking `success` → `waiting_create_loan_operation`; failure alerting. 8 acceptance criteria defined. Core Banking API contract flagged as an open dependency. | [capabilities/disbursement-orchestration/features/FEATURE_fund-transfer-callback-handler.md](../capabilities/disbursement-orchestration/features/FEATURE_fund-transfer-callback-handler.md) |
| 4 | Capability | **Modified**: Underwriting Workflow CAPABILITY.md — state definitions table expanded from 11 to 14 states (added `pending_document_checking`, `waiting_fund_transfer`, `waiting_create_loan_operation`); `next_dummy_state` retired; workflow diagram updated to include new states; cross-capability handoff note added; open questions updated. | [capabilities/underwriting-workflow/CAPABILITY.md](../capabilities/underwriting-workflow/CAPABILITY.md) |
| 5 | Product | **Modified**: Onigiri PRODUCT.md — Disbursement Orchestration added to Capability Registry (5th row); Core Banking fund transfer callback added to RECEIVES section; state diagram updated (3 new Decision phase states; `pending_document_checking` restored; `returned_for_revision` added to Terminal phase); Integration Map updated with Core Banking ↔ Onigiri fund transfer arrow. | [PRODUCT.md](../PRODUCT.md) |
| 6 | Product (Platform) | **Modified**: Core Banking PRODUCT.md — `FundTransferResult` webhook callback added to SENDS section (integration map symmetry). | [product/platform/core-banking/PRODUCT.md](../../../platform/core-banking/PRODUCT.md) |

---

## Rationale

The `next_dummy_state` placeholder was introduced in the Matcha Architecture document with the explicit note: *"Not included: Disbursement and post-verification downstream flows (→ future phase). `next_dummy_state` is a placeholder for the post-verification step."*

This session resolves that deferral. The two new states reflect the actual two-step nature of post-verification disbursement:
1. **Fund transfer** — an external financial operation by Core Banking, asynchronously confirmed back to Onigiri
2. **Loan operation creation** — the next step after funds are confirmed transferred, representing the creation of the formal loan record

Collapsing these into a single `next_dummy_state` (or even a single state) would hide discrete failure points, make SLA measurement on fund transfer latency impossible, and prevent operators from distinguishing "documents approved" from "funds moved."

---

## Decision Log

| Decision | Options Considered | Choice | Rationale | Reversibility |
|---|---|---|---|---|
| New capability vs. extending Underwriting Workflow | (A) Add states to Underwriting Workflow capability. (B) Create new Disbursement Orchestration capability. | (B) New capability | Disbursement orchestration has a distinct external dependency (Core Banking fund transfer), a distinct phase of the application lifecycle, and will likely have its own engineering owner. Adding it to Underwriting Workflow would blur the capability boundary and violate single-purpose ownership. | Medium — moving features between capabilities is a documentation refactor but signals org boundary changes. |
| Two new states vs. one | (A) Single `waiting_disbursement` state covering both fund transfer and loan operation creation. (B) Two separate states. | (B) Two states | Two discrete external system commitments. Each must be independently observable, measurable (SLA), and auditable. Collapsing them hides failure points and makes incident response harder. | Low — splitting an existing state is always possible; merging requires discarding audit granularity. |
| Core Banking PRODUCT.md update | (A) Only update Onigiri's PRODUCT.md. (B) Update both Onigiri and Core Banking PRODUCT.md. | (B) Both | Integration map symmetry is a system invariant. An outbound integration documented only on the receiver's side is an incomplete contract. It will cause confusion in cross-team planning. | None — both sides must always match. |
| Re-decision transitions ownership | (A) Underwriting Workflow owns re-decisions from `waiting_fund_transfer`. (B) Disbursement Orchestration owns them. | (B) Disbursement Orchestration | The re-decision callbacks reach the application while it is in `waiting_fund_transfer`, a state owned by Disbursement Orchestration. State-based routing (not outcome-based routing) is the correct principle. Disbursement Orchestration has context about fund transfer status that Underwriting Workflow does not. | Low — routing logic can be reassigned if organizational ownership changes. |

---

## Follow-Up Actions Required

| # | Action | Owner | Urgency |
|---|--------|-------|---------|
| 1 | **Matcha ARCHITECTURE.md update** — Rows 14-15 of Matcha's transition table still reference `next_dummy_state`. These must be updated to `waiting_fund_transfer` in a separate session with the Matcha/Operations team. File: `product/operations/matcha/ARCHITECTURE.md`. | Operations team (Matcha PO) | Medium — no functional impact until engineering picks up the integration, but leaves a stale reference in Matcha's docs. |
| 2 | **Core Banking fund transfer callback API contract** — The contract for `FundTransferResult` does not exist. It must be designed and documented before `FEATURE_fund-transfer-callback-handler.md` can advance from Concept → Spec. | Core Banking engineering team + Onigiri PO | High — blocks engineering estimation and development. |
| 3 | **Resolve Disbursement Orchestration Open Question #2** — What is the next state after `waiting_create_loan_operation`? The connection to the cash/non-cash routing (Cash? node) must be explicitly confirmed before the Disbursement Orchestration capability can be fully specified. | Credit PO + Onigiri Engineering | High — blocks full capability specification. |
| 4 | **Resolve Disbursement Orchestration Open Question #1** — What triggers the fund transfer itself? Does Onigiri initiate the Core Banking call on entering `waiting_fund_transfer`? If yes, a third feature ("Fund Transfer Initiator") must be added to this capability. | Credit PO + Onigiri Engineering | High — may require adding a feature to this capability. |
