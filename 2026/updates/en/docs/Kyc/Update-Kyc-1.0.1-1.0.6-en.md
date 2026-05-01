## 2026-04-10 – 2026-04-16

**Updates to the `Nano3.Kyc` plugin – Versions 1.0.1 to 1.0.6**

### Summary of Updates

The `Nano3.Kyc` plugin has been developed to be the integrated solution for managing identity verification (KYC) processes within NanoSoft App. The updates from versions 1.0.1 to 1.0.6 included building a complete system comprising:

- **Creating the database structure** and the `nano3_kyc_documents` table designed to accommodate all types of KYC documents.
- **Developing the `DocumentType` class** which defines over 28 document types with their properties, dynamic fields, and validation rules.
- **Developing the `Manager` class** responsible for all document management operations (create, update, delete, approve, reject) with support for transactions, events, and testing.
- **Building the `Document` model** with an integrated set of traits for managing scopes, options, caching, and filters.
- **Developing a complete API** via the `Nano3\Kyc\APIControllers\Documents` controller following RESTful patterns.
- **Developing a complete backend interface** via the `Nano3\Kyc\Controllers\Documents` controller with lists, forms, and dynamic filters.
- **Full multi‑language support** (Arabic and English) with a comprehensive language file.
- **Flexible customisation system** via the configuration file `config/nano3/kyc/document_types.php`.

---

### Version 1.0.1 – Initial Launch and Document Table Creation

#### Version Goals

- Create the base plugin `Nano3.Kyc` within NanoSoft App.
- Design and create the `nano3_kyc_documents` table as the backbone for storing all KYC documents.

#### New Features

##### 1. Creation of the `nano3_kyc_documents` Table

A comprehensive table designed to support all KYC document types with high flexibility:

| Field Group | Description |
| :--- | :--- |
| **Basic Identifiers** | `document_type`, `document_category`, `code`, `barcode`, `document_number` |
| **Dates** | `issue_date`, `expiry_date` (type `date` for performance), `verified_at` |
| **Personal Data** | `full_name`, `date_of_birth`, `nationality`, `country_of_issue`, `religion` |
| **Address** | `address_line1`, `address_line2`, `address_city`, `address_state`, `address_postcode`, `address_country` |
| **Corporate Data** | `company_name`, `registration_number`, `tax_number`, `incorporation_date` |
| **Polymorphic Relations** | `owner_id`/`owner_type`, `verifier_id`/`verifier_type`, `subject_id`/`subject_type` |
| **Verification** | `is_verified`, `verification_score`, `verified_at` |
| **Physical Copies** | `is_physical_submitted`, `physical_copy_type`, `physical_received_at`, `physical_returned_at`, `physical_notes` |
| **Status & Publishing** | `status`, `is_active`, `is_default`, `is_public`, `is_published`, `published_at`, `unpublished_at` |
| **Flexible Data (JSON)** | `fields_data`, `metadata`, `other_data`, `config_data` |
| **Files** | `file_ids`, `file_urls` |
| **Tracking** | `timestamps`, `softDeletes`, `created_by`, `updated_by`, `deleted_by` |

Indexes were added on the most commonly searched fields (`document_type`, `document_number`, `full_name`, `company_name`, `status`, `is_verified`, polymorphic relations) to ensure high performance.

##### 2. Basic Plugin Structure

The initial plugin structure was created:
```
plugins/nano3/kyc/
├── Plugin.php              # Plugin definition file
├── updates/
│   ├── version.yaml        # Version log
│   └── create_table_nano3_kyc_documents.php
└── lang/                   # Translation folder (to be filled later)
```

#### Benefits

- A database ready to accommodate any current or future KYC document type.
- High flexibility thanks to the `fields_data` (JSON) field that stores specialised fields for each document type.
- Optimised indexes for common queries.

---

### Version 1.0.2 – Development of `DocumentType` Class and `Document` Model with Core Traits

#### Version Goals

