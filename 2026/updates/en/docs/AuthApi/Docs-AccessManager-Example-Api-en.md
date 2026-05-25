## 13. Complete API Controller Example (list, show, create, update operations)

```php
<?php namespace Nano\ItemsApi\APIControllers;

use App;
use Lang;
use Event;
use Input;
use Config;
use Carbon\Carbon;

use Nano\API\Classes\ApiController;
use Nano\ItemsApi\Transformers\ItemTransformer;
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\AuthApi\Classes\AccessManager;
use Nano2\QueryBuilder\Classes\AdvancedQueryHelper as AdvancedQueryManager;

use Tss\Items\Models\Item;
use Tss\Student\Models\Student;
use Tss\Student\Models\Mparent;
use Tss\Studyyear\Models\Period;
use Tss\Basic\Helpers\BasicHelper;

use Exception;
use ApplicationException;
use ValidationException;
use October\Rain\Auth\AuthException;

/**
 * Complete API controller example using all AccessManager features.
 * 
 * @package Nano\ItemsApi\APIControllers
 */
class Items extends ApiController
{
    /**
     * Initialise the controller: fetch the current user.
     */
    public function init()
    {
        $this->current_user = $this->getUser() ?? AuthHelpers::getCurrentUser();
    }

    // ======================= LIST ENDPOINT =======================

    /**
     * Fetch a list of items with full support for permissions, scopes, and advanced filters.
     *
     * @return mixed
     */
    public function index()
    {
        try {
            $this->init();
            $user = $this->current_user;

            // -----------------------------------------------------------------
            // 1. Check permission using the simplified resource key from config
            // -----------------------------------------------------------------
            $access = AccessManager::checkByResource('nano.itemsapi::items.list', $user);
            if (!$access['allowed']) {
                return $this->errorUnauthorized($access['message']);
            }

            // -----------------------------------------------------------------
            // 2. Prepare request options
            // -----------------------------------------------------------------
            $options = (array) $this->data;
            $options['access_result'] = $access;

            // -----------------------------------------------------------------
            // 3. Apply advanced filter constraints (is_or / is_not / is_or_null)
            //    - Read settings from config (the same resource)
            //    - Filter options (remove disallowed filters)
            // -----------------------------------------------------------------
            $advFiltersConfig = AccessManager::getAdvancedFiltersConfig('nano.itemsapi::items.list');
            $options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advFiltersConfig]);

            // -----------------------------------------------------------------
            // 4. Special handling for Frontend users (students and parents)

            // Use the dynamic resolver:
            $options = AccessManager::instance()->resolveDynamicFrontendOptions(
                $user,
                $options,
                null,  // do not pass a query here; filters will be applied later in getRecords
                'nano.absenceapi::absences.list'
            );
            //or
            /*
            //    - resolveFrontendAccessOptions resolves student_id, record_id,
            //      and checks parent ownership of children.
            // -----------------------------------------------------------------
            if (AccessManager::instance()->getUserType($user) === 'frontend') {
                try {
                    $options = AccessManager::instance()->resolveFrontendAccessOptions($user, $options);
                } catch (ApplicationException $e) {
                    return $this->errorWrongArgs($e->getMessage());
                }
            }
            */

            // -----------------------------------------------------------------
            // 5. Caching (optional)
            // -----------------------------------------------------------------
            if (Config::get('nano.api::api_enable_cache')) {
                $callable = function () use ($options) {
                    return $this->getRecords($options);
                };
                return $this->cached('nano.itemsapi.items', $callable, [], $this->getLastUpdateAt());
            }

            return $this->getRecords($options);
        } catch (Exception $e) {
            return $this->errorWrongArgs($e->getMessage());
        }
    }

    /**
     * Core function to build and execute the item retrieval query.
     *
     * @param array $options
     * @return mixed
     */
    public function getRecords(array $options = [])
    {
        // ======================= 1. Merge default options =======================
        $defaults = [
            // Basic filters
            'id' => null,
            'is_or_id' => false,
            'is_not_id' => false,
            'is_force_id' => false,
            'is_or_null_id' => false,

            'student_id' => null,
            'is_or_student_id' => false,
            'is_not_student_id' => false,
            'is_force_student_id' => false,
            'is_or_null_student_id' => false,

            'record_id' => null,
            'is_or_record_id' => false,
            'is_not_record_id' => false,
            'is_force_record_id' => false,
            'is_or_null_record_id' => false,

            'year_id' => null,
            'is_or_year_id' => false,
            'is_not_year_id' => false,
            'is_force_year_id' => false,
            'is_or_null_year_id' => false,

            'status' => '*',
            'is_or_status' => false,
            'is_not_status' => false,
            'is_force_status' => false,
            'is_or_null_status' => false,

            'is_active' => true,

            // Text search
            'q' => null,
            'is_advanced_search' => null,

            // Relationships and exclusions
            'exclude' => null,
            'custom_with' => null,
            'with_count' => false,

            // Ordering and grouping
            'orderBy' => null,
            'orderDirection' => 'asc',
            'is_not_sort' => false,
            'group_by' => null,
            'having' => null,

            // Pagination
            'page' => Input::get('page', 1),
            'per_page' => null,
            'take_limet' => null,

            // Output type
            'is_collection' => false,
            'is_paginator' => false,
            'is_query' => false,
            'is_first' => false,
            'is_model' => false,
            'is_to_sql' => false,

            // Permission result
            'access_result' => null,
        ];

        $options = array_merge($defaults, $options);

        // Default settings from config
        $exclude = $options['exclude'] ?? Config::get('nano.itemsapi::items.exclude', '');
        $orderBy = $options['orderBy'] ?? Config::get('nano.itemsapi::items.order_by', 'sort_order');
        $orderDirection = $options['orderDirection'] ?? Config::get('nano.itemsapi::items.order_dir', 'asc');
        $per_page = $options['per_page'] ?? Config::get('nano.itemsapi::items.per_page', 15);

        // ======================= 2. Build the base query =======================
        $model = new Item();
        $table = $model->getTable();
        $query = Item::query();

        // Exclude columns
        if (!empty($exclude) && (method_exists($query, 'scopeExclude') || method_exists($query, 'exclude'))) {
            $columns = is_array($exclude) ? $exclude : explode(',', $exclude);
            $query = $query->exclude($columns);
        }

        // Load required relationships
        if (!empty($options['custom_with'])) {
            $with = is_array($options['custom_with']) ? $options['custom_with'] : explode(',', $options['custom_with']);
            $query = $query->with($with);
        }

        // ======================= 3. Apply access scope =======================
        // You can control which scopes are applied through settings in config.php
        // e.g. in config.php:
        // 'backend' => [
        //     'access_scope' => 'company',
        //     'apply_company_scope' => true,
        //     'apply_department_scope' => false,
        //     'apply_state_scope' => false,
        //     'apply_created_by_scope' => true,
        // ]
        if (!empty($options['access_result']) && $options['access_result']['allowed']) {
            // Read control settings from config (can be stored in access_result or read directly)
            $applyCompany = Config::get('nano.itemsapi::items.list.permission.backend.apply_company_scope', true);
            $applyDepartment = Config::get('nano.itemsapi::items.list.permission.backend.apply_department_scope', true);
            $applyState = Config::get('nano.itemsapi::items.list.permission.backend.apply_state_scope', true);
            $applyCreatedBy = Config::get('nano.itemsapi::items.list.permission.backend.apply_created_by_scope', true);

            // Build dynamic options array based on settings
            $scopeOptions = [];
            if ($applyCompany) $scopeOptions['company_field'] = 'companys_id';
            if ($applyDepartment) $scopeOptions['department_field'] = 'departments_id';
            if ($applyState) $scopeOptions['state_field'] = 'state_id';
            if ($applyCreatedBy) $scopeOptions['created_by_field'] = 'created_by';

            $query = AccessManager::instance()->applyAccessScope($query, $options['access_result'], $scopeOptions);
        }

        // ======================= 4. Apply student/parent filters (optional, useful for absence, result tables) =======================
        // Apply the dynamic resolver and directly modify the query (without storing results)
        $options = AccessManager::instance()->resolveDynamicFrontendOptions(
            $user,
            $options,
            $query,
            'nano.absenceapi::absences.list'
        );
        
        //or
        /*
        // Pass $table to automatically add the table prefix to fields (to avoid conflicts when JOINing)
        $query = AccessManager::instance()->applyFrontendStudentParentFilters(
            $query,
            $user ?? null,
            $options,
            'student_id',          // student_id column name in the current table
            'record_id',           // record_id column name
            $table,                // table prefix (automatically added to fields)
            false                  // resolveIfNeeded = false because we already resolved in index()
        );
        */
        //or
        /*
        $query = AccessManager::instance()->applyFrontendStudentParentFilters(
            $query,
            $user,
            $options,
            'student_id',     // base column name (without prefix)
            'record_id',      // base column name
            $table,           // table name (e.g., 'absences') – will be added automatically
            true              // resolves options automatically if needed
        )
        */

        // ======================= 5. Apply standard filters using AdvancedQueryManager =======================
        // id filter
        if ($options['id'] || $options['is_force_id']) {
            $query = AdvancedQueryManager::scopeWhereField(
                $query, 'id', $options['id'],
                $options['is_or_id'], $table,
                $options['is_not_id'], $options['is_force_id']
            );
        }

        // student_id filter (if not using applyFrontendStudentParentFilters above, it can be applied here)
        if ($options['student_id'] !== null || $options['is_force_student_id']) {
            $query = AdvancedQueryManager::scopeWhereField(
                $query, 'student_id', $options['student_id'],
                $options['is_or_student_id'], $table,
                $options['is_not_student_id'], $options['is_force_student_id']
            );
        }

        // academic year filter (with automatic fallback to the default year)
        if ($options['year_id'] !== null) {
            $query = AdvancedQueryManager::scopeWhereField(
                $query, 'year_id', $options['year_id'],
                $options['is_or_year_id'], $table,
                $options['is_not_year_id'], $options['is_force_year_id']
            );
        } else {
            $primaryYear = Period::getPrimary();
            if ($primaryYear) {
                $query->where($table . '.year_id', $primaryYear->id);
            }
        }

        // status filter
        if ($options['status'] && $options['status'] !== '*') {
            $query = AdvancedQueryManager::scopeWhereField(
                $query, 'status', $options['status'],
                $options['is_or_status'], $table,
                $options['is_not_status'], $options['is_force_status']
            );
        }

        // is_active filter
        if ($options['is_active'] !== null && $options['is_active'] !== '*') {
            $options['is_active'] ? $query->isActive() : $query->isNotActive();
        }

        // Text search (q)
        if ($options['q']) {
            $q = trim($options['q']);
            $isAdvanced = $options['is_advanced_search'] ?? Config::get('nano.itemsapi::items.is_advanced_search', false);
            $advMode = $options['advanced_search_mode'] ?? Config::get('nano.itemsapi::items.advanced_search_mode', 'OR');

            $query->where(function ($sub) use ($q, $isAdvanced, $advMode, $table) {
                $sub->where($table . '.name', 'like', "%{$q}%")
                    ->orWhere($table . '.description', 'like', "%{$q}%");
                if ($isAdvanced && class_exists(\Nano2\ArPhp\Classes\ArPhpHelper::class)) {
                    $condition = \Nano2\ArPhp\Classes\ArPhpHelper::getSearchCondition($q, $table . '.name', $advMode);
                    $sub->orWhereRaw($condition);
                }
            });
        }

        // with_count
        if ($options['with_count']) {
            $relations = is_array($options['with_count']) ? $options['with_count'] : explode(',', $options['with_count']);
            $query->withCount($relations);
        }

        // ======================= 6. Extensible events =======================
        Event::fire('api.list.extendQueryBefore', [$query, $this, $model, $options]);

        // Group by (supports array)
        if ($options['group_by']) {
            $groupFields = is_array($options['group_by']) ? $options['group_by'] : [$options['group_by']];
            foreach ($groupFields as $field) {
                $query->groupBy(\DB::raw($field));
            }
        }

        // Having
        if ($options['having']) {
            if (is_string($options['having'])) {
                $query->havingRaw($options['having']);
            } elseif (is_array($options['having'])) {
                foreach ($options['having'] as $column => $value) {
                    if (is_array($value)) {
                        $query->having($column, $value[0] ?? '=', $value[1] ?? null);
                    } else {
                        $query->having($column, '=', $value);
                    }
                }
            }
        }

        // Ordering
        if (!$options['is_not_sort'] && $orderBy && $orderDirection) {
            $query->orderBy(\DB::raw($orderBy), \DB::raw($orderDirection));
        }

        Event::fire('api.list.extendQuery', [$query, $this, $model, $options]);

        // ======================= 7. Limits and SQL debugging =======================
        if ($options['take_limet']) {
            $query->take((int)$options['take_limet']);
        }
        if ($options['is_to_sql']) {
            $sql = vsprintf(str_replace('?', '"%s"', $query->toSql()), $query->getBindings());
            trace_log($sql);
        }

        // ======================= 8. Execute query according to output type =======================
        if ($options['is_query']) {
            return $query;
        }
        if ($options['is_first'] || $options['is_model']) {
            return $query->first();
        }
        if ($options['is_collection']) {
            return $query->get();
        }
        if ($options['is_paginator']) {
            return $query->paginate((int)$per_page, (int)$options['page']);
        }

        $paginator = $query->paginate((int)$per_page, (int)$options['page']);
        Event::fire('ItemsApi.ApiControllers.Items.List', [$paginator]);

        $transformer = new ItemTransformer();
        return $this->respondWithPaginator($paginator, $transformer);
    }

    // ======================= SHOW ENDPOINT =======================
    public function show($id)
    {
        try {
            $this->init();
            $user = $this->current_user;

            // Use fallback: if show settings are missing, use list settings
            $access = AccessManager::checkWithFallback(
                'nano.itemsapi::items.show',
                'nano.itemsapi::items.list',
                $user
            );
            if (!$access['allowed']) {
                return $this->errorUnauthorized($access['message']);
            }

            $options = (array) $this->data;
            $options['access_result'] = $access;

            // Apply frontend handling if needed
            if (AccessManager::instance()->getUserType($user) === 'frontend') {
                try {
                    $options = AccessManager::instance()->resolveFrontendAccessOptions($user, $options);
                } catch (ApplicationException $e) {
                    return $this->errorWrongArgs($e->getMessage());
                }
            }

            $query = Item::query();
            $query = AccessManager::instance()->applyAccessScope($query, $access, [
                'company_field'    => 'companys_id',
                'department_field' => 'departments_id',
                'state_field'      => 'state_id',
                'created_by_field' => 'created_by',
            ]);

            $query = AccessManager::instance()->applyFrontendStudentParentFilters(
                $query, $user, $options, 'student_id', 'record_id', (new Item())->getTable(), false
            );

            $item = $query->find($id);
            if (!$item) {
                return $this->errorNotFound(Lang::get('nano.itemsapi::lang.items.list.msg_not_found'));
            }
            
            // Additional check: if the user is a parent, ensure the record belongs to one of their children (extra security)
            if ($userType === 'frontend') {
                $refType = AccessManager::instance()->getUserRefType($user);
                if ($refType === 'Tss\Student\Models\Mparent' || $refType === 'parent') {
                    $parent = StudentHelper::getMparentByUser($user);
                    if ($parent) {
                        $studentIds = StudentHelper::getStudentIdsByMparent($parent);
                        if (!in_array($item->student_id, $studentIds)) {
                            return $this->errorUnauthorized('You are not allowed to view this absence record.');
                        }
                    }
                }
            }
    

            Event::fire('ItemsApi.ApiControllers.Items.Show', [$item]);
            return $this->respondWithItem($item, new ItemTransformer());
        } catch (Exception $e) {
            return $this->errorWrongArgs($e->getMessage());
        }
    }

    // ======================= CREATE / UPDATE UNIFIED =======================
    public function store()
    {
        return $this->onStoreUpdate('create');
    }

    public function update($id = null)
    {
        return $this->onStoreUpdate('update', $id);
    }

    protected function onStoreUpdate(string $action = 'create', ?int $id = null)
    {
        $result = [
            'code'    => 200,
            'status'  => true,
            'message' => $action === 'create'
                ? Lang::get('nano.itemsapi::lang.items.create.success')
                : Lang::get('nano.itemsapi::lang.items.update.success'),
            'error'   => null,
            'errors'  => null,
            'data'    => [],
        ];
        $defaultError = $action === 'create'
            ? Lang::get('nano.itemsapi::lang.items.create.error')
            : Lang::get('nano.itemsapi::lang.items.update.error');

        try {
            $user = AuthHelpers::getCurrentUser();
            if (!$user) {
                throw new AuthException(Lang::get('nano.itemsapi::lang.general.not_found_user'));
            }

            $resourceKey = 'nano.itemsapi::items.' . $action;
            $access = AccessManager::checkByResource($resourceKey, $user);
            if (!$access['allowed']) {
                throw new ApplicationException($access['message']);
            }

            $data = $this->data ?: Input::all();
            if (empty($data)) {
                throw new ApplicationException(Lang::get('nano.itemsapi::lang.items.create.missing_data'));
            }

            if ($action === 'update') {
                $model = Item::find($id);
                if (!$model) {
                    throw new ApplicationException(Lang::get('nano.itemsapi::lang.items.update.not_found'));
                }
            } else {
                $model = new Item();
                // Fill default values if available
                if (empty($data['companys_id'])) {
                    $data['companys_id'] = BasicHelper::getCompanysId(true);
                }
                if (empty($data['departments_id'])) {
                    $data['departments_id'] = BasicHelper::getMainDepartmentId(true);
                }
            }

            $model->fill($data);
            $model->save();

            $transformer = new ItemTransformer();
            $result['data'] = $this->withForceArrayOutput(true)->respondWithItem($model, $transformer);
            $this->withForceArrayOutput(false);
            $result['model'] = $model;
            $result['process_data'] = $model->toArray();
            $result['input_data'] = $data;

        } catch (AuthException $e) {
            $result['code'] = 401;
            $result['status'] = false;
            $result['message'] = $defaultError;
            $result['error'] = $e->getMessage();
        } catch (Exception | ApplicationException | ValidationException $e) {
            $result['code'] = 400;
            $result['status'] = false;
            $result['message'] = $defaultError;
            $result['error'] = $e->getMessage();
            if (method_exists($e, 'errors')) $result['errors'] = $e->errors();
            if (Config::get('app.debug')) {
                $result['debug'] = [
                    'line'  => $e->getLine(),
                    'file'  => $e->getFile(),
                    'class' => get_class($e),
                    'trace' => explode("\n", $e->getTraceAsString()),
                ];
            }
        }

        return $this->respondWithItem([], function () use ($result) {
            return $result;
        });
    }

    // ======================= UTILITY =======================
    public function getLastUpdateAt()
    {
        return (new Item())->max('updated_at');
    }

    public function activelystats()
    {
        try {
            $lastUpdated = $this->getLastUpdateAt();
            $result = [
                'code' => 200,
                'data' => [
                    'last_updated'   => $lastUpdated,
                    'activity_stats' => Carbon::parse($lastUpdated) > Carbon::now(),
                ],
            ];
            return $this->respondWithItem([], function () use ($result) {
                return $result;
            });
        } catch (Exception $e) {
            return $this->errorWrongArgs($e->getMessage());
        }
    }
}
```

