# توثيق كلاس DocumentType

## نظرة عامة

`Nano3\Kyc\Classes\DocumentType` هو المكون المركزي لإدارة أنواع وثائق التحقق من الهوية (KYC) ضمن إضافة `Nano3.Kyc` لنظام NanoSoft App. يوفر الكلاس تعريفًا موحدًا وشاملاً لأكثر من **28 نوع وثيقة** موزعة على **6 فئات**، مع خصائص متقدمة تشمل:

- الحقول المطلوبة وقواعد التحقق لكل نوع.
- أنواع الملفات المسموحة والحد الأقصى للحجم.
- أوزان الموثوقية (للتحقق التلقائي).
- دعم الصلاحية (تاريخ الانتهاء أو الحد الأقصى للعمر).
- تعريفات حقول مرنة (Fields Schema) لإنشاء نماذج ديناميكية.
- دوال متكاملة لتحويل البيانات بين المدخلات وقاعدة البيانات.
- دعم كامل لتعدد اللغات وإمكانية التخصيص عبر ملفات الإعدادات (`config`).

---

## الثوابت (Constants)

### أنواع الوثائق (`TYPE_*`)

| الثابت | القيمة | الوصف |
| :--- | :--- | :--- |
| `TYPE_PASSPORT` | `passport` | جواز السفر |
| `TYPE_NATIONAL_ID` | `national_id` | بطاقة الهوية الوطنية |
| `TYPE_DRIVERS_LICENSE` | `drivers_license` | رخصة القيادة |
| `TYPE_RESIDENCE_PERMIT` | `residence_permit` | تصريح الإقامة |
| `TYPE_REFUGEE_DOCUMENT` | `refugee_document` | وثيقة لاجئ |
| `TYPE_VOTER_ID` | `voter_id` | بطاقة ناخب |
| `TYPE_MILITARY_ID` | `military_id` | هوية عسكرية |
| `TYPE_GOVERNMENT_ID` | `government_id` | هوية حكومية |
| `TYPE_UTILITY_BILL` | `utility_bill` | فاتورة مرافق |
| `TYPE_BANK_STATEMENT` | `bank_statement` | كشف حساب بنكي |
| `TYPE_TAX_DOCUMENT` | `tax_document` | مستند ضريبي |
| `TYPE_RENTAL_AGREEMENT` | `rental_agreement` | عقد إيجار |
| `TYPE_MORTGAGE_STATEMENT` | `mortgage_statement` | كشف رهن عقاري |
| `TYPE_CERTIFICATE_OF_INCORPORATION` | `certificate_of_incorporation` | شهادة تأسيس شركة |
| `TYPE_ARTICLES_OF_ASSOCIATION` | `articles_of_association` | عقد التأسيس |
| `TYPE_BUSINESS_LICENSE` | `business_license` | رخصة تجارية |
| `TYPE_TAX_REGISTRATION` | `tax_registration` | تسجيل ضريبي |
| `TYPE_SHAREHOLDER_REGISTER` | `shareholder_register` | سجل المساهمين |
| `TYPE_BOARD_RESOLUTION` | `board_resolution` | قرار مجلس إدارة |
| `TYPE_FINANCIAL_STATEMENTS` | `financial_statements` | قوائم مالية |
| `TYPE_PROOF_OF_ADDRESS_BUSINESS` | `proof_of_address_business` | إثبات عنوان تجاري |
| `TYPE_UBO_DECLARATION` | `ubo_declaration` | إقرار المستفيد الحقيقي |
| `TYPE_OWNERSHIP_STRUCTURE_CHART` | `ownership_structure_chart` | هيكل الملكية |
| `TYPE_TRUST_DEED` | `trust_deed` | صك ائتمان |
| `TYPE_SELFIE` | `selfie` | صورة شخصية (سيلفي) |
| `TYPE_LIVENESS_CHECK` | `liveness_check` | فحص الحيوية |
| `TYPE_VIDEO_VERIFICATION` | `video_verification` | تحقق بالفيديو |
| `TYPE_SIGNATURE_SPECIMEN` | `signature_specimen` | نموذج توقيع |

### فئات الوثائق (`CATEGORY_*`)