- Develop the `DocumentType` class as the central reference for defining document types and their properties.
- Build the `Document` model with a set of core traits.

#### New Features

##### 1. `DocumentType` Class

The `DocumentType` class (`Nano3\Kyc\Classes\DocumentType`) was developed to provide:

- **Definition of 28+ document types** distributed across 6 categories (Primary ID, Secondary ID, Proof of Address, Corporate Documents, UBO, Additional Verification).
- **Advanced properties per type**: `has_expiry`, `max_age_days`, `verification_weight`, `allowed_mime_types`, `max_file_size_kb`, `allowed_for_entity`, `requires_back_side`, `requires_selfie_match`.
- **Dynamic field schema (`fields`)**: complete definition of required fields for each document type with properties (`label`, `type`, `required`, `validation`, `order`, `extractable`).
- **Helper functions for validation**: `getValidationRules()`, `isValidType()`, `isValidForPartyType()`, `isDocumentAcceptable()`, `isDocumentExpiring()`.
- **Data mapping functions**: `mapInputToDocumentData()` and `mapDocumentToOutput()` to simplify working with the hybrid storage structure (static columns + JSON).
- **Loading settings from a `config` file**: with the ability to override programmatic defaults.

**Example document type definition:**
```php
'passport' => [
    'category' => 'primary_id',
    'has_expiry' => true,
    'verification_weight' => 100,
    'allowed_mime_types' => ['image/jpeg', 'image/png', 'application/pdf'],
    'fields' => [
        'document_number' => ['type' => 'text', 'required' => true],
        'full_name' => ['type' => 'text', 'required' => true],
        'expiry_date' => ['type' => 'date', 'required' => true, 'validation' => ['after_or_equal' => 'today']],
        // ... etc
    ],
]
```

##### 2. `Document` Model and Associated Traits

The `Document` model (`Nano3\Kyc\Models\Document`) was built with a set of traits covering all aspects of document management:

| Trait | Description |
| :--- | :--- |
| `HasScopesModel` | Basic search scopes (`IsCompany`, `Departments`, `IsActive`, `IsPublished`, `WhereDocumentType`, ...). |
| `HasDefault` | Default record management (`getDefault`, `makeDefault`). |
| `HasOwnerScopes` / `HasOwnerOptions` | Owner filtering scopes and options (`owner`). |
| `HasVerifierScopes` / `HasVerifierOptions` | Verifier filtering scopes and options (`verifier`). |
| `HasSubjectScopes` / `HasSubjectOptions` | Subject filtering scopes and options (`subject`). |
| `HasRecordsOptions` | Integrated `getRecords()` function for fetching records with flexible filters and multiple output formats (Collection, paginator, query). |
| `ListObjects` / `ListOptions` | Helper functions for fetching cached records and dropdown lists. |
| `FieldsOptions` | Dropdown option functions for forms (`getStatusOptions`, `getDocumentTypeOptions`, ...). |
| `HasCreateDefaultRecords` | Functions for creating default records (for compatibility with other plugin structures). |

##### 3. Integrated `getRecords` Function

The `getRecords` function in the `Document` model supports comprehensive filtering options:
- `id`, `document_type`, `document_category`, `document_number`, `full_name`, `company_name`, `status`, `is_verified`, `is_active`, `is_published`
- `owner_id`/`owner_type`, `verifier_id`/`verifier_type`, `subject_id`/`subject_type`
- Date ranges (`issue_date`, `expiry_date`, `created_at`, `updated_at`)
- Text search (`q`)
- Support for `is_paginator`, `is_collection`, `is_query`, `is_first`

#### Benefits

- Centralised definition of all document types that is easy to maintain and extend.
- Dynamic fields allow building adaptive input forms based on the document type.
- Automatic data mapping between input and database.
- Powerful `Document` model with ready‑to‑use scopes for lists and filters.
- Caching to improve performance in `ListObjects` and `ListOptions`.

---

### Version 1.0.3 – Development of the `Manager` Class (Operations Manager)

