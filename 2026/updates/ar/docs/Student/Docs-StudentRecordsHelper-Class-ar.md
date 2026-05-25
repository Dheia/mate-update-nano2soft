# 📘 توثيق كلاس `StudentRecordsHelper`

**الإصدار:** 1.0.13  
**المسار:** `Tss\Student\Helpers\StudentRecordsHelper`  
**الغرض:** مساعد موحد لجلب سجلات الطلاب (جدول `tss_school_students`) باستخدام نظام تصفية متقدم، تخزين مؤقت، أحداث، ودعم كامل لـ `AccessManager` و `AdvancedQueryHelper`.

---

## 1. مقدمة

كلاس `StudentRecordsHelper` هو المسؤول عن تنفيذ جميع استعلامات جلب بيانات الطلاب في إضافة `Tss.Student`. يتبع هذا الكلاس نمطاً موحداً قابلاً لإعادة الاستخدام، ويمكن تخصيصه لأي نموذج آخر (مثل `StudentRecord` أو `Mparent`) مع إضافة فلاتر خاصة بالنموذج.

**الميزات الرئيسية:**

- **هيكل استجابة موحد:** جميع الدوال تعيد مصفوفة تحتوي على `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`.
- **تصفية ديناميكية متقدمة:** يدعم `is_or`, `is_not`, `is_force`, `is_or_null` لكل حقل قابل للتصفية.
- **بحث نصي ذكي:** مع خيارات البحث المتقدم وترتيب النتائج حسب الأوزان.
- **تخزين مؤقت (Caching):** مدمج بالكامل مع مفاتيح فريدة وعلامات (tags) وإمكانية فرض التحديث.
- **أحداث (Events):** يمكن تعطيلها أو تخصيص أسمائها، لتمكين التوسع دون تعديل الكود الأساسي.
- **دعم `AccessManager`:** يمكن تطبيق نطاقات الوصول (company, department, state, created_by, children) تلقائياً على الاستعلام.
- **مرونة الإخراج:** يمكن إرجاع `Builder`، `Collection`، `Paginator`، أول سجل، أو بيانات محولة عبر Transformer.
- **تحسين الأداء:** استبعاد الأعمدة غير المرغوبة (`exclude`)، وتحميل العلاقات الضرورية فقط.

---

## 2. فوائد الكلاس واستخداماته

| الفائدة | الوصف |
|---------|-------|
| **توحيد منطق الاستعلام** | يمنع تكرار كتابة استعلامات `where` و `join` في متحكمات متعددة. |
| **سهولة الصيانة** | أي تغيير في طريقة التصفية (مثل إضافة دعم `is_or_null`) يتم في مكان واحد. |
| **أمان أعلى** | بفضل دمج `AccessManager` وتطبيق شروط الصلاحية مباشرة على الاستعلام. |
| **مرونة عالية للـ API** | يمكن للعميل تحديد الحقول المستبعدة، العلاقات المضمنة، الترتيب، التصفية المنطقية، إلخ. |
| **جاهز للإنتاج** | يدعم التخزين المؤقت والأحداث، مما يسمح بتحسين الأداء وتوسيع الوظائف دون لمس الكود الأصلي. |
| **اختبار سهل** | يمكن تمرير خيار `is_to_sql` لطباعة الاستعلام النهائي لأغراض التصحيح. |

**الاستخدامات النموذجية:**

- جلب قائمة الطلاب من متحكم API (`Nano\StudentsApi\APIControllers\Students::index`).
- إنشاء تقارير مخصصة في Backend.
- استعلامات معقدة تحتاج إلى دمج عدة شروط منطقية (`OR` / `AND`).
- تصدير بيانات الطلاب مع إمكانية تحديد الأعمدة المطلوبة فقط.

---

## 3. دوال الكلاس وخصائصها

### 3.1 الدوال العامة (public static)

| الدالة | الوصف |
|--------|-------|
| `getRecords(array $options, bool $isException = false): array` | الدالة الأساسية لاسترجاع سجلات الطلاب. |
| `applyFieldFilter(...)` | تطبق فلتر حقل واحد مع دعم `is_or`, `is_not`, `is_force`, `is_or_null`. (يمكن استخدامها خارجياً). |
| `applyImagesFilters(...)` | تطبق فلاتر الصور والملفات (`is_has_any_images`, `is_has_image`, …). |
| `applySearchQuery(...)` | تطبق البحث النصي المتقدم مع خيارات الأوزان. |
| `applyUserFilter(...)` | تطبق فلترة المستخدم (`is_user`). |
| `applyInteractionFilters(...)` | تطبق فلاتر التفاعلات (`isFavorites`, `isLikes`, …). |
| `applyBlockedFilter(...)` | تطبق فلترة الحظر (`is_support_blocked`). |
| `applyGroupBy(...)` | تطبق `group by` بمصفوفة أو نص. |
| `applyOrderBy(...)` | تطبق الترتيب مع دعم `sortable scopes`. |
| `applySingleOrderBy(...)` | تطبق ترتيب حقل واحد مع التحقق من وجود الـ scope. |
| `executeQuery(...)` | تنفذ الاستعلام حسب نوع الإخراج المطلوب. |
| `transformResult(...)` | تحول `Paginator` إلى مصفوفة باستخدام `StudentTransformer`. |
| `normalizeWithArray(...)` | تحول قيمة `custom_with` إلى مصفوفة. |
| `isMethodExists(...)` | تتحقق من وجود دالة في كائن ما. |

