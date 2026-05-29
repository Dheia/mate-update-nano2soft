## 2026-05-27 - 2026-05-30

**إضافة `Nano.HomeworkApi` – الإصدار 1.0.0**

### إنشاء واجهة برمجة تطبيقات RESTful لإدارة الواجبات والتسليمات والتصنيفات

---

### ملخص التحديثات

يقدم الإصدار الأول **1.0.0** من إضافة `Nano.HomeworkApi` واجهة برمجة تطبيقات متكاملة للتفاعل مع نظام `Tss.Homework`، بما يشمل:
- **الواجبات والأنشطة** (`Homeworks`) – إنشاء وتحديث وحذف وعرض مع دعم الواجبات العامة لكل الصفوف (student_id = '*')
- **التسليمات والتقييمات** (`Feedbacks`) – رصد درجات الطلاب وملاحظاتهم
- **تصنيفات الواجبات** (`Categories`) – هيكل شجري للتصنيفات
- **أنواع التصنيفات العامة** (`ClassTypes`) – الأنواع المرجعية
- **أنواع التصنيفات الخاصة بالشركات** (`ClassTypeCompanies`) – مرتبطة بالشركة والفرع

تم بناء الإضافة وفقاً لأحدث معايير `Nano-Api-SKILL.md`، مع الاستفادة الكاملة من `AccessManager` للصلاحيات، و`AdvancedQueryHelper` للتصفية الديناميكية، ومحولات البيانات (Transformers)، ونظام تخزين مؤقت وأحداث موحد.

---

### أهداف الإصدار

- **توفير API موحد** لإدارة كافة كيانات الواجبات المنزلية.
- **تمكين التطبيقات الخارجية** (تطبيقات الطلاب، أولياء الأمور، لوحات المعلمين) من التفاعل مع الواجبات والتسليمات.
- **دعم السيناريوهات المتقدمة** مثل الواجبات العامة لكل الصف (student_id = '*') أو لكل الشعبة (group_id = '*').
- **تطبيق نظام صلاحيات مرن** يتحكم في عمليات القراءة والكتابة لكل نوع مستخدم (backend/frontend/guest).
- **توفير مرشحات متقدمة** (is_or, is_not, is_force, is_or_null) للاستعلامات المعقدة.
- **تسهيل التوسع المستقبلي** بفضل الأحداث وهيكلية الكود الموحدة.

---

### الميزات الجديدة والتحسينات

#### 1. وحدات تحكم RESTful كاملة

تم إنشاء خمس وحدات تحكم تغطي جميع الكيانات المطلوبة:

| المتحكم | النموذج المستهدف | العمليات المدعومة |
|---------|-----------------|-------------------|
| `Homeworks` | `Tss\Homework\Models\Homework` | قائمة، عرض، إنشاء، تحديث، حذف |
| `Feedbacks` | `Tss\Homework\Models\Feedback` | قائمة، عرض، إنشاء، تحديث |
| `Categories` | `Tss\Homework\Models\Categorie` | قائمة، عرض |
| `ClassTypes` | `Tss\Homework\Models\ClassType` | قائمة، عرض |
| `ClassTypeCompanies` | `Tss\Homework\Models\ClassTypeCompany` | قائمة، عرض |

**كل متحكم يدعم:**
- **`index()`**: جلب قائمة مع تصفية وبحث وترتيب وتقسيم صفحات.
- **`show($id)`**: عرض تفاصيل عنصر مع تضمين العلاقات المطلوبة.
- **`getRecords(array $options)`**: دالة أساسية لبناء استعلامات معقدة (تستخدم داخلياً).
- **`activelystats()`**: نقطة نهاية لإرجاع آخر تحديث (للتخزين المؤقت).
- **`store()` و `update()` و `destroy()`** (حسب الحاجة).

#### 2. دعم الواجبات العامة لكل الصف أو لكل الشعبة

تم تنفيذ منطق خاص في `Homeworks::getRecords()`:
- إذا تم تمرير `student_id` بقيمة محددة (ليست `'*'` ولا `null`)، يتم تضمين السجلات التي يكون `student_id` مساوياً لها **أو** `'*'` **أو** `null`.  
  → هذا يعني أن الواجب الذي تم إنشاؤه بـ `student_id = '*'` يظهر لجميع الطلاب عند الاستعلام عن طالب معين.
