# Capability: Loan Campaign Configuration

**Product**: Onigiri — [PRODUCT](../../PRODUCT.md)
**Portfolio**: Credit
**Product Owner**: TBD (Credit PO)
**Status**: 📝 Draft — @FEATURE decomposition pending
**Last Updated**: 2026-03-09

---

## Business Function

Define and manage loan product configurations (campaigns) that house all configuration for the loan a campaign will create — pricing, eligibility, application template, risk strategy, and workflow execution steps — enabling new loan products to be launched without code changes.

## Why It Exists (First Principles)

- **Product Launch Speed**: The business launches new loan products (campaigns) regularly — new car title products, seasonal promotions, segment-specific offers. Each needs distinct eligibility rules, pricing, risk strategies, and field requirements.
- **Operational Independence**: Product managers and credit officers need to configure new campaigns without developer involvement.
- **Single Source of Truth**: A single campaign configuration drives the **entire** application lifecycle — from which form fields appear, to which risk policies execute, to what execution steps run inside each workflow state. This prevents misalignment between intake, underwriting, and decision.

---

## Feature Inventory

| Feature | Status | Description |
|---------|--------|-------------|
| Campaign Builder | Concept | Create and manage loan campaign with all 5 configuration dimensions in one place |
| Pricing Configuration | Concept | Set loan amount range, interest rate, available tenors, max LTV, min/max credit line |
| Eligibility Rules Builder | Concept | Rule-based gateway: configure criteria (customer type, age, collateral, B-score, occupation) evaluated before full application entry |
| Application Template Assignment | Concept | Select which form pages/sections/fields appear; configure required documents; set conditional document logic |
| Risk Strategy Assignment | Concept | Assign which risk strategy (Strategy → Policy → Rule hierarchy) executes for applications under this campaign |
| Workflow Execution Steps Configuration | Concept | Configure which pluggable steps run inside each workflow state for applications under this campaign |
| Campaign Publication Approval Workflow | Spec | Two-tier approval workflow before a campaign transitions to ACTIVE — Tier 1 (parallel): CPO + Risk Officer; Tier 2: CRO. Runs on the Underwriting Workflow state machine engine. — [FEATURE](features/FEATURE_campaign-publication-authorization.md) |

---

## Business Rules

### Campaign Configuration Dimensions

| Dimension | What It Configures |
|-----------|-------------------|
| **Pricing** | Loan amount range, interest rate, available tenors, max LTV, min/max credit line |
| **Eligibility Criteria** | Rule-based gateway (customer type, age, collateral, credit score, occupation) — evaluated before full application entry |
| **Application Template** | Which pages/sections/fields appear; required documents; conditional document logic |
| **Risk Strategy** | Which risk assessment strategy to execute (Strategy → Policy → Rule) |
| **Workflow Execution Steps** | What pluggable steps run inside each workflow state |

### Pricing Parameters

| Parameter | Example Value |
|-----------|---------------|
| Loan amount range | 3,000 – 500,000 |
| Interest rate | 24% |
| Available tenors | 3, 6, 9, 12, ... months |
| Max LTV | 120% |
| Min/Max credit line | Configurable per campaign |

### Campaign Publication Authorization

Any campaign version publication (Draft → ACTIVE) requires a two-tier approval workflow. Product managers cannot publish directly.

Tier 1 is a **parallel gate** — both functional owners must approve simultaneously (in any order) before the change advances to Tier 2.

| Tier | Approver Role | Mode | Responsibility |
|------|--------------|------|----------------|
| Tier 1 | CPO (Chief Product Officer) | Parallel | Reviews all 5 configuration dimensions for business intent, pricing soundness, and eligibility correctness |
| Tier 1 | Risk Officer | Parallel | Reviews the assigned risk strategy for policy soundness and downstream evaluation impact |
| Tier 2 | CRO (Chief Risk Officer) | Sequential (after both T1) | Final authorization — mandatory for all campaign publications |

Both Tier 1 approvers must approve before the change advances to Tier 2. Either Tier 1 approver can reject, returning the campaign to Draft.

**Campaign lifecycle states:**

| State | Mutability | Description |
|-------|------------|-------------|
| Draft | Fully editable | Campaign is being configured; no version increment |
| Pending Approval | Read-only | Submitted; both T1 approvers notified simultaneously |
| Pending CRO | Read-only | Both T1 approvals received; awaiting CRO final sign-off |
| ACTIVE | Append-only | Live; any change creates a new Draft version |
| Archived | Read-only | Superseded by a newer version or manually sunset |

