# Documentation for `Nano2\QueryBuilder\FormWidgets\QueryBuilder`

**NanoSoft Software Module – Visual Query Builder Tool**

---

## 1. Overview

`QueryBuilder` is an advanced FormWidget that allows creating complex query conditions visually and easily, without the need to write SQL or manual programming logic. This widget relies on the `jQuery QueryBuilder` library on the frontend, stores rules as JSON in the database, and allows them to be applied directly to data queries (Models) via the `QueryBuilderParserHelper` class.

This widget is used for:
- Building dynamic reports with multi-level conditions (AND / OR).
- Creating advanced search screens.
- Storing reusable filters by users.
- Supporting various input types (text, numbers, dates, dropdowns, checkboxes, etc.).

---

## 2. Installation and Basic Setup

### 2.1. Include the Module
Make sure the `Nano2.QueryBuilder` module is installed in your NanoSoft project.

### 2.2. Add the Field in the Model's YAML File
```yaml
# models/my_model/fields.yaml
fields:
    report_conditions:
        label: Report Conditions
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        span: full
```

### 2.3. Database Column
The associated column must be of type `text` or `longtext` to store the resulting JSON:
```php
// In the model definition file
public $jsonable = ['report_conditions'];
```

---

## 3. Properties and Configuration

You can customize the widget's behavior by writing properties in the YAML file or setting them directly in the class.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `filters` | array / string | `[]` | Definition of available filters (detailed later). Can be an array or a model method name. |
| `rules` | array | `[]` | Default rules displayed when the interface loads (as JSON or array). |
| `sort_filters` | bool | `false` | Sort filters alphabetically by translation. |
| `allow_groups` | int / bool | `-1` | Maximum number of nested groups (`-1` = unlimited, `0` = no groups, `true` = -1, `false` = 0). |
| `allow_empty` | bool | `false` | Allow empty rules (no condition). |
| `display_errors` | bool | `true` | Show error messages next to rules (e.g., required fields or incorrect format). |
| `conditions` | array / string | `['AND', 'OR']` | List of allowed logical connectors. Can send a single string `'AND'` or `'OR'`. |
| `default_condition` | string | `'AND'` | Default connector for the root group. |
| `inputs_separator` | string | `' , '` | Separator between multiple input fields (e.g., BETWEEN). |
| `display_empty_filter` | bool | `true` | Show an empty option (`------`) in the filters list. |
| `select_placeholder` | string | `'------'` | Text shown when no filter is selected. |
| `default_filter` | string | `null` | Filter ID automatically selected when adding a new rule. |
| `plugins` | array | `[]` | List of enabled plugins (see Plugins section). |
| `isShowParseSqlBtn` | bool | `false` | Show a button to translate rules to SQL (for debugging). |
| `isShowResetBtn` | bool | `true` | Show a button to reset rules. |
| `isShowSetBtn` | bool | `true` | Show a button to load default rules. |
| `isShowGetBtn` | bool | `true` | Show a button to apply rules and save them to the hidden field. |
| `isAllowNotfiy` | bool | `true` | Show a notification when rules are successfully applied or fail. |

**Advanced YAML Configuration Example**:
```yaml
report_conditions:
    type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
    label: Search Conditions
    sort_filters: true
    allow_groups: 3
    allow_empty: false
    conditions: ['AND', 'OR']
    default_condition: 'AND'
    inputs_separator: ' | '
    display_empty_filter: false
    select_placeholder: 'Select field'
    default_filter: 'name'
    plugins:
        sortable: null
        filter-description: { mode: 'popover' }
    isShowParseSqlBtn: true
```

---

## 4. Defining Filters

Filters are the building blocks that allow users to build conditions. Each filter specifies a data field, its type, input method, and allowed operators.

### 4.1. Filter Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Unique filter identifier (used in rules). |
| `label` | string / object | Yes | Display text. Supports translation as array `{'en':'Name', 'ar':'الاسم'}`. |
| `type` | string | Yes | Data type: `string`, `integer`, `double`, `date`, `time`, `datetime`, `boolean`. |
| `input` | string | No | Input element type: `text`, `number`, `textarea`, `radio`, `checkbox`, `select`. Default depends on type. |
| `field` | string | No | Actual database column name (if different from `id`). |
| `operators` | array | No | List of allowed operators for this filter (see table below). If not specified, all suitable operators for the type are used. |
| `default_operator` | string | No | Default operator (must be within `operators`). |
| `values` | array / string | Conditional | Required for `radio`, `checkbox`, `select`. Array of options or a method name to fetch dynamically. |
| `validation` | object | No | Validation rules: `min`, `max`, `step`, `format`, `allow_empty_value`, `messages`. |
| `placeholder` | string | No | Placeholder text inside the field (for `text`, `textarea`, `number`). |
| `size` | int | No | Field width (for `text`, `textarea`). |
| `rows` | int | No | Field height (for `textarea`). |
| `multiple` | bool | No | Allow multiple selection (for `select`). |
| `vertical` | bool | No | Display options vertically (for `radio`, `checkbox`). |
| `plugin` | string | No | jQuery plugin name (e.g., `datepicker`, `selectize`). |
| `plugin_config` | object | No | Plugin settings. |
| `default_value` | mixed | No | Default value automatically filled when creating a new rule. |
| `value_separator` | string | No | Separator used to convert multiple values into a string (for `in`, `not_in` operators). |
| `icon` | string | No | Icon (Bootstrap Icons class) shown next to the filter in the list. |
| `optgroup` | string | No | Group name to organize the dropdown list. |
| `description` | string / function | No | Descriptive text shown via the `filter-description` plugin. |
| `data` | object | No | Additional custom data added to the rule object on export. |

