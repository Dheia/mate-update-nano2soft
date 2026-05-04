# `AccountHelper` Class Documentation – Accounts Helper

## Overview

The `AccountHelper` class is located at `Tss\Accounts\Helpers\AccountHelper` and is one of the core components of the `Tss.Accounts` add-on within NanoSoft applications. The class serves as a comprehensive helper layer for dealing with financial accounts, balances, transactions, details, references, and accounting documents, providing an integrated set of static methods that make it easier for developers to perform daily accounting operations without writing complex queries.

The class responsibilities include:

- **Fetching Balances**: Calculating the balance of a specific account or a specific person (debit, credit, net) according to flexible conditions (department, period, currency, etc.).
- **Fetching Transaction Details**: Advanced querying of `TransactionsDetail` records with support for filtering by account, person, date, transaction type, and more.
- **Creating Header and Detail Objects**: Helper functions to prepare `TransactionHeader` and `TransactionsDetail` objects of various types (BondsDay, CatchReceipt, etc.) based on the document type.
- **Managing References and Parent Accounts**: Searching for a `Reference` or a `Parent Account` for a given model.
- **Automatic Account Creation**: Creating a new sub-account for a model (e.g., employee, student) automatically based on reference and parent account settings.
- **Normalizing Transaction Details**: Merging and aggregating transaction details by `transactions_id` to facilitate display.
- **Processing Query Dates**: A flexible function to apply date conditions (`date_at`, `created_at`) to database queries.
- **Filtering by User Type**: Functions to filter records based on the user's `ref_type` (e.g., driver, customer).

The class is characterized by all its methods being `static`, allowing them to be called directly from anywhere in the application without needing to create an object. It also heavily relies on settings stored in `Config` files (e.g., `tss.accounts::`) to provide flexible default behavior that can be customized per project.

---

## Response Structure for Functions Returning Objects

Some functions like `getRefAccount`, `getParentAccount`, and `createAccountToModel` return a unified array structure containing the operation status, message, and object (if found). This structure makes it easy to check for success and handle errors.

| Key | Type | Description |
| :--- | :--- | :--- |
| `status` | `bool` | Operation status (`true` for success, `false` for failure). |
| `message` | `string` | Descriptive message (in English). |
| `model` | `object\|null` | The object that was found or created (if any). |

Functions that return this structure also support the `$is_model` parameter; if `true`, the object is returned directly instead of the array.

---

## Public Methods (Public API)

### 1. `getBalances`

Calculates the balance of a specific account or a specific person (or both) based on a set of conditions. This function is the backbone of all balance query operations in the system.

```php
public static function getBalances($code, $options = [])
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$code` | `string\|array\|object\|null` | Account code (or array of codes, or person object). If `null`, can query by person only. |
| `$options` | `array` | Array of filtering options (see table below). |

#### Supported `$options` Keys

| Key | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `periods_id` | `string` | `*` (primary period calculated automatically) | Accounting period ID. |
| `departments_id` | `string` | `*` | Department / branch ID. |
| `cost_centers_id` | `string` | `*` | Cost center ID. |
| `is_relay` | `bool` | `false` | Include only posted entries? |
| `currencys_id` | `string` | `*` (base currency calculated automatically) | Currency ID. |
| `person` | `object\|null` | `null` | Person object (extracts `id` and `type`). |
| `person_id` | `int\|string` | `null` | Person ID. |
| `person_type` | `string` | `null` | Person type (Morph Class). |
| `return_value` | `string` | `'stock'` | Value to return: `'stock'`, `'debit'`, `'credit'`, `'stock_alien'`, `'debit_alien'`, `'credit_alien'`, or any other key present in the result. |
| `with_currency` | `bool` | `false` | Whether to return currency information with the result. |
| `add_select` | `array` | `[]` | Additional columns to add to `SELECT`. |
| `groub_by` | `array` | `[]` | Columns to group by (`GROUP BY`). |
| `field_date` | `string\|null` | `null` | Date field to filter on (`'date_at'`, `'created_at'`, or `'*'`). |
| `date_at` | `string\|Carbon\|null` | `null` | Start date for filtering. |
| `date_at_opration` | `string\|null` | `'='` | Comparison operator for the date (e.g., `=`, `>=`, `<=`). |
| `date_at_no_format` | `bool\|null` | `null` | If `true`, the date is used as is without formatting. |
| `to_date_at` | `string\|Carbon\|null` | `null` | End date for filtering. |
| `is_stop_force_periods` | `bool` | `false` | If `true`, automatic period filtering is ignored. |

