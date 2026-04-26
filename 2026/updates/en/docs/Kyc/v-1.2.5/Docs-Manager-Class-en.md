# `Manager` Class Documentation – KYC Operations Manager

## Overview

The `Manager` class is located at `Nano3\Kyc\Classes\Manager` and is the beating heart of the `Nano3.Kyc` add-on, the central coordinator for all Know Your Customer (KYC) document management operations within NanoSoft applications. The class serves as an integrated service layer that interacts with the `Document` model and `DocumentType` class to perform core operations such as:

- Creating a new document.
- Updating document data.
- Deleting and restoring a document (Soft Delete).
- Verifying (approving) a document.
- Rejecting a document.
- Duplicate checking.
- Retrieving records and statistics.
- Assessing KYC status for users and companies (via the `HasAssessKycStatus` trait).

The class features several characteristics that make it powerful and flexible:

- **Unified response structure**: All public methods return an array with the same structure (`code`, `status`, `message`, `model`, `data`, etc.), making it easy for consumers (API, controllers, tests) to handle.
- **Transaction support**: All write operations are wrapped in a database transaction to ensure data integrity and allow rollback in case of errors.
- **Test support (`is_test_create`)**: Ability to execute operations within a transaction that is rolled back after verification, facilitating mock testing without polluting the database.
- **Event firing**: Fires events such as `nano3.kyc.document.created` after completing operations, allowing developers to hook custom logic (notifications, synchronization, auditing).
- **Duplicate checking**: Prevents creating duplicate documents for the same owner based on customizable fields.
- **File attachment handling**: Supports attaching files (images, PDFs) to documents via the `attachOne` and `attachMany` relationships defined in the `Document` model.
- **Clean architecture via traits**: KYC assessment logic has been moved to a separate trait `HasAssessKycStatus`, keeping `Manager` focused on document management operations while enabling easy extensibility.

The class uses the `Singleton` pattern via `October\Rain\Support\Traits\Singleton`, meaning its methods can be accessed statically (`Manager::method()`) without needing to create a new object.

---

## Unified Response Structure

All public methods in `Manager` return a response with a unified structure that is easy for consumers (API, controllers, etc.) to process. This design ensures a consistent experience and reduces the need for custom error-handling logic.

| Key | Type | Description |
| :--- | :--- | :--- |
| `code` | `int` | Status code (200 for success, 400 for error). |
| `status` | `bool` | Operation status (`true` for success, `false` for failure). |
| `message` | `string` | Descriptive message (translatable). |
| `error` | `string\|null` | Error message (if any). |
| `errors` | `array\|null` | Array of validation errors (if any). |
| `model` | `Document\|null` | The created or updated document object. |
| `data` | `array\|null` | Array representation of the document (`toArray()`). |
| `input_data` | `array` | Input parameters after initial processing. |
| `process_data` | `array` | Data actually used during operation execution. |
| `debug` | `array\|null` | Debug information (only appears in development environment when `app.debug = true`). |