| الثابت | القيمة | الوصف |
| :--- | :--- | :--- |
| `CATEGORY_PRIMARY_ID` | `primary_id` | هويات أساسية |
| `CATEGORY_SECONDARY_ID` | `secondary_id` | هويات ثانوية |
| `CATEGORY_ADDRESS` | `address` | إثبات عنوان |
| `CATEGORY_CORPORATE` | `corporate` | وثائق شركات |
| `CATEGORY_UBO` | `ubo` | المستفيد الحقيقي |
| `CATEGORY_ADDITIONAL` | `additional` | تحقق إضافي |

---

## دوال جلب الأنواع والفئات

### `getAllTypeCodes(): array`
ترجع قائمة بجميع أكواد أنواع الوثائق.

```php
$codes = DocumentType::getAllTypeCodes();
// ['passport', 'national_id', 'drivers_license', ...]
```

### `getCategories(): array`
ترجع قائمة الفئات مع أسمائها المترجمة.

```php
$categories = DocumentType::getCategories();
// ['primary_id' => 'هوية أساسية', 'address' => 'إثبات عنوان', ...]
```

### `getAllTypesFlat(): array`
ترجع مصفوفة مسطحة `[code => localized_name]` لجميع الأنواع.

```php
$flat = DocumentType::getAllTypesFlat();
// ['passport' => 'جواز سفر', 'national_id' => 'بطاقة هوية', ...]
```

### `getAllTypesGrouped(bool $includeCategoryNames = true): array`
ترجع الأنواع مجمعة حسب الفئة.

```php
$grouped = DocumentType::getAllTypesGrouped();
/*
[
    'هوية أساسية' => ['passport' => 'جواز سفر', ...],
    'إثبات عنوان' => ['utility_bill' => 'فاتورة مرافق', ...],
]
*/
```

### `getTypesByCategory(string $category): array`
ترجع أنواع فئة محددة.

```php
$addressTypes = DocumentType::getTypesByCategory(DocumentType::CATEGORY_ADDRESS);
// ['utility_bill' => 'فاتورة مرافق', 'bank_statement' => 'كشف حساب', ...]
```

### `getOptionsForSelect(bool $grouped = false, ?string $categoryFilter = null): array`
تجهيز خيارات لقوائم منسدلة.

```php
// قائمة مسطحة
$options = DocumentType::getOptionsForSelect();

// مجمعة حسب الفئة
$options = DocumentType::getOptionsForSelect(true);

// أنواع العناوين فقط
$options = DocumentType::getOptionsForSelect(false, DocumentType::CATEGORY_ADDRESS);
```

---

## دوال معلومات الوثيقة المفصلة

### `getTypeDetails(string $type): ?array`
ترجع معلومات شاملة عن نوع الوثيقة.

```php
$details = DocumentType::getTypeDetails('passport');
/*
[
    'code' => 'passport',
    'name' => 'جواز سفر',
    'category' => 'primary_id',
    'has_expiry' => true,
    'verification_weight' => 100,
    'required_fields' => ['document_front', 'document_number', ...],
    'allowed_mime_types' => ['image/jpeg', 'image/png', 'application/pdf'],
    'fields' => [ ... ], // مخطط الحقول الكامل
    ...
]
*/
```

### `isValidType(?string $type): bool`
التحقق من صلاحية كود النوع.

```php
DocumentType::isValidType('passport');     // true
DocumentType::isValidType('invalid');      // false
```

### `getCategory(string $type): ?string`
ترجع كود الفئة.

```php
$cat = DocumentType::getCategory('utility_bill'); // 'address'
```

### `getTypeName(string $type, ?string $locale = null): string`
الاسم المترجم للنوع.

```php
$name = DocumentType::getTypeName('passport', 'ar'); // 'جواز سفر'
```

### `getTypeDescription(string $type, ?string $locale = null): string`
الوصف المترجم للنوع.

### `getConstantName(string $type): ?string`
اسم الثابت البرمجي المقابل (مثال: `TYPE_PASSPORT`).

---

## دوال الخصائص المتقدمة

| الدالة | الوصف | مثال |
| :--- | :--- | :--- |
| `getRequiredFields(string $type): array` | الحقول الإلزامية | `['document_front', ...]` |
| `getAllowedMimeTypes(string $type): array` | أنواع الملفات المسموحة | `['image/jpeg', 'application/pdf']` |
| `getMaxFileSizeKb(string $type): int` | الحد الأقصى للحجم بالكيلوبايت | `10240` |
| `requiresBackSide(string $type): bool` | هل يتطلب ظهر الوثيقة؟ | `true` / `false` |
| `requiresSelfieMatch(string $type): bool` | هل يتطلب مطابقة سيلفي؟ | `true` / `false` |
| `getExtractionRules(string $type): array` | قواعد استخراج OCR | `['document_number' => [...]]` |
| `getVerificationWeight(string $type): int` | وزن الوثيقة في التقييم (0-100) | `100` |
| `hasExpiry(string $type): bool` | هل تنتهي صلاحيتها؟ | `true` / `false` |
| `getMaxAgeDays(string $type): ?int` | الحد الأقصى للعمر بالأيام | `90` |

