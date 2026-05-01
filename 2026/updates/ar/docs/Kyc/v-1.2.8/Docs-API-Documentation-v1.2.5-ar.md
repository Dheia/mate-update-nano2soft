## توثيق واجهة برمجة التطبيقات (API) للتحقق من الهوية – الإصدار 1.2.0

**الإصدار:** 1.2.0  
**المسار الأساسي:** `/api/v1/kyc`  
**المصادقة:** OAuth 2.0 (توكن الوصول في الهيدر `Authorization: Bearer <token>`)

---

## نظرة عامة

توفر واجهة برمجة تطبيقات (API) `Nano3.Kyc` نقاط نهاية آمنة ومرنة لإدارة وثائق التحقق من الهوية (KYC) في تطبيقات نانوسوفت. يمكن للمطورين من خلالها:

- إنشاء واستعراض وتحديث وحذف الوثائق.
- رفع ملفات الهوية (صور جواز، هوية، توقيع) وربطها بالوثائق.
- اعتماد أو رفض الوثائق.
- تقييم حالة KYC لمستخدم أو جهة معينة (نسبة الاكتمال، الوثائق المفقودة).
- جلب أنواع الوثائق المدعومة وحقولها الديناميكية.
- تشغيل مجموعة اختبارات شاملة (في بيئة التطوير فقط).

