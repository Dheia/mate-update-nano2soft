# FormFieldsManager Class Documentation

## Introduction

The `FormFieldsManager` class is a generic class (not tied to a specific model) designed to uniformly and independently manage and prepare form fields data. The class performs the following tasks:

- Load fields from multiple sources: Array, JSON string, Config file.
- Normalize field data: Fill missing fields with default values, and convert text fields into multilingual formats.
- Validate fields and ensure they match the required schema.
- Merge multiple field sets.
- Sort fields by `sort_order`.
- Filter only active fields (`is_active == true`).
- Find a field by `id`.
- Reorder fields.
- Detect duplicates in `id` or `name`.
- Fetch options for fields from dynamic data sources (API, database, static, functions).
- Convert fields to JSON.
- Obtain the JSON schema for fields.

This class is built to be flexible and extensible, and can be used in any part of an OctoberCMS or Laravel application such as APIs, console commands, or even inside models with minor adjustments.

---

## Properties

The class contains the following properties (can be modified via the constructor):

| Property | Type | Description | Default Value |
|----------|------|-------------|---------------|
| `$supportedLanguages` | `array` | Supported languages for multilingual fields. | `['ar', 'en', 'fr']` |
| `$defaultConfigKey` | `string` | Default key in the config file to load fields. | `'nano.tags::form_fields'` |
| `$defaultRecord` | `array` | Default structure for a field record (populated automatically). | (set in `getDefaultRecordStructure`) |

---

## Loading Methods

### 1. `__construct(array $options = [])`
**Description**: Class constructor, allows customizing supported languages and default config key.

**Input**:
- `$options`: Optional array containing:
  - `supportedLanguages`: Array of required languages.
  - `defaultConfigKey`: Default config key.

**Example**:
```php
$manager = new FormFieldsManager([
    'supportedLanguages' => ['ar', 'en'],
    'defaultConfigKey'   => 'myplugin::custom_fields'
]);
```

---

### 2. `static load($source, array $options = [])`
**Description**: Helper method to load fields from an unspecified source. It creates a temporary instance of the class and loads the data.

**Input**:
- `$source`: Data source, can be:
  - `array`: PHP array.
  - `string`: JSON string or config key (if the string starts with `{` or `[` it is considered JSON, otherwise a config key).
- `$options`: Options to pass to the constructor (optional).

**Output**: Array of fields after normalization.

**Example**:
```php
// From array
$fields = FormFieldsManager::load($array);

// From JSON
$json = '[{"id":"title","label":"العنوان","type":"text"}]';
$fields = FormFieldsManager::load($json);

// From config
$fields = FormFieldsManager::load('nano.tags::form_fields.shop');
```

---

### 3. `fromArray(array $fields)`
**Description**: Load fields from a plain PHP array.

**Input**:
- `$fields`: Array containing field data. Can be:
  - An array of records like `[['id'=>'x', ...], ...]`.
  - An array containing a `'form_fields'` key with the previous array as its value.

**Output**: Array of fields after normalization and sorting.

**Example**:
```php
$input = [
    ['id' => 'title', 'name' => 'title', 'label' => 'العنوان', 'type' => 'text'],
    ['id' => 'content', 'name' => 'content', 'label' => 'المحتوى', 'type' => 'textarea']
];
$fields = $manager->fromArray($input);
```

---

### 4. `fromJson(string $json)`
**Description**: Load fields from a JSON string.

**Input**:
- `$json`: Valid JSON string.

**Output**: Array of fields after normalization.

**Throws**: Exception if JSON is invalid.

**Example**:
```php
$json = '[{"id":"title","label":"العنوان","type":"text"}]';
$fields = $manager->fromJson($json);
```

---

### 5. `fromConfig(string $key = null, string $type = null)`
**Description**: Load fields from a Laravel config file.

**Input**:
- `$key`: Config key (if `null`, uses `$defaultConfigKey`).
- `$type`: Additional part of the key (e.g., `'shop'` or `'products'`) to form the full key `key.type`.

**Output**: Array of fields after normalization.

**Example**:
```php
// Load from config/nano/tags/form_fields.php
$fields = $manager->fromConfig();

// Load from config/nano/tags/form_fields.php under the 'shop' key
$fields = $manager->fromConfig('nano.tags::form_fields', 'shop');
```

---

## Record Creation Methods

### 6. `normalizeFields($fields)`
**Description**: Internal method used to normalize fields. It converts the input into an array of complete records using `createRecord`, and sets `sort_order` if not present.

**Input**:
- `$fields`: Field data (could be an array or anything else).

**Output**: Sorted array of complete records.

---

### 7. `createRecord(array $data)`
**Description**: Creates a complete field record based on the input data. The data is merged with the default structure, then required and type-specific values are applied, values are cleaned, and finally validated.

**Input**:
- `$data`: Array containing some field properties (e.g., `id`, `name`, `type`, `label`, ...).

**Output**: Complete array representing the field.

**Throws**: `ApplicationException` if validation fails.

