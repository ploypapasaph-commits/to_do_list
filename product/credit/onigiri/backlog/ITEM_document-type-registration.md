# ITEM: Document Type Registration

**Status:** Concept
**Capability:** Product Type Configuration
**Product:** Onigiri
**Owner:** TBD
**Created:** 2026-03-10
**Last Updated:** 2026-03-10

---

## What & Why
Self-service document type registration owned by Onigiri. Product Owners register new document type keys (e.g., `motorbike_registration_book`) via Admin UI. Onigiri stores them in its own `document_type_registry` table and syncs to Matcha via API at product type activation. This eliminates the need for engineering to seed Matcha's `DocumentType` table via code migrations for every new collateral type.

## Acceptance Criteria
- [ ] PO can register a new document type with key, display name, and category
- [ ] System enforces snake_case naming convention for keys
- [ ] Duplicate keys rejected (across both Onigiri-registered and Matcha-seeded types)
- [ ] PO can view all document types with source indicator (Onigiri vs Matcha seed)
- [ ] Pre-existing Matcha-seeded types are read-only
- [ ] Activation syncs new types to Matcha via POST /document-types
- [ ] Matcha 409 (already exists) treated as success (idempotent)
- [ ] Matcha failure blocks product type activation with clear error
- [ ] Registered types available in Document Requirement Declaration picker

## Dependencies
- **Matcha Document Type Registration API** — Matcha must expose `POST /document-types` endpoint (cross-product dependency)
- Product Type Publication Authorization feature (sync triggered at activation)
- Document Requirement Declaration feature (consumes registered types)

## Notes / Open Questions
- Should document type categories be extensible by POs, or a fixed enum maintained by engineering?
- Should there be a bulk registration option (e.g., upload CSV)?
- What happens if Matcha deprecates or renames a document type key?
- **Cross-product action required:** Create a backlog item in Matcha for the registration API endpoint

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_document-type-registration.md` created or updated in `capabilities/product-type-configuration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
