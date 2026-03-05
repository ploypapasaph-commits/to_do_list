# ATLAS - Product Capabilities & Strategic Blueprint

**Role**: `@ARCHITECT`
**Purpose**: This document defines the **Capabilities ("What")** of the Sensei Service — the Centralized Branch Worklist & Task Tracker for field staff productivity and management.

> **Sensei (先生)** — The master who provides structure, guidance, and discipline. Sensei does not do the work — it orchestrates *how work is done* across branches, ensuring alignment with policy, visibility for leadership, and productivity for field staff.
>
> **Scope**: Sensei is a **centralized branch worklist**, not a universal workflow engine. Branch-native workflows (collection, renewal) use Sensei's Playbook Engine. Domain-specific workflows (loan underwriting, document verification) stay in their own systems but can **push tasks into Sensei** when they need branch action. Sensei tracks task execution and outcomes — it does not own or orchestrate upstream workflow logic.

---

## 1. Core Capability: Playbook Engine

**Goal**: Provide a structured, reusable system for defining multi-step collection strategies organized into a sequential 4-stage Objective Chain, triggered by priority-classified contract events, and configured per portfolio type and collection urgency.

### Why It Exists (First Principles)

*   **Policy Alignment Problem**: Thousands of branch staff across hundreds of branches must execute consistent strategies. Without a structured playbook, each branch invents its own approach, leading to inconsistent outcomes and compliance risk.
*   **Knowledge Codification**: Effective collection strategies are institutional knowledge. They must be captured as executable templates, not tribal knowledge.
*   **Urgency-Aware Execution**: Not all contracts are equal. The intensity of follow-up (timing, action type, retry count) must reflect how urgent collection is — driven by risk level and portfolio type.
*   **Adaptability**: HQ defines the default strategy per urgency tier, but local conditions require branch-level customization within guardrails.

---

### Configuration Ownership

All business logic components are **fully configurable** (add / adjust / change) by the appropriate role. The only fixed element is the **Objective Configurations structure** (its columns/schema), which is system-defined and immutable.

| Component | Add | Adjust | Change | Who |
|-----------|-----|--------|--------|-----|
| Priority Tiers | ✅ | ✅ | ✅ | HQ |
| Events per Priority | ✅ | ✅ | ✅ | HQ |
| `the_collection_urgency` mapping & thresholds | ✅ | ✅ | ✅ | HQ |
| Objectives (names, sequence, number of stages) | ✅ | ✅ | ✅ | HQ |
| Outcome Routing per Objective | ✅ | ✅ | ✅ | HQ |
| Playbook Templates (per portfolio_type × urgency_tier) | ✅ | ✅ | ✅ | HQ |
| Branch Variants | ✅ | ✅ | ✅ | AM and above |
| Objective Configurations **values** (timing, lifespan, action) | ✅ | ✅ | ✅ | HQ (defaults) / AM+ (within HQ limits) |
| Objective Configurations **structure** (columns/schema) | ❌ | ❌ | ❌ | System-defined — immutable |

> The Objective Configurations structure defines the **fields** each Objective must have: `วันที่สร้างงาน`, `อายุของงาน`, `Action ที่แนะนำ`, `จำนวนทำซ้ำ`. This schema is fixed. The **values** within those fields are fully configurable per role.

---

### Event Triggers by Priority

Every contract event entering Sensei is classified into one of four priority tiers. Priority determines queue ordering and which playbook objective is activated.

| Priority | Event | Meaning |
|----------|-------|---------|
| **P1** | สัญญาถึงวันครบกำหนดชำระ | Contract is at its due date today |
| **P1** | สัญญาที่มีนัดชำระในวันนี้ | Contract has a scheduled payment appointment today |
| **P2** | สัญญาใกล้วันครบกำหนดชำระ | Contract approaching due date (pre-due window) |
| **P2** | แจ้งเตือนก่อนนัดชำระ | Reminder before a scheduled payment appointment |
| **P3** | ไม่มีวันนัดชำระ | Contract has no payment appointment set |
| **P3** | ตัวที่หลุด | Contract missed a previous payment commitment |
| **P4** | Write Off | Contract classified as write-off; transferred to write-off portfolio |

---

### Collection Urgency Scoring

`the_collection_urgency` is calculated per contract and determines which playbook template configuration applies:

| Portfolio Type | Input Dimension | Scale | Priority Order | Interpretation |
|----------------|----------------|-------|----------------|----------------|
| **Active** | `risk_level` | 1 – 6 | Higher score first | 1 = lowest risk; 6 = highest risk / most delinquent — collect highest risk first |
| **Write-Off** | `easiness_to_collect` | 1 – 7 | Higher score first | 7 = easiest to collect; 1 = hardest to collect — collect easiest first to maximize recovery rate |

**Default sort order**: Within the same priority tier, contracts are sorted by **descending score** for both portfolios. For Write-Off, score 7 = highest collection priority (easiest to recover); score 1 = lowest (hardest to recover). Inverse of Active's risk interpretation, but follows the same descending sort rule.

