# FileUploadRegistry Class Documentation

**Overview of the `Nano.FileUpload` package and the file upload management module**

This group of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, and others) is part of the **`Nano.FileUpload` package**, an integrated software package provided by **NanoSoft**. It is specifically designed for **NanoSoft applications** to unify and manage file upload operations via APIs in a secure and flexible manner.

All classes of this module reside within the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models (modules) that contain file upload fields, checking user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before saving the record, and handling errors uniformly. Thanks to this design, developers can easily build scalable file upload systems that meet the needs of advanced applications without compromising application security or performance.

---

### Introduction

The `FileUploadRegistry` class is the **central registry** for the file upload system in NanoSoft applications. This class manages all registered models (modules) that contain file upload fields and provides a unified interface for querying the settings of these models and fields, and for checking user permissions and types before executing operations.

Simply put, `FileUploadRegistry` is the **place where all model and field definitions live**, and it is the means by which other system components (such as `FileUploadService`) access settings and permission information.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$rawDefinitions` | `array` | Raw model definitions (from plugins and callbacks). |
| `$builtModels` | `array` | Built model objects after construction (used for caching). |
| `$loaded` | `bool` | Whether definitions have been loaded from plugins. |
| `$cacheEnabled` | `bool` | Enable/disable caching of query results. |
| `$cacheTtl` | `int` | Cache time-to-live in seconds. |

---

### Key Methods

#### 1. Registering Models

##### `public function registerModel(string $modelClass, array $config = []): self`
- **Purpose:** Register a single model with its custom settings (simplified interface, calls `registerDefinition`).
- **Parameters:**
  - `$modelClass`: Fully qualified class name of the model (e.g., `Nano\Shop\Models\Product`).
  - `$config`: Array of model settings (see structure below).
- **Return:** The current instance for method chaining.

##### `public function registerDefinition(string $modelClass, array $definition): void`
- **Purpose:** Register a raw model definition (used internally and can be used directly).
- **Parameters:**
  - `$modelClass`: Fully qualified class name of the model.
  - `$definition`: Model definition (same structure as `$config` in `registerModel`).
- **Behavior:** Normalizes the model definition (`normalizeModelDefinition`), stores it in `$rawDefinitions`, and clears the cache.

##### `public function registerDefinitions(array $definitions): void`
- **Purpose:** Register multiple definitions at once.

##### `public static function registerCallback(callable $callback): void`
- **Purpose:** Register a callback function to be executed when loading definitions (useful for plugins that do not use `registerFileUploadFields`).

**Settings structure (`$config`):**
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
    'fields' => [
        'image' => [
            'type' => 'image', // image, audio, video, file, multiple
            'label' => 'Main image',
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
                'delete' => 'nano.shop.delete_image',
            ],
            // ⬇⬇⬇ Advanced settings (versions 1.0.6+) ⬇⬇⬇
            'storage_disk'      => null,      // custom storage disk (e.g., 's3')
            'auto_resize'       => false,     // enable automatic resizing
            'resize_options'    => [],        // resize options (width, height, mode)
            'auto_watermark'    => false,     // enable automatic watermarking
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

#### 2. Retrieving Registered Models

##### `public function getRegisteredModels(): array`
- **Purpose:** Return all registered models (loading definitions first).

##### `public function isModelRegistered(string $modelClass): bool`
- **Purpose:** Check whether a specific model is registered (automatically loads definitions).

##### `public function getModelConfig(string $modelClass): ?array`
- **Purpose:** Get the full configuration of a model.

##### `public function getFieldConfig(string $modelClass, string $field): ?array`
- **Purpose:** Get the configuration of a specific field (supports caching).

#### 3. Checking Permissions and Global Settings

##### `public function isUploadEnabledGlobally(): bool`
- **Purpose:** Check whether uploading is globally enabled (from `general.disable_upload` setting).

##### `public function isDeleteEnabledGlobally(): bool`
- **Purpose:** Check whether deletion is globally enabled.

##### `public function isGetEnabledGlobally(): bool`
- **Purpose:** Check whether retrieval is globally enabled.

##### `public function isAutoResizeEnabledGlobally(): bool`
- **Purpose:** Check whether automatic resizing is globally enabled (from `general.disable_auto_resize` setting).

##### `public function isAutoWatermarkEnabledGlobally(): bool`
- **Purpose:** Check whether automatic watermarking is globally enabled.

##### `public function isUserTypeAllowed(string $modelClass, string $userType): bool`
- **Purpose:** Check whether the specified user type is allowed to access the model.

##### `public function can(string $modelClass, string $operation, string $userType, $user, ?string $field = null): bool`
- **Purpose:** Check whether the user has the required permission for a specific operation (add, edit, delete, view) on the model or a specific field.
- **Behavior:** First checks global settings (e.g., `disable_upload`), then model existence, then user type, then the permissions defined for the model or field, and finally calls `FileUploadUserManager::checkPermission`.

#### 4. Retrieving Field Constraints and Advanced Processing Options

##### `public function getFieldConstraints(string $modelClass, string $field): ?array`
- **Purpose:** Extract field constraints (max size, allowed types, etc.) in a format suitable for validation.
- **Return:** An array containing `max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`.

##### `public function getProcessingOptions(string $modelClass, string $field): ?array`
- **Purpose:** Get advanced processing options for a specific field (storage, resizing, watermarking).
- **Return:** An array containing `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.

