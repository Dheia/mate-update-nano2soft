**Documentation of Other Important Functions in `TransactionsMultiHelper`**

This file explains the other important functions in the `TransactionsMultiHelper` trait, especially those related to exchange differences and journal entry validation.

---

## 1. `checkExchangeDifferences`

Checks an existing journal entry to determine whether an exchange difference entry is needed, with the option to automatically create the entry or fire an event.

### Signature
```php
public static function checkExchangeDifferences($input, $options = [])
```

### Input
| Parameter | Type | Description |
|-----------|------|-------------|
| `$input` | `int` \| `TransactionHeader` | Entry ID (number) or the entry object itself |
| `$options` | `array` | Options array to control behaviour |

### `$options` Parameters

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `threshold` | `float` | `0.001` | The minimum financial difference (absolute value) to be considered an exchange difference. Any difference below this value is ignored. |
| `action` | `string` | `'entry'` | Action to take when differences exist:<br>`'entry'` – create a difference entry using `createExchangeDifferenceEntry`<br>`'event'` – only fire the `tss.accounts.exchangeDifference` event<br>`'none'` – only return information about the differences (no entry, no event) |
| `force_check` | `bool` | `false` | If `true`, the check is performed even if there is no posting date (`relay_date`) or if it is equal to the entry date. |
| `relay_date` | `Carbon`\|`null` | `null` | A specific date to use for the check instead of the entry’s posting date. If not provided, `$header->relay_date_at` is used. |
| `exchange_difference_account` | `string`\|`null` | `Config::get('tss.accounts::exchange_difference_account')` | The exchange difference account to use if `action = 'entry'`. If not provided, the value from the configuration file is used. |
| `entry_options` | `array` | `[]` | Additional options passed to `createJournalEntry` when creating the difference entry. Can be used to customise the resulting entry (e.g., `notes`, `transactions_type`, etc.). |

### Return Value
An array with the same format as `createJournalEntry`:
```php
[
    'code'          => 200,        // Status code (200 for success, 400 for error)
    'status'        => true,       // Operation success
    'message'       => string,     // Descriptive message
    'error'         => null,       // Error text if any
    'errors'        => null,       // Detailed error details if any
    'input_data'    => [],         // Input data (for debugging)
    'process_data'  => [],         // Processing data (for debugging)
    'model'         => TransactionHeader, // Original entry
    'data'          => [           // Detailed difference information
        'has_differences' => bool,
        'details'         => array, // Array of differences per detail
        'total_difference' => float,
        'relay_date'      => string,
        'entry_date'      => string,
    ],
    'exchange_entry' => TransactionHeader|null // The difference entry if created
]
```

### Example
```php
$result = TransactionsMultiHelper::checkExchangeDifferences(123, [
    'threshold' => 0.01,
    'action' => 'entry',
    'relay_date' => Carbon::parse('2023-12-31'),
    'entry_options' => [
        'notes' => 'Manual exchange difference entry',
        'transactions_type' => 10,
    ]
]);

if ($result['status']) {
    echo "Checked, total difference: " . $result['data']['total_difference'];
    if ($result['exchange_entry']) {
        echo "Created exchange difference entry number: " . $result['exchange_entry']->id;
    }
}
```

---

## 2. `createExchangeDifferenceEntry`

Creates an accounting entry specifically for an exchange difference for a single difference (typically called from `handleExchangeDifferences`).

### Signature
```php
public static function createExchangeDifferenceEntry($originalHeader, $originalDetail, $difference, $officialRate, $originalOptions = [])
```

### Input
| Parameter | Type | Description |
|-----------|------|-------------|
| `$originalHeader` | `TransactionHeader` | The original entry that caused the difference |
| `$originalDetail` | `array` | The processed original detail data that caused the difference |
| `$difference` | `float` | The difference amount (may be positive or negative) |
| `$officialRate` | `float` | The official exchange rate at the posting date |
| `$originalOptions` | `array` | The original options used to create the original entry (optional) |

### `$originalOptions` (important for the new entry)
- `disabled_rules` – merged with `['balance_check', 'multiplicity']` to disable some rules
- `relay_date_at` – used as the date for the new entry if present

### Return Value
Same result as `createJournalEntry` (array containing `model` and other fields).

### Notes
- Relies on the existence of a configured exchange difference account (`exchange_difference_account`).
- Creates a simple two‑party entry: the exchange difference account (debit/credit) and the original account (opposite sign).
- Links the new entry to the original one via `modul_type`, `extend_id`, `batch_type`, `batch_id`.

---

## 3. `validateJournalEntry`

Validates the data of a journal entry without creating it. Used internally in `createJournalEntry`.

### Signature
```php
public static function validateJournalEntry($options = [])
```

### Input
Same options as `createJournalEntry` (needs the full context).

### Return Value
```php
[
    'details'       => array,  // Processed and validated details
    'header_person' => array|null // Person data at header level (person_id, person_type)
]
```

