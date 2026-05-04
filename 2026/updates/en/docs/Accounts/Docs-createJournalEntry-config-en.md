# Complete Settings List for `createJournalEntry` Function (Unified Comprehensive Version - Updated)

These settings are passed as the `$options` array to the `createJournalEntry` function. All keys are optional and have default values as defined in the `mergeDefaultCompanyValues` function. The list below represents all possible keys with explanations for each, extracted from the reference files and gathering all available options, with default values updated to match the actual code.

```php
<?php

return [
    /*
    |--------------------------------------------------------------------------
    | Default Settings for Creating Journal Entries (createJournalEntry)
    |--------------------------------------------------------------------------
    |
    | These settings are used as default values when creating any journal entry
    | via the createJournalEntry function. They can be customized per document type
    | (e.g., BondsDay, CatchReceipt) via the configuration file for that type.
    |
    */
    'journal_entry_defaults' => [

        // ----------------------------------------------------------------------
        // 1. Basic Header Settings (passed to AccountHelper::getObjHeader)
        // ----------------------------------------------------------------------
        'periods_id'          => null,   // Accounting period ID (determined later)
        'companys_id'         => null,   // Company ID (determined later)
        'departments_id'      => null,   // Department (branch) ID – determined later
        'cost_centers_id'     => null,   // Cost center ID
        'paper_id'            => null,   // Associated paper document ID (optional)
        'employees_id'        => null,   // Responsible employee ID
        'date_at'             => null,   // Entry date (Carbon object or string; if not set, uses current time)
        'transactions_type'   => 4,      // Transaction type (number from tss_accounts_transactions_types table)
        'process_type'        => 4,      // Process type (number or text, e.g., "cash" or "credit") - default 4
        'patterns_id'         => 4,      // Accounting pattern ID (ConstraintsPattern)
        'notes'               => null,   // General notes on the entry header
        'status'              => 'active', // Entry status (active, inactive, ...)
        'is_active'           => true,   // Is the entry active?
        'is_relay'            => false,  // Has it been posted?
        'user_relay_id'       => null,   // ID of the user who posted it
        'relay_date_at'       => null,   // Posting date
        'main_currencys_id'   => null,   // Company's base currency (determined later)
        'currencys_id'        => null,   // Entry currency (determined later)
        'rate'                => 1,      // Exchange rate (decimal number)
        'type_header'         => null,   // Header type (class name, e.g., BondsDay::class)
        'default_notes'       => null,   // Default note for the header if not provided

        // ----------------------------------------------------------------------
        // 2. Person at the Header Level
        // ----------------------------------------------------------------------
        'header_person_id'    => null,   // Person ID associated with the header
        'header_person_type'  => null,   // Person type (Morph Class)
        'header_person'       => null,   // Person object (can be passed instead of id/type)

        // ----------------------------------------------------------------------
        // 3. Paper ID Settings
        // ----------------------------------------------------------------------
        'is_paper_id'               => false,   // Is the field used in the form?
        'paper_id_allowed'          => true,    // Is the user allowed to enter a value?
        'paper_id_auto'             => false,   // Is a value generated automatically if not entered?
        'paper_id_required'         => false,   // Is the field required?
        'paper_id_check_duplicate'  => true,    // Do we check for duplication?
        'paper_id_duplicate_scope'  => [],      // Duplicate check scope (array containing departments_id, periods_id, transactions_type)
        'validat_paper_id'          => 'no_rapet', // Duplicate checking method (for backward compatibility)
        'type_rapet'                => [],      // Old duplicate check conditions

        // ----------------------------------------------------------------------
        // 4. Date Settings
        // ----------------------------------------------------------------------
        'is_custome_date'       => false,   // Is the user allowed to enter the date? (if false, set automatically)
        'date_at_auto'          => false,   // Is the date generated automatically if not entered?
        'date_at_required'      => false,   // Is the date required?

        // ----------------------------------------------------------------------
        // 5. Currency and Exchange Difference Settings
        // ----------------------------------------------------------------------
        'auto_convert_currency'         => false,   // Automatically convert amounts from foreign currency to base currency
        'handle_exchange_differences'   => false,   // Enable handling exchange differences after entry creation
        'exchange_difference_action'    => 'event', // Action when differences exist: 'entry' (create entry), 'event' (fire event), 'none' (ignore)

        // ----------------------------------------------------------------------
        // 6. Entry Details (Required)
        // ----------------------------------------------------------------------
        'details'             => [],     // Array containing entry details (each element is a party)

        // ----------------------------------------------------------------------
        // 7. Party Count Limits (per side)
        // ----------------------------------------------------------------------
        'max_debit_entries'   => null,   // Maximum number of debit parties (null = unlimited)
        'max_credit_entries'  => null,   // Maximum number of credit parties (null = unlimited)
        'allow_multiple_debit'  => true, // Allow multiple debit parties
        'allow_multiple_credit' => true, // Allow multiple credit parties

        // ----------------------------------------------------------------------
        // 8. Person Settings – Debit Side
        // ----------------------------------------------------------------------
        'debit_person_allowed'        => true,   // Allow linking a person to the debit side
        'debit_person_required'       => false,  // Is a person required on the debit side?
        'debit_person_allowed_types'  => null,   // Allowed person types (array, e.g., [User::class])

        // ----------------------------------------------------------------------
        // 9. Person Settings – Credit Side
        // ----------------------------------------------------------------------
        'credit_person_allowed'       => true,   // Allow linking a person to the credit side
        'credit_person_required'      => false,  // Is a person required on the credit side?
        'credit_person_allowed_types' => null,   // Allowed person types

        // ----------------------------------------------------------------------
        // 10. Account Settings – Debit Side
        // ----------------------------------------------------------------------
        'debit_account_allowed'   => true,   // Allow specifying a debit account
        'debit_account_required'  => true,   // Is a debit account required?
        'debit_default_account'   => null,   // Default account for debit (account code)

        // ----------------------------------------------------------------------
        // 11. Account Settings – Credit Side
        // ----------------------------------------------------------------------
        'credit_account_allowed'  => true,   // Allow specifying a credit account
        'credit_account_required' => true,   // Is a credit account required?
        'credit_default_account'  => null,   // Default account for credit (account code)

        // ----------------------------------------------------------------------
        // 12. Notes Settings – Debit Side
        // ----------------------------------------------------------------------
        'debit_notes_allowed'     => true,   // Allow entering a note on the debit side
        'debit_notes_required'    => false,  // Is a note required on the debit side?
        'debit_default_notes'     => null,   // Default note for debit

        // ----------------------------------------------------------------------
        // 13. Notes Settings – Credit Side
        // ----------------------------------------------------------------------
        'credit_notes_allowed'    => true,   // Allow entering a note on the credit side
        'credit_notes_required'   => false,  // Is a note required on the credit side?
        'credit_default_notes'    => null,   // Default note for credit

        // ----------------------------------------------------------------------
        // 14. Total Amount Limits
        // ----------------------------------------------------------------------
        'min_total_amount'        => null,   // Minimum total amount (debit total = credit total)
        'max_total_amount'        => null,   // Maximum total amount
        'check_min_total'         => true,   // Enable checking the minimum total amount
        'check_max_total'         => true,   // Enable checking the maximum total amount

        // ----------------------------------------------------------------------
        // 15. Amount Limits – Debit Side
        // ----------------------------------------------------------------------
        'check_debit_min'         => false,  // Enable checking the minimum total debit amount
        'check_debit_max'         => false,  // Enable checking the maximum total debit amount
        'min_debit_amount'        => null,   // Minimum total debit amount
        'max_debit_amount'        => null,   // Maximum total debit amount

        // ----------------------------------------------------------------------
        // 16. Amount Limits – Credit Side
        // ----------------------------------------------------------------------
        'check_credit_min'        => false,  // Enable checking the minimum total credit amount
        'check_credit_max'        => false,  // Enable checking the maximum total credit amount
        'min_credit_amount'       => null,   // Minimum total credit amount
        'max_credit_amount'       => null,   // Maximum total credit amount

        // ----------------------------------------------------------------------
        // 17. Balance Check – Debit Side
        // ----------------------------------------------------------------------
        'debit_check_balance'          => false,   // Enable balance check before debiting
        'debit_balance_check_callback' => null,    // Custom balance check function (overrides default check)

        // ----------------------------------------------------------------------
        // 18. Balance Check – Credit Side
        // ----------------------------------------------------------------------
        'credit_check_balance'         => false,   // Enable balance check before crediting
        'credit_balance_check_callback'=> null,    // Custom balance check function

        // ----------------------------------------------------------------------
        // 19. Dynamic Account Type Checking – Debit Side
        // ----------------------------------------------------------------------
        'debit_check_account_types'     => false,   // Enable account type checking on debit side
        'debit_rules_account_types'     => null,    // Array of rules for account type checking (see explanation)

        // ----------------------------------------------------------------------
        // 20. Dynamic Account Type Checking – Credit Side
        // ----------------------------------------------------------------------
        'credit_check_account_types'    => false,   // Enable account type checking on credit side
        'credit_rules_account_types'    => null,    // Array of rules for account type checking

        // ----------------------------------------------------------------------
        // 21. Extra Fields in Header
        // ----------------------------------------------------------------------
        'header_extend_id_allowed'      => true,    // Allow using extend_id in header
        'header_extend_id_required'     => false,   // Is extend_id required in header?
        'header_extend_id_default'      => null,    // Default value for extend_id in header

        'header_modul_type_allowed'     => true,    // Allow using modul_type in header
        'header_modul_type_required'    => false,   // Is modul_type required in header?
        'header_modul_type_default'     => null,    // Default value for modul_type in header

        'header_ref_type_allowed'       => true,    // Allow using ref_type in header
        'header_ref_type_required'      => false,   // Is ref_type required in header?
        'header_ref_type_default'       => null,    // Default value for ref_type in header

        'header_ref_key_name_allowed'   => true,    // Allow using ref_key_name in header
        'header_ref_key_name_required'  => false,   // Is ref_key_name required in header?
        'header_ref_key_name_default'   => null,    // Default value for ref_key_name in header

        'header_ref_key_values_allowed' => true,    // Allow using ref_key_values in header
        'header_ref_key_values_required'=> false,   // Is ref_key_values required in header?
        'header_ref_key_values_default' => null,    // Default value for ref_key_values in header

        'header_batch_type_allowed'     => true,    // Allow using batch_type in header
        'header_batch_type_required'    => false,   // Is batch_type required in header?
        'header_batch_type_default'     => null,    // Default value for batch_type in header

        'header_batch_id_allowed'       => true,    // Allow using batch_id in header
        'header_batch_id_required'      => false,   // Is batch_id required in header?
        'header_batch_id_default'       => null,    // Default value for batch_id in header

        // ----------------------------------------------------------------------
        // 22. Extra Fields in Debit Side Details
        // ----------------------------------------------------------------------
        'debit_extend_id_allowed'       => true,
        'debit_extend_id_required'      => false,
        'debit_extend_id_default'       => null,

        'debit_modul_type_allowed'      => true,
        'debit_modul_type_required'     => false,
        'debit_modul_type_default'      => null,

        'debit_ref_type_allowed'        => true,
        'debit_ref_type_required'       => false,
        'debit_ref_type_default'        => null,

        'debit_ref_key_name_allowed'    => true,
        'debit_ref_key_name_required'   => false,
        'debit_ref_key_name_default'    => null,

        'debit_ref_key_values_allowed'  => true,
        'debit_ref_key_values_required' => false,
        'debit_ref_key_values_default'  => null,

        'debit_batch_type_allowed'      => true,
        'debit_batch_type_required'     => false,
        'debit_batch_type_default'      => null,

        'debit_batch_id_allowed'        => true,
        'debit_batch_id_required'       => false,
        'debit_batch_id_default'        => null,

        // ----------------------------------------------------------------------
        // 23. Extra Fields in Credit Side Details
        // ----------------------------------------------------------------------
        'credit_extend_id_allowed'       => true,
        'credit_extend_id_required'      => false,
        'credit_extend_id_default'       => null,

        'credit_modul_type_allowed'      => true,
        'credit_modul_type_required'     => false,
        'credit_modul_type_default'      => null,

        'credit_ref_type_allowed'        => true,
        'credit_ref_type_required'       => false,
        'credit_ref_type_default'        => null,

        'credit_ref_key_name_allowed'    => true,
        'credit_ref_key_name_required'   => false,
        'credit_ref_key_name_default'    => null,

        'credit_ref_key_values_allowed'  => true,
        'credit_ref_key_values_required' => false,
        'credit_ref_key_values_default'  => null,

        'credit_batch_type_allowed'      => true,
        'credit_batch_type_required'     => false,
        'credit_batch_type_default'      => null,

        'credit_batch_id_allowed'        => true,
        'credit_batch_id_required'       => false,
        'credit_batch_id_default'        => null,

        // ----------------------------------------------------------------------
        // 24. Additional Settings (Hooks and Disabling Rules)
        // ----------------------------------------------------------------------
        'auto_generate_custom_name' => true,   // Automatically generate a custom person name if not passed
        'beforeValidate'            => null,   // Function executed before data validation (receives $options and $details)
        'afterValidate'             => null,   // Function executed after data validation (receives $options and $details)
        'disabled_rules'            => [],     // Array of rule names to disable (e.g., ['balance_check', 'multiplicity'])

        // ----------------------------------------------------------------------
        // 25. Note: `id` can be passed in some contexts (for exception when checking duplication)
        // ----------------------------------------------------------------------
    ],
];
```

