# RepeaterFieldsData Class Documentation

## Introduction

The `RepeaterFieldsData` class is part of the `Tss\Basic` plugin in OctoberCMS, designed to facilitate handling repeatable fields of type `jsonable`. It supports five main fields: `phone`, `email`, `website`, `properties`, and `links`, which are frequently used in data models to store multiple pieces of information such as a list of phone numbers, email addresses, website links, etc.

Since these fields may be passed to the API in various formats (single string, array of strings, single object, array of objects, mixed), the class normalizes these inputs into a unified structure that can be saved to the database via the `jsonable` property. It also provides methods for output formatting, validation, merging, and testing.

---

## Benefits and Features

- **Unified Structure** – Converts all varied input formats into a consistent array of items, ensuring data integrity when stored.
- **High Flexibility** – Accepts single strings, string arrays, single objects, object arrays, and even mixed combinations.
- **YAML-Based Definitions** – Reads field definition files (`repeater_fields_*.yaml`) to extract default values and field types, making it adjustable without code changes.
- **Handles Different Field Types** – Intelligently processes `switch`, `dropdown`, and `mediafinder` fields (converts boolean-like values to `'1'`/`'0'`, resolves media paths to full URLs).
- **Comprehensive Methods** – Provides methods for input (`processInput`), output (`formatOutput`), extracting main values (`extractMainValues`), merging (`mergeData`), validation (`validate`), and testing (`runTests`).
- **API-Ready** – Ensures data is formatted appropriately for responses, with options to filter hidden items.
- **Performance Optimized** – Caches field definitions after loading from YAML to prevent repeated disk reads.

---

## Use Cases

- In APIs when receiving data from clients for repeatable fields.
- When saving models that contain `jsonable` properties of repeater type.
- When displaying data in API responses or in templates after formatting.
- When performing partial updates (e.g., merging new lists with old ones).
- To validate data before saving.
- To test the robustness of processing via built-in test methods.

---

## Class Methods and Properties

### 1. `processInput(string $fieldName, mixed $input): array`
- **Description**: Converts various input formats into a unified array ready for storage.
- **Parameters**:
  - `$fieldName`: Field name (phone, email, website, properties, links).
  - `$input`: Input data (string, array, object, null).
- **Returns**: An array containing normalized items with default values and an added `_group` field.

### 2. `formatOutput(string $fieldName, mixed $storedValue, bool $filterHidden = false, bool $resolveMedia = true): array`
- **Description**: Formats stored data for output (API display).
- **Parameters**:
  - `$fieldName`: Field name.
  - `$storedValue`: The stored value (usually an array).
  - `$filterHidden`: If `true`, excludes items where `is_show = '0'`.
  - `$resolveMedia`: If `true`, converts `website_icon` paths to full URLs using `MediaLibrary`.
- **Returns**: An array formatted for output.

### 3. `extractMainValues(string $fieldName, array $data, bool $includeDefaults = false): array`
- **Description**: Extracts only the main values from the data (e.g., phone numbers or email addresses).
- **Parameters**:
  - `$fieldName`: Field name.
  - `$data`: Data array (after `processInput` or from storage).
  - `$includeDefaults`: If `true`, includes only default items (`is_default = '1'`).
- **Returns**: An array of values (strings).

### 4. `mergeData(string $fieldName, array $oldData, array $newData, bool $replace = false): array`
- **Description**: Merges new data with old data (for partial updates).
- **Parameters**:
  - `$fieldName`: Field name.
  - `$oldData`: Old data (array).
  - `$newData`: New data (can be in any format).
  - `$replace`: If `true`, completely replaces old data with new data.
- **Returns**: The merged data array.

### 5. `validate(string $fieldName, array $data): array`
- **Description**: Validates data against field definitions (required fields, allowed dropdown values).
- **Parameters**:
  - `$fieldName`: Field name.
  - `$data`: Data to validate (array).
- **Returns**: An array containing `valid` (boolean) and `errors` (array of error messages).

### 6. `runTests(): array`
- **Description**: Runs a suite of tests on all supported fields to ensure the class works correctly.
- **Returns**: A detailed report containing a summary and results for each test case.