## Detailed explanation of the features used in the example

### 1. **General permission check**
```php
$access = AccessManager::checkByResource('nano.itemsapi::items.list', $user);
```
- Reads the `permission` settings from `config.php` for the specified resource.
- Returns an array containing `allowed`, `scope`, `message`, `applied_scope_details`.
- The result can later be used to apply `applyAccessScope`.

### 2. **Advanced filters filtering**
```php
$advFiltersConfig = AccessManager::getAdvancedFiltersConfig('nano.itemsapi::items.list');
$options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advFiltersConfig]);
```
- Removes any `is_or_*`, `is_not_*`, `is_or_null_*` keys that are not allowed according to the `advanced_filters` configuration in `config.php`.
- Protects against passing dangerous or unwanted filters.

### 3. **Frontend user handling (students and parents)**
```php
if (AccessManager::instance()->getUserType($user) === 'frontend') {
    $options = AccessManager::instance()->resolveFrontendAccessOptions($user, $options);
}
```
- Extracts `student_id` and `record_id` from the user (if a student) or validates the passed data (if a parent).
- Ensures that a parent sees only their own children.

### 4. **Applying the access scope with the ability to control which scopes are enabled**
```php
$applyCompany = Config::get('nano.itemsapi::items.list.permission.backend.apply_company_scope', true);
$applyDepartment = Config::get('...', true);
$applyState = Config::get('...', true);
$applyCreatedBy = Config::get('...', true);

$scopeOptions = [];
if ($applyCompany) $scopeOptions['company_field'] = 'companys_id';
if ($applyDepartment) $scopeOptions['department_field'] = 'departments_id';
if ($applyState) $scopeOptions['state_field'] = 'state_id';
if ($applyCreatedBy) $scopeOptions['created_by_field'] = 'created_by';

$query = AccessManager::instance()->applyAccessScope($query, $access, $scopeOptions);
```
- Certain scopes (e.g., `state` or `created_by`) can be disabled via environment variables or `config.php` settings.
- This provides complete flexibility in multi‑environment setups.

