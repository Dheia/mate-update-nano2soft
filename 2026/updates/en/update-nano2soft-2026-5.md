# Update 2026-5

**Updates month four May**

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
- [`AccountHelper` Class Documentation](./docs/Accounts/Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./docs/Accounts/Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./docs/Accounts/Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./docs/Accounts/Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./docs/Accounts/Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./docs/Accounts/Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./docs/Accounts/Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)


## 2026-05-01 - 2026-05-02

**`Nano3.Kyc` Plugin Updates – Version 1.2.9**

### Summary of Updates

Version 1.2.9 represents a fundamental leap in how KYC documents are linked to various models within NanoSoft applications. While previous versions focused on managing the documents themselves (creation, update, verification, assessment), this version introduces an **integrated system for managing KYC integrations** with any model (users, customers, companies, employees...).

Instead of writing repetitive code for each model, the new version provides:

- A ready-made **`KycDocumentModel` behavior** that can be added to any model to give it a `documents` relationship, advanced filtering scopes, and ready-to-use statistics.
- A **`KycDocumentManager` class** (Singleton) for centralized management of all integrations (behavior, forms, lists, filters, API queries, transformers).
- An **`KycIntegrationHandler`** to automatically bind events and apply integrations without manual intervention.
- **Intelligent handling of API parameters** supporting multiple formats (comma, pipe, JSON, auto-detect) and complex groups with `AND`/`OR` logic.

All of this is accompanied by comprehensive and detailed documentation in both Arabic and English.

---

### Version 1.2.9 – Integrated KYC Integrations System

#### Version Goals

- **Provide a unified behavior** (`KycDocumentModel`) that adds the document relationship, filtering scopes, and statistics to any model.
- **Create a central management class** (`KycDocumentManager`) to register models and manage all types of integrations declaratively.
- **Automate application** via `KycIntegrationHandler` that automatically binds events.
- **Support advanced API filtering** with multiple value processors, complex groups, and `AND`/`OR` logic.
- **Professional documentation** covering all new classes with diverse practical examples.

#### New Features

##### 1. `KycDocumentModel` Behavior with `KycDocumentScopesAndHelpers` Trait

A new behavior `Nano3\Kyc\Behaviors\KycDocumentModel` was created that can be added to any model. The behavior automatically:

- Defines a `documents` relationship (morphMany) with the `Document` model via the `owner` field.
- Provides advanced scopes:
  - `scopeWhereHasDocuments` – filter by any document properties (type, category, status, verification, expiry date...) with support for `count`, `boolean`, and `not`.
  - `scopeHasVerifiedDocuments`, `scopeHasDocumentType`, `scopeHasDocumentCategory`, `scopeHasDocumentStatus` – shorthand scopes for common usage.
  - `scopeAddDocumentsCount` – adds a document count column to query results.
  - `scopeSortByDocumentsCount` – orders results by document count.
- Statistical accessors:
  - `getDocumentsCountAttribute` – total number of documents.
  - `getVerifiedDocumentsCountAttribute` – number of verified documents.
  - `getPendingDocumentsCountAttribute` – number of pending documents.
- Quick check functions:
  - `hasAnyDocument()`, `hasVerifiedDocument()`, `hasDocumentOfType($type)`.

**Example of direct usage after applying the behavior:**

```php
// Filter users who have a verified primary identity document
User::whereHasDocuments([
    'category'    => 'primary_id',
    'is_verified' => true,
])->get();

// Get a user with document count
$user = User::find(1);
echo $user->documents_count;          // 5
echo $user->verified_documents_count; // 3

// Quick check
if ($user->hasVerifiedDocument()) {
    // allow action
}
```

##### 2. `KycDocumentManager` – Central Integration Manager

The `Nano3\Kyc\Classes\KycDocumentManager` class was built as a Singleton to manage all KYC document integrations flexibly and declaratively. It supports **six types of integrations** that can be enabled or disabled per model:

| Integration | Description |
| :--- | :--- |
| `INTEGRATION_BEHAVIOR` | Applies the `KycDocumentModel` behavior to the model. |
| `INTEGRATION_FORM_FIELD` | Adds a custom `partial` field in backend forms. |
| `INTEGRATION_LIST_COLUMN` | Adds a `documents_count` column in backend lists. |
| `INTEGRATION_LIST_FILTER` | Adds a filter by document status in backend lists. |
| `INTEGRATION_QUERY` | Automatic filtering in API and report queries. |
| `INTEGRATION_TRANSFORMER` | Injects `DynamicAddIncludeKyc` into the model's Transformer. |

**Simple model registration:**

```php
$manager = KycDocumentManager::instance();
$manager->registerModel(User::class, [
    'integrations' => [
        KycDocumentManager::INTEGRATION_QUERY => true,
    ],
]);
```

**Or quick registration with simplified options:**

```php
$manager->quickRegister(User::class, null, [
    'with_form'   => true,
    'with_filter' => true,
]);
```

##### 3. Automatic Integration Handler `KycIntegrationHandler`

The `Nano3\Kyc\EventHandlers\KycIntegrationHandler` class binds all events automatically:

- Listens to `eloquent.booting: *` to apply the behavior to registered models.
- In the backend: adds form fields, columns, and filters automatically.
- Always: enables API and Transformer integrations.

Simply register it once:

```php
Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
```

##### 4. Intelligent Handling of API Parameters

`KycDocumentManager` supports extracting and analyzing KYC filter values from API requests in multiple ways:

- **Wildcards**: values `*`, `all`, `ALL` are ignored (no filter applied).
- **Value Processors**: keywords like `verified`, `pending`, `any`, `none` execute custom processors.
- **Multi-value Handlers**: supports `comma`, `pipe`, `json`, `array`, and `auto` for automatic detection.
- **Complex Groups**: values containing `|` (pipe) are split into groups that are combined using `OR`, and within each group `,` (comma) can be used with `AND` logic.

**Examples of supported `kyc_status` values:**

| Value | Interpretation |
| :--- | :--- |
| `verified` | Applies the `verified` processor (verified documents). |
| `verified,pending` | Verified or pending documents (AND). |
| `verified\|pending` | Verified or pending documents (OR). |
| `verified,pending\|rejected` | (verified AND pending) OR (rejected). |

##### 5. `createAdvancedQuery` Function

To build ready-made queries:

```php
$query = KycDocumentManager::instance()->createAdvancedQuery(
    User::class,
    ['kyc_status' => 'verified|pending'],
    [
        'with_documents'  => true,
        'documents_count' => true,
    ]
);
$users = $query->get();
```

#### Additional Improvements

- **General helper functions**: `checkValueIsNotAll`, `scopeWhereField`, `parseComplexFilter`, `applyLogicOperator`.
- **Custom events**: `nano3.kyc.modelRegistered`, `nano3.kyc.beforeQueryProcessing`, `nano3.kyc.afterQueryProcessing`, and more.
- **Comprehensive documentation**: complete documentation files in Arabic and English were created for both `KycDocumentManager` and `KycDocumentModel` with advanced practical examples.

---

### Practical Examples

#### 1. Registering a Customer Model with All Integrations

```php
KycDocumentManager::instance()->registerModel(Customer::class, [
    'controller' => CustomersController::class,
    'integrations' => [
        KycDocumentManager::INTEGRATION_BEHAVIOR    => true,
        KycDocumentManager::INTEGRATION_FORM_FIELD  => true,
        KycDocumentManager::INTEGRATION_LIST_COLUMN => true,
        KycDocumentManager::INTEGRATION_LIST_FILTER => true,
        KycDocumentManager::INTEGRATION_QUERY       => true,
        KycDocumentManager::INTEGRATION_TRANSFORMER => true,
    ],
    'form_config' => ['tab' => 'KYC'],
]);
```

#### 2. Using Scopes in a Custom Report

```php
$expiringUsers = User::whereHasDocuments([
    'status'     => 'verified',
    'has_expiry' => true,
    'conditions' => function ($q) {
        $q->where('expiry_date', '>=', now())
          ->where('expiry_date', '<=', now()->addDays(30));
    }
])->get();
```

#### 3. API Filtering with Complex Parameters

```http
GET /api/v1/users?kyc_status=verified,pending|rejected
```

`KycDocumentManager` automatically analyzes the value and applies the appropriate query without any additional code.

---

### Version Summary (1.2.8 – 1.2.9)

| Version | Key Features and Fixes |
| :--- | :--- |
| 1.2.8 | Radical fix for field merging and validation rules, deletion of unwanted fields (`null`), intelligent merging of validation rules, prevention of default field leakage. |
| 1.2.9 | **Integrated KYC integrations system**: `KycDocumentModel` behavior, `KycDocumentManager` class, `KycIntegrationHandler`, intelligent API parameter handling, comprehensive documentation. |

---

### Upgrade Requirements

1. **Update code**:
   - Add the new files:
     - `behaviors/KycDocumentModel.php`
     - `behaviors/KycDocumentModel/KycDocumentScopesAndHelpers.php`
     - `classes/KycDocumentManager.php`
     - `eventhandlers/KycIntegrationHandler.php`
   - Register `KycIntegrationHandler` in `Plugin.php`:
     ```php
     Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
     ```

2. **Register models**:
   - Add a `registerKycDocumentIntegrations` function to your Plugin to register the models you want to link with KYC documents.

3. **Test integrations**:
   - Ensure that the `whereHasDocuments` scopes work on registered models.
   - Try API filtering using `kyc_status` with various values.
   - Check the appearance of fields and filters in the control panel (if enabled).

4. **Clear cache** (optional):
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### Conclusion

Version 1.2.9 is a pivotal release in the roadmap of the `Nano3.Kyc` add-on, moving it from a mere KYC document management tool to an **integrated ecosystem** that can be easily linked with any model on the platform. Thanks to `KycDocumentManager` and the `KycDocumentModel` behavior, developers can enable advanced integrations with just a few lines of code, benefiting from intelligent parameter handling and automatic generation of user interface elements.

We appreciate your continued follow-up and look forward to your feedback to develop more features in future releases.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`KycDocumentManager` Class Documentation](./docs/Kyc/Docs-KycDocumentManager-Class-en.md)
- [`KycDocumentModel` Behavior Documentation](./docs/Kyc/Docs-KycDocumentModel-Behaviors-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./docs/Kyc/Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)

## 2026-05-02 - 2026-05-03

### Comprehensive Update to Order Filtering Scopes by Product Owner in the `Order` Model

**Plugin:** `Nano.Orders`  
**File:** `HasProductOwnerScopes` trait (`plugins/nano/orders/models/Order/HasProductOwnerScopes.php`)

---

### Development of Advanced Query Scopes for Filtering Orders Based on Product Ownership

The `HasProductOwnerScopes` trait was developed and added to the `Order` model, granting developers the ability to filter orders with high flexibility based on products owned by a specific user. Developers no longer need to write complex subqueries or manually handle nested relationships; instead, they can use ready-made scopes that support:

- Filtering by **product owner** (`user_type` and `user_id` in the `tss_inventory_products` table).
- Specifying the **number of owned products** within the order (using `count` and `countOperator`).
- **Negation** (retrieving orders that do not contain products owned by the owner).
- **Boolean combination** (`AND` / `OR`) to link conditions with other scopes.
- **Additional conditions** on the product itself (price, status...) or on the order item (quantity, actual price...).
- **Adding a computed column** counting owned products in the order, and **sorting** by it.
- **Quick check functions** on a single order object (e.g., `hasAnyProductByOwner`).

All scopes and helper functions were moved to a separate trait to facilitate maintenance and reuse, while adhering to using table name constants to ensure query stability.

---

### 1. Developed Components

| Behavior | Component | Description |
|----------|-----------|-------------|
| `Order` (Model) | `HasProductOwnerScopes` (trait) | Contains all scopes and helper functions for filtering orders by product owner. |

#### Main and Auxiliary Scopes

| Category | Scope | Description |
|----------|-------|-------------|
| **Advanced Scope** | `scopeWhereHasProductsByOwner` | The main scope with a comprehensive options array (`user`, `count`, `boolean`, `not`, `conditions`, `itemConditions`). |
| **Auxiliary Scopes** | `scopeWhereProductOwner` | Quick shortcut: orders containing at least one product owned by the owner. |
| | `scopeHasProductsByOwner` | Filter by a minimum number of owned products. |
| | `scopeDoesntHaveProductsByOwner` | Filter orders that contain no owned product. |
| | `scopeHasProductsByOwnerCount` | Exact (or compared) count of owned products. |
| | `scopeOrWhereHasProductsByOwner` | Link scope with `OR`. |
| **Computed Columns & Sorting** | `scopeWithProductsByOwnerCount` | Add `products_owner_count` column to `SELECT`. |
| | `scopeSortByProductsByOwnerCount` | Sort orders ascending/descending by number of owned products. |
| **Check Functions** | `hasAnyProductByOwner` | Check whether an order contains at least one product owned by the user. |

---

### 2. Code Update Details

#### 2.1 Main Scope `whereHasProductsByOwner`
This scope is designed to be comprehensive and accepts a single options array, unifying the usage interface and eliminating the need for dozens of separate scopes. Supported options:

| Option | Type | Description |
|--------|------|-------------|
| `user` | `Authenticatable\|Model` | The owner user object. |
| `userType` | `string` | User type (e.g., `RainLab\User\Models\User`). |
| `userId` | `int` | User ID. |
| `boolean` | `string` | `'and'` or `'or'` for linking with an outer query. |
| `not` | `bool` | `true` for negation (`doesntHave`). |
| `count` | `int` | Required number of owned products. |
| `countOperator` | `string` | Comparison operator (`>=`, `=`, `<`...). |
| `conditions` | `array\|callable` | Additional conditions on the products table. |
| `itemConditions` | `array\|callable` | Additional conditions on the order items table. |

If no owner data is passed, the scope automatically attempts to fetch the currently logged-in user via `AuthHelpers` or `BackendAuth` or `Auth`. If the owner cannot be determined, the query is returned without applying any filter, protecting against unintended empty results.

