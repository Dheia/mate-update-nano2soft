# توثيق سلوك `DynamicAddIncludeKyc` – تضمين بيانات KYC في استجابات API

## نظرة عامة

سلوك `DynamicAddIncludeKyc` هو جزء من إضافة `Nano3.Kyc`، يقوم تلقائيًا بحقن نقاط تضمين (Includes) جديدة في `Transformers` الخاصة بـ `Nano.API` (مثل `UserTransformer`, `AdminTransformer`, `DeliveryTransformer`). تتيح هذه النقاط للمطورين إثراء استجابات API ببيانات KYC المرتبطة بالكائن المستعرض (مستخدم، طلب، مندوب توصيل) دون الحاجة إلى طلبات منفصلة.

**الـ Includes المدعومة:**

| المفتاح | الوصف |
| :--- | :--- |
| `kyc_status` | تقييم شامل لحالة KYC للمالك، مع إمكانية تحديد مستوى المخاطرة وفئة الوثائق. |
| `kyc_status_by_category` | تقييم حالة KYC بناءً على تصنيف الوثائق (`category`) مع إمكانية تحديد أنواع محددة. |
| `kyc_documents` | قائمة وثائق KYC الخاصة بالمالك مع دعم pagination والفلترة. |
| `kyc_documents_count` | عدد وثائق KYC المرتبطة بالمالك. |

**كيف يعمل؟**
عند إضافة أحد هذه المفاتيح إلى `include` في طلب API (مثلاً `?include=kyc_status`)، يقوم السلوك باستدعاء الدوال المناظرة في `Manager` (`assessKycStatus`, `assessKycStatusByCategory`, `getDocumentRecords`) ويعيد البيانات منسقة ضمن استجابة الكائن الأصلي.

---

## تثبيت السلوك وتفعيله

يتم تفعيل السلوك تلقائيًا عند تثبيت إضافة `Nano3.Kyc`. في `Plugin.php`، يتم استدعاء `extendTransformer()` الذي يضيف السلوك إلى `Transformers` التالية:

- `UserTransformer` (مستخدمي الموقع)
- `AdminTransformer` (مستخدمي لوحة التحكم)
- `DeliveryTransformer` (مندوبي التوصيل)
- `ParentTransformer` (أولياء الأمور)
- `StudentTransformer` (الطلاب)
- `DepartmentTransformer` (الفروع والأقسام)

لا يحتاج المطور إلى أي إعدادات إضافية. بمجرد تثبيت الإضافة، تصبح هذه الـ includes متاحة في نقاط النهاية التي تستخدم الـ `Transformers` المذكورة (مثل `/api/v1/me`, `/api/v1/users/{id}`).

---

## أمثلة الاستخدام في طلبات API

### 1. تضمين تقييم KYC شامل (`kyc_status`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_status HTTP/1.1
Authorization: Bearer ...
```

**معاملات اختيارية:**
يمكن تمرير خيارات إضافية لتخصيص التقييم عبر query parameters:

| المعامل | الوصف | القيم الممكنة |
| :--- | :--- | :--- |
| `kyc_risk_level` | مستوى المخاطرة لتحديد الوثائق الإلزامية. | `low`, `medium` (افتراضي), `high` |
| `kyc_category` | فلترة حسب فئة الوثائق. | `primary_id`, `address`, `corporate`, ... |
| `kyc_document_type` | فلترة حسب نوع وثيقة محدد. | `passport`, `utility_bill`, ... |
| `kyc_include_expired` | تضمين الوثائق المنتهية. | `true` (افتراضي), `false` |
| `kyc_include_pending` | تضمين الوثائق قيد الانتظار. | `true` (افتراضي), `false` |
| `kyc_include_rejected` | تضمين الوثائق المرفوضة. | `true`, `false` (افتراضي) |

**مثال مع خيارات:**
```http
GET /api/v1/me?include=kyc_status&kyc_risk_level=high&kyc_category=primary_id HTTP/1.1
Authorization: Bearer ...
```

**هيكل الاستجابة:**
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
        "pending_documents_count": 1,
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
        "pending_documents": [
            {
                "id": 46,
                "type": "national_id",
                "document_number": "ID987654"
            }
        ],
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

### 2. تضمين تقييم KYC حسب التصنيف (`kyc_status_by_category`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_status_by_category&kyc_category=primary_id&kyc_document_type[]=passport&kyc_document_type[]=national_id HTTP/1.1
Authorization: Bearer ...
```

**معاملات اختيارية:**

