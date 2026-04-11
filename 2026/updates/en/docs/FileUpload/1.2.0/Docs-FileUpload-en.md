# General Documentation: File Upload Management in NanoSoft Applications (Nano.FileUpload)

**Overview of the `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**, specifically designed for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes in this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, as well as unified error handling. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

---

## 1. Introduction

The **`Nano.FileUpload` module** is a specialized add-on provided by **NanoSoft** for NanoSoft and Laravel applications. This module aims to provide an **integrated system for managing file uploads** via APIs in a secure and flexible manner, with support for different user types (backend/frontend) and granular permissions at the field and operation level.

#### Why do we need a centralized file upload system?
In complex administrative applications, many add-ons (e.g., products, users, orders) need to upload files (images, documents, etc.). Instead of rewriting upload logic and permission checks in each add-on, this system provides a **centralized layer** that any add-on can easily register with, ensuring:
- Unified upload process across the application.
- Permission management for different user types.
- Ability to upload files before saving the model (using temporary session keys).
- Support for file replacement (`edit` operation) in one-to-one relations while maintaining integrity using transactions.
- Unified error handling.
- Ready-to-use API interface.

---

## 2. Main System Components

The system consists of a set of specialized classes, which can be divided into several layers:

### a. Registry Layer
- **`FileUploadRegistry`**: The central registry that contains model definitions and upload fields. Provides methods for registering models, querying settings, and checking permissions and user types.
- **Key Features**:
  - Register model with `allowed_user_types` (backend/frontend).
  - Set permissions for each operation (`add`, `edit`, `delete`, `view`) at model or field level.
  - Support default settings to reduce duplication.
  - `updateFieldConfig` function to update specific field settings with normalization and cache clearing.
  - `updateModelConfig` function enhanced to normalize the entire model after modification.
  - Dispatch events (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) to extend the system.
  - Cache support for query results.
  - Global settings to control API operations (`disable_upload`, `disable_delete`, `disable_get`, `disable_edit`).
  - Global settings to control automatic transformations (`disable_auto_resize`, `disable_auto_watermark`).

### b. Service Layer
- **`FileUploadService`**: Responsible for executing file upload, delete, and retrieval operations. Depends on `FileUploadRegistry` for settings and permission checks, and on `FileUploadUserManager` for managing the current user.
- **Main Functions**:
  - Upload single or multiple files (`upload`, `uploadMultiple`).
  - Automatically determine operation (`add` or `edit`) based on existence of an associated file in an `attachOne` relation.
  - Use `Db::transaction` to ensure integrity of replacement operation (upload new, delete old, attach relation).
  - Delete a file with permission check (`deleteFile`).
  - Retrieve files associated with a model or temporary (`getFiles`).
  - Generate temporary session keys (`generateTempSessionKey`) with HMAC-SHA256 signature.
  - Attach temporary files to a saved model (`attachTempFiles`) with unified response.
  - Validate file (size, type, blacklist) before upload (`validateFile`).
  - Apply automatic transformations (resize, watermark) to images (`applyAutoProcessing`).
  - Set custom storage disk (`getStorageDiskForField`).
  - Calculate SHA256 of content and store in `hash`.
  - Store image dimensions in `meta`.
  - Set expiration date for temporary files (`expires_at`) and set `session_key` for temporary files.

### c. User & Permission Layer
- **`FileUploadUserManager`**: User and permission manager. Provides a unified interface to get the current user, their type (`backend`/`frontend`/`guest`), and check permissions.
- **Features**:
  - Default resolver supporting `BackendAuth`, `Auth`, `AuthHelpers`.
  - Ability to customize user resolver (`setUserResolver`).
  - Ability to customize permission checker function (`setPermissionChecker`).
  - Support for `hasAccess` and `hasPermission` functions for backend users, and `canUpload` for frontend users.

### d. API Layer
- **`FileUploadController`**: API controller providing RESTful endpoints for uploading (single/multiple), deleting a file, and retrieving files. Internally uses `FileUploadService`, `FileUploadRegistry`, and `FileUploadUserManager`.
- **Endpoints**:
  - `POST /upload` – Upload a single file.
  - `POST /upload-multiple` – Upload multiple files.
  - `DELETE /delete/{id}` – Delete a file.
  - `GET /files` – Retrieve associated files.
  - `GET /tests` – Run the test suite (available only in development environment).
- **Authentication**: OAuth 2.0 via `oauth-users` middleware.
- **Responses**: Unified according to structure `{code, status, message, data, input_data, process_data, debug, error_code}`.

### e. Exception Handling Layer
- **`FileUploadException`**: Custom exception class providing unique error codes (e.g., `FILE_UPLOAD_FILE_SIZE_EXCEEDED`) and additional error context. Used throughout the service to unify error handling.
- Automatically maps internal error codes to appropriate HTTP codes (401, 403, 404, 422, 500, 503, etc.) based on a predefined map.

### f. Testing Layer
- **`FileUploadPlusTest`**: Comprehensive test class providing full coverage of all add-on classes.
- **Features**:
  - Unify test outputs with the same API structure (`code`, `status`, `test_code`, `name`, `description`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`).
  - Advanced exception handling that prevents one failing test from stopping others.
  - Use `Db::beginTransaction()` and `Db::rollBack()` in all tests that modify the database to ensure no leftover data.
  - Specialized tests for edit operation (`testEditReplaceFile`, `testEditWithoutPermission`, `testEditWithGlobalDisable`, `testEditTriggersEvents`, `testEditKeepsOldFileOnFailure`).
  - Support running tests via browser using parameter `test_version=v1` or `test_version=v2`.