#### 2.2 Auxiliary Scopes
The auxiliary scopes (`HasProductsByOwner`, `DoesntHaveProductsByOwner`, `HasProductsByOwnerCount`, `OrWhereHasProductsByOwner`) are mere wrappers that call the main scope with the appropriate options. This ensures no code duplication and centralizes maintenance.

#### 2.3 Computed Columns and Sorting
- `withProductsByOwnerCount`: Adds a computed column counting owned products in the order using `selectSub`.
- `sortByProductsByOwnerCount`: Sorts results by that column using `orderByRaw` with safe binding passing to avoid errors from using `DB::raw` without bindings.

#### 2.4 Check Functions
- `hasAnyProductByOwner($user)`: Quick function on an `Order` object to check for at least one product owned by the user (or the current user automatically).

#### 2.5 Using Table Constants
To avoid errors when table names change:
- `\Nano\Shop\Models\Product::TABLE_NAME` is used to refer to the products table.
- `(new \Nano\Orders\Models\OrderItem)->getTable()` is used for the order items table.

Also, the table name is prefixed to each field in conditions to avoid ambiguity in queries joining more than one table.

---

### 3. Practical Examples

#### 3.1 Orders containing any product owned by the current user
```php
$orders = Order::hasProductsByOwner()->get();
```

#### 3.2 Orders not containing any product owned by a specific user
```php
$user = \BackendAuth::getUser();
$orders = Order::doesntHaveProductsByOwner($user)->get();
```

#### 3.3 Orders containing exactly 3 owned products with an additional condition on the product (price > 50)
```php
$orders = Order::whereHasProductsByOwner([
    'user'       => $user,
    'count'      => 3,
    'countOperator' => '=',
    'conditions' => function ($q) {
        $q->where('price', '>', 50);
    }
])->get();
```

#### 3.4 Orders containing an owned product with quantity greater than 2 in the order
```php
$orders = Order::whereHasProductsByOwner([
    'user'           => $user,
    'itemConditions' => function ($q) {
        $q->where('quantity', '>', 2);
    }
])->get();
```

#### 3.5 Combining with order status using OR
```php
$orders = Order::isNewOrders()
    ->orWhereHasProductsByOwner(['user' => $user])
    ->get();
```

#### 3.6 Adding the owned products count column and sorting descending
```php
$orders = Order::withProductsByOwnerCount()
    ->sortByProductsByOwnerCount('desc')
    ->get();

foreach ($orders as $order) {
    echo $order->products_owner_count;
}
```

#### 3.7 Checking a single order
```php
$order = Order::find(10);
if ($order->hasAnyProductByOwner($user)) {
    // Processing
}
```

---

### 4. Added Value

- **For developers**: A unified and flexible interface covering all order filtering scenarios based on product ownership, without needing complex subqueries. Advanced options like `count`, `not`, `conditions`, and `itemConditions` open broad possibilities for reports and dashboards.
- **For the application**: Improved performance using `whereHas` and `whereHas` with `count`, and secure subquery bindings. Reliance on table constants prevents errors when the structure changes.
- **For end users**: The ability to build smart lists such as "Orders containing my products" or "Orders from my customers (via my products)" enhances the user experience in multi-vendor marketplace systems.
- **Flexibility**: Boolean combination (`OR`) allows easily mixing conditions with other scopes (order status, type, date...).
- **Extensibility**: New options can be easily added to the main scope, or additional auxiliary scopes can be derived in the same pattern.

---

### 5. Conclusion

This update represents a qualitative leap in managing order queries related to product ownership within the `Nano.Orders` plugin. Through an independent trait and advanced scopes, developers can now build complex filtering systems easily and with high performance, while maintaining full compatibility with the existing structure. The provided documentation ensures quick comprehension and application.

---

**Note**: For deeper technical details, refer to the `HasProductOwnerScopes.php` file and the comments attached to the scopes.

**Reference Documentation**:
- [Trait HasProductOwnerScopes Documentation](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-en.md)

## 2026-05-03 - 2026-05-04

**`Nano3.Kyc` Plugin Updates – Version 1.3.0**

### Summary of Updates

Version 1.3.0 completes the KYC integration system that began in 1.2.9, moving it to a **fully production‑ready** stage with extensive support for creating advanced filters in the control panel, multiple customizable columns, professional Toggle scopes, and centralized configuration management via Config files.

This release focuses on:

- **Advanced Toggle scopes** inspired by the `AssociationsModel` behavior, supporting `switch` and `checkbox` filters.
- **Multiple group filters** with `dependsOn` support for building connected filters (e.g., document category → document type).
- **Multiple list columns** (document counts, is_verifier_*) with full OctoberCMS options.
- **Centralized default settings** via the `config/extend.php` file, allowing customization of columns, filters, and settings without modifying code.
- **Automatic registration of user models** (Frontend and Backend) with basic integrations.
- **Improved Transformer integration** to support `default_include` and `eager_load` for documents.

All of this is accompanied by expanded translation files covering all new labels, and improvements to advanced query processing.

---

### Version 1.3.0 – Full Maturity of KYC Document Integrations

#### Version Goals

- **Expand the `KycDocumentModel` behavior** with Toggle scopes compatible with OctoberCMS filter mechanisms.
- **Support multiple advanced filters** in the control panel (group, checkbox, switch) with dynamic options and dependencies between them.
- **Centralize integration settings** in the `config/extend.php` file to facilitate customization and management.
- **Support multiple columns** in lists (document counts by type, category indicators).
- **Improve Transformer integration** and automatically link common models.

#### New Features and Improvements

##### 1. Advanced Toggle Scopes in `KycDocumentModel`

A set of scopes supporting switch and checkbox filters in OctoberCMS has been added, allowing a smooth filtering experience:

| Scope | Type | Description |
| :--- | :--- | :--- |
| `scopeIsToggelAnyKycDocuments` | switch | Any document (toggles between "has documents" and "has no documents") |
| `scopeIsToggelKycDocumentsVerified` | checkbox | Has verified documents |
| `scopeIsToggelKycDocumentsNotVerified` | checkbox | Has unverified documents |
| `scopeIsToggelKycDocumentsPending` | checkbox | Has pending documents |
| `scopeIsToggelKycDocumentsExpired` | checkbox | Has expired documents |
| `scopeIsToggelKycDocumentsByType` | any | Filter by document type |
| `scopeIsToggelKycDocumentsByCategory` | any | Filter by document category |
| `scopeIsToggelKycDocumentsVerifiedAndCategory` | checkbox | Combined condition: verified document **AND** belonging to a specific category (e.g., `primary_id`) |
| `scopeIsToggelKycDocumentsVerifiedAndType` | checkbox | Combined condition: verified document **AND** of a specific type (e.g., `passport`) |

All of these scopes support an intelligent toggle mechanism: when the filter is clicked, the value is automatically inverted (from 1 to 0) thanks to the `post('scopeName')` function.

**Example of filter settings in the file:**

```yaml
is_toggel_any_kyc_documents:
    label: 'Document status'
    type: switch
    scope: isToggelAnyKycDocuments
    default: 0

is_toggel_kyc_documents_verified_and_category_primary_id:
    label: 'Verified primary identity'
    type: checkbox
    scope: isToggelKycDocumentsVerifiedAndCategoryPrimaryId
    default: 0
```

##### 2. Group Filters with `dependsOn`

The version now supports multiple filters in the same list, linked via `dependsOn`. For example:

- **Document category filter** (`kyc_documents_category`) – the user selects a category (primary_id, address, ...).
- **Document type filter** (`kyc_documents_type`) – depends on the selected category, and shows only document types belonging to that category.

This is achieved through:

- The `getDocumentCategoryFilterOptions` and `getDocumentTypeFilterOptions` functions in the `Document` model.
- Full support for the `dependsOn` option in filter definitions inside the `config/extend.php` file.

Example from the settings file:

```php
[
    'scope_name' => 'kyc_documents_category',
    'label'      => 'Document Category',
    'type'       => 'group',
    'modelClass' => 'Nano3\Kyc\Models\Document',
    'options'    => 'getDocumentCategoryFilterOptions',
    'dependsOn'  => [],
],
[
    'scope_name' => 'kyc_documents_type',
    'label'      => 'Document Type',
    'type'       => 'group',
    'modelClass' => 'Nano3\Kyc\Models\Document',
    'options'    => 'getDocumentTypeFilterOptions',
    'dependsOn'  => ['kyc_documents_category'],
],
```

##### 3. Multiple Customizable List Columns

We added a set of ready‑to‑use default columns in `config/extend.php`, which can be used directly or customized:

- `documents_count` – total number of documents.
- `verified_documents_count` – number of verified documents.
- `not_verified_documents_count` – number of unverified documents.
- `expired_verified_documents_count` – number of expired (verified) documents.
- `is_verifier_primary_id`, `is_verifier_secondary_id`, ... – switch indicators for existence of documents from each category.

Each column supports all standard OctoberCMS options (label, type, sortable, width, align, ...). To ensure statistical values appear, new counting scopes have been added:

```php
public function scopeAddVerifiedDocumentsCount($query, $columnName = 'verified_documents_count')
public function scopeAddPendingDocumentsCount($query, $columnName = 'pending_documents_count')
public function scopeAddRejectedDocumentsCount($query, $columnName = 'rejected_documents_count')
public function scopeAddExpiredDocumentsCount($query, $columnName = 'expired_documents_count')
```

For these columns to work, the scopes must be applied to the list query, and `KycDocumentManager` does this automatically through its integrations.

##### 4. Centralized Default Settings via `config/extend.php`

Instead of writing column and filter settings repeatedly, all default settings have been moved to `plugins/nano3/kyc/config/extend.php`, which contains:

- `form_config` – default form field settings.
- `list_config` – array of default column definitions.
- `filter_config` – array of default filter definitions.
- `query_config` – default API query settings.

When any model is registered, these settings are automatically loaded from the file (if it exists), with the possibility of overriding them through code. This allows comprehensive customization per model without modifying the core files.

##### 5. Improved Transformer Integration

The `applyTransformerIntegration` function has been improved to support:

- `default_include` – adds `kyc_status` to the `defaultIncludes` list, making it appear automatically in every API response.
- `eager_load` – automatically eager‑loads the `documents` relationship with the main query for better performance.
- Support for `additional_fields` – adds custom fields to the Transformer that can be included on request.

##### 6. Automatic Registration of Frontend and Backend Users

The main Plugin's `registerKycDocumentIntegrations` function has been added to register both `RainLab\User\Models\User` and `Backend\Models\User` with the following integrations as default:

```php
'integrations' => [
    KycDocumentManager::INTEGRATION_BEHAVIOR    => true,  // behavior
    KycDocumentManager::INTEGRATION_LIST_COLUMN => true,  // list columns
    KycDocumentManager::INTEGRATION_LIST_FILTER => true,  // list filters
    KycDocumentManager::INTEGRATION_FORM_FIELD  => false, // can be enabled if needed
    KycDocumentManager::INTEGRATION_QUERY       => false,
    KycDocumentManager::INTEGRATION_TRANSFORMER  => false,
],
```

This means that user lists in the control panel will automatically display KYC document columns and filters as soon as the add‑on is activated.

---

### Practical Examples

#### 1. Using the New Toggle Scopes

```php
// Users who have verified documents
User::isToggelKycDocumentsVerified(1)->get();

// Users who have no expired documents (toggle by sending 0)
User::isToggelKycDocumentsExpired(0)->get();

// Users who have a verified primary identity (combined condition)
User::isToggelKycDocumentsVerifiedAndCategory(1, 'primary_id')->get();
```

#### 2. Setting Up `config_filter.yaml` with the New Filters

```yaml
scopes:
    kyc_documents_category:
        label: 'Document Category'
        type: group
        modelClass: Nano3\Kyc\Models\Document
        options: getDocumentCategoryFilterOptions
        scope: kycDocumentsCategory

    kyc_documents_type:
        label: 'Document Type'
        type: group
        modelClass: Nano3\Kyc\Models\Document
        options: getDocumentTypeFilterOptions
        scope: kycDocumentsType
        dependsOn: kyc_documents_category

    is_toggel_any_kyc_documents:
        label: 'Any documents'
        type: switch
        scope: isToggelAnyKycDocuments
        default: 0

    is_toggel_kyc_documents_verified:
        label: 'Verified'
        type: checkbox
        scope: isToggelKycDocumentsVerified
        default: 0
```

#### 3. Customizing Default Columns for a Specific Project

If you want to add a new column or change the order of columns, simply publish the configuration file and edit it:

```bash
php artisan config:publish nano3.kyc
```

Then edit `config/extend.php` in your project.

---

### Version Summary (1.2.9 – 1.3.0)

| Version | Key Features and Fixes |
| :--- | :--- |
| 1.2.9 | Integrated KYC integrations system: `KycDocumentModel` behavior, `KycDocumentManager` class, `KycIntegrationHandler`, intelligent API parameter handling, comprehensive documentation. |
| 1.3.0 | **Integration maturity**: Advanced Toggle scopes, group filters with dependencies, multiple list columns, centralized settings, automatic user model registration, Transformer improvements. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace the following files with the new versions:
     - `behaviors/KycDocumentModel.php`
     - `behaviors/KycDocumentModel/KycDocumentScopesAndHelpers.php`
     - `classes/KycDocumentManager.php`
     - `eventhandlers/KycIntegrationHandler.php`
     - `Plugin.php` (to add the `registerKycDocumentIntegrations` function)
   - Add the `config/extend.php` file if it does not already exist.
   - Update the translation files `lang/ar/lang.php` and `lang/en/lang.php` with the new keys in the `extend` section.

2. **Register additional models**:
   - If you use custom models, you can add them to the `registerKycDocumentIntegrations` function in your own Plugin.

3. **Test filters and columns**:
   - Verify that the new columns appear in user lists.
   - Try the category, type, and switch filters to ensure they work.
   - Ensure that interdependent filters (category → type) function correctly.

4. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### Conclusion

