# DocumentType Class Documentation

## نظرة عامة

`Nano3\Kyc\Classes\DocumentType` هو كلاس مركزي لإدارة أنواع وثائق التحقق من الهوية (KYC) داخل إضافة `Nano3.Kyc` لنظام NanoSoft App. يوفر الكلاس تعريفًا موحدًا لجميع أنواع الوثائق، خصائصها، قواعد التحقق، الحقول المطلوبة، وأوزان الموثوقية، مع دعم كامل لتعدد اللغات وإمكانية التخصيص عبر ملفات الإعدادات.

**الميزات الرئيسية:**
- تعريف أكثر من **28 نوع وثيقة** موزعة على **6 فئات**.
- تحميل الإعدادات من ملف `config` مع وجود قيم افتراضية برمجية.
- دعم الترجمة عبر ملفات اللغة.
- دوال مساعدة للتحقق من صلاحية الوثائق واستخراج قواعد التحقق.
- تخزين مؤقت داخلي لتحسين الأداء.

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
| `TYPE_GOVERNMENT_ID` | `government_id` | هوية حكومية أخرى |
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
| `TYPE_SELFIE` | `selfie` | صورة شخصية |
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
| `CATEGORY_ADDITIONAL` | `additional` | وثائق تحقق إضافية |

---

## الدوال الأساسية (Basic Query Methods)

### `getAllTypeCodes(): array`

ترجع قائمة بجميع أكواد أنواع الوثائق المعرفة.

**مثال:**
```php
$types = DocumentType::getAllTypeCodes();
// ['passport', 'national_id', 'drivers_license', ...]
```

---

### `getCategories(): array`

ترجع قائمة بجميع الفئات مع أسمائها المترجمة.

**مثال:**
```php
$categories = DocumentType::getCategories();
/*
[
    'primary_id'   => 'Primary Identity',
    'secondary_id' => 'Secondary Identity',
    'address'      => 'Address Verification',
    'corporate'    => 'Corporate Documents',
    'ubo'          => 'Beneficial Ownership',
    'additional'   => 'Additional Verification',
]
*/
```

---

### `getAllTypesFlat(): array`

ترجع مصفوفة مسطحة من نوع `[code => localized_name]` لجميع أنواع الوثائق.

**مثال:**
```php
$flat = DocumentType::getAllTypesFlat();
// ['passport' => 'Passport', 'national_id' => 'National ID Card', ...]
```

---

### `getAllTypesGrouped(bool $includeCategoryNames = true): array`

ترجع أنواع الوثائق مجمعة حسب الفئة.

**المدخلات:**
| المعامل | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `$includeCategoryNames` | `bool` | `true` | `true`: مفتاح المجموعة هو اسم الفئة المترجم، `false`: كود الفئة. |

**مثال:**
```php
$grouped = DocumentType::getAllTypesGrouped();
/*
[
    'Primary Identity' => [
        'passport' => 'Passport',
        'national_id' => 'National ID Card',
        ...
    ],
    'Address Verification' => [
        'utility_bill' => 'Utility Bill',
        ...
    ],
    ...
]
*/
```

---

### `getTypesByCategory(string $category): array`

ترجع أنواع الوثائق التي تنتمي لفئة محددة.

