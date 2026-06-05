## توثيق واجهة برمجة تطبيقات Nano.StudyyearApi

**الإصدار:** 1.0.0  
**الفئة المستهدفة:** مطورو تطبيقات الواجهة الأمامية، مطورو التكاملات الخارجية.

---

### المحتويات

1. [مقدمة](#مقدمة)
2. [المصادقة (Authentication)](#المصادقة)
3. [هيكل الاستجابة العام](#هيكل-الاستجابة-العام)
4. [قائمة نقاط النهاية (Endpoints)](#قائمة-نقاط-النهاية)
   - [السنوات الدراسية (Periods)](#1-السنوات-الدراسية-periods)
   - [الفصول الدراسية (Semsters)](#2-الفصول-الدراسية-semsters)
   - [الأشهر الدراسية (Months)](#3-الأشهر-الدراسية-months)
   - [إحصائيات النشاط (Actively Stats)](#4-إحصائيات-النشاط-actively-stats)
5. [معلمات الطلب العامة](#معلمات-الطلب-العامة)
6. [تنسيق التواريخ](#تنسيق-التواريخ)
7. [العلاقات المضمنة (Includes)](#العلاقات-المضمنة)
8. [رموز الأخطاء والتعامل معها](#رموز-الأخطاء-والتعامل-معها)
9. [ملاحظات للمطورين](#ملاحظات-للمطورين)

---

## مقدمة

توفر واجهة برمجة تطبيقات `Nano.StudyyearApi` نقاط نهاية RESTful للوصول إلى بيانات التقويم الدراسي المُدارة بواسطة إضافة `Tss.Studyyear`. يمكن لمطوري التطبيقات استخدام API لجلب السنوات الدراسية (`Periods`) والفصول الدراسية (`Semsters`) والأشهر الدراسية (`Months`). تم بناء API وفقاً لنمط `Nano.*Api` القياسي، مما يضمن الاتساق مع باقي خدمات API في النظام.

**Base URL:**  
`https://yourdomain.com/api/v1/studyyear`

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

**Periods**
```json
{
  "data": [
    {
      "id": 1,
      "code": "2-4-1",
      "barcode": "",
      "name": "2025-2026",
      "short_description": "",
      "description": "",
      "calendar_type": "years",
      "from_date": "2025-03-15 21:14:38",
      "to_date": "2026-03-15 21:14:38",
      "companys_id": "2",
      "departments_id": "4",
      "is_published_results": false,
      "is_published": true,
      "published_at": "",
      "unpublished_at": "",
      "is_default": true,
      "is_active": true,
      "status": "active",
      "sort_order": 1,
      "created_at": "2025-03-15 21:14:55",
      "updated_at": "2025-03-15 21:14:55",
      "object_type": "Tss\\Studyyear\\Models\\Period"
    },
    {
      "id": 2,
      "code": "2-4-2",
      "barcode": "",
      "name": "2023",
      "short_description": "",
      "description": "",
      "calendar_type": "years",
      "from_date": "2026-05-16 20:38:38",
      "to_date": "2027-05-16 20:38:38",
      "companys_id": "2",
      "departments_id": "4",
      "is_published_results": false,
      "is_published": true,
      "published_at": "",
      "unpublished_at": "",
      "is_default": false,
      "is_active": true,
      "status": "active",
      "sort_order": 2,
      "created_at": "2026-05-16 20:39:00",
      "updated_at": "2026-05-16 20:39:00",
      "object_type": "Tss\\Studyyear\\Models\\Period"
    }
  ],
  "meta": {
    "pagination": {
      "total": 2,
      "count": 2,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

**Semsters**
```json
{
  "data": [
    {
      "id": 1,
      "code": "2-4-1",
      "barcode": "",
      "name": "الفصل الدراسي الاول",
      "short_description": "",
      "description": "",
      "calendar_type": "months",
      "from_date": "2025-06-24 16:22:59",
      "to_date": "2025-10-23 16:22:59",
      "modul_type": "",
      "ref_type": "semster1",
      "study_year_id": "1",
      "is_published_results": false,
      "is_published": true,
      "published_at": "",
      "unpublished_at": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_default": true,
      "is_active": true,
      "status": "active",
      "sort_order": 1,
      "created_at": "2025-06-24 16:23:17",
      "updated_at": "2025-06-24 16:25:12",
      "object_type": "Tss\\Studyyear\\Models\\Semster"
    },
    {
      "id": 2,
      "code": "2-4-2",
      "barcode": "",
      "name": "الفصل الدراسي الثاني",
      "short_description": "",
      "description": "",
      "calendar_type": "months",
      "from_date": "2025-12-01 16:23:00",
      "to_date": "2026-04-01 16:23:00",
      "modul_type": "",
      "ref_type": "semster2",
      "study_year_id": "1",
      "is_published_results": false,
      "is_published": true,
      "published_at": "",
      "unpublished_at": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_default": false,
      "is_active": true,
      "status": "active",
      "sort_order": 2,
      "created_at": "2025-06-24 16:24:41",
      "updated_at": "2025-06-24 16:25:12",
      "object_type": "Tss\\Studyyear\\Models\\Semster"
    }
  ],
  "meta": {
    "pagination": {
      "total": 2,
      "count": 2,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

**Months**
```json
{
  "data": [
    {
      "id": 1,
      "code": "2-4-1",
      "barcode": "",
      "name": "الشهر الاول",
      "short_description": "",
      "description": "",
      "calendar_type": "months",
      "from_date": "2025-06-24 16:25:23",
      "to_date": "2025-07-25 16:25:23",
      "modul_type": "",
      "ref_type": "",
      "study_year_id": "1",
      "semsters_id": "1",
      "month_num": "1",
      "month_number": "JUN",
      "is_published_results": false,
      "is_published": true,
      "published_at": "",
      "unpublished_at": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_default": false,
      "is_active": true,
      "status": "active",
      "sort_order": 1,
      "created_at": "2025-06-24 16:25:51",
      "updated_at": "2025-06-24 16:25:51",
      "object_type": "Tss\\Studyyear\\Models\\Month"
    },
    {
      "id": 2,
      "code": "2-4-2",
      "barcode": "",
      "name": "الشهر الثاني",
      "short_description": "",
      "description": "",
      "calendar_type": "months",
      "from_date": "2025-07-25 16:25:00",
      "to_date": "2025-08-25 16:25:00",
      "modul_type": "",
      "ref_type": "",
      "study_year_id": "1",
      "semsters_id": "1",
      "month_num": "2",
      "month_number": "JUL",
      "is_published_results": false,
      "is_published": true,
      "published_at": "",
      "unpublished_at": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_default": false,
      "is_active": true,
      "status": "active",
      "sort_order": 2,
      "created_at": "2025-06-24 16:26:36",
      "updated_at": "2025-06-24 16:26:36",
      "object_type": "Tss\\Studyyear\\Models\\Month"
    },
    {
      "id": 3,
      "code": "2-4-3",
      "barcode": "",
      "name": "الشهر الثالث",
      "short_description": "",
      "description": "",
      "calendar_type": "months",
      "from_date": "2025-08-25 16:26:00",
      "to_date": "2025-09-25 16:26:00",
      "modul_type": "",
      "ref_type": "",
      "study_year_id": "1",
      "semsters_id": "1",
      "month_num": "3",
      "month_number": "AUG",
      "is_published_results": false,
      "is_published": true,
      "published_at": "",
      "unpublished_at": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_default": false,
      "is_active": true,
      "status": "active",
      "sort_order": 3,
      "created_at": "2025-06-24 16:27:26",
      "updated_at": "2025-06-24 16:27:26",
      "object_type": "Tss\\Studyyear\\Models\\Month"
    }
  ],
  "meta": {
    "pagination": {
      "total": 3,
      "count": 3,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

### استجابة ناجحة (عنصر واحد)

**Periods**
```json
{
  "id": 1,
  "code": "2-4-1",
  "barcode": "",
  "name": "2025-2026",
  "short_description": "",
  "description": "",
  "calendar_type": "years",
  "from_date": "2025-03-15 21:14:38",
  "to_date": "2026-03-15 21:14:38",
  "companys_id": "2",
  "departments_id": "4",
  "is_published_results": false,
  "is_published": true,
  "published_at": "",
  "unpublished_at": "",
  "is_default": true,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-03-15 21:14:55",
  "updated_at": "2025-03-15 21:14:55",
  "object_type": "Tss\\Studyyear\\Models\\Period"
}
```

**Semsters**
```json
{
  "id": 1,
  "code": "2-4-1",
  "barcode": "",
  "name": "الفصل الدراسي الاول",
  "short_description": "",
  "description": "",
  "calendar_type": "months",
  "from_date": "2025-06-24 16:22:59",
  "to_date": "2025-10-23 16:22:59",
  "modul_type": "",
  "ref_type": "semster1",
  "study_year_id": "1",
  "is_published_results": false,
  "is_published": true,
  "published_at": "",
  "unpublished_at": "",
  "companys_id": "2",
  "departments_id": "4",
  "is_default": true,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-24 16:23:17",
  "updated_at": "2025-06-24 16:25:12",
  "object_type": "Tss\\Studyyear\\Models\\Semster"
}
```

**Months**
```json
{
  "id": 1,
  "code": "2-4-1",
  "barcode": "",
  "name": "الشهر الاول",
  "short_description": "",
  "description": "",
  "calendar_type": "months",
  "from_date": "2025-06-24 16:25:23",
  "to_date": "2025-07-25 16:25:23",
  "modul_type": "",
  "ref_type": "",
  "study_year_id": "1",
  "semsters_id": "1",
  "month_num": "1",
  "month_number": "JUN",
  "is_published_results": false,
  "is_published": true,
  "published_at": "",
  "unpublished_at": "",
  "companys_id": "2",
  "departments_id": "4",
  "is_default": false,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-24 16:25:51",
  "updated_at": "2025-06-24 16:25:51",
  "object_type": "Tss\\Studyyear\\Models\\Month"
}
```

### استجابة خطأ

```json
{
  "code": "UNAUTHORIZED",
  "http_code": 401,
  "status": false,
  "message": "User ref_type 'Tss\\Student\\Models\\Mparent' is not allowed for operation 'show'.",
  "error": "User ref_type 'Tss\\Student\\Models\\Mparent' is not allowed for operation 'show'."
}
```

```json
{
  "error": {
    "code": 400,
    "message": "Listing periods is disabled for frontend users.",
    "errors": []
  }
}
```

---

## قائمة نقاط النهاية

### 1. السنوات الدراسية (Periods)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/studyyear/periods` | جلب قائمة السنوات الدراسية |
| `GET` | `/api/v1/studyyear/periods/{id}` | جلب سنة دراسية محددة |
| `GET` | `/api/v1/studyyear/periods/activelystats` | إحصائيات نشاط السنوات الدراسية |

**معلمات الفلترة:**

| المعامل | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `companys_id` | string | - | تصفية حسب معرف الشركة |
| `departments_id` | string | - | تصفية حسب معرف الفرع |
| `isActive` | bool\|string | `1` | `1` للنشط فقط، `0` للغير نشط، `*` للكل |
| `calendar_type` | string | `*` | نوع التقويم: `years` للسنوات، `*` للكل |
| `status` | string | `*` | حالة السنة: `active` أو `inactive`، `*` للكل |
| `q` | string | - | بحث نصي في `name`, `code`, `short_description` |
| `orderBy` | string | `sort_order` | ترتيب حسب حقل |
| `orderDirection` | string | `asc` | اتجاه الترتيب (`asc` أو `desc`) |
| `page` | int | 1 | رقم الصفحة |
| `per_page` | int | 15 | عدد العناصر في الصفحة |
| `exclude` | string | - | أسماء الحقول المستبعدة مفصولة بفواصل |
| `include` | string | - | العلاقات المضمنة (`company`, `department`) |

**العلاقات المتاحة:** `company`, `department`

**مثال للطلب:**

```bash
GET /api/v1/studyyear/periods?isActive=1&include=department
```

**مثال للاستجابة (عنصر واحد من القائمة):**

```json
{
  "id": 5,
  "code": "YR-2025-2026",
  "name": "العام الدراسي 2025-2026",
  "short_description": "عام دراسي كامل",
  "description": "العام الدراسي 2025-2026 لجميع المراحل",
  "calendar_type": "years",
  "from_date": "2025-09-01 00:00:00",
  "to_date": "2026-06-30 23:59:59",
  "companys_id": "1",
  "departments_id": "3",
  "is_published": true,
  "published_at": "2025-08-15 10:00:00",
  "unpublished_at": null,
  "is_default": true,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-01 08:30:00",
  "updated_at": "2025-08-20 14:15:00",
  "object_type": "Tss\\Studyyear\\Models\\Period",
  "department": {
    "data": {
      "id": "3",
      "name": "فرع المنطقة الشرقية"
    }
  }
}
```

---

### 2. الفصول الدراسية (Semsters)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/studyyear/semsters` | جلب قائمة الفصول الدراسية |
| `GET` | `/api/v1/studyyear/semsters/{id}` | جلب فصل دراسي محدد |
| `GET` | `/api/v1/studyyear/semsters/activelystats` | إحصائيات نشاط الفصول الدراسية |

**معلمات الفلترة:**

| المعامل | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `companys_id` | string | - | تصفية حسب معرف الشركة |
| `departments_id` | string | - | تصفية حسب معرف الفرع |
| `study_year_id` | string | **تلقائي** | معرف السنة الدراسية. **إذا لم يمرر، تستخدم السنة الافتراضية تلقائياً.** |
| `isActive` | bool\|string | `1` | `1` للنشط فقط، `0` للغير نشط، `*` للكل |
| `calendar_type` | string | `*` | نوع التقويم: `months`، `*` للكل |
| `ref_type` | string | `*` | نوع الفصل: `semster1`, `semster2`، `*` للكل |
| `status` | string | `*` | حالة الفصل: `active` أو `inactive`، `*` للكل |
| `q` | string | - | بحث نصي في `name`, `code`, `short_description` |
| `orderBy` | string | `sort_order` | ترتيب حسب حقل |
| `orderDirection` | string | `asc` | اتجاه الترتيب (`asc` أو `desc`) |
| `page` | int | 1 | رقم الصفحة |
| `per_page` | int | 15 | عدد العناصر في الصفحة |
| `exclude` | string | - | أسماء الحقول المستبعدة مفصولة بفواصل |
| `include` | string | - | العلاقات المضمنة (`company`, `department`, `period`) |

**العلاقات المتاحة:** `company`, `department`, `period`

**مثال للطلب (جلب فصول السنة الافتراضية مع تضمين السنة):**

```bash
GET /api/v1/studyyear/semsters?include=period
```

**مثال للاستجابة (عنصر واحد):**

```json
{
  "id": 12,
  "code": "SEM-012",
  "name": "الفصل الدراسي الأول",
  "short_description": "الفصل الأول 2025-2026",
  "description": "الفصل الدراسي الأول من العام 2025-2026",
  "calendar_type": "months",
  "from_date": "2025-09-01 00:00:00",
  "to_date": "2026-01-15 23:59:59",
  "modul_type": "studyyear",
  "ref_type": "semster1",
  "study_year_id": "5",
  "companys_id": "1",
  "departments_id": "3",
  "is_published": true,
  "published_at": "2025-08-15 10:00:00",
  "unpublished_at": null,
  "is_default": false,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-10 09:00:00",
  "updated_at": "2025-08-20 14:20:00",
  "object_type": "Tss\\Studyyear\\Models\\Semster",
  "period": {
    "data": {
      "id": 5,
      "name": "العام الدراسي 2025-2026"
    }
  }
}
```

---

### 3. الأشهر الدراسية (Months)

| الطريقة | المسار | الوصف |
|--------|--------|-------|
| `GET` | `/api/v1/studyyear/months` | جلب قائمة الأشهر الدراسية |
| `GET` | `/api/v1/studyyear/months/{id}` | جلب شهر دراسي محدد |
| `GET` | `/api/v1/studyyear/months/activelystats` | إحصائيات نشاط الأشهر الدراسية |

**معلمات الفلترة:**

| المعامل | النوع | القيمة الافتراضية | الوصف |
|--------|------|------------------|-------|
| `companys_id` | string | - | تصفية حسب معرف الشركة |
| `departments_id` | string | - | تصفية حسب معرف الفرع |
| `study_year_id` | string | **تلقائي** | معرف السنة الدراسية. **إذا لم يمرر، تستخدم السنة الافتراضية تلقائياً.** |
| `semsters_id` | string | - | معرف الفصل الدراسي |
| `month_num` | string | `*` | رقم الشهر (1،2،3)، `*` للكل |
| `calendar_type` | string | `*` | نوع التقويم: `months`، `*` للكل |
| `status` | string | `*` | حالة الشهر: `active` أو `inactive`، `*` للكل |
| `isActive` | bool\|string | `1` | `1` للنشط فقط، `0` للغير نشط، `*` للكل |
| `q` | string | - | بحث نصي في `name`, `code`, `short_description` |
| `orderBy` | string | `sort_order` | ترتيب حسب حقل (مفيد: `month_num` للترتيب زمنياً) |
| `orderDirection` | string | `asc` | اتجاه الترتيب (`asc` أو `desc`) |
| `page` | int | 1 | رقم الصفحة |
| `per_page` | int | 15 | عدد العناصر في الصفحة |
| `exclude` | string | - | أسماء الحقول المستبعدة مفصولة بفواصل |
| `include` | string | - | العلاقات المضمنة (`company`, `department`, `period`, `semster`) |

**العلاقات المتاحة:** `company`, `department`, `period`, `semster`

**مثال للطلب (جلب أشهر فصل محدد مع ترتيبها):**

```bash
GET /api/v1/studyyear/months?semsters_id=12&orderBy=month_num&orderDirection=asc&include=semster
```

**مثال للاستجابة (عنصر واحد):**

```json
{
  "id": 35,
  "code": "MON-035",
  "name": "الشهر الأول - الفصل الأول",
  "short_description": "سبتمبر - أكتوبر",
  "description": "الشهر الدراسي الأول من الفصل الدراسي الأول",
  "calendar_type": "months",
  "from_date": "2025-09-01 00:00:00",
  "to_date": "2025-10-01 23:59:59",
  "modul_type": "studyyear",
  "ref_type": "semster1",
  "study_year_id": "5",
  "semsters_id": "12",
  "month_num": "1",
  "month_number": "SEP",
  "companys_id": "1",
  "departments_id": "3",
  "is_published": true,
  "published_at": "2025-08-15 10:00:00",
  "unpublished_at": null,
  "is_default": false,
  "is_active": true,
  "status": "active",
  "sort_order": 1,
  "created_at": "2025-06-15 10:00:00",
  "updated_at": "2025-08-20 14:25:00",
  "object_type": "Tss\\Studyyear\\Models\\Month",
  "semster": {
    "data": {
      "id": 12,
      "name": "الفصل الدراسي الأول"
    }
  }
}
```

---

### 4. إحصائيات النشاط (Actively Stats)

كل مورد يمتلك نقطة نهاية `activelystats` تعيد معلومات عن آخر تحديث للبيانات، وتفيد في إبطال الكاش في التطبيقات العميلة.

**طلب مثال:**  
`GET /api/v1/studyyear/periods/activelystats`

**استجابة مثال:**

```json
{
  "data": {
    "code": 200,
    "data": {
      "last_updated": "2026-05-05 09:00:00",
      "activity_stats": false
    }
  }
}
```

- `last_updated`: طابع زمني لآخر تحديث.
- `activity_stats`: `true` إذا كان هناك نشاط أحدث من الوقت الحالي (عادة `false`).

---

## معلمات الطلب العامة

تنطبق هذه المعلمات على جميع نقاط النهاية `list`:

| المعامل | النوع | الافتراضي | الوصف |
|--------|------|----------|-------|
| `isActive` | mixed | `1` | تصفية النشط: `1` نشط، `0` غير نشط، `*` الكل |
| `q` | string | - | بحث نصي في الحقول الأساسية (الاسم، الكود، الوصف). |
| `orderBy` | string | يختلف | اسم الحقل للترتيب حسبه (مثلاً `sort_order`, `name`, `from_date`). |
| `orderDirection` | string | `asc` | اتجاه الترتيب: `asc` تصاعدي، `desc` تنازلي. |
| `page` | int | 1 | رقم الصفحة للتقسيم. |
| `per_page` | int | 15 | عدد العناصر في الصفحة. |
| `exclude` | string | - | أسماء حقول مستبعدة من الاستجابة، مفصولة بفواصل (مثلاً `created_at,updated_at`). |
| `include` | string | - | أسماء العلاقات المراد تضمينها، مفصولة بفواصل (مثلاً `period,semster`). |

---

## تنسيق التواريخ

جميع التواريخ في الاستجابات تكون بصيغة `Y-m-d H:i:s` (مثال: `"2025-09-01 00:00:00"`). الحقول الفارغة تعاد بقيمة `null`.

---

## العلاقات المضمنة

لتضمين كائنات مرتبطة في نفس الطلب، استخدم المعامل `include` بقيمة تحتوي على أسماء العلاقات مفصولة بفواصل.

| المورد | العلاقات المتاحة |
|--------|-----------------|
| `periods` | `company`, `department` |
| `semsters` | `company`, `department`, `period` |
| `months` | `company`, `department`, `period`, `semster` |

**مثال لتضمين السنة والفرع في جلب الفصول:**

```
GET /api/v1/studyyear/semsters?include=period,department
```

إذا كانت العلاقة غير موجودة أو حدث خطأ، تعاد كمصفوفة فارغة `[]` بدلاً من فشل الطلب.

---

## رموز الأخطاء والتعامل معها

| الكود | المعنى |
|------|--------|
| 200 | نجاح - تم جلب البيانات. |
| 400 | خطأ في الطلب - المعطيات غير صالحة أو العملية معطلة إعدادياً أو الصلاحيات غير كافية. |
| 401 | غير مصرح - التوكن مفقود أو غير صالح. |
| 404 | غير موجود - المورد غير موجود. |
| 500 | خطأ خادم داخلي. |

**جسم الخطأ النموذجي:**

```json
{
  "error": {
    "code": 400,
    "message": "Listing semesters is disabled for frontend users.",
    "errors": []
  }
}
```

- `message`: رسالة الخطأ باللغة المحددة (الافتراضي الإنجليزية).
- `errors`: تفاصيل إضافية عن الحقول الفاشلة (إن وجدت).

---

## ملاحظات للمطورين

- **السنة الدراسية الافتراضية التلقائية:** عند استعلام `semsters` أو `months` دون تحديد `study_year_id`، يستخدم API تلقائياً **السنة الدراسية الافتراضية** (المحددة في النظام). هذا يوفر عليك عناء تمرير السنة في كل مرة، خاصة في التطبيقات التي تعمل ضمن سياق السنة الحالية.
- **الترتيب الزمني للأشهر:** للحصول على الأشهر مرتبة زمنياً داخل الفصل، استخدم `orderBy=month_num&orderDirection=asc`.
- **تنسيق البيانات:** لاحظ أن الحقول `is_published`, `is_default`, `is_active` هي قيم منطقية (`true`/`false`)، بينما `sort_order` هو عدد صحيح (`int`).
- **تقليل حجم البيانات:** استخدم `exclude` لتجنب تحميل حقول غير ضرورية مثل `config_data`, `other_data`، خاصة عند جلب قوائم كبيرة.
- **التخزين المؤقت:** استخدم `activelystats` لمعرفة ما إذا كانت البيانات قد تغيرت منذ آخر جلب، مما يساعد في استراتيجيات التخزين المؤقت في تطبيقك.

هذا التوثيق يغطي جميع نقاط API المتاحة في الإصدار 1.0.0 من `Nano.StudyyearApi`. لأية أسئلة أو توسعات، راجع دليل المطور الرئيسي للمشروع.