## File Upload API Documentation – Version 1.2.5

**Version:** 1.2.5  
**Base Path:** `/api/v1/fileupload`  
**Authentication:** OAuth 2.0 (access token in header `Authorization: Bearer <token>`)

---

## Overview

This document is intended for external developers (web stores, e-commerce applications, order delivery applications, and integrated systems) who wish to integrate the file upload service into their applications. The API provides secure and easy-to-use endpoints for uploading single or multiple files, deleting files, retrieving files associated with specific models (such as products, orders, users), as well as replacing files in singular relationships (`attachOne`) and running the integrated test suite.

**Key Features:**
- Support for multiple file formats (multipart/form-data or base64).
- Temporary file upload before saving the model (using temporary session keys).
- **File replacement in singular relationships (`attachOne`)**: When passing `model_id` for a field of type `attachOne` and a file is already associated, it is automatically replaced (`edit` operation).
- Automatic image transformations (resize, watermark) according to system settings.
- Store files on multiple disks (local, S3, FTP).
- Unified responses with unique error codes for easy programmatic handling.
- Full security via OAuth 2.0 and user permission checks.
- **Endpoint to run tests** (available only in development environment).
- **Endpoints to query registered modules, their settings, and constraints** (new in version 1.2.3, improved in 1.2.5).
- **Endpoints to check permissions** at global, module, and field levels (new in version 1.2.3, improved in 1.2.5).
- **Endpoint to validate temporary keys** (new in version 1.2.3).

