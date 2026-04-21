## توثيق كلاس `Column`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `Column` هو الوحدة الأساسية في طبقة تعريف المخطط (Schema Definition Layer) لنظام التقارير الديناميكي `Nano2.QueryBuilder.Reporting`. يمثل هذا الكلاس **عموداً في قاعدة البيانات** أو **عموداً محسوباً (Computed Column)** يُشتق من تعبير SQL. يحتوي على جميع الخصائص اللازمة لوصف العمود وإمكانياته (بحث، ترتيب، تصفية، تجميع) بالإضافة إلى دالة تحويل مخصصة (Transformer) لتنسيق القيم قبل عرضها.

يُستخدم `Column` داخل كلاس `Table` لوصف أعمدة الجدول، وداخل كلاس `Relationship` لوصف أعمدة الجدول المرتبط. كما يمكن استخدامه لإنشاء أعمدة محسوبة معقدة باستخدام دوال SQL مسموحة.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$name` | `string` | اسم العمود الفعلي في قاعدة البيانات (مثل `created_at`). |
| `$label` | `string` | تسمية عرض للمستخدم (مثل "تاريخ الإنشاء")، تُستخدم في واجهة المستخدم. |
| `$type` | `string` | نوع البيانات (string, integer, decimal, date, datetime, boolean, ...). |
| `$description` | `?string` | وصف اختياري للعمود يوضح محتواه. |
| `$format` | `?string` | تنسيق عرض اختياري (مثل `Y-m-d` للتاريخ، `#,##0.00` للرقم). |
| `$nullable` | `bool` | هل يمكن أن تكون القيمة فارغة (NULL)؟ (افتراضي `true`). |
| `$searchable` | `bool` | هل يمكن البحث في هذا العمود؟ (افتراضي `true`). |
| `$sortable` | `bool` | هل يمكن ترتيب النتائج بناءً على هذا العمود؟ (افتراضي `true`). |
| `$filterable` | `bool` | هل يمكن استخدام هذا العمود في شروط التصفية؟ (افتراضي `true`). |
| `$aggregatable` | `bool` | هل يمكن استخدام هذا العمود في دوال التجميع (SUM, AVG, إلخ)؟ (يُحدد تلقائياً للأرقام). |
| `$hidden` | `bool` | إخفاء العمود من الواجهة (يبقى متاحاً للاستعلام). (افتراضي `false`). |
| `$computed` | `bool` | هل هو عمود محسوب (ناتج عن تعبير SQL)؟ (افتراضي `false`). |
| `$computation` | `?string` | تعبير SQL المستخدم لحساب العمود إذا كان `computed = true`. |
| `$transformer` | `?\Closure` | دالة PHP لتحويل القيمة قبل العرض (تطبق على كل قيمة). |
| `$meta` | `array` | بيانات إضافية على شكل key-value (مفيدة للتوسعات). |

---

### أهم الطرق (Methods)

#### 1. إنشاء كائن Column

##### `public static function make(string $name, string $type = 'string'): self`
- **الهدف**: إنشاء عمود عادي جديد.
- **المعاملات**:
  - `$name`: اسم العمود.
  - `$type`: نوع البيانات (افتراضي `string`).
- **الإرجاع**: كائن `Column` جديد.
- **مثال**:
  ```php
  $column = Column::make('status', 'string');
  ```

##### `public static function computed(string $name, string $computation, string $type = 'string', array $options = []): self`
- **الهدف**: إنشاء عمود محسوب بتعبير SQL.
- **المعاملات**:
  - `$name`: اسم العمود (سيظهر بهذا الاسم في النتائج).
  - `$computation`: تعبير SQL (مثل `"ROUND(amount * 1.14, 2)"`).
  - `$type`: نوع البيانات الناتجة.
  - `$options`: مصفوفة خيارات إضافية (مثل `aggregatable`, `sortable`, `searchable`).
- **مثال**:
  ```php
  $totalColumn = Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
      ->label('الإجمالي شامل الضريبة');
  ```

#### 2. دوال مساعدة للأعمدة التجميعية (Aggregate Helpers)
توفر هذه الدوال طرقًا مختصرة لإنشاء أعمدة تمثل دوال تجميع SQL.

##### `public static function count(string $name, string $countField = '*'): self`
- **الهدف**: إنشاء عمود COUNT.
- **المعاملات**:
  - `$name`: اسم العمود.
  - `$countField`: الحقل المراد عده (افتراضي `*`).
- **مثال**:
  ```php
  $orderCount = Column::count('order_count', 'uuid')
      ->label('عدد الطلبات');
  ```

##### `public static function sum(string $name, string $sumField): self`
- **الهدف**: إنشاء عمود SUM.
- **المعاملات**:
  - `$name`: اسم العمود.
  - `$sumField`: الحقل المراد جمع قيمه.
