# ITEM: Document Requirement Declaration

**Status:** Concept
**Capability:** Product Type Configuration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-10
**Last Updated:** 2026-03-10

---

## What & Why
Admin UI that allows Product Owners to declare which documents are required for a collateral type, configure conditional inclusion/exclusion rules (e.g., exclude DLT photo when `bike_act_type = RY-17`), and set data extraction paths. At product type activation, these declarations auto-populate the `document_verification_mapping` table — replacing the current approach of engineering seeding rows via code migrations.

## Acceptance Criteria
- [ ] PO can add document requirements by selecting from Onigiri's document type registry
- [ ] PO can configure conditional inclusion/exclusion rules (WHEN field = value THEN INCLUDE/EXCLUDE)
- [ ] PO can configure JSONPath data extraction paths per document
- [ ] Shared base documents (applicant_id_card, proof_of_income, household_registration) auto-included
- [ ] Preview shows effective document checklist for given field values
- [ ] Activation auto-populates `document_verification_mapping` rows
- [ ] System validates conditional rule trigger fields exist in the associated collateral section
- [ ] Duplicate document type keys blocked per product type
- [ ] Version activation replaces old mapping rows (soft delete)

## Dependencies
- Document Type Registration feature (types must exist in registry first)
- Collateral Section Builder feature (conditional rules reference section fields)
- `document_verification_mapping` table schema (must support `conditional_expr` column)
- Onigiri Worker (reads mapping table at runtime)

## Notes / Open Questions
- Should PO be able to modify the shared base document set, or is that a product-level decision?
- Should JSONPath extraction support be deferred to a later phase?
- How to handle backward compatibility when re-activating a product type with changed document requirements?

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_document-requirement-declaration.md` created or updated in `capabilities/product-type-configuration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
