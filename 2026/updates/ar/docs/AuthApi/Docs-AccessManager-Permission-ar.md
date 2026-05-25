# 📘 دليل إدارة الصلاحيات في AccessManager

**الإصدار:** 1.0.20+  
**المسار:** `Nano\AuthApi\Classes\AccessManager`  
**الغرض:** توثيق كامل لآلية التحكم في الصلاحيات، نطاقات الوصول، الفلاتر المتقدمة، وإعدادات العمليات في نظام Nano.

---

## 1. مقدمة

يوفّر `AccessManager` نظاماً مركزياً وموحداً لإدارة الصلاحيات والوصول إلى موارد API بناءً على:

- **نوع المستخدم**: `backend` (مستخدمي لوحة التحكم)، `frontend` (مستخدمي واجهة المستخدم العادية)، `guest` (زوار غير مسجلين).
- **صلاحيات مخصصة**: لكل عملية (`list`, `create`, `update`, `delete`, `show`, …) يمكن تعريف إعدادات مستقلة.
- **نطاقات الوصول (Access Scopes)**: `all`, `own`, `created_by`, `company`, `department`, `state`, `children`.
- **فلاتر متقدمة**: `is_or`, `is_not`, `is_or_null` مع تحكم دقيق على مستوى الحقل حسب نوع المستخدم.
- **تكامل مع `config.php`**: يتم تعريف جميع الإعدادات في ملف الإعدادات الخاص بكل API plugin، ويمكن تجاوزها عبر متغيرات البيئة.

هذا الدليل يشرح **كيفية تكوين واستخدام نظام الصلاحيات** من خلال `AccessManager` وملف `config.php`.

---

## 2. هيكل إعدادات العملية في `config.php`

لكل عملية (مثل `list`, `create`, `update`, `show`, `delete`) يجب إنشاء مفتاح داخل المورد الخاص بها، ثم داخله مفتاح `permission`. مثال لإعدادات عملية `list` لمورد `absences`:

```php
// config.php
return [
    'absences' => [
        'list' => [
            'permission' => [
                'is_allow' => true,                     // تفعيل العملية (عام)

                'backend'  => [ /* ... */ ],           // إعدادات مستخدمي Backend
                'frontend' => [ /* ... */ ],           // إعدادات مستخدمي Frontend
                'guest'    => [ /* ... */ ],           // إعدادات الزوار (غير مسجلين)

                'advanced_filters' => [ /* ... */ ],   // إعدادات الفلاتر المتقدمة
            ],
        ],
        'create' => [ /* ... */ ],
        'update' => [ /* ... */ ],
        'show'   => [ /* ... */ ],
    ],
];
```

> **ملاحظة:** يمكن تخصيص إعدادات مختلفة لكل عملية. إذا لم توجد إعدادات لعملية معينة، يمكن استخدام آلية الرجوع (fallback) كما هو موضح لاحقاً.

---

## 3. خيارات تكوين كل نوع مستخدم

### 3.1 إعدادات `backend`

```php
'backend' => [
    'allow'              => true,                 // هل يسمح لمستخدمي Backend؟
    'check_permission'   => true,                 // هل نتحقق من الصلاحيات (hasAccess/hasPermission)؟
    'permissions'        => ['tss.example.access'], // صلاحية واحدة أو مصفوفة صلاحيات
    'access_scope'       => 'all',                // نطاق الوصول (انظر القسم 4)
    'company_field'      => 'companys_id',        // حقل الشركة في الجدول
    'department_field'   => 'departments_id',     // حقل القسم
    'state_field'        => 'state_id',           // حقل الولاية
    'created_by_field'   => 'created_by',         // حقل منشئ السجل
    'company_id'         => null,                 // تقييد بشركة معينة (اختياري)
    'department_id'      => null,                 // تقييد بقسم معين (اختياري)
    'state_id'           => null,                 // تقييد بولاية معينة (اختياري)
],
```

#### 3.1.1 نطاقات الوصول المدعومة لـ `backend`

