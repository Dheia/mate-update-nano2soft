## 2026-04-10 – 2026-04-14

**`Nano.FileUpload` Plugin Updates – Versions 1.2.1 and 1.2.2**

### Summary of Updates

The `Nano.FileUpload` add-on has undergone two major consecutive developments in versions **1.2.1** and **1.2.2**. The first focused on a comprehensive restructuring of the permission validation system and the addition of fine-grained control mechanisms at the model and field levels, while the second focused on unifying and simplifying the validation logic for temporary session keys and creating a dedicated trait containing advanced functions for this purpose.

---

## Version 1.2.1 – Restructuring the Permission Validation System

### Version Goals

- **Unify and restructure validation methods** in `FileUploadRegistry` to be clearer and more organized.
- **Add `disabled_operations` mechanism** at model and field levels to independently disable specific operations (`add`, `edit`, `delete`, `view`).
- **Provide specialized `validate` methods** that throw appropriate exceptions instead of returning `false`, simplifying code in the service layer.
- **Improve `FileUploadService` code** to leverage the new `validate` methods, reducing duplicated code and unifying error messages.
- **Add a fully enhanced `attachTempFiles` function** with a unified response structure and WebSocket events.
- **Update `FileUploadException`** with new error codes and modify HTTP codes for some errors to avoid conflicts.

### New Features

#### 1. Specialized Validation Methods in `FileUploadRegistry`

A set of new methods was added to separate validation levels:

| Method | Description |
|--------|-------------|
| `canGlobal($operation)` | Check if the operation is globally enabled (via global settings). |
| `validateGlobal($operation)` | Like `canGlobal` but throws `FileUploadException` if the operation is disabled. |
| `canModel($modelClass, $operation)` | Check if the operation is enabled at model level (`disabled_operations`). |
| `validateModel($modelClass, $operation)` | Like `canModel` but throws exception if operation is disabled. |
| `canField($modelClass, $field, $operation)` | Check if the operation is enabled at field level. |
| `validateField($modelClass, $field, $operation)` | Like `canField` but throws exception. |
| `validate($modelClass, $operation, $userType, $user, $field)` | Integrated method that combines checks for global, module, field, user type, and permissions, throwing the appropriate exception at the first failure. |

#### 2. Addition of `disabled_operations` Property

It is now possible to disable specific operations at the model or field level using the `disabled_operations` key in registration definitions:

```php
// At model level
'disabled_operations' => ['delete', 'edit']

// At field level
'disabled_operations' => ['view']
```

This allows precise prevention of certain operations without needing to disable full permissions.

#### 3. Improved `can()` Method in `FileUploadRegistry`

The `can()` method was rewritten to rely on the new validation chain: global ← module ← field ← user type ← permissions. This ensures that no operation that is disabled at any level is allowed.

#### 4. New Error Codes in `FileUploadException`

The following constants were added:

- `ERR_EDIT_DISABLED_GLOBALLY`
- `ERR_MODEL_OPERATION_DISABLED`
- `ERR_FIELD_OPERATION_DISABLED`

Additionally, the HTTP code for `ERR_PERMISSION_DENIED` was changed from `403` to `422` (Unprocessable Entity) to avoid conflict with the `403` status commonly used for inactive accounts.

HTTP code `503` was assigned to globally disabled errors (`UPLOAD_DISABLED_GLOBALLY`, etc.), and code `423` (Locked) for module or field level errors.

#### 5. Improved `upload` Method in `FileUploadService`

Manual permission checking code was replaced:

```php
// Old code (multiple lines)
if (!$this->registry->can($modelClass, $operation, $userType, $user, $field)) {
    throw new FileUploadException(...);
}

// New code (simple and unified)
$this->registry->validate($modelClass, $operation, $userType, $user, $field);
$this->registry->validateField($modelClass, $field, $operation);
```

#### 6. Complete Rewrite of `attachTempFiles` Method

The method now returns a unified response structure containing `code`, `status`, `message`, `data`, `error_code`, `process_data`, and performs:

- Check that the model and field are registered.
- Permission check (`edit` operation).
- Validate the temporary key and match model, field, and user.
- Find temporary files and attach them to the model.
- Clear file expiration (`expires_at = null`) and `session_key`.
- Fire `nano.fileupload.afterAttach` event.
- Send WebSocket notification.

#### 7. Addition of `skip_permission_check` Option

The `skip_permission_check` option was added to `upload`, `uploadMultiple`, and `attachTempFiles` methods to bypass permission checks, which is very useful in automated testing environments.

#### 8. Improved `getFiles` Method

