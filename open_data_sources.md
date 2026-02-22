# Open Data Sources for AITYA

This document catalogs publicly available datasets relevant to AITYA's [Cash & Collections Intelligence](pragmatic.md) analytics: predictive collection scoring, working-capital benchmarking, dunning optimization, and counterparty risk assessment. Each source is evaluated for bulk collection feasibility, licensing, and mapping to AITYA's analytics needs.

The critical gap across all public data: **no multi-company, multi-tenant invoice-level payment dataset with counterparty identifiers exists.** This is precisely the data AITYA's federated architecture would generate -- which is both the moat and the chicken-and-egg problem.

---

## 1. Invoice-Level Payment Prediction Data

The scarcest and most directly relevant category. Only a handful of public datasets exist, all synthetic or anonymized from single companies.

### 1.1 IBM Late Payment Histories (Kaggle)

Invoice-level accounts receivable dataset with payment delays, customer identifiers, and invoice amounts. The most commonly used dataset for AR payment prediction research.

- **Collection:** `kaggle datasets download -d hhenry/finance-factoring-ibm-late-payment-histories`
- **Format:** CSV
- **License:** Kaggle standard (open for research)
- **AITYA mapping:** Closest public match to AITYA's predictive collection scoring. Useful for prototyping invoice payment prediction models before deploying on SAP data. Limitation: single anonymous company, unlikely to represent Greek B2B payment behavior diversity.

### 1.2 Payment Date Prediction for Invoices (Kaggle)

Invoice-level dataset for predicting payment dates on open invoices, with due dates and actual payment dates.

- **Collection:** `kaggle datasets download -d pradumn203/payment-date-prediction-for-invoices-dataset`
- **Format:** CSV
- **License:** Kaggle standard
- **AITYA mapping:** Complements the IBM dataset with a different data distribution for payment prediction prototyping.

### 1.3 Home Credit Default Risk (Kaggle Competition)

Multi-table dataset: 307,511 loan applications with default labels, plus bureau data, previous loans, credit card balances, installment payments, and POS cash balances (7 relational tables).

- **Collection:** `kaggle competitions download -c home-credit-default-risk`
- **Format:** CSV (7 tables)
- **License:** Competition data (available for research)
- **AITYA mapping:** The multi-table relational structure mirrors the complexity of SAP data. Feature engineering patterns (aggregating payment history, bureau inquiries, balance trends) are directly transferable to AITYA's feature engineering on CDS views. The installment-level payment history is analogous to invoice-level payment tracking.

### 1.4 Italian SME Trade Credit Dataset (Academic)

Panel data of invoice-level information on 534 Italian SMEs from Q1 2015 to Q2 2017, from the paper "Can we trust machine learning to predict the credit risk of small businesses?" (Review of Quantitative Finance and Accounting, 2024).

- **Collection:** Contact authors; dataset availability depends on journal data-sharing policy
- **Paper:** https://link.springer.com/article/10.1007/s11156-024-01278-0
- **Format:** Panel data (likely CSV/Stata)
- **License:** Academic
- **AITYA mapping:** Closest match to AITYA's use case: invoice-level payment data for European SMEs with credit risk labels. Italian and Greek SME payment cultures share structural similarities. If obtainable, an excellent validation set.

### 1.5 Additional Kaggle Datasets

- **Invoices Dataset** (cankatsrc): https://www.kaggle.com/datasets/cankatsrc/invoices
- **Payment Date Dataset** (rajattomar132): https://www.kaggle.com/datasets/rajattomar132/payment-date-dataset
- **Credit Score Classification**: https://www.kaggle.com/datasets/parisrohan/credit-score-classification
- **Credit Risk Dataset**: https://www.kaggle.com/datasets/laotse/credit-risk-dataset

---

## 2. Greek-Specific Sources

### 2.1 Diavgeia (Greek Government Transparency Portal)

Every act and decision by Greek government institutions since 2010, digitally signed. Includes public spending decisions, procurement contracts, and payment orders. ~2M+ payment decisions valued at EUR 44.5B+.

