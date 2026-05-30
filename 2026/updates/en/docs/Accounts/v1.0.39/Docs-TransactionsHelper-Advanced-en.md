# Advanced and Supplemental Documentation for `TransactionsHelper` and `TransactionsMultiHelper`

## 1. Internal Structure and Function Map

To understand the class in depth, it is necessary to know the relationship between public and private methods and how data flows between them.

### 1.1. General Structure
- **`TransactionsHelper`**: The main class. Contains business operation functions (`createSingle`, `addPayment`, ...) and query functions (`getTransactionHeader`, ...). Uses the `TransactionsMultiHelper` trait.
- **`TransactionsMultiHelper`**: A trait that provides:
    - **Public methods:** `createJournalEntry`, `createEntry`, `validateJournalEntry`, `testJournalEntry`, `checkExchangeDifferences`.
    - **Helper methods (protected static):** Responsible for the individual steps in the multi-party entry creation process.

### 1.2. `createJournalEntry` Flow Map
This diagram illustrates the sequence of function calls when executing `createJournalEntry`:
1.  `createJournalEntry(options)`
2.  `mergeDefaultCompanyValues(options)` (merge default values)
3.  `preparePaperId(options)` / `validatePaperIdRequired(options)` / `validatePaperId(options)`
4.  `prepareDateAt(options)` / `validateDateAtRequired(options)`
5.  `validatePeriod(options)`
6.  `fireHook('beforeValidate', options)`
7.  `validateJournalEntry(options)` -> internally calls `validateJournalDetails`, which in turn calls `validateSingleDetail`
8.  `fireHook('afterValidate', options, details)`
9.  `calculateTotals(details)`
10. `prepareHeaderOptions(options, headerPerson)`
11. **Create and save** the header and details.
12. `handleExchangeDifferences(header, details, options)`

---

## 2. Default Value System (`mergeDefaultCompanyValues`)

The `mergeDefaultCompanyValues` function is the key to flexibility in `createJournalEntry`. It merges the passed `$options` with a large array of default values. Understanding these values is the key to using the function correctly.

```php
$defaults = [
    'process_type'        => 4,
    'auto_generate_custom_name' => true,
    // ... all other keys
];
```
**Important Note:** The default values for keys such as `debit_account_required` are `true`, meaning the function enforces the presence of a debit and credit account in every detail **unless explicitly disabled** via options. This differs from `createSingle`, which allows more flexibility in format.

---

## 3. Internal Workings of the "FromOrders" Functions

These functions (`createPaymentFromOrders`, `createBillShopFromOrders`, etc.) follow a standard pattern; understanding this pattern makes debugging and customisation easier.

1.  **Validate the `Order` object**: Check that the order is in the correct state (completed) and that the entry has not already been created (using `TransactionHeader::where(...)->first()`).
2.  **Calculate the amount**: The amount (`$mony`) is determined from the order's properties (`order_total`, `cart_total`, `shipping_total`, etc.).
3.  **Prepare `createSingle` options**:
    - Process settings (e.g., `transactions_type`, `debit_accounts_id`) are fetched from the `Config` file (e.g., `tss.accounts::create_brokerage_shop_from_orders`).
    - `modul_type` (usually `'order'`), `extend_id` (order ID), and `batch_type`/`id` are set to facilitate later tracking of the entry.
    - The "person" (debit or credit) is determined based on the context of the operation (customer, courier, store).
4.  **Call `createSingle`**: Pass all the prepared options to `createSingle`.
5.  **Process the result and fire events**: On success, the order is updated (e.g., set `is_trans_brokerage = true`) and an event such as `tss.accounts.afterBrokerage` is fired.

---

## 4. Currency and Exchange Difference Handling System (Detailed)

### 4.1. Currency Processing Flow in `validateSingleDetail` (the `processCurrency` function)
1.  **Determine the base currency**: Taken from `$context['main_currencys_id']`.
2.  **Determine the detail's currency**: Taken from `$detail['currencys_id']`; if not present, the header currency `$context['currencys_id']` is used, then the base currency.
3.  **Determine the exchange rate (`$rate`)**: Taken from `$detail['rate']`, or `$context['rate']`, or `1`.
4.  **Conversion (if `auto_convert_currency = true` and the currency is foreign)**:
    - The entered `debit` and `credit` values are considered foreign.
    - The original values are copied to `debit_alien` and `credit_alien`.
    - The base values are converted to the base currency: `$debit = $debit * $rate`.
5.  **When not enabled (`auto_convert_currency = false`)**:
    - The entered `debit` and `credit` values are considered to be already in the base currency.
    - If the currency is foreign, it is assumed that the model's `beforeSave` will handle the conversion (assuming the entered amount is the foreign amount). This can be confusing, so it is recommended to use `auto_convert_currency = true` for explicit control.

### 4.2. The `handleExchangeDifferences` Mechanism
- **Activation condition:** The option `handle_exchange_differences = true` must be set (default `false`).
- **Check condition:** The presence of `relay_date_at` (posting date) in the options or in the entry object (`$header->relay_date_at`), and that it differs from `date_at` (entry date).
- **How it works:** For each detail with a non‑base currency, the official exchange rate at `relay_date` is fetched. If it differs from the `rate` used in the detail, the financial difference is calculated. Based on `exchange_difference_action`:
    - `'entry'`: Create a difference entry.
    - `'event'`: Fire an event for external processing.

