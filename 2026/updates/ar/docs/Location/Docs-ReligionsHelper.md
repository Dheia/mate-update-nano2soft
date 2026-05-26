# توثيق كلاس ReligionsHelper

## مقدمة عن الكلاس

كلاس `ReligionsHelper` هو كلاس مساعد (Helper Class) مكتوب بلغة PHP يوفر مجموعة شاملة من الوظائف للتعامل مع بيانات الديانات العالمية. يتميز الكلاس بالدعم الكامل للغة العربية والإنجليزية، ويحتوي على قاعدة بيانات لأكثر من 40 ديانة عالمية مع ترجماتها العربية.

## فوائد واستخدامات الكلاس

1. **إدارة قائمة الديانات**: قاعدة بيانات للديانات العالمية مع التصنيف المناسب
2. **التعددية اللغوية**: يدعم عرض أسماء الديانات باللغتين العربية والإنجليزية
3. **التصنيف الديني**: يقسم الديانات إلى فئات (إبراهيمية، شرقية، رئيسية)
4. **وظائف البحث**: خوارزميات بحث ذكية للعثور على الديانات
5. **التحقق من المدخلات**: وظائف للتحقق من صحة مفاتيح الديانات
6. **التصدير والاستيراد**: دعم تصدير البيانات بصيغ JSON و CSV
7. **التكامل مع أنظمة الترجمة**: متوافق مع نظام RainLab.Translate

## هيكل البيانات

يستخدم الكلاس مصفوفة ثابتة `$religions` لتخزين بيانات الديانات، حيث:
- **المفتاح**: اسم الديانة بالإنجليزية (يستخدم كمعرف فريد)
- **القيمة**: اسم الديانة بالعربية

## أمثلة الاستخدام

### 1. الحصول على جميع الديانات

```php
// الحصول على جميع الديانات باللغة الحالية للنظام
$allReligions = ReligionsHelper::getAllReligions();

// الحصول على جميع الديانات باللغة العربية
$allReligionsInArabic = ReligionsHelper::getAllReligions('ar');

// الحصول على جميع الديانات كمصفوفة مفاتيح-قيم (مناسبة للقوائم المنسدلة)
$keyValueReligions = ReligionsHelper::getAllReligions('en', true);

/*
المخرجات (عند استخدام 'en'):
Array
(
    [0] => Array
        (
            [key] => Islam
            [name] => Islam
            [name_ar] => الإسلام
            [name_en] => Islam
        )
    [1] => Array
        (
            [key] => Christianity
            [name] => Christianity
            [name_ar] => المسيحية
            [name_en] => Christianity
        )
    ... وغيرها
)
*/
```

### 2. الحصول على الديانات الرئيسية

```php
// الحصول على الديانات الرئيسية باللغة الحالية
$mainReligions = ReligionsHelper::getMainReligions();

// الحصول على الديانات الرئيسية بالعربية
$mainReligionsAr = ReligionsHelper::getMainReligions('ar');

/*
المخرجات (عند استخدام 'ar'):
Array
(
    [0] => Array
        (
            [key] => Islam
            [name] => الإسلام
            [name_ar] => الإسلام
            [name_en] => Islam
        )
    [1] => Array
        (
            [key] => Christianity
            [name] => المسيحية
            [name_ar] => المسيحية
            [name_en] => Christianity
        )
    ... ديانات رئيسية أخرى
)
*/
```

### 3. الحصول على الديانات الإبراهيمية (السماوية)

```php
// الحصول على الديانات الإبراهيمية باللغة الحالية
$abrahamicReligions = ReligionsHelper::getAbrahamicReligions();

// الحصول على الديانات الإبراهيمية بالإنجليزية
$abrahamicReligionsEn = ReligionsHelper::getAbrahamicReligions('en');

/*
المخرجات:
Array
(
    [0] => Array
        (
            [key] => Islam
            [name] => الإسلام (أو Islam حسب اللغة)
            [name_ar] => الإسلام
            [name_en] => Islam
        )
    [1] => Array
        (
            [key] => Christianity
            [name] => المسيحية (أو Christianity حسب اللغة)
            [name_ar] => المسيحية
            [name_en] => Christianity
        )
    ... وغيرها
)
*/
```

### 4. الحصول على الديانات الشرقية

