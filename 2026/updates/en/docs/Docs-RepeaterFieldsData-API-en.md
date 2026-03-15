# RepeaterFields Controller API Documentation

## Introduction

The `RepeaterFields` controller is part of the `Nano\BasicApi` plugin and provides an API interface for handling the repeatable fields supported by the `Tss\Basic\Classes\RepeaterFieldsData` class.  
This controller enables various operations on input data for these fields, such as normalization, formatting, extraction, merging, and validation.

**Base URL:**  
`/api/v1/basic/repeater-fields`

**Required Headers:**  
- `Accept: application/json`  
- `Authorization: Bearer {token}` (if authentication is required)

**All responses are in a unified JSON format containing the following fields:**
- `code`: HTTP status code
- `status`: true for success, false for failure
- `message`: Descriptive message
- `data`: Returned data (on success)
- `error`: Detailed error message (on failure)
- `errors`: Array of validation errors (optional)
- `debug`: Additional debugging information (only appears in debug mode)

---

## Endpoints

### 1. Get List of Supported Fields

**GET** `/api/v1/basic/repeater-fields/supported`

Returns a list of field names supported by the controller.

#### Request Example:
```http
GET /api/v1/basic/repeater-fields/supported HTTP/1.1
Host: example.com
Authorization: Bearer {token}
Accept: application/json
```

#### Successful Response Example:
```json
{
    "code": 200,
    "status": true,
    "message": "Supported fields retrieved successfully",
    "data": ["phone", "email", "website", "properties", "links"]
}
```

---

### 2. Process Input

**POST** `/api/v1/basic/repeater-fields/process`

Normalizes the input for a specific field and converts it into a unified structure ready for database storage. It handles all different formats: single string, array of strings, single object, array of objects, mixed.

#### Request Body (JSON):
| Field | Type   | Required | Description |
|-------|--------|----------|-------------|
| field | string | Yes      | Field name (phone, email, website, properties, links) |
| input | mixed  | Yes      | Input data in any supported format |

#### Examples:

##### Example 1: phone field with a single string
**Request:**
```json
{
    "field": "phone",
    "input": "770529482"
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "phone_label": "",
            "phone_number": "770529482",
            "phone_type": null,
            "sort_show": null,
            "is_default": "1",
            "is_show": "1",
            "phone_note": "",
            "_group": "phone"
        }
    ]
}
```

##### Example 2: email field with an array of strings
**Request:**
```json
{
    "field": "email",
    "input": ["user@example.com", "admin@site.com"]
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "email_label": "",
            "email_text": "user@example.com",
            "sort_show": null,
            "is_default": "1",
            "is_show": "1",
            "email_note": "",
            "_group": "email"
        },
        {
            "email_label": "",
            "email_text": "admin@site.com",
            "sort_show": null,
            "is_default": "1",
            "is_show": "1",
            "email_note": "",
            "_group": "email"
        }
    ]
}
```

##### Example 3: website field with a partial single object
**Request:**
```json
{
    "field": "website",
    "input": {
        "website_url": "https://example.com",
        "is_default": "0"
    }
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "website_label": "",
            "website_url": "https://example.com",
            "website_type": null,
            "sort_show": null,
            "is_default": "0",
            "is_show": "1",
            "website_icon": null,
            "website_note": "",
            "_group": "website"
        }
    ]
}
```

##### Example 4: properties field with mixed input (strings + objects)
**Request:**
```json
{
    "field": "properties",
    "input": [
        "Text value",
        {
            "title": "Color",
            "code": "color",
            "value": "Red"
        }
    ]
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "title": null,
            "code": null,
            "value": "Text value",
            "is_default": "1",
            "is_show": "1",
            "sort_show": null,
            "_group": "properties"
        },
        {
            "title": "Color",
            "code": "color",
            "value": "Red",
            "is_default": "1",
            "is_show": "1",
            "sort_show": null,
            "_group": "properties"
        }
    ]
}
```

---

### 3. Format Output

**POST** `/api/v1/basic/repeater-fields/format`

Takes stored data (as an array) and formats it for output, with options to filter hidden items and resolve media paths to full URLs.

#### Request Body (JSON):
| Field         | Type    | Required | Description |
|---------------|---------|----------|-------------|
| field         | string  | Yes      | Field name |
| data          | array   | Yes      | Stored data (array) |
| filter_hidden | boolean | No       | Hide items where `is_show = '0'` (default false) |
| resolve_media | boolean | No       | Convert `website_icon` paths to full URLs (default true) |

#### Example:
**Request:**
```json
{
    "field": "website",
    "data": [
        {
            "website_label": "Facebook",
            "website_url": "https://fb.com/page",
            "website_icon": "icons/fb.png",
            "is_show": "1",
            "_group": "website"
        },
        {
            "website_label": "Old Site",
            "website_url": "https://oldsite.com",
            "website_icon": null,
            "is_show": "0",
            "_group": "website"
        }
    ],
    "filter_hidden": true,
    "resolve_media": true
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data formatted successfully",
    "data": [
        {
            "website_label": "Facebook",
            "website_url": "https://fb.com/page",
            "website_icon": "https://example.com/storage/app/media/icons/fb.png",
            "is_show": "1",
            "_group": "website"
        }
    ]
}
```

