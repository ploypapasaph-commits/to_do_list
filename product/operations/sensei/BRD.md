# Business Requirements Document (BRD)

**Project Name**: Sensei — Branch Operations Orchestration Platform
**Project Requester**: TBD
**Project Owner**: TBD (Operations PO)
**Status**: Draft
**Last Updated**: 2026-03-17

---

## 1. Executive Summary

### Background

Branch Credit Officers (COs) currently work across multiple disconnected systems — Onigiri for loan tasks, Matcha for document verification tasks, and separate delinquency systems for collection follow-up. There is no single place for a CO to see all their work for the day. As a result, tasks are missed, follow-up is inconsistent, and there is no unified view of accountability across the team.

Branch Supervisors and Area Managers (AMs) face the same problem at the oversight level: no consolidated view of team throughput, no contact compliance tracking, and no systematic way to surface exceptions or escalations. Collection strategies exist as tribal knowledge — not as executable, auditable playbooks.

### Objective

Build **Sensei**, a centralized branch worklist and task orchestration platform that:
- Aggregates all branch work (collection, sales, offerings, admin) into a single prioritized queue per CO
- Enforces collection playbooks configured by HQ and executed at the branch level
- Provides role-scoped performance dashboards for Branch, Supervisor, and AM levels
- Enforces contact compliance (daily contact limits and business hours)
- Integrates with Onigiri, Matcha, DaVinci, Core Banking, and Policy Admin as the single task execution layer for branch operations

### Rationale

Without Sensei, field staff will continue to context-switch between systems, creating operational inefficiency, compliance risk, and poor customer experience. The scale of the problem — COs handling 300–500 customer contacts per day per branch — means that even a small improvement in task prioritization and compliance enforcement has significant business impact. Centralizing collection playbooks under HQ control also reduces strategic drift across branches.

---

## 2. Objectives and Reasons Behind Them

| # | Objective | Reason |
|---|-----------|--------|
| 1 | Provide COs with a single prioritized work queue for all task types | Eliminate context-switching; ensure highest-urgency work is done first |
| 2 | Enable HQ to define and enforce collection playbooks (rule chains, objectives, timing) | Standardize collection strategy across branches; reduce reliance on tribal knowledge |
| 3 | Enforce daily contact limits and business hours compliance automatically | Prevent regulatory and policy violations without requiring manual oversight |
| 4 | Surface performance metrics for Branch and AM levels in real-time | Enable management by exception; allow supervisors and AMs to intervene quickly |
| 5 | Provide AM with an execution queue for escalated contracts | Give AMs clear ownership of escalated cases; prevent contracts from falling through the cracks after branch-level exhaustion |
| 6 | Integrate with Onigiri and Matcha as a unified task execution layer | Eliminate duplicate worklists; ensure CO sees all their tasks in one place regardless of originating system |

---

## 3. Success Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| CO Daily Throughput | Average tasks completed per CO per day | > 350 |
| Contact Compliance Rate | % of days with zero contact limit violations per branch | 100% |
| SLA Breach Rate | % of tasks that expire OVERDUE without supervisor intervention within 4 hours | < 5% |
| Playbook Completion Rate | % of initiated playbooks that reach End (Success) vs. End (Failed) | Track per playbook type |
| Collection Write-off vs. Tier 1 | Write-off amount vs. tier 1 incentive target | Tier 1 |
| %C to X | % of C-bucket contracts rolling to X bucket | < 3% |
| %X to 30 | % of X-bucket contracts rolling to 30+ bucket | < 10% |
| %30+ to CX | % of 30+ contracts recovering to CX | > 15% |

---

## 4. Scope

### In Scope

