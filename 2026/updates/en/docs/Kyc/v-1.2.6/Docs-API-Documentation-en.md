## Identity Verification (KYC) API Documentation – Version 1.2.6

**Version:** 1.2.6 
**Base Path:** `/api/v1/kyc`  
**Authentication:** OAuth 2.0 (access token in header `Authorization: Bearer <token>`)

---

## Overview

The `Nano3.Kyc` API provides secure and flexible endpoints for managing Know Your Customer (KYC) identity verification documents in NanoSoft applications. Developers can use it to:

- Create, view, update, and delete documents.
- Upload identity files (passport, ID, signature images) and attach them to documents.
- Verify or reject documents.
- Assess KYC status for a specific user or entity (completion percentage, missing documents).
- Retrieve supported document types and their dynamic fields.
- Run a comprehensive test suite (development environment only).

**Key Features:**
- Support for over 28 document types (passport, national ID, utility bill, commercial license, etc.).
- File upload in multiple formats (multipart/form-data or base64) with `Nano.FileUpload` integration.
- Automatic protection for verified documents against file modifications.
- Instant KYC status assessment with intelligent recommendations.
- Unified responses and consistent data structure.

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
    "model": {},
    "input_data": {},
    "process_data": {},
    "debug": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | `int` | Status code (200 for success, 400 for error). |
| `status` | `bool` | `true` for success, `false` for failure. |
| `message` | `string` | Clear user message (translatable). |
| `error` | `string\|null` | Technical error details (in case of failure). |
| `errors` | `array\|null` | Array of validation errors (e.g., input validation). |
| `data` | `mixed` | Core requested data (e.g., document details). |
| `model` | `object\|null` | Full document object (for create/update operations). |
| `input_data` | `array` | Copy of the data sent by the user (for tracing). |
| `process_data` | `array` | Internal processing information (e.g., partial upload errors). |
| `debug` | `array\|null` | Debug information (only appears when `app.debug` is enabled). |

---

## Authentication and Security

All endpoints are protected by **OAuth 2.0**. The access token must be included in the request header:

```
Authorization: Bearer <your_access_token>
```

- If no token is provided: `401 Unauthorized` response.
- If token is invalid or expired: `401 Unauthorized` response.

**Note:** Some operations require additional permissions (e.g., verifying documents), which are checked via `FileUploadUserManager` and `backend` permissions. Regular (`frontend`) users can only manage their own documents.

---

## Endpoints

### 1. List Documents

**Method:** `GET`  
**Path:** `/documents`

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `document_type` | `string` | No | Document type code (e.g., `passport`). |
| `document_category` | `string` | No | Document category (e.g., `primary_id`). |
| `status` | `string` | No | Document status (`pending`, `verified`, `rejected`). |
| `owner_id` | `int` | No | Owner ID (used by `backend` only). |
| `owner_type` | `string` | No | Owner type (used by `backend` only). |
| `is_verified` | `bool` | No | Filter by verification status (`true`/`false`). |
| `q` | `string` | No | Text search in document number, name, company name, code. |
| `page` | `int` | No | Page number (default 1). |
| `per_page` | `int` | No | Items per page (default 15). |
| `orderBy` | `string` | No | Order by field (default `created_at`). |
| `orderDirection` | `string` | No | Order direction (`asc` or `desc`, default `desc`). |

#### Example Request

```http
GET /api/v1/kyc/documents?document_type=passport&status=pending&per_page=10 HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Records retrieved successfully.",
    "data": [
        {
            "id": 45,
            "code": "KYC20250417001",
            "document_type": "passport",
            "document_type_name": "Passport",
            "document_number": "P12345678",
            "full_name": "Ahmed Mohammed",
            "status": "pending",
            "is_verified": false,
            "issue_date": "2020-01-01",
            "expiry_date": "2030-01-01",
            "created_at": "2026-04-17 10:30:00",
            "owner": {
                "id": 24,
                "name": "Ahmed Mohammed",
                "type": "backend"
            }
        }
    ],
    "meta": {
        "pagination": {
            "total": 1,
            "per_page": 10,
            "current_page": 1,
            "last_page": 1
        }
    }
}
```

**Note:** Regular (`frontend`) users cannot pass `owner_id` or `owner_type`; results are automatically filtered to show only their own documents.

---

### 2. View a Specific Document

**Method:** `GET`  
**Path:** `/documents/{id}`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `int\|string` | Yes | Document ID or its `code`. |

#### Example Request

