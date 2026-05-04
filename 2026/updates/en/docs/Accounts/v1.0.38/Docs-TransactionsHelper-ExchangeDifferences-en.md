# Documentation of Exchange Difference Functions: `checkExchangeDifferences` and `createExchangeDifferenceEntry`

## Overview

Exchange differences represent the financial variances resulting from changes in exchange rates between the date an accounting entry is recorded and its posting date (or any other comparison date). The NanoSoft accounting system provides two essential tools for handling these differences:

- **`checkExchangeDifferences`**: A general function to examine an existing entry, calculate differences for each detail in a non-base currency, and take action (create a difference entry, fire an event, or return information).
- **`createExchangeDifferenceEntry`**: A general helper function that creates a single accounting entry (two parties) to handle a specific exchange difference and links it to the original entry.

Both functions are located in the `TransactionsMultiHelper` trait and can be called statically from `TransactionsHelper`.

---

## 1. `checkExchangeDifferences` Function

### Function Signature

```php
public static function checkExchangeDifferences($input, $options = []): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$input` | `int` or `TransactionHeader` | The ID of the entry to check, or a `TransactionHeader` object. |
| `$options` | `array` | An array of options to control the check and action (see table below). |

### `$options` Parameters

| # | Key | Type | Default | Description |
|---|-----|------|---------|-------------|
| 1 | `threshold` | `float` | `0.001` | The minimum difference in exchange rate that is considered (exceeds rounding). If the difference is less than or equal to this value, it is ignored. |
| 2 | `action` | `string` | `'entry'` | The action to take when differences exist: `'entry'` (create a difference entry), `'event'` (fire an event only), `'none'` (return information only without any action). |
| 3 | `force_check` | `bool` | `false` | If `true`, the check is performed even if no posting date is available or if it equals the entry date. |
| 4 | `relay_date` | `Carbon`/`string`/`null` | `null` | The explicit posting (or comparison) date. If not provided, `relay_date_at` from the entry object is used. |
| 5 | `exchange_difference_account` | `string`/`null` | `Config::get('tss.accounts::exchange_difference_account')` | The exchange difference account code used when creating the entry (if `action = 'entry'`). |
| 6 | `entry_options` | `array` | `[]` | Additional options passed to `createJournalEntry` when creating the difference entry (e.g., `notes`, `transactions_type`...). |

### Return Value

The function returns an array with the same format as `createJournalEntry`, with the addition of an `exchange_entry` key and difference information in `data`.

| Key | Type | Description |
|-----|------|-------------|
| `code` | `int` | `200` for success, `400` for failure. |
| `status` | `bool` | Operation status. |
| `message` | `string` | Descriptive message. |
| `error` | `string`/`null` | Error message. |
| `errors` | `array`/`null` | Validation error details. |
| `model` | `TransactionHeader`/`null` | The original entry object that was checked. |
| `data` | `array` | Difference information: `has_differences` (bool), `details` (array of differences), `total_difference` (float), `relay_date`, `entry_date`. |
| `exchange_entry` | `TransactionHeader`/`null` | The exchange difference entry object if created (only when `action = 'entry'` and creation succeeds). |
| `input_data` | `array` | Input options. |
| `process_data` | `array` | Actual data used. |
| `debug` | `array`/`null` | Debug information (in `debug` environment). |

### Structure of `data['details']` (Differences)

Each element in the `details` array contains:

| Key | Type | Description |
|-----|------|-------------|
| `detail_id` | `int` | ID of the original entry detail. |
| `accounts_id` | `string` | Account code of the detail. |
| `currencys_id` | `string` | Foreign currency code. |
| `original_rate` | `float` | Exchange rate used in the original entry. |
| `official_rate` | `float` | Official exchange rate at the posting date. |
| `rate_difference` | `float` | Rate difference (`official_rate - original_rate`). |
| `alien_amount` | `float` | Amount in foreign currency (`debit_alien` or `credit_alien`). |
| `difference` | `float` | Financial difference amount (`alien_amount * rate_difference`). |
| `person_id` | `int`/`null` | Person ID associated with the detail. |
| `person_type` | `string`/`null` | Person type. |

### Function Execution Flow

