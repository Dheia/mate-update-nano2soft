# ReligionsHelper Class Documentation

## Introduction to the Class

The `ReligionsHelper` class is a helper class written in PHP that provides a comprehensive set of functions for handling global religious data. The class fully supports both Arabic and English and contains a database of over 40 world religions with their Arabic translations.

## Benefits and Uses

1. **Manage religion list**: A database of world religions with appropriate classification.
2. **Multilingual support**: Supports displaying religion names in both Arabic and English.
3. **Religious classification**: Divides religions into categories (Abrahamic, Eastern, Major).
4. **Search functions**: Intelligent search algorithms to find religions.
5. **Input validation**: Functions to validate religion keys.
6. **Export and import**: Supports exporting data in JSON and CSV formats.
7. **Integration with translation systems**: Compatible with RainLab.Translate.

## Data Structure

The class uses a static array `$religions` to store religion data, where:
- **Key**: English name of the religion (used as a unique identifier).
- **Value**: Arabic name of the religion.

## Usage Examples

### 1. Get All Religions

```php
// Get all religions in the current system language
$allReligions = ReligionsHelper::getAllReligions();

// Get all religions in Arabic
$allReligionsInArabic = ReligionsHelper::getAllReligions('ar');

// Get all religions as a key‑value array (suitable for dropdown lists)
$keyValueReligions = ReligionsHelper::getAllReligions('en', true);

/*
Output (when using 'en'):
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
    ... and others
)
*/
```

### 2. Get Main Religions

```php
// Get main religions in the current language
$mainReligions = ReligionsHelper::getMainReligions();

// Get main religions in Arabic
$mainReligionsAr = ReligionsHelper::getMainReligions('ar');

/*
Output (when using 'ar'):
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
    ... other main religions
)
*/
```

### 3. Get Abrahamic Religions

```php
// Get Abrahamic religions in the current language
$abrahamicReligions = ReligionsHelper::getAbrahamicReligions();

// Get Abrahamic religions in English
$abrahamicReligionsEn = ReligionsHelper::getAbrahamicReligions('en');

/*
Output:
Array
(
    [0] => Array
        (
            [key] => Islam
            [name] => Islam (or الإسلام depending on language)
            [name_ar] => الإسلام
            [name_en] => Islam
        )
    [1] => Array
        (
            [key] => Christianity
            [name] => Christianity (or المسيحية depending on language)
            [name_ar] => المسيحية
            [name_en] => Christianity
        )
    ... etc.
)
*/
```

### 4. Get Eastern Religions

```php
// Get Eastern religions in the current language
$easternReligions = ReligionsHelper::getEasternReligions();

// Get Eastern religions in Arabic
$easternReligionsAr = ReligionsHelper::getEasternReligions('ar');
```

### 5. Find a Specific Religion

```php
// Using findReligion
$religion1 = ReligionsHelper::findReligion('Islam'); // by English key
$religion2 = ReligionsHelper::findReligion('الإسلام'); // by Arabic name
$religion3 = ReligionsHelper::findReligion('islam'); // case‑insensitive search

/*
Output (example for search 'Islam'):
Array
(
    [key] => Islam
    [name] => الإسلام  // or Islam depending on current language
    [name_ar] => الإسلام
    [name_en] => Islam
)
*/
```

### 6. Get Religion Name

```php
// Get religion name based on current language
$name1 = ReligionsHelper::getName('Islam', 'en'); // Returns: Islam
$name2 = ReligionsHelper::getName('Islam', 'ar'); // Returns: الإسلام

// Get Arabic name only
$arabicName = ReligionsHelper::getNameAr('Christianity'); // Returns: المسيحية

// Get English name only
$englishName = ReligionsHelper::getNameEn('Christianity'); // Returns: Christianity
```

### 7. Check if a Religion Exists

```php
// Check if a religion exists by key
$exists1 = ReligionsHelper::exists('Islam'); // Returns: true
$exists2 = ReligionsHelper::exists('NonExistent'); // Returns: false
```