| النطاق (`access_scope`) | الوصف | ملاحظات |
|-------------------------|-------|----------|
| `all` | عرض كل السجلات (بدون فلترة إضافية) | يتطلب صلاحية `access_all` عادةً |
| `own` | عرض السجلات التي أنشأها المستخدم فقط (`created_by`) | يُحوَّل تلقائياً إلى `created_by` |
| `created_by` | نفس `own` | يمكن تحديد اسم الحقل عبر `created_by_field` |
| `company` | عرض سجلات شركة معينة (يُحدد `company_id`) | إذا كان المستخدم يملك صلاحية الوصول لكل الشركات، يصبح النطاق `all` |
| `department` | عرض سجلات قسم معين | إذا كان يملك صلاحية الوصول لكل الأقسام أو كل الشركات، يصبح `all` |
| `state` | عرض سجلات ولاية معينة | إذا كان يملك صلاحية الوصول لكل الولايات أو كل الأقسام، يصبح `all` |

**مثال عملي:**  
مستخدم Backend يملك صلاحية `tss.example.access` فقط، ولا يملك `access_all`. إذا تم تعيين `access_scope = 'company'`، فسيتم فلترة السجلات بحيث `companys_id = معرف شركة المستخدم`. إذا كان المستخدم يملك صلاحية `access_all`، فسيتم تجاهل الفلترة ويعرض كل السجلات.

### 3.2 إعدادات `frontend`

```php
'frontend' => [
    'allow'              => true,                 // هل يسمح لمستخدمي Frontend؟
    'allowed_ref_types'  => ['student', 'parent'], // قيم ref_type المسموحة (كاملة أو مختصرة)
    'access_scope'       => 'own',                // 'own' أو 'children'
    'children_relation'  => 'students',           // اسم العلاقة في نموذج ولي الأمر (لـ 'children')
],
```

#### 3.2.1 نطاقات الوصول المدعومة لـ `frontend`

| النطاق (`access_scope`) | الوصف | متطلبات |
|-------------------------|-------|----------|
| `own` | عرض السجلات الخاصة بالمستخدم نفسه (بناءً على `student_id` أو `user_id`) | يتطلب أن المستخدم مرتبط بطالب (ref_type = student) أو يمتلك `student_id` في الخيارات |
| `children` | عرض سجلات أبناء المستخدم (عندما يكون ولي أمر) | يتطلب تعريف `children_relation` في النموذج المرتبط، وتفعيل `resolveFrontendAccessOptions` |

**مثال عملي:**  
ولي أمر (ref_type = parent) مع `access_scope = 'children'`. عند استدعاء `resolveFrontendAccessOptions`، سيتم جلب أبنائه عبر العلاقة `students`، ثم تطبيق فلتر `student_id IN (...)`.

### 3.3 إعدادات `guest`

```php
'guest' => [
    'allow'         => true,            // السماح للزوار غير المسجلين
    'access_scope'  => 'all',           // 'all' (كل السجلات) أو 'none' (لا شيء)
],
```

> **تحذير:** عند تفعيل `guest.allow = true` لأي عملية، كن حذراً بشأن حساسية البيانات. غالباً ما يُستخدم هذا في عمليات `list` العامة.

---

## 4. نطاقات الوصول (Access Scopes) التفصيلية

### 4.1 `all`
لا يضاف أي شرط `where`. يعرض كل السجلات التي تنطبق عليها باقي الفلاتر.

### 4.2 `own` / `created_by`
يضاف شرط `where($created_by_field, $user->id)`.  
**يُستخدم عادة مع صلاحية `access` (بدون `access_all`).**

### 4.3 `company`
يضاف شرط `where($company_field, $companyId)`.  
**يُستخدم في التطبيقات متعددة الشركات (multi‑tenancy).**

### 4.4 `department`
يضاف شرط `where($department_field, $departmentId)`.

### 4.5 `state`
يضاف شرط `where($state_field, $stateId)` أو `whereIn($state_field, $stateIds)`.

### 4.6 `children` (خاص بـ frontend)
يضاف شرط `whereIn($studentIdField, $childrenIds)` بعد جلب أبناء المستخدم من العلاقة المحددة.

---

## 5. التحكم في الفلاتر المتقدمة (`advanced_filters`)

تسمح الفلاتر المتقدمة بتمرير خيارات مثل `is_or`, `is_not`, `is_or_null` للحقول. يمكن التحكم في الحقول المسموح بها لكل نوع مستخدم.

### 5.1 هيكل الإعدادات