Higher urgency = more aggressive timing, lower retry tolerance, earlier escalation to Visit. Mapping from scores to urgency tiers is configured by HQ in the Template Library.

---

### Playbook Objective Chain

Collection for a contract follows a sequential chain of four Objectives. Each Objective is a self-contained playbook stage. The chain terminates only when an End Condition is met.

```
                         risk (contract enters)
                               │
          ┌────────────────────▼─────────────────────┐
          │         เอาวันนัดชำระ                     │
          │         DL: d  │  attempts: 3             │
          └──┬──────────┬──────────┬──────────────────┘
             │          │          │          │
          success   do not      hard       no action
             │       success    reject          │
             │          │          │            ▼
             │          └────┬─────┘    🔒 ส่งเรื่องให้ผู้จัดการพื้นที่
             │               ▼            (AM assign / legal action /
             │    ┌──────────────────────┐  find new address)
             │    │   ติดตามเข้มงวด      │◀──────────────────────────┐
             │    │   DL: d+3 | att: 1   │                           │
             │    │   DL: PTP+1 | att: 1 │                           │
             │    └──────┬──────────┬────┘                           │
             │           │          │    │                            │
             │        get PTP   paid full  no                        │
             │           │          │    └──── re-queue ─────────────┘
             │           └──┐       ▼
             │              │     END ✅
             ▼              │
  ┌──────────────────────┐  │
  │  แจ้งเตือนยืนยัน     │  │ (loop: get PTP in strict mode
  │  นัดชำระ             │  │  re-enters at PTP+1)
  │  DL: PTP-1 | att: 3  │  │
  └──────────┬───────────┘  │
             │ สัญญาว่าจะชำระ │
  ┌──────────▼──────────────▼┐
  │  เก็บยอดตามนัดชำระ       │  ← paid full? check
  │  DL: PTP  │  att: 3      │
  └──────────┬───────────────┘
             │
         paid full
             ▼
           END ✅
```

---

### Objective Configurations

**Ownership split**: HQ defines Objective names, sequence, and default parameter values. Supervisor (AM and above) can adjust `วันที่สร้างงาน`, `อายุของงาน`, and `Action ที่แนะนำ` within HQ-set limits per urgency tier.

| Objective | DL (Deadline) | วันที่สร้างงาน (default) | อายุของงาน (default) | Action ที่แนะนำ (default) | จำนวนทำซ้ำ |
|-----------|--------------|------------------------|---------------------|--------------------------|------------|
| เอาวันนัดชำระ | d (due date) | ก่อน due 7 วัน | 7 วัน | 📞 โทร | 3 ครั้ง |
| แจ้งเตือนยืนยันนัดชำระ | PTP − 1 วัน | ก่อนวันนัดชำระ 1 วัน | ภายในวัน | 📞 โทร | 3 ครั้ง |
| เก็บยอดตามนัดชำระ | PTP (วันนัดชำระ) | วันนัดชำระ | ภายในวัน | 📞 โทร | 3 ครั้ง |
| ติดตามเข้มงวด — รอบแรก | d + 3 | วันที่ list ขึ้น | 3 วัน | 🏠 ลงพื้นที่ | 1 ครั้ง |
| ติดตามเข้มงวด — หลังได้ PTP | PTP + 1 | วันถัดจากวันนัดใหม่ | ภายในวัน | 📞 โทร | 1 ครั้ง |

---

### Outcome Routing per Objective

#### เอาวันนัดชำระ
*(DL: d — due date | attempts: 3)*

| ผลลัพธ์ | Diagram Label | ขั้นตอนถัดไป |
|--------|--------------|-------------|
| ได้วันนัดชำระ | success | → แจ้งเตือนยืนยันนัดชำระ (DL: PTP−1) |
| ไม่ได้วันนัดชำระ | do not success | → ติดตามเข้มงวด (DL: d+3) |
| ปฏิเสธการชำระ | hard reject | → ติดตามเข้มงวด (DL: d+3) |
| ไม่ได้ทำ (หมดอายุ) | no action | → 🔒 ส่งเรื่องให้ผู้จัดการพื้นที่ — **no new task created**; contract moves into AM's responsible list (สัญญาที่อยู่ภายใต้การดูแลของพื้นที่) |

#### แจ้งเตือนยืนยันนัดชำระ
*(DL: PTP−1 | attempts: 3)*

| ผลลัพธ์ | ขั้นตอนถัดไป |
|--------|-------------|
| สัญญาว่าจะชำระ | → เก็บยอดตามนัดชำระ (DL: PTP) |
| ปฏิเสธการชำระ | → ติดตามเข้มงวด (DL: d+3) |
| นัดวันชำระใหม่ | → แจ้งเตือนยืนยันนัดชำระ (re-trigger on new date) |
| ติดต่อไม่ได้ | → ติดตามเข้มงวด (DL: d+3) |

#### เก็บยอดตามนัดชำระ
*(DL: PTP — payment date | attempts: 3 — "paid full?" check)*

