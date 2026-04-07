


# General Documentation: File Upload Management in NanoSoft Applications (Nano.FileUpload)

**Overview of the `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, and others) is part of the **`Nano.FileUpload` package**, a comprehensive software package provided by **NanoSoft**, specifically designed for **NanoSoft applications** to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes of this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before saving the record, as well as unified error handling. With this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

---

## 1. Introduction

The **`Nano.FileUpload` module** is a specialized add-on provided by **NanoSoft** for NanoSoft and Laravel applications. This module aims to provide an **integrated file upload management system** via APIs in a secure and flexible manner, supporting different user types (backend / frontend) and fine-grained permissions at the level of each field and operation.

#### Why do we need a centralized file upload system?
In complex administrative applications, many plugins (such as products, users, orders) need to upload files (images, documents, etc.). Instead of rewriting upload logic and permission checks in each plugin, this system provides a **centralized layer** that any plugin can easily register with, ensuring:
- Unified upload process across the application.
- Management of permissions for different user types.
- Ability to upload files before saving the model (using temporary session keys).
- Unified error handling.
- Ready-to-use API endpoints.

---

## 2. Main Components of the System

The system consists of a set of specialized classes, which can be divided into several layers:

### a. Registry Layer
- **`FileUploadRegistry`**: The central registry that holds model definitions and upload fields. Provides functions to register models, query settings, and check permissions and user types.
- **Key Features**:
  - Register model with `allowed_user_types` (backend/frontend).
  - Set permissions for each operation (`add`, `edit`, `delete`, `view`) at model or field level.
  - Support for defaults to reduce repetition.
  - Fire events (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) to extend the system.
  - Support caching of query results.
  - Global settings to control API operations (`disable_upload`, `disable_delete`, `disable_get`).
  - Global settings to control automatic conversions (`disable_auto_resize`, `disable_auto_watermark`).

### b. Service Layer
- **`FileUploadService`**: Responsible for executing file upload, delete, and retrieval operations. It relies on `FileUploadRegistry` to access settings and verify permissions, and on `FileUploadUserManager` to manage the current user.
- **Main Functions**:
  - Upload single or multiple files (`upload`, `uploadMultiple`).
  - Delete a file with permission check (`deleteFile`).
  - Fetch files associated with a model or temporary (`getFiles`).
  - Generate temporary session keys (`generateTempSessionKey`) with HMAC-SHA256 signature.
  - Link temporary files to a saved model (`attachTempFiles`).
  - Validate file (size, type, blacklist) before upload (`validateFile`).
  - Apply automatic conversions (resizing, watermarking) to images (`applyAutoProcessing`).
  - Set custom storage disk (`getStorageDiskForField`).
  - Calculate SHA256 of content and store in `hash`.
  - Store image dimensions in `meta`.
  - Set expiration date for temporary files (`expires_at`).

### c. User & Permission Layer
- **`FileUploadUserManager`**: User and permission manager. Provides a unified interface to get the current user and their type (`backend`/`frontend`/`guest`) and to check their permissions.
- **Features**:
  - Default user resolver supporting `BackendAuth`, `Auth`, `AuthHelpers`.
  - Ability to customize user resolver (`setUserResolver`).
  - Ability to customize permission check function (`setPermissionChecker`).
  - Support for `hasAccess` and `hasPermission` for backend users, and `canUpload` for frontend users.

### d. API Layer
- **`FileUploadController`**: API controller providing RESTful endpoints for uploading files (single/multiple), deleting a file, and fetching files. Internally uses `FileUploadService`, `FileUploadRegistry`, and `FileUploadUserManager`.
- **Endpoints**:
  - `POST /upload` – upload single file.
  - `POST /upload-multiple` – upload multiple files.
  - `DELETE /delete/{id}` – delete a file.
  - `GET /files` – fetch associated files.
- **Authentication**: OAuth 2.0 via `oauth-users` middleware.
- **Responses**: Unified structure `{code, status, message, data, input_data, process_data, debug, error_code}`.

### e. Exception Handling Layer
- **`FileUploadException`**: Custom exception class providing unique error codes (e.g., `FILE_UPLOAD_FILE_SIZE_EXCEEDED`) and additional error context. Used throughout the service to unify error handling.

---

## 3. How These Classes Work Together (General Flow)

Suppose the `Nano.Shop` plugin wants to upload a main image for a new product. Here's how the components work together:

1. **Register the model (once)**:
   - In the plugin's `Plugin.php`, a `registerFileUploadFields` function is defined that returns an array of model definitions.
   - `FileUploadRegistry` automatically collects these definitions via `PluginManager` and registers them, applying defaults (based on file type) and caching.

2. **Upload the file (via API)**:
   - The client sends a `POST /upload` request with `model_class`, `field`, and the file (or base64).
   - `FileUploadController` checks that the model and field exist in `FileUploadRegistry`.
   - It calls `FileUploadService::upload` with the data.
   - `FileUploadService` uses:
     - `FileUploadRegistry::getFieldConfig` to get field settings.
     - `FileUploadUserManager::getUser` to get the current user.
     - `FileUploadRegistry::can` to check `add` permission (considering global settings).
     - `validateFile` to check blacklist, size, and allowed types.
     - `Base64::onUpload` (from `Nano.Api`) to perform the actual file upload.
     - `applyAutoProcessing` to apply automatic resizing and watermarking (if enabled).
     - Calculate `hash`, `meta`, and `expires_at` (for temporary files).
     - Set `disk` if a custom storage disk is specified.
   - The Controller returns a standardized response containing `data` (file ID, path) and `temp_session_key` if present, and error codes if needed.

3. **Link the file to the model (after saving the model)**:
   - After creating and saving the product, the application uses `FileUploadService::attachTempFiles` to move the temporary files to the model.
   - The temporary key is validated (signature, expiration, matching user and model).

4. **Fetch files**:
   - `GET /files` can be called with `model_class`, `field`, and `model_id` to fetch associated files, or with `temp_session_key` to fetch temporary files (with security checks).

---

## 4. Typical Use Cases

### a. Register a Product Model with a Main Image and an Image Gallery (using advanced settings)
```php
// In Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'auto_resize' => true,
                    'resize_options' => ['width' => 800, 'height' => 600, 'mode' => 'crop'],
                    'auto_watermark' => true,
                    'watermark_options' => ['position' => 'bottom-right', 'resize_percentage' => 15],
                    'storage_disk' => 's3',
                ],
                'gallery' => [
                    'type' => 'multiple',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'max_files' => 10,
                    'multiple' => true,
                ],
            ],
        ],
    ];
}
```

### b. Upload an Image for a New Product (using a temporary session key)
```php
// 1. Upload the image via API
$response = $api->post('/upload', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
    'file_base64' => 'data:image/jpeg;base64,...',
]);
$tempKey = $response['temp_session_key'];

