## 2026-04-10 – 2026-04-11

** `Nano.FileUpload` Plugin Update – Version 1.2.1**

### Restructuring the Permission Validation System and Adding Fine-Grained Control at Model and Field Levels

---

### Summary of Updates

In version **1.2.1**, the permission validation system in `FileUploadRegistry` was completely restructured, with the addition of multiple validation layers (global, module, field) and the ability to independently disable specific operations. `FileUploadService` was also improved to use the new `validate` methods, the `attachTempFiles` function was completely rewritten, and `FileUploadException` was updated with new error codes and appropriate HTTP statuses.

---

### Version Goals

- **Unify and restructure validation methods** in `FileUploadRegistry` to be clearer and more organized.
- **Add `disabled_operations` mechanism** at model and field levels to independently disable specific operations (`add`, `edit`, `delete`, `view`).
- **Provide specialized `validate` methods** that throw appropriate exceptions instead of returning `false`, simplifying code in the service layer.
- **Improve `FileUploadService` code** to leverage the new `validate` methods, reducing duplicated code and unifying error messages.
- **Add a fully enhanced `attachTempFiles` function** with a unified response structure and WebSocket events.
- **Update `FileUploadException`** with new error codes and modify HTTP codes for some errors to avoid conflicts.

---

### New Features

#### 1. Specialized Validation Methods in `FileUploadRegistry`

A set of new methods was added to separate validation levels:

| Method | Description |
|--------|-------------|
| `canGlobal($operation)` | Check if the operation is globally enabled (via global settings `disable_upload`, `disable_edit`, `disable_delete`, `disable_get`). |
| `validateGlobal($operation)` | Like `canGlobal` but throws `FileUploadException` if the operation is disabled. |
| `canModel($modelClass, $operation)` | Check if the operation is enabled at model level (via `disabled_operations` in model definition). |
| `validateModel($modelClass, $operation)` | Like `canModel` but throws exception if operation is disabled. |
| `canField($modelClass, $field, $operation)` | Check if the operation is enabled at field level (via `disabled_operations` in field definition). |
| `validateField($modelClass, $field, $operation)` | Like `canField` but throws exception. |
| `validate($modelClass, $operation, $userType, $user, $field)` | Integrated method that combines checks for global, module, field, user type, and permissions, throwing the appropriate exception at the first failure. |

**Example of using `validate`:**
```php
// In FileUploadService::upload
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

#### 2. Addition of `disabled_operations` Property

It is now possible to disable specific operations at the model or field level using the `disabled_operations` key in registration definitions:

```php
// In model definition (Plugin::registerFileUploadFields)
\Nano\Shop\Models\Product::class => [
    'disabled_operations' => ['delete'], // disable deletion at product level
    'fields' => [
        'image' => [
            'type' => 'image',
            'disabled_operations' => ['edit'], // disable image replacement only
        ],
    ],
];
```

This allows precise prevention of certain operations without needing to disable full permissions or modify global settings.

#### 3. Improved `can()` Method in `FileUploadRegistry`

The `can()` method was rewritten to rely on the new validation chain:

1. Global level check (`canGlobal`)
2. Module level check (`canModel`)
3. Field level check (`canField`) if applicable
4. User type check (`isUserTypeAllowed`)
5. Specific permissions check (`permissions`)

This ensures that no operation that is disabled at any level is allowed.

#### 4. New Error Codes in `FileUploadException`

The following constants were added:

- `ERR_EDIT_DISABLED_GLOBALLY` – when editing is globally disabled (`disable_edit = true`)
- `ERR_MODEL_OPERATION_DISABLED` – when operation is disabled at model level
- `ERR_FIELD_OPERATION_DISABLED` – when operation is disabled at field level

**HTTP code changes:**
- `ERR_PERMISSION_DENIED` HTTP code changed from `403` to `422` (Unprocessable Entity) to avoid conflict with "inactive account" status which uses `403`.
- `ERR_UPLOAD_DISABLED_GLOBALLY`, `ERR_EDIT_DISABLED_GLOBALLY`, `ERR_DELETE_DISABLED_GLOBALLY`, `ERR_GET_DISABLED_GLOBALLY` now use code `503` (Service Unavailable).
- `ERR_MODEL_OPERATION_DISABLED`, `ERR_FIELD_OPERATION_DISABLED` use code `423` (Locked).

#### 5. Improved `upload` Method in `FileUploadService`

Manual permission checking code was replaced with `validate` calls:

**Before:**
```php
if (!$this->registry->can($modelClass, $operation, $userType, $user, $field)) {
    throw new FileUploadException(...);
}
```

**After:**
```php
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

The `skip_permission_check` option was also added to bypass permission checks (useful in automated tests).

#### 6. Complete Rewrite of `attachTempFiles` Method

The method now returns a unified response structure containing `code`, `status`, `message`, `data`, `error_code`, `process_data`, `debug`. It performs:

- Check that the model and field are registered in `FileUploadRegistry`.
- Permission check (using `edit` operation because attaching is considered a modification to the model).
- Validate the temporary key using `validateTempSessionKey`.
- Verify that `modelClass` and `field` match the key data.
- Verify that the current user is the creator of the temporary files (compare `userId` and `userType`).
- Find temporary files (`session_key`).
- Attach files to the model (set `attachment_id`, `attachment_type`, clear `session_key`, `expires_at`).
- Add files to the model relation (if exists).
- Fire `nano.fileupload.afterAttach` event.
- Send WebSocket notification.

