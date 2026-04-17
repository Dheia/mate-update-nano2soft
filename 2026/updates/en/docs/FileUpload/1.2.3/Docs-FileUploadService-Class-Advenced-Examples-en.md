# Advanced and Practical Examples for `FileUploadService` Class Documentation

**Namespace:** `Nano\FileUpload\Classes`  
**Purpose:** Provide comprehensive practical examples for using `FileUploadService` in various scenarios, from basic file upload operations to handling temporary files, attaching to models, checking permissions and constraints, as well as advanced features from versions 1.0.6 and 1.0.7: multiple storage, automatic image transformation, event hooks, WebSocket, calculation of `hash`, `meta`, and `expires_at`, and handling `FileUploadException`.

> **Note:** All examples assume that the `FileUploadService` class is initialized via `FileUploadService::instance()`.

---

## Introduction

This document provides a set of practical examples covering various use cases of the `FileUploadService` class, which represents the **Service Layer** responsible for executing file upload, delete, and retrieval operations in NanoSoft applications. This class depends on `FileUploadRegistry` to access model settings and check permissions, and on `FileUploadUserManager` to manage the current user and check their permissions.

Through these examples, you will learn how to:

- Upload a single file or multiple files using `UploadedFile` data or base64.
- Use temporary session keys (`temp_session_key`) to attach files to unsaved models.
- Attach temporary files to a model after saving it.
- Retrieve files associated with a model, including thumbnails.
- Delete files with permission checking.
- Validate files (size, type) before upload.
- Handle various errors (permissions, constraints, server errors).
- Use multiple storage (S3, FTP, Local disks).
- Apply automatic transformations (resizing and watermarking) to images.
- Listen to events (`beforeUpload`, `afterUpload`, `beforeDelete`, `afterDelete`).
- Send WebSocket notifications when operations complete.
- Handle unique error codes (`error_code`) from `FileUploadException`.
- Leverage new fields (`hash`, `meta`, `expires_at`).

---

## Prerequisites

- `Nano.FileUpload` add-on installed and configured in your application (version 1.0.7 or later).
- The add-on depends on `Nano.Api` which provides `Base64` and `ApiController`.
- Models and upload fields have been registered in `FileUploadRegistry` as described in [Advanced Examples for FileUploadRegistry Class](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md).
- Basic knowledge of `UploadedFile` objects and how to obtain them from requests (e.g., via `$request->file('file')`).

### Setting up the class in the application

```php
use Nano\FileUpload\Classes\FileUploadService;

$service = FileUploadService::instance();
```

Since the class follows the Singleton pattern, it is called via `instance()`.

---

## 1. Upload a Single File Using UploadedFile Data (Direct Attachment to Existing Model)

**Scenario:** An existing product, we want to upload a main image and attach it directly.

```php
$product = Product::find(10);
$uploadedFile = $request->file('image'); // UploadedFile object

$result = $service->upload(Product::class, 'image', $uploadedFile, [
    'model' => $product, // direct attachment
    'title' => 'Product Image',
    'description' => 'Image description',
]);

if ($result['status']) {
    echo "Image uploaded successfully. File ID: " . $result['data']['id'];
    echo "Path: " . $result['data']['path'];
} else {
    echo "Upload failed: " . $result['error'] . " (code: " . $result['error_code'] . ")";
}
```

**Expected Output (on success):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'File uploaded successfully',
    'data' => [
        'id' => 123,
        'path' => '/storage/app/uploads/public/...',
        'thumb' => '/storage/app/uploads/public/...',
    ],
    'process_data' => [
        'model_class' => 'Nano\Shop\Models\Product',
        'field' => 'image',
        'temp_session_key' => null,
        'user_type' => 'backend',
        'storage_disk' => null,
        // ... additional data
    ],
]
```

**Notes:**  
- When an existing saved model (`$product->exists`) is passed, the attachment is done directly without using a temporary session key.  
- You can customize `title` and `description` to add them to the file.

---

## 2. Upload a Single File Using Base64 (Unsaved Model)

**Scenario:** Creating a new product, we want to upload its image before saving the product, then attach it later.

```php
$base64String = 'data:image/jpeg;base64,/9j/4AAQSkZJRg...';

$modelClass = Product::class;
$field = 'image';

// Generate a temporary session key (the service will automatically add userType and timestamp)
$tempKey = $service->generateTempSessionKey($modelClass, $field);

