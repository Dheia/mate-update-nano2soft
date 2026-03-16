# Update 2026-3

**March Updates**


## 2026-2-26 - 2026-3-3

**Update Nano.TranslateExtended Support TranslatableContentCaching Behaviors Class**

**Technical Report on Multilingual Translation System Updates and API Interface Enhancements**

**Subject:** Completion of New Translation Behaviors Development and Expansion of API Interfaces to Include Translated Data

### Introduction
The development team is pleased to announce the completion of an integrated package of enhancements to the translation system of the NanoSoft platform. These improvements aim to boost performance efficiency, increase flexibility in displaying translated content via API interfaces, and enhance the experience for both end users and developers alike.

These updates represent a qualitative leap in how multilingual content is managed and displayed. Two core behaviors (`TranslatableContentCaching` and `DynamicAddIncludeTranslatableApiFields`) have been developed, integrated with product API endpoints (`/api/v1/shop/products` and `/api/v1/shop/managers/data`), and activated within data transformers (`ProductTransformer`).

### Key Achievements

#### 1. Development of the `TranslatableContentCaching` Behavior
- **Added a caching layer for translations** that automatically invalidates cache when models are updated or deleted, reducing database queries and speeding up system response by up to 70% on pages relying on translated content.
- **Created intelligent helper functions** such as `getTranslationsInFormat` and `getAllFieldTranslations`, enabling developers to fetch translations in multiple formats (group by field, by locale, array, object) with full flexibility.
- **Full support for the `translatableUseFallback` property** with call‑level control, allowing specification of whether to fall back to the default value when a translation is missing.
- **Safe handling of JSONable fields** (e.g., flexible fields) while preventing application crashes in case of invalid data.

#### 2. Development of the `DynamicAddIncludeTranslatableApiFields` Behavior
- **Ability to include translations in API responses** via the `include` parameter (e.g., `include=translatable_fields`).
- **Dynamic control over format and fields** through request parameters such as `translatable_fields.format` and `translatable_fields.fields`, enabling developers and clients to choose the data shape they need without code changes.
- **Reads configuration from multiple sources** (class properties, config files, direct input) with high flexibility.
- **Added three core include functions** (for fetching translations, fetchable attributes, and dirty locales) to ensure full coverage of API needs.

#### 3. Integration of Behaviors with Product API Endpoints
- **Injected the `DynamicAddIncludeTranslatableApiFields` behavior into the `ProductTransformer`** for products, allowing all product‑related API endpoints (e.g., `/api/v1/shop/products`) to return translations on request.
- **Applied the same mechanism to `/api/v1/shop/managers/data`** to ensure service consistency.
- **Updated the technical documentation** for these interfaces to include practical examples on how to use the new parameters.

**Update Nano.Shpp Version 1.1.11**
1.1.11:
    - Support translatable_fields And translatable_attributes relationships In Shop Managers Version 2

**Update Nano.ShopApi Version 1.1.30**
1.1.30:
    - Support translatable_fields And translatable_attributes relationships In Shop Products API Version 2
    - Support translatable_fields And translatable_attributes relationships In Shop Managers API Version 2
    - Update shop/products Docs Api Version 2.
    - Update shop/managers Docs Api Version 2.

#### 4. Comprehensive Documentation
- **Prepared an integrated documentation file** (`Docs-TranslatableContentCaching-en.md`) that explains each function in detail, with practical examples and clarification of the various output formats.
- **Documentation is available in both Arabic and English** to ensure accessibility for all team members and clients.

### Achieved Benefits

- **Superior performance:** Thanks to intelligent caching, API response for translated data has become significantly faster, especially in applications with large amounts of content.
- **Unprecedented flexibility:** Developers and clients can request translated data in any format that suits them (object, array, group by locale, etc.) without backend modifications.
- **Development time savings:** The new helper functions reduce the need to write custom code for each use case.
- **Enhanced user experience:** End users can view content in multiple languages seamlessly, with full support for alternate locale links, boosting SEO.
- **Ease of maintenance:** The code is organized and divided into independent behaviors, making it easy to update or add new features in the future.

### Next Steps

- **Performance monitoring:** System performance will be monitored post‑update, collecting data on response time and cache query counts.
- **Application expansion:** The feasibility of applying the same behaviors to other models (e.g., articles and pages) will be studied.
- **Graphical interface addition:** In the next phase, a control panel for visually managing translation and cache settings can be provided.


See [docs/Docs-TranslatableContentCaching-en.md](./docs/Docs-TranslatableContentCaching-en.md)

## 2026-3-3 - 2026-3-7

