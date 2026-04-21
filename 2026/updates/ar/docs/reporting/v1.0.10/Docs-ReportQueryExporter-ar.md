## توثيق كلاس `ReportQueryExporter`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `ReportQueryExporter` هو المسؤول عن **تصدير نتائج التقارير** إلى صيغ ملفات متعددة قابلة للتنزيل. يستقبل البيانات (`$data`) والأعمدة (`$columns`) والبيانات الوصفية (`$metadata`) واسم الجدول (`$tableName`)، ويقوم بإنشاء ملف بصيغة يحددها المستخدم (CSV, Excel, JSON, PDF, XML) مع تنسيق مناسب للبيانات (تواريخ، أرقام، عملات، إلخ). يعتمد الكلاس على مكتبة `PhpOffice\PhpSpreadsheet` لإنشاء ملفات Excel، ويستخدم دوال PHP المضمنة للصيغ الأخرى.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$data` | `array` | مصفوفة النتائج المراد تصديرها (كل عنصر يمكن أن يكون كائنًا `stdClass` أو مصفوفة). |
| `$columns` | `array` | مصفوفة تصف الأعمدة، تحتوي كل منها على `key` (اسم الحقل في البيانات)، `label` (عنوان العمود للعرض)، `type` (نوع البيانات لتنسيقها). |
| `$metadata` | `array` | بيانات وصفية إضافية عن التقرير (مثل وقت الإنشاء، عدد الصفوف) تُضاف إلى بعض الصيغ (JSON، Excel). |
| `$tableName` | `string` | اسم الجدول أو التقرير، يُستخدم في توليد اسم الملف (مثل `report-orders-2025-04-06-143022.xlsx`). |

---

### أهم الطرق

#### 1. `__construct(array $data, array $columns, array $metadata = [], string $tableName = 'report')`
- **الهدف**: إنشاء كائن جديد من `ReportQueryExporter` مع تمرير البيانات والأعمدة والبيانات الوصفية.
- **المعاملات**:
  - `$data`: مصفوفة النتائج (عادة من `$query->get()->toArray()`).
  - `$columns`: مصفوفة الأعمدة كما تُرجعها `getSelectedColumns()` من `ReportQueryConverter`.
  - `$metadata`: بيانات إضافية (مثل `total_rows`, `execution_time_ms`).
  - `$tableName`: اسم الجدول لاستخدامه في اسم الملف.

#### 2. `export(string $format, array $options = []): array`
- **الهدف**: تصدير البيانات بالصيغة المطلوبة.
- **المعاملات**:
  - `$format`: الصيغة المطلوبة (`csv`, `excel`/`xlsx`, `json`, `pdf`, `xml`).
  - `$options`: مصفوفة خيارات إضافية خاصة بكل صيغة (مثل `delimiter` لـ CSV، `include_bom`، `sheet_name` لـ Excel، `compact` لـ JSON).
- **الإرجاع**: مصفوفة تحتوي على معلومات عن ملف التصدير (مثل `success`, `filename`, `filepath`, `size`, `url`, `download_url`).
- **الاستثناءات**: يرمي `InvalidArgumentException` إذا كانت الصيغة غير مدعومة.

#### 3. `exportToCsv(array $options = []): array`
- **الهدف**: تصدير البيانات إلى ملف CSV.
- **الخيارات**:
  - `include_bom`: (boolean) إضافة BOM للترميز UTF-8 (افتراضي `true`).
  - `delimiter`: (string) فاصل الحقول (افتراضي `,`).
  - `enclosure`: (string) محيط النص (افتراضي `"`).
- **الخطوات**:
  - إنشاء اسم ملف فريد (`report-{table}-{timestamp}.csv`).
  - التأكد من وجود مجلد التصدير.
  - فتح الملف للكتابة، كتابة BOM إذا طُلب.
  - كتابة صف الرؤوس باستخدام `fputcsv`.
  - كتابة كل صف بيانات بعد تنسيق القيم عبر `formatCellValue`.
  - إغلاق الملف وإرجاع استجابة التصدير.

#### 4. `exportToExcel(array $options = []): array`
- **الهدف**: تصدير البيانات إلى ملف Excel (XLSX) مع تنسيق متقدم.
- **الخيارات**:
  - `include_metadata`: (boolean) إضافة ورقة منفصلة للبيانات الوصفية (افتراضي `true`).
  - `sheet_name`: (string) اسم الورقة الرئيسية (افتراضي `Str::title($tableName)`).
