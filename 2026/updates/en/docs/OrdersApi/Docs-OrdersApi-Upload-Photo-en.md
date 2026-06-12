# API Documentation for Order Photo Management

**Version:** 1.0.22  
**Base Path:** `/api/v1/orders`  
**Authentication:** OAuth 2.0 (access token in header `Authorization: Bearer <token>`)

---

## Overview

These three endpoints provide the ability to manage photos associated with an order, used specifically for rental scenarios where the owner (`user_id`) and the delivery person (`delivery_user_id`) need to upload photos before and after delivery. Permission checks are based on:

- **User role** (order owner / delivery person / admin).
- **Current order status** (NEW, PROCESSING, DELIVERY, COMPLETE).
- **Maximum number of photos** (10 per field by default).
- **File size and types** (jpg, jpeg, png, gif up to 5 MB).

> **Note:** These endpoints rely on the `StepPhotos` trait in the `Nano.Orders` plugin version 2.2.12 and above.

---

## 1. Upload a Single Photo

**Method:** `POST`  
**Path:** `/upload-photo`  
**Content-Type:** `multipart/form-data` or `application/json`

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `order_id` | `int` | Yes | ID of the order associated with the photo. |
| `field` | `string` | Yes | Field name: `user_before_delivery`, `user_after_delivery`, `delivery_before_delivery`, `delivery_after_delivery`. |
| `file` | `file` | Conditional* | Uploaded file (multipart format). |
| `file_base64` | `string` | Conditional* | File data as base64 (alternative to `file`). |
| `title` | `string` | No | Photo title. |
| `description` | `string` | No | Photo description. |
| `is_public` | `bool` | No | Whether the photo is public (default `false`). |
| `auto_resize` | `bool` | No | Enable automatic resizing. |
| `resize_options` | `object` | No | Resize settings (e.g., `{"width":800,"height":600,"mode":"crop"}`). |
| `auto_watermark` | `bool` | No | Enable watermark (requires Watermark plugin). |
| `custom_message` | `string` | No | Custom success message. |

> * Either `file` (multipart) or `file_base64` (JSON) must be provided.

### Request Examples

#### Example 1: Upload photo using multipart/form-data (order owner)

```http
POST /api/v1/orders/upload-photo HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="order_id"

125
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

user_before_delivery
------WebKitFormBoundary
Content-Disposition: form-data; name="title"

Photo before delivery
------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="product.jpg"
Content-Type: image/jpeg

(binary file data)
------WebKitFormBoundary--
```

#### Example 2: Upload photo using JSON and base64 (delivery person)

```json
POST /api/v1/orders/upload-photo HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "order_id": 125,
    "field": "delivery_before_delivery",
    "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
    "title": "Photo before delivery by courier",
    "description": "Uploaded by courier"
}
```

### Responses

#### ✅ Success response (single photo upload)

```json
{
    "code": 200,
    "status": true,
    "message": "Photo uploaded successfully",
    "data": {
        "id": 456,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg",
        "photos": {
            "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg",
            "medium": "https://yourdomain.com/storage/app/uploads/public/.../medium.jpg",
            "large": "https://yourdomain.com/storage/app/uploads/public/.../large.jpg"
        },
        "title": "Photo before delivery",
        "description": null,
        "size": 123456,
        "content_type": "image/jpeg",
        "created_at": "2026-06-05 10:30:00"
    },
    "input_data": { ... },
    "process_data": { ... }
}
```

#### ❌ Error response (forbidden – user role not allowed)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to upload photo",
    "error": "User of type user is not allowed to upload photos in field delivery_before_delivery",
    "error_code": null
}
```

#### ❌ Error response (order status not allowed for upload)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to upload photo",
    "error": "Cannot upload photos in field user_before_delivery in current status (COMPLETE)",
    "error_code": null
}
```