#### Version Goals

- Develop the `Manager` class (`Nano3\Kyc\Classes\Manager`) to be responsible for all document management operations.
- Provide unified functions for creating, updating, deleting, approving, and rejecting documents with support for transactions, events, and testing.

#### New Features

##### 1. `Manager` Class – KYC Operations Manager

The `Manager` class provides the following functions:

| Function | Description |
| :--- | :--- |
| `createDocument(array $options, bool $is_test_create)` | Create a new document with validation, duplicate checking, and events. |
| `updateDocument($documentId, array $options, bool $is_test_create)` | Update an existing document. |
| `deleteDocument($documentId, bool $is_test_create)` | Delete a document (soft delete). |
| `restoreDocument($documentId)` | Restore a soft‑deleted document. |
| `verifyDocument($documentId, $verifier, $verification_score)` | Verify/approve a document (update status to `verified`). |
| `rejectDocument($documentId, $reason)` | Reject a document with an optional reason. |
| `checkDuplicateDocument(array $options)` | Check for an existing duplicate document for the same owner. |
| `getDocumentRecords($options)` | Fetch document records using the model’s `getRecords` function. |
| `getDocumentStats($options)` | Fetch document statistics. |

##### 2. Unified Response Structure

All `Manager` functions return a unified response structure:

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'Success message',
    'error' => null,
    'errors' => null,
    'model' => $document,
    'data' => $document->toArray(),
    'input_data' => [...],
    'process_data' => [...]
]
```

##### 3. Testing Support (`is_test_create`)

All functions support the `is_test_create` parameter, which allows executing the operation inside a transaction that is rolled back after verifying success. This makes it easy to test functionality without affecting real data.

##### 4. Event Support and Control

- Event firing can be disabled via the `is_stop_event` option.
- Supported events: `nano3.kyc.document.created`, `updated`, `deleted`, `restored`, `verified`, `rejected`.

##### 5. Duplicate Checking

The `checkDuplicateDocument` function checks for an existing matching document based on customisable fields (`document_number`, `owner_id`, `owner_type`, `document_type`).

##### 6. Helper Functions for Compatibility

- `getQueryDate()`: apply flexible date filtering to queries.
- `checkValueIsNotAll()`: check that a value is not a wildcard (`*` or `all`).
- `scopeWhereField()`: generic scope to apply `WHERE` conditions with support for `is_or`, `is_not`, and `is_force`.

#### Benefits

- Unified interface for all document management operations.
- High security with permission and duplicate checking.
- Full transaction support ensures data integrity.
- Easy testing thanks to `is_test_create`.
- Flexibility to customise operation behaviour via events.

---

### Version 1.0.4 – Development of API Controller and Backend Controller

#### Version Goals

- Develop the API controller (`Nano3\Kyc\APIControllers\Documents`) to provide RESTful endpoints for document management.
- Develop the backend controller (`Nano3\Kyc\Controllers\Documents`) to manage documents from the dashboard.

#### New Features

##### 1. API Controller (`Documents`)

The controller provides the following endpoints (base path: `/api/v1/kyc`):

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/documents` | `GET` | List documents with filtering, searching, and sorting. |
| `/documents` | `POST` | Create a new document. |
| `/documents/{id}` | `GET` | View document details. |
| `/documents/{id}` | `PUT` | Update a document. |
| `/documents/{id}` | `DELETE` | Delete a document. |
| `/documents/{id}/verify` | `POST` | Verify/approve a document. |
| `/documents/{id}/reject` | `POST` | Reject a document. |
| `/documents/{id}/restore` | `POST` | Restore a soft‑deleted document. |
| `/document-types` | `GET` | Fetch supported document types (flat or grouped). |
| `/document-types/{id}` | `GET` | Fetch details of a specific document type. |
| `/document-fields/{type}` | `GET` | Fetch fields of a document type (for building dynamic forms). |
| `/documents/stats` | `GET` | Fetch quick statistics about documents. |
| `/documents/activelystats` | `GET` | Check for new updates (for caching purposes). |