- **مثال**:
  ```php
  $totalAmount = Column::sum('total_amount', 'amount')
      ->label('إجمالي المبالغ');
  ```

##### `public static function avg(string $name, string $avgField): self`
- **الهدف**: إنشاء عمود AVG.
- **مثال**:
  ```php
  $averageAmount = Column::avg('avg_amount', 'amount')
      ->label('متوسط المبلغ');
  ```

##### `public static function max(string $name, string $maxField): self`
- **الهدف**: إنشاء عمود MAX.

##### `public static function min(string $name, string $minField): self`
- **الهدف**: إنشاء عمود MIN.

#### 3. تعيين الخصائص (Fluent Setters)
كل هذه الدوال تُعيد الكائن نفسه (`self`) لتمكين التسلسل (chaining).

##### `public function label(string $label): self`
- تعيين تسمية العرض.
- **مثال**: `$column->label('الحالة');`

##### `public function description(string $description): self`
- تعيين الوصف.

##### `public function format(string $format): self`
- تعيين تنسيق العرض (مثل `'Y-m-d'` للتاريخ، `'#,##0.00'` للأرقام).

##### `public function nullable(bool $nullable = true): self`
- تحديد ما إذا كان العمود يقبل قيم NULL.

##### `public function searchable(bool $searchable = true): self`
- تحديد إمكانية البحث.

##### `public function sortable(bool $sortable = true): self`
- تحديد إمكانية الترتيب.

##### `public function filterable(bool $filterable = true): self`
- تحديد إمكانية التصفية.

##### `public function aggregatable(bool $aggregatable = true): self`
- تحديد إمكانية التجميع (يُستخدم في دوال مثل SUM, AVG).

##### `public function hidden(bool $hidden = true): self`
- إخفاء العمود من واجهة المستخدم.

##### `public function meta(string $key, $value): self`
- إضافة بيانات وصفية واحدة.

##### `public function setMeta(array $meta): self`
- تعيين مجموعة من البيانات الوصفية.

#### 4. دوال التحويل (Transformer)

##### `public function transformer(\Closure|callable $transformer): self`
- **الهدف**: تسجيل دالة لتحويل قيمة العمود قبل إرسالها للواجهة.
- **المعاملات**: دالة PHP (إما Closure أو callable) تستقبل القيمة وتُعيد القيمة المحولة.
- **مثال**:
  ```php
  $column->transformer(function ($value) {
      return $value ? 'نشط' : 'غير نشط';
  });
  ```

##### `public function transformValue($value)`
- **الهدف**: تطبيق الـ transformer على قيمة معينة. إذا لم يكن هناك transformer، تُعاد القيمة كما هي.

#### 5. دوال الوصول (Getters)

| الدالة | الإرجاع |
|--------|---------|
| `getName(): string` | اسم العمود |
| `getLabel(): string` | التسمية |
| `getType(): string` | النوع |
| `getDescription(): ?string` | الوصف |
| `getFormat(): ?string` | التنسيق |
| `isNullable(): bool` | هل يقبل NULL؟ |
| `isSearchable(): bool` | هل قابل للبحث؟ |
| `isSortable(): bool` | هل قابل للترتيب؟ |
| `isFilterable(): bool` | هل قابل للتصفية؟ |
| `isAggregatable(): bool` | هل قابل للتجميع؟ |
| `isHidden(): bool` | هل هو مخفي؟ |
| `isComputed(): bool` | هل هو عمود محسوب؟ |
| `getComputation(): ?string` | تعبير الحساب (إذا كان محسوباً) |
| `getTransformer(): ?\Closure` | دالة التحويل |
| `hasTransformer(): bool` | هل يوجد محول؟ |
| `getMeta(?string $key = null)` | البيانات الوصفية (كلها أو مفتاح معين) |

#### 6. دوال إضافية

##### `public function isForeignKey(): bool`
- **الهدف**: التحقق مما إذا كان العمود مفتاحاً أجنبياً (ينتهي بـ `_uuid` أو `_id`).
- **مثال**:
  ```php
  if ($column->isForeignKey()) {
      // هذا العمود يُستخدم في العلاقات
  }
  ```

##### `public function toArray(): array`
- **الهدف**: تحويل العمود إلى مصفوفة قابلة للتسلسل (لإرسالها عبر API). تحتوي على جميع الخصائص العامة.
- **مثال**:
  ```php
  $array = $column->toArray();
  // يمكن إرجاعها كـ JSON
  ```

##### `public function copyWith(array $overrides): self`
- **الهدف**: إنشاء نسخة معدلة من العمود. يُستخدم بشكل خاص في العلاقات المتداخلة لتعديل اسم أو تسمية العمود مع الحفاظ على الخصائص الأخرى.
- **المعاملات**: `$overrides` مصفوفة تحتوي على المفاتيح المراد تغييرها (`name`, `label`, `type`, `description` ...).
- **مثال**:
  ```php
  $newColumn = $originalColumn->copyWith([
      'name' => 'new_name',
      'label' => 'تسمية جديدة'
  ]);
  ```

