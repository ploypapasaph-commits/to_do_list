# Feature: Application High-Water Mark

**Parent Capability**: Underwriting Workflow — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri (Loan Origination System)
**Engineering Owner**: TBD
**Status**: Spec

---

## User Story

As the **workflow engine**, I want to record the highest-order state an application has ever entered on the application record, so that downstream consumers — specifically the Smart Form — can determine which field groups are permanently locked regardless of subsequent returns to Draft.

## Job-to-be-Done

The loan workflow allows applications to return to Draft from multiple states (Risk Assessment, Approval, QA). When this happens, the current workflow state resets to `draft`, destroying the history of how far the application has progressed. Without a separate, immutable record of that progress, the Smart Form has no reliable way to know which fields were already reviewed and authorized by an upstream authority. The HWM is that record.

---

## Acceptance Criteria

### AC-1: HWM Field Exists on Application Record

- [ ] The application record contains a `state_high_water_mark` field (integer).
- [ ] The field is initialized to `1` (Draft) when the application is first created.
- [ ] The field is stored in RDS alongside the workflow state record.

### AC-2: HWM Written on Every State Entry

- [ ] When the workflow engine transitions the application into any state, it writes the HWM **at state entry**, before any execution steps run in that state.
- [ ] If the incoming state's order is **greater than** the current HWM value, the HWM is updated to the incoming state's order.
- [ ] If the incoming state's order is **less than or equal to** the current HWM value, the HWM is **not changed**.
- [ ] The HWM update and the state transition are committed atomically in the same database transaction.

### AC-3: HWM is Monotonically Increasing

- [ ] No code path in the workflow engine sets HWM to a value lower than its current value.
- [ ] This holds true for all return-to-Draft paths: from Risk Assessment, Approval, QA (cash and non-cash), and Supervisor recall.

### AC-4: HWM Order Table

The following integer order values map to workflow states:

| HWM Value | State |
|-----------|-------|
| 1 | `draft` |
| 2 | `risk_assessment` |
| 3 | `pending_approval` |
| 4 | `create_facility` |
| 5 | `cash_routing` / `confirmation` / `qa` |
| 6 | `create_loan_disbursement` |
| 7 | `funded` / `rejected` / `withdrawn` / `expired` |

- [ ] The HWM integer values above are the authoritative mapping used by the workflow engine when evaluating and writing HWM.
- [ ] The Smart Form lockpoint API reads HWM using these integer values — no additional mapping layer is introduced.

### AC-5: HWM Readable by Smart Form API

- [ ] The application data API exposes `state_high_water_mark` as a top-level field in the application response payload.
- [ ] The Smart Form field lock API uses the `state_high_water_mark` value (not the current workflow state) as the sole input for evaluating field lock status.

### AC-6: HWM Visible in Audit Log

- [ ] Every workflow transition audit log entry includes the `state_high_water_mark` value at the time of the transition (before and after, if changed).

---

## Edge Cases & Error States

| Scenario | Expected Behaviour |
|----------|--------------------|
| Application returns to Draft from QA (cash path) after disbursement | HWM remains `6` (`create_loan_disbursement`). Smart Form locks `LOAN_TERMS`, `DISBURSEMENT_CHANNEL`, and `ALL_FINANCIAL` groups. |
| Application returns to Draft from Approval before Create Facility | HWM is `3` (`pending_approval`). Smart Form locks `LOAN_TERMS` group only. `DISBURSEMENT_CHANNEL` group is still editable. |
| Application returns to Draft from Risk Assessment | HWM is `2` (`risk_assessment`). No field groups are locked (lock threshold for `LOAN_TERMS` is `3`). |
| Supervisor recalls application from `waiting_for_confirmation` | HWM does not retreat. It remains at the highest state previously reached. |
| Workflow engine attempts to write HWM lower than current value | Rejected silently — HWM is not updated. No error raised. |
| HWM field is missing on a legacy application record | Treated as HWM = `1` (Draft). Backfill migration required before feature goes live. |

---

## Out-of-Scope

- HWM does **not** determine field lock status directly — that is the responsibility of the Smart Form's Field Lockpoint Enforcement feature.
- HWM does **not** affect workflow routing or state transition eligibility — it is read-only metadata for downstream consumers.
- HWM does **not** replace the workflow audit trail — both records are maintained independently.

---

## Dependencies

| Dependency | Type | Notes |
|------------|------|-------|
| Workflow State Machine Engine | Internal — same capability | HWM write is triggered by the state transition event emitted by the engine |
| Smart Form — Field Lockpoint Enforcement | Consumer | Reads `state_high_water_mark` to evaluate lock groups — [FEATURE](../../smart-form/features/FEATURE_field-lockpoint-enforcement.md) |
| Application RDS schema | Data | `state_high_water_mark` column added to application record |

---

## NFR Escalations (to Capability layer)

- **Atomicity**: HWM update must be in the same transaction as the state transition. Partial writes (state updated, HWM not updated) are a data integrity failure.
- **Auditability**: HWM value at transition time must appear in the workflow audit log entry.
