## توثيق واجهة برمجة التطبيقات (API) لرفع الملفات – الإصدار 1.2.5

**الإصدار:** 1.2.5  
**المسار الأساسي:** `/api/v1/fileupload`  
**المصادقة:** OAuth 2.0 (توكن الوصول في الهيدر `Authorization: Bearer <token>`)

---

## نظرة عامة

هذه الوثيقة موجهة للمطورين الخارجيين (متاجر ويب، تطبيقات التجارة الإلكترونية، تطبيقات توصيل الطلبات، والأنظمة المتكاملة) الذين يرغبون في دمج خدمة رفع الملفات في تطبيقاتهم. توفر واجهة برمجة التطبيقات (API) نقاط نهاية آمنة وسهلة الاستخدام لرفع ملف واحد أو متعدد، وحذف الملفات، واسترجاع الملفات المرتبطة بنماذج محددة (مثل المنتجات، الطلبات، المستخدمين)، بالإضافة إلى استبدال الملفات في العلاقات الأحادية (`attachOne`) وتشغيل مجموعة الاختبارات المتكاملة.

**الميزات الرئيسية:**
- دعم رفع الملفات بصيغ متعددة (multipart/form-data أو base64).
- رفع مؤقت للملفات قبل حفظ النموذج (باستخدام مفاتيح جلسة مؤقتة).
- **استبدال الملفات في العلاقات الأحادية (`attachOne`)**: عند تمرير `model_id` لحقل من نوع `attachOne` ويوجد ملف مرتبط مسبقاً، يتم استبداله تلقائياً (عملية `edit`).
- تحويل تلقائي للصور (تحجيم، علامة مائية) حسب إعدادات النظام.
- تخزين الملفات على أقراص متعددة (محلي، S3، FTP).
- استجابات موحدة مع أكواد خطأ فريدة لتسهيل المعالجة البرمجية.
- أمان كامل عبر OAuth 2.0 والتحقق من صلاحيات المستخدم.
- **نقطة نهاية لتشغيل الاختبارات** (متاحة فقط في بيئة التطوير).
- **نقاط نهاية للاستعلام عن الموديولات المسجلة وإعداداتها وقيودها** (جديد في الإصدار 1.2.3، تم تحسينها في 1.2.5).
- **نقاط نهاية للتحقق من الصلاحيات** على المستويات العالمية، مستوى الموديول، ومستوى الحقل (جديد في الإصدار 1.2.3، تم تحسينها في 1.2.5).
- **نقطة نهاية للتحقق من صحة المفاتيح المؤقتة** (جديد في الإصدار 1.2.3).

