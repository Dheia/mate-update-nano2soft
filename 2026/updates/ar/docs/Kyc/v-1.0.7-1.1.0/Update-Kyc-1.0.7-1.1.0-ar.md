## 2026-04-20

**تحديثات إضافة `Nano3.Kyc` – الإصدارات 1.0.7 إلى 1.1.0**

### ملخص التحديثات

بعد الإطلاق الرسمي للإضافة في الإصدارات السابقة، واصلنا تطوير `Nano3.Kyc` لإضافة ميزات متقدمة تمكن المطورين من تقييم حالة KYC للمستخدمين والجهات المختلفة بسهولة ومرونة. تضمنت التحديثات في الإصدارات من 1.0.7 إلى 1.1.0:

- **إضافة دوال تقييم KYC شاملة** في `Manager` (`assessKycStatus` و `assessKycStatusByCategory`).
- **تطوير سلوك `DynamicAddIncludeKyc`** لحقن بيانات KYC مباشرة في استجابات API عبر `include`.
- **توسيع دعم أنواع المستخدمين** في `DocumentType` ليشمل مستخدمي `backend` و `frontend` كأفراد.
- **تحسين آلية تحديد الوثائق الإلزامية** بناءً على التصنيف (`category`) ونوع الوثيقة (`document_type`).

---

### الإصدار 1.0.7 – إضافة دوال تقييم حالة KYC في `Manager`

#### أهداف الإصدار

- توفير آلية متكاملة لتقييم حالة KYC لمالك معين (فرد/شركة).
- حساب نسبة الاكتمال، الوثائق المتحقق منها، المفقودة، المنتهية، والدرجة الإجمالية.
- دعم فلترة حسب الفئة (`category`) ونوع الوثيقة (`document_type`) ومستوى المخاطرة (`risk_level`).

#### الميزات الجديدة

##### 1. دالة `assessKycStatus` في `Manager`

تمت إضافة دالة `Manager::assessKycStatus` التي تقوم بتقييم شامل لحالة KYC لمالك معين.

**توقيع الدالة:**
```php
public static function assessKycStatus($owner, ?string $ownerType = null, ?int $ownerId = null, array $options = []): array
```

**الخيارات المدعومة:**
| الخيار | الوصف | الافتراضي |
| :--- | :--- | :--- |
| `category` | فلترة حسب فئة وثيقة محددة (`primary_id`, `address`, `corporate`, ...). | `null` |
| `document_type` | فلترة حسب نوع وثيقة محدد (`passport`, `utility_bill`, ...). | `null` |
| `risk_level` | مستوى المخاطرة لتحديد المتطلبات الإلزامية (`low`, `medium`, `high`). | `medium` |
| `include_expired` | تضمين الوثائق المنتهية في النتائج. | `true` |
| `include_pending` | تضمين الوثائق قيد الانتظار. | `true` |
| `include_rejected` | تضمين الوثائق المرفوضة. | `false` |

