## 2026-05-25 - 2026-05-26

**تحديث إضافة `Nano.AuthApi` – الإصدار 1.0.21**

### إضافة ميزات متقدمة إلى `AccessManager`: محلل ديناميكي للطلاب وأولياء الأمور، فلاتر متقدمة، تحميل مركزي للإعدادات، دوال مبسطة، ودعم الرجوع.

---

### ملخص التحديثات

يأتي الإصدار **1.0.21** من إضافة `Nano.AuthApi` بعدد من التحسينات الجوهرية على كلاس `AccessManager`، الذي أصبح الآن مسؤولاً ليس فقط عن إدارة الصلاحيات، بل أيضاً عن معالجة سيناريوهات متقدمة مثل:

- **المحلل الديناميكي للطلاب وأولياء الأمور (`resolveDynamicFrontendOptions`)** – يقوم تلقائياً بتعبئة `student_id` و `record_id` بناءً على نوع المستخدم وقواعد مرنة من `config.php`.
- **التحكم في الفلاتر المتقدمة (`advanced_filters`)** – يسمح بتحديد الحقول التي يمكن استخدام `is_or`, `is_not`, `is_or_null` عليها، مع قواعد حسب نوع المستخدم و `ref_type`.
- **تحميل مركزي لإعدادات الصلاحيات** – دوال `getOperationByConfig`, `checkByResource` تقرأ الإعدادات مباشرة من `config.php` دون الحاجة لبناء مصفوفات يدوية.
- **دعم الرجوع (Fallback)** – دوال `getOperationConfigWithFallback`, `checkWithFallback` تسمح بعمليات مثل `show` بالاستفادة من إعدادات `list`.
- **دوال مبسطة للوصول إلى الإعدادات** – `getOperationBySimpleKey`, `getAdvancedFiltersConfig` تسهل الاستخدام اليومي.

هذه التحسينات تجعل `AccessManager` أداة متكاملة لإدارة الصلاحيات، الفلاتر، ومعالجة متطلبات الطلاب وأولياء الأمور في أي API.

---

### أهداف الإصدار

- **تمكين تطبيقات المدارس** من التعامل بسهولة مع حالات الطلاب وأولياء الأمور عبر `frontend_resolver`.
- **توفير تحكم دقيق في الفلاتر** لضمان أمان البيانات ومنع تمرير فلاتر خطيرة من مستخدمين غير مصرح لهم.
- **مركزية إعدادات الصلاحيات** في ملف `config.php` لتقليل الكود المكرر وتسهيل الصيانة.
- **تبسيط استخدام `AccessManager`** عبر دوال مختصرة (مثل `checkByResource` بدلاً من تمرير 3 معاملات).
- **دعم الرجوع تلقائياً** لعمليات مثل `show` دون الحاجة إلى كتابة إعدادات كاملة.

---

### الميزات الجديدة والتحسينات

#### 1. المحلل الديناميكي للطلاب وأولياء الأمور (`resolveDynamicFrontendOptions`)

تمت إضافة مجموعة دوال للتعامل مع سيناريوهات الطلاب وأولياء الأمور:

- **`resolveDynamicFrontendOptions($user, $options, $query, $resourceKey)`** – دالة رئيسية تقوم بتحليل `ref_type` للمستخدم وتطبيق قواعد محددة في `frontend_resolver` من `config.php`.
- **`getFrontendResolverConfig($resourceKey)`** – تحميل إعدادات المحلل من ملف الإعدادات.
- **`ensureOptionsKeys($options, $keys)`** – التأكد من وجود مفاتيح معينة في مصفوفة الخيارات لتجنب أخطاء `undefined index`.

**مثال على إعدادات `frontend_resolver` في `config.php`:**

```php
'frontend_resolver' => [
    'enabled' => true,
    'apply_for_backend' => false,
    'apply_for_frontend' => true,
    'ref_type_rules' => [
        'student' => [
            'fields' => [
                'student_id' => ['source' => 'user', 'auto_fill' => true, 'force' => true],
                'record_id'  => ['source' => 'current_record', 'auto_fill_if_missing' => true, 'force' => true],
            ],
        ],
        'parent' => [
            'fields' => [
                'student_id' => ['source' => 'request', 'required' => true, 'validate_ownership' => true, 'force' => true],
                'record_id'  => ['source' => 'request_or_current', 'fallback_field' => 'student_id', 'force' => true],
            ],
        ],
    ],
],
```

عند استدعاء `resolveDynamicFrontendOptions`، سيقوم `AccessManager` تلقائياً بتعبئة `student_id` و `record_id` بناءً على هذه القواعد.