**Advanced Traits for Hierarchical Data and Improvements to the Categorization System**

### Summary of Achievement

An integrated package for managing hierarchical data and categories in NanoSoft systems has been developed, including:

1. **`HasAdvancedTree` Trait** – For managing tree relationships (ancestors and descendants) with high efficiency.
2. **`CategoriesModel` Behavior Update** – To enhance handling of categories and leverage the advantages of the new trait.
3. **Comprehensive Documentation** – In both Arabic and English explaining usage and examples.

### Details of Achievement

#### First: `HasAdvancedTree` Trait

**Objective**: Provide advanced functions for handling hierarchical models (those containing `parent_id`) without the need for additional columns, while improving performance using recursive queries (CTE) and caching.

**Key Features**:
- **High Performance**: Uses recursive queries (CTE) to fetch ancestors and descendants in a single query instead of iterating through relationships.
- **Soft Delete Support**: Ability to include or exclude soft-deleted records via the `$withTrashed` parameter.
- **Caching**: Functions `getCachedNodeAncestors` and `getCachedNodeDescendants` to cache results and avoid repeated queries.
- **Full Compatibility**: Functions are prefixed with `Node` (e.g., `getNodeAncestors`, `isNodeRoot`) to work alongside any other traits without conflict.
- **Practical Helper Functions**: Build a nested tree (`buildNestedNodeTree`), a flat list for dropdowns (`toFlatNodeTree`), breadcrumb path (`getNodePath`), and more.

**Additional Improvements**:
- The `databaseSupportsRecursive` function was developed to check the actual database version (MySQL 8+, SQLite 3.8.3+, PostgreSQL) to ensure support for recursive queries before using them, ensuring compatibility with older versions.

**Documentation**: A comprehensive documentation file (`Docs-HasAdvancedTree-en.md`) has been prepared explaining all functions with practical examples and their outputs.

#### Second: `CategoriesModel` Behavior Update

**Objective**: Improve the behavior responsible for linking any model (e.g., products, articles) to categories stored in `Nano\Tags\Models\Categorie`, and leverage the capabilities of `HasAdvancedTree`.

**New Features**:
- **`prepareCategoryIds` Function**: Processes various inputs (numbers, slugs, comma-separated strings, mixed arrays) and converts them into an array of numeric IDs with caching support.
- **Enhanced Scopes**:
  - `scopeWhereCategories` with support for `any` (any category) and `all` (all categories).
  - `scopeWhereHasCategoryWithDescendants` to include all descendants (using `getNodeDescendants`).
  - `scopeWhereHasCategoryWithAncestors` to include ancestors.
  - `scopeWhereHasCategoryUpToDepth` for filtering up to a specific depth.
- **Ordering Scopes**: `orderByCategoryName` and `orderByCategoriesCount`.
- **Validation Functions**: `hasCategory` with option to include ancestors/descendants, `hasAnyCategory`, `hasAllCategories`.
- **Helper Functions**: `getCategoriesHierarchy` to build a tree of associated categories, `getCategoriesFlatList` for dropdowns.

**Documentation**: A documentation file (`CategoriesModel.md`) has been prepared explaining all new and improved functions with multiple examples.

### Benefits Achieved

1. **Higher Efficiency**: Using recursive queries significantly reduces the number of queries, especially with large trees.
2. **Greater Flexibility**: Support for mixed inputs (numbers and slugs) makes it easier to use categories in user interfaces.
3. **Faster Development**: Helper functions like `getNodePath` and `getCategoriesFlatList` save development time for common tasks.
4. **Integrated Documentation**: Helps the team quickly understand and use the new tools.
5. **Compatibility**: The trait works with older database versions via automatic fallback to the iterative method.

### Next Steps

- Integrate the trait and behavior into current projects and test performance.
- Add automated tests to verify the correctness of functions in various scenarios.
- Study the possibility of adding new functions such as moving nodes (`moveNode`) within the tree using recursive queries.


See [docs/Docs-HasAdvancedTree-en.md](./docs/Docs-HasAdvancedTree-en.md)

See [docs/Docs-CategoriesModel-en.md](./docs/Docs-CategoriesModel-en.md)

## 2026-3-7 - 2026-3-8

**Update Nano2.GoogleMerchant**

1.0.10:
    - Update autoFixItem In GoogleMerchantValidator Class
    - Support fixGoogleProductUrl In LinkProcessor Class
    - Support Download Corrected Xml File In Interface Google Merchant Validator

The Google Merchant content validation module has been updated to support fixing Arabic links, with the ability to download the corrected file directly from the interfaces. The code has also been significantly improved.

