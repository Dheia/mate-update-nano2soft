# Manager Class Documentation – KYC Operations Manager

## Overview

The `Manager` class is located at `Nano3\Kyc\Classes\Manager`. It is the central coordinator for all identity verification (KYC) document management operations within NanoSoft App. The class acts as a service layer that interacts with the `Document` model and the `DocumentType` class to perform core operations such as:

- Creating a new document.
- Updating document data.
- Deleting and restoring a document (Soft Delete).
- Verifying (approving) a document.
- Rejecting a document.
- Checking for duplicates.
- Fetching records and statistics.

The class features:

- **Unified response structure**: All functions return an array with the same structure (`code`, `status`, `message`, `model`, `data`, etc.).
- **Transaction support**: All write operations are wrapped in a transaction to ensure data integrity.
- **Testing support (`is_test_create`)**: Ability to run the operation within a transaction that is rolled back after verifying success.
- **Event firing**: Fires events like `nano3.kyc.document.created` after operations are completed.
- **Duplicate checking**: Prevents creating duplicate documents for the same owner.
- **File attachment handling**: Supports attaching files (images, PDFs) to the document via `attachOne` and `attachMany` relationships.

The class follows the `Singleton` pattern, meaning its methods can be accessed statically (`Manager::method()`) without instantiating a new object.

---

## Unified Response Structure

All public functions in `Manager` return a response with a unified structure that is easy to consume by API, controllers, etc.

| Key | Type | Description |
| :--- | :--- | :--- |
| `code` | `int` | Status code (200 for success, 400 for error). |
| `status` | `bool` | Operation status (`true` for success, `false` for failure). |
| `message` | `string` | Descriptive message (localisable). |
| `error` | `string\|null` | Error message (if any). |
| `errors` | `array\|null` | Validation errors array (if any). |
| `model` | `Document\|null` | The document object that was created or updated. |
| `data` | `array\|null` | Array representation of the document (`toArray()`). |
| `input_data` | `array` | Input parameters after preliminary processing. |
| `process_data` | `array` | Data actually used during operation execution. |
| `debug` | `array\|null` | Debugging information (only shown in development environment when `app.debug = true`). |

**Example successful response:**
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

---

## Public API Functions

### 1. `createDocument`

Creates a new KYC document after validating data, document type validity, and duplicate checking.

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
| `document_type` | `string` | **Yes** | Document type code (e.g. `passport`, `national_id`). |
| `owner_id` | `int` | **Yes** | Owner ID. |
| `owner_type` | `string` | **Yes** | Owner type (Morph class name). |
| `owner` | `object` | No | Owner object (automatically extracts `id` and `type`). |
| `verifier_id` | `int` | No | Verifier ID. |
| `verifier_type` | `string` | No | Verifier type. |
| `verifier` | `object` | No | Verifier object. |
| `subject_id` | `int` | No | Associated subject ID. |
| `subject_type` | `string` | No | Associated subject type. |
| `subject` | `object` | No | Associated subject object. |
| `fields` | `array` | No | Dynamic fields array (e.g. `['full_name' => '...', 'document_number' => '...']`). |
| `files` | `array` | No | Array of uploaded files (e.g. `['document_front' => $uploadedFile]`). |
| `status` | `string` | No | Document status (default: `pending`). |
| `is_verified` | `bool` | No | Whether the document is verified (default: `false`). |
| `verification_score` | `int` | No | Trust score (0‑100). |
| `is_physical_submitted` | `bool` | No | Whether a physical copy was submitted. |
| `physical_copy_type` | `string` | No | Copy type (`original` or `copy`). |
| `physical_received_at` | `string\|Carbon` | No | Physical copy receipt date. |
| `physical_notes` | `string` | No | Notes about the physical copy. |
| `is_published` | `bool` | No | Whether the document is published. |
| `is_public` | `bool` | No | Whether the document is public. |
| `is_active` | `bool` | No | Whether the document is active (default: `true`). |
| `is_default` | `bool` | No | Whether the document is default. |
| `extend_id` | `string` | No | External ID for linking. |
| `companys_id` | `string` | No | Company ID. |
| `departments_id` | `string` | No | Branch ID. |
| `metadata` | `array` | No | Metadata (JSON). |
| `other_data` | `array` | No | Additional data (JSON). |
| `config_data` | `array` | No | Configuration data (JSON). |
| `is_stop_event` | `bool` | No | If `true`, the `created` event is not fired. |
| `is_stop_duplicate` | `bool` | No | If `true`, duplicate checking is performed and duplicate creation is prevented. |
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

