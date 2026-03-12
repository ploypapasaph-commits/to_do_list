# BACKLOG: Loan Origination System

**Product:** Onigiri — Loan Origination System
**Portfolio:** Credit
**Last Updated:** 2026-03-11
**Last Session:** Refined Product Type Configuration: renamed Collateral Section Builder → Collateral Section Registry (engineering-owned). PO selects pre-built sections; does not build from scratch. Reduces user complexity.

---

## ✅ LIVE
| Feature | Capability | Shipped | Item File |
|---|---|---|---|

## 🔨 IN PROGRESS
| Feature | Capability | Owner | Started | Item File |
|---|---|---|---|---|

## 📋 SPEC'D (ready to build)
| Feature | Capability | Priority | Item File |
|---|---|---|---|

## 💡 CONCEPT (not yet specced)
| Feature | Capability | Item File |
|---|---|---|
| Collateral Section Registry | Product Type Configuration | [ITEM](backlog/ITEM_collateral-section-registry.md) |
| Document Requirement Declaration | Product Type Configuration | [ITEM](backlog/ITEM_document-requirement-declaration.md) |
| Document Type Registration | Product Type Configuration | [ITEM](backlog/ITEM_document-type-registration.md) |
| Product Type Publication Authorization | Product Type Configuration | [ITEM](backlog/ITEM_product-type-publication-authorization.md) |

## 🗄️ DEPRECATED
| Feature | Capability | Date | Item File |
|---|---|---|---|

---

## SESSION LOG
| Date | Type | Summary | Docs Changed |
|---|---|---|---|
| 2026-03-10 | @CAPABILITY | Added Product Type Configuration capability (4 features) to enable zero-code collateral type definition. Design decisions: Onigiri owns doc type registry, same 2-tier approval, template-based builder. | CAPABILITY.md (new), 4x FEATURE_*.md (new), 4x ITEM_*.md (new), BACKLOG.md (new), PRODUCT.md (updated), CHANGELOG_006 |
| 2026-03-11 | @FEATURE | Refined ownership: Collateral Section Builder → Collateral Section Registry. Engineering creates sections; PO selects. Reduces user complexity — POs don't build 17-60+ field forms. | CAPABILITY.md, FEATURE_collateral-section-registry.md (renamed), ITEM_collateral-section-registry.md (renamed), BACKLOG.md, CHANGELOG_006 |
