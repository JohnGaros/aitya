# SAP S/4HANA Ledger Reconciliation with CDS Views

This document outlines the standard, stable CDS (Core Data Services) views in SAP S/4HANA used for reconciling financial ledger balances. The core principle is to ensure that for any given G/L account, the sum of its transactions within a period correctly explains the change from its beginning balance to its ending balance.

This approach mirrors the data-first strategy outlined in [`pragmatic.md`](pragmatic.md), relying on standard, CISO-friendly views for data extraction.

## The Reconciliation Challenge

The fundamental equation for financial reconciliation is:

$$
\text{Starting Balance} + \sum(\text{Debits}) - \sum(\text{Credits}) = \text{Ending Balance}
$$

To implement this check in S/4HANA, you need reliable views for both aggregated balances and the individual transactions (line items). S/4HANA provides two primary, highly stable CDS views for this purpose, both built on the Universal Journal (`ACDOCA`).

### 1. For Transactions (Debits and Credits): `I_JournalEntryItem`

This is the primary view for accessing individual financial line items. It is the modern successor to the classic `BSEG` table.

- **Purpose:** Provides the granular journal entries (debits and credits) for a given period. This is where you retrieve the individual transactions to be summed up.
- **Stability:** **Very High.** As one of the most fundamental CDS views in S/4HANA Finance, it serves as the primary, stable interface for all line-item reporting.
- **Key Fields for Reconciliation:**
  - `DebitAmountInCoCodeCrcy`
  - `CreditAmountInCoCodeCrcy`
  - `GLAccount`
  - `CompanyCode`
  - `Ledger`
  - `PostingDate`

### 2. For Balances (Beginning and Ending): `I_GLAcctBalance`

This view provides pre-aggregated balances, making it ideal for high-level reporting and for retrieving the start and end points of your reconciliation equation.

- **Purpose:** Calculates and delivers the beginning balance, ending balance, and total debit and credit movements for G/L accounts over a specified period.
- **Stability:** **Very High.** This is the standard, SAP-provided view for any trial balance, balance sheet, or other balance-based financial statement.
- **Key Fields for Reconciliation:**
  - `StartingBalanceInCoCodeCrcy`
  - `EndingBalanceInCoCodeCrcy`
  - `DebitAmountInCoCodeCrcy` (Total for the period)
  - `CreditAmountInCoCodeCrcy` (Total for the period)

### How to Implement the Reconciliation Logic

A robust reconciliation application would use these views together to provide both a summary and detailed proof.

1. **Get Balances from `I_GLAcctBalance`:**
   - Query `I_GLAcctBalance` for a specific `CompanyCode`, `Ledger`, `GLAccount`, and `FiscalYearPeriod`.
   - Retrieve the `StartingBalanceInCoCodeCrcy` and `EndingBalanceInCoCodeCrcy`.

2. **Get Transactions from `I_JournalEntryItem`:**
   - For the same set of filters, query `I_JournalEntryItem`.
   - Calculate `SUM(DebitAmountInCoCodeCrcy)` and `SUM(CreditAmountInCoCodeCrcy)` for all entries within the period.

3. **Validate and Display:**
   - Confirm that the reconciliation equation holds true.
   - In a UI, you can display the balances from `I_GLAcctBalance` and allow a user to "drill down" to see the list of corresponding line items from `I_JournalEntryItem` that make up those balances.

This two-view approach provides a complete solution, offering both the high-level validated summary and the detailed transactional proof required for financial auditing and analysis.
