# `KycDocumentManager` Class Documentation – KYC Document Integrations Manager

## Overview

`Nano3\Kyc\Classes\KycDocumentManager` is a comprehensive Singleton class that acts as a bridge between KYC (Know Your Customer) documents and any model in NanoSoft applications. The class automates the process of linking models (users, customers, companies, etc.) to `Document` records, automatically adding:

- **Behavior** `KycDocumentModel` that provides the `documents` relationship, advanced scopes, and statistics.
- **Custom form fields** in the backend control panel.
- **Multiple list columns** to display document statistics (total documents, verified, unverified, expired, category indicators).
- **Advanced list filters** including group filters with dependencies (`dependsOn`), switch/checkbox filters for presence/absence of documents, and combined (AND) filters.
- **API and report query integration** with intelligent parameter handling (comma, pipe, JSON, auto-detect) and complex groups.
- **Transformer integration** to add KYC status includes to any Transformer, with support for default includes and eager loading.

The class is built on the model of `Nano\Tags\Classes\TagManager` to achieve the highest level of flexibility and compatibility, with full customization for identity document requirements. It allows easy model registration via the `registerKycDocumentIntegrations` function in a Plugin, or using `registerModel` and `quickRegister`, and then handles the rest automatically.

---

## Integration Types

The class supports six types of integrations that can be enabled or disabled for each registered model individually:

| Constant | Type | Description |
|----------|------|-------------|
| `INTEGRATION_BEHAVIOR` | `behavior` | Adds the `KycDocumentModel` behavior (the `documents` relationship, scopes, statistics). |
| `INTEGRATION_FORM_FIELD` | `form_field` | Adds a custom `partial` field in backend forms. |
| `INTEGRATION_LIST_COLUMN` | `list_column` | Adds multiple customizable columns in backend lists. |
| `INTEGRATION_LIST_FILTER` | `list_filter` | Adds multiple advanced filters (group, checkbox, switch) with dependencies. |
| `INTEGRATION_QUERY` | `query` | Automatic filtering in API and report queries with intelligent parameter handling. |
| `INTEGRATION_TRANSFORMER` | `transformer` | Injects the `DynamicAddIncludeKyc` behavior into the model's Transformer, with support for default includes and eager loading. |

---

## Registering Models

### `registerModel(string $modelClass, array $config = []): self`

Registers a new model for KYC document management with the ability to customize settings. The function first loads default settings from the `config/nano3.kyc/extend.php` file if it exists, then merges them with any custom settings passed in `$config`, and uses hardcoded values as a last resort.

**Parameters**:
- `$modelClass`: Full class name of the model (e.g., `RainLab\User\Models\User`).
- `$config`: Optional configuration array that overrides default settings.

**Example**:
```php
$manager = KycDocumentManager::instance();
$manager->registerModel(\RainLab\User\Models\User::class, [
    'enabled' => true,
    'integrations' => [
        KycDocumentManager::INTEGRATION_QUERY => true,
    ],
]);
```

### `quickRegister(string $modelClass, ?string $controllerClass = null, array $options = []): self`

Quick registration with simplified options.

```php
$manager->quickRegister(User::class, null, [
    'with_form'   => true,
    'with_column' => true,
    'with_filter' => true,
    'tab'         => 'KYC',
]);
```

### Automatic Registration from Plugins

Any Plugin can define a `registerKycDocumentIntegrations` method that returns an array of models and their settings. `KycDocumentManager` will automatically load them when `getRegisteredModels()` is called.

```php
class Plugin extends PluginBase
{
    public function registerKycDocumentIntegrations()
    {
        return [
            \RainLab\User\Models\User::class => [
                'integrations' => [
                    KycDocumentManager::INTEGRATION_QUERY => true,
                ],
            ],
            \Backend\Models\User::class => [
                'integrations' => [
                    KycDocumentManager::INTEGRATION_FORM_FIELD => true,
                ],
            ],
        ];
    }
}
```

---

## Default Settings

### First: Central Configuration File `config/extend.php`

All integration settings can be customized via the file `plugins/nano3/kyc/config/extend.php`. The file contains four sections:

- `form_config` – Default form field settings.
- `list_config` – Array of column definitions.
- `filter_config` – Array of filter definitions.
- `query_config` – API and report query settings.

