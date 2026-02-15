# توثيق كلاس DocumentTypes

## 📋 نظرة عامة

`DocumentTypes` هو كلاس مركزي ومهني لإدارة أنواع الوثائق القانونية في نظام Nano2soft (إضافة Nano3.Legal). يوفر هذا الكلاس بنية منظمة ومرنة لتحديد وتصنيف واسترجاع أنواع الوثائق القانونية المختلفة مع دعم كامل للترجمة متعددة اللغات، بالإضافة إلى توفير محتوى افتراضي جاهز للوثائق عبر `DefaultContentDocumentTypes` trait.

## 🎯 الهدف من الكلاس

- **توحيد أنواع الوثائق**: إنشاء مرجع مركزي لجميع أنواع الوثائق في النظام
- **تسهيل الصيانة**: تحديث وإضافة أنواع جديدة في مكان واحد فقط
- **تحسين الأداء**: تقليل الاستعلامات لقاعدة البيانات
- **تعزيز التجربة**: توفير واجهة سهلة الاستخدام للمطورين
- **توفير محتوى افتراضي**: إنشاء وثائق جاهزة بشكل احترافي

## ✨ المزايا الرئيسية

### 1. **المركزية والتنظيم**
- جميع أنواع الوثائق في مكان واحد
- مقسمة إلى 8 فئات رئيسية منطقية
- سهولة الوصول والاستخدام

### 2. **القابلية للصيانة**
- إضافة أنواع جديدة بسهولة
- تحديث الترجمات مرة واحدة فقط
- عدم وجود تكرار في التعريفات

### 3. **الأمان والدقة**
- استخدام الثوابت (Constants) للحد من الأخطاء
- التحقق من صحة الأنواع
- منع إدخال أنواع غير معروفة

### 4. **التجربة المطورة**
- واجهة برمجة تطبيقات (API) بديهية
- دعم متعدد اللغات
- إرجاع معلومات غنية عن كل نوع

### 5. **محتوى افتراضي متكامل**
- توفير محتوى HTML احترافي جاهز للوثائق الأكثر شيوعاً
- قوالب عامة قابلة للتخصيص لبقية الأنواع
- تواريخ ديناميكية وتنسيق متسق

## 📊 الإحصائيات

| البند | العدد |
|--------|--------|
| إجمالي أنواع الوثائق (ref_type) | 60 نوعاً |
| عدد الفئات (document types) | 8 فئات |
| وثائق التجارة الإلكترونية | 10 وثائق |
| وثائق التطبيقات والتأجير | 8 وثائق |
| وثائق الطلبات والعقود | 8 وثائق |
| وثائق المستخدمين والتسجيل | 8 وثائق |
| وثائق التوظيف والعمل | 5 وثائق |
| وثائق الأمن والحماية | 5 وثائق |
| وثائق عامة | 8 وثائق |
| وثائق صناعية | 8 وثائق |
| أنواع العمليات (process_type) | 4 أنواع |
| أنواع التشغيل (operation_type) | 2 نوع |

## 🏗️ الهيكل التنظيمي

### الفئات الرئيسية:
1. **التجارة الإلكترونية** (E-commerce) - `TYPE_ECOMMERCE`
2. **التطبيقات والتأجير** (Rental) - `TYPE_RENTAL`
3. **الطلبات والعقود** (Contracts) - `TYPE_CONTRACT`
4. **المستخدمين والتسجيل** (Users) - `TYPE_USER`
5. **التوظيف والعمل** (Employment) - `TYPE_EMPLOYMENT`
6. **الأمن والحماية** (Security) - `TYPE_SECURITY`
7. **وثائق عامة** (General) - `TYPE_GENERAL`
8. **وثائق صناعية** (Industry) - `TYPE_INDUSTRY`

## 🛠️ طرق الاستخدام

### 1. **الاستخدام الأساسي**

```php
use Nano3\Legal\Classes\DocumentTypes;

// الحصول على جميع الأنواع (كمصفوفة بسيطة)
$allTypes = DocumentTypes::getAllTypes();

// الحصول على الأنواع مصنفة حسب الفئة
$categorizedTypes = DocumentTypes::getAllTypes(true);

// الحصول على الأنواع حسب فئة معينة
$ecommerceTypes = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_ECOMMERCE);
```

