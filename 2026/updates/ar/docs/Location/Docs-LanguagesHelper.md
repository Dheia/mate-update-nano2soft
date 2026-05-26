# توثيق كلاس LanguagesHelper

## مقدمة عن الكلاس

كلاس `LanguagesHelper` هو كلاس مساعد (Helper Class) مكتوب بلغة PHP يوفر مجموعة شاملة من الوظائف للتعامل مع بيانات اللغات العالمية. يتميز الكلاس بالدعم الكامل للغة العربية والإنجليزية، ويحتوي على قاعدة بيانات للغات مع رموز ISO 639-1 المعيارية.

## فوائد واستخدامات الكلاس

1. **إدارة قائمة اللغات**: يحتوي على قاعدة بيانات لأكثر من 80 لغة عالمية
2. **التعددية اللغوية**: يدعم عرض أسماء اللغات باللغتين العربية والإنجليزية
3. **وظائف البحث المتقدمة**: يوفر خوارزميات بحث ذكية للعثور على اللغات
4. **التصنيف الجغرافي**: يقسم اللغات إلى مجموعات (أوروبية، آسيوية، أفريقية)
5. **التحقق من المدخلات**: وظائف للتحقق من صحة رموز اللغات وترميزها
6. **التصدير والاستيراد**: دعم تصدير البيانات بصيغ JSON و CSV
7. **التكامل مع أنظمة الترجمة**: متوافق مع نظام RainLab.Translate

## هيكل البيانات

يستخدم الكلاس مصفوفة ثابتة `$languages` لتخزين بيانات اللغات، حيث:
- **المفتاح**: رمز اللغة مكون من حرفين (ISO 639-1)
- **القيمة**: مصفوفة تحتوي على اسم اللغة بالإنجليزية والعربية

## أمثلة الاستخدام

### 1. الحصول على جميع اللغات

```php
// الحصول على جميع اللغات باللغة الحالية للنظام
$allLanguages = LanguagesHelper::getAllLanguages();

// الحصول على جميع اللغات باللغة العربية
$allLanguagesInArabic = LanguagesHelper::getAllLanguages('ar');

// الحصول على جميع اللغات كمصفوفة مفاتيح-قيم (مناسبة للقوائم المنسدلة)
$keyValueLanguages = LanguagesHelper::getAllLanguages('en', true);

/*
المخرجات (عند استخدام 'en'):
Array
(
    [0] => Array
        (
            [code] => ar
            [name] => Arabic
            [name_ar] => العربية
            [name_en] => Arabic
        )
    [1] => Array
        (
            [code] => en
            [name] => English
            [name_ar] => الإنجليزية
            [name_en] => English
        )
    ... وغيرها
)
*/
```

### 2. الحصول على اللغات الرئيسية

```php
// الحصول على اللغات الرئيسية باللغة الحالية
$mainLanguages = LanguagesHelper::getMainLanguages();

// الحصول على اللغات الرئيسية بالعربية
$mainLanguagesAr = LanguagesHelper::getMainLanguages('ar');

/*
المخرجات (عند استخدام 'ar'):
Array
(
    [0] => Array
        (
            [code] => ar
            [name] => العربية
            [name_ar] => العربية
            [name_en] => Arabic
        )
    [1] => Array
        (
            [code] => en
            [name] => الإنجليزية
            [name_ar] => الإنجليزية
            [name_en] => English
        )
    ... لغات رئيسية أخرى
)
*/
```

### 3. البحث عن لغة محددة

```php
// البحث باستخدام findLanguage (الأسلوب المحسن)
$language1 = LanguagesHelper::findLanguage('ar'); // بالرمز
$language2 = LanguagesHelper::findLanguage('english'); // بالاسم الإنجليزي
$language3 = LanguagesHelper::findLanguage('العربية'); // بالاسم العربي
$language4 = LanguagesHelper::findLanguage('eng'); // بحث جزئي

/*
المخرجات (مثال للبحث بـ 'ar'):
Array
(
    [code] => ar
    [name] => العربية  // أو Arabic حسب اللغة الحالية
    [name_ar] => العربية
    [name_en] => Arabic
)
*/
```

### 4. البحث المتقدم عن لغات متعددة

```php
// البحث المتقدم مع تحديد النتائج
$advancedResults = LanguagesHelper::findLanguagesAdvanced('ch', 'en', 5);

/*
المخرجات:
Array
(
    [0] => Array
        (
            [code] => zh
            [name] => Chinese
            [name_ar] => الصينية
            [name_en] => Chinese
            [score] => 100
            [match_type] => exact
        )
    [1] => Array
        (
            [code] => ch
            [name] => Chinese (الرمز القديم)
            [name_ar] => الصينية
            [name_en] => Chinese
            [score] => 70
            [match_type] => starts_with
        )
    ... وغيرها
)
*/
```

### 5. الحصول على اسم اللغة

