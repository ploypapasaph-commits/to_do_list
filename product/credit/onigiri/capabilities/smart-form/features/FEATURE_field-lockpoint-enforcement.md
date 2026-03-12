# Feature: Field Lockpoint Enforcement

**Parent Capability**: Smart Form — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri (Loan Origination System)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As a **Collection Officer** filling a loan application that has returned to Draft, I want form sections that contain already-reviewed or already-processed data to be read-only, so that I cannot accidentally — or intentionally — modify values that are no longer safe to change.

## Job-to-be-Done

When an application returns to Draft for document corrections, the form looks identical to a brand-new application. Without enforced locks, nothing stops a user (or a direct API caller) from changing the disbursement channel, loan amount, or other critical fields — even after a facility has been created in Core Banking or funds have been disbursed. This creates both a financial control failure and a double-disbursement exploit vector.

Enforcement is modelled at the **section level** — not the individual field level — because application data is stored as a flexible JSON document in DocumentDB. Per-field scanning of the JSON payload on every write would be prohibitively expensive and fragile to schema evolution. Section-level enforcement is a single metadata check against the section's `lock_group` tag, not a document traversal.

---

## Acceptance Criteria

### AC-1: Lock Groups Are Assigned at the Section Level

- [ ] Each section in the form configuration schema carries an optional `lock_group` property: `LOAN_TERMS`, `DISBURSEMENT_CHANNEL`, `ALL_FINANCIAL`, or omitted (no locking).
- [ ] The `lock_group` assignment lives in the section configuration — not in field definitions, not in application logic.
- [ ] Fields within a section inherit the lock state of their parent section. There is no per-field `lock_group` override.

### AC-2: Three Lock Groups and Their Thresholds

| Lock Group | Sections (Examples) | Locks When HWM ≥ |
|------------|---------------------|-----------------|
| `LOAN_TERMS` | Loan Setup — finance options, product type, term, interest rate | `3` (Approval) |
| `DISBURSEMENT_CHANNEL` | Loan Setup — disbursement channel, bank account, payment details | `4` (Create Facility) |
| `ALL_FINANCIAL` | All sections tagged `LOAN_TERMS` + `DISBURSEMENT_CHANNEL` | `6` (Create Loan + Disbursement) |

- [ ] All three groups enforce the HWM thresholds above.
- [ ] `ALL_FINANCIAL` is a superset: when HWM ≥ 6, all sections tagged `LOAN_TERMS` or `DISBURSEMENT_CHANNEL` are also locked (even if their individual thresholds have not been separately checked).

### AC-3: Server-Side Enforcement at Section Granularity

- [ ] The application data update API accepts write requests at the **section level** (each request body identifies the section being updated).
- [ ] Before writing, the API resolves the section's `lock_group` from section configuration and compares it against the application's `state_high_water_mark`.
- [ ] If the section is locked (HWM ≥ group threshold), the API rejects the entire section write with HTTP `422 Unprocessable Entity`.
- [ ] The rejection body identifies the section ID, the `lock_group`, and the HWM value that triggered the lock.
- [ ] No DocumentDB document traversal or per-field inspection is performed during lock evaluation. The check is: `section.lock_group threshold ≤ application.state_high_water_mark`.
- [ ] Server-side enforcement is authoritative. Client-side read-only rendering is UX only.

### AC-4: Read-Only Rendering of Locked Sections

- [ ] When the Smart Form loads, it reads `state_high_water_mark` from the application payload and evaluates lock state for each section before rendering.
- [ ] Sections in a locked group render all their fields as read-only — no input, no edit affordance.
- [ ] Locked sections are **never hidden**. They remain visible with their stored values so the officer can see what was committed.
- [ ] Values in locked sections are not cleared or re-initialized on form load.

### AC-5: Section-Level Lock Indicator and Tooltip

- [ ] Each locked section displays a lock indicator in its section header (not on individual fields).
- [ ] Tapping or hovering the lock indicator shows a tooltip that:
  - States which workflow event caused the lock.
  - States the remediation path.
- [ ] Tooltip copy is defined per lock group:

| Lock Group | Tooltip Text |
|------------|-------------|
| `LOAN_TERMS` | *"Loan terms were reviewed and authorized by a credit authority. Post-approval changes would bypass that authorization. To change these terms, a new application is required."* |
| `DISBURSEMENT_CHANNEL` | *"Disbursement channel was locked when a Core Banking facility was created for this application. To change the disbursement channel, cancel this application and submit a new one."* |
| `ALL_FINANCIAL` | *"Funds have been disbursed. No financial fields may be changed. To correct this application, contact your supervisor."* |

### AC-6: Submit Not Blocked by Locked Sections

- [ ] The Submit action evaluates completeness only over **unlocked** required sections.
- [ ] Locked sections are excluded from the required-section completeness check at submit time.
- [ ] An application where only locked sections have outstanding values cannot reach a submit-blocked state.

---

## Edge Cases & Error States

| Scenario | Expected Behaviour |
|----------|--------------------|
| First-time Draft (HWM = 1) | No sections locked. All groups below threshold. |
| Returned to Draft from Approval (HWM = 3) | `LOAN_TERMS` sections locked. `DISBURSEMENT_CHANNEL` sections still editable. |
| Returned to Draft from QA after Create Facility (HWM = 4 or 5) | `LOAN_TERMS` and `DISBURSEMENT_CHANNEL` sections locked. `ALL_FINANCIAL` threshold not yet reached. |
| Returned to Draft from QA after disbursement (HWM = 6) | All three groups locked. No financial section is editable. |
| Direct API call to update a locked section | HTTP 422. Section ID, lock group, and HWM value returned in response body. No DocumentDB write occurs. |
| Section has no `lock_group` in config | Never locked, regardless of HWM. |
| HWM missing on legacy application record | Treated as HWM = 1. No sections locked. Backfill migration required before feature goes live. |
| New section added to a lock group after application was created | Lock applies at access time based on current config and current HWM. Section is locked if HWM ≥ threshold. |

---

## Out-of-Scope

- This feature does **not** define HWM calculation or write logic — that is owned by [Application High-Water Mark](../../underwriting-workflow/features/FEATURE_application-high-water-mark.md).
- This feature does **not** perform per-field JSON inspection inside the DocumentDB document.
- This feature does **not** block workflow state transitions — it only controls section write access and UI rendering.
- This feature does **not** cover Execution Step Idempotency Guards in the workflow engine — those are a separate defense-in-depth layer (see [Underwriting Workflow CAPABILITY.md](../../underwriting-workflow/CAPABILITY.md)).

---

## Dependencies

| Dependency | Type | Notes |
|------------|------|-------|
| Application High-Water Mark | Upstream — Underwriting Workflow capability | Provides `state_high_water_mark` read on the application record |
| Section Configuration Schema | Internal — Smart Form | Sections must carry `lock_group` assignment; API resolves lock group from config, not from DocumentDB document |
| Application Data Update API | Internal — same product | Must enforce section-level lock check before any DocumentDB write |

---

## NFR Escalations (to Capability layer)

- **Enforcement cost**: Lock check is O(1) — one integer comparison per section write. No document traversal. This must be maintained; do not introduce per-field lock logic.
- **Server-side enforcement is non-optional**: Client read-only rendering is UX; server-side section rejection is the security control. Both must ship together — client alone is insufficient.
- **Config as source of truth for lock_group**: The `lock_group` on a section must be resolved from the section config at request time, not cached on the application document.
