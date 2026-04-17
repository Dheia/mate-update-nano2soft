# Advanced and Practical Examples for `FileUploadRegistry` Class Documentation

**Namespace:** `Nano\FileUpload\Classes`  
**Purpose:** Provide comprehensive practical examples for using `FileUploadRegistry` in various scenarios, from basic registration of models and upload fields to advanced permission checking, event management, constraint querying, and integration with `FileUploadService`, including expected inputs/outputs and best practices.

> **Note:** This document covers features up to version 1.0.7, including global settings for controlling automatic transformations, multiple storage, event hooks, and security and caching improvements.

---

## Introduction

This document provides a set of practical examples covering various use cases of the `FileUploadRegistry` class, which represents the **central registry** for managing model definitions and file upload fields in NanoSoft applications. Through these examples, you will learn how to:

- Register models and their fields with advanced settings (multiple storage, automatic resizing, watermarking).
- Specify allowed user types (`backend`/`frontend`).
- Set granular permissions for each operation (`add`, `edit`, `delete`, `view`) at the model or field level.
- Use validation functions (`can`, `isUserTypeAllowed`, `getFieldConstraints`, `getProcessingOptions`) to ensure security.
- Listen to events (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) to extend the system.
- Use caching to improve performance.
- Integrate `FileUploadRegistry` with `FileUploadService` to upload files.
- Handle errors and apply best practices.

> **Note:** The examples here interact directly with the `FileUploadRegistry` class. In real applications, it is often used via `FileUploadService`, but understanding the underlying settings helps in precisely customizing the system.

---

## Prerequisites

- `Nano.FileUpload` add-on installed and configured in your application (version 1.0.7 or later).
- The add-on depends on `Nano.Api` which provides `Base64` and `ApiController`.
- Basic knowledge of Models and relationships in Laravel / NanoSoft applications.
- (Optional) Use `FileUploadUserManager` to check user permissions.

### Setting up the class in the application

```php
use Nano\FileUpload\Classes\FileUploadRegistry;

$registry = FileUploadRegistry::instance();
```

Since the class follows the Singleton pattern, it is called via `instance()`.

---

## 1. Register a Model with Simple Settings

**Scenario:** Register a `Product` model with a single `image` field (main image) using default settings.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'image' => [
            'type' => 'image',
            'label' => 'Main Image',
            'max_filesize' => 1024, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
        ],
    ],
]);
```

**Output:** The model is added to `$rawDefinitions`. You can verify with:

```php
if ($registry->isModelRegistered(\Nano\Shop\Models\Product::class)) {
    echo "Model registered successfully";
}
```

**Notes:**  
- `type` can be `image`, `file`, `audio`, `video`, or `multiple` (for multiple fields).  
- `max_filesize` is in kilobytes (KB).  
- `allowed_types` is a comma-separated list of extensions or `*` to allow all types.  
- If `label` is not provided, the field name will be used.

---

## 2. Register a Model with Advanced Settings (Multiple Storage and Automatic Image Transformation)

**Scenario:** Register a `Product` model with an `image` field that has:
- Custom storage disk (e.g., `s3`).
- Automatic resizing to width 800 and height 600 (crop mode).
- Automatic watermark in the bottom-right corner at 15% size.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'image' => [
            'type' => 'image',
            'storage_disk' => 's3',                   // custom storage disk
            'auto_resize' => true,
            'resize_options' => [
                'width' => 800,
                'height' => 600,
                'mode' => 'crop',
            ],
            'auto_watermark' => true,
            'watermark_options' => [
                'position' => 'bottom-right',
                'resize_percentage' => 15,
            ],
        ],
    ],
]);
```

**Check processing options:**

```php
$procOptions = $registry->getProcessingOptions(\Nano\Shop\Models\Product::class, 'image');
// $procOptions['storage_disk'] => 's3'
// $procOptions['auto_resize'] => true
// $procOptions['resize_options']['width'] => 800
```

**Notes:**  
- The `s3` disk must be pre-configured in `config/filesystems.php`.  
- `auto_resize` and `auto_watermark` work only if the field type is `image`.  
- Automatic transformations can be disabled globally via global settings (see Example 4).

