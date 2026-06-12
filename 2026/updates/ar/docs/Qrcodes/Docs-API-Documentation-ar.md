# توثيق واجهة برمجة التطبيقات (API) – نظام إدارة الباركودات (QRcodes)

## 1. مقدمة تعريفية

توفّر واجهة برمجة التطبيقات (API) لإدارة الباركودات ضمن إضافة **Nano2.Qrcodes** مجموعة متكاملة من النقاط الطرفية (endpoints) التي تُمكّن المطورين من إنشاء الباركودات بأنواعها المختلفة (1D مثل Code 128، EAN-13، و 2D مثل QR code)، والتحقّق من صحّتها، وتسجيل استخدامها، وجلب إحصائيات الاستخدام، بالإضافة إلى توليد صور الباركود.

تتبع الواجهة بنية RESTful وتدعم تنسيق JSON. وهي محمية بنظام صلاحيات متقدّم (`AccessManager`) يفرق بين مستخدمي لوحة التحكم (backend)، ومستخدمي الواجهة الأمامية (frontend)، والزوار غير المسجلين (guest). يمكن تخصيص الصلاحيات عبر ملف الإعدادات `config.php` باستخدام متغيرات البيئة.

### الميزات الرئيسية:
- **عمليات CRUD** كاملة للباركودات (إنشاء، قراءة، تحديث، حذف).
- **التحقق من الباركود** دون استخدامه (فحص الصلاحية – نشاط، صلاحية، استهلاك).
- **مسح واستخدام الباركود** (زيادة عداد الاستخدامات وربطه بالمستخدم).
- **التحقق الإداري** (تعيين `is_verified` بواسطة مشرف).
- **توليد دفعة واحدة** لعدد كبير من الباركودات (حتى 1000).
- **إحصائيات** حول الباركودات وإحصائيات استخدام كل مستخدم.
- **توليد صورة الباركود** (PNG) مباشرة أو عبر رابط الصورة.
- **دعم التخزين المؤقت** (cache) للطلبات المتكررة (يمكن تفعيله عبر الإعدادات).

### الأساسيات التقنية:
- **نطاق القاعدة**: `/api/v1/qrcodes`
- **التوثيق**: يتم التعرف على المستخدم تلقائياً من خلال الجلسة (session) أو رمز المصادقة (token) الخاص بـ `Nano\AuthApi`. تعتمد الصلاحيات على دور المستخدم وصلاحياته المحددة في ملف `config.php`.
- **تنسيق الطلب والاستجابة**: `application/json`
- **رمز النجاح العام**: `200`
- **رموز الخطأ الشائعة**: `400`, `401`, `404`.

---

## 2. فهرس نقاط النهاية

| الطريقة | النقطة الطرفية | الوصف |
|---------|----------------|--------|
| GET | `/barcodes` | جلب قائمة الباركودات مع خيارات التصفية والترقيم. |
| GET | `/barcodes/activelystats` | الحصول على آخر توقيت تحديث للبيانات (للاستخدام مع الكاش). |
| POST | `/barcodes` | إنشاء باركود جديد. |
| PUT | `/barcodes/{id}` | تحديث باركود موجود. |
| DELETE | `/barcodes/{id}` | حذف (ناعم) لباركود. |
| GET | `/barcodes/{id}` | عرض تفاصيل باركود محدد. |
| POST | `/barcodes/verify` | التحقق من صحة باركود (دون استخدامه). |
| POST | `/barcodes/use` | مسح واستخدام باركود (تسجيل استخدام). |
| POST | `/barcodes/verify/{id}` | التحقق الإداري من باركود (تعيين `is_verified`). |
| POST | `/barcodes/generate` | توليد دفعة من الباركودات (للمشرفين فقط). |
| GET | `/barcodes/stats` | إحصائيات عامة عن الباركودات. |
| GET | `/barcodes/user-stats` | إحصائيات استخدام المستخدم الحالي. |
| GET | `/barcodes/image/{id}` | الحصول على صورة الباركود (PNG). |

---

## 3. تفاصيل نقاط النهاية

### 3.1. جلب قائمة الباركودات

#### **`GET /barcodes`**