### Exceptions
Throws an `ApplicationException` if there is an error in the data (e.g., unbalanced entry, account not found, etc.).

---

## 4. `handleExchangeDifferences`

An internal function called from `createJournalEntry` when `handle_exchange_differences = true`. It checks the entry after it has been created and processes exchange differences according to the settings.

### Signature
```php
protected static function handleExchangeDifferences($header, $details, $options)
```

### Input
- `$header` – The created `TransactionHeader` object
- `$details` – The processed details array
- `$options` – The original options of the entry

### Action
- Determines the posting date (`relay_date`) from the options or from the entry.
- If there is a time difference, examines each detail with a non‑base currency.
- Calculates the difference using the official rate at the posting date.
- Based on `exchange_difference_action`, either creates an entry (`entry`) or fires an event (`event`).

---

## 5. `testJournalEntry`

Simulates the creation of a journal entry without saving to the database, providing a detailed analysis of the result.

### Signature
```php
public static function testJournalEntry($options = [])
```

### Input
Same options as `createJournalEntry`.

### Return Value
Same as the result of `createJournalEntry` with an additional `analysis` key:
```php
[
    // ... regular result fields ...
    'analysis' => [
        'header_prepared'    => array,  // Header after preparation
        'details_prepared'   => array,  // Details after processing
        'model_preview'      => array,  // Representation of the object that would have been created
        'totals'             => array,  // Totals (total_debit_base, total_credit_base, ...)
        'currency_analysis'  => array,  // Analysis of currencies used
        'rule_violations'    => array,  // Rule violations if any
        'warnings'           => array,  // Warnings
        'test_passed'        => bool,   // Whether the test passed
    ],
    'original_options' => $originalOptions // The original options
]
```

### Example
```php
$test = TransactionsMultiHelper::testJournalEntry([
    'details' => [...]
]);
if ($test['analysis']['test_passed']) {
    echo "The entry is valid and can be created.";
} else {
    print_r($test['analysis']['rule_violations']);
}
```

---

## 6. `processCurrency`

An internal function to process the currency and rate for each detail.

### Signature
```php
protected static function processCurrency($detail, $context)
```

### Input
- `$detail` – The detail data array
- `$context` – The general options (contains `currencys_id`, `main_currencys_id`, `auto_convert_currency`)

### Action
- Determines the currency (`currencys_id`), the base currency (`main_currencys_id`), and the `rate`.
- If `auto_convert_currency = true` and the currency is different, converts the amount from the foreign currency to the base currency and stores the original amount in `debit_alien`/`credit_alien`.
- Returns the detail with updated fields.

### Return Value
The processed detail array.

---

## 7. `applyAccountRules`

Applies a set of rules to an account object to check its type or properties.

### Signature
```php
protected static function applyAccountRules($account, $rules, $side, $index)
```

### Input
- `$account` – The `Account` object
- `$rules` – The rules array (see explanation below)
- `$side` – `'debit'` or `'credit'` (for error messages)
- `$index` – The detail index (for error messages)

### Structure of `$rules`
Can be either:
1. **A simple list of rules** (combined with `AND`):
   ```php
   [
       ['field' => 'reports', 'operator' => '=', 'value' => '1'],
       ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe','Cash']],
   ]
   ```
2. **An array with `operator` and `rules`**:
   ```php
   [
       'operator' => 'OR',
       'rules' => [
           ['field' => 'reports', 'operator' => '=', 'value' => '1'],
           ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe','Cash']],
       ]
   ]
   ```
Here, `operator` can be `'AND'` or `'OR'` (default `'AND'`).

### Supported operators
`=`, `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.

---

## 8. `handleException`

Unified exception handling that returns a result array with debug details if `app.debug = true`.

### Signature
```php
protected static function handleException($e, $result, $default_msg_error, $options, $is_log = false)
```

### Input
- `$e` – Exception object
- `$result` – The current result array
- `$default_msg_error` – The default error message
- `$options` – The original options (used for logging)
- `$is_log` – If `true`, logs the error to the `Log` file

### Return Value
The updated `$result` array.

---

## 9. `fireHook`

Fires a custom hook function (if present in `$options`) and a system event.

### Signature
```php
protected static function fireHook($hookName, &$options, &$details = null)
```

### Input
- `$hookName` – The hook name (e.g., `'beforeValidate'`)
- `$options` – The options (allows modification)
- `$details` – The details (if any)

### Action
- If `$options[$hookName]` is callable, it is executed.
- The event `tss.accounts.$hookName` is fired.

---

## 10. `getMaxPaperId`, `generatePaperId`, `preparePaperId`, `validatePaperId`, `validatePaperIdRequired`

A set of helper functions to control `paper_id`. Refer to the `createJournalEntry` documentation for the options related to these functions.

---

## 11. `prepareDateAt`, `validateDateAtRequired`

Helper functions to control `date_at`. Refer to the `createJournalEntry` documentation for the options related to these functions.

---