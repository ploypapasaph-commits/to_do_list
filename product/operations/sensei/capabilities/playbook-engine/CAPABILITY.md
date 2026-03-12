# Capability: Playbook Engine

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-12

---

## Business Function

Single source of truth for task configuration and evaluation logic across all portfolio types. Defines all configurable parameters (gate conditions, rule chain, action types, timing), evaluates each contract against those rules, and creates the correct task. Re-evaluates after every task closure so the next task emerges from updated contract state — no hardcoded transition routing needed.

## Why It Exists (First Principles)

- **Policy Alignment**: Thousands of branch staff must execute consistent strategies. Configurable rules replace tribal knowledge and branch-invented approaches.
- **Separation of config and logic**: HQ defines what the rules are. The engine applies them. Changing a rule doesn't require changing the engine.
- **Stateless evaluation**: Each evaluation reads only current contract attributes — no dependency on task history or chain state.
- **Adaptability**: HQ defines defaults. Branches (AM and above) adjust timing within guardrails.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Gate Evaluator | Draft | On receiving a contract event, checks contract eligibility and portfolio classification. Gate conditions are HQ-configurable per portfolio type. Suppresses evaluation if gate not met. |
| Rule Chain Evaluator | Draft | Runs the rule chain specific to the portfolio type identified by the gate. Each portfolio type has its own chain defined by HQ. Rules evaluated in fixed sequence — first match wins. |
| Task Instantiation | Draft | Creates all step tasks atomically in the Task Engine when an objective is activated. Reads timing params from Objective Configurations. |
| Re-entry Trigger | Draft | After a CO closes a task, re-enters the pipeline from the gate for that contract using updated contract attributes. |
| Playbook Builder (HQ) | Draft | HQ creates and manages System Templates per portfolio type with action types, timing, and compliance locks. |
| Branch Variant Fork | Draft | AM and above fork a System Template into a Branch Variant and adjust within allowed rules. |
| Compliance Lock Enforcement | Draft | Locked steps (🔒) cannot be removed, reordered past their boundary, or have their configuration modified by supervisors. |

---

## Part 1 — Task Creation

How a task gets created and appears in the Work Queue.

---

### 1.1 Gate

The gate is the **first check** on every evaluation — both on initial event and on re-entry after task closure. It does two things:

1. **Eligibility check** — is this contract in a state that requires action?
2. **Portfolio classification** — which portfolio type does it belong to? (determines which rule chain and template apply)

Gate configurations are **HQ-managed per portfolio type** and stored in the Playbook Engine. HQ can register new portfolio types and define their gate conditions and rule chain without engineering changes — the engine evaluates whichever gate configurations are active. If gate not met → suppress. No rule evaluation, no task created.

Each registered portfolio type carries three things: its gate conditions, its rule chain, and its System Template. The gate identifies the portfolio type — everything that follows (rule chain evaluation, task instantiation) is scoped to that type.

| Portfolio Type | Gate Conditions | Rule Chain |
|---|---|---|
| Collection: Active | ยอดที่ชำระ < ยอดตามคาดการณ์ AND due_date อยู่ในช่วง −7 ถึง +30 วันจากวันนี้ | Collection: Active (→ 1.2.1) |
| Collection: Write-off | contract.status = write_off AND ยอดค้างชำระ > 0 | Collection: Write-off (→ 1.2.2) |

> HQ adds new portfolio types by registering gate conditions + a rule chain together. The gate is not hardcoded to collection — it is a configurable construct. When a contract is fully paid, the gate naturally fails on re-entry and no further tasks are created — no explicit "end chain" command needed.

---

### 1.2 Rule Chain Evaluation

Each portfolio type registered in the gate has its **own rule chain**, defined by HQ in the Playbook Builder. The gate identifies which portfolio type the contract belongs to — the engine then runs that type's chain. Rules are evaluated in sequence; **first match wins.** Evaluation is stateless — reads only current contract attributes.

#### 1.2.1 Collection: Active

