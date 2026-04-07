# File Upload API Documentation

**Product:** NanoSoft  
**Version:** 1.0.7  
**Base Path:** `/api/v1/fileupload`  
**Authentication:** OAuth 2.0 (access token in header `Authorization: Bearer <token>`)

---

## Overview

This API provides a set of endpoints for managing file uploads in NanoSoft applications, based on the `Nano.FileUpload` system. These services allow developers to upload a single file or multiple files, delete specific files, and retrieve files associated with given models, with automatic checks for permissions, user types, and pre-registered constraints, in addition to supporting advanced features such as automatic image transformation and multi-storage.

All responses follow the unified structure described below and include a unique error code (`error_code`) to facilitate handling in client applications.

---

## Unified Response Structure

Every API response follows this structure:

```json
{
    "code": 200,
    "status": true,
    "message": "Message text",
    "error": null,
    "errors": [],
    "data": {},
    "meta": {},
    "input_data": {},
    "process_data": {},
    "debug": {},
    "error_code": "FILE_UPLOAD_SUCCESS"
}
```

- `code`: HTTP status code (200 for success, 400/403/404/500 for failure).
- `status`: `true` for success, `false` for failure.
- `message`: Explanatory message in Arabic (or the language set in the application).
- `error`: Error details (in case of failure).
- `errors`: Array of validation errors (if any).
- `data`: Primary response data (e.g., uploaded file information).
- `meta`: Additional data (e.g., pagination info) – currently unused.
- `input_data`: Copy of the data sent by the user (for tracking).
- `process_data`: Internal processing information (e.g., temporary session key, storage disk used).
- `debug`: Debug information (only shown when `app.debug` is enabled).
- `error_code`: Unique error code (e.g., `FILE_UPLOAD_PERMISSION_DENIED`) – appears only in error responses, making it easier to handle errors programmatically.

---

## Authentication

All endpoints are protected by **OAuth 2.0**. You must include an access token in the request header:

```
Authorization: Bearer <your_access_token>
```

If the token is not provided or is invalid, you will receive a `401` response with the message "Authentication failed".

---

## Endpoints

### 1. Upload Single File

**Method:** `POST`  
**Path:** `/upload`

**Request Parameters (Body):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified class name of the model (e.g., `Nano\Shop\Models\Product`). |
| `field` | `string` | Yes | Field name as registered in `FileUploadRegistry`. |
| `file` | `file` | Conditional* | The uploaded file (multipart format). |
| `file_base64` | `string` | Conditional* | File data as a base64 string (alternative to `file`). |
| `temp_session_key` | `string` | No | Temporary session key to link the file later (used with unsaved models). |
| `model_id` | `int` | No | ID of the saved model (if it exists). |

> * Either `file` or `file_base64` must be provided.

**Advanced Notes:**
- If `model_id` is provided and the model exists, the file will be linked directly.
- If `model_id` is not provided or the model is not saved, a temporary session key (`temp_session_key`) will be generated and returned with the response.
- Automatic transformations (resizing, watermarking) are applied to images automatically according to field settings and global settings.
- SHA256 hash of the content is calculated and stored in the `hash` field, image dimensions are stored in `meta`, and an expiration date is set for temporary files.

**Example Request (multipart/form-data):**

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

(file data)
------WebKitFormBoundary--
```

**Example Request (JSON with base64):**

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

**Success Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "File uploaded successfully",
    "data": {
        "id": 123,
        "path": "/storage/app/uploads/public/.../original.jpg",
        "thumb": "/storage/app/uploads/public/.../thumb.jpg"
    },
    "input_data": { ... },
    "process_data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "temp_session_key": "tmp_abc123...",
        "user_type": "backend",
        "storage_disk": "s3"
    }
}
```

**Error Response (Permission Denied - 403):**

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

---

### 2. Upload Multiple Files for a Multiple Field

**Method:** `POST`  
**Path:** `/upload-multiple`

**Request Parameters (Body):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified class name of the model. |
| `field` | `string` | Yes | Name of the multiple field (must be of type `multiple` in the registration). |
| `files` | `array` | Yes | Array of files (each element can be an uploaded file object or a base64 string). |
| `temp_session_key` | `string` | No | Temporary session key (for unsaved models). |
| `model_id` | `int` | No | ID of the saved model. |

