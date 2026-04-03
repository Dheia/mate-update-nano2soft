# General Documentation: File Upload Management in NanoSoft Applications (Nano.FileUpload)

**Overview of the `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload`** package, an integrated software package provided by **NanoSoft**, designed specifically for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible way.

All classes of this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, and handling errors uniformly. With this design, developers can easily build scalable file upload systems that meet advanced application requirements without compromising security or performance.

---

## 1. Introduction

The **`Nano.FileUpload`** module is a specialized plugin provided by **NanoSoft** for NanoSoft and Laravel applications. It aims to provide an **integrated system for managing file uploads** via APIs in a secure and flexible manner, with support for different user types (backend / frontend) and granular permissions at the field and operation level.

#### Why Do We Need a Centralized File Upload System?
In complex administrative applications, many plugins (such as products, users, orders) need to upload files (images, documents, etc.). Instead of rewriting upload and permission logic in each plugin, this system provides a **central layer** that any plugin can register with easily, ensuring:
- Uniform upload process across the application.
- Permission management for different user types.
- Ability to upload files before model saving (using temporary session keys).
- Unified error handling.
- Ready-to-use API.

---

## 2. Main Components of the System

The system consists of a set of specialized classes, which can be divided into several layers:

### a. Registry Layer
- **`FileUploadRegistry`**: Central registry containing model definitions and upload fields. Provides methods to register models, query settings, and check permissions and user types.
- **Key Features**:
  - Register a model with `allowed_user_types` (backend/frontend).
  - Set permissions per operation (`add`, `edit`, `delete`, `view`) at the model or field level.
  - Support default settings (`defaults`) to reduce repetition.
  - Fire events (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) to extend the system.

### b. Service Layer
- **`FileUploadService`**: Responsible for executing file upload, delete, and retrieval operations. Relies on `FileUploadRegistry` to access settings and check permissions, and on `FileUploadUserManager` to manage the current user.
- **Key Functions**:
  - Upload single or multiple files (`upload`, `uploadMultiple`).
  - Delete a file with permission check (`deleteFile`).
  - Retrieve files associated with a model or temporary key (`getFiles`).
  - Generate temporary session keys (`generateTempSessionKey`).
  - Link temporary files to a saved model (`attachTempFiles`).
  - Validate a file (size, type) before upload (`validateFile`).

### c. User & Permission Layer
- **`FileUploadUserManager`**: User and permission manager. Provides a unified interface to get the current user, their type (`backend`/`frontend`/`guest`), and check permissions.
- **Features**:
  - Default user resolver supporting `BackendAuth`, `Auth`, `AuthHelpers`.
  - Ability to customize the user resolver (`setUserResolver`).
  - Ability to customize the permission checker (`setPermissionChecker`).
  - Support for `hasAccess` and `hasPermission` methods for backend users, and a `canUpload` method for frontend users.

### d. API Layer
- **`FileUploadController`**: API controller providing RESTful endpoints for file upload (single/multiple), deletion, and retrieval. Internally uses `FileUploadService`, `FileUploadRegistry`, and `FileUploadUserManager`.
- **Endpoints**:
  - `POST /upload` – Upload a single file.
  - `POST /upload-multiple` – Upload multiple files.
  - `DELETE /delete/{id}` – Delete a file.
  - `GET /files` – Retrieve associated files.
- **Authentication**: OAuth 2.0 via `oauth-users` middleware.
- **Responses**: Unified structure `{code, status, message, data, ...}`.

---

## 3. How These Classes Work Together (General Flow)

Suppose the `Nano.Shop` plugin wants to upload a main image for a new product. Here is how the components work together:

1. **Model Registration (once)**:
   - In the plugin’s `Plugin.php`, a `registerFileUploadFields` method is defined that returns an array of model definitions.
   - `FileUploadRegistry` automatically collects these definitions via `PluginManager` and registers them.