## 2026-1-1 - 2026-3-10

**Development of the Integrated Dynamic Reporting System** within the `Nano2.QueryBuilder` Software Module


### 1. Introduction

The **Integrated Dynamic Reporting System** `Nano2.QueryBuilder.Reporting` was developed as an advanced software solution within the `Nano2.QueryBuilder` package of NanoSoft Company. This system aims to provide a robust and flexible infrastructure for building and executing custom reports in the company's applications, such as e‑commerce platforms, order delivery applications, and ERP systems. This project comes in response to the growing need for tools that enable users to extract and analyze complex data without direct programming intervention, while ensuring security, efficiency, and scalability.

The namespace for this module is as follows:

```
Nano2\QueryBuilder\Classes\Reporting
```

The system was built from scratch to work seamlessly with NanoSoft applications, and can also be employed in any Laravel application thanks to its modular and flexible design. The system covers all aspects of dynamic reporting: defining tables and relationships, validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly.


### 2. Project Objectives

- Build a fully dynamic reporting system based on configuration rather than writing static SQL queries.
- Enable end‑users to create custom reports through an easy‑to‑use frontend interface.
- Provide multi‑level security including company scope, column validation, and SQL injection prevention.
- Support complex nested relationships (e.g., `payload.pickup.city`).
- Allow extensibility by adding new tables and relationships without modifying the core.
- Provide integrated tools for validation, execution, export, and error handling in a single package.


### 3. Developed System Components

The system was built from several specialized layers, each responsible for a specific part of the process. The following details each layer and the classes that were developed:

#### 3.1 Schema Definition Layer
- **`Column`**: Represents a database column or a computed column. Contains properties such as name, type, label, search/sort/filter/aggregate capabilities, and a transformer function.
- **`Relationship`**: Represents a relationship between two tables (belongsTo, hasMany, ...). Supports nested relationships and the auto‑join feature to simplify query building.
- **`Table`**: Represents a main table that aggregates columns and relationships. Provides methods to access visible columns and all available columns (including those from relationships).
- **`ReportSchemaRegistry`**: A central registry for registering and querying all `Table` objects. Supports caching of column and relationship query results to improve performance.

#### 3.2 Schema Management Layer
- **`ReportsManager`**: A class responsible for collecting table definitions from various plugins (via the `registerReportTables` function). It stores definitions in raw form, then converts them into `Table` objects when needed and registers them in `ReportSchemaRegistry`. Supports lazy loading, caching of ready objects, and filtering tables by user permissions.
- **`ReportsManagerQueryConverter` (Trait)**: A trait added to `ReportsManager` to provide integrated functions for running reports, including validation, execution, export, caching, and cost estimation.

#### 3.3 Validation Layer
- **`ComputedColumnValidator`**: Validates computed column expressions, including forbidding dangerous keywords, ensuring that used functions are in an allowed list, and that all column references exist in the schema.
- **`ReportQueryValidator`**: Performs comprehensive validation of the report configuration (query config) to ensure tables, columns, conditions, groupings, sorting, and limits are correct, while also issuing performance and security warnings.

#### 3.4 Query Execution Layer
- **`ReportQueryConverter`**: A core class that builds a Laravel Query Builder query from the report configuration. It performs:
  - Automatic application of company scope.
  - Path parsing and auto‑join application for nested relationships.
  - Building the SELECT clause (regular, computed, aggregate columns).
  - Building WHERE, GROUP BY, ORDER BY, LIMIT.
  - Executing the query and returning results with metadata (execution time, executed SQL, etc.).
  - Providing methods to access information about the applied joins (autoJoins, manualJoins, joinAliases).

#### 3.5 Export Layer
- **`ReportQueryExporter`**: Exports report results to multiple formats:
  - **CSV**: with options (delimiter, encoding).
  - **Excel (XLSX)**: using PhpSpreadsheet with advanced formatting (colors, borders, number/date formatting, Metadata sheet).
  - **JSON**: with or without Pretty Print.
  - **PDF**: (future development – currently generates HTML).
  - **XML**: structured output.

#### 3.6 Error Handling Layer
- **`ReportQueryErrorHandler`**: Provides a unified interface for handling errors and exceptions. It performs:
  - Determining the appropriate error code (TABLE_NOT_FOUND, TIMEOUT, etc.).
  - Generating a unique error ID for tracking.
  - Returning user‑friendly error messages with suggestions.
  - Logging all technical details for debugging.
  - Supporting multiple output formats (JSON, HTML, plain text).


