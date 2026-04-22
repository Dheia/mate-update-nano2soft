# Document Model Documentation – KYC Documents Model

## Overview

The `Document` model is located at `Nano3\Kyc\Models\Document`. It is an Eloquent model responsible for representing and managing all identity verification (KYC) documents in the database. The model is associated with the table `nano3_kyc_documents` and uses a rich set of traits to provide advanced features such as:

- Search and filter scopes.
- Default record management.
- Dropdown options for forms and filters.
- Integrated `getRecords` function for fetching records with flexible filtering.
- Caching for objects and lists.
- Polymorphic relationships with owner, verifier, and associated subject.

---

## Table Structure `nano3_kyc_documents`

| Field Group | Main Fields |
| :--- | :--- |
| **Identifiers** | `id`, `code`, `barcode` |
| **Document Type** | `document_type`, `document_category` |
| **Basic Data** | `document_number`, `issuing_authority`, `issue_date`, `expiry_date` |
| **Personal Data** | `full_name`, `date_of_birth`, `nationality`, `country_of_issue`, `religion` |
| **Address** | `address_line1`, `address_line2`, `address_city`, `address_state`, `address_postcode`, `address_country` |
| **Corporate Data** | `company_name`, `registration_number`, `tax_number`, `incorporation_date` |
| **Relationships** | `owner_id`/`owner_type`, `verifier_id`/`verifier_type`, `subject_id`/`subject_type` |
| **Verification & Status** | `is_verified`, `verified_at`, `verification_score`, `status` |
| **Physical Copies** | `is_physical_submitted`, `physical_copy_type`, `physical_received_at`, `physical_returned_at`, `physical_notes` |
| **Publishing** | `is_published`, `published_at`, `unpublished_at`, `is_public`, `is_active`, `is_default` |
| **Flexible Data** | `fields_data`, `metadata`, `other_data`, `config_data` (JSON) |
| **Files** | `file_ids`, `file_urls` |
| **Tracking** | `created_by`, `updated_by`, `deleted_by`, `timestamps`, `softDeletes` |
| **Multi‑company** | `companys_id`, `departments_id` |

---

## Traits Used in the Model

The `Document` model uses the following traits to organise code and add functionality:

| Trait | Description |
| :--- | :--- |
| `HasScopesModel` | Basic search and filter scopes (e.g. `isActive`, `whereDocumentType`). |
| `HasDefault` | Default record management (`getDefault`, `makeDefault`). |
| `HasUserScopes` / `HasUserOptions` | User filter scopes and options (for compatibility with other plugins). |
| `HasSubjectScopes` / `HasSubjectOptions` | Subject filter scopes and options (`subject`). |
| `HasOwnerScopes` / `HasOwnerOptions` | Owner filter scopes and options (`owner`). |
| `HasVerifierScopes` / `HasVerifierOptions` | Verifier filter scopes and options (`verifier`). |
| `HasRecordsOptions` | Integrated `getRecords` function for fetching records with comprehensive filtering. |
| `ListObjects` | Caching functions for objects (`getRecordObj`, `clearCacheObjectKeyModel`). |
| `ListOptions` | Helper functions for fetching dropdown lists (e.g. `listAvailableDepartments`). |
| `FieldsOptions` | Dropdown option functions for backend forms (`getStatusOptions`, `getDocumentTypeOptions`). |

---

## Trait `HasScopesModel` – Basic Search Scopes

This trait provides a set of ready‑to‑use scopes for Eloquent queries.

### Available Scopes