---

## 3. Register a Model with Advanced Settings (User Types and Permissions)

**Scenario:** Register a `User` model with an `avatar` field, specifying that only `frontend` users are allowed, and each operation requires a specific permission.

```php
$registry->registerModel(\RainLab\User\Models\User::class, [
    'allowed_user_types' => ['frontend'], // only frontend users
    'permissions' => [
        'add'    => null, // no additional permission needed (allowed for everyone)
        'delete' => 'user.delete_avatar',
        'view'   => null,
    ],
    'fields' => [
        'avatar' => [
            'type' => 'image',
            'max_filesize' => 2048,
            'allowed_types' => 'jpg,jpeg,png',
            'permissions' => [
                'add' => 'user.upload_avatar', // override general add permission
            ],
        ],
    ],
]);
```

**Check permission:**

```php
$user = \Auth::getUser(); // frontend user
$userType = $registry->getUserTypeFromModel($user); // should return 'frontend'

if ($registry->can(\RainLab\User\Models\User::class, 'add', $userType, $user, 'avatar')) {
    echo "Allowed to upload image";
} else {
    echo "Not allowed";
}
```

**Output:** If the user has the `user.upload_avatar` permission (in the frontend permission system), returns `true`. Otherwise, `false`.

**Notes:**  
- Field permissions override model permissions.  
- If a permission is `null`, access is automatically allowed after checking the user type.

---

## 4. Using Global Settings to Control Automatic Transformations

**Scenario:** Disable all automatic resizing operations at the entire API level without modifying field definitions.

In `.env` file:
```ini
NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
```

In code:
```php
if ($registry->isAutoResizeEnabledGlobally()) {
    echo "Auto resize is enabled";
} else {
    echo "Auto resize is globally disabled";
}
```

**Note:** The same applies to `disable_auto_watermark`, `disable_upload`, `disable_delete`, `disable_get`, and `disable_edit`.

---

## 5. Register a Multiple Field with Additional Constraints

**Scenario:** Add a `gallery` image gallery that allows up to 10 images, each with a maximum size of 1 MB, and only jpg/png formats.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'gallery' => [
            'type' => 'multiple',
            'label' => 'Image Gallery',
            'max_filesize' => 1024,
            'allowed_types' => 'jpg,jpeg,png',
            'max_files' => 10,
            'multiple' => true,
            'required' => false,
        ],
    ],
]);
```

**Query constraints:**

```php
$constraints = $registry->getFieldConstraints(\Nano\Shop\Models\Product::class, 'gallery');
// $constraints['max_files'] == 10
// $constraints['multiple'] == true
```

---

## 6. Using the `nano.api.fileupload.registerModels` Event to Register Models from Different Plugins

**Scenario:** Register multiple models through an event listener, allowing registrations to be grouped in one place.

```php
Event::listen('nano.api.fileupload.registerModels', function ($registry) {
    $registry->registerModel(\Nano\Shop\Models\Product::class, [
        'allowed_user_types' => ['backend'],
        'fields' => [
            'image' => ['type' => 'image', 'max_filesize' => 1024],
            'gallery' => ['type' => 'multiple', 'multiple' => true, 'max_files' => 10],
        ],
    ]);

    $registry->registerModel(\RainLab\User\Models\User::class, [
        'allowed_user_types' => ['frontend'],
        'fields' => [
            'avatar' => ['type' => 'image', 'max_filesize' => 2048],
        ],
    ]);
});
```

**Notes:**  
- This event is automatically dispatched by `FileUploadRegistry` when `getRegisteredModels()` is called.  
- It can be placed in the main add-on's `boot()` method.

---

## 7. Listening to the `nano.api.fileupload.modelRegistered` Event for Post-Registration Actions

**Scenario:** Register a model and then dynamically register an API route based on its fields.

```php
Event::listen('nano.api.fileupload.modelRegistered', function ($modelClass, $config) {
    // Can add a custom route to get files for this model
    \Route::get('/api/custom/' . class_basename($modelClass) . '/files', function () use ($modelClass) {
        return $registry->getFiles($modelClass, 'image');
    });
});
```

---

## 8. Checking User Type (`isUserTypeAllowed`) and Global Settings

**Scenario:** Display a list of fields only if the user is of an allowed type and upload is globally enabled.

```php
$userType = $userManager->getUserType(); // backend, frontend, guest
$modelClass = \Nano\Shop\Models\Product::class;

