# The Greek Market

*Updated February 2026 with expanded research across SAP case studies, partner customer lists (ELSOP, TEKA Systems, Real Consulting), SAP event attendees, Microsoft case studies, press releases, and job postings*

## Introduction

This document maps Greek enterprises by their SAP S/4HANA adoption status, organized by cloud readiness tier. The research identified **35 confirmed or likely SAP users** in Greece, of which **10 are confirmed on S/4HANA** and **3 are confirmed on RISE with SAP Cloud**. An additional 15 companies appear on SAP partner customer lists with limited public detail.

Six companies previously considered as targets were confirmed as non-SAP customers or non-viable.

---

## Tier 1: Confirmed RISE with SAP / S/4HANA Cloud

These companies have publicly documented RISE with SAP or S/4HANA Cloud deployments. They are the most relevant to AITYA's BTP-dependent architecture.

- **HELLENiQ ENERGY (formerly Hellenic Petroleum):** **RISE with SAP S/4HANA Cloud Private Edition + BTP.** ~EUR 12B revenue, largest refiner in Greece (3 refineries, 57% of Greek refining capacity). Uses SAP Build Apps, SAP Ariba, SAP SuccessFactors, SAP Signavio, SAP Build Process Automation. Clean core architecture. Implementation partner: Netcompany-Intrasoft. ([SAP Customer Story, April 2025](https://www.sap.com/asset/dynamic/2025/04/36474d54-027f-0010-bca6-c68f7e60039b.html)) *Confidence: High.*

- **Sarantis Group:** **RISE with SAP S/4HANA Cloud, Private Edition, Premium Plus** -- described as the **first customer in the world** to choose this tier. Athens-based multinational consumer products company (Beauty & Skin Care, Personal Care, Home Care), 60-year history, operations in 13 countries, 8 business units, distribution to 50+ countries. Greenfield implementation including SAP S/4HANA Cloud for contract/lease management, manufacturing planning/scheduling, SAP Multi-Bank Connectivity, SAP Analytics Cloud, SAP IBP, SAP SuccessFactors. Implementation partner: EY Greece (120+ SAP professionals). ([SAP News Center, May 2024](https://news.sap.com/africa/2024/05/sarantis-group-is-accelerating-digital-transformation-with-sap/); [SAP Asset, June 2024](https://www.sap.com/assetdetail/2024/06/64f279b7-c17e-0010-bca6-c68f7e60039b.html); [Sarantis Group press release](https://www.sarantisgroup.com/news/sarantis-group-is-accelerating-digital-transformation-with-sap/)) *Confidence: High.*

- **Piraeus Bank:** **RISE with SAP S/4HANA Private Cloud Edition on Microsoft Azure.** Largest bank in Greece by total assets (~EUR 80B, 22.88% market share). Financial ERP system finalized in 2024 as part of a multi-year digital transformation program launched in 2021 with Accenture and Microsoft. Implementation partner: PwC Greece (strategic implementation advisor). ([PwC Greece Case Study](https://www.pwc.com/gr/en/publications/greek-thought-leadership/the-future-of-finance-in-banking/Strategic-Implementation-Advisor-for-Piraeus-Bank.html)) *Confidence: High.*

---

## Tier 2: Confirmed S/4HANA (On-Premise, Private Cloud, or Azure IaaS)

These companies are confirmed on S/4HANA but not on RISE with SAP's managed cloud. Some use BTP services alongside on-premise or IaaS-hosted cores.

- **Motor Oil Hellas:** **S/4HANA (greenfield) + BTP cloud services** (HANA Cloud, Analytics Cloud for predictive maintenance, SAP EHS). Top European refinery, 185,000 barrels/day, 2,500 employees, exports to 45+ countries. SAP user since 1999. Go-live January 2023. Whether the S/4HANA core is on-premise or RISE could not be determined; no RISE reference found. Implementation partner: Real Consulting (core), Accenture (analytics). ([SAP Customer Story](https://www.sap.com/greece/about/customer-stories/motor-oil-group.html); [SAPinsider Case Study](https://sapinsider.org/casestudies/motor-oil-tests-sap-predictive-maintenance-to-reduce-refinery-downtime/)) *Confidence: High (SAP confirmed); Medium (deployment model ambiguous).*

- **OPAP:** **S/4HANA on Microsoft Azure.** Leading gaming company in Greece, founded 1958, exclusive rights to lotteries, sports betting, VLTs. Modules: FI/CO, HCM, MM, SD. Cloud-hosted on Azure IaaS. Implementation partner: Real Consulting. ([Microsoft Customer Story](https://customers.microsoft.com/EN-US/story/1601505294623738805-opap-gaming-azure-en-greece)) *Confidence: High.*

- **Coca-Cola HBC:** **S/4HANA in managed private cloud** (OTE/Deutsche Telekom data center, Athens). ~EUR 9B revenue, single instance serving 28 countries. Migrated integration middleware from on-premise SAP PI/PO to SAP BTP Integration Suite. Uses SAP Commerce Cloud. Not on RISE -- the core ERP runs in OTE's infrastructure, not SAP's cloud. ([CustomerTimes Case Study](https://www.customertimes.com/case-study/coca-cola-hbc); [SAP Success Story](https://www.sap.com/asset/dynamic/2024/05/e818ce4e-bd7e-0010-bca6-c68f7e60039b.html); [Deutsche Telekom B2B](https://www.b2b-europe.telekom.com/blog/2017/12/13/ote-group-wins-deal-with-coca-cola-hbc-again)) *Confidence: High.*

- **Titan Cement:** **S/4HANA on-premise**, single instance consolidating 15 countries ("1ERP4All" project). Greece rolled out by June 2020. SAP partner since the 1990s via Elsop Consulting Group. No evidence of RISE or cloud migration. ([Original Software Case Study](https://originalsoftware.com/titan-cement-groups-story/)) *Confidence: High.*

- **Frigoglass:** **S/4HANA + SAP IBP (Integrated Business Planning).** Global commercial refrigeration manufacturer. SAP case study (October 2023) documents SAP IBP for demand planning. "ERP Global template with SAP (S4/HANA) in Greece" per employee profiles. Deployment model (on-premise vs. cloud) unknown. Company navigating a corporate transaction as of early 2025. ([SAP IBP Case Study](https://www.sap.com/documents/2023/10/a2901711-947e-0010-bca6-c68f7e60039b.html)) *Confidence: Medium-High (S/4HANA); Low (deployment model).*

- **ADMIE (IPTO):** **S/4HANA, likely on-premise.** Greece's Independent Power Transmission Operator. EUR 6.5M contract (April 2021, 17 months) with INTRASOFT International + OTE consortium. Covers Finance, Procurement, EAM, SuccessFactors, WorkForce Management. Capex procurement structure and absence of any cloud/RISE references suggest on-premise or private hosting. ([IPTO Press Release](https://www.admie.gr/en/kentro-typoy/press-releases/ipto-acquires-state-art-information-system-intrasoft-and-ote); [Planet S.A. Case Study](https://www.planet.gr/case-studies/developing-iptos-erpeam)) *Confidence: Medium.*

- **Plaisio Computers:** **Migrating from SAP ECC 6.0 (on-premise, IS Retail) to S/4HANA.** Leading Greek retailer for office products and consumer electronics. Warehouse operations ran on SAP ECC 6 with IS Retail per [Rocket Consulting case study](https://blog.rocket-consulting.com/case-studies-sap/plaisio-computers-case-study) (2019). S/4HANA migration underway as of 2024-2025. Target deployment model unknown. *Confidence: High (SAP user); Low (cloud status).*

---

## Tier 3: Confirmed SAP User (Version/Deployment Unknown or Legacy ECC)

These companies are confirmed SAP users but the specific ERP version and deployment model could not be determined from public sources.

- **OTE Group / Cosmote (Deutsche Telekom Greece):** **SAP user, transitioning to RISE with SAP** as part of Deutsche Telekom's group-wide migration (300+ systems, [announced March 2024](https://news.sap.com/2024/03/deutsche-telekom-rise-with-sap-cloud-transformation/)). Largest telecom in Greece, 55% owned by Deutsche Telekom. Whether OTE Greece is in the current migration wave or a subsequent one is not publicly confirmed. SAP ERP usage confirmed via COSMOTE employee profiles. *Confidence: Medium (SAP confirmed; RISE timing for Greek subsidiary uncertain).*

- **Viohalco Group:** **SAP user (group-wide).** EUR 6.6B revenue (2024), leading metal processing companies in Europe. In-house subsidiary TEKA Systems is one of Greece's largest SAP Gold Partners (200+ SAP specialists, 25,000+ SAP users across 23 countries, SAP PCoE certified). Recently deployed SAP Signavio for process mining across lead-to-cash and procure-to-pay processes. ([Viohalco SAP Signavio Case Study, 2025](https://www.sap.com/greece/asset/dynamic/2025/11/447767ee-2d7f-0010-bca6-c68f7e60039b.html); [Viohalco: TEKA Systems](https://www.viohalco.com/787/en/Teka-Systems-SA--Engineering-projects-and-ERP-systems-/)) *Confidence: High (SAP usage); Low (version/deployment).*

- **ElvalHalcor (Viohalco subsidiary):** **SAP user via Viohalco/TEKA Systems.** Major European metals processor (aluminum, copper). TEKA has implemented SAP Invoice Management by OpenText at Halcor (now ElvalHalcor) and its financial service center Metalign. ([TEKA Systems Project](https://tekasystems.com/projects/opentext-teka/)) *Confidence: Medium (SAP usage); Low (version/deployment).*

- **National Bank of Greece:** **SAP user confirmed.** One of the four systemic Greek banks, founded 1841. Uses SAP Bank Analyzer, SAP CRM, SAP ERP for HR, finance, treasury, asset management, real estate, and materials. Has a dedicated "Deputy Director of SAP Banking Accounting Appl & ERP Systems" role. Also offers NBG Connector for corporate SAP-to-bank integration. ([Finextra](https://www.finextra.com/newsarticle/2294/national-bank-of-greece-automates-hr-function-with-sap); [The Org](https://theorg.com/org/national-bank-of-greece-sa/org-chart/ioanna-lazaridou)) *Confidence: High (SAP usage); Low (version -- likely ECC or Suite on HANA).*

- **Eurobank:** **SAP SuccessFactors Time Tracking confirmed.** 4th largest Greek bank (~EUR 63B total assets). Broader SAP usage likely given Eurobank FPS (now doValue) was an ELSOP customer. ([SAP SuccessFactors Case Study](https://www.sap.com/greece/documents/2023/09/a857880f-8f7e-0010-bca6-c68f7e60039b.html)) *Confidence: Medium (SuccessFactors confirmed; broader ERP unknown).*

- **Mytilineos / Metlen Energy & Metals:** **SAP user, version and deployment unknown.** One of Greece's largest industrial companies, EUR 6B+ revenue, 2,700+ employees. Evidence from personnel history (Business Innovation Director was formerly SAP FI/CO Consultant at Aluminium of Greece, a subsidiary) and "SAP Analyst" job postings. Major digital transformation underway (45+ digital products, EUR 30-50M annual digital spend). ([METLEN Transformation](https://www.metlengroup.com/news/company-news/metlen-transformation-the-third-era-progress-in-motion/)) *Confidence: Low-Medium.*

- **TERNA ENERGY (GEK TERNA Group):** **SAP user, version and deployment unknown.** 6,729 employees, leading Greek energy and infrastructure group. Confirmed via SAP FI/CO consultant job postings. Acquired by Masdar (Abu Dhabi) in April 2025, which may trigger ERP changes. *Confidence: Low.*

- **Lamda Development:** **SAP HANA user, edition unclear.** EUR 3.48B property portfolio (2024), leader in Greek retail real estate, developing The Ellinikon project. SAP HANA referenced as a core enterprise platform. May be Suite on HANA or S/4HANA. No evidence of RISE or BTP. Appears on ELSOP customer list. *Confidence: Medium (SAP usage); Low (version/deployment).*

- **Elgeka Group:** **SAP user confirmed.** Leading Greek commercial company in food and consumer goods distribution, operations in Greece, Romania, Cyprus. Company website states it "utilizes comprehensive tools like SAP, along with proprietary, custom solutions." ([Elgeka: Our Services](https://www.elgeka.gr/en/our-services/)) *Confidence: Medium (SAP confirmed; version unknown).*

- **Chipita (now Mondelez subsidiary):** **SAP ECC confirmed.** ~USD 580M revenue (2020), acquired by Mondelez for EUR 2B (2022). ELSOP implemented SAP Fiori apps for purchasing workflow. Post-acquisition, SNP performed data extraction from Chipita's SAP source system for Mondelez integration. ([ELSOP Chipita SAP Fiori](https://www.elsop.gr/en/sap-mobililty/mobility-success-stories/267-chipita-sap-fiori); [SNP/Mondelez Case Study](https://www.snpgroup.com/en/resources/customer-stories/mondelez/)) *Confidence: High (SAP ECC confirmed). Note: now part of Mondelez, may migrate to Mondelez SAP landscape.*

---

## Tier 4: SAP Users (Partner Lists / Thin Evidence)

These companies appear on the [ELSOP customer list](https://www.elsop.gr/customers/our-customers) (a major Greek SAP partner with 80+ SAP projects) or were identified at SAP events. Detailed implementation information is not publicly available.

| Company | Sector | Evidence | Notes |
|---|---|---|---|
| **EKO (HELLENiQ subsidiary)** | Energy / Fuel Distribution | ELSOP implemented SD Credit Release app on SAP Mobile Platform | Part of HELLENiQ ENERGY group; already counted under Tier 1 parent |
| **DEPA (Natural Gas)** | Energy / Gas | ELSOP customer list; field service pilot app | Greek natural gas supplier |
| **Hellas Gold (Eldorado Gold)** | Mining | Presented at SAP Leading2Success Athens 2023 in RISE session | Active engagement with SAP cloud migration |
| **EL PACK Group** | Packaging / Manufacturing | Presented at SAP Leading2Success Athens 2023 in RISE session | Largest paper packaging in Greece |
| **Papadopoulos Biscuits** | Food / FMCG | ELSOP customer list; Mobile Basis app implemented | Greece's largest biscuit manufacturer |
| **FAGE Dairy** | Food / Dairy | SAP Application Specialist at FAGE USA confirmed via LinkedIn | Greek-founded, global dairy company |
| **Creta Farms** | Food / Meat Processing | ELSOP developed Rebate Management in SAP ERP | Largest pork producer in Greece |
| **Athenian Brewery (Heineken)** | Beverages | ELSOP customer list | Largest brewer in Greece, Heineken subsidiary |
| **AB Vassilopoulos (Ahold Delhaize)** | Retail / Supermarkets | ELSOP customer list; employees confirm B2B orders via SAP | 567 stores, 14,000+ employees |
| **Karelia Tobacco** | Tobacco | ELSOP customer list | Largest Greek tobacco manufacturer |
| **Ellaktor** | Construction / Infrastructure | ELSOP customer list | Largest infrastructure group in Greece |
| **Mevgal** | Food / Dairy | ELSOP customer list | 3rd largest dairy in Greece |
| **Minoan Lines** | Shipping / Ferry | ELSOP customer list | Major Greek ferry company |
| **Unilever Greece** | FMCG | ELSOP customer list | Multinational, Greek operations |
| **Sanofi Greece** | Pharma | ELSOP customer list | Multinational, Greek operations |

---

## Not SAP Customers / Non-Viable

| Company | Status | Evidence |
|---|---|---|
| ~~Public Power Corporation (PPC/DEI)~~ | Not confirmed as SAP customer | No public evidence found despite extensive searching. May use Oracle or another ERP. |
| ~~Sklavenitis~~ | Not SAP | Uses Netvolution 5.7 (ATCOM). ([ATCOM](https://www.atcom.gr/productions/recent-productions/sklavenitis/)) |
| ~~Aegean Airlines~~ | Not SAP | Uses Oracle Unity CDP, Oracle CX Cloud, SingularLogic. ([Oracle](https://www.oracle.com/customers/aegean/); [SingularLogic](https://portal.singularlogic.eu/customer/842/aegean-airlines)) |
| ~~Hellenic Post (ELTA)~~ | Not SAP | Uses Oracle Financials. |
| ~~Pavlidis Marbles~~ | Not SAP | Uses Entersoft Business Suite and Entersoft CRM. ([Entersoft](https://www.entersoft.eu/clients/pavlidis-2/)) |
| ~~Folli Follie Group~~ | Defunct | Collapsed after ~USD 1B accounting scandal. Delisted, criminal proceedings, 2021 creditor restructuring. |
| ~~Autohellas (Hertz Greece)~~ | Unconfirmed | No press releases, case studies, or partner references found. Uses SQL + Targit/Power BI for analytics. |

---

## Summary: Verified SAP Status

| Tier | Description | Companies | Count |
|---|---|---|---|
| **Tier 1: RISE with SAP Cloud** | Confirmed RISE / S/4HANA Cloud | HELLENiQ ENERGY, Sarantis Group, Piraeus Bank | **3** |
| **Tier 2: S/4HANA confirmed** | On-premise, private cloud, or Azure IaaS | Motor Oil, OPAP, Coca-Cola HBC, Titan Cement, Frigoglass, ADMIE/IPTO, Plaisio (migrating) | **7** |
| **Tier 3: SAP user, version unclear** | Confirmed SAP but ERP version/deployment unknown | OTE/Cosmote, Viohalco, ElvalHalcor, National Bank of Greece, Eurobank, Mytilineos, TERNA Energy, Lamda Development, Elgeka, Chipita | **10** |
| **Tier 4: Partner list / thin evidence** | ELSOP customer list or SAP event attendance | DEPA, Hellas Gold, EL PACK, Papadopoulos, FAGE, Creta Farms, Athenian Brewery, AB Vassilopoulos, Karelia, Ellaktor, Mevgal, Minoan Lines, Unilever GR, Sanofi GR | **14** |
| **Not SAP / Non-viable** | Confirmed non-SAP or defunct | PPC, Sklavenitis, Aegean Airlines, ELTA, Pavlidis Marbles, Folli Follie, Autohellas | **7** |

**Totals:** 34 confirmed or likely SAP users (Tiers 1-4). 10 confirmed on S/4HANA (Tiers 1-2). 3 confirmed on RISE with SAP Cloud (Tier 1).

*Note: EKO is excluded from the Tier 4 count as a subsidiary of HELLENiQ ENERGY (already counted in Tier 1).*

---

## Strategic Implications

### The Picture Is Better Than Previously Documented -- But Still Constrained

The previous version of this document identified only 1 confirmed RISE with SAP customer (HELLENiQ ENERGY) and concluded "cloud readiness is near-zero." Updated research corrects this:

- **3 confirmed RISE with SAP customers** (HELLENiQ ENERGY, Sarantis Group, Piraeus Bank), spanning energy, consumer products, and banking
- **2 additional companies** (Hellas Gold, EL PACK) presented at SAP's RISE session at Leading2Success Athens 2023, signaling active cloud interest
- **OTE Group** is part of Deutsche Telekom's group-wide RISE migration, though Greek timing is uncertain
- **10 total confirmed S/4HANA** companies across Tiers 1-2

### Impact on AITYA's Thesis

1. **Cloud readiness is no longer near-zero, but it's still thin.** Three RISE customers across three different industries (energy, FMCG, banking) is better than one, but it's not enough to build industry-homogeneous federated cohorts within Greece. The federated architecture requires multiple companies in the same industry running on BTP -- currently the maximum is 1 per sector.

2. **The banking sector is now actionable.** Piraeus Bank (RISE, finalized 2024) and National Bank of Greece (SAP Bank Analyzer, HR, Finance) are confirmed SAP users. With Eurobank on SuccessFactors and the 2027 ECC end-of-life deadline driving migrations, Greek banking could become the first sector where 2-3 RISE customers coexist. However, banks use SAP primarily for back-office functions (HR, fixed assets, treasury) rather than core lending/transaction systems, which limits the financial analytics surface.

3. **Sarantis Group is a flagship prospect.** As the first worldwide customer for RISE Premium Plus, Sarantis has deep BTP adoption (SAP IBP, Analytics Cloud, Multi-Bank Connectivity). Their 13-country footprint and consumer goods focus make them relevant to AITYA's collections intelligence pillar. The implementation partner (EY Greece) could also be a channel partner.

4. **Federated cohort density remains the binding constraint.** Even with 34 SAP users, only 3 are on RISE, and they span different industries. Building the 15+ tenant cohorts required for meaningful federated benchmarking is impossible in Greece alone. The [zero_copy_viability.md](zero_copy_viability.md) analysis remains correct: Greece is a pilot market, not a scale market.

5. **The single-tenant path has a larger addressable base than previously estimated.** The [Option 1 path in zero_copy_viability.md](zero_copy_viability.md) ("Zero-Integration AR Analytics") could target 10 confirmed S/4HANA companies, not 6-7 as previously stated. Adding Tier 3 companies that will likely migrate to S/4HANA before the 2027 ECC deadline expands the medium-term pipeline.

6. **The Viohalco group remains a double-edged sword.** Viohalco's in-house SAP Gold Partner (TEKA Systems, 200+ specialists, now deploying SAP Signavio for process mining) means deep SAP expertise -- but also means they are likely to build analytics internally rather than buy from a startup. The SAP Signavio deployment for lead-to-cash and procure-to-pay process mining specifically overlaps with AITYA's analytics scope.

### What Needs to Happen

- The market sizing in [`pragmatic.md`](pragmatic.md) should be revised to reflect the actual addressable base: ~10 confirmed S/4HANA companies (not 20), with 3 on RISE.
- Sarantis Group and Piraeus Bank should be evaluated as early pilot candidates alongside HELLENiQ ENERGY.
- A pan-European scaling strategy must account for the fact that Greece provides pilot diversity (3 cloud customers across 3 industries) but not cohort density within any single industry.
- The 2027 SAP ECC end-of-life deadline will force Tier 3 and Tier 4 companies to migrate. Monitoring their RISE/cloud adoption decisions is essential for pipeline development.

---

## Sources

### Tier 1 (RISE with SAP Cloud)
- [HELLENiQ ENERGY SAP Customer Story (April 2025)](https://www.sap.com/asset/dynamic/2025/04/36474d54-027f-0010-bca6-c68f7e60039b.html)
- [Sarantis Group SAP Announcement (May 2024)](https://news.sap.com/africa/2024/05/sarantis-group-is-accelerating-digital-transformation-with-sap/)
- [Sarantis Group SAP Asset (June 2024)](https://www.sap.com/assetdetail/2024/06/64f279b7-c17e-0010-bca6-c68f7e60039b.html)
- [Sarantis Group Press Release](https://www.sarantisgroup.com/news/sarantis-group-is-accelerating-digital-transformation-with-sap/)
- [Piraeus Bank PwC Case Study](https://www.pwc.com/gr/en/publications/greek-thought-leadership/the-future-of-finance-in-banking/Strategic-Implementation-Advisor-for-Piraeus-Bank.html)

### Tier 2 (S/4HANA Confirmed)
- [Motor Oil SAP Customer Story](https://www.sap.com/greece/about/customer-stories/motor-oil-group.html)
- [Motor Oil SAPinsider Case Study](https://sapinsider.org/casestudies/motor-oil-tests-sap-predictive-maintenance-to-reduce-refinery-downtime/)
- [Motor Oil S/4HANA Go-Live (WorldEnergyNews.gr)](https://www.worldenergynews.gr/energeia/articles/541064/enarksi-paragogikis-leitourgias-systimatos-s-4hana-tou-omilou-motor-oil)
- [OPAP Microsoft Customer Story](https://customers.microsoft.com/EN-US/story/1601505294623738805-opap-gaming-azure-en-greece)
- [Coca-Cola HBC SAP BTP Migration (CustomerTimes)](https://www.customertimes.com/case-study/coca-cola-hbc)
- [Coca-Cola HBC SAP Success Story](https://www.sap.com/asset/dynamic/2024/05/e818ce4e-bd7e-0010-bca6-c68f7e60039b.html)
- [OTE Group Wins Deal with Coca-Cola HBC (Deutsche Telekom B2B)](https://www.b2b-europe.telekom.com/blog/2017/12/13/ote-group-wins-deal-with-coca-cola-hbc-again)
- [Titan Cement S/4HANA Consolidation (Original Software)](https://originalsoftware.com/titan-cement-groups-story/)
- [Elsop Consulting Group History](https://www.elsop.gr/elsop-company/history)
- [Frigoglass SAP IBP Case Study (October 2023)](https://www.sap.com/documents/2023/10/a2901711-947e-0010-bca6-c68f7e60039b.html)
- [ADMIE/IPTO Press Release](https://www.admie.gr/en/kentro-typoy/press-releases/ipto-acquires-state-art-information-system-intrasoft-and-ote)
- [Planet S.A. Case Study: IPTO ERP/EAM](https://www.planet.gr/case-studies/developing-iptos-erpeam)
- [Plaisio Computers Case Study (Rocket Consulting)](https://blog.rocket-consulting.com/case-studies-sap/plaisio-computers-case-study)

### Tier 3 (SAP User, Version Unclear)
- [Deutsche Telekom Chooses RISE with SAP (March 2024)](https://news.sap.com/2024/03/deutsche-telekom-rise-with-sap-cloud-transformation/)
- [Viohalco SAP Signavio Case Study (2025)](https://www.sap.com/greece/asset/dynamic/2025/11/447767ee-2d7f-0010-bca6-c68f7e60039b.html)
- [Viohalco: TEKA Systems](https://www.viohalco.com/787/en/Teka-Systems-SA--Engineering-projects-and-ERP-systems-/)
- [TEKA Systems SAP Invoice Management at Halcor](https://tekasystems.com/projects/opentext-teka/)
- [National Bank of Greece SAP HR (Finextra)](https://www.finextra.com/newsarticle/2294/national-bank-of-greece-automates-hr-function-with-sap)
- [Eurobank SAP SuccessFactors Case Study](https://www.sap.com/greece/documents/2023/09/a857880f-8f7e-0010-bca6-c68f7e60039b.html)
- [METLEN Transformation Announcement](https://www.metlengroup.com/news/company-news/metlen-transformation-the-third-era-progress-in-motion/)
- [Elgeka: Our Services](https://www.elgeka.gr/en/our-services/)
- [ELSOP Chipita SAP Fiori Case Study](https://www.elsop.gr/en/sap-mobililty/mobility-success-stories/267-chipita-sap-fiori)
- [SNP/Mondelez Chipita Case Study](https://www.snpgroup.com/en/resources/customer-stories/mondelez/)

### Tier 4 (Partner Lists / Events)
- [ELSOP Customer List](https://www.elsop.gr/customers/our-customers)
- [SAP Leading2Success Athens 2023](https://news.sap.com/greece/2023/11/sap-leading2success-athens-2023/)

### Not SAP / Non-Viable
- [Sklavenitis (ATCOM)](https://www.atcom.gr/productions/recent-productions/sklavenitis/)
- [Pavlidis Marbles (Entersoft)](https://www.entersoft.eu/clients/pavlidis-2/)
- [Aegean Airlines (Oracle)](https://www.oracle.com/customers/aegean/)
- [Aegean Airlines (SingularLogic)](https://portal.singularlogic.eu/customer/842/aegean-airlines)
- [ELTA Digital Transformation (BusinessNews.gr)](https://www.businessnews.gr/epixeiriseis/item/255548-pos-synexizetai-o-psifiakos-metasximatismos-ton-elta-to-2023)
- [Folli Follie Restructuring (Global Restructuring Review)](https://globalrestructuringreview.com/article/folli-follie-agrees-restructuring-terms-ad-hoc-creditors-group)

### Additional SAP Ecosystem References
- [KPMG Greece Joins SAP PartnerEdge (April 2024)](https://kpmg.com/gr/en/home/media/press-releases/2024/04/kpmg-greece-joins-sap-partneredge-program.html)
- [Real Consulting SAP Partner](https://www.realconsulting.gr/partners/sap)
- [TEKA Systems SAP Solutions](https://tekasystems.com/sap-solutions-implementation/)
- [EY SAP Greece](https://www.ey.com/en_gr/alliances/sap/accelerate)
