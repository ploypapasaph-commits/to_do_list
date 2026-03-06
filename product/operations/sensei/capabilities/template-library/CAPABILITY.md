# Capability: Template Library

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-06

---

## Business Function

Serve as the **single source of truth for all configurable settings** across Sensei's playbook system. Defines what settings exist, their default values, and which role can add/adjust/change each one. Playbook Engine consumes these definitions to build and execute strategies — Template Library governs the parameters, not the routing logic.

## Why It Exists (First Principles)

- **Standardization**: Without a shared library, each playbook could define "Call" differently — different outcomes, SLAs, required fields. Template Library enforces consistency across all playbooks and branches.
- **Reusability**: Action types, urgency mappings, and objective configurations are the same across delinquency recovery, insurance renewal, and other playbooks. Define once, reference everywhere.
- **Governance**: Every configurable parameter has an explicit owner (HQ / AM). Supervisors cannot modify settings beyond their role boundary. The setting governance table is the authoritative source for what each role can change.
- **Separation of Concerns**: Playbook Engine owns *how objectives are sequenced and routed*. Template Library owns *what parameters those objectives are configured with* and *who can change them*.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Action Type Registry | Draft | HQ-owned action type definitions (Call, Visit, Admin, Wait, Notify Supervisor, Send Notification) with typed outcomes and required fields per outcome |
| SLA Defaults | Draft | Default task SLA per action type; AM can adjust per branch variant within HQ-set limits |
| Retry Limit Registry | Draft | Default retry count per action type per objective; AM can adjust within HQ-set limits |
| Urgency Tier Mapping | Draft | Mapping from `risk_level` (active portfolio) and `easiness_to_collect` (write-off portfolio) to urgency tiers; HQ-owned |
| Priority Event Mapping | Draft | Which contract events map to P1/P2/P3/P4 priority classification; HQ-owned |
| Objective Configurations | Draft | Default timing parameters (DL, วันที่สร้างงาน, อายุของงาน) and recommended action per objective; HQ sets defaults, AM can adjust within limits |
| Setting Governance | Draft | Role-based access control for every setting category — the authoritative table for who can Add / Adjust / Change each setting |

---

## Business Rules

### Setting Governance Table

This table is the **authoritative source** for role-based access across all Sensei settings.

