## 2026-04-03 - 2026-04-05

**Updates to the `Nano.FileUpload` Plugin – Versions 1.0.2 to 1.0.5**

### Summary of Updates

Since the first release of the plugin, we have continued to develop `Nano.FileUpload` based on user feedback and security requirements. Updates in versions 1.0.2 through 1.0.5 included significant improvements in the permission system, default settings for file types, temporary key security, caching, multilingual support, error logging, and blocking dangerous files.

---

## Version 1.0.2 – Security, Permission, and Default Settings Enhancements

### Version Objectives

- Provide global control over API operations (disable upload, delete, retrieve) via environment variables.
- Add default settings for file types (`image`, `audio`, `video`, `file`) to reduce repetition when registering models.
- Enhance temporary session key security by including `userType` and signing with HMAC-SHA256.
- Add validation of the temporary key before linking files.

### New Features

#### 1. Global Control of API Operations

We added global settings in `config.php` that allow disabling upload, delete, or retrieve operations at the entire API level, with control via environment variables:

```ini
NANO_FILE_UPLOAD_DISABLE_UPLOAD=false
NANO_FILE_UPLOAD_DISABLE_DELETE=false
NANO_FILE_UPLOAD_DISABLE_GET=false
```

We added helper methods in `FileUploadRegistry`:
- `isUploadEnabledGlobally()`
- `isDeleteEnabledGlobally()`
- `isGetEnabledGlobally()`

The `can()` method now checks these settings before any operation.

#### 2. Default Settings for File Types

We added a `defaults` section in `config.php` with settings for each file type:
- `image`: max size, allowed types, use caption, thumbnail options.
- `audio`, `video`, `file` with similar settings.

When registering a field, if the developer does not specify `max_filesize`, `allowed_types`, or `use_caption`, the default settings are automatically applied based on the field type.

#### 3. Enhanced Temporary Session Key Security

- Modified `generateTempSessionKey()` to include `userType` and `timestamp` in the signed data.
- Use `hash_hmac('sha256', $data, $secret)` to sign the key, preventing tampering or guessing.
- Added `validateTempSessionKey()` to verify the key's validity and expiration (default expiry of one hour).
- In `attachTempFiles()`, we check:
  - Key validity (signature, expiration).
  - Matching `modelClass` and `field`.
  - Matching `userId` and `userType` with the current user.

This prevents any user from accessing temporary files belonging to another user.

#### 4. Updated `FileUploadRegistry::can()` to Include Global Checks

The `can()` method now first checks the global settings for each operation (`add`, `delete`, `view`) before checking user type and specific permissions.

#### 5. Additional Helper Methods

- `getDefaultConfigForType($type)`: Retrieve default settings for a given type.
- `applyDefaultsToFieldConfig()`: Apply default settings to a field configuration.

### Benefits

- Ability to temporarily disable API operations without modifying code (e.g., for maintenance).
- Reduce repetitive code when registering models.
- Higher security for temporary files, preventing unauthorized access.

---

## Version 1.0.3 – Performance and Flexibility Improvements for `FileUploadRegistry`

### Version Objectives

- Refactor `FileUploadRegistry` to improve performance and scalability.
- Support caching for field configuration and constraints query results.
- Provide flexible registration via `registerDefinition()` and `registerCallback()`.
- Normalize model and field definitions using specialized methods.

### New Features

#### 1. Separation of Raw Definitions from Built Objects

- `$rawDefinitions`: Store definitions as they come from plugins.
- `$builtModels`: Store built objects after construction (Lazy Loading).
- Load definitions from plugins only once (`$loaded` flag).

#### 2. Cache Support

We added a `cache` section in `config.php`:

```php
'registry' => [
    'cache' => [
        'enabled' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_ENABLED', true),
        'ttl' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_TTL', 3600),
    ],
],
```

And methods:
- `getFieldConfig()` and `getFieldConstraints()` use `Cache::get()` and `Cache::put()`.
- `clearModelCache()` to clear cache when updating a model definition.
- `setCacheEnabled()`, `setCacheTtl()` for programmatic control.

#### 3. Flexible Registration Methods

- `registerDefinition($modelClass, array $definition)`: Register a raw definition.
- `registerDefinitions(array $definitions)`: Register multiple definitions.
- `registerCallback(callable $callback)`: Register a callback to be executed at load time (e.g., like `ReportsManager`).
- The `registerModel()` method remains for backward compatibility.

#### 4. Definition Normalization

- `normalizeModelDefinition()`: Merge default settings.
- `normalizeFieldDefinition()`: Apply default settings based on field type.
- `getDefaultConfigForType()`: Fetch settings from the configuration file.

#### 5. Improved Query Methods

- `isModelRegistered()` automatically calls `getRegisteredModels()` to load definitions.
- `getModelConfig()` uses `buildModel()` to build the model when needed.

### Benefits

- Better performance thanks to caching (avoid re‑processing definitions on every request).
- Greater flexibility in registering models (via raw definitions or callbacks).
- Cleaner, more maintainable code structure.

---