```php
'advanced_filters' => [
    'enabled'                => true,                     // تفعيل/تعطيل الكل
    'default_for_backend'    => true,                     // السماح افتراضياً لـ backend
    'default_for_frontend'   => false,                    // منع افتراضياً لـ frontend
    'allowed_fields_backend' => ['*'],                    // كل الحقول مسموحة لـ backend
    'allowed_fields_frontend' => [],                      // لا شيء لـ frontend
    'field_specific_rules'   => [                         // قواعد خاصة بالحقول
        'student_id' => [
            'backend' => true,
            'frontend' => [
                '*' => false,                             // عام لجميع frontend
                'parent' => true,                         // يسمح لولي الأمر
            ],
        ],
        'record_id' => [ /* ... */ ],
    ],
],
```

### 5.2 كيف تعمل؟

- إذا كان `enabled = false`، يتم حذف كافة مفاتيح `is_or_*`, `is_not_*`, `is_or_null_*` من الخيارات.
- يتم فحص كل حقل على حدة:  
  1. إذا كان هناك قاعدة خاصة للحقل (`field_specific_rules`)، تُستخدم.
  2. وإلا، يُستخدم `default_for_backend` / `default_for_frontend` مع قائمة الحقول المسموحة.
- في حالة `frontend`، يمكن تخصيص السماح حسب `ref_type` باستخدام مصفوفة `'frontend' => [ 'ref_type' => true/false, '*' => true/false ]`.

**مثال:** يسمح لولي أمر (parent) باستخدام `is_or_student_id` ولكن لا يسمح لطالب (student).

---

## 6. آلية الرجوع (Fallback) للعمليات غير المحددة

إذا لم توجد إعدادات لعملية معينة (مثل `show`)، يمكن استخدام إعدادات عملية أخرى (مثل `list`) كقاعدة ثم دمجها مع أي إعدادات جزئية موجودة.

**دالة `checkWithFallback` في `AccessManager`:**

```php
AccessManager::checkWithFallback(
    'nano.absenceapi::absences.show',   // المورد الأساسي
    'nano.absenceapi::absences.list',   // المورد الاحتياطي
    $user
);
```

- إذا كان `show.permission.is_allow` موجوداً، يُستخدم مباشرة.
- وإلا، يتم دمج `list.permission` مع `show.permission` (أي إعدادات `show` تتجاوز `list`).
- ثم يتم التحقق كالمعتاد.

---

## 7. دوال جلب الإعدادات والتحقق

### 7.1 دوال مبسطة (موصى بها)

| الدالة | الوصف | مثال |
|--------|-------|------|
| `checkByResource($resourceKey, $user, $overrideConfig)` | يتحقق من الصلاحية باستخدام مفتاح المورد الكامل | `AccessManager::checkByResource('nano.absenceapi::absences.list', $user)` |
| `getOperationByConfig($resourceKey, $overrideConfig)` | يعيد إعدادات العملية بدون تنفيذ التحقق | `$config = AccessManager::getOperationByConfig('nano.absenceapi::absences.list');` |
| `getAdvancedFiltersConfig($resourceKey)` | يعيد فقط إعدادات `advanced_filters` | `$adv = AccessManager::getAdvancedFiltersConfig('nano.absenceapi::absences.list');` |
| `checkWithFallback($resourceKey, $fallbackKey, $user, $overrideConfig)` | تحقق مع الرجوع إلى عملية أخرى | `AccessManager::checkWithFallback('...show', '...list', $user)` |

### 7.2 دوال متقدمة (أقل استعمالاً)

- `checkWithConfig($operation, $plugin, $resource, $user, $overrideConfig)`
- `getOperationConfig($plugin, $resource, $operation, $overrideConfig)`

---

## 8. أمثلة عملية كاملة

### 8.1 السماح لمستخدمي Backend فقط بعملية `delete` (مع صلاحية خاصة)

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

### 8.2 جعل عملية `list` عامة للجميع (بما في ذلك الضيوف) مع فلترة بسيطة

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

### 8.3 السماح لولي الأمر باستخدام `is_or_student_id` فقط

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

### 8.4 استخدام `checkByResource` في متحكم `Absences`

```php
public function index()
{
    $user = $this->current_user;
    $access = AccessManager::checkByResource('nano.absenceapi::absences.list', $user);
    if (!$access['allowed']) {
        return $this->errorUnauthorized($access['message']);
    }
    // ... الباقي
}
```

