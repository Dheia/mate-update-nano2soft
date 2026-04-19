## `FileUploadRegistry` Class Documentation – Version 1.2.3

**Overview of `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**, specifically designed for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes in this module reside under the following Namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, as well as unified error handling. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

---

### Introduction

The `FileUploadRegistry` class is the **Central Registry** for the file upload system in NanoSoft applications. This class manages all registered models (modules) that contain file upload fields, and provides a unified interface for querying settings of these models and fields, and checking user permissions and types before executing operations.

Simply put, `FileUploadRegistry` is **where all model and upload field definitions live**, and is the means by which other system components (such as `FileUploadService`) access settings and permission information.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$rawDefinitions` | `array` | Raw model definitions (from add-ons and callbacks). |
| `$builtModels` | `array` | Built model objects after construction (used for caching). |
| `$loaded` | `bool` | Whether definitions have been loaded from add-ons. |
| `$cacheEnabled` | `bool` | Enable/disable caching of query results. |
| `$cacheTtl` | `int` | Cache time-to-live in seconds. |

---

### Important Methods

#### 1. Model Registration

##### `public function registerModel(string $modelClass, array $config = []): self`
- **Purpose**: Register a single model with its custom settings (simplified interface, calls `registerDefinition`).
- **Parameters**:
  - `$modelClass`: Full class name of the model (e.g., `Nano\Shop\Models\Product`).
  - `$config`: Model settings array (see configuration structure below).
- **Returns**: Current instance for method chaining.

##### `public function registerDefinition(string $modelClass, array $definition): void`
- **Purpose**: Register a raw model definition (used internally, can be used directly).
- **Parameters**:
  - `$modelClass`: Full class name of the model.
  - `$definition`: Model definition (same structure as `$config` in `registerModel`).
- **Behavior**: Applies normalization (`normalizeModelDefinition`), stores in `$rawDefinitions`, clears cache.

##### `public function registerDefinitions(array $definitions): void`
- **Purpose**: Register multiple definitions in bulk.