- نفس المنطق ينطبق على `group_id` و `record_id`.

**مثال:**
```php
// عند إنشاء واجب لكل طلاب الصف
$data = ['student_id' => '*', ...];

// عند جلب واجبات الطالب رقم 15
GET /api/v1/homework/homeworks?student_id=15
// سيعيد الواجب المخصص للطالب 15 + الواجب العام (*)
```

#### 3. نظام صلاحيات مركزي ومتعدد المستويات

تم تعريف جميع إعدادات الصلاحيات في ملف `config.php` لكل عملية ولكل مورد، مع إمكانية التحكم عبر متغيرات البيئة.

**مثال من إعدادات `Homeworks.list`:**
```php
'homeworks' => [
    'list' => [
        'permission' => [
            'is_allow' => env('NANO_HOMEWORKAPI_HOMEWORKS_LIST_IS_ALLOW', true),
            'backend' => [
                'allow' => true,
                'check_permission' => true,
                'permissions' => ['tss.homework.homeworks.access_all', 'tss.homework.homeworks.access'],
                'access_scope' => 'all',
            ],
            'frontend' => [
                'allow' => true,
                'allowed_ref_types' => ['student', 'parent'],
                'access_scope' => 'own',
            ],
        ],
    ],
],
```

يتم التحقق من الصلاحيات عبر `AccessManager`:
```php
$access = AccessManager::checkByResource('nano.homeworkapi::homeworks.list', $user);
if (!$access['allowed']) return $this->errorUnauthorized($access['message']);
```

#### 4. فلاتر متقدمة مع `advanced_filters`

تمت إضافة مصفوفة `advanced_filters` تمكن من:
- تحديد الحقول التي يسمح باستخدام `is_or`, `is_not`, `is_or_null` عليها.
- قواعد خاصة لكل نوع مستخدم (backend/frontend) وحتى لكل `ref_type` (student/parent).

**مثال لتقييد `student_id` للطلاب العاديين وحلّها لأولياء الأمور:**
```php
'advanced_filters' => [
    'field_specific_rules' => [
        'student_id' => [
            'frontend' => [
                '*' => false,
                'parent' => true,
            ],
        ],
    ],
],
```

#### 5. محلل ديناميكي لـ `student_id` و `record_id` (`frontend_resolver`)

تم إضافة مقطع `frontend_resolver` يقوم تلقائياً:
- إذا كان المستخدم **طالباً** (ref_type = student): يقوم بتعبئة `student_id` من المستخدم الحالي وجلب `record_id` الحالي للطالب.
- إذا كان المستخدم **ولي أمر** (ref_type = parent): يتحقق من وجود `student_id` أو `record_id` في الطلب، ويتحقق من ملكية الطالب لولي الأمر، ثم يملأ القيم المفقودة.

**التطبيق في المتحكم:**
```php
$options = AccessManager::instance()->resolveDynamicFrontendOptions(
    $user, $options, null, 'nano.homeworkapi::homeworks.list'
);
```

#### 6. دالة `getRecords` موحدة وغنية بالميزات

كل متحكم يمتلك دالة `getRecords` متطابقة في الهيكل وتدعم:
- **فلاتر متقدمة** عبر `AdvancedQueryManager::scopeWhereField` مع `is_or`, `is_not`, `is_force`, `is_or_null`.
- **نطاق الوصول** (`applyAccessScope`) للشركة والقسم والولاية ومنشئ السجل.
- **استبعاد الأعمدة** (`exclude`) لتقليل حجم البيانات.
- **تضمين العلاقات** (`custom_with`).
- **البحث النصي** (`q`) مع دعم البحث المتقدم (ArPhpHelper).
- **الترتيب** (`orderBy`, `orderDirection`) والمجموعات (`group_by`) و`having`.
- **خيارات الإخراج**: `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`.
- **الأحداث**: `api.list.extendQueryBefore` و `api.list.extendQuery` لتوسيع الاستعلامات.
- **التخزين المؤقت** عبر `$this->cached()`.

