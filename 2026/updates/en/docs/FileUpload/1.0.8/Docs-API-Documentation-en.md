# File Upload API Documentation

**Version:** 1.0.7  
**Base Path:** `/api/v1/fileupload`  
**Authentication:** OAuth 2.0 (access token in header `Authorization: Bearer <token>`)

---

## Overview

This documentation is intended for external developers (web stores, e-commerce applications, order delivery apps, and integrated systems) who wish to integrate the file upload service into their applications. The API provides secure and easy-to-use endpoints for uploading single or multiple files, deleting files, and retrieving files associated with specific models (e.g., products, orders, users).

**Key Features:**
- Support for uploading files in multiple formats (multipart/form-data or base64).
- Temporary file upload before saving the model (using temporary session keys).
- Automatic image conversion (resizing, watermarking) according to system settings.
- Storage of files on multiple disks (local, S3, FTP).
- Standardized responses with unique error codes for easy programmatic handling.
- Full security via OAuth 2.0 and user permission verification.

---

## Standard Response Structure

Every API response follows this structure:

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "error": null,
    "errors": [],
    "data": {},
    "meta": {},
    "input_data": {},
    "process_data": {},
    "debug": null,
    "error_code": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | `int` | HTTP status code (200, 400, 401, 403, 404, 500). |
| `status` | `bool` | `true` for success, `false` for failure. |
| `message` | `string` | Clear user message (translatable). |
| `error` | `string\|null` | Technical error details (on failure). |
| `errors` | `array\|null` | Array of validation errors (e.g., input validity). |
| `data` | `mixed` | Primary data (e.g., uploaded file info). |
| `meta` | `mixed` | Additional data (currently unused). |
| `input_data` | `array` | Copy of data sent by the user (for tracking). |
| `process_data` | `array` | Internal processing information (e.g., temporary session key). |
| `debug` | `array\|null` | Debug information (only shown when `app.debug` is enabled). |
| `error_code` | `string\|null` | Unique error code (e.g., `FILE_UPLOAD_PERMISSION_DENIED`). |

---

## Authentication and Security

All endpoints are protected by **OAuth 2.0**. You must include the access token in the request header:

```
Authorization: Bearer <your_access_token>
```

- If token is not provided: `401 Unauthorized` response.
- If token is invalid or expired: `401 Unauthorized` response.

**Note:** Operation permissions (upload, delete, fetch) depend on the user type (`backend` / `frontend`) and the settings registered for each model and field. Ensure the user has appropriate permissions before calling the API.

---

## Endpoints

### 1. Upload Single File

**Method:** `POST`  
**Path:** `/upload`  
**Content-Type:** `multipart/form-data` or `application/json` (when using base64).

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full model class name (e.g., `Nano\Shop\Models\Product`). |
| `field` | `string` | Yes | Field name as registered in the system (e.g., `image`, `gallery`). |
| `file` | `file` | Conditional* | Uploaded file (multipart format). |
| `file_base64` | `string` | Conditional* | File data in base64 format (alternative to `file`). |
| `temp_session_key` | `string` | No | Temporary session key to link the file later (for unsaved models). |
| `model_id` | `int` | No | Saved model ID (if exists). |

> * Either `file` or `file_base64` must be provided.

#### Request Examples

**multipart/form-data request (direct file upload):**

```http
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="model_class"

Nano\Shop\Models\Product
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

image
------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="product.jpg"
Content-Type: image/jpeg

(binary file data)
------WebKitFormBoundary--
```

