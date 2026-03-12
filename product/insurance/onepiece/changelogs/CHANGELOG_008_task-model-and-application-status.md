# Changelog 008: Task Model & Application Status

> **Date:** 2026-03-05
> **Layer:** Capability (Application Management)
> **Status:** Documented

---

## What Changed

Restructured the Application Management capability to separate two concerns:

1. **Application Status** (new) — user-facing states representing overall application progress: `Draft`, `Ongoing`, `PendingIssuance`, `IssuanceFailed`, `Completed`, `Cancelled`
2. **Tasks** (renamed from "completion steps") — individual units of work that must be completed before the application advances

### Naming Change

| Before | After |
|--------|-------|
| Completion step | Task |
| Step type | Task type |
| Step configuration | Task configuration |
| Step lifecycle | Task lifecycle |
| Application State Machine (single concept) | Application Status + Task Model (two layers) |

### New Application Statuses

| Status | When |
|--------|------|
| `Draft` | Application created, form not yet submitted |
| `Ongoing` | Form submitted, tasks in progress |
| `PendingIssuance` | All tasks done, awaiting insurer |
| `IssuanceFailed` | Issuance error, retry available |
| `Completed` | Policy issued |
| `Cancelled` | Cancelled or timed out |

### New Business Rules

- AM-014: Status transition Draft → Ongoing on form submission
- AM-015: Status transition Ongoing → PendingIssuance when all tasks completed/skipped
- AM-016: Status transition PendingIssuance → IssuanceFailed on issuance error

## Rationale

The previous model conflated internal task tracking with the user-facing application state. Separating them provides:
- A simpler, meaningful status for staff and customers (e.g. "Ongoing" instead of "StepPending: document_upload")
- Finer operational statuses like `PendingIssuance` and `IssuanceFailed` for staff visibility
- Clean separation between "what the user sees" and "what the system tracks internally"

## Documents Updated

- `capabilities/application-management/CAPABILITY.md` — Full rewrite of state machine and task sections; updated business rules AM-005 through AM-012; added AM-014, AM-015, AM-016; updated open questions
