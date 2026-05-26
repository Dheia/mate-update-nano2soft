# LocaleHelper Class Documentation

## Introduction to the Class

The `LocaleHelper` class is an advanced helper class written in PHP that provides comprehensive functions for managing locale settings in Nano2Soft applications. It handles language, time, geographic location, and translation in an integrated manner with Nano2Soft and RainLab.Translate.

## Benefits and Uses

1. **Language management**: Multi‑language support with text direction detection (RTL/LTR).
2. **Time handling**: Managing time zones and conversions.
3. **Geography**: Obtaining and processing geographic location coordinates.
4. **Advanced translation**: Intelligent translation functions with pluralisation support.
5. **System integration**: Integration with RainLab.Translate and Nano2Soft Backend.
6. **Property querying**: Extracting linguistic and geographic information from requests.
7. **Error handling**: Comprehensive error handling with tracing support.

## Constants and Static Variables

```php
// List of RTL languages
private const RTL = [
    'ar', 'ae', 'arc', 'bcc', 'bqi', 'ckb', 'dv', 'fa', 'glk', 'ha',
    'he', 'khw', 'ks', 'ku', 'lrc', 'mzn', 'nqo', 'pnb', 'ps', 'sd',
    'ug', 'ur', 'uz_AF', 'yi'
];

// Temporary request storage variables
public static $locale_in_request;
public static $locale_in_request_header;
public static $locale_in_request_body;
```

## Usage Examples

### 1. Determine Text Direction (RTL/LTR)

```php
// Get text direction for the current language
$direction = LocaleHelper::currentDir(); // Returns: 'rtl' or 'ltr'

// Get text direction for a specific language
$directionAr = LocaleHelper::currentDir('ar'); // Returns: 'rtl'
$directionEn = LocaleHelper::currentDir('en'); // Returns: 'ltr'

// Check if a language is RTL
$isRtlAr = LocaleHelper::isRtl('ar'); // Returns: true
$isRtlEn = LocaleHelper::isRtl('en'); // Returns: false

// Alternative function for direction
$dir = LocaleHelper::language_direction('fa'); // Returns: 'rtl'

/*
Output (currentDir):
'rtl' for languages: Arabic, Persian, Hebrew, Urdu, etc.
'ltr' for languages: English, French, German, etc.
*/
```

### 2. Get Locale Settings

```php
// Get current system language
$currentLocale = LocaleHelper::getLocale();

// Get default system language
$defaultLocale = LocaleHelper::getDefaultLocale();

// Check if translate plugin exists
$hasTranslate = LocaleHelper::hasTranslatePlugin(); // Returns: true/false

// Check if multiple locales are available
$hasMultiple = LocaleHelper::hasMulitpleLocales(); // Returns: true/false

// Get available locale options
$localeOptions = LocaleHelper::getLocaleOptions();

/*
Output (getLocaleOptions):
Array
(
    [ar] => العربية
    [en] => English
    [fr] => Français
    ... etc.
)
*/
```

### 3. Get Language from Different Sources

```php
// Get language from user (control panel)
$userLocale = LocaleHelper::localeFromUSER();

// Get language from URL
$urlLocale = LocaleHelper::localeFromURL();

// Get language from POST data
$postLocale = LocaleHelper::localeFromPost();

// Get language from session
$sessionLocale = LocaleHelper::localeFromSession();

// Get language from browser
$browserLocale = LocaleHelper::localeFromBrowser();

// Get list of accepted languages from browser
$browserLangs = LocaleHelper::getLocaleFromBrowser();

/*
Output (getLocaleFromBrowser):
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
    ... etc.
)
*/
```

### 4. Manage Time Zones

```php
// Get all time zones
$timezones = LocaleHelper::timezones();

/*
Output (timezones):
Array
(
    [Africa/Abidjan] => (+00:00) Africa Abidjan
    [Africa/Accra] => (+00:00) Africa Accra
    [Africa/Addis_Ababa] => (+03:00) Africa Addis Ababa
    [Africa/Algiers] => (+01:00) Africa Algiers
    ... etc.
)
*/
```

### 5. Handle Geographic Location

