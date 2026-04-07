# FileUploadService Class Documentation

**Overview of the `Nano.FileUpload` package and the file upload management module**

This group of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, and others) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**. It is specifically designed for **NanoSoft applications** to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes of this module reside within the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, checking user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before saving the record, and handling errors uniformly. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

> **Note:** This document covers versions up to 1.0.7 and includes advanced features such as multi-storage, automatic image conversion, event hooks, WebSocket, and security improvements and new database fields.

---

### Introduction

The `FileUploadService` class is the **service layer** responsible for executing file upload, delete, and retrieval operations in NanoSoft applications. This class relies on `FileUploadRegistry` to access model settings and check permissions, and on `FileUploadUserManager` to manage the current user and verify their permissions.

Simply put, `FileUploadService` is the **engine that performs the actual upload operations**, ensuring that each operation complies with the registered settings and specified permissions, while supporting temporary uploads (via a temporary session key) to link files with unsaved models.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$registry` | `FileUploadRegistry` | Reference to the registered models registry. |
| `$userManager` | `FileUploadUserManager` | Reference to the user and permission manager. |
| `BLACKLISTED_EXTENSIONS` | `array` | List of dangerous extensions that are permanently forbidden (PHP, JS, HTML, EXE, etc.). |

---

### Key Methods

#### 1. Generate Temporary Session Key

##### `public function generateTempSessionKey(string $modelClass, string $field, $userId = null, $userType = null): string`
- **Purpose:** Generate a unique and secure temporary session key to link files before saving the model.
- **Parameters:**
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$userId`: (Optional) User ID; if not passed, uses the current user.
  - `$userType`: (Optional) User type (`backend`/`frontend`/`guest`).
- **Behavior:**
  - Gets the current user ID and type or uses the passed values.
  - Creates a key in the format `tmp_{base64(data)}:{hash}` where `data = modelClass|field|userId|userType|timestamp` and signed with HMAC-SHA256 using `app.key`.
- **Return:** The temporary session key.
- **Example:**
  ```php
  $tempKey = $service->generateTempSessionKey('Nano\Shop\Models\Product', 'image');
  ```

#### 2. Validate Temporary Session Key

##### `public function validateTempSessionKey(string $tempSessionKey): ?array`
- **Purpose:** Validate the temporary session key (signature, expiration, structure) and extract its data.
- **Parameters:** `$tempSessionKey` – The key to validate.
- **Return:** An array containing `modelClass`, `field`, `userId`, `userType`, `timestamp`, or `null` if invalid.
- **Behavior:**
  - Checks for the `tmp_` prefix.
  - Decodes and verifies the signature.
  - Checks key validity (default 3600 seconds, adjustable via `security.temp_key_ttl`).

#### 3. Upload Single File

##### `public function upload(string $modelClass, string $field, $fileData, array $options = []): array`
- **Purpose:** Upload a single file for a specific field.
- **Parameters:**
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$fileData`: File data (`UploadedFile` object or base64 string).
  - `$options`: Array of additional options:
    - `model`: Model object (if it exists and is saved).
    - `temp_session_key`: Temporary session key (to bypass automatic generation).
    - `title`, `description`: To specify file title and description.
    - `...`: Any additional options passed to `Base64::onUpload`.
- **Behavior:**
  - Checks if the model and field are registered in `FileUploadRegistry`.
  - Checks `add` permission for the current user (considering global settings like `disable_upload`).
  - Dispatches the `nano.fileupload.beforeUpload` event.
  - Validates the file (blacklist, size, types) via `validateFile`.
  - Determines whether the model exists and is saved (direct linking) or needs a temporary session key.
  - Calls `Base64::onUpload` to perform the actual upload.
  - **Applies automatic transformations** (resizing, watermarking) if the field is of type `image` and settings are enabled.
  - **Calculates SHA256** of the content and stores it in `hash` (if not already present).
  - **Stores image dimensions** in `meta` (as JSON) if the file is an image.
  - **Sets `expires_at`** for temporary files (24 hours) if the file is not linked.
  - **Assigns the custom storage disk** (if `storage_disk` is specified in field settings).
  - Saves the file (if not already saved or changes occurred).
  - Adds `temp_session_key` to the result if used.
  - Dispatches the `nano.fileupload.afterUpload` event and sends a WebSocket notification (if enabled).
- **Return:** An array with the same structure as `Base64::onUpload` plus additional fields: `code`, `status`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`, `error_code`.
- **Example:**
  ```php
  $result = $service->upload(
      'Nano\Shop\Models\Product',
      'image',
      $uploadedFile,
      ['temp_session_key' => $tempKey]
  );
  if ($result['status']) {
      echo "File uploaded successfully. ID: " . $result['data']['id'];
  } else {
      echo "Failed: " . $result['error'] . " (code: " . $result['error_code'] . ")";
  }
  ```

