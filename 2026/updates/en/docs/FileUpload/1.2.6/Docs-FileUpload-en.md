# General Documentation: File Upload Management in NanoSoft Applications (Nano.FileUpload)

**Overview of `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**, specifically designed for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes in this module reside under the following Namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, as well as unified error handling. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

---

## 1. Introduction

The **`Nano.FileUpload` module** is a specialized add-on provided by **NanoSoft** for NanoSoft and Laravel applications. This module aims to provide an **integrated file upload management system** via APIs in a secure and flexible manner, supporting different user types (backend/frontend) and granular permissions at the field and operation level.

#### Why do we need a centralized file upload system?
In complex administrative applications, many add-ons (such as products, users, orders) need to upload files (images, documents, etc.). Instead of rewriting upload logic and permission checks in each add-on, this system provides a **centralized layer** that any add-on can easily register with, ensuring:
- Standardized upload process across the application.
- Permission management for different user types.
- Ability to upload files before saving the model (using temporary session keys).
- Support for file replacement (`edit` operation) in singular relationships while maintaining integrity using transactions.
- Unified error handling.
- Ready-to-use API endpoints.

---

## 2. Main System Components

The system consists of several specialized classes, which can be divided into multiple layers:

### a. Registry Layer
- **`FileUploadRegistry`**: The central registry containing model and upload field definitions. Provides methods to register models, query settings, and check permissions and user types.
- **Key Properties**:
  - Register model with `allowed_user_types` (backend/frontend).
  - Assign permissions for each operation (`add`, `edit`, `delete`, `view`) at model or field level.
  - **`disabled_operations` addition** (since v1.2.1): Disable specific operations at model or field level without modifying permissions.
  - **Integrated validation methods** (since v1.2.1): `canGlobal`, `validateGlobal`, `canModel`, `validateModel`, `canField`, `validateField`, `validate` – allow checking permissions at multiple levels and throwing appropriate exceptions.
  - Support default settings (`defaults`) to reduce repetition.
  - `updateFieldConfig` method to update specific field settings with normalization and cache clearing.
  - Enhanced `updateModelConfig` method to normalize entire model after modification.
  - Fire events (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) for extensibility.
  - Cache support for query results.
  - Global settings to control API operations (`disable_upload`, `disable_delete`, `disable_get`, `disable_edit`).
  - Global settings to control automatic transformations (`disable_auto_resize`, `disable_auto_watermark`).

### b. Service Layer
- **`FileUploadService`**: Responsible for executing file upload, delete, and retrieval operations. Depends on `FileUploadRegistry` for settings and permission checks, and on `FileUploadUserManager` for current user management.
- **Main Functions**:
  - Upload single or multiple files (`upload`, `uploadMultiple`).
  - Automatically determine operation (`add` or `edit`) based on existence of associated file in `attachOne` relation.
  - Use `Db::transaction` to ensure integrity of replacement operation (upload new, delete old, attach relation).
  - Delete a file with permission check (`deleteFile`).
  - Retrieve files associated with a model or temporary (`getFiles`).
  - Generate temporary session keys (`generateTempSessionKey`) with HMAC-SHA256 signature.
  - Attach temporary files to a saved model (`attachTempFiles`) with unified response.
  - **Unified temp key validation** (since v1.2.2): Use `HasFileUploadsMatchTempKey` trait and `validateAndMatchTempKey` method to verify and match the key against model, field, and user in all methods (`upload`, `deleteFile`, `attachTempFiles`, `getFiles`).
  - Validate file (size, type, blacklist) before upload (`validateFile`).
  - Apply automatic transformations (resize, watermark) to images (`applyAutoProcessing`).
  - Set custom storage disk for field (`getStorageDiskForField`).
  - Calculate SHA256 of content and store in `hash`.
  - Store image dimensions in `meta`.
  - Set expiration date for temporary files (`expires_at`) and set `session_key` for temporary files.

### c. User & Permission Layer
- **`FileUploadUserManager`**: User and permission manager. Provides a unified interface to get current user, user type (`backend`/`frontend`/`guest`), and check permissions.
- **Features**:
  - Default user resolver supporting `BackendAuth`, `Auth`, `AuthHelpers`.
  - Customizable user resolver (`setUserResolver`).
  - Customizable permission checker (`setPermissionChecker`).
  - Support `hasAccess` and `hasPermission` for backend users, and `canUpload` for frontend users.

### d. API Layer
- **`FileUploadController`**: API controller providing RESTful endpoints for file upload (single/multiple), delete, retrieval, as well as new endpoints for querying modules, permissions, and validating temp keys.
- **Core Endpoints**:
  - `POST /upload` – Upload single file.
  - `POST /upload-multiple` – Upload multiple files.
  - `DELETE /delete/{id}` – Delete a file.
  - `GET /files` – Retrieve associated files.
  - `GET /tests` – Run test suite (available only in development environment).
- **New Endpoints** (since v1.2.3):
  - **Query modules and fields**:
    - `GET /models` – Get list of registered modules (filtered by user permission).
    - `GET /models/{modelClass}` – Get settings for a specific module.
    - `GET /models/{modelClass}/fields/{field}` – Get settings for a specific field.
    - `GET /models/{modelClass}/fields/{field}/constraints` – Get field constraints (size, allowed types, etc.).
    - `GET /processing-options/{modelClass}/{field}` – Get advanced processing options (storage, resize, watermark).
  - **Permission checks**:
    - `GET /permissions/global/{operation}` – Check if an operation is globally enabled.
    - `GET /permissions/model/{modelClass}/{operation}` – Check operation permission at module level.
    - `GET /permissions/field/{modelClass}/{field}/{operation}` – Check operation permission at field level.
    - `POST /permissions/check` – Integrated permission check using `validate()`.
  - **Temp key validation**:
    - `POST /temp-key/validate` – Validate a temporary key and match it using `validateAndMatchTempKey`.
- **Authentication**: OAuth 2.0 via `oauth-users` middleware.
- **Responses**: Standardized according to structure `{code, status, message, data, input_data, process_data, debug, error_code}`.

### e. Exception Handling Layer
- **`FileUploadException`**: Custom exception class providing unique error codes (e.g., `FILE_UPLOAD_FILE_SIZE_EXCEEDED`) and additional error context. Used throughout the service to unify error handling.
- Automatically maps internal error codes to appropriate HTTP status codes (401, 403, 404, 422, 500, 503, 423, etc.) based on a predefined map. **`ERR_PERMISSION_DENIED` code changed from 403 to 422** (since v1.2.1) to avoid conflict with "account inactive" status.

### f. Testing Layer
- **`FileUploadPlusTest`**: Comprehensive test class providing full coverage for all add-on classes.
- **Features**:
  - Standardize test output using same API structure (`code`, `status`, `test_code`, `name`, `description`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`).
  - Advanced exception handling preventing other tests from stopping when one test fails.
  - Use `Db::beginTransaction()` and `Db::rollBack()` in all tests that modify the database to ensure no leftover data.
  - Custom tests for edit operation (`testEditReplaceFile`, `testEditWithoutPermission`, `testEditWithGlobalDisable`, `testEditTriggersEvents`, `testEditKeepsOldFileOnFailure`).
  - Support running tests via browser using `test_version=v1` or `test_version=v2`.

