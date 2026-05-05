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
