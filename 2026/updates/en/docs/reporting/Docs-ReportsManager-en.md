## Documentation of `ReportsManager` Class and `ReportsManagerQueryConverter` Trait

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `ReportsManager` class is the beating heart for managing table definitions in the `Nano2.QueryBuilder` dynamic reporting system. This class acts as a facade over `ReportSchemaRegistry`, collecting table definitions from multiple sources (plugins via `registerReportTables`, direct registration, or callbacks), storing them raw, then converting them into `Table` objects and registering them in `ReportSchemaRegistry` when needed. The class supports lazy loading, caching of ready objects, and filtering tables based on user permissions.

The `ReportsManagerQueryConverter` trait provides an integrated set of functions for running reports, validating them, handling errors, exporting, and advanced options like returning only SQL or estimating cost, relying on the classes `ReportQueryValidator`, `ReportQueryConverter`, `ReportQueryErrorHandler`, and `ReportQueryExporter`. Thanks to this trait, the `ReportsManager` (or any other class using it) can become a comprehensive interface for easily and securely running reports.

---

## 1. `ReportsManager` Class

### Purpose
- Register table definitions (as raw data) from different plugins.
- Convert raw definitions into ready-to-use `Table` objects.
- Manage the lifecycle of these objects (caching, updating, removal).
- Provide an interface to access the `ReportSchemaRegistry` and its registered tables.
- Filter tables by permissions, extension, and category.

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$registry` | `ReportSchemaRegistry` | Reference to the main schema registry (initialized when needed). |
| `$registryInitialized` | `bool` | Flag indicating whether `$registry` has been initialized. |
| `$rawDefinitions` | `array` | Array of raw table definitions (indexed by table name). |
| `$builtTables` | `array` | Array of ready-made `Table` objects (cached). |
| `$loaded` | `bool` | Flag indicating that definitions have been loaded from plugins. |
| `$defaultTableDefinition` | `array` | Default table definition (merged with provided definitions). |
| `$defaultColumnDefinition` | `array` | Default column definition. |
| `$defaultRelationshipDefinition` | `array` | Default relationship definition. |
| `static $callbacks` | `array` | Array of callback functions for manual registration. |

### Main Methods

#### a. Registration and Loading

##### `registerDefinitions(array $definitions)`
- **Description**: Register a set of table definitions at once.
- **Parameters**: `$definitions` – an array keyed by table name with table definition as value.
- **Example**:
  ```php
  $manager->registerDefinitions([
      'orders' => [
          'label' => 'Orders',
          'columns' => ['id', 'amount'],
          'relationships' => [
              'customer' => ['table' => 'customers', 'columns' => ['name']]
          ]
      ]
  ]);
  ```

##### `registerDefinition(string $tableName, array $definition)`
- **Description**: Register a single table definition, merging default values and processing columns and relationships.

##### `static registerCallback(callable $callback)`
- **Description**: Register a callback function to be executed when loading definitions. The callback receives a `ReportsManager` instance.
- **Example**:
  ```php
  ReportsManager::registerCallback(function($manager) {
      $manager->registerDefinition('products', ['label' => 'Products']);
  });
  ```

##### `registerTable(Table $table)`
- **Description**: Directly register a `Table` object. It is added to `$builtTables` and registered in the registry if initialized.

##### `loadDefinitions()`
- **Description**: Load definitions from plugins (those implementing `registerReportTables`) and callbacks. Called automatically when needed.

#### b. Accessing Registry and Tables

##### `getRegistry(): ReportSchemaRegistry`
- **Description**: Get the `ReportSchemaRegistry` object (initializing it if necessary).

##### `getTableNames(): array`
- **Description**: Return names of all registered tables (from raw definitions).

##### `getTableDefinition(string $tableName): ?array`
- **Description**: Get a raw table definition.

##### `hasTable(string $tableName): bool`
- **Description**: Check if a table exists.

##### `getTableNamesByExtension(string $extension): array`
- **Description**: Return names of tables belonging to a specific extension.

##### `getTableNamesByCategory(string $category): array`
- **Description**: Return names of tables belonging to a specific category.

##### `getAvailableTablesForUser(string $extension, ?string $category = null): array`
- **Description**: Return tables available to the current user (after permission filtering) directly from the registry. Benefits from registry caching.
- **Example**:
  ```php
  $tables = $manager->getAvailableTablesForUser('core', 'sales');
  ```

##### `getBuiltTable(string $tableName): ?Table`
- **Description**: Get a ready-made `Table` object (from cache or build if necessary).

##### `getBuiltTables(): array`
- **Description**: Get all cached `Table` objects (building any that aren't built yet).

#### c. Updating and Removing Definitions

##### `updateDefinition(string $tableName, array $newDefinition)`
- **Description**: Update an existing table definition and reapply it to the registry if initialized.

##### `applyDefinitionToRegistry(string $tableName)`
- **Description**: Rebuild a specific table and register it in the registry (used after update).

##### `removeDefinition(string $tableName)`
- **Description**: Remove a table definition and clear associated cache.

##### `clearBuiltTables()`
- **Description**: Clear the cached ready-made objects.

#### d. Direct Access Methods to Registry

##### `getTableColumns(string $tableName): array`
- **Description**: Get table columns (including relationship columns) from the registry.

##### `isColumnAllowed(string $tableName, string $columnPath): bool`
- **Description**: Check if a column with a given path is allowed.

##### `getTableSchemaForUI(string $tableName): array`
- **Description**: Get complete table information (table, columns, relationships) suitable for the frontend.

##### `getAvailableTables(string $extension, ?string $category = null): array`
- **Description**: Get all tables available for a given extension and category (without permission filtering).

---

## 2. `ReportsManagerQueryConverter` Trait

### Purpose
Provide integrated functions for running reports: configuration validation, execution, error handling, exporting, caching, and advanced options like returning only SQL or estimating cost.

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$queryValidator` | `ReportQueryValidator` | Object for validating report configuration (initialized when needed). |
| `$queryErrorHandler` | `ReportQueryErrorHandler` | Object for handling errors. |
| `$cacheDriver` | `Cache\Repository` | Optional cache store for report results. |
| `$queryLogger` | `callable` | Function to log executed queries. |
| `$lastConverter` | `ReportQueryConverter` | The last query converter used (to access its information). |