> **ملاحظة حول الإصدار 1.2.5**: تم إعادة هيكلة نقاط نهاية الاستعلام والصلاحيات لاستخدام معاملات الاستعلام (query parameters) بدلاً من معاملات المسار (route parameters) لتجنب مشاكل ترميز أسماء الموديولات التي تحتوي على خطوط مائلة عكسية (`\`). المسارات الجديدة موثقة أدناه. تم إهمال المسارات القديمة (التي تحتوي على `{modelClass}` في المسار) وستتم إزالتها في إصدار مستقبلي.

---

## هيكل الاستجابة الموحد

كل استجابة من واجهة API تأتي بالهيكل التالي:

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "error": null,
    "errors": [],
    "data": {},
    "meta": {},
    "input_data": {},
    "process_data": {},
    "debug": null,
    "error_code": null
}
```

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `code` | `int` | كود HTTP (200، 400، 401، 403، 404، 422، 500، 503). |
| `status` | `bool` | `true` للنجاح، `false` للفشل. |
| `message` | `string` | رسالة واضحة للمستخدم (قابلة للترجمة). |
| `error` | `string\|null` | تفاصيل الخطأ التقنية (في حالة الفشل). |
| `errors` | `array\|null` | مصفوفة بأخطاء التحقق (مثل صحة المدخلات). |
| `data` | `mixed` | البيانات الأساسية (مثل معلومات الملف المرفوع). |
| `meta` | `mixed` | بيانات إضافية (مثل ملخص الاختبارات). |
| `input_data` | `array` | نسخة من البيانات التي أرسلها المستخدم (للتتبع). |
| `process_data` | `array` | معلومات داخلية عن المعالجة (مثل مفتاح الجلسة المؤقت، العملية المنفذة `add`/`edit`). |
| `debug` | `array\|null` | معلومات التصحيح (تظهر فقط عند تفعيل `app.debug`). |
| `error_code` | `string\|null` | كود خطأ فريد (مثل `FILE_UPLOAD_PERMISSION_DENIED`). |

---

## المصادقة والأمان

جميع نقاط النهاية محمية بـ **OAuth 2.0**. يجب تضمين توكن الوصول في هيدر الطلب:

```
Authorization: Bearer <your_access_token>
```

- إذا لم يتم توفير التوكن: استجابة `401 Unauthorized`.
- إذا كان التوكن غير صالح أو منتهي: استجابة `401 Unauthorized`.

**ملاحظة:** تعتمد صلاحيات العمليات (رفع، استبدال، حذف، جلب) على نوع المستخدم (`backend` / `frontend`) والإعدادات المسجلة لكل نموذج وحقل. تأكد من أن المستخدم لديه الصلاحيات المناسبة قبل استدعاء الـ API. عمليات استبدال الملفات تتطلب صلاحية `edit`.

---

## نقاط النهاية (Endpoints)

### 1. رفع ملف واحد

**الطريقة:** `POST`  
**المسار:** `/upload`  
**نوع المحتوى:** `multipart/form-data` أو `application/json` (عند استخدام base64).

#### معاملات الطلب

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج (مثال: `Nano\Shop\Models\Product`). |
| `field` | `string` | نعم | اسم الحقل كما هو مسجل في النظام (مثال: `image`, `gallery`). |
| `file` | `file` | شرطي* | الملف المرفوع (بصيغة multipart). |
| `file_base64` | `string` | شرطي* | بيانات الملف بصيغة base64 (بديل عن `file`). |
| `temp_session_key` | `string` | لا | مفتاح جلسة مؤقت لربط الملف لاحقاً (للنماذج غير المحفوظة). |
| `model_id` | `int` | لا | معرف النموذج المحفوظ. إذا كان الحقل من نوع `attachOne` وكان النموذج يمتلك ملفاً مرتبطاً مسبقاً، **سيتم استبدال الملف القديم بالجديد** (عملية `edit`). |

> * يجب توفير إما `file` أو `file_base64`.

#### أمثلة على الطلبات

**طلب باستخدام multipart/form-data (رفع ملف مباشر):**

```http
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="model_class"

Nano\Shop\Models\Product
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

image
------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="product.jpg"
Content-Type: image/jpeg

(بيانات الملف الثنائية)
------WebKitFormBoundary--
```

**طلب باستخدام JSON و base64 (للواجهات التي لا تدعم multipart):**

```json
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
    "model_id": 10
}
```

**طلب لرفع ملف مؤقت (قبل حفظ النموذج):**

```json
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,..."
}
```

**طلب لاستبدال صورة منتج موجود (عملية `edit`):**

```json
POST /api/v1/fileupload/upload HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,...",
    "model_id": 10
}
```

#### الاستجابات

**✅ استجابة نجاح (رفع وربط مباشر بنموذج موجود – عملية `add`):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع الملف بنجاح",
    "data": {
        "id": 123,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "process_data": {
        "operation": "add",
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "user_type": "backend"
    }
}
```

**✅ استجابة نجاح (استبدال ملف موجود – عملية `edit`):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع الملف بنجاح",
    "data": {
        "id": 124,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../new.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "process_data": {
        "operation": "edit",
        "existing_file_deleted": 123,
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "user_type": "backend"
    }
}
```

**✅ استجابة نجاح (رفع مؤقت – سيتم ربطه لاحقاً):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع الملف بنجاح",
    "data": {
        "id": 125,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg"
    },
    "process_data": {
        "temp_session_key": "tmp_YWJjMTIzOnNvbWVoYXNo",
        "operation": "add"
    }
}
```

> **هام:** في الرفع المؤقت، يجب حفظ `temp_session_key` في جلسة المستخدم أو قاعدة بيانات لاستخدامه لاحقاً عند ربط الملف بالنموذج المحفوظ.

**❌ استجابة خطأ (صلاحية ممنوعة – أو تعطيل التعديل عالمياً):**

```json
{
    "code": 403,
    "status": false,
    "message": "لا تملك صلاحية رفع الملفات لهذا الحقل",
    "error": "لا تملك صلاحية رفع الملفات لهذا الحقل",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

**❌ استجابة خطأ (نوع ملف غير مسموح به):**

```json
{
    "code": 422,
    "status": false,
    "message": "نوع الملف غير مسموح به. الأنواع المسموحة: jpg,jpeg,png. الامتداد المرفوع: exe.",
    "error": "نوع الملف غير مسموح به. الأنواع المسموحة: jpg,jpeg,png. الامتداد المرفوع: exe.",
    "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
}
```

**❌ استجابة خطأ (حجم ملف كبير جداً):**

```json
{
    "code": 422,
    "status": false,
    "message": "حجم الملف يتجاوز الحد المسموح به (2048 KB). الحجم الفعلي: 5120 KB.",
    "error": "حجم الملف يتجاوز الحد المسموح به (2048 KB). الحجم الفعلي: 5120 KB.",
    "error_code": "FILE_UPLOAD_FILE_SIZE_EXCEEDED"
}
```

**❌ استجابة خطأ (ملف خطير – قائمة سوداء):**

```json
{
    "code": 422,
    "status": false,
    "message": "نوع الملف php غير مسموح به لأسباب أمنية.",
    "error": "نوع الملف php غير مسموح به لأسباب أمنية.",
    "error_code": "FILE_UPLOAD_FILE_TYPE_BLACKLISTED"
}
```

---

### 2. رفع عدة ملفات (لحقل متعدد)

**الطريقة:** `POST`  
**المسار:** `/upload-multiple`

#### معاملات الطلب

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل المتعدد (مثل `gallery`). |
| `files` | `array` | نعم | مصفوفة من الملفات (كل عنصر إما كائن `UploadedFile` أو سلسلة base64). |
| `temp_session_key` | `string` | لا | مفتاح جلسة مؤقت (للنماذج غير المحفوظة). |
| `model_id` | `int` | لا | معرف النموذج المحفوظ. |

#### مثال طلب (multipart)

```http
POST /api/v1/fileupload/upload-multiple HTTP/1.1
Authorization: Bearer ...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="model_class"

Nano\Shop\Models\Product
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

gallery
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="img1.jpg"
Content-Type: image/jpeg

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="img2.png"
Content-Type: image/png

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="img3.pdf"
Content-Type: application/pdf

...
------WebKitFormBoundary--
```

#### الاستجابات

**✅ استجابة نجاح (جميع الملفات رفعت بنجاح):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع 3 من 3 ملف بنجاح",
    "data": [
        {
            "status": true,
            "data": { "id": 101, "path": "...", "thumb": "..." }
        },
        {
            "status": true,
            "data": { "id": 102, "path": "...", "thumb": "..." }
        },
        {
            "status": true,
            "data": { "id": 103, "path": "...", "thumb": "..." }
        }
    ],
    "process_data": {
        "success_count": 3,
        "total": 3
    }
}
```

**⚠️ استجابة نجاح جزئي (بعض الملفات فشلت):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع 2 من 3 ملف بنجاح",
    "data": [
        {
            "status": true,
            "data": { "id": 101, "path": "..." }
        },
        {
            "status": true,
            "data": { "id": 102, "path": "..." }
        },
        {
            "status": false,
            "error": "نوع الملف غير مسموح به",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        }
    ],
    "process_data": {
        "success_count": 2,
        "total": 3
    }
}
```

