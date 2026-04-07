## `FileUploadService` Class Documentation

**Overview of the `Nano.FileUpload` Package and File Upload Management Module**

This set of classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`, etc.) is part of the **`Nano.FileUpload`** package, an integrated software package provided by **NanoSoft**, designed specifically for **NanoSoft** applications to unify and manage file upload operations via APIs in a secure and flexible way.

All classes of this module reside under the following namespace:

```
Nano\FileUpload\Classes
```

This module provides advanced tools for registering models that contain file upload fields, verifying user permissions (including different user types: `backend` and `frontend`), managing temporarily uploaded files before record saving, and handling errors uniformly. With this design, developers can easily build scalable file upload systems that meet advanced application requirements without compromising security or performance.

---

### Introduction

The `FileUploadService` class is the **service layer** responsible for executing file upload, delete, and retrieval operations in NanoSoft applications. This class relies on `FileUploadRegistry` to access model settings and check permissions, and on `FileUploadUserManager` to manage the current user and verify permissions.

In short, `FileUploadService` is the **engine that performs the actual upload operations**, ensuring that each operation complies with the registered settings and defined permissions, while supporting temporary uploads (via temporary session keys) to link files with unsaved models.

---

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$registry` | `FileUploadRegistry` | Reference to the registered models registry. |
| `$userManager` | `FileUploadUserManager` | Reference to the user and permission manager. |

---

### Important Methods

#### 1. Generate a Temporary Session Key

##### `public function generateTempSessionKey(string $modelClass, string $field, $userId = null): string`
- **Purpose**: Generate a unique temporary session key to link files before model saving.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$field`: Field name.
  - `$userId`: (optional) User ID; if not provided, uses the current user.
- **Behavior**:
  - Gets the current user ID or uses `guest`.
  - Creates a key with the format `tmp_{md5(modelClass+field+userId+time+rand)}`.
- **Returns**: The temporary session key.
- **Example**:
  ```php
  $tempKey = $service->generateTempSessionKey('Nano\Shop\Models\Product', 'image');
  ```

#### 2. Upload a Single File

##### `public function upload(string $modelClass, string $field, $fileData, array $options = []): array`
- **Purpose**: Upload a single file for a specific field.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$field`: Field name.
  - `$fileData`: File data (an `UploadedFile` object or a base64 string).
  - `$options`: Additional options array:
    - `model`: The model object (if it exists and is saved).
    - `temp_session_key`: A temporary session key (to bypass auto‑generation).
    - `...`: Any additional options passed to `Base64::onUpload`.
- **Behavior**:
  - Checks that the model and field are registered in `FileUploadRegistry`.
  - Checks `add` permission for the current user.
  - Determines whether the model exists and is saved (for direct linking) or needs a temporary session key.
  - Calls `Base64::onUpload` to perform the actual upload.
  - Adds `temp_session_key` to the result if used.
  - Returns an array with the same structure as `Base64::onUpload` (with `status`, `data`, `message`, ...).
- **Returns**: Array containing the result of the operation.
- **Example**:
  ```php
  $result = $service->upload(
      'Nano\Shop\Models\Product',
      'image',
      $uploadedFile,
      ['temp_session_key' => $tempKey]
  );
  if ($result['status']) {
      echo "File uploaded successfully. ID: " . $result['data']['id'];
  }
  ```

#### 3. Upload Multiple Files (for a Multiple Field)

##### `public function uploadMultiple(string $modelClass, string $field, array $filesData, array $options = []): array`
- **Purpose**: Upload several files for a multiple field in a single request.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$field`: Field name.
  - `$filesData`: Array of file data (each element as in `upload`).
  - `$options`: Same options as `upload`.
- **Behavior**:
  - Iterates over `$filesData` and calls `upload()` for each file.
  - Collects results and counts successes.
  - Returns an array containing `data` (results per file) and `process_data` with statistics.
- **Returns**: Array containing `code`, `status`, `message`, `data`, `process_data`.
- **Example**:
  ```php
  $files = [ $file1, $file2, $file3 ];
  $result = $service->uploadMultiple('Product', 'gallery', $files, ['temp_session_key' => $tempKey]);
  echo "Successfully uploaded " . $result['process_data']['success_count'] . " of " . $result['process_data']['total'];
  ```

#### 4. Delete a File

##### `public function deleteFile(int $fileId, ?string $modelClass = null, ?string $field = null, $user = null): array`
- **Purpose**: Delete a specific file.
- **Parameters**:
  - `$fileId`: File ID.
  - `$modelClass`: (optional) Model class name for permission check.
  - `$field`: (optional) Field name for permission check.
  - `$user`: (optional) User object; if not provided, uses the current user.
- **Behavior**:
  - Finds the file using `File::find()`.
  - If `$modelClass` and `$field` are provided, checks `delete` permission for the current user.
  - Deletes the file.
- **Returns**: Array containing `code`, `status`, `message`, `error`.
- **Example**:
  ```php
  $result = $service->deleteFile(123, 'Nano\Shop\Models\Product', 'image');
  if ($result['status']) {
      echo "File deleted successfully";
  }
  ```

#### 5. Retrieve Files

##### `public function getFiles(string $modelClass, string $field, ?int $modelId = null, array $options = []): array`
- **Purpose**: Retrieve files associated with a specific model and field.
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$field`: Field name.
  - `$modelId`: (optional) ID of the saved model.
  - `$options`: Additional options:
    - `temp_session_key`: Temporary session key to search for unassociated files.
    - `with_thumbs`: `bool` to include thumbnails.
    - `thumb_sizes`: Array of thumbnail sizes (e.g., `['thumb' => [100,100]]`).
