# `Document` Model Documentation – KYC Documents Model

## Overview

The `Document` model is located at `Nano3\Kyc\Models\Document` and is the Eloquent model responsible for representing and managing all Know Your Customer (KYC) documents in the database. The model is associated with the `nano3_kyc_documents` table and uses a rich set of traits to provide advanced features such as:

- Advanced search and filtering scopes.
- Default record management (`HasDefault`).
- Dropdown list options for forms and filters.
- Integrated `getRecords` function for fetching records with comprehensive filtering.
- Caching for objects and lists (`ListObjects`, `ListOptions`).
- Polymorphic relationships with owner (`owner`), verifier (`verifier`), and associated subject (`subject`).
- **Protect Verified Documents** system with flexible permissions.
- **Duplicate Prevention** mechanism for the same owner.
- **Automatic verification processing** (auto-set verifier, verification score, status).

---

## `nano3_kyc_documents` Table Structure

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
| **Hard Copies** | `is_physical_submitted`, `physical_copy_type`, `physical_received_at`, `physical_returned_at`, `physical_notes` |
| **Publishing** | `is_published`, `published_at`, `unpublished_at`, `is_public`, `is_active`, `is_default` |
| **Flexible Data** | `fields_data`, `metadata`, `other_data`, `config_data` (JSON) |
| **Files** | `file_ids`, `file_urls` |
| **Tracking** | `created_by`, `updated_by`, `deleted_by`, `timestamps`, `softDeletes` |
| **Multi-Company** | `companys_id`, `departments_id` |

---

## Traits Used in the Model

The `Document` model uses the following traits to organize code and add functionality:

| Trait | Description |
| :--- | :--- |
| `HasScopesModel` | Basic search and filtering scopes (e.g., `isActive`, `whereDocumentType`). |
| `HasDefault` | Default record management (`getDefault`, `makeDefault`). |
| `HasUserScopes` / `HasUserOptions` | User filtering scopes and options (for compatibility with other add-ons). |
| `HasSubjectScopes` / `HasSubjectOptions` | Associated subject filtering scopes and options (`subject`). |
| `HasOwnerScopes` / `HasOwnerOptions` | Owner filtering scopes and options (`owner`). |
| `HasVerifierScopes` / `HasVerifierOptions` | Verifier filtering scopes and options (`verifier`). |
| `HasRecordsOptions` | Integrated `getRecords` function for fetching records with comprehensive filtering. |
| `ListObjects` | Object caching functions (`getRecordObj`, `clearCacheObjectKeyModel`). |
| `ListOptions` | Helper functions for fetching dropdown lists (e.g., `listAvailableDepartments`). |
| `FieldsOptions` | Dropdown list option functions for backend forms (`getStatusOptions`, `getDocumentTypeOptions`). |

---

## `HasScopesModel` Trait – Basic Search Scopes

This trait provides a set of ready-to-use scopes for Eloquent queries.

### Available Scopes

| Scope | Description | Usage Example |
| :--- | :--- | :--- |
| `IsCompany($companys_id)` | Filter by company. | `Document::IsCompany(5)->get()` |
| `IsDepartment($departments_id)` | Filter by department (supports parent and sub-departments). | `Document::IsDepartment(10)->get()` |
| `Departments($departments_id)` | Flexible department filtering. | `Document::Departments(10)->get()` |
| `IsActive()` | Active records only (`is_active = true`). | `Document::IsActive()->get()` |
| `IsNotActive()` | Inactive records. | `Document::IsNotActive()->get()` |
| `IsPublished()` | Published records within the publication date range. | `Document::IsPublished()->get()` |
| `IsNotPublished()` | Unpublished or out-of-range records. | `Document::IsNotPublished()->get()` |
| `WhereDocumentType($value, $is_or, $is_not, $is_force)` | Filter by document type. | `Document::WhereDocumentType('passport')->get()` |
| `WhereDocumentCategory($value, ...)` | Filter by document category. | `Document::WhereDocumentCategory('primary_id')->get()` |
| `WhereDocumentNumber($value, ...)` | Filter by document number. | `Document::WhereDocumentNumber('A123')->get()` |
| `WhereFullName($value, ...)` | Filter by full name. | `Document::WhereFullName('John')->get()` |
| `WhereCompanyName($value, ...)` | Filter by company name. | `Document::WhereCompanyName('Nano')->get()` |
| `WhereStatus($value, ...)` | Filter by status. | `Document::WhereStatus('pending')->get()` |
| `WhereIsVerified($value)` | Filter by verification status. | `Document::WhereIsVerified(true)->get()` |
| `IsExpired()` | Expired documents (`expiry_date < NOW()`). | `Document::IsExpired()->get()` |
| `IsExpiringSoon($days)` | Documents expiring within a specified number of days. | `Document::IsExpiringSoon(30)->get()` |