**المدخلات:**
| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$category` | `string` | كود الفئة (أحد ثوابت `CATEGORY_*`). |

**مثال:**
```php
$addressTypes = DocumentType::getTypesByCategory(DocumentType::CATEGORY_ADDRESS);
/*
[
    'utility_bill' => 'Utility Bill',
    'bank_statement' => 'Bank Statement',
    'tax_document' => 'Tax Document',
    'rental_agreement' => 'Rental Agreement',
    'mortgage_statement' => 'Mortgage Statement',
]
*/
```

---

### `getOptionsForSelect(bool $grouped = false, ?string $categoryFilter = null): array`

تجهيز خيارات لقائمة منسدلة (Dropdown). مفيدة في نماذج الإدارة والواجهات الأمامية.

**المدخلات:**
| المعامل | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `$grouped` | `bool` | `false` | هل ترجع الخيارات مجمعة حسب الفئة؟ |
| `$categoryFilter` | `?string` | `null` | فلترة حسب فئة محددة. |

**مثال 1 – قائمة مسطحة لجميع الأنواع:**
```php
$options = DocumentType::getOptionsForSelect();
// ['passport' => 'Passport', 'national_id' => 'National ID Card', ...]
```

**مثال 2 – مجمعة حسب الفئة:**
```php
$options = DocumentType::getOptionsForSelect(true);
/*
[
    'Primary Identity' => ['passport' => 'Passport', ...],
    'Address Verification' => ['utility_bill' => 'Utility Bill', ...],
]
*/
```

**مثال 3 – أنواع العناوين فقط:**
```php
$options = DocumentType::getOptionsForSelect(false, DocumentType::CATEGORY_ADDRESS);
// ['utility_bill' => 'Utility Bill', 'bank_statement' => 'Bank Statement', ...]
```

---

## دوال معلومات الوثيقة المفصلة

### `getTypeDetails(string $type): ?array`

ترجع معلومات شاملة عن نوع وثيقة محدد، متضمنة الاسم، الوصف، الفئة، الحقول المطلوبة، قواعد التحقق، وغيرها.

**المدخلات:**
| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$type` | `string` | كود نوع الوثيقة. |

**المخرجات:** مصفوفة معلومات أو `null` إذا كان النوع غير صالح.

**مثال:**
```php
$details = DocumentType::getTypeDetails(DocumentType::TYPE_PASSPORT);

/*
[
    'code' => 'passport',
    'name' => 'Passport',
    'description' => 'International passport issued by a national government.',
    'category' => 'primary_id',
    'category_name' => 'Primary Identity',
    'has_expiry' => true,
    'max_age_days' => null,
    'verification_weight' => 100,
    'required_fields' => ['document_front', 'document_number', 'issue_date', 'expiry_date', 'country_of_issue'],
    'allowed_mime_types' => ['image/jpeg', 'image/png', 'application/pdf'],
    'max_file_size_kb' => 10240,
    'allowed_for_entity' => ['individual'],
    'requires_back_side' => false,
    'requires_selfie_match' => true,
    'extraction_rules' => [
        'document_number' => ['type' => 'alphanumeric', 'pattern' => '/^[A-Z0-9<]+$/'],
        'expiry_date' => ['type' => 'date', 'min' => 'today'],
        'country_of_issue' => ['type' => 'country_code'],
    ],
    'is_primary_id' => true,
    'is_address_document' => false,
    'is_corporate_document' => false,
    'is_ubo_document' => false,
    'is_additional' => false,
]
*/
```

---

### `isValidType(?string $type): bool`

التحقق من صلاحية كود نوع الوثيقة.

**مثال:**
```php
DocumentType::isValidType('passport');        // true
DocumentType::isValidType('invalid_type');    // false
```

---

### `getCategory(string $type): ?string`

ترجع كود الفئة التي ينتمي إليها النوع.

```php
$cat = DocumentType::getCategory(DocumentType::TYPE_UTILITY_BILL);
// 'address'
```

---

### `getTypeName(string $type, ?string $locale = null): string`

ترجع الاسم المترجم لنوع الوثيقة.

```php
$name = DocumentType::getTypeName('passport', 'ar');
// 'جواز سفر'
```

---

### `getTypeDescription(string $type, ?string $locale = null): string`

ترجع الوصف المترجم لنوع الوثيقة.

```php
$desc = DocumentType::getTypeDescription('utility_bill');
// 'A recent utility bill showing your name and address...'
```

---

### `getConstantName(string $type): ?string`

ترجع اسم الثابت البرمجي المقابل لكود النوع.

```php
$constant = DocumentType::getConstantName('national_id');
// 'TYPE_NATIONAL_ID'
```

---

## دوال الخصائص المتقدمة (Advanced Properties)

### `getRequiredFields(string $type): array`

ترجع الحقول المطلوب تعبئتها/استخراجها لهذا النوع من الوثائق.