// Upload the image
$result = $service->upload($modelClass, $field, $base64String, [
    'temp_session_key' => $tempKey,
]);

if ($result['status']) {
    // Save the product
    $product = new Product();
    $product->name = 'New Product';
    $product->save();

    // Attach the image to the product
    $service->attachTempFiles($product, $field, $tempKey);

    echo "Image uploaded and attached to product ID {$product->id}";
} else {
    echo "Upload failed: " . $result['error'];
}
```

**Output:**  
- `upload` returns `temp_session_key` in `$result['temp_session_key']` if used.  
- `attachTempFiles` returns `true` if files were attached, otherwise `false`.  
- **Automatically:** `hash` (SHA256) of the content is calculated, image dimensions are stored in `meta`, and `expires_at` is set for the temporary file (24 hours).

**Notes:**  
- `generateTempSessionKey` is not mandatory; if you don't pass a `temp_session_key`, `upload` will generate one automatically and return it in the result.  
- You must save the `temp_session_key` in the user's session or a temporary database to use it later.

---

## 3. Upload an Image with Automatic Resizing and Watermarking (According to Field Settings)

**Scenario:** After registering the field with `auto_resize` and `auto_watermark` (as in the `FileUploadRegistry` example), the transformations are applied automatically during upload.

```php
// Field definition in registry (once)
// 'image' => [
//     'type' => 'image',
//     'auto_resize' => true,
//     'resize_options' => ['width' => 800, 'height' => 600, 'mode' => 'crop'],
//     'auto_watermark' => true,
//     'watermark_options' => ['position' => 'bottom-right', 'resize_percentage' => 15],
// ]

$result = $service->upload(Product::class, 'image', $uploadedFile, [
    'model' => $product,
]);
// After upload, the image will be resized and watermarked.
```

**Verifying that transformations were applied:** You can monitor the logs (`storage/logs/laravel.log` or `fileupload.log`) to ensure success. If a failure occurs (e.g., missing GD library), an error will appear in the log with `Auto-resize failed`.

**Notes:**  
> **Note:** Automatic resizing and watermarking can be globally disabled via `disable_auto_resize` and `disable_auto_watermark` in global settings.

---

## 4. Upload Multiple Files for a Multiple Field (Gallery)

**Scenario:** Upload a set of images for an existing product's gallery.

```php
$product = Product::find(10);
$files = $request->file('gallery'); // array of UploadedFile

$result = $service->uploadMultiple(Product::class, 'gallery', $files, [
    'model' => $product, // direct attachment
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " out of " . $result['process_data']['total'] . " images.";
    foreach ($result['data'] as $index => $fileResult) {
        if ($fileResult['status']) {
            echo "Image $index: ID {$fileResult['data']['id']}\n";
        } else {
            echo "Image $index failed: {$fileResult['error']} (code: {$fileResult['error_code']})\n";
        }
    }
} else {
    echo "All images failed to upload: " . $result['error'];
}
```

**Expected Output (partial `$result`):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => '2 out of 3 files uploaded successfully',
    'data' => [
        0 => ['status' => true, 'data' => ['id' => 101, ...]],
        1 => ['status' => true, 'data' => ['id' => 102, ...]],
        2 => ['status' => false, 'error' => 'File type not allowed', 'error_code' => 'FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED'],
    ],
    'process_data' => [
        'success_count' => 2,
        'total' => 3,
    ],
]
```

---

## 5. Retrieve Files for a Specific Field with Thumbnails

**Scenario:** Display all gallery images of a product with thumbnails of different sizes.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
        'large' => [800, 600, 'auto'],
    ],
]);

if ($result['status']) {
    foreach ($result['data'] as $image) {
        echo "<img src='{$image['small']}' alt='{$image['title']}'>";
        echo "<a href='{$image['large']}'>View large size</a>";
    }
} else {
    echo "Error: " . $result['error'];
}
```

**Output (example for one file):**

```php
[
    'id' => 101,
    'title' => 'Product Image',
    'description' => null,
    'path' => '/storage/app/uploads/public/.../original.jpg',
    'size' => 123456,
    'content_type' => 'image/jpeg',
    'small' => '/storage/app/uploads/public/.../small.jpg',
    'medium' => '/storage/app/uploads/public/.../medium.jpg',
    'large' => '/storage/app/uploads/public/.../large.jpg',
]
```

---

## 6. Retrieve Temporary Files (Unattached) with Security Check

**Scenario:** After uploading files using a temporary session key, we want to display them before saving the model. The service validates the key and matches the user, model, and field.

```php
$tempKey = $request->input('temp_session_key');
$result = $service->getFiles(Product::class, 'gallery', null, [
    'temp_session_key' => $tempKey,
]);

