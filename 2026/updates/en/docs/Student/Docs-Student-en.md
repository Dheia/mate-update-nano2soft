# 📘 `Tss.Student` Plugin Documentation

**Version:** 1.0.13  
**Path:** `plugins/tss/student`  
**Purpose:** Managing student and parent data and their academic records in the NanoSoft App system, with full support for financial operations (fees) and grade promotion, integrated with the user system (`RainLab.User`) and the financial accounting system (`Tss.Accounts`).

---

## 1. Introduction

The `Tss.Student` plugin is the core component for managing students and parents in the Nano ecosystem. It provides integrated models for students (`Student`), parents (`Mparent`), and annual student records (`StudentRecord`) with fine‑grained permissions, translation support, a complete backend interface, and the ability to link (Frontend/Backend) users to students and parents.

The plugin is built according to Nano Soft App best practices, uses advanced `Traits` such as `SoftDelete`, `Sortable`, and `Purgeable`, and integrates with other plugins like `Tss.Basic`, `Tss.Accounts`, `Tss.School`, and `Nano.AuthApi` to provide a unified experience.

---

## 2. Plugin Features

| Feature | Description |
|---------|-------------|
| **Comprehensive student management** | Stores personal data, contact information, identity, address, images, and files. |
| **Parent management** | Links a student to one parent (or more with custom relationships). |
| **Student academic records** | Enrols a student in an academic year, class, and group, tracking fees, discounts, and record status (continuing, successful, failed, transferred, completed). |
| **Financial operations support** | Calculates tuition fees, registration fees, bus fees, discounts, and automatically creates accounting entries via `Tss.Accounts`. |
| **User linking** | Links any user model (Backend/Frontend) to a student or parent via `user_id` and `user_type` (polymorphic). |
| **Multi‑language support** | Translates basic text fields (`full_name`, `description`, `address`) using `RainLab.Translate`. |
| **Fine‑grained permissions** | Separate permissions for students and parents (view, add, edit, delete, search, export, print, SMS, email). |
| **Integrated backend** | Lists, forms, import/export, relationships, and bulk promotion operations for students. |
| **Integration with the accounting system** | Automatically creates financial accounts for students and parents, and accounting entries for registration fees. |
| **API support** | All data can be accessed via the separate `Nano.StudentsApi` plugin. |

---

## 3. Database Structure

### 3.1 Table `tss_school_students` (students)

| Field | Type | Description |
|-------|------|-------------|
| `id` | int | Primary key. |
| `code`, `barcode`, `manual_code` | string | Identification codes. |
| `full_name`, `first_name`, `last_name` | string | Student names. |
| `gender` | string | Gender (`male`/`female`). |
| `date_of_birth` | timestamp | Date of birth. |
| `place_of_birth` | string | Place of birth. |
| `religion_id` | string | Religion. |
| `nationality`, `language` | string | Nationality and language. |
| `passport_type`, `passport_number`, `passport_issuer`, `passport_release_at`, `passport_expiry_at` | string / timestamp | Passport data. |
| `phone`, `email`, `website`, `mobile` | text / string | Contact information (JSON for phones and emails). |
| `address`, `address_1`, `address_2`, `street_name`, `street_number` | string | Address details. |
| `latitude`, `longitude`, `radius`, `geo_components`, `city`, `zip`, `postcode`, `country_long`, `formataddress`, `vicinity` | string / double | Geographic location data. |
| `country_id`, `state_id`, `directorate_id` | string | IDs for country, state, and directorate (linkable to `RainLab.Location`). |
| `parents_id` | string | Parent ID (links to `tss_school_parents`). |
| `parent_type` | string | Relationship (father, mother, brother, etc.). |
| `year_id` | string | Academic year (links to `Tss.Studyyear`). |
| `last_school` | string | Previous school. |
| `student_status` | string | Student status (continuing, transferred, completed). |
| `regster_date` | timestamp | Registration date. |
| `is_active`, `is_effective`, `is_hidden`, `is_default` | boolean | Student status flags. |
| `accounts_id`, `bank_acc_id`, `currencys_id`, `credit_limit`, `default_discount` | string / decimal | Financial account data. |
| `user_id`, `user_type` | bigInteger / string | User link (polymorphic). |
| `properties`, `config_data`, `other_data` | text | Additional JSON data. |
| `created_by`, `updated_by`, `deleted_by` | string | IDs of users who performed the operation. |
| `created_at`, `updated_at`, `deleted_at` | timestamp | Creation, update, and soft‑delete timestamps. |