1.  **Load the entry**: If `$input` is a number, load the `TransactionHeader` with the `TransactionDetail` relationship. If it is an object, ensure the details are loaded.
2.  **Merge default options**: Merge with the passed options.
3.  **Determine the posting date (`$relayDate`)**: Use `options['relay_date']` or `$header->relay_date_at`. If not present and `force_check = false`, return a result indicating no differences.
4.  **Iterate over entry details**: For each detail where `currencys_id != main_currencys_id`:
    - Fetch the official rate for the currency at the posting date using `CurrencyHelper::getRate`.
    - Calculate the rate difference (`$rateDifference = $officialRate - $originalRate`).
    - If the difference exceeds the `threshold`, calculate the financial difference (`$difference = $alienAmount * $rateDifference`) and add it to the differences list.
5.  **Build and return difference information** in `data`.
6.  **Execute the action** based on `action`:
    - `'none'`: Return the information immediately.
    - `'event'`: Fire the `tss.accounts.exchangeDifference` event and return the information.
    - `'entry'`: Build options for an aggregate difference entry (`$entryOptions`) that includes all differences (each difference results in two parties: the difference account and the original account). Call `createJournalEntry` with these options. If successful, place the resulting entry in `exchange_entry`.
7.  **Return the final result**.

### Examples

#### Example 1: Check differences for an entry and print the result without creating

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'action' => 'none',
]);
print_r($result['data']);
```

#### Example 2: Check differences for an entry and automatically create a difference entry

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'action' => 'entry',
    'threshold' => 0.01,
    'relay_date' => '2026-06-01',
]);

if ($result['exchange_entry']) {
    echo "Created exchange difference entry number: " . $result['exchange_entry']->id;
}
```

#### Example 3: Fire an event only for external processing

```php
TransactionsHelper::checkExchangeDifferences($header, [
    'action' => 'event',
]);
// The tss.accounts.exchangeDifference event will be listened to elsewhere
```

---

## 2. `createExchangeDifferenceEntry` Function

Creates a simple two‑party accounting entry representing a single exchange difference settlement. Typically used internally by `handleExchangeDifferences` (after an entry is created) or can be called directly.

### Function Signature

```php
public static function createExchangeDifferenceEntry(
    TransactionHeader $originalHeader,
    array $originalDetail,
    float $difference,
    float $officialRate,
    array $originalOptions = []
): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$originalHeader` | `TransactionHeader` | The original entry header object that resulted in the difference. |
| `$originalDetail` | `array` | The processed original detail data (same structure as a processed `details` element). |
| `$difference` | `float` | The financial difference amount (positive or negative). |
| `$officialRate` | `float` | The official exchange rate at the posting date. |
| `$originalOptions` | `array` | The original options used to create the original entry (optional). |

### Return Value

An array with the same format as `createJournalEntry`, where `model` is the created exchange difference entry object.

### Mechanism

1.  **Check for the exchange difference account**: Read `tss.accounts::exchange_difference_account` from the configuration. If not defined, throw an exception.
2.  **Prepare options for the new entry**:
    - Copy organisational values from the original entry (`companys_id`, `departments_id`...).
    - Set the date from `relay_date_at` (in `$originalOptions`) or the original entry's posting date, or the current time.
    - Set default notes describing the original entry and the difference amount.
    - Link the new entry to the original via `modul_type = 'exchange_difference'`, `extend_id = originalHeader->id`, and `batch_type`/`batch_id`.
    - Disable balance check and multiplicity rules.
3.  **Build the entry details (two parties)**:
    - **First party: Exchange difference account**:
        - If the difference is positive (the original entry recorded a lower amount in base currency) → **debit**.
        - If the difference is negative (the original entry recorded a higher amount) → **credit**.
        - Amount = absolute value of the difference.
    - **Second party: Same as the original account (opposite sign)**:
        - If the first party is a debit, the original account is **credit**.
        - If the first party is a credit, the original account is **debit**.
        - Copy `person_id`, `person_type`, `custom_name_accounts` from the original detail.
    - The base currency for all parties is the main currency.
4.  **Call `mergeDefaultCompanyValues` and then `createJournalEntry`** to create and save the entry.
5.  **Return the result of `createJournalEntry`**.

### Example

