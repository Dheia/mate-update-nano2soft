## General Documentation: Dynamic Reporting System `Nano2.QueryBuilder.Reporting`

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### 1. Introduction

The **`Nano2\QueryBuilder\Classes\Reporting` module** is part of the `Nano2.QueryBuilder` package, a specialized add-on provided by **NanoSoft Company** for NanoSoft App and Laravel applications. This module aims to provide an **integrated system for building, executing, and managing dynamic reports** in a secure, flexible, and scalable manner.

#### Why do we need a dynamic reporting system?
In complex administrative applications (such as fleet management systems, resource management, or ERP systems), end-users (non-programmers) need the ability to:
- View customized data from multiple tables.
- Apply complex filters and conditions.
- Aggregate data and calculate statistics (sums, averages, etc.).
- Export results in multiple formats (Excel, PDF, CSV...).

Instead of writing manual SQL queries or requesting custom reports from the development team each time, this system provides a **Schema Definition Layer** that allows the frontend to build a configuration representing the desired report. This configuration is then transformed into a secure SQL query, executed, and the results are returned in an organized manner. The `ReportsManager` handles collecting table definitions from plugins and managing their lifecycle, simplifying the integration process.

---

### 2. Main System Components

The system consists of a set of specialized classes, which can be divided into several layers:

#### a. Schema Definition Layer
This layer is responsible for describing the data structure available for reporting (tables, columns, relationships) in a programmatic way.

- **`Column`**: Represents a column in a table (or a computed column). It defines its name, data type, display label, and capabilities (search, sort, filter, aggregate). It can be regular or computed using an SQL expression.
- **`Relationship`**: Represents a relationship between two tables (e.g., `belongsTo`, `hasMany`). It specifies the relationship name, related table, join keys, and JOIN type. It supports the `autoJoin` feature, which automatically adds the JOIN when columns from the relationship are selected. It also supports nested relationships like `payload.pickup`.
- **`Table`**: Represents a main table. It collects regular columns, computed columns, and relationships. It provides methods for accessing visible columns and validating their permissions.
- **`ReportSchemaRegistry`**: A central registry where all `Table` objects are registered. It provides an interface for querying registered tables, their columns, and relationships, with support for caching to improve performance. It acts as the system's "permanent memory."

#### b. Schema Management Layer
This layer is responsible for collecting table definitions from multiple sources (plugins, direct registration, callbacks) and converting them into appropriate objects and delivering them to the `ReportSchemaRegistry`. It provides a unified interface for registering and managing tables, as well as advanced functions for running reports.

- **`ReportsManager`**: The main class in this layer. It aggregates raw table definitions (arrays) from plugins (via the `registerReportTables` function), stores them, and then converts them into `Table` objects when needed and registers them in the `ReportSchemaRegistry`. It supports lazy loading, caching of built objects, and filtering tables by permissions.
- **`ReportsManagerQueryConverter` (trait)**: A trait added to `ReportsManager` to provide it with integrated functions for running reports:
  - Validate the report configuration (`validateReportQuery`).
  - Execute the query (`executeReportQuery`).
  - Handle errors (`handleReportError`).
  - Export results (`runReportAndExport`).
  - Cache results (`runReportWithCache`).
  - Estimate query cost (`estimateQueryCost`).
  - Advanced options (return SQL only, dry run).

#### c. Validation Layer
Before building any query, it must be ensured that the report configuration provided by the user is correct and secure.

- **`ComputedColumnValidator`**: Validates computed column expressions. It ensures:
  - No forbidden keywords (SQL Injection).
  - All used functions are within an allowed list.
  - Column references exist in the table, its relationships, or other computed columns.
- **`ReportQueryValidator`**: Validates the entire report configuration. It includes:
  - Basic structure (presence of a table and columns).
  - Existence of the specified table and columns in the `ReportSchemaRegistry`.
  - Correctness of conditions (operators, values).
  - Correctness of groupings (`groupBy`) and sorting (`sortBy`).
  - Allowed limits (`limit`).
  - Performance warnings (query complexity, number of joins, many conditions).
  - Additional security checks.

#### d. Query Execution Layer
This layer transforms the valid configuration into an actual SQL query.

