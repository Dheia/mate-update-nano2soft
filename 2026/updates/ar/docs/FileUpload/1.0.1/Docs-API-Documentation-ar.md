# توثيق واجهة برمجة التطبيقات (API) لرفع الملفات

**المنتج:** نانوسوفت (NanoSoft)  
**الإصدار:** 1.0  
**المسار الأساسي:** `/api/v1/fileupload`  
**المصادقة:** OAuth 2.0 (توكن الوصول في الهيدر `Authorization: Bearer <token>`)

---

## نظرة عامة

توفر هذه الواجهة مجموعة من نقاط النهاية (Endpoints) لإدارة رفع الملفات في تطبيقات نانوسوفت، بالاعتماد على نظام `Nano.FileUpload`. تتيح هذه الخدمات للمطورين رفع ملف واحد أو عدة ملفات، حذف ملفات محددة، واسترجاع الملفات المرتبطة بنماذج معينة، مع التحقق التلقائي من الصلاحيات وأنواع المستخدمين والقيود المسجلة مسبقًا.

جميع الاستجابات تتبع الهيكل الموحد الموصوف أدناه، مما يسهل التعامل معها في تطبيقات العميل.

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
    "debug": {}
}
```

- `code`: كود HTTP (200 للنجاح، 400/403/404/500 للفشل).
- `status`: `true` للنجاح، `false` للفشل.
- `message`: رسالة توضيحية باللغة العربية.
- `error`: تفاصيل الخطأ (في حال الفشل).
- `errors`: مصفوفة من أخطاء التحقق (إذا وجدت).
- `data`: البيانات الأساسية للاستجابة (مثل معلومات الملف المرفوع).
- `meta`: بيانات إضافية (مثل معلومات التصفح) – حاليًا غير مستخدم.
- `input_data`: نسخة من البيانات التي أرسلها المستخدم (للتتبع).
- `process_data`: بيانات داخلية عن المعالجة (مثل مفتاح الجلسة المؤقت).
- `debug`: معلومات التصحيح (تظهر فقط عند تفعيل `app.debug`).

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
    "message": "تم الرفع بنجاح",
    "data": {
        "id": 123,
        "path": "/storage/app/uploads/public/.../original.jpg",
        "thumb": "/storage/app/uploads/public/.../thumb.jpg"
    },
    "input_data": { ... },
    "process_data": {
        "model_class": "Nano\\Shop\\Models\\Product",
        "field": "image",
        "temp_session_key": null,
        "user_type": "backend"
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
    "error": "الملف غير موجود"
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

---

## رموز الأخطاء الشائعة

| الكود | المعنى | الحل |
|-------|--------|------|
| 400 | بيانات غير صالحة (نقص معاملات، تحقق فاشل) | تحقق من صحة المدخلات. |
| 401 | غير مصرح (توكن غير صالح أو منتهي) | أعد تسجيل الدخول واحصل على توكن جديد. |
| 403 | صلاحية ممنوعة (المستخدم لا يملك صلاحية للعملية) | تأكد من صلاحيات المستخدم أو الحقل. |
| 404 | العنصر غير موجود (نموذج، حقل، ملف) | تأكد من صحة المعرف. |
| 500 | خطأ داخلي في الخادم | راجع سجلات الخادم مع تفعيل `app.debug`. |

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
3. **تعامل مع الأخطاء حسب الكود**: قدم للمستخدم رسائل مناسبة لكل حالة (401: سجل دخول، 403: لا صلاحية، 404: غير موجود).
4. **استخدم `with_thumbs` فقط عند الحاجة**: لتقليل حجم الاستجابة وزيادة الأداء.
5. **فعّل `app.debug` في بيئة التطوير**: للحصول على تفاصيل الأخطاء الكاملة.
6. **احتفظ بمفاتيح الجلسة المؤقتة في الجلسة أو قاعدة البيانات**: لاستخدامها لاحقًا في ربط الملفات.

---

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)