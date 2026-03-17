# Capability: Work Queue

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Present all levels with a role-scoped work queue. Branch staff see a prioritized queue organized by event priority (P1–P4). AM+ see their responsible contract list (สัญญาที่อยู่ภายใต้การดูแลของพื้นที่) — escalated and manually-added contracts awaiting AM action. Both levels drill into a contract row to access the customer page.

## Why It Exists (First Principles)

- **Priority-First Focus**: The most urgent contracts (P1: due today / appointment today) must be visible and actionable immediately. Action type is secondary — urgency is not.
- **Context at a Glance**: A CO must be able to assess a contract's status (deadline, urgency, last contact, forecasted amount) without opening the record, enabling faster triage.
- **Accountability**: Each row surfaces who else is working the contract (`Other Person in Charge`), preventing duplicated contact and enabling handover.
- **Speed**: Collection officers must process 300–500 customers per day. The UI must minimize navigation, pre-load context, and provide quick outcome entry.

---

## Feature Inventory

| Feature | Level | Status | Description |
|---------|-------|--------|-------------|
| Collection Tab | Branch | Draft | P1–P4 priority queue for Active, Write-off, and Litigation contracts |
| Sales Tab | Branch | Draft | Insurance Renewal contracts — sorted by days to expiry |
| Offerings Tab | Branch | Draft | Top-up, Nano, and Insurance eligibility contracts — sorted by campaign priority |
| Contract Table View | Branch | Draft | Sortable table per tab/bucket with all key contract fields |
| Urgency Display | Branch | Draft | Surfaces `risk_level` (1–6) for Active portfolio contracts and `easiness_to_collect` (1–7) for Write-off contracts as the urgency value in the contract table; used for sort order within each priority bucket |
| Customer Page Drill-Through | Branch + AM+ | Draft | Clicking a contract row opens the customer page with collection log and notes |
| One-by-One Processing Mode | Branch | Draft | Primary mode: select contract row, view customer page, execute action, record outcome |
| Queue Overview Header | Branch | Draft | Top-level summary: total contracts today, completed, overdue |
| AM Responsible Contracts (สัญญาที่อยู่ภายใต้การดูแลของพื้นที่) | AM+ | Draft | AM's execution queue — escalated and manually-added contracts; AM chooses one of three actions per contract |

---

## Business Rules

### Queue Structure: Priority Buckets

The queue is organized by **priority**, not action type. Each priority bucket is a tab containing all contracts with active tasks at that level.

Priority is assigned by Work Queue based on the contract's current state at display time — it is **not** set during task creation.

| Priority | สถานะ Priority | Condition (evaluated from contract state) |
|----------|----------------|------------------------------------------|
| **P1** | สัญญาถึงวันครบกำหนดชำระ | `due_date = today` |
| **P1** | สัญญาที่มีนัดชำระในวันนี้ | `PTP_date = today` |
| **P2** | สัญญาใกล้วันครบกำหนดชำระ | `due_date − 7 days = today` |
| **P2** | แจ้งเตือนก่อนนัดชำระ | `PTP_date − 1 day = today` |
| **P2** | วันนัดเลยวันครบกำหนดชำระแล้ว แต่มีวันนัดชำระ | `due_date < today` AND `PTP_date` is set |
| **P3** | เลยวันครบกำหนดชำระ แต่ยังไม่มีนัดชำระ | `PTP_date` is null AND `due_date < today` |
| **P3** | ยังไม่ถึงวันครบกำหนดชำระ แต่ยังไม่มีนัดชำระ | `PTP_date` is null AND `due_date > today` (outside P1/P2 window) |
| **P3** | ผู้จัดการพื้นที่ ตีกลับ | Contract returned from AM's responsible contracts back to branch CO queue |
| **P4** | Write Off | `contract.status = write_off` |

> Conditions are evaluated top-down — first matching condition wins. A contract's P-group can shift between sessions as its state changes (e.g., PTP set → moves from P3 to P2/P1). Work Queue always reads current contract state.

### Contract Table Columns

When a CO opens a priority bucket, they see a table with the following columns:

| Column | Description |
|--------|-------------|
| Priority | P1 / P2 / P3 / P4 |
| Priority Name | Thai status label for the contract's current priority condition (e.g., สัญญาถึงวันครบกำหนดชำระ, แจ้งเตือนก่อนนัดชำระ) |
| Urgency | `the_collection_urgency` score for this contract |
| Objective | Current playbook objective (เอาวันนัดชำระ / แจ้งเตือนฯ / เก็บยอดฯ / ติดตามเข้มงวด) |
| Customer Name | Full name of the contract holder |
| Deadline (date) | The relevant action deadline — PTP_date if set, otherwise due_date. Text color indicates task status: 🔴 Red = task date was > 1 day ago AND CO has not yet followed up; 🔵 Blue = task date was > 1 day ago AND task is still incomplete; ⚫ Black = task date is today |
| Due Date | Contract due date (`due_date`) from Core Banking |
| Last Contact Date | Date of the most recent completed contact task |
| Last Contact Result | Outcome of the most recent contact (e.g., PTP, No Answer, Refused) |
| Payment Status | Current payment status of the contract |
| Forecasted Amount | ยอดตามคาดการณ์ — the expected payment amount for this collection cycle |
| Other Person in Charge | Other COs currently assigned to tasks on this contract |

**Default sort order within each bucket**: Overdue → highest `the_collection_urgency` score → earliest Deadline.

### Customer Page Drill-Through

Clicking any contract row opens the **customer page**, which contains:
- Full customer profile and contract summary
- **Collection log**: chronological record of all contact attempts, outcomes, notes, and action verifications
- Outcome entry form for the current active task

### One-by-One Processing Steps

1. Select a contract row from the priority bucket table
2. Customer page opens with full context pre-loaded
3. Execute the action (make call, conduct visit, complete admin task)
4. Record outcome from the action's defined outcome list
5. Fill conditional fields required by outcome (e.g., PTP → amount + date)
6. Save and return to the queue table; completed task removed from the bucket

### Urgency Scoring

The `Urgency` column in the contract table is sourced directly from the contract record — no calculation performed by Work Queue.

| Portfolio | Field | Scale |
|-----------|-------|-------|
| Active | `risk_level` | 1–6 (higher = more urgent) |
| Write-off | `easiness_to_collect` | 1–7 (higher = easier to collect) |

This value drives sort order within each priority bucket (see Default sort order above) and is visible to COs as a triage signal.

---

## Branch Queue: Sales Tab

Contains Insurance Renewal contracts. Separate top-level tab from Collection.

| Work Domain | Sub-type | Entry Condition | Urgency |
|-------------|----------|----------------|---------|
| Sales | Insurance Renewal | `insurance.status = active` AND `expiry_date` within renewal window | Days to expiry (fewer = higher urgency) |

> Priority bucket structure and sort rules for Sales tab: TBD.

---

## Branch Queue: Offerings Tab

Contains eligibility-based offering contracts. Separate top-level tab from Collection.

| Work Domain | Sub-type | Entry Condition | Urgency |
|-------------|----------|----------------|---------|
| Offerings | Top-up | Customer meets Core Banking top-up eligibility criteria | TBD (campaign priority) |
| Offerings | Nano | Customer meets Core Banking nano eligibility criteria | TBD (campaign priority) |
| Offerings | Insurance | Customer meets insurance product eligibility criteria | TBD (campaign priority) |

> Priority bucket structure and sort rules for Offerings tab: TBD.

---

## AM+ Queue: สัญญาที่อยู่ภายใต้การดูแลของพื้นที่

AM's execution queue — parallel to the Branch P1–P4 queue but scoped to AM level.

### Entry Routes

| Route | Trigger |
|-------|---------|
| Auto-escalated | "ส่งเรื่องให้ผู้จัดการพื้นที่" outcome from Playbook Engine — branch task closes, contract moves atomically to AM's queue; no new branch task created |
| Manually added | AM pulls a contract from Branch Collection Browse (การติดตามหนี้ในแต่ละสาขา) in Performance Dashboard |

### AM Actions per Contract

AM chooses one of three actions for each contract in their queue:

| Action | Thai Name | Description |
|--------|-----------|-------------|
| AM assign | มอบหมายงาน | Assign back to original branch, reassign to another branch in area, or handle directly as AM |
| Legal action | ดำเนินคดี | Escalate contract to legal proceedings |
| Find new address | หาที่อยู่ใหม่ | Initiate address search for uncontactable customer |

### AM Queue Columns

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

## NFRs

| NFR | Requirement |
|-----|-------------|
| Target throughput | Queue UX must support 300–500 task completions per CO per day |
| Pre-loaded context | Customer page and collection log must load without additional navigation steps |
| Table performance | Contract table must render within 2 seconds for up to 500 rows per bucket |
| Escalation atomicity | When "ส่งเรื่องให้ผู้จัดการพื้นที่" fires, contract must appear in AM's queue atomically — no task gap, no duplicate |
| AM queue real-time | AM's responsible contract list updates within 30 seconds of escalation or manual addition |
