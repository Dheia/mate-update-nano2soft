## 2026-03-11 – 2026-04-09

### Adding Visual Query Builder System and Advanced Condition Processor  
Within the `Nano2.QueryBuilder` software module

---

### 1. Introduction

A **Visual Query Builder** system has been developed to provide an interactive, easy-to-use interface that allows end users and developers to create complex query conditions without writing SQL or manual programming logic. The system relies on the `jQuery QueryBuilder` library on the frontend, and provides an integrated backend within NanoSoft software through a custom `FormWidget` and a helper class to convert conditions into executable queries on Data Models.

This update aims to enable use cases such as:
- Building dynamic reports with multi-level conditions (AND/OR) and various input types (text, numbers, dates, dropdowns, checkboxes, etc.).
- Applying conditions directly to a `Model` or `Builder` via the `QueryBuilderParserHelper` class.
- Reusing filter and operation definitions via YAML configuration or model methods.
- Supporting plugins such as drag-and-drop sorting, filter descriptions, custom icons, and more.

---

### 2. Developed Components

#### 2.1 `QueryBuilderParserHelper` – Condition Processing and Query Conversion Class  
- **Path**: `Nano2\QueryBuilder\Classes\QueryBuilderParserHelper`
- **Function**: A helper class that parses the condition object (rules) coming from the FormWidget, applies them to a Builder query, supporting macros, nested tables, and multiple operators.

**Key Methods**:

| Method | Description |
|--------|-------------|
| `parse($builder, $rules, $rows, $is_exception)` | Main method: receives `$builder` (model, model class name, or Builder object) and `$rules` array (usually from QueryBuilder's `getRules()`), returns a `Builder` with conditions applied. |
| `setFields($rules)` | Extracts field names used in the rules to pass to the `timgws\QueryBuilderParser` library (necessary to specify allowed columns). |
| `decodeJSON($json)` | Converts a JSON string to an object to validate rules. |
| `isNested($rule)` | Checks if a rule contains a sub-group of rules (for handling nested conditions). |
| `getQueryOrModel($obj, $is_model, $is_exception)` | Accepts an object that may be a `Model`, class name, or `Builder`, and returns either a `Builder` object or the model itself as needed. |
| `toSql($builder, $expand)` | (Taken from LibreNMS) Converts conditions to an SQL string for debugging or display. |

**Workflow**:
1. The `$rules` object (usually an array or JSON) is converted to a stdClass object using `decodeJSON`.
2. Fields used are extracted via `setFields`.
3. An instance of the `timgws\QueryBuilderParser` library is created with the allowed fields.
4. `parse($json, $query)` is called to apply all `where`, `orWhere`, and logical groups.
5. The modified `Builder` object is returned.

**Usage Example**:
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$rules = [
    'condition' => 'AND',
    'rules' => [
        ['id' => 'price', 'operator' => 'greater', 'value' => 100],
        ['id' => 'category', 'operator' => 'equal', 'value' => 5]
    ]
];
$query = Product::query();
$finalQuery = QueryBuilderParserHelper::parse($query, $rules);
$products = $finalQuery->get();
```

---

#### 2.2 `QueryBuilder` – FormWidget for Building Queries  
- **Path**: `Nano2\QueryBuilder\FormWidgets\QueryBuilder`
- **Function**: Provides a custom field in NanoSoft forms that displays the jQuery QueryBuilder interface to the user, and stores the rules as JSON in the database.

**Main Configurable Properties (via YAML or class properties)**:

| Property | Description | Default Value |
|----------|-------------|---------------|
| `filters` | Definition of available filters (Filter objects). Can be an array or a string referring to a model method. | `[]` |
| `rules` | Default rules displayed when the interface loads. | `[]` |
| `sort_filters` | Sort filters alphabetically by translation. | `false` |
| `allow_groups` | Maximum number of nested groups (`-1` = unlimited, `0` = no groups). | `-1` |
| `allow_empty` | Allow empty rules. | `false` |
| `conditions` | List of allowed logical connectors (`AND`, `OR`). | `['AND', 'OR']` |
| `default_condition` | Default connector for the root group. | `'AND'` |
| `inputs_separator` | Separator between multiple input fields (e.g., BETWEEN). | `' , '` |
| `display_empty_filter` | Show an empty option in the filters list. | `true` |
| `select_placeholder` | Text shown when no filter is selected. | `'------'` |
| `plugins` | List of enabled plugins (e.g., `sortable`, `filter-description`). | `[]` |
| `isShowParseSqlBtn` | Show a button to translate rules to SQL. | `false` |
| `isShowResetBtn` / `isShowSetBtn` / `isShowGetBtn` | Reset, load rules, and get rules buttons. | `true` |

**Workflow in the Form**:
1. In the `init()` method, settings are loaded from YAML or properties.
2. In `prepareVars()`, the `filters` array is prepared (supporting translation, calling model methods to get options).
3. `$value` (stored JSON rules) is converted to a suitable object for JavaScript.
4. Assets (CSS/JS) for the library and plugins are added.
5. When the partial template (`_query_builder_basic.htm`) is rendered, jQuery QueryBuilder is initialized using the options and rules.
6. When the user changes the rules, a hidden input field (`input[type=hidden]`) is updated with the JSON value, and upon form save, the value is stored in the database.

**Support for Dynamic Option Sources**:
The `values` property in a filter can be a string referring to:
- A method name in the current model (e.g., `getCategoryOptions`).
- A static method in the format `ClassName::method`.
- It is called during filter preparation and passed current values and context.

**Example YAML Configuration**:
```yaml
query_builder:
    type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
    filters:
        category:
            id: category
            label: Category
            type: integer
            input: select
            values:
                1: Books
                2: Movies
                3: Music
            operators: ['equal', 'not_equal', 'in']
        price:
            id: price
            label: Price
            type: double
            validation:
                min: 0
                step: 0.01
    rules: 
        condition: AND
        rules: []
    allow_groups: 3
    sort_filters: true
    plugins:
        sortable: null
        filter-description: { mode: 'popover' }
```

---

### 3. Advanced Filter Options

Each filter follows the same structure defined in the jQuery QueryBuilder library with NanoSoft additions:

| Property | Description |
|----------|-------------|
| `id` | Unique filter identifier. |
| `label` | Display text. Supports translation as array or string. |
| `type` | Data type: `string`, `integer`, `double`, `date`, `time`, `datetime`, `boolean`. |
| `input` | Input element type: `text`, `number`, `textarea`, `radio`, `checkbox`, `select`. |
| `values` | Array of options (for `select`, `radio`, `checkbox`). Can be a function or static array. |
| `operators` | List of allowed operators for this filter (e.g., `equal`, `contains`, `between`). |
| `default_operator` | Default operator. |
| `validation` | Validation rules: `min`, `max`, `step`, `format` (regex or date format), `allow_empty_value`. |
| `placeholder` | Placeholder text inside the input field. |
| `plugin` | jQuery plugin name (e.g., `datepicker`, `selectize`). |
| `plugin_config` | Plugin-specific settings. |

**Two ways to provide filters**:
1. **Directly in YAML file** (as in the example above).
2. **Via a model method**:
   - General method: `getQueryBuilderFiltersOptions($columnName)`
   - Field-specific method: `get{FieldName}QueryBuilderFiltersOptions($fieldName, $columnName)`

**Example model method**:
```php
public function getQueryBuilderFiltersOptions($columnName)
{
    if ($columnName == 'filters') {
        return [
            ['id' => 'status', 'label' => 'Status', 'type' => 'integer', 'input' => 'select', 
             'values' => [1 => 'Active', 0 => 'Inactive']],
            // ... 
        ];
    }
    if ($columnName == 'rules') {
        return ['condition' => 'AND', 'rules' => []];
    }
    return [];
}
```

---

### 4. Plugin Support in the FormWidget

A set of plugins provided by the jQuery QueryBuilder library can be enabled with custom settings:

| Plugin Name | Description | Options |
|-------------|-------------|---------|
| `sortable` | Enable drag-and-drop to reorder rules and groups. | `icon`, `inherit_no_sortable` |
| `filter-description` | Show a descriptive tooltip for the filter (Inline, Popover, or Bootbox). | `mode`, `icon` |
| `bt-tooltip-errors` | Display validation errors as Bootstrap tooltips. | `placement` |
| `bt-checkbox` | Style `checkbox` and `radio` elements using Bootstrap. | `color` |
| `unique-filter` | Prevent using the same filter more than once (globally or within a group). | – |
| `not-group` | Add a "NOT" option to invert group logic. | `icon_checked`, `icon_unchecked` |
| `invert` | Button to invert conditions (convert `=` to `!=`, `AND` to `OR`...). | `recursive`, `invert_rules` |

**Activation in YAML**:
```yaml
plugins:
    sortable: { icon: 'bi-sort-down' }
    filter-description: { mode: 'bootbox', icon: 'bi-info-circle' }
    bt-tooltip-errors: { placement: 'top' }
```

---

### 5. Complete Example Using FormWidget and ParserHelper

#### 5.1 Field Definition in a Model (fields.yaml)
```yaml
fields:
    query_conditions:
        label: Report Conditions
        type: Nano2\QueryBuilder\FormWidgets\QueryBuilder
        filters:
            name: { id: name, label: Name, type: string }
            created_at: { id: created_at, label: Created At, type: date, input: datepicker, plugin: datepicker }
        allow_groups: 2
        sort_filters: true
        plugins: { 'sortable': null, 'filter-description': { mode: 'inline' } }
```

#### 5.2 Saving Rules from the Controller
When the form is saved, the value will be stored as JSON in the column associated with the field.

#### 5.3 Using Rules in a Query
```php
use Nano2\QueryBuilder\Classes\QueryBuilderParserHelper;

$reportRules = $model->query_conditions; // JSON string or array
if (!empty($reportRules)) {
    $query = Product::query();
    $query = QueryBuilderParserHelper::parse($query, $reportRules);
    $results = $query->get();
}
```

---

### 6. Added Value

- **For developers**: Significant reduction in development time for reports and search screens; no need to write complex `where` conditions manually. Unified way to define filters and reuse them across projects in the NanoSoft environment.
- **For end users**: Intuitive and easy interface to build custom queries without needing SQL or backend data structure knowledge.
- **For the system**: Separation of query building logic from the user interface, with extensibility via new plugins and seamless integration with the NanoSoft system (translation, permissions, dynamic model methods).

---

### 7. Conclusion

The `Nano2.QueryBuilder` update is a qualitative addition to the NanoSoft platform, providing developers with a powerful and flexible tool to create complex query conditions visually and securely. Thanks to the integration between the powerful `jQuery QueryBuilder` library on one hand, and the `QueryBuilderParserHelper` class that processes conditions and applies them to a `Builder` on the other, it is now possible to build advanced reporting and filtering systems quickly and with high quality. The modular design of the `FormWidget` also allows the component to be reused in any form with full support for customizing filters and plugins.


---

## Reference Documentation

- [`QueryBuilder` FormWidget Documentation](./docs/querybuilder/Docs-FormWidgets-QueryBuilder-en.md)
- [`QueryBuilderParserHelper` Class Documentation](./docs/querybuilder/Docs-QueryBuilderParserHelper-en.md)
