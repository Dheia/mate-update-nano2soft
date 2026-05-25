# 📘 `AccessManager` Class Documentation

**Version:** 1.0.20  
**Path:** `Nano\AuthApi\Classes\AccessManager`  
**Purpose:** Centralised and advanced management of user permissions and access scopes.

---

## 1. Introduction

The `AccessManager` class is a unified solution for handling permission checks in the Nano ecosystem. It covers three types of users:

- **Backend** – control panel users (`Backend\Models\User`).
- **Frontend** – front‑end users (`RainLab\User\Models\User` or any custom model).
- **Guest** – unauthenticated users.

It also supports **advanced access scopes** such as:

- `all` – access to all records.
- `own` / `created_by` – access only to records created by the user.
- `company` – access to records of a specific company.
- `department` – access to records of a specific department.
- `state` – access to records of a specific state.
- `children` – for Frontend users of type **parent**, to display their children’s records.

`AccessManager` leverages available `BasicHelper` functions (e.g., `checkAccessAllCompanys`, `checkAccessAllDepartments`) and integrates seamlessly with `AdvancedQueryHelper` to build permission‑restricted Eloquent queries.

---

## 2. Benefits and Use Cases

| Benefit | Description |
|---------|-------------|
| **Centralised permissions** | Instead of repeating check logic in every API controller, call `AccessManager::check()` in one place. |
| **High flexibility** | Supports multiple user types and access scopes, with configurable field names. |
| **Extensibility** | New scopes (e.g., `team`, `branch`) can be added without modifying the core code. |
| **Integration with `AdvancedQueryHelper`** | `applyAccessScope` methods directly add `where` conditions to Eloquent queries, improving security and efficiency. |
| **Multi‑level checking** | First checks global settings (`is_allow`), then type‑specific settings (backend/frontend/guest), then the required scope, and finally permissions. |
| **Detailed return information** | `check()` returns an array containing `allowed`, `scope`, `message`, `applied_scope_details` for use in upper layers. |

**Typical use cases:**

- Protecting API endpoints (list, create, update, delete).
- Determining whether a user sees only their branch or company records.
- Allowing a parent to see their children’s records.
- Completely disabling certain operations for Guest users.

---

## 3. Class Methods and Properties

### 3.1 Setter / Getter Methods

| Method | Description |
|--------|-------------|
| `setUser($user)` | Manually sets the user (overrides automatic retrieval). |
| `getUser()` | Returns the current user (fetched automatically if not set). |
| `clearUser()` | Clears the cached user. |
| `getUserType($user = null)` | Returns `'backend'`, `'frontend'`, or `'guest'`. |
| `getUserRefType($user = null)` | Extracts `ref_type` from the user. |
| `getUserRefId($user = null)` | Extracts `ref_id` from the user. |
| `getUserCompanyId($user = null)` | Extracts `companys_id` (if exists). |
| `getUserDepartmentId($user = null)` | Extracts `departments_id` (if exists). |

### 3.2 Core Check Method

#### `check(string $operation, array $config = [], $user = null): array`

**Parameters:**

- `$operation`: operation name (`'list'`, `'create'`, `'update'`, `'delete'`, `'view'`, …).
- `$config`: operation configuration array (see the Configuration section below).
- `$user`: user object (optional, fetched automatically if not passed).

**Return value:**

```php
[
    'allowed' => bool,                // whether the operation is allowed
    'scope'   => string,              // effective scope (all, own, created_by, company, department, state, children, none)
    'message' => string,              // explanatory message
    'user_type' => string,            // backend / frontend / guest
    'user'    => mixed,               // user object used in the check
    'config'  => array,               // original configuration
    'applied_scope_details' => array  // details of the applied scope (e.g., company_id, department_id, state_id, user_id)
]
```

### 3.3 Backend Permission Helper Methods

| Method | Description |
|--------|-------------|
| `canAccessAllCompanies($user = null)` | Can the user access all companies? |
| `canAccessAllDepartments($user = null)` | Can the user access all departments? |
| `canAccessAllStates($user = null)` | Can the user access all states? |
| `canAccessDepartment($departmentId, $user = null)` | Can the user access a specific department? |

### 3.4 Methods to Apply Scope to Query Builder

