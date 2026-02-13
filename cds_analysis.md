# CDS View Analysis for Federated ML Viability

This document evaluates SAP S/4HANA CDS views beyond the already-selected pillars ([`pragmatic.md`](pragmatic.md)) for their suitability in AITYA's federated ML architecture. Each candidate is assessed against the three criteria that justify the existing Ledger and Dunning pillars:

1. **Governed by standards or internal consistency checks** -- the data follows rules that make it self-validating (e.g., double-entry accounting, statutory requirements, deterministic formulas).
2. **Minimal customization surface** -- the structure and semantics of the CDS view are nearly identical across all S/4HANA implementations, making tenant-agnostic feature engineering feasible.
3. **Universal business language** -- the data maps to KPIs that every CFO, controller, or treasurer already speaks, requiring no buyer education.

A CDS view that fails any one of these criteria is not viable for federated ML in the AITYA architecture, because:

- Failing criterion 1 means the data has no built-in quality check, and the federated model may learn from noise or subjective inputs.
- Failing criterion 2 means onboarding each tenant requires bespoke mapping or configuration work -- the "multi-month data archaeology project" the product thesis explicitly avoids.
- Failing criterion 3 means the product requires a sales cycle spent on education rather than demonstrating value in vocabulary the buyer already uses.

---

## Already Selected (Reference Baseline)

| CDS View                    | Layer                   | Role                                                                                                                                                                         |
| --------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `I_JournalEntryItem`        | Ledger (transactional)  | Granular debit/credit transactions from ACDOCA                                                                                                                               |
| `I_GLAcctBalance`           | Ledger (transactional)  | Aggregated period balances for G/L accounts                                                                                                                                  |
| `I_OperationalAcctgDocItem` | Ledger (transactional)  | Clearing state and payment behavior at invoice level                                                                                                                         |
| `I_DunningHistory`          | Dunning (transactional) | Collection escalation trail (dunning levels 1-4)                                                                                                                             |
| `I_CompanyCode`             | Master data             | Posting entity context: country (ISO 3166), currency (ISO 4217), fiscal year variant                                                                                         |
| `I_Customer`                | Master data             | Customer segmentation: geography, industry sector, payment terms. Only ISO-governed and SAP-delivered fields are trusted; free-text and company-specific fields are excluded |

These six views plus `ACDOCA` constitute the seven-view scope defined in [`pragmatic.md`](pragmatic.md). Additionally, two SAP customizing tables are read once at deployment for automated cross-tenant normalization:

| Table            | Layer         | Role                                                                                                                            |
| ---------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `T052` / `T052U` | Configuration | Payment terms: resolves every `ZTERM` code to canonical day counts and discount percentages                                     |
| `T047` / `T047A` | Configuration | Dunning procedures: resolves every dunning procedure to its number of levels, interval in days, minimum amounts, and grace days |

These tables contain **deterministic arithmetic** — the actual parameters behind company-configured codes — enabling fully automated normalization without human mapping. See the Configuration Layer section in [`pragmatic.md`](pragmatic.md) for details.

---

## Tier 1: Strong Candidates

These views meet all three criteria with confidence. They represent viable expansion options if AITYA moves beyond Cash & Collections.

### `I_FixedAsset` + `I_FixedAssetForLedger` + `I_AssetValuationForLedger` -- Asset Accounting

**Domain:** Fixed asset lifecycle -- acquisition, depreciation, retirement.

**What they are:** A suite of released CDS views that together expose the full asset lifecycle:

- `I_FixedAsset`: Asset master data (asset class, capitalization date, useful life, depreciation key).
- `I_FixedAssetForLedger`: Ledger-dependent asset values (book value per ledger/depreciation area).
- `I_AssetValuationForLedger`: Current valuation data including accumulated depreciation, net book value, acquisition cost per depreciation area.
- `I_FixedAssetAssgmt`: Time-dependent assignment data (cost center, profit center assignments over time).
- `I_FixedAssetCountryData`: Country-specific regulatory data.

These views were progressively released starting with S/4HANA 2020. In S/4HANA, asset accounting posts directly into `ACDOCA`, so every depreciation posting is reconcilable against the general ledger.

**Criterion 1 -- Standards-governed: STRONG.**
Depreciation is entirely rules-based. Given an asset class, a depreciation key (method: straight-line, declining balance, etc.), a useful life, and an acquisition value, the depreciation schedule is deterministic and auditable. The consistency check is: `Acquisition Cost - Accumulated Depreciation = Net Book Value`. Depreciation areas are mapped to accounting principles (IFRS, US GAAP, local GAAP), providing built-in multi-standard governance.

**Criterion 2 -- Minimal customization: STRONG.**
Asset classes and depreciation keys come from a finite SAP-delivered catalog. The structure of depreciation (areas, methods, useful lives) is universal. A straight-line depreciation over 5 years in Company A produces the same mathematical curve as in Company B. The customization is in parameters to a standard formula, not free-form configuration.

