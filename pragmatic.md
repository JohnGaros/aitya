# AITYA -- Product Thesis

*The reason behind the data.*

---

## The Problem

The global migration to SAP S/4HANA has given enterprises a powerful financial backbone -- the Universal Journal (`ACDOCA`). But most companies lack the intelligence layer to turn that data into proactive strategy. Traditional ERP reporting misses the operational "friction" between departments -- payment delays, receivables drift, cash position deterioration -- until it damages the P&L. Meanwhile, existing AI solutions force a painful choice:

- **Centralized AI** requires extracting sensitive financial data into external lakes, creating security risks, high egress costs, and loss of data sovereignty.
- **Shallow analytics** stay inside the ERP but ignore cross-functional dynamics, producing dashboards that describe the past without predicting the future.

AITYA resolves this trade-off by pushing intelligence to the data rather than pulling data to the intelligence.

---

## The Core Idea: Federated Zero-Export Intelligence

### What It Is

A federated machine learning architecture where models train locally inside each client's SAP BTP tenant. Raw financial data never leaves the client's Virtual Private Cloud (VPC). Only encrypted mathematical gradients -- the "learned lessons" from local training -- are shared with a central aggregator that improves a Global Base Model for all participants.

### How It Works

1. **Local Node:** A Docker container running in SAP AI Core inside the client's BTP tenant pulls data via standard CDS Consumption Views.
2. **Local Training:** The model trains on the client's own data, learning patterns specific to their business.
3. **Gradient Export:** Only encrypted weight updates (not data) leave the tenant, protected by Secure Aggregation and Differential Privacy.
4. **Global Aggregation:** A central node (owned by AITYA) combines gradients from all participating tenants into a smarter Base Model.
5. **Model Push:** The improved Base Model is pushed back to all clients, giving each participant the benefit of collective learning.

### Benefits

**For the client:**

- **Zero data extraction.** Sensitive financial records (`ACDOCA`) stay inside the client's protected environment. No data lakes, no egress costs, no CISO negotiations about where the data lives. (This advantage is strongest for tenants without SAP Business Data Cloud; for tenants on RISE with BDC, zero-copy integrations are narrowing the cost and speed delta. See "SAP zero-copy platform risk" below and [`zero_copy_viability.md`](zero_copy_viability.md).)
- **Regulatory compliance by design.** Because raw PII never travels, the architecture is naturally aligned with GDPR and the 2026 European AI Act. Compliance is structural, not procedural.
- **Collective advantage without exposure.** Small firms benefit from a Global Base Model trained on patterns from many enterprises. Large firms get early-warning signals -- e.g., industry-wide payment slowdowns -- without revealing their own internal KPIs.
- **Benchmarking that is otherwise impossible.** Questions like "Are our customers paying us slower than our competitors' customers?" and "Is our receivables-to-revenue ratio deteriorating faster than our industry peers?" cannot be answered with internal data alone. The federated network makes them answerable.

**For AITYA:**

- **Network-effect moat.** Each new participant improves the Global Base Model. A competitor entering the market would need years of gradient accumulation to match the predictive accuracy of an established network. (But centralized AR intelligence platforms already achieve cross-company learning without federation, and the federated architecture itself is composable on zero-copy infrastructure. See "Established AR intelligence competitors" below.)
- **Industry-scoped intelligence.** Federated models can be segmented by industry and geography, producing signals that are specific enough to be actionable (e.g., payment trends across Greek FMCG/Retail) rather than generically global.
- **Scalable deployment.** Because the model runs on standardized CDS views, onboarding a new tenant does not require a bespoke integration project for each client.

### Risks

**Gradient inversion attacks.** Research has shown that model gradients can, under certain conditions, be reverse-engineered to reconstruct fragments of the original training data. Differential Privacy (adding calibrated noise to gradients) mitigates this but introduces a trade-off: too much noise degrades model accuracy. The optimal privacy-utility balance for financial data is not yet established in production settings and remains an active area of academic research.

**Cross-industry signal dilution.** An anonymized gradient from a Greek retailer may provide limited value to a German chemical manufacturer. If the federated network is too heterogeneous, the Global Base Model may learn "average" patterns that are too generic to drive specific strategic action. Clients paying a premium for federated insights will quickly question the value if the signals are indistinguishable from publicly available macroeconomic reports. This risk is partially mitigated by industry-scoped federation, but scoping reduces the pool size and weakens the network effect.

**CISO resistance to participation.** Even with privacy guarantees, many Chief Information Security Officers may reject participation in any federated network on principle. The perceived risk of information leakage -- however mathematically small -- or being benchmarked against competitors may cause enterprise security teams to block Tier 3 adoption, capping revenue potential at single-tenant analytics.

**Aggregator as single point of trust.** The central aggregation node is owned and operated by AITYA. Clients must trust that AITYA's implementation of Secure Aggregation and Differential Privacy is correct, auditable, and not compromised. A breach or misconfiguration at the aggregator level could undermine the entire trust model. Independent third-party audits of the aggregation protocol would be necessary but are costly and not yet standard practice in the federated ML industry.

**Regulatory uncertainty.** The 2026 European AI Act introduces requirements for "high-risk AI systems," which may include financial prediction models. It is not yet clear whether federated gradient sharing qualifies as data processing under certain interpretations, which could introduce consent and documentation obligations that erode the "compliance by design" advantage.

