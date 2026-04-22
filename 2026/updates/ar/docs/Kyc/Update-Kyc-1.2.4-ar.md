## 2026-04-22 - 2026-04-23

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.4**

### ملخص التحديثات

بعد تحسين أداء دوال التحقق السريع في الإصدار 1.2.3 وإضافة التخزين المؤقت والأقفال الذرية، نقدم في الإصدار 1.2.4 تحسينات شاملة على سلوك `DynamicAddIncludeKyc` – المسؤول عن حقن علاقات KYC داخل `Transformers` الخاصة بـ `Nano.API`. يركز هذا الإصدار على:

- **تحسين جذري للأداء** عند طلب عدة علاقات KYC معاً من خلال تخزين مؤقت على مستوى الطلب.
- **إضافة 7 علاقات جديدة** تشمل ملخص حالة التحقق (`kyc_verification_summary`) وفحص سريع لكل فئة من فئات الوثائق الست (`is_verifier_primary_id`، `is_verifier_address`، ...).
- **دعم خيارات متقدمة** مثل `expiring_within_days` للكشف عن الوثائق التي على وشك الانتهاء.
- **توثيق محدث وشامل** يغطي جميع العلاقات الجديدة مع أمثلة عملية متكاملة.

هذا الإصدار يجعل دمج بيانات KYC في استجابات API أكثر قوة ومرونة، ويوفر للمطورين أدوات دقيقة للتحكم في سلوك الفحص حسب احتياجات التطبيق.

---

### الإصدار 1.2.4 – تحسينات شاملة لسلوك `DynamicAddIncludeKyc` وعلاقات KYC جديدة

#### أهداف الإصدار

- تحسين أداء سلوك `DynamicAddIncludeKyc` من خلال مشاركة استعلامات قاعدة البيانات عبر تخزين مؤقت على مستوى الطلب (`$documentsCache` و `$categoryCheckCache`).
- إضافة علاقة `kyc_verification_summary` التي تقدم ملخصاً سريعاً لحالة جميع فئات الوثائق الست مع إحصائيات (عدد المنتهية، المعلقة، المرفوضة).
- إضافة فاحصات سريعة (Boolean Includes) لكل فئة وثائق: `is_verifier_primary_id`، `is_verifier_secondary_id`، `is_verifier_address`، `is_verifier_corporate`، `is_verifier_ubo`، `is_verifier_additional`.
- دعم خيارات متقدمة لجميع الفاحصات السريعة وملخص التحقق، تشمل `include_pending`، `include_expired`، `check_validity`، و `expiring_within_days` (للكشف عن الوثائق التي ستنتهي خلال عدد أيام محدد).
- تحسين آلية تسجيل العلاقات (`includes`) داخل `__construct` بحيث يتم إضافتها إلى `secretIncludes` فقط إذا كان `Transformer` الأصلي يدعمها، وإلا تُضاف إلى `availableIncludes` العامة.
- تحديث شامل لملفي التوثيق `Docs-DynamicAddIncludeKyc-Behaviors-ar.md` و `Docs-API-Documentation-ar.md` لتغطية كافة الميزات الجديدة مع أمثلة عملية.

#### الميزات الجديدة

##### 1. تخزين مؤقت على مستوى الطلب (Request-Level Caching)

في السابق، كان طلب عدة علاقات KYC في استدعاء API واحد (مثل `?include=is_verifier_primary_id,is_verifier_address,kyc_verification_summary`) يؤدي إلى تنفيذ استعلامات قاعدة بيانات منفصلة ومتكررة لنفس المالك. في هذا الإصدار، تمت إضافة آليتين للتخزين المؤقت على مستوى الطلب:

| الخاصية | الوصف |
| :--- | :--- |
| `$documentsCache` | تخزن جميع وثائق المالك المسترجعة من قاعدة البيانات. يتم جلبها مرة واحدة فقط لكل طلب، بغض النظر عن عدد العلاقات المطلوبة. |
| `$categoryCheckCache` | تخزن نتائج فحص الفئات (`true`/`false`) مع مراعاة الخيارات الممررة. يمنع إعادة حساب نفس الفئة بنفس الخيارات خلال الطلب الواحد. |

**النتيجة:** تحسن كبير في زمن الاستجابة وتقليل الحمل على قاعدة البيانات عند استخدام عدة علاقات KYC معاً.

##### 2. علاقة `kyc_verification_summary` – ملخص حالة التحقق

