# Advanced and Practical Examples for the `FileUploadService` Class

**Namespace:** `Nano\FileUpload\Classes`  
**Purpose:** To provide comprehensive application examples for using `FileUploadService` in various scenarios, from basic file upload operations to handling temporary files, linking to models, permission and constraint checks, with expected inputs/outputs and best practices.

---

## Introduction

This document presents a set of practical examples covering various use cases of the `FileUploadService` class, which represents the **service layer** responsible for executing file upload, delete, and retrieval operations in NanoSoft applications. This class relies on `FileUploadRegistry` to access model settings and check permissions, and on `FileUploadUserManager` to manage the current user and verify permissions.

Through these examples, you will learn how to:

- Upload a single file or multiple files using `UploadedFile` or base64 data.
- Use temporary session keys (`temp_session_key`) to link files with unsaved models.
- Link temporary files to a model after saving it.
- Retrieve files associated with a model, with thumbnails.
- Delete files with permission checks.
- Validate a file (size, type) before upload.
- Handle various errors (permissions, constraints, server errors).
- Integrate the service with APIs in NanoSoft applications.

> **Note:** The examples here use `FileUploadService` directly. In real applications, it is often called via `FileUploadController` or from within plugin code.

---

## Prerequisites

- The `Nano.FileUpload` plugin installed and configured in your application.
- The plugin depends on `Nano.Api` which provides `Base64` and `ApiController`.
- Models and upload fields are registered in `FileUploadRegistry` as shown in [Advanced Examples for FileUploadRegistry](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md).
- Basic knowledge of `UploadedFile` objects and how to obtain them from requests (e.g., via `$request->file('file')`).

### Setting Up the Class in the Application

```php
use Nano\FileUpload\Classes\FileUploadService;

$service = FileUploadService::instance();
```

Because the class follows the Singleton pattern, it is accessed via `instance()`.

---

## 1. Upload a Single File Using UploadedFile (Direct Linking to an Existing Model)

**Scenario:** An existing product; we want to upload a main image and link it directly.

```php
$product = Product::find(10);
$uploadedFile = $request->file('image'); // UploadedFile object

$result = $service->upload(Product::class, 'image', $uploadedFile, [
    'model' => $product, // direct linking
    'title' => 'Product image',
    'description' => 'Image description',
]);

if ($result['status']) {
    echo "Image uploaded successfully. File ID: " . $result['data']['id'];
    echo "Path: " . $result['data']['path'];
} else {
    echo "Upload failed: " . $result['error'];
}
```

**Expected Output (on success):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'File uploaded successfully',
    'data' => [
        'id' => 123,
        'path' => '/storage/app/uploads/public/...',
        'thumb' => '/storage/app/uploads/public/...',
    ],
    'process_data' => [
        'model_class' => 'Nano\Shop\Models\Product',
        'field' => 'image',
        'temp_session_key' => null,
        'user_type' => 'backend',
        // ... additional data from Base64::onUpload
    ],
]
```

**Notes:**  
- When a saved `model` is passed (with `$product->exists` true), linking is done directly without a temporary session key.  
- You can set a custom `title` and `description` for the file.

---

## 2. Upload a Single File Using Base64 (Unsaved Model)

**Scenario:** Creating a new product; we want to upload its image before saving the product, then link it later.

```php
$base64String = 'data:image/jpeg;base64,/9j/4AAQSkZJRg...';

$modelClass = Product::class;
$field = 'image';

// Generate a temporary session key
$tempKey = $service->generateTempSessionKey($modelClass, $field);

// Upload the image
$result = $service->upload($modelClass, $field, $base64String, [
    'temp_session_key' => $tempKey,
]);

if ($result['status']) {
    // Save the product
    $product = new Product();
    $product->name = 'New product';
    $product->save();

    // Link the image to the product
    $service->attachTempFiles($product, $field, $tempKey);

    echo "Image uploaded and linked to product ID {$product->id}";
} else {
    echo "Upload failed: " . $result['error'];
}
```

**Output:**  
- `upload` returns `temp_session_key` in `$result['temp_session_key']` if used.  
- `attachTempFiles` returns `true` if files were linked, otherwise `false`.

**Notes:**  
- `generateTempSessionKey` is optional; if you don't pass a `temp_session_key`, `upload` will generate one and return it in the result.  
- The `temp_session_key` should be stored in the user's session or a temporary database to use later.

---

## 3. Upload Multiple Files for a Multiple Field (Gallery)

**Scenario:** Upload a set of images for an existing product gallery.

```php
$product = Product::find(10);
$files = $request->file('gallery'); // array of UploadedFile