**هيكل الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "data": {
        "is_compliant": false,
        "completion_percentage": 0,
        "overall_score": 0,
        "total_documents_required": 0,
        "verified_documents_count": 0,
        "pending_documents_count": 0,
        "expired_documents_count": 0,
        "rejected_documents_count": 0,
        "verified_documents": [],
        "pending_documents": [],
        "expired_documents": [],
        "rejected_documents": [],
        "missing_required_types": [],
        "recommendations": []
    }
}
```

##### 2. آلية تحديد الوثائق الإلزامية والموصى بها

تستخدم الدالة دوال `DocumentType` التالية:
- `getMandatoryTypesForParty($ownerType, $riskLevel)`: لتحديد أنواع الوثائق الإلزامية بناءً على نوع المالك ومستوى المخاطرة.
- `getRecommendedTypes($ownerType, $riskLevel)`: لتحديد أنواع الوثائق الموصى بها.

##### 3. التحقق من صلاحية الوثائق

تقوم الدالة بالتحقق من صلاحية كل وثيقة متحقق منها (`verified`) باستخدام:
- `DocumentType::hasExpiry()` و `DocumentType::isDocumentAcceptable()` للوثائق ذات تاريخ انتهاء.
- `DocumentType::getMaxAgeDays()` و `DocumentType::isDocumentAcceptable()` للوثائق محدودة العمر.

#### الفوائد

- تقييم موحد وشامل لحالة KYC يمكن استخدامه في عمليات اتخاذ القرار (مثل السماح بإجراء معاملات).
- مرونة عالية في تخصيص التقييم حسب الفئة أو نوع الوثيقة أو مستوى المخاطرة.
- توصيات تلقائية بالوثائق المفقودة أو التي تحتاج تجديد.

---

### الإصدار 1.0.8 – سلوك `DynamicAddIncludeKyc` لحقن بيانات KYC في API

#### أهداف الإصدار

- تمكين استجابات API من تضمين بيانات KYC المرتبطة بالكائن (مستخدم، طلب، منتج) بسهولة عبر `include`.
- توفير آلية مرنة لحقن دوال `include` في أي `Transformer` دون تعديل الكود الأصلي.
- دعم التقييم (`kyc_status`) وقائمة الوثائق (`kyc_documents`) وعدد الوثائق (`kyc_documents_count`).

#### الميزات الجديدة

##### 1. سلوك `DynamicAddIncludeKyc`

تم إنشاء كلاس `DynamicAddIncludeKyc` (`Nano3\Kyc\Behaviors\DynamicAddIncludeKyc`) الذي يقوم بحقن الدوال التالية في أي `Transformer`:

| الدالة | الوصف |
| :--- | :--- |
| `includeKycStatus` | تقييم حالة KYC للمالك (تستخدم `Manager::assessKycStatus`). |
| `includeKycDocuments` | قائمة وثائق المالك مع دعم pagination. |
| `includeKycDocumentsCount` | عدد وثائق المالك. |

**طريقة الاستخدام في طلب API:**
```
GET /api/v1/me?include=kyc_status,kyc_documents,kyc_documents_count
```

##### 2. دعم خيارات متقدمة عبر query parameters

يمكن تمرير خيارات إضافية لكل `include`:

```
?include=kyc_status&kyc_status[risk_level]=high&kyc_status[category]=primary_id
?include=kyc_documents&kyc_documents[per_page]=20&kyc_documents[status]=verified
```

##### 3. حقن السلوك في Transformers متعددة

تم تحديث `Plugin::boot()` ليشمل حقن السلوك في Transformers التالية:
- `UserTransformer` (مستخدمي الموقع)
- `AdminTransformer` (مستخدمي لوحة التحكم)
- `DeliveryTransformer` (مندوبي التوصيل)
- `ParentTransformer` (أولياء الأمور)
- `StudentTransformer` (الطلاب)
- `DepartmentTransformer` (الفروع والأقسام)

#### الفوائد

- إثراء استجابات API ببيانات KYC دون الحاجة لطلبات منفصلة.
- تكامل سلس مع نظام `Nano.API` و `Fractal`.
- قابلية التوسع لإضافة Transformers جديدة بسهولة.

---

### الإصدار 1.0.9 – إضافة `assessKycStatusByCategory` وتحديث السلوك

#### أهداف الإصدار

- توفير دالة تقييم KYC تعتمد مباشرة على تصنيف الوثائق (`category`) دون الحاجة لاستخدام `risk_level` أو `getMandatoryTypesForParty`.
- إضافة `include` جديد باسم `kyc_status_by_category` في السلوك لاستخدام هذه الدالة.
- تحسين المرونة في تقييم فئات محددة من الوثائق.

#### الميزات الجديدة

##### 1. دالة `assessKycStatusByCategory` في `Manager`

تمت إضافة دالة جديدة تسمح بتقييم KYC بناءً على تصنيف الوثائق مباشرة.

**توقيع الدالة:**
```php
public static function assessKycStatusByCategory(
    $owner,
    ?string $ownerType = null,
    ?int $ownerId = null,
    ?string $category = null,
    $documentType = null,
    array $options = []
): array
```

**المعاملات:**
| المعامل | الوصف |
| :--- | :--- |
| `$category` | تصنيف الوثائق (إلزامي، افتراضي `CATEGORY_PRIMARY_ID`). |
| `$documentType` | نوع وثيقة محدد أو مصفوفة من الأنواع (اختياري). |

**آلية العمل:**
- تبني قائمة الوثائق المطلوبة مباشرة من جميع أنواع الوثائق التي تنتمي إلى `$category`.
- إذا تم تمرير `$documentType`، يتم تضييق النطاق ليشمل فقط الأنواع المحددة.
- لا تعتمد على `risk_level` أو `getMandatoryTypesForParty`.

##### 2. تحديث `DynamicAddIncludeKyc` بإضافة `includeKycStatusByCategory`

تمت إضافة دالة `includeKycStatusByCategory` إلى السلوك، والتي تستخدم `Manager::assessKycStatusByCategory`.

**مثال للاستخدام:**
```
GET /api/v1/me?include=kyc_status_by_category&kyc_category=address&kyc_document_type[]=utility_bill&kyc_document_type[]=bank_statement
```

**الخيارات المدعومة:**
| الخيار | الوصف |
| :--- | :--- |
| `category` | تصنيف الوثائق (مطلوب، افتراضي `primary_id`). |
| `document_type` | نوع وثيقة محدد أو مصفوفة. |
| `include_expired` | تضمين الوثائق المنتهية. |
| `include_pending` | تضمين الوثائق قيد الانتظار. |
| `include_rejected` | تضمين الوثائق المرفوضة. |

#### الفوائد

- تقييم دقيق لفئة معينة من الوثائق (مثل "إثبات العنوان") دون الحاجة لتحديد مستوى مخاطرة.
- مثالية للتحقق من متطلبات محددة (مثل: "هل لدى المستخدم إثبات عنوان ساري؟").
- تكامل كامل مع نظام `include` في API.

---

### الإصدار 1.1.0 – تحسينات على `DocumentType` ودعم أنواع المستخدمين

#### أهداف الإصدار

- توسيع دالة `getMandatoryTypesForParty` لدعم مستخدمي `backend` و `frontend` كأفراد (`individual`).
- ضمان أن تقييم KYC يعمل بشكل صحيح لجميع أنواع المستخدمين في النظام.
- تحسينات عامة على الكود وتوحيد المخرجات.

#### الميزات الجديدة

##### 1. تحديث `DocumentType::getMandatoryTypesForParty`

تم تعديل الدالة لتشمل تحويل أنواع المستخدمين التالية إلى `individual`:
- `RainLab\User\Models\User` (مستخدمي الموقع)
- `Backend\Models\User` (مستخدمي لوحة التحكم)

**الكود قبل التحديث:**
```php
if ($partyType === 'individual') {
    $mandatory[] = self::TYPE_NATIONAL_ID;
}
```

**الكود بعد التحديث:**
```php
if (in_array($partyType, ['RainLab\User\Models\User', 'Backend\Models\User'])) {
    $partyType = 'individual';
}

