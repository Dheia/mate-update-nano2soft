# LanguagesHelper Class Documentation

## Introduction to the Class

The `LanguagesHelper` class is a helper class written in PHP that provides a comprehensive set of functions for handling global language data. The class fully supports both Arabic and English and contains a database of languages with standard ISO 639-1 codes.

## Benefits and Uses

1. **Manage language list**: Contains a database of over 80 languages.
2. **Multilingual support**: Supports displaying language names in both Arabic and English.
3. **Advanced search functions**: Provides intelligent search algorithms to find languages.
4. **Geographic classification**: Divides languages into groups (European, Asian, African).
5. **Input validation**: Functions to validate language codes and convert them.
6. **Export and import**: Supports exporting data in JSON and CSV formats.
7. **Integration with translation systems**: Compatible with RainLab.Translate.

## Data Structure

The class uses a static array `$languages` to store language data, where:
- **Key**: Two‑letter language code (ISO 639-1).
- **Value**: Array containing the language name in English and Arabic.

## Usage Examples

### 1. Get All Languages

```php
// Get all languages in the current system language
$allLanguages = LanguagesHelper::getAllLanguages();

// Get all languages in Arabic
$allLanguagesInArabic = LanguagesHelper::getAllLanguages('ar');

// Get all languages as a key‑value array (suitable for dropdown lists)
$keyValueLanguages = LanguagesHelper::getAllLanguages('en', true);

/*
Output (when using 'en'):
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
    ... and others
)
*/
```

### 2. Get Main Languages

```php
// Get main languages in the current language
$mainLanguages = LanguagesHelper::getMainLanguages();

// Get main languages in Arabic
$mainLanguagesAr = LanguagesHelper::getMainLanguages('ar');

/*
Output (when using 'ar'):
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
    ... other main languages
)
*/
```

### 3. Find a Specific Language

```php
// Using findLanguage (improved method)
$language1 = LanguagesHelper::findLanguage('ar'); // by code
$language2 = LanguagesHelper::findLanguage('english'); // by English name
$language3 = LanguagesHelper::findLanguage('العربية'); // by Arabic name
$language4 = LanguagesHelper::findLanguage('eng'); // partial search

/*
Output (example for search 'ar'):
Array
(
    [code] => ar
    [name] => العربية  // or Arabic depending on current language
    [name_ar] => العربية
    [name_en] => Arabic
)
*/
```

### 4. Advanced Multi‑Language Search

```php
// Advanced search with result limit
$advancedResults = LanguagesHelper::findLanguagesAdvanced('ch', 'en', 5);

/*
Output:
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
            [name] => Chinese (old code)
            [name_ar] => الصينية
            [name_en] => Chinese
            [score] => 70
            [match_type] => starts_with
        )
    ... etc.
)
*/
```

### 5. Get Language Name

```php
// Get language name based on current language
$name1 = LanguagesHelper::getName('ar', 'en'); // Returns: Arabic
$name2 = LanguagesHelper::getName('ar', 'ar'); // Returns: العربية

// Get Arabic name only
$arabicName = LanguagesHelper::getNameAr('fr'); // Returns: الفرنسية

// Get English name only
$englishName = LanguagesHelper::getNameEn('fr'); // Returns: French
```

### 6. Get Language Code from Name

```php
// Get code from English name
$code1 = LanguagesHelper::getCodeByName('Arabic'); // Returns: ar

// Get code from Arabic name
$code2 = LanguagesHelper::getCodeByArabicName('الفرنسية'); // Returns: fr

// Search using getCode (shortcut function)
$code3 = LanguagesHelper::getCode('español'); // Returns: es
$code4 = LanguagesHelper::getCode('invalid', 'en'); // Returns: en (default value)
```

### 7. Smart Language Search

```php
// Basic search
$searchResults = LanguagesHelper::search('ger', 'en');

// Smart search with advanced options
$smartResults = LanguagesHelper::smartSearch('ger', 'en', [
    'limit' => 5,
    'prioritize_code' => true,
    'include_match_info' => true
]);

/*
Output (smartSearch):
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
    ... etc.
)
*/
```

### 8. Get Languages by Geographic Region

```php
// European languages
$european = LanguagesHelper::getEuropeanLanguages('en');

// Asian languages
$asian = LanguagesHelper::getAsianLanguages('ar');

// African languages
$african = LanguagesHelper::getAfricanLanguages('en');

// Get by type
$byType = LanguagesHelper::getLanguagesByType('european', 'en');
```

### 9. Check Language Types

```php
// Check if language is European
$isEuropean = LanguagesHelper::isEuropean('fr'); // Returns: true
$isEuropean2 = LanguagesHelper::isEuropean('ar'); // Returns: false

// Check if language is Asian
$isAsian = LanguagesHelper::isAsian('ja'); // Returns: true

// Check if language is African
$isAfrican = LanguagesHelper::isAfrican('sw'); // Returns: true

// Check if language is main
$isMain = LanguagesHelper::isMainLanguage('zh'); // Returns: true
```

