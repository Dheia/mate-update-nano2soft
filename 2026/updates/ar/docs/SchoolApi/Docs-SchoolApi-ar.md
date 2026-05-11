## توثيق واجهة برمجة تطبيقات Nano.SchoolApi

**الإصدار:** 1.0.0  
**الفئة المستهدفة:** مطورو تطبيقات الواجهة الأمامية، مطورو التكاملات الخارجية.

---

### المحتويات

1. [مقدمة](#مقدمة)
2. [المصادقة (Authentication)](#المصادقة)
3. [هيكل الاستجابة العام](#هيكل-الاستجابة-العام)
4. [قائمة نقاط النهاية (Endpoints)](#قائمة-نقاط-النهاية)
   - [الصفوف الدراسية](#1-الصفوف-الدراسية-classes)
   - [المراحل الدراسية](#2-المراحل-الدراسية-stages)
   - [المواد الدراسية](#3-المواد-الدراسية-subjects)
   - [الشعب الدراسية](#4-الشعب-الدراسية-groups)
   - [توزيع الشعب على الصفوف](#5-توزيع-الشعب-على-الصفوف-classgroups)
   - [توزيع المواد على الصفوف](#6-توزيع-المواد-على-الصفوف-classsubjects)
   - [توزيع المعلمين على المواد](#7-توزيع-المعلمين-على-المواد-teachersubjects)
   - [مراكز التكلفة المرتبطة بالصفوف](#8-مراكز-التكلفة-المرتبطة-بالصفوف-classcostcenters)
   - [أنواع الرسوم الدراسية](#9-أنواع-الرسوم-الدراسية-feestypes)
   - [أنواع الوثائق](#10-أنواع-الوثائق-documenttypes)
   - [المدارس الخارجية](#11-المدارس-الخارجية-outschools)
   - [وثائق الطلاب](#12-وثائق-الطلاب-studentdocuments)
   - [رسوم الطلاب](#13-رسوم-الطلاب-studentsfees)
   - [علامات آخر سنة دراسية](#14-علامات-آخر-سنة-دراسية-laststudentmarks)
   - [إحصائيات النشاط (Actively Stats)](#15-إحصائيات-النشاط-actively-stats)
5. [معلمات الطلب العامة](#معلمات-الطلب-العامة)
6. [تنسيق التواريخ](#تنسيق-التواريخ)
7. [العلاقات المضمنة (Includes)](#العلاقات-المضمنة)
8. [رموز الأخطاء والتعامل معها](#رموز-الأخطاء-والتعامل-معها)

---

## مقدمة

توفر واجهة برمجة تطبيقات `Nano.SchoolApi` نقاط نهاية RESTful للوصول إلى جميع الكيانات الأساسية لنظام إدارة المدرسة (Tss.School). يمكن لمطوري تطبيقات الواجهة الأمامية والتطبيقات الخارجية استخدام API لاسترجاع الصفوف والمراحل والمواد والشعب وتوزيعاتها، بالإضافة إلى الرسوم والوثائق وعلامات الطلاب.

تم بناء API وفقاً لنمط `Nano.*Api` القياسي، مما يضمن الاتساق مع باقي خدمات API في النظام.

**Base URL:**  
`https://yourdomain.com/api/v1/school`

---

## المصادقة

جميع نقاط النهاية محمية وتتطلب **رمز حامل (Bearer token)** صالحًا. يتم تمرير الرمز في ترويسة `Authorization` بصيغة:

```
Authorization: Bearer <your_token>
```

يتم التحقق من المستخدم ونوعه (backend / frontend) لتطبيق صلاحيات الوصول المحددة لكل نقطة نهاية. يجب أن يمتلك المستخدم الصلاحيات المناسبة في النظام الخلفي إذا تم تفعيل خيار `is_check_list_permission`.

---

## هيكل الاستجابة العام

### استجابة ناجحة (قائمة)

```json
{
  "data": [
    {
      "id": 1,
      "name": "الصف الأول",
      ...
      "object_type": "Tss\\School\\Models\\TbClass"
    }
  ],
  "meta": {
    "pagination": {
      "total": 150,
      "count": 15,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 10,
      "links": {
        "next": "https://...?page=2",
        "previous": null
      }
    }
  }
}
```

### استجابة ناجحة (عنصر واحد)

```json
{
  "data": {
    "id": 1,
    "name": "الصف الأول",
    ...
  }
}
```

### استجابة خطأ

```json
{
  "error": {
    "code": 400,
    "message": "Listing classes is disabled for frontend users.",
    "errors": []
  }
}
```

---

## قائمة نقاط النهاية

### 1. الصفوف الدراسية (Classes)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/classes` | جلب قائمة الصفوف |
| `GET` | `/api/v1/school/classes/{id}` | جلب صف محدد |
| `GET` | `/api/v1/school/classes/activelystats` | إحصائيات نشاط الصفوف |

**معلمات الفلترة:**

| المعامل | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `companys_id` | string | - | تصفية حسب معرف الشركة |
| `departments_id` | string | - | تصفية حسب معرف الفرع |
| `stage_id` | string | - | تصفية حسب معرف المرحلة |
| `isActive` | bool\|string | `1` | `1` للنشط فقط، `0` للغير نشط، `*` للكل |
| `q` | string | - | بحث نصي في الاسم والكود |
| `orderBy` | string | `sort_order` | ترتيب حسب حقل |
| `orderDirection` | string | `asc` | اتجاه الترتيب (`asc` أو `desc`) |
| `page` | int | 1 | رقم الصفحة |
| `per_page` | int | 15 | عدد العناصر في الصفحة |
| `exclude` | string | - | أسماء الحقول المستبعدة مفصولة بفواصل |
| `include` | string | - | العلاقات المضمنة (company, department, stage) |

**مثال للطلب:**

```bash
GET /api/v1/school/classes?stage_id=2&isActive=1&include=stage,department
```

**مثال للاستجابة:**

```json
{
  "data": {
    "id": 5,
    "code": "CLS-005",
    "name": "الصف الثاني الثانوي",
    "companys_id": "1",
    "departments_id": "3",
    "stage_id": "2",
    "study_fees": 2500.0,
    "registration_fees": 500.0,
    "class_rank": "2",
    "class_stage_rank": "2",
    "class_type": "primary",
    "is_active": true,
    "status": "active",
    "sort_order": 2,
    "created_at": "2025-08-15 10:30:00",
    "updated_at": "2026-01-10 14:22:00",
    "object_type": "Tss\\School\\Models\\TbClass",
    "stage": {
      "data": {
        "id": 2,
        "name": "المرحلة الثانوية"
      }
    }
  }
}
```

---

### 2. المراحل الدراسية (Stages)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/stages` | جلب قائمة المراحل |
| `GET` | `/api/v1/school/stages/{id}` | جلب مرحلة محددة |
| `GET` | `/api/v1/school/stages/activelystats` | إحصائيات نشاط المراحل |

**معلمات الفلترة:** مشابهة للصفوف + `departments_id`.

**العلاقات المتاحة:** `company`, `department`.

---

### 3. المواد الدراسية (Subjects)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/subjects` | جلب قائمة المواد |
| `GET` | `/api/v1/school/subjects/{id}` | جلب مادة محددة |
| `GET` | `/api/v1/school/subjects/activelystats` | إحصائيات نشاط المواد |

**معلمات إضافية:** `departments_id`.

**العلاقات المتاحة:** `company`, `department`.

---

### 4. الشعب الدراسية (Groups)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/groups` | جلب قائمة الشعب |
| `GET` | `/api/v1/school/groups/{id}` | جلب شعبة محددة |
| `GET` | `/api/v1/school/groups/activelystats` | إحصائيات نشاط الشعب |

**العلاقات المتاحة:** `company`, `department`.

---

### 5. توزيع الشعب على الصفوف (ClassGroups)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/class-groups` | جلب قائمة توزيعات الشعب |
| `GET` | `/api/v1/school/class-groups/{id}` | جلب توزيعة محددة |
| `GET` | `/api/v1/school/class-groups/activelystats` | إحصائيات نشاط التوزيعات |

**معلمات إضافية:**

| المعامل | الوصف |
|----------|-------|
| `class_id` | تصفية حسب معرف الصف |
| `group_id` | تصفية حسب معرف الشعبة |
| `year_id` | **مطلوب فعلياً** - معرف السنة الدراسية. إذا لم يمرر، يتم استخدام السنة الافتراضية للنظام. |

**العلاقات المتاحة:** `company`, `department`, `class`, `group`, `study_year`.

---

### 6. توزيع المواد على الصفوف (ClassSubjects)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/class-subjects` | جلب قائمة توزيعات المواد |
| `GET` | `/api/v1/school/class-subjects/{id}` | جلب توزيعة محددة |
| `GET` | `/api/v1/school/class-subjects/activelystats` | إحصائيات نشاط التوزيعات |

**معلمات إضافية:**

| المعامل | الوصف |
|----------|-------|
| `class_id` | تصفية حسب الصف |
| `subject_id` | تصفية حسب المادة |
| `semster` | تصفية حسب الفصل (`semster1`, `semster2`, `all`) |
| `subject_type` | نوع المادة (`primary`, `requirement`) |
| `subject_kind` | طبيعة المادة (`academic`, `practical`, `activity`) |

**العلاقات المتاحة:** `company`, `department`, `class`, `subject`.

---

### 7. توزيع المعلمين على المواد (TeacherSubjects)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/teacher-subjects` | جلب قائمة توزيعات المعلمين |
| `GET` | `/api/v1/school/teacher-subjects/{id}` | جلب توزيعة محددة |
| `GET` | `/api/v1/school/teacher-subjects/activelystats` | إحصائيات نشاط التوزيعات |

**معلمات إضافية:**

| المعامل | الوصف |
|----------|-------|
| `class_id` | تصفية حسب الصف |
| `group_id` | تصفية حسب الشعبة |
| `subject_id` | تصفية حسب المادة |
| `teacher_id` | تصفية حسب المعلم |
| `year_id` | معرف السنة الدراسية (افتراضي: السنة الأساسية) |

**العلاقات المتاحة:** `company`, `department`, `class`, `group`, `subject`, `teacher`, `study_year`.

---

### 8. مراكز التكلفة المرتبطة بالصفوف (ClassCostCenters)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/class-cost-centers` | جلب قائمة ربط مراكز التكلفة |
| `GET` | `/api/v1/school/class-cost-centers/{id}` | جلب ربط محدد |
| `GET` | `/api/v1/school/class-cost-centers/activelystats` | إحصائيات نشاط الربط |

**معلمات إضافية:** `class_id`, `cost_centers_id`, `year_id`.

**العلاقات المتاحة:** `company`, `department`, `class`, `cost_center`, `study_year`.

---

### 9. أنواع الرسوم الدراسية (FeesTypes)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/fees-types` | جلب قائمة أنواع الرسوم |
| `GET` | `/api/v1/school/fees-types/{id}` | جلب نوع محدد |
| `GET` | `/api/v1/school/fees-types/activelystats` | إحصائيات نشاط أنواع الرسوم |

**العلاقات المتاحة:** `company`, `department`.

---

### 10. أنواع الوثائق (DocumentTypes)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/document-types` | جلب قائمة أنواع الوثائق |
| `GET` | `/api/v1/school/document-types/{id}` | جلب نوع محدد |
| `GET` | `/api/v1/school/document-types/activelystats` | إحصائيات نشاط أنواع الوثائق |

**العلاقات المتاحة:** `company`, `department`.

---

### 11. المدارس الخارجية (OutSchools)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/out-schools` | جلب قائمة المدارس الخارجية |
| `GET` | `/api/v1/school/out-schools/{id}` | جلب مدرسة محددة |
| `GET` | `/api/v1/school/out-schools/activelystats` | إحصائيات نشاط المدارس الخارجية |

**العلاقات المتاحة:** `company`, `department`.

---

### 12. وثائق الطلاب (StudentDocuments)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/student-documents` | جلب قائمة وثائق الطلاب |
| `GET` | `/api/v1/school/student-documents/{id}` | جلب وثيقة محددة |
| `GET` | `/api/v1/school/student-documents/activelystats` | إحصائيات نشاط وثائق الطلاب |

**معلمات إضافية:**

| المعامل | الوصف |
|--------|-------|
| `student_id` | تصفية حسب الطالب |
| `document_id` | تصفية حسب نوع الوثيقة |
| `year_id` | معرف السنة الدراسية |

**العلاقات المتاحة:** `company`, `department`, `student`, `documents` (نوع الوثيقة), `study_year`.

---

### 13. رسوم الطلاب (StudentsFees)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/students-fees` | جلب قائمة رسوم الطلاب |
| `GET` | `/api/v1/school/students-fees/{id}` | جلب رسم محدد |
| `GET` | `/api/v1/school/students-fees/activelystats` | إحصائيات نشاط رسوم الطلاب |

**معلمات إضافية:**

| المعامل | الوصف |
|--------|-------|
| `student_id` | تصفية حسب الطالب |
| `record_id` | تصفية حسب سجل الطالب |
| `fees_type_id` | تصفية حسب نوع الرسم |
| `year_id` | معرف السنة الدراسية |

**العلاقات المتاحة:** `company`, `department`, `student`, `student_record`, `fees_type`, `study_year`.

---

### 14. علامات آخر سنة دراسية (LastStudentMarks)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/school/last-student-marks` | جلب قائمة علامات آخر سنة |
| `GET` | `/api/v1/school/last-student-marks/{id}` | جلب علامة محددة |
| `GET` | `/api/v1/school/last-student-marks/activelystats` | إحصائيات نشاط العلامات |

**معلمات إضافية:**

| المعامل | الوصف |
|--------|-------|
| `student_id` | تصفية حسب الطالب |
| `class_id` | تصفية حسب الصف |
| `out_school_id` | تصفية حسب المدرسة الخارجية |
| `year_id` | معرف السنة الدراسية |

**العلاقات المتاحة:** `company`, `department`, `student`, `class`, `out_school`, `student_record`, `study_year`.

---

### 15. إحصائيات النشاط (Actively Stats)

كل مورد يمتلك نقطة نهاية `/activelystats` تعيد معلومات عن آخر تحديث للبيانات، وتفيد في إبطال الكاش في التطبيقات.

**طلب مثال:**

```bash
GET /api/v1/school/classes/activelystats
```

**استجابة مثال:**

```json
{
  "data": {
    "code": 200,
    "data": {
      "last_updated": "2026-05-05 08:30:00",
      "activity_stats": false
    }
  }
}
```

- `last_updated`: طابع زمني لآخر تحديث.
- `activity_stats`: `true` إذا كان هناك نشاط أحدث من الوقت الحالي (عادة `false` في البيئات الحية).

---

## معلمات الطلب العامة

تنطبق هذه المعلمات على معظم نقاط النهاية `list`:

| المعامل | النوع | الافتراضي | الوصف |
|--------|------|----------|-------|
| `isActive` | mixed | `1` | تصفية النشط: `1` نشط، `0` غير نشط، `*` الكل |
| `q` | string | - | بحث نصي في الحقول الأساسية (الاسم، الكود). |
| `orderBy` | string | يختلف | اسم الحقل للترتيب حسبه (مثلاً `sort_order`, `name`, `created_at`). |
| `orderDirection` | string | `asc` | اتجاه الترتيب: `asc` تصاعدي، `desc` تنازلي. |
| `page` | int | 1 | رقم الصفحة للتقسيم. |
| `per_page` | int | 15 | عدد العناصر في الصفحة (الحد الأقصى يحدد إعدادياً). |
| `exclude` | string | - | أسماء حقول مستبعدة من الاستجابة، مفصولة بفواصل (مثلاً `created_at,updated_at`). |
| `include` | string | - | أسماء العلاقات المراد تضمينها، مفصولة بفواصل (مثلاً `stage,department`). |

**ملاحظة:** قيم `isActive` يمكن أن تكون `"1"`, `"0"`, `"*"`, أو `"all"` لتعطيل الفلتر.

---

## تنسيق التواريخ

جميع التواريخ في الاستجابات تكون بصيغة `Y-m-d H:i:s` (مثال: `"2026-05-05 12:30:45"`). الحقول الفارغة تعاد بقيمة `null`.

---

## العلاقات المضمنة

لتضمين كائنات مرتبطة في نفس الطلب، استخدم المعامل `include` بقيمة تحتوي على أسماء العلاقات مفصولة بفواصل.

**مثال لتضمين المرحلة والفرع للصف:**

```
GET /api/v1/school/classes/5?include=stage,department
```

إذا كانت العلاقة غير موجودة أو حدث خطأ، تعاد كمصفوفة فارغة `[]` بدلاً من فشل الطلب.

---

## رموز الأخطاء والتعامل معها

| الكود | المعنى |
|------|--------|
| 200 | نجاح - تم جلب البيانات. |
| 400 | خطأ في الطلب - المعطيات غير صالحة أو العملية معطلة. |
| 401 | غير مصرح - التوكن مفقود أو غير صالح. |
| 404 | غير موجود - المورد غير موجود. |
| 500 | خطأ خادم داخلي. |

**جسم الخطأ النموذجي:**

```json
{
  "error": {
    "code": 400,
    "message": "Listing classes is disabled for frontend users.",
    "errors": []
  }
}
```

- `message`: رسالة الخطأ باللغة المحددة (الافتراضي الإنجليزية).
- `errors`: تفاصيل إضافية عن الحقول الفاشلة (إن وجدت).

---

## ملاحظات للمطورين

- يمكن تعطيل/تفعيل كل نقطة نهاية ونوع مستخدم من خلال متغيرات البيئة في ملف `config.php`. راجع توثيق مدير النظام لتفاصيل الإعداد.
- في الموارد التي تعتمد على `year_id` (مثل `ClassGroups`)، إذا لم يتم تمريرها، يستخدم API **السنة الدراسية الافتراضية** تلقائياً، مما يسهل العمل في السياق الحالي.
- استخدم `exclude` لتقليل حجم الاستجابة: مثلاً `exclude=created_at,updated_at,deleted_at`.
- يوصى باستخدام التخزين المؤقت في التطبيقات العميلة بالاعتماد على قيمة `last_updated` من `activelystats`.

هذا التوثيق يغطي جميع نقاط API المتاحة في الإصدار 1.0.0 من `Nano.SchoolApi`. لأية أسئلة أو توسعات، راجع دليل المطور الرئيسي للمشروع.