# Comprehensive Documentation of the `createJournalEntry` Function

## Overview

The `createJournalEntry` function resides within the `TransactionsMultiHelper` trait used by the `TransactionsHelper` class. This function allows the creation of multi‑leg journal entries with a comprehensive validation system, advanced currency handling, balance checks, dynamic account type rules, and the ability to automate exchange difference processing. It represents the core foundation for creating all types of advanced entries in the NanoSoft financial system.

---

## Function Signature

```php
public static function createJournalEntry(array $options = [], bool $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$options` | `array` | An array of options for creating the entry (all keys are optional except `details`). |
| `$is_test_create` | `bool` | If `true`, the operation is executed within a transaction and then rolled back (for testing purposes). |

---

## Function Execution Flow

To understand how each option affects the function's behaviour, it is important to know the internal sequence of operations:

1.  **Validate the input**: Check that `$options` is a non‑empty array.
2.  **`mergeDefaultCompanyValues`** – Merge all passed options with the comprehensive default values.
3.  **`preparePaperId`** – Prepare `paper_id` (auto‑generate if `paper_id_auto = true`, clear the value if `paper_id_allowed = false`).
4.  **`validatePaperIdRequired`** – Check if `paper_id` is required.
5.  **`prepareDateAt`** – Prepare `date_at` (set to the current date if `is_custome_date = false`).
6.  **`validatePaperId`** – Check for duplicate `paper_id` if required.
7.  **`validatePeriod`** – Check the accounting period's validity (the date is within the period and the period is open).
8.  **`fireHook('beforeValidate', ...)`** – Execute the pre‑validation hook.
9.  **`validateJournalEntry`** – Comprehensive validation of the entry (balance, currencies, multiplicity, limits, balances). Returns the processed details.
10. **`fireHook('afterValidate', ...)`** – Execute the post‑validation hook.
11. **`calculateTotals`** – Calculate the base and foreign debit and credit totals.
12. **Fire the `tss.accounts.beforeJournalEntry` event**.
13. **`prepareHeaderOptions`** – Prepare the header options.
14. **Create and save the header object (`TransactionHeader`)**.
15. **Save the details** – For each element in `details`, call `prepareDetailOptions` and then `AccountHelper::getObjDetail` and save it.
16. **`handleExchangeDifferences`** – If `handle_exchange_differences = true`, check for exchange differences and create a difference entry (or fire an event).
17. **`Db::commit()`** (on success) or **`Db::rollBack()`** (on error or in test mode).

---

## Complete Options Array (`$options`)

### 1. Basic Header Fields

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 1.1 | `companys_id` | `int`/`string`/`null` | Calculated automatically from `getCheckCompanysId` | Company ID. |
| 1.2 | `departments_id` | `int`/`string`/`null` | Calculated automatically from `getCheckDepartmentsId` | Department (branch) ID. |
| 1.3 | `cost_centers_id` | `int`/`string`/`null` | Calculated automatically from `CostCenter::getPrimary` | Cost center ID. |
| 1.4 | `periods_id` | `int`/`string`/`null` | Calculated automatically from `Period::getPrimary` | Accounting period ID. |
| 1.5 | `paper_id` | `mixed`/`null` | `null` | Paper document number (text or numeric). Controlled by `is_paper_id`. |
| 1.6 | `employees_id` | `int`/`null` | `null` | ID of the employee responsible for the entry. |
| 1.7 | `date_at` | `Carbon`/`string`/`null` | `Carbon::now()` | Entry date. Controlled by `is_custome_date`. |
| 1.8 | `transactions_type` | `int` | `4` | Transaction type (from the `tss_accounts_transactions_types` table). |
| 1.9 | `process_type` | `int`/`string` | `4` | Process type (e.g., `cash` or `credit`). |
| 1.10| `patterns_id` | `int` | `4` | Accounting pattern ID (`ConstraintsPattern`). |
| 1.11| `notes` | `string`/`null` | `null` | General notes on the entry header. |
| 1.12| `default_notes` | `string`/`null` | `null` | Default notes used if `notes` is empty. |
| 1.13| `type_header` | `string`/`null` | `null` | Header type (e.g., `BondsDay::class`). |
| 1.14| `status` | `string` | `'active'` | Entry status (`active`, `inactive`, ...). |
| 1.15| `is_active` | `bool` | `true` | Is the entry active? |
| 1.16| `is_relay` | `bool` | `false` | Has it been posted? |
| 1.17| `user_relay_id` | `int`/`null` | `null` | ID of the user who posted it. |
| 1.18| `relay_date_at` | `Carbon`/`null` | `null` | Posting date. |

