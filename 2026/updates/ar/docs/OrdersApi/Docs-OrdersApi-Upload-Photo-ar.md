# توثيق واجهة برمجة التطبيقات (API) لإدارة صور الطلبات

**الإصدار:** 1.0.22  
**المسار الأساسي:** `/api/v1/orders`  
**المصادقة:** OAuth 2.0 (توكن الوصول في الهيدر `Authorization: Bearer <token>`)

---

## نظرة عامة

توفر هذه النقاط الثلاث إمكانية إدارة الصور المرتبطة بالطلب (Order)، وتستخدم خصيصاً لسيناريوهات تأجير السلع حيث يحتاج المالك (`user_id`) والموصل (`delivery_user_id`) إلى رفع صور قبل وبعد التسليم. يتم التحقق من الصلاحيات بناءً على:

- **دور المستخدم** (مالك الطلب / موصل الطلب / مدير).
- **حالة الطلب الحالية** (NEW, PROCESSING, DELIVERY, COMPLETE).
- **الحد الأقصى لعدد الصور** (10 صور لكل حقل افتراضياً).
- **حجم الملف وأنواعه** (jpg, jpeg, png, gif حتى 5 ميجابايت).

> **ملاحظة:** هذه النقاط تعتمد على السمات `StepPhotos` في إضافة `Nano.Orders` الإصدار 2.2.12 وما فوق.

---

## 1. رفع صورة واحدة

**الطريقة:** `POST`  
**المسار:** `/upload-photo`  
**نوع المحتوى:** `multipart/form-data` أو `application/json`

### معاملات الطلب

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `order_id` | `int` | نعم | معرف الطلب المرتبط بالصورة. |
| `field` | `string` | نعم | اسم الحقل: `user_before_delivery`, `user_after_delivery`, `delivery_before_delivery`, `delivery_after_delivery`. |
| `file` | `file` | شرطي* | الملف المرفوع (بصيغة multipart). |
| `file_base64` | `string` | شرطي* | بيانات الملف بصيغة base64 (بديل عن `file`). |
| `title` | `string` | لا | عنوان الصورة. |
| `description` | `string` | لا | وصف الصورة. |
| `is_public` | `bool` | لا | هل الصورة عامة (افتراضي `false`). |
| `auto_resize` | `bool` | لا | تفعيل التحجيم التلقائي للصورة. |
| `resize_options` | `object` | لا | إعدادات التحجيم (مثال: `{"width":800,"height":600,"mode":"crop"}`). |
| `auto_watermark` | `bool` | لا | تفعيل العلامة المائية (يتطلب إضافة Watermark). |
| `custom_message` | `string` | لا | رسالة نجاح مخصصة. |

> * يجب توفير إما `file` (multipart) أو `file_base64` (JSON).

### أمثلة على الطلبات

#### مثال 1: رفع صورة باستخدام multipart/form-data (مالك الطلب)

```http
POST /api/v1/orders/upload-photo HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="order_id"

125
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

user_before_delivery
------WebKitFormBoundary
Content-Disposition: form-data; name="title"

صورة قبل التسليم
------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="product.jpg"
Content-Type: image/jpeg

(بيانات الملف الثنائية)
------WebKitFormBoundary--
```

#### مثال 2: رفع صورة باستخدام JSON و base64 (موصل الطلب)

```json
POST /api/v1/orders/upload-photo HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "order_id": 125,
    "field": "delivery_before_delivery",
    "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
    "title": "صورة قبل التسليم من الموصل",
    "description": "تم الرفع بواسطة الموصل"
}
```

### الاستجابات

