# Capability: AM Worklist

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-05

---

## Business Function

Provide the Area Manager (AM) with an operational contract list for managing escalated and manually-added contracts — separate from the performance monitoring dashboard. AM Worklist is an execution queue: contracts enter it with no pending branch task, and AM decides the next action.

## Why It Exists (First Principles)

- **Escalation End-Point**: When collection exhausts branch-level attempts (ไม่ได้ทำ หมดอายุ on เอาวันนัดชำระ), the contract needs AM-level action. Without a dedicated list, escalated contracts have no clear ownership and risk being dropped.
- **AM Agency**: AMs can proactively pull any high-risk contract from any branch under their area into direct oversight — not only wait for auto-escalation.
- **Action-Oriented**: AM's Responsible Contracts is an execution queue, not a monitoring report. Each contract surfaces AM-only actions (assign, legal, find address) that drive the next step.
- **Separation of Concerns**: Monitoring (how branches are performing) belongs in Performance Dashboard. Execution (what AM must act on) belongs here.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Responsible Contracts List (สัญญาที่อยู่ภายใต้การดูแลของพื้นที่) | Draft | AM's active action queue — auto-escalated and manually added contracts. AM executes one of three actions per contract. |
| Branch Collection Browse (การติดตามหนี้ในแต่ละสาขา) | Draft | Full contract list per branch under AM's area. AM reads, filters, and pulls any contract into their responsible list. |

---

## Entry Routes

Contracts enter the AM Worklist via two routes:

| Route | Trigger | Notes |
|-------|---------|-------|
| **Auto-escalated** | "ส่งเรื่องให้ผู้จัดการพื้นที่" outcome — เอาวันนัดชำระ expires with no action (ไม่ได้ทำ หมดอายุ) | **No new task is created.** Contract moves atomically into AM's list. |
| **Manually added** | AM pulls a contract from การติดตามหนี้ในแต่ละสาขา | AM can pull any contract from any branch under their area at any time. |

---

## Business Rules

### AM Actions per Contract

When a contract is in AM's responsible list, AM chooses one of three actions:

| Action | Thai Name | Description |
|--------|-----------|-------------|
| AM assign | มอบหมายงาน | Assign back to original branch, reassign to another branch in area, or handle directly as AM |
| Legal action | ดำเนินคดี | Escalate contract to legal proceedings |
| Find new address | หาที่อยู่ใหม่ | Initiate address search for uncontactable customer |

---

### B2. สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ (AM's Responsible Contracts)

AM's active action queue — contracts with no pending branch task; AM owns the next decision.

| Column | Description |
|--------|-------------|
| ชื่อ-นามสกุล (ชื่อเล่น) | Customer full name and nickname |
| Due date | Relevant due date (color-coded: overdue = orange/red) |
| สถานะการจ่าย | Payment status badge |
| ยอดตามคาดการณ์ | Forecasted payment amount |
| วันที่ติดต่อล่าสุด | Date of most recent contact |
| ผลการติดต่อล่าสุด | Outcome of most recent contact |
| สาขาต้นทาง | Source branch |
| มอบหมายให้สาขา | Dropdown: assign to branch (or keep with AM) |
| หมายเหตุ | Free-text note |

---

### B3. การติดตามหนี้ในแต่ละสาขา (Branch Collection Browse)

Full contract list per branch under AM's area. AM reads, filters, and can pull any contract into their responsible list. Clicking a row opens the customer page (same drill-through as Work Queue).

**Filters**: Search by ชื่อ-นามสกุล / เลขที่สัญญา / เบอร์โทร / เลขโปรเจคติด / เลขบัตรประชาชน; filter by เลขแมนเอดิต, ถ่วตัวรอง.

| Column | Description |
|--------|-------------|
| ความเสี่ยง | Risk level (color-coded: เสี่ยงสูง red, เสี่ยงกลาง orange, เสี่ยงต่ำ green/blue) |
| ชื่อ-นามสกุล (ชื่อเล่น) | Customer full name and nickname |
| % ต่อพอร์ต | Contract weight as % of branch portfolio |
| Due date | Relevant due date |
| Action | Recommended action for current Objective |
| ผลลัพธ์ที่คาดหวัง | Current Objective (e.g., เอาวันนัดชำระ) |
| สถานการจ่าย | Payment status badge |
| ยอดตามคาดการณ์ | Forecasted payment amount |
| วันที่ติดต่อล่าสุด | Date of most recent contact |
| ผลการติดตามล่าสุด | Outcome of most recent contact |
| สาขา | Branch name |
| ผู้รับผิดชอบเพิ่มเติม | Additional CO(s) assigned |
| เรื่องกฎสัญญา | Compliance flag (⚠ if issue exists) |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Escalation atomicity | When "ส่งเรื่องให้ผู้จัดการพื้นที่" fires, contract must appear in AM's list atomically — no task gap, no duplicate |
| Branch list performance | การติดตามหนี้ในแต่ละสาขา must render within 2 seconds for full branch portfolio |
| Real-time updates | AM's responsible contract list updates within 30 seconds of escalation or manual addition |
