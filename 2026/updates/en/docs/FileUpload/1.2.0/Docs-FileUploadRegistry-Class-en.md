# `FileUploadRegistry` Class Documentation

**Overview of the `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**, specifically designed for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes in this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, as well as unified error handling. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

---

### Introduction

The `FileUploadRegistry` class is the **Central Registry** of the file upload system in NanoSoft applications. This class manages all registered models (modules) that contain file upload fields, and provides a unified interface for querying settings of these models and fields, and checking user permissions and types before performing operations.

Simply put, `FileUploadRegistry` is **where all definitions of models and upload fields live**, and it is the means used by other system components (such as `FileUploadService`) to access settings and permissions information.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$rawDefinitions` | `array` | Raw definitions of models (from add-ons and callbacks). |
| `$builtModels` | `array` | Built model objects after construction (used for caching). |
| `$loaded` | `bool` | Whether definitions have been loaded from add-ons. |
| `$cacheEnabled` | `bool` | Enable/disable caching of query results. |
| `$cacheTtl` | `int` | Cache time-to-live in seconds. |

---

### Important Methods

#### 1. Registering Models

##### `public function registerModel(string $modelClass, array $config = []): self`
- **Purpose**: Register a single model with its custom settings (simplified interface, calls `registerDefinition`).
- **Parameters**:
  - `$modelClass`: Fully qualified class name of the model (e.g., `Nano\Shop\Models\Product`).
  - `$config`: Array of model settings (see configuration structure below).
- **Returns**: Current instance for method chaining.

##### `public function registerDefinition(string $modelClass, array $definition): void`
- **Purpose**: Register a raw model definition (used internally, can be used directly).
- **Parameters**:
  - `$modelClass`: Fully qualified class name of the model.
  - `$definition`: Model definition (same structure as `$config` in `registerModel`).
- **Behavior**: Applies normalization (`normalizeModelDefinition`), stores in `$rawDefinitions`, and clears cache.

##### `public function registerDefinitions(array $definitions): void`
- **Purpose**: Register a batch of definitions at once.

##### `public static function registerCallback(callable $callback): void`
- **Purpose**: Register a callback function to be executed when loading definitions (useful for add-ons that do not use `registerFileUploadFields`).

**Configuration Structure (`$config`):**
```php
[
    'enabled' => true,
    'allowed_user_types' => ['backend', 'frontend'],
    'permissions' => [
        'add'    => null,
        'edit'   => null,   // ⬅️ New in version 1.1.0
        'delete' => null,
        'view'   => null,
    ],
    'fields' => [
        'image' => [
            'type' => 'image', // image, audio, video, file, multiple
            'label' => 'Main Image',
            'max_filesize' => 2048, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'max_files' => null, // only for multiple fields
            'is_public' => true,
            'use_caption' => true,
            'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
            'allowed_user_types' => ['backend'],
            'permissions' => [ // field-specific permissions (override model permissions)
                'add'    => 'nano.shop.upload_image',
                'edit'   => 'nano.shop.edit_image',   // ⬅️ New
                'delete' => 'nano.shop.delete_image',
            ],
            // ⬇⬇⬇ Advanced settings (versions 1.0.6+) ⬇⬇⬇
            'storage_disk'      => null,      // custom storage disk (e.g., 's3')
            'auto_resize'       => false,     // enable automatic resizing
            'resize_options'    => [],        // resize options (width, height, mode)
            'auto_watermark'    => false,     // enable automatic watermark
            'watermark_options' => [],        // watermark options
        ],
    ],
    'defaults' => [ // default settings for fields (can be overridden per field)
        'max_filesize' => 2048,
        'allowed_types' => '*',
        'is_public' => true,
        'use_caption' => true,
        'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
    ],
]
```

#### 2. Getting Registered Models

##### `public function getRegisteredModels(): array`
- **Purpose**: Return all registered models (loads definitions first).

##### `public function isModelRegistered(string $modelClass): bool`
- **Purpose**: Check if a specific model is registered (loads definitions automatically).

##### `public function getModelConfig(string $modelClass): ?array`
- **Purpose**: Get the full configuration of a model.

##### `public function getFieldConfig(string $modelClass, string $field): ?array`
- **Purpose**: Get the configuration of a specific field (supports caching).

#### 3. Checking Permissions and Global Settings

##### `public function isUploadEnabledGlobally(): bool`
- **Purpose**: Check if upload is globally enabled (from `general.disable_upload` setting).

##### `public function isEditEnabledGlobally(): bool` **(New in version 1.1.0)**
- **Purpose**: Check if editing (file replacement) is globally enabled (from `general.disable_edit` setting).

