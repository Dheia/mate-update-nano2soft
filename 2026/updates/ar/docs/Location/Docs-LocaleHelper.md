# توثيق كلاس LocaleHelper

## مقدمة عن الكلاس

كلاس `LocaleHelper` هو كلاس مساعد (Helper Class) متقدم مكتوب بلغة PHP يوفر وظائف شاملة لإدارة الإعدادات المحلية (Locale) في تطبيقات Nano2Soft. يعمل الكلاس على إدارة اللغة، الوقت، الموقع الجغرافي، والترجمة بشكل متكامل مع أنظمة Nano2Soft و RainLab.Translate.

## فوائد واستخدامات الكلاس

1. **إدارة اللغة**: دعم متعدد اللغات مع تحديد اتجاه النص (RTL/LTR)
2. **التعامل مع الزمن**: إدارة المناطق الزمنية والتحويلات
3. **الجغرافيا**: الحصول على إحداثيات الموقع الجغرافي ومعالجتها
4. **الترجمة المتقدمة**: وظائف ترجمة ذكية مع دعم الجمع (Pluralization)
5. **التكامل مع الأنظمة**: تكامل مع RainLab.Translate و Nano2Soft Backend
6. **الاستعلام عن الخصائص**: استخراج المعلومات اللغوية والجغرافية من الطلبات
7. **معالجة الأخطاء**: معالجة شاملة للأخطاء مع دعم التتبع

## الثوابت والمتغيرات الثابتة

```php
// قائمة اللغات ذات الكتابة من اليمين لليسار (RTL)
private const RTL = [
    'ar', 'ae', 'arc', 'bcc', 'bqi', 'ckb', 'dv', 'fa', 'glk', 'ha',
    'he', 'khw', 'ks', 'ku', 'lrc', 'mzn', 'nqo', 'pnb', 'ps', 'sd',
    'ug', 'ur', 'uz_AF', 'yi'
];

// متغيرات تخزين مؤقتة للطلبات
public static $locale_in_request;
public static $locale_in_request_header;
public static $locale_in_request_body;
```

## أمثلة الاستخدام

### 1. تحديد اتجاه النص (RTL/LTR)

```php
// الحصول على اتجاه النص للغة الحالية
$direction = LocaleHelper::currentDir(); // Returns: 'rtl' أو 'ltr'

// الحصول على اتجاه النص للغة محددة
$directionAr = LocaleHelper::currentDir('ar'); // Returns: 'rtl'
$directionEn = LocaleHelper::currentDir('en'); // Returns: 'ltr'

// التحقق إذا كانت اللغة من اليمين لليسار
$isRtlAr = LocaleHelper::isRtl('ar'); // Returns: true
$isRtlEn = LocaleHelper::isRtl('en'); // Returns: false

// دالة بديلة لتحديد الاتجاه
$dir = LocaleHelper::language_direction('fa'); // Returns: 'rtl'

/*
المخرجات (currentDir):
'rtl' للغات: العربية، الفارسية، العبرية، الأردية، إلخ.
'ltr' للغات: الإنجليزية، الفرنسية، الألمانية، إلخ.
*/
```

### 2. الحصول على الإعدادات المحلية

```php
// الحصول على اللغة الحالية للنظام
$currentLocale = LocaleHelper::getLocale();

// الحصول على اللغة الافتراضية للنظام
$defaultLocale = LocaleHelper::getDefaultLocale();

// التحقق من وجود إضافة الترجمة
$hasTranslate = LocaleHelper::hasTranslatePlugin(); // Returns: true/false

// التحقق من وجود لغات متعددة
$hasMultiple = LocaleHelper::hasMulitpleLocales(); // Returns: true/false

// الحصول على خيارات اللغات المتاحة
$localeOptions = LocaleHelper::getLocaleOptions();

/*
المخرجات (getLocaleOptions):
Array
(
    [ar] => العربية
    [en] => English
    [fr] => Français
    ... وغيرها
)
*/
```

### 3. الحصول على اللغة من مصادر مختلفة

```php
// الحصول على اللغة من المستخدم (لوحة التحكم)
$userLocale = LocaleHelper::localeFromUSER();

// الحصول على اللغة من الرابط
$urlLocale = LocaleHelper::localeFromURL();

// الحصول على اللغة من بيانات POST
$postLocale = LocaleHelper::localeFromPost();

// الحصول على اللغة من الجلسة
$sessionLocale = LocaleHelper::localeFromSession();

// الحصول على اللغة من المتصفح
$browserLocale = LocaleHelper::localeFromBrowser();

// الحصول على قائمة اللغات المقبولة من المتصفح
$browserLangs = LocaleHelper::getLocaleFromBrowser();

/*
المخرجات (getLocaleFromBrowser):
Array
(
    [0] => Array
        (
            [code] => en-US
            [q] => 1
        )
    [1] => Array
        (
            [code] => en
            [q] => 0.8
        )
    ... وغيرها
)
*/
```