| ผลลัพธ์ | Diagram Label | ขั้นตอนถัดไป |
|--------|--------------|-------------|
| ชำระตามยอดคาดการณ์ | paid full | → สิ้นสุดการตาม ✅ |
| ไม่ชำระ / ชำระไม่ครบ | do not get payment / not full | → ติดตามเข้มงวด (DL: d+3) |

#### ติดตามเข้มงวด
*(รอบแรก DL: d+3 | attempts: 1 → หลังได้ PTP ใหม่ DL: PTP+1 | attempts: 1)*

| ผลลัพธ์ | Diagram Label | ขั้นตอนถัดไป |
|--------|--------------|-------------|
| ได้นัดชำระใหม่ | get PTP | → เก็บยอดตามนัดชำระ (re-enter at DL: PTP+1) |
| ชำระตามยอดคาดการณ์ | paid full | → สิ้นสุดการตาม ✅ |
| ไม่ได้ผล | no | → ติดตามเข้มงวด (re-queue, loop) |

---

### End Conditions

| Condition | Trigger | Status |
|-----------|---------|--------|
| ชำระตามยอดตามคาดการณ์ | CO records full payment of forecasted amount | ✅ End Chain (Success) |
| Restructure | Contract restructured | ✅ End Chain (Success) — TBD |
| Repossession | Asset repossessed | ✅ End Chain (Closed) — TBD |

---

### Playbook Hierarchy

```
  ┌──────────────────────────────────────────────┐
  │   System Template (HQ-owned)                 │
  │   Keyed by: portfolio_type × urgency_tier    │
  └────────────┬─────────────────────────────────┘
               │ fork
  ┌────────────▼─────────────────────────────────┐
  │   Branch Variant                             │
  │   Supervisor customizes within edit rules    │
  └──────────────────────────────────────────────┘
```

*   **System Templates** are blueprints owned by HQ, organized by `portfolio_type × urgency_tier`. The same template can be reused across multiple combinations — there is no strict one-to-one requirement. **Action types are set by HQ and cannot be changed by branches. Timing of actions and deadlines can be adjusted by AM role and above.**
*   **Branch Variants** are created when an AM (or above) modifies a system template. Tracks which template version it was forked from.
*   **Compliance-Locked Steps** (🔒): Cannot be removed, reordered past their boundary, or have outcome transitions modified by supervisors.
*   **Template Version Sync**: HQ publishes a new template version → branches with variants are notified to review and merge.

### Supervisor Edit Rules

| Allowed | Not Allowed |
|---------|-------------|
| Drag & drop to reorder steps within an Objective | Delete 🔒 locked steps |
| Drag outcome transitions to different target steps | Edit System Templates directly |
| Add optional steps | Remove audit trail / compliance logging |
| Add/remove outcomes on non-locked steps | Modify outcome transitions on 🔒 locked steps |
| Adjust timing within HQ-set limits | Reorder locked steps past compliance boundary |
| Change assignee rules | Modify urgency-tier assignments |
| Set retry limits on outcomes | Bypass publishing workflow |
| Remove non-locked steps | |

---

## 2. Core Capability: Task Engine (Centralized Branch Worklist)

**Goal**: Provide a unified task tracking system for branches — aggregating work from Sensei's own playbooks, external service requests, and supervisor-created tasks into a single worklist with lifecycle tracking, SLA enforcement, and outcome recording.

### Why It Exists (First Principles)

*   **Accountability**: Every interaction with a customer must be recorded. Without a task record, there is no audit trail of attempts, outcomes, or compliance adherence.
*   **Throughput Visibility**: Management cannot measure what is not tracked. Tasks provide the atomic unit of measurement for staff productivity.
*   **Automation**: Tasks generated automatically from events and playbooks eliminate reliance on supervisors manually assigning work.
*   **Unified Inbox**: Branch staff work across multiple products and processes. Without a centralized worklist, they must switch between systems. Sensei presents all branch work in one place regardless of which system originated it.

### Architectural Position

Sensei is a **task tracker, not a workflow orchestrator**. Domain-specific workflows (loan underwriting in Onigiri, document verification in Matcha) stay inside their own bounded contexts with their own state machines and business rules. When these systems need branch action, they push a task into Sensei. Sensei tracks execution and reports outcomes back.

```
  ┌────────────────┐       task event       ┌─────────────────────────────┐
  │ Onigiri (LOS)  │ ─────────────────▶ │                             │
  └────────────────┘                       │                             │
                                       │    SENSEI                    │
  ┌────────────────┐       task event       │    Centralized Branch        │
  │ Matcha (Docs)  │ ─────────────────▶ │    Worklist                  │
  └────────────────┘                       │                             │
                                       │    📞 Calls (22)              │
  ┌────────────────┐       task event       │    🏠 Visits (4)              │
  │ Sensei's own   │ ─────────────────▶ │    📋 Admin (6)               │
  │ Playbooks      │                       │    📦 External Requests (3)   │
  └────────────────┘                       │                             │
                          ◀────────── │  completion event            │
                                       └─────────────────────────────┘
```