**API Controller Features:**
- Uses `Manager` to execute operations.
- Supports caching to improve performance.
- Unified response structure compatible with `Nano.API`.
- Professional error handling with localisable messages.

##### 2. Data Transformer `DocumentTransformer`

The `DocumentTransformer` (`Nano3\Kyc\Transformers\DocumentTransformer`) was developed to format API responses, with support for:
- Including relationships (`owner`, `verifier`, `subject`, `company`, `department`).
- Formatting dates and statuses.
- Handling JSON fields (`fields_data`, `metadata`).
- Excluding fields on demand (`exclude`).

##### 3. Backend Controller (`Documents`)

The controller provides complete document management from the dashboard:
- **Documents list**: with search, filtering, sorting, and bulk actions.
- **Create/Edit form**: divided into organised tabs (Basic, Owner, Verification, Physical Copy, Files, Publishing, Relations, Audit).
- **Advanced filters**: via `config_filter.yaml` covering all important table fields.
- **Customisable columns**: via `columns.yaml`.

##### 4. Backend Configuration Files

| File | Description |
| :--- | :--- |
| `config_filter.yaml` | Defines list filters (document type, status, owner, dates, ...). |
| `columns.yaml` | Defines list columns with search and sort capabilities. |
| `fields.yaml` | Defines input form fields, divided into tabs. |

#### Benefits

- Ready‑to‑use API for integration with mobile and web applications.
- Easy‑to‑use backend interface for manual document management.
- Unified data formatting via `DocumentTransformer`.
- Customisable filters and columns to fit each project’s needs.

---

### Version 1.0.5 – Improvements to Translation Files, Lists, and Forms

#### Version Goals

- Complete the Arabic translation file `lang.php` to include all keys used in controllers, lists, and forms.
- Improve `columns.yaml` and `fields.yaml` to provide an enhanced user experience in the backend.

#### New Features

##### 1. Comprehensive Translation File `lang.php`

A complete Arabic translation file was prepared containing:

- **Icons**: menu and status icons.
- **Menus**: main and sub‑menu titles with descriptions.
- **Permissions**: permission keys and texts for all operations.
- **Messages**: success, error, and warning messages for all operations.
- **Controller translations**: translations specific to the `Documents` controller including tabs, buttons, messages, filters, form fields, and list columns.
- **Document type translations**: categories, types, descriptions, and fields for each document type.

##### 2. Improved `columns.yaml`

The list columns file was enhanced to include:
- **Visible by default columns**: `id`, `document_type`, `document_number`, `full_name`, `company_name`, `owner`, `status`, `is_verified`, `issue_date`, `expiry_date`, `created_at`.
- **Hidden but showable columns**: `code`, `document_category`, `verifier`, `verification_score`, `is_physical_submitted`, `departments_id`, `subject_name`, `updated_at`.
- **Custom formatting**: using `formatter` to display document type name, owner, and verifier appropriately.
- **Search and sort support** for most columns.

##### 3. Improved `fields.yaml`

The input form fields were organised into tabs:

| Tab | Fields |
| :--- | :--- |
| **Basic** | `document_category`, `document_type`, `document_number`, `issuing_authority`, `issue_date`, `expiry_date` |
| **Owner** | `owner_type`, `owner_id` (with `@create` and `@update` to control editability) |
| **Verification** | `verifier_type`, `verifier_id`, `is_verified`, `verification_score`, `verified_at` |
| **Physical Copy** | `is_physical_submitted`, `physical_copy_type`, `physical_received_at`, `physical_returned_at`, `physical_notes` |
| **Files** | `document_front`, `document_back`, `signature_image`, `image`, `files` |
| **Publishing & Status** | `status`, `is_published`, `published_at`, `unpublished_at`, `is_active`, `is_default`, `is_public` |
| **Additional Relations** | `subject_type`, `subject_id`, `extend_id`, `departments_id` |
| **Audit** | `created_by`, `updated_by`, `created_at`, `updated_at` (read‑only) |