2. **File Upload (via API)**:
   - The client sends a `POST /upload` request with `model_class`, `field`, and the file.
   - `FileUploadController` verifies that the model and field exist in `FileUploadRegistry`.
   - It calls `FileUploadService::upload` with the data.
   - `FileUploadService` uses:
     - `FileUploadRegistry::getFieldConfig` to get field settings.
     - `FileUploadUserManager::getUser` to get the current user.
     - `FileUploadRegistry::can` to check `add` permission.
     - `Base64::onUpload` (from `Nano.Api`) to perform the actual upload.
   - If `temp_session_key` is passed or no saved model is provided, a temporary session key is used.
   - The controller returns a unified response containing `data` (file ID, path) and `temp_session_key` if any.

3. **Linking Files to the Model (after model save)**:
   - After creating and saving the product, the application uses `FileUploadService::attachTempFiles` to move temporary files to the model.

4. **Retrieving Files**:
   - `GET /files` can be called with `model_class`, `field`, and `model_id` to retrieve associated files.

---

## 4. Typical Use Cases

### a. Register a Product Model with a Main Image and Gallery

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

### b. Upload an Image for a New Product (Using Temporary Session Key)

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
$product->name = 'New product';
$product->save();

// 3. Link the image to the product (done on the server)
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

### d. Retrieve a Product Gallery with Thumbnails

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

---

## 5. Benefits and Advantages

- **Security**:
  - Permission checks (backend/frontend) before every operation.
  - Support for different user types with customizable checks.
  - Unified user retrieval via `FileUploadUserManager`.
  - File size and type validation before upload via `validateFile`.
  - Secure `Base64::onUpload` (from `Nano.Api`).

- **Flexibility**:
  - Register any model with custom settings per field.
  - Specify `allowed_user_types` per model or field.
  - Permissions per operation (`add`, `edit`, `delete`, `view`) at model or field level.
  - Ability to customize user resolver and permission checker.
  - Support for multiple upload formats (multipart, base64).
  - Temporary upload for unsaved models.

- **Performance**:
  - Temporary session keys avoid storing unassociated files.
  - Ability to retrieve files with custom thumbnails.
  - Model settings are cached in memory after loading.

- **Extensibility**:
  - Models can be registered from any plugin via `registerFileUploadFields` or the `nano.api.fileupload.registerModels` event.
  - Logic can be extended via events (`modelRegistered`).
  - Clear separation between registry, service, user, and API layers.

- **Developer Experience**:
  - Simple and unified API.
  - Comprehensive documentation (originally in Arabic, now translated).
  - Support for single and multiple uploads.
  - Automatic temporary file management.

---

## 6. Complete Practical Example

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
        return response()->json(['error' => $uploadResult['error']], 400);
    }

    // Create the product
    $product = new Product();
    $product->name = $productName;
    $product->save();

    // Link the image to the product
    $service->attachTempFiles($product, 'image', $tempKey);

    return response()->json(['product_id' => $product->id]);
}
```

---

## 7. Dependencies and Requirements

- **PHP** (>= 8.0).
- **Laravel / OctoberCMS** (any framework compatible with `System\Models\File`).
- **`Nano.Api`** plugin (to provide `Base64` and `ApiController`).
- **Composer** to install the package.

---

## 8. Conclusion

The `Nano.FileUpload` classes represent a comprehensive and professional solution for managing file uploads in NanoSoft applications. Thanks to the layered design (Registry, Service, UserManager, API), developers can build powerful and secure file upload systems quickly and easily. With the addition of `FileUploadService`, the system is more organized and easier to use, providing a single entry point for upload, delete, and retrieval operations, with advanced options such as temporary upload, thumbnails, and permission checks for different user types. This design ensures scalability and maintainability, meeting the needs of advanced applications without compromising security or performance.

---

## 9. Additional Documentation

For more details about each class, please refer to the following documents:

- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
