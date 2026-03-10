## توثيق كلاسات نظام التقارير الديناميكي في NanoSoft App - حزمة Nano2.QueryBuilder

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
تهدف مجموعة الكلاسات `ReportSchemaRegistry` و `Table` و `Relationship` و `Column` إلى بناء **طبقة تعريفية (Schema)** لأنظمة التقارير الديناميكية في تطبيقات NanoSoft App، كجزء من حزمة **Nano2.QueryBuilder** المطورة بواسطة **شركة نانوسوفت (NanoSoft)**. تسمح هذه الطبقة بتسجيل الجداول المتاحة للتقارير مع أعمدةها وعلاقاتها، وتوفر واجهة موحدة للاستعلام عن هذه المعلومات، مما يسهل بناء واجهات المستخدم واستعلامات SQL الديناميكية بطريقة آمنة ومنظمة.

---

## 1. كلاس `Column` (تمثيل العمود)

### الغرض
يمثل عموداً في جدول قاعدة البيانات أو عموداً محسوباً (Computed Column) يُشتق من تعبير SQL. يحتوي على جميع الخصائص اللازمة لوصف العمود وإمكانياته (بحث، ترتيب، تصفية، تجميع) بالإضافة إلى دالة تحويل مخصصة (Transformer).

### الخصائص الأساسية
- **name**: اسم العمود الفعلي في قاعدة البيانات (مثل `created_at`).
- **label**: تسمية عرض للمستخدم (مثل "تاريخ الإنشاء").
- **type**: نوع البيانات (string, integer, decimal, date, datetime, boolean, ...).
- **description**: وصف اختياري للعمود.
- **format**: تنسيق عرض اختياري (مثل `Y-m-d` للتاريخ).
- **nullable**: هل يمكن أن تكون القيمة فارغة؟
- **searchable, sortable, filterable**: إمكانيات العمود.
- **aggregatable**: هل يمكن استخدامه في دوال التجميع (SUM, AVG)؟
- **hidden**: إخفاء العمود من الواجهة (يبقى متاحاً للاستعلام).
- **computed**: هل هو عمود محسوب؟
- **computation**: تعبير SQL للحساب (إذا كان computed).
- **transformer**: دالة PHP لتحويل القيمة قبل العرض.
- **meta**: بيانات إضافية على شكل key-value.

### أهم الطرق

#### إنشاء عمود عادي
```php
public static function make(string $name, string $type = 'string'): self
```
مثال:
```php
$column = Column::make('status', 'string')
    ->label('الحالة')
    ->searchable(true)
    ->sortable(true)
    ->filterable(true);
```

#### تعيين الخصائص
- `label(string $label)`
- `description(string $description)`
- `format(string $format)`
- `nullable(bool $nullable = true)`
- `searchable(bool $searchable = true)`
- `sortable(bool $sortable = true)`
- `filterable(bool $filterable = true)`
- `aggregatable(bool $aggregatable = true)`
- `hidden(bool $hidden = true)`
- `meta(string $key, $value)`
- `setMeta(array $meta)`

#### إنشاء أعمدة محسوبة (Computed Columns)
```php
public static function computed(string $name, string $computation, string $type = 'string', array $options = []): self
```
مثال:
```php
$totalColumn = Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
    ->label('الإجمالي شامل الضريبة')
    ->aggregatable(true);
```

#### دوال مساعدة للأعمدة التجميعية
- `Column::count(string $name, string $countField = '*')`: عمود COUNT.
- `Column::sum(string $name, string $sumField)`: عمود SUM.
- `Column::avg(string $name, string $avgField)`: عمود AVG.
- `Column::max(string $name, string $maxField)`: عمود MAX.
- `Column::min(string $name, string $minField)`: عمود MIN.

مثال:
```php
$orderCount = Column::count('order_count', 'uuid')
    ->label('عدد الطلبات');
```

#### دوال التحويل (Transformer)
يمكن تسجيل دالة لتحويل قيمة العمود قبل إرسالها للواجهة:
```php
public function transformer(\Closure|callable $transformer): self
```
مثال:
```php
$column->transformer(function ($value) {
    return $value ? 'نشط' : 'غير نشط';
});
```