| # | Objective | Conditions | Adjustable By |
|---|---|---|---|
| 1 | เอาวันนัดชำระ | No PTP set (`PTP_date` is null) | HQ only |
| 2 | แจ้งเตือนยืนยันนัดชำระ | PTP exists AND `PTP_date − 1 day = today` | HQ only |
| 3 | เก็บยอดตามนัดชำระ | PTP exists AND `PTP_date = today` AND no payment recorded | HQ only |
| 4 | ติดตามเข้มงวด | `due_date` passed with no PTP OR PTP broken (`PTP_date < today`, no payment) | HQ only |
| 5 | ส่งเรื่องให้ผู้จัดการพื้นที่ | TBD | HQ only 🔒 |
| Default | ติดตามหนี้ | No rule above matched | HQ only |

**Worked examples:**

| Contract Attributes | Objective Activated |
|---|---|
| No PTP set | เอาวันนัดชำระ (Rule 1) |
| PTP exists, PTP_date − 1 = today | แจ้งเตือนยืนยันนัดชำระ (Rule 2) |
| PTP exists, PTP_date = today, no payment | เก็บยอดตามนัดชำระ (Rule 3) |
| due_date passed with no PTP | ติดตามเข้มงวด (Rule 4) |
| PTP broken (PTP_date < today, no payment) | ติดตามเข้มงวด (Rule 4) |
| Passes gate, no rule matched | ติดตามหนี้ (Default) |

#### 1.2.2 Collection: Write-off

> Rule chain TBD — objectives and conditions to be defined with stakeholders.

---

### 1.3 Task Instantiation

Once the objective is determined, Playbook Engine instructs Task Engine to create all step tasks atomically. Either all tasks are created or none.

Tasks are created in `CREATED` state and auto-assigned per the template's assignee rules. The applicable System Template is determined by `portfolio_type` (set by the gate in 1.1). Contact gate checks (daily limit + contact window) apply before creation for Call and Visit tasks.

---

## Part 2 — Execution

How a CO works the task after it appears in their queue.

---

### 2.1 Action Types

Each objective is configured with a default action type. Action types are HQ-defined — no action type can be used unless it exists in the registry below.

| Action Type | Typed Outcomes | Required Fields on Specific Outcomes |
|-------------|----------------|--------------------------------------|
| 📞 Call | PTP, No Answer, Refused, Callback, Wrong Number, Line Busy, Voicemail | PTP → PTP amount + PTP date; Callback → scheduled date/time |
| 🏠 Visit | Met Customer, Not Home, Address Invalid, PTP (in-person), Refused | PTP → PTP amount + PTP date; Address Invalid → new address |
| 📋 Admin | Completed, Incomplete, Escalated | Escalated → escalation reason |
| ⏳ Wait | (auto-advances; no manual outcome) | — |
| 🔔 Notify Supervisor | Acknowledged, No Response | — |
| 📧 Send Notification | (system-dispatched; Delivered/Failed) | — |

**Default action per objective** (AM can adjust within HQ limits):

| Objective | Default Action |
|---|---|
| เอาวันนัดชำระ | 📞 Call |
| แจ้งเตือนยืนยันนัดชำระ | 📞 Call |
| เก็บยอดตามนัดชำระ | 📞 Call |
| ติดตามเข้มงวด — รอบแรก | 🏠 Visit 🔒 |
| ติดตามหนี้ | 📞 Call |

---

### 2.2 Outcome Recording

CO records one outcome per task on closure. Each outcome updates one or more contract attributes, which are what the rule chain reads on re-entry.

| Outcome | Contract Attribute Updated |
|---|---|
| PTP date set | `PTP_date` |
| Payment recorded | `payment_amount`, `payment_status` |
| Refused | `refusal_recorded` |
| Can't contact | `last_contact_attempt_result` |
| Rescheduled PTP | `PTP_date` (updated to new date) |
| Task expired without action | `last_task_expired` |

> After task closure, the system automatically re-enters the pipeline (→ Part 3).

---

### 2.3 Compliance Locks

**Compliance-locked steps (🔒)** are mandated by HQ and cannot be modified by supervisors:
- Cannot be removed from the template
- Cannot be reordered past a defined boundary
- Timing adjustable only within HQ-set limits
- Outcome options visible but not modifiable

---

### 2.4 Supervisor Customization

Supervisors fork a System Template into a Branch Variant and may adjust within the following rules:

