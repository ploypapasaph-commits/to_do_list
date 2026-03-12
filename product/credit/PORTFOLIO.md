# Portfolio: Credit

**Strategic Domain**: Consumer and SME Lending Lifecycle
**Business Owner**: CPO / Chief Credit Officer
**Status**: Active Investment
**Last Updated**: 2026-03-09

---

## Investment Thesis

The Credit portfolio owns the full lending lifecycle — from the moment a customer starts a loan application to the last day any loan is active. Today, **Onigiri** covers the origination leg (application → underwriting → disbursement). As the business scales, the portfolio will expand to include **Loan Servicing** (repayment scheduling, statements, early settlement) and **Collections** (delinquency management, recovery tracking) as distinct products with distinct lifecycles and owners.

These products belong in the same portfolio because:
- They share the same customer entity (resolved via DaVinci).
- They share the same document verification infrastructure (Matcha / Wasabi).
- Their data flows sequentially — an approved loan in Onigiri becomes an active loan in Servicing; a delinquent loan in Servicing becomes a collection case in Collections.
- Portfolio-level OKRs (loan book quality, origination-to-collection funnel) require unified measurement.

**Why not a single product?** Each lifecycle stage has a distinct operational owner, team, technology surface, and performance cadence. Origination is sales-led (branch COs), Servicing is operations-led (back-office), and Collections is specialist-led (collection officers). Separate products prevent scope creep and ownership dilution.

---

## Constituent Products

| Product | Codename | Status | PRODUCT.md | Description |
|---------|----------|--------|------------|-------------|
| Loan Origination System | Onigiri (おにぎり) | 📝 Draft | [PRODUCT](onigiri/PRODUCT.md) | Loan application intake → underwriting workflow → credit decision → disbursement |
| Credit Scoring Service | Miso (味噌) | 📝 Draft | [PRODUCT](miso/PRODUCT.md) | Campaign-to-model routing, A/B traffic splitting, model inference execution, standardized score output, immutable audit log |
| Asset Valuation Service | Dashi (だし) | 📝 Draft | [PRODUCT](turbo-rate/PRODUCT.md) | Multi-source vehicle price ingestion, configurable cleansing pipeline, vehicle identity resolution, price consolidation, risk team rate management dashboard, canonical market rate API |
| Loan Servicing | TBD | ⏳ To be defined | — | Repayment scheduling, statement generation, early settlement |
| Collections | TBD | ⏳ To be defined | — | Delinquency management, collection workflow, recovery tracking |

---

## Portfolio-Level OKRs

| Objective | Key Result | Target |
|-----------|-----------|--------|
| Accelerate loan origination | Avg. origination cycle time (Draft → Funded) | < 5 business days |
| Increase underwriting automation | % of applications with zero manual risk rule changes post-launch | > 90% |
| Reduce document errors at intake | % of Matcha tasks auto-verified by AI (no human QA) | > 70% |
| Enable rapid product launch | Time to launch a new loan campaign (without code changes) | < 2 business days |
| Improve portfolio quality | NPL rate within 6 months of origination | < 3% |

---

## Cross-Product Dependencies

```
Credit Portfolio — Internal
  Onigiri → Miso       : application JSON + campaign_id for credit scoring (Risk Assessment state)
  Miso → Onigiri       : standardized score object { rating, risk_band, indicators[], trace_id }
  Onigiri → Dashi      : vehicle identifier (make + model + year + grade) for canonical market rate lookup
  Dashi → Onigiri      : canonical market rate { vehicle_id, canonical_rate, rate_basis, adjustment_applied }

Credit Portfolio → Operations Portfolio
  Onigiri → Matcha     : Document verification after facility creation
  Onigiri → Wasabi     : AI early-warning scan during Draft phase
  Onigiri → Sensei     : Task creation when branch action required

Credit Portfolio → Platform Portfolio
  Onigiri → DaVinci    : Customer identity resolution on application creation
  Onigiri → DaVinci    : ApplicationCreated, ApplicationApproved events

Credit Portfolio → External
  Onigiri    → Core Banking : Facility creation + fund transfer initiation
  Core Banking → Onigiri   : Fund transfer COMPLETE callback { status, transferResult: Success|Reject, transferReferenceId }
  Onigiri    → NCB          : Credit bureau inquiry via OTP consent
```

---

## Strategic Roadmap

| Horizon | Initiative | Owner |
|---------|-----------|-------|
| **Now** | Onigiri — complete capability implementation (Smart Form, Underwriting, Risk Engine, Campaign Config) | Credit PO |
| **Now** | Onigiri — Wasabi early-warning integration in Draft state | Credit PO + AI/ML |
| **Now** | Miso — feature decomposition and engineering owner assignment | Credit PO + Engineering |
| **Now** | Dashi — feature decomposition and engineering owner assignment | Credit PO + Risk Management |
| **Next** | Miso — integrate with Onigiri Risk Assessment state (score routing active) | Credit PO + Engineering |
| **Next** | Dashi — source onboarding: register initial price sources and activate cleansing configs | Risk Management + Engineering |
| **Next** | Dashi — integrate with Onigiri collateral valuation step (vehicle rate lookup) | Credit PO + Engineering |
| **Next** | Onigiri — AI-first verification routing via Matcha/Wasabi (Phase 2) | Credit PO + Operations |
| **Next** | Define Loan Servicing product (PRODUCT.md + capability registry) | Credit CPO |
| **Later** | Define Collections product (PRODUCT.md + capability registry) | Credit CPO |
| **Later** | Portfolio-level reporting: origination → servicing → collections funnel | Analytics |

---

## Key Risks and Constraints

| Risk | Severity | Mitigation |
|------|----------|-----------|
| Onigiri scope creep beyond origination | High | Strict IS/IS NOT boundary enforced in PRODUCT.md. Servicing and Collections are separate products. |
| Campaign configuration complexity grows unbounded | Medium | Campaign configuration must remain no-code. Any feature that requires code to configure is a boundary violation. |
| Wasabi confidence threshold causes false positives | Medium | 99.99% threshold is ultra-conservative. Monitor false positive rate via spot-check sampling. |
| Servicing/Collections products not defined before Onigiri goes live | Low | Onigiri ends at disbursement. Integration handoffs (loan_id to Core Banking) are the boundary. Servicing definition can follow. |
| Dashi rate coverage gaps (vehicle not in catalog) | Medium | Rate Publishing API returns a structured `not_found` response. Onigiri must implement graceful fallback for vehicles without a Dashi rate. Fallback policy (reject application vs. use manual valuation) is a Credit PO decision. |
| Source data quality — inconsistent or fraudulent price sources | Medium | Immutable raw storage + cleansing audit trail enables post-hoc investigation. Quarantine workflow prevents bad data reaching canonical store without super-user review. |