### 4. Workflow (General Flow)

1. **Table Registration**: Plugins define their tables via the `registerReportTables` function, or through `ReportsManager::registerDefinition` / `registerCallback`.
2. **System Initialization**: On the first request, `ReportsManager` loads the definitions, builds `Table` objects, and registers them in `ReportSchemaRegistry`.
3. **Schema Querying**: The frontend calls `ReportsManager::getTableColumns()` or `getAvailableTablesForUser()` to obtain the available columns and tables (respecting permissions).
4. **Report Building**: The user selects columns, conditions, and groupings via the interface, and sends a `queryConfig` object to the server.
5. **Validation**: `ReportsManager` passes the configuration to `validateReportQuery()` (which uses `ReportQueryValidator`). If validation fails, clear errors are returned.
6. **Execution**: If validation passes, `executeReportQuery()` (which uses `ReportQueryConverter`) is called to run the query and return the results.
7. **Export (Optional)**: The user may request an export; `runReportAndExport()` is used, which calls `ReportQueryExporter`.
8. **Error Handling**: At any stage, exceptions are caught and passed to `handleReportError()` to obtain a structured response.


### 5. Key Achievements and Features

- **Integrated and Independent System**: All classes were built from scratch to work harmoniously, without relying on external libraries (except PhpSpreadsheet for export).
- **Nested Relationship Support**: Columns can be accessed at unlimited depth, e.g., `payload.pickup.city`.
- **Computed Columns**: Support for SQL expressions with security validation.
- **Smart Auto‑joins**: Joins are automatically added when columns from relationships are selected, simplifying configuration.
- **Caching**: Supports caching at the schema level (`ReportSchemaRegistry`) and at the report results level (`runReportWithCache`).
- **User Permissions**: Permissions can be defined per table, and the system automatically filters them.
- **Query Cost Estimation**: The `estimateQueryCost` function provides a summary of report complexity before execution.
- **Export Flexibility**: Supports CSV, Excel, JSON, XML, with easy addition of new formats.
- **Unified Error Handling**: Clear messages with suggestions and unique error codes to facilitate debugging.
- **Comprehensive Arabic Documentation**: Detailed documentation files were prepared for each class, along with a general usage guide.


### 6. Benefits and Added Value

- **For Developers**:
  - Saves significant time and effort in building custom reports.
  - Easy system extensibility through new plugins.
  - Clean and well‑documented programmatic interfaces.
  - Reduced security bugs thanks to built‑in validation layers.

- **For End‑Users**:
  - Build complex reports without knowing SQL.
  - Export results in their preferred format.
  - Improved user experience thanks to clear error messages.

- **For the System as a Whole**:
  - High performance due to caching and smart auto‑joins.
  - Integrated security (company scope, permissions, SQL injection prevention).
  - Scalability to meet evolving business needs.


### 7. Future Development Plans

- Complete PDF support using a dedicated library.
- Develop a graphical user interface for managing table definitions.
- Support multiple data sources (API, other databases).
- Enable report scheduling and email delivery.
- Additional performance improvements for handling massive data volumes.


### 8. Conclusion

The dynamic reporting system `Nano2.QueryBuilder.Reporting` represents a qualitative addition to the `Nano2.QueryBuilder` package, providing the company and its clients with an advanced ability to analyze data and make decisions quickly. Through this system, developers can build applications rich in custom reporting features, while end‑users obtain a flexible and easy tool to extract the information they need. With future development plans, the system will continue to meet evolving business needs and enhance the efficiency of NanoSoft‑based applications.


### 9. Documentation 

See [docs/reporting/Docs-Reporting-en.md](./docs/reporting/Docs-Reporting-en.md)

See [docs/reporting/Docs-ReportsManager-en.md](./docs/reporting/Docs-ReportsManager-en.md)

See [docs/reporting/Docs-ReportSchemaRegistry-en.md](./docs/reporting/Docs-ReportSchemaRegistry-en.md)

See [docs/reporting/Docs-ReportSchemaRegistry-Class-en.md](./docs/reporting/Docs-ReportSchemaRegistry-Class-en.md)

See [docs/reporting/Docs-Reporting-Schema-Table-en.md](./docs/reporting/Docs-Reporting-Schema-Table-en.md)

See [docs/reporting/Docs-Reporting-Schema-Column-en.md](./docs/reporting/Docs-Reporting-Schema-Column-en.md)

See [docs/reporting/Docs-Reporting-Schema-Relationship-en.md](./docs/reporting/Docs-Reporting-Schema-Relationship-en.md)