| Scope | Description | Usage Example |
| :--- | :--- | :--- |
| `IsCompany($companys_id)` | Filter by company. | `Document::IsCompany(5)->get()` |
| `IsDepartment($departments_id)` | Filter by branch (supports main and sub‑branches). | `Document::IsDepartment(10)->get()` |
| `Departments($departments_id)` | Flexible branch filtering. | `Document::Departments(10)->get()` |
| `IsActive()` | Only active records (`is_active = true`). | `Document::IsActive()->get()` |
| `IsNotActive()` | Only inactive records. | `Document::IsNotActive()->get()` |
| `IsPublished()` | Published records that fall within publication date range. | `Document::IsPublished()->get()` |
| `IsNotPublished()` | Unpublished records or outside date range. | `Document::IsNotPublished()->get()` |
| `WhereDocumentType($value, $is_or, $is_not, $is_force)` | Filter by document type. | `Document::WhereDocumentType('passport')->get()` |
| `WhereDocumentCategory($value, ...)` | Filter by document category. | `Document::WhereDocumentCategory('primary_id')->get()` |
| `WhereDocumentNumber($value, ...)` | Filter by document number. | `Document::WhereDocumentNumber('A123')->get()` |
| `WhereFullName($value, ...)` | Filter by full name. | `Document::WhereFullName('John')->get()` |
| `WhereCompanyName($value, ...)` | Filter by company name. | `Document::WhereCompanyName('Nano')->get()` |
| `WhereStatus($value, ...)` | Filter by status. | `Document::WhereStatus('pending')->get()` |
| `WhereIsVerified($value)` | Filter by verification status. | `Document::WhereIsVerified(true)->get()` |
| `IsExpired()` | Expired documents (`expiry_date < NOW()`). | `Document::IsExpired()->get()` |
| `IsExpiringSoon($days)` | Documents that will expire within a given number of days. | `Document::IsExpiringSoon(30)->get()` |

**Note:** Scopes with parameters `$is_or`, `$is_not`, `$is_force` allow fine‑grained control over the condition behaviour. `$is_force` forces the condition even if the value is `*` or `all`.

---

## Trait `HasRecordsOptions` – Integrated `getRecords` Function

The `getRecords` function is one of the most powerful tools in the model. It allows fetching records with comprehensive filtering options and multiple output formats.

### Function Signature

```php
public static function getRecords(array $options = [], bool $isException = false): array
```

### Response Structure

The function returns an array with the following structure:

```php
[
    'code' => 200,
    'status' => true,
    'message' => '...',
    'error' => null,
    'errors' => null,
    'data' => $result, // Could be Collection, Paginator, or Query Builder
    'input_data' => [...],
    'process_data' => [...]
]
```

### Supported Options (`$options`)

| Key | Type | Description |
| :--- | :--- | :--- |
| `id` | `mixed` | Filter by ID. |
| `is_force_id` | `bool` | Force `id` filter even if value is `*`. |
| `companys_id` | `string` | Filter by company. |
| `departments_id` | `string` | Filter by branch. |
| `document_type` | `mixed` | Filter by document type. |
| `is_or_document_type` | `bool` | Use `OR` for the `document_type` condition. |
| `is_not_document_type` | `bool` | Negate the `document_type` condition. |
| `is_force_document_type` | `bool` | Force the filter. |
| `document_category` | `mixed` | Filter by document category (supports `is_or`, `is_not`, `is_force`). |
| `document_number` | `mixed` | Filter by document number. |
| `full_name` | `mixed` | Filter by full name. |
| `company_name` | `mixed` | Filter by company name. |
| `status` | `mixed` | Filter by status. |
| `is_verified` | `mixed` | Filter by verification status (can be `true`, `false`, or `'*'`). |
| `is_active` | `mixed` | Filter by active status. |
| `is_default` | `mixed` | Filter by default flag. |
| `is_public` | `mixed` | Filter by public flag. |
| `is_published` | `mixed` | Filter by published flag. |
| `owner` / `owner_id` / `owner_type` | `mixed` | Filter by owner (supports object or ID + type). |
| `verifier` / `verifier_id` / `verifier_type` | `mixed` | Filter by verifier. |
| `subject` / `subject_id` / `subject_type` | `mixed` | Filter by associated subject. |
| `issue_date` / `expiry_date` | `string\|array` | Filter by issue/expiry date (supports range). |
| `q` | `string` | Text search in (`document_number`, `full_name`, `company_name`, `code`). |
| `orderBy` | `string` | Column name for ordering (default: `created_at`). |
| `orderDirection` | `string` | Order direction (`asc` or `desc`). |
| `is_paginator` | `bool` | Return a `Paginator` instead of a `Collection`. |
| `is_collection` | `bool` | Return a `Collection`. |
| `is_query` | `bool` | Return a `Builder` instance. |
| `is_first` | `bool` | Return only the first record. |
| `per_page` | `int` | Number of records per page (when paginating). |
| `page` | `int` | Page number. |
| `withTrashed` | `bool` | Include soft‑deleted records. |
| `onlyTrashed` | `bool` | Only soft‑deleted records. |
| `exclude` | `string` | Comma‑separated column names to exclude from results. |