---

## 3. How These Classes Work Together (General Flow)

Suppose the `Nano.Shop` add-on wants to upload a main image for a new product, then replace it later. Here's how the components work together:

1. **Register the model (once)**:
   - In the add-on's `Plugin.php`, define a `registerFileUploadFields` function that returns an array of model definitions.
   - `FileUploadRegistry` automatically collects these definitions via `PluginManager` and registers them, applying default settings (based on file type) and caching.
   - **Since v1.2.1** you can add `disabled_operations` at model or field level to disable specific operations.

2. **Upload the first file (add operation)**:
   - Client sends `POST /upload` request with `model_class`, `field`, and the file.
   - `FileUploadController` checks that the model and field exist in `FileUploadRegistry`.
   - Calls `FileUploadService::upload` with the data.
   - `FileUploadService` determines the operation as `add` (no associated file).
   - **If `temp_session_key` is passed, it is validated and matched using `validateAndMatchTempKey`** (since v1.2.2).
   - Checks permission via `validate($modelClass, 'add', ...)`.
   - Uploads the file via `Base64::onUpload`, calculates `hash`, `meta`, sets `expires_at` and `session_key` for temporary files (if no saved model).
   - Returns `temp_session_key` in the response.

3. **Attach the file to the model after product creation**:
   - After saving the product, the application calls `FileUploadService::attachTempFiles` to attach the temporary file to the model.
   - **`attachTempFiles` uses `validateAndMatchTempKey`** to verify the key, user, model, and field in one step.

