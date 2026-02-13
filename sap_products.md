# SAP S/4HANA Products and Deployment Models

*Reference document explaining the SAP product landscape relevant to AITYA's architecture.*

---

## S/4HANA: The ERP Software

SAP S/4HANA is SAP's current-generation enterprise resource planning suite, running on the SAP HANA in-memory database. It replaced SAP ECC (ERP Central Component), which reaches end of mainstream maintenance in **December 2027**. S/4HANA contains the Universal Journal (`ACDOCA`), the CDS views AITYA depends on (`I_JournalEntryItem`, `I_GLAcctBalance`, `I_DunningHistory`, etc.), and the ABAP application server.

S/4HANA is a single software product. The differences that matter for AITYA are in how it is **deployed** and **operated**.

---

## Deployment Models

| Model | Infrastructure managed by | SAP basis/updates managed by | BTP included | Custom ABAP allowed | AITYA deployment friction |
|---|---|---|---|---|---|
| **On-premise** | Customer (own data center) | Customer | No (separate procurement) | Yes (full) | High -- customer must separately procure BTP/AI Core |
| **Private cloud (IaaS)** | Cloud provider (Azure, AWS, GCP) or hosting partner | Customer | No (separate procurement) | Yes (full) | High -- same as on-premise, just different hosting |
| **RISE with SAP (Private Cloud Edition)** | SAP (via hyperscaler) | SAP | Yes (bundled credits) | Limited (clean core model) | Low -- BTP entitlements included, side-by-side extension model |
| **S/4HANA Cloud Public Edition** | SAP | SAP (multi-tenant, forced upgrades) | Yes (bundled) | No (extension via BTP only) | Medium -- BTP available but config table reads restricted |

---

## RISE with SAP

RISE with SAP is not a different ERP product. It is a **commercial and operational wrapper** around S/4HANA that bundles:

1. **Managed infrastructure.** SAP operates the S/4HANA instance on a hyperscaler (Azure, AWS, or GCP). The customer does not manage servers, patching, or basis operations. Single subscription replaces separate license + hosting + basis contracts.

