# NationalityHelper Class Documentation

## 📋 General Introduction

**NationalityHelper** is a comprehensive PHP class for managing nationalities in a Laravel/Nano2Soft‑based system. It provides a complete database of countries with all necessary information in both Arabic and English, and contains advanced functions for searching, filtering, and converting between various international encoding systems.

## 🎯 Benefits and Uses

### ✅ Main benefits:
- **Unified data source**: Contains information on 240+ countries and nationalities.
- **Bilingual support**: Arabic and English.
- **Comprehensive data**: ISO codes, phone codes, currencies, flags, regions, languages, religions.
- **High performance**: Uses maps for O(1) quick access.
- **Ease of use**: Clear and intuitive API.
- **Easy extensibility**: New countries can be added easily.

### 🛠 Use cases:
1. **Populating dropdowns** for nationalities in registration forms.
2. **Validating** country codes and phone numbers.
3. **Displaying country information** with flags and currencies.
4. **Filtering by region**, language, or religion.
5. **Integration with CRM systems** for customer data management.
6. **Automatic phone code generation** in contact forms.

## 📁 General Structure

```php
<?php namespace Nano\Location\Classes;

class NationalityHelper
{
    // Static array containing all nationalities
    public static $nationalities = [...];
    
    // Maps for quick access
    private static $alpha2Map = [];
    private static $alpha3Map = [];
    private static $numericMap = [];
    private static $phoneCodeMap = [];
    
    // Main retrieval and search methods
    public static function getAllNationalities(...) {...}
    public static function findNationality(...) {...}
    public static function getByAlpha2(...) {...}
    // ... and other functions
}
```

## 📊 Detailed Function Examples

### 1️⃣ **Basic Retrieval Functions**

#### `getAllNationalities()` – Get all nationalities
```php
// Example 1: Get all nationalities in Arabic
$allNationalities = NationalityHelper::getAllNationalities('ar');
/*
Output: an array containing:
[
    [
        'id' => '887',
        'name' => 'Yemeni',
        'country_name' => 'Yemen',
        'alpha2' => 'YE',
        'alpha3' => 'YEM',
        'phone_code' => '967',
        'flag_emoji' => '🇾🇪',
        // ... other fields
    ],
    // ... all other countries
]
*/

// Example 2: Get options for dropdowns
$options = NationalityHelper::getAllNationalities('ar', true);
/*
Output: key‑value array:
[
    'YE' => 'Yemeni',
    'SA' => 'Saudi',
    'AE' => 'Emirati',
    // ...
]
*/

// Example 3: Get in English
$englishNationalities = NationalityHelper::getAllNationalities('en');
```

#### `getAllNationalitiesOptions()` – Smart options for lists
```php
// Example: Get options with caching
$options = NationalityHelper::getAllNationalitiesOptions('ar');
/*
Output: same as getAllNationalities with the second parameter true.
Useful when calling the function repeatedly to avoid re‑processing.
*/
```

### 2️⃣ **Region‑Specific Functions**

#### `getArabNationalities()` – Only Arab nationalities
```php
$arabNationalities = NationalityHelper::getArabNationalities('ar');
/*
Output: array containing only Arab countries (22 countries)
[
    ['name' => 'Saudi', 'alpha2' => 'SA', ...],
    ['name' => 'Egyptian', 'alpha2' => 'EG', ...],
    ['name' => 'Emirati', 'alpha2' => 'AE', ...],
    // ... all Arab countries
]
*/

// In English
$arabNationalitiesEn = NationalityHelper::getArabNationalities('en');
```

#### `getCommonNationalities()` – Common nationalities
```php
$common = NationalityHelper::getCommonNationalities('ar');
/*
Output: Arab countries + major world countries
Contains: Saudi Arabia, Egypt, UAE, USA, UK, Canada, etc.
*/
```

### 3️⃣ **Single Search Functions**

