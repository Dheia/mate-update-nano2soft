# Documentation of the `TransactionsHelper` Class and `TransactionsMultiHelper` Trait – Accounting Operations Helper

## Overview

The `TransactionsHelper` class is located at `Tss\Accounts\Helpers\TransactionsHelper` and is primarily responsible for executing all accounting operations related to entries and financial transactions within the `Tss.Accounts` add‑on. The class acts as a comprehensive service layer for creating various types of accounting entries (simple and multi‑party), checking balances, managing payments, settling exchange differences, and other advanced operations.

Starting from version **1.0.38**, the class has been significantly enhanced by integrating the `TransactionsMultiHelper` trait, which added the ability to create multi‑party entries (`createJournalEntry`) with an integrated validation system, exchange difference support, account type checking, and control over extra fields on each side of the entry.

The class is characterised by all its methods being `static`, allowing them to be called directly without creating an object. It also adheres to returning a unified response array (`result array`) in most functions, making it easy to handle success and failure consistently.

---

## Unified Response Structure (Result Array)

Functions that perform write operations (creating entries, adding payments, etc.) return an array with the following structure:

| Key | Type | Description |
| :--- | :--- | :--- |
| `code` | `int` | Status code (`200` for success, `400` for error). |
| `status` | `bool` | Operation status (`true` for success, `false` for failure). |
| `message` | `string` | Descriptive, translatable message. |
| `error` | `string\|null` | Error message (if any). |
| `errors` | `array\|null` | Array of validation errors (if any). |
| `model` | `TransactionHeader\|null` | The created entry header object. |
| `data` | `mixed\|null` | Additional data (depending on context). |
| `input_data` | `array` | Input parameters after initial processing. |
| `process_data` | `array` | The data actually used during the operation. |
| `debug` | `array\|null` | Debug information (only appears in a development environment when `app.debug = true`). |

---

## 1. Basic Entry Creation Functions

### 1.1 `createSingle`

Creates a simple two‑party accounting entry (debit and credit). This is the basic function relied upon by most financial operations, such as adding an opening balance, transferring a balance, and making payments.

```php
public static function createSingle($options = []): array
```

#### Basic Options

| Key | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `companys_id` | `int` | (calculated automatically) | Company ID. |
| `departments_id` | `int` | (calculated automatically) | Department ID. |
| `cost_centers_id` | `int` | (calculated automatically) | Cost center ID. |
| `periods_id` | `int` | (calculated automatically) | Accounting period ID. |
| `date_at` | `Carbon/string` | `now()` | Entry date. |
| `transactions_type` | `int` | `4` | Transaction type. |
| `process_type` | `int/string` | `4` | Process type. |
| `patterns_id` | `int` | `4` | Pattern ID. |
| `notes` | `string` | `null` | General notes. |
| `type_header` | `string` | `null` | Entry header type (e.g., `BondsDay::class`). |
| `mony` | `float` | `0` | **Amount (required)**. |
| `debit_accounts_id` | `string` | **Required** | Debit account code. |
| `credit_accounts_id` | `string` | **Required** | Credit account code. |
| `debit_person` | `object` | `null` | Debit person object. |
| `debit_person_id` / `debit_person_type` | `int/string` | `null` | Debit person ID and type. |
| `credit_person` | `object` | `null` | Credit person object. |
| `credit_person_id` / `credit_person_type` | `int/string` | `null` | Credit person ID and type. |
| `debit_notes` | `string` | `null` | Debit side notes. |
| `credit_notes` | `string` | `null` | Credit side notes. |
| `default_notes` | `string` | `null` | Default note for the header (if `notes` is not provided). |

In addition to many other options to control extra fields (`extend_id`, `modul_type`, `ref_type`…), currencies (`currencys_id`, `rate`) and status (`status`, `is_active`).

#### Example

```php
$result = TransactionsHelper::createSingle([
    'mony'               => 1500,
    'debit_accounts_id'  => '2-2-1253010001', // treasury
    'credit_accounts_id' => '2-2-1231010001', // customer
    'notes'              => 'Cash deposit',
]);

if ($result['status']) {
    echo "Entry created successfully, transaction number: " . $result['model']->id;
}
```

