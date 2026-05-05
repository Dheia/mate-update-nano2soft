# `KycDocumentManager` Class Documentation – KYC Document Integrations Manager

## Overview

`Nano3\Kyc\Classes\KycDocumentManager` is a comprehensive Singleton class that acts as a bridge between KYC (Know Your Customer) documents and any model in NanoSoft applications. The class automates the process of linking models (users, customers, companies, etc.) to `Document` records, automatically adding:

- **Behavior** `KycDocumentModel` that provides the `documents` relationship, scopes, and statistics.
- **Custom form fields** in the backend control panel.
- **List columns** to display document counts.
- **List filters** to filter by document status.
- **Query integration** for API and reports with intelligent parameter handling (comma, pipe, JSON, auto-detect).
- **Transformer integration** to add KYC status includes to any Transformer.

The class is built on the model of `Nano\Tags\Classes\TagManager` to achieve the highest level of flexibility and compatibility, with full customization for identity document requirements. It allows easy model registration via the `registerKycDocumentIntegrations` function in a Plugin, and then handles the rest automatically.

---

## Integration Types

The class supports five types of integrations that can be enabled or disabled for each registered model individually:

| Constant | Type | Description |
|----------|------|-------------|
| `INTEGRATION_BEHAVIOR` | `behavior` | Adds the `KycDocumentModel` behavior (the `documents` relationship, scopes, statistics). |
| `INTEGRATION_FORM_FIELD` | `form_field` | Adds a custom `partial` field in backend forms. |
| `INTEGRATION_LIST_COLUMN` | `list_column` | Adds a `documents_count` column in backend lists. |
| `INTEGRATION_LIST_FILTER` | `list_filter` | Adds a scope filter by document status in backend lists. |
| `INTEGRATION_QUERY` | `query` | Automatic filtering in API and report queries with intelligent parameter handling. |
| `INTEGRATION_TRANSFORMER` | `transformer` | Injects the `DynamicAddIncludeKyc` behavior into the model's Transformer. |

---

## Registering Models

### `registerModel(string $modelClass, array $config = []): self`

Registers a new model for KYC document management with the ability to customize settings. If no settings are provided, the global default values detailed below are used.

**Parameters**:
- `$modelClass`: Full class name of the model (e.g., `RainLab\User\Models\User`).
- `$config`: Optional configuration array.

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

When registering a model without custom settings, the class relies on the following default settings (any part can be customized):

### `form_config` – Form Field Settings

| Key | Default | Description |
|-----|---------|-------------|
| `field_name` | `'documents'` | Name of the added field. |
| `label` | `'nano3.kyc::lang.documents.form.documents_label'` | Field label. |
| `comment` | `'nano3.kyc::lang.documents.form.documents_comment'` | Comment text. |
| `tab` | `'nano3.kyc::lang.plugin.tab'` | Tab where the field will appear. |
| `type` | `'partial'` | Field type (partial for custom template). |
| `path` | `'~/plugins/nano3/kyc/partials/_documents_relation.htm'` | Path to the partial template. |
| `span` | `'storm'` | Field width. |
| `cssClass` | `'col-sm-12'` | CSS class for the field. |
| `conditions` | `null` | Conditions to show the field (array or callable). |
| `trigger` | `[]` | Trigger settings (action, field, condition). |
| `dependsOn` | `[]` | DependsOn array. |

### `list_config` – List Column Settings

| Key | Default | Description |
|-----|---------|-------------|
| `column_name` | `'documents_count'` | Column name. |
| `label` | `'nano3.kyc::lang.documents.list.documents_count'` | Column label. |
| `type` | `'text'` | Column type. |
| `searchable` | `false` | Searchable flag. |
| `sortable` | `false` | Sortable flag. |

### `filter_config` – List Filter Settings

| Key | Default | Description |
|-----|---------|-------------|
| `scope_name` | `'kyc_documents_status'` | Scope name for the filter. |
| `label` | `'nano3.kyc::lang.documents.filter.status_label'` | Filter label. |
| `type` | `'group'` | Filter type. |
| `options` | `[verified, pending, rejected, expired]` | Filter options (array). |
| `conditions` | `null` | Conditions to show the filter. |
| `dependsOn` | `[]` | Filter dependency. |

### `query_config` – API and Report Query Settings

| Key | Default | Description |
|-----|---------|-------------|
| `param_keys` | `['kyc_status', 'kycStatus', 'document_status', 'documentStatus']` | Parameter keys searched in `$params`. |
| `wildcard_values` | `['*', 'all', 'ALL']` | Values meaning "skip the filter". |
| `scope_method` | `'whereHasDocuments'` | Name of the scope to be called on the query. |
| `logic_operator` | `'AND'` | Default logical operator. |
| `multiple_value_handler` | `['comma', 'array']` | Multiple value handlers (array or `'auto'`). |
| `value_processors` | `[verified, pending, rejected, expired, any, none]` | Custom processors for keywords (see explanation below). |
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

