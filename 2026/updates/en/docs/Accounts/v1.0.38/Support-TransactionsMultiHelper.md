## Summary of Updates to `TransactionsHelper` and `TransactionsMultiHelper`

### 1. Added a unified function for creating accounting entries
- **`createEntry`**: A unified function that automatically chooses between:
  - `createSingle` (for two‑party entries).
  - `createJournalEntry` (for multi‑party entries, more than two parties).
  - You can force the use of `createJournalEntry` via the option `forceJournal => true`.

### 2. Multi‑party journal entry creation function (`createJournalEntry`)
- Creates entries with an unlimited number of parties (debit/credit).
- Maintains the same result array structure used in `createSingle`, adding `input_data` and `process_data`.
- Uses `Db::transaction` to ensure data integrity.

### 3. Validation improvements
- Checks for the existence of the account (`Account::where('code')`) and its activity (`is_active`).
- Prevents posting to main accounts (optional via settings).
- Checks the entry balance (total debit = total credit with a tolerance of 0.001).
- Checks the accounting period validity (`Period::isValidDate`) and that it is open (`Period::isOpen`).

### 4. Limits on the number of parties per side
- `max_debit_entries`, `max_credit_entries`: specify the maximum number of parties.
- `allow_multiple_debit`, `allow_multiple_credit`: allow or prevent multiple parties on a given side.

### 5. Control over the use of a person
- **For each side**:
  - `debit_person_allowed` / `credit_person_allowed`: allow linking a person.
  - `debit_person_required` / `credit_person_required`: require a person.
  - `debit_person_allowed_types` / `credit_person_allowed_types`: specify allowed person types (array of class names).
- Handles a `person` object if passed directly.

### 6. Control over the account
- **For each side**:
  - `debit_account_allowed` / `credit_account_allowed`: allow specifying an account.
  - `debit_account_required` / `credit_account_required`: require an account.
  - `debit_default_account` / `credit_default_account`: default account code used if not specified.
- Checks the account’s existence and activity after applying rules.

### 7. Control over notes
- **For each side**:
  - `debit_notes_allowed` / `credit_notes_allowed`: allow entering a note.
  - `debit_notes_required` / `credit_notes_required`: require a note.
  - `debit_default_notes` / `credit_default_notes`: default note.

### 8. Amount limits
- **Entry total**:
  - `min_total_amount`, `max_total_amount`, `check_min_total`, `check_max_total`.
- **For each side**:
  - `min_debit_amount`, `max_debit_amount`, `check_debit_min`, `check_debit_max`.
  - `min_credit_amount`, `max_credit_amount`, `check_credit_min`, `check_credit_max`.

### 9. Balance checking
- **For each side**:
  - `debit_check_balance` / `credit_check_balance`: enable the check.
  - `debit_balance_check_callback` / `credit_balance_check_callback`: custom callback function to override the default check (receives: `$account`, `$amount`, `$balance`, `$personId`, `$personType`, `$context`).
- The default check uses `AccountHelper::getBalances` taking the person into account if present.

### 10. Dynamic account type checking with rules
- **For each side**:
  - `debit_check_account_types` / `credit_check_account_types`: enable the check.
  - `debit_rules_account_types` / `credit_rules_account_types`: an array of rules.
- **Rule format**:
  ```php
  [
      'operator' => 'AND|OR', // optional, default AND
      'rules' => [
          ['field' => 'account_number', 'operator' => 'LIKE', 'value' => '1253%'],
          ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Bank']],
          // ...
      ]
  ]
  ```
- **Supported fields**: all columns of the `accounts` table (e.g., `reports`, `final_account_id`, `modul_type`, `ref_type`, `person_type`, `account_number`, `level`, `parent_id`, ...).
- **Supported operators**: `=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.
- Rules are applied after checking the account’s existence and before the balance check.

### 11. Control over extra fields (`extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`)
- **At the header level**:
  - `header_extend_id_allowed`, `header_extend_id_required`, `header_extend_id_default` (and similarly for the other fields).
- **At the detail level for each side**:
  - `debit_extend_id_allowed`, `debit_extend_id_required`, `debit_extend_id_default` ... (for debit side).
  - `credit_extend_id_allowed`, `credit_extend_id_required`, `credit_extend_id_default` ... (for credit side).
- The rules (allowed/required/default) are applied independently for each field.

### 12. Added events
- `tss.accounts.beforeJournalEntry`: fired before the transaction begins, receives `$options` and `$processedDetails`.
- `tss.accounts.afterJournalEntry`: fired after the transaction succeeds, receives `$model`, `$options`, `$processedDetails`.

### 13. Improved error handling and reporting
- Returns `input_data` and `process_data` in the result array to facilitate tracking.
- Logs errors in the log file using `Log::error`.
- Detailed, translatable error messages with the specific detail number and side.
- Supports `debug` in development mode.

### 14. Improved code structure and reusability
- Separate helper functions:
  - `validateJournalDetails`
  - `validateSingleDetail`
  - `validateSideField` (for simple fields)
  - `applyAccountRules` (applies account type rules)
  - `compareAttribute` (compares values with operators)
  - `checkAccountBalance` (balance checking)
- Uses `array_merge` to merge default values with passed options.

### 15. Updated `BaseDocumentController` to take advantage of the new features
- Uses `createEntry` instead of `createSingle`.
- Passes `details` directly (with automatic conversion from the old format `debit_accounts_id`/`credit_accounts_id`).
- Merges type settings (e.g., `bonds_days`) with the general default settings (`journal_entry_defaults`).

### 16. Comprehensive configurable settings
- Provides a `journal_entry_defaults` array in the `config.php` file containing all the above settings with their default values.
- Can be customised per document type (e.g., in `bonds_days`, `catch_receipts`) via the configuration file for that type.

---

With this, the system is able to create complex accounting entries with full control over all aspects (permissions, accounts, persons, amounts, balance, account types, extra fields) with high flexibility and extensibility.