**الميزات الرئيسية:**
- دعم أكثر من 28 نوع وثيقة (جواز سفر، هوية وطنية، فاتورة، رخصة تجارية، إلخ).
- رفع ملفات بصيغ متعددة (multipart/form-data أو base64) مع تكامل `Nano.FileUpload`.
- حماية تلقائية للوثائق المعتمدة من تعديل المرفقات.
- تقييم فوري لحالة KYC مع توصيات ذكية.
- استجابات موحدة وهيكل بيانات متسق.

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
    "model": {},
    "input_data": {},
    "process_data": {},
    "debug": null
}
```

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `code` | `int` | كود الحالة (200 للنجاح، 400 للخطأ). |
| `status` | `bool` | `true` للنجاح، `false` للفشل. |
| `message` | `string` | رسالة واضحة للمستخدم (قابلة للترجمة). |
| `error` | `string\|null` | تفاصيل الخطأ التقنية (في حالة الفشل). |
| `errors` | `array\|null` | مصفوفة أخطاء التحقق (مثل صحة المدخلات). |
| `data` | `mixed` | البيانات الأساسية المطلوبة (مثل تفاصيل الوثيقة). |
| `model` | `object\|null` | كائن الوثيقة الكامل (في عمليات الإنشاء والتحديث). |
| `input_data` | `array` | نسخة من البيانات التي أرسلها المستخدم (للتتبع). |
| `process_data` | `array` | معلومات داخلية عن المعالجة (مثل أخطاء الرفع الجزئية). |
| `debug` | `array\|null` | معلومات التصحيح (تظهر فقط عند تفعيل `app.debug`). |

---

## المصادقة والأمان

جميع نقاط النهاية محمية بـ **OAuth 2.0**. يجب تضمين توكن الوصول في هيدر الطلب:

```
Authorization: Bearer <your_access_token>
```

- إذا لم يتم توفير التوكن: استجابة `401 Unauthorized`.
- إذا كان التوكن غير صالح أو منتهي: استجابة `401 Unauthorized`.

**ملاحظة:** بعض العمليات تتطلب صلاحيات إضافية (مثل اعتماد الوثائق) والتي يتم التحقق منها عبر `FileUploadUserManager` وصلاحيات `backend`. المستخدمون العاديون (`frontend`) يمكنهم فقط إدارة وثائقهم الخاصة.

---

## نقاط النهاية (Endpoints)

### 1. عرض قائمة الوثائق

**الطريقة:** `GET`  
**المسار:** `/documents`

#### معاملات الاستعلام (Query Parameters)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `document_type` | `string` | لا | كود نوع الوثيقة (مثال: `passport`). |
| `document_category` | `string` | لا | فئة الوثيقة (مثال: `primary_id`). |
| `status` | `string` | لا | حالة الوثيقة (`pending`, `verified`, `rejected`). |
| `owner_id` | `int` | لا | معرف المالك (يُستخدم من قبل `backend` فقط). |
| `owner_type` | `string` | لا | نوع المالك (يُستخدم من قبل `backend` فقط). |
| `is_verified` | `bool` | لا | تصفية حسب حالة التحقق (`true`/`false`). |
| `q` | `string` | لا | بحث نصي في رقم الوثيقة، الاسم، اسم الشركة، الكود. |
| `page` | `int` | لا | رقم الصفحة (الافتراضي 1). |
| `per_page` | `int` | لا | عدد العناصر في الصفحة (الافتراضي 15). |
| `orderBy` | `string` | لا | ترتيب حسب حقل معين (الافتراضي `created_at`). |
| `orderDirection` | `string` | لا | اتجاه الترتيب (`asc` أو `desc`، الافتراضي `desc`). |

#### مثال طلب

```http
GET /api/v1/kyc/documents?document_type=passport&status=pending&per_page=10 HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب السجلات بنجاح.",
    "data": [
        {
            "id": 45,
            "code": "KYC20250417001",
            "document_type": "passport",
            "document_type_name": "جواز سفر",
            "document_number": "P12345678",
            "full_name": "أحمد محمد",
            "status": "pending",
            "is_verified": false,
            "issue_date": "2020-01-01",
            "expiry_date": "2030-01-01",
            "created_at": "2026-04-17 10:30:00",
            "owner": {
                "id": 24,
                "name": "أحمد محمد",
                "type": "backend"
            }
        }
    ],
    "meta": {
        "pagination": {
            "total": 1,
            "per_page": 10,
            "current_page": 1,
            "last_page": 1
        }
    }
}
```

**ملاحظة:** المستخدم العادي (`frontend`) لا يمكنه تمرير `owner_id` أو `owner_type`؛ يتم فلترة النتائج تلقائياً لعرض وثائقه فقط.

---

### 2. عرض تفاصيل وثيقة محددة

**الطريقة:** `GET`  
**المسار:** `/documents/{id}`

#### معاملات المسار

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `id` | `int\|string` | نعم | معرف الوثيقة أو `code` الخاص بها. |

#### مثال طلب

```http
GET /api/v1/kyc/documents/45 HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب البيانات بنجاح",
    "data": {
        "id": 45,
        "code": "KYC20250417001",
        "document_type": "passport",
        "document_type_name": "جواز سفر",
        "document_category": "primary_id",
        "document_category_name": "هوية أساسية",
        "document_number": "P12345678",
        "issuing_authority": "وزارة الداخلية",
        "issue_date": "2020-01-01",
        "expiry_date": "2030-01-01",
        "full_name": "أحمد محمد",
        "date_of_birth": "1990-05-15",
        "nationality": "يمني",
        "country_of_issue": "اليمن",
        "religion": "مسلم",
        "address_line1": "شارع 45",
        "address_city": "صنعاء",
        "address_country": "اليمن",
        "company_name": null,
        "owner_id": 24,
        "owner_type": "Backend\\Models\\User",
        "is_verified": false,
        "verification_score": null,
        "status": "pending",
        "is_active": true,
        "is_published": true,
        "document_front": {
            "original": "https://account.now-ye.com/storage/app/uploads/public/.../original.jpg",
            "thumb": "https://account.now-ye.com/storage/app/uploads/public/.../thumb.jpg"
        },
        "fields_data": {
            "passport_type": "P"
        },
        "created_at": "2026-04-17 10:30:00",
        "updated_at": "2026-04-17 10:30:00"
    }
}
```

**❌ استجابة خطأ (وثيقة غير موجودة):**

```json
{
    "code": 404,
    "status": false,
    "message": "Document not found.",
    "error": "Document not found."
}
```

---

### 3. إنشاء وثيقة جديدة

**الطريقة:** `POST`  
**المسار:** `/documents`

#### معاملات الطلب (JSON أو multipart)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `document_type` | `string` | نعم | كود نوع الوثيقة (مثال: `passport`). |
| `owner_id` | `int` | نعم* | معرف المالك (*إلا إذا كان المستخدم الحالي هو المالك). |
| `owner_type` | `string` | نعم* | نوع المالك (*إلا إذا كان المستخدم الحالي هو المالك). |
| `fields` | `object` | لا | بيانات الحقول الديناميكية حسب نوع الوثيقة (مثال: `{"full_name": "...", "document_number": "..."}`). |
| `document_front` | `file\|base64` | لا | صورة وجه الوثيقة (أو base64). |
| `document_back` | `file\|base64` | لا | صورة ظهر الوثيقة (أو base64). |
| `signature_image` | `file\|base64` | لا | صورة التوقيع (أو base64). |
| `image` | `file\|base64` | لا | صورة شخصية إضافية (أو base64). |
| `files[]` | `file[]\|base64[]` | لا | ملفات إضافية متعددة (أو base64). |
| `status` | `string` | لا | حالة الوثيقة (الافتراضي `pending`). |
| `is_published` | `bool` | لا | نشر الوثيقة (الافتراضي `false`). |
| `is_active` | `bool` | لا | تفعيل الوثيقة (الافتراضي `true`). |

> **ملاحظة:** عند إرسال ملفات base64، استخدم نفس اسم الحقل مع لاحقة `_base64` (مثال: `document_front_base64`).

#### مثال طلب (multipart)

```http
POST /api/v1/kyc/documents HTTP/1.1
Authorization: Bearer ...
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="document_type"

