# توثيق سيناريو إنشاء وثيقة KYC عبر API

## نظرة عامة

يشرح هذا الدليل الخطوات الكاملة لبناء واجهة أمامية تفاعلية لإنشاء وثيقة KYC جديدة. يتضمن السيناريو:

1. جلب تصنيفات الوثائق (اختياري) – لتمكين المستخدم من اختيار الفئة أولاً.
2. جلب أنواع الوثائق المتاحة (مع إمكانية فلترتها حسب التصنيف).
3. جلب الحقول الديناميكية الخاصة بنوع الوثيقة المختار – لبناء نموذج الإدخال.
4. إرسال بيانات النموذج مع المرفقات إلى نقطة نهاية إنشاء الوثيقة.

جميع الطلبات تتطلب مصادقة عبر `Authorization: Bearer <token>`.

---

## الخطوة 1: جلب تصنيفات الوثائق (اختياري)

يمكن للمطور عرض قائمة التصنيفات أولاً ليختار المستخدم الفئة المطلوبة (مثلاً: هوية أساسية، إثبات عنوان).

**نقطة النهاية:** `GET /api/v1/kyc/document-categories-list`

**الاستخدام:**
```http
GET /api/v1/kyc/document-categories-list HTTP/1.1
Authorization: Bearer {{access_token}}
```

**الاستجابة (بدون تضمين الأنواع):**
```json
{
    "code": 200,
    "status": true,
    "data": [
        {
            "code": "primary_id",
            "name_ar": "هوية أساسية",
            "name_en": "Primary ID",
            "name": "هوية أساسية"
        },
        {
            "code": "secondary_id",
            "name_ar": "هوية ثانوية",
            "name_en": "Secondary ID",
            "name": "هوية ثانوية"
        },
        // ... باقي التصنيفات
    ]
}
```

> **ملاحظة:** يمكن طلب التصنيفات مع أنواعها مباشرة لتوفير طلب إضافي (انظر الخطوة 2).

---

## الخطوة 2: جلب أنواع الوثائق

بعد أن يحدد المستخدم التصنيف (أو مباشرة)، نجلب قائمة أنواع الوثائق المتاحة. يمكن فلترة القائمة حسب التصنيف لتقليل الخيارات.

**نقطة النهاية:** `GET /api/v1/kyc/document-types-list`

### أ. جلب جميع الأنواع (بدون فلترة)

```http
GET /api/v1/kyc/document-types-list HTTP/1.1
Authorization: Bearer {{access_token}}
```

### ب. جلب أنواع تصنيف محدد فقط

لإظهار أنواع "الهوية الأساسية" فقط:

```http
GET /api/v1/kyc/document-types-list?category_filter=primary_id HTTP/1.1
```

**الاستجابة (مثال لنوعين):**
```json
{
    "code": 200,
    "status": true,
    "data": [
        {
            "code": "passport",
            "name_ar": "جواز سفر",
            "name_en": "Passport",
            "name": "جواز سفر",
            "description": "جواز سفر ساري المفعول يحتوي على صورة شخصية وبيانات المسافر."
        },
        {
            "code": "national_id",
            "name_ar": "بطاقة هوية وطنية",
            "name_en": "National ID Card",
            "name": "بطاقة هوية وطنية",
            "description": "بطاقة الهوية الوطنية الصادرة عن الجهات الرسمية."
        }
    ]
}
```

> **نصيحة:** يمكن تضمين `include_category=1` للحصول على معلومات التصنيف الأب لكل نوع.

---

## الخطوة 3: جلب حقول نوع الوثيقة (لبناء النموذج)

بعد أن يختار المستخدم نوع الوثيقة (مثلاً `passport`)، نقوم بجلب تعريف الحقول الخاص بهذا النوع لبناء نموذج الإدخال ديناميكيًا.

**نقطة النهاية:** `GET /api/v1/kyc/document-fields/{type}`

**مثال لجواز السفر:**
```http
GET /api/v1/kyc/document-fields/passport HTTP/1.1
Authorization: Bearer {{access_token}}
```

