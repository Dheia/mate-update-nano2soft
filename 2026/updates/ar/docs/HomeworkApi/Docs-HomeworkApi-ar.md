## 📘 توثيق واجهة برمجة تطبيقات Nano.HomeworkApi

**الإصدار:** 1.0.0  
**الفئة المستهدفة:** مطورو تطبيقات الواجهة الأمامية، مطورو التكاملات الخارجية.

---

### المحتويات

1. [مقدمة](#مقدمة)
2. [المصادقة (Authentication)](#المصادقة)
3. [هيكل الاستجابة العام](#هيكل-الاستجابة-العام)
4. [قائمة نقاط النهاية (Endpoints)](#قائمة-نقاط-النهاية)
   - [الواجبات والأنشطة (Homeworks)](#1-الواجبات-والأنشطة-homeworks)
     - [جلب قائمة الواجبات](#جلب-قائمة-الواجبات)
     - [عرض واجب محدد](#عرض-واجب-محدد)
     - [إنشاء واجب جديد](#إنشاء-واجب-جديد)
     - [تحديث واجب](#تحديث-واجب)
     - [حذف واجب](#حذف-واجب)
   - [التسليمات والتقييمات (Feedbacks)](#2-التسليمات-والتقييمات-feedbacks)
   - [تصنيفات الواجبات (Categories)](#3-تصنيفات-الواجبات-categories)
   - [أنواع التصنيفات العامة (ClassTypes)](#4-أنواع-التصنيفات-العامة-classtypes)
   - [أنواع التصنيفات الخاصة بالشركات (ClassTypeCompanies)](#5-أنواع-التصنيفات-الخاصة-بالشركات-classtypecompanies)
   - [إحصائيات النشاط (Actively Stats)](#6-إحصائيات-النشاط-actively-stats)
5. [معلمات الطلب العامة](#معلمات-الطلب-العامة)
6. [تنسيق التواريخ](#تنسيق-التواريخ)
7. [العلاقات المضمنة (Includes)](#العلاقات-المضمنة)
8. [رموز الأخطاء والتعامل معها](#رموز-الأخطاء-والتعامل-معها)

---

## مقدمة

توفر واجهة برمجة تطبيقات `Nano.HomeworkApi` نقاط نهاية RESTful للوصول إلى بيانات الواجبات المنزلية والأنشطة، وتسليمات الطلاب وتقييماتهم، بالإضافة إلى التصنيفات وأنواعها في نظام `Tss.Homework`. يمكن لمطوري تطبيقات الواجهة الأمامية والتطبيقات الخارجية استخدام الـ API لإدارة الواجبات، رصد تسليمات الطلاب، وتصفية البيانات بشكل متقدم.

**Base URL:**  
`https://yourdomain.com/api/v1/homework`

---

## المصادقة

جميع نقاط النهاية محمية وتتطلب **رمز حامل (Bearer token)** صالحاً. يتم تمرير الرمز في ترويسة `Authorization` بصيغة:

```
Authorization: Bearer <your_token>
```

يتم التحقق من المستخدم ونوعه (backend / frontend) لتطبيق صلاحيات الوصول المحددة لكل نقطة نهاية وعملية. تختلف الصلاحيات المطلوبة حسب العملية (قراءة، إنشاء، تحديث، حذف) ويتم ضبطها إعدادياً عبر متغيرات البيئة.

---

## هيكل الاستجابة العام

### استجابة ناجحة (قائمة)

```json
{
  "data": [
    {
      "id": 42,
      "name": "واجب الرياضيات الأسبوعي",
      "student_id": "15",
      "student_name": "أحمد محمد",
      ...
      "object_type": "Tss\\Homework\\Models\\Homework"
    }
  ],
  "meta": {
    "pagination": {
      "total": 320,
      "count": 15,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 22,
      "links": {
        "next": "https://...?page=2",
        "previous": null
      }
    }
  }
}
```

### استجابة ناجحة (عنصر واحد / عملية كتابة)

```json
{
  "data": {
    "id": 42,
    "name": "واجب الرياضيات الأسبوعي",
    ...
  }
}
```

### استجابة عملية كتابة (إنشاء / تحديث) بهيكل موحد

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
    "input_data": {...},
    "process_data": {...}
  }
}
```

### استجابة خطأ

```json
{
  "error": {
    "code": 400,
    "message": "Listing homeworks is disabled for frontend users.",
    "errors": []
  }
}
```

---

## قائمة نقاط النهاية

### 1. الواجبات والأنشطة (Homeworks)

#### جلب قائمة الواجبات

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/homework/homeworks` | جلب قائمة الواجبات والأنشطة مع فلترة متقدمة |

**معلمات الفلترة الخاصة بـ Homeworks:**

| المعامل | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `companys_id` | string | - | تصفية حسب معرف الشركة |
| `departments_id` | string | - | تصفية حسب معرف الفرع |
| `year_id` | string | - | معرف السنة الدراسية. إذا لم يمرر، تستخدم السنة الافتراضية. |
| `semster` | string | `*` | الفصل الدراسي (`semster1` أو `semster2` أو `*` للكل) |
| `class_id` | string | - | معرف الصف |
| `group_id` | string | - | معرف الشعبة. يدعم `*` للكل أو `null` للجميع |
| `student_id` | string | - | معرف الطالب. يدعم `*` للكل أو `null` للجميع |
| `record_id` | string | - | معرف سجل الطالب (StudentRecord). يدعم `*` للكل |
| `subject_id` | string | - | معرف المادة |
| `teacher_id` | string | - | معرف المعلم |
| `educators_id` | string | - | معرف مربي الصف |
| `categories_id` | string | - | معرف التصنيف |
| `ref_type_class` | string | - | نوع الواجب (مثل `homework`, `activity`) |
| `month_num` | string | `*` | رقم الشهر (1،2،3) |
| `min_degree` | number | - | أقل درجة (يدعم مقارنات مثل `>=5`) |
| `max_degree` | number | - | أعلى درجة (يدعم مقارنات) |
| `target_rate` | number | - | نسبة الإنجاز (يدعم مقارنات) |
| `is_active` | bool | `true` | فلتر النشاط |
| `status` | string | `*` | الحالة (`active`, `inactive`, `proposed`) |
| `date_at` | string | - | تاريخ محدد بصيغة `Y-m-d` |
| `q` | string | - | بحث نصي في `name`, `short_description`, `description`, `solation` |

**ملاحظات هامة:**
- حقول `student_id`، `record_id`، `group_id` تدعم القيم الخاصة:
  - إذا أرسلت معرفاً محدداً، فسيتم جلب السجلات التي تطابق هذا المعرف **أو** قيمتها `*` **أو** `null` (أي الواجبات المخصصة لكل الطلاب أو لكل المجموعات).
- `year_id`: إذا لم يتم تمريره، يتم استخدام السنة الدراسية الافتراضية تلقائياً.

**العلاقات المتاحة:** `company`, `department`, `category`, `type_class`, `student`, `record`, `subject`, `teacher`

**مثال الطلب:**

```bash
GET /api/v1/homework/homeworks?year_id=5&class_id=3&student_id=15&include=student,subject
```

**مثال الاستجابة (عنصر واحد من القائمة):**

```json
{
  "id": 42,
  "code": "HW-042",
  "name": "واجب الرياضيات - الأسبوع الثاني",
  "short_description": "حل المعادلات من صفحة 10 إلى 15",
  "description": "<p>قم بحل جميع المعادلات...</p>",
  "solation": "<p>الحل النموذجي...</p>",
  "categories_id": "5",
  "ref_type_class": "homework",
  "year_id": "5",
  "semster": "semster2",
  "class_id": "3",
  "group_id": "2",
  "student_id": "15",
  "student_name": "أحمد محمد",
  "record_id": "8",
  "subject_id": "21",
  "teacher_id": "7",
  "educators_id": "12",
  "month_num": "2",
  "min_degree": 5.00,
  "max_degree": 10.00,
  "target_rate": 85.50,
  "is_default": false,
  "is_effective": true,
  "is_hidden": false,
  "is_active": true,
  "status": "active",
  "is_published": true,
  "published_at": "2026-05-01 08:00:00",
  "start_at": "2026-05-01 00:00:00",
  "end_at": "2026-05-07 23:59:59",
  "sort_order": 1,
  "created_at": "2026-05-01 08:00:00",
  "updated_at": "2026-05-01 08:00:00",
  "object_type": "Tss\\Homework\\Models\\Homework",
  "student": {
    "data": {
      "id": 15,
      "full_name": "أحمد محمد"
    }
  }
}
```

#### عرض واجب محدد

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/homework/homeworks/{id}` | عرض واجب محدد مع العلاقات |

**مثال:**  
`GET /api/v1/homework/homeworks/42?include=student,subject,type_class`

#### إنشاء واجب جديد

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `POST` | `/api/v1/homework/homeworks` | إنشاء واجب أو نشاط جديد |

**البيانات المطلوبة (JSON Body):**

| الحقل | النوع | إجباري | الوصف |
|--------|------|--------|-------|
| `name` | string | نعم | عنوان الواجب |
| `slug` | string | نعم | معرف فريد (يُولد تلقائياً من الاسم إذا لم يحدد) |
| `departments_id` | string | نعم | معرف الفرع |
| `ref_type_class` | string | نعم | نوع الواجب (مثل `homework`, `activity`) |
| `year_id` | string | نعم* | معرف السنة الدراسية (يؤخذ الافتراضي إذا لم يحدد) |
| `semster` | string | نعم | الفصل (`semster1` أو `semster2`) |
| `class_id` | string | نعم | معرف الصف |
| `subject_id` | string | نعم | معرف المادة |
| `student_id` | string | لا* | معرف الطالب (إذا كان `*` أو `null`، فسيتم توجيه الواجب لكل طلاب الصف) |
| `record_id` | string | لا* | معرف سجل الطالب |
| `group_id` | string | لا* | معرف الشعبة (إذا كان `*` فلكل المجموعات في الصف) |
| `teacher_id` | string | لا | معرف المعلم |
| `educators_id` | string | لا | معرف مربي الصف |
| `categories_id` | string | لا | معرف التصنيف |
| `month_num` | string | لا | رقم الشهر (1،2،3) |
| `min_degree` | float | لا | الحد الأدنى للدرجة |
| `max_degree` | float | لا | الحد الأقصى للدرجة |
| `target_rate` | float | لا | نسبة الإنجاز المستهدفة |
| `short_description` | string | لا | وصف قصير |
| `description` | string | لا | وصف منسق |
| `solation` | string | لا | الحل النموذجي |
| `start_at` | string | لا | تاريخ البدء (Y-m-d H:i:s) |
| `end_at` | string | لا | تاريخ الانتهاء |
| `is_active` | bool | لا | `true` (افتراضي) |
| `is_published` | bool | لا | `true` |
| `published_at` | string | لا | تاريخ النشر |

\* إذا لم يتم تمرير `companys_id` أو `departments_id` أو `year_id`، يتم تعبئتها تلقائياً من القيم الافتراضية للنظام.

**مثال لإنشاء واجب لطلاب الصف بأكمله (student_id = '*'):**

```bash
POST /api/v1/homework/homeworks
Content-Type: application/json

{
  "name": "واجب الرياضيات الأسبوعي",
  "departments_id": "1",
  "ref_type_class": "homework",
  "year_id": "5",
  "semster": "semster2",
  "class_id": "3",
  "subject_id": "21",
  "student_id": "*",
  "group_id": "*",
  "min_degree": 5,
  "max_degree": 10,
  "description": "حل تمارين الصفحات 10-15"
}
```

**مثال الاستجابة (نجاح):**

```json
{
  "data": {
    "code": 200,
    "status": true,
    "message": "Homework created successfully.",
    "data": {
      "id": 43,
      "name": "واجب الرياضيات الأسبوعي",
      "student_id": "*",
      ...
    }
  }
}
```

#### تحديث واجب

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `PUT` | `/api/v1/homework/homeworks/{id}` | تحديث واجب موجود |

يتم إرسال الحقول المراد تحديثها فقط.

**مثال لتغيير درجة الحد الأعلى:**

```bash
PUT /api/v1/homework/homeworks/42
Content-Type: application/json

{
  "max_degree": 12
}
```

#### حذف واجب

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `DELETE` | `/api/v1/homework/homeworks/{id}` | حذف واجب (يتطلب صلاحية الحذف) |

**مثال:**  
`DELETE /api/v1/homework/homeworks/42`

---

### 2. التسليمات والتقييمات (Feedbacks)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/homework/feedbacks` | جلب قائمة تسليمات الطلاب |
| `GET` | `/api/v1/homework/feedbacks/{id}` | عرض تسليمة محددة |
| `POST` | `/api/v1/homework/feedbacks` | إضافة تسليمة جديدة (تقييم الطالب) |
| `PUT` | `/api/v1/homework/feedbacks/{id}` | تحديث تسليمة |
| `GET` | `/api/v1/homework/feedbacks/activelystats` | إحصائيات نشاط التسليمات |

**معلمات الفلترة الخاصة بـ Feedbacks:**

| المعامل | الوصف |
|--------|--------|
| `homeworks_id` | معرف الواجب |
| `student_id` | معرف الطالب (يدعم `*` للكل) |
| `record_id` | معرف السجل الدراسي (يدعم `*` للكل) |
| `evaluation` | التقييم (مثل `excellent`, `good`, `average`, `poor`) |
| `status_homework` | حالة التسليم (`success`، `failed`) |
| `is_recive` | هل تم الاستلام (true/false) |
| `is_degree` | هل له درجة (true/false) |
| `degree` | الدرجة الحاصل عليها (يدعم مقارنات مثل `>=5`) |
| `target_rate` | نسبة الإنجاز |
| `date_at` | تاريخ التسليم |
| `q` | بحث في `name`, `description`, `note` |

**العلاقات المتاحة:** `homework`, `student`, `record`, `subject`, `teacher`

**مثال لجلب تسليمات واجب معين:**

```bash
GET /api/v1/homework/feedbacks?homeworks_id=42&include=student
```

**مثال الاستجابة (قائمة):**

```json
{
  "data": [
    {
      "id": 101,
      "homeworks_id": "42",
      "name": "تسليم واجب الرياضيات",
      "student_id": "15",
      "student_name": "أحمد محمد",
      "record_id": "8",
      "degree": 9.50,
      "evaluation": "excellent",
      "status_homework": "success",
      "is_recive": true,
      "note": "أحسنت",
      "teacher_note": "",
      "created_at": "2026-05-05 14:30:00"
    }
  ]
}
```

---

### 3. تصنيفات الواجبات (Categories)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/homework/categories` | جلب قائمة التصنيفات (دعم الشجرة) |
| `GET` | `/api/v1/homework/categories/{id}` | جلب تصنيف محدد |
| `GET` | `/api/v1/homework/categories/activelystats` | إحصائيات نشاط التصنيفات |

**معلمات الفلترة:**

| المعامل | الوصف |
|----------|-------|
| `companys_id` | تصفية حسب الشركة |
| `departments_id` | تصفية حسب الفرع |
| `ref_type_class` | نوع التصنيف |
| `parent_id` | معرف التصنيف الأب |
| `is_active` | فلتر النشاط |
| `q` | بحث في `name`, `slug`, `description` |

**العلاقات المتاحة:** `company`, `department`, `type_class`, `parent`, `children`

**مثال لجلب التصنيفات الرئيسية فقط:**  
`GET /api/v1/homework/categories?parent_id=null`

---

### 4. أنواع التصنيفات العامة (ClassTypes)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/homework/class-types` | جلب قائمة أنواع التصنيفات العامة |
| `GET` | `/api/v1/homework/class-types/{id}` | جلب نوع محدد |
| `GET` | `/api/v1/homework/class-types/activelystats` | إحصائيات النشاط |

**معلمات الفلترة:**

| المعامل | الوصف |
|----------|-------|
| `ref_type` | الكود المرجعي (مثل `homework`, `activity`, `image`) |
| `is_active` | فلتر النشاط |
| `q` | بحث في `name`, `ref_type`, `description` |

**العلاقات المتاحة:** `company`

---

### 5. أنواع التصنيفات الخاصة بالشركات (ClassTypeCompanies)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/homework/class-type-companies` | جلب قائمة تصنيفات الشركات |
| `GET` | `/api/v1/homework/class-type-companies/{id}` | جلب تصنيف شركة محدد |
| `GET` | `/api/v1/homework/class-type-companies/activelystats` | إحصائيات النشاط |

**معلمات الفلترة:**

| المعامل | الوصف |
|----------|-------|
| `companys_id` | تصفية حسب الشركة |
| `departments_id` | تصفية حسب الفرع |
| `ref_type` | الكود المرجعي |
| `is_active` | فلتر النشاط |
| `q` | بحث في `name`, `ref_type`, `description` |

**العلاقات المتاحة:** `company`, `department`, `class_type`

---

### 6. إحصائيات النشاط (Actively Stats)

كل مورد يمتلك نقطة نهاية `activelystats` تعيد معلومات عن آخر تحديث للبيانات، مفيدة لإبطال الكاش في التطبيقات العميلة.

**طلب مثال:**  
`GET /api/v1/homework/homeworks/activelystats`

**استجابة مثال:**

```json
{
  "data": {
    "code": 200,
    "data": {
      "last_updated": "2026-05-05 09:15:00",
      "activity_stats": false
    }
  }
}
```

---

## معلمات الطلب العامة

تنطبق هذه المعلمات على معظم نقاط النهاية `list`:

| المعامل | النوع | الافتراضي | الوصف |
|--------|------|----------|-------|
| `isActive` | mixed | `1` | تصفية النشط: `1` نشط، `0` غير نشط، `*` الكل |
| `q` | string | - | بحث نصي في الحقول الأساسية. |
| `orderBy` | string | يختلف | اسم الحقل للترتيب حسبه. |
| `orderDirection` | string | `asc` | اتجاه الترتيب: `asc` أو `desc`. |
| `page` | int | 1 | رقم الصفحة. |
| `per_page` | int | 15 | عدد العناصر في الصفحة. |
| `exclude` | string | - | أسماء حقول مستبعدة من الاستجابة، مفصولة بفواصل. |
| `include` | string | - | أسماء العلاقات المضمنة، مفصولة بفواصل. |

**ملاحظة حول `isActive`:** يمكن استخدام القيم `"1"`, `"0"`, `"*"`, أو `"all"`.

---

## تنسيق التواريخ

جميع التواريخ في الاستجابات بصيغة `Y-m-d H:i:s` (مثال: `"2026-05-05 12:30:45"`). الحقول الفارغة تعاد بقيمة `null`.

---

## العلاقات المضمنة

لتضمين كائنات مرتبطة في نفس الطلب، استخدم المعامل `include` بقيمة تحتوي على أسماء العلاقات مفصولة بفواصل.

**مثال لتضمين الطالب والمادة في واجب:**

```
GET /api/v1/homework/homeworks/42?include=student,subject
```

إذا كانت العلاقة غير موجودة أو حدث خطأ، تعاد كمصفوفة فارغة `[]` بدلاً من فشل الطلب.

---

## رموز الأخطاء والتعامل معها

| الكود | المعنى |
|------|--------|
| 200 | نجاح - تم جلب البيانات أو تنفيذ العملية. |
| 400 | خطأ في الطلب - المعطيات غير صالحة، العملية معطلة إعدادياً، أو الصلاحيات غير كافية. |
| 401 | غير مصرح - التوكن مفقود أو غير صالح. |
| 404 | غير موجود - المورد المطلوب غير موجود. |
| 500 | خطأ خادم داخلي. |

**جسم الخطأ النموذجي:**

```json
{
  "error": {
    "code": 400,
    "message": "You do not have permission to list homeworks.",
    "errors": []
  }
}
```

- `message`: رسالة الخطأ باللغة المحددة (الافتراضي العربية في هذا التوثيق، لكن الـ API يعيدها حسب لغة المستخدم).
- `errors`: تفاصيل إضافية عن الحقول الفاشلة (خاصة في عمليات الإنشاء والتحديث عند فشل التحقق من صحة البيانات).

---

## ملاحظات للمطورين

- **الواجبات الشاملة للصف/الشعبة:** عند إنشاء واجب، يمكنك تعيين `student_id = "*"` أو `group_id = "*"` لجعل الواجب موزعاً على كل الطلاب أو كل المجموعات. عند جلب البيانات، سيتم تضمين هذه السجلات في الاستعلامات العادية لأن منطق الفلترة يتعامل مع `*` و `null` كقيمة صالحة.
- **الدرجات والمقارنات:** يمكنك استخدام مقارنات مثل `>=5` أو `['>', 10]` مع حقول `min_degree`, `max_degree`, `target_rate`, `degree` لتطبيق شروط متقدمة.
- **القيم الافتراضية:** في عمليات الإنشاء (`POST`)، إذا لم ترسل `year_id` أو `companys_id` أو `departments_id`، سيستخدم API الإعدادات الافتراضية للنظام مما يسهل عملية التسجيل في السياق الحالي.
- **الصلاحيات:** يتم التحكم في إمكانية الوصول لكل عملية (عرض، إنشاء، تحديث، حذف) لكل نوع مستخدم (backend/frontend) عبر متغيرات البيئة. راجع مسؤول النظام للتخصيص.
- **التخزين المؤقت:** استخدم `activelystats` لتحديد ما إذا كانت بياناتك محدثة والحاجة لإعادة الجلب.

هذا التوثيق يغطي جميع نقاط API المتاحة في الإصدار 1.0.0 من `Nano.HomeworkApi`. لأية أسئلة أو توسعات، راجع دليل المطور الرئيسي للمشروع.