> **Note about version 1.2.5:** The query and permission endpoints have been restructured to use query parameters instead of route parameters to avoid encoding issues with model names containing backslashes (`\`). The new routes are documented below. The old routes (containing `{modelClass}` in the path) are deprecated and will be removed in a future version.

---

## Unified Response Structure

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
| `code` | `int` | HTTP status code (200, 400, 401, 403, 404, 422, 500, 503). |
| `status` | `bool` | `true` for success, `false` for failure. |
| `message` | `string` | Clear user message (translatable). |
| `error` | `string\|null` | Technical error details (in case of failure). |
| `errors` | `array\|null` | Array of validation errors (e.g., input validation). |
| `data` | `mixed` | Core data (e.g., uploaded file information). |
| `meta` | `mixed` | Additional data (e.g., test summary). |
| `input_data` | `array` | Copy of the data sent by the user (for tracing). |
| `process_data` | `array` | Internal processing information (e.g., temporary session key, operation performed `add`/`edit`). |
| `debug` | `array\|null` | Debug information (only appears when `app.debug` is enabled). |
| `error_code` | `string\|null` | Unique error code (e.g., `FILE_UPLOAD_PERMISSION_DENIED`). |

---

## Authentication and Security

All endpoints are protected by **OAuth 2.0**. The access token must be included in the request header:

```
Authorization: Bearer <your_access_token>
```

- If no token is provided: `401 Unauthorized` response.
- If token is invalid or expired: `401 Unauthorized` response.

**Note:** Permissions for operations (upload, replace, delete, retrieve) depend on the user type (`backend` / `frontend`) and the settings registered for each model and field. Ensure the user has appropriate permissions before calling the API. File replacement operations require `edit` permission.

---

## Endpoints

### 1. Upload Single File

**Method:** `POST`  
**Path:** `/upload`  
**Content-Type:** `multipart/form-data` or `application/json` (when using base64).

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model (e.g., `Nano\Shop\Models\Product`). |
| `field` | `string` | Yes | Field name as registered in the system (e.g., `image`, `gallery`). |
| `file` | `file` | Conditional* | The uploaded file (multipart format). |
| `file_base64` | `string` | Conditional* | File data in base64 format (alternative to `file`). |
| `temp_session_key` | `string` | No | Temporary session key to attach the file later (for unsaved models). |
| `model_id` | `int` | No | Saved model ID. If the field is of type `attachOne` and the model already has an associated file, **the old file will be replaced with the new one** (`edit` operation). |

> * Either `file` or `file_base64` must be provided.

#### Request Examples

**Multipart/form-data request (direct file upload):**

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

**Request to replace an existing product image (`edit` operation):**

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

**✅ Success response (direct upload and attach to existing model – `add` operation):**

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

**✅ Success response (replace existing file – `edit` operation):**

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

**✅ Success response (temporary upload – to be attached later):**

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

> **Important:** For temporary uploads, you must save `temp_session_key` in the user session or database to use later when attaching the file to the saved model.

**❌ Error response (permission denied – or edit globally disabled):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to upload files for this field",
    "error": "You do not have permission to upload files for this field",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

**❌ Error response (file type not allowed):**

```json
{
    "code": 422,
    "status": false,
    "message": "File type not allowed. Allowed types: jpg,jpeg,png. Uploaded extension: exe.",
    "error": "File type not allowed. Allowed types: jpg,jpeg,png. Uploaded extension: exe.",
    "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
}
```

**❌ Error response (file too large):**

```json
{
    "code": 422,
    "status": false,
    "message": "File size exceeds the maximum allowed (2048 KB). Actual size: 5120 KB.",
    "error": "File size exceeds the maximum allowed (2048 KB). Actual size: 5120 KB.",
    "error_code": "FILE_UPLOAD_FILE_SIZE_EXCEEDED"
}
```

**❌ Error response (dangerous file – blacklisted):**

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

### 2. Upload Multiple Files (for multiple field)

**Method:** `POST`  
**Path:** `/upload-multiple`

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model. |
| `field` | `string` | Yes | Name of the multiple field (e.g., `gallery`). |
| `files` | `array` | Yes | Array of files (each element is either an `UploadedFile` object or base64 string). |
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

**✅ Success response (all files uploaded successfully):**

```json
{
    "code": 200,
    "status": true,
    "message": "Successfully uploaded 3 of 3 files",
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

**⚠️ Partial success response (some files failed):**

```json
{
    "code": 200,
    "status": true,
    "message": "Successfully uploaded 2 of 3 files",
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

**❌ Complete failure response (all files failed):**

```json
{
    "code": 400,
    "status": false,
    "message": "All files failed to upload",
    "error": "File type not allowed, file too large",
    "data": [
        {
            "status": false,
            "error": "File type not allowed",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        },
        {
            "status": false,
            "error": "File size exceeds maximum allowed",
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
| `model_class` | `string` | No | Model class name (to check delete permission). |
| `field` | `string` | No | Field name (to check delete permission). |

#### Example Request

```http
DELETE /api/v1/fileupload/delete/123?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### Responses

**✅ Success response:**

```json
{
    "code": 200,
    "status": true,
    "message": "File deleted",
    "data": null
}
```

**❌ Error response (file not found):**

```json
{
    "code": 404,
    "status": false,
    "message": "File not found",
    "error": "File not found",
    "error_code": "FILE_UPLOAD_FILE_NOT_FOUND"
}
```

**❌ Error response (permission denied – user does not have delete permission):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to delete this file",
    "error": "You do not have permission to delete this file",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

**❌ Error response (delete operations globally disabled):**

```json
{
    "code": 503,
    "status": false,
    "message": "File deletion operations are currently disabled",
    "error": "File deletion operations are currently disabled",
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
| `model_class` | `string` | Yes | Full class name of the model. |
| `field` | `string` | Yes | Field name. |
| `model_id` | `int` | No | Saved model ID (to retrieve associated files). |
| `temp_session_key` | `string` | No | Temporary session key (to retrieve unattached temporary files). |
| `with_thumbs` | `bool` | No | Include thumbnails (`true` / `false`, default `false`). |
| `thumb_sizes` | `object` | No | Thumbnail sizes (array of size names and dimensions). |

#### Request Examples

**Retrieve product gallery images (associated with saved model):**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10 HTTP/1.1
Authorization: Bearer ...
```

**Retrieve temporary files (unattached) using session key:**

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

**✅ Success response (associated files):**

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

**✅ Success response (with thumbnails):**

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

**✅ Success response (temporary files – after security validation):**

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

**❌ Error response (invalid or expired temporary key):**

```json
{
    "code": 400,
    "status": false,
    "message": "Temporary session key is invalid or expired",
    "error": "Temporary session key is invalid or expired",
    "error_code": "FILE_UPLOAD_TEMP_KEY_INVALID"
}
```

**❌ Error response (temporary key does not belong to current user/model/field):**

```json
{
    "code": 403,
    "status": false,
    "message": "Temporary key does not belong to this model/field",
    "error": "Temporary key does not belong to this model/field",
    "error_code": "FILE_UPLOAD_TEMP_KEY_MISMATCH"
}
```

---

### 5. Run Add-on Tests (Local Environment Only)

**Method:** `GET`  
**Path:** `/tests`

This endpoint is intended for developers to test the add-on installation and core functionality. **Cannot be used in production** (requires `app.debug = true` or `local` environment).

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `test_version` | `string` | No | Test version (`v1` or `v2`, default `v2`). Version `v2` is the latest and provides unified output. |
| `filter` | `string` | No | Filter results (`all`, `passed`, `failed`, default `all`). |

#### Example Request

```http
GET /api/v1/fileupload/tests?test_version=v2&filter=passed HTTP/1.1
Authorization: Bearer <token>
```

#### Response

Returns an array of all test results (or filtered) with a summary (`total`, `passed`, `failed`, `success_rate`). Each test contains `test_code`, `name`, `description`, `status`, `message`, `data`, `error`, `debug`, etc.

**✅ Success response (abbreviated example):**

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
            "success_rate": 100,
            "test_version": "v2",
            "filter_applied": "passed"
        }
    }
}
```

> **Warning:** Never use this endpoint in production, as it may expose internal system information.

---

### 6. Get All Registered Modules (unchanged)

**Method:** `GET`  
**Path:** `/models`

**Description:** Returns a list of registered modules that the current user has `view` permission on, with brief information about their fields (name, type, whether multiple). This endpoint is useful for front-end applications that need to build dynamic upload forms based on actual settings.

#### Query Parameters

None required.

#### Example Request

```http
GET /api/v1/fileupload/models HTTP/1.1
Authorization: Bearer ...
```

#### Response (same as before)

---

### 7. Get Settings for a Specific Module (updated in 1.2.5)

**Method:** `GET`  
**Path:** `/model/config`

**Description:** Returns the full settings of a specific module (after checking existence and `view` permission). Includes registered fields with some details (type, max size, allowed types, etc.).

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model (e.g., `Nano\Shop\Models\Product`). **No need to encode it**, pass as a normal value. |

#### Example Request

```http
GET /api/v1/fileupload/model/config?model_class=Nano\Shop\Models\Product HTTP/1.1
Authorization: Bearer ...
```

#### Response

**✅ Success response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "enabled": true,
        "label": "Product",
        "allowed_user_types": ["backend", "frontend"],
        "disabled_operations": [],
        "fields": {
            "image": {
                "type": "image",
                "label": "image",
                "multiple": false,
                "required": false,
                "max_filesize": 2048,
                "allowed_types": "jpg,jpeg,png",
                "use_caption": true,
                "disabled_operations": []
            },
            "gallery": {
                "type": "multiple",
                "label": "gallery",
                "multiple": true,
                "required": false,
                "max_filesize": 2048,
                "allowed_types": "jpg,jpeg,png",
                "use_caption": true,
                "disabled_operations": []
            }
        }
    }
}
```

**❌ Error response (module not registered):**

```json
{
    "code": 404,
    "status": false,
    "message": "Model is not registered in the system",
    "error_code": "FILE_UPLOAD_MODEL_NOT_REGISTERED"
}
```

**❌ Error response (no view permission):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to upload files for this field",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

---

### 8. Get Settings for a Specific Field (updated in 1.2.5)

**Method:** `GET`  
**Path:** `/field/config`

**Description:** Returns settings for a specific field (e.g., type, max size, allowed types). Use this to check field settings before attempting upload.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model. |
| `field` | `string` | Yes | Field name. |

#### Example Request

```http
GET /api/v1/fileupload/field/config?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### Response

**✅ Success response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "type": "image",
        "label": "image",
        "multiple": false,
        "required": false,
        "max_filesize": 2048,
        "allowed_types": "jpg,jpeg,png",
        "use_caption": true,
        "disabled_operations": []
    }
}
```

**❌ Error response (field not registered):**

```json
{
    "code": 404,
    "status": false,
    "message": "Field 'video' is not registered for model 'Nano\\Shop\\Models\\Product'",
    "error_code": "FILE_UPLOAD_FIELD_NOT_REGISTERED"
}
```

---

### 9. Get Field Constraints (updated in 1.2.5)

**Method:** `GET`  
**Path:** `/field/constraints`

**Description:** Returns the field constraints used in file validation (`max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`). Front-ends can apply the same constraints before uploading to avoid server errors.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model. |
| `field` | `string` | Yes | Field name. |

#### Example Request

```http
GET /api/v1/fileupload/field/constraints?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### Response

**✅ Success response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "max_filesize": 2048,
        "allowed_types": "jpg,jpeg,png",
        "multiple": false,
        "max_files": null,
        "required": false,
        "is_public": true,
        "use_caption": true,
        "thumb_options": {
            "mode": "crop",
            "extension": "auto"
        },
        "type": "image"
    }
}
```

---

### 10. Get Advanced Processing Options for a Specific Field (updated in 1.2.5)

**Method:** `GET`  
**Path:** `/processing-options`

**Description:** Returns the advanced processing options for the field: storage disk (`storage_disk`), automatic resizing (`auto_resize` with `resize_options`), and watermark (`auto_watermark` with `watermark_options`). This information is useful for advanced front-ends that need to know how files will be processed after upload.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model. |
| `field` | `string` | Yes | Field name. |

#### Example Request

```http
GET /api/v1/fileupload/processing-options?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### Response