If the file exists and contains values, they are automatically loaded when registering models, and can be overridden by passing custom values in `$config`. If the file is empty, the class falls back to hardcoded default values.

#### Structure of `extend.php`

```php
return [
    'form_config' => [
        'field_name' => 'documents',
        'label'      => 'nano3.kyc::lang.extend.form.documents_label',
        // ... remaining settings
    ],
    'list_config' => [
        [
            'column_name' => 'documents_count',
            'label'       => 'nano3.kyc::lang.extend.list.documents_count',
            'type'        => 'number',
            'sortable'    => true,
            'width'       => '10%',
            'align'       => 'center',
        ],
        // ... additional columns
    ],
    'filter_config' => [
        [
            'scope_name' => 'kyc_documents_category',
            'label'      => 'nano3.kyc::lang.extend.filter.category_label',
            'type'       => 'group',
            'modelClass' => 'Nano3\Kyc\Models\Document',
            'options'    => 'getDocumentCategoryFilterOptions',
            'dependsOn'  => [],
        ],
        // ... additional filters
    ],
    'query_config' => [
        'param_keys' => ['kyc_status', 'kycStatus', 'document_status'],
        // ... remaining settings
    ],
];
```

#### Details of `list_config` (Columns)

Multiple columns can be defined as an array, and each column supports all standard OctoberCMS options:

| Option | Description |
|--------|-------------|
| `column_name` | Column name (required). |
| `label` | Display label. |
| `type` | Column type (text, number, switch, ...). |
| `searchable` | Searchable flag (true/false). |
| `sortable` | Sortable flag (true/false). |
| `invisible` | Hide the column by default. |
| `width` | Column width (e.g., "10%"). |
| `align` | Content alignment (left, right, center). |
| `clickable` | Enable clicking on the column. |
| `select` | Custom SQL query for the value. |
| `valueFrom` / `displayFrom` | Customize value source. |
| `relation` / `relationCount` | Display relationship count. |
| `cssClass` / `headCssClass` | CSS classes. |
| `permissions` | Required permissions to see the column. |

**Default columns**:
- `documents_count` – Total number of documents.
- `verified_documents_count` – Number of verified documents.
- `not_verified_documents_count` – Number of unverified documents.
- `expired_verified_documents_count` – Number of expired (verified) documents.
- `is_verifier_primary_id`, `is_verifier_secondary_id`, ... – switch indicators for existence of documents from each category.

#### Details of `filter_config` (Filters)

Multiple filters can be defined as an array, and each filter supports all standard OctoberCMS options:

| Option | Description |
|--------|-------------|
| `scope_name` | Filter scope name (required). |
| `label` | Display label. |
| `type` | Filter type (group, checkbox, switch, ...). |
| `scope` | Name of the query scope in the model. |
| `options` | Filter options (static array or model method name). |
| `modelClass` | Model class used for dynamic options (optional). |
| `default` | Default filter value. |
| `dependsOn` | Array of other filter names that this filter depends on. |
| `conditions` | Conditions for showing the filter (array or callable). |
| `emptyOption` | Label for the empty option. |
| `permissions` | Required permissions to see the filter. |

**Default filters**:
1. **Document category filter** (`kyc_documents_category`) – group, options from `DocumentType::getCategories()`.
2. **Document type filter** (`kyc_documents_type`) – group, depends on the selected category, options from `getDocumentTypeFilterOptions`.
3. **Document status filter** (`kyc_documents_status`) – group, static options (verified, pending, rejected, expired).
4. **"Any documents" toggle filter** (`is_toggel_any_kyc_documents`) – switch.
5. **"Verified documents" filter** (`is_toggel_kyc_documents_verified`) – checkbox.
6. **"Expired documents" filter** (`is_toggel_kyc_documents_expired`) – checkbox.
7. **"Verified primary identity" filter** (`is_toggel_kyc_documents_verified_and_category_primary_id`) – checkbox.

#### Details of `query_config` (API Queries)

| Key | Default | Description |
|-----|---------|-------------|
| `param_keys` | `['kyc_status', 'kycStatus', 'document_status', 'documentStatus']` | Parameter keys used in requests. |
| `wildcard_values` | `['*', 'all', 'ALL']` | Values meaning "skip the filter". |
| `scope_method` | `'whereHasDocuments'` | Name of the scope to be called on the query. |
| `logic_operator` | `'AND'` | Default logical operator. |
| `multiple_value_handler` | `['comma', 'array']` | Multiple value handlers (array or `'auto'`). |
| `value_processors` | `[verified, pending, rejected, expired, any, none]` | Custom processors for keywords. |
| `advanced_features` | `[debug_mode => false...]` | Advanced options (enable debug, disable certain processors...). |