---

## 9. ملخص الخيارات والإعدادات

| المفتاح | المستوى | القيم الممكنة | الشرح |
|---------|---------|---------------|-------|
| `is_allow` | عام | true/false | تفعيل/تعطيل العملية بالكامل |
| `backend.allow` | backend | true/false | السماح لمستخدمي Backend |
| `backend.check_permission` | backend | true/false | تفعيل فحص الصلاحيات |
| `backend.permissions` | backend | array | قائمة الصلاحيات المطلوبة (مصفوفة أو نص مفصول بفواصل) |
| `backend.access_scope` | backend | all, own, created_by, company, department, state | نطاق الوصول |
| `backend.company_field` | backend | string | اسم حقل الشركة في الجدول |
| `backend.department_field` | backend | string | اسم حقل القسم |
| `backend.state_field` | backend | string | اسم حقل الولاية |
| `backend.created_by_field` | backend | string | اسم حقل منشئ السجل |
| `backend.company_id` | backend | int/string/null | تحديد شركة معينة (تتجاوز شركة المستخدم) |
| `frontend.allow` | frontend | true/false | السماح لمستخدمي Frontend |
| `frontend.allowed_ref_types` | frontend | array | أنواع المستخدمين المسموحة (كاملة أو مختصرة) |
| `frontend.access_scope` | frontend | own, children | نطاق الوصول |
| `frontend.children_relation` | frontend | string | اسم العلاقة لجلب الأبناء (لـ children) |
| `guest.allow` | guest | true/false | السماح للزوار غير المسجلين |
| `guest.access_scope` | guest | all, none | نطاق الوصول للضيوف |
| `advanced_filters.enabled` | advanced_filters | true/false | تفعيل الفلاتر المتقدمة |
| `advanced_filters.default_for_backend` | advanced_filters | true/false | السماح الافتراضي لـ backend |
| `advanced_filters.default_for_frontend` | advanced_filters | true/false | السماح الافتراضي لـ frontend |
| `advanced_filters.allowed_fields_backend` | advanced_filters | array | قائمة الحقول المسموحة لـ backend (['*'] للكل) |
| `advanced_filters.allowed_fields_frontend` | advanced_filters | array | قائمة الحقول المسموحة لـ frontend |
| `advanced_filters.field_specific_rules` | advanced_filters | array | قواعد لكل حقل |

---

## 10. نصائح وتوصيات

- **استخدم `checkByResource`** بدلاً من `check` المباشر لتقليل الكود والاعتماد على `config.php`.
- **لعمليات `show`**، يمكنك إما تخصيص إعدادات خاصة بها أو استخدام `checkWithFallback` للرجوع إلى `list`.
- **للفلاتر المتقدمة**، كن دقيقاً في تحديد `allowed_fields_frontend` و `field_specific_rules` لمنع تسريب البيانات.
- **اختبار الصلاحيات**: يمكنك استخدام `AccessManager::instance()->setUser($fakeUser)` لمحاكاة مستخدم معين في بيئة الاختبار.
- **استخدم متغيرات البيئة** في ملف `.env` لتجاوز القيم الافتراضية للإعدادات دون تعديل `config.php`.

---

## 11. الخاتمة

نظام الصلاحيات في `AccessManager` يوفر مرونة عالية تمكنك من التحكم في الوصول إلى كل نقطة نهاية API بدقة، بناءً على نوع المستخدم ونطاق الوصول والصلاحيات التقليدية. من خلال توحيد الإعدادات في ملف `config.php` واستخدام الدوال المبسطة، يصبح من السهل صيانة وتوسيع نظام الصلاحيات لمشاريع Nano المختلفة.

**المراجع:**

- [توثيق كلاس `AccessManager`](./Docs-AccessManager-ar.md)
- [توثيق الصلاحيات `AccessManager-Permission`](./Docs-AccessManager-Permission-ar.md)
- [امثلة عامة `AccessManager-Example`](./Docs-AccessManager-Example-ar.md)
- [مثال عملى شامل  `AccessManager-Example-Api`](./Docs-AccessManager-Example-Api-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)
- [متحكم `Absences` كنموذج تطبيقي](../AbsenceApi/APIControllers/Absences.php)