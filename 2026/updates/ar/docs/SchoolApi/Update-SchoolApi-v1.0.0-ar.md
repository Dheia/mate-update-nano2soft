## 2026-05-08 - 2026-05-10

**إضافة `Nano.SchoolApi` – الإصدار 1.0.0**

### إنشاء واجهة برمجة تطبيقات RESTful شاملة لإدارة بيانات المدرسة الأساسية

---

### ملخص التحديثات

يقدم الإصدار الأول **1.0.0** من إضافة `Nano.SchoolApi` واجهة برمجة تطبيقات متكاملة للاستعلام عن جميع الكيانات الأساسية التي تديرها إضافة `Tss.School`. تم بناء الإضافة وفقاً لنمط `Nano.*Api` المعتمد في المشروع، مع توفير نقاط نهاية محمية لاسترجاع بيانات الصفوف (`Classes`)، المراحل (`Stages`)، المواد (`Subjects`)، الشعب (`Groups`)، توزيع الشعب على الصفوف (`ClassGroups`)، توزيع المواد على الصفوف (`ClassSubjects`)، توزيع المعلمين على المواد (`TeacherSubjects`)، ربط مراكز التكلفة بالصفوف (`ClassCostCenters`)، أنواع الرسوم (`FeesTypes`)، أنواع الوثائق (`DocumentTypes`)، المدارس الخارجية (`OutSchools`)، وثائق الطلاب (`StudentDocuments`)، رسوم الطلاب (`StudentsFees`) وعلامات آخر سنة (`LastStudentMarks`). يدعم الإصدار نظام صلاحيات متعدد المستويات لكل مورد، ومحولات بيانات احترافية مع دعم كامل للترجمة.

---

### أهداف الإصدار

- **توفير API موحد** لجميع كيانات `Tss.School` الحيوية، مما يفتح البيانات لاستخدامات متعددة.
- **تطبيق نمط معماري متناسق** مع إضافات `Nano.*Api` الأخرى لتسهيل الصيانة والتوسع.
- **تمكين التطبيقات الخارجية والواجهات الأمامية** من الوصول إلى بيانات المدرسة (الصفوف، المواد، الجداول، الرسوم، الوثائق) بسهولة وأمان.
- **توفير تحكم دقيق في صلاحيات الوصول** لكل مورد، مما يسمح بنشر الإضافة في بيئات إنتاجية مع صلاحيات متفاوتة.
- **دعم التصفية المتقدمة والبحث والترتيب** لتلبية احتياجات التقارير والتكاملات.
- **تسهيل بناء تقارير وإحصائيات** تعتمد على هذه البيانات دون الحاجة للوصول المباشر لقاعدة البيانات.

---

### الميزات الجديدة والتحسينات

#### 1. وحدات تحكم RESTful شاملة (14 متحكم)

تم إنشاء 14 وحدة تحكم تغطي جميع الكيانات الأساسية لإضافة `Tss.School`، وكلها تدعم عمليات القراءة (`list` و `show`):

| المتحكم | النموذج المستهدف | الوصف |
|--------|-----------------|-------|
| `Classes` | `TbClass` | الصفوف الدراسية (مع الرسوم والترتيب والمرحلة) |
| `Stages` | `Stage` | المراحل الدراسية |
| `Subjects` | `Subject` | المواد الدراسية |
| `Groups` | `Group` | الشعب الدراسية |
| `ClassGroups` | `ClassGroup` | ربط الشعب بالصفوف وسنة دراسية |
| `ClassSubjects` | `ClassSubject` | توزيع المواد على الصفوف (الوحدات، الدرجات) |
| `TeacherSubjects` | `TeacherSubject` | توزيع المعلمين على المواد والشعب |
| `ClassCostCenters` | `ClassCostCenter` | ربط مراكز التكلفة بالصفوف |
| `FeesTypes` | `FeesType` | أنواع الرسوم الدراسية |
| `DocumentTypes` | `DocumentType` | أنواع الوثائق المطلوبة |
| `OutSchools` | `OutSchool` | المدارس الخارجية |
| `StudentDocuments` | `StudentDocument` | وثائق الطلاب (حالة التسليم) |
| `StudentsFees` | `StudentsFee` | رسوم الطلاب (المبلغ، الخصم) |
| `LastStudentMarks` | `LastStudentMark` | علامات آخر سنة دراسية للطالب |

**كل متحكم يقدم:**

- **`index()`**: جلب قائمة بالسجلات مع تصفية متعددة وبحث وترتيب وتقسيم صفحات.
- **`show($id)`**: عرض تفاصيل سجل واحد مع العلاقات.
- **`getRecords(array $options)`**: دالة أساسية لبناء استعلامات معقدة مع دعم إعدادات افتراضية من `config.php`.
- **`activelystats()`**: نقطة نهاية لمراقبة آخر تحديث (لأغراض التخزين المؤقت).
- **`getLastUpdateAt()`**: إرجاع الطابع الزمني لآخر تحديث.
- **`validationList($user)`**: دالة موحدة للتحقق من صلاحيات الوصول.