**Note:** Scopes with `$is_or`, `$is_not`, `$is_force` parameters allow fine-grained control over the condition behavior. `$is_force` forces the condition even if the value is `*` or `all`.

---

## `HasRecordsOptions` Trait – Integrated `getRecords` Function

The `getRecords` function is one of the most powerful tools in the model, allowing record retrieval with comprehensive filtering options and multiple output formats.

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
| `is_force_id` | `bool` | Force `id` filtering even if `*`. |
| `companys_id` | `string` | Filter by company. |
| `departments_id` | `string` | Filter by department. |
| `document_type` | `mixed` | Filter by document type. |
| `is_or_document_type` | `bool` | Use `OR` for `document_type` condition. |
| `is_not_document_type` | `bool` | Negate `document_type` condition. |
| `is_force_document_type` | `bool` | Force filtering. |
| `document_category` | `mixed` | Filter by document category (supports `is_or`, `is_not`, `is_force`). |
| `document_number` | `mixed` | Filter by document number. |
| `full_name` | `mixed` | Filter by full name. |
| `company_name` | `mixed` | Filter by company name. |
| `status` | `mixed` | Filter by status. |
| `is_verified` | `mixed` | Filter by verification status (can be `true`, `false`, `'*'`). |
| `is_active` | `mixed` | Filter by active status. |
| `is_default` | `mixed` | Filter by default status. |
| `is_public` | `mixed` | Filter by public status. |
| `is_published` | `mixed` | Filter by published status. |
| `owner` / `owner_id` / `owner_type` | `mixed` | Filter by owner (supports object or ID and type). |
| `verifier` / `verifier_id` / `verifier_type` | `mixed` | Filter by verifier. |
| `subject` / `subject_id` / `subject_type` | `mixed` | Filter by associated subject. |
| `issue_date` / `expiry_date` | `string\|array` | Filter by issue/expiry date (supports range). |
| `q` | `string` | Text search in (`document_number`, `full_name`, `company_name`, `code`). |
| `orderBy` | `string` | Column name for ordering (default: `created_at`). |
| `orderDirection` | `string` | Order direction (`asc` or `desc`). |
| `is_paginator` | `bool` | Return `Paginator` instead of `Collection`. |
| `is_collection` | `bool` | Return `Collection`. |
| `is_query` | `bool` | Return `Builder` object for further query modification. |
| `is_first` | `bool` | Return only the first record. |
| `per_page` | `int` | Number of records per page (when paginating). |
| `page` | `int` | Page number. |
| `withTrashed` | `bool` | Include soft-deleted records. |
| `onlyTrashed` | `bool` | Only soft-deleted records. |
| `exclude` | `string` | Comma-separated column names to exclude from results. |

### Examples of Using `getRecords`

#### 1. Fetch all active documents of type "passport"

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

#### 3. Fetch a ready-to-modify query

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

## `HasOwnerScopes` and `HasOwnerOptions` Traits

