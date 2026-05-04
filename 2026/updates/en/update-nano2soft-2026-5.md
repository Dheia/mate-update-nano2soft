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


