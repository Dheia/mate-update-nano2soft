## 2026-05-27 – 2026-05-30

**`Nano.HomeworkApi` Plugin – Version 1.0.0**

### Creating a RESTful API for Managing Homeworks, Submissions, and Classifications

---

### Summary of Updates

The first version **1.0.0** of the `Nano.HomeworkApi` plugin provides a complete RESTful API to interact with the `Tss.Homework` system, including:
- **Homeworks and activities** (`Homeworks`) – create, update, delete, and view, with support for general assignments for whole classes (`student_id = '*'`)
- **Submissions and evaluations** (`Feedbacks`) – record student grades and notes
- **Homework classifications** (`Categories`) – tree‑structured categories
- **General classification types** (`ClassTypes`) – reference types
- **Company‑specific classification types** (`ClassTypeCompanies`) – linked to company and department

The plugin was built according to the latest `Nano-Api-SKILL.md` standards, making full use of `AccessManager` for permissions, `AdvancedQueryHelper` for dynamic filtering, data transformers, and a unified caching and event system.

---

### Release Objectives

- **Provide a unified API** to manage all homework entities.
- **Enable external applications** (student apps, parent portals, teacher dashboards) to interact with homeworks and submissions.
- **Support advanced scenarios** such as general assignments for whole classes (`student_id = '*'`) or whole groups (`group_id = '*'`).
- **Implement a flexible permission system** that controls read and write operations for each user type (backend/frontend/guest).
- **Provide advanced filters** (`is_or`, `is_not`, `is_force`, `is_or_null`) for complex queries.
- **Facilitate future extensibility** thanks to events and a unified code structure.

---

### New Features and Improvements

#### 1. Complete RESTful Controllers

Five controllers were created covering all required entities:

| Controller | Target Model | Supported Operations |
|------------|--------------|----------------------|
| `Homeworks` | `Tss\Homework\Models\Homework` | list, show, create, update, delete |
| `Feedbacks` | `Tss\Homework\Models\Feedback` | list, show, create, update |
| `Categories` | `Tss\Homework\Models\Categorie` | list, show |
| `ClassTypes` | `Tss\Homework\Models\ClassType` | list, show |
| `ClassTypeCompanies` | `Tss\Homework\Models\ClassTypeCompany` | list, show |

**Each controller supports:**
- **`index()`**: fetch a list with filtering, search, ordering, and pagination.
- **`show($id)`**: view details of an item, including requested relationships.
- **`getRecords(array $options)`**: core function for building complex queries (used internally).
- **`activelystats()`**: an endpoint to return the last update timestamp (for caching).
- **`store()`, `update()`, `destroy()`** as needed.

#### 2. Support for General Homeworks for Whole Classes or Whole Groups

Special logic was implemented in `Homeworks::getRecords()`:
- If a specific `student_id` (not `'*'` and not `null`) is passed, records where `student_id` equals that value **or** `'*'` **or** `null` are included.  
  → This means that an assignment created with `student_id = '*'` appears for all students when querying for a specific student.
- The same logic applies to `group_id` and `record_id`.

**Example:**
```php
// When creating an assignment for all students in a class
$data = ['student_id' => '*', ...];

// When fetching assignments for student 15
GET /api/v1/homework/homeworks?student_id=15
// This will return the assignment specific to student 15 + the general assignment (*)
```

#### 3. Centralised, Multi‑Level Permission System

All permission settings are defined in `config.php` for each operation and each resource, with the ability to control them via environment variables.

**Example from `Homeworks.list` settings:**
```php
'homeworks' => [
    'list' => [
        'permission' => [
            'is_allow' => env('NANO_HOMEWORKAPI_HOMEWORKS_LIST_IS_ALLOW', true),
            'backend' => [
                'allow' => true,
                'check_permission' => true,
                'permissions' => ['tss.homework.homeworks.access_all', 'tss.homework.homeworks.access'],
                'access_scope' => 'all',
            ],
            'frontend' => [
                'allow' => true,
                'allowed_ref_types' => ['student', 'parent'],
                'access_scope' => 'own',
            ],
        ],
    ],
],
```

Permissions are checked via `AccessManager`:
```php
$access = AccessManager::checkByResource('nano.homeworkapi::homeworks.list', $user);
if (!$access['allowed']) return $this->errorUnauthorized($access['message']);
```