### 7. `getSupportedFields(): array`
- **Description**: Returns a list of supported field names.
- **Returns**: An array of strings.

---

## Illustrative Examples for Each Field

In all examples, assume the class is imported as follows:
```php
use Tss\Basic\Classes\RepeaterFieldsData;
$repeater = new RepeaterFieldsData();
```

### Phone Field

#### Example 1: Single String
**Input:**
```php
$input = "770529482";
$result = $repeater->processInput('phone', $input);
```
**Output (after processInput):**
```php
[
    [
        'phone_label'  => '',
        'phone_number' => '770529482',
        'phone_type'   => null,
        'sort_show'    => null,
        'is_default'   => '1', // default value from YAML
        'is_show'      => '1',
        'phone_note'   => '',
        '_group'       => 'phone',
    ]
]
```

#### Example 2: Array of Strings
**Input:**
```php
$input = ["770529482", "780505400"];
$result = $repeater->processInput('phone', $input);
```
**Output:**
```php
[
    ['phone_number' => '770529482', '_group' => 'phone', ...],
    ['phone_number' => '780505400', '_group' => 'phone', ...]
]
```

#### Example 3: Single Complete Object
**Input:**
```php
$input = [
    'phone_label'  => 'WhatsApp',
    'phone_number' => '+967780505400',
    'phone_type'   => 'mobile',
    'sort_show'    => '1',
    'is_default'   => '1',
    'is_show'      => '1',
    'phone_note'   => 'Note',
];
$result = $repeater->processInput('phone', $input);
```
**Output:**
```php
[
    [
        'phone_label'  => 'WhatsApp',
        'phone_number' => '+967780505400',
        'phone_type'   => 'mobile',
        'sort_show'    => '1',
        'is_default'   => '1',
        'is_show'      => '1',
        'phone_note'   => 'Note',
        '_group'       => 'phone',
    ]
]
```

#### Example 4: Array of Objects
**Input:**
```php
$input = [
    ['phone_number' => '111111111', 'is_default' => '0'],
    ['phone_number' => '222222222', 'is_default' => '1']
];
$result = $repeater->processInput('phone', $input);
```
**Output:**
```php
[
    ['phone_number' => '111111111', 'is_default' => '0', '_group' => 'phone', ...],
    ['phone_number' => '222222222', 'is_default' => '1', '_group' => 'phone', ...]
]
```

#### Example 5: Mixed (Strings + Objects)
**Input:**
```php
$input = [
    "333333333",
    ['phone_number' => '444444444', 'is_default' => '0'],
    "555555555"
];
$result = $repeater->processInput('phone', $input);
```
**Output:** Three items (string items are converted to objects with the main field `phone_number`).

---

### Email Field

#### Example: Single String
**Input:** `"test@example.com"`
**Output:** One item with `email_text = test@example.com` and other default values.

#### Example: Array of Partial Objects
**Input:**
```php
[
    ['email_text' => 'one@site.com'],
    ['email_text' => 'two@site.com', 'is_default' => '0']
]
```
**Output:** Two items with default values merged.

---

### Website Field

#### Example: With Icon (mediafinder)
**Input:**
```php
[
    'website_label' => 'Facebook',
    'website_url'   => 'https://fb.com/page',
    'website_icon'  => 'icons/fb.png'
]
```
**Output (after processInput):** The relative path is preserved.
**When formatOutput with resolveMedia = true:**
```php
// `icons/fb.png` is converted to the full media URL
'website_icon' => 'https://example.com/storage/app/media/icons/fb.png'
```

---

### Properties Field

#### Example: Single String (treated as the main field `value`)
**Input:** `"Test value"`
**Output:**
```php
[
    [
        'title'      => null,   // default values from YAML
        'code'       => null,
        'value'      => 'Test value',
        'is_default' => '1',
        'is_show'    => '1',
        'sort_show'  => null,
        '_group'     => 'properties',
    ]
]
```

