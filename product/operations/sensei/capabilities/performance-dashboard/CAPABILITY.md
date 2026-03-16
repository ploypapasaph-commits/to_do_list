# Capability: Performance Dashboard

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-16

---

## Business Function

Provide a consistent performance dashboard for all branch positions — showing today's work snapshot (daily) and cumulative progress against targets (monthly). All positions see the same two-feature layout; visible sections and data scope are determined by role.

## Why It Exists (First Principles)

- **Management by Exception**: Both branch staff and AMs need surfaced priorities — who is behind, what needs intervention — scoped to their level of accountability.
- **Area-Level Accountability**: AMs oversee multiple branches and need cross-branch performance, DPD movement, and their own escalated contract list in one place.
- **Staff Motivation**: Transparent personal metrics and gamified rankings drive healthy competition at the branch level.
- **Accountability Chain**: Branch → Area → Region aggregation enables consistent performance reporting upward.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Home Dashboard | Draft | Entry screen for all positions — summary widget, work settings, tracking table, and collection list; visible sections and data scope determined by role |
| Performance Summary | Draft | Daily and monthly performance view for all positions — การขาย and การเก็บหนี้ metrics; data scope determined by role |
| Staff Self-Service Metrics | 💡 Good-to-Have | Personal metrics: tasks completed, PTP rate, visit success, SLA compliance. Not in current scope — deferred. |

---

## Role-Based Access

All positions use the same two features. Role determines which sections are visible and what scope of data is shown.

### Home Dashboard — Sections by Level

| Section | Branch | AM+ |
|---------|--------|-----|
| **งานที่ต้องจัดการ** — own task queue count | ✅ | — |
| **งานที่พื้นที่ต้องจัดการ** — area contract count | — | ✅ |
| **การตั้งค่าการทำงาน** — sort order, strategy | ✅ | ✅ |
| **ติดตามผลการทำงานของทีม** — per-CO workload table | ✅ | — |
| **ติดตามผลการทำงานของสาขา** — per-branch metrics table | — | ✅ |

### Performance Summary — Sections by Level

| Section | Branch | AM+ |
|---------|--------|-----|
| **ภาพรวมการทำงานประจำวันนี้** | ✅ branch-scoped | ✅ area-scoped |
| **ภาพรวมการทำงานเดือนนี้** | ✅ branch-scoped | ✅ area-scoped |
| Staff Self-Service Metrics | 💡 Deferred | — |
| Monthly Objectives Tracker | 💡 Deferred | — |
| Branch Rank & Leaderboard | 💡 Deferred | — |

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

### Feature 1: Home Dashboard

#### งานที่ต้องจัดการ / งานที่พื้นที่ต้องจัดการ (Summary Widget)

| Level | Widget Shown | Content | Navigation |
|-------|-------------|---------|-----------|
| Branch | งานที่ต้องจัดการ | Total active tasks in own queue today | Click → Work Queue |
| AM+ | งานที่พื้นที่ต้องจัดการ | Count of contracts in AM's responsible contract list | Click → AM Worklist |

#### การตั้งค่าการทำงาน (Work Settings)

Available to all levels. Controls: การเรียงลำดับงาน (sort order), Strategy ในการทำงาน.

#### Tracking Table

| Level | Table Shown | Columns |
|-------|------------|---------|
| Branch | ติดตามผลการทำงานของทีม | Per-CO row: queue size, completed, completion rate, PTP amount, exception flags |
| AM+ | ติดตามผลการทำงานของสาขา | Per-branch row: ติดตามหนี้ + เสนอขาย metrics — วันนี้ and สัปดาห์นี้ |

---

### Feature 2: Performance Summary

#### ภาพรวมการทำงานประจำวันนี้ (Daily Snapshot)

Available to all levels. Data scope differs by level:

| Level | Scope | Content |
|-------|-------|---------|
| Branch | Branch | การขาย metrics vs. plan (see Metric Registry) |
| AM+ | Area | การขาย metrics vs. plan scoped to area |

#### ภาพรวมการทำงานเดือนนี้ (Monthly Cumulative)

Available to all levels. Data scope differs by level:

| Level | Scope | Content |
|-------|-------|---------|
| Branch | Branch | การขาย + การเก็บหนี้ + DPD movement vs. target |
| AM+ | Area | การขาย + การเก็บหนี้ + DPD movement vs. target scoped to area |

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Real-time updates | Both features update without page reload (near-real-time, ≤ 30 seconds) |
| Historical data | Monthly objectives and PTP data retained for at least 12 months |
