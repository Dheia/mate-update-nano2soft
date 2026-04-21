## توثيق كلاس `ReportSchemaRegistry`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `ReportSchemaRegistry` هو **السجل المركزي (Central Registry)** لنظام التقارير الديناميكي `Nano2.QueryBuilder.Reporting`. يقوم هذا الكلاس بإدارة جميع كائنات `Table` المسجلة في النظام، ويوفر واجهة موحدة للاستعلام عن الجداول وأعمدةها وعلاقاتها. كما يدعم التخزين المؤقت (Caching) لنتائج الاستعلامات المتكررة لتحسين الأداء، ويتيح للمطورين تسجيل جداول جديدة من مصادر متعددة (مثل الإضافات المختلفة).

ببساطة، `ReportSchemaRegistry` هو **المكان الذي تعيش فيه جميع تعريفات الجداول**، وهو الوسيلة التي تستخدمها بقية مكونات النظام (مثل `ReportQueryValidator` و `ReportQueryConverter`) للوصول إلى معلومات المخطط.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$tables` | `array` | مصفوفة تخزن كائنات `Table`، حيث المفتاح هو اسم الجدول. |
| `$cacheEnabled` | `bool` | علم يفعل أو يعطل التخزين المؤقت (افتراضي `true`). |
| `$cacheTtl` | `int` | مدة البقاء الافتراضية للذاكرة المؤقتة بالثواني (افتراضي `3600` = ساعة). |

---

### أهم الطرق (Methods)

#### 1. تسجيل الجداول

##### `public function registerTable(Table $table): void`
- **الهدف**: تسجيل كائن `Table` واحد في السجل.
- **المعاملات**: `$table` - كائن `Table` المراد تسجيله.
- **السلوك**: يضيف الجدول إلى `$tables` باستخدام `$table->getName()` كمفتاح. ثم يستدعي `clearTableCache()` لمسح أي ذاكرة مخبأة لهذا الجدول (لضمان تحديث البيانات).
- **مثال**:
  ```php
  $registry->registerTable($ordersTable);
  ```

##### `public function registerTables(array $tables): void`
- **الهدف**: تسجيل مجموعة من الجداول دفعة واحدة.
- **المعاملات**: `$tables` - مصفوفة من كائنات `Table`.
- **السلوك**: يتكرر على المصفوفة ويستدعي `registerTable()` لكل عنصر.
- **مثال**:
  ```php
  $registry->registerTables([$ordersTable, $usersTable, $productsTable]);
  ```

#### 2. الحصول على الجداول

##### `public function getTable(string $name): ?Table`
- **الهدف**: الحصول على كائن `Table` بالاسم.
- **المعاملات**: `$name` - اسم الجدول.
- **الإرجاع**: كائن `Table` إذا وُجد، وإلا `null`.
- **مثال**:
  ```php
  $orders = $registry->getTable('orders');
  ```

##### `public function isTableRegistered(string $name): bool`
- **الهدف**: التحقق مما إذا كان جدول مسجلاً.
- **المعاملات**: `$name` - اسم الجدول.
- **الإرجاع**: `true` إذا كان موجوداً، وإلا `false`.

##### `public function hasTable(string $name): bool`
- **الهدف**: مرادف لـ `isTableRegistered()` (للراحة).

##### `public function getRegisteredTableNames(): array`
- **الهدف**: الحصول على قائمة بأسماء جميع الجداول المسجلة.

#### 3. الحصول على معلومات الجداول للواجهة الأمامية

##### `public function getAvailableTables(string $extension, ?string $category = null): array`
- **الهدف**: إرجاع قائمة بالجداول المتاحة للواجهة الأمامية، مصفاة حسب الامتداد (extension) والتصنيف (category).
- **المعاملات**:
  - `$extension`: اسم الامتداد (مثل `core`, `fleet`).
  - `$category`: تصنيف الجدول (مثل `sales`, `inventory`) - اختياري.
- **السلوك**:
  - يبحث عن الجداول التي تطابق `$extension` (و `$category` إذا وُجدت).
  - لكل جدول مطابق، يبني مصفوفة تحتوي على: `name`, `label`, `description`, `category`, `extension`, `columns` (من `getTableColumns()`), `relationships` (من `getTableRelationships()`), `auto_join_columns` (من `getAutoJoinColumns()`), `supports_aggregates`, `max_rows`, `has_auto_joins`, `has_manual_joins`.
  - يستخدم التخزين المؤقت (`Cache::get` / `Cache::put`) إذا كان `$cacheEnabled = true`، مع مفتاح مثل `report_tables_{extension}_{category}`.
- **الإرجاع**: مصفوفة من الجداول الجاهزة للإرسال كـ JSON.
- **مثال**:
  ```php
  $coreSalesTables = $registry->getAvailableTables('core', 'sales');
  ```

#### 4. الحصول على أعمدة جدول معين

##### `public function getTableColumns(string $tableName): array`
- **الهدف**: إرجاع جميع الأعمدة المتاحة للاختيار من جدول معين (بما في ذلك أعمدة علاقات auto-join).
- **المعاملات**: `$tableName` - اسم الجدول.
- **السلوك**:
  - يحاول القراءة من الذاكرة المؤقتة باستخدام مفتاح `report_columns_{$tableName}`.
  - إذا لم يكن مخبأً، يحصل على كائن `Table`، ثم يستدعي `$table->getAllAvailableColumns()` للحصول على الأعمدة (مع توسيع العلاقات).
  - يحول كل عمود إلى مصفوفة باستخدام `$column->toArray()`.
  - يخزن النتيجة في الذاكرة المؤقتة إذا كان التخزين مفعلاً.
- **الإرجاع**: مصفوفة من الأعمدة (كل عمود كمصفوفة).
- **مثال**:
  ```php
  $columns = $registry->getTableColumns('orders');
  // النتيجة: [['name' => 'uuid', 'label' => 'المعرف', ...], ['name' => 'customer.name', 'label' => 'العميل - الاسم', ...], ...]
  ```

#### 5. الحصول على علاقات جدول معين

##### `public function getTableRelationships(string $tableName): array`
- **الهدف**: إرجاع جميع العلاقات المتاحة لجدول معين (للاستخدام في الواجهة الأمامية أو للتحقق).
- **المعاملات**: `$tableName` - اسم الجدول.
- **السلوك**:
  - يحاول القراءة من الذاكرة المؤقتة باستخدام مفتاح `report_relationships_{$tableName}`.
  - يحصل على كائن `Table`، ثم يستدعي `$table->getRelationships()`.
  - يحول كل علاقة إلى مصفوفة باستخدام `$relationship->toArray()`، ويضيف خاصية `available_for_manual_join` (دائماً `true` حاليًا).
  - يخزن في الذاكرة المؤقتة.
- **الإرجاع**: مصفوفة من العلاقات.

#### 6. الحصول على أعمدة auto-join (بدون تداخل)

##### `public function getAutoJoinColumns(string $tableName): array`
- **الهدف**: إرجاع أعمدة علاقات auto-join بشكل مسطح (مفيد لبعض الحالات الخاصة).
- **المعاملات**: `$tableName` - اسم الجدول.
- **السلوك**:
  - يحصل على كائن `Table`، ثم يستدعي `$table->getAutoJoinRelationships()`.
  - لكل علاقة، يستدعي `$relationship->getAllAvailableColumns()` ويجمع الأعمدة.
  - يحول كل عمود إلى مصفوفة مع إضافة `auto_join_path` و `relationship` و `column`.
- **الإرجاع**: مصفوفة من الأعمدة (بدون تداخل كامل، ولكن مع معلومات المسار).

#### 7. التحقق من صلاحية عمود

##### `public function isColumnAllowed(string $tableName, string $columnPath): bool`
- **الهدف**: التحقق مما إذا كان عمود بمسار معين (مثل `payload.pickup.city`) موجوداً ومسموحاً به.
- **المعاملات**:
  - `$tableName`: اسم الجدول الأساسي.
  - `$columnPath`: مسار العمود (قد يحتوي على نقاط).
- **السلوك**:
  - إذا كان المسار لا يحتوي على نقاط، يتحقق مباشرة من الجدول باستخدام `$table->isColumnAllowed($columnPath)`.
  - إذا كان به نقاط، يقسم المسار إلى أجزاء (مثل `['payload', 'pickup', 'city']`).
  - يحصل على كائن `Table` للجدول الأساسي.
  - يتنقل عبر الأجزاء (باستثناء الأخير) للعثور على العلاقات المتسلسلة:
    - الجزء الأول يبحث عن علاقة في الجدول الأساسي.
    - الأجزاء التالية يبحث عن علاقات متداخلة في العلاقة السابقة.
    - إذا فشل أي جزء أو كانت العلاقة ليست auto-join، يعيد `false`.
  - بعد الوصول إلى آخر علاقة، يتحقق من وجود العمود النهائي في أعمدة تلك العلاقة.
- **الإرجاع**: `true` إذا كان العمود صالحاً، وإلا `false`.
- **مثال**:
  ```php
  if ($registry->isColumnAllowed('orders', 'payload.pickup.city')) {
      // العمود صالح للاستخدام
  }
  ```

#### 8. إدارة التخزين المؤقت

##### `public function clearTableCache(string $tableName): void`
- **الهدف**: مسح جميع مفاتيح التخزين المؤقت المرتبطة بجدول معين.
- **المعاملات**: `$tableName` - اسم الجدول.
- **السلوك**:
  - يحصل على كائن `Table` لاستخراج `extension` و `category`.
  - ينشئ قائمة بالمفاتيح التي يجب مسحها:
    - `report_columns_{$tableName}`
    - `report_relationships_{$tableName}`
    - `report_tables_{$extension}_all`
    - `report_tables_{$extension}_{$category}` (إذا وُجدت)
  - يستدعي `Cache::forget()` لكل مفتاح.
- **مثال**:
  ```php
  $registry->clearTableCache('orders'); // بعد تحديث تعريف الجدول
  ```

##### `public function clearAllCache(): void`
- **الهدف**: مسح جميع مفاتيح التخزين المؤقت لجميع الجداول المسجلة.
- **السلوك**: يتكرر على جميع الجداول ويستدعي `clearTableCache()` لكل منها.

##### `public function setCacheEnabled(bool $enabled): void`
- **الهدف**: تفعيل أو تعطيل التخزين المؤقت.

##### `public function setCacheTtl(int $ttl): void`
- **الهدف**: تعيين مدة البقاء الافتراضية للذاكرة المؤقتة.

#### 9. دوال مساعدة داخلية

##### `private function flattenRelationshipColumns($relationship, string $path, array $labelTrail): array`
- تُستخدم داخلياً في `getTableColumns` لتوسيع أعمدة العلاقات المتداخلة وإنشاء مسارات مثل `payload.pickup.address`.
- تعدل اسم العمود (`{$path}.{$column->getName()}`) وتسميته باستخدام `$labelTrail` لإنشاء تسمية سياقية (مثل "Pickup Address").

##### `private function shortRelationshipLabel(string $label): string`
- تُستخدم لتوليد بادئة مختصرة للتسمية (مثل "Pickup" من "Pickup Location").

##### `private function getChildRelationship($parentRelationship, string $name)`
- تُستخدم في `isColumnAllowed` للبحث عن علاقة متداخلة في علاقة أب.

---

### أمثلة عملية شاملة

#### مثال 1: تسجيل عدة جداول
```php
use Nano2\QueryBuilder\Classes\Reporting\ReportSchemaRegistry;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Relationship;