```http
GET /api/v1/kyc/documents/45 HTTP/1.1
Authorization: Bearer ...
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Data retrieved successfully",
    "data": {
        "id": 45,
        "code": "KYC20250417001",
        "document_type": "passport",
        "document_type_name": "Passport",
        "document_category": "primary_id",
        "document_category_name": "Primary Identity",
        "document_number": "P12345678",
        "issuing_authority": "Ministry of Interior",
        "issue_date": "2020-01-01",
        "expiry_date": "2030-01-01",
        "full_name": "Ahmed Mohammed",
        "date_of_birth": "1990-05-15",
        "nationality": "Yemeni",
        "country_of_issue": "Yemen",
        "religion": "Muslim",
        "address_line1": "45 Street",
        "address_city": "Sana'a",
        "address_country": "Yemen",
        "company_name": null,
        "owner_id": 24,
        "owner_type": "Backend\\Models\\User",
        "is_verified": false,
        "verification_score": null,
        "status": "pending",
        "is_active": true,
        "is_published": true,
        "document_front": {
            "original": "https://account.now-ye.com/storage/app/uploads/public/.../original.jpg",
            "thumb": "https://account.now-ye.com/storage/app/uploads/public/.../thumb.jpg"
        },
        "fields_data": {
            "passport_type": "P"
        },
        "created_at": "2026-04-17 10:30:00",
        "updated_at": "2026-04-17 10:30:00"
    }
}
```

**❌ Error Response (Document not found):**

```json
{
    "code": 404,
    "status": false,
    "message": "Document not found.",
    "error": "Document not found."
}
```

---

### 3. Create a New Document

**Method:** `POST`  
**Path:** `/documents`

#### Request Parameters (JSON or multipart)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `document_type` | `string` | Yes | Document type code (e.g., `passport`). |
| `owner_id` | `int` | Conditional* | Owner ID (*unless the current user is the owner). |
| `owner_type` | `string` | Conditional* | Owner type (*unless the current user is the owner). |
| `fields` | `object` | No | Dynamic field data according to the document type (e.g., `{"full_name": "...", "document_number": "..."}`). |
| `document_front` | `file\|base64` | No | Front side image of the document (or base64). |
| `document_back` | `file\|base64` | No | Back side image of the document (or base64). |
| `signature_image` | `file\|base64` | No | Signature image (or base64). |
| `image` | `file\|base64` | No | Additional profile image (or base64). |
| `files[]` | `file[]\|base64[]` | No | Multiple additional files (or base64). |
| `status` | `string` | No | Document status (default `pending`). |
| `is_published` | `bool` | No | Publish the document (default `false`). |
| `is_active` | `bool` | No | Activate the document (default `true`). |

> **Note:** When sending base64 files, use the same field name with the suffix `_base64` (e.g., `document_front_base64`).

#### Example Request (multipart)

```http
POST /api/v1/kyc/documents HTTP/1.1
Authorization: Bearer ...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="document_type"

passport
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[full_name]"

Ahmed Mohammed Ali
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[document_number]"

P12345678
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[issue_date]"

2020-01-01
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[expiry_date]"

2030-01-01
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[country_of_issue]"

YE
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[nationality]"

YE
------WebKitFormBoundary
Content-Disposition: form-data; name="document_front"; filename="passport.jpg"
Content-Type: image/jpeg

(binary file data)
------WebKitFormBoundary--
```

#### Example Request (JSON with base64)

```json
POST /api/v1/kyc/documents HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "document_type": "passport",
    "fields": {
        "full_name": "Ahmed Mohammed Ali",
        "document_number": "P12345678",
        "issue_date": "2020-01-01",
        "expiry_date": "2030-01-01",
        "country_of_issue": "YE",
        "nationality": "YE"
    },
    "document_front_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
}
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Document created successfully.",
    "data": {
        "id": 46,
        "document_type": "passport",
        "document_number": "P12345678",
        "full_name": "Ahmed Mohammed Ali",
        "status": "pending",
        "document_front": {
            "original": "https://account.now-ye.com/storage/app/uploads/public/.../original.jpg"
        }
    },
    "model": { ... },
    "process_data": {
        "file_errors": []
    }
}
```

**⚠️ Partial Success Response (some files failed to upload):**

```json
{
    "code": 200,
    "status": true,
    "message": "Document created but some files failed to upload.",
    "data": { ... },
    "process_data": {
        "file_errors": {
            "document_back": "File size exceeds the maximum allowed (10240 KB)"
        }
    }
}
```

**❌ Error Response (invalid document type):**