> Many fields have been added via migration files such as `builder_table_add_address_columns_to_students.php`, `builder_table_add_passport_columns.php`, etc.

### 3.2 Table `tss_school_parents` (parents)

Structure similar to the students table but with fewer columns, focusing on `full_name`, `phone`, `email`, `mobile`, `address`, `user_id`, `user_type`, `accounts_id`.  
The main relationship is `hasMany` with students via `parents_id`.

### 3.3 Table `tss_school_student_record` (student academic records)

| Field | Type | Description |
|-------|------|-------------|
| `id` | int | Primary key. |
| `student_id` | string | Student ID. |
| `year_id` | string | Academic year. |
| `class_id` | string | Class. |
| `group_id` | string | Group. |
| `semster` | string | Semester (`semster1`/`semster2`). |
| `status_class` | string | Record status (`continue`, `successfull`, `fail`, `complete`, `transfer`, `unconfirmation`). |
| `status_mony` | string | Payment status. |
| `study_fees`, `registration_fees`, `bus_fees`, `discount` | decimal | Tuition fees, registration fees, bus fees, discount. |
| `is_discount`, `is_registration_fees`, `is_bus` | boolean | Flags to enable discount, registration fees, or bus fees. |
| `student_acc`, `discount_acc`, `study_fees_acc`, `registration_fees_acc`, `bus_fees_acc` | string | Accounting account numbers used in the journal entry. |
| `*_details_id` | text | IDs of accounting entry details (format `bond_id:detail_id`). |
| `currencys_id`, `rate`, `periods_id`, `cost_centers_id` | string / decimal | Currency, exchange rate, financial period, and cost centre data. |
| `regster_date` | timestamp | Registration date. |
| `notes`, `properties`, `other_data` | text | Notes and additional data. |
| `country_id`, `state_id`, `directorate_id` | string | Record location (optional). |
| `is_active`, `is_default` | boolean | Record status. |

---

## 4. Models

### 4.1 `Tss\Student\Models\Student`

**Table:** `tss_school_students`  
**Traits used:** `Validation`, `SoftDelete`, `Sortable`, `Purgeable`, `YearTrait`.  
**Behaviors:** `TranslatableModel`, `LocationModel`, `BasicModel`, `BasicScope`.

**Main relationships:**

```php
public $belongsTo = [
    'Department' => [Department::class, 'key' => 'departments_id'],
    'Parent' => [Mparent::class, 'key' => 'parents_id'],
    'MParents' => [Mparent::class, 'key' => 'parents_id'],
    'StudyYear' => [Period::class, 'key' => 'year_id'],
    'Account' => [Account::class, 'key' => 'accounts_id'],
    'Created_by' => [BackendUser::class, 'key' => 'created_by'],
    // ...
];

public $hasMany = [
    'StudentRecord' => [StudentRecord::class, 'key' => 'student_id'],
    'StudentNote' => [StudentNote::class, 'key' => 'student_id'],
];
```

**Main events:**

- `beforeCreate` / `beforeUpdate`: sets default values (company, department), normalises the name, prevents duplicates (`checkMatchFullName`), processes the mobile number (`preperSaveMobile`), and checks for duplicate linked user (`preperSaveUser`).
- `afterCreate` / `afterUpdate`: creates a financial account (if `is_auto_accounts` is enabled) and creates a user (if `is_auto_users` is enabled).
- `afterSave`: sets a default code (`setCodeDefault`).