Version 1.3.0 is the culmination of a series of developments that have made the `Nano3.Kyc` add‑on an integrated, flexible tool for managing identity documents in NanoSoft applications. Thanks to advanced scopes, control panel integrations, and centralized settings, developers can now build complex KYC systems quickly and easily, with a sophisticated user experience in the control panel.

We appreciate your continued support and welcome your feedback to further improve the add‑on.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`KycDocumentManager` Class Documentation](./docs/Kyc/Docs-KycDocumentManager-Class-en.md)
- [`KycDocumentModel` Behavior Documentation](./docs/Kyc/Docs-KycDocumentModel-Behaviors-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./docs/Kyc/Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)

## 2026-03-20 – 2026-05-05

**`Nano.AccountsApi` Add‑on Update – Version 1.1.0**

### Comprehensive Restructuring and Addition of Integrated Accounting Document Types

---

### Summary of Updates

In version **1.1.0**, a major architectural leap was made in the `Nano.AccountsApi` add‑on. An abstract base class `BaseDocumentController` was extracted to manage common operations for different types of accounting documents. Full support was added for five new types of financial documents (daily entries, payment vouchers, debit notes, credit notes, transfers) with their own models, transformers and settings, together with an improved version of receipt vouchers (`CatchReceiptsV2`). The configuration file and routes were extended to support these new types, with multiple improvements to existing controllers and transformers.

---

### Version Goals

- **Unify operation logic** through `BaseDocumentController` to eliminate code duplication and make it easy to add new document types.
- **Add common accounting document types** to cover most daily financial operations via the API.
- **Provide flexible settings and separate permissions** for each document type, supported by environment variables to simplify customisation.
- **Improve developer experience** by providing a unified API interface for CRUD operations and permission checks.
- **Maintain backward compatibility** with older controllers and transformers, which continue to work unchanged.

---

### New Features and Improvements

#### 1. Unified Abstract Base Class `BaseDocumentController`

An abstract base class named `BaseDocumentController` was created, inheriting from `ApiController`, providing the following shared functionality:

- **Management of the current user** and permission checks (`checkAddPermission`, `checkListPermission`, `checkViewPermission`, `checkUpdatePermission`, `checkDeletePermission`).
- **Data preparation** through functions such as `prepareCreateData`, `prepareListOptions`, `validateInput`.
- **Operation execution** via `performCreate`, `performList`, `performShow`, `performUpdate`, `performDelete`, which return a unified result array.
- **Automatic transformer application** to API responses.

Any new financial document controller can now inherit this class and define only `$modelClass`, `$transformerClass`, `$configKey`, and `$typeHeader` to obtain full CRUD functionality.

#### 2. Addition of New Document Types

Five new controllers were added, each based on `BaseDocumentController`, with its own model, transformer and dedicated settings:

| Controller | Model | Transformer | typeHeader | Description |
|------------|-------|-------------|------------|-------------|
| `BondsDays` | `BondsDay` | `BondsDayTransformer` | `BondsDay::class` | Daily entries (multi‑party) with debit/credit balance checking |
| `PayReceipts` | `PayReceipt` | `PayReceiptTransformer` | `PayReceipt::class` | Payment vouchers (cheque, cash) |
| `DebitNotes` | `DebitNote` | `DebitNoteTransformer` | `DebitNote::class` | Debit notes |
| `CreditNotes` | `CreditNote` | `CreditNoteTransformer` | `CreditNote::class` | Credit notes |
| `Transfers` | `Transfer` | `TransferTransformer` | `Transfer::class` | Transfers between accounts |

An improved version of receipt vouchers was also added:

- `CatchReceiptsV2` (based on `BaseDocumentController` and the `CatchReceipt` model), while the old controller `CatchReceipts` remains unchanged.

**Validation rules specific to each type:**

- **BondsDays:** Checks that `details` exists with at least two parties, that debit equals credit, and prevents zero amounts.
- **CatchReceiptsV2 / PayReceipts:** Supports fields such as `payment_method`, `cheque_number`, `cheque_date`, `bank_name`, `beneficiary`.
- **DebitNotes / CreditNotes:** Fields `invoice_number` and `reason`.
- **Transfers:** Verifies that the two accounts are different, with fields `reference_number` and `reason`.

#### 3. New Transformer System

- A `BondsDayTransformer` was created, inheriting from `TransactionHeaderTransformer`, adding the total debit/credit for each entry.
- Transformers `CatchReceiptTransformer`, `PayReceiptTransformer`, `DebitNoteTransformer`, `CreditNoteTransformer`, `TransferTransformer` were created, all inheriting from `TransactionHeaderTransformer` and adding their own fields (e.g. `payment_method`, `cheque_number` …).
- `TransactionHeaderTransformer` was updated to accept all the new model types through union types in the signature of the `data()` function, allowing it to serve as a base transformer for any financial transaction header.

#### 4. Expanded Settings and Permissions

New sections were added in the `config.php` file for each document type, with support for environment variables to control:

- **Operational settings:** `order_by`, `order_dir`, `transactions_type`, `process_type`, `patterns_id`, `notes`.
- **Separate permissions:** `is_allow_add`, `is_allow_add_backend`, `is_check_add_permission`, `is_allow_add_frontend` (and similarly for list, view, update, delete).
- **Transformer settings:** `is_stop_to_array` for each type.

The added sections are: `bonds_days`, `catch_receipts_v2`, `pay_receipts`, `debit_notes`, `credit_notes`, `transfers`.

#### 5. Expanded API Routes

New routes were added within the `oauth-users` group in `routes.php`:

```
// Daily entries
POST  bonds/add → BondsDays@create
POST  bonds/test → BondsDays@test
GET   bonds → BondsDays@index
GET   bonds/{id} → BondsDays@show

// Payment vouchers
POST  payreceipts/add → PayReceipts@create
GET   payreceipts → PayReceipts@index
GET   payreceipts/{id} → PayReceipts@show

// Debit notes
POST  debitnotes/add → DebitNotes@create
GET   debitnotes → DebitNotes@index
GET   debitnotes/{id} → DebitNotes@show

// Credit notes
POST  creditnotes/add → CreditNotes@create
GET   creditnotes → CreditNotes@index
GET   creditnotes/{id} → CreditNotes@show

// Transfers
POST  transfers/add → Transfers@create
GET   transfers → Transfers@index
GET   transfers/{id} → Transfers@show
```

Note: Routes for update and delete (`PUT` and `DELETE`) were added as comments, ready to be activated in the future.

#### 6. Additional Improvements

- **The `show` function in `Accounts.php`** now supports looking up an account by its `code` in addition to its `id`, giving developers more flexibility.
- **The `test` function in `BondsDays`** allows testing a daily entry via `TransactionsHelper::testJournalEntry` before saving, with special permissions for backend users.
- **`TransactionHeaderTransformer`** was extended to cover all financial document types, while keeping its core functionality unchanged.

---

### Usage Examples

#### Create a multi‑party daily entry

```bash
curl -X POST "https://yourdomain.com/api/v1/accounts/bonds/add" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "date_at": "2026-04-20",
    "notes": "Settlement entry",
    "details": [
      {"accounts_id": "2-2-1253010001", "debit": 1500},
      {"accounts_id": "2-2-1231010001", "credit": 1000},
      {"accounts_id": "2-2-4111010001", "credit": 500}
    ]
  }'
```

#### List receipt vouchers (new version)

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/catchreceipts" \
  -H "Authorization: Bearer <token>"
```

#### Show an account using its code

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/accounts/2-2-1231010001" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Faster development**: Any new financial document type can be added by writing just a few lines of code, simply by inheriting `BaseDocumentController`.
- **Reduced errors**: Unifying permission‑checking logic and data preparation reduces the chance of human error.
- **Customisation flexibility**: Each document type has its own settings and separate permissions, allowing the add‑on to be adapted to the requirements of each project.
- **Enhanced user experience**: Full API support for all core financial operations allows building rich front‑end applications.
- **Stronger security**: Uses the same permission system already present in the base add‑on (`Tss.Accounts`) via the API.
- **Future‑proof**: The new architecture makes it easy to add update and delete operations (technically ready, waiting to be activated).

---

### Upgrade Requirements

1. **Update the code**:
   - Replace the core files with the new versions:
     - `APIControllers/BaseDocumentController.php`
     - `APIControllers/BondsDays.php`
     - `APIControllers/PayReceipts.php`
     - `APIControllers/DebitNotes.php`
     - `APIControllers/CreditNotes.php`
     - `APIControllers/Transfers.php`
     - `APIControllers/CatchReceiptsV2.php`
     - `Transformers/BondsDayTransformer.php`
     - `Transformers/CatchReceiptTransformer.php`
     - `Transformers/PayReceiptTransformer.php`
     - `Transformers/DebitNoteTransformer.php`
     - `Transformers/CreditNoteTransformer.php`
     - `Transformers/TransferTransformer.php`
   - Update the following files:
     - `config.php`
     - `routes.php`
     - `TransactionHeaderTransformer.php`
     - `Accounts.php` (the `show` improvement)
2. **No new database migrations**.
3. **API permissions**:
   - The new endpoints are protected by `oauth-users`, so the user must have a valid token, and their permissions must match the settings defined for each type.
4. **Test connectivity**:
   - Make sure the environment variables for the new settings are set correctly (especially `is_allow_add_backend` and the transaction type `transactions_type`).
   - Try creating a simple daily entry using `bonds/add` and check the response.
5. **Clear the cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### Conclusion

Version **1.1.0** represents a qualitative leap for the `Nano.AccountsApi` add‑on, moving it from a collection of individual controllers to a small framework for managing financial documents via the API. Thanks to `BaseDocumentController`, adding new document types has become quick and safe, opening the door to future expansions without complexity. The comprehensive coverage of entry types, vouchers, notes and transfers makes the add‑on suitable for a wide range of accounting applications, while maintaining backward compatibility with previous versions.

---

### Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./docs/Accounts/Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./docs/Accounts/Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./docs/Accounts/Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./docs/Accounts/Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./docs/Accounts/Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./docs/Accounts/Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./docs/Accounts/Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)

## 2026-05-06 - 2026-05-07

**Addition of `Nano.StudyyearApi` – Version 1.0.0**

### Creating a RESTful API for Managing Academic Years, Semesters, and Months

---

### Update Summary

The first version **1.0.0** of the `Nano.StudyyearApi` plugin provides a complete and unified API for interacting with data managed by the `Tss.Studyyear` plugin. This plugin is built according to the project's standard `Nano.*Api` pattern, offering clear and protected endpoints for retrieving academic years (`Periods`), semesters (`Semsters`), and academic months (`Months`). This version also includes a comprehensive configuration and multi-level permission system, professional data transformers, and full translation support.

---

### Release Objectives

- **Provide a unified API** for all academic year entities (years, semesters, months).
- **Apply a consistent architectural pattern** with other `Nano.*Api` plugins (such as `Nano.StudentsApi` and `Nano.AbsenceApi`).
- **Enable external applications and frontends** to easily and securely access this vital data.
- **Provide high flexibility in settings and permissions** for each resource individually.
- **Support advanced filtering and search** across all endpoints.
- **Facilitate future expansion** to add write operations (create, update, delete) without fundamental structural changes.

---

### New Features and Improvements

#### 1. Complete RESTful Controllers

Three main controllers were created following the unified `ApiController` pattern:

| Controller | Targeted Model | Description |
|------------|----------------|-------------|
| `Periods` | `Tss\Studyyear\Models\Period` | Managing academic years |
| `Semsters` | `Tss\Studyyear\Models\Semster` | Managing semesters (first and second) |
| `Months` | `Tss\Studyyear\Models\Month` | Managing academic months |

**Each controller provides:**

- **`index()`**: Fetch a list with full support for filtering, search, sorting, and pagination.
- **`show($id)`**: View details of a single record with all related relations.
- **`getRecords(array $options)`**: A customizable core function used internally for building complex queries.
- **`activelystats()`**: Endpoint for monitoring the last data update (for caching purposes).
- **`getLastUpdateAt()`**: Return the timestamp of the last update.
- **`validationList($user)`**: Unified function for checking access permissions based on user type (backend / frontend) and settings.

#### 2. Tight and Customizable Permission System

A multi-level permission system was applied, allowing administrators to precisely control access to each endpoint:

- **At the general user level**: Is the `list` operation allowed at all?
- **At the user type level**: Is it allowed for backend users? For frontend users?
- **At the specific permission level**: Does the user need to hold a specific permission (e.g., `tss.studyyear.periods.access`)?

These settings are controlled via `config.php` using environment variables (`env`), allowing behavior changes without touching the code.

**Example of `periods` settings in `config.php`:**
```php
'periods' => [
    'is_allow_list'            => env('NANO_STUDYYEARAPI_PERIODS_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_STUDYYEARAPI_PERIODS_IS_ALLOW_LIST_BACKEND', true),
    'is_allow_list_frontend'   => env('NANO_STUDYYEARAPI_PERIODS_IS_ALLOW_LIST_FRONTEND', true),
    'is_check_access_list'     => env('NANO_STUDYYEARAPI_PERIODS_IS_CHECK_ACCESS_LIST', true),
    'is_check_list_permission' => env('NANO_STUDYYEARAPI_PERIODS_IS_CHECK_LIST_PERMISSION', true),
    // ...
],
```

#### 3. Integrated Data Transformers

Three data transformers were created to format API responses professionally:

- **`PeriodTransformer`**: Displays academic year fields with included relations (`company`, `department`).
- **`SemsterTransformer`**: Displays semester fields with included relations (`company`, `department`, `period`).
- **`MonthTransformer`**: Displays academic month fields with included relations (`company`, `department`, `period`, `semster`).

All transformers support:
- **Data formatting**: Converting values to correct types (int, string, bool, float, date).
- **`formatDate` function**: For safe date formatting.
- **Field filtering (`exclude`)**: Users can specify field names they do not wish to receive to reduce transmitted data size.
- **`object_type`**: Adding object type to the response.
- **Error handling**: All `include` functions are wrapped in `try-catch` blocks to avoid failing the entire query due to a missing relation.

#### 4. Smart Support for Default Values

When the academic year ID (`study_year_id`) is not passed when fetching semesters (`Semsters`) or months (`Months`), the controllers automatically adopt the **default academic year** (`Period::getPrimary()`) as a filter, ensuring a smooth developer experience without explicitly specifying the year in every request.

**Practical example in `Semsters::getRecords()`:**
```php
if (!$study_year_id) {
    $primary = Period::getPrimary();
    if ($primary) {
        $study_year_id = $primary->id;
    }
}
```

#### 5. Caching Support

All `index` endpoints support caching via `nano.api::api_enable_cache` settings, significantly improving performance on repeated requests. Cache is automatically invalidated when any record is updated via `getLastUpdateAt()`.

#### 6. Advanced Filtering and Search

Numerous filters can be passed to `getRecords` such as:
- `companys_id` and `departments_id` (with automatic `IsCompany` scope support).
- `isActive`, `calendar_type`, `ref_type`, `status`.
- `study_year_id`, `semsters_id`, `month_num` (depending on the resource).
- Text search `q` across name, short description, and code.

#### 7. Multilingual Error Messages

A complete translation file (`lang/en/lang.php`) is included containing all success, error, and permission messages, making Arabic or other language translations easy.

---

### Usage Examples

#### Fetch active academic years

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/periods?isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### Fetch semesters of the default academic year, including year and company

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/semsters?include=period,company" \
  -H "Authorization: Bearer <token>"
```

