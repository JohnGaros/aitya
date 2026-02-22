# SAP Payment Terms Infrastructure

Deep-dive into how SAP S/4HANA stores, configures, and resolves payment terms. This document extends the Configuration Layer section of [`pragmatic.md`](pragmatic.md) and supports future elaboration of the normalization pipeline design.

---

## Why This Matters for AITYA

[`pragmatic.md`](pragmatic.md) claims that "a single read of `T052` at onboarding resolves every payment terms code to a canonical representation." That claim is correct in principle -- payment terms are deterministic arithmetic, not free-text taxonomy -- but the underlying table structure is more layered than the current description conveys. The normalization pipeline must handle four distinct complexity layers, not just the simple "Net 30, 2/10" case. This document maps those layers and identifies the implementation gaps.

---

## Table Architecture

SAP payment terms are distributed across three related customizing tables:

| Table | Purpose | Cardinality | Read frequency |
|---|---|---|---|
| `T052` | Payment terms master: day counts, discount percentages, baseline date logic, fixed-day rules, installment flag | One row per `ZTERM` × `ZTAGG` (day-limit version) | Once at deployment |
| `T052U` | Descriptive texts per `ZTERM` (language-dependent) | One row per `ZTERM` × language | Once at deployment |
| `T052S` | Installment splitting: links parent `ZTERM` to child payment terms with percentage allocations | One row per parent `ZTERM` × installment sequence number | Once at deployment |

All three are small customizing tables (typically a few hundred rows total). The sensitivity profile is negligible -- these contain payment policy parameters, not transactional or customer data.

---

## T052 Field Inventory (26 Fields)

### Key Fields

| Field | Type | Description |
|---|---|---|
| `MANDT` | CLNT 3 | Client (key) |
| `ZTERM` | CHAR 4 | Payment terms key -- the 4-character alphanumeric code referenced throughout transactional and master data |
| `ZTAGG` | NUMC 2 | Day limit -- enables multiple versions of the same `ZTERM` (see Layer 2 below) |

### Baseline Date Calculation

| Field | Type | Description |
|---|---|---|
| `ZDART` | CHAR 1 | Date type for baseline date: posting date, document date, or entry date |
| `ZFAEL` | NUMC 2 | Fixed calendar day for baseline date calculation (e.g., "15" means baseline snaps to the 15th) |

### Discount Periods and Net Due (Layer 1)

| Field | Type | Description |
|---|---|---|
| `ZTAG1` | NUMC 3 | Days from baseline date -- first discount period |
| `ZPRZ1` | DEC 5.3 | Cash discount percentage -- first period |
| `ZTAG2` | NUMC 3 | Days from baseline date -- second discount period |
| `ZPRZ2` | DEC 5.3 | Cash discount percentage -- second period |
| `ZTAG3` | NUMC 3 | Days from baseline date -- net due (no discount) |

**Field name note:** The T052 discount percentage fields are `ZPRZ1`/`ZPRZ2`. The fields `ZBD1P`/`ZBD2P`/`ZBD3P` referenced in [`pragmatic.md`](pragmatic.md) are the corresponding *document-level* fields in BSEG, where the resolved payment terms are stored per line item. The distinction matters because T052 has two discount percentage fields while BSEG has three -- the third BSEG field (`ZBD3P`) is always zero and exists for structural symmetry with the three day-count fields. This should be corrected in `pragmatic.md` when the configuration layer description is next revised.

### Fixed-Day / End-of-Month Logic (Layer 3)

| Field | Type | Description |
|---|---|---|
| `ZMONA` | NUMC 2 | Additional months added to baseline before applying day counts |
| `ZSTG1` | NUMC 2 | Fixed due day for special condition (term 1) |
| `ZSMN1` | NUMC 2 | Additional months for special condition (term 1) |
| `ZSTG2` | NUMC 2 | Fixed due day for special condition (term 2) |
| `ZSMN2` | NUMC 2 | Additional months for special condition (term 2) |
| `ZSTG3` | NUMC 2 | Fixed due day for special condition (term 3) |
| `ZSMN3` | NUMC 2 | Additional months for special condition (term 3) |

