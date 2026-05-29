## 📘 Nano.HomeworkApi Documentation

**Version:** 1.0.0  
**Target audience:** Front‑end developers, external integration developers.

---

### Table of Contents

1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [General Response Structure](#general-response-structure)
4. [List of Endpoints](#list-of-endpoints)
   - [Homeworks and Activities](#1-homeworks-and-activities)
     - [Fetch a List of Homeworks](#fetch-a-list-of-homeworks)
     - [View a Specific Homework](#view-a-specific-homework)
     - [Create a New Homework](#create-a-new-homework)
     - [Update a Homework](#update-a-homework)
     - [Delete a Homework](#delete-a-homework)
   - [Submissions and Evaluations (Feedbacks)](#2-submissions-and-evaluations-feedbacks)
   - [Homework Classifications (Categories)](#3-homework-classifications-categories)
   - [General Classification Types (ClassTypes)](#4-general-classification-types-classtypes)
   - [Company‑Specific Classification Types (ClassTypeCompanies)](#5-company-specific-classification-types-classtypecompanies)
   - [Activity Statistics (Actively Stats)](#6-activity-statistics-actively-stats)
5. [General Request Parameters](#general-request-parameters)
6. [Date Format](#date-format)
7. [Included Relationships](#included-relationships)
8. [Error Codes and Handling](#error-codes-and-handling)

---

## Introduction

The `Nano.HomeworkApi` provides RESTful endpoints to access homework and activity data, student submissions and evaluations, as well as classifications and their types within the `Tss.Homework` system. Front‑end developers and external applications can use the API to manage assignments, record student submissions, and filter data in advanced ways.

**Base URL:**  
`https://yourdomain.com/api/v1/homework`

---

## Authentication

All endpoints are protected and require a valid **Bearer token**. The token is passed in the `Authorization` header as:

```
Authorization: Bearer <your_token>
```

The user and their type (backend / frontend) are validated to apply the specific access permissions defined for each endpoint and operation. The required permissions vary by operation (read, create, update, delete) and are configured via environment variables.

---

## General Response Structure

### Successful Response (List)

```json
{
  "data": [
    {
      "id": 42,
      "name": "Weekly Math Homework",
      "student_id": "15",
      "student_name": "Ahmed Mohamed",
      ...
      "object_type": "Tss\\Homework\\Models\\Homework"
    }
  ],
  "meta": {
    "pagination": {
      "total": 320,
      "count": 15,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 22,
      "links": {
        "next": "https://...?page=2",
        "previous": null
      }
    }
  }
}
```

### Successful Response (Single Item / Write Operation)

```json
{
  "data": {
    "id": 42,
    "name": "Weekly Math Homework",
    ...
  }
}
```

### Write Operation Response (Create / Update) with Unified Structure

```json
{
  "data": {
    "code": 200,
    "status": true,
    "message": "Homework created successfully.",
    "error": null,
    "errors": null,
    "data": { ... },
    "model": { ... },
    "input_data": {...},
    "process_data": {...}
  }
}
```

### Error Response

```json
{
  "error": {
    "code": 400,
    "message": "Listing homeworks is disabled for frontend users.",
    "errors": []
  }
}
```

---

## List of Endpoints

### 1. Homeworks and Activities

#### Fetch a List of Homeworks

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/homework/homeworks` | Fetch a list of homeworks and activities with advanced filtering |

**Homework‑specific filter parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `companys_id` | string | - | Filter by company ID |
| `departments_id` | string | - | Filter by department ID |
| `year_id` | string | - | Academic year ID. If not provided, the default year is used. |
| `semster` | string | `*` | Semester (`semster1` or `semster2` or `*` for all) |
| `class_id` | string | - | Class ID |
| `group_id` | string | - | Group ID. Supports `*` for all or `null` for any |
| `student_id` | string | - | Student ID. Supports `*` for all or `null` for any |
| `record_id` | string | - | Student record ID (`StudentRecord`). Supports `*` for all |
| `subject_id` | string | - | Subject ID |
| `teacher_id` | string | - | Teacher ID |
| `educators_id` | string | - | Class teacher ID |
| `categories_id` | string | - | Category ID |
| `ref_type_class` | string | - | Homework type (e.g., `homework`, `activity`) |
| `month_num` | string | `*` | Month number (1, 2, 3) |
| `min_degree` | number | - | Minimum grade (supports comparisons like `>=5`) |
| `max_degree` | number | - | Maximum grade (supports comparisons) |
| `target_rate` | number | - | Target completion percentage (supports comparisons) |
| `is_active` | bool | `true` | Active filter |
| `status` | string | `*` | Status (`active`, `inactive`, `proposed`) |
| `date_at` | string | - | Specific date in `Y-m-d` format |
| `q` | string | - | Text search in `name`, `short_description`, `description`, `solation` |

**Important notes:**
- The `student_id`, `record_id`, and `group_id` fields support special values:
  - If a specific ID is sent, records matching that ID **or** `*` **or** `null` are fetched (i.e., assignments meant for all students or all groups).
- `year_id`: if not provided, the default academic year is used automatically.

**Available relationships:** `company`, `department`, `category`, `type_class`, `student`, `record`, `subject`, `teacher`

**Example request:**

```bash
GET /api/v1/homework/homeworks?year_id=5&class_id=3&student_id=15&include=student,subject
```

**Example response (single item from the list):**

```json
{
  "id": 42,
  "code": "HW-042",
  "name": "Math Homework - Week 2",
  "short_description": "Solve equations from pages 10 to 15",
  "description": "<p>Solve all the equations...</p>",
  "solation": "<p>Sample solution...</p>",
  "categories_id": "5",
  "ref_type_class": "homework",
  "year_id": "5",
  "semster": "semster2",
  "class_id": "3",
  "group_id": "2",
  "student_id": "15",
  "student_name": "Ahmed Mohamed",
  "record_id": "8",
  "subject_id": "21",
  "teacher_id": "7",
  "educators_id": "12",
  "month_num": "2",
  "min_degree": 5.00,
  "max_degree": 10.00,
  "target_rate": 85.50,
  "is_default": false,
  "is_effective": true,
  "is_hidden": false,
  "is_active": true,
  "status": "active",
  "is_published": true,
  "published_at": "2026-05-01 08:00:00",
  "start_at": "2026-05-01 00:00:00",
  "end_at": "2026-05-07 23:59:59",
  "sort_order": 1,
  "created_at": "2026-05-01 08:00:00",
  "updated_at": "2026-05-01 08:00:00",
  "object_type": "Tss\\Homework\\Models\\Homework",
  "student": {
    "data": {
      "id": 15,
      "full_name": "Ahmed Mohamed"
    }
  }
}
```

#### View a Specific Homework

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/homework/homeworks/{id}` | View a specific homework with its relationships |

**Example:**  
`GET /api/v1/homework/homeworks/42?include=student,subject,type_class`

#### Create a New Homework

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/homework/homeworks` | Create a new homework or activity |

**Required data (JSON Body):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Homework title |
| `slug` | string | Yes | Unique identifier (auto‑generated from name if not provided) |
| `departments_id` | string | Yes | Department ID |
| `ref_type_class` | string | Yes | Homework type (e.g., `homework`, `activity`) |
| `year_id` | string | Yes* | Academic year ID (default is used if not provided) |
| `semster` | string | Yes | Semester (`semster1` or `semster2`) |
| `class_id` | string | Yes | Class ID |
| `subject_id` | string | Yes | Subject ID |
| `student_id` | string | No* | Student ID (if `*` or `null`, the homework is assigned to all students in the class) |
| `record_id` | string | No* | Student record ID |
| `group_id` | string | No* | Group ID (if `*`, the homework is assigned to all groups in the class) |
| `teacher_id` | string | No | Teacher ID |
| `educators_id` | string | No | Class teacher ID |
| `categories_id` | string | No | Category ID |
| `month_num` | string | No | Month number (1, 2, 3) |
| `min_degree` | float | No | Minimum grade |
| `max_degree` | float | No | Maximum grade |
| `target_rate` | float | No | Target completion percentage |
| `short_description` | string | No | Short description |
| `description` | string | No | Formatted description |
| `solation` | string | No | Sample solution |
| `start_at` | string | No | Start date (`Y-m-d H:i:s`) |
| `end_at` | string | No | End date |
| `is_active` | bool | No | `true` (default) |
| `is_published` | bool | No | `true` |
| `published_at` | string | No | Publication date |

\* If `companys_id`, `departments_id`, or `year_id` are not provided, they are automatically filled from the system default values.

**Example: Creating a homework for the whole class (`student_id = '*'`):**

```bash
POST /api/v1/homework/homeworks
Content-Type: application/json

{
  "name": "Weekly Math Homework",
  "departments_id": "1",
  "ref_type_class": "homework",
  "year_id": "5",
  "semster": "semster2",
  "class_id": "3",
  "subject_id": "21",
  "student_id": "*",
  "group_id": "*",
  "min_degree": 5,
  "max_degree": 10,
  "description": "Solve exercises pages 10-15"
}
```

**Example response (success):**

```json
{
  "data": {
    "code": 200,
    "status": true,
    "message": "Homework created successfully.",
    "data": {
      "id": 43,
      "name": "Weekly Math Homework",
      "student_id": "*",
      ...
    }
  }
}
```

#### Update a Homework

| Method | Path | Description |
|--------|------|-------------|
| `PUT` | `/api/v1/homework/homeworks/{id}` | Update an existing homework |

Only the fields to be updated need to be sent.

**Example: Changing the maximum grade:**

```bash
PUT /api/v1/homework/homeworks/42
Content-Type: application/json

{
  "max_degree": 12
}
```

#### Delete a Homework

| Method | Path | Description |
|--------|------|-------------|
| `DELETE` | `/api/v1/homework/homeworks/{id}` | Delete a homework (requires delete permission) |

**Example:**  
`DELETE /api/v1/homework/homeworks/42`

---

### 2. Submissions and Evaluations (Feedbacks)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/homework/feedbacks` | Fetch a list of student submissions |
| `GET` | `/api/v1/homework/feedbacks/{id}` | View a specific submission |
| `POST` | `/api/v1/homework/feedbacks` | Add a new submission (student evaluation) |
| `PUT` | `/api/v1/homework/feedbacks/{id}` | Update a submission |
| `GET` | `/api/v1/homework/feedbacks/activelystats` | Submission activity statistics |

**Feedback‑specific filter parameters:**

| Parameter | Description |
|-----------|-------------|
| `homeworks_id` | Homework ID |
| `student_id` | Student ID (supports `*` for all) |
| `record_id` | Academic record ID (supports `*` for all) |
| `evaluation` | Evaluation (e.g., `excellent`, `good`, `average`, `poor`) |
| `status_homework` | Submission status (`success`, `failed`) |
| `is_recive` | Received flag (`true`/`false`) |
| `is_degree` | Has a grade (`true`/`false`) |
| `degree` | Obtained grade (supports comparisons like `>=5`) |
| `target_rate` | Completion percentage |
| `date_at` | Submission date |
| `q` | Text search in `name`, `description`, `note` |

**Available relationships:** `homework`, `student`, `record`, `subject`, `teacher`

**Example: Fetch submissions for a specific homework:**

```bash
GET /api/v1/homework/feedbacks?homeworks_id=42&include=student
```

**Example response (list):**

```json
{
  "data": [
    {
      "id": 101,
      "homeworks_id": "42",
      "name": "Math Homework Submission",
      "student_id": "15",
      "student_name": "Ahmed Mohamed",
      "record_id": "8",
      "degree": 9.50,
      "evaluation": "excellent",
      "status_homework": "success",
      "is_recive": true,
      "note": "Well done",
      "teacher_note": "",
      "created_at": "2026-05-05 14:30:00"
    }
  ]
}
```

---

### 3. Homework Classifications (Categories)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/homework/categories` | Fetch a list of classifications (tree structure supported) |
| `GET` | `/api/v1/homework/categories/{id}` | Fetch a specific classification |
| `GET` | `/api/v1/homework/categories/activelystats` | Classification activity statistics |

**Filter parameters:**

| Parameter | Description |
|-----------|-------------|
| `companys_id` | Filter by company |
| `departments_id` | Filter by department |
| `ref_type_class` | Classification type |
| `parent_id` | Parent classification ID |
| `is_active` | Active filter |
| `q` | Text search in `name`, `slug`, `description` |

**Available relationships:** `company`, `department`, `type_class`, `parent`, `children`

**Example: Fetch only top‑level classifications:**  
`GET /api/v1/homework/categories?parent_id=null`

---

### 4. General Classification Types (ClassTypes)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/homework/class-types` | Fetch a list of general classification types |
| `GET` | `/api/v1/homework/class-types/{id}` | Fetch a specific type |
| `GET` | `/api/v1/homework/class-types/activelystats` | Activity statistics |

**Filter parameters:**

| Parameter | Description |
|-----------|-------------|
| `ref_type` | Reference code (e.g., `homework`, `activity`, `image`) |
| `is_active` | Active filter |
| `q` | Text search in `name`, `ref_type`, `description` |

**Available relationships:** `company`

---

### 5. Company‑Specific Classification Types (ClassTypeCompanies)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/homework/class-type-companies` | Fetch a list of company classifications |
| `GET` | `/api/v1/homework/class-type-companies/{id}` | Fetch a specific company classification |
| `GET` | `/api/v1/homework/class-type-companies/activelystats` | Activity statistics |

**Filter parameters:**

| Parameter | Description |
|-----------|-------------|
| `companys_id` | Filter by company |
| `departments_id` | Filter by department |
| `ref_type` | Reference code |
| `is_active` | Active filter |
| `q` | Text search in `name`, `ref_type`, `description` |

**Available relationships:** `company`, `department`, `class_type`

---

### 6. Activity Statistics (Actively Stats)

Every resource has an `activelystats` endpoint that returns information about the last data update, useful for invalidating caches in client applications.

**Example request:**  
`GET /api/v1/homework/homeworks/activelystats`

**Example response:**

```json
{
  "data": {
    "code": 200,
    "data": {
      "last_updated": "2026-05-05 09:15:00",
      "activity_stats": false
    }
  }
}
```

---

## General Request Parameters

The following parameters apply to most `list` endpoints:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `isActive` | mixed | `1` | Filter active status: `1` active, `0` inactive, `*` all |
| `q` | string | - | Text search in main fields. |
| `orderBy` | string | varies | Field name to order by. |
| `orderDirection` | string | `asc` | Order direction: `asc` or `desc`. |
| `page` | int | 1 | Page number. |
| `per_page` | int | 15 | Number of items per page. |
| `exclude` | string | - | Comma‑separated list of field names to exclude from the response. |
| `include` | string | - | Comma‑separated list of relationship names to include. |

**Note about `isActive`:** Values `"1"`, `"0"`, `"*"`, or `"all"` can be used.

---

## Date Format

All dates in responses are in the format `Y-m-d H:i:s` (example: `"2026-05-05 12:30:45"`). Empty fields are returned as `null`.

---

## Included Relationships

To include related objects in the same request, use the `include` parameter with comma‑separated relationship names.

**Example: Including student and subject in a homework:**

```
GET /api/v1/homework/homeworks/42?include=student,subject
```

If the relationship does not exist or an error occurs, an empty array `[]` is returned instead of failing the request.

---

## Error Codes and Handling

| Code | Meaning |
|------|---------|
| 200 | Success – data was retrieved or the operation was performed. |
| 400 | Bad request – invalid parameters, operation disabled by configuration, or insufficient permissions. |
| 401 | Unauthorised – token missing or invalid. |
| 404 | Not found – the requested resource does not exist. |
| 500 | Internal server error. |

**Typical error body:**

```json
{
  "error": {
    "code": 400,
    "message": "You do not have permission to list homeworks.",
    "errors": []
  }
}
```

- `message`: The error message in the selected language (the default in this documentation is Arabic, but the API returns it according to the user’s language).
- `errors`: Additional details about the failed fields (especially for create/update operations when validation fails).

---

## Notes for Developers

- **General homeworks for the whole class/group:** When creating a homework, you can set `student_id = "*"` or `group_id = "*"` to distribute the homework to all students or all groups. When fetching data, these records will be included in regular queries because the filtering logic treats `*` and `null` as valid values.
- **Grades and comparisons:** You can use comparisons such as `>=5` or `['>', 10]` with the fields `min_degree`, `max_degree`, `target_rate`, and `degree` to apply advanced conditions.
- **Default values:** In create operations (`POST`), if you do not send `year_id`, `companys_id`, or `departments_id`, the API will use the system default settings, making registration easier in the current context.
- **Permissions:** Access to each operation (view, create, update, delete) for each user type (backend/frontend) is controlled via environment variables. Contact your system administrator for customisation.
- **Caching:** Use `activelystats` to determine whether your data is up‑to‑date and whether you need to refetch it.

This documentation covers all API endpoints available in version 1.0.0 of `Nano.HomeworkApi`. For any questions or extensions, refer to the main project developer guide.
