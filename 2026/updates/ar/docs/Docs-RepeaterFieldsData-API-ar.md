# توثيق متحكم `RepeaterFields` لواجهة برمجة التطبيقات (API)

## مقدمة

متحكم `RepeaterFields` هو جزء من إضافة `Nano\BasicApi`، ويوفر واجهة برمجة تطبيقات (API) للتعامل مع الحقول القابلة للتكرار (repeater fields) المدعومة في كلاس `Tss\Basic\Classes\RepeaterFieldsData`.  
يسمح هذا المتحكم بإجراء عمليات مختلفة على البيانات المدخلة لهذه الحقول مثل التطبيع، التنسيق، الاستخراج، الدمج، والتحقق.

**المسار الأساسي:**  
`/api/v1/basic/repeater-fields`

**الرؤوس المطلوبة:**  
- `Accept: application/json`  
- `Authorization: Bearer {token}` (إذا كانت المصادقة مطلوبة)

**جميع الاستجابات تكون بتنسيق JSON موحد يحتوي على الحقول التالية:**
- `code`: رمز حالة HTTP
- `status`: true للنجاح، false للفشل
- `message`: رسالة وصفية
- `data`: البيانات المُرجَعة (في حالة النجاح)
- `error`: رسالة خطأ مفصلة (في حالة الفشل)
- `errors`: مصفوفة أخطاء التحقق (اختياري)
- `debug`: معلومات إضافية للتصحيح (تظهر فقط في وضع `debug`)

---

## نقاط النهاية (Endpoints)

### 1. الحصول على قائمة الحقول المدعومة

**GET** `/api/v1/basic/repeater-fields/supported`

يعيد قائمة بأسماء الحقول التي يدعمها المتحكم.

#### مثال للطلب:
```http
GET /api/v1/basic/repeater-fields/supported HTTP/1.1
Host: example.com
Authorization: Bearer {token}
Accept: application/json
```

#### مثال للاستجابة الناجحة:
```json
{
    "code": 200,
    "status": true,
    "message": "Supported fields retrieved successfully",
    "data": ["phone", "email", "website", "properties", "links"]
}
```

---

### 2. معالجة الإدخال (Process Input)

**POST** `/api/v1/basic/repeater-fields/process`

يقوم بتطبيع المدخلات (input) لحقل معين وتحويلها إلى بنية موحدة قابلة للحفظ في قاعدة البيانات. يدخل جميع الصيغ المختلفة: نص مفرد، مصفوفة نصوص، كائن واحد، مصفوفة كائنات، خليط.

#### جسم الطلب (JSON):
| الحقل     | النوع  | الإلزام | الوصف |
|-----------|--------|---------|--------|
| field     | string | نعم     | اسم الحقل (phone, email, website, properties, links) |
| input     | mixed  | نعم     | البيانات المدخلة بأي صيغة مدعومة |

#### أمثلة:

##### مثال 1: حقل phone مع نص مفرد
**الطلب:**
```json
{
    "field": "phone",
    "input": "770529482"
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "phone_label": "",
            "phone_number": "770529482",
            "phone_type": null,
            "sort_show": null,
            "is_default": "1",
            "is_show": "1",
            "phone_note": "",
            "_group": "phone"
        }
    ]
}
```

##### مثال 2: حقل email مع مصفوفة نصوص
**الطلب:**
```json
{
    "field": "email",
    "input": ["user@example.com", "admin@site.com"]
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "email_label": "",
            "email_text": "user@example.com",
            "sort_show": null,
            "is_default": "1",
            "is_show": "1",
            "email_note": "",
            "_group": "email"
        },
        {
            "email_label": "",
            "email_text": "admin@site.com",
            "sort_show": null,
            "is_default": "1",
            "is_show": "1",
            "email_note": "",
            "_group": "email"
        }
    ]
}
```

##### مثال 3: حقل website مع كائن واحد جزئي
**الطلب:**
```json
{
    "field": "website",
    "input": {
        "website_url": "https://example.com",
        "is_default": "0"
    }
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "website_label": "",
            "website_url": "https://example.com",
            "website_type": null,
            "sort_show": null,
            "is_default": "0",
            "is_show": "1",
            "website_icon": null,
            "website_note": "",
            "_group": "website"
        }
    ]
}
```

