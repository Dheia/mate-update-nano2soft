# توثيق كلاس NationalityHelper

## 📋 مقدمة عامة

**NationalityHelper** هو كلاس PHP شامل لإدارة الجنسيات العالمية في نظام مبنى على Laravel/Nano2Soft. يوفر قاعدة بيانات كاملة للدول مع جميع المعلومات اللازمة باللغتين العربية والإنجليزية، ويحتوي على وظائف متقدمة للبحث والتصفية والتحويل بين مختلف أنظمة الترميز الدولية.

## 🎯 فوائد واستخدامات الكلاس

### ✅ الفوائد الرئيسية:
- **مصدر موحد للبيانات**: يحتوي على معلومات 240+ دولة وجنسية
- **دعم ثنائي اللغة**: العربية والإنجليزية
- **بيانات شاملة**: أكواد ISO، أكواد الهاتف، العملات، الأعلام، المناطق، اللغات، الأديان
- **أداء عالي**: يستخدم خرائط (Maps) للوصول السريع O(1)
- **سهولة الاستخدام**: واجهة برمجية واضحة وبديهية
- **التوسعة السهلة**: يمكن إضافة دول جديدة بسهولة

### 🛠 حالات الاستخدام:
1. **تعبئة قوائم منسدلة** للجنسيات في نماذج التسجيل
2. **التحقق من صحة** أكواد الدول وأرقام الهواتف
3. **عرض معلومات الدول** مع الأعلام والعملات
4. **التصفية حسب المنطقة** أو اللغة أو الدين
5. **التكامل مع أنظمة CRM** لإدارة بيانات العملاء
6. **توليد أكواد الهاتف** التلقائية في نماذج الاتصال

## 📁 الهيكل العام

```php
<?php namespace Nano\Location\Classes;

class NationalityHelper
{
    // مصفوفة ثابتة تحتوي على جميع الجنسيات
    public static $nationalities = [...];
    
    // خرائط للوصول السريع
    private static $alpha2Map = [];
    private static $alpha3Map = [];
    private static $numericMap = [];
    private static $phoneCodeMap = [];
    
    // دوال رئيسية للجلب والبحث
    public static function getAllNationalities(...) {...}
    public static function findNationality(...) {...}
    public static function getByAlpha2(...) {...}
    // ... وغيرها من الدوال
}
```

## 📊 أمثلة تفصيلية للدوال

### 1️⃣ **الدوال الأساسية للجلب**

#### `getAllNationalities()` - جلب جميع الجنسيات
```php
// المثال 1: جلب جميع الجنسيات باللغة العربية
$allNationalities = NationalityHelper::getAllNationalities('ar');
/*
الإخراج: مصفوفة تحتوي على:
[
    [
        'id' => '887',
        'name' => 'يمني',
        'country_name' => 'اليمن',
        'alpha2' => 'YE',
        'alpha3' => 'YEM',
        'phone_code' => '967',
        'flag_emoji' => '🇾🇪',
        // ... وغيرها من الحقول
    ],
    // ... جميع الدول الأخرى
]
*/

// المثال 2: جبل خيارات للقوائم المنسدلة
$options = NationalityHelper::getAllNationalities('ar', true);
/*
الإخراج: مصفوفة علاقة (key-value):
[
    'YE' => 'يمني',
    'SA' => 'سعودي',
    'AE' => 'إماراتي',
    // ...
]
*/

// المثال 3: جبل باللغة الإنجليزية
$englishNationalities = NationalityHelper::getAllNationalities('en');
```

#### `getAllNationalitiesOptions()` - خيارات ذكية للقوائم
```php
// المثال: جبل الخيارات مع التخزين المؤقت
$options = NationalityHelper::getAllNationalitiesOptions('ar');
/*
الإخراج: نفس getAllNationalities مع المعلمة الثانية true
مفيد عند استخدام الدالة مراراً لتجنب إعادة المعالجة
*/
```

### 2️⃣ **الدوال الخاصة بالمناطق**

