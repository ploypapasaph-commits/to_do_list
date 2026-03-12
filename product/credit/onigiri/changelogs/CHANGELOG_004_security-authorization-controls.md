# Changelog: Security & Authorization Controls

**Product**: Onigiri (Loan Origination System)
**Portfolio**: Credit
**Changelog #**: 004
**Layer Affected**: Capability
**Date**: 2026-03-10
**Author**: Product Audit Session (2026-03-09)
**Audit Reference**: `product/credit/onigiri/audit_result.md`

---

## What Changed (Documentation)

| File | Capability | Change |
|------|-----------|--------|
| `capabilities/underwriting-workflow/CAPABILITY.md` | Underwriting Workflow | Added `Inbound Callback Authentication` feature + business rule section (IS-1) |
| `capabilities/risk-assessment-engine/CAPABILITY.md` | Risk Assessment Engine | Added `Rule Change Authorization` feature + business rule section (AI-1) |
| `capabilities/loan-campaign-configuration/CAPABILITY.md` | Loan Campaign Configuration | Added `Campaign Publication Approval Workflow` feature + business rule section; updated user flow diagram; resolved open question on approval gate (AI-2) |

---

## Engineering Work Required

The three audit findings below are resolved at the documentation layer. Engineering must now implement the controls. Changes are grouped by natural build boundary.

---

### Group A — Inbound Webhook Authentication
*Audit finding: IS-1 | Severity: Critical (now mitigated in spec)*
*Capability: Underwriting Workflow*

**What to build**: An authentication gate that sits in front of all inbound callback handlers. No state transition may be triggered by an unauthenticated callback.

| Requirement | Detail |
|-------------|--------|
| Algorithm | HMAC-SHA256 |
| Secret storage | Onigiri secrets manager (not hardcoded, not in config files) |
| Callers in scope | Matcha (QA outcome webhook) and Wasabi (async document verification report) |
| Missing signature | Return HTTP 401; raise security alert; do not transition state |
| Invalid signature | Return HTTP 403; raise security alert; do not transition state |
| Valid but stale (> 5 min) | Return HTTP 400; log replay attempt; do not transition state |
| Valid and timely | Return HTTP 200; proceed with state transition |
| Replay protection | Timestamp must be embedded in the signed payload; reject if outside the 5-minute window |

**Authentication contract** (to be agreed with Matcha and Wasabi teams):
- The shared secret is provisioned per-caller (Matcha has its own secret; Wasabi has its own secret)
- The HMAC covers: `{caller_id}.{timestamp_unix}.{callback_payload_hash}`
- Signature is delivered in the `X-Onigiri-Signature` request header
- Secret rotation must be supported without downtime (dual-accept window during rotation)

**Out of scope for this ticket**: Changes to Matcha or Wasabi — those teams must implement the signing side. Onigiri only implements the verification side.

---

### Group B — Tiered Approval Workflow (Risk Rules + Campaign Publication)
*Audit findings: AI-1, AI-2 | Severity: High (both mitigated in spec)*
*Capabilities: Risk Assessment Engine + Loan Campaign Configuration*

AI-1 and AI-2 are intentionally grouped into a single engineering work stream. The two capabilities share a **combined approval package** mechanism — a risk strategy change that affects an active campaign must be co-approved with the new campaign version in one CRO decision. Build these together.

#### B1 — Approval Workflow State Machine (shared infrastructure)

Both capabilities require the same two-tier approval state machine. Build once, apply to both.

| State | Description |
|-------|-------------|
| `DRAFT` | Editable; not yet submitted for approval |
| `PENDING_T1` | Submitted; awaiting Tier 1 approver action |
| `PENDING_T2` | Tier 1 approved; awaiting CRO action |
| `APPROVED` / `ACTIVE` | CRO approved; change is live |
| `RETURNED` | Rejected at either tier; returned to DRAFT with feedback |

