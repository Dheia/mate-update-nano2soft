## 13. مثال كامل لمتحكم API (العمليات list, show, create, update)


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
 * مثال متكامل لمتحكم API يستخدم كافة ميزات AccessManager.
 * 
 * @package Nano\ItemsApi\APIControllers
 */
class Items extends ApiController
{
    /**
     * تهيئة المتحكم: جلب المستخدم الحالي.
     */
    public function init()
    {
        $this->current_user = $this->getUser() ?? AuthHelpers::getCurrentUser();
    }

    // ======================= LIST ENDPOINT =======================

    /**
     * جلب قائمة العناصر مع دعم كامل للصلاحيات والنطاقات والفلاتر المتقدمة.
     *
     * @return mixed
     */
    public function index()
    {
        try {
            $this->init();
            $user = $this->current_user;

            // -----------------------------------------------------------------
            // 1. التحقق من الصلاحية باستخدام المفتاح المبسط من config
            // -----------------------------------------------------------------
            $access = AccessManager::checkByResource('nano.itemsapi::items.list', $user);
            if (!$access['allowed']) {
                return $this->errorUnauthorized($access['message']);
            }

            // -----------------------------------------------------------------
            // 2. تجهيز خيارات الطلب
            // -----------------------------------------------------------------
            $options = (array) $this->data;
            $options['access_result'] = $access;

            // -----------------------------------------------------------------
            // 3. تطبيق قيود الفلاتر المتقدمة (is_or / is_not / is_or_null)
            //    - نقرأ الإعدادات من config (نفس المورد)
            //    - نقوم بتصفية الخيارات (حذف الفلاتر غير المسموحة)
            // -----------------------------------------------------------------
            $advFiltersConfig = AccessManager::getAdvancedFiltersConfig('nano.itemsapi::items.list');
            $options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advFiltersConfig]);

            // -----------------------------------------------------------------
            // 4. معالجة خاصة لمستخدمي Frontend (طلاب وأولياء أمور)

