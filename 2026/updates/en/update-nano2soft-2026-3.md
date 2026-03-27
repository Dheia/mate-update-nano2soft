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

## 2026-3-17

**Support WhatsApp field in Shop Products (Frontend, Backend, and API)**

A new field for WhatsApp phone number has been added to products. This field is used in the Haraj, Taysir, and Souq Aqar platforms to allow product or ad owners to specify a separate number for WhatsApp communication, instead of relying on the previously existing mobile field.
The interfaces in the admin panel have been updated, as well as the API section for products and product management. Documentation has also been updated.

**Update Nano.Shop**
1.1.12:
    - Support WhatsApp field in Product Backend Interface
    - Support HasWhatsappField in Product Models and Shop Managers Version 2

**Update Nano.ShopApi**
1.2.1:
    - Support WhatsApp field in Shop Products API Version 2
    - Support WhatsApp field in Shop Managers Products API Version 2
    - Update shop/products API documentation Version 2
    - Update shop/managers API documentation Version 2

## 2026-01-28 - 2026-03-18

**Comprehensive Development of the Units Management System and Addition of an Integrated Unit Converter**  
Within the `Tss.Inventory` Package

---

### 1. Introduction

An integrated development package has been implemented aimed at bringing about a quantum leap in units management within the TSS Inventory system. It is no longer limited to simply storing unit names; we now have an integrated system that supports **unit conversion**, **different contexts**, **subtypes**, and logical validation.

Everything related to units has been completely restructured, starting from the database, through the models, and ending with helper classes that provide a rich and easy-to-use programming interface. The goal is to enable developers to handle units with high flexibility, whether in conversion operations, linking with products, or displaying data in user interfaces.

This update aims to address advanced use cases such as:
- Converting values between different measurement units (length, weight, volume, quantity, …) with high precision.
- Handling complex quantity units (dozen, ream, pair, …) within their logical contexts.
- Preventing illogical conversions between different contexts (e.g., converting dozen to ream).
- Flexibly linking units to products with the ability to specify the main unit and conversion factors.
- Providing helper classes for creating units, retrieving information, and advanced searching.

---

### 2. Developed Components

#### 2.1 Conversion System Classes (within `Tss\Inventory\Classes\Units`)

| Class | Description |
|-------|-------------|
| `UnitType` | A static class for defining unit types (length, weight, volume, …) and quantity subtypes. Provides methods for type validation, getting labels, and type categorization. |
| `UnitNameHelper` | A helper class for handling unit names and symbols. Provides lists of units with Arabic and English labels, unit searching, retrieving unit information, and validity checking. |
| `UnitConverter` | The heart of the conversion system. Contains the base conversion data for all types and performs conversion operations with precise support for temperatures. |
| `UnitContextValidator` | Responsible for verifying the logical validity of conversions between different quantity units (contexts). Prevents illogical conversions such as dozen to ream. |
| `UnitFactory` | A factory for creating `Unit` objects from symbols or types. |
| `UnitHelper` | General helper functions for dealing with units, such as formatting, getting the best display unit, and generating HTML options. |
| `UnitCreator` | A specialized class for creating units in the database and linking them to products and prices. Provides safe methods with transaction support. |
| `ConversionForm` | (For testing purposes) Provides HTML forms to test conversions. |
| `SmartUnitConverter` | Provides conversions with warnings or strict logical checking. |
| `UnitsInformation` | Displays complete information about all supported units in a formatted HTML interface (for documentation and testing purposes). |

#### 2.2 Model Enhancements

The following models have been updated to support the new system:

- **`Unit`**: Added new columns (`unit_symbol`, `conversion_factor`, `value`, `is_base_unit`, `is_public`, `is_published`, `published_at`, `unpublished_at`). Added Traits: `HasBaseUnit`, `HasDefault`, `HasRecordsOptions`, `ListObjects`, `ListOptions`, `FieldsOptions`, `HasScopesModel` to provide advanced features such as caching, record retrieval, and dynamic options.
- **`ProductsUnit`**: Added `unit_symbol` and `type` columns to link units to products while retaining basic unit information. Improved duplicate checking.
- **`ProductsPricesUnit`**: Added `unit_symbol` column for easier access and searching. Improved `makePrimary` and `makeIsActive` methods.

#### 2.3 `UnitManager` (New Helper Class)

A new class has been created at `Tss\Inventory\Classes\UnitManager` to provide static methods that simplify unit management and common operations such as creation, validation, and printing. This class relies on the subclasses (`UnitCreator`, `UnitConverter`, `UnitHelper`, …) and provides a unified interface for developers.

**Available Methods**:
- `createUnit`: Create a new unit in the database.
- `checkUnit`: Validate a unit (its symbol, balance, …).
- `getQueryDate`: Apply date conditions to queries.
- `scopeWhereField`: A helper scope to apply conditions on fields with support for `*` and `all`.
- `printReports`: Print unit reports.
- `getAllUnitsGrouped`: Get all units grouped by type.
- `getUnitsForType`: Get units of a specific type.
- `getRentalTimeUnits`: Get time units used in rental contexts.
- `formatDecimal`: Format decimal numbers without trailing zeros.

#### 2.4 Database Updates (Migrations)

Several migration files were added to prepare the tables for the new system:

| Version | Description | File |
|---------|-------------|------|
| 1.1.13 | Add `unit_symbol` column to `tss_inventory_units` | `builder_table_add_unit_symbol_columns_to_tss_inventory_units.php` |
| 1.1.14 | Add `unit_symbol` column to `tss_inventory_products_units` | `builder_table_add_unit_symbol_columns_to_tss_inventory_products_units.php` |
| 1.1.15 | Add `unit_symbol` column to `tss_inventory_products_prices_units` | `builder_table_add_unit_symbol_columns_to_tss_inventory_products_prices_units.php` |
| 1.1.16 | Add `conversion_factor` and `value` columns to `tss_inventory_units` | `builder_table_add_conversion_factor_columns_to_tss_inventory_units.php` |
| 1.1.17 | Update `type` values in `tss_inventory_units` from old to new (Numerical → quantity, Mass → weight, …) | `update_type_values_in_iss_inventory_units_table.php` |
| 1.1.18 | Add `is_base_unit` column to `tss_inventory_units` | `builder_table_add_is_base_unit_columns_to_tss_inventory_units.php` |
| 1.1.19 | Add `type` column to `tss_inventory_products_units` | `builder_table_add_type_columns_to_tss_inventory_products_units.php` |
| 1.1.20 | Add `is_public`, `is_published`, `published_at`, `unpublished_at` columns to `tss_inventory_units` | `builder_table_add_is_published_columns_to_tss_inventory_units.php` |
| 1.1.21 | Add the same columns to `tss_inventory_prices` | `builder_table_add_is_published_columns_to_tss_inventory_prices.php` |
| 1.1.22 | Support the new fields in the units and prices interfaces (reuse the same migration) | - |
| 1.1.23 | Support the `CUSTOM` type in `UnitType` (code update) | - |
| 1.1.24 | Change the `conversion_factor` and `value` columns to `decimal(20,10)` for higher precision | `builder_table_change_conversion_factor_columns_to_tss_inventory_units.php` |
| 1.1.25 | Change the same columns in `tss_inventory_products_units` as well | `builder_table_change_conversion_factor_columns_to_tss_inventory_products_units.php` |

---

### 3. Details of Code Updates

#### 3.1 `UnitType` – Redefining Types

`UnitType` has been expanded to include main and subtypes:

```php
const LENGTH = 'length';
const WEIGHT = 'weight';
const VOLUME = 'volume';
const AREA = 'area';
const TEMPERATURE = 'temperature';
const TIME = 'time';
const SPEED = 'speed';
const PRESSURE = 'pressure';
const ENERGY = 'energy';
const POWER = 'power';
const DATA = 'data';
const ANGLE = 'angle';
const QUANTITY = 'quantity';
const CUSTOM = 'custom';

// Quantity subtypes
const QUANTITY_GENERAL = 'quantity_general';
const QUANTITY_PACKAGING = 'quantity_packaging';
const QUANTITY_PAPER = 'quantity_paper';
const QUANTITY_PAIRS = 'quantity_pairs';
const QUANTITY_LARGE = 'quantity_large';
```

Methods such as `isQuantitySubtype`, `getMainType`, `getAllTypesWithLabels` were added to facilitate categorization.

#### 3.2 `UnitNameHelper` – Unit Information

This class provides comprehensive lists of all supported units with Arabic and English labels, and allows advanced searching:

```php
// Get a unit label
UnitNameHelper::getUnitLabel('kg', 'ar'); // "كيلوجرام"

// Advanced unit search
UnitNameHelper::searchUnitAdvanced('كيلو'); // Returns info of the first matching unit

// Get complete info
UnitNameHelper::getUnitInfo('dozen', 'ar');
/*
[
    'symbol' => 'dozen',
    'label' => 'درزن',
    'english_label' => 'Dozen',
    'type' => 'quantity_packaging',
    'type_label' => 'Packaging Quantity',
    'is_base_unit' => true,
    'context' => 'packaging',
    'conversion_factor' => 12.0,
    ...
]
*/
```

#### 3.3 `UnitConverter` – Unit Converter

