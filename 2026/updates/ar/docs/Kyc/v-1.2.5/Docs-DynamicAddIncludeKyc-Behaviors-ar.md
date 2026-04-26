# توثيق سلوك `DynamicAddIncludeKyc` – تضمين بيانات KYC في استجابات API

## نظرة عامة

سلوك `DynamicAddIncludeKyc` هو امتداد ديناميكي (Behavior) يُضاف تلقائيًا إلى `Transformers` الخاصة بـ `Nano.API` (مثل `UserTransformer`, `AdminTransformer`, `DeliveryTransformer`). يقوم بحقن نقاط تضمين (Includes) جديدة تتيح استعلام بيانات التحقق (KYC) المرتبطة بالكائن المستعرض (مستخدم، طلب، فرع، مندوب توصيل) ضمن استجابة واحدة، مما يقلل عدد الطلبات ويحسن تجربة التطوير.

**الـ Includes المدعومة:**

| المفتاح | الوصف | النوع |
| :--- | :--- | :--- |
| `kyc_status` | تقييم شامل لحالة KYC للمالك، مع إمكانية تحديد مستوى المخاطرة وفئة الوثائق. | كائن |
| `kyc_status_by_category` | تقييم حالة KYC بناءً على تصنيف وثائق محدد. | كائن |
| `kyc_documents` | قائمة وثائق KYC الخاصة بالمالك مع دعم التصفح والفلترة. | مجموعة |
| `kyc_documents_count` | عدد وثائق KYC المرتبطة بالمالك. | رقم صحيح |
| `kyc_verification_summary` | ملخص سريع لحالة جميع فئات الوثائق الست مع إحصائيات. | كائن |
| `is_verifier_primary_id` | وجود وثيقة هوية أساسية صالحة (`true`/`false`). | منطقي |
| `is_verifier_secondary_id` | وجود وثيقة هوية ثانوية صالحة. | منطقي |
| `is_verifier_address` | وجود وثيقة إثبات عنوان صالحة. | منطقي |
| `is_verifier_corporate` | وجود وثيقة شركات صالحة. | منطقي |
| `is_verifier_ubo` | وجود وثيقة مالك مستفيد صالحة. | منطقي |
| `is_verifier_additional` | وجود وثيقة إضافية صالحة. | منطقي |

**ملاحظة:** العلاقات `is_verifier_secondary_id` حتى `is_verifier_additional` تُضاف إلى `secretIncludes` (لا تظهر في التوثيق التلقائي للـ API)، بينما `is_verifier_primary_id` تُضاف إلى `availableIncludes` العامة.

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

## أمثلة متقدمة للاستخدام

### 1. تضمين تقييم KYC شامل (`kyc_status`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_status HTTP/1.1
Authorization: Bearer ...
```

**معاملات اختيارية:**
| المعامل | الوصف | القيم الممكنة |
| :--- | :--- | :--- |
| `kyc_risk_level` | مستوى المخاطرة لتحديد الوثائق الإلزامية. | `low`, `medium` (افتراضي), `high` |
| `kyc_category` | فلترة حسب فئة الوثائق. | `primary_id`, `address`, `corporate`, ... |
| `kyc_document_type` | فلترة حسب نوع وثيقة محدد. | `passport`, `utility_bill`, ... |
| `kyc_include_expired` | تضمين الوثائق المنتهية. | `true` (افتراضي), `false` |
| `kyc_include_pending` | تضمين الوثائق قيد الانتظار. | `true` (افتراضي), `false` |
| `kyc_include_rejected` | تضمين الوثائق المرفوضة. | `true`, `false` (افتراضي) |

**مثال متقدم:**
```http
GET /api/v1/me?include=kyc_status&kyc_risk_level=high&kyc_category=primary_id&kyc_include_pending=false HTTP/1.1
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
            "Missing required document types: national_id"
        ]
    }
}
```

---

### 2. تضمين تقييم KYC حسب التصنيف (`kyc_status_by_category`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_status_by_category&kyc_category=address&kyc_document_type[]=utility_bill&kyc_document_type[]=bank_statement HTTP/1.1
Authorization: Bearer ...
```

**معاملات اختيارية:**

| المعامل | الوصف |
| :--- | :--- |
| `kyc_category` | **مطلوب**. تصنيف الوثائق. إذا لم يحدد، يستخدم `primary_id` افتراضيًا. |
| `kyc_document_type` | نوع وثيقة محدد أو مصفوفة أنواع. |
| `kyc_include_expired` | تضمين الوثائق المنتهية (`true`/`false`). |
| `kyc_include_pending` | تضمين الوثائق قيد الانتظار (`true`/`false`). |
| `kyc_include_rejected` | تضمين الوثائق المرفوضة (`true`/`false`). |

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

