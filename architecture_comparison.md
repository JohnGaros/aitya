# Model-to-Data vs. Data-to-Model: Architecture Comparison for S/4HANA Cloud

This document compares AITYA's model-to-data (federated, zero-export) approach against the traditional data-to-model (centralized extraction) approach for ML analytics on SAP S/4HANA. The comparison is grounded in the current (2025-2026) SAP ecosystem, including SAP's own analytics stack evolution, third-party AR/Collections competitors, and the regulatory landscape.

For the product thesis and scope rationale, see [`pragmatic.md`](pragmatic.md). For the CDS view viability analysis that determines scope boundaries, see [`cds_analysis.md`](cds_analysis.md).

---

## Architecture and Data Flow

### Data-to-Model (Traditional)

Data leaves S/4HANA and travels to an external platform where models live. The extraction chain typically involves:

- **CDS extraction views** exposed via OData or ODP (Operational Data Provisioning) as the semantic interface to `ACDOCA` and other tables.
- **An ETL/replication layer** -- SAP's own ABAP Push Replication Services (APRS) to SAP Business Data Cloud, or third-party connectors (Fivetran, SNP Glue, Azure Data Factory, Informatica) to external platforms like Databricks, Snowflake, or Azure Synapse.
- **A data lake or warehouse** where raw SAP tables are landed, curated through medallion layers (Bronze/Silver/Gold), and joined with non-SAP data.
- **An ML training environment** (Databricks ML Runtime, SageMaker, Vertex AI, or SAP AI Core in a centralized topology) that reads from the curated layer.
- **A feedback loop** pushing predictions back to S/4HANA via APIs or BTP middleware.

The architecture is well-understood, tooling-rich, and supports arbitrary model complexity. SAP's direction since 2024 has been toward "zero-copy" data sharing via the Delta Sharing protocol in SAP Business Data Cloud, where S/4HANA data products are replicated to a HANA Cloud Data Lake object store (S3, ADLS Gen2) and made accessible to Databricks or Snowflake without full physical extraction. This is still data-to-model -- the data moves to an external compute layer -- but SAP is reducing the friction of that movement. BDC Connect for Databricks reached GA on all hyperscalers in late 2025; the SAP Snowflake joint product is planned for Q1 2026; BDC Connect for Microsoft Fabric for Q3 2026.

### Model-to-Data (AITYA)

The model travels to the data. A Docker container running in SAP AI Core inside the client's BTP tenant reads CDS views directly via Cloud Connector. No data leaves the tenant's VPC. Training happens locally; only encrypted gradients exit for federated aggregation.

The critical architectural difference: there is no ETL pipeline, no data lake, no replication layer, and no medallion architecture. The model reads live operational data through the same CDS views that SAP's own embedded analytics use, processes it in-memory within AI Core's Argo Workflows, and discards the raw data after training. The persistent artifact is the model weights, not the data copy.

### Comparison

| Dimension | Data-to-Model | Model-to-Data |
|---|---|---|
| Data copies created | 1-3 (raw, curated, serving) | 0 |
| Infrastructure layers | 4-6 (source, ETL, lake, transform, ML, serving) | 2 (source, BTP container) |
| Latency to fresh data | Minutes to hours (batch), seconds (CDC streaming) | Real-time (direct CDS read) |
| Non-SAP data joinability | Native -- the whole point is combining sources | Absent -- the container sees only S/4HANA |
| Architectural maturity | 15+ years of enterprise precedent | Novel; no production-scale precedent in SAP ecosystem |

---

## Data Sovereignty and Compliance

### Data-to-Model

Financial data from `ACDOCA` -- including customer identifiers, transaction amounts, payment behavior, and G/L balances -- physically moves to an external environment. Even with SAP Business Data Cloud's "zero-copy" Delta Sharing, the compute layer that reads the shared data is outside the S/4HANA tenant. This creates:

- **GDPR Article 28 obligations.** The external platform operator becomes a data processor. A Data Processing Agreement (DPA) is required. If the platform is a US hyperscaler, Schrems II transfer impact assessments apply for EU data subjects.
- **European AI Act exposure.** Financial prediction models trained on extracted data are the AI system. The entity operating the external platform bears documentation and risk-management obligations under the Act's high-risk AI provisions.
- **CISO negotiation overhead.** Every new external data destination requires security review: where is the data stored, who has access, what happens at contract termination, how is data deleted? For sensitive financial data, this review is often months of back-and-forth before a contract is signed.
- **SAP EU Access Only Option.** SAP offers an option to restrict data access during maintenance to EU-based personnel, priced as a percentage surcharge on the product contract value (reported at ~10%). This addresses maintenance access but not the fundamental data residency question for third-party platforms.