$result = $service->uploadMultiple(Product::class, 'gallery', $files, [
    'model' => $product, // direct linking
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " of " . $result['process_data']['total'] . " images.";
    foreach ($result['data'] as $index => $fileResult) {
        if ($fileResult['status']) {
            echo "Image $index: ID {$fileResult['data']['id']}\n";
        } else {
            echo "Image $index failed: {$fileResult['error']}\n";
        }
    }
} else {
    echo "All images failed to upload: " . $result['error'];
}
```

**Expected Output (part of `$result`):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'Successfully uploaded 2 out of 3 files',
    'data' => [
        0 => ['status' => true, 'data' => ['id' => 101, ...]],
        1 => ['status' => true, 'data' => ['id' => 102, ...]],
        2 => ['status' => false, 'error' => 'File type not allowed'],
    ],
    'process_data' => [
        'success_count' => 2,
        'total' => 3,
    ],
]
```

---

## 4. Retrieve Files for a Specific Field with Thumbnails

**Scenario:** Display all images of a product gallery with thumbnails in various sizes.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
        'large' => [800, 600, 'auto'],
    ],
]);

if ($result['status']) {
    foreach ($result['data'] as $image) {
        echo "<img src='{$image['small']}' alt='{$image['title']}'>";
        echo "<a href='{$image['large']}'>View large</a>";
    }
} else {
    echo "Error: " . $result['error'];
}
```

**Output (example for one file):**

```php
[
    'id' => 101,
    'title' => 'Product image',
    'description' => null,
    'path' => '/storage/app/uploads/public/.../original.jpg',
    'size' => 123456,
    'content_type' => 'image/jpeg',
    'small' => '/storage/app/uploads/public/.../small.jpg',
    'medium' => '/storage/app/uploads/public/.../medium.jpg',
    'large' => '/storage/app/uploads/public/.../large.jpg',
]
```

---

## 5. Delete a File with Permission Check

**Scenario:** A user wants to delete a main product image; we check permissions first.

```php
$fileId = 123;
$result = $service->deleteFile($fileId, Product::class, 'image');

if ($result['status']) {
    echo "File deleted successfully";
} else {
    echo "Deletion failed: " . $result['message'];
}
```

**Notes:**  
- If `$modelClass` and `$field` are not provided, no permission check is performed.  
- The check uses the `delete` permission from the field or model settings.

---

## 6. Validate a File Before Upload

**Scenario:** Before uploading, check the file's size and type according to the registered field constraints.

```php
$uploadedFile = $request->file('image');

try {
    $service->validateFile(Product::class, 'image', $uploadedFile);
    // File is valid
    $result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
} catch (ApplicationException $e) {
    echo "Invalid file: " . $e->getMessage();
}
```

**Common exceptions:**  
- `File size exceeds allowed limit (2048 KB)`  
- `File type not allowed. Allowed types: jpg,jpeg,png`

---

## 7. Handling Permission Errors

**Scenario:** Attempt to upload a file without permission.

```php
$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);

if (!$result['status'] && $result['code'] == 403) {
    echo "You do not have permission to upload files for this field";
}
```

**HTTP codes used:**  
- `400`: Invalid data or field constraints violated.  
- `403`: Permission denied.  
- `404`: Model or field not registered.  
- `500`: Internal error (details appear in debug).

---

## 8. Upload a File Using Base64 from an API

**Scenario:** An API endpoint receives an image as a base64 string and uploads it.

```php
public function uploadBase64(Request $request)
{
    $base64 = $request->input('image_base64');
    $productId = $request->input('product_id');

    $service = FileUploadService::instance();

    $product = Product::find($productId);
    if (!$product) {
        return response()->json(['error' => 'Product not found'], 404);
    }

    $result = $service->upload(Product::class, 'image', $base64, ['model' => $product]);

    return response()->json($result);
}
```

---

## 9. Upload a Temporary File and Use It for Multiple Models

**Scenario:** A file is uploaded temporarily (e.g., a profile image) and then linked to more than one model (e.g., a user and an article).

```php
// Step 1: Upload the image with a temporary key
$tempKey = $service->generateTempSessionKey(User::class, 'avatar');
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['temp_session_key' => $tempKey]);

if (!$result['status']) die('Upload failed');

// Step 2: Save the user and article, then link the same image
$user = new User();
$user->name = 'Ahmed';
$user->save();

$article = new Article();
$article->title = 'New article';
$article->save();

// Link the image to the user
$service->attachTempFiles($user, 'avatar', $tempKey);

// Link the same image to the article (if the article has an image field)
$service->attachTempFiles($article, 'image', $tempKey);

