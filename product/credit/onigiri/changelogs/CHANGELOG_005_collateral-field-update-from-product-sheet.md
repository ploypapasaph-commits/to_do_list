# Changelog 005: Collateral Field Update — Real Field Data from Product Sheet

**Product**: Onigiri (Loan Origination System) + Matcha (Document Verification Service)
**Portfolio**: Credit + Operations
**Changelog #**: 005
**Layer Affected**: Capability (Smart Form) + Architecture (Matcha ARCHITECTURE.md)
**Date**: 2026-03-06
**Session Type**: @CAPABILITY field update + Architecture alignment

---

## Summary

Replaces placeholder collateral field definitions in Smart Form CAPABILITY.md with confirmed field data sourced from the *All Product Key-in Fields* product sheet. Updates Matcha ARCHITECTURE.md to reflect new document type keys, collateral-conditional document logic, and the car-only constraint on `carCheckConfig`.

**Source:** `All-Product-Key-in-Fields.md` — field data extracted from the Google Sheet "All product Key-in" for Bike, Tractor, and Land collateral types.

---

## What Changed

| # | Layer | Change | Document |
|---|-------|--------|----------|
| 1 | Capability | **Modified**: Replaced placeholder `collateral_bike` section (7 fields) with confirmed 17-field definition including ownership/history fields (`bike_registration_date`, `bike_possession_date`, `bike_ownership_type`, `bike_previous_owner`), act type (`bike_act_type`), engine CC (`bike_engine_cc`), tax fields, and name match check | [capabilities/smart-form/CAPABILITY.md](../capabilities/smart-form/CAPABILITY.md) |
| 2 | Capability | **Modified**: Replaced placeholder `collateral_tractor` section (6 fields) with confirmed 21-field definition including ownership/history fields, act type, color, horsepower, working hours, modification status, and accessories checklist | [capabilities/smart-form/CAPABILITY.md](../capabilities/smart-form/CAPABILITY.md) |
| 3 | Capability | **Modified**: Replaced placeholder `collateral_land` section (9 fields) with confirmed 5-sub-section structure: (1) Ownership — 7 fields; (2) Title Deed Details — 24 fields + 7 condominium conditional fields; (3) Land Characteristics — 3 fields; (4) Appraisal Data — 4 fields; (5) Registered Address — 16 fields | [capabilities/smart-form/CAPABILITY.md](../capabilities/smart-form/CAPABILITY.md) |
| 4 | Capability | **Modified**: Replaced placeholder `collateral_car` section (8 fields) with confirmed 28-field definition from UI screenshot, including vehicle type selector (รถเก๋ง/รถกระบะ/รถตู้), ownership/history block, gear, sub-model, door count, description, system-not-found checkbox, multi-select color picker (up to 3), mileage, gas modification status, other modification status, name match radio | [capabilities/smart-form/CAPABILITY.md](../capabilities/smart-form/CAPABILITY.md) |
| 5 | Architecture | **Modified**: Expanded `document_verification_mapping` entity description in S3 to include the full document type key registry per collateral type (Car, Bike, Tractor, Land) with conditional column; added `carCheckConfig` applicability note (car-only) | [product/operations/matcha/ARCHITECTURE.md](../../operations/matcha/ARCHITECTURE.md) |
| 6 | Architecture | **Modified**: Updated `carCheckConfig.applicationId` field description in S6 API Contracts to explicitly state car-applications-only; omitted for Bike, Tractor, Land | [product/operations/matcha/ARCHITECTURE.md](../../operations/matcha/ARCHITECTURE.md) |
| 7 | Architecture | **Modified**: Updated F1 acceptance criteria to include conditional document inclusion logic (RY-17 bike, RY-13 tractor) and `carCheckConfig` car-only constraint | [product/operations/matcha/ARCHITECTURE.md](../../operations/matcha/ARCHITECTURE.md) |
| 8 | Architecture | **Modified**: Updated F2 acceptance criteria — expanded document type seed count from "35 types" to "35 + 7 new types = 42 total" (pending final count confirmation) | [product/operations/matcha/ARCHITECTURE.md](../../operations/matcha/ARCHITECTURE.md) |

---

## Key Findings from Product Sheet

### Bike / Tractor — Shared Pattern
Both bike and tractor collateral sections share the same ownership/history structure (registration date, possession date, ownership type, previous owner). Key differences:
- **Bike**: engine CC (`จำนวนซีซี`) — was an open question, now confirmed required; chassis/engine numbers use select+text (source type dropdown + manual entry); no color field
- **Tractor**: color field present; horsepower + working hours fields (agricultural usage metrics); accessories checklist; modification status

### DLT Web Page Photo — Conditional Rules
| Collateral | Act Type Exclusion | Document Key |
|-----------|-------------------|--------------|
| Bike | RY-17 (ร.ย.17) → no DLT photo | `motorbike_dlt_web_page` |
| Tractor | RY-13 (ร.ย.13) → no DLT photo | `tractor_dlt_web_page` |