#### Return Value

The function returns an array containing the following keys:

| Key | Type | Description |
| :--- | :--- | :--- |
| `debit` | `float` | Total debit amounts (in base currency). |
| `credit` | `float` | Total credit amounts (in base currency). |
| `stock` | `float` | Net balance (`debit - credit`). |
| `debit_alien` | `float` | Total debit amounts in foreign currency. |
| `credit_alien` | `float` | Total credit amounts in foreign currency. |
| `stock_alien` | `float` | Net balance in foreign currency. |

Plus any additional columns requested via `add_select`.

If a specific `return_value` is passed (e.g., `'stock'`), a single `float` value is returned instead of the array.

If `with_currency = true`, the result also contains a `currency` key with currency information.

#### Examples

**Get balance of a specific account:**
```php
$balance = AccountHelper::getBalances('2-2-1231010001');
echo "Balance: " . $balance['stock'];
// Or directly:
$stock = AccountHelper::getBalances('2-2-1231010001', ['return_value' => 'stock']);
```

**Get balance of a specific customer in a specific branch and period:**
```php
$balance = AccountHelper::getBalances(null, [
    'person_type' => 'RainLab\User\Models\User',
    'person_id'   => 15,
    'departments_id' => 3,
    'periods_id'  => 5,
    'return_value' => 'stock',
]);
```

**Get balance with currency details:**
```php
$result = AccountHelper::getBalances('2-2-1231010001', [
    'with_currency' => true,
]);
print_r($result['currency']); // Currency information
echo $result['stock'];        // Balance
```

---

### 2. `getTransactionsDetails`

Fetches transaction details (`TransactionsDetail`) with the ability to apply a wide range of filters. This function is mainly used in transaction reports and in `AccountsHelpers` in `Nano.AccountsApi`.

```php
public static function getTransactionsDetails($code, $options = [], $is_sql = false)
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$code` | `string\|array\|object\|null` | Account code (or array of codes, or person object). |
| `$options` | `array` | Array of filtering options (see table below). |
| `$is_sql` | `bool` | If `true`, returns the Eloquent query without executing it (for debugging). |

#### Supported `$options` Keys

| Key | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `periods_id` | `string` | `*` (calculated automatically) | Period ID. |
| `departments_id` | `string` | `*` | Department ID. |
| `cost_centers_id` | `string` | `*` | Cost center ID. |
| `is_relay` | `bool` | `false` | Include only posted entries? |
| `currencys_id` | `string` | `*` | Currency ID. |
| `person` | `object\|null` | `null` | Person object. |
| `person_id` | `int\|string` | `null` | Person ID. |
| `person_type` | `string` | `null` | Person type. |
| `accounts_type` | `string` | `'custom_accounts'` | Type of account search: `'custom_accounts'`, `'parent_accounts'`, `'custom_type'`. |
| `details_type` | `string\|null` | `null` | Detail type: `'debit'` or `'credit'`. |
| `transactions_type` | `string\|array\|null` | `null` | Transaction type (or array). |
| `transactions_id` | `int\|null` | `null` | Transaction header ID. |
| `not_transactions_id` | `int\|null` | `null` | Exclude a specific transaction header. |
| `details_id` | `int\|null` | `null` | Specific detail ID. |
| `paper_id` | `string\|null` | `null` | Paper document number. |
| `created_by` | `int\|null` | `null` | Creator ID. |
| `notes` | `string\|null` | `null` | Notes text (partial search). |
| `field_date` | `string\|null` | `null` | Date field for filtering. |
| `date_at` | `string\|Carbon\|null` | `null` | Start date. |
| `to_date_at` | `string\|Carbon\|null` | `null` | End date. |
| `date_at_opration` | `string\|null` | `'='` | Comparison operator. |
| `date_at_no_format` | `bool\|null` | `null` | Use date without formatting. |
| `custome_opration` | `string\|null` | `null` | Custom comparison operator for amount (`=`, `>`, `<`). |
| `custome_value` | `float\|null` | `null` | Amount value for custom comparison. |
| `custom_with` | `array\|string\|null` | `null` | Additional relationships to eager load. |
| `with_count` | `bool` | `false` | Load relationship counts. |
| `take_limet` | `int\|null` | `null` | Number of records to take. |
| `orderBy` | `string` | `'created_at'` | Order by column. |
| `orderDirection` | `string` | `'asc'` | Order direction. |
| `is_stop_force_periods` | `bool` | `false` | Ignore automatic period filtering. |

