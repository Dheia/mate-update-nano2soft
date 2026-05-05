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
