# Documentation of `AvailableFilterManager` Class

## Introduction

The `AvailableFilterManager` class is a general-purpose class (not tied to a specific model) designed to uniformly manage and prepare filter data (available filters). The class performs the following tasks:

- Load filters from multiple sources: Array, JSON string, Config file.
- Normalize filter data: Fill missing fields with default values, and convert textual fields into multilingual formats.
- Validate filters and ensure they conform to the required schema.
- Merge multiple filter sets.
- Sort filters by `sort_order`.
- Filter only active filters (`is_active == true`).
- Find a filter by `id`.
- Reorder filters.
- Detect duplicates in `id` or `name`.
- Fetch options for filters from dynamic data sources (API, database, static, functions).
- Convert filters to JSON.
- Obtain the JSON schema of the filters.

---

## Basic Usage

### Creating an Instance of the Class
```php
use Nano\Tags\Classes\AvailableFilterManager;

$manager = new AvailableFilterManager();
```
You can pass an options array to the constructor to customize supported languages or the default config key:
```php
$manager = new AvailableFilterManager([
    'supportedLanguages' => ['ar', 'en'],      // Supported languages
    'defaultConfigKey'   => 'custom.config.key' // Custom config key
]);
```

### Quick Loading Using the Helper Function `load`
```php
$filters = AvailableFilterManager::load($source);
```
Where `$source` can be:
- A PHP array.
- A JSON string.
- A config key (string) such as `'nano.tags::available_filter.shop'`.

---

## Loading Methods

### 1. `fromArray(array $filters)`
**Description**: Load filters from a regular array.

**Input**:
- `$filters`: An array containing filter data (can be in the form `[['id'=>'x', ...], ...]` or `['available_filter' => [...]]`).

**Output**: An array of filters after applying normalization (filling missing fields, converting multilingual fields, etc.).

**Example**:
```php
$input = [
    ['id' => 'country_id', 'name' => 'country_id', 'label' => 'الدولة', 'type' => 'select']
];
$filters = $manager->fromArray($input);
// Result: a complete array containing all default fields (sort_order, placeholder, comment, ...)
```

### 2. `fromJson(string $json)`
**Description**: Load filters from a JSON string.

**Input**:
- `$json`: A JSON string representing the filters.

**Output**: An array of filters after normalization.

**Example**:
```php
$json = '[{"id":"country_id","name":"country_id","label":"الدولة","type":"select"}]';
$filters = $manager->fromJson($json);
```

### 3. `fromConfig(string $key = null, string $type = null)`
**Description**: Load filters from a Laravel config file.

**Input**:
- `$key`: The config key (if `null`, uses `defaultConfigKey`).
- `$type`: An additional part of the key (e.g., 'shop' or 'products').

**Output**: An array of filters after normalization.

**Example**:
```php
// Load filters from config/nano/tags/available_filter.php under the key 'shop'
$filters = $manager->fromConfig('nano.tags::available_filter', 'shop');
```

### 4. `static load($source, array $options = [])`
**Description**: A helper function to load filters from an unspecified source.

**Input**:
- `$source`: An array, JSON string, or config key.
- `$options`: An options array to initialize the class (optional).

**Output**: An array of filters after normalization.

**Example**:
```php
// From array
$filters1 = AvailableFilterManager::load($array);

// From JSON
$filters2 = AvailableFilterManager::load($jsonString);

// From config
$filters3 = AvailableFilterManager::load('nano.tags::available_filter.shop');
```

---

## Filter Processing Methods

### 5. `normalizeFilters($filters)`
**Description**: Normalize filters (convert them into an array of complete records). Used internally in loading methods.

**Input**:
- `$filters`: Filter data (array or anything else).

**Output**: An array of filters after creating each record using `createRecord()` and sorting.

### 6. `createRecord(array $data)`
**Description**: Create a complete filter record based on the input data, adding default values and performing validation.

**Input**:
- `$data`: An array containing some filter properties (e.g., `id`, `name`, `type`, ...).

**Output**: A complete array representing the filter after adding all default fields and applying validation.

**Example**:
```php
$record = $manager->createRecord(['id' => 'price', 'type' => 'number']);
/* Result:
[
    'id' => 'price',
    'name' => 'price',
    'type' => 'number',
    'label' => [],
    'placeholder' => null,
    'comment' => null,
    'commentPosition' => 'below',
    'commentHtml' => false,
    'default' => 0,               // Automatically added because type=number
    'dependsOn' => null,
    'trigger' => null,
    'is_active' => true,
    'showSearch' => false,
    'sort_order' => 0,
    'options' => null,
    'data_source' => null,
    ...
]
*/
```

### 7. `validateRecord(array $record, $throw = true)`
**Description**: Validate a single filter record.

**Input**:
- `$record`: An array representing the filter.
- `$throw`: If `true`, throws an exception on errors.

**Output**: An array `['valid' => bool, 'errors' => array]` (if `$throw = false`).

**Example**:
```php
$validation = $manager->validateRecord($record, false);
if (!$validation['valid']) {
    print_r($validation['errors']);
}
```

### 8. `validateAll(array $filters)`
**Description**: Validate an entire set of filters.

**Input**:
- `$filters`: An array of filters.

**Output**: An array `['valid' => bool, 'errors' => array]` where keys indicate the index of the invalid item.

**Example**:
```php
$result = $manager->validateAll($filters);
if (!$result['valid']) {
    foreach ($result['errors'] as $index => $errors) {
        echo "Error in filter $index: " . implode(', ', $errors);
    }
}
```

### 9. `sort(array $filters)`
**Description**: Sort filters in ascending order by `sort_order`.

**Input**:
- `$filters`: An array of filters.