passport
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[full_name]"

أحمد محمد علي
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[document_number]"

P12345678
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[issue_date]"

2020-01-01
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[expiry_date]"

2030-01-01
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[country_of_issue]"

YE
------WebKitFormBoundary
Content-Disposition: form-data; name="fields[nationality]"

YE
------WebKitFormBoundary
Content-Disposition: form-data; name="document_front"; filename="passport.jpg"
Content-Type: image/jpeg

(بيانات الملف الثنائية)
------WebKitFormBoundary--
```

#### مثال طلب (JSON مع base64)

```json
POST /api/v1/kyc/documents HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "document_type": "passport",
    "fields": {
        "full_name": "أحمد محمد علي",
        "document_number": "P12345678",
        "issue_date": "2020-01-01",
        "expiry_date": "2030-01-01",
        "country_of_issue": "YE",
        "nationality": "YE"
    },
    "document_front_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
}
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم إنشاء الوثيقة بنجاح.",
    "data": {
        "id": 46,
        "document_type": "passport",
        "document_number": "P12345678",
        "full_name": "أحمد محمد علي",
        "status": "pending",
        "document_front": {
            "original": "https://account.now-ye.com/storage/app/uploads/public/.../original.jpg"
        }
    },
    "model": { ... },
    "process_data": {
        "file_errors": []
    }
}
```

**⚠️ استجابة نجاح جزئي (فشل رفع بعض الملفات):**

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

**❌ استجابة خطأ (نوع وثيقة غير صالح):**

```json
{
    "code": 400,
    "status": false,
    "message": "حدث خطأ أثناء إنشاء الوثيقة.",
    "error": "نوع الوثيقة غير صالح."
}
```

---

### 4. تحديث وثيقة

**الطريقة:** `PUT`  
**المسار:** `/documents/{id}`

#### معاملات الطلب

نفس معاملات الإنشاء، ولكن جميعها اختيارية. يمكن تحديث الحقول الديناميكية عبر `fields` والملفات بنفس أسماء الحقول.

**قاعدة هامة:** إذا كانت الوثيقة معتمدة (`is_verified = true`)، **لا يمكن تعديل أو استبدال أي ملف مرتبط بها** (مثل `document_front`). ستتمكن فقط من تحديث الحقول النصية غير المرتبطة بالملفات.

#### مثال طلب (تحديث الاسم فقط)

```json
PUT /api/v1/kyc/documents/46 HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "fields": {
        "full_name": "أحمد محمد عبدالله"
    }
}
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم تحديث الوثيقة بنجاح.",
    "data": {
        "id": 46,
        "full_name": "أحمد محمد عبدالله",
        "updated_at": "2026-04-17 12:15:00"
    }
}
```

**❌ استجابة خطأ (محاولة تغيير ملف لوثيقة معتمدة):**

```json
{
    "code": 400,
    "status": false,
    "message": "حدث خطأ أثناء تحديث الوثيقة.",
    "error": "لا يمكن تغيير الملف document_front لوثيقة معتمدة."
}
```

---

### 5. حذف وثيقة (Soft Delete)

**الطريقة:** `DELETE`  
**المسار:** `/documents/{id}`

#### مثال طلب

```http
DELETE /api/v1/kyc/documents/46 HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم حذف الوثيقة بنجاح."
}
```

**❌ استجابة خطأ (غير موجودة):**

```json
{
    "code": 400,
    "status": false,
    "message": "حدث خطأ أثناء حذف الوثيقة.",
    "error": "الوثيقة المطلوبة غير موجودة."
}
```

---

### 6. استعادة وثيقة محذوفة

**الطريقة:** `POST`  
**المسار:** `/documents/restore/{id?}`

#### مثال طلب

```http
POST /api/v1/kyc/documents/restore/45 HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم استعادة الوثيقة بنجاح.",
    "data": { ... }
}
```

---

### 7. اعتماد (تحقق) وثيقة

**الطريقة:** `POST`  
**المسار:** `/documents/verify/{id?}`

#### معاملات الطلب (JSON)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `verification_score` | `int` | لا | درجة الموثوقية (0-100، الافتراضي 100). |

#### مثال طلب

```json
POST /api/v1/kyc/documents/verify/45 HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "verification_score": 95
}
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم اعتماد الوثيقة بنجاح.",
    "data": {
        "id": 45,
        "is_verified": true,
        "verified_at": "2026-04-17 14:20:00",
        "verification_score": 95,
        "status": "verified"
    }
}
```

**ملاحظة:** بمجرد اعتماد الوثيقة، لا يمكن تعديل ملفاتها.

---

### 8. رفض وثيقة

**الطريقة:** `POST`  
**المسار:** `/documents/reject/{id?}`

#### معاملات الطلب (JSON)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `reason` | `string` | لا | سبب الرفض. |

#### مثال طلب

```json
POST /api/v1/kyc/documents/reject/45 HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "reason": "الصورة غير واضحة"
}
```

#### الاستجابة

**✅ استجابة نجاح:**

```json
{
    "code": 200,
    "status": true,
    "message": "تم رفض الوثيقة بنجاح.",
    "data": {
        "id": 45,
        "status": "rejected",
        "metadata": {
            "rejection_reason": "الصورة غير واضحة"
        }
    }
}
```

---

### 9. جلب أنواع الوثائق المدعومة

**الطريقة:** `GET`  
**المسار:** `/document-types` أو `/document-types/{id}`

#### معاملات المسار (اختياري)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `id` | `string` | لا | كود نوع الوثيقة للحصول على تفاصيله فقط. |

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `grouped` | `bool` | لا | تجميع النتائج حسب الفئة (الافتراضي `false`). |
| `category` | `string` | لا | فلترة حسب فئة محددة (مثال: `primary_id`). |
| `locale` | `string` | لا | اللغة (`ar` أو `en`). |

#### مثال طلب (جميع الأنواع مسطحة)

```http
GET /api/v1/kyc/document-types HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب البيانات بنجاح",
    "data": {
        "passport": "جواز سفر",
        "national_id": "بطاقة هوية وطنية",
        "drivers_license": "رخصة قيادة",
        "utility_bill": "فاتورة مرافق",
        ...
    }
}
```

#### مثال طلب (مجمعة حسب الفئة)

```http
GET /api/v1/kyc/document-types?grouped=true HTTP/1.1
```

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب البيانات بنجاح",
    "data": {
        "هوية أساسية": {
            "passport": "جواز سفر",
            "national_id": "بطاقة هوية وطنية",
            ...
        },
        "إثبات عنوان": {
            "utility_bill": "فاتورة مرافق",
            ...
        }
    }
}
```