**✅ Success response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "storage_disk": "s3",
        "auto_resize": true,
        "resize_options": {
            "width": 1920,
            "height": 1080,
            "mode": "auto"
        },
        "auto_watermark": true,
        "watermark_options": {
            "position": "bottom-right",
            "resize_percentage": 15
        }
    }
}
```

**❌ Error response (field not registered):** (same error structure as before)

---

### 11. Permission Checks

#### 11.1 Check if an Operation is Globally Enabled (unchanged)

**Method:** `GET`  
**Path:** `/permissions/global/{operation}`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `string` | Yes | Operation: `add`, `edit`, `delete`, `view`. |

**Example Request:**

```http
GET /api/v1/fileupload/permissions/global/edit HTTP/1.1
Authorization: Bearer ...
```

**✅ Success response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "operation": "edit",
        "allowed": true,
        "disabled_globally": false
    }
}
```

---

#### 11.2 Check Operation Permission at a Specific Module Level (updated in 1.2.5)

**Method:** `GET`  
**Path:** `/permissions/model`

**Description:** Checks whether a specific operation is enabled at a specific module level (considering `disabled_operations` at the model level).

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model. |
| `operation` | `string` | Yes | Operation: `add`, `edit`, `delete`, `view`. |

#### Example Request

```http
GET /api/v1/fileupload/permissions/model?model_class=Nano\Shop\Models\Product&operation=edit HTTP/1.1
Authorization: Bearer ...
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "operation": "edit",
        "model_operation_enabled": true,
        "user_type_allowed": true,
        "can_proceed": true
    }
}
```

