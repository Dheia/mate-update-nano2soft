# 📘 AccessManager Permission Management Guide

**Version:** 1.0.20+  
**Path:** `Nano\AuthApi\Classes\AccessManager`  
**Purpose:** Complete documentation of the permission control mechanism, access scopes, advanced filters, and operation settings in the Nano system.

---

## 1. Introduction

`AccessManager` provides a centralised and unified system for managing permissions and access to API resources based on:

- **User type**: `backend` (control panel users), `frontend` (regular front‑end users), `guest` (unauthenticated visitors).
- **Custom permissions**: each operation (`list`, `create`, `update`, `delete`, `show`, …) can have independent settings.
- **Access scopes**: `all`, `own`, `created_by`, `company`, `department`, `state`, `children`.
- **Advanced filters**: `is_or`, `is_not`, `is_or_null` with fine‑grained control per field and user type.
- **Integration with `config.php`**: all settings are defined in the configuration file of each API plugin and can be overridden via environment variables.

This guide explains **how to configure and use the permission system** through `AccessManager` and the `config.php` file.

---

## 2. Structure of Operation Settings in `config.php`

For each operation (e.g., `list`, `create`, `update`, `show`, `delete`), create a key inside the resource, and inside it a `permission` key. Example for the `list` operation of the `absences` resource:

```php
// config.php
return [
    'absences' => [
        'list' => [
            'permission' => [
                'is_allow' => true,                     // enable the operation (global)

                'backend'  => [ /* ... */ ],           // backend user settings
                'frontend' => [ /* ... */ ],           // frontend user settings
                'guest'    => [ /* ... */ ],           // guest (unauthenticated) settings

                'advanced_filters' => [ /* ... */ ],   // advanced filters settings
            ],
        ],
        'create' => [ /* ... */ ],
        'update' => [ /* ... */ ],
        'show'   => [ /* ... */ ],
    ],
];
```

> **Note:** Different operations can have different settings. If settings for a particular operation are missing, you can use the fallback mechanism as described later.

---

## 3. Configuration Options for Each User Type

### 3.1 `backend` Settings

```php
'backend' => [
    'allow'              => true,                 // whether backend users are allowed
    'check_permission'   => true,                 // whether to check permissions (hasAccess/hasPermission)
    'permissions'        => ['tss.example.access'], // single permission or array of permissions
    'access_scope'       => 'all',                // access scope (see section 4)
    'company_field'      => 'companys_id',        // company field in the table
    'department_field'   => 'departments_id',     // department field
    'state_field'        => 'state_id',           // state field
    'created_by_field'   => 'created_by',         // record creator field
    'company_id'         => null,                 // restrict to a specific company (optional)
    'department_id'      => null,                 // restrict to a specific department (optional)
    'state_id'           => null,                 // restrict to a specific state (optional)
],
```

#### 3.1.1 Supported Access Scopes for `backend`

| Scope (`access_scope`) | Description | Notes |
|------------------------|-------------|-------|
| `all` | Show all records (no additional filtering) | Usually requires the `access_all` permission |
| `own` | Show only records created by the user (`created_by`) | Automatically converted to `created_by` |
| `created_by` | Same as `own` | The field name can be set via `created_by_field` |
| `company` | Show records of a specific company (defined by `company_id`) | If the user can access all companies, the scope becomes `all` |
| `department` | Show records of a specific department | If the user can access all departments or all companies, becomes `all` |
| `state` | Show records of a specific state | If the user can access all states or all departments, becomes `all` |

**Practical example:**  
A backend user has only the permission `tss.example.access` and does not have `access_all`. If `access_scope = 'company'` is set, records will be filtered so that `companys_id = the user’s company id`. If the user has the `access_all` permission, the filter is ignored and all records are shown.

### 3.2 `frontend` Settings

```php
'frontend' => [
    'allow'              => true,                 // whether frontend users are allowed
    'allowed_ref_types'  => ['student', 'parent'], // allowed ref_type values (full or short)
    'access_scope'       => 'own',                // 'own' or 'children'
    'children_relation'  => 'students',           // relationship name on the parent model (for 'children')
],
```

#### 3.2.1 Supported Access Scopes for `frontend`

| Scope (`access_scope`) | Description | Requirements |
|------------------------|-------------|--------------|
| `own` | Show the user’s own records (based on `student_id` or `user_id`) | Requires that the user is linked to a student (ref_type = student) or has `student_id` in the options |
| `children` | Show records of the user’s children (when the user is a parent) | Requires `children_relation` to be defined on the associated model and `resolveFrontendAccessOptions` to be enabled |