#### 2. التحكم في الفلاتر المتقدمة (`advanced_filters`)

تمت إضافة دوال للتحقق مما إذا كان المستخدم مسموحاً له باستخدام مفاتيح `is_or_*`, `is_not_*`, `is_or_null_*`:

- **`isAdvancedFilterAllowed($user, $field, $operationConfig)`** – تتحقق من السماح بناءً على `advanced_filters` في إعدادات العملية.
- **`filterAdvancedOptions($user, $options, $operationConfig)`** – تزيل أي مفاتيح غير مسموحة من `$options`.
- **`applyAdvancedFilterConstraints($user, $options, $operationConfig)`** – دالة شاملة (مرادف لـ `filterAdvancedOptions`).

**مثال على إعدادات `advanced_filters`:**

```php
'advanced_filters' => [
    'enabled' => true,
    'default_for_backend' => true,
    'default_for_frontend' => false,
    'field_specific_rules' => [
        'student_id' => [
            'backend' => true,
            'frontend' => ['*' => false, 'parent' => true],
        ],
    ],
],
```

#### 3. تحميل مركزي لإعدادات الصلاحيات من `config.php`

تمت إضافة دوال لقراءة الإعدادات وتطبيعها:

- **`getDefaultOperationConfig()`** – تعيد الإعدادات الافتراضية القياسية.
- **`normalizeOperationConfig($config)`** – تدمج الإعدادات المقدمة مع الافتراضية.
- **`arrayMergeRecursiveDistinct($array1, $array2)`** – دمج مصفوفات متكرر مع استبدال القيم وليس دمجها.
- **`getOperationConfig($plugin, $resource, $operation, $overrideConfig)`** – تجلب الإعدادات من `config` باستخدام ثلاثة معاملات.
- **`checkWithConfig($operation, $plugin, $resource, $user, $overrideConfig)`** – تجمع بين الجلب والتحقق.

#### 4. دوال مبسطة للوصول إلى الإعدادات (Single Key)

لتسهيل الاستخدام، تمت إضافة دوال تستقبل مفتاحاً واحداً بدلاً من ثلاثة:

- **`getOperationByConfig($resourceKey, $overrideConfig)`** – مفتاح مثل `'nano.absenceapi::absences.list'`.
- **`checkByResource($resourceKey, $user, $overrideConfig)`** – تستخرج العملية تلقائياً من آخر جزء من المفتاح.
- **`getAdvancedFiltersConfig($resourceKey, $overrideConfig)`** – تعيد فقط مصفوفة `advanced_filters` للمورد.
- **`getOperationBySimpleKey($simpleKey, $overrideConfig)`** – تختصر كتابة `'nano.absenceapi::'` (مثال: `'absences.list'`).

#### 5. دعم الرجوع (Fallback) بين العمليات

تساعد هذه الدوال في سيناريوهات مثل `show` التي يمكنها استخدام إعدادات `list` إذا لم تكن إعدادات `show` مكتملة:

- **`getOperationConfigWithFallback($resourceKey, $fallbackResourceKey, $overrideConfig)`**
- **`checkWithFallback($resourceKey, $fallbackResourceKey, $user, $overrideConfig)`**

إذا كان `is_allow` موجوداً في الإعدادات الأساسية، يُستخدم مباشرة. وإلا، يتم دمج الإعدادات الاحتياطية مع الأساسية.

#### 6. دوال مساعدة إضافية لسيناريوهات الطلاب / أولياء الأمور

- **`applyFrontendStudentParentFilters($query, $user, $options, $studentIdColumn, $recordIdColumn, $tablePrefix, $resolveIfNeeded)`** – تطبق الفلاتر المستنتجة مباشرة على الاستعلام.
- **`getStudentIdForFrontendUser($user, $options)`**, **`getRecordIdForFrontendUser($user, $options)`** – دوال مختصرة لجلب القيم المحلولة.
- **`validateParentStudentRelation($user, $studentId, $recordId)`** – تتحقق من أن الطالب يخص ولي الأمر.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. التحقق من صلاحية `list` باستخدام `checkByResource`

```php
$access = AccessManager::checkByResource('nano.absenceapi::absences.list', $user);
if (!$access['allowed']) {
    return $this->errorUnauthorized($access['message']);
}
```

#### 2. استخدام `checkWithFallback` لعملية `show`

```php
$access = AccessManager::checkWithFallback(
    'nano.absenceapi::absences.show',
    'nano.absenceapi::absences.list',
    $user
);
```

#### 3. تطبيق الفلاتر المتقدمة وإزالة غير المسموحة