if ($partyType === 'individual') {
    $mandatory[] = self::TYPE_NATIONAL_ID;
    if (in_array($riskLevel, ['high', 'very_high'])) {
        $mandatory[] = self::TYPE_PASSPORT;
    }
}
```

##### 2. تأثير التحديث على تقييم KYC

- **قبل التحديث**: مستخدمو `backend` لم تكن لديهم أي وثائق إلزامية (`total_documents_required = 0`).
- **بعد التحديث**: يتم التعامل معهم كأفراد، وبالتالي يُطلب منهم `national_id` كحد أدنى (و `passport` في حالات المخاطرة العالية).

##### 3. تحسينات عامة

- توحيد هيكل الاستجابة في جميع دوال `Manager`.
- إضافة تفاصيل تصحيح (`debug`) في بيئة التطوير.
- تحسين معالجة الأخطاء في `DynamicAddIncludeKyc`.

#### الفوائد

- تغطية شاملة لجميع أنواع المستخدمين في النظام.
- سلوك متوقع ومنطقي لتقييم KYC.
- جاهزية الإضافة للاستخدام في سيناريوهات واقعية متنوعة.

---

### ملخص الإصدارات (1.0.7 – 1.1.0)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.0.7 | إضافة `assessKycStatus` في `Manager` لتقييم شامل لحالة KYC. |
| 1.0.8 | تطوير سلوك `DynamicAddIncludeKyc` لحقن بيانات KYC في Transformers API. |
| 1.0.9 | إضافة `assessKycStatusByCategory` ودعم `kyc_status_by_category` في API. |
| 1.1.0 | تحديث `DocumentType` لدعم مستخدمي `backend` و `frontend` كأفراد في التقييم. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملفات `Manager.php`، `DocumentType.php`، `DynamicAddIncludeKyc.php`، و `Plugin.php` بالنسخ الجديدة.

2. **لا توجد هجرات جديدة**:
   - لا يتطلب هذا الإصدار تغييرات في قاعدة البيانات.

3. **تحديث تعريفات API** (اختياري):
   - للاستفادة من `include` الجديدة، تأكد من أن العملاء (Clients) يستخدمون المسارات المحدثة.

4. **اختبار التوافق**:
   - يُنصح باختبار دوال التقييم مع أنواع المستخدمين المختلفة للتأكد من النتائج المتوقعة.

---

### الخاتمة

مع الإصدارات 1.0.7 إلى 1.1.0، أصبحت إضافة `Nano3.Kyc` أكثر ذكاءً وتكاملاً. يمكنها الآن:

- **تقييم حالة KYC** بشكل شامل مع توصيات تلقائية.
- **التكامل مع API** بسلاسة عبر `include` دون الحاجة لطلبات منفصلة.
- **دعم جميع أنواع المستخدمين** في النظام بشكل موحد.
- **توفير مرونة عالية** في تقييم فئات محددة من الوثائق.

هذه التحسينات تجعل الإضافة أداة قوية لفرض سياسات KYC واتخاذ القرارات المستندة إلى حالة التحقق من الهوية.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