```json
{
    "code": 400,
    "status": false,
    "message": "An error occurred while creating the document.",
    "error": "Invalid document type."
}
```

---

### 4. Update a Document

**Method:** `PUT`  
**Path:** `/documents/{id}`

#### Request Parameters

Same as create parameters, but all are optional. Dynamic fields can be updated via `fields`, and files using the same field names.

**Important Rule:** If the document is verified (`is_verified = true`), **you cannot modify or replace any associated files** (e.g., `document_front`). Only textual fields not related to files can be updated.

#### Example Request (update name only)

```json
PUT /api/v1/kyc/documents/46 HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "fields": {
        "full_name": "Ahmed Mohammed Abdullah"
    }
}
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Document updated successfully.",
    "data": {
        "id": 46,
        "full_name": "Ahmed Mohammed Abdullah",
        "updated_at": "2026-04-17 12:15:00"
    }
}
```

**❌ Error Response (attempting to change a file on a verified document):**

```json
{
    "code": 400,
    "status": false,
    "message": "An error occurred while updating the document.",
    "error": "Cannot change the file document_front for a verified document."
}
```

---

### 5. Delete a Document (Soft Delete)

**Method:** `DELETE`  
**Path:** `/documents/{id}`

#### Example Request

```http
DELETE /api/v1/kyc/documents/46 HTTP/1.1
Authorization: Bearer ...
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Document deleted successfully."
}
```

**❌ Error Response (not found):**

```json
{
    "code": 400,
    "status": false,
    "message": "An error occurred while deleting the document.",
    "error": "The requested document does not exist."
}
```

---

### 6. Restore a Deleted Document

**Method:** `POST`  
**Path:** `/documents/restore/{id?}`

#### Example Request

```http
POST /api/v1/kyc/documents/restore/45 HTTP/1.1
Authorization: Bearer ...
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Document restored successfully.",
    "data": { ... }
}
```

---

### 7. Verify (Approve) a Document

**Method:** `POST`  
**Path:** `/documents/verify/{id?}`

#### Request Parameters (JSON)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `verification_score` | `int` | No | Trust score (0-100, default 100). |

#### Example Request

```json
POST /api/v1/kyc/documents/verify/45 HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "verification_score": 95
}
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Document verified successfully.",
    "data": {
        "id": 45,
        "is_verified": true,
        "verified_at": "2026-04-17 14:20:00",
        "verification_score": 95,
        "status": "verified"
    }
}
```

**Note:** Once a document is verified, its files cannot be modified.

---

### 8. Reject a Document

**Method:** `POST`  
**Path:** `/documents/reject/{id?}`

#### Request Parameters (JSON)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | `string` | No | Reason for rejection. |

#### Example Request

```json
POST /api/v1/kyc/documents/reject/45 HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "reason": "Image is unclear"
}
```

#### Response

**✅ Success Response:**

```json
{
    "code": 200,
    "status": true,
    "message": "Document rejected successfully.",
    "data": {
        "id": 45,
        "status": "rejected",
        "metadata": {
            "rejection_reason": "Image is unclear"
        }
    }
}
```

---

### 9. Retrieve Supported Document Types

**Method:** `GET`  
**Path:** `/document-types` or `/document-types/{id}`

#### Path Parameters (optional)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `string` | No | Document type code to get its details only. |

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `grouped` | `bool` | No | Group results by category (default `false`). |
| `category` | `string` | No | Filter by a specific category (e.g., `primary_id`). |
| `locale` | `string` | No | Language (`ar` or `en`). |

#### Example Request (all types, flat)

```http
GET /api/v1/kyc/document-types HTTP/1.1
Authorization: Bearer ...
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Data retrieved successfully",
    "data": {
        "passport": "Passport",
        "national_id": "National ID Card",
        "drivers_license": "Driver's License",
        "utility_bill": "Utility Bill",
        ...
    }
}
```

#### Example Request (grouped by category)

```http
GET /api/v1/kyc/document-types?grouped=true HTTP/1.1
```

```json
{
    "code": 200,
    "status": true,
    "message": "Data retrieved successfully",
    "data": {
        "Primary Identity": {
            "passport": "Passport",
            "national_id": "National ID Card",
            ...
        },
        "Proof of Address": {
            "utility_bill": "Utility Bill",
            ...
        }
    }
}
```

#### Example Request (specific type details)

```http
GET /api/v1/kyc/document-types/passport HTTP/1.1
```