---

## Detailed Explanation of `debit_rules_account_types` (and its counterpart `credit_rules_account_types`)

This option can come in two formats:

### Simplified Format (Array of rules with automatic AND)
```php
$rules = [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
];
```

### Advanced Format with Logical Operator
```php
$rules = [
    'operator' => 'OR',   // or 'AND'
    'rules'    => [
        ['field' => 'reports', 'operator' => '=', 'value' => '1'],
        ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
        ['field' => 'account_number', 'operator' => 'REGEXP', 'value' => '/^1253/'],
    ],
];
```

### Supported Fields from the `accounts` Table:
- `reports` (1: balance sheet, 2: profit and loss)
- `final_account_id`
- `modul_type` (e.g., "Boxe", "Suppler", "Customer")
- `account_normal` (0 debit, 1 credit)
- `groups_accounts_id`
- `ref_type`
- `person_type`
- `account_number`
- `parent_id`
- `level`
- `main_sub` (main / sub)
- and other columns of the `accounts` table.

### Supported Operators:
- `=`, `==`, `!=`, `<>`
- `>`, `>=`, `<`, `<=`
- `LIKE`, `NOT LIKE`
- `IN`, `NOT IN` (with array)
- `REGEXP` (regular expression)

### Various Examples of `debit_rules_account_types`:

```php
// Allow only treasury accounts (modul_type = 'Boxe')
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
]

// Allow customer or supplier accounts (ref_type = 14 for customers, 13 for suppliers)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    'operator' => 'OR',
    'rules' => [
        ['field' => 'ref_type', 'operator' => '=', 'value' => '14'],
        ['field' => 'ref_type', 'operator' => '=', 'value' => '13']
    ]
]

// Asset accounts (reports = 1) and not main (main_sub = 'sub')
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'main_sub', 'operator' => '=', 'value' => 'sub']
]

// Accounts starting with a specific prefix in account_number (e.g., 1253 for treasuries)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'account_number', 'operator' => 'LIKE', 'value' => '1253%']
]
// Or using REGEXP:
// ['field' => 'account_number', 'operator' => 'REGEXP', 'value' => '/^1253/']

// Accounts from a specific group (groups_accounts_id = 5) or debit nature (account_normal = 0)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    'operator' => 'OR',
    'rules' => [
        ['field' => 'groups_accounts_id', 'operator' => '=', 'value' => '5'],
        ['field' => 'account_normal', 'operator' => '=', 'value' => '0']
    ]
]

// Accounts that are not of type 'Suppler' (i.e., not supplier accounts)
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'modul_type', 'operator' => '!=', 'value' => 'Suppler']
]

// Complex rules using IN with multiple values (e.g., modul_type in ['Boxe', 'Bank'])
'debit_check_account_types' => true,
'debit_rules_account_types' => [
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Bank']]
]
```