Improved extraction of `morphClass` using `app($modelClass)->getMorphClass()` instead of `$modelClass::getMorphClass()` to avoid errors in some contexts. Permission checking was also unified using `validate()`.

#### 9. Improved `deleteFile` Method

Four cases for permission checking were covered:

1. Explicitly passing `modelClass` and `field`.
2. File attached to a saved model (extracting `attachment_type` and `field`).
3. Temporary file (`session_key`) – validate key and match user.
4. If insufficient information is available, deletion is rejected.

`registry->can()` is used in the first two cases to ensure permission checking at model and field levels.

#### 10. Improved `logFailedAttempt` Method

Improved extraction of `userId` to support multiple types of user objects: `getKey()`, `id`, `getId()`, reducing errors when different user types (backend, frontend, custom) exist.

### Benefits

- **Cleaner code**: `validate` methods eliminate the need for repetitive conditional blocks for permission checking.
- **Fine-grained control**: `disabled_operations` allows disabling specific operations without affecting overall permissions.
- **Better compatibility with authentication systems**: Changing HTTP code for `403` avoids confusion between "unauthorized" and "inactive account".
- **Professional `attachTempFiles` method**: The method is now complete and integrated with the unified response structure and events.
- **Testing flexibility**: The `skip_permission_check` option simplifies writing unit tests that don't rely on a real user.

---

## Version 1.2.2 – Unifying Temporary Session Key Validation

### Version Goals

- **Create a dedicated trait** (`HasFileUploadsMatchTempKey`) containing an advanced function to validate temporary keys and match them against model, field, and user.
- **Unify the validation logic** used in four different methods (`upload`, `deleteFile`, `attachTempFiles`, `getFiles`) in one place.
- **Remove duplicated code** that previously exceeded 50 lines of scattered manual validation operations.
- **Add advanced options** to control validation behavior: strict mode, grace period for expired keys, result caching, and performance data collection.
- **Improve event compatibility** by adding the `fireEventSafeInTrait` function that supports both Laravel and OctoberCMS.

### New Features

#### 1. Trait `HasFileUploadsMatchTempKey`

The trait was created at path `Nano\FileUpload\Classes\FileUploadService\HasFileUploadsMatchTempKey` and includes the following components:

##### a. `validateAndMatchTempKey` Function (Core Function)

This function combines everything needed for temporary key validation in one place. It accepts the following parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `$tempKey` | `string|array` | Temporary session key (string) or its data (array) |
| `$modelClass` | `string` | Expected model class name |
| `$field` | `string` | Expected field name |
| `$user` | `mixed|null` | Current user (optional) |
| `$options` | `array` | Additional options to control behavior |

And provides the following options (with default values):

| Option | Type | Description |
|--------|------|-------------|
| `throw_on_failure` | `bool` | Throw exception on failure (`true`) or return error result array (`false`) |
| `skip_user_check` | `bool` | Skip user check (for internal use or tests) |
| `strict_mode` | `bool` | Strict mode: requires exact model and field match (`true`) |
| `stop_on_first_failure` | `bool` | Stop at first failure (`true`) or collect errors (`false`) |
| `allow_expired_key` | `bool` | Allow expired keys within a grace period |
| `expiry_grace_period` | `int` | Grace period in seconds (default `300`) |
| `custom_validator` | `callable|null` | Additional validation function (receives `$keyData` and returns `bool`) |
| `cache_results` | `bool` | Cache validation results within the same request |
| `collect_metadata` | `bool` | Collect performance data (execution time, steps performed) |
| `key_data_overrides` | `array` | Overrides for key data (e.g., change `userId`) |
| `before_validation` | `callable|null` | Function called before validation starts |
| `after_validation` | `callable|null` | Function called after validation ends |
| `log_failures_only` | `bool` | Log failures only (`true`) or log everything (`false`) |

**Basic usage example:**
```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true]
);
```

##### b. `manualValidateTempSessionKeyWithGrace` Function

An internal helper function used when the `allow_expired_key` option is enabled. It validates an expired key with a grace period without modifying the original `validateTempSessionKey` function.

##### c. `fireEventSafeInTrait` Function

A helper function to fire events safely and compatibly with both Laravel and OctoberCMS. It first checks whether the used object (e.g., `FileUploadService`) has a `fireEventSafe` method, uses it if found, otherwise applies equivalent logic supporting `Event::fire`, `Event::dispatch`, and the `event()` helper.

#### 2. Updated `upload` Method in `FileUploadService`

When `temp_session_key` is present in options, `validateAndMatchTempKey` is now called to verify that the key belongs to the same model, field, and user before using it.