---

### 1.2 `createEntry` (from `TransactionsMultiHelper`)

A unified entry creation function that automatically chooses between `createSingle` (if the number of parties ≤ 2) and `createJournalEntry` (if the number of parties > 2). You can force the use of `createJournalEntry` via the option `forceJournal => true`.

```php
public static function createEntry($options = [], $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$options` | `array` | Entry options; must contain `details` if multi‑party. |
| `$is_test_create` | `bool` | If `true`, the operation is rolled back (for testing purposes). |

#### Example

```php
// Multi‑party entry (3 parties)
$result = TransactionsHelper::createEntry([
    'date_at' => '2026-05-18',
    'notes'   => 'Settlement entry',
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 3000],
        ['accounts_id' => '2-2-4111010001', 'credit' => 2000],
    ],
]);
```

---

### 1.3 `createJournalEntry` (from `TransactionsMultiHelper`)

Creates a multi‑party journal entry (more than two parties) with advanced validation, currency handling, and exchange differences.

```php
public static function createJournalEntry($options = [], $is_test_create = false): array
```

This function supports a **huge number of options** to control every aspect of the entry. A summary of the most important groups is given below; the complete list is in the [createJournalEntry-config.md](./createJournalEntry-config.md) file.

#### Highlights of the option groups

- **Basic header fields**: `companys_id`, `departments_id`, `date_at`, `transactions_type`, `process_type`, `patterns_id`, `notes`, `type_header`…
- **Person at the header level**: `header_person_id`, `header_person_type`, `header_person`.
- **Party count limits**: `max_debit_entries`, `max_credit_entries`, `allow_multiple_debit`, `allow_multiple_credit`.
- **Person control per side**: `debit_person_allowed`, `debit_person_required`, `debit_person_allowed_types` (and their credit counterparts).
- **Account control per side**: `debit_account_allowed`, `debit_account_required`, `debit_default_account`.
- **Notes control per side**: `debit_notes_allowed`, `debit_notes_required`, `debit_default_notes`.
- **Amount limits**: total (`min_total_amount`, `max_total_amount`) and per side (`min_debit_amount`, `max_credit_amount`…).
- **Balance checking**: `debit_check_balance` with support for custom callback functions (`debit_balance_check_callback`).
- **Dynamic account type checking with rules**: `debit_check_account_types` and `debit_rules_account_types`.
- **Extra field control**: `header_extend_id_allowed`, `debit_batch_id_required`… (covers 7 fields).
- **Currencies**: `auto_convert_currency`, `handle_exchange_differences`, `exchange_difference_action`.
- **`paper_id` management**: `is_paper_id`, `paper_id_allowed`, `paper_id_auto`, `paper_id_check_duplicate`…
- **Others**: `disabled_rules`, `beforeValidate`, `afterValidate`.

