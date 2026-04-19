## `FileUploadService` Class Documentation – Version 1.2.3

**Overview of `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**, specifically designed for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes in this module reside under the following Namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, as well as unified error handling. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

> **Note:** This document covers versions up to 1.2.3, and includes advanced features such as multiple storage, automatic image transformation, event hooks, WebSocket, security enhancements, support for `edit` operation (file replacement), and use of transactions (`Db::transaction`) to ensure integrity, as well as unified temp key validation logic via the `HasFileUploadsMatchTempKey` trait.

---

### Introduction

The `FileUploadService` class is the **Service Layer** responsible for executing file upload, delete, and retrieval operations in NanoSoft applications. This class depends on `FileUploadRegistry` to access model settings and check permissions, and on `FileUploadUserManager` to manage the current user and check their permissions.

Simply put, `FileUploadService` is the **engine that performs actual upload operations**, ensuring that each operation follows registered settings and specified permissions, with support for temporary uploads (via temporary session key) to attach files to unsaved models. Starting from version 1.1.0, the class also supports **file replacement (edit)** in `attachOne` relations using transactions (`Db::transaction`) to ensure no data loss. In version 1.2.2, temp key validation logic was unified via the `HasFileUploadsMatchTempKey` trait, increasing security and reducing code duplication.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$registry` | `FileUploadRegistry` | Reference to the registered models registry. |
| `$userManager` | `FileUploadUserManager` | Reference to the user and permission manager. |
| `BLACKLISTED_EXTENSIONS` | `array` | List of dangerous extensions that are permanently forbidden (PHP, JS, HTML, EXE, etc.). |

---

### Important Methods

#### 1. Generate Temporary Session Key

##### `public function generateTempSessionKey(string $modelClass, string $field, $userId = null, $userType = null): string`
- **Purpose**: Generate a unique and secure temporary session key to attach files before saving the model.
- **Parameters**:
  - `$modelClass`: Full class name of the model.
  - `$field`: Field name.
  - `$userId`: (Optional) User ID; if not provided, uses current user.
  - `$userType`: (Optional) User type (`backend`/`frontend`/`guest`).
- **Behavior**:
  - Gets current user ID and type or uses passed values.
  - Creates a key in format `tmp_{base64(data)}:{hash}` where `data = modelClass|field|userId|userType|timestamp` and signed with HMAC-SHA256 using `app.key`.
- **Returns**: The temporary session key.
- **Example**:
  ```php
  $tempKey = $service->generateTempSessionKey('Nano\Shop\Models\Product', 'image');
  ```

#### 2. Validate Temporary Session Key (Basic)

##### `public function validateTempSessionKey(string $tempSessionKey): ?array`
- **Purpose**: Validate a temporary session key (signature, expiration, structure) and extract its data.
- **Parameters**: `$tempSessionKey` – The key to validate.
- **Returns**: Array containing `modelClass`, `field`, `userId`, `userType`, `timestamp`, or `null` if invalid.
- **Behavior**:
  - Checks prefix `tmp_`.
  - Decodes and verifies signature.
  - Checks key validity (default 3600 seconds, configurable via `security.temp_key_ttl`).

#### 3. Advanced Temp Key Validation and Matching (from trait)

##### `public function validateAndMatchTempKey($tempKey, string $modelClass, string $field, $user = null, array $options = []): array`
- **Purpose**: An integrated method that combines key validation and matching against model, field, and user, with advanced options to control behavior.
- **Parameters**:
  - `$tempKey`: Temporary session key (string) or its data (array).
  - `$modelClass`: Expected model class name.
  - `$field`: Expected field name.
  - `$user`: Current user (optional, used to get userId and userType).
  - `$options`: Array of advanced options (see table below).
- **Advanced Options**:
  | Option | Type | Description |
  |--------|------|-------------|
  | `throw_on_failure` | bool | Throw exception on failure (`true`) or return error result array (`false`). |
  | `skip_user_check` | bool | Skip user check. |
  | `strict_mode` | bool | Strict mode: requires exact model and field match. |
  | `stop_on_first_failure` | bool | Stop at first failure (`true`) or collect errors. |
  | `allow_expired_key` | bool | Allow expired keys within a grace period. |
  | `expiry_grace_period` | int | Grace period in seconds (default 300). |
  | `custom_validator` | callable | Additional validation function. |
  | `cache_results` | bool | Cache validation results within the same request. |
  | `collect_metadata` | bool | Collect performance data. |
