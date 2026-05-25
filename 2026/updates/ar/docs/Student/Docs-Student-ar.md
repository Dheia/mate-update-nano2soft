# 📘 توثيق إضافة `Tss.Student`

**الإصدار:** 1.0.13  
**المسار:** `plugins/tss/student`  
**الغرض:** إدارة بيانات الطلاب وأولياء الأمور وسجلاتهم الدراسية في نظام NanoSoft App، مع دعم كامل للعمليات المالية (الرسوم) والترفيع، وتكامل مع نظام المستخدمين (`RainLab.User`) ونظام الحسابات المالية (`Tss.Accounts`).

---

## 1. مقدمة

إضافة `Tss.Student` هي المكون الأساسي لإدارة الطلاب وأولياء الأمور في نظام Nano البيئي. توفر الإضافة نماذج (Models) متكاملة للطلاب (`Student`) وأولياء الأمور (`Mparent`) وسجلات الطالب السنوية (`StudentRecord`) مع صلاحيات دقيقة، ودعم للترجمة، وواجهة خلفية متكاملة، وإمكانية ربط المستخدمين (Frontend/Backend) بالطلاب وأولياء الأمور.

تم بناء الإضافة وفقاً لأفضل ممارسات تطبيقات Nano Soft App، وتستخدم `Traits` متقدمة مثل `SoftDelete`, `Sortable`, `Purgeable`، وتدمج مع إضافات أخرى مثل `Tss.Basic`, `Tss.Accounts`, `Tss.School`, `Nano.AuthApi` لتوفير تجربة موحدة.

---

## 2. مميزات الإضافة

| الميزة | الوصف |
|--------|-------|
| **إدارة شاملة للطلاب** | تخزين البيانات الشخصية، معلومات الاتصال، الهوية، العنوان، الصور، الملفات. |
| **إدارة أولياء الأمور** | ربط الطالب بولي أمر واحد (أو أكثر عبر تخصيص العلاقة). |
| **سجلات الطالب الدراسية** | تسجيل الطالب في سنة دراسية، صف، شعبة، مع تتبع الرسوم والخصومات وحالة السجل (مستمر، ناجح، راسب، منقول، منهي). |
| **دعم العمليات المالية** | حساب الرسوم الدراسية، رسوم التسجيل، رسوم الباص، الخصومات، وإنشاء قيود محاسبية تلقائية عبر `Tss.Accounts`. |
| **ربط المستخدمين** | ربط أي نموذج مستخدم (Backend/Frontend) بالطالب أو ولي الأمر عبر `user_id` و `user_type` (Morph). |
| **دعم تعدد اللغات** | ترجمة الحقول النصية الأساسية (`full_name`, `description`, `address`) باستخدام `RainLab.Translate`. |
| **صلاحيات دقيقة** | صلاحيات منفصلة للطلاب وأولياء الأمور (عرض، إضافة، تعديل، حذف، بحث، تصدير، طباعة، SMS، بريد إلكتروني). |
| **واجهة خلفية متكاملة** | قوائم، نماذج، استيراد/تصدير، علاقات، وعمليات ترفيع جماعية للطلاب. |
| **تكامل مع نظام الحسابات** | إنشاء حسابات مالية تلقائية للطلاب وأولياء الأمور، وإنشاء قيود محاسبية لرسوم التسجيل. |
| **دعم واجهات API** | يمكن الوصول إلى جميع البيانات عبر إضافة `Nano.StudentsApi` (منفصلة). |

---

## 3. هيكل قاعدة البيانات

