# توثيق السمة `HasAssessKycStatus` – محرك تقييم حالة KYC

## نظرة عامة

السمة `HasAssessKycStatus` توجد في المسار `Nano3\Kyc\Classes\Manager\HasAssessKycStatus`، وهي مسؤولة عن كامل منطق تقييم حالة التحقق من الهوية (KYC) ضمن إضافة `Nano3.Kyc`. تم استخراج هذه السمة من كلاس `Manager` الرئيسي بهدف:

- **فصل المسؤوليات**: جعل كلاس `Manager` أكثر تركيزاً على عمليات إدارة الوثائق (CRUD)، بينما تتولى السمة حصراً مهام التقييم والتحليل.
- **تحسين قابلية الصيانة والاختبار**: يمكن تعديل أو اختبار منطق تقييم KYC بشكل مستقل دون التأثير على بقية الكلاس.
- **دعم التوسع المستقبلي**: تسهيل إضافة سياسات تقييم جديدة أو مجموعات بديلة دون تعقيد الكود الأساسي.

تعتمد السمة على كلاس `DocumentType` للحصول على تعريفات الوثائق والمجموعات البديلة، وعلى موديل `Document` لجلب الوثائق الخاصة بالمالك. وتقدم دالتين رئيسيتين للتقييم:

- `assessKycStatus`: تقييم شامل لحالة KYC بناءً على مستوى المخاطرة والمتطلبات الإلزامية.
- `assessKycStatusByCategory`: تقييم حالة KYC مقتصر على فئة وثائق محددة (مثل الهوية الأساسية، إثبات العنوان).

**أهم ما يميز هذه السمة** هو دعمها لنظام **المجموعات البديلة (Alternative Groups)**، والذي يتيح تعريف مجموعات من أنواع الوثائق (مثل `primary_identity` التي تضم جواز السفر والهوية الوطنية ورخصة القيادة)، ويعتبر أي وثيقة من المجموعة كافية لاستيفاء المتطلب الإلزامي. هذا الأسلوب يعكس الممارسات الواقعية في أنظمة الامتثال حيث لا يُشترط نوع محدد بل فئة من الوثائق المقبولة.

---

## هيكل الاستجابة الموحد

جميع دوال التقييم في السمة تعيد استجابة بنفس الهيكل الموحد المستخدم في `Manager`، مما يسهل تكاملها مع واجهات API والتطبيقات المستهلكة.

| المفتاح | النوع | الوصف |
| :--- | :--- | :--- |
| `code` | `int` | كود الحالة (200 للنجاح، 400 للخطأ). |
| `status` | `bool` | حالة العملية (`true` للنجاح، `false` للفشل). |
| `message` | `string` | رسالة وصفية (قابلة للترجمة). |
| `error` | `string\|null` | رسالة الخطأ (إن وجدت). |
| `data` | `array` | بيانات التقييم (انظر هيكل `data` أدناه). |
| `input_data` | `array` | المعاملات المدخلة بعد المعالجة الأولية. |
| `process_data` | `array` | البيانات المستخدمة فعلياً أثناء تنفيذ العملية. |
| `debug` | `array\|null` | معلومات تصحيح (تظهر فقط في بيئة التطوير عند `app.debug = true`). |

**هيكل `data` الخاص بالتقييم:**

| المفتاح | النوع | الوصف |
| :--- | :--- | :--- |
| `is_compliant` | `bool` | هل حالة KYC متوافقة تماماً مع المتطلبات؟ |
| `completion_percentage` | `float` | نسبة اكتمال المتطلبات الإلزامية (0-100). |
| `overall_score` | `int` | الدرجة الإجمالية للتحقق (0-100) محسوبة من أوزان الوثائق. |
| `total_documents_required` | `int` | عدد المتطلبات الإلزامية (بما فيها المجموعات). |
| `verified_documents_count` | `int` | عدد الوثائق المعتمدة والصالحة. |
| `pending_documents_count` | `int` | عدد الوثائق قيد الانتظار. |
| `expired_documents_count` | `int` | عدد الوثائق المنتهية الصلاحية. |
| `rejected_documents_count` | `int` | عدد الوثائق المرفوضة. |
| `verified_documents` | `array` | قائمة الوثائق المعتمدة مع تفاصيل مختصرة. |
| `pending_documents` | `array` | قائمة الوثائق قيد الانتظار. |
| `expired_documents` | `array` | قائمة الوثائق المنتهية. |
| `rejected_documents` | `array` | قائمة الوثائق المرفوضة. |
| `missing_required_types` | `array` | قائمة بأنواع الوثائق المفقودة (أو أمثلة عن المجموعات غير المستوفاة). |
| `recommendations` | `array` | توصيات نصية للمستخدم (مثل "مطلوب تقديم إثبات عنوان"). |