#### `getArabNationalities()` - الجنسيات العربية فقط
```php
$arabNationalities = NationalityHelper::getArabNationalities('ar');
/*
الإخراج: مصفوفة تحتوي فقط على الدول العربية (22 دولة)
[
    ['name' => 'سعودي', 'alpha2' => 'SA', ...],
    ['name' => 'مصري', 'alpha2' => 'EG', ...],
    ['name' => 'إماراتي', 'alpha2' => 'AE', ...],
    // ... جميع الدول العربية
]
*/

// باللغة الإنجليزية
$arabNationalitiesEn = NationalityHelper::getArabNationalities('en');
```

#### `getCommonNationalities()` - الجنسيات الشائعة
```php
$common = NationalityHelper::getCommonNationalities('ar');
/*
الإخراج: الدول العربية + دول مهمة عالمياً
يحتوي على: السعودية، مصر، الإمارات، أمريكا، بريطانيا، كندا، إلخ.
*/
```

### 3️⃣ **دوال البحث الفردي**

#### `findNationality()` - البحث بأي نوع من الأكواد
```php
// البحث بكود Alpha2
$result1 = NationalityHelper::findNationality('SA', 'ar');
/*
الإخراج:
[
    'id' => '682',
    'name' => 'سعودي',
    'country_name' => 'السعودية',
    'alpha2' => 'SA',
    'alpha3' => 'SAU',
    'phone_code' => '966',
    // ... إلخ
]
*/

// البحث بكود Alpha3
$result2 = NationalityHelper::findNationality('SAU', 'ar');
// نفس النتيجة السابقة

// البحث بالرقم الرمزي
$result3 = NationalityHelper::findNationality('682', 'ar');

// البحث بكود الهاتف
$result4 = NationalityHelper::findNationality('966', 'ar');

// البحث بالاسم العربي
$result5 = NationalityHelper::findNationality('سعودي', 'ar');

// البحث بالاسم الإنجليزي
$result6 = NationalityHelper::findNationality('Saudi', 'en');
```

#### `getByAlpha2()`, `getByAlpha3()`, `getByNumeric()`, `getByPhoneCode()`
```php
// طرق متخصصة للبحث
$byAlpha2 = NationalityHelper::getByAlpha2('AE', 'ar');
$byAlpha3 = NationalityHelper::getByAlpha3('ARE', 'ar');
$byNumeric = NationalityHelper::getByNumeric('784', 'ar');
$byPhone = NationalityHelper::getByPhoneCode('971', 'ar');

// للمقارنة: getByPhoneCodeV1 (النسخة القديمة)
$byPhoneV1 = NationalityHelper::getByPhoneCodeV1('971', 'ar');
```

#### `searchByName()` - البحث النصي في الأسماء
```php
$results = NationalityHelper::searchByName('سعو', 'ar');
/*
الإخراج: جميع الدول التي تحتوي اسمها على "سعو"
[
    ['name' => 'سعودي', ...],
    ['name' => 'سوري', ...],
    // إلخ
]
*/

$resultsEn = NationalityHelper::searchByName('saud', 'en');
// البحث باللغة الإنجليزية
```

### 4️⃣ **دوال التحويل بين الأنظمة**

#### التحويل من نظام لآخر
```php
// Alpha2 → Alpha3
$alpha3 = NationalityHelper::convertAlpha2ToAlpha3('SA'); // 'SAU'

// Alpha3 → Alpha2
$alpha2 = NationalityHelper::convertAlpha3ToAlpha2('SAU'); // 'SA'

// Alpha2 → رقم رمزي
$numeric = NationalityHelper::convertAlpha2ToNumeric('SA'); // '682'

// رقم رمزي → Alpha2
$alpha2 = NationalityHelper::convertNumericToAlpha2('682'); // 'SA'

// رقم رمزي → Alpha3
$alpha3 = NationalityHelper::convertNumericToAlpha3('682'); // 'SAU'
```

### 5️⃣ **دوال الحصول على معلومات محددة**

#### `getName()` - اسم الجنسية فقط
```php
$nameAr = NationalityHelper::getName('SA', 'ar'); // 'سعودي'
$nameEn = NationalityHelper::getName('SA', 'en'); // 'Saudi'
```