```php
// Get geographic location from request
$requestGeo = LocaleHelper::getGeolocationInRequest();

// Get from MaxMindGeoApi service
$maxmindGeo = LocaleHelper::getGeolocationInMaxMindGeoApi();

// Get from freegeoip.net service
$freegeoipGeo = LocaleHelper::getGeolocationInGeoIP();

// Get user geographic location
$userGeo = LocaleHelper::getUserGeolocation($user);

// Set user geographic location
LocaleHelper::setUserGeolocation(['latitude' => 24.7136, 'longitude' => 46.6753], $user);

// Set only user latitude
LocaleHelper::setUserLatitude(24.7136, $user);

// Set only user longitude
LocaleHelper::setUserLongitude(46.6753, $user);

// Get current location (combines multiple sources)
$currentGeo = LocaleHelper::getCurrentGeolocation($user, ['latitude' => 0, 'longitude' => 0]);

// Get coordinates as a string
$coordinates = LocaleHelper::getCurrentCoordinates($user); // Returns: "24.7136,46.6753"

// Get coordinates as an object
$coordsObject = LocaleHelper::getCurrentCoordinates($user, null, true);

// Check if location exists
$hasGeo = LocaleHelper::hasGeolocation($userGeo); // Returns: true/false

// Compare user location with request location
$isEqual = LocaleHelper::equalsGeolocation($userGeo, $requestGeo); // Returns: true/false

/*
Output (getGeolocationInRequest):
Array
(
    [latitude] => 24.7136
    [longitude] => 46.6753
)
*/
```

### 6. Extract Language from Request

```php
// Get language from request (combines headers and POST data)
$localeInRequest = LocaleHelper::getLocaleInRequest('en');

// Get language from request headers only
$localeInHeader = LocaleHelper::getLocaleInRequestHeader('en');

// Get language from POST data only
$localeInBody = LocaleHelper::getLocaleInRequestBody('en');
```

### 7. Advanced Translation

```php
// Basic translation
$translated = LocaleHelper::translateNano('messages.welcome');

// Translation with pluralisation
$translatedPlural = LocaleHelper::translateNano('messages.apples', 5);

// Translation with parameters
$translatedWithParams = LocaleHelper::translateNano(
    'messages.greeting',
    null,
    ['name' => 'Ahmed']
);

// Translation in a specific language
$translatedInFrench = LocaleHelper::translateNano(
    'messages.welcome',
    null,
    null,
    'fr'
);

// Translation without message translation (direct usage)
$translatedNoMessage = LocaleHelper::translateNano(
    'messages.welcome',
    null,
    null,
    null,
    true,
    0
);

// Raw translation
$translatedRaw = LocaleHelper::translateNano(
    'messages.welcome',
    null,
    null,
    null,
    true,
    1,
    1
);

// Shortcut functions
$short1 = LocaleHelper::trans_nano('messages.welcome');
$short2 = LocaleHelper::translateString('messages.welcome');
$short3 = LocaleHelper::translateRawString('messages.welcome');
$short4 = LocaleHelper::translatePlural('messages.apples', 5);
$short5 = LocaleHelper::translateRawPlural('messages.apples', 5);

/*
Output (translateNano with parameters):
"Hello Ahmed!" if the message is: 'messages.greeting' => 'Hello :name!'
*/
```

### 8. Other Helper Functions

```php
// Get all language codes
$allLangCodes = LocaleHelper::get_all_lang_codes();

/*
Output (get_all_lang_codes):
Array
(
    [af] => Afrikaans
    [af_NA] => Afrikaans (Namibia)
    [af_ZA] => Afrikaans (South Africa)
    [agq] => Aghem
    [agq_CM] => Aghem (Cameroon)
    ... etc. (hundreds of languages)
)
*/
```

### 9. Get Full Language Options

```php
// Get all language options (with flags)
$allLocaleOptions = LocaleHelper::getAllLocaleOptions();

/*
Output (getAllLocaleOptions):
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
    ... etc.
)
*/
```

## Advanced Examples

### Example 1: Using Translation with Complex Pluralisation

```php
// In translation file
// messages.php:
// 'apples' => '{0} No apples|{1} One apple|[2,10] :count apples|[11,*] many apples'

$count = 0;
echo LocaleHelper::translatePlural('messages.apples', $count);
// Output: "No apples"

$count = 1;
echo LocaleHelper::translatePlural('messages.apples', $count);
// Output: "One apple"

$count = 5;
echo LocaleHelper::translatePlural('messages.apples', $count, ['count' => $count]);
// Output: "5 apples"

$count = 15;
echo LocaleHelper::translatePlural('messages.apples', $count);
// Output: "many apples"
```

### Example 2: Integrated Language Determination System