##### `public function isDeleteEnabledGlobally(): bool`
- **Purpose**: Check if deletion is globally enabled.

##### `public function isGetEnabledGlobally(): bool`
- **Purpose**: Check if retrieval is globally enabled.

##### `public function isAutoResizeEnabledGlobally(): bool`
- **Purpose**: Check if automatic resizing is globally enabled (from `general.disable_auto_resize` setting).

##### `public function isAutoWatermarkEnabledGlobally(): bool`
- **Purpose**: Check if automatic watermark is globally enabled.

##### `public function isUserTypeAllowed(string $modelClass, string $userType): bool`
- **Purpose**: Check if the given user type is allowed to access the model.

##### `public function can(string $modelClass, string $operation, string $userType, $user, ?string $field = null): bool`
- **Purpose**: Check if the user has the required permission for a given operation (`add`, `edit`, `delete`, `view`) on a model or a specific field.
- **Behavior**: First checks global settings (e.g., `disable_upload`, `disable_edit`, etc.), then model existence, then user type, then permissions defined in the model or field, and finally calls `FileUploadUserManager::checkPermission`.

#### 4. Getting Field Constraints and Advanced Processing Options

##### `public function getFieldConstraints(string $modelClass, string $field): ?array`
- **Purpose**: Extract field constraints (max size, allowed types, etc.) in a format suitable for validation.
- **Returns**: Array containing `max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`.

##### `public function getProcessingOptions(string $modelClass, string $field): ?array`
- **Purpose**: Get advanced processing options for a specific field (storage, resizing, watermark).
- **Returns**: Array containing `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.

#### 5. Cache Management

##### `public function clearModelCache(string $modelClass): void`
- **Purpose**: Clear all cache keys for a specific model (e.g., `field_config`, `constraints`).

##### `public function clearFieldCache(string $modelClass, string $fieldName): void` **(New in version 1.1.0)**
- **Purpose**: Clear the cache for a specific field in a model (used internally by `updateFieldConfig`).

##### `public function setCacheEnabled(bool $enabled): void`
- **Purpose**: Enable/disable caching.

##### `public function setCacheTtl(int $ttl): void`
- **Purpose**: Set the cache time-to-live.

#### 6. Updating a Registered Model's Settings

##### `public function updateModelConfig(string $modelClass, array $config): self` **(Enhanced in version 1.1.0)**
- **Purpose**: Update the settings of a previously registered model (merges with existing settings).
- **Enhancement**: Now re-normalizes the entire model using `normalizeModelDefinition` after merging, ensuring all default fields are complete.

##### `public function updateFieldConfig(string $modelClass, string $fieldName, array $newConfig): self` **(New in version 1.1.0)**
- **Purpose**: Update the settings of a specific field without needing to re-register the entire model.
- **Parameters**:
  - `$modelClass`: Fully qualified class name of the model.
  - `$fieldName`: Name of the field to update.
  - `$newConfig`: New settings (merged with existing settings).
- **Behavior**:
  1. Checks existence of model and field.
  2. Merges new settings with existing using `array_replace_recursive`.
  3. Applies normalization via `normalizeFieldDefinition`.
  4. Updates raw definition (`rawDefinitions`).
  5. Resets built model (`builtModels`).
  6. Clears cache for this specific field via `clearFieldCache`.
- **Returns**: Current instance for method chaining.
- **Exceptions**: Throws `ApplicationException` if model or field is not registered.

---

### Events

| Event Name | Description |
|------------|-------------|
| `nano.api.fileupload.registerModels` | Dispatched when registered models need to be loaded, passing a `FileUploadRegistry` object so developers can add their models by listening to this event. |
| `nano.api.fileupload.modelRegistered` | Dispatched after a model is successfully registered, passing `$modelClass` and `$config`. |

---

### Comprehensive Practical Examples

#### Example 1: Register a Model from within an Plugin (with advanced settings)

```php
// In Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'permissions' => [
                'add'    => 'nano.shop.product.upload',
                'edit'   => 'nano.shop.product.edit',   // new
                'delete' => 'nano.shop.product.delete',
            ],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'auto_resize' => true,
                    'resize_options' => ['width' => 800, 'height' => 600, 'mode' => 'crop'],
                    'auto_watermark' => true,
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

#### Example 2: Register a Model via an Event

```php
Event::listen('nano.api.fileupload.registerModels', function ($registry) {
    $registry->registerModel(\RainLab\User\Models\User::class, [
        'allowed_user_types' => ['frontend'],
        'fields' => [
            'avatar' => [
                'type' => 'image',
                'max_filesize' => 2048,
                'allowed_types' => 'jpg,jpeg,png',
            ],
        ],
    ]);
});
```