**❌ استجابة فشل كامل (جميع الملفات فشلت):**

```json
{
    "code": 400,
    "status": false,
    "message": "فشل رفع جميع الملفات",
    "error": "نوع الملف غير مسموح به, حجم الملف كبير جداً",
    "data": [
        {
            "status": false,
            "error": "نوع الملف غير مسموح به",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        },
        {
            "status": false,
            "error": "حجم الملف يتجاوز الحد المسموح به",
            "error_code": "FILE_UPLOAD_FILE_SIZE_EXCEEDED"
        }
    ],
    "process_data": {
        "success_count": 0,
        "total": 2
    }
}
```

---

### 3. حذف ملف

**الطريقة:** `DELETE`  
**المسار:** `/delete/{id}`

#### معاملات المسار

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `id` | `int` | نعم | معرف الملف المراد حذفه. |

#### معاملات الاستعلام (Query Parameters)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | لا | اسم النموذج (للتحقق من صلاحية الحذف). |
| `field` | `string` | لا | اسم الحقل (للتحقق من صلاحية الحذف). |

#### مثال طلب

```http
DELETE /api/v1/fileupload/delete/123?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابات

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم حذف الملف",
    "data": null
}
```

**❌ استجابة خطأ (الملف غير موجود):**

```json
{
    "code": 404,
    "status": false,
    "message": "الملف غير موجود",
    "error": "الملف غير موجود",
    "error_code": "FILE_UPLOAD_FILE_NOT_FOUND"
}
```

**❌ استجابة خطأ (صلاحية ممنوعة – لا يملك المستخدم صلاحية الحذف):**

```json
{
    "code": 403,
    "status": false,
    "message": "لا تملك صلاحية حذف هذا الملف",
    "error": "لا تملك صلاحية حذف هذا الملف",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

**❌ استجابة خطأ (عمليات الحذف معطلة عالمياً):**

```json
{
    "code": 503,
    "status": false,
    "message": "عمليات حذف الملفات معطلة حالياً",
    "error": "عمليات حذف الملفات معطلة حالياً",
    "error_code": "FILE_UPLOAD_DELETE_DISABLED_GLOBALLY"
}
```

---

### 4. جلب الملفات المرتبطة

**الطريقة:** `GET`  
**المسار:** `/files`

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل. |
| `model_id` | `int` | لا | معرف النموذج المحفوظ (لجلب الملفات المرتبطة). |
| `temp_session_key` | `string` | لا | مفتاح الجلسة المؤقت (لجلب الملفات المؤقتة غير المرتبطة). |
| `with_thumbs` | `bool` | لا | إضافة صور مصغرة (`true` / `false`، افتراضي `false`). |
| `thumb_sizes` | `object` | لا | أحجام الصور المصغرة (مصفوفة بأسماء الأحجام وأبعادها). |

#### أمثلة الطلبات

**جلب صور معرض منتج (مرتبطة بنموذج محفوظ):**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10 HTTP/1.1
Authorization: Bearer ...
```

**جلب ملفات مؤقتة (غير مرتبطة) باستخدام مفتاح جلسة:**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=image&temp_session_key=tmp_YWJjMTIzOnNvbWVoYXNo HTTP/1.1
Authorization: Bearer ...
```

**جلب صور معرض مع صور مصغرة بأحجام مخصصة:**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10&with_thumbs=true&thumb_sizes[small][0]=150&thumb_sizes[small][1]=150&thumb_sizes[small][2]=crop&thumb_sizes[medium][0]=300&thumb_sizes[medium][1]=300 HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابات

**✅ استجابة نجاح (ملفات مرتبطة):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب الملفات",
    "data": [
        {
            "id": 101,
            "title": null,
            "description": null,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
            "size": 102400,
            "content_type": "image/jpeg"
        },
        {
            "id": 102,
            "title": null,
            "description": null,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../original.png",
            "size": 204800,
            "content_type": "image/png"
        }
    ]
}
```

