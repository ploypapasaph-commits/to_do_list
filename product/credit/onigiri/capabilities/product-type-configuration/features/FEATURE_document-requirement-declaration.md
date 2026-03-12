# Feature: Document Requirement Declaration

**Parent Capability**: Product Type Configuration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-10

---

## User Story

As a **Product Owner**, I want to declare which documents are required for a collateral type and under what conditions, so that Matcha receives the correct document checklist without engineering seeding the `document_verification_mapping` table.

## Job-to-be-Done

Today, the `document_verification_mapping` table is seeded via code — one row per upload box per collateral type. Adding documents for a new collateral type (e.g., Bike: `motorbike_registration_book`, `motorbike_insurance`, `motorbike_dlt_web_page`) requires an engineering deployment. This feature replaces the seed-data approach with an Admin UI where POs declare document requirements per product type, configure conditional inclusion rules (e.g., exclude DLT photo when `bike_act_type = RY-17`), and configure data extraction paths. At product type activation, these declarations auto-populate the `document_verification_mapping` table.

---

## Scope

**IN scope:**
- Declare document type requirements for a product type (select from Onigiri's document type registry)
- Declare shared base documents (applied to all product types: `applicant_id_card`, `proof_of_income`, `household_registration`)
- Configure conditional inclusion/exclusion rules per document
- Configure JSONPath data extraction paths per document (select from known application fields)
- Preview the resulting document checklist (showing which documents would be sent to Matcha for a given set of field values)
- Auto-populate `document_verification_mapping` rows at product type activation

**OUT of scope:**
- Matcha verification rule configuration (Matcha owns how it verifies a document)
- Document type creation (handled by Document Type Registration feature)
- Shared base document set management (changes to the base set are product-level decisions, not per-product-type)

---

## Document Requirement Configuration UI

### Document List

PO selects document types from Onigiri's registry and configures each:

| Config Field | Required | Description |
|-------------|----------|-------------|
| Document type key | Yes | Selected from registry (e.g., `motorbike_registration_book`) |
| Display name | Auto-filled | From registry; PO can override for this product type |
| Required | Yes | Always / Conditional |
| Conditional rule | If conditional | `WHEN [field] = [value] THEN EXCLUDE` |
| JSONPath extraction | No | Path(s) to extract data from application JSON for Matcha payload |

### Conditional Inclusion Rules

Same pattern as Collateral Section Builder visibility rules:

```
WHEN [field_name] = [value] THEN [INCLUDE|EXCLUDE] [document_type_key]
```

| Constraint | Rule |
|-----------|------|
| Trigger field | Must be a field in the associated collateral section |
| Condition | Equality only (`= value`) |
| Action | Include (default) or Exclude |
| Multiple conditions per document | Not supported — one rule per document. If complex logic needed, escalate to engineering. |

**Example:**
- `motorbike_registration_book` — Always required
- `motorbike_insurance` — Always required
- `motorbike_dlt_web_page` — `WHEN bike_act_type = "RY-17" THEN EXCLUDE` (otherwise included)

### Mapping to `document_verification_mapping` Table

At product type activation, each document requirement row is compiled into a `document_verification_mapping` row:

| Source (Admin UI) | Target (DB Column) |
|------------------|-------------------|
| Document type key | `document_type_key` |
| Collateral type (from product type) | `collateral_type` |
| Conditional rule | `conditional_expr` (compiled to JMESPath expression) |
| JSONPath extraction paths | `extraction_config` (JSON) |

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | PO adds a document requirement | Document type appears in the product type's document list |
| AC-2 | PO configures a conditional rule | Rule saved; preview shows document excluded when condition met |
| AC-3 | PO configures JSONPath extraction | Extraction path stored; available to Onigiri Worker at runtime |
| AC-4 | Activation populates mapping table | `document_verification_mapping` rows created matching declared requirements |
| AC-5 | Shared base documents auto-included | `applicant_id_card`, `proof_of_income`, `household_registration` included without PO action |
| AC-6 | Preview shows effective checklist | PO enters sample field values; preview displays which documents would be sent to Matcha |
| AC-7 | Document type must exist in registry | System prevents adding an unregistered document type key |
| AC-8 | Conditional rule references valid field | System validates trigger field exists in associated collateral section |
| AC-9 | Duplicate document type blocked | Cannot add the same document type key twice for one product type |
| AC-10 | Version immutability | Activating a new version replaces mapping rows for that product type; old version's rows archived |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| PO removes a document that a conditional rule references | System removes the rule; PO warned |
| Collateral section field referenced in condition is deleted | Validation error at save: "Field [X] no longer exists in section" |
| Product type activated with 0 type-specific documents | Allowed — only shared base documents sent to Matcha |
| JSONPath references a field that doesn't exist | Runtime: Worker logs warning, sends null for that extraction field |
| Re-activation (new version of same product type) | Old mapping rows soft-deleted; new rows inserted; in-flight applications use pinned version |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Document Type Registration | Internal — [FEATURE](FEATURE_document-type-registration.md) | Document types must exist in registry before they can be declared as requirements |
| Collateral Section Builder | Internal — [FEATURE](FEATURE_collateral-section-builder.md) | Conditional rules reference fields defined in the collateral section |
| `document_verification_mapping` table | Internal — Onigiri DB | Target table for compiled requirements |
| Onigiri Worker | Internal | Worker reads mapping table at `create_facility` to build Matcha POST /task payload |