if ($result['status']) {
    echo "Number of temporary files: " . count($result['data']);
    foreach ($result['data'] as $file) {
        echo "<img src='{$file['path']}'>";
    }
} else {
    echo "Failed to retrieve temporary files: " . $result['error'];
}
```

**Security Notes:**  
- If the key is expired or has been tampered with, the service will reject the request with an error message.  
- If another user tries to use another user's key, the service will reject the request with code `FILE_UPLOAD_TEMP_KEY_MISMATCH`.

---

## 7. Delete a File with Permission Check and Events

**Scenario:** A user wants to delete a product's main image; we check their permission first and use events to log the operation.

```php
Event::listen('nano.fileupload.beforeDelete', function ($fileId, $modelClass, $field) {
    \Log::info("Attempting to delete file: {$fileId} from model {$modelClass}");
});

Event::listen('nano.fileupload.afterDelete', function ($fileId, $modelClass, $field) {
    \Log::info("File deleted: {$fileId}");
});

$result = $service->deleteFile(123, Product::class, 'image');

if ($result['status']) {
    echo "File deleted successfully";
} else {
    echo "Deletion failed: " . $result['message'] . " (code: " . $result['error_code'] . ")";
}
```

**Notes:**  
- If `$modelClass` and `$field` are not passed, permission will not be checked.  
- The check uses the `delete` permission from the field or model settings, as well as the global setting `disable_delete`.

---

## 8. Validate a File Before Upload (Detects Dangerous Files)

**Scenario:** Before uploading a file, we check its size, type, and the blacklist of dangerous extensions. If the file is of type PHP, it will be rejected with a specific error code.

```php
$uploadedFile = $request->file('image');

try {
    $service->validateFile(Product::class, 'image', $uploadedFile);
    // File is valid
    $result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
} catch (FileUploadException $e) {
    echo "File invalid: " . $e->getMessage();
    echo "Error code: " . $e->getErrorCode(); // e.g., FILE_UPLOAD_FILE_TYPE_BLACKLISTED
    echo "Context: " . print_r($e->getContext(), true);
}
```

**Common exceptions:**  
- `FILE_UPLOAD_FILE_SIZE_EXCEEDED` – File size exceeds the allowed limit.  
- `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` – File extension not allowed.  
- `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` – Dangerous extension (PHP, JS, HTML, EXE, etc.) – cannot be bypassed.

---

## 9. Handling Permission Errors and Global Settings

**Scenario:** Attempt to upload a file without permission or when uploads are globally disabled.

```php
$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);

if (!$result['status']) {
    switch ($result['error_code']) {
        case 'FILE_UPLOAD_PERMISSION_DENIED':
            echo "You do not have permission to upload files for this field";
            break;
        case 'FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY':
            echo "File upload operations are currently disabled";
            break;
        case 'FILE_UPLOAD_FIELD_NOT_REGISTERED':
            echo "Field not registered";
            break;
        default:
            echo "Error: " . $result['error'];
    }
}
```

**HTTP status codes used:**  
- `400`: Invalid data or field constraints.  
- `403`: Permission denied (including globally disabled operations).  
- `404`: Model or field not registered.  
- `500`: Internal error (details appear in debug).

---

## 10. Using `beforeUpload` and `afterUpload` Events to Extend Behavior

**Scenario:** Log upload attempts, modify options, or send notifications.

```php
// Before upload
Event::listen('nano.fileupload.beforeUpload', function ($modelClass, $field, &$fileData, &$options) {
    \Log::info("Attempting to upload file: {$modelClass} - {$field}");
    // Add an extra option
    $options['custom_flag'] = true;
    // $fileData or $options can be modified as needed
});

// After upload
Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
    \Log::info("File uploaded: {$file->id} for model {$modelClass}");
    // Send a notification via email or WebSocket
    broadcast(new FileUploadedEvent($file));
});