### 4. إدارة المناطق الزمنية

```php
// الحصول على جميع المناطق الزمنية
$timezones = LocaleHelper::timezones();

/*
المخرجات (timezones):
Array
(
    [Africa/Abidjan] => (+00:00) Africa Abidjan
    [Africa/Accra] => (+00:00) Africa Accra
    [Africa/Addis_Ababa] => (+03:00) Africa Addis Ababa
    [Africa/Algiers] => (+01:00) Africa Algiers
    ... وغيرها
)
*/
```

### 5. التعامل مع الموقع الجغرافي

```php
// الحصول على الموقع الجغرافي من الطلب
$requestGeo = LocaleHelper::getGeolocationInRequest();

// الحصول من خدمة MaxMindGeoApi
$maxmindGeo = LocaleHelper::getGeolocationInMaxMindGeoApi();

// الحصول من خدمة freegeoip.net
$freegeoipGeo = LocaleHelper::getGeolocationInGeoIP();

// الحصول على الموقع الجغرافي للمستخدم
$userGeo = LocaleHelper::getUserGeolocation($user);

// تعيين الموقع الجغرافي للمستخدم
LocaleHelper::setUserGeolocation(['latitude' => 24.7136, 'longitude' => 46.6753], $user);

// تعيين خط العرض فقط
LocaleHelper::setUserLatitude(24.7136, $user);

// تعيين خط الطول فقط
LocaleHelper::setUserLongitude(46.6753, $user);

// الحصول على الموقع الحالي (يجمع مصادر متعددة)
$currentGeo = LocaleHelper::getCurrentGeolocation($user, ['latitude' => 0, 'longitude' => 0]);

// الحصول على الإحداثيات كسلسلة نصية
$coordinates = LocaleHelper::getCurrentCoordinates($user); // Returns: "24.7136,46.6753"

// الحصول على الإحداثيات ككائن
$coordsObject = LocaleHelper::getCurrentCoordinates($user, null, true);

// التحقق من وجود إحداثيات
$hasGeo = LocaleHelper::hasGeolocation($userGeo); // Returns: true/false

// مقارنة إحداثيات المستخدم مع إحداثيات الطلب
$isEqual = LocaleHelper::equalsGeolocation($userGeo, $requestGeo); // Returns: true/false

/*
المخرجات (getGeolocationInRequest):
Array
(
    [latitude] => 24.7136
    [longitude] => 46.6753
)
*/
```

### 6. استخراج اللغة من الطلب

```php
// الحصول على اللغة من الطلب (يدمج الرؤوس وبيانات POST)
$localeInRequest = LocaleHelper::getLocaleInRequest('en');

// الحصول على اللغة من رؤوس الطلب فقط
$localeInHeader = LocaleHelper::getLocaleInRequestHeader('en');

// الحصول على اللغة من بيانات POST فقط
$localeInBody = LocaleHelper::getLocaleInRequestBody('en');
```

### 7. الترجمة المتقدمة

```php
// الترجمة الأساسية
$translated = LocaleHelper::translateNano('messages.welcome');

// الترجمة مع الجمع (pluralization)
$translatedPlural = LocaleHelper::translateNano('messages.apples', 5);

// الترجمة مع المعلمات
$translatedWithParams = LocaleHelper::translateNano(
    'messages.greeting',
    null,
    ['name' => 'أحمد']
);

// الترجمة بلغة محددة
$translatedInFrench = LocaleHelper::translateNano(
    'messages.welcome',
    null,
    null,
    'fr'
);

// الترجمة بدون ترجمة الرسائل (استخدام مباشر)
$translatedNoMessage = LocaleHelper::translateNano(
    'messages.welcome',
    null,
    null,
    null,
    true,
    0
);

// الترجمة الخام (Raw)
$translatedRaw = LocaleHelper::translateNano(
    'messages.welcome',
    null,
    null,
    null,
    true,
    1,
    1
);

// دوال مختصرة
$short1 = LocaleHelper::trans_nano('messages.welcome');
$short2 = LocaleHelper::translateString('messages.welcome');
$short3 = LocaleHelper::translateRawString('messages.welcome');
$short4 = LocaleHelper::translatePlural('messages.apples', 5);
$short5 = LocaleHelper::translateRawPlural('messages.apples', 5);

/*
المخرجات (translateNano مع معلمات):
"مرحباً أحمد!" إذا كانت الرسالة: 'messages.greeting' => 'مرحباً :name!'
*/
```

