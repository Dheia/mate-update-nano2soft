# توثيق إضافة `Nano2.ProposalsApi`

## مقدمة

توفّر إضافة **Nano2.ProposalsApi** واجهة برمجة تطبيقات (API) متكاملة لإدارة **المقترحات (Proposals)** و **الشكاوي (Complaints)** و **البلاغات (Reports)** داخل نظام NanoSoft App / NanoSoft. تم تصميم الإضافة وفق نمط `Nano.*Api` لتتيح للمستخدمين (الأفراد، الطلاب، أولياء الأمور) إنشاء واستعراض وتعديل السجلات بسهولة، مع دعم التصنيفات (`categories`)، والبحث، والترتيب، والتخزين المؤقت، والتحكم بالصلاحيات.

---

## فهرس المحتويات

1. [المتطلبات الأساسية](#المتطلبات-الأساسية)
2. [مفاهيم أساسية](#مفاهيم-أساسية)
   - أنواع السجلات
   - الكائن المستهدف (`target_type` / `target_id`)
   - التصنيفات (`categories_id`)
   - دعم أولياء الأمور (`is_stop_user`)
3. [نقاط النهاية (Endpoints)](#نقاط-النهاية-endpoints)
   - [1. جلب قائمة المقترحات / الشكاوي / البلاغات](#1-جلب-قائمة-المقترحات--الشكاوي--البلاغات)
   - [2. عرض سجل واحد](#2-عرض-سجل-واحد)
   - [3. إنشاء سجل جديد](#3-إنشاء-سجل-جديد)
   - [4. تحديث سجل](#4-تحديث-سجل)
   - [5. جلب خيارات الحقول](#5-جلب-خيارات-الحقول)
   - [6. التحقق من آخر تحديث](#6-التحقق-من-آخر-تحديث)
   - [7. إدارة التصنيفات](#7-إدارة-التصنيفات)
4. [الفلاتر المدعومة](#الفلاتر-المدعومة)
5. [العلاقات القابلة للتضمين (`include`)](#العلاقات-القابلة-للتضمين-include)
6. [نماذج البيانات](#نماذج-البيانات)
7. [رموز الخطأ](#رموز-الخطأ)
8. [أمثلة شاملة](#أمثلة-شاملة)
9. [الخاتمة](#الخاتمة)

---

## المتطلبات الأساسية

- نظام NanoSoft App الإصدار 3.x أو 2.x مع دعم Laravel.
- إضافة `Nano.API` و `Nano.AuthApi` لتوفير المصادقة الأساسية والتحكم بالمستخدمين.
- قاعدة بيانات تحتوي على جداول الإضافة بعد تنفيذ `php artisan plugin:refresh Nano2.ProposalsApi`.
- **يجب تسجيل دخول المستخدم** لاستخدام أي نقطة نهاية (JWT Token أو Session Authentication).

---

## مفاهيم أساسية

### 1. أنواع السجلات (`type`)

| القيمة | الوصف |
|--------|-------|
| `proposals` | مقترحات عامة أو موجهة لجهة معينة (طالب، شركة، إلخ) |
| `complaints` | شكاوى ضد شخص أو جهة |
| `reports`  | بلاغات (غالباً ضد سلوك مخالف) |

> **ملاحظة:** عند إنشاء `complaints` أو `reports`، يجب توفير الحقلين `target_type` و `target_id`.

### 2. الكائن المستهدف (`target_type` / `target_id`)

- `target_type`: النموذج المستهدف (مثل `RainLab\User\Models\User`، `Tss\Student\Models\Student`، `Tss\Basic\Models\Department`، أو حتى `products` كقيمة رمزية).
- `target_id`: معرف الكائن المستهدف داخل ذلك النموذج.
- يمكن استخدام `target_type` فارغاً في حالة المقترحات العامة.

### 3. التصنيفات (`categories_id`)

- الإصدار 1.0.2 فما فوق يدعم تصنيف السجلات عبر تمرير `categories_id` عند الإنشاء أو الفلترة عند الجلب.
- يمكن جلب قائمة التصنيفات الخاصة بكل نوع عبر:
  - `/api/v1/proposals/categories?type=proposals`
  - `/api/v1/proposals/categories?type=complaints`
  - `/api/v1/proposals/categories?type=reports`

### 4. دعم أولياء الأمور (`is_stop_user`)

- بدءاً من الإصدار 1.0.3، يمكن لولي الأمر (مستخدم من نوع `parent` أو `student`) تجاوز فلتر المستخدم العادي باستخدام المعامل `is_stop_user=1`، شرط أن يحدّد `target_type` و `target_id` لطالب معين. هذا يسمح لولي الأمر برؤية جميع السجلات الخاصة بابنه.

---

## نقاط النهاية (Endpoints)

جميع نقاط النهاية تبدأ بـ `/api/v1/proposals/proposals` (باستثناء نقاط التصنيفات التي تبدأ بـ `/api/v1/proposals/categories` ونقطة `options`).

### 1. جلب قائمة المقترحات / الشكاوي / البلاغات

```
GET /api/v1/proposals/proposals
```

#### المعاملات الاختيارية (الفلاتر)

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `q` | string | بحث نصي في `name` و `content` |
| `type` | string | فلترة حسب النوع: `proposals`, `complaints`, `reports` (الكل إذا لم تمرر) |
| `categories_id` | integer/string | فلترة حسب تصنيف معين (يمكن تمرير `1,2,3` مفصولة بفواصل) |
| `target_type` | string | نوع الكائن المستهدف (مثل `Tss\Student\Models\Student`) |
| `target_id` | integer | معرف الكائن المستهدف |
| `is_stop_user` | boolean | إذا كان `1` والمستخدم من نوع `parent` أو `student` و `target_type` = طالب، يتم تجاوز فلتر `User()` وإظهار سجلات الطالب المحدد |
| `isFavorites` | boolean | جلب السجلات المفضلة لدى المستخدم الحالي |
| `isLikes` | boolean | جلب السجلات التي أعجب بها المستخدم الحالي |
| `isBookmarks` | boolean | جلب السجلات المحفوضة (إشارات مرجعية) |
| `isReactions` | boolean | جلب السجلات التي تفاعل معها المستخدم |
| `isActive` | boolean | `true` للسجلات النشطة فقط (افتراضي `true`) |
| `companys_id` | integer | فلترة حسب الشركة |
| `departments_id` | integer | فلترة حسب القسم |
| `orderBy` | string | ترتيب حسب عمود (افتراضي `created_at`) |
| `orderDirection` | string | `asc` أو `desc` (افتراضي `desc`) |
| `per_page` | integer | عدد النتائج في الصفحة (افتراضي 15) |
| `page` | integer | رقم الصفحة |
| `exclude` | string | استبعاد أعمدة من النتيجة (مثال `exclude=created_at,updated_at`) |
| `include` | string | تضمين علاقات (مفصولة بفواصل) مثل `categorie`, `user`, `companys`, `department`, `target` |

#### مثال 1: جلب جميع المقترحات (للمستخدم الحالي)

```http
GET /api/v1/proposals/proposals
Authorization: Bearer {token}
```

#### مثال 2: جلب مقترحات خاصة بطالب معين (ولي الأمر)

```http
GET /api/v1/proposals/proposals?type=proposals&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1
Authorization: Bearer {token}
```

#### مثال 3: جلب بلاغات ضد مستخدم Frontend معين

```http
GET /api/v1/proposals/proposals?type=reports&target_type=RainLab\User\Models\User&target_id=550
```

---

### 2. عرض سجل واحد

```
GET /api/v1/proposals/proposals/{id}
```

| المعامل | الوصف |
|---------|-------|
| `id` | معرف السجل (مطلوب) |

#### مثال

```http
GET /api/v1/proposals/proposals/2
Authorization: Bearer {token}
```

**الاستجابة** (نموذج مختصر):

```json
{
  "id": 2,
  "name": "عنوان المقترح",
  "content": "نص المقترح",
  "type": "proposals",
  "categories_id": "5",
  "target_type": null,
  "target_id": 0,
  "created_at": "2023-12-04 17:47:11",
  "updated_at": "2023-12-04 17:47:11"
}
```

---

### 3. إنشاء سجل جديد

```
POST /api/v1/proposals/proposals/create
```

#### الحقول المطلوبة والاختيارية

| الحقل | النوع | الحالة | الوصف |
|-------|-------|--------|-------|
| `name` | string | مطلوب | عنوان السجل (3-250 حرف) |
| `content` | string | مطلوب | نص السجل (على الأقل 3 أحرف) |
| `type` | string | اختياري | `proposals` (افتراضي), `complaints`, `reports` |
| `categories_id` | integer | اختياري | رقم التصنيف |
| `target_type` | string | **مطلوب** لنوعي `complaints` و `reports` | النموذج المستهدف (مثل `Tss\Student\Models\Student`) |
| `target_id` | integer | **مطلوب** لنوعي `complaints` و `reports` | معرف الكائن المستهدف |
| `image` | string/base64 | اختياري | صورة السجل (يمكن تمريرها كـ Base64 أو ملف مباشرة) |
| `is_private` | boolean | اختياري | هل السجل خاص (افتراضي `true`) |
| `date_at` | date | اختياري | تاريخ السجل (افتراضي الآن) |

> **ملاحظة:** عند إنشاء `reports`، يوصى بتمرير `categories_id` مناسب لتصنيف البلاغ (مثل التنمر، إساءة، إلخ).

#### مثال: إنشاء مقترح عام

```http
POST /api/v1/proposals/proposals/create
Content-Type: application/json
Authorization: Bearer {token}

{
  "name": "تطوير منصة التعلم",
  "content": "أقترح إضافة مكتبة فيديوهات تفاعلية..."
}
```

#### مثال: إنشاء بلاغ ضد طالب

```http
POST /api/v1/proposals/proposals/create
{
  "type": "reports",
  "categories_id": 6,
  "target_type": "Tss\\Student\\Models\\Student",
  "target_id": 8,
  "name": "التهجم على زميلة",
  "content": "تم التهجم على زميلة في الصف"
}
```

**الاستجابة** (مختصرة):

```json
{
  "code": 200,
  "status": true,
  "message": "تمت عملية الاضافة بنجاح",
  "proposals_id": 245,
  "data": { ... }
}
```

---

### 4. تحديث سجل

```
PUT /api/v1/proposals/proposals/update/{id}
```

يمكن تحديث الحقول التالية فقط (لا يمكن تغيير النوع أو المستخدم المستهدف):
- `name`
- `content`
- `categories_id`
- `is_active`
- `is_private`
- `date_at`
- `image`

#### مثال

```http
PUT /api/v1/proposals/proposals/update/5
Content-Type: application/json
Authorization: Bearer {token}

{
  "name": "عنوان معدل",
  "content": "نص معدل"
}
```

---

### 5. جلب خيارات الحقول

```
GET /api/v1/proposals/proposals/options
```

| المعامل | الوصف |
|---------|-------|
| `fields` | اسم الحقل (مثل `type` أو `status`) - ليعرض خيارات حقل واحد |
| `is_collection` | إذا كانت `0` ترجع الخيارات ككائن (key=>value)، إذا كانت `1` ترجع مصفوفة من الكائنات `{id, name}` (افتراضي `1`) |

#### مثال: جلب أنواع السجلات كمصفوفة كائنات

```http
GET /api/v1/proposals/proposals/options?fields=type
```

```json
{
  "code": 200,
  "data": {
    "type": [
      {"id": "proposals", "name": "مقترحات"},
      {"id": "complaints", "name": "شكاوي"},
      {"id": "reports", "name": "بلاغ"}
    ]
  }
}
```

---

### 6. التحقق من آخر تحديث

```
GET /api/v1/proposals/proposals/activelystats
```

| المعامل | الوصف |
|---------|-------|
| `date` | تاريخ مقارنة (صيغة `Y-m-d H:i:s`) – اختياري، الافتراضي الآن |

**الاستجابة**:

```json
{
  "code": "200",
  "data": {
    "activity_stats": true,   // true إذا كان هناك تحديث بعد التاريخ الممرر
    "check_date": "2026-06-01 12:00:00",
    "last_updated": "2026-06-01 15:30:00",
    "other_updated": null
  }
}
```

---

### 7. إدارة التصنيفات

#### 7.1 جلب قائمة التصنيفات

```
GET /api/v1/proposals/categories
```

| المعامل | الوصف |
|---------|-------|
| `type` | فلترة حسب نوع التصنيف: `proposals`, `complaints`, `reports` (الكل إذا لم تمرر) |
| `parent_id` | جلب التصنيفات الفرعية لتصنيف أب محدد |
| `isActive` | `true` للتصنيفات النشطة (افتراضي `true`) |
| `isPublished` | `true` للمنشورة (افتراضي `true`) |
| `IsMain` | `true` للتصنيفات الرئيسية (يؤثر حسب الإعدادات) |
| `IsSub` | `true` للتصنيفات الفرعية (افتراضي `true`) |
| `companys_id` / `departments_id` | فلترة حسب الشركة/القسم |
| `q` | بحث في الاسم والوصف |
| `orderBy` / `orderDirection` | ترتيب |
| `per_page` / `page` | صفحات |
| `exclude` | استبعاد الحقول |
| `include` | تضمين علاقات مثل `companys`, `department`, `parent`, `children` |

#### مثال: جلب تصنيفات المقترحات فقط

```http
GET /api/v1/proposals/categories?type=proposals
```

#### 7.2 عرض تصنيف واحد

```
GET /api/v1/proposals/categories/{id}
```

#### 7.3 التحقق من آخر تحديث للتصنيفات

```
GET /api/v1/proposals/categories/activelystats
```

---

## الفلاتر المدعومة

### فلاتر قائمة المقترحات (`/proposals`)

| الفلتر | النوع | مثال | ملاحظات |
|--------|-------|------|---------|
| `q` | string | `?q=تحسين` | بحث في `name` و `content` |
| `type` | string | `?type=proposals` | `proposals`, `complaints`, `reports` |
| `categories_id` | integer/string | `?categories_id=2` أو `?categories_id=2,5` | يمكن تمرير عدة IDs مفصولة بفواصل |
| `target_type` | string | `?target_type=Tss\Student\Models\Student` | يجب أن يكون class موجود |
| `target_id` | integer | `?target_id=8` | مع `target_type` |
| `is_stop_user` | boolean | `?is_stop_user=1` | لتجاوز فلتر المستخدم للطلاب وأولياء الأمور |
| `isFavorites` | boolean | `?isFavorites=1` | جلب المفضلة لدى المستخدم |
| `isLikes` | boolean | `?isLikes=1` | جلب التي أعجب بها |
| `isBookmarks` | boolean | `?isBookmarks=1` | جلب المحفوظة |
| `isReactions` | boolean | `?isReactions=1` | جلب التي تفاعل معها |
| `isActive` | boolean | `?isActive=1` | افتراضي `true` |
| `companys_id` | integer | `?companys_id=2` | فلترة حسب الشركة |
| `departments_id` | integer | `?departments_id=4` | فلترة حسب القسم |
| `orderBy` | string | `?orderBy=created_at` | أعمدة مثل `name`, `created_at` |
| `orderDirection` | string | `?orderDirection=asc` | `asc` أو `desc` |
| `per_page` | integer | `?per_page=20` | عدد النتائج |
| `page` | integer | `?page=2` | رقم الصفحة |
| `exclude` | string | `?exclude=created_at,updated_at` | استبعاد حقول من الاستجابة |
| `include` | string | `?include=categorie,user` | تضمين علاقات |

### فلاتر التصنيفات (`/categories`)

| الفلتر | النوع | مثال |
|--------|-------|------|
| `type` | string | `?type=proposals` |
| `parent_id` | integer | `?parent_id=1` |
| `isActive` | boolean | `?isActive=1` |
| `isPublished` | boolean | `?isPublished=1` |
| `IsMain` | boolean | `?IsMain=1` |
| `IsSub` | boolean | `?IsSub=1` |
| `companys_id` | integer | `?companys_id=2` |
| `departments_id` | integer | `?departments_id=4` |
| `q` | string | `?q=خدمة` |
| `orderBy` | string | `?orderBy=sort_order` |
| `orderDirection` | string | `?orderDirection=asc` |

---

## العلاقات القابلة للتضمين (`include`)

### في `/proposals` (نموذج `Proposal`)

| العلاقة | الوصف | مثال |
|---------|-------|------|
| `companys` | بيانات الشركة | `?include=companys` |
| `department` | بيانات القسم | `?include=department` |
| `categorie` | التصنيف المرتبط (من `categories_id`) | `?include=categorie` |
| `user` | المستخدم المنشئ للسجل | `?include=user` |
| `target` | الكائن المستهدف (Polymorphic) | `?include=target` |
| `user_object_favorite` | كائن الإعجاب المفضل للمستخدم (إن وجد) | `?include=user_object_favorite` |
| `user_object_like` | كائن الإعجاب (Like) | `?include=user_object_like` |
| `user_object_bookmark` | كائن الحفظ (Bookmark) | `?include=user_object_bookmark` |
| `user_object_reaction` | كائن التفاعل (Reaction) | `?include=user_object_reaction` |

> **ملاحظة:** `target` يعمل مع أي نموذج يدعم `MorphTo` (مثل `User`, `Student`, `Department`).

### في `/categories` (نموذج `Categorie`)

| العلاقة | الوصف |
|---------|-------|
| `companys` | الشركة |
| `department` | القسم |
| `parent` | التصنيف الأب |
| `children` | التصنيفات الفرعية (مع دعم الصفحات) |
| `user_object_favorite` | كائن الإعجاب المفضل للمستخدم |
| `user_object_like` | كائن الإعجاب |
| `user_object_bookmark` | كائن الحفظ |
| `user_object_reaction` | كائن التفاعل |

> **ملاحظة:** علاقة `proposals` غير مفعلة حالياً في المحول، ولكن يمكن تفعيلها إذا لزم الأمر.

---

## نماذج البيانات

### هيكل سجل المقترح/الشكوى/البلاغ (`Proposal`)

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `id` | int | المعرف الفريد |
| `code` | string | كود تلقائي (شركة-قسم-رقم) |
| `name` | string | العنوان |
| `content` | string (HTML) | النص (يدعم HTML) |
| `type` | string | `proposals`, `complaints`, `reports` |
| `categories_id` | int | معرف التصنيف المرتبط |
| `user_type` | string | نوع المستخدم المنشئ (Morph) |
| `user_id` | int | معرف المنشئ |
| `target_type` | string | نوع الكائن المستهدف (Morph) |
| `target_id` | int | معرف الكائن المستهدف |
| `is_active` | boolean | فعال أم لا |
| `is_private` | boolean | خاص (لا يراه إلا المنشئ والمسؤول) |
| `date_at` | datetime | تاريخ السجل |
| `created_by` | string | منشئ السجل (قد يكون ID) |
| `image` | object | كائن الصورة (إن وجدت) |
| `created_at` | datetime | تاريخ الإنشاء |
| `updated_at` | datetime | تاريخ آخر تعديل |
| `object_type` | string | `Nano2\Proposals\Models\Proposal` |

### هيكل التصنيف (`Categorie`)

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `id` | int | المعرف |
| `name` | string | اسم التصنيف |
| `type` | string | نوع التصنيف (`proposals`, `complaints`, `reports`) |
| `parent_id` | int | التصنيف الأب |
| `is_active` | boolean | نشط |
| `image` | object | صورة التصنيف |
| ... | ... | (باقي الحقول مثل `description`, `meta`, إلخ) |

---

## رموز الخطأ

| كود HTTP | المعنى |
|----------|--------|
| 200 | نجاح (يحتوي على `status: true` في البدن) |
| 400 | خطأ في الطلب (نقص بيانات، تحقق فاشل) – يحتوي البدن على `status: false` و `errors` |
| 401 | غير مصرح (المستخدم غير مسجل دخول أو التوكن غير صالح) |
| 404 | السجل غير موجود |

**نموذج خطأ التحقق**:

```json
{
  "code": 400,
  "status": false,
  "message": "لم تتام عملية الاضافة بنجاح",
  "error": "الخاصية name حقل مطلوب",
  "errors": {
    "name": ["الخاصية name حقل مطلوب"]
  },
  "input_data": { ... },
  "debug": { ... }   // فقط في وضع التصحيح
}
```

---

## أمثلة شاملة

### مثال 1: تسجيل الدخول والحصول على توكن (باستخدام `Nano.AuthApi`)

```http
POST /api/v1/auth/login
{
  "login": "parent@example.com",
  "password": "123456"
}
```

### مثال 2: جلب مقترحات طالب معين (ولي الأمر) مع تضمين التصنيف

```http
GET /api/v1/proposals/proposals?type=proposals&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1&include=categorie
Authorization: Bearer {token}
```

### مثال 3: إضافة بلاغ ضد متجر (Department)

```http
POST /api/v1/proposals/proposals/create
Authorization: Bearer {token}
{
  "type": "reports",
  "categories_id": 10,
  "target_type": "Tss\\Basic\\Models\\Department",
  "target_id": 4,
  "name": "تأخير في الشحن",
  "content": "تم تأخير الطلب أكثر من 10 أيام"
}
```

### مثال 4: تحديث شكوى موجودة (جعلها غير خاصة)

```http
PUT /api/v1/proposals/proposals/42
{
  "is_private": false,
  "content": "تم تعديل النص بعد مراجعة الإدارة"
}
```

### مثال 5: البحث عن بلاغات تحتوي كلمة "تنمر"

```http
GET /api/v1/proposals/proposals?type=reports&q=تنمر&per_page=20
```

### مثال 6: جلب خيارات أنواع السجلات (ككائن key-value)

```http
GET /api/v1/proposals/proposals/options?is_collection=0&fields=type
```

### مثال 7: جلب تصنيفات البلاغات فقط

```http
GET /api/v1/proposals/categories?type=reports
```

### مثال 8: التحقق من آخر تحديث للسجلات بعد تاريخ معين

```http
GET /api/v1/proposals/proposals/activelystats?date=2026-06-01%2000:00:00
```

---

## الخاتمة

يمثل الإصدار **1.0.3** من `Nano2.ProposalsApi` حلاً متكاملاً لإدارة المقترحات والشكاوي والبلاغات عبر واجهة برمجة تطبيقات RESTful. بفضل التصميم الموحد، والفلاتر المتعددة، ودعم العلاقات، والتكامل مع `Nano.AuthApi` و `Nano.MarkableApi`، يمكن للمطورين بناء تطبيقات مرنة مثل منصات التواصل المدرسي، وأنظمة الدعم، وتطبيقات الشكاوى بكل سهولة.

تدعم الإضافة:

- ثلاثة أنواع رئيسية من السجلات.
- تصنيفات حسب النوع.
- استهداف أي نموذج في النظام (Polymorphic).
- صلاحيات مرنة للمستخدمين العاديين وأولياء الأمور والطلاب.
- عمليات `CRUD` كاملة (إنشاء، قراءة، تحديث) مع التحقق من الصحة.
- البحث، الترتيب، التصفية المتقدمة، وتضمين العلاقات.

نوصي بالترقية إلى أحدث إصدار (1.0.3) للاستفادة من ميزة `is_stop_user` التي تمكن أولياء الأمور من مراقبة سجلات أبنائهم.

---

## وثائق إضافية

- [توثيق إضافة `Nano2.ProposalsApi`](./docs/ProposalsApi/Docs-ProposalsApi-ar.md)
- [توثيق مختصر إضافة ProposalsApi API (Nano2.ProposalsApi)](./docs/ProposalsApi/Docs-ProposalsApi-Short-ar.md)
- [أمثلة عملية شاملة – إضافة ProposalsApi](./docs/ProposalsApi/Docs-ProposalsApi-Examples-ar.md)
- [دليل تطوير إضافات Nano API (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)

**تاريخ آخر تحديث:** 2026-06-01  
**الإصدار الموثق:** 1.0.3  