#### 7. محولات البيانات (Transformers)

تم إنشاء خمسة محولات:
- **`HomeworkTransformer`**: يعرض بيانات الواجب مع العلاقات (company, department, category, type_class, student, record, subject, teacher).
- **`FeedbackTransformer`**: يعرض بيانات التسليمة مع العلاقات (homework, student, record).
- **`CategoryTransformer`**: يعرض بيانات التصنيف مع العلاقات (company, department, type_class, parent).
- **`ClassTypeTransformer`**: يعرض أنواع التصنيفات العامة.
- **`ClassTypeCompanyTransformer`**: يعرض تصنيفات الشركات مع العلاقات (company, department, class_type).

كل المحولات تدعم:
- تنسيق التواريخ.
- استبعاد الحقول (`exclude`).
- معالجة الأخطاء في العلاقات عبر `try-catch` (تعيد مصفوفة فارغة بدلاً من فشل الطلب).

#### 8. استجابة موحدة لعمليات الإنشاء والتحديث

تتبع عمليات `store` و `update` هيكل استجابة موحد:
```json
{
  "data": {
    "code": 200,
    "status": true,
    "message": "Homework created successfully.",
    "error": null,
    "errors": null,
    "data": { ... },
    "model": { ... },
    "input_data": { ... },
    "process_data": { ... },
    "debug": { ... }  // فقط في وضع التطوير
  }
}
```

#### 9. دعم القيم الافتراضية الذكية

في عمليات الإنشاء، إذا لم يتم تمرير `companys_id` أو `departments_id` أو `year_id`، يتم تعبئتها تلقائياً من القيم الافتراضية للنظام:
- `companys_id` ← `BasicHelper::getCompanysId(true)`
- `departments_id` ← `BasicHelper::getMainDepartmentId(true)`
- `year_id` ← `Period::getPrimary()->id`

#### 10. تسجيل نطاق `scopeExclude`

تم تسجيل النطاق الديناميكي `scopeExclude` لجميع النماذج الخمسة في `Plugin::boot()`:
```php
\Tss\Homework\Models\Homework::extend(function($model) {
    $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
        $tableColumns = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
        return $query->select(array_diff($tableColumns, (array) $columns));
    });
});
// وكذلك لـ Feedback, Categorie, ClassType, ClassTypeCompany
```

#### 11. دعم كامل للترجمة

جميع رسائل النجاح والخطأ والصلاحيات موجودة في ملفات الترجمة (`lang/en/lang.php` و `lang/ar/lang.php`) وهي جاهزة للتعريب أو الترجمة للغات أخرى.

---

### نقاط النهاية المدعومة

| المورد | الطرق المدعومة | المسارات |
|--------|---------------|----------|
| **Homeworks** | GET, POST, PUT, DELETE | `/homeworks`, `/homeworks/{id}`, `/homeworks/activelystats` |
| **Feedbacks** | GET, POST, PUT | `/feedbacks`, `/feedbacks/{id}`, `/feedbacks/activelystats` |
| **Categories** | GET | `/categories`, `/categories/{id}`, `/categories/activelystats` |
| **ClassTypes** | GET | `/class-types`, `/class-types/{id}`, `/class-types/activelystats` |
| **ClassTypeCompanies** | GET | `/class-type-companies`, `/class-type-companies/{id}`, `/class-type-companies/activelystats` |

**قاعدة المسار الأساسية:** `/api/v1/homework`

---

### أمثلة على الاستخدام

#### 1. جلب قائمة الواجبات لطالب محدد مع تضمين المادة والمعلم

```bash
curl -X GET "https://yourdomain.com/api/v1/homework/homeworks?student_id=15&include=subject,teacher" \
  -H "Authorization: Bearer <token>"
```

#### 2. إنشاء واجب جديد لكل طلاب الصف (student_id = '*')

```bash
curl -X POST "https://yourdomain.com/api/v1/homework/homeworks" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "واجب الرياضيات الأسبوعي",
    "departments_id": "1",
    "ref_type_class": "homework",
    "year_id": "5",
    "semster": "semster2",
    "class_id": "3",
    "subject_id": "21",
    "student_id": "*",
    "min_degree": 5,
    "max_degree": 10,
    "description": "حل تمارين الصفحات 10-15"
  }'
```