### 3.1 جدول `tss_school_students` (الطلاب)

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `id` | int | المفتاح الأساسي. |
| `code`, `barcode`, `manual_code` | string | رموز تعريفية. |
| `full_name`, `first_name`, `last_name` | string | أسماء الطالب. |
| `gender` | string | الجنس (male/female). |
| `date_of_birth` | timestamp | تاريخ الميلاد. |
| `place_of_birth` | string | مكان الميلاد. |
| `religion_id` | string | الديانة. |
| `nationality`, `language` | string | الجنسية واللغة. |
| `passport_type`, `passport_number`, `passport_issuer`, `passport_release_at`, `passport_expiry_at` | string / timestamp | بيانات جواز السفر. |
| `phone`, `email`, `website`, `mobile` | text / string | معلومات الاتصال (JSON للهواتف والإيميلات). |
| `address`, `address_1`, `address_2`, `street_name`, `street_number` | string | تفاصيل العنوان. |
| `latitude`, `longitude`, `radius`, `geo_components`, `city`, `zip`, `postcode`, `country_long`, `formataddress`, `vicinity` | string / double | بيانات الموقع الجغرافي. |
| `country_id`, `state_id`, `directorate_id` | string | معرفات الدولة والمحافظة والمديرية (قابلة للربط بـ `RainLab.Location`). |
| `parents_id` | string | معرف ولي الأمر (ربط بـ `tss_school_parents`). |
| `parent_type` | string | صلة القرابة (أب، أم، أخ، إلخ). |
| `year_id` | string | السنة الدراسية (ربط بـ `Tss.Studyyear`). |
| `last_school` | string | المدرسة السابقة. |
| `student_status` | string | حالة الطالب (مستمر، منقول، منهي). |
| `regster_date` | timestamp | تاريخ التسجيل. |
| `is_active`, `is_effective`, `is_hidden`, `is_default` | boolean | حالات الطالب. |
| `accounts_id`, `bank_acc_id`, `currencys_id`, `credit_limit`, `default_discount` | string / decimal | بيانات الحسابات المالية. |
| `user_id`, `user_type` | bigInteger / string | ربط المستخدم (Morph). |
| `properties`, `config_data`, `other_data` | text | بيانات إضافية بصيغة JSON. |
| `created_by`, `updated_by`, `deleted_by` | string | معرف المستخدم الذي أجرى العملية. |
| `created_at`, `updated_at`, `deleted_at` | timestamp | تواريخ الإنشاء والتعديل والحذف الناعم. |

> تمت إضافة العديد من الحقول عبر ملفات الهجرة (migrations) مثل `builder_table_add_address_columns_to_students.php`, `builder_table_add_passport_columns.php`, إلخ.

### 3.2 جدول `tss_school_parents` (أولياء الأمور)

هيكل مشابه لجدول الطلاب لكن بأعمدة أقل، مع التركيز على `full_name`, `phone`, `email`, `mobile`, `address`, `user_id`, `user_type`, `accounts_id`.  
العلاقة الأساسية: `hasMany` مع الطلاب عبر `parents_id`.

### 3.3 جدول `tss_school_student_record` (سجلات الطالب الدراسية)

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `id` | int | المفتاح الأساسي. |
| `student_id` | string | معرف الطالب. |
| `year_id` | string | السنة الدراسية. |
| `class_id` | string | الصف الدراسي. |
| `group_id` | string | الشعبة. |
| `semster` | string | الفصل الدراسي (semster1/semster2). |
| `status_class` | string | حالة السجل (continue, successfull, fail, complete, transfer, unconfirmation). |
| `status_mony` | string | حالة الدفع. |
| `study_fees`, `registration_fees`, `bus_fees`, `discount` | decimal | الرسوم الدراسية، التسجيل، الباص، الخصم. |
| `is_discount`, `is_registration_fees`, `is_bus` | boolean | أعلام تفعيل الخصم أو رسوم التسجيل أو الباص. |
| `student_acc`, `discount_acc`, `study_fees_acc`, `registration_fees_acc`, `bus_fees_acc` | string | أرقام الحسابات المالية المستخدمة في القيد. |
| `*_details_id` | text | معرفات تفاصيل القيد المحاسبي (بصيغة `bond_id:detail_id`). |
| `currencys_id`, `rate`, `periods_id`, `cost_centers_id` | string / decimal | بيانات العملة والدورة المالية ومركز التكلفة. |
| `regster_date` | timestamp | تاريخ التسجيل. |
| `notes`, `properties`, `other_data` | text | ملاحظات وبيانات إضافية. |
| `country_id`, `state_id`, `directorate_id` | string | موقع السجل (اختياري). |
| `is_active`, `is_default` | boolean | حالة السجل. |

