# AITYA -- Summary of Key Ideas

**Tagline:** *The reason behind the data.*

---

## Company Identity

AITYA is a planned company whose name operates on multiple levels: it echoes the Greek word for "reason" and the English word "idea," while the prefix "AI" signals Artificial Intelligence and the letter "Y" evokes "why" -- the question at the heart of every analytical pursuit. The company's mission is to extract strategic meaning from enterprise data that currently sits underutilized inside SAP S/4HANA systems.

---

## Product Thesis

The full product thesis -- including the core idea, federated architecture, the two semantic pillars, deployment architecture, and product definition -- is maintained in [`pragmatic.md`](pragmatic.md). What follows is a condensed overview.

### The Problem

Enterprises on SAP S/4HANA possess the Universal Journal (`ACDOCA`) but lack the intelligence to turn it into proactive strategy. Centralized AI requires exporting sensitive data (security risks, egress costs, sovereignty loss). Shallow analytics stay inside the ERP but only describe the past.

### Federated Zero-Export Intelligence

AITYA resolves this trade-off with federated ML: models train locally inside each client's BTP tenant; only encrypted gradients leave. Benefits include zero data extraction, regulatory compliance by design (GDPR, EU AI Act), collective learning, and cross-company benchmarking. Risks include gradient inversion attacks, cross-industry signal dilution, CISO resistance, aggregator trust dependency, regulatory uncertainty, SAP zero-copy platform erosion (BDC + Snowflake/Databricks/BigQuery are eliminating the data access friction AITYA depends on), and established AR intelligence competitors (HighRadius: 430+ customers; Sidetrade: 285M invoices; SAP Joule AR Agent). See [`pragmatic.md`](pragmatic.md) for the full benefits-and-risks analysis.

### The Two Semantic Pillars: Ledger and Dunning

The product scope is narrowed to two data domains that share three properties making them viable for tenant-agnostic federated feature engineering:

1. **Governed by double-entry accounting** -- self-validating via the reconciliation equation.
2. **Minimal customization surface** -- receivables/cash G/L structures and dunning levels are among the most standardized elements in S/4HANA.
3. **Universal business language** -- DSO, cash conversion cycle, bad debt ratio require no buyer education.

**Ledger Pillar:** `I_JournalEntryItem` (granular transactions), `I_GLAcctBalance` (aggregated balances), and `I_OperationalAcctgDocItem` (clearing and payment behavior). Enables receivables trending, reconciliation-backed validation, segment-level exposure quantification, and invoice-level payment punctuality analysis. Clearing is binary at the line-item level (open or cleared), but partial payments introduce ambiguity: SAP's two partial payment methods (partial payment vs. residual payment) produce incompatible data signatures in the same view, and feature engineering must detect and normalize both patterns. See [`pragmatic.md`](pragmatic.md) for the full treatment.

**Dunning Pillar:** `I_DunningHistory` (collection escalation trail). Enables invoice-level risk scoring, collection prioritization, and early-warning detection.

**Master Data Layer:** `I_CompanyCode` (entity context: country, currency) and `I_Customer` (customer context: geography, industry sector, payment terms). Not a third pillar -- these are dimension tables that provide the segmentation axes for the transactional pillars. Industry-scoped federation depends on `I_Customer`.`IndustrySector` being populated. Only fields with external governance (ISO country/region codes) or SAP-delivered code lists are trusted; free-text fields (city, address) and company-specific taxonomies (customer group) are excluded.

**Configuration Layer:** Two SAP customizing tables (`T052` for payment terms, `T047` for dunning procedures) are read once at deployment. These contain the machine-readable arithmetic behind company-configured codes -- actual day counts, discount percentages, dunning intervals, and escalation levels. This automates cross-tenant normalization for the two largest sources of behavioral variation, eliminating manual mapping work during onboarding. The payment terms infrastructure is more layered than the summary suggests: `T052` supports four complexity layers (simple terms, day limits, fixed-day/end-of-month rules, installment plans via `T052S`), all deterministic arithmetic but requiring progressively more complex canonical representations and comparison functions. End-of-month terms (Layer 3) are standard in European B2B and installment plans (Layer 4) are common in capital-intensive industries -- both directly relevant to the Greek target market. See [`payment_terms.md`](payment_terms.md) for the full table architecture and gap analysis.

