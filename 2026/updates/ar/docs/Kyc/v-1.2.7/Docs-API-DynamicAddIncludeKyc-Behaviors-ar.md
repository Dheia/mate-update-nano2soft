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