**✅ استجابة نجاح (مع صور مصغرة):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب الملفات",
    "data": [
        {
            "id": 101,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
            "small": "https://yourdomain.com/storage/app/uploads/public/.../small.jpg",
            "medium": "https://yourdomain.com/storage/app/uploads/public/.../medium.jpg"
        }
    ]
}
```

**✅ استجابة نجاح (ملفات مؤقتة – بعد التحقق الأمني):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب الملفات",
    "data": [
        {
            "id": 200,
            "path": "https://yourdomain.com/storage/app/uploads/public/.../temp.jpg"
        }
    ]
}
```

**❌ استجابة خطأ (مفتاح مؤقت غير صالح أو منتهي):**

```json
{
    "code": 400,
    "status": false,
    "message": "مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية",
    "error": "مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية",
    "error_code": "FILE_UPLOAD_TEMP_KEY_INVALID"
}
```

**❌ استجابة خطأ (مفتاح مؤقت لا يخص المستخدم الحالي):**

```json
{
    "code": 403,
    "status": false,
    "message": "المفتاح المؤقت غير مخصص لهذا النموذج/الحقل",
    "error": "المفتاح المؤقت غير مخصص لهذا النموذج/الحقل",
    "error_code": "FILE_UPLOAD_TEMP_KEY_MISMATCH"
}
```

---

### 5. تشغيل اختبارات الإضافة (للبيئة المحلية فقط)

**الطريقة:** `GET`  
**المسار:** `/tests`

هذه النقطة مخصصة للمطورين لاختبار صحة تثبيت الإضافة ووظائفها الأساسية. **لا يمكن استخدامها في بيئة الإنتاج** (تتطلب `app.debug = true` أو بيئة `local`).

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `test_version` | `string` | لا | إصدار الاختبارات (`v1` أو `v2`، الافتراضي `v2`). الإصدار `v2` هو الأحدث ويوفر مخرجات موحدة. |
| `filter` | `string` | لا | تصفية النتائج (`all`, `passed`, `failed`، الافتراضي `all`). |

#### مثال طلب

```http
GET /api/v1/fileupload/tests?test_version=v2&filter=passed HTTP/1.1
Authorization: Bearer <token>
```

#### الاستجابة

تعيد مصفوفة بنتائج جميع الاختبارات (أو المصفاة) مع ملخص (`total`, `passed`, `failed`, `success_rate`). كل اختبار يحتوي على `test_code`, `name`, `description`, `status`, `message`, `data`, `error`, `debug` إلخ.

**✅ استجابة نجاح (مثال مختصر):**

```json
{
    "code": 200,
    "status": true,
    "message": "Test suite completed",
    "data": [
        {
            "test_code": "REGISTER_MODEL_001",
            "name": "تسجيل نموذج وحقول في FileUploadRegistry",
            "status": true,
            "message": "تم تسجيل النموذج والحقل بنجاح"
        }
    ],
    "meta": {
        "summary": {
            "total": 43,
            "passed": 43,
            "failed": 0,
            "success_rate": 100,
            "test_version": "v2",
            "filter_applied": "passed"
        }
    }
}
```

> **تنبيه:** لا تستخدم هذه النقطة في الإنتاج أبداً، لأنها قد تكشف معلومات داخلية عن النظام.

---

### 6. جلب جميع الموديولات المسجلة (بدون تغيير)

**الطريقة:** `GET`  
**المسار:** `/models`

**الوصف:** يعيد قائمة الموديولات المسجلة في النظام التي يملك المستخدم الحالي صلاحية `view` عليها، مع معلومات مختصرة عن حقولها (الاسم، النوع، هل هو متعدد). هذه النقطة مفيدة لتطبيقات الواجهة الأمامية التي تحتاج إلى بناء نماذج رفع ديناميكية بناءً على الإعدادات الفعلية.

#### معاملات الاستعلام

لا توجد معاملات مطلوبة.

#### مثال طلب