#### 4. Upload Multiple Files (for a multiple field)

##### `public function uploadMultiple(string $modelClass, string $field, array $filesData, array $options = []): array`
- **Purpose:** Upload multiple files for a multiple field in a single request.
- **Parameters:** Same as `upload`, but `$filesData` is an array of file data.
- **Behavior:** Iterates over `$filesData` and calls `upload()` for each file, then aggregates results and calculates success count.
- **Return:** An array containing `code`, `status`, `message`, `data` (results for each file), `process_data` (success/total count).
- **Example:**
  ```php
  $files = [ $file1, $file2, $file3 ];
  $result = $service->uploadMultiple('Product', 'gallery', $files, ['model' => $product]);
  echo "Successfully uploaded " . $result['process_data']['success_count'] . " out of " . $result['process_data']['total'];
  ```

#### 5. Delete File

##### `public function deleteFile(int $fileId, ?string $modelClass = null, ?string $field = null, $user = null): array`
- **Purpose:** Delete a specific file with permission checking.
- **Parameters:**
  - `$fileId`: File ID.
  - `$modelClass`: (Optional) Class name to check delete permission.
  - `$field`: (Optional) Field name to check delete permission.
  - `$user`: (Optional) User object.
- **Behavior:**
  - Dispatches the `nano.fileupload.beforeDelete` event.
  - Finds the file using `File::find()`.
  - If `$modelClass` and `$field` are provided, checks `delete` permission for the current user (considering `disable_delete`).
  - Deletes the file.
  - Dispatches the `nano.fileupload.afterDelete` event and sends a WebSocket notification.
- **Return:** An array containing `code`, `status`, `message`, `error`, `error_code`, etc.
- **Example:**
  ```php
  $result = $service->deleteFile(123, 'Nano\Shop\Models\Product', 'image');
  if ($result['status']) {
      echo "File deleted successfully";
  }
  ```

#### 6. Retrieve Files

