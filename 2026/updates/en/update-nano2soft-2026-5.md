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


## 2026-05-15 - 2026-05-16

**Updates for `Nano.AuthApi` Plugin – version 1.0.19**

### Summary of Updates

After providing comprehensive management for user profiles and login, version 1.0.19 arrives to enhance the developer experience by **extending the API with a set of new endpoints** that enable:

- **Fetching ready field options** (such as gender, account type, language, nationality...) dynamically.
- **Managing user group lists** with support for advanced filters including default status and ordering.
- **Full support for new columns** (`is_new_user_default`, `is_active`, `sort_order`) in the `user_groups` table.

This release aims to empower frontend applications to build dynamic registration and search forms without multiple requests, and to unify the mechanism for accessing system options via a professional API.

---

### Version 1.0.19 – Field Options, User Group Endpoints, and Structural Expansion

#### Release Goals

- **Provide endpoints to fetch field options** (`/user/options`, `/user/frontend-options`, `/user/backend-options`).
- **Launch the `UserGroups` controller** to manage user groups via API with filters compatible with the new columns.
- **Update `UserGroupTransformer`** to include new fields (`is_active`, `is_new_user_default`, `sort_order`).
- **Execute a migration** to add these columns to the `user_groups` table.
- **Update config file** (`config.php`) to add settings specific to user groups (`user_groups`).

#### New Features

##### 1. Field Options Endpoints

The `Profiles` controller was extended with new functions to extract all necessary options for building user forms:

| Path | Description |
| :--- | :--- |
| `GET /api/v1/user/options` | Fetch general options (fields can be specified via `fields`). |
| `GET /api/v1/user/frontend-options` | Same options but dedicated to the frontend. |
| `GET /api/v1/user/backend-options` | Options dedicated to the backend panel (backend users). |

**Responsible functions:**
- `getOptions($data)`: the main function that extracts options based on the `fields` parameter (e.g., `gender`, `ref_type`, `language`, `nationality`, `passport_type`...).
- `getAutoptions()`, `getFrontendOptions()`, `getBackendOptions()`: wrapper functions to automatically determine the `provider`.
- `getCollectionItem($items)`: converts associative arrays to a collection of objects `{id, name}` for uniform formatting.

**Currently supported field options:**

| Field | Description |
| :--- | :--- |
| `gender_type` | Additional gender types (if any) |
| `gender` | Gender options (male/female) |
| `ref_type` | Available account types (regular user, delivery worker...) |
| `language` | List of supported languages |
| `nationality` | List of nationalities |
| `passport_type` | Passport types |
| `marital_status` | Marital status |
| `fuel_type` | Fuel types |
| `website_type` | Website types (website, facebook) |
| `phone_type` | Phone types (mobile, home, work...) |

##### 2. `UserGroups` Controller for Managing User Groups

`Nano\AuthApi\Controllers\UserGroups` was created to provide:

- **`index()`**: fetch list of groups with support for filters (code, status, default group, text search), ordering, and pagination.
- **`show($id)`**: fetch details of a single group.
- **`activelystats()`**: statistics about the last update (for caching purposes).

**Key features of `getRecords()`:**
- Support for `provider` (frontend/backend) to choose the appropriate group model.
- Filter `is_new_user_default` to limit results to the default group for new users.
- Filter `code`, `status`, and `q` (search in name, description, and code).
- Support for direct return options: `is_collection`, `is_paginator`, `is_query`.
- Events `api.list.extendQueryBefore` and `api.list.extendQuery` to allow developers to customize the query.
- Use of `UserGroupTransformer` for output formatting.

##### 3. Support for New Columns in `user_groups`

The following columns were added via the migration `builder_table_add_is_active_columns_to_frontend_user_groups_table.php`:

| Column | Type | Description |
| :--- | :--- | :--- |
| `is_new_user_default` | boolean | Set the group as the default for new users. |
| `is_active` | boolean | Enable/Disable the group. |
| `sort_order` | integer | Order of the group in lists. |

**Duplicate prevention**: uses the condition `if(!Schema::hasColumn(...))` to avoid errors when running the update on databases that already contain the columns.

##### 4. Updates in `UserGroupTransformer`

The new fields were added to the JSON response to include `is_active`, `is_new_user_default`, `sort_order` alongside the basic fields (`id`, `name`, `code`, `description`, dates). This ensures all group endpoints are compatible with this data.

##### 5. New Settings in `config.php`

A key `user_groups` was added to the `config.php` file:

```php
'user_groups' => [
    'is_allow_list' => true,
    'is_allow_list_backend' => true,
    'is_allow_list_frontend' => true,
    'is_check_access_list' => false,
    'is_check_list_permission' => false,
    'order_by' => 'code',
    'order_dir' => 'asc',
    'per_page' => 15,
    'exclude' => '',
],
```

These settings allow controlling the behavior of endpoints and their access privileges.

---

### Practical Examples

#### 1. Fetching Gender and Account Type Options

**Request:**
```http
GET /api/v1/user/options?fields=gender,ref_type HTTP/1.1
Authorization: Bearer ...
```

**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Options fetched successfully",
    "data": {
        "gender": [
            {"id": "male", "name": "Male"},
            {"id": "female", "name": "Female"}
        ],
        "ref_type": [
            {"id": "user", "name": "Regular User"},
            {"id": "department", "name": "Facility or Company"},
            ...
        ]
    }
}
```

#### 2. Fetching Only Active Groups Ordered by `sort_order`

```http
GET /api/v1/user/groups?is_active=1&orderBy=sort_order&orderDirection=asc HTTP/1.1
```

#### 3. Fetching the Default Group for New Users

```http
GET /api/v1/user/groups?is_new_user_default=1 HTTP/1.1
```

#### 4. Fetching Details of a Specific Group

```http
GET /api/v1/user/groups/1 HTTP/1.1
```

---

### Version Summary (1.0.18 – 1.0.19)

| Version | Key Features |
| :--- | :--- |
| 1.0.18 | Support `bio`, `website`, `links` fields in the profile. |
| 1.0.19 | Field options endpoints (`/user/options`), `UserGroups` controller for managing groups, support for `is_new_user_default`, `is_active`, `sort_order` columns, and updates to transformers and settings. |

---

### Upgrade Requirements

1. **Update code**:
   - Add the new `UserGroups.php` controller in the specified path.
   - Add the new functions in `Profiles.php` (if not already present).
   - Update `UserGroupTransformer.php` to include the new fields.
   - Update `routes.php` file to add the new routes.

2. **Execute migration**:
   - Run the migration `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` to add the new columns. You can use `php artisan october:up` or upgrade the plugin from the backend.

3. **Review settings**:
   - Update the `config.php` file to include the `user_groups` key with appropriate default settings.

4. **Test new endpoints**:
   - Try `GET /api/v1/user/options?fields=gender,ref_type,language`.
   - Try `GET /api/v1/user/groups` with different filter parameters.

5. **Clear cache** (optional):
   - `php artisan cache:clear` to ensure new settings are loaded.

---

### Conclusion

With version 1.0.19, we have taken an important step towards making `Nano.AuthApi` a comprehensive API for managing users and their groups. The new endpoints for providing options reduce dependence on separate interfaces, while `UserGroups` provides powerful search and filtering tools suited for modern applications.

This integration opens the door to building fully dynamic registration and preference experiences, and makes it easier for developers to create flexible dashboards that rely completely on APIs.

We look forward to your feedback and suggestions to continue developing this vital plugin.

## 2026-05-06 – 2026-05-18

### Adding and Updating the BasPay Payment Gateway (BAS Platform) in the NanoSoft Payment System

Within the software module `Nano.Yepayment`

---

### 1. Introduction

A new payment method named **BasPay** was developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition comes in response to the need to support local payment gateways in Yemen, where the **BAS Platform** is one of the electronic payment platforms that allows accepting payments via a direct API with AES-256-CBC encrypted signature.

After careful examination of the payment flow on the BAS platform, **the gateway was updated** to operate according to the **Two‑Step Payment** pattern, which is the pattern that matches the nature of the platform where:
1. **Step One (process):** A transaction is created and a `trxToken` is obtained, without the amount being deducted yet.
2. **Step Two (complete):** After the customer completes the payment via the BAS app, the transaction status is verified and the order is updated to "paid".

This update made the gateway fully compatible with `OrderManager`, `Checkout2`, and `PaymentResult`, and prevents the order from moving to paid prematurely.

The gateway was developed to the highest security and quality standards, with token caching, use of the unified `HttpHelper` for all API requests, and a mechanism for generating a signature that exactly matches the BAS platform requirements (SHA256 + AES-256-CBC). Additionally, comprehensive test interfaces and dedicated API endpoints for developers and administrators were provided. `RedirectHelper` was also supported to ensure compatibility with mobile apps that need return URLs after payment.

---

### 2. Developed Components

#### 2.1. `BasPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\BasPay`
- **Inheritance:** Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication with the BAS platform via OAuth2, executing step one (transaction creation), step two (status check and payment confirmation), and generating the encrypted signature.

**Main Functions:**

| Function | Description |
|----------|-------------|
| `process(PaymentResult $result)` | **Step one:** Creates a transaction via `initiateTransaction`, receives `trxToken`, and saves it in the order without changing payment status. Returns success with a confirmation message. |
| `complete(PaymentResult $result)` | **Step two:** Checks transaction status via `checkTransactionStatusByToken`, and if status is `SUCCESS` calls `$result->success()` to update the order to `PaidState`. |
| `checkAndCompletePay(array $options): array` | Public `static` function used in the `baspay/success` route to link the return from the app with the confirmation process. |
| `getAuthToken(): ?string` | Requests an OAuth 2.0 token from BAS via `POST /api/v1/auth/token` (grant_type: client_credentials), and caches it for 3500 seconds. |
| `initiateTransaction($token): array` | Sends `POST /api/v1/merchant/sdk-payment/initiate-transaction` with the request body and signature. |
| `checkTransactionStatusByToken($token, $trxToken): array` | Queries transaction status via `POST /api/v1/merchant/sdk-payment/get-transaction-status` (used internally). |
| `checkTransactionStatus($trxToken): array` | Public function to query a transaction status (for external use). |
| `generateSignature($paramsString): string` | Generates BAS signature: random salt (4 chars), SHA256, then AES-256-CBC encryption using Merchant Key and IV. |
| `encryptAes256Cbc($input, $key, $iv): string` | AES-256-CBC encryption function using OpenSSL. |
| `parseResponse($response): array` | Converts HTTP response to a PHP array. |
| `getCommonHeaders(): array` | Returns required headers for every request (x-client-id, x-app-id, x-sdk-version, ...). |
| `settings(): array` | Defines settings fields in the control panel (URL, Client ID, Client Secret, App ID, Merchant Key, IV, default currency). |
| `encryptedSettings(): array` | Specifies fields that are stored encrypted (`baspay_client_secret`, `baspay_merchant_key`). |

#### 2.2. Partials and Settings

| File | Path | Description |
|------|------|-------------|
| `_info.htm` | `paymenttypes/baspay/_info.htm` | Displays setup instructions and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/baspay/_test_info.htm` | Quick test tools within the settings page (authentication, create payment, status check buttons). |
| `baspay-ui.htm` | `views/baspay-ui.htm` | Full interactive web interface for testing all BasPay functions (manual, automatic, statistics, logs). |

#### 2.3. API Endpoints (`routes.php`)

A complete set of routes was added within the `yepayment` group to support testing and monitoring:

| Route | Method | Description |
|-------|--------|-------------|
| `/baspay/test-auth` | POST | Test authentication and obtain Bearer Token. |
| `/baspay/test-create-payment` | POST | Create a new transaction (step one). |
| `/baspay/test-check-status` | POST | Check a transaction's status using `trx_token`. |
| `/baspay/test-full-payment` | POST | Full test (create transaction + check status). |
| `/baspay/stats` | GET | Gateway usage statistics (request count, success rate). |
| `/baspay/test-ui` | GET | Full test interface (HTML). |
| `/baspay/success` | GET | Endpoint to confirm payment after user returns from BAS app. |
| `/baspay/cancel` | GET | Optional endpoint for cancellation redirect. |

All routes (except success/cancel) are protected with `BackendAuth::getUser()` to ensure only administrators can access them.

#### 2.4. `Plugin.php` – Payment Provider Registration

The following line was added in the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
}
```

#### 2.5. Language Files (Translation)

Translation keys were added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, public messages, and errors (such as `payment_success`, `auth_failed`, `order_already_paid`).

---

### 3. Payment Workflow – Two‑Step Pattern

BasPay operates according to a **Two‑Step** flow:

1. **Authentication**  
   `getAuthToken()` is called:
   - Checks for `access_token` in Cache.
   - If not present, sends `POST /api/v1/auth/token` with `client_id`, `client_secret`, and `grant_type=client_credentials`.
   - The token is stored in cache for 3500 seconds.

2. **Step One – Transaction Creation (`process`)**  
   - Checks that the order is not already paid.
   - Builds the request body (`amount`, `currency`, `orderId`, `appId`, `requestTimestamp`).
   - Generates the signature via `generateSignature()`.
   - Sends `POST /api/v1/merchant/sdk-payment/initiate-transaction`.
   - Receives `trxToken` and stores it in `order.payment_first_trans_id` and `order.other_data['baspay']`.
   - **Does not call `$result->success()`**, and the order does not change to `PaidState`. Instead, the gateway returns `successful = true` with a `confirmation_required` message.

3. **Step Two – Payment Confirmation (`complete`)**  
   - Called from the `baspay/success` route after the user returns from the BAS app.
   - Extracts `trxToken` from the order, and communicates with BAS via `POST /api/v1/merchant/sdk-payment/get-transaction-status`.
   - If `trxStatus` equals `SUCCESS` or `COMPLETED`:
     - `$result->success()` is called, which **changes the order status to `PaidState`** and triggers events (like `nano.orders.paymentProcessed`).
   - If `PENDING`: `$result->pending()` is called and the order remains pending.

4. **Optional Return URLs**  
   When `callback_success_url` or `callback_error_url` are provided, they are stored in `other_data`. After confirmation, the developer can use `RedirectHelper` to intelligently redirect the user (Deep Link or web).

---

### 4. Configuration

To enable BasPay, the following settings must be entered in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|-----|-------------|---------------|
| Base API URL | `baspay_url` | BAS Platform API address | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | Client identifier from BAS platform | - |
| Client Secret | `baspay_client_secret` | Secret key (stored encrypted) | - |
| App ID | `baspay_app_id` | Application ID | - |
| Merchant Key | `baspay_merchant_key` | Merchant key for signing (stored encrypted) | - |
| IV | `baspay_iv` | Initialization vector (16 bytes) | `@@@@&&&&####$$$$` |
| Default Currency | `baspay_default_currency` | Currency used if the order does not specify one | `YER` |

**Security Notes:**
- `baspay_client_secret` and `baspay_merchant_key` are stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

### 5. Usage Examples

#### 5.1. Initiate Payment – Step One (Transaction Creation)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'callback_success_url' => 'myapp://pay/success',   // Optional
]);
$result = new PaymentResult($bas, $order);
$processResult = $bas->process($result);