| المعامل | الوصف |
| :--- | :--- |
| `kyc_category` | **مطلوب**. تصنيف الوثائق (مثال: `primary_id`, `address`). إذا لم يحدد، يستخدم `primary_id` افتراضيًا. |
| `kyc_document_type` | نوع وثيقة محدد أو مصفوفة أنواع (مثال: `passport` أو `[]=passport&[]=national_id`). |
| `kyc_include_expired` | تضمين الوثائق المنتهية (`true`/`false`). |
| `kyc_include_pending` | تضمين الوثائق قيد الانتظار (`true`/`false`). |
| `kyc_include_rejected` | تضمين الوثائق المرفوضة (`true`/`false`). |

**مثال أبسط (بدون مصفوفة):**
```http
GET /api/v1/me?include=kyc_status_by_category&kyc_category=address&kyc_document_type=utility_bill HTTP/1.1
```

**هيكل الاستجابة:**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "kyc_status_by_category": {
        "is_compliant": true,
        "completion_percentage": 100,
        "overall_score": 60,
        "total_documents_required": 1,
        "verified_documents_count": 1,
        "pending_documents_count": 0,
        "expired_documents_count": 0,
        "rejected_documents_count": 0,
        "verified_documents": [
            {
                "id": 50,
                "type": "utility_bill",
                "document_number": "INV-2026-001",
                "issue_date": "2026-03-01",
                "verification_score": 60
            }
        ],
        "pending_documents": [],
        "expired_documents": [],
        "rejected_documents": [],
        "missing_required_types": [],
        "recommendations": []
    }
}
```

---

### 3. تضمين قائمة وثائق KYC (`kyc_documents`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_documents HTTP/1.1
Authorization: Bearer ...
```

**معاملات اختيارية (للتخصيص والـ pagination):**

| المعامل | الوصف | الافتراضي |
| :--- | :--- | :--- |
| `kyc_documents[per_page]` | عدد الوثائق في الصفحة. | 15 |
| `kyc_documents[page]` | رقم الصفحة. | 1 |
| `kyc_documents[document_type]` | فلترة حسب نوع الوثيقة. | - |
| `kyc_documents[status]` | فلترة حسب الحالة (`pending`, `verified`, `rejected`). | - |
| `kyc_documents[paginate]` | تفعيل pagination (`true`/`false`). | `true` |

**مثال مع Pagination وفلترة:**
```http
GET /api/v1/me?include=kyc_documents&kyc_documents[per_page]=5&kyc_documents[status]=verified HTTP/1.1
```

**هيكل الاستجابة:**
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
                "full_name": "أحمد محمد",
                "status": "verified",
                "issue_date": "2020-01-01",
                "expiry_date": "2030-01-01",
                "is_verified": true,
                "document_front": {
                    "original": "https://.../original.jpg",
                    "thumb": "https://.../thumb.jpg"
                }
            },
            {
                "id": 50,
                "document_type": "utility_bill",
                "document_number": "INV-2026-001",
                "status": "verified",
                "issue_date": "2026-03-01",
                "is_verified": true
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

---

### 4. تضمين عدد وثائق KYC (`kyc_documents_count`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_documents_count HTTP/1.1
Authorization: Bearer ...
```

**هيكل الاستجابة:**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "kyc_documents_count": 3
}
```

---

### 5. دمج عدة Includes في طلب واحد

يمكن طلب أكثر من include في نفس الاستدعاء:

```http
GET /api/v1/me?include=kyc_status,kyc_documents_count,kyc_documents&kyc_documents[per_page]=10 HTTP/1.1
```

**الاستجابة (مختصرة):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "kyc_status": { ... },
    "kyc_documents_count": 3,
    "kyc_documents": { ... }
}
```

---

## ملاحظات هامة

- **الصلاحيات:** بيانات KYC المعروضة تخضع لصلاحيات المستخدم. المستخدم العادي يرى بياناته فقط، بينما مستخدم `backend` يمكنه رؤية بيانات أي كائن إذا كانت الصلاحيات تسمح.
- **التخزين المؤقت:** استجابات API التي تحتوي على `include` قد تستفيد من التخزين المؤقت الخاص بـ `Nano.API`. يمكن التحكم في ذلك عبر إعدادات `api_enable_cache`.
- **الأداء:** استخدام `kyc_documents` مع pagination كبير أو بدون فلترة قد يؤثر على الأداء. يُنصح دائمًا باستخدام `per_page` مناسب وفلترة حسب الحالة (`status=verified`) عند الحاجة.
- **التوافق:** السلوك يعمل مع أي `Transformer` تم تمديده في `Plugin::extendTransformer()`. لإضافة دعم لأنواع جديدة، يمكن تعديل `Plugin.php`.

---


## التوثيق الإضافي

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
