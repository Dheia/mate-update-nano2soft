# Documentation for Creating a KYC Document via API – Scenario

## Overview

This guide explains the complete steps for building an interactive front-end to create a new KYC document. The scenario includes:

1. Fetching document categories (optional) – to allow the user to select a category first.
2. Fetching available document types (with optional filtering by category).
3. Fetching the dynamic fields for the selected document type – to build the input form.
4. Submitting the form data with attachments to the document creation endpoint.

All requests require authentication via `Authorization: Bearer <token>`.

---

## Step 1: Fetch Document Categories (Optional)

The developer can display a list of categories first so the user can select the desired class (e.g., primary identity, proof of address).

**Endpoint:** `GET /api/v1/kyc/document-categories-list`

**Usage:**
```http
GET /api/v1/kyc/document-categories-list HTTP/1.1
Authorization: Bearer {{access_token}}
```

**Response (without including types):**
```json
{
    "code": 200,
    "status": true,
    "data": [
        {
            "code": "primary_id",
            "name_ar": "Primary Identity",
            "name_en": "Primary ID",
            "name": "Primary Identity"
        },
        {
            "code": "secondary_id",
            "name_ar": "Secondary Identity",
            "name_en": "Secondary ID",
            "name": "Secondary Identity"
        }
        // ... remaining categories
    ]
}
```

> **Note:** You can request categories along with their types directly to save an extra request (see Step 2).

---

## Step 2: Fetch Document Types

After the user selects a category (or directly), we fetch the list of available document types. The list can be filtered by category to reduce options.

**Endpoint:** `GET /api/v1/kyc/document-types-list`

### a. Fetch All Types (No Filtering)

```http
GET /api/v1/kyc/document-types-list HTTP/1.1
Authorization: Bearer {{access_token}}
```

### b. Fetch Only Types for a Specific Category

To show only types belonging to "Primary Identity":

```http
GET /api/v1/kyc/document-types-list?category_filter=primary_id HTTP/1.1
```

**Response (example for two types):**
```json
{
    "code": 200,
    "status": true,
    "data": [
        {
            "code": "passport",
            "name_ar": "Passport",
            "name_en": "Passport",
            "name": "Passport",
            "description": "A valid passport containing a photo and traveler data."
        },
        {
            "code": "national_id",
            "name_ar": "National ID Card",
            "name_en": "National ID Card",
            "name": "National ID Card",
            "description": "National ID card issued by official authorities."
        }
    ]
}
```

> **Tip:** You can include `include_category=1` to get the parent category information for each type.

---

## Step 3: Fetch Document Type Fields (To Build the Form)

After the user selects a document type (e.g., `passport`), we fetch the field definition for that type to dynamically build the input form.

**Endpoint:** `GET /api/v1/kyc/document-fields/{type}`

**Example for passport:**
```http
GET /api/v1/kyc/document-fields/passport HTTP/1.1
Authorization: Bearer {{access_token}}
```

**Response (abbreviated):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "type": "passport",
        "type_name": "Passport",
        "fields": [
            {
                "id": "document_front",
                "type": "fileupload",
                "mode": "image",
                "label": { "ar": "Document Front", "en": "Document Front" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "mimes", "value": "jpeg,jpg,png,pdf" },
                    { "rule": "max", "value": 10240 }
                ]
            },
            {
                "id": "document_number",
                "type": "text",
                "label": { "ar": "Document Number", "en": "Document Number" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "max", "value": 50 }
                ]
            },
            {
                "id": "full_name",
                "type": "text",
                "label": { "ar": "Full Name", "en": "Full Name" },
                "required": false,
                "validation": [ { "rule": "max", "value": 100 } ]
            },
            {
                "id": "issue_date",
                "type": "date",
                "label": { "ar": "Issue Date", "en": "Issue Date" },
                "required": false,
                "validation": [
                    { "rule": "date" },
                    { "rule": "before_or_equal", "value": "today" }
                ]
            },
            {
                "id": "expiry_date",
                "type": "date",
                "label": { "ar": "Expiry Date", "en": "Expiry Date" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "date" },
                    { "rule": "after_or_equal", "value": "today" }
                ]
            },
            {
                "id": "country_of_issue",
                "type": "select",
                "label": { "ar": "Country of Issue", "en": "Country of Issue" },
                "required": true,
                "data_source": {
                    "type": "api",
                    "endpoint": "api/v1/location/countries",
                    "value_field": "code",
                    "label_field": "name"
                },
                "resolved_options": [
                    { "id": "SA", "name": "Saudi Arabia" },
                    { "id": "EG", "name": "Egypt" }
                    // ...
                ]
            }
            // ... remaining fields
        ],
        "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
        "max_file_size_kb": 10240,
        "requires_back_side": false,
        "requires_selfie_match": true
    }
}
```

### Explanation of Important Properties

| Property | Description |
|----------|-------------|
| `type` | Input field type (`text`, `date`, `select`, `fileupload`, ...). |
| `required` | Is the field mandatory? |
| `validation` | Validation rules (can be converted to Laravel rules or used for client-side validation). |
| `data_source` | Source for dropdown options (external API). |
| `resolved_options` | Pre-resolved options ready for use in `<select>` (fetched automatically). |
| `mode` | For file fields (`image`, `file`). |
| `allowed_mime_types` | Allowed file extensions. |

> **Developer Tip:** Use `validation` to build instant validation rules on the front-end, and use `resolved_options` to populate dropdown lists.

---

## Step 4: Submit Document Data and Files

After the user fills out the form, we send the data to the creation endpoint. The endpoint supports file submission in two ways:

- **multipart/form-data** (direct file upload).
- **JSON with Base64** (by adding the `_base64` suffix to the field name).

**Endpoint:** `POST /api/v1/kyc/documents`

### a. Submission using multipart/form-data

**Example (JavaScript – FormData):**
```javascript
const formData = new FormData();
formData.append('document_type', 'passport');
formData.append('document_number', 'P12345678');
formData.append('full_name', 'Ahmed Mohammed');
formData.append('issue_date', '2020-01-01');
formData.append('expiry_date', '2030-01-01');
formData.append('country_of_issue', 'YE');
formData.append('nationality', 'YE');
formData.append('document_front', fileInput.files[0]);