#### دوال الوصول (Getters)
- `getName()`, `getLabel()`, `getType()`, `getDescription()`, `getFormat()`
- `isNullable()`, `isSearchable()`, `isSortable()`, `isFilterable()`, `isAggregatable()`, `isHidden()`, `isComputed()`
- `getComputation()`, `getTransformer()`, `getMeta()`

#### دوال إضافية
- `isForeignKey()`: هل العمود مفتاح أجنبي (ينتهي بـ `_uuid` أو `_id`)؟
- `transformValue($value)`: تطبيق الـ transformer على قيمة.
- `toArray()`: تحويل العمود إلى مصفوفة (لإرسالها عبر API).
- `copyWith(array $overrides)`: إنشاء نسخة معدلة من العمود (مفيد للعلاقات المتداخلة).

### مثال كامل لاستخدام Column
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;

$columns = [
    Column::make('uuid', 'string')->label('المعرف')->hidden(true),
    Column::make('created_at', 'datetime')
        ->label('تاريخ الإنشاء')
        ->format('Y-m-d H:i')
        ->sortable(true),
    Column::make('status', 'string')
        ->label('الحالة')
        ->filterable(true)
        ->transformer(fn($v) => $v === 'completed' ? 'مكتمل' : 'قيد التنفيذ'),
    Column::computed('total', 'SUM(amount)', 'decimal')
        ->label('الإجمالي')
        ->aggregatable(true),
];
```

---

## 2. كلاس `Relationship` (تمثيل العلاقة)

### الغرض
يمثل علاقة بين جدولين، مثل `belongsTo` أو `hasMany`. يوفر معلومات عن الجدول المرتبط، مفاتيح الربط، نوع الجوين، والأعمدة المتاحة من تلك العلاقة. يدعم **العلاقات المتداخلة** (nested relationships) وخاصية **auto-join** التي تسمح بإضافة الجوين تلقائياً عند اختيار أعمدة من العلاقة.

### الخصائص الأساسية
- **name**: اسم العلاقة (مثل `customer`, `payload`).
- **table**: اسم الجدول المرتبط في قاعدة البيانات.
- **label**: تسمية عرض للعلاقة (مثل "العميل").
- **type**: نوع الجوين (`left`, `right`, `inner`).
- **localKey**: المفتاح المحلي في الجدول الحالي (افتراضياً `{name}_uuid`).
- **foreignKey**: المفتاح الأجنبي في الجدول المرتبط (افتراضياً `uuid`).
- **enabled**: هل العلاقة مفعلة؟
- **autoJoin**: هل تضاف تلقائياً عند اختيار أعمدة منها؟
- **description**: وصف اختياري.
- **columns**: مصفوفة من كائنات `Column` تمثل أعمدة الجدول المرتبط.
- **nestedRelationships**: مصفوفة من كائنات `Relationship` تمثل علاقات داخل الجدول المرتبط (مثل `payload.pickup`).
- **meta**: بيانات إضافية.

### أهم الطرق

#### إنشاء علاقة
```php
public static function make(string $name, string $table, string $type = 'left'): self
```
مثال:
```php
$customerRel = Relationship::make('customer', 'customers')
    ->label('العميل')
    ->localKey('customer_uuid')
    ->foreignKey('uuid')
    ->joinType('left');
```

#### أنواع العلاقات المساعدة
- `Relationship::belongsTo(string $name, string $table)`: مرادف لـ `make` مع type = 'left'.
- `Relationship::hasMany(string $name, string $table)`: مرادف.
- `Relationship::hasOne(string $name, string $table)`: مرادف.
- `Relationship::hasAutoJoin(string $name, string $table, string $type = 'left')`: لإنشاء علاقة مع تفعيل auto-join.

#### تعيين المفاتيح
```php
public function localKey(string $localKey): self
public function foreignKey(string $foreignKey): self
```

#### إضافة أعمدة إلى العلاقة
```php
public function columns(array $columns): self
public function addColumn(Column $column): self
```
مثال:
```php
$customerRel->columns([
    Column::make('name')->label('الاسم'),
    Column::make('email')->label('البريد'),
]);
```

#### إضافة علاقات متداخلة
```php
public function with(array $relationships): self
public function addNestedRelationship(Relationship $relationship): self
```
مثال:
```php
$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->columns(['weight', 'dimensions'])
    ->with([
        Relationship::hasAutoJoin('pickup', 'addresses')->columns(['address', 'city'])
    ]);