### Task Lifecycle

```
  ┌──────────┐   assign   ┌──────────┐   start   ┌──────────┐   record    ┌──────────┐
  │ CREATED  │ ─────────▶ │ ASSIGNED │ ────────▶ │ ACTIVE   │ ──────────▶ │ CLOSED   │
  └──────────┘            └──────────┘           └──────────┘            └──────────┘
       │                       │                      │
       │                       │ reassign             │ escalate
       │                       ▼                      ▼
       │                  ┌──────────┐          ┌──────────┐
       │                  │ ASSIGNED │          │ ESCALATED│
       │                  │ (new CO) │          └──────────┘
       │                  └──────────┘
       │
       │ expire (SLA)
       ▼
  ┌──────────┐
  │ OVERDUE  │ → surfaces in supervisor exception panel
  └──────────┘
```

### Task Sources

Tasks enter the worklist from four distinct sources:

| Source | How Tasks Are Created | Example |
|--------|----------------------|---------|
| `playbook_step` | Automatically by Sensei's Playbook Engine when a playbook is instantiated for a customer | Delinquency playbook generates "Call customer" task |
| `event_rule` | Automatically by Sensei's event rules matching DaVinci/Core Banking events | `DPD > 30` event → Call task |
| `manual` | Supervisor creates a one-off task in the UI | "Visit customer for special follow-up" |
| `external` | **External service** pushes a task via event/API when it needs branch action | Onigiri needs docs collected; Matcha needs physical ID |

### External Task Creation Contract

When an external service needs branch action, it publishes a `TaskCreationRequest` event that Sensei consumes:

```
TaskCreationRequest {
  customer_id        // DaVinci customer ID (required)
  action_type        // call | visit | admin | review (required)
  title              // Human-readable, e.g., "เก็บเอกสาร KYC" (required)
  description        // Context for the CO (optional)
  priority           // high | normal | low (default: normal)
  sla_deadline       // ISO timestamp (optional)
  source_system      // "onigiri" | "matcha" | etc. (required)
  source_ref_id      // e.g., loan_application_id (required)
  source_callback    // event topic for completion notification (optional)
  metadata           // opaque JSON — Sensei stores but does not interpret (optional)
}
```

Sensei creates a task from this event using the same lifecycle as any other task. The CO sees it in their worklist alongside playbook-generated tasks. **Sensei does not need domain knowledge** about the upstream workflow — it just presents the task and records the outcome.

### Task Completion Feedback

When a task is closed, Sensei publishes a `TaskCompleted` event:

```
TaskCompleted {
  task_id             // Sensei task ID
  customer_id         // DaVinci customer ID
  action_type         // what was done
  outcome             // typed outcome (e.g., PTP, Completed, Refused)
  outcome_data        // structured data (e.g., PTP amount + date)
  completed_by        // CO identifier
  completed_at        // ISO timestamp
  source_system       // original source (for routing)
  source_ref_id       // original reference (for correlation)
  source_callback     // echo back for routing
  metadata            // echo back opaque metadata
}
```

The originating system consumes this event and advances its own workflow accordingly. For example: Onigiri receives "docs collected" and moves the loan application to the next underwriting stage. **Sensei is not aware of what happens next** — it only reports that the task was done.

### Task Properties

*   **Source**: `playbook_step`, `event_rule`, `manual`, `external`
*   **Action Type**: `Call`, `Visit`, `Admin`, `Review`, `Notify`
*   **Priority**: Derived from DPD bucket, playbook urgency, SLA proximity, or external request
*   **Outcome**: Typed per action (e.g., Call outcomes: `PTP`, `No Answer`, `Refused`, `Callback`, `Wrong Number`)
*   **SLA**: Configurable deadline. Breached tasks surface in supervisor exception panel.
*   **Linked Entities**: `customer_id` (DaVinci), `contract_id`, `playbook_id`, `playbook_step_id`, `source_system`, `source_ref_id`

### Event-Driven Task Generation (Internal)

*   **Event Rules**: System or branch-defined rules that create tasks from external events:
    *   `Delinquency > 30 DPD` → Create Call task from "Delinquency Recovery" playbook
    *   `Insurance expiry within 15 days` → Create Call task from "Insurance Renewal" playbook
    *   `KYC expiry within 30 days` → Create Admin task for document collection
*   **Event Sources**: DaVinci (customer events), Core Banking (delinquency events), Policy Admin (renewal events)

### Supervisor Task Controls

*   **Reassign**: Move a task from one CO (Collection Officer) to another
*   **Override Playbook Step**: Skip or insert a step for a specific customer's playbook instance
*   **Add Manual Task**: Create a one-off task not tied to any playbook
*   **Bulk Operations**: Reassign all tasks from one CO to another (e.g., staff absence)
*   **View External Context**: For `external` tasks, a link to the source system is provided for supervisor reference

---

## 3. Core Capability: Work Queue (High-Volume Processing)

