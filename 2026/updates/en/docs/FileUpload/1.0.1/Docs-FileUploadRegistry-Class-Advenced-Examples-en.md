# Advanced and Practical Examples for the `FileUploadRegistry` Class

**Namespace:** `Nano\FileUpload\Classes`  
**Purpose:** To provide comprehensive application examples for using `FileUploadRegistry` in various scenarios, from basic model registration to advanced permission checks, event handling, constraint queries, and integration with `FileUploadService`, including expected inputs/outputs and best practices.

---

## Introduction

This document presents a set of practical examples covering various use cases of the `FileUploadRegistry` class, which acts as the **central registry** for managing model definitions and file upload fields in NanoSoft applications. Through these examples, you will learn how to:

- Register models and their fields with advanced settings.
- Specify allowed user types (backend/frontend).
- Set granular permissions per operation (add, edit, delete, view) at the model or field level.
- Use verification methods (`can`, `isUserTypeAllowed`, `getFieldConstraints`) to ensure security.
- Listen to events (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) to extend the system.
- Integrate `FileUploadRegistry` with `FileUploadService` for file uploads.
- Handle errors and apply best practices.

> **Note:** The examples here interact directly with `FileUploadRegistry`. In real applications, it is often used through `FileUploadService`, but understanding the underlying settings helps in customizing the system accurately.

---

## Prerequisites

- The `Nano.FileUpload` plugin installed in your application.
- The plugin depends on `Nano.Api` which provides `Base64` and `ApiController`.
- Basic knowledge of models and relationships in Laravel / NanoSoft applications.
- (Optional) Use of `FileUploadUserManager` to verify user permissions.

### Setting Up the Class in the Application

```php
use Nano\FileUpload\Classes\FileUploadRegistry;

$registry = FileUploadRegistry::instance();
```

Because the class follows the Singleton pattern, it is accessed via `instance()`.

---

## 1. Registering a Model with Simple Settings

**Scenario:** Register a `Product` model with a single `image` field (main image) with default settings.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'image' => [
            'type' => 'image',
            'label' => 'Main image',
            'max_filesize' => 1024, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
        ],
    ],
]);
```

**Output:** The model is added to `$registeredModels`. You can verify with:

```php
if ($registry->isModelRegistered(\Nano\Shop\Models\Product::class)) {
    echo "Model registered successfully";
}
```

**Notes:**  
- `type` can be `image`, `file`, or `multiple` (for multiple fields).  
- `max_filesize` is in kilobytes (KB).  
- `allowed_types` is a comma-separated list of extensions or `*` to allow all types.  
- If `label` is not specified, the field name will be used.

---

## 2. Registering a Model with Advanced Settings (User Types and Permissions)

**Scenario:** Register a `User` model with an `avatar` field, allowing only `frontend` users, and specific permissions for each operation.

```php
$registry->registerModel(\RainLab\User\Models\User::class, [
    'allowed_user_types' => ['frontend'], // only frontend users
    'permissions' => [
        'add'    => null, // no extra permission needed (everyone allowed)
        'delete' => 'user.delete_avatar',
        'view'   => null,
    ],
    'fields' => [
        'avatar' => [
            'type' => 'image',
            'max_filesize' => 2048,
            'allowed_types' => 'jpg,jpeg,png',
            'permissions' => [
                'add' => 'user.upload_avatar', // override the general add permission
            ],
        ],
    ],
]);
```

**Checking Permission:**

```php
$user = \Auth::getUser(); // frontend user
$userType = $registry->getUserTypeFromModel($user); // should return 'frontend'

if ($registry->can(\RainLab\User\Models\User::class, 'add', $userType, $user, 'avatar')) {
    echo "Allowed to upload image";
} else {
    echo "Not allowed";
}
```

**Output:** If the user has the `user.upload_avatar` permission (in the frontend permission system), returns `true`. Otherwise `false`.

**Notes:**  
- Field permissions override model permissions.  
- If a permission is `null`, access is automatically granted after checking the user type.

---

## 3. Registering a Multiple Field with Additional Constraints

**Scenario:** Add a `gallery` field allowing up to 10 images, each up to 1 MB, only jpg/png formats.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'gallery' => [
            'type' => 'multiple',
            'label' => 'Image gallery',
            'max_filesize' => 1024,
            'allowed_types' => 'jpg,jpeg,png',
            'max_files' => 10,
            'multiple' => true,
            'required' => false,
        ],
    ],
]);
```