```

#### تفعيل auto-join
```php
public function autoJoin(bool $autoJoin = true): self
// أو استخدام hasAutoJoin عند الإنشاء
```

#### دوال الوصول
- `getName()`, `getTable()`, `getLabel()`, `getType()`, `getLocalKey()`, `getForeignKey()`
- `isEnabled()`, `isAutoJoin()`
- `getColumns()`, `getNestedRelationships()`
- `getAllAvailableColumns()`: إرجاع جميع الأعمدة بما في ذلك أعمدة العلاقات المتداخلة (مع بادئة المسار).
- `getAutoJoinRelationships()`: إرجاع العلاقات المتداخلة التي تحمل خاصية auto-join.
- `hasNestedRelationships()`

#### تحويل إلى مصفوفة
```php
public function toArray(): array
```

### مثال كامل لبناء علاقة مع تداخل
```php
$addressColumns = [
    Column::make('address')->label('العنوان'),
    Column::make('city')->label('المدينة'),
    Column::make('country')->label('البلد'),
];

$pickupRel = Relationship::hasAutoJoin('pickup', 'addresses')
    ->label('موقع الاستلام')
    ->columns($addressColumns);

$dropoffRel = Relationship::hasAutoJoin('dropoff', 'addresses')
    ->label('موقع التسليم')
    ->columns($addressColumns);

$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->label('بيانات الشحنة')
    ->columns([
        Column::make('weight')->label('الوزن')->type('decimal'),
        Column::make('dimensions')->label('الأبعاد'),
    ])
    ->with([$pickupRel, $dropoffRel]);

$orderRel = Relationship::belongsTo('customer', 'customers')
    ->columns([
        Column::make('name')->label('الاسم'),
        Column::make('email')->label('البريد'),
    ]);
```

---

## 3. كلاس `Table` (تمثيل الجدول)

### الغرض
يمثل جدولاً رئيسياً في قاعدة البيانات متاحاً للتقارير. يجمع بين الأعمدة العادية، الأعمدة المحسوبة، والعلاقات مع الجداول الأخرى. يوفر طرقاً للوصول إلى الأعمدة القابلة للعرض والتحقق من صلاحياتها.

### الخصائص الأساسية
- **name**: اسم الجدول في قاعدة البيانات.
- **label**: تسمية عرض للجدول.
- **description**: وصف اختياري.
- **category**: تصنيف الجدول (مثل `sales`, `inventory`).
- **extension**: الامتداد (extension) الذي ينتمي إليه الجدول (مثل `core`, `fleet`).
- **columns**: مصفوفة من `Column` للأعمدة العادية.
- **computedColumns**: مصفوفة من `Column` للأعمدة المحسوبة (تظهر فقط عند استخدامها في التجميع أو كحقول محسوبة).
- **relationships**: مصفوفة من `Relationship`.
- **excludedColumns**: قائمة بأسماء الأعمدة المستبعدة من العرض (تبقى متاحة للاستعلام).
- **supportsAggregates**: هل يدعم دوال التجميع؟
- **maxRows**: الحد الأقصى للصفوف المسموح بها.
- **cacheable**: هل يمكن تخزين نتائج الاستعلام مؤقتاً؟
- **cacheTtl**: مدة التخزين المؤقت.
- **permissions**: صلاحيات الوصول للجدول.
- **meta**: بيانات إضافية.

### أهم الطرق

#### إنشاء جدول
```php
public static function make(string $name): self
```
مثال:
```php
$ordersTable = Table::make('orders')
    ->label('الطلبات')
    ->category('sales')
    ->extension('core');
