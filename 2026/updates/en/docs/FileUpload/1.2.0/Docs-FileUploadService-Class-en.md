# `FileUploadService` Class Documentation

**Overview of the `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**, specifically designed for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes in this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, as well as unified error handling. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

> **Note:** This document covers versions up to 1.2.0, and includes advanced features such as multiple storage, automatic image transformation, event hooks, WebSocket, security enhancements, support for the `edit` operation (file replacement), and the use of transactions (`Db::transaction`) to ensure integrity.

---

### Introduction

The `FileUploadService` class is the **Service Layer** responsible for executing file upload, delete, and retrieval operations in NanoSoft applications. This class depends on `FileUploadRegistry` to access model settings and check permissions, and on `FileUploadUserManager` to manage the current user and check their permissions.

Simply put, `FileUploadService` is **the engine that performs the actual upload operations**, ensuring that each operation complies with the registered settings and specified permissions, with support for temporary uploads (via a temporary session key) to attach files to unsaved models. Starting from version 1.1.0, the class also supports **file replacement (edit)** in `attachOne` relations using transactions (`Db::transaction`) to prevent data loss.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$registry` | `FileUploadRegistry` | Reference to the registered models registry. |
| `$userManager` | `FileUploadUserManager` | Reference to the user and permission manager. |
| `BLACKLISTED_EXTENSIONS` | `array` | List of dangerous extensions that are permanently forbidden (PHP, JS, HTML, EXE, etc.). |

---

### Important Methods

#### 1. Generate a Temporary Session Key

##### `public function generateTempSessionKey(string $modelClass, string $field, $userId = null, $userType = null): string`
- **Purpose**: Generate a unique and secure temporary session key for attaching files before saving the model.
- **Parameters**:
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$userId`: (Optional) User ID; if not provided, uses the current user.
  - `$userType`: (Optional) User type (`backend`/`frontend`/`guest`).
- **Behavior**:
  - Gets the current user ID and type or uses the passed values.
  - Creates a key in the format `tmp_{base64(data)}:{hash}` where `data = modelClass|field|userId|userType|timestamp` and signs it with HMAC-SHA256 using `app.key`.
- **Returns**: The temporary session key.
- **Example**:
  ```php
  $tempKey = $service->generateTempSessionKey('Nano\Shop\Models\Product', 'image');
  ```

#### 2. Validate a Temporary Session Key

##### `public function validateTempSessionKey(string $tempSessionKey): ?array`
- **Purpose**: Validate the temporary session key (signature, expiry, structure) and extract its data.
- **Parameters**: `$tempSessionKey` – The key to validate.
- **Returns**: An array containing `modelClass`, `field`, `userId`, `userType`, `timestamp`, or `null` if invalid.
- **Behavior**:
  - Checks the prefix `tmp_`.
  - Decodes and verifies the signature.
  - Checks key validity (default 3600 seconds, adjustable via `security.temp_key_ttl`).

#### 3. Upload a Single File

##### `public function upload(string $modelClass, string $field, $fileData, array $options = []): array`
- **Purpose**: Upload a single file for a specific field, supporting `add` (new upload) and `edit` (replace existing file in an `attachOne` relation) operations.
- **Parameters**:
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$fileData`: File data (`UploadedFile` object or base64 string).
  - `$options`: Additional options array:
    - `model`: Model object (if it exists and is saved).
    - `temp_session_key`: Temporary session key (to bypass automatic generation).
    - `skip_permission_check`: `bool` to skip permission checking (useful for tests).
    - `title`, `description`: To set file title and description.
    - `...`: Any additional options passed to `Base64::onUpload`.
- **Behavior**:
  - Checks that the model and field are registered in `FileUploadRegistry`.
  - **Determines the operation**:
    - If `options['model']` exists and is saved (`exists`) and has an `attachOne` relation for the field and an attached file is found, the operation becomes `edit` (replace).
    - Otherwise, the operation becomes `add` (new upload).
  - Checks permission for the operation (`add` or `edit`) for the current user (considering global settings like `disable_upload`, `disable_edit`).
  - Dispatches event `nano.fileupload.beforeUpload` (passing the operation).
  - Validates the file (blacklist, size, types) via `validateFile`.
  - **Uses a database transaction (`Db::transaction`)** to ensure integrity:
    - Upload and save the new file (including setting `session_key`, `expires_at`, `hash`, `meta`, `disk`).
    - **If the operation is `edit` and the new file upload succeeds**:
      - Dispatches event `nano.fileupload.beforeEditDelete`.
      - Deletes the old file.
      - Dispatches event `nano.fileupload.afterEditDelete`.
    - Attaches the new file to the model relation (if it exists).
  - Dispatches event `nano.fileupload.afterUpload` and sends a WebSocket notification (if enabled).