### Payment Processing

| Field | Type | Description |
|---|---|---|
| `ZLSCH` | CHAR 1 | Default payment method (references `T042Z`) |
| `ZSCHF` | CHAR 1 | Payment block (default value) |
| `XCHPB` | CHAR 1 | Transfer payment block when terms change |
| `XCHPM` | CHAR 1 | Transfer payment method when terms change |
| `TXN08` | CHAR 10 | Standard text number for printed output |
| `XZBRV` | CHAR 1 | Print terms on RV (sales) documents |

### Installment and Account Type Indicators

| Field | Type | Description |
|---|---|---|
| `XSPLT` | CHAR 1 | Installment payment indicator -- when set, this `ZTERM` is a parent that splits via `T052S` |
| `KOART` | CHAR 1 | Account type (customer/vendor) -- determines which subledger the terms apply to |
| `XSCRC` | CHAR 1 | Recurring entries: add terms from master record |

---

## T052S Field Inventory (Installment Splitting)

| Field | Type | Description |
|---|---|---|
| `MANDT` | CLNT 3 | Client (key) |
| `ZTERM` | CHAR 4 | Parent payment terms key (key) |
| `RATNR` | NUMC 2 | Installment sequence number (key) -- ordering of the tranches |
| `RATPZ` | DEC 5.3 | Percentage of total amount allocated to this installment |
| `RATZT` | CHAR 4 | Child payment terms key -- itself a `ZTERM` in `T052` with its own day counts, discounts, and conditions |

Configured via transaction `OBB9`. The parent `ZTERM` (flagged with `XSPLT`) does not itself carry day counts -- it delegates to N child `ZTERM` entries, each with independent payment conditions.

---

## Four Layers of Complexity

### Layer 1: Simple Terms

**What it covers:** "Net 30", "2/10 Net 30", "1/10 2/20 Net 45".

**How it works:** `ZTAG1`/`ZPRZ1` define the first discount window, `ZTAG2`/`ZPRZ2` the second, and `ZTAG3` the net due date. The baseline date is determined by `ZDART` (posting date, document date, or entry date).

**Canonical representation:** A flat tuple: `(discount_1_days, discount_1_pct, discount_2_days, discount_2_pct, net_days)`.

**Normalization complexity:** Trivial. This is the case [`pragmatic.md`](pragmatic.md) currently describes.

### Layer 2: Day Limits (Conditional Terms)

**What it covers:** "If invoice date is before the 15th, net 30; if after the 15th, net 45."

**How it works:** The `ZTAGG` (day limit) field enables multiple rows in `T052` for the same `ZTERM`, each with a different day-limit threshold. The system selects the version whose `ZTAGG` value is >= the baseline date's day-of-month. This allows a single payment terms code to produce different effective terms depending on *when* the invoice is posted.

**Canonical representation:** A conditional tuple: `[(day_limit_1, terms_1), (day_limit_2, terms_2), ...]` ordered by `ZTAGG`.

**Normalization complexity:** Moderate. The normalization pipeline must either:
- Resolve against actual transaction baseline dates at feature-engineering time (accurate but couples normalization to transaction data), or
- Represent the full conditional structure in the canonical form and let the ML pipeline handle the conditionality (clean separation but more complex feature schema).

**Prevalence:** Common in companies with month-end billing cycles or seasonal invoicing patterns.

### Layer 3: Fixed-Day and End-of-Month Terms

**What it covers:** "Due on the 15th of the following month", "End of month + 60 days", "Due on the last day of the second month after invoice".

**How it works:** The `ZMONA` field adds calendar months to the baseline before day counts apply. The `ZSTG1-3`/`ZSMN1-3` fields define fixed calendar days with their own month offsets for each discount/net-due period. The `ZFAEL` field can snap the baseline date itself to a fixed calendar day.