```

#### إضافة الأعمدة
```php
public function columns(array $columns): self
public function addColumn(Column $column): self
```
يمكن تمرير مصفوفة من كائنات `Column` أو أسماء أعمدة نصية (يتم تحويلها تلقائياً إلى كائنات Column بنوع string).

مثال:
```php
$ordersTable->columns([
    'uuid',   // يتحول إلى Column::make('uuid')
    'created_at',
    Column::make('amount', 'decimal')->label('المبلغ'),
]);
```

#### إضافة الأعمدة المحسوبة
```php
public function computedColumns(array $columns): self
public function addComputedColumn(Column $column): self
```

#### إضافة العلاقات
```php
public function relationships(array $relationships): self
public function addRelationship(Relationship $relationship): self
```
مثال:
```php
$ordersTable->relationships([
    $customerRel,
    $payloadRel,
]);
```

#### استبعاد أعمدة من العرض
```php
public function excludeColumns(array $columnNames): self
```
مثال:
```php
$ordersTable->excludeColumns(['deleted_at', 'internal_note']);
```

#### تحديد الخصائص العامة
- `supportsAggregates(bool $supports = true)`
- `maxRows(int $maxRows)`
- `cacheable(bool $cacheable = true)`
- `cacheTtl(int $ttl)`
- `permissions(array $permissions)`
- `meta(string $key, $value)`

#### دوال الوصول
- `getName()`, `getLabel()`, `getDescription()`, `getCategory()`, `getExtension()`
- `getColumns()`, `getComputedColumns()`, `getAllColumns()`
- `getRelationships()`, `getAutoJoinRelationships()`, `getManualJoinRelationships()`
- `getExcludedColumns()`
- `getSupportsAggregates()`, `getMaxRows()`, `isCacheable()`, `getCacheTtl()`
- `getPermissions()`, `getMeta()`

#### دوال للتحقق من الأعمدة والعلاقات
- `getVisibleColumns()`: الأعمدة القابلة للعرض (غير مخفية وغير مستبعدة وليست مفاتيح أجنبية).
- `getAllAvailableColumns()`: جميع الأعمدة القابلة للاختيار، بما في ذلك أعمدة علاقات auto-join (مع مسارات).
- `getRelationship(string $name)`: الحصول على علاقة بالاسم.
- `getColumn(string $name)`: الحصول على عمود بالاسم.
- `hasColumn(string $name)`
- `hasRelationship(string $name)`
- `isColumnAllowed(string $name)`: التحقق مما إذا كان العمود مسموحاً به (موجود وغير مخفي وغير مستبعد).

#### تحويل إلى مصفوفة
```php
public function toArray(): array
```

### مثال كامل لبناء جدول
```php
// إنشاء الأعمدة
$columns = [
    Column::make('uuid')->label('المعرف')->hidden(true),
    Column::make('created_at', 'datetime')->label('تاريخ الإنشاء')->sortable(true),
    Column::make('status', 'string')->label('الحالة')->filterable(true),
    Column::make('amount', 'decimal')->label('المبلغ')->aggregatable(true),
];

// إنشاء العلاقات
$customerRel = Relationship::hasAutoJoin('customer', 'customers')
    ->columns([
        Column::make('name')->label('الاسم'),
        Column::make('email')->label('البريد'),
    ]);

$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->columns(['weight', 'dimensions'])
    ->with([
        Relationship::hasAutoJoin('pickup', 'addresses')
            ->columns(['address', 'city'])
            ->label('موقع الاستلام')
    ]);

// إنشاء الجدول
$ordersTable = Table::make('orders')
    ->label('الطلبات')
    ->description('جدول الطلبات الرئيسي')
    ->category('sales')
    ->extension('core')
    ->columns($columns)
    ->relationships([$customerRel, $payloadRel])
    ->supportsAggregates(true)
    ->maxRows(20000)
    ->cacheable(true)
    ->cacheTtl(1800); // 30 دقيقة
