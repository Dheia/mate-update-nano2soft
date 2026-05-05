### 16. جلب تصنيفات الوثائق المدعومة (قائمة منظمة)

**الطريقة:** `GET`  
**المسار:** `/document-categories-list` أو `/document-categories-list/{id}`

يوفر نقطتي نهاية لجلب تصنيفات الوثائق (فئاتها) ببيانات منظمة تدعم الترجمة وخيارات تضمين متقدمة.

#### معاملات المسار (اختياري)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `id` | `string` | لا | كود التصنيف (مثال: `primary_id`). إذا تم تمريره، يتم جلب تفاصيل هذا التصنيف فقط. |

#### معاملات الاستعلام (Query Parameters)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `locale` | `string` | لا | اللغة المطلوبة (`ar` أو `en`). الافتراضي لغة النظام. |
| `include_types` | `bool` | لا | تضمين أنواع الوثائق التي تنتمي إلى كل تصنيف. الافتراضي `false`. |
| `include_details` | `bool` | لا | تضمين تفاصيل كل نوع وثيقة (مثل الوزن، الحقول المطلوبة، إلخ). لا يعمل إلا مع `include_types=1`. |
| `include_fields` | `bool` | لا | تضمين مخطط الحقول (`fields`) لكل نوع وثيقة. |
| `include_rules` | `bool` | لا | تضمين قواعد التحقق (`rules`) لكل نوع وثيقة. |
| `category_filter` | `string` | لا | فلترة التصنيفات حسب كود محدد (مثال: `primary_id`). يمكن استخدامه عند جلب القائمة. |

#### مثال 1: جلب جميع التصنيفات (بدون تفاصيل)

**الطلب:**
```http
GET /api/v1/kyc/document-categories-list HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب بيانات التصنيفات بنجاح",
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
        {
            "code": "address",
            "name_ar": "إثبات عنوان",
            "name_en": "Proof of Address",
            "name": "إثبات عنوان"
        }
        // ... باقي التصنيفات
    ]
}
```

#### مثال 2: جلب تصنيف محدد مع أنواعه وتفاصيلها

**الطلب:**
```http
GET /api/v1/kyc/document-categories-list/primary_id?include_types=1&include_details=1 HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "code": "primary_id",
        "name_ar": "هوية أساسية",
        "name_en": "Primary ID",
        "name": "هوية أساسية",
        "types": [
            {
                "code": "passport",
                "name_ar": "جواز سفر",
                "name_en": "Passport",
                "name": "جواز سفر",
                "description": "جواز سفر ساري المفعول يحتوي على صورة شخصية...",
                "details": {
                    "verification_weight": 100,
                    "has_expiry": true,
                    "required_fields": ["document_front", "document_number", ...]
                }
            },
            {
                "code": "national_id",
                "name_ar": "بطاقة هوية وطنية",
                "name_en": "National ID Card",
                "name": "بطاقة هوية وطنية",
                "description": "بطاقة الهوية الوطنية الصادرة عن الجهات الرسمية.",
                "details": {
                    "verification_weight": 95,
                    "has_expiry": true,
                    "requires_back_side": true
                }
            }
            // ... باقي الأنواع
        ]
    }
}
```

#### مثال 3: جلب جميع التصنيفات مع تضمين الأنواع والحقول والقواعد

**الطلب:**
```http
GET /api/v1/kyc/document-categories-list?include_types=1&include_fields=1&include_rules=1 HTTP/1.1
Authorization: Bearer ...
```

---

### 17. جلب أنواع الوثائق المدعومة (قائمة منظمة)

**الطريقة:** `GET`  
**المسار:** `/document-types-list` أو `/document-types-list/{id}`

يوفر نقطتي نهاية لجلب أنواع الوثائق الفردية ببيانات منظمة تدعم الترجمة وخيارات تضمين متقدمة.

#### معاملات المسار (اختياري)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `id` | `string` | لا | كود نوع الوثيقة (مثال: `passport`). إذا تم تمريره، يتم جلب تفاصيل هذا النوع فقط. |

#### معاملات الاستعلام (Query Parameters)

| المعامل | النوع | مطلوب | الوصف |
|---------|------|-------|-------|
| `locale` | `string` | لا | اللغة المطلوبة (`ar` أو `en`). الافتراضي لغة النظام. |
| `include_category` | `bool` | لا | تضمين معلومات التصنيف الأب (مثل `primary_id`) لكل نوع. الافتراضي `false`. |
| `include_details` | `bool` | لا | تضمين تفاصيل النوع (الوزن، الحقول المطلوبة، الملفات المسموحة...). الافتراضي `false`. |
| `include_fields` | `bool` | لا | تضمين مخطط الحقول (`fields`) الكامل للنوع. |
| `include_rules` | `bool` | لا | تضمين قواعد التحقق (`rules`) الخاصة بكل حقل. |
| `category_filter` | `string` | لا | فلترة الأنواع حسب تصنيف محدد (مثال: `primary_id`). يمكن استخدامه عند جلب القائمة. |

#### مثال 1: جلب جميع أنواع الوثائق (بدون تفاصيل)

**الطلب:**
```http
GET /api/v1/kyc/document-types-list HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب بيانات أنواع الوثائق بنجاح",
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
        // ... باقي الأنواع
    ]
}
```

#### مثال 2: جلب نوع وثيقة محدد مع التصنيف والتفاصيل والحقول والقواعد

**الطلب:**
```http
GET /api/v1/kyc/document-types-list/passport?include_category=1&include_details=1&include_fields=1&include_rules=1 HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "code": "passport",
        "name_ar": "جواز سفر",
        "name_en": "Passport",
        "name": "جواز سفر",
        "description": "جواز سفر ساري المفعول يحتوي على صورة شخصية...",
        "category": {
            "code": "primary_id",
            "name_ar": "هوية أساسية",
            "name_en": "Primary ID",
            "name": "هوية أساسية"
        },
        "details": {
            "category": "primary_id",
            "category_name": "هوية أساسية",
            "has_expiry": true,
            "verification_weight": 100,
            "required_fields": ["document_front", "document_number", "issue_date", ...],
            "allowed_mime_types": ["image/jpeg", "image/png", "application/pdf"],
            "max_file_size_kb": 10240,
            "requires_back_side": false,
            "requires_selfie_match": true
        },
        "fields": {
            "document_front": {
                "label": "document_front",
                "type": "file",
                "required": true,
                "validation": { "mimes": ["jpeg","jpg","png","pdf"], "max_size_kb": 10240 }
            },
            "document_number": {
                "label": "document_number",
                "type": "text",
                "required": true,
                "validation": { "max_length": 50 }
            }
            // ... باقي الحقول
        },
        "rules": {
            "document_front": ["required", "file", "mimes:jpeg,jpg,png,pdf", "max:10240"],
            "document_number": ["required", "max:50"]
            // ... باقي القواعد
        }
    }
}
```

#### مثال 3: جلب أنواع تصنيف "إثبات عنوان" فقط مع التصنيف الأب

**الطلب:**
```http
GET /api/v1/kyc/document-types-list?category_filter=address&include_category=1 HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "code": 200,
    "status": true,
    "data": [
        {
            "code": "utility_bill",
            "name": "فاتورة مرافق",
            "category": {
                "code": "address",
                "name": "إثبات عنوان"
            }
        },
        {
            "code": "bank_statement",
            "name": "كشف حساب بنكي",
            "category": {
                "code": "address",
                "name": "إثبات عنوان"
            }
        }
        // ...
    ]
}
```