##### مثال 4: حقل properties مع خليط (نصوص + كائنات)
**الطلب:**
```json
{
    "field": "properties",
    "input": [
        "قيمة نصية",
        {
            "title": "اللون",
            "code": "color",
            "value": "أحمر"
        }
    ]
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Input processed successfully",
    "data": [
        {
            "title": null,
            "code": null,
            "value": "قيمة نصية",
            "is_default": "1",
            "is_show": "1",
            "sort_show": null,
            "_group": "properties"
        },
        {
            "title": "اللون",
            "code": "color",
            "value": "أحمر",
            "is_default": "1",
            "is_show": "1",
            "sort_show": null,
            "_group": "properties"
        }
    ]
}
```

---

### 3. تنسيق الإخراج (Format Output)

**POST** `/api/v1/basic/repeater-fields/format`

يأخذ البيانات المخزنة (كمصفوفة) ويقوم بتنسيقها للإخراج، مع خيارات إخفاء العناصر غير الظاهرة وتحويل مسارات الوسائط إلى روابط كاملة.

#### جسم الطلب (JSON):
| الحقل         | النوع   | الإلزام | الوصف |
|---------------|---------|---------|--------|
| field         | string  | نعم     | اسم الحقل |
| data          | array   | نعم     | البيانات المخزنة (مصفوفة) |
| filter_hidden | boolean | لا      | إخفاء العناصر التي `is_show = '0'` (افتراضي false) |
| resolve_media | boolean | لا      | تحويل مسارات `website_icon` إلى روابط كاملة (افتراضي true) |

#### مثال:
**الطلب:**
```json
{
    "field": "website",
    "data": [
        {
            "website_label": "فيسبوك",
            "website_url": "https://fb.com/page",
            "website_icon": "icons/fb.png",
            "is_show": "1",
            "_group": "website"
        },
        {
            "website_label": "موقع قديم",
            "website_url": "https://oldsite.com",
            "website_icon": null,
            "is_show": "0",
            "_group": "website"
        }
    ],
    "filter_hidden": true,
    "resolve_media": true
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data formatted successfully",
    "data": [
        {
            "website_label": "فيسبوك",
            "website_url": "https://fb.com/page",
            "website_icon": "https://example.com/storage/app/media/icons/fb.png",
            "is_show": "1",
            "_group": "website"
        }
    ]
}
```

---

### 4. استخراج القيم الرئيسية (Extract Main Values)

**POST** `/api/v1/basic/repeater-fields/extract`

يستخرج القيم الأساسية من البيانات (مثل أرقام الهواتف أو عناوين البريد) ويرجعها كمصفوفة.

#### جسم الطلب (JSON):
| الحقل           | النوع   | الإلزام | الوصف |
|-----------------|---------|---------|--------|
| field           | string  | نعم     | اسم الحقل |
| data            | array   | نعم     | البيانات المخزنة |
| include_defaults| boolean | لا      | تضمين العناصر الافتراضية فقط (`is_default = '1'`) (افتراضي false) |

#### مثال:
**الطلب:**
```json
{
    "field": "phone",
    "data": [
        {"phone_number": "770529482", "is_default": "1"},
        {"phone_number": "780505400", "is_default": "0"},
        {"phone_number": "790505500", "is_default": "1"}
    ],
    "include_defaults": true
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Main values extracted successfully",
    "data": ["770529482", "790505500"]
}
```

---

### 5. دمج البيانات (Merge Data)

**POST** `/api/v1/basic/repeater-fields/merge`

يدمج بيانات جديدة مع بيانات قديمة. يمكن إما استبدال القديم بالكامل أو إضافة الجديد إليه.

#### جسم الطلب (JSON):
| الحقل    | النوع   | الإلزام | الوصف |
|----------|---------|---------|--------|
| field    | string  | نعم     | اسم الحقل |
| old_data | array   | نعم     | البيانات القديمة (مصفوفة) |
| new_data | mixed   | نعم     | البيانات الجديدة (يمكن أن تكون بأي صيغة) |
| replace  | boolean | لا      | إذا كان true يتم استبدال القديم بالجديد، وإلا يُضاف الجديد (افتراضي false) |