if ($processResult->successful) {
    // Transaction created and trxToken saved, but payment not yet complete
    $trxToken = $order->payment_first_trans_id;
    // User must be directed to the BAS app to complete payment
}
```

#### 5.2. Confirm Payment – Step Two (Status Verification)

After the customer completes payment in the BAS app, the process can be confirmed as follows:

```php
$result = \Nano\Yepayment\PaymentTypes\BasPay::checkAndCompletePay(['order_id' => 200]);
if ($result['success']) {
    // Payment confirmed and order updated to PaidState
}
```

#### 5.3. Direct Status Query for a Transaction

```php
$bas = new BasPay();
$status = $bas->checkTransactionStatus('bas_trx_a1b2c3d4...');
if ($status['success']) {
    // $status['data'] contains transaction details
}
```

#### 5.4. Using the Full Test Interface

After logging into the control panel as an administrator, open the URL:
```
https://yourdomain.com/api/v1/yepayment/baspay/test-ui
```

The interface allows:
- Creating a new transaction (Initiate).
- Checking status.
- Automatic testing with a specified number of repetitions (up to 10).
- Viewing usage statistics (request count, success rate).
- Local logs for tests.

---

### 6. Handling Redirects (Deeplinks & Callbacks)

Optional support for redirect URLs has been integrated using `RedirectHelper`, to ensure compatibility with mobile apps that need to open a specific screen after payment.

- When `callback_success_url` or `callback_error_url` are passed in payment data, they are stored in `order.other_data['baspay']`.
- After confirming payment via `complete()`, the developer can direct the user to `/baspay/success` (with `order_id`) so that the gateway extracts the return URL and performs intelligent redirection (deeplink or web) with status data.
- The same mechanism applies to `/baspay/cancel`.

---

### 7. Added Value

- **For developers:** A comprehensive model of a **Two‑Step** payment gateway that uses OAuth2 with AES-256-CBC encrypted signature, illustrating how to build gateways that require external confirmation without an SDK. It also demonstrates full integration with `OrderManager`, `PaymentResult`, and `RedirectHelper`.
- **For merchants:** Support for the local BAS platform in Yemen, opening the door to easily accept electronic payments through the merchant's BAS account.
- **For end users:** A reliable two‑step payment experience: confirm the order in the store, then complete payment in the BAS app.

---

### 8. Integration Testing

Several testing layers were provided:

- **Test environment:** The same production URL (`https://api.basgate.com`) can be used with trial credentials from the BAS platform. If a future test environment becomes available, its URL can be entered in the `baspay_url` field.
- **Quick test interface:** From the gateway settings page (partial `_test_info.htm`), you can test authentication, create a transaction, and check status directly.
- **Full test interface:** `/api/v1/yepayment/baspay/test-ui` provides all necessary tools to test the gateway thoroughly (including simulating both steps), with the ability to perform automated repeat testing.
- **Independent API endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints like `/baspay/test-full-payment`.

**Quick test steps:**
1. Ensure Client ID, Client Secret, App ID, Merchant Key, IV are set in the gateway settings page.
2. Open `/api/v1/yepayment/baspay/test-ui`.
3. Click "Test Authentication" to verify credentials.
4. Enter an order ID and amount, then click "Create New Transaction".
5. A `trxToken` will appear – you can use it in "Check Status" to simulate the second step.

---

### 9. Developer Notes

- **Two‑Step pattern:** `process()` **does not** complete the payment; it only creates the transaction. You must call `complete()` or `checkAndCompletePay` to confirm payment and update the order to `PaidState`.
- **Token caching:** The Bearer Token is cached for 3500 seconds (slightly less than the actual token expiry, estimated at 3600 seconds).
- **Encrypted signature:** The signature mechanism was implemented manually using OpenSSL without relying on an external package, to ensure compatibility with the NanoSoft environment and prevent conflicts.
- **`HttpHelper` usage:** All API requests (JSON, Form) use the unified `HttpHelper`, simplifying error tracking and gateway expansion.
- **`RedirectHelper` support:** Full redirection capability is included, making the gateway suitable for mobile apps that use Deep Links.
- **Extensibility:** Webhook or refund support can be added in the future via additional functions using different BAS endpoints.

---

### 10. Bug Fixes

None – this release is dedicated to adding and updating a new feature only (BasPay).

---

### 11. Development and Testing Period

The BasPay gateway was initially developed as a direct payment, then **updated** to the two‑step pattern during the period from **May 6, 2026** to **May 18, 2026**. This period included writing the core code for the BasPay class and implementing the AES-256-CBC encrypted signature mechanism, creating partial interface files (_info.htm, _test_info.htm) and the full test interface (baspay-ui.htm), as well as writing the necessary API endpoints for testing and monitoring, and modifying `process()` and `complete()` to align with the two‑step flow. Comprehensive integration tests were also conducted to verify the gateway's correct operation with the BAS platform environment, and signature compatibility with the responses received was confirmed.

---

### 12. Related Links