---

## دوال صلاحية الوثائق

### `isDocumentAcceptable(string $type, $date): bool`
تتحقق مما إذا كانت الوثيقة ما زالت مقبولة (لم تنته صلاحيتها ولم تتجاوز الحد الأقصى للعمر).

```php
// جواز سفر ينتهي في 2028
$acceptable = DocumentType::isDocumentAcceptable('passport', '2028-12-31'); // true

// فاتورة صادرة منذ 120 يومًا (أقدم من 90 يوم)
$acceptable = DocumentType::isDocumentAcceptable('utility_bill', Carbon::now()->subDays(120)); // false
```

### `isDocumentExpiring(string $type, $date, int $warningDays = 30): bool`
تتحقق مما إذا كانت الوثيقة منتهية أو على وشك الانتهاء.

```php
$expiring = DocumentType::isDocumentExpiring('passport', Carbon::now()->addDays(20), 30); // true
```

---

## دوال مخطط الحقول (Fields Schema)

### `getFieldsSchema(string $type): array`
ترجع مخطط الحقول الكامل للنوع، متضمنًا التسميات، الأنواع، الإلزامية، وقواعد التحقق.

```php
$schema = DocumentType::getFieldsSchema('passport');
/*
[
    'document_front' => [
        'label' => 'صورة الجواز',
        'type' => 'file',
        'required' => true,
        'validation' => ['mimes' => ['jpeg', 'jpg', 'png', 'pdf'], 'max_size_kb' => 10240],
        'order' => 10,
    ],
    'document_number' => [
        'label' => 'رقم الجواز',
        'type' => 'text',
        'required' => true,
        'validation' => ['max_length' => 50],
        'extractable' => true,
        'order' => 30,
    ],
    // ... جميع الحقول
]
*/
```

### `getFieldDefinition(string $type, string $field): ?array`
ترجع تعريف حقل محدد.

```php
$def = DocumentType::getFieldDefinition('passport', 'expiry_date');
```

### `getFieldLabel(string $type, string $field, ?string $locale = null): string`
التسمية المترجمة لحقل معين.

```php
$label = DocumentType::getFieldLabel('passport', 'expiry_date', 'ar'); // 'تاريخ الانتهاء'
```

### `getAllFieldCodes(): array`
جميع أكواد الحقول المستخدمة عبر كل الأنواع.

---

## دوال التحقق (Validation)

### `getValidationRules(string $type): array`
ترجع قواعد التحقق لجميع حقول نوع الوثيقة، جاهزة للاستخدام مع `Validator`.

```php
$rules = DocumentType::getValidationRules('passport');
/*
[
    'document_front' => ['required', 'file', 'mimes:jpeg,jpg,png,pdf', 'max:10240'],
    'document_number' => ['required', 'string', 'max:50'],
    'expiry_date' => ['required', 'date', 'after_or_equal:today'],
    ...
]
*/
```

### `getValidationRulesForField(string $type, string $field): array`
قواعد التحقق لحقل واحد فقط.

```php
$rules = DocumentType::getValidationRulesForField('passport', 'expiry_date');
// ['required', 'date', 'after_or_equal:today']
```

---

## دوال تعيين البيانات بين المدخلات وقاعدة البيانات

تسهل هذه الدوال عملية تحويل البيانات من نموذج الإدخال إلى هيكل قاعدة البيانات (أعمدة منفصلة + JSON) والعكس.

### `getFieldColumnMapping(): array`
ترجع خريطة بأسماء الحقول التي لها أعمدة فعلية في جدول `nano3_kyc_documents`.

```php
$map = DocumentType::getFieldColumnMapping();
// ['document_number' => 'document_number', 'full_name' => 'full_name', ...]
```

### `mapInputToDocumentData(string $type, array $input): array`
تحول بيانات الإدخال الخام (مثل `$request->all()`) إلى هيكل مناسب للحفظ.