- **`ReportQueryConverter`**: The main class that builds and executes a Laravel Query Builder query. It performs:
  - Automatic application of company scope (`company_uuid`).
  - Parsing paths (e.g., `payload.pickup.address`) and applying necessary auto-joins using relationship information from `Table` and `Relationship`.
  - Building the SELECT clause with regular, computed, and aggregate columns.
  - Building the WHERE clause from nested conditions.
  - Building GROUP BY, ORDER BY, and LIMIT.
  - Executing the query and returning results with metadata (execution time, executed query, etc.).

#### e. Export Layer
After obtaining results, they can be exported in multiple formats.

- **`ReportQueryExporter`**: Exports the data (`$data`) to files in various formats:
  - **CSV**: with options (delimiter, encoding).
  - **Excel (XLSX)**: using PhpSpreadsheet with advanced formatting (colors, borders, number/date formatting, Metadata sheet).
  - **JSON**: with or without Pretty Print.
  - **PDF**: (future development) currently generates HTML.
  - **XML**: structured output.

#### f. Error Handling Layer
- **`ReportQueryErrorHandler`**: Provides a unified interface for handling errors and exceptions. It performs:
  - Determining the appropriate error code (e.g., `TABLE_NOT_FOUND`, `TIMEOUT`).
  - Creating a unique error ID for tracking.
  - Returning user-friendly error messages with suggestions.
  - Logging all technical details for diagnosis.

---

### 3. How These Classes Work Together (General Flow)

Suppose a frontend user wants to create a report on orders including the customer name and pickup city.

1. **Schema Registration (one-time)**:
   - In any plugin's `Plugin.php` file, the `registerReportTables` function is defined, returning an array of table definitions.
   - The `ReportsManager` (a Singleton class) automatically discovers these plugins via the `PluginManager`, collects the definitions, and stores them raw (`$rawDefinitions`). Manual registration is also possible via `registerDefinition` or `registerCallback`.
   - On the first call to `getRegistry()`, a `ReportSchemaRegistry` is initialized, `Table` objects are built from the raw definitions, and registered in the registry. These objects are also stored in `$builtTables` for speed.

2. **Frontend Request for Schema**:
   - The frontend calls an API like `/api/reports/schema/orders`.
   - `ReportsManager::getTableColumns('orders')` is used, which passes the request to `ReportSchemaRegistry::getTableColumns()`. Columns can be filtered by user permissions later.

3. **Building the Report Configuration**:
   - The user selects columns, conditions, groupings, sorting, and limit.
   - The frontend sends a JSON object representing the `queryConfig` (matching an expected structure).

4. **Validation and Execution via `ReportsManager`**:
   - Instead of manually creating `ReportQueryValidator` and `ReportQueryConverter`, the `ReportsManager` methods added by the trait are used:
     ```php
     $manager = ReportsManager::instance();
     $result = $manager->runReport($queryConfig);
     ```
   - The `runReport` function performs validation (`validateReportQuery`) and then execution (`executeReportQuery`) with automatic error handling. It also supports options like `skip_validation`, `return_sql_only`, `dry_run`.

5. **Export (optional)**:
   - `runReportAndExport` can be used to run the report and export it directly.

6. **Error Handling**:
   - Any exception is caught and processed via `handleReportError`, which internally uses `ReportQueryErrorHandler`.

---

### 4. Typical Use Cases

#### a. Simple Report Without Aggregation
- **Scenario**: A sales manager wants to view all completed orders with creation date and customer name.
- **Configuration**: `orders` table, columns `created_at` and `customer.name`, condition `status = 'completed'`, order by date descending.
- **Classes Involved**: `ReportsManager` (for registration and execution), `ReportQueryValidator`, `ReportQueryConverter`, `ReportSchemaRegistry`.

#### b. Report with Aggregation and Computed Columns
- **Scenario**: An inventory manager wants to know total sales per city with tax calculation.
- **Configuration**:
  - `groupBy` on `payload.pickup.city`.
  - `aggregateFn` = `sum` on the `amount` column.
  - `aggregateFn` = `avg` on the `amount` column.
  - Computed column: `total_with_tax` = `ROUND(amount * 1.14, 2)`.
- **Classes Involved**: `ComputedColumnValidator`, `ReportQueryConverter`, `ReportsManager`.

#### c. Report with Deep Nested Relationships
- **Scenario**: A report on shipments including pickup address (from `payload.pickup.address`) and dropoff city (from `payload.dropoff.city`).
- **Configuration**: Columns `payload.pickup.address`, `payload.dropoff.city`.
- **Classes Involved**: `ReportSchemaRegistry::isColumnAllowed`, `ReportQueryConverter::resolveAliasAndColumn`, `ReportsManager`.