- **API:** https://mef.diavgeia.gov.gr/pages/API (XML-based, no auth required)
- **Python parser:** https://github.com/Stavros/diavgeia
- **Format:** XML
- **License:** CC-BY
- **AITYA mapping:** Uniquely valuable. Cross-referencing with SAP customer data reveals: (a) which counterparties are government suppliers, (b) actual government payment timelines, (c) contract values as revenue proxies. Directly feeds counterparty risk assessment and dunning optimization -- if a customer is waiting on a government payment, aggressive dunning won't help. Also feeds the [counterparty graph strategy](network_effects.md).

### 2.2 publicspending.gr (Greek Government Spending Linked Data)

Linked Open Data representation of Diavgeia spending data. ~2M payment decisions, EUR 44.5B, 65M RDF triples with entity linking and network analysis.

- **Collection:** SPARQL endpoint for structured queries
- **Format:** RDF (via SPARQL), CSV
- **License:** CC-BY (based on Diavgeia)
- **Reference:** https://www.w3.org/2012/06/pmod/pmod2012_submission_32.pdf
- **AITYA mapping:** Same data as Diavgeia but with SPARQL queryability and entity linking. The payment relationship network between government entities and companies could directly feed the counterparty graph.

### 2.3 Bank of Greece Open Data

Greek lending rates to corporations, credit aggregates by sector, payment system statistics.

- **Portal:** https://opendata.bankofgreece.gr/
- **Also via ECB API:** https://data.ecb.europa.eu/data/geographical-areas/greece
- **Format:** CSV/Excel; also queryable via ECB REST API (no auth)
- **License:** Public central bank data
- **AITYA mapping:** Greek-specific macro features for payment prediction models. If lending to Greek SMEs tightens, expect slower payment behavior. The most relevant macro signal source for the Greek market.

### 2.4 ATHEX (Athens Stock Exchange) ESEF Filings

Annual financial reports for Greek listed companies in machine-readable iXBRL format (mandatory since FY2021). Covers Greek listed SAP users.

- **Via filings.xbrl.org** (bulk) or https://www.athexgroup.gr/en/market-data/financial-data
- **Format:** iXBRL/XHTML, xBRL-JSON
- **License:** Public regulatory filings
- **AITYA mapping:** Most directly relevant source for Greek listed companies' financial health. Extractable working capital metrics (receivables, revenue, payables) for the Greek market.

### 2.5 GEMI (Greek General Commercial Registry)

Registration data for all Greek businesses: company name, legal form, TIN (AFM), date of incorporation, status, capital, activities, management, financial data.

- **API:** Accepts queries by GEMI number, TIN, or business name
- **Access restriction:** API limited to public bodies and financial institutions
- **Workaround:** OpenCorporates aggregates some GEMI data (see 3.3)
- **Reference:** https://www.gov.gr/en/upourgeia/upourgeio-anaptuxes/anaptuxes/stoikheia-demosiotetas-emporikon-epikheireseon-eggegrammenon-sto-geme
- **AITYA mapping:** Critical for the Greek market -- financial statements of the 34 target SAP users and their counterparties. The access restriction is a significant barrier; may require partnering with a Greek financial institution.

### 2.6 data.gov.gr (Greek Open Data Portal)

10,500+ datasets across 17 topics: taxation data, commerce data (VAT numbers, company names, legal status), economic indicators.

- **Portal:** https://data.gov.gr/
- **API:** JSON-based with filtering/sorting
- **Format:** JSON, CSV, XLSX
- **License:** Greek government open data
- **AITYA mapping:** Ancillary enrichment for counterparty profiles. Tax registry data can verify company status; economic indicators provide local context features.

### 2.7 ELSTAT (Hellenic Statistical Authority)

Official Greek statistics: business structure (enterprises by size and NACE sector), industrial production, trade, employment, GDP, consumer/producer prices.

- **Portal:** https://www.statistics.gr/en/home/
- **Format:** Excel, PDF, CSV
- **License:** Public government statistics
- **AITYA mapping:** Calibrates counterparty risk priors. Understanding how many Greek enterprises exist by sector and size class informs what "normal" looks like for different counterparty segments.

### 2.8 ESIDIS/KIMDIS (Greek Electronic Public Procurement)

All Greek public procurement above EUR 1,000 (KIMDIS registry) and electronic tenders above EUR 30,000 (ESIDIS/Promitheus platform).

- **Portal:** http://www.promitheus.gov.gr
- **Some data on:** data.gov.gr
- **License:** Public procurement data
- **AITYA mapping:** Complementary to Diavgeia for identifying counterparties involved in Greek public procurement -- a proxy for creditworthiness and revenue stability.