### 8. دوال مساعدة أخرى

```php
// الحصول على جميع رموز اللغات
$allLangCodes = LocaleHelper::get_all_lang_codes();

/*
المخرجات (get_all_lang_codes):
Array
(
    [af] => Afrikaans
    [af_NA] => Afrikaans (Namibia)
    [af_ZA] => Afrikaans (South Africa)
    [agq] => Aghem
    [agq_CM] => Aghem (Cameroon)
    ... وغيرها (مئات اللغات)
)
*/
```

### 9. الحصول على خيارات اللغات الكاملة

```php
// الحصول على جميع خيارات اللغات (مع الأعلام)
$allLocaleOptions = LocaleHelper::getAllLocaleOptions();

/*
المخرجات (getAllLocaleOptions):
Array
(
    [ar] => Array
        (
            [0] => العربية
            [1] => flag-sa
        )
    [be] => Array
        (
            [0] => Беларуская
            [1] => flag-by
        )
    [bg] => Array
        (
            [0] => Български
            [1] => flag-bg
        )
    ... وغيرها
)
*/
```

## أمثلة متقدمة

### مثال 1: استخدام الترجمة مع الجمع المعقد

```php
// في ملف الترجمة
// messages.php:
// 'apples' => '{0} لا توجد تفاحات|{1} تفاحة واحدة|[2,10] :count تفاحات|[11,*] الكثير من التفاحات'

$count = 0;
echo LocaleHelper::translatePlural('messages.apples', $count);
// المخرجات: "لا توجد تفاحات"

$count = 1;
echo LocaleHelper::translatePlural('messages.apples', $count);
// المخرجات: "تفاحة واحدة"

$count = 5;
echo LocaleHelper::translatePlural('messages.apples', $count, ['count' => $count]);
// المخرجات: "5 تفاحات"

$count = 15;
echo LocaleHelper::translatePlural('messages.apples', $count);
// المخرجات: "الكثير من التفاحات"
```

### مثال 2: نظام متكامل لتحديد اللغة

```php
public function determineLocale()
{
    // محاولة الحصول على اللغة من مصادر مختلفة
    $locale = LocaleHelper::getLocaleInRequest();
    
    if (!$locale) {
        $locale = LocaleHelper::localeFromUSER();
    }
    
    if (!$locale) {
        $locale = LocaleHelper::localeFromBrowser();
    }
    
    if (!$locale) {
        $locale = LocaleHelper::getDefaultLocale();
    }
    
    // تعيين اللغة في التطبيق
    app()->setLocale($locale);
    
    // تحديد اتجاه النص
    $direction = LocaleHelper::currentDir($locale);
    
    return [
        'locale' => $locale,
        'direction' => $direction,
        'is_rtl' => LocaleHelper::isRtl($locale)
    ];
}

/*
المخرجات:
Array
(
    [locale] => ar
    [direction] => rtl
    [is_rtl] => true
)
*/
```

### مثال 3: نظام الموقع الجغرافي الذكي

```php
public function getUserLocation($user)
{
    // محاولة الحصول على الموقع من مصادر مختلفة
    $location = LocaleHelper::getCurrentGeolocation($user);
    
    if (!LocaleHelper::hasGeolocation($location)) {
        // استخدام موقع افتراضي
        $location = [
            'latitude' => 24.7136,
            'longitude' => 46.6753,
            'source' => 'default'
        ];
    } else {
        $location['source'] = 'user';
    }
    
    // التحقق من الموقع في الطلب الحالي
    $requestLocation = LocaleHelper::getGeolocationInRequest();
    
    if (LocaleHelper::hasGeolocation($requestLocation)) {
        if (!LocaleHelper::equalsGeolocation($location, $requestLocation)) {
            // تحديث موقع المستخدم إذا اختلف
            LocaleHelper::setUserGeolocation($requestLocation, $user);
            $location = $requestLocation;
            $location['source'] = 'request';
        }
    }
    
    return $location;
}
```

### مثال 4: نظام الترجمة المتقدم مع معالجة الأخطاء

