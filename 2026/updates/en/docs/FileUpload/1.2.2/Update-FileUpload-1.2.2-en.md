## 2026-04-12 – 2026-04-13

**`Nano.FileUpload` Plugin Updates – Version 1.2.2**

### Summary of Updates

As part of the ongoing effort to enhance the security and flexibility of the file upload system, version 1.2.2 focuses on unifying and simplifying the validation process for temporary session keys across all service methods. A new trait called `HasFileUploadsMatchTempKey` was created, containing an advanced function `validateAndMatchTempKey` that combines key validation and matching against model, field, and user, with advanced options to control behavior (strict mode, grace period, caching, etc.). A helper function `fireEventSafeInTrait` was also added to ensure event firing compatibility with both Laravel and OctoberCMS.

The `upload`, `deleteFile`, `attachTempFiles`, and `getFiles` methods in `FileUploadService` were updated to use this new function instead of duplicated code, resulting in reduced complexity and increased security and uniformity.

---

### Version Goals

- **Unify temporary key validation logic**: Create a single central point for validating keys and matching them against model, field, and user.
- **Remove duplicated code**: Replace repetitive manual validation operations in several methods with a single function call.
- **Enhance security**: Add advanced options such as strict mode, grace period for expired keys, and retry support on expiry.
- **Improve compatibility**: Ensure events fire correctly in both Laravel and OctoberCMS environments via `fireEventSafeInTrait`.
- **Facilitate testing**: Provide options like `skip_user_check` and `cache_results` to simplify unit testing.

---

### New Features

#### 1. Trait `HasFileUploadsMatchTempKey`

The trait was created at path `Nano\FileUpload\Classes\FileUploadService\HasFileUploadsMatchTempKey` and contains:

##### a. `validateAndMatchTempKey` Function

This function is the core of the update. It validates the temporary key and matches it against expected data. The function provides the following options:

| Option | Type | Description |
|--------|------|-------------|
| `throw_on_failure` | bool | Throw exception on failure (true) or return error result array (false) |
| `skip_user_check` | bool | Skip user check (for internal use or tests) |
| `strict_mode` | bool | Strict mode: requires exact model and field match (true), or allows mismatch (false) |
| `stop_on_first_failure` | bool | Stop at first failure (true) or collect errors (false) |
| `allow_expired_key` | bool | Allow expired keys within a grace period |
| `expiry_grace_period` | int | Grace period in seconds (default 300) |
| `custom_validator` | callable | Additional validation function (receives `$keyData` and returns bool) |
| `cache_results` | bool | Cache validation results within the same request |
| `collect_metadata` | bool | Collect performance data (execution time, steps performed) |

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

- **Before update**: Any temporary key passed from the client was accepted without validating or matching it against the model and field.
- **After update**: When `temp_session_key` is present in options, `validateAndMatchTempKey` is called to verify that the key belongs to the same model, field, and user; otherwise an appropriate exception is thrown.

**New code:**
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

- **Before update**: Manual key validation followed by separate `userId` comparison.
- **After update**: Use `validateAndMatchTempKey` with `strict_mode = false` (because a temporary file may not have an `attachment_type`) to unify the logic.

**New code:**
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

- **Before update**: Called `validateTempSessionKey` then separately compared `modelClass`, `field`, `userId`, and `userType` (more than 20 lines of code).
- **After update**: Replaced all this logic with a single call to `validateAndMatchTempKey`.

**New code:**
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

- **Before update**: Manual key validation followed by separate comparison of `userId`, `userType`, `modelClass`, and `field`.
- **After update**: Use `validateAndMatchTempKey` to validate the key before using it in the query.

**New code:**
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

All direct `Event::fire` calls inside `validateAndMatchTempKey` were replaced with calls to `$this->fireEventSafeInTrait`, ensuring events work in both Laravel and OctoberCMS environments, and leveraging the `fireEventSafe` method in `FileUploadService` if available.

---

### Benefits

- **Reduced duplicated code**: Removed more than 50 lines of repetitive code from various methods.
- **Increased security**: Unified validation ensures no gaps from forgetting to check model or user matching in any function.
- **High flexibility**: Multiple options in `validateAndMatchTempKey` allow customizing behavior per case (strict mode, grace period, caching, etc.).
- **Improved compatibility**: `fireEventSafeInTrait` ensures events work in all environments.
- **Ease of testing**: `skip_user_check`, `cache_results`, and `collect_metadata` options simplify unit testing and performance analysis.
- **Built-in documentation**: The new function includes comprehensive documentation for all options.

---

### Upgrade Requirements

1. **Update code**: Replace `FileUploadService.php` with the updated version that uses the new trait.
2. **Add the trait**: Create the file `HasFileUploadsMatchTempKey.php` in the path `classes/FileUploadService/` with the required content.
3. **No new migrations**: This version does not require database changes.
4. **No configuration changes**: `config.php` remains the same.
5. **Compatibility testing**: It is recommended to run the `FileUploadPlusTest` tests to ensure all operations work correctly.

---

### Conclusion

Version 1.2.2 is an important step toward unifying and simplifying temporary key validation logic in the file upload system. By introducing the `HasFileUploadsMatchTempKey` trait and the `validateAndMatchTempKey` function, it is now possible to perform secure and comprehensive validation of temporary keys in any function that needs it, with advanced options for fine-grained control. The updates made to the `upload`, `deleteFile`, `attachTempFiles`, and `getFiles` methods make the code cleaner and less error-prone, while maintaining full backward compatibility.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)

---
