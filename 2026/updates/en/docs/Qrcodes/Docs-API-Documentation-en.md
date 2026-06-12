# API Documentation – Barcode Management System (QRcodes)

## 1. Introduction

The API for barcode management within the **Nano2.Qrcodes** plugin provides a comprehensive set of endpoints that enable developers to create barcodes of various types (1D such as Code 128, EAN-13, and 2D such as QR codes), verify their validity, record their usage, retrieve usage statistics, and generate barcode images.

The API follows a RESTful structure and supports JSON format. It is secured by an advanced permission system (`AccessManager`) that distinguishes between backend users (control panel), frontend users (regular website users), and guests (unauthenticated visitors). Permissions can be customised via the `config.php` file using environment variables.

### Main Features:
- **Full CRUD operations** for barcodes (create, read, update, delete).
- **Verify a barcode** without using it (check validity – active, not expired, not used).
- **Scan and use a barcode** (increment usage counter and link to the user).
- **Administrative verification** (set `is_verified` by an admin).
- **Batch generation** of many barcodes at once (up to 1000).
- **Statistics** about barcodes and per‑user usage statistics.
- **Generate barcode image** (PNG) either directly or via a URL.
- **Caching support** for repeated requests (can be enabled via settings).

### Technical basics:
- **Base path**: `/api/v1/qrcodes`
- **Authentication**: The user is automatically recognised through the session or the authentication token from `Nano\AuthApi`. Permissions depend on the user’s role and specific permissions defined in `config.php`.
- **Request and response format**: `application/json`
- **General success code**: `200`
- **Common error codes**: `400`, `401`, `404`.

---

## 2. Endpoint Index

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/barcodes` | Fetch a list of barcodes with filtering and pagination. |
| GET | `/barcodes/activelystats` | Get the latest data update timestamp (for use with caching). |
| POST | `/barcodes` | Create a new barcode. |
| PUT | `/barcodes/{id}` | Update an existing barcode. |
| DELETE | `/barcodes/{id}` | Soft‑delete a barcode. |
| GET | `/barcodes/{id}` | View details of a specific barcode. |
| POST | `/barcodes/verify` | Verify a barcode (without using it). |
| POST | `/barcodes/use` | Scan and use a barcode (record usage). |
| POST | `/barcodes/verify/{id}` | Administrative verification of a barcode (set `is_verified`). |
| POST | `/barcodes/generate` | Generate a batch of barcodes (admin only). |
| GET | `/barcodes/stats` | General barcode statistics. |
| GET | `/barcodes/user-stats` | Usage statistics for the current user. |
| GET | `/barcodes/image/{id}` | Get the barcode image (PNG). |

---

## 3. Endpoint Details

### 3.1. Fetch List of Barcodes

#### **`GET /barcodes`**

**Introduction**:  
This endpoint fetches a list of barcodes with support for filtering, searching, ordering, and pagination. Access scopes are automatically applied so that each user only sees the barcodes they are allowed to see (their own, those belonging to their department, or all depending on permissions).

**Parameters** (all optional, passed as query string):

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `page` | int | Page number (for pagination) | `page=2` |
| `per_page` | int | Number of results per page (max 100) | `per_page=20` |
| `orderBy` | string | Field name to order by (e.g., `created_at`, `code`, `barcode`) | `orderBy=code` |
| `orderDirection` | string | `asc` or `desc` | `orderDirection=desc` |
| `search` | string | Search text (in fields: `code`, `barcode`, `product_name`, `product_sku`) | `search=ABC` |
| `barcode_type` | string | Barcode type (`C128`, `EAN13`, `QR`, ...) | `barcode_type=QR` |
| `status` | string | Status (`active`, `inactive`, `used`, `expired`) | `status=active` |
| `is_active` | bool | `true` or `false` | `is_active=true` |
| `is_verified` | bool | `true` or `false` | `is_verified=true` |
| `is_use` | bool | `true` or `false` | `is_use=false` |
| `companys_id` | int | Company ID | `companys_id=2` |
| `departments_id` | int | Department ID | `departments_id=5` |
| `product_id` | int | Product ID | `product_id=12` |
| `owner_id` | int | Owner ID | `owner_id=7` |
| `owner_type` | string | Owner type (e.g., `RainLab\User\Models\User`) | `owner_type=RainLab\User\Models\User` |
| `issue_date_from` | date | Issue date from | `issue_date_from=2025-01-01` |
| `issue_date_to` | date | Issue date to | `issue_date_to=2025-12-31` |
| `expiry_date_from` | date | Expiry date from | `expiry_date_from=2025-01-01` |
| `expiry_date_to` | date | Expiry date to | `expiry_date_to=2025-12-31` |
| `exclude` | string | Comma‑separated columns to exclude (will not appear in the response) | `exclude=metadata,other_data` |
| `advanced_filters` | - | You can use `is_or_*`, `is_not_*`, `is_or_null_*` for allowed fields (see permissions section) | `is_or_status=true` |

**Practical example**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes?page=1&per_page=5&barcode_type=C128&status=active&orderBy=created_at&orderDirection=desc"
```