**Together:** The data sources form a complete chain: ledger shows what is owed and at what magnitude, clearing shows when and how it was actually paid, dunning shows what happened when it was not paid, master data provides the geographic and industry axes for segmentation, and the configuration layer normalizes policy differences automatically. Combined capabilities include receivables-at-risk quantification, payment behavior prediction, cash discount leakage quantification, cash conversion tracking, and reconciliation-backed audit trails.

**Cross-tenant dunning policy evaluation:** The federated network also enables a prescriptive capability beyond descriptive benchmarking. Because different tenants configure different dunning procedures via `T047` (varying intervals, level counts, thresholds), the network observes clearing outcomes under natural policy variation. This supports offline policy evaluation: estimating which dunning configurations produce better cash conversion for similar customer profiles. The capability is limited by confounding (tenants differ in more than just dunning policy), unobservable collection actions (phone calls, negotiations are not in the CDS scope), and cohort size requirements. See the full analysis including risks in [`pragmatic.md`](pragmatic.md).

**RL feasibility boundary:** The current scope does not support genuine reinforcement learning. The only observable action is dunning level escalation -- a deterministic automation, not a human decision. Real collection actions (calls, payment plans, legal escalation) are not recorded in the seven-view scope. The data supports supervised prediction (which customers will pay late) and cross-tenant policy comparison (which dunning configs work better), but not state-action value estimation. See [`rl_feasibility.md`](rl_feasibility.md) for the full analysis.

**Risks of the two-pillar scope:** May be too narrow for enterprise buyers; receivables mapping is simpler but not trivial; dunning process variation and payment terms are auto-normalized via config reads but residual risk remains for edge cases (operational delays vs. configured intervals, complex installment terms); partial clearing ambiguity complicates feature engineering (two methods, incompatible data signatures, potential dependency on BSEG field `REBZG` not exposed in the CDS view); `ClearingAccountingDocument` deprecation in recent S/4HANA releases introduces forward-compatibility risk; missing context without supply/sales data limits root-cause attribution.

**Phase 2 scope expansion via model-to-data:** The two-pillar boundary applies to federated ML, where features must be cross-tenant portable. The model-to-data pillar imposes a weaker constraint: the pipeline needs only to read one tenant's configuration. When the cross-tenant requirement is dropped, SAP config tables become an automated feature schema — the pipeline reads hierarchies (`SETNODE`), pricing procedures (`T683S` → `T685`), and bank topology (`T012`) to structure features without human mapping. This makes Controlling, Billing, and Cash Management conditionally viable for single-tenant analytics. A hard architectural constraint applies: expanded features from these domains must not enter the federated gradient pipeline, because config-assisted category mappings introduce cross-tenant semantic variability that the aggregation server cannot detect or correct — unlike `T052`/`T047` parameters, which are arithmetic and identical everywhere, category mappings are judgment calls whose consistency degrades across tenants and time. Phase 2 must maintain strict separation between federated features (narrow, two-pillar, alignment by construction) and single-tenant features (broad, config-assisted, per-tenant semantics only). This is documented as a Phase 2 option, not a current commitment — pursuing it before Cash & Collections is validated would dilute the launch product and risk weakening the federated moat. Access depends on on-premise/private cloud deployment; S/4HANA Public Cloud blocks most config table reads. See [`cds_analysis.md`](cds_analysis.md) for the full analysis.

### The Product: Cash & Collections Intelligence