#### Example: Complete Object
**Input:**
```php
[
    'title' => 'Color',
    'code'  => 'color',
    'value' => 'Red'
]
```
**Output:** One item with the provided values.

---

### Links Field

#### Example: Single String (url)
**Input:** `"https://example.com"`
**Output:** One item with `url = https://example.com` and other default values (e.g., `target = _blank`).

#### Example: Array of Objects with Different Targets
**Input:**
```php
[
    ['title' => 'Link1', 'url' => 'https://link1.com'],
    ['title' => 'Link2', 'url' => 'https://link2.com', 'target' => '_self']
]
```
**Output:** Two items, the first with default target `_blank`, the second with `_self`.

---

## Advanced Complete Examples

### Example 1: Using the Class in a Controller to Receive API Data and Save to a Model

Assume we have a `Department` model with `jsonable = ['phone', 'email', 'website']`.

```php
use Tss\Basic\Classes\RepeaterFieldsData;
use Tss\Basic\Models\Department;

class DepartmentController
{
    public function store()
    {
        $data = input(); // request data

        $repeater = new RepeaterFieldsData();

        // Prepare repeatable fields
        $department = new Department();
        $department->name = $data['name'];
        $department->phone = $repeater->processInput('phone', $data['phone'] ?? null);
        $department->email = $repeater->processInput('email', $data['email'] ?? null);
        $department->website = $repeater->processInput('website', $data['website'] ?? null);

        $department->save();

        return response()->json([
            'success' => true,
            'data'    => $department
        ]);
    }
}
```

### Example 2: Partial Update (Merging New Data with Old)

```php
public function update($id)
{
    $department = Department::find($id);
    $data = input();

    $repeater = new RepeaterFieldsData();

    if (isset($data['phone'])) {
        // Merge new phones with old ones without full replacement
        $oldPhones = $department->phone ?? [];
        $department->phone = $repeater->mergeData('phone', $oldPhones, $data['phone']);
    }

    if (isset($data['email'])) {
        // Completely replace email
        $department->email = $repeater->processInput('email', $data['email']);
    }

    $department->save();

    return response()->json($department);
}
```

### Example 3: Formatting Output with Hidden Filtering and Media Resolution

```php
public function show($id)
{
    $department = Department::with('someRelation')->find($id);

    $repeater = new RepeaterFieldsData();

    $response = [
        'id'   => $department->id,
        'name' => $department->name,
        'phone' => $repeater->formatOutput('phone', $department->phone, true), // hide non-visible
        'email' => $repeater->formatOutput('email', $department->email, false), // show all
        'website' => $repeater->formatOutput('website', $department->website, true, true), // hide + resolve media
    ];

    return response()->json($response);
}
```

### Example 4: Extracting Only Default Phone Numbers

```php
$phones = $department->phone; // stored data
$defaultPhoneNumbers = $repeater->extractMainValues('phone', $phones, true);
// Returns an array of phone numbers where is_default = 1
```

### Example 5: Validating Data Before Saving

```php
$data = input('website'); // might be invalid data
$processed = $repeater->processInput('website', $data);

$validation = $repeater->validate('website', $processed);
if (!$validation['valid']) {
    return response()->json(['errors' => $validation['errors']], 422);
}

// If valid, save
$department->website = $processed;
```

### Example 6: Running Tests to Ensure Class Integrity

Developers can run tests manually (e.g., in a debug script or a dedicated controller):

```php
Route::get('test-repeater', function() {
    $repeater = new RepeaterFieldsData();
    $report = $repeater->runTests();
    return response()->json($report);
});
```

This will output a report of all test cases and their results, helping to verify that the class works as expected after any modifications.

---

## Conclusion

The `RepeaterFieldsData` class provides a unified and flexible solution for handling repeatable fields in OctoberCMS, reducing complexity when building APIs or models that contain multiple pieces of data. Thanks to its YAML-based definitions and comprehensive methods, developers can rely on it fully in their projects without reinventing the wheel.

**Note:** Ensure that the YAML definition files exist at the specified paths (`$/tss/basic/classes/partials/`) before using the class, and adjust the paths if necessary.