Post-July 2025, SAP unbundled Datasphere from the RISE with SAP base package. Clients who previously assumed Datasphere was included now face additional licensing costs for SAP's own data-to-model pathway -- and those choosing third-party platforms (Databricks, Snowflake) face the full sovereignty review regardless.

### Model-to-Data

Raw data never leaves the tenant's VPC. The CISO conversation changes from "where does our financial data go?" to "what runs inside our existing BTP container?" The former is a data sovereignty question; the latter is a software supply chain question.

- **GDPR simplification.** No data processing by third parties occurs. The model container is software deployed within the client's own infrastructure, analogous to any BTP side-by-side extension. No DPA required for data movement because there is none.
- **AI Act positioning.** The AI system runs within the client's control boundary. The client and AITYA share responsibility for the model, but the data governance chain is entirely internal.
- **Gradient export as residual risk.** Federated gradient sharing does transmit mathematical derivatives to AITYA's aggregator. Whether this constitutes "data processing" under GDPR is unresolved. Differential Privacy mitigates the reconstruction risk, but the legal classification of encrypted gradients is not settled.

### Assessment

Model-to-data has a genuine structural advantage on sovereignty. But the advantage is not absolute: the federated tier reintroduces a data-movement question (gradients), and the single-tenant tier (no federation) eliminates it entirely at the cost of losing the network effect. Data-to-model's sovereignty costs are real and growing -- European regulators are tightening -- but enterprises are already solving them daily for non-SAP data. The incremental burden of adding S/4HANA to an existing Databricks or Snowflake environment is lower than the burden of extracting it for the first time.

The EU Data Act (effective September 12, 2025) partially de-risks data-to-model. It mandates cloud providers support seamless switching, eliminate lock-in fees, and maintain interoperability registers. This reduces -- but does not eliminate -- the vendor lock-in argument that model-to-data claims as an advantage. Clients extracting S/4HANA data to Databricks now have stronger legal rights to port that data elsewhere. The sovereignty question (where is the data?) remains, but the switching cost question is weakened.

---

## Available Data Scope

This is where model-to-data pays its sharpest penalty.

### Data-to-Model

Once data reaches an external platform, the model can consume anything: S/4HANA financials, Ariba procurement, SuccessFactors HR, CRM pipeline data, external market data, credit bureau feeds, weather data, macroeconomic indicators. The analytical scope is bounded only by what the organization is willing to extract and join. This is why the data lake pattern won: it trades sovereignty for scope.

Specialized AR/Collections vendors exploit this. HighRadius extracts S/4HANA AR data into its cloud via its SAP-certified HEX (HighRadius Extractor) ABAP plug-in, then enriches it with external payment behavior signals, credit ratings, and cross-industry payment benchmarks from its own multi-tenant data pool. Sidetrade does the same with its "Aimie" AI, trained on what it reports as 285 million invoices and $1.7T in captured transaction data (2025). These vendors are doing federated learning in effect -- they pool anonymized patterns across clients -- but they do it by centralizing the data first.

### Model-to-Data

The model sees only what the CDS views inside the tenant expose. For AITYA, that is precisely seven views plus two config tables. The entire analytical surface is: journal entries, G/L balances, clearing states, dunning history, company codes, customer master (selected fields), payment terms, and dunning procedures.

This scope is deliberately narrow and well-defended (see [`cds_analysis.md`](cds_analysis.md) for why expansion fails the federated ML criteria). But the constraint is architectural, not analytical. A data-to-model competitor can answer: "This customer is paying late because their industry is experiencing a downturn (macro data), their procurement volume with us dropped 40% last quarter (Ariba data), and their credit rating was downgraded last month (external feed)." AITYA can answer: "This customer is paying late based on dunning escalation velocity and historical clearing patterns." Both are useful; only one explains *why*.

### Comparison

| Dimension | Data-to-Model | Model-to-Data |
|---|---|---|
| Financial data depth | Full (ACDOCA + all CDS views) | Narrow (4 transactional + 2 master data views) |
| Non-financial enrichment | Unlimited (external joins) | None |
| Explanatory power | Multivariate (can attribute root causes) | Univariate within the financial domain |
| Cross-system correlation | Native | Requires BTP integration (not in current scope) |

