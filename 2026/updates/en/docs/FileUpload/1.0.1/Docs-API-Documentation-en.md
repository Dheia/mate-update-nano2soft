# File Upload API Documentation

**Product:** NanoSoft  
**Version:** 1.0  
**Base Path:** `/api/v1/fileupload`  
**Authentication:** OAuth 2.0 (access token in header `Authorization: Bearer <token>`)

---

## Overview

This API provides a set of endpoints for managing file uploads in NanoSoft applications, relying on the `Nano.FileUpload` system. These services allow developers to upload a single file or multiple files, delete specific files, and retrieve files associated with certain models, with automatic permission checks, user types, and pre‑registered constraints.

All responses follow the unified structure described below, making them easy to handle in client applications.

---

## Unified Response Structure

Every API response has the following structure:

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
    "debug": {}
}
```

- `code`: HTTP status code (200 for success, 400/403/404/500 for failure).
- `status`: `true` for success, `false` for failure.
- `message`: Explanatory message (in Arabic in the original, but the structure remains).
- `error`: Error details (if failure).
- `errors`: Array of validation errors (if any).
- `data`: Core response data (e.g., uploaded file information).
- `meta`: Additional data (e.g., pagination info) – currently unused.
- `input_data`: Copy of the user‑sent data (for tracking).
- `process_data`: Internal processing data (e.g., temporary session key).
- `debug`: Debug information (only appears when `app.debug` is enabled).

---

## Authentication

All endpoints are protected by **OAuth 2.0**. The access token must be included in the request header:

```
Authorization: Bearer <your_access_token>
```

If the token is not provided or is invalid, you will receive a response with code `401` and a message "Authentication failed".

---

## Endpoints

### 1. Upload a Single File

**Method:** `POST`  
**Path:** `/upload`

**Request Parameters (Body):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified model class name (e.g., `Nano\Shop\Models\Product`). |
| `field` | `string` | Yes | Field name as registered in `FileUploadRegistry`. |
| `file` | `file` | Conditional* | Uploaded file (multipart). |
| `file_base64` | `string` | Conditional* | File data as base64 string (alternative to `file`). |
| `temp_session_key` | `string` | No | Temporary session key to link the file later (used with unsaved models). |
| `model_id` | `int` | No | ID of the saved model (if exists). |

> * Either `file` or `file_base64` must be provided.

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
        "temp_session_key": null,
        "user_type": "backend"
    }
}
```

**Error Response (Permission Denied – 403):**

```json
{
    "code": 403,
    "status": false,
    "message": "You do not have permission to upload files for this field",
    "error": "You do not have permission to upload files for this field",
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
| `model_class` | `string` | Yes | Fully qualified model class name. |
| `field` | `string` | Yes | Multiple field name (must be of type `multiple` in registration). |
| `files` | `array` | Yes | Array of files (each can be an uploaded file object or base64 string). |
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

---

### 3. Delete a File

**Method:** `DELETE`  
**Path:** `/delete/{id}`

**Path Parameters (URL):**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `int` | Yes | ID of the file to delete. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | No | Model class name (for permission check). |
| `field` | `string` | No | Field name (for permission check). |

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

**Error Response (File Not Found – 404):**

```json
{
    "code": 404,
    "status": false,
    "message": "File not found",
    "error": "File not found"
}
```

---

### 4. Retrieve Associated Files

**Method:** `GET`  
**Path:** `/files`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_class` | `string` | Yes | Fully qualified model class name. |
| `field` | `string` | Yes | Field name. |
| `model_id` | `int` | No | ID of the saved model (if exists). |
| `temp_session_key` | `string` | No | Temporary session key (to retrieve temporary unassociated files). |
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

---

## Common Error Codes

| Code | Meaning | Solution |
|------|---------|----------|
| 400 | Invalid data (missing parameters, validation failed) | Check input validity. |
| 401 | Unauthorized (invalid or expired token) | Re‑authenticate and obtain a new token. |
| 403 | Permission denied (user does not have permission for the operation) | Verify user permissions or field settings. |
| 404 | Item not found (model, field, file) | Ensure the identifier is correct. |
| 500 | Internal server error | Check server logs with `app.debug` enabled. |

---

## Practical Examples

### Example 1: Upload an Image for a New Product (Using Temporary Session Key)

**Step 1:** Upload the image and obtain a `temp_session_key`

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

**Step 3:** Link the image to the product via `attachTempFiles` (done on the server, not directly via API). You can add a custom endpoint or execute it in server code.

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

### Example 3: Delete a Main Product Image

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

### Example 4: Retrieve a Product Gallery with Thumbnails

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
2. **Validate input before sending the request**: especially `model_class` and `field` to ensure they are registered.
3. **Handle errors by code**: provide appropriate messages to the user for each case (401: log in, 403: no permission, 404: not found).
4. **Use `with_thumbs` only when needed**: to reduce response size and improve performance.
5. **Enable `app.debug` in development**: to get full error details.
6. **Store temporary session keys in session or database**: to use them later for file linking.

---
## Additional Documentation

- [General Plugin Documentation](./Docs-FileUpload-en.md)
- [`FileUploadRegistry` Class Documentation](./Docs-FileUploadRegistry-Class-en.md)
- [Advanced Examples for `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-en.md)
- [`FileUploadService` Class Documentation](./Docs-FileUploadService-Class-en.md)
- [Advanced Examples for `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-en.md)
- [`FileUploadUserManager` Class Documentation](./Docs-FileUploadUserManager-Class-en.md)
- [Advanced Examples for `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-en.md)