**JSON request with base64 (for interfaces that don't support multipart):**

```json
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
    "model_id": 10
}
```

**Request to upload temporary file (before saving the model):**

```json
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,..."
}
```

#### Responses

**✅ Success Response (direct upload and link to existing model):**

```json
{
    "code": 200,
    "status": true,
    "message": "File uploaded successfully",
    "error": null,
    "errors": [],
    "data": {
        "id": 123,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "input_data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "model_id": 10
    },
    "process_data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "temp_session_key": null,
        "user_type": "backend",
        "storage_disk": null
    },
    "debug": null,
    "error_code": null
}
```

**✅ Success Response (temporary upload – will be linked later):**

```json
{
    "code": 200,
    "status": true,
    "message": "File uploaded successfully",
    "data": {
        "id": 124,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "process_data": {
        "temp_session_key": "tmp_YWJjMTIzOnNvbWVoYXNo"
    },
    ...
}
```

> **Important:** Save the `temp_session_key` in the user session or database to use later when linking the file to the saved model.

**❌ Error Response (permission denied):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to upload files for this field",
    "error": "You do not have permission to upload files for this field",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED",
    "data": null
}
```

**❌ Error Response (disallowed file type):**

```json
{
    "code": 400,
    "status": false,
    "message": "File type not allowed. Allowed types: jpg,jpeg,png. Uploaded extension: exe.",
    "error": "File type not allowed. Allowed types: jpg,jpeg,png. Uploaded extension: exe.",
    "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
}
```

**❌ Error Response (file too large):**

```json
{
    "code": 400,
    "status": false,
    "message": "File size exceeds the allowed limit (2048 KB). Actual size: 5120 KB.",
    "error": "File size exceeds the allowed limit (2048 KB). Actual size: 5120 KB.",
    "error_code": "FILE_UPLOAD_FILE_SIZE_EXCEEDED"
}
```

**❌ Error Response (dangerous file – blacklisted):**

```json
{
    "code": 400,
    "status": false,
    "message": "php file type is not allowed for security reasons.",
    "error": "php file type is not allowed for security reasons.",
    "error_code": "FILE_UPLOAD_FILE_TYPE_BLACKLISTED"
}
```

---

### 2. Upload Multiple Files (for multiple field)

**Method:** `POST`  
**Path:** `/upload-multiple`

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full model class name. |
| `field` | `string` | Yes | Name of the multiple field (e.g., `gallery`). |
| `files` | `array` | Yes | Array of files (each element is either an `UploadedFile` object or a base64 string). |
| `temp_session_key` | `string` | No | Temporary session key (for unsaved models). |
| `model_id` | `int` | No | Saved model ID. |

#### Example Request (multipart)

```http
POST /api/v1/fileupload/upload-multiple HTTP/1.1
Authorization: Bearer ...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="model_class"

Nano\Shop\Models\Product
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

gallery
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="img1.jpg"
Content-Type: image/jpeg

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="img2.png"
Content-Type: image/png

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="img3.pdf"
Content-Type: application/pdf

...
------WebKitFormBoundary--
```

#### Responses

**✅ Success Response (all files uploaded successfully):**

```json
{
    "code": 200,
    "status": true,
    "message": "3 of 3 files uploaded successfully",
    "data": [
        {
            "status": true,
            "data": { "id": 101, "path": "...", "thumb": "..." }
        },
        {
            "status": true,
            "data": { "id": 102, "path": "...", "thumb": "..." }
        },
        {
            "status": true,
            "data": { "id": 103, "path": "...", "thumb": "..." }
        }
    ],
    "process_data": {
        "success_count": 3,
        "total": 3
    }
}
```

**⚠️ Partial Success Response (some files failed):**

```json
{
    "code": 200,
    "status": true,
    "message": "2 of 3 files uploaded successfully",
    "data": [
        {
            "status": true,
            "data": { "id": 101, "path": "..." }
        },
        {
            "status": true,
            "data": { "id": 102, "path": "..." }
        },
        {
            "status": false,
            "error": "File type not allowed",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        }
    ],
    "process_data": {
        "success_count": 2,
        "total": 3
    }
}
```

**❌ Complete Failure Response (all files failed):**

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to upload all files",
    "error": "File type not allowed, File size too large",
    "data": [
        {
            "status": false,
            "error": "File type not allowed",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        },
        {
            "status": false,
            "error": "File size exceeds allowed limit",
            "error_code": "FILE_UPLOAD_FILE_SIZE_EXCEEDED"
        }
    ],
    "process_data": {
        "success_count": 0,
        "total": 2
    }
}
```

---

### 3. Delete File

**Method:** `DELETE`  
**Path:** `/delete/{id}`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `int` | Yes | ID of the file to delete. |

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | No | Model name (to verify delete permission). |
| `field` | `string` | No | Field name (to verify delete permission). |

#### Example Request

```http
DELETE /api/v1/fileupload/delete/123?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### Responses

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "File deleted",
    "data": null
}
```

**❌ Error Response (file not found):**

```json
{
    "code": 404,
    "status": false,
    "message": "File not found",
    "error": "File not found",
    "error_code": "FILE_UPLOAD_FILE_NOT_FOUND"
}
```