```http
GET /api/v1/fileupload/models HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة (نفس السابق)

---

### 7. جلب إعدادات موديول معين (تم التحديث في 1.2.5)

**الطريقة:** `GET`  
**المسار:** `/model/config`

**الوصف:** يعيد الإعدادات الكاملة لموديول معين (بعد التحقق من وجوده وصلاحية `view`). تشمل الإعدادات الحقول المسجلة مع بعض التفاصيل (النوع، الحجم الأقصى، الأنواع المسموحة، إلخ).

#### معاملات الاستعلام (Query Parameters)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج (مثال: `Nano\Shop\Models\Product`). **لا حاجة لترميزه**، يمرر كقيمة عادية. |

#### مثال طلب

```http
GET /api/v1/fileupload/model/config?model_class=Nano\Shop\Models\Product HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "enabled": true,
        "label": "Product",
        "allowed_user_types": ["backend", "frontend"],
        "disabled_operations": [],
        "fields": {
            "image": {
                "type": "image",
                "label": "image",
                "multiple": false,
                "required": false,
                "max_filesize": 2048,
                "allowed_types": "jpg,jpeg,png",
                "use_caption": true,
                "disabled_operations": []
            },
            "gallery": {
                "type": "multiple",
                "label": "gallery",
                "multiple": true,
                "required": false,
                "max_filesize": 2048,
                "allowed_types": "jpg,jpeg,png",
                "use_caption": true,
                "disabled_operations": []
            }
        }
    }
}
```

**❌ استجابة خطأ (موديول غير مسجل):**

```json
{
    "code": 404,
    "status": false,
    "message": "النموذج غير مسجل في النظام",
    "error_code": "FILE_UPLOAD_MODEL_NOT_REGISTERED"
}
```

**❌ استجابة خطأ (لا توجد صلاحية view):**

```json
{
    "code": 403,
    "status": false,
    "message": "لا تملك صلاحية رفع الملفات لهذا الحقل",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

---

### 8. جلب إعدادات حقل معين (تم التحديث في 1.2.5)

**الطريقة:** `GET`  
**المسار:** `/field/config`

**الوصف:** يعيد إعدادات حقل معين (مثل النوع، الحجم الأقصى، الأنواع المسموحة). يستخدم هذا للتحقق من إعدادات الحقل قبل محاولة الرفع.

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل. |

#### مثال طلب

```http
GET /api/v1/fileupload/field/config?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "type": "image",
        "label": "image",
        "multiple": false,
        "required": false,
        "max_filesize": 2048,
        "allowed_types": "jpg,jpeg,png",
        "use_caption": true,
        "disabled_operations": []
    }
}
```

**❌ استجابة خطأ (حقل غير مسجل):**

```json
{
    "code": 404,
    "status": false,
    "message": "الحقل 'video' غير مسجل في النموذج 'Nano\\Shop\\Models\\Product'",
    "error_code": "FILE_UPLOAD_FIELD_NOT_REGISTERED"
}
```

---

### 9. جلب قيود الحقل (تم التحديث في 1.2.5)

**الطريقة:** `GET`  
**المسار:** `/field/constraints`

**الوصف:** يعيد قيود الحقل المستخدمة في عملية التحقق من صحة الملف (`max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`). يمكن للواجهات الأمامية تطبيق نفس القيود قبل رفع الملف لتجنب أخطاء الخادم.

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل. |

#### مثال طلب

```http
GET /api/v1/fileupload/field/constraints?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "max_filesize": 2048,
        "allowed_types": "jpg,jpeg,png",
        "multiple": false,
        "max_files": null,
        "required": false,
        "is_public": true,
        "use_caption": true,
        "thumb_options": {
            "mode": "crop",
            "extension": "auto"
        },
        "type": "image"
    }
}
```

---

### 10. جلب خيارات المعالجة المتقدمة لحقل معين (تم التحديث في 1.2.5)

**الطريقة:** `GET`  
**المسار:** `/processing-options`

**الوصف:** يعيد خيارات المعالجة المتقدمة الخاصة بالحقل: قرص التخزين (`storage_disk`)، التحجيم التلقائي (`auto_resize` مع `resize_options`)، والعلامة المائية (`auto_watermark` مع `watermark_options`). هذه المعلومات مفيدة للواجهات المتقدمة التي تحتاج إلى معرفة كيفية معالجة الملفات بعد الرفع.

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل. |

#### مثال طلب

```http
GET /api/v1/fileupload/processing-options?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "storage_disk": "s3",
        "auto_resize": true,
        "resize_options": {
            "width": 1920,
            "height": 1080,
            "mode": "auto"
        },
        "auto_watermark": true,
        "watermark_options": {
            "position": "bottom-right",
            "resize_percentage": 15
        }
    }
}
```

**❌ استجابة خطأ (حقل غير مسجل):** (نفس هيكل الخطأ السابق)

---

### 11. التحقق من الصلاحيات

#### 11.1 التحقق من تفعيل عملية معينة عالمياً (بدون تغيير)

**الطريقة:** `GET`  
**المسار:** `/permissions/global/{operation}`

**معاملات المسار:**

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `operation` | `string` | نعم | العملية: `add`, `edit`, `delete`, `view`. |

**مثال طلب:**

```http
GET /api/v1/fileupload/permissions/global/edit HTTP/1.1
Authorization: Bearer ...
```

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "operation": "edit",
        "allowed": true,
        "disabled_globally": false
    }
}
```

---

#### 11.2 التحقق من صلاحية عملية على مستوى موديول معين (تم التحديث في 1.2.5)

**الطريقة:** `GET`  
**المسار:** `/permissions/model`

**الوصف:** يتحقق مما إذا كانت عملية معينة مفعلة على مستوى موديول معين (مع مراعاة `disabled_operations` على مستوى النموذج).

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `operation` | `string` | نعم | العملية: `add`, `edit`, `delete`, `view`. |

#### مثال طلب

```http
GET /api/v1/fileupload/permissions/model?model_class=Nano\Shop\Models\Product&operation=edit HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "operation": "edit",
        "model_operation_enabled": true,
        "user_type_allowed": true,
        "can_proceed": true
    }
}
```

---

#### 11.3 التحقق من صلاحية عملية على مستوى حقل معين (تم التحديث في 1.2.5)

**الطريقة:** `GET`  
**المسار:** `/permissions/field`

**الوصف:** يتحقق مما إذا كانت عملية معينة مفعلة على مستوى حقل معين (مع مراعاة `disabled_operations` على مستوى الحقل وصلاحيات المستخدم).

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل. |
| `operation` | `string` | نعم | العملية: `add`, `edit`, `delete`, `view`. |

#### مثال طلب

```http
GET /api/v1/fileupload/permissions/field?model_class=Nano\Shop\Models\Product&field=image&operation=edit HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "operation": "edit",
        "field_operation_enabled": true,
        "user_has_full_permission": true,
        "can_proceed": true
    }
}
```

> **ملاحظة:** `user_has_full_permission` يشمل التحقق من نوع المستخدم، الصلاحيات المحددة، والإعدادات العامة والموديول والحقل.

---

#### 11.4 التحقق المتكامل من الصلاحية (باستخدام validate) – بدون تغيير

**الطريقة:** `POST`  
**المسار:** `/permissions/check`

**الوصف:** يتحقق من الصلاحية بشكل متكامل (العالمية، الموديول، الحقل، نوع المستخدم، الصلاحيات المحددة) ويعيد النتيجة دون الحاجة لمحاولة رفع ملف فعلي.

#### معاملات الطلب (JSON)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `operation` | `string` | نعم | العملية: `add`, `edit`, `delete`, `view`. |
| `field` | `string` | لا | اسم الحقل (اختياري، إذا كان التحقق على مستوى حقل). |

#### مثال طلب

```json
POST /api/v1/fileupload/permissions/check HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "model_class": "Nano\\Shop\\Models\\Product",
    "operation": "edit",
    "field": "image"
}
```

#### الاستجابات

**✅ استجابة نجاح (مسموح):**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "allowed": true,
        "model_class": "Nano\\Shop\\Models\\Product",
        "operation": "edit",
        "field": "image",
        "message": "Permission granted"
    }
}
```

