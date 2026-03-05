# Capability: Action Guide

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-05

---

## Business Function

Surface per-contract action intelligence directly within the Work Queue and Customer Page — telling COs **when to act** (timing signals based on deadline, urgency, and priority) and **how to act** (approach guidance based on contract type, portfolio, urgency tier, and current objective stage).

## Why It Exists (First Principles)

- **300–500 contracts per day**: COs cannot be expected to recall the right approach for each contract. Embedded guidance removes cognitive load and reduces errors.
- **Contract type matters**: A risk_level 6 active portfolio contract requires a very different approach than an easiness_to_collect 7 write-off contract. Generic action labels (โทร / ลงพื้นที่) are insufficient.
- **Objective stage context**: The same outcome (e.g., customer not answering) means different things at เอาวันนัดชำระ vs. ติดตามเข้มงวด — the guidance must reflect where in the chain the CO is.
- **New CO ramp-up**: Embedded talking points and required outcome reminders reduce training dependency and ensure new COs execute the playbook as designed.
- **Consistency**: If every CO applies different judgment, the playbook strategy is not actually being executed. Guidance enforces consistent execution across the team.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Timing Signal Panel | Draft | Per-contract: priority tier badge, deadline countdown, urgency score, days since last contact |
| Action Approach Guide | Draft | Per-contract: recommended approach based on portfolio type + urgency tier + current objective |
| Contract Type Context Card | Draft | Quick reference: portfolio type, risk/urgency score, payment behavior summary, last 3 contact outcomes |
| Talking Points Engine | Draft | Suggested conversation approach per objective + contract type; defined in Playbook Template by HQ |
| Required Outcome Reminder | Draft | Shows what fields the CO must capture for this specific objective step before they can save |

---

## Business Rules

### Where It Appears

| Location | What Is Shown |
|----------|--------------|
| Work Queue (contract row) | Priority badge, DL countdown, Urgency score, current Objective — already in table columns |
| Customer Page — Action Guide panel | Full Timing Signal Panel + Action Approach Guide + Talking Points |
| Rapid-Fire Mode — side panel | Condensed Timing Signal + Talking Points + Required Outcome Reminder |

---

### Timing Signal Panel

Displayed at the top of the Action Guide panel on the Customer Page. Derived in real-time from task and contract data.

| Signal | Source | Display |
|--------|--------|---------|
| Priority Tier | Event classification | P1 / P2 / P3 / P4 badge (color-coded) |
| Deadline Countdown | DL value from current objective | "X วันถึง Deadline" — red if ≤ 1 day |
| Urgency Score | `the_collection_urgency` | Score badge + tier label (e.g., "เร่งด่วนสูง") |
| Days Since Last Contact | Last completed contact task date | "ติดต่อล่าสุด X วันที่แล้ว" — flagged if gap > configured threshold |
| Objective Stage | Current position in the chain | Step indicator: เอาวันนัดชำระ → แจ้งเตือน → เก็บยอด → ติดตามเข้มงวด |
| Attempts Remaining | จำนวนทำซ้ำ remaining for this objective | "เหลือ X ครั้ง" — shown when attempts are limited |

---

### Action Approach Guide

The recommended approach varies by **portfolio type** and **current objective**:

#### Active Portfolio (risk_level 1–6)

| Objective | risk_level | Recommended Approach |
|-----------|-----------|----------------------|
| เอาวันนัดชำระ | 1–3 (ต่ำ–กลาง) | โทรแจ้งกำหนดชำระ, ขอยืนยันวันนัด, tone: friendly reminder |
| เอาวันนัดชำระ | 4–6 (สูง) | โทรเน้นความสำคัญ, escalate to visit if no answer after 2 attempts |
| แจ้งเตือนยืนยันนัดชำระ | 1–3 | โทรยืนยัน 1 วันก่อน, tone: confirmatory |
| แจ้งเตือนยืนยันนัดชำระ | 4–6 | โทรยืนยัน + เตรียมทางเลือกหากไม่สามารถชำระได้ |
| เก็บยอดตามนัดชำระ | 1–3 | โทรตรวจสอบการชำระ, ยืนยันยอด |
| เก็บยอดตามนัดชำระ | 4–6 | โทรพร้อมแจ้งผลกระทบหากไม่ชำระ, เสนอ restructure ถ้าจำเป็น |
| ติดตามเข้มงวด | 1–3 | โทร + เตรียม visit หากไม่ได้ผล |
| ติดตามเข้มงวด | 4–6 | ลงพื้นที่เป็นหลัก, ประสาน AM ถ้าไม่ได้ผล |