#### 4. Advanced Filters with `advanced_filters`

An `advanced_filters` array was added to:
- Specify which fields are allowed to use `is_or`, `is_not`, `is_or_null`.
- Define special rules per user type (backend/frontend) and even per `ref_type` (student/parent).

**Example limiting `student_id` for ordinary students while allowing it for parents:**
```php
'advanced_filters' => [
    'field_specific_rules' => [
        'student_id' => [
            'frontend' => [
                '*' => false,
                'parent' => true,
            ],
        ],
    ],
],
```

#### 5. Dynamic Resolver for `student_id` and `record_id` (`frontend_resolver`)

A `frontend_resolver` section was added that automatically:
- If the user is a **student** (`ref_type = student`): populates `student_id` from the current user and fetches the current `record_id` for the student.
- If the user is a **parent** (`ref_type = parent`): checks whether `student_id` or `record_id` is provided in the request, verifies student ownership for the parent, and fills missing values.

**Implementation in the controller:**
```php
$options = AccessManager::instance()->resolveDynamicFrontendOptions(
    $user, $options, null, 'nano.homeworkapi::homeworks.list'
);
```

#### 6. Unified `getRecords` Function with Rich Features

Each controller has an identical `getRecords` function that supports:
- **Advanced filters** via `AdvancedQueryManager::scopeWhereField` with `is_or`, `is_not`, `is_force`, `is_or_null`.
- **Access scope** (`applyAccessScope`) for company, department, state, and record creator.
- **Excluding columns** (`exclude`) to reduce data size.
- **Including relationships** (`custom_with`).
- **Text search** (`q`) with advanced search support (ArPhpHelper).
- **Ordering** (`orderBy`, `orderDirection`) and grouping (`group_by`) with `having`.
- **Output options**: `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`.
- **Events**: `api.list.extendQueryBefore` and `api.list.extendQuery` to extend queries.
- **Caching** via `$this->cached()`.

#### 7. Data Transformers

Five transformers were created:
- **`HomeworkTransformer`**: displays assignment data with relationships (company, department, category, type_class, student, record, subject, teacher).
- **`FeedbackTransformer`**: displays submission data with relationships (homework, student, record).
- **`CategoryTransformer`**: displays category data with relationships (company, department, type_class, parent).
- **`ClassTypeTransformer`**: displays general classification types.
- **`ClassTypeCompanyTransformer`**: displays company classifications with relationships (company, department, class_type).

All transformers support:
- Date formatting.
- Field exclusion (`exclude`).
- Error handling in relationships via `try-catch` (return an empty array instead of failing the request).

#### 8. Unified Response Structure for Create and Update Operations

The `store` and `update` operations follow a unified response structure:
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
    "input_data": { ... },
    "process_data": { ... },
    "debug": { ... }  // only in development mode
  }
}
```

#### 9. Intelligent Default Values

In create operations, if `companys_id`, `departments_id`, or `year_id` are not passed, they are automatically filled from system default values:
- `companys_id` ← `BasicHelper::getCompanysId(true)`
- `departments_id` ← `BasicHelper::getMainDepartmentId(true)`
- `year_id` ← `Period::getPrimary()->id`

#### 10. Registering the `scopeExclude` Scope

The dynamic scope `scopeExclude` was registered for all five models in `Plugin::boot()`:
```php
\Tss\Homework\Models\Homework::extend(function($model) {
    $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
        $tableColumns = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
        return $query->select(array_diff($tableColumns, (array) $columns));
    });
});
// Similarly for Feedback, Categorie, ClassType, ClassTypeCompany
```

#### 11. Full Translation Support

All success messages, error messages, and permissions are present in the translation files (`lang/en/lang.php` and `lang/ar/lang.php`) and are ready for localisation.

---

### Supported Endpoints

| Resource | Supported Methods | Paths |
|----------|------------------|-------|
| **Homeworks** | GET, POST, PUT, DELETE | `/homeworks`, `/homeworks/{id}`, `/homeworks/activelystats` |
| **Feedbacks** | GET, POST, PUT | `/feedbacks`, `/feedbacks/{id}`, `/feedbacks/activelystats` |
| **Categories** | GET | `/categories`, `/categories/{id}`, `/categories/activelystats` |
| **ClassTypes** | GET | `/class-types`, `/class-types/{id}`, `/class-types/activelystats` |
| **ClassTypeCompanies** | GET | `/class-type-companies`, `/class-type-companies/{id}`, `/class-type-companies/activelystats` |

**Base path:** `/api/v1/homework`

---

### Usage Examples

#### 1. Fetch a List of Homeworks for a Specific Student, Including Subject and Teacher

```bash
curl -X GET "https://yourdomain.com/api/v1/homework/homeworks?student_id=15&include=subject,teacher" \
  -H "Authorization: Bearer <token>"
