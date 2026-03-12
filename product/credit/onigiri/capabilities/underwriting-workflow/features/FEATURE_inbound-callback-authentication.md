# Feature: Inbound Callback Authentication

**Parent Capability**: Underwriting Workflow — [CAPABILITY](../CAPABILITY.md)
**Product**: Onigiri — [PRODUCT](../../../PRODUCT.md)
**Engineering Owner**: TBD
**Status**: Spec
**Changelog Reference**: CHANGELOG_004 — Group A (IS-1)
**Last Updated**: 2026-03-10

---

## User Story

As the **Onigiri platform**, I want to verify the authenticity of every inbound callback from Matcha and Wasabi before triggering any workflow state transition, so that a network-level attacker cannot forge a callback to advance a loan application past a human control gate without actual credentials.

## Job-to-be-Done

Matcha and Wasabi are external services that trigger state transitions inside Onigiri via HTTP callbacks. These callbacks carry no current authentication. A forged `APPROVED` callback from Matcha bypasses document QA entirely — no credential theft required, only network access. This feature closes that gap by requiring HMAC-SHA256 signature verification on every inbound callback before any transition logic executes.

---

## Acceptance Criteria

| # | Criterion | Pass Condition |
|---|-----------|---------------|
| AC-1 | Callback with no `X-Onigiri-Signature` header | Returns HTTP 401; security alert raised; no state transition triggered |
| AC-2 | Callback with a syntactically malformed signature | Returns HTTP 403; security alert raised; no state transition triggered |
| AC-3 | Callback with a valid signature but `timestamp` > 5 minutes old | Returns HTTP 400; replay attempt logged; no state transition triggered |
| AC-4 | Callback with a valid signature and `timestamp` within the 5-minute window | Returns HTTP 200; state transition proceeds normally |
| AC-5 | Matcha and Wasabi each have independent secrets | Compromising one secret does not affect the other caller's callbacks |
| AC-6 | Secrets are retrieved from the secrets manager at runtime | No secret is hardcoded in source code or config files |
| AC-7 | Secret rotation: during a rotation window, both old and new secrets are accepted | Zero-downtime secret rotation is possible without dropping valid callbacks |
| AC-8 | Security alerts (AC-1, AC-2) are observable | Alerts are emitted to the monitoring/alerting system with caller ID, endpoint, and timestamp |

---

## Signature Contract

The HMAC is computed over the canonical string:

```
{caller_id}.{timestamp_unix}.{sha256_hex(raw_request_body)}
```

- `caller_id`: `matcha` or `wasabi` (agreed at onboarding)
- `timestamp_unix`: Unix epoch seconds, included in the request body or a separate header
- `sha256_hex(raw_request_body)`: hex-encoded SHA-256 hash of the raw (unparsed) request body

Signature is delivered in the `X-Onigiri-Signature` request header as:

```
X-Onigiri-Signature: sha256={hex_encoded_hmac}
```

**Onigiri's responsibility**: Verify the signature. Reject if invalid.
**Matcha/Wasabi's responsibility**: Compute and include the signature. (Out of scope for this feature — coordinate with those teams.)

---

## Edge Cases & Error States

| Scenario | Expected Behavior |
|----------|------------------|
| Callback body is empty | Treat as `sha256("")` — still subject to full signature verification |
| Clock skew between caller and Onigiri | Allow ±30 seconds tolerance within the 5-minute window |
| Same valid request replayed within the 5-minute window | Accept (idempotency is handled at the state transition layer, not here) |
| Secrets manager unavailable | Fail closed — return HTTP 503; do not fall back to accepting unsigned callbacks |
| Unknown `caller_id` | Return HTTP 401; log unknown caller; no state transition |

---

## Dependencies

| Dependency | Type | Notes |
|------------|------|-------|
| Secrets manager integration | Internal platform | Secret storage and retrieval infrastructure must be available |
| Matcha team | External team | Must implement the signing side of this contract |
| Wasabi team | External team | Must implement the signing side of this contract |
| Workflow state machine | Internal | Signature verification is a pre-condition gate — state transition logic is unchanged |
| Monitoring / alerting system | Internal platform | Required for AC-8 alert emission |

---

## Out of Scope

- Changes to Matcha or Wasabi source code — this feature covers Onigiri's verification side only
- Rotation UI — secret rotation is an operational procedure; this feature only requires dual-accept at the verification layer
- Payload-level validation (schema validation of the callback body) — separate concern

---

## NFR Escalations (to Capability layer)

- Latency: HMAC verification must add < 5ms to callback processing time (secrets should be cached, not fetched per-request)
- Availability: Secrets manager cache must support graceful degradation window (e.g., 60-second stale cache) before failing closed