#### Write-Off Portfolio (easiness_to_collect 1–7)

| Objective | easiness_to_collect | Recommended Approach |
|-----------|--------------------|-----------------------|
| เอาวันนัดชำระ | 5–7 (ง่าย) | โทรเสนอข้อตกลงการชำระ, tone: cooperative |
| เอาวันนัดชำระ | 1–4 (ยาก) | ลงพื้นที่ + ประสาน AM สำหรับ legal action หรือ find new address |
| ติดตามเข้มงวด | 5–7 | โทรยืนยันข้อตกลง, เร่งปิด |
| ติดตามเข้มงวด | 1–4 | เตรียม legal action route, ส่ง AM |

---

### Contract Type Context Card

Displayed as a collapsible card on the Customer Page alongside the Action Guide panel.

| Field | Description |
|-------|-------------|
| Portfolio Type | Active / Write-Off badge |
| Risk / Urgency Tier | Score + label (e.g., risk_level 5 — เสี่ยงสูง) |
| Payment Behavior | Pattern summary: paid on time / consistently late / never paid (derived from history) |
| Last 3 Contact Results | Most recent 3 outcomes with date (e.g., 📵 ไม่รับสาย, 📋 ได้วันนัด, ✅ ชำระแล้ว) |
| PTP Success Rate | % of past PTPs that resulted in actual payment |
| DPD Current | Current days-past-due |

---

### Talking Points Engine

Talking points are defined per **Objective + portfolio_type** in the Playbook Template by HQ. AM and above can adjust within HQ limits. They appear as a collapsible script panel in the Action Guide.

| Objective | Example Talking Points (default — configurable by HQ) |
|-----------|------------------------------------------------------|
| เอาวันนัดชำระ | "สวัสดีครับ/ค่ะ คุณ [ชื่อ] ขณะนี้สัญญาของท่านจะครบกำหนดชำระในวันที่ [วันที่] ท่านสะดวกนัดวันชำระได้หรือไม่?" |
| แจ้งเตือนยืนยันนัดชำระ | "สวัสดีครับ/ค่ะ ขอยืนยันว่าพรุ่งนี้คือวันนัดชำระของท่าน ท่านยืนยันจะชำระตามนัดไหมครับ/ค่ะ?" |
| เก็บยอดตามนัดชำระ | "สวัสดีครับ/ค่ะ วันนี้คือวันที่ท่านนัดชำระไว้ ขอทราบว่าท่านชำระเรียบร้อยแล้วหรือยัง?" |
| ติดตามเข้มงวด | "สวัสดีครับ/ค่ะ บัญชีของท่านขณะนี้มีสถานะเกินกำหนดชำระ ขอนัดวันชำระโดยเร็วที่สุดได้หรือไม่?" |

**Configuration rule**: HQ defines default talking points per objective. AM+ can replace or append per branch variant. Structure (which objective gets a talking point) is fixed; content is fully configurable.

---

### Required Outcome Reminder

Before the CO can save a task outcome, the Required Outcome Reminder panel shows which fields are mandatory for the current objective:

| Objective | Required Fields on Outcome |
|-----------|--------------------------|
| เอาวันนัดชำระ — ได้วันนัดชำระ | วันนัดชำระ (date), ยอดคาดการณ์ (amount) |
| เก็บยอดตามนัดชำระ — ชำระตามยอดคาดการณ์ | ยอดที่ชำระ (amount paid), วันที่ชำระจริง |
| ติดตามเข้มงวด — ได้นัดชำระใหม่ | วันนัดชำระใหม่ (date), ยอดคาดการณ์ใหม่ |
| Any objective — ปฏิเสธการชำระ | เหตุผลที่ปฏิเสธ (reason), follow-up note (optional) |

Required fields are defined in the Template Library. CO cannot save until all required fields are filled.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Load with customer page | Action Guide panel must render as part of the customer page — no separate navigation or load |
| Real-time objective sync | Timing signals and talking points update immediately when objective changes |
| Configurability | All talking points and approach guidance must be editable by HQ (default) and AM+ (branch variant) via Playbook Template — no hardcoding |
| Graceful fallback | If no talking points are configured for an objective, panel collapses silently — does not block CO |