#### `findNationality()` – Search by any type of code
```php
// Search by Alpha2 code
$result1 = NationalityHelper::findNationality('SA', 'ar');
/*
Output:
[
    'id' => '682',
    'name' => 'Saudi',
    'country_name' => 'Saudi Arabia',
    'alpha2' => 'SA',
    'alpha3' => 'SAU',
    'phone_code' => '966',
    // ... etc.
]
*/

// Search by Alpha3 code
$result2 = NationalityHelper::findNationality('SAU', 'ar');
// Same as above

// Search by numeric code
$result3 = NationalityHelper::findNationality('682', 'ar');

// Search by phone code
$result4 = NationalityHelper::findNationality('966', 'ar');

// Search by Arabic name
$result5 = NationalityHelper::findNationality('سعودي', 'ar');

// Search by English name
$result6 = NationalityHelper::findNationality('Saudi', 'en');
```

#### `getByAlpha2()`, `getByAlpha3()`, `getByNumeric()`, `getByPhoneCode()`
```php
// Specialised search methods
$byAlpha2 = NationalityHelper::getByAlpha2('AE', 'ar');
$byAlpha3 = NationalityHelper::getByAlpha3('ARE', 'ar');
$byNumeric = NationalityHelper::getByNumeric('784', 'ar');
$byPhone = NationalityHelper::getByPhoneCode('971', 'ar');

// For comparison: getByPhoneCodeV1 (old version)
$byPhoneV1 = NationalityHelper::getByPhoneCodeV1('971', 'ar');
```

#### `searchByName()` – Text search in names
```php
$results = NationalityHelper::searchByName('سعو', 'ar');
/*
Output: all countries whose names contain "سعو"
[
    ['name' => 'Saudi', ...],
    ['name' => 'Syrian', ...],
    // etc.
]
*/

$resultsEn = NationalityHelper::searchByName('saud', 'en');
// Search in English
```

### 4️⃣ **Conversion Functions**

#### Convert from one system to another
```php
// Alpha2 → Alpha3
$alpha3 = NationalityHelper::convertAlpha2ToAlpha3('SA'); // 'SAU'

// Alpha3 → Alpha2
$alpha2 = NationalityHelper::convertAlpha3ToAlpha2('SAU'); // 'SA'

// Alpha2 → numeric
$numeric = NationalityHelper::convertAlpha2ToNumeric('SA'); // '682'

// Numeric → Alpha2
$alpha2 = NationalityHelper::convertNumericToAlpha2('682'); // 'SA'

// Numeric → Alpha3
$alpha3 = NationalityHelper::convertNumericToAlpha3('682'); // 'SAU'
```

### 5️⃣ **Specific Information Retrieval**

#### `getName()` – Nationality name only
```php
$nameAr = NationalityHelper::getName('SA', 'ar'); // 'Saudi'
$nameEn = NationalityHelper::getName('SA', 'en'); // 'Saudi'
```

#### `getCountryName()` – Country name
```php
$countryAr = NationalityHelper::getCountryName('SA', 'ar'); // 'Saudi Arabia'
$countryEn = NationalityHelper::getCountryName('SA', 'en'); // 'Saudi Arabia'
```

#### `getPhoneCode()` – Phone code
```php
$phoneCode = NationalityHelper::getPhoneCode('SA'); // '966'
$formatted = NationalityHelper::getFormattedPhoneCode('SA'); // '+966'
```

#### `getCurrency()` – Currency
```php
$currency = NationalityHelper::getCurrency('SA'); // ['SAR']
$currencyUS = NationalityHelper::getCurrency('US'); // ['USD']
```

#### `getFlagEmoji()` – Flag emoji
```php
$flag = NationalityHelper::getFlagEmoji('SA'); // '🇸🇦'
$flagUS = NationalityHelper::getFlagEmoji('US'); // '🇺🇸'
```

#### `getFlagUrl()` – Flag image URL
```php
$flagUrl = NationalityHelper::getFlagUrl('SA', 'flat');
// 'https://flagcdn.com/flat/sa.png'

$flagUrlShiny = NationalityHelper::getFlagUrl('SA', 'shiny');
// 'https://flagcdn.com/shiny/sa.png'
```

#### `getRegion()`, `getSubregion()` – Geographic location
```php
$region = NationalityHelper::getRegion('SA'); // 'Asia'
$subregion = NationalityHelper::getSubregion('SA'); // 'Western Asia'
```

