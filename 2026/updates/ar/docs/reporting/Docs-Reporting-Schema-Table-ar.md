## توثيق كلاس `Table`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `Table` هو أحد الكلاسات الأساسية في طبقة تعريف المخطط (Schema Definition Layer) لنظام التقارير الديناميكي `Nano2.QueryBuilder.Reporting`. يمثل هذا الكلاس **جدولاً في قاعدة البيانات** متاحاً للاستعلام عبر نظام التقارير. يقوم بتجميع كافة المعلومات المتعلقة بالجدول: أعمدةه العادية، أعمدةه المحسوبة، العلاقات مع الجداول الأخرى، بالإضافة إلى خصائص وصفية مثل التصنيف (category)، الامتداد (extension)، والصلاحيات.

يعمل `Table` كحاوية شاملة توفر واجهة موحدة للوصول إلى جميع البيانات القابلة للتقرير من هذا الجدول، سواء كانت مباشرة أو عبر العلاقات.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$name` | `string` | اسم الجدول الفعلي في قاعدة البيانات (مثل `orders`). |
| `$label` | `string` | تسمية عرض للجدول (مثل "الطلبات") تستخدم في واجهة المستخدم. |
| `$description` | `?string` | وصف اختياري للجدول يوضح محتواه أو استخدامه. |
| `$category` | `?string` | تصنيف الجدول (مثل `sales`، `inventory`، `hr`) لتنظيم الجداول في مجموعات. |
| `$extension` | `?string` | اسم الامتداد (extension) الذي ينتمي إليه الجدول (مثل `core`، `fleet`). |
| `$columns` | `array` | مصفوفة من كائنات `Column` تمثل الأعمدة العادية في الجدول. |
| `$computedColumns` | `array` | مصفوفة من كائنات `Column` تمثل الأعمدة المحسوبة (المشتقة من تعبير SQL). |
| `$relationships` | `array` | مصفوفة من كائنات `Relationship` تمثل العلاقات مع الجداول الأخرى. |
| `$excludedColumns` | `array` | قائمة بأسماء الأعمدة المستبعدة من العرض في واجهة المستخدم (تبقى متاحة للاستعلام). |
| `$supportsAggregates` | `bool` | هل يدعم الجدول دوال التجميع (مثل COUNT, SUM, AVG)؟ |
| `$maxRows` | `?int` | الحد الأقصى لعدد الصفوف المسموح بإرجاعها في أي استعلام على هذا الجدول. |
| `$cacheable` | `bool` | هل يمكن تخزين نتائج استعلامات هذا الجدول مؤقتاً؟ |
| `$cacheTtl` | `int` | مدة التخزين المؤقت (بالثواني) إذا كان `$cacheable = true`. |
| `$permissions` | `array` | مصفوفة تحدد صلاحيات الوصول للجدول (يمكن استخدامها لاحقاً). |
| `$meta` | `array` | بيانات إضافية على شكل key-value (مفيدة للتوسعات). |

---

### أهم الطرق (Methods)

#### 1. إنشاء كائن Table

##### `public static function make(string $name): self`
- **الهدف**: إنشاء كائن جديد من `Table` بالاسم المحدد.
- **المعاملات**: `$name` - اسم الجدول في قاعدة البيانات.
- **الإرجاع**: كائن `Table` جديد.
- **مثال**:
  ```php
  $ordersTable = Table::make('orders');
  ```

##### `public function label(string $label): self`
- **الهدف**: تعيين تسمية العرض للجدول.
- **مثال**:
  ```php
  $ordersTable->label('الطلبات');
  ```

##### `public function description(string $description): self`
- **الهدف**: تعيين وصف للجدول.
- **مثال**:
  ```php
  $ordersTable->description('جدول الطلبات الرئيسي');
  ```

##### `public function category(string $category): self`
- **الهدف**: تعيين تصنيف الجدول.
- **مثال**:
  ```php
  $ordersTable->category('sales');
  ```

##### `public function extension(string $extension): self`
- **الهدف**: تعيين الامتداد (extension) الذي يوفره الجدول.
- **مثال**:
  ```php
  $ordersTable->extension('core');
  ```

#### 2. إضافة الأعمدة

##### `public function columns(array $columns): self`
- **الهدف**: إضافة مجموعة من الأعمدة إلى الجدول. يمكن تمرير:
  - كائنات `Column` جاهزة.
  - مصفوفات تحتوي على `name` و `type` (سيتم تحويلها إلى `Column`).
  - أسماء أعمدة نصية (سيتم تحويلها إلى `Column` بنوع `string`).
- **مثال**:
  ```php
  $ordersTable->columns([
      'uuid',
      'created_at',
      ['name' => 'amount', 'type' => 'decimal'],
      Column::make('status', 'string')->label('الحالة'),
  ]);
  ```

##### `public function addColumn(Column $column): self`
- **الهدف**: إضافة عمود واحد إلى الجدول.
- **مثال**:
  ```php
  $ordersTable->addColumn(Column::make('notes')->label('ملاحظات'));
  ```

##### `public function computedColumns(array $columns): self`
- **الهدف**: إضافة مجموعة من الأعمدة المحسوبة (يجب أن تكون كائنات `Column` مع `computed = true`).
- **مثال**:
  ```php
  $ordersTable->computedColumns([
      Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal'),
  ]);
  ```

##### `public function addComputedColumn(Column $column): self`
- **الهدف**: إضافة عمود محسوب واحد.

#### 3. إضافة العلاقات

##### `public function relationships(array $relationships): self`
- **الهدف**: إضافة مجموعة من العلاقات (كائنات `Relationship`).
- **مثال**:
  ```php
  $ordersTable->relationships([
      $customerRel,
      $payloadRel,
  ]);
  ```

##### `public function addRelationship(Relationship $relationship): self`
- **الهدف**: إضافة علاقة واحدة.

#### 4. إدارة الأعمدة المستبعدة

##### `public function excludeColumns(array $columnNames): self`
- **الهدف**: استبعاد أعمدة معينة من الظهور في واجهة المستخدم (مع بقائها متاحة للاستعلام).
- **مثال**:
  ```php
  $ordersTable->excludeColumns(['deleted_at', 'internal_notes']);
  ```

#### 5. تعيين الخصائص العامة

##### `public function supportsAggregates(bool $supports = true): self`
- **الهدف**: تحديد ما إذا كان الجدول يدعم دوال التجميع.

##### `public function maxRows(int $maxRows): self`
- **الهدف**: تعيين الحد الأقصى لعدد الصفوف المسموح بها.

##### `public function cacheable(bool $cacheable = true): self`
- **الهدف**: تفعيل/تعطيل التخزين المؤقت.

##### `public function cacheTtl(int $ttl): self`
- **الهدف**: تعيين مدة التخزين المؤقت.

##### `public function permissions(array $permissions): self`
- **الهدف**: تعيين صلاحيات الوصول للجدول.

##### `public function meta(string $key, $value): self`
- **الهدف**: إضافة بيانات وصفية.

#### 6. دوال الوصول (Getters)

##### `public function getName(): string`
##### `public function getLabel(): string`
##### `public function getDescription(): ?string`
##### `public function getCategory(): ?string`
##### `public function getExtension(): ?string`
##### `public function getColumns(): array`
- **الإرجاع**: مصفوفة الأعمدة العادية.

##### `public function getComputedColumns(): array`
- **الإرجاع**: مصفوفة الأعمدة المحسوبة.

##### `public function getAllColumns(): array`
- **الإرجاع**: دمج الأعمدة العادية والمحسوبة في مصفوفة واحدة.

##### `public function getRelationships(): array`
##### `public function getExcludedColumns(): array`
##### `public function getSupportsAggregates(): bool`
##### `public function getMaxRows(): ?int`
##### `public function isCacheable(): bool`
##### `public function getCacheTtl(): int`
##### `public function getPermissions(): array`
##### `public function getMeta(?string $key = null)`

#### 7. دوال التحقق والاستعلام المتقدم

##### `public function getVisibleColumns(): array`
- **الهدف**: إرجاع الأعمدة القابلة للعرض في واجهة المستخدم. تستبعد:
  - الأعمدة المخفية (`hidden = true`).
  - الأعمدة الموجودة في `$excludedColumns`.
  - الأعمدة التي هي مفاتيح أجنبية (تنتهي بـ `_uuid` أو `_id`).

##### `public function getAutoJoinRelationships(): array`
- **الهدف**: إرجاع العلاقات التي تحمل خاصية `autoJoin = true` فقط.

##### `public function getManualJoinRelationships(): array`
- **الهدف**: إرجاع العلاقات التي لا تحمل خاصية `autoJoin`.

##### `public function getRelationship(string $name): ?Relationship`
- **الهدف**: البحث عن علاقة بالاسم المحدد.

##### `public function getColumn(string $name): ?Column`
- **الهدف**: البحث عن عمود بالاسم (يبحث في الأعمدة العادية والمحسوبة).

##### `public function hasColumn(string $name): bool`
- **الهدف**: التحقق من وجود عمود بالاسم.

##### `public function hasRelationship(string $name): bool`
- **الهدف**: التحقق من وجود علاقة بالاسم.

##### `public function isColumnAllowed(string $name): bool`
- **الهدف**: التحقق مما إذا كان العمود مسموحاً به (أي موجود وغير مخفي وغير مستبعد).
- **مثال**:
  ```php
  if ($table->isColumnAllowed('amount')) {
      // يمكن استخدام العمود
  }
  ```

##### `public function getAllAvailableColumns(): array`
- **الهدف**: إرجاع جميع الأعمدة القابلة للاختيار، بما في ذلك أعمدة علاقات auto-join. يتم ذلك عن طريق:
  - الحصول على الأعمدة المرئية (`getVisibleColumns`).
  - لكل علاقة auto-join، يتم جلب جميع أعمدة العلاقة (بما في ذلك المتداخلة) باستخدام `getAllAvailableColumns` من `Relationship`، وإضافة `auto_join_path` إلى الـ meta.
- هذا هو المصدر الرئيسي للأعمدة التي ستظهر للمستخدم في واجهة بناء التقرير.

##### `public function toArray(): array`
- **الهدف**: تحويل الجدول إلى مصفوفة قابلة للتسلسل (مثلاً لإرسالها عبر API). تحتوي على جميع المعلومات الأساسية.

##### `protected function isForeignKeyColumn(string $name): bool`
- **الهدف**: التحقق مما إذا كان اسم العمود يشير إلى مفتاح أجنبي (ينتهي بـ `_uuid` أو `_id`).

##### `protected function generateLabel(string $name): string`
- **الهدف**: توليد تسمية افتراضية من اسم الجدول (تحويل underscores والشرطات إلى مسافات، ثم تحويل الحرف الأول إلى كبير).

---

### أمثلة عملية شاملة

#### مثال 1: إنشاء جدول كامل (مع أعمدة وعلاقات)
لنفترض أننا نبني جدول `orders` مع أعمدة وعلاقات `customer` و `payload` التي تحتوي على `pickup` و `dropoff`.

```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Relationship;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;