---

## الدوال العامة في السمة

### 1. `assessKycStatus`

تقييم شامل لحالة KYC لمالك معين (فرد أو شركة). تعتمد على `DocumentType::getMandatoryTypesForParty` لتحديد المتطلبات الإلزامية بناءً على نوع الطرف ومستوى المخاطرة، وتستخدم نظام المجموعات البديلة لتحديد ما إذا كانت المتطلبات مستوفاة.

```php
public static function assessKycStatus($owner, ?string $ownerType = null, ?int $ownerId = null, array $options = []): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$owner` | `object\|string` | كائن المالك **أو** نوع المالك (إذا تم تمرير `$ownerType` و `$ownerId`). |
| `$ownerType` | `?string` | نوع المالك (مطلوب إذا كان `$owner` ليس كائنًا). مثال: `RainLab\User\Models\User`. |
| `$ownerId` | `?int` | معرف المالك (مطلوب إذا كان `$owner` ليس كائنًا). |
| `$options` | `array` | خيارات إضافية (انظر الجدول أدناه). |

**الخيارات المدعومة في `$options`:**

| الخيار | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `risk_level` | `string` | `'medium'` | مستوى المخاطرة لتحديد الوثائق الإلزامية (`low`, `medium`, `high`). |
| `category` | `?string` | `null` | فلترة حسب فئة وثيقة محددة (`primary_id`, `address`, `corporate`, ...). |
| `document_type` | `string\|array\|null` | `null` | فلترة حسب نوع وثيقة محدد أو مصفوفة من الأنواع. |
| `include_expired` | `bool` | `true` | تضمين الوثائق المنتهية في النتائج. |
| `include_pending` | `bool` | `true` | تضمين الوثائق قيد الانتظار. |
| `include_rejected` | `bool` | `false` | تضمين الوثائق المرفوضة. |

**مثال عملي (تقييم مستخدم بمخاطرة متوسطة):**

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatus($user, null, null, ['risk_level' => 'medium']);

if ($assessment['status']) {
    echo "نسبة الاكتمال: " . $assessment['data']['completion_percentage'] . "%\n";
    foreach ($assessment['data']['recommendations'] as $rec) {
        echo "- $rec\n";
    }
}
```

---

### 2. `assessKycStatusByCategory`

تقييم حالة KYC لمالك معين بناءً على تصنيف وثائق محدد (مثل `primary_id`). على عكس الدالة العامة، هذه الدالة **لا** تعتمد على `risk_level` أو `getMandatoryTypesForParty`، بل تحدد المتطلبات بناءً على الفئة المطلوبة والأنواع المحددة اختيارياً، مع الاستفادة أيضاً من نظام المجموعات البديلة.

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

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$owner` | `object\|string` | كائن المالك **أو** نوع المالك. |
| `$ownerType` | `?string` | نوع المالك (مطلوب إذا كان `$owner` ليس كائنًا). |
| `$ownerId` | `?int` | معرف المالك. |
| `$category` | `?string` | تصنيف الوثائق (**إلزامي**، إذا كان `null` أو فارغًا يستخدم `CATEGORY_PRIMARY_ID`). |
| `$documentType` | `string\|array\|null` | نوع وثيقة محدد أو مصفوفة من الأنواع (اختياري). إذا تم تمريره، يتم تضييق نطاق المتطلبات ضمن هذه الأنواع فقط. |
| `$options` | `array` | خيارات إضافية (`include_expired`, `include_pending`, `include_rejected`). |

**مثال عملي (تقييم فئة الهوية الأساسية لمستخدم مع اشتراط جواز سفر فقط):**

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatusByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'passport',
    ['include_expired' => true]
);

