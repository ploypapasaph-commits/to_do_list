# Product: Core Banking

**Codename**: TBD
**Portfolio**: Platform → [PORTFOLIO](../../PORTFOLIO.md)
**Status**: 📝 Draft
**Executive Owner**: CTO / Head of Core Banking
**Last Updated**: 2026-03-04

---

## Problem Statement

Every credit product the company offers requires a single, authoritative store for loan account state. Without a centralised Core Banking system:
- Each product would need to implement its own payment allocation logic, leading to inconsistency across loan types.
- Interest accrual and fee calculation would be duplicated and drift out of sync over time.
- Days Past Due (DPD) would be computed differently by each product, making portfolio-level NPL reporting unreliable.
- Downstream systems (DaVinci, Collections, Servicing) would have no single source of truth for account balance and delinquency status.

---

## Value Proposition

The authoritative financial ledger for all loan accounts. Core Banking owns the lifecycle of every account from disbursement to closure, and is the single source of truth for account balance, payment allocation, interest/fee accrual, and DPD status. All downstream systems query or subscribe to Core Banking data — they do not duplicate it.

**For whom**: All downstream systems needing loan account state (Onigiri, DaVinci, Collections, Loan Servicing, Risk Analytics); Finance and compliance teams requiring an auditable financial ledger; Operations teams managing overdue accounts.

---

## Product Boundary

**This product IS responsible for:**
- Loan Account Management: account lifecycle (Active → Settled / Written Off), authoritative balance, immutable transaction ledger
- Payment Hierarchy Engine: configurable allocation rules determining how incoming payments are applied across outstanding components (fees, penalty interest, contractual interest, principal)
- Interest & Fee Calculation: scheduled accrual, penalty interest on overdue amounts, fee amortization — produces statement-ready figures
- DPD Engine: authoritative Days Past Due counter, daily tick, reset on full catch-up, LoanDPDChanged event emission

**This product IS NOT responsible for:**
- Loan origination workflow, application management, or credit decisions (owned by **Onigiri**)
- Customer identity or contact data (owned by **DaVinci**)
- Collection workflow, assignment, or action/result tracking (owned by **Collections**)
- Repayment scheduling, statement generation, or early settlement request handling (owned by **Loan Servicing**, TBD)
- Branch task execution (owned by **Sensei**)

**This product RECEIVES from:**
- Onigiri → `CreateFacility` command (creates loan account structure at credit decision) → via synchronous API
- Onigiri → `DisburseLoan` command (activates account, triggers disbursement) → via synchronous API
- Payment Gateway / External → `PaymentReceived` event (incoming payment to be allocated) → via event
- Internal Scheduler → `DailyAccrualTrigger` (triggers interest calculation and DPD tick) → via scheduled job

**This product SENDS to:**
- DaVinci → `LoanDisbursed` event (account created and funded) → via event
- DaVinci → `LoanPaymentReceived` event (payment processed, with new balance breakdown) → via event
- DaVinci → `LoanStatusChanged` event (status transitions: e.g., Active → Default) → via event
- DaVinci → `LoanDPDChanged` event (DPD bucket change: e.g., 0→1, 30→31, 90→91) → via event
- DaVinci → `LoanClosed` event (account settled or written off) → via event
- Onigiri → `CreateFacilityResult` (loan account ID, confirmation) → via synchronous API response
- Onigiri → `FundTransferResult` (fund transfer outcome: success / failure, with `transferReferenceId`) → via webhook callback

---

## Capability Registry

| Capability | Owner | Status | Description |
|-----------|-------|--------|-------------|
| [Loan Account Management](capabilities/loan-account-management/CAPABILITY.md) | Engineering | Draft | Account lifecycle (Active → Settled / Written Off). Authoritative balance store: principal, outstanding balance, accrued interest, accrued fees, overdue amounts. Immutable transaction ledger. Multi-product support (personal, car, SME). Account status events emitted. |
| [Payment Hierarchy Engine](capabilities/payment-hierarchy-engine/CAPABILITY.md) | Engineering | Draft | Configurable payment allocation rules per product type. Defines order of application across outstanding components: penalty interest, contractual interest, principal, fees. Handles partial, full, over, and prepayments. Payment allocation events emitted with component breakdown. |
| [Interest & Fee Calculation](capabilities/interest-fee-calculation/CAPABILITY.md) | Engineering | Draft | Scheduled interest accrual (daily or monthly per product configuration). Penalty interest calculation on overdue amounts. Fee amortization. Produces statement-ready figures for each accounting period. Feeds Loan Account Management balance store. |
| [DPD Engine](capabilities/dpd-engine/CAPABILITY.md) | Engineering | Draft | Authoritative Days Past Due counter. Ticks daily when scheduled payment is missed. Resets on full catch-up payment. Emits `LoanDPDChanged` event on bucket transitions consumed by DaVinci, Collections, and analytics downstream. |

---

## Loan Account State Machine

```mermaid
stateDiagram-v2
    [*] --> Active : DisburseLoan command (Onigiri)

    Active --> Overdue : Payment missed — DPD > 0
    Overdue --> Active : Full catch-up payment — DPD resets to 0
    Overdue --> Default : DPD ≥ 90 (non-performing threshold)
    Default --> Overdue : Partial recovery — DPD drops below 90
    Default --> Settled : Full outstanding balance recovered
    Default --> WrittenOff : Write-off approval granted

    Active --> Settled : Full outstanding balance paid
    Overdue --> Settled : Full outstanding balance paid (with arrears)

    Settled --> [*]
    WrittenOff --> [*]
```

**DPD Bucket Reference** (used by downstream systems for segmentation):

| Bucket | DPD Range | Label |
|--------|-----------|-------|
| 0 | 0 | Current |
| 1 | 1–30 | Early Overdue |
| 2 | 31–90 | Mid Overdue |
| 3 | 91–180 | Default |
| 4 | 181+ | Severe Default |

---

## Product-Level Metrics and KPIs

| Metric | Description | Target |
|--------|-------------|--------|
| Balance Accuracy | % of account balances matching independent ledger reconciliation | 100% |
| DPD Event Latency | Time from daily scheduler trigger to `LoanDPDChanged` event emitted (p95) | < 5 minutes |
| Payment Processing Latency | Time from `PaymentReceived` event to balance updated and `LoanPaymentReceived` emitted (p95) | < 30 seconds |
| Accrual Completeness | % of accounts with no missed daily accrual (nightly batch audit) | 100% |
| Event Delivery Reliability | % of state-change events successfully delivered to DaVinci (with retry) | 99.99% |

---

## Detailed Reference

For technical architecture and data models, see: [ARCHITECTURE.md](ARCHITECTURE.md) *(to be created)*
