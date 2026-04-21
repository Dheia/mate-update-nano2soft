## `HasStoredReports` Trait – Predefined Reports System

**Overview of `Nano2.QueryBuilder` Package and `Reporting` Module**

This set of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder`** package, an integrated software package provided by **NanoSoft**, specifically designed for **Laravel** applications to facilitate the secure, flexible construction and management of complex queries and dynamic reports.

All classes in this module are under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

### Introduction

The **`HasStoredReports` trait** is a specialized addition to the `ReportsManager` class (or any other class) that enables **registering, managing, and executing predefined reports (report templates)**. This trait allows defining static (stored) reports that contain a base query configuration, customizable filters, access permissions, and export options. Authorized users can browse these reports and run them by passing dynamic filter values, simplifying the creation of customized reporting interfaces without redefining the query each time.

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `$storedReports` | `array` | An array containing all registered reports, indexed by `report_id`. |

### Key Methods

#### 1. Registering Reports

##### `public function registerReport(array $definition): void`
- **Purpose**: Register a single report in the system.
- **Parameters**:
  - `$definition`: An array containing the report data (see Report Definition Structure below).
- **Exceptions**: Throws `InvalidArgumentException` if the `report_id` already exists or the definition is invalid.
- **Example**:
  ```php
  $manager->registerReport([
      'report_id' => 'inventory_summary',
      'name' => 'Inventory Summary',
      'query_config' => [
          'table' => ['name' => 'tss_inventory_products'],
          'columns' => [['name' => 'name'], ['name' => 'quantity']],
      ],
  ]);
  ```

##### `public function registerReports(array $reports): void`
- **Purpose**: Register multiple reports at once.
- **Parameters**: `$reports` is an array of report definitions.
- **Example**:
  ```php
  $manager->registerReports([
      ['report_id' => 'rep1', 'name' => 'Report 1', 'query_config' => [...]],
      ['report_id' => 'rep2', 'name' => 'Report 2', 'query_config' => [...]],
  ]);
  ```

#### 2. Retrieving Stored Reports

##### `public function getStoredReport(string $reportId): ?array`
- **Purpose**: Get a specific report definition.
- **Parameters**: `$reportId` – The report identifier.
- **Returns**: The report definition as an array, or `null` if not found.

##### `public function getStoredReports(): array`
- **Purpose**: Get all registered reports.
- **Returns**: An array indexed by `report_id`.

##### `public function getAvailableStoredReports($user = null): array`
- **Purpose**: Get the reports available to a given user (or the current user if none provided) based on the defined permissions.
- **Parameters**: `$user` (optional) – A user object to check permissions against.
- **Returns**: An array of simplified report definitions (without internal details) that the user is allowed to access.
- **Example**:
  ```php
  $availableReports = $manager->getAvailableStoredReports(BackendAuth::getUser());
  ```

#### 3. Running a Stored Report

##### `public function runStoredReport(string $reportId, array $filterValues = [], array $options = []): array`
- **Purpose**: Execute a stored report after applying the provided filter values.
- **Parameters**:
  - `$reportId`: The report identifier.
  - `$filterValues`: An array of filter values (key = filter name, value = filter value).
  - `$options`: Additional options passed to `runReport` (e.g., `skip_validation`, `return_sql_only`).
- **Returns**: The report result (same format as `runReport`).
- **Exceptions**: Throws `InvalidArgumentException` if the report is not found or required filters are missing.
- **Example**:
  ```php
  $result = $manager->runStoredReport('inventory_summary', [
      'category' => 'electronics',
      'min_price' => 100
  ]);
  ```

### Report Definition Structure

A report definition array must contain the following keys:

| Key | Type | Description | Required |
|-----|------|-------------|----------|
| `report_id` | `string` | Unique identifier for the report | Yes |
| `name` | `string` | Report name (displayed to the user) | Yes |
| `query_config` | `array` | Base query configuration (as passed to `runReport`) | Yes |
| `description` | `string` | Report description (optional) | No |
| `filters` | `array` | Definition of available filters (see below) | No |
| `permissions` | `array` | Array of access permissions for the report | No |
| `export_options` | `array` | Allowed export options (default: `['formats' => ['csv', 'excel', 'json'], 'max_rows' => 50000]`) | No |
| `meta` | `array` | Additional metadata (optional) | No |

#### Filter Structure (`filters`)

An array where the key is the filter name (must match the `variable` used in `query_config` conditions), and the value is an array containing:

- `type`: Filter data type (optional, for validation).
- `required`: `true` if the filter is mandatory (default `false`).
- `options`: Additional options (e.g., dropdown values).

Example:
```php
'filters' => [
    'branch_id' => ['type' => 'integer', 'required' => true],
    'min_price' => ['type' => 'decimal', 'required' => false],
]
```

### Using Variables in `query_config`

For the trait to substitute filter values, the `query_config` must contain conditions with a `variable` key instead of a fixed `value`. Example:

```php
'query_config' => [
    'table' => ['name' => 'tss_inventory_products'],
    'columns' => [['name' => 'name']],
    'conditions' => [
        ['field' => ['name' => 'branch_id'], 'operator' => ['value' => '='], 'variable' => 'branch_id'],
        ['field' => ['name' => 'price'], 'operator' => ['value' => '>'], 'variable' => 'min_price'],
    ],
]
```

When the report is executed, each `variable` is replaced with the corresponding value from `$filterValues`.

### Comprehensive Examples

#### Example 1: Registering a Report with a Required Filter

```php
$manager->registerReport([
    'report_id' => 'products_by_branch',
    'name' => 'Products by Branch',
    'description' => 'Displays products in a specific branch',
    'query_config' => [
        'table' => ['name' => 'tss_inventory_products'],
        'columns' => [
            ['name' => 'name', 'label' => 'Name'],
            ['name' => 'price', 'label' => 'Price'],
        ],
        'conditions' => [
            ['field' => ['name' => 'branch_id'], 'operator' => ['value' => '='], 'variable' => 'branch_id'],
        ],
    ],
    'filters' => [
        'branch_id' => ['type' => 'integer', 'required' => true],
    ],
    'permissions' => ['inventory.reports.view'],
]);
```

#### Example 2: Getting Reports Available to the Current User

```php
public function listReports()
{
    $manager = ReportsManager::instance();
    $reports = $manager->getAvailableStoredReports();
    return response()->json($reports);
}
```

#### Example 3: Running a Report with Filters

```php
public function runReport($reportId, Request $request)
{
    $manager = ReportsManager::instance();
    $result = $manager->runStoredReport($reportId, $request->input('filters', []));
    return response()->json($result);
}
```

#### Example 4: Defining a Report in a Plugin

```php
public function registerReports()
{
    return [
        'monthly_sales' => [
            'report_id' => 'monthly_sales',
            'name' => 'Monthly Sales Report',
            'query_config' => [
                'table' => ['name' => 'sales_orders'],
                'columns' => [
                    ['name' => 'created_at', 'label' => 'Date'],
                    ['name' => 'customer.name', 'label' => 'Customer'],
                    ['name' => 'total_amount', 'label' => 'Total'],
                ],
                'conditions' => [
                    [
                        'field' => ['name' => 'created_at'],
                        'operator' => ['value' => '>='],
                        'variable' => 'start_date'
                    ],
                    [
                        'field' => ['name' => 'created_at'],
                        'operator' => ['value' => '<='],
                        'variable' => 'end_date'
                    ],
                ],
            ],
            'filters' => [
                'start_date' => ['type' => 'date', 'required' => true],
                'end_date' => ['type' => 'date', 'required' => true],
            ],
            'permissions' => ['sales.reports'],
        ],
    ];
}
```

### Interaction with Other Classes

- **`ReportsManager`**: The trait is designed to be used inside `ReportsManager` and relies on its methods such as `runReport`, `handleReportError`, and `getUserManager`.
- **`ReportUserManager`**: The trait uses the user manager to verify access permissions for reports (`permissions`).
- **`ReportQueryConverter`**: The merged `query_config` (with applied filters) is passed to `runReport`, which uses the converter to execute the query.

### Abstract Methods

For the trait to function correctly, the consuming class (e.g., `ReportsManager`) must implement the following methods:

- `abstract protected function getUserManager(): ReportUserManager;`
- `abstract protected function runReport(array $queryConfig, array $options = []): array;`
- `abstract protected function handleReportError(\Throwable $exception, array $context = []): array;`

### Benefits of Using This Trait

- **Reusability**: Define static reports once and run them flexibly with different filters.
- **Security**: Control access via specific permissions per report.
- **Easy Integration**: Plugins can register their reports via a `registerReports` method (similar to `registerReportTables`).
- **Filter Flexibility**: Support for variables in conditions with the ability to define mandatory and optional filters.
- **Compatibility with Existing System**: It does not interfere with other `ReportsManager` functions and works alongside dynamic, ad-hoc reports.

### Dependencies and Requirements

- **PHP** (>= 8.0)
- **Laravel** (>= 8.x) or **OctoberCMS** with support for `ReportsManager`.
- Requires the presence of `ReportsManager` with the `ReportsManagerQueryConverter` trait (which provides `getUserManager`, `runReport`, and `handleReportError`).

### Conclusion

The `HasStoredReports` trait is a powerful enhancement to the dynamic reporting system, enabling a library of predefined reports with full control over permissions and filters. Thanks to its modular design, developers can provide advanced reporting capabilities to users without complexity, while maintaining system security and performance.