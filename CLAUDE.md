# SYSTEM: PRODUCT DEVELOPMENT ENGINE

*Documentation as Truth. Atomic Architecture. Nothing skips a layer: Feature → Capability → Product → Portfolio.*

---

## SESSION BOOTSTRAP — RUN FIRST, EVERY SESSION

1. Read `PORTFOLIO_CATALOG.md`.
2. Read the in-scope product's `BACKLOG.md`.
3. Ask the user **before any other action**:

> **"Are you (A) updating implementation status — marking features as built, shipped, or deprecated — or (B) building out new capabilities or features?"**

Wait for an explicit answer. Do not assume.

| Answer | Mode activated |
|---|---|
| **A — Updating status** | `STATUS_UPDATE` (see below) |
| **B — New work** | `@PORTFOLIO` / `@PRODUCT` / `@CAPABILITY` / `@FEATURE` modes |

---

## THE FOUR LAYERS

| Layer | Owner | Definition |
|---|---|---|
| **Feature** | Engineering | Smallest independently deliverable unit of value. Single purpose. Independently buildable and testable. |
| **Capability** | Product Owner | Cohesive group of features enabling a named business function. Unit of PO accountability. |
| **Product** | CPO / Executive | Self-contained offering with defined customer value, explicit boundaries, and its own lifecycle. |
| **Portfolio** | Strategic Investment | Collection of related products serving a strategic business domain. Unit of investment and resource allocation. |

**Rule:** Before responding to any request — identify its layer. A capability is not a product. A feature is not a capability.

---

## FILE SYSTEM

```
/product/
├── PORTFOLIO_CATALOG.md                  ← Master registry. Read first.
└── <portfolio-name>/
    ├── PORTFOLIO.md
    └── <product-name>/
        ├── PRODUCT.md                    ← Source of truth for the product.
        ├── BACKLOG.md                    ← ★ Index of all backlog items. Read every session.
        ├── ARCHITECTURE.md               ← (Optional) Technical design.
        ├── backlog/                      ← ★ One file per backlog item.
        │   └── ITEM_<name>.md            ← Lightweight item card. Promoted to capability on Live.
        ├── capabilities/
        │   └── <capability-name>/
        │       ├── CAPABILITY.md
        │       └── features/
        │           └── FEATURE_<name>.md
        ├── changelogs/
        │   └── CHANGELOG_NNN_<name>.md   ← Append-only.
        └── artifacts/
            ├── diagrams/
            └── research/
```

---

## DOCUMENT SCHEMAS

### PORTFOLIO_CATALOG.md
Master entry point. Contains: portfolio names, product names, statuses, links, one-line summaries.

### PORTFOLIO.md
Contains: portfolio name/owner, investment thesis, constituent products and statuses, portfolio OKRs, cross-product dependencies, Now/Next/Later roadmap, key risks.

### PRODUCT.md
Contains: product name/codename/status/owner, problem statement, value proposition, IS/IS NOT boundary definition, capability registry (table), integration map (inputs/outputs/mechanisms), product metrics/KPIs, product-level Mermaid diagrams.

### BACKLOG.md ★
Index of all backlog items. Updated at the end of every session. Each row links to its `ITEM_<name>.md` file in `backlog/`. Statuses here must always match the corresponding `ITEM_<name>.md` and `FEATURE_<name>.md`. If they conflict, `BACKLOG.md` is stale — fix it.

**Schema — use exactly:**

```markdown
# BACKLOG: <Product Name>

**Product:** <Codename> — <Full Name>
**Portfolio:** <Portfolio Name>
**Last Updated:** YYYY-MM-DD
**Last Session:** <one-line summary>

---

## ✅ LIVE
| Feature | Capability | Shipped | Item File |
|---|---|---|---|

## 🔨 IN PROGRESS
| Feature | Capability | Owner | Started | Item File |
|---|---|---|---|---|

## 📋 SPEC'D (ready to build)
| Feature | Capability | Priority | Item File |
|---|---|---|---|

## 💡 CONCEPT (not yet specced)
| Feature | Capability | Item File |
|---|---|---|

## 🗄️ DEPRECATED
| Feature | Capability | Date | Item File |
|---|---|---|---|

---

## SESSION LOG
| Date | Type | Summary | Docs Changed |
|---|---|---|---|
```