- **الخطوات**:
  - إنشاء كائن `Spreadsheet` من PhpSpreadsheet.
  - تعيين اسم الورقة.
  - إذا طُلب، إضافة ورقة metadata عبر `addMetadataSheet`.
  - تطبيق أنماط على الرؤوس (خط عريض، لون خلفية، حدود).
  - كتابة البيانات مع تطبيق التنسيق المناسب لكل عمود عبر `applyCellFormatting`.
  - ضبط عرض الأعمدة تلقائيًا.
  - إضافة حدود لصفوف البيانات.
  - تجميد الصف الأول.
  - حفظ الملف باستخدام `Xlsx` writer.
  - إرجاع استجابة التصدير.

#### 5. `exportToJson(array $options = []): array`
- **الهدف**: تصدير البيانات إلى ملف JSON.
- **الخيارات**:
  - `compact`: (boolean) إذا كان `true`، يُنتج JSON بدون مسافات (أصغر حجماً). افتراضي `false` (مع `JSON_PRETTY_PRINT`).
- **الخطوات**:
  - إنشاء مصفوفة `$exportData` تحتوي على `metadata`, `columns`, `data`.
  - تحويلها إلى JSON مع الخيارات المناسبة.
  - حفظ المحتوى في ملف.
  - إرجاع استجابة التصدير.

#### 6. `exportToPdf(array $options = []): array`
- **الهدف**: تصدير البيانات إلى ملف PDF.
- **ملاحظة**: هذا التطبيق حاليًا **تجريبي** وينشئ ملف HTML يمكن تحويله لاحقاً باستخدام مكتبة PDF (مثل DomPDF). يُنشئ ملف HTML بجانب ملف PDF (فارغ) ويعيد معلومات عنه.
- **الخيارات**:
  - `title`: عنوان التقرير (افتراضي `Str::title($tableName) . ' Report'`).
- **الخطوات**:
  - إنشاء اسم ملف PDF.
  - توليد HTML عبر `generatePdfHtml`.
  - حفظ HTML في ملف منفصل (للاستخدام اليدوي).
  - إرجاع استجابة PDF مع ملاحظة تفيد بضرورة دمج مكتبة PDF حقيقية.

#### 7. `exportToXml(array $options = []): array`
- **الهدف**: تصدير البيانات إلى ملف XML.
- **الخطوات**:
  - إنشاء كائن `SimpleXMLElement` بجذر `<report>`.
  - إضافة عقدة `<metadata>` تحتوي على البيانات الوصفية.
  - إضافة عقدة `<columns>` تحتوي على كل عمود مع خصائصه.
  - إضافة عقدة `<data>` تحتوي على صفوف (`<row>`) مع كل عمود كعنصر فرعي باسم المفتاح.
  - تنسيق XML باستخدام `DOMDocument` مع `formatOutput = true`.
  - حفظ الملف.
  - إرجاع استجابة التصدير.

#### 8. `formatCellValue($value, array $column)`
- **الهدف**: تنسيق قيمة خلية بناءً على نوع العمود.
- **الأنواع المدعومة**:
  - `date`: تنسيق `Y-m-d` باستخدام `Carbon`.
  - `datetime`: تنسيق `Y-m-d H:i:s`.
  - `number`, `decimal`: تحويل إلى رقم عشري.
  - `currency`: تنسيق عملة (يستخدم `Utils::moneyFormat` افتراضيًا بـ USD).
  - `percentage`: ضرب القيمة في 100 وإضافة `%` مع منزلتين عشريتين.
  - `boolean`: تحويل إلى `'Yes'` / `'No'`.
  - أنواع أخرى: تحويل إلى نص.

#### 9. `applyCellFormatting($cell, array $column, $value)`
- **الهدف**: تطبيق تنسيق Excel على الخلية (محاذاة، تنسيق رقم، تاريخ، إلخ) باستخدام PhpSpreadsheet.
- يعتمد على نوع العمود لتعيين `NumberFormat` و `Alignment`.

#### 10. `addMetadataSheet(Spreadsheet $spreadsheet)`
- **الهدف**: إضافة ورقة جديدة باسم "Metadata" تحتوي على البيانات الوصفية (مثل `exported_at`, `total_rows`, `columns_count`).

#### 11. `generatePdfHtml(array $options = []): string`
- **الهدف**: إنشاء محتوى HTML لملف PDF.
- يتضمن عنوانًا، جدولًا بالبيانات، وتنسيق CSS بسيط.

#### 12. `generateFileName(string $extension): string`
- **الهدف**: توليد اسم ملف فريد باستخدام `$tableName` والتاريخ والوقت.
- مثال: `report-orders-2025-04-06-143022.xlsx`

#### 13. `getExportPath(string $fileName): string`
- **الهدف**: إرجاع المسار الكامل للملف داخل `storage_path('app/exports/')`.