**Example Request (multipart):**

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
------WebKitFormBoundary--
```

**Success Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "Successfully uploaded 2 out of 2 files",
    "data": [
        {
            "status": true,
            "data": { "id": 101, "path": "...", "thumb": "..." }
        },
        {
            "status": true,
            "data": { "id": 102, "path": "...", "thumb": "..." }
        }
    ],
    "process_data": {
        "success_count": 2,
        "total": 2
    }
}
```

**Partial Response (some files failed - 200 with details):**

```json
{
    "code": 200,
    "status": true,
    "message": "Successfully uploaded 1 out of 2 files",
    "data": [
        {
            "status": true,
            "data": { "id": 101 }
        },
        {
            "status": false,
            "error": "File type not allowed",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        }
    ],
    "process_data": {
        "success_count": 1,
        "total": 2
    }
}
```

---

### 3. Delete File

**Method:** `DELETE`  
**Path:** `/delete/{id}`

**Path Parameters (URL):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `int` | Yes | ID of the file to delete. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | No | Model name (to check delete permission). |
| `field` | `string` | No | Field name (to check delete permission). |

**Example Request:**

```http
DELETE /api/v1/fileupload/delete/123?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

**Success Response (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "File deleted",
    "data": null
}
```

**Error Response (File Not Found - 404):**

```json
{
    "code": 404,
    "status": false,
    "message": "File not found",
    "error": "File not found",
    "error_code": "FILE_UPLOAD_FILE_NOT_FOUND"
}
```

**Error Response (Permission Denied - 403):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to delete this file",
    "error": "You do not have permission to delete this file",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

---

### 4. Retrieve Associated Files

**Method:** `GET`  
**Path:** `/files`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified class name of the model. |
| `field` | `string` | Yes | Field name. |
| `model_id` | `int` | No | ID of the saved model (if it exists). |
| `temp_session_key` | `string` | No | Temporary session key (to retrieve unlinked temporary files). |
| `with_thumbs` | `bool` | No | Include thumbnails (`true` / `false`, default `false`). |
| `thumb_sizes` | `object` | No | Thumbnail sizes (array of size names and dimensions). |

**Example Request (retrieve product gallery images with thumbnails):**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10&with_thumbs=true&thumb_sizes[small][0]=150&thumb_sizes[small][1]=150&thumb_sizes[small][2]=crop HTTP/1.1
Authorization: Bearer ...
```

**Success Response (200):**

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
            "path": "/storage/app/uploads/public/.../original.jpg",
            "size": 102400,
            "content_type": "image/jpeg",
            "small": "/storage/app/uploads/public/.../small.jpg"
        },
        {
            "id": 102,
            "title": null,
            "description": null,
            "path": "/storage/app/uploads/public/.../original.png",
            "size": 204800,
            "content_type": "image/png",
            "small": "/storage/app/uploads/public/.../small.png"
        }
    ]
}
```

**Error Response (Invalid Temporary Key - 400):**

```json
{
    "code": 400,
    "status": false,
    "message": "Invalid or expired temporary session key",
    "error": "Invalid or expired temporary session key",
    "error_code": "FILE_UPLOAD_TEMP_KEY_INVALID"
}
```

---

## Common Error Codes (`error_code`)

| error_code | HTTP Code | Meaning | Solution |
|------------|-----------|---------|----------|
| `FILE_UPLOAD_PERMISSION_DENIED` | 403 | User does not have permission for the operation | Check user or field permissions. |
| `FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY` | 403 | Upload operations are globally disabled | Wait until enabled or review settings. |
| `FILE_UPLOAD_DELETE_DISABLED_GLOBALLY` | 403 | Delete operations are globally disabled | Wait until enabled. |
| `FILE_UPLOAD_GET_DISABLED_GLOBALLY` | 403 | Retrieve operations are globally disabled | Wait until enabled. |
| `FILE_UPLOAD_MODEL_NOT_REGISTERED` | 400 | Model not registered in the system | Verify `model_class` is correct. |
| `FILE_UPLOAD_FIELD_NOT_REGISTERED` | 400 | Field not registered for the model | Verify `field` is correct and registered. |
| `FILE_UPLOAD_MODEL_NOT_FOUND` | 404 | Model not found (when `model_id` is provided) | Check if the model exists in the database. |
| `FILE_UPLOAD_FILE_NOT_FOUND` | 404 | File not found | Verify the `id` is correct. |
| `FILE_UPLOAD_INVALID_FILE_DATA` | 400 | Invalid file data | Ensure a valid file or base64 is sent. |
| `FILE_UPLOAD_FILE_SIZE_EXCEEDED` | 400 | File size exceeds allowed limit | Use a smaller file or increase the limit. |
| `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` | 400 | File type not allowed | Use an allowed type (see `allowed_types`). |
| `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` | 400 | Dangerous file type (PHP, JS, HTML, etc.) | Use a safe file type. |
| `FILE_UPLOAD_FILE_UPLOAD_FAILED` | 500 | File upload failed (internal error) | Check server logs. |
| `FILE_UPLOAD_TEMP_KEY_INVALID` | 400 | Invalid temporary key | Re-upload the file or use a new key. |
| `FILE_UPLOAD_TEMP_KEY_EXPIRED` | 400 | Temporary key expired | Re-upload the file. |
| `FILE_UPLOAD_TEMP_KEY_MISMATCH` | 403 | Temporary key does not belong to the current user | Ensure you are using the correct key. |
| `FILE_UPLOAD_USER_TYPE_NOT_ALLOWED` | 403 | User type not allowed | Check `allowed_user_types` in the registration. |