```

#### 2. Create a New Homework for All Students in a Class (`student_id = '*'`)

```bash
curl -X POST "https://yourdomain.com/api/v1/homework/homeworks" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Weekly Math Homework",
    "departments_id": "1",
    "ref_type_class": "homework",
    "year_id": "5",
    "semster": "semster2",
    "class_id": "3",
    "subject_id": "21",
    "student_id": "*",
    "min_degree": 5,
    "max_degree": 10,
    "description": "Solve exercises pages 10-15"
  }'
```

#### 3. Update a Homework’s Maximum Grade

```bash
curl -X PUT "https://yourdomain.com/api/v1/homework/homeworks/42" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"max_degree": 12}'
```

#### 4. Delete a Homework

```bash
curl -X DELETE "https://yourdomain.com/api/v1/homework/homeworks/42" \
  -H "Authorization: Bearer <token>"
```

#### 5. Submit a Homework (Create a Feedback) by a Student

```bash
curl -X POST "https://yourdomain.com/api/v1/homework/feedbacks" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "homeworks_id": "42",
    "student_id": "15",
    "record_id": "8",
    "degree": 9.5,
    "is_recive": true,
    "note": "Submitted successfully"
  }'
```

#### 6. Fetch Top‑Level Categories (Parent = null)

```bash
curl -X GET "https://yourdomain.com/api/v1/homework/categories?parent_id=null" \
  -H "Authorization: Bearer <token>"
```

#### 7. Use `is_or` to Search for Multiple Classes

```bash
curl -X GET "https://yourdomain.com/api/v1/homework/homeworks?class_id=3,5&is_or_class_id=true" \
  -H "Authorization: Bearer <token>"
```

---

### Benefits and Added Value

- **Decentralised homework management**: Teachers can record assignments via the API without accessing the backend.
- **Integration with student applications**: Students can view their assignments and submit their work directly.
- **Parent role**: Parents can monitor their children’s assignments and confirm receipt.
- **High flexibility**: Thanks to advanced filters and general assignments, intelligent task distribution systems can be built.
- **Excellent performance**: Excluding unnecessary columns and caching ensures fast responses.
- **Extensibility**: Thanks to events, other plugins can modify the API’s behaviour without altering the core code.

---

### Installation and Upgrade Requirements

1. **Install the plugin**: Copy the `nano/homeworkapi` folder to `plugins/nano/homeworkapi`.
2. **Register the plugin**:
   ```bash
   php artisan plugin:refresh Nano.HomeworkApi
   ```
3. **Environment settings**: Add the required environment variables to your `.env` file (see `config/config.php` for all options).
   ```env
   NANO_HOMEWORKAPI_HOMEWORKS_LIST_IS_ALLOW=true
   NANO_HOMEWORKAPI_HOMEWORKS_CREATE_FRONTEND_ALLOW=false
   # ... etc.
   ```
4. **Permissions**: Ensure the appropriate roles have permissions such as `tss.homework.homeworks.add`, `tss.homework.homeworks.edit`, etc.
5. **Clear cache**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **No database migrations** are required (the plugin depends on the existing tables of `Tss.Homework`).

---

### Conclusion

Version **1.0.0** of `Nano.HomeworkApi` is a complete and powerful addition for managing homeworks and submissions via an API. By following the latest Nano API standards, the plugin provides high security, flexible filtering, and easy extensibility. All endpoints are ready to be used by web applications, mobile apps, and any external system that needs to interact with the homework system.

---

**Reference documentation**:
- [`Nano.HomeworkApi` documentation](./docs/HomeworkApi/Docs-HomeworkApi-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)
- [`AccessManager` class documentation](./docs/AuthApi/Docs-AccessManager-en.md)
- [`AdvancedQueryHelper` class documentation](./docs/querybuilder/Docs-AdvancedQueryHelper.md)