**معاملات اختيارية:**

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

### 5. ملخص حالة التحقق (`kyc_verification_summary`)

**الطلب:**
```http
GET /api/v1/me?include=kyc_verification_summary HTTP/1.1
Authorization: Bearer ...
```

**معاملات اختيارية:**
| المعامل | الوصف | الافتراضي |
| :--- | :--- | :--- |
| `kyc_verification_summary[include_pending]` | تضمين الوثائق المعلقة في الفحص. | `false` |
| `kyc_verification_summary[include_expired]` | تضمين الوثائق المنتهية. | `false` |
| `kyc_verification_summary[check_validity]` | فحص صلاحية التواريخ. | `true` |
| `kyc_verification_summary[expiring_within_days]` | فحص الوثائق التي ستنتهي خلال عدد أيام محدد. | `null` |

**مثال متقدم (فحص الوثائق التي ستنتهي خلال 30 يومًا مع تضمين المعلقة):**
```http
GET /api/v1/me?include=kyc_verification_summary&kyc_verification_summary[include_pending]=true&kyc_verification_summary[expiring_within_days]=30 HTTP/1.1
```

**هيكل الاستجابة:**
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

---

### 6. الفاحصات السريعة للفئات (Boolean Includes)

**العلاقات المتاحة:**

| المفتاح | الوصف |
| :--- | :--- |
| `is_verifier_primary_id` | هل يوجد وثيقة هوية أساسية صالحة؟ |
| `is_verifier_secondary_id` | هل يوجد وثيقة هوية ثانوية صالحة؟ |
| `is_verifier_address` | هل يوجد وثيقة إثبات عنوان صالحة؟ |
| `is_verifier_corporate` | هل يوجد وثيقة شركات صالحة؟ |
| `is_verifier_ubo` | هل يوجد وثيقة مالك مستفيد صالحة؟ |
| `is_verifier_additional` | هل يوجد وثيقة إضافية صالحة؟ |

**الطلب الأساسي:**
```http
GET /api/v1/me?include=is_verifier_primary_id,is_verifier_address HTTP/1.1
```

**معاملات اختيارية (لكل فاحصة على حدة):**
| المعامل | الوصف | الافتراضي |
| :--- | :--- | :--- |
| `is_verifier_primary_id[include_pending]` | تضمين الوثائق المعلقة. | `false` |
| `is_verifier_primary_id[include_expired]` | تضمين الوثائق المنتهية. | `false` |
| `is_verifier_primary_id[check_validity]` | فحص صلاحية التواريخ. | `true` |
| `is_verifier_primary_id[expiring_within_days]` | فحص الوثائق التي ستنتهي خلال عدد أيام. | `null` |

**مثال متقدم (فحص وثيقة هوية ستنتهي خلال 60 يومًا):**
```http
GET /api/v1/me?include=is_verifier_primary_id&is_verifier_primary_id[expiring_within_days]=60 HTTP/1.1
```

**هيكل الاستجابة:**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "is_verifier_primary_id": true,
    "is_verifier_address": false
}
```

---

### 7. سيناريو متكامل (جميع الـ Includes في طلب واحد)

**الطلب:**
```http
GET /api/v1/me?include=kyc_verification_summary,kyc_documents_count,kyc_documents&kyc_documents[per_page]=10&kyc_verification_summary[expiring_within_days]=30 HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مختصرة):**
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
    },
    "kyc_documents_count": 3,
    "kyc_documents": {
        "data": [ ... ],
        "meta": { ... }
    }
}
```

---

## ملاحظات هامة

- **الصلاحيات:** بيانات KYC تخضع لصلاحيات المستخدم. المستخدم العادي يرى بياناته فقط.
- **التخزين المؤقت:** السلوك يستخدم تخزينًا مؤقتًا متقدمًا على مستوى الطلب (`$documentsCache`) لتجنب تكرار استعلامات قاعدة البيانات عند طلب عدة فاحصات معًا.
- **المجموعات البديلة:** في `kyc_status` و `kyc_status_by_category`، يمكن استخدام `group:primary_identity` للإشارة إلى أي وثيقة من مجموعة الهوية الأساسية.
- **الأداء:** عند استخدام الفاحصات السريعة، يتم جلب جميع وثائق المالك مرة واحدة فقط لكل طلب، مما يحسن الأداء بشكل كبير.

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
