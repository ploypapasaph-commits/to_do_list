# Changelog 007: DaVinci Pre-fill Branch-Only with Selective Control

> **Date:** 2026-03-05
> **Layer:** Capability (Application Management)
> **Status:** Documented

---

## What Changed

Clarified that DaVinci customer data pre-fill is available only in the branch sale channel. Online applications do not have pre-fill.

Additionally, branch staff must be able to choose which parts of the information they want to pre-fill from DaVinci — it is not an all-or-nothing operation.

## Documents Updated

- `capabilities/application-management/CAPABILITY.md`
  - Feature #1 (Application Creation): updated description to note branch-only pre-fill and no pre-fill for online
  - Business rule AM-001: scoped to branch channel only; changed from automatic to staff-triggered
  - Business rule AM-013 (new): selective pre-fill — staff chooses which sections/fields to populate from DaVinci

## Rationale

Branch staff operate on behalf of walk-in customers and need control over which data gets pulled from DaVinci (e.g. customer may have updated address not yet reflected in DaVinci, or staff may want to enter vehicle details manually). Online customers enter their own data directly and do not have access to DaVinci pre-fill.