// 1. إنشاء الأعمدة العادية
$columns = [
    Column::make('uuid')->label('المعرف')->hidden(true),
    Column::make('created_at', 'datetime')->label('تاريخ الإنشاء')->sortable(true),
    Column::make('amount', 'decimal')->label('المبلغ')->aggregatable(true),
    Column::make('status', 'string')->label('الحالة')->filterable(true),
];

// 2. إنشاء الأعمدة المحسوبة
$computedColumns = [
    Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
        ->label('الإجمالي شامل الضريبة'),
];

// 3. إنشاء علاقة customer (auto-join)
$customerRel = Relationship::hasAutoJoin('customer', 'customers')
    ->label('العميل')
    ->localKey('customer_uuid')
    ->foreignKey('uuid')
    ->columns([
        Column::make('name')->label('الاسم'),
        Column::make('email')->label('البريد الإلكتروني'),
    ]);

// 4. إنشاء علاقة payload (auto-join) مع علاقات متداخلة
$addressColumns = [
    Column::make('address')->label('العنوان'),
    Column::make('city')->label('المدينة'),
    Column::make('country')->label('الدولة'),
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
        Column::make('weight', 'decimal')->label('الوزن'),
        Column::make('dimensions')->label('الأبعاد'),
    ])
    ->with([$pickupRel, $dropoffRel]);