---

## 4. النماذج (Models)

### 4.1 `Tss\Student\Models\Student`

**الجدول:** `tss_school_students`  
**الـ Traits المستخدمة:** `Validation`, `SoftDelete`, `Sortable`, `Purgeable`, `YearTrait`.  
**الـ Behaviors:** `TranslatableModel`, `LocationModel`, `BasicModel`, `BasicScope`.

**العلاقات الرئيسية:**

```php
public $belongsTo = [
    'Department' => [Department::class, 'key' => 'departments_id'],
    'Parent' => [Mparent::class, 'key' => 'parents_id'],
    'MParents' => [Mparent::class, 'key' => 'parents_id'],
    'StudyYear' => [Period::class, 'key' => 'year_id'],
    'Account' => [Account::class, 'key' => 'accounts_id'],
    'Created_by' => [BackendUser::class, 'key' => 'created_by'],
    // ...
];

public $hasMany = [
    'StudentRecord' => [StudentRecord::class, 'key' => 'student_id'],
    'StudentNote' => [StudentNote::class, 'key' => 'student_id'],
];
```

**الأحداث الرئيسية:**

- `beforeCreate` / `beforeUpdate`: تعيين القيم الافتراضية (الشركة، القسم)، توحيد الاسم، منع التكرار (عبر `checkMatchFullName`)، معالجة رقم الجوال (`preperSaveMobile`)، والتحقق من عدم تكرار المستخدم المرتبط (`preperSaveUser`).
- `afterCreate` / `afterUpdate`: إنشاء حساب مالي (عند تفعيل `is_auto_accounts`) وإنشاء مستخدم (عند تفعيل `is_auto_users`).
- `afterSave`: تعيين رمز افتراضي (`setCodeDefault`).

**دوال النطاقات (Scopes):**

- `scopeByUser`, `scopeWhereByUser`, `scopeWhereNotNullUser`, `scopeWhereNullUser`, `scopeWhereHasUserTrashed`.
- `scopeIsContinue` (لحالة الطالب `continue`).

### 4.2 `Tss\Student\Models\Mparent`

مشابه لـ `Student` ولكن مع علاقات مبسطة.

**العلاقات:**

```php
public $hasMany = [
    'Student' => [Student::class, 'key' => 'parents_id'],
    'Student_count' => [Student::class, 'key' => 'parents_id', 'count' => true],
];
```

**الأحداث:** نفس منطق `Student` (التحقق من الاسم، رقم الجوال، المستخدم).

### 4.3 `Tss\School\Models\StudentRecord`

**الجدول:** `tss_school_student_record`  
**الـ Traits:** `Validation`, `SoftDelete`, `Purgeable`, `Sortable`, `YearTrait`.

**العلاقات:**

```php
public $belongsTo = [
    'Student' => [Student::class, 'key' => 'student_id'],
    'Department' => [Department::class, 'key' => 'departments_id'],
    'RClass' => [TbClass::class, 'key' => 'class_id'],
    'Group' => [Group::class, 'key' => 'group_id'],
    'StudyYear' => [Period::class, 'key' => 'year_id'],
    'CostCenter' => [CostCenter::class, 'key' => 'cost_centers_id'],
    // ...
];
```

**الأحداث الهامة:**

