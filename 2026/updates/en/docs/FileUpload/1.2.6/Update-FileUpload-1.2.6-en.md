## 2026-04-18 – 2026-04-19

**Nano.FileUpload Plugin Update – Version 1.2.6**

### Fix for Double-Deletion Issue When Uploading to `AttachOne` Relations

---

### Summary of Updates

Version **1.2.6** addresses a critical bug that caused newly uploaded files to be deleted immediately when uploaded to fields of type `AttachOne` (e.g., cover image, profile picture, etc.). The issue stemmed from a double invocation of the file attachment process (`$relation->add()`), which resulted in a `DELETE` query being executed right after the `INSERT`, causing the file record to vanish from the database despite a seemingly successful upload.

The fix involves:

1. Adding fine-grained control options to the `Base64::onUpload` method to manage the setting of fields (`field`, `attachment_id`, `attachment_type`) and the execution of the relation binding (`skip_relation_add`).
2. Modifying `FileUploadService::upload` to utilize these new options, so that the file is saved without being attached within `Base64`, and then attached exactly once after processing the file and deleting any old file (if applicable).
3. Eliminating the duplicate call to `add()`, thereby removing the unwanted `DELETE` queries.

---

### Objectives of This Release

- **Fix a critical bug** that prevented files from being correctly uploaded to `AttachOne` relations (especially in testing environments with nested transactions).
- **Prevent the unintended deletion of newly uploaded files** that occurred immediately after insertion.
- **Improve the stability and reliability** of the upload system, particularly for fields that accept only a single file.
- **Provide finer-grained control** over the behavior of `Base64::onUpload` through new options that allow skipping the assignment of specific fields or skipping the attachment process entirely.
- **Ensure the success of automated tests** related to file uploads (`FILE_CREATE_001`, `FILE_UPDATE_001`), which were consistently failing due to this bug.

---

### The Original Problem (Before the Fix)

#### Description of the Erroneous Behavior

When uploading a new file to an `AttachOne` field (e.g., `document_front`), the system performed the following steps:

1. **`Base64::onUpload`**: Saved the file to the `system_files` table **while setting `attachment_id` and `attachment_type`**, then called `$fileRelation->add($file)` to attach it to the model.
2. **`FileUploadService::upload`**: After calling `onUpload`, it **again** called `$relation->add($newFile)`.

**Result:** The second invocation of `add()` would delete all files associated with the field (including the newly attached file) and then re-attach it. However, in certain scenarios (especially with nested transactions), the `DELETE` query would permanently remove the record without properly re-inserting it, causing the file to disappear from the database.

#### Observed SQL Queries (Evidence of the Problem)

```
INSERT INTO `system_files` (...) VALUES (...)  -- New file inserted (ID=7556)
DELETE FROM `system_files` WHERE `attachment_id` = ... AND `field` = ...  -- New file deleted!
```

Outcome of this behavior: `file_exists_in_db = false` and failure of file upload tests.

---

### Solutions Implemented in Version 1.2.6

#### 1. Added New Control Options to `Base64::onUpload`

Four new options were added to the `$options` array in the `onUpload` method:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `skip_set_field` | `bool` | `false` | If `true`, the `field` attribute is not set on the `File` model. |
| `skip_set_attachment_type` | `bool` | `false` | If `true`, the `attachment_type` attribute is not set. |
| `skip_set_attachment_id` | `bool` | `false` | If `true`, the `attachment_id` attribute is not set. |
| `skip_relation_add` | `bool` | `false` | If `true`, `$fileRelation->add()` is not called (i.e., the file is not attached). |

**Code added to `onUpload`:**

```php
$d_options = array_merge([
    // ... previous options
    'skip_set_field' => false,
    'skip_set_attachment_type' => false,
    'skip_set_attachment_id' => false,
    'skip_relation_add' => false,
], $options);

// ...

if (!$skip_set_field && $field) {
    $file->field = $field;
}
if (!$skip_set_attachment_type && $attachment_type) {
    $file->attachment_type = $attachment_type;
}
if (!$skip_set_attachment_id && $attachment_id) {
    $file->attachment_id = $attachment_id;
}

// ...

if (!$skip_relation_add && is_object($fileRelation)) {
    // ... perform attachment
}
```