### 3.2 دالة `getRecords` (التفاصيل)

**المعاملات (`$options`):**

| المجموعة | الخيارات الأساسية |
|----------|-------------------|
| التصفية الأساسية | `id`, `students_id`, `parents_id`, `gender`, `marital_status`, `passport_type`, `groups_class_id`, `type`, `country_id`, `state_id`, `directorate_id`, `year_id`, `class_id`, `group_id` |
| الحالات | `is_active`, `is_effective`, `is_published`, `has_parents` |
| الموقع والمسافة | `latitude`, `longitude`, `radius` |
| البحث | `q`, `is_advanced_search`, `advanced_search_mode`, `is_search_weights`, `search_weights_direction` |
| المستخدم والتفاعلات | `is_user`, `user`, `user_type`, `user_id`, `is_support_blocked`, `isFavorites`, `isLikes`, `isBookmarks`, `isReactions` |
| الاستعلام المتقدم | `exclude`, `custom_with`, `with_count`, `orderBy`, `orderDirection`, `group_by`, `group_by2`, `group_by3`, `having` |
| الإخراج | `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`, `withTrashed`, `onlyTrashed` |
| التخزين المؤقت | `cache_enabled`, `cache_key`, `cache_ttl`, `cache_tags`, `cache_force_refresh` |
| الأحداث | `fire_before_event`, `fire_after_event`, `event_before_name`, `event_after_name`, `event_list_name` |
| Transformer | `is_mate_data_images`, `is_show_image`, `is_show_files`, `is_mask_location`, `is_stop_user_include`, `is_stop_user_secret`, `is_stop_parent_include`, `is_stop_parent_secret` |
| التحكم في `AdvancedQueryManager` | `disable_advanced_query`, `force_advanced_query` |

**قيمة العودة (مصفوفة):**

```php
[
    'code'    => 200,
    'status'  => true,
    'message' => 'تم جلب البيانات بنجاح',
    'data'    => [...]   // Builder, Collection, Paginator, أو مصفوفة محولة
    'error'   => null,
    'errors'  => null,
    'input_data'   => [...],  // الخيارات الأصلية
    'process_data' => [...],  // الخيارات بعد الدمج
    'debug'   => [...]        // فقط عند APP_DEBUG = true
]
```

---

## 4. أمثلة متكاملة ومفصلة

### 4.1 الاستخدام الأساسي – جلب الطلاب النشطين مع الترتيب حسب الاسم

```php
$result = StudentRecordsHelper::getRecords([
    'is_active' => true,
    'orderBy' => 'full_name',
    'orderDirection' => 'asc',
    'per_page' => 20,
]);

if ($result['status']) {
    $paginator = $result['data']; // Paginator object
    foreach ($paginator as $student) {
        echo $student->full_name;
    }
} else {
    Log::error($result['error']);
}
```

### 4.2 استخدام الفلاتر المتقدمة (is_or, is_not, is_force, is_or_null)

```php
$result = StudentRecordsHelper::getRecords([
    'gender' => 'male,female',
    'is_or_gender' => true,
    'marital_status' => 'single,married',
    'is_or_marital_status' => true,
    'is_not_marital_status' => false,
    'country_id' => 'YE',
    'is_or_null_country_id' => true,   // يشمل السجلات التي country_id = null
]);
```

### 4.3 البحث النصي المتقدم مع ترتيب حسب الأوزان

```php
$result = StudentRecordsHelper::getRecords([
    'q' => 'محمد',
    'is_advanced_search' => true,
    'advanced_search_mode' => 'OR',
    'is_search_weights' => true,
    'search_weights_direction' => 'desc',
]);
```

### 4.4 تطبيق نطاق الصلاحيات (AccessManager) عبر `access_result`

```php
$access = AccessManager::instance()->check('list', $config, $user);
$options = [
    'access_result' => $access,
    'is_active' => true,
];
$result = StudentRecordsHelper::getRecords($options);
// داخل الدالة، سيتم استدعاء AccessManager::applyAccessScope تلقائياً.
```

### 4.5 استبعاد أعمدة وتحميل علاقات محددة

```php
$result = StudentRecordsHelper::getRecords([
    'exclude' => 'password,remember_token',   // أعمدة لا تريد جلبها
    'custom_with' => ['parents', 'studentRecord'], // تحميل العلاقات
    'with_count' => ['parents'], // جلب عدد أولياء الأمور
]);
```

### 4.6 استخدام التخزين المؤقت مع علامات (tags)