| Capability | Description |
|-----------|-------------|
| Playbook Engine | Gate evaluation, rule chain (objective selection), action types, timing parameters, compliance-locked steps, HQ System Templates, Branch Variant fork model, setting governance |
| Task Engine | Unified task lifecycle (CREATED → ASSIGNED → ACTIVE → CLOSED); event-driven task generation from DaVinci / Core Banking / Policy Admin; external task creation (TaskCreationRequest from Onigiri / Matcha); contact limit pre-check; ContactWindowClosed enforcement |
| Work Queue — Branch | P1–P4 priority queue (Collection / Sales / Offerings tabs); contract table with urgency display; one-by-one processing mode |
| Work Queue — AM+ | AM execution queue (สัญญาที่อยู่ภายใต้การดูแลของพื้นที่); escalated and manually-added contracts; AM actions: มอบหมายงาน / ดำเนินคดี / หาที่อยู่ใหม่ |
| Performance Dashboard | Home Dashboard + Performance Summary for all levels (role-scoped); Branch Collection Browse (การติดตามหนี้ในแต่ละสาขา) for AM oversight |
| Contact Compliance Enforcement | Daily contact limit enforcement via BOS log check at task generation; business hours enforcement via ContactWindowClosed event |

### Out of Scope

| Item | Owner |
|------|-------|
| Contact compliance data ownership (contact log, frequency limits, cross-product aggregation) | DaVinci |
| Loan workflow orchestration, application state management | Onigiri |
| Document verification logic or QA workflow | Matcha |
| Customer master data and Golden Record | DaVinci |
| ResolutionRequest lifecycle state | DaVinci |

---

## 5. Stakeholders

| Department | Person (If any) | Scope |
|-----------|-----------------|-------|
| Branch Operations | TBD | Primary users — COs who execute daily work via Work Queue |
| Branch Supervision | TBD | Branch-level supervisors who monitor team performance via Performance Dashboard |
| Area Management | TBD | AMs who manage escalated contracts via AM Queue and monitor branch performance |
| HQ — Credit / Collections | TBD | Define and maintain collection playbooks (rule chains, gate conditions, timing parameters) via Playbook Engine |
| Data and Platform | Amp Panus Sriwattana (Director II) | System integration owner — DaVinci contact log, risk scores, event infrastructure |
| Technology / Engineering | TBD | Build and operate Sensei platform (Task Engine, Work Queue, integrations) |
| PMO | Suprasert Khaolaorr (Senior Manager II) | Project governance and delivery oversight |
| Onigiri Product Team | TBD | Integration — TaskCreationRequest and TaskCompleted events |
| Matcha Product Team | TBD | Integration — TaskCreationRequest and TaskCompleted events |
| Compliance / Legal | TBD | Validate compliance-locked steps; contact limit rules |

---

## 6. Business Requirements (High-Level)

### BR-01: Unified Work Queue
- The system must provide a single prioritized work queue for each CO showing all tasks regardless of originating system (Sensei-generated, Onigiri-requested, Matcha-requested, or manually created by supervisor)
- Tasks must be grouped by priority (P1–P4) for Collection and by separate tabs for Sales and Offerings
- COs must be able to drill into any contract to access full collection history and record outcomes

### BR-02: HQ-Configured Collection Playbooks
- HQ must be able to define collection rule chains (gate conditions, objective selection rules, timing parameters, compliance-locked steps) without engineering changes
- Rule chains must support at least two portfolio types: Collection Active and Collection Write-off
- Compliance-locked steps must not be removable or reorderable by branch supervisors

### BR-03: Contact Compliance Enforcement
- The system must automatically suppress task creation if a CO has reached the daily contact limit for a contract (checked against BOS collection note log)
- The system must not create contact tasks outside defined business hours (ContactWindowClosed event from DaVinci)

### BR-04: AM Escalation Queue
- Contracts that exhaust branch-level collection attempts (ส่งเรื่องให้ผู้จัดการพื้นที่) must appear atomically in the AM's execution queue with no gap between branch task closure and AM queue entry
- AM must be able to act on each contract with one of three actions: assign (มอบหมายงาน), legal (ดำเนินคดี), or find address (หาที่อยู่ใหม่)
- AM must be able to manually pull any contract from any branch under their area into their queue

### BR-05: Performance Dashboard — Branch Level
- Branch and Supervisor must see a Home Dashboard showing: task queue summary, team workload tracking (per-CO), and work settings
- Both must see a Performance Summary showing daily and monthly การขาย and การเก็บหนี้ metrics vs. targets

