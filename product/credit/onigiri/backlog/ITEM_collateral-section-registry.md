# ITEM: Collateral Section Registry

**Status:** Concept
**Capability:** Product Type Configuration
**Product:** Onigiri
**Owner:** TBD (Engineering)
**Created:** 2026-03-10
**Last Updated:** 2026-03-11

---

## What & Why
Engineering-owned registry of pre-built collateral sections (field definitions, validation rules, conditional visibility, lockpoint groups). Product Owners select from this registry when assembling product types — they do not build sections from scratch. This reduces user complexity (sections have 17–60+ fields with complex logic) while still enabling POs to independently assemble and launch product types. Engineering creates sections once; POs reuse them across many product types.

## Acceptance Criteria
- [ ] Engineer can register a new collateral section with unique section_id, field definitions, and ACTIVE status
- [ ] Section ID uniqueness enforced (duplicate rejected)
- [ ] PO can browse available ACTIVE sections in the Product Type Builder
- [ ] PO can preview a section's rendered form (read-only)
- [ ] PO can select a section for inclusion in a product type
- [ ] Section versioning: updates create new version; existing product types pinned to activation-time version
- [ ] Deprecated sections hidden from new selection; existing product types continue to work
- [ ] Every field has: name, type, label, required flag, validation rules, lockpoint group

## Dependencies
- Smart Form capability (field specifications alignment)
- Product Type Builder UI (PO-facing consumer of the registry)
- Existing section specs in CAPABILITY.md and CHANGELOGs (Car, Bike, Tractor, Land all fully defined)

## Notes / Open Questions
- Should section IDs follow an enforced naming convention (e.g., `collateral_<type>`)?
- Should there be an API/CLI for engineers to register sections, or a database migration approach?
- ~~Should POs build sections from scratch?~~ **Resolved:** No — too complex. Engineering creates; PO selects.

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_collateral-section-registry.md` created or updated in `capabilities/product-type-configuration/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