##### `public static function registerCallback(callable $callback): void`
- **Purpose**: Register a callback function to be executed when loading definitions (useful for add-ons that don't use `registerFileUploadFields`).

**Configuration Structure (`$config`):**
```php
[
    'enabled' => true,
    'allowed_user_types' => ['backend', 'frontend'],
    'permissions' => [
        'add'    => null,
        'edit'   => null,
        'delete' => null,
        'view'   => null,
    ],
    'disabled_operations' => [], // ➡️ Disable specific operations at model level (new in 1.2.1)
    'fields' => [
        'image' => [
            'type' => 'image', // image, audio, video, file, multiple
            'label' => 'Main Image',
            'max_filesize' => 2048, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'max_files' => null,
            'is_public' => true,
            'use_caption' => true,
            'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
            'allowed_user_types' => ['backend'],
            'permissions' => [ // Field-specific permissions (override model permissions)
                'add'    => 'nano.shop.upload_image',
                'edit'   => 'nano.shop.edit_image',
                'delete' => 'nano.shop.delete_image',
            ],
            'disabled_operations' => [], // ➡️ Disable operations at field level (new in 1.2.1)
            'storage_disk'      => null,
            'auto_resize'       => false,
            'resize_options'    => [],
            'auto_watermark'    => false,
            'watermark_options' => [],
        ],
    ],
    'defaults' => [ // Default field settings
        'max_filesize' => 2048,
        'allowed_types' => '*',
        'is_public' => true,
        'use_caption' => true,
        'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
    ],
]
```

> **Note about `disabled_operations`**: Specific operations (`add`, `edit`, `delete`, `view`) can be disabled independently. If an operation is disabled at model level, it will be prevented for all its fields unless overridden at field level. This property is useful for temporarily disabling upload without changing permissions.

#### 2. Getting Registered Models

##### `public function getRegisteredModels(): array`
- **Purpose**: Return all registered models (loading definitions first).

##### `public function isModelRegistered(string $modelClass): bool`
- **Purpose**: Check if a specific model is registered (automatically loads definitions).

##### `public function getModelConfig(string $modelClass): ?array`
- **Purpose**: Get the full configuration of a model.

##### `public function getFieldConfig(string $modelClass, string $field): ?array`
- **Purpose**: Get configuration for a specific field (supports caching).

#### 3. Permission and Global Settings Checks

##### `public function isUploadEnabledGlobally(): bool`
- **Purpose**: Check if upload is globally enabled (from `general.disable_upload` setting).

##### `public function isEditEnabledGlobally(): bool`
- **Purpose**: Check if edit (file replacement) is globally enabled (from `general.disable_edit` setting).

##### `public function isDeleteEnabledGlobally(): bool`
- **Purpose**: Check if delete is globally enabled.

##### `public function isGetEnabledGlobally(): bool`
- **Purpose**: Check if retrieval is globally enabled.

##### `public function isAutoResizeEnabledGlobally(): bool`
- **Purpose**: Check if automatic resize is globally enabled (from `general.disable_auto_resize` setting).

##### `public function isAutoWatermarkEnabledGlobally(): bool`
- **Purpose**: Check if automatic watermark is globally enabled.

##### `public function isUserTypeAllowed(string $modelClass, string $userType): bool`
- **Purpose**: Check if the specified user type is allowed to access the model.

#### 4. Integrated Validation Methods (New in version 1.2.1)

A set of methods has been added that separate validation levels and allow fine-grained permission control:

##### `public function canGlobal(string $operation): bool`
- **Purpose**: Check if a specific operation is globally enabled (without checking model or user).
- **Parameters**:
  - `$operation`: The operation (`add`, `edit`, `delete`, `view`).
- **Returns**: `true` if the operation is enabled in global settings, otherwise `false`.

##### `public function validateGlobal(string $operation): void`
- **Purpose**: Like `canGlobal` but throws `FileUploadException` if the operation is disabled.
- **Exceptions**: `FileUploadException` with appropriate code (`ERR_UPLOAD_DISABLED_GLOBALLY`, `ERR_EDIT_DISABLED_GLOBALLY`, etc.).

##### `public function canModel(string $modelClass, string $operation): bool`
- **Purpose**: Check if a specific operation is enabled at model level (via `disabled_operations`).
- **Parameters**:
  - `$modelClass`: Model class name.
  - `$operation`: The operation.
- **Returns**: `true` if the operation is not disabled in the model, otherwise `false`.

##### `public function validateModel(string $modelClass, string $operation): void`
- **Purpose**: Like `canModel` but throws `FileUploadException` if the operation is disabled (with code `ERR_MODEL_OPERATION_DISABLED`).

##### `public function canField(string $modelClass, ?string $field, string $operation): bool`
- **Purpose**: Check if a specific operation is enabled at a specific field level (via field's `disabled_operations`).
- **Parameters**:
  - `$modelClass`: Model class name.
  - `$field`: Field name (if `null`, returns `false`).
  - `$operation`: The operation.
- **Returns**: `true` if the operation is not disabled in the field, otherwise `false`.

##### `public function validateField(string $modelClass, ?string $field, string $operation): void`
- **Purpose**: Like `canField` but throws `FileUploadException` if the operation is disabled or field not found.

##### `public function can(string $modelClass, string $operation, string $userType, $user, ?string $field = null, ?bool $isForceField = false): bool`
- **Purpose**: Check permission for an operation across the entire chain (global → module → field → user type → specific permissions).
- **Parameters**:
  - `$modelClass`: Model class name.
  - `$operation`: The operation (`add`, `edit`, `delete`, `view`).
  - `$userType`: User type (`backend`, `frontend`, `guest`).
  - `$user`: Full user object (for checking specific permissions).
  - `$field`: Field name (optional).
  - `$isForceField`: If `true`, checks the field even if `null` (used in certain contexts).
- **Returns**: `true` if all levels passed, otherwise `false`.

##### `public function validate(string $modelClass, string $operation, string $userType, $user, ?string $field = null, ?bool $isForceField = false): void`
- **Purpose**: Like `can` but throws `FileUploadException` at the first failure, simplifying code in the service layer.
- **Exceptions**: Throws `FileUploadException` with appropriate code depending on the failure level.

##### `public function isModelOperationEnabled(string $modelClass, string $operation): bool`
- **Purpose**: Check if the operation is enabled at model level (helper method used by `canModel`).
- **Returns**: `true` if the operation is not disabled in model's `disabled_operations`, otherwise `false`.

##### `public function isFieldOperationEnabled(string $modelClass, ?string $field, string $operation): bool`
- **Purpose**: Check if the operation is enabled at field level (helper method used by `canField`).
- **Returns**: `true` if the operation is not disabled in field's `disabled_operations`, otherwise `false`.

#### 5. Getting Field Constraints and Advanced Processing Options

##### `public function getFieldConstraints(string $modelClass, string $field): ?array`
- **Purpose**: Extract field constraints (max size, allowed types, etc.) in a format suitable for validation.
- **Returns**: Array containing `max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`.

##### `public function getProcessingOptions(string $modelClass, string $field): ?array`
- **Purpose**: Get advanced processing options for a specific field (storage, resize, watermark).
- **Returns**: Array containing `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.

#### 6. Cache Management

##### `public function clearModelCache(string $modelClass): void`
- **Purpose**: Clear all cache keys for a specific model (e.g., `field_config` and `constraints`).

##### `public function clearFieldCache(string $modelClass, string $fieldName): void`
- **Purpose**: Clear the cache for a specific field in a model (used internally in `updateFieldConfig`).

##### `public function setCacheEnabled(bool $enabled): void`
- **Purpose**: Enable/disable caching.

##### `public function setCacheTtl(int $ttl): void`
- **Purpose**: Set cache TTL.

##### `public function geCacheEnabled(): bool`
- **Purpose**: Return cache enabled status (note typo in method name, but kept for compatibility).

#### 7. Updating a Registered Model's Settings

##### `public function updateModelConfig(string $modelClass, array $config): self`
- **Purpose**: Update settings of a previously registered model (merge with current settings). Now normalizes the entire model using `normalizeModelDefinition` after merging.

##### `public function updateFieldConfig(string $modelClass, string $fieldName, array $newConfig): self`
- **Purpose**: Update settings for a specific field without needing to re-register the entire model.
- **Parameters**:
  - `$modelClass`: Full class name of the model.
  - `$fieldName`: Name of the field to update.
  - `$newConfig`: New settings (merged with current settings).
- **Behavior**:
  1. Checks existence of model and field.
  2. Merges new settings with existing using `array_replace_recursive`.
  3. Applies normalization via `normalizeFieldDefinition`.
  4. Updates the raw definition (`rawDefinitions`).
  5. Resets the built model (`builtModels`).
  6. Clears cache for this specific field via `clearFieldCache`.
- **Returns**: Current instance for method chaining.
- **Exceptions**: Throws `ApplicationException` if model or field is not registered.

---

### Events

| Event Name | Description |
|------------|-------------|
| `nano.api.fileupload.registerModels` | Fired when loading registered models; the `FileUploadRegistry` object is passed so developers can add their models by listening to this event. |
| `nano.api.fileupload.modelRegistered` | Fired after a model is successfully registered, passing `$modelClass` and `$config`. |

---

### Comprehensive Practical Examples

#### Example 1: Register a model from within an add-on (with advanced settings and `disabled_operations`)

```php
// In Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'disabled_operations' => ['delete'], // disable deletion at product level
            'permissions' => [
                'add'    => 'nano.shop.product.upload',
                'edit'   => 'nano.shop.product.edit',
                'delete' => 'nano.shop.product.delete',
            ],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'disabled_operations' => ['edit'], // disable image replacement only
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
                ],
            ],
        ],
    ];
}
```

#### Example 2: Register a model via event

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

#### Example 3: Check permission using `validate` (instead of `can`)

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();
$userType = $userManager->getUserType();

try {
    $registry->validate('Nano\Shop\Models\Product', 'edit', $userType, $user, 'image');
    // Allowed to replace image
} catch (FileUploadException $e) {
    // Not allowed - error message in $e->getMessage()
}
```