##### `protected function generateLabel(string $name): string`
- **الهدف**: توليد تسمية افتراضية من اسم العمود (تحويل underscores والشرطات إلى مسافات، ثم تحويل الحرف الأول إلى كبير).

##### `protected function determineAggregatable(string $type): bool`
- **الهدف**: تحديد القيمة الافتراضية لخاصية `aggregatable` بناءً على نوع البيانات (الأرقام والتواريخ قابلة للتجميع، النصوص لا).

---

### أمثلة عملية شاملة

#### مثال 1: إنشاء أعمدة عادية متنوعة
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;

$columns = [
    // عمود نصي عادي
    Column::make('status', 'string')
        ->label('الحالة')
        ->searchable(true)
        ->sortable(true)
        ->filterable(true),

    // عمود تاريخ مع تنسيق
    Column::make('created_at', 'datetime')
        ->label('تاريخ الإنشاء')
        ->format('Y-m-d H:i')
        ->sortable(true),

    // عمود رقمي مع دعم التجميع
    Column::make('amount', 'decimal')
        ->label('المبلغ')
        ->aggregatable(true)
        ->format('#,##0.00'),

    // عمود مخفي (للاستخدام الداخلي)
    Column::make('internal_code')
        ->label('الكود الداخلي')
        ->hidden(true),

    // عمود مع محول (transformer)
    Column::make('is_active', 'boolean')
        ->label('نشط')
        ->transformer(fn($v) => $v ? 'نعم' : 'لا')
];
```

#### مثال 2: إنشاء أعمدة محسوبة
```php
// عمود محسوب يجمع الاسم الأول والأخير
$fullNameColumn = Column::computed('full_name', "CONCAT(first_name, ' ', last_name)", 'string')
    ->label('الاسم الكامل')
    ->searchable(true); // يمكن البحث في العمود المحسوب (يتطلب دعم قاعدة البيانات)

// عمود محسوب يحسب الضريبة
$taxColumn = Column::computed('tax_amount', "ROUND(amount * 0.14, 2)", 'decimal')
    ->label('الضريبة')
    ->aggregatable(true);

// عمود محسوب باستخدام CASE
$statusTextColumn = Column::computed('status_text', 
    "CASE WHEN status = 'completed' THEN 'مكتمل' WHEN status = 'pending' THEN 'قيد الانتظار' ELSE 'آخر' END", 
    'string'
)->label('نص الحالة');
```

#### مثال 3: استخدام دوال التجميع المساعدة
```php
$orderCount = Column::count('order_count', 'uuid')
    ->label('عدد الطلبات');

$totalRevenue = Column::sum('total_revenue', 'amount')
    ->label('إجمالي الإيرادات');

$averageRating = Column::avg('avg_rating', 'rating')
    ->label('متوسط التقييم');

$maxPrice = Column::max('max_price', 'price')
    ->label('أعلى سعر');

$minPrice = Column::min('min_price', 'price')
    ->label('أقل سعر');
```

#### مثال 4: استخدام copyWith في العلاقات المتداخلة
لنفترض أن لدينا علاقة متداخلة مثل `payload.pickup`، ونريد إنشاء عمود `address` لهذا المستوى مع تسمية مناسبة:

```php
// العمود الأصلي في جدول العناوين
$originalAddressColumn = Column::make('address')->label('العنوان');

// عند دمجه في علاقة pickup، نريد تعديل التسمية لتصبح "عنوان الاستلام"
$pickupAddressColumn = $originalAddressColumn->copyWith([
    'label' => 'عنوان الاستلام'
]);

// أو تعديل الاسم ليشمل المسار (يتم هذا تلقائياً في Relationship::getAllAvailableColumns)
$pickupAddressColumnWithPath = $originalAddressColumn->copyWith([
    'name' => 'payload.pickup.address',
    'label' => 'عنوان الاستلام'
]);
```

#### مثال 5: استخدام transformer لتحويل القيم
```php
// تحويل قيمة منطقية إلى نص عربي
$activeColumn = Column::make('is_active', 'boolean')
    ->label('الحالة')
    ->transformer(function ($value) {
        return $value ? 'نشط' : 'غير نشط';
    });

// تحويل تاريخ إلى تنسيق مختلف
$dateColumn = Column::make('created_at', 'datetime')
    ->label('التاريخ')
    ->transformer(function ($value) {
        return $value ? \Carbon\Carbon::parse($value)->format('d/m/Y') : '';
    });

