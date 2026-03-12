# Feature: Document Type Registration

**Parent Capability**: Product Type Configuration — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Concept
**Last Updated**: 2026-03-10

---

## User Story

As a **Product Owner**, I want to register new document types in Onigiri's Admin UI, so that I can declare document requirements for new collateral types without asking engineering to seed Matcha's database.

## Job-to-be-Done

Today, Matcha's `DocumentType` table contains ~35 seeded types, added via code migrations (Liquibase/EF Core). Adding new types for a new collateral type (e.g., `motorbike_registration_book`, `motorbike_insurance`, `motorbike_dlt_web_page`) requires an engineering deployment to both Matcha (seed data) and Onigiri (mapping rows). This feature creates an Onigiri-owned document type registry where POs register new types. At product type activation, Onigiri syncs new types to Matcha via API — no code deployment required.

---

## Scope

**IN scope:**
- Register new document type keys in Onigiri's `document_type_registry` table
- Define: key (unique identifier), display name, category (e.g., "Vehicle", "Land", "Identity")
- View all registered document types (both Onigiri-registered and pre-existing Matcha-seeded)
- Sync newly registered types to Matcha via `POST /document-types` at product type activation
- Mark types as synced/unsynced with Matcha

**OUT of scope:**
- Matcha verification rule configuration per document type (Matcha owns this)
- Modifying or deleting pre-existing Matcha-seeded document types
- Matcha Admin UI changes (Matcha only needs to expose a registration API)

---

## Document Type Registry Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `key` | String | Yes | Unique identifier. Convention: `<collateral>_<document_name>` (e.g., `motorbike_registration_book`) |
| `display_name` | String | Yes | Human-readable name (e.g., "Motorbike Registration Book") |
| `display_name_th` | String | No | Thai display name (e.g., "สมุดทะเบียนรถจักรยานยนต์") |
| `category` | Enum | Yes | `vehicle`, `land`, `identity`, `income`, `other` |
| `source` | Enum | Auto | `onigiri` (registered via Admin UI) or `matcha_seed` (pre-existing) |
| `matcha_synced` | Boolean | Auto | `true` once synced to Matcha; `false` until activation |
| `created_by` | String | Auto | PO user ID |
| `created_at` | Timestamp | Auto | Registration timestamp |

### Key Naming Convention

System enforces: `<category>_<descriptive_name>` in snake_case.

| Valid | Invalid | Why |
|-------|---------|-----|
| `motorbike_registration_book` | `MotorbikeRegistration` | Not snake_case |
| `land_title_deed` | `document_1` | Not descriptive |
| `tractor_dlt_web_page` | `motorbike_registration_book` (duplicate) | Already exists |

---

## Matcha Sync Protocol

```mermaid
sequenceDiagram
    participant PO as Product Owner
    participant Onigiri as Onigiri Admin UI
    participant Registry as document_type_registry
    participant Matcha as Matcha API

    PO->>Onigiri: Register new document type
    Onigiri->>Registry: INSERT (matcha_synced = false)
    Note over Registry: Type exists in Onigiri only

    PO->>Onigiri: Submit product type for approval
    Note over Onigiri: 2-tier approval workflow runs

    Onigiri->>Onigiri: CRO approves → activation begins
    Onigiri->>Registry: SELECT WHERE matcha_synced = false AND product_type = X

    loop For each unsynced type
        Onigiri->>Matcha: POST /document-types { key, display_name, category }
        alt Success
            Matcha-->>Onigiri: 201 Created
            Onigiri->>Registry: UPDATE matcha_synced = true
        else Failure
            Matcha-->>Onigiri: 4xx/5xx Error
            Onigiri->>Onigiri: Activation blocked; error shown to PO
        end
    end
```

### Failure Handling

| Scenario | Behavior |
|----------|----------|
| Matcha returns 409 (type already exists) | Treat as success — type was previously synced (idempotent) |
| Matcha returns 400 (invalid key format) | Activation blocked; PO must fix the key in the registry |
| Matcha returns 500 / timeout | Activation blocked; retry available; PO sees "Matcha sync failed" error |
| Partial sync (some types synced, some failed) | Rollback: unmark synced types; activation blocked; PO retries full sync |

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | PO registers a new document type | Type appears in registry with `matcha_synced = false` |
| AC-2 | System enforces key naming convention | Invalid keys rejected with helpful error message |
| AC-3 | Duplicate key rejected | System prevents registration of a key that already exists (in Onigiri or Matcha seed) |
| AC-4 | PO views all document types | Registry shows both Onigiri-registered and Matcha-seeded types with source indicator |
| AC-5 | Activation syncs to Matcha | New types synced via `POST /document-types`; `matcha_synced` updated to `true` |
| AC-6 | Matcha 409 treated as success | Idempotent handling; activation proceeds |
| AC-7 | Matcha failure blocks activation | Product type stays in approval state; PO sees clear error |
| AC-8 | Pre-existing Matcha types are read-only | PO cannot edit or delete types with `source = matcha_seed` |
| AC-9 | Registered type available for document requirement declaration | New types appear in the document picker of the Document Requirement Declaration feature |

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| PO deletes a registered type that's used in a document requirement | Deletion blocked: "Type is referenced by product type [X]" |
| Matcha API is down during activation | Activation blocked; PO retries later; no partial state |
| PO registers a type but product type is never activated | Type remains in registry with `matcha_synced = false`; no Matcha impact |
| Two POs register the same key concurrently | Database unique constraint prevents duplicate; second PO sees conflict error |
| Matcha changes its API contract | Engineering updates the sync adapter; no PO-facing changes |

---

## Cross-Product Dependency

**Matcha must expose a document type registration API.** This is a prerequisite for this feature.

| Endpoint | Method | Payload | Response |
|----------|--------|---------|----------|
| `/document-types` | POST | `{ key: string, display_name: string, category: string }` | 201 Created / 409 Conflict / 400 Bad Request |

**Action required:** Create a backlog item in Matcha product for exposing this endpoint.

---

## Dependencies

| Dependency | Type | Notes |
|-----------|------|-------|
| Matcha Document Type Registration API | External — Matcha | Must expose `POST /document-types`; cross-product dependency |
| Product Type Publication Authorization | Internal — [FEATURE](FEATURE_product-type-publication-authorization.md) | Sync happens at activation (after CRO approval) |
| Document Requirement Declaration | Internal — [FEATURE](FEATURE_document-requirement-declaration.md) | Registered types are consumed by document requirement configuration |
