## 2026-04-05- 2026-04-07

**Updates to `Nano.FileUpload` Plugin – Versions 1.0.6 and 1.0.7**

### Summary of Updates

After releasing version 1.0.5, which focused on multi-language support, we continued developing the add-on by adding advanced features aimed at increasing flexibility, scalability, and improving file management at the database level. Versions 1.0.6 and 1.0.7 included:

- Support for multi-storage via different disks (S3, FTP, Local).
- Automatic image transformation (automatic resizing and watermarking) with global control via settings.
- Event hooks before and after upload and delete operations.
- Optional WebSocket notifications.
- Adding new columns to the `system_files` table: `disk`, `hash`, `meta`, `expires_at`.
- Automatic population of the new fields (SHA256 of content, image dimensions, expiration date for temporary files).
- General performance and security improvements.

---

## Version 1.0.6 – Multi-Storage, Automatic Image Transformation, Event Hooks, and Database Enhancements

### Version Goals

- Enable uploading files to multiple storage services (AWS S3, FTP, Local) by specifying the storage disk at the field level.
- Add the ability to automatically transform images (resizing and watermarking) during upload.
- Provide event hooks to extend behavior without modifying the core code.
- Support WebSocket notifications for real-time updates in frontends.
- Add new columns to the `system_files` table to support advanced features (storage, deduplication, metadata, expiration).

### New Features

#### 1. Multi-Storage Support

We added the ability to specify a custom storage disk per field via the `storage_disk` setting in the field definition. This overrides the default disk set in the system configuration.

- **Added `disk` column** to the `system_files` table to store the name of the disk used.
- **Modified `System\Models\File` model** (via `Plugin::extendSystemFile()`) to support the `disk` property and override the `getDisk()` method to use the custom disk.
- **Added `getStorageDiskForField()` method** in `FileUploadService` to retrieve the disk name from field settings and check its existence.
- During upload, `$file->disk = $diskName` is set and then the file is saved on the specified disk.

#### 2. Automatic Image Transformation (Auto-resize & Auto-watermark)

We added new settings in the field definition:
- `auto_resize` (bool): enable automatic image resizing.
- `resize_options` (array): resize options (width, height, mode).
- `auto_watermark` (bool): enable automatic watermarking.
- `watermark_options` (array): watermark options (position, resize percentage).

We added the `applyAutoProcessing()` method in `FileUploadService` which:
- Checks if the file is an image.
- Uses `October\Rain\Resize\Resizer` to resize the image according to the options.
- Uses the `Nano2.Watermark` add-on (if installed and enabled) to add the watermark.

#### 3. Event Hooks

We added the following events to easily extend behavior:

| Event | Call Location | Parameters |
|-------|---------------|------------|
| `nano.fileupload.beforeUpload` | Before starting upload processing | `$modelClass, $field, $fileData, &$options` |
| `nano.fileupload.afterUpload` | After successful upload and before returning the result | `$file, $modelClass, $field, $options` |
| `nano.fileupload.beforeDelete` | Before deleting the file | `$fileId, $modelClass, $field` |
| `nano.fileupload.afterDelete` | After deleting the file | `$fileId, $modelClass, $field` |

Developers can listen to these events to add custom logic (e.g., sending notifications, additional logging, modifying data).

#### 4. WebSocket Support for Instant Notifications

We added `websocket` settings in `config.php`:
- `enabled`: enable/disable notifications.
- `channel`: the channel name used.

We added the `notifyWebSocket()` method in `FileUploadService` which dispatches the `nano.fileupload.websocket.notify` event after a successful upload or deletion. Developers can use a WebSocket library (e.g., Laravel WebSockets or Pusher) to catch this event and send notifications.

#### 5. Adding New Columns to the `system_files` Table

We created a migration `add_columns_to_system_files.php` to add the following columns:

| Column | Type | Purpose |
|--------|------|---------|
| `disk` | `string, nullable` | Store the name of the custom storage disk (e.g., `s3`, `ftp`). |
| `hash` | `string, nullable, index` | Store SHA256 of the content to prevent duplicates. |
| `meta` | `text, nullable` | Store additional data as JSON (image dimensions, audio duration, etc.). |
| `expires_at` | `dateTime, nullable` | Expiration date for temporary files to be automatically cleaned up. |

### Benefits

- Ability to store files on multiple cloud services as needed.
- Improved user experience by automatically resizing images and adding watermarks without manual intervention.
- Easy system extension via events without modifying core code.
- Instant notifications to users when upload or deletion completes.
- Better management of duplicate files, metadata, and temporary files.

---

## Version 1.0.7 – Completing Automatic Population of `hash`, `meta`, `expires_at` Fields and Adding Global Control over Transformations

### Version Goals

- Complete the implementation of automatically populating the new fields (`hash`, `meta`, `expires_at`) during upload.
- Add global settings to enable/disable automatic transformations (`auto_resize`, `auto_watermark`) at the entire API level.
- Improve performance and security by calculating hashes, storing image dimensions, and setting expiration for temporary files.

### New Features

#### 1. Automatic Population of `hash` (SHA256)