The heart of the system, based on optimized `CONVERSION_DATA` arrays. Supports conversion between any two compatible units, with special handling for temperatures.

```php
// Convert 5 kilograms to pounds
$result = UnitConverter::convert(5, 'kg', 'lb', 4); // 11.0231

// Convert temperature
$result = UnitConverter::convert(100, 'c', 'f', 2); // 212.00

// Check if conversion is possible
UnitConverter::canConvert('dozen', 'ream'); // false
```

#### 3.4 `UnitContextValidator` – Logical Validation

Ensures that conversions between quantity units are logical (within the same context):

```php
UnitContextValidator::isConversionLogical('dozen', 'box'); // true (both packaging)
UnitContextValidator::isConversionLogical('dozen', 'ream'); // false (packaging → paper)
```

#### 3.5 Model Enhancements

- **`Unit`**: Traits were added to organize the code and provide advanced features:
  - `HasBaseUnit`: Handling the base unit of a type.
  - `HasDefault`: Making a unit default.
  - `ListObjects` & `ListOptions`: Caching records and option lists.
  - `HasRecordsOptions`: `getRecords` method for flexible record retrieval.
  - `FieldsOptions`: Field options methods (for interfaces).
  - `HasScopesModel`: Advanced query scopes.

- **`ProductsUnit`**: Added `unit_symbol` column to speed up queries, and `type` column for classification. Improved duplicate checking.

- **`ProductsPricesUnit`**: Added `unit_symbol` column, and improved `makePrimary` and `makeIsActive` methods to correctly update records.

#### 3.6 `UnitManager` – Unified Interface

`UnitManager` was created as a single entry point for developers:

```php
use Tss\Inventory\Classes\UnitManager;

// Create a new unit
$result = UnitManager::createUnit([
    'name' => 'Kilogram',
    'unit_symbol' => 'kg',
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
]);

// Validate a unit (e.g., a recharge card code)
$result = UnitManager::checkUnit(['unit_symbol' => 'CARD123']);

// Get rental time units
$rentalUnits = UnitManager::getRentalTimeUnits('en'); // ['h' => 'Hour', 'day' => 'Day', ...]

// Print a report
UnitManager::printReports($options);
```

---

### 4. Practical Examples

#### 4.1 Creating a New Unit

```php
$result = UnitManager::createUnit([
    'name' => 'Dozen',
    'unit_symbol' => 'dozen',
    'type' => UnitType::QUANTITY_PACKAGING,
    'is_base_unit' => true,
    'conversion_factor' => 12,
]);

if ($result['status']) {
    echo "Unit created successfully: " . $result['model']->name;
}
```

#### 4.2 Attaching a Unit to a Product

```php
use Tss\Inventory\Classes\Units\UnitCreator;

$result = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => 5,
    'is_main' => true,
    'conversion_factor' => 1,
]);
```

#### 4.3 Creating a Price for a Product Unit

```php
$result = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => 5,
    'prices_id' => 1,
    'value' => 100.50,
    'currencys_id' => 1,
]);
```

#### 4.4 Searching for a Unit

```php
$unitInfo = UnitNameHelper::searchUnitAdvanced('kilo');
if ($unitInfo) {
    echo $unitInfo['unit_name_en']; // "Kilogram"
}
```

#### 4.5 Converting a Value Using `UnitConverter`

```php
$value = UnitConverter::convert(10, 'dozen', 'pcs'); // 120
$description = UnitConverter::describeConversion(10, 'dozen', 'pcs');
// "10 dozen = 120 pieces"
```

#### 4.6 Getting Rental Time Units

```php
$rentalUnits = UnitManager::getRentalTimeUnits('en');
// ['h' => 'Hour', 'day' => 'Day', 'week' => 'Week', 'month' => 'Month']
```

#### 4.7 Using `Unit` Scopes

```php
$units = Unit::isActive()
    ->whereType(UnitType::WEIGHT)
    ->whereIsBaseUnit(true)
    ->get();
```

#### 4.8 Creating All Standard Units for a Specific Type

```php
$result = UnitCreator::createStandardUnitsForType([
    'type' => UnitType::LENGTH,
    'companys_id' => 1,
    'departments_id' => 2,
]);
```

---

### 5. Summary of Options in `UnitManager` and Helper Classes

| Class / Method | Description |
|----------------|-------------|
| `UnitManager::createUnit()` | Create a new unit in the database |
| `UnitManager::checkUnit()` | Validate a unit (symbol, balance, user) |
| `UnitManager::getAllUnitsGrouped()` | Get all units grouped by type |
| `UnitManager::getUnitsForType()` | Get units of a specific type |
| `UnitManager::getRentalTimeUnits()` | Get time units used in rental contexts |
| `UnitManager::formatDecimal()` | Format decimal numbers without trailing zeros |
| `UnitCreator::attachUnitToProduct()` | Attach a unit to a product |
| `UnitCreator::createProductUnitPrice()` | Create a price for a product unit |
| `UnitConverter::convert()` | Convert a value between two units |
| `UnitNameHelper::getUnitInfo()` | Get complete information about a unit |
| `UnitNameHelper::searchUnitAdvanced()` | Advanced search for a unit |
| `UnitContextValidator::isConversionLogical()` | Check if a conversion is logical |

---

### 6. Added Value

- **For Developers**: An integrated system for handling units providing ready-made methods for conversion, linking, and querying. The system can be easily extended by adding new types.
- **For End Users**: High precision in conversions, smarter interfaces that prevent illogical conversions, and broad support for various units.
- **For the System**: A modular and clean design that separates data from logic, with caching used to improve performance.

---

### 7. Conclusion

This update represents a quantum leap in units management within the TSS Inventory system. Thanks to the comprehensive restructuring and the addition of specialized classes, developers can now build applications that rely on complex conversions easily and safely. As development continues, this system will remain capable of meeting the needs of diverse projects thanks to its flexibility and robust design.

See [docs/UnitManager/Docs-UnitManager-en.md](./docs/UnitManager/Docs-UnitManager-en.md)

See [docs/UnitManager/Docs-UnitManager-Class-en.md](./docs/UnitManager/Docs-UnitManager-Class-en.md)

See [docs/UnitManager/Docs-UnitType-Class-en.md](./docs/UnitManager/Docs-UnitType-Class-en.md)

See [docs/UnitManager/Docs-UnitNameHelper-Class-en.md](./docs/UnitManager/Docs-UnitNameHelper-Class-en.md)

See [docs/UnitManager/Docs-UnitConverter-Class-en.md](./docs/UnitManager/Docs-UnitConverter-Class-en.md)

See [docs/UnitManager/Docs-UnitHelper-Class-en.md](./docs/UnitManager/Docs-UnitHelper-Class-en.md)

See [docs/UnitManager/Docs-UnitCreator-Class-en.md](./docs/UnitManager/Docs-UnitCreator-Class-en.md)

See [docs/UnitManager/Docs-UnitsInformation-Class-en.md](./docs/UnitManager/Docs-UnitsInformation-Class-en.md)

See [docs/UnitManager/Docs-ConversionForm-Class-en.md](./docs/UnitManager/Docs-ConversionForm-Class-en.md)


## 2026-03-18 - 2026-03-19

The Items Management section of the API has been updated to support all recent changes made to the unit system. A new API specifically for handling units has been created, along with its documentation. The documentation for the Items Management section has also been updated.

**Update Nano.ShopApi Managers**
1.1.29:
    - Support api/v1/shop/units endpoint In Shop API Version 2
    - Support api/v1/shop/managers/units endpoint In Shop API Version 2
    - Support api/v1/shop/managers/groupsproducts endpoint In Shop API Version 2
    - Update shop/products Docs Api Version 2.
    - Update shop/managers Docs Api Version 2.
    - Create shop/units Docs Api Version 2.

## 2026-03-19 - 2026-03-22

**Comprehensive Development of Product Associations System and Addition of Advanced Dynamic Filters**  
Within the `Nano.Shop` plugin

Update of Product Associations Behavior - Advanced Version

---

### 1. Introduction

An integrated development package has been implemented aimed at enhancing the flexibility and effectiveness of managing product associations within the **NanoSoft** system. It is no longer limited to simply creating simple relationships between products (alternatives, complements, newer versions), but rather the `AssociationsModel` behavior has been transformed into a highly flexible tool that allows creating and querying these relationships in advanced ways. A comprehensive development has been carried out on this system within the framework of `Nano.Shop`, transforming it from a simple binary relationship manager into an integrated query platform that enables developers and end-users to benefit from advanced query scopes, ready-made filters for management, precise control over relationship direction and type, in addition to the ability to control the appearance of these filters through the configuration file, allowing the system to be adapted according to each project's needs.

This update adds and meets advanced use cases such as:

**Firstly: Advanced Control over Association Queries**
- Ability to filter products that have alternatives (in any direction, only as parent, or only as target).
- Filter products that have no associations at all.
- Combine association conditions with other conditions using `orWhere` or `andWhere` flexibly.
- Add additional conditions on the associations table (e.g., `is_active`, `is_public`, `companys_id`, `departments_id`).
- Apply specific scopes related to the associations table (`isCompany`, `isDepartment`, ...) with the ability to pass parameters.
- Customize conditions per direction (`conditionsPerDirection`, `scopesPerDirection`, `typePerDirection`).
- Full control over the logic linking directions (`relationOperator`: `and` / `or`) and with the original scope (`boolean`: `and` / `or`).
- Advanced support for `callback` with passing the direction name to customize the query dynamically.

