## 12. Complete Example of API Controller Settings (list, show, create, update operations)

Here we provide a complete example of settings for a hypothetical `items` controller, demonstrating how to configure permissions for each operation using environment variables. The same structure applies to any resource (e.g., `absences`, `students`, `parents`).

### 12.1 Structure of the `config.php` File

```php
<?php

return [
    'items' => [
        // ------------------------------
        // list operation (fetching a list)
        // ------------------------------
        'list' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_LIST_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_LIST_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_LIST_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.access_all', 'tss.items.access'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_LIST_BACKEND_SCOPE', 'all'),
                    'company_field'      => env('NANO_ITEMSAPI_ITEMS_LIST_COMPANY_FIELD', 'companys_id'),
                    'department_field'   => env('NANO_ITEMSAPI_ITEMS_LIST_DEPARTMENT_FIELD', 'departments_id'),
                    'state_field'        => env('NANO_ITEMSAPI_ITEMS_LIST_STATE_FIELD', 'state_id'),
                    'created_by_field'   => env('NANO_ITEMSAPI_ITEMS_LIST_CREATED_BY_FIELD', 'created_by'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_LIST_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_SCOPE', 'own'),
                    'children_relation'  => env('NANO_ITEMSAPI_ITEMS_LIST_CHILDREN_RELATION', 'students'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_LIST_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_LIST_GUEST_SCOPE', 'none'),
                ],

                'advanced_filters' => [
                    'enabled'                => env('NANO_ITEMSAPI_ITEMS_ADVANCED_FILTERS_ENABLED', true),
                    'default_for_backend'    => env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_DEFAULT', true),
                    'default_for_frontend'   => env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_DEFAULT', false),
                    'allowed_fields_backend' => explode(',', env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_FIELDS', '*')),
                    'allowed_fields_frontend' => explode(',', env('NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_FIELDS', '')),
                    'field_specific_rules'   => [
                        'student_id' => [
                            'backend' => true,
                            'frontend' => [
                                '*' => false,
                                'parent' => true,
                            ],
                        ],
                        'record_id' => [
                            'backend' => true,
                            'frontend' => [
                                '*' => false,
                                'parent' => true,
                            ],
                        ],
                    ],
                ],
            ],
        ],

        // ------------------------------
        // show operation (display a single item)
        // ------------------------------
        'show' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_SHOW_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_SHOW_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_SHOW_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.access', 'tss.items.access_all'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_SHOW_BACKEND_SCOPE', 'all'),
                    'company_field'      => env('NANO_ITEMSAPI_ITEMS_SHOW_COMPANY_FIELD', 'companys_id'),
                    'department_field'   => env('NANO_ITEMSAPI_ITEMS_SHOW_DEPARTMENT_FIELD', 'departments_id'),
                    'state_field'        => env('NANO_ITEMSAPI_ITEMS_SHOW_STATE_FIELD', 'state_id'),
                    'created_by_field'   => env('NANO_ITEMSAPI_ITEMS_SHOW_CREATED_BY_FIELD', 'created_by'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_SHOW_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_SHOW_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_SHOW_FRONTEND_SCOPE', 'own'),
                    'children_relation'  => env('NANO_ITEMSAPI_ITEMS_SHOW_CHILDREN_RELATION', 'students'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_SHOW_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_SHOW_GUEST_SCOPE', 'none'),
                ],
            ],
        ],

        // ------------------------------
        // create operation (create a new record)
        // ------------------------------
        'create' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_CREATE_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_CREATE_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_CREATE_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.add'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_CREATE_BACKEND_SCOPE', 'all'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_CREATE_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_CREATE_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_CREATE_FRONTEND_SCOPE', 'own'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_CREATE_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_CREATE_GUEST_SCOPE', 'none'),
                ],
            ],
        ],

        // ------------------------------
        // update operation (update an existing record)
        // ------------------------------
        'update' => [
            'permission' => [
                'is_allow' => env('NANO_ITEMSAPI_ITEMS_UPDATE_IS_ALLOW', true),

                'backend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_UPDATE_BACKEND_ALLOW', true),
                    'check_permission'   => env('NANO_ITEMSAPI_ITEMS_UPDATE_CHECK_PERMISSION', true),
                    'permissions'        => ['tss.items.edit'],
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_UPDATE_BACKEND_SCOPE', 'all'),
                ],

                'frontend' => [
                    'allow'              => env('NANO_ITEMSAPI_ITEMS_UPDATE_FRONTEND_ALLOW', true),
                    'allowed_ref_types'  => explode(',', env('NANO_ITEMSAPI_ITEMS_UPDATE_ALLOWED_REF_TYPES', 'student,parent')),
                    'access_scope'       => env('NANO_ITEMSAPI_ITEMS_UPDATE_FRONTEND_SCOPE', 'own'),
                ],

                'guest' => [
                    'allow'         => env('NANO_ITEMSAPI_ITEMS_UPDATE_GUEST_ALLOW', false),
                    'access_scope'  => env('NANO_ITEMSAPI_ITEMS_UPDATE_GUEST_SCOPE', 'none'),
                ],
            ],
        ],

        // ------------------------------
        // General settings for the resource (optional)
        // ------------------------------
        'order_by'       => env('NANO_ITEMSAPI_ITEMS_ORDER_BY', 'sort_order'),
        'order_dir'      => env('NANO_ITEMSAPI_ITEMS_ORDER_DIR', 'asc'),
        'per_page'       => env('NANO_ITEMSAPI_ITEMS_PER_PAGE', 15),
        'exclude'        => env('NANO_ITEMSAPI_ITEMS_EXCLUDE', ''),
    ],
];
```