- **Returns**: Array containing `status`, `message`, `data` (key data and matching results), `process_data`, `error_code`, etc.
- **Internal usage**: Used by `upload`, `deleteFile`, `attachTempFiles`, `getFiles` to validate temp keys provided by the client.
- **Example**:
  ```php
  $result = $service->validateAndMatchTempKey($tempKey, 'Product', 'image', $user, [
      'throw_on_failure' => false,
      'strict_mode' => true,
  ]);
  if ($result['status']) {
      echo "Key is valid and matches model, field, and user";
  } else {
      echo "Validation failed: " . $result['message'];
  }
  ```

> **Note:** This method is part of the `HasFileUploadsMatchTempKey` trait used by `FileUploadService`. It was added in version 1.2.2.

#### 4. Upload Single File

##### `public function upload(string $modelClass, string $field, $fileData, array $options = []): array`
- **Purpose**: Upload a single file to a specific field, supporting `add` (new upload) and `edit` (replace existing file in `attachOne` relation).
- **Parameters**:
  - `$modelClass`: Full class name of the model.
  - `$field`: Field name.
  - `$fileData`: File data (`UploadedFile` object or base64 string).
  - `$options`: Additional options array:
    - `model`: Model object (if exists and saved).
    - `temp_session_key`: Temporary session key (to bypass automatic generation). **Since v1.2.2, this key is validated and matched using `validateAndMatchTempKey` before use.**
    - `skip_permission_check`: `bool` to skip permission check (useful for tests).
    - `title`, `description`: To set file title and description.
    - `...`: Any additional options passed to `Base64::onUpload`.
- **Behavior**:
  - Checks that model and field are registered in `FileUploadRegistry`.
  - **Determines operation**:
    - If `options['model']` exists and is saved (`exists`) and has an `attachOne` relation for the field and an associated file is found, operation becomes `edit` (replacement).
    - Otherwise operation becomes `add` (new upload).
  - **If `temp_session_key` is passed**, calls `validateAndMatchTempKey` to verify that the key belongs to the same model, field, and user. If validation fails, throws an exception.
  - Checks permission for the operation (`add` or `edit`) for the current user (considering global settings like `disable_upload`, `disable_edit`).
  - Fires `nano.fileupload.beforeUpload` event (passing the operation).
  - Validates the file (blacklist, size, types) via `validateFile`.
  - **Uses a database transaction (`Db::transaction`)** to ensure integrity:
    - Upload new file and save it (including setting `session_key`, `expires_at`, `hash`, `meta`, `disk`).
    - **If operation is `edit` and new file upload succeeds**:
      - Fires `nano.fileupload.beforeEditDelete` event.
      - Deletes the old file.
      - Fires `nano.fileupload.afterEditDelete` event.
    - Attaches the new file to the model relation (if exists).
  - Fires `nano.fileupload.afterUpload` event and sends WebSocket notification (if enabled).