---

## 3. European Company Financials

### 3.1 EU ESEF Filings (filings.xbrl.org)

Annual financial reports of all EU-listed companies in iXBRL format since 2021. Consolidated IFRS statements with machine-readable tagging of primary financial statement items (receivables, revenue, cash, payables). 4,000+ filings across EU jurisdictions.

- **Bulk repository:** https://filings.xbrl.org
- **ESMA tools:** Python extraction tools on ESMA's GitHub
- **Format:** iXBRL (XHTML with XBRL tags), xBRL-JSON
- **License:** Public filings
- **Note:** European Single Access Point (ESAP), launching July 2027, will centralize all this data via a unified API managed by ESMA.
- **AITYA mapping:** Structured European company financials with cross-country comparability via IFRS taxonomy. Extract receivables, revenue, and working capital components to build European DSO benchmarks by sector.

### 3.2 UK Companies House

Company registration and annual accounts for all ~5M active UK limited companies. Accounts include balance sheets; larger companies file P&L statements.

- **Bulk download (company data):** https://download.companieshouse.gov.uk/en_output.html (monthly)
- **Bulk download (accounts):** https://download.companieshouse.gov.uk/en_accountsdata.html (daily/monthly ZIPs)
- **REST API:** https://developer.company-information.service.gov.uk/ (free, API key via instant registration)
- **Streaming API:** Real-time changes feed
- **Format:** CSV (bulk), iXBRL/XHTML (accounts), JSON (API)
- **License:** Open Government Licence (free commercial use with attribution)
- **AITYA mapping:** Largest freely available source of European company balance sheets. Accounts receivable and payables data can feed DSO/DPO estimation models at scale. SIC codes enable sector benchmarking.

### 3.3 OpenCorporates

World's largest open company database: 200M+ companies across 170+ jurisdictions, aggregated from primary registries. Covers Greece via GEMI integration. Basic company existence data, directors, filings.

- **API:** https://api.opencorporates.com/ (requires API key)
- **Free tier:** Available for journalists, NGOs, academics
- **Commercial tier:** From GBP 2,250/year
- **Format:** JSON
- **License:** Share-alike attribution (open); commercial license for business use
- **AITYA mapping:** Counterparty entity resolution -- look up legal entity information for business partners found in SAP AR data. Does not include financial statements or payment behavior.

### 3.4 BRIS (Business Registers Interconnection System)

Harmonized company existence data across all 27 EU member states: name, legal form, registered seat, registration number.

- **Web interface:** https://e-justice.europa.eu (search portal, no bulk API)
- **License:** Free basic data
- **AITYA mapping:** Limited -- existence verification only, not financials. Useful for confirming counterparty legal status across EU borders.

---

## 4. US Company Financials

### 4.1 SEC EDGAR

Financial statements (10-K, 10-Q) for all US-listed companies in structured XBRL. 18M+ filings. Accounts receivable, revenue, total assets -- raw inputs for DSO, cash conversion cycle, and working capital ratios.

- **Bulk download:** https://www.sec.gov/Archives/edgar/daily-index/xbrl/companyfacts.zip (nightly recompilation)
- **REST API:** `https://data.sec.gov/api/xbrl/companyfacts/CIK{number}.json` (no auth, 10 req/sec)
- **Pre-extracted numerics:** https://www.sec.gov/dera/data/financial-statement-data-sets (quarterly ZIP/CSV)
- **Python:** `edgartools` (`pip install edgartools`)
- **Format:** JSON (API), ZIP/CSV (bulk), XBRL/XML (raw)
- **License:** Public domain
- **AITYA mapping:** Largest structured financial dataset available. Compute DSO, DPO, CCC benchmarks by SIC industry. Useful for pre-training working capital models at scale. Limitation: US companies, not directly representative of Greek SMEs.

---

## 5. Payment Behavior Benchmarks

Aggregate reports providing country-level and sector-level payment behavior statistics. No transaction-level data; useful for calibration and cross-tenant benchmarking targets.

### 5.1 EU Payment Observatory

B2B and G2B payment behavior across all EU member states. Annual reports with payment duration statistics, late payment rates, and trends by country. Established 2023 under the EU Late Payment Directive (2011/7/EU).