### 2. Person at the Header Level (`Header Person`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 2.1 | `header_person_id` | `int`/`null` | `null` | Person ID associated with the entry header. |
| 2.2 | `header_person_type` | `string`/`null` | `null` | Person type (Morph Class). |
| 2.3 | `header_person` | `object`/`null` | `null` | Person object (from which `id` and `type` are extracted automatically). |

### 3. `paper_id` Settings (Paper Document)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 3.1 | `is_paper_id` | `bool` | `false` | Is the `paper_id` system enabled? If `false`, all following settings are ignored. |
| 3.2 | `paper_id_allowed` | `bool` | `true` | Is the user allowed to enter a `paper_id` value? If `false`, any entered value is removed. |
| 3.3 | `paper_id_auto` | `bool` | `false` | Is a `paper_id` generated automatically if not entered? |
| 3.4 | `paper_id_required` | `bool` | `false` | Is `paper_id` required? (checked in `validatePaperIdRequired`). |
| 3.5 | `paper_id_check_duplicate` | `bool` | `true` | Do we check for duplicate `paper_id`? |
| 3.6 | `paper_id_duplicate_scope` | `array` | `[]` | Duplicate check scope: array of `'departments_id'`, `'periods_id'`, `'transactions_type'`. |
| 3.7 | `validat_paper_id` | `string` | `'no_rapet'` | (for compatibility) Old duplicate checking method. |
| 3.8 | `type_rapet` | `array` | `[]` | (for compatibility) Old duplicate check conditions (e.g., `['rapet_other_periods', 'rapet_other_type']`). |

### 4. `date_at` Settings (Date)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 4.1 | `is_custome_date` | `bool` | `false` | Is the user allowed to enter the date? If `false`, it is set to `now()`. |
| 4.2 | `date_at_auto` | `bool` | `false` | (for compatibility) Automatically generate the date if not entered. |
| 4.3 | `date_at_required` | `bool` | `false` | Is the date required? |

### 5. Currencies and Exchange Differences

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 5.1 | `main_currencys_id` | `string`/`null` | Calculated automatically | The company's base currency. |
| 5.2 | `currencys_id` | `string`/`null` | Calculated automatically | The entry's currency. |
| 5.3 | `rate` | `float` | `1` | Exchange rate (used if the currency is foreign). |
| 5.4 | `auto_convert_currency` | `bool` | `false` | Automatically convert amounts from the foreign currency to the base currency. |
| 5.5 | `handle_exchange_differences` | `bool` | `false` | Enable checking and creating exchange differences after saving the entry. |
| 5.6 | `exchange_difference_action` | `string` | `'event'` | Action when differences exist: `'entry'` (create an entry), `'event'` (fire an event), `'none'` (ignore). |

### 6. Party Count Limits (`Multiplicity`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 6.1 | `max_debit_entries` | `int`/`null` | `null` | Maximum number of debit parties (`null` = unlimited). |
| 6.2 | `max_credit_entries` | `int`/`null` | `null` | Maximum number of credit parties. |
| 6.3 | `allow_multiple_debit` | `bool` | `true` | Allow multiple debit parties. |
| 6.4 | `allow_multiple_credit` | `bool` | `true` | Allow multiple credit parties. |

### 7. Person Settings per Side

#### 7.1 Debit Side (`Debit Person`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 7.1.1 | `debit_person_allowed` | `bool` | `true` | Allow linking a person to the debit side. |
| 7.1.2 | `debit_person_required` | `bool` | `false` | Is a person required on the debit side? |
| 7.1.3 | `debit_person_allowed_types` | `array`/`null` | `null` | Allowed person types (array of Morph Classes). `null` = all types. |

