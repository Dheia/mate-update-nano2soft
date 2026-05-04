## 2026-02-17 – 2026-05-01

**Comprehensive Update of `Nano.AccountsApi` and `Tss.Accounts` – Version 1.0.38**

### Adding Support for Multi‑Party Journal Entries, Exchange Differences, and Advanced Validation

---

### Summary of Updates

In version **1.0.38**, a qualitative leap was made in the accounting infrastructure of the core add‑ons (`Tss.Accounts`) and the API layer (`Nano.AccountsApi`), including:

- **Addition of the `TransactionsMultiHelper` trait** which provides an integrated set of tools for creating multi‑party journal entries with a comprehensive and customisable validation system.
- **Full support for exchange differences** through the functions `checkExchangeDifferences`, `createExchangeDifferenceEntry` and `handleExchangeDifferences`, allowing automatic creation of exchange difference settlement entries.
- **Extended control options** including amount limits (total and per side), balance checks, dynamic rules for checking account types, and control over extra fields (e.g. `modul_type`, `batch_id`).
- **Updated `TransactionsHelper`** to integrate the new trait, and provided the unified `createEntry` function that automatically chooses between a simple entry and a multi‑party entry.
- **Improved `paper_id` management** with options for auto‑generation and duplicate checking.
- **Helper functions for testing and simulation** such as `testJournalEntry` to analyse an entry before creating it.
- **Minor updates to `AccountHelper`** to ensure compatibility with new types.

All these improvements directly affect `Nano.AccountsApi`, where `BaseDocumentController` and all new controllers (BondsDays, CatchReceiptsV2, …) benefit from them.

---

### Version Goals

- **Support complex accounting entries** (more than two parties) natively and securely via the API.
- **Automate exchange difference processing** resulting from changes in exchange rates between the entry date and the posting date.
- **Provide fine‑grained control over every detail** (person, account, notes, extra fields) for each side of the entry.
- **Increase data reliability and integrity** through multiple checks (balance, amount limits, account type, currency balance).
- **Make it easy to test entries** before saving them to the database using the `testJournalEntry` function.
- **Maintain backward compatibility** with existing functions (`createSingle`, `addPayment`, etc.).

---

### New Features and Improvements

#### 1. `TransactionsMultiHelper` Trait (New)

A new trait `TransactionsMultiHelper` was added in the file `Tss\Accounts\Helpers\TransactionsMultiHelper.php`, containing all the functions related to multi‑party entries. It was designed to be easy to use and fully integrated with `TransactionsHelper`.

**Main functions:**

| Function | Description |
|----------|-------------|
| `createJournalEntry` | Creates a multi‑party journal entry (3+ parties) with full validation, currency handling, exchange differences, and fires `beforeJournalEntry` and `afterJournalEntry` events. |
| `createEntry` | Unified function that automatically chooses between `createSingle` (for two parties) and `createJournalEntry` (for more than two parties); can force the use of `createJournalEntry` via the option `forceJournal => true`. |
| `validateJournalEntry` | Validates multi‑party entry data without creating the entry, returning the processed details. |
| `checkExchangeDifferences` | Checks an existing entry for exchange differences (based on the posting date and official exchange rate), with options to create (`entry`), fire an event (`event`) or only return data (`none`). |
| `createExchangeDifferenceEntry` | Creates a settlement entry for a single exchange difference, linking it to the original entry. |
| `handleExchangeDifferences` | Internal processing called after an entry is created to check for differences and take the specified action. |
| `testJournalEntry` | Fully simulates the creation of an entry (including validation and currency processing) with detailed analysis of results and rule violations. |
| `applyAccountRules` | Applies a set of dynamic rules to an account to check its type or properties (e.g. `reports = 1` and `modul_type IN ['Boxe','Bank']`). |
| `checkAccountBalance` | Checks the balance of a given account before executing the transaction. |
| `processCurrency` | Processes the currency and exchange rate for each detail, with support for automatic conversion (`auto_convert_currency`). |
| `calculateTotals` | Calculates the totals (base and foreign) of the details. |
| `validateMultiplicity` | Checks party count constraints (maximum number, allowing multiple debit/credit parties). |
| `validateAmountLimits` | Checks total and per‑side amount limits. |
| `validateBalance` | Enables the balance check for each detail according to the settings. |
| `validatePaperId` / `preparePaperId` | Manages automatically generating and validating `paper_id`. |