**Secondly: Facilitating Filter Integration in Admin Interfaces**
- Adding ready-made filters to the product management interface easily via a helper class (`AssociationsHelper`) providing static ready-made functions.
- Control the activation or deactivation of specific types of association filters through environment variables or configuration files, allowing interface customization per project.
- Ability to inject or remove filters in a flexible pattern (specific types such as alternatives, complements, newer versions, or any association).

**Thirdly: Extending Toggle Scopes to Include AND and OR Logic**
- Toggle scopes have been improved to respond correctly to values `1` and `0` while handling special cases coming from `post('scopeName')`.
- Added separate scopes for `_or` logic (association exists in either direction) and `_and` logic (association exists in both directions), enabling precise control over filtering products based on association strength.

Thanks to these updates, the product associations system in NanoSoft has become more powerful and ready for use in complex e-commerce projects, providing developers with advanced query tools, and end-users with an enhanced management and filtering experience.

---

### 2. Developed Components

#### 2.1 `AssociationsModel` (Behavior)

This behavior has been fundamentally updated to become more flexible and powerful. The behavior is responsible for managing relationships between products and providing advanced query scopes.

##### A. Basic Scopes (per Association Type)

| Scope | Description |
|-------|-------------|
| `scopeHasAlternate` | Products that have alternative associations (in either direction) |
| `scopeHasCrossSell` | Products that have complementary associations |
| `scopeHasUpSell` | Products that have newer version associations |
| `scopeHasNotAlternate` | Products that have no alternatives |
| `scopeHasNotCrossSell` | Products that have no complements |
| `scopeHasNotUpSell` | Products that have no newer versions |

##### B. Scopes by Direction

| Scope | Description |
|-------|-------------|
| `scopeHasAlternateAsParent` | Products that are parents to alternatives |
| `scopeHasAlternateAsTarget` | Products that are targets (alternatives) |
| `scopeHasCrossSellAsParent` | Products that are parents to complements |
| `scopeHasCrossSellAsTarget` | Products that are targets (complements) |
| `scopeHasUpSellAsParent` | Products that are parents to newer versions |
| `scopeHasUpSellAsTarget` | Products that are targets (newer versions) |

##### C. Scopes for Any Association (Existence / Non-Existence)

| Scope | Description |
|-------|-------------|
| `scopeHasAnyAssociationBoth` | Products that have any association (as parent or target) |
| `scopeHasNotAnyAssociationBoth` | Products that have no associations |

##### D. Advanced `scopeWhereAssociation` Scope

This is the most powerful and flexible scope, accepting an array of options for full control over association type, direction, additional conditions, and integration with the query.

```php
scopeWhereAssociation($query, $options = [], $boolean = 'and', $not = false)
```

**Supported Options**:

| Option | Type | Description |
|--------|------|-------------|
| `type` | string\|array\|null | Relationship type (e.g., `'alternate'`) or array (`['alternate', 'cross_sell']`). |
| `direction` | string\|array | Direction: `'parent'`, `'target'`, `'both'`, or array of directions. (Default `'both'`) |
| `has` | bool | `true` = existence (whereHas), `false` = non-existence (whereDoesntHave). |
| `boolean` | string | `'and'` or `'or'` to link this condition with the original query. (Default `'and'`) |
| `relationOperator` | string\|null | `'and'` or `'or'` for linking multiple directions. If not specified: `'or'` if `has = true`, and `'and'` if `has = false`. |
| `callback` | callable\|null | Function (relationship query, direction name) for additional customization per direction. |
| `callbacks` | array | Array per direction: `['parent' => callable, 'target' => callable]`. |
| `typePerDirection` | array | Customize association type per direction: `['parent' => 'alternate', 'target' => 'cross_sell']`. |
| `conditions` | array\|callable\|null | Additional conditions on the association table (e.g., `['is_active' => 1]`). |
| `scopes` | array | Scopes to apply on the association query (e.g., `['isCompany', 'isDepartment' => [1]]`). |
| `conditionsPerDirection` | array | Conditions specific to each direction: `['parent' => ['is_active' => 1], 'target' => ['is_public' => 1]]`. |
| `scopesPerDirection` | array | Scopes specific to each direction: `['parent' => ['isCompany'], 'target' => ['isDepartment' => [1]]]`. |

**Additional Parameters**:
- `$boolean`: `'and'` or `'or'` to combine the condition.
- `$not`: true to invert the condition.

> **Note:** The `callback` and `callbacks` options are still supported and can be combined with the new options.

##### E. Helper Scopes for OR and NOT

| Scope | Description |
|-------|-------------|
| `scopeOrWhereAssociation` | `orWhere` condition using the same options |
| `scopeWhereNotAssociation` | Negation condition (doesntHave) |
| `scopeOrWhereNotAssociation` | `orWhere` condition with negation |

##### F. General Scopes for Ease of Use

| Scope | Description |
|-------|-------------|
| `scopeHasAssociation` | Has an association of a specific type |
| `scopeHasNotAssociation` | Does not have an association of a specific type |
| `scopeHasAnyAssociation` | Has any association |
| `scopeHasNotAnyAssociation` | Does not have any association |

##### G. Advanced Toggle Scopes

Scopes such as `isToggelAnyAssociation`, `isToggelAnyAlternate`, `isToggelAnyCrossSell`, `isToggelAnyUpSell` have been improved to respond correctly to values `1` and `0` while handling special cases coming from `post('scopeName')`. Additionally, scopes supporting **OR** (either direction) and **AND** (both directions) logic have been added for precise filtering control:

| Scope | Description |
|-------|-------------|
| `scopeIsToggelAnyAlternateOr` | Toggle alternatives: 1 = has alternative in either direction, 0 = has no alternative |
| `scopeIsToggelAnyAlternateAnd` | Toggle alternatives: 1 = has alternative in both directions, 0 = has no alternative |
| `scopeIsToggelAnyCrossSellOr` | Toggle complements: 1 = has complement in either direction, 0 = has no complement |
| `scopeIsToggelAnyCrossSellAnd` | Toggle complements: 1 = has complement in both directions, 0 = has no complement |
| `scopeIsToggelAnyUpSellOr` | Toggle newer versions: 1 = has newer version in either direction, 0 = has none |
| `scopeIsToggelAnyUpSellAnd` | Toggle newer versions: 1 = has newer version in both directions, 0 = has none |
| `scopeIsToggelAnyAssociationBothOr` | Toggle any association: 1 = has association in either direction, 0 = has no association |
| `scopeIsToggelAnyAssociationBothAnd` | Toggle any association: 1 = has association in both directions, 0 = has no association |

##### H. Internal Helper Functions

Internal functions have been added to apply conditions and scopes cleanly:

- `applyAssociationsRelationConditions($query, $type, $conditions, $scopes)`
- `normalizeAssociationsDirections($direction)`
- `prepareAssociationsCallbacks($options, $directions)`
- `buildAssociationsRelationQuery($query, $baseMethod, $directions, $callbacks, $relationOperator)`

These functions make the code more maintainable and extensible.

---

#### 2.2 `AssociationsHelper` – Integrated Helper Class

A new class `Nano\Shop\Classes\AssociationsHelper` has been created to simplify managing association filters in admin interfaces (e.g., `ListController`). Main functions:

**Available Functions**:

- `getAlternateFilterScopes($defaultValue = 0, $is_force = false)`: Returns an array of alternative filters (includes existence, non-existence, as parent, as target, and OR/AND toggle scopes).
- `getCrossSellFilterScopes($defaultValue = 0, $is_force = false)`: Complement filters.
- `getUpSellFilterScopes($defaultValue = 0, $is_force = false)`: Newer version filters.
- `getAnyAssociationFilterScopes($defaultValue = 0, $is_force = false)`: "Any association" filters (existence/non-existence with OR/AND logic).
- `getAllAssociationFilterScopes($defaultValue = 0, $is_force = false)`: Combines all the above filters.
- `injectAssociationScopes($filter, $types = null, $defaultValue = 0, $is_force = false)`: Injects filters into the filter tool (`$filter`). Supports `$types` with the following values:
  - `null` or `false`: Adds nothing.
  - `true`: Adds all filters.
  - String like `'any,alternate'`: Adds only the specified types.
  - Array like `['alternate', 'up_sell']`.
- `removeAssociationScopes($filter, $types = null)`: Removes specific filters from the filter tool.
- `isTypeEnabled($type)`: Checks whether a specific filter type is enabled according to settings.

> **Note:** Filters have been improved to include `_or` and `_and` scopes instead of a single `isToggel*` scope, providing precise control over filtering logic.

---

### 3. Method for Adding Filters to Product Admin Interfaces

Association filters can be added to the product management page via the `extendListFilterScopes` function in the `ShopProductsController`. Using `AssociationsHelper`, this becomes simple and flexible.

#### Example in `Plugin.php`:

```php
use Nano\Shop\Classes\AssociationsHelper;

public function boot()
{
    \ShopProductsController::extendListFilterScopes(function($filter) {
        $allowFilter = \Config::get('nano.shop::products.associations.allow_filter', false);
        if ($allowFilter && class_exists(AssociationsHelper::class)) {
            AssociationsHelper::injectAssociationScopes($filter, $allowFilter);
        }

        // Remove alternative filters if undesirable
        if (!\Config::get('nano.shop::allow_alternate_filter', true)) {
            AssociationsHelper::removeAssociationScopes($filter, 'alternate');
        }
    });
}
```

---

### 4. Control via Configuration File

A new key has been added in the `config.php` file for the `Nano.Shop` plugin to specify which types of association filters appear in the interface.

```php
// config.php
'products' => [
    'associations' => [
        // Can be:
        // - false: No association filters appear.
        // - true: All filters appear (any, alternate, cross_sell, up_sell).
        // - String like 'any,alternate': Only the mentioned types appear.
        'allow_filter' => env('NANO_SHOP_PRODUCTS_ASSOCIATIONS_ALLOW_FILTER', false),
    ],
],
```

This value can be set via the environment variable `NANO_SHOP_PRODUCTS_ASSOCIATIONS_ALLOW_FILTER` in the `.env` file:

```
NANO_SHOP_PRODUCTS_ASSOCIATIONS_ALLOW_FILTER=any,alternate
```

---

### 5. Application Examples

#### 5.1 Using Basic Scopes in Controller

```php
// Products that have alternatives (in either direction)
$products = Product::hasAlternate()->get();

// Products that have complements as parent only
$products = Product::hasCrossSellAsParent()->get();

// Products that have no associations
$products = Product::hasNotAnyAssociationBoth()->get();
```

#### 5.2 Using Advanced `scopeWhereAssociation`

```php
// Products that have alternatives or complements
$products = Product::whereAssociation([
    'type' => ['alternate', 'cross_sell'],
    'direction' => 'both'
])->get();

// Products that have active alternatives belonging to the current company
$products = Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => ['is_active' => 1],
    'scopes' => ['isCompany']
])->get();

// Products that have alternatives as parent or complements as target
$products = Product::whereAssociation([
    'direction' => ['parent', 'target'],
    'typePerDirection' => [
        'parent' => 'alternate',
        'target' => 'cross_sell'
    ]
])->get();

// Products that have associations in both directions (parent and target) simultaneously
$products = Product::whereAssociation([
    'direction' => 'both',
    'relationOperator' => 'and',
    'conditions' => ['is_active' => 1]
])->get();

// Using callback with direction distinction
$products = Product::whereAssociation([
    'type' => 'cross_sell',
    'callback' => function($query, $direction) {
        if ($direction === 'parent') {
            $query->where('sort_order', '>', 0);
        } else {
            $query->where('is_public', 1);
        }
    }
])->get();
```

#### 5.3 Combining with Other Conditions using `orWhere`

```php
$products = Product::where('price', '>', 100)
    ->orWhereAssociation(['type' => 'up_sell', 'direction' => 'parent'])
    ->get();
```

#### 5.4 Adding Filters to Admin Interface via `AssociationsHelper`

```php
// Add all filters
AssociationsHelper::injectAssociationScopes($filter, true);

// Add only alternative and newer version filters
AssociationsHelper::injectAssociationScopes($filter, 'alternate,up_sell');
```

---

### 6. Summary of Options in `AssociationsHelper`

| Function | Description |
|----------|-------------|
| `getAlternateFilterScopes()` | Array of alternative filters (6 filters: existence, non-existence, as parent, as target, OR toggle, AND toggle) |
| `getCrossSellFilterScopes()` | Array of complement filters (6 filters) |
| `getUpSellFilterScopes()` | Array of newer version filters (6 filters) |
| `getAnyAssociationFilterScopes()` | Array of any association filters (2 filters: OR toggle, AND toggle) |
| `getAllAssociationFilterScopes()` | All the above filters (20 filters) |
| `injectAssociationScopes()` | Inject filters into the filter tool |
| `removeAssociationScopes()` | Remove specific filters |
| `isTypeEnabled()` | Check if a specific type is enabled |

---

### 7. Summary of Key Changes

- **Added advanced options** in `scopeWhereAssociation` (conditions, scopes, typePerDirection, conditionsPerDirection, scopesPerDirection, relationOperator).
- **Improved negation logic** (when `has = false` and `direction = both`) to use `AND` between the two directions instead of `OR`.
- **Added `AssociationsHelper` class** to simplify integrating filters into the backend interface.
- **Added config settings** to control filter visibility.
- **Updated helper scopes** (`hasAssociation`, `hasNotAssociation`, ...) to remain compatible with older interfaces.
- **Fixed issue** in negation scopes (e.g., `hasNotAnyAssociationBoth`) that were returning incorrect results.
- **Expanded Toggle scopes** to include `OR` and `AND` logic with handling of `post('scopeName')`.

---

### 8. Added Value

- **For Developers**: Full control over association queries through flexible scopes covering all cases, with support for additional conditions on the associations table (activity, company, department, date...), and the ability to add ready-made interface filters in just a few lines of code.
- **For End Users**: Precise filtering of products based on their associations (alternatives, complements, newer versions) with multiple options (existence, non-existence, specific direction, OR/AND logic), improving the search and management experience.
- **For the System**: Modular and extensible design allowing new types of associations to be added in the future without needing to restructure the code.

---

### 9. Conclusion

This update represents a **qualitative leap** in how product associations are managed and queried within the NanoSoft system. By combining advanced scopes, precise condition control per direction, and the practical helper class `AssociationsHelper`, the system has become more powerful and ready for use in complex e-commerce projects.

**For Developers**: They can now build sophisticated admin interfaces with just a few lines of code, with full control over association queries via `scopeWhereAssociation` which supports advanced conditions, flexible logic for linking directions, and customizing types and callbacks per direction. The simple settings via the `config.php` file and environment variables also provide the ability to adapt filter visibility according to each project's needs.

**For End Users**: The new filters offer an enhanced search and management experience, allowing them to filter products based on their associations flexibly (existence, non-existence, specific direction, AND/OR logic), facilitating advanced searches and inventory management.

**For the System**: These updates have been designed with high modularity and extensibility, allowing new types of associations to be added in the future without needing to restructure the core code.

We recommend developers review the complete documentation and advanced examples to make the most of these features. With continued development, this system will remain capable of meeting the needs of diverse projects thanks to its flexibility and robust design.

**Reference Documentation:**
- [Basic Documentation for AssociationsModel Behavior](./docs/AssociationsModel/Docs-AssociationsModel-Behaviors-ar.md)
- [Advanced Practical Examples](./docs/AssociationsModel/Docs-AssociationsModel-Behaviors-Advenced-Examples-ar.md)

## 2026-3-22 - 2026-3-23

Comprehensive Update for Query Scopes in Reviews, Markables, and Proposals Systems

**Comprehensive Development of Query Scopes in Behaviors for:**

- Reviews (`ReviewRateable`)
- Favorites (`FavoriteableModel`)
- Likes (`LikeableModel`)
- Reactions (`ReactionableModel`)
- Bookmarks (`BookmarkableModel`)
- Proposals (`ProposalModel`)

Within the `Nano.Reviews`, `Nano.Markable`, and `Nano2.Proposals` packages

---

### 1. Introduction

An integrated development package has been implemented to improve the performance and flexibility of queries related to reviews, favorites, likes, reactions, bookmarks, and proposals. Developers no longer need to write complex queries or misuse `GROUP BY` and `LIMIT` inside subqueries. Instead, they now have a set of ready-made scopes that enable:

- Sorting by **the date of the latest item** (newest).
- Sorting by **the count of items** (most).
- Adding computed columns to `SELECT` (such as `last_created_at` or `count`) without affecting the sorting.
- The ability to filter by **value** (`value`) in systems that support it (e.g., likes and reactions).
- The ability to filter by **type** (`type`) in the proposals system.

Existing scopes in all mentioned behaviors have been restructured, and new scopes have been added to ensure correct results and high performance, while providing a unified and easy-to-use programming interface.

This update aims to meet advanced use cases such as:
- Sorting products by **highest rated** (based on average ratings).
- Sorting products by **most recently rated** (latest review date).
- Sorting articles by **most liked** (differentiating between `like` and `dislike`).
- Sorting content by **most reacted** (specific reactions like `wow`, `sad`).
- Obtaining a list of products with the **most reports** of a specific type (`reports` or `complaints`).
- Adding a computed column containing the **latest review date** or **review count** to the query result without affecting sorting.

---

### 2. Developed Components

#### 2.1 Behaviors and Added Scopes