### `transformer_config` – Transformer Settings

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
- Powerful scopes such as `scopeWhereHasDocuments`, `scopeHasVerifiedDocuments`, and `scopeAddDocumentsCount`.
- Statistical methods such as `getDocumentsCountAttribute` and `hasVerifiedDocument()`.

### `applyFormFieldIntegration($modelClass)`

Listens to the `backend.form.extendFieldsBefore` event and adds a `partial` field to the specified tab, with support for `conditions`, `trigger`, and `dependsOn`.

### `applyListColumnIntegration($modelClass)`

Adds a `documents_count` column to backend lists.

### `applyListFilterIntegration($modelClass)`

Adds a `kyc_documents_status` filter with status options to backend lists, with support for `conditions` and `dependsOn`.

### `applyQueryIntegration($modelClass)`

The most important integration. It listens to the `api.list.extendQuery`, `reports.list.extendQuery`, and `EVENT_EXTEND_QUERY` events, automatically filtering queries based on the incoming parameters. It uses advanced logic to process values as described below.

### `applyTransformerIntegration($modelClass)`

Injects the `DynamicAddIncludeKyc` behavior into the model's Transformer, adding includes such as `kyc_status`, `kyc_documents`, `kyc_verification_summary`, and others.

---

## Advanced Query Integration

The class extracts the filter value from `$params` using `extractDocumentParams`, then applies one of the following paths:

1. **Processor** – if the value matches a `value_processor` (e.g., `verified`, `any`, `none`).
2. **Complex Groups** – if the value contains `|` (pipe), it is split into groups that are combined using `OR`.
3. **Simple Values** – processed using `processMultipleValues` with support for comma, pipe, JSON, array, auto-detect.

### `extractDocumentParams(array $params, array $config): ?array`

- Searches for the first key from `param_keys` present in `$params`.
- If the value is a wildcard (`*`, `all`, `ALL`) or empty, returns `null` (filter not applied).
- Checks for processors.
- If the value contains `|` and `complex_handling_enabled`, splits into groups.
- Otherwise, processes the value using `processMultipleValues`.

### `processMultipleValues($value, $handler)`

Supports handlers:

- `'comma'`: `"verified,pending"` → `['_values' => ['verified','pending'], '_logic' => 'AND']`
- `'pipe'`: `"verified|pending"` → `['_values' => ['verified','pending'], '_logic' => 'OR']`
- `'json'`: `'["verified","pending"]'` → same as comma
- `'array'`: single value.
- `'auto'`: automatically detects the format.

You can pass a handler array (e.g., `['comma', 'array']`) to try each in sequence and merge the results.

### `applyAdvancedQueryLogic($query, $extracted, array $qc, string $modelClass)`

The central function that applies the extracted logic to the query:

1. If `_processor` exists, executes the processor.
2. If `_complex` exists, applies `applyComplexQuery` (OR-connected groups).
3. If `_values` exists, applies `applyQueryGroup` respecting the logical operator (`AND`/`OR`).

### `applyComplexQuery($query, $complexData, $scopeMethod, $advanced)`

Applies filter groups with each other using `OR`. Example:

```
kyc_status = verified,pending|rejected
```
**Interpretation**: Users who have (verified or pending) or (rejected).

**Flow**:
```php
$query->where(function ($q) {
    $q->whereHasDocuments(['status' => ['verified','pending']]);  // first group
    $q->orWhere(function ($orQ) {
        $orQ->whereHasDocuments(['status' => ['rejected']]);      // second group
    });
});
```

### `applyQueryGroup($query, $group, $scopeMethod)`

Applies a single group, supporting internal `OR` logic. If the group has multiple values and the logic is `OR`, it will generate an `orWhere` query for each value.

---

## Transformer Integration

Adds the `DynamicAddIncludeKyc` behavior, which registers includes such as `kyc_status`, `kyc_documents`, `kyc_verification_summary`, and others.

Supports:

- `default_include`: Automatically includes `kyc_status` in every Response.
- `eager_load`: Preloads the `documents` relationship.
- `additional_fields`: Adds dynamic fields like `documents_count` to the API Response.

---

## Automatic Integration Handler `KycIntegrationHandler`

`Nano3\Kyc\EventHandlers\KycIntegrationHandler` is the class responsible for binding events and activating all integrations automatically at startup. Register it once in `Plugin.php`:

```php
Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
```

The handler will:

- Load all registered models.
- Listen to the `eloquent.booting: *` event to apply the behavior.
- In backend: apply form field, list column, and filter integrations.
- Always: apply query and transformer integrations.

---

## General Helper Functions

