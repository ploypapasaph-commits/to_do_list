# Capability: Work Queue

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Present field staff with a prioritized work queue organized by event priority (P1–P4), where each priority bucket displays a sortable contract table. COs drill into a contract row to access the customer page with full collection history and notes.

## Why It Exists (First Principles)

- **Priority-First Focus**: The most urgent contracts (P1: due today / appointment today) must be visible and actionable immediately. Action type is secondary — urgency is not.
- **Context at a Glance**: A CO must be able to assess a contract's status (deadline, urgency, last contact, forecasted amount) without opening the record, enabling faster triage.
- **Accountability**: Each row surfaces who else is working the contract (`Other Person in Charge`), preventing duplicated contact and enabling handover.
- **Speed**: Collection officers must process 300–500 customers per day. The UI must minimize navigation, pre-load context, and provide quick outcome entry.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Priority Buckets (P1–P4) | Draft | Queue organized into four priority tabs; each tab shows a contract table |
| Contract Table View | Draft | Sortable table per priority bucket with all key contract fields |
| Customer Page Drill-Through | Draft | Clicking a contract row opens the customer page with collection log and notes |
| One-by-One Processing Mode | Draft | Primary mode: select contract row, view customer page, execute action, record outcome |
| Daily Contact Limit Enforcement | Draft | Auto-skip / flag contracts when customer's daily contact limit is reached |
| Queue Overview Header | Draft | Top-level summary: total contracts today, completed, overdue, contact-blocked |

---

## Business Rules

### Queue Structure: Priority Buckets

The queue is organized by **event priority**, not action type. Each priority bucket is a tab that contains a table of all contracts with active tasks at that priority level.

| Priority | Events | Meaning |
|----------|--------|---------|
| **P1** | สัญญาถึงวันครบกำหนดชำระ · สัญญาที่มีนัดชำระในวันนี้ | Due today / Appointment today — highest urgency |
| **P2** | สัญญาใกล้วันครบกำหนดชำระ · แจ้งเตือนก่อนนัดชำระ | Approaching due / Pre-appointment reminder |
| **P3** | ไม่มีวันนัดชำระ · ตัวที่หลุด | No appointment set / Missed commitment |
| **P4** | Write Off | Write-off portfolio contracts |

### Contract Table Columns

When a CO opens a priority bucket, they see a table with the following columns:

| Column | Description |
|--------|-------------|
| Priority | P1 / P2 / P3 / P4 |
| Customer Name | Full name of the contract holder |
| Deadline (date) | The relevant due date or appointment date |
| Urgency | `the_collection_urgency` score for this contract |
| Action | Recommended action from the current Objective (`โทร` / `ลงพื้นที่` / `Admin`) |
| Objective | Current playbook objective (เอาวันนัดชำระ / แจ้งเตือนฯ / เก็บยอดฯ / ติดตามเข้มงวด) |
| Payment Status | Current payment status of the contract |
| Forecasted Amount | ยอดตามคาดการณ์ — the expected payment amount for this collection cycle |
| Last Contact Date | Date of the most recent completed contact task |
| Last Contact Result | Outcome of the most recent contact (e.g., PTP, No Answer, Refused) |
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

### Daily Contact Limit Rules

- Maximum contacts per customer per day: configurable (default: 2)
- System enforces limit — task is flagged / auto-skipped if limit reached
- Visual indicator on the contract row: "⚠️ Contact limit reached"
- DaVinci owns contact count data and emits `ContactLimitReached` event; Sensei enforces the skip

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Target throughput | Queue UX must support 300–500 task completions per CO per day |
| Contact limit enforcement | System must not allow CO to initiate contact when limit reached |
| Pre-loaded context | Customer page and collection log must load without additional navigation steps |
| Table performance | Contract table must render within 2 seconds for up to 500 rows per bucket |