#### 7.2 Credit Side (`Credit Person`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 7.2.1 | `credit_person_allowed` | `bool` | `true` | Allow linking a person to the credit side. |
| 7.2.2 | `credit_person_required` | `bool` | `false` | Is a person required on the credit side? |
| 7.2.3 | `credit_person_allowed_types` | `array`/`null` | `null` | Allowed person types for the credit side. |

### 8. Account Settings per Side

#### 8.1 Debit Side (`Debit Account`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 8.1.1 | `debit_account_allowed` | `bool` | `true` | Allow specifying a debit account. |
| 8.1.2 | `debit_account_required` | `bool` | `true` | Is a debit account required? |
| 8.1.3 | `debit_default_account` | `string`/`null` | `null` | Default debit account (account code) if not specified. |

#### 8.2 Credit Side (`Credit Account`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 8.2.1 | `credit_account_allowed` | `bool` | `true` | Allow specifying a credit account. |
| 8.2.2 | `credit_account_required` | `bool` | `true` | Is a credit account required? |
| 8.2.3 | `credit_default_account` | `string`/`null` | `null` | Default credit account. |

### 9. Notes Settings per Side

#### 9.1 Debit Side (`Debit Notes`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 9.1.1 | `debit_notes_allowed` | `bool` | `true` | Allow entering a note for the debit side. |
| 9.1.2 | `debit_notes_required` | `bool` | `false` | Is a note required for the debit side? |
| 9.1.3 | `debit_default_notes` | `string`/`null` | `null` | Default note for the debit side. |

#### 9.2 Credit Side (`Credit Notes`)

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 9.2.1 | `credit_notes_allowed` | `bool` | `true` | Allow entering a note for the credit side. |
| 9.2.2 | `credit_notes_required` | `bool` | `false` | Is a note required for the credit side? |
| 9.2.3 | `credit_default_notes` | `string`/`null` | `null` | Default note for the credit side. |

### 10. Amount Limits

#### 10.1 Entry Total

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 10.1.1 | `min_total_amount` | `float`/`null` | `null` | Minimum total amount. |
| 10.1.2 | `max_total_amount` | `float`/`null` | `null` | Maximum total amount. |
| 10.1.3 | `check_min_total` | `bool` | `true` | Enable checking the minimum total amount. |
| 10.1.4 | `check_max_total` | `bool` | `true` | Enable checking the maximum total amount. |

#### 10.2 Debit Side

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 10.2.1 | `check_debit_min` | `bool` | `false` | Enable checking the minimum debit total. |
| 10.2.2 | `check_debit_max` | `bool` | `false` | Enable checking the maximum debit total. |
| 10.2.3 | `min_debit_amount` | `float`/`null` | `null` | Minimum debit total. |
| 10.2.4 | `max_debit_amount` | `float`/`null` | `null` | Maximum debit total. |

#### 10.3 Credit Side

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 10.3.1 | `check_credit_min` | `bool` | `false` | Enable checking the minimum credit total. |
| 10.3.2 | `check_credit_max` | `bool` | `false` | Enable checking the maximum credit total. |
| 10.3.3 | `min_credit_amount` | `float`/`null` | `null` | Minimum credit total. |
| 10.3.4 | `max_credit_amount` | `float`/`null` | `null` | Maximum credit total. |

### 11. Balance Checking

#### 11.1 Debit Side

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 11.1.1 | `debit_check_balance` | `bool` | `false` | Enable checking the debit account's balance before execution. |
| 11.1.2 | `debit_balance_check_callback` | `callable`/`null` | `null` | Custom balance check function (receives `$account`, `$amount`, `$balance`, `$personId`, `$personType`, `$context`). |

#### 11.2 Credit Side

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 11.2.1 | `credit_check_balance` | `bool` | `false` | Enable checking the credit account's balance. |
| 11.2.2 | `credit_balance_check_callback` | `callable`/`null` | `null` | Custom balance check function for the credit side. |

