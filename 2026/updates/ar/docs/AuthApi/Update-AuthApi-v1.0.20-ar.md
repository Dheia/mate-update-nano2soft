## 2026-05-24 - 2026-05-25

**تحديث إضافة `Nano.AuthApi` – الإصدار 1.0.20**

### إضافة كلاس `AccessManager` لإدارة الصلاحيات والوصول بشكل مركزي ومتقدم

---

### ملخص التحديثات

يقدم الإصدار **1.0.20** من إضافة `Nano.AuthApi` كلاساً جديداً هو `AccessManager`، وهو مسؤول عن إدارة صلاحيات المستخدمين (Backend، Frontend، ضيف) والتحكم في نطاق الوصول (Access Scopes) بطريقة موحدة وقابلة للتوسع. يدمج الكلاس دوال مساعدة من `BasicHelper` (مثل `checkAccessAllCompanys`) ويوفر واجهة بسيطة لتطبيق قيود الوصول على استعلامات Eloquent.

يهدف هذا التحديث إلى توحيد منطق الصلاحيات عبر جميع إضافات النظام، وتقليل الكود المكرر، ودعم سيناريوهات متقدمة مثل الوصول حسب الشركة أو القسم أو الولاية، أو عرض السجلات التي أنشأها المستخدم فقط، أو عرض أبناء ولي الأمر.

---

### أهداف الإصدار

- **توفير كلاس مركزي** لإدارة الصلاحيات بدلاً من توزيع المنطق في متحكمات متعددة.
- **دعم أنواع المستخدمين الثلاثة:** Backend (Nano Soft App)، Frontend (RainLab.User أو أي نموذج مخصص)، Guest.
- **دعم نطاقات وصول متعددة (Scopes):**
  - `all`: الوصول إلى كل السجلات.
  - `own`: السجلات التي أنشأها المستخدم (عبر `created_by`).
  - `created_by`: نفس `own`.
  - `company`: الوصول إلى سجلات شركة محددة فقط (مع إمكانية الوصول إلى كل الشركات للمستخدمين المخولين).
  - `department`: الوصول إلى سجلات قسم معين فقط.
  - `state`: الوصول إلى سجلات ولاية معينة فقط.
  - `children`: لمستخدمي Frontend من نوع ولي أمر، لعرض سجلات أبنائه.
- **توفير دوال لتطبيق النطاق على Query Builder** (`applyAccessScope`, `applyCompanyScope`, `applyDepartmentScope`, `applyStateScope`, `applyCreatedByScope`, `applyUserScope`, `applyChildrenScope`).
- **دعم تخصيص أسماء الحقول** (مثل `companys_id`, `departments_id`, `state_id`, `created_by`) عبر خيارات مرنة.
- **دعم التحقق من الصلاحيات (Permissions)** لمستخدمي Backend باستخدام `hasAccess` أو `hasPermission`.
- **دعم تصفية أنواع المستخدمين Frontend** عبر `ref_type` (مثل `student`, `parent`).
- **إرجاع معلومات مفصلة** عن نتيجة التحقق (السماح، النطاق، الرسالة، تفاصيل النطاق المطبق).

---

### الميزات الجديدة والتحسينات

#### 1. كلاس `AccessManager` – الوظائف الأساسية

- **`setUser($user) / getUser()`**: تعيين أو جلب المستخدم الحالي (يدعم الجلب التلقائي من `AuthHelpers`, `BackendAuth`, `Auth`).
- **`getUserType($user)`**: يحدد نوع المستخدم (`backend`, `frontend`, `guest`).
- **`getUserRefType($user)`, `getUserRefId($user)`**: يستخرج `ref_type` و `ref_id` (للمستخدمين المرتبطين بنماذج مثل الطلاب).
- **`getUserCompanyId($user)`, `getUserDepartmentId($user)`**: يستخرج معرف الشركة والقسم من المستخدم (إذا كان يمتلك الخصائص).
- **`check(string $operation, array $config, $user)`**: دالة التحقق الأساسية. تعيد مصفوفة تحتوي على `allowed`, `scope`, `message`, `user_type`, `user`, `config`, `applied_scope_details`.