#### ❌ Error response (maximum number of photos reached)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to upload photo",
    "error": "Maximum number of photos (10) reached in field user_before_delivery",
    "error_code": null
}
```

#### ❌ Error response (file missing)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to upload photo",
    "error": "File is required",
    "error_code": null
}
```

#### ❌ Error response (unauthorised – token missing)

```json
{
    "code": 401,
    "status": false,
    "message": "Failed to upload photo",
    "error": "User not found",
    "error_code": null
}
```

---

## 2. Upload Multiple Photos (for a multi‑field)

**Method:** `POST`  
**Path:** `/upload-multiple-photos`  
**Content-Type:** `multipart/form-data` or `application/json`

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `order_id` | `int` | Yes | ID of the order associated with the photos. |
| `field` | `string` | Yes | Field name (same values as above). |
| `files` | `array` | Yes | Array of files (each element either an `UploadedFile` object or a base64 string). |
| `title` | `string` | No | Generic title for all photos (applies to all). |
| `description` | `string` | No | Generic description for all photos. |
| `is_public` | `bool` | No | Whether the photos are public (default `false`). |
| `auto_resize` | `bool` | No | Enable automatic resizing for all photos. |

### Example Request (multipart/form-data)

```http
POST /api/v1/orders/upload-multiple-photos HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="order_id"

125
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

user_after_delivery
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="photo1.jpg"
Content-Type: image/jpeg

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="photo2.png"
Content-Type: image/png

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="photo3.jpg"
Content-Type: image/jpeg

...
------WebKitFormBoundary--
```

### Example Request (JSON with base64)

```json
POST /api/v1/orders/upload-multiple-photos HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "order_id": 125,
    "field": "user_after_delivery",
    "files": [
        "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
        "data:image/png;base64,iVBORw0KGgo...",
        "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
    ],
    "title": "Photos after delivery"
}
```

### Responses

#### ✅ Success response (all photos uploaded successfully)

```json
{
    "code": 200,
    "status": true,
    "message": "All photos uploaded successfully",
    "data": [
        {
            "status": true,
            "data": {
                "id": 456,
                "path": "https://.../photo1.jpg",
                "thumb": "https://.../thumb1.jpg"
            }
        },
        {
            "status": true,
            "data": {
                "id": 457,
                "path": "https://.../photo2.png",
                "thumb": "https://.../thumb2.jpg"
            }
        },
        {
            "status": true,
            "data": {
                "id": 458,
                "path": "https://.../photo3.jpg",
                "thumb": "https://.../thumb3.jpg"
            }
        }
    ],
    "process_data": {
        "success_count": 3,
        "total": 3
    },
    "errors": []
}
```

#### ⚠️ Partial success response (some photos failed)

```json
{
    "code": 207,
    "status": true,
    "message": "2 out of 3 photos uploaded successfully",
    "data": [
        {
            "status": true,
            "data": {
                "id": 456,
                "path": "https://.../photo1.jpg"
            }
        },
        {
            "status": true,
            "data": {
                "id": 457,
                "path": "https://.../photo2.png"
            }
        },
        {
            "status": false,
            "error": "File is required",
            "data": null
        }
    ],
    "process_data": {
        "success_count": 2,
        "total": 3
    },
    "errors": {
        "2": "File is required"
    }
}
```

#### ❌ Complete failure response (all photos failed)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to upload photo",
    "data": null,
    "process_data": {
        "success_count": 0,
        "total": 2
    },
    "errors": {
        "0": "File type not allowed",
        "1": "File size exceeds the allowed limit"
    }
}
```

---

## 3. Delete a Photo

**Method:** `POST` or `DELETE`  
**Path:** 
- `/delete-photo` (POST) – parameters passed in the request body (JSON).
- `/delete-photo/{orderId}/{field}/{fileId}` (DELETE) – parameters passed in the path.

### Request Parameters (POST JSON)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `order_id` | `int` | Yes | Order ID. |
| `field` | `string` | Yes | Field name containing the photo. |
| `file_id` | `int` | Yes | ID of the photo to delete. |

### Path Parameters (DELETE)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `orderId` | `int` | Yes | Order ID. |
| `field` | `string` | Yes | Field name. |
| `fileId` | `int` | Yes | Photo ID. |

### Request Examples

#### Example 1: Delete using POST JSON

```http
POST /api/v1/orders/delete-photo HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "order_id": 125,
    "field": "user_before_delivery",
    "file_id": 456
}
```

#### Example 2: Delete using DELETE with path parameters

```http
DELETE /api/v1/orders/delete-photo/125/user_before_delivery/456 HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