### 4.2. Supported Operators

| Operator | Description | Applies to Types |
|----------|-------------|-------------------|
| `equal` | Equals | string, number, datetime, boolean |
| `not_equal` | Not equals | string, number, datetime, boolean |
| `in` | In a set | string, number, datetime |
| `not_in` | Not in a set | string, number, datetime |
| `less` | Less than | number, datetime |
| `less_or_equal` | Less than or equal | number, datetime |
| `greater` | Greater than | number, datetime |
| `greater_or_equal` | Greater than or equal | number, datetime |
| `between` | Between (two values) | number, datetime |
| `not_between` | Not between | number, datetime |
| `begins_with` | Begins with | string |
| `not_begins_with` | Does not begin with | string |
| `contains` | Contains | string |
| `not_contains` | Does not contain | string |
| `ends_with` | Ends with | string |
| `not_ends_with` | Does not end with | string |
| `is_empty` | Empty (equals `''`) | string |
| `is_not_empty` | Not empty | string |
| `is_null` | Equals NULL | all types |
| `is_not_null` | Does not equal NULL | all types |

### 4.3. Examples of Filter Definitions

#### Example 1: Simple Text Filter
```yaml
name:
    id: name
    label: Name
    type: string
    operators: ['equal', 'contains', 'begins_with']
    placeholder: Enter name
```

#### Example 2: Numeric Filter with Validation
```yaml
price:
    id: price
    label: Price
    type: double
    validation:
        min: 0
        step: 0.01
    default_value: 0
```

#### Example 3: Dropdown Filter
```yaml
category:
    id: category_id
    label: Category
    type: integer
    input: select
    values:
        1: Books
        2: Movies
        3: Music
    operators: ['equal', 'not_equal', 'in']
    default_operator: 'in'
```

#### Example 4: Date Filter with Datepicker Plugin
```yaml
created_at:
    id: created_at
    label: Created At
    type: date
    input: text
    plugin: datepicker
    plugin_config:
        format: 'yyyy-mm-dd'
    validation:
        format: 'yyyy-mm-dd'
```

#### Example 5: Multiple Option Filter (Checkbox)
```yaml
status:
    id: status
    label: Status
    type: integer
    input: checkbox
    values:
        1: Active
        0: Inactive
    vertical: true
    operators: ['equal']
```

#### Example 6: Filter Using a Model Method for Options
```yaml
user:
    id: user_id
    label: User
    type: integer
    input: select
    values: getUsersOptions   # method name in the model
```

---

## 5. Defining Default Rules

You can set initial rules that appear when the interface loads via the `rules` property. The syntax is as follows:

```yaml
rules:
    condition: AND
    rules:
        - id: price
          operator: greater
          value: 100
        - condition: OR
          rules:
              - id: category
                operator: equal
                value: 1
              - id: category
                operator: equal
                value: 2
```

**Note**: You can also pass rules as a PHP array in the YAML file (it will be automatically converted to JSON).

---

## 6. Plugins

Plugins extend the interface functionality and add interactive features. They are enabled via the `plugins` property as an array.

### 6.1. List of Supported Plugins

| Plugin Name | Description | Options |
|-------------|-------------|---------|
| `sortable` | Enable drag-and-drop to reorder rules and groups. | `icon` (e.g., `bi-sort-down`), `inherit_no_sortable`, `inherit_no_drop` |
| `filter-description` | Show a descriptive tooltip for the filter. | `mode` (`inline`, `popover`, `bootbox`), `icon` |
| `bt-tooltip-errors` | Display validation errors as Bootstrap tooltips. | `placement` |
| `bt-checkbox` | Style `checkbox` and `radio` elements using Bootstrap. | `color` |
| `unique-filter` | Prevent using the same filter more than once (globally or within a group). | – |
| `not-group` | Add a "NOT" button to invert group logic. | `icon_checked`, `icon_unchecked` |
| `invert` | Button to invert conditions (convert `=` to `!=`, `AND` to `OR`). | `recursive`, `invert_rules`, `display_rules_button`, `icon` |

### 6.2. Example of Enabling Plugins
```yaml
plugins:
    sortable: { icon: 'bi-arrows-move' }
    filter-description: { mode: 'popover', icon: 'bi-question-circle' }
    bt-tooltip-errors: { placement: 'top' }
    unique-filter: null
    not-group: { icon_unchecked: 'bi-square', icon_checked: 'bi-check-square' }
```

