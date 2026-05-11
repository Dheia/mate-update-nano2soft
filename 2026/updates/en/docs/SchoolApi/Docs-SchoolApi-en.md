## Nano.SchoolApi API Documentation

**Version:** 1.0.0  
**Target Audience:** Frontend application developers, external integration developers.

---

### Table of Contents

1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [General Response Structure](#general-response-structure)
4. [List of Endpoints](#list-of-endpoints)
   - [Classes](#1-classes)
   - [Stages](#2-stages)
   - [Subjects](#3-subjects)
   - [Groups](#4-groups)
   - [Class Groups Distribution](#5-classgroups)
   - [Class Subjects Distribution](#6-classsubjects)
   - [Teacher Subjects Distribution](#7-teachersubjects)
   - [Cost Centers Linked to Classes](#8-classcostcenters)
   - [Fee Types](#9-feestypes)
   - [Document Types](#10-documenttypes)
   - [External Schools](#11-outschools)
   - [Student Documents](#12-studentdocuments)
   - [Student Fees](#13-studentsfees)
   - [Last Year Student Marks](#14-laststudentmarks)
   - [Activity Statistics (Actively Stats)](#15-activity-statistics-actively-stats)
5. [General Request Parameters](#general-request-parameters)
6. [Date Format](#date-format)
7. [Included Relations (Includes)](#included-relations-includes)
8. [Error Codes and Handling](#error-codes-and-handling)

---

## Introduction

The `Nano.SchoolApi` API provides RESTful endpoints to access all core entities of the school management system (Tss.School). Frontend developers and external applications can use the API to retrieve classes, stages, subjects, groups, their distributions, as well as fees, documents, and student marks.

The API is built according to the standard `Nano.*Api` pattern, ensuring consistency with other API services in the system.

**Base URL:**  
`https://yourdomain.com/api/v1/school`

---

## Authentication

All endpoints are protected and require a valid **Bearer token**. The token is passed in the `Authorization` header in the format:

```
Authorization: Bearer <your_token>
```

The user and their type (backend / frontend) are verified to apply the access permissions defined for each endpoint. The user must possess the appropriate permissions in the backend system if the `is_check_list_permission` option is enabled.

---

## General Response Structure

### Successful Response (List)

```json
{
  "data": [
    {
      "id": 1,
      "name": "First Grade",
      ...
      "object_type": "Tss\\School\\Models\\TbClass"
    }
  ],
  "meta": {
    "pagination": {
      "total": 150,
      "count": 15,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 10,
      "links": {
        "next": "https://...?page=2",
        "previous": null
      }
    }
  }
}
```

### Successful Response (Single Item)

```json
{
  "data": {
    "id": 1,
    "name": "First Grade",
    ...
  }
}
```

### Error Response

```json
{
  "error": {
    "code": 400,
    "message": "Listing classes is disabled for frontend users.",
    "errors": []
  }
}
```

---

## List of Endpoints

### 1. Classes

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/classes` | Fetch a list of classes |
| `GET` | `/api/v1/school/classes/{id}` | Fetch a specific class |
| `GET` | `/api/v1/school/classes/activelystats` | Activity statistics for classes |

**Filtering Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `companys_id` | string | - | Filter by company ID |
| `departments_id` | string | - | Filter by branch ID |
| `stage_id` | string | - | Filter by stage ID |
| `isActive` | bool\|string | `1` | `1` for active only, `0` for inactive, `*` for all |
| `q` | string | - | Text search in name and code |
| `orderBy` | string | `sort_order` | Order by field |
| `orderDirection` | string | `asc` | Sort direction (`asc` or `desc`) |
| `page` | int | 1 | Page number |
| `per_page` | int | 15 | Items per page |
| `exclude` | string | - | Comma-separated field names to exclude |
| `include` | string | - | Included relations (company, department, stage) |

**Example Request:**

```bash
GET /api/v1/school/classes?stage_id=2&isActive=1&include=stage,department
```

**Example Response:**

```json
{
  "data": {
    "id": 5,
    "code": "CLS-005",
    "name": "Second Secondary Grade",
    "companys_id": "1",
    "departments_id": "3",
    "stage_id": "2",
    "study_fees": 2500.0,
    "registration_fees": 500.0,
    "class_rank": "2",
    "class_stage_rank": "2",
    "class_type": "primary",
    "is_active": true,
    "status": "active",
    "sort_order": 2,
    "created_at": "2025-08-15 10:30:00",
    "updated_at": "2026-01-10 14:22:00",
    "object_type": "Tss\\School\\Models\\TbClass",
    "stage": {
      "data": {
        "id": 2,
        "name": "Secondary Stage"
      }
    }
  }
}
```

---

### 2. Stages

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/stages` | Fetch a list of stages |
| `GET` | `/api/v1/school/stages/{id}` | Fetch a specific stage |
| `GET` | `/api/v1/school/stages/activelystats` | Activity statistics for stages |

**Filtering Parameters:** Similar to classes + `departments_id`.

**Available Relations:** `company`, `department`.

---

### 3. Subjects

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/subjects` | Fetch a list of subjects |
| `GET` | `/api/v1/school/subjects/{id}` | Fetch a specific subject |
| `GET` | `/api/v1/school/subjects/activelystats` | Activity statistics for subjects |

**Additional Parameters:** `departments_id`.

**Available Relations:** `company`, `department`.

---

### 4. Groups

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/groups` | Fetch a list of groups |
| `GET` | `/api/v1/school/groups/{id}` | Fetch a specific group |
| `GET` | `/api/v1/school/groups/activelystats` | Activity statistics for groups |

**Available Relations:** `company`, `department`.

---

### 5. Class Groups Distribution (ClassGroups)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/class-groups` | Fetch a list of group distributions |
| `GET` | `/api/v1/school/class-groups/{id}` | Fetch a specific distribution |
| `GET` | `/api/v1/school/class-groups/activelystats` | Activity statistics for distributions |

**Additional Parameters:**

| Parameter | Description |
|-----------|-------------|
| `class_id` | Filter by class ID |
| `group_id` | Filter by group ID |
| `year_id` | **Effectively required** - Academic year ID. If not passed, the system default year is used. |

**Available Relations:** `company`, `department`, `class`, `group`, `study_year`.

---

### 6. Class Subjects Distribution (ClassSubjects)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/class-subjects` | Fetch a list of subject distributions |
| `GET` | `/api/v1/school/class-subjects/{id}` | Fetch a specific distribution |
| `GET` | `/api/v1/school/class-subjects/activelystats` | Activity statistics for distributions |

**Additional Parameters:**

| Parameter | Description |
|-----------|-------------|
| `class_id` | Filter by class |
| `subject_id` | Filter by subject |
| `semster` | Filter by semester (`semster1`, `semster2`, `all`) |
| `subject_type` | Subject type (`primary`, `requirement`) |
| `subject_kind` | Subject kind (`academic`, `practical`, `activity`) |

**Available Relations:** `company`, `department`, `class`, `subject`.

---

### 7. Teacher Subjects Distribution (TeacherSubjects)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/teacher-subjects` | Fetch a list of teacher distributions |
| `GET` | `/api/v1/school/teacher-subjects/{id}` | Fetch a specific distribution |
| `GET` | `/api/v1/school/teacher-subjects/activelystats` | Activity statistics for distributions |

**Additional Parameters:**

| Parameter | Description |
|-----------|-------------|
| `class_id` | Filter by class |
| `group_id` | Filter by group |
| `subject_id` | Filter by subject |
| `teacher_id` | Filter by teacher |
| `year_id` | Academic year ID (default: primary year) |

**Available Relations:** `company`, `department`, `class`, `group`, `subject`, `teacher`, `study_year`.

---

### 8. Cost Centers Linked to Classes (ClassCostCenters)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/class-cost-centers` | Fetch a list of cost center links |
| `GET` | `/api/v1/school/class-cost-centers/{id}` | Fetch a specific link |
| `GET` | `/api/v1/school/class-cost-centers/activelystats` | Activity statistics for links |

**Additional Parameters:** `class_id`, `cost_centers_id`, `year_id`.

**Available Relations:** `company`, `department`, `class`, `cost_center`, `study_year`.

---

### 9. Fee Types (FeesTypes)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/fees-types` | Fetch a list of fee types |
| `GET` | `/api/v1/school/fees-types/{id}` | Fetch a specific fee type |
| `GET` | `/api/v1/school/fees-types/activelystats` | Activity statistics for fee types |

**Available Relations:** `company`, `department`.

---

### 10. Document Types (DocumentTypes)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/document-types` | Fetch a list of document types |
| `GET` | `/api/v1/school/document-types/{id}` | Fetch a specific document type |
| `GET` | `/api/v1/school/document-types/activelystats` | Activity statistics for document types |

**Available Relations:** `company`, `department`.

---

### 11. External Schools (OutSchools)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/out-schools` | Fetch a list of external schools |
| `GET` | `/api/v1/school/out-schools/{id}` | Fetch a specific school |
| `GET` | `/api/v1/school/out-schools/activelystats` | Activity statistics for external schools |

**Available Relations:** `company`, `department`.

---

### 12. Student Documents (StudentDocuments)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/student-documents` | Fetch a list of student documents |
| `GET` | `/api/v1/school/student-documents/{id}` | Fetch a specific document |
| `GET` | `/api/v1/school/student-documents/activelystats` | Activity statistics for student documents |

**Additional Parameters:**

| Parameter | Description |
|-----------|-------------|
| `student_id` | Filter by student |
| `document_id` | Filter by document type |
| `year_id` | Academic year ID |

**Available Relations:** `company`, `department`, `student`, `documents` (document type), `study_year`.

---

### 13. Student Fees (StudentsFees)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/students-fees` | Fetch a list of student fees |
| `GET` | `/api/v1/school/students-fees/{id}` | Fetch a specific fee |
| `GET` | `/api/v1/school/students-fees/activelystats` | Activity statistics for student fees |

**Additional Parameters:**

| Parameter | Description |
|-----------|-------------|
| `student_id` | Filter by student |
| `record_id` | Filter by student record |
| `fees_type_id` | Filter by fee type |
| `year_id` | Academic year ID |

**Available Relations:** `company`, `department`, `student`, `student_record`, `fees_type`, `study_year`.

---

### 14. Last Year Student Marks (LastStudentMarks)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/school/last-student-marks` | Fetch a list of last year marks |
| `GET` | `/api/v1/school/last-student-marks/{id}` | Fetch a specific mark |
| `GET` | `/api/v1/school/last-student-marks/activelystats` | Activity statistics for marks |

**Additional Parameters:**

| Parameter | Description |
|-----------|-------------|
| `student_id` | Filter by student |
| `class_id` | Filter by class |
| `out_school_id` | Filter by external school |
| `year_id` | Academic year ID |

**Available Relations:** `company`, `department`, `student`, `class`, `out_school`, `student_record`, `study_year`.

---

### 15. Activity Statistics (Actively Stats)

Every resource has an `/activelystats` endpoint that returns information about the last update of the data, useful for cache invalidation in client applications.

**Example Request:**

```bash
GET /api/v1/school/classes/activelystats
```

**Example Response:**

```json
{
  "data": {
    "code": 200,
    "data": {
      "last_updated": "2026-05-05 08:30:00",
      "activity_stats": false
    }
  }
}
```

- `last_updated`: Timestamp of the last update.
- `activity_stats`: `true` if there is activity more recent than the current time (usually `false` in live environments).

---

## General Request Parameters

These parameters apply to most `list` endpoints:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `isActive` | mixed | `1` | Activity filter: `1` active, `0` inactive, `*` all |
| `q` | string | - | Text search in primary fields (name, code). |
| `orderBy` | string | varies | Field name to order by (e.g. `sort_order`, `name`, `created_at`). |
| `orderDirection` | string | `asc` | Sort direction: `asc` ascending, `desc` descending. |
| `page` | int | 1 | Page number for pagination. |
| `per_page` | int | 15 | Items per page (max limit set in configuration). |
| `exclude` | string | - | Comma-separated field names to exclude from response (e.g. `created_at,updated_at`). |
| `include` | string | - | Comma-separated relation names to include (e.g. `stage,department`). |

**Note:** `isActive` values can be `"1"`, `"0"`, `"*"`, or `"all"` to disable the filter.

---

## Date Format

All dates in responses are in `Y-m-d H:i:s` format (e.g. `"2026-05-05 12:30:45"`). Empty fields are returned as `null`.

---

## Included Relations

To include related objects in the same request, use the `include` parameter with a comma-separated list of relation names.

**Example to include stage and department for a class:**

```
GET /api/v1/school/classes/5?include=stage,department
```

If a relation does not exist or an error occurs, it is returned as an empty array `[]` instead of causing the request to fail.

---

## Error Codes and Handling

| Code | Meaning |
|------|---------|
| 200 | Success - data fetched. |
| 400 | Bad Request - invalid data or operation disabled. |
| 401 | Unauthorized - missing or invalid token. |
| 404 | Not Found - resource not found. |
| 500 | Internal Server Error. |

**Typical Error Body:**

```json
{
  "error": {
    "code": 400,
    "message": "Listing classes is disabled for frontend users.",
    "errors": []
  }
}
```

- `message`: Error message in the specified language (default English).
- `errors`: Additional details about failed fields (if any).

---

## Notes for Developers

- Each endpoint and user type can be enabled/disabled through environment variables in the `config.php` file. Refer to the system administrator's documentation for configuration details.
- For resources that depend on `year_id` (such as `ClassGroups`), if not passed, the API automatically uses the **default academic year**, simplifying operations in the current context.
- Use `exclude` to reduce response size: for example `exclude=created_at,updated_at,deleted_at`.
- Caching in client applications is recommended, relying on the `last_updated` value from `activelystats`.

This documentation covers all API endpoints available in version 1.0.0 of `Nano.SchoolApi`. For any questions or expansions, refer to the project's main developer guide.
