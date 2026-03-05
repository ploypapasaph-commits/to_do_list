# Capability: Contact Compliance

**Product**: Sensei — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Operations
**Product Owner**: TBD (Operations PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-04

---

## Business Function

Ensure all customer interactions comply with Thai debt collection regulations regarding contact frequency, timing, and methods — by checking DaVinci contact log facts before generating contact tasks, enforcing business hour windows, and verifying recorded call outcomes via 3CX cross-reference.

## ⚠️ Critical Boundary Note

**This capability does NOT own contact compliance data.**

- Contact frequency enforcement is based on the **BOS collection note log** — Sensei queries this log fact at task generation time to determine whether the daily contact limit has been reached. *(BOS collection note log existence and API contract TBD — see Open Questions.)*
- Sensei is a **consumer** of contact data, not an owner. It does not maintain its own contact count.
- Sensei feeds contact outcomes back to DaVinci (ContactRecorded event) so the centralized cross-product count stays accurate.
- **3CX is for trust-but-verify call validation only** — it is not the source for contact frequency checking.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Contact Limit Pre-Check | Draft | Query BOS collection note log at task generation time; count today's contacts for the customer — if count ≥ daily limit, task is not generated and contract is flagged in supervisor exception panel |
| Contact Window Enforcement | Draft | Subscribe to ContactWindowClosed event; do not present contact tasks outside business hours |
| ContactRecorded Feedback | Draft | Publish ContactRecorded event to DaVinci after every contact task completion |
| 3CX Call Log Verification | Draft | Cross-reference recorded Call outcomes against 3CX call logs after task completion (trust-but-verify — separate from contact frequency check) |
| Verification Status Display | Draft | Show verification status on each Call task: ✅ Verified / ⚠️ Unverified / ❌ Mismatch |
| Supervisor Mismatch Panel | Draft | Surface Unverified and Mismatch tasks in supervisor exception panel for review |

---

## Business Rules

### Contact Limit Pre-Check Rule

Before generating a contact task for a customer, Sensei queries the BOS collection note log:

```
GET /bos/customers/{customer_id}/collection-notes/today
→ { contact_count: N, daily_limit: M }
```

| Result | Sensei Behavior |
|--------|----------------|
| `contact_count < daily_limit` | Task is generated normally |
| `contact_count >= daily_limit` | Task is **not generated**; contract flagged in supervisor exception panel as "Contact limit reached" |

> ⚠️ **Dependency**: BOS collection note log API — existence and contract TBD (see Open Questions).

### Contact Window Rule

| Event | Sensei Response |
|-------|----------------|
| `ContactWindowClosed` | Do not present contact tasks outside business hours; CO sees "Contact window closed" indicator |

### ContactRecorded Feedback Rule

When Sensei records a contact outcome (CO completes a Call or Visit task), it **must** publish a `ContactRecorded` event to DaVinci so the centralized contact count stays accurate.

```
ContactRecorded {
  customer_id       // DaVinci customer ID
  channel           // call | visit | sms | email
  subsidiary_id     // originating subsidiary
  timestamp         // when contact was made
  outcome           // CO-recorded outcome
  task_id           // Sensei task ID for cross-reference
}
```

### 3CX Call Log Verification Rules (Trust-but-Verify)

| Verification Check | Criteria |
|-------------------|---------|
| Call placed | Outbound call record exists in 3CX log |
| Duration > 0 | Call connected and lasted > 0 seconds |
| Caller ID match | CO's phone number matches logged caller ID |
| Timestamp match | Call timestamp within ±5 minutes of task completion time |

| Verification Status | Meaning | Supervisor Action |
|--------------------|---------|--------------------|
| ✅ Verified | All criteria met — call log matches | No action needed |
| ⚠️ Unverified | No matching log found | Surface in exception panel for review |
| ❌ Mismatch | Log contradicts outcome (e.g., "PTP" recorded but call duration was 0 seconds) | Surface in exception panel; may require investigation |

### Trust-but-Verify Design Rule

3CX verification does **not** block CO from recording outcomes in real-time — it performs after-the-fact verification. This maintains throughput while enabling accountability. Mismatches are flagged for supervisor review, not auto-reversed.

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Pre-check latency | BOS collection note log query at task generation must complete within 500ms; on timeout, fail-safe behavior applies (see Open Questions) |
| No data ownership | Sensei must not maintain its own contact frequency count — always read from BOS collection note log |
| ContactRecorded always published | On every contact task closure, ContactRecorded must be published to DaVinci (not optional) |
| Non-blocking verification | 3CX verification must not block CO workflow; runs asynchronously after task closure |

---

## Open Questions

- **BOS collection note log**: Does this log already exist in BOS? What is the API contract for querying today's contact count per customer? This is a **critical dependency** for the Contact Limit Pre-Check feature — if the log does not exist, this feature cannot be built without BOS delivering it first.
- **Pre-check fail-safe**: If the BOS collection note log query times out at task generation time, should Sensei (a) suppress the task (conservative — never over-contact), (b) create the task anyway (permissive — maintain throughput), or (c) retry with backoff?
- **3CX API unavailability**: If 3CX API is unavailable, should missing call verification default to "Unverified" or be left blank?