**المخرجات:** مصفوفة تحتوي على:
- `attributes`: قيم الأعمدة الفعلية.
- `fields_data`: القيم التي ستخزن في حقل JSON.

**مثال:**
```php
$input = [
    'document_number' => 'A12345678',
    'full_name' => 'John Doe',
    'expiry_date' => '2030-01-01',
    'passport_type' => 'P', // حقل مخصص غير موجود كعمود
];

$mapped = DocumentType::mapInputToDocumentData('passport', $input);

/*
[
    'attributes' => [
        'document_type' => 'passport',
        'document_category' => 'primary_id',
        'document_number' => 'A12345678',
        'full_name' => 'John Doe',
        'expiry_date' => '2030-01-01',
    ],
    'fields_data' => [
        'passport_type' => 'P',
    ],
]
*/

// استخدامها مع الموديل
$document = new Document();
$document->fill($mapped['attributes']);
$document->fields_data = $mapped['fields_data'];
$document->save();
```

### `mapDocumentToOutput(string $type, $document): array`
العملية العكسية: تأخذ كائن الموديل وتعيد مصفوفة موحدة تحتوي على جميع الحقول (من الأعمدة + من `fields_data`). مفيدة لتعبئة نماذج التعديل.

```php
$document = Document::find(1);
$formData = DocumentType::mapDocumentToOutput('passport', $document);

/*
[
    'document_number' => 'A12345678',
    'full_name' => 'John Doe',
    'expiry_date' => '2030-01-01',
    'passport_type' => 'P',
    // ... جميع الحقول الأخرى بقيمها أو null
]
*/
```

---

## دوال التوصيات والأنواع الإلزامية

### `isValidForPartyType(string $docType, string $partyType): bool`
هل نوع الوثيقة صالح لنوع الطرف (`individual`, `corporate`, `trust`)؟

```php
DocumentType::isValidForPartyType('certificate_of_incorporation', 'individual'); // false
DocumentType::isValidForPartyType('certificate_of_incorporation', 'corporate');  // true
```

### `getRecommendedTypes(string $partyType, string $riskLevel): array`
ترجع أنواع الوثائق الموصى بها بناءً على نوع الطرف ومستوى المخاطرة (`low`, `medium`, `high`).

```php
$recommended = DocumentType::getRecommendedTypes('individual', 'high');
// ['passport', 'national_id', 'utility_bill', ...]
```

### `getMandatoryTypesForParty(string $partyType, string $riskLevel): array`
الأنواع الإلزامية التي يجب تقديمها.

```php
$mandatory = DocumentType::getMandatoryTypesForParty('corporate', 'high');
// ['certificate_of_incorporation', 'tax_registration', 'ubo_declaration', ...]
```

---

## دوال التخزين المؤقت

### `clearCache(): void`
تمسح الذاكرة المؤقتة الداخلية للإعدادات والتعريفات. تستخدم بعد تحديث ملفات التكوين في وقت التشغيل.

```php
DocumentType::clearCache();
```

---

## التهيئة عبر ملف الإعدادات

يمكن تخصيص أي خاصية من خصائص أنواع الوثائق عبر ملف `config/nano3/kyc/document_types.php`. القيم الموجودة في الملف ستتجاوز القيم الافتراضية البرمجية.

**مثال: ملف `config/nano3/kyc/document_types.php`**

```php
<?php
return [
    'passport' => [
        'max_file_size_kb' => 5120, // 5MB بدلاً من 10MB
        'requires_selfie_match' => false,
        'fields' => [
            'expiry_date' => ['required' => true],
            'document_back' => null, // إخفاء حقل ظهر الجواز
            'religion' => ['required' => true], // جعل الديانة إلزامية
        ],
    ],
    'utility_bill' => [
        'max_age_days' => 60, // تقليل الصلاحية إلى 60 يومًا
    ],
];
```

بعد حفظ الملف، استدعاء `DocumentType::getTypeDetails('passport')` سيعكس القيم الجديدة تلقائيًا (بعد مسح الكاش إذا لزم الأمر).

---

## أمثلة عملية متكاملة

### 1. حفظ وثيقة جديدة من طلب HTTP