// 2. Create and save the product
$product = new Product();
$product->name = 'New Product';
$product->save();

// 3. Link the image to the product (done server-side)
FileUploadService::instance()->attachTempFiles($product, 'image', $tempKey);
```

### c. Upload Multiple Images for an Existing Product Gallery
```php
$response = $api->post('/upload-multiple', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'files' => [$file1, $file2],
]);
```

### d. Fetch Product Gallery Images with Thumbnails
```php
$files = $api->get('/files', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
    ],
]);
```

### e. Listen to Events (e.g., Send WebSocket Notification)
```php
Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
    broadcast(new FileUploadedEvent($file));
});
```

---

## 5. Benefits and Advantages

- **Security**:
  - User permission checks (backend/frontend) before every operation.
  - Support for different user types, with customizable verification.
  - Use of unified `FileUploadUserManager` to get the current user.
  - File size and type validation before upload via `validateFile` (including blacklist of dangerous files).
  - Temporary session keys signed with HMAC-SHA256 containing `userType` and `timestamp` to prevent tampering.
  - Ability to completely disable API operations (`disable_upload`, `disable_delete`, `disable_get`).

- **Flexibility**:
  - Register any model with custom settings per field.
  - Specify `allowed_user_types` per model or field.
  - Support permissions for each operation (`add`, `edit`, `delete`, `view`) at model or field level.
  - Customizable user resolver and permission check function.
  - Support for file uploads in multiple formats (multipart, base64).
  - Support for temporary upload for unsaved models.
  - Support for multi-storage (S3, FTP, Local) via `storage_disk`.
  - Support for automatic image conversion (resizing, watermarking).
  - WebSocket support via events.

- **Performance**:
  - Use of temporary session keys to avoid storing unassociated files.
  - Ability to fetch files with custom thumbnails.
  - Model settings stored in memory after loading.
  - Caching of `getFieldConfig` and `getFieldConstraints` results to reduce database queries.

- **Scalability**:
  - Models from any plugin can be registered via `registerFileUploadFields` function or the `nano.api.fileupload.registerModels` event.
  - Logic can be extended via events (`modelRegistered`, `beforeUpload`, `afterUpload`, etc.).
  - Clear separation between registry, service, user management, and API layers.
  - New fields in the `system_files` table (`disk`, `hash`, `meta`, `expires_at`) enabling advanced features.

- **Developer Experience**:
  - Simple and unified interface.
  - Comprehensive documentation in English.
  - Support for single and multiple upload operations.
  - Automatic temporary file management.
  - Unique error codes and translated messages.

---

## 6. Complete Practical Example (Full Code with Advanced Features)

```php
<?php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadRegistry;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

