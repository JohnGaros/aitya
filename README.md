# AITYA

*The reason behind the data.*

AITYA is a business plan for a startup building a **federated AI analytics engine for SAP S/4HANA**. The core idea: train ML models locally inside each client's SAP BTP tenant so that raw financial data never leaves the premises, while encrypted gradients are shared across tenants to enable collective intelligence.

The product focus is **Cash & Collections Intelligence** -- predictive collection scoring and working-capital analytics built on two data domains (General Ledger and Dunning) chosen for their cross-tenant portability and minimal customization surface.

## Repository Structure

This is a document-only repository. Start with [`summary.md`](summary.md) for an overview or [`pragmatic.md`](pragmatic.md) for the full product thesis.

| File | Purpose |
|---|---|
| [`summary.md`](summary.md) | Consolidated overview of all key ideas |
| [`pragmatic.md`](pragmatic.md) | Product thesis: architecture, semantic pillars, deployment, and product definition |
| [`greek_market.md`](greek_market.md) | Target market analysis (Greek SAP users by cloud readiness) |
| [`sap_products.md`](sap_products.md) | SAP S/4HANA deployment models and BTP reference |
| [`sap_partners.md`](sap_partners.md) | Go-to-market: Greek SAP partners and Big 4 strategy |
| [`cds_analysis.md`](cds_analysis.md) | CDS view viability evaluation for federated ML |
| [`price_elasticity.md`](price_elasticity.md) | Why current CDS scope cannot support price elasticity |
| [`rl_feasibility.md`](rl_feasibility.md) | Reinforcement learning feasibility analysis |
| [`architecture_comparison.md`](architecture_comparison.md) | Model-to-data vs. data-to-model trade-offs |
| [`zero_copy_viability.md`](zero_copy_viability.md) | Viability given SAP's zero-copy partnerships |