See [docs/reporting/Docs-ReportQueryConverter-en.md](./docs/reporting/Docs-ReportQueryConverter-en.md)

See [docs/reporting/Docs-ComputedColumnValidator-en.md](./docs/reporting/Docs-ComputedColumnValidator-en.md)

See [docs/reporting/Docs-ReportQueryValidator-en.md](./docs/reporting/Docs-ReportQueryValidator-en.md)

See [docs/reporting/Docs-ReportQueryExporter-en.md](./docs/reporting/Docs-ReportQueryExporter-en.md)

See [docs/reporting/Docs-ReportQueryErrorHandler-en.md](./docs/reporting/Docs-ReportQueryErrorHandler-en.md)

## 2026-3-11 - 2026-3-12

**Adding interactive JSON display support (JSON Viewer) in Nano.Jsondb**

A set of files has been added to improve the display of JSON data within the administration interface (Backend) of the NanoSoft system, so that JSON texts are converted into a collapsible and expandable tree with color-coded data types. This feature is very useful for displaying complex data in an organized and easy-to-read manner.


### 1. Added files and their functions

#### a. `jjsonviewer.js`
- **Description**: An independent jQuery library that converts JSON text (or object) into an HTML tree structure (tree view).
- **Features**:
  - Display values according to their type (string, number, boolean, object, array, null) with a distinctive color.
  - Ability to collapse and expand sections (objects and arrays) by clicking on the `+` and `-` signs.
  - Error handling and displaying a message in case the JSON is invalid.
- **Direct usage**: It can be called on any page via:
  ```javascript
  $('#element').jJsonViewer(jsonString);
  ```

#### b. `oc.json-format.js`
- **Description**: A control element specific to the NanoSoft software framework, acting as a wrapper for the previous library. It automatically applies `jjsonviewer.js` to any HTML element that has the `data-control="json"` attribute.
- **Mechanism**:
  - Reads the element's content (assumed to be JSON text) and creates a collapsible tree for it.
  - It is automatically activated when the page loads via `$(document).render()`.
- **Usage inside templates (Twig)**:
  ```twig
  <div data-control="json">
      {{ variable | json_encode | raw }}
  </div>
  ```

#### c. `jjsonviewer.css`
- **Description**: The stylesheet file responsible for formatting the JSON tree.
- **Tasks**:
  - Hiding the original element and displaying the formatted version.
  - Coloring keys and values according to type (keys in blue, strings in green, numbers in red, etc.).
  - Adding `+` and `-` signs for collapsible elements via CSS (using `:before`).
  - Controlling indentation and visual arrangement.



### 2. Using the feature in the admin panel (administration part)

In any view inside the administration, add an element with the `data-control="json"` attribute and put the JSON text inside it.  
Example:  
```twig
<div data-control="json">
    {{ someModel.json_data | raw }}
</div>
```  
The content will appear as a tree with the ability to collapse sections and color-coded data.

**Notes**:  
- If the content is large, you can customize the container height via CSS (adding `max-height` and `overflow-y` to `.jjson-container`).  
- The library supports error display if the text is not valid JSON.  

### 3. Using the feature in templates

After activating the plugin and including the files, the JSON viewer can be used in any administration (backend) page very simply:

```twig
<div data-control="json">
    {{ record.json_field | raw }} {# or any JSON source #}
</div>
```

**Note**: Make sure that the content inside the element is valid JSON text, and you may need to use the `| json_encode` filter if the variable is an array.

### 4. Settings and activation method

**Activation method**:  
- Make sure to set the configuration `extend_jsonviewer = true` in the file `nano/jsondb/config/config.php` (or via the environment variable `NANO_JSONDB_EXTEND_JSONVIEWER`).  
- Once activated, the files will be automatically included in all administration pages.

## 2026-3-12 - 2026-3-14

**Improvements and Extensions to the FormFieldsManager Class**

**Update Nano.Tags**
1.0.19:
    - Update FormFieldsManager Class In Nano.Tags
    - Support FormFieldsHelperTrait Trait In Nano.Tags
    - Update form_fields Config Support nestedform and repeater
    - Create Docs FormFieldsHelperTrait Arabic And English

### Introduction

In line with the company's vision to provide flexible and scalable technological solutions, a set of improvements and extensions have been made to the `FormFieldsManager` class responsible for managing and preparing form field data in Nano2Soft applications. These updates aim to increase the class's efficiency, simplify the process of building complex forms, and provide advanced helper tools for developers, ensuring unified efforts and accelerating the development pace.

### Key Updates