#### مثال طلب (تفاصيل نوع محدد)

```http
GET /api/v1/kyc/document-types/passport HTTP/1.1
```

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب البيانات بنجاح",
    "data": {
        "code": "passport",
        "name": "جواز سفر",
        "description": "جواز سفر ساري المفعول يحتوي على صورة شخصية وبيانات المسافر.",
        "category": "primary_id",
        "category_name": "هوية أساسية",
        "has_expiry": true,
        "verification_weight": 100,
        "required_fields": ["document_front", "document_number", "issue_date", "expiry_date", "country_of_issue"],
        "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
        "max_file_size_kb": 10240,
        "requires_back_side": false,
        "requires_selfie_match": true,
        "fields": {
            "document_front": { "type": "file", "required": true, "label": "وجه الوثيقة" },
            "document_number": { "type": "text", "required": true, "label": "رقم الوثيقة" },
            ...
        }
    }
}
```

---

### 10. جلب حقول نوع وثيقة (لبناء نماذج ديناميكية)

**الطريقة:** `GET`  
**المسار:** `/document-fields/{type}`

#### معاملات المسار

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `type` | `string` | نعم | كود نوع الوثيقة (مثال: `passport`). |

#### مثال طلب

```http
GET /api/v1/kyc/document-fields/passport HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب البيانات بنجاح",
    "data": {
        "type": "passport",
        "type_name": "جواز سفر",
        "fields": {
            "document_front": {
                "label": "وجه الوثيقة",
                "type": "file",
                "required": true,
                "validation": { "mimes": ["jpeg","jpg","png","pdf"], "max_size_kb": 10240 }
            },
            "document_number": {
                "label": "رقم الوثيقة",
                "type": "text",
                "required": true,
                "validation": { "max_length": 50 }
            },
            "full_name": { ... },
            "issue_date": { ... },
            "expiry_date": { ... },
            "country_of_issue": { ... },
            "nationality": { ... },
            "date_of_birth": { ... },
            "religion": { ... },
            "passport_type": {
                "label": "نوع الجواز",
                "type": "select",
                "required": false,
                "options": { "P": "Personal", "D": "Diplomatic", "O": "Official" }
            }
        },
        "rules": {
            "document_front": ["required", "file", "mimes:jpeg,jpg,png,pdf", "max:10240"],
            "document_number": ["required", "string", "max:50"],
            "expiry_date": ["required", "date", "after_or_equal:today"]
        },
        "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
        "max_file_size_kb": 10240,
        "requires_back_side": false,
        "requires_selfie_match": true
    }
}
```

---

### 11. إحصائيات سريعة عن الوثائق

**الطريقة:** `GET`  
**المسار:** `/documents/stats`

#### مثال طلب

```http
GET /api/v1/kyc/documents/stats HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب الإحصائيات بنجاح",
    "data": {
        "total": 156,
        "active": 142,
        "published": 130,
        "public": 120,
        "default": 5,
        "by_type": {
            "passport": 45,
            "national_id": 38,
            "utility_bill": 22,
            ...
        },
        "by_ref_type": { ... }
    }
}
```

---

### 12. التحقق من وجود تحديثات جديدة (للتخزين المؤقت)

**الطريقة:** `GET`  
**المسار:** `/documents/activelystats`

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `date` | `string` | لا | تاريخ للمقارنة (افتراضي `now`). |

#### مثال طلب

```http
GET /api/v1/kyc/documents/activelystats?date=2026-04-17 10:00:00 HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة

```json
{
    "code": 200,
    "status": true,
    "data": {
        "activity_stats": true,
        "check_date": "2026-04-17 10:00:00",
        "last_updated": "2026-04-17 14:30:00"
    }
}
```

- `activity_stats = true` يعني وجود تحديثات منذ التاريخ المطلوب.

---

### 13. تقييم حالة KYC لمالك معين

**الطريقة:** `GET` أو `POST`  
**المسار:** `/assess`

#### معاملات الطلب (JSON أو Query)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `owner_type` | `string` | لا* | نوع المالك (إذا كان `backend` ولم يحدد، يستخدم نفسه). |
| `owner_id` | `int` | لا* | معرف المالك. |
| `risk_level` | `string` | لا | مستوى المخاطرة (`low`, `medium`, `high`، الافتراضي `medium`). |
| `category` | `string` | لا | فلترة حسب فئة الوثائق (`primary_id`, `address`, ...). |
| `document_type` | `string\|array` | لا | فلترة حسب نوع وثيقة محدد. |
| `include_expired` | `bool` | لا | تضمين الوثائق المنتهية (الافتراضي `true`). |
| `include_pending` | `bool` | لا | تضمين الوثائق قيد الانتظار (الافتراضي `true`). |
| `include_rejected` | `bool` | لا | تضمين الوثائق المرفوضة (الافتراضي `false`). |

> *إذا كان المستخدم `frontend`، يتم تقييم نفسه تلقائياً.

#### مثال طلب (تقييم مستخدم backend لنفسه بمخاطرة عالية)

```http
GET /api/v1/kyc/assess?risk_level=high HTTP/1.1
Authorization: Bearer ...
```

#### مثال طلب (تقييم مستخدم frontend معين بواسطة backend)

```json
POST /api/v1/kyc/assess HTTP/1.1
Authorization: Bearer ...
Content-Type: application/json

{
    "owner_type": "RainLab\\User\\Models\\User",
    "owner_id": 532,
    "risk_level": "high",
    "include_expired": true
}
```

#### الاستجابة

```json
{
    "code": 200,
    "status": true,
    "message": "تم تقييم حالة KYC بنجاح.",
    "data": {
        "is_compliant": false,
        "completion_percentage": 50.0,
        "overall_score": 45,
        "total_documents_required": 2,
        "verified_documents_count": 1,
        "pending_documents_count": 0,
        "expired_documents_count": 0,
        "rejected_documents_count": 0,
        "verified_documents": [
            {
                "id": 45,
                "type": "passport",
                "document_number": "P12345678",
                "expiry_date": "2030-01-01",
                "verification_score": 100
            }
        ],
        "pending_documents": [],
        "expired_documents": [],
        "rejected_documents": [],
        "missing_required_types": ["national_id"],
        "recommendations": [
            "Missing required document types: national_id",
            "Recommended additional documents: drivers_license, utility_bill"
        ]
    }
}
```

