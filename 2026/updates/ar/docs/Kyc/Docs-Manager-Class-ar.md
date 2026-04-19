# توثيق كلاس `Manager` – مدير عمليات KYC

## نظرة عامة

يقع كلاس `Manager` في المسار `Nano3\Kyc\Classes\Manager`، وهو المنسق المركزي لجميع عمليات إدارة وثائق التحقق من الهوية (KYC) ضمن برمجيات نانوسوفت (NanoSoft App). يعمل الكلاس كطبقة خدمة وسيطة (Service Layer) تتعامل مع موديل `Document` وكلاس `DocumentType` لتنفيذ العمليات الأساسية مثل:

- إنشاء وثيقة جديدة.
- تحديث بيانات وثيقة.
- حذف واستعادة وثيقة (Soft Delete).
- اعتماد وثيقة (تحقق).
- رفض وثيقة.
- التحقق من التكرار.
- جلب السجلات والإحصائيات.

يتميز الكلاس بما يلي:
- **هيكل استجابة موحد**: جميع الدوال تعيد مصفوفة بنفس البنية (`code`، `status`، `message`، `model`، `data`، إلخ).
- **دعم المعاملات (Transactions)**: جميع عمليات الكتابة محاطة بمعاملة لضمان تكامل البيانات.
- **دعم الاختبارات (`is_test_create`)**: إمكانية تنفيذ العملية ضمن معاملة يتم التراجع عنها بعد التحقق من نجاحها.
- **إطلاق أحداث (Events)**: إطلاق أحداث مثل `nano3.kyc.document.created` بعد إتمام العمليات.
- **التحقق من التكرار**: منع إنشاء وثائق مكررة لنفس المالك.
- **معالجة الملفات المرفقة**: دعم ربط الملفات (صور، PDF) بالوثيقة عبر علاقات `attachOne` و `attachMany`.

يستخدم الكلاس نمط `Singleton`، مما يعني أنه يمكن الوصول إلى دواله بشكل ثابت (`Manager::method()`) دون الحاجة إلى إنشاء كائن جديد.

---

## هيكل الاستجابة الموحد

جميع الدوال العامة في `Manager` تعيد استجابة بهيكل موحد يسهل معالجته من قبل المستهلكين (API، وحدات التحكم، إلخ).

| المفتاح | النوع | الوصف |
| :--- | :--- | :--- |
| `code` | `int` | كود الحالة (200 للنجاح، 400 للخطأ). |
| `status` | `bool` | حالة العملية (`true` للنجاح، `false` للفشل). |
| `message` | `string` | رسالة وصفية (قابلة للترجمة). |
| `error` | `string\|null` | رسالة الخطأ (إن وجدت). |
| `errors` | `array\|null` | مصفوفة أخطاء التحقق (إن وجدت). |
| `model` | `Document\|null` | كائن الوثيقة الذي تم إنشاؤه أو تحديثه. |
| `data` | `array\|null` | تمثيل مصفوفي للوثيقة (`toArray()`). |
| `input_data` | `array` | المعاملات المدخلة بعد المعالجة الأولية. |
| `process_data` | `array` | البيانات المستخدمة فعلياً أثناء تنفيذ العملية. |
| `debug` | `array\|null` | معلومات تصحيح (تظهر فقط في بيئة التطوير عند `app.debug = true`). |