| Method | Description |
|--------|-------------|
| `applyAccessScope($query, array $accessResult, array $options = [])` | Applies the general scope based on the result of `check()`. |
| `applyCompanyScope($query, $companyId = null, string $field = 'companys_id')` | Adds a `where` condition on the company field. |
| `applyDepartmentScope($query, $departmentId = null, string $field = 'departments_id')` | Adds a `where` condition on the department field. |
| `applyStateScope($query, $stateId = null, string $field = 'state_id')` | Adds a `where` condition on the state field (supports array). |
| `applyCreatedByScope($query, $userId = null, string $field = 'created_by')` | Adds a `where` condition on the record creator field. |
| `applyUserScope($query, $user = null, string $userIdField = 'user_id', string $userTypeField = 'user_type')` | Adds `where` conditions on `user_id` and `user_type`. |
| `applyChildrenScope($query, $parent, string $studentIdField = 'student_id', string $relation = 'students')` | Adds a `whereIn` condition on the child IDs of a given parent. |

### 3.5 Additional Helper Methods

- **`getPermissionArray(string $operation, array $config = []): array`**  
  Returns an array similar to that used in `OrderHelper::getArrayDataAfterPermission`, easing integration with legacy code.

- **`make(): self`** – creates a new instance (for non‑singleton use).
- **`instance(): self`** – returns the singleton instance.

---

## 4. Operation Configuration (`$config`)

The `$config` array is the heart of `AccessManager`. It should contain the following keys (some may be omitted if not needed):

```php
$config = [
    // Global settings
    'is_allow' => true,   // if false, the operation is immediately denied

    // Guest settings (optional)
    'guest' => [
        'allow' => false,
        'access_scope' => 'none',
    ],

    // Backend settings
    'backend' => [
        'allow'              => true,
        'check_permission'   => true,
        'permissions'        => ['tss.example.access'],   // can be comma‑separated string or array
        'access_scope'       => 'all',                    // all, own, created_by, company, department, state
        'company_field'      => 'companys_id',
        'department_field'   => 'departments_id',
        'state_field'        => 'state_id',
        'created_by_field'   => 'created_by',
        'company_id'         => null,                     // specific company ID (optional)
        'department_id'      => null,
        'state_id'           => null,
    ],

    // Frontend settings
    'frontend' => [
        'allow'              => true,
        'allowed_ref_types'  => ['student', 'parent'],    // allowed ref_type values
        'access_scope'       => 'own',                    // own or children
        'children_relation'  => 'students',               // relationship name to fetch children (required if access_scope = children)
    ],

    // Default scope (used if not specified in the appropriate type)
    'default_access_scope' => 'none',
];
```

**Important notes:**

- If `backend.access_scope` is set to `company`, `department`, or `state`, the class first checks the `canAccessAll*` permission. If `true`, the effective scope becomes `all`; otherwise it uses the specified ID.
- `frontend.allowed_ref_types` can contain fully qualified class names (e.g., `Tss\Student\Models\Student`) or short names (`student`). Matching is done automatically.

---

## 5. Complete and Detailed Examples

### 5.1 Basic Backend Check (Scope `all`)

```php
use Nano\AuthApi\Classes\AccessManager;

$config = [
    'is_allow' => true,
    'backend' => [
        'allow' => true,
        'check_permission' => true,
        'permissions' => ['tss.student.students.access_all', 'tss.student.students.access'],
        'access_scope' => 'all',
    ],
];

$user = BackendAuth::getUser();
$access = AccessManager::instance()->check('list', $config, $user);

if (!$access['allowed']) {
    throw new ApplicationException($access['message']);
}

// $access['scope'] === 'all'
```

### 5.2 `company` Scope for a Backend User (access to a single company)

```php
$config = [
    'backend' => [
        'allow' => true,
        'access_scope' => 'company',
        'company_field' => 'companys_id',
        // 'company_id' => 2, // if not passed, the user's own company_id is used
    ],
];

$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed'] && $access['scope'] === 'company') {
    $query = Order::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access);
    // Condition added: companys_id = (company ID)
}
```

### 5.3 `children` Scope for a Frontend Parent User

```php
$config = [
    'frontend' => [
        'allow' => true,
        'allowed_ref_types' => ['parent', 'Tss\Student\Models\Mparent'],
        'access_scope' => 'children',
        'children_relation' => 'students', // relationship on the Mparent model
    ],
];

$user = AuthHelpers::getCurrentUser(); // Frontend user linked to an Mparent
$access = AccessManager::instance()->check('list', $config, $user);

if ($access['allowed'] && $access['scope'] === 'children') {
    // Apply the scope manually or use applyAccessScope then applyChildrenScope
    $query = StudentRecord::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access);
    // Now call applyChildrenScope if not automatically applied (optional)
    $parent = StudentHelper::getMparentByUser($user);
    $query = AccessManager::instance()->applyChildrenScope($query, $parent, 'student_id', 'students');
}
```