---

## 7. Fetching Dynamic Options from the Model

To provide filters with dynamic options (e.g., list of users, departments, products), you can define a method in your model. There are two ways:

### 7.1. General Method for All QueryBuilder Fields
```php
public function getQueryBuilderFiltersOptions($columnName)
{
    if ($columnName == 'filters') {
        return [
            [
                'id' => 'user_id',
                'label' => 'User',
                'type' => 'integer',
                'input' => 'select',
                'values' => \App\Models\User::pluck('name', 'id')->toArray()
            ],
            // ...
        ];
    }
    return [];
}
```

### 7.2. Field-Specific Method
If the QueryBuilder field name is `report_conditions`, the method name would be:
```php
public function getReportConditionsQueryBuilderFiltersOptions($fieldName, $columnName)
{
    // $fieldName = 'report_conditions', $columnName = 'filters' or 'rules'
    if ($columnName == 'filters') {
        return [...];
    }
}
```

**Note**: The method can also return default rules by checking `$columnName == 'rules'`.

---

## 8. Complete Practical Examples

### 8.1. Product Model with Dynamic Category Filter

**fields.yaml**:
```yaml
fields:
    search_rules:
        label: Advanced Search Conditions
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        sort_filters: true
        allow_groups: 2
        filters:
            name:
                id: name
                label: Product Name
                type: string
                operators: ['contains', 'begins_with', 'equal']
            price:
                id: price
                label: Price
                type: double
                validation: { min: 0, step: 0.01 }
            category_id:
                id: category_id
                label: Category
                type: integer
                input: select
                values: getCategoryOptions   # method in the model
                operators: ['equal', 'in']
        rules:
            condition: AND
            rules:
                - id: price
                  operator: greater
                  value: 10
```

**In the model (Product.php)**:
```php
public function getCategoryOptions()
{
    return Category::pluck('name', 'id')->toArray();
}
```

### 8.2. Applying Conditions to a Query Using QueryBuilderParserHelper

After storing the rules in the database (as JSON), you can use them to filter results:

```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

// $product = Product::find($id);
$rules = $product->search_rules; // array or JSON string

if (!empty($rules)) {
    $query = Product::query();
    $filteredQuery = QueryBuilderParserHelper::parse($query, $rules);
    $results = $filteredQuery->get();
} else {
    $results = Product::all();
}
```

### 8.3. Complete Controller Example

```php
public function filterProducts(Request $request)
{
    $rules = $request->input('rules'); // JSON from the frontend
    $query = Product::query();
    
    if ($rules) {
        $query = QueryBuilderParserHelper::parse($query, json_decode($rules, true));
    }
    
    $products = $query->paginate(20);
    return view('products.index', compact('products'));
}
```

---

## 9. Handling Stored Values

When the form is saved, `QueryBuilder` stores the rules as JSON in the associated column. You can access them as an array by making the column `jsonable` in the model:

```php
class Report extends Model
{
    protected $jsonable = ['query_conditions'];
}
```

Then use it as:
```php
$report = Report::find(1);
$rules = $report->query_conditions; // ready-to-use PHP array
```

If you don't use `$jsonable`, you can manually decode:
```php
$rules = json_decode($report->query_conditions, true);
```

---

## 10. Frequently Asked Questions

### Q1: How can I translate filter labels and operators?
You can use the array syntax in the `label` property:
```yaml
label:
    en: 'Product Name'
    ar: 'اسم المنتج'
```
You can also use translation keys in the NanoSoft system (via `Lang::get`).

### Q2: How do I prevent users from adding a specific filter more than once?
Use the `unique-filter` plugin and set the filter as unique:
```yaml
filters:
    category:
        id: category
        label: Category
        unique: true   # or unique: 'group' to be unique only within the group
```

### Q3: How do I validate values before applying?
You can rely on the `validation` property in the filter, or perform manual validation before calling `QueryBuilderParserHelper`:
```php
$builder = QueryBuilderParserHelper::parse($query, $rules);
// $builder will execute the query only if values are valid, otherwise throw an exception.
```

### Q4: Can this widget be used outside forms (e.g., in a standalone controller interface)?
Yes, you can create the field in any template using `FormWidget` directly, providing the necessary data. Refer to NanoSoft documentation on using `FormWidget` outside the model context.

### Q5: How do I retrieve rules from the database and display them again in the interface?
When the form loads, the `FormWidget` will automatically read the stored value and rebuild the interface with the same rules. No additional code is needed.

---

## 11. Conclusion

`Nano2\QueryBuilder\FormWidgets\QueryBuilder` is a powerful and flexible tool that allows developers and end users to build complex queries easily. Thanks to its support for multiple input types, plugins, and seamless integration with `QueryBuilderParserHelper`, you can create advanced reporting and search systems in record time. Use this documentation as a comprehensive reference to explore all the widget's capabilities and start applying it in your NanoSoft projects.

---

## Additional Documentation

- [`QueryBuilderParserHelper` Class Documentation](./Docs-QueryBuilderParserHelper-en.md)