### 8. Get Key from Arabic Name

```php
// Get the key (English) from the Arabic name
$key = ReligionsHelper::getKeyByArabicName('الإسلام'); // Returns: Islam
$key2 = ReligionsHelper::getKeyByArabicName('المسيحية'); // Returns: Christianity
```

### 9. Search in Religions

```php
// Search for religions containing a certain word
$searchResults = ReligionsHelper::search('christ', 'en');
$searchResultsAr = ReligionsHelper::search('مسي', 'ar');

/*
Output (search in English):
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

### 10. Get Religions by Type

```php
// Get religions by type
$byType1 = ReligionsHelper::getReligionsByType('abrahamic', 'en');
$byType2 = ReligionsHelper::getReligionsByType('eastern', 'ar');
$byType3 = ReligionsHelper::getReligionsByType('main', 'en');
$byType4 = ReligionsHelper::getReligionsByType('all', 'ar');

/*
Output (getReligionsByType('abrahamic', 'en')):
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
    ... other Abrahamic religions
)
*/
```

### 11. Get Dropdown Options

```php
// All religions for dropdowns
$dropdownAll = ReligionsHelper::getDropdownOptions('ar');

// Only main religions
$dropdownMain = ReligionsHelper::getMainDropdownOptions('en');

// Abrahamic religions
$dropdownAbrahamic = ReligionsHelper::getAbrahamicDropdownOptions('ar');

/*
Output (getDropdownOptions in Arabic):
Array
(
    [Islam] => الإسلام
    [Christianity] => المسيحية
    [Judaism] => اليهودية
    [Hinduism] => الهندوسية
    ... etc.
)
*/
```

### 12. Export and Import

```php
// Export to JSON
$jsonData = ReligionsHelper::getJsonData('en');

// Export to CSV
$csvData = ReligionsHelper::exportToCsv('ar', true);

/*
Output (getJsonData):
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
    ... etc.
]
*/
```

### 13. Miscellaneous Helper Functions

```php
// Get the system default language
$defaultLocale = ReligionsHelper::getDefaultLocale();

// Get the number of available religions
$count = ReligionsHelper::getCount(); // Returns: number of religions

// Get religions sorted alphabetically
$sorted = ReligionsHelper::getSortedReligions('ar', 'name');

// Get a set of religions by specific keys
$selectedReligions = ReligionsHelper::getReligionsByKeys(['Islam', 'Christianity', 'Judaism'], 'en');

