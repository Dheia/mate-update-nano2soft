# `DynamicAddIncludeKyc` Behavior Documentation – Injecting KYC Data into API Responses

## Overview

The `DynamicAddIncludeKyc` behavior is a dynamic extension that is automatically added to `Transformers` of `Nano.API` (such as `UserTransformer`, `AdminTransformer`, `DeliveryTransformer`). It injects new include points that allow querying verification (KYC) data associated with the viewed object (user, order, branch, delivery person) within a single response, reducing the number of requests and improving the development experience.

**Supported Includes:**

| Key | Description | Type |
| :--- | :--- | :--- |
| `kyc_status` | Comprehensive KYC status assessment for the owner, with ability to specify risk level and document category. | object |
| `kyc_status_by_category` | KYC status assessment based on a specific document category. | object |
| `kyc_documents` | List of the owner's KYC documents with pagination and filtering support. | collection |
| `kyc_documents_count` | Number of KYC documents associated with the owner. | integer |
| `kyc_verification_summary` | Quick summary of all six document categories with statistics. | object |
| `is_verifier_primary_id` | Existence of a valid primary identity document (`true`/`false`). | boolean |
| `is_verifier_secondary_id` | Existence of a valid secondary identity document. | boolean |
| `is_verifier_address` | Existence of a valid proof of address document. | boolean |
| `is_verifier_corporate` | Existence of a valid corporate document. | boolean |
| `is_verifier_ubo` | Existence of a valid ultimate beneficial owner document. | boolean |
| `is_verifier_additional` | Existence of a valid additional document. | boolean |

**Note:** Relationships from `is_verifier_secondary_id` to `is_verifier_additional` are added to `secretIncludes` (do not appear in automatic API documentation), while `is_verifier_primary_id` is added to `availableIncludes`.

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

## Advanced Usage Examples

### 1. Including Comprehensive KYC Assessment (`kyc_status`)

**Request:**
```http
GET /api/v1/me?include=kyc_status HTTP/1.1
Authorization: Bearer ...
```

**Optional parameters:**
| Parameter | Description | Possible Values |
| :--- | :--- | :--- |
| `kyc_risk_level` | Risk level to determine mandatory documents. | `low`, `medium` (default), `high` |
| `kyc_category` | Filter by document category. | `primary_id`, `address`, `corporate`, ... |
| `kyc_document_type` | Filter by a specific document type. | `passport`, `utility_bill`, ... |
| `kyc_include_expired` | Include expired documents. | `true` (default), `false` |
| `kyc_include_pending` | Include pending documents. | `true` (default), `false` |
| `kyc_include_rejected` | Include rejected documents. | `true`, `false` (default) |

**Advanced example:**
```http
GET /api/v1/me?include=kyc_status&kyc_risk_level=high&kyc_category=primary_id&kyc_include_pending=false HTTP/1.1
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
            "Missing required document types: national_id"
        ]
    }
}
```

---

### 2. Including KYC Assessment by Category (`kyc_status_by_category`)

**Request:**
```http
GET /api/v1/me?include=kyc_status_by_category&kyc_category=address&kyc_document_type[]=utility_bill&kyc_document_type[]=bank_statement HTTP/1.1
Authorization: Bearer ...
```

**Optional parameters:**

| Parameter | Description |
| :--- | :--- |
| `kyc_category` | **Required**. Document category. If not specified, defaults to `primary_id`. |
| `kyc_document_type` | Specific document type or array of types. |
| `kyc_include_expired` | Include expired documents (`true`/`false`). |
| `kyc_include_pending` | Include pending documents (`true`/`false`). |
| `kyc_include_rejected` | Include rejected documents (`true`/`false`). |

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

**Optional parameters:**

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

### 5. Verification Summary (`kyc_verification_summary`)

**Request:**
```http
GET /api/v1/me?include=kyc_verification_summary HTTP/1.1
Authorization: Bearer ...
```

**Optional parameters:**
| Parameter | Description | Default |
| :--- | :--- | :--- |
| `kyc_verification_summary[include_pending]` | Include pending documents in the check. | `false` |
| `kyc_verification_summary[include_expired]` | Include expired documents. | `false` |
| `kyc_verification_summary[check_validity]` | Check date validity. | `true` |
| `kyc_verification_summary[expiring_within_days]` | Check documents expiring within a specified number of days. | `null` |

**Advanced example (check documents expiring within 30 days, including pending):**
```http
GET /api/v1/me?include=kyc_verification_summary&kyc_verification_summary[include_pending]=true&kyc_verification_summary[expiring_within_days]=30 HTTP/1.1
```

**Response structure:**
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

---

### 6. Boolean Verifiers for Categories

**Available relationships:**

| Key | Description |
| :--- | :--- |
| `is_verifier_primary_id` | Is there a valid primary identity document? |
| `is_verifier_secondary_id` | Is there a valid secondary identity document? |
| `is_verifier_address` | Is there a valid proof of address document? |
| `is_verifier_corporate` | Is there a valid corporate document? |
| `is_verifier_ubo` | Is there a valid ultimate beneficial owner document? |
| `is_verifier_additional` | Is there a valid additional document? |

**Basic request:**
```http
GET /api/v1/me?include=is_verifier_primary_id,is_verifier_address HTTP/1.1
```

**Optional parameters (per verifier):**
| Parameter | Description | Default |
| :--- | :--- | :--- |
| `is_verifier_primary_id[include_pending]` | Include pending documents. | `false` |
| `is_verifier_primary_id[include_expired]` | Include expired documents. | `false` |
| `is_verifier_primary_id[check_validity]` | Check date validity. | `true` |
| `is_verifier_primary_id[expiring_within_days]` | Check documents expiring within a specified number of days. | `null` |

**Advanced example (check identity expiring within 60 days):**
```http
GET /api/v1/me?include=is_verifier_primary_id&is_verifier_primary_id[expiring_within_days]=60 HTTP/1.1
```

**Response structure:**
```json
{
    "id": 24,
    "name": "Ahmed Mohammed",
    "is_verifier_primary_id": true,
    "is_verifier_address": false
}
```

---

### 7. Integrated Scenario (All Includes in One Request)

**Request:**
```http
GET /api/v1/me?include=kyc_verification_summary,kyc_documents_count,kyc_documents&kyc_documents[per_page]=10&kyc_verification_summary[expiring_within_days]=30 HTTP/1.1
Authorization: Bearer ...
```

**Response (abbreviated):**
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
    },
    "kyc_documents_count": 3,
    "kyc_documents": {
        "data": [ ... ],
        "meta": { ... }
    }
}
```

---

## Important Notes

- **Permissions:** Displayed KYC data is subject to user permissions. Regular users see only their own data.
- **Caching:** The behavior uses advanced request-level caching (`$documentsCache`) to avoid repeated database queries when multiple verifiers are requested together.
- **Alternative Groups:** In `kyc_status` and `kyc_status_by_category`, you can use `group:primary_identity` to refer to any document from the primary identity group.
- **Performance:** When using boolean verifiers, all owner documents are fetched only once per request, significantly improving performance.

---

## Additional Documentation

**Reference Documentation**:
- [General Plugin Documentation](./Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./Docs-Document-Model-en.md)
- [API Documentation](./Docs-API-Documentation-en.md)