#### Advanced example

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 2500],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1500],
        ['accounts_id' => '2-2-4111010001', 'credit' => 1000],
    ],
    'min_total_amount'       => 1000,
    'max_total_amount'       => 10000,
    'debit_check_account_types' => true,
    'debit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
    ],
    'handle_exchange_differences' => true,
]);
```

---

## 2. Specialised Financial Operations Functions

### 2.1 `createMonyDeliverys`

Adds an opening balance to a courier (Delivery) from the company. Used in delivery systems when activating courier accounts.

```php
public static function createMonyDeliverys($user, $mony = null): array
```

- `$user`: User object (or `Delivery` object).
- `$mony`: Amount (if not specified, read from the configuration `tss.accounts::create_mony_deliverys.mony`).

The debit and credit account settings and the transaction type are fetched from the configuration file (`tss.accounts::create_mony_deliverys`). The function checks that the courier does not already have an opening balance.

---

### 2.2 `createMonyDeliverysFromSyndical`

Transfers a balance to a courier from an agent (Syndical/Department). Checks the agent's balance before deducting.

```php
public static function createMonyDeliverysFromSyndical($user, $department, $mony, $currencys_id = null, $rate = null): array
```

- `$department`: The department (agent) object from whose account the amount will be deducted.
- `$mony`: Amount (subject to `min_mony` and `max_mony` limits from the configuration).
- `$currencys_id`, `$rate`: Currency and exchange rate.

---

### 2.3 `createMonyDeliverysFromCompanys`

Transfers a balance to a courier from the official company (requires backend user permissions).

```php
public static function createMonyDeliverysFromCompanys($user, $department, $mony, $currencys_id = null, $rate = null, $other_options = []): array
```

- `$other_options`: Additional options array such as `periods_id`, `notes`, `date_at`, `paper_id`…

---

### 2.4 `createDeductingMonyDeliverysFromCompanys`

Deducts an amount from a courier's account (e.g., app commission).

```php
public static function createDeductingMonyDeliverysFromCompanys($user, $mony, $currencys_id = null, $rate = null, $other_options = []): array
```

Supports additional options to customise notes and persons (`person_notes`, `debit_notes`, `credit_notes`…).

---

### 2.5 `addPayment`

Adds a financial payment from a third party (e.g., bank) to a specific account (e.g., student account). Widely used in school management systems.

```php
public static function addPayment($options = []): array
```

Its behaviour heavily depends on the settings in `tss.accounts::add_payment`, which control which side the person appears on and which accounts are used.

#### Example (bank payment to a student)

```php
$result = TransactionsHelper::addPayment([
    'mony'              => 500,
    'credit_accounts_id'=> '2-2-1232010001', // student's account
    'credit_person'     => $student,
    'notes'             => 'Payment via bank',
]);
```

---

### 2.6 Order‑related functions

A set of specialised functions for recording accounting entries linked to the order lifecycle in an e‑commerce system:

| Function | Description |
| :--- | :--- |
| `createBrokerageFromOrders` | Deduct delivery commission from the courier. |
| `createPaymentFromOrders` | Record an order payment entry (from the customer to the company/courier account). |
| `createBillShopFromOrders` | Record the store's purchase value (or the courier in the case of online payment). |
| `createBrokerageShopFromOrders` | Deduct app commission from the store. |
| `createShippingTotalFromOrders` | Record the delivery cost entry for the courier (in the case of online payment). |
| `createTipAmountFromOrders` | Record the tip entry for the courier. |
| `createPaymentMonyDeliverysFromCompanys` | (Legacy) Previous attempt at a comprehensive payment entry for an order. |

All of these functions accept an `Order` object, calculate the appropriate amounts (based on `cart_total`, `shipping_total`, `order_total` …), and pass them to `createSingle` or `createDeductingMonyDeliverysFromCompanys`. They also check that an entry for the same operation does not already exist (to prevent duplication).

---

## 3. Validation and Query Functions

### 3.1 `checkAllowPay`

Checks whether the balance of a given account is sufficient to carry out a purchase (or any operation that requires a credit balance).

```php
public static function checkAllowPay($options = []): array
```

| Option | Description |
| :--- | :--- |
| `total` | The total amount to be checked. |
| `code` | The account code to be checked. |
| (same as `getBalances` options) | For filtering (department, period, currency…). |

Throws an exception if the balance is insufficient.

#### Example

```php
TransactionsHelper::checkAllowPay([
    'total' => 250,
    'code'  => '2-2-1231010001',
]);
// If no exception is thrown, the balance is sufficient.
```

---

### 3.2 `checkTransactions`

Searches for a specific entry (transaction header) and verifies its existence.

```php
public static function checkTransactions($options = []): array
```

Mainly used to check for the existence of an entry before performing updates or cancellations. Returns the unified array with `model` (the entry object) and `transactions_id`.

---

### 3.3 `getTransactionHeader`

Advanced query on transaction headers (`TransactionHeader`) with comprehensive filtering.

```php
public static function getTransactionHeader($options = [], $isException = false): array
```

Supports a large set of filtering, ordering, and grouping options, searching by:

- Entry ID (`id`, `is_force_id`)
- Department, period, transaction type, process type, paper document type.
- Advanced fields: `extend_id`, `modul_type`, `ref_type`, `batch_type`, `batch_id`
- Search through details (`accounts_id`, `person_id`, `person_type`, `currencys_id`)
- Filter by user type (`users_ref_type`)
- Date ranges (using `AccountHelper::getQueryDate`)
- Return options: `is_query`, `is_first`, `is_collection`, `is_paginator`
- Special ordering by likes, views, etc. (same as `AccountsApi` behaviour)

#### Example

```php
$result = TransactionsHelper::getTransactionHeader([
    'transactions_type' => [1,2], // specific types
    'departments_id'    => 3,
    'person_id'         => 42,
    'person_type'       => 'RainLab\User\Models\User',
    'is_paginator'      => true,
    'per_page'          => 20,
]);
$paginator = $result['data'];
```

---

## 4. Advanced Trait Functions (`TransactionsMultiHelper`)

### 4.1 `validateJournalEntry`

Validates the data of a multi‑party journal entry without creating it. Returns the processed details.

```php
public static function validateJournalEntry($options = []): array
```

It checks:
- The existence of `details`.
- Entry balance (total debit = total credit).
- Foreign currency balance (if currencies are mixed).
- Party count limits (`multiplicity`).
- Amount limits.
- Balances.

#### Example

```php
$validation = TransactionsHelper::validateJournalEntry([
    'details' => [...],
]);
// If no exception is thrown, the entry is valid.
```

---

### 4.2 `testJournalEntry`

Fully simulates the creation of a multi‑party journal entry with detailed analysis (without saving to the database).

```php
public static function testJournalEntry($options = []): array
```

Returns the unified array with an additional `analysis` key containing:
- `header_prepared`, `details_prepared`
- `totals`, `currency_analysis`
- `rule_violations`, `warnings`
- `test_passed` (did it pass the test?)

#### Example

```php
$test = TransactionsHelper::testJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 1000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1000],
    ],
]);
print_r($test['analysis']);
```

---

### 4.3 `checkExchangeDifferences`

Checks an existing journal entry to determine whether an exchange difference entry is needed, with the option to automatically create the entry.

```php
public static function checkExchangeDifferences($input, $options = []): array
```

| Parameter | Description |
| :--- | :--- |
| `$input` | Entry ID (int) or a `TransactionHeader` object. |
| `$options` | `threshold`, `action` (`'entry'`, `'event'`, `'none'`), `force_check`, `relay_date`, `exchange_difference_account`. |

#### Example

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'threshold' => 0.01,
    'action'    => 'entry',
]);
if ($result['exchange_entry']) {
    echo "Exchange difference entry: " . $result['exchange_entry']->id;
}
```