// 5. إنشاء الجدول
$ordersTable = Table::make('orders')
    ->label('الطلبات')
    ->description('جدول الطلبات الرئيسي')
    ->category('sales')
    ->extension('core')
    ->columns($columns)
    ->computedColumns($computedColumns)
    ->relationships([$customerRel, $payloadRel])
    ->excludeColumns(['deleted_at', 'internal_note'])
    ->supportsAggregates(true)
    ->maxRows(50000)
    ->cacheable(true)
    ->cacheTtl(3600); // ساعة واحدة
```

#### مثال 2: استخدام دوال الوصول
```php
// الحصول على جميع الأعمدة القابلة للعرض (بدون مفاتيح أجنبية)
$visibleColumns = $ordersTable->getVisibleColumns();
foreach ($visibleColumns as $column) {
    echo $column->getName() . ' - ' . $column->getLabel() . PHP_EOL;
}
// الناتج: uuid - المعرف, created_at - تاريخ الإنشاء, amount - المبلغ, status - الحالة

// الحصول على جميع الأعمدة المتاحة (بما في ذلك أعمدة العلاقات)
$allAvailable = $ordersTable->getAllAvailableColumns();
foreach ($allAvailable as $column) {
    echo $column->getName() . ' (' . $column->getMeta('auto_join_path') . ')' . PHP_EOL;
}
// الناتج: uuid, created_at, amount, status, customer.name (customer), customer.email (customer), payload.weight (payload), payload.dimensions (payload), payload.pickup.address (payload.pickup), payload.pickup.city (payload.pickup), ...