### 5. **Applying `student_id` and `record_id` filters generically**
```php
$query = AccessManager::instance()->applyFrontendStudentParentFilters(
    $query, $user, $options, 'student_id', 'record_id', $table, false
);
```
- A generic method usable on any table that contains `student_id` and `record_id`.
- Automatically adds the table prefix to avoid column name conflicts when `JOIN`ing.
- Can disable automatic resolution (`resolveIfNeeded = false`) if resolution has already been done in `index()`.

### 6. **Using `AdvancedQueryManager::scopeWhereField`**
- Applies a flexible filter that supports `is_or`, `is_not`, `is_force`, `is_or_null`.
- Protects against errors caused by the values `'*'` or `'all'`.

### 7. **Events for extensibility**
- `api.list.extendQueryBefore` and `api.list.extendQuery` allow other plugins to modify the query without changing the core code.

### 8. **Multiple output options**
- `is_query`: returns a `Builder` object.
- `is_first` / `is_model`: returns only the first record.
- `is_collection`: returns a `Collection`.
- `is_paginator`: returns a `Paginator` without transformation.
- Default behaviour: returns a `Paginator` transformed via a `Transformer`.

## Configuration in `config.php` to control which scopes are applied

```php
// config.php
'items' => [
    'list' => [
        'permission' => [
            'backend' => [
                'access_scope' => 'company',
                'apply_company_scope' => true,
                'apply_department_scope' => false,   // do not apply department filter
                'apply_state_scope' => false,        // do not apply state filter
                'apply_created_by_scope' => true,    // apply created_by filter if scope is 'created_by'
            ],
            // ... other settings
        ],
    ],
],
```

Environment variables can also be used:

```env
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_COMPANY_SCOPE=true
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_DEPARTMENT_SCOPE=false
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_STATE_SCOPE=false
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_CREATED_BY_SCOPE=true
```

This example covers all advanced features of `AccessManager` and demonstrates how to use them in a real API controller.