// إنشاء registry
$registry = new ReportSchemaRegistry();

// إنشاء جدول orders
$ordersTable = Table::make('orders')
    ->label('الطلبات')
    ->category('sales')
    ->extension('core')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('created_at', 'datetime')->label('تاريخ الإنشاء'),
        Column::make('amount', 'decimal')->label('المبلغ'),
    ]);

// إنشاء جدول users
$usersTable = Table::make('users')
    ->label('المستخدمين')
    ->category('users')
    ->extension('core')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('name')->label('الاسم'),
        Column::make('email')->label('البريد'),
    ]);

// تسجيل الجداول
$registry->registerTables([$ordersTable, $usersTable]);
```

#### مثال 2: الحصول على أعمدة جدول معين (للواجهة الأمامية)
```php
// في Controller
public function getTableColumns(Request $request, ReportSchemaRegistry $registry)
{
    $tableName = $request->input('table');
    
    if (!$registry->hasTable($tableName)) {
        return response()->json(['error' => 'Table not found'], 404);
    }
    
    $columns = $registry->getTableColumns($tableName);
    return response()->json($columns);
}
```

#### مثال 3: الحصول على الجداول المتاحة لامتداد معين
```php
$coreTables = $registry->getAvailableTables('core');
$fleetTables = $registry->getAvailableTables('fleet');