### 12. Dynamic Account Type Checking with Rules

#### 12.1 Debit Side

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 12.1.1 | `debit_check_account_types` | `bool` | `false` | Enable checking the account type on the debit side. |
| 12.1.2 | `debit_rules_account_types` | `array`/`null` | `null` | Account type rules (see the rules explanation section below). |

#### 12.2 Credit Side

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 12.2.1 | `credit_check_account_types` | `bool` | `false` | Enable checking the account type on the credit side. |
| 12.2.2 | `credit_rules_account_types` | `array`/`null` | `null` | Rules for the credit side. |

### 13. Extra Fields

The extra fields are: `extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`.
For each of these fields, there are three sets of settings: at the header level (`header`), at the debit detail level (`debit`), and at the credit detail level (`credit`). Each set contains three options:

| # | Key (pattern) | Type | Default | Description |
|---|--------------|------|---------|-------------|
| 13.1 | `header_{field}_allowed` | `bool` | `true` | Allow using the field in the header. |
| 13.2 | `header_{field}_required` | `bool` | `false` | Is the field required in the header? |
| 13.3 | `header_{field}_default` | `mixed` | `null` | Default value for the field in the header. |
| 13.4 | `debit_{field}_allowed` | `bool` | `true` | Allow using the field on the debit side. |
| 13.5 | `debit_{field}_required` | `bool` | `false` | Is the field required on the debit side? |
| 13.6 | `debit_{field}_default` | `mixed` | `null` | Default value for the field on the debit side. |
| 13.7 | `credit_{field}_allowed` | `bool` | `true` | Allow using the field on the credit side. |
| 13.8 | `credit_{field}_required` | `bool` | `false` | Is the field required on the credit side? |
| 13.9 | `credit_{field}_default` | `mixed` | `null` | Default value for the field on the credit side. |

### 14. Additional Settings

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 14.1 | `auto_generate_custom_name` | `bool` | `true` | Automatically generate a custom name for the person if not passed. |
| 14.2 | `beforeValidate` | `callable`/`null` | `null` | Function executed before data validation (receives `&$options`, `&$details`). |
| 14.3 | `afterValidate` | `callable`/`null` | `null` | Function executed after validation (receives `&$options`, `&$details`). |
| 14.4 | `disabled_rules` | `string[]` | `[]` | Names of rules to disable. Supported values: `'balance_check'`, `'multiplicity'`, `'amount_limits'`. |

---

## Structure of `details`

The `details` key is the **only required key** for creating a multi‑party entry. It contains an array of detail elements, each representing a single accounting party.

### Structure of a single `details` element

| Key | Type | Description |
|-----|------|-------------|
| `accounts_id` | `string` | **Required**. Account code (e.g., `'2-2-1253010001'`). |
| `debit` | `float` | Debit amount (must be > 0 if this is a debit party, otherwise 0). |
| `credit` | `float` | Credit amount (must be > 0 if this is a credit party, otherwise 0). |
| `notes` | `string`/`null` | A note specific to this party (optional). |
| `person_id` | `int`/`null` | Person ID associated with the party (optional). |
| `person_type` | `string`/`null` | Person type (Morph Class). |
| `person` | `object`/`null` | Person object (from which `id` and `type` are extracted automatically). |
| `custom_name_accounts` | `string`/`null` | A custom name for the account (optional). |
| `currencys_id` | `string`/`null` | The currency of this party (if different from the header currency). |
| `main_currencys_id` | `string`/`null` | The base currency (usually left to be determined automatically). |
| `rate` | `float`/`null` | The exchange rate for this party (optional). |
| `extend_id` | `mixed`/`null` | (optional) External ID. |
| `modul_type` | `mixed`/`null` | (optional) Module type. |
| `ref_type` | `mixed`/`null` | (optional) Reference type. |
| `ref_key_name` | `mixed`/`null` | (optional) Reference key name. |
| `ref_key_values` | `mixed`/`null` | (optional) Reference key value. |
| `batch_type` | `mixed`/`null` | (optional) Batch type. |
| `batch_id` | `mixed`/`null` | (optional) Batch ID. |