---

#### 11.3 Check Operation Permission at a Specific Field Level (updated in 1.2.5)

**Method:** `GET`  
**Path:** `/permissions/field`

**Description:** Checks whether a specific operation is enabled at a specific field level (considering `disabled_operations` at the field level and user permissions).

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model. |
| `field` | `string` | Yes | Field name. |
| `operation` | `string` | Yes | Operation: `add`, `edit`, `delete`, `view`. |

#### Example Request

```http
GET /api/v1/fileupload/permissions/field?model_class=Nano\Shop\Models\Product&field=image&operation=edit HTTP/1.1
Authorization: Bearer ...
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "operation": "edit",
        "field_operation_enabled": true,
        "user_has_full_permission": true,
        "can_proceed": true
    }
}
```

> **Note:** `user_has_full_permission` includes checking user type, specific permissions, and global/model/field settings.

---

#### 11.4 Integrated Permission Check (using validate) – unchanged

**Method:** `POST`  
**Path:** `/permissions/check`

**Description:** Checks permission comprehensively (global, module, field, user type, specific permissions) and returns the result without needing to attempt an actual file upload.

#### Request Parameters (JSON)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Full class name of the model. |
| `operation` | `string` | Yes | Operation: `add`, `edit`, `delete`, `view`. |
| `field` | `string` | No | Field name (optional, if checking at field level). |

#### Example Request

```json
POST /api/v1/fileupload/permissions/check HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "operation": "edit",
    "field": "image"
}
```

#### Responses