#### 2. نظام صلاحيات محكم لكل مورد

تم تطبيق نظام صلاحيات منفصل لكل من الموارد الأربعة عشر، مع تحكم كامل عبر إعدادات `config.php` ومتغيرات البيئة (`env`). كل مورد يمتلك:

- `is_allow_list`: تفعيل/تعطيل عملية الجرد.
- `is_allow_list_backend` / `is_allow_list_frontend`: السماح لمستخدمي الخلفية أو الواجهة الأمامية.
- `is_check_access_list`: هل يجب التحقق من وجود مستخدم صحيح.
- `is_check_list_permission`: هل يجب التحقق من صلاحية محددة (مثل `tss.school.tbclasses.access`).

بالإضافة إلى إعدادات التصفية: `order_by`, `order_dir`, `per_page`, `exclude`.

**مثال من `config.php`:**
```php
'classes' => [
    'is_allow_list'            => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_BACKEND', true),
    'is_allow_list_frontend'   => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND', true),
    'is_check_access_list'     => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_ACCESS_LIST', true),
    'is_check_list_permission' => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_LIST_PERMISSION', true),
    'order_by'                 => env('NANO_SCHOOLAPI_CLASSES_ORDER_BY', 'sort_order'),
    'order_dir'                => env('NANO_SCHOOLAPI_CLASSES_ORDER_DIR', 'asc'),
    'per_page'                 => env('NANO_SCHOOLAPI_CLASSES_PER_PAGE', 15),
    'exclude'                  => env('NANO_SCHOOLAPI_CLASSES_EXCLUDE', ''),
],
```

#### 3. دعم ذكي للقيم الافتراضية للموارد المعتمدة على السنة الدراسية

العديد من الموارد (مثل `ClassGroups`, `TeacherSubjects`, `ClassCostCenters`, `StudentDocuments`, `StudentsFees`) ترتبط بسنة دراسية (`year_id`). عند عدم تمرير هذه القيمة، تقوم المتحكمات تلقائياً باعتماد **السنة الدراسية الافتراضية** (`Period::getPrimary()`) مما يضمن تجربة سلسة دون أخطاء.

**مثال من `ClassGroups::getRecords()`:**
```php
if (!$year_id) {
    $primary = Period::getPrimary();
    if ($primary) {
        $year_id = $primary->id;
    }
}
```

#### 4. محولات بيانات شاملة (14 محول)

تم إنشاء محول بيانات لكل نموذج، تقوم بتنسيق الاستجابة وتوفير العلاقات المضمنة مع معالجة الأخطاء:

- **`ClassTransformer`**: الصفوف مع `company`, `department`, `stage`.
- **`StageTransformer`**: المراحل مع `company`, `department`.
- **`SubjectTransformer`**: المواد مع `company`, `department`.
- **`GroupTransformer`**: الشعب مع `company`, `department`.
- **`ClassGroupTransformer`**: ربط الشعب مع `company`, `department`, `class`, `group`, `study_year`.
- **`ClassSubjectTransformer`**: توزيع المواد مع `company`, `department`, `class`, `subject`.
- **`TeacherSubjectTransformer`**: توزيع المعلمين مع `company`, `department`, `class`, `group`, `subject`, `teacher`, `study_year`.
- **`ClassCostCenterTransformer`**: ربط مراكز التكلفة مع `company`, `department`, `class`, `cost_center`, `study_year`.
- **`FeesTypeTransformer`**: أنواع الرسوم مع `company`, `department`.
- **`DocumentTypeTransformer`**: أنواع الوثائق مع `company`, `department`.
- **`OutSchoolTransformer`**: المدارس الخارجية مع `company`, `department`.
- **`StudentDocumentTransformer`**: وثائق الطلاب مع `company`, `department`, `student`, `documents`, `study_year`.
- **`StudentsFeeTransformer`**: رسوم الطلاب مع `company`, `department`, `student`, `student_record`, `fees_type`, `study_year`.
- **`LastStudentMarkTransformer`**: علامات آخر سنة مع `company`, `department`, `student`, `class`, `out_school`, `student_record`, `study_year`.

**ميزات المحولات:**
- تنسيق القيم إلى الأنواع الصحيحة (int, string, bool, float, date).
- دالة `formatDate` لتحويل التواريخ بشكل آمن.
- تصفية الحقول (`exclude`) عبر querystring للتحكم في حجم البيانات المرتجعة.
- إضافة `object_type` للتعرف على نوع الكائن.
- كل علاقة محاطة بـ `try-catch` لتجنب فشل الاستعلام الكامل.

