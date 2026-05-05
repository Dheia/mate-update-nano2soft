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