#### 14. `ensureExportDirectory(): void`
- **الهدف**: التأكد من وجود مجلد `exports` داخل storage، وإنشائه إذا لم يكن موجودًا.

#### 15. `buildExportResponse(string $format, string $fileName, string $filePath, array $additional = []): array`
- **الهدف**: بناء مصفوفة الاستجابة النهائية للتصدير، تحتوي على:
  - `success`: true
  - `format`: الصيغة
  - `filename`: اسم الملف
  - `filepath`: المسار الكامل
  - `size`: حجم الملف بالبايت
  - `rows`: عدد الصفوف
  - `url`: رابط عام (يفترض وجود route `reports.download`)
  - `download_url`: رابط التحميل (يمكن استخدامه في الواجهة)

#### 16. `getSupportedFormats(): array` (static)
- **الهدف**: إرجاع قائمة بجميع الصيغ المدعومة مع معلومات عنها (label, mime_type, extension, description). مفيدة لعرضها في الواجهة الأمامية.

---

### أمثلة عملية

#### مثال 1: تصدير إلى CSV (مع خيارات مخصصة)
```php
use Nano2\QueryBuilder\Classes\Reporting\ReportQueryExporter;

// بعد تنفيذ التقرير والحصول على $data, $columns, $metadata
$exporter = new ReportQueryExporter($data, $columns, $metadata, 'orders');
$result = $exporter->export('csv', [
    'delimiter' => ';',
    'include_bom' => true,
]);

if ($result['success']) {
    return response()->download($result['filepath'], $result['filename']);
}
```

#### مثال 2: تصدير إلى Excel مع ورقة Metadata
```php
$result = $exporter->export('excel', [
    'sheet_name' => 'تقرير الطلبات',
    'include_metadata' => true,
]);

// يمكن إرجاع رابط التحميل
return response()->json(['download_url' => $result['download_url']]);
```

#### مثال 3: تصدير إلى JSON بشكل مضغوط
```php
$result = $exporter->export('json', ['compact' => true]);
```

#### مثال 4: الحصول على قائمة الصيغ المدعومة
```php
$formats = ReportQueryExporter::getSupportedFormats();
// $formats = [
//     'csv' => ['label' => 'CSV', 'mime_type' => 'text/csv', ...],
//     'excel' => ...
// ]
```

#### مثال 5: استخدام في Controller كامل
```php
class ReportExportController extends Controller
{
    public function export(Request $request, ReportQueryConverter $converter)
    {
        $queryConfig = $request->input('query');
        $format = $request->input('format', 'csv');

        // تنفيذ التقرير
        $result = $converter->execute();

        if (!$result['success']) {
            return response()->json($result, 400);
        }

        // تصدير النتائج
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

#### مثال 6: التعامل مع خطأ التصدير
```php
try {
    $exportResult = $exporter->export('pdf');
} catch (\Exception $e) {
    // استخدام error handler
    $errorHandler = app(ReportQueryErrorHandler::class);
    return $errorHandler->handleExportError($e, 'pdf', ['table' => $tableName]);
}
```

---

### ملاحظات هامة
- **الاعتماديات**: يتطلب تصدير Excel تثبيت مكتبة `phpoffice/phpspreadsheet`. يجب إضافتها عبر Composer.
- **PDF**: التطبيق الحالي لـ PDF غير مكتمل؛ ينشئ HTML فقط. لاستخدام PDF حقيقي، يمكن دمج مكتبة مثل `barryvdh/laravel-dompdf`.
- **تنسيق الأرقام**: يتم تحويل الأرقام العشرية باستخدام `(float) $value` للحفاظ على الدقة.
- **الترميز**: CSV يُضيف BOM (`\xEF\xBB\xBF`) لضمان عرض UTF-8 بشكل صحيح في Excel.
- **الأداء**: لكميات كبيرة من البيانات، قد تحتاج تصدير Excel إلى تحسين (مثل الكتابة في chunks) لتجنب استهلاك الذاكرة.

---

### توسعة الكلاس
يمكن إضافة صيغ جديدة بسهولة عن طريق:
1. إضافة حالة جديدة في `export()`.
2. إنشاء دالة خاصة بالصيغة (مثل `exportToHtml`).
3. تحديث `getSupportedFormats()` static.

---

### الخلاصة
`ReportQueryExporter` يوفر واجهة موحدة ومرنة لتصدير بيانات التقارير إلى صيغ متعددة، مع تنسيق ذكي للبيانات حسب النوع. يعزل منطق التصدير عن باقي النظام، مما يسهل صيانته وتوسيعه. بفضل الخيارات القابلة للتخصيص، يمكن للمطورين تلبية احتياجات متنوعة للمستخدمين النهائيين.