4. **Replace the file (edit operation)**:
   - Client sends `POST /upload` again with same `model_class` and `field` but with `model_id` of the existing product.
   - `FileUploadService` detects an existing file in the `attachOne` relation, so it determines the operation as `edit`.
   - Checks `edit` permission via `validate($modelClass, 'edit', ...)`.
   - Executes a transaction (`Db::transaction`):
     - Upload new file and save it.
     - Fire `nano.fileupload.beforeEditDelete` event.
     - Delete the old file.
     - Fire `nano.fileupload.afterEditDelete` event.
     - Attach the new file to the model relation.
   - If any part fails, all changes are rolled back (old file is not deleted).

5. **Retrieve files**:
   - Can call `GET /files` with `model_class`, `field`, `model_id` to get associated files, or with `temp_session_key` to get temporary files.
   - **When using `temp_session_key`, it is validated and matched using `validateAndMatchTempKey`** before executing the query.

6. **Using new endpoints (since v1.2.3)**:
   - Front-ends can use `/models` to discover available modules and fields.
   - Can use `/permissions/check` to verify user permission before attempting upload.
   - Can use `/temp-key/validate` to ensure a temporary key is valid before using it.

---

## 4. Typical Use Cases

### a. Register a Product model with main image and gallery (using advanced settings and `disabled_operations`)
```php
// In Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'disabled_operations' => ['delete'], // disable deletion at product level
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
                    'disabled_operations' => ['edit'], // disable image replacement only
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

### b. Upload an image for a new product (using temporary session key)
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

// 3. Attach the image to the product (done server-side)
FileUploadService::instance()->attachTempFiles($product, 'image', $tempKey);
```

### c. Replace an existing product image (edit operation)
```php
// Upload new image with model_id
$response = $api->post('/upload', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
    'model_id' => 10,
    'file_base64' => 'data:image/jpeg;base64,...',
]);
// The new image will automatically replace the old one, and the old one will be deleted
```

### d. Upload multiple images to an existing product gallery
```php
$response = $api->post('/upload-multiple', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'files' => [$file1, $file2],
]);
```

### e. Retrieve product gallery images with thumbnails
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

### f. Listen to events (e.g., send WebSocket notification when file is replaced)
```php
Event::listen('nano.fileupload.afterEditDelete', function ($oldFile, $modelClass, $field, $model) {
    // Send notification that the old file was deleted and replaced by a new one
    broadcast(new FileReplacedEvent($oldFile, $model));
});
```

### g. Using new endpoints to query modules (since v1.2.3)
```php
// Get all modules available to the current user
$models = $api->get('/models');

// Get settings for a specific field
$fieldConfig = $api->get('/models/Nano%5CShop%5CModels%5CProduct/fields/image');

// Check user permission to replace a product image
$permission = $api->post('/permissions/check', [
    'model_class' => 'Nano\Shop\Models\Product',
    'operation' => 'edit',
    'field' => 'image',
]);
if ($permission['data']['allowed']) {
    // Show image replacement UI
}

// Validate a temporary key before using it
$tempKeyValidation = $api->post('/temp-key/validate', [
    'temp_key' => $tempKey,
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
]);
if ($tempKeyValidation['data']['valid']) {
    // Key is valid, can be used
}
```

---

## 5. Benefits and Advantages

- **Security**:
  - User permission checks (backend/frontend) before every operation.
  - Support different user types, with customizable verification.
  - Use unified `FileUploadUserManager` to get current user.
  - Validate file size and type before upload via `validateFile` (including blacklist for dangerous files).
  - Temporary session keys signed with HMAC-SHA256 containing `userType` and `timestamp` to prevent tampering.
  - **Unified temp key validation** (since v1.2.2) via `validateAndMatchTempKey` used in all methods.
  - Ability to globally disable API operations (`disable_upload`, `disable_delete`, `disable_get`, `disable_edit`).
  - Ability to disable specific operations at model or field level via `disabled_operations` (since v1.2.1).

- **Flexibility**:
  - Register any model with custom settings per field.
  - Specify `allowed_user_types` per model or field.
  - Support permissions per operation (`add`, `edit`, `delete`, `view`) at model or field level.
  - Customizable user resolver and permission checker.
  - Support multiple file formats (multipart, base64).
  - Support temporary upload for unsaved models.
  - Support multiple storage disks (S3, FTP, Local) via `storage_disk`.
  - Support automatic image transformations (resize, watermark).
  - Support WebSocket via events.
  - `updateFieldConfig` method to update specific field settings without re-registering the model.
  - **New API endpoints for querying modules, permissions, and validating temp keys** (since v1.2.3).