```php
$fields = DocumentType::getRequiredFields(DocumentType::TYPE_PASSPORT);
// ['document_front', 'document_number', 'issue_date', 'expiry_date', 'country_of_issue']
```

---

### `getAllowedMimeTypes(string $type): array`

أنواع الملفات المسموح برفعها.

```php
$mimes = DocumentType::getAllowedMimeTypes('passport');
// ['image/jpeg', 'image/png', 'application/pdf']
```

---

### `getMaxFileSizeKb(string $type): int`

الحد الأقصى لحجم الملف بالكيلوبايت.

```php
$maxKb = DocumentType::getMaxFileSizeKb('passport');
// 10240
```

---

### `requiresBackSide(string $type): bool`

هل تتطلب الوثيقة تحميل صورة الوجه الخلفي؟

```php
DocumentType::requiresBackSide('drivers_license');   // true
DocumentType::requiresBackSide('passport');          // false
```

---

### `requiresSelfieMatch(string $type): bool`

هل تتطلب الوثيقة مطابقة صورة شخصية (Selfie) مع صورة الوثيقة؟

```php
DocumentType::requiresSelfieMatch('passport');   // true
DocumentType::requiresSelfieMatch('utility_bill'); // false
```

---

### `getExtractionRules(string $type): array`

قواعد استخراج البيانات (OCR) لكل حقل. تستخدم في عمليات التحقق الآلي.

```php
$rules = DocumentType::getExtractionRules('passport');
/*
[
    'document_number' => ['type' => 'alphanumeric', 'pattern' => '/^[A-Z0-9<]+$/'],
    'expiry_date' => ['type' => 'date', 'min' => 'today'],
    'country_of_issue' => ['type' => 'country_code'],
]
*/
```

---

### `getVerificationWeight(string $type): int`

وزن الوثيقة في عملية احتساب درجة التحقق (0-100).

```php
$weight = DocumentType::getVerificationWeight('passport');
// 100
```

---

### `hasExpiry(string $type): bool`

هل للوثيقة تاريخ انتهاء صلاحية؟

```php
DocumentType::hasExpiry('passport');      // true
DocumentType::hasExpiry('utility_bill');  // false
```

---

### `getMaxAgeDays(string $type): ?int`

الحد الأقصى لعمر الوثيقة بالأيام (للوثائق غير منتهية الصلاحية مثل فاتورة الكهرباء).

```php
$days = DocumentType::getMaxAgeDays('utility_bill');
// 90
```

---

## دوال صلاحية الوثائق (Document Validity)

### `isDocumentAcceptable(string $type, $date): bool`

تتحقق مما إذا كانت الوثيقة ما زالت مقبولة (لم تنته صلاحيتها ولم تتجاوز الحد الأقصى للعمر).

**المدخلات:**
| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$type` | `string` | كود نوع الوثيقة. |
| `$date` | `string\|DateTimeInterface` | تاريخ الإصدار (للوثائق محدودة العمر) أو تاريخ الانتهاء (للوثائق منتهية الصلاحية). |

**مثال 1 – جواز سفر صالح:**
```php
$expiryDate = '2028-12-31';
$acceptable = DocumentType::isDocumentAcceptable(DocumentType::TYPE_PASSPORT, $expiryDate);
// true
```

**مثال 2 – فاتورة كهرباء قديمة:**
```php
$issueDate = Carbon::now()->subDays(120); // أقدم من 90 يوم
$acceptable = DocumentType::isDocumentAcceptable(DocumentType::TYPE_UTILITY_BILL, $issueDate);
// false
```

---

### `isDocumentExpiring(string $type, $date, int $warningDays = 30): bool`

تتحقق مما إذا كانت الوثيقة منتهية الصلاحية أو على وشك الانتهاء خلال فترة الإنذار.

**المدخلات:**
| المعامل | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `$type` | `string` | - | كود نوع الوثيقة. |
| `$date` | `string\|DateTimeInterface` | - | تاريخ الإصدار/الانتهاء. |
| `$warningDays` | `int` | `30` | عدد الأيام قبل الانتهاء لاعتبارها "على وشك الانتهاء". |

**مثال:**
```php
$expiryDate = Carbon::now()->addDays(20); // ستنتهي بعد 20 يوم
$expiring = DocumentType::isDocumentExpiring(DocumentType::TYPE_PASSPORT, $expiryDate, 30);
// true (لأنها أقل من 30 يوم)
```

---

## دوال التحقق والقواعد (Validation & Rules)

### `getValidationRules(string $type): array`

ترجع قواعد تحقق جاهزة للاستخدام مع `Validator` الخاص بـ Laravel.

**المدخلات:**
| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$type` | `string` | كود نوع الوثيقة. |