**SAP zero-copy platform risk.** SAP's platform strategy is systematically eliminating the data access friction that AITYA's value proposition depends on. The November 2025 SAP-Snowflake zero-copy partnership (Q1 2026 GA), the Databricks BDC Connect GA, and planned Google BigQuery and Microsoft Fabric integrations collectively make zero-copy access to S/4HANA data available without ETL, replication, or schema mapping. For tenants on RISE with BDC, a competitor can activate zero-copy access and build analytics on Snowpark, Databricks ML Runtime, or Vertex AI in days -- the same onboarding speed AITYA claims as a differentiator -- at a marginal cost approaching zero because BDC is shared across all analytics use cases. AITYA's speed and cost advantage is real today for on-premise holdouts, but that population is shrinking: 30% of organizations are now on S/4HANA Cloud Private Edition (up from 19% the prior year), and mid-market companies migrate at twice the rate of large enterprises. The addressable market for AITYA's current thesis has a visible expiration date of 3-5 years. See [`zero_copy_viability.md`](zero_copy_viability.md) for the full analysis.

**Established AR intelligence competitors.** AITYA is entering a market with funded incumbents that already deliver the business outcome -- predictive collections, payment risk scoring, cross-company benchmarking -- without federation. HighRadius processes $4.6T+ in AR transactions across 430+ customers with a SAP-certified extractor. Sidetrade captures $1.7T across 285M invoices. Both achieve cross-company intelligence by pooling centralized data, which is statistically superior to noise-injected federated gradients from a small network. Additionally, SAP announced a Joule AR Agent at SAP Connect 2025 that analyzes overdue receivables and automates follow-up, with a Cash Management Agent planned for Q1 2026. These SAP-native agents are embedded, zero-integration, and included in the S/4HANA license. They are shallow today but will not remain so. The AR Automation market is projected to reach $6.57B by 2031 -- growth that benefits incumbents first. The "Black Box" Docker container protects AITYA's implementation details but does not prevent a well-resourced competitor from building an equivalent product once the concept is validated, and Google Cloud already publishes a cross-silo federated learning reference architecture (BDC Connect + BigQuery + Vertex AI + TFF + TEEs) that could deliver AITYA's claimed outcome on competitor infrastructure.

---

## The Two Semantic Pillars: Ledger and Dunning

### Why These Two -- and Not the Full Five-Pillar EVI

The original vision proposed a five-pillar Enterprise Vitality Index (Growth, Supply, Capital, Payment, ESG) spanning dozens of CDS views. The skeptic's critique was valid: mapping an enterprise's entire operational landscape to a universal model requires massive per-client consulting, relies on views with variable customization depth, and produces abstract metrics that CFOs do not naturally speak.

The two pillars selected here -- **Ledger** and **Dunning** -- survive scrutiny because they share three properties that the other three pillars lack:

1. **Governed by double-entry accounting.** Ledger and dunning data follow the fundamental accounting equation. This means the data is self-validating: Starting Balance + Debits - Credits = Ending Balance. Any inconsistency is a data quality signal, not a subjective interpretation. The other pillars (Growth via sales quotations, Supply via purchase order confirmations, ESG via emissions) have no equivalent internal consistency check.

2. **Minimal customization surface.** Payment terms, dunning levels, G/L account structures for receivables and cash, journal entry formats -- these are among the most standardized elements in any S/4HANA implementation. Unlike sales pricing conditions (`I_SalesPricingConditionItem`) or inventory configurations (`I_MaterialStock`), which are heavily customized per client, the receivables-and-cash data domain has a narrow semantic surface. This makes tenant-agnostic feature engineering feasible without a "multi-month data archaeology project."

3. **Universal business language.** Every CFO speaks DSO, cash conversion cycle, bad debt ratio, and receivables aging. These are not proprietary metrics requiring a sales cycle spent on education. The product translates directly into the financial vocabulary that controllers and treasurers already use.

### Pillar 1: Ledger (Financial Position, Movement, and Clearing)

Built on three highly stable CDS views, all rooted in the Universal Journal (`ACDOCA`):

**`I_JournalEntryItem`** -- Granular Transactions

- The modern successor to the classic `BSEG` table.
- Provides individual debit and credit line items for any period.
- Stability: **Very High.** One of the most fundamental CDS views in S/4HANA Finance.
- Key fields: `DebitAmountInCoCodeCrcy`, `CreditAmountInCoCodeCrcy`, `GLAccount`, `CompanyCode`, `Ledger`, `PostingDate`.

**`I_GLAcctBalance`** -- Aggregated Balances

- Delivers pre-computed starting balance, ending balance, and total period movements for G/L accounts.
- Stability: **Very High.** The standard view for trial balances, balance sheets, and all balance-based financial statements.
- Key fields: `StartingBalanceInCoCodeCrcy`, `EndingBalanceInCoCodeCrcy`, `DebitAmountInCoCodeCrcy`, `CreditAmountInCoCodeCrcy`.

**`I_OperationalAcctgDocItem`** -- Clearing and Payment Behavior

- The operational counterpart to `I_JournalEntryItem`. Exposes the clearing state of every subledger line item: whether an invoice has been paid, when, and against which clearing document.
- Stability: **High, with a qualification.** At the individual line-item level, clearing is binary -- an item is either open (`ClearingAccountingDocument` empty) or cleared (`ClearingAccountingDocument` populated). This makes it more rigid than most ERP status fields. However, **partial payments introduce ambiguity** that the binary model does not surface. SAP offers two methods for partial payments, each producing a different data signature in the view (see "Partial clearing ambiguity" in Risks below).
- Key fields: `ClearingAccountingDocument`, `ClearingDate`, `NetDueDate`, `CashDiscount1Days`, `CashDiscount1Amount`, `AccountType` (D=Customer, K=Vendor), `CompanyCode`. Note: `ClearingAccountingDocument` and `ClearingDocFiscalYear` are deprecated in recent S/4HANA releases; the replacement access pattern is not yet well-documented by SAP.

**What the Ledger Pillar enables:**