**Example of a successful response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Document created successfully.",
    "error": null,
    "errors": null,
    "model": { ... },
    "data": { ... },
    "input_data": { ... },
    "process_data": { ... }
}
```

**Example of a failed response (validation error):**
```json
{
    "code": 400,
    "status": false,
    "message": "Failed to create document.",
    "error": "The document number field is required.",
    "errors": {
        "document_number": ["The document number field is required."]
    },
    "model": null,
    "data": null
}
```

---

## Public Methods (Public API)

### 1. `createDocument`

Creates a new KYC document after validating data, document type validity, and optionally duplicate checking. The process goes through several stages:

1. Verify the existence of the owner (`owner_id`, `owner_type`).
2. Validate the `document_type` using `DocumentType`.
3. Check that the document type is allowed for the owner type (`isValidForPartyType`).
4. (Optional) Check for duplicate documents for the same owner.
5. Validate dynamic fields (`fields`) against `DocumentType::getValidationRules`.
6. Map fields to database structure (`mapInputToDocumentData`).
7. Execute within a transaction: save the document, handle attached files.
8. Fire the `nano3.kyc.document.created` event (unless disabled).
9. Return a unified response.

```php
public static function createDocument(array $options = [], bool $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$options` | `array` | Array of document creation options (see table below). |
| `$is_test_create` | `bool` | If `true`, the operation is rolled back after execution (for testing purposes). |

#### Supported `$options` Keys

| Key | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `document_type` | `string` | **Yes** | Document type code (e.g., `passport`, `national_id`). |
| `owner_id` | `int` | **Yes** | Owner ID. |
| `owner_type` | `string` | **Yes** | Owner type (Morph class name). |
| `owner` | `object` | No | Owner object (automatically extracts `id` and `type`). |
| `verifier_id` | `int` | No | Verifier ID. |
| `verifier_type` | `string` | No | Verifier type. |
| `verifier` | `object` | No | Verifier object. |
| `subject_id` | `int` | No | Associated subject ID. |
| `subject_type` | `string` | No | Associated subject type. |
| `subject` | `object` | No | Associated subject object. |
| `fields` | `array` | No | Array of dynamic fields (e.g., `['full_name' => '...', 'document_number' => '...']`). |
| `files` | `array` | No | Array of uploaded files (e.g., `['document_front' => $uploadedFile]`). |
| `status` | `string` | No | Document status (default: `pending`). |
| `is_verified` | `bool` | No | Is the document verified? (default: `false`). |
| `verification_score` | `int` | No | Trust score (0-100). |
| `is_physical_submitted` | `bool` | No | Has a hard copy been submitted? |
| `physical_copy_type` | `string` | No | Copy type (`original` or `copy`). |
| `physical_received_at` | `string\|Carbon` | No | Date the hard copy was received. |
| `physical_notes` | `string` | No | Notes about the hard copy. |
| `is_published` | `bool` | No | Is the document published? |
| `is_public` | `bool` | No | Is the document public? |
| `is_active` | `bool` | No | Is the document active? (default: `true`). |
| `is_default` | `bool` | No | Is the document default? |
| `extend_id` | `string` | No | External ID for linking. |
| `companys_id` | `string` | No | Company ID. |
| `departments_id` | `string` | No | Branch ID. |
| `metadata` | `array` | No | Metadata (JSON). |
| `other_data` | `array` | No | Additional data (JSON). |
| `config_data` | `array` | No | Configuration data (JSON). |
| `is_stop_event` | `bool` | No | If `true`, the `created` event is not fired. |
| `is_stop_duplicate` | `bool` | No | If `true`, duplicate checking is performed and duplicate creation is prevented. |
| `skip_file_check` | `bool` | No | If `true`, file fields are excluded from validation rules. |
| `validation_rules_except` | `array` | No | List of validation rules to exclude (default: file fields). |
| `duplicate_check_fields` | `array` | No | Fields used for duplicate checking (default: `['document_number', 'owner_id', 'owner_type']`). |

#### Practical Example

```php
use Nano3\Kyc\Classes\Manager;

$result = Manager::createDocument([
    'document_type' => 'passport',
    'owner'         => $currentUser, // user object
    'fields'        => [
        'document_number' => 'A12345678',
        'full_name'       => 'John Doe',
        'issue_date'      => '2020-01-01',
        'expiry_date'     => '2030-01-01',
        'country_of_issue'=> 'US',
        'nationality'     => 'US',
    ],
    'files' => [
        'document_front' => Input::file('front_image')
    ],
    'is_stop_duplicate' => true,
]);

if ($result['status']) {
    $document = $result['model'];
    echo "Document created with ID: " . $document->id;
} else {
    echo "Error: " . $result['error'];
}
```

---

### 2. `updateDocument`

Updates an existing document's data. The process is similar to `createDocument`, with the difference that fields not mentioned remain unchanged.

```php
public static function updateDocument($documentId, array $options = [], bool $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | ID of the document to update. |
| `$options` | `array` | Array of fields to update (similar to `createDocument` options). |
| `$is_test_create` | `bool` | For testing only. |

**Note:** Dynamic fields can be updated via the `fields` key, and files via the `files` key. Fields not mentioned in `$options` remain unchanged. If `is_verified = true` is passed for a document that was not verified, `verified_at` is automatically set.

---

### 3. `deleteDocument`

Soft deletes a document. The document is not permanently deleted; instead, `is_active = false` and `deleted_at = now()` are set, and it is saved. This allows later restoration.

```php
public static function deleteDocument($documentId, bool $is_test_create = false): array
```

**Note:** In the current version, soft deletion is performed manually rather than using `$document->delete()` to avoid potential conflicts with other traits. If you use the `SoftDelete` trait in the model, you can modify this method to call `delete()` directly.

---

### 4. `restoreDocument`

Restores a soft-deleted document. Uses `onlyTrashed()->find()` to locate the document and then calls `restore()`.

```php
public static function restoreDocument($documentId): array
```

---

### 5. `verifyDocument`

Verifies (approves) a document. Sets `is_verified` to `true`, records the verifier, verification date, and trust score.

```php
public static function verifyDocument($documentId, $verifier = null, $verification_score = 100, bool $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | Document ID. |
| `$verifier` | `object\|null` | Verifier object (optional). If provided, `verifier_id` and `verifier_type` are recorded. |
| `$verification_score` | `int` | Trust score (0-100, default 100). |
| `$is_test_create` | `bool` | For testing only. |

---

### 6. `rejectDocument`

Rejects a document with an optional reason. Sets `is_verified = false`, changes status to `rejected`, and stores the reason in `metadata['rejection_reason']`.

```php
public static function rejectDocument($documentId, $reason = null, bool $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | Document ID. |
| `$reason` | `string\|null` | Rejection reason (stored in `metadata['rejection_reason']`). |
| `$is_test_create` | `bool` | For testing only. |

---

### 7. `checkDuplicateDocument`

Checks for duplicate documents based on specified fields. Useful before creating a new document.

```php
public static function checkDuplicateDocument(array $options = []): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$options` | `array` | Array containing the fields used for checking (`document_number`, `owner_id`, `owner_type`, `document_type`, `exclude_id`). |

**Example:**
```php
$result = Manager::checkDuplicateDocument([
    'document_number' => 'A12345678',
    'owner_id'        => 5,
    'owner_type'      => 'RainLab\User\Models\User',
]);
if (!$result['status']) {
    echo "Document already exists: " . $result['error'];
}
```

---

### 8. `getDocumentRecords`

Retrieves document records using the `getRecords` method of the `Document` model. Supports advanced filtering options (document type, status, owner, date ranges, etc.).

```php
public static function getDocumentRecords($options = []): array
```

**Example:**
```php
$result = Manager::getDocumentRecords([
    'document_type' => 'passport',
    'status'        => 'pending',
    'is_paginator'  => true,
    'per_page'      => 20,
]);
$documents = $result['data']; // Paginator object
```

---

### 9. `getDocumentStats`

Retrieves quick statistics about documents (total count, active, published, distribution by type).

```php
public static function getDocumentStats($options = []): array
```

---

### 10. Internal Helper Methods

These methods are defined as `protected static` and are used internally to simplify code and avoid repetition.

| Method | Description |
| :--- | :--- |
| `prepareDocumentOptions()` | Prepares default company and branch values from `BasicHelper`. |
| `getQueryDate()` | Applies a flexible date filter to a query (used in `getRecords`). |
| `checkValueIsNotAll()` | Checks that a value is not a wildcard (`*` or `all`). |
| `scopeWhereField()` | Generic scope for applying `WHERE` conditions with support for `is_or`, `is_not`, and `is_force`. |

---

### 11. KYC Status Assessment Methods (Delegated to `HasAssessKycStatus`)

To assess KYC status, `Manager` uses the `HasAssessKycStatus` trait, which provides the following methods. For full details, refer to the [`HasAssessKycStatus` Trait Documentation](./Docs-HasAssessKycStatus-Trait-en.md).

#### `assessKycStatus`

Comprehensive assessment of KYC status for a given owner (individual or company) based on risk level and mandatory requirements. Supports alternative groups.

#### `assessKycStatusByCategory`

KYC status assessment limited to a specific document category (e.g., primary identity, proof of address), also supporting alternative groups.

**Quick example:**
```php
$assessment = Manager::assessKycStatus($user);
if ($assessment['data']['is_compliant']) {
    // KYC is complete
}
```

---

## Events

`Manager` fires the following events, allowing developers to hook custom logic (e.g., sending notifications, logging, updating external systems):

| Event | Parameters | Description |
| :--- | :--- | :--- |
| `nano3.kyc.document.created` | `$document, $user, $subject` | When a new document is created. |
| `nano3.kyc.document.updated` | `$document` | When a document is updated. |
| `nano3.kyc.document.deleted` | `$document` | When a document is deleted. |
| `nano3.kyc.document.restored` | `$document` | When a document is restored. |
| `nano3.kyc.document.verified` | `$document` | When a document is verified. |
| `nano3.kyc.document.rejected` | `$document` | When a document is rejected. |

**Example of listening to an event:**
```php
Event::listen('nano3.kyc.document.verified', function ($document) {
    // Send a notification to the owner that their document has been verified
    Notification::send($document->owner, new DocumentVerifiedNotification($document));
});
```

**Disabling events:** Event firing can be disabled for a specific operation by passing the option `'is_stop_event' => true` in `$options`.

---

## Duplicate Checking Mechanism

`Manager` provides the `checkDuplicateDocument` function and the `is_stop_duplicate` option in `createDocument` to prevent creating duplicate documents. The mechanism relies on the fields specified in `duplicate_check_fields` (default: `document_number`, `owner_id`, `owner_type`). These fields can be customized as needed.

**Example to prevent duplicate passport:**
```php
Manager::createDocument([
    // ... data
    'is_stop_duplicate' => true,
    'duplicate_check_fields' => ['document_number', 'document_type', 'owner_id', 'owner_type'],
]);
```

---

## Test Support (`is_test_create`)

All write methods (`create`, `update`, `delete`, `verify`, `reject`) accept an `is_test_create` parameter. When `true`, the operation is executed within a transaction that is rolled back before returning the response. This allows testing the entire logic (including saves and relationships) without leaving any trace in the database.

**Example for a unit test:**
```php
public function testCreateDocument()
{
    $result = Manager::createDocument([...], true);
    $this->assertTrue($result['status']);
    // Nothing was saved in the database
}
```

---

## File Attachment Handling

When a `files` array is passed in `createDocument` or `updateDocument`, `Manager` creates `File` records and attaches them to the document via the relationships defined in the `Document` model (`attachOne` and `attachMany`).

**Example for uploading a passport image:**
```php
$result = Manager::createDocument([
    // ...
    'files' => [
        'document_front' => $uploadedFile, // UploadedFile object
        'document_back'  => $anotherFile,
    ]
]);
```

Files are saved immediately after the document is saved. If the document is verified (`is_verified = true`) in the same operation, files are attached normally. (Note: The API interface applies additional protection to prevent changing files of verified documents; see `FileUploadDocuments`.)

---

## Dependencies

`Manager` depends on the following components:

- **`Nano3\Kyc\Models\Document`**: The main model.
- **`Nano3\Kyc\Classes\DocumentType`**: For validating document types, rules, and alternative groups.
- **`Nano3\Kyc\Classes\Manager\HasAssessKycStatus`**: KYC assessment trait.
- **`Tss\Basic\Helpers\BasicHelper`**: For preparing default company and branch IDs.
- **`Illuminate\Support\Facades\DB`**: For transactions.
- **`Illuminate\Support\Facades\Event`**: For firing events.
- **`Illuminate\Support\Facades\Validator`**: For validating dynamic fields.
- **`Carbon\Carbon`**: For date handling.

---

## Important Notes

1. **Permission checking**: The current class does not perform user permission checks (e.g., `can('create')`). It is assumed that permission checking is done at the controller layer before calling `Manager`.
2. **File handling**: It is assumed that files passed in the `files` option are `UploadedFile` objects ready for saving. They are saved using the `attachOne` and `attachMany` relationships defined in the `Document` model.
3. **Caching**: `Manager` does not directly interact with cache, but `Document` uses traits (`ListObjects`, `ListOptions`) to manage record caching. When using `Manager` to update or delete a document, the cache is automatically cleared via model events.
4. **KYC assessment**: The logic has been completely moved to the `HasAssessKycStatus` trait, making `Manager` cleaner and more focused. Any modifications to assessment logic should be made in that trait.
5. **Testing**: Use the `is_test_create = true` option to test methods without affecting the real database.

---

## Integrated Examples

### Scenario 1: Uploading a new passport via API

```php
public function store(Request $request)
{
    $result = Manager::createDocument([
        'document_type' => 'passport',
        'owner'         => Auth::getUser(),
        'fields'        => $request->input('fields'),
        'files'         => [
            'document_front' => $request->file('document_front')
        ],
        'is_stop_duplicate' => true,
    ]);

    if ($result['status']) {
        return response()->json($result);
    }

    return response()->json($result, 400);
}
```

### Scenario 2: Checking KYC completeness before allowing a purchase

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatus($user, null, null, ['risk_level' => 'medium']);

if (!$assessment['data']['is_compliant']) {
    $missing = implode(', ', $assessment['data']['missing_required_types']);
    throw new ApplicationException("Please complete the following documents: $missing");
}
// Allow purchase
```

---

## Conclusion

The `Manager` class is the backbone of the `Nano3.Kyc` add-on, providing a unified, secure, and flexible interface for managing the lifecycle of KYC documents and assessing compliance status. With support for transactions, events, testing, and alternative groups, developers can rely on it to build complex compliance applications with ease and confidence.

---

## Additional Documentation

**Reference Documents**:
- [General Plugin Documentation](./Docs-Nano3-Kyc-en.md)
- [DocumentType Class Documentation](./Docs-DocumentType-Class-en.md)
- [HasAssessKycStatus Trait Documentation](./Docs-HasAssessKycStatus-Trait-en.md)
- [HasValidKycDocuments Trait Documentation](./Docs-HasValidKycDocuments-Trait-en.md)
- [Document Model Documentation](./Docs-Document-Model-en.md)
- [DynamicAddIncludeKyc Behavior Documentation](./Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