**Features of the trait:**

- **Full integration** with `TransactionsHelper` – just add `use TransactionsMultiHelper;`.
- **Uses `Db::transaction`** to guarantee data integrity in `createJournalEntry`.
- **Supports `beforeJournalEntry` and `afterJournalEntry` events**.
- **Supports `beforeValidate` and `afterValidate` as hooks**.
- **Allows disabling specific rules** via `disabled_rules` (e.g. `'balance_check'`, `'multiplicity'`).
- **Unified options structure** covering all possible settings for the header and details.

#### 2. Comprehensive and Customisable Settings in `createJournalEntry`

The new trait provides complete control over every aspect of an accounting entry through the `$options` array. The main groups of supported settings are listed below (see the file `createJournalEntry-config.md` for the complete list):

- **Basic header fields:** `companys_id`, `departments_id`, `date_at`, `transactions_type`, etc.
- **Person at header level:** `header_person_id`, `header_person_type`, `header_person`.
- **Party count limits:** `max_debit_entries`, `max_credit_entries`, `allow_multiple_debit`, `allow_multiple_credit`.
- **Person control per side:** `debit_person_allowed`, `debit_person_required`, `debit_person_allowed_types` (and their credit counterparts).
- **Account control per side:** `debit_account_allowed`, `debit_account_required`, `debit_default_account`.
- **Notes control per side:** `debit_notes_allowed`, `debit_notes_required`, `debit_default_notes`.
- **Amount limits:** `min_total_amount`, `max_total_amount`, `check_debit_min`, `min_debit_amount`, etc.
- **Balance checking:** `debit_check_balance` and `debit_balance_check_callback` (with support for custom callbacks).
- **Account type checking:** `debit_check_account_types` and `debit_rules_account_types` (dynamic rules supporting `AND/OR` and operators such as `=`, `IN`, `LIKE`, `REGEXP`).
- **Extra field control:** `header_extend_id_allowed`, `debit_batch_id_required`, `credit_modul_type_default`, etc. (covering the fields: `extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`).
- **`paper_id` settings:** `is_paper_id`, `paper_id_allowed`, `paper_id_auto`, `paper_id_required`, `paper_id_check_duplicate`, `paper_id_duplicate_scope`.
- **Date settings:** `is_custome_date`, `date_at_auto`, `date_at_required`.
- **Currencies and exchange differences:** `auto_convert_currency`, `handle_exchange_differences`, `exchange_difference_action` (`'entry'`, `'event'`, `'none'`).
- **Hooks:** `beforeValidate`, `afterValidate`.
- **Disabling rules:** `disabled_rules`.

#### 3. Integration of `TransactionsMultiHelper` with `TransactionsHelper`

- The line `use \Tss\Accounts\Helpers\TransactionsMultiHelper;` was added inside the `TransactionsHelper` class, making all the functions of the new trait available as part of the class.
- A minor modification was made to `createSingle` to initialise `$result = []` to avoid errors in various scenarios.

#### 4. Exchange Differences Support

Integrated mechanisms were added to handle exchange differences resulting from changes in exchange rates between the entry date and the posting date:

- **`checkExchangeDifferences`**: Accepts an entry ID or object, inspects every detail with a non‑base currency, and compares its original rate with the official rate at the posting date.
- **`createExchangeDifferenceEntry`**: Creates an accounting entry to settle the difference (debit/credit) between the exchange difference account and the original account.
- **`handleExchangeDifferences`**: Automatically called after a successful `createJournalEntry` if the option `handle_exchange_differences = true`.
- **Control options:** `threshold` (minimum difference threshold), `action` (`'entry'`, `'event'`, `'none'`), `force_check`, `relay_date`, `exchange_difference_account`.

#### 5. Improvements to `AccountHelper`

Although the changes are minor, `AccountHelper::getObjHeaderFromType` now supports class names in addition to string type identifiers (e.g. `'Transfer'` and `Transfer::class`), which improves compatibility with the new controllers.

