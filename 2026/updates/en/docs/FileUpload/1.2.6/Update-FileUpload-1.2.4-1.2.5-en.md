## 2026-04-16 – 2026-04-18

**`Nano.FileUpload` Add-on Updates – Versions 1.2.4 and 1.2.5**

### Summary of Updates

The add-on has seen two important developments in versions **1.2.4** and **1.2.5**:

- **Version 1.2.4** added a new security feature at the field level: `allow_upload_only_when_model_exists`, which prevents uploading files to a specific field until the original model has been saved (a valid `model_id` exists). This feature is particularly useful for multiple fields (e.g., image galleries) where images should not be uploaded before the main record is created.

- **Version 1.2.5** completely restructured the API endpoints for querying modules and permissions by moving the `model_class` and `field` parameters from route parameters to query parameters. This change solves the problem of encoding model names containing backslashes (`\`) and makes the routes cleaner and more secure.

---

## Version 1.2.4 – Preventing File Upload Before Model Save

### Version Goals

- **Add an additional security layer** to prevent uploading files to fields that require the model to exist first (e.g., image galleries, documents attached to a saved record).
- **Improve developer experience** by providing the `allow_upload_only_when_model_exists` option in field definitions, allowing the prevention of temporary uploads (using `temp_session_key`) for that field.
- **Avoid accumulation of orphan files** (unattached) in the system.

### New Features

#### 1. Adding the `allow_upload_only_when_model_exists` Option in Field Definition

It is now possible to set this option on any field definition (especially for fields of type `multiple` or `image` that require the model to exist):

```php
'fields' => [
    'gallery' => [
        'type' => 'multiple',
        'max_filesize' => 2048,
        'allowed_types' => 'jpg,jpeg,png',
        'max_files' => 10,
        'allow_upload_only_when_model_exists' => true, // ⬅️ new
    ],
],
```

- **Default value:** `false` (allows temporary upload even for unsaved models).
- **When enabled (`true`):** Prevents uploading files to this field if a valid `model_id` is not provided (i.e., the model does not exist in the database). The request will be rejected with an error message `FILE_UPLOAD_PERMISSION_DENIED` and the message `model_must_exist_before_upload`.

#### 2. Validation in `FileUploadService::upload()`

Logic was added to the `upload()` method to check this option:

- It checks whether the model exists (`model_exists`) either through `options['model']` (if present and saved) or by searching for `model_id` and fetching the model.
- If the option is enabled and the model does not exist, a `FileUploadException` is thrown with the appropriate error code.

#### 3. New Translation Message

A new translation key `errors.model_must_exist_before_upload` was added:

```php
'model_must_exist_before_upload' => 'Cannot upload files to the ":field" field until ":model" is saved. Please save the :model first and then try again.',
```

### Benefits

- **Prevent orphan files:** Ensures there are no files unattached to any database record.
- **Fine-grained control:** The feature can be enabled only on specific fields (e.g., galleries) while leaving other fields (e.g., avatar) allowing temporary upload.
- **Enhanced security:** Prevents users from uploading files to fields that require the original record to exist before it is created.

---

## Version 1.2.5 – Restructuring API Routes (Query Parameters instead of Route Parameters)

### Version Goals

- **Solve the problem of encoding model names** containing backslashes (`\`) in routes (e.g., `Nano\Shop\Models\Product`). Passing them in the route required encoding (`Nano%5CShop%5CModels%5CProduct`), making routes impractical and unreadable.
- **Simplify endpoints** by using query parameters (`?model_class=...`) instead of route parameters.
- **Maintain backward compatibility** by keeping the old routes (deprecated) while making the new routes available.

### New Features

#### 1. New Endpoints (using query parameters)

| Function | Old Path (deprecated) | New Path |
|----------|----------------------|----------|
| Get module config | `GET /models/{modelClass}` | `GET /model/config?model_class=...` |
| Get field config | `GET /models/{modelClass}/fields/{field}` | `GET /field/config?model_class=...&field=...` |
| Get field constraints | `GET /models/{modelClass}/fields/{field}/constraints` | `GET /field/constraints?model_class=...&field=...` |
| Get processing options | `GET /processing-options/{modelClass}/{field}` | `GET /processing-options?model_class=...&field=...` |
| Check model permission | `GET /permissions/model/{modelClass}/{operation}` | `GET /permissions/model?model_class=...&operation=...` |
| Check field permission | `GET /permissions/field/{modelClass}/{field}/{operation}` | `GET /permissions/field?model_class=...&field=...&operation=...` |

**Note:** The endpoints `GET /models`, `GET /permissions/global/{operation}`, `POST /permissions/check`, and `POST /temp-key/validate` remain unchanged.

#### 2. Updated Controller Methods in `FileUploadController`

The following methods were modified to read parameters from `Input::get()` (or query parameters) instead of route parameters, with support for passing parameters via the old method as a fallback (for backward compatibility):

- `getModelConfig($modelClass = null)`
- `getFieldConfig($modelClass = null, $field = null)`
- `getFieldConstraints($modelClass = null, $field = null)`
- `getProcessingOptions($modelClass = null, $field = null)`
- `checkModelPermission($modelClass = null, $operation = null)`
- `checkFieldPermission($modelClass = null, $field = null, $operation = null)`

**Example of the new method for `getModelConfig`:**

```php
public function getModelConfig($modelClass = null)
{
    $modelClass = $modelClass ?? Input::get('model_class');
    if (!$modelClass) {
        return $this->errorResponse('Missing model_class parameter', null, 400);
    }
    // ... remaining logic
}
```

#### 3. Updated `routes.php` File

Old routes were commented out (using `/* ... */`) and new routes were added:

```php
// New routes
Route::get('model/config', [ 'as' => 'model.config', 'uses' => 'FileUploadController@getModelConfig' ]);
Route::get('field/config', [ 'as' => 'field.config', 'uses' => 'FileUploadController@getFieldConfig' ]);
Route::get('field/constraints', [ 'as' => 'field.constraints', 'uses' => 'FileUploadController@getFieldConstraints' ]);
Route::get('processing-options', [ 'as' => 'processing.options', 'uses' => 'FileUploadController@getProcessingOptions' ]);
Route::get('permissions/model', [ 'as' => 'permissions.model', 'uses' => 'FileUploadController@checkModelPermission' ]);
Route::get('permissions/field', [ 'as' => 'permissions.field', 'uses' => 'FileUploadController@checkFieldPermission' ]);
```

#### 4. Examples of New Requests

**Get `Product` module config:**

```http
GET /api/v1/fileupload/model/config?model_class=Nano\Shop\Models\Product HTTP/1.1
Authorization: Bearer ...
```

**Get constraints for `image` field:**

```http
GET /api/v1/fileupload/field/constraints?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

**Check `edit` permission on `image` field:**

```http
GET /api/v1/fileupload/permissions/field?model_class=Nano\Shop\Models\Product&field=image&operation=edit HTTP/1.1
Authorization: Bearer ...
```

### Benefits

- **Clean and readable routes** – no need to encode model names.
- **Ease of use from any HTTP client** – pass `model_class` as a normal value in the query string.
- **Backward compatibility** – old routes still work (but are deprecated) and will be removed in a future version.
- **Additional security** – avoid handling user input as part of the route (reduces risk of malicious routing).

---

## Upgrade Requirements (from 1.2.3 to 1.2.5)

1. **Update code**:
   - Replace `FileUploadController.php` with the version containing the updated methods (supports query parameters).
   - Replace `routes.php` with the version containing the new routes.
   - Replace `FileUploadRegistry.php` with the version containing the `allow_upload_only_when_model_exists` option (included in version 1.2.4).
   - Replace `FileUploadService.php` with the version implementing the new option validation.

2. **Update model definitions** (optional):
   - If you wish to use `allow_upload_only_when_model_exists`, add it to the desired field definitions.

3. **No new migrations** – no database changes.

4. **No configuration changes** – `config.php` remains the same.

5. **Update API clients**:
   - If you are using the old endpoints (with curly braces), it is recommended to switch to the new routes immediately. The old routes still work currently but are deprecated and will be removed in version 1.3.0.

6. **Compatibility testing**:
   - Ensure that all new endpoints work correctly with `model_class` as a query parameter.
   - Test a field with `allow_upload_only_when_model_exists = true` to ensure upload is rejected before the model is saved.

---

## Conclusion

Versions **1.2.4** and **1.2.5** represent an important step toward improving system security and ease of use of the API. While version 1.2.4 added fine-grained control over when file uploads are allowed (preventing temporary uploads for fields that require the model to exist), version 1.2.5 introduced a comprehensive restructuring of endpoints to be cleaner and more secure. These updates make the add-on more professional and facilitate its integration with external applications.

---

**Reference Documentation**:
- [General Add-on Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

---

