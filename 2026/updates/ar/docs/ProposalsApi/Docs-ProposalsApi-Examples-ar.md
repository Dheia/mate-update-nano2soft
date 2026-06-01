# أمثلة عملية شاملة – إضافة `Nano2.ProposalsApi`

يحتوي هذا المستند على أمثلة حقيقية ومباشرة لاستخدام جميع نقاط النهاية الخاصة بإدارة **المقترحات (Proposals)**، **الشكاوي (Complaints)**، و**البلاغات (Reports)** عبر API. جميع البيانات التالية مستخرجة من استجابات فعلية للنظام، مع الحفاظ على الدقة والتنسيق الأصلي.

---

## فهرس الأمثلة

1. [جلب القائمة (List)](#1-جلب-القائمة-list)
   - 1.1 جميع السجلات
   - 1.2 المقترحات فقط
   - 1.3 الشكاوي فقط
   - 1.4 البلاغات فقط
   - 1.5 البلاغات ضد مستخدم معين
   - 1.6 البلاغات ضد أقسام (متاجر)
   - 1.7 البلاغات ضد قسم معين
   - 1.8 استخدام `is_stop_user` مع الطلاب (لأولياء الأمور)
     - 1.8.1 مقترحات مقدمة لطالب
     - 1.8.2 شكاوي مقدمة لطالب
     - 1.8.3 بلاغات مقدمة عن طالب
2. [عرض سجل واحد (Show)](#2-عرض-سجل-واحد-show)
3. [إنشاء سجل جديد (Create)](#3-إنشاء-سجل-جديد-create)
   - 3.1 إنشاء مقترح عام
   - 3.2 إنشاء شكوى
   - 3.3 إنشاء بلاغ ضد مستخدم
   - 3.4.1 إنشاء بلاغ ضد قسم (متجر)
   - 3.4.2 إنشاء بلاغ ضد منتج
4. [جلب خيارات الحقول (Options)](#4-جلب-خيارات-الحقول-options)
   - 4.1 جميع الخيارات (مصفوفة كائنات)
   - 4.2 خيارات حقل `type` فقط (مصفوفة كائنات)
   - 4.3 جميع الخيارات (كائن key-value)
5. [تحديث سجل (Update)](#5-تحديث-سجل-update)
6. [التحقق من آخر تحديث (ActivelyStats)](#6-التحقق-من-آخر-تحديث-activelystats)
7. [إدارة التصنيفات (Categories)](#7-إدارة-التصنيفات-categories)
   - 7.1 جلب جميع التصنيفات
   - 7.2 تصنيفات المقترحات فقط
   - 7.3 تصنيفات الشكاوي فقط
   - 7.4 تصنيفات البلاغات فقط
   - 7.5 عرض تصنيف واحد
   - 7.6 التحقق من آخر تحديث للتصنيفات

---

## 1. جلب القائمة (List)

### 1.1 جميع السجلات (مقترحات + شكاوي + بلاغات)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 5,
      "code": "2-4-5",
      "barcode": "",
      "name": "عنوان الشكوى رقم اثنين بعد التعديل",
      "content": "نص موضوع الشكوي رقم اثنان بعد التعديل",
      "type": "complaints",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:07:25",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 5,
      "created_at": "2023-12-04 18:07:25",
      "updated_at": "2023-12-04 18:13:39",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 4,
      "code": "2-4-4",
      "barcode": "",
      "name": "عنوان الشكوى رقم واحد",
      "content": "نص موضوع الشكوي رقم واحد",
      "type": "complaints",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:05:59",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 4,
      "created_at": "2023-12-04 18:05:59",
      "updated_at": "2023-12-04 18:05:59",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 3,
      "code": "2-4-3",
      "barcode": "",
      "name": "عنوان المقترح الثالث",
      "content": "نص موضوع المقترح الثالث",
      "type": "proposals",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 17:59:07",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 3,
      "created_at": "2023-12-04 17:59:07",
      "updated_at": "2023-12-04 17:59:07",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 2,
      "code": "2-4-2",
      "barcode": "",
      "name": "عنوان المقترح الثاني",
      "content": "نص موضوع المقترح الثاني",
      "type": "proposals",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 17:47:11",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 2,
      "created_at": "2023-12-04 17:47:11",
      "updated_at": "2023-12-04 17:47:11",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 1,
      "code": "2-4-1",
      "barcode": "",
      "name": "عنوان المقترح الاول",
      "content": "نص موضوع المقترح الاول",
      "type": "proposals",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 17:41:11",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 1,
      "created_at": "2023-12-04 17:41:11",
      "updated_at": "2023-12-04 17:41:11",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    }
  ],
  "meta": {
    "pagination": {
      "total": 5,
      "count": 5,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.2 المقترحات فقط (`type=proposals`)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=proposals
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 3,
      "code": "2-4-3",
      "barcode": "",
      "name": "عنوان المقترح الثالث",
      "content": "نص موضوع المقترح الثالث",
      "type": "proposals",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 17:59:07",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 3,
      "created_at": "2023-12-04 17:59:07",
      "updated_at": "2023-12-04 17:59:07",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 2,
      "code": "2-4-2",
      "barcode": "",
      "name": "عنوان المقترح الثاني",
      "content": "نص موضوع المقترح الثاني",
      "type": "proposals",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 17:47:11",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 2,
      "created_at": "2023-12-04 17:47:11",
      "updated_at": "2023-12-04 17:47:11",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 1,
      "code": "2-4-1",
      "barcode": "",
      "name": "عنوان المقترح الاول",
      "content": "نص موضوع المقترح الاول",
      "type": "proposals",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 17:41:11",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 1,
      "created_at": "2023-12-04 17:41:11",
      "updated_at": "2023-12-04 17:41:11",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
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

---

### 1.3 الشكاوي فقط (`type=complaints`)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=complaints
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 5,
      "code": "2-4-5",
      "barcode": "",
      "name": "عنوان الشكوى رقم اثنين بعد التعديل",
      "content": "نص موضوع الشكوي رقم اثنان بعد التعديل",
      "type": "complaints",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:07:25",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 5,
      "created_at": "2023-12-04 18:07:25",
      "updated_at": "2023-12-04 18:13:39",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 4,
      "code": "2-4-4",
      "barcode": "",
      "name": "عنوان الشكوى رقم واحد",
      "content": "نص موضوع الشكوي رقم واحد",
      "type": "complaints",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "",
      "target_id": 0,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:05:59",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 4,
      "created_at": "2023-12-04 18:05:59",
      "updated_at": "2023-12-04 18:05:59",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
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

---

### 1.4 البلاغات فقط (`type=reports`)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=reports
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 10,
      "code": "2-4-10",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ على المستخدم رقم 550",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "RainLab\\User\\Models\\User",
      "target_id": 550,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 19:05:52",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 10,
      "created_at": "2023-12-04 19:05:52",
      "updated_at": "2023-12-04 19:05:52",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 9,
      "code": "2-4-9",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ عن المتجر رقم 4",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "Tss\\Basic\\Models\\Department",
      "target_id": 4,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 19:03:08",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 9,
      "created_at": "2023-12-04 19:03:08",
      "updated_at": "2023-12-04 19:03:08",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 8,
      "code": "2-4-8",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ عن المتجر رقم 4",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "Tss\\Basic\\Models\\Department",
      "target_id": 4,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:58:27",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 8,
      "created_at": "2023-12-04 18:58:27",
      "updated_at": "2023-12-04 18:58:27",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 7,
      "code": "2-4-7",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ عن المتجر رقم 4",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "TssBasicModelsDepartment",
      "target_id": 4,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:54:15",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 7,
      "created_at": "2023-12-04 18:54:15",
      "updated_at": "2023-12-04 18:54:15",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 6,
      "code": "2-4-6",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ هنا",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "RainLab\\User\\Models\\User",
      "target_id": 550,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:51:52",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 6,
      "created_at": "2023-12-04 18:51:52",
      "updated_at": "2023-12-04 18:51:52",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    }
  ],
  "meta": {
    "pagination": {
      "total": 5,
      "count": 5,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.5 البلاغات ضد مستخدم فرنت معين (`type=reports`, `target_type=User`, `target_id=550`)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=reports&target_type=RainLab\\User\\Models\\User&target_id=550
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 10,
      "code": "2-4-10",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ على المستخدم رقم 550",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "RainLab\\User\\Models\\User",
      "target_id": 550,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 19:05:52",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 10,
      "created_at": "2023-12-04 19:05:52",
      "updated_at": "2023-12-04 19:05:52",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 6,
      "code": "2-4-6",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ هنا",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "RainLab\\User\\Models\\User",
      "target_id": 550,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:51:52",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 6,
      "created_at": "2023-12-04 18:51:52",
      "updated_at": "2023-12-04 18:51:52",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
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

---

### 1.6 البلاغات ضد الأقسام (المتاجر) – `type=reports`, `target_type=Department`

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=reports&target_type=Tss\\Basic\\Models\\Department
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 11,
      "code": "2-4-11",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ عن المتجر رقم 3",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "Tss\\Basic\\Models\\Department",
      "target_id": 3,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 19:22:35",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 11,
      "created_at": "2023-12-04 19:22:35",
      "updated_at": "2023-12-04 19:22:35",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 9,
      "code": "2-4-9",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ عن المتجر رقم 4",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "Tss\\Basic\\Models\\Department",
      "target_id": 4,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 19:03:08",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 9,
      "created_at": "2023-12-04 19:03:08",
      "updated_at": "2023-12-04 19:03:08",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    },
    {
      "id": 8,
      "code": "2-4-8",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ عن المتجر رقم 4",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "Tss\\Basic\\Models\\Department",
      "target_id": 4,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 18:58:27",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 8,
      "created_at": "2023-12-04 18:58:27",
      "updated_at": "2023-12-04 18:58:27",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
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

---

### 1.7 البلاغات ضد قسم معين (ID=3)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=reports&target_type=Tss\\Basic\\Models\\Department&target_id=3
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 11,
      "code": "2-4-11",
      "barcode": "",
      "name": "عنوان البلاغ ان وجد",
      "content": "نص البلاغ عن المتجر رقم 3",
      "type": "reports",
      "categories_id": "",
      "user_type": "RainLab\\User\\Models\\User",
      "user_id": 549,
      "target_type": "Tss\\Basic\\Models\\Department",
      "target_id": 3,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 1,
      "date_at": "2023-12-04 19:22:35",
      "created_by": "",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 11,
      "created_at": "2023-12-04 19:22:35",
      "updated_at": "2023-12-04 19:22:35",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

### 1.8 استخدام `is_stop_user=1` مع الطلاب (لأولياء الأمور)

#### 1.8.1 مقترحات مقدمة لطالب معين (ID=8)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=proposals&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1&include=categorie
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 243,
      "code": "2-4-243",
      "barcode": "",
      "name": "مقترح لتحسين مهارات الطالبة",
      "content": "<p>الطالبة ممتازه ولكن يلزم تعليمها اسلوب الالقاء والمشاركة فى الاذاعه المدرسية&nbsp;</p>",
      "type": "proposals",
      "categories_id": "2",
      "user_type": "",
      "user_id": 0,
      "target_type": "Tss\\Student\\Models\\Student",
      "target_id": 8,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 0,
      "date_at": "2026-06-01 14:24:22",
      "created_by": "24",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 243,
      "created_at": "2026-06-01 14:24:22",
      "updated_at": "2026-06-01 15:26:26",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal",
      "categorie": {
        "id": 2,
        "code": "2-2-2",
        "name": "لتحسين الخدمة",
        "slug": "lthsyn-alkhdm",
        "short_description": "",
        "description": "<p>لتحسين الخدمة</p>",
        "meta_title": "لتحسين الخدمة",
        "meta_description": "",
        "keywords": "",
        "type": "proposals",
        "ref_type_class": "",
        "ref_key_values_class": "",
        "companys_id": "2",
        "departments_id": "2",
        "is_default": 0,
        "is_active": 1,
        "is_published": 1,
        "published_at": "",
        "unpublished_at": "",
        "main_sub": "sub",
        "parent_id": "1",
        "level": 2,
        "properties": [],
        "created_at": "2024-06-10 18:01:32",
        "updated_at": "2024-06-10 18:01:32",
        "image": null,
        "images": [],
        "files": [],
        "object_type": "Nano2\\Proposals\\Models\\Categorie"
      }
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

#### 1.8.2 شكاوي مقدمة لطالب معين (ID=8)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=complaints&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 246,
      "code": "2-4-246",
      "barcode": "",
      "name": "شكوى من قبل المعلمين",
      "content": "المشاغبة اثناء الحصة",
      "type": "complaints",
      "categories_id": "9",
      "user_type": "",
      "user_id": 0,
      "target_type": "Tss\\Student\\Models\\Student",
      "target_id": 8,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 0,
      "date_at": "2026-06-01 15:15:22",
      "created_by": "24",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 246,
      "created_at": "2026-06-01 15:15:58",
      "updated_at": "2026-06-01 15:15:58",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

#### 1.8.3 بلاغات مقدمة عن طالب معين (ID=8)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals?type=reports&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 245,
      "code": "2-4-245",
      "barcode": "",
      "name": "بلاغ بالتهجم على زميلة بالصف",
      "content": "التهجم على زميلة فى الصف",
      "type": "reports",
      "categories_id": "6",
      "user_type": "",
      "user_id": 0,
      "target_type": "Tss\\Student\\Models\\Student",
      "target_id": 8,
      "reply_at": "",
      "reply_content": "",
      "reply_by": "",
      "read_at": "",
      "is_important": 0,
      "is_relay": 0,
      "user_relay_id": "",
      "relay_date_at": "",
      "relay_content": "",
      "companys_id": "2",
      "departments_id": "4",
      "is_active": 1,
      "is_private": 0,
      "date_at": "2026-06-01 15:13:07",
      "created_by": "24",
      "properties": null,
      "other_data": null,
      "config_data": null,
      "sort_order": 245,
      "created_at": "2026-06-01 15:14:51",
      "updated_at": "2026-06-01 15:14:51",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Proposal"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

---

## 2. عرض سجل واحد (Show)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals/2
```

**الاستجابة**

```json
{
  "id": 2,
  "code": "2-4-2",
  "barcode": "",
  "name": "عنوان المقترح الثاني",
  "content": "نص موضوع المقترح الثاني",
  "type": "proposals",
  "categories_id": "",
  "user_type": "RainLab\\User\\Models\\User",
  "user_id": 549,
  "target_type": "",
  "target_id": 0,
  "reply_at": "",
  "reply_content": "",
  "reply_by": "",
  "read_at": "",
  "is_important": 0,
  "is_relay": 0,
  "user_relay_id": "",
  "relay_date_at": "",
  "relay_content": "",
  "companys_id": "2",
  "departments_id": "4",
  "is_active": 1,
  "is_private": 1,
  "date_at": "2023-12-04 17:47:11",
  "created_by": "",
  "properties": null,
  "other_data": null,
  "config_data": null,
  "sort_order": 2,
  "created_at": "2023-12-04 17:47:11",
  "updated_at": "2023-12-04 17:47:11",
  "image": null,
  "images": [],
  "files": [],
  "object_type": "Nano2\\Proposals\\Models\\Proposal"
}
```

---

## 3. إنشاء سجل جديد (Create)

### 3.1 إنشاء مقترح عام

**الطلب**

```http
POST http://localhost:8006/api/v1/proposals/proposals/create?name=%D8%B9%D9%86%D9%88%D8%A7%D9%86%20%D8%A7%D9%84%D9%85%D9%82%D8%AA%D8%B1%D8%AD%20%D8%A7%D9%84%D8%AB%D8%A7%D9%84%D8%AB&content=%D9%86%D8%B5%20%D9%85%D9%88%D8%B6%D9%88%D8%B9%20%D8%A7%D9%84%D9%85%D9%82%D8%AA%D8%B1%D8%AD%20%D8%A7%D9%84%D8%AB%D8%A7%D9%84%D8%AB
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تمت عملية الاضافة بنجاح ",
  "error": null,
  "errors": null,
  "proposals_id": 3,
  "data": {
    "id": 3,
    "code": "2-4-3",
    "barcode": "",
    "name": "عنوان المقترح الثالث",
    "content": "نص موضوع المقترح الثالث",
    "type": "proposals",
    "categories_id": "",
    "user_type": "RainLab\\User\\Models\\User",
    "user_id": 549,
    "target_type": "",
    "target_id": 0,
    "reply_at": "",
    "reply_content": "",
    "reply_by": "",
    "read_at": "",
    "is_important": 0,
    "is_relay": 0,
    "user_relay_id": "",
    "relay_date_at": "",
    "relay_content": "",
    "companys_id": "2",
    "departments_id": 4,
    "is_active": 1,
    "is_private": 1,
    "date_at": "2023-12-04 17:59:07",
    "created_by": "",
    "properties": null,
    "other_data": null,
    "config_data": null,
    "sort_order": null,
    "created_at": "2023-12-04 17:59:07",
    "updated_at": "2023-12-04 17:59:07",
    "image": null,
    "images": [],
    "files": [],
    "object_type": "Nano2\\Proposals\\Models\\Proposal"
  }
}
```

---

### 3.2 إنشاء شكوى (`type=complaints`)

**الطلب**

```http
POST http://localhost:8006/api/v1/proposals/proposals/create?name=%D8%B9%D9%86%D9%88%D8%A7%D9%86%20%D8%A7%D9%84%D8%B4%D9%83%D9%88%D9%89%20%D8%B1%D9%82%D9%85%20%D9%88%D8%A7%D8%AD%D8%AF%20&content=%D9%86%D8%B5%20%D9%85%D9%88%D8%B6%D9%88%D8%B9%20%D8%A7%D9%84%D8%B4%D9%83%D9%88%D9%8A%20%D8%B1%D9%82%D9%85%20%D9%88%D8%A7%D8%AD%D8%AF&type=complaints
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تمت عملية الاضافة بنجاح ",
  "error": null,
  "errors": null,
  "proposals_id": 4,
  "data": {
    "id": 4,
    "code": "2-4-4",
    "barcode": "",
    "name": "عنوان الشكوى رقم واحد",
    "content": "نص موضوع الشكوي رقم واحد",
    "type": "complaints",
    "categories_id": "",
    "user_type": "RainLab\\User\\Models\\User",
    "user_id": 549,
    "target_type": "",
    "target_id": 0,
    "reply_at": "",
    "reply_content": "",
    "reply_by": "",
    "read_at": "",
    "is_important": 0,
    "is_relay": 0,
    "user_relay_id": "",
    "relay_date_at": "",
    "relay_content": "",
    "companys_id": "2",
    "departments_id": 4,
    "is_active": 1,
    "is_private": 1,
    "date_at": "2023-12-04 18:05:59",
    "created_by": "",
    "properties": null,
    "other_data": null,
    "config_data": null,
    "sort_order": null,
    "created_at": "2023-12-04 18:05:59",
    "updated_at": "2023-12-04 18:05:59",
    "image": null,
    "images": [],
    "files": [],
    "object_type": "Nano2\\Proposals\\Models\\Proposal"
  }
}
```

---

### 3.3 إنشاء بلاغ ضد مستخدم (`type=reports`, `target_type=User`, `target_id=550`)

**الطلب**

```http
POST http://localhost:8006/api/v1/proposals/proposals/create?name=%D8%B9%D9%86%D9%88%D8%A7%D9%86%20%D8%A7%D9%84%D8%A8%D9%84%D8%A7%D8%BA%20%D8%A7%D9%86%20%D9%88%D8%AC%D8%AF%20&content=%D9%86%D8%B5%20%D8%A7%D9%84%D8%A8%D9%84%D8%A7%D8%BA%20%D8%B9%D9%84%D9%89%20%D8%A7%D9%84%D9%85%D8%B3%D8%AA%D8%AE%D8%AF%D9%85%20%D8%B1%D9%82%D9%85%20550&type=reports&target_type=RainLab\\User\\Models\\User&target_id=550
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تمت عملية الاضافة بنجاح ",
  "error": null,
  "errors": null,
  "proposals_id": 10,
  "data": {
    "id": 10,
    "code": "2-4-10",
    "barcode": "",
    "name": "عنوان البلاغ ان وجد",
    "content": "نص البلاغ على المستخدم رقم 550",
    "type": "reports",
    "categories_id": "",
    "user_type": "RainLab\\User\\Models\\User",
    "user_id": 549,
    "target_type": "RainLab\\User\\Models\\User",
    "target_id": 550,
    "reply_at": "",
    "reply_content": "",
    "reply_by": "",
    "read_at": "",
    "is_important": 0,
    "is_relay": 0,
    "user_relay_id": "",
    "relay_date_at": "",
    "relay_content": "",
    "companys_id": "2",
    "departments_id": 4,
    "is_active": 1,
    "is_private": 1,
    "date_at": "2023-12-04 19:05:52",
    "created_by": "",
    "properties": null,
    "other_data": null,
    "config_data": null,
    "sort_order": null,
    "created_at": "2023-12-04 19:05:52",
    "updated_at": "2023-12-04 19:05:52",
    "image": null,
    "images": [],
    "files": [],
    "object_type": "Nano2\\Proposals\\Models\\Proposal"
  }
}
```

---

### 3.4.1 إنشاء بلاغ ضد قسم (متجر) مع تصنيف

**الطلب**

```http
POST http://localhost:8006/api/v1/proposals/proposals/create?name=%D8%B9%D9%86%D9%88%D8%A7%D9%86%20%D8%A7%D9%84%D8%A8%D9%84%D8%A7%D8%BA%20%D8%A7%D9%86%20%D9%88%D8%AC%D8%AF%20&content=%D9%86%D8%B5%20%D8%A7%D9%84%D8%A8%D9%84%D8%A7%D8%BA%20%D8%B9%D9%86%20%D8%A7%D9%84%D9%85%D8%AA%D8%AC%D8%B1%20%D8%B1%D9%82%D9%85%204%20&type=reports&target_type=Tss\\Basic\\Models\\Department&target_id=4&categories_id=10
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تمت عملية الاضافة بنجاح ",
  "error": null,
  "errors": null,
  "proposals_id": 9,
  "data": {
    "id": 9,
    "code": "2-4-9",
    "barcode": "",
    "name": "عنوان البلاغ ان وجد",
    "content": "نص البلاغ عن المتجر رقم 4",
    "type": "reports",
    "categories_id": "10",
    "user_type": "RainLab\\User\\Models\\User",
    "user_id": 549,
    "target_type": "Tss\\Basic\\Models\\Department",
    "target_id": 4,
    "reply_at": "",
    "reply_content": "",
    "reply_by": "",
    "read_at": "",
    "is_important": 0,
    "is_relay": 0,
    "user_relay_id": "",
    "relay_date_at": "",
    "relay_content": "",
    "companys_id": "2",
    "departments_id": 4,
    "is_active": 1,
    "is_private": 1,
    "date_at": "2023-12-04 19:03:08",
    "created_by": "",
    "properties": null,
    "other_data": null,
    "config_data": null,
    "sort_order": null,
    "created_at": "2023-12-04 19:03:08",
    "updated_at": "2023-12-04 19:03:08",
    "image": null,
    "images": [],
    "files": [],
    "object_type": "Nano2\\Proposals\\Models\\Proposal"
  }
}
```

---

### 3.4.2 إنشاء بلاغ ضد منتج (`target_type=products`)

**الطلب**

```http
POST http://localhost:8006/api/v1/proposals/proposals/create?name=%D8%B9%D9%86%D9%88%D8%A7%D9%86%20%D8%A7%D9%84%D8%A8%D9%84%D8%A7%D8%BA%20%D8%A7%D9%86%20%D9%88%D8%AC%D8%AF%20&content=%D9%86%D8%B5%20%D8%A7%D9%84%D8%A8%D9%84%D8%A7%D8%BA%20%D8%B9%D9%86%20%D8%A7%D9%84%D9%85%D8%AA%D8%AC%D8%B1%20%D8%B1%D9%82%D9%85%204%20&type=reports&target_type=products&target_id=2&categories_id=10
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تمت عملية الاضافة بنجاح ",
  "error": null,
  "errors": null,
  "proposals_id": 9,
  "data": {
    "id": 9,
    "code": "2-4-9",
    "barcode": "",
    "name": "عنوان البلاغ ان وجد",
    "content": "نص البلاغ عن الصنف رقم 2",
    "type": "reports",
    "categories_id": "10",
    "user_type": "RainLab\\User\\Models\\User",
    "user_id": 549,
    "target_type": "products",
    "target_id": 2,
    "reply_at": "",
    "reply_content": "",
    "reply_by": "",
    "read_at": "",
    "is_important": 0,
    "is_relay": 0,
    "user_relay_id": "",
    "relay_date_at": "",
    "relay_content": "",
    "companys_id": "2",
    "departments_id": 4,
    "is_active": 1,
    "is_private": 1,
    "date_at": "2023-12-04 19:03:08",
    "created_by": "",
    "properties": null,
    "other_data": null,
    "config_data": null,
    "sort_order": null,
    "created_at": "2023-12-04 19:03:08",
    "updated_at": "2023-12-04 19:03:08",
    "image": null,
    "images": [],
    "files": [],
    "object_type": "Nano2\\Proposals\\Models\\Proposal"
  }
}
```

---

## 4. جلب خيارات الحقول (Options)

### 4.1 جميع الخيارات (مصفوفة كائنات)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals/options
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تم جلب الخيارات بنجاح ",
  "error": "",
  "data": {
    "type": [
      {
        "id": "proposals",
        "name": "مقترحات"
      },
      {
        "id": "complaints",
        "name": "شكاوي"
      },
      {
        "id": "reports",
        "name": "بلاغ"
      }
    ],
    "status": [
      {
        "id": "active",
        "name": "نشط"
      },
      {
        "id": "inactive",
        "name": "غير نشط"
      }
    ]
  }
}
```

### 4.2 خيارات حقل `type` فقط (مصفوفة كائنات)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals/options?fields=type
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تم جلب الخيارات بنجاح ",
  "error": "",
  "data": {
    "type": [
      {
        "id": "proposals",
        "name": "مقترحات"
      },
      {
        "id": "complaints",
        "name": "شكاوي"
      },
      {
        "id": "reports",
        "name": "بلاغ"
      }
    ]
  }
}
```

### 4.3 جميع الخيارات ككائن (key-value)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/proposals/options?is_collection=0
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تم جلب الخيارات بنجاح ",
  "error": "",
  "data": {
    "type": {
      "proposals": "مقترحات",
      "complaints": "شكاوي",
      "reports": "بلاغ"
    },
    "status": {
      "active": "نشط",
      "inactive": "غير نشط"
    }
  }
}
```

---

## 5. تحديث سجل (Update)

**الطلب** – تعديل عنوان ومحتوى الشكوى رقم 5

```http
PUT http://localhost:8006/api/v1/proposals/proposals/update/5?name=%D8%B9%D9%86%D9%88%D8%A7%D9%86%20%D8%A7%D9%84%D8%B4%D9%83%D9%88%D9%89%20%D8%B1%D9%82%D9%85%20%D8%A7%D8%AB%D9%86%D9%8A%D9%86%20%D8%A8%D8%B9%D8%AF%20%D8%A7%D9%84%D8%AA%D8%B9%D8%AF%D9%8A%D9%84&content=%D9%86%D8%B5%20%D9%85%D9%88%D8%B6%D9%88%D8%B9%20%D8%A7%D9%84%D8%B4%D9%83%D9%88%D9%8A%20%D8%B1%D9%82%D9%85%20%D8%A7%D8%AB%D9%86%D8%A7%D9%86%20%D8%A8%D8%B9%D8%AF%20%D8%A7%D9%84%D8%AA%D8%B9%D8%AF%D9%8A%D9%84
```

**الاستجابة**

```json
{
  "code": 200,
  "status": true,
  "message": "تم التعديل بنجاح ",
  "error": "",
  "data": {
    "id": 5,
    "code": "2-4-5",
    "barcode": "",
    "name": "عنوان الشكوى رقم اثنين بعد التعديل",
    "content": "نص موضوع الشكوي رقم اثنان بعد التعديل",
    "type": "complaints",
    "categories_id": "",
    "user_type": "RainLab\\User\\Models\\User",
    "user_id": 549,
    "target_type": "",
    "target_id": 0,
    "reply_at": "",
    "reply_content": "",
    "reply_by": "",
    "read_at": "",
    "is_important": 0,
    "is_relay": 0,
    "user_relay_id": "",
    "relay_date_at": "",
    "relay_content": "",
    "companys_id": "2",
    "departments_id": "4",
    "is_active": 1,
    "is_private": 1,
    "date_at": "2023-12-04 18:07:25",
    "created_by": "",
    "properties": null,
    "other_data": null,
    "config_data": null,
    "sort_order": 5,
    "created_at": "2023-12-04 18:07:25",
    "updated_at": "2023-12-04 18:13:39",
    "image": null,
    "images": [],
    "files": [],
    "object_type": "Nano2\\Proposals\\Models\\Proposal"
  }
}
```

---

## 6. التحقق من آخر تحديث (ActivelyStats)

**الطلب** – التحقق من وجود تحديثات بعد تاريخ 2022-12-15 17:10:00

```http
GET http://localhost:8006/api/v1/proposals/proposals/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 17:10:00",
    "last_updated": "2022-12-20 18:15:32",
    "other_updated": null
  }
}
```

**بدون تمرير تاريخ (استخدام الوقت الحالي)**

```http
GET http://localhost:8006/api/v1/proposals/proposals/activelystats
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": false,
    "check_date": "2022-12-20 18:17:04",
    "last_updated": "2022-12-20 18:15:32",
    "other_updated": null
  }
}
```

---

## 7. إدارة التصنيفات (Categories)

### 7.1 جلب جميع التصنيفات (بغض النظر عن النوع)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/categories
```

**الاستجابة** (مختصرة – أول عنصرين)

```json
{
  "data": [
    {
      "id": 2,
      "code": "2-2-2",
      "name": "لتحسين الخدمة",
      "slug": "lthsyn-alkhdm",
      "short_description": "",
      "description": "<p>لتحسين الخدمة<\/p>",
      "meta_title": "لتحسين الخدمة",
      "meta_description": "",
      "keywords": "",
      "type": "proposals",
      "ref_type_class": "",
      "ref_key_values_class": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "1",
      "level": 2,
      "properties": [],
      "created_at": "2024-06-10 18:01:32",
      "updated_at": "2024-06-10 18:01:32",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Categorie"
    },
    {
      "id": 5,
      "code": "2-2-5",
      "name": "تسجيل الدخول للتطبيق",
      "slug": "tsgyl-aldkhol-llttbyk",
      "short_description": "مشاكل فى تسجيل الدخول للتطبيق",
      "description": "<p>مشاكل فى تسجيل الدخول للتطبيق<\/p>",
      "meta_title": "تسجيل الدخول للتطبيق",
      "meta_description": "مشاكل فى تسجيل الدخول للتطبيق",
      "keywords": "",
      "type": "complaints",
      "ref_type_class": "",
      "ref_key_values_class": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "3",
      "level": 2,
      "properties": [],
      "created_at": "2024-06-10 18:14:01",
      "updated_at": "2024-06-10 18:14:01",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Categorie"
    }
  ],
  "meta": {
    "pagination": {
      "total": 5,
      "count": 5,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

### 7.2 تصنيفات المقترحات فقط (`type=proposals`)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/categories?type=proposals
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 2,
      "code": "2-2-2",
      "name": "لتحسين الخدمة",
      "slug": "lthsyn-alkhdm",
      "short_description": "",
      "description": "<p>لتحسين الخدمة<\/p>",
      "meta_title": "لتحسين الخدمة",
      "meta_description": "",
      "keywords": "",
      "type": "proposals",
      "ref_type_class": "",
      "ref_key_values_class": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "1",
      "level": 2,
      "properties": [],
      "created_at": "2024-06-10 18:01:32",
      "updated_at": "2024-06-10 18:01:32",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Categorie"
    },
    {
      "id": 8,
      "code": "2-2-8",
      "name": "تحسين فى التطبيق",
      "slug": "thsyn-f-alttbyk",
      "short_description": "",
      "description": "",
      "meta_title": "تحسين فى التطبيق",
      "meta_description": "",
      "keywords": "",
      "type": "proposals",
      "ref_type_class": "",
      "ref_key_values_class": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "1",
      "level": 2,
      "properties": [],
      "created_at": "2024-06-10 18:19:27",
      "updated_at": "2024-06-10 18:19:27",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Categorie"
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

### 7.3 تصنيفات الشكاوي فقط (`type=complaints`)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/categories?type=complaints
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 5,
      "code": "2-2-5",
      "name": "تسجيل الدخول للتطبيق",
      "slug": "tsgyl-aldkhol-llttbyk",
      "short_description": "مشاكل فى تسجيل الدخول للتطبيق",
      "description": "<p>مشاكل فى تسجيل الدخول للتطبيق<\/p>",
      "meta_title": "تسجيل الدخول للتطبيق",
      "meta_description": "مشاكل فى تسجيل الدخول للتطبيق",
      "keywords": "",
      "type": "complaints",
      "ref_type_class": "",
      "ref_key_values_class": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "3",
      "level": 2,
      "properties": [],
      "created_at": "2024-06-10 18:14:01",
      "updated_at": "2024-06-10 18:14:01",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Categorie"
    }
  ],
  "meta": {
    "pagination": {
      "total": 1,
      "count": 1,
      "per_page": 15,
      "current_page": 1,
      "total_pages": 1,
      "links": {}
    }
  }
}
```

### 7.4 تصنيفات البلاغات فقط (`type=reports`)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/categories?type=reports
```

**الاستجابة**

```json
{
  "data": [
    {
      "id": 6,
      "code": "2-2-6",
      "name": "عن مفقودات",
      "slug": "aan-mfkodat",
      "short_description": "بلاغ عن مفقودات",
      "description": "<p>بلاغ عن مفقودات&nbsp;<\/p>",
      "meta_title": "عن مفقودات",
      "meta_description": "بلاغ عن مفقودات",
      "keywords": "",
      "type": "reports",
      "ref_type_class": "",
      "ref_key_values_class": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "4",
      "level": 2,
      "properties": [],
      "created_at": "2024-06-10 18:14:42",
      "updated_at": "2024-06-10 18:14:42",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Categorie"
    },
    {
      "id": 7,
      "code": "2-2-7",
      "name": "تاخير فى الطلب",
      "slug": "takhyr-f-altlb",
      "short_description": "تاخير فى الطلب",
      "description": "<p>تاخير فى الطلب<\/p>",
      "meta_title": "تاخير فى الطلب",
      "meta_description": "تاخير فى الطلب",
      "keywords": "",
      "type": "reports",
      "ref_type_class": "",
      "ref_key_values_class": "",
      "companys_id": "2",
      "departments_id": "2",
      "is_default": 0,
      "is_active": 1,
      "is_published": 1,
      "published_at": "",
      "unpublished_at": "",
      "main_sub": "sub",
      "parent_id": "4",
      "level": 2,
      "properties": [],
      "created_at": "2024-06-10 18:15:39",
      "updated_at": "2024-06-10 18:15:39",
      "image": null,
      "images": [],
      "files": [],
      "object_type": "Nano2\\Proposals\\Models\\Categorie"
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

### 7.5 عرض تصنيف واحد (ID=2)

**الطلب**

```http
GET http://localhost:8006/api/v1/proposals/categories/2
```

**الاستجابة**

```json
{
  "id": 2,
  "code": "2-2-2",
  "name": "لتحسين الخدمة",
  "slug": "lthsyn-alkhdm",
  "short_description": "",
  "description": "<p>لتحسين الخدمة<\/p>",
  "meta_title": "لتحسين الخدمة",
  "meta_description": "",
  "keywords": "",
  "type": "proposals",
  "ref_type_class": "",
  "ref_key_values_class": "",
  "companys_id": "2",
  "departments_id": "2",
  "is_default": 0,
  "is_active": 1,
  "is_published": 1,
  "published_at": "",
  "unpublished_at": "",
  "main_sub": "sub",
  "parent_id": "1",
  "level": 2,
  "properties": [],
  "created_at": "2024-06-10 18:01:32",
  "updated_at": "2024-06-10 18:01:32",
  "image": null,
  "images": [],
  "files": [],
  "object_type": "Nano2\\Proposals\\Models\\Categorie"
}
```

### 7.6 التحقق من آخر تحديث للتصنيفات

**الطلب** (بتمرير تاريخ)

```http
GET http://localhost:8006/api/v1/proposals/categories/activelystats?date=2022-12-15%2017:10:00
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,
    "check_date": "2022-12-15 2017:10:00",
    "last_updated": "2024-01-12 16:28:16",
    "other_updated": {
      "activity_cache.categorie": "2024-01-12 16:28:16"
    }
  }
}
```

**بدون تاريخ**

```http
GET http://localhost:8006/api/v1/proposals/categories/activelystats
```

**الاستجابة**

```json
{
  "code": "200",
  "data": {
    "activity_stats": false,
    "check_date": "2024-01-12 16:28:30",
    "last_updated": "2024-01-12 16:28:30",
    "other_updated": {
      "activity_cache.categorie": "2024-01-12 16:28:30"
    }
  }
}
```

---

**تاريخ إعداد الأمثلة:** 2026-06-01  
**الاعتماد على استجابات فعلية من الإصدار 1.0.3 من `Nano2.ProposalsApi`**
