## توثيق كلاس `Relationship`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `Relationship` هو أحد الكلاسات الأساسية في طبقة تعريف المخطط (Schema Definition Layer) لنظام التقارير الديناميكي `Nano2.QueryBuilder.Reporting`. يمثل هذا الكلاس **علاقة بين جدولين** في قاعدة البيانات، مثل `belongsTo`، `hasMany`، أو `hasOne`. يوفر معلومات شاملة عن العلاقة: اسم العلاقة، الجدول المرتبط، مفاتيح الربط، نوع الـ JOIN، والأعمدة المتاحة من الجدول المرتبط. كما يدعم **العلاقات المتداخلة (Nested Relationships)** وخاصية **Auto-Join** التي تسمح بإضافة JOIN تلقائياً عند اختيار أعمدة من العلاقة.

يُستخدم `Relationship` بشكل أساسي داخل كلاس `Table` لوصف كيفية ارتباط الجدول الحالي بجداول أخرى، مما يتيح لنظام التقارير بناء استعلامات معقدة تشمل عدة جداول دون الحاجة لكتابة JOINs يدوياً.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$name` | `string` | اسم العلاقة (مثل `customer`، `payload`). يُستخدم كمعرف للعلاقة في المسارات (مثل `customer.name`). |
| `$table` | `string` | اسم الجدول المرتبط في قاعدة البيانات (مثل `customers`، `order_payloads`). |
| `$label` | `string` | تسمية عرض للعلاقة (مثل "العميل"، "بيانات الشحنة") تستخدم في واجهة المستخدم. |
| `$type` | `string` | نوع الـ JOIN (`left`، `right`، `inner`). افتراضي `left`. |
| `$localKey` | `string` | المفتاح المحلي في الجدول الحالي الذي يرتبط بالعلاقة (افتراضياً `{name}_uuid`). |
| `$foreignKey` | `string` | المفتاح الأجنبي في الجدول المرتبط (افتراضياً `uuid`). |
| `$enabled` | `bool` | هل العلاقة مفعلة؟ (افتراضي `true`). يمكن تعطيلها مؤقتاً دون حذفها. |
| `$autoJoin` | `bool` | هل تُضاف هذه العلاقة تلقائياً عند اختيار أعمدة منها؟ (افتراضي `false`). |
| `$description` | `?string` | وصف اختياري للعلاقة. |
| `$columns` | `array` | مصفوفة من كائنات `Column` تمثل الأعمدة المتاحة من الجدول المرتبط. |
| `$nestedRelationships` | `array` | مصفوفة من كائنات `Relationship` تمثل علاقات داخل الجدول المرتبط (مثل `payload.pickup`). |
| `$meta` | `array` | بيانات إضافية على شكل key-value (مفيدة للتوسعات). |

---

### أهم الطرق (Methods)

#### 1. إنشاء كائن Relationship

##### `public static function make(string $name, string $table, string $type = 'left'): self`
- **الهدف**: إنشاء علاقة جديدة بالاسم والجدول المحددين.
- **المعاملات**:
  - `$name`: اسم العلاقة (مثل `customer`).
  - `$table`: اسم الجدول المرتبط.
  - `$type`: نوع الـ JOIN (افتراضي `left`).
- **الإرجاع**: كائن `Relationship` جديد.
- **مثال**:
  ```php
  $customerRel = Relationship::make('customer', 'customers');
  ```

##### أنواع العلاقات المساعدة
توفر هذه الدوال طرقًا مختصرة لإنشاء أنواع شائعة من العلاقات:

- **`public static function belongsTo(string $name, string $table): self`**  
  مرادف لـ `make` مع `type = 'left'` (افتراضي).  
  مثال: `Relationship::belongsTo('user', 'users')`

- **`public static function hasMany(string $name, string $table): self`**  
  مرادف لـ `make` مع `type = 'left'`.  
  مثال: `Relationship::hasMany('posts', 'posts')`

- **`public static function hasOne(string $name, string $table): self`**  
  مرادف لـ `make` مع `type = 'left'`.  
  مثال: `Relationship::hasOne('profile', 'profiles')`

- **`public static function hasAutoJoin(string $name, string $table, string $type = 'left'): self`**  
  إنشاء علاقة مع تفعيل خاصية `autoJoin`.  
  مثال: `Relationship::hasAutoJoin('payload', 'order_payloads')`

#### 2. تعيين الخصائص (Fluent Setters)

##### `public function label(string $label): self`
- تعيين تسمية العرض للعلاقة.
- **مثال**: `$customerRel->label('العميل');`

##### `public function description(string $description): self`
- تعيين وصف العلاقة.

##### `public function localKey(string $localKey): self`
- تعيين المفتاح المحلي (العمود في الجدول الحالي).
- **مثال**: `$customerRel->localKey('customer_uuid');`

##### `public function foreignKey(string $foreignKey): self`
- تعيين المفتاح الأجنبي (العمود في الجدول المرتبط).
- **مثال**: `$customerRel->foreignKey('uuid');`

##### `public function joinType(string $type): self`
- تعيين نوع الـ JOIN (`left`, `right`, `inner`).

##### `public function enabled(bool $enabled = true): self`
- تفعيل أو تعطيل العلاقة.

##### `public function autoJoin(bool $autoJoin = true): self`
- تفعيل أو تعطيل خاصية auto-join.

##### `public function columns(array $columns): self`
- إضافة مجموعة من الأعمدة إلى العلاقة. يمكن تمرير:
  - كائنات `Column` جاهزة.
  - مصفوفات تحتوي على `name` و `type` (سيتم تحويلها إلى `Column`).
  - أسماء أعمدة نصية (سيتم تحويلها إلى `Column` بنوع `string`).
- **مثال**:
  ```php
  $customerRel->columns([
      'name',
      'email',
      Column::make('phone')->label('رقم الهاتف'),
  ]);
  ```

##### `public function addColumn(Column $column): self`
- إضافة عمود واحد إلى العلاقة.

##### `public function with(array $relationships): self`
- إضافة مجموعة من العلاقات المتداخلة (nested relationships). يجب أن تكون كائنات `Relationship`.
- **مثال**:
  ```php
  $payloadRel->with([
      $pickupRel,
      $dropoffRel,
  ]);
  ```

##### `public function addNestedRelationship(Relationship $relationship): self`
- إضافة علاقة متداخلة واحدة.

##### `public function meta(string $key, $value): self`
- إضافة بيانات وصفية.

#### 3. دوال الوصول (Getters)

| الدالة | الإرجاع |
|--------|---------|
| `getName(): string` | اسم العلاقة |
| `getTable(): string` | اسم الجدول المرتبط |
| `getLabel(): string` | التسمية |
| `getType(): string` | نوع الـ JOIN |
| `getLocalKey(): string` | المفتاح المحلي |
| `getForeignKey(): string` | المفتاح الأجنبي |
| `getDescription(): ?string` | الوصف |
| `isEnabled(): bool` | هل العلاقة مفعلة؟ |
| `isAutoJoin(): bool` | هل هي auto-join؟ |
| `getColumns(): array` | الأعمدة المباشرة (بدون المتداخلة) |
| `getNestedRelationships(): array` | العلاقات المتداخلة |
| `getMeta(?string $key = null)` | البيانات الوصفية |

#### 4. دوال متقدمة

##### `public function getAllAvailableColumns(): array`
- **الهدف**: إرجاع جميع الأعمدة المتاحة من هذه العلاقة، بما في ذلك أعمدة العلاقات المتداخلة. يتم ذلك عن طريق:
  - البدء بالأعمدة المباشرة (`$this->columns`).
  - لكل علاقة متداخلة، يتم جلب أعمدةها باستخدام `getAllAvailableColumns()` الخاص بها، ثم تعديل كل عمود باستخدام `copyWith()` لإضافة بادئة المسار (مثل `pickup.`) إلى اسم العمود وتعديل التسمية لتعكس السياق.
- **الإرجاع**: مصفوفة من كائنات `Column` (نسخ معدلة).
- **مثال**:
  ```php
  $allColumns = $payloadRel->getAllAvailableColumns();
  // يحتوي على أعمدة مثل: weight, dimensions, pickup.address, pickup.city, dropoff.address, ...
  ```

##### `public function getNestedRelationship(string $name): ?Relationship`
- **الهدف**: البحث عن علاقة متداخلة بالاسم المحدد.
- **مثال**:
  ```php
  $pickup = $payloadRel->getNestedRelationship('pickup');
  ```

##### `public function getAutoJoinRelationships(): array`
- **الهدف**: إرجاع العلاقات المتداخلة التي تحمل خاصية `autoJoin = true`.

##### `public function getManualJoinRelationships(): array`
- **الهدف**: إرجاع العلاقات المتداخلة التي لا تحمل خاصية `autoJoin`.

##### `public function hasNestedRelationships(): bool`
- **الهدف**: التحقق من وجود علاقات متداخلة.

##### `public function toArray(): array`
- **الهدف**: تحويل العلاقة إلى مصفوفة قابلة للتسلسل (لإرسالها عبر API). تحتوي على جميع الخصائص الأساسية.

#### 5. دوال داخلية محمية

##### `protected function setAutoJoin(bool $autoJoin): self`
- **الهدف**: تعيين خاصية auto-join (تُستخدم داخلياً).

##### `protected function generateLabel(string $name): string`
- **الهدف**: توليد تسمية افتراضية من اسم العلاقة (تحويل underscores والشرطات إلى مسافات، ثم تحويل الحرف الأول إلى كبير).

##### `protected function generateLocalKey(string $name): string`
- **الهدف**: توليد مفتاح محلي افتراضي بالصيغة `{name}_uuid` (لأن نظام Fleetbase يستخدم `_uuid` كلاحقة للمفاتيح الأجنبية).

---

### أمثلة عملية شاملة

#### مثال 1: إنشاء علاقة بسيطة (belongsTo)
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Relationship;
use Nano2\QueryBuilder\Classes\Reporting\Schema\Column;

$customerRel = Relationship::belongsTo('customer', 'customers')
    ->label('العميل')
    ->localKey('customer_uuid')
    ->foreignKey('uuid')
    ->columns([
        Column::make('name')->label('الاسم'),
        Column::make('email')->label('البريد الإلكتروني'),
        Column::make('phone')->label('رقم الهاتف'),
    ]);
```

