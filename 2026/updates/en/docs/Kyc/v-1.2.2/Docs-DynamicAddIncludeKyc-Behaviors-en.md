# `DynamicAddIncludeKyc` Behavior Documentation – Injecting KYC Data into API Responses

## Overview

The `DynamicAddIncludeKyc` behavior is part of the `Nano3.Kyc` add-on. It automatically injects new include points into `Transformers` of `Nano.API` (such as `UserTransformer`, `AdminTransformer`, `DeliveryTransformer`). These points allow developers to enrich API responses with KYC data associated with the viewed object (user, order, delivery person) without requiring separate requests.

**Supported Includes:**

| Key | Description |
| :--- | :--- |
| `kyc_status` | Comprehensive KYC status assessment for the owner, with ability to specify risk level and document category. |
| `kyc_status_by_category` | KYC status assessment based on document `category` with ability to specify specific types. |
| `kyc_documents` | List of KYC documents for the owner with pagination and filtering support. |
| `kyc_documents_count` | Number of KYC documents associated with the owner. |

**How it works?**
When one of these keys is added to the `include` parameter in an API request (e.g., `?include=kyc_status`), the behavior calls the corresponding functions in `Manager` (`assessKycStatus`, `assessKycStatusByCategory`, `getDocumentRecords`) and returns the formatted data within the original object's response.

---

## Installing and Activating the Behavior

The behavior is automatically activated when the `Nano3.Kyc` add-on is installed. In `Plugin.php`, `extendTransformer()` is called to add the behavior to the following `Transformers`:

- `UserTransformer` (website users)
- `AdminTransformer` (control panel users)
- `DeliveryTransformer` (delivery personnel)
- `ParentTransformer` (parents)
- `StudentTransformer` (students)
- `DepartmentTransformer` (branches and departments)

No additional configuration is required from the developer. Once the add-on is installed, these includes become available at endpoints that use the mentioned `Transformers` (e.g., `/api/v1/me`, `/api/v1/users/{id}`).

---

## Usage Examples in API Requests

### 1. Including Comprehensive KYC Assessment (`kyc_status`)

**Request:**
```http
GET /api/v1/me?include=kyc_status HTTP/1.1
Authorization: Bearer ...
```

**Optional parameters:**
Additional options can be passed via query parameters to customize the assessment:

| Parameter | Description | Possible Values |
| :--- | :--- | :--- |
| `kyc_risk_level` | Risk level to determine mandatory documents. | `low`, `medium` (default), `high` |
| `kyc_category` | Filter by document category. | `primary_id`, `address`, `corporate`, ... |
| `kyc_document_type` | Filter by a specific document type. | `passport`, `utility_bill`, ... |
| `kyc_include_expired` | Include expired documents. | `true` (default), `false` |
| `kyc_include_pending` | Include pending documents. | `true` (default), `false` |
| `kyc_include_rejected` | Include rejected documents. | `true`, `false` (default) |

**Example with options:**
```http
GET /api/v1/me?include=kyc_status&kyc_risk_level=high&kyc_category=primary_id HTTP/1.1
Authorization: Bearer ...
```

**Response structure:**
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
        "pending_documents_count": 1,
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
        "pending_documents": [
            {
                "id": 46,
                "type": "national_id",
                "document_number": "ID987654"
            }
        ],
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

### 2. Including KYC Assessment by Category (`kyc_status_by_category`)

**Request:**
```http
GET /api/v1/me?include=kyc_status_by_category&kyc_category=primary_id&kyc_document_type[]=passport&kyc_document_type[]=national_id HTTP/1.1
Authorization: Bearer ...
```

**Optional parameters:**

| Parameter | Description |
| :--- | :--- |
| `kyc_category` | **Required**. Document category (e.g., `primary_id`, `address`). If not specified, defaults to `primary_id`. |
| `kyc_document_type` | Specific document type or array of types (e.g., `passport` or `[]=passport&[]=national_id`). |
| `kyc_include_expired` | Include expired documents (`true`/`false`). |
| `kyc_include_pending` | Include pending documents (`true`/`false`). |
| `kyc_include_rejected` | Include rejected documents (`true`/`false`). |