#### Return Value

- If `$is_sql = true`, returns a `Query Builder` object (not executed).
- In normal case, returns a `Collection` of `TransactionsDetail` objects with `Account`, `Currency`, `TransactionsType` relationships.

#### Examples

**Fetch transaction details for a specific account in a specific month:**
```php
$details = AccountHelper::getTransactionsDetails('2-2-1231010001', [
    'field_date' => 'date_at',
    'date_at'    => '2026-01-01',
    'to_date_at' => '2026-01-31',
    'orderBy'    => 'date_at',
    'orderDirection' => 'desc',
]);
foreach ($details as $detail) {
    echo $detail->debit . ' | ' . $detail->credit;
}
```

**Fetch transaction details for a specific customer:**
```php
$details = AccountHelper::getTransactionsDetails(null, [
    'person_type' => 'RainLab\User\Models\User',
    'person_id'   => 42,
]);
```

---

### 3. `getTotalAcc`

A function similar to `getBalances` but simpler and less flexible. Calculates the total for a single account (`debit`, `credit`, or `stock`). It is recommended to use `getBalances` in new developments.

```php
public static function getTotalAcc($code, $options = [])
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$code` | `string` | Account code. |
| `$options` | `array` | Filtering options (similar to `getBalances` but simpler). |

#### Supported `$options`

- `periods_id`, `departments_id`, `cost_centers_id`, `is_relay`, `is_debit`, `currencys_id`, `return_value`.

#### Example

```php
$total = AccountHelper::getTotalAcc('2-2-1231010001', [
    'return_value' => 'stock',
]);
```

---

### 4. `getQueryDate`

An internal helper function to apply date conditions to a given query. Supports three filter patterns: on `date_at` only, on `created_at` only, or on both (`*`). It is widely used inside `getBalances`, `getTransactionsDetails`, and `getTransactionHeader`.

```php
public static function getQueryDate($records, $options = [])
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$records` | `Query Builder` | The Eloquent query to apply the filter to. |
| `$options` | `array` | Date options: `field_date`, `date_at`, `date_at_opration`, `date_at_no_format`, `to_date_at`. |

#### Return Value

The query after applying the date conditions.

#### Example

```php
$query = TransactionsDetail::query();
$query = AccountHelper::getQueryDate($query, [
    'field_date' => 'date_at',
    'date_at'    => '2026-01-01',
    'to_date_at' => '2026-01-31',
]);
// Now execute $query->get()
```

---

### 5. `getObjHeader`

Creates a transaction header object (`TransactionHeader`) of the appropriate type (based on `type_header`) populated with default values and passed options. This function is the primary entry point for creating any entry, voucher, or note header.

```php
public static function getObjHeader($options = [])
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$options` | `array` | Transaction header options. |

#### Basic Options