توفر هذه العلاقة كائناً يحتوي على حالة جميع فئات الوثائق الست (`primary_id`، `secondary_id`، `address`، `corporate`، `ubo`، `additional`) بالإضافة إلى إحصائيات سريعة:

| الحقل | الوصف |
| :--- | :--- |
| `primary_id` | `true` إذا كان لدى المالك وثيقة هوية أساسية صالحة. |
| `secondary_id` | `true` إذا كان لدى المالك وثيقة هوية ثانوية صالحة. |
| `address` | `true` إذا كان لدى المالك وثيقة إثبات عنوان صالحة. |
| `corporate` | `true` إذا كان لدى المالك وثيقة شركات صالحة. |
| `ubo` | `true` إذا كان لدى المالك وثيقة مالك مستفيد صالحة. |
| `additional` | `true` إذا كان لدى المالك وثيقة إضافية صالحة. |
| `total_verified` | عدد الوثائق المعتمدة. |
| `total_pending` | عدد الوثائق قيد الانتظار. |
| `total_expired` | عدد الوثائق المنتهية. |
| `total_rejected` | عدد الوثائق المرفوضة. |

**خيارات مدعومة:** `include_pending`، `include_expired`، `check_validity`، `expiring_within_days`.

##### 3. فاحصات سريعة لكل فئة (Boolean Includes)

تمت إضافة 6 علاقات جديدة من النوع المنطقي (`true`/`false`) تتيح التحقق السريع من وجود وثيقة صالحة ضمن فئة محددة:

| العلاقة | الوصف |
| :--- | :--- |
| `is_verifier_primary_id` | هل يوجد وثيقة هوية أساسية صالحة؟ |
| `is_verifier_secondary_id` | هل يوجد وثيقة هوية ثانوية صالحة؟ |
| `is_verifier_address` | هل يوجد وثيقة إثبات عنوان صالحة؟ |
| `is_verifier_corporate` | هل يوجد وثيقة شركات صالحة؟ |
| `is_verifier_ubo` | هل يوجد وثيقة مالك مستفيد صالحة؟ |
| `is_verifier_additional` | هل يوجد وثيقة إضافية صالحة؟ |

جميع هذه العلاقات تدعم نفس الخيارات المتقدمة (`include_pending`، `include_expired`، `check_validity`، `expiring_within_days`).

> **ملاحظة:** العلاقات `is_verifier_secondary_id` حتى `is_verifier_additional` تُضاف إلى `secretIncludes` (لا تظهر في التوثيق التلقائي للـ API ولكن يمكن استخدامها)، بينما `is_verifier_primary_id` و `kyc_verification_summary` علاقات عامة تُضاف إلى `availableIncludes`.

##### 4. دعم خيار `expiring_within_days`

يمكن للمطورين الآن تمرير خيار `expiring_within_days` (عدد صحيح موجب) إلى أي من الفاحصات السريعة أو ملخص التحقق. عند تحديده، سيعيد الفحص `true` إذا كانت هناك وثيقة **صالحة حالياً** ولكنها ستنتهي صلاحيتها خلال العدد المحدد من الأيام. هذا مفيد لتنبيه المستخدمين لتجديد وثائقهم قبل انتهائها.

**مثال:** التحقق مما إذا كان لدى المستخدم هوية ستنتهي خلال 30 يوماً:
```
?include=is_verifier_primary_id&is_verifier_primary_id[expiring_within_days]=30
```

##### 5. تحسين آلية تسجيل العلاقات (`__construct`)

تم تحسين دالة البناء (`__construct`) لسلوك `DynamicAddIncludeKyc` لتكون أكثر ذكاءً وتوافقاً مع جميع أنواع `Transformers`:

- يتم التحقق من وجود الخاصية `secretIncludes` في الكلاس الأصلي (`property_exists`).
- إذا كانت الخاصية موجودة، يتم إضافة العلاقات الجديدة إلى `secretIncludes` (باستثناء العلاقات العامة التي تضاف أيضاً إلى `availableIncludes`).
- إذا لم تكن الخاصية موجودة (أي أن الـ `Transformer` لا يدعم العلاقات السرية)، يتم إضافة جميع العلاقات إلى `availableIncludes` مباشرة.

هذا يضمن سلوكاً متوقعاً في جميع البيئات دون أخطاء.

##### 6. تحديث التوثيق