**Criterion 3 -- Universal language: STRONG.**
CapEx, depreciation expense, net book value, ROA, asset turnover -- standard balance sheet and income statement items that every CFO reads.

**KPIs it could power:**

- Depreciation expense trending (actual vs. planned)
- Capital intensity ratio (CapEx / Revenue)
- Asset age distribution (average remaining useful life)
- Asset turnover ratio
- Return on assets (ROA)
- Impairment risk scoring (assets approaching end of useful life with high book value)
- CapEx efficiency (revenue generated per unit of net book value)

**Risks and limitations:**

- **Depreciation area heterogeneity.** Different companies may have different numbers of depreciation areas (one for local GAAP, one for IFRS, one for tax). A federated model must normalize to a single "primary" area or handle multi-area inputs. `I_FixedAssetForLedger` is keyed by ledger, which helps, but the choice of which ledger is "primary" varies.
- **Asset class mapping.** Asset class hierarchies are company-specific. A "Vehicle" asset class at Company A might be code 3100 and at Company B might be 2200. Feature engineering would need to map to a canonical asset taxonomy. SAP provides asset super-classes but adoption varies.
- **Low transaction frequency.** Unlike journal entries (daily), asset transactions are infrequent (acquisitions, retirements, transfers). This creates sparse training data per tenant, making federated aggregation more important but also more prone to overfitting on small datasets.
- **Master data sensitivity.** Asset registers can reveal capex strategy, plant locations, and investment plans. CISOs may consider this more sensitive than transaction-level data for competitive intelligence reasons.
- **Not a burning pain point.** Asset analytics is a "nice to have" for most mid-market CFOs, not a daily priority. The pain is in receivables and cash, not in depreciation schedules. This pillar would broaden the product but could dilute the sales message.

**Verdict:** Viable as a Phase 2 expansion after Cash & Collections is established. Opens a genuinely new analytical domain (capital allocation efficiency) without overlapping the existing scope.

---

### `I_GLAccountLineItemCube` -- G/L Line Items with Balance Carry Forward

**Domain:** General ledger reporting with year-to-date balances via line items.

**What it is:** An analytical cube view that provides journal entry line items including the balance carry forward entries. Aggregating all records for a given G/L account gives the cumulative balance as of any date. This is the view underlying SAP's standard trial balance and balance sheet reports.

**Criterion 1 -- Standards-governed: STRONG.**
Identical governance to `I_JournalEntryItem` (double-entry, ACDOCA-backed). The balance carry forward is a hard accounting close process -- mechanistic, not subjective.

**Criterion 2 -- Minimal customization: STRONG.**
Same narrow surface as the existing Ledger pillar. The carry forward logic is standard SAP.

**Criterion 3 -- Universal language: STRONG.**
Balance sheets, income statements, trial balances.

**KPIs it could power:**

- Point-in-time balance sheet reconstruction
- Intra-period cash balance trajectories (daily/weekly cash position changes)
- Working capital evolution over arbitrary time windows
- Segment-level P&L trending

**Risks and limitations:**

- **Not extraction-enabled by default.** SAP note 3036772 confirms that `I_GLAcctBalanceCube` has `ANALYTICS.DATAEXTRACTION.ENABLED = false`. Data extraction must use alternative methods (ODP, custom wrapper views). This is a deployment friction point.
- **Significant overlap with existing Ledger pillar.** The incremental value over `I_JournalEntryItem` + `I_GLAcctBalance` is the carry-forward convenience. The feature engineering team could reconstruct this from the existing views.

**Verdict:** Not a new pillar. Consider as a technical enrichment of the existing Ledger pillar if carry-forward data simplifies the feature engineering pipeline, but it does not expand the analytical surface area.

---

## Tier 2: Partially Viable (Meet 2 of 3 Criteria)

These views have real strengths but fail one criterion in a way that limits federated ML viability.

### `I_BankAccountAnalysisCube` + `I_HouseBankAccountAnalysisCube` -- Cash Management

**Domain:** Bank account balance and transaction analytics.

**What they are:** Released in S/4HANA 2020, these cube views provide bank account balance and transaction analytics. Related views include `I_BankGroupBankFeeCube` (bank fees analysis) and `I_BankGroupLiquidityStatusCube` (liquidity position by bank group).

**Criterion 1 -- Standards-governed: STRONG.**
Bank account balances reconcile against external bank statements (MT940/CAMT.053, ISO 20022). This is arguably stronger than self-validation -- a third party (the bank) confirms the balance.

**Criterion 2 -- Minimal customization: MODERATE.**
Bank account management was modernized in S/4HANA so that bank accounts are master data, not configuration. The structure is relatively standardized. However, the mapping of bank accounts to house banks, account IDs, and the liquidity hierarchy is company-specific.