**المقدمة**:  
تُستخدم هذه النقطة الطرفية لجلب قائمة الباركودات مع دعم التصفية، البحث، الترتيب، وتحديد عدد النتائج في الصفحة. يتم تطبيق نطاقات الصلاحيات تلقائياً بحيث يرى كل مستخدم فقط الباركودات المسموح له بها (الخاصة به، أو التابعة لقسمه، أو جميعها حسب صلاحياته).

**البراميترات** (جميعها اختيارية وتُمرّر كـ query string):

| البراميتر | النوع | الوصف | مثال |
|-----------|-------|-------|------|
| `page` | int | رقم الصفحة (للترقيم) | `page=2` |
| `per_page` | int | عدد النتائج في الصفحة (الحد الأقصى 100) | `per_page=20` |
| `orderBy` | string | اسم الحقل للترتيب (مثل `created_at`, `code`, `barcode`) | `orderBy=code` |
| `orderDirection` | string | `asc` أو `desc` | `orderDirection=desc` |
| `search` | string | نص البحث (في الحقول: `code`, `barcode`, `product_name`, `product_sku`) | `search=ABC` |
| `barcode_type` | string | نوع الباركود (`C128`, `EAN13`, `QR`, ...) | `barcode_type=QR` |
| `status` | string | الحالة (`active`, `inactive`, `used`, `expired`) | `status=active` |
| `is_active` | bool | `true` أو `false` | `is_active=true` |
| `is_verified` | bool | `true` أو `false` | `is_verified=true` |
| `is_use` | bool | `true` أو `false` | `is_use=false` |
| `companys_id` | int | معرف الشركة | `companys_id=2` |
| `departments_id` | int | معرف القسم | `departments_id=5` |
| `product_id` | int | معرف المنتج | `product_id=12` |
| `owner_id` | int | معرف المالك | `owner_id=7` |
| `owner_type` | string | نوع المالك (مثل `RainLab\User\Models\User`) | `owner_type=RainLab\User\Models\User` |
| `issue_date_from` | date | تاريخ الإصدار من | `issue_date_from=2025-01-01` |
| `issue_date_to` | date | تاريخ الإصدار إلى | `issue_date_to=2025-12-31` |
| `expiry_date_from` | date | تاريخ الانتهاء من | `expiry_date_from=2025-01-01` |
| `expiry_date_to` | date | تاريخ الانتهاء إلى | `expiry_date_to=2025-12-31` |
| `exclude` | string | أعمدة مستبعدة مفصولة بفواصل (لن تظهر في الاستجابة) | `exclude=metadata,other_data` |
| `advanced_filters` | - | يمكن استخدام `is_or_*`, `is_not_*`, `is_or_null_*` للحقول المسموحة (راجع قسم الصلاحيات) | `is_or_status=true` |

