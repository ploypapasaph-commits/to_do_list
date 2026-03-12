# Capability: Package Master & Channel Configuration

> **Parent Product:** OnePiece (Insurance Distribution Platform)
> **Product Owner:** TBD
> **Status:** Draft
> **Last Updated:** 2026-03-06

---

## Business Function

Manages the master configuration and reference data for all insurance packages, vehicles, insurer partnerships, channel availability rules, payment eligibility, and policy issuance method mappings. This capability is the single source of truth for "what can be sold, for which vehicles, where, how it's paid, and how the policy is issued." It provides the data that the Product Catalog (in Quotation & Comparison) presents to users. Vehicle master data is insurance-specific — other products (e.g. loans) maintain their own vehicle data sources.

---

## Feature Inventory

| # | Feature | Status | Description |
|---|---------|--------|-------------|
| 1 | Insurance Product Registry | Concept | Master list of insurance products (CMI, VMI types) with attributes |
| 2 | Insurer-Product Mapping | Concept | Configuration of which insurers offer which products, by coverage year, with available packages. Package channel settings (release date, end date, selling price) are configured **per sale channel** — see Package Pricing & Channel Configuration section below. Product managers manage packages via a management interface (create, edit, delete, release). Commission management is entirely outside the system. |
| 3 | Channel Availability Rules | Concept | Channel availability is configuration-driven — a product is available on a channel when active packages exist for that channel (controlled via per-channel release date/end date). No hard product-level or coverage-year-level channel restrictions. |
| 4 | Payment Eligibility Matrix | Concept | Configuration of eligible **payment terms** (full payment, installment) and **payment channels** (cash, QR, 2C2P variants) based on 4 dimensions: **insurer**, **product type**, **coverage year**, and **sale channel**. See Payment Eligibility Matrix section below. |
| 5 | Issuance Method Configuration | Concept | Per insurer-product mapping of policy issuance method (file transfer, API, manual) |
| 6 | Vehicle Master Data | Concept | Insurance-specific vehicle reference data sourced from Red Book. Each vehicle is identified by a unique vehicle key. Maintained via quarterly upload (mostly). Includes channel availability per vehicle — not all vehicles are available on every sale channel. |
| 7 | Vehicle-Channel Availability | Concept | Configuration of which vehicles are available on which sale channels, configured per individual vehicle (by Red Book vehicle key). Determines whether a given vehicle can be quoted and purchased through branch, online, or both. |

---

## Package Pricing & Channel Configuration

Each package has a **gross premium** (the cost — the amount transferred to the insurer) which is the same regardless of channel. However, the following attributes are configured **per package per sale channel**:

| Attribute | Scope | Description |
|-----------|-------|-------------|
| **Gross Premium** | Per package (channel-independent) | The amount to be transferred to the insurer. Think of this as the cost. |
| **Selling Price** | Per package × per channel | The price shown to the customer or branch staff. For **full payment**, this is the amount the customer pays. For **installment**, this is the **principal** (loan amount). There is no fixed relationship to gross premium — selling price can be less than, equal to, or greater than gross premium. |
| **Release Date** | Per package × per channel | The date from which the package becomes visible on this channel. |
| **End Date** | Per package × per channel | The date after which the package is no longer visible on this channel. |

> **Key implication:** The same package may have different selling prices, and different active periods, on branch vs. online. A package could be live on branch but not yet released on online, or priced differently across channels.

### Example

```
Package: CHUBB × VMI-CAR-1 × 1-year
├── Gross Premium: 15,000 THB (same for all channels)
│
├── Branch:
│   ├── Selling Price: 14,500 THB
│   ├── Release Date: 2026-01-01
│   └── End Date: 2026-12-31
│
└── Online:
    ├── Selling Price: 13,900 THB
    ├── Release Date: 2026-03-01
    └── End Date: 2026-12-31
```

---

## Payment Eligibility Matrix

