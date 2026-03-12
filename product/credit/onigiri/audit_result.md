# Security Architecture Review — Onigiri (Loan Origination System)

**Scope**: All four capabilities + integration surface + workflow state machine
**Reviewed by**: Claude (AI-assisted SA review)
**Date**: 2026-03-09
**Status of prior fix**: FI-1 (double disbursement via channel change post-facility) — **MITIGATED**

---

## Finding Summary

| ID | Finding | Category | Severity |
|----|---------|----------|----------|
| IS-1 | No webhook authentication on Matcha/Wasabi callbacks | Integration Security | ~~Critical~~ **Mitigated** |
| FI-2 | Collateral fields not in any lockpoint group post-facility | Financial Integrity | **High** |
| AI-1 | Risk rule deactivation has no dual-authorization gate | Authorization Integrity | ~~High~~ **Mitigated** |
| AI-2 | Campaign publication has no approval workflow | Authorization Integrity | ~~High~~ **Mitigated** |
| AC-1 | Supervisor recall unrestricted by financial commitment state | Access Control | **High** |
| AC-2 | RBAC model for workflow transitions not defined | Access Control | **High** |
| IS-2 | Core Banking idempotency assumed, not specified as a requirement | Integration Security | **High** |
| IS-4 | Dashi response carries no integrity guarantee | Integration Security | **High** |
| FI-3 | NCB re-inquiry on re-submission — consent scope and cost | Financial Integrity | **Medium** |
| DI-1 | Application JSON not snapshotted at Risk Assessment entry | Data Integrity | **Medium** |
| DI-2 | No input sanitization documented for JMESPath evaluation inputs | Data Integrity | **Medium** |
| IS-3 | Miso response not schema-validated | Integration Security | **Medium** |
| AU-1 | Audit log immutability mechanism not specified | Audit | **Medium** |
| FI-1 | Double disbursement via disbursement channel change post-facility | Financial Integrity | ~~Critical~~ **Mitigated** |

---

## High

### FI-2 — Collateral data not locked post-facility

**Capability affected**: Smart Form (Field Lockpoint Groups)

The `DISBURSEMENT_CHANNEL` lockpoint group locks at `Create Facility`. But collateral fields (vehicle make/model/year/grade — fed to Dashi; land parcel ID) are not in any lockpoint group. After `Create Facility` executes, a return from QA → Draft allows a CO to change collateral details. Re-submission calls Dashi with new collateral data, produces a new valuation, and re-runs risk rules. The facility in Core Banking was created against the original collateral — the loan disbursed is now against different collateral than what Core Banking holds. This is both a fraud vector and a collateral integrity failure.

**Recommendation**: Add a `COLLATERAL` lockpoint group to the Smart Form Field Lockpoint Groups, locking at `Create Facility` (same threshold as `DISBURSEMENT_CHANNEL`). Collateral is part of the facility contract.

---

### AC-1 — Supervisor recall unrestricted by financial commitment state

**Capability affected**: Underwriting Workflow (Return Paths to Draft)

The Return Paths table allows supervisor recall from "any active state." This includes `Create Facility` and states within the Decision phase after disbursement. Recalling after disbursement creates a critical state inconsistency: money has moved in Core Banking, but the application is back in Draft. No reconciliation path for this scenario is documented.

**Recommendation**: Restrict supervisor recall to states with HWM < `Create Facility` (i.e., before financial commitment). Once HWM ≥ `Create Facility`, supervisor recall must be blocked at the workflow level. Any reversal post-facility requires an explicit Cancellation workflow with a Core Banking reversal command — not a Draft return. Update the Open Question in Underwriting Workflow CAPABILITY.md.

---

### AC-2 — RBAC model for workflow transitions not defined

**Capability affected**: Underwriting Workflow

No document defines which roles can trigger which transitions. There is no stated prevention of a CO approving their own application, a CO triggering a transition that requires an Area Manager, or any other separation-of-duties requirement. The approval authority table in Risk Assessment Engine maps risk levels to approver roles — but there is no corresponding enforcement of those role gates in the Underwriting Workflow capability.

**Recommendation**: Define a `Transition Authorization Table` in the Underwriting Workflow CAPABILITY.md mapping each state transition to: (1) the required role, and (2) the separation-of-duties rule (e.g., "the approver cannot be the same user who created the application"). This is a core regulatory requirement for loan origination.

---

### IS-2 — Core Banking idempotency assumed, not a documented requirement

**Capability affected**: Underwriting Workflow (Execution Step Idempotency Guards)