**Scopes:**

- `scopeByUser`, `scopeWhereByUser`, `scopeWhereNotNullUser`, `scopeWhereNullUser`, `scopeWhereHasUserTrashed`.
- `scopeIsContinue` (student status `continue`).

### 4.2 `Tss\Student\Models\Mparent`

Similar to `Student` but with simplified relationships.

**Relationships:**

```php
public $hasMany = [
    'Student' => [Student::class, 'key' => 'parents_id'],
    'Student_count' => [Student::class, 'key' => 'parents_id', 'count' => true],
];
```

**Events:** Same logic as `Student` (name validation, mobile number, user).

### 4.3 `Tss\School\Models\StudentRecord`

**Table:** `tss_school_student_record`  
**Traits:** `Validation`, `SoftDelete`, `Purgeable`, `Sortable`, `YearTrait`.

**Relationships:**

```php
public $belongsTo = [
    'Student' => [Student::class, 'key' => 'student_id'],
    'Department' => [Department::class, 'key' => 'departments_id'],
    'RClass' => [TbClass::class, 'key' => 'class_id'],
    'Group' => [Group::class, 'key' => 'group_id'],
    'StudyYear' => [Period::class, 'key' => 'year_id'],
    'CostCenter' => [CostCenter::class, 'key' => 'cost_centers_id'],
    // ...
];
```

**Important events:**

- `beforeCreate` / `beforeUpdate`: checks for duplicate records with `continue` status for the same student (`checkDoubleContinue`) and prevents double enrolment in the same class with success or continue status (`checkDoubleClass`).
- `afterCreate`: calls `createFees()` to create the accounting entries for fees.
- `createFees`: creates a separate accounting entry for each of: tuition fees, discount, registration fees, bus fees (if enabled). Uses `AccountHelper` to create the bonds and links them to the `*_details_id` fields.

**Scopes:**

- `scopeIsContinue`, `scopeIsActiveContinue`.

---

## 5. Helpers

### 5.1 `Tss\Student\Helpers\StudentHelper`

The main interface for developers. Contains:

- **`getRecords(array $options, bool $isException = false): array`** – fetch students (calls `StudentRecordsHelper`).
- **`getParentRecords(array $options, bool $isException = false): array`** – fetch parents with special filters.
- **`getStudentRecordRecords(array $options, bool $isException = false): array`** – fetch student records.
- Functions to link users to students / parents.
- `Mparent`‑specific functions: `getStudentIdsByMparent`, `getStudentsByMparent`, `getStudentsByUser`.
- `StudentRecord`‑specific functions: `getStudentIdFromRecord`, `getCurrentStudentRecord`.
- Legacy functions (for compatibility): `getStudentRecordV1`, `getResultRecord`, `setUpStudentRecord`, etc.

> This class is fully documented in `Docs-StudentHelper-Class-en.md`.

### 5.2 `Tss\Student\Helpers\StudentRecordsHelper`

Contains the core logic for fetching records with advanced filtering, caching, events, and `AccessManager` integration. Used internally by `StudentHelper`.  
> Documented in `Docs-StudentRecordsHelper-Class-en.md`.

---

## 6. Backend Controllers

### 6.1 `Tss\Student\Controllers\Students`

- Uses `ListController`, `FormController`, `ImportExportController`, `RelationController`.
- Supports import/export of student data (via `ImportExportController`).
- Overrides `create_onSave` to call `autoCreateAccounts` and `autoCreateUsers` after saving a student.
- Implements `listExtendQuery` to apply access permissions (`access_all` vs `access`).

### 6.2 `Tss\Student\Controllers\Parents`

Similar to `Students` but without import/export. Uses `ListController`, `FormController`, `ReorderController`.