In-flight applications use the campaign version at submission time — no retroactive version migration.

**Risk strategy coupling:**

Because the Risk Officer is a mandatory Tier 1 approver on every campaign publication, campaign version and risk strategy alignment is enforced structurally — neither side can be published without the other functional owner's sign-off.

*Resolves audit finding AI-2 and the Open Question: "Is there a campaign approval workflow before publishing?" → **Yes.***

---

### Zero-Code Launch Rule

A new campaign must be launchable by a product manager without any code deployment. If configuring a new campaign requires a code change, it is a violation of this capability's design intent and must be escalated to engineering for resolution.

---

### Collateral Type as a Configuration Driver

Collateral Type is the primary axis that differentiates campaigns within Onigiri. A campaign is, in practice, a collateral type + credit policy combination. Every one of the five configuration dimensions is shaped by Collateral Type:

| Configuration Dimension | How Collateral Type Affects It |
|------------------------|-------------------------------|
| **Pricing** | LTV ceiling and loan amount range differ by collateral type (e.g., land carries a lower LTV ceiling than vehicles) |
| **Eligibility Criteria** | The `collateral_type =` rule gates which campaign a customer enters |
| **Application Template** | Exactly one Collateral Section (from the Smart Form Collateral Section Registry) is selected per campaign; each section carries its own field list and document declarations |
| **Risk Strategy** | A named strategy (e.g., `BikeTitleDefault`, `LandTitleDefault`) is assigned; each strategy contains policies specific to that collateral type's risk characteristics |
| **Workflow Execution Steps** | Identical across all collateral types — the fixed underwriting topology does not change by collateral type |

To launch Bike, Tractor, or Land campaigns, no new capability is required. A product manager creates a campaign, selects the appropriate Collateral Section in the Application Template, assigns the collateral-specific risk strategy, and configures pricing. Everything else is inherited from the existing infrastructure.

---

### Collateral Type Variants Reference

Authoritative cross-reference for all four supported collateral types. Each row maps a collateral type to its Smart Form section ID, Matcha document keys, and indicative pricing parameters. Values marked "(example)" require confirmation from the Credit PO and Risk Officer before campaign go-live.

| Collateral Type | Campaign Example | Smart Form Section ID | Matcha Document Keys (type-specific) | Typical LTV Ceiling | Typical Loan Amount Range |
|----------------|-----------------|----------------------|--------------------------------------|--------------------|-----------------------|
| **Car** | Car Title Default 2026 | `collateral_car` | `vehicle_registration_book`, `vehicle_insurance` | 120% (example) | 3,000 – 500,000 (example) |
| **Bike (Motorbike)** | Bike Title Default 2026 | `collateral_bike` | `motorbike_registration_book`, `motorbike_insurance` | 80% (example) | 3,000 – 150,000 (example) |
| **Tractor** | Tractor Title Default 2026 | `collateral_tractor` | `tractor_registration_document`, `proof_of_agricultural_use` | 70% (example) | 5,000 – 300,000 (example) |
| **Land** | Land Title Default 2026 | `collateral_land` | `land_title_deed`, `land_appraisal_certificate` | 50% (example) | 10,000 – 2,000,000 (example) |

**Shared Base Document Set (all collateral types):**

| Matcha Document Key | Document Name |
|--------------------|--------------|
| `applicant_id_card` | National ID Card |
| `proof_of_income` | Proof of Income (payslip / bank statement / business registration) |
| `household_registration` | Household Registration (ทะเบียนบ้าน) |

**How the document list reaches Matcha:** The Application Template for a campaign declares the required document type keys (shared base + collateral-type-specific). When the application transitions to `create_facility`, the Onigiri worker reads this list from the `document_verification_mapping` table and constructs the `documents[]` array in the POST /task payload to Matcha. Campaign configuration is therefore the upstream source of Matcha's document checklist — not a separate configuration maintained in Matcha.

---

### Eligibility Criteria Examples

The table below shows representative eligibility rules per collateral type. A campaign for one collateral type combines the collateral type gate with age, B-score, and occupation criteria.