**مثال عملي**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes?page=1&per_page=5&barcode_type=C128&status=active&orderBy=created_at&orderDirection=desc"
```

**استجابة ناجحة (200 OK)**:

```json
{
    "data": [
        {
            "id": 10,
            "code": "BC000010",
            "barcode": "BC000010",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "used",
            "status_color": "info",
            "status_icon": "icon-check-circle text-info",
            "is_active": true,
            "is_verified": false,
            "is_use": true,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 1,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": "2026-06-04 20:45:17",
            "last_used_at": "2026-06-04 20:45:17",
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 20:45:17"
        },
        {
            "id": 9,
            "code": "BC000009",
            "barcode": "BC000009",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "used",
            "status_color": "info",
            "status_icon": "icon-check-circle text-info",
            "is_active": true,
            "is_verified": false,
            "is_use": true,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 1,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": "2026-06-04 23:32:35",
            "last_used_at": "2026-06-04 23:32:35",
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 23:32:35"
        },
        {
            "id": 8,
            "code": "BC000008",
            "barcode": "BC000008",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        },
        {
            "id": 7,
            "code": "BC000007",
            "barcode": "BC000007",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        },
        {
            "id": 6,
            "code": "BC000006",
            "barcode": "BC000006",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        },
        {
            "id": 5,
            "code": "BC000005",
            "barcode": "BC000005",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        },
        {
            "id": 4,
            "code": "BC000004",
            "barcode": "BC000004",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        },
        {
            "id": 3,
            "code": "BC000003",
            "barcode": "BC000003",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        },
        {
            "id": 2,
            "code": "BC000002",
            "barcode": "BC000002",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        },
        {
            "id": 1,
            "code": "BC000001",
            "barcode": "BC000001",
            "barcode_type": "C128",
            "barcode_type_name": "Code 128",
            "status": "active",
            "status_color": "success",
            "status_icon": "icon-check text-success",
            "is_active": true,
            "is_verified": false,
            "is_use": false,
            "is_published": false,
            "is_public": false,
            "is_default": false,
            "used_count": 0,
            "generated_count": 1,
            "product_sku": "",
            "product_name": "",
            "product_brand": "",
            "product_category": "",
            "unit_name": "",
            "price": 0,
            "old_price": 0,
            "quantity": 1,
            "batch_number": "",
            "issue_date": null,
            "expiry_date": "2027-06-04",
            "used_at": null,
            "last_used_at": null,
            "verified_at": null,
            "verification_score": 0,
            "image_url": null,
            "created_at": "2026-06-04 19:39:31",
            "updated_at": "2026-06-04 19:39:31"
        }
    ],
    "meta": {
        "pagination": {
            "total": 10,
            "count": 10,
            "per_page": 20,
            "current_page": 1,
            "total_pages": 1,
            "links": {}
        }
    }
}
```

**استجابة فاشلة (401 – غير مصرح)**:

```json
{
  "code": "UNAUTHORIZED",
  "http_code": 401,
  "status": false,
  "message": "You do not have the required permissions to perform 'list'.",
  "error": "You do not have the required permissions to perform 'list'."
}
```

---

### 3.2. الحصول على آخر تحديث (للكاش)

#### **`GET /barcodes/activelystats`**

**المقدمة**:  
نقطة طرفية خفيفة لإرجاع أحدث توقيت تم فيه تحديث أي باركود (`updated_at`). يستخدمها العميل لاتخاذ قرار بشأن إعادة تحميل البيانات المخزنة مؤقتاً.

**البراميترات**: لا توجد.

**مثال**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/activelystats"
```

**استجابة ناجحة (200)**:

```json
{
  "code": 200,
  "data": {
    "last_updated": "2025-03-15 14:30:00",
    "activity_stats": true
  }
}
```

---

### 3.3. إنشاء باركود جديد

#### **`POST /barcodes`**

**المقدمة**:  
إنشاء باركود جديد. الحقول الأساسية المطلوبة: `barcode` (القيمة المشفرة) و `barcode_type`. يتم إنشاء `code` تلقائياً إذا لم يُمرّر (مع توليد فريد). يمكن ربط الباركود بمنتج، مالك، أو مستخدم محدد. صلاحية الإنشاء: عادةً لمستخدمي لوحة التحكم فقط (يمكن تخصيصها).

**البراميترات** (تُمرّر في جسم الطلب بصيغة JSON):

| البراميتر | النوع | إلزامي | الوصف |
|-----------|-------|--------|-------|
| `barcode` | string | نعم | القيمة التي سيتم ترميزها داخل الباركود. |
| `barcode_type` | string | نعم | النوع (`C128`, `QR`, `EAN13`، ...). |
| `code` | string | لا | كود فريد اختياري (يُولد تلقائياً إذا لم يُعطى). |
| `product_id` | int | لا | معرف المنتج المرتبط. |
| `product_name` | string | لا | اسم المنتج. |
| `price` | float | لا | السعر. |
| `quantity` | float | لا | الكمية (افتراضي 1). |
| `owner_id` | int | لا | معرف المالك. |
| `owner_type` | string | لا | نوع المالك (يتم تعبئته تلقائياً من المستخدم الحالي إذا كان `create`). |
| `user_id` | int | لا | معرف المستخدم الذي سيسجل الاستخدام لاحقاً. |
| `issue_date` | date | لا | تاريخ الإصدار. |
| `expiry_date` | date | لا | تاريخ انتهاء الصلاحية. |
| `status` | string | لا | `active`, `inactive`, `pending` (افتراضي `active`). |
| `is_active` | bool | لا | `true` (افتراضي). |
| `is_published` | bool | لا | `false` (افتراضي). |
| `is_verified` | bool | لا | `false`. |
| `batch_number` | string | لا | رقم الدفعة. |
| `companys_id` | int | لا | معرف الشركة (يؤخذ من الإعدادات الافتراضية إذا لم يُمرر). |
| `departments_id` | int | لا | معرف القسم. |
| `fields_data`, `metadata`, ... | object | لا | بيانات JSON إضافية. |