### 10. Get Dropdown Options

```php
// All languages for dropdowns
$dropdownAll = LanguagesHelper::getDropdownOptions('ar');

// Only main languages
$dropdownMain = LanguagesHelper::getMainDropdownOptions('en');

// European languages
$dropdownEuropean = LanguagesHelper::getEuropeanDropdownOptions('ar');

/*
Output (getDropdownOptions in Arabic):
Array
(
    [ar] => العربية
    [en] => الإنجليزية
    [fr] => الفرنسية
    ... etc.
)
*/
```

### 11. Export and Import

```php
// Export to JSON
$jsonData = LanguagesHelper::getJsonData('en');

// Export to CSV
$csvData = LanguagesHelper::exportToCsv('ar', true);

/*
Output (getJsonData):
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
    ... etc.
}
*/
```

### 12. Validate Language Codes

```php
// Simple validation
$validCode = LanguagesHelper::getCheckCode('english', 'en');

// Advanced validation
$validatedCode = LanguagesHelper::getValidatedCode('español', [
    'strict' => false,
    'fallback' => 'en',
    'return_info' => true
]);

/*
Output (getValidatedCode with return_info = true):
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

// Validation with error reasons
$validation = LanguagesHelper::validateCode('invalid', true);

/*
Output on error:
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

### 13. Get Additional Information

```php
// Get countries where the language is official
$countries = LanguagesHelper::getCountryForLanguage('ar');
// Returns: ['SA', 'EG', 'AE', 'MA', 'DZ', ...]

// Get extra language information
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

// Get quick search suggestions
$suggestions = LanguagesHelper::getQuickSearchSuggestions('chi', 'en', 3);
```

### 14. Miscellaneous Helper Functions

```php
// Get the system default locale
$defaultLocale = LanguagesHelper::getDefaultLocale();

// Get the number of available languages
$count = LanguagesHelper::getCount(); // Returns: number of languages

// Get languages sorted alphabetically
$sorted = LanguagesHelper::getSortedLanguages('ar', 'name');

// Get a set of languages by specific codes
$selectedLanguages = LanguagesHelper::getLanguagesByCodes(['ar', 'en', 'fr'], 'en');

// Get a single language by code
$singleLanguage = LanguagesHelper::getLanguagesByCode('ja', 'ar');
```

## Important Notes

1. **Default language**: If the `$locale` parameter is not passed, the class uses the current system language (`\Lang::getLocale()`).

2. **Case sensitivity**: Most search functions are case‑insensitive.

3. **Search priorities**: The search functions use intelligent algorithms that give different priorities to match types.

4. **Arabic support**: All functions fully support Arabic, both in input and output.

5. **Default values**: Many functions support specifying default values when results are not found.

6. **Performance**: The code is optimised for good performance even with a large number of languages.

## Integration Examples

### Example 1: Create a Language Dropdown in a Form

```php
// In the controller
$languages = LanguagesHelper::getMainDropdownOptions(app()->getLocale());

// In the Blade view
<select name="language">
    @foreach($languages as $code => $name)
        <option value="{{ $code }}">{{ $name }}</option>
    @endforeach
</select>
```

### Example 2: Search and Filter in the User Interface

```php
// Search for a language based on user input
$userInput = request()->input('language_search');
$results = LanguagesHelper::findLanguagesAdvanced($userInput, app()->getLocale(), 10);

// Display results
foreach ($results as $language) {
    echo "{$language['name']} ({$language['code']}) - Score: {$language['score']}<br>";
}
```

### Example 3: Validate Language Input

```php
// In form validation
$validatedData = request()->validate([
    'language' => ['required', function ($attribute, $value, $fail) {
        if (!LanguagesHelper::validateCode($value)) {
            $fail('The selected language is invalid.');
        }
    }],
]);

// Or using getCode with a default value
$languageCode = LanguagesHelper::getCode(request('language'), 'en');
```

### Example 4: Process a User’s Language List

```php
// User speaks several languages
$userLanguages = ['ar', 'en', 'fr'];

// Get information about these languages
$languagesInfo = LanguagesHelper::getLanguagesByCodes($userLanguages, 'ar');

// Display them to the user
foreach ($languagesInfo as $lang) {
    echo "{$lang['name']} ({$lang['code']})<br>";
}
```

## Possible Exceptions

1. **Non‑existent codes**: When using non‑existent codes, most functions return `null` or the specified default value.

2. **Empty input**: Functions that deal with searching return an empty array or `null` when empty input is given.

3. **Encoding issues**: The code is written with full UTF‑8 support, including Arabic characters.

## Summary

The `LanguagesHelper` class provides a comprehensive solution for managing languages in multilingual applications. It features an easy‑to‑use API, strong Arabic support, and intelligent search algorithms, making it suitable for applications that need to handle multiple languages efficiently.