```php
$advConfig = AccessManager::getAdvancedFiltersConfig('nano.absenceapi::absences.list');
$options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advConfig]);
```

#### 4. تطبيق المحلل الديناميكي لحل `student_id` و `record_id`

```php
$options = AccessManager::instance()->resolveDynamicFrontendOptions(
    $user, $options, null, 'nano.absenceapi::absences.list'
);
// ثم في getRecords()، يمكن تطبيق الفلاتر المستنتجة
if (!empty($options['student_id']) && $options['is_force_student_id']) {
    $query->where('student_id', $options['student_id']);
}
```

#### 5. تطبيق `applyAccessScope` مع إمكانية تعطيل نطاقات

```php
$query = AccessManager::instance()->applyAccessScope($query, $access, [
    'company_field'    => 'companys_id',
    'department_field' => 'departments_id',
    // 'state_field' omitted -> no state filter
    'created_by_field' => 'created_by',
]);
```

---

### التوافق مع الإصدارات السابقة

- **جميع الدوال القديمة** (مثل `check`, `applyAccessScope`, `canAccessAllCompanies`) لا تزال تعمل كما هي.
- **الكلاس لم يفقد أي وظيفة** – تمت الإضافة فقط، لم يتم الحذف.
- **لا توجد هجرات قاعدة بيانات** مطلوبة.
- **يمكن الترقية مباشرة** دون تعديل الكود الحالي.

---

### متطلبات الترقية (من 1.0.20 إلى 1.0.21)

1. **تحديث الكود**:
   - استبدال ملف `AccessManager.php` بالنسخة الجديدة (الموجودة في المرفقات).
   - لا توجد تغييرات أخرى في الإضافة.

2. **تحديث ملف `version.yaml`**:
   - أضف الإصدار `1.0.21` كما هو موضح أعلاه.

3. **تحديث ملفات `config.php` لإضافات API** (اختياري):
   - يمكن إضافة أقسام `advanced_filters` و `frontend_resolver` إلى إعدادات الموارد للاستفادة من الميزات الجديدة.

4. **تنفيذ `php artisan plugin:refresh Nano.AuthApi`** (اختياري).

5. **اختبار الميزات الجديدة**:
   - جرب `checkByResource` بدلاً من `check` التقليدية.
   - جرب `checkWithFallback` لـ `show` endpoint.
   - جرب `filterAdvancedOptions` لتنظيف خيارات الطلب.
   - جرب `resolveDynamicFrontendOptions` في متحكم يستخدم الطلاب وأولياء الأمور.

---

### الفوائد والقيمة المضافة

- **تقليل الكود المكرر** في متحكمات API بنسبة تصل إلى 70%.
- **مركزية التحكم** في الصلاحيات والفلاتر، مما يسهل تغيير السلوك عبر ملفات الإعدادات فقط.
- **أمان أعلى** بفضل التحكم الدقيق في الفلاتر المتقدمة ومنع المستخدمين غير المخولين من استخدامها.
- **دعم السيناريوهات المدرسية** (طلاب وأولياء أمور) بشكل طبيعي ومباشر.
- **سهولة الصيانة** حيث أن أي تعديل في منطق الصلاحيات يتم في مكان واحد.

---

### الخاتمة

الإصدار **1.0.21** من `Nano.AuthApi` يضع `AccessManager` كأداة شاملة ومتقدمة لإدارة الصلاحيات والفلاتر ومعالجة العلاقات بين المستخدمين والكيانات (مثل الطلاب وأولياء الأمور). بفضل الدوال الجديدة، أصبح بناء API آمن ومرن أسهل بكثير، ويتماشى تماماً مع دليل `Nano-Api-SKILL.md`.

---

**الوثائق المرجعية**:
- [توثيق كلاس `AccessManager`](./docs/AuthApi/Docs-AccessManager-ar.md)
-
- [توثيق الصلاحيات `AccessManager-Permission`](./docs/AuthApi/Docs-AccessManager-Permission-ar.md)
- [امثلة عامة `AccessManager-Example`](./docs/AuthApi/Docs-AccessManager-Example-ar.md)
- [مثال عملى شامل  `AccessManager-Example-Api`](./docs/AuthApi/Docs-AccessManager-Example-Api-ar.md)
- [توثيق كلاس `AuthHelpers`](./docs/AuthApi/Docs-AuthHelpers-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md
- [تحديث إضافة `Nano.AbsenceApi` (نموذج تطبيقي)](./docs/AbsenceApi/Update-AbsenceApi-v1.1.0-ar.md)