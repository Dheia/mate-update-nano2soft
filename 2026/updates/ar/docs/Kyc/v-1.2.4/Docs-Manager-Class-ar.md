# توثيق كلاس `Manager` – مدير عمليات KYC

## نظرة عامة

يقع كلاس `Manager` في المسار `Nano3\Kyc\Classes\Manager`، وهو القلب النابض لإضافة `Nano3.Kyc` والمنسق المركزي لجميع عمليات إدارة وثائق التحقق من الهوية (KYC) ضمن برمجيات نانوسوفت (NanoSoft App). يعمل الكلاس كطبقة خدمة وسيطة (Service Layer) متكاملة تتعامل مع موديل `Document` وكلاس `DocumentType` لتنفيذ العمليات الأساسية مثل:

- إنشاء وثيقة جديدة.
- تحديث بيانات وثيقة.
- حذف واستعادة وثيقة (Soft Delete).
- اعتماد وثيقة (تحقق).
- رفض وثيقة.
- التحقق من التكرار.
- جلب السجلات والإحصائيات.
- تقييم حالة KYC للمستخدمين والشركات (عبر السمة `HasAssessKycStatus`).

يتميز الكلاس بعدة خصائص تجعله قوياً ومرناً في الوقت نفسه:

- **هيكل استجابة موحد**: جميع الدوال العامة تعيد مصفوفة بنفس البنية (`code`، `status`، `message`، `model`، `data`، إلخ) مما يسهل التعامل معها من قبل المستهلكين (API، وحدات التحكم، الاختبارات).
- **دعم المعاملات (Transactions)**: جميع عمليات الكتابة محاطة بمعاملة قاعدة بيانات لضمان تكامل البيانات وإمكانية التراجع في حالة حدوث خطأ.
- **دعم الاختبارات (`is_test_create`)**: إمكانية تنفيذ العملية ضمن معاملة يتم التراجع عنها بعد التحقق من نجاحها، مما يسهل كتابة اختبارات وهمية دون تلويث قاعدة البيانات.
- **إطلاق أحداث (Events)**: إطلاق أحداث مثل `nano3.kyc.document.created` بعد إتمام العمليات، مما يسمح للمطورين بربط منطق مخصص (إشعارات، مزامنة، تدقيق).
- **التحقق من التكرار**: منع إنشاء وثائق مكررة لنفس المالك بناءً على حقول قابلة للتخصيص.
- **معالجة الملفات المرفقة**: دعم ربط الملفات (صور، PDF) بالوثيقة عبر علاقات `attachOne` و `attachMany` الموجودة في موديل `Document`.
- **هندسة نظيفة عبر السمات (Traits)**: تم نقل منطق تقييم KYC إلى سمة مستقلة `HasAssessKycStatus`، مما يحافظ على تركيز `Manager` في عمليات إدارة الوثائق مع إتاحة التوسع بسهولة.

يستخدم الكلاس نمط `Singleton` عبر السمة `October\Rain\Support\Traits\Singleton`، مما يعني أنه يمكن الوصول إلى دواله بشكل ثابت (`Manager::method()`) دون الحاجة إلى إنشاء كائن جديد.

---

## هيكل الاستجابة الموحد

جميع الدوال العامة في `Manager` تعيد استجابة بهيكل موحد يسهل معالجته من قبل المستهلكين (API، وحدات التحكم، إلخ). هذا التصميم يضمن تجربة متسقة ويقلل من الحاجة إلى كتابة منطق معالجة أخطاء مخصص.

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

**مثال على استجابة فاشلة (خطأ تحقق):**
```json
{
    "code": 400,
    "status": false,
    "message": "فشل إنشاء الوثيقة.",
    "error": "The document number field is required.",
    "errors": {
        "document_number": ["حقل رقم الوثيقة مطلوب."]
    },
    "model": null,
    "data": null
}
```

---

## الدوال العامة (Public API)

### 1. `createDocument`

تنشئ وثيقة KYC جديدة بعد التحقق من صحة البيانات، صلاحية نوع الوثيقة، وعدم التكرار (اختيارياً). تمر العملية بعدة مراحل:

1. التحقق من وجود المالك (`owner_id`, `owner_type`).
2. التحقق من صلاحية `document_type` باستخدام `DocumentType`.
3. التحقق من أن نوع الوثيقة مسموح به لنوع المالك (`isValidForPartyType`).
4. (اختياري) التحقق من عدم وجود وثيقة مكررة لنفس المالك.
5. التحقق من صحة الحقول الديناميكية (`fields`) مقابل قواعد `DocumentType::getValidationRules`.
6. تحويل الحقول إلى هيكل قاعدة البيانات (`mapInputToDocumentData`).
7. تنفيذ العملية داخل معاملة: حفظ الوثيقة، معالجة الملفات المرفقة.
8. إطلاق حدث `nano3.kyc.document.created` (ما لم يتم إيقافه).
9. إعادة استجابة موحدة.

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
| `skip_file_check` | `bool` | لا | إذا كان `true`، يتم استثناء حقول الملفات من قواعد التحقق. |
| `validation_rules_except` | `array` | لا | قائمة بقواعد التحقق التي سيتم استثناؤها (افتراضي: حقول الملفات). |
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

