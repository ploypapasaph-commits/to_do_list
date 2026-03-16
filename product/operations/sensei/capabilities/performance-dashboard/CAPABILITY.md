# Capability: Performance Dashboard

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-16

---

## Business Function

Provide a consistent performance dashboard for all branch positions — showing today's work snapshot (daily) and cumulative progress against targets (monthly). Both Branch and Supervisor levels see the same dashboard layout; scope differs by role (branch vs. area).

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
| Branch Performance Summary | Branch | Draft | Daily + Monthly view: การขาย metrics and การเก็บหนี้ metrics scoped to branch |
| Staff Self-Service Metrics | Branch (CO) | 💡 Good-to-Have | Personal metrics: tasks completed, PTP rate, visit success, SLA compliance. Not in current scope — deferred. |
| Supervisor Home Dashboard | Supervisor (AM+) | Draft | งานที่พื้นที่ต้องจัดการ widget + branch performance tracking (daily & monthly) + area metrics |
| Area Performance Summary | Supervisor (AM+) | Draft | Daily + Monthly view: การขาย metrics and การเก็บหนี้ metrics scoped to area |

---

## Dashboard Structure (Both Levels Share Same Pattern)

| Section | Branch Level | Supervisor Level (AM+) |
|---------|-------------|------------------------|
| **Summary Widget** | งานที่ต้องจัดการ — count of active tasks in CO's queue | งานที่พื้นที่ต้องจัดการ — count of contracts in AM's responsible list |
| **Work Setting** | การเรียงลำดับงาน / Strategy ในการทำงาน | การเรียงลำดับงาน / Strategy ในการทำงาน |
| **Tracking Table** | Team workload per CO (queue size, completed, rate, PTP, alerts) | Branch performance per branch (ติดตามหนี้ + เสนอขาย metrics — วันนี้ and สัปดาห์นี้) |
| **Collection List** | Priority-based contract queue (Work Queue) | AM Worklist: สัญญาที่อยู่ภายใต้การดูแลของพื้นที่ + การติดตามหนี้ในแต่ละสาขา |
| **ภาพรวมการทำงานประจำวันนี้** | Daily snapshot vs. plan — การขาย and การเก็บหนี้ | Same, scoped to area |
| **ภาพรวมการทำงานเดือนนี้** | Monthly cumulative vs. target — การขาย, การเก็บหนี้, DPD movement | Same, scoped to area |

---

## Metric Registry

> **Metrics can be added at any time** — add a row to the relevant table below and specify: Metric name, Target, and Direction.

### การขาย (Sales Metrics)

Shown in both daily and monthly views. Daily shows actual vs. plan; monthly shows ส่วนต่าง / ยอดสะสม / เป้าหมาย.

| Metric | Unit | Target | Direction | Note |
|--------|------|--------|-----------|------|
| Net Booking เทียบเป้า | บาท | เราเอง (branch own target) | — | |
| ประกันรวม เทียบเป้า | บาท | เราเอง (branch own target) | — | |
| ลูกค้าใหม่ เทียบเป้า | คน | tier 1 | — | |
| %การทำ Top Up Nano | % | 40% | ยิ่งมากยิ่งดี | ดูเป็นรายคน — ทำ ≥ 1 รายการ/คน นับว่าทำ |

**ยอดสินเชื่อ breakdown** (แสดงแยก On Top / Top Up / ลูกค้าใหม่):
- Daily: ยอดที่ทำได้ (บาท) / จำนวน (สัญญา/คน)
- Monthly: ส่วนต่าง / ยอดสะสม / เป้าหมาย

---

### การเก็บหนี้ (Collection Metrics)

Shown in monthly view. DPD movement compares เดือนนี้ vs. เดือนก่อน vs. เป้าหมาย.

| Metric | Unit | Target | Direction | Note |
|--------|------|--------|-----------|------|
| %C เทียบคาดการณ์ | % | — | ยิ่งมากยิ่งดี | |
| %CX เทียบคาดการณ์ | % | — | ยิ่งมากยิ่งดี | |
| %C to X | % | 3% | ยิ่งน้อยยิ่งดี | |
| %X to 30 | % | 10% | ยิ่งน้อยยิ่งดี | |
| %30+ to CX | % | 15% | ยิ่งมากยิ่งดี | |
| Write-off เทียบเป้า | บาท | tier 1 | — | |

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
| **ภาพรวมการทำงานประจำวันนี้** | Daily snapshot: การขาย metrics vs. plan (see Metric Registry above) |
| **ภาพรวมการทำงานเดือนนี้** | Monthly cumulative: การขาย + การเก็บหนี้ + DPD movement vs. target |
| Staff Self-Service Metrics | 💡 Good-to-Have — Personal: tasks completed, PTP rate, visit success, SLA compliance. Not in current scope — deferred. |
| Monthly Objectives Tracker | 💡 Good-to-Have — Count of succeeded / in-progress / failed objectives per CO. Deferred. |
| Branch Rank & Leaderboard | 💡 Good-to-Have — Gamified ranking within branch. Deferred. |

---

### B. Supervisor Level (AM and Above)

#### B1. Supervisor Home Dashboard (หน้าหลัก)

| Component | Description |
|-----------|-------------|
| **งานที่พื้นที่ต้องจัดการ** | Count of contracts in สัญญาที่อยู่ภายใต้การดูแลของพื้นที่; click navigates to AM's contract list |
| **การตั้งค่าการทำงาน** | Work configuration: การเรียงลำดับงาน (sort order), Strategy ในการทำงาน |
| **ติดตามผลการทำงานของสาขา** | Branch performance table — วันนี้ and สัปดาห์นี้; ติดตามหนี้ and เสนอขาย metrics per branch under AM's area |
| **ภาพรวมการทำงานประจำวันนี้** | Daily snapshot: การขาย metrics vs. plan scoped to area (see Metric Registry above) |
| **ภาพรวมการทำงานเดือนนี้** | Monthly cumulative: การขาย + การเก็บหนี้ + DPD movement vs. target scoped to area |

#### B2 & B3: AM Contract Lists

Moved to **AM Worklist** capability — see [CAPABILITY.md](../am-worklist/CAPABILITY.md).

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Real-time updates | Both dashboards update without page reload (near-real-time, ≤ 30 seconds) |
| Exception alerting | Branch exceptions surfaced within 5 minutes of trigger condition |
| Historical data | Monthly objectives and PTP data retained for at least 12 months |