**Canonical representation:** A date-calculation rule, not a simple day count. E.g., `baseline + 1 month → snap to 15th` or `baseline + 2 months → last day of month`. The rule is deterministic but requires a date arithmetic engine to resolve to actual due dates.

**Normalization complexity:** Moderate-to-high. The canonical representation is no longer a flat "Net N days" -- it is a parameterized date function. Cross-tenant comparison requires either:
- Resolving to effective day counts for a reference date (lossy -- the same terms produce different day counts in different months), or
- Comparing the date-calculation rules structurally (preserves semantics but requires a richer comparison function than simple numeric equality).

**Prevalence:** Very common in European B2B (where end-of-month terms are standard practice) -- directly relevant to AITYA's Greek target market.

### Layer 4: Installment Plans

**What it covers:** "30% due in 30 days, 30% due in 60 days, 40% due in 90 days", retainage/holdback in construction, milestone-based billing.

**How it works:** The parent `ZTERM` has `XSPLT = X` and delegates to `T052S`, which maps each installment to a child `ZTERM` (with its own Layer 1-3 conditions) and a percentage allocation. Each child is itself a full payment term that can use day limits, fixed-day logic, etc.

**Canonical representation:** A tree:
```
Parent ZTERM (XSPLT = X)
├── Installment 1: 30% → Child ZTERM_A (Net 30)
├── Installment 2: 30% → Child ZTERM_B (Net 60)
└── Installment 3: 40% → Child ZTERM_C (Net 90, end-of-month)
```

Each child node is itself a Layer 1-3 canonical representation.

**Normalization complexity:** High. Cross-tenant comparison of installment plans requires comparing trees of payment schedules. Two companies with "3-tranche, equal split, 30/60/90 net" are semantically identical even if their `ZTERM` codes, child codes, and percentage splits differ slightly (e.g., 33/33/34 vs. 30/30/40). The normalization pipeline needs a similarity function over payment schedule trees, not just equality over flat tuples.

**Prevalence:** Common in construction, engineering, project-based industries, and large capital goods transactions. Less common in standard receivables but not negligible in enterprise SAP environments. [`pragmatic.md`](pragmatic.md) currently flags this as "uncommon in standard receivables" -- that assessment may understate the prevalence for AITYA's enterprise target market.

---

## Gap Between Current Documentation and Implementation Reality

| Aspect | What `pragmatic.md` currently says | What implementation requires |
|---|---|---|
| Tables read | `T052` / `T052U` | `T052` + `T052U` + `T052S` (for installment terms) |
| Canonical form | Flat string: "Net 30 days, 2% discount if paid within 10 days" | Conditional tree: day-limit versions × fixed-day rules × installment splits |
| Cross-tenant comparison | Implicit: compare canonical strings | Requires structured comparison: numeric equality for Layer 1, conditional matching for Layer 2, date-rule comparison for Layer 3, tree similarity for Layer 4 |
| Field names | References `ZBD1P`/`ZBD2P`/`ZBD3P` | These are BSEG document-level fields; T052 uses `ZPRZ1`/`ZPRZ2` |
| Complexity assessment | "No human interpretation required" | True for all layers (all are deterministic arithmetic), but the *implementation* effort increases significantly from Layer 1 to Layer 4 |

**The core claim remains valid:** all four layers are parameterized formulas with deterministic semantics. Payment terms do not degrade into free-text taxonomy at any layer. The "deterministic arithmetic" property that [`cds_analysis.md`](cds_analysis.md) identifies as the precondition for automated normalization holds across the full complexity stack. The gap is not conceptual but implementational -- the pipeline is more complex than a flat table lookup.

---

## Implications for AITYA's Normalization Pipeline

### Minimum Viable Normalization (Launch)

