# Comprehensive Index of `TransactionsHelper` and `TransactionsMultiHelper` Functions

This document provides a complete index of all functions available in the `TransactionsHelper` class and the `TransactionsMultiHelper` trait, organized by their area of use and nature.

> **Note:** All functions are `static` and can be called directly via `TransactionsHelper::methodName()`.

---

## 1. Journal Entry Creation Functions

| Function | Type | Description |
| :--- | :--- | :--- |
| `createSingle` | `public` | Creates a simple two-party journal entry (debit and credit). The backbone of basic financial operations. Used internally by most payment and business operation functions. |
| `createEntry` | `public` | A unified function that automatically chooses between `createSingle` (for two parties) and `createJournalEntry` (for more than two parties). Can force the use of `createJournalEntry` via the `forceJournal => true` option. |
| `createJournalEntry` | `public` | Creates a multi-party journal entry with comprehensive validation, currency handling, balance checks, dynamic account type rules, and exchange differences. |
| `testJournalEntry` | `public` | Fully simulates the creation of a multi-party journal entry without saving to the database, with detailed result analysis (totals, currencies, violations, warnings). |

---

## 2. Specialized Financial Operations Functions (from `TransactionsHelper`)

### 2.1 Courier and Customer Balance Management

| Function | Type | Description |
| :--- | :--- | :--- |
| `createMonyDeliverys` | `public` | Add an opening balance for a courier (Delivery) from the company, with duplicate prevention. |
| `createMonyDeliverysFromSyndical` | `public` | Transfer balance to a courier from an agent (Syndical/Department), checking the agent's balance. |
| `createMonyDeliverysFromCompanys` | `public` | Transfer balance to a courier from the official company (requires backend user permissions). |
| `createDeductingMonyDeliverysFromCompanys` | `public` | Deduct an amount from a courier's account (e.g., app commission), with support for custom notes and persons. |

### 2.2 Order-Related Operations

| Function | Type | Description |
| :--- | :--- | :--- |
| `createBrokerageFromOrders` | `public` | Deduct delivery commission from the courier based on the order. |
| `createPaymentFromOrders` | `public` | Record an order payment entry (from customer to company/courier account). |
| `createBillShopFromOrders` | `public` | Record store purchase value (or courier in case of online payment). |
| `createBrokerageShopFromOrders` | `public` | Deduct app commission from the store. |
| `createShippingTotalFromOrders` | `public` | Record delivery cost entry for the courier (in case of online payment). |
| `createTipAmountFromOrders` | `public` | Record tip amount entry for the courier. |
| `createPaymentFromOrdersOld` | `public` | (Legacy) Older version of the order payment entry function. |
| `createPaymentMonyDeliverysFromCompanys` | `public` | (Legacy) Previous attempt at a comprehensive payment entry for orders. |

### 2.3 Payment and Verification Functions

| Function | Type | Description |
| :--- | :--- | :--- |
| `addPayment` | `public` | Add a financial payment from a third party (e.g., bank) to a specific account (e.g., student account), with full control over debit and credit parties via settings. |
| `checkAllowPay` | `public` | Check if the balance of a given account (must be credit) is sufficient to execute a purchase or payment. |
| `checkTransactions` | `public` | Search for a specific transaction header and ensure it exists. Used before update or cancellation. |

---

## 3. Query and Data Retrieval Functions

| Function | Type | Description |
| :--- | :--- | :--- |
| `getTransactionHeader` | `public` | Advanced query on transaction headers (`TransactionHeader`) with comprehensive filtering by department, period, transaction type, person, advanced fields, and date ranges. |
| `seedDocsNumber` | `public` | Number old records in the `tss_accounts_transaction_headers` table that do not have a `docs_number`. |

---

## 4. Exchange Difference Handling Functions (from `TransactionsMultiHelper`)

| Function | Type | Description |
| :--- | :--- | :--- |
| `checkExchangeDifferences` | `public` | Check an existing journal entry to determine exchange differences based on the posting date, with options to create a difference entry, fire an event, or return information only. |
| `createExchangeDifferenceEntry` | `public` | Create an exchange difference entry resulting from a change in exchange rate between the entry date and the posting date, linking it to the original entry. |
| `handleExchangeDifferences` | `protected` | Internal handling of exchange differences that is automatically called after saving an entry if `handle_exchange_differences = true`. |

---

## 5. Validation and Internal Checking Functions (from `TransactionsMultiHelper`)