// 1. Register the model (in Plugin.php)
public function registerFileUploadFields()
{
    return [
        Product::class => [
            'allowed_user_types' => ['backend'],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'auto_resize' => true,
                    'resize_options' => ['width' => 800, 'height' => 600],
                    'auto_watermark' => true,
                    'storage_disk' => 's3',
                ],
            ],
        ],
    ];
}

// 2. In an API controller
public function createProduct(Request $request)
{
    $base64Image = $request->input('image_base64');
    $productName = $request->input('name');

    // Upload the image temporarily
    $service = FileUploadService::instance();
    $tempKey = $service->generateTempSessionKey(Product::class, 'image');
    $uploadResult = $service->upload(Product::class, 'image', $base64Image, [
        'temp_session_key' => $tempKey,
    ]);

    if (!$uploadResult['status']) {
        return response()->json(['error' => $uploadResult['error'], 'error_code' => $uploadResult['error_code']], $uploadResult['code']);
    }

    // Create the product
    $product = new Product();
    $product->name = $productName;
    $product->save();

    // Link the image to the product
    $service->attachTempFiles($product, 'image', $tempKey);

    // Fire custom event (optional)
    Event::fire('product.created', [$product]);

    return response()->json(['product_id' => $product->id]);
}
```

---

## 7. Dependencies and Requirements

- **PHP** (>= 8.0).
- **Laravel / OctoberCMS** (any framework compatible with `System\Models\File`).
- **`Nano.Api` add-on** (provides `Base64` and `ApiController`).
- **Composer** to install the package.
- (Optional) **`Nano2.Watermark` add-on** for automatic watermarking.
- (Optional) **WebSocket library** for real-time notifications.

---

## 8. Conclusion

The `Nano.FileUpload` classes represent a complete and professional solution for managing file uploads in NanoSoft applications. Thanks to the layered design (Registry, Service, UserManager, API), developers can quickly and easily build powerful and secure file upload systems. With the addition of `FileUploadService`, the system becomes even more organized and easy to use, providing a single entry point for performing upload, delete, and retrieval operations, while supporting advanced options such as temporary uploads, thumbnails, and permission checks for different user types. This design ensures scalability and maintainability, meeting the needs of advanced applications without compromising system security or performance.

---

## 9. Additional Documentation

For more details about each class, please refer to the following documents:

- [FileUploadRegistry Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for FileUploadRegistry Class](./Docs-FileUploadRegistry-Class-Advanced-Examples-en.md)
- [FileUploadService Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for FileUploadService Class](./Docs-FileUploadService-Class-Advanced-Examples-en.md)
- [FileUploadUserManager Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for FileUploadUserManager Class](./Docs-FileUploadUserManager-Class-Advanced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