| Behavior | Added Scopes | Description |
|----------|--------------|-------------|
| `ReviewRateable` | `scopeSortByCreatedAtReviews`, `scopeAddSortByCreatedAtReviews`, `scopeWithSortByCreatedAtReviews`<br>`scopeSortByAvgRating`, `scopeAddSortByAvgRating`, `scopeWithSortByAvgRating`<br>`scopeSortByCountReviews`, `scopeAddSortByCountReviews`, `scopeWithSortByCountReviews` | Sorting and adding computed columns for reviews by date (latest review), by average rating, and by review count. |
| `FavoriteableModel` | `scopeSortByCreatedAtFavorites`, `scopeAddSortByCreatedAtFavorites`, `scopeWithSortByCreatedAtFavorites`<br>`scopeSortByCountFavorites`, `scopeAddSortByCountFavorites`, `scopeWithSortByCountFavorites` | Sorting and adding computed columns for favorites by date and count. |
| `LikeableModel` | `scopeSortByCreatedAtLikes`, `scopeAddSortByCreatedAtLikes`, `scopeWithSortByCreatedAtLikes`<br>`scopeSortByCountLikes`, `scopeAddSortByCountLikes`, `scopeWithSortByCountLikes` | Sorting and adding computed columns for likes by date and count, with the ability to filter by value (`like`/`dislike`) using the `$value` parameter. |
| `ReactionableModel` | `scopeSortByCreatedAtReactions`, `scopeAddSortByCreatedAtReactions`, `scopeWithSortByCreatedAtReactions`<br>`scopeSortByCountReactions`, `scopeAddSortByCountReactions`, `scopeWithSortByCountReactions` | Sorting and adding computed columns for reactions by date and count, with the ability to filter by value (e.g., `wow`, `sad`, `angry`). |
| `BookmarkableModel` | `scopeSortByCreatedAtBookmarks`, `scopeAddSortByCreatedAtBookmarks`, `scopeWithSortByCreatedAtBookmarks`<br>`scopeSortByCountBookmarks`, `scopeAddSortByCountBookmarks`, `scopeWithSortByCountBookmarks` | Sorting and adding computed columns for bookmarks by date and count, with the ability to filter by value (e.g., `favorite`, `read_later`). |
| `ProposalModel` | `scopeSortByCreatedAtProposals`, `scopeAddSortByCreatedAtProposals`, `scopeWithSortByCreatedAtProposals`<br>`scopeSortByCountProposals`, `scopeAddSortByCountProposals`, `scopeWithSortByCountProposals` | Sorting and adding computed columns for proposals by date and count, with the ability to filter by type (`proposals`, `reports`, `complaints`) using the `$type` parameter. |

---

### 3. Details of Code Updates

#### 3.1 Fixing Common Errors in Subqueries

Old scopes (in some behaviors) relied on using `GROUP BY` and `LIMIT` inside correlated subqueries to obtain an aggregated value (`MAX`, `AVG`, `COUNT`). This approach leads to incorrect and random results because `GROUP BY` creates multiple groups, and then `LIMIT 1` selects only the first group arbitrarily.

**New Approach**:
- Use an aggregate function (`MAX`, `AVG`, `COUNT`) directly in the subquery without `GROUP BY` or `LIMIT`. The aggregate function in a correlated subquery automatically operates on all rows matching the join condition.
- Write the full table name before each field to avoid ambiguity.
- Add additional conditions (such as `value`, `type`, `approved`) via `where` inside the subquery.

Example of the correct scope for sorting by latest review (`ReviewRateable`):

```php
public function scopeSortByCreatedAtReviews(Builder $query, $orderDirection = 'DESC')
{
    $orderDirection = strtoupper($orderDirection) === 'ASC' ? 'ASC' : 'DESC';
    
    $subQuery = Review::selectRaw('MAX(nano_reviews_reviews.created_at)')
        ->whereColumn('nano_reviews_reviews.reviewrateable_id', $this->model->getTable() . '.id')
        ->where('nano_reviews_reviews.reviewrateable_type', $this->model->getMorphClass());
    
    return $query->orderBy($subQuery, $orderDirection);
}
```

#### 3.2 Unifying the Scope Interface

The function signatures (parameters) have been unified across all behaviors to be as follows:
- `$orderDirection`: Sorting direction (default `DESC`).
- `$filter` (or `$value`/`$type`): An optional value to filter results by value (in `Likeable`, `Reactionable`, `Bookmarkable`) or by type (in `ProposalModel`). In `ReviewRateable`, a parameter `$approved` can be added to filter only approved reviews (optional).
- `$columnName`: The name of the added column in the case of `AddSortBy` and `WithSortBy` (a suitable default like `last_created_at_reviews` or `avg_rating`).

This unification makes it easier for developers to remember and use scopes across different behaviors.

#### 3.3 Handling Default Values in `LikeableModel`

Since `LikeableModel` relies on the `value` field to distinguish between like and dislike, the logic for obtaining the default value from the `Like` class via `Like::getDefaultValue()` has been integrated when no value is passed.

```php
if (is_null($value) && $defaultValue = Like::getDefaultValue()) {
    $value = $defaultValue;
}
```

#### 3.4 Adding Average Rating Scopes (`ReviewRateable`)

Special scopes for average rating (`avg_rating`) have been added because they are a common requirement in rating systems:

```php
public function scopeSortByAvgRating(Builder $query, $orderDirection = 'DESC', $columnName = 'avg_rating')
{
    $orderDirection = strtoupper($orderDirection) === 'ASC' ? 'ASC' : 'DESC';
    
    $subQuery = Review::selectRaw('AVG(rating)')
        ->whereColumn('reviewrateable_id', $this->model->getTable() . '.id')
        ->where('reviewrateable_type', $this->model->getMorphClass())
        ->where('approved', true); // Optional: can be made a parameter
    
    return $query->orderBy($subQuery, $orderDirection);
}
```

#### 3.5 Scopes for Adding Computed Columns

In addition to sorting scopes (`SortBy`), `AddSortBy` scopes have been provided that only add the computed column to `SELECT`, and `WithSortBy` scopes that both add the column and sort. This gives developers full flexibility in controlling the query.

Example of `AddSortByCountFavorites`:

```php
public function scopeAddSortByCountFavorites(Builder $query, $orderDirection = 'DESC', $value = null, $columnName = 'count_favorites')
{
    $subQuery = Favorite::selectRaw('COUNT(nano_markable_favorites.id)')
        ->whereColumn('nano_markable_favorites.markable_id', $this->model->getTable() . '.id')
        ->where('nano_markable_favorites.markable_type', $this->model->getMorphClass());

    if ($value) {
        $subQuery->where('nano_markable_favorites.value', $value);
    }

    return $query->addSelect([$columnName => $subQuery]);
}
```

---

### 4. Practical Examples

#### 4.1 Reviews System (`ReviewRateable`)

```php
// Fetch products sorted by highest rated (average rating)
$products = Product::sortByAvgRating('desc')->get();

// Fetch products with an added column for the latest review date
$products = Product::addSortByCreatedAtReviews('desc', 'last_review_date')->get();

// Fetch products sorted by review count with the column added
$products = Product::withSortByCountReviews('desc', 'reviews_count')->get();
```

#### 4.2 Favorites System (`FavoriteableModel`)

```php
// Most favorited
$products = Product::sortByCountFavorites('desc')->get();

// Latest favorited with date column added
$products = Product::withSortByCreatedAtFavorites('desc', 'last_favorite_at')->get();
```

#### 4.3 Likes System (`LikeableModel`)

```php
// Most liked (likes only)
$articles = Article::sortByCountLikes('desc', 'like')->get();

// Most disliked
$articles = Article::sortByCountLikes('desc', 'dislike')->get();

// Add likes count column
$articles = Article::addSortByCountLikes('desc', 'like', 'likes_count')->get();
```

#### 4.4 Reactions System (`ReactionableModel`)

```php
// Most reacted with type 'wow'
$posts = Post::sortByCountReactions('desc', 'wow')->get();

// Latest reactions (all reactions)
$posts = Post::sortByCreatedAtReactions('desc')->get();
```

#### 4.5 Bookmarks System (`BookmarkableModel`)

```php
// Most bookmarked of type 'favorite'
$products = Product::sortByCountBookmarks('desc', 'favorite')->get();

// Add last bookmark date column
$products = Product::addSortByCreatedAtBookmarks('desc', null, 'last_bookmark')->get();
```

#### 4.6 Proposals System (`ProposalModel`)

```php
// Most reported of type 'reports'
$products = Product::sortByCountProposals('desc', 'reports')->get();

// Latest proposal of type 'complaints'
$products = Product::sortByCreatedAtProposals('desc', 'complaints')->get();
```

#### 4.7 Combining Scopes

```php
// Fetch active products, sorted by most favorited, with favorites count column added
$products = Product::isActive()
    ->withSortByCountFavorites('desc', 'favorites_count')
    ->get();
```

---

### 5. Added Value

- **For Developers**: An integrated set of ready-made scopes saves time and reduces errors. Developers no longer need to write complex subqueries or worry about result correctness. The unified interface facilitates learning and usage across different behaviors.
- **For End Users**: The ability to present intelligently sorted lists (most popular, most recent activity, highest rated) improves user experience and increases application effectiveness.
- **For the System**: Better performance through optimized SQL queries, eliminating inefficient queries that misused `GROUP BY` and `LIMIT`. Correlated subqueries now work with high efficiency.
- **Flexibility**: The ability to filter by value or type allows building complex systems such as distinguishing likes from dislikes, categorizing reports, or handling multiple reactions.
- **Scalability**: Adding new scopes for any future behavior follows the same pattern, ensuring code consistency and ease of maintenance.

---

### 6. Conclusion

This update represents a qualitative leap in managing queries for reviews, markables, and proposals within the system. By restructuring and unifying scopes across all relevant behaviors, developers can now build applications relying on advanced sorting and filtering with ease and safety. As development continues, this system will remain capable of meeting the needs of diverse projects thanks to its flexibility and robust design.