$result = $service->upload(Product::class, 'image', $fileData);
```

---

## 11. Using WebSocket for Real-time Notifications

**Scenario:** Enable WebSocket in settings, then listen to the `nano.fileupload.websocket.notify` event to send notifications to the frontend.

**Settings in `.env`:**
```ini
NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=true
NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads
```

**Code in the application:**
```php
Event::listen('nano.fileupload.websocket.notify', function ($channel, $event, $data) {
    // Use a WebSocket library such as Pusher or Laravel WebSockets
    broadcast(new \App\Events\FileUploadWebsocketEvent($channel, $event, $data));
});

$result = $service->upload(Product::class, 'image', $uploadedFile);
// After successful upload, the event will be dispatched and the notification sent.
```

---

## 12. Upload a Temporary File and Use It for Multiple Models

**Scenario:** A file is uploaded temporarily (e.g., a profile picture) and then attached to more than one model (e.g., a user and an article).

```php
// Step 1: Upload the image with a temporary key
$tempKey = $service->generateTempSessionKey(User::class, 'avatar');
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['temp_session_key' => $tempKey]);

if (!$result['status']) die('Upload failed');

// Step 2: Save the user and the article, and attach the same image
$user = new User();
$user->name = 'Ahmed';
$user->save();

$article = new Article();
$article->title = 'New Article';
$article->save();

// Attach the image to the user
$service->attachTempFiles($user, 'avatar', $tempKey);

// Attach the same image to the article (if the article has an 'image' field)
$service->attachTempFiles($article, 'image', $tempKey);

// Note: attachTempFiles searches for all files with that key and will attach all existing files.
// If you want to attach the same file to both models, the attachment will be duplicated (ensure files are independent).
```

---

## 13. Using Multiple Storage via `storage_disk`

**Scenario:** Upload a file directly to an S3 disk (or FTP) instead of the local disk.

```php
// Field definition with storage_disk (in registry)
// 'image' => ['type' => 'image', 'storage_disk' => 's3']

$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
// $file->disk will be set to 's3' before saving, and the custom disk will be used.
```

**Note:** The `s3` disk must be pre-configured in `config/filesystems.php`.

---

## 14. Upload Multiple Files Using Base64 Data from a Single Request

**Scenario:** An API receives an array of base64 strings and uploads them all.

```php
$base64Array = $request->input('images'); // ['data:image/...', 'data:image/...', ...]