```php
// الحصول على الديانات الشرقية باللغة الحالية
$easternReligions = ReligionsHelper::getEasternReligions();

// الحصول على الديانات الشرقية بالعربية
$easternReligionsAr = ReligionsHelper::getEasternReligions('ar');
```

### 5. البحث عن ديانة محددة

```php
// البحث باستخدام findReligion
$religion1 = ReligionsHelper::findReligion('Islam'); // بالمفتاح الإنجليزي
$religion2 = ReligionsHelper::findReligion('الإسلام'); // بالاسم العربي
$religion3 = ReligionsHelper::findReligion('islam'); // بحث غير حساس لحالة الأحرف

/*
المخرجات (مثال للبحث بـ 'Islam'):
Array
(
    [key] => Islam
    [name] => الإسلام  // أو Islam حسب اللغة الحالية
    [name_ar] => الإسلام
    [name_en] => Islam
)
*/
```

### 6. الحصول على اسم الديانة

```php
// الحصول على اسم الديانة حسب اللغة الحالية
$name1 = ReligionsHelper::getName('Islam', 'en'); // Returns: Islam
$name2 = ReligionsHelper::getName('Islam', 'ar'); // Returns: الإسلام

// الحصول على الاسم بالعربية فقط
$arabicName = ReligionsHelper::getNameAr('Christianity'); // Returns: المسيحية

// الحصول على الاسم بالإنجليزية فقط
$englishName = ReligionsHelper::getNameEn('Christianity'); // Returns: Christianity
```

### 7. التحقق من وجود ديانة

```php
// التحقق من وجود ديانة بالمفتاح
$exists1 = ReligionsHelper::exists('Islam'); // Returns: true
$exists2 = ReligionsHelper::exists('NonExistent'); // Returns: false
```

### 8. الحصول على المفتاح من الاسم العربي

```php
// الحصول على المفتاح (الإنجليزية) من الاسم العربي
$key = ReligionsHelper::getKeyByArabicName('الإسلام'); // Returns: Islam
$key2 = ReligionsHelper::getKeyByArabicName('المسيحية'); // Returns: Christianity
```

### 9. البحث في الديانات

```php
// البحث عن ديانات تحتوي على كلمة معينة
$searchResults = ReligionsHelper::search('christ', 'en');
$searchResultsAr = ReligionsHelper::search('مسي', 'ar');

/*
المخرجات (search باللغة الإنجليزية):
Array
(
    [0] => Array
        (
            [key] => Christianity
            [name] => Christianity
            [name_ar] => المسيحية
            [name_en] => Christianity
        )
    [1] => Array
        (
            [key] => Other Christianity
            [name] => Other Christianity
            [name_ar] => مسيحية أخرى
            [name_en] => Other Christianity
        )
)
*/
```

### 10. الحصول على ديانات حسب النوع

```php
// الحصول على ديانات حسب النوع
$byType1 = ReligionsHelper::getReligionsByType('abrahamic', 'en');
$byType2 = ReligionsHelper::getReligionsByType('eastern', 'ar');
$byType3 = ReligionsHelper::getReligionsByType('main', 'en');
$byType4 = ReligionsHelper::getReligionsByType('all', 'ar');

/*
المخرجات (getReligionsByType('abrahamic', 'en')):
Array
(
    [0] => Array
        (
            [key] => Islam
            [name] => Islam
            [name_ar] => الإسلام
            [name_en] => Islam
        )
    [1] => Array
        (
            [key] => Christianity
            [name] => Christianity
            [name_ar] => المسيحية
            [name_en] => Christianity
        )
    ... ديانات إبراهيمية أخرى
)
*/
```

### 11. الحصول على خيارات القوائم المنسدلة

```php
// جميع الديانات للقوائم المنسدلة
$dropdownAll = ReligionsHelper::getDropdownOptions('ar');

// الديانات الرئيسية فقط
$dropdownMain = ReligionsHelper::getMainDropdownOptions('en');

// الديانات الإبراهيمية
$dropdownAbrahamic = ReligionsHelper::getAbrahamicDropdownOptions('ar');

/*
المخرجات (getDropdownOptions بالعربية):
Array
(
    [Islam] => الإسلام
    [Christianity] => المسيحية
    [Judaism] => اليهودية
    [Hinduism] => الهندوسية
    ... وغيرها
)
*/
```

### 12. التصدير والاستيراد