#### مثال 2: إنشاء علاقة auto-join مع أعمدة
```php
$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->label('بيانات الشحنة')
    ->columns([
        Column::make('weight', 'decimal')->label('الوزن'),
        Column::make('dimensions')->label('الأبعاد'),
        Column::make('notes')->label('ملاحظات'),
    ]);
```

#### مثال 3: إنشاء علاقات متداخلة (Nested Relationships)
لنفترض أن جدول `order_payloads` له علاقات متعددة مع جدول `addresses` (pickup و dropoff):

```php
// الأعمدة المشتركة للعناوين
$addressColumns = [
    Column::make('address')->label('العنوان'),
    Column::make('city')->label('المدينة'),
    Column::make('country')->label('الدولة'),
    Column::make('postal_code')->label('الرمز البريدي'),
];

// علاقة pickup
$pickupRel = Relationship::hasAutoJoin('pickup', 'addresses')
    ->label('موقع الاستلام')
    ->localKey('payload_uuid')  // المفتاح المحلي في جدول addresses يشير إلى payload
    ->foreignKey('uuid')
    ->columns($addressColumns)
    ->autoJoin(true);

// علاقة dropoff
$dropoffRel = Relationship::hasAutoJoin('dropoff', 'addresses')
    ->label('موقع التسليم')
    ->localKey('payload_uuid')
    ->foreignKey('uuid')
    ->columns($addressColumns)
    ->autoJoin(true);

// العلاقة الرئيسية مع payload
$payloadRel = Relationship::hasAutoJoin('payload', 'order_payloads')
    ->label('بيانات الشحنة')
    ->columns([
        Column::make('weight', 'decimal')->label('الوزن'),
        Column::make('dimensions')->label('الأبعاد'),
    ])
    ->with([$pickupRel, $dropoffRel]); // إضافة العلاقات المتداخلة
```