---

## Detailed Explanation of `debit_balance_check_callback` and `credit_balance_check_callback`

These are custom functions for checking balance before executing the transaction, and they override the default check. The function receives the following parameters:
- `$account`: The Account object
- `$amount`: The required amount
- `$balance`: The current balance of the account (calculated according to context)
- `$personId`: The associated person ID (may be null)
- `$personType`: The person type (Morph Class)
- `$context`: An array containing additional context (e.g., `transactions_type`, `process_type`, etc.)

### Examples:

```php
// A simple function that logs a warning but does not prevent the operation
'debit_check_balance' => true,
'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    if ($balance < $amount) {
        Log::warning("Insufficient balance in account {$account->code}: balance {$balance}, required {$amount}");
    }
    return true; // Allow the operation with a warning log
}

// A function that allows withdrawal up to a certain credit limit (10,000)
'credit_check_balance' => true,
'credit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    $creditLimit = 10000;
    if ($balance + $creditLimit < $amount) {
        throw new ApplicationException("Exceeding the credit limit for account {$account->code}");
    }
    return true;
}

// A function that uses person information to determine the allowed limit (if the person exists)
'debit_check_balance' => true,
'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    if ($personId && $personType) {
        $person = \Tss\Accounts\Helpers\PersonHelper::getPersonObj($personType, $personId);
        if ($person && property_exists($person, 'credit_limit') && $person->credit_limit) {
            if ($balance + $person->credit_limit < $amount) {
                throw new ApplicationException("Insufficient credit limit for the person");
            }
        }
    }
    // Default check
    if ($balance < $amount) {
        throw new ApplicationException("Insufficient account balance");
    }
    return true;
}

// A function that allows withdrawal only if the amount is less than 50,000 regardless of balance (for certain accounts)
'credit_check_balance' => true,
'credit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    // If the account is of treasury type, allow large amounts
    if ($account->modul_type == 'Boxe') {
        if ($amount > 100000) {
            throw new ApplicationException("Amount exceeds the maximum allowed for treasury");
        }
        return true;
    }
    // Other accounts follow the normal check
    if ($balance < $amount) {
        throw new ApplicationException("Insufficient account balance");
    }
    return true;
}

// A function that uses additional context (e.g., transaction type) to determine check behavior
'credit_check_balance' => true,
'credit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    // If the transaction is of type 'cash', allow more flexibility
    if (($context['transactions_type'] ?? null) == 2) { // 2 = receipt voucher, for example
        if ($balance + 5000 < $amount) {
            throw new ApplicationException("Insufficient balance even with flexibility");
        }
    } else {
        if ($balance < $amount) {
            throw new ApplicationException("Insufficient account balance");
        }
    }
    return true;
}
```

