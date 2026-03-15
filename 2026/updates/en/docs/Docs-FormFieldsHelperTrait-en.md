### **Additional Features (FormFieldsHelperTrait)**

A set of new helper functions has been added via the `FormFieldsHelperTrait` to extend the capabilities of the `FormFieldsManager` class. These functions cover multiple areas such as value validation, output generation, advanced filtering, and searching. Below is a detailed explanation of each group with examples.

---

#### 1. **Validation Simulation**

##### `validateFieldValue(array $field, $value)`
Simulates validation of a given value against the validation rules of a field. Returns an array containing `valid` (boolean) and `errors` (array of error messages).

**Example:**
```php
$field = [
    'validation' => [
        ['rule' => 'required'],
        ['rule' => 'email', 'messages' => ['ar' => 'The email is incorrect', 'en' => 'The email is incorrect']]
    ]
];
$result = $manager->validateFieldValue($field, 'test@example.com');
// $result: ['valid' => true, 'errors' => []]

$result = $manager->validateFieldValue($field, 'invalid-email');
// $result: ['valid' => false, 'errors' => ['The email is incorrect']]
```

##### `getValidationRulesFlattened(array $field)`
Converts a field's validation rules into Laravel's string format (e.g., `'required|email|min:5'`).

**Example:**
```php
$field = [
    'validation' => [
        ['rule' => 'required'],
        ['rule' => 'min', 'value' => 5],
        ['rule' => 'email']
    ]
];
$rules = $manager->getValidationRulesFlattened($field);
// $rules: 'required|min:5|email'
```

---

#### 2. **Output Generation**

##### `toHtml(array $field, $lang = null)`
Generates simple preliminary HTML code for the field. Takes into account the specified language for texts (e.g., `label`, `placeholder`).

**Example:**
```php
$field = [
    'id' => 'username',
    'name' => 'username',
    'type' => 'text',
    'label' => ['ar' => 'Username', 'en' => 'Username'],
    'placeholder' => ['ar' => 'Enter username', 'en' => 'Enter username'],
    'required' => true
];
echo $manager->toHtml($field, 'en');
// Output: <div class="form-group"><label for="username">Username</label><input type="text" id="username" name="username" value="" placeholder="Enter username" required ></div>
```

##### `toJsonSchema(array $fields, array $options = [])`
Converts a set of fields to a custom JSON Schema structure, with the possibility to add a title and general description.

**Example:**
```php
$schema = $manager->toJsonSchema($fields, [
    'title' => 'Registration Form',
    'description' => 'Registration form fields'
]);
// Result: an array representing JSON Schema
```

---

#### 3. **Filtering by Specific Properties**

##### `getFieldsByDataSourceType(array $fields, $type)`
Retrieves fields that use a specific data source (`api`, `database`, `static`, `function`).

**Example:**
```php
$apiFields = $manager->getFieldsByDataSourceType($fields, 'api');
```

##### `getFieldsWithValidation(array $fields)`, `getFieldsWithoutValidation(array $fields)`
Fields that have / do not have validation rules.

##### `getFieldsByReadOnly(array $fields, $readOnly = true)`, `getFieldsByDisabled`, `getFieldsByHidden`
Fields based on `readOnly`, `disabled`, `hidden` status.

##### `getFieldsByTabGroup(array $fields)`
Groups fields into an associative array where the key is the tab name.

**Example:**
```php
$grouped = $manager->getFieldsByTabGroup($fields);
foreach ($grouped as $tab => $tabFields) {
    echo "Tab: $tab";
    // Display fields
}
```

##### `groupFieldsByType(array $fields)`
Groups fields by their `type`.

##### `getFieldsWithChangeHandler(array $fields)`
Fields that contain a `changeHandler`.

##### `getFieldsWithOptions(array $fields)`, `getFieldsWithoutOptions(array $fields)`
Retrieves list-type fields (select, dropdown, ...) that have options (static or dynamic) or those without options.

---

#### 4. **Search and Query**

##### `hasFieldWithId(array $fields, $id)`, `hasFieldWithName(array $fields, $name)`
Checks if a field exists with a given ID or name.

##### `findByName(array $fields, $name)`
Searches for a field by `name`.

##### `findBy(array $fields, $key, $value)`
General search for a field by a given key and value.

**Example:**
```php
$field = $manager->findBy($fields, 'type', 'email');
```

##### `filterByCallback(array $fields, callable $callback)`
Custom filtering using a callback.

**Example:**
```php
$filtered = $manager->filterByCallback($fields, function($field) {
    return strpos($field['id'], 'address') !== false;
});
```

---

#### 5. **Data Transformation and Processing**

##### `extractFieldValues(array $fields, $key)`
Extracts values of a specific key from all fields (e.g., extracting every `label`).

**Example:**
```php
$labels = $manager->extractFieldValues($fields, 'label');
```

##### `mapFieldsToAssoc(array $fields, $key = 'id')`
Converts the fields array into an associative array using `id` or `name` as the key.

**Example:**
```php
$assoc = $manager->mapFieldsToAssoc($fields, 'name');
$field = $assoc['email']; // Direct access
```

##### `sortFieldsBy(array $fields, $key, $direction = 'asc')`
Sorts fields by any key (e.g., `id`, `name`, `sort_order`).

##### `sliceFields(array $fields, $offset, $length = null)`
Extracts a slice of fields.

##### `paginateFields(array $fields, $perPage = 10, $page = 1)`
Paginates fields with metadata.

**Example:**
```php
$page2 = $manager->paginateFields($fields, 5, 2);
// Contains: data, total, per_page, current_page, last_page
```

##### `localizeField(array &$field, $lang)`
Converts all multilingual fields in a single field record to simple text in a specified language (modifies by reference).

##### `localizeFields(array &$fields, $lang)`
Applies `localizeField` to all fields.

##### `stripMultilingualFields(array &$fields, $lang)`
Synonym for `localizeFields`, removes the multilingual structure.

##### `toArraySimple(array $fields, $lang = null)`
Converts fields to a simplified format (without language structure) suitable for APIs.

**Example:**
```php
$simple = $manager->toArraySimple($fields, 'en');
// All fields now contain direct English texts
```

---

### **Conclusion**

These additions represent a qualitative leap in the flexibility of the `FormFieldsManager` class, enabling developers to handle form fields in more advanced ways: automatic validation, output generation, intelligent filtering, and various transformations. It is recommended to use these functions to reduce redundancy and increase maintainability in projects that rely heavily on form fields.