### 2. **القوائم المنسدلة في النماذج**

```php
// في سمة FieldsOptions
public function getRefTypeOptions($value = null, $formData = null)
{
    return DocumentTypes::getRefTypeOptionsForSelect();
}

// مع التجميع حسب الفئات
public function getRefTypeOptionsGrouped($value = null, $formData = null)
{
    return DocumentTypes::getRefTypeOptionsForSelect(true);
}

// خيارات فئات الوثائق
public function getDocumentTypeOptions($value = null, $formData = null)
{
    return DocumentTypes::getTypeOptionsForSelect();
}
```

### 3. **التحقق والتحليلات**

```php
// التحقق من صحة نوع الوثيقة (ref_type)
if (DocumentTypes::isValidRefType('privacy_policy')) {
    // النوع صالح
}

// التحقق من صحة الفئة (type)
if (DocumentTypes::isValidType('ecommerce')) {
    // الفئة صالحة
}

// الحصول على معلومات كاملة عن نوع الوثيقة
$typeInfo = DocumentTypes::getRefTypeInfo('privacy_policy');
// النتيجة: ['code' => 'privacy_policy', 'name' => 'سياسة الخصوصية', 'type' => 'ecommerce', ...]

// الحصول على الفئة التي ينتمي إليها نوع معين
$category = DocumentTypes::getCategoryByRefType('privacy_policy');

// الحصول على الأنواع الأساسية (الأكثر استخداماً)
$essentialTypes = DocumentTypes::getEssentialRefTypes();
```

### 4. **الاستخدام في المشاهد (Views)**

```blade
<!-- في نموذج اختيار نوع المستند مع تجميع حسب الفئة -->
<select name="ref_type">
    @foreach(DocumentTypes::getRefTypeOptionsForSelect(true) as $category => $types)
        <optgroup label="{{ $category }}">
            @foreach($types as $value => $label)
                <option value="{{ $value }}">{{ $label }}</option>
            @endforeach
        </optgroup>
    @endforeach
</select>

<!-- قائمة بسيطة -->
<select name="ref_type">
    @foreach(DocumentTypes::getAllTypes() as $value => $label)
        <option value="{{ $value }}">{{ $label }}</option>
    @endforeach
</select>
```

### 5. **الاستخدام مع الموافقات (Consents)**

```php
// الحصول على أنواع العمليات (process types) للموافقات
$processTypes = DocumentTypes::getProcessTypes();
// النتيجة: ['add' => 'إضافة', 'remove' => 'إزالة', ...]

// الحصول على أنواع التشغيل (operation types)
$operationTypes = DocumentTypes::getOperationTypes();
// النتيجة: ['manual' => 'يدوي', 'auto' => 'تلقائي']
```

### 6. **المحتوى الافتراضي للوثائق**

```php
// الحصول على محتوى افتراضي لنوع معين (إصدار أحدث)
$content = DocumentTypes::getDefaultContentByRefType('privacy_policy');

// الحصول على محتوى عام لأي نوع آخر
$generalContent = DocumentTypes::getGeneralDocumentContent('some_other_type');

// الحصول على وصف مختصر من ملفات اللغة
$description = DocumentTypes::getDefaultDescriptionsByRefType('privacy_policy', 'ar');
```

## 📝 قائمة الثوابت المتاحة

### أنواع الوثائق الأساسية (REF_TYPE_*):
```php
// التجارة الإلكترونية
DocumentTypes::REF_TYPE_PRIVACY_POLICY          // سياسة الخصوصية
DocumentTypes::REF_TYPE_TERMS_OF_SERVICE        // شروط الخدمة
DocumentTypes::REF_TYPE_RETURN_POLICY           // سياسة الإرجاع
DocumentTypes::REF_TYPE_SHIPPING_POLICY         // سياسة الشحن
DocumentTypes::REF_TYPE_CANCELLATION_POLICY     // سياسة الإلغاء
DocumentTypes::REF_TYPE_WARRANTY_POLICY         // سياسة الضمان
DocumentTypes::REF_TYPE_PAYMENT_TERMS           // شروط الدفع
DocumentTypes::REF_TYPE_COOKIE_POLICY           // سياسة الكوكيز
DocumentTypes::REF_TYPE_AFFILIATE_AGREEMENT     // اتفاقية العمولة
DocumentTypes::REF_TYPE_PARTNERSHIP_AGREEMENT   // اتفاقية الشراكة

// ... وهكذا لبقية الفئات
```