Payment eligibility determines which **payment terms** (full payment, installment) and **payment channels** (cash, QR, 2C2P variants) are available for a given package. Eligibility is evaluated across **4 dimensions**:

| Dimension | Description | Example Impact |
|-----------|-------------|----------------|
| **Insurer** | Which insurer underwrites the package | Some insurers may not support installment |
| **Product Type** | The insurance product (CMI, VMI types) | CMI may have different payment options than VMI Type 1 |
| **Coverage Year** | 1 year vs. > 1 year | Multi-year coverage may restrict certain payment channels |
| **Sale Channel** | Branch or online | Online has different payment channels than branch |

### Eligibility Output

For a given combination of the 4 dimensions, the matrix returns:

1. **Eligible payment terms:** Which of {Full Payment, Installment} are available
2. **Eligible payment channels per term:** For each eligible payment term, which of {Cash, Bill Payment QR, 2C2P QR, 2C2P DPAY, 2C2P Credit Card, 2C2P IRR} are available

### Known Constraints (Current)

| Constraint | Dimensions Involved | Rule |
|------------|-------------------|------|
| Online = full payment only | Sale Channel | Online channel does not offer installment |
| Cash and Bill Payment QR are branch-only | Sale Channel | Physical payment methods require branch presence |
| No credit-based payment for installment | Payment Term × Payment Channel type | **A loan cannot be paid by another loan.** Any credit-based payment channel (2C2P CC, 2C2P IRR, or any future credit instrument) is blocked when the payment term is installment. This is a general principle, not limited to specific channels. |
| Installment requires loan approval | Payment Term | All installment payments trigger loan approval flow via Matcha |

> **Note:** The constraints above are the currently known rules. Additional insurer-specific, product-specific, or coverage-year-specific rules may exist and should be configured in the eligibility matrix.

---

## Vehicle Master Data — Car Attributes

The car master data contains the following attributes. This is insurance-specific reference data — other products (e.g. loans) maintain their own vehicle data sources.

| # | Attribute | Description |
|---|-----------|-------------|
| 1 | Auto Type | Vehicle category (e.g. car, motorcycle) |
| 2 | Body Type | Body style classification |
| 3 | Brand | Vehicle manufacturer / make |
| 4 | Model | Vehicle model |
| 5 | Sub Model | Vehicle sub-model variant |
| 6 | Gear Type | Transmission type (e.g. automatic, manual) |
| 7 | Year Group | Model year grouping |
| 8 | Door | Number of doors |
| 9 | Model Description | Descriptive text for the model |
| 10 | Fuel Type | Fuel type (e.g. petrol, diesel, electric, hybrid) |
| 11 | Import Status | Whether the vehicle is locally assembled or imported |
| 12 | Series | Vehicle series |
| 13 | Badge Secondary Description (EN) | Secondary badge description in English |
| 14 | Badge Secondary Description (TH) | Secondary badge description in Thai |
| 15 | Drive Description | Drive type (e.g. 2WD, 4WD) |
| 16 | Engine Size | Engine displacement |
| 17 | Engine Description | Engine specification details |
| 18 | Combined Power | Combined power output |
| 19 | Capacity Battery | Battery capacity (for electric/hybrid vehicles) |
| 20 | Release Year | Vehicle release year |
| 21 | Release Month | Vehicle release month |
| 22 | Plate Province | Vehicle registration province |
| 23 | Accessory | List of installed accessories |

> **Note:** Motorcycle master data attributes are not yet defined.
>
> **Note:** Search input fields (which attributes the user fills to find packages) are documented in [Quotation & Comparison](../quotation/CAPABILITY.md), as they are part of the user-facing catalog experience.

---

## Business Rules

