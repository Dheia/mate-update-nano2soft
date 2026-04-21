## DynamicReports Controller – API for Dynamic and Stored Reports

**Overview of `Nano2.QueryBuilder` Package and `Reporting` Module**

This set of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, etc.) is part of the **`Nano2.QueryBuilder`** package, an integrated software package provided by **NanoSoft**, specifically designed for **Laravel** and **OctoberCMS** applications to facilitate the secure, flexible construction and management of complex queries and dynamic reports.

The `DynamicReports` controller provides a unified API interface for interacting with the reporting system, whether for pre‑registered report templates or for ad‑hoc reports built on the fly.

### Authentication & Permissions

- All endpoints are protected by the `oauth-users` middleware, meaning a valid access token must be provided.
- User permissions are automatically checked when accessing a specific report or table, based on the permissions defined in the table and report definitions.

---

## Endpoints

### 1. List Available Stored Reports

**GET** `/api/v1/querybuilder/dynamic-reports`

#### Description
Returns a list of all pre‑registered reports that the current user is allowed to access.

#### Input
No additional parameters.

#### Output
An array of simplified report objects (without the internal `query_config`), each containing:
- `report_id`: the report identifier.
- `name`: the report name.
- `description`: a description of the report.
- `filters`: definitions of the available filters.
- `export_options`: allowed export options.
- `meta`: additional metadata (optional).

#### Example Request
```http
GET /api/v1/querybuilder/dynamic-reports
Authorization: Bearer {token}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Available reports retrieved successfully",
    "data": [
        {
            "report_id": "products_with_units",
            "name": "Products with Units",
            "description": "Products that have units (is_units = 1) with optional filtering by company and branch.",
            "filters": {
                "companys_id": {"type": "integer", "required": false, "label": "Company"},
                "departments_id": {"type": "integer", "required": false, "label": "Branch"}
            },
            "export_options": {
                "formats": ["csv", "excel", "json"],
                "max_rows": 50000
            },
            "meta": []
        },
        {
            "report_id": "products_without_units",
            "name": "Products without Units",
            "description": "Products that have no units (is_units = 0)",
            "filters": {
                "companys_id": {"type": "integer", "required": false},
                "departments_id": {"type": "integer", "required": false}
            },
            "export_options": {
                "formats": ["csv", "excel", "json"],
                "max_rows": 50000
            },
            "meta": []
        }
    ]
}
```

---

### 2. Show Details of a Stored Report

**GET** `/api/v1/querybuilder/dynamic-reports/{reportId}`

#### Description
Returns the details of a specific report (including `filters`, `export_options`, `meta`).

#### Input
- `reportId` (in path): the report identifier.

#### Output
The report object (without the internal `query_config`).

#### Example Request
```http
GET /api/v1/querybuilder/dynamic-reports/products_with_units
Authorization: Bearer {token}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Report details retrieved successfully",
    "data": {
        "report_id": "products_with_units",
        "name": "Products with Units",
        "description": "Products that have units (is_units = 1) with optional filtering by company and branch.",
        "filters": {
            "companys_id": {"type": "integer", "required": false, "label": "Company"},
            "departments_id": {"type": "integer", "required": false, "label": "Branch"}
        },
        "export_options": {
            "formats": ["csv", "excel", "json"],
            "max_rows": 50000
        },
        "meta": []
    }
}
```

---

### 3. Run a Stored Report

**POST** `/api/v1/querybuilder/dynamic-reports/run/{reportId}`

#### Description
Executes a stored report after applying the provided filter values (if any) and additional options (such as pagination and export settings).

#### Input (JSON Body)
- `filters` (optional): an object containing the values for the report’s filters. The keys must match the filter names defined in the report.
- `options` (optional): an object containing additional options:
  - `page` (integer): page number (for pagination).
  - `per_page` (integer): number of records per page.
  - `limit` (integer): number of records requested (alternative to `per_page`).
  - `offset` (integer): starting point (alternative to `page`).
  - `expose_meta_details` (boolean): if `true`, additional details (like the SQL query and joins used) are returned in `meta` – useful for debugging. The default follows `app.debug`.

