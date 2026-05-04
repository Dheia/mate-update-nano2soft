# Update 2026-5

**Updates month four May**

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