---

### 4.4 `createExchangeDifferenceEntry`

Creates an exchange difference entry resulting from a change in the exchange rate between the entry date and the posting date. Usually called internally.

```php
public static function createExchangeDifferenceEntry($originalHeader, $originalDetail, $difference, $officialRate, $originalOptions = []): array
```

---

### 4.5 Selected internal helper functions

| Function | Description |
| :--- | :--- |
| `applyAccountRules` | Applies a set of rules to an account to check its type (supports `AND`/`OR` and multiple operators). |
| `compareAttribute` | Compares a value with an operator (`=`, `LIKE`, `IN`, `REGEXP`…). |
| `validateSingleDetail` | Comprehensive validation of a single detail (account, person, notes, extra fields). |
| `processCurrency` | Processes the currency and rate for each detail, with automatic conversion support. |
| `checkAccountBalance` | Checks an account's balance using `AccountHelper::getBalances`. |
| `mergeDefaultCompanyValues` | Merges the default values for company, period, and currencies. |
| `prepareHeaderOptions` / `prepareDetailOptions` | Prepares the header and detail options arrays. |
| `validateMultiplicity` / `validateAmountLimits` / `validateBalance` | Entry constraint checks. |
| `preparePaperId` / `validatePaperId` / `generatePaperId` | Manages `paper_id` (auto‑generation, duplicate checking). |
| `prepareDateAt` / `validateDateAtRequired` | Manages the entry date. |
| `fireHook` | Executes hooks (`beforeValidate`, `afterValidate`). |

---

## 5. Document Numbering Function `seedDocsNumber`