**مثال على استجابة ناجحة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم إنشاء الوثيقة بنجاح.",
    "error": null,
    "errors": null,
    "model": { ... },
    "data": { ... },
    "input_data": { ... },
    "process_data": { ... }
}
```

---

## الدوال العامة (Public API)

### 1. `createDocument`

تنشئ وثيقة KYC جديدة بعد التحقق من صحة البيانات، صلاحية نوع الوثيقة، وعدم التكرار.

```php
public static function createDocument(array $options = [], bool $is_test_create = false): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$options` | `array` | مصفوفة خيارات إنشاء الوثيقة (انظر الجدول أدناه). |
| `$is_test_create` | `bool` | إذا كان `true`، يتم التراجع عن العملية بعد التنفيذ (لأغراض الاختبار). |

#### خيارات `$options` المدعومة

| المفتاح | النوع | إلزامي | الوصف |
| :--- | :--- | :--- | :--- |
| `document_type` | `string` | **نعم** | كود نوع الوثيقة (مثال: `passport`، `national_id`). |
| `owner_id` | `int` | **نعم** | معرف المالك. |
| `owner_type` | `string` | **نعم** | نوع المالك (اسم كلاس الـ Morph). |
| `owner` | `object` | لا | كائن المالك (يتم استخراج `id` و `type` منه تلقائياً). |
| `verifier_id` | `int` | لا | معرف المحقق. |
| `verifier_type` | `string` | لا | نوع المحقق. |
| `verifier` | `object` | لا | كائن المحقق. |
| `subject_id` | `int` | لا | معرف الموضوع المرتبط. |
| `subject_type` | `string` | لا | نوع الموضوع المرتبط. |
| `subject` | `object` | لا | كائن الموضوع المرتبط. |
| `fields` | `array` | لا | مصفوفة الحقول الديناميكية (مثل `['full_name' => '...', 'document_number' => '...']`). |
| `files` | `array` | لا | مصفوفة الملفات المرفوعة (مثل `['document_front' => $uploadedFile]`). |
| `status` | `string` | لا | حالة الوثيقة (الافتراضي: `pending`). |
| `is_verified` | `bool` | لا | هل الوثيقة معتمدة؟ (الافتراضي: `false`). |
| `verification_score` | `int` | لا | درجة الموثوقية (0-100). |
| `is_physical_submitted` | `bool` | لا | هل تم تسليم نسخة ورقية؟ |
| `physical_copy_type` | `string` | لا | نوع النسخة (`original` أو `copy`). |
| `physical_received_at` | `string\|Carbon` | لا | تاريخ استلام النسخة الورقية. |
| `physical_notes` | `string` | لا | ملاحظات حول النسخة الورقية. |
| `is_published` | `bool` | لا | هل الوثيقة منشورة؟ |
| `is_public` | `bool` | لا | هل الوثيقة عامة؟ |
| `is_active` | `bool` | لا | هل الوثيقة نشطة؟ (الافتراضي: `true`). |
| `is_default` | `bool` | لا | هل الوثيقة افتراضية؟ |
| `extend_id` | `string` | لا | معرف خارجي للربط. |
| `companys_id` | `string` | لا | معرف الشركة. |
| `departments_id` | `string` | لا | معرف الفرع. |
| `metadata` | `array` | لا | بيانات وصفية (JSON). |
| `other_data` | `array` | لا | بيانات إضافية (JSON). |
| `config_data` | `array` | لا | بيانات إعدادات (JSON). |
| `is_stop_event` | `bool` | لا | إذا كان `true`، لا يتم إطلاق حدث `created`. |
| `is_stop_duplicate` | `bool` | لا | إذا كان `true`، يتم التحقق من التكرار ومنع إنشاء وثيقة مكررة. |
| `duplicate_check_fields` | `array` | لا | الحقول المستخدمة للتحقق من التكرار (الافتراضي: `['document_number', 'owner_id', 'owner_type']`). |

#### مثال عملي

```php
use Nano3\Kyc\Classes\Manager;

$result = Manager::createDocument([
    'document_type' => 'passport',
    'owner'         => $currentUser, // كائن المستخدم
    'fields'        => [
        'document_number' => 'A12345678',
        'full_name'       => 'John Doe',
        'issue_date'      => '2020-01-01',
        'expiry_date'     => '2030-01-01',
        'country_of_issue'=> 'US',
        'nationality'     => 'US',
    ],
    'files' => [
        'document_front' => Input::file('front_image')
    ],
    'is_stop_duplicate' => true,
]);

if ($result['status']) {
    $document = $result['model'];
    echo "تم إنشاء الوثيقة برقم: " . $document->id;
} else {
    echo "خطأ: " . $result['error'];
}
```

---

### 2. `updateDocument`

تحديث بيانات وثيقة موجودة.

```php
public static function updateDocument($documentId, array $options = [], bool $is_test_create = false): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | معرف الوثيقة المراد تحديثها. |
| `$options` | `array` | مصفوفة الحقول المطلوب تحديثها (نفس خيارات `createDocument` تقريباً). |
| `$is_test_create` | `bool` | للاختبار فقط. |

**ملاحظة:** يمكن تحديث الحقول الديناميكية عبر المفتاح `fields`، والملفات عبر المفتاح `files`. الحقول التي لم تُذكر في `$options` تبقى دون تغيير.

---

### 3. `deleteDocument`

حذف وثيقة (Soft Delete). الوثيقة لا تُحذف نهائياً بل تُؤرشَف.

```php
public static function deleteDocument($documentId, bool $is_test_create = false): array
```

---

### 4. `restoreDocument`

استعادة وثيقة محذوفة (Soft Deleted).

```php
public static function restoreDocument($documentId): array
```

---

### 5. `verifyDocument`