---

### 14. تقييم حالة KYC حسب تصنيف الوثائق

**الطريقة:** `GET` أو `POST`  
**المسار:** `/assess/{category}`

#### معاملات المسار

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `category` | `string` | لا | تصنيف الوثائق (`primary_id`, `address`, `corporate`... الافتراضي `primary_id`). |

#### معاملات الطلب (نفس `assess` بالإضافة إلى)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `document_type` | `string\|array` | لا | نوع وثيقة محدد أو مصفوفة أنواع ضمن التصنيف. |

#### مثال طلب (تقييم فئة العنوان)

```http
GET /api/v1/kyc/assess/address?include_expired=false HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة (مشابهة لـ `assess` لكن `total_documents_required` يشمل فقط أنواع التصنيف المحدد)

```json
{
    "code": 200,
    "status": true,
    "message": "تم تقييم حالة KYC حسب التصنيف بنجاح.",
    "data": {
        "is_compliant": true,
        "completion_percentage": 100.0,
        "overall_score": 60,
        "total_documents_required": 1,
        "verified_documents_count": 1,
        "verified_documents": [
            {
                "id": 50,
                "type": "utility_bill",
                "document_number": "INV-2026-001",
                "issue_date": "2026-03-01",
                "verification_score": 60
            }
        ],
        "missing_required_types": [],
        "recommendations": []
    }
}
```

---

### 15. تشغيل اختبارات الحزمة (للتطوير فقط)

**الطريقة:** `GET`  
**المسار:** `/tests`

> **تنبيه هام:** هذه النقطة متاحة فقط عندما يكون `app.debug = true` أو البيئة `local`. لا يمكن استخدامها في الإنتاج.

#### معاملات الاستعلام

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `test_version` | `string` | لا | `v1` أو `v2` (الافتراضي `v2`). |
| `filter` | `string` | لا | `all`, `passed`, `failed` (الافتراضي `all`). |

#### مثال طلب

```http
GET /api/v1/kyc/tests?test_version=v2&filter=passed HTTP/1.1
Authorization: Bearer ...
```

#### الاستجابة (مثال مختصر)

```json
{
    "code": 200,
    "status": true,
    "message": "KYC Test suite completed",
    "data": [
        {
            "code": 200,
            "status": true,
            "test_code": "DOCTYPE_REG_001",
            "name": "تسجيل أنواع الوثائق في DocumentType",
            "description": "يتحقق من أن DocumentType يحتوي على الأنواع الأساسية...",
            "message": "Test passed",
            "data": {
                "total_types": 28,
                "has_passport": true,
                "has_national_id": true
            }
        },
        ...
    ],
    "meta": {
        "summary": {
            "total": 18,
            "passed": 18,
            "failed": 0,
            "success_rate": 100,
            "test_version": "v2",
            "filter_applied": "passed"
        }
    },
    "process_data": {
        "timestamp": "2026-04-17 15:00:00"
    }
}
```

---

## تضمين بيانات KYC ضمن استجابات API (علاقات DynamicAddIncludeKyc)

يتكامل نظام KYC مع بنية `Nano.API` لتوفير إمكانية تضمين بيانات التحقق مباشرةً ضمن استجابات نقاط النهاية الأخرى (مثل معلومات المستخدم، الطلب، أو المندوب). يتم ذلك عبر تمرير مفتاح `include` في الـ Query String مع أحد الأسماء المدرجة أدناه.

> **ملاحظة هامة:** هذه الميزة متاحة فقط في `Transformers` التي تم تمديدها بواسطة الإضافة (مثل `UserTransformer` و `AdminTransformer`). ليست نقاط نهاية مستقلة، بل تُضاف إلى استجابات الكيانات التي تدعمها.

### قائمة العلاقات المتاحة

| اسم العلاقة (`include`) | الوصف | النوع | متى تستخدم؟ |
| :--- | :--- | :--- | :--- |
| `kyc_status` | تقييم شامل لحالة KYC للمالك، مع إمكانية تحديد مستوى المخاطرة وفئة الوثائق. | `object` | عندما تحتاج معرفة حالة التحقق الكاملة (مكتمل/غير مكتمل، النسبة المئوية، الوثائق المفقودة، التوصيات). |
| `kyc_status_by_category` | تقييم حالة KYC بناءً على تصنيف وثائق محدد (مثل `primary_id` أو `address`). | `object` | عندما تريد التحقق من حالة فئة معينة فقط (مثلاً: هل أكمل المستخدم وثائق الهوية الأساسية؟). |
| `kyc_documents` | قائمة وثائق KYC الخاصة بالمالك مع دعم التصفح والفلترة. | `collection` | لعرض قائمة وثائق المستخدم داخل صفحة ملفه الشخصي. |
| `kyc_documents_count` | عدد وثائق KYC المرتبطة بالمالك. | `int` | لعرض ملخص سريع (مثلاً: "لديك 3 وثائق"). |
| `kyc_verification_summary` | ملخص سريع لحالة جميع فئات الوثائق الست مع إحصائيات (عدد المنتهية، المعلقة...). | `object` | لبناء لوحة تحكم حالة KYC للمستخدم. |
| `is_verifier_primary_id` | هل يوجد وثيقة هوية أساسية صالحة؟ (`true`/`false`). | `bool` | لفحص شرط سريع (مثلاً: "هل يمكنه إتمام عملية تتطلب هوية أساسية؟"). |
| `is_verifier_secondary_id` | هل يوجد وثيقة هوية ثانوية صالحة؟ | `bool` | للتحقق من وجود وثيقة داعمة (بطاقة ناخب، هوية عسكرية). |
| `is_verifier_address` | هل يوجد وثيقة إثبات عنوان صالحة؟ | `bool` | للتحقق من عنوان المستخدم قبل الشحن أو التسليم. |
| `is_verifier_corporate` | هل يوجد وثيقة شركات صالحة؟ | `bool` | للتحقق من حالة توثيق حساب تجاري/شركة. |
| `is_verifier_ubo` | هل يوجد وثيقة مالك مستفيد صالحة؟ | `bool` | للتحقق من الامتثال التنظيمي (مكافحة غسل الأموال). |
| `is_verifier_additional` | هل يوجد وثيقة إضافية صالحة؟ | `bool` | للتحقق من وجود وثائق تكميلية (مثل توقيع، سيلفي). |

> **ملاحظة:** العلاقات `is_verifier_secondary_id` حتى `is_verifier_additional` تُضاف إلى `secretIncludes` (لا تظهر في توثيق الـ API التلقائي ولكن يمكن استخدامها)، بينما `is_verifier_primary_id` و `kyc_documents_count` و `kyc_verification_summary` علاقات عامة.

---

### كيفية الاستخدام

لطلب أي من هذه العلاقات، أضف `?include=اسم_العلاقة` إلى طلب API الخاص بالكيان (مثل `/api/v1/me` أو `/api/v1/users/24`). يمكن طلب عدة علاقات معاً بفصلها بفاصلة `,`.

#### 1. مثال: تضمين تقييم KYC شامل (`kyc_status`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_status&kyc_risk_level=high HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "email": "ahmed@example.com",
    "kyc_status": {
        "is_compliant": false,
        "completion_percentage": 50.0,
        "overall_score": 45,
        "total_documents_required": 2,
        "verified_documents_count": 1,
        "missing_required_types": ["national_id"],
        "recommendations": ["Missing required document types: national_id"]
    }
}
```