- Period-over-period receivables and cash position trending.
- Reconciliation-backed validation: every model input can be verified against the accounting equation, providing an auditable proof chain from prediction to posted financials.
- Segment-level balance-sheet exposure quantification (how much capital is trapped by specific customer groups, business units, or geographies).
- Payment punctuality analysis: the delta between `NetDueDate` and `ClearingDate` is the most direct measure of actual vs. promised payment behavior at the invoice level -- for fully-cleared invoices. For partial payments, the delta is either absent (partial payment method) or reflects only the first partial settlement (residual method); see "Partial clearing ambiguity" in Risks.
- Cash discount capture rate: what proportion of early-payment discounts are being realized, by customer segment.
- Clearing velocity: how many days elapse from posting to clearing, revealing the true cash conversion tempo independent of dunning escalation.

**Why the Ledger Pillar supports tenant-agnostic feature engineering:**

The key fields are identical across all S/4HANA implementations. `CompanyCode`, `Ledger`, `GLAccount`, and `PostingDate` are non-negotiable structural elements of the Universal Journal. While the *values* in `GLAccount` vary per client (every company has its own Chart of Accounts), the *structure* and *semantics* of the views are fixed. Feature engineering targets the patterns in debit/credit movements and balance trajectories, not the specific G/L codes -- making the features portable across tenants. The only per-client configuration required is identifying which G/L accounts represent receivables and cash, a mapping that is orders of magnitude simpler than the full Chart of Accounts mapping the original five-pillar EVI demanded.

The clearing fields in `I_OperationalAcctgDocItem` are equally portable. `ClearingAccountingDocument`, `ClearingDate`, and `NetDueDate` have identical semantics across every S/4HANA tenant. For fully-cleared invoices, the clearing process is structurally invariant: an open item transitions to cleared when a payment or credit memo is matched against it, regardless of the company's industry or configuration. Features derived from full-clearing behavior -- days-past-due at clearing, discount utilization rate, clearing velocity by customer segment -- require no per-tenant customization. Partial payments are a complication: the same economic event (partial settlement) produces different data signatures depending on whether the client uses partial payment (no clearing, invoice stays open) or residual payment (invoice clears, new residual item created). Feature engineering must detect and normalize both patterns (see "Partial clearing ambiguity" in Risks). Payment terms codes (`ZTERM` values) vary per tenant, but the configuration layer reads `T052` at deployment to automatically resolve every code to canonical day counts and discount percentages, eliminating the need for manual mapping.

### Pillar 2: Dunning (Payment Behavior and Credit Risk)

Built on `I_DunningHistory`, also rooted in `ACDOCA`:

**`I_DunningHistory`** -- Collection Escalation Trail

- Tracks the dunning level progression for each customer and invoice.
- Stability: **Very High.** Dunning is a core S/4HANA Finance process with a standardized escalation structure (typically Levels 1-4).
- Key fields: Dunning level, dunning date, customer, amount, company code.

**What the Dunning Pillar enables:**

- Invoice-level payment risk scoring: predict which open invoices will default based on historical dunning patterns.
- Collection prioritization: direct AR resources to the highest-risk, highest-value accounts.
- Customer-segment risk profiling: identify which industries, geographies, or customer cohorts are deteriorating.
- Early-warning detection: rising dunning velocity across a customer base signals a broader economic downturn before it hits the P&L.

**Why the Dunning Pillar supports tenant-agnostic feature engineering:**

Dunning levels, escalation timelines, and the relationship between dunning events and payment outcomes follow a universal process structure in SAP. Whether a company runs four dunning levels or three, the *pattern* of escalation and resolution is semantically consistent. Features like "days between dunning levels," "ratio of Level 2+ invoices to total open items," and "dunning velocity by customer segment" are meaningful across any S/4HANA tenant without per-client customization.

### How the Two Pillars Complement Each Other

Dunning, clearing, and ledger data answer different parts of the same question. The three CDS data sources form a complete chain: the ledger shows what is owed and at what magnitude, clearing shows when and how it was actually paid, and dunning shows what happened when it was not paid.

| Question | Dunning Provides | Clearing Provides | Ledger (Balances + Journals) Provides |
|---|---|---|---|
| Who is paying late? | Customer-level risk scores and dunning progression | Invoice-level proof: `NetDueDate` vs. `ClearingDate` delta | -- |
| How much is at stake? | -- | -- | Exact balance-sheet exposure tied to at-risk receivables |
| Are collections working? | Dunning resolution rates and escalation velocity | Whether cleared items are settling faster or slower over time | Whether cash positions are actually improving period over period |
| Is the trend getting worse? | Rising dunning levels across customer segments | Lengthening clearing velocity across the portfolio | Receivables-to-revenue ratio trends and cash conversion cycle |
| Are we capturing discounts? | -- | Cash discount utilization rate by customer segment | Financial impact of missed discounts on the P&L |
| Can we trust the numbers? | -- | Clearing is deterministic at the line-item level (open → cleared), though partial payments require multi-item reconstruction | Reconciliation equation validates data integrity before model training |

**Combined analytical capabilities:**

- **Receivables-at-risk:** Cross-reference high-dunning invoices with G/L balances to quantify the financial weight of distressed customers on the balance sheet -- not just invoice counts, but capital impact.
- **Cash conversion tracking:** Correlate dunning escalation velocity with clearing dates and debit/credit movements to detect whether collection activity is converting to actual cash inflows or merely delaying write-offs. Clearing data provides the ground truth: did the money actually arrive?
- **Payment behavior prediction:** The `NetDueDate` vs. `ClearingDate` delta from `I_OperationalAcctgDocItem` is the most direct feature for predicting future payment behavior on fully-cleared invoices. For partially-paid invoices, the feature engineering must reconstruct settlement timelines from multiple open items or residual chains (see "Partial clearing ambiguity" in Risks). Combined with dunning progression, it distinguishes customers who pay late-but-reliably from those who are genuinely deteriorating.
- **Reconciliation-backed audit trail:** Every predictive risk score traces back to verified journal entries, giving the CFO a proof chain from ML prediction to posted financials.
- **Segment-level cash conversion cycle:** Combine payment timing from clearing and dunning with debit/credit movements from journal entries to compute cash conversion metrics by customer segment, geography, or business unit.
- **Discount leakage quantification:** Identify which customer segments or business units are systematically missing early-payment discounts and quantify the P&L impact via ledger movements.