### فئات الوثائق (TYPE_*):
```php
DocumentTypes::TYPE_ECOMMERCE    // التجارة الإلكترونية
DocumentTypes::TYPE_RENTAL       // التطبيقات والتأجير
DocumentTypes::TYPE_CONTRACT     // الطلبات والعقود
DocumentTypes::TYPE_USER         // المستخدمين والتسجيل
DocumentTypes::TYPE_EMPLOYMENT   // التوظيف والعمل
DocumentTypes::TYPE_SECURITY     // الأمن والحماية
DocumentTypes::TYPE_GENERAL      // وثائق عامة
DocumentTypes::TYPE_INDUSTRY     // وثائق صناعية
```

### أنواع العمليات (PROCESS_TYPE_*):
```php
DocumentTypes::PROCESS_TYPE_ADD      // إضافة
DocumentTypes::PROCESS_TYPE_REMOVE   // إزالة
DocumentTypes::PROCESS_TYPE_OUT      // خروج
DocumentTypes::PROCESS_TYPE_RESET    // إعادة تعيين
// (ملاحظة: PROCESS_TYPE_LEVEL_UP و PROCESS_TYPE_WINNER موجودة لكنها معلقة)
```

### أنواع التشغيل (OPERATION_TYPE_*):
```php
DocumentTypes::OPERATION_TYPE_MANUAL  // يدوي
DocumentTypes::OPERATION_TYPE_AUTO    // تلقائي
```

## 🔧 الطرق المتاحة

| الطريقة | الوصف | المثال |
|---------|--------|---------|
| `getAllRefTypes($includeCategories = false)` | جميع أنواع الوثائق (ref_type) | `DocumentTypes::getAllRefTypes(true)` |
| `getAllTypes($includeCategories = false)` | مرادف لـ getAllRefTypes | `DocumentTypes::getAllTypes()` |
| `findRefType($id, $locale = null)` | البحث عن تسمية نوع معين | `DocumentTypes::findRefType('privacy_policy')` |
| `getAllTypesOptions($locale = null, $is_key_val = false)` | خيارات جميع الأنواع | `DocumentTypes::getAllTypesOptions()` |
| `groupByCategories($refTypes)` | تجميع الأنواع حسب الفئة | `DocumentTypes::groupByCategories($types)` |
| `getTypesByCategory($category)` | الأنواع حسب الفئة | `DocumentTypes::getTypesByCategory('ecommerce')` |
| `getDocumentTypes()` | جميع فئات الوثائق | `DocumentTypes::getDocumentTypes()` |
| `findDocumentType($id, $locale = null)` | البحث عن تسمية فئة | `DocumentTypes::findDocumentType('ecommerce')` |
| `getAllDocumentTypes($locale = null, $is_key_val = false)` | جميع الفئات | `DocumentTypes::getAllDocumentTypes()` |
| `getCategoryName($category)` | اسم الفئة المترجم | `DocumentTypes::getCategoryName('ecommerce')` |
| `getCategoryOptions()` | خيارات الفئات للقوائم | `DocumentTypes::getCategoryOptions()` |
| `getProcessTypes()` | أنواع العمليات للموافقات | `DocumentTypes::getProcessTypes()` |
| `getOperationTypes()` | أنواع التشغيل | `DocumentTypes::getOperationTypes()` |
| `getRefTypeInfo($refType)` | معلومات كاملة عن نوع الوثيقة | `DocumentTypes::getRefTypeInfo('privacy_policy')` |
| `getRefTypeConstantName($refType)` | اسم الثابت للنوع | `DocumentTypes::getRefTypeConstantName('privacy_policy')` |
| `getCategoryByRefType($refType)` | الفئة التي ينتمي إليها النوع | `DocumentTypes::getCategoryByRefType('privacy_policy')` |
| `getEssentialTypes()` | الأنواع الأساسية (قديمة) | `DocumentTypes::getEssentialTypes()` |
| `getEssentialRefTypes()` | الأنواع الأساسية (محدثة) | `DocumentTypes::getEssentialRefTypes()` |
| `getEssentialRefTypesOptions($locale = null, $is_key_val = false)` | خيارات الأنواع الأساسية | `DocumentTypes::getEssentialRefTypesOptions()` |
| `isValidRefType($refType)` | التحقق من صحة نوع الوثيقة | `DocumentTypes::isValidRefType('privacy_policy')` |
| `isValidType($type)` | التحقق من صحة الفئة | `DocumentTypes::isValidType('ecommerce')` |
| `getRefTypeOptionsForSelect($groupByCategory = false)` | خيارات للقوائم المنسدلة | `DocumentTypes::getRefTypeOptionsForSelect(true)` |
| `getTypeOptionsForSelect()` | خيارات الفئات | `DocumentTypes::getTypeOptionsForSelect()` |
| `getProcessTypeOptionsForSelect()` | خيارات أنواع العمليات | `DocumentTypes::getProcessTypeOptionsForSelect()` |
| `getOperationTypeOptionsForSelect()` | خيارات أنواع التشغيل | `DocumentTypes::getOperationTypeOptionsForSelect()` |
| `getRefTypesByCategory($category)` | أنواع وثائق حسب الفئة | `DocumentTypes::getRefTypesByCategory('ecommerce')` |
| `getOptionsForSelect($groupByCategory = false)` | مرادف لـ getRefTypeOptionsForSelect | `DocumentTypes::getOptionsForSelect(true)` |
| `getRefTypesCount()` | عدد أنواع الوثائق | `DocumentTypes::getRefTypesCount()` |
| `getConstantName($ref_type)` | اسم الثابت | `DocumentTypes::getConstantName('privacy_policy')` |
| `getDefaultDescriptionsByRefType($refType, $locale = null, $defaultValue = null)` | وصف مختصر افتراضي | `DocumentTypes::getDefaultDescriptionsByRefType('privacy_policy', 'ar')` |
| `getDefaultDescriptionsLangByRefType($refType, $locale = null, $defaultValue = null)` | وصف بلغة محددة | `DocumentTypes::getDefaultDescriptionsLangByRefType('privacy_policy', 'en')` |
| `getDefaultConditionsByRefType($refType)` | شروط افتراضية مختصرة | `DocumentTypes::getDefaultConditionsByRefType('privacy_policy')` |