**❌ استجابة نجاح (غير مسموح – مع رسالة الخطأ):**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت العملية بنجاح",
    "data": {
        "allowed": false,
        "model_class": "Nano\\Shop\\Models\\Product",
        "operation": "edit",
        "field": "image",
        "message": "لا تملك صلاحية رفع الملفات لهذا الحقل"
    }
}
```

> **ملاحظة:** في حالة عدم السماح، يبقى `code` = 200 و `status` = true لأن الطلب نفسه نجح في إجراء التحقق. الخطأ يُبلغ في `data.message`.

---

### 12. التحقق من صحة مفتاح مؤقت ومطابقته (بدون تغيير)

**الطريقة:** `POST`  
**المسار:** `/temp-key/validate`

**الوصف:** يتحقق من صحة مفتاح الجلسة المؤقت (صيغة، توقيع، صلاحية) ومطابقته مع النموذج والحقل والمستخدم المقدمين. هذه النقطة مفيدة للتطبيقات التي تحتاج إلى التأكد من أن المفتاح المؤقت الذي تحتفظ به لا يزال صالحاً قبل استخدامه في عمليات الربط أو الرفع.

#### معاملات الطلب (JSON)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `temp_key` | `string` | نعم | مفتاح الجلسة المؤقت (يبدأ بـ `tmp_`). |
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج المتوقع. |
| `field` | `string` | نعم | اسم الحقل المتوقع. |
| `strict_mode` | `bool` | لا | وضع صارم (يتطلب تطابق تام للنموذج والحقل). افتراضي `true`. |
| `allow_expired_key` | `bool` | لا | السماح بالمفاتيح منتهية الصلاحية ضمن فترة سماح. افتراضي `false`. |
| `expiry_grace_period` | `int` | لا | فترة السماح بالثواني (عند `allow_expired_key=true`). افتراضي `300` (5 دقائق). |

#### مثال طلب

```json
POST /api/v1/fileupload/temp-key/validate HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "temp_key": "tmp_YWJjMTIzOnNvbWVoYXNo",
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "strict_mode": true,
    "allow_expired_key": false
}
```

#### الاستجابات

**✅ استجابة نجاح (مفتاح صالح):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم التحقق من صحة المفتاح المؤقت",
    "data": {
        "valid": true,
        "key_data": {
            "modelClass": "Nano\\Shop\\Models\\Product",
            "field": "image",
            "userId": "123",
            "userType": "backend",
            "timestamp": 1744123456
        },
        "matched_data": {
            "model_matched": true,
            "field_matched": true,
            "user_matched": true,
            "user_id_matched": true,
            "user_type_matched": true
        }
    },
    "process_data": {
        "validated": true,
        "model_matched": true,
        "field_matched": true,
        "user_matched": true
    }
}
```

**❌ استجابة خطأ (مفتاح غير صالح):**