if ($registry->isUserTypeAllowed($modelClass, $userType) && $registry->isUploadEnabledGlobally()) {
    // Show file upload UI
    $fields = $registry->getModelConfig($modelClass)['fields'];
    foreach ($fields as $field => $config) {
        echo "Field: {$field} - Type: {$config['type']}\n";
    }
} else {
    echo "Access not allowed or upload is disabled";
}
```

---

## 9. Using `can` to Check Delete Permission with Global Settings

**Scenario:** Check whether the current user has permission to delete a file from a specific field, considering global settings.

```php
$user = \BackendAuth::getUser(); // backend user
$userType = $registry->getUserType(); // 'backend'
$modelClass = \Nano\Shop\Models\Product::class;
$field = 'image';

if ($registry->can($modelClass, 'delete', $userType, $user, $field)) {
    // Show delete button
} else {
    // Hide delete button
}
```

**Output:** If the user has `delete` permission (at model or field level) and deletion is globally enabled (`disable_delete = false`), returns `true`.

---

## 10. Getting Field Constraints and Validating Before Upload

**Scenario:** In the upload interface, display the maximum size and allowed types to the user, and validate before sending the request.

```php
$constraints = $registry->getFieldConstraints(\Nano\Shop\Models\Product::class, 'image');
if ($constraints['max_filesize']) {
    echo "Maximum size: {$constraints['max_filesize']} KB\n";
}
if ($constraints['allowed_types'] != '*') {
    echo "Allowed types: {$constraints['allowed_types']}\n";
}
if ($constraints['required']) {
    echo "This field is required\n";
}
```

---

## 11. Using Cache to Improve Performance

**Scenario:** Manually enable caching (although default settings are enabled) and set a longer TTL.

```php
$registry->setCacheEnabled(true);
$registry->setCacheTtl(7200); // two hours

// After updating a model definition, you can clear its cache
$registry->clearModelCache(\Nano\Shop\Models\Product::class);
```

**Note:** Cache can be controlled globally via environment variables:
```ini
NANO_FILE_UPLOAD_REGISTRY_CACHE_ENABLED=true
NANO_FILE_UPLOAD_REGISTRY_CACHE_TTL=3600
```

---

## 12. Integrating `FileUploadRegistry` with `FileUploadService` in a Complete Upload Process Using Advanced Features

**Scenario:** Create a new `Product` model and upload a main image for it with automatic resizing and watermarking as defined in the registration.

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadRegistry;

$registry = FileUploadRegistry::instance();
$service = FileUploadService::instance();

$modelClass = \Nano\Shop\Models\Product::class;
$field = 'image';

// Check if the model is registered
if (!$registry->isModelRegistered($modelClass)) {
    die('Model not registered');
}

// Generate a temporary session key
$tempKey = $service->generateTempSessionKey($modelClass, $field);

// Upload the image (assuming $uploadedFile is an UploadedFile object)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey
]);

if ($uploadResult['status']) {
    // Create and save the product
    $product = new \Nano\Shop\Models\Product();
    $product->name = 'New Product';
    $product->save();

    // Attach the temporary files to the saved model
    $service->attachTempFiles($product, $field, $tempKey);

    echo "Image uploaded and attached to product";
} else {
    echo "Upload failed: " . $uploadResult['error'] . " (code: " . $uploadResult['error_code'] . ")";
}
```

**Notes:**  
- `attachTempFiles` will search for files with `session_key = $tempKey` and update `attachment_id` and `attachment_type` to attach them to the model.  
- If the field is `multiple`, all files will be added to the relation.

---

## 13. Handling Registration Errors (Non-existent Model)

**Scenario:** Attempt to register a non-existent model.

```php
try {
    $registry->registerModel('NonExistentModel', []);
} catch (\ApplicationException $e) {
    echo "Error: " . $e->getMessage(); // "Model NonExistentModel does not exist"
}
```

---