### 6.3 `Tss\Student\Controllers\StudentRecordUps`

A specialised controller for promoting students (moving them from one class to another or changing their status). Contains AJAX methods:

- `onRecordCount`: counts the affected records.
- `onUpStudentRecord`: executes the promotion using `StudentHelper::setUpStudentRecord`.

---

## 7. Configuration

The `config.php` file in `plugins/tss/student/config.php` contains settings for `students`, `parents`, and `student_record_ups`.

**Example for student settings:**

```php
'students' => [
    'department_type' => env('TSS_STUDENT_STUDENTS_DEPARTMENT_TYPE', 'main'),
    'is_allow_auto_accounts' => env('TSS_STUDENT_STUDENTS_IS_ALLOW_AUTO_ACCOUNTS', true),
    'is_allow_auto_users' => env('TSS_STUDENT_STUDENTS_IS_ALLOW_AUTO_USERS', true),
    'match_full_name' => env('TSS_STUDENT_STUDENTS_MATCH_FULL_NAME', true),
    'sensitive_full_name' => env('TSS_STUDENT_STUDENTS_SENSITIVE_FULL_NAME', true),
    // ...
],
'parents' => [ ... ],
'student_record_ups' => [
    'process_type' => [
        'update_record' => true,
        'update_create_record' => false,
        'update_create_accuonts_record' => false,
    ],
],
```

These values can be overridden via environment variables (`.env`) or directly in the file.

---

## 8. Permissions

Permissions are defined in `Plugin::registerPermissions()`:

| Permission | Description |
|------------|-------------|
| `tss.student.students.access` | Access to the student list (restricted by `created_by` if `access_all` is not held). |
| `tss.student.students.access_all` | Access to all student records. |
| `tss.student.students.add`, `edit`, `delete`, `search`, `export`, `print`, `email`, `sms` | Individual operation permissions. |
| `tss.student.parents.*` | Same for parents. |
| `tss.student.studentrecordups.manage` | Manage student promotions. |

These permissions are used in the backend controllers (`Students`, `Parents`, `StudentRecordUps`) to control button visibility and allow operations.

---

## 9. Events

The plugin fires the following events (can be used by other plugins):

- `api.list.extendQueryBefore`, `api.list.extendQuery` – used in `getRecords` methods to extend the query.
- `StudentsApi.ApiControllers.Students.List` – after fetching the student list (in the API).
- `ParentsApi.ApiControllers.Parents.List` – for the parent list.
- `StudentRecordsApi.ApiControllers.StudentRecords.List` – for the student record list.
- Inside the `StudentRecord` model: `afterCreate` does not explicitly fire an event, but you can call `Event::fire` in `createFees` if desired.

---

## 10. Compatibility and Dependencies

- **Requires the following plugins** (listed in `$require`):
  - `Rainlab.Translate`
  - `Rainlab.Location`
  - `Rainlab.User`
  - `Tss.Tools`
  - `Tss.BarcodeGenerator`
  - `Tss.Basic`
  - `Tss.Accounts`
  - `Tss.School`
  - `Tss.Studyyear`
- **To get full API functionality**, you need to install `Nano.StudentsApi`.
- **To benefit from advanced permission functions** (such as `AccessManager`), it is recommended to install `Nano.AuthApi`.

---

## 11. Conclusion

The `Tss.Student` plugin is the foundation for managing students and parents in any system based on Nano Soft App technology. Thanks to its modular design, flexible permissions, and integration with accounting and user systems, it can be easily used in school, university, and training centre projects.

To get the most out of the plugin, we recommend reading the following documentation:

- [`StudentHelper` class documentation](./Docs-StudentHelper-Class-en.md)
- [`StudentRecordsHelper` class documentation](./Docs-StudentRecordsHelper-Class-en.md)
- [`Nano.StudentsApi` plugin documentation (API)](../StudentsApi/Docs-StudentsApi-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)

---