- **Performance**:
  - Use temporary session keys to avoid saving unassociated files.
  - Ability to retrieve files with custom thumbnails.
  - Store model settings in memory after loading.
  - Cache results of `getFieldConfig` and `getFieldConstraints` to reduce database queries.
  - Use transactions (`Db::transaction`) in replacement operations to ensure integrity and avoid data loss.

- **Scalability**:
  - Models can be registered from any add-on via `registerFileUploadFields` or `nano.api.fileupload.registerModels` event.
  - Extend logic via events (`modelRegistered`, `beforeUpload`, `afterUpload`, `beforeEditDelete`, `afterEditDelete`, etc.).
  - Clear separation between registry, service, user management, and API layers.
  - Support new columns in `system_files` table (`disk`, `hash`, `meta`, `expires_at`, `session_key`) enabling advanced features.

- **Developer Experience**:
  - Simple and unified API interface.
  - Comprehensive documentation in Arabic.
  - Support single and multiple uploads.
  - Automatic temporary file management.
  - Unique error codes and translated messages.
  - Integrated test suite (`FileUploadPlusTest`) easily run via API to verify installation.
  - **Query API endpoints** facilitate building dynamic front-ends.

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

    // Upload image temporarily
    $service = FileUploadService::instance();
    $tempKey = $service->generateTempSessionKey(Product::class, 'image');
    $uploadResult = $service->upload(Product::class, 'image', $base64Image, [
        'temp_session_key' => $tempKey,
    ]);

    if (!$uploadResult['status']) {
        return response()->json(['error' => $uploadResult['error'], 'error_code' => $uploadResult['error_code']], $uploadResult['code']);
    }

    // Create product
    $product = new Product();
    $product->name = $productName;
    $product->save();

    // Attach image to product
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
        // 'skip_permission_check' => true // only for tests
    ]);

    if (!$result['status']) {
        return response()->json(['error' => $result['error'], 'error_code' => $result['error_code']], $result['code']);
    }

    return response()->json(['message' => 'Image replaced successfully']);
}

// 4. Example of using new endpoints to check permission before displaying form
public function showProductEditForm($id)
{
    $product = Product::findOrFail($id);
    // Check user permission to replace image
    $client = new \GuzzleHttp\Client();
    $response = $client->post(env('API_URL') . '/permissions/check', [
        'headers' => ['Authorization' => 'Bearer ' . $request->bearerToken()],
        'json' => [
            'model_class' => Product::class,
            'operation' => 'edit',
            'field' => 'image',
        ],
    ]);
    $permission = json_decode($response->getBody(), true);
    if (!$permission['data']['allowed']) {
        return view('product.edit', ['product' => $product, 'can_edit_image' => false]);
    }
    return view('product.edit', ['product' => $product, 'can_edit_image' => true]);
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

**Note:** The `Base64` class was moved from `Nano.API` into the add-on (`Nano\FileUpload\Classes\Base64`) starting from version 1.1.0, so there is no mandatory dependency on `Nano.API` except for `ApiController`.

---

## 8. Conclusion

The `Nano.FileUpload` classes represent a complete and professional solution for managing file uploads in NanoSoft applications. Thanks to the layered design (Registry, Service, UserManager, API), developers can quickly and easily build powerful and secure file upload systems. With the addition of `FileUploadService`, the system is more organized and easier to use, providing a single entry point for executing upload, delete, and retrieval operations, with support for advanced options such as temporary uploads, thumbnails, and permission checks for different user types. Support for the `edit` operation (file replacement) using transactions and custom events ensures data integrity and security. Recent versions (1.2.1, 1.2.2, 1.2.3) added significant improvements: integrated validation methods (`validate`, `canGlobal`, ...), ability to disable specific operations (`disabled_operations`), unified temp key validation logic via `HasFileUploadsMatchTempKey` trait, and new API endpoints for querying modules, permissions, and validating keys. This design ensures scalability and maintainability, meeting the needs of advanced applications without compromising system security or performance.

---

## 9. Additional Documentation

For more details on each class, please refer to the following documents:

- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advanced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advanced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advanced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

---