### BR-06: Performance Dashboard — AM Level
- AM must see a Home Dashboard showing: area contract count, per-branch performance table, and work settings
- AM must see a Performance Summary showing area-scoped daily and monthly metrics
- AM must have access to Branch Collection Browse (การติดตามหนี้ในแต่ละสาขา) to view full contract lists per branch under their area

### BR-07: External Task Integration
- The system must accept TaskCreationRequest events from Onigiri and Matcha and create tasks for the appropriate CO
- On task completion, the system must publish TaskCompleted events back to the originating system with outcome and source reference

---

## 7. Things to Consider and Be Aware Of

- **Contact limit data ownership**: Sensei enforces the limit but does not own the contact log. Any discrepancy between BOS log and actual contact must be handled by DaVinci, not Sensei.
- **Playbook complexity at scale**: Rule chains are evaluated after every task closure. High-frequency evaluation at branch scale requires careful performance design in Task Engine.
- **AM queue atomicity**: Escalation from branch to AM must be atomic — there must be no window where a contract has no owner (branch task closed but not yet in AM queue).
- **Role-based access control**: Performance Dashboard content is scoped by level (Branch vs. AM+). Access rules must be enforced at the API level, not just UI.
- **Write-off contact cooldown**: Write-off rule chain uses a 5-day contact cooldown (not a count cap). The Task Engine must correctly evaluate last contact date for cooldown logic.
- **Collection metrics are monthly**: การเก็บหนี้ metrics (DPD movement) are monthly — daily view only shows การขาย. Make sure metric display logic handles this correctly.

---

## 8. Prerequisites and Dependencies

| Dependency | System | Type | Description |
|-----------|--------|------|-------------|
| Contact limit log | DaVinci / BOS | Data | Daily contact count per CO per contract — queried at task generation time |
| ContactWindowClosed event | DaVinci | Event | Business hours enforcement — Sensei subscribes to this event |
| risk_level (Active portfolio) | DaVinci | Data | Urgency score for sort order in Work Queue |
| easiness_to_collect (Write-off portfolio) | DaVinci | Data | Urgency score for sort order in Work Queue |
| customer.resolution_required event | DaVinci | Event | Triggers Admin tasks for COs |
| Delinquency events | Core Banking → DaVinci | Event | Triggers collection task generation |
| Insurance renewal events | Policy Admin → DaVinci | Event | Triggers Sales task generation |
| TaskCreationRequest | Onigiri, Matcha | Event | External task creation contract |
| Branch + AM user roles | Identity / IAM | Data | Role-based access control for dashboard scoping |

---

## 9. Budget

- **Investment Budget**: TBD
  *(งบลงทุนที่จ่ายได้ เมื่อเทียบกับ value ของโครงการนี้ / งบสูงสุดที่แบ่งให้ได้)*

---

## 10. Deadline

- **Expected Launch Date**: TBD

---

## 11. Additional Attachments

- [ ] **Business Cases**: Collection playbook options (Active vs. Write-off rule chains); incentive tier structure
- [ ] **Business Process**: Collection workflow (playbook evaluation flow, task lifecycle, escalation path to AM)
- [ ] **Model**: Task lifecycle state diagram (CREATED → ASSIGNED → ACTIVE → CLOSED / OVERDUE / ESCALATED)
- [ ] **Calculation / Formula**: Priority bucket conditions (P1–P4), Deadline color coding rules, contact cooldown logic
- [ ] **Integration Map**: Sensei ↔ DaVinci / Onigiri / Matcha / Core Banking / Policy Admin
- [ ] **Detailed Reference**: [ATLAS.md](ATLAS.md) — full capability specifications and design decisions

---

## Signature (For Completed BRD)

| Role | Name | Position | Division | Signature |
|------|------|----------|----------|-----------|
| **Project Owner** (Verify overall project) | TBD | TBD | TBD | |
| | | Chief of Division | | |
| **PMO** (Verify overall project) | Suprasert Khaolaorr | Senior Manager II — Project Management Office | CEO Office | |
| **Key Stakeholder** (Verify overall project) | Amp Panus Sriwattana | Director II — Data and Platform | Data and Platform | |