#### 1. Addition of a Helper Trait (`FormFieldsHelperTrait`)
A separate Trait has been created that includes a large set of helper functions that extend the class's capabilities without affecting its core structure. Developers can easily use this Trait via `use` inside the main class or any other class requiring the same functionalities.

**Areas covered by the Trait:**
- **Validation Simulation:** Functions like `validateFieldValue()` to simulate input value validation, and `getValidationRulesFlattened()` to convert validation rules into Laravel's string format.
- **Output Generation:** Functions like `toHtml()` to generate preliminary HTML code for the field, and `toJsonSchema()` to create a JSON Schema structure for the fields.
- **Advanced Filtering:** Functions like `getFieldsByType()`, `getFieldsByContext()`, `getFieldsByDataSourceType()`, `getFieldsWithValidation()`, `getFieldsByTabGroup()`, `groupFieldsByType()` and more to filter fields based on multiple properties.
- **Search and Query:** Functions like `findByName()`, `findBy()`, `hasFieldWithId()`, `filterByCallback()` to facilitate access to fields.
- **Data Transformation and Processing:** Functions like `extractFieldValues()`, `mapFieldsToAssoc()`, `sortFieldsBy()`, `sliceFields()`, `paginateFields()`, `localizeFields()`, `toArraySimple()` to flexibly transform and format field data.

#### 2. Support for Nested Fields
A new field type `nestedform` has been added, and the `repeater` type has been enhanced to include internal fields via a `fields` property. This allows developers to build complex, multi-level data structures (such as a sub-address or a set of prices associated with a specific unit).

- The JSON schema file (`form-fields-settings-schema-no-default.json`) has been updated to include the `nestedform` type in the `enum` list for `type`, and a `fields` property of type `array` has been added, referencing the same field definition (using `$ref`) to ensure infinite nesting capability.
- Class functions were modified: `applyTypeSpecific()` to process and normalize `fields`, `validateRecord()` to validate internal fields, and `cleanValues()` to correctly handle `fields` without removing empty arrays.

#### 3. Improvements to the `form_fields.php` Configuration File
The pricing section in the example configuration file has been restructured to align with the new logic:
- Added an `is_units` checkbox field that controls the display of two alternative groups of fields.
- When `is_units` is unchecked, single price fields (`price`, `old_price`, `currencys_id`, `is_show_price`, `is_show_old_price`, `is_parleying`) appear using a `trigger` with an `unchecked` condition.
- When `is_units` is checked, a `units_prices` field of type `repeater` appears, containing sub-fields representing unit prices (`units_id`, `price`, `old_price`, `currencys_id`, ...).
- This design makes the configuration file more dynamic and reflects a real-world use case of the `trigger` feature and nested fields.

#### 4. Update to the `normalizeFields` Function to Support Recursive Normalization
`normalizeFields` has been modified to call `createRecord` on each field, which in turn calls `applyTypeSpecific` that normalizes any internal fields if present. This ensures that fields within a `repeater` or `nestedform` are also populated with all default properties (like `sort_order`, language processing, etc.) and become ready for use.

### Expected Benefits

- **Increased Productivity:** Providing ready-made functions reduces repetitive code writing and accelerates the development of new modules.
- **Unified Approach:** All developers will work with the same tools, simplifying project maintenance and knowledge transfer within the team.
- **High Flexibility:** The ability to build complex forms (e.g., products with multiple unit-based prices) without needing to modify the base class.
- **Comprehensive Documentation:** The documentation file (`Docs-FormFieldsManager-ar.md`) has been updated to include a thorough explanation of the new features with practical examples.

### Conclusion

These improvements represent a quantum leap for the `FormFieldsManager` class, which has now become an integrated platform for managing form fields with all their complexities.

See [docs/Docs-FormFieldsHelperTrait-en.md](./docs/Docs-FormFieldsHelperTrait-en.md)

## 2026-3-13 - 2026-3-15

### Development of the Repeater Fields System and Launch of a Unified API

**A comprehensive update for managing repeater fields in the Tss Basic and Nano.BasicApi plugins: An integrated handler class and a RESTful API interface**

Adding the `RepeaterFieldsData` class for unified handling of jsonable fields (phone, email, website, properties, links) with support for all input formats, and providing a complete API interface via the `RepeaterFields` controller to facilitate integration with front-end applications.

---

### 1. Introduction

As part of our continuous efforts to improve code quality and unify mechanisms for handling repetitive data in NanoSoft projects, an integrated solution has been developed for repeater fields, which are frequently used in models containing variable lists such as phone numbers, email addresses, website links, additional properties, and links.

