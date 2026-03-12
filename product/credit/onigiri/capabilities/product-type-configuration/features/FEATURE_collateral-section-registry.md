# Feature: Collateral Section Registry

**Parent Capability**: Product Type Configuration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-11

---

## User Story

As an **Engineer**, I want to create and register collateral sections (field definitions, validation rules, conditional visibility) in a structured registry, so that Product Owners can select from pre-built sections when assembling product types — without needing to understand form internals.

As a **Product Owner**, I want to browse and preview available collateral sections, so that I can select the right one when assembling a product type.

## Job-to-be-Done

Collateral sections are complex: 17–60+ fields each, with specific types, validation rules, conditional visibility logic, and lockpoint group assignments. Expecting POs to build these from scratch would introduce high error risk and unnecessary cognitive load. Instead, engineering owns section creation — defining fields, types, validation, and conditional logic — and registers each section in a queryable registry. POs then **select** a section from the registry when assembling a product type. This preserves engineering quality control over form definitions while still allowing POs to independently assemble and launch product types.

---

## Scope

**IN scope (Engineering):**
- Define a collateral section: section ID, display name, field list (name, type, label, validation, required flag, conditional visibility, lockpoint group)
- Organize fields into named sub-sections
- Register the section in the Collateral Section Registry (database table or config store)
- Version sections: changes to an existing section create a new version

**IN scope (PO — via Product Type Builder):**
- Browse available sections in the registry
- Preview a section's rendered form (read-only)
- Select a section for inclusion in a product type

**OUT of scope:**
- PO creating or modifying section field definitions (engineering-only)
- PO writing validation rules or conditional visibility logic
- Runtime dynamic form generation from arbitrary field configs

---

## Collateral Section Registry Schema

| Field | Type | Description |
|-------|------|-------------|
| `section_id` | String (unique) | Convention: `collateral_<type>` (e.g., `collateral_bike`) |
| `display_name` | String | Human-readable name (e.g., "Bike (Motorbike Title)") |
| `version` | Integer | Monotonically increasing; new version on each change |
| `status` | Enum | `ACTIVE` / `DEPRECATED` |
| `field_count` | Integer | Total fields in the section |
| `sub_sections` | JSON | Named groupings of fields |
| `fields` | JSON | Full field definition array (see below) |
| `created_by` | String | Engineer who created the section |
| `created_at` | Timestamp | Registration timestamp |

### Field Definition Schema (per field)

| Property | Type | Description |
|----------|------|-------------|
| `field_name` | String | Unique within section (e.g., `bike_brand`) |
| `label` | String | Display label |
| `label_th` | String | Thai display label |
| `type` | Enum | `text`, `number`, `date`, `dropdown`, `file_upload`, `checkbox`, `gps_coordinates` |
| `required` | Boolean | Whether field is mandatory |
| `validation` | JSON | Type-specific rules (max_length, min/max, regex, value_list, etc.) |
| `conditional_visibility` | JSON | `{ trigger_field, trigger_value, action: show/hide }` or null |
| `lockpoint_group` | String | Which lockpoint group this field belongs to |
| `sub_section` | String | Which sub-section this field is grouped under |

---

## Current Section Registry

| Section ID | Display Name | Fields | Sub-sections | Status |
|-----------|-------------|--------|-------------|--------|
| `collateral_car` | Car (Vehicle Title) | 28 | 1 | Spec'd (in Smart Form CAPABILITY.md) |
| `collateral_bike` | Bike (Motorbike Title) | 17 | 1 | Spec'd (in CHANGELOG_004/005) |
| `collateral_tractor` | Tractor (Agricultural Equipment) | 21 | 1 | Spec'd (in CHANGELOG_004/005) |
| `collateral_land` | Land (Title Deed) | 59 | 5 | Spec'd (in CHANGELOG_004/005) |

All 4 sections are fully spec'd in existing documentation. Engineering implementation is pending.

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Engineer registers a new collateral section | Section stored in registry with unique `section_id`, field definitions, and `ACTIVE` status |
| AC-2 | Section ID uniqueness enforced | Duplicate `section_id` rejected at registration time |
| AC-3 | PO can browse available sections | Product Type Builder shows all `ACTIVE` sections with display name and field count |
| AC-4 | PO can preview a section | Read-only form preview renders all fields, sub-sections, and conditional logic |
| AC-5 | PO can select a section for a product type | Selected section linked to product type; section's field list available for document requirement configuration |
| AC-6 | Section versioning works | Updating a section creates a new version; existing product types pinned to activation-time version |
| AC-7 | Deprecated sections hidden from new selection | `DEPRECATED` sections do not appear in the picker; existing product types continue to work |
| AC-8 | Field definitions are complete | Every field has: name, type, label, required flag, validation rules, lockpoint group |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| PO needs a section that doesn't exist yet | PO requests from engineering; engineering creates and registers it; PO selects once available |
| Section updated after product type activation | Existing product types pinned to activation-time version; new product types use latest version |
| Section deprecated while product types reference it | Existing product types continue to work; no new product types can select it |
| Engineer registers section with 0 fields | Validation error: "Section must have at least one field" |
| Two engineers update the same section concurrently | Optimistic locking: second save fails with conflict error |

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Smart Form capability | Internal — [CAPABILITY](../../smart-form/CAPABILITY.md) | Field specifications aligned with Smart Form's existing field type definitions |
| Product Type Builder | Internal — Product Type Configuration | PO-facing UI that reads from the registry |
| Existing section specs | Documentation | Car, Bike, Tractor, Land sections already fully defined in CAPABILITY.md and CHANGELOGs |