**Practical example:**  
A parent (ref_type = parent) with `access_scope = 'children'`. When `resolveFrontendAccessOptions` is called, their children are fetched via the `students` relationship, and then a `student_id IN (...)` filter is applied.

### 3.3 `guest` Settings

```php
'guest' => [
    'allow'         => true,            // allow unauthenticated visitors
    'access_scope'  => 'all',           // 'all' (all records) or 'none' (nothing)
],
```

> **Warning:** When enabling `guest.allow = true` for any operation, be careful about data sensitivity. This is often used for public `list` operations.

---

## 4. Detailed Access Scopes

### 4.1 `all`
No additional `where` condition is added. Shows all records that match the other filters.

### 4.2 `own` / `created_by`
Adds a condition `where($created_by_field, $user->id)`.  
**Typically used with the `access` permission (without `access_all`).**

### 4.3 `company`
Adds a condition `where($company_field, $companyId)`.  
**Used in multi‑tenant (multi‑company) applications.**

### 4.4 `department`
Adds a condition `where($department_field, $departmentId)`.

### 4.5 `state`
Adds a condition `where($state_field, $stateId)` or `whereIn($state_field, $stateIds)`.

### 4.6 `children` (frontend‑only)
Adds a condition `whereIn($studentIdField, $childrenIds)` after fetching the user’s children from the specified relationship.

---

## 5. Advanced Filters Control (`advanced_filters`)

Advanced filters allow passing options like `is_or`, `is_not`, `is_or_null` on fields. You can control which fields are allowed for each user type.

### 5.1 Settings Structure

```php
'advanced_filters' => [
    'enabled'                => true,                     // enable/disable globally
    'default_for_backend'    => true,                     // allow by default for backend
    'default_for_frontend'   => false,                    // deny by default for frontend
    'allowed_fields_backend' => ['*'],                    // all fields allowed for backend
    'allowed_fields_frontend' => [],                      // nothing for frontend
    'field_specific_rules'   => [                         // special rules per field
        'student_id' => [
            'backend' => true,
            'frontend' => [
                '*' => false,                             // general for all frontend
                'parent' => true,                         // allow for parent
            ],
        ],
        'record_id' => [ /* ... */ ],
    ],
],
```

### 5.2 How It Works

- If `enabled = false`, all `is_or_*`, `is_not_*`, `is_or_null_*` keys are removed from the options.
- Each field is checked individually:  
  1. If a special rule exists for the field (`field_specific_rules`), it is used.  
  2. Otherwise, `default_for_backend` / `default_for_frontend` along with the allowed fields list is used.
- For `frontend`, you can customise allowance per `ref_type` using an array `'frontend' => [ 'ref_type' => true/false, '*' => true/false ]`.

**Example:** Allows a parent to use `is_or_student_id` but does not allow a student to use it.

---

## 6. Fallback Mechanism for Undefined Operations

If settings for a particular operation are missing (e.g., `show`), you can use the settings of another operation (e.g., `list`) as a base and merge any partial settings that exist.

**`checkWithFallback` function in `AccessManager`:**

```php
AccessManager::checkWithFallback(
    'nano.absenceapi::absences.show',   // primary resource
    'nano.absenceapi::absences.list',   // fallback resource
    $user
);
```

- If `show.permission.is_allow` exists, it is used directly.
- Otherwise, `list.permission` is merged with `show.permission` (any `show` settings override `list`).
- Then the check proceeds as usual.

---

## 7. Configuration Retrieval and Check Methods

### 7.1 Simplified Methods (recommended)

| Method | Description | Example |
|--------|-------------|---------|
| `checkByResource($resourceKey, $user, $overrideConfig)` | Checks permission using the full resource key | `AccessManager::checkByResource('nano.absenceapi::absences.list', $user)` |
| `getOperationByConfig($resourceKey, $overrideConfig)` | Returns the operation settings without performing the check | `$config = AccessManager::getOperationByConfig('nano.absenceapi::absences.list');` |
| `getAdvancedFiltersConfig($resourceKey)` | Returns only the `advanced_filters` array | `$adv = AccessManager::getAdvancedFiltersConfig('nano.absenceapi::absences.list');` |
| `checkWithFallback($resourceKey, $fallbackKey, $user, $overrideConfig)` | Checks with fallback to another operation | `AccessManager::checkWithFallback('...show', '...list', $user)` |

### 7.2 Advanced Methods (less common)

- `checkWithConfig($operation, $plugin, $resource, $user, $overrideConfig)`
- `getOperationConfig($plugin, $resource, $operation, $overrideConfig)`

---

## 8. Complete Practical Examples