### ITEM_\<name\>.md
One file per backlog item, stored in `backlog/`. Lightweight card that travels with the item through its lifecycle. When the item reaches `✅ LIVE`, its content is merged into the relevant `CAPABILITY.md` and `FEATURE_<name>.md`, then the file is archived (moved to `backlog/archived/`).

**Schema — use exactly:**

```markdown
# ITEM: <Feature Name>

**Status:** Concept / Spec'd / In-Dev / Live / Deprecated
**Capability:** <parent capability name>
**Product:** <Codename>
**Owner:** <name or team>
**Created:** YYYY-MM-DD
**Last Updated:** YYYY-MM-DD

---

## What & Why
<One paragraph: what this item does and the business reason it exists.>

## Acceptance Criteria
- [ ] <testable condition>
- [ ] <testable condition>

## Dependencies
- <other item or system this depends on>

## Notes / Open Questions
- <any outstanding decisions or blockers>

---

## Merge Checklist
*Complete before marking Live and archiving this file.*
- [ ] `FEATURE_<name>.md` created or updated in `capabilities/<capability>/features/`
- [ ] `CAPABILITY.md` feature inventory updated
- [ ] `PRODUCT.md` capability registry updated (if new capability)
- [ ] `BACKLOG.md` row moved to ✅ LIVE
- [ ] `CHANGELOG` entry written
- [ ] This file moved to `backlog/archived/`
```

### CAPABILITY.md
Contains: capability name/product/owner, business function description, feature inventory table (name + status), business rules as decision tables, Mermaid flow diagrams (happy path + exceptions), NFRs, open questions.

### FEATURE_\<name\>.md
Contains: feature name/capability/owner, user story (`As a [persona], I want [action], so that [outcome]`), job-to-be-done, acceptance criteria (testable, pass/fail), edge cases and error states, dependencies, status: `Concept` / `Spec` / `In-Dev` / `Live` / `Deprecated`.

### ARCHITECTURE.md
Contains: API contracts, request/response schemas, data models, infrastructure decisions, code patterns, ADRs. No business justification or user stories.

### CHANGELOG_NNN_\<name\>.md
Append-only. Contains: layer affected, what changed, rationale, decision log, links to updated documents.

---

## STATUS_UPDATE MODE

*Activated on answer A at session start.*

1. Ask: *"Which features have changed status since the last session?"*
2. For each **newly conceived** item: create `backlog/ITEM_<name>.md` using the schema; add row to `💡 CONCEPT` in `BACKLOG.md`.
3. For each **in-progress** item: set `ITEM_<name>.md` status → `In-Dev`; move row to `🔨 IN PROGRESS` in `BACKLOG.md`.
4. For each **implemented** item — run the Merge Protocol:
   - Confirm all Merge Checklist items in `ITEM_<name>.md` are done.
   - Update or create `FEATURE_<name>.md` in `capabilities/<capability>/features/`.
   - Update the feature inventory table in the parent `CAPABILITY.md`.
   - If a new capability was introduced, update the capability registry in `PRODUCT.md`.
   - Move the `BACKLOG.md` row to `✅ LIVE`.
   - Move `ITEM_<name>.md` to `backlog/archived/`.
5. Update `BACKLOG.md` header (`Last Updated`, `Last Session`) and append to `SESSION LOG`.
6. Create a `CHANGELOG` entry for all changes.

**Do not** add features, change specs, or redesign capabilities in this mode. If new work surfaces, create an `ITEM_<name>.md` under `💡 CONCEPT` and defer to a `@CAPABILITY` or `@FEATURE` session.

---

## AI MODES

Prefix every instruction with the mode tag. If no tag is given, ask which layer before proceeding.

### @PORTFOLIO
1. Read `PORTFOLIO_CATALOG.md`.
2. Identify or propose the portfolio.
3. Check cross-product impacts: shared capabilities, data flows, dependency chains.
4. Enforce coherence: no duplication across products, no strategic gaps.

**Outputs:** `PORTFOLIO_CATALOG.md` update, `PORTFOLIO.md` create/update.

### @PRODUCT
1. Read `PRODUCT.md` (create if absent).
2. Duplication check: does this capability exist in another product? Reuse vs. build — document the decision.
3. Boundary enforcement: define IS/IS NOT for any cross-product request.
4. Classify the request: new capability, extension, or refinement?

