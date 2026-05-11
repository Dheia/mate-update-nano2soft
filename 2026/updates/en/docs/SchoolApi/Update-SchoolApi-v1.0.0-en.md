## 2026-05-08 - 2026-05-10

**Addition of `Nano.SchoolApi` – Version 1.0.0**

### Creating a Comprehensive RESTful API for Core School Data Management

---

### Update Summary

The first version **1.0.0** of the `Nano.SchoolApi` plugin introduces a complete API for querying all core entities managed by the `Tss.School` plugin. The plugin is built according to the project's standard `Nano.*Api` pattern, providing protected endpoints to retrieve data for classes (`Classes`), stages (`Stages`), subjects (`Subjects`), groups (`Groups`), group-to-class distribution (`ClassGroups`), subject-to-class distribution (`ClassSubjects`), teacher-to-subject distribution (`TeacherSubjects`), cost center linking to classes (`ClassCostCenters`), fee types (`FeesTypes`), document types (`DocumentTypes`), external schools (`OutSchools`), student documents (`StudentDocuments`), student fees (`StudentsFees`) and last year marks (`LastStudentMarks`). The version supports a multi-level permission system for each resource, professional data transformers, and full translation support.

---

### Release Objectives

- **Provide a unified API** for all vital `Tss.School` entities, opening data for multiple uses.
- **Implement a consistent architectural pattern** with other `Nano.*Api` plugins for ease of maintenance and expansion.
- **Enable external applications and frontends** to access school data (classes, subjects, schedules, fees, documents) easily and securely.
- **Provide fine-grained access control** for each resource, allowing the plugin to be deployed in production environments with varying permissions.
- **Support advanced filtering, search, and sorting** to meet reporting and integration needs.
- **Facilitate building reports and statistics** based on this data without direct database access.

---

### New Features and Improvements

#### 1. Comprehensive RESTful Controllers (14 Controllers)

14 controllers were created covering all core entities of the `Tss.School` plugin, all supporting read operations (`list` and `show`):

| Controller | Targeted Model | Description |
|------------|----------------|-------------|
| `Classes` | `TbClass` | Academic classes (with fees, ranking, stage) |
| `Stages` | `Stage` | Academic stages |
| `Subjects` | `Subject` | Academic subjects |
| `Groups` | `Group` | Academic groups |
| `ClassGroups` | `ClassGroup` | Group-class linking with academic year |
| `ClassSubjects` | `ClassSubject` | Subject distribution to classes (units, grades) |
| `TeacherSubjects` | `TeacherSubject` | Teacher distribution to subjects and groups |
| `ClassCostCenters` | `ClassCostCenter` | Cost center linking to classes |
| `FeesTypes` | `FeesType` | Fee types |
| `DocumentTypes` | `DocumentType` | Required document types |
| `OutSchools` | `OutSchool` | External schools |
| `StudentDocuments` | `StudentDocument` | Student documents (submission status) |
| `StudentsFees` | `StudentsFee` | Student fees (amount, discount) |
| `LastStudentMarks` | `LastStudentMark` | Last academic year marks for students |

**Each controller provides:**

- **`index()`**: Fetch a paginated list with multiple filtering, search, sorting, and pagination.
- **`show($id)`**: View details of a single record with relations.
- **`getRecords(array $options)`**: Core function for building complex queries with support for default settings from `config.php`.
- **`activelystats()`**: Endpoint for monitoring last update (for caching purposes).
- **`getLastUpdateAt()`**: Returns the timestamp of the last update.
- **`validationList($user)`**: Unified function for checking access permissions.

#### 2. Tight Permission System for Each Resource

A separate permission system was applied for each of the fourteen resources, with full control via `config.php` settings and environment variables (`env`). Each resource has:

- `is_allow_list`: Enable/disable listing operation.
- `is_allow_list_backend` / `is_allow_list_frontend`: Allow backend or frontend users.
- `is_check_access_list`: Whether to verify a valid user.
- `is_check_list_permission`: Whether to check for a specific permission (e.g., `tss.school.tbclasses.access`).

In addition to filter settings: `order_by`, `order_dir`, `per_page`, `exclude`.

**Example from `config.php`:**
```php
'classes' => [
    'is_allow_list'            => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_BACKEND', true),
    'is_allow_list_frontend'   => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND', true),
    'is_check_access_list'     => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_ACCESS_LIST', true),
    'is_check_list_permission' => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_LIST_PERMISSION', true),
    'order_by'                 => env('NANO_SCHOOLAPI_CLASSES_ORDER_BY', 'sort_order'),
    'order_dir'                => env('NANO_SCHOOLAPI_CLASSES_ORDER_DIR', 'asc'),
    'per_page'                 => env('NANO_SCHOOLAPI_CLASSES_PER_PAGE', 15),
    'exclude'                  => env('NANO_SCHOOLAPI_CLASSES_EXCLUDE', ''),
],
```

#### 3. Smart Default Values for Resources Dependent on Academic Year

Many resources (such as `ClassGroups`, `TeacherSubjects`, `ClassCostCenters`, `StudentDocuments`, `StudentsFees`) are linked to an academic year (`year_id`). When not provided, the controllers automatically adopt the **default academic year** (`Period::getPrimary()`), ensuring a smooth experience without errors.