## 14. Updating a Registered Model's Settings (e.g., Changing Add Permission)

**Scenario:** After registering the model, we want to change the add permission for the `image` field from `null` to a specific permission.

```php
$registry->updateModelConfig(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'image' => [
            'permissions' => [
                'add' => 'nano.shop.upload_image_v2',
            ],
        ],
    ],
]);
```

Now when `can` is called for the `image` field, the new permission will be used.

#### 14.1 Updating a Specific Field's Settings Using `updateFieldConfig` (New in version 1.1.0)

**Scenario:** After registering the model, we want to change the maximum size of the `image` field from 1 MB to 2 MB without re-registering the entire model.

```php
$registry = FileUploadRegistry::instance();

$registry->updateFieldConfig(\Nano\Shop\Models\Product::class, 'image', [
    'max_filesize' => 2048, // 2 MB
]);
```

**Verification:** Calling `getFieldConfig` will return the new value.

```php
$config = $registry->getFieldConfig(\Nano\Shop\Models\Product::class, 'image');
echo $config['max_filesize']; // 2048
```

**Notes:**  
- The function merges new settings with existing ones (`array_replace_recursive`) and applies normalization (`normalizeFieldDefinition`).  
- It clears the cache for this specific field only, so it does not affect other fields.  
- Throws `ApplicationException` if the model or field is not registered.

#### 14.2 Updating `edit` Permission for a Specific Field (New in version 1.1.0)

**Scenario:** Add an `edit` permission to the `image` field to allow replacing the image.

```php
$registry->updateFieldConfig(\Nano\Shop\Models\Product::class, 'image', [
    'permissions' => [
        'edit' => 'nano.shop.product.edit_image',
    ],
]);
```

**Verification:** Use `can` with the `edit` operation:

```php
$user = \BackendAuth::getUser();
$userType = 'backend';
if ($registry->can(Product::class, 'edit', $userType, $user, 'image')) {
    echo "Allowed to replace the image";
}
```

---

## 15. Using Defaults to Reduce Duplication

**Scenario:** Many fields share the same settings (size 2 MB, image types, public). You can specify `defaults` and then override individual fields.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'defaults' => [
        'max_filesize' => 2048,
        'allowed_types' => 'jpg,jpeg,png',
        'is_public' => true,
    ],
    'fields' => [
        'image' => [
            'type' => 'image',
            'required' => true,
        ],
        'gallery' => [
            'type' => 'multiple',
            'max_files' => 10,
            'multiple' => true,
        ],
    ],
]);
```

In `image`, `max_filesize=2048` and `allowed_types=...` will be used, but `required` is set to `true`.  
In `gallery`, the same defaults will be used, with `max_files` and `multiple` added.

---

## 16. Getting Full Model or Field Configuration

**Scenario:** Print all settings of a specific model for debugging purposes.

```php
$config = $registry->getModelConfig(\Nano\Shop\Models\Product::class);
print_r($config);
```

**Result:** An array containing `enabled`, `allowed_user_types`, `permissions`, `fields`, `defaults`.

---

## 17.1. Checking `view` Permission Before Displaying File List (Considering Global Settings)

**Scenario:** In an API controller, check if the user has permission to view files for a specific model, and if retrieval is globally enabled.

```php
public function getFiles(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $user = \Auth::getUser();
    $userType = $this->userManager->getUserType();

    $registry = FileUploadRegistry::instance();
    if (!$registry->can($modelClass, 'view', $userType, $user, $field)) {
        return response()->json(['error' => 'Not allowed'], 403);
    }

    // Retrieve files...
}
```

## 17.2. Using the Global `disable_edit` Setting (New in version 1.1.0)

**Scenario:** Disable file replacement operations at the entire API level without modifying field definitions.

In `.env` file:
```ini
NANO_FILE_UPLOAD_DISABLE_EDIT=true
```

In code:
```php
if ($registry->isEditEnabledGlobally()) {
    echo "File replacement is enabled";
} else {
    echo "File replacement is globally disabled";
}
```

**Note:** This value affects `FileUploadRegistry::can('edit')` and prevents any `edit` operation even if the permission exists.

---

## 18. Testing Field Permission Using a Path Column (for Reporting) – Not directly applicable to FileUploadRegistry, but an example of checking `view` permission for the field itself.

**Scenario:** Check that the user has permission to view a specific field before displaying it in the UI.

```php
$user = \Auth::getUser();
$userType = 'frontend';
$modelClass = \RainLab\User\Models\User::class;
$field = 'avatar';

