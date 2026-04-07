# توثيق واجهة برمجة التطبيقات (API) لرفع الملفات

**المنتج:** نانوسوفت (NanoSoft)  
**الإصدار:** 1.0.7  
**المسار الأساسي:** `/api/v1/fileupload`  
**المصادقة:** OAuth 2.0 (توكن الوصول في الهيدر `Authorization: Bearer <token>`)

---

## نظرة عامة

توفر هذه الواجهة مجموعة من نقاط النهاية (Endpoints) لإدارة رفع الملفات في تطبيقات نانوسوفت، بالاعتماد على نظام `Nano.FileUpload`. تتيح هذه الخدمات للمطورين رفع ملف واحد أو عدة ملفات، حذف ملفات محددة، واسترجاع الملفات المرتبطة بنماذج معينة، مع التحقق التلقائي من الصلاحيات وأنواع المستخدمين والقيود المسجلة مسبقًا، بالإضافة إلى دعم الميزات المتقدمة مثل التحويل التلقائي للصور والتخزين المتعدد.

جميع الاستجابات تتبع الهيكل الموحد الموصوف أدناه، وتحتوي على كود خطأ فريد (`error_code`) لتسهيل التعامل معها في تطبيقات العميل.

---

## هيكل الاستجابة الموحد

كل استجابة من واجهة API تأتي بالهيكل التالي:

```json
{
    "code": 200,
    "status": true,
    "message": "نص الرسالة",
    "error": null,
    "errors": [],
    "data": {},
    "meta": {},
    "input_data": {},
    "process_data": {},
    "debug": {},
    "error_code": "FILE_UPLOAD_SUCCESS"
}
```

- `code`: كود HTTP (200 للنجاح، 400/403/404/500 للفشل).
- `status`: `true` للنجاح، `false` للفشل.
- `message`: رسالة توضيحية باللغة العربية (أو اللغة المحددة في التطبيق).
- `error`: تفاصيل الخطأ (في حال الفشل).
- `errors`: مصفوفة من أخطاء التحقق (إذا وجدت).
- `data`: البيانات الأساسية للاستجابة (مثل معلومات الملف المرفوع).
- `meta`: بيانات إضافية (مثل معلومات التصفح) – حاليًا غير مستخدم.
- `input_data`: نسخة من البيانات التي أرسلها المستخدم (للتتبع).
- `process_data`: بيانات داخلية عن المعالجة (مثل مفتاح الجلسة المؤقت، قرص التخزين المستخدم).
- `debug`: معلومات التصحيح (تظهر فقط عند تفعيل `app.debug`).
- `error_code`: كود خطأ فريد (على سبيل المثال `FILE_UPLOAD_PERMISSION_DENIED`) – يظهر فقط في استجابات الخطأ، ويسهل التعامل مع الأخطاء برمجياً.

---

## المصادقة

جميع نقاط النهاية محمية بـ **OAuth 2.0**. يجب تضمين توكن الوصول في هيدر الطلب:

```
Authorization: Bearer <your_access_token>
```

إذا لم يتم توفير التوكن أو كان غير صالح، ستتلقى استجابة بالكود `401` مع رسالة "فشل المصادقة".

---

## نقاط النهاية

### 1. رفع ملف واحد

**الطريقة:** `POST`  
**المسار:** `/upload`

**معاملات الطلب (Body):**

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج (مثال: `Nano\Shop\Models\Product`). |
| `field` | `string` | نعم | اسم الحقل كما هو مسجل في `FileUploadRegistry`. |
| `file` | `file` | شرطي* | الملف المرفوع (بصيغة multipart). |
| `file_base64` | `string` | شرطي* | بيانات الملف بصيغة base64 (بديل عن `file`). |
| `temp_session_key` | `string` | لا | مفتاح جلسة مؤقت لربط الملف لاحقًا (يُستخدم مع النماذج غير المحفوظة). |
| `model_id` | `int` | لا | معرف النموذج المحفوظ (إذا كان موجودًا). |

> * يجب توفير إما `file` أو `file_base64`.