**مثال:**
```php
$rules = DocumentType::getValidationRules('passport');
/*
[
    'file' => ['required', 'file', 'mimes:jpeg,jpg,png,pdf', 'max:10240'],
    'document_number' => ['required', 'regex:/^[A-Z0-9<]+$/'],
    'expiry_date' => ['required', 'date', 'after_or_equal:today'],
]
*/
```

**استخدام عملي في `Controller`:**
```php
$rules = DocumentType::getValidationRules($request->input('document_type'));
$validator = Validator::make($request->all(), $rules);
if ($validator->fails()) {
    // handle errors
}
```

---

### `isValidForPartyType(string $docType, string $partyType): bool`

هل نوع الوثيقة صالح لنوع الطرف (فرد/شركة/ائتمان)؟

**المدخلات:**
| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$docType` | `string` | كود نوع الوثيقة. |
| `$partyType` | `string` | نوع الطرف (`individual`, `corporate`, `trust`). |

**مثال:**
```php
DocumentType::isValidForPartyType('certificate_of_incorporation', 'individual'); // false
DocumentType::isValidForPartyType('certificate_of_incorporation', 'corporate');  // true
```

---

## دوال التوصيات والأنواع الإلزامية

### `getRecommendedTypes(string $partyType, string $riskLevel): array`

ترجع قائمة بأنواع الوثائق الموصى بها بناءً على نوع الطرف ومستوى المخاطرة.

**المدخلات:**
| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$partyType` | `string` | نوع الطرف (`individual`, `corporate`, `trust`). |
| `$riskLevel` | `string` | مستوى المخاطرة (`low`, `medium`, `high`). |

**مثال:**
```php
$recommended = DocumentType::getRecommendedTypes('individual', 'high');
// ['passport', 'national_id', 'drivers_license', 'utility_bill', 'bank_statement']
```

---

### `getMandatoryTypesForParty(string $partyType, string $riskLevel): array`

ترجع أنواع الوثائق **الإلزامية** التي يجب تقديمها بغض النظر عن أي عوامل أخرى.

**مثال:**
```php
$mandatory = DocumentType::getMandatoryTypesForParty('corporate', 'high');
// ['certificate_of_incorporation', 'tax_registration', 'ubo_declaration', 'ownership_structure_chart']
```

---

## دوال الحقول والترجمة

### `getFieldLabel(string $fieldCode): string`

ترجع التسمية المترجمة لحقل معين (مثل `document_number` -> "رقم الوثيقة").

```php
$label = DocumentType::getFieldLabel('expiry_date');
// 'Expiry Date' (أو 'تاريخ الانتهاء' حسب اللغة)
```

---

### `getAllFieldCodes(): array`

ترجع جميع أكواد الحقول المستخدمة عبر جميع أنواع الوثائق.

```php
$allFields = DocumentType::getAllFieldCodes();
// ['document_front', 'document_number', 'issue_date', 'expiry_date', 'country_of_issue', ...]
```

---

## دوال إدارة التخزين المؤقت

### `clearCache(): void`

تمسح الذاكرة المؤقتة الداخلية للإعدادات والتعريفات. تستخدم عند الحاجة لإعادة تحميل الإعدادات من ملف `config` (مثلاً بعد تحديث الإعدادات في وقت التشغيل).

```php
DocumentType::clearCache();
```

---

## التهيئة والتخصيص عبر ملف الإعدادات

