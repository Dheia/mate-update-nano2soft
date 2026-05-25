# 📘 توثيق كلاس `StudentHelper`

**الإصدار:** 1.0.13  
**المسار:** `Tss\Student\Helpers\StudentHelper`  
**الغرض:** توفير واجهة موحدة ومبسطة للتعامل مع بيانات الطلاب وأولياء الأمور وسجلاتهم الدراسية، مع دمج دوال متقدمة من `StudentRecordsHelper` ودوال مساعدة للربط بين المستخدمين والكيانات المختلفة.

---

## 1. مقدمة

كلاس `StudentHelper` هو الواجهة الرئيسية التي يستخدمها المطورون (في الـ API أو Backend) للتفاعل مع بيانات الطلاب وأولياء الأمور وسجلات الطلاب (`StudentRecord`). تم إعادة هيكلة هذا الكلاس في الإصدار 1.0.13 ليعتمد على `StudentRecordsHelper` في عمليات الاستعلام (`getRecords`) مع الاحتفاظ بكامل التوافق العكسي. كما يضم الكلاس دوالاً مساعدة هامة مثل:

- ربط المستخدم (`User`) بالطالب (`Student`) أو ولي الأمر (`Mparent`).
- جلب أبناء ولي الأمر.
- استخراج معرف الطالب من سجل `StudentRecord`.
- جلب السجل الدراسي الحالي للطالب.

**الميزات الرئيسية للكلاس:**

- **توافق عكسي كامل:** دوال `getRecords` القديمة لا تزال تعمل بنفس التوقيع السابق.
- **مرونة في الاستعلام:** يمكنك استخدام `getRecords`, `getParentRecords`, `getStudentRecordRecords` حسب الحاجة.
- **دوال مساعدة للربط:** دوال مثل `getStudentByUser`, `getUserByStudent`, `getMparentByUser`, `getUserByMparent` تسهل الانتقال بين الكيانات.
- **دعم العلاقات العائلية:** دوال `getStudentIdsByMparent`, `getStudentsByMparent`, `getStudentsByUser` تمكن من جلب أبناء ولي الأمر بسهولة.
- **دوال تاريخية مساعدة:** `getStudentRecord`, `getResultRecord`, `setUpStudentRecord` (للترفيع) لا تزال موجودة للاستخدام القديم.

---

## 2. فوائد الكلاس واستخداماته

| الفائدة | الوصف |
|--------|-------|
| **واجهة واحدة للطلاب وأولياء الأمور** | يغطي الكلاس جميع العمليات الأساسية المتعلقة بالطلاب وأولياء الأمور وسجلاتهم. |
| **سهولة الاستخدام في الـ API** | يمكن استدعاء `StudentHelper::getRecords` مباشرة من متحكمات API دون الحاجة لإنشاء كائنات إضافية. |
| **ربط المستخدمين بالطلاب وأولياء الأمور** | دوال `get*ByUser` تسمح بالحصول على الطالب أو ولي الأمر المرتبط بالمستخدم الحالي (أو أي مستخدم). |
| **دعم السيناريوهات العائلية** | جلب أبناء ولي أمر معين أو جلب الطلاب المرتبطين بمستخدم (سواء كان ولي أمر أو طالباً). |
| **الحفاظ على الكود القديم** | الدوال القديمة مثل `getStudentRecord`, `getResultRecord`, `setUpStudentRecord` لم يتم حذفها، وتم إعادة تسمية النسخة الحديثة من `getStudentRecord` إلى `getStudentRecordV1` لتجنب التعارض. |
| **تكامل مع `StudentRecordsHelper`** | جميع دوال الاستعلام الجديدة تستفيد من التخزين المؤقت، الأحداث، والفلاتر المتقدمة دون إعادة تنفيذ المنطق. |

**الاستخدامات النموذجية:**

- جلب قائمة الطلاب في متحكم `StudentsApi::index`.
- عرض بيانات الطالب الشخصية المرتبطة بالمستخدم الحالي.
- في واجهة ولي الأمر: عرض قائمة أبنائه المسجلين في النظام.
- الحصول على السجل الدراسي الحالي للطالب (الصف، السنة، الرسوم).
- تحويل معرف سجل الطالب إلى معرف الطالب نفسه.