**Important notes:**
- The sum of `debit` values across all details must equal the sum of `credit` values (with a tolerance of 0.001).
- If `auto_convert_currency = true` and the detail's currency differs from the base currency, the amounts are automatically converted and the original values are stored in `debit_alien` and `credit_alien`.
- Both `debit` and `credit` cannot be > 0 in the same detail.

---

## Explanation of Helper Functions Related to the Options

### `applyAccountRules` and Account Type Checking

When `debit_check_account_types` or `credit_check_account_types` is enabled, the `applyAccountRules` function is called, which compares the account's properties against a set of rules. The rules support two structures:

**a. Simplified format (automatic AND):**
```php
$rules = [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
];
```

**b. Advanced format with a logical operator:**
```php
$rules = [
    'operator' => 'OR',   // or 'AND'
    'rules'    => [
        ['field' => 'reports', 'operator' => '=', 'value' => '1'],
        ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
    ],
];
```

**Supported fields:** Any column from the `tss_accounts_accounts` table (`reports`, `final_account_id`, `modul_type`, `account_normal`, `groups_accounts_id`, `ref_type`, `person_type`, `account_number`, `parent_id`, `level`, `main_sub`, and others).

**Supported operators:** `=`, `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.

### Balance Check Callback Functions

The `debit_balance_check_callback` and `credit_balance_check_callback` functions receive six parameters:

| # | Parameter | Type | Description |
|---|-----------|------|-------------|
| 1 | `$account` | `Account` | The account object being checked. |
| 2 | `$amount` | `float` | The requested amount (debit or credit). |
| 3 | `$balance` | `float` | The account's current balance (according to the context). |
| 4 | `$personId` | `int`/`null` | The person ID associated with the detail. |
| 5 | `$personType` | `string`/`null` | The person type. |
| 6 | `$context` | `array` | The general entry options (the same `$options`). |

If a `callback` is passed, it is called **instead of** the default check. The function can throw an `ApplicationException` to prevent the entry, or simply `return` to allow it.

### `handleExchangeDifferences`

Only works if `handle_exchange_differences = true` is set. For each detail with a non‑base currency, it calculates the difference between the `rate` used and the official rate at `relay_date`. Based on `exchange_difference_action`:

- `'entry'`: Create a new exchange difference entry using `createExchangeDifferenceEntry`.
- `'event'`: Fire the `tss.accounts.exchangeDifference` event.
- `'none'`: Do nothing.

---

## Comprehensive Practical Examples

### Example 1: Simple daily entry (two parties)

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 5000],
    ],
    'notes' => 'Cash deposit',
    'type_header' => BondsDay::class,
]);

if ($result['status']) {
    echo "Entry created successfully, entry number: " . $result['model']->id;
}
```

### Example 2: Multi‑party entry with persons and account type checking

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-1253010001',
            'debit' => 3000,
            'person' => $cashier, // cashier object
            'notes' => 'Cash received'
        ],
        [
            'accounts_id' => '2-2-1231010001',
            'credit' => 2000,
            'person' => $customer,
        ],
        [
            'accounts_id' => '2-2-4111010001',
            'credit' => 1000,
            'notes' => 'Service revenue'
        ],
    ],
    'debit_check_account_types' => true,
    'debit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
    ],
    'credit_person_required' => true,
]);

if (!$result['status']) {
    echo "Creation failed: " . $result['error'];
}
```

### Example 3: Creating an entry with amount limits and a balance check with a `Callback`

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1231010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1252010001', 'credit' => 5000],
    ],
    'min_total_amount' => 100,
    'max_total_amount' => 50000,
    'debit_check_balance' => true,
    'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
        $creditLimit = 2000; // credit limit
        if ($balance + $creditLimit < $amount) {
            throw new ApplicationException("Insufficient balance for account {$account->code} even with the credit limit.");
        }
    },
]);
```

