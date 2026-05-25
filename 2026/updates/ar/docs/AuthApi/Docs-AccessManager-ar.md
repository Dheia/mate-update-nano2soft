# 📘 توثيق كلاس `AccessManager`

**الإصدار:** 1.0.20  
**المسار:** `Nano\AuthApi\Classes\AccessManager`  
**الغرض:** إدارة صلاحيات المستخدمين والتحكم في نطاق الوصول (Access Scopes) بشكل مركزي ومتقدم.

---

## 1. مقدمة

كلاس `AccessManager` هو حل موحد لإدارة عمليات التحقق من الصلاحيات في نظام Nano البيئي. يغطي الكلاس ثلاثة أنواع من المستخدمين:

- **Backend** – مستخدمي لوحة التحكم (Nano Soft App `Backend\Models\User`).
- **Frontend** – مستخدمي الواجهة الأمامية (RainLab\User\Models\User أو أي نموذج مخصص).
- **Guest** – المستخدمون غير المسجلين.

بالإضافة إلى دعم **نطاقات الوصول المتقدمة** مثل:

- `all` – الوصول إلى كل السجلات.
- `own` / `created_by` – الوصول إلى السجلات التي أنشأها المستخدم فقط.
- `company` – الوصول إلى سجلات شركة محددة.
- `department` – الوصول إلى سجلات قسم معين.
- `state` – الوصول إلى سجلات ولاية معينة.
- `children` – لمستخدمي Frontend من نوع **ولي أمر**، لعرض سجلات أبنائه.

يستفيد `AccessManager` من دوال `BasicHelper` المتاحة (مثل `checkAccessAllCompanys`, `checkAccessAllDepartments`) ويتكامل بسلاسة مع `AdvancedQueryHelper` لبناء استعلامات Eloquent مقيدة بالصلاحيات.

---

## 2. فوائد الكلاس واستخداماته

| الفائدة | الوصف |
|---------|-------|
| **مركزية الصلاحيات** | بدلاً من تكرار منطق التحقق في كل متحكم API، يمكن استدعاء `AccessManager::check()` في مكان واحد. |
| **مرونة عالية** | يدعم أنواعاً متعددة من المستخدمين ونطاقات وصول مختلفة، مع إمكانية تخصيص أسماء الحقول. |
| **قابلية التوسع** | يمكن إضافة نطاقات جديدة (مثل `team`, `branch`) دون تعديل الكود الأساسي. |
| **تكامل مع `AdvancedQueryHelper`** | دوال `applyAccessScope` تُطبق شروط الوصول مباشرة على استعلام Eloquent، مما يوفر أماناً وفعالية. |
| **دعم متعدد المستويات** | يفحص أولاً الإعدادات العامة (is_allow)، ثم صلاحيات النوع (backend/frontend/guest)، ثم النطاق المطلوب، ثم الصلاحيات النهائية. |
| **إرجاع معلومات تفصيلية** | تعيد `check()` مصفوفة تحتوي على `allowed`, `scope`, `message`, `applied_scope_details` لاستخدامها في الطبقات العليا. |

**الاستخدامات النموذجية:**

- حماية نقاط نهاية API (قوائم، إنشاء، تحديث، حذف).
- تحديد ما إذا كان المستخدم يرى فقط سجلات فرعه أو شركته.
- السماح لولي الأمر برؤية سجلات أبنائه.
- تعطيل عمليات معينة لمستخدمي Guest بشكل كامل.

---

## 3. دوال الكلاس وخصائصها

### 3.1 دوال الإعداد والجلب

| الدالة | الوصف |
|--------|-------|
| `setUser($user)` | يعين المستخدم يدوياً (يتجاوز الجلب التلقائي). |
| `getUser()` | يعيد المستخدم الحالي (يجلبه تلقائياً إذا لم يُعين). |
| `clearUser()` | يعيد تعيين المستخدم المخبأ. |
| `getUserType($user = null)` | يعيد `'backend'`, `'frontend'`, `'guest'`. |
| `getUserRefType($user = null)` | يستخرج `ref_type` من المستخدم. |
| `getUserRefId($user = null)` | يستخرج `ref_id` من المستخدم. |
| `getUserCompanyId($user = null)` | يستخرج `companys_id` (إن وجد). |
| `getUserDepartmentId($user = null)` | يستخرج `departments_id` (إن وجد). |

### 3.2 دالة التحقق الأساسية

#### `check(string $operation, array $config = [], $user = null): array`

**المعاملات:**

- `$operation`: اسم العملية (`'list'`, `'create'`, `'update'`, `'delete'`, `'view'`…).
- `$config`: مصفوفة إعدادات العملية (انظر قسم الإعدادات لاحقاً).
- `$user`: كائن المستخدم (اختياري، يُجلب تلقائياً إذا لم يُمرر).

**قيمة العودة:**