```php
use Nano3\Kyc\Classes\DocumentType;
use Nano3\Kyc\Models\Document;
use Validator;

public function store(Request $request)
{
    $type = $request->input('document_type');
    
    // 1. التحقق من صحة النوع
    if (!DocumentType::isValidType($type)) {
        throw new \Exception('نوع وثيقة غير صالح');
    }
    
    // 2. جلب قواعد التحقق والتحقق من المدخلات
    $rules = DocumentType::getValidationRules($type);
    $validator = Validator::make($request->all(), $rules);
    $validated = $validator->validate();
    
    // 3. تحويل البيانات إلى هيكل قاعدة البيانات
    $mapped = DocumentType::mapInputToDocumentData($type, $validated);
    
    // 4. إنشاء وحفظ الوثيقة
    $document = new Document();
    $document->fill($mapped['attributes']);
    $document->fields_data = $mapped['fields_data'];
    $document->owner_id = auth()->id();
    $document->owner_type = get_class(auth()->user());
    $document->status = 'pending';
    $document->save();
    
    // 5. التعامل مع الملفات المرفوعة (إذا وجدت)
    if ($request->hasFile('document_front')) {
        $document->document_front()->create(['data' => $request->file('document_front')]);
    }
    
    return redirect()->back()->with('success', 'تم حفظ الوثيقة بنجاح');
}
```

### 2. عرض نموذج تعديل وثيقة

```php
public function edit($id)
{
    $document = Document::findOrFail($id);
    $type = $document->document_type;
    
    // الحصول على البيانات الموحدة (أعمدة + JSON)
    $formData = DocumentType::mapDocumentToOutput($type, $document);
    
    // الحصول على مخطط الحقول لبناء النموذج
    $fieldsSchema = DocumentType::getFieldsSchema($type);
    
    return view('documents.edit', compact('document', 'formData', 'fieldsSchema'));
}
```

**في قالب Blade:**
```blade
<form method="POST">
    @foreach($fieldsSchema as $fieldCode => $fieldDef)
        <label>{{ DocumentType::getFieldLabel($type, $fieldCode) }}</label>
        
        @if($fieldDef['type'] === 'text')
            <input type="text" name="{{ $fieldCode }}" value="{{ $formData[$fieldCode] ?? '' }}">
        @elseif($fieldDef['type'] === 'date')
            <input type="date" name="{{ $fieldCode }}" value="{{ $formData[$fieldCode] ?? '' }}">
        @elseif($fieldDef['type'] === 'file')
            <input type="file" name="{{ $fieldCode }}">
        @endif
        <!-- إلخ... -->
    @endforeach
    <button type="submit">تحديث</button>
</form>
```

### 3. التحقق من صلاحية وثيقة قبل قبولها

```php
$document = Document::find($id);
$type = $document->document_type;

if (DocumentType::hasExpiry($type)) {
    $expiry = $document->expiry_date;
    if (!DocumentType::isDocumentAcceptable($type, $expiry)) {
        throw new \Exception('الوثيقة منتهية الصلاحية');
    }
} elseif ($maxAge = DocumentType::getMaxAgeDays($type)) {
    $issue = $document->issue_date;
    if (!DocumentType::isDocumentAcceptable($type, $issue)) {
        throw new \Exception('الوثيقة أقدم من الحد المسموح به');
    }
}
```

### 4. بناء قائمة منسدلة ديناميكية لاختيار نوع الوثيقة

```blade
<select name="document_type" class="form-control">
    @foreach(DocumentType::getOptionsForSelect(true) as $category => $types)
        <optgroup label="{{ $category }}">
            @foreach($types as $code => $name)
                <option value="{{ $code }}">{{ $name }}</option>
            @endforeach
        </optgroup>
    @endforeach
</select>
```

---

## ملاحظات هامة

- **التخزين المؤقت**: الكلاس يستخدم تخزينًا مؤقتًا داخليًا. أي تغيير في ملف الإعدادات أثناء تنفيذ الطلب لن ينعكس حتى يتم مسح الكاش بـ `DocumentType::clearCache()` أو إعادة تحميل الصفحة في بيئة التطوير.
- **ملفات اللغة**: يجب توفير مفاتيح الترجمة في `plugins/nano3/kyc/lang/ar/lang.php` تحت المفتاح `nano3.kyc::lang.document_types`.
- **القيم الافتراضية**: في حال عدم وجود ترجمة أو إعداد، سيتم الرجوع إلى قيم برمجية منسقة.

بهذا التوثيق الشامل يمكن للمطورين الاستفادة الكاملة من إمكانيات كلاس `DocumentType` لبناء نظام KYC مرن وقابل للتوسع.