```php
public function translateContent($key, $params = [], $locale = null)
{
    try {
        // محاولة الترجمة مع معالجة الأخطاء المدمجة
        return LocaleHelper::translateNano(
            $key,
            null, // count
            $params,
            $locale,
            true, // fallback
            1,    // is_message
            0,    // is_raw
            null  // is_force_plural
        );
    } catch (Exception $e) {
        // تسجيل الخطأ وإرجاع النص الأصلي
        Log::error('Translation failed', [
            'key' => $key,
            'locale' => $locale,
            'error' => $e->getMessage()
        ]);
        
        return $key; // إرجاع المفتاح كنص بديل
    }
}

// استخدام النظام
$message = $this->translateContent('dashboard.welcome', ['user' => 'أحمد'], 'ar');
```

### مثال 5: تكامل مع واجهة المستخدم

```php
// في الكونترولر
public function getPageData()
{
    return [
        'locale' => LocaleHelper::getLocale(),
        'direction' => LocaleHelper::currentDir(),
        'is_rtl' => LocaleHelper::isRtl(),
        'timezones' => LocaleHelper::timezones(),
        'available_locales' => LocaleHelper::getLocaleOptions(),
        'user_location' => LocaleHelper::getCurrentGeolocation(auth()->user())
    ];
}

// في البلاد فيو (Blade)
<html dir="{{ $direction }}" lang="{{ $locale }}">
<head>
    @if($is_rtl)
        <link rel="stylesheet" href="{{ asset('css/rtl.css') }}">
    @endif
</head>
<body>
    <div class="location-info">
        خط العرض: {{ $user_location['latitude'] ?? 'غير معروف' }}<br>
        خط الطول: {{ $user_location['longitude'] ?? 'غير معروف' }}
    </div>
    
    <select name="timezone">
        @foreach($timezones as $key => $value)
            <option value="{{ $key }}">{{ $value }}</option>
        @endforeach
    </select>
</body>
</html>
```

### مثال 6: نظام الترجمة الديناميكي للنماذج

```php
// ترجمة حقول النموذج ديناميكياً
public function translateModelFields($model, $fields)
{
    $translations = [];
    
    foreach ($fields as $field) {
        $key = strtolower(class_basename($model)) . ".{$field}";
        
        $translations[$field] = LocaleHelper::translateNano($key);
        
        // إذا فشلت الترجمة، استخدام اسم الحقل
        if ($translations[$field] === $key) {
            $translations[$field] = ucfirst(str_replace('_', ' ', $field));
        }
    }
    
    return $translations;
}

// استخدام النظام
$userFields = ['first_name', 'last_name', 'email', 'phone'];
$translated = $this->translateModelFields($user, $userFields);

/*
المخرجات:
Array
(
    [first_name] => الاسم الأول
    [last_name] => اسم العائلة
    [email] => البريد الإلكتروني
    [phone] => الهاتف
)
*/
```

## معالجة الأخطاء

يحتوي الكلاس على نظام متكامل لمعالجة الأخطاء في دوال الترجمة:

```php
try {
    $result = LocaleHelper::translateNano('invalid.key');
} catch (Exception $e) {
    // يتم تسجيل الخطأ تلقائياً إذا كان التصحيح مفعلاً
    // وإرجاع النص الأصلي لمنع تعطل التطبيق
}
```

## إعدادات التكوين

```php
// في config/nano/helpers.php
return [
    'locale_helper' => [
        'is_trace_log_error' => env('APP_DEBUG', true),
        'default_geolocation' => [
            'latitude' => 24.7136,
            'longitude' => 46.6753
        ]
    ]
];
```

## ملاحظات مهمة

1. **الأداء**: يستخدم الكلاس التخزين المؤقت (static caching) لتحسين الأداء
2. **المرونة**: يدعم مصادر متعددة للغة والموقع الجغرافي
3. **التوافق**: يعمل مع Nano2Soft Core و Laravel وأنظمة الترجمة المختلفة
4. **الأمان**: يحتوي على معالجة شاملة للأخطاء ومنع الثغرات
5. **الدولية**: يدعم جميع اللغات واتجاهات النص
6. **التكامل**: يمكن دمجه بسهولة مع أنظمة الترجمة الحالية

## الخلاصة

كلاس `LocaleHelper` يوفر نظاماً متكاملاً لإدارة جميع الجوانب المتعلقة بالإعدادات المحلية في تطبيقات Nano2Soft. يتميز بالمرونة، القوة، وسهولة الاستخدام، مما يجعله حلاً مثالياً للتطبيقات متعددة اللغات التي تحتاج إلى إدارة دقيقة للغة، الوقت، والموقع الجغرافي.