#### `getCountryName()` - اسم الدولة
```php
$countryAr = NationalityHelper::getCountryName('SA', 'ar'); // 'السعودية'
$countryEn = NationalityHelper::getCountryName('SA', 'en'); // 'Saudi Arabia'
```

#### `getPhoneCode()` - كود الهاتف
```php
$phoneCode = NationalityHelper::getPhoneCode('SA'); // '966'
$formatted = NationalityHelper::getFormattedPhoneCode('SA'); // '+966'
```

#### `getCurrency()` - العملة
```php
$currency = NationalityHelper::getCurrency('SA'); // ['SAR']
$currencyUS = NationalityHelper::getCurrency('US'); // ['USD']
```

#### `getFlagEmoji()` - العلم (إيموجي)
```php
$flag = NationalityHelper::getFlagEmoji('SA'); // '🇸🇦'
$flagUS = NationalityHelper::getFlagEmoji('US'); // '🇺🇸'
```

#### `getFlagUrl()` - رابط صورة العلم
```php
$flagUrl = NationalityHelper::getFlagUrl('SA', 'flat');
// 'https://flagcdn.com/flat/sa.png'

$flagUrlShiny = NationalityHelper::getFlagUrl('SA', 'shiny');
// 'https://flagcdn.com/shiny/sa.png'
```

#### `getRegion()`, `getSubregion()` - الموقع الجغرافي
```php
$region = NationalityHelper::getRegion('SA'); // 'Asia'
$subregion = NationalityHelper::getSubregion('SA'); // 'Western Asia'
```

#### `getLanguages()`, `getReligions()` - اللغات والديانات
```php
$languages = NationalityHelper::getLanguages('SA'); // ['ar']
$religions = NationalityHelper::getReligions('SA'); // ['Islam']
```

### 6️⃣ **دوال التصفية حسب المعايير**

#### `getNationalitiesByRegion()` - حسب القارة
```php
$asianCountries = NationalityHelper::getNationalitiesByRegion('Asia', 'ar');
$europeanCountries = NationalityHelper::getNationalitiesByRegion('Europe', 'ar');
$africanCountries = NationalityHelper::getNationalitiesByRegion('Africa', 'ar');
```

#### `getNationalitiesBySubregion()` - حسب المنطقة الفرعية
```php
$arabCountries = NationalityHelper::getNationalitiesBySubregion('Western Asia', 'ar');
$northAfrica = NationalityHelper::getNationalitiesBySubregion('Northern Africa', 'ar');
```

#### `getNationalitiesByLanguage()` - حسب اللغة
```php
$arabicSpeaking = NationalityHelper::getNationalitiesByLanguage('ar', 'ar');
$englishSpeaking = NationalityHelper::getNationalitiesByLanguage('en', 'en');
$frenchSpeaking = NationalityHelper::getNationalitiesByLanguage('fr', 'fr');
```

#### `getNationalitiesByReligion()` - حسب الدين
```php
$islamicCountries = NationalityHelper::getNationalitiesByReligion('Islam', 'ar');
$christianCountries = NationalityHelper::getNationalitiesByReligion('Christianity', 'en');
```

### 7️⃣ **دوال القوائم المنسدلة**

#### `getDropdownOptions()` - خيارات عامة
```php
// خيارات بسيطة
$simpleOptions = NationalityHelper::getDropdownOptions('ar');
/* الإخراج: ['SA' => 'سعودي', 'EG' => 'مصري', ...] */

// مع كود الهاتف
$optionsWithPhone = NationalityHelper::getDropdownOptions('ar', true);
/* الإخراج: ['SA' => 'سعودي (+966)', 'EG' => 'مصري (+20)', ...] */

// مع اسم الدولة
$optionsWithCountry = NationalityHelper::getDropdownOptions('ar', false, false, true);
/* الإخراج: ['SA' => 'السعودية (سعودي)', ...] */

// مع كل الإضافات
$fullOptions = NationalityHelper::getDropdownOptions('ar', true, true, true);
/* الإخراج: ['SA' => 'السعودية (سعودي) (+966) - Asia', ...] */
```

