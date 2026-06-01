# Documentation of `Nano2.ProposalsApi` (Short Version)

## Introduction

The **Nano2.ProposalsApi** plugin provides a fully integrated API for managing **Proposals**, **Complaints**, and **Reports** within the NanoSoft App system. The plugin is designed according to the `Nano.*Api` pattern to allow users (individuals, students, parents) to easily create, view, and edit records, with support for categories, search, ordering, caching, and permission control.

---

## Basic Requirements

- NanoSoft App version 3.x or 2.x with Laravel support.
- `Nano.API` and `Nano.AuthApi` plugins to provide basic authentication and user management.
- Database containing the plugin tables after running `php artisan plugin:refresh Nano2.ProposalsApi`.
- **User must be logged in** to use any endpoint (JWT Token or Session Authentication).

---

## Core Concepts

### 1. Record Types (`type`)

| Value | Description |
|-------|-------------|
| `proposals` | General proposals or directed to a specific entity (student, company, etc.) |
| `complaints` | Complaints against a person or entity |
| `reports`  | Reports (often against violating behaviour) |

> **Note:** When creating `complaints` or `reports`, the fields `target_type` and `target_id` must be provided.

### 2. Target Object (`target_type` / `target_id`)

- `target_type`: The target model (e.g., `RainLab\User\Models\User`, `Tss\Student\Models\Student`, `Tss\Basic\Models\Department`, or even a symbolic value like `products`).
- `target_id`: The ID of the target object within that model.
- `target_type` can be left empty for general proposals.

### 3. Categories (`categories_id`)

- Version 1.0.2 and above supports categorising records by passing `categories_id` when creating or filtering when fetching.
- You can fetch category lists for each type via:
  - `/api/v1/proposals/categories?type=proposals`
  - `/api/v1/proposals/categories?type=complaints`
  - `/api/v1/proposals/categories?type=reports`

### 4. Parent Support (`is_stop_user`)

- Starting from version 1.0.3, a parent user (of type `parent` or `student`) can bypass the normal user filter using the parameter `is_stop_user=1`, provided they specify `target_type` and `target_id` for a specific student. This allows the parent to see all records related to that student.

---

## Endpoints

All endpoints start with `/api/v1/proposals/proposals` (except for categories and the `options` endpoint).

### 1. Fetch List (List)

```
GET /api/v1/proposals/proposals
```

#### Optional Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Text search in `name` and `content` |
| `type` | string | Filter by type: `proposals`, `complaints`, `reports` (all if omitted) |
| `categories_id` | integer/string | Filter by a specific category (can pass comma‑separated `1,2,3`) |
| `target_type` | string | Type of the target object (e.g., `Tss\Student\Models\Student`) |
| `target_id` | integer | ID of the target object |
| `is_stop_user` | boolean | If `1` and the user is `parent` or `student` and `target_type` is a student, the `User()` filter is bypassed, showing the specified student’s records |
| `isFavorites` | boolean | Fetch records favourited by the current user |
| `isLikes` | boolean | Fetch records liked by the current user |
| `isBookmarks` | boolean | Fetch records bookmarked by the current user |
| `companys_id` | integer | Filter by company |
| `departments_id` | integer | Filter by department |
| `orderBy` | string | Order by column (default `created_at`) |
| `orderDirection` | string | `asc` or `desc` (default `desc`) |
| `per_page` | integer | Number of results per page (default 15) |
| `page` | integer | Page number |
| `exclude` | string | Exclude columns from the result (e.g., `exclude=created_at,updated_at`) |
| `include` | string | Include relationships such as `categorie`, `user`, `companys`, `department` |

#### Example: Fetch proposals targeting a specific student (parent)

```http
GET /api/v1/proposals/proposals?type=proposals&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1
Authorization: Bearer {token}
```

#### Example: Fetch complaints against a specific student

```http
GET /api/v1/proposals/proposals?type=complaints&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1
```

#### Example: Fetch reports against a Frontend user

```http
GET /api/v1/proposals/proposals?type=reports&target_type=RainLab\User\Models\User&target_id=550
```

---

### 2. View a Single Record (Show)

```
GET /api/v1/proposals/proposals/{id}
```

| Parameter | Description |
|-----------|-------------|
| `id` | Record ID (required) |

#### Example

```http
GET /api/v1/proposals/proposals/2
Authorization: Bearer {token}
```

**Response** (example):

```json
{
  "id": 2,
  "name": "عنوان المقترح",
  "content": "نص المقترح",
  "type": "proposals",
  "categories_id": "5",
  "target_type": null,
  "target_id": 0,
  ...
}
```

---

### 3. Create a New Record (Create)

```
POST /api/v1/proposals/proposals/create
```

#### Required and Optional Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Required | Record title (3-250 characters) |
| `content` | string | Required | Record text (at least 3 characters) |
| `type` | string | Optional | `proposals` (default), `complaints`, `reports` |
| `categories_id` | integer | Optional | Category ID |
| `target_type` | string | **Required** for `complaints` and `reports` | Target model (e.g., `Tss\Student\Models\Student`) |
| `target_id` | integer | **Required** for `complaints` and `reports` | ID of the target object |
| `image` | string/base64 | Optional | Record image (can be passed as Base64 or file directly) |
| `is_private` | boolean | Optional | Whether the record is private (default `true`) |
| `date_at` | date | Optional | Record date (default now) |

> **Note:** When creating `reports`, it is recommended to pass a suitable `categories_id` to classify the report (e.g., bullying, abuse, etc.).

#### Example: Create a general proposal