if ($assessment['status']) {
    echo "الوثائق المطلوبة: " . $assessment['data']['total_documents_required'] . "\n";
    echo "الوثائق المتحقق منها: " . $assessment['data']['verified_documents_count'] . "\n";
}
```

---

## الدوال المساعدة الداخلية

لفهم آلية عمل السمة، من المفيد الاطلاع على الدوال المساعدة التالية (جميعها `protected static`):

| الدالة | الوصف |
| :--- | :--- |
| `getEmptyAssessmentData()` | تعيد هيكل البيانات الفارغ للتقييم. |
| `processMandatoryRequirements($mandatoryRequirements, $category, $documentType)` | تعالج المتطلبات الإلزامية (قد تحتوي على نصوص مثل `group:primary_identity`) وتحولها إلى قائمتين: `individual_types` (الأنواع الفردية) و `group_requirements` (المجموعات وأنواعها). |
| `analyzeDocuments($documents)` | تحلل مجموعة الوثائق المسترجعة وتستخرج الإحصائيات (عدد المعتمدة، المعلقة، المنتهية) وقائمة الأنواع المتوافقة (`compliant_types`). |
| `determineSatisfiedGroups($groupRequirements, $compliantTypes)` | تحدد المجموعات البديلة التي تم استيفاؤها (أي لديها على الأقل وثيقة واحدة معتمدة). |
| `determineMissingRequirements($mandatoryRequirements, $groupRequirements, $satisfiedGroups, $compliantTypes)` | تحدد المتطلبات المفقودة سواء كانت أنواعاً فردية أو مجموعات بديلة غير مستوفاة. |
| `buildRecommendations(...)` | تبني قائمة توصيات نصية بناءً على نتائج التحليل (مثل "مطلوب تقديم إثبات عنوان"). |

---

## نظام المجموعات البديلة (Alternative Groups)

جوهر المرونة التي تقدمها السمة هو دعم **المجموعات البديلة** المعرفة في `DocumentType` عبر الدالة `getAlternativeGroups()`. حالياً، المجموعات المدعومة افتراضياً هي:

| مفتاح المجموعة | الوصف | أنواع الوثائق المنتمية |
| :--- | :--- | :--- |
| `primary_identity` | وثائق الهوية الأساسية | `passport`, `national_id`, `drivers_license`, `residence_permit`, `refugee_document` |
| `address_verification` | وثائق إثبات العنوان | `utility_bill`, `bank_statement`, `tax_document`, `rental_agreement`, `mortgage_statement` |

عندما يُعرّف متطلب إلزامي على شكل `'group:primary_identity'` (كما هو الحال في `DocumentType::getMandatoryTypesForParty`)، فإن السمة تتعامل معه على أنه "مطلوب وثيقة واحدة على الأقل من هذه المجموعة". إذا وُجدت أي وثيقة معتمدة من المجموعة، يُعتبر المتطلب مستوفى.

يمكن توسيع هذه المجموعات أو تعديلها عبر ملف الإعدادات `config/nano3/kyc/document_types.php` في الإصدارات المستقبلية.

---

## أمثلة متكاملة

### المثال 1: التحقق من اكتمال KYC قبل السماح بإجراء معين

```php
use Nano3\Kyc\Classes\Manager;

$user = Auth::getUser();
$assessment = Manager::assessKycStatus($user);

if ($assessment['status'] && $assessment['data']['is_compliant']) {
    // السماح بالإجراء
} else {
    $missing = implode(', ', $assessment['data']['missing_required_types']);
    throw new ApplicationException("يرجى استكمال الوثائق المطلوبة: " . $missing);
}
```

### المثال 2: الحصول على نسبة اكتمال الهوية الأساسية فقط

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatusByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID
);

$progress = $assessment['data']['completion_percentage'];
echo "تقدم الهوية الأساسية: $progress%";
```

---

## الاعتماديات (Dependencies)

تعتمد السمة `HasAssessKycStatus` على المكونات التالية:

- **`Nano3\Kyc\Classes\DocumentType`**: للحصول على تعريفات الوثائق، المجموعات البديلة، والمتطلبات الإلزامية.
- **`Nano3\Kyc\Models\Document`**: لجلب وثائق المالك وتطبيق نطاقات البحث.
- **`Carbon\Carbon`**: لمعالجة التواريخ (التحقق من الصلاحية).

---

## ملاحظات هامة

1. **التقييم يعتمد على الوثائق المعتمدة فقط (`verified`)** لتحديد استيفاء المتطلبات، بينما الوثائق المعلقة أو المرفوضة لا تحتسب في الاكتمال.
2. **الدرجة الإجمالية (`overall_score`)** تُحسب بناءً على أوزان الوثائق الفردية التي تشكل المتطلبات الإلزامية (بعد فك المجموعات)، وليس على جميع الوثائق الموجودة.
3. **التوصيات** تولد تلقائياً بناءً على المتطلبات المفقودة ووجود وثائق منتهية أو معلقة.
4. **التوسع**: يمكن إضافة مجموعات بديلة جديدة بسهولة عبر تعديل `DocumentType::getAlternativeGroups()`، دون الحاجة لتغيير كود السمة.
5. **التوافق**: السمة مصممة للعمل بشكل ثابت (`static`)، وبالتالي يمكن استدعاء دوالها مباشرة من `Manager` دون الحاجة لنسخ كائن.

---

## الخلاصة

توفر السمة `HasAssessKycStatus` محركاً مرناً وقوياً لتقييم حالة KYC، مع فصل واضح للمسؤوليات ودعم كامل للمجموعات البديلة. بفضل هذه السمة، أصبح بإمكان المطورين الحصول على تقييمات دقيقة وواقعية لحالة الامتثال، مع إمكانية تخصيص المتطلبات بسهولة وفق سياسات العمل المختلفة.


**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)