**Querying Constraints:**

```php
$constraints = $registry->getFieldConstraints(\Nano\Shop\Models\Product::class, 'gallery');
// $constraints['max_files'] == 10
// $constraints['multiple'] == true
```

---

## 4. Using the `nano.api.fileupload.registerModels` Event to Register Models from Different Plugins

**Scenario:** Register multiple models through an event listener, allowing registrations to be centralized.

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
- This event is automatically fired by `FileUploadRegistry` when `getRegisteredModels()` is called.  
- It can be placed in the main plugin's `boot()` method.

---

## 5. Listening to the `nano.api.fileupload.modelRegistered` Event to Perform Additional Actions After Registration

**Scenario:** After registering a model, dynamically register an API route based on its fields.

```php
Event::listen('nano.api.fileupload.modelRegistered', function ($modelClass, $config) {
    // Could add a custom route to retrieve files for this model
    \Route::get('/api/custom/' . class_basename($modelClass) . '/files', function () use ($modelClass) {
        return $registry->getFiles($modelClass, 'image');
    });
});
```

---

## 6. Checking User Type (`isUserTypeAllowed`)

**Scenario:** Display a list of fields only if the user is of an allowed type.

```php
$userType = $userManager->getUserType(); // backend, frontend, guest
$modelClass = \Nano\Shop\Models\Product::class;

if ($registry->isUserTypeAllowed($modelClass, $userType)) {
    // Show file upload interface
    $fields = $registry->getModelConfig($modelClass)['fields'];
    foreach ($fields as $field => $config) {
        echo "Field: {$field} - Type: {$config['type']}\n";
    }
} else {
    echo "Access not allowed";
}
```

---

## 7. Using `can` to Check Delete Permission

**Scenario:** Check if the current user has permission to delete a file from a specific field.

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

**Output:** If the user has the `delete` permission (at model or field level), returns `true`.

---

## 8. Getting Field Constraints and Validating Before Upload

**Scenario:** In the upload interface, display maximum size and allowed types to the user, and validate before sending the request.

```php
$constraints = $registry->getFieldConstraints(\Nano\Shop\Models\Product::class, 'image');
if ($constraints['max_filesize']) {
    echo "Maximum file size: {$constraints['max_filesize']} KB\n";
}
if ($constraints['allowed_types'] != '*') {
    echo "Allowed types: {$constraints['allowed_types']}\n";
}
if ($constraints['required']) {
    echo "This field is required\n";
}
```

---

## 9. Integrating `FileUploadRegistry` with `FileUploadService` in a Complete Upload Process

**Scenario:** Create a new `Product` model and upload a main image using a temporary session key.

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadRegistry;

$registry = FileUploadRegistry::instance();
$service = FileUploadService::instance();

$modelClass = \Nano\Shop\Models\Product::class;
$field = 'image';