```php
// تصدير إلى JSON
$jsonData = ReligionsHelper::getJsonData('en');

// تصدير إلى CSV
$csvData = ReligionsHelper::exportToCsv('ar', true);

/*
المخرجات (getJsonData):
[
    {
        "key": "Islam",
        "name": "Islam",
        "name_ar": "الإسلام",
        "name_en": "Islam"
    },
    {
        "key": "Christianity",
        "name": "Christianity",
        "name_ar": "المسيحية",
        "name_en": "Christianity"
    },
    ... وغيرها
]
*/
```

### 13. دوال مساعدة متنوعة

```php
// الحصول على اللغة الافتراضية للنظام
$defaultLocale = ReligionsHelper::getDefaultLocale();

// الحصول على عدد الديانات المتاحة
$count = ReligionsHelper::getCount(); // Returns: عدد الديانات

// الحصول على ديانات مرتبة أبجدياً
$sorted = ReligionsHelper::getSortedReligions('ar', 'name');

// الحصول على مجموعة ديانات حسب مفاتيح محددة
$selectedReligions = ReligionsHelper::getReligionsByKeys(['Islam', 'Christianity', 'Judaism'], 'en');

/*
المخرجات (getReligionsByKeys):
Array
(
    [0] => Array
        (
            [key] => Islam
            [name] => Islam
            [name_ar] => الإسلام
            [name_en] => Islam
        )
    [1] => Array
        (
            [key] => Christianity
            [name] => Christianity
            [name_ar] => المسيحية
            [name_en] => Christianity
        )
    [2] => Array
        (
            [key] => Judaism
            [name] => Judaism
            [name_ar] => اليهودية
            [name_en] => Judaism
        )
)
*/
```

### 14. التحقق من أنواع الديانات

```php
// التحقق مما إذا كانت الديانة إبراهيمية
$isAbrahamic = ReligionsHelper::isAbrahamic('Islam'); // Returns: true
$isAbrahamic2 = ReligionsHelper::isAbrahamic('Hinduism'); // Returns: false

// التحقق مما إذا كانت الديانة شرقية
$isEastern = ReligionsHelper::isEastern('Hinduism'); // Returns: true
$isEastern2 = ReligionsHelper::isEastern('Islam'); // Returns: false

// التحقق مما إذا كانت الديانة رئيسية
$isMain = ReligionsHelper::isMainReligion('Buddhism'); // Returns: true
$isMain2 = ReligionsHelper::isMainReligion('Taoism'); // Returns: false
```

## ملاحظات مهمة

1. **اللغة الافتراضية**: إذا لم يتم تمرير معلمة `$locale`، يستخدم الكلاس اللغة الحالية للنظام (`\Lang::getLocale()`)

2. **حساسية الأحرف**: دوال البحث غير حساسة لحالة الأحرف (case-insensitive) بالنسبة للأسماء الإنجليزية

3. **المفتاح الأساسي**: المفتاح المستخدم في المصفوفة `$religions` هو الاسم الإنجليزي للديانة، وهو المعرّف الفريد

4. **الدعم العربي**: جميع الدوال تدعم اللغة العربية بشكل كامل، سواء في المدخلات أو المخرجات

5. **القيم الافتراضية**: العديد من الدوال ترجع `null` أو مصفوفة فارغة عند عدم العثور على النتائج

6. **التصنيف الديني**: يقسم الديانات إلى:
   - **إبراهيمية**: الإسلام، المسيحية، اليهودية، البهائية، الدروز
   - **شرقية**: الهندوسية، البوذية، السيخية، الجاينية، الطاوية، الكونفوشيوسية، الشنتوية
   - **رئيسية**: الإسلام، المسيحية، اليهودية، الهندوسية، البوذية، السيخية، الإلحاد، لا دين، أديان أخرى

## أمثلة تكاملية

### مثال 1: إنشاء قائمة منسدلة للديانات في نموذج

```php
// في الكونترولر
$religions = ReligionsHelper::getMainDropdownOptions(app()->getLocale());

// في البلاد فيو (Blade)
<select name="religion">
    <option value="">اختر الديانة</option>
    @foreach($religions as $key => $name)
        <option value="{{ $key }}" {{ old('religion') == $key ? 'selected' : '' }}>
            {{ $name }}
        </option>
    @endforeach
</select>
```

### مثال 2: البحث والفلترة في واجهة المستخدم