#### 6. Impact of the Updates on `Nano.AccountsApi`

- **`BaseDocumentController`**: Now uses `TransactionsHelper::createEntry` (brought in from the trait) instead of directly calling `createSingle`, allowing the creation of entries with any number of parties.
- **`BondsDays`**: Fully benefits from `createJournalEntry` via `createEntry`, with balance checking and preventing posting to main accounts enabled.
- **All controllers** (`CatchReceiptsV2`, `PayReceipts`, etc.) can now accept multiple details and pass advanced options.
- **The `test` function in `BondsDays`** uses `TransactionsHelper::testJournalEntry` to test an entry before saving it.

---

### Usage Examples

#### Create a multi‑party entry with amount limits and account type checking

```php
$result = TransactionsHelper::createEntry([
    'date_at' => '2026-05-18',
    'notes'   => 'Settlement entry',
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 3000],
        ['accounts_id' => '2-2-4111010001', 'credit' => 2000],
    ],
    'min_total_amount' => 1000,
    'max_total_amount' => 10000,
    'debit_check_account_types' => true,
    'debit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
    ],
]);
```

#### Test an entry before creating it

```php
$test = TransactionsHelper::testJournalEntry([
    'details' => [...],
]);
if ($test['analysis']['test_passed']) {
    echo "✅ The entry is valid and ready to be saved.";
} else {
    print_r($test['analysis']['rule_violations']);
}
```

#### Check exchange differences for an existing entry and create a settlement entry

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'threshold' => 0.01,
    'action'    => 'entry',
    'relay_date' => Carbon::parse('2026-05-20'),
]);
if ($result['exchange_entry']) {
    echo "Exchange difference entry created: " . $result['exchange_entry']->id;
}
```

---

### Benefits and Added Value

- **Unprecedented flexibility**: Developers can now create entries with any number of parties, with fine‑grained control over each side, opening the door to advanced accounting applications.
- **Security and accuracy**: Balance checks, account type checks and amount limits reduce the risk of accounting errors.
- **Process automation**: Exchange differences are handled automatically without manual intervention, saving time and preventing mistakes.
- **Ease of development**: Thanks to helper functions such as `testJournalEntry`, developers can test entries before sending them.
- **Full backward compatibility**: All existing functions continue to work with the same signatures, and new features can be adopted incrementally.
- **Improved performance of `Nano.AccountsApi`**: The new controllers can handle the complex scenarios required by modern financial systems.

---

### Upgrade Requirements

1. **Update the code**:
   - Replace the following files in `Tss.Accounts`:
     - `helpers/TransactionsHelper.php`
     - Add the new file `helpers/TransactionsMultiHelper.php`
     - (Optional) `helpers/AccountHelper.php` if the minor updates are needed.
   - Update `Nano.AccountsApi` to the version that uses `createEntry` (usually `BaseDocumentController` and the controllers built on it – if you haven’t updated them before, you will need version 1.1.0 first or merge the features together).

2. **No new database migrations**.
3. **Clear the cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
4. **Test the functionality**:
   - Try creating a multi‑party entry using `bonds/add`.
   - Use `bonds/test` to validate the entry.
   - If you use multiple currencies, verify that exchange differences are handled correctly.

---

### Version Summary

| Version | Key Features and Fixes |
| :--- | :--- |
| 1.1.0 | Added `BaseDocumentController` and new document types (BondsDays, PayReceipts, DebitNotes, CreditNotes, Transfers, CatchReceiptsV2). |
| 1.0.38 | Added `TransactionsMultiHelper`: multi‑party entries, exchange differences, advanced checks (balance, account type, amount limits), comprehensive settings, `testJournalEntry` function. |

---

### Conclusion

Version 1.0.38 represents a qualitative leap in the power and flexibility of the accounting system on the NanoSoft platform. With `TransactionsMultiHelper`, developers can now easily and securely create complex accounting entries, covering real‑world use cases such as exchange differences and account validation. These features translate directly into `Nano.AccountsApi`, providing a rich and powerful API for financial applications.

We thank you for your continued support and look forward to your feedback for further development.

---

### Additional Documentation

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