**Goal**: Present field staff with a prioritized work queue organized by event priority (P1–P4), where each priority bucket displays a sortable contract table. COs drill into a contract row to access the customer page with full collection history and notes.

### Why It Exists (First Principles)

*   **Priority-First Focus**: The most urgent contracts (P1: due today / appointment today) must be visible and actionable immediately. Action type is secondary — urgency is not.
*   **Context at a Glance**: A CO must assess a contract's status without opening the record, enabling faster triage across 300–500 contracts per day.
*   **Accountability**: Each row surfaces who else is working the contract, preventing duplicated contact and enabling handover.

### Queue Structure: Priority Buckets

The queue is organized by **event priority**, not action type. Each priority bucket is a tab containing a table of all contracts with active tasks at that priority level.

| Priority | Events | Meaning |
|----------|--------|---------|
| **P1** | สัญญาถึงวันครบกำหนดชำระ · สัญญาที่มีนัดชำระในวันนี้ | Due today / Appointment today |
| **P2** | สัญญาใกล้วันครบกำหนดชำระ · แจ้งเตือนก่อนนัดชำระ | Approaching due / Pre-appointment reminder |
| **P3** | ไม่มีวันนัดชำระ · ตัวที่หลุด | No appointment set / Missed commitment |
| **P4** | Write Off | Write-off portfolio contracts |

### Contract Table Columns

When a CO opens a priority bucket, they see a sortable table with the following columns:

| Column | Description |
|--------|-------------|
| Priority | P1 / P2 / P3 / P4 |
| Customer Name | Full name of the contract holder |
| Deadline (date) | The relevant due date or appointment date |
| Urgency | `the_collection_urgency` score for this contract |
| Action | Recommended action from the current Objective (`โทร` / `ลงพื้นที่` / `Admin`) |
| Objective | Current playbook objective (เอาวันนัดชำระ / แจ้งเตือนฯ / เก็บยอดฯ / ติดตามเข้มงวด) |
| Payment Status | Current payment status of the contract |
| Forecasted Amount | ยอดตามคาดการณ์ — expected payment amount for this collection cycle |
| Last Contact Date | Date of the most recent completed contact task |
| Last Contact Result | Outcome of the most recent contact (e.g., PTP, No Answer, Refused) |
| Other Person in Charge | Other COs currently assigned to tasks on this contract |

**Default sort order within each bucket**: Overdue → highest `the_collection_urgency` → earliest Deadline.

### Customer Page Drill-Through

Clicking any contract row opens the **customer page**, which contains:
- Full customer profile and contract summary
- **Collection log**: chronological record of all contact attempts, outcomes, notes, and action verifications
- Outcome entry form for the current active task

### Primary Processing: One-by-One

1. Select a contract row from the priority bucket table
2. Customer page opens with full context and collection log pre-loaded
3. Execute action (make call, conduct visit, complete admin task)
4. Record outcome and fill conditional fields (e.g., PTP → amount + date)
5. Save and return to table; completed task removed from the bucket

### Extended Capability: Rapid-Fire Mode

Accelerated mode for experienced COs on high-volume buckets:
- Triggered by "Start Queue ▶" on a priority bucket
- One contract at a time, customer page pre-loaded
- Large tap targets for common outcomes; "Save → Next ▶" auto-advances
- Progress bar: "X of Y"; real-time contact limit display
- Exitable at any time to return to standard table view

### Key Business Rules

*   **Daily Contact Limit**: Max contacts per customer per day (default: 2). Contract row flagged "⚠️ Contact limit reached" when exceeded; task auto-skipped.
*   **Supervisor Message**: Daily motivational/tactical message from supervisor displayed at queue top

---

## 4. Core Capability: Performance & Visibility Dashboard

**Goal**: Provide a consistent dashboard structure across two access levels — **Branch** (all positions within the branch) and **Supervisor** (AM and above). Both levels share the same layout pattern — home dashboard, collection list, performance summary — with content scoped to their level.

### Why It Exists (First Principles)

*   **Management by Exception**: Both branch and AM levels need surfaced priorities scoped to their accountability.
*   **Area-Level Accountability**: AMs oversee multiple branches and need cross-branch performance, DPD movement, and their own escalated contract list in one place.
*   **Staff Motivation**: Transparent personal metrics and gamified rankings drive healthy competition at branch level.
*   **Accountability Chain**: Branch → Area → Region aggregation enables consistent performance reporting upward.

### Dashboard Structure (Shared Pattern)

| Section | Branch Level | Supervisor Level (AM+) |
|---------|-------------|------------------------|
| **Summary Widget** | งานที่ต้องจัดการ — active task count | งานที่พื้นที่ต้องจัดการ — AM contract count |
| **Work Setting** | การเรียงลำดับงาน / Strategy | การเรียงลำดับงาน / Strategy |
| **Tracking Table** | Team workload per CO | Branch performance per branch (daily + weekly) |
| **Collection List** | Work Queue (priority-based) | → AM Worklist (Section 6): สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ + การติดตามหนี้ในแต่ละสาขา |
| **Performance Summary** | Daily/weekly/monthly scorecard + leaderboard | Area targets + DPD movement |