We added the following inside the `upload()` method after obtaining the file object:
```php
if (!$file->hash) {
    $content = file_get_contents($file->getLocalPath());
    $file->hash = hash('sha256', $content);
    $is_changed = true;
}
```
This ensures a unique hash of the content is calculated, helping to prevent storing duplicate copies (the `hash` can be checked before saving).

#### 2. Automatic Population of `meta` with Image Dimensions

We added:
```php
if ($file->isImage()) {
    $dimensions = getimagesize($file->getLocalPath());
    $file->meta = array_merge($file->meta ?: [], [
        'width'  => $dimensions[0],
        'height' => $dimensions[1],
        'mime'   => $dimensions['mime'],
    ]);
    $is_changed = true;
}
```
Dimensions and MIME type are stored in the `meta` field as JSON, making it easy to retrieve them later without needing to read the file again.

#### 3. Setting `expires_at` for Temporary Files

When a `temp_session_key` is used (i.e., the file is not yet linked to a model), we automatically set an expiration date:
```php
if ($tempSessionKey) {
    if (!$file->expires_at && (!$file->attachment_type || !$file->attachment_id)) {
        $file->expires_at = Carbon::now()->addHours(24);
        $is_changed = true;
    }
}
```
A scheduled task (cron) can be run to automatically delete expired and unlinked files.

#### 4. Adding Global Control over Automatic Transformations

We added two settings in the `general` section of `config.php`:
```php
'disable_auto_resize'    => env('NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE', true),
'disable_auto_watermark' => env('NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK', true),
```
And two methods in `FileUploadRegistry`:
- `isAutoResizeEnabledGlobally()`
- `isAutoWatermarkEnabledGlobally()`

Then we modified the `applyAutoProcessing()` method to respect these settings:
```php
if ($this->registry->isAutoResizeEnabledGlobally() && $procOptions['auto_resize']) { ... }
if ($this->registry->isAutoWatermarkEnabledGlobally() && $procOptions['auto_watermark'] && ...) { ... }
```
This allows the administrator to temporarily disable all resizing or watermarking operations via environment variables without modifying field definitions.

#### 5. Other Improvements

- Use `Carbon::now()` instead of `now()` to ensure compatibility.
- Fixed variable name `$is_changed` (instead of `$is_chage`).
- Ensure the file is saved again when any of the fields (`hash`, `meta`, `expires_at`) are changed.

### Benefits

- Better management of duplicate files via `hash` checking.
- Store useful metadata (image dimensions) for use in frontends.
- Automatic cleanup of temporary files via `expires_at`.
- Centralized control over enabling/disabling automatic transformations.

---

## Version Summary (1.0.6 and 1.0.7)

| Version | Key Features |
|---------|---------------|
| 1.0.6 | Multi-storage support (disk), automatic image transformation, event hooks, WebSocket, adding new columns to database. |
| 1.0.7 | Automatic population of `hash`, `meta`, `expires_at`, global control over automatic transformations, performance and security improvements. |

---

## Upgrade Requirements

1. **Run the migration**:
   ```bash
   php artisan plugin:refresh Nano.FileUpload
   ```
   Or execute the new migration `add_columns_to_system_files.php`.

2. **Add environment variables** (optional) in your `.env` file:
   ```ini
   # Disable automatic transformations (default values: true)
   NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
   NANO_FILE_UPLOAD_DISABLE_AUTO_WATERMARK=true

   # WebSocket settings
   NANO_FILE_UPLOAD_WEBSOCKET_ENABLED=false
   NANO_FILE_UPLOAD_WEBSOCKET_CHANNEL=file-uploads

   # Multi-storage settings (at field level, not global)
   # (set directly in field definitions)
   ```

3. **Update field definitions** to add `storage_disk`, `auto_resize`, `auto_watermark` as needed.

4. **Listen to events** to extend behavior:
   ```php
   Event::listen('nano.fileupload.afterUpload', function ($file, $modelClass, $field, $options) {
       // Send WebSocket notification
   });
   ```

---

## Conclusion

Thanks to versions 1.0.6 and 1.0.7, the `Nano.FileUpload` add-on is now more powerful and flexible than ever. It can now:

- Handle multiple storage disks (S3, FTP, Local).
- Automatically transform images (resizing, watermarking).
- Dispatch custom events to extend behavior.
- Send instant notifications via WebSocket.
- Store rich metadata (image dimensions, hashes).
- Automatically clean up temporary files.

These features make the add-on ready for use in large, complex projects that demand high performance and advanced security.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/FileUpload/Docs-FileUpload.md)
- [`FileUploadRegistry` Class Documentation](./docs/FileUpload/Docs-FileUploadRegistry-Class.md)
- [Advanced Examples for `FileUploadRegistry` Class](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples.md)
- [`FileUploadService` Class Documentation](./docs/FileUpload/Docs-FileUploadService-Class.md)
- [Advanced Examples for `FileUploadService` Class](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples.md)
- [`FileUploadUserManager` Class Documentation](./docs/FileUpload/Docs-FileUploadUserManager-Class.md)
- [Advanced Examples for `FileUploadUserManager` Class](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples.md)
- [API Documentation](./docs/FileUpload/Docs-API-Documentation.md)