#### Output
- `data`: an array of the records returned by the report.
- `columns`: information about the selected columns.
- `meta`: metadata including execution time, total rows, and pagination information (if requested).

#### Example Request
```http
POST /api/v1/querybuilder/dynamic-reports/run/products_with_units
Authorization: Bearer {token}
Content-Type: application/json

{
    "filters": {
        "companys_id": 1
    },
    "options": {
        "page": 2,
        "per_page": 50
    }
}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Report executed successfully",
    "data": {
        "success": true,
        "data": [
            {
                "id": 51,
                "name": "Product 51",
                "price": "250.00",
                "brand_name": null,
                "units_units_id": 3
            },
            ...
        ],
        "columns": [
            {"name": "id", "column_name": "id", "label": "ID", "type": "string"},
            {"name": "name", "column_name": "name", "label": "Name", "type": "string"},
            {"name": "price", "column_name": "price", "label": "Price", "type": "string"},
            {"name": "brand_name", "column_name": "brand.name", "label": "Brand", "type": "string"},
            {"name": "units_units_id", "column_name": "units.units_id", "label": "Unit ID", "type": "string"}
        ],
        "meta": {
            "total_rows": 50,
            "execution_time_ms": 4.21,
            "pagination": {
                "total": 1240,
                "count": 50,
                "per_page": 50,
                "current_page": 2,
                "total_pages": 25,
                "links": {
                    "previous": "https://example.com/api/v1/querybuilder/dynamic-reports/run/products_with_units?page=1&per_page=50",
                    "next": "https://example.com/api/v1/querybuilder/dynamic-reports/run/products_with_units?page=3&per_page=50"
                }
            }
        }
    }
}
```

---

### 4. Run a Dynamic (Ad‑hoc) Report

**POST** `/api/v1/querybuilder/dynamic-reports/execute`

#### Description
Executes a dynamic report that is fully defined in the request, without needing to be pre‑registered.

#### Input (JSON Body)
- `query` (required): an object representing the query configuration, following the same structure as `query_config` used in stored reports. It must contain:
  - `table`: an object with `name` (the base table name).
  - `columns`: an array of selected columns.
  - `conditions` (optional): an array of conditions.
  - `computed_columns` (optional): an array of computed columns.
  - `joins` (optional): an array of manual joins.
  - `groupBy` (optional): an array of groupings.
  - `sortBy` (optional): an array of orderings.
  - `limit` (optional): number of records.
  - `offset` (optional): starting point.
- `options` (optional): same `options` as described in the previous endpoint (including `page`, `per_page`, `expose_meta_details`).

#### Output
Same as for a stored report.

#### Example Request
```http
POST /api/v1/querybuilder/dynamic-reports/execute
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [
            {"name": "id", "label": "ID"},
            {"name": "name", "label": "Name"},
            {"name": "price", "label": "Price"},
            {"name": "brand.name", "label": "Brand"}
        ],
        "conditions": [
            {"field": {"name": "is_active"}, "operator": {"value": "="}, "value": 1},
            {"field": {"name": "companys_id"}, "operator": {"value": "="}, "value": 1}
        ],
        "limit": 20
    },
    "options": {
        "page": 1,
        "per_page": 10
    }
}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Report executed successfully",
    "data": {
        "success": true,
        "data": [...],
        "columns": [...],
        "meta": {
            "total_rows": 10,
            "execution_time_ms": 3.12,
            "pagination": {
                "total": 45,
                "count": 10,
                "per_page": 10,
                "current_page": 1,
                "total_pages": 5,
                "links": {
                    "next": "https://example.com/api/v1/querybuilder/dynamic-reports/execute?page=2&per_page=10"
                }
            }
        }
    }
}
```

---

### 5. Export a Report

**POST** `/api/v1/querybuilder/dynamic-reports/export`

#### Description
Runs a report (stored or dynamic) and then exports the results in a specified format (e.g., CSV, Excel, JSON).

#### Input (JSON Body)
You must provide either `report_id` (for a stored report) or `query` (for a dynamic report).