---

### 4. Extract Main Values

**POST** `/api/v1/basic/repeater-fields/extract`

Extracts the main values from the data (e.g., phone numbers or email addresses) and returns them as an array.

#### Request Body (JSON):
| Field           | Type    | Required | Description |
|-----------------|---------|----------|-------------|
| field           | string  | Yes      | Field name |
| data            | array   | Yes      | Stored data |
| include_defaults| boolean | No       | Include only default items (`is_default = '1'`) (default false) |

#### Example:
**Request:**
```json
{
    "field": "phone",
    "data": [
        {"phone_number": "770529482", "is_default": "1"},
        {"phone_number": "780505400", "is_default": "0"},
        {"phone_number": "790505500", "is_default": "1"}
    ],
    "include_defaults": true
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Main values extracted successfully",
    "data": ["770529482", "790505500"]
}
```

---

### 5. Merge Data

**POST** `/api/v1/basic/repeater-fields/merge`

Merges new data with old data. It can either completely replace the old data or append the new data.

#### Request Body (JSON):
| Field    | Type    | Required | Description |
|----------|---------|----------|-------------|
| field    | string  | Yes      | Field name |
| old_data | array   | Yes      | Old data (array) |
| new_data | mixed   | Yes      | New data (can be in any format) |
| replace  | boolean | No       | If true, completely replaces old data with new data; otherwise, appends new data (default false) |

#### Example 1: Append (merge)
**Request:**
```json
{
    "field": "email",
    "old_data": [
        {"email_text": "old@site.com", "is_default": "1"}
    ],
    "new_data": ["new@site.com"]
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data merged successfully",
    "data": [
        {"email_text": "old@site.com", "is_default": "1", "_group": "email", ...},
        {"email_text": "new@site.com", "is_default": "1", "_group": "email", ...}
    ]
}
```

#### Example 2: Full replacement
**Request:**
```json
{
    "field": "phone",
    "old_data": [
        {"phone_number": "111111", "is_default": "1"}
    ],
    "new_data": ["222222", "333333"],
    "replace": true
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data merged successfully",
    "data": [
        {"phone_number": "222222", "is_default": "1", "_group": "phone", ...},
        {"phone_number": "333333", "is_default": "1", "_group": "phone", ...}
    ]
}
```

---

### 6. Validate Data

**POST** `/api/v1/basic/repeater-fields/validate`

Validates the input data against field definitions (required fields, allowed dropdown values). Returns `valid` (boolean) and a list of errors if any.

#### Request Body (JSON):
| Field | Type   | Required | Description |
|-------|--------|----------|-------------|
| field | string | Yes      | Field name |
| data  | array  | Yes      | Data to validate |

#### Example:
**Request:**
```json
{
    "field": "links",
    "data": [
        {"title": "Good Link", "url": "https://good.com"},
        {"url": "https://missing-title.com"}  // title is required
    ]
}
```
**Response:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data validated successfully",
    "data": {
        "valid": false,
        "errors": [
            "Field 'title' is required at index 1."
        ]
    }
}
```

---

### 7. Run Tests – For Developers Only

**GET** `/api/v1/basic/repeater-fields/tests`

Runs a suite of tests on all supported fields to verify that the class works correctly. This endpoint is protected: either `app.debug = true` or the user must have the `developer.access` permission.

#### Request Example:
```http
GET /api/v1/basic/repeater-fields/tests HTTP/1.1
Authorization: Bearer {token}
Accept: application/json
```

#### Response Example (abbreviated):
```json
{
    "code": 200,
    "status": true,
    "message": "Tests completed successfully",
    "data": {
        "summary": {
            "total": 45,
            "passed": 45,
            "failed": 0
        },
        "results": {
            "phone": [
                {
                    "case": "Single string",
                    "input": "770529482",
                    "output": [...],
                    "passed": true
                },
                ...
            ],
            ...
        }
    }
}
```

---

## Common Error Codes

| Code | Meaning |
|------|---------|
| 400  | Bad request (missing field, unsupported field, etc.) |
| 401  | Unauthorized |
| 403  | Forbidden |
| 404  | Not found |
| 422  | Validation error |
| 500  | Internal server error |

---

## Additional Notes

- All endpoints (except `supported` and `tests`) require authentication via the `oauth-users` middleware (as defined in the `routes.php` file).
- In production (`app.debug = false`), sensitive error details (e.g., SQL queries) are hidden and replaced with generic messages.
- The `tests` endpoint is only available in debug mode or for users with the `developer.access` permission.

---

This documentation covers all functionalities of the `RepeaterFields` controller with practical examples for each endpoint. It can be used as a reference for developers wishing to integrate this service into their applications.