**الاستجابة (مختصرة):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "type": "passport",
        "type_name": "جواز سفر",
        "fields": [
            {
                "id": "document_front",
                "type": "fileupload",
                "mode": "image",
                "label": { "ar": "وجه الوثيقة", "en": "Document Front" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "mimes", "value": "jpeg,jpg,png,pdf" },
                    { "rule": "max", "value": 10240 }
                ]
            },
            {
                "id": "document_number",
                "type": "text",
                "label": { "ar": "رقم الوثيقة", "en": "Document Number" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "max", "value": 50 }
                ]
            },
            {
                "id": "full_name",
                "type": "text",
                "label": { "ar": "الاسم الكامل", "en": "Full Name" },
                "required": false,
                "validation": [ { "rule": "max", "value": 100 } ]
            },
            {
                "id": "issue_date",
                "type": "date",
                "label": { "ar": "تاريخ الإصدار", "en": "Issue Date" },
                "required": false,
                "validation": [
                    { "rule": "date" },
                    { "rule": "before_or_equal", "value": "today" }
                ]
            },
            {
                "id": "expiry_date",
                "type": "date",
                "label": { "ar": "تاريخ الانتهاء", "en": "Expiry Date" },
                "required": true,
                "validation": [
                    { "rule": "required" },
                    { "rule": "date" },
                    { "rule": "after_or_equal", "value": "today" }
                ]
            },
            {
                "id": "country_of_issue",
                "type": "select",
                "label": { "ar": "بلد الإصدار", "en": "Country of Issue" },
                "required": true,
                "data_source": {
                    "type": "api",
                    "endpoint": "api/v1/location/countries",
                    "value_field": "code",
                    "label_field": "name"
                },
                "resolved_options": [
                    { "id": "SA", "name": "السعودية" },
                    { "id": "EG", "name": "مصر" }
                    // ...
                ]
            }
            // ... باقي الحقول
        ],
        "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
        "max_file_size_kb": 10240,
        "requires_back_side": false,
        "requires_selfie_match": true
    }
}
```

### شرح الحقول المهمة

| الخاصية | الوصف |
|----------|-------|
| `type` | نوع حقل الإدخال (`text`, `date`, `select`, `fileupload`, ...). |
| `required` | هل الحقل إجباري؟ |
| `validation` | قواعد التحقق (يمكن تحويلها لـ Laravel rules أو استخدامها في التحقق من جانب العميل). |
| `data_source` | مصدر خيارات القوائم المنسدلة (API خارجي). |
| `resolved_options` | الخيارات المحلولة الجاهزة للاستخدام في `<select>` (تُجلب تلقائيًا). |
| `mode` | لحقول الملفات (`image`, `file`). |
| `allowed_mime_types` | الامتدادات المسموحة للملفات. |

> **نصيحة للمطور:** استخدم `validation` لبناء قواعد تحقق فورية على الواجهة الأمامية، واستخدم `resolved_options` لملء القوائم المنسدلة.

---

## الخطوة 4: إرسال بيانات الوثيقة والملفات

بعد أن يملأ المستخدم النموذج، نرسل البيانات إلى نقطة نهاية الإنشاء. تدعم نقطة النهاية إرسال الملفات بطريقتين:

- **multipart/form-data** (ملفات مرفوعة مباشرة).
- **JSON مع Base64** (بإضافة لاحقة `_base64` لاسم الحقل).

**نقطة النهاية:** `POST /api/v1/kyc/documents`

### أ. الإرسال باستخدام multipart/form-data

**مثال (JavaScript – FormData):**
```javascript
const formData = new FormData();
formData.append('document_type', 'passport');
formData.append('document_number', 'P12345678');
formData.append('full_name', 'أحمد محمد');
formData.append('issue_date', '2020-01-01');
formData.append('expiry_date', '2030-01-01');
formData.append('country_of_issue', 'YE');
formData.append('nationality', 'YE');
formData.append('document_front', fileInput.files[0]);

