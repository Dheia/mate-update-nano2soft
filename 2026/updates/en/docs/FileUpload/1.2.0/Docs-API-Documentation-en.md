# File Upload API Documentation

**Version:** 1.2.0  
**Base Path:** `/api/v1/fileupload`  
**Authentication:** OAuth 2.0 (access token in header `Authorization: Bearer <token>`)

---

## Overview

This documentation is intended for external developers (web stores, e-commerce applications, order delivery applications, and integrated systems) who wish to integrate the file upload service into their applications. The API provides secure and easy-to-use endpoints for uploading single or multiple files, deleting files, retrieving files associated with specific models (e.g., products, orders, users), as well as replacing files in one-to-one relations (`attachOne`) and running the integrated test suite.

**Key Features:**
- Support for uploading files in multiple formats (multipart/form-data or base64).
- Temporary file upload before saving the model (using temporary session keys).
- **File replacement in one-to-one relations (`attachOne`)**: When `model_id` is passed for a field of type `attachOne` and the model already has an associated file, it is automatically replaced (edit operation).
- Automatic image transformation (resizing, watermarking) according to system settings.
- File storage on multiple disks (local, S3, FTP).
- Unified responses with unique error codes for easy programmatic handling.
- Full security via OAuth 2.0 and user permission checks.
- **Endpoint to run tests** (available only in development environment).

---

## Unified Response Structure

Every API response follows this structure:

```json
{
    "code": 200,
    "status": true,
    "message": "Operation successful",
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
| `code` | `int` | HTTP status code (200, 400, 401, 403, 404, 422, 500, 503). |
| `status` | `bool` | `true` for success, `false` for failure. |
| `message` | `string` | Clear message for the user (translatable). |
| `error` | `string\|null` | Technical error details (in case of failure). |
| `errors` | `array\|null` | Array of validation errors (e.g., input validation). |
| `data` | `mixed` | Main data (e.g., uploaded file information). |
| `meta` | `mixed` | Additional data (currently unused). |
| `input_data` | `array` | Copy of the data sent by the user (for tracking). |
| `process_data` | `array` | Internal processing information (e.g., temporary session key, performed operation `add`/`edit`). |
| `debug` | `array\|null` | Debugging information (only appears when `app.debug` is enabled). |
| `error_code` | `string\|null` | Unique error code (e.g., `FILE_UPLOAD_PERMISSION_DENIED`). |

---

## Authentication and Security

All endpoints are protected by **OAuth 2.0**. You must include an access token in the request header:

```
Authorization: Bearer <your_access_token>
```

- If no token is provided: `401 Unauthorized` response.
- If the token is invalid or expired: `401 Unauthorized` response.

**Note:** Permissions for operations (upload, replace, delete, retrieve) depend on the user type (`backend` / `frontend`) and the settings registered for each model and field. Ensure the user has the appropriate permissions before calling the API. Replacement operations require `edit` permission.

---

## Endpoints

### 1. Upload a Single File

**Method:** `POST`  
**Path:** `/upload`  
**Content-Type:** `multipart/form-data` or `application/json` (when using base64).

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified class name of the model (e.g., `Nano\Shop\Models\Product`). |
| `field` | `string` | Yes | Field name as registered in the system (e.g., `image`, `gallery`). |
| `file` | `file` | Conditional* | The uploaded file (multipart format). |
| `file_base64` | `string` | Conditional* | File data in base64 format (alternative to `file`). |
| `temp_session_key` | `string` | No | Temporary session key to attach the file later (for unsaved models). |
| `model_id` | `int` | No | Saved model ID. If the field is of type `attachOne` and the model already has an associated file, **the old file will be replaced with the new one** (edit operation). |

> * Either `file` or `file_base64` must be provided.

#### Example Requests

**Request using multipart/form-data (direct file upload):**

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

**Request using JSON and base64 (for interfaces that do not support multipart):**

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

**Request to upload a temporary file (before saving the model):**

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

**Request to replace an existing product image (edit operation):**

```json
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,...",
    "model_id": 10
}
```

#### Responses

**✅ Success Response (direct upload and attachment to existing model – add operation):**

```json
{
    "code": 200,
    "status": true,
    "message": "File uploaded successfully",
    "data": {
        "id": 123,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "process_data": {
        "operation": "add",
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "user_type": "backend"
    }
}
```

**✅ Success Response (replacing an existing file – edit operation):**

```json
{
    "code": 200,
    "status": true,
    "message": "File uploaded successfully",
    "data": {
        "id": 124,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../new.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "process_data": {
        "operation": "edit",
        "existing_file_deleted": 123,
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "user_type": "backend"
    }
}
```

**✅ Success Response (temporary upload – to be attached later):**

```json
{
    "code": 200,
    "status": true,
    "message": "File uploaded successfully",
    "data": {
        "id": 125,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "process_data": {
        "temp_session_key": "tmp_YWJjMTIzOnNvbWVoYXNo",
        "operation": "add"
    }
}
```

> **Important:** In temporary upload, you must save the `temp_session_key` in the user's session or database to use it later when attaching the file to the saved model.

**❌ Error Response (permission denied – or editing globally disabled):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to upload files for this field",
    "error": "You do not have permission to upload files for this field",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

**❌ Error Response (file type not allowed):**

```json
{
    "code": 422,
    "status": false,
    "message": "File type not allowed. Allowed types: jpg,jpeg,png. Uploaded extension: exe.",
    "error": "File type not allowed. Allowed types: jpg,jpeg,png. Uploaded extension: exe.",
    "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
}
```

**❌ Error Response (file size too large):**

```json
{
    "code": 422,
    "status": false,
    "message": "File size exceeds the allowed limit (2048 KB). Actual size: 5120 KB.",
    "error": "File size exceeds the allowed limit (2048 KB). Actual size: 5120 KB.",
    "error_code": "FILE_UPLOAD_FILE_SIZE_EXCEEDED"
}
```

**❌ Error Response (dangerous file – blacklisted):**

```json
{
    "code": 422,
    "status": false,
    "message": "File type php is not allowed for security reasons.",
    "error": "File type php is not allowed for security reasons.",
    "error_code": "FILE_UPLOAD_FILE_TYPE_BLACKLISTED"
}
```

---

### 2. Upload Multiple Files (for a Multiple Field)

**Method:** `POST`  
**Path:** `/upload-multiple`

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified class name of the model. |
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
    "message": "3 out of 3 files uploaded successfully",
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
    "message": "2 out of 3 files uploaded successfully",
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
    "message": "All files failed to upload",
    "error": "File type not allowed, file size too large",
    "data": [
        {
            "status": false,
            "error": "File type not allowed",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        },
        {
            "status": false,
            "error": "File size exceeds the allowed limit",
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

### 3. Delete a File

**Method:** `DELETE`  
**Path:** `/delete/{id}`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `int` | Yes | ID of the file to delete. |

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | No | Model name (to check delete permission). |
| `field` | `string` | No | Field name (to check delete permission). |

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

**❌ Error Response (delete operations globally disabled):**

```json
{
    "code": 503,
    "status": false,
    "message": "File delete operations are currently disabled",
    "error": "File delete operations are currently disabled",
    "error_code": "FILE_UPLOAD_DELETE_DISABLED_GLOBALLY"
}
```

---

### 4. Retrieve Associated Files

**Method:** `GET`  
**Path:** `/files`

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified class name of the model. |
| `field` | `string` | Yes | Field name. |
| `model_id` | `int` | No | Saved model ID (to retrieve attached files). |
| `temp_session_key` | `string` | No | Temporary session key (to retrieve unattached temporary files). |
| `with_thumbs` | `bool` | No | Include thumbnails (`true` / `false`, default `false`). |
| `thumb_sizes` | `object` | No | Thumbnail sizes (array of size names and dimensions). |

#### Example Requests

**Retrieve gallery images for a product (attached to a saved model):**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10 HTTP/1.1
Authorization: Bearer ...
```

**Retrieve temporary files (unattached) using a session key:**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=image&temp_session_key=tmp_YWJjMTIzOnNvbWVoYXNo HTTP/1.1
Authorization: Bearer ...
```

**Retrieve gallery images with custom thumbnail sizes:**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10&with_thumbs=true&thumb_sizes[small][0]=150&thumb_sizes[small][1]=150&thumb_sizes[small][2]=crop&thumb_sizes[medium][0]=300&thumb_sizes[medium][1]=300 HTTP/1.1
Authorization: Bearer ...
```

#### Responses

**✅ Success Response (attached files):**

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

**❌ Error Response (temporary key does not belong to current user/model/field):**

```json
{
    "code": 403,
    "status": false,
    "message": "Temporary key does not match this model/field",
    "error": "Temporary key does not match this model/field",
    "error_code": "FILE_UPLOAD_TEMP_KEY_MISMATCH"
}
```

---

### 5. Run Plugin Tests (Local Environment Only)

**Method:** `GET`  
**Path:** `/tests`

This endpoint is intended for developers to verify the correct installation and basic functionality of the add-on. **Cannot be used in production** (requires `app.debug = true` or `local` environment).

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `test_version` | `string` | No | Test version (`v1` or `v2`, default `v2`). Version `v2` is the latest and provides unified outputs. |

#### Example Request

```http
GET /api/v1/fileupload/tests?test_version=v2 HTTP/1.1
Authorization: Bearer <token>
```

#### Response

Returns an array with results of all tests and a summary (`total`, `passed`, `failed`, `success_rate`). Each test contains `test_code`, `name`, `description`, `status`, `message`, `data`, `error`, `debug`, etc.

**✅ Success Response (shortened example):**

```json
{
    "code": 200,
    "status": true,
    "message": "Test suite completed",
    "data": [
        {
            "test_code": "REGISTER_MODEL_001",
            "name": "Register model and fields in FileUploadRegistry",
            "status": true,
            "message": "Model and field registered successfully"
        }
    ],
    "meta": {
        "summary": {
            "total": 43,
            "passed": 43,
            "failed": 0,
            "success_rate": 100
        }
    }
}
```

> **Warning:** Never use this endpoint in production, as it may expose internal system information.

---

## Error Codes – Complete List

| error_code | HTTP Code | Meaning |
|------------|-----------|---------|
| `FILE_UPLOAD_GENERAL` | 500 | General unexpected error. |
| `FILE_UPLOAD_PERMISSION_DENIED` | 403 | User does not have permission for the operation (or `disable_edit` is enabled). |
| `FILE_UPLOAD_UNAUTHORIZED` | 401 | Unauthorized (not authenticated). |
| `FILE_UPLOAD_MODEL_NOT_REGISTERED` | 400 | Model is not registered in the system. |
| `FILE_UPLOAD_FIELD_NOT_REGISTERED` | 400 | Field is not registered for the model. |
| `FILE_UPLOAD_MODEL_NOT_FOUND` | 404 | Model not found (when `model_id` is provided). |
| `FILE_UPLOAD_FILE_NOT_FOUND` | 404 | File not found. |
| `FILE_UPLOAD_INVALID_FILE_DATA` | 400 | Invalid file data (neither file nor base64). |
| `FILE_UPLOAD_FILE_SIZE_EXCEEDED` | 422 | File size exceeds the allowed limit. |
| `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` | 422 | File extension is not allowed. |
| `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` | 422 | File extension is dangerous (PHP, JS, HTML, etc.). |
| `FILE_UPLOAD_FILE_UPLOAD_FAILED` | 500 | File upload failed (internal error). |
| `FILE_UPLOAD_MAX_FILES_EXCEEDED` | 422 | Number of files exceeds the allowed limit (for multiple fields). |
| `FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY` | 503 | Upload operations are globally disabled. |
| `FILE_UPLOAD_DELETE_DISABLED_GLOBALLY` | 503 | Delete operations are globally disabled. |
| `FILE_UPLOAD_GET_DISABLED_GLOBALLY` | 503 | Get operations are globally disabled. |
| `FILE_UPLOAD_USER_TYPE_NOT_ALLOWED` | 403 | User type is not allowed for this operation. |
| `FILE_UPLOAD_TEMP_KEY_INVALID` | 400 | Temporary key is invalid (format or signature). |
| `FILE_UPLOAD_TEMP_KEY_EXPIRED` | 400 | Temporary key has expired. |
| `FILE_UPLOAD_TEMP_KEY_MISMATCH` | 403 | Temporary key does not belong to the current user or model. |

---

## Integration Best Practices

1. **Use `temp_session_key` for unsaved models**  
   When creating a new entity (e.g., product, order), upload the files first using `temp_session_key`, then save the entity, and finally attach the files by calling the `attachTempFiles` function on the server side (or via a custom endpoint).

2. **To replace an existing file** (edit operation): Use `model_id` with a field of type `attachOne`. The old file will be automatically deleted after the new one is uploaded, with data integrity guaranteed via transactions. Ensure the user has `edit` permission.

3. **Handle `error_code` programmatically**  
   Instead of relying on text messages, use `error_code` to determine the error type and display appropriate messages to the user (e.g., `FILE_UPLOAD_PERMISSION_DENIED` → "Not authorized").

4. **Use `with_thumbs` only when needed**  
   Requesting thumbnails consumes additional time to generate them (if they don't already exist). Use them only on display screens that need them.

5. **Enable `app.debug` in development environment**  
   When encountering 500 errors, enable debug mode to get full details in the `debug` field to help diagnose the issue.

6. **Store temporary session keys securely**  
   Save `temp_session_key` in the user's session or a temporary database, and do not expose it in the frontend unless absolutely necessary (because it contains sensitive information).

7. **Review system settings**  
   Ensure that upload fields are correctly registered in `FileUploadRegistry`, and that global settings (e.g., `disable_upload`, `disable_edit`, `disable_auto_resize`) are set according to your needs.

8. **Monitor `fileupload.log`**  
   The dedicated log file (`storage/logs/fileupload.log`) contains details of failed upload attempts, including attempts to upload dangerous files, helping with security monitoring.

---

## Frequently Asked Questions

**Q: How do I attach temporary files to a model after saving it?**  
A: After saving the model (e.g., `$product->save()`), call the `attachTempFiles` function from server code:
```php
\Nano\FileUpload\Classes\FileUploadService::instance()->attachTempFiles($product, 'image', $tempSessionKey);
```
You can create a custom endpoint for this purpose if you want to attach via API.

**Q: How do I know whether an upload operation was `add` or `edit`?**  
A: In a successful upload response, the `process_data` field contains the key `operation` with the value `"add"` or `"edit"`.

**Q: What happens if the new file upload fails during an `edit` operation?**  
A: Thanks to the use of transactions (`Db::transaction`), the old file will not be deleted. The data remains as it was before the attempt.

**Q: What thumbnail sizes can I request?**  
A: You can request any dimensions you want via `thumb_sizes`, and the thumbnail will be generated automatically (if not already present). Common sizes: `[150,150,'crop']`, `[300,300,'crop']`, `[800,600,'auto']`.

**Q: How do I know if a file has been resized or watermarked?**  
A: These operations happen automatically according to the field settings in `FileUploadRegistry`. If you are a developer, review the field settings (`auto_resize`, `auto_watermark`). If you are an API user, you don't need to do anything extra.

**Q: What if I want to upload a file with the same name twice?**  
A: The system automatically generates a unique name (`disk_name`), so no conflict will occur. You can rely on `id` and `path` for differentiation.

**Q: Can I upload PDF or DOCX files?**  
A: Yes, if the field is registered with type `file` or appropriate `allowed_types` are specified. Supported file types by default include `pdf,doc,docx,xls,xlsx,zip,rar,txt` (for type `file`).

---

## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)