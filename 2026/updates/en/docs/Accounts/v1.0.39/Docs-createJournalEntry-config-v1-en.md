# Complete Settings List for `createJournalEntry` Function

These settings are passed as the `$options` array to the `createJournalEntry` function. All keys are optional and have default values as defined in the `mergeDefaultCompanyValues` function.

---

## 1. Basic Header Fields (passed to `AccountHelper::getObjHeader`)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `companys_id` | int/string | `null` (calculated later) | Company ID |
| `departments_id` | int/string | `null` (calculated later) | Department (branch) ID |
| `cost_centers_id` | int/string | `null` (calculated later) | Cost center ID |
| `periods_id` | int/string | `null` (calculated later) | Accounting period ID |
| `paper_id` | mixed | `null` | Paper document number (can be text or number) |
| `employees_id` | int | `null` | Responsible employee ID |
| `date_at` | Carbon/string | `Carbon::now()` | Entry date (adjusted according to `is_custome_date` and `date_at_auto`) |
| `transactions_type` | int | `4` | Transaction type (from `transactions_types` table) |
| `process_type` | string | `'4'` | Process type (e.g., "cash") |
| `patterns_id` | int | `4` | Accounting pattern ID (ConstraintsPattern) |
| `notes` | string | `null` | General notes on the entry header |
| `default_notes` | string | `null` | Default note if `notes` is not provided |
| `type_header` | string | `null` | Header type (class name, e.g., `BondsDay::class`) |
| `status` | string | `'active'` | Entry status |
| `is_active` | bool | `true` | Is the entry active? |
| `is_relay` | bool | `false` | Has it been posted? |
| `user_relay_id` | int | `null` | ID of the user who posted it |
| `relay_date_at` | Carbon | `null` | Posting date |

---

## 2. Person at the Header Level

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `header_person_id` | int | `null` | Person ID associated with the header |
| `header_person_type` | string | `null` | Person type (Morph Class) |
| `header_person` | object | `null` | Person object (can be passed instead of id/type) |

---

## 3. Paper ID Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `is_paper_id` | bool | `false` | Is the field used in the form? |
| `paper_id_allowed` | bool | `true` | Is the user allowed to enter a value? |
| `paper_id_auto` | bool | `false` | Is a value generated automatically if not entered? |
| `paper_id_required` | bool | `false` | Is the field required? |
| `paper_id_check_duplicate` | bool | `true` | Do we check for duplication? |
| `paper_id_duplicate_scope` | array | `[]` | Duplicate check scope: can contain `'departments_id'`, `'periods_id'`, `'transactions_type'` |
| `validat_paper_id` | string | `'no_rapet'` | (for compatibility) Duplicate checking method |
| `type_rapet` | array | `[]` | (for compatibility) Old duplicate check conditions |

---

## 4. Date Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `is_custome_date` | bool | `false` | Is the user allowed to enter the date? (if `false`, set automatically) |
| `date_at_auto` | bool | `false` | Is the date generated automatically if not entered? |
| `date_at_required` | bool | `false` | Is the date required? |

---

## 5. Currency and Exchange Difference Settings

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `main_currencys_id` | string | `null` (calculated later) | Base currency |
| `currencys_id` | string | `null` (calculated later) | Entry currency |
| `rate` | float | `1` | Exchange rate |
| `auto_convert_currency` | bool | `false` | Automatically convert amounts from foreign currency to base currency |
| `handle_exchange_differences` | bool | `false` | Enable handling exchange differences after entry creation |
| `exchange_difference_action` | string | `'entry'` | Action when differences exist: `'entry'` (create entry), `'event'` (fire event), `'none'` (ignore) |

---

## 6. Party Count Limits (Multiplicity Controls)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `max_debit_entries` | int/null | `null` | Maximum number of debit parties (null = unlimited) |
| `max_credit_entries` | int/null | `null` | Maximum number of credit parties (null = unlimited) |
| `allow_multiple_debit` | bool | `true` | Allow multiple debit parties |
| `allow_multiple_credit` | bool | `true` | Allow multiple credit parties |