### Master Data Layer: Dimensional Context for the Pillars

The transactional views above carry foreign keys (`CompanyCode`, `Customer`) that are meaningless without master data to resolve them. When this document refers to "customer-segment risk profiling," "industry-scoped federation," or "geography-based benchmarking," it implicitly depends on master data joins. Two master data views are consumed to provide this dimensional context.

**`I_CompanyCode`** -- Posting Entity Context

- Resolves the `CompanyCode` key present on every transactional view to the legal entity's country, currency, and fiscal year variant.
- Stability: **Very High.** Company code is the most fundamental organizational unit in SAP FI.
- Key fields: `CompanyCode`, `Country` (ISO 3166), `Currency` (ISO 4217), `FiscalYearVariant`, `ChartOfAccounts`.

All fields are externally governed or SAP-delivered. `Country` follows ISO 3166, `Currency` follows ISO 4217, and `FiscalYearVariant` is selected from a standard SAP catalog. The structure is identical across every S/4HANA tenant. This view is pure reference data: low volume, no extraction concerns, no sensitivity issues.

**`I_Customer`** -- Customer Segmentation Context

- Resolves the `Customer` key on `I_OperationalAcctgDocItem` and `I_DunningHistory` to the customer's geography and, where maintained, industry classification.
- Stability: **Very High.** Customer master is a core SAP Business Partner object since S/4HANA.
- Key fields used: `Customer`, `Country` (ISO 3166), `Region` (ISO 3166-2 via SAP region codes), `IndustrySector` (SAP-delivered codes), `PaymentTerms`.

**Not all fields on `I_Customer` are trusted for federated feature engineering.** The view is consumed selectively:

| Field | Trusted? | Justification |
|---|---|---|
| `Country` | **Yes** | ISO 3166, externally governed, universally populated |
| `Region` | **Yes** | ISO 3166-2, standardized per country |
| `IndustrySector` | **Conditional** | SAP-delivered code list, but maintenance quality varies across tenants. Usable as a feature where populated; cannot be treated as a required input |
| `PaymentTerms` | **Yes, automated** | Company-configured codes, but `T052` contains the actual day counts and discount percentages. The configuration layer resolves these automatically at deployment |
| `CustomerGroup` | **No** | Company-specific taxonomy with no universal meaning |
| `CityName`, `PostalCode`, `StreetName` | **No** | Free-text, inconsistent formatting, no structural validation |
| `CustomerName` | **No** | PII -- excluded from the feature pipeline entirely |

**Why master data is not a "third pillar":**

These views do not generate analytical features on their own. They are dimension tables that enrich the transactional pillars with segmentation context. `I_CompanyCode` tells the model *where* a posting occurred (country, currency). `I_Customer` tells the model *who* the counterparty is (geography, industry, payment terms). The analytical intelligence comes from the transactional patterns in ledger, clearing, and dunning data; master data provides the axes along which those patterns are sliced.

**Why master data matters for the federated architecture:**

Industry-scoped federation -- the mechanism that produces actionable benchmarks rather than generic signals -- depends on `I_Customer`.`IndustrySector` being populated. If a tenant's customer master lacks industry classification, that tenant can still benefit from single-tenant analytics but cannot participate in industry-specific federated cohorts. This is an onboarding qualification check, not a per-client customization project: either the field is maintained or it is not.

### Configuration Layer: Automated Cross-Tenant Normalization

Two company-configured fields -- payment terms and dunning procedures -- appear throughout the transactional and master data layers. Their values are tenant-specific codes, but the SAP customizing tables that define those codes contain machine-readable semantics: the actual arithmetic behind each code. By reading these tables at deployment, the onboarding pipeline can automatically normalize cross-tenant variation without human mapping.

**`T052` / `T052U`** -- Payment Terms Configuration

- The customizing table behind every `ZTERM` (payment terms) code in the system.
- Contains the actual day counts and discount percentages: `ZTAG1`/`ZTAG2`/`ZTAG3` (day thresholds for discount periods and net due), `ZBD1P`/`ZBD2P`/`ZBD3P` (discount percentages per period), and fixed-day/fixed-month indicators for end-of-month terms.
- A single read of `T052` at onboarding resolves every payment terms code to a canonical representation (e.g., "Net 30 days, 2% discount if paid within 10 days") with no human interpretation required. The full `T052` structure supports four complexity layers beyond this simple case -- day limits, fixed-day/end-of-month rules, and installment plans via `T052S` -- all deterministic arithmetic but requiring progressively richer canonical representations. See [`payment_terms.md`](payment_terms.md) for the complete field inventory, layer analysis, and implementation gap assessment.

**`T047` / `T047A`** -- Dunning Procedure Configuration

- The customizing table behind every dunning procedure assigned to customer accounts.
- Contains the number of dunning levels, the dunning interval in days per level, minimum amounts and minimum percentages per level, and grace days before the first dunning notice.
- A single read of `T047` at onboarding tells the federated model exactly how each tenant's dunning policy is calibrated: Company A dunns every 14 days across 4 levels; Company B dunns every 30 days across 3 levels. The model can factor out policy differences and isolate the underlying credit risk signal.