Idempotency guards for Create Facility and Create Loan + Disbursement check `facility_id` and `disbursement_id` on the Onigiri application record. These guards live inside Onigiri. If the Core Banking integration is called directly (bypassing the execution step layer), or if Onigiri's guards fail silently, there is no backstop. The PRODUCT.md mentions Core Banking as an integration target but imposes no idempotency requirement on it.

**Recommendation**: Add an explicit integration contract to PRODUCT.md Product Boundary: Core Banking must accept `application_id` as an idempotency key on Create Facility and Create Loan + Disbursement commands. Duplicate calls with the same `application_id` must return the existing record, not create new entries. This is a hard architectural constraint on Core Banking, not a nice-to-have.

---

### IS-4 — Dashi response carries no integrity guarantee

**Capability affected**: PRODUCT.md Integration Map (Dashi boundary)

Dashi returns `{ vehicle_id, canonical_rate, rate_basis, adjustment_applied }` via REST. A higher `canonical_rate` inflates collateral valuation, which increases the approved loan amount. There is no documented response signing, TLS certificate pinning, or response schema validation. Exploitability requires a MITM position, but in a branch operations context with local network segments, lateral movement into the integration layer is a plausible threat.

**Recommendation**: Dashi responses must be schema-validated against a defined envelope before being written to the application record. Consider response signing for the `canonical_rate` field specifically, since it directly influences credit decisions. Define this as an integration contract in PRODUCT.md.

---

## Medium

### FI-3 — NCB re-inquiry on re-submission

**Capability affected**: Smart Form (NCB Consent + OTP Flow)

NCB consent is collected via OTP in the Borrower stage. When an application returns to Draft and re-submits, it re-enters Risk Assessment. It is undocumented whether a new NCB inquiry is triggered on re-entry or whether the prior result is cached. If a new inquiry is triggered: (1) the original OTP consent scoped to a single inquiry may not cover a second pull, creating a regulatory consent violation, and (2) NCB inquiries incur a per-query cost.

**Recommendation**: Cache the NCB result on the application record with a TTL (e.g., 30 days). On re-entry to Risk Assessment, reuse the cached result if within TTL. A new consent + OTP is required only if the cached result has expired. Add as an Open Question in Smart Form CAPABILITY.md.

---

### DI-1 — Application JSON not snapshotted at Risk Assessment entry

**Capability affected**: Risk Assessment Engine

The Risk Assessment Engine evaluates the live application JSON from DocumentDB. If a CO saves a field edit while evaluation is running asynchronously, the evaluation trace log and the data it evaluated diverge. The evaluation trace is intended to be auditable and reproducible — but if the JSON it evaluated is mutable, reproducibility cannot be guaranteed.

**Recommendation**: On every transition to Risk Assessment state, snapshot the full application JSON into an immutable `assessment_snapshot` record (append-only, keyed by `application_id + assessment_sequence`). The evaluation trace log references the snapshot ID, not the live document. Re-runs always evaluate against the snapshot.

---

### DI-2 — No input sanitization documented for JMESPath evaluation inputs

**Capability affected**: Risk Assessment Engine

JMESPath expressions are configured by Risk Officers and evaluated against borrower-supplied application data. JMESPath is read-only and cannot modify data, but if a borrower input yields an unexpected data type where the expression expects a scalar (e.g., an array where a string is expected), evaluation could produce a null result or exception. The aggregation model takes `max(risk_level)` — a skipped rule silently reduces the effective risk level.

**Recommendation**: Define the expected JSON schema for the application object evaluated by the risk engine. Validate the application JSON against this schema before execution step entry into Risk Assessment. A schema validation failure blocks Risk Assessment entry and returns the application to Draft with a validation error.

---

### IS-3 — Miso response not schema-validated

**Capability affected**: Underwriting Workflow / Risk Assessment Engine (Miso integration boundary)

Onigiri receives `{ rating, risk_band, indicators[], model_version, trace_id }` from Miso. If Miso returns an unexpected schema (null `rating`, out-of-range `risk_band`, missing `model_version`), it is undocumented what happens to Risk Assessment execution. A null or missing score could cause Miso-dependent rules to skip, reducing the effective risk level silently.

**Recommendation**: Validate Miso's response against the defined schema before it is written into the application record. On schema violation, halt Risk Assessment, log the error, and raise an alert. Do not proceed with incomplete scoring data.

---

### AU-1 — Audit log immutability mechanism not specified