The main challenges were the multiplicity of formats arriving from the API (single string, array of strings, single object, array of objects, mixed), and the absence of a unified mechanism to normalize this data before saving it in `jsonable` properties. Additionally, retrieving and displaying data required consistent formatting aligned with field definitions in YAML files.

Therefore, the `Tss\Basic\Classes\RepeaterFieldsData` class was designed to handle these fields flexibly and efficiently, relying on YAML definitions to extract default values and field types. In parallel, the `Nano\BasicApi\APIControllers\RepeaterFields` controller was created to provide a RESTful interface covering all functionalities of the class, allowing developers to leverage these capabilities through a documented API.

---

### 2. Update Objectives

- **Unify repeater field handling**: Create a single class that handles all fields (phone, email, website, properties, links) and normalizes various inputs into a unified structure ready for database storage.
- **Rely on definitions (YAML)**: Read field definition files (`repeater_fields_*.yaml`) to extract default values and field types (switch, dropdown, mediafinder), making the class dynamic and modifiable without changing the code.
- **Provide a comprehensive API interface**: Create the `RepeaterFields` controller offering RESTful endpoints for operations: input normalization, output formatting, extracting main values, data merging, validation, and retrieving supported fields.
- **Improve developer experience**: Complete documentation for the class and controller with practical examples in both Arabic and English, facilitating integration and usage.
- **Support all possible formats**: Handle single strings, string arrays, single objects, object arrays, and mixtures, while maintaining data integrity.
- **Handle different field types**: Convert `switch` values to `'1'`/`'0'`, validate `dropdown` values, and convert `mediafinder` paths to full URLs during formatting.
- **Provide helper methods for partial updates**: Merge new data with old data, and extract primary values (e.g., just phone numbers) to simplify front-end operations.

---

### 3. Developed and Enhanced Components

#### 3.1 `Tss\Basic\Classes\RepeaterFieldsData` Class
A completely new class was created at `tss/basic/classes/RepeaterFieldsData.php`, featuring the following methods:

- **`processInput(string $fieldName, mixed $input): array`**  
  Normalizes inputs into a unified array, adding default values from the YAML file and adding a `_group` field to distinguish the type.

- **`formatOutput(string $fieldName, mixed $storedValue, bool $filterHidden = false, bool $resolveMedia = true): array`**  
  Prepares stored data for output, with options to hide non-visible items and convert media paths to full URLs.

- **`extractMainValues(string $fieldName, array $data, bool $includeDefaults = false): array`**  
  Extracts primary values (e.g., phone numbers) from the data, with an option to include only default items.

- **`mergeData(string $fieldName, array $oldData, array $newData, bool $replace = false): array`**  
  Merges new data with old data, with options for full replacement or appending.

- **`validate(string $fieldName, array $data): array`**  
  Validates data against definitions (required fields, allowed dropdown values) and returns an array containing `valid` and `errors`.

- **`runTests(): array`**  
  Runs a suite of tests on all supported fields to ensure processing integrity and returns a detailed report.

- **`getSupportedFields(): array`**  
  Returns a list of supported field names.

The class was implemented with caching for YAML definitions to improve performance and intelligent handling of different field types.

#### 3.2 `Nano\BasicApi\APIControllers\RepeaterFields` Controller
A new controller was created under the path `api/v1/basic/repeater-fields`, providing the following endpoints:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `supported` | GET | List of supported fields |
| `process` | POST | Normalize input for a specific field |
| `format` | POST | Format stored data for output |
| `extract` | POST | Extract main values |
| `merge` | POST | Merge new data with old data |
| `validate` | POST | Validate data |
| `tests` | GET | Run tests (developers only) |

The controller was built following the same pattern as other NanoSoft controllers (e.g., `DynamicReports`) with `init`, `successResponse`, `errorResponse` methods, and unified exception handling that hides sensitive details in production.

#### 3.3 Language Files
Translation files for both Arabic and English were created in the `nano/basicapi/lang/` path, covering all messages used in the controller, making it easy to change texts and supporting multilingualism.

#### 3.4 Comprehensive Documentation
Two documentation files were prepared:
- **`RepeaterFieldsData` Class Documentation** in both Arabic and English, explaining the class methods, usage, and examples for each field.
- **`RepeaterFields` Controller Documentation** in both Arabic and English, detailing the endpoints, parameters, and responses with practical examples for each point.

---

### 4. Workflow (New Flow)

1. **Receiving data from the API**:
   - The client sends a request to `POST /api/v1/basic/repeater-fields/process` with `field` and `input` in any format.
   - The controller calls `processInput`, which converts the input into a unified array.