| Key | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `type_header` | `string\|null` | `null` | Header type (e.g., `'BondsDay'` or `BondsDay::class`). If not specified, a generic `TransactionHeader` is created. |
| `periods_id` | `string` | base period | Period ID. |
| `departments_id` | `string\|null` | `null` | Department ID. |
| `cost_centers_id` | `string\|null` | `null` | Cost center ID. |
| `employees_id` | `int\|null` | employee linked to current user | Employee ID. |
| `date_at` | `Carbon` | `Carbon::now()` | Transaction date. |
| `transactions_type` | `int` | `4` | Transaction type. |
| `process_type` | `int\|string` | `4` | Process type. |
| `patterns_id` | `int` | `4` | Pattern ID. |
| `status` | `string` | `'active'` | Status. |
| `is_active` | `bool` | `true` | Is active? |
| `is_relay` | `bool` | `false` | Has it been posted? |

#### Example

```php
$header = AccountHelper::getObjHeader([
    'type_header'      => BondsDay::class,
    'departments_id'   => 2,
    'transactions_type'=> 1,
    'notes'            => 'Trial daily entry',
]);
$header->save();
```

---

### 6. `getObjHeaderFromType`

A helper function to convert a document type name (string or class name) into an object of the appropriate type. Used internally by `getObjHeader`.

```php
public static function getObjHeaderFromType($type_header = null)
```

Supported types:

| Value (string or class) | Created Object |
| :--- | :--- |
| `'BondsDay'` / `BondsDay::class` | `new BondsDay()` |
| `'Bonds2Day'` / `Bonds2Day::class` | `new Bonds2Day()` |
| `'OpeningBalance'` / `OpeningBalance::class` | `new OpeningBalance()` |
| `'CatchReceipt'` / `CatchReceipt::class` | `new CatchReceipt()` |
| `'PayReceipt'` / `PayReceipt::class` | `new PayReceipt()` |
| `'DebitNote'` / `DebitNote::class` | `new DebitNote()` |
| `'CreditNote'` / `CreditNote::class` | `new CreditNote()` |
| `'Income'` / `Income::class` | `new Income()` |
| `'Expense'` / `Expense::class` | `new Expense()` |
| `'Transfer'` / `Transfer::class` | `new Transfer()` |
| Any other value | `new TransactionHeader()` |

#### Example

```php
$payReceipt = AccountHelper::getObjHeaderFromType('PayReceipt');
// or
$payReceipt = AccountHelper::getObjHeaderFromType(PayReceipt::class);
```

---

### 7. `getObjDetail`

Creates a transaction detail object (`TransactionsDetail`) populated with default values and passed options. Used to create entry parties.

```php
public static function getObjDetail($options = [])
```

#### Basic Options

| Key | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `currencys_id` | `string` | base currency | Currency ID. |
| `rate` | `float` | `1` | Exchange rate. |
| ... | ... | ... | (remaining options similar to `getObjHeader`) |

#### Example

```php
$detail = AccountHelper::getObjDetail([
    'accounts_id'      => '2-2-1231010001',
    'debit'            => 1000,
    'currencys_id'     => 1,
    'transactions_id'  => 55,
]);
$detail->save();
```

---

### 8. `getRefAccount`

Fetches the `Reference` associated with a given model. Relies on the `tss_accounts_references` table to find the appropriate reference by table name, code, or ID.

```php
public static function getRefAccount($model, $options = [], $is_model = false)
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$model` | `string\|object` | Class path or model object. |
| `$options` | `array` | Additional options (`companys_id`, `ref_id`, `ref_code`, `ref_table_name`, `ref_code_name`). |
| `$is_model` | `bool` | If `true`, returns the object directly. |

#### Return Value

- If `$is_model = false` (default): array containing `status`, `message`, `model` (`Reference` object or `null`).
- If `$is_model = true`: `Reference` object or `null`.

#### Example

```php
$ref = AccountHelper::getRefAccount(\Tss\Basic\Models\Employee::class, [], true);
if ($ref) {
    echo "Code name: " . $ref->code_name;
}
```