| Rule ID | Rule | Condition | Result |
|---------|------|-----------|--------|
| PC-002 | Cash and Bill Payment QR are branch-only | Channel = Online AND Payment Channel in {Cash, Bill Payment QR} | Block: payment channel not available |
| PC-003 | Online is full payment only | Channel = Online | Installment not offered |
| PC-003a | No credit-based payment channel for installment | Payment Term = Installment AND Payment Channel is credit-based (e.g. 2C2P Credit Card, 2C2P IRR, or any future credit instrument) | Block: payment channel not available. **Principle: a loan cannot be paid by another loan.** This applies to all current and future credit-based payment channels — not just CC and IRR. |
| PC-015 | Payment eligibility is multi-dimensional | Always | Eligible payment terms and payment channels are determined by the combination of insurer, product type, coverage year, and sale channel — not by sale channel alone |
| PC-005 | Issuance method determined by insurer-product-coverage combination | Insurer = X AND Product = Y AND Coverage = Z | Return configured issuance method (REST API or Manual) |
| PC-006 | Manual issuance products are branch-only | Issuance Method = Manual AND Channel = Online | Block: not available online |
| PC-007 | Installment requires loan approval | Payment Term = Installment | Trigger loan approval flow |
| PC-008 | Vehicle-channel availability | Vehicle not available on requested channel | Block: vehicle not available for this channel |
| PC-009 | Vehicle master is insurance-specific | Always | Vehicle reference data is owned by OnePiece; not shared with other products (e.g. loans have separate vehicle data sources) |
| PC-010 | Package active date range (per channel) | Current date < release date OR current date > end date OR dates not set for this channel | Package is hidden on this channel — not visible to customers or branch staff in quotation results for this channel |
| PC-011 | Released package visibility (per channel) | Release date ≤ current date ≤ end date for this channel | Package is visible and available for quotation on this channel at the channel-specific selling price |
| PC-012 | Selling price determines payment amount | Payment Term = Full Payment | Customer pays the channel-specific selling price |
| PC-013 | Selling price as installment principal | Payment Term = Installment | Channel-specific selling price is used as the principal (loan amount) for installment calculation |
| PC-014 | Gross premium is channel-independent | Always | Gross premium (amount transferred to insurer) is the same regardless of sale channel; no fixed relationship to selling price |
| PC-016 | In-flight orders are protected from package expiration | Order submitted (form completed) AND package end date passes | Order continues; package expiration does not cancel or block in-flight orders |
| PC-017 | Vehicle-channel availability is per vehicle | Always | Channel availability is configured per individual vehicle (identified by Red Book vehicle key), not per vehicle type or make/model |

---

## Open Questions

- ~~What is the data model for insurer partnership management?~~ **Resolved:** Commission management is entirely outside the system — not stored or calculated in OnePiece.
- ~~How frequently do insurer package offerings change?~~ **Resolved:** No fixed frequency. Product managers manage packages via a management interface (create, edit, delete, release).
- ~~What vehicle characteristics are included in the master data?~~ **Resolved:** 23 car attributes documented in Vehicle Master Data — Car Attributes section above.
- ~~Which specific vehicle attributes are required as search input for each sale channel?~~ **Resolved:** Documented per channel × product type in [Quotation & Comparison](../quotation/CAPABILITY.md) capability (Search Input Fields section).
- ~~What determines vehicle-channel availability?~~ **Resolved:** Configured per individual vehicle, identified by Red Book vehicle key.
- ~~How is vehicle master data maintained?~~ **Resolved:** Quarterly upload (mostly). Data sourced from Red Book.
- ~~If a package's end date passes while there are in-flight orders using that package, should those orders still be allowed to complete or be cancelled?~~ **Resolved:** In-flight orders (form submitted) continue regardless of package expiration. No retroactive cancellation.
- ~~Is there a minimum selling price threshold for installment eligibility?~~ **Resolved:** No. Selling price is not a dimension of installment eligibility.
- ~~Are there coverage-year-specific payment restrictions?~~ **Resolved:** No coverage-year-specific payment restrictions currently exist.
- ~~How is the full payment eligibility matrix maintained?~~ **Resolved:** Data seeding for now. Management UI planned for future.
