## 2026-05-24 – 2026-05-25

**Update to the `Nano.AuthApi` Plugin – Version 1.0.20**

### Adding the `AccessManager` Class for Centralised and Advanced Permission and Access Management

---

### Summary of Updates

Version **1.0.20** of the `Nano.AuthApi` plugin introduces a new class called `AccessManager`, which is responsible for managing user permissions (Backend, Frontend, Guest) and controlling access scopes in a unified and extensible manner. The class integrates helper functions from `BasicHelper` (e.g., `checkAccessAllCompanys`) and provides a simple interface for applying access restrictions to Eloquent queries.

This update aims to unify permission logic across all system plugins, reduce duplicate code, and support advanced scenarios such as access by company, department, or state, showing only records created by the user, or displaying the children of a parent.

---

### Release Objectives

- **Provide a centralised class** for managing permissions instead of distributing logic across multiple controllers.
- **Support three user types:** Backend (Nano Soft App), Frontend (RainLab.User or any custom model), and Guest.
- **Support multiple access scopes:**
  - `all`: access to all records.
  - `own`: records created by the user (via `created_by`).
  - `created_by`: same as `own`.
  - `company`: access only to records of a specific company (with the ability to access all companies for authorised users).
  - `department`: access only to records of a specific department.
  - `state`: access only to records of a specific state.
  - `children`: for Frontend users of type parent, to display their children's records.
- **Provide methods to apply the scope to Query Builder** (`applyAccessScope`, `applyCompanyScope`, `applyDepartmentScope`, `applyStateScope`, `applyCreatedByScope`, `applyUserScope`, `applyChildrenScope`).
- **Support custom field names** (e.g., `companys_id`, `departments_id`, `state_id`, `created_by`) via flexible options.
- **Support permission checking** for Backend users using `hasAccess` or `hasPermission`.
- **Support filtering Frontend user types** via `ref_type` (e.g., `student`, `parent`).
- **Return detailed information** about the check result (allowed, scope, message, user type, user, config, applied scope details).

---

### New Features and Improvements

#### 1. `AccessManager` Class – Core Functions

- **`setUser($user) / getUser()`**: Set or get the current user (supports automatic retrieval from `AuthHelpers`, `BackendAuth`, `Auth`).
- **`getUserType($user)`**: Determines the user type (`backend`, `frontend`, `guest`).
- **`getUserRefType($user)`, `getUserRefId($user)`**: Extracts `ref_type` and `ref_id` (for users linked to models such as students).
- **`getUserCompanyId($user)`, `getUserDepartmentId($user)`**: Extracts the company ID and department ID from the user (if the user has those properties).
- **`check(string $operation, array $config, $user)`**: The core check method. Returns an array containing `allowed`, `scope`, `message`, `user_type`, `user`, `config`, `applied_scope_details`.

#### 2. Advanced Backend Scope Support

Backend behaviour can be customised via the `backend` option in the `$config` array:

```php
'backend' => [
    'allow' => true,                         // whether Backend users are allowed
    'check_permission' => true,              // whether to check permissions
    'permissions' => ['tss.example.access'], // required permissions
    'access_scope' => 'company',             // required scope
    'company_field' => 'companys_id',        // company field in the table
    'department_field' => 'departments_id',
    'state_field' => 'state_id',
    'created_by_field' => 'created_by',
    'company_id' => 2,                       // specific company ID (optional)
    'department_id' => 5,
    'state_id' => 1,
],
```

Supported `access_scope` values: `all`, `own`, `created_by`, `company`, `department`, `state`.

When using the `company`, `department`, or `state` scope, `AccessManager` first checks whether the user has permission to access all companies/departments/states (via `canAccessAllCompanies()` etc.). If they do, the effective scope becomes `all`. Otherwise, it uses the ID associated with the user or the ID passed in the settings.

#### 3. Frontend Support with `ref_type` Filtering and `children` Scope

```php
'frontend' => [
    'allow' => true,
    'allowed_ref_types' => ['student', 'parent'], // allowed types
    'access_scope' => 'own',                     // 'own' or 'children'
    'children_relation' => 'students',           // relationship name to fetch children
],
```

If the scope is `children`, the `children_relation` must be defined (e.g., `students` on a parent model). Then `applyChildrenScope()` can be used to apply the filter.

#### 4. Methods to Apply Scope to Query Builder

- **`applyAccessScope($query, $accessResult, $options)`**: A general method that uses the result of `check()` to apply the appropriate scope.
- **`applyCompanyScope($query, $companyId, $field)`**, **`applyDepartmentScope`**, **`applyStateScope`**, **`applyCreatedByScope`**, **`applyUserScope`**.
- **`applyChildrenScope($query, $parent, $studentIdField, $relation)`**: Applies a filter to fetch records of a specific parent’s children.