if ($registry->can($modelClass, 'view', $userType, $user, $field)) {
    echo '<img src="' . $user->avatar->getPath() . '">';
} else {
    echo 'Image not shown';
}
```

---

## 19. Best Practices for Modern Versions (1.0.7)

1. **Register models early**: Use the `nano.api.fileupload.registerModels` event in the main add-on's `boot()`.
2. **Use `allowed_user_types` wisely**: Do not allow all types if not needed.
3. **Set granular permissions for each operation**: At least `add` and `delete`, to prevent unauthorized uploads or deletions.
4. **Use `getFieldConstraints` for pre-validation**: Before uploading a file, check size and types to avoid server errors.
5. **Document field settings**: Use `label` and `description` to make it easier for other developers to understand.
6. **Use `defaults`** to reduce duplication and simplify maintenance.
7. **Do not rely solely on global permissions**: Customize permissions per field when needed.
8. **Monitor events**: `modelRegistered` can be used to register API routes or additional initialization.
9. **Test permissions**: Ensure unauthorized users cannot upload files via the API.
10. **Use `FileUploadService` instead of directly interacting with `FileUploadRegistry` for upload operations**: To ensure all checks pass through the correct layer.
11. **Use global settings (`disable_upload`, `disable_auto_resize`) as an emergency measure** to temporarily disable operations without modifying code.
12. **Enable caching in production** (`cache.enabled = true`) for better performance, with an appropriate TTL.
13. **Use `storage_disk` only when needed** – prefer the default disk to simplify management.
14. **Set `auto_resize` and `auto_watermark` at the field level** to avoid unnecessary processing for non-image files.

---

## 20. Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `ApplicationException: Model ... does not exist` | Attempting to register a non-existent model. | Ensure the class name is correct and imported. |
| Field does not appear in UI | `allowed_user_types` does not include the current user type. | Check `$userType` or add the appropriate type. |
| `add` permission does not work | Permission not set in `permissions` or `permissions[add]` missing. | Specify the required permission or set it to `null` to allow everyone. |
| `isModelRegistered` returns `false` | Called `getRegisteredModels()` after registration? | Ensure registration happens before querying (the function now auto-loads definitions). |
| Temporary files are not attached | Incorrect `temp_session_key` passed to `attachTempFiles`. | Use the same key returned by `upload`. |
| `Auto-resize failed` | GD or Imagick library not installed, or file path invalid. | Install GD or Imagick and check file permissions. |
| `Auto-watermark failed` | `Nano2.Watermark` add-on not installed or logo path incorrect. | Install the add-on and ensure the logo file exists at the specified path. |
| `Storage disk 's3' not found` | `s3` disk not configured in `config/filesystems.php`. | Add appropriate disk configuration. |
| `Auto resize does not work despite being enabled` | `disable_auto_resize` is set to `true` in global settings. | Check the environment variable or disable it. |

---

## 21. Integration with NanoSoft Permission System

In NanoSoft applications, `FileUploadUserManager` is used to check permissions. You can set a custom permission checker for `frontend` users:

```php
FileUploadUserManager::instance()->setPermissionChecker(function ($user, $operation, $permissions) {
    // Custom logic to check frontend user permissions
    // e.g., check a custom permissions table
    return $user->hasCustomPermission($operation, $permissions);
});
```

---

## Summary

The `FileUploadRegistry` class is the backbone of file upload management in NanoSoft applications. Through the examples above, developers can register any model with flexible settings (including multiple storage and automatic transformation), finely control user types and permissions, leverage caching and events, and integrate the system with `FileUploadService` to securely upload files. We recommend using default settings and leveraging events to extend the system, and checking permissions at every entry point to ensure the highest levels of security.

For details on the response structure when using the `FileUploadController`, refer to [API Documentation](./Docs-API-Documentation-en.md).

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