- **Reports:** https://single-market-economy.ec.europa.eu/smes/challenges-and-resilience/late-payment/eu-payment-observatory_en
- **2024 annual report (PDF):** https://cdn.ceps.eu/wp-content/uploads/2024/12/EU-Payment-Observatory_Annual-Report-2024_EA-01-24-061-EN-C.pdf
- **Format:** PDF with embedded tables
- **License:** EU publication, freely accessible
- **AITYA mapping:** Country-level payment behavior benchmarks as macro features for prediction models (e.g., "average B2B payment delay in Greece is X days"). Contains Greece-specific data.

### 5.2 Intrum European Payment Report

Annual survey of ~9,150 executives across 25 European countries. Payment terms offered, actual payment duration, late payment rates, bad debt write-offs, time spent chasing payments. Country and sector breakdowns.

- **Download:** https://www.intrum.com/insights/publications/epr-2025/ (registration may be required)
- **Format:** PDF
- **License:** Free download
- **AITYA mapping:** Sector-level payment terms and late payment rates for Greece. Benchmark data for "what normal looks like." Limitation: survey-based, subject to self-reporting bias.

### 5.3 Atradius Payment Practices Barometer

Annual survey of ~6,500 companies across 31 countries. B2B payment terms, overdue invoices, bad debt rates, payment delay causes. Western Europe coverage includes Greece.

- **Download:** https://group.atradius.com/multinationals/payment-practices-trends
- **Format:** PDF
- **License:** Free download
- **AITYA mapping:** Second independent source of European payment behavior benchmarks. Key data points: average Western European B2B payment term ~52 days, 47% of B2B invoices overdue, ~6% bad debt rate. Useful as calibration targets for AITYA's prediction models.

### 5.4 Hackett Group Working Capital Survey

Annual analysis of DSO, DPO, DIO, and cash conversion cycle for the top 1,000 US and European non-financial public companies. Industry-by-industry rankings.

- **Download:** https://www.thehackettgroup.com/insights/2025-working-capital-survey-2508/ (free with registration)
- **Format:** PDF with data tables
- **License:** Free with registration
- **AITYA mapping:** The calibration target for AITYA's cross-tenant benchmarking: "Your DSO of 65 days is 15 days worse than the industry median of 50 days." Separate US and European surveys.

### 5.5 Dun & Bradstreet AR/DSO Industry Reports

Quarterly reports covering 229 industries by SIC code: percentage of companies current on payments vs. 30/60/90 days late. PAYDEX scores (proprietary payment performance index).

- **Sample:** https://www.dnb.co.uk/content/dam/english/dnb-data-insight/AR_and_DSO_Industry_Report_Q1_2020.pdf
- **Format:** PDF
- **License:** Free for published reports; API is commercial
- **AITYA mapping:** Industry-level payment timeliness data comparable to what AITYA aims to produce via federated learning. Validation target: if AITYA's cross-tenant benchmark for Greek manufacturing DSO deviates significantly from D&B's equivalent, something is wrong.

---

## 6. Credit Risk & Country Risk

### 6.1 Coface Country & Sector Risk

Risk ratings for 160+ countries and 13 sectors, updated quarterly. Insolvency forecasts, macroeconomic analysis, trade credit risk. Ratings on a 1-4 scale.

- **Dashboard:** https://www.coface.us/news-economy-and-business-insights/economic-risk-dashboard
- **Format:** PDF (handbook), web-based (dashboard)
- **License:** Freely accessible
- **AITYA mapping:** Sector risk ratings as features in counterparty risk models. Quarterly updates provide timely macro-risk signals.

### 6.2 Allianz Trade Insolvency Data

Country risk ratings (1-4 scale), global insolvency reports with forecasts by country, collection complexity assessments. Forecasts: global insolvencies +6% in 2025, +3% in 2026.

- **Country risk:** https://www.allianz.com/en/economic_research/country-and-sector-risk/country-risk.html
- **Insolvency reports:** https://www.allianz-trade.com/en_global/news-insights/news/insolvency-report-2025.html
- **Format:** PDF, web-based maps
- **License:** Freely accessible
- **AITYA mapping:** Insolvency forecasts by country are a direct input to counterparty risk assessment. If Greek insolvencies are forecast to rise 10%, that should tighten collection scoring thresholds. Collection complexity assessments inform dunning strategy by jurisdiction.