#### 3. تحديث درجة واجب

```bash
curl -X PUT "https://yourdomain.com/api/v1/homework/homeworks/42" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"max_degree": 12}'
```

#### 4. حذف واجب

```bash
curl -X DELETE "https://yourdomain.com/api/v1/homework/homeworks/42" \
  -H "Authorization: Bearer <token>"
```

#### 5. تسليم واجب من قبل طالب (إنشاء Feedback)

```bash
curl -X POST "https://yourdomain.com/api/v1/homework/feedbacks" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "homeworks_id": "42",
    "student_id": "15",
    "record_id": "8",
    "degree": 9.5,
    "is_recive": true,
    "note": "تم التسليم بنجاح"
  }'
```

#### 6. جلب التصنيفات الرئيسية (بدون أب)

```bash
curl -X GET "https://yourdomain.com/api/v1/homework/categories?parent_id=null" \
  -H "Authorization: Bearer <token>"
```

#### 7. استخدام `is_or` للبحث عن عدة صفوف

```bash
curl -X GET "https://yourdomain.com/api/v1/homework/homeworks?class_id=3,5&is_or_class_id=true" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **إدارة لا مركزية للواجبات**: يمكن للمعلمين تسجيل الواجبات عبر API دون الدخول إلى لوحة التحكم.
- **تكامل مع تطبيقات الطلاب**: يمكن للطلاب عرض واجباتهم وتقديم التسليمات مباشرة.
- **دور أولياء الأمور**: يمكن لولي الأمر متابعة واجبات أبنائه والتأكيد على استلامها.
- **مرونة عالية**: بفضل الفلاتر المتقدمة والواجبات العامة، يمكن بناء أنظمة ذكية للتوزيع التلقائي للمهام.
- **أداء ممتاز**: استبعاد الأعمدة غير الضرورية والتخزين المؤقت يضمن استجابات سريعة.
- **قابلية للتوسع**: بفضل الأحداث، يمكن للإضافات الأخرى تعديل سلوك الـ API دون تعديل الكود الأساسي.

---

### متطلبات التثبيت والترقية

1. **تثبيت الإضافة**: انسخ مجلد `nano/homeworkapi` إلى `plugins/nano/homeworkapi`.
2. **تسجيل الإضافة**:
   ```bash
   php artisan plugin:refresh Nano.HomeworkApi
   ```
3. **إعدادات البيئة**: أضف متغيرات البيئة المطلوبة في ملف `.env` (راجع `config/config.php` لجميع الخيارات).
   ```env
   NANO_HOMEWORKAPI_HOMEWORKS_LIST_IS_ALLOW=true
   NANO_HOMEWORKAPI_HOMEWORKS_CREATE_FRONTEND_ALLOW=false
   # ... إلخ
   ```
4. **الصلاحيات**: تأكد من أن الأدوار المناسبة تمتلك الصلاحيات مثل `tss.homework.homeworks.add`, `tss.homework.homeworks.edit` إلخ.
5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **لا توجد هجرات قاعدة بيانات** مطلوبة (تعتمد الإضافة على جداول `Tss.Homework` الموجودة).

---

### الخاتمة

يمثل الإصدار **1.0.0** من `Nano.HomeworkApi` إضافة متكاملة وقوية لإدارة الواجبات المنزلية والتسليمات عبر API. بفضل اتباع أحدث معايير Nano API، توفر الإضافة أماناً عالياً، مرونة في التصفية، وسهولة في التوسع. جميع نقاط النهاية جاهزة للاستخدام من قبل تطبيقات الويب، تطبيقات الجوال، وأي نظام خارجي يحتاج إلى التفاعل مع نظام الواجبات.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano.HomeworkApi`](./docs/HomeworkApi/Docs-HomeworkApi-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)
- [توثيق كلاس `AccessManager`](./docs/AuthApi/Docs-AccessManager-ar.md)
- [توثيق كلاس `AdvancedQueryHelper`](./docs/querybuilder/Docs-AdvancedQueryHelper.md)