#### Fetch months of a specific semester ordered by month

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/months?semsters_id=5&orderBy=month_num&orderDirection=asc" \
  -H "Authorization: Bearer <token>"
```

#### View details of a specific academic year

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/periods/12" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Seamless integration with user interfaces**: Any frontend application (React, Vue, Angular) can easily fetch year, semester, and month data to build academic calendars or registration forms.
- **Reduced developer burden**: Instead of writing complex queries manually, the plugin provides a rich and direct query interface.
- **Production-ready**: The advanced permission and settings system allows direct deployment in production environments with full access control.
- **Simple future expansion**: Adding `POST`, `PUT`, and `DELETE` operations in the future will only require extending the existing controllers without restructuring.
- **High performance**: Using `scopeExclude` for controlling retrieved columns and caching ensures fast response times.

---

### Upgrade Requirements

1. **Install Plugin**: Copy the `nano/studyyearapi` folder to `plugins/nano/studyyearapi`.
2. **Register Plugin**: Ensure you run `php artisan plugin:refresh Nano.StudyyearApi`.
3. **Environment Variables**: Adjust `env` variables as needed (e.g., enabling or disabling access for frontend users).
4. **Permissions**: Ensure that roles and users in the backend system possess the appropriate permissions (e.g., `tss.studyyear.periods.access`) if the `is_check_list_permission` option is enabled.
5. **Clear Cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **No database migrations** are required for this version.

---

### Conclusion

Version **1.0.0** of `Nano.StudyyearApi` lays the foundation for dealing with academic calendar data through a modern and secure API. By following best practices and the unified architectural pattern of `Nano.*Api` plugins, this plugin provides a solid base upon which to build more features such as creating and modifying academic years, and integrating with attendance or timetable systems.

---

**Reference Documentation**:
- [`Nano.StudyyearApi` Documentation](./docs/StudyyearApi/Docs-StudyyearApi-en.md)
- [`Nano.StudentsApi` Documentation](./docs/StudentsApi/Docs-StudentsApi-en.md)

## 2026-05-02 – 2026-05-08

**Nano.Coupons Plugin Update – Version 2.1.1 – Comprehensive Improvements to the "Product Limit" Condition**

---

### Update Summary

Version **2.1.1** introduces fundamental enhancements to the "Product Limit" feature that was launched in version 2.1.0. The validation logic has been completely restructured through the addition of a new, independent class `ProductLimitValidator`, providing unprecedented flexibility in handling product constraints. The update includes support for advanced scenarios such as limiting the number of orders for specific order types (regardless of product) and detailed multilingual error messages that show when the user can reorder. Significant performance improvements have also been made through caching and preventing redundant database queries.

---

### 1. Release Objectives

- **Separate validation logic** from the cart condition into an independent `ProductLimitValidator` class to facilitate reuse and maintenance.
- **Support new scenarios**: Limiting the number of allowed orders for a specific order type during a period, without the need to restrict a specific product.
- **Improve user experience** through clear and detailed error messages that include the maximum limit, current usage, period, and the date of the next possible order.
- **Significant performance improvements** through caching of constraints (`Cache`) and preventing repetitive expensive database queries.
- **Expand the admin interface** to include product search (`recordfinder`) and support selecting the unit for a specified product.
- **Add new scopes** `scopeOnProductLimit` and `scopeNotOnProductLimit` to the coupon model.

---

### 2. Developed Components

#### 2.1 `ProductLimitValidator` Class (New)

Location: `plugins/nano/coupons/classes/ProductLimitValidator.php`

An independent class specialized in checking product limit constraints. It accepts a user and configuration options (`use_cache`, `cache_ttl`) and provides comprehensive public functions:

| Function | Description |
|----------|-------------|
| `checkOrFail(...)` | Comprehensive check that throws an exception with a detailed error message when limits are exceeded. |
| `check(...)` | Silent check returning `true/false`. |
| `getLimits(...)` | Fetch applicable constraints with multiple strategies (`auto`, `cache`, `fresh_db`, `direct_query`). |
| `getLimitsForProduct(...)` | Shortcut to fetch constraints for a specific product. |
| `checkOrderTypeLimitOrFail(...)` | Check the number of orders for a specific order type. |
| `getOrderTypeLimits(...)` | Fetch constraints for a specific order type. |
| `clearCache()` | Manually clear cache. |
| `setUser(User $user)` | Set the user. |

**Supported Fetch Strategies:**
- `auto`: Automatically uses cache if enabled.
- `cache`: Force loading constraints from cache.
- `fresh_db`: Ignore cache and load directly from the database.
- `direct_query`: Execute a direct query on the coupons table with specified conditions.

**Key Logic Improvements:**

- **First Case (Specific Product):**
  - If `max_quantity > 0`, the total quantity (cart + historical + newly added) is checked.
  - If `max_orders > 0`, the number of orders containing this product (or its unit) is checked.
  - Respects `units_id`, `order_restriction`, `shop_restriction`, and `period_days`.

- **Second Case (No Product – Order Count Limit Only):**
  - If `product_id` is empty, the `max_orders` check is applied based on `order_types`, `shop_ids`, and `period_days`.

#### 2.2 Updates to the `ProductLimit` Cart Condition

Location: `plugins/nano/coupons/cartconditions/ProductLimit.php`

- The class has been simplified to act as a mediator between the cart system (`CartCondition`) and `ProductLimitValidator`.
- The `validateCartItem` function now uses `ProductLimitValidator` directly.
- The `beforeApply` has been temporarily disabled to prevent any additional load during cart total calculation.
- Reliance on cache via `Cache::remember` in `loadAllProductLimits` with `is_expired` stored to avoid serialization issues.

#### 2.3 Updates to the `Coupon` and `Coupons_model` Models

- Added cast properties `'product_id' => 'integer'`, `'units_id' => 'integer'`, `'max_quantity' => 'integer'`, `'max_orders' => 'integer'`, `'period_days' => 'integer'`.
- Added `getProductIdOptions` function to list active products.
- Expanded `getUnitsIdOptions` to dynamically fetch units of a specific product.
- Added scopes `scopeOnProductLimit` and `scopeNotOnProductLimit` to filter coupons.
- In `afterSave`, the cache `nano.coupons.product_limits` is cleared when a `product_limit` coupon is modified/added.

#### 2.4 Admin Interface Updates (`fields.yaml`)

- Changed `product_id` from `type: dropdown` to `type: recordfinder` for advanced product search.
- Added translation support for `placeholder`, `emptyOption`, `title`, `prompt` properties.
- Updated `units_id` to work with `dependsOn: [product_id]` to dynamically update the unit list.
- Show fields `product_id`, `units_id`, `max_quantity`, `max_orders`, `period_days` only when `apply_coupon_on = product_limit` is selected via `trigger`.

#### 2.5 Translation Files (`lang/ar/lang.php`)

- New keys for product limit messages with parameters:
  - `max_quantity_exceeded` with `:max` and `:count`.
  - `max_orders_exceeded` with `:max`, `:count`, `:days`, `:next_date`.
  - `unlimited` and new field label items.

#### 2.6 Configuration File (`config.php`)

- Added key `is_support_product_limit` with default value `false` (to control showing the type in lists).

#### 2.7 Updates to `Plugin.php` (Nano.Coupons)

- Registered the `ProductLimit` condition in `registerCartConditions` only when the class exists.
- Modified `bindCouponsEvent` to add `notOnProductLimit()` to avoid applying automatic coupons to a cart that has a product limit.

---

### 3. Workflow (New Flow)

1. **Setup (Once):** The admin creates a `product_limit` coupon, selecting either a specific product (and optional unit) or leaving it empty to restrict the number of orders by order type.
2. **Loading Constraints:** When the cart loads, constraints are loaded from cache (or database) once.
3. **Immediate Check on Product Addition:** `ProductLimit` calls `ProductLimitValidator::checkOrFail` using the `nano.cart.adding` event.
4. **Quantity / Order Count Check:**
   - For product: total quantity + new quantity ≤ `max_quantity`.
   - For product or order type: previous order count < `max_orders`.
5. **Detailed Error Message:** "You have exceeded the allowed number of orders (maximum 3). Current order count: 3. Period: 7 days. You can order again after 2026-05-10."

---

### 4. Key Enhancements and Achievements

- **Excellent Performance:** Using `Cache` for constraints with automatic clearing when the coupon is modified.
- **Bulk Queries:** `getBulkHistoricalQuantities` and `getBulkHistoricalOrderCounts` aggregate data in a single query.
- **High Flexibility:**
  - Check a specific product, a specific order type, or both.
  - Check quantity, number of orders, or both.
  - Optionally specify `unit_id`.
- **Professional Error Messages:** Display the maximum limit, current usage, period, and the date when reordering is allowed.
- **Full Translation Support** (Arabic/English).
- **Separation of Concerns:** `ProductLimitValidator` is reusable anywhere (API, CLI, etc.) without depending on the cart system.

---

### 5. Requirements and Upgrade

- **File Updates:**
  - `plugins/nano/coupons/classes/ProductLimitValidator.php` (new)
  - `plugins/nano/coupons/cartconditions/ProductLimit.php`
  - `plugins/nano/coupons/models/Coupon.php`
  - `plugins/nano/coupons/models/Coupons_model.php`
  - `plugins/nano/coupons/models/coupon/fields.yaml`
  - `plugins/nano/coupons/lang/ar/lang.php`
  - `plugins/nano/coupons/Plugin.php`
  - `plugins/nano/coupons/config/config.php`
- **Run Migration:** No new migrations in this version (columns were added in 2.1.0).
- **Clear Cache:** It is recommended to run `php artisan cache:clear` after upgrading to ensure loading of new translation files.

---

### 6. Additional Notes

- The `product_limit` feature can be enabled/disabled from the `.env` file via the `NANO_COUPONS_IS_SUPPORT_PRODUCT_LIMIT` key.
- For better performance with a large number of coupons, adjust `cache_ttl` to an appropriate duration (default 300 seconds).
- All queries are safe against SQL Injection and use parameter binding.

---

### 7. Future Development Plans

- Support specifying multiple products in a single `product_limit` coupon.
- Develop a reporting interface to track customer consumption of product limits.
- Add the ability to disable the constraint for specific customer groups.

---

### 8. Conclusion

Version 2.1.1 completes the "Product Limit" system and makes it more powerful, flexible, and professional. Thanks to the separate `ProductLimitValidator`, developers can easily integrate limit checks anywhere, while performance improvements ensure a smooth end-user experience even with a large number of active constraints.

**Reference Documentation**:

- [`ProductLimitValidator` Class Documentation](./docs/Coupons/Classes/Docs-ProductLimitValidator-Class-en.md)
- [`ProductLimit` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-ProductLimit-CartCondition-en.md)
- [`Coupon` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-Coupon-CartCondition-en.md)
- [`AutoCoupon` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-AutoCoupon-CartCondition-en.md)
- [`AutoCouponShipping` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-AutoCouponShipping-CartCondition-en.md)

## 2026-05-06 – 2026-05-08

**Updates for `Nano.Orders` Plugin – Version 2.2.10**  
**Updates for `Nano.OrdersApi` Plugin – Version 1.0.20**

### Summary of Updates

The `Nano.Orders` plugin received a pivotal update in version **2.2.10** focusing on enabling advanced filtering of orders by product owner, developing a professional mechanism for updating order status with graduated permissions and multiple roles, along with improvements in `OrderHelper` for managing users and couriers.  
Meanwhile, version **1.0.20** of `Nano.OrdersApi` provided a unified API endpoint for updating order status, leveraging the new functions in `OrderManager`, with full support for custom options and permissions.

---

## Nano.Orders v2.2.10 – Product Owner Filtering and Graduated Permission Order Status Update

### Release Objectives

- **Provide advanced query scopes** for filtering orders based on the ownership of associated products.
- **Create a comprehensive professional function** to update order status with different roles (admin, owner, courier) with flexible transition rules and customizable permissions.
- **Add helper functions** to extract courier from user and vice versa.
- **Integrate product owner filtering** inside `OrderHelper::getOrdersRecords` for smarter reports.
- **Support external configuration** (`config`) to control transition rules and permissions.

### New Features

#### 1. `HasProductOwnerScopes` Trait in the `Order` Model

The trait was created at path `Nano\Orders\Models\Order\HasProductOwnerScopes` and includes advanced scopes and functions for filtering orders by product owner:

| Scope / Function | Description |
|------------------|-------------|
| `scopeWhereHasProductsByOwner` | Main scope with comprehensive options array: `user`, `count`, `not`, `boolean`, `conditions`, `itemConditions`. |
| `scopeWhereProductOwner` | Quick shortcut (at least one product). |
| `scopeHasProductsByOwner` | Filter by a minimum number of owned products. |
| `scopeDoesntHaveProductsByOwner` | Filter orders containing no owned product. |
| `scopeHasProductsByOwnerCount` | Exact (or compared) count of owned products. |
| `scopeOrWhereHasProductsByOwner` | Link scope with `OR`. |
| `scopeWithProductsByOwnerCount` | Add a computed column counting owned products. |
| `scopeSortByProductsByOwnerCount` | Sort orders by number of owned products. |
| `hasAnyProductByOwner` | Check function on a single order object. |
| `applyOwnerToProductQuery` | Internal helper to apply owner condition to a product query. |

All scopes use table name constants (`Product::TABLE_NAME`) to ensure lasting compatibility.

**Usage Example:**
```php
// Orders containing 2 or more products owned by the current user
$orders = Order::whereHasProductsByOwner([
    'count' => 2,
    'countOperator' => '>='
])->get();
```

#### 2. `updateOrderStatusAdvanced` Function in `StepStatus` Trait

An integrated function was added meeting all requirements for updating order status with graduated permissions:

| Feature | Description |
|---------|-------------|
| **Automatic actor detection** | Uses the currently logged-in user (backend admin, owner, courier). |
| **Three status fields** | `order_states_ref_type` (for management), `user_status` (for owner), `delivery_status` (for courier). |
| **Customizable transition rules** | Default + from settings + role-specific + custom per call. |
| **Prevent modifying completed** | Changing a completed order's status is not allowed (except with admin override). |
| **Automatic sync** | When `user_status` equals `delivery_status`, the main status is updated. |
| **Cancellation reasons** | `because_cancel` (owner) and `delivery_because_cancel` (courier). |
| **Comprehensive control options** | `is_save`, `is_event`, `is_logs`, `skip_permission`, `admin_override`, `custom_message`. |
| **Unified response structure** | Contains `input_data`, `process_data`, `model`, `debug`. |

**Role Examples:**
- **Admin**: Can change any field, with the ability to override transition constraints (`admin_override`).
- **Owner**: Changes `user_status` only, and can cancel the order with a reason.
- **Courier**: Changes `delivery_status`, and can only move `user_status` from `NEW` to `DELIVERY`.

#### 3. Helper Functions in `OrderHelper`

- `getDeliveryByUser($user)` – Extracts the courier object from a user.
- `getUserByDelivery($delivery)` – Extracts the user object from a courier.

#### 4. Product Owner Filtering Support in `OrderHelper::getOrdersRecords`

The following options were added:
- `is_has_products_by_owner` (enable filter)
- `products_by_owner_*` (all options of the main scope)
- `is_has_products_by_owner_or_delivery` to combine the filter with delivery conditions using `OR`.

#### 5. New Settings in `config.php`

```php
'nano.orders::manager.edit_status.allowed_transitions'
'nano.orders::manager.edit_status.admin.allowed_transitions'
'nano.orders::manager.edit_status.user.allowed_transitions'
'nano.orders::manager.edit_status.delivery.allowed_transitions'
```

---

## Nano.OrdersApi v1.0.20 – API Endpoint for Updating Order Status

### Release Objectives

- **Provide a unified API interface** for updating order status by the owner, courier, or admin.
- **Full integration with `updateOrderStatusAdvanced`** to ensure consistent permissions and logic.
- **Support custom options** via request JSON.
- **Maintain a consistent response structure** with the rest of the API.

### New Features

#### 1. Endpoint `POST /api/v1/orders/orders/update-status`

Accepts a JSON request with the following fields (all optional and depend on the role):
- `order_id` or `id`
- `order_states_ref_type`
- `user_status`
- `delivery_status`
- `because_cancel`
- `delivery_because_cancel`
- `is_save`, `is_event`, `is_logs`
- `skip_permission`, `admin_override`
- `custom_message`, `custom_error`

#### 2. `updateStatus` Function in `Orders` Controller

Extracts the order, prepares options, calls `OrderManager::updateOrderStatusAdvanced()` with all parameters, and returns a unified response containing the new status.

**Example successful response:**
```json
{
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "Duplicate order"
    }
}
```

---

## Version Summary (2.2.10 and 1.0.20)

| Version | Key Features |
|---------|--------------|
| **Nano.Orders 2.2.10** | `HasProductOwnerScopes` (product owner filtering scopes), `updateOrderStatusAdvanced` (graduated permission status update), `getDeliveryByUser` / `getUserByDelivery` functions, product owner filtering support in `OrderHelper::getOrdersRecords`, flexible transition rules configuration. |
| **Nano.OrdersApi 1.0.20** | API endpoint `POST orders/update-status`, integration with the professional function, support for all custom options, unified response. |

---

### Upgrade Requirements

1. **Update Files**:
   - Add `HasProductOwnerScopes` trait in `models/Order/HasProductOwnerScopes.php`.
   - Add `StepStatus` trait in `traits/steps/StepStatus.php`.
   - Update `OrderHelper.php` with new functions and product owner filter support.
   - Update `OrderManager.php` to use `StepStatus`.
   - Update `Orders.php` in `OrdersApi` to add `updateStatus` function.
   - Update `routes.php` in `OrdersApi` to add the new route.

2. **No New Migrations**: The two versions require no database changes.

3. **Translation Files**:
   - Add `lang/ar/manager/update_status.php` file in `Nano.Orders` with the required keys.

4. **Optional Settings**:
   - Custom transition rules can be added in `config.php` under `nano.orders::manager.edit_status`.

5. **Compatibility Testing**:
   - Test the new scopes with different queries.
   - Test status updates from the three roles.
   - Test the API from mobile applications (using the appropriate user token).

---

### Conclusion

Versions **Nano.Orders 2.2.10** and **Nano.OrdersApi 1.0.20** represent a qualitative leap in the flexibility of the orders system. On one hand, the `HasProductOwnerScopes` scopes provided unprecedented filtering of orders based on product ownership, meeting the needs of multi-vendor marketplaces. On the other hand, the `updateOrderStatusAdvanced` function and the integrated API endpoint delivered a unified and secure system for updating order status by all parties (admin, owner, courier) with fully customizable settings. The code is clean, documented, and easily extensible.

---

**Reference Documentation**:
- [`OrderManager` and its traits Documentation](./docs/Orders/Classes/Docs-OrderManager-Class-en.md)
- [`StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-en.md)
- [Advanced `StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-en.md)
- [`HasProductOwnerScopes` Scopes Documentation](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-en.md)
- [Orders API Documentation](./docs/OrdersApi/Docs-OrdersApi-en.md)

## 2026-05-04 – 2026-05-10

**JaibPay Payment Gateway Updates (Jeib Wallet) – May 2026**

### Comprehensive Improvements and Fixes to JaibPay Payment Gateway

Within the `Nano.Yepayment` module

---

### 1. Introduction

After completing the initial release of the **JaibPay** (Jeib Wallet) gateway, a series of essential improvements and fixes were implemented to ensure the gateway works professionally and is fully integrated with the Nano2Soft system. The updates included fixing the authentication function, improving API response handling mechanisms, adding port testing and response processing functions, creating a refund function, and resolving the issue of saving encrypted settings in `PaymentGatewaySettings`.

The gateway was actually tested with the Jaib Pay platform across multiple different payment and refund scenarios, and all tests proved successful integration.

---

### 2. Developed and Updated Components

#### 2.1. `JaibPay` – Main Payment Gateway Class

**New and improved functions:**

| Function | Description | Type |
|--------|-------|-------|
| `normalizeApiResponse($response): array` | Process Jaib API response and convert it into a unified structure containing `success`, `status_code`, `error_code`, `error_message`, `data`, `raw`. Handles HTTP 200 success, HTTP 500 errors, and JSON errors. | **New** |
| `looksLikeJson($string): bool` | Helper function to check if text resembles JSON. | **New** |
| `getSettingsByKey($key, $defaultValue = null)` | Correctly retrieve a setting value from `PaymentGatewaySettings`, supporting encrypted fields via `getEncryptableValue()`. | **New** |
| `testPortConnection(string $url, ?int $port, int $timeout): array` | Test whether a specific port is open on a server. Automatically extracts port from URL, supports DNS analysis, and measures response time. | **New** |
| `refund(string $referenceID, ...): array` | Refund a transaction via `POST /api/v1/BuyOnline/RefoundBuy`. Changes order status to `RefundedState` and stores refund data in `other_data['jaibpay_refund']`. | **New** |
| `getAuthToken()` | **Updated** – Uses `normalizeApiResponse` to display clear error messages, and uses `getSettingsByKey` to correctly fetch the encrypted password. | **Updated** |
| `executeBuy(...)` | **Updated** – Uses `normalizeApiResponse` to display precise error messages like "Invalid purchase code" or "Code already used". | **Updated** |
| `checkTransactionStatus(...)` | **Updated** – Uses `normalizeApiResponse` to improve display of query results. | **Updated** |
| `process(PaymentResult $result)` | **Updated** – Sets `$result->message` from the exception before calling `fail()` to ensure the error message appears in the user interface. | **Updated** |

#### 2.2. `_test_info.htm` – Quick Testing Tools

The quick test interface was updated to include:

| Button | Description |
|------|-------|
| **Test Authentication** | Verify login credentials (username/password/agentCode) |
| **Test Port** | Check connectivity to port 5088 on the remote server (uses `testPortConnection`) |
| **Execute Payment** | Create a new payment transaction |
| **Check Status** | Query transaction status |
| **Comprehensive Test** | Execute payment + query |
| **Refund** | Send a refund request using `referenceID` |

#### 2.3. `routes.php` – API Endpoints

New routes added:

| Path | Method | Description |
|--------|---------|-------|
| `/jaibpay/test-port` | GET | Port test (uses `JaibPay::testPortConnection`) |
| `/jaibpay/test-refund` | POST | Refund a transaction amount |

#### 2.4. `PaymentGatewaySettings` – Fix for Saving Encrypted Settings

The `Nano\MicroCart\Models\PaymentGatewaySettings` class was modified to fix the problem of encrypted values being replaced with empty values when saving settings:

- **Issue:** When opening the gateway settings page and pressing "Save" without re-entering passwords (password-type fields), the encrypted values were replaced with empty values.
- **Solution:** The magic `__set` function was overridden to prevent assigning empty values to encrypted fields if the current value is not empty, ensuring sensitive settings remain unchanged.

---

### 3. Payment Working Cycle After Update

The payment pattern remained **Direct (instant)** as before, but with fundamental improvements in error handling and user experience:

1. **Authentication** – Uses the improved `getAuthToken()` which:
   - Correctly retrieves the encrypted password via `getSettingsByKey()`.
   - Uses `normalizeApiResponse` to provide clear error messages (e.g., "Incorrect username or password").

2. **Payment Execution** – Uses the improved `executeBuy()` which:
   - Uses `normalizeApiResponse` to extract error messages directly from Jaib's response.
   - Displays precise errors such as:
     - `"Invalid purchase code"` (code: 51)
     - `"The code has already been used"` (code: -1026)
     - `"Invalid code number"` (code: 51)

3. **Error Handling** – In `process()`, `$result->message` is set before calling `fail()` to ensure the error message appears in the user interface.

4. **Refund** – The new `refund()` function which:
   - Sends a refund request to `/api/v1/BuyOnline/RefoundBuy`.
   - Changes the order status to `RefundedState`.
   - Stores refund data in `order->other_data['jaibpay_refund']`.

---

### 4. Detailed New Functions

#### 4.1. `normalizeApiResponse` – Unified API Response Processing

```php
private function normalizeApiResponse($response): array
```

**Functionality:** Receives an HTTP response (or exception) and uniformly extracts:
- `success` – operation success
- `status_code` – HTTP code
- `error_code` – error code from Jaib (e.g., `51`, `-100`, `-1026`)
- `error_message` – error message from Jaib (e.g., "Invalid purchase code")
- `data` – result data (`result` from Jaib response)
- `raw` – full raw response

**Usage:** Used in all API functions (`getAuthToken`, `executeBuy`, `checkTransactionStatus`, `refund`) to unify response handling.

#### 4.2. `testPortConnection` – Port Testing

```php
public static function testPortConnection(string $url, ?int $port = null, int $timeout = 5): array
```

**Functionality:** Tests whether a specific port is open on a server. Supports:
- Automatic port extraction from URL (e.g., `https://www.api2.e-jaib.com:5088`).
- Prioritizing the passed parameter port over the URL port.
- Using the default port according to the scheme (443 for HTTPS, 80 for HTTP).
- DNS check and determining if the issue is DNS or the port.

**Usage:** Used in the `jaibpay/test-port` route and in `_test_info.htm` to diagnose connection issues.

#### 4.3. `refund` – Refund

```php
public function refund(string $referenceID, ?string $requestID = null, ?float $amount = null, ?string $currencyCode = null, ?string $notes = null): array
```

**Functionality:** Performs a full refund operation:
- Uses `normalizeApiResponse` to handle the response.
- On success: changes `order->payment_state` to `RefundedState` and stores refund data in `order->other_data['jaibpay_refund']`.
- On failure: displays the error message from Jaib.

---

### 5. Bug Fixes

| Issue | Description | Solution |
|---------|-------|------|
| **Saving encrypted settings with empty values** | When saving the settings page without re-entering the password, encrypted values were replaced with empty values. | Override `__set` in `PaymentGatewaySettings` to prevent assigning empty values to encrypted fields. |
| **Unclear error messages** | When payment failed, it showed `Server error: POST ... 500 Internal Server Error` instead of the actual error message. | Use `normalizeApiResponse` to extract `error.message` from Jaib's response. |
| **`$result->message` empty on failure** | In `process()`, the error message was not passed to `PaymentResult`. | Add `$result->message = $e->getMessage();` before `$result->fail()`. |
| **Fetching encrypted password** | `PaymentGatewaySettings::get('jaibpay_password')` returned empty value for encrypted fields. | Use `getSettingsByKey()` with `getEncryptableValue()`. |
| **`null` error message not appearing** | On `process()` failure, `$result->message` remained `null`. | Improve `catch` so message is set before `fail`. |

---

### 6. Actual Integration Tests

Actual integration tests were conducted with the Jaib Pay platform across all possible scenarios:

#### 6.1. Authentication Test
| Scenario | Result |
|-----------|---------|
| Correct credentials | ✅ Success – `accessToken` and `pinApi` returned |
| Wrong credentials | ✅ Failure – "Invalid username or password" message |

#### 6.2. Port Test
| Scenario | Result |
|-----------|---------|
| Check port 5088 on `www.api2.e-jaib.com` | ✅ Success – Connected successfully |

#### 6.3. Payment Execution Test
| Scenario | Inputs | Result |
|-----------|----------|---------|
| **Correct code + amount + SAR currency** | `code`: valid, `amount`: 5000, `currency`: SAR | ✅ Success – Payment executed |
| **Correct code + wrong amount** | `code`: valid, `amount`: 6000 (exceeds balance) | ❌ Failure – Error message from Jaib |
| **Wrong code + correct amount** | `code`: invalid, `amount`: 5000 | ❌ Failure – "Invalid purchase code" |
| **Correct code + correct amount + SAR currency** | `code`: valid, `amount`: 5000, `currency`: SAR | ✅ Success – Payment executed |
| **Code already used** | `code`: used | ❌ Failure – "The code has already been used" |
| **Correct code + correct amount + YER currency** | `code`: valid, `amount`: 5000, `currency`: YER | ✅ Success – Payment executed |
| **Correct code + correct amount + expired** | `code`: expired | ❌ Failure – Error message from Jaib |

#### 6.4. Refund Test
| Scenario | Result |
|-----------|---------|
| Refund a successful transaction amount | ✅ Success – Refund executed |
| Refund a non-existent transaction | ❌ Failure – "Invalid transaction reference fee" |
| Refund without `reference_id` | ✅ Automatically fetched from order data |
| Attempt to refund an unpaid order | ❌ Failure – "Cannot refund an unpaid order" |

---

### 7. Notes for Developers

- **Using `normalizeApiResponse` has become mandatory** in all API functions to ensure accurate error messages are displayed.
- **The `getSettingsByKey` function** must be used to fetch any encrypted setting instead of direct `PaymentGatewaySettings::get()`.
- **Port testing** is available via `JaibPay::testPortConnection()` and can be used to diagnose any connection issue.
- **Refund** changes the order status to `RefundedState` automatically and stores refund data under `other_data['jaibpay_refund']`.
- **Improvement to `PaymentGatewaySettings`** prevents loss of encrypted settings, but ensure to occasionally clear cache (`Cache::forget('system::settings...')`) after modifications.
- **The `refund` function** requires the `referenceID` of the original transaction (obtainable from `order->payment_trans_id` or `other_data['jaibpay']['reference_id']`).

---

### 8. Related Links

- [JaibPay Documentation (Developer Guide)](./docs/JaibPay/Docs-JaibPay-en.md)
- [Jaib Pay API Documentation – Login (Login.pdf)](./Login.pdf)
- [Jaib Pay API Documentation – Execute and Query Payment (Jaib Wallet Pay API.pdf)](./Jaib%20Wallet%20Pay%20API.pdf)
- [Postman Collection for Jaib Pay API Testing](./Jaib%20Pay%20API.postman_collection.json)
- [Nano2Soft Payment Gateway Development Guide](./SKILL.md)
- [Technical Support Channel](https://nano2soft.com)

---

**This update has been prepared by:**  
Nano2Soft Development Team – Electronic Payments Division  
**Reviewer:** Dheia Ali, Nano2Soft

## 2026-05-08 - 2026-05-10

**Addition of `Nano.SchoolApi` – Version 1.0.0**

### Creating a Comprehensive RESTful API for Core School Data Management

---

### Update Summary

The first version **1.0.0** of the `Nano.SchoolApi` plugin introduces a complete API for querying all core entities managed by the `Tss.School` plugin. The plugin is built according to the project's standard `Nano.*Api` pattern, providing protected endpoints to retrieve data for classes (`Classes`), stages (`Stages`), subjects (`Subjects`), groups (`Groups`), group-to-class distribution (`ClassGroups`), subject-to-class distribution (`ClassSubjects`), teacher-to-subject distribution (`TeacherSubjects`), cost center linking to classes (`ClassCostCenters`), fee types (`FeesTypes`), document types (`DocumentTypes`), external schools (`OutSchools`), student documents (`StudentDocuments`), student fees (`StudentsFees`) and last year marks (`LastStudentMarks`). The version supports a multi-level permission system for each resource, professional data transformers, and full translation support.

---

### Release Objectives

- **Provide a unified API** for all vital `Tss.School` entities, opening data for multiple uses.
- **Implement a consistent architectural pattern** with other `Nano.*Api` plugins for ease of maintenance and expansion.
- **Enable external applications and frontends** to access school data (classes, subjects, schedules, fees, documents) easily and securely.
- **Provide fine-grained access control** for each resource, allowing the plugin to be deployed in production environments with varying permissions.
- **Support advanced filtering, search, and sorting** to meet reporting and integration needs.
- **Facilitate building reports and statistics** based on this data without direct database access.

---

### New Features and Improvements

#### 1. Comprehensive RESTful Controllers (14 Controllers)

14 controllers were created covering all core entities of the `Tss.School` plugin, all supporting read operations (`list` and `show`):

| Controller | Targeted Model | Description |
|------------|----------------|-------------|
| `Classes` | `TbClass` | Academic classes (with fees, ranking, stage) |
| `Stages` | `Stage` | Academic stages |
| `Subjects` | `Subject` | Academic subjects |
| `Groups` | `Group` | Academic groups |
| `ClassGroups` | `ClassGroup` | Group-class linking with academic year |
| `ClassSubjects` | `ClassSubject` | Subject distribution to classes (units, grades) |
| `TeacherSubjects` | `TeacherSubject` | Teacher distribution to subjects and groups |
| `ClassCostCenters` | `ClassCostCenter` | Cost center linking to classes |
| `FeesTypes` | `FeesType` | Fee types |
| `DocumentTypes` | `DocumentType` | Required document types |
| `OutSchools` | `OutSchool` | External schools |
| `StudentDocuments` | `StudentDocument` | Student documents (submission status) |
| `StudentsFees` | `StudentsFee` | Student fees (amount, discount) |
| `LastStudentMarks` | `LastStudentMark` | Last academic year marks for students |

**Each controller provides:**

- **`index()`**: Fetch a paginated list with multiple filtering, search, sorting, and pagination.
- **`show($id)`**: View details of a single record with relations.
- **`getRecords(array $options)`**: Core function for building complex queries with support for default settings from `config.php`.
- **`activelystats()`**: Endpoint for monitoring last update (for caching purposes).
- **`getLastUpdateAt()`**: Returns the timestamp of the last update.
- **`validationList($user)`**: Unified function for checking access permissions.

#### 2. Tight Permission System for Each Resource

A separate permission system was applied for each of the fourteen resources, with full control via `config.php` settings and environment variables (`env`). Each resource has:

- `is_allow_list`: Enable/disable listing operation.
- `is_allow_list_backend` / `is_allow_list_frontend`: Allow backend or frontend users.
- `is_check_access_list`: Whether to verify a valid user.
- `is_check_list_permission`: Whether to check for a specific permission (e.g., `tss.school.tbclasses.access`).

In addition to filter settings: `order_by`, `order_dir`, `per_page`, `exclude`.

**Example from `config.php`:**
```php
'classes' => [
    'is_allow_list'            => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_BACKEND', true),
    'is_allow_list_frontend'   => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND', true),
    'is_check_access_list'     => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_ACCESS_LIST', true),
    'is_check_list_permission' => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_LIST_PERMISSION', true),
    'order_by'                 => env('NANO_SCHOOLAPI_CLASSES_ORDER_BY', 'sort_order'),
    'order_dir'                => env('NANO_SCHOOLAPI_CLASSES_ORDER_DIR', 'asc'),
    'per_page'                 => env('NANO_SCHOOLAPI_CLASSES_PER_PAGE', 15),
    'exclude'                  => env('NANO_SCHOOLAPI_CLASSES_EXCLUDE', ''),
],
```

#### 3. Smart Default Values for Resources Dependent on Academic Year

Many resources (such as `ClassGroups`, `TeacherSubjects`, `ClassCostCenters`, `StudentDocuments`, `StudentsFees`) are linked to an academic year (`year_id`). When not provided, the controllers automatically adopt the **default academic year** (`Period::getPrimary()`), ensuring a smooth experience without errors.

**Example from `ClassGroups::getRecords()`:**
```php
if (!$year_id) {
    $primary = Period::getPrimary();
    if ($primary) {
        $year_id = $primary->id;
    }
}
```

#### 4. Comprehensive Data Transformers (14 Transformers)

A data transformer was created for each model, formatting the response and providing included relations with error handling:

- **`ClassTransformer`**: Classes with `company`, `department`, `stage`.
- **`StageTransformer`**: Stages with `company`, `department`.
- **`SubjectTransformer`**: Subjects with `company`, `department`.
- **`GroupTransformer`**: Groups with `company`, `department`.
- **`ClassGroupTransformer`**: Group links with `company`, `department`, `class`, `group`, `study_year`.
- **`ClassSubjectTransformer`**: Subject distribution with `company`, `department`, `class`, `subject`.
- **`TeacherSubjectTransformer`**: Teacher distribution with `company`, `department`, `class`, `group`, `subject`, `teacher`, `study_year`.
- **`ClassCostCenterTransformer`**: Cost center links with `company`, `department`, `class`, `cost_center`, `study_year`.
- **`FeesTypeTransformer`**: Fee types with `company`, `department`.
- **`DocumentTypeTransformer`**: Document types with `company`, `department`.
- **`OutSchoolTransformer`**: External schools with `company`, `department`.
- **`StudentDocumentTransformer`**: Student documents with `company`, `department`, `student`, `documents`, `study_year`.
- **`StudentsFeeTransformer`**: Student fees with `company`, `department`, `student`, `student_record`, `fees_type`, `study_year`.
- **`LastStudentMarkTransformer`**: Last year marks with `company`, `department`, `student`, `class`, `out_school`, `student_record`, `study_year`.

**Transformer Features:**
- Formatting values to correct types (int, string, bool, float, date).
- `formatDate` function for safe date conversion.
- Field filtering (`exclude`) via querystring to control returned data size.
- Adding `object_type` to identify the object type.
- Each relation is wrapped in `try-catch` to avoid failing the entire query.

#### 5. `scopeExclude` Registration for All Models

The dynamic scope `scopeExclude` was registered for all fourteen models inside `Plugin::boot()`, allowing clients to request only specific columns and reduce transmitted data over the network.

```php
foreach ($models as $model) {
    $model::extend(function($model) {
        $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
            $getTable = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
            return $query->select(array_diff($getTable, (array) $columns));
        });
    });
}
```

#### 6. Caching Support

All `index` endpoints support caching via `nano.api::api_enable_cache`, with automatic cache invalidation upon any record update.

#### 7. Comprehensive Translation

The `lang/en/lang.php` file contains all success and error messages and permissions for each resource, making translation to any language easy.

---

### Usage Examples

#### Fetch active classes for a specific stage

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes?stage_id=2&isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### Fetch teacher subject distribution for a specific year with teacher and subject included

```bash
curl -X GET "https://yourdomain.com/api/v1/school/teacher-subjects?year_id=5&include=teacher,subject" \
  -H "Authorization: Bearer <token>"
```

#### Fetch fees for a specific student

```bash
curl -X GET "https://yourdomain.com/api/v1/school/students-fees?student_id=15" \
  -H "Authorization: Bearer <token>"
```

#### Search for a subject by name

```bash
curl -X GET "https://yourdomain.com/api/v1/school/subjects?q=Mathematics" \
  -H "Authorization: Bearer <token>"
```

#### View details of a class with its stage

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes/3?include=stage" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Unprecedented Integration**: For the first time, any external system (mobile app, parent portal, dashboards) can access all school data through a single API.
- **Faster Report Building**: Developers can now create custom reports on classes, subjects, fees, and documents using simple API queries instead of writing complex SQL.
- **Improved User Experience**: Frontends can dynamically fetch dropdown lists (like classes, stages, subjects) from the server.
- **Production-Ready**: The advanced permission system allows precise control over each user's access, making the plugin suitable for real-world deployment.
- **Extensible Design**: `POST` and `PUT` operations can be added for any resource in the future easily without changing the core structure.
- **Excellent Performance**: Using `scopeExclude` with caching ensures fast responses even with thousands of records.

---

### Upgrade Requirements

1. **Install Plugin**: Copy the `nano/schoolapi` folder to `plugins/nano/schoolapi`.
2. **Register Plugin**: Run `php artisan plugin:refresh Nano.SchoolApi`.
3. **Environment Variables**: Adjust `env` variables as needed (e.g., `NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND`).
4. **Permissions**: Ensure roles have the appropriate access permissions in the backend system if `is_check_list_permission` is enabled.
5. **Clear Cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **No database migrations** are required.

---

### Conclusion

Version **1.0.0** of `Nano.SchoolApi` represents a quantum leap in making school data available via API. By covering 14 core entities, the plugin provides a solid foundation for building integrated systems that rely on `Tss.School` data without hassle. The unified design and high extensibility make it an ideal platform for adding more operations and reports in the future.

---

**Reference Documentation**:
- [`Nano.StudyyearApi` Documentation](./docs/StudyyearApi/Docs-StudyyearApi-en.md)
- [`Nano.SchoolApi` Documentation](./docs/SchoolApi/Docs-SchoolApi-en.md)
- [`Nano.StudentsApi` Documentation](./docs/StudentsApi/Docs-StudentsApi-en.md)

## 2026-05-11 - 2026-05-12

**Addition of `Nano.AbsenceApi` – Version 1.0.0**

### Creating a RESTful API for Attendance, Absence, Sheets, and Their Types

---

### Update Summary

The first version **1.0.0** of the `Nano.AbsenceApi` plugin introduces a complete API for interacting with attendance and absence records, attendance sheets, and their classifications. The plugin is built according to the standard `Nano.*Api` pattern, providing protected endpoints for retrieving and managing absence records (`Absences`), sheet files (`Files`), and classifications (`ClassTypes` and `ClassTypeCompanies`). This version also supports creating and updating attendance records via the API, with a multi-level permission system and professional data transformers.

---

### Release Objectives

- **Provide a unified API** for all attendance and absence entities (records, sheets, classifications).
- **Implement a consistent architectural pattern** with other `Nano.*Api` plugins.
- **Enable external applications** to record attendance, modify it, and query reports via the API.
- **Provide fine-grained permission control** for each resource and each operation (read, create, update).
- **Support advanced filtering** by academic year, semester, month, class, group, student, attendance status, and more.
- **Facilitate future expansion** for adding reports or bulk delete operations.

---

### New Features and Improvements

#### 1. Comprehensive RESTful Controllers

Four main controllers were created covering all entities:

| Controller | Targeted Model | Supported Operations |
|------------|----------------|----------------------|
| `Absences` | `Tss\Absence\Models\Absence` | List, view, **create, update** |
| `Files` | `Tss\Absence\Models\File` | List, view |
| `ClassTypes` | `Tss\Absence\Models\ClassType` | List, view |
| `ClassTypeCompanies` | `Tss\Absence\Models\ClassTypeCompany` | List, view |

**Each controller supports:**

- **`index()`**: Fetch a paginated list with multiple filtering, search, sorting, and pagination.
- **`show($id)`**: View details of a single record with relations.
- **`getRecords(array $options)`**: Core function for building complex queries.
- **`activelystats()`**: Endpoint for monitoring last update (for caching).
- **`validationList($user)`**: Unified check for access permissions.

**Specifically for `Absences`:**

- **`store()`**: Create a new absence record with permission checks, input validation, and setting default values (company, branch, year).
- **`update($id)`**: Update an existing record, reusing the same creation logic (function `onStoreUpdateAbsence`).

#### 2. Multi-Level and Customizable Permission System

A separate permission system was applied for each operation (list, create, update) and for each user type (backend, frontend). Control is available via `config.php` using environment variables:

```php
'absences' => [
    'is_allow_list'            => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_LIST_BACKEND', true),
    // ...
    'is_allow_create'          => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_CREATE', true),
    'is_allow_create_backend'  => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_CREATE_BACKEND', true),
    'is_check_create_permission'=> env('NANO_ABSENCEAPI_ABSENCES_IS_CHECK_CREATE_PERMISSION', true),
    // ...
    'is_allow_update'          => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_UPDATE', true),
    // ...
],
```

Additionally, permissions can be linked to backend system permissions (e.g., `tss.absence.absences.add`).

#### 3. Smart Default Values in Attendance Records

When creating a new absence record, if the academic year ID (`year_id`), branch ID (`departments_id`), or company ID (`companys_id`) are not provided, the function automatically uses system default values (such as `Period::getPrimary()`), simplifying quick registration.

**Example from `Absences::onStoreUpdateAbsence()`:**
```php
if (empty($data['companys_id'])) {
    $data['companys_id'] = \Tss\Basic\Helpers\BasicHelper::getCompanysId(true);
}
if (empty($data['departments_id'])) {
    $data['departments_id'] = \Tss\Basic\Helpers\BasicHelper::getMainDepartmentId(true);
}
if (empty($data['year_id'])) {
    $primary = Period::getPrimary();
    if ($primary) $data['year_id'] = $primary->id;
}
```

#### 4. Integrated Data Transformers

Four transformers were created:

- **`AbsenceTransformer`**: Displays attendance record data with included relations (`company`, `department`, `file`, `student`, `record`).
- **`FileTransformer`**: Displays sheet file data with included relations (`company`, `department`).
- **`ClassTypeTransformer`**: Displays main classifications with relations (`company`).
- **`ClassTypeCompanyTransformer`**: Displays company classifications with relations (`company`, `department`, `class_type`).

All transformers support:
- Formatting values to correct types.
- `formatDate` function for dates.
- Field filtering (`exclude`) to reduce data size.
- `object_type` to identify the object type.
- Error handling of relations via `try-catch`.

#### 5. Unified Response for Create and Update Operations

`Absences` `store` and `update` operations follow a unified response structure consistent with other Nano plugins. The response contains:
- `code`, `status`, `message`
- `error`, `errors`
- `data` (transformed data via Transformer)
- `model` (raw model)
- `input_data`, `process_data`
- `debug` (in development mode)

This facilitates error handling in frontends.

#### 6. Advanced Filtering and Search

Numerous filters can be passed to `getRecords` depending on the resource:

- **`Absences`**: `year_id`, `semster`, `month_num`, `class_id`, `group_id`, `student_id`, `record_id`, `absences_status`, `date_at`, `files_id`, `ref_type_class`, etc.
- **`Files`**: `companys_id`, `departments_id`, `ref_type_class`.
- **`ClassTypes`**: `ref_type`.
- **`ClassTypeCompanies`**: `companys_id`, `departments_id`, `ref_type`.

In addition to text search `q` across logical fields (name, code, notes, etc.) and sorting by any column.

#### 7. `scopeExclude` Registration

The dynamic scope `scopeExclude` was registered for all four models (`Absence`, `File`, `ClassType`, `ClassTypeCompany`), allowing the user to select only the required columns and reduce transmitted data.

```php
\Tss\Absence\Models\Absence::extend(function($model) {
    $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
        $getTable = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
        return $query->select(array_diff($getTable, (array) $columns));
    });
});
```

#### 8. Full Translation Support

All success and error messages and permissions are translatable via the `lang/en/lang.php` file, making Arabic or other language translations easy.

---

### Usage Examples

#### Fetch a list of absence records for a specific day with student included

```bash
curl -X GET "https://yourdomain.com/api/v1/absence/absences?date_at=2026-05-01&include=student" \
  -H "Authorization: Bearer <token>"
```

#### Record a new student absence

```bash
curl -X POST "https://yourdomain.com/api/v1/absence/absences" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "15",
    "record_id": "8",
    "year_id": "3",
    "class_id": "2",
    "semster": "semster2",
    "month_num": "2",
    "files_id": "5",
    "ref_type_class": "day",
    "absences_status": "absent",
    "absences_bacuse": "Sick",
    "date_at": "2026-05-05"
  }'
```

#### Update absence status to "present"

```bash
curl -X PUT "https://yourdomain.com/api/v1/absence/absences/42" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "absences_status": "present"
  }'
```

#### Fetch active daily attendance sheets

```bash
curl -X GET "https://yourdomain.com/api/v1/absence/files?ref_type_class=day&isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### Fetch available classifications for the current company

```bash
curl -X GET "https://yourdomain.com/api/v1/absence/class-type-companies" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Attendance Automation**: External applications (mobile apps, notification systems) can record absence directly without using the dashboard.
- **Integration with Reports**: Reporting tools can fetch raw data via API instead of manual CSV exports.
- **High Permission Flexibility**: Allows teachers to record absence via API while delete permissions remain restricted to administrators.
- **Reduced Input Errors**: Validation of input data via API prevents incomplete records.
- **Excellent Performance**: Use of `scopeExclude` and caching ensures fast responses even with thousands of records.

---

### Upgrade Requirements

1. **Install Plugin**: Copy the `nano/absenceapi` folder to `plugins/nano/absenceapi`.
2. **Register Plugin**: Run `php artisan plugin:refresh Nano.AbsenceApi`.
3. **Environment Settings**: Adjust required `env` variables (e.g., `NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_CREATE_FRONTEND=true`).
4. **Permissions**: Ensure roles possess permissions like `tss.absence.absences.add` and `tss.absence.absences.edit` if `is_check_create_permission` and `is_check_update_permission` are enabled.
5. **Clear Cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **No database migrations** are required.

---

### Conclusion

Version **1.0.0** of `Nano.AbsenceApi` represents a quantum leap in managing attendance and absence via API. With its unified and extensible design, developers can now build integrated applications directly and securely relying on attendance data. The plugin covers all basic needs of querying, recording, and updating, with a solid foundation for adding delete operations and advanced reports in future versions.

---

**Reference Documentation**:


- [`Nano.StudyyearApi` Documentation](./docs/StudyyearApi/Docs-StudyyearApi-en.md)
- [`Nano.SchoolApi` Documentation](./docs/SchoolApi/Docs-SchoolApi-en.md)
- [`Nano.StudentsApi` Documentation](./docs/StudentsApi/Docs-StudentsApi-en.md)
- [`Nano.AbsenceApi` Documentation](./docs/AbsenceApi/Docs-AbsenceApi-en.md)

## 2026-05-13 - 2026-05-14

**Updates for `Nano.UserPlus` Plugin – version 1.1.7**

### Summary of Updates

Following support for multiple account types in previous versions, version 1.1.7 arrives to complete building a flexible and integrated user management experience. This release focuses on three main axes:

- **Improving the account type definition mechanism** and making it fully controllable from the config file.
- **Full compatibility** with modern versions of RainLab.User (which use `first_name` and `last_name` instead of `name` and `surname`).
- **Advanced management of user groups** by adding new fields (`is_new_user_default`, `is_active`, `sort_order`) to the `user_groups` table, and extending the backend interface to manage them seamlessly.

These improvements give developers and system administrators finer control over the user experience, unify default group behavior, and ensure the plugin's compatibility with the latest platform versions.

---

### Version 1.1.7 – Flexible Account Types, Improved Compatibility, and Advanced User Group Management

#### Release Goals

- **Make account types (`ref_type`) customizable** from the config file without touching the code.
- **Ensure compatibility** with the name field changes in RainLab.User 2.x via a custom event handler.
- **Extend the `user_groups` table** with columns for activation control, ordering, and default assignment.
- **Integrate the new fields** into the user group management forms and lists in the backend panel.
- **Support multilingual** for all new extensions with safe error handling.

#### New Features

##### 1. Customizable Account Types via Settings

The `getRefTypeList` function in the `Nano\UserPlus\Models\Preference` model has been restructured to be fully dynamic, relying on configuration keys like:
```php
Config::get('nano.userplus::ref_type.department', true)
```
Thus, any account type (such as `delivery`, `partner`, `student`, `employee`...) can be enabled or disabled through the `config.php` file without modifying the source code. The helper functions `getRefTypeOptions` in the user model were also updated to feed dropdowns with the same dynamic options.

##### 2. Compatibility with RainLab.User 2.x

Due to the change of `name` and `surname` fields to `first_name` and `last_name` in recent versions of RainLab.User, an event class `Nano\UserPlus\EventHandlers\RainlabUser2CompatibilityHandler` was added that creates a virtual link between the old and new fields via dynamic methods (`addDynamicMethod`). This ensures any code depending on `name` and `surname` continues to work without errors, while preserving the original data in `first_name` and `last_name`.

This handler is automatically activated when the `first_name` column exists in the `users` table.

##### 3. New Columns in the `user_groups` Table

The migration `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` was added which introduces the following columns to the `user_groups` table:

| Column | Type | Description |
| :--- | :--- | :--- |
| `is_new_user_default` | boolean (default: false) | Determine whether this group is the default for new users. |
| `is_active` | boolean (default: true) | Enable or disable the group (to hide it from lists). |
| `sort_order` | integer (nullable) | Group display order in dropdown lists and interfaces. |

##### 4. Extension of `RainLab\User\Models\UserGroup` Model

Through the `extendUserGroupModel` function in `Plugin.php`:

- Fields were added to `$fillable`.
- Appropriate validation rules (`boolean`, `integer`) were set.
- Smart default values were set when creating a new group.
- Helper functions like `getIsActiveOptions` and `getIsNewUserDefaultOptions` were added to return translated options.

All of this with `try-catch` exception handling and error logging without disrupting the system.

##### 5. Improved User Groups Management Interface

The `RainLab\User\Controllers\UserGroups` controller was extended in two complementary ways:

- **List Columns**: the columns `is_new_user_default` and `is_active` (as a Switch) and `sort_order` were added to the groups list, with sorting and show/hide abilities.
- **Form Fields**: the three fields were injected into the group creation/edit form, with helper comments, distributed over `left`/`right` sections for space organization.

All extensions were done considering context checks (e.g., the form is not nested `isNested`) to ensure no interference with other forms.

##### 6. Full Translation Support

New translation keys were added in the `lang.php` file under the `user_groups` section, including:

- Field names.
- Helper comments.
- Value options (`is_default_yes`, `is_active_yes`...).

The `trans()` function was used in all locations to ensure text appears in the appropriate language.

---

### Practical Examples

#### 1. Customizing Account Types via Settings

In the `config.php` file, you can disable the "Delivery" type, for example:

```php
'ref_type' => [
    'delivery' => false,
    'department' => true,
    // ... remaining types
]
```

Once the file is saved, the "Delivery Worker" options will disappear from `ref_type` dropdowns in user forms.

#### 2. Setting a Default Group for New Users

After the update, when entering the group management, you can select a group and make it the "default group for new users". This is coupled with an event that listens for a new user registration to automatically link them to this group (if the feature is enabled).

#### 3. Sorting Groups in the Frontend

Using the `sort_order` field, groups can be ordered in dropdown lists, improving the user experience during registration or in the backend panel.

---

### Version Summary (1.1.6 – 1.1.7)

| Version | Key Features |
| :--- | :--- |
| 1.1.6 | Support additional user fields (`phone`, `mobile`, `company`...) and extended filters. |
| 1.1.7 | Dynamic account types, `RainlabUser2CompatibilityHandler` compatibility handler, new `user_groups` columns, and extended group management interface. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace the `Plugin.php` file with the new version containing `extendUserGroupModel` and `extendUserGroupsController` functions.
   - Add the file `eventhandlers/RainlabUser2CompatibilityHandler.php`.
   - Update the `models/Preference.php` file if the type-related changes are included.
   - Update the language file `lang.php` to add `user_groups` keys.

2. **Execute migration**:
   - Make sure to run the migration `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` that adds the new columns to the `user_groups` table. This can be executed via `php artisan october:up` or manually.

3. **Update settings** (optional):
   - Review the `config.php` file to customize the desired account types (e.g., `ref_type.delivery` = true/false).

4. **Clear cache**:
   - After updating, run `php artisan cache:clear` to ensure new translation and configuration files are loaded.

5. **Test functionalities**:
   - Verify that the new columns appear in the user groups list.
   - Try creating a new group and enabling the "default group" option, then register a new user to check automatic linkage.
   - Verify that account types (`ref_type`) appear according to `config.php` settings.

---

### Conclusion

With version 1.1.7, we have taken a significant step towards making the `Nano.UserPlus` plugin more flexible and integrated with the platform. Account type management has become fully dynamic, the compatibility challenge with modern versions has been resolved, and user group management has been enhanced with advanced control tools.

These improvements open the door to more customized uses, such as:

- Automatically directing new users to specific groups.
- Hiding inactive groups from interfaces.
- Ordering groups by priority in registration forms.

We look forward to your feedback and suggestions to continue developing this vital plugin.

---

**Reference Documents**:
- [UserPlus Plugin Documentation](./docs/UserPlus/Docs-Nano-UserPlus-en.md)
- [Preference Model Documentation](./docs/UserPlus/Docs-Preference-Model-en.md)
- [UserModelBehavior Documentation](./docs/UserPlus/Docs-UserModelBehavior-en.md)

## 2026-05-6 – 2026-05-15

### Adding BasPay Payment Gateway (BAS Platform) to the Payment System in Nanosoft

Within the `Nano.Yepayment` module

---

### 1. Introduction

A new payment method named **BasPay** has been developed and added to the unified payment system in Nanosoft within the `Nano.Yepayment` module. This addition comes in response to the need to support local payment gateways in Yemen, where **BAS Platform** is an electronic payment platform that allows accepting payments via a direct API with AES-256-CBC encrypted signature.

The BasPay gateway uses a **Direct / Immediate Payment** method, where the amount is deducted immediately upon calling the payment API, without needing to redirect the user to an external page or wait for additional confirmation (unlike ThawaniPay which requires redirection, or YottaPay/QasemiPay which require confirmation). This method is ideal for immediate payments via applications or e-commerce sites where the merchant wants to complete the transaction in the background.

The gateway was developed according to the highest security and quality standards, with token caching, use of a unified `HttpHelper` for all API requests, and a signature generation mechanism perfectly matching BAS Platform requirements (SHA256 + AES-256-CBC), in addition to integrated test interfaces and dedicated API endpoints for developers and administrators. `RedirectHelper` has also been optionally supported to ensure the gateway is compatible with mobile applications that need return URLs after payment.

---

### 2. Developed Components

#### 2.1. `BasPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\BasPay`
- **Inheritance:** extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication with BAS Platform via OAuth2, executing direct payment, checking transaction status, and generating encrypted signature.

**Main Methods:**

| Method | Description |
|--------|-------------|
| `process(PaymentResult $result)` | Execute direct payment: get token, send transaction creation request with signature, save `trxToken`, and update order to paid. |
| `complete(PaymentResult $result)` | Not used in this type (immediate payment), but exists for interface compatibility. |
| `getAuthToken(): ?string` | Request OAuth 2.0 token from BAS via `POST /api/v1/auth/token` (grant_type: client_credentials), caching it for 3500 seconds. |
| `createPayment($token): array` | Send `POST /api/v1/merchant/sdk-payment/initiate-transaction` with request body and signature. |
| `checkTransactionStatus($trxToken): array` | Query transaction status via `POST /api/v1/merchant/sdk-payment/get-transaction-status`. |
| `generateSignature($paramsString): string` | Generate BAS signature: random 4-character salt, SHA256, then AES-256-CBC encryption using Merchant Key and IV. |
| `encryptAes256Cbc($input, $key, $iv): string` | AES-256-CBC encryption function using OpenSSL. |
| `parseResponse($response): array` | Convert HTTP response to PHP array. |
| `getCommonHeaders(): array` | Return required headers for each request (x-client-id, x-app-id, x-sdk-version, ...). |
| `settings(): array` | Define settings fields in the control panel (URL, Client ID, Client Secret, App ID, Merchant Key, IV, default currency). |
| `encryptedSettings(): array` | Specify fields stored encrypted (`baspay_client_secret`, `baspay_merchant_key`). |

#### 2.2. Partial Files (Partials) and Settings

| File | Path | Description |
|------|------|-------------|
| `_info.htm` | `paymenttypes/baspay/_info.htm` | Display setup guidance and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/baspay/_test_info.htm` | Quick test tools within the settings page (buttons for auth, create payment, check status). |
| `baspay-ui.htm` | `views/baspay-ui.htm` | Integrated interactive web interface to test all BasPay functions (manual, automatic, statistics, logs). |

#### 2.3. API Endpoints (`routes.php`)

A complete set of routes has been added within the `yepayment` group to support testing and monitoring:

| Path | Method | Description |
|------|--------|-------------|
| `/baspay/test-auth` | POST | Test authentication and obtain Bearer Token. |
| `/baspay/test-create-payment` | POST | Create an immediate payment for a specified order. |
| `/baspay/test-check-status` | POST | Check transaction status using `trx_token`. |
| `/baspay/test-full-payment` | POST | Comprehensive test (create payment + check status). |
| `/baspay/stats` | GET | Gateway usage statistics (number of requests, success rate). |
| `/baspay/test-ui` | GET | Integrated test interface (HTML). |
| `/baspay/success` | GET | Optional endpoint for redirect after payment (for applications). |
| `/baspay/cancel` | GET | Optional endpoint for redirect on cancellation. |

All routes (except success/cancel) are protected by `BackendAuth::getUser()` to ensure only admin access.

#### 2.4. `Plugin.php` – Registering the Payment Provider

The following line has been added in the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
}
```

#### 2.5. Language Files (Translation)

Translation keys have been added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, general messages, and errors (like `payment_success`, `auth_failed`, `order_already_paid`).

---

### 3. Payment Workflow

BasPay relies on a **Direct (Immediate)** flow without redirection or extra confirmation:

1. **Authentication**  
   `getAuthToken()` is called:
   - Check for `access_token` in Cache.
   - If not present, send `POST /api/v1/auth/token` with `client_id`, `client_secret`, and `grant_type=client_credentials`.
   - Token is cached for 3500 seconds.

2. **Payment Execution (Initiate Transaction)**  
   When `process()` is called:
   - Verify the order is not already paid.
   - Build request body (`amount`, `currency`, `orderId`, `appId`, `requestTimestamp`).
   - Generate signature via `generateSignature()`.
   - Send `POST /api/v1/merchant/sdk-payment/initiate-transaction` with custom headers (`x-client-id`, `x-app-id`, `Authorization`).
   - Receive response: if `status=1` and `code='1111'`, extract `trxToken` from `body.trxToken`.
   - Save `trxToken` in `order.payment_first_trans_id` and `order.payment_trans_id`.
   - Store additional data in `order.other_data['baspay']` (including optional return URLs).
   - Update order status to `PaidState` via `$result->success()`.

3. **Status Check (Check Status)**  
   `checkTransactionStatus($trxToken)` can be called to query the transaction via `POST /api/v1/merchant/sdk-payment/get-transaction-status`.

4. **Optional Return URLs**  
   If `callback_success_url` or `callback_error_url` are passed (e.g., from a mobile app), they are stored in `other_data`. The developer can use the `/baspay/success` and `/baspay/cancel` paths after payment to redirect the user using `RedirectHelper`, making the gateway fully compatible with mobile applications.

---

### 4. Configuration

To activate BasPay, the following settings must be entered in the payment gateway settings interface in the Nanosoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|-----|-------------|---------------|
| Base API URL | `baspay_url` | BAS Platform API address | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | Client identifier from BAS Platform | - |
| Client Secret | `baspay_client_secret` | Secret key (stored encrypted) | - |
| App ID | `baspay_app_id` | Application identifier | - |
| Merchant Key | `baspay_merchant_key` | Merchant key for signature (stored encrypted) | - |
| IV | `baspay_iv` | Initialization vector (16 bytes) | `@@@@&&&&####$$$$` |
| Default Currency | `baspay_default_currency` | Currency used if order does not specify one | `YER` |

**Security Notes:**
- `baspay_client_secret` and `baspay_merchant_key` are stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

### 5. Usage Examples

#### 5.1. Create a Payment for an Existing Order (within a Nanosoft Application)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'amount'   => 100,
    'currency' => 'YER',
    'callback_success_url' => 'myapp://pay/success',   // optional
]);
$result = new PaymentResult($bas, $order);
$processResult = $bas->process($result);