- **Behavior**:
  - Checks that the field is registered and that the user has `view` permission.
  - Builds a query on `File` filtered by `field` and either (`attachment_id` + `attachment_type` if `modelId` exists) or `session_key` if `temp_session_key` exists.
  - Returns files with basic data (id, title, description, path, size, content_type) and thumbnails if requested.
- **Returns**: Array containing `code`, `status`, `message`, `data`.
- **Example**:
  ```php
  $files = $service->getFiles('Nano\Shop\Models\Product', 'image', 456);
  foreach ($files['data'] as $file) {
      echo "<img src='{$file['path']}'>";
  }
  ```

#### 6. Link Temporary Files to a Saved Model

##### `public function attachTempFiles(Model $model, string $field, string $tempSessionKey): bool`
- **Purpose**: Transfer files uploaded with a temporary session key to a saved model.
- **Parameters**:
  - `$model`: The saved model object.
  - `$field`: Field name.
  - `$tempSessionKey`: The temporary session key used during upload.
- **Behavior**:
  - Searches for files with `session_key = $tempSessionKey` and `field = $field`.
  - For each file, updates `attachment_id` and `attachment_type` to link it to the model and clears the `session_key`.
  - If the model has a relationship for that field, uses `add()`; otherwise sets the field directly.
- **Returns**: `true` if files were linked, otherwise `false`.
- **Example**:
  ```php
  $product = new Product();
  $product->name = 'New product';
  $product->save();

  $service->attachTempFiles($product, 'image', $tempKey);
  ```

#### 7. Validate a File (Size, Type)

##### `public function validateFile(string $modelClass, string $field, $fileData): bool`
- **Purpose**: Check that a file meets the field constraints (max size, allowed types).
- **Parameters**:
  - `$modelClass`: Fully qualified model class name.
  - `$field`: Field name.
  - `$fileData`: File data (an `UploadedFile` object or a base64 string).
- **Behavior**:
  - Calls `getFieldConstraints()` from `FileUploadRegistry`.
  - Compares file size with `max_filesize`.
  - Compares file extension with `allowed_types`.
- **Returns**: `true` if the file is valid, otherwise throws `ApplicationException`.
- **Example**:
  ```php
  try {
      $service->validateFile('Product', 'image', $uploadedFile);
      // File is valid
  } catch (ApplicationException $e) {
      echo "Invalid file: " . $e->getMessage();
  }
  ```

---

### Events

| Event Name | Description |
|------------|-------------|
| `nano.api.fileupload.beforeUpload` | (Optional, not yet implemented) Fired before upload; can be used to modify data or log. |
| `nano.api.fileupload.afterUpload` | (Optional, not yet implemented) Fired after successful upload; for additional updates. |

---

### Comprehensive Examples

#### Example 1: Upload a Main Image for a New Product Using a Temporary Session Key

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

$service = FileUploadService::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';

// 1. Generate a temporary session key
$tempKey = $service->generateTempSessionKey($modelClass, $field, $userManager->getId());

// 2. Upload the image (assuming $uploadedFile is an UploadedFile object)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey,
    'title' => 'Product image', // optional
    'description' => 'Image description',
]);

if (!$uploadResult['status']) {
    die("Upload failed: " . $uploadResult['error']);
}

// 3. Create and save the product
$product = new Product();
$product->name = 'New product';
$product->save();

// 4. Link the image to the product
$service->attachTempFiles($product, $field, $tempKey);

echo "Image uploaded and linked to product ID {$product->id}";
```

#### Example 2: Upload Multiple Images for an Existing Product Gallery

```php
$product = Product::find(10); // existing product

// Upload a set of images
$filesData = [
    $file1,
    $file2,
    $file3,
];