**Outputs:** `PRODUCT.md` update, capability registry change, `PORTFOLIO_CATALOG.md` status update.

### @CAPABILITY
1. Read `CAPABILITY.md` (create if absent).
2. Verify scope — flag any cross-capability boundary explicitly.
3. Decompose into independently deliverable features.
4. Write business rules as decision tables, not prose.
5. Map happy path + all exception paths as Mermaid diagrams.
6. Identify NFRs (latency, scale, consistency, auditability).

**Outputs:** `CAPABILITY.md` create/update, `PRODUCT.md` capability registry update, `ITEM_<name>.md` created in `backlog/` for each new feature, `BACKLOG.md` updated with new `💡 CONCEPT` or `📋 SPEC'D` entries.

### @FEATURE
1. Decompose to single purpose. If two values, split.
2. Write user story (persona / action / outcome).
3. Write acceptance criteria — every line must be pass/fail verifiable.
4. Document all edge cases and error states.
5. Escalate NFRs to `@CAPABILITY`.
6. Update parent `CAPABILITY.md` feature inventory.
7. Add to `BACKLOG.md` under `📋 SPEC'D`.

**Outputs:** `FEATURE_<name>.md` create/update, `ITEM_<name>.md` created in `backlog/` (status `📋 SPEC'D`), `CAPABILITY.md` inventory update, `BACKLOG.md` update, `CHANGELOG` entry.

---

## CROSS-CUTTING RULES

**Before every response:** Why does this exist? Which layer? Does it already exist somewhere? What is the business value?

**Every decision:** Document the reasoning. Enumerate trade-offs including rejected alternatives. Assess reversibility. No "best practice" without evidence.

**Every session:** Produce a `CHANGELOG` entry. Update `BACKLOG.md` (last updated, last session, session log row).

**Diagrams (Mermaid.js, required at all layers):** Portfolio → product relationship/data flow maps. Product → capability wheel, integration map, user journeys. Capability → flow diagrams, state machines, decision trees. Feature → state diagrams, edge case flows. Store in `artifacts/diagrams/` or inline.

---

## PRODUCT BOUNDARY TEMPLATE

```
## Product Boundary: <Product Name>

IS responsible for:
- [scope item]

IS NOT responsible for:
- [out-of-scope item — note which product owns it]

RECEIVES from:
- [system] → [what] → [event / API / batch]

SENDS to:
- [system] → [what] → [event / API / batch]
```

---

## CURRENT PRODUCT STATE

Structure lives under `/product/`. `PORTFOLIO_CATALOG.md` is the master entry point. Products migrating from flat `ATLAS.md` — during migration, `ATLAS.md` is the source for extracting capabilities and features.

| Codename | Product | Status | Notes |
|---|---|---|---|
| **Onigiri** | Loan Origination System | Draft | Credit Portfolio. Application → underwriting → decision. |
| **Matcha** | Document Verification Service | Active | Cross-portfolio shared service. Owned by Operations. |
| **Wasabi** | AI Document Verification | Draft | Matcha's AI routing layer. Same portfolio as Matcha. |
| **DaVinci** | Customer & Product Master Data | Draft | Platform-level shared service. Data/Platform Portfolio. |
| **Sensei** | Branch Operations Orchestration | Draft | Operations Portfolio. |

**Next action:** Run `@PORTFOLIO` to define portfolio structure and update `PORTFOLIO_CATALOG.md` before any product-level work.

---

## LAYER EXAMPLE

```
Portfolio:  Credit
            └── Product: Loan Origination (Onigiri)       ← ends at disbursement
            │            ├── Capability: Application Management
            │            │   ├── Feature: Smart Form Builder
            │            │   ├── Feature: Application State Machine
            │            │   └── Feature: Document Checklist Engine
            │            ├── Capability: Underwriting Workflow
            │            │   ├── Feature: 4-Phase Workflow Execution
            │            │   └── Feature: Risk Rule Engine
            │            └── Capability: Campaign Configuration
            │                ├── Feature: Eligibility Rule Builder
            │                └── Feature: Pricing Configuration
            └── Product: Loan Servicing     [to be defined]
            └── Product: Collections        [to be defined]
```