**Example**:
```php
$record = $manager->createRecord([
    'id' => 'price',
    'type' => 'number',
    'label' => 'السعر'
]);
/* Result contains:
    'id' => 'price',
    'name' => 'price', (automatically set)
    'type' => 'number',
    'label' => ['ar' => 'السعر', 'en' => 'السعر', 'fr' => 'السعر'],
    'default' => 0, (specific to number type)
    ...
*/
```

---

### 8. `applyRequired(array &$record)`
**Description**: Internal method to apply required values and handle multilingual fields.

- Sets `id` if empty.
- Sets `name` from `id` if empty.
- Converts fields (`label`, `placeholder`, `comment`, `emptyOption`) to a language array if they are not already.
- Ensures `sort_order` is numeric.

---

### 9. `applyTypeSpecific(array &$record)`
**Description**: Internal method that adds default values specific to the field type (e.g., `placeholder` for texts, `options` for lists, `default` for numbers, etc.).

**Note**: This method covers all supported field types and sets the appropriate properties.

---

### 10. `cleanValues(array &$record)`
**Description**: Internal method to clean unnecessary values:
- Converts empty values to `null` except for some fields.
- Converts `validation` from a JSON string to an array if needed.
- Converts `attributes` and `containerAttributes` to `null` if they are empty arrays.

---

## Validation Methods

### 11. `validateRecord(array $record, $throw = true)`
**Description**: Validates a single field record.

**Input**:
- `$record`: Field array.
- `$throw`: If `true`, throws an exception on errors; otherwise returns a result array.

**Output**: Array `['valid' => bool, 'errors' => array]` (if `$throw = false`).

**Example**:
```php
$validation = $manager->validateRecord($record, false);
if (!$validation['valid']) {
    foreach ($validation['errors'] as $error) {
        echo $error;
    }
}
```

---

### 12. `getSupportedFieldTypes()`
**Description**: Internal method that returns a list of all supported field types (according to the schema).

**Output**: `array` of names.

---

### 13. `validateAll(array $fields)`
**Description**: Validates a complete set of fields.

**Input**:
- `$fields`: Array of fields.

**Output**: Array `['valid' => bool, 'errors' => array]` where each error key is the index of the invalid item.

**Example**:
```php
$result = $manager->validateAll($fields);
if (!$result['valid']) {
    foreach ($result['errors'] as $index => $errors) {
        echo "Error in field $index: " . implode(', ', $errors);
    }
}
```

---

## Utility Methods

### 14. `sort(array $fields)`
**Description**: Sorts fields ascending by `sort_order` (default `9999` if not present).

**Input**:
- `$fields`: Array of fields.

**Output**: Sorted array of fields.

---

### 15. `merge(array $fields1, array $fields2, $override = true)`
**Description**: Merges two field sets. Merging is based on each field's `id`. If a field with the same `id` exists in both sets, its value is replaced with that from `$fields2` if `$override = true`; otherwise, the original value is kept.

**Input**:
- `$fields1`, `$fields2`: Field arrays.
- `$override`: Override existing fields.

**Output**: Merged array of fields (sorted).

**Example**:
```php
$merged = $manager->merge($defaultFields, $userFields, true);
```

---

### 16. `getActiveOnly(array $fields)`
**Description**: Returns only active fields where `is_active == true`.

**Input**:
- `$fields`: Array of fields.

**Output**: Array containing active fields.

---

### 17. `findById(array $fields, $id)`
**Description**: Finds a field by `id`.

**Input**:
- `$fields`: Array of fields.
- `$id`: Desired `id` value.

**Output**: Field array if found, or `null` if not found.

---

### 18. `reorder(array $fields)`
**Description**: Reorders fields by assigning a new ascending `sort_order` (1, 2, 3, ...).

**Input**:
- `$fields`: Array of fields.

**Output**: Array of fields after modification (modifies the original array).

---

### 19. `checkDuplicates(array $fields)`
**Description**: Detects duplicates in `id` or `name` among fields.

**Input**:
- `$fields`: Array of fields.

**Output**: Array `['valid' => bool, 'duplicates' => ['id' => [...], 'name' => [...]]]`.

**Example**:
```php
$result = $manager->checkDuplicates($fields);
if (!$result['valid']) {
    echo "Duplicate id: " . implode(', ', $result['duplicates']['id']);
    echo "Duplicate name: " . implode(', ', $result['duplicates']['name']);
}
```

---

## Options & Data Sources Methods

### 20. `getResolvedOptions(array $field, $isResolvedField = true)`
**Description**: Retrieves resolved options for a field. If the field contains `options`, they are returned directly. If it contains `data_source`, the data is fetched from the source using `fetchOptionsFromDataSource`.

**Input**:
- `$field`: Field array.
- `$isResolvedField`: If `true` (default), converts data coming from the source into the format `['id' => ..., 'name' => ...]`; otherwise returns raw data.

**Output**: Array of options (may be empty).

**Example**:
```php
$options = $manager->getResolvedOptions($field);
```

---

### 21. `enrichWithOptions(array $field, $isResolvedField = true)`
**Description**: Enriches the field by adding a new field `resolved_options` containing the resolved options.