```php
// Create a difference entry directly after calculating the difference
$diff = 15.75; // Positive (original entry recorded less than actual)
$result = TransactionsHelper::createExchangeDifferenceEntry(
    $originalHeader,
    $processedDetail,
    $diff,
    3.75, // official rate
    $originalCreateOptions
);

if ($result['status']) {
    echo "Created exchange difference entry number: " . $result['model']->id;
}
```

### Notes on Sign and Amount

- **Positive difference** (`$difference > 0`): The value recorded in the original entry is lower than the actual value in the base currency. Therefore we need to **increase** the exchange difference account (debit) and decrease the original account (credit).
- **Negative difference** (`$difference < 0`): The recorded value is higher than the actual, so we decrease the exchange difference account (credit) and increase the original account (debit).

---

## 3. Relationship Between the Two Functions and Automatic Difference Handling

- When creating an entry via `createJournalEntry` with the option `handle_exchange_differences = true`, the protected function `handleExchangeDifferences` is automatically called after the entry is saved. This function in turn checks details and calls `createExchangeDifferenceEntry` for each difference that exceeds the threshold (if `exchange_difference_action = 'entry'`).
- Developers can also use `checkExchangeDifferences` independently to check any existing entry at any time (e.g., via scheduled tasks or console commands).
- The `tss.accounts.exchangeDifference` event is fired whether an entry is created or not (if `action = 'event'`), and developers can listen to it to perform custom actions.

---

## 4. Common Errors and Troubleshooting

1.  **"Account not defined" (`account_not_defined`)**: Make sure `tss.accounts::exchange_difference_account` is set in the configuration file.
2.  **No differences found**: Check that `relay_date` is different from `date_at`, and that the entries contain foreign currencies. Use `force_check = true` to bypass date conditions.
3.  **Failed to create the difference entry**: Check the passed `entry_options`; some rules (e.g., amount limits) may conflict with the difference entry. You can add `disabled_rules` to `entry_options` to bypass them.

---

## 5. Best Practices

- **Periodic checking**: Schedule calls to `checkExchangeDifferences` on unposted entries at the end of each period.
- **Use a threshold**: To avoid creating entries for pennies due to exchange rate rounding.
- **Link to original entries**: Take advantage of `batch_id` and `extend_id` to track difference entries linked to a specific original entry.
- **Test scenarios**: Use `testJournalEntry` with options that simulate the presence of differences to verify your business logic.
- **Avoid unnecessary loops**: If you are checking a large number of entries, make sure to eager‑load the `TransactionDetail` relationship (`with('TransactionDetail')`) to avoid the N+1 problem.

---

## 6. Integrated Practical Scenarios

### Scenario 1: Processing exchange differences for all unposted entries at month end

```php
$unrelayedHeaders = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->whereHas('TransactionDetail', function($q) {
        $q->whereColumn('currencys_id', '!=', 'main_currencys_id');
    })
    ->get();

foreach ($unrelayedHeaders as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now()->endOfMonth(),
        'threshold' => 0.01,
    ]);

    if ($result['exchange_entry']) {
        Log::info("Created exchange difference entry for entry {$header->id}");
    }
}
```

### Scenario 2: Using an event for review before creation

```php
Event::listen('tss.accounts.exchangeDifference', function ($payload) {
    $header     = $payload['header'];
    $differences= $payload['differences'];
    $total      = $payload['total_difference'];

    // Could send a notification to the financial reviewer
    if (abs($total) > 1000) {
        Notification::send($auditor, new LargeExchangeDifferenceFound($header, $total));
    }
});

// Then check with action = 'event'
TransactionsHelper::checkExchangeDifferences($header, ['action' => 'event']);
```

---

## 7. Conclusion

The `checkExchangeDifferences` and `createExchangeDifferenceEntry` functions provide a complete and flexible solution for managing exchange differences in the accounting system. By combining detailed checking, customisable thresholds, and multiple actions (create, fire an event, or just return), developers can automate exchange difference handling or integrate it with a manual review workflow as needed. Proper use of these two functions ensures calculation accuracy and completeness of accounting entries in a multi‑currency environment.

We have thus completed the comprehensive documentation of the exchange differences functions. Developers can now refer to this guide to understand the mechanism of each option and benefit from the practical examples to apply them in their projects.