**Example:**

```php
$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed']) {
    $query = Student::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access, [
        'company_field' => 'companys_id',
        'department_field' => 'departments_id',
        'state_field' => 'state_id',
        'created_by_field' => 'created_by',
    ]);
    // if scope = children
    if ($access['scope'] === 'children') {
        $parent = Mparent::byUser($user)->first();
        $query = AccessManager::instance()->applyChildrenScope($query, $parent, 'student_id', 'students');
    }
}
```

#### 5. Helper Methods for Checking Company/Department/State Permissions

- **`canAccessAllCompanies($user)`**: Returns `true` if the user can access all companies (uses `BasicHelper::checkAccessAllCompanys`).
- **`canAccessAllDepartments($user)`**
- **`canAccessAllStates($user)`**
- **`canAccessDepartment($departmentId, $user)`**

These methods simplify the logic and ensure consistency with the rest of the system.

#### 6. `getPermissionArray` Method

Returns an array similar to that used in `OrderHelper::getArrayDataAfterPermission`, making it easier to integrate with legacy code.

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
// ['allowed'=>true, 'scope'=>'department', 'check_departments'=>true, 'departments_id'=>5, ...]
```

#### 7. Full Guest Support

Guest (unauthenticated) users can be allowed to access some operations via the `guest` settings:

```php
'guest' => [
    'allow' => true,
    'access_scope' => 'none',
],
```

---

### Usage Examples

#### 1. Checking Access to a List of Students for a Frontend Parent User

```php
$config = [
    'is_allow' => true,
    'frontend' => [
        'allow' => true,
        'allowed_ref_types' => ['parent'],
        'access_scope' => 'children',
        'children_relation' => 'students',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
if (!$access['allowed']) {
    throw new ApplicationException($access['message']);
}
// $access['scope'] = 'children'
```

#### 2. Applying `company` Scope for a Backend User

```php
$config = [
    'backend' => [
        'allow' => true,
        'check_permission' => true,
        'permissions' => ['tss.student.students.access'],
        'access_scope' => 'company',
        'company_field' => 'companys_id',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed']) {
    $query = Student::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access);
    // A condition will be added: companys_id = (the user's associated company ID or the one specified in config)
}
```

#### 3. Getting a Permission Settings Array for Use in `listExtendQuery`

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
if ($perms['allowed']) {
    if ($perms['check_departments']) {
        $query->where('departments_id', $perms['departments_id']);
    }
}
```

---

### Backward Compatibility

- **No old methods have been changed** in `Nano.AuthApi`. Only the `AccessManager` class has been added.
- Existing `AuthHelpers` functions (`getCurrentUser`, `isBackendUser`, etc.) can still be used without any impact.
- No database or configuration changes have been made.

---

### Upgrade Requirements (from any previous version to 1.0.20)

1. **Update the code**:
   - Replace the `classes/AccessManager.php` file with the new version (attached).
   - (Optional) Update `Plugin.php` if you want to register `AccessManager` as a Singleton or provide an alias, but it is not required.

2. **No new database migrations**.

3. **No configuration changes** – the `config.php` file remains the same.

4. **Update the `version.yaml` file**:
   - Add the line `1.0.20:` as shown at the beginning of this document.

5. **Run `php artisan plugin:refresh Nano.AuthApi`** (optional) to register the new version.

6. **Test the functionality**:
   - Test `AccessManager::check()` with different scenarios (Backend, Frontend, Guest).
   - Ensure that `applyAccessScope` adds correct `where` conditions.
   - Verify the `canAccessAllCompanies` and other helper methods.

---

### Benefits and Added Value

- **Unifies permission logic** across all API plugins (`Nano.StudentsApi`, `Nano.AbsenceApi`, ...), reducing code duplication and standardising behaviour.
- **Precise access control** based on company/department/state, making it easy to build multi‑tenant systems.
- **Native support for the `children` scope** for parents, simplifying school‑oriented applications.
- **Easy extensibility** to add new scopes (e.g., `team`, `project`) without modifying the base class.
- **Improved security** through a clear separation of permission logic from business logic.

---

### Conclusion

Version **1.0.20** of `Nano.AuthApi` represents an important step towards unifying permission management in the Nano ecosystem. With `AccessManager`, developers can easily apply complex access restrictions while maintaining flexibility and extensibility. We recommend using this class in all future API plugins to ensure security consistency and ease of maintenance.

---

**Reference documentation**:
- [`AccessManager` class documentation](./docs/AuthApi/Docs-AccessManager-en.md)
- [`AuthHelpers` class documentation](./docs/AuthApi/Docs-AuthHelpers-en.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md)