---

## 3. دوال الكلاس وخصائصها

### 3.1 دوال الاستعلام الرئيسية

| الدالة | الوصف |
|--------|-------|
| `getRecords(array $options, bool $isException = false): array` | تستدعي `StudentRecordsHelper::getRecords` لجلب بيانات الطلاب مع دعم جميع خيارات التصفية المتقدمة. |
| `getParentRecords(array $options, bool $isException = false): array` | جلب بيانات أولياء الأمور مع دعم فلاتر خاصة مثل `has_students`, `students_id`, `is_has_students_count`. |
| `getStudentRecordRecords(array $options, bool $isException = false): array` | جلب سجلات الطلاب (`StudentRecord`) مع دعم فلاتر خاصة بالرسوم والتواريخ والحالات. |

جميع هذه الدوال تعيد مصفوفة موحدة تحتوي على `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`.

### 3.2 دوال الربط بين المستخدمين والطلاب / أولياء الأمور

| الدالة | الوصف |
|--------|-------|
| `getStudentByUser($user = null, $isForce = false)` | يعيد كائن الطالب المرتبط بالمستخدم (عبر `user_id`/`user_type` أو `ref_id`/`ref_type`). |
| `getUserByStudent($student = null, $isForce = false)` | يعيد كائن المستخدم المرتبط بالطالب. |
| `getMparentByUser($user = null, $isForce = false)` | يعيد كائن ولي الأمر المرتبط بالمستخدم. |
| `getUserByMparent($mparent = null, $isForce = false)` | يعيد كائن المستخدم المرتبط بولي الأمر. |

### 3.3 دوال خاصة بالأبناء (لـ `Mparent`)

| الدالة | الوصف |
|--------|-------|
| `getStudentIdsByMparent($mparent, bool $useCache = true): array` | يعيد مصفوفة بأرقام الطلاب التابعين لولي أمر معين. |
| `getStudentsByMparent($mparent, array $options = [])` | يعيد كائنات الطلاب التابعين لولي أمر معين (يدعم نفس خيارات `getRecords`). |
| `getStudentsByUser($user = null, array $options = [])` | جلب الطلاب المرتبطين بمستخدم معين. إذا كان المستخدم ولي أمر، يرجع أبناءه؛ وإذا كان طالباً، يرجع بياناته فقط. |

### 3.4 دوال خاصة بـ `StudentRecord`

| الدالة | الوصف |
|--------|-------|
| `getStudentIdFromRecord($record): ?int` | يستخرج معرف الطالب من كائن `StudentRecord` أو رقم السجل أو مصفوفة تحتوي على `student_id`. |
| `getCurrentStudentRecord($student, array $options = []): ?StudentRecord` | يجلب السجل الدراسي الحالي للطالب (عادة بحالة `status_class = 'continue'`) مع دعم تحديد السنة والصف. |

### 3.5 دوال تاريخية (للتوافق مع الإصدارات السابقة)

| الدالة | الوصف |
|--------|-------|
| `getStudentRecordV1(array $options = [])` | النسخة القديمة من `getStudentRecord` (تمت إعادة تسميتها). تجلب سجلات الطلاب بناءً على خيارات محدودة. |
| `getResultRecord(array $options = [])` | تجلب سجلات النتائج (من `Tss\SchoolControl\Models\Result`). |
| `setUpStudentRecord(array $options = [])` | تنفذ عملية ترفيع الطلاب (تغيير `status_class`) بناءً على نتائجهم وخيارات التحكم. |
| `getQueryDate($records , $options = [])` | دالة مساعدة لفلترة التاريخ باستخدام `AdvancedQueryHelper`. |
| `checkValueIsNotAll($value = null)` | تتحقق مما إذا كانت القيمة ليست `*` أو `'all'`. |
| `scopeWhereField(...)` | تطبق شرط `where` على حقل مع دعم `is_or`, `is_not`, `is_force`. |
| `sanitizeDateFields(...)`, `parseCarbon`, `makeCarbon`, `w3cDatetime`, `toDatetime` | دوال مساعدة لمعالجة التواريخ (تستخدم داخلياً). |