A predictive analytics engine answering: (1) which customers will fail to pay and what to do about it, and (2) what is the balance-sheet impact and is it improving. Pitched in standard financial KPIs (DSO, bad debt, working capital), not proprietary metrics.

### Deployment

Side-by-Side BTP via SAP AI Core (Docker containers). Read-only access to seven CDS views (four transactional, two master data, one underlying journal) plus two customizing tables read once at deployment for automated normalization. Local training with gradient-only export. Built-in data quality validation via the ledger reconciliation equation.

**BTP vs. on-premise tension:** The BTP/AI Core deployment assumption misaligns with the Greek target market, where 7 of 10 confirmed S/4HANA companies are on-premise or IaaS (not RISE). Federated learning does not technically require BTP -- it needs local compute, CDS view access, and outbound gradient connectivity, all achievable on-premise. On-premise customers also have *better* config table access and face *no* zero-copy competitive threat (BDC Connect requires RISE). The segment where AITYA's competitive position is strongest is the segment where the current deployment architecture does not work. Three options are evaluated in [`pragmatic.md`](pragmatic.md): on-premise container deployment (expands market from 3 to 10, adds operational complexity), BTP-only (accepts narrow base), or a hybrid approach (most realistic, splits engineering effort).

---

## Revenue Forecast (3-Year)

See [`revenue_forecast.md`](revenue_forecast.md) for full assumptions.

| Metric | Year 1 | Year 2 | Year 3 |
|---|---|---|---|
| Total active customers | 5 | 30 | 100 |
| Net revenue (projected) | $515K | $3.59M | $13.23M |

---

## Target Market: Greece as Test Bed

See [`greek_market.md`](greek_market.md) for the full list and verified SAP deployment status.

A February 2026 audit expanded the original 20-company target list to 34 confirmed or likely SAP users, while also disqualifying 7 companies as non-SAP or non-viable. The market is larger than previously documented but still cloud-thin:

- **3 companies** are confirmed on RISE with SAP Cloud: HELLENiQ ENERGY (energy), Sarantis Group (consumer products, first worldwide RISE Premium Plus customer), and Piraeus Bank (banking, finalized 2024). This is up from 1 in the prior audit.
- **7 additional companies** are confirmed on S/4HANA but on-premise, private cloud, or Azure IaaS (Motor Oil, OPAP, Coca-Cola HBC, Titan Cement, Frigoglass, ADMIE/IPTO, Plaisio migrating).
- **10 companies** are confirmed SAP users with unclear version/deployment (OTE/Cosmote transitioning via Deutsche Telekom RISE deal, Viohalco, ElvalHalcor, National Bank of Greece, Eurobank, Mytilineos, TERNA ENERGY, Lamda Development, Elgeka, Chipita).
- **14 additional companies** appear on SAP partner customer lists (ELSOP) or SAP event attendee lists with limited public detail.
- **7 companies** are confirmed non-SAP or non-viable (PPC/DEI, Sklavenitis, Aegean Airlines, ELTA, Pavlidis Marbles, Folli Follie, Autohellas).

**Impact:** The addressable market for AITYA's federated architecture under the current BTP-only deployment assumption is 3 confirmed RISE customers today -- an improvement, but they span 3 different industries (energy, FMCG, banking), so industry-homogeneous cohorts remain impossible in Greece alone. The single-tenant option targets ~10 confirmed S/4HANA companies (up from ~7). The 2027 SAP ECC end-of-life deadline will force Tier 3 and Tier 4 companies to migrate, expanding the medium-term pipeline. Market sizing in [`pragmatic.md`](pragmatic.md) still needs revision.

**Why Greece was chosen:** High concentration (few companies, large GDP share), dense SAP talent pool in Athens, EU RRF funding for AI/ML extensions. These advantages remain. The SAP footprint is broader than originally feared (~34 SAP users) but cloud readiness is still thin (3 RISE customers across 3 industries). Greece provides pilot diversity but not federated cohort density.

---

## Go-to-Market: SAP Partners in Greece