**Example from `ClassGroups::getRecords()`:**
```php
if (!$year_id) {
    $primary = Period::getPrimary();
    if ($primary) {
        $year_id = $primary->id;
    }
}
```

#### 4. Comprehensive Data Transformers (14 Transformers)

A data transformer was created for each model, formatting the response and providing included relations with error handling:

- **`ClassTransformer`**: Classes with `company`, `department`, `stage`.
- **`StageTransformer`**: Stages with `company`, `department`.
- **`SubjectTransformer`**: Subjects with `company`, `department`.
- **`GroupTransformer`**: Groups with `company`, `department`.
- **`ClassGroupTransformer`**: Group links with `company`, `department`, `class`, `group`, `study_year`.
- **`ClassSubjectTransformer`**: Subject distribution with `company`, `department`, `class`, `subject`.
- **`TeacherSubjectTransformer`**: Teacher distribution with `company`, `department`, `class`, `group`, `subject`, `teacher`, `study_year`.
- **`ClassCostCenterTransformer`**: Cost center links with `company`, `department`, `class`, `cost_center`, `study_year`.
- **`FeesTypeTransformer`**: Fee types with `company`, `department`.
- **`DocumentTypeTransformer`**: Document types with `company`, `department`.
- **`OutSchoolTransformer`**: External schools with `company`, `department`.
- **`StudentDocumentTransformer`**: Student documents with `company`, `department`, `student`, `documents`, `study_year`.
- **`StudentsFeeTransformer`**: Student fees with `company`, `department`, `student`, `student_record`, `fees_type`, `study_year`.
- **`LastStudentMarkTransformer`**: Last year marks with `company`, `department`, `student`, `class`, `out_school`, `student_record`, `study_year`.

**Transformer Features:**
- Formatting values to correct types (int, string, bool, float, date).
- `formatDate` function for safe date conversion.
- Field filtering (`exclude`) via querystring to control returned data size.
- Adding `object_type` to identify the object type.
- Each relation is wrapped in `try-catch` to avoid failing the entire query.

#### 5. `scopeExclude` Registration for All Models

The dynamic scope `scopeExclude` was registered for all fourteen models inside `Plugin::boot()`, allowing clients to request only specific columns and reduce transmitted data over the network.

```php
foreach ($models as $model) {
    $model::extend(function($model) {
        $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
            $getTable = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
            return $query->select(array_diff($getTable, (array) $columns));
        });
    });
}
```

#### 6. Caching Support

All `index` endpoints support caching via `nano.api::api_enable_cache`, with automatic cache invalidation upon any record update.

#### 7. Comprehensive Translation

The `lang/en/lang.php` file contains all success and error messages and permissions for each resource, making translation to any language easy.

---

### Usage Examples

#### Fetch active classes for a specific stage

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes?stage_id=2&isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### Fetch teacher subject distribution for a specific year with teacher and subject included

```bash
curl -X GET "https://yourdomain.com/api/v1/school/teacher-subjects?year_id=5&include=teacher,subject" \
  -H "Authorization: Bearer <token>"
```

#### Fetch fees for a specific student

```bash
curl -X GET "https://yourdomain.com/api/v1/school/students-fees?student_id=15" \
  -H "Authorization: Bearer <token>"
```

#### Search for a subject by name

```bash
curl -X GET "https://yourdomain.com/api/v1/school/subjects?q=Mathematics" \
  -H "Authorization: Bearer <token>"
```

#### View details of a class with its stage

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes/3?include=stage" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Unprecedented Integration**: For the first time, any external system (mobile app, parent portal, dashboards) can access all school data through a single API.
- **Faster Report Building**: Developers can now create custom reports on classes, subjects, fees, and documents using simple API queries instead of writing complex SQL.
- **Improved User Experience**: Frontends can dynamically fetch dropdown lists (like classes, stages, subjects) from the server.
- **Production-Ready**: The advanced permission system allows precise control over each user's access, making the plugin suitable for real-world deployment.
- **Extensible Design**: `POST` and `PUT` operations can be added for any resource in the future easily without changing the core structure.
- **Excellent Performance**: Using `scopeExclude` with caching ensures fast responses even with thousands of records.

---

### Upgrade Requirements

1. **Install Plugin**: Copy the `nano/schoolapi` folder to `plugins/nano/schoolapi`.
2. **Register Plugin**: Run `php artisan plugin:refresh Nano.SchoolApi`.
3. **Environment Variables**: Adjust `env` variables as needed (e.g., `NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND`).
4. **Permissions**: Ensure roles have the appropriate access permissions in the backend system if `is_check_list_permission` is enabled.
5. **Clear Cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **No database migrations** are required.

---

### Conclusion

Version **1.0.0** of `Nano.SchoolApi` represents a quantum leap in making school data available via API. By covering 14 core entities, the plugin provides a solid foundation for building integrated systems that rely on `Tss.School` data without hassle. The unified design and high extensibility make it an ideal platform for adding more operations and reports in the future.

---

**Reference Documentation**:
- [`Nano.StudyyearApi` Documentation](./docs/StudyyearApi/Docs-StudyyearApi-en.md)
- [`Nano.SchoolApi` Documentation](./docs/SchoolApi/Docs-SchoolApi-en.md)
- [`Nano.StudentsApi` Documentation](./docs/StudentsApi/Docs-StudentsApi-en.md)