**Example success response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Successfully attached 3 files",
    "data": [
        {"id": 10, "title": "image.jpg", "path": "/storage/...", "size": 1024, "content_type": "image/jpeg"}
    ],
    "process_data": {
        "field_config": {...},
        "key_data": {...},
        "attached_count": 3,
        "temp_session_key": "tmp_xxx",
        "user_type": "backend"
    }
}
```

#### 7. Improved `getFiles` Method

Improved extraction of `morphClass` using `app($modelClass)->getMorphClass()` instead of `$modelClass::getMorphClass()` to avoid errors in some contexts (e.g., when the model is not preloaded). Permission checking for `view` was also unified using `$this->registry->validate($modelClass, 'view', ...)`.

#### 8. Improved `deleteFile` Method

The permission checking logic was restructured to cover four cases:

| Case | Description | Validation Method |
|------|-------------|-------------------|
| 1 | `modelClass` and `field` explicitly provided | Use `registry->can()` with provided parameters |
| 2 | File attached to saved model (`attachment_type`, `attachment_id`, `field`) | Extract values from file and use `registry->can()` |
| 3 | Temporary file (`session_key`) | Validate key and match user |
| 4 | Insufficient information | Reject deletion directly |

**Improvement for temporary files:**
```php
elseif ($file->session_key) {
    $keyData = $this->validateTempSessionKey($file->session_key);
    if (!$keyData) throw ...;
    $currentUserId = $user->getKey() ?? 'guest';
    if ($keyData['userId'] != $currentUserId) throw ...;
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 9. Improved `logFailedAttempt` Method

Improved extraction of `userId` to support multiple types of user objects:

```php
$userId = 'guest';
if ($user) {
    if (method_exists($user, 'getKey')) $userId = $user->getKey();
    elseif (property_exists($user, 'id')) $userId = $user->id;
    elseif (method_exists($user, 'getId')) $userId = $user->getId();
    else $userId = 'unknown';
}
```

This reduces errors when different user types (backend, frontend, custom) exist.

#### 10. Added `skip_permission_check` Option in Multiple Methods

The `skip_permission_check` option was added to `upload`, `uploadMultiple`, and `attachTempFiles` methods to completely bypass permission checks. This option is very useful in automated test environments (e.g., `FileUploadPlusTest`) where there is no real user or specific permissions.

**Usage example:**
```php
$result = $service->upload($modelClass, $field, $file, [
    'skip_permission_check' => true
]);
```

---

### Benefits and Added Value

- **Cleaner code**: `validate` methods eliminate the need for repetitive conditional blocks for permission checking.
- **Fine-grained control**: `disabled_operations` allows disabling specific operations at model or field level without affecting overall permissions or global settings.
- **Better compatibility with authentication systems**: Changing HTTP code for `ERR_PERMISSION_DENIED` from `403` to `422` avoids confusion between "unauthorized" and "inactive account".
- **Professional `attachTempFiles` method**: The method is now complete and integrated with the unified response structure, events, and WebSocket notifications.
- **Testing flexibility**: The `skip_permission_check` option simplifies writing unit tests that don't rely on a real user.
- **Enhanced security**: Permission checking in `deleteFile` covers all possible cases, preventing unauthorized deletion.

---

### Breaking Changes

1. **HTTP code change for `ERR_PERMISSION_DENIED`**:
   - If you have code that expects a `403` for this error, it will now receive `422`. It is recommended to update any error handling that relies on the code.

2. **`attachTempFiles` method now returns a different structure**:
   - Previously returned `true/false` or a simple array. Now returns a unified array containing `code`, `status`, `message`, `data`, etc. If you call this method directly, you will need to update your code to handle the new structure.

3. **Addition of `disabled_operations`**:
   - Does not affect existing settings unless you explicitly add it. Default settings allow all operations.

4. **New `validate` methods**:
   - They are used inside `FileUploadService`, but if you call `can()` directly, it will still work (with the improved chain).

---

### Upgrade Requirements

1. **Update code**:
   - Replace `FileUploadRegistry.php` with the version containing the new `validate` methods and `disabled_operations` property.
   - Replace `FileUploadService.php` with the improved version that uses `validate` and contains the new `attachTempFiles`.
   - Replace `FileUploadException.php` with the version containing the new error codes.

2. **No new migrations**:
   - This version does not require database changes or new columns.

3. **No configuration changes**:
   - `config.php` remains the same. Existing environment variables (`disable_upload`, `disable_edit`, `disable_delete`, `disable_get`) can be used the same way.

4. **Review model definitions** (optional):
   - If you wish to use `disabled_operations`, add the key to model or field definitions as shown in the examples above.

5. **Compatibility testing**:
   - It is recommended to run the `FileUploadPlusTest` (version 1.2.0) to ensure all operations work correctly with the new changes.
   - Verify that `edit` permissions work as expected when replacing files (if used).

6. **Update any code that depends on `attachTempFiles`**:
   - If you call `attachTempFiles` directly, you will need to modify your code to handle the new response structure.

---

### Conclusion

Version **1.2.1** represents a significant leap in the maturity of permission management within the `Nano.FileUpload` add-on. By adding specialized `validate` methods, the `disabled_operations` property, and improving the `attachTempFiles` and `deleteFile` methods, the system has become more organized, secure, and flexible. These changes enable the add-on to handle complex scenarios such as file replacement, temporarily disabling specific operations, and multi-level permission checks, while maintaining backward compatibility as much as possible.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

---