// مع تصنيف محدد
$salesTables = $registry->getAvailableTables('core', 'sales');
```

#### مثال 4: التحقق من صلاحية عمود (قبل بناء الاستعلام)
```php
function validateColumn($tableName, $columnPath, ReportSchemaRegistry $registry) {
    if (!$registry->isColumnAllowed($tableName, $columnPath)) {
        throw new \Exception("العمود {$columnPath} غير مسموح به");
    }
}

// مثال:
validateColumn('orders', 'payload.pickup.city', $registry); // يمر إذا كان مسار صحيح
validateColumn('orders', 'non_existent', $registry); // يرمي خطأ
```

#### مثال 5: استخدام isColumnAllowed مع مسارات مختلفة
```php
// مسار مباشر
$registry->isColumnAllowed('orders', 'amount'); // true (إذا كان موجوداً)

// مسار بعلاقة واحدة
$registry->isColumnAllowed('orders', 'customer.name'); // true (إذا كانت علاقة customer موجودة ولديها عمود name)

// مسار بعلاقتين متداخلتين
$registry->isColumnAllowed('orders', 'payload.pickup.city'); // true (إذا كانت المسارات صحيحة)

// مسار غير موجود
$registry->isColumnAllowed('orders', 'payload.invalid.city'); // false
```

#### مثال 6: إدارة التخزين المؤقت
```php
// تعطيل التخزين المؤقت مؤقتاً (مثلاً أثناء التطوير)
$registry->setCacheEnabled(false);