if ($processResult->successful) {
    $trxToken = $order->payment_first_trans_id;
    // Payment succeeded
}
```

#### 5.2. Query Transaction Status

```php
$bas = new BasPay();
$status = $bas->checkTransactionStatus('bas_trx_a1b2c3d4...');
if ($status['success']) {
    // $status['data'] contains transaction details
}
```

#### 5.3. Using the Integrated Test Interface

After logging into the control panel as an admin, open the link:
```
https://yourdomain.com/api/v1/yepayment/baspay/test-ui
```

The interface allows:
- Creating an immediate payment.
- Checking status.
- Automated testing with a specified number of times (up to 10).
- Displaying usage statistics (number of requests, success rate).
- Local test logs.

---

### 6. Handling Redirects (Deeplinks & Callbacks)

Although BasPay is of the direct payment type, **optional** support for redirect URLs has been integrated using `RedirectHelper` (as in ThawaniPay), to ensure compatibility with mobile apps that need to open a specific screen after payment.

- When `callback_success_url` or `callback_error_url` is passed in payment data, they are stored in `order.other_data['baspay']`.
- After executing `process()`, the developer can direct the user to `/baspay/success` (with `order_id`) so the gateway extracts the return URL and performs a smart redirect (deeplink or web) passing status data.
- The same mechanism applies to `/baspay/cancel`.

This makes BasPay fully compatible with various usage patterns without sacrificing the simplicity of direct payment.

---

### 7. Added Value

- **For Developers:** An additional model of a **Direct** payment gateway using OAuth2 with AES-256-CBC encrypted signature, enriching the library of payment patterns in `SKILL-en.md`. It also illustrates how to integrate an external signing mechanism (without an SDK package) within `PaymentProvider`.
- **For Merchants:** Support for the local BAS platform in Yemen, opening the door to accepting electronic payments easily through the merchant's BAS account.
- **For End Users:** A fast and invisible payment experience (executed in the background) without redirection, reducing the payment abandonment rate.

---

### 8. Integration Testing

Several testing layers have been provided:

- **Test Environment:** The same production URL (`https://api.basgate.com`) can be used with trial credentials from BAS Platform. If a future test environment becomes available, its URL can be entered in the `baspay_url` field.
- **Quick Test Interface:** Through the gateway settings page (partial file `_test_info.htm`), authentication, payment creation, and status checking can be tested directly.
- **Integrated Test Interface:** `/api/v1/yepayment/baspay/test-ui` provides all necessary tools to test the gateway comprehensively, with the ability for automated iterative testing.
- **Standalone API Endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints such as `/baspay/test-full-payment`.

