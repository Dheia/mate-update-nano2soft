## 2026-04-07 – 2026-04-10

**`Nano.FileUpload` Plugin Updates – Versions 1.0.8 to 1.2.0**

### Update Summary

After previous releases that focused on multiple storage support, automatic image transformation, and event hooks, we continued developing the add-on to enhance security, flexibility, and add advanced file management features. Updates in versions 1.0.8 through 1.2.0 included:

- **Addition of a comprehensive testing suite** to verify all add-on classes.
- **Addition of `session_key` column** to the `system_files` table to support temporary uploads and deferred binding.
- **Support for Edit operation** to replace files in `attachOne` relations with custom permissions.
- **Addition of field-level update functions** in `FileUploadRegistry`.
- **Performance and reliability improvements** for upload and delete operations using transactions and new events.
- **Restructuring of the test class** into `FileUploadPlusTest` with unified output format and advanced exception handling.

---

### Version 1.0.8 – Comprehensive Testing System and Improved Delete Security

#### Release Goals

- Create an integrated test class (`FileUploadTest`) covering all add-on classes (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager`).
- Improve security of the `deleteFile` function to automatically extract `modelClass` and `field` from the file record if not provided.
- Integrate tests into `Plugin.php` so they can be run via a `test_fileupload` parameter in development environment.

#### New Features

##### 1. `FileUploadTest` Class for Comprehensive Testing

A test class was created located at `Nano\FileUpload\Tests\FileUploadTest` containing functions to test:

- **FileUploadRegistry**: model registration, getting field config, model existence check, defaults, processing options, model update, global API operation controls, and caching mechanism.
- **FileUploadUserManager**: getting user, determining user type, permission checking, customizing user resolver, and customizing permission checker function.
- **FileUploadService**: generating temporary keys, validating them, direct file upload, base64 upload, multiple upload, size/type/blacklist validation, temporary upload, attaching temporary files, retrieving files, deleting files, automatic resizing, watermarking, custom storage disk, `hash` and `meta` calculation, temporary file expiry, events, WebSocket, and globally disabling operations.

The test uses an isolated environment (`Storage::fake('local')`) and a dummy model (`TestModel`) to ensure no effect on real data.

##### 2. Improved Security of `deleteFile` Function

The `deleteFile` function now handles three cases for permission checking:

- **Case 1:** `modelClass` and `field` are provided – permission is checked using them.
- **Case 2:** File is attached to a saved model (`attachment_type` and `attachment_id`) – `modelClass` and `field` are extracted from the file record and permission is checked.
- **Case 3:** File is temporary (`session_key`) – the key is validated and that the current user is the one who created the temporary file.

This ensures no user can delete a file they do not have permission for, even if they do not explicitly pass `modelClass` and `field`.

##### 3. Integration of Tests into `Plugin.php`

We added a `testFileUpload()` function in `Plugin.php` that is called when the `is_testFileUpload=true` parameter is present in the request. It creates an instance of `FileUploadTest`, runs all tests, and displays results. This allows developers to easily run tests via browser.

#### Benefits

- Ensure code quality through automated testing of all core functions.
- Early detection of bugs during development.
- Living documentation of class behavior via tests.
- Increased confidence in add-on stability when adding new features.

---

### Version 1.0.9 – Support for `session_key` in Files Table

#### Release Goals

- Add a `session_key` column to the `system_files` table to fully support deferred binding.
- Enable filtering of temporary files belonging to a specific user session.

#### New Features

##### 1. Migration `add_session_key_to_system_files`

A migration file was created that adds the `session_key` column of type `string` and `nullable` with an index (after the `field` column). This column is used to store the temporary session key when a file is uploaded without being attached to a saved model.

##### 2. Update `FileUploadService::upload`

During upload, if a temporary key (`tempSessionKey`) is used, `$file->session_key = $tempSessionKey` is set before saving the file. This ensures temporary files can later be found using this key.

##### 3. Update `FileUploadService::attachTempFiles`

The `attachTempFiles` function now relies on the `session_key` column to find temporary files to attach to the saved model. After attachment, `session_key` is cleared and `attachment_id` and `attachment_type` are set.

##### 4. Update `FileUploadService::deleteFile`

In the case of a temporary file, the `session_key` is checked to ensure the current user is the one who created the file before allowing deletion.

#### Benefits

- Full support for temporary upload and deferred binding.
- Ability to clean up expired temporary files via scheduled task using `expires_at` and `session_key`.
- Higher security: cannot access temporary files belonging to another user.

---

### Version 1.1.0 – Support for Edit Operation and File Replacement

#### Release Goals

- Enable replacing an existing file in an `attachOne` relation by uploading a new file (edit operation).
- Add a global `disable_edit` setting to enable/disable file replacement at the API level.
- Add `updateFieldConfig` function in `FileUploadRegistry` to update a specific field's settings with normalization and cache clearing.
- Improve `updateModelConfig` to normalize the entire model after merging.
- Move the `Base64` class from `Nano.API` into the add-on to reduce dependencies.
- Add new events `beforeEditDelete` and `afterEditDelete` for the edit lifecycle.
- Improve `attachTempFiles` function to return a unified response.

#### New Features

##### 1. Determine Upload Operation (add/edit)

In `FileUploadService::upload`, it checks if the model exists and has an `attachOne` relation for the field. If a file is already attached, `$operation = 'edit'` is set; otherwise `'add'`.

##### 2. Permission Check for `edit` Operation

We added in `FileUploadRegistry`:

- `isEditEnabledGlobally()` function that checks `disable_edit` in global settings.
- `can()` function now supports the `'edit'` operation and checks the global setting and field/model permissions.
- In model and field default definitions, we added the `'edit' => null` key in the `permissions` array.

##### 3. Use Transaction in `upload` Operation

The upload operation is now wrapped in `Db::transaction` to ensure integrity:

- The new file is uploaded and saved first.
- If the operation is `edit` and the upload succeeds, the old file is deleted.
- Then the new file is attached to the model relation.
- If any part fails, all changes are rolled back (including deletion of the old file).

This prevents loss of the old file if the new file upload fails.

##### 4. New Events for Edit Operation

- `nano.fileupload.beforeEditDelete`: Dispatched before deleting the old file (when `operation === 'edit'`).
- `nano.fileupload.afterEditDelete`: Dispatched after deleting the old file.

This allows developers to execute custom logic (e.g., send notification, log, or backup) before and after replacing the file.

##### 5. `updateFieldConfig` Function in `FileUploadRegistry`

This function provides an updated way to modify settings of a specific field without re-registering the entire model. It:

- Checks existence of model and field.
- Merges new settings with existing using `array_replace_recursive`.
- Normalizes the field via `normalizeFieldDefinition`.
- Updates `rawDefinitions` and resets built model (`builtModels`).
- Clears cache for this specific field via `clearFieldCache`.

##### 6. Improved `updateModelConfig`

The function now re-normalizes the entire model after merging new settings, ensuring all default fields (e.g., `is_public`, `thumb_options`) are complete.

##### 7. Move `Base64` Class into the Plugin

The `Base64` class was moved from `Nano\API\Classes\Base64` to `Nano\FileUpload\Classes\Base64` with advanced functions for handling file sizes (`normalizeUploadFileSize`, `getUploadMaxFilesize`, `checkUploadMaxFilesize`). This removes the dependency on `Nano.API` and makes the add-on independent.

##### 8. Improved `attachTempFiles`

The function now returns a unified array (like `upload`) containing `code`, `status`, `message`, `data`, `process_data`, `error`, `debug`, and accepts the `skip_permission_check` option to bypass permission checks in tests. It also adds an `afterAttach` event and WebSocket notification.

#### Benefits

- Ability to safely replace files in `attachOne` relations (using transactions).
- Fine-grained control over edit permission via global setting or custom permissions.
- More flexibility in updating field settings without re-registering the model.
- Reduced dependencies by moving `Base64` into the add-on.
- Unified output for `attachTempFiles` with other service functions.

---

### Version 1.2.0 – Testing System Overhaul

#### Release Goals

- Completely rewrite the test class into `FileUploadPlusTest` with a unified output structure.
- Unify each test's output to include: `code`, `status`, `test_code`, `name`, `description`, `message`, `error`, `errors`, `data`, `input_data`, `process_data`, `debug`.
- Add advanced exception handling so that one test failure does not stop the execution of other tests.
- Use transactions (`Db::beginTransaction()`, `Db::rollBack()`) in all tests that modify the database.
- Remove any dependency on external testing packages (e.g., `assert`).
- Add new tests for the edit operation (`testEditReplaceFile`, `testEditWithoutPermission`, `testEditWithGlobalDisable`, `testEditTriggersEvents`, `testEditKeepsOldFileOnFailure`).

#### New Features

##### 1. Unifying Test Outputs

We added a `recordResult(array $result)` function that accepts an array with the required structure and merges it with default values. This ensures all test results are consistent and easy to process automatically.

Example of a successful test output:
```json
{
    "code": 200,
    "status": true,
    "test_code": "UPLOAD_DIRECT_001",
    "name": "Direct file upload (UploadedFile) with temporary key",
    "description": "Checks uploading an image file using UploadedFile object...",
    "message": "File uploaded successfully",
    "error": null,
    "errors": [],
    "data": { ... },
    "input_data": { ... },
    "process_data": { ... },
    "debug": null
}
```

##### 2. Exception Handling

Each test is wrapped in `try-catch`. In case of an exception, `status` is set to `false`, `code` to `500`, the error message is stored in `error`, and debug details in `debug` (if `app.debug` is enabled). This ensures other tests continue to run.

##### 3. Using Transactions in Tests

All tests that make changes to the database (upload, delete, update, attach) use `Db::beginTransaction()` and `Db::rollBack()` in a `finally` block. This ensures no leftover data after tests finish.

##### 4. Adding Edit Operation Tests

We added 5 new tests to cover file replacement scenarios:

- `testEditReplaceFile`: Test successful replacement of a file in an `attachOne` relation.
- `testEditWithoutPermission`: Test rejection of replacement when `edit` permission is missing.
- `testEditWithGlobalDisable`: Test rejection of replacement when editing is globally disabled (`disable_edit = true`).
- `testEditTriggersEvents`: Test that `beforeEditDelete` and `afterEditDelete` events are dispatched.
- `testEditKeepsOldFileOnFailure`: Test that the old file is not deleted when the new file upload fails (thanks to transaction).

##### 5. Restructuring Existing Tests

All existing tests (43 tests) were rewritten to conform to the new structure. A unique `test_code` was added for each test to facilitate identification.

##### 6. Update `FileUploadController::tests()`

The controller now supports running both versions of tests (`FileUploadTest` for old version and `FileUploadPlusTest` for new version) via the `test_version=v1` or `test_version=v2` parameter (default is `v2`). This allows gradual migration.

#### Benefits

- Unified and easy-to-read test results.
- Ability to integrate tests into CI/CD systems easily.
- Complete isolation of tests from real data.
- Full coverage of edit operations added in version 1.1.0.
- Ease of adding new tests in the future by following the same pattern.

---

### Version Summary (1.0.8 – 1.2.0)

| Version | Key Features |
|---------|---------------|
| 1.0.8 | Comprehensive testing system (`FileUploadTest`), improved `deleteFile` security, test integration into `Plugin.php`. |
| 1.0.9 | Add `session_key` column to `system_files` to fully support temporary uploads. |
| 1.1.0 | Support for file replacement (edit), add `disable_edit`, `updateFieldConfig` function, improve `updateModelConfig`, move `Base64` into add-on, `beforeEditDelete/afterEditDelete` events, improve `attachTempFiles`. |
| 1.2.0 | Restructure testing system into `FileUploadPlusTest` with unified outputs, advanced exception handling, use of transactions, and addition of edit tests. |

---

### Upgrade Requirements

1. **Run new migrations** (to add `session_key` column):
   ```bash
   php artisan plugin:refresh Nano.FileUpload
   ```
   Or manually execute the `add_session_key_to_system_files.php` migration.

2. **Add environment variable** (optional) to control edit operation:
   ```ini
   NANO_FILE_UPLOAD_DISABLE_EDIT=false
   ```

3. **Update field definitions** to add `edit` permission if needed:
   ```php
   'permissions' => [
       'add'    => 'some.permission',
       'edit'   => 'some.edit.permission', // new
       'delete' => 'some.delete.permission',
       'view'   => 'some.view.permission',
   ]
   ```

4. **Listen to new events** (optional):
   ```php
   Event::listen('nano.fileupload.beforeEditDelete', function ($file, $modelClass, $field, $model) {
       // Backup or log
   });
   ```

5. **Run tests** (in development environment):
   - Via browser: `https://yourdomain.com/api/v1/fileupload/tests?test_version=v2`
   - Or via command line (after adding a custom command).

---

### Conclusion

Thanks to versions 1.0.8 through 1.2.0, the `Nano.FileUpload` add-on is more mature and reliable than ever. It can now:

- **Test itself automatically** through an integrated test system compatible with CI/CD.
- **Support file replacement** in `attachOne` relations safely using transactions and custom events.
- **Update field settings flexibly** without redefining the entire model.
- **Fully manage temporary files** via the `session_key` column.
- **Provide unified test results** to facilitate debugging and integration with external tools.

These improvements make the add-on ready for use in the largest and most complex projects, ensuring stability, security, and ease of maintenance.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService` Class](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)
- 