**Criterion 3 -- Universal language: MODERATE.**
"Cash position" and "bank balance" are universally understood, but the granularity varies hugely (3 accounts vs. 300). The KPIs -- cash concentration efficiency, idle cash ratio, bank fee optimization -- are meaningful but treasury-specific rather than CFO-universal.

**Why it falls short for federated ML:**

- **Uneven module adoption.** Not all S/4HANA clients use advanced Cash Management (formerly TRM). Many use basic FI bank accounting only. This creates a data availability gap across the federated network -- you cannot build a federated model when half the tenants lack the underlying module.
- **Extreme sensitivity.** Bank account balances are among the most sensitive financial data points. CISOs will resist federated learning over cash positions far more aggressively than over receivables aging.
- **Company-specific topology.** The number of accounts, currencies, banking partners, and house bank hierarchy varies dramatically. Feature engineering must abstract away a topology that differs fundamentally per tenant.

**Verdict:** Not viable for the current federated architecture. The module adoption gap fragments the network, and the sensitivity profile makes CISO buy-in unrealistic for early-stage AITYA.

---

### `I_WithholdingTaxItem` + `I_OperationalAccountingTaxItem` -- Tax Reporting

**Domain:** Tax line items, withholding tax amounts, tax balances.

**What they are:** `I_WithholdingTaxItem` provides withholding tax amounts in local currency. `I_OperationalAccountingTaxItem` exposes fields from the tax tables (the `BSET` equivalent in S/4). Supporting views include `I_TaxBaseBalancesGroup` and `I_TaxBalancesGrp` for aggregated tax balance reporting.

**Criterion 1 -- Standards-governed: VERY STRONG.**
Tax is governed by statute, not by company preference. Tax codes, rates, and withholding obligations are defined by law. This is the most externally governed data domain in ERP -- more so than double-entry accounting, which is a professional standard rather than a legal mandate.

**Criterion 2 -- Minimal customization: MODERATE.**
Tax code configuration is country-specific and relatively standardized within a country (all Greek companies handle Greek VAT codes similarly). However, the number and mapping of tax codes varies per company. A company with complex international operations might have hundreds of tax codes; a domestic-only company might have 10.

**Criterion 3 -- Universal language: WEAK.**
"Effective tax rate" and "VAT liability" are understood at the executive level, but the detail level of tax data requires domain expertise. Tax analytics are more relevant to tax directors than to CFOs or COOs. The KPI surface is narrower than receivables or asset data.

**Why it falls short for federated ML:**

- **Jurisdiction-specific semantics destroy cross-tenant comparability.** Greek VAT rules are fundamentally different from German Umsatzsteuer. A federated model trained on Greek tax data provides zero value to a German tenant. This severely limits the federated network effect -- you would need per-country cohorts, and even then the value is questionable.
- **Tax calculations are deterministic.** Rate times base equals tax. There is limited predictive value in ML over data that is already the output of a formula. The "intelligence" in tax is in tax planning, which requires master data and future projections, not historical transaction pattern recognition.
- **Regulatory sensitivity.** Tax data is subject to audit privilege and legal confidentiality in many jurisdictions. Sharing even encrypted gradients derived from tax data may face legal challenges beyond standard GDPR concerns.

**Verdict:** Not viable. The jurisdiction-specific semantics eliminate the core benefit of federation (cross-tenant learning), and the deterministic nature of tax calculations limits what ML can add.

---

### `I_CostCenter` + `I_ProfitCenter` + Hierarchy Views -- Management Accounting / Controlling

**Domain:** Cost allocation, internal management reporting, organizational structure.

**What they are:** `I_CostCenter` and `I_ProfitCenter` are master data views providing organizational structure. The actual cost/revenue data flows through `I_JournalEntryItemCube` (which includes `ControllingArea`, `CostCenter`, `ProfitCenter` as dimension fields, since ACDOCA unified FI and CO). Supporting views include `I_CostCenterHierarchy`, `I_CostCenterHierarchyNode`, `I_ProfitCenterHierarchy`, and `I_ProfitCenterHierarchyNode`.