---

### 9. `getParentAccount`

Fetches the parent account (`Account`) associated with a given model, relying on the `Reference` or `config` settings (e.g., `tss.accounts::accounts.employees_account`). This function is heavily used in automatically creating sub-accounts.

```php
public static function getParentAccount($model_path, $options = [], $is_model = false)
```

#### Example

```php
$parentAcc = AccountHelper::getParentAccount(Employee::class, [], true);
if ($parentAcc) {
    echo "Parent account: " . $parentAcc->name;
}
```

---

### 10. `createAccountToModel`

Creates a new sub-account (`Account`) for a given model (e.g., employee, student) and automatically links it to the reference and parent account. Fields are automatically populated (name, description, `modul_type`, `ref_type`, etc.) and the account is saved. If the option `is_update_model_accounts_id` is enabled (default), the model's `accounts_id` field is updated with the new account code.

```php
public static function createAccountToModel(Model $model, $parentAcc = null, $options = [], $is_model = false)
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$model` | `Model` | The model object to create an account for. |
| `$parentAcc` | `Account\|null` | Parent account (if not passed, it is searched automatically). |
| `$options` | `array` | Additional options (e.g., `acc_name`, `acc_prefix`). |
| `$is_model` | `bool` | If `true`, returns the account directly. |

#### Example

```php
$employee = Employee::find(10);
$result = AccountHelper::createAccountToModel($employee);
if ($result['status']) {
    echo "Account created: " . $result['model']->code;
    echo "Employee updated: " . $employee->accounts_id;
}
```

---

### 11. `getNormalizDetailTrans`

Normalizes (aggregates) multiple transaction details into a single array that groups transactions by `transactions_id`. Useful when displaying a statement where `debit` and `credit` are aggregated per transaction.

```php
public static function getNormalizDetailTrans($detailsObj)
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$detailsObj` | `Collection` | A collection of `TransactionsDetail` objects. |

#### Return Value

An associative array keyed by `transactions_id`, with each value containing:
- `transactions_id`, `date_at`, `created_at`
- `debit`, `credit`, `debit_alien`, `credit_alien` (aggregated)
- `is_alien` (is the currency foreign?)
- `mony` (final amount in the appropriate currency)

#### Example

```php
$details = TransactionsDetail::whereIn('transactions_id', [10,11,12])->get();
$normalized = AccountHelper::getNormalizDetailTrans($details);
foreach ($normalized as $transId => $data) {
    echo "Transaction $transId: debit {$data['debit']} | credit {$data['credit']}";
}
```

---

### 12. User Filtering Functions

**`getQueryWhereHasMorphPersonUsersRefType`**: Filters `TransactionsDetail` records where the `person` is of type `User` and has a specific `ref_type` (e.g., `delivery`).

**`getQueryWhereUsersRefType`**: Filters user records directly based on `ref_type`.

```php
public static function getQueryWhereHasMorphPersonUsersRefType($records, $users_ref_type)
public static function getQueryWhereUsersRefType($records, $users_ref_type, $columnName = 'users.ref_type')
```

#### Example

```php
$query = TransactionsDetail::query();
$query = AccountHelper::getQueryWhereHasMorphPersonUsersRefType($query, 'delivery');
// Fetches only transaction details where the person is a delivery driver
```

---

### 13. `checkShopReport` and `checkShopReportData`

Special functions for checking shop reports in the case of multiple branches (restaurants, stores). Determines whether additional filtering should be applied to reports based on the `tss.accounts::check_shop_report` setting.

```php
public static function checkShopReport($type = 'sub', $is_force = false)
public static function checkShopReportData($data = [], $type = 'sub')
```

#### Example

```php
$reportFilters = AccountHelper::checkShopReportData([
    'departments_id' => 5,
]);
// Now $reportFilters can be used in the report query
```

---

## Events