تم تحديث ملفي التوثيق الرئيسيين ليعكسا كافة الميزات الجديدة:

| الملف | التحديثات |
| :--- | :--- |
| `Docs-DynamicAddIncludeKyc-Behaviors-ar.md` | إعادة كتابة كاملة لتشمل شرحاً مفصلاً لجميع العلاقات الجديدة، آلية التخزين المؤقت على مستوى الطلب، الخيارات المتقدمة، وأمثلة عملية متكاملة. |
| `Docs-API-Documentation-ar.md` | إضافة قسم مخصص بعنوان "تضمين بيانات KYC ضمن استجابات API" يشرح العلاقات المتاحة، كيفية استخدامها، الخيارات المدعومة، مع أمثلة عملية. |

---

### أمثلة عملية

#### 1. الحصول على ملخص سريع لحالة KYC مع فحص الوثائق التي ستنتهي خلال 30 يوماً

**الطلب:**
```http
GET /api/v1/me?include=kyc_verification_summary&kyc_verification_summary[include_pending]=true&kyc_verification_summary[expiring_within_days]=30 HTTP/1.1
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

#### 2. استخدام عدة فاحصات سريعة معاً في طلب واحد

**الطلب:**
```http
GET /api/v1/me?include=is_verifier_primary_id,is_verifier_address,is_verifier_corporate HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة (مقتطف):**
```json
{
    "id": 24,
    "name": "أحمد محمد",
    "is_verifier_primary_id": true,
    "is_verifier_address": false,
    "is_verifier_corporate": false
}
```

> **ملاحظة الأداء:** في هذا المثال، يتم جلب وثائق المستخدم من قاعدة البيانات **مرة واحدة فقط** ويتم تخزينها في `$documentsCache`، ثم يتم فحص كل فئة من نفس مجموعة البيانات المخزنة مؤقتاً.

#### 3. فحص وثيقة هوية أساسية مع تضمين الوثائق قيد الانتظار

**الطلب:**
```http
GET /api/v1/me?include=is_verifier_primary_id&is_verifier_primary_id[include_pending]=true HTTP/1.1
Authorization: Bearer ...
```

في هذه الحالة، سيعيد `is_verifier_primary_id` القيمة `true` حتى لو كانت الوثيقة لا تزال في حالة `pending` (قيد المراجعة).

---

### ملخص الإصدارات (1.2.3 – 1.2.4)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.3 | نقل دوال التحقق السريع إلى سمة `HasValidKycDocuments`، تخزين مؤقت متعدد المستويات، أقفال ذرية، مسح تلقائي للكاش. |
| 1.2.4 | تحسين أداء سلوك `DynamicAddIncludeKyc` عبر تخزين مؤقت على مستوى الطلب، إضافة 7 علاقات جديدة (`kyc_verification_summary` و 6 فاحصات فئات)، دعم `expiring_within_days`، تحسين آلية تسجيل العلاقات، تحديث شامل للتوثيق. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `behaviors/DynamicAddIncludeKyc.php` بالنسخة الجديدة.
   - تأكد من وجود السمة `HasValidKycDocuments.php` (تمت إضافتها في الإصدار 1.2.3).

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **تحديث تعريفات API** (لا توجد تغييرات في نقاط النهاية).

4. **اختبار العلاقات الجديدة**:
   - جرب طلب `?include=kyc_verification_summary` على `/api/v1/me` للتأكد من عمل الملخص بشكل صحيح.
   - جرب طلب عدة فاحصات سريعة معاً ولاحظ تحسن زمن الاستجابة.

5. **مراجعة التوثيق الجديد**:
   - اطلع على قسم "تضمين بيانات KYC ضمن استجابات API" في `Docs-API-Documentation-ar.md` لفهم كافة الخيارات المتاحة.

---

### الخاتمة

مع الإصدار 1.2.4، نكون قد أكملنا دورة تحسين أداء وكفاءة نظام KYC من جميع الزوايا: من دوال التحقق الخلفية (الإصدار 1.2.3) إلى واجهة API وتضمين البيانات (الإصدار 1.2.4). أصبح بإمكان المطورين الآن:

- استخدام علاقات KYC بكثافة داخل استجابات API دون القلق بشأن أداء قاعدة البيانات.
- الحصول على ملخصات سريعة أو فحوصات دقيقة حسب الحاجة.
- تخصيص سلوك الفحص بدقة عبر خيارات متقدمة مثل `expiring_within_days`.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