**Criterion 1 -- Standards-governed: MODERATE.**
Cost allocation and profit center accounting follow management accounting principles, but there is no statutory requirement for double-entry in CO (though S/4HANA's unified journal means CO postings are double-entry in ACDOCA). Internal orders and cost allocations involve subjective methodologies -- allocation keys, assessment cycles, distribution rules.

**Criterion 2 -- Minimal customization: WEAK.**
This is where controlling fails the test entirely. Cost center hierarchies, profit center structures, allocation methods, and internal order categories are heavily customized per company. Two companies in the same industry may have completely different cost center trees. There is no universal taxonomy, and building one would be the exact multi-month consulting project the AITYA architecture is designed to avoid.

**Criterion 3 -- Universal language: MODERATE.**
"Cost per unit," "overhead rate," and "profit center P&L" are widely understood concepts. But the specific cost center structure is meaningless outside the company that created it. You can say "overhead is 23%" universally, but the path to that number is entirely company-specific.

**Why it falls short for federated ML:**

- **Hierarchy incompatibility is a show-stopper.** Company A's cost center "4711-Marketing" has no semantic relationship to Company B's "CC-MKT-001." Feature engineering would need to map every tenant's cost center tree to a universal taxonomy. No such taxonomy exists. Creating one per-tenant is the exact customization burden AITYA is architected to avoid.
- **Subjectivity in allocation.** Cost allocations are management decisions, not governed facts. Different allocation methodologies produce different cost center "actual" values from the same underlying transactions. This injects systematic bias that varies per tenant -- the federated model would learn each company's allocation philosophy, not objective cost patterns.
- **Already partially available via the Ledger pillar.** Since ACDOCA includes `ProfitCenter` as a dimension on `I_JournalEntryItem`, the federated model can already slice ledger data by profit center without needing a separate pillar. The master data views add organizational context but not transactional intelligence.

**Verdict:** Not viable. The customization surface is too wide and the subjectivity in allocation methods injects tenant-specific bias that a federated model cannot normalize without per-client consulting.

---

## Tier 3: Not Viable

These views fail one or more criteria decisively. Included for completeness and to document why they were rejected.

### `I_BillingDocumentItem` -- Billing / Revenue

**Domain:** Sales billing document line items -- invoiced quantities, amounts, pricing conditions, revenue recognition.

**Why it fails:**

- **Criterion 2 (customization): FAIL.** Billing documents are shaped by pricing conditions (`KONV`), output types, and revenue recognition rules that vary wildly per company. The SAP pricing condition technique is one of the most heavily customized areas in any implementation. A pricing condition record at one company bears no structural resemblance to another's. Tenant-agnostic feature engineering is not feasible without building a full pricing normalization layer per client.
- Revenue recognition rules further diverge under different accounting standards (IFRS 15 vs. local GAAP), and company-specific contract structures make the billing data semantically incomparable across tenants.

**Verdict:** Rejected. Pricing customization makes cross-tenant feature engineering infeasible.

---

### Intercompany Reconciliation (ICMR Views)

**Domain:** Intercompany matching and reconciliation across legal entities within a group.

**Why it fails:**

- **Criterion 2 (customization): FAIL.** ICMR reconciliation cases are defined per company group with custom CDS views. There is no single standard `I_IntercompanyReconciliation` view -- the ICMR framework is deliberately flexible, reading from ACDOCA, BKPF/BSEG, ACDOCU, or a union of multiple tables depending on the implementation. Every deployment defines its own reconciliation CDS views.
- **Scope exclusion.** Intercompany data only exists for companies with multiple legal entities. Single-entity companies -- which may represent a significant portion of AITYA's Greek mid-market target -- are excluded entirely, fragmenting the federated network.

**Verdict:** Rejected. No standard view exists; every implementation is bespoke.

---

### `I_CaseAttributes` / `I_CaseReason` -- Dispute Management (FSCM-DM)

**Domain:** Customer payment dispute lifecycle -- dispute reasons, statuses, resolution.

**Why it falls short:**

- **Criterion 2 (customization): WEAK.** Dispute reason codes and status workflows are company-configured. The dispute lifecycle is flexible by design, not standardized.
- **Module adoption gap.** Dispute Management is an optional FSCM module that many S/4HANA clients do not activate. A federated model requiring FSCM-DM data would exclude non-adopters from the network.

**Potential as enrichment:** For tenants that do use FSCM-DM, dispute data is a powerful complement to the existing Dunning pillar. Disputes explain _why_ dunning is happening (pricing disagreement, quality complaint, missing documentation). This context could improve the Dunning pillar's risk scores by distinguishing legitimate disputes from genuine credit deterioration.

**Verdict:** Not viable as a standalone pillar. Consider as an optional enrichment of the Dunning pillar for tenants that have FSCM-DM active, but do not design the federated architecture to depend on it.

---

## Summary Matrix

| CDS View(s)                       | Domain                 | Standards | Customization | Language | Federated ML Viability                             | Tier |
| --------------------------------- | ---------------------- | --------- | ------------- | -------- | -------------------------------------------------- | ---- |
| `I_FixedAsset` suite              | Asset Accounting       | HIGH      | HIGH          | HIGH     | **Strong** (Phase 2)                               | 1    |
| `I_GLAccountLineItemCube`         | G/L with carry forward | HIGH      | HIGH          | HIGH     | **Overlap** (enrichment only)                      | 1\*  |
| `I_BankAccountAnalysisCube`       | Cash Management        | HIGH      | MODERATE      | MODERATE | **Blocked** (adoption gap, sensitivity)            | 2    |
| `I_WithholdingTaxItem`            | Tax                    | VERY HIGH | MODERATE      | WEAK     | **Blocked** (jurisdiction-specific, deterministic) | 2    |
| `I_CostCenter` + `I_ProfitCenter` | Controlling            | MODERATE  | WEAK          | MODERATE | **Blocked** (hierarchy incompatibility)            | 2    |
| `I_BillingDocumentItem`           | Billing/Revenue        | MODERATE  | FAIL          | MODERATE | **Rejected**                                       | 3    |
| ICMR views                        | Intercompany           | HIGH      | FAIL          | MODERATE | **Rejected**                                       | 3    |
| `I_CaseAttributes` (FSCM-DM)      | Disputes               | MODERATE  | WEAK          | LOW      | **Enrichment only**                                | 3    |

---

## Configuration Layer: Does It Change Any Verdicts?

The selected scope in [`pragmatic.md`](pragmatic.md) includes a configuration layer that reads SAP customizing tables (`T052` for payment terms, `T047` for dunning procedures) at deployment to automatically normalize company-configured fields. This eliminates manual mapping for the two largest sources of cross-tenant behavioral variation in the Cash & Collections scope.

The natural question is whether this pattern — reading config tables to auto-normalize — could rescue any of the excluded CDS views. The answer is no. Below is the analysis for each.

### Why the pattern works for payment terms and dunning procedures

`T052` and `T047` contain **deterministic arithmetic**: inputs to a formula that produces a number. Payment terms are day counts and discount percentages. Dunning procedures are intervals and level counts. Given the parameters, the behavior is fully reconstructable. The config table contains the math, and math normalizes automatically.

This pattern generalizes only to fields where the configuration is a **parameterized formula with deterministic semantics**.

### Why it does not rescue the excluded domains

| Excluded Domain                                    | Relevant Config Tables                                                       | What the config contains                                               | Why it doesn't help                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| -------------------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Controlling** (`I_CostCenter`, `I_ProfitCenter`) | CSKS/CSKT (cost center master), SETNODE/SETHEADER (hierarchy tables)         | Cost center codes, text descriptions, hierarchy structure              | The config gives labels, not cross-tenant semantic equivalence. Company A's "4711-Marketing" has no machine-readable relationship to Company B's "CC-MKT-001." Mapping would require NLP-based semantic matching of description text — a speculative research project, not a deployment automation. Furthermore, allocation methods (assessment cycles, distribution keys in COBRB/TKAS) are subjective management decisions. Reading the config tells you _what_ the allocation is, not whether it is _comparable_ to another tenant's allocation. The blocker is organizational design incompatibility, not code normalization. |
| **Tax** (`I_WithholdingTaxItem`)                   | T007A (tax codes), T007V (tax rates)                                         | Tax code definitions, rates per jurisdiction                           | Reading the config resolves codes to rates, but rates alone do not create cross-tenant comparability. Greek VAT at 24% and German Umsatzsteuer at 19% are not semantically equivalent — different exemption rules, reverse charge logic, intra-community supply treatment, and withholding obligations. The blocker is jurisdictional incomparability: the _structure_ of taxation differs by country, not just the parameters. Additionally, tax calculations are deterministic (rate × base = tax), so ML adds no predictive value regardless of normalization quality.                                                         |
| **Cash Management** (`I_BankAccountAnalysisCube`)  | T012 (house banks), T012K (house bank accounts)                              | Bank topology: house bank codes, account IDs, bank keys                | Reading the config reveals the bank account structure, but the blockers are not normalization problems. They are: (1) module adoption gaps — many S/4HANA tenants use only basic FI bank accounting, not advanced Cash Management, and config reads cannot activate an undeployed module; (2) CISO sensitivity — bank balances are among the most sensitive financial data, and no config-level abstraction changes the risk profile of sharing cash position gradients.                                                                                                                                                          |
| **Billing** (`I_BillingDocumentItem`)              | T685 (condition types), T683S (pricing procedures), KONP (condition records) | Condition type definitions, pricing procedure steps, condition records | The pricing procedure is not a parameterized formula — it is a combinatorial engine. A typical company has 50+ condition types arranged in a multi-step procedure with customer-specific, material-specific, and time-dependent condition records. "Normalizing" this means reconstructing each tenant's entire pricing logic, which is the opposite of automated. The config tables are extensive but their complexity is the problem, not the solution.                                                                                                                                                                         |
| **Intercompany** (ICMR views)                      | No standard config — ICMR reconciliation cases are defined per company group | N/A                                                                    | There is no standard CDS view to normalize. The framework is deliberately flexible, with each implementation defining its own reconciliation views. Config reads cannot create a standard where none exists.                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **Disputes** (`I_CaseAttributes`)                  | SCMG_T_REAS (dispute reasons), status profile config                         | Dispute reason codes and descriptions, status workflow definitions     | Marginally helpful for tenants that have FSCM-DM active — dispute reason descriptions could be mapped to canonical categories (price dispute, quality complaint, documentation issue). But the primary blocker is module adoption: Dispute Management is optional, and many tenants do not activate it. Config reads cannot bridge a module adoption gap. The verdict remains "optional enrichment for tenants with FSCM-DM," not a viable pillar.                                                                                                                                                                                |

### The generalization boundary

The configuration layer automates normalization when and only when:

1. **The config table contains deterministic arithmetic** — parameters to a formula, not labels or taxonomies.
2. **The formula's output is semantically universal** — "Net 30 days" means the same thing at every company; "Cost Center 4711" does not.
3. **The underlying module is universally deployed** — config reads require the module to be active.

Payment terms and dunning procedures satisfy all three conditions. None of the excluded domains do. This is not a limitation of the implementation — it reflects a structural property of SAP's configuration architecture: the more business-critical and standardized a process is, the more its configuration resembles math; the more discretionary and organizational a process is, the more its configuration resembles free-form taxonomy.

---

## Model-to-Data Pillar: Does Dropping the Cross-Tenant Constraint Change Any Verdicts?

The analysis above evaluates every CDS domain against a three-criteria test designed for **federated ML** — specifically, the requirement that features be portable across tenants without per-client customization (Criterion 2). But the AITYA architecture rests on two separable pillars: **federated gradient aggregation** and **model-to-data deployment**. These impose different constraints.

| Criterion | Required by model-to-data (single-tenant)? | Required by federated ML? |
|---|---|---|
| 1. Standards-governed / self-validating | Yes — data quality matters regardless | Yes |
| 2. Minimal customization surface | **No** — the pipeline reads ONE tenant's config, not many | Yes — features must be portable across tenants |
| 3. Universal business language | Partially — the buyer must still care about the KPIs | Yes — metrics must resonate across organizations |

Criterion 2 is the gatekeeper that kills the most domains in this document. Controlling fails on hierarchy incompatibility. Billing fails on pricing customization. Both are *cross-tenant* problems. In a single-tenant deployment — where the container trains only on the client's own data with no gradient sharing — Criterion 2 transforms. The question shifts from "Is this semantically identical across tenants?" to "Can a deployment pipeline programmatically read this tenant's configuration and auto-generate features from it?"

### Configuration trees as automated feature schema

The `T052`/`T047` pattern in [`pragmatic.md`](pragmatic.md) uses config tables for **normalization** — converting company-specific codes into canonical values for cross-tenant comparability. In single-tenant mode, the same class of config reads serves a different purpose: **automated feature organization**. The pipeline reads the tenant's configuration not to normalize across companies, but to understand how to structure features for that specific company without human mapping.

SAP customizing tables fall into four categories of machine readability:

| Config type | Example | What the pipeline extracts | Automatable? |
|---|---|---|---|
| **Parameterized formulas** | `T052` (payment terms), `T047` (dunning), `T093` (depreciation areas) | Arithmetic: day counts, percentages, intervals, depreciation methods | Fully — the config contains math |
| **Hierarchical structures** | `SETNODE`/`SETHEADER` (cost center/profit center hierarchies) | Parent-child trees: organizational aggregation levels | Fully — standard tree traversal |
| **Categorical definitions** | `T685` (pricing condition types: price, discount, surcharge, tax), `T003` (document types) | SAP-delivered classification labels for grouping transactional data | Yes, where categories are SAP-delivered; no, where company-specific |
| **Free-text taxonomies** | `CSKT` (cost center descriptions), `CustomerGroup` (`T151`) | Labels with no cross-tenant semantic equivalence | No — would require NLP-based semantic matching |

The first three categories enable what amounts to automated feature *organization*: the pipeline reads the config to learn how to bucket, aggregate, and slice the transactional data without a consultant explaining the tenant's setup. This is distinct from feature *generation*, which still comes from the transactional patterns themselves. Config tells the pipeline how to structure features, not what the features' values should be. The distinction matters because it bounds the engineering investment: the pipeline is reading a schema, not learning from a new data source.

No published precedent exists for using ERP configuration trees as an automated feature schema for ML models. SAP's own ML investments (AI Core, HANA PAL, Data Intelligence) treat configuration as setup, not as signal. This is either genuine whitespace or a signal that practitioners have evaluated the idea and found the ROI insufficient.

### Which verdicts change

**Controlling (`I_CostCenter` + `I_ProfitCenter`):** Changes from **Blocked** to **Conditionally Viable**.

The hierarchy incompatibility that blocks federated ML is irrelevant in single-tenant. Within one tenant, the cost center tree in `SETNODE`/`SETHEADER` is a fixed, machine-readable structure. A pipeline can traverse the hierarchy and auto-generate features at every aggregation level (department → division → area → company). Since `ACDOCA` already carries `CostCenter` and `ProfitCenter` on every journal entry, the existing Ledger pillar's data scope already contains the raw transactions — what's missing is the dimensional context to slice them intelligently.

Potential features: cost trends at each hierarchy level, overhead allocation ratio stability, cost center utilization anomalies, profit center margin decomposition.

*Residual risk:* Allocation methods (`COBRB`, assessment cycles) are management decisions, not economic facts. The model learns a company's internal cost attribution philosophy, which may shift when a controller restructures the hierarchy. Features derived from allocation rules are fragile over time within a single tenant.

**Billing (`I_BillingDocumentItem`):** Changes from **Rejected** to **Conditionally Viable**.

The pricing procedure (`T683S`) is a sequential algorithm where each step references a condition type from `T685`. Crucially, `T685` carries a SAP-delivered classification — condition class (price, discount, surcharge, tax) and calculation type (percentage, fixed amount, quantity-based). A pipeline could read `T683S` → `T685` and auto-categorize every pricing step, then generate features from billing document results organized by these categories: revenue decomposition by condition class, discount utilization rates, price erosion trends, margin compression signals.

The pipeline operates on billing document *results* (what was invoiced), using the procedure structure as a classification schema. It does not need to read condition *records* (`KONP` / `A***` tables), which contain commercially sensitive pricing data.

*Residual risk:* The combinatorial complexity of pricing procedures (50+ condition types, customer/material-specific records, time-dependent validity) means edge cases are abundant. Statistical-only condition types, inactive conditions, and complex rebate arrangements may produce misleading feature categories. The feature organization is automatable for the standard 80% of the pricing procedure; the remaining 20% may require per-tenant review.

**Cash Management (`I_BankAccountAnalysisCube`):** Changes from **Blocked** to **Conditionally Viable**.

Both blockers — module adoption gap and CISO sensitivity — are federated concerns. Module adoption is a per-tenant qualification check in single-tenant mode (either the tenant runs Cash Management or it does not). CISO resistance to local analytics within the tenant's own BTP container is categorically different from resistance to federated gradient sharing over cash positions.

The house bank topology (`T012`/`T012K`) is machine-readable and structurally simple. Feature engineering for cash position trends, inflow/outflow patterns, and idle cash detection is straightforward.

*Residual risk:* Only viable for tenants running advanced Cash Management, which narrows the eligible population. Not a universal expansion.

**Tax (`I_WithholdingTaxItem`):** Verdict unchanged — **Not Viable**. Dropping the cross-tenant constraint removes jurisdictional incomparability but does not solve the fundamental issue: tax calculations are deterministic (rate × base = tax). ML adds marginal predictive value over data that is already the output of a formula.

**Disputes (`I_CaseAttributes`):** Verdict unchanged — **Enrichment only**. Model-to-data does not change this; the blocker is module adoption, not cross-tenant semantics.

**Intercompany (ICMR):** Verdict unchanged — **Rejected**. The lack of standard CDS views is structural regardless of deployment mode.

### Updated verdict matrix (model-to-data single-tenant)

| CDS View(s) | Domain | Federated ML Verdict | Model-to-Data Verdict | What Changes |
|---|---|---|---|---|
| `I_CostCenter` + `I_ProfitCenter` | Controlling | Blocked | **Conditionally Viable** | Hierarchy trees readable via `SETNODE`; allocation subjectivity is residual risk |
| `I_BillingDocumentItem` | Billing/Revenue | Rejected | **Conditionally Viable** | Pricing procedure classifiable via `T683S` → `T685`; combinatorial edge cases are residual risk |
| `I_BankAccountAnalysisCube` | Cash Management | Blocked | **Conditionally Viable** | Module adoption is per-tenant check, not network-wide gap |
| `I_FixedAsset` suite | Asset Accounting | Strong (Phase 2) | **Strong (Phase 2)** | Already viable; single-tenant removes asset class mapping concern |
| `I_WithholdingTaxItem` | Tax | Blocked | **Not Viable** | Deterministic nature limits ML value-add regardless of scope |
| `I_CaseAttributes` (FSCM-DM) | Disputes | Enrichment only | **Enrichment only** | Module adoption remains the blocker |
| ICMR views | Intercompany | Rejected | **Rejected** | No standard views regardless of deployment mode |

### Access constraint: deployment model dependency

A critical caveat applies to all "Conditionally Viable" verdicts above. The config table reads that enable automated feature organization depend on the S/4HANA deployment model:

| Config table | On-Premise / Private Cloud | Public Cloud Edition |
|---|---|---|
| `SETNODE` / `SETHEADER` (hierarchies) | Accessible via RFC + Cloud Connector | Not directly accessible; `API_COSTCENTER_SRV` exposes master data but not full hierarchy config |
| `T683S` (pricing procedures) | Accessible via RFC + Cloud Connector | **Not accessible** — no released API exposes pricing procedure structure |
| `T685` (condition type definitions) | Accessible via RFC + Cloud Connector | **Not accessible** |
| `T012` / `T012K` (house banks) | Accessible via RFC + Cloud Connector | Partially accessible via bank master APIs |
| `T093` (depreciation areas) | Accessible via RFC + Cloud Connector | **Not accessible** as standalone config; asset depreciation data accessible per-asset |

The current architecture in [`pragmatic.md`](pragmatic.md) describes Side-by-Side BTP with Cloud Connector, implying on-premise or private cloud S/4HANA. The Greek mid-market target operates predominantly on-premise. Config-enabled scope expansion is therefore technically feasible for the current target market. But building product features that depend on deep config table reads creates a deployment-model dependency that does not currently exist in the architecture. If AITYA later targets S/4HANA Public Cloud Edition customers, these modules would be inaccessible without SAP releasing additional APIs.

### Strategic assessment

The model-to-data pillar is compatible with a broader CDS scope than the federated pillar requires. But compatibility is not the same as advisability.

**The case for capturing this optionality (Phase 2):**
- The deployment container is already inside the tenant. Reading additional config tables at onboarding is incremental engineering, not a new architecture.
- Controlling and billing analytics address different buyer personas (Controller, Head of Revenue Accounting) and could expand contract sizes.
- Single-tenant config-enabled analytics could serve as the entry point that leads to federated participation — the upsell path is: local analytics first, federation later.

**The case against pursuing it now:**
- The current product thesis was deliberately narrowed from five pillars to two. Expanding single-tenant scope based on config readability risks reconstructing the breadth problem under a different name.
- The strongest single-tenant features (cost hierarchy analysis, revenue decomposition) come from domains that *cannot* be federated. If these become the core value, the federated network effect — the claimed moat — becomes a minority of the product's value proposition.
- Each additional config tree adds onboarding validation complexity. `T052` and `T047` are two small tables with deterministic arithmetic. Pricing procedures and cost center hierarchies are orders of magnitude more complex, with more edge cases and more per-tenant debugging.

**Recommendation:** Document as a Phase 2 architecture option. The model-to-data pillar's broader compatibility is a strategic asset — it means the product can expand into Controlling, Billing, Cash Management, and Asset Accounting analytics without re-architecting the deployment — but pursuing it before Cash & Collections is validated would dilute the launch product and complicate the sales message. See the corresponding note in [`pragmatic.md`](pragmatic.md).

---

## Conclusion

The Ledger and Dunning pillars -- now including `I_OperationalAcctgDocItem` for clearing behavior -- sit close to the natural boundary of what federated ML can credibly address in SAP S/4HANA without per-client customization. The configuration layer (`T052`, `T047`) automates normalization for the two fields within this scope that are company-configured, but as demonstrated above, this pattern does not extend to the excluded domains. The domains that remain standardized enough for tenant-agnostic feature engineering are a much smaller set than the full ERP footprint suggests.

The only genuinely new analytical domain that passes all three criteria is **Asset Accounting** (`I_FixedAsset` suite). It opens capital allocation analytics without overlapping Cash & Collections, but it is not a burning pain point for the target buyer and should be considered a Phase 2 expansion after the core product is validated. Notably, asset accounting's depreciation config (table `T093` for depreciation areas, `ANKA` for asset classes) partially fits the config-layer pattern — depreciation methods are parameterized formulas — but asset class mapping remains a semantic problem (Company A's code 3100 "Vehicle" vs. Company B's code 2200 "Vehicle") that config reads alone cannot resolve.

Everything else fails on customization surface (Controlling, Billing, Intercompany), cross-tenant comparability (Tax), module adoption consistency (Cash Management, Dispute Management), or is already subsumed by the existing pillar scope (`I_GLAccountLineItemCube`). The configuration layer does not change any of these verdicts because the blockers are structural — organizational design incompatibility, jurisdictional semantics, combinatorial customization depth, or absent modules — not normalization problems that a config read can solve.

However, the federated ML boundary is not the only boundary that matters. As analyzed in the Model-to-Data section above, the architecture's two pillars — federated gradient aggregation and model-to-data deployment — impose different constraints. When the cross-tenant portability requirement is dropped, Controlling, Billing, and Cash Management become conditionally viable for single-tenant analytics via automated config reads. This does not change the federated scope but it expands the product's addressable surface for local intelligence. The distinction between the two boundaries — narrow for federation, broader for model-to-data — is a Phase 2 strategic asset.

This narrow *federated* viability is both a risk and a validation: narrow scope invites "point solution" perception, but it also confirms that AITYA has already captured the strongest candidates for cross-tenant learning. The configuration layer strengthens the selected scope by automating what was previously manual onboarding work, and the model-to-data architecture preserves optionality for broader single-tenant analytics without requiring re-architecture.