### طرق المحتوى الافتراضي (من trait DefaultContentDocumentTypes)

| الطريقة | الوصف |
|---------|--------|
| `getDefaultContentByRefTypeV1($refType)` | محتوى افتراضي قديم (أساسي) |
| `getDefaultContentByRefType($refType)` | محتوى افتراضي محدث (احترافي) |
| `getPrivacyPolicyContent()` | محتوى سياسة الخصوصية |
| `getTermsOfServiceContent()` | محتوى شروط الخدمة |
| `getReturnPolicyContent()` | محتوى سياسة الإرجاع |
| `getShippingPolicyContent()` | محتوى سياسة الشحن |
| `getUserAgreementContent()` | محتوى اتفاقية المستخدم |
| `getCookiePolicyContent()` | محتوى سياسة الكوكيز |
| `getGeneralDocumentContent($refType)` | محتوى عام لأي نوع آخر |

## 🚀 حالات الاستخدام العملية

### 1. **نظام التجارة الإلكترونية**
```php
// عند إنشاء مستند جديد
$ecommerceDocs = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_ECOMMERCE);
// النتيجة: سياسة الخصوصية، شروط الخدمة، سياسة الإرجاع، إلخ (10 وثائق)

// إنشاء مستند مع محتوى افتراضي
$document = new Document;
$document->ref_type = DocumentTypes::REF_TYPE_PRIVACY_POLICY;
$document->content = DocumentTypes::getDefaultContentByRefType(DocumentTypes::REF_TYPE_PRIVACY_POLICY);
$document->save();
```

### 2. **نظام إدارة العقود**
```php
// عرض أنواع العقود المتاحة
$contractDocs = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_CONTRACT);
// النتيجة: اتفاقية الشراء، عقد البيع، شروط الفاتورة، إلخ (8 وثائق)

// التحقق من صحة العقد المرفوع
if (DocumentTypes::isValidRefType($input->ref_type)) {
    // معالجة العقد
}
```

