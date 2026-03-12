# Changelog 005: Configurable Completion Steps

> **Date:** 2026-03-05
> **Layer:** Capability
> **Affected:** Application Management (`CAPABILITY.md`)

---

## What Changed

Replaced the hardcoded two-track completion model (document upload + payment) with a **generic, configurable completion step model**.

### Before

- Application state machine was built around exactly two tracks: document upload and payment.
- Flow mode toggled ordering between these two fixed tracks.
- Could not accommodate insurer-specific processes beyond documents and payment.

### After

- **Completion steps** are now a configurable list per insurer × product × channel combination.
- Step types are extensible (e.g. `document_upload`, `payment`, `health_declaration`, `beneficiary_nomination`, `vehicle_inspection`, `consent_signature`, `additional_info`).
- Flow modes generalized: Single step, Parallel, Sequential, and Grouped (parallel within groups, sequential between groups).
- Each step has its own lifecycle: Blocked → Pending → InProgress → Completed (or Skipped).
- State machine is generic — processes a step queue without encoding specific step types.
- New business rules added: AM-011 (step extensibility), AM-012 (step skip/waiver).

## Rationale

Each insurer has different requirements for what information and processes are needed before policy issuance. Hardcoding only document upload and payment would require state machine changes every time a new insurer or product type with different requirements is onboarded. The configurable model allows new step types to be added without modifying the core state machine logic.

## Decision Log

- **Alternative considered:** Keep two tracks but add optional sub-steps within each. Rejected because it still assumes all processes fit into "documents" or "payment" categories, which breaks for health declarations, beneficiary nominations, etc.
- **Alternative considered:** Free-form workflow engine. Rejected as over-engineering — the four flow modes (single, parallel, sequential, grouped) cover all known patterns without requiring a full workflow DSL.