#### مثال 4: استخدام getAllAvailableColumns
```php
// بعد تعريف العلاقات كما في المثال السابق
$allColumns = $payloadRel->getAllAvailableColumns();

foreach ($allColumns as $column) {
    echo $column->getName() . ' - ' . $column->getLabel() . PHP_EOL;
}

// الناتج:
// weight - الوزن
// dimensions - الأبعاد
// pickup.address - موقع الاستلام - العنوان
// pickup.city - موقع الاستلام - المدينة
// pickup.country - موقع الاستلام - الدولة
// pickup.postal_code - موقع الاستلام - الرمز البريدي
// dropoff.address - موقع التسليم - العنوان
// dropoff.city - موقع التسليم - المدينة
// dropoff.country - موقع التسليم - الدولة
// dropoff.postal_code - موقع التسليم - الرمز البريدي
```

لاحظ كيف تم تعديل أسماء الأعمدة بإضافة بادئة المسار (`pickup.`، `dropoff.`) وتعديل التسميات لتعكس السياق.

#### مثال 5: استخدام العلاقات في بناء جدول
```php
use Nano2\QueryBuilder\Classes\Reporting\Schema\Table;

$ordersTable = Table::make('orders')
    ->label('الطلبات')
    ->columns([
        Column::make('uuid')->hidden(true),
        Column::make('created_at', 'datetime')->label('تاريخ الإنشاء'),
        Column::make('amount', 'decimal')->label('المبلغ'),
    ])
    ->relationships([
        $customerRel,  // علاقة customer
        $payloadRel,   // علاقة payload (التي تحتوي على pickup و dropoff داخلياً)
    ]);
```