### Example 4: Using `beforeValidate` to automatically modify details

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 1500],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1500],
    ],
    'beforeValidate' => function(&$options, &$details) {
        // Add 15% tax to the debit amount
        foreach ($details as &$d) {
            if ($d['debit'] > 0) {
                $d['debit'] = round($d['debit'] * 1.15, 2);
            }
        }
    },
]);
```

### Example 5: Creating an entry with a foreign currency and handling exchange differences

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-1253010001',
            'debit' => 1000,
            'currencys_id' => 'USD', // foreign currency
            'rate' => 3.67,
        ],
        [
            'accounts_id' => '2-2-1231010001',
            'credit' => 1000,
            'currencys_id' => 'USD',
            'rate' => 3.67,
        ],
    ],
    'main_currencys_id' => 'SAR',
    'currencys_id' => 'USD',
    'auto_convert_currency' => true, // convert amounts to SAR
    'handle_exchange_differences' => true,
    'relay_date_at' => '2026-06-01', // different posting date
    'exchange_difference_action' => 'entry',
]);

if ($result['status']) {
    echo "Entry created. If exchange differences exist, an additional entry was created.";
    // $result['exchange_entry'] might contain the difference entry if it was created via checkExchangeDifferences
}
```

### Example 6: Using `disabled_rules` to disable some checks

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 5000],
    ],
    'disabled_rules' => ['balance_check', 'multiplicity'], // disable balance and party count checks
    'debit_check_balance' => true, // will be ignored because of disabled_rules
]);
```

---

## Testing Mode

You can use the `$is_test_create = true` parameter to simulate the entire operation and then roll it back. It returns the entry object with `test_mode => true`.

```php
$test = TransactionsHelper::createJournalEntry([
    'details' => [...],
], true);