### Examples of Using `getRecords`

#### 1. Fetch all active passport documents

```php
$result = Document::getRecords([
    'document_type' => 'passport',
    'is_active' => true,
    'is_paginator' => true,
    'per_page' => 20,
]);

if ($result['status']) {
    $documents = $result['data']; // LengthAwarePaginator
    foreach ($documents as $doc) {
        echo $doc->document_number;
    }
}
```

#### 2. Search for documents for a specific user with date filtering

```php
$result = Document::getRecords([
    'owner' => $user, // user object
    'issue_date' => ['2024-01-01', '2024-12-31'],
    'status' => 'verified',
    'orderBy' => 'created_at',
    'orderDirection' => 'desc',
]);

$documents = $result['data']; // Collection
```

#### 3. Get a query builder for further modification

```php
$result = Document::getRecords([
    'is_verified' => false,
    'is_query' => true,
]);

if ($result['status']) {
    $query = $result['data']; // Builder
    $query->where('verification_score', '>', 50);
    $docs = $query->get();
}
```

#### 4. Use text search

```php
$result = Document::getRecords([
    'q' => 'NanoSoft',
    'is_paginator' => true,
]);

// Searches in document_number, full_name, company_name, code
```

---

## Traits `HasOwnerScopes` and `HasOwnerOptions`

### `HasOwnerScopes` – Owner Scopes

This trait provides advanced scopes for filtering by owner.

| Scope | Description |
| :--- | :--- |
| `WhereOwnerType($value, $is_or, $is_not, $is_force)` | Filter by owner type. |
| `WhereOwnerId($value, ...)` | Filter by owner ID. |
| `ByOwner($query, ...$params)` | **Advanced scope** that accepts an object or detailed options array. |

#### Advanced Scope `byOwner`

It can be called in several ways:

```php
// 1. Pass an object directly
Document::byOwner($user)->get();

// 2. Pass a detailed options array
Document::byOwner([
    'owner_type' => ['value' => 'RainLab\User\Models\User', 'is_force' => true],
    'owner_id'   => ['value' => 5, 'is_or' => true],
    'global_is_or' => false,
    'logic_between' => 'OR',
])->get();

// 3. Separate parameters (for compatibility)
Document::byOwner($user, null, null, false, false, true)->get();
```

`byOwner` supports the following properties at field or group level:

- `is_or`: makes the condition `OR` instead of `AND`.
- `is_not`: negates the condition.
- `is_force`: forces the condition even if the value is `*` or `all`.
- `logic_between`: relationship between `owner_type` and `owner_id` (`AND` or `OR`).

Other scopes:

- `WhereNotNullOwner()` / `WhereNullOwner()`: filter records that have an owner or do not have one.
- `WhereHasOwnerTrashed()`: fetch records whose owner has been soft‑deleted.

### `HasOwnerOptions` – Owner Options

Provides functions to generate dropdown options for forms and filters:

| Function | Description |
| :--- | :--- |
| `getOwnerTypeOptions()` | List of owner types registered in the system. |
| `getOwnerIdOptions()` | List of owner IDs (depends on `owner_type`). |
| `getOwnerTypeFilterOptions($scopes)` | Owner type filter options. |
| `getOwnerIdFilterOptions($scopes)` | Owner ID filter options. |
| `getOwnerNameAttribute()` | Accessor to return the owner's name. |

**Example in `fields.yaml`:**