---

## 4. أمثلة متكاملة ومفصلة

### 4.1 جلب قائمة الطلاب مع فلترة متقدمة

```php
$result = StudentHelper::getRecords([
    'gender' => 'male,female',
    'is_or_gender' => true,
    'is_active' => true,
    'q' => 'محمد',
    'orderBy' => 'full_name',
    'orderDirection' => 'asc',
    'per_page' => 20,
    'cache_enabled' => true,
]);

if ($result['status']) {
    $paginator = $result['data'];
    foreach ($paginator as $student) {
        echo $student->full_name . '<br>';
    }
} else {
    Log::error($result['error']);
}
```

### 4.2 جلب أولياء الأمور الذين لديهم أكثر من ولدين

```php
$result = StudentHelper::getParentRecords([
    'is_has_students_count' => ['>', 2],
    'orderBy' => 'full_name',
]);

if ($result['status']) {
    $parents = $result['data'];
    foreach ($parents as $parent) {
        echo $parent->full_name . ' - عدد الأبناء: ' . $parent->students_count . '<br>';
    }
}
```

### 4.3 جلب سجلات الطلاب (StudentRecord) لسنة دراسية معينة وحالة مستمر

```php
$result = StudentHelper::getStudentRecordRecords([
    'year_id' => 3,
    'status_class' => 'continue',
    'with_count' => ['student', 'class'],
]);

if ($result['status']) {
    $records = $result['data']['data']; // في الوضع الافتراضي يعيد Paginator مع transformer
    foreach ($records as $record) {
        echo "الطالب: {$record['student']['full_name']}, الصف: {$record['class']['name']}<br>";
    }
}
```

### 4.4 الحصول على الطالب المرتبط بالمستخدم الحالي

```php
$user = BackendAuth::getUser(); // أو AuthHelpers::getCurrentUser()
$student = StudentHelper::getStudentByUser($user);
if ($student) {
    echo "مرحباً {$student->full_name}";
} else {
    echo "أنت لست طالباً مسجلاً في النظام.";
}
```

### 4.5 جلب أبناء ولي أمر معين

```php
$mparent = Mparent::find(10);
$students = StudentHelper::getStudentsByMparent($mparent, [
    'is_active' => true,
    'orderBy' => 'full_name',
]);

foreach ($students as $student) {
    echo $student->full_name . '<br>';
}
```

### 4.6 جلب الطلاب المرتبطين بمستخدم (سواء ولي أمر أو طالب)

```php
$user = AuthHelpers::getCurrentUser();
$students = StudentHelper::getStudentsByUser($user, ['is_active' => true]);

if ($students) {
    foreach ($students as $student) {
        echo $student->full_name . '<br>';
    }
}
```

### 4.7 استخراج معرف الطالب من سجل

```php
$record = StudentRecord::find(100);
$studentId = StudentHelper::getStudentIdFromRecord($record); // أو تمرير 100 مباشرة
if ($studentId) {
    $student = Student::find($studentId);
    echo $student->full_name;
}
```

### 4.8 جلب السجل الدراسي الحالي للطالب

```php
$student = Student::find(5);
$currentRecord = StudentHelper::getCurrentStudentRecord($student, [
    'year_id' => 3, // السنة الدراسية الحالية (اختياري، سيتم جلبها تلقائياً)
]);

if ($currentRecord) {
    echo "الصف: " . $currentRecord->class_id;
    echo "الرسوم: " . $currentRecord->study_fees;
}
```

### 4.9 استخدام دالة `getResultRecord` لجلب نتائج الطلاب

```php
$results = StudentHelper::getResultRecord([
    'year_id' => 3,
    'class_id' => 2,
    'result_status' => 'success', // ناجحين فقط
]);
```

### 4.10 ترفيع الطلاب (تغيير حالة سجلاتهم)