---

## 7. Person Settings per Side

### Debit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `debit_person_allowed` | bool | `true` | Allow linking a person to the debit side |
| `debit_person_required` | bool | `false` | Is a person required on the debit side? |
| `debit_person_allowed_types` | array/null | `null` | Allowed person types (array of class names) |

### Credit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `credit_person_allowed` | bool | `true` | Allow linking a person to the credit side |
| `credit_person_required` | bool | `false` | Is a person required on the credit side? |
| `credit_person_allowed_types` | array/null | `null` | Allowed person types |

---

## 8. Account Settings per Side

### Debit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `debit_account_allowed` | bool | `true` | Allow specifying a debit account |
| `debit_account_required` | bool | `true` | Is a debit account required? |
| `debit_default_account` | string/null | `null` | Default account for debit (account code) |

### Credit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `credit_account_allowed` | bool | `true` | Allow specifying a credit account |
| `credit_account_required` | bool | `true` | Is a credit account required? |
| `credit_default_account` | string/null | `null` | Default account for credit (account code) |

---

## 9. Notes Settings per Side

### Debit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `debit_notes_allowed` | bool | `true` | Allow entering a note on the debit side |
| `debit_notes_required` | bool | `false` | Is a note required on the debit side? |
| `debit_default_notes` | string/null | `null` | Default note for debit |

### Credit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `credit_notes_allowed` | bool | `true` | Allow entering a note on the credit side |
| `credit_notes_required` | bool | `false` | Is a note required on the credit side? |
| `credit_default_notes` | string/null | `null` | Default note for credit |

---

## 10. Amount Limits

### Total Amount

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `min_total_amount` | float/null | `null` | Minimum total amount (debit total = credit total) |
| `max_total_amount` | float/null | `null` | Maximum total amount |
| `check_min_total` | bool | `true` | Enable checking the minimum total amount |
| `check_max_total` | bool | `true` | Enable checking the maximum total amount |

### Debit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `min_debit_amount` | float/null | `null` | Minimum total debit amount |
| `max_debit_amount` | float/null | `null` | Maximum total debit amount |
| `check_debit_min` | bool | `false` | Enable checking the minimum debit amount |
| `check_debit_max` | bool | `false` | Enable checking the maximum debit amount |

### Credit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `min_credit_amount` | float/null | `null` | Minimum total credit amount |
| `max_credit_amount` | float/null | `null` | Maximum total credit amount |
| `check_credit_min` | bool | `false` | Enable checking the minimum credit amount |
| `check_credit_max` | bool | `false` | Enable checking the maximum credit amount |

---

## 11. Balance Check per Side

### Debit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `debit_check_balance` | bool | `false` | Enable balance check before debiting |
| `debit_balance_check_callback` | callable/null | `null` | Custom balance check function for debit (overrides default) |

### Credit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `credit_check_balance` | bool | `false` | Enable balance check before crediting |
| `credit_balance_check_callback` | callable/null | `null` | Custom balance check function for credit |

---

## 12. Dynamic Account Type Rules per Side

### Debit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `debit_check_account_types` | bool | `false` | Enable account type checking on debit side using rules |
| `debit_rules_account_types` | array/null | `null` | Rules for account type checking on debit side (see explanation below) |

### Credit Side

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `credit_check_account_types` | bool | `false` | Enable account type checking on credit side using rules |
| `credit_rules_account_types` | array/null | `null` | Rules for account type checking on credit side |

---

## 13. Extra Fields

These settings apply to the fields: `extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`.

For each of these fields, there are three sets of settings:

### At the Header Level

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `header_{field}_allowed` | bool | `true` | Allow using the field in the header |
| `header_{field}_required` | bool | `false` | Is the field required in the header? |
| `header_{field}_default` | mixed | `null` | Default value for the field in the header |

### At the Debit Side Detail Level

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `debit_{field}_allowed` | bool | `true` | Allow using the field in the debit side |
| `debit_{field}_required` | bool | `false` | Is the field required on the debit side? |
| `debit_{field}_default` | mixed | `null` | Default value for the field on the debit side |