#### d. Exporting a Report
- **Scenario**: After viewing the report, the user wants to download it as an Excel file.
- **Action**: Use `ReportsManager::runReportAndExport($queryConfig, 'excel')`.
- **Result**: A formatted Excel file.

#### e. Cost Estimation Before Execution
- **Scenario**: Before running a complex report, the system wants to warn the user if the query is heavy.
- **Action**: `ReportsManager::estimateQueryCost($queryConfig)` returns a complexity score and expected performance.

---

### 5. Benefits and Advantages

- **Security**:
  - Company scope (`company_uuid`) is automatically applied to all queries.
  - `ComputedColumnValidator` prevents the use of dangerous SQL keywords.
  - Validation of column existence in the schema before building the query prevents errors.
  - Use of Prepared Statements via Laravel Query Builder.
  - `ReportsManager` checks user permissions before displaying tables or executing reports.

- **Flexibility**:
  - Any table can be defined with any number of nested relationships.
  - Support for computed columns with limited and safe SQL expressions.
  - Registration via `registerReportTables` in plugins, or via `registerDefinition`, or `registerCallback`.
  - Ability to pass advanced options to `runReport` (skip validation, return SQL only, etc.).

- **Performance**:
  - `auto-join` is used only when needed (avoiding unnecessary JOINs).
  - Schema caching in `ReportSchemaRegistry` reduces repeated database queries.
  - Ability to set `maxRows` and pagination limits.
  - Support for caching report results via `runReportWithCache`.

- **Scalability**:
  - Clear separation of responsibilities (Schema, Management, Validation, Conversion, Export, Error Handling).
  - Any part can be easily replaced or extended (e.g., adding a new data source, or a new export format).
  - `ReportsManager` centralizes all definitions, making them easier to track and update.

- **User Experience**:
  - Clear error messages with suggestions.
  - Support for multiple export formats.
  - Ability to build complex reports without needing to know SQL.
  - Simple programmatic interface via `ReportsManager::runReport()`.

---

### 6. Quick Practical Example (Integrated Code)

Using `ReportsManager`, the code in the Controller becomes simpler and clearer:

```php
<?php namespace Nano\Reports\Http\Controllers;

use Illuminate\Routing\Controller;
use Illuminate\Http\Request;
use Nano2\QueryBuilder\Classes\Reporting\ReportsManager;

class ReportController extends Controller
{
    public function generate(Request $request)
    {
        $queryConfig = $request->input('query');
        $manager = ReportsManager::instance();

        // Run the report with options (e.g., skip validation)
        $result = $manager->runReport($queryConfig, [
            'skip_validation' => false
        ]);

        if (!$result['success']) {
            return response()->json($result, 400);
        }

        // If export is requested
        if ($request->input('export')) {
            $export = $manager->runReportAndExport(
                $queryConfig, 
                $request->input('format', 'csv')
            );
            return response()->json($export);
        }

        return response()->json($result);
    }

    public function estimate(Request $request)
    {
        $queryConfig = $request->input('query');
        $cost = ReportsManager::instance()->estimateQueryCost($queryConfig);
        return response()->json($cost);
    }

    public function schema($tableName)
    {
        $manager = ReportsManager::instance();
        $columns = $manager->getTableColumns($tableName);
        return response()->json($columns);
    }
}
```

**Example of registering tables from a plugin** (in `Plugin.php`):

```php
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
```

---

### 7. Dependencies and Requirements

- **Laravel** (>= 8.x) with Query Builder support.
- **PHP** (>= 8.0).
- **OctoberCMS** (optional, but the system is designed to work with it).
- **Composer** for package installation.
- **PhpOffice/PhpSpreadsheet** (for Excel export) – can be added optionally.
- **Redis/Memcached** (optional) for caching in `ReportSchemaRegistry` and `ReportsManager`.

---

### 8. Conclusion

The `Nano2\QueryBuilder\Classes\Reporting` classes represent a comprehensive and professional solution for building a dynamic reporting system in NanoSoft App and Laravel applications. Thanks to the layered design (Schema, Management, Validation, Conversion, Export, Error Handling), developers can build complex reports quickly and securely. With the addition of `ReportsManager`, the system has become more organized and easier to use, providing a single entry point for managing table definitions and executing reports, while supporting advanced options such as caching, export, and cost estimation. This design ensures scalability and maintainability, meeting advanced user needs without compromising system performance or security.