```json
{
    "code": 200,
    "status": true,
    "message": "Data retrieved successfully",
    "data": {
        "code": "passport",
        "name": "Passport",
        "description": "A valid passport containing a photo and traveler data.",
        "category": "primary_id",
        "category_name": "Primary Identity",
        "has_expiry": true,
        "verification_weight": 100,
        "required_fields": ["document_front", "document_number", "issue_date", "expiry_date", "country_of_issue"],
        "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
        "max_file_size_kb": 10240,
        "requires_back_side": false,
        "requires_selfie_match": true,
        "fields": {
            "document_front": { "type": "file", "required": true, "label": "Document Front" },
            "document_number": { "type": "text", "required": true, "label": "Document Number" },
            ...
        }
    }
}
```

---

### 10. Retrieve Document Type Fields (for building dynamic forms)

**Method:** `GET`  
**Path:** `/document-fields/{type}`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | `string` | Yes | Document type code (e.g., `passport`). |

#### Example Request

```http
GET /api/v1/kyc/document-fields/passport HTTP/1.1
Authorization: Bearer ...
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Data retrieved successfully",
    "data": {
        "type": "passport",
        "type_name": "Passport",
        "fields": {
            "document_front": {
                "label": "Document Front",
                "type": "file",
                "required": true,
                "validation": { "mimes": ["jpeg","jpg","png","pdf"], "max_size_kb": 10240 }
            },
            "document_number": {
                "label": "Document Number",
                "type": "text",
                "required": true,
                "validation": { "max_length": 50 }
            },
            "full_name": { ... },
            "issue_date": { ... },
            "expiry_date": { ... },
            "country_of_issue": { ... },
            "nationality": { ... },
            "date_of_birth": { ... },
            "religion": { ... },
            "passport_type": {
                "label": "Passport Type",
                "type": "select",
                "required": false,
                "options": { "P": "Personal", "D": "Diplomatic", "O": "Official" }
            }
        },
        "rules": {
            "document_front": ["required", "file", "mimes:jpeg,jpg,png,pdf", "max:10240"],
            "document_number": ["required", "string", "max:50"],
            "expiry_date": ["required", "date", "after_or_equal:today"]
        },
        "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
        "max_file_size_kb": 10240,
        "requires_back_side": false,
        "requires_selfie_match": true
    }
}
```

---

### 11. Quick Document Statistics

**Method:** `GET`  
**Path:** `/documents/stats`

#### Example Request

```http
GET /api/v1/kyc/documents/stats HTTP/1.1
Authorization: Bearer ...
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Statistics retrieved successfully",
    "data": {
        "total": 156,
        "active": 142,
        "published": 130,
        "public": 120,
        "default": 5,
        "by_type": {
            "passport": 45,
            "national_id": 38,
            "utility_bill": 22,
            ...
        },
        "by_ref_type": { ... }
    }
}
```

---

### 12. Check for New Updates (for caching)

**Method:** `GET`  
**Path:** `/documents/activelystats`

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `date` | `string` | No | Date for comparison (default `now`). |

#### Example Request

```http
GET /api/v1/kyc/documents/activelystats?date=2026-04-17 10:00:00 HTTP/1.1
Authorization: Bearer ...
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "data": {
        "activity_stats": true,
        "check_date": "2026-04-17 10:00:00",
        "last_updated": "2026-04-17 14:30:00"
    }
}
```

- `activity_stats = true` means there have been updates since the requested date.

---

### 13. Assess KYC Status for a Specific Owner

**Method:** `GET` or `POST`  
**Path:** `/assess`

#### Request Parameters (JSON or Query)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `owner_type` | `string` | Conditional* | Owner type (if `backend` and not provided, uses itself). |
| `owner_id` | `int` | Conditional* | Owner ID. |
| `risk_level` | `string` | No | Risk level (`low`, `medium`, `high`, default `medium`). |
| `category` | `string` | No | Filter by document category (`primary_id`, `address`, ...). |
| `document_type` | `string\|array` | No | Filter by a specific document type. |
| `include_expired` | `bool` | No | Include expired documents (default `true`). |
| `include_pending` | `bool` | No | Include pending documents (default `true`). |
| `include_rejected` | `bool` | No | Include rejected documents (default `false`). |

> *If the user is `frontend`, the assessment is automatically performed on themselves.

#### Example Request (backend user assessing themselves with high risk)

```http
GET /api/v1/kyc/assess?risk_level=high HTTP/1.1
Authorization: Bearer ...
```

#### Example Request (backend assessing a specific frontend user)