اعتماد وثيقة (التحقق من صحتها). تقوم بتحديث `is_verified` إلى `true`، وتسجيل المحقق، وتاريخ التحقق، ودرجة الموثوقية.

```php
public static function verifyDocument($documentId, $verifier = null, $verification_score = 100, bool $is_test_create = false): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | معرف الوثيقة. |
| `$verifier` | `object\|null` | كائن المحقق (اختياري). إذا تم تمريره، يتم تسجيل `verifier_id` و `verifier_type`. |
| `$verification_score` | `int` | درجة الموثوقية (0-100، الافتراضي 100). |
| `$is_test_create` | `bool` | للاختبار فقط. |

---

### 6. `rejectDocument`

رفض وثيقة مع إمكانية إضافة سبب الرفض.

```php
public static function rejectDocument($documentId, $reason = null, bool $is_test_create = false): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | معرف الوثيقة. |
| `$reason` | `string\|null` | سبب الرفض (يُخزن في `metadata['rejection_reason']`). |
| `$is_test_create` | `bool` | للاختبار فقط. |

---

### 7. `checkDuplicateDocument`

التحقق من وجود وثيقة مكررة بناءً على حقول محددة. مفيدة قبل إنشاء وثيقة جديدة.

```php
public static function checkDuplicateDocument(array $options = []): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$options` | `array` | مصفوفة تحتوي على الحقول المستخدمة للتحقق (`document_number`، `owner_id`، `owner_type`، `document_type`، `exclude_id`). |

**مثال:**
```php
$result = Manager::checkDuplicateDocument([
    'document_number' => 'A12345678',
    'owner_id'        => 5,
    'owner_type'      => 'RainLab\User\Models\User',
]);
if (!$result['status']) {
    echo "الوثيقة موجودة مسبقاً: " . $result['error'];
}
```

---

### 8. `getDocumentRecords`

جلب سجلات الوثائق باستخدام دالة `getRecords` الموجودة في موديل `Document`. تدعم هذه الدالة خيارات فلترة متقدمة (نوع الوثيقة، الحالة، المالك، نطاقات التاريخ، إلخ).

```php
public static function getDocumentRecords($options = []): array
```

**مثال:**
```php
$result = Manager::getDocumentRecords([
    'document_type' => 'passport',
    'status'        => 'pending',
    'is_paginator'  => true,
    'per_page'      => 20,
]);
$documents = $result['data']; // كائن Paginator
```

---

### 9. `getDocumentStats`

جلب إحصائيات سريعة عن الوثائق (العدد الإجمالي، النشطة، المنشورة، موزعة حسب النوع).

```php
public static function getDocumentStats($options = []): array
```

---

### 10. دوال مساعدة داخلية

| الدالة | الوصف |
| :--- | :--- |
| `prepareDocumentOptions()` | تجهيز القيم الافتراضية للشركة والفرع من `BasicHelper`. |
| `getQueryDate()` | تطبيق فلتر تاريخ مرن على استعلام (يُستخدم في `getRecords`). |
| `checkValueIsNotAll()` | التحقق من أن القيمة ليست عامة (`*` أو `all`). |
| `scopeWhereField()` | نطاق عام لتطبيق شروط `WHERE` مع دعم `is_or` و `is_not`. |

---

### 11. دوال تقييم حالة KYC (KYC Assessment)

#### `assessKycStatus`

تقييم شامل لحالة KYC لمالك معين (فرد أو شركة). تقوم الدالة بفحص جميع الوثائق المرتبطة بالمالك، وتحديد الوثائق المكتملة، المفقودة، المنتهية، وحساب نسبة الاكتمال والدرجة الإجمالية للتحقق.

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

**هيكل الاستجابة:**

```json
{
    "code": 200,
    "status": true,
    "message": "KYC assessment completed successfully.",
    "error": null,
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
    },
    "input_data": [],
    "process_data": []
}
```

**مثال عملي (تقييم مستخدم بمخاطرة عالية):**

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatus($user, null, null, ['risk_level' => 'high']);