// تغيير مدة البقاء إلى 30 دقيقة
$registry->setCacheTtl(1800);

// بعد تعديل تعريف جدول، نمسح الذاكرة المخبأة له
$registry->clearTableCache('orders');

// أو مسح كل شيء
$registry->clearAllCache();
```

#### مثال 7: استخدام registry في ReportQueryValidator
```php
class ReportQueryValidator
{
    protected ReportSchemaRegistry $registry;
    
    public function __construct(ReportSchemaRegistry $registry)
    {
        $this->registry = $registry;
    }
    
    public function validate(array $queryConfig)
    {
        // التحقق من وجود الجدول
        if (!$this->registry->hasTable($queryConfig['table']['name'])) {
            $this->errors[] = "الجدول غير موجود";
        }
        
        // التحقق من الأعمدة
        foreach ($queryConfig['columns'] as $column) {
            if (!$this->registry->isColumnAllowed($queryConfig['table']['name'], $column['name'])) {
                $this->errors[] = "العمود {$column['name']} غير مسموح به";
            }
        }
        
        // ... بقية التحقق
    }
}
```

#### مثال 8: تسجيل جداول من إضافة (Extension) باستخدام واجهة ReportSchema
```php
// كلاس ينفذ واجهة ReportSchema
class CoreReportSchema implements \Nano2\QueryBuilder\Classes\Reporting\Contracts\ReportSchema
{
    public function registerReportSchema(ReportSchemaRegistry $registry): void
    {
        $ordersTable = Table::make('orders')...;
        $usersTable = Table::make('users')...;
        
        $registry->registerTables([$ordersTable, $usersTable]);
    }
}

// في Service Provider
public function boot()
{
    $registry = $this->app->make(ReportSchemaRegistry::class);
    
    // استدعاء جميع الكلاسات التي تنفذ ReportSchema
    foreach ($this->app->tagged('report-schemas') as $schema) {
        $schema->registerReportSchema($registry);
    }
}
```

#### مثال 9: الحصول على جميع أسماء الجداول المسجلة
```php
$tableNames = $registry->getRegisteredTableNames();
// ['orders', 'users', 'products', ...]
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `Table`**: `ReportSchemaRegistry` يدير مجموعة من كائنات `Table`. يستخدم دوال `Table` مثل `getName()`، `getAllAvailableColumns()`، `getRelationships()`، `isColumnAllowed()`، `getAutoJoinRelationships()`.
- **مع `Relationship`**: عند توسيع الأعمدة في `getTableColumns()`، يستدعي دوال `Relationship` مثل `getAllAvailableColumns()` و `getName()` و `getLabel()`.
- **مع `Column`**: يحول الأعمدة إلى مصفوفات باستخدام `toArray()`، ويستخدم `copyWith()` داخلياً في `flattenRelationshipColumns()`.
- **مع `ReportQueryValidator` و `ReportQueryConverter`**: يعتمدان على `ReportSchemaRegistry` للوصول إلى معلومات المخطط والتحقق من صحة البيانات.

---

### أفضل الممارسات

1. **سجل الجداول في Service Provider**: قم بتسجيل جميع الجداول مرة واحدة عند بدء التطبيق (في `boot()` أو `register()`).
2. **استخدم واجهة `ReportSchema`**: لتوحيد طريقة تسجيل الجداول من الإضافات المختلفة.
3. **فعّل التخزين المؤقت في الإنتاج**: مع TTL مناسب (مثل 3600 ثانية) لتقليل الحمل على قاعدة البيانات.
4. **امسح الذاكرة المخبأة بعد تحديث تعريف أي جدول**: لتجنب عرض بيانات قديمة.
5. **استخدم `isColumnAllowed` للتحقق قبل بناء الاستعلام**: هذا يمنع الأخطاء ويحسن الأمان.
6. **نظم الجداول باستخدام `category` و `extension`**: لتسهيل تصفيتها في الواجهة الأمامية.

---

### الخلاصة
`ReportSchemaRegistry` هو قلب نظام التقارير الديناميكي. يقوم بتجميع كل معرفة النظام عن الجداول المتاحة وعلاقاتها، ويوفر واجهة برمجية نظيفة وفعالة للوصول إلى هذه المعلومات. بفضل دعمه للتخزين المؤقت والتحقق من صلاحية الأعمدة، يمكن للمطورين بناء أنظمة تقارير قوية وآمنة دون القلق بشأن الأداء أو صحة البيانات. يتكامل هذا الكلاس بشكل وثيق مع `Table` و `Relationship` و `Column` ليشكل معاً طبقة تعريفية متكاملة تمكن المستخدمين النهائيين من إنشاء تقارير معقدة بسهولة.