---

## 3. How These Classes Work Together (General Flow)

Suppose the `Nano.Shop` add-on wants to upload a main image for a new product, and later replace it. Here's how the components work together:

1. **Register the model (once)**:
   - In the add-on's `Plugin.php`, define the `registerFileUploadFields` function that returns an array of model definitions.
   - `FileUploadRegistry` automatically collects these definitions via `PluginManager` and registers them, applying defaults (per file type) and caching.

2. **Upload the first file (add operation)**:
   - Client sends a `POST /upload` request with `model_class`, `field`, and the file.
   - `FileUploadController` checks if the model and field exist in `FileUploadRegistry`.
   - Calls `FileUploadService::upload` with the data.
   - `FileUploadService` determines the operation as `add` (no existing file attached).
   - Checks permission via `can($modelClass, 'add', ...)`.
   - Uploads the file via `Base64::onUpload`, calculates `hash`, `meta`, sets `expires_at` and `session_key` for temporary files (if no saved model).
   - Returns `temp_session_key` in the response.

3. **Attach the file to the model after product creation**:
   - After saving the product, the application calls `FileUploadService::attachTempFiles` to attach the temporary file to the model.

4. **Replace the file (edit operation)**:
   - Client sends another `POST /upload` request with the same `model_class` and `field` but with `model_id` of the existing product.
   - `FileUploadService` checks if a file is already attached via `attachOne`, and determines the operation as `edit`.
   - Checks `edit` permission via `can($modelClass, 'edit', ...)`.
   - Executes a transaction (`Db::transaction`):
     - Upload and save the new file.
     - Dispatch event `nano.fileupload.beforeEditDelete`.
     - Delete the old file.
     - Dispatch event `nano.fileupload.afterEditDelete`.
     - Attach the new file to the model relation.
   - If any part fails, all changes are rolled back (old file is not deleted).

5. **Retrieve files**:
   - Call `GET /files` with `model_class`, `field`, and `model_id` to get attached files, or with `temp_session_key` to get temporary files (with security check).

---

## 4. Typical Use Cases

### a. Register a Product Model with Main Image and Gallery (using advanced settings)
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

### b. Upload an Image for a New Product (using temporary session key)
```php
// 1. Upload image via API
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

// 3. Attach image to product (done server-side)
FileUploadService::instance()->attachTempFiles($product, 'image', $tempKey);
```

### c. Replace an Image for an Existing Product (edit operation)
```php
// Upload the new image while passing model_id
$response = $api->post('/upload', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
    'model_id' => 10,
    'file_base64' => 'data:image/jpeg;base64,...',
]);
// The new image will automatically replace the old one, and the old one will be deleted
```

### d. Upload Multiple Images for an Existing Product Gallery
```php
$response = $api->post('/upload-multiple', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'files' => [$file1, $file2],
]);
```

### e. Retrieve Gallery Images for a Product with Thumbnails
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

### f. Listen to Events (e.g., Send WebSocket Notification when a File is Replaced)
```php
Event::listen('nano.fileupload.afterEditDelete', function ($oldFile, $modelClass, $field, $model) {
    // Send notification that the old file has been deleted and replaced with a new one
    broadcast(new FileReplacedEvent($oldFile, $model));
});
```

---

## 5. Benefits and Advantages