| Allowed | Not Allowed |
|---------|-------------|
| Reorder non-locked steps | Delete 🔒 locked steps |
| Add optional steps | Edit System Templates directly |
| Add/remove outcomes on non-locked steps | Modify outcome options on 🔒 locked steps |
| Adjust timing (within HQ-set limits) | Reorder locked steps past compliance boundary |
| Change assignee rules | Bypass publishing workflow |
| Set retry limits on non-locked outcomes | Modify urgency-tier assignments |

---

## Part 3 — Results

What happens after a CO records an outcome.

---

### 3.1 Re-entry

After a CO closes a task with an outcome:

1. Contract attributes updated based on recorded outcome
2. System re-enters the pipeline from the **gate** (Part 1.2)
3. Gate re-evaluates:
   - **Gate not met** → no new task → contract exits collection
   - **Gate met** → rule chain re-evaluates
4. Rule chain re-evaluates against updated attributes:
   - Different rule matches → new task created for the new objective
   - Same rule still matches → task re-created per timing params (retry within limit)
   - No rule matches → default ติดตามหนี้ task created

```
CO closes task
      │
      ▼
Contract attributes updated
      │
      ▼
Re-enter pipeline ──→ Gate fails → No task · Contract exits collection
      │
   Gate passes
      │
      ▼
Rule Chain evaluates
      ├── Different rule matches → New objective task created
      ├── Same rule matches     → Re-queue (retry)
      └── No rule matches       → ติดตามหนี้ (default)
```

---

### 3.2 End Conditions

Collection ends when the gate condition is no longer met on re-entry.

| Condition | How It Ends |
|---|---|
| ยอดที่ชำระ >= ยอดตามคาดการณ์ | Gate fails on re-entry → no new task → contract exits collection |
| Restructure | TBD — closes gate condition |
| Repossession | TBD — closes gate condition |

---

## Configuration & Governance

---

### Setting Governance

This table is the **authoritative source** for role-based access across all Playbook Engine settings.

| Setting Category | Add | Adjust | Change | Who |
|-----------------|-----|--------|--------|-----|
| Action types (name, properties) | ✅ | ✅ | ✅ | HQ only |
| Outcomes per action type | ✅ | ✅ | ✅ | HQ only |
| Required fields per outcome | ✅ | ✅ | ✅ | HQ only |
| SLA defaults per action type | ✅ | ✅ | ✅ | HQ (global default); AM (per branch variant, within HQ limits) |
| Retry limits per action type | ✅ | ✅ | ✅ | HQ (global default); AM (per branch variant, within HQ limits) |
| Priority event mapping (event → P1–P4) | ✅ | ✅ | ✅ | HQ only |
| Gate configurations (per portfolio type) | ✅ | ✅ | ✅ | HQ only |
| Rule chain order and conditions (per portfolio type) | ✅ | ❌ | ❌ | HQ only — HQ defines each portfolio type's chain; order and conditions are system-enforced once set |
| Objective timing & action (values) | ✅ | ✅ | ✅ | HQ (global default); AM (within HQ limits) |
| Compliance-locked steps | ✅ | ❌ | ❌ | HQ only — lock/unlock |
| Playbook Engine structure (schema) | ❌ | ❌ | ❌ | System-defined — immutable |

> **Branch variant scope**: AM adjustments apply to their branch variant only. HQ global default is unchanged. AM adjustments cannot exceed HQ-set limits.

---

### SLA Defaults by Action Type

| Action Type | Default SLA | Adjustable By |
|-------------|-------------|---------------|
| 📞 Call | 4 hours | AM (per branch variant, within HQ limits) |
| 🏠 Visit | 8 hours | AM (per branch variant, within HQ limits) |
| 📋 Admin | 24 hours | AM (per branch variant, within HQ limits) |
| ⏳ Wait | Duration defined in objective configuration | HQ only |

---

### Timing Parameters by Objective