fetch('/api/v1/kyc/documents', {
    method: 'POST',
    headers: { 'Authorization': 'Bearer ' + token },
    body: formData
});
```

### b. Submission using JSON + Base64

**Example (JavaScript):**
```javascript
const payload = {
    document_type: 'passport',
    document_number: 'P12345678',
    full_name: 'Ahmed Mohammed',
    issue_date: '2020-01-01',
    expiry_date: '2030-01-01',
    country_of_issue: 'YE',
    nationality: 'YE',
    document_front_base64: 'data:image/jpeg;base64,/9j/4AAQSkZJRg...'
};

fetch('/api/v1/kyc/documents', {
    method: 'POST',
    headers: {
        'Authorization': 'Bearer ' + token,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(payload)
});
```

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `document_type` | string | Yes | Document type code (e.g., `passport`). |
| `owner_type` / `owner_id` | string/int | No* | If the user is `backend`, they can specify another owner. |
| **Form fields** | varied | Depends on type | All visible fields in `fields` (e.g., `document_number`, `full_name`). |
| **Files** | file/base64 | Depends on type | `document_front`, `document_back`, `signature_image`, `image`, `files[]`. |

> *For `frontend` users, they are automatically set as the owner.

### Success Response (Document Created and Files Uploaded)

```json
{
    "code": 200,
    "status": true,
    "message": "Document created successfully.",
    "data": {
        "id": 142,
        "document_type": "passport",
        "document_number": "P12345678",
        "full_name": "Ahmed Mohammed",
        "status": "pending",
        "created_at": "2026-04-22 14:30:00",
        "document_front": {
            "original": "https://account.now-ye.com/storage/app/uploads/public/.../original.jpg",
            "thumb": "https://account.now-ye.com/storage/app/uploads/public/.../thumb.jpg"
        }
    },
    "process_data": {
        "file_errors": []
    }
}
```

### Partial Success Response (Some Files Failed)

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

### Error Response (Validation Failed)

```json
{
    "code": 400,
    "status": false,
    "message": "An error occurred while creating the document.",
    "error": "The document number field is required.",
    "errors": {
        "document_number": ["The document number field is required."],
        "expiry_date": ["The expiry date field is required."]
    }
}
```

---

## Workflow Summary

| Step | Action | Endpoint |
|------|--------|----------|
| 1 | (Optional) Display categories | `GET /document-categories-list` |
| 2 | Display document types (filtered by category) | `GET /document-types-list?category_filter=...` |
| 3 | Fetch fields for the selected type | `GET /document-fields/{type}` |
| 4 | Build the form dynamically based on `fields` | - |
| 5 | Submit data + files | `POST /documents` (multipart or JSON+Base64) |
| 6 | Show creation result (success / failure) | - |

---

## Developer Tips

- **Use `resolved_options` directly** in dropdowns; do not make a separate API request.
- **Validate against `validation`** to build local validation rules before submission.
- **Handle `file_errors`** in the response to inform the user about failed uploads.
- **When uploading large files**, prefer using `multipart/form-data`.
- **For applications that do not support multipart** (e.g., some mobile environments), use Base64.