- `beforeCreate` / `beforeUpdate`: التحقق من عدم وجود سجلين بحالة `continue` لنفس الطالب (`checkDoubleContinue`)، ومنع التسجيل المزدوج في نفس الصف بحالة ناجح أو مستمر (`checkDoubleClass`).
- `afterCreate`: استدعاء `createFees()` لإنشاء القيود المحاسبية للرسوم.
- `createFees`: تقوم بإنشاء قيد محاسبي منفصل لكل من: رسوم الدراسة، الخصم، رسوم التسجيل، رسوم الباص (إذا كانت مفعلة). تستخدم `AccountHelper` لإنشاء السندات وتربطها بالحقول `*_details_id`.

**دوال النطاقات:**

- `scopeIsContinue`, `scopeIsActiveContinue`.

---

## 5. المساعدات (Helpers)

### 5.1 `Tss\Student\Helpers\StudentHelper`

الواجهة الرئيسية للمطورين. يحتوي على:

- **`getRecords(array $options, bool $isException = false): array`** – جلب الطلاب (يستدعي `StudentRecordsHelper`).
- **`getParentRecords(array $options, bool $isException = false): array`** – جلب أولياء الأمور مع فلاتر خاصة.
- **`getStudentRecordRecords(array $options, bool $isException = false): array`** – جلب سجلات الطالب.
- دوال الربط بين المستخدمين والطلاب/أولياء الأمور.
- دوال خاصة بـ `Mparent`: `getStudentIdsByMparent`, `getStudentsByMparent`, `getStudentsByUser`.
- دوال خاصة بـ `StudentRecord`: `getStudentIdFromRecord`, `getCurrentStudentRecord`.
- دوال تاريخية (للتوافق): `getStudentRecordV1`, `getResultRecord`, `setUpStudentRecord`, إلخ.

> تم توثيق هذا الكلاس بشكل كامل في ملف `Docs-StudentHelper-Class-ar.md`.

### 5.2 `Tss\Student\Helpers\StudentRecordsHelper`

يحتوي على المنطق الأساسي لجلب السجلات مع دعم الفلاتر المتقدمة، التخزين المؤقت، الأحداث، وتكامل `AccessManager`. يُستخدم داخلياً بواسطة `StudentHelper`.  
> تم توثيقه في `Docs-StudentRecordsHelper-Class-ar.md`.

---

## 6. وحدات التحكم (Controllers – Backend)

### 6.1 `Tss\Student\Controllers\Students`

- يستخدم `ListController`, `FormController`, `ImportExportController`, `RelationController`.
- يدعم استيراد وتصدير بيانات الطلاب (عبر `ImportExportController`).
- يتجاوز دالة `create_onSave` لاستدعاء `autoCreateAccounts` و `autoCreateUsers` بعد حفظ الطالب.
- تطبق `listExtendQuery` صلاحيات الوصول (`access_all` مقابل `access`).

### 6.2 `Tss\Student\Controllers\Parents`

مشابه لـ `Students` لكن بدون استيراد/تصدير. يدعم `ListController`, `FormController`, `ReorderController`.

### 6.3 `Tss\Student\Controllers\StudentRecordUps`

متحكم خاص لترفيع الطلاب (نقلهم من صف إلى آخر أو تغيير حالتهم). يحتوي على دوال AJAX:

- `onRecordCount`: حساب عدد السجلات المتأثرة.
- `onUpStudentRecord`: تنفيذ عملية الترفيع باستخدام `StudentHelper::setUpStudentRecord`.

---

## 7. الإعدادات (Configuration)

ملف `config.php` في `plugins/tss/student/config.php` يتضمن إعدادات لكل من `students`, `parents`, `student_record_ups`.

**مثال للإعدادات الخاصة بالطلاب:**