---

### A. Branch Level (All Branch Positions)

#### Branch Home Dashboard

| Component | Description |
|-----------|-------------|
| **งานที่ต้องจัดการ** | Active task count; click → Work Queue |
| **การตั้งค่าการทำงาน** | Sort order and strategy settings |
| **ติดตามผลการทำงานของทีม** | Per-CO: queue size, completed, rate, PTP, exception flags |
| **Exception Panel** | Idle Staff · Low Performance · Contact Blocked · Playbook Stuck · SLA Breach |
| **Contact Compliance** | Real-time team compliance status |

#### Branch Performance Summary

| Component | Purpose |
|-----------|---------|
| Daily Scorecard | Team metrics: today / this week / this month / vs. target |
| Active Playbooks Panel | Per-playbook: case count, on-track / at-risk / failed / succeeded |
| Staff Self-Service Metrics | Personal: tasks completed, PTP rate, visit success, SLA compliance |
| Monthly Objectives Tracker | Succeeded / in-progress / failed objectives per CO |
| Branch Rank & Leaderboard | 🥇🥈🥉 composite score: completion + PTP + SLA + contact compliance |

---

### B. Supervisor Level (AM and Above)

#### B1. Supervisor Home Dashboard (หน้าหลัก)

| Component | Description |
|-----------|-------------|
| **งานที่พื้นที่ต้องจัดการ** | Count of contracts in AM's responsible list; click → AM contract list |
| **การตั้งค่าการทำงาน** | Sort order and strategy settings |
| **ติดตามผลการทำงานของสาขา** | Branch performance — วันนี้ and สัปดาห์นี้; ติดตามหนี้ + เสนอขาย per branch |
| **ภาพรวมการทำงานประจำวันนี้** | Daily: ยอดสินเชื่อ and ยอดการขาย (ส่วนต่าง / ทำได้ / เป้าหมาย) |
| **ภาพรวมการทำงานเดือนนี้** | Monthly cumulative vs. target |
| **การไหลของ DPD** | DPD movement (C to X, X to 30) — current and prior month |

#### B2 & B3: AM Contract Lists