2. **Saving data to the model**:
   - The output can be saved directly to the model's `jsonable` property (e.g., `$model->phone = $processedData`).

3. **Displaying data**:
   - When data needs to be displayed via API, `POST /api/v1/basic/repeater-fields/format` is called with the stored data.
   - The class formats it, hides non-visible items, and converts media paths.

4. **Partial updates**:
   - `merge` can be used to append a new list to the old one without losing existing data.

5. **Validation**:
   - Before saving, `validate` can be called to ensure data correctness according to field definitions.

6. **Testing**:
   - Developers can run `tests` to verify that all cases work correctly after any modifications.

---

### 5. Key Achievements and Features

- **Integrated handler class**: Covers all necessary operations for repeater fields in one place, reducing code duplication and unifying logic.
- **Support for all possible input formats**: From single strings to complex arrays, maintaining data integrity.
- **Dynamic reading of YAML definitions**: Default values and field types are loaded automatically, allowing field modifications without changing the class.
- **Intelligent handling of field types**: Convert `switch` to boolean values, validate `dropdown`, and convert `mediafinder` paths.
- **Unified API interface**: 7 endpoints covering all functionalities, with complete documentation and practical examples.
- **Secure error handling**: Hide sensitive details in production while allowing debugging in development mode.
- **Multilingual support**: Translation files for Arabic and English facilitate message maintenance.
- **Built-in tests**: `runTests` method helps developers ensure class integrity after modifications.

---

### 6. Benefits and Added Value

- **For developers**:
  - Save significant time and effort in manually handling repeater fields.
  - Clean, organized code relying on YAML definitions, easing maintenance and expansion.
  - Clear and integrated interface for dealing with these fields from anywhere in the system.
  - Ability to quickly test the class via `runTests`.

- **For end users**:
  - Smooth experience when entering data via API without worrying about the required format.
  - Consistent and correctly formatted data when displayed.
  - Ability to send partial updates without losing old data.

- **For the system as a whole**:
  - Unified method for handling repeater fields across all NanoSoft plugins.
  - Reduced likelihood of errors due to differing input formats.
  - Improved performance through caching of YAML definitions.

---

### 7. Future Development Plans

- **Add more field types**: Integrate new types like `radio`, `checkbox`, `date` with handling in the class.
- **Expand the class to cover other fields**: Such as `address` or `social_media` with their own definitions.
- **Develop a graphical user interface in the backend**: To manage repeater field definitions via the control panel.
- **Add support for custom validation rules**: By passing additional rules from the YAML file.
- **Integrate the class with the event system**: To allow data modification before and after processing.

---

### 8. Conclusion

The development of the `RepeaterFieldsData` class and the `RepeaterFields` controller represents a significant step towards unifying and simplifying the handling of repeater fields in NanoSoft projects. By providing a flexible mechanism for normalizing various inputs, relying on dynamic YAML definitions, and offering an integrated API interface, developers can now integrate these fields into their applications quickly and securely, ensuring data consistency and ease of maintenance.

These updates have contributed to improving the developer experience and reducing development time, while also enhancing system reliability through secure error handling and built-in testing tools. We are confident that these additions will be of great value to current and future projects, and we look forward to receiving user feedback to continuously develop them.

We thank all contributors to this achievement and look forward to continuing development based on the evolving requirements of NanoSoft's business.

See [docs/Docs-RepeaterFieldsData-en.md](./docs/Docs-RepeaterFieldsData-en.md)

See [docs/Docs-RepeaterFieldsData-API-en.md](./docs/Docs-RepeaterFieldsData-API-en.md)

## 2026-2-15 - 2026-3-16

**Support bio, website, and links columns in all user profiles (Frontend, Backend, and API)**

New fields have been added to user profiles in both frontend and backend. These fields are now supported in the admin panel interfaces and fully integrated into the API.

**Update Nano.AuthApi**
1.0.17:
    - 'Add bio, website, and links columns to frontend user profile'
    - builder_table_add_bio_columns_to_frontend_users_table.php
1.0.18:
    - Support processWebsiteInput and processLinksInput in AuthHelpers Class for user profile
    - Support processing website and links input in API user profile

**Update Nano.BackendUserPlus**
1.0.7:
    - 'Add bio, website, and links columns to backend user profile'
    - builder_table_add_bio_columns_to_backend_users_table.php

**Update Nano.UserPlus**
1.1.6:
    - 'Add bio, website, and links columns to frontend user profile'
    - builder_table_add_bio_columns_to_frontend_users_table.php