- **Returns**: An array with the same structure as `Base64::onUpload` plus additional fields: `code`, `status`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`, `error_code`. `process_data` contains `operation` (`add`/`edit`) and `existing_file_deleted` (in case of `edit`).
- **Example**:
  ```php
  // New upload (add)
  $result = $service->upload('Product', 'image', $file, ['temp_session_key' => $tempKey]);
  
  // File replacement (edit)
  $product = Product::find(10);
  $result = $service->upload('Product', 'image', $newFile, ['model' => $product]);
  
  if ($result['status']) {
      echo "File uploaded successfully. Operation: " . $result['process_data']['operation'];
  } else {
      echo "Failed: " . $result['error'] . " (code: " . $result['error_code'] . ")";
  }
  ```

#### 4. Upload Multiple Files (for a Multiple Field)

##### `public function uploadMultiple(string $modelClass, string $field, array $filesData, array $options = []): array`
- **Purpose**: Upload multiple files for a multiple field in a single request.
- **Parameters**: Same as `upload`, but `$filesData` is an array of file data.
- **Behavior**: Iterates over `$filesData` and calls `upload()` for each file, then aggregates the results and calculates success counts.
- **Returns**: An array containing `code`, `status`, `message`, `data` (results for each file), `process_data` (success count / total).
- **Example**:
  ```php
  $files = [ $file1, $file2, $file3 ];
  $result = $service->uploadMultiple('Product', 'gallery', $files, ['model' => $product]);
  echo "Successfully uploaded " . $result['process_data']['success_count'] . " out of " . $result['process_data']['total'];
  ```

#### 5. Delete a File

##### `public function deleteFile(int $fileId, ?string $modelClass = null, ?string $field = null, $user = null): array`
- **Purpose**: Delete a specific file with permission checking.
- **Parameters**:
  - `$fileId`: File ID.
  - `$modelClass`: (Optional) Class name to check delete permission.
  - `$field`: (Optional) Field name to check delete permission.
  - `$user`: (Optional) User object.
- **Behavior**:
  - Dispatches event `nano.fileupload.beforeDelete`.
  - Finds the file using `File::find()`.
  - Checks permission based on three cases:
    1. `modelClass` and `field` are provided: checks `delete` permission for the current user (considering `disable_delete`).
    2. The file is attached to a saved model: extracts `modelClass` and `field` from `attachment_type` and `field` in the file record.
    3. The file is temporary (`session_key` exists): validates the key and that the current user is the creator.
  - If the user is not authorized, throws an exception.
  - Deletes the file.
  - Dispatches event `nano.fileupload.afterDelete` and sends a WebSocket notification.
- **Returns**: An array containing `code`, `status`, `message`, `error`, `error_code`, etc.
- **Example**:
  ```php
  $result = $service->deleteFile(123, 'Nano\Shop\Models\Product', 'image');
  if ($result['status']) {
      echo "File deleted successfully";
  }
  ```

#### 6. Retrieve Files

##### `public function getFiles(string $modelClass, string $field, ?int $modelId = null, array $options = []): array`
- **Purpose**: Retrieve files associated with a specific model and field, or temporary files.
- **Parameters**:
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$modelId`: (Optional) Saved model ID.
  - `$options`: Additional options:
    - `temp_session_key`: Temporary session key (to retrieve unattached files).
    - `with_thumbs`: `bool` to include thumbnails.
    - `thumb_sizes`: Array of thumbnail sizes (e.g., `['thumb' => [100,100]]`).
- **Behavior**:
  - Checks that the field is registered and has `view` permission (considering `disable_get`).
  - Builds a query on `File` filtering by `field` and either (`attachment_id` + `attachment_type` if `modelId` is provided) or `session_key` if `temp_session_key` is provided.
  - **When using `temp_session_key`**, validates the key (signature, expiry, matching user, model, field) before executing the query.
  - Returns files with basic data (id, title, description, path, size, content_type) and thumbnails if requested.