```json
POST /api/v1/kyc/assess HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "owner_type": "RainLab\\User\\Models\\User",
    "owner_id": 532,
    "risk_level": "high",
    "include_expired": true
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "KYC status assessed successfully.",
    "data": {
        "is_compliant": false,
        "completion_percentage": 50.0,
        "overall_score": 45,
        "total_documents_required": 2,
        "verified_documents_count": 1,
        "pending_documents_count": 0,
        "expired_documents_count": 0,
        "rejected_documents_count": 0,
        "verified_documents": [
            {
                "id": 45,
                "type": "passport",
                "document_number": "P12345678",
                "expiry_date": "2030-01-01",
                "verification_score": 100
            }
        ],
        "pending_documents": [],
        "expired_documents": [],
        "rejected_documents": [],
        "missing_required_types": ["national_id"],
        "recommendations": [
            "Missing required document types: national_id",
            "Recommended additional documents: drivers_license, utility_bill"
        ]
    }
}
```

---

### 14. Assess KYC Status by Document Category

**Method:** `GET` or `POST`  
**Path:** `/assess/{category}`

#### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `category` | `string` | No | Document category (`primary_id`, `address`, `corporate`..., default `primary_id`). |

#### Request Parameters (same as `assess` plus)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `document_type` | `string\|array` | No | Specific document type or array of types within the category. |

#### Example Request (assess address category)

```http
GET /api/v1/kyc/assess/address?include_expired=false HTTP/1.1
Authorization: Bearer ...
```

#### Response (similar to `assess` but `total_documents_required` includes only the specified category types)

```json
{
    "code": 200,
    "status": true,
    "message": "KYC status by category assessed successfully.",
    "data": {
        "is_compliant": true,
        "completion_percentage": 100.0,
        "overall_score": 60,
        "total_documents_required": 1,
        "verified_documents_count": 1,
        "verified_documents": [
            {
                "id": 50,
                "type": "utility_bill",
                "document_number": "INV-2026-001",
                "issue_date": "2026-03-01",
                "verification_score": 60
            }
        ],
        "missing_required_types": [],
        "recommendations": []
    }
}
```

---

### 15. Run Package Tests (Development Only)

**Method:** `GET`  
**Path:** `/tests`

> **Important Warning:** This endpoint is only available when `app.debug = true` or the environment is `local`. It cannot be used in production.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `test_version` | `string` | No | `v1` or `v2` (default `v2`). |
| `filter` | `string` | No | `all`, `passed`, `failed` (default `all`). |

#### Example Request

```http
GET /api/v1/kyc/tests?test_version=v2&filter=passed HTTP/1.1
Authorization: Bearer ...
```

#### Response (abbreviated example)

```json
{
    "code": 200,
    "status": true,
    "message": "KYC Test suite completed",
    "data": [
        {
            "code": 200,
            "status": true,
            "test_code": "DOCTYPE_REG_001",
            "name": "Register document types in DocumentType",
            "description": "Checks that DocumentType contains basic types...",
            "message": "Test passed",
            "data": {
                "total_types": 28,
                "has_passport": true,
                "has_national_id": true
            }
        },
        ...
    ],
    "meta": {
        "summary": {
            "total": 18,
            "passed": 18,
            "failed": 0,
            "success_rate": 100,
            "test_version": "v2",
            "filter_applied": "passed"
        }
    },
    "process_data": {
        "timestamp": "2026-04-17 15:00:00"
    }
}
```

---

### 16. Fetch Supported Document Categories (Structured List)

**Method:** `GET`  
**Path:** `/document-categories-list` or `/document-categories-list/{id}`

Provides two endpoints to fetch document categories (their classes) with structured data supporting translation and advanced include options.

#### Path Parameters (optional)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `string` | No | Category code (e.g., `primary_id`). If provided, only details for this category are fetched. |

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `locale` | `string` | No | Requested language (`ar` or `en`). Default is the system language. |
| `include_types` | `bool` | No | Include document types belonging to each category. Default `false`. |
| `include_details` | `bool` | No | Include details of each document type (e.g., weight, required fields, etc.). Only works with `include_types=1`. |
| `include_fields` | `bool` | No | Include the field schema (`fields`) for each document type. |
| `include_rules` | `bool` | No | Include validation rules (`rules`) for each document type. |
| `category_filter` | `string` | No | Filter categories by a specific code (e.g., `primary_id`). Can be used when fetching the list. |

#### Example 1: Fetch All Categories (No Details)