#### `getArabDropdownOptions()` - خيارات الدول العربية
```php
$arabOptions = NationalityHelper::getArabDropdownOptions('ar', true);
/* الإخراج: ['SA' => 'سعودي (+966)', 'EG' => 'مصري (+20)', ...] */
```

#### `getCountriesDropdownOptions()` - أسماء الدول فقط
```php
$countries = NationalityHelper::getCountriesDropdownOptions('ar');
/* الإخراج: ['SA' => 'السعودية', 'EG' => 'مصر', ...] */
```

#### `getPhoneCodeDropdownOptions()` - أكواد الهواتف
```php
$phoneCodes = NationalityHelper::getPhoneCodeDropdownOptions();
/* الإخراج: ['+966' => '+966 (Saudi)', '+20' => '+20 (Egyptian)', ...] */
```

### 8️⃣ **دوال الحصول على القوائم الرئيسية**

#### `getAllRegions()` - جميع القارات
```php
$regions = NationalityHelper::getAllRegions();
/* الإخراج: ['Africa', 'Americas', 'Asia', 'Europe', 'Oceania'] */
```

#### `getAllSubregions()` - جميع المناطق الفرعية
```php
$subregions = NationalityHelper::getAllSubregions();
/* الإخراج: ['Australia and New Zealand', 'Central Asia', ...] */
```

#### `getAllLanguages()` - جميع اللغات
```php
$languages = NationalityHelper::getAllLanguages();
/* الإخراج: ['ar', 'en', 'fr', 'es', ...] */
```

#### `getAllReligions()` - جميع الأديان
```php
$religions = NationalityHelper::getAllReligions();
/* الإخراج: ['Christianity', 'Islam', 'Buddhism', ...] */
```

### 9️⃣ **دوال التصدير والتحقق**

#### `getJsonData()` - تصدير إلى JSON
```php
$jsonData = NationalityHelper::getJsonData('ar');
/* الإخراج: JSON كامل يحتوي على جميع البيانات */
```

#### `exportToCsv()` - تصدير إلى CSV
```php
$csvData = NationalityHelper::exportToCsv('ar');
/* الإخراج: بيانات CSV يمكن حفظها في ملف */
```

#### `validateNationalityData()` - التحقق من البيانات
```php
$data = [
    'name_ar' => 'سعودي',
    'name_en' => 'Saudi',
    'alpha2' => 'SA',
    // ... إلخ
];

$isValid = NationalityHelper::validateNationalityData($data);
// true إذا كانت البيانات صحيحة
```

### 🔟 **الدوال المساعدة**

#### `getCount()` - عدد الجنسيات
```php
$count = NationalityHelper::getCount(); // 240+
```

#### `getArabCountryCodes()` - أكواد الدول العربية
```php
$arabCodes = NationalityHelper::getArabCountryCodes();
/* الإخراج: ['SA', 'EG', 'AE', ...] */
```

#### `isArabCountry()` - التحقق من عربية الدولة
```php
$isArab = NationalityHelper::isArabCountry('SA'); // true
$isArab2 = NationalityHelper::isArabCountry('US'); // false
```

#### `isIslamicCountry()` - التحقق من الإسلام كدين أغلبية
```php
$isIslamic = NationalityHelper::isIslamicCountry('SA'); // true
$isIslamic2 = NationalityHelper::isIslamicCountry('US'); // false
```

#### `getDefaultNationality()` - الجنسية الافتراضية
```php
$default = NationalityHelper::getDefaultNationality('ar');
/* الإخراج: البيانات الكاملة للسعودية */
```

## 💡 أمثلة عملية متكاملة

### مثال 1: نموذج تسجيل مستخدم
```php
// في الكونترولر
public function onRegister()
{
    $countries = NationalityHelper::getCountriesDropdownOptions('ar', true);
    $this->page['countries'] = $countries;
}

// في التيمبلت
<select name="nationality">
    {% for code, name in countries %}
        <option value="{{ code }}">{{ name }}</option>
    {% endfor %}
</select>
```