```json
{
    "code": 400,
    "status": false,
    "message": "مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية",
    "error_code": "FILE_UPLOAD_TEMP_KEY_INVALID"
}
```

**❌ استجابة خطأ (مفتاح لا يطابق النموذج):**

```json
{
    "code": 403,
    "status": false,
    "message": "المفتاح المؤقت غير مخصص لهذا النموذج/الحقل",
    "error_code": "FILE_UPLOAD_TEMP_KEY_MISMATCH"
}
```

---

## أكواد الأخطاء (error_code) – قائمة كاملة

| error_code | كود HTTP | المعنى |
|------------|----------|--------|
| `FILE_UPLOAD_GENERAL` | 500 | خطأ عام غير متوقع. |
| `FILE_UPLOAD_PERMISSION_DENIED` | 403 | المستخدم لا يملك صلاحية للعملية (أو `disable_edit` مفعّل). |
| `FILE_UPLOAD_UNAUTHORIZED` | 401 | غير مصرح به (لم يتم المصادقة). |
| `FILE_UPLOAD_MODEL_NOT_REGISTERED` | 400 | النموذج غير مسجل في النظام. |
| `FILE_UPLOAD_FIELD_NOT_REGISTERED` | 400 | الحقل غير مسجل للنموذج. |
| `FILE_UPLOAD_MODEL_NOT_FOUND` | 404 | النموذج غير موجود (في حالة `model_id`). |
| `FILE_UPLOAD_FILE_NOT_FOUND` | 404 | الملف غير موجود. |
| `FILE_UPLOAD_INVALID_FILE_DATA` | 400 | بيانات الملف غير صالحة (لا ملف ولا base64). |
| `FILE_UPLOAD_FILE_SIZE_EXCEEDED` | 422 | حجم الملف يتجاوز الحد المسموح. |
| `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` | 422 | امتداد الملف غير مسموح به. |
| `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` | 422 | امتداد الملف خطير (PHP, JS, HTML, إلخ). |
| `FILE_UPLOAD_FILE_UPLOAD_FAILED` | 500 | فشل رفع الملف (خطأ داخلي). |
| `FILE_UPLOAD_MAX_FILES_EXCEEDED` | 422 | عدد الملفات يتجاوز الحد المسموح (للحقول المتعددة). |
| `FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY` | 503 | عمليات الرفع معطلة عالمياً. |
| `FILE_UPLOAD_DELETE_DISABLED_GLOBALLY` | 503 | عمليات الحذف معطلة عالمياً. |
| `FILE_UPLOAD_GET_DISABLED_GLOBALLY` | 503 | عمليات الجلب معطلة عالمياً. |
| `FILE_UPLOAD_USER_TYPE_NOT_ALLOWED` | 403 | نوع المستخدم غير مسموح لهذه العملية. |
| `FILE_UPLOAD_TEMP_KEY_INVALID` | 400 | مفتاح مؤقت غير صالح (تنسيق أو توقيع). |
| `FILE_UPLOAD_TEMP_KEY_EXPIRED` | 400 | مفتاح مؤقت منتهي الصلاحية. |
| `FILE_UPLOAD_TEMP_KEY_MISMATCH` | 403 | مفتاح مؤقت لا يخص المستخدم أو النموذج الحالي. |

---

## أفضل الممارسات للتكامل

1. **استخدم `temp_session_key` للنماذج غير المحفوظة**  
   عند إنشاء كيان جديد (مثل منتج، طلب)، قم برفع الملفات أولاً باستخدام `temp_session_key`، ثم احفظ الكيان، وأخيراً اربط الملفات باستدعاء دالة `attachTempFiles` من جانب الخادم (أو عبر نقطة نهاية مخصصة).

2. **لاستبدال ملف موجود** (عملية `edit`): استخدم `model_id` مع الحقل من نوع `attachOne`. سيتم حذف الملف القديم تلقائياً بعد رفع الجديد، مع ضمان سلامة البيانات عبر المعاملات. تأكد من أن المستخدم لديه صلاحية `edit`.

3. **تعامل مع `error_code` برمجياً**  
   بدلاً من الاعتماد على رسائل نصية، استخدم `error_code` لتحديد نوع الخطأ وعرض رسائل مناسبة للمستخدم.

4. **استخدم نقاط نهاية الاستعلام (`/models`, `/field/config`, `/field/constraints`) لبناء واجهات ديناميكية**  
   يمكنك جلب قيود الحقول وعرضها للمستخدم (مثال: عرض الحد الأقصى للحجم، الأنواع المسموحة) قبل أن يختار الملف.

5. **تحقق من الصلاحيات مسبقاً باستخدام `/permissions/check`**  
   لتجنب محاولات الرفض، يمكنك التحقق من صلاحية المستخدم للعملية قبل عرض نموذج رفع الملف.

6. **تحقق من صحة المفتاح المؤقت قبل استخدامه**  
   استخدم نقطة `/temp-key/validate` للتأكد من أن المفتاح الذي تخزنه في الجلسة لا يزال صالحاً (لم تنته صلاحيته، ولا يخص مستخدم آخر).