The scope constraint is the single largest disadvantage of model-to-data for analytical quality. It is also the reason model-to-data is deployable without a consulting project -- the two properties are inseparable.

---

## Why the CDS Exclusion Criteria Don't Apply to Data-to-Model

The three criteria in [`cds_analysis.md`](cds_analysis.md) that gate AITYA's scope are:

1. **Governed by standards or internal consistency checks** -- self-validating data.
2. **Minimal customization surface** -- structure and semantics nearly identical across all S/4HANA implementations.
3. **Universal business language** -- maps to KPIs every CFO already speaks.

These criteria conflate two separate requirements: **(a)** is this data usable for ML, and **(b)** is this data portable across tenants without per-client human mapping? AITYA's federated architecture demands both simultaneously. Data-to-model demands only (a).

### How data-to-model handles each excluded domain

**Controlling (`I_CostCenter` + `I_ProfitCenter`) -- blocked by hierarchy incompatibility in federated ML**

The federated blocker: Company A's "4711-Marketing" has no semantic relationship to Company B's "CC-MKT-001." A federated model cannot combine gradients from features that mean different things across tenants.

How data-to-model solves this: it does not need cross-tenant compatibility. A HighRadius or Databricks implementation reads *this client's* hierarchy from `SETNODE`/`SETHEADER` during implementation, builds features against *this client's* cost center tree, and trains a model specific to *this client's* organizational structure. The hierarchy is machine-readable -- the problem was never "can a pipeline read this?" but "can a pipeline read this without human guidance?" Data-to-model accepts human guidance at onboarding. An implementation consultant maps the client's organizational structure to the vendor's internal schema. For pooled multi-tenant models, controlling data is still excluded from cross-client comparison -- but it does not need to be. The vendor pools *receivables behavior features* (which are standardized) across clients and uses cost center features *per-client* for within-tenant analytics.

**Billing (`I_BillingDocumentItem`) -- blocked by pricing customization in federated ML**

The federated blocker: pricing procedures (`T683S` → `T685` → `KONP`) are a combinatorial engine, different at every client. Federated ML cannot normalize a 50-condition pricing procedure to a portable feature set.

How data-to-model solves this: the vendor's SAP adapter reads the client's pricing procedure at implementation and maps condition types to canonical categories (price, discount, surcharge, tax) using `T685`'s SAP-delivered classification -- exactly what [`cds_analysis.md`](cds_analysis.md) describes as "conditionally viable for single-tenant." The difference: data-to-model vendors have been doing this per-client mapping for a decade. They have built transformation libraries that handle the combinatorial complexity. The edge cases (statistical conditions, rebate arrangements, intercompany pricing) are documented in their implementation playbooks.

`cds_analysis.md` already identifies billing data as viable for single-tenant model-to-data in its Phase 2 section. Data-to-model competitors are doing Phase 2 today, because their architecture permits it.

**Cash Management (`I_BankAccountAnalysisCube`) -- blocked by module adoption gap and CISO sensitivity in federated ML**

The federated blockers: not all clients run advanced Cash Management, and CISOs resist sharing cash position gradients.

How data-to-model solves this: module adoption is a per-client qualification check in any architecture -- the vendor simply does not sell cash management features to clients that lack the module. The CISO objection shifts: extracting cash data to a vendor's cloud is arguably *more* sensitive than sharing encrypted gradients -- but established vendors have 15+ years of SOC 2 certifications, DPAs, and insurance policies to satisfy security reviews. The CISO objection is solved with compliance infrastructure, not architecture.

**Tax (`I_WithholdingTaxItem`) -- blocked by jurisdictional incomparability and deterministic nature**

Data-to-model does not meaningfully solve this either. Tax calculations are deterministic (rate x base = tax), limiting what ML can add regardless of architecture. Where data-to-model gains marginal value is in tax *provision forecasting* and effective tax rate anomaly detection, but these are secondary features. This is the one domain where the `cds_analysis.md` verdict holds equally for both approaches.

### The structural difference

The CDS exclusion criteria in [`cds_analysis.md`](cds_analysis.md) are **self-imposed constraints driven by the business model** (no consulting, fast onboarding, federated-ready), not inherent limitations of the data. Every domain that AITYA excludes for federated ML reasons is already being consumed by data-to-model competitors who accepted the per-client integration cost.