**Request:**
```http
GET /api/v1/kyc/document-categories-list HTTP/1.1
Authorization: Bearer ...
```

**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Categories data retrieved successfully",
    "data": [
        {
            "code": "primary_id",
            "name_ar": "هوية أساسية",
            "name_en": "Primary ID",
            "name": "هوية أساسية"
        },
        {
            "code": "secondary_id",
            "name_ar": "هوية ثانوية",
            "name_en": "Secondary ID",
            "name": "هوية ثانوية"
        },
        {
            "code": "address",
            "name_ar": "إثبات عنوان",
            "name_en": "Proof of Address",
            "name": "إثبات عنوان"
        }
        // ... remaining categories
    ]
}
```

#### Example 2: Fetch a Specific Category with Its Types and Details

**Request:**
```http
GET /api/v1/kyc/document-categories-list/primary_id?include_types=1&include_details=1 HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "code": "primary_id",
        "name_ar": "هوية أساسية",
        "name_en": "Primary ID",
        "name": "هوية أساسية",
        "types": [
            {
                "code": "passport",
                "name_ar": "جواز سفر",
                "name_en": "Passport",
                "name": "Passport",
                "description": "A valid passport containing a photo and traveler data.",
                "details": {
                    "verification_weight": 100,
                    "has_expiry": true,
                    "required_fields": ["document_front", "document_number", ...]
                }
            },
            {
                "code": "national_id",
                "name_ar": "بطاقة هوية وطنية",
                "name_en": "National ID Card",
                "name": "National ID Card",
                "description": "National ID card issued by official authorities.",
                "details": {
                    "verification_weight": 95,
                    "has_expiry": true,
                    "requires_back_side": true
                }
            }
            // ... remaining types
        ]
    }
}
```

#### Example 3: Fetch All Categories with Types, Fields, and Rules

**Request:**
```http
GET /api/v1/kyc/document-categories-list?include_types=1&include_fields=1&include_rules=1 HTTP/1.1
Authorization: Bearer ...
```

---

### 17. Fetch Supported Document Types (Structured List)

**Method:** `GET`  
**Path:** `/document-types-list` or `/document-types-list/{id}`

Provides two endpoints to fetch individual document types with structured data supporting translation and advanced include options.

#### Path Parameters (optional)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `string` | No | Document type code (e.g., `passport`). If provided, only details for this type are fetched. |

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `locale` | `string` | No | Requested language (`ar` or `en`). Default is the system language. |
| `include_category` | `bool` | No | Include parent category information (e.g., `primary_id`) for each type. Default `false`. |
| `include_details` | `bool` | No | Include type details (weight, required fields, allowed files...). Default `false`. |
| `include_fields` | `bool` | No | Include the full field schema (`fields`) for the type. |
| `include_rules` | `bool` | No | Include validation rules (`rules`) for each field. |
| `category_filter` | `string` | No | Filter types by a specific category (e.g., `primary_id`). Can be used when fetching the list. |

#### Example 1: Fetch All Document Types (No Details)

**Request:**
```http
GET /api/v1/kyc/document-types-list HTTP/1.1
Authorization: Bearer ...
```

**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Document types data retrieved successfully",
    "data": [
        {
            "code": "passport",
            "name_ar": "جواز سفر",
            "name_en": "Passport",
            "name": "Passport",
            "description": "A valid passport containing a photo and traveler data."
        },
        {
            "code": "national_id",
            "name_ar": "بطاقة هوية وطنية",
            "name_en": "National ID Card",
            "name": "National ID Card",
            "description": "National ID card issued by official authorities."
        }
        // ... remaining types
    ]
}
```

#### Example 2: Fetch a Specific Document Type with Category, Details, Fields, and Rules

**Request:**
```http
GET /api/v1/kyc/document-types-list/passport?include_category=1&include_details=1&include_fields=1&include_rules=1 HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "code": "passport",
        "name_ar": "جواز سفر",
        "name_en": "Passport",
        "name": "Passport",
        "description": "A valid passport containing a photo and traveler data.",
        "category": {
            "code": "primary_id",
            "name_ar": "هوية أساسية",
            "name_en": "Primary ID",
            "name": "Primary ID"
        },
        "details": {
            "category": "primary_id",
            "category_name": "Primary ID",
            "has_expiry": true,
            "verification_weight": 100,
            "required_fields": ["document_front", "document_number", "issue_date", ...],
            "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
            "max_file_size_kb": 10240,
            "requires_back_side": false,
            "requires_selfie_match": true
        },
        "fields": {
            "document_front": {
                "label": "document_front",
                "type": "file",
                "required": true,
                "validation": { "mimes": ["jpeg","jpg","png","pdf"], "max_size_kb": 10240 }
            },
            "document_number": {
                "label": "document_number",
                "type": "text",
                "required": true,
                "validation": { "max_length": 50 }
            }
            // ... remaining fields
        },
        "rules": {
            "document_front": ["required", "file", "mimes:jpeg,jpg,png,pdf", "max:10240"],
            "document_number": ["required", "max:50"]
            // ... remaining rules
        }
    }
}
```