### Responses

#### ✅ Success response (photo deleted)

```json
{
    "code": 200,
    "status": true,
    "message": "Photo deleted successfully",
    "data": {
        "deleted_file_id": 456
    },
    "error": null,
    "errors": null
}
```

#### ❌ Error response (file not found)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to delete photo",
    "error": "Photo does not exist",
    "data": null
}
```

#### ❌ Error response (forbidden – user not the owner of the photo)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to delete photo",
    "error": "User of type delivery is not allowed to upload photos in field user_before_delivery",
    "data": null
}
```

#### ❌ Error response (missing photo ID)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to delete photo",
    "error": "File ID is required",
    "data": null
}
```

---

## 4. Field Restrictions and Types

| Field | Authorised User | Allowed Statuses | Maximum Photos |
|-------|-----------------|------------------|----------------|
| `user_before_delivery` | `user` (order owner) | `NEW`, `PROCESSING` | 10 |
| `user_after_delivery` | `user` (order owner) | `DELIVERY` | 10 |
| `delivery_before_delivery` | `delivery` (courier) | `NEW`, `PROCESSING` | 10 |
| `delivery_after_delivery` | `delivery` (courier) | `DELIVERY` | 10 |

> **Note:** The admin has full permission to upload photos in all fields and any status.

---

## 5. Best Practices

1. **Use `multipart/form-data` when uploading files directly** – it is more efficient than base64.
2. **Use `file_base64` only when necessary** (e.g., applications that do not support multipart or when combining photos from other data).
3. **Make sure the order status is appropriate for uploading photos** before calling the endpoint (you can check beforehand using `GET /orders/{id}` and inspecting `order_states_ref_type`).
4. **Handle `code = 207` in `upload-multiple-photos`** as a partial success, showing errors for each failed file.
5. **To delete photos, prefer using the `DELETE` method** because it better expresses the intent of the operation (RESTful).
6. **Ensure the user has delete permission** – only an admin can delete photos belonging to other parties.

---

## 6. Common Error Codes (HTTP Codes)

| HTTP Code | Meaning |
|-----------|---------|
| `200` | Success (all photos uploaded / deleted). |
| `207` | Partial success (some photos uploaded, some failed). |
| `400` | Bad request (invalid data, permission denied, invalid file). |
| `401` | Unauthorised (OAuth token missing or invalid). |
| `404` | Order or photo not found. |
| `500` | Internal server error (details appear in `debug` when debug mode is enabled). |

---

## 7. Integration with `update-status`

When attempting to change the order status to `DELIVERY` or `COMPLETE` via the `update-status` endpoint, the request will be rejected if the required photos are missing, with a message indicating the required field.

**Example of status change failure due to missing photos:**

```http
POST /api/v1/orders/update-status
Content-Type: application/json

{
    "order_id": 125,
    "order_states_ref_type": "DELIVERY"
}
```

**Response:**

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to update order status.",
    "error": "You must upload photos in field user_before_delivery before changing the status to DELIVERY",
    "data": null
}
```

To bypass this check (for administrators only), you can pass `skip_photo_check: true`:

```json
{
    "order_id": 125,
    "order_states_ref_type": "DELIVERY",
    "skip_photo_check": true,
    "admin_override": true
}
```

---

## 8. Complete Practical Examples