- [BasPay Documentation (Developer Guide)](./docs/BasPay/Docs-BasPay-en.md)
- [SKILL-en.md for creating payment gateways](./docs/BasPay/SKILL-en.md)
- [BAS Platform API Guide](https://basgate.apidog.io)
- [BAS Platform API Guide](./docs/BasPay/external/BAS/README-en.md)
- [BasGate documentation repository on GitHub](https://github.com/basgate/basgate.github.io)
- [Laravel Payment SDK on GitHub](https://github.com/basgate/laravel-payment-sdk)
- [BasPaymentFlutter repository on GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [bas_php_sdk on GitHub](https://github.com/basgate/bas_php_sdk)
- [bas-laravel-sdk on GitHub](https://github.com/basgate/bas-laravel-sdk)
- [Technical Support Channel](https://nano2soft.com)

---

**This update was prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**Reviewer:** Dheia Ali, Nano2Soft

## 2026-05-18 - 2026-05-19

### Comprehensive Update for Query Scopes in `ProposalModel` Behavior

**Complete Development of Query Scopes and Helper Functions within the `Nano2.Proposals` Package**

An integrated development package has been implemented aimed at improving the performance and flexibility of queries related to the Proposals/Reports/Complaints system in NanoSoft applications. Developers no longer need to write complex queries or incorrectly rely on `GROUP BY` and `LIMIT` within subqueries. Instead, they now have a set of ready-made scopes that enable:

- Ordering by **report count** (with the ability to distinguish report types: `proposals`, `reports`, `complaints`).
- Ordering by **latest report date** (most recent).
- Ordering by **earliest report date** (oldest).
- Adding calculated columns to `SELECT` (e.g., `proposals_count`, `latest_proposal_at`) without affecting the ordering.
- Filtering objects that the user has reported (as `target`).
- Filtering objects that have sent a report (as `user`).
- Filtering objects that have reports (with a minimum count) or that have no reports.
- Adding the column `is_proposed_by_user` (did the user send a report for the object) and `is_proposed_to_user` (is the object the target of a report sent by the user).
- An advanced scope (`scopeWhereHasProposalAdvanced`) supporting both sides (`target` and `user`) with full flexibility, control over `user_id`/`user_type` and `target_id`/`target_type` conditions, use of `has`/`orHas`/`doesntHave`/`orDoesntHave`, and support for `withTrashed`/`onlyTrashed` as special keywords.
- Advanced statistical functions to get total reports, distribution by user type, and the list of reporting users.

Existing scopes have been restructured and new scopes added, while maintaining backward compatibility via wrapper functions marked as `@deprecated`. All scopes and helper functions have been moved to an independent trait `ProposalScopesAndHelpers` to facilitate maintenance and reuse.

---

### 1. Developed Components

| Behavior | Component | Description |
|----------|-----------|-------------|
| `ProposalModel` | `ProposalScopesAndHelpers` (trait) | Contains all advanced scopes and helper functions for the reporting system. |

#### New and Improved Scopes

| Category | Scope | Description |
|----------|-------|-------------|
| **Report Count** | `scopeAddCountProposals` | Add a report count column to `SELECT` (without ordering). |
| | `scopeSortByCountProposals` | Order results by report count (without adding the column). |
| | `scopeWithCountProposals` | Add the column and ordering together. |
| **Latest Report Date** | `scopeAddLatestProposal` | Add a column with the latest report date. |
| | `scopeSortByLatestProposal` | Order by the latest report date. |
| | `scopeWithLatestProposal` | Add the column and ordering together. |
| **Earliest Report Date** | `scopeAddEarliestProposal` | Add a column with the earliest report date. |
| | `scopeSortByEarliestProposal` | Order by the earliest report date. |
| | `scopeWithEarliestProposal` | Add the column and ordering together. |
| **Filtering by User (as Target)** | `scopeProposedByUser` | Filter objects that the user has reported. |
| | `scopeNotProposedByUser` | Filter objects that the user has not reported. |
| **Filtering by User (as Sender)** | `scopeProposedToUser` | Filter objects that have sent a report (i.e., the object is the sender). |
| | `scopeNotProposedToUser` | Filter objects that have not sent a report. |
| **Filtering by Existence of Reports** | `scopeHasProposals` | Filter objects that have reports (with a minimum count). |
| | `scopeHasNoProposals` | Filter objects that have no reports. |
| **Status Column for User** | `scopeWithIsProposedByUser` | Add the `is_proposed_by_user` column. |
| | `scopeWithIsProposedToUser` | Add the `is_proposed_to_user` column. |
| **Advanced Scope** | `scopeWhereHasProposalAdvanced` | Flexible scope for filtering by reports with full support for both sides (target/user), advanced conditions, and multiple relationship types (`has`, `orHas`, `doesntHave`, `orDoesntHave`). |
| **Special Scopes** | `scopeTopProposed` | Retrieve the objects with the most reports (by report count). |

#### Helper Statistical Functions

| Function | Description |
|----------|-------------|
| `getTotalProposals` | Total number of reports (sum of `COUNT`) with filtering options. |
| `getProposalsCountByType` | Distribution of reports by user type (`user_type`). |
| `getProposersUsers` | List of users who sent reports for the object. |

All functions and scopes support filtering options: `$type` (report type), `$onlyActive`, `$withTrashed`.

---

### 2. Details of Code Updates

#### 2.1 Fixing Subquery Errors
Old scopes in the original behavior relied on using `GROUP BY` and `LIMIT` inside subqueries to get aggregated values (e.g., `COUNT`, `MAX`). This approach leads to incorrect and unpredictable results.

**New Approach**:
- Use an aggregate function directly in the subquery without `GROUP BY` or `LIMIT`.
- Write the full table name before each field to avoid ambiguity.
- Add additional conditions (e.g., `type`, `is_active`) via `where` inside the subquery.

#### 2.2 Standardizing Scope Interface
Function signatures have been standardized to include:
- `$orderDirection`: Sorting direction (default `DESC`).
- `$columnName`: Name of the added column (appropriate default like `proposals_count`, `latest_proposal_at`).
- `$type`: Filter by report type (`proposals`, `reports`, `complaints`).
- `$onlyActive`: Filter by `is_active`.
- `$withTrashed`: Include soft-deleted records.

#### 2.3 Adding the Advanced Scope `scopeWhereHasProposalAdvanced`
This scope is the new core for filtering by reports, supporting:
- **Required Side** (`side`): `target` (reports received by the object) or `user` (reports sent from the object).
- **Relationship Type** (`type`): `has`, `orHas`, `doesntHave`, `orDoesntHave`.
- **Flexible Conditions** (`conditions`): Can be an array of fields and values, a closure, or an array supporting advanced queries (e.g., `['whereIn', [1,2,3]]`).
- **Control over Applying User/Target Conditions** (`useUserConditions`, `useTargetConditions`): Can be set to `true` (apply all), `false` (apply none), or an array containing the required field names (e.g., `['user_type']`).
- **Control over Forcing Conditions When Entity is Absent** (`isForceUser`, `isForceTarget`): If `true` and the user (or target) does not exist while its conditions are required, an impossible condition (`whereRaw('1 = 0')`) is added to return no results. If `false`, no condition is added.
- **Source for Obtaining User/Target** (`userSource`, `targetSource`): Can be `'current_user'` (fetch from AuthHelpers) or `'current_model'` (use the current model). Reads from settings or can be passed.
- **Support for `withTrashed` and `onlyTrashed`**: Handled as special keywords within `conditions`, and the appropriate scope is called on the subquery.

#### 2.4 Supporting `withTrashed` and `onlyTrashed` as Keywords
In `$conditions`, passing `'withTrashed' => true` or `'onlyTrashed' => true` will call the appropriate function (`withTrashed()` or `onlyTrashed()`) on the subquery instead of treating them as `where` conditions.

#### 2.5 Moving Scopes to a Separate Trait
To facilitate maintenance and reuse, all scopes and helper functions have been moved to a new trait:
`Nano2\Proposals\Behaviors\ProposalModel\ProposalScopesAndHelpers`

#### 2.6 Backward Compatibility
Old scopes have been retained as wrapper functions in the main class, with an `@deprecated` comment. Functions `scopeWhereHasProposal`, `scopeWhereHasProposals`, `scopeWhereHasUserProposal`, `scopeWhereHasUserProposals` are also provided, redirecting users to the advanced scope.

**List of Supported Old Functions:**
- `scopeSortByCountProposalsOld` → `scopeSortByCountProposals`
- `scopeWithSortByCountProposals` → `scopeWithCountProposals`
- `scopeAddSortByCountProposals` → `scopeAddCountProposals`
- `scopeSortByCreatedAtProposals` → `scopeSortByLatestProposal`
- `scopeAddSortByCreatedAtProposals` → `scopeAddLatestProposal`
- `scopeWithSortByCreatedAtProposals` → `scopeWithLatestProposal`
- `scopeWhereHasProposal` → `scopeWhereHasProposalAdvanced` (target side)
- `scopeWhereHasProposals` → `scopeWhereHasProposalAdvanced`
- `scopeWhereHasUserProposal` → `scopeWhereHasProposalAdvanced` (user side)
- `scopeWhereHasUserProposals` → `scopeWhereHasProposalAdvanced`

---

### 3. Practical Examples

#### 3.1 Ordering by Report Count
```php
// Products with the most reports (all types)
$products = Product::sortByCountProposals('DESC')->get();

// Products with the most reports of type 'complaints'
$products = Product::sortByCountProposals('DESC', 'complaints')->get();

// Add report count column with ordering
$products = Product::withCountProposals('DESC', 'proposals_count', 'reports')->get();
```

#### 3.2 Ordering by Date
```php
// Products with the most recent report (all types)
$products = Product::sortByLatestProposal('DESC')->get();

// Products with the most recent report of type 'reports'
$products = Product::sortByLatestProposal('DESC', 'latest_report_at', 'reports')->get();

// Add latest report date column with ordering
$products = Product::withLatestProposal('DESC', 'last_proposal_date')->get();
```

#### 3.3 Filtering by User (Target Side)
```php
// Products that the current user has reported
$products = Product::proposedByUser()->get();

// Products that a specific user has reported, of type 'complaints'
$user = User::find(1);
$products = Product::proposedByUser($user, 'complaints')->get();

// Products that the current user has not reported
$products = Product::notProposedByUser()->get();
```

#### 3.4 Filtering by User (User Side – Object is the Sender)
```php
// Users who have sent reports (as senders)
$users = User::proposedToUser()->get();

// Users who have sent reports of type 'reports'
$users = User::proposedToUser(null, 'reports')->get();
```

#### 3.5 Filtering by Existence of Reports
```php
// Products that have more than 5 reports of type 'complaints'
$products = Product::hasProposals(5, 'complaints')->get();

// Products that have no reports
$products = Product::hasNoProposals()->get();
```

#### 3.6 Adding a Status Column for the Current User
```php
$products = Product::withIsProposedByUser()->get();
foreach ($products as $product) {
    echo $product->is_proposed_by_user ? 'You have reported this' : 'You have not reported this';
}
```

#### 3.7 Advanced Usage of the `scopeWhereHasProposalAdvanced` Scope

```php
// Products that have reports of type 'reports' (any user)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->get();

// Products that have active reports of type 'complaints' from the current user
$products = Product::whereHasProposalAdvanced([
    'side' => 'target',
    'conditions' => ['type' => 'complaints', 'is_active' => true],
])->get();

// Products that have no reports of type 'proposals'
$products = Product::whereHasProposalAdvanced([
    'type' => 'doesntHave',
    'conditions' => ['type' => 'proposals']
])->get();

// Using a closure for complex conditions
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();

// Using whereIn on the report type
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => ['whereIn', ['reports', 'complaints']],
        'is_active' => true,
    ]
])->get();

// Controlling the application of user conditions (target side)
$products = Product::whereHasProposalAdvanced([
    'useUserConditions' => false, // do not apply user_id/user_type
    'conditions' => ['type' => 'reports']
])->get();

// Searching for reports of a specific type, including soft-deleted ones
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'complaints',
        'withTrashed' => true,
    ]
])->get();

// Controlling the user source when side=user
$users = User::whereHasProposalAdvanced([
    'side' => 'user',
    'userSource' => 'current_model', // use the current model as the user
])->get();
```

#### 3.8 Statistical Functions
```php
$product = Product::find(1);
echo "Total reports of type 'reports': " . $product->getTotalProposals('reports');
print_r($product->getProposalsCountByType('reports')->toArray());
$users = $product->getProposersUsers('reports');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

---

### 4. Added Value

- **For Developers**: An integrated set of ready-made scopes saves time and reduces errors. The unified interface makes learning and usage easy. The advanced scope `scopeWhereHasProposalAdvanced` meets all complex use cases.
- **For End Users**: The ability to present intelligently ordered lists (most reported, latest reported) improves the user experience.
- **For the System**: Better performance through optimized SQL queries, eliminating inefficient queries that incorrectly used `GROUP BY` and `LIMIT`.
- **Flexibility**: Ability to filter by report type, activity status, user, and side (target/user), with full control over user/target conditions.
- **Scalability**: Adding new scopes for any future behavior follows the same pattern, ensuring code consistency and ease of maintenance.

---

### 5. Conclusion

This update represents a qualitative shift in managing report, proposal, and complaint queries within NanoSoft applications. By restructuring scopes and moving them to an independent trait, along with adding advanced functions for ordering, filtering, and statistics, developers can now build sophisticated reporting systems with ease and high performance. Backward compatibility ensures a smooth transition for existing projects, while the new additions open up vast possibilities for rich user experiences.

---

**Note**: For more details about each scope and how to use it, please refer to the `ProposalModel` behavior documentation file or review the examples provided above.

See [docs/ProposalModel/Docs-ProposalModel-en.md](./docs/ProposalModel/Docs-ProposalModel-en.md)

See [docs/ProposalModel/Docs-ProposalModel-Advenced-Examples-en.md](./docs/ProposalModel/Docs-ProposalModel-Advenced-Examples-en.md)

## 2026-05-19 - 2026-05-20

### Comprehensive Update of Query Scopes in `VisitModel` Behavior

**Complete development of query scopes and helper functions within the `Nano2.Visitors` package**

An integrated development package has been implemented aimed at improving the performance and flexibility of queries related to the visits and views system in NanoSoft applications. Developers no longer need to write complex queries or rely on incorrect usage of `GROUP BY` and `LIMIT` within subqueries; instead, they have a set of ready‑made scopes that enable:

- Sorting by **visit count** (number of records).
- Sorting by **total visits** (field `visits`).
- Sorting by **total views** (field `views`).
- Sorting by **latest visit date** (most recent).
- Sorting by **earliest visit date** (oldest).
- Adding computed columns to `SELECT` (such as `visits_count`, `sum_visits`, `latest_visit_at`) without affecting ordering.
- Filtering objects that are visited (or not visited) by a specific user.
- Filtering objects that have (or do not have) visits, with the ability to specify a minimum count.
- Adding the `is_visited_by_user` column that indicates whether the current user has visited the object.
- Advanced statistical functions to get total visits, total views, visit distribution by user type, and the list of visitor users.

Existing scopes have been restructured and new scopes added, while preserving backward compatibility via wrapper functions marked `@deprecated`. All scopes and helper functions have been moved to a standalone trait `VisitScopesAndHelpers` to facilitate maintenance and reuse.

---

### 1. Developed Components

| Behavior | Component | Description |
|----------|-----------|-------------|
| `VisitModel` | `VisitScopesAndHelpers` (trait) | Contains all advanced scopes and helper functions for the visits and views system. |

#### New and Improved Scopes

| Category | Scope | Description |
|----------|-------|-------------|
| **Visit Count** | `scopeAddCountVisits` | Adds a visit count column to `SELECT` (without sorting). |
| | `scopeSortByCountVisits` | Sorts results by visit count (without adding the column). |
| | `scopeWithCountVisits` | Adds the column and sorts together. |
| **Total Visits** | `scopeAddSumVisits` | Adds a column with the total visits (field `visits`). |
| | `scopeSortBySumVisits` | Sorts by total visits. |
| | `scopeWithSumVisits` | Adds the column and sorts together. |
| **Total Views** | `scopeAddSumViews` | Adds a column with the total views (field `views`). |
| | `scopeSortBySumViews` | Sorts by total views. |
| | `scopeWithSumViews` | Adds the column and sorts together. |
| **Latest Visit Date** | `scopeAddLatestVisit` | Adds a column with the latest visit date. |
| | `scopeSortByLatestVisit` | Sorts by the latest visit date. |
| | `scopeWithLatestVisit` | Adds the column and sorts together. |
| **Earliest Visit Date** | `scopeAddEarliestVisit` | Adds a column with the earliest visit date. |
| | `scopeSortByEarliestVisit` | Sorts by the earliest visit date. |
| | `scopeWithEarliestVisit` | Adds the column and sorts together. |
| **Filtering by User** | `scopeVisitedByUser` | Filters objects visited by a specific user. |
| | `scopeNotVisitedByUser` | Filters objects not visited by a specific user. |
| **Filtering by Visit Presence** | `scopeHasVisits` | Filters objects that have visits (with a minimum count). |
| | `scopeHasNoVisits` | Filters objects that have no visits. |
| **User Status Column** | `scopeWithIsVisitedByUser` | Adds the `is_visited_by_user` column. |
| **Special Scopes** | `scopeTopVisited` | Retrieves the most visited objects (by record count). |
| | `scopeTopSumVisits` | Retrieves the most visited objects (by sum of `visits`). |

#### New Helper Functions

| Function | Description |
|----------|-------------|
| `getTotalVisits` | Total number of visits (sum of `visits`) with filtering options. |
| `getTotalViews` | Total number of views (sum of `views`) with filtering options. |
| `getVisitsCountByType` | Distribution of visits by user type (`user_type`). |
| `getVisitorsUsers` | List of users who visited the object. |

All these functions accept the filtering options: `$onlyActive`, `$withTrashed`, `$type`, `$processType`, `$eventType`.

---

### 2. Details of Code Updates

#### 2.1 Fixing Subquery Errors
The legacy scopes in the original behavior relied on using `GROUP BY` and `LIMIT` inside subqueries to obtain aggregated values (such as `COUNT`, `SUM`, `MAX`). This approach leads to incorrect and unpredictable results, because `GROUP BY` creates multiple groups and then `LIMIT 1` picks only the first group.

**New Approach**:
- Use a direct aggregate function in the subquery without `GROUP BY` or `LIMIT`. An aggregate function in a correlated subquery automatically works on all rows matching the join condition.
- Prefix each field with the full table name to avoid ambiguity.
- Add additional conditions (such as `type`, `processType`, `eventType`) via `where` inside the subquery.

Example of the correct query for sorting by visit count:

```php
$subQuery = Visit::selectRaw('COUNT(' . $table . '.id)')
    ->whereColumn($table . '.visitable_id', $this->model->getTable() . '.id')
    ->where($table . '.visitable_type', $this->model->getMorphClass())
    ->where(...);
```

#### 2.2 Unifying Scope Signatures
Function signatures have been unified to include:
- `$orderDirection`: sort direction (default `DESC`).
- `$columnName`: name of the added column (with a sensible default such as `visits_count`, `sum_visits`, `latest_visit_at`).
- `$onlyActive`: filter by `is_active`.
- `$withTrashed`: include soft‑deleted records.
- `$type`, `$processType`, `$eventType`: additional filtering options by visit type, process type, and event type.
- For date scopes, an additional `$field` parameter has been added to specify the timestamp field to use (`created_at`, `updated_at`, `last_visit_at`, `date_at`).

#### 2.3 Supporting `withTrashed` (Soft‑Deleted Visits)
The `$withTrashed` parameter has been added to all scopes and statistical functions. When `true` is passed, soft‑deleted visit records are included in the subquery. This requires that the `Visit` model uses the `SoftDeletes` trait.

#### 2.4 Moving Scopes to a Separate Trait
To facilitate maintenance and reuse, all scopes and helper functions have been moved to a new trait:  
`Nano2\Visitors\Behaviors\VisitModel\VisitScopesAndHelpers`

This trait is then used inside the `VisitModel` behavior via `use`.

#### 2.5 Backward Compatibility
Legacy scopes have been retained as wrapper functions in the main class, with a `@deprecated` annotation to guide developers toward the new alternatives. The functions `scopeWhereHasVisit` and `scopeWhereHasVisits` have also been modified to preserve the original behavior (when no user is present, they return no results) while adding an `$isForceUser` parameter to control this behavior.

**List of Supported Legacy Functions:**
- `scopeSortByCountVisitsOld` → `scopeSortByCountVisits`
- `scopeWithSortByCountVisits` → `scopeWithCountVisits`
- `scopeSortBySumVisitsOld` → `scopeSortBySumVisits`
- `scopeWithSortBySumVisits` → `scopeWithSumVisits`
- `scopeSortBySumViewsOld` → `scopeSortBySumViews`
- `scopeWithSortBySumViews` → `scopeWithSumViews`
- `scopeWhereHasVisit` → `scopeVisitedByUser` (with enhancements)
- `scopeWhereHasVisits` → `scopeVisitedByUser`

---

### 3. Practical Examples

#### 3.1 Sorting by Visit Count
```php
// Most visited products (by record count)
$products = Product::sortByCountVisits('DESC', 'visits_count')->take(10)->get();

// Add a visit count column
$products = Product::addCountVisits()->get();
foreach ($products as $product) {
    echo $product->visits_count;
}
```

#### 3.2 Sorting by Total Visits (field visits)
```php
$products = Product::sortBySumVisits('DESC', 'sum_visits')->get();
```

#### 3.3 Adding the `is_visited_by_user` Column for the Current User
```php
$products = Product::withIsVisitedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->is_visited_by_user ? 'Visited' : 'Not visited';
}
```

#### 3.4 Filtering Products Visited by the Current User
```php
$visitedProducts = Product::visitedByUser()->get();
```

#### 3.5 Filtering by Visit Type and Process
```php
// Products visited by the current user with type "add" (entry) and event "contest"
$products = Product::visitedByUser(null, null, false, 'add', 'in', 'contest')->get();
```

#### 3.6 Statistics
```php
$product = Product::find(1);
echo "Total visits: " . $product->getTotalVisits(true); // active only
echo "Visitor distribution by type: ";
print_r($product->getVisitsCountByType(true)->toArray());
$visitors = $product->getVisitorsUsers(true);
```

#### 3.7 Combining Advanced Scopes
```php
// Products with more than 5 active visits, sorted by latest visit
$products = Product::hasVisits(5, true)
    ->sortByLatestVisit('DESC', 'latest_visit_at')
    ->get();
```

---

### 4. Added Value

- **For Developers**: A comprehensive set of ready‑made scopes saves time and reduces errors. Developers no longer need to write complex subqueries or worry about result correctness. The unified interface makes learning and usage easier across different behaviors.
- **For End Users**: Ability to present intelligently sorted lists (most visited, most recently active) improves user experience and increases the effectiveness of applications.
- **For the System**: Better performance through optimized SQL queries, eliminating inefficient queries that incorrectly used `GROUP BY` and `LIMIT`.
- **Flexibility**: Filtering by type (`type`), process (`processType`), and event (`eventType`) allows building complex systems such as detailed visit and view analytics.
- **Scalability**: Adding new scopes for any future behavior follows the same pattern, ensuring code consistency and ease of maintenance.

---

### 5. Conclusion

This update represents a qualitative leap in managing visit and view queries within NanoSoft applications. By restructuring scopes and moving them to a standalone trait, while adding advanced sorting, filtering, and statistical functions, developers can now build sophisticated visit tracking systems with ease and high performance. Backward compatibility ensures a smooth transition for existing projects, while the new additions open wide possibilities for rich user experiences.

---

**Note**: For more details about each scope and its usage, please refer to the documentation file for the `VisitModel` behavior or review the examples provided above.

See [docs/VisitModel/Docs-VisitModel-en.md](./docs/VisitModel/Docs-VisitModel-en.md)

See [docs/VisitModel/Docs-VisitModel-Advenced-Examples-en.md](./docs/VisitModel/Docs-VisitModel-Advenced-Examples-en.md)

## 2026-05-20 – 2026-05-21

### Updates to the `Nano.Location` Plugin and the `Nano.LocationApi` Plugin

### Version 1.1.0 – Support for Multimedia in Geographic Models and API

**Version:** 1.1.0 (for `Nano.LocationApi`) and 1.0.16 (for `Nano.Location`)

---

#### Summary of Updates

This release enhances the `Nano.Location` plugin by adding full support for multimedia (images, videos, audio files, documents) to the **Country**, **State**, and **Directorate** models. The update covers both the backend (where fields and columns have been added to administration lists and forms) and the API (where a custom behavior has been added to dynamically include this media in API responses via the location transformers – `CountryTransformer`, `StateTransformer`, `DirectorateTransformer`).

The solution is designed to be **extensible**, **compatible with the `Nano.API` architecture**, and follows the same pattern used in `ProductTransformer` and `DynamicAddIncludeKyc` to ensure consistency and ease of maintenance.

---

### Version 1.1.0 – Update Details

#### Release Objectives

- **Add media relationships** to geographic models (Country, State, Directorate) using `attachOne` and `attachMany`.
- **Extend administration lists** to display main images and image galleries in a visual way.
- **Add file upload fields** to administration forms, supporting images, videos, audio, and general files.
- **Create a new Behavior** `DynamicAddMediaIncludes` inside `Nano.LocationApi` to dynamically add includes to API transformers.
- **Attach this Behavior** to `CountryTransformer`, `StateTransformer`, and `DirectorateTransformer` to enable requesting media via the API using the same `include` mechanism.
- **Support returning optional metadata** (detailed file information) via the `is_mate_data_media` parameter or via per‑include options.
- **Ensure backward compatibility** with existing systems and not break any existing endpoints.

---

#### New Features

##### 1. Support for Multimedia in Geographic Models

The following models have been extended with the following relationships (dynamically via `Plugin.php` without modifying the original model files):

- `RainLab\Location\Models\Country`
- `RainLab\Location\Models\State`
- `Nano\Location\Models\State`
- `Nano\Location\Models\Directorate`

**Added relationships:**

```php
public $attachOne = [
    'image'      => \System\Models\File::class,
    'book_intro' => \System\Models\File::class,
];

public $attachMany = [
    'images' => \System\Models\File::class,
    'videos' => \System\Models\File::class,
    'audios' => \System\Models\File::class,
    'files'  => \System\Models\File::class,
];
```

###### The following table explains each relationship:

| Relationship | Type | Description | Typical Use |
| :--- | :--- | :--- | :--- |
| `image` | `attachOne` | Single image (logo, flag) | Main image for a country or city |
| `images` | `attachMany` | Multiple image gallery | Multiple images for tourist attractions |
| `videos` | `attachMany` | Multiple videos | Introductory videos |
| `audios` | `attachMany` | Multiple audio files | Audio recordings of heritage |
| `files` | `attachMany` | Multiple general files | PDF documents, additional files |
| `book_intro` | `attachOne` | Single introductory file | Single brochure or PDF |

##### 2. Backend Improvements

###### a. Added List Columns

The following columns have been added to the `Country`, `State`, and `Directorate` lists via the `backend.list.extendColumns` event:

| Column | Type | Visible by default | Description |
| :--- | :--- | :--- | :--- |
| `image` | `simpleimage` | Yes | Displays the main image as a thumbnail |
| `images` | `simpleimages` | No | Shows an icon indicating the presence of an image gallery |
| `book_intro` | `partial` (link) | No | Link to download the introductory file |
| `files` | `partial` (link) | No | Link to download the general files |
| `videos` | `partial` (link) | No | Link to download the videos |
| `audios` | `partial` (link) | No | Link to download the audio recordings |

**Example of the `image` column in `columns.yaml` (dynamically):**

```php
'image' => [
    'label' => Lang::get('nano.location::lang.public.models.image'),
    'type' => 'simpleimage',
    'searchable' => false,
    'sortable' => false,
    'invisible' => false,
    'cssClass' => 'nolink',
],
```

###### b. Added Form Fields

A new tab named `Media` (or `الوسائط` depending on language) has been added with the following fields via the `backend.form.extendFields` event:

| Field | Type | Options | Tab |
| :--- | :--- | :--- | :--- |
| `image` | `fileupload` | mode: image, imageWidth: 120, imageHeight: 120, useCaption: true | Media |
| `images` | `fileupload` | mode: image, multiple: true | Media |
| `videos` | `fileupload` | mode: file, fileTypes: mp4,avi,mov,mkv,webm, maxFiles: 5, maxFilesize: 15 | Media |
| `audios` | `fileupload` | mode: file, fileTypes: mp3,wav,wma,m4a,ogg, maxFiles: 5, maxFilesize: 15 | Media |
| `book_intro` | `fileupload` | mode: file, useCaption: true | Media |
| `files` | `fileupload` | mode: file, multiple: true | Media |

**Example code for adding fields in `Plugin.php`:**

```php
$widget->addTabFields([
    'image' => [
        'label' => Lang::get('nano.location::lang.public.models.image'),
        'type' => 'fileupload',
        'mode' => 'image',
        'imageWidth' => 120,
        'imageHeight' => 120,
        'useCaption' => true,
        'span' => 'auto',
        'tab' => $tab,
    ],
    // ... remaining fields
]);
```

##### 3. New Behavior: `DynamicAddMediaIncludes` (in the `Nano.LocationApi` plugin)

A Behavior has been created under the namespace `Nano\LocationApi\Behaviors\DynamicAddMediaIncludes` to provide the functions for media includes in a unified and reusable way. This Behavior follows the same pattern as `DynamicAddIncludeKyc` used in the `Nano3.Kyc` plugin.

###### a. Structure of the Behavior

```php
<?php namespace Nano\LocationApi\Behaviors;

use October\Rain\Extension\ExtensionBase;
use Nano\API\Classes\Transformer;
use League\Fractal\Resource\Primitive;
use League\Fractal\Resource\Collection;

class DynamicAddMediaIncludes extends ExtensionBase
{
    protected Transformer $transformer;
    
    public function __construct(Transformer $transformer)
    {
        $this->transformer = $transformer;
        
        // Add includes to availableIncludes
        $mediaIncludes = ['image', 'images', 'videos', 'audios', 'files', 'book_intro'];
        foreach ($mediaIncludes as $include) {
            if (!in_array($include, $this->transformer->availableIncludes, true)) {
                $this->transformer->availableIncludes[] = $include;
            }
        }
    }
    
    public function includeImage($model) { /* ... */ }
    public function includeImages($model) { /* ... */ }
    public function includeVideos($model) { /* ... */ }
    public function includeAudios($model) { /* ... */ }
    public function includeFiles($model) { /* ... */ }
    public function includeBookIntro($model) { /* ... */ }
}
```

###### b. Main Features of the Behavior:

- **Automatic inclusion of includes**: The Behavior automatically adds `image`, `images`, `videos`, `audios`, `files`, `book_intro` to the `availableIncludes` of the Transformer.
- **Control over metadata**: You can control the return of metadata for each include individually via request parameters:
  - Global: `?is_mate_data_media=1`
  - Specific: `?image[mate_data]=1&files[mate_data]=1`
- **Integration with base Transformer methods**: The Behavior uses the `image()`, `images()`, `file()`, `files()` methods already present in `Nano\API\Classes\Transformer` to ensure a unified format.
- **Safe exception handling**: In case of an error (e.g., the relationship does not exist on the model), default values are returned (`null` for `attachOne`, `[]` for `attachMany`) instead of throwing an exception, ensuring that the API response does not break.

###### c. Detailed Behavior Methods:

| Method | Resource Type | Default value on error | Description |
| :--- | :--- | :--- | :--- |
| `includeImage` | `Primitive` | `null` | Returns main image data |
| `includeImages` | `Collection` | `[]` | Returns a list of gallery image data |
| `includeVideos` | `Collection` | `[]` | Returns a list of video data |
| `includeAudios` | `Collection` | `[]` | Returns a list of audio recording data |
| `includeFiles` | `Collection` | `[]` | Returns a list of general file data |
| `includeBookIntro` | `Primitive` | `null` | Returns introductory file data |

##### 4. Extending the Geographic Transformers

The following Transformers have been extended to include the new Behavior:

- `Nano\LocationApi\Transformers\CountryTransformer`
- `Nano\LocationApi\Transformers\StateTransformer`
- `Nano\LocationApi\Transformers\DirectorateTransformer`

This was done via the `extendTransformerDynamicAddMediaIncludes` function inside `Plugin.php` of `Nano.LocationApi`:

```php
protected function extendTransformerDynamicAddMediaIncludes($className): void
{
    try {
        $className::extend(function ($transformer) {
            if (!$transformer->isClassExtendedWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes')) {
                $transformer->extendClassWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes');
            }
        });
    } catch (\Exception | \Throwable $e) {
        Log::error("Error extending transformer: " . $e->getMessage());
    }
}
```

##### 5. Translation Support (Multi‑language)

New translation keys have been added in the Arabic and English language files inside the `Nano.Location` plugin.

**File:** `plugins/nano/location/lang/ar/lang.php`

```php
'models' => [
    'media_tab' => 'الوسائط',
    'image' => 'الصورة الرئيسية',
    'images' => 'معرض الصور',
    'book_intro' => 'ملف تمهيدي',
    'files' => 'الملفات المرفقة',
    'videos' => 'الفيديوهات',
    'audios' => 'التسجيلات الصوتية',
    'image_comment' => 'اختر صورة واحدة',
    'images_comment' => 'اختر عدة صور',
    'book_intro_comment' => 'رفع ملف PDF أو مستند',
    'files_comment' => 'رفع ملفات متنوعة',
    'videos_comment' => 'رفع فيديوهات (mp4, avi, mov, mpg, mpeg, mkv, webm)',
    'audios_comment' => 'رفع ملفات صوتية (mp3, wav, wma, m4a, ogg)',
],
```

**File:** `plugins/nano/location/lang/en/lang.php`

```php
'models' => [
    'media_tab' => 'Media',
    'image' => 'Main Image',
    'images' => 'Gallery Images',
    'book_intro' => 'Introductory File',
    'files' => 'Attached Files',
    'videos' => 'Videos',
    'audios' => 'Audio Recordings',
    'image_comment' => 'Choose a single image',
    'images_comment' => 'Choose multiple images',
    'book_intro_comment' => 'Upload a PDF or document',
    'files_comment' => 'Upload various files',
    'videos_comment' => 'Upload videos (mp4, avi, mov, mpg, mpeg, mkv, webm)',
    'audios_comment' => 'Upload audio files (mp3, wav, wma, m4a, ogg)',
],
```

---

#### Practical Examples

##### 1. Using the API to Retrieve a Country with its Images and Videos

**Request:**
```http
GET /api/v1/countries/1?include=image,images,videos&is_mate_data_media=1
```

**Response (JSON):**
```json
{
  "data": {
    "id": 1,
    "name": "Yemen",
    "code": "YE",
    "image": {
      "original": "https://domain.com/storage/app/media/flags/ye.png",
      "small": "https://domain.com/storage/app/media/flags/ye_small.png",
      "thumb": "https://domain.com/storage/app/media/flags/ye_thumb.png",
      "mate_data": {
        "id": 101,
        "title": "Flag of Yemen",
        "description": "Official flag",
        "file_name": "ye.png",
        "file_size": 45678,
        "extension": "png",
        "mime_type": "image/png"
      }
    },
    "images": [
      {
        "original": "https://domain.com/storage/app/media/gallery/sanaa.jpg",
        "small": "https://domain.com/storage/app/media/gallery/sanaa_small.jpg",
        "mate_data": { ... }
      }
    ],
    "videos": [
      {
        "file_name": "yemen_intro.mp4",
        "path": "https://domain.com/storage/app/media/videos/yemen_intro.mp4",
        "mate_data": { ... }
      }
    ]
  }
}
```

##### 2. Requesting a City with only the Introductory File Included

**Request:**
```http
GET /api/v1/states/5?include=book_intro
```

**Response (excerpt):**
```json
{
  "data": {
    "id": 5,
    "name": "Sana'a",
    "country_id": 1,
    "book_intro": {
      "file_name": "sanaa_guide.pdf",
      "path": "https://domain.com/storage/app/media/guides/sanaa_guide.pdf",
      "file_size": 2048000
    }
  }
}
```

##### 3. Requesting a Directorate with Images and Files and Custom Metadata per Include

**Request:**
```http
GET /api/v1/directorates/10?include=images,files&images[mate_data]=1&files[mate_data]=0
```

In this example, metadata will be returned for images only, not for files.

##### 4. Backend: Uploading a Main Image for a Country

1. Go to `Geographic Location > Countries`.
2. Choose a country from the list or create a new one.
3. In the `Media` tab, upload an image in the `Main Image` field.
4. Save the changes.
5. The thumbnail will appear in the countries list.

##### 5. Using the Behavior in a Custom Transformer

If you have a custom transformer and want to add media support, you can simply attach the Behavior:

```php
use Nano\LocationApi\Behaviors\DynamicAddMediaIncludes;

class MyCustomTransformer extends Transformer
{
    public $implement = [
        DynamicAddMediaIncludes::class,
    ];
    
    // ... rest of the code
}
```

---

#### Improvements and Fixes

| Area | Improvement |
| :--- | :--- |
| **Exception handling** | If a media relationship does not exist on the model (e.g., `image` is not defined), `null` or `[]` is returned instead of throwing an error. |
| **Partial metadata support** | Metadata can be requested per include (`image[mate_data]=1`) or globally (`is_mate_data_media=1`). |
| **Full backward compatibility** | All existing transformers are unaffected; media is only included if explicitly requested (`include=image,...`). |
| **Improved caching performance** | Media is treated like any other relationship in Fractal, allowing smart caching. |
| **Easy extensibility** | New media relationships can be added in the future simply by adding the appropriate methods to the Behavior and adding them to `availableIncludes`. |
| **Compatibility with `System\Models\File`** | Uses the standard October CMS file system, ensuring full compatibility with all features of `System\Models\File` (e.g., thumbnails, access control, etc.). |

---

#### Upgrade Requirements

##### 1. Update the `Nano.Location` Plugin Code

- **Replace** the file `plugins/nano/location/Plugin.php` with the new version (which contains the functions `extendCountryWithMedia`, `extendRainLabStateWithMedia`, etc., and calls them in `boot()`).
- **Add** the new translation keys to `plugins/nano/location/lang/ar/lang.php` and `plugins/nano/location/lang/en/lang.php`.
- **Create** the partial files for displaying links in the columns (optional, as non‑`image` columns are hidden by default). If you wish to use them, create the folder `plugins/nano/location/models/country/` and add the following files:
  - `_link_book_intro.htm`
  - `_link_files.htm`
  - `_link_videos.htm`
  - `_link_audios.htm`

**Example content for `_link_book_intro.htm`:**
```twig
{% if model.book_intro %}
    <a href="{{ model.book_intro.path }}" target="_blank">Download</a>
{% endif %}
```

##### 2. Update the `Nano.LocationApi` Plugin Code

- **Replace** the file `plugins/nano/locationapi/Plugin.php` with the new version (which contains the function `extendTransformerDynamicAddMediaIncludes` and calls it in `boot()`).
- **Add** the `behaviors` folder in `plugins/nano/locationapi/` and place the file `DynamicAddMediaIncludes.php` inside it.
- **Update** the `version.yaml` file of `Nano.LocationApi` to add version `1.1.0` as shown in the updates section.

##### 3. Update `version.yaml` Files

**For `Nano.Location` (`plugins/nano/location/updates/version.yaml`):**
```yaml
1.0.16:
    - 'Add media relations (image, images, videos, audios, files, book_intro) to Country, State, and Directorate models'
    - 'Add media fields and list columns to backend forms and lists for Country, State, Directorate'
    - 'Add translations for media fields (ar/en)'
```

**For `Nano.LocationApi` (`plugins/nano/locationapi/updates/version.yaml`):**
```yaml
1.1.0:
    - 'Add DynamicAddMediaIncludes behavior for CountryTransformer, StateTransformer, DirectorateTransformer'
    - 'Support dynamic media includes (image, images, videos, audios, files, book_intro) in Location API'
    - 'Add is_mate_data_media parameter to control returning file metadata in API responses'
    - 'Extend availableIncludes automatically for media relations'
```

##### 4. Run Update Commands

```bash
# Clear cache
php artisan cache:clear
php artisan config:clear
```

##### 5. Verify File Upload Permissions

Make sure the `storage/app/uploads` folder is writable by the web server:

```bash
chmod -R 775 storage/app/uploads
```

##### 6. Test the Functionality

- **Test the backend:** Go to `Geographic Location > Countries`, try uploading an image for a country and verify that it appears in the list.
- **Test the API:** Use `curl` or Postman to test the following endpoints:

```bash
# Fetch a country with its main image
curl -X GET "https://yourdomain.com/api/v1/countries/1?include=image"

# Fetch a city with its image gallery and files
curl -X GET "https://yourdomain.com/api/v1/states/5?include=images,files&is_mate_data_media=1"
```

---

#### Backward Compatibility

- **No breaking changes** to the database structure – only `attachOne` and `attachMany` relationships have been added dynamically.
- **Media is not automatically included** in existing API responses, so existing applications that use the API will not be affected.
- **All existing API methods** work unchanged.
- **The existing backend** has not been altered; only a new tab has been added without removing any existing fields.

---

#### Conclusion

Version 1.1.0 of the `Nano.Location` and `Nano.LocationApi` plugins represents a major step forward in multimedia support for geographic models. Thanks to the modular behavior‑based architecture, it is now easy to add and manage images, videos, and files for countries, states, and directorates through an intuitive administration interface, and to retrieve them via the API in the same way as any other relationship.

---

**Reference documentation:**
- [Nano.Location Plugin Documentation](./docs/Location/Docs-Nano-Location-en.md)
- [DynamicAddMediaIncludes Behavior Documentation](./docs/Location/Docs-DynamicAddMediaIncludes-en.md)
- [Location API Documentation](./docs/Location/Docs-LocationAPI-en.md)


## 2026-05-20 – 2026-05-22

**CashPay Payment Gateway Update (Electronic Cash Payment – OTP) in the NanoSoft Payment System**


### Addition of CashPay Payment Gateway (Electronic Cash Payment – OTP)

Within the `Nano.Yepayment` software module

---

## 1. Introduction

A new payment method named **CashPay** has been developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition responds to the need to support electronic payment gateways that rely on payment confirmation via OTP (One‑Time Password), as **CashPay** is a payment platform that enables accepting payments through a direct API with AES‑256‑CBC password encryption and a secure exchange mechanism.

CashPay operates according to the **Two‑Step Payment using OTP** pattern, which matches the platform's nature where:
1. **First step (`process`):** A transaction (InitPayment) is created and a `TransactionRef` is obtained, without deducting the amount yet. At this stage, the customer is asked to enter the OTP that will be sent later.
2. **Second step (`complete`):** After the customer enters the OTP, a payment confirmation request (ConfirmPayment) is sent to CashPay. If the code is correct, the amount is deducted and the order is updated to "paid".

This design makes the gateway fully compatible with `OrderManager`, `Checkout2`, and `PaymentResult`, and prevents the order from being marked as paid before the correct OTP is entered.

The gateway has been developed according to the highest security and quality standards, using the unified `HttpHelper` for all API requests, encrypting the password in the header using AES‑256‑CBC, and providing integrated testing interfaces and dedicated API endpoints for developers and administrators. Support for `RedirectHelper` has also been added to ensure compatibility with mobile applications that need return URLs after payment.

---

## 2. Developed Components

### 2.1. `CashPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\CashPay`
- **Inheritance:** Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication via encrypted header (`encPassword`), executing the first step (InitPayment), the second step (ConfirmPayment), querying transaction status (OperationStatus), and changing the password (ChangePass).

**Main Methods:**

| Method | Description |
|--------|-------|
| `process(PaymentResult $result)` | **First step:** Checks for an existing transaction, creates a new transaction via `InitPayment`, receives `TransactionRef`, saves it in the order without changing payment status. Returns success with a message "Enter OTP". |
| `complete(PaymentResult $result)` | **Second step:** Verifies `TransactionRef` and `OTP`, sends a `ConfirmPayment` request, and on success calls `$result->success()` to update the order to `PaidState`. |
| `operationStatus(string $requestId, string $type): array` | Queries transaction status using `RequestID` (type `InitOP` for initial transaction, `PayOP` for confirmation). |
| `changePassword(string $newPassword): array` | Changes the merchant's password with CashPay (when expired or needs updating). |
| `getAuthToken(): ?string` | (Not used in CashPay because authentication relies on `encPassword` in the header. Present for compatibility.) |
| `initPayment(): array` | Builds the `InitPayment` request and sends it to `/api/CashPay/InitPayment`, returning `TransactionRef`. |
| `confirmPayment(string $transactionRef, string $otp): array` | Builds the `ConfirmPayment` request with `TRCode = md5(TransactionRef + OTP)` and sends it to `/api/CashPay/ConfirmPayment`. |
| `encryptAes256Cbc(string $data, string $key): string` | Encrypts data (password) using AES‑256‑CBC to obtain `encPassword`. |
| `getEncryptedPassword(): string` | Encrypts the stored password using the encryption key from settings. |
| `buildHeaders(): array` | Builds request headers (`encPassword`, `unixtimestamp`, `Content-Type`). |
| `settings(): array` | Defines the settings fields in the control panel (URL, Username, Password, Encryption Key, SpId, default currency). |
| `encryptedSettings(): array` | Specifies fields that are stored encrypted (`cashpay_password`, `cashpay_encryption_key`). |
| `getSupportedCurrencies(): array` | List of supported currencies with their numeric IDs (YER=2, USD=1, SAR=3). |
| `getCurrencyId($currency): ?int` | Converts a currency code or number to the correct supported numeric ID. |
| `isCurrencySupported($currency): bool` | Checks whether a given currency is supported. |

### 2.2. Partials and Settings Files

| File | Path | Description |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/cashpay/_info.htm` | Displays setup instructions and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/cashpay/_test_info.htm` | Quick testing tools within the settings page (buttons for authentication, create transaction, confirm, query, change password, full flow). |
| `cashpay-ui.htm` | `views/cashpay-ui.htm` | Complete interactive web interface for testing all CashPay functions (manual, automated, statistics, logs). |

### 2.3. API Endpoints (`routes.php`)

A complete set of routes has been added within the `yepayment` group to support testing and monitoring:

| Path | Method | Description |
|--------|---------|-------|
| `/cashpay/test-auth` | POST | Test settings validity (Username, Password, Encryption Key, SpId). |
| `/cashpay/test-create-payment` | POST | Create a new transaction (InitPayment). |
| `/cashpay/test-confirm-payment` | POST | Confirm payment (ConfirmPayment) using OTP. |
| `/cashpay/test-check-status` | POST | Query transaction status (OperationStatus). |
| `/cashpay/test-change-password` | POST | Change password (ChangePass). |
| `/cashpay/test-full-payment` | POST | Full flow test (InitPayment + ConfirmPayment). |
| `/cashpay/stats` | GET | Gateway usage statistics (number of orders, success rate). |
| `/cashpay/test-ui` | GET | Integrated testing interface (HTML). |
| `/cashpay/success` | GET | Endpoint for redirection on payment success (uses `RedirectHelper`). |
| `/cashpay/cancel` | GET | Endpoint for redirection on cancellation. |

All routes (except success/cancel) are protected by `BackendAuth::getUser()` to ensure only administrators can access them.

### 2.4. `Plugin.php` – Payment Provider Registration

The following line was added to the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\CashPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\CashPay();
}
```

### 2.5. Translation Files

Translation keys were added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, general messages, and errors (e.g., `payment_success`, `auth_success`, `confirmation_required`, `order_already_paid`).

---

## 3. Payment Workflow – Two‑Step (OTP) Pattern

CashPay operates according to a **Two‑Step (OTP)** flow:

1. **Authentication**  
   There is no separate token session. Every request is sent with an `encPassword` header, which is the merchant's password encrypted with AES‑256‑CBC using the provided encryption key.  
   A `unixtimestamp` (in milliseconds) is also used to prevent replay attacks.

2. **First step – Create Transaction (`process`)**  
   - Ensure the order is not already paid.
   - Check for an existing transaction associated with the order via `payment_first_trans_id` or `other_data['cashpay']['transaction_ref']`:
     - If found, call `operationStatus()` to query its status.
     - If status is `success` → calls `$result->success()` (order already paid).
     - If status is `pending` → returns a message "Enter OTP" without creating a new transaction.
   - **No existing transaction or invalid status:**  
     - Build the request body (`RequestID`, `UserName`, `SpId`, `MDToken`, `TargetMSISDN`, `CustomerCashPayCode`, `Amount`, `CurrencyId`, `Desc`).
     - Send a `POST /api/CashPay/InitPayment` request.
     - Receive `TransactionRef` and store it in `order.payment_first_trans_id` and `order.payment_trans_id`.
     - Store additional data in `order.other_data['cashpay']` (amount, currency, mobile number, payment code, creation date, callback URLs).
     - **Do not call `$result->success()`**, and do not change the order to `PaidState`. Instead, return `successful = true` with a `confirmation_required` message.
     - Log the payment attempt via `$result->logSuccessfulPayment()`.

3. **Second step – Confirm Payment (`complete`)**  
   - Called after the customer enters the OTP (passed in `$this->data['otp']`).
   - Extract `TransactionRef` from the order.
   - Compute `TRCode = md5(TransactionRef . OTP)`.
   - Send a `POST /api/CashPay/ConfirmPayment` request with `TransactionRef` and `TRCode`.
   - If `ResultCode == 1`:
     - Call `$result->success()`, which **changes the order status to `PaidState`**, triggers events (e.g., `nano.orders.paymentProcessed`), and clears the cart.
   - If `ResultCode == 6022`: return an error "Password must be changed".
   - Any other code is considered a failure.

4. **Optional Callback URLs**  
   When `callback_success_url` or `callback_error_url` are passed in the payment data, they are stored in `other_data`. After confirmation, the developer can use `RedirectHelper` to intelligently redirect (deep link or web).

---

## 4. Configuration

To activate CashPay, the following settings must be entered in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|---------|-------|-------------------|
| Base API URL | `cashpay_url` | Production API endpoint | `https://api.cash-pay.com` |
| Test API URL | `cashpay_test_url` | UAT API endpoint (optional) | `https://test-api.cash-pay.com` |
| Username | `cashpay_username` | Merchant username | - |
| Password | `cashpay_password` | Merchant password (stored encrypted) | - |
| Encryption Key (AES‑256‑CBC) | `cashpay_encryption_key` | Key used to encrypt the password in the header | - |
| Client Code (SpId) | `cashpay_sp_id` | Service Provider ID provided by CashPay | - |
| Default Currency | `cashpay_default_currency_id` | Default currency (shown as dropdown) | `YER` |

**Security Notes:**
- `cashpay_password` and `cashpay_encryption_key` are stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

## 5. Usage Examples

### 5.1. Starting Payment – First Step (InitPayment)

```php
use Nano\Yepayment\PaymentTypes\CashPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$cash = new CashPay($order, [
    'target_msisdn' => '771234567',
    'customer_cash_pay_code' => 555,
    'amount' => 100,
    'currency' => 'YER',
    'desc' => 'Payment for order #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($cash, $order);
$processResult = $cash->process($result);

if ($processResult->successful) {
    // Transaction created and TransactionRef saved, system awaits OTP input
    $transactionRef = $order->payment_first_trans_id;
    // Show OTP input field to the user
}
```

### 5.2. Confirming Payment – Second Step (ConfirmPayment)

After the user enters the OTP, you can call `complete()` directly:

```php
$cash = new CashPay($order, ['otp' => '1234']);
$result = new PaymentResult($cash, $order);
$completeResult = $cash->complete($result);

if ($completeResult->successful) {
    // Payment confirmed and order updated to PaidState
}
```

Or using a static helper function (via the `success` route):

```php
$result = \Nano\Yepayment\PaymentTypes\CashPay::checkAndCompletePay(['order_id' => 200, 'otp' => '1234']);
if ($result['success']) {
    // Payment successful
}
```

### 5.3. Direct Transaction Status Query

```php
$cash = new CashPay();
$status = $cash->operationStatus('1234567890', 'InitOP');
if ($status['success']) {
    echo "Status: " . $status['status']; // success, pending, failed
}
```

### 5.4. Changing Password (ChangePass)

When receiving error code 6022, you must change the password:

```php
$cash = new CashPay();
$result = $cash->changePassword('NewStrongPass123');
if ($result['success']) {
    // Password changed successfully, also update the settings in the control panel
}
```

### 5.5. Using the Integrated Testing Interface

After logging into the control panel as an administrator, open the URL:
```
https://yourdomain.com/api/v1/yepayment/cashpay/test-ui
```

The interface allows:
- Test authentication (verify settings validity).
- Create transaction (InitPayment) and display `TransactionRef`.
- Confirm payment (ConfirmPayment) using OTP.
- Query status (OperationStatus) using `RequestID`.
- Change password (ChangePass).
- Full flow (create + confirm) in one step.
- Automated testing with a configurable number of attempts (up to 5 times).
- Usage statistics (number of orders, success rate).
- Local test logs.

---

## 6. Handling Error Codes and Response Codes

| Code | Meaning | Appropriate Action |
|------|--------|------------------|
| 1 | Success | Continue normal flow. |
| 35 | Invalid input data | Check the fields sent. |
| 6022 | Password expired or invalid | Call `changePassword()` to change the password. |
| 6025 | Customer balance insufficient | Inform customer to top up balance. |
| 9999 | General internal error | Retry or contact support. |

> **Note:** Any other error code is displayed as received from CashPay via `ResultMessage`.

---

## 7. Integration Testing

Several testing layers are provided:

- **Test environment:** Use the UAT URL provided by CashPay (set in the `cashpay_test_url` field).
- **Quick test interface:** From the gateway settings page (partial file `_test_info.htm`), you can test authentication, create transaction, confirm, query, and change password.
- **Integrated testing interface:** `/api/v1/yepayment/cashpay/test-ui` provides all necessary tools to fully test the gateway (manual, automated, statistics).
- **Independent API endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints such as `/cashpay/test-create-payment`.

**Quick test steps:**
1. Ensure Username, Password, Encryption Key, and SpId are set in the gateway settings page.
2. Open `/api/v1/yepayment/cashpay/test-ui`.
3. Click "Test authentication" to verify credentials.
4. Enter an order number, mobile number, payment code, amount, then click "Create transaction".
5. `TransactionRef` will appear – enter the correct OTP (sent to the mobile) in the OTP field and click "Confirm payment".
6. The order status will change to paid.

---

## 8. Developer Notes

- **Two‑Step pattern:** `process()` **does not** complete the payment; it only creates the transaction. You must call `complete()` with the correct OTP to confirm the payment and update the order to `PaidState`.
- **Password encryption:** The password in the request header is encrypted using `encryptAes256Cbc()` with the dedicated encryption key. Make sure the key matches the one provided by CashPay.
- **Password change:** When error code `6022` appears, you should call `changePassword()` immediately (you can show a message to the administrator or direct them to change the password in the control panel).
- **Existing transaction check:** In `process()`, the system first checks for an existing transaction associated with the order. If its status is `success`, the order is completed immediately, preventing duplicate transactions.
- **Using `HttpHelper`:** All API requests use the unified `HttpHelper`, making it easier to track errors and extend the gateway.
- **Currency support:** Functions `getSupportedCurrencies()`, `getCurrencyId()`, and `isCurrencySupported()` are included to check supported currencies and convert codes to the correct numeric IDs.
- **Support for `RedirectHelper`:** Full redirection capability has been included, making the gateway suitable for mobile applications that use Deep Links.
- **Extensibility:** Support for webhooks or refunds can be added in the future via additional methods using different CashPay endpoints.

---

## 9. Bug Fixes

None – this release is dedicated to adding a new feature (CashPay).

---

## 10. Development and Testing Period

The CashPay gateway was developed during the period from **May 15, 2026** to **May 20, 2026**. This period included:
- Writing the core code for the `CashPay` class (extending `PaymentProvider`).
- Implementing `process()`, `complete()`, `initPayment()`, `confirmPayment()`, `operationStatus()`, and `changePassword()` methods.
- Adding the existing transaction check in `process()` to prevent duplication.
- Implementing the password encryption mechanism using AES‑256‑CBC.
- Creating partial view files (`_info.htm`, `_test_info.htm`) and the integrated testing interface (`cashpay-ui.htm`).
- Writing the necessary API endpoints for testing and monitoring in `routes.php` (10 endpoints).
- Adding translation keys in language files.
- Registering the gateway in `Plugin.php`.
- Conducting comprehensive integration tests to verify functionality with the CashPay environment (simulated using Postman and UI).

---

## 11. Relevant Links

- [CashPay Documentation (Developer Guide)](./docs/CashPay/Docs-CashPay-en.md)
- [SKILL.md for Creating Payment Gateways](./docs/CashPay/SKILL-en.md)
- [Cash-Pay API Documentation](./docs/CashPay/external/Cash-Pay%20API%20Doc2.pdf)
- [CashPay.php Class](./Nano/Yepayment/PaymentTypes/CashPay.php)
- [routes.php file of Nano.Yepayment](./routes.php)
- [Technical Support Channel](https://nano2soft.com)

---

**This update prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**References:** Dheia Ali, Nano2Soft

## 2026-05-18 – 2026-05-24

### Addition and Update of FloosakPay Payment Gateway (Floosak Wallet)

---

## 1. Introduction

A new payment method named **FloosakPay** has been developed and added to the unified payment system in NanoSoft within the `Nano.Yepayment` module. This addition responds to the need to support digital wallet payment gateways in Yemen, where **Floosak Wallet** is one of the leading digital financial solutions that allows users to make electronic payments easily and securely.

FloosakPay relies on the **Floosak Wallet API** (v1), which provides:
- Merchant authentication via `/api/v1/auth/login` to obtain a `Bearer token`.
- Create a pending purchase via `/api/v1/merchant/p2mcl`.
- Confirm payment using an OTP via `/api/v1/merchant/p2mcl/confirm`.
- Query transaction status via `/api/v1/merchant/check-status`.
- Refund an amount via `/api/v1/merchant/p2mcl/refund`.

The gateway is designed according to the **Two‑Step payment pattern with OTP confirmation**, where:
1. **First step (`process`):** A pending transaction (Send) is created, an OTP is sent to the customer’s mobile, and transaction data (`purchase_id`) is returned. **The order is not changed to paid.**
2. **Second step (`complete`):** After the customer enters the code, the payment is confirmed using the OTP. If the code is correct, the order is updated to `PaidState` and the cart is cleared.

This design ensures full compatibility with `OrderManager`, `Checkout2`, and `PaymentResult`, and prevents the order from being marked as paid before the correct OTP is confirmed.

The gateway has been developed according to the highest security and quality standards, with tokens stored in Cache, using the unified `HttpHelper` for all API requests, and providing integrated testing interfaces and dedicated API endpoints for developers and administrators. Support for `RedirectHelper` has also been added to ensure compatibility with mobile applications that need return URLs after payment, along with an **idempotency mechanism** via `request_id` to prevent duplicate transactions, and logging of `PaymentLog` in the first stage to track incomplete attempts.

---

## 2. Developed Components

### 2.1. `FloosakPay` – Main Payment Gateway Class

- **Path:** `Nano\Yepayment\PaymentTypes\FloosakPay`
- **Inheritance:** Extends `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **Function:** Responsible for authentication via `/api/v1/auth/login`, creating a pending transaction (Send), confirming payment (Confirm), querying status, and refunding.

**Main Methods:**

| Method | Description |
|--------|-------|
| `process(PaymentResult $result)` | **First step:** Validates input (`target_phone`, `amount`, `purpose`), checks for an existing transaction (idempotency), obtains a token, sends Send request, saves `request_id` and `purchase_id` in the order, logs `PaymentLog` as `initiated`, then returns `successful = true` with a message asking for OTP (**does not call `$result->success()`**). |
| `complete(PaymentResult $result)` | **Second step:** Extracts `purchase_id`, obtains a new token, sends Confirm request with OTP. If successful and status is `Completed`, calls `$result->success()` to update the order to `PaidState`. |
| `reverse(string $refNo, string $reason = ''): array` | Refunds a completed transaction via `/api/v1/merchant/p2mcl/refund`. On success, changes order status to `RefundedState` and stores refund data in `other_data`. |
| `checkTransactionStatus(string $requestId): array` | Queries transaction status using `request_id` via `/api/v1/merchant/check-status`. Used in idempotency and testing tools. |
| `getTransactionStatusForOrder(string $orderId): array` | Helper function to query status using `order_id` directly. |
| `getAuthToken(bool $useCache = true): ?string` | Obtains a `Bearer token` via `POST /api/v1/auth/login` (with `x-channel: merchant`). Token is cached for 3500 seconds. |
| `sendPayment(string $token): array` | Sends `POST /api/v1/merchant/p2mcl` to create a pending transaction. Generates `request_id` as UUID. Returns `purchase_id`, `reference_id`, `fee`, `gross`. |
| `confirmPayment(string $token, int $purchaseId, string $otp): array` | Sends `POST /api/v1/merchant/p2mcl/confirm` to confirm payment. Checks that `status.en == "Completed"`. |
| `refundPayment(string $token, int $transactionId, string $requestId, float $amount, string $reason): array` | Sends `POST /api/v1/merchant/p2mcl/refund`. |
| `settings(): array` | Defines settings fields in the control panel (URL, phone, password, source_wallet_id, default_purpose). |
| `encryptedSettings(): array` | Specifies fields stored encrypted (`floosakpay_password`). |
| `defineValidationRules(): array` | Validation rules for `target_phone`, `amount`, `purpose`. |
| `handleSpecificErrors(array $responseData, string $defaultMessage): array` | Extracts error messages from Floosak response (supports `errors` and `message`). |

### 2.2. Idempotency Mechanism (Preventing Duplicate Transactions)

- In `process()`, a search is performed for an existing transaction via `payment_first_trans_id` or `other_data['floosakpay']['request_id']`.
- If found, `checkTransactionStatus()` is called:
  - If status is `Completed` → calls `$result->success()` directly (order already paid).
  - If status is `Pending` → informs the user to wait for OTP without creating a new transaction.
  - If status is unknown or failed → creates a new transaction.
- This ensures no duplicate transactions are created when the user retries the payment process.

### 2.3. Logging `PaymentLog` in the First Stage

- After successfully creating the transaction, a `PaymentLog` is manually logged:
  ```php
  $paymentLog = $result->logSuccessfulPayment($sendResult, null, 'initiated');
  $this->order->payment_id = $paymentLog->id;
  $this->order->save();
  ```
- This allows tracking of payment attempts even if the customer never completes the OTP, helping to analyse completion rates.

### 2.4. Partials and Settings Files

| File | Path | Description |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/floosakpay/_info.htm` | Displays setup instructions and gateway information in the control panel. |
| `_test_info.htm` | `paymenttypes/floosakpay/_test_info.htm` | Quick testing tools within the settings page (authentication, create, confirm, query, refund buttons). |
| `floosakpay-ui.htm` | `views/floosakpay-ui.htm` | Complete interactive web interface for testing all FloosakPay functions (manual, automated, statistics, logs). |

### 2.5. API Endpoints (`routes.php`)

A complete set of routes has been added within the `yepayment` group to support testing and monitoring:

| Path | Method | Description |
|--------|---------|-------|
| `/floosakpay/test-auth` | POST | Test authentication and obtain Bearer Token. |
| `/floosakpay/test-create-payment` | POST | Create a pending transaction (simulates `process`). |
| `/floosakpay/test-confirm-payment` | POST | Confirm payment using OTP (simulates `complete`). |
| `/floosakpay/test-check-status` | GET | Query transaction status using `request_id` or `order_id`. |
| `/floosakpay/test-refund` | POST | Refund a completed transaction. |
| `/floosakpay/test-full-payment` | POST | Full test (create + confirm + query) in one step. |
| `/floosakpay/stats` | GET | Gateway usage statistics (number of orders, success rate, recent logs). |
| `/floosakpay/logs` | GET | Transaction logs from the database (for admin). |
| `/floosakpay/test-ui` | GET | Integrated testing interface (HTML). |
| `/floosakpay/success` | GET | Endpoint to confirm payment after OTP entry (uses `checkAndCompletePay`). |
| `/floosakpay/cancel` | GET | Optional endpoint for cancellation redirection. |

All routes (except success/cancel) are protected by `BackendAuth::getUser()` to ensure only administrators can access them.

### 2.6. `Plugin.php` – Payment Provider Registration

The following line was added to the `registerPaymentProviders()` function within the `allow_yemen_payment` section:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\FloosakPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\FloosakPay();
}
```

### 2.7. Translation Files

Translation keys were added in `lang/ar/lang.php` and `lang/en/lang.php` for settings, general messages, and errors (e.g., `auth_success`, `confirmation_required`, `payment_success`, `order_already_paid`, `auth_failed`, `missing_credentials`, `missing_wallet_id`).

---

## 3. Payment Workflow – Two‑Step (OTP) Pattern

FloosakPay operates according to a **Two‑Step** flow with OTP confirmation:

### 3.1. Authentication

`getAuthToken()` is called:
- Checks for an `access_token` in Cache (validity 3500 seconds).
- If not present, sends a `POST /api/v1/auth/login` request with `phone`, `password`, and `x-channel: merchant`.
- The token is stored in Cache for slightly less than its actual lifetime.

### 3.2. First Step – Create Pending Transaction (`process`)

- Validate data (`target_phone`, `amount`, `purpose`) using `Validator`.
- Ensure the order is not already paid (`payment_state != PaidState`).
- **Check for an existing transaction (Idempotency):**
  - Search for `request_id` in `payment_first_trans_id` or `other_data['floosakpay']['request_id']`.
  - If found, call `checkTransactionStatus()`:
    - If `Completed` → call `$result->success()` (order already paid).
    - If `Pending` → return a message asking for OTP without creating a new transaction.
    - Otherwise → continue with creation.
- **No existing transaction or invalid status:**
  - Obtain `access_token`.
  - Send `POST /api/v1/merchant/p2mcl` with:
    - `source_wallet_id` (from settings)
    - `request_id` (unique UUID)
    - `target_phone`, `amount`, `purpose`
  - Receive `purchase_id`, `reference_id`, `fee`, `gross`.
  - Save `request_id` in `order.payment_first_trans_id`.
  - Save `purchase_id` in `order.payment_trans_id`.
  - Store additional data in `order.other_data['floosakpay']` (phone, amount, purpose, return URLs, etc.).
  - **Do not call `$result->success()`**, and do not change order to `PaidState`.
  - Log a `PaymentLog` with status `'initiated'`.
  - Return `$result->successful = true` with a message "Please enter the verification code".

### 3.3. Second Step – Confirm Payment (`complete`)

- Called from the `floosakpay/success` route after the customer enters the OTP.
- Extract `purchase_id` from `order.payment_trans_id`.
- Obtain a fresh `access_token` (may have expired).
- Send `POST /api/v1/merchant/p2mcl/confirm` with `otp` and `purchase_id`.
- Parse the response:
  - If `is_success == true` and `status.en == "Completed"`:
    - Call `$result->success()` which:
      - Changes `payment_state` to `PaidState`.
      - Sets `processed = true`.
      - Notifies the system of associated events (sends email, updates inventory, clears cart).
  - If `status.en == "Pending"` → return a message that the code is incorrect or the transaction is still pending.
- Any other status is considered a failure.

### 3.4. Optional Callback URLs

When `callback_success_url` or `callback_error_url` are passed in the payment data, they are stored in `other_data`. After confirmation, the developer can use `RedirectHelper` in the `success` route to intelligently redirect the user (deep link or web) while passing the payment result.

---

## 4. Configuration

To activate FloosakPay, the following settings must be entered in the payment gateway settings interface in the NanoSoft system (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| Setting | Key | Description | Default Value |
|---------|---------|-------|-------------------|
| Base API URL | `floosakpay_url` | Production API endpoint | `https://staging.fintech-expert.net` |
| Test API URL | `floosakpay_test_url` | Test environment API endpoint (optional) | `https://staging.fintech-expert.net` |
| Merchant Phone Number | `floosakpay_phone` | Mobile number used for authentication (with country code) | - |
| Password | `floosakpay_password` | Merchant account password (stored encrypted) | - |
| Source Wallet ID | `floosakpay_source_wallet_id` | ID of the wallet from which funds will be deducted | - |
| Default Purpose | `floosakpay_default_purpose` | Text sent as transaction description | `دفع الطلب` (Payment for order) |

**Security Notes:**
- `floosakpay_password` is stored encrypted via `encryptedSettings()`.
- These sensitive values are not logged.

---

## 5. Usage Examples

### 5.1. Starting Payment – First Step (Create Pending Transaction)

```php
use Nano\Yepayment\PaymentTypes\FloosakPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$floosak = new FloosakPay($order, [
    'target_phone' => '967771234567',
    'amount'       => 100,
    'purpose'      => 'دفع الطلب #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($floosak, $order);
$processResult = $floosak->process($result);

if ($processResult->successful && !$processResult->redirect) {
    // Show OTP input interface to the user
    $purchaseId = $order->payment_trans_id;
    $requestId  = $order->payment_first_trans_id;
}
```

### 5.2. Confirming Payment – Second Step (Enter OTP)

After the customer enters the code, the process can be confirmed as follows:

```php
$floosak = new FloosakPay($order, ['otp' => '123456']);
$result = new PaymentResult($floosak, $order);
$completeResult = $floosak->complete($result);

if ($completeResult->successful) {
    // Payment confirmed and order updated to PaidState
}
```

Or using the static function `checkAndCompletePay`:

```php
$result = \Nano\Yepayment\PaymentTypes\FloosakPay::checkAndCompletePay(['order_id' => 200, 'otp' => '123456']);
if ($result['success']) {
    // Success
}
```

### 5.3. Direct Transaction Status Query

```php
$floosak = new FloosakPay();
$status = $floosak->checkTransactionStatus('550e8400-...');
if ($status['success'] && $status['completed']) {
    echo "Transaction completed";
}
```

### 5.4. Refunding a Transaction

```php
$order = Order::find(200);
$floosak = new FloosakPay($order);
$refundResult = $floosak->reverse($order->payment_first_trans_id, 'Refund requested by customer');
if ($refundResult['success']) {
    // Refund successful, order status becomes RefundedState
}
```

### 5.5. Using the Integrated Testing Interface

After logging into the control panel as an administrator, open the URL:
```
https://yourdomain.com/api/v1/yepayment/floosakpay/test-ui
```

The interface allows:
- Test authentication (verify phone, password, source_wallet_id).
- Create a pending transaction (simulate `process`) and display `purchase_id` and `request_id`.
- Confirm payment using OTP (manually or default `123456` in the test environment).
- Query transaction status.
- Refund an amount.
- Full flow (create + confirm + query) in one step.
- Automated testing with a specified number of attempts (up to 5 times).
- Usage statistics (number of orders, success rate, recent logs).
- Local test logs (stored in `localStorage`).

---

## 6. Handling the Test Environment (Sandbox)

- In the test environment, `x-channel: merchant` is set as in production.
- The OTP value is fixed: **`123456`** for testing purposes (no real SMS is sent).
- You can use the test URL `https://staging.fintech-expert.net` (set in the `floosakpay_test_url` field).
- Ensure that the `source_wallet_id` is valid in the test environment.

---

## 7. Added Value

- **For developers:** A complete example of a **Two‑Step** payment gateway using OTP, with idempotency, token caching, and PaymentLog logging in the first stage. Demonstrates how to handle APIs that require `x-channel` and standard JSON body.
- **For merchants:** Support for Floosak Wallet as a digital payment method, enabling customers to pay directly from their digital wallets without needing credit cards.
- **For end users:** A seamless payment experience via OTP, where the customer receives a code on their phone and confirms it inside the app, enhancing security.

---

## 8. Integration Testing

Several testing layers are provided:

- **Test environment:** Use the URL `staging.fintech-expert.net` (set in `floosakpay_test_url`). OTP is fixed `123456`.
- **Quick test interface:** From the gateway settings page (partial file `_test_info.htm`), you can test authentication, create a transaction, confirm, and query directly.
- **Integrated testing interface:** `/api/v1/yepayment/floosakpay/test-ui` provides all necessary tools to fully test the gateway (manual, automated, statistics, logs).
- **Independent API endpoints:** Developers can use tools like Postman or cURL to directly connect to endpoints such as `/floosakpay/test-create-payment` and `/floosakpay/test-confirm-payment`.

**Quick test steps:**
1. Ensure `phone`, `password`, and `source_wallet_id` are set in the gateway settings page.
2. Open `/api/v1/yepayment/floosakpay/test-ui`.
3. Click "Test authentication" to verify credentials.
4. Enter an order number, customer phone number, and amount, then click "Create transaction".
5. `purchase_id` will appear with a message "Please enter OTP". Use OTP `123456`.
6. Click "Confirm payment" – it should succeed and the order status will change to paid.
7. Try "Query status" and "Refund".

---

## 9. Developer Notes

- **Two‑Step pattern:** `process()` **does not** complete the payment; it only creates a pending transaction and returns a message asking for OTP. You must call `complete()` (or `checkAndCompletePay`) after the customer enters the code to confirm the payment and update the order to `PaidState`.
- **Idempotency:** An existing transaction is checked at the beginning of `process()` to prevent duplicate creation. This is very important when the payment page is reloaded or the "Pay" button is clicked multiple times.
- **Token caching:** The Bearer Token is stored in Cache for 3500 seconds (slightly less than the actual 3600‑second expiry) to avoid using an expired token.
- **Using `HttpHelper`:** All API requests use `HttpHelper::sendJson()` with appropriate headers (`x-channel: merchant`).
- **PaymentLog logging:** The payment attempt is logged in the first stage (status `initiated`) even if the customer never completes the OTP, helping to track and analyse completion rates.
- **Support for `RedirectHelper`:** Full redirection capability has been included, making the gateway suitable for mobile applications that use Deep Links.
- **Testing OTP:** In the test environment, the OTP is fixed as `123456`. No real SMS is sent.
- **Extensibility:** Support for webhooks or additional interfaces can be added in the future using the available endpoints.

---

## 10. Bug Fixes

None – this release is dedicated to adding a new feature (FloosakPay).

---

## 11. Development and Testing Period

The FloosakPay gateway was developed during the period from **May 18, 2026** to **May 24, 2026**. This period included:
- Writing the core code for the `FloosakPay` class (extending `PaymentProvider`).
- Implementing `process()`, `complete()`, `reverse()`, `getAuthToken()`, `sendPayment()`, `confirmPayment()`, and `checkTransactionStatus()` methods.
- Adding the idempotency mechanism (check for existing transaction).
- Adding `PaymentLog` logging in `process()` with status `initiated`.
- Creating partial view files (`_info.htm`, `_test_info.htm`) and the integrated testing interface (`floosakpay-ui.htm`).
- Writing the necessary API endpoints for testing and monitoring in `routes.php` (11 endpoints).
- Adding translation keys in language files.
- Registering the gateway in `Plugin.php`.
- Conducting comprehensive integration tests to verify functionality with the Floosak environment (simulated using Postman and UI).

---

## 12. Relevant Links

- [FloosakPay Documentation (Developer Guide)](./docs/FloosakPay/Docs-FloosakPay-en.md)
- [SKILL-v2.5 for Creating Payment Gateways](./docs/FloosakPay/SKILL-v2.5.md)
- [Official Floosak API Documentation](./docs/FloosakPay/external/Merchant_Wallet_Integration_API_Documentation_RefStyle.pdf)
- [Postman Collection for FloosakPay Testing](./docs/FloosakPay/external/Marchant%20Last%20Version.json)
- [FloosakPay.php Class](./Nano/Yepayment/PaymentTypes/FloosakPay.php)
- [routes.php file of Nano.Yepayment (FloosakPay section)](./routes.php)
- [Technical Support Channel](https://nano2soft.com)

---

**This update prepared by:**  
NanoSoft Development Team – Electronic Payments Department  
**References:** Dheia Ali, Nano2Soft

## 2026-05-24 – 2026-05-25

**Update to the `Nano.AuthApi` Plugin – Version 1.0.20**

### Adding the `AccessManager` Class for Centralised and Advanced Permission and Access Management

---

### Summary of Updates

Version **1.0.20** of the `Nano.AuthApi` plugin introduces a new class called `AccessManager`, which is responsible for managing user permissions (Backend, Frontend, Guest) and controlling access scopes in a unified and extensible manner. The class integrates helper functions from `BasicHelper` (e.g., `checkAccessAllCompanys`) and provides a simple interface for applying access restrictions to Eloquent queries.

This update aims to unify permission logic across all system plugins, reduce duplicate code, and support advanced scenarios such as access by company, department, or state, showing only records created by the user, or displaying the children of a parent.

---

### Release Objectives

- **Provide a centralised class** for managing permissions instead of distributing logic across multiple controllers.
- **Support three user types:** Backend (Nano Soft App), Frontend (RainLab.User or any custom model), and Guest.
- **Support multiple access scopes:**
  - `all`: access to all records.
  - `own`: records created by the user (via `created_by`).
  - `created_by`: same as `own`.
  - `company`: access only to records of a specific company (with the ability to access all companies for authorised users).
  - `department`: access only to records of a specific department.
  - `state`: access only to records of a specific state.
  - `children`: for Frontend users of type parent, to display their children's records.
- **Provide methods to apply the scope to Query Builder** (`applyAccessScope`, `applyCompanyScope`, `applyDepartmentScope`, `applyStateScope`, `applyCreatedByScope`, `applyUserScope`, `applyChildrenScope`).
- **Support custom field names** (e.g., `companys_id`, `departments_id`, `state_id`, `created_by`) via flexible options.
- **Support permission checking** for Backend users using `hasAccess` or `hasPermission`.
- **Support filtering Frontend user types** via `ref_type` (e.g., `student`, `parent`).
- **Return detailed information** about the check result (allowed, scope, message, user type, user, config, applied scope details).

---

### New Features and Improvements

#### 1. `AccessManager` Class – Core Functions

- **`setUser($user) / getUser()`**: Set or get the current user (supports automatic retrieval from `AuthHelpers`, `BackendAuth`, `Auth`).
- **`getUserType($user)`**: Determines the user type (`backend`, `frontend`, `guest`).
- **`getUserRefType($user)`, `getUserRefId($user)`**: Extracts `ref_type` and `ref_id` (for users linked to models such as students).
- **`getUserCompanyId($user)`, `getUserDepartmentId($user)`**: Extracts the company ID and department ID from the user (if the user has those properties).
- **`check(string $operation, array $config, $user)`**: The core check method. Returns an array containing `allowed`, `scope`, `message`, `user_type`, `user`, `config`, `applied_scope_details`.

#### 2. Advanced Backend Scope Support

Backend behaviour can be customised via the `backend` option in the `$config` array:

```php
'backend' => [
    'allow' => true,                         // whether Backend users are allowed
    'check_permission' => true,              // whether to check permissions
    'permissions' => ['tss.example.access'], // required permissions
    'access_scope' => 'company',             // required scope
    'company_field' => 'companys_id',        // company field in the table
    'department_field' => 'departments_id',
    'state_field' => 'state_id',
    'created_by_field' => 'created_by',
    'company_id' => 2,                       // specific company ID (optional)
    'department_id' => 5,
    'state_id' => 1,
],
```

Supported `access_scope` values: `all`, `own`, `created_by`, `company`, `department`, `state`.

When using the `company`, `department`, or `state` scope, `AccessManager` first checks whether the user has permission to access all companies/departments/states (via `canAccessAllCompanies()` etc.). If they do, the effective scope becomes `all`. Otherwise, it uses the ID associated with the user or the ID passed in the settings.

#### 3. Frontend Support with `ref_type` Filtering and `children` Scope

```php
'frontend' => [
    'allow' => true,
    'allowed_ref_types' => ['student', 'parent'], // allowed types
    'access_scope' => 'own',                     // 'own' or 'children'
    'children_relation' => 'students',           // relationship name to fetch children
],
```

If the scope is `children`, the `children_relation` must be defined (e.g., `students` on a parent model). Then `applyChildrenScope()` can be used to apply the filter.

#### 4. Methods to Apply Scope to Query Builder

- **`applyAccessScope($query, $accessResult, $options)`**: A general method that uses the result of `check()` to apply the appropriate scope.
- **`applyCompanyScope($query, $companyId, $field)`**, **`applyDepartmentScope`**, **`applyStateScope`**, **`applyCreatedByScope`**, **`applyUserScope`**.
- **`applyChildrenScope($query, $parent, $studentIdField, $relation)`**: Applies a filter to fetch records of a specific parent’s children.

**Example:**

```php
$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed']) {
    $query = Student::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access, [
        'company_field' => 'companys_id',
        'department_field' => 'departments_id',
        'state_field' => 'state_id',
        'created_by_field' => 'created_by',
    ]);
    // if scope = children
    if ($access['scope'] === 'children') {
        $parent = Mparent::byUser($user)->first();
        $query = AccessManager::instance()->applyChildrenScope($query, $parent, 'student_id', 'students');
    }
}
```

#### 5. Helper Methods for Checking Company/Department/State Permissions

- **`canAccessAllCompanies($user)`**: Returns `true` if the user can access all companies (uses `BasicHelper::checkAccessAllCompanys`).
- **`canAccessAllDepartments($user)`**
- **`canAccessAllStates($user)`**
- **`canAccessDepartment($departmentId, $user)`**

These methods simplify the logic and ensure consistency with the rest of the system.

#### 6. `getPermissionArray` Method

Returns an array similar to that used in `OrderHelper::getArrayDataAfterPermission`, making it easier to integrate with legacy code.

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
// ['allowed'=>true, 'scope'=>'department', 'check_departments'=>true, 'departments_id'=>5, ...]
```

#### 7. Full Guest Support

Guest (unauthenticated) users can be allowed to access some operations via the `guest` settings:

```php
'guest' => [
    'allow' => true,
    'access_scope' => 'none',
],
```

---

### Usage Examples

#### 1. Checking Access to a List of Students for a Frontend Parent User

```php
$config = [
    'is_allow' => true,
    'frontend' => [
        'allow' => true,
        'allowed_ref_types' => ['parent'],
        'access_scope' => 'children',
        'children_relation' => 'students',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
if (!$access['allowed']) {
    throw new ApplicationException($access['message']);
}
// $access['scope'] = 'children'
```

#### 2. Applying `company` Scope for a Backend User

```php
$config = [
    'backend' => [
        'allow' => true,
        'check_permission' => true,
        'permissions' => ['tss.student.students.access'],
        'access_scope' => 'company',
        'company_field' => 'companys_id',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed']) {
    $query = Student::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access);
    // A condition will be added: companys_id = (the user's associated company ID or the one specified in config)
}
```

#### 3. Getting a Permission Settings Array for Use in `listExtendQuery`

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
if ($perms['allowed']) {
    if ($perms['check_departments']) {
        $query->where('departments_id', $perms['departments_id']);
    }
}
```

---

### Backward Compatibility

- **No old methods have been changed** in `Nano.AuthApi`. Only the `AccessManager` class has been added.
- Existing `AuthHelpers` functions (`getCurrentUser`, `isBackendUser`, etc.) can still be used without any impact.
- No database or configuration changes have been made.

---

### Upgrade Requirements (from any previous version to 1.0.20)

1. **Update the code**:
   - Replace the `classes/AccessManager.php` file with the new version (attached).
   - (Optional) Update `Plugin.php` if you want to register `AccessManager` as a Singleton or provide an alias, but it is not required.

2. **No new database migrations**.

3. **No configuration changes** – the `config.php` file remains the same.

4. **Update the `version.yaml` file**:
   - Add the line `1.0.20:` as shown at the beginning of this document.

5. **Run `php artisan plugin:refresh Nano.AuthApi`** (optional) to register the new version.

6. **Test the functionality**:
   - Test `AccessManager::check()` with different scenarios (Backend, Frontend, Guest).
   - Ensure that `applyAccessScope` adds correct `where` conditions.
   - Verify the `canAccessAllCompanies` and other helper methods.

---

### Benefits and Added Value

- **Unifies permission logic** across all API plugins (`Nano.StudentsApi`, `Nano.AbsenceApi`, ...), reducing code duplication and standardising behaviour.
- **Precise access control** based on company/department/state, making it easy to build multi‑tenant systems.
- **Native support for the `children` scope** for parents, simplifying school‑oriented applications.
- **Easy extensibility** to add new scopes (e.g., `team`, `project`) without modifying the base class.
- **Improved security** through a clear separation of permission logic from business logic.

---

### Conclusion

Version **1.0.20** of `Nano.AuthApi` represents an important step towards unifying permission management in the Nano ecosystem. With `AccessManager`, developers can easily apply complex access restrictions while maintaining flexibility and extensibility. We recommend using this class in all future API plugins to ensure security consistency and ease of maintenance.

---

**Reference documentation**:
- [`AccessManager` class documentation](./docs/AuthApi/Docs-AccessManager-en.md)
- [`AuthHelpers` class documentation](./docs/AuthApi/Docs-AuthHelpers-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md)

## 2026-05-24 – 2026-05-26

**Update to the `Tss.Student` Plugin – Version 1.0.13**

### Restructuring `getRecords` Functions, Adding Specialised Helpers, and Supporting `AccessManager` and `AdvancedQueryHelper`

---

### Summary of Updates

Version **1.0.13** of the `Tss.Student` plugin represents a qualitative leap in the way student, parent, and academic record (`StudentRecord`) data is retrieved via the API and the Backend system. The `getRecords` functions have been restructured and unified in a new `StudentRecordsHelper` class, with similar functions added for parents (`getParentRecords`) and student records (`getStudentRecordRecords`). Advanced techniques such as `AccessManager` for flexible permission control and `AdvancedQueryHelper` for dynamic filtering have also been integrated.

This release aims to provide a unified, secure, and extensible interface for student and parent queries, with improved performance through built‑in caching, support for advanced filters (`is_or`, `is_not`, `is_force`, `is_or_null`), and intelligent text search.

---

### Release Objectives

- **Unify record retrieval logic** in a single helper class (`StudentRecordsHelper`) that can be used from both API and Backend controllers.
- **Create specialised functions** for:
  - Students (`getRecords`)
  - Parents (`getParentRecords`)
  - Student academic records (`getStudentRecordRecords`)
- **Integrate `AccessManager`** for managing permissions and access scopes (`all`, `own`, `created_by`, `company`, `department`, `state`, `children`).
- **Leverage `AdvancedQueryHelper`** to execute dynamic filters with support for advanced logic (`is_or`, `is_not`, `is_force`, `is_or_null`).
- **Provide a unified response structure** for all `getRecords` functions, including `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`.
- **Support caching** with unique keys, tags, and forced refresh.
- **Support events** (`event_before_name`, `event_after_name`, `event_list_name`) with the ability to disable them.
- **Add linking functions** between user and student/parent (e.g., `getStudentByUser`, `getUserByStudent`, `getMparentByUser`, `getUserByMparent`).
- **Add `Mparent`‑specific helper functions** such as `getStudentIdsByMparent`, `getStudentsByMparent`, `getStudentsByUser`.
- **Add `StudentRecord`‑specific helper functions** such as `getCurrentStudentRecord`, `getStudentIdFromRecord`.

---

### New Features and Improvements

#### 1. `StudentRecordsHelper` Class – Unified Record Retrieval Helper

A new class has been created at `Tss\Student\Helpers\StudentRecordsHelper` containing a general `getRecords` function that can be used for any model, along with specialised functions for students, parents, and student records.

**Key features of the general `getRecords` function:**

- **Safely merges default options** (checking for key existence using `array_key_exists`).
- **Uses `AdvancedQueryManager::scopeWhereField`** to apply filters with support for `is_or`, `is_not`, `is_force`, `is_or_null`.
- **Supports advanced text search (`q`)** with `is_advanced_search` and `is_search_weights` options.
- **Supports `with_count`** to load counts of related relationships.
- **Supports `group_by`** (as array or string) and `having`.
- **Supports caching** via `cache_enabled`, `cache_key`, `cache_ttl`, `cache_tags`, `cache_force_refresh`.
- **Supports events** (`fire_before_event`, `fire_after_event`) with customisable event names.
- **Supports `is_to_sql`** to log the SQL query in `trace_log` for debugging.
- **Supports `withTrashed` and `onlyTrashed`** for working with soft‑deleted records.
- **Supports different output options**: `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`.

#### 2. `getRecords` Function in `StudentHelper` (for fetching students)

The existing `StudentHelper::getRecords` function has been updated to become a simple wrapper that calls `StudentRecordsHelper::getRecords` with the `Student` model. This ensures backward compatibility with existing code while taking advantage of the new features.

**Usage example:**

```php
$result = StudentHelper::getRecords([
    'gender' => 'male',
    'is_active' => true,
    'q' => 'Mohamed',
    'orderBy' => 'full_name',
    'per_page' => 20,
    'cache_enabled' => true,
]);
```

#### 3. `getParentRecords` Function – Fetching Parents

A new function `StudentHelper::getParentRecords` has been added, which calls `StudentRecordsHelper` with the `Mparent` model. It supports all the filtering options available in `getRecords` plus additional filters specific to `Mparent`:

- `students_id`: filter parents who have children with specific IDs.
- `has_students` / `is_has_students`: filter by having children (or not).
- `has_students_active` / `is_has_students_active`: filter by having active children.
- `is_has_students_count`: filter by the number of children (supports comparisons like `['>', 2]` or `">=3"`).

**Usage example:**

```php
$result = StudentHelper::getParentRecords([
    'has_students' => true,
    'is_has_students_count' => ['>=', 2],
    'orderBy' => 'full_name',
]);
```

#### 4. `getStudentRecordRecords` Function – Fetching Student Academic Records

A new function `StudentHelper::getStudentRecordRecords` has been added, dedicated to the `Tss\School\Models\StudentRecord` model. It supports the following filters (in addition to the general filters):

- `student_id`, `year_id`, `class_id`, `group_id`.
- `status_class`, `status_mony`, `semster`, `currencys_id`, `periods_id`, `cost_centers_id`.
- `is_bus`, `is_discount`, `is_registration_fees`.
- Numeric filters with comparisons for `study_fees`, `registration_fees`, `discount`, `bus_fees` (e.g., `['>', 1000]`).
- Date filters (`field_date`, `date_at`, `to_date_at`, `created_at_from`, `created_at_to`, `updated_at_from`, `updated_at_to`).

**Usage example:**

```php
$result = StudentHelper::getStudentRecordRecords([
    'year_id' => 3,
    'status_class' => 'continue',
    'study_fees' => ['>', 5000],
    'with_count' => ['student', 'class'],
]);
```

#### 5. `AccessManager` Support for Permissions

`AccessManager` has been integrated into the `getRecords` functions so that a permission check result (`access_result`) can be passed in and the scopes automatically applied to the query.

**Example in an API controller:**

```php
$config = [
    'is_allow' => true,
    'backend' => [
        'allow' => true,
        'check_permission' => true,
        'permissions' => ['tss.student.students.access'],
        'access_scope' => 'all',
    ],
    'frontend' => [
        'allow' => true,
        'allowed_ref_types' => ['student', 'parent'],
        'access_scope' => 'own',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
$options['access_result'] = $access;
$result = StudentHelper::getRecords($options);
```

Inside `StudentRecordsHelper::getRecords`, `AccessManager::instance()->applyAccessScope()` is called to automatically apply the permission conditions.

#### 6. User‑Student and User‑Parent Linking Functions

The following functions have been added (or improved) in `StudentHelper`:

- `getStudentByUser($user)`: retrieves the student object associated with a given user.
- `getUserByStudent($student)`: retrieves the user object associated with a student.
- `getMparentByUser($user)`: retrieves the parent object associated with a given user.
- `getUserByMparent($mparent)`: retrieves the user object associated with a parent.
- `getStudentIdsByMparent($mparent, $useCache)`: returns an array of student IDs belonging to a parent.
- `getStudentsByMparent($mparent, array $options)`: retrieves student objects belonging to a parent, supporting `getRecords` options.
- `getStudentsByUser($user, array $options)`: retrieves students associated with a user (supports the student themselves or a parent’s children).
- `getStudentIdFromRecord($record)`: extracts the student ID from a `StudentRecord` object or record ID.
- `getCurrentStudentRecord($student, array $options)`: retrieves the current student record (usually with `status_class = 'continue'`) with support for year and class options.

**Example using `getStudentsByUser`:**

```php
$students = StudentHelper::getStudentsByUser(null, ['is_active' => true]);
// If the current user is a parent -> returns their children
// If the current user is a student -> returns only their own data
```

#### 7. Improved Advanced Filter Handling (AdvancedQueryHelper)

`AdvancedQueryManager::scopeWhereField` has been applied to all filters, allowing comma‑separated multiple values, and the use of `is_or`, `is_not`, `is_force`, `is_or_null` logic. This is now available for all filterable fields in students, parents, and student records.

**Example:**

```php
$options = [
    'gender' => 'male,female',           // values separated by commas
    'is_or_gender' => true,              // OR between values
    'is_not_gender' => false,
    'is_force_gender' => true,
    'is_or_null_gender' => false,        // include records where gender = null
];
```

#### 8. Unified Response Structure

All `getRecords` functions (for students, parents, student records) now return a unified array with the following structure:

```php
[
    'code'    => 200,
    'status'  => true,
    'message' => '...',
    'data'    => $data,          // Builder, Collection, Paginator, or transformed array
    'error'   => null,
    'errors'  => null,
    'input_data'   => $input,    // original options
    'process_data' => $processed, // options after merging
    'debug'   => [...]            // only when APP_DEBUG=true
]
```

This simplifies handling results in API layers and provides tracing information for development.

#### 9. Caching Support

Full caching options have been added:

- `cache_enabled`: enable caching (default `false`, can be enabled via settings).
- `cache_key`: custom key (auto‑generated if not provided).
- `cache_ttl`: time‑to‑live in minutes (default `60`).
- `cache_tags`: cache tags (e.g., for Redis).
- `cache_force_refresh`: ignore cache and fetch fresh data.

#### 10. Event Support

Three customisable events are fired:

- `event_before_name` (default `api.list.extendQueryBefore`) – before applying filters.
- `event_after_name` (default `api.list.extendQuery`) – after applying filters and before execution.
- `event_list_name` (default `StudentsApi.ApiControllers.Students.List`) – after obtaining a `Paginator` and before transformation (Transformer).

Events can be disabled via `fire_before_event` and `fire_after_event`.

---

### Practical Usage Examples

#### 1. Fetching Active Students with a Governorate Filter and Ordering by Name

```php
$result = StudentHelper::getRecords([
    'is_active' => true,
    'state_id' => '5',
    'orderBy' => 'full_name',
    'orderDirection' => 'asc',
]);
```

#### 2. Fetching Parents Who Have More Than Two Children

```php
$result = StudentHelper::getParentRecords([
    'is_has_students_count' => ['>', 2],
    'with_count' => ['Student'],
]);
```

#### 3. Fetching Student Records for a Specific Academic Year with Study Fees Greater Than 5000

```php
$result = StudentHelper::getStudentRecordRecords([
    'year_id' => 3,
    'study_fees' => ['>', 5000],
    'orderBy' => 'study_fees',
    'orderDirection' => 'desc',
]);
```

#### 4. Using `getCurrentStudentRecord` for a Specific Student

```php
$student = Student::find(100);
$currentRecord = StudentHelper::getCurrentStudentRecord($student, [
    'year_id' => 3,   // if not passed, the default year is used
    'status_class' => 'continue',
]);
if ($currentRecord) {
    echo "Class: " . $currentRecord->class_id;
}
```

#### 5. Using `getStudentsByUser` in an API Controller

```php
public function myChildren()
{
    $user = AuthHelpers::getCurrentUser();
    $students = StudentHelper::getStudentsByUser($user, ['is_active' => true]);
    return $this->respondWithCollection($students, new StudentTransformer);
}
```

---

### Backward Compatibility

- **All old functions in `StudentHelper` remain unchanged** (e.g., `getStudentRecord`, `getResultRecord`, `setUpStudentRecord`, date functions). No old function has been removed or modified.
- **The old `StudentHelper::getRecords` function** now transparently uses `StudentRecordsHelper::getRecords`, while preserving the previous output structure (array with `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`).
- **Any code that directly called `StudentHelper::getRecords` will continue to work** without modification.
- **Only new functions have been added** (`getParentRecords`, `getStudentRecordRecords`, additional linking functions). No breaking changes or radical modifications have been made.

---

### Upgrade Requirements (from 1.0.12 to 1.0.13)

1. **Update the code**:
   - Replace the `helpers/StudentHelper.php` file with the new version (which calls `StudentRecordsHelper` and contains the new functions).
   - Add the new file `helpers/StudentRecordsHelper.php`.
   - (Optional) Add `helpers/ParentRecordsHelper.php` and `StudentRecordRecordsHelper.php` if you wish to separate the functions, but they have been merged into `StudentHelper` to reduce dependencies.

2. **Update the `version.yaml` file**:
   Add the new version as follows:
   ```yaml
   1.0.13:
       - 'Improve StudentHelper and add new helpers with advanced features'
       - 'Add StudentRecordsHelper, ParentRecordsHelper, StudentRecordRecordsHelper'
       - 'Support AccessManager for permissions and AdvancedQueryHelper for dynamic filtering'
       - 'Add unified getRecords, getParentRecords, getStudentRecordRecords methods'
       - 'Enhance caching, event hooks, and sorting scopes'
   ```

3. **Run migrations**: No new database migrations.

4. **Update configuration (optional)**:
   New keys can be added in `config.php` for `nano.studentsapi::students.cache_enabled`, etc., but this is outside the scope of the `Tss.Student` plugin. It is recommended to update the `Nano.StudentsApi` settings to benefit from caching.

5. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

6. **Test the functionality**:
   - Ensure that `getRecords`, `getParentRecords`, and `getStudentRecordRecords` return expected results.
   - Test advanced filters (`is_or`, `is_not`, etc.) with multiple values.
   - Ensure that caching works when enabled.
   - Verify that events are fired at the appropriate times.

---

### Benefits and Added Value

- **Unified query handling** across students, parents, and student records, reducing code duplication and standardising behaviour.
- **High filtering flexibility** thanks to `AdvancedQueryHelper`, allowing clients to easily build complex queries.
- **Better performance** through built‑in caching and the ability to omit unnecessary columns.
- **Improved security** by integrating `AccessManager` and automatically applying access scopes, preventing unauthorised data leaks.
- **Full backward compatibility**, allowing the plugin to be upgraded without changing existing applications.
- **Extensibility** through events that allow other plugins to modify the query dynamically.

---

### Conclusion

Version **1.0.13** of the `Tss.Student` plugin represents a significant leap in the querying capabilities for student and parent data. By restructuring the `getRecords` functions, adding specialised helpers, and integrating `AccessManager` and `AdvancedQueryHelper`, the system can now provide a unified, secure, and high‑performance interface. All new functions maintain backward compatibility, ensuring a smooth upgrade process for existing projects. These improvements open the door to more interactive applications and advanced reports based on a clean and powerful API.

---

**Reference documentation**:
- [General documentation for the Tss.Student plugin](./docs/Student/Docs-Student-en.md)
- [StudentHelper class documentation](./docs/Student/Docs-StudentHelper-Class-en.md)
- [StudentRecordsHelper class documentation](./docs/Student/Docs-StudentRecordsHelper-Class-en.md)
- [AccessManager methods documentation](./docs/AuthApi/Docs-AccessManager-en.md)
- [AdvancedQueryHelper documentation](./docs/querybuilder/Docs-AdvancedQueryHelper-en.md)

## 2026-05-25 – 2026-05-26

**Update to the `Nano.AuthApi` Plugin – Version 1.0.21**

### Adding Advanced Features to `AccessManager`: Dynamic Resolver for Students and Parents, Advanced Filters, Centralised Configuration Loading, Simplified Methods, and Fallback Support.

---

### Summary of Updates

Version **1.0.21** of the `Nano.AuthApi` plugin introduces several substantial improvements to the `AccessManager` class, which is now responsible not only for permission management but also for handling advanced scenarios such as:

- **Dynamic resolver for students and parents (`resolveDynamicFrontendOptions`)** – automatically populates `student_id` and `record_id` based on user type and flexible rules from `config.php`.
- **Advanced filters control (`advanced_filters`)** – allows specifying which fields can use `is_or`, `is_not`, `is_or_null`, with rules per user type and `ref_type`.
- **Centralised permission configuration loading** – methods `getOperationByConfig`, `checkByResource` read settings directly from `config.php` without manual array building.
- **Fallback support** – methods `getOperationConfigWithFallback`, `checkWithFallback` allow operations like `show` to reuse `list` settings.
- **Simplified access methods** – `getOperationBySimpleKey`, `getAdvancedFiltersConfig` ease daily usage.

These improvements make `AccessManager` a comprehensive tool for permission management, filtering, and handling student/parent requirements in any API.

---

### Release Objectives

- **Enable school applications** to easily handle student and parent scenarios via `frontend_resolver`.
- **Provide fine-grained filter control** to ensure data security and prevent unauthorised users from passing dangerous filters.
- **Centralise permission settings** in `config.php` to reduce code duplication and simplify maintenance.
- **Simplify `AccessManager` usage** via shortcut methods (e.g., `checkByResource` instead of passing three parameters).
- **Support automatic fallback** for operations like `show` without writing complete settings.

---

### New Features and Improvements

#### 1. Dynamic Resolver for Students and Parents (`resolveDynamicFrontendOptions`)

A set of methods has been added to handle student and parent scenarios:

- **`resolveDynamicFrontendOptions($user, $options, $query, $resourceKey)`** – main method that analyses the user’s `ref_type` and applies rules defined in `frontend_resolver` from `config.php`.
- **`getFrontendResolverConfig($resourceKey)`** – loads resolver settings from the configuration file.
- **`ensureOptionsKeys($options, $keys)`** – ensures certain keys exist in the options array to avoid `undefined index` errors.

**Example `frontend_resolver` configuration in `config.php`:**

```php
'frontend_resolver' => [
    'enabled' => true,
    'apply_for_backend' => false,
    'apply_for_frontend' => true,
    'ref_type_rules' => [
        'student' => [
            'fields' => [
                'student_id' => ['source' => 'user', 'auto_fill' => true, 'force' => true],
                'record_id'  => ['source' => 'current_record', 'auto_fill_if_missing' => true, 'force' => true],
            ],
        ],
        'parent' => [
            'fields' => [
                'student_id' => ['source' => 'request', 'required' => true, 'validate_ownership' => true, 'force' => true],
                'record_id'  => ['source' => 'request_or_current', 'fallback_field' => 'student_id', 'force' => true],
            ],
        ],
    ],
],
```

When `resolveDynamicFrontendOptions` is called, `AccessManager` will automatically populate `student_id` and `record_id` according to these rules.

#### 2. Advanced Filters Control (`advanced_filters`)

Methods have been added to check whether a user is allowed to use `is_or_*`, `is_not_*`, `is_or_null_*` keys:

- **`isAdvancedFilterAllowed($user, $field, $operationConfig)`** – checks allowance based on `advanced_filters` in the operation configuration.
- **`filterAdvancedOptions($user, $options, $operationConfig)`** – removes any disallowed keys from `$options`.
- **`applyAdvancedFilterConstraints($user, $options, $operationConfig)`** – comprehensive function (alias for `filterAdvancedOptions`).

**Example `advanced_filters` configuration:**

```php
'advanced_filters' => [
    'enabled' => true,
    'default_for_backend' => true,
    'default_for_frontend' => false,
    'field_specific_rules' => [
        'student_id' => [
            'backend' => true,
            'frontend' => ['*' => false, 'parent' => true],
        ],
    ],
],
```

#### 3. Centralised Permission Configuration Loading from `config.php`

Methods have been added to read and normalise settings:

- **`getDefaultOperationConfig()`** – returns standard default settings.
- **`normalizeOperationConfig($config)`** – merges provided configuration with defaults.
- **`arrayMergeRecursiveDistinct($array1, $array2)`** – recursively merges arrays, replacing values instead of concatenating.
- **`getOperationConfig($plugin, $resource, $operation, $overrideConfig)`** – fetches settings from `config` using three parameters.
- **`checkWithConfig($operation, $plugin, $resource, $user, $overrideConfig)`** – combines fetching and checking.

#### 4. Simplified Methods Using a Single Key

To ease usage, methods that accept a single key instead of three have been added:

- **`getOperationByConfig($resourceKey, $overrideConfig)`** – key like `'nano.absenceapi::absences.list'`.
- **`checkByResource($resourceKey, $user, $overrideConfig)`** – automatically extracts the operation from the last part of the key.
- **`getAdvancedFiltersConfig($resourceKey, $overrideConfig)`** – returns only the `advanced_filters` array for the resource.
- **`getOperationBySimpleKey($simpleKey, $overrideConfig)`** – shortens writing `'nano.absenceapi::'` (e.g., `'absences.list'`).

#### 5. Fallback Support Between Operations

These methods help in scenarios such as `show` being able to use `list` settings if `show` settings are incomplete:

- **`getOperationConfigWithFallback($resourceKey, $fallbackResourceKey, $overrideConfig)`**
- **`checkWithFallback($resourceKey, $fallbackResourceKey, $user, $overrideConfig)`**

If `is_allow` exists in the primary settings, it is used directly. Otherwise, the fallback settings are merged with the primary ones.

#### 6. Additional Helper Methods for Student/Parent Scenarios

- **`applyFrontendStudentParentFilters($query, $user, $options, $studentIdColumn, $recordIdColumn, $tablePrefix, $resolveIfNeeded)`** – applies the resolved filters directly to the query.
- **`getStudentIdForFrontendUser($user, $options)`**, **`getRecordIdForFrontendUser($user, $options)`** – shortcut methods to fetch resolved values.
- **`validateParentStudentRelation($user, $studentId, $recordId)`** – checks that the student belongs to the parent.

---

### Practical Examples of the New Usage

#### 1. Checking `list` Permission Using `checkByResource`

```php
$access = AccessManager::checkByResource('nano.absenceapi::absences.list', $user);
if (!$access['allowed']) {
    return $this->errorUnauthorized($access['message']);
}
```

#### 2. Using `checkWithFallback` for the `show` Operation

```php
$access = AccessManager::checkWithFallback(
    'nano.absenceapi::absences.show',
    'nano.absenceapi::absences.list',
    $user
);
```

#### 3. Applying Advanced Filters and Removing Disallowed Ones

```php
$advConfig = AccessManager::getAdvancedFiltersConfig('nano.absenceapi::absences.list');
$options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advConfig]);
```

#### 4. Applying the Dynamic Resolver to Resolve `student_id` and `record_id`

```php
$options = AccessManager::instance()->resolveDynamicFrontendOptions(
    $user, $options, null, 'nano.absenceapi::absences.list'
);
// Then in getRecords(), apply the resolved filters
if (!empty($options['student_id']) && $options['is_force_student_id']) {
    $query->where('student_id', $options['student_id']);
}
```

#### 5. Applying `applyAccessScope` with the Ability to Disable Scopes

```php
$query = AccessManager::instance()->applyAccessScope($query, $access, [
    'company_field'    => 'companys_id',
    'department_field' => 'departments_id',
    // 'state_field' omitted -> no state filter
    'created_by_field' => 'created_by',
]);
```

---

### Backward Compatibility

- **All old methods** (e.g., `check`, `applyAccessScope`, `canAccessAllCompanies`) still work as before.
- **No functionality has been removed** – only additions have been made.
- **No database migrations are required.**
- **You can upgrade directly** without modifying existing code.

---

### Upgrade Requirements (from 1.0.20 to 1.0.21)

1. **Update the code**:
   - Replace the `AccessManager.php` file with the new version (attached).
   - No other changes in the plugin are required.

2. **Update the `version.yaml` file**:
   - Add version `1.0.21` as shown above.

3. **Update `config.php` files of API plugins (optional)**:
   - You can add `advanced_filters` and `frontend_resolver` sections to the resource settings to benefit from the new features.

4. **Run `php artisan plugin:refresh Nano.AuthApi` (optional).**

5. **Test the new features**:
   - Try `checkByResource` instead of the traditional `check`.
   - Try `checkWithFallback` for a `show` endpoint.
   - Try `filterAdvancedOptions` to clean request options.
   - Try `resolveDynamicFrontendOptions` in a controller that uses students and parents.

---

### Benefits and Added Value

- **Reduces duplicate code** in API controllers by up to 70%.
- **Centralises control** over permissions and filters, making behaviour changes easy via configuration files only.
- **Higher security** through fine‑grained control over advanced filters, preventing unauthorised users from using them.
- **Native support for school scenarios** (students and parents) in a natural and straightforward way.
- **Ease of maintenance** because any change to permission logic is made in one place.

---

### Conclusion

Version **1.0.21** of `Nano.AuthApi` establishes `AccessManager` as a comprehensive and advanced tool for managing permissions, filters, and relationships between users and entities (such as students and parents). Thanks to the new methods, building a secure and flexible API has become much easier and aligns perfectly with the `Nano-Api-SKILL.md` guide.

---

**Reference documentation**:
- [`AccessManager` class documentation](./docs/AuthApi/Docs-AccessManager-en.md)
- [`AccessManager-Permission` documentation](./docs/AuthApi/Docs-AccessManager-Permission-en.md)
- [General examples `AccessManager-Example`](./docs/AuthApi/Docs-AccessManager-Example-en.md)
- [Comprehensive practical example `AccessManager-Example-Api`](./docs/AuthApi/Docs-AccessManager-Example-Api-en.md)
- [`AuthHelpers` class documentation](./docs/AuthApi/Docs-AuthHelpers-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md)
- [Update to the `Nano.AbsenceApi` plugin (practical example)](./docs/AbsenceApi/Update-AbsenceApi-v1.1.0-en.md)

