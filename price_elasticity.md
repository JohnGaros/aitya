# Price Elasticity Estimation: CDS View Feasibility Analysis

This document evaluates whether the CDS views available in AITYA's current scope ([`pragmatic.md`](pragmatic.md)) can support price elasticity estimation, and what additional views and configuration tables would be required.

---

## Short Answer

**No. The current scope cannot estimate price elasticities. The data inputs required are fundamentally absent.**

Price elasticity estimation requires three things the current CDS views do not provide:

### What's Missing

**1. No prices.** The ledger views (`I_JournalEntryItem`, `I_GLAcctBalance`) record monetary amounts — debits and credits in company code currency — but not unit prices. A €50,000 journal entry tells you revenue was recognized; it does not tell you whether that's 500 units at €100 or 5,000 units at €10.

**2. No quantities.** Journal entries are purely financial. There are no physical quantity fields, no units sold, no volume metrics. You cannot construct a demand curve without observing quantity variation.

**3. No product dimension.** The views are keyed by G/L account, company code, customer, and posting date — but not by material or product. You cannot decompose revenue into product-level price-quantity pairs, which is the fundamental unit of observation for elasticity estimation.

---

## What You Would Need

The minimum viable addition is **`I_BillingDocumentItem`**, which carries:

- `BillingQuantity` / `BillingQuantityUnit` (quantity sold)
- `NetAmount` (invoiced amount — gives you effective price per unit)
- `Material` (product dimension)
- `SoldToParty` (customer)
- `BillingDocumentDate` (time dimension for observing variation)

This is the view that [`cds_analysis.md`](cds_analysis.md) explicitly **rejected for federated ML** and rated only "conditionally viable" for single-tenant analytics, because the pricing condition technique (`KONV`, `T683S` → `T685`) is one of the most heavily customized areas in any S/4HANA implementation.

To decompose price into its components (base price, volume discounts, rebates, surcharges), you would additionally need the **pricing condition tables**:

- `T685` — condition type definitions (price, discount, surcharge, tax)
- `T683S` — pricing procedure steps
- `KONP` / `A***` tables — condition records (commercially sensitive)

These are inaccessible in S/4HANA Public Cloud Edition and carry significant residual complexity in on-premise deployments (50+ condition types, customer/material-specific records, time-dependent validity).

---

## Even with the Right Views, the Methodology Is Fragile

Suppose you add `I_BillingDocumentItem` and the pricing config tables. You now have price-quantity pairs by product-customer-period. Can you estimate elasticity? Only under conditions that rarely hold in S/4HANA's typical user base:

**Endogeneity.** Prices in ERP systems are not randomly assigned — they are set by sales teams responding to the same demand conditions you are trying to measure. Without exogenous price variation (natural experiments, A/B tests, instrumental variables), any elasticity estimate conflates cause and effect. "Revenue dropped after a price increase" might mean elastic demand, or it might mean the price increase was triggered by a market downturn that independently reduced volume.

**Insufficient price variation.** Most S/4HANA customers are B2B enterprises. B2B pricing is contract-driven: a customer negotiates a price list or framework agreement that holds for 12-24 months. You might observe 2-3 price changes per customer-product pair over several years — far too few data points for a statistical estimate. This is fundamentally different from B2C/retail, where prices change weekly and elasticity estimation is routine.

**Missing confounders.** Competitor pricing, macroeconomic conditions, seasonality, promotional activity, substitute availability — all influence demand but are not captured anywhere in S/4HANA's financial or billing data. An elasticity model without these controls produces biased estimates. The ERP records the outcome of market interactions, not the market context.

**Aggregation level mismatch.** Price elasticity is meaningful at the product-market level. S/4HANA billing data is at the invoice line item level. Aggregating to the right level requires knowing the company's product hierarchy, market segmentation, and channel structure — all company-specific taxonomies with no cross-tenant portability.

---

## Summary Matrix

| Requirement | Available in current scope? | Available with additions? |
|---|---|---|
| Unit prices | No | Yes, via `I_BillingDocumentItem` |
| Quantities sold | No | Yes, via `I_BillingDocumentItem` |
| Product dimension | No | Yes, via `Material` field on billing |
| Price decomposition | No | Partially, via `T683S`/`T685` (on-prem only) |
| Exogenous price variation | No | No — requires experimental design or external data |
| Confounder controls | No | No — competitor/market data is outside ERP |
| Sufficient price variation (B2B) | N/A | Unlikely — contract pricing creates sparse observations |

---

## Why This Is Outside AITYA's Natural Domain

The current two-pillar scope was deliberately selected because ledger and dunning data answer questions about **payment behavior and credit risk** — domains where the CDS views contain the complete causal chain (invoice → due date → payment/escalation → clearing). Price elasticity is a fundamentally different question — it asks about **demand response to price changes** — and the causal chain (price change → quantity change, controlling for everything else) is not observable from financial postings alone.

Adding `I_BillingDocumentItem` gets you the raw ingredients but not the experimental design. You would need to pair it with external market data or exploit quasi-natural experiments (e.g., price list changes that affected some customer segments but not others), which moves well beyond what an automated CDS-based pipeline can deliver.