| Function | Description |
|----------|-------------|
| `checkValueIsNotAll($value)` | Checks that the value is not a wildcard (`*`, `all`). |
| `scopeWhereField($query, $field, $value, …)` | Flexible scope to apply `WHERE` conditions with support for `is_or`, `is_not`, `is_force`. |
| `parseComplexFilter($str, $sep)` | Parses a complex string into `_values` groups. |
| `applyLogicOperator($query, $conditions, $operator, $callback)` | Applies multiple conditions with a logical operator (`AND`/`OR`). |
| `isJson($str)` | Validates JSON. |
| `checkConditions($model, $conditions)` | Evaluates conditions for showing fields/filters (callable or array). |
| `handleDebug($msg)` | Logs errors only in development mode. |

---

## Comprehensive Practical Examples

### 1. Registering the User Model with Only API Query Integration

```php
// In Plugin.php
public function registerKycDocumentIntegrations()
{
    return [
        \RainLab\User\Models\User::class => [
            'integrations' => [
                KycDocumentManager::INTEGRATION_QUERY => true,
            ],
        ],
    ];
}
```

Now when calling `GET /api/users?kyc_status=verified`, users with verified documents are automatically filtered.

### 2. Registering with Full Backend Integration

```php
$manager->registerModel(\Backend\Models\User::class, [
    'controller' => \Backend\Controllers\Users::class,
    'integrations' => [
        KycDocumentManager::INTEGRATION_BEHAVIOR    => true,
        KycDocumentManager::INTEGRATION_FORM_FIELD  => true,
        KycDocumentManager::INTEGRATION_LIST_COLUMN => true,
        KycDocumentManager::INTEGRATION_LIST_FILTER => true,
        KycDocumentManager::INTEGRATION_QUERY       => true,
    ],
    'form_config' => ['tab' => 'KYC Documents'],
]);
```

### 3. Using `createAdvancedQuery` in a Custom Report

```php
$query = KycDocumentManager::instance()->createAdvancedQuery(
    User::class,
    ['kyc_status' => 'verified|pending'],
    [
        'with_documents'    => true,
        'documents_count'   => true,
        'documents_sort'    => ['column' => 'created_at', 'direction' => 'desc'],
        'conditions'        => function ($q) {
            $q->where('is_active', true);
        }
    ]
);

$users = $query->get();

foreach ($users as $user) {
    echo $user->name . ' has ' . $user->documents_count . ' documents.';
    foreach ($user->documents as $doc) {
        echo ' - ' . $doc->document_type . ' (' . $doc->status . ')';
    }
}
```

### 4. Enabling Transformer with default_include

```php
$manager->registerModel(User::class, [
    'transformer' => \Nano\AuthApi\Transformers\UserTransformer::class,
    'transformer_config' => [
        'default_include' => true,  // kyc_status appears automatically
        'eager_load'      => true,   // preload documents
        'additional_fields' => [
            'verified_documents_count' => function ($model) {
                return $model->documents()->where('is_verified', true)->count();
            }
        ]
    ],
]);
```

Then calling `GET /api/users/1` will automatically return `kyc_status` and `verified_documents_count`.

### 5. Using Scopes Directly in Code

After adding the behavior, you can use scopes directly:

```php
// Users who have verified primary identity documents
User::whereHasDocuments([
    'category' => 'primary_id',
    'is_verified' => true
])->get();

// Sort users by their document count
User::addDocumentsCount('total_docs')
    ->sortByDocumentsCount('DESC')
    ->get();

// Check a specific user
$user = User::find(1);
if ($user->hasVerifiedDocument()) {
    echo "Has verified documents: " . $user->verified_documents_count;
}
```

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
| `getRegisteredModelClasses()` | List of registered model class names. |
| `hasRegisteredModels()` | Whether any models are registered. |
| `getModelsWithIntegration($type)` | Models that have a specific integration enabled. |
| `clearRegisteredModels()` | Clears all registrations (for testing). |

---

## Important Notes

- **Caching**: The class does not use its own caching, but it listens to model events (`eloquent.booting: *`) to apply the behavior. When changing settings at runtime, you may need to reload the page or clear the general cache.
- **Dependency on `DocumentType`**: The default filter options (statuses) are built on the values supported by `DocumentType`. Any expansion in status types should be reflected here.
- **Security**: The class does not perform user permission checks. It is assumed that the Controller does that before calling these functions.

---

## Conclusion

`KycDocumentManager` is the comprehensive and flexible solution for linking KYC documents to any model in your system. Thanks to its design inspired by `TagManager`, it provides a unified and powerful experience that allows you to enable multiple integrations with just a few lines of code. Whether you are building an API, a control panel, or reports, you will find in this class the tools you need to manage KYC status with ease and professionalism.

## Additional Documentation

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./docs/Kyc/Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [`KycDocumentModel` Behavior Documentation](./docs/Kyc/Docs-KycDocumentModel-Behaviors-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