#### Value Processors (`value_processors`)

| Key | Behavior |
|-----|----------|
| `verified` | Searches for verified documents (`hasVerifiedDocuments`) |
| `pending` | Searches for documents with status `pending` |
| `rejected` | Rejected documents |
| `expired` | Expired documents |
| `any` | Any existing documents (`has('documents')`) |
| `none` | No documents (`doesntHave('documents')`) |

### Second: `transformer_config` Settings

| Key | Default | Description |
|-----|---------|-------------|
| `include_relation` | `'kyc_status'` | Name of the added include. |
| `include_method` | `'includeKycStatus'` | Name of the method in the Transformer. |
| `default_include` | `false` | Whether to include it automatically in the Response. |
| `eager_load` | `false` | Whether to eager load the `documents` relationship in the Transformer. |
| `additional_fields` | `[documents_count => callback]` | Additional dynamically generated fields (e.g., document count). |

---

## Applying Integrations Automatically

### `applyBehaviorIntegration($modelClass)`

Adds the `KycDocumentModel` behavior to the model automatically on the `eloquent.booting: *` event. This behavior defines:

- A `documents` relationship (morphMany) to the `Document` model via the `owner` field.
- Powerful scopes such as `scopeWhereHasDocuments`, `scopeHasVerifiedDocuments`, `scopeAddDocumentsCount`, and Toggle scopes like `scopeIsToggelAnyKycDocuments`, `scopeIsToggelKycDocumentsVerified`, and others.
- Statistical methods such as `getDocumentsCountAttribute` and `hasVerifiedDocument()`.

### `applyFormFieldIntegration($modelClass)`

Listens to the `backend.form.extendFieldsBefore` event and adds a `partial` field to the specified tab, with support for `conditions`, `trigger`, and `dependsOn`.

### `applyListColumnIntegration($modelClass)`

Adds multiple columns (according to the `list_config` definitions) to backend lists. Uses the `normalizeColumnConfig` function to normalize options and ensure compatibility with OctoberCMS standards.

### `applyListFilterIntegration($modelClass)`

Adds multiple filters (according to the `filter_config` definitions) to backend lists, with support for `conditions`, `dependsOn`, and dynamic options. Uses the `normalizeFilterConfig` function to normalize options.

### `applyQueryIntegration($modelClass)`

The most important integration. It listens to the `api.list.extendQuery`, `reports.list.extendQuery`, and `EVENT_EXTEND_QUERY` events, automatically filtering queries based on the incoming parameters. It uses advanced logic to process values.

### `applyTransformerIntegration($modelClass)`

Injects the `DynamicAddIncludeKyc` behavior into the model's Transformer, and adds support for `default_include` (automatic inclusion), `eager_load` (preloading), and `additional_fields`.

---

## Advanced Query Integration

### Extracting and Analyzing Parameters

- Searches for the first key from `param_keys` present in `$params`.
- If the value is a wildcard, returns `null` (no filter).
- Checks for value processors.
- If the value contains `|`, splits into groups (complex).
- Otherwise, processes using `processMultipleValues` which supports comma, pipe, JSON, array, auto-detect.

### Logical Processors

- `applyAdvancedQueryLogic`: applies processor, complex, or simple values.
- `applyComplexQuery`: OR-connected groups.
- `applyQueryGroup`: a single group with internal OR/AND support.

### `createAdvancedQuery` Function

To build ready-made KYC queries with support for eager loading, document counts, and sorting.

---

## Automatic Integration Handler `KycIntegrationHandler`

Automatically binds events: applies the behavior on `eloquent.booting: *`, adds form fields/columns/filters in the backend, and enables API and Transformer integrations. Simply register it in `Plugin.php`:

```php
Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
```

---

## General Helper Functions