```

---

## 4. كلاس `ReportSchemaRegistry` (سجل المخطط)

### الغرض
يمثل سجلاً مركزياً لجميع الجداول المسجلة في النظام. يوفر واجهة لتسجيل الجداول واسترجاع معلومات الأعمدة والعلاقات مع دعم التخزين المؤقت (Cache) لتحسين الأداء.

### الخصائص الأساسية
- **$tables**: مصفوفة من كائنات `Table` مفهرسة بأسماء الجداول.
- **$cacheEnabled**: هل التخزين المؤقت مفعل؟
- **$cacheTtl**: مدة البقاء الافتراضية للذاكرة المؤقتة (ثواني).

### أهم الطرق

#### تسجيل الجداول
```php
public function registerTable(Table $table): void
public function registerTables(array $tables): void
```
مثال:
```php
$registry = new ReportSchemaRegistry();
$registry->registerTable($ordersTable);
$registry->registerTable($usersTable);
```

#### الحصول على جدول
```php
public function getTable(string $name): ?Table
public function isTableRegistered(string $name): bool
public function hasTable(string $name): bool // alias
public function getRegisteredTableNames(): array
```

#### الحصول على الجداول المتاحة للواجهة
```php
public function getAvailableTables(string $extension, ?string $category = null): array
```
يعيد مصفوفة من الجداول مع معلوماتها (الاسم، التسمية، الوصف، التصنيف، الأعمدة، العلاقات). يدعم التخزين المؤقت.

مثال:
```php
$tables = $registry->getAvailableTables('core', 'sales');
```

#### الحصول على أعمدة جدول
```php
public function getTableColumns(string $tableName): array
```
يعيد مصفوفة من الأعمدة القابلة للاختيار (مع توسيع علاقات auto-join). يتم تخزين النتيجة مؤقتاً.

مثال:
```php
$columns = $registry->getTableColumns('orders');
// النتيجة تحتوي على أعمدة مثل: ['uuid', 'created_at', 'customer.name', 'payload.pickup.address', ...]
```

#### الحصول على علاقات جدول
```php
public function getTableRelationships(string $tableName): array
```
يعيد مصفوفة من العلاقات (بما في ذلك المتداخلة).

#### الحصول على أعمدة auto-join (بدون تداخل)
```php
public function getAutoJoinColumns(string $tableName): array
```
يعيد أعمدة من علاقات auto-join بشكل مسطح.

#### التحقق من صلاحية عمود
```php
public function isColumnAllowed(string $tableName, string $columnPath): bool
```
يتحقق مما إذا كان العمود المحدد (مع مسار مثل `payload.pickup.city`) موجوداً ومسموحاً به. يستخدم للتحقق قبل بناء الاستعلام.

مثال:
```php
if ($registry->isColumnAllowed('orders', 'payload.pickup.city')) {
    // العمود صالح
}
```

#### دوال إدارة التخزين المؤقت
```php
public function clearTableCache(string $tableName): void
public function clearAllCache(): void
public function setCacheEnabled(bool $enabled): void
public function setCacheTtl(int $ttl): void
```

### أمثلة عملية على استخدام Registry

#### تسجيل عدة جداول
```php
$registry = new ReportSchemaRegistry();