#### Example 4: Check if edit operation is globally enabled

```php
if ($registry->canGlobal('edit')) {
    // Edit operation is globally enabled
} else {
    // Disabled via disable_edit = true
}
```

#### Example 5: Check if upload operation is enabled at model level

```php
if ($registry->canModel('Nano\Shop\Models\Product', 'add')) {
    // Upload is enabled at product level (not disabled in disabled_operations)
}
```

#### Example 6: Update a specific field's settings (change max file size)

```php
$registry->updateFieldConfig('Nano\Shop\Models\Product', 'image', [
    'max_filesize' => 2048, // increase limit to 2 MB
]);
```

#### Example 7: Temporarily disable delete operation at model level without changing permissions

```php
$registry->updateModelConfig('Nano\Shop\Models\Product', [
    'disabled_operations' => ['delete'],
]);
```

#### Example 8: Get constraints for a specific field to display in front-end

```php
$constraints = $registry->getFieldConstraints('Nano\Shop\Models\Product', 'image');
// Can pass $constraints['max_filesize'] and $constraints['allowed_types'] to JavaScript
```

#### Example 9: Check user type before displaying upload UI

```php
$userType = $userManager->getUserType();
if ($registry->isUserTypeAllowed('Nano\Shop\Models\Product', $userType)) {
    // Show upload UI
} else {
    // Hide UI
}
```

