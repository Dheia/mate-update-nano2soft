## Documentation of `ReportQueryExporter` Class

**Introduction to the `Nano2.QueryBuilder` Package and the `Reporting` Module**

This group of classes (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`, and others) is part of the **`Nano2.QueryBuilder` package**, a comprehensive software package provided by **NanoSoft Company**, specifically designed for **Laravel** applications to facilitate the building and management of complex queries and dynamic reports in a secure and flexible manner.

All classes of this module fall under the following namespace:

```
Nano2\QueryBuilder\Classes\Reporting
```

This module provides advanced tools for defining the data structure (tables, columns, relationships), validating queries, executing them efficiently, exporting results in multiple formats, and handling errors uniformly. Thanks to this design, developers can easily build scalable reporting systems and meet advanced user needs without compromising application performance or integrity.

### Introduction
The `ReportQueryExporter` class is responsible for **exporting report results** to multiple downloadable file formats. It receives data (`$data`), columns (`$columns`), metadata (`$metadata`), and the table name (`$tableName`), and creates a file in a user-specified format (CSV, Excel, JSON, PDF, XML) with appropriate formatting for data (dates, numbers, currencies, etc.). The class relies on the `PhpOffice\PhpSpreadsheet` library for creating Excel files and uses built-in PHP functions for other formats.

---

### Properties

| Property | Type | Description |
|---------|------|-------------|
| `$data` | `array` | Array of results to be exported (each element can be a `stdClass` object or an array). |
| `$columns` | `array` | An array describing the columns, each containing `key` (field name in data), `label` (column title for display), `type` (data type for formatting). |
| `$metadata` | `array` | Additional descriptive data about the report (e.g., creation time, row count) added to some formats (JSON, Excel). |
| `$tableName` | `string` | The name of the table or report, used in generating the file name (e.g., `report-orders-2025-04-06-143022.xlsx`). |

---

### Main Methods

#### 1. `__construct(array $data, array $columns, array $metadata = [], string $tableName = 'report')`
- **Purpose**: Creates a new `ReportQueryExporter` object with the provided data, columns, and metadata.
- **Parameters**:
  - `$data`: Array of results (typically from `$query->get()->toArray()`).
  - `$columns`: Array of columns as returned by `getSelectedColumns()` from `ReportQueryConverter`.
  - `$metadata`: Additional data (e.g., `total_rows`, `execution_time_ms`).
  - `$tableName`: The table name to be used in the file name.

#### 2. `export(string $format, array $options = []): array`
- **Purpose**: Exports the data in the requested format.
- **Parameters**:
  - `$format`: The requested format (`csv`, `excel`/`xlsx`, `json`, `pdf`, `xml`).
  - `$options`: An array of additional options specific to each format (e.g., `delimiter` for CSV, `include_bom`, `sheet_name` for Excel, `compact` for JSON).
- **Returns**: An array containing information about the exported file (e.g., `success`, `filename`, `filepath`, `size`, `url`, `download_url`).
- **Exceptions**: Throws `InvalidArgumentException` if the format is not supported.

#### 3. `exportToCsv(array $options = []): array`
- **Purpose**: Exports the data to a CSV file.
- **Options**:
  - `include_bom`: (boolean) Add a BOM for UTF-8 encoding (default `true`).
  - `delimiter`: (string) Field delimiter (default `,`).
  - `enclosure`: (string) Text enclosure (default `"`).
- **Steps**:
  - Generate a unique file name (`report-{table}-{timestamp}.csv`).
  - Ensure the export directory exists.
  - Open the file for writing, write BOM if requested.
  - Write the header row using `fputcsv`.
  - Write each data row after formatting values via `formatCellValue`.
  - Close the file and return the export response.

#### 4. `exportToExcel(array $options = []): array`
- **Purpose**: Exports the data to an Excel (XLSX) file with advanced formatting.
- **Options**:
  - `include_metadata`: (boolean) Add a separate sheet for metadata (default `true`).
  - `sheet_name`: (string) Name of the main sheet (default `Str::title($tableName)`).
- **Steps**:
  - Create a `Spreadsheet` object from PhpSpreadsheet.
  - Set the sheet name.
  - If requested, add a metadata sheet via `addMetadataSheet`.
  - Apply styles to headers (bold, background color, borders).
  - Write data with appropriate formatting per column via `applyCellFormatting`.
  - Auto-size columns.
  - Add borders to data rows.
  - Freeze the first row.
  - Save the file using `Xlsx` writer.
  - Return the export response.

#### 5. `exportToJson(array $options = []): array`
- **Purpose**: Exports the data to a JSON file.
- **Options**:
  - `compact`: (boolean) If `true`, produces JSON without spaces (smaller size). Default `false` (with `JSON_PRETTY_PRINT`).
- **Steps**:
  - Create an `$exportData` array containing `metadata`, `columns`, `data`.
  - Convert to JSON with appropriate options.
  - Save content to a file.
  - Return the export response.

#### 6. `exportToPdf(array $options = []): array`
- **Purpose**: Exports the data to a PDF file.
- **Note**: This implementation is currently **experimental** and generates an HTML file that can later be converted using a PDF library (e.g., DomPDF). It creates an HTML file alongside a (dummy) PDF file and returns information about it.
- **Options**:
  - `title`: Report title (default `Str::title($tableName) . ' Report'`).
- **Steps**:
  - Generate a PDF file name.
  - Generate HTML via `generatePdfHtml`.
  - Save the HTML in a separate file (for manual use).
  - Return a PDF response with a note about the need to integrate a real PDF library.

#### 7. `exportToXml(array $options = []): array`
- **Purpose**: Exports the data to an XML file.
- **Steps**:
  - Create a `SimpleXMLElement` object with root `<report>`.
  - Add a `<metadata>` node containing the metadata.
  - Add a `<columns>` node containing each column with its attributes.
  - Add a `<data>` node containing rows (`<row>`) with each column as a child element named by the key.
  - Format the XML using `DOMDocument` with `formatOutput = true`.
  - Save the file.
  - Return the export response.

#### 8. `formatCellValue($value, array $column)`
- **Purpose**: Formats a cell value based on the column type.
- **Supported types**:
  - `date`: Format `Y-m-d` using `Carbon`.
  - `datetime`: Format `Y-m-d H:i:s`.
  - `number`, `decimal`: Convert to float.
  - `currency`: Format currency (uses `Utils::moneyFormat` defaulting to USD).
  - `percentage`: Multiply value by 100 and add `%` with two decimal places.
  - `boolean`: Convert to `'Yes'` / `'No'`.
  - Other types: Convert to string.

#### 9. `applyCellFormatting($cell, array $column, $value)`
- **Purpose**: Applies Excel formatting to the cell (alignment, number format, date, etc.) using PhpSpreadsheet.
- Relies on column type to set `NumberFormat` and `Alignment`.

#### 10. `addMetadataSheet(Spreadsheet $spreadsheet)`
- **Purpose**: Adds a new sheet named "Metadata" containing the metadata (e.g., `exported_at`, `total_rows`, `columns_count`).

#### 11. `generatePdfHtml(array $options = []): string`
- **Purpose**: Generates HTML content for the PDF file.
- Includes a title, a data table, and basic CSS styling.

#### 12. `generateFileName(string $extension): string`
- **Purpose**: Generates a unique file name using `$tableName`, date, and time.
- Example: `report-orders-2025-04-06-143022.xlsx`

#### 13. `getExportPath(string $fileName): string`
- **Purpose**: Returns the full path to the file within `storage_path('app/exports/')`.

#### 14. `ensureExportDirectory(): void`
- **Purpose**: Ensures the `exports` directory exists within storage, and creates it if not.

#### 15. `buildExportResponse(string $format, string $fileName, string $filePath, array $additional = []): array`
- **Purpose**: Builds the final export response array, containing:
  - `success`: true
  - `format`: The format
  - `filename`: The file name
  - `filepath`: The full path
  - `size`: File size in bytes
  - `rows`: Number of rows
  - `url`: Public URL (assuming a `reports.download` route exists)
  - `download_url`: Download link (can be used in the frontend)

#### 16. `getSupportedFormats(): array` (static)
- **Purpose**: Returns a list of all supported formats with information about each (label, mime_type, extension, description). Useful for displaying in the frontend.

---

### Practical Examples

#### Example 1: Export to CSV (with custom options)
```php
use Nano2\QueryBuilder\Classes\Reporting\ReportQueryExporter;

// After executing the report and obtaining $data, $columns, $metadata
$exporter = new ReportQueryExporter($data, $columns, $metadata, 'orders');
$result = $exporter->export('csv', [
    'delimiter' => ';',
    'include_bom' => true,
]);

if ($result['success']) {
    return response()->download($result['filepath'], $result['filename']);
}
```

#### Example 2: Export to Excel with Metadata sheet
```php
$result = $exporter->export('excel', [
    'sheet_name' => 'Orders Report',
    'include_metadata' => true,
]);

// Can return a download link
return response()->json(['download_url' => $result['download_url']]);
```

#### Example 3: Export to JSON in compact form
```php
$result = $exporter->export('json', ['compact' => true]);
```

#### Example 4: Get list of supported formats
```php
$formats = ReportQueryExporter::getSupportedFormats();
// $formats = [
//     'csv' => ['label' => 'CSV', 'mime_type' => 'text/csv', ...],
//     'excel' => ...
// ]
```

#### Example 5: Use in a complete Controller
```php
class ReportExportController extends Controller
{
    public function export(Request $request, ReportQueryConverter $converter)
    {
        $queryConfig = $request->input('query');
        $format = $request->input('format', 'csv');

        // Execute the report
        $result = $converter->execute();

        if (!$result['success']) {
            return response()->json($result, 400);
        }

        // Export the results
        $exporter = new ReportQueryExporter(
            $result['data'],
            $result['columns'],
            $result['meta'],
            $queryConfig['table']['name'] ?? 'report'
        );

        $exportResult = $exporter->export($format, $request->input('options', []));

        return response()->json($exportResult);
    }
}
```

#### Example 6: Handling export errors
```php
try {
    $exportResult = $exporter->export('pdf');
} catch (\Exception $e) {
    // Use error handler
    $errorHandler = app(ReportQueryErrorHandler::class);
    return $errorHandler->handleExportError($e, 'pdf', ['table' => $tableName]);
}
```

---

### Important Notes
- **Dependencies**: Exporting to Excel requires the `phpoffice/phpspreadsheet` library. It must be added via Composer.
- **PDF**: The current PDF implementation is incomplete; it only generates HTML. To use a real PDF, a library like `barryvdh/laravel-dompdf` can be integrated.
- **Number Formatting**: Decimal numbers are converted using `(float) $value` to preserve precision.
- **Encoding**: CSV adds a BOM (`\xEF\xBB\xBF`) to ensure proper UTF-8 display in Excel.
- **Performance**: For large datasets, Excel export may need optimization (e.g., writing in chunks) to avoid memory exhaustion.

---

### Extending the Class
New formats can be easily added by:
1. Adding a new case in `export()`.
2. Creating a format-specific function (e.g., `exportToHtml`).
3. Updating the static `getSupportedFormats()` method.

---

### Summary
`ReportQueryExporter` provides a unified and flexible interface for exporting report data to multiple formats, with smart data formatting based on type. It isolates the export logic from the rest of the system, making it easier to maintain and extend. Thanks to customizable options, developers can meet various end-user needs.