### `HasOwnerScopes` – Owner Scopes

This trait provides advanced scopes for filtering by owner (`owner`).

| Scope | Description |
| :--- | :--- |
| `WhereOwnerType($value, $is_or, $is_not, $is_force)` | Filter by owner type. |
| `WhereOwnerId($value, ...)` | Filter by owner ID. |
| `ByOwner($query, ...$params)` | **Advanced scope** accepts an object or detailed options array. |

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

`byOwner` supports the following properties at the field or group level:

- `is_or`: Makes the condition `OR` instead of `AND`.
- `is_not`: Negates the condition.
- `is_force`: Forces the condition even if the value is `*` or `all`.
- `logic_between`: Relationship between `owner_type` and `owner_id` (`AND` or `OR`).

Other scopes:

- `WhereNotNullOwner()` / `WhereNullOwner()`: Filter records that have an owner or do not have one.
- `WhereHasOwnerTrashed()`: Fetch records whose owner has been soft-deleted.

### `HasOwnerOptions` – Owner Options

Provides functions to generate dropdown list options for forms and filters:

| Function | Description |
| :--- | :--- |
| `getOwnerTypeOptions()` | List of registered owner types in the system. |
| `getOwnerIdOptions()` | List of owner IDs (depends on `owner_type`). |
| `getOwnerTypeFilterOptions($scopes)` | Owner type filter options. |
| `getOwnerIdFilterOptions($scopes)` | Owner ID filter options. |
| `getOwnerNameAttribute()` | Accessor to return the owner's name. |

---

## Other Traits (Briefly)

### `HasSubjectScopes` / `HasSubjectOptions`

Scopes and options similar to `Owner` but for the `subject` relationship. Supports the same advanced options in `bySubject`.

### `HasVerifierScopes` / `HasVerifierOptions`

Scopes and options for the `verifier` relationship. Used to filter by who verified the document.

### `HasDefault`

Default record management per department. Main functions:

- `getDefault($departments_id)`: Get the default document.
- `makeDefault()`: Set the current document as default (clearing default from other documents of the same type and owner).

### `ListObjects`

Caching functions for objects to improve performance:

- `getRecordObj($key)`: Get a document object from cache or database (supports `id` or `code`).
- `clearCacheObjectKeyModel($key)`: Clear the cache for a specific record.
- `clearCacheListObjectKeyByDepartments()`: Clear the cache for all records of a specific department.

### `ListOptions`

Helper functions to fetch dropdown lists (for companies and departments) with caching:

- `listAvailableCompanys()` / `listEnabledCompanys()`
- `listAvailableDepartments()` / `listEnabledDepartments()`
- `clearCacheListByCompanys()` / `clearCacheListByDepartments()`

### `FieldsOptions`

Functions to generate dropdown list options used in backend administration forms:

- `getStatusOptions()`
- `getDocumentTypeOptions()`
- `getDocumentCategoryOptions()`
- `getPhysicalCopyTypeOptions()`
- `getDepartmentsIdOptions()`
- `getPartyTypeOptions()`

---

## Protect Verified Documents System – New in 1.2.5

An integrated protection layer has been added to prevent unauthorized modification of verified (`is_verified = true`) and valid documents. This is controlled by a set of settings in `config.php` and the following functions:

### Settings (`config.php`)

```php
'protect_verified' => [
    'is_check' => true,                     // Enable/disable protection entirely
    'allow_owner_modify_verified' => false, // Allow the document owner to modify it
    'backend_permissions' => 'nano3.kyc.documents.edit_verified', // Special backend permission
    'allowed_fields' => [                   // Fields allowed for modification on verified documents
        'verification_score',
        'is_verified',
        'status',
        'metadata',
        'other_data',
        'config_data',
    ],
],
'is_check_duplicate' => true, // Prevent duplicate document types for the same owner
```

### New Functions