#### Example 10: Use `can` method with field only (no additional permissions)

```php
// If no specific permissions defined on model or field, returns true
if ($registry->can('Nano\Shop\Models\Product', 'add', $userType, $user, 'image')) {
    // Allowed
}
```

---

### Interaction with Other Classes

- **With `FileUploadService`**: `FileUploadService` depends on `FileUploadRegistry` to get model settings (`getFieldConfig`, `getProcessingOptions`) and check permissions (`can` and `validate`).
- **With `FileUploadUserManager`**: `FileUploadRegistry` uses the `FileUploadUserManager::instance()` object for actual permission checking via `checkPermission()`.
- **With `Base64`**: No direct interaction, but `FileUploadService` uses `Base64` for uploading.

---

### Best Practices

1. **Register models at application startup**: Register all models once during add-on initialization (in `boot()` or `register()`).
2. **Use the `nano.api.fileupload.registerModels` event**: To standardize model registration from different add-ons.
3. **Specify allowed user types precisely**: Do not allow all types unless necessary.
4. **Define permissions for each operation**: Use appropriate permissions (e.g., `add`, `edit`, `delete`, `view`) to restrict access.
5. **Use `disabled_operations` to temporarily disable operations**: Better than modifying permissions or removing registration.
6. **Use field constraints (`getFieldConstraints`)**: For pre-validation of file size and type before upload.
7. **Use advanced processing options (`getProcessingOptions`)** to apply automatic resizing and watermarking.
8. **Enable caching in production**: For better performance (default settings have caching enabled).
9. **Use global settings (`disable_upload`, `disable_edit`, `disable_auto_resize`)** as emergency measures to temporarily disable operations.
10. **Use `updateFieldConfig` to modify a single field** instead of re-registering the entire model.
11. **Use `validate` instead of `can` in the service layer**: Because it throws standardized exceptions and reduces repetitive code.

---

### Summary

`FileUploadRegistry` is the heart of the file upload system in NanoSoft applications. It aggregates all system knowledge about models containing file upload fields, and provides a clean and efficient interface to access this information and check permissions. Thanks to its support for different user types (`backend`, `frontend`), granular permissions per operation (including `edit`), global settings to control API and automatic transformations, caching, flexible update methods (`updateModelConfig`, `updateFieldConfig`), and integrated validation methods (`canGlobal`, `validate`, `canModel`, `canField`), developers can build powerful and secure file upload systems without worrying about security or performance. This class integrates closely with `FileUploadService` and `FileUploadUserManager` to form a complete layer managing all aspects of file uploads in the application.

---

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advanced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

---