تحديث بيانات وثيقة موجودة. تمر العملية بمراحل مشابهة لـ `createDocument`، مع اختلاف أن الحقول غير المذكورة تبقى كما هي.

```php
public static function updateDocument($documentId, array $options = [], bool $is_test_create = false): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$documentId` | `int\|string` | معرف الوثيقة المراد تحديثها. |
| `$options` | `array` | مصفوفة الحقول المطلوب تحديثها (نفس خيارات `createDocument` تقريباً). |
| `$is_test_create` | `bool` | للاختبار فقط. |

**ملاحظة:** يمكن تحديث الحقول الديناميكية عبر المفتاح `fields`، والملفات عبر المفتاح `files`. الحقول التي لم تُذكر في `$options` تبقى دون تغيير. إذا تم تمرير `is_verified = true` لوثيقة لم تكن معتمدة، يتم تعيين `verified_at` تلقائياً.

---

### 3. `deleteDocument`

حذف وثيقة (Soft Delete). الوثيقة لا تُحذف نهائياً بل يتم تعيين `is_active = false` و `deleted_at = now()`، ثم يتم حفظها. هذا يسمح باستعادتها لاحقاً.

```php
public static function deleteDocument($documentId, bool $is_test_create = false): array
```

**ملاحظة:** في الإصدار الحالي، يتم تنفيذ الحذف الناعم يدوياً بدلاً من استخدام `$document->delete()` لتجنب أي تعارضات محتملة مع سمات أخرى. إذا كنت تستخدم `SoftDelete` trait في الموديل، يمكن تعديل هذه الدالة لاستدعاء `delete()` مباشرة.

---

### 4. `restoreDocument`

استعادة وثيقة محذوفة (Soft Deleted). تستخدم `onlyTrashed()->find()` للعثور على الوثيقة ثم استدعاء `restore()`.

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

رفض وثيقة مع إمكانية إضافة سبب الرفض. تقوم بتعيين `is_verified = false`، وتغيير الحالة إلى `rejected`، وحفظ سبب الرفض في `metadata['rejection_reason']`.

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

هذه الدوال معرفة كـ `protected static` وتستخدم داخلياً لتبسيط الكود وتجنب التكرار.

| الدالة | الوصف |
| :--- | :--- |
| `prepareDocumentOptions()` | تجهيز القيم الافتراضية للشركة والفرع من `BasicHelper`. |
| `getQueryDate()` | تطبيق فلتر تاريخ مرن على استعلام (يُستخدم في `getRecords`). |
| `checkValueIsNotAll()` | التحقق من أن القيمة ليست عامة (`*` أو `all`). |
| `scopeWhereField()` | نطاق عام لتطبيق شروط `WHERE` مع دعم `is_or` و `is_not` و `is_force`. |

---

### 11. دوال تقييم حالة KYC (مفوضة إلى `HasAssessKycStatus`)

لتقييم حالة KYC، يستخدم `Manager` السمة `HasAssessKycStatus` التي توفر الدوال التالية. راجع [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md) للحصول على تفاصيل كاملة.

#### `assessKycStatus`

تقييم شامل لحالة KYC لمالك معين (فرد أو شركة) بناءً على مستوى المخاطرة والمتطلبات الإلزامية. تدعم المجموعات البديلة.

#### `assessKycStatusByCategory`

تقييم حالة KYC مقتصر على فئة وثائق محددة (مثل الهوية الأساسية، إثبات العنوان)، مع دعم المجموعات البديلة أيضاً.