**Capability affected**: Underwriting Workflow (Workflow Audit Log)

The Workflow Audit Log NFR states "immutable RDS record" but no mechanism is defined for how immutability is enforced — no mention of append-only table design, revoked UPDATE/DELETE grants for the application service role, or external audit log export. A database administrator with write access could modify the audit trail.

**Recommendation**: Specify the immutability mechanism: application service role has INSERT-only grants on the audit log table. No UPDATE or DELETE at the application layer. Export audit events to an immutable external ledger (e.g., append-only S3 bucket with object lock) for regulatory-grade tamper evidence. Document in the Workflow Audit Log NFR.

---

## Mitigated

### IS-1 — No webhook authentication on Matcha/Wasabi callbacks ~~Critical~~ → **Mitigated**

**Fixed in**: audit session (2026-03-09)

**Mechanism**: Added `Inbound Callback Authentication` business rule to Underwriting Workflow CAPABILITY.md.
- HMAC-SHA256 signature required on all inbound callbacks from Matcha and Wasabi
- Shared secret stored in Onigiri secrets manager
- Four-condition enforcement table: missing signature → 401 + alert; invalid signature → 403 + alert; valid but stale (>5 min) → 400 + replay log; valid → accept and trigger transition
- No state transition fires without a valid, timely signature

---

### AI-1 — Risk rule deactivation has no dual-authorization gate ~~High~~ → **Mitigated**

**Fixed in**: audit session (2026-03-09)

**Mechanism**: Added `Rule Change Authorization` business rule to Risk Assessment Engine CAPABILITY.md.
- Two-tier approval workflow for all rule changes: Tier 1 = Risk Officer (functional owner); Tier 2 = CRO
- Auto-decline class (risk_level ≥ 70): same two tiers + Compliance notification
- Campaign coupling: strategy change + new campaign version must be submitted as a combined approval package; CRO approves both in one Tier 2 decision
- All changes recorded in immutable audit trail with actor ID, approver IDs, and affected campaign IDs

---

### AI-2 — Campaign publication has no approval workflow ~~High~~ → **Mitigated**

**Fixed in**: audit session (2026-03-09)

**Mechanism**: Added `Campaign Publication Authorization` business rule to Loan Campaign Configuration CAPABILITY.md.
- Two-tier approval workflow: Tier 1 = CPO; Tier 2 = CRO — product managers cannot publish directly
- Campaign lifecycle states: Draft (editable) → Pending Approval (read-only) → ACTIVE (append-only) → Archived
- Risk strategy coupling: campaign version assigning a new/changed risk strategy must be submitted as a combined approval package with that strategy change — CRO approves both together
- Resolved Open Question: "Is there a campaign approval workflow before publishing?" → Yes
- User flow diagram updated with CPO and CRO review gates

---

### FI-1 — Double disbursement via disbursement channel change post-facility ~~Critical~~ → **Mitigated**

**Fixed in**: CHANGELOG_003 (2026-03-09)

**Mechanism**: Three-layer defense applied.
1. `DISBURSEMENT_CHANNEL` lockpoint group added to Smart Form — locks at `Create Facility` HWM; field is read-only in all subsequent Draft states
2. Execution step idempotency guards added to Underwriting Workflow — Create Facility skips if `facility_id` exists; Create Loan + Disbursement hard-blocks if `disbursement_id` exists
3. Core Banking idempotency contract documented as a requirement (IS-2 above tracks enforcement)

---

## Priority Order for Next Action

| Priority | Finding | Action Required |
|----------|---------|-----------------|
| ~~1~~ | ~~IS-1~~ | ~~Document Matcha/Wasabi webhook authentication contract~~ — **Done** |
| ~~4~~ | ~~AI-1~~ | ~~Add dual-authorization constraint for auto-decline rule changes~~ — **Done** |
| ~~5~~ | ~~AI-2~~ | ~~Define campaign publication approval requirement~~ — **Done** |
| 1 | AC-2 | Define Transition Authorization Table in Underwriting Workflow CAPABILITY.md |
| 2 | FI-2 | Add `COLLATERAL` lockpoint group to Smart Form CAPABILITY.md |
| 3 | AC-1 | Restrict supervisor recall scope in Underwriting Workflow CAPABILITY.md; resolve Open Question |
| 4 | IS-2 | Add Core Banking idempotency contract to PRODUCT.md Product Boundary |
| 5 | DI-1 | Define assessment snapshot requirement in Risk Assessment Engine CAPABILITY.md |