if ($test['status']) {
    echo "The entry is valid and can be created.";
    // $test['model'] contains the object that would have been saved
    // No changes were made to the database
}
```

---

## Response Structure

| Key | Type | Description |
|-----|------|-------------|
| `code` | `int` | `200` for success, `400` for failure. |
| `status` | `bool` | Operation status. |
| `message` | `string` | Descriptive message. |
| `error` | `string`/`null` | Error message. |
| `errors` | `array`/`null` | Validation error details. |
| `model` | `TransactionHeader`/`null` | The created entry header object. |
| `input_data` | `array` | The input options. |
| `process_data` | `array` | The actual data used (after merging). |
| `debug` | `array`/`null` | Debug information (in a `debug` environment). |

---

## Detailed Currency Handling Mechanism

### Automatic Conversion (`auto_convert_currency`)
When `auto_convert_currency = true` is set, the helper function `processCurrency` (called inside `validateSingleDetail`) performs the following steps:

1.  **Determine the base currency**: From `$context['main_currencys_id']` or `CurrencyHelper::primaryCode()`.
2.  **Determine the detail's currency**: If `currencys_id` exists in the detail it is used; otherwise, the header currency (`$context['currencys_id']`) is used, then the base currency.
3.  **If the currency is foreign**: The entered `debit`/`credit` values are copied to `debit_alien`/`credit_alien`, and then the base values are converted by multiplying by `rate`:  
    `$debit = $debit * $rate;`
4.  **If the currency is base or `auto_convert_currency = false`**: No conversion takes place, and the values are left as they are. (In the case of `false` and a foreign currency, it is assumed that the model's `beforeSave` will handle the conversion based on `rate` and `debit_alien`/`credit_alien`).

**Practical example:**
```php
// Creating an entry in USD (foreign) with automatic conversion enabled
$options = [
    'main_currencys_id' => 'SAR',
    'currencys_id'      => 'USD',
    'auto_convert_currency' => true,
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 1000, 'rate' => 3.75],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1000, 'rate' => 3.75],
    ],
];
// Result in the database:
// debit = 1000 * 3.75 = 3750 SAR,  debit_alien = 1000 USD
// credit = 3750 SAR, credit_alien = 1000 USD
```

### Exchange Difference Handling (`handleExchangeDifferences`)
This feature works after the entry has been completely saved and checks for a `relay_date_at` different from `date_at`. For each foreign‑currency detail:

1.  Calls `CurrencyHelper::getRate` to obtain the official rate at `relay_date`.
2.  Calculates the difference: `$difference = $alienAmount * ($officialRate - $originalRate)`.
3.  If the difference exceeds the `threshold` (default 0.001), an action is taken based on `exchange_difference_action`:
    *   `'entry'`: Create a new exchange difference entry via `createExchangeDifferenceEntry`.
    *   `'event'`: Fire the `tss.accounts.exchangeDifference` event for external processing.
    *   `'none'`: Nothing.

You can also independently check differences for any existing entry using the public function `TransactionsHelper::checkExchangeDifferences`.

---

## Integration with Default Values (`mergeDefaultCompanyValues`)

This function is responsible for filling any missing options with intelligent default values. It relies on:

*   **General settings**: `tss.accounts::use_default_company` controls whether the company and department default values are forced.
*   **Helper functions**: `getCheckCompanysId`, `getCheckDepartmentsId`, `Period::getPrimary`, `Currency::getPrimary` to extract values from the database based on context.
*   **A constants array**: Contains all the above‑mentioned keys with their default values as described.

**Important point:** You can customise these values for your project by creating a settings file (e.g., `config/nano/accountsapi.php`) containing a `journal_entry_defaults` key and passing your custom values. They are merged with the default values inside `createJournalEntry` (it is preferable to merge them before calling).

---

## Analysis and Testing Functions

### `testJournalEntry`
To test an entry before actually creating it, use `testJournalEntry`:

```php
$testResult = TransactionsHelper::testJournalEntry($options);
print_r($testResult['analysis']);
/*
[
    'header_prepared' => [...],
    'details_prepared' => [...],
    'totals' => ['total_debit_base' => 5000, 'total_credit_base' => 5000, ...],
    'currency_analysis' => ['SAR' => ['debit' => 5000, 'credit' => 5000]],
    'test_passed' => true,
    'rule_violations' => [],
    'warnings' => [],
]
*/
```

### `validateJournalEntry`
To validate without creating (used internally and also externally):

```php
try {
    $validation = TransactionsHelper::validateJournalEntry($options);
    // The entry is valid
} catch (ApplicationException $e) {
    echo $e->getMessage();
}
```

---

## Full Management of Extra Fields

The extra fields (`extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`) are a powerful tool for linking entries to other business units (orders, invoices, employees). The system allows controlling their presence and requirement at three levels: **entry header**, **debit details**, **credit details**.

**Example: Making `batch_id` required in the header and preventing `modul_type` in the debit details:**

```php
$options = [
    'details' => [...],
    // header: batch_id required
    'header_batch_id_required' => true,
    'batch_id' => 'BATCH-001',
    // debit: modul_type not allowed
    'debit_modul_type_allowed' => false,
];
```
If any debit detail tries to send `modul_type`, the entry will be rejected. If `batch_id` is not sent in the main options, the entry will be rejected because `header_batch_id_required = true`.

---

## Common Errors and How to Avoid Them

1.  **"Call to undefined method"**: Occurs if `TransactionsHelper` has not been updated to use the `use TransactionsMultiHelper` trait. Make sure the line is present in the class.
2.  **"The given data was invalid" without enough details**: Use `testJournalEntry` to see exactly which rules were violated (`rule_violations`).
3.  **Entry imbalance due to currencies**: If currencies are mixed and you are using `auto_convert_currency`, make sure the `rate` for each detail is correct. Use `testJournalEntry` to examine the `debit_alien` and `credit_alien` totals.
4.  **`handle_exchange_differences` not working**: Make sure `relay_date_at` is passed in the options or present in the entry object, and that it is different from `date_at`. Also ensure there is an exchange difference account defined in `tss.accounts::exchange_difference_account`.
5.  **`disabled_rules` being ignored**: Remember that disabling `'balance_check'`, for example, means ignoring `debit_check_balance` even if it is `true`.

---

## Best Practices

*   **Test before executing**: Use `testJournalEntry` or `createJournalEntry($opts, true)` in the development environment.
*   **Centralise settings**: Instead of passing options like `debit_account_required` on every call, define your project's defaults in a `config` file and merge them before calling.
*   **Use the correct `type_header`**: Always use `BondsDay::class` or the like instead of plain text to ensure the correct model is used in `AccountHelper`.
*   **Disable checks with caution**: `disabled_rules` is powerful, but misusing it could introduce inconsistent data. Use it only in exceptional cases (e.g., migrating legacy data).
*   **Monitor performance**: `debit_check_balance` executes a `getBalances` query for each detail. For very large entries (hundreds of parties), it might be better to perform the balance check in the business layer in a batched manner before calling the function.
*   **Take advantage of events**: The `beforeJournalEntry` and `afterJournalEntry` events allow you to hook custom logic (notifications, audits) without modifying the core code.

---

## Unified `createEntry` Integration

The `createEntry` function (also in the same `TransactionsMultiHelper` trait) is a simplified interface that automatically chooses between `createSingle` (for two parties) and `createJournalEntry` (for more than two parties). You can use it with the same options without worrying about which function will be called:

```php
// Will automatically use createSingle (two parties)
TransactionsHelper::createEntry([
    'mony' => 100,
    'debit_accounts_id' => '...',
    'credit_accounts_id' => '...',
]);