### 12.2 Explanation of the Environment Variables Used

| Environment variable | Purpose | Possible values (default) |
|----------------------|---------|---------------------------|
| `NANO_ITEMSAPI_ITEMS_LIST_IS_ALLOW` | Enable/disable the `list` operation | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_BACKEND_ALLOW` | Allow backend users | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_CHECK_PERMISSION` | Enable permission checking for backend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_BACKEND_SCOPE` | Access scope for backend | `all`, `own`, `company`, `department`, `state` |
| `NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_ALLOW` | Allow frontend users | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_LIST_ALLOWED_REF_TYPES` | Allowed `ref_type` values (comma‑separated) | `student,parent` |
| `NANO_ITEMSAPI_ITEMS_LIST_FRONTEND_SCOPE` | Access scope for frontend | `own`, `children` |
| `NANO_ITEMSAPI_ITEMS_LIST_CHILDREN_RELATION` | Relationship name to fetch children (for `children`) | `students` |
| `NANO_ITEMSAPI_ITEMS_LIST_GUEST_ALLOW` | Allow guests | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADVANCED_FILTERS_ENABLED` | Enable advanced filters | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_DEFAULT` | Default allowance for backend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_DEFAULT` | Default allowance for frontend | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_BACKEND_FIELDS` | Allowed fields for backend (comma‑separated) | `*` or `field1,field2` |
| `NANO_ITEMSAPI_ITEMS_ADV_FILTERS_FRONTEND_FIELDS` | Allowed fields for frontend | `""` or `field1,field2` |
| `NANO_ITEMSAPI_ITEMS_SHOW_IS_ALLOW` | Enable the `show` operation | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_CREATE_IS_ALLOW` | Enable the `create` operation | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_UPDATE_IS_ALLOW` | Enable the `update` operation | `true` / `false` |
| `NANO_ITEMSAPI_ITEMS_ORDER_BY` | Default ordering field | `sort_order` |
| `NANO_ITEMSAPI_ITEMS_ORDER_DIR` | Default ordering direction | `asc` or `desc` |
| `NANO_ITEMSAPI_ITEMS_PER_PAGE` | Default number of results per page | integer |
| `NANO_ITEMSAPI_ITEMS_EXCLUDE` | Default excluded columns | `""` or comma‑separated column names |

---

### 12.3 How to Use These Settings in the `Items` Controller

In the `Items` controller, you can use the simplified `AccessManager` methods as follows:

```php
public function index()
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.itemsapi::items.list', $user);
    // ...
}

public function show($id)
{
    $user = $this->current_user;
    // show may use the same settings as list if not customised, or use checkWithFallback
    $access = AccessManager::checkWithFallback(
        'nano.itemsapi::items.show',
        'nano.itemsapi::items.list',
        $user
    );
    // ...
}

public function store()
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.itemsapi::items.create', $user);
    // ...
}

public function update($id)
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.itemsapi::items.update', $user);
    // ...
}
```

> **Note:** `checkByResource` expects a `permission` key inside the settings. Therefore, the `config.php` structure shown above fully matches these methods.

---

### 12.4 Override Strategy

You can override any setting by passing `$overrideConfig` to methods like `checkByResource` or `getOperationByConfig`. For example:

```php
$override = [
    'backend' => [
        'allow' => false,
    ],
];
$access = AccessManager::checkByResource('nano.itemsapi::items.list', $user, $override);
```

This is useful when you want to change behaviour for a specific context without modifying the `config.php` file.

---

## 13. Summary

- **All permission settings** must be placed under the `permission` key of each operation inside `config.php`.
- **Use environment variables** to avoid hard‑coded values and to ease switching between environments (development, testing, production).
- **Access scopes** such as `company`, `department`, `state`, `children` are handled automatically by `AccessManager` when `applyAccessScope` is applied.
- **Advanced filters** give fine‑grained control over which fields users are allowed to use `is_or`, `is_not`, `is_or_null` on.
- **Simplified methods** (`checkByResource`, `getOperationByConfig`, `checkWithFallback`) make using `AccessManager` clean and fully configuration‑driven.

With this framework, you can build any API plugin with a flexible and easily extensible permission system.