#### 2. دعم نطاقات متقدمة لـ Backend

يمكن تخصيص سلوك Backend عبر خيارات `backend` في مصفوفة `$config`:

```php
'backend' => [
    'allow' => true,                         // هل مسموح لمستخدمي Backend
    'check_permission' => true,              // هل نتحقق من الصلاحيات
    'permissions' => ['tss.example.access'], // الصلاحيات المطلوبة
    'access_scope' => 'company',             // النطاق المطلوب
    'company_field' => 'companys_id',        // حقل الشركة في الجدول
    'department_field' => 'departments_id',
    'state_field' => 'state_id',
    'created_by_field' => 'created_by',
    'company_id' => 2,                       // رقم شركة محدد (اختياري)
    'department_id' => 5,
    'state_id' => 1,
],
```

يدعم `access_scope` القيم: `all`, `own`, `created_by`, `company`, `department`, `state`.

عند استخدام نطاق `company` أو `department` أو `state`، يتحقق `AccessManager` أولاً مما إذا كان المستخدم يملك صلاحية الوصول إلى كل الشركات/الأقسام/الولايات (عبر `canAccessAllCompanies()` وغيرها). إذا كان يملك، يصبح النطاق الفعلي `all`. وإلا، يستخدم المعرف المرتبط بالمستخدم أو المعرف الممرر في الإعدادات.

#### 3. دعم Frontend مع تصفية `ref_type` ونطاق `children`

```php
'frontend' => [
    'allow' => true,
    'allowed_ref_types' => ['student', 'parent'], // الأنواع المسموحة
    'access_scope' => 'own',                     // 'own' أو 'children'
    'children_relation' => 'students',           // اسم العلاقة لجلب الأبناء
],
```

إذا كان النطاق `children`، يجب تعريف `children_relation` (مثل `students` في نموذج ولي الأمر). ثم يمكن استخدام `applyChildrenScope()` لتطبيق الفلترة.

#### 4. دوال تطبيق النطاق على Query Builder

- **`applyAccessScope($query, $accessResult, $options)`**: دالة عامة تستخدم نتيجة `check()` لتطبيق النطاق المناسب.
- **`applyCompanyScope($query, $companyId, $field)`**, **`applyDepartmentScope`**, **`applyStateScope`**, **`applyCreatedByScope`**, **`applyUserScope`**.
- **`applyChildrenScope($query, $parent, $studentIdField, $relation)`**: تطبق فلترة لجلب سجلات أبناء ولي أمر معين.

**مثال:**

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
    // إذا كان scope = children
    if ($access['scope'] === 'children') {
        $parent = Mparent::byUser($user)->first();
        $query = AccessManager::instance()->applyChildrenScope($query, $parent, 'student_id', 'students');
    }
}
```

#### 5. دوال مساعدة للتحقق من صلاحيات الشركات والأقسام

- **`canAccessAllCompanies($user)`**: تعيد `true` إذا كان المستخدم يمكنه الوصول إلى كل الشركات (يستخدم `BasicHelper::checkAccessAllCompanys`).
- **`canAccessAllDepartments($user)`**
- **`canAccessAllStates($user)`**
- **`canAccessDepartment($departmentId, $user)`**

هذه الدوال تبسط المنطق وتضمن الاتساق مع بقية النظام.

#### 6. دالة `getPermissionArray`

تعيد مصفوفة مشابهة لتلك المستخدمة في `OrderHelper::getArrayDataAfterPermission`، مما يسهل التكامل مع الكود القديم.

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
// ['allowed'=>true, 'scope'=>'department', 'check_departments'=>true, 'departments_id'=>5, ...]
```

#### 7. دعم متكامل للـ Guest

يمكن السماح للمستخدمين غير المسجلين بالوصول إلى بعض العمليات عبر إعدادات `guest`:

```php
'guest' => [
    'allow' => true,
    'access_scope' => 'none',
],
```

---

### أمثلة على الاستخدام

#### 1. التحقق من صلاحية الوصول إلى قائمة الطلاب لمستخدم Frontend من نوع ولي أمر

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

#### 2. تطبيق نطاق `company` لمستخدم Backend

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
    // سيتم إضافة شرط companys_id = (معرف الشركة المرتبطة بالمستخدم أو المحدد في config)
}
```

#### 3. الحصول على مصفوفة إعدادات الصلاحيات لاستخدامها في دوال `listExtendQuery`

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
if ($perms['allowed']) {
    if ($perms['check_departments']) {
        $query->where('departments_id', $perms['departments_id']);
    }
}
```

---

### التوافق مع الإصدارات السابقة

- **لم يتم تغيير أي دوال قديمة** في `Nano.AuthApi`. تمت إضافة كلاس `AccessManager` فقط.
- يمكن الاستمرار في استخدام دوال `AuthHelpers` القديمة (`getCurrentUser`, `isBackendUser`, إلخ) دون أي تأثير.
- لا توجد تغييرات في قاعدة البيانات أو الإعدادات.

---

### متطلبات الترقية (من أي إصدار سابق إلى 1.0.20)

1. **تحديث الكود**:
   - استبدال ملف `classes/AccessManager.php` بالنسخة الجديدة (المرفقة).
   - (اختياري) تحديث `Plugin.php` إذا كنت تريد تسجيل `AccessManager` كـ Singleton أو توفير alias، لكن غير مطلوب.

2. **لا توجد هجرات قاعدة بيانات** جديدة.

3. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو.

4. **تحديث ملف الإصدارات `version.yaml`**:
   - أضف السطر `1.0.20:` كما هو موضح في بداية هذا المستند.

5. **تنفيذ `php artisan plugin:refresh Nano.AuthApi`** (اختياري) لتسجيل الإصدار الجديد.

6. **اختبار الوظائف**:
   - اختبر `AccessManager::check()` مع سيناريوهات مختلفة (Backend، Frontend، Guest).
   - تأكد من أن `applyAccessScope` تضيف شروط `where` صحيحة.
   - تحقق من دوال `canAccessAllCompanies` وغيرها.

---

### الفوائد والقيمة المضافة

- **توحيد منطق الصلاحيات** عبر جميع إضافات API (`Nano.StudentsApi`, `Nano.AbsenceApi`, ...)، مما يقلل تكرار الكود ويوحد السلوك.
- **تحكم دقيق في الوصول** بناءً على الشركة/القسم/الولاية، مما يتيح بناء أنظمة متعددة الفروع (Multi‑tenancy) بسهولة.
- **دعم طبيعي لنطاق `children`** لولاة الأمور، مما يسهل بناء تطبيقات مدرسية.
- **سهولة التوسع** بإضافة نطاقات جديدة (مثل `team`, `project`) دون تعديل الكلاس الأساسي.
- **تحسين الأمان** من خلال الفصل الواضح بين منطق الصلاحيات ومنطق الأعمال.

---

### الخاتمة

يمثل الإصدار **1.0.20** من `Nano.AuthApi` خطوة مهمة نحو توحيد إدارة الصلاحيات في نظام Nano البيئي. بفضل `AccessManager`، أصبح بإمكان المطورين تطبيق قيود وصول معقدة بسهولة، مع الحفاظ على مرونة التخصيص وقابلية التوسع. نوصي باستخدام هذا الكلاس في جميع إضافات API المستقبلية لضمان اتساق الأمان وسهولة الصيانة.

---

**الوثائق المرجعية**:
- [توثيق كلاس `AccessManager`](./docs/AuthApi/Docs-AccessManager-ar.md)
- [توثيق كلاس `AuthHelpers`](./docs/AuthApi/Docs-AuthHelpers-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md)