**What the configuration layer automates:**

The two largest sources of cross-tenant behavioral variation in the current scope -- payment terms and dunning escalation rules -- are not free-form customizations. They are parameterized configurations with deterministic semantics stored in standard SAP tables. Reading these tables converts what would otherwise be a per-client normalization exercise into an automated pipeline step: deploy the container, read the config tables, build the normalization mappings, begin training. No consulting, no workshops, no mapping spreadsheets.

**What the configuration layer does not solve:**

- `CustomerGroup` (`T151`): the config table contains codes and text descriptions, but the semantic intent behind the grouping is arbitrary per company. One tenant uses it for geographic segmentation, another for customer size, another for sales channel. Reading the config gives labels, not meaning. This field remains excluded from the feature pipeline.
- `IndustrySector` (`T016`): SAP delivers ~40 standard codes. Tenants using SAP defaults are automatically normalized. Tenants with custom codes require a one-time mapping review -- not fully automatable, but a bounded task (typically fewer than 10 custom codes).
- Free-text fields (`CityName`, `PostalCode`): no configuration table can normalize unstructured text. These remain excluded.

**Data access footprint:**

Two additional small customizing tables (a few hundred rows each), read once at deployment. These contain payment policy parameters, not transactional or customer data. The sensitivity profile is negligible -- CISOs should have no objection to read-only access to payment terms and dunning procedure definitions.

### Risks Specific to the Two-Pillar Scope

**Scope may be too narrow for enterprise buyers.** A product that only addresses receivables and collections -- however well -- may be perceived as a "point solution" rather than a strategic platform. Enterprise buyers often prefer integrated suites. The narrow scope could limit contract sizes and make AITYA vulnerable to displacement by broader analytics platforms that add "good enough" collections features.

**Receivables G/L mapping is simpler but not trivial.** While identifying receivables and cash accounts is far easier than mapping an entire Chart of Accounts, it is not zero-effort. Companies with complex intercompany structures, multiple ledgers, or non-standard receivables classifications (e.g., factored receivables, consignment arrangements) will still require per-client configuration work.

**Dunning process variation (mitigated).** While dunning levels are structurally standardized, the business rules governing escalation vary significantly. Some companies escalate aggressively (daily dunning runs); others are lenient (quarterly). The configuration layer reads `T047` at deployment to automatically normalize for these policy differences, allowing the federated model to factor out dunning policy and isolate actual credit risk. The residual risk is that normalization assumes the config accurately reflects practice -- if a company's configured dunning interval is 14 days but the actual dunning run is executed monthly due to operational delays, the model's policy adjustment will be miscalibrated.

**Payment terms normalization (mitigated).** The clearing view (`I_OperationalAcctgDocItem`) provides structurally identical fields across tenants, but the underlying payment terms (`ZTERM` values) are company-configured. The configuration layer reads `T052` at deployment to automatically resolve every payment terms code to canonical day counts and discount percentages. No human mapping is required. The residual risk is payment terms that reference complex conditions (e.g., installment plans, milestone-based payments) where the `T052` parameters do not fully capture the payment schedule. These edge cases are uncommon in standard receivables but may appear in project-based industries.

**Partial clearing ambiguity.** The clearing model is binary at the line-item level (open or cleared), but partial payments break the assumption that `ClearingDate` reflects full settlement. SAP offers two methods for partial payments, each producing a different data signature:

- *Partial payment (no clearing):* The payment posts as a separate open item. The original invoice stays open with `ClearingAccountingDocument` empty. No clearing date exists until final settlement. The fact that a partial payment was made is only discoverable by summing all open items for that business partner -- there is no "partially paid" status field.
- *Residual payment (clearing + new residual item):* The original invoice is cleared and looks identical to a fully-paid invoice (`ClearingAccountingDocument` populated, `ClearingDate` set). A new open item is created for the unpaid remainder, linked to the original invoice via the `REBZG` field in BSEG. The clearing date reflects the partial settlement, not full payment.

The same economic event (partial settlement) produces incompatible data signatures depending on the method used, and the method can vary by client, by clerk, or even by transaction. Features derived from `ClearingDate` -- clearing velocity, days-past-due at clearing, payment punctuality -- are clean for fully-cleared invoices but misleading for partial clearings under either method. Feature engineering must detect both patterns: for partial payments, reconstruct settlement progress from open-item sums; for residual payments, trace the residual chain back to the original invoice. The residual chain depends on `REBZG`, which is a BSEG field and may not be exposed in `I_OperationalAcctgDocItem` -- if it is not, reconstructing residual payment histories from the CDS view alone is impossible without also reading from BSEG or ACDOCA directly. Additionally, `ClearingAccountingDocument` and `ClearingDocFiscalYear` are deprecated in recent S/4HANA releases; the replacement access pattern is not yet clearly documented by SAP, introducing a forward-compatibility risk for any feature engineering that depends on these fields.

**Clearing data volume.** `I_OperationalAcctgDocItem` can contain millions of line items per company code per year. It includes all subledger item types (customer, vendor, G/L), requiring `AccountType` filtering to isolate customer receivables. Feature engineering must aggregate intelligently to avoid gradient explosion during federated training.

**Industry classification dependency.** Industry-scoped federation relies on `I_Customer`.`IndustrySector` being populated. This field uses SAP-delivered codes but is not mandatory -- tenants with poorly maintained customer master data cannot participate in industry-specific cohorts. If a significant proportion of pilot tenants lack this classification, the federated benchmarking tier loses its segmentation capability and reverts to geography-only cohorts, which are less actionable for CFOs asking "how do we compare to our sector peers?"