#### ✅ استجابة نجاح (رفع صورة واحدة)

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع الصورة بنجاح",
    "data": {
        "id": 456,
        "path": "https://yourdomain.com/storage/app/uploads/public/.../original.jpg",
        "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg",
        "photos": {
            "thumb": "https://yourdomain.com/storage/app/uploads/public/.../thumb.jpg",
            "medium": "https://yourdomain.com/storage/app/uploads/public/.../medium.jpg",
            "large": "https://yourdomain.com/storage/app/uploads/public/.../large.jpg"
        },
        "title": "صورة قبل التسليم",
        "description": null,
        "size": 123456,
        "content_type": "image/jpeg",
        "created_at": "2026-06-05 10:30:00"
    },
    "input_data": { ... },
    "process_data": { ... }
}
```

#### ❌ استجابة خطأ (صلاحية ممنوعة – دور المستخدم غير مسموح)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل رفع الصورة",
    "error": "المستخدم من نوع user غير مسموح له برفع الصور في حقل delivery_before_delivery",
    "error_code": null
}
```

#### ❌ استجابة خطأ (حالة الطلب غير مسموحة للرفع)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل رفع الصورة",
    "error": "لا يمكن رفع الصور في حقل user_before_delivery في الحالة الحالية (COMPLETE)",
    "error_code": null
}
```

#### ❌ استجابة خطأ (الحد الأقصى لعدد الصور)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل رفع الصورة",
    "error": "تم الوصول إلى الحد الأقصى لعدد الصور (10) في حقل user_before_delivery",
    "error_code": null
}
```

#### ❌ استجابة خطأ (الملف مطلوب)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل رفع الصورة",
    "error": "الملف مطلوب",
    "error_code": null
}
```

#### ❌ استجابة خطأ (طلب غير مصرح به – توكن مفقود)

```json
{
    "code": 401,
    "status": false,
    "message": "فشل رفع الصورة",
    "error": "User not found",
    "error_code": null
}
```

---

## 2. رفع عدة صور (لحقل متعدد)

**الطريقة:** `POST`  
**المسار:** `/upload-multiple-photos`  
**نوع المحتوى:** `multipart/form-data` أو `application/json`

### معاملات الطلب

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `order_id` | `int` | نعم | معرف الطلب المرتبط بالصورة. |
| `field` | `string` | نعم | اسم الحقل (نفس القيم المذكورة أعلاه). |
| `files` | `array` | نعم | مصفوفة من الملفات (كل عنصر إما كائن `UploadedFile` أو سلسلة base64). |
| `title` | `string` | لا | عنوان عام للصور (سيتم تطبيقه على جميع الصور). |
| `description` | `string` | لا | وصف عام للصور. |
| `is_public` | `bool` | لا | هل الصور عامة (افتراضي `false`). |
| `auto_resize` | `bool` | لا | تفعيل التحجيم التلقائي لجميع الصور. |

### مثال طلب (multipart/form-data)

```http
POST /api/v1/orders/upload-multiple-photos HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="order_id"

125
------WebKitFormBoundary
Content-Disposition: form-data; name="field"

user_after_delivery
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="photo1.jpg"
Content-Type: image/jpeg

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="photo2.png"
Content-Type: image/png

...
------WebKitFormBoundary
Content-Disposition: form-data; name="files[]"; filename="photo3.jpg"
Content-Type: image/jpeg

...
------WebKitFormBoundary--
```

### مثال طلب (JSON مع base64)

```json
POST /api/v1/orders/upload-multiple-photos HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "order_id": 125,
    "field": "user_after_delivery",
    "files": [
        "data:image/jpeg;base64,/9j/4AAQSkZJRg...",
        "data:image/png;base64,iVBORw0KGgo...",
        "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
    ],
    "title": "صور بعد التسليم"
}
```

### الاستجابات

#### ✅ استجابة نجاح (جميع الصور رُفعت بنجاح)

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع جميع الصور بنجاح",
    "data": [
        {
            "status": true,
            "data": {
                "id": 456,
                "path": "https://.../photo1.jpg",
                "thumb": "https://.../thumb1.jpg"
            }
        },
        {
            "status": true,
            "data": {
                "id": 457,
                "path": "https://.../photo2.png",
                "thumb": "https://.../thumb2.jpg"
            }
        },
        {
            "status": true,
            "data": {
                "id": 458,
                "path": "https://.../photo3.jpg",
                "thumb": "https://.../thumb3.jpg"
            }
        }
    ],
    "process_data": {
        "success_count": 3,
        "total": 3
    },
    "errors": []
}
```