```php
// البحث عن ديانة بناءً على إدخال المستخدم
$userInput = request()->input('religion_search');
$results = ReligionsHelper::search($userInput, app()->getLocale());

// عرض النتائج
if (!empty($results)) {
    foreach ($results as $religion) {
        echo "<div class='religion-item'>";
        echo "<strong>{$religion['name']}</strong> ({$religion['key']})<br>";
        echo "بالعربية: {$religion['name_ar']}<br>";
        echo "</div>";
    }
} else {
    echo "لم يتم العثور على نتائج.";
}
```

### مثال 3: التحقق من صحة إدخال الديانة

```php
// في تحقق النموذج (Form Validation)
$validatedData = request()->validate([
    'religion' => ['required', function ($attribute, $value, $fail) {
        if (!ReligionsHelper::exists($value)) {
            $fail('الديانة المحددة غير صالحة.');
        }
    }],
]);

// أو بشكل أبسط
$religion = request('religion');
if (ReligionsHelper::exists($religion)) {
    // الديانة صحيحة، يمكن المتابعة
    $religionName = ReligionsHelper::getName($religion, app()->getLocale());
} else {
    // الديانة غير صحيحة
    session()->flash('error', 'الديانة المحددة غير موجودة.');
}
```

### مثال 4: معالجة ديانات متعددة للمستخدم

```php
// مستخدم يتبع عدة ديانات (في حالة الديانات المتعددة)
$userReligions = ['Islam', 'Christianity'];

// الحصول على معلومات هذه الديانات
$religionsInfo = ReligionsHelper::getReligionsByKeys($userReligions, 'ar');

// عرضها للمستخدم
echo "الديانات المحددة:<br>";
foreach ($religionsInfo as $religion) {
    echo "- {$religion['name']} ({$religion['key']})<br>";
    
    // التحقق من النوع
    if (ReligionsHelper::isAbrahamic($religion['key'])) {
        echo "  (ديانة إبراهيمية)<br>";
    } elseif (ReligionsHelper::isEastern($religion['key'])) {
        echo "  (ديانة شرقية)<br>";
    }
}
```

### مثال 5: استخدام في لوحة تحكم الإدارة

```php
// في لوحة تحكم الإدارة لعرض إحصائيات الديانات
$religionsCount = ReligionsHelper::getCount();
$mainReligions = ReligionsHelper::getMainReligions('ar');

echo "<div class='stats-card'>";
echo "<h3>إحصائيات الديانات</h3>";
echo "<p>عدد الديانات المتاحة: {$religionsCount}</p>";
echo "<h4>الديانات الرئيسية:</h4>";
echo "<ul>";

foreach ($mainReligions as $religion) {
    echo "<li>{$religion['name']} ({$religion['key']})</li>";
}

echo "</ul>";
echo "</div>";
```

### مثال 6: تصدير بيانات الديانات

```php
// تصدير بيانات الديانات لاستخدامها في تطبيقات أخرى
$jsonData = ReligionsHelper::getJsonData('en');

// حفظ في ملف
file_put_contents('religions_data.json', $jsonData);

// أو إرجاع كاستجابة JSON في API
return response()->json([
    'success' => true,
    'data' => json_decode($jsonData, true),
    'count' => ReligionsHelper::getCount(),
    'exported_at' => now()->toDateTimeString()
]);
```

## استثناءات محتملة

1. **مفاتيح غير موجودة**: عند استخدام مفاتيح غير موجودة، ترجع معظم الدوال `null` أو مصفوفة فارغة

2. **مدخلات فارغة**: الدوال التي تتعامل مع البحث ترجع مصفوفة فارغة عند إدخال بحث فارغ

3. **مشاكل الترميز**: تم كتابة الكود بدعم كامل لـ UTF-8 بما في ذلك الأحرف العربية

4. **أسماء مكررة**: كل ديانة لها مفتاح فريد بالإنجليزية، مما يمنع التكرار

## الخلاصة

كلاس `ReligionsHelper` يوفر حلاً شاملاً لإدارة الديانات في التطبيقات التي تحتاج إلى التعامل مع تنوع ديني. يتميز بواجهة برمجية سهلة الاستخدام، ودعم قوي للغة العربية، وتصنيف منطقي للديانات يسهل عملية البحث والتصفية. الكلاس مناسب للتطبيقات التعليمية، والمنصات الاجتماعية، وأنظمة إدارة المحتوى التي تحتاج إلى التعامل مع بيانات دينية متنوعة.