- **Returns**: Array with same structure as `Base64::onUpload` with additional fields: `code`, `status`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`, `error_code`. `process_data` contains `operation` (`add`/`edit`) and `existing_file_deleted` (in case of `edit`).
- **Example**:
  ```php
  // New upload (add) with temp key (will be validated)
  $result = $service->upload('Product', 'image', $file, ['temp_session_key' => $tempKey]);
  
  // Replace file (edit)
  $product = Product::find(10);
  $result = $service->upload('Product', 'image', $newFile, ['model' => $product]);
  
  if ($result['status']) {
      echo "File uploaded successfully. Operation: " . $result['process_data']['operation'];
  } else {
      echo "Failed: " . $result['error'] . " (code: " . $result['error_code'] . ")";
  }
  ```

#### 5. Upload Multiple Files (for multiple field)

##### `public function uploadMultiple(string $modelClass, string $field, array $filesData, array $options = []): array`
- **Purpose**: Upload multiple files to a multiple field in one request.
- **Parameters**: Same as `upload`, but `$filesData` is an array of file data.
- **Behavior**: Iterates over `$filesData` and calls `upload()` for each file, then aggregates results and counts successes.
- **Returns**: Array containing `code`, `status`, `message`, `data` (results per file), `process_data` (success/total counts).
- **Example**:
  ```php
  $files = [ $file1, $file2, $file3 ];
  $result = $service->uploadMultiple('Product', 'gallery', $files, ['model' => $product]);
  echo "Successfully uploaded " . $result['process_data']['success_count'] . " of " . $result['process_data']['total'];
  ```

#### 6. Delete File

##### `public function deleteFile(int $fileId, ?string $modelClass = null, ?string $field = null, $user = null): array`
- **Purpose**: Delete a specific file with permission check.
- **Parameters**:
  - `$fileId`: File ID.
  - `$modelClass`: (Optional) Model class name to check delete permission.
  - `$field`: (Optional) Field name to check delete permission.
  - `$user`: (Optional) User object.
- **Behavior**:
  - Fires `nano.fileupload.beforeDelete` event.
  - Finds the file using `File::find()`.
  - Checks permission based on three cases:
    1. `modelClass` and `field` provided: checks `delete` permission for current user (considering `disable_delete`).
    2. File is attached to a saved model: extracts `modelClass` and `field` from `attachment_type` and `field` in the file record.
    3. File is temporary (`session_key` exists): **uses `validateAndMatchTempKey` to verify the key and that the current user is the creator (with `strict_mode = false` because `attachment_type` may be null).**
  - If user is not authorized, throws exception.
  - Deletes the file.
  - Fires `nano.fileupload.afterDelete` event and sends WebSocket notification.
- **Returns**: Array containing `code`, `status`, `message`, `error`, `error_code`, etc.
- **Example**:
  ```php
  $result = $service->deleteFile(123, 'Nano\Shop\Models\Product', 'image');
  if ($result['status']) {
      echo "File deleted successfully";
  }
  ```

#### 7. Retrieve Files

##### `public function getFiles(string $modelClass, string $field, ?int $modelId = null, array $options = []): array`
- **Purpose**: Retrieve files associated with a model and field, or temporary files.
- **Parameters**:
  - `$modelClass`: Full class name of the model.
  - `$field`: Field name.
  - `$modelId`: (Optional) Saved model ID.
  - `$options`: Additional options:
    - `temp_session_key`: Temporary session key (to retrieve unassociated files). **Since v1.2.2, this key is validated and matched using `validateAndMatchTempKey` before use.**
    - `with_thumbs`: `bool` to include thumbnails.
    - `thumb_sizes`: Array of thumbnail sizes (e.g., `['thumb' => [100,100]]`).
- **Behavior**:
  - Checks that the field is registered and `view` permission (considering `disable_get`).
  - Builds query on `File` filtering by `field` and (`attachment_id` + `attachment_type` if `modelId` provided) or `session_key` if `temp_session_key` provided.
  - **When using `temp_session_key`**, calls `validateAndMatchTempKey` to validate the key and match it (model, field, user) before executing the query. If validation fails, throws an exception.
  - Returns files with basic data (id, title, description, path, size, content_type) and thumbnails if requested.
- **Returns**: Array containing `code`, `status`, `message`, `data`.
- **Example**:
  ```php
  $files = $service->getFiles('Product', 'image', 456, ['with_thumbs' => true]);
  foreach ($files['data'] as $file) {
      echo "<img src='{$file['thumb']}'>";
  }
  ```

#### 8. Attach Temporary Files to Saved Model

##### `public function attachTempFiles(Model $model, string $field, string $tempSessionKey, array $options = []): array`
- **Purpose**: Transfer files uploaded using a temporary session key to a saved model.
- **Parameters**:
  - `$model`: Saved model object.
  - `$field`: Field name.
  - `$tempSessionKey`: Temporary session key used during upload.
  - `$options`: Additional options (e.g., `skip_permission_check` for tests).
- **Behavior**:
  - **Uses `validateAndMatchTempKey`** to validate the key and match it against the model, field, and user in one integrated step (instead of previously duplicated manual code). If validation fails, throws an exception.
  - Checks `edit` permission (because attaching is considered a modification to the model) unless overridden by `skip_permission_check`.
  - Finds files with `session_key = $tempSessionKey` and `field = $field`.
  - For each file, updates `attachment_id` and `attachment_type` to attach to the model, and clears `session_key` and `expires_at`.
  - If the model has a relation for this field, uses `add()`.
  - Fires `nano.fileupload.afterAttach` event and sends WebSocket notification.
- **Returns**: Unified array containing `code`, `status`, `message`, `data` (attached files), `process_data` (number of attached files, etc.).
- **Example**:
  ```php
  $product = new Product();
  $product->name = 'New Product';
  $product->save();
  $result = $service->attachTempFiles($product, 'image', $tempKey);
  if ($result['status']) {
      echo "Attached " . $result['process_data']['attached_count'] . " files";
  }
  ```

#### 9. Validate File

##### `public function validateFile(string $modelClass, string $field, $fileData): void`
- **Purpose**: Validate that the file matches the registered field constraints (max size, allowed types) as well as the blacklist.
- **Parameters**:
  - `$modelClass`: Full class name of the model.
  - `$field`: Field name.
  - `$fileData`: File data (`UploadedFile` object or base64 string).
- **Behavior**:
  - Calls `getFieldConstraints()` from `FileUploadRegistry`.
  - **Checks blacklist first** (dangerous extensions).
  - Checks file size (compares with `max_filesize`).
  - Checks file extension against `allowed_types`.
- **Returns**: Nothing, but throws `FileUploadException` with appropriate error code if validation fails, and logs the failed attempt in `fileupload.log`.
- **Example**:
  ```php
  try {
      $service->validateFile('Product', 'image', $uploadedFile);
      // File is valid
  } catch (FileUploadException $e) {
      echo "File invalid: " . $e->getMessage() . " (code: " . $e->getErrorCode() . ")";
  }
  ```

#### 10. Helper Methods (Storage, Transformation, Notifications)

##### `protected function getStorageDiskForField($modelClass, $field): ?string`
- Determines the custom storage disk name for the field (from `getProcessingOptions`) and checks its existence.

##### `protected function applyAutoProcessing($file, $modelClass, $field): void`
- Applies automatic resizing and watermarking to images according to field options and global settings (`disable_auto_resize`, `disable_auto_watermark`).

##### `protected function notifyWebSocket($event, $data): void`
- Fires `nano.fileupload.websocket.notify` event if WebSocket is enabled in settings.

##### `protected function fireEventSafe(string $eventName, $params = [], $halt = false)`
- Safely fires an event compatible with both Laravel and OctoberCMS. Used by service methods and events inside the trait `HasFileUploadsMatchTempKey` via `fireEventSafeInTrait`.

---

### Events

| Event Name | Call Location | Parameters |
|------------|---------------|------------|
| `nano.fileupload.beforeUpload` | Start of `upload` (before transaction) | `$modelClass, $field, $fileData, &$options, $operation` |
| `nano.fileupload.afterUpload` | End of `upload` (after success) | `$file, $modelClass, $field, $options, $operation` |
| `nano.fileupload.beforeEditDelete` | During `upload` (`edit` operation, before deleting old) | `$existingFile, $modelClass, $field, $model` |
| `nano.fileupload.afterEditDelete` | During `upload` (`edit` operation, after deleting old) | `$existingFile, $modelClass, $field, $model` |
| `nano.fileupload.beforeDelete` | Start of `deleteFile` | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | End of `deleteFile` (after deletion) | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterAttach` | End of `attachTempFiles` (after attachment) | `$model, $field, $attachedFiles, $tempSessionKey` |
| `nano.fileupload.websocket.notify` | After `upload`, `deleteFile`, `attachTempFiles` (if WebSocket enabled) | `$channel, $event, $data` |