#### 2. مثال: ملخص سريع لحالة جميع الفئات (`kyc_verification_summary`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_verification_summary&kyc_verification_summary[include_pending]=true HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "kyc_verification_summary": {
        "primary_id": true,
        "secondary_id": false,
        "address": true,
        "corporate": false,
        "ubo": false,
        "additional": false,
        "total_verified": 2,
        "total_pending": 1,
        "total_expired": 0,
        "total_rejected": 0
    }
}
```

#### 3. مثال: الفاحصات السريعة (Boolean Includes)

للتحقق من وجود وثيقة هوية أساسية وإثبات عنوان:

```http
GET /api/v1/me?include=is_verifier_primary_id,is_verifier_address HTTP/1.1
```

**الاستجابة (مقتطف):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "is_verifier_primary_id": true,
    "is_verifier_address": false
}
```

**معاملات اختيارية للفاحصات السريعة:**

يمكنك تخصيص سلوك الفحص بتمرير معاملات عبر الـ Query String:

```
?include=is_verifier_primary_id&is_verifier_primary_id[include_pending]=true&is_verifier_primary_id[expiring_within_days]=30
```

| المعامل | الوصف | الافتراضي |
| :--- | :--- | :--- |
| `[include_pending]` | تضمين الوثائق قيد الانتظار. | `false` |
| `[include_expired]` | تضمين الوثائق المنتهية. | `false` |
| `[check_validity]` | فحص صلاحية التواريخ. | `true` |
| `[expiring_within_days]` | فحص الوثائق التي ستنتهي خلال عدد أيام محدد. | `null` |