**Simpler example (without array):**
```http
GET /api/v1/me?include=kyc_status_by_category&kyc_category=address&kyc_document_type=utility_bill HTTP/1.1
```

**Response structure:**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "kyc_status_by_category": {
        "is_compliant": true,
        "completion_percentage": 100,
        "overall_score": 60,
        "total_documents_required": 1,
        "verified_documents_count": 1,
        "pending_documents_count": 0,
        "expired_documents_count": 0,
        "rejected_documents_count": 0,
        "verified_documents": [
            {
                "id": 50,
                "type": "utility_bill",
                "document_number": "INV-2026-001",
                "issue_date": "2026-03-01",
                "verification_score": 60
            }
        ],
        "pending_documents": [],
        "expired_documents": [],
        "rejected_documents": [],
        "missing_required_types": [],
        "recommendations": []
    }
}
```

---

### 3. Including KYC Documents List (`kyc_documents`)

**Request:**
```http
GET /api/v1/me?include=kyc_documents HTTP/1.1
Authorization: Bearer ...
```

**Optional parameters (for customization and pagination):**

| Parameter | Description | Default |
| :--- | :--- | :--- |
| `kyc_documents[per_page]` | Number of documents per page. | 15 |
| `kyc_documents[page]` | Page number. | 1 |
| `kyc_documents[document_type]` | Filter by document type. | - |
| `kyc_documents[status]` | Filter by status (`pending`, `verified`, `rejected`). | - |
| `kyc_documents[paginate]` | Enable pagination (`true`/`false`). | `true` |

**Example with pagination and filtering:**
```http
GET /api/v1/me?include=kyc_documents&kyc_documents[per_page]=5&kyc_documents[status]=verified HTTP/1.1
```

**Response structure:**
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
                "full_name": "Ahmed Mohammed",
                "status": "verified",
                "issue_date": "2020-01-01",
                "expiry_date": "2030-01-01",
                "is_verified": true,
                "document_front": {
                    "original": "https://.../original.jpg",
                    "thumb": "https://.../thumb.jpg"
                }
            },
            {
                "id": 50,
                "document_type": "utility_bill",
                "document_number": "INV-2026-001",
                "status": "verified",
                "issue_date": "2026-03-01",
                "is_verified": true
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

---

### 4. Including KYC Documents Count (`kyc_documents_count`)

**Request:**
```http
GET /api/v1/me?include=kyc_documents_count HTTP/1.1
Authorization: Bearer ...
```

**Response structure:**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "kyc_documents_count": 3
}
```

---

### 5. Combining Multiple Includes in a Single Request

You can request more than one include in the same call:

```http
GET /api/v1/me?include=kyc_status,kyc_documents_count,kyc_documents&kyc_documents[per_page]=10 HTTP/1.1
```

**Response (abbreviated):**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "kyc_status": { ... },
    "kyc_documents_count": 3,
    "kyc_documents": { ... }
}
```

---

## Important Notes

- **Permissions:** The displayed KYC data is subject to user permissions. Regular users see only their own data, while `backend` users can view any object's data if permissions allow.
- **Caching:** API responses containing `include` may benefit from `Nano.API` caching. This can be controlled via `api_enable_cache` settings.
- **Performance:** Using `kyc_documents` with large pagination or no filtering may impact performance. It is recommended to always use an appropriate `per_page` and filter by status (`status=verified`) when needed.
- **Compatibility:** The behavior works with any `Transformer` extended in `Plugin::extendTransformer()`. To add support for new types, modify `Plugin.php`.

---

## Additional Documentation

- [`Manager` Class Documentation](./Docs-Manager-Class-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
- [`DocumentType` Class Documentation](./Docs-DocumentType-Class-en.md)