| Function | Description |
|----------|-------------|
| `normalizeColumnConfig(array $config)` | Normalizes column settings to be OctoberCMS-compatible. |
| `normalizeFilterConfig(array $config, $model)` | Normalizes filter settings to be OctoberCMS-compatible. |
| `checkConditions($model, $conditions)` | Evaluates conditions for showing a field/filter. |
| `checkValueIsNotAll($value)` | Checks that the value is not a wildcard. |
| `scopeWhereField(...)` | Flexible `WHERE` condition scope. |
| `parseComplexFilter($str, $sep)` | Parses a complex string into groups. |
| `applyLogicOperator(...)` | Applies multiple conditions with AND/OR. |

---

## Comprehensive Practical Examples (Updated)

### 1. Enabling Full Integration for Users via the Configuration File

Once the `extend.php` file exists with default settings and models are registered in `registerKycDocumentIntegrations`:

```php
// Plugin.php
public function registerKycDocumentIntegrations()
{
    return [
        \RainLab\User\Models\User::class => [
            'controller'   => \RainLab\User\Controllers\Users::class,
            'transformer'  => \Nano\AuthApi\Transformers\UserTransformer::class,
            'integrations' => [
                KycDocumentManager::INTEGRATION_BEHAVIOR    => true,
                KycDocumentManager::INTEGRATION_LIST_COLUMN => true,
                KycDocumentManager::INTEGRATION_LIST_FILTER => true,
            ],
        ],
    ];
}
```

The default columns and filters will automatically appear in the user list.

### 2. Using Interdependent Filters (Category => Type)

The `kyc_documents_type` filter is configured to depend on `kyc_documents_category`. When a category is selected, the type options change dynamically.

### 3. Using the New Toggle Scopes

```php
// Users who have a verified primary identity document (combined condition)
User::where(function($q) {
    $q->hasVerifiedDocuments()
      ->whereHasDocuments(['category' => 'primary_id']);
})->get();

// Or using the shorthand scope
User::isToggelKycDocumentsVerifiedAndCategory(1, 'primary_id')->get();
```

### 4. Customizing Columns and Filters for a Specific Project

Copy the `extend.php` file to your project using `php artisan config:publish nano3.kyc`, then modify the columns and filters as needed.

---

## Custom Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `nano3.kyc.modelRegistered` | `$modelClass, $config` | When a new model is registered. |
| `nano3.kyc.beforeApplyIntegrations` | `$modelClass, $config` | Before applying integrations. |
| `nano3.kyc.afterApplyIntegrations` | `$modelClass, $config` | After applying integrations. |
| `nano3.kyc.beforeQueryProcessing` | `$rawValue, $params, $config` | Before processing the filter value. |
| `nano3.kyc.afterQueryProcessing` | `$result, $params, $config` | After processing the filter value. |
| `nano3.kyc.extendQuery` | `$query, $params, $context` | To extend queries externally. |

---

## Managing Registered Models

| Function | Description |
|----------|-------------|
| `getRegisteredModels()` | Loads all registered models (from memory or plugins). |
| `isModelRegistered($class)` | Checks if the model is already registered. |
| `getModelConfig($class)` | Retrieves the configuration of a model. |
| `getQueryConfig($class)` | Retrieves the API query configuration of a model. |
| `getTransformerConfig($class)` | Retrieves the Transformer configuration of a model. |
| `getRegisteredModelClasses()` | List of registered model class names. |
| `hasRegisteredModels()` | Whether any models are registered. |
| `getModelsWithIntegration($type)` | Models that have a specific integration enabled. |
| `clearRegisteredModels()` | Clears all registrations (for testing). |

---

## Important Notes

- **Centralized settings**: It is recommended to customize columns and filters via the `config/extend.php` file rather than modifying code, to facilitate maintenance and upgrades.
- **Performance**: Statistical columns (e.g., `verified_documents_count`) require applying the appropriate scopes to the list query. `KycDocumentManager` does this automatically when the `list_column` integration is enabled.
- **Security**: The class does not perform user permission checks. It is assumed that the Controller does that before calling these functions.

---

## Conclusion

`KycDocumentManager` is the comprehensive and flexible solution for linking KYC documents to any model in your system. Thanks to its design based on centralized settings, support for multiple columns and filters, and intelligent Toggle scopes, you can build sophisticated KYC management experiences in the control panel and API interfaces with just a few lines of code.

---

## Additional Documentation

**Reference Documentation**:
- [General Plugin Documentation](./Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./Docs-Document-Model-en.md)
- [`KycDocumentModel` Behavior Documentation](./Docs-KycDocumentModel-Behaviors-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