```yaml
owner_type:
    label: Owner Type
    type: dropdown
    options: getOwnerTypeOptions

owner_id:
    label: Owner
    type: dropdown
    options: getOwnerIdOptions
    dependsOn: owner_type
```

---

## Remaining Traits in Brief

### `HasSubjectScopes` / `HasSubjectOptions`

Scopes and options similar to `Owner` but for the `subject` relationship. Supports the same advanced options in `bySubject`.

### `HasVerifierScopes` / `HasVerifierOptions`

Scopes and options for the `verifier` relationship. Used to filter by who verified the document.

### `HasDefault`

Manages the default record per branch. Main functions:

- `getDefault($departments_id)`: fetch the default document.
- `makeDefault()`: set the current document as default (and unset default on other documents of the same type and owner).

### `ListObjects`

Caching functions for objects to improve performance:

- `getRecordObj($key)`: fetch a document object from cache or database (supports `id` or `code`).
- `clearCacheObjectKeyModel($key)`: clear cache for a specific record.
- `clearCacheListObjectKeyByDepartments()`: clear cache for all records of a given branch.

### `ListOptions`

Helper functions to fetch dropdown lists (for companies and branches) with caching:

- `listAvailableCompanys()` / `listEnabledCompanys()`
- `listAvailableDepartments()` / `listEnabledDepartments()`
- `clearCacheListByCompanys()` / `clearCacheListByDepartments()`

### `FieldsOptions`

Functions to generate dropdown options used in backend forms:

- `getStatusOptions()`
- `getDocumentTypeOptions()`
- `getDocumentCategoryOptions()`
- `getPhysicalCopyTypeOptions()`
- `getDepartmentsIdOptions()`
- `getPartyTypeOptions()`

---

## Model Events

The model is associated with the following events during its lifecycle:

| Event | Description |
| :--- | :--- |
| `beforeValidate` | Before data validation. |
| `beforeSave` | Before saving. Sets `companys_id` and `departments_id`, and updates `verified_at` when `is_verified` changes. |
| `beforeCreate` | Before creation. Sets `created_by`. |
| `beforeUpdate` | Before update. Sets `updated_by`. |
| `beforeDelete` | Before deletion. Sets `deleted_by`. |
| `afterSave` | After saving. Updates cache and manages default record. |
| `afterDelete` | After deletion. |

---

## Useful Accessors

| Accessor | Description |
| :--- | :--- |
| `getDocumentTypeNameAttribute()` | Returns the localized name of the document type. |
| `getStatusColorAttribute()` | Bootstrap colour appropriate for the status. |
| `getStatusIconAttribute()` | Font Awesome icon appropriate for the status. |
| `isExpired()` | Whether the document has expired. |
| `isExpiringSoon($days)` | Whether it will expire within `$days` days. |

---

## Complete Example: Create, Update, and Retrieve a Document

```php
use Nano3\Kyc\Models\Document;
use Nano3\Kyc\Classes\Manager;

// Create a document
$result = Manager::createDocument([
    'document_type' => 'national_id',
    'owner'         => Auth::getUser(),
    'fields'        => [
        'document_number' => '987654321',
        'full_name'       => 'Ahmed Mohamed',
        'issue_date'      => '2022-05-10',
        'expiry_date'     => '2032-05-10',
    ],
    'files' => ['document_front' => $uploadedFile],
]);

if ($result['status']) {
    $document = $result['model'];
}

// Fetch active documents for a specific user
$docs = Document::byOwner(Auth::getUser())
    ->IsActive()
    ->orderBy('created_at', 'desc')
    ->get();

// Use getRecords with advanced filtering
$result = Document::getRecords([
    'document_type' => 'passport',
    'is_verified' => true,
    'expiry_date' => ['2025-01-01', '2025-12-31'],
    'is_paginator' => true,
]);
```

---

## Dependencies

- `Tss\Basic\Helpers\BasicHelper` – for company and branch management.
- `Nano3\Kyc\Classes\DocumentType` – for obtaining document type information.
- `Nano3\Kyc\Classes\Manager` – for some helper functions (`scopeWhereField`, `getQueryDate`).

---