##### `public function getFiles(string $modelClass, string $field, ?int $modelId = null, array $options = []): array`
- **Purpose:** Retrieve files associated with a specific model and field, or temporary files.
- **Parameters:**
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$modelId`: (Optional) ID of the saved model.
  - `$options`: Additional options:
    - `temp_session_key`: Temporary session key (to retrieve unlinked files).
    - `with_thumbs`: `bool` to include thumbnails.
    - `thumb_sizes`: Array of thumbnail sizes (e.g., `['thumb' => [100,100]]`).
- **Behavior:**
  - Checks if the field is registered and the `view` permission (considering `disable_get`).
  - Builds a query on `File` filtered by `field` and (`attachment_id` + `attachment_type` if `modelId` exists) or `session_key` if `temp_session_key` exists.
  - **When using `temp_session_key`**, validates the key (signature, expiration, matching user, model, field) before executing the query.
  - Returns files with basic data (id, title, description, path, size, content_type) and thumbnails if requested.
- **Return:** An array containing `code`, `status`, `message`, `data`.
- **Example:**
  ```php
  $files = $service->getFiles('Nano\Shop\Models\Product', 'image', 456, ['with_thumbs' => true]);
  foreach ($files['data'] as $file) {
      echo "<img src='{$file['thumb']}'>";
  }
  ```

#### 7. Attach Temporary Files to Saved Model

##### `public function attachTempFiles(Model $model, string $field, string $tempSessionKey): bool`
- **Purpose:** Transfer files uploaded using a temporary session key to a saved model.
- **Parameters:**
  - `$model`: The saved model object.
  - `$field`: Field name.
  - `$tempSessionKey`: The temporary session key used during upload.
- **Behavior:**
  - Validates the key (signature, expiration).
  - Checks that the `modelClass` and `field` from the key match the model.
  - Checks that the current user is the same user who created the key (same `userId` and `userType`).
  - Finds files with `session_key = $tempSessionKey` and `field = $field`.
  - For each file, updates `attachment_id` and `attachment_type` to link it to the model and removes `session_key`.
  - If the model has a relationship for this field, uses `add()`; otherwise sets the field directly.
- **Return:** `true` if files were attached, otherwise `false`.
- **Example:**
  ```php
  $product = new Product();
  $product->name = 'New product';
  $product->save();
  $service->attachTempFiles($product, 'image', $tempKey);
  ```

#### 8. Validate File

##### `public function validateFile(string $modelClass, string $field, $fileData): void`
- **Purpose:** Ensure the file matches the registered field constraints (max size, allowed types) as well as the blacklist.
- **Parameters:**
  - `$modelClass`: Fully qualified class name of the model.
  - `$field`: Field name.
  - `$fileData`: File data (`UploadedFile` object or base64 string).
- **Behavior:**
  - Calls `getFieldConstraints()` from `FileUploadRegistry`.
  - **Checks the blacklist first** (dangerous extensions).
  - Checks file size (compares with `max_filesize`).
  - Checks file extension against `allowed_types`.
- **Return:** None, but throws `FileUploadException` with an appropriate error code if validation fails, and logs the failed attempt in `fileupload.log`.
- **Example:**
  ```php
  try {
      $service->validateFile('Product', 'image', $uploadedFile);
      // File is valid
  } catch (FileUploadException $e) {
      echo "Invalid file: " . $e->getMessage() . " (code: " . $e->getErrorCode() . ")";
  }
  ```

#### 9. Helper Methods (Storage, Transformation, Notifications)

##### `protected function getStorageDiskForField($modelClass, $field): ?string`
- Determines the custom storage disk name for the field (from `getProcessingOptions`) and checks its existence.

##### `protected function applyAutoProcessing($file, $modelClass, $field): void`
- Applies automatic resizing and watermarking to images according to field options and global settings.

##### `protected function notifyWebSocket($event, $data): void`
- Dispatches the `nano.fileupload.websocket.notify` event if WebSocket is enabled in the settings.

---

### Events

| Event Name | Call Location | Parameters |
|------------|---------------|------------|
| `nano.fileupload.beforeUpload` | Start of `upload` | `$modelClass, $field, $fileData, &$options` |
| `nano.fileupload.afterUpload` | End of `upload` (after success) | `$file, $modelClass, $field, $options` |
| `nano.fileupload.beforeDelete` | Start of `deleteFile` | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | End of `deleteFile` (after deletion) | `$fileId, $modelClass, $field` |
| `nano.fileupload.websocket.notify` | After `upload` or `delete` (if WebSocket enabled) | `$channel, $event, $data` |

Developers can listen to these events to extend behavior (e.g., send notifications, additional logging, modify data).

---

### Comprehensive Practical Examples

#### Example 1: Upload a main image for a new product using a temporary session key (with automatic resizing and watermarking)

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

$service = FileUploadService::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';

// 1. Generate a temporary session key (includes userType and timestamp)
$tempKey = $service->generateTempSessionKey($modelClass, $field, $userManager->getId(), $userManager->getUserType());

// 2. Upload the image (automatic transformations will be applied according to field settings)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey,
    'title' => 'Product image',
    'description' => 'Image description',
]);

if (!$uploadResult['status']) {
    die("Upload failed: " . $uploadResult['error'] . " (code: " . $uploadResult['error_code'] . ")");
}

// 3. Create and save the product
$product = new Product();
$product->name = 'New product';
$product->save();

// 4. Attach the image to the product (the service checks the key and user)
$service->attachTempFiles($product, $field, $tempKey);

echo "Image uploaded and attached to product ID {$product->id}";
```

#### Example 2: Upload multiple images for an existing product gallery (with WebSocket)

```php
$product = Product::find(10); // Existing product

// Upload a set of images
$filesData = [$file1, $file2, $file3];
$result = $service->uploadMultiple(Product::class, 'gallery', $filesData, [
    'model' => $product, // direct linking
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " images";
    // afterUpload event will be triggered for each file, and WebSocket can notify the frontend
} else {
    echo "Failed to upload some images: " . $result['error'];
}
```

#### Example 3: Retrieve all images for a product gallery with custom thumbnails

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

#### Example 4: Delete a main image after permission check (with events)

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

#### Example 5: Validate a file before upload (detects dangerous PHP file)

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

#### Example 6: Use `afterUpload` event to send an email

```php
Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
    // Send notification to admin
    Mail::to('admin@example.com')->send(new FileUploadedNotification($file));
});
```

#### Example 7: Integrate `FileUploadService` with an API using error codes

In an API controller:

```php
public function uploadImage(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $file = $request->file('file');

    $service = FileUploadService::instance();
    $result = $service->upload($modelClass, $field, $file, [
        'temp_session_key' => $request->input('temp_session_key')
    ]);

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

#### Example 8: Use WebSocket for instant notifications

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
- **With `FileUploadUserManager`**: Uses `$userManager` to get the current user and type (`getUser`, `getUserType`, `getId`).
- **With `Base64` (from `Nano.Api`)**: Calls `Base64::onUpload`, `Base64::is_base64`, `Base64::base64ToUploadedFile`, `Base64::getBase64ImageSize`, `Base64::getDataInBase64`.
- **With `October\Rain\Resize\Resizer`**: For image resizing (if `auto_resize` is enabled).
- **With `Nano2\Watermark\Classes\WatermarkHelper`**: For watermarking (if the plugin exists and `auto_watermark` is enabled).

---

### Best Practices

1. **Always use a temporary session key for new models**: to avoid losing files if the model is not saved yet.
2. **Validate the file before upload using `validateFile`**: to provide clear error messages to the user (the service does this automatically, but you can also call it manually).
3. **Log transaction IDs** in the database to track upload operations.
4. **Use `uploadMultiple` for multiple fields** instead of calling `upload` in a loop, because `uploadMultiple` aggregates errors and provides statistics.
5. **Do not rely solely on `temp_session_key`**: after saving the model, use `attachTempFiles` to transfer ownership.
6. **Handle errors by code**: 400 (bad request), 403 (forbidden), 404 (not found), 500 (internal error) – and use `error_code` for fine-grained differentiation.
7. **Enable `app.debug` in development environment** to get full error details.
8. **Use `with_thumbs` only when needed** to reduce response size and increase performance.
9. **Keep temporary session keys in the session or temporary database** to avoid losing them.
10. **Test permissions thoroughly**: ensure unauthorized users cannot upload or delete files.
11. **Use automatic transformations (`auto_resize`, `auto_watermark`) wisely**: they may consume server resources, so enable them only when necessary.
12. **Monitor the `fileupload.log`** to detect attempts to upload dangerous files or recurring errors.
13. **Create a scheduled cron task** to clean up expired (`expires_at`) and unlinked files.

---

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `Field ... is not registered for model ...` | The field is not registered in `FileUploadRegistry`. | Add the field to the registry. |
| `You do not have permission to upload files for this field` | The user does not have the `add` permission for the field, or `disable_upload` is active. | Check `permissions` settings in the Registry or global settings, and user permissions. |
| `Invalid file data` | The passed data is neither an `UploadedFile` nor a valid base64 string. | Verify the data format. |
| `File not found` (when deleting) | The file ID does not exist. | Ensure the ID is correct. |
| `File size exceeds allowed limit` | File size is larger than `max_filesize`. | Compress the file or increase `max_filesize` in field settings or default settings. |
| `File type not allowed` | File extension is not listed in `allowed_types`. | Use an allowed type or modify `allowed_types`. |
| `File type ... is not allowed for security reasons` | The extension is in the blacklist (`BLACKLISTED_EXTENSIONS`). | This cannot be bypassed for security reasons; use safe file types. |
| `Invalid or expired temporary session key` | The key has expired (more than `temp_key_ttl`) or has been tampered with. | Re-upload the file or use a new key. |
| `You are not authorized to access these temporary files` | Attempting to access temporary files of another user or with a different model/field. | Ensure the key belongs to the current user and the correct model/field. |
| `Auto-resize failed` | GD or Imagick library is not installed, or the file path is invalid. | Install `php-gd` or `php-imagick`, and check file permissions. |
| `Auto-watermark failed` | The `Nano2.Watermark` plugin is not installed or the logo path is incorrect. | Install the plugin and ensure the logo file exists at the specified path. |
| `Storage disk 's3' not found` | The `s3` disk is not configured in `config/filesystems.php`. | Add the appropriate disk configuration. |

---

### Conclusion

The `FileUploadService` class is the backbone of file upload operations in NanoSoft applications. It provides a unified and secure interface for uploading, deleting, and retrieving files, with full support for different user types and granular permissions, temporary linking for unsaved models, automatic image transformations, multi-storage, event hooks, and WebSocket notifications. Thanks to its reliance on `FileUploadRegistry` and `FileUploadUserManager`, developers can build robust and scalable file upload systems without worrying about implementation details or data security.

For additional advanced examples, see the [Advanced Examples Documentation for `FileUploadService` Class](./Docs-FileUploadService-Class-Advenced-Examples.md).

## Additional Documentation

- [General Add-on Documentation](./Docs-FileUpload.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advenced-Examples.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advenced-Examples.md)
- [API Documentation](./Docs-API-Documentation.md)

