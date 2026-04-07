## `FileUploadRegistry` Class Documentation

**Overview of the `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload`** package, an integrated software package provided by **NanoSoft**, designed specifically for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible way.

All classes of this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, and handling errors uniformly. With this design, developers can easily build scalable file upload systems that meet advanced application requirements without compromising security or performance.

---

### Introduction

The `FileUploadRegistry` class is the **central registry** for the file upload system in NanoSoft applications. This class manages all registered models that contain file upload fields and provides a unified interface to query settings for these models and fields, and to check user permissions and types before performing operations.

Simply put, `FileUploadRegistry` is the **place where all model and upload field definitions live**, and it is the means by which other system components (like `FileUploadService`) access configuration and permission information.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$registeredModels` | `array` | Array storing registered model settings, keyed by the fully qualified model class name. |

---

### Important Methods

#### 1. Registering Models

##### `public function registerModel(string $modelClass, array $config = []): self`
- **Purpose**: Register a single model with its custom settings.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name (e.g., `Nano\Shop\Models\Product`).
  - `$config`: Array of model settings (see structure below).
- **Behavior**:
  - Checks if the class exists (`class_exists`); throws `ApplicationException` if not.
  - Merges the passed configuration with default settings (`defaultConfig`).
  - For each field in `fields`, merges field settings with model defaults.
  - Stores the final configuration in `$registeredModels` using `$modelClass` as key.
  - Fires the `nano.api.fileupload.modelRegistered` event after registration.
- **Returns**: The current instance for method chaining.

**Configuration Structure (`$config`):**

```php
[
    'enabled' => true, // enable/disable the model
    'allowed_user_types' => ['backend', 'frontend'], // allowed user types
    'permissions' => [ // general model permissions (apply to all fields unless overridden)
        'add'    => null, // can be a single permission string or an array of permissions
        'edit'   => null,
        'delete' => null,
        'view'   => null,
    ],
    'fields' => [ // upload fields
        'image' => [
            'type' => 'image', // file, image, multiple
            'label' => 'Main image',
            'max_filesize' => 2048, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'max_files' => null, // only for multiple fields
            'is_public' => true,
            'use_caption' => true,
            'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
            'allowed_user_types' => ['backend'], // inherits from model unless customized
            'permissions' => [ // field-specific permissions (override model permissions)
                'add'    => 'nano.shop.upload_image',
                'delete' => 'nano.shop.delete_image',
            ],
        ],
    ],
    'defaults' => [ // default field settings (can be overridden per field)
        'max_filesize' => 2048,
        'allowed_types' => '*',
        'is_public' => true,
        'use_caption' => true,
        'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
    ],
]
```

##### `public function registerTables(array $tables): void` – (not present in current class, but can be added as needed)

#### 2. Retrieving Registered Models

##### `public function getRegisteredModels(): array`
- **Purpose**: Return all registered models with their settings.
- **Behavior**:
  - If `$registeredModels` is empty, loads models via:
    - Event `nano.api.fileupload.registerModels` (plugins can listen to it).
    - Calling `registerFileUploadFields()` methods from all registered plugins.
  - Returns the full array.
- **Returns**: Array of model settings.

##### `public function isModelRegistered(string $modelClass): bool`
- **Purpose**: Check whether a specific model is registered.
- **Returns**: `true` if exists, otherwise `false`.

##### `public function getModelConfig(string $modelClass): ?array`
- **Purpose**: Get the configuration of a specific model.
- **Returns**: Array of settings or `null` if not registered.

##### `public function getFieldConfig(string $modelClass, string $field): ?array`
- **Purpose**: Get the configuration of a specific field belonging to a registered model.
- **Returns**: Array of field settings or `null` if not found.

#### 3. Permission Checks

##### `public function isUserTypeAllowed(string $modelClass, string $userType): bool`
- **Purpose**: Check whether the given user type is allowed to access the model.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$userType`: User type (`backend`, `frontend`, `guest`).
- **Returns**: `true` if allowed, otherwise `false`.