#### Example 3: Fetch Types of the "Proof of Address" Category Only with Parent Category

**Request:**
```http
GET /api/v1/kyc/document-types-list?category_filter=address&include_category=1 HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "code": 200,
    "status": true,
    "data": [
        {
            "code": "utility_bill",
            "name": "Utility Bill",
            "category": {
                "code": "address",
                "name": "Proof of Address"
            }
        },
        {
            "code": "bank_statement",
            "name": "Bank Statement",
            "category": {
                "code": "address",
                "name": "Proof of Address"
            }
        }
        // ...
    ]
}
```

---

## Including KYC Data in API Responses (DynamicAddIncludeKyc Relationships)

The KYC system integrates with the `Nano.API` structure to allow including verification data directly within responses of other endpoints (e.g., user information, orders, or delivery personnel). This is done by passing an `include` key in the query string with one of the names listed below.

> **Important Note:** This feature is only available in `Transformers` that have been extended by the add-on (e.g., `UserTransformer` and `AdminTransformer`). These are not standalone endpoints; they are added to responses of supported entities.

### Available Relationships

| Relationship Name (`include`) | Description | Type | When to use? |
| :--- | :--- | :--- | :--- |
| `kyc_status` | Comprehensive KYC status assessment for the owner, with ability to specify risk level and document category. | `object` | When you need complete verification status (complete/incomplete, percentage, missing documents, recommendations). |
| `kyc_status_by_category` | KYC status assessment based on a specific document category (e.g., `primary_id` or `address`). | `object` | When you want to check only a specific category (e.g., "Has the user completed primary identity documents?"). |
| `kyc_documents` | List of the owner's KYC documents with pagination and filtering support. | `collection` | To display the user's document list within their profile page. |
| `kyc_documents_count` | Number of KYC documents associated with the owner. | `int` | For a quick summary (e.g., "You have 3 documents"). |
| `kyc_verification_summary` | Quick summary of all six document categories with statistics (expired, pending, etc.). | `object` | To build a user KYC status dashboard. |
| `is_verifier_primary_id` | Is there a valid primary identity document? (`true`/`false`). | `bool` | For a quick condition check (e.g., "Can they complete an action requiring primary identity?"). |
| `is_verifier_secondary_id` | Is there a valid secondary identity document? | `bool` | To check for supporting documents (voter card, military ID). |
| `is_verifier_address` | Is there a valid proof of address document? | `bool` | To verify user address before shipping or delivery. |
| `is_verifier_corporate` | Is there a valid corporate document? | `bool` | To check verification status of a business/company account. |
| `is_verifier_ubo` | Is there a valid ultimate beneficial owner document? | `bool` | For regulatory compliance (anti-money laundering). |
| `is_verifier_additional` | Is there a valid additional document? | `bool` | To check for supplementary documents (e.g., signature, selfie). |

> **Note:** Relationships from `is_verifier_secondary_id` to `is_verifier_additional` are added to `secretIncludes` (do not appear in automatic API documentation but can be used), while `is_verifier_primary_id`, `kyc_documents_count`, and `kyc_verification_summary` are public relationships.

---

### How to Use

To request any of these relationships, add `?include=relationship_name` to the API request of the entity (e.g., `/api/v1/me` or `/api/v1/users/24`). Multiple relationships can be requested together by separating them with commas `,`.

#### 1. Example: Including Comprehensive KYC Assessment (`kyc_status`)

**Request:**
```http
GET /api/v1/me?include=kyc_status&kyc_risk_level=high HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "email": "ahmed@example.com",
    "kyc_status": {
        "is_compliant": false,
        "completion_percentage": 50.0,
        "overall_score": 45,
        "total_documents_required": 2,
        "verified_documents_count": 1,
        "missing_required_types": ["national_id"],
        "recommendations": ["Missing required document types: national_id"]
    }
}
```

#### 2. Example: Quick Summary of All Categories (`kyc_verification_summary`)