For the Cash & Collections MVP, Layers 1-2 likely cover the majority of payment terms encountered in standard receivables. A launch-quality pipeline could:
1. Read `T052` for all `ZTERM` codes where `XSPLT` is not set.
2. Resolve Layer 1 terms to flat tuples.
3. Handle Layer 2 day limits by resolving against actual transaction baseline dates.
4. Flag Layer 3 (fixed-day/end-of-month) terms for date-rule-based resolution.
5. Flag Layer 4 (installment) terms as requiring `T052S` reads and tree comparison.

### Full Normalization (Required for Enterprise Credibility)

Enterprise SAP customers -- especially in the European markets AITYA targets -- will routinely use Layers 3-4. End-of-month terms are the norm in European B2B, and installment plans are standard in capital-intensive industries. A pipeline that only handles "Net N days" will silently misrepresent payment behavior for a significant fraction of receivables.

### Feature Engineering Consequences

The complexity layer of the payment terms affects how features derived from payment timing should be interpreted:

- **Layer 1-2:** `days_past_due = actual_payment_date - (baseline_date + ZTAG3)` is straightforward.
- **Layer 3:** `days_past_due` requires resolving the date-calculation rule against the specific invoice's baseline date and calendar context.
- **Layer 4:** A single invoice generates multiple due dates. `days_past_due` becomes a vector, not a scalar. The ML pipeline must decide whether to treat each installment as an independent observation or model the payment schedule as a structured feature.

---

## Open Questions for Future Elaboration

1. **Prevalence by layer in the Greek target market.** What percentage of `ZTERM` codes in a typical Greek enterprise SAP system fall into each layer? If Layer 3 (end-of-month) is dominant in European B2B, the "simple case" described in `pragmatic.md` may be the minority case in practice.

2. **Interaction with partial clearing.** Installment payment terms (Layer 4) interact directly with the partial clearing ambiguity documented in [`pragmatic.md`](pragmatic.md). Each installment generates a separate accounting document -- does SAP use the "partial payment" or "residual payment" method for installments, or is it configurable per tenant?

3. **Day-limit resolution strategy.** Should the normalization pipeline resolve day limits at onboarding (producing a single canonical term per `ZTERM` based on average or most-common baseline dates) or at feature-engineering time (producing transaction-specific effective terms)? The former is simpler but lossy; the latter is accurate but couples normalization to transaction data.

4. **`T052S` access in S/4HANA Public Cloud.** [`pragmatic.md`](pragmatic.md) notes that S/4HANA Public Cloud blocks most customizing table reads. Is `T052S` accessible, or does the installment layer become invisible for Public Cloud tenants?

5. **Payment method field (`ZLSCH`).** `T052` carries a default payment method. This field is not currently in AITYA's feature scope but could be a useful segmentation axis (bank transfer vs. check vs. direct debit have different clearing velocity profiles). Worth evaluating for future feature expansion.

6. **Holdback/retainage semantics.** `T052S` is documented by SAP as "Terms of Payment for Holdback/Retainage" -- suggesting its primary use case is construction-style holdbacks, not just generic installment splitting. The distinction matters: holdback terms may have different prevalence and feature-engineering implications than equal-split installment plans.

---

## References

- [SAP T052 Table Structure -- SAP Datasheet](https://www.sapdatasheet.org/abap/tabl/t052.html)
- [SAP T052S Table Structure -- SAP Datasheet](https://www.sapdatasheet.org/abap/tabl/t052s.html)
- [SAP Payment Terms Tables -- SAP4TECH](https://sap4tech.net/sap-payment-terms-tables/)
- [SAP T052 Field Reference -- LeanX](https://leanx.eu/en/sap/table/t052)
- [Installment Payment Configuration -- SAP Community](https://community.sap.com/t5/enterprise-resource-planning-q-a/how-to-configure-installment-payment-terms/qaq-p/9316702)
- [Configuring Payment Terms and Cash Discounts -- SAP Learning](https://learning.sap.com/courses/customizing-core-settings-in-financial-accounting-in-sap-s4hana/configuring-payment-terms-and-cash-discounts)
