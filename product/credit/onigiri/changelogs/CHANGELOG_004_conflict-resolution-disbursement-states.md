# Changelog 004: Conflict Resolution — Disbursement Orchestration State Consistency

**Product**: Onigiri (Loan Origination System)
**Portfolio**: Credit
**Changelog #**: 004
**Layer Affected**: Product / Capability / Feature
**Date**: 2026-03-07
**Session Type**: Conflict resolution pass

---

## Summary

This session resolves seven consistency conflicts introduced between CHANGELOG_003 (2026-03-04) and the feature-level updates made on 2026-03-07 (Matcha callback handler and fund transfer callback handler). Three conflicts were caused by CHANGELOG_003 documenting changes that were never written to the actual files. Four conflicts were caused by feature-level decisions made in this session (new intermediate state `waiting_for_confirmation`, three-outcome Core Banking callback model) that left the CAPABILITY.md and PRODUCT.md stale.

---

## What Changed

| # | Layer | Change | Document |
|---|-------|--------|----------|
| 1 | Capability | **Modified**: Decision Table 1 — Matcha initial approved callback now transitions to `waiting_for_confirmation` (was `waiting_fund_transfer`). | [disbursement-orchestration/CAPABILITY.md](../capabilities/disbursement-orchestration/CAPABILITY.md) |
| 2 | Capability | **Modified**: Decision Table 2 — Re-decision routing now applies from `waiting_for_confirmation` **or** `waiting_fund_transfer` (was `waiting_fund_transfer` only). Guard condition and all rows updated. | [disbursement-orchestration/CAPABILITY.md](../capabilities/disbursement-orchestration/CAPABILITY.md) |
| 3 | Capability | **Modified**: Decision Table 3 — Replaced two-outcome model (`success`/`failure`) with three-outcome COMPLETE envelope model: `transferResult=Success` → `waiting_create_loan_operation`; `transferResult=Reject` → `rejected`; `transferResult=Return` → `returned_for_revision`. | [disbursement-orchestration/CAPABILITY.md](../capabilities/disbursement-orchestration/CAPABILITY.md) |
| 4 | Capability | **Modified**: State Flow Diagram — Added `WaitConfirm → ReturnedForRevision` and `WaitConfirm → PendingApproval` re-decision branches; added `WaitFund → Rejected` (CB Reject) and `WaitFund → ReturnedForRevision` (CB Return); added `Rejected` to Terminal group. | [disbursement-orchestration/CAPABILITY.md](../capabilities/disbursement-orchestration/CAPABILITY.md) |
| 5 | Capability | **Modified**: Feature Inventory — Updated one-line descriptions for Matcha Callback Handler and Fund Transfer Callback Handler to reflect current transitions. | [disbursement-orchestration/CAPABILITY.md](../capabilities/disbursement-orchestration/CAPABILITY.md) |
| 6 | Product | **Modified**: Capability Registry — Added Disbursement Orchestration as 5th capability row (was missing despite being claimed in CHANGELOG_003). | [PRODUCT.md](../PRODUCT.md) |
| 7 | Product | **Modified**: RECEIVES section — Added Core Banking fund transfer COMPLETE callback as an explicit inbound integration. | [PRODUCT.md](../PRODUCT.md) |
| 8 | Product | **Modified**: State diagram — Replaced non-cash path's `Confirmation → Create Loan + Disbursement → Funded` with the full disbursement orchestration state chain: `pending_document_checking → waiting_for_confirmation → waiting_fund_transfer → waiting_create_loan_operation → funded`. Added `returned_for_revision` and `rejected` to Terminal phase. Added Matcha re-decision and Core Banking result branches. | [PRODUCT.md](../PRODUCT.md) |
| 9 | Product | **Modified**: Integration Map — Added `Core Banking → Onigiri: Fund Transfer COMPLETE callback` arrow. | [PRODUCT.md](../PRODUCT.md) |
| 10 | Capability | **Modified**: Underwriting Workflow CAPABILITY.md — State definitions table expanded from 12 to 17 states; added `pending_document_checking`, `waiting_for_confirmation`, `waiting_fund_transfer`, `waiting_create_loan_operation`, `returned_for_revision`; resolved `pending_approval` as the canonical machine-readable ID for the "Approval + Risk Level" state; updated Cash vs. Non-Cash path table; updated workflow diagram. | [underwriting-workflow/CAPABILITY.md](../capabilities/underwriting-workflow/CAPABILITY.md) |
| 11 | Portfolio | **Modified**: Cross-Product Dependencies map — Added `Core Banking → Onigiri` fund transfer COMPLETE callback as an explicit inbound integration from External. | [../../PORTFOLIO.md](../../PORTFOLIO.md) |

---

## Conflict Resolution Log

| Conflict # | Root Cause | Resolution |
|---|---|---|
| C1 | CAPABILITY Decision Table 1 not updated after `waiting_for_confirmation` was introduced as an intermediate state | Updated table row: `approved` → `waiting_for_confirmation` |
| C2 | CAPABILITY Decision Table 2 not updated after re-decision scope was extended to cover `waiting_for_confirmation` | Updated guard condition and all rows to cover both states |
| C3 | CAPABILITY Decision Table 3 not updated after three-outcome Core Banking model was defined | Replaced table with COMPLETE envelope schema and Success/Reject/Return rows |
| C4 | State Flow Diagram missing `Reject → rejected` and `Return → returned_for_revision` from `waiting_fund_transfer`; missing re-decision branches from `waiting_for_confirmation` | Added all missing transitions; added `Rejected` to Terminal group |
| C5 | PRODUCT.md Capability Registry not updated despite CHANGELOG_003 claiming it was | Added Disbursement Orchestration as 5th capability |
| C6 | PRODUCT.md state diagram not updated despite CHANGELOG_003 claiming it was | Full diagram replacement covering 17-state machine |
| C7 | Underwriting Workflow CAPABILITY.md not updated despite CHANGELOG_003 claiming it was | Full state table replacement and diagram update |
| C8 (naming) | `pending_approval` (feature files) vs "Approval + Risk Level" (CAPABILITY.md) — same state, two names | `pending_approval` is now the canonical state ID; "Approval + Risk Level" is the human label |
| C9 (portfolio) | PORTFOLIO.md cross-product map missing Core Banking → Onigiri callback | Added explicit reverse integration entry |

---

## Decisions Recorded

| Decision | Rationale |
|---|---|
| `pending_approval` is the canonical machine-readable state ID | Feature files and disbursement orchestration already used `pending_approval`. The capability CAPABILITY.md used a human label. Machine-readable IDs should be snake_case throughout. |
| `returned_for_revision` is classified as "Terminal (re-entry)" not truly terminal | The state exits the Disbursement Orchestration capability scope. What happens next (re-entry to underwriting, return to draft) is owned by upstream capability. Labelled accordingly to prevent confusion. |
| Non-cash path `Confirmation` and `Create Loan + Disbursement` steps replaced by disbursement orchestration states | The old ATLAS placeholders mapped 1:1 to the new states: `Confirmation` → `waiting_for_confirmation`; `Create Loan + Disbursement` → `waiting_fund_transfer` + `waiting_create_loan_operation`. The new states are more precise and owned by Disbursement Orchestration. |