| | Model-to-Data (AITYA) | Data-to-Model |
|---|---|---|
| **Who normalizes** | The architecture (only selecting self-normalizing data) | The implementation team (per-client mapping at onboarding) |
| **Cost per client** | Near-zero (automated config reads) | Significant (weeks-months of implementation consulting) |
| **Domains accessible** | Narrow (4 transactional views + 2 master data) | Broad (entire S/4HANA footprint) |
| **Cross-tenant comparability** | Inherent (features are identical by construction) | Engineered (vendor's canonical schema, built over years) |
| **Onboarding speed** | Days | Weeks to months |
| **Marginal cost of scope expansion** | High (each new domain requires proving portability) | Low (each new domain is another adapter in the library) |

---

## Onboarding and Integration Complexity

### Data-to-Model

Getting S/4HANA data into an external platform is well-documented but non-trivial:

- **Extraction setup.** CDS extraction views must be activated, ODP queues configured, or APRS replication flows defined. For S/4HANA Public Cloud, SAP now provides pre-built "data products" that simplify activation, but custom extraction requirements still need custom CDS views or API orchestration.
- **Schema mapping and transformation.** SAP table structures (`ACDOCA`'s 300+ columns, `KONV`'s pricing conditions, Chart of Accounts mappings) must be transformed into analytics-ready schemas. Even for just receivables, the typical transformation project involves mapping G/L accounts to receivables/cash categories, resolving payment terms codes, normalizing currencies, handling fiscal year variants, and building time-series feature tables.
- **Ongoing pipeline maintenance.** S/4HANA upgrades can change CDS view structures. Custom CDS views need regression testing. Delta replication flows need monitoring. The ETL pipeline is a permanent operational cost.
- **Consulting dependency.** Implementation typically requires SAP integration specialists. HighRadius implementations routinely take 3-6 months with dedicated consultants.

### Model-to-Data

Onboarding is designed to be a deployment task, not an integration project:

- **Cloud Connector configuration** for read-only access to the seven CDS views and two config tables.
- **Automated config reads** of `T052` and `T047` to resolve payment terms and dunning procedures without human mapping.
- **Receivables G/L account identification** -- the one per-client step, but bounded (identify which G/L accounts are receivables and cash, not the full Chart of Accounts).
- **Docker container deployment** in AI Core via standard Argo Workflows.

The onboarding promise is: days, not months. No consulting project, no schema mapping workshops, no ETL pipeline to build or maintain.

### Assessment

Model-to-data has a genuine advantage here -- possibly its strongest operational selling point. But the advantage depends on the scope staying narrow. The moment scope expands (Phase 2 config-enabled analytics for Controlling, Billing, Cash Management), the onboarding complexity grows toward the data-to-model baseline. The simplicity is a function of the constraint, not of the architecture itself.

Established competitors have also reduced their onboarding friction over time. HighRadius and Serrala have pre-built SAP adapters refined across hundreds of implementations. Their onboarding is no longer a greenfield ETL project -- it is a configured deployment. The gap between model-to-data's "days" and a mature competitor's "weeks" is smaller than the gap between model-to-data and a first-time custom Databricks integration ("months").

---

## Model Quality and Training Dynamics

### Data-to-Model

- **Training data volume.** The model trains on all extracted data -- historical depth is limited only by extraction scope. Multi-year history is standard.
- **Feature richness.** Arbitrary features can be engineered from the full data lake: cross-system joins, temporal aggregations at any granularity, enrichment with external signals.
- **Model flexibility.** Any ML framework (PyTorch, XGBoost, LightGBM, transformer architectures) can be used without compute constraints beyond what the platform provides.
- **Multi-tenant learning (centralized).** Vendors like HighRadius and Sidetrade train models on pooled data from hundreds of clients. This is functionally equivalent to federated learning but with full data visibility. The statistical power is higher because the aggregation happens on raw features, not compressed gradients. Sidetrade's Aimie AI executed or recommended 5.1 million collection actions in 2025.

### Model-to-Data

- **Training data volume.** Limited to a single tenant's data. A mid-market Greek company might have 50,000-500,000 journal entries per year. Adequate for tabular ML but thin for deep learning architectures.
- **Feature richness.** Constrained to what the seven CDS views provide. Feature engineering is clever (clearing velocity, dunning escalation patterns, discount capture rates) but narrow.
- **Model flexibility.** SAP AI Core supports Docker containers, so any framework is technically usable. But the compute budget within a client's BTP tenant is constrained by their AI Core allocation, which is typically modest for mid-market companies.
- **Federated learning premium.** The global model benefits from cross-tenant gradient aggregation, which is the claimed quality advantage. But the privacy-utility trade-off (Differential Privacy noise) degrades the gradient signal. The network size at launch (targeting ~20 Greek tenants) spans FMCG, construction, shipping, manufacturing, and financial services -- highly heterogeneous. Research on federated learning for financial data suggests that meaningful cross-tenant benefit requires dozens to hundreds of participants with reasonably similar data distributions.

### Assessment

Data-to-model wins on raw model quality for any single client, especially through centralized multi-tenant vendors who pool data from hundreds of clients. Their "federated learning by centralization" is statistically superior because raw features contain more information than compressed, noise-injected gradients.

Model-to-data's quality proposition depends entirely on whether the federated network reaches sufficient density. At pilot scale, the single-tenant model is likely weaker than a mature centralized competitor's model. At scale (hundreds of tenants in homogeneous industry cohorts), the federated model could theoretically match centralized quality while preserving privacy. This is a hypothesis, not a demonstrated capability.

---

## Cost Structure

### Data-to-Model

- **Data platform licensing.** Databricks, Snowflake, or SAP Datasphere are not free. Datasphere was unbundled from RISE with SAP in July 2025 and is now a separate add-on, with minimum capacity at 1,532 CU/month (reduced from 4,300 in April 2025) at approximately USD 12-15 per CU annually. Databricks and Snowflake charge by compute consumption.
- **Data egress.** Moving data from S/4HANA's VPC to an external platform incurs network transfer costs. Standard hyperscaler egress rates are $0.085-$0.09 per GB. Over 50% of European firms report exceeding cloud budgets, with hidden egress fees cited as a primary driver.
- **ETL tooling.** Fivetran, Informatica, or equivalent connectors have per-connector pricing.
- **Storage.** Maintaining copies of SAP data in an external data lake adds storage costs that grow with retention depth and table count.
- **Implementation services.** The integration project itself (consultants, SAP basis support, testing) is a significant upfront cost.

Rough indicative range for a mid-market S/4HANA client: EUR 50K-200K implementation + EUR 30K-100K/year platform and tooling recurring.

### Model-to-Data

- **BTP/AI Core compute.** The Docker container consumes AI Core resources within the client's BTP tenant. If the client already has BTP (standard for RISE), the incremental cost is AI Core compute credits for training runs. AI Core is metered in node-hours with baseline charges capped at 730 hours/month.
- **No data platform licensing.** No Datasphere, no Databricks, no Snowflake.
- **No ETL tooling.** No connector licensing.
- **No data storage duplication.** The model reads live data and discards it after training.
- **AITYA subscription.** Replaces the platform costs with a single SaaS fee.

The cost structure is fundamentally different: model-to-data replaces a stack of platform licenses and data infrastructure costs with a single subscription. This is attractive for mid-market buyers who do not already have a data lake and do not want one. It is less compelling for enterprises that already operate Databricks or Snowflake environments -- they have sunk cost in data infrastructure, and adding one more SAP data source to an existing lake is an incremental project, not a new platform decision.

---

## S/4HANA Deployment Model Constraints

S/4HANA Public Cloud Edition increasingly restricts access to customizing tables and direct ABAP extensibility under SAP's "clean core" strategy:

| Access Pattern | Public Cloud | Private Cloud / On-Prem |
|---|---|---|
| Standard CDS Consumption Views (`I_*`) | Available via OData APIs | Available via RFC + Cloud Connector |
| Customizing tables (`T052`, `T047`, `T683S`, `SETNODE`) | Mostly **not accessible** | Fully accessible |
| Custom ABAP / CDS views | **Restricted** (Key User extensibility only) | Fully available |
| Data extraction (ODP/APRS) | SAP-managed data products only | Full flexibility |
| RFC/BAPI/direct database access | **Not supported** | Fully supported |

### Implications for Data-to-Model

SAP is addressing the extraction constraint through pre-built "data products" in SAP Business Data Cloud that handle extraction from Public Cloud. SAP wants data-to-model to work through *their* governed pipeline (BDC + Delta Sharing), not through raw table access. Databricks, Snowflake, and Google BigQuery are official partners in this architecture. The direction is clear: SAP is investing in making data-to-model frictionless for Public Cloud customers.

### Implications for Model-to-Data (AITYA)

The config table reads that enable automated normalization (`T052`, `T047`) work on Private Cloud and on-premise but may not be accessible on Public Cloud. The current target market (Greek mid-market, predominantly on-premise or Private Cloud) avoids this constraint. But if AITYA expands to Public Cloud Edition customers -- which is the growing segment of the market -- the automated onboarding advantage degrades because config tables become inaccessible.

This is a significant long-term risk for the model-to-data approach. SAP's product direction favors data-to-model through its own managed stack (BDC + Datasphere + AI Core). The clean core strategy progressively restricts the deep system access that model-to-data relies on. AITYA's architecture is well-suited for the current on-premise/private cloud install base but faces increasing friction as the market migrates to Public Cloud.

---

## Competitive Moat and Defensibility

### Data-to-Model

The moat for centralized competitors comes from:

- **Data accumulation.** HighRadius processes $4.6T+ in AR transactions annually. Sidetrade captures $1.7T across 285 million invoices. This pooled data trains models that no single client could build alone.
- **Domain-specific IP.** Years of feature engineering, model tuning, and AR-specific workflow optimization are embedded in the product.
- **Switching costs.** Once data pipelines are built, integrations configured, and workflows adopted, switching is expensive (though the EU Data Act now mandates reduced switching barriers).
- **SAP ecosystem embeddedness.** Serrala's FS2 is an SAP-certified ABAP add-on that deploys inside S/4HANA itself -- even more embedded than a BTP side-by-side extension.

This is a proven moat. HighRadius raised $300M+ in funding and serves hundreds of enterprise clients.

### Model-to-Data

AITYA's claimed moat is the federated network effect: each new participant improves the Global Base Model, creating a self-reinforcing advantage. The critique from [`pragmatic.md`](pragmatic.md) applies: this is a theoretical moat at 20 tenants and only becomes real at hundreds. During the gap between launch and critical mass, the moat does not exist.

The deployment architecture itself (Docker in AI Core, CDS-based feature engineering) is replicable by any competitor or by SAP. If the concept proves commercially valid, a Big 4 firm with existing SAP client relationships could build an equivalent product with deeper access and broader data scope.

### Assessment

Data-to-model's moat (centralized data pools, years of domain engineering, ecosystem integration) is proven and durable. Model-to-data's moat (federated network effect) is theoretical and size-dependent. At small scale, AITYA has no moat. At large scale, it would have a novel and potentially powerful one -- but reaching large scale requires crossing a valley of vulnerability where centralized competitors are unambiguously stronger.

---

## The SAP-Native Threat to Both Approaches

At Sapphire 2025, SAP announced Joule Agents that automate accounts receivable tasks directly inside S/4HANA -- no extraction, no side-by-side extension, no third party. This is a threat to *both* approaches. If SAP delivers "good enough" embedded AR intelligence, the question shifts from "model-to-data vs. data-to-model" to "why not just use what SAP ships?"

The native path has zero integration cost and zero data movement. It is shallow (HANA PAL, not deep learning), but for the majority of use cases that are pattern-based collection prioritization, shallow may suffice. SAP also announced its own foundation model (SAP-RPT-1, GA Q4 2025) and is integrating AI across the Business Suite.

Neither AITYA nor the centralized competitors can match SAP's system-level access. If SAP decides to build serious AR/Collections ML into the core product -- not just Joule prompts but actual predictive models -- the differentiation window for all third-party solutions narrows regardless of architecture.

---

## Where Model-to-Data Genuinely Wins

1. **The CISO conversation is qualitatively different.** "Nothing leaves" vs. "we encrypt it" is a binary distinction for many security teams. For companies with categorical prohibitions on financial data extraction -- and some European enterprises have them -- model-to-data is the only option. This is a market segment, not a market.

2. **Zero infrastructure overhead.** A Greek mid-market company with 200 users does not want to build and operate a data lake. They do not have the team. Model-to-data's BTP-native deployment is genuinely lighter for companies without existing data infrastructure.

3. **Cross-tenant benchmarking without data pooling.** The ability to answer "how does our DSO compare to industry peers?" without any company revealing their data is conceptually powerful. No centralized competitor can make this claim with the same privacy guarantee. Whether the mathematical privacy guarantee translates to perceived privacy in the buyer's mind is a separate question.

4. **Reconciliation-backed predictions.** Because the model reads directly from the ledger's CDS views, every prediction traces to posted journal entries that satisfy the accounting equation. This audit trail is architecturally inherent, not bolted on. Centralized competitors must reconstruct this chain after extraction, and the chain breaks at every transformation step.

5. **No vendor lock-in on the data side.** The client's data never leaves their system. If they stop using AITYA, there is nothing to migrate back -- the model simply stops running. With data-to-model, unwinding an integration means managing data copies, pipeline decommissioning, and potential retention obligations.

---

## Where Data-to-Model Genuinely Wins

1. **Analytical scope is unbounded.** The ability to join financial data with procurement, sales, HR, and external signals produces models that explain *why* things happen, not just *that* they happen. For a CFO trying to understand cash flow drivers, "your DSO is rising" is less useful than "your DSO is rising because your top 5 customers in construction are all experiencing project delays."

2. **Battle-tested at scale.** HighRadius, Sidetrade, Serrala, and Billtrust have processed billions of transactions across hundreds of clients. Their models are production-proven. AITYA's federated architecture has zero production deployments.

3. **SAP's own product direction validates it.** SAP Business Data Cloud, Delta Sharing, the Databricks/Snowflake/Google BigQuery partnerships all push toward data-to-model. SAP is investing billions in making data extraction easier, governed, and platform-native. AITYA is swimming against the current of SAP's own strategy.

4. **Centralized "federation" is statistically superior.** When HighRadius trains a model on pooled AR data from 500 clients, it has full access to raw features -- payment history, invoice attributes, customer profiles, industry codes. AITYA's federated model has access to compressed, noise-injected gradients from 20 clients. The statistical power is not comparable at current scale.

5. **Non-financial buyers exist.** Data-to-model serves the CDO, the Head of Data, the analytics team -- buyers who want a platform, not a point solution. Model-to-data serves the Head of AR and the CFO. The former are often the budget holders for analytics initiatives in large enterprises.

---

## Summary Matrix

| Dimension | Data-to-Model | Model-to-Data (AITYA) | Edge |
|---|---|---|---|
| Data sovereignty | Managed via contracts and encryption | Structural (data never leaves) | **Model-to-Data** |
| GDPR/AI Act compliance burden | High (DPAs, transfer assessments) | Low (local processing) | **Model-to-Data** |
| Analytical scope | Unlimited (cross-system joins) | Narrow (7 CDS views) | **Data-to-Model** |
| Onboarding speed | Weeks to months | Days (at current scope) | **Model-to-Data** |
| Model quality (single client) | Strong (rich features, deep history) | Constrained (narrow features, single tenant) | **Data-to-Model** |
| Multi-tenant learning | Proven (centralized pooling) | Unproven (federated at small scale) | **Data-to-Model** |
| Infrastructure cost | High (platform + ETL + storage) | Low (BTP-native compute only) | **Model-to-Data** |
| S/4HANA Public Cloud compatibility | Improving (SAP-managed data products) | At risk (config table access restrictions) | **Data-to-Model** |
| Competitive moat | Proven (data pools, domain IP) | Theoretical (network effect at scale) | **Data-to-Model** |
| Auditability | Reconstructed after extraction | Inherent (reconciliation equation) | **Model-to-Data** |
| Vendor lock-in for client | High (data copies, pipeline dependencies) | Low (nothing to migrate) | **Model-to-Data** |
| SAP strategic alignment | High (BDC, Delta Sharing, partnerships) | Contrarian (side-by-side, not data pipeline) | **Data-to-Model** |
| SAP-native threat exposure | High (Joule Agents automate AR natively) | High (SAP has deepest system access) | **Even** |

---

## Conclusion

Model-to-data wins on trust, simplicity, and cost for a specific buyer profile: mid-market, privacy-sensitive, no existing data infrastructure, on-premise or private cloud S/4HANA. Data-to-model wins on analytical power, proven scale, and alignment with where SAP and the broader market are heading.

AITYA's bet is that the trust-and-simplicity segment is large enough -- and underserved enough -- to build a business before the federated network reaches the scale where it becomes a genuine moat. That is a timing bet, not a technology bet. The CDS scope constraints that define the product are self-imposed trade-offs for deployability, not inherent limitations of what ML can do with S/4HANA data. Data-to-model competitors pay more (per-client integration) to access more (full data scope). AITYA pays less (automated onboarding) and accepts less (narrow scope). Whether that trade creates a viable business depends on whether the target buyer values what AITYA offers more than what it gives up.
