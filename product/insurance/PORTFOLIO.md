# Portfolio: Insurance

> **Strategic Domain:** Insurance Distribution & Brokerage
> **Business Owner:** TBD
> **Last Updated:** 2026-03-05

---

## Investment Thesis

The Insurance portfolio covers all systems required to operate as an insurance broker. The company does not underwrite risk -- it distributes insurance products from multiple insurer partners to end customers through branch and online channels.

The strategic value of this portfolio is:
1. **Multi-insurer aggregation** -- customers compare packages across insurers in a single experience
2. **Multi-channel distribution** -- branch (staff-operated) and online (self-service) with channel-specific rules
3. **Insurer integration abstraction** -- heterogeneous insurer systems (file transfer, REST API, manual) are hidden behind a unified product interface

---

## Constituent Products

| Product | Codename | Status | Summary |
|---------|----------|--------|---------|
| Insurance Distribution Platform | OnePiece (ワンピース) | Draft | End-to-end insurance sales platform. Quotation comparison across insurers, multi-channel application management (branch + online), payment processing with channel-specific routing, policy issuance via multiple insurer integration methods, and post-sale endorsement/cancellation brokering. |

---

## Portfolio-Level OKRs

| Objective | Key Result | Status |
|-----------|------------|--------|
| Launch insurance distribution capability | OnePiece product defined with all capabilities specified | In Progress |
| Multi-insurer coverage | Support at least 5 insurer partners at launch | In Progress (5 partners active) |
| Multi-channel sales | Branch and online channels operational | Planned |

---

## Cross-Product Dependencies

| Dependency | Product | Portfolio | Nature |
|------------|---------|-----------|--------|
| Customer master data | DaVinci | Platform | OnePiece reads/writes customer data via DaVinci golden record |
| Document verification | Matcha | Operations | Installment payment (insurance loan) requires document verification through Matcha |

---

## Strategic Roadmap

| Now | Next | Later |
|-----|------|-------|
| Define product structure and capabilities | Build core sales flow (quote -> apply -> pay -> issue) | Policy renewal |
| Map insurer integration methods | Launch branch and online channels | Expand insurer partnerships |
| Define payment and issuance rules | Integrate with DaVinci and Matcha | Claims brokering |

---

## Key Risks and Constraints

| Risk | Impact | Mitigation |
|------|--------|------------|
| Insurer integration heterogeneity | Each insurer has different issuance methods; high integration effort | Abstract behind Policy Issuance capability with per-insurer adapters |
| Channel-specific business rules | Branch and online have different product availability, payment options, and flows | Centralize rules in Product Catalog & Channel Configuration capability |
| Installment payment complexity | Requires loan approval + document verification (Matcha) | Clearly scope the boundary -- OnePiece triggers Matcha, does not own verification logic |