**ملاحظات متقدمة:**
- إذا تم توفير `model_id` وكان النموذج موجوداً، سيتم ربط الملف مباشرة.
- إذا لم يتم توفير `model_id` أو كان النموذج غير محفوظ، سيتم إنشاء مفتاح جلسة مؤقت (`temp_session_key`) وإعادته مع الاستجابة.
- يتم تطبيق التحويلات التلقائية (تحجيم، علامة مائية) على الصور تلقائياً وفق إعدادات الحقل والإعدادات العامة.
- يتم حساب تجزئة SHA256 للمحتوى وتخزينها في حقل `hash`، وتخزين أبعاد الصورة في `meta`، وتعيين تاريخ انتهاء للملفات المؤقتة.

**مثال طلب (multipart/form-data):**

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

(بيانات الملف)
------WebKitFormBoundary--
```

**مثال طلب (JSON مع base64):**

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

**استجابة نجاح (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع الملف بنجاح",
    "data": {
        "id": 123,
        "path": "/storage/app/uploads/public/.../original.jpg",
        "thumb": "/storage/app/uploads/public/.../thumb.jpg"
    },
    "input_data": { ... },
    "process_data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "temp_session_key": "tmp_abc123...",
        "user_type": "backend",
        "storage_disk": "s3"
    }
}
```

**استجابة خطأ (صلاحية ممنوعة - 403):**

```json
{
    "code": 403,
    "status": false,
    "message": "لا تملك صلاحية رفع الملفات لهذا الحقل",
    "error": "لا تملك صلاحية رفع الملفات لهذا الحقل",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED",
    "data": null
}
```

---

### 2. رفع عدة ملفات لحقل متعدد

**الطريقة:** `POST`  
**المسار:** `/upload-multiple`

**معاملات الطلب (Body):**

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل المتعدد (يجب أن يكون من نوع `multiple` في التسجيل). |
| `files` | `array` | نعم | مصفوفة من الملفات (كل عنصر يمكن أن يكون كائن ملف مرفوع أو سلسلة base64). |
| `temp_session_key` | `string` | لا | مفتاح جلسة مؤقت (للنماذج غير المحفوظة). |
| `model_id` | `int` | لا | معرف النموذج المحفوظ. |

**مثال طلب (multipart):**

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
------WebKitFormBoundary--
```

**استجابة نجاح (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع 2 من 2 ملف بنجاح",
    "data": [
        {
            "status": true,
            "data": { "id": 101, "path": "...", "thumb": "..." }
        },
        {
            "status": true,
            "data": { "id": 102, "path": "...", "thumb": "..." }
        }
    ],
    "process_data": {
        "success_count": 2,
        "total": 2
    }
}
```

**استجابة جزئية (بعض الملفات فشلت - 200 مع تفاصيل):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع 1 من 2 ملف بنجاح",
    "data": [
        {
            "status": true,
            "data": { "id": 101 }
        },
        {
            "status": false,
            "error": "نوع الملف غير مسموح به",
            "error_code": "FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED"
        }
    ],
    "process_data": {
        "success_count": 1,
        "total": 2
    }
}
```

---

### 3. حذف ملف

**الطريقة:** `DELETE`  
**المسار:** `/delete/{id}`

**معاملات المسار (URL):**

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `id` | `int` | نعم | معرف الملف المراد حذفه. |

**معاملات الاستعلام (Query Parameters):**

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | لا | اسم النموذج (للتحقق من صلاحية الحذف). |
| `field` | `string` | لا | اسم الحقل (للتحقق من صلاحية الحذف). |

**مثال طلب:**

```http
DELETE /api/v1/fileupload/delete/123?model_class=Nano\Shop\Models\Product&field=image HTTP/1.1
Authorization: Bearer ...
```

**استجابة نجاح (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم حذف الملف",
    "data": null
}
```

**استجابة خطأ (الملف غير موجود - 404):**

```json
{
    "code": 404,
    "status": false,
    "message": "الملف غير موجود",
    "error": "الملف غير موجود",
    "error_code": "FILE_UPLOAD_FILE_NOT_FOUND"
}
```

**استجابة خطأ (صلاحية ممنوعة - 403):**

```json
{
    "code": 403,
    "status": false,
    "message": "لا تملك صلاحية حذف هذا الملف",
    "error": "لا تملك صلاحية حذف هذا الملف",
    "error_code": "FILE_UPLOAD_PERMISSION_DENIED"
}
```

---

### 4. جلب الملفات المرتبطة

**الطريقة:** `GET`  
**المسار:** `/files`