#### ⚠️ استجابة نجاح جزئي (بعض الصور فشلت)

```json
{
    "code": 207,
    "status": true,
    "message": "تم رفع 2 من 3 صورة بنجاح",
    "data": [
        {
            "status": true,
            "data": {
                "id": 456,
                "path": "https://.../photo1.jpg"
            }
        },
        {
            "status": true,
            "data": {
                "id": 457,
                "path": "https://.../photo2.png"
            }
        },
        {
            "status": false,
            "error": "الملف مطلوب",
            "data": null
        }
    ],
    "process_data": {
        "success_count": 2,
        "total": 3
    },
    "errors": {
        "2": "الملف مطلوب"
    }
}
```

#### ❌ استجابة فشل كامل (جميع الصور فشلت)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل رفع الصورة",
    "data": null,
    "process_data": {
        "success_count": 0,
        "total": 2
    },
    "errors": {
        "0": "نوع الملف غير مسموح به",
        "1": "حجم الملف يتجاوز الحد المسموح به"
    }
}
```

---

## 3. حذف صورة

**الطريقة:** `POST` أو `DELETE`  
**المسار:** 
- `/delete-photo` (POST) – يتم تمرير المعاملات في نص الطلب (JSON).
- `/delete-photo/{orderId}/{field}/{fileId}` (DELETE) – يتم تمرير المعاملات في المسار.

### معاملات الطلب (POST JSON)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `order_id` | `int` | نعم | معرف الطلب. |
| `field` | `string` | نعم | اسم الحقل الذي يحتوي على الصورة. |
| `file_id` | `int` | نعم | معرف الصورة المراد حذفها. |

### معاملات المسار (DELETE)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `orderId` | `int` | نعم | معرف الطلب. |
| `field` | `string` | نعم | اسم الحقل. |
| `fileId` | `int` | نعم | معرف الصورة. |

### أمثلة على الطلبات

#### مثال 1: حذف باستخدام POST JSON

```http
POST /api/v1/orders/delete-photo HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
Content-Type: application/json

{
    "order_id": 125,
    "field": "user_before_delivery",
    "file_id": 456
}
```

#### مثال 2: حذف باستخدام DELETE مع معاملات المسار

```http
DELETE /api/v1/orders/delete-photo/125/user_before_delivery/456 HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

### الاستجابات

#### ✅ استجابة نجاح (حذف الصورة)

```json
{
    "code": 200,
    "status": true,
    "message": "تم حذف الصورة بنجاح",
    "data": {
        "deleted_file_id": 456
    },
    "error": null,
    "errors": null
}
```

#### ❌ استجابة خطأ (الملف غير موجود)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل حذف الصورة",
    "error": "الصورة غير موجودة",
    "data": null
}
```

#### ❌ استجابة خطأ (صلاحية ممنوعة – المستخدم ليس مالك الصورة)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل حذف الصورة",
    "error": "المستخدم من نوع delivery غير مسموح له برفع الصور في حقل user_before_delivery",
    "data": null
}
```

#### ❌ استجابة خطأ (عدم وجود معرف الصورة)

```json
{
    "code": 400,
    "status": false,
    "message": "فشل حذف الصورة",
    "error": "معرف الصورة مطلوب",
    "data": null
}
```

---

## 4. قيود الحقول وأنواعها

| الحقل | المستخدم المخوّل | الحالات المسموحة | الحد الأقصى للصور |
|-------|------------------|------------------|-------------------|
| `user_before_delivery` | `user` (مالك الطلب) | `NEW`, `PROCESSING` | 10 |
| `user_after_delivery` | `user` (مالك الطلب) | `DELIVERY` | 10 |
| `delivery_before_delivery` | `delivery` (موصل) | `NEW`, `PROCESSING` | 10 |
| `delivery_after_delivery` | `delivery` (موصل) | `DELIVERY` | 10 |