**Additional Features:**
- Using `dependsOn` to link dropdowns (e.g. `owner_id` depends on `owner_type`).
- Using `trigger` to show/hide `published_at` and `unpublished_at` fields based on `is_published` status.
- Responsive layout using `span` and `cssClass`.

#### Benefits

- Fully localised backend ready for use.
- Logical organisation of fields that simplifies data entry.
- Smooth user experience with dependent dropdowns and conditional fields.

---

### Version 1.0.6 – Final Improvements and Release Preparation

#### Version Goals

- Make final improvements to the code and structure.
- Prepare `composer.json` and `README.md` for documentation and distribution.
- Standardise outputs and ensure the plugin is ready for production use.

#### New Features

##### 1. Improvements to `Manager` and `DocumentType`

- Improved `createDocument` function to handle files and fields separately.
- Added `prepareDocumentOptions` function to prepare default company and branch values.
- Improved error handling and added debug details in development environment.

##### 2. `composer.json` File

The `composer.json` file was created with the following settings:
- **Name**: `nano3/kyc`
- **Type**: `nano-extension` (specific to NanoSoft App)
- **Dependencies**: `php >= 8.0`, `nano/api`, `tss/basic`
- **License**: `proprietary`

##### 3. `README.md` File

A professional `README.md` file was prepared containing:
- General description of the plugin and its features.
- List of supported document types.
- System requirements and installation instructions.
- Usage examples for the API, `Manager`, and `DocumentType` classes.
- Explanation of the database structure.
- Licensing and support information.

##### 4. Update Documentation

This update documentation file (`Update-Nano3-Kyc-en.md`) was prepared to cover all versions from 1.0.1 to 1.0.6 in a professional format compatible with NanoSoft App documentation standards.

#### Benefits

- Plugin ready for distribution and use in production projects.
- Comprehensive documentation that makes it easy for developers to understand and use the plugin.
- Integrated structure that supports future maintenance and development.

---

### Summary of Versions (1.0.1 – 1.0.6)

| Version | Key Features |
| :--- | :--- |
| 1.0.1 | Created the `nano3_kyc_documents` table and the basic plugin structure. |
| 1.0.2 | Developed the `DocumentType` class and the `Document` model with core traits. |
| 1.0.3 | Developed the `Manager` class (operations manager) with transaction, event, and testing support. |
| 1.0.4 | Developed the API controller and backend controller with configuration files. |
| 1.0.5 | Improved translation files, lists, and forms (`lang.php`, `columns.yaml`, `fields.yaml`). |
| 1.0.6 | Final improvements, prepared `composer.json` and `README.md`, and documented updates. |

---

### Upgrade Requirements

1. **Run migrations**:
   ```bash
   php artisan nano:up
   ```
   Or execute the `create_table_nano3_kyc_documents.php` migration manually.

2. **Publish configuration file** (optional):
   ```bash
   php artisan config:publish nano3.kyc
   ```
   To customise document type properties via `config/nano3/kyc/document_types.php`.

3. **Set up permissions**:
   - Grant appropriate permissions to roles (`documents.access`, `documents.verify`, etc.) from the dashboard.

4. **Update API definitions** (if using `Nano.API`):
   - Ensure the API routes are included in the main routing file.

---

### Conclusion

The `Nano3.Kyc` plugin has been developed as a complete, flexible solution for managing identity verification processes within NanoSoft App. Thanks to its extensible structure, support for over 28 document types, integrated API, and easy‑to‑use admin interface, developers can quickly integrate it into any project that requires user or company identity verification.

We look forward to continuing to develop the plugin based on user feedback and evolving requirements, with future plans including:
- OCR support for automatic data extraction.
- Integration with external document verification services.
- Advanced analytics dashboards.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [DocumentType Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [Manager Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [Document Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)