// Check if the model exists and permission is granted
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
    // Now create the model (e.g., $product = new Product(); $product->save();)
    $product = new \Nano\Shop\Models\Product();
    $product->name = 'New product';
    $product->save();

    // Link the temporary files to the saved model
    $service->attachTempFiles($product, $field, $tempKey);

    echo "Image uploaded and linked to product";
} else {
    echo "Upload failed: " . $uploadResult['error'];
}
```

**Notes:**  
- `attachTempFiles` will search for files with `session_key` = `$tempKey` and update `attachment_id` and `attachment_type` to link them to the model.  
- If the field is `multiple`, all files will be added to the relationship.

---

## 10. Handling Registration Errors (Model Not Found)

**Scenario:** Attempt to register a non‑existent model.

```php
try {
    $registry->registerModel('NonExistentModel', []);
} catch (\ApplicationException $e) {
    echo "Error: " . $e->getMessage(); // "Model NonExistentModel does not exist"
}
```

---

## 11. Updating a Registered Model's Configuration

**Scenario:** After registering the model, change the add permission for the `image` field from `null` to a specific permission.

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

Now when calling `can` for the `image` field, the new permission will be used.

---

## 12. Using Defaults to Reduce Repetition

**Scenario:** Several fields share the same settings (size 2 MB, image types, public). You can define `defaults` and then override individual fields.

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
In `gallery`, the same defaults apply plus `max_files` and `multiple`.

---

## 13. Getting Full Model or Field Configuration

**Scenario:** Print all settings for a specific model for debugging.

```php
$config = $registry->getModelConfig(\Nano\Shop\Models\Product::class);
print_r($config);
```

**Output:** An array containing `enabled`, `allowed_user_types`, `permissions`, `fields`, `defaults`.

---

## 14. Checking `view` Permission Before Displaying File List

**Scenario:** In an API controller, check if the user has permission to view files for a given model.

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

---

## 15. Testing Field Permission with a Custom Approach – Not directly applicable, but can add an example of checking `view` permission for a field.

**Scenario:** Check if the user has permission to view a specific field before displaying it in the UI.

```php
$user = \Auth::getUser();
$userType = 'frontend';
$modelClass = \RainLab\User\Models\User::class;
$field = 'avatar';

if ($registry->can($modelClass, 'view', $userType, $user, $field)) {
    echo '<img src="' . $user->avatar->getPath() . '">';
} else {
    echo 'Do not show image';
}
```

---

## 16. Best Practices

1. **Register models early**: Use the `nano.api.fileupload.registerModels` event in the main plugin's `boot()`.
2. **Use `allowed_user_types` wisely**: Do not allow all types unless necessary.
3. **Define precise permissions per operation**: At least `add` and `delete` to prevent unauthorized uploads or deletions.
4. **Use `getFieldConstraints` for pre‑validation**: Check size and types before uploading to avoid server errors.
5. **Document field settings**: Use `label` and `description` to help other developers understand.
6. **Use defaults (`defaults`)**: To reduce repetition and ease maintenance.
7. **Don't rely solely on general permissions**: Customize permissions per field when needed.
8. **Monitor events**: `modelRegistered` can be used to register API routes or perform additional setup.
9. **Test permissions thoroughly**: Ensure unauthorized users cannot upload files via the API.
10. **Use `FileUploadService` instead of directly interacting with `FileUploadRegistry`** for upload operations: to ensure all checks pass through the correct layer.

---

## 17. Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `ApplicationException: Model ... does not exist` | Attempting to register a non‑existent model. | Ensure the class name is correct and imported. |
| Field not appearing in the interface | `allowed_user_types` does not include the current user type. | Check `$userType` or add the appropriate type. |
| `add` permission not working | Permission not set in `permissions` or `permissions[add]` missing. | Define the required permission or set it to `null` to allow everyone. |
| `isModelRegistered` returns `false` | Was `getRegisteredModels()` called after registration? | Ensure registration occurs before the query. |
| Temporary files not linked | The correct `temp_session_key` was not passed to `attachTempFiles`. | Use the same key returned by `upload`. |

---

## 18. Integration with NanoSoft Permission System

In NanoSoft applications, `FileUploadUserManager` is used to check permissions. You can set a custom checker for frontend users:

```php
FileUploadUserManager::instance()->setPermissionChecker(function ($user, $operation, $permissions) {
    // Custom logic for frontend user permissions
    // e.g., check a custom permissions table
    return $user->hasCustomPermission($operation, $permissions);
});
```

---

## Conclusion

The `FileUploadRegistry` class is the backbone of file upload management in NanoSoft applications. Through the examples above, developers can register any model with flexible settings, precisely control user types and permissions, and integrate with `FileUploadService` to securely upload files. We recommend using default settings and leveraging events to extend the system, and always checking permissions at every entry point to ensure the highest security levels.

For details on the response structure when using the `FileUploadController`, refer to [API Documentation](./Docs-API-Documentation-en.md).


## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

