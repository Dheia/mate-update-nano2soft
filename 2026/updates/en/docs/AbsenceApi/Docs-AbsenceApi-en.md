## Nano.AbsenceApi API Documentation

**Version:** 1.0.0  
**Target Audience:** Frontend application developers, external integration developers.

---

### Table of Contents

1. [Introduction](#introduction)
2. [Authentication](#authentication)
3. [General Response Structure](#general-response-structure)
4. [List of Endpoints](#list-of-endpoints)
   - [Attendance & Absence Records (Absences)](#1-attendance--absence-records-absences)
     - [Fetch List of Absence Records](#fetch-list-of-absence-records)
     - [View a Specific Absence Record](#view-a-specific-absence-record)
     - [Create a New Absence Record](#create-a-new-absence-record)
     - [Update an Absence Record](#update-an-absence-record)
   - [Attendance Sheets (Files)](#2-attendance-sheets-files)
   - [Main Classifications (ClassTypes)](#3-main-classifications-classtypes)
   - [Company Classifications (ClassTypeCompanies)](#4-company-classifications-classtypecompanies)
   - [Activity Statistics (Actively Stats)](#5-activity-statistics-actively-stats)
5. [General Request Parameters](#general-request-parameters)
6. [Date Format](#date-format)
7. [Included Relations (Includes)](#included-relations-includes)
8. [Error Codes and Handling](#error-codes-and-handling)

---

## Introduction

The `Nano.AbsenceApi` API provides RESTful endpoints to access attendance, absence, attendance sheets, and their classifications data within the `Tss.Absence` system. Frontend developers and external applications can use the API to fetch absence records, create new ones, update them, and query sheets and classifications.

**Base URL:**  
`https://yourdomain.com/api/v1/absences`

---

## Authentication

All endpoints are protected and require a valid **Bearer token**. The token is passed in the `Authorization` header in the format:

```
Authorization: Bearer <your_token>
```

The user and their type (backend / frontend) are verified to apply the specific access permissions defined for each endpoint and operation. Required permissions vary by operation (read, create, update) and are pre-configured.

---

## General Response Structure

### Successful Response (List)

```json
{
  "data": [
    {
      "id": 42,
      "student_name": "Ahmed Mohamed",
      "absences_status": "absent",
      ...
      "object_type": "Tss\\Absence\\Models\\Absence"
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

### Successful Response (Single Item / Create or Update Operation)

```json
{
  "data": {
    "id": 42,
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
    "message": "Absence record created successfully.",
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
    "message": "Listing absences is disabled for frontend users.",
    "errors": []
  }
}
```

---

## List of Endpoints

### 1. Attendance & Absence Records (Absences)

#### Fetch List of Absence Records

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/absence/absences` | Fetch a list of attendance and absence records with advanced filtering |

**Filtering Parameters specific to Absences:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `companys_id` | string | - | Filter by company ID |
| `departments_id` | string | - | Filter by branch ID |
| `year_id` | string | - | Academic year ID. If not passed, the default year is used. |
| `semster` | string | - | Semester (`semster1` or `semster2`) |
| `semsters_id` | string | - | Semester ID (Semster model from Studdyear) |
| `months_id` | string | - | Academic month ID (Month model) |
| `month_num` | string | - | Month number within the semester (1,2,3) |
| `class_id` | string | - | Class ID |
| `group_id` | string | - | Group ID |
| `student_id` | string | - | Student ID |
| `record_id` | string | - | Student record ID (StudentRecord) |
| `absences_status` | string | `*` | Attendance status: `present`, `absent`, `permitted`, `*` for all |
| `files_id` | string | - | Sheet file ID |
| `ref_type_class` | string | - | Sheet type (e.g. `day`, `subject`, `exam`) |
| `date_at` | string | - | Specific date (in `Y-m-d` format) |
| `q` | string | - | Text search in `student_name`, `absences_bacuse`, `notes` |

**Available Relations:** `company`, `department`, `file`, `student`, `record`

**Example Request:**

```bash
GET /api/v1/absence/absences?date_at=2026-05-01&class_id=3&include=student,file
```

**Example Response (one item from the list):**

```json
{
  "id": 42,
  "code": "ABS-042",
  "student_id": "15",
  "student_name": "Ahmed Mohamed",
  "record_id": "8",
  "class_id": "3",
  "group_id": "2",
  "year_id": "5",
  "semster": "semster2",
  "month_num": "2",
  "absences_status": "absent",
  "absences_bacuse": "Sick",
  "notes": "Absent with excuse",
  "date_at": "2026-05-01 00:00:00",
  "is_finality": false,
  "is_active": true,
  "files_id": "5",
  "ref_type_class": "day",
  "created_at": "2026-05-01 08:00:00",
  "updated_at": "2026-05-01 08:00:00",
  "object_type": "Tss\\Absence\\Models\\Absence",
  "student": {
    "data": {
      "id": 15,
      "full_name": "Ahmed Mohamed"
    }
  }
}
```

#### View a Specific Absence Record

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/absence/absences/{id}` | View a specific absence record with relations |

**Example:**  
`GET /api/v1/absence/absences/42?include=student,file`

#### Create a New Absence Record

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/v1/absence/absences` | Create a new absence record |

**Required Data (JSON Body):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `student_id` | string | Yes | Student ID |
| `record_id` | string | Yes | Student record ID (StudentRecord) |
| `year_id` | string | No* | Academic year ID (taken from default if not specified) |
| `class_id` | string | Yes | Class ID |
| `group_id` | string | No | Group ID |
| `semster` | string | Yes | Semester (`semster1` or `semster2`) |
| `month_num` | string | No | Month number (1-3) |
| `files_id` | string | Yes | Sheet file ID |
| `ref_type_class` | string | Yes | Sheet type (`day`, `subject`, `exam`, `activete`) |
| `absences_status` | string | No | Attendance status: `present`, `absent`, `permitted` (default `present`) |
| `absences_bacuse` | string | No | Reason for absence |
| `notes` | string | No | Notes |
| `date_at` | string | Yes | Date in `Y-m-d` format |

* If `companys_id`, `departments_id`, or `year_id` are not passed, they are automatically filled from system defaults.

**Example Request:**

```bash
POST /api/v1/absence/absences
Content-Type: application/json

{
  "student_id": "15",
  "record_id": "8",
  "year_id": "5",
  "class_id": "3",
  "semster": "semster2",
  "month_num": "2",
  "files_id": "5",
  "ref_type_class": "day",
  "absences_status": "absent",
  "absences_bacuse": "Sick",
  "date_at": "2026-05-05"
}
```

**Example Response (Success):**

```json
{
  "data": {
    "code": 200,
    "status": true,
    "message": "Absence record created successfully.",
    "error": null,
    "errors": null,
    "data": {
      "id": 43,
      "student_name": "Ahmed Mohamed",
      "absences_status": "absent",
      ...
    },
    "model": { ... },
    "input_data": { ... },
    "process_data": { ... }
  }
}
```

#### Update an Absence Record

| Method | Path | Description |
|--------|------|-------------|
| `PUT` | `/api/v1/absence/absences/{id}` | Update an existing absence record |

Only the fields to be updated are sent. Unmentioned data will not change. The same permissions and validation rules apply (e.g., no duplicate record for the same student on the same day and sheet).

**Example to change absence status to "present":**

```bash
PUT /api/v1/absence/absences/42
Content-Type: application/json

{
  "absences_status": "present"
}
```

**Response structure is identical to the create structure.**

---

### 2. Attendance Sheets (Files)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/absence/files` | Fetch a list of attendance sheets |
| `GET` | `/api/v1/absence/files/{id}` | Fetch a specific sheet |
| `GET` | `/api/v1/absence/files/activelystats` | Activity statistics for sheets |

**Filtering Parameters:**

| Parameter | Description |
|-----------|-------------|
| `companys_id` | Filter by company |
| `departments_id` | Filter by branch |
| `ref_type_class` | Filter by sheet type (`day`, `subject`, etc.) |
| `isActive` | `1` for active, `0` for inactive |
| `q` | Text search in `name`, `slug`, `short_description` |

**Available Relations:** `company`, `department`.

---

### 3. Main Classifications (ClassTypes)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/absence/class-types` | Fetch a list of main classification types for sheets |
| `GET` | `/api/v1/absence/class-types/{id}` | Fetch a specific classification |
| `GET` | `/api/v1/absence/class-types/activelystats` | Activity statistics for classifications |

**Filtering Parameters:**

| Parameter | Description |
|-----------|-------------|
| `ref_type` | Filter by reference code (e.g. `day`, `subject`) |
| `isActive` | Activity filter |
| `q` | Search in `name`, `ref_type`, `description` |

**Available Relations:** `company`

---

### 4. Company Classifications (ClassTypeCompanies)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/api/v1/absence/class-type-companies` | Fetch a list of company classifications |
| `GET` | `/api/v1/absence/class-type-companies/{id}` | Fetch a specific company classification |
| `GET` | `/api/v1/absence/class-type-companies/activelystats` | Activity statistics for classifications |

**Filtering Parameters:**

| Parameter | Description |
|-----------|-------------|
| `companys_id` | Filter by company |
| `departments_id` | Filter by branch |
| `ref_type` | Filter by reference code |
| `isActive` | Activity filter |
| `q` | Search in `name`, `ref_type`, `description` |

**Available Relations:** `company`, `department`, `class_type` (main classification).

---

### 5. Activity Statistics (Actively Stats)

Every resource has an `activelystats` endpoint that returns information about the last update of the data, useful for cache invalidation in client applications.

**Example Request:**  
`GET /api/v1/absence/absences/activelystats`

**Example Response:**

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

These parameters apply to most `list` endpoints:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `isActive` | mixed | `1` | Activity filter: `1` active, `0` inactive, `*` all |
| `q` | string | - | Text search in primary fields. |
| `orderBy` | string | varies | Field name to order by. |
| `orderDirection` | string | `asc` | Sort direction: `asc` or `desc`. |
| `page` | int | 1 | Page number. |
| `per_page` | int | 15 | Number of items per page. |
| `exclude` | string | - | Comma-separated field names to exclude from response. |
| `include` | string | - | Comma-separated relation names to include. |

**Note on `isActive`:** Values `"1"`, `"0"`, `"*"`, or `"all"` can be used.

---

## Date Format

All dates in responses are in `Y-m-d H:i:s` format (e.g. `"2026-05-05 12:30:45"`). Empty fields are returned as `null`.

---

## Included Relations

To include related objects in the same request, use the `include` parameter with a comma-separated list of relation names.

**Example to include student and sheet in an absence record:**

```
GET /api/v1/absence/absences/42?include=student,file
```

If a relation does not exist or an error occurs, it is returned as an empty array `[]` instead of causing the request to fail.

---

## Error Codes and Handling

| Code | Meaning |
|------|---------|
| 200 | Success - data fetched or operation executed. |
| 400 | Bad Request - invalid data, operation disabled by configuration, or insufficient permissions. |
| 401 | Unauthorized - missing or invalid token. |
| 404 | Not Found - requested resource does not exist. |
| 500 | Internal Server Error. |

**Typical Error Body:**

```json
{
  "error": {
    "code": 400,
    "message": "Creating absences is disabled for frontend users.",
    "errors": []
  }
}
```

- `message`: Error message in the specified language (default English).
- `errors`: Additional details about failed fields (if any), especially for create/update operations when validation fails.

---

## Notes for Developers

- **Permission Settings:** Access for each operation (view, create, update) per user type (backend/frontend) is controlled via environment variables. Consult the system administrator for customization.
- **Default Values:** In create operations (`POST`), if you do not send `year_id`, `companys_id`, or `departments_id`, the API will use system default settings, simplifying registration in the current context.
- **Duplicate Check:** You cannot create two absence records for the same student on the same day and same sheet (`files_id`). You will receive an error if you attempt this.
- **`absences_status`:** Accepted values are `present`, `absent`, `permitted`. Ensure they match exactly.
- **Date Format:** When sending `date_at` in `POST` requests, use the format `Y-m-d` (without time).
- **Caching:** Use `activelystats` to determine if your data is up-to-date and whether re-fetching is needed.

This documentation covers all API endpoints available in version 1.0.0 of `Nano.AbsenceApi`. For any questions or expansions, refer to the project's main developer guide.

