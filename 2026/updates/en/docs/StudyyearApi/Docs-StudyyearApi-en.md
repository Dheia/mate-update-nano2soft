## Nano.StudyyearApi API Documentation

**Version:** 1.0.0  
**Target Audience:** Frontend application developers, external integration developers.

---

### Table of Contents

1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [General Response Structure](#general-response-structure)
4. [List of Endpoints](#list-of-endpoints)
   - [Academic Years (Periods)](#1-academic-years-periods)
   - [Semesters (Semsters)](#2-semesters-semsters)
   - [Academic Months (Months)](#3-academic-months-months)
   - [Activity Statistics (Actively Stats)](#4-activity-statistics-actively-stats)
5. [General Request Parameters](#general-request-parameters)
6. [Date Format](#date-format)
7. [Included Relations (Includes)](#included-relations-includes)
8. [Error Codes and Handling](#error-codes-and-handling)
9. [Notes for Developers](#notes-for-developers)

---

## Introduction

The `Nano.StudyyearApi` API provides RESTful endpoints to access academic calendar data managed by the `Tss.Studyyear` plugin. Application developers can use the API to fetch academic years (`Periods`), semesters (`Semsters`), and academic months (`Months`). The API is built according to the standard `Nano.*Api` pattern, ensuring consistency with other API services in the system.

**Base URL:**  
`https://yourdomain.com/api/v1/studyyear`

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
      "name": "Academic Year 2025-2026",
      ...
      "object_type": "Tss\\Studyyear\\Models\\Period"
    }
  ],
  "meta": {
    "pagination": {
      "total": 5,
      "count": 5,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {
        "next": null,
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
    "id": 3,
    "name": "First Semester",
    ...
  }
}
```

### Error Response

```json
{
  "error": {
    "code": 400,
    "message": "Listing periods is disabled for frontend users.",
    "errors": []
  }
}
```

---

## List of Endpoints

### 1. Academic Years (Periods)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/studyyear/periods` | Fetch a list of academic years |
| `GET` | `/api/v1/studyyear/periods/{id}` | Fetch a specific academic year |
| `GET` | `/api/v1/studyyear/periods/activelystats` | Activity statistics for academic years |

**Filtering Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `companys_id` | string | - | Filter by company ID |
| `departments_id` | string | - | Filter by branch ID |
| `isActive` | bool\|string | `1` | `1` for active only, `0` for inactive, `*` for all |
| `calendar_type` | string | `*` | Calendar type: `years` for years, `*` for all |
| `status` | string | `*` | Year status: `active` or `inactive`, `*` for all |
| `q` | string | - | Text search in `name`, `code`, `short_description` |
| `orderBy` | string | `sort_order` | Order by field |
| `orderDirection` | string | `asc` | Sort direction (`asc` or `desc`) |
| `page` | int | 1 | Page number |
| `per_page` | int | 15 | Items per page |
| `exclude` | string | - | Comma-separated field names to exclude |
| `include` | string | - | Included relations (`company`, `department`) |

**Available Relations:** `company`, `department`

**Example Request:**

```bash
GET /api/v1/studyyear/periods?isActive=1&include=department
```

**Example Response (one item from the list):**

```json
{
  "id": 5,
  "code": "YR-2025-2026",
  "name": "Academic Year 2025-2026",
  "short_description": "Full academic year",
  "description": "Academic year 2025-2026 for all stages",
  "calendar_type": "years",
  "from_date": "2025-09-01 00:00:00",
  "to_date": "2026-06-30 23:59:59",
  "companys_id": "1",
  "departments_id": "3",
  "is_published": true,
  "published_at": "2025-08-15 10:00:00",
  "unpublished_at": null,
  "is_default": true,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-01 08:30:00",
  "updated_at": "2025-08-20 14:15:00",
  "object_type": "Tss\\Studyyear\\Models\\Period",
  "department": {
    "data": {
      "id": "3",
      "name": "Eastern Region Branch"
    }
  }
}
```

---

### 2. Semesters (Semsters)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/studyyear/semsters` | Fetch a list of semesters |
| `GET` | `/api/v1/studyyear/semsters/{id}` | Fetch a specific semester |
| `GET` | `/api/v1/studyyear/semsters/activelystats` | Activity statistics for semesters |

**Filtering Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `companys_id` | string | - | Filter by company ID |
| `departments_id` | string | - | Filter by branch ID |
| `study_year_id` | string | **Automatic** | Academic year ID. **If not passed, the default year is used automatically.** |
| `isActive` | bool\|string | `1` | `1` for active only, `0` for inactive, `*` for all |
| `calendar_type` | string | `*` | Calendar type: `months`, `*` for all |
| `ref_type` | string | `*` | Semester type: `semster1`, `semster2`, `*` for all |
| `status` | string | `*` | Semester status: `active` or `inactive`, `*` for all |
| `q` | string | - | Text search in `name`, `code`, `short_description` |
| `orderBy` | string | `sort_order` | Order by field |
| `orderDirection` | string | `asc` | Sort direction (`asc` or `desc`) |
| `page` | int | 1 | Page number |
| `per_page` | int | 15 | Items per page |
| `exclude` | string | - | Comma-separated field names to exclude |
| `include` | string | - | Included relations (`company`, `department`, `period`) |

**Available Relations:** `company`, `department`, `period`

**Example Request (fetch semesters of the default year, including year):**

```bash
GET /api/v1/studyyear/semsters?include=period
```

**Example Response (one item):**

```json
{
  "id": 12,
  "code": "SEM-012",
  "name": "First Semester",
  "short_description": "First Semester 2025-2026",
  "description": "First semester of the academic year 2025-2026",
  "calendar_type": "months",
  "from_date": "2025-09-01 00:00:00",
  "to_date": "2026-01-15 23:59:59",
  "modul_type": "studyyear",
  "ref_type": "semster1",
  "study_year_id": "5",
  "companys_id": "1",
  "departments_id": "3",
  "is_published": true,
  "published_at": "2025-08-15 10:00:00",
  "unpublished_at": null,
  "is_default": false,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-10 09:00:00",
  "updated_at": "2025-08-20 14:20:00",
  "object_type": "Tss\\Studyyear\\Models\\Semster",
  "period": {
    "data": {
      "id": 5,
      "name": "Academic Year 2025-2026"
    }
  }
}
```

---

### 3. Academic Months (Months)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/studyyear/months` | Fetch a list of academic months |
| `GET` | `/api/v1/studyyear/months/{id}` | Fetch a specific academic month |
| `GET` | `/api/v1/studyyear/months/activelystats` | Activity statistics for academic months |

**Filtering Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `companys_id` | string | - | Filter by company ID |
| `departments_id` | string | - | Filter by branch ID |
| `study_year_id` | string | **Automatic** | Academic year ID. **If not passed, the default year is used automatically.** |
| `semsters_id` | string | - | Semester ID |
| `month_num` | string | `*` | Month number (1,2,3), `*` for all |
| `calendar_type` | string | `*` | Calendar type: `months`, `*` for all |
| `status` | string | `*` | Month status: `active` or `inactive`, `*` for all |
| `isActive` | bool\|string | `1` | `1` for active only, `0` for inactive, `*` for all |
| `q` | string | - | Text search in `name`, `code`, `short_description` |
| `orderBy` | string | `sort_order` | Order by field (useful: `month_num` for chronological order) |
| `orderDirection` | string | `asc` | Sort direction (`asc` or `desc`) |
| `page` | int | 1 | Page number |
| `per_page` | int | 15 | Items per page |
| `exclude` | string | - | Comma-separated field names to exclude |
| `include` | string | - | Included relations (`company`, `department`, `period`, `semster`) |

**Available Relations:** `company`, `department`, `period`, `semster`

**Example Request (fetch months of a specific semester, ordered chronologically):**

```bash
GET /api/v1/studyyear/months?semsters_id=12&orderBy=month_num&orderDirection=asc&include=semster
```

**Example Response (one item):**

```json
{
  "id": 35,
  "code": "MON-035",
  "name": "First Month - First Semester",
  "short_description": "September - October",
  "description": "First academic month of the first semester",
  "calendar_type": "months",
  "from_date": "2025-09-01 00:00:00",
  "to_date": "2025-10-01 23:59:59",
  "modul_type": "studyyear",
  "ref_type": "semster1",
  "study_year_id": "5",
  "semsters_id": "12",
  "month_num": "1",
  "month_number": "SEP",
  "companys_id": "1",
  "departments_id": "3",
  "is_published": true,
  "published_at": "2025-08-15 10:00:00",
  "unpublished_at": null,
  "is_default": false,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-15 10:00:00",
  "updated_at": "2025-08-20 14:25:00",
  "object_type": "Tss\\Studyyear\\Models\\Month",
  "semster": {
    "data": {
      "id": 12,
      "name": "First Semester"
    }
  }
}
```

---

### 4. Activity Statistics (Actively Stats)

Every resource has an `activelystats` endpoint that returns information about the last update of the data, useful for cache invalidation in client applications.

**Example Request:**  
`GET /api/v1/studyyear/periods/activelystats`

**Example Response:**

```json
{
  "data": {
    "code": 200,
    "data": {
      "last_updated": "2026-05-05 09:00:00",
      "activity_stats": false
    }
  }
}
```

- `last_updated`: Timestamp of the last update.
- `activity_stats`: `true` if there is activity more recent than the current time (usually `false`).

---

## General Request Parameters

These parameters apply to all `list` endpoints:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `isActive` | mixed | `1` | Activity filter: `1` active, `0` inactive, `*` all |
| `q` | string | - | Text search in primary fields (name, code, description). |
| `orderBy` | string | varies | Field name to order by (e.g. `sort_order`, `name`, `from_date`). |
| `orderDirection` | string | `asc` | Sort direction: `asc` ascending, `desc` descending. |
| `page` | int | 1 | Page number for pagination. |
| `per_page` | int | 15 | Items per page. |
| `exclude` | string | - | Comma-separated field names to exclude from response (e.g. `created_at,updated_at`). |
| `include` | string | - | Comma-separated relation names to include (e.g. `period,semster`). |

---

## Date Format

All dates in responses are in `Y-m-d H:i:s` format (e.g. `"2025-09-01 00:00:00"`). Empty fields are returned as `null`.

---

## Included Relations

To include related objects in the same request, use the `include` parameter with a comma-separated list of relation names.

| Resource | Available Relations |
|----------|---------------------|
| `periods` | `company`, `department` |
| `semsters` | `company`, `department`, `period` |
| `months` | `company`, `department`, `period`, `semster` |

**Example to include year and branch when fetching semesters:**

```
GET /api/v1/studyyear/semsters?include=period,department
```

If a relation does not exist or an error occurs, it is returned as an empty array `[]` instead of causing the request to fail.

---

## Error Codes and Handling

| Code | Meaning |
|------|---------|
| 200 | Success - data fetched. |
| 400 | Bad Request - invalid data or operation disabled by configuration or insufficient permissions. |
| 401 | Unauthorized - missing or invalid token. |
| 404 | Not Found - resource not found. |
| 500 | Internal Server Error. |

**Typical Error Body:**

```json
{
  "error": {
    "code": 400,
    "message": "Listing semesters is disabled for frontend users.",
    "errors": []
  }
}
```

- `message`: Error message in the specified language (default English).
- `errors`: Additional details about failed fields (if any).

---

## Notes for Developers

- **Automatic Default Academic Year:** When querying `semsters` or `months` without specifying `study_year_id`, the API automatically uses the **default academic year** (as set in the system). This saves you from passing the year every time, especially in applications operating within the current year context.
- **Chronological Order of Months:** To get months ordered chronologically within a semester, use `orderBy=month_num&orderDirection=asc`.
- **Data Format:** Note that `is_published`, `is_default`, `is_active` are boolean values (`true`/`false`), while `sort_order` is an integer (`int`).
- **Reducing Data Size:** Use `exclude` to avoid loading unnecessary fields like `config_data`, `other_data`, especially when fetching large lists.
- **Caching:** Use `activelystats` to determine whether data has changed since the last fetch, which helps with caching strategies in your application.

This documentation covers all API endpoints available in version 1.0.0 of `Nano.StudyyearApi`. For any questions or expansions, refer to the project's main developer guide.