#### `getLanguages()`, `getReligions()` – Languages and religions
```php
$languages = NationalityHelper::getLanguages('SA'); // ['ar']
$religions = NationalityHelper::getReligions('SA'); // ['Islam']
```

### 6️⃣ **Filtering by Criteria**

#### `getNationalitiesByRegion()` – By continent
```php
$asianCountries = NationalityHelper::getNationalitiesByRegion('Asia', 'ar');
$europeanCountries = NationalityHelper::getNationalitiesByRegion('Europe', 'ar');
$africanCountries = NationalityHelper::getNationalitiesByRegion('Africa', 'ar');
```

#### `getNationalitiesBySubregion()` – By sub‑region
```php
$arabCountries = NationalityHelper::getNationalitiesBySubregion('Western Asia', 'ar');
$northAfrica = NationalityHelper::getNationalitiesBySubregion('Northern Africa', 'ar');
```

#### `getNationalitiesByLanguage()` – By language
```php
$arabicSpeaking = NationalityHelper::getNationalitiesByLanguage('ar', 'ar');
$englishSpeaking = NationalityHelper::getNationalitiesByLanguage('en', 'en');
$frenchSpeaking = NationalityHelper::getNationalitiesByLanguage('fr', 'fr');
```

#### `getNationalitiesByReligion()` – By religion
```php
$islamicCountries = NationalityHelper::getNationalitiesByReligion('Islam', 'ar');
$christianCountries = NationalityHelper::getNationalitiesByReligion('Christianity', 'en');
```

### 7️⃣ **Dropdown Options Functions**

#### `getDropdownOptions()` – General options
```php
// Simple options
$simpleOptions = NationalityHelper::getDropdownOptions('ar');
/* Output: ['SA' => 'Saudi', 'EG' => 'Egyptian', ...] */

// With phone code
$optionsWithPhone = NationalityHelper::getDropdownOptions('ar', true);
/* Output: ['SA' => 'Saudi (+966)', 'EG' => 'Egyptian (+20)', ...] */

// With country name
$optionsWithCountry = NationalityHelper::getDropdownOptions('ar', false, false, true);
/* Output: ['SA' => 'Saudi Arabia (Saudi)', ...] */

// With all additions
$fullOptions = NationalityHelper::getDropdownOptions('ar', true, true, true);
/* Output: ['SA' => 'Saudi Arabia (Saudi) (+966) - Asia', ...] */
```

#### `getArabDropdownOptions()` – Arab country options
```php
$arabOptions = NationalityHelper::getArabDropdownOptions('ar', true);
/* Output: ['SA' => 'Saudi (+966)', 'EG' => 'Egyptian (+20)', ...] */
```

#### `getCountriesDropdownOptions()` – Country names only
```php
$countries = NationalityHelper::getCountriesDropdownOptions('ar');
/* Output: ['SA' => 'Saudi Arabia', 'EG' => 'Egypt', ...] */
```

#### `getPhoneCodeDropdownOptions()` – Phone codes
```php
$phoneCodes = NationalityHelper::getPhoneCodeDropdownOptions();
/* Output: ['+966' => '+966 (Saudi)', '+20' => '+20 (Egyptian)', ...] */
```

### 8️⃣ **Main List Retrieval**

#### `getAllRegions()` – All continents
```php
$regions = NationalityHelper::getAllRegions();
/* Output: ['Africa', 'Americas', 'Asia', 'Europe', 'Oceania'] */
```

#### `getAllSubregions()` – All sub‑regions
```php
$subregions = NationalityHelper::getAllSubregions();
/* Output: ['Australia and New Zealand', 'Central Asia', ...] */
```

#### `getAllLanguages()` – All languages
```php
$languages = NationalityHelper::getAllLanguages();
/* Output: ['ar', 'en', 'fr', 'es', ...] */
```

#### `getAllReligions()` – All religions
```php
$religions = NationalityHelper::getAllReligions();
/* Output: ['Christianity', 'Islam', 'Buddhism', ...] */
```

### 9️⃣ **Export and Validation Functions**

#### `getJsonData()` – Export to JSON
```php
$jsonData = NationalityHelper::getJsonData('ar');
/* Output: Full JSON containing all data */
```

#### `exportToCsv()` – Export to CSV
```php
$csvData = NationalityHelper::exportToCsv('ar');
/* Output: CSV data that can be saved to a file */
```