```php
[
    'allowed' => bool,                // هل العملية مسموحة؟
    'scope'   => string,              // النطاق الفعلي (all, own, created_by, company, department, state, children, none)
    'message' => string,              // رسالة توضيحية
    'user_type' => string,            // backend / frontend / guest
    'user'    => mixed,               // كائن المستخدم المستخدم في التحقق
    'config'  => array,               // الإعدادات الأصلية
    'applied_scope_details' => array  // تفاصيل النطاق المطبق (مثل company_id, department_id, state_id, user_id)
]
```

### 3.3 دوال التحقق من صلاحيات Backend

| الدالة | الوصف |
|--------|-------|
| `canAccessAllCompanies($user = null)` | هل يمكنه الوصول إلى كل الشركات؟ |
| `canAccessAllDepartments($user = null)` | هل يمكنه الوصول إلى كل الأقسام؟ |
| `canAccessAllStates($user = null)` | هل يمكنه الوصول إلى كل الولايات؟ |
| `canAccessDepartment($departmentId, $user = null)` | هل يمكنه الوصول إلى قسم معين؟ |

### 3.4 دوال تطبيق النطاق على Query Builder

| الدالة | الوصف |
|--------|-------|
| `applyAccessScope($query, array $accessResult, array $options = [])` | يطبق النطاق العام بناءً على نتيجة `check()`. |
| `applyCompanyScope($query, $companyId = null, string $field = 'companys_id')` | يضيف `where` على حقل الشركة. |
| `applyDepartmentScope($query, $departmentId = null, string $field = 'departments_id')` | يضيف `where` على حقل القسم. |
| `applyStateScope($query, $stateId = null, string $field = 'state_id')` | يضيف `where` على حقل الولاية (يدعم مصفوفة). |
| `applyCreatedByScope($query, $userId = null, string $field = 'created_by')` | يضيف `where` على حقل منشئ السجل. |
| `applyUserScope($query, $user = null, string $userIdField = 'user_id', string $userTypeField = 'user_type')` | يضيف `where` على `user_id` و `user_type`. |
| `applyChildrenScope($query, $parent, string $studentIdField = 'student_id', string $relation = 'students')` | يضيف `whereIn` على معرفات الأبناء لولي أمر معين. |

### 3.5 دوال مساعدة إضافية

- **`getPermissionArray(string $operation, array $config = []): array`**  
  تعيد مصفوفة شبيهة بتلك المستخدمة في `OrderHelper::getArrayDataAfterPermission`، لتسهيل التكامل مع الكود القديم.

- **`make(): self`** – تنشئ كائن جديد (للاستخدام غير المفرد).
- **`instance(): self`** – تعيد الكائن المفرد (Singleton).

---

## 4. إعدادات العملية (`$config`)

مصفوفة `$config` هي قلب `AccessManager`. يجب أن تحتوي على المفاتيح التالية (مع إمكانية تجاهل بعضها حسب الحاجة):

```php
$config = [
    // الإعدادات العامة
    'is_allow' => true,   // إذا false، يتم رفض العملية فوراً

    // إعدادات ضيف (اختياري)
    'guest' => [
        'allow' => false,
        'access_scope' => 'none',
    ],

    // إعدادات Backend
    'backend' => [
        'allow'              => true,
        'check_permission'   => true,
        'permissions'        => ['tss.example.access'],   // يمكن أن تكون نصاً مفصولاً بفواصل أو مصفوفة
        'access_scope'       => 'all',                    // all, own, created_by, company, department, state
        'company_field'      => 'companys_id',
        'department_field'   => 'departments_id',
        'state_field'        => 'state_id',
        'created_by_field'   => 'created_by',
        'company_id'         => null,                     // رقم شركة محدد (اختياري)
        'department_id'      => null,
        'state_id'           => null,
    ],

    // إعدادات Frontend
    'frontend' => [
        'allow'              => true,
        'allowed_ref_types'  => ['student', 'parent'],    // قيم ref_type المسموحة
        'access_scope'       => 'own',                    // own أو children
        'children_relation'  => 'students',               // اسم العلاقة لجلب الأبناء (يُطلب إذا access_scope = children)
    ],

    // النطاق الافتراضي (يُستخدم إذا لم يُحدد في النوع المناسب)
    'default_access_scope' => 'none',
];
```

**ملاحظات هامة:**

- إذا تم تعيين `backend.access_scope` إلى `company`, `department`, أو `state`، سيتحقق الكلاس أولاً من صلاحية `canAccessAll*`. إذا كانت `true`، يصبح النطاق الفعلي `all`؛ وإلا يستخدم المعرف المحدد.
- `frontend.allowed_ref_types` يمكن أن يحتوي على أسماء كلاسات كاملة (مثل `Tss\Student\Models\Student`) أو أسماء مختصرة (`student`). يتم المطابقة تلقائياً.

---

## 5. أمثلة متكاملة ومفصلة

### 5.1 التحقق الأساسي لمستخدم Backend (نطاق `all`)

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

### 5.2 نطاق `company` لمستخدم Backend (الوصول إلى شركة واحدة)

```php
$config = [
    'backend' => [
        'allow' => true,
        'access_scope' => 'company',
        'company_field' => 'companys_id',
        // 'company_id' => 2, // إذا لم يُمرر، يُستخدم company_id الخاص بالمستخدم
    ],
];

$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed'] && $access['scope'] === 'company') {
    $query = Order::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access);
    // يضاف الشرط: companions_id = (معرف الشركة)
}
```