```http
POST /api/v1/proposals/proposals/create
Content-Type: application/json
Authorization: Bearer {token}

{
  "name": "تطوير منصة التعلم",
  "content": "أقترح إضافة مكتبة فيديوهات تفاعلية...",
  "type": "proposals"
}
```

#### Example: Create a report against a student (parent)

```http
POST /api/v1/proposals/proposals/create
{
  "type": "reports",
  "categories_id": 6,
  "target_type": "Tss\\Student\\Models\\Student",
  "target_id": 8,
  "name": "التهجم على زميلة",
  "content": "تم التهجم على زميلة في الصف"
}
```

**Response**:

```json
{
  "code": 200,
  "status": true,
  "message": "تمت عملية الاضافة بنجاح",
  "proposals_id": 245,
  "data": { ... }
}
```

---

### 4. Update a Record (Update)

```
PUT /api/v1/proposals/proposals/update/{id}
```

Only the following fields can be updated (the type or target user cannot be changed):
- `name`
- `content`
- `categories_id`
- `is_active`
- `is_private`
- `date_at`
- `image`

#### Example

```http
PUT /api/v1/proposals/proposals/update/5
Content-Type: application/json
Authorization: Bearer {token}

{
  "name": "عنوان معدل",
  "content": "نص معدل"
}
```

---

### 5. Fetch Field Options (Options)

```
GET /api/v1/proposals/proposals/options
```

| Parameter | Description |
|-----------|-------------|
| `fields` | Field name (e.g., `type` or `status`) – returns options for a single field |
| `is_collection` | If `0`, returns options as a key=>value object; if `1`, returns an array of objects `{id, name}` (default `1`) |

#### Example: Fetch record types as an array of objects

```http
GET /api/v1/proposals/proposals/options?fields=type
```

```json
{
  "code": 200,
  "data": {
    "type": [
      {"id": "proposals", "name": "مقترحات"},
      {"id": "complaints", "name": "شكاوي"},
      {"id": "reports", "name": "بلاغ"}
    ]
  }
}
```

---

### 6. Check Last Update (Actively Stats)

```
GET /api/v1/proposals/proposals/activelystats
```

| Parameter | Description |
|-----------|-------------|
| `date` | Comparison date (format `Y-m-d H:i:s`) – optional, default now |

**Response**:

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,   // true if there is an update after the passed date
    "check_date": "2026-06-01 12:00:00",
    "last_updated": "2026-06-01 15:30:00",
    "other_updated": null
  }
}
```

---

## Data Models

### Structure of a Proposal / Complaint / Report Record

| Field | Type | Description |
|-------|------|-------------|
| `id` | int | Unique identifier |
| `code` | string | Auto‑generated code (company-department-number) |
| `name` | string | Title |
| `content` | string (HTML) | Text (supports HTML) |
| `type` | string | `proposals`, `complaints`, `reports` |
| `categories_id` | int | Associated category ID |
| `user_type` | string | Creator user type |
| `user_id` | int | Creator ID |
| `target_type` | string | Target object type |
| `target_id` | int | Target object ID |
| `is_active` | boolean | Active or not |
| `is_private` | boolean | Private (visible only to creator and admin) |
| `date_at` | datetime | Record date |
| `image` | object | Image object (if any) |
| `created_at` | datetime | Creation date |
| `updated_at` | datetime | Last update date |
| `object_type` | string | Constant `Nano2\Proposals\Models\Proposal` |

---

## Error Codes

| HTTP Code | Meaning |
|-----------|---------|
| 200 | Success (contains `status: true` in the body) |
| 400 | Bad request (missing data, validation fails) – body contains `status: false` and `errors` |
| 401 | Unauthorised (user not logged in or token invalid) |
| 404 | Record not found |

**Validation error example**:

```json
{
  "code": 400,
  "status": false,
  "message": "لم تتام عملية الاضافة بنجاح",
  "error": "الخاصية  name حقل مطلوب",
  "errors": {
    "name": ["الخاصية  name حقل مطلوب"]
  },
  "input_data": { ... },
  "debug": { ... } // only in debug mode
}
```

---

## Comprehensive Application Examples

### 1. Login and obtain a token (using Nano.AuthApi)

```http
POST /api/v1/auth/login
{
  "login": "parent@example.com",
  "password": "123456"
}
```

### 2. Fetch a student’s proposals (parent)

```http
GET /api/v1/proposals/proposals?type=proposals&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1&include=categorie
Authorization: Bearer {token}
```

### 3. Add a report against a store (Department)

```http
POST /api/v1/proposals/proposals/create
Authorization: Bearer {token}
{
  "type": "reports",
  "categories_id": 10,
  "target_type": "Tss\\Basic\\Models\\Department",
  "target_id": 4,
  "name": "تأخير في الشحن",
  "content": "تم تأخير الطلب أكثر من 10 أيام"
}
```

### 4. Update an existing complaint

```http
PUT /api/v1/proposals/proposals/42
{
  "is_private": false,
  "content": "تم تعديل النص بعد مراجعة الإدارة"
}
```

### 5. Search for reports containing the word "bullying"

```http
GET /api/v1/proposals/proposals?type=reports&q=تنمر&per_page=20
```

---

## Additional Documentation

- [Documentation of `Nano2.ProposalsApi`](./docs/ProposalsApi/Docs-ProposalsApi-en.md)
- [Comprehensive Practical Examples – ProposalsApi](./docs/ProposalsApi/Docs-ProposalsApi-Examples-en.md)
- [Nano API Plugin Development Guide (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)

---