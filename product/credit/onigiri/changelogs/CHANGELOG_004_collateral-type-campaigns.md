# Changelog 004: Collateral Type Campaign Documentation — Bike, Tractor, Land

**Product**: Onigiri (Loan Origination System)
**Portfolio**: Credit
**Changelog #**: 004
**Layer Affected**: Capability
**Date**: 2026-03-05
**Session Type**: @CAPABILITY documentation expansion

---

## Summary

Expands Onigiri documentation to explicitly support Bike (motorbike title), Tractor (agricultural equipment title), and Land (title deed) as loan campaign collateral types. No new capabilities, features, or architectural changes are introduced.

The existing five-dimension campaign configuration model (Pricing, Eligibility, Application Template, Risk Strategy, Workflow Steps) already supports these types fully. This changelog makes the support explicit by:

1. Introducing **Collateral Type** as a named, first-class concept in Loan Campaign Configuration — with a cross-reference table covering all four types (Car, Bike, Tractor, Land)
2. Documenting the linkage between Application Template document declarations and the Matcha POST /task `documents[]` array
3. Expanding the Eligibility Criteria Examples table from car-only to all four collateral types
4. Naming the expected risk strategies for Bike, Tractor, and Land
5. Adding a **Collateral Section Variants** registry to Smart Form with field-level definitions for all four collateral types

---

## What Changed

| # | Layer | Change | Document |
|---|-------|--------|----------|
| 1 | Capability | **Modified**: Added "Collateral Type as Configuration Driver" section — 5-row table showing how each configuration dimension is shaped by collateral type; establishes that no new capability is required to launch Bike/Tractor/Land campaigns | [capabilities/loan-campaign-configuration/CAPABILITY.md](../capabilities/loan-campaign-configuration/CAPABILITY.md) |
| 2 | Capability | **Modified**: Added "Collateral Type Variants Reference" table — authoritative cross-reference of all four collateral types with Smart Form section IDs, Matcha document key lists (shared base + type-specific), and indicative LTV/loan amount ranges; includes a paragraph documenting how Application Template declarations flow to Matcha | [capabilities/loan-campaign-configuration/CAPABILITY.md](../capabilities/loan-campaign-configuration/CAPABILITY.md) |
| 3 | Capability | **Modified**: Replaced car-only "Eligibility Criteria Examples" table with a four-column table (Car, Bike, Tractor, Land) covering collateral type gate, age, B-score, occupation group, vehicle brand examples, and land deed type eligibility | [capabilities/loan-campaign-configuration/CAPABILITY.md](../capabilities/loan-campaign-configuration/CAPABILITY.md) |
| 4 | Capability | **Modified**: Added "Risk Strategy Names by Collateral Type" section — names `BikeTitleDefault`, `TractorTitleDefault`, `LandTitleDefault`; establishes the `<CollateralType>Title<Variant>` naming convention; notes these are created in admin UI without code deployment | [capabilities/loan-campaign-configuration/CAPABILITY.md](../capabilities/loan-campaign-configuration/CAPABILITY.md) |
| 5 | Capability | **Modified**: Added "Collateral Section Variants" registry to Smart Form — field-level definitions for `collateral_car`, `collateral_bike`, `collateral_tractor`, `collateral_land`; each entry includes field definition table, Information Owner, Document Declarations, and open questions | [capabilities/smart-form/CAPABILITY.md](../capabilities/smart-form/CAPABILITY.md) |
| 6 | Capability | **Modified**: Added "Section Selection Rule" to Smart Form — clarifies that one campaign = one collateral type = one collateral section; enforcement via eligibility rule | [capabilities/smart-form/CAPABILITY.md](../capabilities/smart-form/CAPABILITY.md) |

---

## Rationale

Bike, Tractor, and Land campaigns are the next planned collateral types after Car. The existing campaign configuration architecture already supports them fully — the Smart Form's Page/Section/Field model, the Risk Assessment Engine's named-strategy model, and the Application Template's document declaration mechanism are all collateral-type-agnostic by design.

The gap was that no document made Collateral Type explicit as the primary differentiator. This created two practical risks:

1. **Campaign configuration errors**: Product managers configuring new campaigns would not know which Smart Form section ID to select or which Matcha document keys to declare, because this was nowhere written down.
2. **Strategy naming inconsistency**: Risk officers creating Bike/Tractor/Land strategies had no documented naming convention or structural reference, risking drift from the `CarTitleDefault` pattern.

This changelog resolves both risks by making the implicit explicit. It does not introduce any new architectural decisions — it surfaces decisions already embedded in the design of the five configuration dimensions.

---

## Decision Log