| Criteria | Operator | Car Campaign | Bike Campaign | Tractor Campaign | Land Campaign |
|----------|----------|-------------|--------------|-----------------|--------------|
| Collateral type | `=` | `"car"` | `"bike"` | `"tractor"` | `"land"` |
| Customer age | `>=` | 20 | 20 | 20 | 20 |
| Customer age | `<=` | 70 | 70 | 70 | 70 |
| Customer type | `=` | `"new"` / `"existing"` | `"new"` / `"existing"` | `"new"` / `"existing"` | `"new"` / `"existing"` |
| B-score | `>` | 500 (example) | 500 (example) | 450 (example) | 600 (example) |
| Occupation group | `in` | Civil servant, Salaried | Civil servant, Salaried | Farmer, Agricultural worker | Any (example) |
| Vehicle brand | `like` | Toyota, Honda (example) | Honda, Yamaha, Kawasaki (example) | Kubota, Yanmar (example) | — |
| Land title deed type | `=` | — | — | — | `"Chanote"` (example) |

> **Note on land deed type:** Land campaigns may restrict eligible collateral to specific deed types (e.g., Chanote only vs. accepting NS-3K/NS-3G). This is an eligibility rule, not a form field — the gate is evaluated before the form loads. Confirm acceptable deed types per campaign with the Credit PO.

---

### Risk Strategy Names by Collateral Type

Each collateral type should have at least one named strategy in the Risk Assessment Engine. Strategies are created in the admin UI — no code deployment required. The naming convention is `<CollateralType>Title<Variant>`:

| Collateral Type | Example Strategy Name | Notes |
|----------------|----------------------|-------|
| Car | `CarTitleDefault` | Existing strategy — reference implementation |
| Bike | `BikeTitleDefault` | To be created — mirrors CarTitleDefault structure with bike-specific policies (brand, age, engine size) |
| Tractor | `TractorTitleDefault` | To be created — likely adds agricultural usage area and seasonal income policies |
| Land | `LandTitleDefault` | To be created — likely adds title deed type, land area, and appraisal value policies |

Strategy design (which policies and rules to include) is a separate @CAPABILITY session with the Risk Officer.

---

## User Flow

```mermaid
flowchart TD
    A[Product Manager opens Campaign Builder] --> B[Configure Pricing\nloan amount, rate, tenors, LTV]
    B --> C[Configure Eligibility Rules\nage, collateral type, B-score, occupation]
    C --> D[Assign Application Template\nselect form pages, sections, fields, required docs]
    D --> E[Assign Risk Strategy\nselect Strategy from Risk Assessment Engine]
    E --> F[Configure Workflow Execution Steps\nper-state step assignments]
    F --> G[Preview + Validate Campaign\ncheck for conflicts or missing required config]
    G --> H{Valid?}
    H -- Errors --> F
    H -- Valid --> I[Submit for Approval\nCampaign enters Pending Approval — read-only\nCPO + Risk Officer notified simultaneously]
    I --> J1[Tier 1a: CPO Review\nreviews all 5 config dimensions]
    I --> J2[Tier 1b: Risk Officer Review\nreviews assigned risk strategy]
    J1 --> K1{CPO Decision}
    J2 --> K2{Risk Officer Decision}
    K1 -- Reject / Revise --> L[Return to Draft\nwith feedback]
    K2 -- Reject / Revise --> L
    L --> B
    K1 -- Approve --> WAIT{Both T1\napproved?}
    K2 -- Approve --> WAIT
    WAIT -- No --> PENDING[Waiting for\nother T1 approver]
    WAIT -- Yes --> M[Tier 2: CRO Review\nfinal authorization]
    M --> N{CRO Decision}
    N -- Reject / Revise --> L
    N -- Approve --> O[Campaign ACTIVE\nCO can now create applications under this campaign]
```

---

## NFRs

| NFR | Requirement |
|-----|-------------|
| Zero-code campaign creation | New campaigns require no code deployment — configuration only |
| Single umbrella | All 5 configuration dimensions live under one campaign entity — no split-configuration paths |
| Campaign versioning | Changes to an active campaign must be versioned — in-flight applications use the campaign version at submission time |

---

## Open Questions

- Can a campaign be edited while applications are in-flight? Or must it be versioned + cloned?
- ~~Is there a campaign approval workflow before publishing, or can product managers publish directly?~~ **Resolved**: Two-tier approval required (Tier 1: CPO, Tier 2: CRO). Product managers cannot publish directly. See Campaign Publication Authorization.
- What is the campaign archive / sunset process for old campaigns?