```php
// الحصول على اسم اللغة حسب اللغة الحالية
$name1 = LanguagesHelper::getName('ar', 'en'); // Returns: Arabic
$name2 = LanguagesHelper::getName('ar', 'ar'); // Returns: العربية

// الحصول على الاسم بالعربية فقط
$arabicName = LanguagesHelper::getNameAr('fr'); // Returns: الفرنسية

// الحصول على الاسم بالإنجليزية فقط
$englishName = LanguagesHelper::getNameEn('fr'); // Returns: French
```

### 6. الحصول على رمز اللغة من الاسم

```php
// الحصول على الرمز من الاسم الإنجليزي
$code1 = LanguagesHelper::getCodeByName('Arabic'); // Returns: ar

// الحصول على الرمز من الاسم العربي
$code2 = LanguagesHelper::getCodeByArabicName('الفرنسية'); // Returns: fr

// البحث باستخدام getCode (دالة مختصرة)
$code3 = LanguagesHelper::getCode('español'); // Returns: es
$code4 = LanguagesHelper::getCode('invalid', 'en'); // Returns: en (القيمة الافتراضية)
```

### 7. البحث الذكي في اللغات

```php
// البحث الأساسي
$searchResults = LanguagesHelper::search('ger', 'en');

// البحث الذكي مع خيارات متقدمة
$smartResults = LanguagesHelper::smartSearch('ger', 'en', [
    'limit' => 5,
    'prioritize_code' => true,
    'include_match_info' => true
]);

/*
المخرجات (smartSearch):
Array
(
    [0] => Array
        (
            [code] => de
            [name] => German
            [name_ar] => الألمانية
            [name_en] => German
            [match_level] => contains
        )
    ... وغيرها
)
*/
```

### 8. الحصول على لغات حسب المنطقة الجغرافية

```php
// اللغات الأوروبية
$european = LanguagesHelper::getEuropeanLanguages('en');

// اللغات الآسيوية
$asian = LanguagesHelper::getAsianLanguages('ar');

// اللغات الأفريقية
$african = LanguagesHelper::getAfricanLanguages('en');

// الحصول حسب النوع
$byType = LanguagesHelper::getLanguagesByType('european', 'en');
```

### 9. التحقق من أنواع اللغات

```php
// التحقق مما إذا كانت اللغة أوروبية
$isEuropean = LanguagesHelper::isEuropean('fr'); // Returns: true
$isEuropean2 = LanguagesHelper::isEuropean('ar'); // Returns: false

// التحقق مما إذا كانت اللغة آسيوية
$isAsian = LanguagesHelper::isAsian('ja'); // Returns: true

// التحقق مما إذا كانت اللغة أفريقية
$isAfrican = LanguagesHelper::isAfrican('sw'); // Returns: true

// التحقق مما إذا كانت اللغة رئيسية
$isMain = LanguagesHelper::isMainLanguage('zh'); // Returns: true
```

### 10. الحصول على خيارات القوائم المنسدلة

```php
// جميع اللغات للقوائم المنسدلة
$dropdownAll = LanguagesHelper::getDropdownOptions('ar');

// اللغات الرئيسية فقط
$dropdownMain = LanguagesHelper::getMainDropdownOptions('en');

// اللغات الأوروبية
$dropdownEuropean = LanguagesHelper::getEuropeanDropdownOptions('ar');

/*
المخرجات (getDropdownOptions بالعربية):
Array
(
    [ar] => العربية
    [en] => الإنجليزية
    [fr] => الفرنسية
    ... وغيرها
)
*/
```

### 11. التصدير والاستيراد

```php
// تصدير إلى JSON
$jsonData = LanguagesHelper::getJsonData('en');

// تصدير إلى CSV
$csvData = LanguagesHelper::exportToCsv('ar', true);

/*
المخرجات (getJsonData):
{
    {
        "code": "ar",
        "name": "Arabic",
        "name_ar": "العربية",
        "name_en": "Arabic"
    },
    {
        "code": "en",
        "name": "English",
        "name_ar": "الإنجليزية",
        "name_en": "English"
    },
    ... وغيرها
}
*/
```

### 12. التحقق من صحة رموز اللغات

```php
// التحقق البسيط
$validCode = LanguagesHelper::getCheckCode('english', 'en');

// التحقق المتقدم
$validatedCode = LanguagesHelper::getValidatedCode('español', [
    'strict' => false,
    'fallback' => 'en',
    'return_info' => true
]);

/*
المخرجات (getValidatedCode مع return_info = true):
Array
(
    [code] => es
    [name_en] => Spanish
    [name_ar] => الإسبانية
    [match_type] => partial_english
    [confidence] => 60
    [alternatives] => Array(...)
)
*/

// التحقق مع إرجاع أسباب الخطأ
$validation = LanguagesHelper::validateCode('invalid', true);

/*
المخرجات عند وجود خطأ:
Array
(
    [valid] => false
    [reasons] => Array
        (
            [input_empty] => false
            [not_found] => true
            [suggestions] => Array(...)
        )
)
*/
```