```php
} elseif (isset($options['temp_session_key']) && !empty($options['temp_session_key'])) {
    $tempSessionKey = $options['temp_session_key'];
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey($tempSessionKey, $modelClass, $field, $user, [
        'throw_on_failure' => true,
        'strict_mode' => true,
        'skip_user_check' => false,
    ]);
}
```

#### 3. Updated `deleteFile` Method in `FileUploadService`

The manual code for temporary files was replaced with a call to `validateAndMatchTempKey` with `strict_mode = false` (because a temporary file may not have an `attachment_type`).

```php
elseif ($file->session_key) {
    $keyData = $this->validateAndMatchTempKey(
        $file->session_key,
        $file->attachment_type ?: '',
        $file->field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => false, 'skip_user_check' => false]
    );
    $usedModelClass = $keyData['modelClass'];
    $usedField = $keyData['field'];
    $isAuthorized = true;
}
```

#### 4. Updated `attachTempFiles` Method in `FileUploadService`

More than 20 lines of manual validation (key validation, comparing `modelClass`, comparing `field`, comparing `userId`, comparing `userType`) were replaced with a single call:

```php
$keyData = $this->validateAndMatchTempKey(
    $tempSessionKey,
    $modelClass,
    $field,
    $user,
    ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
);
$process_data['key_data'] = $keyData;
```

#### 5. Updated `getFiles` Method in `FileUploadService`

The manual key validation code before using it in the query was replaced:

```php
} elseif (isset($options['temp_session_key'])) {
    $user = $this->userManager->getUser();
    $this->validateAndMatchTempKey(
        $options['temp_session_key'],
        $modelClass,
        $field,
        $user,
        ['throw_on_failure' => true, 'strict_mode' => true, 'skip_user_check' => false]
    );
    $query->where('session_key', $options['temp_session_key']);
}
```

#### 6. Improved Event Firing in the Trait

All direct `Event::fire` calls inside `validateAndMatchTempKey` were replaced with calls to `$this->fireEventSafeInTrait`, ensuring that events work uniformly in both Laravel and OctoberCMS environments.

### Benefits

- **Removed duplicated code**: Eliminated more than 50 lines of repetitive manual validation operations.
- **Increased security**: Unified validation ensures no gaps from forgetting to check model or user matching in any function.
- **High flexibility**: Multiple options in `validateAndMatchTempKey` allow customizing behavior per case (strict mode, grace period, caching, etc.).
- **Improved compatibility**: `fireEventSafeInTrait` ensures events work in all environments.
- **Ease of testing**: `skip_user_check`, `cache_results`, and `collect_metadata` options simplify unit testing and performance analysis.
- **Built-in documentation**: The new function includes comprehensive documentation for all options and parameters.

---

## Version Summary (1.2.1 and 1.2.2)

| Version | Key Features |
|---------|---------------|
| **1.2.1** | Restructured permission validation system by adding specialized `validate` methods, added `disabled_operations` at model and field levels, improved `can()` method, added new error codes, improved `upload`, `deleteFile`, `attachTempFiles`, `getFiles` methods, and added `skip_permission_check` option. |
| **1.2.2** | Created `HasFileUploadsMatchTempKey` trait with advanced `validateAndMatchTempKey` function, unified temporary key validation logic across all methods, added advanced options (strict mode, grace period, caching, etc.), and added `fireEventSafeInTrait` function to improve event compatibility. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace `FileUploadService.php` with the updated version (1.2.2).
   - Add the trait `HasFileUploadsMatchTempKey` at path `classes/FileUploadService/HasFileUploadsMatchTempKey.php`.
   - Update `FileUploadRegistry.php` with the version containing the new `validate` methods (1.2.1).
   - Update `FileUploadException.php` with the version containing the new error codes.

2. **No new migrations**: Both versions do not require database changes.

3. **No configuration changes**: `config.php` remains the same.

4. **Compatibility testing**:
   - It is recommended to run the `FileUploadPlusTest` tests to ensure all operations work correctly.
   - Verify that `edit` permissions work as expected when replacing files.
   - Ensure that invalid temporary keys cause appropriate errors.

5. **Review model definitions** (optional):
   - If using `disabled_operations`, ensure they are correctly formatted in field definitions.
   - If using `edit` permission, ensure it is added in the `permissions` array.

---

### Conclusion

Versions **1.2.1** and **1.2.2** represent a significant leap in the maturity of the `Nano.FileUpload` add-on. While the first focused on restructuring the permission validation system to be more organized and precise, the second introduced a unified and advanced mechanism for validating temporary session keys, making the code cleaner and more secure. With these updates, the add-on is capable of handling complex scenarios such as file replacement, multi-level permission checks, and temporary file management with high flexibility, while maintaining full compatibility with Laravel and OctoberCMS environments.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

