## توثيق كلاس `ReportsManager` والسمة `ReportsManagerQueryConverter`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.


### مقدمة
يشكل كلاس `ReportsManager` القلب النابض لإدارة تعريفات الجداول في نظام التقارير الديناميكي `Nano2.QueryBuilder`. يعمل هذا الكلاس كطبقة وسطى (Facade) فوق `ReportSchemaRegistry`، حيث يقوم بجمع تعريفات الجداول من مصادر متعددة (الإضافات عبر `registerReportTables`، أو التسجيل المباشر، أو الـ callbacks)، وتخزينها بشكل خام، ثم تحويلها إلى كائنات `Table` وتسجيلها في `ReportSchemaRegistry` عند الحاجة. يدعم الكلاس التحميل البطيء (Lazy Loading) والتخزين المؤقت للكائنات الجاهزة، بالإضافة إلى فلترة الجداول بناءً على صلاحيات المستخدم.

أما السمة `ReportsManagerQueryConverter` فتوفر مجموعة متكاملة من الوظائف لتنفيذ التقارير والتحقق منها ومعالجة أخطائها وتصديرها، بالاعتماد على الكلاسات `ReportQueryValidator`، `ReportQueryConverter`، `ReportQueryErrorHandler`، و `ReportQueryExporter`. بفضل هذه السمة، يمكن لكلاس `ReportsManager` (أو أي كلاس آخر يستخدمها) أن يصبح واجهة شاملة لتشغيل التقارير بسهولة وأمان.

---

## 1. كلاس `ReportsManager`