**✅ Success response (allowed):**

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "allowed": true,
        "model_class": "Nano\\Shop\\Models\\Product",
        "operation": "edit",
        "field": "image",
        "message": "Permission granted"
    }
}
```

**❌ Success response (not allowed – with error message):**

```json
{
    "code": 200,
    "status": true,
    "message": "Operation completed successfully",
    "data": {
        "allowed": false,
        "model_class": "Nano\\Shop\\Models\\Product",
        "operation": "edit",
        "field": "image",
        "message": "You do not have permission to upload files for this field"
    }
}
```

> **Note:** In case of not allowed, `code` remains 200 and `status` true because the request itself succeeded in performing the check. The error is reported in `data.message`.

---

### 12. Validate and Match a Temporary Key (unchanged)

**Method:** `POST`  
**Path:** `/temp-key/validate`

**Description:** Validates a temporary session key (format, signature, expiration) and matches it against the provided model, field, and user. This endpoint is useful for applications that need to ensure a stored temporary key is still valid before using it in attach or upload operations.

#### Request Parameters (JSON)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `temp_key` | `string` | Yes | Temporary session key (starts with `tmp_`). |
| `model_class` | `string` | Yes | Expected full class name of the model. |
| `field` | `string` | Yes | Expected field name. |
| `strict_mode` | `bool` | No | Strict mode (requires exact model and field match). Default `true`. |
| `allow_expired_key` | `bool` | No | Allow expired keys within a grace period. Default `false`. |
| `expiry_grace_period` | `int` | No | Grace period in seconds (when `allow_expired_key=true`). Default `300` (5 minutes). |

#### Example Request

```json
POST /api/v1/fileupload/temp-key/validate HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "temp_key": "tmp_YWJjMTIzOnNvbWVoYXNo",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true,
    "allow_expired_key": false
}
```

#### Responses

**✅ Success response (valid key):**

```json
{
    "code": 200,
    "status": true,
    "message": "Temporary key validated successfully",
    "data": {
        "valid": true,
        "key_data": {
            "modelClass": "Nano\\Shop\\Models\\Product",
            "field": "image",
            "userId": "123",
            "userType": "backend",
            "timestamp": 1744123456
        },
        "matched_data": {
            "model_matched": true,
            "field_matched": true,
            "user_matched": true,
            "user_id_matched": true,
            "user_type_matched": true
        }
    },
    "process_data": {
        "validated": true,
        "model_matched": true,
        "field_matched": true,
        "user_matched": true
    }
}
```

**❌ Error response (invalid key):**

```json
{
    "code": 400,
    "status": false,
    "message": "Temporary session key is invalid or expired",
    "error_code": "FILE_UPLOAD_TEMP_KEY_INVALID"
}
```

**❌ Error response (key does not match model/field):**

```json
{
    "code": 403,
    "status": false,
    "message": "Temporary key does not belong to this model/field",
    "error_code": "FILE_UPLOAD_TEMP_KEY_MISMATCH"
}
```

---

## Error Codes – Complete List

| error_code | HTTP Code | Meaning |
|------------|-----------|---------|
| `FILE_UPLOAD_GENERAL` | 500 | General unexpected error. |
| `FILE_UPLOAD_PERMISSION_DENIED` | 403 | User does not have permission for the operation (or `disable_edit` enabled). |
| `FILE_UPLOAD_UNAUTHORIZED` | 401 | Unauthorized (not authenticated). |
| `FILE_UPLOAD_MODEL_NOT_REGISTERED` | 400 | Model is not registered in the system. |
| `FILE_UPLOAD_FIELD_NOT_REGISTERED` | 400 | Field is not registered for the model. |
| `FILE_UPLOAD_MODEL_NOT_FOUND` | 404 | Model not found (when `model_id` provided). |
| `FILE_UPLOAD_FILE_NOT_FOUND` | 404 | File not found. |
| `FILE_UPLOAD_INVALID_FILE_DATA` | 400 | Invalid file data (neither file nor base64). |
| `FILE_UPLOAD_FILE_SIZE_EXCEEDED` | 422 | File size exceeds the allowed limit. |
| `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` | 422 | File extension not allowed. |
| `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` | 422 | File extension is dangerous (PHP, JS, HTML, etc.). |
| `FILE_UPLOAD_FILE_UPLOAD_FAILED` | 500 | File upload failed (internal error). |
| `FILE_UPLOAD_MAX_FILES_EXCEEDED` | 422 | Number of files exceeds the allowed limit (for multiple fields). |
| `FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY` | 503 | Upload operations are globally disabled. |
| `FILE_UPLOAD_DELETE_DISABLED_GLOBALLY` | 503 | Delete operations are globally disabled. |
| `FILE_UPLOAD_GET_DISABLED_GLOBALLY` | 503 | Retrieve operations are globally disabled. |
| `FILE_UPLOAD_USER_TYPE_NOT_ALLOWED` | 403 | User type is not allowed for this operation. |
| `FILE_UPLOAD_TEMP_KEY_INVALID` | 400 | Invalid temporary key (format or signature). |
| `FILE_UPLOAD_TEMP_KEY_EXPIRED` | 400 | Temporary key has expired. |
| `FILE_UPLOAD_TEMP_KEY_MISMATCH` | 403 | Temporary key does not belong to the current user/model/field. |

---

## Best Practices for Integration

1. **Use `temp_session_key` for unsaved models**  
   When creating a new entity (e.g., product, order), upload files first using `temp_session_key`, then save the entity, and finally attach the files by calling the `attachTempFiles` function on the server side (or via a custom endpoint).

2. **To replace an existing file** (`edit` operation): Use `model_id` with a field of type `attachOne`. The old file will be automatically deleted after the new one is uploaded, with data integrity guaranteed via transactions. Ensure the user has `edit` permission.

3. **Handle `error_code` programmatically**  
   Instead of relying on text messages, use `error_code` to determine the error type and display appropriate user messages.

4. **Use query endpoints (`/models`, `/field/config`, `/field/constraints`) to build dynamic interfaces**  
   You can fetch field constraints and display them to the user (e.g., show max size, allowed types) before they select a file.

5. **Check permissions beforehand using `/permissions/check`**  
   To avoid rejection attempts, you can check user permission for the operation before displaying the file upload form.

6. **Validate the temporary key before using it**  
   Use the `/temp-key/validate` endpoint to ensure the key you store in the session is still valid (not expired, does not belong to another user).

7. **Use `with_thumbs` only when needed**  
   Requesting thumbnails consumes additional time to generate them (if not already present). Use it only on screens that need them.

8. **Enable `app.debug` in development**  
   When encountering 500 errors, enable debug mode to get full details in the `debug` field to help diagnose the issue.

9. **Store temporary session keys securely**  
   Store `temp_session_key` in the user session or a temporary database, and do not expose it in the front-end unless absolutely necessary (as it carries sensitive information).

10. **Review system settings**  
    Ensure that upload fields are registered correctly, and that global settings (e.g., `disable_upload`, `disable_edit`, `disable_auto_resize`) are set according to your needs.

11. **Monitor the `fileupload.log`**  
    The dedicated log file (`storage/logs/fileupload.log`) contains details of failed upload attempts, including attempts to upload dangerous files, aiding security monitoring.

---

## Frequently Asked Questions

**Q: How do I attach temporary files to a model after saving it?**  
A: After saving the model (e.g., `$product->save()`), call the `attachTempFiles` function from server code:
```php
\Nano\FileUpload\Classes\FileUploadService::instance()->attachTempFiles($product, 'image', $tempSessionKey);
```
You can create a custom endpoint for this purpose if you want to attach via API.

**Q: How do I know whether an upload operation was `add` or `edit`?**  
A: In the success response, the `process_data` field contains the key `operation` with value `"add"` or `"edit"`.

**Q: What happens if the new file upload fails during an `edit` operation?**  
A: Thanks to the use of transactions (`Db::transaction`), the old file will not be deleted. The data remains as it was before the attempt.

**Q: What thumbnail sizes can I request?**  
A: You can request any dimensions you want via `thumb_sizes`, and the thumbnail will be generated automatically (if not already present). Common sizes: `[150,150,'crop']`, `[300,300,'crop']`, `[800,600,'auto']`.

**Q: How do I know if a file has been resized or watermarked?**  
A: These operations are performed automatically according to the field settings. If you are a developer, review the field settings (`auto_resize`, `auto_watermark`). If you are an API user, you don't need to do anything extra.

**Q: What if I want to upload a file with the same name twice?**  
A: The system generates a unique `disk_name` automatically, so no conflict occurs. You can rely on `id` and `path` for differentiation.

**Q: Can I upload PDF or DOCX files?**  
A: Yes, if the field is registered as type `file` or with the appropriate `allowed_types`. Supported file types by default include `pdf,doc,docx,xls,xlsx,zip,rar,txt` (for `file` type).

**Q: How can I get a list of all modules and fields available to the current user?**  
A: Use the `/models` endpoint. You will get modules for which the user has `view` permission, along with basic field information.

**Q: How can I check whether a user has permission to upload a file to a specific field without attempting upload?**  
A: Use the `/permissions/check` endpoint with `operation: "add"` and the desired field.

**Q: What is the validity period of the temporary key I receive from the upload operation?**  
A: The key is valid for one hour (configurable in system settings). You can check its validity using the `/temp-key/validate` endpoint.

**Q: Why were the routes changed in version 1.2.5?**  
A: To avoid encoding issues with model names containing backslashes (`\`) when passed in the route. The new routes using query parameters are simpler and more secure. The old routes still work currently but are deprecated and will be removed in version 1.3.0. We recommend switching to the new routes immediately.

---

## Additional Documentation

- [General Add-on Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advanced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advanced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advanced-Examples-en.md)