##### `public function can(string $modelClass, string $operation, string $userType, $user, ?string $field = null): bool`
- **Purpose**: Check whether the user has the required permission for a specific operation (`add`, `edit`, `delete`, `view`) on a model or a specific field.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$operation`: The operation (`add`, `edit`, `delete`, `view`).
  - `$userType`: User type (from `FileUploadUserManager`).
  - `$user`: The full user object (passed to `FileUploadUserManager::checkPermission`).
  - `$field`: (optional) Field name – if provided, checks field permissions first, then model permissions.
- **Behavior**:
  - First checks `isUserTypeAllowed()`.
  - Looks for required permissions from the field (if any) then from the model.
  - If no permissions are defined, access is granted automatically.
  - Uses `FileUploadUserManager::checkPermission()` for the actual permission check.
- **Returns**: `true` if permission is granted, otherwise `false`.

#### 4. Getting Field Constraints

##### `public function getFieldConstraints(string $modelClass, string $field): ?array`
- **Purpose**: Extract field constraints (max size, allowed types, etc.) in a format suitable for validation.
- **Returns**: Array containing:
```php
[
    'max_filesize' => 2048,
    'allowed_types' => 'jpg,jpeg,png',
    'multiple' => false,
    'max_files' => null,
    'required' => false,
    'is_public' => true,
    'use_caption' => true,
    'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
    'type' => 'image',
]
```

#### 5. Updating a Registered Model's Configuration

##### `public function updateModelConfig(string $modelClass, array $config): self`
- **Purpose**: Update the configuration of a previously registered model.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$config`: New configuration array (merged with existing configuration using `array_replace_recursive`).
- **Returns**: The current instance for method chaining.

---

### Events

| Event Name | Description |
|------------|-------------|
| `nano.api.fileupload.registerModels` | Fired when registered models need to be loaded; passes a `FileUploadRegistry` instance so developers can add their models by listening to this event. |
| `nano.api.fileupload.modelRegistered` | Fired after a model is successfully registered; passes `$modelClass` and `$config`. |

---

### Comprehensive Examples

#### Example 1: Registering a Model from Within a Plugin

```php
// In Plugin.php (or any class that registers the plugin)
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'], // only backend users
            'permissions' => [
                'add'    => 'nano.shop.product.upload',
                'delete' => 'nano.shop.product.delete',
            ],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'required' => false,
                    'permissions' => [
                        'add'    => 'nano.shop.product.upload_image',
                        'delete' => 'nano.shop.product.delete_image',
                    ],
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

#### Example 2: Registering a Model via an Event

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

#### Example 3: Checking Permission Before Uploading a File

```php
use Nano\FileUpload\Classes\FileUploadRegistry;
use Nano\FileUpload\Classes\FileUploadUserManager;

$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();
$userType = $userManager->getUserType();

$modelClass = 'Nano\Shop\Models\Product';
$field = 'image';

if ($registry->can($modelClass, 'add', $userType, $user, $field)) {
    // Allowed to upload the image
} else {
    // Not allowed
}
```

#### Example 4: Getting Constraints for a Specific Field

```php
$constraints = $registry->getFieldConstraints('Nano\Shop\Models\Product', 'image');
// $constraints['max_filesize'] => 1024
// $constraints['allowed_types'] => 'jpg,jpeg,png'
```

#### Example 5: Updating a Registered Model's Configuration

```php
$registry->updateModelConfig('Nano\Shop\Models\Product', [
    'permissions' => [
        'add' => 'nano.shop.product.upload_v2', // change add permission
    ],
]);
```

#### Example 6: Retrieving All Registered Model Class Names

```php
$modelClasses = array_keys($registry->getRegisteredModels());
// ['Nano\Shop\Models\Product', 'RainLab\User\Models\User', ...]
```

#### Example 7: Checking User Type Before Displaying Upload Interface

```php
$userType = $userManager->getUserType();
if ($registry->isUserTypeAllowed('Nano\Shop\Models\Product', $userType)) {
    // Show image upload interface
} else {
    // Hide the interface
}
```

---

### Interaction with Other Classes

- **With `FileUploadService`**: `FileUploadService` relies on `FileUploadRegistry` to get model settings and check permissions before executing upload and delete operations.
- **With `FileUploadUserManager`**: `FileUploadRegistry` uses `FileUploadUserManager::instance()` for actual permission checks via `checkPermission()`.
- **With `Base64` (from `Nano.Api`)**: No direct interaction, but `FileUploadService` uses `Base64` for uploads.

---

### Best Practices

1. **Register models early in the application**: Do it once when the plugin initializes (in `boot()` or `register()`).
2. **Use the `nano.api.fileupload.registerModels` event**: To unify model registration from different plugins.
3. **Specify allowed user types precisely**: Do not allow all types unless necessary.
4. **Define permissions for each operation**: Use appropriate permissions (e.g., `add`, `delete`) to restrict access.
5. **Use field constraints (`getFieldConstraints`)**: For pre‑upload validation of file size and type.
6. **Don't rely only on default settings**: Customize `max_filesize` and `allowed_types` according to each field's requirements.

---

### Conclusion

`FileUploadRegistry` is the heart of the file upload system in NanoSoft applications. It aggregates all knowledge about models that contain file upload fields and provides a clean and efficient API to access this information and check permissions. Thanks to its support for different user types (`backend`, `frontend`) and granular permissions per operation, developers can build powerful and secure file upload systems without worrying about security or performance. This class integrates closely with `FileUploadService` and `FileUploadUserManager` to form a complete layer managing all aspects of file uploads in the application.


## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [Advanced Examples for `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