```php
public function determineLocale()
{
    // Try to get language from different sources
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
    
    // Set the language in the application
    app()->setLocale($locale);
    
    // Determine text direction
    $direction = LocaleHelper::currentDir($locale);
    
    return [
        'locale' => $locale,
        'direction' => $direction,
        'is_rtl' => LocaleHelper::isRtl($locale)
    ];
}

/*
Output:
Array
(
    [locale] => ar
    [direction] => rtl
    [is_rtl] => true
)
*/
```

### Example 3: Smart Geographic Location System

```php
public function getUserLocation($user)
{
    // Try to get location from various sources
    $location = LocaleHelper::getCurrentGeolocation($user);
    
    if (!LocaleHelper::hasGeolocation($location)) {
        // Use default location
        $location = [
            'latitude' => 24.7136,
            'longitude' => 46.6753,
            'source' => 'default'
        ];
    } else {
        $location['source'] = 'user';
    }
    
    // Check location in the current request
    $requestLocation = LocaleHelper::getGeolocationInRequest();
    
    if (LocaleHelper::hasGeolocation($requestLocation)) {
        if (!LocaleHelper::equalsGeolocation($location, $requestLocation)) {
            // Update user location if different
            LocaleHelper::setUserGeolocation($requestLocation, $user);
            $location = $requestLocation;
            $location['source'] = 'request';
        }
    }
    
    return $location;
}
```

### Example 4: Advanced Translation System with Error Handling

```php
public function translateContent($key, $params = [], $locale = null)
{
    try {
        // Attempt translation with built‑in error handling
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
        // Log the error and return the original key
        Log::error('Translation failed', [
            'key' => $key,
            'locale' => $locale,
            'error' => $e->getMessage()
        ]);
        
        return $key; // Return the key as a fallback text
    }
}

// Using the system
$message = $this->translateContent('dashboard.welcome', ['user' => 'Ahmed'], 'ar');
```

### Example 5: Integration with the User Interface

```php
// In the controller
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

// In the Blade view
<html dir="{{ $direction }}" lang="{{ $locale }}">
<head>
    @if($is_rtl)
        <link rel="stylesheet" href="{{ asset('css/rtl.css') }}">
    @endif
</head>
<body>
    <div class="location-info">
        Latitude: {{ $user_location['latitude'] ?? 'Unknown' }}<br>
        Longitude: {{ $user_location['longitude'] ?? 'Unknown' }}
    </div>
    
    <select name="timezone">
        @foreach($timezones as $key => $value)
            <option value="{{ $key }}">{{ $value }}</option>
        @endforeach
    </select>
</body>
</html>
```

### Example 6: Dynamic Model Field Translation

```php
// Dynamically translate model fields
public function translateModelFields($model, $fields)
{
    $translations = [];
    
    foreach ($fields as $field) {
        $key = strtolower(class_basename($model)) . ".{$field}";
        
        $translations[$field] = LocaleHelper::translateNano($key);
        
        // If translation fails, use the field name
        if ($translations[$field] === $key) {
            $translations[$field] = ucfirst(str_replace('_', ' ', $field));
        }
    }
    
    return $translations;
}

// Using the system
$userFields = ['first_name', 'last_name', 'email', 'phone'];
$translated = $this->translateModelFields($user, $userFields);

/*
Output:
Array
(
    [first_name] => First Name
    [last_name] => Last Name
    [email] => Email
    [phone] => Phone
)
*/
```

## Error Handling

The class contains an integrated error handling system for translation functions:

```php
try {
    $result = LocaleHelper::translateNano('invalid.key');
} catch (Exception $e) {
    // The error is automatically logged if debugging is enabled
    // and the original key is returned to prevent application breakdown
}
```

## Configuration Settings

```php
// In config/nano/helpers.php
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

## Important Notes

1. **Performance**: Uses static caching to improve performance.
2. **Flexibility**: Supports multiple sources for language and geographic location.
3. **Compatibility**: Works with Nano2Soft Core, Laravel, and various translation systems.
4. **Security**: Includes comprehensive error handling and vulnerability prevention.
5. **Internationalisation**: Supports all languages and text directions.
6. **Integration**: Can be easily integrated with existing translation systems.

## Summary

The `LocaleHelper` class provides an integrated system for managing all aspects of locale settings in Nano2Soft applications. It is flexible, powerful, and easy to use, making it an ideal solution for multilingual applications that require precise management of language, time, and geographic location.