**❌ Error Response (permission denied – user does not have delete permission):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to delete this file",
    "error": "You do not have permission to delete this file",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

**❌ Error Response (delete operations disabled globally):**

```json
{
    "code": 403,
    "status": false,
    "message": "File delete operations are currently disabled",
    "error": "File delete operations are currently disabled",
    "error_code": "FILE_UPLOAD_DELETE_DISABLED_GLOBALLY"
}
```

---

### 4. Fetch Associated Files

**Method:** `GET`  
**Path:** `/files`

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full model class name. |
| `field` | `string` | Yes | Field name. |
| `model_id` | `int` | No | Saved model ID (to fetch associated files). |
| `temp_session_key` | `string` | No | Temporary session key (to fetch temporary unassociated files). |
| `with_thumbs` | `bool` | No | Include thumbnails (`true` / `false`, default `false`). |
| `thumb_sizes` | `object` | No | Thumbnail sizes (array of size names and dimensions). |

#### Request Examples

**Fetch product gallery images (associated with saved model):**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10 HTTP/1.1
Authorization: Bearer ...
```

**Fetch temporary files (unassociated) using session key:**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=image&temp_session_key=tmp_YWJjMTIzOnNvbWVoYXNo HTTP/1.1
Authorization: Bearer ...
```

**Fetch gallery images with custom thumbnail sizes:**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10&with_thumbs=true&thumb_sizes[small][0]=150&thumb_sizes[small][1]=150&thumb_sizes[small][2]=crop&thumb_sizes[medium][0]=300&thumb_sizes[medium][1]=300 HTTP/1.1
Authorization: Bearer ...
```

#### Responses

**✅ Success Response (associated files):**

```json
{
    "code": 200,
    "status": true,
    "message": "Files retrieved",
    "data": [
        {
            "id": 101,
            "title": null,
            "description": null,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
            "size": 102400,
            "content_type": "image/jpeg"
        },
        {
            "id": 102,
            "title": null,
            "description": null,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../original.png",
            "size": 204800,
            "content_type": "image/png"
        }
    ]
}
```

**✅ Success Response (with thumbnails):**

```json
{
    "code": 200,
    "status": true,
    "message": "Files retrieved",
    "data": [
        {
            "id": 101,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
            "small": "https://yourdomain.com/storage/app/uploads/public/.../small.jpg",
            "medium": "https://yourdomain.com/storage/app/uploads/public/.../medium.jpg"
        }
    ]
}
```

**✅ Success Response (temporary files – after security check):**

```json
{
    "code": 200,
    "status": true,
    "message": "Files retrieved",
    "data": [
        {
            "id": 200,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../temp.jpg"
        }
    ]
}
```

**❌ Error Response (invalid or expired temporary key):**

```json
{
    "code": 400,
    "status": false,
    "message": "Temporary session key is invalid or expired",
    "error": "Temporary session key is invalid or expired",
    "error_code": "FILE_UPLOAD_TEMP_KEY_INVALID"
}
```

**❌ Error Response (temporary key not associated with current user/model):**

```json
{
    "code": 403,
    "status": false,
    "message": "Temporary key is not assigned to this model/field",
    "error": "Temporary key is not assigned to this model/field",
    "error_code": "FILE_UPLOAD_TEMP_KEY_MISMATCH"
}
```

---

## Error Codes – Complete List

| error_code | HTTP Code | Meaning |
|------------|-----------|---------|
| `FILE_UPLOAD_GENERAL` | 500 | General unexpected error. |
| `FILE_UPLOAD_PERMISSION_DENIED` | 403 | User does not have permission for the operation. |
| `FILE_UPLOAD_MODEL_NOT_REGISTERED` | 400 | Model is not registered in the system. |
| `FILE_UPLOAD_FIELD_NOT_REGISTERED` | 400 | Field is not registered for the model. |
| `FILE_UPLOAD_MODEL_NOT_FOUND` | 404 | Model not found (when `model_id` is provided). |
| `FILE_UPLOAD_FILE_NOT_FOUND` | 404 | File not found. |
| `FILE_UPLOAD_INVALID_FILE_DATA` | 400 | Invalid file data (neither file nor base64). |
| `FILE_UPLOAD_FILE_SIZE_EXCEEDED` | 400 | File size exceeds allowed limit. |
| `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` | 400 | File extension not allowed. |
| `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` | 400 | File extension is dangerous (PHP, JS, HTML, etc.). |
| `FILE_UPLOAD_FILE_UPLOAD_FAILED` | 500 | File upload failed (internal error). |
| `FILE_UPLOAD_MAX_FILES_EXCEEDED` | 400 | Number of files exceeds allowed limit (for multiple fields). |
| `FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY` | 403 | Upload operations disabled globally. |
| `FILE_UPLOAD_DELETE_DISABLED_GLOBALLY` | 403 | Delete operations disabled globally. |
| `FILE_UPLOAD_GET_DISABLED_GLOBALLY` | 403 | Fetch operations disabled globally. |
| `FILE_UPLOAD_USER_TYPE_NOT_ALLOWED` | 403 | User type not allowed for this operation. |
| `FILE_UPLOAD_TEMP_KEY_INVALID` | 400 | Invalid temporary key (format or signature). |
| `FILE_UPLOAD_TEMP_KEY_EXPIRED` | 400 | Temporary key expired. |
| `FILE_UPLOAD_TEMP_KEY_MISMATCH` | 403 | Temporary key does not belong to the current user or model. |

---

## Best Practices for Integration

1. **Use `temp_session_key` for unsaved models**  
   When creating a new entity (e.g., product, order), upload files first using `temp_session_key`, then save the entity, and finally link the files by calling the `attachTempFiles` function on the server side (or via a custom endpoint).

2. **Handle `error_code` programmatically**  
   Instead of relying on text messages, use `error_code` to determine the error type and display appropriate messages to the user (e.g., `FILE_UPLOAD_PERMISSION_DENIED` → "Not authorized").

3. **Use `with_thumbs` only when needed**  
   Requesting thumbnails consumes extra time to generate them (if not already present). Use it only on display screens that require them.

4. **Enable `app.debug` in development environment**  
   When encountering 500 errors, enable debug mode to get full details in the `debug` field to help diagnose the issue.

5. **Store temporary session keys securely**  
   Store `temp_session_key` in the user session or a temporary database, and do not expose it in the frontend unless absolutely necessary (as it contains sensitive information).

6. **Review system settings**  
   Ensure that upload fields are properly registered in `FileUploadRegistry`, and that global settings (e.g., `disable_upload`, `disable_auto_resize`) are set according to your needs.

7. **Monitor the `fileupload.log`**  
   The dedicated log file (`storage/logs/fileupload.log`) contains details of failed upload attempts, including attempts to upload dangerous files, which helps with security monitoring.

---

## Frequently Asked Questions

**Q: How do I link temporary files to a model after saving it?**  
A: After saving the model (e.g., `$product->save()`), call the `attachTempFiles` function from server code:
```php
\Nano\FileUpload\Classes\FileUploadService::instance()->attachTempFiles($product, 'image', $tempSessionKey);
```
You can create a custom endpoint for this purpose if you want to link via API.

**Q: What thumbnail sizes can I request?**  
A: You can request any dimensions you want via `thumb_sizes`, and the thumbnail will be generated automatically (if not already present). Common sizes: `[150,150,'crop']`, `[300,300,'crop']`, `[800,600,'auto']`.

**Q: How do I know if a file has been resized or watermarked?**  
A: These operations happen automatically according to the field settings in `FileUploadRegistry`. If you are a developer, check the field settings (`auto_resize`, `auto_watermark`). If you are an API user, you don't need to do anything extra.

**Q: What if I want to upload a file with the same name twice?**  
A: The system generates a unique `disk_name` automatically, so no conflict will occur. You can rely on `id` and `path` to differentiate.

**Q: Can I upload PDF or DOCX files?**  
A: Yes, if the field is registered with type `file` or appropriate `allowed_types` is specified. Supported file types by default include `pdf,doc,docx,xls,xlsx,zip,rar,txt` (for `file` type).

---

## Additional Documentation

- [General Add-on Documentation](./Docs-FileUpload-en.md)
- [FileUploadRegistry Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for FileUploadRegistry Class](./Docs-FileUploadRegistry-Class-Advanced-Examples-en.md)
- [FileUploadService Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for FileUploadService Class](./Docs-FileUploadService-Class-Advanced-Examples-en.md)
- [FileUploadUserManager Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for FileUploadUserManager Class](./Docs-FileUploadUserManager-Class-Advanced-Examples-en.md)