**مثال سريع:**
```php
$assessment = Manager::assessKycStatus($user);
if ($assessment['data']['is_compliant']) {
    // KYC مكتمل
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

**تعطيل الأحداث:** يمكن تعطيل إطلاق الأحداث لعملية معينة عن طريق تمرير الخيار `'is_stop_event' => true` في `$options`.

---

## آلية التحقق من التكرار

يوفر `Manager` دالة `checkDuplicateDocument` وخيار `is_stop_duplicate` في `createDocument` لمنع إنشاء وثائق مكررة. تعتمد الآلية على الحقول المحددة في `duplicate_check_fields` (الافتراضي: `document_number`, `owner_id`, `owner_type`). يمكن تخصيص هذه الحقول حسب الحاجة.

**مثال لمنع تكرار جواز سفر:**
```php
Manager::createDocument([
    // ... البيانات
    'is_stop_duplicate' => true,
    'duplicate_check_fields' => ['document_number', 'document_type', 'owner_id', 'owner_type'],
]);
```

---

## دعم الاختبارات (`is_test_create`)

جميع دوال الكتابة (`create`, `update`, `delete`, `verify`, `reject`) تقبل معامل `is_test_create`. عند تمرير `true`، يتم تنفيذ العملية داخل معاملة ثم التراجع عنها (`rollBack`) قبل إعادة الاستجابة. هذا يسمح باختبار المنطق بالكامل (بما في ذلك عمليات الحفظ والعلاقات) دون ترك أي أثر في قاعدة البيانات.

**مثال لاختبار وحدة:**
```php
public function testCreateDocument()
{
    $result = Manager::createDocument([...], true);
    $this->assertTrue($result['status']);
    // لم يتم حفظ أي شيء في قاعدة البيانات
}
```

---

## معالجة الملفات المرفقة

عند تمرير مصفوفة `files` في `createDocument` أو `updateDocument`، يقوم `Manager` بإنشاء سجلات `File` وربطها بالوثيقة عبر العلاقات المعرفة في موديل `Document` (`attachOne` و `attachMany`).

**مثال لرفع صورة جواز السفر:**
```php
$result = Manager::createDocument([
    // ...
    'files' => [
        'document_front' => $uploadedFile, // كائن UploadedFile
        'document_back'  => $anotherFile,
    ]
]);
```

يتم حفظ الملفات مباشرة بعد حفظ الوثيقة، وإذا كانت الوثيقة معتمدة (`is_verified = true`) في نفس العملية، فإن الملفات ترتبط بشكل طبيعي. (ملاحظة: واجهة API تطبق حماية إضافية لمنع تغيير ملفات الوثائق المعتمدة، راجع `FileUploadDocuments`).

---

## الاعتماديات (Dependencies)

يعتمد `Manager` على المكونات التالية:

- **`Nano3\Kyc\Models\Document`**: الموديل الرئيسي.
- **`Nano3\Kyc\Classes\DocumentType`**: للتحقق من أنواع الوثائق وقواعدها والمجموعات البديلة.
- **`Nano3\Kyc\Classes\Manager\HasAssessKycStatus`**: سمة تقييم KYC.
- **`Tss\Basic\Helpers\BasicHelper`**: لتجهيز معرفات الشركة والفرع الافتراضية.
- **`Illuminate\Support\Facades\DB`**: للمعاملات.
- **`Illuminate\Support\Facades\Event`**: لإطلاق الأحداث.
- **`Illuminate\Support\Facades\Validator`**: للتحقق من صحة الحقول الديناميكية.
- **`Carbon\Carbon`**: لمعالجة التواريخ.

---

## ملاحظات هامة

1. **التحقق من الصلاحيات**: الكلاس الحالي لا يقوم بالتحقق من صلاحيات المستخدم (مثل `can('create')`). يُفترض أن يتم التحقق من الصلاحيات في طبقة المتحكم (Controller) قبل استدعاء `Manager`.
2. **معالجة الملفات**: يفترض أن الملفات الممررة في خيار `files` هي كائنات `UploadedFile` جاهزة. يتم حفظها باستخدام علاقات `attachOne` و `attachMany` الموجودة في موديل `Document`.
3. **التخزين المؤقت**: `Manager` لا يتعامل مباشرة مع الكاش، ولكن `Document` يستخدم سمات (`ListObjects`, `ListOptions`) لإدارة التخزين المؤقت للسجلات. عند استخدام `Manager` لتحديث أو حذف وثيقة، يتم مسح الكاش تلقائياً عبر أحداث الموديل.
4. **تقييم KYC**: تم نقل منطقه بالكامل إلى السمة `HasAssessKycStatus`، مما يجعل `Manager` أنظف وأكثر تركيزاً. لأي تعديلات على منطق التقييم، يجب العمل في تلك السمة.
5. **الاختبارات**: استخدم الخيار `is_test_create = true` لاختبار الدوال دون التأثير على قاعدة البيانات الحقيقية.

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

### السيناريو 2: التحقق من اكتمال KYC قبل إجراء عملية شراء

```php
$user = Auth::getUser();
$assessment = Manager::assessKycStatus($user, null, null, ['risk_level' => 'medium']);

if (!$assessment['data']['is_compliant']) {
    $missing = implode(', ', $assessment['data']['missing_required_types']);
    throw new ApplicationException("يجب استكمال الوثائق التالية: $missing");
}
// السماح بالشراء
```

---

## الخاتمة

يمثل كلاس `Manager` العمود الفقري لإضافة `Nano3.Kyc`، حيث يقدم واجهة موحدة وآمنة ومرنة لإدارة دورة حياة وثائق KYC وتقييم حالة الامتثال. بفضل دعم المعاملات، الأحداث، الاختبارات، والمجموعات البديلة، يمكن للمطورين الاعتماد عليه في بناء تطبيقات متطلبات امتثال معقدة بسهولة وثقة.

---


## التوثيق الإضافي

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