            // نستخدم المحلل الديناميكي:
            $options = AccessManager::instance()->resolveDynamicFrontendOptions(
                $user,
                $options,
                null,  // لا نمرر query هنا، سنطبق الفلاتر لاحقاً في getRecords
                'nano.absenceapi::absences.list'
            );
            //or
            /*
            //    - تقوم resolveFrontendAccessOptions بحل student_id, record_id,
            //      والتحقق من صلاحية ولي الأمر لأبنائه.
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
            // 5. تخزين مؤقت (اختياري)
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
     * الدالة الأساسية لبناء وتنفيذ استعلام جلب العناصر.
     *
     * @param array $options
     * @return mixed
     */
    public function getRecords(array $options = [])
    {
        // ======================= 1. دمج الخيارات الافتراضية =======================
        $defaults = [
            // فلاتر أساسية
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

            // بحث نصي
            'q' => null,
            'is_advanced_search' => null,

            // علاقات واستبعاد
            'exclude' => null,
            'custom_with' => null,
            'with_count' => false,

            // ترتيب وتجميع
            'orderBy' => null,
            'orderDirection' => 'asc',
            'is_not_sort' => false,
            'group_by' => null,
            'having' => null,

            // ترقيم
            'page' => Input::get('page', 1),
            'per_page' => null,
            'take_limet' => null,

            // نوع الإخراج
            'is_collection' => false,
            'is_paginator' => false,
            'is_query' => false,
            'is_first' => false,
            'is_model' => false,
            'is_to_sql' => false,

            // نتيجة الصلاحيات
            'access_result' => null,
        ];

        $options = array_merge($defaults, $options);

        // إعدادات افتراضية من config
        $exclude = $options['exclude'] ?? Config::get('nano.itemsapi::items.exclude', '');
        $orderBy = $options['orderBy'] ?? Config::get('nano.itemsapi::items.order_by', 'sort_order');
        $orderDirection = $options['orderDirection'] ?? Config::get('nano.itemsapi::items.order_dir', 'asc');
        $per_page = $options['per_page'] ?? Config::get('nano.itemsapi::items.per_page', 15);

        // ======================= 2. بناء الاستعلام الأساسي =======================
        $model = new Item();
        $table = $model->getTable();
        $query = Item::query();

        // استبعاد الأعمدة (exclude)
        if (!empty($exclude) && (method_exists($query, 'scopeExclude') || method_exists($query, 'exclude'))) {
            $columns = is_array($exclude) ? $exclude : explode(',', $exclude);
            $query = $query->exclude($columns);
        }

        // تحميل العلاقات المطلوبة
        if (!empty($options['custom_with'])) {
            $with = is_array($options['custom_with']) ? $options['custom_with'] : explode(',', $options['custom_with']);
            $query = $query->with($with);
        }

        // ======================= 3. تطبيق نطاق الصلاحيات (Access Scope) =======================
        // يمكن التحكم في النطاقات المطبقة من خلال الإعدادات في config.php
        // مثلاً في config.php نضع:
        // 'backend' => [
        //     'access_scope' => 'company',          // النطاق الأساسي
        //     'apply_company_scope' => true,        // هل نطبق شرط الشركة؟
        //     'apply_department_scope' => false,    // تعطيل فلترة القسم
        //     'apply_state_scope' => false,         // تعطيل فلترة الولاية
        //     'apply_created_by_scope' => true,     // تطبيق فلترة منشئ السجل إذا كان النطاق 'created_by'
        // ]
        if (!empty($options['access_result']) && $options['access_result']['allowed']) {
            // نقرأ إعدادات التحكم من config (يمكن تخزينها داخل access_result أو قراءتها مباشرة)
            $applyCompany = Config::get('nano.itemsapi::items.list.permission.backend.apply_company_scope', true);
            $applyDepartment = Config::get('nano.itemsapi::items.list.permission.backend.apply_department_scope', true);
            $applyState = Config::get('nano.itemsapi::items.list.permission.backend.apply_state_scope', true);
            $applyCreatedBy = Config::get('nano.itemsapi::items.list.permission.backend.apply_created_by_scope', true);

            // بناء مصفوفة الخيارات الديناميكية بناءً على الإعدادات
            $scopeOptions = [];
            if ($applyCompany) $scopeOptions['company_field'] = 'companys_id';
            if ($applyDepartment) $scopeOptions['department_field'] = 'departments_id';
            if ($applyState) $scopeOptions['state_field'] = 'state_id';
            if ($applyCreatedBy) $scopeOptions['created_by_field'] = 'created_by';

            $query = AccessManager::instance()->applyAccessScope($query, $options['access_result'], $scopeOptions);
        }

        // ======================= 4. تطبيق فلاتر الطالب/ولي الأمر (اختياري، يفيد في جداول الغيابات والنتائج) =======================
        // تطبيق المحلل الديناميكي مع تعديل الاستعلام مباشرة (بدون تخزين النتائج)
        $options = AccessManager::instance()->resolveDynamicFrontendOptions(
            $user,
            $options,
            $query,
            'nano.absenceapi::absences.list'
        );
        
        //or
        /*
        // نمرر $table لضمان إضافة بادئة الجدول للحقول (لتجنب التعارض عند JOIN)
        $query = AccessManager::instance()->applyFrontendStudentParentFilters(
            $query,
            $user ?? null,
            $options,
            'student_id',          // اسم عمود student_id في الجدول الحالي
            'record_id',           // اسم عمود record_id
            $table,                // بادئة الجدول (تُضاف تلقائياً للحقول)
            false                  // resolveIfNeeded = false لأننا قمنا بالحل مسبقاً في index()
        );
        */
        //or
        /*
        $query = AccessManager::instance()->applyFrontendStudentParentFilters(
            $query,
            $user,
            $options,
            'student_id',     // اسم العمود الأساسي (بدون بادئة)
            'record_id',      // اسم العمود الأساسي
            $table,           // اسم الجدول (مثلاً 'absences') – سيتم إضافته تلقائياً
            true              // يحل الخيارات تلقائياً إذا لزم الأمر
        )
        */

        // ======================= 5. تطبيق الفلاتر العادية باستخدام AdvancedQueryManager =======================
        // فلتر id
        if ($options['id'] || $options['is_force_id']) {
            $query = AdvancedQueryManager::scopeWhereField(
                $query, 'id', $options['id'],
                $options['is_or_id'], $table,
                $options['is_not_id'], $options['is_force_id']
            );
        }

        // فلتر student_id (إذا لم نستخدم applyFrontendStudentParentFilters أعلاه، يمكن تطبيقه هنا)
        if ($options['student_id'] !== null || $options['is_force_student_id']) {
            $query = AdvancedQueryManager::scopeWhereField(
                $query, 'student_id', $options['student_id'],
                $options['is_or_student_id'], $table,
                $options['is_not_student_id'], $options['is_force_student_id']
            );
        }

        // فلتر سنة دراسية (مع تعبئة تلقائية من السنة الافتراضية)
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

        // فلتر status
        if ($options['status'] && $options['status'] !== '*') {
            $query = AdvancedQueryManager::scopeWhereField(
                $query, 'status', $options['status'],
                $options['is_or_status'], $table,
                $options['is_not_status'], $options['is_force_status']
            );
        }

        // فلتر is_active
        if ($options['is_active'] !== null && $options['is_active'] !== '*') {
            $options['is_active'] ? $query->isActive() : $query->isNotActive();
        }

        // البحث النصي (q)
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

        // ======================= 6. أحداث (Events) قابلة للتوسع =======================
        Event::fire('api.list.extendQueryBefore', [$query, $this, $model, $options]);

        // Group by (يدعم مصفوفة)
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

        // ترتيب
        if (!$options['is_not_sort'] && $orderBy && $orderDirection) {
            $query->orderBy(\DB::raw($orderBy), \DB::raw($orderDirection));
        }

        Event::fire('api.list.extendQuery', [$query, $this, $model, $options]);

        // ======================= 7. حدود وإخراج SQL للتصحيح =======================
        if ($options['take_limet']) {
            $query->take((int)$options['take_limet']);
        }
        if ($options['is_to_sql']) {
            $sql = vsprintf(str_replace('?', '"%s"', $query->toSql()), $query->getBindings());
            trace_log($sql);
        }

        // ======================= 8. تنفيذ الاستعلام حسب نوع الإخراج =======================
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

            // استخدام fallback: إذا لم توجد إعدادات show, نستخدم إعدادات list
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

            // تطبيق معالجة frontend إن لزم
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
            
            // التحقق الإضافي: إذا كان المستخدم من نوع ولي أمر، نتأكد أن السجل يخص أحد أبنائه (ضمان أمني إضافي)
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
                // تعبئة القيم الافتراضية إن وجدت
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

## شرح تفصيلي للميزات المستخدمة في المثال

### 1. **التحقق العام من الصلاحية**
```php
$access = AccessManager::checkByResource('nano.itemsapi::items.list', $user);
```
- يقوم بقراءة إعدادات `permission` من `config.php` للمورد المحدد.
- يعيد مصفوفة تحتوي على `allowed`, `scope`, `message`, `applied_scope_details`.
- يمكن استخدام النتيجة لاحقاً في تطبيق `applyAccessScope`.

### 2. **تصفية الفلاتر المتقدمة (Advanced Filters)**
```php
$advFiltersConfig = AccessManager::getAdvancedFiltersConfig('nano.itemsapi::items.list');
$options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advFiltersConfig]);
```
- يزيل أي مفاتيح `is_or_*`, `is_not_*`, `is_or_null_*` غير المسموحة وفق إعدادات `advanced_filters` في `config.php`.
- يُحمي من تمرير فلاتر خطيرة أو غير مرغوب فيها.

### 3. **معالجة مستخدمي Frontend (طلاب وأولياء أمور)**
```php
if (AccessManager::instance()->getUserType($user) === 'frontend') {
    $options = AccessManager::instance()->resolveFrontendAccessOptions($user, $options);
}
```
- يستخرج `student_id` و `record_id` من المستخدم (إذا كان طالباً) أو يتحقق من صحة البيانات الممررة (إذا كان ولي أمر).
- يضمن أن ولي الأمر يرى فقط أبناءه.

### 4. **تطبيق نطاق الوصول (Access Scope) مع إمكانية التحكم في تفعيل النطاقات**
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
- يمكن تعطيل نطاقات معينة (مثل `state` أو `created_by`) عن طريق متغيرات البيئة أو إعدادات `config.php`.
- هذا يمنح مرونة كاملة في بيئات متعددة.

### 5. **تطبيق فلاتر `student_id` و `record_id` بشكل عام**
```php
$query = AccessManager::instance()->applyFrontendStudentParentFilters(
    $query, $user, $options, 'student_id', 'record_id', $table, false
);
```
- دالة عامة تستخدم في أي جدول يحتوي على `student_id` و `record_id`.
- تضيف بادئة الجدول تلقائياً لمنع تضارب الأسماء عند `JOIN`.
- يمكن تعطيل الحل التلقائي (`resolveIfNeeded = false`) إذا سبق أن قمنا بالحل في `index()`.

### 6. **استخدام `AdvancedQueryManager::scopeWhereField`**
- يطبق فلتراً مرناً يدعم `is_or`, `is_not`, `is_force`, `is_or_null`.
- يحمي من الأخطاء الناتجة عن القيم `'*'` أو `'all'`.

### 7. **أحداث (Events) للتوسع**
- `api.list.extendQueryBefore` و `api.list.extendQuery` تسمح للإضافات الأخرى بتعديل الاستعلام دون تعديل الكود الأساسي.

### 8. **خيارات إخراج متعددة**
- `is_query`: يعيد كائن `Builder`.
- `is_first` / `is_model`: يعيد أول سجل فقط.
- `is_collection`: يعيد `Collection`.
- `is_paginator`: يعيد `Paginator` بدون تحويل.
- الوضع الافتراضي: يعيد `Paginator` مع تحويل عبر `Transformer`.

## إعدادات `config.php` للتحكم في النطاقات المطبقة

```php
// config.php
'items' => [
    'list' => [
        'permission' => [
            'backend' => [
                'access_scope' => 'company',
                'apply_company_scope' => true,
                'apply_department_scope' => false,   // لا نطبق فلترة القسم
                'apply_state_scope' => false,        // لا نطبق فلترة الولاية
                'apply_created_by_scope' => true,    // نطبق فلترة created_by إذا كان النطاق 'created_by'
            ],
            // ... باقي الإعدادات
        ],
    ],
],
```

يمكن أيضاً استخدام متغيرات البيئة:

```env
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_COMPANY_SCOPE=true
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_DEPARTMENT_SCOPE=false
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_STATE_SCOPE=false
NANO_ITEMSAPI_ITEMS_LIST_BACKEND_APPLY_CREATED_BY_SCOPE=true
```

بهذا الشكل، يغطي المثال كافة الميزات المتقدمة لـ `AccessManager` ويوضح كيفية استخدامها في متحكم API حقيقي.