**Successful response (200 OK)**:

```json
{
    "data": [
        {
            "id": 10,
            "code": "BC000010",
            "barcode": "BC000010",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "used",
            "status_color": "info",
            "status_icon": "icon-check-circle text-info",
            "is_active": true,
            "is_verified": false,
            "is_use": true,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 1,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": "2026-06-04 20:45:17",
            "last_used_at": "2026-06-04 20:45:17",
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 20:45:17"
        },
        // ... more barcode objects
    ],
    "meta": {
        "pagination": {
            "total": 10,
            "count": 10,
            "per_page": 20,
            "current_page": 1,
            "total_pages": 1,
            "links": {}
        }
    }
}
```

**Failure response (401 – Unauthorised)**:

```json
{
  "code": "UNAUTHORIZED",
  "http_code": 401,
  "status": false,
  "message": "You do not have the required permissions to perform 'list'.",
  "error": "You do not have the required permissions to perform 'list'."
}
```

---

### 3.2. Get Last Update (for Cache)

#### **`GET /barcodes/activelystats`**

**Introduction**:  
A lightweight endpoint that returns the latest timestamp when any barcode was updated (`updated_at`). It is used by the client to decide whether to reload cached data.

**Parameters**: None.

**Example**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/activelystats"
```

**Successful response (200)**:

```json
{
  "code": 200,
  "data": {
    "last_updated": "2025-03-15 14:30:00",
    "activity_stats": true
  }
}
```

---

### 3.3. Create a New Barcode

#### **`POST /barcodes`**

**Introduction**:  
Create a new barcode. The required fields are `barcode` (the encoded value) and `barcode_type`. A `code` is automatically generated if not provided (unique). The barcode can be linked to a product, owner, or a specific user. Create permission is typically for backend users only (can be customised).

**Parameters** (passed in JSON request body):

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `barcode` | string | Yes | The value to be encoded inside the barcode. |
| `barcode_type` | string | Yes | The type (`C128`, `QR`, `EAN13`, ...). |
| `code` | string | No | Optional unique code (auto‑generated if not given). |
| `product_id` | int | No | Related product ID. |
| `product_name` | string | No | Product name. |
| `price` | float | No | Price. |
| `quantity` | float | No | Quantity (default 1). |
| `owner_id` | int | No | Owner ID. |
| `owner_type` | string | No | Owner type (automatically filled from the current user if `create`). |
| `user_id` | int | No | ID of the user who will later record usage. |
| `issue_date` | date | No | Issue date. |
| `expiry_date` | date | No | Expiry date. |
| `status` | string | No | `active`, `inactive`, `pending` (default `active`). |
| `is_active` | bool | No | `true` (default). |
| `is_published` | bool | No | `false` (default). |
| `is_verified` | bool | No | `false`. |
| `batch_number` | string | No | Batch number. |
| `companys_id` | int | No | Company ID (taken from default settings if not provided). |
| `departments_id` | int | No | Department ID. |
| `fields_data`, `metadata`, ... | object | No | Additional JSON data. |

**Example**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes" \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "5901234123457",
    "barcode_type": "EAN13",
    "product_name": "Programming Book",
    "price": 45.00,
    "quantity": 1,
    "expiry_date": "2028-12-31"
  }'
```