- **Returns**: An array containing `code`, `status`, `message`, `data`.
- **Example**:
  ```php
  $files = $service->getFiles('Product', 'image', 456, ['with_thumbs' => true]);
  foreach ($files['data'] as $file) {
      echo "<img src='{$file['thumb']}'>";
  }
  ```

#### 7. Attach Temporary Files to a Saved Model

##### `public function attachTempFiles(Model $model, string $field, string $tempSessionKey, array $options = []): array` (Improved in version 1.1.0)
- **Purpose**: Move files uploaded using a temporary session key to a saved model.
- **Parameters**:
  - `$model`: Saved model object.
  - `$field`: Field name.
  - `$tempSessionKey`: The temporary session key used during upload.
  - `$options`: Additional options (e.g., `skip_permission_check` for tests).
- **Behavior**:
  - Validates the key (signature, expiry).
  - Checks that the `modelClass` and `field` from the key match the model.
  - Checks that the current user is the same user who created the key (same `userId` and `userType`).
  - **Checks `edit` permission** (because attaching is considered a model modification) unless overridden by `skip_permission_check`.
  - Finds files with `session_key = $tempSessionKey` and `field = $field`.
  - For each file, updates `attachment_id` and `attachment_type` to attach it to the model, and clears `session_key` and `expires_at`.
  - If the model has a relation for this field, uses `add()`.
- **Returns**: A unified array containing `code`, `status`, `message`, `data` (attached files), `process_data` (number of attached files, etc.).
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

#### 8. Validate a File

##### `public function validateFile(string $modelClass, string $field, $fileData): void`
- **Purpose**: Check that the file matches the registered field constraints (max size, allowed types) as well as the blacklist.
- **Parameters**:
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$fileData`: File data (`UploadedFile` object or base64 string).
- **Behavior**:
  - Calls `getFieldConstraints()` from `FileUploadRegistry`.
  - **Checks the blacklist** first (dangerous extensions).
  - Checks file size (compares with `max_filesize`).
  - Checks file extension against `allowed_types`.
- **Returns**: Nothing, but throws `FileUploadException` with an appropriate error code if validation fails, and logs the failed attempt in `fileupload.log`.
- **Example**:
  ```php
  try {
      $service->validateFile('Product', 'image', $uploadedFile);
      // File is valid
  } catch (FileUploadException $e) {
      echo "File invalid: " . $e->getMessage() . " (code: " . $e->getErrorCode() . ")";
  }
  ```

#### 9. Helper Functions (Storage, Transformation, Notifications)

##### `protected function getStorageDiskForField($modelClass, $field): ?string`
- Determines the custom storage disk name for the field (from `getProcessingOptions`) and checks if it exists.

##### `protected function applyAutoProcessing($file, $modelClass, $field): void`
- Applies automatic resizing and watermarking to images according to field options and global settings (`disable_auto_resize`, `disable_auto_watermark`).

##### `protected function notifyWebSocket($event, $data): void`
- Dispatches the `nano.fileupload.websocket.notify` event if WebSocket is enabled in settings.

---

### Events

| Event Name | Call Location | Parameters |
|------------|---------------|------------|
| `nano.fileupload.beforeUpload` | Start of `upload` (before transaction) | `$modelClass, $field, $fileData, &$options, $operation` |
| `nano.fileupload.afterUpload` | End of `upload` (after success) | `$file, $modelClass, $field, $options, $operation` |
| `nano.fileupload.beforeEditDelete` | During `upload` (edit operation, before deleting old file) | `$existingFile, $modelClass, $field, $model` |
| `nano.fileupload.afterEditDelete` | During `upload` (edit operation, after deleting old file) | `$existingFile, $modelClass, $field, $model` |
| `nano.fileupload.beforeDelete` | Start of `deleteFile` | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | End of `deleteFile` (after deletion) | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterAttach` | End of `attachTempFiles` (after attachment) | `$model, $field, $attachedFiles, $tempSessionKey` |
| `nano.fileupload.websocket.notify` | After `upload`, `deleteFile`, `attachTempFiles` (if WebSocket enabled) | `$channel, $event, $data` |

Developers can listen to these events to extend behavior (e.g., send notifications, additional logging, modify data).

---

### Comprehensive Practical Examples

