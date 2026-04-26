## 2026-04-25 - 2026-04-26

**`Nano3.Kyc` Plugin Updates – Version 1.2.6**

### Summary of Updates

After enhancing the security of KYC documents in version 1.2.5, version 1.2.6 brings comprehensive improvements to the **API** for fetching category and document type data. These updates aim to provide structured and flexible data suitable for front-end applications, with support for translation and advanced include options.

This release focuses on:

- **New endpoints** for fetching categories (`document-categories-list`) and document types (`document-types-list`).
- **Advanced functions in `DocumentType`** to return structured data supporting translation and relationship inclusion (types, details, fields, validation rules).
- **Improvements in error handling** within `FileUploadDocuments` to return detailed debugging information.
- **Standardization of API outputs** using Fractal for verification operations (verify, reject, restore).

These improvements make developing KYC user interfaces easier and more efficient.

---

### Version 1.2.6 – New Endpoints for Categories and Types, API Improvements

#### Version Goals

- Provide professional API endpoints to fetch KYC document categories and types with structured data.
- Support flexible include options (`include_types`, `include_details`, `include_fields`, `include_rules`, `include_category`).
- Improve `DocumentType` functions to generate responses compatible with front-end application standards.
- Standardize API responses for (verify, reject, restore) operations using Fractal Transformer.
- Improve file upload error reporting to facilitate debugging.

#### New Features

##### 1. New Endpoints for Categories and Types

Two new routes have been added to `routes.php`:

| Path | Description |
| :--- | :--- |
| `GET /api/v1/kyc/document-categories-list` | Fetch a list of document categories with advanced include options. |
| `GET /api/v1/kyc/document-categories-list/{id}` | Fetch details of a specific category with its types. |
| `GET /api/v1/kyc/document-types-list` | Fetch a list of document types with advanced include options. |
| `GET /api/v1/kyc/document-types-list/{id}` | Fetch details of a specific document type. |

##### 2. New and Improved Functions in `DocumentType`

| Function | Description |
| :--- | :--- |
| `isValidCategory($category)` | Check the validity of a category code. |
| `getCategoriesList($locale, $options)` | Fetch a list of categories with support for including types and details. |
| `getCategoryItem($code, $locale, $options)` | Fetch a single category item with options to include its types. |
| `getDocumentTypeList($locale, $options)` | Fetch a list of document types with support for category filtering and advanced include options. |
| `getDocumentTypeItem($code, $locale, $options)` | Fetch a single document type item with its full details. |

**Supported Include Options (`$options`):**

| Option | Description |
| :--- | :--- |
| `include_types` | Include document types within each category. |
| `include_details` | Include type details (weight, required fields, allowed files, etc.). |
| `include_fields` | Include the full field schema (`fields`) of the type. |
| `include_rules` | Include the validation rules (`rules`) for each field. |
| `include_category` | Include the parent category object within the type item. |
| `category_filter` | Filter types by a specific category (e.g., `primary_id`). |

##### 3. Improvements in File Upload Error Handling

The `create` and `update` functions in `FileUploadDocuments` have been improved to include:

- Return validation error details (`errors`) from `Manager`.
- Include debug information (`debug`) when `app.debug` is enabled.
- Use more precise exception messages.

##### 4. Standardization of API Outputs for Verification Operations

The `verify`, `reject`, and `restore` functions in the `Documents` API Controller have been modified to use `DocumentTransformer` via Fractal, ensuring data format consistency with other endpoints.

---

### Practical Examples

#### 1. Fetching Only the List of Categories

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
        ...
    ]
}
```

#### 2. Fetching a Specific Category with Its Types and Details

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
        "name": "هوية أساسية",
        "types": [
            {
                "code": "passport",
                "name": "Passport",
                "details": {
                    "verification_weight": 100,
                    "has_expiry": true,
                    "required_fields": ["document_front", "document_number", ...]
                }
            }
        ]
    }
}
```

#### 3. Fetching All Document Types with Parent Category, Fields, and Rules

**Request:**
```http
GET /api/v1/kyc/document-types-list?include_category=1&include_fields=1&include_rules=1 HTTP/1.1
Authorization: Bearer ...
```

#### 4. Fetching a Specific Document Type (Passport) with All Data

**Request:**
```http
GET /api/v1/kyc/document-types-list/passport?include_category=1&include_details=1&include_fields=1&include_rules=1 HTTP/1.1
Authorization: Bearer ...
```

---

### Version Summary (1.2.5 – 1.2.6)

| Version | Key Features |
| :--- | :--- |
| 1.2.5 | Integrated verified document protection system, duplicate prevention, improved verification process, backend permission integration. |
| 1.2.6 | New endpoints for categories and types (`document-categories-list`, `document-types-list`), advanced `DocumentType` functions, improved error handling, standardized API outputs. |

---

### Upgrade Requirements

1. **Update code**:
   - Replace `classes/DocumentType.php` with the new version.
   - Replace `apicontrollers/Documents.php` and `apicontrollers/documents/FileUploadDocuments.php`.
   - Update `routes.php` to add the new routes.

2. **Run database migrations** (no changes in this version).

3. **Test the new endpoints**:
   - Try `GET /api/v1/kyc/document-categories-list` with different include options.
   - Try `GET /api/v1/kyc/document-types-list/passport?include_fields=1`.

4. **Review the documentation**:
   - The `docs.md` files for the new endpoints have been updated with complete examples.

---

### Conclusion

With version 1.2.6, we have completed building a professional and flexible API for the KYC system, where developers can now easily fetch category and type data with full control over included data. This facilitates building dynamic forms and rich user interfaces without needing multiple requests or manual data processing.

We look forward to your feedback and suggestions to continue developing this essential add-on.

---

**Reference Documentation**:
- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`HasValidKycDocuments` Trait Documentation](./docs/Kyc/Docs-HasValidKycDocuments-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [`DynamicAddIncludeKyc` Behavior Documentation](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)