| Decision | Options Considered | Choice | Rationale | Reversibility |
|---|---|---|---|---|
| New capability vs. documentation expansion | (A) Create a new "Collateral Type Registry" capability. (B) Expand existing Loan Campaign Configuration and Smart Form CAPABILITY.md files. | (B) Expand existing files | A new capability requires a distinct business function owner and an independent deliverable unit. "Collateral Type" is a cross-cutting concern that configures all five existing dimensions — it is not a sixth dimension with its own lifecycle. Creating a new capability here would inflate the capability count without adding business clarity. | None — adding a new capability later remains possible if collateral type configuration becomes complex enough to warrant it (e.g., if collateral appraisal becomes a first-class step in the workflow). |
| One campaign per collateral type vs. multi-collateral campaigns | (A) Allow a single campaign to accept multiple collateral types. (B) One campaign = one collateral type, enforced by eligibility rules. | (B) One campaign per collateral type | A campaign's Application Template selects one Collateral Section. Supporting multiple sections per template would require conditional form rendering logic not currently in scope. More importantly, a single campaign with mixed collateral types would make Risk Strategy Assignment ambiguous — two collateral types may require different strategies with no clean way to select both. | Medium — multi-collateral campaigns could be supported in a future iteration via conditional section rendering in the Smart Form. This would require a separate @CAPABILITY session on Smart Form and Loan Campaign Configuration. |
| Conditional vs. unconditional tractor agricultural certificate | (A) Always require `proof_of_agricultural_use`. (B) Conditionally require it based on a boolean flag in the form. | (B) Conditional (with open question flagged) | Not all tractor owners may hold an agricultural usage certificate. A conditional flag allows the campaign to require the document only when it exists, without creating separate campaigns for certificate holders vs. non-holders. | Medium — if the business decides to always require the certificate, the flag field is removed and the document declaration becomes unconditional. This is a configuration change with no structural impact. |
| Example values vs. blank fields | (A) Leave all validation values blank until confirmed by Credit PO. (B) Provide illustrative examples annotated with "(example)" and flag open questions. | (B) Examples with annotations | Blank fields make the document non-actionable for engineering estimation. Illustrative examples with explicit "(example)" annotations enable engineering to proceed against representative data while the Credit PO confirms exact values. Open questions are explicitly listed per section. | None — example values are documentation, not code. |

---

## Follow-Up Actions Required

| # | Action | Owner | Urgency |
|---|--------|-------|---------|
| 1 | **Confirm Bike collateral fields** — Confirm whether engine displacement (cc), mileage, or any additional fields are required for `collateral_bike` beyond the seven currently defined. Update Smart Form CAPABILITY.md if fields are added or removed. | Credit PO | High — blocks Smart Form engineering for Bike campaigns |
| 2 | **Confirm Tractor agricultural certificate conditionality** — Is `proof_of_agricultural_use` always required, or only when `tractor_agricultural_certificate_flag = true`? If always required, remove the flag field and make the document declaration unconditional. | Credit PO | High — blocks campaign configuration for Tractor |
| 3 | **Confirm Land field completeness** — (a) Is Tambon (sub-district) required in addition to Amphoe (district)? (b) Are GPS coordinates or a map attachment required for land collateral? (c) Are all three deed types (Chanote, NS-3K, NS-3G) acceptable, or only Chanote? | Credit PO | High — blocks Smart Form engineering and eligibility rule configuration for Land |
| 4 | **Confirm pricing parameters** — LTV ceilings and loan amount ranges in the Collateral Type Variants Reference table are marked "(example)". Must be confirmed with Credit PO and Risk Officer before any campaign is published. | Credit PO + Risk Officer | High — blocks campaign go-live |
| 5 | **Create Bike, Tractor, Land risk strategies** — Create `BikeTitleDefault`, `TractorTitleDefault`, `LandTitleDefault` strategies in the Risk Assessment Engine admin UI. Strategy design (which policies and rules) is a separate @CAPABILITY session with the Risk Officer. | Risk Officer + Risk Assessment Engine PO | Medium — blocks Risk Assessment state for new campaigns; does not block Smart Form development |
| 6 | **Seed `collateral_bike`, `collateral_tractor`, `collateral_land` sections** — Engineering must implement these three new Smart Form sections before any campaign template can reference them. Implementation depends on field definition confirmation (actions 1–3 above). | Engineering (Smart Form owner) | Medium — depends on Credit PO field confirmations |
| 7 | **Register new document type keys in Matcha** — The Matcha `DocumentType` table must include `motorbike_registration_book`, `motorbike_insurance`, `tractor_registration_document`, `proof_of_agricultural_use`, `land_title_deed`, `land_appraisal_certificate` before Onigiri can include them in the POST /task `documents[]` payload. | Operations team (Matcha PO) | Medium — blocks Pending Document Checking state for new campaigns |
| 8 | **Confirm whether Bike/Tractor campaigns require physical inspection (carCheckConfig)** — For car campaigns, Matcha's POST /task includes a `carCheckConfig` that triggers a physical vehicle inspection. If Bike or Tractor campaigns require an equivalent inspection, the `carCheckConfig` or an equivalent field must be populated when seeding the `document_verification_mapping` table. If not required, the field is left blank. | Credit PO + Matcha PO | Medium — affects `document_verification_mapping` table seeding for Bike/Tractor |