---

## Comprehensive Practical Examples

### Example 1: Upload an Image for a New Product (Using a Temporary Session Key)

**Step 1:** Upload the image and get `temp_session_key`

```http
POST /api/v1/fileupload/upload
{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,..."
}
```

**Response:**

```json
{
    "status": true,
    "data": { "id": 101, "path": "..." },
    "temp_session_key": "tmp_abc123"
}
```

**Step 2:** Create the product (in your application) and save it

```php
$product = new Product();
$product->name = 'New product';
$product->save();
```

**Step 3:** Attach the image to the product via `attachTempFiles` (done server-side, not directly via API).  
You can add a custom endpoint for this purpose or execute it in server-side code.

---

### Example 2: Upload Multiple Images for an Existing Product Gallery

```http
POST /api/v1/fileupload/upload-multiple
Content-Type: multipart/form-data

model_class: Nano\Shop\Models\Product
field: gallery
model_id: 10
files[]: (file1)
files[]: (file2)
```

**Response:**

```json
{
    "status": true,
    "message": "Successfully uploaded 2 out of 2 files",
    "data": [ { "data": { "id": 102 } }, { "data": { "id": 103 } } ]
}
```

---

### Example 3: Delete a Main Image from a Product

```http
DELETE /api/v1/fileupload/delete/101?model_class=Nano\Shop\Models\Product&field=image
```

**Response:**

```json
{
    "status": true,
    "message": "File deleted"
}
```

---

### Example 4: Retrieve Product Gallery Images with Thumbnails

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10&with_thumbs=true&thumb_sizes[medium][0]=300&thumb_sizes[medium][1]=300&thumb_sizes[medium][2]=crop
```

**Response:**

```json
{
    "status": true,
    "data": [
        {
            "id": 102,
            "path": "...",
            "medium": "/storage/.../medium.jpg"
        }
    ]
}
```

---

## Best Practices

1. **Use a temporary session key for new models**: to avoid losing files if the model is not saved yet.
2. **Validate input before sending the request**: especially `model_class` and `field` to ensure they exist in the registration.
3. **Handle errors by `error_code`**: provide appropriate user messages for each case (e.g., `FILE_UPLOAD_PERMISSION_DENIED` → "Unauthorized").
4. **Use `with_thumbs` only when needed**: to reduce response size and increase performance.
5. **Enable `app.debug` in development environment**: to get full error details (appear in the `debug` field).
6. **Keep temporary session keys in the session or database**: to use them later for attaching files.
7. **Monitor the `fileupload.log`**: to detect attempts to upload dangerous files or recurring errors.
8. **Use appropriate `thumb_sizes`**: to avoid creating too many thumbnails that consume storage space.

---

## Additional Documentation

- [General Add-on Documentation](./Docs-FileUpload.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class.md)
- [Advanced Examples for `FileUploadRegistry` Class](./Docs-FileUploadRegistry-Class-Advenced-Examples.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class.md)
- [Advanced Examples for `FileUploadService` Class](./Docs-FileUploadService-Class-Advenced-Examples.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class.md)
- [Advanced Examples for `FileUploadUserManager` Class](./Docs-FileUploadUserManager-Class-Advenced-Examples.md)