#### Example 1: Upload a Main Image for a New Product Using a Temporary Session Key (with automatic resizing and watermarking)

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

$service = FileUploadService::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';

// 1. Create a temporary session key (includes userType and timestamp)
$tempKey = $service->generateTempSessionKey($modelClass, $field, $userManager->getId(), $userManager->getUserType());

// 2. Upload the image (automatic transformations will be applied according to field settings)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey,
    'title' => 'Product Image',
    'description' => 'Image description',
]);

if (!$uploadResult['status']) {
    die("Upload failed: " . $uploadResult['error'] . " (code: " . $uploadResult['error_code'] . ")");
}

// 3. Create and save the product
$product = new Product();
$product->name = 'New Product';
$product->save();

// 4. Attach the image to the product (service validates key and user)
$attachResult = $service->attachTempFiles($product, $field, $tempKey);
if (!$attachResult['status']) {
    die("Attachment failed: " . $attachResult['error']);
}

echo "Image uploaded and attached to product ID {$product->id}";
```

#### Example 2: Replace an Image for an Existing Product (Edit Operation)

```php
$product = Product::find(10); // Existing product
$newImageFile = $request->file('new_image');

$result = $service->upload(Product::class, 'image', $newImageFile, [
    'model' => $product, // Pass the saved model
]);

if ($result['status']) {
    echo "Image replaced successfully. Operation: " . $result['process_data']['operation']; // edit
    echo "Old file ID deleted: " . $result['process_data']['existing_file_deleted'];
} else {
    echo "Replacement failed: " . $result['error'];
}
```

#### Example 3: Upload Multiple Images for an Existing Product Gallery (with WebSocket)

```php
$product = Product::find(10); // Existing product

