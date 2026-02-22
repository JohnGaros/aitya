# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

AITYA ("The reason behind the data") is a business plan for a startup building a federated AI analytics engine for SAP S/4HANA. This is a **document-only repository** -- there is no source code, build system, or tests. All content is in Markdown files.

## Document Map

| File | Purpose |
|---|---|
| `summary.md` | **Start here.** Consolidated overview of all key ideas. |
| `pragmatic.md` | **Product thesis.** Self-contained document covering the core idea, federated architecture (benefits and risks), the two semantic pillars (Ledger and Dunning), deployment architecture, and the product definition. This is the single source of truth for AITYA's product ideas. |
| `greek_market.md` | Target market: 34 confirmed/likely Greek SAP users organized by cloud readiness tier (3 RISE, 7 S/4HANA, 10 SAP version unclear, 14 partner-list evidence) |
| `sap_products.md` | Reference: SAP S/4HANA deployment models (on-premise, IaaS, RISE, Public Cloud), BTP services, RISE tiers, ECC end-of-life, and why deployment model determines AITYA's go-to-market friction |
| `sap_partners.md` | Go-to-market: 5 Greek SAP partners + Big 4 co-opetitor strategy |
| `cds_analysis.md` | CDS view evaluation: which S/4HANA views are viable for federated ML and why others fail; includes model-to-data single-tenant viability analysis and config-as-feature-schema taxonomy |
| `price_elasticity.md` | Analysis of why the current CDS scope cannot support price elasticity estimation; identifies missing data inputs, required additional views (`I_BillingDocumentItem`, pricing config tables), and fundamental methodological barriers in B2B ERP data |
| `rl_feasibility.md` | Analysis of whether the current CDS scope supports reinforcement learning-style state-action value estimation; concludes the observable action space (dunning escalation only) is too narrow for genuine RL but supports supervised prediction and cross-tenant dunning policy evaluation |
| `architecture_comparison.md` | Model-to-data vs. data-to-model comparison for S/4HANA Cloud: architecture, sovereignty, data scope, onboarding, model quality, cost, moat, and why the CDS exclusion criteria are self-imposed trade-offs that data-to-model competitors solve at the integration layer |
| `zero_copy_viability.md` | Strategic evaluation of AITYA's viability given SAP's zero-copy partnerships (Snowflake, Databricks, Google BigQuery): how SAP's platform strategy erodes data access friction, where federated analytics retains unique value, realistic GTM segments and revenue models, and five hard questions the business must answer |
| `payment_terms.md` | Deep-dive into SAP's payment terms table architecture (`T052`, `T052U`, `T052S`): four complexity layers (simple terms, day limits, fixed-day/end-of-month, installment plans), field inventory, gap analysis between `pragmatic.md`'s normalization claim and implementation reality, and open questions for pipeline design |
| `network_effects.md` | Analysis of network effect strategies beyond federated learning: evaluates cross-tenant benchmarking, counterparty graph, two-sided marketplace, and community effects against genuine network-effect criteria; concludes no strategy is viable at launch scale and recommends sequencing benchmarking first while architecting toward the counterparty graph |

**Archived in `archive/` (superseded by `pragmatic.md`):** `executive_summary.md`, `intelligence_engine.md`, `value_proposition.md`, `sap_ledger_balance.md`, `revenue_forecast.md`, `skeptic_review.md`. These files are kept for reference but their content has been consolidated and refined in `pragmatic.md`.

## Key Concepts

- **Federated Zero-Export Intelligence:** Models train locally inside each client's BTP tenant via SAP AI Core; only encrypted gradients are shared. Raw data never leaves. Benefits and risks are detailed with equal weight in `pragmatic.md`.
- **Two Semantic Pillars:** Ledger (`I_JournalEntryItem`, `I_GLAcctBalance`) and Dunning (`I_DunningHistory`) -- selected because they are governed by double-entry accounting, have minimal customization surface, and map to universal financial language (DSO, cash conversion, bad debt). These properties make them viable for tenant-agnostic feature engineering in a federated ML setting.
- **Cash & Collections Intelligence:** The product built on the two pillars. Combines predictive collection scoring with reconciliation-backed working-capital analytics.

## Critical Thinking Mandate

When the user asks for an opinion on an idea, **default to skepticism**. The role is adversarial reviewer, not cheerleader. Specifically:

1. **Find flaws first.** Identify logical gaps, weak assumptions, and dependencies that could break the idea. Do not lead with praise or validation.
2. **Provide counterarguments.** For every claimed advantage, articulate the strongest case against it -- what a sophisticated competitor, a skeptical CFO, or a hostile investor would say.
3. **Demand missing evidence.** Call out assertions that lack data, citations, back-testing, or real-world validation. Ask: "What would need to be true for this to work, and is there proof that it is?"
4. **Surface unintended consequences.** Think second- and third-order effects: if this idea succeeds, what new problems does it create? Who loses, and how might they react?
5. **Think long-term.** Evaluate whether the idea builds a durable position or creates fragile dependencies that compound over time (vendor lock-in, partner power shifts, regulatory changes).
6. **Use web search when needed.** Do not rely on assumptions about market size, competitor capabilities, regulatory status, or technology maturity. Search for current data to ground the critique in reality rather than speculation.
7. **Create doubt, not despair.** The goal is to stress-test, not to dismiss. After identifying weaknesses, suggest what evidence or changes would resolve them -- but do not soften the critique itself.

The `archive/skeptic_review.md` document is a model for this tone. Match its directness.

## Editing Conventions

- Cross-reference other documents with relative Markdown links: `[text](filename.md)`.
- When adding new analytical ideas, update both the relevant source document **and** `summary.md`.
- Risks are integrated directly into `pragmatic.md` alongside benefits. New ideas should include their own risk assessment in the same document, not in a separate file.
- SAP CDS view names use the exact SAP naming convention (e.g., `I_JournalEntryItem`, `I_GLAcctBalance`, `I_DunningHistory`).