- `report_id` (optional): the identifier of a stored report.
- `query` (optional): the query configuration (as used in `execute`).
- `filters` (optional): filter values (if using `report_id`).
- `format` (required): the export format – one of `csv`, `excel`, `json`, `pdf`, `xml`.
- `export_options` (optional): additional export options (e.g., `delimiter` for CSV, `sheet_name` for Excel).

#### Output
An object containing information about the generated file:
- `success`: true.
- `format`: the used format.
- `filename`: the file name.
- `filepath`: the full path on the server.
- `size`: file size in bytes.
- `rows`: number of exported rows.
- `url`: a download URL.
- `download_url`: an alternative download URL (if a route is defined).

#### Example Request (exporting a stored report)
```http
POST /api/v1/querybuilder/dynamic-reports/export
Authorization: Bearer {token}
Content-Type: application/json

{
    "report_id": "products_with_units",
    "filters": {
        "companys_id": 1
    },
    "format": "excel",
    "export_options": {
        "sheet_name": "Products with Units"
    }
}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Report exported successfully",
    "data": {
        "success": true,
        "format": "excel",
        "filename": "report-products_with_units-2025-04-13-153045.xlsx",
        "filepath": "/var/www/html/storage/app/exports/report-products_with_units-2025-04-13-153045.xlsx",
        "size": 25600,
        "rows": 1240,
        "url": "https://example.com/v1/reports/export-download/report-products_with_units-2025-04-13-153045.xlsx",
        "download_url": "https://example.com/reports/download/report-products_with_units-2025-04-13-153045.xlsx"
    }
}
```

---

### 6. Get Table Schema (Available Columns)

**GET** `/api/v1/querybuilder/dynamic-reports/schema/{tableName}`

#### Description
Returns a list of all columns that can be queried from a given table (including columns from auto‑join relationships).

#### Input
- `tableName` (in path): the table name (e.g., `tss_inventory_products`).

#### Output
An array of columns, each containing:
- `name`: the actual column name (including the path if from a relationship).
- `label`: a display label.
- `type`: the data type.
- `auto_join_path`: the relationship path (if any).

#### Example Request
```http
GET /api/v1/querybuilder/dynamic-reports/schema/tss_inventory_products
Authorization: Bearer {token}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Table schema retrieved successfully",
    "data": [
        {"name": "id", "label": "Id", "type": "integer", "auto_join_path": null},
        {"name": "name", "label": "Name", "type": "string", "auto_join_path": null},
        {"name": "price", "label": "Price", "type": "decimal", "auto_join_path": null},
        {"name": "brand.name", "label": "Brand Name", "type": "string", "auto_join_path": "brand"},
        {"name": "currency.name", "label": "Currency Name", "type": "string", "auto_join_path": "currency"},
        {"name": "categories.name", "label": "Categories Name", "type": "string", "auto_join_path": "categories"}
    ]
}
```

---

### 7. Validate a Report Configuration

**GET** or **POST** `/api/v1/querybuilder/dynamic-reports/validate`

#### Description
Validates a report configuration (whether for a dynamic report or part of a stored report) without executing it.

#### Input
The query configuration can be passed as a JSON object in the body (for POST requests) or as a `query` parameter in the query string (for GET requests).

- `query` (required): the query configuration object.

#### Output
- `valid`: true if the configuration is valid.
- `errors`: an array of errors (if any).
- `warnings`: an array of warnings.
- `summary`: a summary including complexity and expected performance.

#### Example Request (POST)
```http
POST /api/v1/querybuilder/dynamic-reports/validate
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [{"name": "id"}, {"name": "name"}, {"name": "brand.name"}],
        "conditions": [
            {"field": {"name": "is_active"}, "operator": {"value": "="}, "value": 1}
        ]
    }
}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Report configuration is valid",
    "data": {
        "valid": true,
        "errors": [],
        "warnings": [],
        "summary": {
            "complexity": "low",
            "total_columns": 3,
            "total_joins": 1,
            "total_conditions": 1,
            "has_grouping": false,
            "has_sorting": false,
            "has_limit": false,
            "estimated_performance": "fast"
        }
    }
}
```

---

### 8. Estimate Query Cost

**GET** or **POST** `/api/v1/querybuilder/dynamic-reports/estimate`