### Scenario 1: Order owner uploads photos before delivery

**Request (multipart):**
```http
POST /api/v1/orders/upload-photo
Authorization: Bearer {user_token}
Content-Type: multipart/form-data

order_id: 125
field: user_before_delivery
file: @/home/user/photo.jpg
title: Photo of the item before delivery
```

**Expected response (after success):**
```json
{
    "code": 200,
    "status": true,
    "message": "Photo uploaded successfully",
    "data": {
        "id": 456,
        "path": "https://example.com/storage/.../photo.jpg",
        "thumb": "https://example.com/storage/.../thumb.jpg",
        "photos": {
            "thumb": "https://example.com/storage/.../thumb.jpg",
            "medium": "https://example.com/storage/.../medium.jpg",
            "large": "https://example.com/storage/.../large.jpg"
        }
    }
}
```

### Scenario 2: Delivery person uploads multiple photos before delivery

**Request (JSON with base64):**
```json
POST /api/v1/orders/upload-multiple-photos
Authorization: Bearer {delivery_token}
Content-Type: application/json

{
    "order_id": 125,
    "field": "delivery_before_delivery",
    "files": ["data:image/jpeg;base64,...", "data:image/jpeg;base64,..."],
    "title": "Photos from the courier before delivery"
}
```

**Expected response (partial success – second file failed due to base64 issue):**
```json
{
    "code": 207,
    "status": true,
    "message": "1 out of 2 photos uploaded successfully",
    "data": [
        {
            "status": true,
            "data": {
                "id": 457,
                "path": "https://example.com/storage/.../photo1.jpg"
            }
        },
        {
            "status": false,
            "error": "Invalid file data",
            "data": null
        }
    ],
    "process_data": {
        "success_count": 1,
        "total": 2
    }
}
```

### Scenario 3: Delete a photo (delivery user deletes only his own photo)

**Request (DELETE):**
```http
DELETE /api/v1/orders/delete-photo/125/delivery_before_delivery/457
Authorization: Bearer {delivery_token}
```

**Expected response (success):**
```json
{
    "code": 200,
    "status": true,
    "message": "Photo deleted successfully",
    "data": {
        "deleted_file_id": 457
    }
}
```

### Scenario 4: Attempt to upload a photo after the order is completed (rejected)

**Request:**
```http
POST /api/v1/orders/upload-photo
Authorization: Bearer {user_token}
Content-Type: multipart/form-data

order_id: 125
field: user_before_delivery
file: @photo.jpg
```

**Response (since the order status is now `COMPLETE`):**
```json
{
    "code": 400,
    "status": false,
    "message": "Failed to upload photo",
    "error": "Cannot upload photos in field user_before_delivery in current status (COMPLETE)"
}
```

---

## 9. Additional Notes

- **Default file size:** 5 MB (5120 KB). Can be changed via `photo_rules` in `config/nano/orders.php`.
- **Allowed types:** `jpg, jpeg, png, gif` (by default). You can add `webp` or other types via configuration.
- **Thumbnails:** Automatically generated in sizes `thumb` (150×150), `medium` (300×300), `large` (800×600). You can request custom sizes via `thumb_sizes` in the options (not documented here, but available in the `getOrderPhotoUrls` function).
- **Events:** When a photo is successfully uploaded, the event `nano.orders.photo.uploaded` is fired. When deleted, `nano.orders.photo.deleted` is fired; you can listen to these to extend behaviour.
- **Translation:** All error and success messages are translatable via the `lang` files of `Nano.OrdersApi`.

---

**Reference documentation:**
- [`StepPhotos` trait documentation](./docs/Orders/Traits/Steps/Docs-StepPhotos-Trait-en.md)
- [`OrderManager` and its attributes documentation](./docs/Orders/Classes/Docs-OrderManager-Class-en.md)
- [Order status update API documentation](./docs/OrdersApi/Docs-OrdersApi-UpdateStatus-en.md)