### مثال 2: عرض معلومات ملف شخصي
```php
public function onShowProfile($userId)
{
    $user = User::find($userId);
    $nationalityInfo = NationalityHelper::findNationality($user->nationality_code, 'ar');
    
    return [
        'user' => $user,
        'nationality' => [
            'name' => $nationalityInfo['name'],
            'flag' => $nationalityInfo['flag_emoji'],
            'phone_code' => $nationalityInfo['formatted_phone_code']
        ]
    ];
}
```

### مثال 3: فلترة المستخدمين حسب المنطقة
```php
public function filterByRegion($region)
{
    $countries = NationalityHelper::getNationalitiesByRegion($region, 'en');
    $countryCodes = array_column($countries, 'alpha2');
    
    return User::whereIn('nationality_code', $countryCodes)->get();
}
```

### مثال 4: توليد قائمة اتصال دولية
```php
public function generateContactList($users)
{
    $contacts = [];
    
    foreach ($users as $user) {
        $nationality = NationalityHelper::getByAlpha2($user->country_code, 'en');
        $contacts[] = [
            'name' => $user->name,
            'phone' => $nationality['formatted_phone_code'] . $user->phone,
            'country' => $nationality['country_name'],
            'flag' => $nationality['flag_emoji']
        ];
    }
    
    return $contacts;
}
```

## ⚠️ ملاحظات هامة

### التخزين المؤقت
```php
// الدوال تستخدم خرائط داخلية للتخزين المؤقت
// أول استدعاء يهيئ الخرائط، الاستدعاءات اللاحقة سريعة
NationalityHelper::initMaps(); // يتم استدعاؤه تلقائياً
```

### معالجة التكرارات
```php
// الدالة cleanDuplicateData() تزيل التكرارات تلقائياً
// يمكن استدعاؤها يدوياً إذا لزم الأمر
NationalityHelper::cleanDuplicateData();
```

### الأخطاء الشائعة
```php
// تأكد من صحة الكود قبل الاستخدام
$result = NationalityHelper::getByAlpha2('XX'); // null (غير موجود)

// تحقق من وجود النتيجة قبل استخدامها
if ($nationality = NationalityHelper::getByAlpha2('SA')) {
    // استخدم $nationality
} else {
    // تعامل مع الخطأ
}
```

## 🔄 الترقية والصيانة

### إضافة دولة جديدة
```php
// أضف عنصراً جديداً لمصفوفة $nationalities
$newCountry = [
    'name_ar' => 'جديد',
    'name_en' => 'New',
    'country_name_ar' => 'دولة جديدة',
    'country_name_en' => 'New Country',
    'alpha2' => 'NC',
    'alpha3' => 'NEW',
    // ... أكمل جميع الحقول المطلوبة
];

// أضفها إلى المصفوفة
NationalityHelper::$nationalities[] = $newCountry;

// أعد تهيئة الخرائط
NationalityHelper::initMaps();
```

### تحديث بيانات دولة
```php
// ابحث عن الدولة بالتعديل المباشر في المصفوفة
foreach (NationalityHelper::$nationalities as &$country) {
    if ($country['alpha2'] === 'SA') {
        $country['phone_code'] = '966';
        // ... تحديث الحقول الأخرى
        break;
    }
}
```

## 📈 إحصائيات الكلاس

- **عدد الدول**: 240+ دولة وإقليم
- **عدد الحقول لكل دولة**: 19 حقل
- **اللغات المدعومة**: العربية والإنجليزية
- **أنظمة الترميز المدعومة**: Alpha2, Alpha3, Numeric, Phone Code
- **التصنيفات الجغرافية**: 5 قارات، 25+ منطقة فرعية
- **اللغات المتاحة**: 100+ لغة
- **الأديان المتاحة**: 15+ دين

---

## 🎉 خاتمة

كلاس **NationalityHelper** يمثل حلاً شاملاً لإدارة الجنسيات في التطبيقات العربية والدولية. تصميمه المرن وأداؤه العالي يجعله مناسباً للتطبيقات الصغيرة والكبيرة على حد سواء. الدوال الواضحة والتوثيق الشامل يسهلان عملية التكامل والصيانة.