#### 4. مثال: تضمين قائمة الوثائق مع Pagination (`kyc_documents`)

```http
GET /api/v1/me?include=kyc_documents&kyc_documents[per_page]=5&kyc_documents[status]=verified HTTP/1.1
```

**الاستجابة (مقتطف):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "kyc_documents": {
        "data": [
            {
                "id": 45,
                "document_type": "passport",
                "document_number": "P12345678",
                "status": "verified",
                "document_front": { "thumb": "https://..." }
            }
        ],
        "meta": {
            "pagination": {
                "total": 2,
                "per_page": 5,
                "current_page": 1,
                "last_page": 1
            }
        }
    }
}
```

#### 5. مثال: دمج عدة علاقات في طلب واحد

```http
GET /api/v1/me?include=kyc_verification_summary,kyc_documents_count,is_verifier_primary_id HTTP/1.1
```

---

### خيارات متقدمة

يمكن تمرير خيارات إضافية لكل علاقة عبر معاملات `query` باسم العلاقة متبوعة بأقواس مربعة `[]`:

| العلاقة | الخيارات المدعومة |
| :--- | :--- |
| `kyc_status` | `risk_level`, `category`, `document_type`, `include_expired`, `include_pending`, `include_rejected` |
| `kyc_status_by_category` | `category` (مطلوب), `document_type`, `include_expired`, `include_pending`, `include_rejected` |
| `kyc_documents` | `per_page`, `page`, `document_type`, `status`, `paginate` |
| `kyc_verification_summary` | `include_pending`, `include_expired`, `check_validity`, `expiring_within_days` |
| `is_verifier_*` | `include_pending`, `include_expired`, `check_validity`, `expiring_within_days` |

---

### اعتبارات الأمان

- بيانات KYC المعروضة تخضع لصلاحيات المستخدم. المستخدم العادي يرى بياناته فقط، بينما مستخدم `backend` يمكنه رؤية بيانات أي كائن إذا كانت الصلاحيات تسمح.
- لا يمكن تجاوز نطاق المالك عبر هذه العلاقات؛ حيث يتم تحديد المالك تلقائيًا من الكائن المستعرض (مثلاً: `User` في `UserTransformer`).

---

## أكواد الأخطاء الشائعة

| كود HTTP | الوصف |
|----------|-------|
| 400 | طلب غير صالح (بيانات مفقودة أو غير صالحة). |
| 401 | غير مصرح (التوكن مفقود أو غير صالح). |
| 403 | صلاحية ممنوعة (المستخدم لا يملك صلاحية الوصول للمورد). |
| 404 | المورد غير موجود (وثيقة، نوع وثيقة، إلخ). |
| 422 | فشل التحقق من صحة البيانات المدخلة. |
| 500 | خطأ داخلي في الخادم. |

---

## أفضل الممارسات للتكامل

1. **استخدم `temp_session_key` للنماذج غير المحفوظة**: إذا كنت تبني واجهة متعددة الخطوات، استخدم مفتاح الجلسة المؤقت لرفع الملفات أولاً ثم اربطها بعد إنشاء الوثيقة.
2. **تحقق من صلاحية المستخدم قبل عرض أزرار التعديل**: استخدم نقاط النهاية `assess` لمعرفة حالة KYC وعرض المطالبات المناسبة.
3. **تعامل مع أخطاء رفع الملفات الجزئية**: في `process_data.file_errors` بعد الإنشاء/التحديث، لعرض رسائل دقيقة للمستخدم.
4. **لا تحاول تعديل ملفات الوثائق المعتمدة**: تحقق من `is_verified` قبل عرض خيارات تحرير الملفات.
5. **استخدم `document-fields/{type}` لبناء نماذج ديناميكية**: لتوليد حقول الإدخال وقواعد التحقق تلقائياً حسب نوع الوثيقة المختار.
6. **فعّل `app.debug` في بيئة التطوير فقط**: للحصول على تفاصيل الأخطاء في `debug`.
7. **راقب `activity_stats`**: لتحديث الكاش في تطبيقات العميل عند وجود تغييرات جديدة.

---


## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