### Main Methods

#### a. Basic Validation and Execution

##### `validateReportQuery(array $queryConfig): array`
- **Description**: Validate report configuration using `ReportQueryValidator`.
- **Returns**: Array containing `valid`, `errors`, `warnings`, `summary`.

##### `executeReportQuery(array $queryConfig): array`
- **Description**: Execute the query and return results (with execution time logging if `queryLogger` is enabled).
- **Returns**: Same as `ReportQueryConverter::execute()` result.

##### `handleReportError(\Throwable $exception, array $context = []): array`
- **Description**: Handle any exception and convert it to a structured response using `ReportQueryErrorHandler`.

##### `handleReportValidationError(array $validationResult, array $context = []): array`
- **Description**: Handle validation errors and return a structured response.

#### b. Advanced Report Execution Methods

##### `runReport(array $queryConfig, array $options = []): array`
- **Description**: Comprehensive function to run a report with advanced options.
- **Options**:
  - `skip_validation` (bool): Skip validation step (default false).
  - `return_sql_only` (bool): Return only the SQL object without executing (default false).
  - `dry_run` (bool): Only estimate cost (default false).
- **Returns**: Either report result, SQL information, or cost estimate.
- **Example**:
  ```php
  $result = $manager->runReport($queryConfig, ['return_sql_only' => true]);
  ```

##### `runReportWithCache(array $queryConfig, int $ttl = 3600, array $options = []): array`
- **Description**: Run the report with caching support. First checks `$cacheDriver`, then executes and caches the result if successful.

##### `runReportAndExport(array $queryConfig, string $format, array $exportOptions = []): array`
- **Description**: Execute the report then export results in the requested format using `ReportQueryExporter`.
- **Parameters**:
  - `$format`: Export format (csv, excel, json, pdf, xml).
  - `$exportOptions`: Additional export options.
- **Returns**: Export result (containing `filename`, `filepath`, `url`, etc.).

##### `estimateQueryCost(array $queryConfig): array`
- **Description**: Estimate query cost (complexity, expected performance, number of columns/joins/conditions) without executing it.

#### c. Helper Methods

##### `validateTableAccess(?string $tableName, $user = null): bool`
- **Description**: Check if the user has permission to access a specific table (based on registered table permissions).

##### `buildQueryConfigFromInput(array $input): array`
- **Description**: Convert request input (usually from JSON) into a standard `queryConfig` structure.

##### `enableCache($cacheDriver): void`
- **Description**: Enable caching by setting the `cacheDriver`.

##### `enableQueryLogging(callable $logger): void`
- **Description**: Enable query logging via a function that receives (`$queryConfig`, `$result`, `$executionTime`).

##### `formatError(array $errorResponse, string $outputType = 'json')`
- **Description**: Format the error for different outputs (json, html, text).

##### `formatForDataTables(array $result, int $draw = 1): array`
- **Description**: Format report results to suit the DataTables library in the frontend.

##### `getLastQueryAutoJoins(): array`
- **Description**: Get information about auto‑joins applied in the last query.

##### `getLastQueryJoinAliases(): array`, `getLastQueryManualJoins(): array`
- **Description**: Get alias information and manual joins from the last query.

##### `fireEvent($event, $payload)`
- **Description**: Fire an event to allow extensions to interact with the report running process. Available events: `reports.beforeRun`, `reports.afterRun`.