// تحويل رقم إلى عملة
$amountColumn = Column::make('amount', 'decimal')
    ->label('المبلغ')
    ->transformer(function ($value) {
        return number_format($value, 2) . ' د.ك';
    });
```

#### مثال 6: استخدام الخصائص في بناء جدول
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;

$ordersTable = Table::make('orders')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('created_at', 'datetime')->label('تاريخ الطلب')->sortable(true),
        Column::make('amount', 'decimal')->label('المبلغ')->aggregatable(true),
        Column::make('status', 'string')->label('الحالة')->filterable(true),
    ])
    ->computedColumns([
        Column::computed('total_with_tax', 'ROUND(amount * 1.14, 2)', 'decimal')
            ->label('الإجمالي شامل الضريبة'),
    ]);
```

#### مثال 7: الوصول إلى الخصائص والتحقق
```php
$column = Column::make('email', 'string')
    ->label('البريد الإلكتروني')
    ->searchable(true)
    ->sortable(true)
    ->nullable(false);

echo $column->getName(); // email
echo $column->getLabel(); // البريد الإلكتروني
echo $column->isSearchable() ? 'نعم' : 'لا'; // نعم
echo $column->isNullable() ? 'نعم' : 'لا'; // لا
echo $column->isAggregatable() ? 'نعم' : 'لا'; // لا (لأنه نص)

if ($column->isForeignKey()) {
    // لا، لأنه لا ينتهي بـ _uuid أو _id
}
```

#### مثال 8: تحويل العمود إلى مصفوفة
```php
$column = Column::make('status', 'string')
    ->label('الحالة')
    ->filterable(true)
    ->meta('custom_key', 'custom_value');

$array = $column->toArray();
/*
[
    'name' => 'status',
    'label' => 'الحالة',
    'type' => 'string',
    'description' => null,
    'format' => null,
    'nullable' => true,
    'searchable' => true,
    'sortable' => true,
    'filterable' => true,
    'aggregatable' => false,
    'hidden' => false,
    'computed' => false,
    'computation' => null,
    'transformer' => false,
    'meta' => ['custom_key' => 'custom_value']
]
*/
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `Table`**: يُستخدم `Column` داخل `Table` لتعريف أعمدة الجدول عبر دوال `columns()` و `computedColumns()`. كما يستخدم `Table` دوال مثل `getColumn()` و `getAllAvailableColumns()` التي تعتمد على `Column`.
- **مع `Relationship`**: يُستخدم `Column` داخل `Relationship` لتعريف أعمدة الجدول المرتبط عبر دوال `columns()`. كما تقوم `Relationship::getAllAvailableColumns()` بإرجاع نسخ معدلة من الأعمدة (باستخدام `copyWith`) لتضمين مسار العلاقة في الاسم.
- **مع `ReportQueryConverter`**: عند بناء الاستعلام، يقوم المحول بتحويل اسم العمود إلى مرجع `alias.column` باستخدام معلومات الـ join. كما يستخدم `ComputedColumnValidator` للتحقق من صحة تعبيرات الأعمدة المحسوبة.

---

### أفضل الممارسات

1. **استخدم التسميات الواضحة (`label`)**: اجعل التسميات مفهومة للمستخدم النهائي.
2. **حدد الخصائص بدقة**: لا تجعل كل الأعمدة قابلة للبحث والترتيب إذا لم تكن هناك حاجة؛ هذا يحسن أداء الواجهة.
3. **استخدم `hidden` للأعمدة التقنية**: مثل `uuid`، `created_at` قد تكون مفيدة للاستعلام ولكن لا يحتاج المستخدم رؤيتها.
4. **حدد `aggregatable` للأعمدة الرقمية فقط**: تجنب استخدام التجميع على النصوص.
5. **استخدم `transformer` لتنسيق القيم بدلاً من تغييرها في الواجهة**: هذا يحافظ على فصل الاهتمامات.
6. **عند إنشاء أعمدة محسوبة، تأكد من أن التعبير يستخدم دوال مسموحة ويتوافق مع SQL الخاص بقاعدة البيانات المستخدمة**.
7. **استخدم `copyWith` بحذر**: عادة ما يُستخدم داخلياً في `Relationship`، ولكن يمكن استخدامه إذا كنت بحاجة إلى نسخة معدلة.

---

### الخلاصة
كلاس `Column` هو الوحدة الأساسية لوصف البيانات في نظام التقارير الديناميكي. يوفر طريقة مرنة وقوية لتحديد خصائص الأعمدة، سواء كانت عادية أو محسوبة، مع إمكانية تخصيص كل جانب من جوانب سلوكها (بحث، ترتيب، تصفية، تجميع، تحويل). من خلال استخدامه مع `Table` و `Relationship`، يمكن بناء طبقة تعريفية متكاملة تمكن المستخدمين من إنشاء تقارير معقدة بسهولة وأمان.