**مثال**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes" \
  -H "Content-Type: application/json" \
  -d '{
    "barcode": "5901234123457",
    "barcode_type": "EAN13",
    "product_name": "كتاب تعلم البرمجة",
    "price": 45.00,
    "quantity": 1,
    "expiry_date": "2028-12-31"
  }'
```

**استجابة ناجحة (200)**:

```json
{
  "code": 200,
  "status": true,
  "message": "تم إنشاء الباركود بنجاح.",
  "error": null,
  "errors": null,
  "data": {
    "id": 110,
    "code": "BC000110",
    "barcode": "5901234123457",
    "barcode_type": "EAN13",
    "status": "active",
    ...
  },
  "input_data": { ... },
  "process_data": { ... }
}
```

**استجابة فاشلة (400 – بيانات مفقودة أو غير صالحة)**:

```json
{
  "code": 400,
  "status": false,
  "message": "حدث خطأ أثناء إنشاء الباركود.",
  "error": "The barcode field is required.",
  "errors": { "barcode": ["The barcode field is required."] }
}
```

---

### 3.4. تحديث باركود موجود

#### **`PUT /barcodes/{id}`**

**المقدمة**:  
تحديث بيانات باركود معين. لا يمكن تعديل الباركود الذي تم استخدامه بالفعل (`is_use = true`) إلا إذا تم تمرير `force_update=true`. تُحدَّث الحقول المُمرّرة فقط.

**البراميترات**: نفس البراميترات الخاصة بـ `POST /barcodes` ولكن يمكن تمرير حقول جزئية.

**مثال**:

```bash
curl -X PUT "https://yourdomain.com/api/v1/qrcodes/barcodes/110" \
  -H "Content-Type: application/json" \
  -d '{
    "price": 49.99,
    "is_published": true
  }'
```

**استجابة ناجحة** (200):

```json
{
  "code": 200,
  "status": true,
  "message": "تم تحديث الباركود بنجاح.",
  "error": null,
  "data": { ... }
}
```

---

### 3.5. حذف باركود (حذف ناعم)

#### **`DELETE /barcodes/{id}`**

**المقدمة**:  
حذف الباركود بشكل ناعم (soft delete) – يظل السجل موجوداً مع تعيين `deleted_at`. لا يمكن استرجاعه إلا عبر لوحة التحكم (لا توجد نقطة نهاية للاسترجاع في API).

**البراميترات**: لا توجد.

**مثال**:

```bash
curl -X DELETE "https://yourdomain.com/api/v1/qrcodes/barcodes/110"
```

**استجابة ناجحة** (200):

```json
{
  "code": 200,
  "status": true,
  "message": "تم حذف الباركود بنجاح.",
  "error": null,
  "data": null
}
```

**استجابة فاشلة (404)**:

```json
{
  "code": 400,
  "status": false,
  "message": "حدث خطأ أثناء حذف الباركود.",
  "error": "الباركود المطلوب غير موجود."
}
```

---

### 3.6. عرض تفاصيل باركود

#### **`GET /barcodes/{id}`**

**المقدمة**:  
استرجاع بيانات باركود واحد بالكامل، مع تطبيق نفس نطاق الصلاحيات المستخدم في جلب القائمة.

**البراميترات**: لا توجد.

**مثال**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/101"
```

**استجابة ناجحة** (200):

```json
{
  "code": 200,
  "status": true,
  "message": "",
  "data": { /* نفس بنية الكائن الفردي من القائمة */ }
}
```

---

### 3.7. التحقق من صحة باركود (بدون استخدام)

#### **`POST /barcodes/verify`**

**المقدمة**:  
التحقق مما إذا كان الباركود صالحاً للاستخدام (موجود، نشط، غير منتهي، لم يستنفذ عدد استخداماته) دون تغيير حالته. يُعيد معلومات وصفية حول حالته.