#### Example 3: Update a Specific Field's Settings (New)

```php
$registry = FileUploadRegistry::instance();
// Update the max size of the 'image' field to 2 MB
$registry->updateFieldConfig('Nano\Shop\Models\Product', 'image', [
    'max_filesize' => 2048,
]);
// Update the edit permission for the 'image' field
$registry->updateFieldConfig('Nano\Shop\Models\Product', 'image', [
    'permissions' => [
        'edit' => 'nano.shop.product.edit_image',
    ],
]);
```

#### Example 4: Check Permission Before Uploading a File (considering global settings)

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();
$userType = $userManager->getUserType();
$modelClass = 'Nano\Shop\Models\Product';
$field = 'image';

if ($registry->can($modelClass, 'add', $userType, $user, $field)) {
    // Allowed to upload the image
} else {
    // Not allowed (may be due to global settings or permissions)
}
```

#### Example 5: Get Constraints of a Specific Field

```php
$constraints = $registry->getFieldConstraints('Nano\Shop\Models\Product', 'image');
// $constraints['max_filesize'] => 1024
// $constraints['allowed_types'] => 'jpg,jpeg,png'
```

#### Example 6: Get Advanced Processing Options

```php
$procOptions = $registry->getProcessingOptions('Nano\Shop\Models\Product', 'image');
// $procOptions['auto_resize'] => true
// $procOptions['storage_disk'] => 's3'
```

#### Example 7: Update a Registered Model's Settings (e.g., change add permission)

```php
$registry->updateModelConfig('Nano\Shop\Models\Product', [
    'permissions' => [
        'add' => 'nano.shop.product.upload_v2',
    ],
]);
```

#### Example 8: Check User Type Before Displaying Upload UI (with global settings)

```php
$userType = $userManager->getUserType();
if ($registry->isUserTypeAllowed('Nano\Shop\Models\Product', $userType)) {
    // Show image upload UI
} else {
    // Hide UI
}
```

#### Example 9: Disable Auto Resize or Edit Globally via Environment Variables

```ini
NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
NANO_FILE_UPLOAD_DISABLE_EDIT=true
```

Then in code:
```php
if ($registry->isAutoResizeEnabledGlobally()) {
    // Auto resize is enabled
} else {
    // Disabled
}

if ($registry->isEditEnabledGlobally()) {
    // File replacement operation is enabled
} else {
    // Disabled
}
```

---

### Interaction with Other Classes

- **With `FileUploadService`**: `FileUploadService` relies on `FileUploadRegistry` to get model settings (`getFieldConfig`, `getProcessingOptions`) and check permissions (`can`), including the new `edit` operation.
- **With `FileUploadUserManager`**: `FileUploadRegistry` uses the `FileUploadUserManager::instance()` object to actually check permissions via `checkPermission()`.
- **With `Base64` (moved inside the add-on)**: No direct interaction, but `FileUploadService` uses `Base64` for uploading.

---

### Best Practices

1. **Register models at application startup**: Register all models once when initializing the add-on (in `boot()` or `register()`).
2. **Use the `nano.api.fileupload.registerModels` event**: To unify model registration from different add-ons.
3. **Be precise about allowed user types**: Do not allow all types if not needed.
4. **Define permissions for each operation**: Use appropriate permissions (e.g., `add`, `edit`, `delete`, `view`) to restrict access.
5. **Use field constraints (`getFieldConstraints`)**: To pre-validate file size and type before upload.
6. **Use advanced processing options (`getProcessingOptions`)** to apply automatic resizing and watermarking.
7. **Enable caching in production environment**: For better performance (default settings are enabled).
8. **Use global settings (`disable_upload`, `disable_edit`, `disable_auto_resize`)** as an emergency measure to temporarily disable operations.
9. **Use `updateFieldConfig` to modify a single field** instead of re-registering the entire model.

---

### Summary

`FileUploadRegistry` is the heart of the file upload system in NanoSoft applications. It aggregates all system knowledge about models that contain file upload fields, and provides a clean and efficient programming interface to access this information and check permissions. Thanks to its support for different user types (`backend`, `frontend`), granular permissions per operation (including `edit`), global settings to control API and automatic transformations, caching, and flexible update functions (`updateModelConfig`, `updateFieldConfig`), developers can build robust and secure file upload systems without worrying about security or performance. This class integrates closely with `FileUploadService` and `FileUploadUserManager` to form a complete layer that manages all aspects of file uploads in the application.

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