fetch('/api/v1/kyc/documents', {
    method: 'POST',
    headers: { 'Authorization': 'Bearer ' + token },
    body: formData
});
```

### ب. الإرسال باستخدام JSON + Base64

**مثال (JavaScript):**
```javascript
const payload = {
    document_type: 'passport',
    document_number: 'P12345678',
    full_name: 'أحمد محمد',
    issue_date: '2020-01-01',
    expiry_date: '2030-01-01',
    country_of_issue: 'YE',
    nationality: 'YE',
    document_front_base64: 'data:image/jpeg;base64,/9j/4AAQSkZJRg...'
};

fetch('/api/v1/kyc/documents', {
    method: 'POST',
    headers: {
        'Authorization': 'Bearer ' + token,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(payload)
});
```

### معاملات الطلب

| المعامل | النوع | مطلوب | الوصف |
|----------|------|-------|-------|
| `document_type` | string | نعم | كود نوع الوثيقة (مثلاً `passport`). |
| `owner_type` / `owner_id` | string/int | لا* | إذا كان المستخدم `backend` يمكنه تحديد مالك آخر. |
| **حقول النموذج** | متنوع | حسب النوع | جميع الحقول المرئية في `fields` (مثلاً `document_number`, `full_name`). |
| **الملفات** | file/base64 | حسب النوع | `document_front`, `document_back`, `signature_image`, `image`, `files[]`. |

> *المستخدم `frontend` يتم تعيينه تلقائياً كمالك.

### استجابة النجاح (تم إنشاء الوثيقة ورفع الملفات)

```json
{
    "code": 200,
    "status": true,
    "message": "تم إنشاء الوثيقة بنجاح.",
    "data": {
        "id": 142,
        "document_type": "passport",
        "document_number": "P12345678",
        "full_name": "أحمد محمد",
        "status": "pending",
        "created_at": "2026-04-22 14:30:00",
        "document_front": {
            "original": "https://account.now-ye.com/storage/app/uploads/public/.../original.jpg",
            "thumb": "https://account.now-ye.com/storage/app/uploads/public/.../thumb.jpg"
        }
    },
    "process_data": {
        "file_errors": []
    }
}
```

### استجابة نجاح جزئي (بعض الملفات فشلت)

```json
{
    "code": 200,
    "status": true,
    "message": "تم إنشاء الوثيقة ولكن بعض الملفات فشل رفعها.",
    "data": { ... },
    "process_data": {
        "file_errors": {
            "document_back": "حجم الملف يتجاوز الحد المسموح به (10240 KB)"
        }
    }
}
```

### استجابة خطأ (فشل التحقق)

```json
{
    "code": 400,
    "status": false,
    "message": "حدث خطأ أثناء إنشاء الوثيقة.",
    "error": "الخاصية document number حقل مطلوب",
    "errors": {
        "document_number": ["الخاصية document number حقل مطلوب"],
        "expiry_date": ["الخاصية expiry date حقل مطلوب"]
    }
}
```

---

## ملخص سير العمل

| الخطوة | الإجراء | نقطة النهاية |
|--------|---------|---------------|
| 1 | (اختياري) عرض التصنيفات | `GET /document-categories-list` |
| 2 | عرض أنواع الوثائق (فلترة حسب التصنيف) | `GET /document-types-list?category_filter=...` |
| 3 | جلب حقول النوع المختار | `GET /document-fields/{type}` |
| 4 | بناء النموذج ديناميكيًا بناءً على `fields` | - |
| 5 | إرسال البيانات + الملفات | `POST /documents` (multipart أو JSON+Base64) |
| 6 | عرض نتيجة الإنشاء (نجاح / فشل) | - |

---

## نصائح للمطورين

- **استخدم `resolved_options` مباشرة** في القوائم المنسدلة، ولا تقم بطلب API منفصل.
- **تحقق من `validation`** لبناء قواعد تحقق محلية قبل الإرسال.
- **تعامل مع `file_errors`** في الاستجابة لإعلام المستخدم بالملفات التي فشلت.
- **عند رفع ملفات كبيرة**، يُفضل استخدام `multipart/form-data`.
- **للتطبيقات التي لا تدعم multipart** (مثل بعض بيئات الجوال)، استخدم Base64.