// Upload a set of images
$filesData = [$file1, $file2, $file3];
$result = $service->uploadMultiple(Product::class, 'gallery', $filesData, [
    'model' => $product, // Direct attachment
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " images";
    // An afterUpload event will be dispatched for each file, and WebSocket can notify the frontend
} else {
    echo "Some images failed to upload: " . $result['error'];
}
```

#### Example 4: Retrieve All Gallery Images for a Product with Custom Thumbnails

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

#### Example 5: Delete a Main Image After Permission Check (with Events)

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

#### Example 6: Validate a File Before Upload (Detects Dangerous PHP File)

```php
$maliciousFile = $request->file('upload'); // php file
try {
    $service->validateFile(Product::class, 'image', $maliciousFile);
    // Will not reach here
} catch (FileUploadException $e) {
    echo $e->getErrorCode(); // FILE_UPLOAD_FILE_TYPE_BLACKLISTED
    echo $e->getMessage();   // "File type php is not allowed for security reasons."
}
```

#### Example 7: Use the `afterEditDelete` Event to Log the Replacement

```php
Event::listen('nano.fileupload.afterEditDelete', function ($oldFile, $modelClass, $field, $model) {
    \Log::info("Replaced old file {$oldFile->id} with a new file for model {$model->id}");
});
```

#### Example 8: Integrate `FileUploadService` with an API Using Error Codes

In an API controller:

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

#### Example 9: Use WebSocket for Real-time Notifications

In `.env` file:
```ini
NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=true
NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads
```

Then listen for the event:
```php
Event::listen('nano.fileupload.websocket.notify', function ($channel, $event, $data) {
    broadcast(new \App\Events\FileUploadWebsocketEvent($channel, $event, $data));
});
```

---

### Interaction with Other Classes

- **With `FileUploadRegistry`**: Uses `$registry` to get field settings (`getFieldConfig`, `getFieldConstraints`, `getProcessingOptions`) and check permissions (`can`).
- **With `FileUploadUserManager`**: Uses `$userManager` to get the current user and their type (`getUser`, `getUserType`, `getId`).
- **With `Base64` (moved inside the add-on)**: Calls `Base64::onUpload`, `Base64::is_base64`, `Base64::base64ToUploadedFile`, `Base64::getBase64ImageSize`, `Base64::getDataInBase64`.
- **With `October\Rain\Resize\Resizer`**: For resizing images (if `auto_resize` is enabled).
- **With `Nano2\Watermark\Classes\WatermarkHelper`**: For watermarking (if the add-on exists and `auto_watermark` is enabled).

---

### Best Practices

1. **Always use a temporary session key for new models**: To avoid losing files if the model is not yet saved.
2. **For replacement (edit), use `model` in the options** instead of a temporary key, and let the service handle the transaction (`Db::transaction`) and old file deletion.
3. **Validate the file before upload using `validateFile`**: To provide clear error messages to the user (the service does this automatically, but you can also call it manually).
4. **Log transaction IDs** in the database to track upload operations.
5. **Use `uploadMultiple` for multiple fields** instead of calling `upload` in a loop, because `uploadMultiple` aggregates errors and provides statistics.
6. **Do not rely solely on `temp_session_key`**: After saving the model, use `attachTempFiles` to transfer ownership.
7. **Handle errors based on the code**: 400 (bad request), 403 (permission denied), 404 (not found), 422 (validation failed), 500 (internal error) – and use `error_code` for fine-grained differentiation.
8. **Enable `app.debug` in development environment** to get full error details.
9. **Use `with_thumbs` only when needed** to reduce response size and improve performance.
10. **Keep temporary session keys in the session or a temporary database** to avoid losing them.
11. **Test permissions thoroughly**: Ensure that unauthorized users cannot upload, delete, or replace files.
12. **Use automatic transformations (`auto_resize`, `auto_watermark`) wisely**: They may consume server resources, so enable them only when necessary.
13. **Monitor the `fileupload.log`** to detect attempts to upload dangerous files or recurring errors.
14. **Create a scheduled task (cron)** to clean up expired (`expires_at`) and unattached files.
15. **Use `skip_permission_check => true` only in tests**, not in production.

---

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `Field ... is not registered for model ...` | The field is not registered in `FileUploadRegistry`. | Add the field to the registration. |
| `You do not have permission to upload files for this field` | The user does not have `add` permission for the field, or `disable_upload` is enabled. | Check `permissions` settings in the Registry or global settings, and user permissions. |
| `You do not have permission to replace files for this field` | Attempting `edit` without `edit` permission or `disable_edit` is enabled. | Check `edit` permission in the Registry and global settings. |
| `Invalid file data` | The passed data is neither an `UploadedFile` nor a valid base64 string. | Ensure the data format is correct. |
| `File not found` (when deleting) | The file ID does not exist. | Verify the ID is correct. |
| `File size exceeds the allowed limit` | File size is larger than `max_filesize`. | Compress the file or increase `max_filesize` in field settings or defaults. |
| `File type not allowed` | File extension is not listed in `allowed_types`. | Use an allowed type or modify `allowed_types`. |
| `File type ... is not allowed for security reasons` | The extension is in the blacklist (`BLACKLISTED_EXTENSIONS`). | This block cannot be bypassed for security reasons; use safe file types. |
| `Temporary session key is invalid or expired` | The key has expired (more than `temp_key_ttl`) or has been tampered with. | Re-upload the file or use a new key. |
| `You are not authorized to access these temporary files` | Attempting to access temporary files belonging to another user or a different model/field. | Ensure the key belongs to the current user and the correct model/field. |
| `Auto-resize failed` | GD or Imagick library is not installed, or the file path is invalid. | Install `php-gd` or `php-imagick`, and check file permissions. |
| `Auto-watermark failed` | The `Nano2.Watermark` add-on is not installed or the logo path is incorrect. | Install the add-on and ensure the logo file exists at the specified path. |
| `Storage disk 's3' not found` | The `s3` disk is not configured in `config/filesystems.php`. | Add the appropriate disk configuration. |
| `The old file was deleted even though the new file upload failed` | (Rare) Transaction or database configuration issue. | Ensure you are using the InnoDB engine to support transactions. |

---

### Summary

The `FileUploadService` class is the backbone of file upload operations in NanoSoft applications. It provides a unified and secure interface for uploading, deleting, and retrieving files, with full support for different user types and granular permissions, temporary binding for unsaved models, automatic image transformations, multiple storage, event hooks, and WebSocket notifications. Starting from version 1.1.0, it also supports file replacement (edit) in `attachOne` relations using transactions (`Db::transaction`) and custom events, ensuring data integrity and security. By relying on `FileUploadRegistry` and `FileUploadUserManager`, developers can build robust and scalable file upload systems without worrying about implementation details or data security.

For additional advanced examples, refer to [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advenced-Examples-en.md).

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