---

## Alternative Hierarchical Structure (using nested keys)

Settings can also be organized hierarchically to group options related to each side. This structure is supported as an alternative. The example below illustrates this organization:

```php
'journal_entry_defaults' => [
    // Basic header settings (as above)
    'periods_id' => null,
    // ... remaining header settings

    // Debit side settings
    'debit' => [
        'max_entries'           => null,
        'allow_multiple'        => true,
        'person_allowed'        => true,
        'person_required'       => false,
        'person_allowed_types'  => null,
        'account_allowed'       => true,
        'account_required'      => true,
        'default_account'       => null,
        'notes_allowed'         => true,
        'notes_required'        => false,
        'default_notes'         => null,
        'check_balance'         => false,
        'balance_check_callback'=> null,
        'check_account_types'   => false,
        'account_type_rules'    => null,
        'extend_id_allowed'     => true,
        // ... remaining extra fields for debit side
    ],

    // Credit side settings (same structure)
    'credit' => [ ... ],

    // Header extra fields settings
    'header' => [ ... ],

    // Amount limits
    'amount_limits' => [
        'total' => ['min' => null, 'max' => null, 'check_min' => true, 'check_max' => true],
        'debit' => ['min' => null, 'max' => null, 'check_min' => false, 'check_max' => false],
        'credit' => ['min' => null, 'max' => null, 'check_min' => false, 'check_max' => false],
    ],
];
```

You can use either the flat or nested structure according to your preference, as both are supported in the code.

---

## Concluding Notes

- Default values for company ID, department ID, cost center, and period are automatically fetched from helper functions (`getCheckCompanysId`, `getCheckDepartmentsId`, etc.) based on user context or general settings.
- If `use_default_company` is enabled in the add-on settings, the input values will be overridden by the company's default values.
- Any of these options can be passed optionally, and they will be merged with the default values defined in `mergeDefaultCompanyValues`.
- The `disabled_rules` field can be used to disable certain validation rules, such as `'balance_check'`, `'multiplicity'`, or `'account_type_rules'`.
- The `beforeValidate` and `afterValidate` functions allow you to execute custom code before and after the validation process. They receive `$options` and `$details` (after processing) and can modify them or add additional validation.
- In some contexts (e.g., update), an `id` key can be passed to exclude the current entry when checking `paper_id` duplication.

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