#### 5. Cache Management

##### `public function clearModelCache(string $modelClass): void`
- **Purpose:** Clear all cache keys for a specific model (e.g., `field_config` and `constraints`).

##### `public function setCacheEnabled(bool $enabled): void`
- **Purpose:** Enable/disable caching.

##### `public function setCacheTtl(int $ttl): void`
- **Purpose:** Set the cache time-to-live.

#### 6. Updating a Registered Model's Configuration

##### `public function updateModelConfig(string $modelClass, array $config): self`
- **Purpose:** Update the configuration of a previously registered model (merges with existing settings).

---

### Events

| Event Name | Description |
|------------|-------------|
| `nano.api.fileupload.registerModels` | Dispatched when model definitions need to be loaded; the `FileUploadRegistry` object is passed so that developers can add their models by listening to this event. |
| `nano.api.fileupload.modelRegistered` | Dispatched after a model is successfully registered, passing `$modelClass` and `$config`. |

---

### Comprehensive Practical Examples

#### Example 1: Register a Model from Within a Plugin (with Advanced Settings)

```php
// In Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'permissions' => [
                'add'    => 'nano.shop.product.upload',
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

#### Example 3: Check Permission Before Uploading a File (Considering Global Settings)

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
    // Not allowed (could be due to global settings or permissions)
}
```

#### Example 4: Retrieve Constraints for a Specific Field

```php
$constraints = $registry->getFieldConstraints('Nano\Shop\Models\Product', 'image');
// $constraints['max_filesize'] => 1024
// $constraints['allowed_types'] => 'jpg,jpeg,png'
```

#### Example 5: Retrieve Advanced Processing Options

```php
$procOptions = $registry->getProcessingOptions('Nano\Shop\Models\Product', 'image');
// $procOptions['auto_resize'] => true
// $procOptions['storage_disk'] => 's3'
```

#### Example 6: Update a Registered Model's Configuration (e.g., Change Add Permission)

```php
$registry->updateModelConfig('Nano\Shop\Models\Product', [
    'permissions' => [
        'add' => 'nano.shop.product.upload_v2',
    ],
]);
```

#### Example 7: Check User Type Before Displaying Upload Interface (Considering Global Settings)

```php
$userType = $userManager->getUserType();
if ($registry->isUserTypeAllowed('Nano\Shop\Models\Product', $userType)) {
    // Display file upload interface
} else {
    // Hide the interface
}
```

#### Example 8: Globally Disable Automatic Resizing via Environment Variable

```ini
NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
```

Then in code:
```php
if ($registry->isAutoResizeEnabledGlobally()) {
    // Automatic resizing is enabled
} else {
    // Disabled
}
```

---

### Interaction with Other Classes

- **With `FileUploadService`**: `FileUploadService` relies on `FileUploadRegistry` to obtain model settings (`getFieldConfig`, `getProcessingOptions`) and check permissions (`can`).
- **With `FileUploadUserManager`**: `FileUploadRegistry` uses the `FileUploadUserManager::instance()` object to actually check permissions via `checkPermission()`.
- **With `Base64` (from `Nano.Api`)**: No direct interaction, but `FileUploadService` uses `Base64` for uploading.

---

### Best Practices

1. **Register models at application startup**: Register all models once when initializing the plugin (in `boot()` or `register()`).
2. **Use the `nano.api.fileupload.registerModels` event**: to unify the way models are registered from different plugins.
3. **Specify allowed user types precisely**: do not allow all types unless necessary.
4. **Define permissions for each operation**: use appropriate permissions (e.g., `add`, `delete`) to restrict access.
5. **Use field constraints (`getFieldConstraints`)**: for pre-upload validation of file size and type.
6. **Use advanced processing options (`getProcessingOptions`)** to apply automatic resizing and watermarking.
7. **Enable caching in production environment**: for better performance (default settings are enabled).
8. **Use global settings (`disable_upload`, `disable_auto_resize`)** as an emergency measure to temporarily disable operations.

---

### Conclusion

`FileUploadRegistry` is the heart of the file upload system in NanoSoft applications. It aggregates all system knowledge about models that contain file upload fields and provides a clean and efficient programming interface to access this information and check permissions. Thanks to its support for different user types (`backend`, `frontend`), granular permissions for each operation, global settings for API control and automatic transformations, and caching, developers can build powerful and secure file upload systems without worrying about security or performance. This class integrates closely with `FileUploadService` and `FileUploadUserManager` to form an integrated layer that manages all aspects of file uploads in the application.

## Additional Documentation

- [General Add-on Documentation](./Docs-FileUpload.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class.md)
- [API Documentation](./Docs-API-Documentation.md)