### 3. **نظام إدارة المستخدمين**
```php
// وثائق تسجيل المستخدمين
$userDocs = DocumentTypes::getTypesByCategory(DocumentTypes::TYPE_USER);
// النتيجة: اتفاقية المستخدم، شروط التسجيل، مدونة السلوك، إلخ (8 وثائق)

// عرض الأنواع الأساسية فقط للمستخدم الجديد
$essentialDocs = DocumentTypes::getEssentialRefTypesOptions();
```

### 4. **نظام الموافقات (Consents)**
```php
// إنشاء سجل موافقة جديد
$consent = new Consent;
$consent->ref_type = DocumentTypes::REF_TYPE_PRIVACY_POLICY;
$consent->process_type = DocumentTypes::PROCESS_TYPE_ADD; // إضافة
$consent->operation_type = DocumentTypes::OPERATION_TYPE_MANUAL; // يدوي
$consent->save();
```

### 5. **تصدير تقارير تحليلية**
```php
public function getDocumentStats()
{
    $stats = [];
    $categories = DocumentTypes::getDocumentTypes();
    
    foreach (array_keys($categories) as $category) {
        $stats[$category] = [
            'category_name' => DocumentTypes::getCategoryName($category),
            'count' => count(DocumentTypes::getRefTypesByCategory($category)),
            'types' => DocumentTypes::getRefTypesByCategory($category)
        ];
    }
    
    return $stats;
}
```

## 📈 فوائد الأداء

1. **تقليل استعلامات قاعدة البيانات**: لا حاجة لاستعلام `SELECT DISTINCT` في كل مرة
2. **تحسين الذاكرة**: تخزين الذاكرة المؤقتة للأنواع
3. **سرعة الاستجابة**: استرجاع فوري للبيانات
4. **تقليل الأخطاء**: استخدام الثوابت يمنع أخطاء الكتابة

## 🔄 كيفية إضافة أنواع جديدة

### خطوة 1: إضافة الثابت في `DocumentTypes.php`
```php
const REF_TYPE_NEW_DOCUMENT = 'new_document';
```

### خطوة 2: إضافة الترجمة في ملف اللغة (مثل `lang/ar/lang.php`)
```php
'document_types' => [
    'ref_types' => [
        // ... أنواع سابقة
        'new_document' => 'المستند الجديد',
    ],
    'descriptions' => [
        // ... أوصاف سابقة
        'new_document' => 'وصف مختصر للمستند الجديد',
    ],
],
```

### خطوة 3: إضافة إلى الفئة المناسبة في دالة `groupByCategories`
```php
self::TYPE_GENERAL => [
    // ... أنواع سابقة
    self::REF_TYPE_NEW_DOCUMENT => $types[self::REF_TYPE_NEW_DOCUMENT],
],
```

### خطوة 4: (اختياري) إضافة محتوى افتراضي في `DefaultContentDocumentTypes`
```php
public static function getDefaultContentByRefType($refType)
{
    $defaultContent = [
        // ... أنواع سابقة
        self::REF_TYPE_NEW_DOCUMENT => self::getNewDocumentContent(),
    ];
    // ...
}

public static function getNewDocumentContent()
{
    return '<h1>المستند الجديد</h1>...';
}
```

## 🧪 الاختبار والتحقق

```php
class DocumentTypesTest extends \TestCase
{
    public function testAllTypesCount()
    {
        $types = DocumentTypes::getAllRefTypes();
        $this->assertCount(60, $types); // 60 نوعاً
    }
    
    public function testEssentialTypes()
    {
        $essential = DocumentTypes::getEssentialRefTypes();
        $this->assertContains('privacy_policy', $essential);
        $this->assertContains('terms_of_service', $essential);
    }
    
    public function testTypeInfo()
    {
        $info = DocumentTypes::getRefTypeInfo('privacy_policy');
        $this->assertEquals('سياسة الخصوصية', $info['name']);
        $this->assertEquals('ecommerce', $info['type']);
    }
    
    public function testCategoryName()
    {
        $this->assertEquals('التجارة الإلكترونية', DocumentTypes::getCategoryName('ecommerce'));
    }
    
    public function testValidRefType()
    {
        $this->assertTrue(DocumentTypes::isValidRefType('privacy_policy'));
        $this->assertFalse(DocumentTypes::isValidRefType('invalid_type'));
    }
}
```

## 📚 أمثلة متقدمة