**البراميترات** (جسم الطلب JSON):

| البراميتر | النوع | إلزامي | الوصف |
|-----------|-------|--------|-------|
| `barcode` | string | نعم | قيمة الباركود المطلوب التحقق منه. |

**مثال**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/verify" \
  -H "Content-Type: application/json" \
  -d '{"barcode": "BC000101"}'
```

**استجابة (باركود صالح)**:

```json
{
  "code": 200,
  "status": true,
  "message": "الباركود صالح.",
  "error": null,
  "data": {
    "id": 101,
    "code": "BC000101",
    "barcode": "BC000101",
    "barcode_type": "C128",
    "is_active": true,
    "is_use": false,
    "is_expired": false,
    "verification_score": 100,
    "can_use": true
  }
}
```

**استجابة (باركود مستخدم سابقاً)**:

```json
{
  "code": 200,
  "status": false,
  "message": "تم استخدام هذا الباركود بالفعل.",
  "error": null,
  "data": { "id": 101, "is_use": true, "used_count": 1 }
}
```

---

### 3.8. مسح واستخدام باركود

#### **`POST /barcodes/use`**

**المقدمة**:  
تستخدم هذه النقطة لمسح باركود (مثلاً عبر تطبيق الهاتف) وتسجيل استخدامه. يتم زيادة `used_count`، تعيين `is_use = true` إذا بلغ العدد الحد الأقصى (`generated_count`)، وتسجيل `used_at` و `last_used_at` وربطه بالمستخدم الحالي (إن وُجد). بالإضافة إلى ذلك، يتم احتساب الاستخدام ضمن حدود الاستخدام اليومية للمستخدم (إذا تم تفعيل `usage_limits`).

**البراميترات** (جسم الطلب JSON):

| البراميتر | النوع | إلزامي | الوصف |
|-----------|-------|--------|-------|
| `barcode` | string | نعم | قيمة الباركود الممسوح ضوئياً. |

**مثال**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/use" \
  -H "Content-Type: application/json" \
  -d '{"barcode": "BC000101"}'
```

**استجابة ناجحة** (تم الاستخدام لأول مرة):

```json
{
  "code": 200,
  "status": true,
  "message": "تم استخدام الباركود بنجاح.",
  "data": {
    "id": 101,
    "code": "BC000101",
    "barcode": "BC000101",
    "is_use": true,
    "used_count": 1,
    "used_at": "2025-03-15T14:30:00+03:00",
    "last_used_at": "2025-03-15T14:30:00+03:00"
  }
}
```

**استجابة فاشلة (تجاوز الحد اليومي للمستخدم)**:

```json
{
  "code": 400,
  "status": false,
  "message": "حدث خطأ أثناء استخدام الباركود.",
  "error": "لقد تجاوزت الحد المسموح لاستخدام الباركودات اليوم (10)."
}
```

---

### 3.9. التحقق الإداري من باركود (تعيين is_verified)

#### **`POST /barcodes/verify/{id}`**

**المقدمة**:  
نقطة خاصة بالمشرفين (backend) لتوثيق الباركود (تعيين `is_verified = true`). يمكن تحديد درجة التحقق `verification_score` (0-100). يستخدم هذا عادةً بعد فحص يدوي للوثائق المرتبطة.

**البراميترات** (جسم الطلب JSON اختياري):

| البراميتر | النوع | إلزامي | الوصف |
|-----------|-------|--------|-------|
| `score` | int | لا | درجة التحقق (0-100، افتراضي 100). |

**مثال**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/verify/101" \
  -H "Content-Type: application/json" \
  -d '{"score": 95}'