---

## 5. Extension of Advanced Options

### 5.1. Effectively Using `disabled_rules`
You can pass a `disabled_rules` array to disable certain checks in special cases. The currently supported string identifiers are:
- `'balance_check'`: Disable the balance check.
- `'multiplicity'`: Disable the party count check.
- `'amount_limits'`: Disable the amount limits check (total and side).

**Example:** Create a reconciliation entry that does not require party count balance or a balance check.
```php
TransactionsHelper::createEntry([
    'details' => [...],
    'disabled_rules' => ['balance_check', 'multiplicity'],
]);
```

### 5.2. Hooks System (`beforeValidate` and `afterValidate`)
You can pass anonymous functions (Closures) to control logic before or after the entry is validated.

**Example: Automatically modify detail amounts before validation.**
```php
TransactionsHelper::createJournalEntry([
    'details' => [...],
    'beforeValidate' => function(&$options, &$details) {
        // Add 10% tax to every debit amount
        foreach ($details as &$detail) {
            if ($detail['debit'] > 0) {
                $detail['debit'] *= 1.10;
            }
        }
    },
]);
```

---

## 6. Advanced Usage Scenarios (Not Previously Covered)

### 6.1. Batch Programming to Create Periodic Entries
Using `createJournalEntry`, you can easily build a system for periodic entries.

```php
// inside a scheduled command
$templates = RecurringJournalTemplate::where('next_run_at', '<=', now())->get();
foreach ($templates as $template) {
    $options = $template->options; // load options from template
    $options['date_at'] = now();

    $result = TransactionsHelper::createJournalEntry($options);

    if ($result['status']) {
        $template->last_run_at = now();
        $template->next_run_at = now()->addDays($template->interval_days);
        $template->save();
    } else {
        Log::error("Failed recurring journal: " . $result['error']);
    }
}
```

### 6.2. Building a `JournalEntryBuilder` Library (Fluent API)
To simplify the creation of complex entries, you can build a Builder class:

```php
class JournalEntryBuilder
{
    protected $options = [];

    public function setType($typeHeader, $transactionsType) { ... }
    public function setDate($date) { ... }
    public function addDebit($accountId, $amount, $person = null) { ... }
    public function addCredit($accountId, $amount, $person = null) { ... }
    public function setNotes($notes) { ... }
    public function withBalanceCheck() { ... }
    public function execute() { 
        return TransactionsHelper::createEntry($this->options); 
    }
}

// Usage:
$entry = new JournalEntryBuilder();
$entry->setType(BondsDay::class, 1)
      ->setDate(today())
      ->addDebit('2-2-1253010001', 500)
      ->addCredit('2-2-1231010001', 500)
      ->execute();
```

### 6.3. Automatically Creating a "Reconciliation" Entry from an Unbalanced Entry
You can program a function that uses `testJournalEntry` and then `afterValidate` to automatically create a reconciliation entry for the difference:

```php
$test = TransactionsHelper::testJournalEntry($options);
if (!$test['analysis']['test_passed']) {
    $difference = $test['analysis']['totals']['total_debit_base'] - $test['analysis']['totals']['total_credit_base'];
    if ($difference > 0) {
        $options['details'][] = ['accounts_id' => '2-2-SUSPENSE_ACCOUNT', 'credit' => $difference];
    } else {
        $options['details'][] = ['accounts_id' => '2-2-SUSPENSE_ACCOUNT', 'debit' => abs($difference)];
    }
    TransactionsHelper::createJournalEntry($options);
}
```

---

## 7. Debugging and Common Errors

### 7.1. How to Use `process_data` and `input_data`
The function results contain `input_data` (inputs after initial cleaning) and `process_data` (the data actually used after merging with default values). When debugging, compare them to understand how your options were interpreted.

### 7.2. Common Errors
- **"Call to undefined method"**: Caused by not using `use \Tss\Accounts\Helpers\TransactionsMultiHelper;` in an older version of `TransactionsHelper`. Make sure the class is updated.
- **"The given data was invalid" without details**: There may be an issue with custom validation rules. Check the Laravel logs (`/storage/logs/laravel.log`) or use `testJournalEntry` to see the violations.
- **Exchange differences are not created**: Make sure `handle_exchange_differences => true` is set in the options, and that the entry has a `relay_date_at` different from `date_at`.

---

## 8. Best Practices and Tips

1.  **Test the entry first**: Use `testJournalEntry` or `createJournalEntry($options, true)` before actual creation.
2.  **Set general options in a `config` file**: To avoid repeating settings, put your project's default values in a settings file (e.g., `journal_entry_defaults`) and merge them with the options before calling.
3.  **Use the appropriate `type_header`**: Always use `BondsDay::class` or `CatchReceipt::class` instead of plain strings to benefit from the specialised models.
4.  **Disable checks with caution**: `disabled_rules` is useful in exceptional cases, but misusing it could lead to inconsistent data.
5.  **Log errors**: Use `Log::error` with `$result` in case of failure to track problems in the production environment.

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