// Will automatically use createJournalEntry because the number of parties > 2
TransactionsHelper::createEntry([
    'details' => [
        ['accounts_id' => '1', 'debit' => 500],
        ['accounts_id' => '2', 'credit' => 300],
        ['accounts_id' => '3', 'credit' => 200],
    ],
]);
```
To force the use of `createJournalEntry` even with two parties, use the option `'forceJournal' => true`.

---

## Best Practices and Tips

1.  **Test before executing**: Use `testJournalEntry` or `createJournalEntry($opts, true)` in the development environment to verify the correctness of the entry and your business logic before committing data.
2.  **Centralise settings**: Instead of passing options like `debit_account_required` on every call, define your project's defaults in a `config` file and merge them before calling. This makes the code cleaner and more consistent.
3.  **Use the correct `type_header`**: Use `BondsDay::class` or `CatchReceipt::class` instead of plain text to ensure the correct model is used in `AccountHelper::getObjHeaderFromType`.
4.  **Disable checks with caution**: The `disabled_rules` feature is powerful, but misusing it could lead to inconsistent financial data. Use it only in exceptional cases (e.g., migrating historical data) and preferably log the reason for disabling in `notes`.
5.  **Monitor performance**: The `debit_check_balance` option executes a database query (`AccountHelper::getBalances`) for each debit detail. For very large entries (hundreds of parties), it might be better to perform the balance check in the business layer in a batched manner before calling the function.
6.  **Take advantage of events**: The `beforeJournalEntry`, `afterJournalEntry`, and `tss.accounts.exchangeDifference` events allow you to hook custom logic (notifications, audits, external system integration) without modifying the core code.
7.  **Document the entry**: Use the extra fields (especially `batch_id`, `batch_type`, `extend_id`) liberally to link entries to the business entities that created them (purchase orders, invoices, contracts). This makes it easier to track transactions and perform audits.

---

## Closing Notes

- **Default values for organisational keys**: Company IDs (`companys_id`), department IDs (`departments_id`), cost centre IDs (`cost_centers_id`), and period IDs (`periods_id`) are automatically fetched from settings and user context. If the `use_default_company` setting is enabled, the parent company's default values are forced and any passed values are ignored.
- **Error handling**: If any error occurs during validation or saving, the entire operation is rolled back and an array containing `status => false` and the error message in `error` is returned. In `debug` mode, a `debug` key with detailed information is added.
- **Extensibility**: Developers can add new validation rules or modify the function's behaviour through the `beforeValidate`/`afterValidate` hooks or by listening to the fired events.

We have now covered all aspects of the `createJournalEntry` function, from the basic options to the internal mechanisms and best practices. Developers can now use this function to its full potential to build a powerful and flexible accounting system.

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)