## 2026-3-23 - 2026-3-24

### Comprehensive Update for Query Scopes in `ReviewRateable` Behavior

**Complete Development of Query Scopes and Helper Functions within the `Nano.Reviews` Package**

An integrated development package has been implemented aimed at improving the performance and flexibility of queries related to the reviews system in NanoSoft applications. Developers no longer need to write complex queries or incorrectly rely on `GROUP BY` and `LIMIT` within subqueries. Instead, they now have a set of ready-made scopes that enable:

- Ordering by **number of reviews**.
- Ordering by **average rating**.
- Ordering by **latest review date** (most recent).
- Ordering by **earliest review date** (oldest).
- Adding calculated columns to `SELECT` (e.g., `reviews_count`, `avg_rating`, `latest_review_at`) without affecting the ordering.
- Filtering objects that a specific user has rated (or has not rated).
- Filtering objects that have reviews (with a minimum count) or that have no reviews.
- Adding the column `is_rated_by_user` which indicates whether the current user has rated the object.
- Advanced statistical functions to get total ratings, average rating, distribution of ratings by user type, and the list of users who rated the object.

Existing scopes have been restructured and new scopes added, while maintaining backward compatibility via wrapper functions marked as `@deprecated`. All scopes and helper functions have been moved to an independent trait `ReviewScopesAndHelpers` to facilitate maintenance and reuse.

---

### 1. Developed Components

| Behavior | Component | Description |
|----------|-----------|-------------|
| `ReviewRateable` | `ReviewScopesAndHelpers` (trait) | Contains all advanced scopes and helper functions for the reviews system. |

#### New and Improved Scopes

| Category | Scope | Description |
|----------|-------|-------------|
| **Number of Reviews** | `scopeAddCountReviews` | Add a review count column to `SELECT` (without ordering). |
| | `scopeSortByCountReviews` | Order results by number of reviews (without adding the column). |
| | `scopeWithCountReviews` | Add the column and ordering together. |
| **Average Rating** | `scopeAddAvgRating` | Add an average rating column (with rounding). |
| | `scopeSortByAvgRating` | Order by average rating. |
| | `scopeWithAvgRating` | Add the column and ordering together. |
| **Latest Review Date** | `scopeAddLatestReview` | Add a column with the latest review date. |
| | `scopeSortByLatestReview` | Order by the latest review date. |
| | `scopeWithLatestReview` | Add the column and ordering together. |
| **Earliest Review Date** | `scopeAddEarliestReview` | Add a column with the earliest review date. |
| | `scopeSortByEarliestReview` | Order by the earliest review date. |
| | `scopeWithEarliestReview` | Add the column and ordering together. |
| **Filtering by User** | `scopeRatedByUser` | Filter objects that a specific user has rated. |
| | `scopeNotRatedByUser` | Filter objects that a specific user has not rated. |
| **Filtering by Existence of Reviews** | `scopeHasRatings` | Filter objects that have reviews (with a minimum count). |
| | `scopeHasNoRatings` | Filter objects that have no reviews. |
| **Status Column for User** | `scopeWithIsRatedByUser` | Add the `is_rated_by_user` column. |
| **Special Scopes** | `scopeTopRated` | Retrieve the highest-rated objects (by average rating). |
| | `scopeTopRatedByCount` | Retrieve the most-reviewed objects (by number of reviews). |

#### Helper Statistical Functions

| Function | Description |
|----------|-------------|
| `getTotalRatings` | Total number of ratings (sum of `rating`) with filtering options. |
| `getAverageRating` | Average rating with filtering options. |
| `getRatingsCountByType` | Distribution of ratings by user type (`user_type`). |
| `getRatersUsers` | List of users who rated the object. |

All functions and scopes support filtering options: `$onlyActive`, `$onlyApproved`, `$withTrashed`, `$isPositive`, `$ratingValue`.

---

### 2. Details of Code Updates

#### 2.1 Fixing Subquery Errors
Old scopes in the original behavior relied on using `GROUP BY` and `LIMIT` inside subqueries to get aggregated values (e.g., `AVG`, `COUNT`, `MAX`). This approach leads to incorrect and unpredictable results, because `GROUP BY` creates multiple groups and then `LIMIT 1` selects only the first group.

**New Approach**:
- Use an aggregate function directly in the subquery without `GROUP BY` or `LIMIT`. An aggregate function in a related subquery automatically works on all rows matching the join condition.
- Write the full table name before each field to avoid ambiguity.
- Add additional conditions (e.g., `approved`, `is_positive`, `rating`) via `where` inside the subquery.

Example of a correct query for ordering by number of reviews:

```php
$subQuery = Review::selectRaw('COUNT(' . $table . '.id)')
    ->whereColumn($table . '.reviewrateable_id', $this->model->getTable() . '.id')
    ->where($table . '.reviewrateable_type', $this->model->getMorphClass())
    ->where(...);
```

#### 2.2 Standardizing Scope Interface
Function signatures have been standardized to include:
- `$orderDirection`: Sorting direction (default `DESC`).
- `$columnName`: Name of the added column (appropriate default like `reviews_count`, `avg_rating`, `latest_review_at`).
- `$onlyActive`: Filter by `is_active`.
- `$withTrashed`: Include soft-deleted records.
- `$onlyApproved`: Filter by `approved` (approved reviews only).
- `$isPositive`: Filter by `is_positive` (positive/negative rating).
- `$ratingValue`: Filter by rating value (e.g., 5 stars).
- In date scopes, a `$field` parameter was added to specify the temporal field used (`created_at`, `updated_at`).

#### 2.3 Supporting `withTrashed` (Soft-Deleted Reviews)
The `$withTrashed` parameter was added to all scopes and statistical functions. When `true` is passed, soft-deleted review records are included in the subquery. This requires that the `Review` model uses the `SoftDeletes` trait.

#### 2.4 Moving Scopes to a Separate Trait
To facilitate maintenance and reuse, all scopes and helper functions have been moved to a new trait:  
`Nano\Reviews\Behaviors\ReviewRateable\ReviewScopesAndHelpers`

This trait is then used inside the `ReviewRateable` behavior via `use`.

#### 2.5 Backward Compatibility
Old scopes have been retained as wrapper functions in the main class, with an `@deprecated` comment to guide developers toward the new alternatives.

**List of Supported Old Functions:**
- `scopeSortByRating` → `scopeSortByAvgRating`
- `scopeWithRating` → `scopeWithAvgRating`
- `scopeAddSortByRating` → `scopeAddAvgRating`
- `scopeWithSortByCountReviews` → `scopeWithCountReviews`
- `scopeSortByCountReviewsOld` → `scopeSortByCountReviews`
- `scopeSortByCreatedAtReviews` → `scopeSortByLatestReview`
- `scopeAddSortByCreatedAtReviews` → `scopeAddLatestReview`
- `scopeWithSortByCreatedAtReviews` → `scopeWithLatestReview`

---

### 3. Practical Examples

#### 3.1 Ordering by Number of Reviews
```php
// Most reviewed products (by number of reviews)
$products = Product::sortByCountReviews('DESC', 'reviews_count')->take(10)->get();

// Add a review count column
$products = Product::addCountReviews()->get();
foreach ($products as $product) {
    echo $product->reviews_count;
}
```

#### 3.2 Ordering by Average Rating
```php
$products = Product::sortByAvgRating('DESC', 'avg_rating', null, false, true)->get(); // approved reviews only
```

#### 3.3 Adding the `is_rated_by_user` Column for the Current User
```php
$products = Product::withIsRatedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->is_rated_by_user ? 'You rated' : 'You did not rate';
}
```

#### 3.4 Filtering Products that the Current User Has Rated
```php
$ratedProducts = Product::ratedByUser()->get();
```

#### 3.5 Filtering by Positive Reviews Only
```php
$products = Product::hasRatings(1, true, false, true, true)->get(); // at least one positive review
```

#### 3.6 Statistics
```php
$product = Product::find(1);
echo "Total ratings: " . $product->getTotalRatings(true, false, true);
echo "Average rating: " . $product->getAverageRating(true, false, true);
print_r($product->getRatingsCountByType(true, false, true)->toArray());
$users = $product->getRatersUsers(true, false, true);
```

#### 3.7 Combining Advanced Scopes
```php
// Products that have at least 5 approved reviews, ordered by average rating
$products = Product::hasRatings(5, true, false, true)
    ->sortByAvgRating('DESC', 'avg_rating')
    ->get();
```

---

### 4. Added Value

- **For Developers**: An integrated set of ready-made scopes saves time and reduces errors. Developers no longer need to write complex subqueries or worry about result correctness. The unified interface makes learning and usage easy across different behaviors.
- **For End Users**: The ability to present intelligently ordered lists (highest rated, latest reviewed) improves the user experience and increases application effectiveness.
- **For the System**: Better performance through optimized SQL queries, eliminating inefficient queries that incorrectly used `GROUP BY` and `LIMIT`.
- **Flexibility**: The ability to filter by `approved`, `is_positive`, `ratingValue` allows building advanced rating systems (e.g., filter positive reviews only, or 5-star ratings).
- **Scalability**: Adding new scopes for any future behavior follows the same pattern, ensuring code consistency and ease of maintenance.