**Quick Test Steps:**
1. Ensure Client ID, Client Secret, App ID, Merchant Key, IV are set in the gateway settings page.
2. Open `/api/v1/yepayment/baspay/test-ui`.
3. Click "Test Authentication" to verify data correctness.
4. Enter an order number and amount, then click "Create New Payment".
5. Copy the `trxToken` and use it in the status check field.

---

### 9. Developer Notes

- **BasPay does not rely on redirection:** The `complete()` function is not actually used, but exists to fulfill contract requirements.
- **Token Caching:** Bearer Token is cached for 3500 seconds (less than the actual token validity estimated at 3600 seconds).
- **Encrypted Signature:** The signature mechanism was implemented manually using OpenSSL without relying on an external package, to ensure compatibility with the Nanosoft environment and avoid conflicts.
- **Use of `HttpHelper`:** All API requests (JSON, Form) use the unified `HttpHelper`, making error tracking and gateway expansion easier.
- **Optional Support for `RedirectHelper`:** Redirection capability is included without being mandatory, making the gateway suitable for both background (Backend) use and scenarios requiring user interaction.
- **Extensibility:** Support for Webhook or refunds can be added in the future via additional methods using different BAS endpoints.

---

### 10. Bug Fixes

None – this release is dedicated to adding a new feature only (BasPay).