| Setting Category | Add | Adjust | Change | Who |
|-----------------|-----|--------|--------|-----|
| Action types (name, properties) | ✅ | ✅ | ✅ | HQ only |
| Outcomes per action type | ✅ | ✅ | ✅ | HQ only |
| Required fields per outcome | ✅ | ✅ | ✅ | HQ only |
| SLA defaults per action type | ✅ | ✅ | ✅ | HQ (global default); AM (per branch variant, within HQ limits) |
| Retry limits per action type | ✅ | ✅ | ✅ | HQ (global default); AM (per branch variant, within HQ limits) |
| Urgency tier mapping (`risk_level` → tier) | ✅ | ✅ | ✅ | HQ only |
| Urgency tier mapping (`easiness_to_collect` → tier) | ✅ | ✅ | ✅ | HQ only |
| Priority event mapping (event → P1–P4) | ✅ | ✅ | ✅ | HQ only |
| Objective configurations — timing & action (values) | ✅ | ✅ | ✅ | HQ (global default); AM (within HQ limits) |
| Objective configurations — structure (schema/columns) | ❌ | ❌ | ❌ | System-defined — immutable |
| Compliance-locked steps | ✅ | ❌ | ❌ | HQ only — lock/unlock |
| Template Library structure (this table's schema) | ❌ | ❌ | ❌ | System-defined — immutable |

> **Branch variant scope**: When AM adjusts a setting, it applies to their branch variant only. The HQ global default is unchanged. AM adjustments cannot exceed HQ-set upper/lower limits.

---

### Action Type Definitions

| Action Type | Typed Outcomes | Required Fields on Specific Outcomes |
|-------------|----------------|--------------------------------------|
| 📞 Call | PTP, No Answer, Refused, Callback, Wrong Number, Line Busy, Voicemail | PTP → PTP amount + PTP date; Callback → scheduled date/time |
| 🏠 Visit | Met Customer, Not Home, Address Invalid, PTP (in-person), Refused | PTP → PTP amount + PTP date; Address Invalid → new address |
| 📋 Admin | Completed, Incomplete, Escalated | Escalated → escalation reason |
| ⏳ Wait | (auto-advances; no manual outcome) | — |
| 🔔 Notify Supervisor | Acknowledged, No Response | — |
| 📧 Send Notification | (system-dispatched; Delivered/Failed) | — |

---

### SLA Defaults by Action Type

| Action Type | Default SLA | Adjustable By |
|-------------|-------------|---------------|
| 📞 Call | 4 hours | AM (per branch variant, within HQ limits) |
| 🏠 Visit | 8 hours | AM (per branch variant, within HQ limits) |
| 📋 Admin | 24 hours | AM (per branch variant, within HQ limits) |
| ⏳ Wait | Duration defined in objective configuration | HQ only |

---

### Urgency Tier Mapping

Maps raw contract scores to urgency tiers used by Playbook Engine to select the correct System Template.

| Portfolio | Input | Scale | Sort Order | Tier Assignment |
|-----------|-------|-------|------------|----------------|
| Active | `risk_level` | 1–6 | Higher first | Configured by HQ; e.g., 5–6 = Tier 1, 3–4 = Tier 2, 1–2 = Tier 3 |
| Write-Off | `easiness_to_collect` | 1–7 | Higher first | Configured by HQ; e.g., 6–7 = Tier 1, 4–5 = Tier 2, 1–3 = Tier 3 |

---

### Priority Event Mapping

Maps contract events to P1–P4 priority classification and defines when each event fires. Drives queue ordering in Work Queue. Playbook Engine's Event Trigger Processor reads this mapping at runtime.

| Priority | Event | Trigger Date (default) | Adjustable By |
|----------|-------|------------------------|---------------|
| P1 | สัญญาถึงวันครบกำหนดชำระ | due_date = today | HQ only |
| P1 | สัญญาที่มีนัดชำระในวันนี้ | PTP_date = today | HQ only |
| P2 | สัญญาใกล้วันครบกำหนดชำระ | due_date − 7 days | HQ only |
| P2 | แจ้งเตือนก่อนนัดชำระ | PTP_date − 1 day | HQ only |
| P3 | ไม่มีวันนัดชำระ | daily check: no PTP_date set on contract | HQ only |
| P3 | ตัวที่หลุด | PTP_date < today with no payment recorded | HQ only |
| P4 | Write Off | contract.status → write_off | HQ only |

> **Trigger Date** is the date condition Sensei evaluates daily per contract. When the condition is met, the event fires and Playbook Engine creates the corresponding task based on Objective Configurations.

---

### Objective Configurations

Default timing and action parameters per Objective. Playbook Engine reads these; AM can adjust values within HQ-set limits via branch variant.

| Objective | DL | วันที่สร้างงาน (default) | อายุของงาน (default) | Action (default) | Retry (default) | AM-Adjustable |
|-----------|----|-----------------------|---------------------|-----------------|----------------|---------------|
| เอาวันนัดชำระ | d (due date) | ก่อน due 7 วัน | 7 วัน | 📞 โทร | 3 ครั้ง | Timing + action ✅ |
| แจ้งเตือนยืนยันนัดชำระ | PTP − 1 วัน | ก่อนวันนัดชำระ 1 วัน | ภายในวัน | 📞 โทร | 3 ครั้ง | Timing + action ✅ |
| เก็บยอดตามนัดชำระ | PTP | วันนัดชำระ | ภายในวัน | 📞 โทร | 3 ครั้ง | Timing + action ✅ |
| ติดตามเข้มงวด — รอบแรก | d + 3 | วันที่ list ขึ้น | 3 วัน | 🏠 ลงพื้นที่ | 1 ครั้ง | Timing ✅; Action ❌ (locked) |
| ติดตามเข้มงวด — หลังได้ PTP | PTP + 1 | วันถัดจากวันนัดใหม่ | ภายในวัน | 📞 โทร | 1 ครั้ง | Timing ✅; Action ❌ (locked) |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| HQ-only modification for restricted settings | Settings marked "HQ only" in the governance table must be restricted to HQ admin role at the system level |
| AM boundary enforcement | AM adjustments must be validated against HQ-set limits at save time — values outside limits are rejected |
| Reference stability | Existing playbook instances referencing a setting must not break when the setting is updated (versioning required) |
| Required field enforcement | Task Engine validates required fields on task closure based on action type definitions in Template Library |