**معاملات الاستعلام (Query Parameters):**

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `model_class` | `string` | نعم | اسم الفئة الكامل للنموذج. |
| `field` | `string` | نعم | اسم الحقل. |
| `model_id` | `int` | لا | معرف النموذج المحفوظ (إذا كان موجودًا). |
| `temp_session_key` | `string` | لا | مفتاح الجلسة المؤقت (لجلب الملفات المؤقتة غير المرتبطة). |
| `with_thumbs` | `bool` | لا | إضافة صور مصغرة (`true` / `false`، افتراضي `false`). |
| `thumb_sizes` | `object` | لا | أحجام الصور المصغرة (مصفوفة بأسماء الأحجام وأبعادها). |

**مثال طلب (جلب صور معرض منتج بصور مصغرة):**

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10&with_thumbs=true&thumb_sizes[small][0]=150&thumb_sizes[small][1]=150&thumb_sizes[small][2]=crop HTTP/1.1
Authorization: Bearer ...
```

**استجابة نجاح (200):**

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
            "path": "/storage/app/uploads/public/.../original.jpg",
            "size": 102400,
            "content_type": "image/jpeg",
            "small": "/storage/app/uploads/public/.../small.jpg"
        },
        {
            "id": 102,
            "title": null,
            "description": null,
            "path": "/storage/app/uploads/public/.../original.png",
            "size": 204800,
            "content_type": "image/png",
            "small": "/storage/app/uploads/public/.../small.png"
        }
    ]
}
```

**استجابة خطأ (مفتاح مؤقت غير صالح - 400):**

```json
{
    "code": 400,
    "status": false,
    "message": "مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية",
    "error": "مفتاح جلسة مؤقت غير صالح أو منتهي الصلاحية",
    "error_code": "FILE_UPLOAD_TEMP_KEY_INVALID"
}
```

---

## رموز الأخطاء الشائعة (error_code)

| error_code | كود HTTP | المعنى | الحل |
|------------|----------|--------|------|
| `FILE_UPLOAD_PERMISSION_DENIED` | 403 | المستخدم لا يملك صلاحية للعملية | تأكد من صلاحيات المستخدم أو الحقل. |
| `FILE_UPLOAD_UPLOAD_DISABLED_GLOBALLY` | 403 | عمليات الرفع معطلة عالمياً | انتظر حتى يتم تفعيلها أو راجع الإعدادات. |
| `FILE_UPLOAD_DELETE_DISABLED_GLOBALLY` | 403 | عمليات الحذف معطلة عالمياً | انتظر حتى يتم تفعيلها. |
| `FILE_UPLOAD_GET_DISABLED_GLOBALLY` | 403 | عمليات الجلب معطلة عالمياً | انتظر حتى يتم تفعيلها. |
| `FILE_UPLOAD_MODEL_NOT_REGISTERED` | 400 | النموذج غير مسجل في النظام | تأكد من صحة `model_class`. |
| `FILE_UPLOAD_FIELD_NOT_REGISTERED` | 400 | الحقل غير مسجل للنموذج | تأكد من صحة `field` وتسجيله. |
| `FILE_UPLOAD_MODEL_NOT_FOUND` | 404 | النموذج غير موجود (في حالة `model_id`) | تحقق من وجود النموذج في قاعدة البيانات. |
| `FILE_UPLOAD_FILE_NOT_FOUND` | 404 | الملف غير موجود | تحقق من صحة `id`. |
| `FILE_UPLOAD_INVALID_FILE_DATA` | 400 | بيانات الملف غير صالحة | تأكد من إرسال ملف صالح أو base64. |
| `FILE_UPLOAD_FILE_SIZE_EXCEEDED` | 400 | حجم الملف يتجاوز الحد المسموح | استخدم ملفاً أصغر أو زد الحد الأقصى. |
| `FILE_UPLOAD_FILE_TYPE_NOT_ALLOWED` | 400 | نوع الملف غير مسموح | استخدم نوعاً مسموحاً (راجع `allowed_types`). |
| `FILE_UPLOAD_FILE_TYPE_BLACKLISTED` | 400 | نوع الملف خطير (PHP, JS, HTML, إلخ) | استخدم نوعاً آمناً. |
| `FILE_UPLOAD_FILE_UPLOAD_FAILED` | 500 | فشل رفع الملف (خطأ داخلي) | راجع سجلات الخادم. |
| `FILE_UPLOAD_TEMP_KEY_INVALID` | 400 | مفتاح مؤقت غير صالح | أعد رفع الملف أو استخدم مفتاحاً جديداً. |
| `FILE_UPLOAD_TEMP_KEY_EXPIRED` | 400 | انتهت صلاحية المفتاح المؤقت | أعد رفع الملف. |
| `FILE_UPLOAD_TEMP_KEY_MISMATCH` | 403 | المفتاح المؤقت لا يخص المستخدم الحالي | تأكد من استخدام المفتاح الصحيح. |
| `FILE_UPLOAD_USER_TYPE_NOT_ALLOWED` | 403 | نوع المستخدم غير مسموح | تحقق من `allowed_user_types` في التسجيل. |