/*
Output (getReligionsByKeys):
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

### 14. Check Religion Types

```php
// Check if a religion is Abrahamic
$isAbrahamic = ReligionsHelper::isAbrahamic('Islam'); // Returns: true
$isAbrahamic2 = ReligionsHelper::isAbrahamic('Hinduism'); // Returns: false

// Check if a religion is Eastern
$isEastern = ReligionsHelper::isEastern('Hinduism'); // Returns: true
$isEastern2 = ReligionsHelper::isEastern('Islam'); // Returns: false

// Check if a religion is major
$isMain = ReligionsHelper::isMainReligion('Buddhism'); // Returns: true
$isMain2 = ReligionsHelper::isMainReligion('Taoism'); // Returns: false
```

## Important Notes

1. **Default language**: If the `$locale` parameter is not passed, the class uses the current system language (`\Lang::getLocale()`).

2. **Case sensitivity**: Search functions are case‑insensitive for English names.

3. **Primary key**: The key used in the `$religions` array is the English name of the religion, which is the unique identifier.

4. **Arabic support**: All functions fully support Arabic, both in input and output.

5. **Default values**: Many functions return `null` or an empty array when results are not found.

6. **Religious classification**: Religions are divided into:
   - **Abrahamic**: Islam, Christianity, Judaism, Baháʼí, Druze.
   - **Eastern**: Hinduism, Buddhism, Sikhism, Jainism, Taoism, Confucianism, Shinto.
   - **Major**: Islam, Christianity, Judaism, Hinduism, Buddhism, Sikhism, Atheism, No religion, Other religions.

## Integration Examples

### Example 1: Create a Religion Dropdown in a Form

```php
// In the controller
$religions = ReligionsHelper::getMainDropdownOptions(app()->getLocale());

// In the Blade view
<select name="religion">
    <option value="">Select religion</option>
    @foreach($religions as $key => $name)
        <option value="{{ $key }}" {{ old('religion') == $key ? 'selected' : '' }}>
            {{ $name }}
        </option>
    @endforeach
</select>
```

### Example 2: Search and Filter in the User Interface

```php
// Search for a religion based on user input
$userInput = request()->input('religion_search');
$results = ReligionsHelper::search($userInput, app()->getLocale());

// Display results
if (!empty($results)) {
    foreach ($results as $religion) {
        echo "<div class='religion-item'>";
        echo "<strong>{$religion['name']}</strong> ({$religion['key']})<br>";
        echo "In Arabic: {$religion['name_ar']}<br>";
        echo "</div>";
    }
} else {
    echo "No results found.";
}
```

### Example 3: Validate Religion Input

```php
// In form validation
$validatedData = request()->validate([
    'religion' => ['required', function ($attribute, $value, $fail) {
        if (!ReligionsHelper::exists($value)) {
            $fail('The selected religion is invalid.');
        }
    }],
]);

// Or more simply
$religion = request('religion');
if (ReligionsHelper::exists($religion)) {
    // Religion is valid, continue
    $religionName = ReligionsHelper::getName($religion, app()->getLocale());
} else {
    // Religion is invalid
    session()->flash('error', 'The selected religion does not exist.');
}
```

### Example 4: Handle Multiple Religions for a User

```php
// User follows several religions (in case of multiple beliefs)
$userReligions = ['Islam', 'Christianity'];

// Get information about these religions
$religionsInfo = ReligionsHelper::getReligionsByKeys($userReligions, 'ar');

// Display them to the user
echo "Selected religions:<br>";
foreach ($religionsInfo as $religion) {
    echo "- {$religion['name']} ({$religion['key']})<br>";
    
    // Check the type
    if (ReligionsHelper::isAbrahamic($religion['key'])) {
        echo "  (Abrahamic religion)<br>";
    } elseif (ReligionsHelper::isEastern($religion['key'])) {
        echo "  (Eastern religion)<br>";
    }
}
```

### Example 5: Use in Admin Dashboard

```php
// In the admin dashboard to display religion statistics
$religionsCount = ReligionsHelper::getCount();
$mainReligions = ReligionsHelper::getMainReligions('ar');

echo "<div class='stats-card'>";
echo "<h3>Religion Statistics</h3>";
echo "<p>Number of available religions: {$religionsCount}</p>";
echo "<h4>Major religions:</h4>";
echo "<ul>";

foreach ($mainReligions as $religion) {
    echo "<li>{$religion['name']} ({$religion['key']})</li>";
}

echo "</ul>";
echo "</div>";
```

### Example 6: Export Religion Data

```php
// Export religion data for use in other applications
$jsonData = ReligionsHelper::getJsonData('en');

// Save to a file
file_put_contents('religions_data.json', $jsonData);

// Or return as a JSON response in an API
return response()->json([
    'success' => true,
    'data' => json_decode($jsonData, true),
    'count' => ReligionsHelper::getCount(),
    'exported_at' => now()->toDateTimeString()
]);
```

## Possible Exceptions

1. **Non‑existent keys**: When using non‑existent keys, most functions return `null` or an empty array.

2. **Empty input**: Functions that deal with searching return an empty array when empty input is given.

3. **Encoding issues**: The code is written with full UTF‑8 support, including Arabic characters.

4. **Duplicate names**: Each religion has a unique English key, preventing duplication.

## Summary

The `ReligionsHelper` class provides a comprehensive solution for managing religions in applications that need to handle religious diversity. It features an easy‑to‑use API, strong Arabic support, and logical classification of religions that facilitates searching and filtering. The class is suitable for educational applications, social platforms, and content management systems that need to handle diverse religious data.