**Missing context without the other three pillars.** Payment behavior does not exist in isolation. A customer paying late because of a supply chain disruption (Supply pillar) is fundamentally different from one paying late due to financial distress. Without supply chain and sales data, the model may misattribute root causes, leading to incorrect risk scores. This is a deliberate trade-off: the two-pillar scope accepts reduced explanatory power in exchange for feasibility and speed to market.

### Phase 2 Scope Expansion via Model-to-Data

The two-pillar boundary above applies to **federated ML**, where features must be portable across tenants without per-client customization. But the architecture rests on two separable pillars: federated gradient aggregation and model-to-data deployment. These impose different constraints. Federated ML requires cross-tenant semantic standardization (Criterion 2 in [`cds_analysis.md`](cds_analysis.md)); model-to-data deployment requires only that a pipeline can programmatically read a single tenant's configuration.

When the cross-tenant portability requirement is dropped, SAP customizing tables can serve as an **automated feature schema** — the pipeline reads the tenant's configuration not to normalize across companies, but to understand how to structure features for that specific company without human mapping. This extends the `T052`/`T047` config-read pattern from normalization to feature organization.

Three domains that fail the federated ML test become conditionally viable for single-tenant analytics:

- **Controlling** (`I_CostCenter` + `I_ProfitCenter`): cost center/profit center hierarchies are machine-readable tree structures in `SETNODE`/`SETHEADER`. Since `ACDOCA` already carries `CostCenter` and `ProfitCenter` on every journal entry, the existing data scope contains the transactions; config reads provide the organizational schema to slice them.
- **Billing** (`I_BillingDocumentItem`): the pricing procedure (`T683S` → `T685`) carries SAP-delivered condition type classifications (price, discount, surcharge, tax) that enable automated revenue decomposition without reconstructing the full pricing logic.
- **Cash Management** (`I_BankAccountAnalysisCube`): the federated blockers (module adoption gap, CISO sensitivity to gradient sharing) do not apply to local analytics within the tenant's own BTP container.

