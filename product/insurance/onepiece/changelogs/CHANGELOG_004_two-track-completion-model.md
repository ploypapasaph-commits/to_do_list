# Changelog 004: Two-Track Completion Model

> **Date:** 2026-03-05
> **Layer Affected:** Capability
> **Author:** AI Assistant

---

## What Changed

### Capability Layer (Application Management)

- **Replaced sequential state machine** with a **two-track completion model**: Document Upload track and Payment track
- Both tracks must complete before policy issuance can proceed
- **Flow mode** is configurable per sale channel x product combination (insurer, product type, coverage year):
  - **No docs required**: Payment track only
  - **Parallel**: Both tracks start simultaneously
  - **Sequential docs first**: Document upload must complete before payment begins
  - **Sequential payment first**: Payment must complete before document upload begins
- Updated flowchart from linear stateDiagram to branching flowchart showing all 4 flow modes
- Added business rules AM-006 through AM-010 for flow mode configuration and issuance gate
- Updated Feature #3 (Application State Machine) description to reflect two-track model
- Updated open questions

---

## Key Corrections from Previous State

- Previous model assumed a fixed sequence: DocumentUpload -> Submitted -> PaymentPending -> PaymentConfirmed
- In reality, document upload and payment can be **parallel** for some products
- The ordering (parallel vs. sequential, and which direction) is **not hardcoded** -- it is configured per sale channel x product combination
- This flexibility applies to **both branch and online** channels, not just branch

---

## Links

- [Application Management CAPABILITY.md](../capabilities/application-management/CAPABILITY.md)