See [`sap_partners.md`](sap_partners.md) for the five target partners (Real Consulting, PwC Greece, TEKMON, Deloitte Greece, IBM Greece) and the Big 4 co-opetitor strategy.

---

## Architecture Comparison: Model-to-Data vs. Data-to-Model

See [`architecture_comparison.md`](architecture_comparison.md) for the full analysis.

AITYA's model-to-data (federated, zero-export) architecture is compared against the traditional data-to-model (centralized extraction) approach across twelve dimensions. Key findings:

- **Model-to-data wins on:** data sovereignty (structural, not contractual), GDPR/AI Act compliance burden, onboarding speed (days vs. months at current scope), infrastructure cost (no data platform licensing), auditability (reconciliation-backed predictions), and vendor lock-in for the client (nothing to migrate).
- **Data-to-model wins on:** analytical scope (unlimited cross-system joins vs. 7 CDS views), model quality (rich features + centralized pooling from hundreds of clients), competitive moat (proven data pools vs. theoretical network effect), S/4HANA Public Cloud compatibility (SAP-managed data products vs. restricted config table access), and SAP strategic alignment (BDC, Delta Sharing, Databricks/Snowflake partnerships all favor data-to-model).
- **The CDS exclusion criteria are self-imposed trade-offs.** The domains excluded in [`cds_analysis.md`](cds_analysis.md) (Controlling, Billing, Cash Management) fail the *cross-tenant portability* requirement, not the *ML usability* requirement. Data-to-model competitors solve the customization problem at the integration layer (per-client mapping during implementation) rather than at the architecture layer (selecting only self-normalizing data). Every domain AITYA excludes is already consumed by centralized competitors who accepted the per-client integration cost.
- **SAP-native threat.** SAP's Joule Agents (Sapphire 2025) automate AR tasks directly inside S/4HANA with zero integration cost, threatening both approaches equally.

The bottom line: model-to-data is a timing bet that a trust-and-simplicity buyer segment is large enough to sustain a business before the federated network reaches moat-scale. Data-to-model competitors pay more per-client to access more data. AITYA pays less and accepts less.

---

## Zero-Copy Viability Assessment (February 2026)

See [`zero_copy_viability.md`](zero_copy_viability.md) for the full analysis.

SAP's November 2025 Snowflake partnership, the Databricks BDC Connect GA, and planned Google BigQuery / Microsoft Fabric integrations represent a decisive shift toward frictionless data-to-model access. Key findings:

- **Data access economics are shifting against AITYA.** Zero-copy via BDC collapses the ETL/data-lake cost structure that AITYA's pricing advantage depends on. A mid-market RISE customer can now activate BDC + Snowflake zero-copy access in days -- matching AITYA's claimed onboarding speed. The cost advantage persists for on-premise holdouts but has a visible 2-3 year expiration date as SAP pushes the market toward RISE + BDC.
- **Federated analytics retains one genuine differentiator:** privacy-preserving cross-company benchmarking without data pooling. Zero-copy solves data movement but not data pooling. However, the federated value proposition faces production readiness challenges (5.2% of FL research reaches production), centralized competitors with superior statistical power (HighRadius: 430+ customers, $4.6T+ AR transactions), and SAP's own Joule AR Agent (GA Q1 2026).
- **The Greek market cannot provide federated density.** A February 2026 audit expanded the target list to ~34 confirmed SAP users (of which 10 are on S/4HANA and 3 are on RISE with SAP Cloud). However, the 3 RISE customers span 3 different industries (energy, consumer products, banking) and cannot generate the 30-50+ homogeneous-cohort participants needed for meaningful federated signals. Greece is a viable pilot for single-tenant analytics but insufficient for the federated network effect.
- **Four GTM options evaluated:** (1) Single-tenant zero-integration AR analytics (most viable, EUR 1-3M ARR, 3-5 year window); (2) Federated benchmarking as Phase 2 premium upsell (conditional on achieving cohort density); (3) White-label engine for SAP partners (avoids credibility gap but risks replication); (4) Sovereign AI for regulated industries (long shot, narrow scope insufficient for regulated buyers).
- **The core strategic question:** bootstrapped niche business (viable, bounded) vs. venture-backed platform play (requires federated network effect to materialize at scale, no production precedent in ERP analytics).