| Function | Type | Description |
| :--- | :--- | :--- |
| `validateJournalEntry` | `public` | Validate the data of a multi-party journal entry without creating it; returns the processed details after checking balance and constraints. |
| `validateJournalDetails` | `protected` | Comprehensive validation of all entry details (each `details` element) and returns the cleaned array of details. |
| `validateSingleDetail` | `protected` | Validate a single detail (accounting party) including account, person, notes, currencies, and extra fields. |
| `validateSideField` | `protected` | Validate a specific field (e.g., `account` or `notes`) on a specific side (debit/credit) based on `allowed`/`required`/`default` rules. |
| `validateMultiplicity` | `protected` | Check multiplicity constraints (maximum number of entries, allowing multiple debit/credit). |
| `validateAmountLimits` | `protected` | Check total and per-side amount limits. |
| `validateBalance` | `protected` | Enable balance checking for each detail according to settings (debit/credit). |
| `checkDetailBalance` | `protected` | Check the balance of a single detail using `checkAccountBalance` or a custom callback. |
| `validatePeriod` | `protected` | Check the validity of the accounting period (date within range, period open). |
| `validatePaperId` | `protected` | Check for duplicate `paper_id` if required (according to the duplication scope). |
| `validatePaperIdRequired` | `protected` | Check if `paper_id` is required. |
| `validateDateAtRequired` | `protected` | Check if `date_at` is required. |

---

## 6. Data Preparation and Building Functions (from `TransactionsMultiHelper`)

| Function | Type | Description |
| :--- | :--- | :--- |
| `mergeDefaultCompanyValues` | `protected` | Merge all passed options with comprehensive default values (company, period, currency, extra fields...). |
| `prepareHeaderOptions` | `protected` | Prepare the header options array for `TransactionHeader` to pass to `AccountHelper::getObjHeader`. |
| `prepareDetailOptions` | `protected` | Prepare the detail options array for `TransactionsDetail` to pass to `AccountHelper::getObjDetail`. |
| `processHeaderPerson` | `protected` | Process the person object at the header level and convert it to `id` and `type`. |
| `preparePaperId` | `protected` | Prepare `paper_id` (auto-generate if `paper_id_auto = true`, clear value if `paper_id_allowed = false`). |
| `generatePaperId` | `protected` | Generate a `paper_id` based on the maximum value in the specified scope. |
| `getMaxPaperId` | `protected` | Get the largest `paper_id` recorded according to the given conditions. |
| `prepareDateAt` | `protected` | Prepare `date_at` (set to current date if `is_custome_date = false`). |

---

## 7. Currency and Account Processing Functions (from `TransactionsMultiHelper`)

| Function | Type | Description |
| :--- | :--- | :--- |
| `processCurrency` | `protected` | Process currency and exchange rate for each detail, with support for automatic conversion (`auto_convert_currency`). |
| `calculateTotals` | `protected` | Calculate debit and credit totals (both base and foreign) and determine whether currencies are mixed. |
| `applyAccountRules` | `protected` | Apply a set of dynamic rules to an account to check its type or properties (e.g., `modul_type = 'Boxe'`). |
| `compareAttribute` | `protected` | Compare an attribute value of an account with an operator (`=`, `LIKE`, `IN`, `REGEXP`...). |
| `checkAccountBalance` | `protected` | Check the balance of a given account using `AccountHelper::getBalances`. |

---

## 8. General Helper Functions (from `TransactionsMultiHelper`)

| Function | Type | Description |
| :--- | :--- | :--- |
| `handleException` | `protected` | Unified exception handling, returning a result array with debugging details in `debug` mode. |
| `fireHook` | `protected` | Execute custom `beforeValidate`/`afterValidate` functions and fire system events. |
| `isUseDefaultCompany` | `public` | Check whether the `use_default_company` setting is enabled. |
| `getCheckCompanysId` | `public` | Get the appropriate company ID (considering `use_default_company` settings). |
| `getCheckDepartmentsId` | `public` | Get the appropriate department ID. |

---

## 9. Quick Summary of Common Use Cases

| Scenario | Functions Used |
| :--- | :--- |
| Create a simple entry (two parties) | `createSingle` or `createEntry` |
| Create a complex entry (more than two parties) | `createEntry` or `createJournalEntry` |
| Test an entry before saving | `testJournalEntry` |
| Add a student payment via bank | `addPayment` |
| Transfer balance to a courier | `createMonyDeliverys` / `createMonyDeliverysFromCompanys` |
| Process an order lifecycle (payment, delivery, commission) | `createPaymentFromOrders`, `createBillShopFromOrders`, `createBrokerageShopFromOrders`... |
| Check exchange differences for an entry | `checkExchangeDifferences` |
| Create an exchange difference entry | `createExchangeDifferenceEntry` |
| Search for entries with advanced filters | `getTransactionHeader` |
| Check balance before a purchase | `checkAllowPay` |

---

This index covers all available functions in the class and trait, with a precise and clear description for each. Refer to the detailed documentation of each function for deeper information on options and advanced usage.

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