```

**استجابة ناجحة**:

```json
{
  "code": 200,
  "status": true,
  "message": "تم التحقق من الباركود بنجاح.",
  "data": {
    "id": 101,
    "is_verified": true,
    "verification_score": 95,
    "verified_at": "2025-03-15T15:00:00+03:00"
  }
}
```

---

### 3.10. توليد دفعة من الباركودات

#### **`POST /barcodes/generate`**

**المقدمة**:  
توليد عدة باركودات دفعة واحدة (حتى 1000). هذه النقطة متاحة فقط لمستخدمي لوحة التحكم ذوي صلاحية `generate`. تحدد عدد الباركودات، البادئة، والنوع، وتُنتج باركودات متسلسلة (مثل BC000001, BC000002، …).

**البراميترات** (جسم الطلب JSON):

| البراميتر | النوع | إلزامي | الوصف |
|-----------|-------|--------|-------|
| `count` | int | لا | عدد الباركودات المطلوبة (1-1000، افتراضي 10). |
| `prefix` | string | لا | بادئة الكود (افتراضي `BC`). |
| `barcode_type` | string | لا | نوع الباركود (افتراضي `C128`). |
| `expiry_days` | int | لا | عدد أيام الصلاحية (0 = لا تنتهي، افتراضي 365). |
| `product_id` | int | لا | ربط بمنتج معين. |
| `price` | float | لا | سعر موحد للباركودات. |
| `companys_id` | int | لا | معرف الشركة. |
| `departments_id` | int | لا | معرف القسم. |

**مثال**:

```bash
curl -X POST "https://yourdomain.com/api/v1/qrcodes/barcodes/generate" \
  -H "Content-Type: application/json" \
  -d '{"count": 5, "prefix": "SALE", "barcode_type": "QR", "price": 29.99}'
```

**استجابة ناجحة**:

```json
{
  "code": 200,
  "status": true,
  "message": "تم إنشاء 5 باركود بنجاح.",
  "data": {
    "success_count": 5,
    "failed_count": 0,
    "success": [
      { "id": 111, "code": "SALE000001", "barcode": "SALE000001" },
      { "id": 112, "code": "SALE000002", "barcode": "SALE000002" }
    ],
    "failed": []
  }
}
```

---

### 3.11. إحصائيات الباركودات العامة

#### **`GET /barcodes/stats`**

**المقدمة**:  
إرجاع إحصائيات حول الباركودات بناءً على صلاحيات المستخدم. بالنسبة لمستخدمي frontend، يتم تصفية الإحصائيات لتشمل فقط الباركودات المرتبطة بهم (عبر `user_id`). لمستخدمي backend مع صلاحية `access_all`، تشمل الإحصائيات كل الباركودات، وإلا تقتصر على ما أنشأه هو.

**البراميترات**: لا توجد (تؤخذ الفلاتر من المستخدم).

**مثال**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/stats"
```

**استجابة ناجحة**:

```json
{
  "code": 200,
  "status": true,
  "message": "تم جلب إحصائيات الباركودات بنجاح.",
  "data": {
    "total": 152,
    "active": 120,
    "inactive": 32,
    "published": 100,
    "verified": 80,
    "used": 45,
    "expired": 12,
    "expiring_soon": 5,
    "by_type": { "C128": 70, "QR": 50, "EAN13": 32 },
    "by_status": { "active": 120, "used": 45, "expired": 12 }
  }
}
```

---

### 3.12. إحصائيات استخدام المستخدم الحالي

#### **`GET /barcodes/user-stats`**

**المقدمة**:  
تُظهر إحصائيات خاصة بالمستخدم المُصادق (frontend أو backend) حول عدد الباركودات التي استخدمها، والحدود المتبقية اليومية، وآخر استخدام. تعتمد على الكاش لتحديد الحدود اليومية إن تم تفعيلها.

**البراميترات**: لا توجد.