#### Description
Estimates the cost of executing a query (complexity, expected performance) without actually running it.

#### Input
Same as for `validate`.

#### Output
- `success`: true if the configuration is valid.
- `complexity`: the degree of complexity (`low`, `medium`, `high`).
- `estimated_performance`: the expected performance (`fast`, `moderate`, `slow`).
- `total_columns`, `total_joins`, `total_conditions`: statistics.
- `warnings`: warnings.
- `errors`: errors (if any).

#### Example Request (POST)
```http
POST /api/v1/querybuilder/dynamic-reports/estimate
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [{"name": "id"}, {"name": "name"}],
        "joins": [
            {"table": "tss_inventory_companys_products", "type": "left", "local_key": "companys_products_id", "foreign_key": "id"}
        ]
    }
}
```

#### Example Response
```json
{
    "code": 200,
    "status": true,
    "message": "Query cost estimated successfully",
    "data": {
        "success": true,
        "complexity": "low",
        "estimated_performance": "fast",
        "total_columns": 2,
        "total_joins": 1,
        "total_conditions": 0,
        "warnings": [],
        "errors": []
    }
}
```

---

## Complete Examples

### Example 1: Simple Dynamic Report – First Page

**Request**:
```http
POST /api/v1/querybuilder/dynamic-reports/execute
Authorization: Bearer {token}
Content-Type: application/json

{
    "query": {
        "table": {"name": "tss_inventory_products"},
        "columns": [
            {"name": "id", "label": "ID"},
            {"name": "name", "label": "Name"},
            {"name": "price", "label": "Price"}
        ],
        "limit": 10
    }
}
```

**Response**:
```json
{
    "code": 200,
    "status": true,
    "message": "Report executed successfully",
    "data": {
        "success": true,
        "data": [
            {"id": 1, "name": "Product 1", "price": "100.00"},
            {"id": 2, "name": "Product 2", "price": "150.00"}
        ],
        "columns": [
            {"name": "id", "column_name": "id", "label": "ID", "type": "string"},
            {"name": "name", "column_name": "name", "label": "Name", "type": "string"},
            {"name": "price", "column_name": "price", "label": "Price", "type": "string"}
        ],
        "meta": {
            "total_rows": 2,
            "execution_time_ms": 2.34,
            "pagination": {
                "total": 500,
                "count": 2,
                "per_page": 10,
                "current_page": 1,
                "total_pages": 50
            }
        }
    }
}
```

### Example 2: Stored Report with Filter and Second Page

**Request**:
```http
POST /api/v1/querybuilder/dynamic-reports/run/products_with_units
Authorization: Bearer {token}
Content-Type: application/json

{
    "filters": {
        "companys_id": 2
    },
    "options": {
        "page": 2,
        "per_page": 25
    }
}
```

**Response** (similar to previous, but with different data).

### Example 3: Export a Stored Report to CSV

**Request**:
```http
POST /api/v1/querybuilder/dynamic-reports/export
Authorization: Bearer {token}
Content-Type: application/json

{
    "report_id": "products_without_units",
    "filters": {
        "companys_id": 1,
        "departments_id": 2
    },
    "format": "csv",
    "export_options": {
        "delimiter": ";",
        "include_bom": true
    }
}
```

**Response**:
```json
{
    "code": 200,
    "status": true,
    "message": "Report exported successfully",
    "data": {
        "success": true,
        "format": "csv",
        "filename": "report-products_without_units-2025-04-13-160215.csv",
        "filepath": "...",
        "size": 15200,
        "rows": 320,
        "url": "...",
        "download_url": "..."
    }
}
```

---

## Important Notes

- In the production environment (`app.debug = false`), sensitive details (like the SQL query) are hidden from `meta` and from error messages.
- When an error occurs, a user‑friendly message is returned together with an error code and a unique error ID for tracking.
- Additional details in `meta` can be controlled via the `expose_meta_details` option inside `options` – this is primarily intended for development environments.
- All dates and numbers are returned in a suitable format (dates as ISO strings, decimals as strings to preserve precision).

This covers all aspects of the `DynamicReports` controller with practical examples illustrating the expected usage.