**Request:**
```http
GET /api/v1/me?include=kyc_verification_summary&kyc_verification_summary[include_pending]=true HTTP/1.1
Authorization: Bearer ...
```

**Response (excerpt):**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "kyc_verification_summary": {
        "primary_id": true,
        "secondary_id": false,
        "address": true,
        "corporate": false,
        "ubo": false,
        "additional": false,
        "total_verified": 2,
        "total_pending": 1,
        "total_expired": 0,
        "total_rejected": 0
    }
}
```

#### 3. Example: Boolean Verifiers (Quick Checks)

To check for a valid primary identity and proof of address:

```http
GET /api/v1/me?include=is_verifier_primary_id,is_verifier_address HTTP/1.1
```

**Response (excerpt):**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "is_verifier_primary_id": true,
    "is_verifier_address": false
}
```

**Optional parameters for verifiers:**

You can customize the check behavior by passing query parameters:

```
?include=is_verifier_primary_id&is_verifier_primary_id[include_pending]=true&is_verifier_primary_id[expiring_within_days]=30
```

| Parameter | Description | Default |
| :--- | :--- | :--- |
| `[include_pending]` | Include pending documents. | `false` |
| `[include_expired]` | Include expired documents. | `false` |
| `[check_validity]` | Check date validity. | `true` |
| `[expiring_within_days]` | Check documents expiring within a specified number of days. | `null` |

#### 4. Example: Including Document List with Pagination (`kyc_documents`)

```http
GET /api/v1/me?include=kyc_documents&kyc_documents[per_page]=5&kyc_documents[status]=verified HTTP/1.1
```

**Response (excerpt):**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "kyc_documents": {
        "data": [
            {
                "id": 45,
                "document_type": "passport",
                "document_number": "P12345678",
                "status": "verified",
                "document_front": { "thumb": "https://..." }
            }
        ],
        "meta": {
            "pagination": {
                "total": 2,
                "per_page": 5,
                "current_page": 1,
                "last_page": 1
            }
        }
    }
}
```

#### 5. Example: Combining Multiple Relationships in One Request

```http
GET /api/v1/me?include=kyc_verification_summary,kyc_documents_count,is_verifier_primary_id HTTP/1.1
```

---

### Advanced Options

Additional options can be passed for each relationship via query parameters using the relationship name followed by square brackets `[]`:

| Relationship | Supported Options |
| :--- | :--- |
| `kyc_status` | `risk_level`, `category`, `document_type`, `include_expired`, `include_pending`, `include_rejected` |
| `kyc_status_by_category` | `category` (required), `document_type`, `include_expired`, `include_pending`, `include_rejected` |
| `kyc_documents` | `per_page`, `page`, `document_type`, `status`, `paginate` |
| `kyc_verification_summary` | `include_pending`, `include_expired`, `check_validity`, `expiring_within_days` |
| `is_verifier_*` | `include_pending`, `include_expired`, `check_validity`, `expiring_within_days` |

---

### Security Considerations

- Displayed KYC data is subject to user permissions. Regular users see only their own data, while `backend` users can view any object's data if permissions allow.
- The owner scope cannot be bypassed through these relationships; the owner is automatically determined from the viewed object (e.g., `User` in `UserTransformer`).

---

## Common Error Codes

| HTTP Code | Description |
|-----------|-------------|
| 400 | Bad request (missing or invalid data). |
| 401 | Unauthorized (token missing or invalid). |
| 403 | Permission denied (user does not have access to the resource). |
| 404 | Resource not found (document, document type, etc.). |
| 422 | Validation failure for input data. |
| 500 | Internal server error. |

---

## Best Practices for Integration

1. **Use `temp_session_key` for unsaved models**: If building a multi-step interface, use a temporary session key to upload files first, then attach them after creating the document.
2. **Check user permissions before showing edit buttons**: Use the `assess` endpoints to understand KYC status and display appropriate prompts.
3. **Handle partial file upload errors**: Check `process_data.file_errors` after create/update to show accurate messages to the user.
4. **Do not attempt to modify files of verified documents**: Check `is_verified` before displaying file editing options.
5. **Use `document-fields/{type}` to build dynamic forms**: Automatically generate input fields and validation rules according to the selected document type.
6. **Enable `app.debug` only in development**: To get detailed error information in `debug`.
7. **Monitor `activity_stats`**: To refresh cache in client applications when new changes occur.

---

## Additional Documentation

- [General Plugin Documentation](./Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./Docs-Document-Model-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
