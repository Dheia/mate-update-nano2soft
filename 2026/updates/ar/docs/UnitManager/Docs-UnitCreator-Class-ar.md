# توثيق الكلاس `UnitCreator`

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 جدول المحتويات

1. [مقدمة](#مقدمة)
2. [الدوال المساعدة الداخلية](#الدوال-المساعدة-الداخلية)
   - [getAndCheckDepartmentsId](#getandcheckdepartmentsid)
   - [getAndCheckCompanysId](#getandcheckcompanysid)
3. [الدوال الرئيسية](#الدوال-الرئيسية)
   - [createUnit](#createunit)
   - [createUnitFromSymbol](#createunitfromsymbol)
   - [createStandardUnitsForType](#createstandardunitsfortype)
   - [createAllStandardUnits](#createallstandardunits)
   - [attachUnitToProduct](#attachunittoproduct)
   - [createProductUnitPrice](#createproductunitprice)
   - [createProductUnitsStructure](#createproductunitsstructure)
   - [createUnitConversion](#createunitconversion)
   - [importUnitsFromSymbols](#importunitsfromsymbols)
   - [createStandardQuantityUnits](#createstandardquantityunits)
   - [firstOrCreateFromSymbol](#firstorcreatefromsymbol)
4. [هيكل النتيجة (Result Array)](#هيكل-النتيجة-result-array)
5. [أمثلة شاملة](#أمثلة-شاملة)

---

## مقدمة

كلاس `UnitCreator` هو المسؤول عن إنشاء وإدارة الوحدات في قاعدة البيانات، بالإضافة إلى ربط الوحدات بالمنتجات والأسعار. يوفر هذا الكلاس دوالاً آمنة تستخدم المعاملات (transactions) وتتعامل مع الأخطاء بشكل منظم. يعتمد على النماذج (Models) مثل `Unit`, `ProductsUnit`, `ProductsPricesUnit` لإجراء العمليات.

الهدف من هذا الكلاس هو توفير واجهة برمجية موحدة لإنشاء الوحدات (سواء من بيانات كاملة أو من رموز قياسية)، ربطها بالمنتجات، وإنشاء الأسعار المرتبطة بها، مع ضمان سلامة البيانات وعدم التكرار.

---

## الدوال المساعدة الداخلية

### `getAndCheckDepartmentsId`

```php
public static function getAndCheckDepartmentsId($departments_id = null)
```

**الوصف:**  
التحقق من معرف القسم (Department ID) وإرجاع القيمة الصحيحة بناءً على إعدادات النظام (مثل استخدام الشركة الافتراضية، نوع القسم الرئيسي/الفرعي). تُستخدم داخلياً لتوحيد منطق الحصول على القسم.

**المعاملات:**
- `$departments_id` : معرف القسم المُدخل (اختياري).

**الإرجاع:**  
معرف القسم الصحيح (integer).

---

### `getAndCheckCompanysId`

```php
public static function getAndCheckCompanysId($departments_id = null, $companys_id = null)
```

**الوصف:**  
التحقق من معرف الشركة (Company ID) وإرجاع القيمة الصحيحة. إذا تم تمرير `departments_id`، يتم استخراج الشركة المرتبطة به. وإلا يتم الرجوع إلى الإعدادات الافتراضية.

**المعاملات:**
- `$departments_id` : معرف القسم (اختياري).
- `$companys_id` : معرف الشركة المُدخل (اختياري).

**الإرجاع:**  
معرف الشركة الصحيح (integer).

---

## الدوال الرئيسية

### `createUnit`

```php
public static function createUnit(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء وحدة جديدة في قاعدة البيانات. تقوم الدالة بالتحقق من المدخلات، التأكد من عدم وجود وحدة مكررة، ثم إنشاء السجل في جدول `units` مع دعم المعاملات.

**المعاملات:**
- `$options` : مصفوفة تحتوي على بيانات الوحدة. من أهم الخيارات:
  - `name` (string) : اسم الوحدة (مطلوب).
  - `unit_symbol` (string) : رمز الوحدة (مثل 'kg', 'm').
  - `type` (string) : نوع الوحدة (أحد ثوابت `UnitType`). الافتراضي `UnitType::QUANTITY_GENERAL`.
  - `companys_id` (int) : معرف الشركة.
  - `departments_id` (int) : معرف القسم.
  - `main_sub` (string) : `'main'` أو `'sub'` (افتراضي `'main'`).
  - `parent_id` (int) : معرف الوالد (إذا كانت الوحدة فرعية).
  - `conversion_factor` (float) : عامل التحويل (افتراضي 1.0).
  - `value` (float) : قيمة الوحدة (عادة نفس `conversion_factor`).
  - `is_active` (bool) : حالة التفعيل (افتراضي true).
  - `is_default` (bool) : هل هي الوحدة الافتراضية؟ (افتراضي false).
  - `is_force_create` (bool) : إذا كان true، يتجاوز خطأ التكرار (ينشئ وحدة مكررة).
  - `status` (string) : الحالة (افتراضي `'active'`).
  - `date_at` (Carbon) : تاريخ الإنشاء (افتراضي الآن).
  - وغيرها من الحقول الموجودة في نموذج `Unit`.
- `$is_test_create` : إذا كانت `true`، يتم إجراء العملية في معاملة ثم التراجع عنها (لأغراض الاختبار).

**الإرجاع:**  
مصفوفة تحتوي على:
- `status` (bool) : نجاح العملية أو فشلها.
- `message` (string) : رسالة توضيحية.
- `error` (string|null) : نص الخطأ إن وجد.
- `errors` (array|null) : تفاصيل الأخطاء (إن وجدت).
- `model` (object|null) : كائن الوحدة المُنشأة (نموذج `Unit`).
- `data` (array) : بيانات الوحدة (نسخة `toArray()`).
- `input_data` (array) : نسخة من الخيارات المُدخلة.
- `process_data` (array) : بيانات إضافية عن العملية.
- `code` (int) : كود الحالة (200 للنجاح، 400 للخطأ).

**مثال:**
```php
$result = UnitCreator::createUnit([
    'name' => 'كيلوجرام',
    'unit_symbol' => 'kg',
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
    'conversion_factor' => 1,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if ($result['status']) {
    $unit = $result['model'];
    echo "تم إنشاء الوحدة: " . $unit->name;
} else {
    echo "خطأ: " . $result['error'];
}
```

---

### `createUnitFromSymbol`

```php
public static function createUnitFromSymbol(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء وحدة من رمز معروف (مثل 'kg', 'm', 'dozen'). تقوم الدالة بالاستعلام عن معلومات الوحدة باستخدام `UnitNameHelper` وملء البيانات تلقائياً (الاسم، النوع، الوصف المختصر). ثم تستدعي `createUnit` لإنشاء الوحدة.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `symbol` (string) : رمز الوحدة (مطلوب).
  - `make_default` (bool) : هل تجعل الوحدة افتراضية؟ (افتراضي false).
  - `companys_id` (int) : معرف الشركة.
  - `departments_id` (int) : معرف القسم.
  - `additional_data` (array) : بيانات إضافية يمكن دمجها مع بيانات الوحدة (مثل `main_sub`, `parent_id`, ...).
- `$is_test_create` : للاختبار.

**الإرجاع:**  
نفس هيكل `createUnit`.

**مثال:**
```php
$result = UnitCreator::createUnitFromSymbol([
    'symbol' => 'dozen',
    'make_default' => true,
    'companys_id' => 1,
    'departments_id' => 2,
    'additional_data' => [
        'is_base_unit' => true,
    ],
]);

if ($result['status']) {
    echo "تم إنشاء وحدة الدزينة: " . $result['model']->name;
}
```

---

### `createStandardUnitsForType`

```php
public static function createStandardUnitsForType(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء جميع الوحدات القياسية لنوع معين (مثل كل وحدات الطول). تستخدم `UnitConverter::getUnitsByType` للحصول على قائمة الرموز، ثم تنشئ كل وحدة باستخدام `createUnitFromSymbol`. تُعيد مصفوفة تحتوي على الوحدات المنشأة والأخطاء.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `type` (string) : نوع الوحدة (مطلوب، من ثوابت `UnitType`).
  - `additional_data` (array) : بيانات إضافية تُمرر لكل وحدة.
  - `companys_id` (int) : معرف الشركة.
  - `departments_id` (int) : معرف القسم.
- `$is_test_create` : للاختبار.

**الإرجاع:**  
مصفوفة تحتوي على:
- `status`, `message`, `error`, `errors` كالمعتاد.
- `data` (array) : قائمة بالوحدات المنشأة (كائنات `Unit`).
- `models` (array) : نفس `data`.
- `process_data` : يحتوي على `created_count` و `errors` (تفاصيل الأخطاء لكل رمز).

**مثال:**
```php
$result = UnitCreator::createStandardUnitsForType([
    'type' => UnitType::LENGTH,
    'companys_id' => 1,
    'departments_id' => 2,
]);

echo "تم إنشاء " . $result['process_data']['created_count'] . " وحدة.";
if (!empty($result['process_data']['errors'])) {
    print_r($result['process_data']['errors']);
}
```

---

### `createAllStandardUnits`

```php
public static function createAllStandardUnits(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء جميع الوحدات القياسية لجميع الأنواع الرئيسية. تستدعي `createStandardUnitsForType` لكل نوع في `UnitType::getMainTypes()`.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `additional_data` (array) : بيانات إضافية.
  - `companys_id` (int) : معرف الشركة.
  - `departments_id` (int) : معرف القسم.
- `$is_test_create` : للاختبار.

**الإرجاع:**  
مصفوفة تحتوي على:
- `data` (array) : مصفوفة ترابطية `[النوع => [وحدات منشأة]]`.
- `process_data` : يحتوي على `created_count` و `errors_count`.

**مثال:**
```php
$result = UnitCreator::createAllStandardUnits([
    'companys_id' => 1,
    'departments_id' => 2,
]);

echo "تم إنشاء " . $result['process_data']['created_count'] . " وحدة إجمالاً.";
```

---

### `attachUnitToProduct`

```php
public static function attachUnitToProduct(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
ربط وحدة بمنتج (إنشاء سجل في جدول `products_units`). تتحقق من عدم التكرار، ومن عدم وجود وحدة رئيسية مكررة إذا طُلب ذلك.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `products_id` (int) : معرف المنتج (مطلوب).
  - `units_id` (int) : معرف الوحدة (مطلوب).
  - `is_main` (bool) : هل هذه الوحدة هي الرئيسية للمنتج؟ (افتراضي false).
  - `conversion_factor` (float) : عامل التحويل (افتراضي 1.0).
  - `value` (float) : قيمة الوحدة (افتراضي 1.0).
  - `barcode` (string) : الباركود (اختياري).
  - `is_active` (bool) : حالة التفعيل (افتراضي true).
  - `is_force_create` (bool) : تجاوز خطأ التكرار.
  - `type` (string) : نوع العملية (`'add'` أو `'edit'`).

**الإرجاع:**  
مصفوفة تحتوي على `model` (كائن `ProductsUnit`) و `data`.

**مثال:**
```php
$result = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => 5,
    'is_main' => true,
    'conversion_factor' => 1,
]);

if ($result['status']) {
    echo "تم ربط الوحدة بالمنتج.";
}
```

---

### `createProductUnitPrice`

```php
public static function createProductUnitPrice(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء سعر لوحدة منتج (في جدول `products_prices_units`). إذا لم تكن الوحدة مرتبطة بالمنتج بعد، يمكن ربطها تلقائياً إذا كان `is_allow_add_unit = true`.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `products_id` (int) : معرف المنتج (مطلوب).
  - `units_id` (int) : معرف الوحدة (مطلوب).
  - `prices_id` (int) : معرف قائمة الأسعار (مطلوب).
  - `currencys_id` (int) : معرف العملة (إذا لم يُعطَ، يحاول استخراجها من المنتج أو العملة الافتراضية).
  - `value` (float) : سعر البيع.
  - `old_price` (float) : السعر القديم.
  - `value_purchas` (float) : سعر الشراء.
  - `costed_amount` (float) : التكلفة.
  - `manage_stock` (bool) : إدارة المخزون.
  - `shop_stock` (float) : الكمية المتاحة.
  - `is_show_old_price` (bool) : إظهار السعر القديم.
  - `is_parleying` (bool) : قابل للتفاوض.
  - `is_default` (bool) : سعر افتراضي.
  - `is_active` (bool) : نشط.
  - `process_type` (string) : `'add'`, `'edit'`, `'add_or_edit'`.
  - وغيرها.

**الإرجاع:**  
مصفوفة تحتوي على `model` (كائن `ProductsPricesUnit`).

**مثال:**
```php
$result = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => 5,
    'prices_id' => 1,
    'value' => 100.50,
    'currencys_id' => 1,
    'is_default' => true,
]);

if ($result['status']) {
    echo "تم إنشاء السعر.";
}
```

---

### `createProductUnitsStructure`

```php
public static function createProductUnitsStructure(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء هيكل كامل من الوحدات والأسعار لمنتج معين. تستقبل مصفوفة من بيانات الوحدات والأسعار وتقوم بإنشاء كل شيء دفعة واحدة.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `products_id` (int) : معرف المنتج.
  - `units_data` (array) : مصفوفة من بيانات الوحدات. كل عنصر يمكن أن يحتوي على `units_id` أو `unit_symbol`، بالإضافة إلى `is_main`, `conversion_factor`, `value`, `barcode`.
  - `prices_data` (array) : مصفوفة من بيانات الأسعار مرتبطة بنفس ترتيب `units_data`. كل عنصر هو مصفوفة من الأسعار لهذه الوحدة.
  - `companys_id`, `departments_id` اختيارية.

**الإرجاع:**  
مصفوفة تحتوي على:
- `data` : `['units' => [...], 'prices' => [...]]`
- `errors` : قائمة بالأخطاء التي حدثت أثناء العملية.

**مثال:**
```php
$result = UnitCreator::createProductUnitsStructure([
    'products_id' => 123,
    'units_data' => [
        ['unit_symbol' => 'kg', 'is_main' => true],
        ['unit_symbol' => 'g', 'conversion_factor' => 0.001],
    ],
    'prices_data' => [
        [ // أسعار للوحدة الأولى
            ['prices_id' => 1, 'value' => 10],
        ],
        [ // أسعار للوحدة الثانية
            ['prices_id' => 1, 'value' => 0.01],
        ],
    ],
]);
```

---

### `createUnitConversion`

```php
public static function createUnitConversion(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء علاقة تحويل بين وحدتين (للاستخدام المستقبلي). تتحقق من أن الوحدتين من نفس النوع، ثم تربط الوحدة الهدف بالمنتج.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `products_id` (int) : معرف المنتج.
  - `from_units_id` (int) : معرف الوحدة المصدر.
  - `to_units_id` (int) : معرف الوحدة الهدف.
  - `conversion_factor` (float) : عامل التحويل.

**الإرجاع:**  
مصفوفة تحتوي على نتيجة الربط.

---

### `importUnitsFromSymbols`

```php
public static function importUnitsFromSymbols(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
استيراد وحدات من قائمة رموز. تتحقق من وجود كل وحدة مسبقاً، وتنشئ الجديدة فقط.

**المعاملات:**
- `$options` : مصفوفة تحتوي على:
  - `unit_symbols` (array) : قائمة برموز الوحدات.
  - `common_data` (array) : بيانات مشتركة تضاف لكل وحدة.
  - `companys_id`, `departments_id`.

**الإرجاع:**  
مصفوفة تحتوي على:
- `data` : `['created' => [...], 'skipped' => [...], 'errors' => [...]]`

**مثال:**
```php
$result = UnitCreator::importUnitsFromSymbols([
    'unit_symbols' => ['kg', 'g', 'lb'],
    'common_data' => ['is_active' => true],
    'companys_id' => 1,
]);
```

---

### `createStandardQuantityUnits`

```php
public static function createStandardQuantityUnits(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
إنشاء وحدات الكميات القياسية (مثل 'pcs', 'dozen', 'box', ...). تستخدم قائمة محددة مسبقاً.

**المعاملات:**
- `$options` : تحتوي على `common_data`, `companys_id`, `departments_id`.

**الإرجاع:**  
مصفوفة تحتوي على الوحدات المنشأة.

---

### `firstOrCreateFromSymbol`

```php
public static function firstOrCreateFromSymbol(array $options = [], bool $is_test_create = false): array
```

**الوصف:**  
البحث عن وحدة برمز معين، وإنشاؤها إذا لم توجد.

**المعاملات:**
- `$options` : تحتوي على `symbol`, `additional_data`, `companys_id`, `departments_id`.

**الإرجاع:**  
مصفوفة تحتوي على `model` (الوحدة الموجودة أو الجديدة) و `process_data['is_existing']` للإشارة إلى ما إذا كانت موجودة مسبقاً.

**مثال:**
```php
$result = UnitCreator::firstOrCreateFromSymbol([
    'symbol' => 'kg',
    'companys_id' => 1,
]);

if ($result['process_data']['is_existing']) {
    echo "الوحدة موجودة مسبقاً.";
} else {
    echo "تم إنشاء الوحدة.";
}
```

---

## هيكل النتيجة (Result Array)

معظم دوال `UnitCreator` تُعيد مصفوفة بالهيكل التالي:

```php
[
    'code' => 200,                     // كود الحالة
    'status' => true,                   // نجاح/فشل
    'message' => 'رسالة توضيحية',       // رسالة عامة
    'error' => null,                     // نص الخطأ (إن وجد)
    'errors' => null,                    // تفاصيل الأخطاء (مصفوفة)
    'data' => null,                      // البيانات المطلوبة (قد تكون مصفوفة أو كائن)
    'model' => null,                      // كائن النموذج (إن وجد)
    'input_data' => [],                   // نسخة من المدخلات
    'process_data' => [],                 // بيانات إضافية عن العملية
    'debug' => []                         // معلومات للتصحيح (في وضع debug)
]
```

---

## أمثلة شاملة

### مثال 1: إنشاء وحدة جديدة وربطها بمنتج مع سعر
```php
use Tss\Inventory\Classes\Units\UnitCreator;

// 1. إنشاء الوحدة
$unitResult = UnitCreator::createUnitFromSymbol([
    'symbol' => 'dozen',
    'make_default' => true,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if (!$unitResult['status']) {
    throw new Exception($unitResult['error']);
}

$unit = $unitResult['model'];

// 2. ربط الوحدة بالمنتج (معرف المنتج 123)
$attachResult = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => $unit->id,
    'is_main' => true,
    'conversion_factor' => 12,
]);

if (!$attachResult['status']) {
    throw new Exception($attachResult['error']);
}

// 3. إنشاء سعر للوحدة
$priceResult = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => $unit->id,
    'prices_id' => 1,
    'value' => 50.00,
    'currencys_id' => 1,
    'is_default' => true,
]);

if ($priceResult['status']) {
    echo "تم إنشاء الوحدة وربطها بالمنتج وإضافة السعر بنجاح.";
}
```

### مثال 2: استيراد وحدات من قائمة رموز
```php
$symbols = ['kg', 'g', 'lb', 'oz'];
$result = UnitCreator::importUnitsFromSymbols([
    'unit_symbols' => $symbols,
    'common_data' => [
        'is_active' => true,
        'is_base_unit' => false,
    ],
    'companys_id' => 1,
    'departments_id' => 2,
]);

echo "تم إنشاء: " . count($result['data']['created']) . "\n";
echo "موجود مسبقاً: " . count($result['data']['skipped']) . "\n";
echo "أخطاء: " . count($result['data']['errors']) . "\n";
```

### مثال 3: إنشاء جميع وحدات الطول والوزن
```php
foreach ([UnitType::LENGTH, UnitType::WEIGHT] as $type) {
    $result = UnitCreator::createStandardUnitsForType([
        'type' => $type,
        'companys_id' => 1,
        'departments_id' => 2,
    ]);
    echo "النوع $type: تم إنشاء {$result['process_data']['created_count']} وحدة.\n";
}
```

### مثال 4: استخدام firstOrCreateFromSymbol
```php
$result = UnitCreator::firstOrCreateFromSymbol([
    'symbol' => 'kg',
    'additional_data' => ['is_base_unit' => true],
    'companys_id' => 1,
    'departments_id' => 2,
]);

$unit = $result['model'];
if ($result['process_data']['is_existing']) {
    echo "تم العثور على الوحدة: " . $unit->name;
} else {
    echo "تم إنشاء الوحدة: " . $unit->name;
}
```

---

بهذا نكون قد غطينا جميع دوال الكلاس `UnitCreator` مع شرح وافٍ وأمثلة عملية. هذا الكلاس يُعد الأداة الرئيسية لإنشاء وإدارة الوحدات في قاعدة البيانات، وهو جزء أساسي من نظام إدارة الوحدات.