Developers can listen to these events to extend behavior (e.g., send notifications, additional logging, modify data).

---

### Comprehensive Practical Examples

#### Example 1: Upload main image for a new product using temporary session key (with auto resize and watermark)

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

$service = FileUploadService::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';

// 1. Generate temporary session key (includes userType and timestamp)
$tempKey = $service->generateTempSessionKey($modelClass, $field, $userManager->getId(), $userManager->getUserType());

// 2. Upload image (automatic transformations will be applied according to field settings)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey,
    'title' => 'Product Image',
    'description' => 'Image description',
]);

if (!$uploadResult['status']) {
    die("Upload failed: " . $uploadResult['error'] . " (code: " . $uploadResult['error_code'] . ")");
}

// 3. Create and save product
$product = new Product();
$product->name = 'New Product';
$product->save();

// 4. Attach image to product (service validates key and user via validateAndMatchTempKey)
$attachResult = $service->attachTempFiles($product, $field, $tempKey);
if (!$attachResult['status']) {
    die("Attachment failed: " . $attachResult['error']);
}

echo "Image uploaded and attached to product ID {$product->id}";
```

#### Example 2: Replace an existing product image (edit operation)

```php
$product = Product::find(10); // existing product
$newImageFile = $request->file('new_image');

$result = $service->upload(Product::class, 'image', $newImageFile, [
    'model' => $product, // pass saved model
]);

if ($result['status']) {
    echo "Image replaced successfully. Operation: " . $result['process_data']['operation']; // edit
    echo "Old file ID deleted: " . $result['process_data']['existing_file_deleted'];
} else {
    echo "Replacement failed: " . $result['error'];
}
```

#### Example 3: Upload multiple images to an existing product gallery (with WebSocket)

```php
$product = Product::find(10); // existing product