7. **استخدم `with_thumbs` فقط عند الحاجة**  
   طلب الصور المصغرة يستهلك وقتاً إضافياً لإنشاءها (إذا لم تكن موجودة). استخدمها فقط في شاشات العرض التي تحتاجها.

8. **فعّل `app.debug` في بيئة التطوير**  
   عند مواجهة أخطاء 500، قم بتفعيل وضع التصحيح للحصول على تفاصيل كاملة في حقل `debug` تساعدك في تشخيص المشكلة.

9. **حافظ على مفاتيح الجلسة المؤقتة بشكل آمن**  
   خزّن `temp_session_key` في جلسة المستخدم أو قاعدة بيانات مؤقتة، ولا تعرضه في الواجهة الأمامية إلا للضرورة القصوى (لأنه يحمل معلومات حساسة).

10. **راجع إعدادات النظام**  
    تأكد من أن حقول الرفع مسجلة بشكل صحيح، وأن الإعدادات العامة (مثل `disable_upload`, `disable_edit`, `disable_auto_resize`) مضبوطة حسب احتياجاتك.

11. **راقب سجل `fileupload.log`**  
    ملف السجل المخصص (`storage/logs/fileupload.log`) يحتوي على تفاصيل محاولات الرفع الفاشلة، بما في ذلك محاولات رفع ملفات خطيرة، مما يساعد في مراقبة الأمان.

---

## الأسئلة الشائعة

**س: كيف أربط الملفات المؤقتة بنموذج بعد حفظه؟**  
ج: بعد حفظ النموذج (مثل `$product->save()`)، استدعِ دالة `attachTempFiles` من كود الخادم:
```php
\Nano\FileUpload\Classes\FileUploadService::instance()->attachTempFiles($product, 'image', $tempSessionKey);
```
يمكنك إنشاء نقطة نهاية مخصصة لهذا الغرض إذا كنت تريد ربطها عبر API.

**س: كيف أعرف أن عملية الرفع كانت `add` أم `edit`؟**  
ج: في استجابة الرفع الناجحة، يحتوي حقل `process_data` على المفتاح `operation` بقيمة `"add"` أو `"edit"`.

**س: ماذا يحدث إذا فشل رفع الملف الجديد في عملية `edit`؟**  
ج: بفضل استخدام المعاملات (`Db::transaction`)، لن يتم حذف الملف القديم. تبقى البيانات كما كانت قبل المحاولة.

**س: ما هي أحجام الصور المصغرة التي يمكن طلبها؟**  
ج: يمكنك طلب أي أبعاد تريدها عبر `thumb_sizes`، وسيتم إنشاء الصورة المصغرة تلقائياً (إذا لم تكن موجودة). الأحجام الشائعة: `[150,150,'crop']`, `[300,300,'crop']`, `[800,600,'auto']`.

**س: كيف أعرف أن الملف قد تم تحجيمه أو إضافة علامة مائية؟**  
ج: هذه العمليات تتم تلقائياً وفق إعدادات الحقل. إذا كنت مطوراً، راجع إعدادات الحقل (`auto_resize`, `auto_watermark`). إذا كنت مستخدم API، لا تحتاج إلى القيام بأي شيء إضافي.

**س: ماذا لو أردت رفع ملف بنفس الاسم مرتين؟**  
ج: النظام يقوم بتوليد اسم فريد (`disk_name`) تلقائياً، لذا لن يحدث تعارض. يمكنك الاعتماد على `id` و `path` للتمييز.

**س: هل يمكنني رفع ملفات بصيغة PDF أو DOCX؟**  
ج: نعم، إذا كان الحقل مسجلاً بنوع `file` أو تم تحديد `allowed_types` المناسب. أنواع الملفات المدعومة افتراضياً تشمل `pdf,doc,docx,xls,xlsx,zip,rar,txt` (لنوع `file`).

**س: كيف يمكنني الحصول على قائمة بجميع الموديولات والحقول المتاحة للمستخدم الحالي؟**  
ج: استخدم نقطة النهاية `/models`. ستحصل على الموديولات التي يملك المستخدم صلاحية `view` عليها مع الحقول الأساسية.

**س: كيف يمكنني التحقق مما إذا كان المستخدم يملك صلاحية رفع ملف لحقل معين دون محاولة الرفع؟**  
ج: استخدم نقطة النهاية `/permissions/check` مع `operation: "add"` والحقل المطلوب.

**س: المفتاح المؤقت الذي أحصل عليه من عملية الرفع، ما هي صلاحيته؟**  
ج: المفتاح صالح لمدة ساعة واحدة (قابلة للتغيير في إعدادات النظام). يمكنك التحقق من صلاحيته باستخدام نقطة `/temp-key/validate`.

**س: لماذا تم تغيير المسارات في الإصدار 1.2.5؟**  
ج: لتجنب مشاكل ترميز أسماء الموديولات التي تحتوي على خطوط مائلة عكسية (`\`) عند تمريرها في المسار. المسارات الجديدة باستخدام query parameters أبسط وأكثر أماناً. المسارات القديمة ما زالت تعمل حالياً ولكنها مهملة وستتم إزالتها في الإصدار 1.3.0. نوصي بالانتقال إلى المسارات الجديدة فوراً.

---

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