**Successful response (200)**:

```json
{
  "code": 200,
  "status": true,
  "message": "Barcode created successfully.",
  "error": null,
  "errors": null,
  "data": {
    "id": 110,
    "code": "BC000110",
    "barcode": "5901234123457",
    "barcode_type": "EAN13",
    "status": "active",
    ...
  },
  "input_data": { ... },
  "process_data": { ... }
}
```

**Failure response (400 – missing or invalid data)**:

```json
{
  "code": 400,
  "status": false,
  "message": "An error occurred while creating the barcode.",
  "error": "The barcode field is required.",
  "errors": { "barcode": ["The barcode field is required."] }
}
```

---

### 3.4. Update an Existing Barcode

#### **`PUT /barcodes/{id}`**

**Introduction**:  
Update data for a specific barcode. A barcode that has already been used (`is_use = true`) cannot be modified unless `force_update=true` is passed. Only the fields provided will be updated.

**Parameters**: Same as for `POST /barcodes`, but partial fields can be sent.

**Example**:

```bash
curl -X PUT "https://yourdomain.com/api/v1/qrcodes/barcodes/110" \
  -H "Content-Type: application/json" \
  -d '{
    "price": 49.99,
    "is_published": true
  }'
```

**Successful response (200)**:

```json
{
  "code": 200,
  "status": true,
  "message": "Barcode updated successfully.",
  "error": null,
  "data": { ... }
}
```

---

### 3.5. Delete a Barcode (Soft Delete)

#### **`DELETE /barcodes/{id}`**

**Introduction**:  
Soft‑delete the barcode – the record remains in the database with a `deleted_at` timestamp. It cannot be restored via the API (only via the backend control panel).

**Parameters**: None.

**Example**:

```bash
curl -X DELETE "https://yourdomain.com/api/v1/qrcodes/barcodes/110"
```

**Successful response (200)**:

```json
{
  "code": 200,
  "status": true,
  "message": "Barcode deleted successfully.",
  "error": null,
  "data": null
}
```

**Failure response (404)**:

```json
{
  "code": 400,
  "status": false,
  "message": "An error occurred while deleting the barcode.",
  "error": "The requested barcode does not exist."
}
```

---

### 3.6. View Barcode Details

#### **`GET /barcodes/{id}`**

**Introduction**:  
Retrieve the full data of a single barcode, applying the same access scope as used when fetching the list.

**Parameters**: None.

**Example**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/101"
```

**Successful response (200)**:

```json
{
  "code": 200,
  "status": true,
  "message": "",
  "data": { /* same structure as a single object in the list */ }
}
```

---

### 3.7. Verify a Barcode (without using it)

#### **`POST /barcodes/verify`**

**Introduction**:  
Check whether a barcode is valid for use (exists, active, not expired, has not exhausted its usage count) without changing its state. Returns descriptive information about its status.

**Parameters** (JSON request body):

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `barcode` | string | Yes | The barcode value to verify. |

**Example**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/verify" \
  -H "Content-Type: application/json" \
  -d '{"barcode": "BC000101"}'
```

**Response (valid barcode)**:

```json
{
  "code": 200,
  "status": true,
  "message": "Barcode is valid.",
  "error": null,
  "data": {
    "id": 101,
    "code": "BC000101",
    "barcode": "BC000101",
    "barcode_type": "C128",
    "is_active": true,
    "is_use": false,
    "is_expired": false,
    "verification_score": 100,
    "can_use": true
  }
}
```

**Response (already used barcode)**:

```json
{
  "code": 200,
  "status": false,
  "message": "This barcode has already been used.",
  "error": null,
  "data": { "id": 101, "is_use": true, "used_count": 1 }
}
```

---

### 3.8. Scan and Use a Barcode

#### **`POST /barcodes/use`**