#### مثال 1: إضافة جديدة (دمج)
**الطلب:**
```json
{
    "field": "email",
    "old_data": [
        {"email_text": "old@site.com", "is_default": "1"}
    ],
    "new_data": ["new@site.com"]
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data merged successfully",
    "data": [
        {"email_text": "old@site.com", "is_default": "1", "_group": "email", ...},
        {"email_text": "new@site.com", "is_default": "1", "_group": "email", ...}
    ]
}
```

#### مثال 2: استبدال كامل
**الطلب:**
```json
{
    "field": "phone",
    "old_data": [
        {"phone_number": "111111", "is_default": "1"}
    ],
    "new_data": ["222222", "333333"],
    "replace": true
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data merged successfully",
    "data": [
        {"phone_number": "222222", "is_default": "1", "_group": "phone", ...},
        {"phone_number": "333333", "is_default": "1", "_group": "phone", ...}
    ]
}
```

---

### 6. التحقق من صحة البيانات (Validate)

**POST** `/api/v1/basic/repeater-fields/validate`

يتحقق من صحة البيانات المدخلة وفقًا لتعريفات الحقل (الحقول المطلوبة، قيم dropdown المسموحة). يُعيد `valid` (boolean) وقائمة بالأخطاء إن وجدت.

#### جسم الطلب (JSON):
| الحقل | النوع  | الإلزام | الوصف |
|-------|--------|---------|--------|
| field | string | نعم     | اسم الحقل |
| data  | array  | نعم     | البيانات المراد التحقق منها |

#### مثال:
**الطلب:**
```json
{
    "field": "links",
    "data": [
        {"title": "رابط جيد", "url": "https://good.com"},
        {"url": "https://missing-title.com"}  // title مطلوب
    ]
}
```
**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "Data validated successfully",
    "data": {
        "valid": false,
        "errors": [
            "Field 'title' is required at index 1."
        ]
    }
}
```

---

### 7. تشغيل الاختبارات (Run Tests) – للمطورين فقط

**GET** `/api/v1/basic/repeater-fields/tests`

يقوم بتشغيل مجموعة اختبارات على جميع الحقول المدعومة للتحقق من عمل الكلاس بشكل صحيح. هذه النقطة محمية: إما أن يكون `app.debug = true` أو أن المستخدم لديه صلاحية `developer.access`.

#### مثال للطلب:
```http
GET /api/v1/basic/repeater-fields/tests HTTP/1.1
Authorization: Bearer {token}
Accept: application/json
```

#### مثال للاستجابة (مختصرة):
```json
{
    "code": 200,
    "status": true,
    "message": "Tests completed successfully",
    "data": {
        "summary": {
            "total": 45,
            "passed": 45,
            "failed": 0
        },
        "results": {
            "phone": [
                {
                    "case": "نص مفرد",
                    "input": "770529482",
                    "output": [...],
                    "passed": true
                },
                ...
            ],
            ...
        }
    }
}
```

---

## رموز الأخطاء الشائعة

| الرمز | المعنى |
|-------|--------|
| 400   | طلب غير صحيح (missing field, unsupported field, etc.) |
| 401   | غير مصادق (unauthorized) |
| 403   | غير مصرح (forbidden) |
| 404   | غير موجود (not found) |
| 422   | خطأ في التحقق (validation error) |
| 500   | خطأ داخلي في الخادم |

---

## ملاحظات إضافية

- جميع نقاط النهاية (عدا `supported` و `tests`) تتطلب مصادقة عبر `oauth-users` middleware (كما هو موضح في ملف `routes.php`).
- في بيئة الإنتاج (`app.debug = false`) يتم إخفاء التفاصيل الحساسة للأخطاء (مثل استعلامات SQL) واستبدالها برسائل عامة.
- نقطة `tests` متاحة فقط في وضع التصحيح أو للمستخدمين الذين يمتلكون صلاحية `developer.access`.

---

بهذا نكون قد غطينا جميع وظائف المتحكم `RepeaterFields` مع أمثلة عملية لكل نقطة نهاية. يمكن استخدام هذا التوثيق كمرجع للمطورين الذين يرغبون في دمج هذه الخدمة في تطبيقاتهم.