### الغرض
- تسجيل تعريفات الجداول (كبيانات خام) من الإضافات المختلفة.
- تحويل التعريفات الخام إلى كائنات `Table` جاهزة للاستخدام.
- إدارة دورة حياة هذه الكائنات (تخزين مؤقت، تحديث، إزالة).
- توفير واجهة للوصول إلى `ReportSchemaRegistry` وجداوله المسجلة.
- فلترة الجداول حسب الصلاحيات والامتداد والتصنيف.

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$registry` | `ReportSchemaRegistry` | مرجع إلى سجل المخطط الرئيسي (يتم تهيئته عند الحاجة). |
| `$registryInitialized` | `bool` | علم يشير إلى تهيئة `$registry`. |
| `$rawDefinitions` | `array` | مصفوفة من التعريفات الخام للجداول (مفهرسة باسم الجدول). |
| `$builtTables` | `array` | مصفوفة من كائنات `Table` الجاهزة (مخبأة). |
| `$loaded` | `bool` | علم يشير إلى تحميل التعريفات من الإضافات. |
| `$defaultTableDefinition` | `array` | تعريف افتراضي للجدول (يُدمج مع التعريفات المقدمة). |
| `$defaultColumnDefinition` | `array` | تعريف افتراضي للعمود. |
| `$defaultRelationshipDefinition` | `array` | تعريف افتراضي للعلاقة. |
| `static $callbacks` | `array` | مصفوفة من دوال الـ callback للتسجيل اليدوي. |

### أهم الطرق

#### أ. التسجيل والتحميل

##### `registerDefinitions(array $definitions)`
- **الوصف**: تسجيل مجموعة من تعريفات الجداول دفعة واحدة.
- **المعاملات**: `$definitions` مصفوفة مفتاحها اسم الجدول وقيمتها تعريف الجدول.
- **مثال**:
  ```php
  $manager->registerDefinitions([
      'orders' => [
          'label' => 'الطلبات',
          'columns' => ['id', 'amount'],
          'relationships' => [
              'customer' => ['table' => 'customers', 'columns' => ['name']]
          ]
      ]
  ]);
  ```

##### `registerDefinition(string $tableName, array $definition)`
- **الوصف**: تسجيل تعريف جدول واحد مع دمج القيم الافتراضية ومعالجة الأعمدة والعلاقات.

##### `static registerCallback(callable $callback)`
- **الوصف**: تسجيل دالة callback تُنفذ عند تحميل التعريفات. تستقبل الدالة كائن `ReportsManager`.
- **مثال**:
  ```php
  ReportsManager::registerCallback(function($manager) {
      $manager->registerDefinition('products', ['label' => 'المنتجات']);
  });
  ```

##### `registerTable(Table $table)`
- **الوصف**: تسجيل كائن `Table` مباشرة. يُضاف إلى `$builtTables` ويُسجل في الـ registry إذا كان مهيأً.

##### `loadDefinitions()`
- **الوصف**: تحميل التعريفات من الإضافات (التي تطبق `registerReportTables`) والـ callbacks. تُستدعى تلقائياً عند الحاجة.

#### ب. الوصول إلى Registry والجداول

##### `getRegistry(): ReportSchemaRegistry`
- **الوصف**: الحصول على كائن `ReportSchemaRegistry` (مع تهيئته إذا لزم الأمر).

##### `getTableNames(): array`
- **الوصف**: إرجاع أسماء جميع الجداول المسجلة (من التعريفات الخام).

##### `getTableDefinition(string $tableName): ?array`
- **الوصف**: الحصول على تعريف جدول خام.

##### `hasTable(string $tableName): bool`
- **الوصف**: التحقق من وجود جدول.

##### `getTableNamesByExtension(string $extension): array`
- **الوصف**: إرجاع أسماء الجداول التي تنتمي إلى امتداد معين.

##### `getTableNamesByCategory(string $category): array`
- **الوصف**: إرجاع أسماء الجداول التي تنتمي إلى تصنيف معين.

##### `getAvailableTablesForUser(string $extension, ?string $category = null): array`
- **الوصف**: إرجاع الجداول المتاحة للمستخدم الحالي (بعد فلترة الصلاحيات) من الـ registry مباشرة. تستفيد من التخزين المؤقت في الـ registry.
- **مثال**:
  ```php
  $tables = $manager->getAvailableTablesForUser('core', 'sales');
  ```

##### `getBuiltTable(string $tableName): ?Table`
- **الوصف**: الحصول على كائن `Table` جاهز (من الذاكرة المخبأة أو بنائه إذا لزم الأمر).

##### `getBuiltTables(): array`
- **الوصف**: الحصول على جميع كائنات `Table` المخبأة (مع بناء غير المبني منها).

#### ج. تحديث وإزالة التعريفات

##### `updateDefinition(string $tableName, array $newDefinition)`
- **الوصف**: تحديث تعريف جدول موجود، وإعادة تطبيقه على الـ registry إذا كان مهيأً.

##### `applyDefinitionToRegistry(string $tableName)`
- **الوصف**: إعادة بناء جدول معين وتسجيله في الـ registry (يُستخدم بعد التحديث).

##### `removeDefinition(string $tableName)`
- **الوصف**: إزالة تعريف جدول ومسح الكاش المرتبط به.

##### `clearBuiltTables()`
- **الوصف**: مسح الذاكرة المخبأة للكائنات الجاهزة.

#### د. دوال الوصول إلى Registry مباشرة

##### `getTableColumns(string $tableName): array`
- **الوصف**: الحصول على أعمدة الجدول (بما في ذلك أعمدة العلاقات) من الـ registry.

##### `isColumnAllowed(string $tableName, string $columnPath): bool`
- **الوصف**: التحقق من صلاحية عمود بمسار معين.

##### `getTableSchemaForUI(string $tableName): array`
- **الوصف**: الحصول على معلومات كاملة عن الجدول (جدول، أعمدة، علاقات) مناسبة للواجهة الأمامية.

##### `getAvailableTables(string $extension, ?string $category = null): array`
- **الوصف**: الحصول على جميع الجداول المتاحة لامتداد وتصنيف معينين (بدون فلترة صلاحيات).

---

## 2. السمة `ReportsManagerQueryConverter`

### الغرض
توفير وظائف متكاملة لتشغيل التقارير: التحقق من صحة التكوين، التنفيذ، معالجة الأخطاء، التصدير، التخزين المؤقت، وخيارات متقدمة مثل إرجاع SQL فقط أو تقدير التكلفة.

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$queryValidator` | `ReportQueryValidator` | كائن للتحقق من صحة تكوين التقرير (يُهيأ عند الحاجة). |
| `$queryErrorHandler` | `ReportQueryErrorHandler` | كائن لمعالجة الأخطاء. |
| `$cacheDriver` | `Cache\Repository` | مُخزن مؤقت اختياري لنتائج التقارير. |
| `$queryLogger` | `callable` | دالة لتسجيل الاستعلامات المنفذة. |
| `$lastConverter` | `ReportQueryConverter` | آخر محول استعلام تم استخدامه (للوصول إلى معلوماته). |

### أهم الطرق

#### أ. التحقق والتنفيذ الأساسي