```php
$result = StudentRecordsHelper::getRecords([
    'cache_enabled' => true,
    'cache_ttl' => 30, // 30 دقيقة
    'cache_tags' => ['students', 'list'],
    'cache_force_refresh' => false,
]);
```

### 4.7 تصدير SQL لتصحيح الأخطاء

```php
$result = StudentRecordsHelper::getRecords([
    'is_to_sql' => true,
    'gender' => 'male',
]);
// ستتم طباعة الاستعلام الكامل في trace_log
```

### 4.8 جلب مجموعة (Collection) بدلاً من Paginator

```php
$result = StudentRecordsHelper::getRecords([
    'is_collection' => true,
    'is_active' => true,
    'take_limet' => 50,
]);
$students = $result['data']; // Collection
```

### 4.9 استخدام الفلاتر القديمة (disable_advanced_query)

```php
$result = StudentRecordsHelper::getRecords([
    'disable_advanced_query' => true,
    'gender' => 'male', // سيتم تحويله إلى where('gender', 'male') بدلاً من scopeWhereField
]);
```

---

## 5. دوال مساعدة (Helper Methods) قابلة للاستخدام الخارجي

### 5.1 `applyFieldFilter`

يمكن استخدامها لتطبيق فلتر متقدم على أي استعلام:

```php
$query = Student::query();
StudentRecordsHelper::applyFieldFilter(
    $query, 'tss_school_students', 'gender', 'male,female',
    ['is_or_gender' => true], $useAdvanced = true, $useLegacy = false, 'gender'
);
// النتيجة: حيث gender = 'male' OR gender = 'female'
```

### 5.2 `applyImagesFilters`

تطبق فلاتر الصور والملفات بسهولة:

```php
$query = Student::query();
StudentRecordsHelper::applyImagesFilters($query, [
    'is_has_any_images' => true,
    'is_has_files' => false,
]);
```

### 5.3 `normalizeWithArray`

تحويل `custom_with` إلى مصفوفة مهيأة:

```php
$with = StudentRecordsHelper::normalizeWithArray('parents,studentRecord');
// ['parents', 'studentRecord']
```

### 5.4 `isMethodExists`

التحقق من وجود دالة قبل استدعائها:

```php
if (StudentRecordsHelper::isMethodExists($query, 'withRating')) {
    $query->withRating('desc');
}
```

---

## 6. التوافق مع الإصدارات السابقة

- **لم يتم تغيير أي دالة قديمة** في `StudentHelper` أو `Tss\School\Models\StudentRecord`.
- الدالة `StudentHelper::getRecords` أصبحت مجرد واجهة تستدعي `StudentRecordsHelper::getRecords`، مما يضمن استمرار عمل الكود القديم.
- يمكن استخدام `StudentRecordsHelper` مباشرة في المشاريع الجديدة دون الحاجة إلى `StudentHelper`.

---

## 7. الخلاصة

| الميزة | التفاصيل |
|--------|----------|
| **مركزية استعلامات الطلاب** | كل استعلامات جلب الطلاب تمر عبر كلاس واحد. |
| **دعم كامل لـ AdvancedQueryHelper** | `scopeWhereField`, `getQueryDate`, إلخ. |
| **دعم AccessManager** | تطبيق نطاقات الصلاحيات تلقائياً. |
| **تخزين مؤقت متكامل** | مفاتيح فريدة، علامات، إجبار التحديث. |
| **أحداث مرنة** | يمكن تعطيلها أو تخصيص أسمائها. |
| **مرونة الإخراج** | Builder, Collection, Paginator, أول سجل, أو محول (Transformer). |
| **سهولة التوسع** | إضافة فلاتر جديدة يكون بإضافة بضعة أسطر فقط. |

**الاستخدام الموصى به:**

في أي متحكم API يرغب في جلب قائمة الطلاب، قم باستدعاء `StudentRecordsHelper::getRecords` مباشرة، مع تمرير خيار `access_result` إذا كنت تستخدم `AccessManager`. هذا يضمن أداءً عالياً وأماناً محكماً.

---

## 8. الخاتمة

كلاس `StudentRecordsHelper` هو حجر الأساس لجميع استعلامات الطلاب في نظام Nano البيئي. بفضل تصميمه المعياري واستفادته من `AdvancedQueryHelper` و `AccessManager`، يوفر مرونة غير مسبوقة في بناء استعلامات معقدة مع الحفاظ على أداء ممتاز وأمان عالٍ. نوصي بشدة باستخدام هذا الكلاس كمرجع عند إنشاء دوال `getRecords` لأي نموذج آخر (مثل `Mparent` أو `StudentRecord`).

**المراجع:**

- [التوثيق الكامل لإضافة `Tss.Student`](./Docs-Student-ar.md)
- [كلاس `AccessManager` (Nano.AuthApi)](../AuthApi/Docs-AccessManager-ar.md)
- [كلاس `AdvancedQueryHelper` (Nano2.QueryBuilder)](../querybuilder/Docs-AdvancedQueryHelper.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)