```php
$upgradeResult = StudentHelper::setUpStudentRecord([
    'departments_id' => 1,
    'year_id' => 3,
    'class_id' => 2,
    'change_status_class' => 'auto',
    'record_up_to' => 'success', // فقط الناجحين
]);

if ($upgradeResult['status']) {
    echo "تم ترفيع {$upgradeResult['count_success']} طالباً بنجاح.";
} else {
    echo $upgradeResult['error'];
}
```

---

## 5. التوافق مع الإصدارات السابقة

- **الدالة `getRecords`:** أصبحت تستدعي `StudentRecordsHelper::getRecords` بدلاً من التنفيذ المباشر، ولكن توقيعها ومخرجاتها لم يتغيرا. جميع الأكواد التي كانت تستخدم `StudentHelper::getRecords` ستستمر في العمل.
- **الدالة `getStudentRecord`:** تمت إعادة تسميتها إلى `getStudentRecordV1` لتجنب التعارض مع دوال `StudentRecordsHelper`. إذا كنت تستخدمها في مشروعك، استبدلها بـ `getStudentRecordV1` أو قم بالترقية لاستخدام `getStudentRecordRecords`.
- **الدوال الأخرى (`getResultRecord`, `setUpStudentRecord`, دوال التواريخ):** بقيت دون تغيير للحفاظ على التوافق.

---

## 6. الخلاصة

| الميزة | التفاصيل |
|--------|----------|
| **واجهة موحدة للطلاب وأولياء الأمور** | دوال `getRecords`, `getParentRecords`, `getStudentRecordRecords` تغطي الاحتياجات الأساسية. |
| **ربط المستخدمين** | دوال `get*ByUser` و `getUserBy*` تسهل الانتقال بين الكيانات. |
| **دعم العلاقات العائلية** | دوال `getStudentsByMparent`, `getStudentsByUser` و `getStudentIdsByMparent`. |
| **دعم السجلات الدراسية** | دوال `getCurrentStudentRecord`, `getStudentIdFromRecord`. |
| **توافق عكسي** | الدوال القديمة لا تزال متاحة (باستثناء `getStudentRecord` أعيدت تسميتها). |
| **تكامل مع `StudentRecordsHelper`** | دوال الاستعلام الحديثة تستفيد من التخزين المؤقت، الأحداث، والفلاتر المتقدمة. |

**الاستخدام الموصى به:**

- للمشاريع الجديدة: استخدم `StudentHelper::getRecords`, `getParentRecords`, `getStudentRecordRecords` مع الخيارات المتقدمة.
- للتعامل مع علاقات المستخدمين: استخدم دوال `getStudentByUser`, `getMparentByUser`, `getStudentsByUser`.
- للحصول على السجل الدراسي الحالي: استخدم `getCurrentStudentRecord`.
- إذا كنت بحاجة إلى منطق ترفيع الطلاب: استخدم `setUpStudentRecord`.

---

## 7. الخاتمة

كلاس `StudentHelper` هو بوابة الدخول الرئيسية لجميع عمليات القراءة والربط المتعلقة بالطلاب وأولياء الأمور في نظام Nano البيئي. بفضل إعادة الهيكلة في الإصدار 1.0.13، أصبح الكلاس أكثر قوة ومرونة، مع الحفاظ على التوافق العكسي الكامل. نوصي بالاعتماد على هذا الكلاس في جميع التطبيقات الجديدة وتحديث التطبيقات القديمة للاستفادة من الميزات المتقدمة (خاصة دوال `getParentRecords` و `getStudentRecordRecords`).

**المراجع:**

- [التوثيق الكامل لإضافة `Tss.Student`](./Docs-Student-ar.md)
- [كلاس `StudentRecordsHelper`](./Docs-StudentRecordsHelper-Class-ar.md)
- [كلاس `AccessManager` (Nano.AuthApi)](../AuthApi/Docs-AccessManager-ar.md)
- [كلاس `AdvancedQueryHelper` (Nano2.QueryBuilder)](../querybuilder/Docs-AdvancedQueryHelper.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)