### 8.1 Allow Only Backend Users to Perform the `delete` Operation (with a specific permission)

```php
'delete' => [
    'permission' => [
        'is_allow' => true,
        'backend' => [
            'allow' => true,
            'check_permission' => true,
            'permissions' => ['tss.absence.absences.delete'],
            'access_scope' => 'all',
        ],
        'frontend' => ['allow' => false],
        'guest' => ['allow' => false],
    ],
],
```

### 8.2 Make the `list` Operation Public (including guests) with Simple Filtering

```php
'list' => [
    'permission' => [
        'is_allow' => true,
        'guest' => ['allow' => true, 'access_scope' => 'all'],
        'frontend' => ['allow' => true, 'allowed_ref_types' => ['*']],
        'backend' => ['allow' => true, 'permissions' => []],
    ],
],
```

### 8.3 Allow Only Parents to Use `is_or_student_id`

```php
'advanced_filters' => [
    'enabled' => true,
    'default_for_frontend' => false,
    'field_specific_rules' => [
        'student_id' => [
            'frontend' => [
                'parent' => true,
                '*' => false,
            ],
        ],
    ],
],
```

### 8.4 Using `checkByResource` in the `Absences` Controller

```php
public function index()
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.absenceapi::absences.list', $user);
    if (!$access['allowed']) {
        return $this->errorUnauthorized($access['message']);
    }
    // ... rest of the code
}
```

---

## 9. Summary of Options and Settings

| Key | Level | Possible values | Explanation |
|-----|-------|-----------------|-------------|
| `is_allow` | global | true/false | Enable/disable the operation entirely |
| `backend.allow` | backend | true/false | Allow backend users |
| `backend.check_permission` | backend | true/false | Enable permission checking |
| `backend.permissions` | backend | array | List of required permissions (array or comma‑separated string) |
| `backend.access_scope` | backend | all, own, created_by, company, department, state | Access scope |
| `backend.company_field` | backend | string | Company field name in the table |
| `backend.department_field` | backend | string | Department field name |
| `backend.state_field` | backend | string | State field name |
| `backend.created_by_field` | backend | string | Record creator field name |
| `backend.company_id` | backend | int/string/null | Override company ID (overrides the user’s company) |
| `frontend.allow` | frontend | true/false | Allow frontend users |
| `frontend.allowed_ref_types` | frontend | array | Allowed user types (full or short) |
| `frontend.access_scope` | frontend | own, children | Access scope |
| `frontend.children_relation` | frontend | string | Relationship name to fetch children (for `children`) |
| `guest.allow` | guest | true/false | Allow unauthenticated guests |
| `guest.access_scope` | guest | all, none | Access scope for guests |
| `advanced_filters.enabled` | advanced_filters | true/false | Enable advanced filters |
| `advanced_filters.default_for_backend` | advanced_filters | true/false | Default allowance for backend |
| `advanced_filters.default_for_frontend` | advanced_filters | true/false | Default allowance for frontend |
| `advanced_filters.allowed_fields_backend` | advanced_filters | array | List of allowed fields for backend (`['*']` for all) |
| `advanced_filters.allowed_fields_frontend` | advanced_filters | array | List of allowed fields for frontend |
| `advanced_filters.field_specific_rules` | advanced_filters | array | Per‑field rules |

---

## 10. Tips and Recommendations

- **Use `checkByResource`** instead of direct `check` to reduce code and rely on `config.php`.
- **For `show` operations**, you can either customise its own settings or use `checkWithFallback` to fall back to `list`.
- **For advanced filters**, be precise when defining `allowed_fields_frontend` and `field_specific_rules` to prevent data leakage.
- **Testing permissions**: you can use `AccessManager::instance()->setUser($fakeUser)` to simulate a specific user in the test environment.
- **Use environment variables** in your `.env` file to override default settings without modifying `config.php`.

---

## 11. Conclusion

The permission system in `AccessManager` offers great flexibility, allowing you to control access to every API endpoint with precision, based on user type, access scope, and traditional permissions. By centralising settings in `config.php` and using simplified methods, it becomes easy to maintain and extend the permission system for various Nano projects.

**References:**

- [`AccessManager` class documentation](./Docs-AccessManager-en.md)
- [`AccessManager-Permission` documentation](./Docs-AccessManager-Permission-en.md)
- [General examples `AccessManager-Example`](./Docs-AccessManager-Example-en.md)
- [Comprehensive practical example `AccessManager-Example-Api`](./Docs-AccessManager-Example-Api-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)
- [`Absences` controller as a practical example](../AbsenceApi/APIControllers/Absences.php)