Moved to **AM Worklist** capability — see [Section 6](#6-core-capability-am-worklist).

---

## 5. Core Capability: Action Guide

**Goal**: Surface per-contract action intelligence directly on the Customer Page — telling COs **when to act** (timing signals) and **how to act** (approach guidance based on contract type, portfolio, urgency tier, and current objective stage).

### Why It Exists (First Principles)

- **300–500 contracts per day**: COs cannot recall the right approach for every contract. Embedded guidance removes cognitive load and reduces errors.
- **Contract type matters**: A risk_level 6 active portfolio contract needs a different approach than an easiness_to_collect 7 write-off contract.
- **Objective stage context**: The same outcome means different things at เอาวันนัดชำระ vs. ติดตามเข้มงวด — guidance must reflect the CO's position in the chain.
- **Consistency**: Embedded guidance enforces uniform playbook execution across all COs and branches.

### Where It Appears

| Location | Content |
|----------|---------|
| Customer Page — Action Guide panel | Full Timing Signal Panel + Action Approach Guide + Talking Points + Required Outcome Reminder |
| Rapid-Fire Mode — side panel | Condensed timing signal + talking points + required outcome fields |
| Work Queue table | Key signals already visible as columns: Priority, Urgency, Action, Objective |

### Timing Signal Panel

Rendered at the top of the Action Guide panel. Derived in real-time from task and contract data.

| Signal | Source | Display |
|--------|--------|---------|
| Priority Tier | Event classification | P1 / P2 / P3 / P4 badge (color-coded) |
| Deadline Countdown | DL from current objective | "X วันถึง Deadline" — red if ≤ 1 day |
| Urgency Score | `the_collection_urgency` | Score + tier label (e.g., "เร่งด่วนสูง") |
| Days Since Last Contact | Last completed contact task | "ติดต่อล่าสุด X วันที่แล้ว" — flagged if gap > threshold |
| Objective Stage | Current position in chain | Step indicator: เอาวันนัดชำระ → แจ้งเตือน → เก็บยอด → ติดตามเข้มงวด |
| Attempts Remaining | จำนวนทำซ้ำ remaining | "เหลือ X ครั้ง" — shown when attempts are limited |

### Action Approach Guide

Recommended approach varies by **portfolio type × current objective × urgency tier**:

#### Active Portfolio (risk_level 1–6)

| Objective | risk_level | Recommended Approach |
|-----------|-----------|----------------------|
| เอาวันนัดชำระ | 1–3 | โทรแจ้งกำหนดชำระ, ขอยืนยันวันนัด — tone: friendly reminder |
| เอาวันนัดชำระ | 4–6 | โทรเน้นความสำคัญ, escalate to visit if no answer after 2 attempts |
| เก็บยอดตามนัดชำระ | 1–3 | โทรตรวจสอบการชำระ, ยืนยันยอด |
| เก็บยอดตามนัดชำระ | 4–6 | โทรพร้อมแจ้งผลกระทบ, เสนอ restructure ถ้าจำเป็น |
| ติดตามเข้มงวด | 1–3 | โทร + เตรียม visit หากไม่ได้ผล |
| ติดตามเข้มงวด | 4–6 | ลงพื้นที่เป็นหลัก, ประสาน AM ถ้าไม่ได้ผล |

#### Write-Off Portfolio (easiness_to_collect 1–7)

| Objective | easiness_to_collect | Recommended Approach |
|-----------|--------------------|-----------------------|
| เอาวันนัดชำระ | 5–7 (ง่าย) | โทรเสนอข้อตกลงการชำระ — tone: cooperative |
| เอาวันนัดชำระ | 1–4 (ยาก) | ลงพื้นที่ + ประสาน AM สำหรับ legal action หรือ find new address |
| ติดตามเข้มงวด | 5–7 | โทรยืนยันข้อตกลง, เร่งปิด |
| ติดตามเข้มงวด | 1–4 | เตรียม legal action route, ส่ง AM |

### Contract Type Context Card

Collapsible card on the Customer Page alongside the Action Guide panel.

| Field | Description |
|-------|-------------|
| Portfolio Type | Active / Write-Off badge |
| Risk / Urgency Tier | Score + label (e.g., risk_level 5 — เสี่ยงสูง) |
| Payment Behavior | Pattern: paid on time / consistently late / never paid (from history) |
| Last 3 Contact Results | Most recent 3 outcomes with date |
| PTP Success Rate | % of past PTPs that resulted in actual payment |
| DPD Current | Current days-past-due |

### Talking Points Engine

Talking points defined per **objective + portfolio_type** in the Playbook Template by HQ. AM+ can adjust per branch variant.

| Objective | Default Talking Points (configurable) |
|-----------|--------------------------------------|
| เอาวันนัดชำระ | "คุณ [ชื่อ] สัญญาของท่านจะครบกำหนดชำระวันที่ [วันที่] ท่านสะดวกนัดวันชำระได้หรือไม่?" |
| แจ้งเตือนยืนยันนัดชำระ | "ขอยืนยันว่าพรุ่งนี้คือวันนัดชำระของท่าน ท่านยืนยันจะชำระตามนัดไหมครับ/ค่ะ?" |
| เก็บยอดตามนัดชำระ | "วันนี้คือวันที่ท่านนัดชำระไว้ ขอทราบว่าท่านชำระเรียบร้อยแล้วหรือยัง?" |
| ติดตามเข้มงวด | "บัญชีของท่านขณะนี้มีสถานะเกินกำหนดชำระ ขอนัดวันชำระโดยเร็วที่สุดได้หรือไม่?" |

### Required Outcome Reminder

Fields the CO must complete before saving — sourced from Template Library definitions.

| Objective + Outcome | Required Fields |
|--------------------|----------------|
| เอาวันนัดชำระ — ได้วันนัดชำระ | วันนัดชำระ (date), ยอดคาดการณ์ (amount) |
| เก็บยอดตามนัดชำระ — ชำระตามยอดคาดการณ์ | ยอดที่ชำระ, วันที่ชำระจริง |
| ติดตามเข้มงวด — ได้นัดชำระใหม่ | วันนัดชำระใหม่, ยอดคาดการณ์ใหม่ |
| Any — ปฏิเสธการชำระ | เหตุผลที่ปฏิเสธ (required), follow-up note (optional) |

---

## 6. Core Capability: AM Worklist

**Goal**: Provide the Area Manager (AM) with an operational contract list for managing escalated and manually-added contracts — separate from the performance monitoring dashboard.

### Why It Exists (First Principles)

- **Escalation End-Point**: When collection exhausts branch-level attempts (ไม่ได้ทำ หมดอายุ on เอาวันนัดชำระ), the contract needs AM-level action. Without a dedicated list, escalated contracts have no clear ownership.
- **AM Agency**: AMs can proactively pull any high-risk contract from any branch into their direct oversight — not only wait for auto-escalation.
- **Action-Oriented**: AM's Responsible Contracts is an execution queue, not a monitoring report. Each contract surfaces AM-only actions.
- **Separation of Concerns**: Monitoring belongs in Performance Dashboard. Execution belongs in AM Worklist.

### Entry Routes

| Route | Trigger |
|-------|---------|
| **Auto-escalated** | "ส่งเรื่องให้ผู้จัดการพื้นที่" outcome on เอาวันนัดชำระ expiry — **no new task created** |
| **Manually added** | AM pulls any contract from การติดตามหนี้ในแต่ละสาขา at any time |

### AM Actions per Contract

| Action | Thai Name | Description |
|--------|-----------|-------------|
| AM assign | มอบหมายงาน | Assign back to original branch, reassign to another branch, or handle directly |
| Legal action | ดำเนินคดี | Escalate contract to legal proceedings |
| Find new address | หาที่อยู่ใหม่ | Initiate address search for uncontactable customer |

### B2. สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ (AM's Responsible Contracts)

AM's active action queue — contracts with no pending branch task; AM owns the next decision.

| Column | Description |
|--------|-------------|
| ชื่อ-นามสกุล (ชื่อเล่น) | Customer full name and nickname |
| Due date | Relevant due date (overdue = orange/red) |
| สถานะการจ่าย | Payment status badge |
| ยอดตามคาดการณ์ | Forecasted payment amount |
| วันที่ติดต่อล่าสุด | Date of most recent contact |
| ผลการติดต่อล่าสุด | Outcome of most recent contact |
| สาขาต้นทาง | Source branch |
| มอบหมายให้สาขา | Dropdown: assign to branch (or keep with AM) |
| หมายเหตุ | Free-text note |

### B3. การติดตามหนี้ในแต่ละสาขา (Branch Collection Browse)

Full contract list per branch. AM reads, filters, and can pull any contract into their responsible list. Clicking a row opens the customer page.

**Filters**: Search by ชื่อ-นามสกุล / เลขที่สัญญา / เบอร์โทร / เลขโปรเจคติด / เลขบัตรประชาชน; filter by เลขแมนเอดิต, ถ่วตัวรอง.

| Column | Description |
|--------|-------------|
| ความเสี่ยง | Risk level (color-coded) |
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

## Resolved Design Decisions (ADR Style)

| # | Decision | Context | Consequence |
|---|----------|---------|-------------|
| D1 | **Vertical step pipeline, not Kanban board** | Playbook steps are strictly sequential; order matters. | Simpler mental model for supervisors. Drag & drop is vertical reordering, not category movement. |
| D2 | **Branch-variant model, not in-place editing** | Supervisors need flexibility, but HQ needs control over the canonical strategy. | Supervisors fork a system template into a branch variant. HQ can push template updates; branches decide whether to merge. Adds version management complexity but prevents policy drift. |
| D3 | **Compliance-locked steps** | Certain steps (reporting, supervisor notification) are legally or operationally mandated. | Locked steps cannot be removed or reordered past boundaries. Reduces supervisor flexibility slightly but ensures compliance. Reversible: HQ can unlock a step if policy changes. |
| D4 | **Linear default + outcome routing (Option C)** | Pure linear sequences can't handle failed conditions (No Answer vs PTP lead to different next steps). Full flowcharts are too complex for supervisors. | Each step carries outcome transition rules that override the default next step. Visual remains a vertical pipeline; outcomes appear as a sub-list per step. Supervisors drag outcome targets to different steps. Balances expressiveness with simplicity. |
| D5 | **Task Engine is event-driven, not batch-scheduled** | Branch operations are real-time — delinquency changes, payments, and renewals happen continuously. | Tasks are created from events (via DaVinci), not nightly batch jobs. Higher infrastructure complexity but enables same-day response. |
| D6 | **One-by-one as primary, rapid-fire as extended** | COs handle 300-500 customers/day but complex cases need full context. | One-by-one processing is the default. Rapid-fire is an optional accelerated mode for experienced COs. Prevents mistakes from rushing while enabling throughput for skilled users. |
| D7 | **Gamified leaderboard** | Staff motivation in high-volume roles benefits from visible competition. | Leaderboard with medals drives engagement. Risk: may create unhealthy competition — mitigated by combining rank with supervisor feedback and collaborative metrics. |
| D8 | **BOS log-fact check at task generation, not event subscription** | Contact limit enforcement must happen at task creation time — not reactively after an event arrives. DaVinci owns the cross-product contact count; BOS owns the collection note log used for real-time frequency checks. | Sensei queries the BOS collection note log before generating each contact task. If the customer's daily contact count ≥ limit, the task is not created. No dependency on ContactLimitReached or ContactLimitApproaching events. DaVinci still receives ContactRecorded events from Sensei to maintain the centralized cross-product count. BOS collection note log existence and API contract is TBD — critical open dependency. |
| D9 | **Trust-but-verify via 3CX cross-check** | COs could fabricate outcomes (record "called" without calling). Direct blocking would slow down legitimate work. | Sensei cross-references 3CX call logs after the fact. Flags discrepancies in supervisor exception panel. Does not block COs in real-time — maintains throughput while enabling accountability. |
| D10 | **Centralized worklist, not workflow orchestrator** | Domain-specific workflows (loan underwriting, doc verification) have their own complex state machines. Forcing them through Sensei creates a god-service with too many concerns. | Sensei owns task tracking + branch UX, not upstream workflow logic. External services push tasks via `TaskCreationRequest` events. Sensei records outcomes and publishes `TaskCompleted` events. Upstream systems advance their own workflows. Low coupling, domain integrity preserved. |
| D11 | **Product name: Sensei** | The system guides and structures branch operations — it teaches the organization how to work effectively. | Clear metaphor. Aligns with Japanese-themed naming convention (先生). |