$result = $service->uploadMultiple(Product::class, 'gallery', $base64Array, [
    'model' => $product,
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " images.";
}
```

---

## 15. Add Custom Thumbnail Sizes When Retrieving Files

**Scenario:** We want custom thumbnail sizes different from the default ones.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'icon' => [50, 50, 'crop'],
        'preview' => [200, 150, 'auto'],
        'full' => [800, 600, 'auto'],
    ],
]);

// The result will contain keys icon, preview, full for each file.
```

---

## 16. Handling Delete Errors (File Not Found)

```php
$result = $service->deleteFile(999999);

if (!$result['status']) {
    switch ($result['error_code']) {
        case 'FILE_UPLOAD_FILE_NOT_FOUND':
            echo "File not found";
            break;
        default:
            echo "Error: " . $result['message'];
    }
}
```

---

## 17. Integrating `FileUploadService` with `FileUploadUserManager` to Customize Frontend Permissions

**Scenario:** A frontend user wants to upload a profile picture; we use `FileUploadUserManager` to check a custom permission.

```php
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();

if (!$user) {
    return response()->json(['error' => 'Unauthorized'], 401);
}

// Assuming the user model is registered in the Registry with add permission for avatar
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['model' => $user]);
```

---

## 18. Using `validateFile` with Base64 Data in an API

**Scenario:** Before uploading an image via base64, validate it.

```php
$base64 = $request->input('image_base64');

try {
    $service->validateFile(Product::class, 'image', $base64);
    // File is valid
    $result = $service->upload(Product::class, 'image', $base64, ['model' => $product]);
} catch (FileUploadException $e) {
    return response()->json([
        'error' => $e->getMessage(),
        'error_code' => $e->getErrorCode(),
    ], 400);
}
```

---

## 19. Retrieve Temporary Files with Full Security Check

**Scenario:** After uploading temporary files, we want to display them before saving the model, ensuring that the current user is the owner of the key.

```php
$tempKey = $request->input('temp_session_key');
$result = $service->getFiles(Product::class, 'gallery', null, ['temp_session_key' => $tempKey]);

if ($result['status']) {
    // The files belong to the current user
    foreach ($result['data'] as $file) {
        echo "File: {$file['path']}<br>";
    }
} else {
    echo "Error: " . $result['error']; // Could be due to invalid key, expired, or belonging to another user
}
```

---

## 20. Best Practices for Versions 1.0.6+

1. **Always use temporary session keys for new models**: To avoid losing files if the model is not saved.
2. **Validate the file before upload using `validateFile`**: The service does this automatically, but you can also call it manually.
3. **Log transaction IDs** in the database to track upload operations.
4. **Use `uploadMultiple` for multiple fields** instead of calling `upload` in a loop, because `uploadMultiple` aggregates errors and provides statistics.
5. **Do not rely solely on `temp_session_key`**: After saving the model, use `attachTempFiles` to transfer ownership.
6. **Handle errors based on the code and `error_code`**: Provide appropriate messages to the user for each case.
7. **Enable `app.debug` in development environment** to get full error details.
8. **Use `with_thumbs` only when needed** to reduce response size and improve performance.
9. **Keep temporary session keys in the session or a temporary database** to avoid losing them.
10. **Test permissions thoroughly**: Ensure unauthorized users cannot upload or delete files.
11. **Use automatic transformations (`auto_resize`, `auto_watermark`) wisely** – they may consume server resources.
12. **Monitor the `fileupload.log`** to detect attempts to upload dangerous files or recurring errors.
13. **Create a scheduled task (cron)** to clean up expired (`expires_at`) and unattached files.

---

## 21. Common Errors and Solutions (Updated)

| Error | Cause | Solution |
|-------|-------|----------|
| `ApplicationException: Field ... is not registered for model ...` | The field is not registered in `FileUploadRegistry`. | Add the field to the registration. |
| `You do not have permission to upload files for this field` | The user does not have `add` permission for the field, or `disable_upload` is enabled. | Check `permissions` settings in the Registry or global settings, and user permissions. |
| `Invalid file data` | The passed data is neither an `UploadedFile` nor a valid base64 string. | Ensure the data format is correct. |
| `File not found` (when deleting) | The file ID does not exist. | Verify the ID is correct. |
| `File size exceeds the allowed limit` | File size is larger than `max_filesize`. | Compress the file or increase `max_filesize`. |
| `File type not allowed` | File extension is not listed in `allowed_types`. | Use an allowed type or modify `allowed_types`. |
| `File type ... is not allowed for security reasons` | The extension is in the blacklist (`BLACKLISTED_EXTENSIONS`). | Cannot be bypassed; use safe file types. |
| `Temporary session key is invalid or expired` | The key has expired or has been tampered with. | Re-upload the file or use a new key. |
| `You are not authorized to access these temporary files` | Attempting to access temporary files belonging to another user or a different model/field. | Ensure the key belongs to the current user and the correct model/field. |
| `Auto-resize failed` | GD or Imagick library is not installed, or the file path is invalid. | Install `php-gd` or `php-imagick`, and check file permissions. |
| `Auto-watermark failed` | The `Nano2.Watermark` add-on is not installed or the logo path is incorrect. | Install the add-on and ensure the logo file exists. |
| `Storage disk 's3' not found` | The `s3` disk is not configured in `config/filesystems.php`. | Add the appropriate disk configuration. |

---

## 22. Integration with Custom Permission System

In NanoSoft applications, you can customize the permission checker function for frontend users via `FileUploadUserManager`:

```php
FileUploadUserManager::instance()->setPermissionChecker(function ($user, $operation, $permissions) {
    // Custom logic: e.g., check a custom permissions table
    if ($user->hasRole('editor')) {
        return true;
    }
    return $user->hasPermission($permissions);
});
```

---

## 23. Replace an Existing File in an `attachOne` Relation (Edit Operation) – New in version 1.1.0

**Scenario:** An existing product has a main image; we want to replace it with a new image. The service will automatically delete the old image after successfully uploading the new one, using a database transaction to ensure integrity.

```php
$product = Product::find(10);
$newImage = $request->file('new_image');

$result = $service->upload(Product::class, 'image', $newImage, [
    'model' => $product, // pass the saved model
]);

if ($result['status']) {
    echo "Image replaced successfully. Operation: " . $result['process_data']['operation']; // 'edit'
    echo "Deleted old file ID: " . $result['process_data']['existing_file_deleted'];
} else {
    echo "Replacement failed: " . $result['error'] . " (code: " . $result['error_code'] . ")";
}
```

**Success Output:** `process_data` contains `operation` and `existing_file_deleted`.

**Notes:**  
- If the new file upload fails (e.g., size too large or type not allowed), the old file will not be deleted thanks to the transaction.  
- The user must have `edit` permission on the field (or model) and `disable_edit` must not be globally enabled.

## 24. Using Edit Operation Specific Events (`beforeEditDelete` and `afterEditDelete`) – New in version 1.1.0

**Scenario:** Log the image replacement operation and send a notification.

```php
Event::listen('nano.fileupload.beforeEditDelete', function ($oldFile, $modelClass, $field, $model) {
    \Log::info("About to delete old file {$oldFile->id} for model {$model->id}");
});

Event::listen('nano.fileupload.afterEditDelete', function ($oldFile, $modelClass, $field, $model) {
    \Log::info("Deleted old file {$oldFile->id}");
    // Send notification to admin
});

// Then perform the upload as usual
$result = $service->upload(Product::class, 'image', $newImage, ['model' => $product]);
```

## 25. Attaching Temporary Files Using Improved `attachTempFiles` (Unified Response) – New in version 1.1.0

**Scenario:** Upload temporary files and then attach them to a saved model, getting operation details.

```php
$tempKey = $service->generateTempSessionKey(Product::class, 'gallery');
// Upload multiple files...
$uploadResult = $service->uploadMultiple(Product::class, 'gallery', $files, ['temp_session_key' => $tempKey]);

// After saving the product
$product = new Product();
$product->name = 'New Product';
$product->save();

$attachResult = $service->attachTempFiles($product, 'gallery', $tempKey);

if ($attachResult['status']) {
    echo "Successfully attached " . $attachResult['process_data']['attached_count'] . " files";
    foreach ($attachResult['data'] as $file) {
        echo "File: {$file['id']} - {$file['path']}\n";
    }
} else {
    echo "Attachment failed: " . $attachResult['error'];
}
```

**Output of `attachResult`:** Unified structure containing `code`, `status`, `message`, `data` (attached files), `process_data` (number of files, field, model).

## 26. Skipping Permission Check in Tests Using `skip_permission_check` – New in version 1.1.0

**Scenario:** In a test environment, we want to upload files without setting real permissions.

```php
$result = $service->upload(Product::class, 'image', $file, [
    'model' => $product,
    'skip_permission_check' => true, // bypasses any permission checks
]);
```

**Notes:** This option is only for tests and should not be used in production.

#### 12. Using Transactions (`Db::transaction`) in `edit` Operation to Ensure Integrity

**Scenario:** Illustrate how the transaction maintains data integrity when the new file upload fails.

```php
// This code is internal to `upload`, but can be tested by simulating a failure (e.g., uploading a file that is too large)
$oldFileId = $product->image->id;
$result = $service->upload(Product::class, 'image', $tooLargeFile, ['model' => $product]);

if (!$result['status']) {
    // The old file was not deleted
    $oldFileStillExists = File::find($oldFileId) !== null;
    echo "Old file still exists: " . ($oldFileStillExists ? 'Yes' : 'No'); // Yes
}
```

## 27. Using `updateFieldConfig` to Temporarily Modify Field Constraints Before Upload (Integration with `FileUploadRegistry`)

**Scenario:** Temporarily increase the maximum image size to upload a large file, then restore it.

```php
$registry = FileUploadRegistry::instance();
$originalMaxSize = $registry->getFieldConfig(Product::class, 'image')['max_filesize'];

// Temporarily increase the limit
$registry->updateFieldConfig(Product::class, 'image', ['max_filesize' => 5120]); // 5 MB

$result = $service->upload(Product::class, 'image', $largeFile, ['model' => $product]);

// Restore the original limit
$registry->updateFieldConfig(Product::class, 'image', ['max_filesize' => $originalMaxSize]);
```

---

## Summary

The `FileUploadService` class provides a powerful and unified interface for managing file uploads in NanoSoft applications. Through the examples above, developers can implement any file upload scenario: from simple uploads with direct attachment to temporary uploads with unsaved models, permission and constraint checking, retrieving files with thumbnails, and secure deletion. Additionally, the service supports advanced features such as multiple storage, automatic image transformation, event hooks, WebSocket, and unique error codes. We recommend using this service as the single layer for handling files throughout the application to ensure consistency and security.

For details on the response structure when using the `FileUploadController`, refer to [API Documentation](./Docs-API-Documentation-en.md).

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