$ordersTable = Table::make('orders')...;
$usersTable = Table::make('users')...;
$registry->registerTables([$ordersTable, $usersTable]);
```

#### استخدام في Controller لجلب أعمدة جدول
```php
class ReportController extends Controller
{
    public function getTableColumns(Request $request)
    {
        $tableName = $request->input('table');
        $registry = ReportSchemaRegistry::instance(); // أو عبر حقن الاعتماديات
        
        if (!$registry->hasTable($tableName)) {
            return response()->json(['error' => 'Table not found'], 404);
        }
        
        $columns = $registry->getTableColumns($tableName);
        return response()->json($columns);
    }
}
```

#### التحقق من عمود قبل إضافته لاستعلام
```php
function validateColumn($tableName, $columnPath, ReportSchemaRegistry $registry) {
    if (!$registry->isColumnAllowed($tableName, $columnPath)) {
        throw new \Exception("Column {$columnPath} not allowed");
    }
}
```

#### الحصول على جميع الجداول المتاحة لامتداد معين
```php
$coreTables = $registry->getAvailableTables('core');
$fleetTables = $registry->getAvailableTables('fleet');
```

---

## ملخص العلاقات بين الكلاسات

- **Column** هو الوحدة الأساسية لوصف حقل.
- **Relationship** يبني على Column لإضافة أعمدة من جدول آخر، ويمكنه احتواء **Relationship** أخرى (تداخل).
- **Table** يجمع Columns و Relationships لوصف جدول كامل.
- **ReportSchemaRegistry** يدير مجموعة من Tables ويوفر واجهة استعلام مع تخزين مؤقت.

هذا التصميم يسمح ببناء مخطط تقارير ديناميكي وقابل للتوسع، يمكن استخدامه في أنظمة نانوسوفت المتقدمة حيث توجد علاقات متعددة المستويات (مثل `orders.payload.pickup.address`).

### كيف تعمل الكلاسات معًا (سيناريو نموذجي)

1. **التعريف (Definition):** يقوم المطور (أو نظام الإضافة) بإنشاء كائنات `Column` و `Relationship` و `Table` لوصف هيكل البيانات.
2. **التسجيل (Registration):** تُسجل كائنات `Table` في `ReportSchemaRegistry` مرة واحدة (عادة في Service Provider).
3. **الاستعلام (Querying):** عندما تحتاج الواجهة الأمامية إلى عرض أعمدة متاحة لتقرير جديد، يتم استدعاء `$registry->getTableColumns('orders')`، والذي يقوم بدوره بجمع الأعمدة العادية وأعمدة العلاقات (`getAllAvailableColumns`) وإعادتها كقائمة مسطحة مع مساراتها (مثل `customer.name`).
4. **التحقق (Validation):** قبل تنفيذ استعلام تقرير مخصص، يتم استخدام `$registry->isColumnAllowed('orders', 'payload.pickup.city')` للتأكد من أن كل عمود في تكوين التقرير صالح ومسموح به.
5. **بناء الاستعلام (Query Building):** يستخدم `ReportQueryConverter` (كلاس آخر في نفس الحزمة) نفس `$registry` لتحويل مسارات الأعمدة إلى استعلام SQL مع الجوينات المناسبة، بالاعتماد على معلومات العلاقة المخزنة.

### فوائد هذا التصميم

- **الفصل بين التعريف والاستخدام:** يتم تعريف هيكل البيانات بشكل صريح في كائنات منفصلة، مما يجعل النظام قابلاً للصيانة والتوسع.
- **إعادة الاستخدام:** يمكن إعادة استخدام تعريفات العلاقات (`Relationship`) عبر جداول متعددة.
- **السلامة والأمان:** يوفر `ReportSchemaRegistry` نقطة تحقق مركزية للتأكد من أن الأعمدة التي يطلبها المستخدم موجودة ومسموح بها، مما يمنع محاولات الوصول غير المصرح بها أو استعلامات SQL غير الصالحة.
- **الأداء:** استخدام التخزين المؤقت (`cache`) لمخرجات `getTableColumns` و `getAvailableTables` يقلل من استعلامات قاعدة البيانات المتكررة ويسرّع استجابات API.
- **المرونة:** دعم العلاقات المتداخلة والمسارات النقطية (مثل `payload.pickup.address`) يسمح ببناء تقارير معقدة تغطي أعماقًا متعددة من البيانات.

### أفضل الممارسات عند استخدام هذه الكلاسات

1. **توحيد التسميات:** استخدم أسماء متسقة للعلاقات والأعمدة لتجنب الارتباك (مثل استخدام `_uuid` كلاحقة للمفاتيح الأجنبية).
2. **تحديد `autoJoin` بحذر:** اجعل العلاقات `autoJoin` فقط عندما تكون الأعمدة المرتبطة بها ضرورية بشكل متكرر في التقارير، لتجنب جلب بيانات غير ضرورية.
3. **الاستفادة من `excludeColumns`:** استبعد الأعمدة الحساسة أو التقنية (مثل `password`، `remember_token`) من العرض العام.
4. **تسجيل الجداول في الوقت المناسب:** قم بتسجيل الجداول في Service Provider للتطبيق أو الإضافات لضمان توفرها قبل أي طلب.
5. **استخدام `copyWith` في العلاقات المتداخلة:** عند إنشاء علاقات متداخلة عميقة، قد تحتاج إلى تعديل نسخة من عمود (مثل تغيير التسمية) باستخدام `copyWith` للحفاظ على السياق.

## خاتمة

تشكل هذه الكلاسات الأربعة (`Column`, `Relationship`, `Table`, `ReportSchemaRegistry`) العمود الفقري لطبقة تعريف التقارير في حزمة **Nano2.QueryBuilder** من **شركة نانوسوفت (NanoSoft)**. توفر معًا طريقة منظمة وقوية لوصف أي هيكل قاعدة بيانات معقد، مع إمكانية توسيعها بسهولة بواسطة إضافات جديدة. من خلال فصل التعريف عن التنفيذ، وتوفير آليات التحقق والتخزين المؤقت، يضمن هذا التصميم بناء تقارير ديناميكية بأمان وكفاءة عالية، مما يلبي احتياجات المستخدمين المتقدمة دون المساس بأداء النظام أو سلامته.