### 13. الحصول على معلومات إضافية

```php
// الحصول على معلومات دولة/دول اللغة الرسمية
$countries = LanguagesHelper::getCountryForLanguage('ar');
// Returns: ['SA', 'EG', 'AE', 'MA', 'DZ', ...]

// الحصول على معلومات إضافية عن اللغة
$languageInfo = LanguagesHelper::getLanguageInfo('zh');
/*
Returns:
Array
(
    [native_name] => 中文
    [family] => Sino-Tibetan
    [speakers] => 1.3 billion
)
*/

// الحصول على اقتراحات للبحث السريع
$suggestions = LanguagesHelper::getQuickSearchSuggestions('chi', 'en', 3);
```

### 14. دوال مساعدة متنوعة

```php
// الحصول على اللغة الافتراضية للنظام
$defaultLocale = LanguagesHelper::getDefaultLocale();

// الحصول على عدد اللغات المتاحة
$count = LanguagesHelper::getCount(); // Returns: عدد اللغات

// الحصول على لغات مرتبة أبجدياً
$sorted = LanguagesHelper::getSortedLanguages('ar', 'name');

// الحصول على مجموعة لغات حسب رموز محددة
$selectedLanguages = LanguagesHelper::getLanguagesByCodes(['ar', 'en', 'fr'], 'en');

// الحصول على لغة واحدة حسب الرمز
$singleLanguage = LanguagesHelper::getLanguagesByCode('ja', 'ar');
```

## ملاحظات مهمة

1. **اللغة الافتراضية**: إذا لم يتم تمرير معلمة `$locale`، يستخدم الكلاس اللغة الحالية للنظام (`\Lang::getLocale()`)

2. **حساسية الأحرف**: معظم دوال البحث غير حساسة لحالة الأحرف (case-insensitive)

3. **أولويات البحث**: تستخدم دوال البحث خوارزميات ذكية تعطي أولويات مختلفة لأنواع المطابقة

4. **الدعم العربي**: جميع الدوال تدخل اللغة العربية بشكل كامل، سواء في المدخلات أو المخرجات

5. **القيم الافتراضية**: العديد من الدوال تدعم تحديد قيم افتراضية عند عدم العثور على النتائج

6. **الأداء**: تم تحسين الكود للأداء الجيد حتى مع عدد كبير من اللغات

## أمثلة تكاملية

### مثال 1: إنشاء قائمة منسدلة للغات في نموذج

```php
// في الكونترولر
$languages = LanguagesHelper::getMainDropdownOptions(app()->getLocale());

// في البلاد فيو (Blade)
<select name="language">
    @foreach($languages as $code => $name)
        <option value="{{ $code }}">{{ $name }}</option>
    @endforeach
</select>
```

### مثال 2: البحث والفلترة في واجهة المستخدم

```php
// البحث عن لغة بناءً على إدخال المستخدم
$userInput = request()->input('language_search');
$results = LanguagesHelper::findLanguagesAdvanced($userInput, app()->getLocale(), 10);

// عرض النتائج
foreach ($results as $language) {
    echo "{$language['name']} ({$language['code']}) - Score: {$language['score']}<br>";
}
```

### مثال 3: التحقق من صحة إدخال اللغة

```php
// في تحقق النموذج (Form Validation)
$validatedData = request()->validate([
    'language' => ['required', function ($attribute, $value, $fail) {
        if (!LanguagesHelper::validateCode($value)) {
            $fail('اللغة المحددة غير صالحة.');
        }
    }],
]);

// أو استخدام دالة getCode مع قيمة افتراضية
$languageCode = LanguagesHelper::getCode(request('language'), 'en');
```

### مثال 4: معالجة قائمة لغات المستخدم

```php
// مستخدم يتحدث عدة لغات
$userLanguages = ['ar', 'en', 'fr'];

// الحصول على معلومات هذه اللغات
$languagesInfo = LanguagesHelper::getLanguagesByCodes($userLanguages, 'ar');

// عرضها للمستخدم
foreach ($languagesInfo as $lang) {
    echo "{$lang['name']} ({$lang['code']})<br>";
}
```

## استثناءات محتملة

1. **رموز غير موجودة**: عند استخدام رموز غير موجودة، ترجع معظم الدوال `null` أو القيمة الافتراضية المحددة

2. **مدخلات فارغة**: الدوال التي تتعامل مع البحث ترجع مصفوفة فارغة أو `null` عند إدخال بحث فارغ

3. **مشاكل الترميز**: تم كتابة الكود بدعم كامل لـ UTF-8 بما في ذلك الأحرف العربية

## الخلاصة

كلاس `LanguagesHelper` يوفر حلاً شاملاً لإدارة اللغات في التطبيقات متعددة اللغات. يتميز بواجهة برمجية سهلة الاستخدام، ودعم قوي للغة العربية، وخوارزميات بحث ذكية تجعله مناسبًا للتطبيقات التي تحتاج إلى التعامل مع لغات متعددة بكفاءة.