---

### 5. Conclusion

This update represents a qualitative shift in managing review queries within NanoSoft applications. By restructuring scopes and moving them to an independent trait, along with adding advanced functions for ordering, filtering, and statistics, developers can now build sophisticated rating systems with ease and high performance. Backward compatibility ensures a smooth transition for existing projects, while the new additions open up vast possibilities for rich user experiences.

---

**Note**: For more details about each scope and how to use it, please refer to the `ReviewRateable` behavior documentation file or review the examples provided above.

See [docs/ReviewRateable/Docs-ReviewRateable-en.md](./docs/ReviewRateable/Docs-ReviewRateable-en.md)

See [docs/ReviewRateable/Docs-ReviewRateable-Advenced-Examples-en.md](./docs/ReviewRateable/Docs-ReviewRateable-Advenced-Examples-en.md)

## 2026-3-24 - 2026-3-25

### Comprehensive Update of Query Scopes in `FollowableModel` Behavior

**Complete development of query scopes and helper functions within the `Nano.Follows` package**

An integrated development package has been implemented aimed at improving the performance and flexibility of queries related to the Follows system in NanoSoft applications. Developers no longer need to write complex queries or rely on incorrect usage of `GROUP BY` and `LIMIT` within subqueries; instead, they have a set of ready‑made scopes that enable:

- Sorting by **number of follows** (most followed).
- Sorting by **latest follow date** (most recently active).
- Sorting by **earliest follow date** (first follow).
- Adding computed columns to `SELECT` (such as `follows_count` or `latest_follow_at`) without affecting ordering.
- Filtering objects that are followed (or not followed) by a specific user.
- Filtering objects that have (or do not have) followers, with the ability to specify a minimum count.
- Adding the `is_followed_by_user` column that indicates whether the current user follows the object.
- Advanced statistical functions to get total followers, their distribution by user type, and the list of follower users.

Existing scopes have been restructured and new scopes added, while preserving backward compatibility via wrapper functions marked `@deprecated`. All scopes and helper functions have been moved to a standalone trait `FollowableScopesAndHelpers` to facilitate maintenance and reuse.

---

### 1. Developed Components

| Behavior | Component | Description |
|----------|-----------|-------------|
| `FollowableModel` | `FollowableScopesAndHelpers` (trait) | Contains all advanced scopes and helper functions for the follow system. |

#### New and Improved Scopes

| Category | Scope | Description |
|----------|-------|-------------|
| **Follow Count** | `scopeAddCountFollows` | Adds a follow count column to `SELECT` (without sorting). |
| | `scopeSortByCountFollows` | Sorts results by follow count (without adding the column). |
| | `scopeWithCountFollows` | Adds the column and sorts together. |
| **Latest Date** | `scopeAddLatestFollow` | Adds a column with the latest follow date. |
| | `scopeSortByLatestFollow` | Sorts by the latest follow date. |
| | `scopeWithLatestFollow` | Adds the column and sorts together. |
| **Earliest Date** | `scopeAddEarliestFollow` | Adds a column with the earliest follow date. |
| | `scopeSortByEarliestFollow` | Sorts by the earliest follow date. |
| | `scopeWithEarliestFollow` | Adds the column and sorts together. |
| **Filtering by User** | `scopeFollowedByUser` | Filters objects followed by a specific user. |
| | `scopeNotFollowedByUser` | Filters objects not followed by a specific user. |
| | `scopeWhereHasFollow` (legacy) | For backward compatibility; `FollowedByUser` is recommended. |
| | `scopeWhereHasFollows` (legacy) | Alias of the legacy one. |
| **Filtering by Follower Presence** | `scopeHasFollowers` | Filters objects that have followers (with a minimum count). |
| | `scopeHasNoFollowers` | Filters objects that have no followers. |
| **User Status Column** | `scopeWithIsFollowedByUser` | Adds the `is_followed_by_user` column indicating whether the current user follows. |
| **Special Scopes** | `scopeTopFollowed` | Retrieves the most followed objects (with a limit). |
| | `scopeSortByAcceptedFollowsCount` | Shortcut to sort by accepted follows only. |
| | `scopeSortByActiveFollowsCount` | Shortcut to sort by active follows only. |

#### New Helper Functions

| Function | Description |
|----------|-------------|
| `getFollowersCountByType` | Returns the number of followers grouped by `user_type`. |
| `getTotalFollowers` | Total number of followers with filtering options. |
| `getFollowersUsers` | Returns the collection of users who follow the object. |

---

### 2. Details of Code Updates

#### 2.1 Fixing Subquery Errors
Some previous queries (in legacy scopes not present in this behavior) might have used `GROUP BY` and `LIMIT` inside subqueries to obtain aggregated values (e.g., `MAX` or `COUNT`). This approach leads to incorrect and unpredictable results. In the current update, subqueries are built correctly using direct aggregate functions without `GROUP BY` or `LIMIT`, with the full table name prefixed to each field to avoid ambiguity.

Example of the correct query for sorting by follow count:

```php
$subQuery = Follow::selectRaw('COUNT(' . $followTable . '.id)')
    ->whereColumn($followTable . '.followable_id', $this->model->getTable() . '.id')
    ->where($followTable . '.followable_type', $this->model->getMorphClass())
    ->where(...);
```

#### 2.2 Unifying Scope Signatures
Function signatures have been unified to include:
- `$orderDirection`: sort direction (default `DESC`).
- `$columnName`: name of the added column (with a sensible default such as `follows_count`, `latest_follow_at`).
- `$onlyAccepted` / `$onlyActive` / `$withTrashed`: optional filtering parameters.
- For date scopes, an additional `$field` parameter has been added to specify the timestamp field to use (`created_at`, `updated_at`, `accepted_at`, `re_follow_at`).

#### 2.3 Supporting `withTrashed` (Soft‑Deleted Follows)
The `$withTrashed` parameter has been added to all scopes and statistical functions. When `true` is passed, soft‑deleted follow records are included in the subquery. This requires that the `Follow` model uses the `SoftDeletes` trait.

#### 2.4 Moving Scopes to a Separate Trait
To facilitate maintenance and reuse, all scopes and helper functions have been moved to a new trait:  
`Nano\Follows\Behaviors\FollowableModel\FollowableScopesAndHelpers`

This trait is then used inside the `FollowableModel` behavior via `use`.

#### 2.5 Backward Compatibility
Legacy scopes (such as `addSortByCountFollows`, `withSortByLatestAtFollows`, etc.) have been retained as wrapper functions in the main class, with a `@deprecated` annotation to guide developers toward the new alternatives. This ensures that existing code relying on these names does not break.

---

### 3. Practical Examples

#### 3.1 Sorting by Follow Count
```php
// Most followed products (accepted only)
$products = Product::topFollowed(10)->get();

// Descending order by number of active follows (including soft‑deleted)
$products = Product::sortByCountFollows('DESC', 'follows_count', null, true, true)->get();
```

#### 3.2 Adding a Follow Count Column
```php
$products = Product::addCountFollows()->get();
foreach ($products as $product) {
    echo $product->follows_count;
}
```

#### 3.3 Filtering by Current User
```php
$myFollowed = Product::followedByUser()->get();
```

#### 3.4 Filtering by Follower Presence
```php
// Products with at least 5 accepted followers
$products = Product::hasFollowers(5, true)->get();
```

#### 3.5 Adding the `is_followed_by_user` Column
```php
$products = Product::withIsFollowedByUser()->paginate(20);
foreach ($products as $product) {
    if ($product->is_followed_by_user) echo "Followed";
}
```

#### 3.6 Sorting by Latest Follow (Custom Field)
```php
$products = Product::sortByLatestFollow('DESC', 'latest_follow_at', true, null, false, 'accepted_at')->get();
```

#### 3.7 Statistics
```php
$product = Product::find(1);
echo $product->getTotalFollowers(true, true); // Accepted and active
print_r($product->getFollowersCountByType(true, true)->toArray());
$users = $product->getFollowersUsers(true, true);
```

---

### 4. Added Value

- **For Developers**: A comprehensive set of ready‑made scopes saves time and reduces errors. Developers no longer need to write complex subqueries or worry about result correctness. The unified interface makes learning and usage easier across different behaviors.
- **For End Users**: Ability to present intelligently sorted lists (most followed, most recently active) improves user experience and increases the effectiveness of social applications.
- **For the System**: Better performance through optimized SQL queries, eliminating inefficient queries that incorrectly used `GROUP BY` and `LIMIT`. The correlated subqueries now operate with high efficiency.
- **Flexibility**: Filtering by acceptance, activity, and soft‑deleted follows allows building complex systems such as "pending follows" or "follow statistics over time."
- **Scalability**: Adding new scopes for any future behavior follows the same pattern, ensuring code consistency and ease of maintenance.

---

### 5. Conclusion

This update represents a qualitative leap in managing follow‑system queries within NanoSoft applications. By restructuring scopes and moving them to a standalone trait, while adding advanced sorting, filtering, and statistical functions, developers can now build sophisticated follow systems with ease and high performance. Backward compatibility ensures a smooth transition for existing projects, while the new additions open wide possibilities for rich user experiences.

---