| Objective | DL | วันที่สร้างงาน (default) | อายุของงาน (default) | Retry (default) | AM-Adjustable |
|-----------|----|-----------------------|---------------------|----------------|---------------|
| เอาวันนัดชำระ | d (due date) | ก่อน due 7 วัน | 7 วัน | 3 ครั้ง | Timing ✅ |
| แจ้งเตือนยืนยันนัดชำระ | PTP − 1 วัน | ก่อนวันนัดชำระ 1 วัน | ภายในวัน | 3 ครั้ง | Timing ✅ |
| เก็บยอดตามนัดชำระ | PTP | วันนัดชำระ | ภายในวัน | 3 ครั้ง | Timing ✅ |
| ติดตามเข้มงวด — รอบแรก | d + 3 | วันที่ list ขึ้น | 3 วัน | 1 ครั้ง | Timing ✅; Action ❌ (locked) |
| ติดตามเข้มงวด — หลังได้ PTP | PTP + 1 | วันถัดจากวันนัดใหม่ | ภายในวัน | 1 ครั้ง | Timing ✅; Action ❌ (locked) |
| ติดตามหนี้ (default) | d | วันที่ list ขึ้น | 3 วัน | 2 ครั้ง | Timing ✅ |

---

### Playbook Hierarchy

| Level | Owner | Can Edit |
|-------|-------|----------|
| System Template | HQ | Gate conditions, rule chain, action types, timing, compliance locks |
| Branch Variant | AM and above | Timing + action type within HQ-set limits; cannot change gate or rule conditions |

System Templates are organized by `portfolio_type`. HQ manages which template applies to each portfolio type.

---

## Flow Diagram

```mermaid
flowchart TD
    EVENT[📥 Contract Event\nor Re-entry after task closure] --> GATE{"Gate\nEligibility check +\nPortfolio classification\n(HQ-configurable per portfolio type)"}

    GATE -->|Not met| SUPPRESS[🚫 No task created\nContract exits pipeline if re-entry]
    GATE -->|"Collection: Write-off"| CHAIN_B[Rule Chain: Collection Write-off\nTBD]

    GATE -->|"Collection: Active"| R1{"Rule 1\nNo PTP set?"}
    R1 -->|Match| OBJ1[เอาวันนัดชำระ 📞]
    R1 -->|No match| R2{"Rule 2\nPTP_date − 1 = today?"}
    R2 -->|Match| OBJ2[แจ้งเตือนยืนยันนัดชำระ 📞]
    R2 -->|No match| R3{"Rule 3\nPTP_date = today, no payment?"}
    R3 -->|Match| OBJ3[เก็บยอดตามนัดชำระ 📞]
    R3 -->|No match| R4{"Rule 4\nOverdue or PTP broken?"}
    R4 -->|Match| OBJ4[ติดตามเข้มงวด 🏠]
    R4 -->|No match| R5{"Rule 5\nTBD?"}
    R5 -->|Match| OBJ5[ส่งเรื่องให้ผู้จัดการพื้นที่ 🔒]
    R5 -->|No match| DEFT[ติดตามหนี้ 📞\nDefault]

    OBJ1 & OBJ2 & OBJ3 & OBJ4 & DEFT --> CGATE{"Contact Gate\nCall/Visit only"}
    CGATE -->|Passes| TASK[✅ Task CREATED\nCO works task · records outcome]
    CGATE -->|Fails| SUPPRESS

    TASK -->|Outcome recorded\nContract attributes updated| EVENT
```

> Each portfolio type registered in the gate has its own rule chain. The diagram above shows Collection: Active in full. Additional portfolio types (e.g., Write-off, Sales) follow the same pattern with their own rules defined by HQ.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Gate configurability | HQ must be able to register new portfolio types and configure gate conditions without engineering changes |
| Re-entry atomicity | Re-entry evaluation and task creation after task closure must be atomic |
| Compliance lock integrity | Locked steps cannot be removed or reordered by any user except HQ |
| Version tracking | Branch variants must track which system template version they were forked from |
| Instantiation atomicity | Task instantiation must be atomic — all tasks created or none |
| Stateless evaluation | Rule chain reads only current contract attributes — no dependency on task history |
| No duplicate active task | Only one active task per objective per contract at any time |
| HQ-only enforcement | Settings marked "HQ only" must be restricted at system level |
| AM boundary enforcement | AM adjustments validated against HQ-set limits at save time |
| Reference stability | Existing instances referencing a setting must not break when the setting is updated |