| Transition | Trigger | Guard |
|------------|---------|-------|
| DRAFT → PENDING_T1 | Submitter submits for approval | Completeness validation passes |
| PENDING_T1 → PENDING_T2 | Tier 1 approves | Actor is Tier 1 approver role AND is not the submitter |
| PENDING_T1 → RETURNED | Tier 1 rejects | Actor is Tier 1 approver role |
| PENDING_T2 → APPROVED | Tier 2 (CRO) approves | Actor holds CRO role |
| PENDING_T2 → RETURNED | Tier 2 (CRO) rejects | Actor holds CRO role |

#### B2 — Risk Rule Change Authorization (AI-1)
*Tier 1: Risk Officer | Tier 2: CRO*

| Requirement | Detail |
|-------------|--------|
| Scope | Any create, edit, deactivate, or delete on: Strategy, Policy, Rule |
| Tier 1 approver role | Risk Officer — must not be the same user who initiated the change |
| Tier 2 approver role | CRO |
| Auto-decline class (risk_level ≥ 70) | After CRO approval, additionally trigger Compliance notification (async, non-blocking) |
| Audit trail | On every state transition: record actor ID, action type, entity before/after state snapshot, approver IDs, timestamp, affected campaign IDs |
| Rule is not live | Until APPROVED state is reached, the rule change must not affect any evaluation |

#### B3 — Campaign Publication Approval Workflow (AI-2)
*Tier 1: CPO | Tier 2: CRO*

| Requirement | Detail |
|-------------|--------|
| Scope | Any Draft → ACTIVE publication of a campaign version |
| Tier 1 approver role | CPO — must not be the same user who built the campaign |
| Tier 2 approver role | CRO |
| Campaign lifecycle enforcement | Draft: editable. PENDING_T1 / PENDING_T2: read-only. ACTIVE: append-only (any edit creates a new Draft version). Archived: read-only. |
| In-flight applications | Always use the campaign version at submission time — no retroactive version migration on version change |

#### B4 — Combined Approval Package (coupling constraint)

This is the key constraint that makes AI-1 and AI-2 a single build group.

| Requirement | Detail |
|-------------|--------|
| Trigger condition | A new campaign version changes the assigned risk strategy (new strategy ID or new strategy version) |
| Coupling rule | The campaign version and the risk strategy change must be linked into a single approval package and submitted together |
| CRO decision | One Tier 2 decision approves or rejects both simultaneously — they cannot be approved independently |
| Blocking condition | A campaign version referencing a risk strategy version that has not been co-approved via a combined package cannot transition to ACTIVE |
| UX implication | When a product manager submits a campaign that includes a risk strategy change, the system must surface both changes in the approval review UI as a linked bundle |

---

## Decision Log

| Decision | Rationale |
|----------|-----------|
| Tiered approval (2-level) rather than countersign | Countersign implies peer review; the intent is escalation — functional owner reviews first, CRO has final authority at all times |
| AI-1 and AI-2 built together | The combined approval package coupling means the approval state machine is shared infrastructure; building separately would require integration later and risks inconsistent behavior |
| Matcha/Wasabi secrets are per-caller | Shared secret across callers creates a blast radius — one compromised secret would affect both integration surfaces |
| Risk strategy change is a campaign-level event | The campaign is the single source of truth for what executes against an application; a strategy version change without a corresponding campaign version change would silently alter live behavior |

---

## Links

- [Underwriting Workflow CAPABILITY.md](../capabilities/underwriting-workflow/CAPABILITY.md) — IS-1 spec
- [Risk Assessment Engine CAPABILITY.md](../capabilities/risk-assessment-engine/CAPABILITY.md) — AI-1 spec
- [Loan Campaign Configuration CAPABILITY.md](../capabilities/loan-campaign-configuration/CAPABILITY.md) — AI-2 spec
- [Audit Result](../audit_result.md) — full finding detail and priority order