### 5.4 Using `getPermissionArray` to Obtain a Settings Array Similar to `OrderHelper`

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
if ($perms['allowed']) {
    if ($perms['check_departments']) {
        // In legacy listExtendQuery methods
        $query->where('departments_id', $perms['departments_id']);
    }
    if ($perms['check_state']) {
        $query->where('state_id', $perms['state_id']);
    }
}
```

### 5.5 Integrating `AccessManager` with `AdvancedQueryHelper` in a Full API Controller

```php
public function index()
{
    $this->init();
    $user = $this->current_user;

    $config = [
        'is_allow' => Config::get('nano.exampleapi::items.is_allow_list', true),
        'backend' => [
            'allow' => Config::get('nano.exampleapi::items.is_allow_list_backend', true),
            'check_permission' => Config::get('nano.exampleapi::items.is_check_list_permission', true),
            'permissions' => ['tss.example.items.access'],
            'access_scope' => 'company',
            'company_field' => 'companys_id',
        ],
        'frontend' => [
            'allow' => Config::get('nano.exampleapi::items.is_allow_list_frontend', true),
            'allowed_ref_types' => ['student', 'parent'],
            'access_scope' => 'own',
        ],
    ];

    $access = AccessManager::instance()->check('list', $config, $user);
    if (!$access['allowed']) {
        return $this->errorUnauthorized($access['message']);
    }

    $options = (array) $this->data;
    $options['access_result'] = $access;

    // Using StudentRecordsHelper which applies the scope automatically
    $result = StudentRecordsHelper::getRecords(new Student(), $options);
    return $result['data']; // or transform as needed
}
```

### 5.6 `created_by` Scope to Show Only the User’s Own Records

```php
$config = [
    'backend' => [
        'allow' => true,
        'access_scope' => 'created_by',
        'created_by_field' => 'created_by',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
// If the user has access_all permissions, scope becomes 'all'; otherwise 'created_by'
```

### 5.7 Completely Disable an Operation for All Users

```php
$config = [
    'is_allow' => false,   // overrides any other permissions
];
$access = AccessManager::instance()->check('delete', $config);
// $access['allowed'] === false, $access['message'] indicates the operation is disabled.
```

---

## 6. Summary

| Feature | Details |
|---------|---------|
| **Ease of use** | A single `check()` function covers most scenarios. |
| **Flexible configuration** | Each operation (`list`, `create`, `update`, `delete`) can be customised independently. |
| **Multi‑environment support** | Works with Nano Soft App Backend, RainLab.User Frontend, and any custom models. |
| **Integration with `BasicHelper`** | Uses `checkAccessAllCompanys` and similar functions to maintain consistency. |
| **Automatic scope application** | `applyAccessScope` methods simplify adding `where` conditions to Eloquent queries. |
| **Test‑friendly** | You can easily inject a mock user with `setUser()`. |

**Common usage scenarios:**

- Protecting student lists so that each employee sees only students of their branch.
- Allowing a parent to see their children’s records but not others.
- Displaying financial reports only for authorised users (via `created_by`).
- Restricting delete operations to users who have a specific permission.

---

## 7. Conclusion

The `AccessManager` class is a cornerstone for any secure API system in the Nano environment. Thanks to its flexible and integrated design, developers can focus on business logic while ensuring that permissions are applied in a unified and maintainable way. We strongly recommend using this class in all future `Nano.*Api` plugins, and updating existing plugins to benefit from it.

**References:**

- [`AccessManager-Permission` documentation](./Docs-AccessManager-Permission-en.md)
- [General examples `AccessManager-Example`](./Docs-AccessManager-Example-en.md)
- [Comprehensive practical example `AccessManager-Example-Api`](./Docs-AccessManager-Example-Api-en.md)
- [Full documentation of the `Nano.AuthApi` plugin](./Docs-AuthApi-index.md)
- [`AdvancedQueryHelper` class (Nano2.QueryBuilder)](../querybuilder/Docs-AdvancedQueryHelper.md)
- [API Plugin Development Guide (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)

---