// Note: attachTempFiles searches for all files with that session key and will link all of them.
// If you want to link the same file to both models, it will duplicate the link (ensure files are independent).
```

**Note:** `attachTempFiles` links all files with the given `session_key`. If there are multiple files under the same key, all will be linked. You can use separate keys for each operation if you need to differentiate.

---

## 10. Using `validateFile` with Base64 Data in an API

**Scenario:** Before uploading a base64 image, validate it.

```php
$base64 = $request->input('image_base64');

try {
    $service->validateFile(Product::class, 'image', $base64);
    // File is valid
    $result = $service->upload(Product::class, 'image', $base64, ['model' => $product]);
} catch (ApplicationException $e) {
    return response()->json(['error' => $e->getMessage()], 400);
}
```

---

## 11. Retrieve Temporary Files (Unassociated)

**Scenario:** After uploading files with a temporary session key, display them before saving the model.

```php
$tempKey = $request->input('temp_session_key');
$result = $service->getFiles(Product::class, 'gallery', null, ['temp_session_key' => $tempKey]);

if ($result['status']) {
    echo "Number of temporary files: " . count($result['data']);
    foreach ($result['data'] as $file) {
        echo "<img src='{$file['path']}'>";
    }
}
```

---

## 12. Handling Delete Errors (File Not Found)

```php
$result = $service->deleteFile(999999);

if (!$result['status']) {
    switch ($result['code']) {
        case 404:
            echo "File not found";
            break;
        default:
            echo "Error: " . $result['message'];
    }
}
```

---

## 13. Integrating `FileUploadService` with `FileUploadUserManager` for Custom Frontend Permissions

**Scenario:** A frontend user wants to upload an avatar; we use `FileUploadUserManager` to check a custom permission.

```php
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();

if (!$user) {
    return response()->json(['error' => 'Unauthorized'], 401);
}

// Assume the user model is registered in Registry with add permission for avatar
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['model' => $user]);
```

---

## 14. Adding Custom Thumbnail Sizes When Retrieving Files

**Scenario:** We need custom thumbnail sizes different from the defaults.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'icon' => [50, 50, 'crop'],
        'preview' => [200, 150, 'auto'],
        'full' => [800, 600, 'auto'],
    ],
]);

// The result will contain keys icon, preview, full for each file.
```

---

## 15. Upload Multiple Files Using Base64 Data in a Single Request

**Scenario:** An API endpoint receives an array of base64 strings and uploads them all.

```php
$base64Array = $request->input('images'); // ['data:image/...', 'data:image/...', ...]

$result = $service->uploadMultiple(Product::class, 'gallery', $base64Array, [
    'model' => $product,
]);

if ($result['status']) {
    echo "Successfully uploaded " . $result['process_data']['success_count'] . " images.";
}
```

---

## 16. Best Practices

1. **Use temporary session keys for new models**: to avoid losing files if the model is not saved.
2. **Validate the file before upload using `validateFile`**: to provide clear error messages.
3. **Log transaction IDs**: to track upload operations.
4. **Use `uploadMultiple` for multiple fields**: instead of calling `upload` in a loop, as it aggregates errors and provides statistics.
5. **Don't rely only on `temp_session_key`**: after saving the model, use `attachTempFiles` to transfer ownership.
6. **Handle errors by code**: 400, 403, 404, 500 to give appropriate responses.
7. **Enable `app.debug` in development**: for full error details.
8. **Use `with_thumbs` only when needed**: to reduce response size and improve performance.
9. **Store temporary session keys in session or a temporary database**: to avoid losing them.
10. **Test permissions thoroughly**: ensure unauthorized users cannot upload or delete files.

---

## 17. Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| `ApplicationException: Field ... not registered for model ...` | Field not registered in `FileUploadRegistry`. | Add the field to the registration. |
| `You do not have permission to upload files for this field` | User lacks `add` permission for the field. | Check `permissions` settings in Registry or user permissions. |
| `Invalid file data` | Provided data is neither `UploadedFile` nor valid base64. | Ensure correct data format. |
| `File not found` (when deleting) | File ID does not exist. | Verify the ID. |
| `File size exceeds allowed limit` | File size > `max_filesize`. | Compress the file or increase `max_filesize`. |
| `File type not allowed` | Extension not in `allowed_types`. | Use an allowed type or adjust `allowed_types`. |

---

## 18. Integration with Custom Permission Systems

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

## Conclusion

The `FileUploadService` class provides a powerful and unified interface for managing file uploads in NanoSoft applications. Through the examples above, developers can implement any file upload scenario: from simple uploads with direct linking to temporary uploads for unsaved models, permission and constraint checks, retrieving files with thumbnails, and secure deletion. We recommend using this service as the single layer for handling files throughout the application to ensure consistency and security.

For details on the response structure when using the `FileUploadController`, refer to [API Documentation](./Docs-API-Documentation-en.md).

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)

