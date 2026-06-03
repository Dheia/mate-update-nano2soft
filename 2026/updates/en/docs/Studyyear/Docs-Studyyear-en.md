# Documentation for the `Tss.Studyyear` Plugin

**Current version:** 1.0.7  
**Developer:** Dheia Ali  
**Dependencies:** RainLab.Translate, RainLab.Location, RainLab.User, Tss.Tools, Tss.BarcodeGenerator, Tss.Basic, Tss.School

---

## Introduction

The `Tss.Studyyear` plugin is the core component for managing **academic years**, **semesters (terms)**, and **academic months** within the integrated NanoSoft system. It provides a flexible and extensible data structure, a fully integrated backend user interface, an advanced permission system, and complete translation support (Arabic/English).

This plugin is the cornerstone of any educational system, as it is relied upon by many other plugins such as `Tss.School` (students and classes), `Tss.Absence` (attendance), `Nano.HomeworkApi` (homework), and others.

---

## Main Features

- **Manage academic years (Periods):**  
  Create multiple academic years with start and end dates, and set a default year.

- **Manage semesters (Semsters):**  
  Each academic year can have two semesters (`semster1`, `semster2`), with their own dates.

- **Manage academic months (Months):**  
  Each semester consists of three academic months (`month_num` 1, 2, 3), with the ability to specify month names (January, February, etc.).

- **Intelligent scopes system:**  
  - `IsCompany()`: Filter by the current company.  
  - `IsActive()`: Show only active records.  
  - `IsCheckDate()`: Check date validity (do not show expired records) – can be disabled via environment variables.  
  - `IsPublished()` / `IsNotPublished()`: Filter published/unpublished records based on `is_published`, `published_at`, and `unpublished_at`.  
  - `scopeIsPublishedResults`: Filter by `is_published_results`.

- **Scheduled publishing support:**  
  Using the fields `is_published`, `published_at`, and `unpublished_at`, you can schedule when records appear on front‑end interfaces and the API.

- **New field `is_published_results`:**  
  Allows separation of general data publishing from results publishing (e.g., making an academic year appear in a results application).

- **Optimised caching:**  
  All dropdown lists and repeated queries are cached to improve performance.

- **Translation support:**  
  All text is translatable, with ready‑to‑use Arabic and English language files.

- **Detailed permissions:**  
  Each entity (years, semesters, months) has separate permissions for access, add, edit, delete, and export.

- **Seeder for initial data:**  
  Transfers data from the old `Tss\School\Models\StudyYear` model to the new tables, automatically creating semesters and months.

---

## Requirements

- Smart Nano Soft App based on **version 3.x** or **2.x** (supported).
- PHP 7.4+.
- The following plugins installed and activated:
  - `RainLab.Translate`
  - `RainLab.Location`
  - `RainLab.User`
  - `Tss.Tools`
  - `Tss.BarcodeGenerator`
  - `Tss.Basic`
  - `Tss.School` (to run the Seeder)

---

## Installation and Upgrade

### Fresh installation

1. Copy the `tss/studyyear` folder to `plugins/tss/studyyear`.
2. Run the command:
   ```bash
   php artisan plugin:refresh Tss.Studyyear
   ```
   The tables will be created and the version will be recorded.

3. (Optional) To populate data from `Tss.School`, run the seeder:
   ```bash
   php artisan tss:seed Tss.Studyyear
   ```

### Upgrade from a previous version (before 1.0.6)

1. Replace all files with the new version.
2. Run:
   ```bash
   php artisan plugin:refresh Tss.Studyyear
   ```
   The `is_published_results` column will be added to the tables, and existing data will not be deleted.
3. Clear the cache:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

## Models

### 1. `Period` – Academic Year

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Year name (e.g., 2025-2026) |
| `calendar_type` | string | Calendar type (`years`) |
| `from_date` | timestamp | Year start date |
| `to_date` | timestamp | Year end date |
| `is_published` | boolean | Publish the year on the site |
| `published_at` | timestamp | Publication start date |
| `unpublished_at` | timestamp | Publication end date |
| `is_published_results` | boolean | Publish the year in report results |
| `is_default` | boolean | Default year (automatically used in the API) |
| `is_active` | boolean | Activate the year |

**Relationships:**
- `hasMany` → `semsters`, `months`, `studentRecords`
- `belongsTo` → `Company`, `Department`

**Core methods:**
- `getPrimary()`: Returns the default year (first by `is_default` and active).
- `getNameList()`: List of active, published, and valid years.
- `clearCache()`: Clear the cache after any modification.