**Introduction**:  
This endpoint is used to scan a barcode (e.g., via a mobile app) and record its usage. It increments `used_count`, sets `is_use = true` if the count reaches the maximum (`generated_count`), records `used_at` and `last_used_at`, and links the usage to the current user (if logged in). Additionally, the usage is counted towards the user’s daily usage limits (if `usage_limits` are enabled).

**Parameters** (JSON request body):

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `barcode` | string | Yes | The scanned barcode value. |

**Example**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/use" \
  -H "Content-Type: application/json" \
  -d '{"barcode": "BC000101"}'
```

**Successful response (first time used)**:

```json
{
  "code": 200,
  "status": true,
  "message": "Barcode used successfully.",
  "data": {
    "id": 101,
    "code": "BC000101",
    "barcode": "BC000101",
    "is_use": true,
    "used_count": 1,
    "used_at": "2025-03-15T14:30:00+03:00",
    "last_used_at": "2025-03-15T14:30:00+03:00"
  }
}
```

**Failure response (daily limit exceeded for the user)**:

```json
{
  "code": 400,
  "status": false,
  "message": "An error occurred while using the barcode.",
  "error": "You have exceeded your daily barcode usage limit (10)."
}
```

---

### 3.9. Administrative Verification of a Barcode (set is_verified)

#### **`POST /barcodes/verify/{id}`**

**Introduction**:  
An admin‑only endpoint to certify a barcode (set `is_verified = true`). You can optionally provide a `verification_score` (0‑100). This is typically used after manual verification of associated documents.

**Parameters** (JSON request body, optional):

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `score` | int | No | Verification score (0‑100, default 100). |

**Example**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/verify/101" \
  -H "Content-Type: application/json" \
  -d '{"score": 95}'
```

**Successful response**:

```json
{
  "code": 200,
  "status": true,
  "message": "Barcode verified successfully.",
  "data": {
    "id": 101,
    "is_verified": true,
    "verification_score": 95,
    "verified_at": "2025-03-15T15:00:00+03:00"
  }
}
```

---

### 3.10. Batch Generate Barcodes

#### **`POST /barcodes/generate`**

**Introduction**:  
Generate multiple barcodes at once (up to 1000). This endpoint is available only to backend users with the `generate` permission. You specify the count, prefix, and type, and it produces sequential barcodes (e.g., BC000001, BC000002, …).

**Parameters** (JSON request body):

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `count` | int | No | Number of barcodes required (1‑1000, default 10). |
| `prefix` | string | No | Code prefix (default `BC`). |
| `barcode_type` | string | No | Barcode type (default `C128`). |
| `expiry_days` | int | No | Number of days until expiry (0 = never expire, default 365). |
| `product_id` | int | No | Link to a specific product. |
| `price` | float | No | Uniform price for all generated barcodes. |
| `companys_id` | int | No | Company ID. |
| `departments_id` | int | No | Department ID. |

**Example**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/generate" \
  -H "Content-Type: application/json" \
  -d '{"count": 5, "prefix": "SALE", "barcode_type": "QR", "price": 29.99}'
```

**Successful response**:

```json
{
  "code": 200,
  "status": true,
  "message": "5 barcodes created successfully.",
  "data": {
    "success_count": 5,
    "failed_count": 0,
    "success": [
      { "id": 111, "code": "SALE000001", "barcode": "SALE000001" },
      { "id": 112, "code": "SALE000002", "barcode": "SALE000002" }
      // ... up to 5
    ],
    "failed": []
  }
}
```

---

### 3.11. General Barcode Statistics

#### **`GET /barcodes/stats`**

**Introduction**:  
Returns statistics about barcodes based on the user’s permissions. For frontend users, the statistics are filtered to include only barcodes associated with them (via `user_id`). For backend users with the `access_all` permission, statistics cover all barcodes; otherwise, they are limited to those created by the user.

**Parameters**: None (filters are derived from the user).

**Example**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/stats"
```

**Successful response**:

```json
{
  "code": 200,
  "status": true,
  "message": "Barcode statistics retrieved successfully.",
  "data": {
    "total": 152,
    "active": 120,
    "inactive": 32,
    "published": 100,
    "verified": 80,
    "used": 45,
    "expired": 12,
    "expiring_soon": 5,
    "by_type": { "C128": 70, "QR": 50, "EAN13": 32 },
    "by_status": { "active": 120, "used": 45, "expired": 12 }
  }
}
```

