# توثيق إضافة `Nano2.ProposalsApi` (نسخة مختصرة )

## مقدمة

توفّر إضافة **Nano2.ProposalsApi** واجهة برمجة تطبيقات (API) متكاملة لإدارة **المقترحات (Proposals)** و **الشكاوي (Complaints)** و **البلاغات (Reports)** داخل نظام NanoSoft APP. تم تصميم الإضافة وفق نمط `Nano.*Api` لتتيح للمستخدمين (الأفراد، الطلاب، أولياء الأمور) إنشاء واستعراض وتعديل السجلات بسهولة، مع دعم التصنيفات (`categories`)، والبحث، والترتيب، والتخزين المؤقت، والتحكم بالصلاحيات.

---

## المتطلبات الأساسية

- نظام NanoSoft APP الإصدار 3.x أو 2.x مع دعم Laravel.
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

جميع نقاط النهاية تبدأ بـ `/api/v1/proposals/proposals` (باستثناء نقاط التصنيفات والـ options).

### 1. جلب القائمة (List)

```
GET /api/v1/proposals/proposals
```

#### المعاملات الاختيارية

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
| `companys_id` | integer | فلترة حسب الشركة |
| `departments_id` | integer | فلترة حسب القسم |
| `orderBy` | string | ترتيب حسب عمود (افتراضي `created_at`) |
| `orderDirection` | string | `asc` أو `desc` (افتراضي `desc`) |
| `per_page` | integer | عدد النتائج في الصفحة (افتراضي 15) |
| `page` | integer | رقم الصفحة |
| `exclude` | string | استبعاد أعمدة من النتيجة (مثال `exclude=created_at,updated_at`) |
| `include` | string | تضمين علاقات مثل `categorie`, `user`, `companys`, `department` |

#### مثال: جلب مقترحات خاصة بطالب معين (ولي الأمر)

```http
GET /api/v1/proposals/proposals?type=proposals&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1
Authorization: Bearer {token}
```

#### مثال: جلب شكاوى موجهة لطالب معين

```http
GET /api/v1/proposals/proposals?type=complaints&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1
```

#### مثال: جلب بلاغات ضد مستخدم Frontend

```http
GET /api/v1/proposals/proposals?type=reports&target_type=RainLab\User\Models\User&target_id=550
```

---

### 2. عرض سجل واحد (Show)

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

**الاستجابة** (نموذج):

```json
{
  "id": 2,
  "name": "عنوان المقترح",
  "content": "نص المقترح",
  "type": "proposals",
  "categories_id": "5",
  "target_type": null,
  "target_id": 0,
  ...
}
```

---

### 3. إنشاء سجل جديد (Create)

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
  "content": "أقترح إضافة مكتبة فيديوهات تفاعلية...",
  "type": "proposals"
}
```

#### مثال: إنشاء بلاغ ضد طالب (ولي الأمر)

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

**الاستجابة**:

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

### 4. تحديث سجل (Update)

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

### 5. جلب خيارات الحقول (Options)

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

### 6. التحقق من آخر تحديث (Actively Stats)

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

## نماذج البيانات

### هيكل سجل المقترح/الشكوى/البلاغ

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `id` | int | المعرف الفريد |
| `code` | string | كود تلقائي (شركة-قسم-رقم) |
| `name` | string | العنوان |
| `content` | string (HTML) | النص (يدعم HTML) |
| `type` | string | `proposals`, `complaints`, `reports` |
| `categories_id` | int | معرف التصنيف المرتبط |
| `user_type` | string | نوع المستخدم المنشئ |
| `user_id` | int | معرف المنشئ |
| `target_type` | string | نوع الكائن المستهدف |
| `target_id` | int | معرف الكائن المستهدف |
| `is_active` | boolean | فعال أم لا |
| `is_private` | boolean | خاص (لا يراه إلا المنشئ والمسؤول) |
| `date_at` | datetime | تاريخ السجل |
| `image` | object | كائن الصورة (إن وجدت) |
| `created_at` | datetime | تاريخ الإنشاء |
| `updated_at` | datetime | تاريخ آخر تعديل |
| `object_type` | string | ثابت `Nano2\Proposals\Models\Proposal` |

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
  "error": "الخاصية  name حقل مطلوب",
  "errors": {
    "name": ["الخاصية  name حقل مطلوب"]
  },
  "input_data": { ... },
  "debug": { ... } // فقط في وضع التصحيح
}
```

---

## أمثلة شاملة للتطبيق

### 1. تسجيل الدخول والحصول على التوكن (باستخدام Nano.AuthApi)

```http
POST /api/v1/auth/login
{
  "login": "parent@example.com",
  "password": "123456"
}
```

### 2. جلب مقترحات طالب معين (ولي الأمر)

```http
GET /api/v1/proposals/proposals?type=proposals&target_type=Tss\Student\Models\Student&target_id=8&is_stop_user=1&include=categorie
Authorization: Bearer {token}
```

### 3. إضافة بلاغ ضد متجر (Department)

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

### 4. تحديث شكوى موجودة

```http
PUT /api/v1/proposals/proposals/42
{
  "is_private": false,
  "content": "تم تعديل النص بعد مراجعة الإدارة"
}
```

### 5. البحث عن بلاغات تحتوي كلمة "تنمر"

```http
GET /api/v1/proposals/proposals?type=reports&q=تنمر&per_page=20
```

---

## وثائق إضافية

- [توثيق إضافة `Nano2.ProposalsApi`](./docs/ProposalsApi/Docs-ProposalsApi-ar.md)
- [توثيق مختصر إضافة ProposalsApi API (Nano2.ProposalsApi)](./docs/ProposalsApi/Docs-ProposalsApi-Short-ar.md)
- [أمثلة عملية شاملة – إضافة ProposalsApi](./docs/ProposalsApi/Docs-ProposalsApi-Examples-ar.md)
- [دليل تطوير إضافات Nano API (Nano-Api-SKILL.md)](./docs/mcp/Nano-Api-SKILL.md)