Updates an existing document.

```php
public static function updateDocument($documentId, array $options = [], bool $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | ID of the document to update. |
| `$options` | `array` | Array of fields to update (similar to `createDocument` options). |
| `$is_test_create` | `bool` | For testing only. |

**Note:** Dynamic fields can be updated via the `fields` key, and files via the `files` key. Fields not mentioned in `$options` remain unchanged.

---

### 3. `deleteDocument`

Soft‑deletes a document (archives it, not permanently deleted).

```php
public static function deleteDocument($documentId, bool $is_test_create = false): array
```

---

### 4. `restoreDocument`

Restores a soft‑deleted document.

```php
public static function restoreDocument($documentId): array
```

---

### 5. `verifyDocument`

Verifies (approves) a document. Updates `is_verified` to `true`, records the verifier, verification date, and trust score.

```php
public static function verifyDocument($documentId, $verifier = null, $verification_score = 100, bool $is_test_create = false): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | Document ID. |
| `$verifier` | `object\|null` | Verifier object (optional). If provided, `verifier_id` and `verifier_type` are recorded. |
| `$verification_score` | `int` | Trust score (0‑100, default 100). |
| `$is_test_create` | `bool` | For testing only. |

---

### 6. `rejectDocument`

Rejects a document, optionally with a rejection reason.

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

Checks for an existing duplicate document based on specified fields. Useful before creating a new document.

```php
public static function checkDuplicateDocument(array $options = []): array
```

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$options` | `array` | Array containing fields used for checking (`document_number`, `owner_id`, `owner_type`, `document_type`, `exclude_id`). |

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

Fetches document records using the `getRecords` function from the `Document` model. Supports advanced filtering options (document type, status, owner, date ranges, etc.).

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

Fetches quick statistics about documents (total count, active, published, distribution by type).

```php
public static function getDocumentStats($options = []): array
```

---

### 10. Internal Helper Functions

| Function | Description |
| :--- | :--- |
| `prepareDocumentOptions()` | Prepares default company and branch values from `BasicHelper`. |
| `getQueryDate()` | Applies flexible date filtering to a query (used in `getRecords`). |
| `checkValueIsNotAll()` | Checks that a value is not a wildcard (`*` or `all`). |
| `scopeWhereField()` | Generic scope to apply `WHERE` conditions with support for `is_or` and `is_not`. |

---

## Events

The `Manager` fires the following events, allowing developers to attach custom logic (e.g. sending notifications, logging, updating external systems):

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
    // Send a notification to the owner that their document has been approved
    Notification::send($document->owner, new DocumentVerifiedNotification($document));
});
```

---

## Dependencies

The `Manager` depends on the following components:

- **`Nano3\Kyc\Models\Document`**: The main model.
- **`Nano3\Kyc\Classes\DocumentType`**: For validating document types and their rules.
- **`Tss\Basic\Helpers\BasicHelper`**: For preparing default company and branch IDs.
- **`Illuminate\Support\Facades\DB`**: For transactions.
- **`Illuminate\Support\Facades\Event`**: For firing events.
- **`Illuminate\Support\Facades\Validator`**: For validating dynamic fields.

---

## Important Notes

1. **Permission checking**: The current class does not check user permissions (e.g. `can('create')`). It is assumed that permission checks are performed in the controller layer before calling `Manager`.
2. **File handling**: It is assumed that files passed in the `files` option are `UploadedFile` objects ready for processing. They are saved using the `attachOne` and `attachMany` relationships defined in the `Document` model.
3. **Caching**: `Manager` does not directly interact with caching, but `Document` uses traits (`ListObjects`, `ListOptions`) to manage record caching. When using `Manager` to update or delete a document, the cache is automatically cleared via model events.
4. **Testing functions**: Use the `is_test_create = true` option to test functions without affecting the actual database.

---

## Complete Examples

### Scenario 1: Upload a new passport via API

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

### Scenario 2: Scheduled task to check document validity

```php
public function fire()
{
    $documents = Document::where('status', 'verified')
        ->whereNotNull('expiry_date')
        ->get();

    foreach ($documents as $doc) {
        if (Carbon::parse($doc->expiry_date)->isPast()) {
            $doc->status = 'expired';
            $doc->save();
        }
    }
}
```