### 6.3 EU Insolvency Registers

Bankruptcy and insolvency records across EU member states, cross-linked since 2014.

- **Search:** https://e-justice.europa.eu/topics/registers-business-insolvency-land/bankruptcy-insolvency-registers-search-insolvent-debtors-eu_en
- **Limitation:** Web-based search only, no bulk API
- **License:** Free access, varies by country
- **AITYA mapping:** Lagging but definitive signal: "has this counterparty ever been insolvent." Limited by lack of bulk API -- would require periodic scraping or manual matching.

### 6.4 ICAP CRIF (Greek Business Credit Data) -- Commercial

Largest business database in southeastern Europe: 8.7M companies across Greece, Bulgaria, Romania, Cyprus. 900,000+ incorporated company financial statements. ESMA-certified credit ratings, payment history analysis.

- **Website:** https://icapcrif.com/en/
- **Access:** Commercial only, no public API
- **AITYA mapping:** The single most relevant data source for Greek counterparty risk, but not open or free. Worth exploring as a data partnership or validation source. Also a competitive threat -- ICAP already sells analytics in AITYA's target space.

---

## 7. Macroeconomic Data (API Access)

All sources below provide programmatic API access suitable for automated feature pipelines.

| Source | Coverage | API Endpoint | Auth | Python Package |
|---|---|---|---|---|
| **ECB Data Portal** | Euro area rates, inflation, GDP, Greek series | `https://data-api.ecb.europa.eu/` | None | `ecbdata` |
| **Eurostat** | EU business demography, enterprise survival by NACE/country | SDMX 3.0 via `ec.europa.eu/eurostat` | None | `eurostat`, `pandaSDMX` |
| **FRED** | 800K+ US and international time series | `https://fred.stlouisfed.org/docs/api/fred/` | Free key | `fredapi` |
| **OECD Data Explorer** | Cross-country SME financing, credit conditions | SDMX via `data-explorer.oecd.org` | None | `pandaSDMX` |
| **World Bank Enterprise Surveys** | Credit access, payment terms (160+ economies) | `https://www.enterprisesurveys.org/en/data` | Free registration | -- |

**AITYA mapping:** Interest rates and lending conditions directly affect payment behavior. These macro features are standard exogenous variables in credit risk models. ECB + Bank of Greece provide the most relevant Greek-specific signals; Eurostat business demography gives enterprise survival rates by NACE sector -- powerful priors for counterparty default probability.

---

## 8. Public Procurement Data

### 8.1 TED (Tenders Electronic Daily)

~800,000 public procurement notices per year from all EU member states, worth EUR 815B+ annually. Contract awards, tender notices, and outcome data since 1993.

- **Bulk XML:** `ftp://ted.europa.eu` (credentials: guest/guest)
- **REST API v3:** https://api.ted.europa.eu
- **CSV subset:** https://data.europa.eu/data/datasets/ted-1
- **Python:** `ExtracTED` scripts on GitHub for parsing
- **Format:** XML (full), CSV (subset), JSON (API)
- **License:** EU open data
- **AITYA mapping:** Identifies which companies win government contracts (more predictable revenue stream = lower counterparty risk). Greek government payment delays to suppliers are notoriously slow and visible in this data. Also identifies companies heavily dependent on government revenue -- higher risk if government payments slow further.

---

## 9. Bankruptcy Prediction Datasets (Academic)

### 9.1 UCI Polish Companies Bankruptcy

Financial data of Polish companies with bankruptcy labels from EMIS (Emerging Markets Information Service). Multiple datasets covering 1-5 year prediction horizons.

- **Collection:** `from ucimlrepo import fetch_ucirepo` or direct download
- **Format:** CSV
- **License:** CC BY 4.0
- **AITYA mapping:** European company data (Poland is closer to Greece in economic structure than Taiwan or the US). Pre-train financial distress models before fine-tuning on SAP data.

### 9.2 UCI Taiwanese Bankruptcy Prediction

Financial ratios of 6,819 Taiwanese companies (1999-2009) with binary bankruptcy labels. 95 financial ratio features.