> **ملاحظة:** المدير (`admin`) لديه صلاحية كاملة لرفع الصور في جميع الحقول وفي أي حالة.

---

## 5. أفضل الممارسات

1. **استخدم `multipart/form-data` عند رفع ملفات مباشرة**، فهو أكثر كفاءة من base64.
2. **استخدم `file_base64` فقط عند الضرورة** (مثل التطبيقات التي لا تدعم multipart أو عند دمج الصور من بيانات أخرى).
3. **تأكد من أن حالة الطلب مناسبة لرفع الصور** قبل استدعاء الواجهة (يمكن التحقق مسبقاً باستخدام `GET /orders/{id}` وعرض `order_states_ref_type`).
4. **تعامل مع `code = 207` في `upload-multiple-photos`** كنجاح جزئي، وعرض الأخطاء لكل ملف فاشل.
5. **لحذف الصور، يفضل استخدام طريقة `DELETE`** لأنها تعبر بشكل أفضل عن نية العملية (RESTful).
6. **تأكد من أن المستخدم لديه صلاحية الحذف** – المدير فقط يمكنه حذف صور الأطراف الأخرى.

---

## 6. أكواد الأخطاء الشائعة (HTTP Codes)

| كود HTTP | المعنى |
|----------|--------|
| `200` | نجاح (جميع الصور رُفعت / حُذفت). |
| `207` | نجاح جزئي (بعض الصور رُفعت، وبعضها فشل). |
| `400` | خطأ في الطلب (بيانات غير صحيحة، صلاحية مرفوضة، ملف غير صالح). |
| `401` | غير مصرح (توكن OAuth مفقود أو غير صالح). |
| `404` | الطلب أو الصورة غير موجودة. |
| `500` | خطأ داخلي في الخادم (يظهر تفاصيل في `debug` عند تفعيل وضع التصحيح). |

---

## 7. التكامل مع `update-status`

عند محاولة تغيير حالة الطلب إلى `DELIVERY` أو `COMPLETE` عبر نقطة `update-status`، سيتم رفض الطلب إذا كانت الصور المطلوبة غير موجودة، مع رسالة توضح الحقل المطلوب.

**مثال على فشل تغيير الحالة بسبب نقص الصور:**

```http
POST /api/v1/orders/update-status
Content-Type: application/json

{
    "order_id": 125,
    "order_states_ref_type": "DELIVERY"
}
```

**الاستجابة:**

```json
{
    "code": 400,
    "status": false,
    "message": "فشل تحديث حالة الطلب.",
    "error": "يجب رفع الصور في حقل user_before_delivery قبل تغيير الحالة إلى DELIVERY",
    "data": null
}
```

لتجاوز هذا الفحص (للمسؤولين فقط)، يمكن تمرير `skip_photo_check: true`:

```json
{
    "order_id": 125,
    "order_states_ref_type": "DELIVERY",
    "skip_photo_check": true,
    "admin_override": true
}
```

---

## 8. أمثلة عملية متكاملة

### سيناريو 1: مالك الطلب يرفع صوراً قبل التسليم

**الطلب (multipart):**
```http
POST /api/v1/orders/upload-photo
Authorization: Bearer {user_token}
Content-Type: multipart/form-data

order_id: 125
field: user_before_delivery
file: @/home/user/photo.jpg
title: صورة السلعة قبل التوصيل
```

**الاستجابة المتوقعة (بعد النجاح):**
```json
{
    "code": 200,
    "status": true,
    "message": "تم رفع الصورة بنجاح",
    "data": {
        "id": 456,
        "path": "https://example.com/storage/.../photo.jpg",
        "thumb": "https://example.com/storage/.../thumb.jpg",
        "photos": {
            "thumb": "https://example.com/storage/.../thumb.jpg",
            "medium": "https://example.com/storage/.../medium.jpg",
            "large": "https://example.com/storage/.../large.jpg"
        }
    }
}
```