if ($assessment['status']) {
    echo "نسبة الاكتمال: " . $assessment['data']['completion_percentage'] . "%\n";
    foreach ($assessment['data']['recommendations'] as $rec) {
        echo "- $rec\n";
    }
}
```

#### `assessKycStatusByCategory`

تقييم حالة KYC لمالك معين بناءً على تصنيف الوثائق (`category`) ونوع الوثيقة المحدد. لا تعتمد هذه الدالة على `risk_level` أو `getMandatoryTypesForParty`، بل تبني قائمة الوثائق المطلوبة مباشرة من التصنيف والأنواع المحددة.

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
| `$documentType` | `string\|array\|null` | نوع وثيقة محدد أو مصفوفة من الأنواع (اختياري). |
| `$options` | `array` | خيارات إضافية (`include_expired`, `include_pending`, `include_rejected`). |

**مثال عملي (تقييم فئة الهوية الأساسية لمستخدم):**

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatusByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    ['passport', 'national_id'],
    ['include_expired' => true]
);

if ($assessment['status']) {
    echo "الوثائق المطلوبة: " . $assessment['data']['total_documents_required'] . "\n";
    echo "الوثائق المتحقق منها: " . $assessment['data']['verified_documents_count'] . "\n";
}
```

---

## الأحداث (Events)

يطلق `Manager` الأحداث التالية، مما يسمح للمطورين بربط منطق مخصص (مثل إرسال إشعارات، تسجيل سجل، تحديث أنظمة خارجية):

| الحدث | المعاملات | الوصف |
| :--- | :--- | :--- |
| `nano3.kyc.document.created` | `$document, $user, $subject` | عند إنشاء وثيقة جديدة. |
| `nano3.kyc.document.updated` | `$document` | عند تحديث وثيقة. |
| `nano3.kyc.document.deleted` | `$document` | عند حذف وثيقة. |
| `nano3.kyc.document.restored` | `$document` | عند استعادة وثيقة. |
| `nano3.kyc.document.verified` | `$document` | عند اعتماد وثيقة. |
| `nano3.kyc.document.rejected` | `$document` | عند رفض وثيقة. |

**مثال للاستماع لحدث:**
```php
Event::listen('nano3.kyc.document.verified', function ($document) {
    // إرسال إشعار للمالك بأن وثيقته تم اعتمادها
    Notification::send($document->owner, new DocumentVerifiedNotification($document));
});
```

---

## الاعتماديات (Dependencies)

يعتمد `Manager` على المكونات التالية:

- **`Nano3\Kyc\Models\Document`**: الموديل الرئيسي.
- **`Nano3\Kyc\Classes\DocumentType`**: للتحقق من أنواع الوثائق وقواعدها.
- **`Tss\Basic\Helpers\BasicHelper`**: لتجهيز معرفات الشركة والفرع الافتراضية.
- **`Illuminate\Support\Facades\DB`**: للمعاملات.
- **`Illuminate\Support\Facades\Event`**: لإطلاق الأحداث.
- **`Illuminate\Support\Facades\Validator`**: للتحقق من صحة الحقول الديناميكية.

---

## ملاحظات هامة

1. **التحقق من الصلاحيات**: الكلاس الحالي لا يقوم بالتحقق من صلاحيات المستخدم (مثل `can('create')`). يُفترض أن يتم التحقق من الصلاحيات في طبقة المتحكم (Controller) قبل استدعاء `Manager`.
2. **معالجة الملفات**: يفترض أن الملفات الممررة في خيار `files` هي كائنات `UploadedFile` جاهزة. يتم حفظها باستخدام علاقات `attachOne` و `attachMany` الموجودة في موديل `Document`.
3. **التخزين المؤقت**: `Manager` لا يتعامل مباشرة مع الكاش، ولكن `Document` يستخدم سمات (`ListObjects`, `ListOptions`) لإدارة التخزين المؤقت للسجلات. عند استخدام `Manager` لتحديث أو حذف وثيقة، يتم مسح الكاش تلقائياً عبر أحداث الموديل.
4. **اختبار الدوال**: استخدم الخيار `is_test_create = true` لاختبار الدوال دون التأثير على قاعدة البيانات الحقيقية.

---

## أمثلة متكاملة

### السيناريو 1: رفع جواز سفر جديد من خلال API

```php
public function store(Request $request)
{
    $result = Manager::createDocument([
        'document_type' => 'passport',
        'owner'         => Auth::getUser(),
        'fields'        => $request->input('fields'),
        'files'         => [
            'document_front' => $request->file('document_front')
        ],
        'is_stop_duplicate' => true,
    ]);

    if ($result['status']) {
        return response()->json($result);
    }

    return response()->json($result, 400);
}
```

### السيناريو 2: مهمة مجدولة للتحقق من صلاحية الوثائق

```php
public function fire()
{
    $documents = Document::where('status', 'verified')
        ->whereNotNull('expiry_date')
        ->get();

    foreach ($documents as $doc) {
        if (Carbon::parse($doc->expiry_date)->isPast()) {
            $doc->status = 'expired';
            $doc->save();
        }
    }
}
```