---

## Network Effects Beyond Federated Learning (February 2026)

See [`network_effects.md`](network_effects.md) for the full analysis.

The federated gradient accumulation claimed as a "network-effect moat" in [`pragmatic.md`](pragmatic.md) is more accurately a data scale advantage (linear, asymptotic) than a genuine network effect (non-linear, increasing returns). Four alternative network effect strategies were evaluated:

- **Cross-tenant benchmarking:** Genuine same-side network effect but weak -- asymptotes at 30-50 participants, easily replicated, and requires industry-homogeneous cohorts that the Greek market cannot provide at launch (3 RISE customers across 3 different industries).
- **Cross-entity counterparty graph:** The strongest genuine network effect available -- non-asymptotic, defensible at scale, central to the value proposition. But faces three independent feasibility barriers: (1) geographic/industry density that a startup cannot engineer quickly, (2) direct competition against established credit intelligence providers (Dun & Bradstreet, Coface) who already own this graph at global scale, and (3) **structural GDPR and EU AI Act obstacles.** The counterparty graph is functionally a credit information service -- aggregating entity-level payment behavior across creditors. GDPR purpose limitation (Art. 5(1)(b)) challenges repurposing billing data for cross-entity credit assessment. The CJEU SCHUFA ruling (C-634/21, December 2023) classifies credit score production as automated decision-making under Art. 22. The October 2025 Experian fine (EUR 2.7M) and Austrian KSV1870 prohibition demonstrate active DPA enforcement. If scored counterparties include sole proprietors (likely in the Greek SME-heavy market), the EU AI Act classifies the system as high-risk (Annex III, 5(b)), requiring conformity assessment by August 2026. Greece's Tiresias-governed credit information regime adds national regulatory complexity. These barriers may make the feature legally unshippable without counterparty consent -- and consent-gated network effects do not scale.
- **Two-sided marketplace (analytics modules):** Not viable at sub-100-tenant scale. Requires user density that vertical B2B analytics products rarely achieve.
- **Community/expertise effects:** Not viable at launch scale. Requires hundreds of active users and years of investment to reach critical mass.

**No network effect strategy is viable at AITYA's launch scale.** The first 10-20 customers will buy on product quality and domain expertise, not network effects. Recommended sequencing: launch with benchmarking (sales tool, not a moat), **commission a GDPR/AI Act legal assessment before investing in counterparty graph architecture** (the regulatory barriers are structural, not just perceptual -- legal clarity must precede architecture investment), defer marketplace and community strategies.

---

## Strategic Path Forward

1. **Launch with Cash & Collections Intelligence** -- the two-pillar MVP with dual ROI (collection efficiency + working-capital visibility).
2. **Prove tenant-agnostic feature engineering** on the narrow five-view scope before expanding.
3. **Pilot in Greece** with 3-5 anchor clients to build a localized federated model.
4. **Pursue SAP PartnerEdge Build certification** for SAP Store distribution.
5. **Expand single-tenant scope via config-enabled feature organization** -- Controlling, Billing, and Cash Management become viable when the pipeline reads tenant-specific config trees (`SETNODE`, `T683S`/`T685`, `T012`) as an automated feature schema. This is a model-to-data expansion, not a federated expansion; these domains cannot be federated. See [`cds_analysis.md`](cds_analysis.md).
6. **Expand federated pillars incrementally** -- Asset Accounting is the only new domain that passes the federated ML viability test. Supply and Growth remain blocked by customization surface.
7. **Build a direct customer relationship** alongside partner channels to avoid dependency risk.
