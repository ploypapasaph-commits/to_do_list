# Changelog 009: Order → Application Two-Level Model

> **Date:** 2026-03-05
> **Layer:** Capability (Application Management)
> **Status:** Documented

---

## What Changed

Restructured the Application Management capability from a single "application" concept into a two-level **Order → Application** model.

### Why

Bundle checkout (1 compulsory + 1 voluntary insurance) requires a single payment but separate policy issuance per insurer-product. Task configurations (e.g. document upload, vehicle inspection) differ per insurer × product × channel — so each package needs its own lifecycle. Payment, however, is collected once for the whole checkout.

### New Model

| Concept | Scope | Contains |
|---------|-------|----------|
| **Order** | The checkout (1 or 2 packages) | Customer info, sale channel, order-level tasks (payment), 1–2 applications |
| **Application** | A single insurer × product | Application-level tasks, issuance tracking, policy document |

### Order Status (new)

`Draft` → `Ongoing` → `Completed` / `Cancelled`

Order is `Completed` only when all applications reach `Completed`.

### Application Status (revised)

`Ongoing` → `PendingIssuance` → `Completed` / `IssuanceFailed` / `Cancelled`

Applications no longer have `Draft` — they begin in `Ongoing` when the order is submitted. If an application has no application-level tasks and payment is done, it starts directly in `PendingIssuance`.

### Task Levels (new)

- **Order-level tasks:** `payment` — collected once per order
- **Application-level tasks:** `document_upload`, `vehicle_inspection`, `health_declaration`, etc. — configured per insurer × product × channel

### Application History (revised)

- Branch staff: view order list with pending tasks and summarized info
- Online customer: view all orders with application statuses; download/print issued policy documents

### New Business Rules

- AM-017: Single package → order with 1 application
- AM-018: Bundle → order with 2 applications, payment collected once
- AM-019: Payment is always order-level
- AM-020: Order status transition to Ongoing
- AM-021: Order status transition to Completed
- AM-022: Order cancellation cascades to all applications
- AM-023: Application with no tasks starts in PendingIssuance

### Revised Business Rules

- AM-005: Timeout now applies at order level
- AM-007: Issuance gate now requires both application-level tasks AND order-level payment
- AM-014: Application starts in Ongoing (no Draft)
- AM-015: PendingIssuance requires order-level payment completed

## Documents Updated

- `capabilities/application-management/CAPABILITY.md` — Full restructuring of business function, feature inventory, status model, task model, business rules, and open questions