Numbers old records in the `tss_accounts_transaction_headers` table that do not have a `docs_number`.

```php
public static function seedDocsNumber($options = []): array
```

---

## 6. Events

The class and trait fire the following events, allowing custom logic to be hooked:

| Event | Parameters | Description |
| :--- | :--- | :--- |
| `tss.accounts.afterAddOpeningBalanceDelivery` | `$options, $header, $user` | After adding an opening balance to a courier. |
| `tss.accounts.afterAddMonyDelivery` | `$options, $header, $user` | After transferring a balance to a courier from the company. |
| `tss.accounts.afterBrokerage` | `$order, $state, $options` | After deducting a commission from a courier or store. |
| `tss.accounts.beforeJournalEntry` | `$options, $details` | Before creating a multi‑party entry. |
| `tss.accounts.afterJournalEntry` | `$header, $options, $details` | After creating a multi‑party entry. |
| `tss.accounts.exchangeDifference` | `$header, $detail, $difference, $officialRate, $options` | When an exchange difference is detected (if `action='event'`). |
| `tss.accounts.beforeValidate` / `tss.accounts.afterValidate` | `$options, $details` | From the trait's hooks. |

---

## 7. Dependencies

- `Tss\Accounts\Helpers\AccountHelper`
- `Tss\Accounts\Helpers\PersonHelper`
- `Tss\Accounts\Models\*` (TransactionHeader, TransactionsDetail, Account, Period, Currency, CostCenter…)
- `Tss\Basic\Helpers\BasicHelper`
- `Tss\Currency\Facades\Currency` (to access exchange rates)
- `Illuminate\Support\Facades\DB` (for transactions and raw queries)
- `Illuminate\Support\Facades\Event`
- `Carbon\Carbon`
- `Config` (to read settings)

---

## 8. Important Notes

1. **Backward compatibility**: All functions from before version 1.0.38 still work with the same signature. Only `use TransactionsMultiHelper;` was added, along with initialising the result array (`$result = [];`) to avoid any issues.
2. **Test support**: The `createJournalEntry` and `createEntry` functions support the `is_test_create` mode, which rolls back the operation after validation, making unit tests easy.
3. **User permissions**: These functions do not directly check permissions; that is assumed to be done at higher layers (controllers). Some functions, such as `createMonyDeliverysFromCompanys`, perform some checks (e.g., `hasAccess`).
4. **`paper_id` management**: The trait now supports advanced options for controlling the paper document number; see `preparePaperId` and the related options.
5. **Account type checking**: The new feature (`debit_rules_account_types`/`credit_rules_account_types`) allows you to prevent entries on accounts that do not belong to certain types (e.g., “treasury only”). See the usage examples in the `createJournalEntry-config.md` file.

---

## 9. Integrated Examples

### E‑commerce scenario – recording a complete payment entry when an order is completed

```php
$order = Order::find(100);

// 1. Order payment entry (from customer to company or courier, depending on the payment method)
$paymentResult = TransactionsHelper::createPaymentFromOrders($order);

// 2. If it's a store, record the store's purchase value
$billResult = TransactionsHelper::createBillShopFromOrders($order);

// 3. If it's a store and a commission applies, record the app commission deduction
$brokerageResult = TransactionsHelper::createBrokerageShopFromOrders($order);

// 4. If the payment method is online, record the delivery cost for the courier
$shippingResult = TransactionsHelper::createShippingTotalFromOrders($order);
```

### Checking exchange differences for all unposted entries

```php
$unrelayedHeaders = TransactionHeader::where('is_relay', false)->get();
foreach ($unrelayedHeaders as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, ['action' => 'entry']);
    if ($result['exchange_entry']) {
        Log::info("Created exchange difference entry for header {$header->id}");
    }
}
```

---

## Conclusion

The `TransactionsHelper` class together with the `TransactionsMultiHelper` trait form the backbone of all accounting operations on the NanoSoft platform. Thanks to the unified functions, comprehensive validation, and advanced support for currencies and exchange differences, developers can build complex financial systems with ease and security. We hope this documentation serves as a comprehensive guide to help you make the most of these powerful tools.

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