```php
'students' => [
    'department_type' => env('TSS_STUDENT_STUDENTS_DEPARTMENT_TYPE', 'main'),
    'is_allow_auto_accounts' => env('TSS_STUDENT_STUDENTS_IS_ALLOW_AUTO_ACCOUNTS', true),
    'is_allow_auto_users' => env('TSS_STUDENT_STUDENTS_IS_ALLOW_AUTO_USERS', true),
    'match_full_name' => env('TSS_STUDENT_STUDENTS_MATCH_FULL_NAME', true),
    'sensitive_full_name' => env('TSS_STUDENT_STUDENTS_SENSITIVE_FULL_NAME', true),
    // ...
],
'parents' => [ ... ],
'student_record_ups' => [
    'process_type' => [
        'update_record' => true,
        'update_create_record' => false,
        'update_create_accuonts_record' => false,
    ],
],
```

يمكن تعديل هذه القيم عبر متغيرات البيئة (`.env`) أو مباشرة في الملف.

---

## 8. الصلاحيات (Permissions)

تم تعريف الصلاحيات في `Plugin::registerPermissions()`:

| الصلاحية | الوصف |
|----------|-------|
| `tss.student.students.access` | الوصول إلى قائمة الطلاب (مع تقييد حسب `created_by` إن لم يملك `access_all`). |
| `tss.student.students.access_all` | الوصول إلى كل سجلات الطلاب. |
| `tss.student.students.add`, `edit`, `delete`, `search`, `export`, `print`, `email`, `sms` | صلاحيات العمليات الفردية. |
| `tss.student.parents.*` | نفس الشيء لأولياء الأمور. |
| `tss.student.studentrecordups.manage` | إدارة ترفيع الطلاب. |

تُستخدم هذه الصلاحيات في وحدات التحكم الخلفية (`Students`, `Parents`, `StudentRecordUps`) للتحكم في إظهار الأزرار والسماح بالعمليات.

---

## 9. الأحداث (Events)

الإضافة تطلق الأحداث التالية (يمكن استخدامها من إضافات أخرى):

- `api.list.extendQueryBefore`, `api.list.extendQuery` – تُستخدم في دوال `getRecords` لتوسيع الاستعلام.
- `StudentsApi.ApiControllers.Students.List` – بعد جلب قائمة الطلاب (في API).
- `ParentsApi.ApiControllers.Parents.List` – لقائمة أولياء الأمور.
- `StudentRecordsApi.ApiControllers.StudentRecords.List` – لقائمة سجلات الطلاب.
- داخل نموذج `StudentRecord`: `afterCreate` لا يطلق حدثاً صريحاً، لكن يمكن استدعاء الأحداث عبر `Event::fire` في `createFees` إذا رغبت.

---

## 10. التوافق والاعتماديات

- **يتطلب وجود الإضافات التالية** (مدرجة في `$require`):
  - `Rainlab.Translate`
  - `Rainlab.Location`
  - `Rainlab.User`
  - `Tss.Tools`
  - `Tss.BarcodeGenerator`
  - `Tss.Basic`
  - `Tss.Accounts`
  - `Tss.School`
  - `Tss.Studyyear`
- **للحصول على كامل الوظائف API**، تحتاج إلى تثبيت `Nano.StudentsApi`.
- **للاستفادة من دوال الصلاحيات المتقدمة** (مثل `AccessManager`) يوصى بتثبيت `Nano.AuthApi`.

---

## 11. الخاتمة

إضافة `Tss.Student` هي حجر الأساس لإدارة الطلاب وأولياء الأمور في أي نظام يعتمد على تقنيات Nano Soft App. بفضل تصميمها المعياري، وصلاحياتها المرنة، وتكاملها مع أنظمة الحسابات والمستخدمين، يمكن استخدامها في مشاريع المدارس والجامعات ومراكز التدريب بسهولة.

للاستفادة القصوى من الإضافة، يُنصح بالاطلاع على الوثائق التالية:

- [توثيق كلاس `StudentHelper`](./Docs-StudentHelper-Class-ar.md)
- [توثيق كلاس `StudentRecordsHelper`](./Docs-StudentRecordsHelper-Class-ar.md)
- [توثيق إضافة `Nano.StudentsApi` (API)](../StudentsApi/Docs-StudentsApi-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](../mcp/Nano-Api-SKILL.md)