**Note**: For more details about each scope and its usage, please refer to the documentation file `Docs-FollowableModel-en.md` or review the examples provided above.

See [docs/FollowableModel/Docs-FollowableModel-en.md](./docs/FollowableModel/Docs-FollowableModel-en.md)

See [docs/FollowableModel/Docs-FollowableModel-Advenced-Examples-en.md](./docs/FollowableModel/Docs-FollowableModel-Advenced-Examples-en.md)

## 2026-3-26 - 2026-3-27

### Comprehensive Update for Query Scopes in `FavoriteableModel` Behavior

**Complete Development of Query Scopes and Helper Functions within the `Nano.Markable` Package**

An integrated development package has been implemented aimed at improving the performance and flexibility of queries related to the favorites system in NanoSoft applications. Developers no longer need to write complex queries or incorrectly rely on `GROUP BY` and `LIMIT` within subqueries. Instead, they now have a set of ready-made scopes that enable:

- Ordering by **favorite count**.
- Ordering by **latest favorite date** (most recent).
- Ordering by **earliest favorite date** (oldest).
- Adding calculated columns to `SELECT` (e.g., `favorites_count`, `latest_favorite_at`) without affecting the ordering.
- Filtering objects that a specific user has favorited (or has not favorited).
- Filtering objects that have favorites (with a minimum count) or that have no favorites.
- Adding the column `is_favorited_by_user` which indicates whether the current user has favorited the object.
- Advanced statistical functions to get total favorites, distribution of favorites by user type, and the list of users who favorited the object.

Existing scopes have been restructured and new scopes added, while maintaining backward compatibility via wrapper functions marked as `@deprecated`. All scopes and helper functions have been moved to an independent trait `FavoriteScopesAndHelpers` to facilitate maintenance and reuse.

---

### 1. Developed Components

| Behavior | Component | Description |
|----------|-----------|-------------|
| `FavoriteableModel` | `FavoriteScopesAndHelpers` (trait) | Contains all advanced scopes and helper functions for the favorites system. |

#### New and Improved Scopes

| Category | Scope | Description |
|----------|-------|-------------|
| **Favorite Count** | `scopeAddCountFavorites` | Add a favorite count column to `SELECT` (without ordering). |
| | `scopeSortByCountFavorites` | Order results by favorite count (without adding the column). |
| | `scopeWithCountFavorites` | Add the column and ordering together. |
| **Latest Favorite Date** | `scopeAddLatestFavorite` | Add a column with the latest favorite date. |
| | `scopeSortByLatestFavorite` | Order by the latest favorite date. |
| | `scopeWithLatestFavorite` | Add the column and ordering together. |
| **Earliest Favorite Date** | `scopeAddEarliestFavorite` | Add a column with the earliest favorite date. |
| | `scopeSortByEarliestFavorite` | Order by the earliest favorite date. |
| | `scopeWithEarliestFavorite` | Add the column and ordering together. |
| **Filtering by User** | `scopeFavoritedByUser` | Filter objects that a specific user has favorited. |
| | `scopeNotFavoritedByUser` | Filter objects that a specific user has not favorited. |
| **Filtering by Existence of Favorites** | `scopeHasFavorites` | Filter objects that have favorites (with a minimum count). |
| | `scopeHasNoFavorites` | Filter objects that have no favorites. |
| **Status Column for User** | `scopeWithIsFavoritedByUser` | Add the `is_favorited_by_user` column. |
| **Special Scopes** | `scopeTopFavorited` | Retrieve the objects with the most favorites (by favorite count). |

#### Helper Statistical Functions

| Function | Description |
|----------|-------------|
| `getTotalFavorites` | Total number of favorites (sum of `COUNT`) with filtering options. |
| `getFavoritesCountByType` | Distribution of favorites by user type (`user_type`). |
| `getFavoritersUsers` | List of users who have favorited the object. |

All functions and scopes support filtering options: `$value` (to filter favorite type, e.g., `like`/`favorite`).

---

### 2. Details of Code Updates

#### 2.1 Fixing Subquery Errors
Old scopes in the original behavior relied on using `GROUP BY` and `LIMIT` inside subqueries to get aggregated values (e.g., `COUNT`, `MAX`). This approach leads to incorrect and unpredictable results.

**New Approach**:
- Use an aggregate function directly in the subquery without `GROUP BY` or `LIMIT`.
- Write the full table name before each field to avoid ambiguity.
- Add additional conditions (e.g., `value`) via `where` inside the subquery.

#### 2.2 Standardizing Scope Interface
Function signatures have been standardized to include:
- `$orderDirection`: Sorting direction (default `DESC`).
- `$columnName`: Name of the added column (appropriate default like `favorites_count`, `latest_favorite_at`).
- `$value`: Filter by favorite value (e.g., `like`).
- Other optional parameters (`$onlyActive`, `$withTrashed`) to prepare for future expansion.

#### 2.3 Supporting `$value` to Distinguish Favorite Types
The `$value` parameter has been added to all scopes and statistical functions, allowing filtering for a specific favorite type (e.g., `like`, `favorite`, `bookmark`).

#### 2.4 Moving Scopes to a Separate Trait
To facilitate maintenance and reuse, all scopes and helper functions have been moved to a new trait:  
`Nano\Markable\Behaviors\FavoriteableModel\FavoriteScopesAndHelpers`

#### 2.5 Backward Compatibility
Old scopes have been retained as wrapper functions in the main class, with an `@deprecated` comment. The old functions (`scopeWhereHasFavorite`, `scopeWhereHasFavorites`, `scopeWhereHasUserFavorite`, `scopeWhereHasUserFavorites`) have also been improved to support the `$isForceUser` parameter for controlling query behavior when no user is present, with the default value read from settings.

**List of Supported Old Functions:**
- `scopeSortByCountFavoritesOld` → `scopeSortByCountFavorites`
- `scopeWithSortByCountFavorites` → `scopeWithCountFavorites`
- `scopeAddSortByCountFavorites` → `scopeAddCountFavorites`
- `scopeSortByCreatedAtFavorites` → `scopeSortByLatestFavorite`
- `scopeAddSortByCreatedAtFavorites` → `scopeAddLatestFavorite`
- `scopeWithSortByCreatedAtFavorites` → `scopeWithLatestFavorite`
- `scopeWhereHasFavorite` → `scopeFavoritedByUser` (with improvements)
- `scopeWhereHasFavorites` → `scopeFavoritedByUser`
- `scopeWhereHasUserFavorite` → `scopeFavoritedByUser`
- `scopeWhereHasUserFavorites` → `scopeFavoritedByUser`

---

### 3. Practical Examples

#### 3.1 Ordering by Favorite Count
```php
// Most favorited products
$products = Product::topFavorited(10)->get();

// Add favorite count column with ordering
$products = Product::withCountFavorites('DESC', 'favorites_count')->get();
```

#### 3.2 Adding the `is_favorited_by_user` Column for the Current User
```php
$products = Product::withIsFavoritedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->is_favorited_by_user ? 'Favorite' : 'Not favorite';
}
```

#### 3.3 Filtering Products that the Current User Has Favorited
```php
$myFavorites = Product::favoritedByUser()->get();
```

#### 3.4 Filtering by Favorite Type (e.g., `like`)
```php
$likedProducts = Product::hasFavorites(1, 'like')->get();
```

#### 3.5 Statistics
```php
$product = Product::find(1);
echo "Number of favorites: " . $product->getTotalFavorites();
print_r($product->getFavoritesCountByType()->toArray());
$users = $product->getFavoritersUsers();
```

#### 3.6 Combining Advanced Scopes
```php
// Products that have more than 5 favorites of type 'like', ordered by latest favorite
$products = Product::hasFavorites(5, 'like')
    ->sortByLatestFavorite('DESC', 'latest_favorite_at')
    ->get();
```

---

### 4. Added Value

- **For Developers**: An integrated set of ready-made scopes saves time and reduces errors. The unified interface makes learning and usage easy.
- **For End Users**: The ability to present intelligently ordered lists (most favorited, latest favorited) improves the user experience.
- **For the System**: Better performance through optimized SQL queries, eliminating inefficient queries that incorrectly used `GROUP BY` and `LIMIT`.
- **Flexibility**: The ability to filter by `value` allows building multi-type systems (likes, favorites, bookmarks).
- **Scalability**: Adding new scopes for any future behavior follows the same pattern, ensuring code consistency and ease of maintenance.

---

### 5. Conclusion

This update represents a qualitative shift in managing favorite queries within NanoSoft applications. By restructuring scopes and moving them to an independent trait, along with adding advanced functions for ordering, filtering, and statistics, developers can now build sophisticated favorite systems with ease and high performance. Backward compatibility ensures a smooth transition for existing projects, while the new additions open up vast possibilities for rich user experiences.

---

**Note**: For more details about each scope and how to use it, please refer to the `FavoriteableModel` behavior documentation file or review the examples provided above.

See [docs/FavoriteableModel/Docs-FavoriteableModel-en.md](./docs/FavoriteableModel/Docs-FavoriteableModel-en.md)

See [docs/FavoriteableModel/Docs-FavoriteableModel-Advenced-Examples-en.md](./docs/FavoriteableModel/Docs-FavoriteableModel-Advenced-Examples-en.md)

