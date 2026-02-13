# Strategic Evaluation: AITYA Viability in the Age of SAP Zero-Copy Partnerships

*February 2026*

SAP's platform strategy has shifted decisively toward **making data extraction frictionless**, not preventing it. The November 2025 SAP-Snowflake partnership, the Databricks BDC Connect GA, and the planned Google BigQuery and Microsoft Fabric integrations collectively represent SAP investing billions to make the data-to-model path easier, cheaper, and governed. This is the single most important environmental change since AITYA's thesis was written, and it undermines several of AITYA's core assumptions while leaving a narrow -- but real -- window of opportunity.

For the product thesis and architecture, see [`pragmatic.md`](pragmatic.md). For the model-to-data vs. data-to-model comparison that preceded this analysis, see [`architecture_comparison.md`](architecture_comparison.md).

---

## (a) How SAP's Platform Strategy Affects Data Access Economics

### The Old Friction Is Disappearing

AITYA's cost advantage rested on the premise that extracting S/4HANA data into external platforms was expensive, slow, and sovereignty-hostile. As of early 2026, that premise is eroding:

- **SAP Snowflake** is [scheduled for Q1 2026 GA](https://news.sap.com/2025/11/sap-snowflake-data-enterprise-ai-business-data-fabric/), offering bi-directional zero-copy access. Data stays in SAP's object store; Snowflake queries it in place. No ETL pipeline, no replication, no medallion architecture.
- **BDC Connect for Databricks** reached GA across all hyperscalers in late 2025. [SAP is offering a 15% discount on SAP Databricks](https://www.sap.com/products/data-cloud/pricing.html) until April 2026 to drive adoption.
- **BDC Connect for Google BigQuery** is planned for [H1 2026](https://community.sap.com/t5/technology-blog-posts-by-sap/fast-track-ai-driven-insights-with-zero-copy-data-access-between-sap-bdc/ba-p/14283360); Microsoft Fabric for H2 2026.
- SAP announced plans to deliver [hundreds of pre-built "data products"](https://www.sap.com/events/sapphire/innovation-guide/data-cloud.html) covering the full Business Suite by end of 2025 -- curated, governed, semantically enriched data ready for external consumption.

All three partnerships use the same **BDC Connect** service layer: SAP exposes governed data products and BDC Connect shares them zero-copy to the partner platform. The consumer queries the data in place; it never physically moves.

### What These Partners Are -- and Are Not

A critical clarification: **Snowflake, Databricks, and BigQuery are data access infrastructure, not decision-support solutions.** They provide the plumbing (zero-copy access to SAP data), the compute environment (Snowpark, Databricks ML Runtime, Vertex AI), and generic ML building blocks (time-series forecasting, anomaly detection, LLM functions). They do **not** provide:

- Pre-built AR collection scoring models or dunning strategy optimization
- Domain-specific feature engineering for SAP financial data (e.g., mapping `I_JournalEntryItem` to DSO signals)
- Turnkey "plug in and get receivables predictions" capabilities
- Cross-company benchmarking intelligence

To go from "Snowflake can see my SAP data" to "I have a predictive collections system," a company still needs data scientists to define features from raw CDS views, build and validate ML models, create the application layer, and maintain the system over time. This is the gap that turnkey AR intelligence vendors like [HighRadius](https://www.highradius.com/resources/Blog/ar-automation-tools-guide/) and [Sidetrade](https://www.sidetrade.com/) fill -- and that SAP's own [Joule AR Agent](https://news.sap.com/2025/10/sap-connect-finance-ai-innovation/) aims to fill natively.

The threat to AITYA is therefore **indirect but real**: zero-copy partners do not compete with AITYA on intelligence, but they eliminate the data access friction that made building intelligence hard. Once Snowflake or Databricks can see SAP data zero-copy, a Big 4 team, a HighRadius implementation, or a custom ML pipeline becomes dramatically cheaper and faster to deploy -- eroding AITYA's "no data lake needed" speed advantage.

### Impact on AITYA's Cost Argument

The [`pragmatic.md`](pragmatic.md) cost comparison (EUR 50K-200K implementation + 30K-100K/year for data-to-model vs. a single AITYA subscription) assumed the old world of custom ETL, Fivetran connectors, and schema mapping workshops. Zero-copy via BDC collapses most of that. A mid-market company already on RISE with SAP could activate BDC, enable a data product, and give Databricks or Snowflake zero-copy access in **days, not months** -- the same onboarding speed AITYA claims as a differentiator.

The cost does not disappear entirely. [SAP Business Data Cloud uses Capacity Unit licensing](https://barc.com/sap-bdc-barc-perspective/) (~EUR 1/CU), and as of January 2026, SAC and Datasphere are no longer standalone but bundled into BDC. A mid-market company will pay for BDC + compute consumption on Snowflake/Databricks. But this cost is **shared across all analytics use cases**, not dedicated to AR/Collections alone. The marginal cost of adding receivables analytics to an existing BDC + Snowflake environment is approaching zero.

### The Licensing Trap for AITYA's Target Market

AITYA targets Greek mid-market companies, predominantly on-premise or private cloud. These companies are not yet on RISE and may not have BDC. For them, the old friction still exists. But SAP's entire go-to-market is pushing these companies toward RISE + BDC. [30% of organizations are now fully on S/4HANA Cloud Private Edition, up from 19% last year](https://erp.today/rise-with-sap-cloud-erp-adoption-accelerates-ahead-of-2027-deadline-sapinsider-benchmark-report/), and mid-market companies are twice as likely to be live on RISE compared to large enterprises. AITYA's addressable market is the segment that hasn't migrated yet -- and that segment is shrinking every quarter.

### Assessment

SAP's platform strategy is systematically lowering the data access barriers that AITYA's value proposition depends on. The cost advantage is real today for on-premise holdouts but has a visible expiration date. Within 2-3 years, zero-copy access will be the default for most S/4HANA customers, and the "no data lake needed" argument becomes moot.

---

## (b) Federated Analytics vs. Zero-Copy Solutions

As established in section (a), zero-copy partners are infrastructure providers that solve the **data movement** problem. They do not solve the **data intelligence** problem -- someone still needs to build domain-specific models and applications on top of zero-copy access. But a previous version of this section claimed that zero-copy also cannot solve the **data pooling** problem (learning across companies without sharing raw data). That claim was wrong. Zero-copy and federated learning operate at different layers and can be composed.

### Zero-Copy + Federated Learning: A Composable Architecture

A competitor could build cross-company intelligence on top of zero-copy infrastructure by layering federated learning on the data access layer:

1. **BDC Connect** gives each tenant's Snowflake/Databricks/BigQuery environment zero-copy access to their SAP data.
2. Each tenant trains a **local model** on their own data within their own compute environment.
3. Only **model parameters or gradients** (not raw data) are sent to a central aggregator.
4. The aggregator combines local updates into a **global model** and redistributes it.

This is standard cross-silo federated learning. Google Cloud already publishes a [reference architecture](https://cloud.google.com/architecture/cross-silo-cross-device-federated-learning-google-cloud) for exactly this pattern, using GKE + TensorFlow Federated (TFF) with [trusted execution environments (TEEs)](https://cloud.google.com/architecture/cross-silo-cross-device-federated-learning-google-cloud) for secure gradient aggregation. Databricks has distributed training primitives (TorchDistributor) and a [published framework for FL implementation](https://www.ijfmr.com/research-paper.php?id=55515), though it is not a native product feature. Snowflake has no federated learning capabilities.

The implication is severe for AITYA: the Google Cloud stack (BDC Connect for BigQuery + Vertex AI + TFF + TEEs) could deliver the same outcome AITYA claims as unique -- cross-company learning without raw data leaving any tenant -- on Google's infrastructure, with Google's security certifications, and without AITYA's novel (unproven) architecture.

### What Zero-Copy + FL Does Not Provide (Yet)

This composite architecture is feasible but does not exist as a turnkey product today. To get from "the components exist" to "a working cross-company AR intelligence system," someone would still need to:

1. Build the **FL orchestration layer** -- tenant enrollment, training scheduling, gradient aggregation, model versioning -- on top of Google's reference architecture or from scratch on Databricks.
2. Develop **domain-specific feature engineering** for SAP financial data (mapping `I_JournalEntryItem` and `I_DunningHistory` to ML-ready features).
3. Solve the **privacy engineering** -- differential privacy budgets, secure aggregation protocols, compliance certification for gradient sharing across EU tenants.
4. Create the **application layer** -- dashboards, alerts, collection recommendations -- that turns model outputs into business decisions.

None of this is trivial. But none of it is architecturally blocked, either. The barrier is engineering effort and domain expertise, not a fundamental capability gap that only AITYA can fill.

### Why the Federated Value Proposition Is in Trouble

**Production readiness.** [Only 5.2% of federated learning research has reached production deployment](https://introl.com/blog/federated-learning-infrastructure-privacy-preserving-enterprise-ai-guide-2025). AITYA would be attempting a novel FL architecture in a domain (SAP financial data) where no precedent exists. The [FL market was $0.1B in 2025](https://www.360iresearch.com/library/intelligence/federated-learning-solutions), overwhelmingly in healthcare imaging and fraud detection -- not ERP analytics.

**Centralized competitors already do "federation" better.** HighRadius processes [$4.6T+ in AR transactions annually](https://www.highradius.com/resources/Blog/ar-automation-tools-guide/) across [430+ customers](https://6sense.com/tech/accounts-receivable/highradius-vs-sidetrade). Sidetrade captures $1.7T across 285 million invoices. They achieve cross-company intelligence by pooling centralized data -- statistically superior to compressed, noise-injected federated gradients from 20 Greek companies.

**SAP is building native AR intelligence.** SAP announced an [Accounts Receivable Agent](https://news.sap.com/2025/10/sap-connect-finance-ai-innovation/) at SAP Connect 2025 that analyzes overdue receivables and automates follow-up. A [Cash Management Agent](https://www.sap.com/products/artificial-intelligence/ai-agents/agent-use-cases.html) automates reconciliation with up to 70% time savings, GA planned Q1 2026. These are embedded, zero-integration, zero-data-movement solutions. They are shallow today. They will not be shallow in 3 years.

**The network effect requires density that doesn't exist.** 20 Greek companies spanning energy, FMCG, manufacturing, telecom, and logistics is too heterogeneous for meaningful federated signals. As [`pragmatic.md`](pragmatic.md) itself acknowledges, "a federated model with three Greek retailers may produce noisy, unreliable benchmarks." The minimum viable cohort for a single industry vertical in a single geography likely needs 30-50+ participants with similar data distributions. Greece cannot provide this.

### Where Genuine Differentiation Survives

Given that zero-copy + FL is composable and that centralized competitors already achieve cross-company intelligence through data pooling, AITYA's remaining differentiation is narrow:

1. **Time-to-market.** The composite zero-copy + FL architecture does not exist as a product today. AITYA could be first to deliver it for SAP financial data. But this is a timing advantage, not a structural one -- it erodes as Google Cloud, Databricks, or a systems integrator assembles the same stack.
2. **Pre-built SAP financial domain models.** Feature engineering from `I_JournalEntryItem`, `I_GLAcctBalance`, and `I_DunningHistory` into ML-ready representations (DSO signals, payment probability features, cash conversion metrics) is non-trivial domain work. A competitor using zero-copy + TFF would still need to build this layer.
3. **Orchestration simplicity.** A single integrated product vs. a multi-vendor composite (BDC + BigQuery + Vertex AI + TFF + custom app layer) has a UX and operational cost advantage for mid-market buyers who lack platform engineering teams.

None of these are durable moats. They are head-start advantages that buy time -- plausibly 2-3 years before the composite stack matures or a larger player (SAP, Google, HighRadius) packages it.

---

## (c) Realistic Go-to-Market Segments and Revenue Models

### Option 1: Single-Tenant "Zero-Integration" AR Analytics (Most Viable)

Drop federation as the core value proposition. Reposition as the fastest, lightest AR analytics deployment for on-premise/private cloud S/4HANA customers who haven't built data infrastructure yet.

- **Target:** Greek mid-market (the [20 companies](greek_market.md) identified) and similarly positioned companies across Southern/Eastern Europe.
- **Value prop:** "Get predictive collections intelligence in days, not months, with zero data extraction." The model-to-data speed advantage without the federated complexity.
- **Revenue model:** Annual SaaS subscription, EUR 30K-80K per tenant. Low enough to avoid enterprise procurement cycles; high enough to build a business on 20-50 customers.
- **Moat:** Weak. Onboarding speed and reconciliation-backed auditability are real but replicable advantages. This is a race to land customers before SAP Joule Agents and zero-copy make this unnecessary.
- **Honest assessment:** A viable small business (EUR 1-3M ARR) with a 3-5 year window. Not a venture-scale opportunity because the addressable market shrinks as SAP's platform improves.

### Option 2: Federated Benchmarking as Premium Upsell (Conditional)

Keep single-tenant as the entry product. Offer federated benchmarking as a premium tier once 15+ tenants are onboarded within a single industry-geography cohort.

- **Target:** Greek FMCG/Retail (Coca-Cola HBC, Sklavenitis, etc.) or Energy (PPC, Motor Oil, HELLENiQ) -- the two sectors with enough density for cohorts.
- **Revenue model:** Base subscription (single-tenant) + EUR 15K-30K annual premium for federated benchmarking tier.
- **Prerequisite:** Achieving sufficient density in a single cohort. In Greece, this means 5+ companies in the same sector -- plausible for Energy (5 targets) but tight for FMCG (3-4 targets after excluding Swiss-HQ Coca-Cola HBC).
- **Honest assessment:** The federated premium is the strategic differentiator but cannot be the launch product. It is a Phase 2 value-add that depends on Phase 1 succeeding.

### Option 3: White-Label Analytics Engine for SAP Partners (Alternative GTM)

Instead of selling directly to enterprises, sell the technology to [SAP partners](sap_partners.md) (Real Consulting, TEKMON) as a white-label capability they embed in their post-migration services.

- **Target:** The 5 Greek SAP partners, plus SAP partners in other Southern European markets (Italy, Spain, Portugal with similar on-premise profiles).
- **Revenue model:** Revenue share (15-25% of partner subscription pricing) or per-tenant licensing to the partner.
- **Advantage:** Avoids the startup-to-enterprise credibility gap. Partners have existing client relationships, CISO trust, and implementation access.
- **Risk:** Partners may white-label the capability, learn the approach, and replicate it. The "Black Box" Docker container protects code, not concepts. The Big 4 co-opetitor risk identified in [`sap_partners.md`](sap_partners.md) is real.

### Option 4: Niche "Sovereign AI" Play for Regulated Industries (Long Shot)

Position AITYA specifically for industries where data sovereignty is not a preference but a regulatory requirement.

- **Target:** EU critical infrastructure operators, financial institutions under ECB supervision, defense contractors.
- **Value prop:** "AI analytics that satisfies your regulator's data residency requirements without any architectural exceptions."
- **Revenue model:** Premium subscription (EUR 80K-150K/year) justified by compliance value.
- **Risk:** These buyers move slowly, require extensive security certification (ISO 27001, SOC 2, C5), and demand capabilities far beyond AR analytics. The narrow two-pillar scope may be too limited for regulated enterprise buyers who expect platform-level solutions.

---

## The Hard Questions AITYA Must Answer

1. **Is this a timing play or a structural advantage?** SAP's zero-copy partnerships are closing the data access gap that AITYA exists to fill. If the answer is "timing play," the business plan needs to specify what AITYA becomes after that window closes. If the answer is "structural," the plan needs to explain why federated gradient aggregation remains superior to centralized pooling as network sizes grow -- and there is no evidence it does.

2. **Why would a Greek CFO choose AITYA over HighRadius?** HighRadius has a [SAP-certified extractor](https://www.highradius.com/resources/Blog/ar-automation-tools-guide/), 430+ customers providing cross-company intelligence, proven production models, and Big 4 partnerships of its own. AITYA has a thesis. The [AR Automation market is growing to $6.57B by 2031](https://www.mordorintelligence.com/industry-reports/accounts-receivable-automation-market), but that growth benefits incumbents first.

3. **What happens when SAP Joule AR Agent reaches maturity?** SAP's embedded AR Agent is free with the S/4HANA license, requires zero integration, and has the deepest possible system access. It is shallow today. It will not be shallow in 3 years. If SAP invests in making Joule AR "good enough" for 80% of use cases, AITYA's addressable market collapses to the 20% that need more -- and those 20% are the ones who will choose HighRadius or Sidetrade, not an unproven startup.

4. **Can you survive the valley of vulnerability?** The federated moat only activates at scale. At 5-10 tenants, there is no moat. HighRadius, Sidetrade, or SAP itself could replicate the *outcome* (not the code, but the business result) faster than AITYA can accumulate enough tenants for the network effect to matter. What is the plan for the 2-3 years when you are competing without your claimed differentiator?

5. **Is the Greek market a launchpad or a dead end?** 20 target companies across 5 industries in a single small economy. If the federated play requires industry-homogeneous cohorts of 15+, Greece alone cannot provide them. The plan needs a credible path from Greece to pan-European scale, and that path runs through markets (Germany, France, Nordics) where competitors are stronger and on-premise is less prevalent.

---

## Bottom Line

There is a viable business in AITYA's architecture, but it is smaller, less defensible, and more time-constrained than the business plan implies. The honest opportunity is:

**A fast-deployment, privacy-preserving AR analytics tool for mid-market European companies that haven't yet migrated to SAP's cloud data stack.** This is a real gap today. It will narrow as SAP's BDC + zero-copy partnerships mature, as Joule AR Agents improve, and as the on-premise S/4HANA population shrinks. The federated benchmarking layer is a genuine differentiator but one that depends on achieving network density that the current market targeting cannot provide.

The strategic choice is whether to pursue this as a **bootstrapped niche business** (viable, bounded, honest) or a **venture-backed platform play** (requires the federated network effect to materialize at scale, which has no production precedent in ERP analytics and faces structural headwinds from SAP's own platform direction).

---

## Sources

- [SAP and Snowflake Partnership Announcement (Nov 2025)](https://news.sap.com/2025/11/sap-snowflake-data-enterprise-ai-business-data-fabric/)
- [SAP Snowflake Data Fabric Innovations](https://news.sap.com/2025/11/sap-snowflake-new-data-fabric-innovations-sap-bdc-sap-hana-cloud/)
- [BARC Perspective on SAP x Snowflake](https://barc.com/barc-perspective-on-the-sap-x-snowflake-partnership/)
- [SAP BDC + Google BigQuery Zero-Copy Access](https://community.sap.com/t5/technology-blog-posts-by-sap/fast-track-ai-driven-insights-with-zero-copy-data-access-between-sap-bdc/ba-p/14283360)
- [SAP BDC Pricing](https://www.sap.com/products/data-cloud/pricing.html)
- [BARC Perspective on SAP BDC](https://barc.com/sap-bdc-barc-perspective/)
- [SAP Connect 2025: Finance AI Innovation](https://news.sap.com/2025/10/sap-connect-finance-ai-innovation/)
- [SAP Joule Agent Use Cases](https://www.sap.com/products/artificial-intelligence/ai-agents/agent-use-cases.html)
- [SAP Sapphire Innovation Guide: Business Data Cloud](https://www.sap.com/events/sapphire/innovation-guide/data-cloud.html)
- [RISE with SAP Adoption Benchmark (SAPinsider 2025)](https://erp.today/rise-with-sap-cloud-erp-adoption-accelerates-ahead-of-2027-deadline-sapinsider-benchmark-report/)
- [AR Automation Market Size (Mordor Intelligence)](https://www.mordorintelligence.com/industry-reports/accounts-receivable-automation-market)
- [Federated Learning Infrastructure Guide 2025](https://introl.com/blog/federated-learning-infrastructure-privacy-preserving-enterprise-ai-guide-2025)
- [Federated Learning Solutions Market](https://www.360iresearch.com/library/intelligence/federated-learning-solutions)
- [SAP Clean Core Extensibility Guide](https://community.sap.com/t5/technology-blog-posts-by-sap/abap-extensibility-guide-clean-core-for-sap-s-4hana-cloud-august-2025/ba-p/14175399)
- [Google Cloud: Cross-Silo Federated Learning Reference Architecture](https://cloud.google.com/architecture/cross-silo-cross-device-federated-learning-google-cloud)
- [Federated Learning Implementation Framework using Databricks (IJFMR)](https://www.ijfmr.com/research-paper.php?id=55515)
- [SAP Datasphere Annual Review 2025](https://www.nextlytics.com/blog/sap-datasphere-annual-review-2025)
- [SAP Data & Analytics Licensing Playbook](https://redresscompliance.com/sap-data-analytics-licensing-strategic-playbook-for-cios/)