#### 5. تسجيل نطاق `scopeExclude` لجميع النماذج

تم تسجيل النطاق الديناميكي `scopeExclude` لجميع النماذج الأربعة عشر داخل `Plugin::boot()`، مما يسمح للعملاء بطلب أعمدة محددة فقط وتقليل البيانات المنقولة عبر الشبكة.

```php
foreach ($models as $model) {
    $model::extend(function($model) {
        $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
            $getTable = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
            return $query->select(array_diff($getTable, (array) $columns));
        });
    });
}
```

#### 6. دعم التخزين المؤقت

جميع نقاط النهاية `index` تدعم التخزين المؤقت عبر `nano.api::api_enable_cache`، مع إبطال الكاش تلقائياً عند تحديث أي سجل.

#### 7. ترجمة شاملة

ملف `lang/en/lang.php` يحتوي على جميع رسائل النجاح والخطأ والصلاحيات لكل مورد، مما يسهل ترجمتها إلى أي لغة.

---

### أمثلة على الاستخدام

#### جلب قائمة الصفوف النشطة لمرحلة معينة

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes?stage_id=2&isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### جلب توزيع المعلمين على المواد لسنة معينة مع تضمين المعلم والمادة

```bash
curl -X GET "https://yourdomain.com/api/v1/school/teacher-subjects?year_id=5&include=teacher,subject" \
  -H "Authorization: Bearer <token>"
```

#### جلب رسوم طالب محدد

```bash
curl -X GET "https://yourdomain.com/api/v1/school/students-fees?student_id=15" \
  -H "Authorization: Bearer <token>"
```

#### البحث عن مادة دراسية بالاسم

```bash
curl -X GET "https://yourdomain.com/api/v1/school/subjects?q=رياضيات" \
  -H "Authorization: Bearer <token>"
```

#### عرض تفاصيل صف دراسي مع المرحلة

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes/3?include=stage" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تكامل غير مسبوق**: ولأول مرة، يمكن لأي نظام خارجي (تطبيق جوال، موقع أولياء الأمور، لوحات معلومات) الوصول إلى جميع بيانات المدرسة عبر API واحد.
- **تسريع بناء التقارير**: يمكن للمطورين الآن إنشاء تقارير مخصصة عن الصفوف والمواد والرسوم والوثائق باستخدام استعلامات API بسيطة بدلاً من كتابة استعلامات SQL معقدة.
- **تحسين تجربة المستخدم**: الواجهات الأمامية يمكنها جلب القوائم المنسدلة (مثل الصفوف والمراحل والمواد) بشكل ديناميكي من الخادم.
- **جاهزية عالية للإنتاج**: نظام الصلاحيات المتقدم يسمح بالتحكم الدقيق في وصول كل مستخدم، مما يجعل الإضافة مناسبة للنشر في بيئات حقيقية.
- **تصميم قابل للتوسع**: يمكن إضافة عمليات `POST` و `PUT` لأي مورد مستقبلاً بسهولة دون تغيير الهيكل الأساسي.
- **أداء ممتاز**: استخدام `scopeExclude` مع التخزين المؤقت يضمن استجابات سريعة حتى مع آلاف السجلات.

---

### متطلبات الترقية

1. **تثبيت الإضافة**: انسخ مجلد `nano/schoolapi` إلى `plugins/nano/schoolapi`.
2. **تسجيل الإضافة**: شغّل `php artisan plugin:refresh Nano.SchoolApi`.
3. **المتغيرات البيئية**: اضبط متغيرات `env` حسب الحاجة (مثل `NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND`).
4. **الصلاحيات**: تأكد من أن الأدوار تمتلك صلاحيات الوصول المناسبة في النظام الخلفي إذا تم تفعيل `is_check_list_permission`.
5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **لا توجد هجرات قاعدة بيانات** مطلوبة.

---

### الخاتمة

يمثل الإصدار **1.0.0** من `Nano.SchoolApi` نقلة نوعية في إتاحة بيانات المدرسة عبر API. بتغطية 14 كياناً أساسياً، توفر الإضافة قاعدة متينة لبناء أنظمة متكاملة تعتمد على بيانات `Tss.School` دون عناء. التصميم الموحد وقابلية التوسع العالية يجعلانها منصة مثالية لإضافة المزيد من العمليات والتقارير في المستقبل.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano.StudyyearApi`](./docs/StudyyearApi/Docs-StudyyearApi-ar.md)
- [توثيق إضافة `Nano.SchoolApi`](./docs/SchoolApi/Docs-SchoolApi-ar.md)
- [توثيق إضافة `Nano.StudentsApi`](./docs/StudentsApi/Docs-StudentsApi-ar.md)