**Input**:
- `$field`: Field array.
- `$isResolvedField`: Same meaning as above.

**Output**: Copy of the field with the additional field.

**Example**:
```php
$enrichedField = $manager->enrichWithOptions($field);
```

---

### 22. `fetchOptionsFromDataSource(array $dataSource, $isResolvedField = false)`
**Description**: Internal method to fetch options from a data source. Supports the following types:

- **`api`**: Performs an HTTP GET request to `endpoint` with `params`, and supports converting relative URLs to absolute using `url()`. Attempts to use `Illuminate\Support\Facades\Http` if available, or `October\Rain\Network\Http` as a fallback.
- **`database`**: Queries an Eloquent model with optional `conditions` and `order_by`.
- **`static`**: Returns `options` directly.
- **`function`**: Calls a function (or a method inside a class) with `params`.

**Input**:
- `$dataSource`: Array containing `type` and the required data according to the type.
- `$isResolvedField`: If `true`, converts API data to the format `['id', 'name']` using `value_field` and `label_field`.

**Output**: Array of options.

---

## Conversion & Schema Methods

### 23. `toJson(array $fields, $pretty = false)`
**Description**: Converts the fields array to a JSON string.

**Input**:
- `$fields`: Array of fields.
- `$pretty`: If `true`, formats JSON in a readable way.

**Output**: JSON string.

**Example**:
```php
$json = $manager->toJson($fields, true);
echo $json;
```

---

### 24. `getSchema($format = 'array')`
**Description**: Retrieves the JSON schema for fields. Attempts to read the file `form-fields-settings-schema-no-default.json` from the path `plugins/nano/tags/models/type/`. If not found, returns a simplified default schema.

**Input**:
- `$format`: `'array'` to return an array, `'json'` to return a JSON string.

**Output**: Array or JSON string.

**Example**:
```php
$schemaArray = $manager->getSchema('array');
$schemaJson = $manager->getSchema('json');
```

---

### 25. `deepMerge(array $array1, array $array2)`
**Description**: Internal method for deep merging two arrays (used internally).

---

### 26. `getDefaultRecordStructure()`
**Description**: Internal method that returns the complete default structure for a field record, containing all possible properties with their default values (as described in the schema).

---

## Comprehensive Example

Suppose we want to build an API that accepts form field data from the user, normalizes and validates it, and then returns it with resolved options.

```php
use Nano\Tags\Classes\FormFieldsManager;

// Data sent from the client (could be JSON or array)
$input = [
    [
        'id' => 'title',
        'label' => 'العنوان',
        'type' => 'text',
        'required' => true,
        'sort_order' => 1
    ],
    [
        'id' => 'country',
        'label' => 'الدولة',
        'type' => 'select',
        'data_source' => [
            'type' => 'api',
            'endpoint' => '/api/countries',
            'value_field' => 'code',
            'label_field' => 'name_ar'
        ],
        'sort_order' => 2
    ]
];

// 1. Create the manager
$manager = new FormFieldsManager();

// 2. Load and normalize fields
$fields = $manager->fromArray($input);

// 3. Validate
$validation = $manager->validateAll($fields);
if (!$validation['valid']) {
    return response()->json([
        'error' => 'Invalid fields data',
        'details' => $validation['errors']
    ], 422);
}

// 4. Enrich fields with options (to send to the frontend)
$enriched = [];
foreach ($fields as $field) {
    $enriched[] = $manager->enrichWithOptions($field);
}

// 5. Convert to JSON
$output = $manager->toJson($enriched, true);

// 6. Return response
return response($output, 200)->header('Content-Type', 'application/json');
```

**Expected Output (JSON)**:
```json
[
    {
        "id": "title",
        "name": "title",
        "type": "text",
        "label": {
            "ar": "العنوان",
            "en": "العنوان",
            "fr": "العنوان"
        },
        "required": true,
        "sort_order": 1,
        "placeholder": {
            "ar": "أدخل العنوان",
            "en": "Enter العنوان",
            "fr": "Entrez العنوان"
        },
        "resolved_options": []
    },
    {
        "id": "country",
        "name": "country",
        "type": "select",
        "label": {
            "ar": "الدولة",
            "en": "الدولة",
            "fr": "الدولة"
        },
        "data_source": {
            "type": "api",
            "endpoint": "/api/countries",
            "value_field": "code",
            "label_field": "name_ar"
        },
        "sort_order": 2,
        "emptyOption": {
            "ar": "اختر",
            "en": "Select",
            "fr": "Sélectionner"
        },
        "showSearch": true,
        "resolved_options": [
            { "id": "SA", "name": "السعودية" },
            { "id": "EG", "name": "مصر" },
            ...
        ]
    }
]
```

---

## Conclusion

The `FormFieldsManager` class provides a comprehensive and powerful interface for handling form fields data in OctoberCMS and Laravel applications. Using it you can:

- Unify the way fields are loaded from multiple sources.
- Ensure data completeness and validity.
- Easily process fields (merge, sort, filter).
- Dynamically obtain options from APIs or databases.
- Convert fields to JSON for use in APIs or storage.