الآن، عند استدعاء `$ordersTable->getAllAvailableColumns()`، ستظهر أعمدة مثل `customer.name`، `payload.weight`، `payload.pickup.city`، إلخ.

#### مثال 6: البحث عن علاقة متداخلة
```php
$pickup = $payloadRel->getNestedRelationship('pickup');
if ($pickup) {
    echo $pickup->getLabel(); // "موقع الاستلام"
    $pickupColumns = $pickup->getColumns(); // أعمدة pickup فقط
}
```

#### مثال 7: التحقق من خصائص العلاقة
```php
if ($payloadRel->isAutoJoin()) {
    echo "هذه العلاقة ستُضاف تلقائياً عند الحاجة";
}

if ($payloadRel->hasNestedRelationships()) {
    echo "تحتوي على علاقات متداخلة: " . count($payloadRel->getNestedRelationships());
}
```

#### مثال 8: تحويل العلاقة إلى مصفوفة
```php
$array = $payloadRel->toArray();
// يمكن إرسالها للواجهة الأمامية لعرض معلومات العلاقة
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `Table`**: يُستخدم `Relationship` داخل `Table` عبر دوال `relationships()` و `addRelationship()`. يوفر `Table` دوال مثل `getRelationships()`، `getAutoJoinRelationships()`، `getRelationship(name)` التي تعتمد على `Relationship`.
- **مع `Column`**: يحتوي `Relationship` على مصفوفة من كائنات `Column` تمثل أعمدة الجدول المرتبط. كما تستخدم `getAllAvailableColumns()` دالة `copyWith()` من `Column` لتعديل نسخ الأعمدة.
- **مع `ReportQueryConverter`**: عند بناء الاستعلام، يستخدم المحول معلومات العلاقة (المفاتيح، نوع الـ JOIN، الجدول) لإنشاء JOINs. كما يستخدم `getAllAvailableColumns()` (عبر `Table`) لتحديد الأعمدة المتاحة.

---

### أفضل الممارسات

1. **استخدم أسماء علاقات واضحة وموحدة**: مثل `customer`، `payload`، `pickup`، `dropoff`. هذا يسهل فهم المسارات (مثل `customer.name`).
2. **فعّل `autoJoin` بحكمة**: اجعل العلاقات `autoJoin` فقط عندما تتوقع أن المستخدمين سيحتاجون بشكل متكرر إلى أعمدة من تلك العلاقة. الإفراط في auto-join قد يؤدي إلى JOINs غير ضرورية.
3. **حدد المفاتيح بشكل صريح**: لا تعتمد على القيم الافتراضية (`{name}_uuid`) إذا كانت المفاتيح مختلفة. استخدم `localKey()` و `foreignKey()` لتحديدها بدقة.
4. **نظم العلاقات المتداخلة بشكل هرمي**: عند وجود عدة مستويات من العلاقات (مثل `payload.pickup`)، قم ببنائها بشكل متسلسل باستخدام `with()`.
5. **استخدم `copyWith` في الأعمدة المتداخلة بحذر**: يتم ذلك تلقائياً في `getAllAvailableColumns()`، لذا لا حاجة لاستخدامه يدوياً إلا في حالات خاصة.
6. **تجنب التدوير (Cyclic Relationships)**: لا تُنشئ علاقات متداخلة تؤدي إلى دورة (مثل A → B → A) لأنها قد تسبب مشاكل في بناء الاستعلام.
7. **استخدم `enabled(false)` لتعطيل العلاقات مؤقتاً**: بدلاً من حذفها، يمكن تعطيلها إذا لزم الأمر.

---

### الخلاصة
كلاس `Relationship` هو العمود الفقري لتمثيل العلاقات بين الجداول في نظام التقارير الديناميكي. بفضل دعمه للعلاقات المتداخلة وخاصية auto-join، يمكن بناء مخططات بيانات معقدة تعكس بدقة هيكل قاعدة البيانات الحقيقي. من خلال توفير أعمدة منسقة مع مسارات واضحة، يسهل على المستخدمين النهائيين اختيار البيانات من أي مستوى من العلاقات دون الحاجة لفهم تفاصيل JOINs. يتكامل `Relationship` بشكل وثيق مع `Table` و `Column` ليشكل معاً طبقة تعريفية قوية ومرنة.