### 2. `Semster` – Semester

| Field | Type | Description |
|-------|------|-------------|
| `ref_type` | string | `semster1` or `semster2` |
| `study_year_id` | integer | Academic year ID (links to `Period`) |
| `from_date`, `to_date` | timestamp | Semester start and end dates (optional) |
| `is_published_results` | boolean | Publish semester results |

**Relationships:**
- `belongsTo` → `Period`
- `hasMany` → `months`

### 3. `Month` – Academic Month

| Field | Type | Description |
|-------|------|-------------|
| `month_num` | integer | Month number within the semester (1, 2, 3) |
| `month_number` | string | Month code (JAN, FEB, …) |
| `semsters_id` | integer | Semester ID |
| `study_year_id` | integer | Academic year ID |
| `from_date`, `to_date` | timestamp | Month start and end dates (automatically calculated on creation) |

**Relationships:**
- `belongsTo` → `Semster`, `Period`

**Note:** `to_date` is automatically calculated by adding 31 days to `from_date` (can be edited manually).

---

## Backend Interface

### Menus

The plugin appears in the control panel under the **Academic Years** main menu and in the settings sidebar.

**Paths:**
- Years: `backend/tss/studyyear/periods`
- Semesters: `backend/tss/studyyear/semsters`
- Months: `backend/tss/studyyear/months`

### Lists (`columns.yaml`)

| Column | Description | Visible by default |
|--------|-------------|---------------------|
| `id` | ID | No |
| `code` | Code | No |
| `name` | Name | Yes |
| `companys_id` | Company | No |
| `departments_id` | Department | No |
| `calendar_type` | Calendar type | Yes |
| `from_date` | From date | Yes |
| `to_date` | To date | Yes |
| `is_published_results` | Publish results | No |
| `is_published` | Published | No |
| `published_at` | Publication date | No |
| `unpublished_at` | Expiry date | No |
| `is_default` | Default | Yes |
| `is_active` | Active | Yes |
| `status` | Status | No |
| `sort_order` | Order | No |

### Forms (`fields.yaml`)

Supports:

- **Tabs**: Basic, Details, Options.
- **Dependent dropdowns (`dependsOn`)**:  
  - Select department → determines academic year → determines semester.
- **Date fields**: `from_date`, `to_date` with automatic calculation of `to_date` when `from_date` changes.
- **Switches**: `is_default`, `is_active`, `is_published`, `is_published_results`.
- **Scheduled date fields**: `published_at`, `unpublished_at` appear only when `is_published` is enabled.

### Filters (`config_filter.yaml`)

| Filter | Type | Description |
|--------|------|-------------|
| `departments_id` | Group (Dropdown) | Filter by department |
| `study_year_id` | Group | Depends on `departments_id` |
| `semsters_id` | Group (for months) | Depends on `study_year_id` |
| `month_num` | Group | Filter by month number |
| `ref_type` | Group (for semesters) | `semster1` / `semster2` |
| `is_active` | Switch | Active / Inactive |
| `is_published` | Checkbox | Shows only published records (uses `scopeIsPublished`) |
| `is_not_published` | Checkbox | Shows only unpublished records |
| `is_published_results` | Switch | 0/1 |
| `is_default` | Switch | Default only |
| `id` | Text | Search by ID |
| `created_at`, `updated_at` | Date range | Filter by creation or update date |

---

## Permissions

Permissions are defined for `periods`, `semsters`, and `months` in `Plugin::registerPermissions()`.

**Example – Academic year permissions:**

| Key | Description |
|-----|-------------|
| `tss.studyyear.periods.access` | Access to the academic years list |
| `tss.studyyear.periods.add` | Add a new academic year |
| `tss.studyyear.periods.edit` | Edit an academic year |
| `tss.studyyear.periods.delete` | Delete an academic year |
| `tss.studyyear.periods.export` | Export academic year data |

The same structure applies to `semsters` and `months`.

> **Note:** Permissions are assigned to the "developer" group by default; an administrator can distribute them to other roles.

---

## Configuration

A `config.php` file exists at `plugins/tss/studyyear/config/config.php` supporting the following environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `TSS_STUDYYEAR_CHECK_DATE_YEAR` | `true` | Enable date validity checking for academic years (do not show expired years) |
| `TSS_STUDYYEAR_CHECK_DATE_SEMSTERS` | `true` | Enable date validity checking for semesters |
| `TSS_STUDYYEAR_CHECK_DATE_MONTHS` | `true` | Enable date validity checking for months |

You can disable checking by setting the variables to `false` in your `.env` file:

```ini
TSS_STUDYYEAR_CHECK_DATE_YEAR=false
```

---

## Database Tables

### `tss_studyyear_periods`
- Basic fields: `id`, `code`, `barcode`, `name`, `short_description`, `description`, `calendar_type`, `from_date`, `to_date`
- Publishing fields: `is_published`, `published_at`, `unpublished_at`, `is_published_results`
- Configuration fields: `is_default`, `is_active`, `status`, `sort_order`
- Relationship fields: `companys_id`, `departments_id`
- Audit fields: `created_by`, `updated_by`, `deleted_by`, `created_at`, `updated_at`, `deleted_at`

### `tss_studyyear_semsters`
- Additional fields: `study_year_id`, `ref_type`
- Similar structure to `periods` with `ref_type` required.

### `tss_studyyear_months`
- Additional fields: `study_year_id`, `semsters_id`, `month_num`, `month_number`
- Similar structure.

All tables use `InnoDB` and support `SoftDelete`.

---

## Seeder Tool

### `SeederSchoolStudyyears`

This seeder transfers data from the `Tss\School\Models\StudyYear` model (the old `Tss.School` plugin) to the `Tss.Studyyear` system.

**Logic:**
- For each academic year existing in `tss_school_studyyears`, a record is created in `tss_studyyear_periods`.
- Then two semesters (`semster1`, `semster2`) are created for each year.
- For each semester, three academic months (`month_num` 1,2,3) are created, with `month_number` automatically set (JAN, FEB, MAR for the first semester, and APR, MAY, JUN for the second).

**Run:**
```bash
php artisan tss:seed Tss.Studyyear
```

> **Note:** Requires the `Tss.School` plugin to be installed and activated; otherwise the seeder will throw an exception.

---

## API Integration – `Nano.StudyyearApi`

The `Nano.StudyyearApi` plugin provides a RESTful API to access academic year, semester, and month data. It depends on the `Tss.Studyyear` models and supports:

- Endpoints: `periods`, `semsters`, `months`.
- Filters: `is_published`, `is_published_results`, `is_default`, `study_year_id`, `semsters_id`, `month_num`, etc.
- Includes: `company`, `department`, `period`, `semster`.
- Caching and pagination.

**Example request to fetch months published for results only:**
```bash
GET /api/v1/studyyear/months?is_published_results=1&study_year_id=5
```

For more details, refer to the [`Nano.StudyyearApi` documentation](./Docs-StudyyearApi-en.md).

---

## Usage Examples

### Create a New Academic Year via the Backend

1. Go to `Academic Years > Academic Year`.
2. Click **Add new**.
3. Enter the name: `2026-2027`.
4. Choose start date: `2026-09-01`, end date: `2027-08-31`.
5. Enable `Publish on site`, and set `published_at` = `2026-09-01`.
6. Enable `Publish results in the application` if you want it to appear in the results app.
7. Save.

### Set a Default Academic Year

- From the years list, open the desired year.
- Enable the `Default` option.
- Save. The default flag will be automatically removed from other years.

### Using the Scopes in Code

```php
// Fetch the default active and published academic year
$defaultPeriod = Period::getPrimary();

// Fetch all active semesters for year 5
$semsters = Semster::IsCompany()->where('study_year_id', 5)->isActive()->get();

// Fetch only published months (respecting dates)
$months = Month::isPublished()->get();
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.1 | – | Plugin initialisation |
| 1.0.2 | – | Create `tss_studyyear_periods` table |
| 1.0.3 | – | Create `tss_studyyear_semsters` table |
| 1.0.4 | – | Create `tss_studyyear_months` table |
| 1.0.5 | – | Seeder to transfer data from `Tss.School` |
| **1.0.6** | 2026-05-30 | Add `is_published_results` column to all tables |
| **1.0.7** | 2026-05-30 | Support for scheduled publishing fields (`is_published`, `published_at`, `unpublished_at`) in the user interface; support for advanced filters; add `is_published_results` to lists, forms, and filters |

---

## Summary

The `Tss.Studyyear` plugin is the backbone for managing the temporal structure of the academic year in the NanoSoft system. Thanks to its flexible design, powerful scopes system, scheduled publishing support, and separation of results publishing, any educational system can rely on it to build precise and extensible academic calendars.

The plugin also provides seamless integration with the API (`Nano.StudyyearApi`), allowing external applications to securely and quickly benefit from academic year, semester, and month data.

---

**References:**
- [`Nano.StudyyearApi` documentation](./Docs-StudyyearApi-en.md)