**مثال**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/user-stats"
```

**استجابة ناجحة**:

```json
{
  "code": 200,
  "status": true,
  "message": "User statistics retrieved successfully",
  "data": {
    "total_used": 8,
    "today_used": 2,
    "this_week_used": 6,
    "this_month_used": 8,
    "last_used_at": "2025-03-15T13:20:00+03:00",
    "today_limit": 10,
    "today_remaining": 8
  }
}
```

---

### 3.13. الحصول على صورة الباركود

#### **`GET /barcodes/image/{id}`**

**المقدمة**:  
تُعيد صورة الباركود بصيغة PNG مباشرة (Content-Type: image/png). إذا كانت الصورة مخزنة مسبقاً في `image_path` تُعاد منها، وإلا تُولد في الطلب باستخدام `BarcodeGenerator`. تطبق نفس صلاحيات العرض (show) للتأكد من أن المستخدم مخوّل برؤية هذا الباركود.

**البراميترات**: لا توجد.

**مثال**:

```bash
curl -X GET "https://yourdomain.com/api/v1/qrcodes/barcodes/image/101" --output barcode.png
```

**استجابة ناجحة**: ملف صورة PNG (يتم عرضه أو تحميله).

**استجابة فاشلة (404)**:

```json
{
  "code": 400,
  "status": false,
  "message": "Unable to generate barcode image.",
  "error": "Unable to generate barcode image."
}
```

---

## 4. أفضل الممارسات والنصائح للمطورين

1. **استخدام التخزين المؤقت (Cache)**  
   - فعّل `enable_cache` في `config.php` لتحسين أداء نقاط `index` و `stats`.  
   - استخدم نقطة `activelystats` لتحديث الكاش بشكل ذكي (مقارنة `last_updated` مع الطابع المخزن محلياً).

2. **إدارة حدود الاستخدام**  
   - اضبط `usage_limits` في ملف الإعدادات حسب احتياجات عملك. يوصى باستخدام `tracking_type = database` للدقة في حال أهمية سجل الاستخدام الفعلي.  
   - وفر واجهة أمامية تعرض للمستخدم المتبقي من حده اليومي (عبر `userStats`).

3. **التحقق من صحة البيانات**  
   - استخدم `validateBarcode` قبل محاولة إنشاء أو تحديث باركود للتأكد من أن القيمة تتوافق مع النوع المحدد (مثل EAN-13).  
   - تحقق من وجود `barcode` و `barcode_type` في طلب `store`، إذ هما الحقلان الوحيدان الإلزاميان من جانب API (توليد `code` تلقائي).

4. **إدارة الصلاحيات**  
   - راجع ملف `config.php` لفهم العمليات المتاحة لكل نوع مستخدم (backend/frontend/guest).  
   - يُنصح بمنح صلاحية `use` للمستخدمين frontend فقط، بينما `generate` و `verify` للمشرفين.

5. **التعامل مع الأخطاء**  
   - جميع الاستجابات تتبع هيكلاً موحداً. افحص `status` و `code` لتحديد نجاح العملية.  
   - استفد من حقل `errors` لمعرفة تفاصيل أخطاء التحقق من صحة البيانات (عند `ValidationException`).

6. **تحميل الصور**  
   - للحصول على صورة الباركود بدقة وجودة عالية، يمكن استدعاء نقطة `image` مع إمكانية تمرير معاملات الحجم والتنسيق عبر `BarcodeGenerator` (غير مطبقة حالياً في نقطة `image` ولكن يمكن توسعتها).

7. **الترقيم والتصفية المتقدمة**  
   - استخدم البراميترات `is_or_*`, `is_not_*`, `is_or_null_*` فقط للحقول المسموحة في `advanced_filters` (الموجودة في `config.php` لتلافي إساءة الاستخدام).  
   - كن حذراً عند تمرير `exclude` قد يؤدي إلى إزالة حقول أساسية يحتاجها العميل.

---

## 5. خاتمة

واجهة برمجة التطبيقات `Barcodes` تقدّم حلاً كاملاً لإدارة دورة حياة الباركودات: من الإنشاء والتحديث إلى الاستخدام والإحصائيات. بفضل نظام الصلاحيات المرن ودعم حدود الاستخدام، يمكن دمجها بسهولة في أنظمة الفوترة، تتبع المنتجات، برامج الولاء، والمزيد. الوثائق أعلاه تغطي جميع النقاط الطرفية مع أمثلة عملية ونماذج استجابة واضحة. للتخصيص أو التوسع، يُنصح بالرجوع إلى كود المصدر واستخدام كلاس `Manager` لإجراء عمليات متقدمة.

إذا كانت هناك حاجة لتعديل سلوك أي نقطة طرفية، يمكن تعديل ملف المتحكم `Barcodes.php` أو إعدادات `config.php` (خاصة صلاحيات العمليات وفلاتر `advanced_filters`). فريق التطوير على استعداد لدعم الاستفسارات الإضافية.