## توثيق واجهة برمجة تطبيقات Nano.AbsenceApi

**الإصدار:** 1.0.0  
**الفئة المستهدفة:** مطورو تطبيقات الواجهة الأمامية، مطورو التكاملات الخارجية.

---

### المحتويات

1. [مقدمة](#مقدمة)
2. [المصادقة (Authentication)](#المصادقة)
3. [هيكل الاستجابة العام](#هيكل-الاستجابة-العام)
4. [قائمة نقاط النهاية (Endpoints)](#قائمة-نقاط-النهاية)
   - [سجلات الحضور والغياب (Absences)](#1-سجلات-الحضور-والغياب-absences)
     - [جلب قائمة سجلات الغياب](#جلب-قائمة-سجلات-الغياب)
     - [عرض سجل غياب محدد](#عرض-سجل-غياب-محدد)
     - [إنشاء سجل غياب جديد](#إنشاء-سجل-غياب-جديد)
     - [تحديث سجل غياب](#تحديث-سجل-غياب)
   - [كشوفات التحضير (Files)](#2-كشوفات-التحضير-files)
   - [التصنيفات الرئيسية (ClassTypes)](#3-التصنيفات-الرئيسية-classtypes)
   - [تصنيفات الشركات (ClassTypeCompanies)](#4-تصنيفات-الشركات-classtypecompanies)
   - [إحصائيات النشاط (Actively Stats)](#5-إحصائيات-النشاط-actively-stats)
5. [معلمات الطلب العامة](#معلمات-الطلب-العامة)
6. [تنسيق التواريخ](#تنسيق-التواريخ)
7. [العلاقات المضمنة (Includes)](#العلاقات-المضمنة)
8. [رموز الأخطاء والتعامل معها](#رموز-الأخطاء-والتعامل-معها)

---

## مقدمة

توفر واجهة برمجة تطبيقات `Nano.AbsenceApi` نقاط نهاية RESTful للوصول إلى بيانات الحضور والغياب وكشوفات التحضير وتصنيفاتها في نظام `Tss.Absence`. يمكن لمطوري تطبيقات الواجهة الأمامية والتطبيقات الخارجية استخدام API لجلب سجلات الغياب، إنشاء سجلات جديدة، تحديثها، بالإضافة إلى استعلام الكشوفات والتصنيفات.

**Base URL:**  
`https://yourdomain.com/api/v1/absences`

---

## المصادقة

جميع نقاط النهاية محمية وتتطلب **رمز حامل (Bearer token)** صالحًا. يتم تمرير الرمز في ترويسة `Authorization` بصيغة:

```
Authorization: Bearer <your_token>
```

يتم التحقق من المستخدم ونوعه (backend / frontend) لتطبيق صلاحيات الوصول المحددة لكل نقطة نهاية وعملية. تختلف الصلاحيات المطلوبة حسب العملية (قراءة، إنشاء، تحديث) ويتم ضبطها إعدادياً.

---

## هيكل الاستجابة العام

### استجابة ناجحة (قائمة)

```json
{
  "data": [
    {
      "id": 42,
      "student_name": "أحمد محمد",
      "absences_status": "absent",
      ...
      "object_type": "Tss\\Absence\\Models\\Absence"
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

### استجابة ناجحة (عنصر واحد / عملية إنشاء أو تحديث)

```json
{
  "data": {
    "id": 42,
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
    "message": "Absence record created successfully.",
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
    "message": "Listing absences is disabled for frontend users.",
    "errors": []
  }
}
```

---

## قائمة نقاط النهاية

### 1. سجلات الحضور والغياب (Absences)

#### جلب قائمة سجلات الغياب

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/absence/absences` | جلب قائمة سجلات الحضور والغياب مع فلترة متقدمة |

**معلمات الفلترة الخاصة بـ Absences:**

| المعامل | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `companys_id` | string | - | تصفية حسب معرف الشركة |
| `departments_id` | string | - | تصفية حسب معرف الفرع |
| `year_id` | string | - | معرف السنة الدراسية. إذا لم يمرر، تستخدم السنة الافتراضية. |
| `semster` | string | - | الفصل الدراسي (`semster1` أو `semster2`) |
| `semsters_id` | string | - | معرف الفصل الدراسي (نموذج Semster من Studdyear) |
| `months_id` | string | - | معرف الشهر الدراسي (نموذج Month) |
| `month_num` | string | - | رقم الشهر داخل الفصل (1،2،3) |
| `class_id` | string | - | معرف الصف |
| `group_id` | string | - | معرف الشعبة |
| `student_id` | string | - | معرف الطالب |
| `record_id` | string | - | معرف سجل الطالب (StudentRecord) |
| `absences_status` | string | `*` | حالة الحضور: `present`، `absent`، `permitted`، `*` للكل |
| `files_id` | string | - | معرف ملف الكشف |
| `ref_type_class` | string | - | نوع الكشف (مثل `day`, `subject`, `exam`) |
| `date_at` | string | - | تاريخ محدد (بصيغة `Y-m-d`) |
| `q` | string | - | بحث نصي في `student_name`, `absences_bacuse`, `notes` |

**العلاقات المتاحة:** `company`, `department`, `file`, `student`, `record`

**مثال الطلب:**

```bash
GET /api/v1/absence/absences?date_at=2026-05-01&class_id=3&include=student,file
```

**مثال الاستجابة (عنصر واحد من القائمة):**

```json
{
  "id": 42,
  "code": "ABS-042",
  "student_id": "15",
  "student_name": "أحمد محمد",
  "record_id": "8",
  "class_id": "3",
  "group_id": "2",
  "year_id": "5",
  "semster": "semster2",
  "month_num": "2",
  "absences_status": "absent",
  "absences_bacuse": "مرض",
  "notes": "غائب بعذر",
  "date_at": "2026-05-01 00:00:00",
  "is_finality": false,
  "is_active": true,
  "files_id": "5",
  "ref_type_class": "day",
  "created_at": "2026-05-01 08:00:00",
  "updated_at": "2026-05-01 08:00:00",
  "object_type": "Tss\\Absence\\Models\\Absence",
  "student": {
    "data": {
      "id": 15,
      "full_name": "أحمد محمد"
    }
  }
}
```

#### عرض سجل غياب محدد

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/absence/absences/{id}` | عرض سجل غياب محدد مع العلاقات |

**مثال:**  
`GET /api/v1/absence/absences/42?include=student,file`

#### إنشاء سجل غياب جديد

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `POST` | `/api/v1/absence/absences` | إنشاء سجل غياب جديد |

**البيانات المطلوبة (JSON Body):**

| الحقل | النوع | إجباري | الوصف |
|--------|------|--------|-------|
| `student_id` | string | نعم | معرف الطالب |
| `record_id` | string | نعم | معرف سجل الطالب (StudentRecord) |
| `year_id` | string | لا* | معرف السنة الدراسية (يؤخذ الافتراضي إذا لم يحدد) |
| `class_id` | string | نعم | معرف الصف |
| `group_id` | string | لا | معرف الشعبة |
| `semster` | string | نعم | الفصل (`semster1` أو `semster2`) |
| `month_num` | string | لا | رقم الشهر (1-3) |
| `files_id` | string | نعم | معرف ملف الكشف |
| `ref_type_class` | string | نعم | نوع الكشف (`day`, `subject`, `exam`, `activete`) |
| `absences_status` | string | لا | حالة الحضور: `present`, `absent`, `permitted` (الافتراضي `present`) |
| `absences_bacuse` | string | لا | سبب الغياب |
| `notes` | string | لا | ملاحظات |
| `date_at` | string | نعم | تاريخ اليوم بصيغة `Y-m-d` |

* إذا لم يتم تمرير `companys_id` أو `departments_id` أو `year_id`، يتم تعبئتها تلقائياً من القيم الافتراضية للنظام.

**مثال للطلب:**

```bash
POST /api/v1/absence/absences
Content-Type: application/json

{
  "student_id": "15",
  "record_id": "8",
  "year_id": "5",
  "class_id": "3",
  "semster": "semster2",
  "month_num": "2",
  "files_id": "5",
  "ref_type_class": "day",
  "absences_status": "absent",
  "absences_bacuse": "مرض",
  "date_at": "2026-05-05"
}
```

**مثال الاستجابة (نجاح):**

```json
{
  "data": {
    "code": 200,
    "status": true,
    "message": "Absence record created successfully.",
    "error": null,
    "errors": null,
    "data": {
      "id": 43,
      "student_name": "أحمد محمد",
      "absences_status": "absent",
      ...
    },
    "model": { ... },
    "input_data": { ... },
    "process_data": { ... }
  }
}
```

#### تحديث سجل غياب

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `PUT` | `/api/v1/absence/absences/{id}` | تحديث سجل غياب موجود |

يتم إرسال الحقول المراد تحديثها فقط. البيانات غير المذكورة لن تتغير. تنطبق نفس الصلاحيات وقواعد التحقق (مثلاً لا يمكن تكرار سجل لنفس الطالب في نفس اليوم والكشف).

**مثال لتغيير حالة الغياب إلى "حاضر":**

```bash
PUT /api/v1/absence/absences/42
Content-Type: application/json

{
  "absences_status": "present"
}
```

**هيكل الاستجابة مطابق لهيكل الإنشاء.**

---

### 2. كشوفات التحضير (Files)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/absence/files` | جلب قائمة كشوفات التحضير |
| `GET` | `/api/v1/absence/files/{id}` | جلب كشف محدد |
| `GET` | `/api/v1/absence/files/activelystats` | إحصائيات نشاط الكشوفات |

**معلمات الفلترة:**

| المعامل | الوصف |
|----------|-------|
| `companys_id` | تصفية حسب الشركة |
| `departments_id` | تصفية حسب الفرع |
| `ref_type_class` | تصفية حسب نوع الكشف (`day`, `subject`, إلخ) |
| `isActive` | `1` للنشط، `0` للغير نشط |
| `q` | بحث نصي في `name`, `slug`, `short_description` |

**العلاقات المتاحة:** `company`, `department`.

---

### 3. التصنيفات الرئيسية (ClassTypes)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/absence/class-types` | جلب قائمة التصنيفات الرئيسية لأنواع الكشوفات |
| `GET` | `/api/v1/absence/class-types/{id}` | جلب تصنيف محدد |
| `GET` | `/api/v1/absence/class-types/activelystats` | إحصائيات نشاط التصنيفات |

**معلمات الفلترة:**

| المعامل | الوصف |
|----------|-------|
| `ref_type` | تصفية حسب الكود المرجعي (مثل `day`, `subject`) |
| `isActive` | فلتر النشاط |
| `q` | بحث في `name`, `ref_type`, `description` |

**العلاقات المتاحة:** `company`

---

### 4. تصنيفات الشركات (ClassTypeCompanies)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/absence/class-type-companies` | جلب قائمة تصنيفات الشركات |
| `GET` | `/api/v1/absence/class-type-companies/{id}` | جلب تصنيف شركة محدد |
| `GET` | `/api/v1/absence/class-type-companies/activelystats` | إحصائيات نشاط التصنيفات |

**معلمات الفلترة:**

| المعامل | الوصف |
|----------|-------|
| `companys_id` | تصفية حسب الشركة |
| `departments_id` | تصفية حسب الفرع |
| `ref_type` | تصفية حسب الكود المرجعي |
| `isActive` | فلتر النشاط |
| `q` | بحث في `name`, `ref_type`, `description` |

**العلاقات المتاحة:** `company`, `department`, `class_type` (تصنيف رئيسي).

---

### 5. إحصائيات النشاط (Actively Stats)

كل مورد يمتلك نقطة نهاية `activelystats` تعيد معلومات عن آخر تحديث للبيانات، مفيدة لإبطال الكاش في التطبيقات العميلة.

**طلب مثال:**  
`GET /api/v1/absence/absences/activelystats`

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

**مثال لتضمين الطالب والكشف في سجل غياب:**

```
GET /api/v1/absence/absences/42?include=student,file
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
    "message": "Creating absences is disabled for frontend users.",
    "errors": []
  }
}
```

- `message`: رسالة الخطأ باللغة المحددة (الافتراضي الإنجليزية).
- `errors`: تفاصيل إضافية عن الحقول الفاشلة (إن وجدت)، خاصة في عمليات الإنشاء والتحديث عند فشل التحقق.

---

## ملاحظات للمطورين

- **إعدادات الصلاحيات:** يتم التحكم في إمكانية الوصول لكل عملية (عرض، إنشاء، تحديث) لكل نوع مستخدم (backend/frontend) عبر متغيرات البيئة. راجع مسؤول النظام للتخصيص.
- **القيم الافتراضية:** في عمليات الإنشاء (`POST`)، إذا لم ترسل `year_id` أو `companys_id` أو `departments_id`، سيستخدم API الإعدادات الافتراضية للنظام مما يسهل عملية التسجيل في السياق الحالي.
- **التحقق من التكرار:** لا يمكن إنشاء سجلي غياب لنفس الطالب في نفس اليوم ونفس الكشف (`files_id`). ستتلقى خطأ إذا حاولت ذلك.
- **حالة `absences_status`:** القيم المقبولة هي `present`, `absent`, `permitted`. تأكد من مطابقتها تماماً.
- **تنسيق التاريخ:** عند إرسال `date_at` في طلبات `POST`، استخدم الصيغة `Y-m-d` (بدون وقت).
- **التخزين المؤقت:** استخدم `activelystats` لتحديد ما إذا كانت بياناتك محدثة والحاجة لإعادة الجلب.

هذا التوثيق يغطي جميع نقاط API المتاحة في الإصدار 1.0.0 من `Nano.AbsenceApi`. لأية أسئلة أو توسعات، راجع دليل المطور الرئيسي للمشروع.