**Federated boundary constraint.** If Phase 2 proceeds, expanded features from these domains must not enter the federated gradient pipeline. The reason is architectural: human-assisted or config-assisted category mappings (e.g., classifying a tenant's custom pricing condition types into canonical categories via `T685`) introduce cross-tenant semantic variability that the gradient aggregation server cannot detect or correct. Unlike `T052` parameters — where "Net 30 days" is arithmetic and identical everywhere — category mappings are judgment calls whose consistency degrades across tenants, mapping personnel, and time. Feeding semantically misaligned features into gradient aggregation corrupts the global model without any observable error signal, because the aggregation layer sees only compressed gradients, not the feature definitions behind them. Phase 2 must therefore maintain strict architectural separation between the **federated feature set** (narrow, two-pillar, semantic alignment guaranteed by construction) and the **single-tenant feature set** (broad, config-assisted, per-tenant semantics only). The model must run separate feature pipelines for federated and local training. This is a hard constraint — not a preference to be relaxed as mapping quality improves — because the federated guarantee is architectural while mapping consistency is procedural, and procedural guarantees do not scale.

**This is a Phase 2 option, not a current commitment.** Expanding single-tenant scope before Cash & Collections is validated would dilute the launch product and complicate the sales message. The strongest single-tenant features come from domains that cannot be federated, which risks weakening the network-effect moat if they become the core value. Additionally, config-enabled scope expansion depends on on-premise or private cloud S/4HANA (the current Greek target market); S/4HANA Public Cloud Edition blocks most customizing table access. See [`cds_analysis.md`](cds_analysis.md) for the full analysis, including the config readability taxonomy, updated verdict matrix, and deployment model constraints.

---

## Deployment Architecture

### "Code-to-Data" via SAP AI Core (Side-by-Side BTP)

The model is deployed as a Docker container inside the client's BTP tenant, connecting to S/4HANA via the Cloud Connector. This is the "Side-by-Side" architecture:

- **Read-only access** through a single Cloud Connector configuration to: four transactional CDS views (`I_DunningHistory`, `I_JournalEntryItem`, `I_GLAcctBalance`, `I_OperationalAcctgDocItem`), two master data views (`I_CompanyCode`, `I_Customer`), the underlying Universal Journal (`ACDOCA`), and two customizing tables (`T052` for payment terms, `T047` for dunning procedures) read once at deployment for automated cross-tenant normalization.
- **Local training** via Argo Workflows inside SAP AI Core. The model trains on the client's own data without any data leaving the tenant.
- **Gradient export only.** After local training, only encrypted weight updates are sent to the central aggregator.
- **Model update.** The improved Global Base Model is pushed back to the client's AI Core instance.

### Why Side-by-Side and Not Embedded or Datasphere

| Option | Trade-off |
|---|---|
| **Embedded (HANA ML/PAL)** | Zero latency but limited to shallow models; cannot support federated orchestration. |
| **Side-by-Side (SAP AI Core)** | Full flexibility for deep learning and federated logic; data stays within the client's cloud but streams to the BTP container. **Selected.** |
| **Datasphere Federation** | Can join data across S/4HANA, Ariba, SuccessFactors -- but requires Datasphere licensing, introduces architectural complexity, and is overkill for a two-pillar scope. |

### Built-In Data Quality Validation

The ledger pillar provides a self-checking mechanism that most ML pipelines lack: the reconciliation equation.

$$
\text{Starting Balance} + \sum(\text{Debits}) - \sum(\text{Credits}) = \text{Ending Balance}
$$

Before any model training begins, the pipeline validates that `I_JournalEntryItem` transactions reconcile to `I_GLAcctBalance` movements. If they do not, the system flags a data quality issue and halts training rather than learning from corrupt data. This directly addresses the "garbage in, garbage out" risk that plagues ML systems operating on enterprise data.

### The Greek Market Tension: BTP Assumption vs. On-Premise Reality

The deployment architecture above assumes SAP AI Core inside the client's BTP tenant. This creates a misalignment with the actual Greek market landscape documented in [`greek_market.md`](greek_market.md).

**The market is mostly on-premise.** Of the 10 confirmed S/4HANA companies in Greece, only 3 are on RISE with SAP Cloud (HELLENiQ ENERGY, Sarantis Group, Piraeus Bank). The remaining 7 (Motor Oil, OPAP, Coca-Cola HBC, Titan Cement, Frigoglass, ADMIE, Plaisio) run S/4HANA on-premise, on Azure IaaS, or in managed private cloud. If BTP is a hard requirement, the addressable base for the federated architecture is 3 companies across 3 different industries -- too few for federated benchmarking and too heterogeneous for industry-scoped cohorts.

**Federated learning does not technically require BTP.** The architecture needs three things: local compute to train a model, data access to CDS views, and outbound connectivity for gradient export. All three are achievable on-premise. On-premise S/4HANA exposes the same CDS Consumption Views via OData or RFC. The customizing tables (`T052`, `T047`) that enable automated cross-tenant normalization are *more* accessible on-premise than on S/4HANA Public Cloud, where SAP's clean core strategy restricts customizing table access (see [`architecture_comparison.md`](architecture_comparison.md) for the full access matrix). The deployment model where AITYA's technical design works best is on-premise, not cloud.

**The competitive landscape inverts for on-premise.** SAP's zero-copy partnerships (Snowflake, Databricks, BigQuery via BDC Connect) -- identified as the primary platform threat in [`zero_copy_viability.md`](zero_copy_viability.md) -- require RISE with SAP Business Data Cloud. On-premise customers cannot activate zero-copy access. For these companies, the traditional data-to-model friction (ETL pipelines, schema mapping, sovereignty negotiations, EUR 50K-200K implementation cost) still fully applies. Model-to-data's speed, cost, and sovereignty advantages are strongest precisely where zero-copy is unavailable.

This creates an inverted relationship between the two threats the thesis identifies:

| Customer Segment | Zero-Copy Threat | BTP Deployment Constraint |
|---|---|---|
| **RISE with BDC** (3 companies) | High -- competitors activate zero-copy in days | None -- BTP/AI Core available |
| **On-premise / IaaS** (7 companies) | Absent -- zero-copy requires RISE/BDC | High -- no BTP/AI Core without deployment model change |

The segment where AITYA's competitive position is strongest (on-premise, no zero-copy alternative, full config table access) is the segment where the current deployment architecture does not work. The segment where deployment is straightforward (RISE/BTP) is the segment where zero-copy erodes the value proposition.

**Resolving the tension requires an architectural decision:**

**Option A: On-premise container deployment.** Extend the architecture to deploy the training container directly in the client's on-premise environment -- a VM or Kubernetes cluster adjacent to S/4HANA, reading CDS views over the local network. This expands the addressable market from 3 to 10 companies and eliminates the zero-copy competitive threat for 7 of them. But it introduces significant operational costs:

- *Infrastructure heterogeneity.* Each client's on-premise environment differs in OS, container runtime, network topology, and available compute. The "deploy in days" promise depends on a uniform target; on-premise diversity pushes toward per-client engineering.
- *Gradient export through corporate firewalls.* BTP has managed connectivity to external services. On-premise environments require firewall exceptions for outbound gradient traffic to AITYA's aggregator -- a harder CISO conversation than "what runs inside our BTP container," because it becomes "why is a startup's container in our data center sending traffic to an external server."
- *No managed orchestration.* SAP AI Core provides Argo Workflows, GPU scheduling, and monitoring. On-premise deployment requires AITYA to provide or manage equivalent infrastructure, or accept less reliable training runs.

**Option B: BTP-only deployment, accept the narrow base.** Maintain the current architecture and accept that the Greek pilot addresses 3 RISE customers. Use these as proof-of-concept deployments while waiting for the remaining 7 to migrate toward RISE -- which the 2027 ECC end-of-life deadline and SAP's go-to-market pressure will encourage. The risk: the migration timeline is uncertain, and by the time these companies reach RISE, zero-copy may have eliminated the value proposition that single-tenant analytics was meant to prove.

**Option C: Hybrid approach.** Deploy via BTP/AI Core where available (Tier 1), and offer a lightweight on-premise container for Tier 2 companies willing to provide a Kubernetes endpoint. Accept higher onboarding cost for on-premise clients (weeks, not days) while maintaining automated onboarding for RISE clients. This is the most realistic path but adds product complexity and splits engineering effort across two deployment targets at a stage when neither has been validated.

The current deployment architecture section assumes Option B without acknowledging the trade-off. The Greek market data forces the question: is the BTP dependency a design principle or a tactical convenience? If the former, the addressable market is 3 companies and the federated thesis cannot be validated in Greece. If the latter, the architecture should describe both deployment paths and their respective costs.

---

## The Product: Cash & Collections Intelligence

### What It Does

A predictive analytics engine that combines dunning and ledger data to answer two questions:

1. **Which customers will fail to pay, and what should we do about it?** (Collections scoring)
2. **What is the balance-sheet impact of payment behavior, and is it improving?** (Working-capital analytics)

### Who Pays for It

The Head of Accounts Receivable, the Group CFO, or the Head of Financial Controlling in mid-to-large S/4HANA enterprises. The pitch translates directly into metrics they already track:

- "Reduce your Days Sales Outstanding (DSO) by X%."
- "Cut bad debt write-offs by Y%."
- "See exactly how much balance-sheet capital is trapped by your slowest-paying segments -- and track whether it is improving."

### What to Cut from the Original Vision

- **The five-pillar EVI.** Replaced by two pillars with robust data foundations. The other three pillars (Growth, Supply, ESG) can be added incrementally if the two-pillar foundation proves viable.
- **The universal Agnostic Semantic Layer.** Replaced by a narrow mapping scope: receivables/cash G/L accounts and dunning configuration. This is orders of magnitude simpler than mapping an entire Chart of Accounts.
- **Abstract proprietary metrics.** Replaced by standard financial KPIs (DSO, cash conversion cycle, bad debt ratio, receivables aging) that require no buyer education.
- **The "Enterprise Vitality" brand.** Replaced by "Cash & Collections Intelligence" -- a name that describes exactly what the product does.

### What to Keep

- **Federated Zero-Export architecture.** This is the structural differentiator. The benefits (privacy, compliance, collective learning) and risks (gradient inversion, CISO resistance, signal dilution) are detailed above.
- **Industry-scoped federation.** Rather than a single global model, federated cohorts are segmented by industry and geography. This produces actionable benchmarks ("How does our cash conversion cycle compare to our sector peers?") rather than generic signals.
- **Reconciliation-backed predictions.** Every risk score traces back to verified ledger entries. This is a credibility advantage that pure-ML competitors cannot replicate without access to the same governed data layer.

---

## Federated Benchmarking: The Premium Tier

Beyond single-tenant analytics, the federated network enables benchmarking questions that no individual company can answer alone:

- "Are our customers paying us slower than they are paying our direct competitors?"
- "Is our receivables-to-revenue ratio deteriorating faster or slower than our industry peers?"
- "How does our cash conversion cycle compare to the sector median?"

These questions have tangible strategic value for the CFO. They turn a predictive collections tool into a competitive intelligence platform. However, the premium depends entirely on achieving sufficient participant density within an industry-geography cohort. A federated model with three Greek retailers may produce noisy, unreliable benchmarks. The minimum viable cohort size for statistically meaningful benchmarking is an open question that must be answered empirically during the pilot phase.

### Cross-Tenant Dunning Policy Evaluation

The benchmarking questions above are descriptive: they compare KPIs across peers. The federated architecture also enables a prescriptive capability that is harder to replicate: **evaluating which dunning policy configurations produce better cash conversion outcomes for similar customer profiles.**

**How it works.** Different tenants configure different dunning procedures via `T047`: Tenant A dunns every 14 days across 4 levels with a €100 minimum; Tenant B dunns every 30 days across 3 levels with a €500 minimum. The configuration layer already reads these parameters at deployment (see Configuration Layer above). Combined with clearing outcomes from `I_OperationalAcctgDocItem` and customer state data (aging, amount, segment, historical payment behavior), the federated network observes the same underlying question — "what happens when you apply policy X to customer-state Y?" — answered under natural variation across tenants.

This is offline policy evaluation: estimating the expected cash recovery outcome of a dunning configuration for a given customer profile, using observed outcomes across tenants whose configurations differ. In contextual bandit terms, the customer state is the context, the dunning configuration parameters (interval, levels, thresholds) are the action, and the clearing outcome (time-to-payment, amount recovered) is the reward. Cross-tenant gradient aggregation provides the variation that no single tenant can generate internally, because each tenant runs one fixed policy.

**What it could tell a CFO:** "For customers in your aging profile — mid-sized, 60-90 days past due, manufacturing sector — tenants with aggressive 14-day intervals recover cash 11 days faster on average than those with 30-day intervals, but experience 8% higher dispute rates." This is not a descriptive benchmark; it is an actionable recommendation grounded in observed outcomes across the network.

**Risks and limitations:**

- **Confounding.** Tenants with different dunning policies also differ in customer base, credit policies, economic environment, and unobserved collection actions (phone calls, negotiations, legal escalation). The observed outcome difference between a 14-day and 30-day interval reflects the entire business context, not just the dunning timing. Without randomized experimentation, causal attribution is unreliable. Statistical methods (propensity matching on observable customer states, doubly robust estimators) can reduce but not eliminate confounding.
- **Unobservable actions.** The dunning notice is the only collection action recorded in the current CDS scope. Real collections workflows include phone calls, payment plan negotiations, discount extensions, and legal threats — none of which appear in `I_DunningHistory`. The policy evaluation attributes outcomes to dunning configuration when the actual causal driver may be an unrecorded human intervention. This systematically biases estimates toward tenants with more active (but invisible) manual collection processes.
- **Narrow action space.** The evaluable "policy" is limited to `T047` parameters: interval, level count, minimum amounts, grace days. These are the least interesting collection decisions. The high-value questions — "Should we call this customer?", "Should we offer a payment plan?" — require FSCM Collections Management or CRM data that is outside the current scope.
- **Cohort size requirements.** Meaningful policy evaluation requires sufficient tenants with sufficiently different configurations serving sufficiently similar customer profiles. The intersection of these three conditions may be small, especially in the initial Greek market with ~20 target tenants. Policy evaluation is more credible at network scale than at pilot scale.
- **Presentation risk.** Framing dunning configuration recommendations as "AI-optimized collection strategy" overstates what the data supports. The honest framing is: "observed outcome differences across tenants with different dunning configurations, conditional on customer profile, with known confounders." Overstating the causal claim invites scrutiny from sophisticated buyers and erodes trust.

**Relationship to single-tenant scoring.** Policy evaluation is complementary to, not a replacement for, the collection scoring described above. Single-tenant scoring predicts *which* customers will pay late. Policy evaluation suggests *what dunning configuration* might improve outcomes for a given customer profile. The first is feasible with the current scope; the second is feasible only with the federated network and carries the caveats above.