### سيناريو 2: موصل الطلب يرفع صوراً متعددة قبل التسليم

**الطلب (JSON with base64):**
```json
POST /api/v1/orders/upload-multiple-photos
Authorization: Bearer {delivery_token}
Content-Type: application/json

{
    "order_id": 125,
    "field": "delivery_before_delivery",
    "files": ["data:image/jpeg;base64,...", "data:image/jpeg;base64,..."],
    "title": "صور من الموصل قبل التوصيل"
}
```

**الاستجابة المتوقعة (نجاح جزئي – الملف الثاني فشل لوجود مشكلة في base64):**
```json
{
    "code": 207,
    "status": true,
    "message": "تم رفع 1 من 2 صورة بنجاح",
    "data": [
        {
            "status": true,
            "data": {
                "id": 457,
                "path": "https://example.com/storage/.../photo1.jpg"
            }
        },
        {
            "status": false,
            "error": "بيانات الملف غير صالحة",
            "data": null
        }
    ],
    "process_data": {
        "success_count": 1,
        "total": 2
    }
}
```

### سيناريو 3: حذف صورة (المستخدم الموصل يحذف صورته فقط)

**الطلب (DELETE):**
```http
DELETE /api/v1/orders/delete-photo/125/delivery_before_delivery/457
Authorization: Bearer {delivery_token}
```

**الاستجابة المتوقعة (نجاح):**
```json
{
    "code": 200,
    "status": true,
    "message": "تم حذف الصورة بنجاح",
    "data": {
        "deleted_file_id": 457
    }
}
```

### سيناريو 4: محاولة رفع صورة بعد إكمال الطلب (مرفوض)

**الطلب:**
```http
POST /api/v1/orders/upload-photo
Authorization: Bearer {user_token}
Content-Type: multipart/form-data

order_id: 125
field: user_before_delivery
file: @photo.jpg
```

**الاستجابة (بما أن حالة الطلب الآن `COMPLETE`):**
```json
{
    "code": 400,
    "status": false,
    "message": "فشل رفع الصورة",
    "error": "لا يمكن رفع الصور في حقل user_before_delivery في الحالة الحالية (COMPLETE)"
}
```

---

## 9. ملاحظات إضافية

- **حجم الملف الافتراضي:** 5 ميجابايت (5120 كيلوبايت). يمكن تغييره عبر إعدادات `photo_rules` في `config/nano/orders.php`.
- **الأنواع المسموحة:** `jpg, jpeg, png, gif` (افتراضياً). يمكن إضافة `webp` أو أنواع أخرى عبر الإعدادات.
- **الصور المصغرة:** تُنشأ تلقائياً بأحجام `thumb` (150×150)، `medium` (300×300)، `large` (800×600). يمكن طلب أحجام مخصصة عبر `thumb_sizes` في الخيارات (غير موثق هنا، ولكنه موجود في دالة `getOrderPhotoUrls`).
- **الأحداث:** عند رفع الصورة بنجاح، يتم إطلاق حدث `nano.orders.photo.uploaded`. عند الحذف، يتم إطلاق `nano.orders.photo.deleted`، يمكن الاستماع لهما لتوسيع السلوك.
- **الترجمة:** جميع رسائل الخطأ والنجاح قابلة للترجمة عبر ملفات `lang` الخاصة بـ `Nano.OrdersApi`.

---

**الوثائق المرجعية:**
- [توثيق سمة `StepPhotos`](./docs/Orders/Traits/Steps/Docs-StepPhotos-Trait-ar.md)
- [توثيق `OrderManager` وسماته](./docs/Orders/Classes/Docs-OrderManager-Class-ar.md)
- [توثيق واجهة API لتحديث حالة الطلب](./docs/OrdersApi/Docs-OrdersApi-UpdateStatus-ar.md)