#### `validateNationalityData()` – Validate data
```php
$data = [
    'name_ar' => 'Saudi',
    'name_en' => 'Saudi',
    'alpha2' => 'SA',
    // ... etc.
];

$isValid = NationalityHelper::validateNationalityData($data);
// true if data is valid
```

### 🔟 **Helper Functions**

#### `getCount()` – Number of nationalities
```php
$count = NationalityHelper::getCount(); // 240+
```

#### `getArabCountryCodes()` – Arab country codes
```php
$arabCodes = NationalityHelper::getArabCountryCodes();
/* Output: ['SA', 'EG', 'AE', ...] */
```

#### `isArabCountry()` – Check if a country is Arab
```php
$isArab = NationalityHelper::isArabCountry('SA'); // true
$isArab2 = NationalityHelper::isArabCountry('US'); // false
```

#### `isIslamicCountry()` – Check if Islam is the majority religion
```php
$isIslamic = NationalityHelper::isIslamicCountry('SA'); // true
$isIslamic2 = NationalityHelper::isIslamicCountry('US'); // false
```

#### `getDefaultNationality()` – Default nationality
```php
$default = NationalityHelper::getDefaultNationality('ar');
/* Output: full data for Saudi Arabia */
```

## 💡 Complete Practical Examples

### Example 1: User Registration Form
```php
// In the controller
public function onRegister()
{
    $countries = NationalityHelper::getCountriesDropdownOptions('ar', true);
    $this->page['countries'] = $countries;
}

// In the template
<select name="nationality">
    {% for code, name in countries %}
        <option value="{{ code }}">{{ name }}</option>
    {% endfor %}
</select>
```

### Example 2: Display Profile Information
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

### Example 3: Filter Users by Region
```php
public function filterByRegion($region)
{
    $countries = NationalityHelper::getNationalitiesByRegion($region, 'en');
    $countryCodes = array_column($countries, 'alpha2');
    
    return User::whereIn('nationality_code', $countryCodes)->get();
}
```

### Example 4: Generate International Contact List
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

## ⚠️ Important Notes

### Caching
```php
// Functions use internal maps for caching
// The first call initialises the maps, subsequent calls are fast
NationalityHelper::initMaps(); // called automatically
```

### Handling duplicates
```php
// The cleanDuplicateData() function removes duplicates automatically
// Can be called manually if needed
NationalityHelper::cleanDuplicateData();
```

### Common errors
```php
// Ensure the code is valid before use
$result = NationalityHelper::getByAlpha2('XX'); // null (not found)

// Check the result before using it
if ($nationality = NationalityHelper::getByAlpha2('SA')) {
    // use $nationality
} else {
    // handle the error
}
```

## 🔄 Upgrade and Maintenance

### Adding a new country
```php
// Add a new element to the $nationalities array
$newCountry = [
    'name_ar' => 'New',
    'name_en' => 'New',
    'country_name_ar' => 'New Country',
    'country_name_en' => 'New Country',
    'alpha2' => 'NC',
    'alpha3' => 'NEW',
    // ... complete all required fields
];

// Add it to the array
NationalityHelper::$nationalities[] = $newCountry;

// Re‑initialise the maps
NationalityHelper::initMaps();
```

### Updating a country’s data
```php
// Search and modify directly in the array
foreach (NationalityHelper::$nationalities as &$country) {
    if ($country['alpha2'] === 'SA') {
        $country['phone_code'] = '966';
        // ... update other fields
        break;
    }
}
```

## 📈 Class Statistics

- **Number of countries**: 240+ countries and territories
- **Number of fields per country**: 19 fields
- **Supported languages**: Arabic and English
- **Supported encoding systems**: Alpha2, Alpha3, Numeric, Phone Code
- **Geographic classifications**: 5 continents, 25+ sub‑regions
- **Available languages**: 100+ languages
- **Available religions**: 15+ religions

---

## 🎉 Conclusion

The **NationalityHelper** class represents a comprehensive solution for managing nationalities in Arabic and international applications. Its flexible design and high performance make it suitable for both small and large applications. The clear functions and extensive documentation facilitate integration and maintenance.