The `AccountHelper` class does not directly fire events, but it relies on `TransactionsHelper` events (such as `tss.accounts.beforeJournalEntry`) when used with the new `TransactionsMultiHelper` trait.

---

## Dependencies

`AccountHelper` depends on the following components:

- **`Tss\Accounts\Models\*`**: All accounting models (Account, TransactionHeader, TransactionsDetail, BondsDay, CatchReceipt, etc.).
- **`Tss\Currency\Models\Currency`**: For managing currencies.
- **`Tss\Accounts\Helpers\PersonHelper`**: For handling person types (Morph Classes).
- **`Tss\Basic\Helpers\BasicHelper`**: For preparing default company and department IDs.
- **`Illuminate\Support\Facades\DB`**: For building raw queries.
- **`Carbon\Carbon`**: For handling dates.
- **`BackendAuth`**: For getting the current backend user.
- **`Config`**: For reading default settings.

---

## Important Notes

1. **Automatic Default Values**: In `getBalances` and `getTransactionsDetails`, if `periods_id` or `currencys_id` are not provided, they are automatically fetched from settings or `getPrimary`. This can be disabled with `is_stop_force_periods = true`.

2. **Handling `use_default_company`**: If the setting `tss.accounts::use_default_company` is enabled, the passed department and company values are ignored and the parent company's default values are used. This is an important behavior to consider.

3. **Foreign Currency Support**: Results contain `_alien` keys (`debit_alien`, `credit_alien`, `stock_alien`) representing values in foreign currency. When a specific `return_value` is requested, the appropriate currency value is returned automatically (foreign if the specified currency is not base).

4. **Flexible Date Filtering**: The `getQueryDate` function supports searching by a specific date (`=`) or a range (`>=...<=`) with the option to ignore formatting (`date_at_no_format`), allowing precise time-based search.

5. **Automatic Account Creation**: `createAccountToModel` relies on `ref_type` and `parent_id` being correctly defined. Ensure that references (`References`) and parent accounts exist before using it.

6. **Static Nature**: All methods are static, meaning they do not maintain internal state and can be called from anywhere. This facilitates testing but also means they cannot be easily overridden. For extension, create additional helper classes instead of modifying this class.

---

## Integrated Examples

### Scenario 1: Calculate a customer's balance and display it in the UI

```php
$user = Auth::getUser();
$balance = AccountHelper::getBalances(null, [
    'person_type' => $user->getMorphClass(),
    'person_id'   => $user->getKey(),
    'return_value' => 'stock',
]);
echo "Your current balance: " . number_format($balance, 2) . " Riyals";
```

### Scenario 2: Automatically create a sub-account when a new student is registered

```php
$student = new Student();
$student->fill($request->all());
$student->save();

$result = AccountHelper::createAccountToModel($student);
if ($result['status']) {
    Log::info('Account created for student: ' . $student->name . ' with number ' . $student->accounts_id);
}
```

### Scenario 3: Prepare a daily entry header and details and save them

```php
$header = AccountHelper::getObjHeader([
    'type_header' => BondsDay::class,
    'transactions_type' => 1,
    'notes' => 'Settlement entry',
]);
$header->save();

$detail1 = AccountHelper::getObjDetail([
    'accounts_id' => '2-2-1253010001',
    'debit'       => 500,
    'transactions_id' => $header->id,
]);
$detail1->save();

$detail2 = AccountHelper::getObjDetail([
    'accounts_id' => '2-2-1231010001',
    'credit'      => 500,
    'transactions_id' => $header->id,
]);
$detail2->save();
```

---

## Conclusion

The `AccountHelper` class represents an indispensable tool in the NanoSoft accounting system, offering a comprehensive set of helper functions that simplify dealing with accounts, balances, transactions, and references. Thanks to its ability to handle various types of financial documents, support for multiple currencies, and flexibility in applying filters, developers can build robust accounting applications relying on this class as a solid foundation. We hope this documentation serves as a comprehensive guide to help you make the most of this tool.

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