2. **SAP BTP credits.** RISE includes entitlements for SAP Business Technology Platform -- the execution environment for extensions, integrations, analytics, and AI. BTP components relevant to AITYA include:
   - **SAP AI Core** -- container runtime for ML model training and inference (AITYA's federated training environment)
   - **SAP HANA Cloud** -- cloud database for analytics workloads
   - **SAP Analytics Cloud** -- planning and BI
   - **SAP Integration Suite** -- API and event-based integration
   - **SAP Build** -- low-code extensibility

3. **Clean core commitment.** RISE pushes customers to extend via BTP side-by-side applications rather than modifying the S/4HANA ABAP core. This keeps the ERP kernel upgradeable and is the architectural pattern AITYA follows (read-only CDS access from a BTP-deployed container).

4. **SAP controls the stack.** SAP manages the hyperscaler tenant, applies patches, and enforces upgrade cadences. The customer trades infrastructure control for operational simplicity.

### RISE Tiers

| Tier | Description |
|---|---|
| **RISE with SAP** (standard) | Managed S/4HANA Private Cloud + BTP credits |
| **RISE Premium** | Adds SAP Signavio (process mining), SAP Business Network |
| **RISE Premium Plus** | Adds SAP IBP, SAP Analytics Cloud for planning, additional BTP capacity. Sarantis Group is the first worldwide customer for this tier. |

---

## S/4HANA Cloud Public Edition

A fully SAP-managed, multi-tenant S/4HANA instance. Key differences from RISE (Private Cloud Edition):

- **Multi-tenant.** Multiple customers share infrastructure (logically isolated). RISE Private Cloud is single-tenant.
- **No custom ABAP.** All extensions must be built on BTP. RISE Private Cloud allows limited custom ABAP in the core.
- **Forced quarterly upgrades.** SAP pushes updates on its schedule. RISE Private Cloud offers more flexibility on upgrade timing.
- **Restricted config table access.** SAP manages the system configuration. Direct reads of customizing tables (`T052`, `T047`, `SETNODE`, etc.) that AITYA's feature engineering depends on may be blocked or require SAP-approved APIs. This is a material constraint for AITYA's config-as-feature-schema approach documented in [`cds_analysis.md`](cds_analysis.md).

No Greek companies in the current [`greek_market.md`](greek_market.md) target list are confirmed on S/4HANA Cloud Public Edition. All three RISE customers (HELLENiQ ENERGY, Sarantis Group, Piraeus Bank) are on Private Cloud Edition.

---

## Legacy: SAP ECC and Suite on HANA

Companies not yet on S/4HANA may be running:

- **SAP ECC 6.0** -- the prior-generation ERP. Runs on various databases (Oracle DB, SQL Server, DB2, or HANA). Mainstream maintenance ends December 2027, with extended maintenance available through 2030 at a premium. Several Greek companies (e.g., Plaisio, Chipita) are confirmed on ECC and migrating.
- **SAP Business Suite on HANA (SoH)** -- ECC running on the HANA database but without the S/4HANA application layer. Provides some performance benefits but does not include the Universal Journal (`ACDOCA`) or S/4HANA-specific CDS views. Not viable for AITYA.

The 2027 deadline is a forcing function: Tier 3 and Tier 4 companies in [`greek_market.md`](greek_market.md) that are still on ECC must migrate to S/4HANA (in some deployment model) or pay for extended maintenance. This expands AITYA's addressable pipeline over the next 2-3 years.

---

## SAP BTP (Business Technology Platform)

BTP is SAP's platform-as-a-service layer. It is the **execution environment** for AITYA's federated engine. Key services:

| Service | AITYA relevance |
|---|---|
| **SAP AI Core** | Docker container runtime for ML training/inference. AITYA's models train here inside each tenant. |
| **SAP HANA Cloud** | Cloud database. Could serve as the analytics data store for AITYA's predictions. |
| **SAP Integration Suite** | API management and event mesh. Connects BTP extensions to S/4HANA CDS views. |
| **SAP Datasphere** | Data warehousing and federation. Competes with AITYA's approach -- enables cross-system joins without AITYA's privacy guarantees. |

**BTP access by deployment model:**
- **RISE customers** get BTP credits bundled. AITYA can deploy without the customer making a separate platform procurement decision.
- **On-premise / IaaS customers** must procure BTP separately. This adds a sales prerequisite and extends the deal cycle.
- **S/4HANA Cloud Public Edition** customers have BTP but face restrictions on config table access that may limit AITYA's feature engineering.

---

## Why This Matters for AITYA

The deployment model determines AITYA's go-to-market friction:

| Customer deployment | Can AITYA deploy? | Friction level | Greek examples |
|---|---|---|---|
| RISE with SAP (Private Cloud) | Yes -- BTP bundled, CDS views accessible, config tables readable | **Low** | HELLENiQ ENERGY, Sarantis Group, Piraeus Bank |
| S/4HANA on-premise or IaaS | Yes -- but customer must procure BTP separately | **High** | Motor Oil, OPAP, Titan Cement, Coca-Cola HBC, ADMIE, Frigoglass, Plaisio |
| S/4HANA Cloud Public Edition | Partially -- BTP available but config table access may be restricted | **Medium** | None confirmed in Greece |
| SAP ECC (pre-S/4HANA) | No -- lacks Universal Journal and S/4HANA CDS views | **Blocked** | Chipita (migrating), various Tier 3/4 companies |

The ideal customer profile is a RISE with SAP (Private Cloud Edition) customer with BTP already active. In Greece, that is currently 3 companies. The 2027 ECC end-of-life deadline and SAP's aggressive RISE sales motion will expand this pool, but the timing is uncertain for individual companies.

---

## Sources

- [SAP S/4HANA Deployment Options](https://www.sap.com/products/erp/s4hana.html)
- [RISE with SAP Overview](https://www.sap.com/products/erp/rise.html)
- [SAP BTP Overview](https://www.sap.com/products/technology-platform.html)
- [SAP ECC End of Maintenance Timeline](https://community.sap.com/t5/enterprise-resource-planning-q-a/why-businesses-must-move-to-s-4hana-before-2027-risks-and-opportunities/qaq-p/14058316)
- [Deutsche Telekom RISE with SAP (March 2024)](https://news.sap.com/2024/03/deutsche-telekom-rise-with-sap-cloud-transformation/)
- [Sarantis Group RISE Premium Plus (May 2024)](https://news.sap.com/africa/2024/05/sarantis-group-is-accelerating-digital-transformation-with-sap/)