$result = $service->uploadMultiple(Product::class, 'gallery', $filesData, [
    'model' => $product, // direct linking because the product exists
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " images";
} else {
    echo "Some images failed to upload: " . $result['error'];
}
```

#### Example 3: Retrieve All Images of a Product Gallery with Thumbnails

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
    ],
]);

foreach ($result['data'] as $image) {
    echo "<img src='{$image['small']}' alt='{$image['title']}'>";
}
```

#### Example 4: Delete a Main Image After Checking Permission

```php
$result = $service->deleteFile(123, Product::class, 'image');

if ($result['status']) {
    echo "Image deleted successfully";
} else {
    echo "Error: " . $result['message'];
}
```

#### Example 5: Validate a File Before Upload

```php
try {
    $service->validateFile(Product::class, 'image', $uploadedFile);
    // File is valid
} catch (ApplicationException $e) {
    echo "Invalid file: " . $e->getMessage();
    return;
}

$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
```

#### Example 6: Using `validateFile` with Base64 Data

```php
$base64String = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...';
$fileData = $base64String;

try {
    $service->validateFile(Product::class, 'image', $fileData);
    echo "File is valid";
} catch (ApplicationException $e) {
    echo "Invalid file: " . $e->getMessage();
}
```

#### Example 7: Handling Upload Errors

```php
$result = $service->upload(Product::class, 'image', $fileData, []);

if (!$result['status']) {
    switch ($result['code']) {
        case 400:
            echo "Bad request: " . $result['error'];
            break;
        case 401:
            echo "Unauthorized: " . $result['error'];
            break;
        case 403:
            echo "Permission denied: " . $result['error'];
            break;
        default:
            echo "Unknown error: " . $result['error'];
    }
}
```

#### Example 8: Integrating `FileUploadService` with an API Controller

In an API controller:

```php
public function uploadImage(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $file = $request->file('file');

    $service = FileUploadService::instance();

    // Check permissions and constraints
    $result = $service->upload($modelClass, $field, $file, [
        'temp_session_key' => $request->input('temp_session_key')
    ]);

    if ($result['status']) {
        return response()->json([
            'success' => true,
            'data' => $result['data'],
            'temp_session_key' => $result['temp_session_key'] ?? null,
        ]);
    } else {
        return response()->json([
            'success' => false,
            'error' => $result['error'],
            'code' => $result['code'],
        ], $result['code']);
    }
}
```

---

### Interaction with Other Classes

- **With `FileUploadRegistry`**: Uses `$registry` to get field settings (`getFieldConfig`, `getFieldConstraints`) and check permissions (`can`).
- **With `FileUploadUserManager`**: Uses `$userManager` to get the current user and type (`getUser`, `getUserType`).
- **With `Base64` (from `Nano.Api`)**: Calls `Base64::onUpload`, `Base64::is_base64`, `Base64::base64ToUploadedFile`, `Base64::getBase64ImageSize`, `Base64::getDataInBase64`.

---

### Best Practices

1. **Always use a temporary session key for new models**: to avoid losing files if the model is not saved yet.
2. **Validate the file before upload using `validateFile`**: to provide clear error messages.
3. **Log transaction IDs**: to track upload operations and file status.
4. **Use `uploadMultiple` for multiple fields**: instead of calling `upload` in a loop, because `uploadMultiple` aggregates errors and provides statistics.
5. **Don't rely only on `temp_session_key`**: after saving the model, use `attachTempFiles` to transfer ownership.
6. **Handle errors by code**: 400 (bad request), 401 (unauthorized), 403 (forbidden), 404 (not found), 500 (internal error).
7. **Enable `app.debug` in development**: to get full error details.
8. **Use `with_thumbs` only when needed**: to reduce response size and improve performance.
9. **Store temporary session keys in session or a temporary database**: to avoid losing them.
10. **Test permissions thoroughly**: ensure unauthorized users cannot upload or delete files.

---

### Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `ApplicationException: Field ... not registered for model ...` | Field not registered in `FileUploadRegistry`. | Add the field to the registration. |
| `You do not have permission to upload files for this field` | User lacks the `add` permission for the field. | Check `permissions` settings in Registry or user permissions. |
| `Invalid file data` | Provided data is neither an `UploadedFile` nor a valid base64 string. | Ensure correct data format. |
| `File not found` (when deleting) | File ID does not exist. | Verify the ID. |
| `File size exceeds allowed limit` | File size exceeds `max_filesize`. | Compress the file or increase `max_filesize`. |
| `File type not allowed` | File extension not in `allowed_types`. | Use an allowed type or adjust `allowed_types`. |

---

### Integration with Custom Permission Systems

In NanoSoft applications, you can customize the permission checker for frontend users via `FileUploadUserManager`:

```php
FileUploadUserManager::instance()->setPermissionChecker(function ($user, $operation, $permissions) {
    // Custom logic: e.g., check a custom permissions table
    if ($user->hasRole('editor')) {
        return true;
    }
    return $user->hasPermission($permissions);
});
```

---

### Conclusion

The `FileUploadService` class provides a powerful and unified interface for managing file uploads in NanoSoft applications. Through the examples above, developers can implement any file upload scenario: from simple uploads with direct linking to temporary uploads for unsaved models, permission and constraint checks, retrieving files with thumbnails, and secure deletion. We recommend using this service as the single layer for handling files throughout the application to ensure consistency and security.

For details on the response structure when using the `FileUploadController`, refer to [API Documentation](./Docs-API-Documentation-en.md).

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