**Output**: The sorted array of filters.

### 10. `merge(array $filters1, array $filters2, $override = true)`
**Description**: Merge two filter sets. If a filter with the same `id` exists in both sets, it is replaced by the value from `$filters2` if `$override = true`.

**Input**:
- `$filters1`, `$filters2`: Arrays of filters.
- `$override`: Whether to override existing filters.

**Output**: The merged array of filters (sorted).

**Example**:
```php
$merged = $manager->merge($defaultFilters, $userFilters);
```

### 11. `getActiveOnly(array $filters)`
**Description**: Return only active filters (where `is_active == true`).

**Input**:
- `$filters`: An array of filters.

**Output**: An array containing only active filters.

### 12. `findById(array $filters, $id)`
**Description**: Find a filter by `id`.

**Input**:
- `$filters`: An array of filters.
- `$id`: The `id` value to search for.

**Output**: The filter array if found, or `null` if not found.

### 13. `reorder(array $filters)`
**Description**: Reorder filters by assigning new ascending `sort_order` values (1,2,3,...).

**Input**:
- `$filters`: An array of filters.

**Output**: The modified array of filters.

### 14. `checkDuplicates(array $filters)`
**Description**: Detect duplicates in `id` or `name`.

**Input**:
- `$filters`: An array of filters.

**Output**: An array `['valid' => bool, 'duplicates' => ['id' => [...], 'name' => [...]]]`.

**Example**:
```php
$result = $manager->checkDuplicates($filters);
if (!$result['valid']) {
    echo "Duplicate id: " . implode(', ', $result['duplicates']['id']);
    echo "Duplicate name: " . implode(', ', $result['duplicates']['name']);
}
```

---

## Options and Data Source Methods

### 15. `getResolvedOptions(array $filter)`
**Description**: Get the resolved options for a filter. If the filter has `options`, they are returned; if it has `data_source`, the data is fetched from the source.

**Input**:
- `$filter`: An array representing the filter.

**Output**: An array of options (each option contains `id` and `name`).

**Example**:
```php
$options = $manager->getResolvedOptions($filter);
```

### 16. `enrichWithOptions(array $filter)`
**Description**: Enrich the filter by adding a `resolved_options` field containing the resolved options.

**Input**:
- `$filter`: The filter array.

**Output**: A copy of the filter with the new `resolved_options` field.

### 17. `fetchOptionsFromDataSource(array $dataSource)`
**Description**: Fetch options from a data source (used internally). Supports types: `api`, `database`, `static`, `function`.

**Input**:
- `$dataSource`: An array containing the details of the data source.

**Output**: An array of options.

**Note**: You must implement the HTTP request for the API yourself (not included in the class). The class currently returns an empty array and leaves expansion to you.

---

## Conversion and Documentation Methods

### 18. `toJson(array $filters, $pretty = false)`
**Description**: Convert filters to a JSON string.

**Input**:
- `$filters`: An array of filters.
- `$pretty`: If `true`, formats the JSON in a readable way.

**Output**: A JSON string.

**Example**:
```php
$json = $manager->toJson($filters, true);
```

### 19. `getSchema($format = 'array')`
**Description**: Get the JSON schema of the filters (reads from `available-filter-schema.json` if it exists, otherwise returns a default schema).

**Input**:
- `$format`: `'array'` to return an array, `'json'` to return a JSON string.

**Output**: An array or JSON string representing the schema.

---

## Comprehensive Example

Assume we have a simple array of filters and we want to load and prepare them for use.

```php
use Nano\Tags\Classes\AvailableFilterManager;

// Raw data from some source (could be from POST or database)
$input = [
    [
        'id' => 'country_id',
        'name' => 'country_id',
        'label' => 'الدولة',
        'type' => 'select',
        'data_source' => [
            'type' => 'api',
            'endpoint' => '/api/countries'
        ]
    ],
    [
        'id' => 'price',
        'name' => 'price',
        'label' => 'السعر',
        'type' => 'number',
        'sort_order' => 5
    ]
];

// Create the manager
$manager = new AvailableFilterManager();

// 1. Load and normalize filters
$filters = $manager->fromArray($input);

// 2. Validate
$validation = $manager->validateAll($filters);
if (!$validation['valid']) {
    // Handle errors
    exit;
}

// 3. Enrich filters with resolved options (optional)
$enriched = [];
foreach ($filters as $filter) {
    $enriched[] = $manager->enrichWithOptions($filter);
}

// 4. Convert to JSON for sending to the frontend or API
$jsonOutput = $manager->toJson($enriched, true);

echo $jsonOutput;
```

**Example Output** (abbreviated):
```json
[
    {
        "id": "country_id",
        "name": "country_id",
        "type": "select",
        "label": {
            "ar": "الدولة",
            "en": "الدولة",
            "fr": "الدولة"
        },
        "sort_order": 0,
        "data_source": {
            "type": "api",
            "endpoint": "/api/countries"
        },
        "resolved_options": []  // Actual options would be here if fetched from API
    },
    {
        "id": "price",
        "name": "price",
        "type": "number",
        "label": {
            "ar": "السعر",
            "en": "السعر",
            "fr": "السعر"
        },
        "sort_order": 5,
        "default": 0,
        "resolved_options": []
    }
]
```

---

## Conclusion

The `AvailableFilterManager` class provides a unified and powerful interface for handling filter data in Laravel applications. Using it, you can:

- Load filters from any source.
- Ensure data completeness and compliance with the required structure.
- Easily process filters (merge, sort, filter).
- Dynamically obtain their options.
- Convert them to JSON for use in APIs or frontend interfaces.

You can extend the class as needed, especially in methods for fetching options from APIs or databases.