| Function | Type | Description |
| :--- | :--- | :--- |
| `canModifyDocument(?Document $doc, $user)` | `public static` | Determines whether the user (or current user) is allowed to modify the document. Considers settings and permissions. |
| `shouldProtectVerifiedDocument(?Document $doc)` | `public static` | Quick check to see if protection should be applied to the document (regardless of user). |
| `isDocumentValid(Document $doc)` | `public static` | Checks document validity (future expiry date, within maximum age). |
| `checkBackendPermissions($user, $permissions)` | `protected static` | Checks backend user permissions using the OctoberCMS permission system. |
| `protectVerifiedDocument()` | `protected` | Applies protection during `beforeSave`, preventing unauthorized modification. |
| `prepareDuplicate()` | `public` | Prevents duplicate document types for the same owner (if `is_check_duplicate` is enabled). |
| `prepareChangesIsVerified()` | `public` | Handles `is_verified` changes, automatically calculates `verification_score`, and sets `verified_at`. |
| `preperSetVerifier($user)` | `public` | Automatically sets `verifier_id` and `verifier_type` from the current user upon verification. |

### How It Works

1. When attempting to save (update) a document, `protectVerifiedDocument()` is called within `beforeSave`.
2. The function checks `canModifyDocument()` to determine if the current user is allowed to modify.
3. If allowed, full modification is permitted.
4. If disallowed (frontend user or owner without permission), modification is completely blocked or only allowed for the fields specified in `allowed_fields` (depending on user type).

---

## Model Events

| Event | Description |
| :--- | :--- |
| `beforeValidate` | Before data validation. |
| `beforeSave` | Before saving. Sets `companys_id` and `departments_id`, checks for duplicates, processes verification, and applies protection. |
| `beforeCreate` | Before creation. Sets `created_by`. |
| `beforeUpdate` | Before update. Sets `updated_by`. |
| `beforeDelete` | Before deletion. Sets `deleted_by`. |
| `afterSave` | After saving. Updates cache, clears KYC cache, and manages default records. |
| `afterDelete` | After deletion. Clears KYC cache. |

---

## Useful Accessors

| Accessor | Description |
| :--- | :--- |
| `getDocumentTypeNameAttribute()` | Returns the localized name of the document type. |
| `getStatusColorAttribute()` | Bootstrap color appropriate for the status. |
| `getStatusIconAttribute()` | Font Awesome icon appropriate for the status. |
| `isExpired()` | Is the document expired? |
| `isExpiringSoon($days)` | Will it expire within `$days` days? |

---

## Integrated Example: Create, Update, and Fetch a Document

```php
use Nano3\Kyc\Models\Document;
use Nano3\Kyc\Classes\Manager;

// Create a document
$result = Manager::createDocument([
    'document_type' => 'national_id',
    'owner'         => Auth::getUser(),
    'fields'        => [
        'document_number' => '987654321',
        'full_name'       => 'Ahmed Mohammed',
        'issue_date'      => '2022-05-10',
        'expiry_date'     => '2032-05-10',
    ],
    'files' => ['document_front' => $uploadedFile],
]);

if ($result['status']) {
    $document = $result['model'];
}

// Verify the document (verified_at, verifier, verification_score set automatically)
$document->is_verified = true;
$document->save();

// Check modification permission
if (Document::canModifyDocument($document)) {
    $document->full_name = 'Ahmed Mohammed Ali';
    $document->save();
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

- `Tss\Basic\Helpers\BasicHelper` – For managing companies and departments.
- `Nano3\Kyc\Classes\DocumentType` – For obtaining document type information.
- `Nano3\Kyc\Classes\Manager` – For some helper functions (`scopeWhereField`, `getQueryDate`).
- `Nano\AuthApi\Classes\AuthHelpers` – For obtaining the current user.

---

## Additional Documentation

**Reference Documentation**:
- [General Plugin Documentation](./Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./Docs-HasValidKycDocuments-Trait-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