This conditional logic lives in Onigiri's `document_verification_mapping` table — a `conditional_expr` column evaluated by the Worker before building the POST /task payload. Matcha itself has no knowledge of act types.

### Land — Multiple Sections, GPS Required, Tambon Required
The land collateral section confirmed:
- Tambon (sub-district) IS required — resolves previous open question
- GPS coordinates (latitude/longitude) ARE required — resolves previous open question
- Google Map Pin (map attachment) IS required — resolves previous open question
- Land section has 5 distinct sub-sections, with 7 condominium-specific conditional fields (triggered by `land_collateral_type = condominium / อช.2`)
- Collateral type (`ประเภทหลักประกัน`) and sub-type (`ประเภทหลักประกันย่อย`) are both captured — providing more granularity than the previous title deed type field alone

### carCheckConfig — Car Only
Confirmed that `carCheckConfig` in the Matcha POST /task payload is only populated for car loan applications. Bike, Tractor, and Land applications do not trigger a car check and must omit this field.

---

## Resolved Open Questions (from CHANGELOG_004)

| # | Original Question | Resolution |
|---|------------------|------------|
| 1 | Are Bike collateral fields complete (engine displacement / mileage needed)? | ✅ Resolved: engine CC (`จำนวนซีซี`) is required. Mileage is not in the field list. |
| 2 | Is tractor agricultural certificate always required or conditional? | ✅ Resolved: tractor section has no agricultural certificate flag. The tractor document set is: registration book + DLT photo (conditional on act type). No agricultural usage certificate in the confirmed field list. |
| 3 | Is Tambon required for Land? | ✅ Resolved: Yes — `ตำบล` (Tambon/subdistrict) is required. |
| 4 | Are GPS coordinates or map attachment required for Land? | ✅ Resolved: Yes — both `ละติจูด/ลองติจูด` and `Pin google map` are required fields. |
| 5 | Are all 3 deed types acceptable, or Chanote only? | ⚠️ Partially resolved: the field `land_title_deed_type` exists as a select, implying multiple types. Acceptable values list requires confirmation from Credit PO. |

---

## Remaining Open Questions

| # | Question | Affects | Owner |
|---|----------|---------|-------|
| 1 | What are the acceptable values for `land_title_deed_type`? Are NS-3K and NS-3G accepted in addition to Chanote? | Land eligibility rule + `land_title_deed_type` field options | Credit PO |
| 2 | What are the acceptable values for `land_collateral_type` and `land_collateral_subtype`? | `collateral_land` field options; condominium conditional logic trigger | Credit PO |
| 3 | ~~What is the DLT photo exclusion rule for Car (`vehicle_dlt_web_page`)? Does a car equivalent of the RY-17/RY-13 rule exist?~~ | ✅ Resolved — Car has no act-type exclusion. `vehicle_dlt_web_page` is always required. | — |
| 4 | ~~Does the car collateral section include ownership/history fields?~~ | ✅ Resolved via UI screenshot — car section confirmed | — |
| 5 | What is the confirmed total count of Matcha `DocumentType` seed entries after adding the 7 new types? | F2 acceptance criteria final count | Engineering (Matcha) |
| 6 | ~~Does the `car_vehicle_type` selector (รถเก๋ง / รถกระบะ / รถตู้) change which fields appear, or do all three sub-types share the same field set shown in the screenshot?~~ | ✅ Resolved — Sedan and Van share the same field set. Pickup Truck has one additional field: `car_canopy_installed` (ติดคอกตู้หรือไม่). | — |

---

## Follow-Up Actions Required

| # | Action | Owner | Urgency |
|---|--------|-------|---------|
| 1 | Add `conditional_expr` column to `document_verification_mapping` table schema — required to support act-type-based DLT photo exclusion for Bike and Tractor | Engineering (Onigiri Worker) | High — blocks correct Matcha task creation for Bike/Tractor campaigns |
| 2 | Seed `document_verification_mapping` with Bike, Tractor, Land rows including conditional logic | Engineering (Onigiri) | High — blocks `pending_document_checking` state for new campaigns |
| 3 | Add 7 new `DocumentType` seeds to Matcha repo (`motorbike_registration_book`, `motorbike_insurance`, `motorbike_dlt_web_page`, `tractor_registration_book`, `tractor_dlt_web_page`, `land_title_deed`, `land_appraisal_certificate`) | Engineering (Matcha) | High — Matcha rejects POST /task if document type key not found |
| 4 | Implement `collateral_bike`, `collateral_tractor`, `collateral_land` Smart Form sections | Engineering (Smart Form) | Medium — depends on field confirmations (Open Questions 1–2) |
| 5 | Confirm whether `car_vehicle_type` (sedan/pickup/van) changes the field set or is purely a classification field — update `collateral_car` if sub-type drives field variants | Credit PO / Engineering | Medium |