### مثال 1: إنشاء وثيقة ديناميكية مع محتوى افتراضي
```php
class DocumentManager
{
    public function createDocument($refType, $title = null)
    {
        if (!DocumentTypes::isValidRefType($refType)) {
            throw new \InvalidArgumentException("نوع وثيقة غير صالح: $refType");
        }
        
        $document = new Document;
        $document->ref_type = $refType;
        $document->title = $title ?: DocumentTypes::getRefTypeInfo($refType)['name'];
        $document->content = DocumentTypes::getDefaultContentByRefType($refType);
        $document->type = DocumentTypes::getCategoryByRefType($refType);
        $document->save();
        
        return $document;
    }
}
```

### مثال 2: فلترة الوثائق حسب الفئة مع خيارات متقدمة
```php
class DocumentFilter
{
    public function getOptionsForBusiness($businessType)
    {
        $options = [];
        
        switch ($businessType) {
            case 'ecommerce':
                $categories = [DocumentTypes::TYPE_ECOMMERCE, DocumentTypes::TYPE_CONTRACT];
                break;
            case 'saas':
                $categories = [DocumentTypes::TYPE_RENTAL, DocumentTypes::TYPE_USER, DocumentTypes::TYPE_SECURITY];
                break;
            default:
                $categories = [DocumentTypes::TYPE_GENERAL];
        }
        
        foreach ($categories as $category) {
            $options[DocumentTypes::getCategoryName($category)] = DocumentTypes::getRefTypesByCategory($category);
        }
        
        return $options;
    }
}
```

### مثال 3: دمج مع نظام الموافقات
```php
class ConsentLogger
{
    public function logConsent($userId, $refType, $processType)
    {
        if (!DocumentTypes::isValidRefType($refType)) {
            throw new \Exception("نوع وثيقة غير معروف");
        }
        
        if (!array_key_exists($processType, DocumentTypes::getProcessTypes())) {
            throw new \Exception("نوع عملية غير صالح");
        }
        
        $consent = new Consent;
        $consent->user_id = $userId;
        $consent->ref_type = $refType;
        $consent->process_type = $processType;
        $consent->operation_type = DocumentTypes::OPERATION_TYPE_AUTO;
        $consent->save();
        
        return $consent;
    }
}
```

## 🎨 أفضل الممارسات

1. **استخدام الثوابت دائماً**: `DocumentTypes::REF_TYPE_PRIVACY_POLICY` بدلاً من النص المباشر
2. **التحقق من الصحة**: `DocumentTypes::isValidRefType($refType)` قبل المعالجة
3. **استخدام المجموعات**: `DocumentTypes::getRefTypeOptionsForSelect(true)` للقوائم المعقدة
4. **التخزين المؤقت**: تخزين النتائج إذا كانت تستخدم بكثرة (مثل `getAllRefTypes`)
5. **التحديث الدوري**: مراجعة الأنواع وإضافة الجديد منها مع التوثيق المناسب
6. **الاستفادة من المحتوى الافتراضي**: استخدام `getDefaultContentByRefType` كنقطة بداية للوثائق الجديدة
7. **توحيد الأسماء**: الالتزام بنمط تسمية موحد للثوابت (`REF_TYPE_*`, `TYPE_*`)

## 🏆 الخلاصة

`DocumentTypes` هو كلاس حيوي وأساسي في نظام إدارة الوثائق القانونية، حيث يوفر:

- **نظام مركزي** لإدارة أنواع الوثائق (60 نوعاً موزعة على 8 فئات)
- **بنية منظمة** تسهل التوسعة والصيانة
- **واجهة بديهية** للمطورين مع العديد من الطرق المساعدة
- **أداء ممتاز** مع دعم كامل للترجمة
- **مرونة عالية** في الاستخدام لأنظمة مختلفة (تجارة إلكترونية، عقود، مستخدمين، إلخ)
- **محتوى افتراضي جاهز** لتسريع التطوير وتوحيد التنسيق
- **دعم للموافقات** عبر أنواع العمليات المختلفة

يعد هذا الكلاس حلاً مثالياً للمشاريع التي تحتاج إلى إدارة شاملة للوثائق القانونية مع الحفاظ على التنظيم وسهولة الصيانة، ويوفر مع الـ `DefaultContentDocumentTypes` حزمة متكاملة لإنشاء وإدارة الوثائق القانونية بكفاءة عالية.

---