// Upload multiple images
$filesData = [$file1, $file2, $file3];
$result = $service->uploadMultiple(Product::class, 'gallery', $filesData, [
    'model' => $product, // direct attach
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " images";
    // afterUpload event will fire for each file, and WebSocket can notify the front-end
} else {
    echo "Failed to upload some images: " . $result['error'];
}
```

#### Example 4: Retrieve all images of a product gallery with custom thumbnails

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
    ],
]);

foreach ($result['data'] as $image) {
    echo "<img src='{$image['small']}' alt='{$image['title']}'>";
}
```

#### Example 5: Delete a main image after permission check (with events)

```php
Event::listen('nano.fileupload.beforeDelete', function ($fileId, $modelClass, $field) {
    \Log::info("Attempting to delete file {$fileId}");
});

$result = $service->deleteFile(123, Product::class, 'image');

if ($result['status']) {
    echo "Image deleted successfully";
} else {
    echo "Error: " . $result['error'] . " (code: " . $result['error_code'] . ")";
}
```

#### Example 6: Use `validateAndMatchTempKey` directly to validate a temp key (e.g., in a custom API endpoint)

```php
$tempKey = $request->input('temp_key');
$user = auth()->user();

$result = $service->validateAndMatchTempKey($tempKey, 'Product', 'image', $user, [
    'throw_on_failure' => false, // we want JSON response instead of exception
    'strict_mode' => true,
]);

if ($result['status']) {
    return response()->json(['valid' => true, 'data' => $result['data']]);
} else {
    return response()->json(['valid' => false, 'error' => $result['message']], $result['code']);
}
```

#### Example 7: Use `afterEditDelete` event to log replacement

```php
Event::listen('nano.fileupload.afterEditDelete', function ($oldFile, $modelClass, $field, $model) {
    \Log::info("Old file {$oldFile->id} replaced with new file for model {$model->id}");
});
```

#### Example 8: Integrate `FileUploadService` with API using error codes

In API controller:

```php
public function uploadImage(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $file = $request->file('file');
    $modelId = $request->input('model_id');
    
    $service = FileUploadService::instance();
    
    $options = [];
    if ($modelId) {
        $model = $modelClass::find($modelId);
        if (!$model) {
            return response()->json(['error' => 'Model not found'], 404);
        }
        $options['model'] = $model;
    } else {
        $tempKey = $service->generateTempSessionKey($modelClass, $field);
        $options['temp_session_key'] = $tempKey;
    }
    
    $result = $service->upload($modelClass, $field, $file, $options);
    
    if ($result['status']) {
        return response()->json($result, 200);
    } else {
        return response()->json([
            'error' => $result['error'],
            'error_code' => $result['error_code'],
            'message' => $result['message']
        ], $result['code']);
    }
}
```

---

### Interaction with Other Classes

- **With `FileUploadRegistry`**: Uses `$registry` to get field settings (`getFieldConfig`, `getFieldConstraints`, `getProcessingOptions`) and check permissions (`can`, `validate`).
- **With `FileUploadUserManager`**: Uses `$userManager` to get current user and type (`getUser`, `getUserType`, `getId`).
- **With `Base64` (moved into the add-on)**: Calls `Base64::onUpload`, `Base64::is_base64`, `Base64::base64ToUploadedFile`, `Base64::getBase64ImageSize`, `Base64::getDataInBase64`.
- **With `HasFileUploadsMatchTempKey` trait**: The class uses this trait to leverage `validateAndMatchTempKey` and `fireEventSafeInTrait`. These methods are available as if part of `FileUploadService`.
- **With `October\Rain\Resize\Resizer`**: For image resizing (if `auto_resize` is enabled).
- **With `Nano2\Watermark\Classes\WatermarkHelper`**: For watermarking (if the add-on exists and `auto_watermark` is enabled).

---

### Best Practices

1. **Always use a temporary session key for new models**: To avoid losing files if the model is not saved yet.
2. **For replacement (edit), use `model` in options** instead of a temp key, and let the service handle the transaction (`Db::transaction`) and old file deletion.
3. **Do not trust temp keys provided by the client without validation**: The service does this automatically via `validateAndMatchTempKey`, but ensure you do not bypass this step.
4. **Validate the file before upload using `validateFile`**: To provide clear error messages to the user (the service does this automatically, but you can also call it manually).
5. **Log transaction IDs in the database** to track upload operations.
6. **Use `uploadMultiple` for multiple fields** instead of calling `upload` in a loop, because `uploadMultiple` aggregates errors and provides statistics.
7. **Do not rely solely on `temp_session_key`**: After saving the model, use `attachTempFiles` to transfer ownership.
8. **Handle errors according to status code**: 400 (bad request), 403 (permission denied), 404 (not found), 422 (validation failed), 500 (internal error) – and use `error_code` for fine-grained differentiation.
9. **Enable `app.debug` in development environment** to get full error details.
10. **Use `with_thumbs` only when needed** to reduce response size and increase performance.
11. **Keep temporary session keys in session or temporary database** to avoid losing them.
12. **Test permissions thoroughly**: Ensure unauthorized users cannot upload, delete, or replace files.
13. **Use automatic transformations (`auto_resize`, `auto_watermark`) wisely**: They may consume server resources, so enable only when needed.
14. **Monitor the `fileupload.log`** to detect attempts to upload dangerous files or recurring errors.
15. **Create a scheduled cron job** to clean up expired files (`expires_at`) that are unattached.
16. **Use `skip_permission_check => true` only in tests**, not in production.

---

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `Field ... not registered for model ...` | Field not registered in `FileUploadRegistry`. | Add the field in registration. |
| `You do not have permission to upload files for this field` | User lacks `add` permission for the field or `disable_upload` is enabled. | Check `permissions` settings in Registry or global settings, and user permissions. |
| `You do not have permission to replace files for this field` | Attempting `edit` without `edit` permission or `disable_edit` enabled. | Check `edit` permission in Registry and global settings. |
| `Invalid file data` | Passed data is neither `UploadedFile` nor valid base64. | Ensure correct data format. |
| `File not found` (when deleting) | File ID does not exist. | Verify the ID is correct. |
| `File size exceeds the maximum allowed` | File size larger than `max_filesize`. | Compress the file or increase `max_filesize` in field settings or defaults. |
| `File type not allowed` | File extension not listed in `allowed_types`. | Use an allowed type or modify `allowed_types`. |
| `File type ... is not allowed for security reasons` | Extension is in the blacklist (`BLACKLISTED_EXTENSIONS`). | This block cannot be overridden for security reasons; use safe file types. |
| `Temporary session key is invalid or expired` | Key expired (more than `temp_key_ttl`) or was tampered with, or validation failed in `validateAndMatchTempKey`. | Re-upload the file or use a new key. |
| `Temporary key does not belong to this model/field` | Model or field mismatch in `validateAndMatchTempKey`. | Ensure the key belongs to the same model and field. |
| `Temporary key belongs to another user` | User mismatch in `validateAndMatchTempKey`. | Ensure the key belongs to the current user. |
| `Auto-resize failed` | GD or Imagick library not installed, or file path invalid. | Install `php-gd` or `php-imagick`, check file permissions. |
| `Auto-watermark failed` | `Nano2.Watermark` add-on not installed or logo path incorrect. | Install the add-on and ensure the logo file exists at the specified path. |
| `Storage disk 's3' not found` | `s3` disk not configured in `config/filesystems.php`. | Add appropriate disk configuration. |
| `Old file was deleted despite new upload failing` | (Rare) Transaction error or database settings issue. | Ensure InnoDB engine is used for transaction support. |

---

### Summary

The `FileUploadService` class is the backbone of file upload operations in NanoSoft applications. It provides a unified and secure interface for uploading, deleting, and retrieving files, with full support for different user types, granular permissions, temporary attachment for unsaved models, automatic image transformations, multiple storage, event hooks, and WebSocket notifications. Starting from version 1.1.0, it also supports file replacement (`edit`) in `attachOne` relations using transactions (`Db::transaction`) and custom events. In version 1.2.2, the `HasFileUploadsMatchTempKey` trait was added, providing the `validateAndMatchTempKey` method to unify temp key validation logic, increasing security and reducing code duplication in `upload`, `deleteFile`, `attachTempFiles`, and `getFiles`. Thanks to its dependency on `FileUploadRegistry` and `FileUploadUserManager`, developers can build powerful and scalable file upload systems without worrying about implementation details or data security.

For additional advanced examples, refer to [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advanced-Examples-en.md).

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advanced-Examples-en.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advanced-Examples-en.md)
- [validateAndMatchTempKey Function for `FileUploadService` Class](./Docs-FileUploadService-validateAndMatchTempKey-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advanced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

---
