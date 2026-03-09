# Capability: Performance Dashboard

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-05

---

## Business Function

Provide a consistent dashboard structure across two access levels — **Branch** (all positions within the branch) and **Supervisor** (AM and above). Both levels share the same dashboard layout pattern: a home page with summary widgets, a collection list, and a performance summary. Content and scope differ by level.

## Why It Exists (First Principles)

- **Management by Exception**: Both branch staff and AMs need surfaced priorities — who is behind, what needs intervention — scoped to their level of accountability.
- **Area-Level Accountability**: AMs oversee multiple branches and need cross-branch performance, DPD movement, and their own escalated contract list in one place.
- **Staff Motivation**: Transparent personal metrics and gamified rankings drive healthy competition at the branch level.
- **Accountability Chain**: Branch → Area → Region aggregation enables consistent performance reporting upward.

---

## Feature Inventory

| Feature | Level | Status | Description |
|---------|-------|--------|-------------|
| Branch Home Dashboard | Branch | Draft | งานที่ต้องจัดการ widget + team performance summary + exception alerts |
| Branch Performance Summary | Branch | Draft | Team-level metrics: daily scorecard, contact compliance, active playbooks, leaderboard |
| Staff Self-Service Metrics | Branch (CO) | Draft | Personal metrics: tasks completed, PTP rate, visit success, SLA compliance |
| Supervisor Home Dashboard | Supervisor (AM+) | Draft | งานที่พื้นที่ต้องจัดการ widget + branch performance tracking (daily & weekly) + area metrics |
| Area Performance Summary | Supervisor (AM+) | Draft | ยอดสินเชื่อ / ยอดการขาย daily and monthly vs. target; DPD movement (C to X, X to 30) |

---

## Dashboard Structure (Both Levels Share Same Pattern)

| Section | Branch Level | Supervisor Level (AM+) |
|---------|-------------|------------------------|
| **Summary Widget** | งานที่ต้องจัดการ — count of active tasks in CO's queue | งานที่พื้นที่ต้องจัดการ — count of contracts in AM's responsible list |
| **Work Setting** | การเรียงลำดับงาน / Strategy ในการทำงาน | การเรียงลำดับงาน / Strategy ในการทำงาน |
| **Tracking Table** | Team workload per CO (queue size, completed, rate, PTP, alerts) | Branch performance per branch (ติดตามหนี้ + เสนอขาย metrics — วันนี้ and สัปดาห์นี้) |
| **Collection List** | Priority-based contract queue (Work Queue) | → AM Worklist: สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ + การติดตามหนี้ในแต่ละสาขา |
| **Performance Summary** | Daily / weekly / monthly scorecard + contact compliance | Daily and monthly area targets + DPD movement |

---

## Business Rules

---

### A. Branch Level (All Branch Positions)

#### A1. Branch Home Dashboard

| Component | Description |
|-----------|-------------|
| **งานที่ต้องจัดการ** | Summary widget: total active tasks in queue today; click navigates to Work Queue |
| **การตั้งค่าการทำงาน** | Work configuration: การเรียงลำดับงาน (sort order), Strategy ในการทำงาน |
| **ติดตามผลการทำงานของทีม** | Per-CO row: queue size, completed, completion rate, PTP amount, exception flags |
| **Exception Alerts** | Surfaced for supervisor role within branch |

#### A2. Branch Performance Summary

| Component | Purpose |
|-----------|---------|
| Daily Scorecard | Team-level metrics: today / this week / this month / vs. target |
| Active Playbooks Panel | Per-playbook: case count, on-track / at-risk / failed / succeeded |
| Staff Self-Service Metrics | Personal: tasks completed, PTP rate, visit success, SLA compliance |
| Monthly Objectives Tracker | Count of succeeded / in-progress / failed objectives per CO |
| Branch Rank & Leaderboard | Gamified ranking within branch (🥇🥈🥉); composite score = completion rate + PTP rate + SLA compliance + contact compliance |

---

### B. Supervisor Level (AM and Above)

#### B1. Supervisor Home Dashboard (หน้าหลัก)

| Component | Description |
|-----------|-------------|
| **งานที่พื้นที่ต้องจัดการ** | Count of contracts in สัญญาที่อยู่ภายใต้การดูแลของพื้นที่; click navigates to AM's contract list |
| **การตั้งค่าการทำงาน** | Work configuration: การเรียงลำดับงาน (sort order), Strategy ในการทำงาน |
| **ติดตามผลการทำงานของสาขา** | Branch performance table — วันนี้ and สัปดาห์นี้; ติดตามหนี้ and เสนอขาย metrics per branch under AM's area |
| **ภาพรวมการทำงานประจำวันนี้** | Daily: ยอดสินเชื่อ and ยอดการขาย (ส่วนต่าง / ทำได้ / เป้าหมาย) |
| **ภาพรวมการทำงานเดือนนี้** | Monthly cumulative vs. target for same metrics |
| **การไหลของ DPD** | DPD movement rates (C to X, X to 30) for current and prior month |

#### B2 & B3: AM Contract Lists

Moved to **AM Worklist** capability — see [CAPABILITY.md](../am-worklist/CAPABILITY.md).

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Real-time updates | Both dashboards update without page reload (near-real-time, ≤ 30 seconds) |
| Exception alerting | Branch exceptions surfaced within 5 minutes of trigger condition |
| Historical data | Monthly objectives and PTP data retained for at least 12 months |