---

## أمثلة عملية متكاملة

### مثال 1: رفع صورة لمنتج جديد (باستخدام مفتاح جلسة مؤقت)

**الخطوة 1:** رفع الصورة والحصول على `temp_session_key`

```http
POST /api/v1/fileupload/upload
{
    "model_class": "Nano\\Shop\\Models\\Product",
    "field": "image",
    "file_base64": "data:image/jpeg;base64,..."
}
```

**الاستجابة:**

```json
{
    "status": true,
    "data": { "id": 101, "path": "..." },
    "temp_session_key": "tmp_abc123"
}
```

**الخطوة 2:** إنشاء المنتج (في تطبيقك) وحفظه

```php
$product = new Product();
$product->name = 'منتج جديد';
$product->save();
```

**الخطوة 3:** ربط الصورة بالمنتج عبر `attachTempFiles` (يتم من الخادم، وليس عبر API مباشرة).  
يمكن إضافة نقطة نهاية مخصصة لهذا الغرض أو تنفيذها في كود الخادم.

---

### مثال 2: رفع عدة صور لمعرض منتج موجود

```http
POST /api/v1/fileupload/upload-multiple
Content-Type: multipart/form-data

model_class: Nano\Shop\Models\Product
field: gallery
model_id: 10
files[]: (ملف1)
files[]: (ملف2)
```

**الاستجابة:**

```json
{
    "status": true,
    "message": "تم رفع 2 من 2 ملف بنجاح",
    "data": [ { "data": { "id": 102 } }, { "data": { "id": 103 } } ]
}
```

---

### مثال 3: حذف صورة رئيسية من منتج

```http
DELETE /api/v1/fileupload/delete/101?model_class=Nano\Shop\Models\Product&field=image
```

**الاستجابة:**

```json
{
    "status": true,
    "message": "تم حذف الملف"
}
```

---

### مثال 4: جلب صور معرض منتج مع صور مصغرة

```http
GET /api/v1/fileupload/files?model_class=Nano\Shop\Models\Product&field=gallery&model_id=10&with_thumbs=true&thumb_sizes[medium][0]=300&thumb_sizes[medium][1]=300&thumb_sizes[medium][2]=crop
```

**الاستجابة:**

```json
{
    "status": true,
    "data": [
        {
            "id": 102,
            "path": "...",
            "medium": "/storage/.../medium.jpg"
        }
    ]
}
```

---

## أفضل الممارسات

1. **استخدم مفتاح جلسة مؤقت للنماذج الجديدة**: لتجنب فقدان الملفات إذا لم يتم حفظ النموذج بعد.
2. **تحقق من صحة الإدخال قبل إرسال الطلب**: خاصة `model_class` و `field` لضمان وجودهما في التسجيل.
3. **تعامل مع الأخطاء حسب `error_code`**: قدم للمستخدم رسائل مناسبة لكل حالة (مثلاً `FILE_UPLOAD_PERMISSION_DENIED` → "غير مصرح").
4. **استخدم `with_thumbs` فقط عند الحاجة**: لتقليل حجم الاستجابة وزيادة الأداء.
5. **فعّل `app.debug` في بيئة التطوير**: للحصول على تفاصيل الأخطاء الكاملة (تظهر في حقل `debug`).
6. **احتفظ بمفاتيح الجلسة المؤقتة في الجلسة أو قاعدة البيانات**: لاستخدامها لاحقًا في ربط الملفات.
7. **راقب سجل `fileupload.log`** للكشف عن محاولات رفع الملفات الخطيرة أو الأخطاء المتكررة.
8. **استخدم `thumb_sizes` بأحجام مناسبة** لتجنب إنشاء صور مصغرة كثيرة تستهلك مساحة التخزين.

---

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)