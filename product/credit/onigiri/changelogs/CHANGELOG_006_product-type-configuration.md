# CHANGELOG 006: Product Type Configuration Capability

**Date**: 2026-03-10 (created), 2026-03-11 (refined)
**Layer**: Capability (new) + Feature (4 new)
**Product**: Onigiri — Loan Origination System
**Author**: Session — @CAPABILITY mode

---

## What Changed

### New Capability: Product Type Configuration

Added a new capability to Onigiri that enables Product Owners to assemble collateral-backed product types — selecting from engineering-provided collateral sections, configuring document requirements, and registering document types — through an Admin UI.

This extends the existing zero-code principle (Architectural Decision D4 in Loan Campaign Configuration) **upstream** from campaign configuration to product type assembly.

### New Features (4)

| Feature | Description | Owner | Status |
|---------|-------------|-------|--------|
| **Collateral Section Registry** | Engineering pre-creates collateral sections (field definitions, validation, conditional visibility). PO selects from registry. | Engineering | Concept |
| **Document Requirement Declaration** | Admin UI to declare required documents per collateral type with conditional inclusion rules | PO (self-service) | Concept |
| **Document Type Registration** | Onigiri-owned document type registry with Matcha sync via API at activation | PO (self-service) | Concept |
| **Product Type Publication Authorization** | Two-tier approval workflow (CPO + Risk Officer → CRO) before product type activation | Cross-functional | Concept |

### Design Refinement (2026-03-11): Section Ownership

**Changed:** Collateral Section Builder → **Collateral Section Registry**

**Reason:** Collateral sections are complex (17–60+ fields with types, validation, conditional visibility). Having POs build these from scratch adds unnecessary risk and cognitive load. Engineering pre-creates sections as reusable building blocks; POs select from the registry.

**Impact:** Collateral section creation is the one remaining engineering dependency. Document requirements and document type registration remain fully self-service.

### New Files Created

| File | Path |
|------|------|
| `CAPABILITY.md` | `capabilities/product-type-configuration/CAPABILITY.md` |
| `FEATURE_collateral-section-builder.md` | `capabilities/product-type-configuration/features/` |
| `FEATURE_document-requirement-declaration.md` | `capabilities/product-type-configuration/features/` |
| `FEATURE_document-type-registration.md` | `capabilities/product-type-configuration/features/` |
| `FEATURE_product-type-publication-authorization.md` | `capabilities/product-type-configuration/features/` |
| `ITEM_collateral-section-builder.md` | `backlog/` |
| `ITEM_document-requirement-declaration.md` | `backlog/` |
| `ITEM_document-type-registration.md` | `backlog/` |
| `ITEM_product-type-publication-authorization.md` | `backlog/` |
| `BACKLOG.md` | `product/credit/onigiri/BACKLOG.md` (new — first creation) |

### Updated Files

| File | Change |
|------|--------|
| `PRODUCT.md` | Added Product Type Configuration to capability registry |

---

## Rationale

### Problem

Launching a new collateral type (e.g., Bike Title Loan) requires 3 engineering steps before a Product Manager can configure a campaign:
1. Implement Smart Form collateral section (hardcoded field definitions)
2. Register document types in Matcha (code seed via Liquibase/EF Core)
3. Seed `document_verification_mapping` rows (hardcoded rows with `conditional_expr`)

This violates the zero-code campaign launch rule (D4) and creates a dependency on engineering sprints for every new collateral type.

### Solution

A new capability — Product Type Configuration — that sits before Campaign Configuration in the workflow. Product types are reusable building blocks: defined once, used across many campaigns. The capability provides:
- Template-based form builder (POs pick from pre-built field types — no custom validation)
- Document requirement configuration with simple conditional rules
- Onigiri-owned document type registry with Matcha sync
- Same 2-tier approval governance as campaigns

### Design Decisions

| # | Decision | Choice | Rejected Alternatives |
|---|----------|--------|----------------------|
| D1 | Document Type ownership | Onigiri owns registry; syncs to Matcha via API | Matcha owns it (would require PO to use two admin UIs); Shared platform registry (over-engineering for current scale) |
| D2 | Approval workflow | Same 2-tier (CPO + Risk Officer → CRO) | Lighter single-approver (insufficient for high-impact definitions); No approval (governance risk) |
| D3 | Builder flexibility | Template-based (pick from pre-built field types) | Full flexibility (PO writes regex/validation — too risky); Guided builder with curated field types (less flexible, similar effort to implement) |
| D4 | Separate capability vs. extending Campaign Configuration | Separate capability | Extending Campaign Configuration (conflates "define what it IS" with "create a specific campaign using it"; different lifecycle and personas) |

---

## Cross-Product Dependencies

| Product | Dependency | Action Required |
|---------|-----------|----------------|
| **Matcha** | Must expose `POST /document-types` registration API | Create backlog item in Matcha product |

---

## Decision Log

| # | Question | Decision | Reversibility |
|---|----------|----------|---------------|
| 1 | Who owns document type registration? | Onigiri — single Admin UI; syncs to Matcha | Medium — would require migrating registry to Matcha if reversed |
| 2 | What approval workflow for product types? | Same 2-tier as campaigns | Easy — could simplify later if governance burden too high |
| 3 | How flexible should the form builder be? | Template-based — POs pick from pre-built types | Easy — could add custom validation templates later without breaking existing ones |
| 4 | New capability or extend existing? | New capability: Product Type Configuration | Medium — could merge into Campaign Configuration later, but separation is cleaner |