#### 2. Modified `FileUploadService::upload` to Use the New Options

The `$uploadOptions` passed to `Base64::onUpload` were updated:

```php
$uploadOptions = [
    // ... previous options
    'skip_set_field'           => false,
    'skip_set_attachment_type' => false,
    'skip_set_attachment_id'   => true,   // ⬅️ Do not set attachment_id in Base64
    'skip_relation_add'        => true,   // ⬅️ Do not attach in Base64
];
```

After calling `onUpload` and obtaining `$newFile`, the following steps are performed **within `FileUploadService`**:

1. Set `field` if not already set.
2. Calculate `hash` and `meta`, then save changes.
3. Delete the old file (`$existingFile`) if the operation is `edit`.
4. **Attach the new file exactly once** using `$relation->add($newFile)`.

**Final code in `FileUploadService::upload`:**

```php
Db::transaction(function () use (...) {
    $uploadResult = Base64::onUpload($uploadOptions, $uploadedFile);
    $newFile = $uploadResult['model'];

    // Set additional fields and save
    if (!$newFile->field) {
        $newFile->field = $field;
    }
    // ... calculate hash, meta, disk ...

    $newFile->save();

    // Delete old file if this is an edit operation
    if ($operation === 'edit' && $existingFile) {
        $existingFile->delete();
    }

    // Attach the new file exactly once
    if ($model && $model->exists && $model->hasRelation($field)) {
        $relation = $model->{$field}();
        if ($relation instanceof \October\Rain\Database\Relations\AttachOne) {
            $relation->add($newFile);
        }
    }
});
```

---

### Results After the Fix

- **Elimination of the unwanted `DELETE` query** that followed the `INSERT`.
- **The file record remains in the database** (`file_exists_in_db = true`).
- **Reliable success of `FILE_CREATE_001` and `FILE_UPDATE_001` tests**.
- **No need for workarounds in tests** (such as `savepoint` or manual cleanup).

---

### Upgrade Requirements (from 1.2.5 to 1.2.6)

1. **Code Update**:
   - Replace `Base64.php` with the version containing the new options (`skip_set_field`, `skip_set_attachment_type`, `skip_set_attachment_id`, `skip_relation_add`).
   - Replace `FileUploadService.php` with the version that utilizes these options and performs attachment only once.

2. **No New Migrations** – No database changes are required.

3. **No Configuration Changes** – The `config.php` file remains unchanged.

4. **No API Endpoint Changes** – All endpoints continue to function as before without modification.

5. **Compatibility Testing**:
   - It is recommended to run the `KycPlusTest` (or `FileUploadPlusTest`) suite to verify the success of upload and update operations.
   - Any errors related to `FILE_CREATE_001` or `FILE_UPDATE_001` should now be resolved.

---

### Benefits and Added Value

- **Fixes a critical bug** that affected any `AttachOne` field (e.g., avatars, cover images, single-file uploads).
- **Increases system reliability**, especially in testing environments with nested transactions.
- **Improves code quality** by separating the responsibilities of `Base64` (file saving only) from `FileUploadService` (full control over the upload and attachment lifecycle).
- **Provides fine-grained control options** that can be used in other advanced scenarios.
- **Paves the way for future releases** that may add features like pre-uploading files without immediate attachment.

---

### Conclusion

Version **1.2.6** delivers a fundamental fix for an issue that negatively impacted the stability of the file upload system and the reliability of automated tests. By restructuring the attachment logic between `Base64` and `FileUploadService` and introducing new control options, the upload process for single-file fields (`AttachOne`) is now more reliable and less error-prone. These enhancements strengthen the plugin and make it suitable for use in both production and development environments.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload-en.md)
- [FileUploadService Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class-en.md)
- [Base64 Class Documentation](./docs/FileUpload/Docs-Base64-Class-en.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation-en.md)