- **Security**:
  - Check user permissions (backend/frontend) before each operation.
  - Support different user types with customizable checks.
  - Unified `FileUploadUserManager` to get current user.
  - Validate file size and type before upload via `validateFile` (including dangerous file blacklist).
  - Temporary session keys signed with HMAC-SHA256 containing `userType` and `timestamp` to prevent tampering.
  - Ability to globally disable API operations (`disable_upload`, `disable_delete`, `disable_get`, `disable_edit`).

- **Flexibility**:
  - Register any model with custom settings per field.
  - Specify `allowed_user_types` per model or field.
  - Support permissions per operation (`add`, `edit`, `delete`, `view`) at model or field level.
  - Ability to customize user resolver and permission checker function.
  - Support uploading files in multiple formats (multipart, base64).
  - Support temporary upload for unsaved models.
  - Support multiple storage disks (S3, FTP, Local) via `storage_disk`.
  - Support automatic image transformations (resize, watermark).
  - Support WebSocket via events.
  - `updateFieldConfig` function to update settings of a specific field without re-registering the model.

- **Performance**:
  - Use temporary session keys to avoid saving files unassociated.
  - Ability to retrieve files with custom thumbnails.
  - Cache model settings after loading.
  - Cache `getFieldConfig` and `getFieldConstraints` results to reduce database queries.
  - Use transactions (`Db::transaction`) in replacement operations to ensure integrity and avoid data loss.

- **Scalability**:
  - Register models from any add-on via `registerFileUploadFields` function or `nano.api.fileupload.registerModels` event.
  - Extend logic via events (`modelRegistered`, `beforeUpload`, `afterUpload`, `beforeEditDelete`, `afterEditDelete`, etc.).
  - Clear separation between registry, service, user, and API layers.
  - Support new columns in `system_files` table (`disk`, `hash`, `meta`, `expires_at`, `session_key`) enabling advanced features.

- **Developer Experience**:
  - Simple and unified API interface.
  - Comprehensive documentation in Arabic.
  - Support single and multiple upload operations.
  - Automatic management of temporary files.
  - Unique error codes and translated messages.
  - Integrated test system (`FileUploadPlusTest`) easily run via API to verify installation.

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
            'permissions' => [
                'add'    => 'products.create',
                'edit'   => 'products.update',
                'delete' => 'products.delete',
                'view'   => 'products.view',
            ],
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

// 2. In API controller to create a product
public function createProduct(Request $request)
{
    $base64Image = $request->input('image_base64');
    $productName = $request->input('name');

    // Temporarily upload the image
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

    // Attach the image to the product
    $attachResult = $service->attachTempFiles($product, 'image', $tempKey);
    if (!$attachResult['status']) {
        // Handle error
    }

    return response()->json(['product_id' => $product->id]);
}

// 3. In API controller to update product image (replace)
public function updateProductImage(Request $request, $id)
{
    $product = Product::findOrFail($id);
    $base64Image = $request->input('image_base64');

    $service = FileUploadService::instance();
    $result = $service->upload(Product::class, 'image', $base64Image, [
        'model' => $product,
        // 'skip_permission_check' => true // for tests only
    ]);

    if (!$result['status']) {
        return response()->json(['error' => $result['error'], 'error_code' => $result['error_code']], $result['code']);
    }

    return response()->json(['message' => 'Image replaced successfully']);
}
```

---

## 7. Dependencies and Requirements

- **PHP** (>= 8.0).
- **Laravel / OctoberCMS** (any framework compatible with `System\Models\File`).
- **`Nano.Api` add-on** (to provide `ApiController`).
- **Composer** to install the package.
- (Optional) **`Nano2.Watermark` add-on** for automatic watermark.
- (Optional) **WebSocket library** for real-time notifications.

**Note**: The `Base64` class was moved from `Nano.API` into this add-on (`Nano\FileUpload\Classes\Base64`) starting from version 1.1.0, so there is no longer a mandatory dependency on `Nano.API` except for `ApiController`.

---

## 8. Conclusion

The `Nano.FileUpload` classes represent a complete and professional solution for managing file uploads in NanoSoft applications. Thanks to the layered design (Registry, Service, UserManager, API), developers can quickly and easily build robust and secure file upload systems. With the addition of `FileUploadService`, the system is more organized and easier to use, providing a single entry point for upload, delete, and retrieval operations, while supporting advanced options like temporary uploads, thumbnails, and permission checks for different user types. Additionally, support for the `edit` operation (file replacement) using transactions and dedicated events ensures data integrity and security. This design guarantees scalability and maintainability, meeting the needs of advanced applications without compromising system security or performance.

---

## 9. Additional Documentation

For more details on each class, please refer to the following documents:

- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