### At the Credit Side Detail Level

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `credit_{field}_allowed` | bool | `true` | Allow using the field in the credit side |
| `credit_{field}_required` | bool | `false` | Is the field required on the credit side? |
| `credit_{field}_default` | mixed | `null` | Default value for the field on the credit side |

**List of fields:**
- `extend_id`
- `modul_type`
- `ref_type`
- `ref_key_name`
- `ref_key_values`
- `batch_type`
- `batch_id`

---

## 14. Additional Settings (Hooks & Disabled Rules)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `auto_generate_custom_name` | bool | `true` | Automatically generate a custom person name if not passed |
| `beforeValidate` | callable/null | `null` | Function executed before data validation (receives `$options` and `$details`) |
| `afterValidate` | callable/null | `null` | Function executed after data validation (receives `$options` and `$details`) |
| `disabled_rules` | array | `[]` | Array of rule names to disable (e.g., `['balance_check', 'multiplicity']`) |

---

## 15. Entry Details

This key is required and must be passed to create the entry.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `details` | array | `[]` | Array of detail elements, each representing an accounting party. |

### Structure of a single `details` element:

| Key | Type | Description |
|-----|------|-------------|
| `accounts_id` | string | Account code (required) |
| `debit` | float | Debit amount (must be > 0 if this is a debit party, otherwise 0) |
| `credit` | float | Credit amount (must be > 0 if this is a credit party, otherwise 0) |
| `notes` | string | Note specific to this party (optional) |
| `person_id` | int | Associated person ID (optional) |
| `person_type` | string | Person type (Morph Class) (optional) |
| `custom_name_accounts` | string | Custom account name (optional, generated automatically if enabled) |
| `currencys_id` | string | Currency for this party (if different from the header currency) |
| `main_currencys_id` | string | Base currency (usually left to be determined automatically) |
| `rate` | float | Exchange rate for this party (if different from the header rate) |
| `extend_id` | mixed | (optional) |
| `modul_type` | mixed | (optional) |
| `ref_type` | mixed | (optional) |
| `ref_key_name` | mixed | (optional) |
| `ref_key_values` | mixed | (optional) |
| `batch_type` | mixed | (optional) |
| `batch_id` | mixed | (optional) |
| `person` | object | (optional) Person object can be passed instead of id/type |

---

## Explanation of `debit_rules_account_types` and `credit_rules_account_types`

The rules can come in two formats:

### Simplified Format (automatic AND)
```php
$rules = [
    ['field' => 'reports', 'operator' => '=', 'value' => '1'],
    ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
];
```

### Advanced Format with `operator`
```php
$rules = [
    'operator' => 'OR',   // or 'AND'
    'rules'    => [
        ['field' => 'reports', 'operator' => '=', 'value' => '1'],
        ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Cash']],
    ],
];
```

### Supported Fields from the `accounts` Table:
- `reports`, `final_account_id`, `modul_type`, `account_normal`, `groups_accounts_id`, `ref_type`, `person_type`, `account_number`, `parent_id`, `level`, `main_sub`, and others.

### Supported Operators:
- `=`, `==`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.

---

## Examples of `balance_check_callback`

The callback function receives the following parameters:
- `$account`: `Account` object
- `$amount`: Required amount
- `$balance`: Current balance
- `$personId`: Person ID (if any)
- `$personType`: Person type (if any)
- `$context`: General entry options

**Example:**
```php
'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
    if ($balance < $amount) {
        throw new ApplicationException("Insufficient account balance");
    }
}
```

---

## Concluding Notes

- Default values for company ID, department ID, cost center, and period are automatically fetched from helper functions (`getCheckCompanysId`, `getCheckDepartmentsId`, etc.) based on user context or general settings.
- If `use_default_company` is enabled in the add-on settings, the input values will be overridden by the company's default values.
- Any of these options can be passed optionally, and they will be merged with the default values defined in `mergeDefaultCompanyValues`.

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