---

### 11. Development and Testing Period

The BasPay gateway was fully developed and integrated into the `Nano.Yepayment` system during the period from **May 6, 2026** to **May 15, 2026**. This period included writing the core code for the BasPay class and implementing the AES-256-CBC encrypted signature mechanism, creating the partial interface files (_info.htm, _test_info.htm) and the integrated test interface (baspay-ui.htm), as well as writing the necessary API endpoints for testing and monitoring. Comprehensive integration tests were also conducted to ensure the gateway works correctly with the BAS Platform environment, and signature compatibility with received responses was verified.

---

### 12. Related Links

- [BasPay Documentation (Developer Guide)](./docs/BasPay/Docs-BasPay-en.md)
- [SKILL-en.md Document for Creating Payment Gateways](./docs/BasPay/SKILL-en.md)
- [BAS Platform API Guide](./docs/BasPay/external/BAS/README-en.md)
- [BasGate Documentation Repository on GitHub](https://github.com/basgate/basgate.github.io)
- [Laravel Payment SDK Package on GitHub](https://github.com/basgate/laravel-payment-sdk)
- [BasPaymentFlutter Repository on GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [bas_php_sdk Package on GitHub](https://github.com/basgate/bas_php_sdk)
- [bas-laravel-sdk Package on GitHub](https://github.com/basgate/bas-laravel-sdk)
- [Technical Support Channel](https://nano2soft.com)

---

**This update was prepared by:**  
Nanosoft Development Team – Electronic Payments Department  
**Reviewer:** Dheia Ali, Nano2Soft