---

### 3.12. Current User’s Usage Statistics

#### **`GET /barcodes/user-stats`**

**Introduction**:  
Displays statistics specific to the authenticated user (frontend or backend) regarding the number of barcodes they have used, remaining daily limits, and last usage. Relies on caching to determine daily limits if enabled.

**Parameters**: None.

**Example**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/user-stats"
```

**Successful response**:

```json
{
  "code": 200,
  "status": true,
  "message": "User statistics retrieved successfully",
  "data": {
    "total_used": 8,
    "today_used": 2,
    "this_week_used": 6,
    "this_month_used": 8,
    "last_used_at": "2025-03-15T13:20:00+03:00",
    "today_limit": 10,
    "today_remaining": 8
  }
}
```

---

### 3.13. Get the Barcode Image

#### **`GET /barcodes/image/{id}`**

**Introduction**:  
Returns the barcode image directly in PNG format (Content-Type: image/png). If the image is already stored in `image_path`, it is returned from storage; otherwise, it is generated on the fly using `BarcodeGenerator`. The same show permissions are applied to ensure the user is authorised to view this barcode.

**Parameters**: None.

**Example**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/image/101" --output barcode.png
```

**Successful response**: A PNG image file (displayed or downloaded).

**Failure response (404)**:

```json
{
  "code": 400,
  "status": false,
  "message": "Unable to generate barcode image.",
  "error": "Unable to generate barcode image."
}
```

---

## 4. Best Practices and Tips for Developers

1. **Use caching**  
   - Enable `enable_cache` in `config.php` to improve performance for the `index` and `stats` endpoints.  
   - Use the `activelystats` endpoint to intelligently refresh the cache (compare `last_updated` with a locally stored timestamp).

2. **Manage usage limits**  
   - Configure `usage_limits` in the settings file according to your business needs. It is recommended to use `tracking_type = database` for accuracy when the actual usage log is important.  
   - Provide a frontend display showing the user their remaining daily limit (via `userStats`).

3. **Validate data**  
   - Use `validateBarcode` before attempting to create or update a barcode to ensure the value matches the chosen type (e.g., EAN-13).  
   - Ensure that `barcode` and `barcode_type` are present in the `store` request, as they are the only mandatory fields from the API side (a `code` is auto‑generated).

4. **Manage permissions**  
   - Review the `config.php` file to understand which operations are available for each user type (backend/frontend/guest).  
   - It is recommended to grant `use` permission only to frontend users, while `generate` and `verify` should be reserved for administrators.

5. **Handle errors**  
   - All responses follow a unified structure. Check `status` and `code` to determine whether the operation succeeded.  
   - Use the `errors` field to see detailed validation error messages (when a `ValidationException` occurs).

6. **Image loading**  
   - To obtain a barcode image with high quality and resolution, you can call the `image` endpoint; it is possible to extend it with size and format parameters via `BarcodeGenerator` (currently not implemented in the `image` endpoint, but can be expanded).

7. **Advanced filtering and pagination**  
   - Use the `is_or_*`, `is_not_*`, `is_or_null_*` parameters only for fields allowed in `advanced_filters` (defined in `config.php`) to prevent misuse.  
   - Be cautious when using `exclude` – it may remove essential fields that the client needs.

---

## 5. Conclusion

The `Barcodes` API provides a complete solution for managing the lifecycle of barcodes: from creation and updating to usage and statistics. Thanks to its flexible permission system and support for usage limits, it can be easily integrated into billing systems, product tracking, loyalty programmes, and more. The documentation above covers all endpoints with practical examples and clear response models. For customisation or extension, it is recommended to refer to the source code and use the `Manager` class for advanced operations.

If you need to modify the behaviour of any endpoint, you can edit the controller file `Barcodes.php` or the `config.php` settings (especially operation permissions and `advanced_filters`). The development team is ready to support any further inquiries.