### 5.3 نطاق `children` لمستخدم Frontend من نوع ولي أمر

```php
$config = [
    'frontend' => [
        'allow' => true,
        'allowed_ref_types' => ['parent', 'Tss\Student\Models\Mparent'],
        'access_scope' => 'children',
        'children_relation' => 'students', // العلاقة في نموذج Mparent
    ],
];

$user = AuthHelpers::getCurrentUser(); // مستخدم Frontend مرتبط بـ Mparent
$access = AccessManager::instance()->check('list', $config, $user);

if ($access['allowed'] && $access['scope'] === 'children') {
    // يجب تطبيق النطاق يدوياً أو استخدام applyAccessScope ثم applyChildrenScope
    $query = StudentRecord::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access);
    // الآن يمكن استدعاء applyChildrenScope إذا لم تُطبق تلقائياً (اختياري)
    $parent = StudentHelper::getMparentByUser($user);
    $query = AccessManager::instance()->applyChildrenScope($query, $parent, 'student_id', 'students');
}
```

### 5.4 استخدام `getPermissionArray` للحصول على مصفوفة إعدادات مشابهة لـ `OrderHelper`

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
if ($perms['allowed']) {
    if ($perms['check_departments']) {
        // في دوال listExtendQuery القديمة
        $query->where('departments_id', $perms['departments_id']);
    }
    if ($perms['check_state']) {
        $query->where('state_id', $perms['state_id']);
    }
}
```

### 5.5 دمج `AccessManager` مع `AdvancedQueryHelper` في متحكم API كامل

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

    // استخدام StudentRecordsHelper الذي يطبق النطاق تلقائياً
    $result = StudentRecordsHelper::getRecords(new Student(), $options);
    return $result['data']; // أو تحويل حسب الحاجة
}
```

### 5.6 نطاق `created_by` لعرض سجلات المستخدم فقط

```php
$config = [
    'backend' => [
        'allow' => true,
        'access_scope' => 'created_by',
        'created_by_field' => 'created_by',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
// إذا كان المستخدم يملك صلاحيات access_all، سيكون النطاق 'all'؛ وإلا 'created_by'
```

### 5.7 تعطيل العملية بالكامل لجميع المستخدمين

```php
$config = [
    'is_allow' => false,   // يلغي أي صلاحيات أخرى
];
$access = AccessManager::instance()->check('delete', $config);
// $access['allowed'] === false, $access['message'] يوضح أن العملية معطلة.
```

---

## 6. الخلاصة

| الميزة | التفاصيل |
|--------|----------|
| **سهولة الاستخدام** | دالة `check()` واحدة تغطي معظم السيناريوهات. |
| **مرونة الإعدادات** | يمكن تخصيص كل عملية (`list`, `create`, `update`, `delete`) على حدة. |
| **دعم متعدد البيئات** | يعمل مع Nano Soft App Backend و RainLab.User Frontend وأي نماذج مخصصة. |
| **تكامل مع `BasicHelper`** | يستفيد من دوال `checkAccessAllCompanys` وغيرها للحفاظ على الاتساق. |
| **تطبيق تلقائي للنطاق** | دوال `applyAccessScope` تبسط إضافة شروط `where` إلى استعلامات Eloquent. |
| **صديق للاختبارات** | يمكن حقن مستخدم وهمي بسهولة باستخدام `setUser()`. |

**سيناريوهات الاستخدام الشائعة:**

- حماية قوائم الطلاب بحيث يرى كل موظف فقط طلاب فرعه.
- السماح لولي الأمر برؤية سجلات أبنائه دون غيره.
- عرض التقارير المالية للمستخدمين المصرح لهم فقط (عبر `created_by`).
- تقييد عمليات الحذف على المستخدمين الذين يمتلكون صلاحية خاصة.

---

## 7. الخاتمة

يمثل كلاس `AccessManager` حجر الزاوية لأي نظام API آمن في بيئة Nano. بفضل تصميمه المرن والمتكامل، يمكن للمطورين التركيز على منطق الأعمال مع ضمان أن الصلاحيات تُطبق بشكل موحد وسهل الصيانة. نوصي بشدة باستخدام هذا الكلاس في جميع إضافات `Nano.*Api` المستقبلية، وتحديث الإضافات القديمة لتستفيد منه.

**المراجع:**

- [توثيق الصلاحيات `AccessManager-Permission`](./Docs-AccessManager-Permission-ar.md)
- [امثلة عامة `AccessManager-Example`](./Docs-AccessManager-Example-ar.md)
- [مثال عملى شامل  `AccessManager-Example-Api`](./Docs-AccessManager-Example-Api-ar.md)
- [التوثيق الكامل لإضافة `Nano.AuthApi`](./Docs-AuthApi-index.md)
- [كلاس `AdvancedQueryHelper` (Nano2.QueryBuilder)](../querybuilder/Docs-AdvancedQueryHelper.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)