##### `validateReportQuery(array $queryConfig): array`
- **الوصف**: التحقق من صحة تكوين التقرير باستخدام `ReportQueryValidator`.
- **الإرجاع**: مصفوفة تحتوي على `valid`, `errors`, `warnings`, `summary`.

##### `executeReportQuery(array $queryConfig): array`
- **الوصف**: تنفيذ الاستعلام وإرجاع النتائج (مع تسجيل وقت التنفيذ إذا تم تفعيل `queryLogger`).
- **الإرجاع**: نفس نتيجة `ReportQueryConverter::execute()`.

##### `handleReportError(\Throwable $exception, array $context = []): array`
- **الوصف**: معالجة أي استثناء وتحويله إلى استجابة منظمة باستخدام `ReportQueryErrorHandler`.

##### `handleReportValidationError(array $validationResult, array $context = []): array`
- **الوصف**: معالجة أخطاء التحقق وإرجاع استجابة منظمة.

#### ب. دوال متقدمة لتشغيل التقارير

##### `runReport(array $queryConfig, array $options = []): array`
- **الوصف**: دالة شاملة لتشغيل تقرير مع خيارات متقدمة.
- **الخيارات**:
  - `skip_validation` (bool): تخطي مرحلة التحقق (افتراضي false).
  - `return_sql_only` (bool): إرجاع كائن SQL فقط دون تنفيذ (افتراضي false).
  - `dry_run` (bool): تقدير التكلفة فقط (افتراضي false).
- **الإرجاع**: إما نتيجة التقرير، أو معلومات SQL، أو تقدير التكلفة.
- **مثال**:
  ```php
  $result = $manager->runReport($queryConfig, ['return_sql_only' => true]);
  ```

##### `runReportWithCache(array $queryConfig, int $ttl = 3600, array $options = []): array`
- **الوصف**: تشغيل التقرير مع دعم التخزين المؤقت. يبحث في `$cacheDriver` أولاً، ثم ينفذ ويخزن النتيجة إذا كانت ناجحة.

##### `runReportAndExport(array $queryConfig, string $format, array $exportOptions = []): array`
- **الوصف**: تنفيذ التقرير ثم تصدير النتائج بالصيغة المطلوبة باستخدام `ReportQueryExporter`.
- **المعاملات**:
  - `$format`: صيغة التصدير (csv, excel, json, pdf, xml).
  - `$exportOptions`: خيارات إضافية للتصدير.
- **الإرجاع**: نتيجة التصدير (تحتوي على `filename`, `filepath`, `url`, إلخ).

##### `estimateQueryCost(array $queryConfig): array`
- **الوصف**: تقدير تكلفة الاستعلام (تعقيد، أداء متوقع، عدد الأعمدة/الـ joins/الشروط) دون تنفيذه.

#### ج. دوال مساعدة

##### `validateTableAccess(?string $tableName, $user = null): bool`
- **الوصف**: التحقق من صلاحية المستخدم للوصول إلى جدول معين (بناءً على صلاحيات الجدول المسجلة).

##### `buildQueryConfigFromInput(array $input): array`
- **الوصف**: تحويل مدخلات الطلب (عادة من JSON) إلى بنية `queryConfig` موحدة.

##### `enableCache($cacheDriver): void`
- **الوصف**: تفعيل التخزين المؤقت بتعريف `cacheDriver`.

##### `enableQueryLogging(callable $logger): void`
- **الوصف**: تفعيل تسجيل الاستعلامات عبر دالة تستقبل (`$queryConfig`, `$result`, `$executionTime`).

##### `formatError(array $errorResponse, string $outputType = 'json')`
- **الوصف**: تنسيق الخطأ لمخرجات مختلفة (json, html, text).

##### `formatForDataTables(array $result, int $draw = 1): array`
- **الوصف**: تنسيق نتائج التقرير لتناسب مكتبة DataTables في الواجهة الأمامية.

##### `getLastQueryAutoJoins(): array`
- **الوصف**: الحصول على معلومات الـ auto-joins التي تم تطبيقها في آخر استعلام.

##### `getLastQueryJoinAliases(): array`, `getLastQueryManualJoins(): array`
- **الوصف**: الحصول على معلومات الـ aliases والـ manual joins لآخر استعلام.

##### `fireEvent($event, $payload)`
- **الوصف**: إطلاق حدث (Event) لتمكين الإضافات من التفاعل مع عملية تشغيل التقرير. الأحداث المتاحة: `reports.beforeRun`, `reports.afterRun`.

---

## 3. أمثلة عملية