// التحقق من وجود عمود معين
if ($ordersTable->hasColumn('amount')) {
    echo "العمود amount موجود";
}

// الحصول على علاقة معينة
$customer = $ordersTable->getRelationship('customer');
if ($customer) {
    echo $customer->getLabel(); // "العميل"
}
```

#### مثال 3: التحقق من صلاحية عمود
```php
// التحقق مما إذا كان عمود معين مسموحاً به للاستخدام في التقارير
if ($ordersTable->isColumnAllowed('amount')) {
    // يمكن إضافته إلى التقرير
}

if (!$ordersTable->isColumnAllowed('deleted_at')) {
    // هذا العمود مستبعد
}

if ($ordersTable->isColumnAllowed('payload.pickup.city')) {
    // بالرغم أن هذا العمود ليس مباشراً، لكنه يظهر في getAllAvailableColumns
    // (التحقق هنا يعتمد على getColumn الذي لا يتعامل مع المسارات؛ يحتاج Registry)
    // لذا يُفضل استخدام isColumnAllowed من Registry للمسارات.
}
```

#### مثال 4: التفاعل مع ReportSchemaRegistry
بعد إنشاء الجدول، يجب تسجيله في `ReportSchemaRegistry`:

```php
$registry = app(ReportSchemaRegistry::class);
$registry->registerTable($ordersTable);

// لاحقاً، يمكن استرجاع الجدول
$retrieved = $registry->getTable('orders');
if ($retrieved) {
    $columns = $retrieved->getAllAvailableColumns();
}
```

#### مثال 5: استخدام toArray لعرض معلومات الجدول
```php
$array = $ordersTable->toArray();
// يمكن إرجاع هذا JSON للواجهة الأمامية
return response()->json($array);
```

---

### التفاعل مع الكلاسات الأخرى

- **`Column`**: يستخدم `Table` كائنات `Column` لتمثيل الأعمدة. يوفر `Table` دوال لإضافة واسترجاع الأعمدة.
- **`Relationship`**: يستخدم `Table` كائنات `Relationship` لتمثيل العلاقات. يوفر دوال لتصفية العلاقات حسب `autoJoin`.
- **`ReportSchemaRegistry`**: يقوم `Table` بالتسجيل في `Registry`، ويسأل `Registry` عن الجداول الأخرى عند الحاجة (مثل التحقق من وجود أعمدة في العلاقات).

---

### أفضل الممارسات

1. **توحيد التسميات**: استخدم أسماء متسقة للجداول والعلاقات (مثل استخدام `snake_case`).
2. **تحديد `autoJoin` بحكمة**: اجعل العلاقات `autoJoin` فقط عندما تتوقع أن المستخدمين سيحتاجون بشكل متكرر إلى أعمدة من تلك العلاقة. هذا يمنع JOINs غير ضرورية.
3. **استخدام `excludeColumns` لإخفاء الأعمدة الحساسة**: مثل `password`، `api_token`، `remember_token`.
4. **تحديد `maxRows`**: ضع حداً معقولاً لحماية أداء النظام.
5. **الاستفادة من التخزين المؤقت**: اضبط `cacheable` و `cacheTtl` حسب تحديثات الجدول.

---

### الخلاصة
كلاس `Table` هو حجر الزاوية في نظام تعريف المخطط للتقارير. يوفر طريقة منظمة وقوية لوصف أي جدول في قاعدة البيانات، مع كافة تفاصيله وعلاقاته. من خلال واجهته الواضحة، يمكن للمطورين بناء طبقة تعريفية متكاملة تغذي نظام التقارير الديناميكي بكل البيانات اللازمة، مما يتيح للمستخدمين النهائيين إنشاء تقارير معقدة بسهولة وأمان.