## Version 1.0.4 – Enhanced Security, Logging, and Exceptions

### Version Objectives

- Permanently block dangerous files (PHP, JS, HTML, EXE, etc.) even if they are in `allowed_types`.
- Create a separate log channel for recording failed upload attempts.
- Create a custom exception class `FileUploadException` with unique error codes.
- Improve error messages and make them translatable (foundation for localization).

### New Features

#### 1. Blacklist for Dangerous Files

We added a constant `BLACKLISTED_EXTENSIONS` in `FileUploadService` containing dangerous extensions such as:
`php, js, html, exe, bat, sh, dll, ...`

And a method `isBlacklistedExtension()` that checks the extension before any other validation. If the extension is blacklisted, the file is immediately rejected with an appropriate message.

#### 2. Logging Failed Upload Attempts

- Added a new log channel `fileupload` in `config/logging.php` (configured in `Plugin::boot()`).
- The `logFailedAttempt()` method logs the following details:
  - User (ID, type, IP, User Agent)
  - Model and field
  - Reason for failure (size, type, blacklist, permission)
  - File information (name, size, type, extension)
- This method is called whenever an error occurs in `validateFile()`.

#### 3. `FileUploadException` Class

A custom exception class inheriting from `Exception` was created, providing:
- Numeric error codes (1000-1999 general, 2000-2999 for files, 3000-3999 for permissions, 4000-4999 for temporary keys).
- Generation of a unique textual error code (e.g., `FILE_UPLOAD_FILE_SIZE_EXCEEDED`).
- Ability to add context to the error (e.g., allowed size, uploaded extension).

#### 4. Updated `FileUploadService::validateFile()` to include:

- Blacklist check first.
- Size and allowed type checks (as before).
- Throwing `FileUploadException` instead of `ApplicationException`.
- Calling `logFailedAttempt()` before throwing the exception.

#### 5. Updated `FileUploadController`:

- The `handleException()` method recognises `FileUploadException` and extracts the `errorCode`.
- The `errorResponse()` method accepts an `$errorCode` parameter and adds it to the response.

### Benefits

- Higher security by blocking dangerous files even if `allowed_types` is misconfigured.
- Ability to track attacks or issues through a separate log.
- Unified error handling with unique codes to facilitate frontend handling.

---

## Version 1.0.5 – Full Multilingual Support

### Version Objectives

- Make all API messages (success, error) translatable into Arabic and English.
- Add translation keys for every operation (`upload`, `uploadMultiple`, `delete`, `getFiles`).
- Update the controller and service to use `trans()` instead of hard‑coded strings.

### New Features

#### 1. Added Translation Keys to `lang.php`

We added a new section `public.helpers.upload` containing keys such as:
- `msg_upload_success`
- `msg_upload_multiple_success`
- `msg_delete_success`
- `msg_get_success`
- `msg_permission_denied`
- `msg_validation_failed`
- `msg_upload_disabled`, `msg_delete_disabled`, `msg_get_disabled`
- and other common error messages.

We also added keys in the `errors` section for detailed messages such as `file_size_exceeded`, `file_type_not_allowed`, `file_type_blacklisted`.

#### 2. Updated `FileUploadController`

- The `successResponse()` and `errorResponse()` methods use `trans()` with default values.
- The `upload()`, `uploadMultiple()`, `delete()`, `getFiles()` methods use `trans()` for every message (success or failure).
- The `getSafeErrorMessage()` method uses `trans()` for generic messages.
- In `uploadMultiple()`, `success_count` and `total` are passed to the translation function.

#### 3. Updated `FileUploadService`

- Replaced all hard‑coded `ApplicationException` messages with `trans()` using the appropriate keys.
- In `validateFile()`, `trans()` is used in `FileUploadException` messages, passing parameters (e.g., `max_size`, `actual_size`, `extension`, `allowed`).

#### 4. Support for Arabic and English

- Translation files are located in `lang/ar/lang.php` and `lang/en/lang.php`.
- Users can switch language via `app.locale`.

### Benefits

- Improved user experience for Arabic and English speakers.
- Easy addition of new languages in the future.
- All messages are centralised in translation files, facilitating maintenance and modification.

---

## Version Summary (1.0.2 – 1.0.5)

| Version | Key Features |
|---------|---------------|
| 1.0.2 | Global API control, default file type settings, enhanced temporary key security. |
| 1.0.3 | Refactored `FileUploadRegistry`, caching support, flexible registration via definitions and callbacks. |
| 1.0.4 | Blacklist for dangerous files, logging of failed upload attempts, custom exception class. |
| 1.0.5 | Full multilingual support (Arabic/English) for all API messages. |

---

## Conclusion

Thanks to these updates, the `Nano.FileUpload` plugin has become more secure, performant, and flexible. It now provides:
- Fine‑grained permission control at multiple levels.
- Smart default settings that reduce code duplication.
- High security for temporary files and protection against dangerous files.
- Comprehensive error logging to monitor malicious attempts.
- A unified API with translatable messages.

These improvements make the plugin ready for use in the largest projects, with the ability to scale in the future.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)