يمكن تخصيص خصائص أي نوع وثيقة عبر ملف `config/nano3/kyc/document_types.php`. القيم الموجودة في الملف ستتجاوز القيم الافتراضية البرمجية.

**مثال لتخصيص جواز السفر:**
```php
// config/nano3/kyc/document_types.php
return [
    'passport' => [
        'max_file_size_kb' => 5120, // تغيير الحجم الأقصى إلى 5MB
        'requires_selfie_match' => false, // إلغاء شرط مطابقة السيلفي
        'extraction_rules' => [
            'document_number' => ['type' => 'alphanumeric', 'pattern' => '/^[A-Z]{2}[0-9]{7}$/'],
        ],
    ],
];
```

بعد التعديل، استدعاء `DocumentType::getTypeDetails('passport')` سيعكس القيم الجديدة.

---

## مثال متكامل: نموذج رفع وثيقة

```php
use Nano3\Kyc\Classes\DocumentType;
use Carbon\Carbon;
use Validator;

class DocumentUploadController
{
    public function upload(Request $request)
    {
        $docType = $request->input('document_type');
        
        // 1. التحقق من صلاحية النوع
        if (!DocumentType::isValidType($docType)) {
            return response()->json(['error' => 'Invalid document type'], 400);
        }
        
        // 2. الحصول على قواعد التحقق
        $rules = DocumentType::getValidationRules($docType);
        
        $validator = Validator::make($request->all(), $rules);
        if ($validator->fails()) {
            return response()->json(['errors' => $validator->errors()], 422);
        }
        
        // 3. معالجة الملف المرفوع
        $file = $request->file('file');
        $mimeType = $file->getMimeType();
        $sizeKb = $file->getSize() / 1024;
        
        $allowedMimes = DocumentType::getAllowedMimeTypes($docType);
        $maxSize = DocumentType::getMaxFileSizeKb($docType);
        
        if (!in_array($mimeType, $allowedMimes)) {
            return response()->json(['error' => 'File type not allowed'], 400);
        }
        
        if ($sizeKb > $maxSize) {
            return response()->json(['error' => 'File too large'], 400);
        }
        
        // 4. التحقق من صلاحية البيانات المستخرجة (OCR simulated)
        $extractedData = $this->ocrService->extract($file, $docType);
        $issueDate = $extractedData['issue_date'] ?? $extractedData['statement_date'] ?? null;
        
        if ($issueDate && !DocumentType::isDocumentAcceptable($docType, $issueDate)) {
            return response()->json(['error' => 'Document is too old or expired'], 400);
        }
        
        // 5. حفظ الوثيقة
        $document = $this->saveDocument($file, $docType, $extractedData);
        
        return response()->json(['success' => true, 'document_id' => $document->id]);
    }
}
```

---

## مثال: واجهة اختيار نوع الوثيقة

```php
// في قالب Blade
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

**الناتج (HTML مبسط):**
```html
<select name="document_type">
    <optgroup label="Primary Identity">
        <option value="passport">Passport</option>
        <option value="national_id">National ID Card</option>
        ...
    </optgroup>
    <optgroup label="Address Verification">
        <option value="utility_bill">Utility Bill</option>
        ...
    </optgroup>
</select>
```

---

## ملاحظات هامة

- **التخزين المؤقت**: الكلاس يستخدم تخزينًا مؤقتًا داخليًا (`static`). أي تغيير في ملف الإعدادات أثناء تنفيذ الطلب لن ينعكس حتى يتم مسح الكاش بـ `DocumentType::clearCache()` أو إعادة تحميل الصفحة في بيئة التطوير.
- **ملفات اللغة**: تأكد من وجود مفاتيح الترجمة في `plugins/nano3/kyc/lang/ar/lang.php` و `en/lang.php` تحت المفتاح `nano3.kyc::lang.document_types`.
- **القيم الافتراضية**: في حال عدم وجود ترجمة أو إعداد، سيتم الرجوع إلى القيم البرمجية المنسقة.

---

بهذا نكون قد وثقنا الكلاس `DocumentType` بشكل شامل مع أمثلة عملية تغطي معظم حالات الاستخدام.