---

## 3. Practical Examples

#### Example 1: Registering a Table via a Plugin (in `Plugin.php`)

```php
<?php namespace Nano\Orders;

use System\Classes\PluginBase;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Relationship;

class Plugin extends PluginBase
{
    public function registerReportTables()
    {
        return [
            'orders' => [
                'label' => 'Orders',
                'category' => 'sales',
                'extension' => 'orders',
                'columns' => [
                    'uuid' => ['label' => 'ID', 'hidden' => true],
                    'created_at' => ['type' => 'datetime', 'label' => 'Creation Date'],
                    'amount' => ['type' => 'decimal', 'label' => 'Amount', 'aggregatable' => true],
                ],
                'relationships' => [
                    'customer' => [
                        'table' => 'customers',
                        'label' => 'Customer',
                        'auto_join' => true,
                        'columns' => [
                            'name' => ['label' => 'Name'],
                            'email' => ['label' => 'Email'],
                        ]
                    ]
                ],
                'permissions' => ['nano.orders.access_reports'],
            ]
        ];
    }
}
```

#### Example 2: Using `ReportsManager` in a Controller to Fetch Table Columns

```php
use Nano2\QueryBuilder\Classes\Reporting\ReportsManager;

class ReportController extends Controller
{
    public function getTableColumns($tableName)
    {
        $manager = ReportsManager::instance();
        
        if (!$manager->hasTable($tableName)) {
            return response()->json(['error' => 'Table not found'], 404);
        }
        
        $columns = $manager->getTableColumns($tableName);
        return response()->json($columns);
    }
}
```

#### Example 3: Running a Report with Basic Validation

```php
$queryConfig = [
    'table' => ['name' => 'orders'],
    'columns' => [
        ['name' => 'created_at', 'alias' => 'order_date'],
        ['name' => 'customer.name', 'alias' => 'customer_name'],
    ],
    'conditions' => [
        ['field' => ['name' => 'amount'], 'operator' => ['value' => '>'], 'value' => 100]
    ],
    'limit' => 50
];

$manager = ReportsManager::instance();
$result = $manager->runReport($queryConfig);

if ($result['success']) {
    return response()->json($result['data']);
} else {
    return response()->json($result['error'], 400);
}
```

#### Example 4: Using Advanced Options (Return SQL Only)

```php
$sqlInfo = $manager->runReport($queryConfig, ['return_sql_only' => true]);
// $sqlInfo = [
//     'success' => true,
//     'sql' => 'select ...',
//     'bindings' => [...],
//     'auto_joins' => [...]
// ]
```

#### Example 5: Estimating Query Cost

```php
$cost = $manager->estimateQueryCost($queryConfig);
if ($cost['complexity'] === 'high') {
    // warn the user
}
```

#### Example 6: Exporting Report Results

```php
$exportResult = $manager->runReportAndExport($queryConfig, 'excel', [
    'sheet_name' => 'Orders Report'
]);

if ($exportResult['success']) {
    return response()->download($exportResult['filepath']);
}
```

#### Example 7: Using Caching

```php
$manager->enableCache(Cache::store('redis'));
$cachedResult = $manager->runReportWithCache($queryConfig, 1800); // 30 minutes
```

#### Example 8: Formatting Results for DataTables

```php
$result = $manager->runReport($queryConfig);
$dataTablesResponse = $manager->formatForDataTables($result, request('draw'));
return response()->json($dataTablesResponse);
```

---

## 4. Relationships with Other Classes

- **`ReportSchemaRegistry`**: `ReportsManager` owns and feeds this registry with registered tables. All access methods for columns, relationships, and permission checks use the registry after it is initialized.
- **`Table`, `Column`, `Relationship`**: `ReportsManager` uses these classes to build ready objects from raw definitions.
- **`ReportQueryValidator`, `ReportQueryConverter`, `ReportQueryErrorHandler`, `ReportQueryExporter`**: The `ReportsManagerQueryConverter` trait uses them to implement its advanced functions.
- **`PluginManager` (from OctoberCMS)**: `ReportsManager` uses it to discover plugins implementing `registerReportTables`.
- **`BackendAuth`**: Used to filter tables by user permissions.

---

## 5. Conclusion

The `ReportsManager` class together with the `ReportsManagerQueryConverter` trait form a complete solution for managing and running dynamic reports in OctoberCMS applications. Thanks to its modular design, developers can:

- Easily register report tables from any plugin using simple definitions.
- Control the lifecycle of these definitions and update them.
- Run reports with multiple options (validation, caching, export, cost estimation).
- Provide clean programmatic interfaces for the frontend (e.g., fetching table columns, running reports, exporting).
- Ensure security via permission filtering and company scope (built into `ReportQueryConverter`).

This design makes it easy to build a powerful and flexible reporting system that meets advanced user needs without compromising system performance or integrity.