#### مثال 1: تسجيل جدول عبر إضافة (في Plugin.php)

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
                'label' => 'الطلبات',
                'category' => 'sales',
                'extension' => 'orders',
                'columns' => [
                    'uuid' => ['label' => 'المعرف', 'hidden' => true],
                    'created_at' => ['type' => 'datetime', 'label' => 'تاريخ الإنشاء'],
                    'amount' => ['type' => 'decimal', 'label' => 'المبلغ', 'aggregatable' => true],
                ],
                'relationships' => [
                    'customer' => [
                        'table' => 'customers',
                        'label' => 'العميل',
                        'auto_join' => true,
                        'columns' => [
                            'name' => ['label' => 'الاسم'],
                            'email' => ['label' => 'البريد'],
                        ]
                    ]
                ],
                'permissions' => ['nano.orders.access_reports'],
            ]
        ];
    }
}
```

#### مثال 2: استخدام `ReportsManager` في Controller لجلب أعمدة جدول

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

#### مثال 3: تشغيل تقرير مع التحقق الأساسي

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

#### مثال 4: استخدام خيارات متقدمة (إرجاع SQL فقط)

```php
$sqlInfo = $manager->runReport($queryConfig, ['return_sql_only' => true]);
// $sqlInfo = [
//     'success' => true,
//     'sql' => 'select ...',
//     'bindings' => [...],
//     'auto_joins' => [...]
// ]
```

#### مثال 5: تقدير تكلفة الاستعلام

```php
$cost = $manager->estimateQueryCost($queryConfig);
if ($cost['complexity'] === 'high') {
    // تحذير المستخدم
}
```

#### مثال 6: تصدير نتائج التقرير

```php
$exportResult = $manager->runReportAndExport($queryConfig, 'excel', [
    'sheet_name' => 'تقرير الطلبات'
]);

if ($exportResult['success']) {
    return response()->download($exportResult['filepath']);
}
```

#### مثال 7: استخدام التخزين المؤقت

```php
$manager->enableCache(Cache::store('redis'));
$cachedResult = $manager->runReportWithCache($queryConfig, 1800); // 30 دقيقة
```

#### مثال 8: تنسيق النتائج لـ DataTables

```php
$result = $manager->runReport($queryConfig);
$dataTablesResponse = $manager->formatForDataTables($result, request('draw'));
return response()->json($dataTablesResponse);
```

---

## 4. العلاقات مع الكلاسات الأخرى

- **`ReportSchemaRegistry`**: `ReportsManager` يملك ويغذي هذا الـ registry بالجداول المسجلة. جميع دوال الوصول إلى الأعمدة والعلاقات والتحقق من الصلاحية تستخدم الـ registry بعد تهيئته.
- **`Table`, `Column`, `Relationship`**: يستخدم `ReportsManager` هذه الكلاسات لبناء الكائنات الجاهزة من التعريفات الخام.
- **`ReportQueryValidator` و `ReportQueryConverter` و `ReportQueryErrorHandler` و `ReportQueryExporter`**: تستخدمها السمة `ReportsManagerQueryConverter` لتنفيذ وظائفها المتقدمة.
- **`PluginManager` (من OctoberCMS)**: يستخدم `ReportsManager` للكشف عن الإضافات التي تطبق `registerReportTables`.
- **`BackendAuth`**: يستخدم لفلترة الجداول حسب صلاحيات المستخدم.

---

## 5. خاتمة

يشكل كلاس `ReportsManager` مع السمة `ReportsManagerQueryConverter` حلاً متكاملاً لإدارة وتشغيل التقارير الديناميكية في تطبيقات OctoberCMS. بفضل تصميمه النمطي، يمكن للمطورين:

- تسجيل جداول التقارير بسهولة من أي إضافة باستخدام تعريفات بسيطة.
- التحكم في دورة حياة هذه التعريفات وتحديثها.
- تنفيذ التقارير مع خيارات متعددة (تحقق، تخزين مؤقت، تصدير، تقدير تكلفة).
- توفير واجهات برمجية نظيفة للواجهة الأمامية (مثل الحصول على أعمدة الجدول، تشغيل التقارير، تصديرها).
- ضمان الأمان عبر فلترة الصلاحيات ونطاق الشركة (المدمج في `ReportQueryConverter`).

هذا التصميم يجعل من السهل بناء نظام تقارير قوي ومرن يلبي احتياجات المستخدمين المتقدمة دون المساس بأداء النظام أو سلامته.