- **Collection:** `from ucimlrepo import fetch_ucirepo; dataset = fetch_ucirepo(id=572)`
- **Format:** CSV
- **License:** CC BY 4.0
- **AITYA mapping:** Useful for benchmarking financial distress prediction approaches. Financial ratios (debt ratio, ROA, current ratio) map to what AITYA could compute from SAP ledger data (`I_GLAcctBalance`). Limitation: Taiwanese market dynamics differ from Greek.

---

## 10. Synthetic Data Tools

Since real SAP AR data is inherently private, synthetic data generation is relevant for development and testing.

### 10.1 SDV (Synthetic Data Vault)

Open-source Python framework for generating synthetic tabular data. Supports multi-table relational structures using statistical models and GANs.

- **Install:** `pip install sdv`
- **GitHub:** https://github.com/sdv-dev/SDV
- **License:** MIT
- **AITYA mapping:** Can generate synthetic multi-table AR datasets for model development: invoices → line items → payments → dunning actions. Train a generative model on one client's data (with permission) and generate synthetic variants for testing.

### 10.2 SAP Datasphere Sample Content

Sample data for SAP Datasphere exploration, including CSV files with SAP-structured data.

- **GitHub:** https://github.com/SAP-samples/datasphere-content
- **License:** SAP sample license
- **AITYA mapping:** Potentially useful for testing CDS view extraction pipelines with SAP-structured sample data.

---

## 11. Relevant Academic Papers

| Paper | Data/Contribution | Reference |
|---|---|---|
| Appel et al., 2019 -- "Optimize Cash Collection" | Invoice payment prediction framework from multinational bank partnership | [arXiv:1912.10828](https://arxiv.org/abs/1912.10828) |
| Appel et al., 2020 -- "Predicting Account Receivables with ML" | AR prediction framework with real company data | [arXiv:2008.07363](https://arxiv.org/pdf/2008.07363) |
| "A framework for modelling customer invoice payment predictions" (2024) | Combined ML and survival analysis for invoice payments | [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2666827024000549) |
| "Late payment prediction through graph features" (TU Delft) | Graph-based features for invoice payment prediction | [TU Delft Repository](https://repository.tudelft.nl/islandora/object/uuid:1b482e0f-6e06-465a-8f7d-ec680d80ac0b/datastream/OBJ/download) |
| "Can we trust ML to predict credit risk of small businesses?" (2024) | Italian SME invoice panel data (534 companies, 2015-2017) | [Springer](https://link.springer.com/article/10.1007/s11156-024-01278-0) |

---

## 12. Notable Non-Options

| Source | Why It's Not Available |
|---|---|
| **PSD2/Open Banking** | Per-consent only; no bulk datasets exist. Theoretically relevant as a future enrichment channel but operationally complex and far beyond MVP scope. |
| **ECB AnaCredit** | Loan-by-loan euro area corporate credit data -- the most granular European credit dataset -- but restricted to central banks and supervisory authorities. Worth monitoring for future data-sharing initiatives. |
| **GEMI full API** | Restricted to public bodies and financial institutions. The most relevant Greek company registry but inaccessible for startups without a partnership. |

---

## Collection Priority Sequence

| Phase | Source | Why | Effort |
|---|---|---|---|
| **1 (prototype)** | Kaggle invoice datasets (IBM + Payment Date) | Only public invoice-level AR data for ML prototyping | Low |
| **2 (macro features)** | ECB + Bank of Greece APIs | Greek-specific macro features via well-documented APIs | Low |
| **3 (Greek context)** | Diavgeia API | Greek government payment graph -- unique counterparty signal | Medium |
| **4 (pre-training)** | SEC EDGAR bulk download | Large-scale DSO/CCC data for pre-training working capital models | Low |
| **5 (EU financials)** | ESEF/ATHEX iXBRL filings | European/Greek listed company balance sheets in structured format | Medium |
| **6 (procurement)** | TED API | Procurement-based counterparty revenue stability signals | Medium |
| **7 (UK financials)** | Companies House bulk | Additional European training data at scale | Medium |
| **8 (benchmarks)** | Intrum, Atradius, Hackett reports | Payment behavior and working capital benchmarks for calibration | Low |
| **9 (risk ratings)** | Coface, Allianz Trade, Eurostat demography | Country/sector risk features and enterprise survival priors | Low |
| **10 (entity resolution)** | OpenCorporates | Counterparty identity resolution across jurisdictions | Medium (cost) |
