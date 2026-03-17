# توثيق كلاسات التحويل والتحقق من الوحدات

## 📋 جدول المحتويات

1. [UnitConverter - محول الوحدات الأساسي](#unitconverter---محول-الوحدات-الأساسي)
   - [مقدمة](#مقدمة-unitconverter)
   - [الثوابت](#الثوابت-unitconverter)
   - [الدوال الرئيسية](#الدوال-الرئيسية-unitconverter)
   - [أمثلة شاملة](#أمثلة-شاملة-unitconverter)
2. [UnitContextValidator - مدقق سياق الوحدات](#unitcontextvalidator---مدقق-سياق-الوحدات)
   - [مقدمة](#مقدمة-unitcontextvalidator)
   - [الثوابت](#الثوابت-unitcontextvalidator)
   - [الدوال الرئيسية](#الدوال-الرئيسية-unitcontextvalidator)
   - [أمثلة](#أمثلة-unitcontextvalidator)
3. [SmartUnitConverter - محول ذكي](#smartunitconverter---محول-ذكي)
   - [مقدمة](#مقدمة-smartunitconverter)
   - [الدوال الرئيسية](#الدوال-الرئيسية-smartunitconverter)
   - [أمثلة](#أمثلة-smartunitconverter)
4. [الخلاصة](#الخلاصة)

---

## UnitConverter - محول الوحدات الأساسي

**Namespace:** `Tss\Inventory\Classes\Units`

### مقدمة {#مقدمة-unitconverter}

كلاس `UnitConverter` هو قلب نظام تحويل الوحدات. يحتوي على بيانات التحويل لجميع الوحدات المدعومة (أكثر من 80 وحدة موزعة على 16 نوعًا) ويقوم بإجراء التحويلات الرياضية بدقة عالية. يدعم التحويل بين الوحدات من نفس النوع، مع معالجة خاصة لدرجات الحرارة التي تتطلب معادلات خاصة. كما يوفر دوالاً مساعدة للاستعلام عن الوحدات المدعومة، الحصول على عوامل التحويل، والتحقق من إمكانية التحويل.

### الثوابت {#الثوابت-unitconverter}

#### `CONVERSION_DATA`
مصفوفة تحتوي على بيانات التحويل لجميع أنواع الوحدات. كل مفتاح رئيسي يمثل نوع الوحدة (مثل `'length'`، `'weight'`) وقيمته مصفوفة ترابطية من رموز الوحدات وعوامل التحويل بالنسبة للوحدة الأساسية لذلك النوع.

**مثال من الكود:**
```php
private const CONVERSION_DATA = [
    'length' => [
        'mm'    => 0.001,
        'cm'    => 0.01,
        'm'     => 1.0,
        'km'    => 1000.0,
        // ...
    ],
    'weight' => [
        'mg'    => 0.000001,
        'g'     => 0.001,
        'kg'    => 1.0,
        // ...
    ],
    // ... باقي الأنواع
];
```

#### `UNIT_CONTEXTS`
مصفوفة تحدد السياق (Context) لكل وحدة من وحدات الكميات (مثل `'dozen'` في سياق `'quantity_packaging'`). تستخدم في التحقق من منطقية التحويل في الوضع الصارم.

```php
private const UNIT_CONTEXTS = [
    'pcs'         => 'quantity_general',
    'dozen'       => 'quantity_packaging',
    'half_dozen'  => 'quantity_packaging',
    'box'         => 'quantity_packaging',
    // ...
];
```

### الدوال الرئيسية {#الدوال-الرئيسية-unitconverter}

#### `convert(float $value, string $from, string $to, int $precision = 6, bool $strictContext = false): float`

**الوصف:**  
تحويل قيمة من وحدة إلى أخرى. هذه هي الدالة الأساسية في الكلاس.

**المعاملات:**
- `$value` : القيمة الرقمية المراد تحويلها.
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.
- `$precision` : عدد المنازل العشرية للنتيجة (افتراضي 6).
- `$strictContext` : إذا كانت `true`، يتم تطبيق التحقق الصارم من السياق لوحدات الكميات (يمنع التحويل بين سياقات مختلفة).

**الإرجاع:**  
القيمة المحولة (float).

**الاستثناءات:**
- `InvalidArgumentException` إذا كانت الوحدة غير معروفة، أو إذا كان التحويل غير ممكن (أنواع مختلفة)، أو إذا كان التحويل غير منطقي في الوضع الصارم.

**أمثلة:**
```php
// تحويل 5 كيلوجرام إلى جرام
$result = UnitConverter::convert(5, 'kg', 'g'); // 5000

// تحويل 1 متر إلى سنتيمتر بدقة منزلتين عشريتين
$result = UnitConverter::convert(1, 'm', 'cm', 2); // 100.00

// تحويل درجة حرارة
$result = UnitConverter::convert(100, 'c', 'f', 2); // 212.00

// تحويل وحدات كميات (دزينة إلى حبة)
$result = UnitConverter::convert(2, 'dozen', 'pcs'); // 24

// تحويل مع الوضع الصارم (سيمنع التحويل من دزينة إلى ريم لأنهما في سياقين مختلفين)
try {
    $result = UnitConverter::convert(1, 'dozen', 'ream', 6, true);
} catch (InvalidArgumentException $e) {
    echo $e->getMessage(); // "تحويل غير منطقي: دزينة إلى ريم"
}
```

#### `convertBatch(array $values, string $from, string $to, int $precision = 6): array`

**الوصف:**  
تحويل مجموعة من القيم دفعة واحدة من وحدة إلى أخرى.

**المعاملات:**
- `$values` : مصفوفة من القيم الرقمية.
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.
- `$precision` : عدد المنازل العشرية.

**الإرجاع:**  
مصفوفة من القيم المحولة بنفس ترتيب المدخلات.

**مثال:**
```php
$values = [1, 2, 3, 4, 5];
$results = UnitConverter::convertBatch($values, 'm', 'cm', 2);
// [100.00, 200.00, 300.00, 400.00, 500.00]
```

#### `describeConversion(float $value, string $from, string $to, int $precision = 6): string`

**الوصف:**  
إرجاع وصف نصي للتحويل مناسب للعرض للمستخدم.

**المعاملات:**
- `$value` : القيمة المراد تحويلها.
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.
- `$precision` : عدد المنازل العشرية.

**الإرجاع:**  
سلسلة نصية مثل `"1 متر = 100.00 سنتيمتر"`.

**مثال:**
```php
echo UnitConverter::describeConversion(1, 'm', 'cm', 2);
// "1 متر = 100.00 سنتيمتر"
```

#### `isUnitSupported(string $unit): bool`

**الوصف:**  
التحقق مما إذا كانت الوحدة مدعومة في النظام.

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
`true` إذا كانت الوحدة موجودة في بيانات التحويل.

**مثال:**
```php
if (UnitConverter::isUnitSupported('kg')) {
    echo "مدعومة";
}
```

#### `getUnitType(string $unit): string`

**الوصف:**  
الحصول على نوع الوحدة (مثل `'length'`، `'weight'`).

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
نوع الوحدة كسلسلة نصية.

**الاستثناءات:**  
`InvalidArgumentException` إذا كانت الوحدة غير معروفة.

**مثال:**
```php
$type = UnitConverter::getUnitType('kg'); // 'weight'
```

#### `getConversionFactor(string $unit, string $type): float`

**الوصف:**  
الحصول على عامل التحويل للوحدة نسبة إلى الوحدة الأساسية للنوع. هذه دالة داخلية تستخدمها `convert`.

**المعاملات:**
- `$unit` : رمز الوحدة.
- `$type` : نوع الوحدة.

**الإرجاع:**  
عامل التحويل (float).

#### `getConversionFactorUnit(string $unit): float`

**الوصف:**  
الحصول على عامل التحويل للوحدة (بالنسبة للوحدة الأساسية للنوع). هذه الدالة أكثر أمانًا لأنها تحدد النوع تلقائيًا.

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
عامل التحويل.

**الاستثناءات:**  
- إذا كانت الوحدة غير معروفة.
- إذا كانت الوحدة من نوع درجة حرارة (ليس لها عامل تحويل ثابت).
- إذا كان عامل التحويل غير صحيح.

**مثال:**
```php
$factor = UnitConverter::getConversionFactorUnit('km'); // 1000.0
```

#### `canConvert(string $from, string $to): bool`

**الوصف:**  
التحقق من إمكانية التحويل بين وحدتين دون رمي استثناء.

**المعاملات:**
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.

**الإرجاع:**  
`true` إذا كان التحويل ممكنًا، `false` إذا كان غير ممكن.

**مثال:**
```php
if (UnitConverter::canConvert('dozen', 'box')) {
    echo "يمكن التحويل";
}
```

#### `getUnitsByType(string $type, bool $withLabels = false, $lang = 'ar'): array`

**الوصف:**  
إرجاع قائمة بالوحدات التي تنتمي إلى نوع معين.

**المعاملات:**
- `$type` : نوع الوحدة (أحد ثوابت `UnitType`).
- `$withLabels` : إذا كانت `true`، تُرجع مصفوفة ترابطية `[الرمز => التسمية]`؛ وإلا تُرجع قائمة عادية من الرموز.
- `$lang` : اللغة المستخدمة في التسميات (إذا طلبت).

**الإرجاع:**  
مصفوفة من الرموز أو التسميات.

**مثال:**
```php
$lengthUnits = UnitConverter::getUnitsByType(UnitType::LENGTH, true, 'ar');
/*
[
    'mm' => 'مليمتر',
    'cm' => 'سنتيمتر',
    'm' => 'متر',
    ...
]
*/
```

#### `getAllUnitsGrouped(bool $groupByMainType = true, $lang = 'ar'): array`

**الوصف:**  
إرجاع جميع الوحدات مصنفة حسب النوع، مع تسميات. مفيدة لبناء واجهات المستخدم.

**المعاملات:**
- `$groupByMainType` : إذا كانت `true`، يتم التجميع حسب الأنواع الرئيسية (مثل `length`, `weight`). إذا كانت `false`، يتم تضمين الأنواع الفرعية.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية حيث المفتاح هو النوع، والقيمة مصفوفة تحتوي على `label` و `units` و `type`.

**مثال:**
```php
$grouped = UnitConverter::getAllUnitsGrouped(true, 'ar');
foreach ($grouped as $type => $info) {
    echo $info['label'] . ":\n";
    foreach ($info['units'] as $symbol => $label) {
        echo "  $symbol : $label\n";
    }
}
```

#### `getUnitInfo(string $unit, $lang = 'ar'): array`

**الوصف:**  
الحصول على معلومات كاملة عن وحدة معينة، تتضمن الرمز، النوع، التسمية، عامل التحويل، السياق، وما إذا كانت وحدة درجة حرارة أو كمية.

**المعاملات:**
- `$unit` : رمز الوحدة.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة تحتوي على:
- `symbol` : رمز الوحدة.
- `type` : نوع الوحدة.
- `label` : التسمية حسب اللغة.
- `base_factor` : عامل التحويل.
- `context` : السياق (إن وجد).
- `is_temperature` : هل هي وحدة درجة حرارة.
- `is_quantity` : هل هي وحدة كمية.
- بالإضافة إلى المعلومات من `UnitNameHelper::getUnitInfo`.

**الاستثناءات:**  
`InvalidArgumentException` إذا كانت الوحدة غير مدعومة.

**مثال:**
```php
$info = UnitConverter::getUnitInfo('dozen', 'ar');
/*
[
    'symbol' => 'dozen',
    'type' => 'quantity_packaging',
    'label' => 'درزن',
    'base_factor' => 12.0,
    'context' => 'quantity_packaging',
    'is_temperature' => false,
    'is_quantity' => true,
    ...
]
*/
```

#### `getSupportedUnits($type = null, bool $includeSubtypes = false, $lang = 'ar'): array`

**الوصف:**  
إرجاع قائمة بجميع رموز الوحدات المدعومة، أو لنوع معين مع إمكانية تضمين الأنواع الفرعية.

**المعاملات:**
- `$type` : نوع الوحدة (إذا كان `null`، ترجع كل الوحدات).
- `$includeSubtypes` : إذا كانت `true`، يتم تضمين الأنواع الفرعية (مثل `quantity_packaging`). إذا كانت `false`، يتم التعامل مع `QUANTITY` كمجموع للأنواع الفرعية.
- `$lang` : اللغة (تُستخدم فقط عند استدعاء دوال أخرى، لكنها غير مستخدمة هنا مباشرة).

**الإرجاع:**  
مصفوفة من رموز الوحدات.

**أمثلة:**
```php
// كل الوحدات
$all = UnitConverter::getSupportedUnits();

// وحدات الطول
$lengthUnits = UnitConverter::getSupportedUnits('length');

// وحدات الكميات (جميع الأنواع الفرعية)
$quantityUnits = UnitConverter::getSupportedUnits(UnitType::QUANTITY, false);
// ['pcs', 'dozen', 'half_dozen', 'box', ...]
```

#### `getSupportedUnitsWithLabels($type = null, bool $includeSubtypes = false, $lang = 'ar'): array`

**الوصف:**  
مثل `getSupportedUnits` لكن مع التسميات.

**الإرجاع:**  
مصفوفة ترابطية `[الرمز => التسمية]`.

#### `getSupportedConversionTypes(bool $includeSubtypes = false): array`

**الوصف:**  
إرجاع قائمة بأنواع التحويل المدعومة (أسماء الأنواع).

**المعاملات:**
- `$includeSubtypes` : تضمين الأنواع الفرعية أم لا.

**الإرجاع:**  
مصفوفة من أسماء الأنواع.

#### `getSupportedConversionTypesWithLabels(bool $includeSubtypes = false, $lang = 'ar'): array`

**الوصف:**  
مثل السابقة مع التسميات.

#### `isTypeSupported(string $type): bool`

**الوصف:**  
التحقق مما إذا كان نوع الوحدة مدعومًا.

#### `getUnitContext(string $unit): ?string`

**الوصف:**  
الحصول على السياق الخاص بالوحدة (للوحدات الكمية). تُرجع `null` إذا لم يكن للوحدة سياق.

**مثال:**
```php
$context = UnitConverter::getUnitContext('dozen'); // 'quantity_packaging'
```

### دوال داخلية خاصة

- `buildUnitLookupCache()`: بناء كاش للبحث السريع عن الوحدات.
- `validateUnit(string $unit)`: التحقق من صحة الوحدة.
- `validateCompatibleTypes()`: التحقق من توافق الأنواع في الوضع العادي.
- `validateStrictConversion()`: التحقق من التوافق في الوضع الصارم.
- `convertTemperature()`: معالجة تحويل درجات الحرارة.
- `isTemperatureUnit()`: التحقق مما إذا كانت الوحدة من وحدات درجة الحرارة.
- `hasQuantityUnits()`: التحقق من وجود وحدات كميات.

### أمثلة شاملة {#أمثلة-شاملة-unitconverter}

#### مثال 1: تحويلات متنوعة
```php
// طول
echo UnitConverter::convert(10, 'km', 'm'); // 10000

// وزن
echo UnitConverter::convert(2.5, 'kg', 'lb', 2); // 5.51

// حجم
echo UnitConverter::convert(3, 'l', 'ml'); // 3000

// مساحة
echo UnitConverter::convert(1, 'acre', 'm2', 2); // 4046.86

// سرعة
echo UnitConverter::convert(100, 'km/h', 'm/s', 2); // 27.78

// ضغط
echo UnitConverter::convert(1, 'bar', 'psi', 2); // 14.50

// طاقة
echo UnitConverter::convert(500, 'cal', 'kj', 4); // 2.0920

// قدرة
echo UnitConverter::convert(10, 'hp', 'kw', 2); // 7.46

// بيانات
echo UnitConverter::convert(1, 'gb', 'mb'); // 1024

// زاوية
echo UnitConverter::convert(180, 'deg', 'rad', 4); // 3.1416
```

#### مثال 2: التحقق من إمكانية التحويل
```php
$from = 'dozen';
$to = 'box';

if (UnitConverter::canConvert($from, $to)) {
    $result = UnitConverter::convert(2, $from, $to);
    echo "2 دزينة = $result علبة";
} else {
    echo "لا يمكن التحويل";
}
```

#### مثال 3: الحصول على معلومات وحدة
```php
$unit = 'kg';
if (UnitConverter::isUnitSupported($unit)) {
    $info = UnitConverter::getUnitInfo($unit, 'ar');
    echo "الوحدة: {$info['label']}\n";
    echo "النوع: {$info['type']}\n";
    echo "عامل التحويل: {$info['base_factor']}\n";
}
```

#### مثال 4: استخدام convertBatch
```php
$kilograms = [1, 2, 3, 4, 5];
$pounds = UnitConverter::convertBatch($kilograms, 'kg', 'lb', 2);
// [2.20, 4.41, 6.61, 8.82, 11.02]
```

#### مثال 5: الحصول على جميع وحدات نوع معين
```php
$weightUnits = UnitConverter::getSupportedUnitsWithLabels('weight', false, 'ar');
foreach ($weightUnits as $symbol => $label) {
    echo "$symbol : $label\n";
}
```

---

## UnitContextValidator - مدقق سياق الوحدات

**Namespace:** `Tss\Inventory\Classes\Units`

### مقدمة {#مقدمة-unitcontextvalidator}

كلاس `UnitContextValidator` مسؤول عن التحقق من منطقية تحويل وحدات الكميات بناءً على سياقها. يمنع التحويلات غير المنطقية مثل تحويل دزينة (تعبئة) إلى ريم (ورق). يعتمد على قاعدة أساسية أن وحدة `pcs` (الحبة) هي وحدة عالمية يمكن تحويلها من وإلى أي وحدة كمية.

### الثوابت {#الثوابت-unitcontextvalidator}

#### `UNIVERSAL_UNIT`
```php
private const UNIVERSAL_UNIT = 'pcs';
```
الوحدة العالمية التي يمكن تحويلها إلى أي وحدة كمية أخرى.

#### `CONVERSION_RULES`
مصفوفة من القواعد المحددة التي تحدد العلاقات المنطقية بين أزواج معينة من الوحدات. كل قاعدة تحتوي على:
- الوحدة المصدر
- الوحدة الهدف
- `true` إذا كان التحويل منطقيًا، `false` إذا كان غير منطقي
- سبب منطقي (نص وصفي)

```php
private const CONVERSION_RULES = [
    ['dozen', 'box', true, 'تعبئة إلى تعبئة'],
    ['box', 'dozen', true, 'تعبئة إلى تعبئة'],
    ['pair', 'dozen', true, 'أزواج إلى تعبئة'],
    ['dozen', 'pair', true, 'تعبئة إلى أزواج'],
    ['ream', 'dozen', false, 'ورق إلى تعبئة (غير منطقي)'],
    // ...
];
```

### الدوال الرئيسية {#الدوال-الرئيسية-unitcontextvalidator}

#### `isConversionLogical(string $from, string $to): bool`

**الوصف:**  
التحقق مما إذا كان التحويل بين وحدتين منطقياً.

**المعاملات:**
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.

**الإرجاع:**  
`true` إذا كان التحويل منطقياً، `false` إذا كان غير منطقي.

**آلية العمل:**
1. إذا كانت إحدى الوحدتين هي `pcs`، يُعتبر التحويل منطقياً دائماً.
2. يتم البحث في `CONVERSION_RULES` عن تطابق تام للزوج `($from, $to)`. إذا وُجد، تُرجع القيمة المسجلة.
3. إذا لم يوجد، يتم استخراج سياق كل وحدة عبر `UnitNameHelper::getUnitInfo`، وإذا تطابق السياقان، يُعتبر التحويل منطقياً. وإلا فهو غير منطقي.

**أمثلة:**
```php
// تحويل منطقي
UnitContextValidator::isConversionLogical('dozen', 'box'); // true
UnitContextValidator::isConversionLogical('pair', 'dozen'); // true
UnitContextValidator::isConversionLogical('pcs', 'ream'); // true (لأن pcs عالمية)

// تحويل غير منطقي
UnitContextValidator::isConversionLogical('dozen', 'ream'); // false
UnitContextValidator::isConversionLogical('ream', 'pair'); // false
```

#### `getConversionReason(string $from, string $to): string`

**الوصف:**  
إرجاع سبب منطقية أو عدم منطقية التحويل كنص وصفي.

**المعاملات:**
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.

**الإرجاع:**  
سلسلة نصية تشرح السبب.

**أمثلة:**
```php
echo UnitContextValidator::getConversionReason('dozen', 'box');
// "نفس السياق: كميات التعبئة"

echo UnitContextValidator::getConversionReason('dozen', 'ream');
// "تحويل بين سياقات مختلفة: كميات التعبئة إلى كميات الورق"

echo UnitContextValidator::getConversionReason('pcs', 'dozen');
// "الحبة هي الوحدة الأساسية للكميات"
```

#### `getCompatibleUnits(string $unit): array`

**الوصف:**  
إرجاع قائمة بالوحدات المتوافقة منطقياً مع وحدة معينة (باستثناء الوحدة نفسها). كل عنصر في المصفوفة هو مصفوفة تحتوي على `unit` (الرمز) و `label` (التسمية).

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
مصفوفة من الوحدات المتوافقة.

**مثال:**
```php
$compatible = UnitContextValidator::getCompatibleUnits('dozen');
foreach ($compatible as $item) {
    echo "{$item['unit']} : {$item['label']}\n";
}
// box : علبة
// carton : كرتونة
// half_dozen : نصف درزن
// pack : عبوة
// tray : صينية
// pair : زوج
// pcs : حبة
// ...
```

### دوال داخلية خاصة

- `getUnitContext(string $unit): ?string` : الحصول على سياق الوحدة باستخدام `UnitNameHelper`.
- `getContextLabel(?string $context): string` : الحصول على التسمية العربية للسياق.

### أمثلة {#أمثلة-unitcontextvalidator}

#### مثال 1: التحقق من منطقية التحويل قبل إجرائه
```php
$from = 'dozen';
$to = 'ream';

if (UnitContextValidator::isConversionLogical($from, $to)) {
    $result = UnitConverter::convert(1, $from, $to);
    echo "النتيجة: $result";
} else {
    echo "التحويل غير منطقي: " . UnitContextValidator::getConversionReason($from, $to);
}
```

#### مثال 2: الحصول على الوحدات المتوافقة لعرضها في واجهة المستخدم
```php
$unit = 'box';
$compatible = UnitContextValidator::getCompatibleUnits($unit);

echo "<select name='to_unit'>";
foreach ($compatible as $item) {
    echo "<option value='{$item['unit']}'>{$item['label']}</option>";
}
echo "</select>";
```

#### مثال 3: استخدام القواعد المخصصة
```php
// يمكن إضافة قواعد جديدة في التوسعات المستقبلية
```

---

## SmartUnitConverter - محول ذكي

**Namespace:** `Tss\Inventory\Classes\Units`

### مقدمة {#مقدمة-smartunitconverter}

كلاس `SmartUnitConverter` يوفر واجهة أكثر أماناً وذكاءً للتحويل. يقوم بالتحقق من منطقية التحويل قبل إجرائه، ويمكنه إرجاع تحذيرات بدلاً من رمي استثناءات. مناسب للاستخدام في التطبيقات التي تحتاج إلى توجيه المستخدم أو منع التحويلات غير المنطقية.

### الدوال الرئيسية {#الدوال-الرئيسية-smartunitconverter}

#### `safeConvert(float $value, string $from, string $to, int $precision = 6): float`

**الوصف:**  
تحويل آمن مع التحقق من المنطقية. إذا كان التحويل غير منطقي، يتم رمي استثناء.

**المعاملات:**
- `$value` : القيمة المراد تحويلها.
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.
- `$precision` : عدد المنازل العشرية.

**الإرجاع:**  
القيمة المحولة (float).

**الاستثناءات:**  
`InvalidArgumentException` إذا كان التحويل غير منطقي (حسب `UnitContextValidator::isConversionLogical`).

**مثال:**
```php
try {
    $result = SmartUnitConverter::safeConvert(2, 'dozen', 'box');
    echo "2 دزينة = $result علبة";
} catch (InvalidArgumentException $e) {
    echo "خطأ: " . $e->getMessage();
}
```

#### `convertWithWarning(float $value, string $from, string $to, int $precision = 6): array`

**الوصف:**  
تحويل مع إرجاع تحذير إذا كان التحويل غير منطقي. لا يرمي استثناء، بل يعيد مصفوفة تحتوي على النتيجة ومعلومات إضافية.

**المعاملات:**
- `$value` : القيمة المراد تحويلها.
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.
- `$precision` : عدد المنازل العشرية.

**الإرجاع:**  
مصفوفة تحتوي على:
- `value` : القيمة المحولة.
- `unit` : رمز الوحدة الهدف.
- `unit_label` : تسمية الوحدة الهدف.
- `is_logical` : `true` إذا كان التحويل منطقياً، `false` إذا كان غير منطقي.
- `warning` : نص تحذيري (إذا كان التحويل غير منطقي) أو `null`.
- `description` : وصف التحويل من `UnitConverter::describeConversion`.

**مثال:**
```php
$result = SmartUnitConverter::convertWithWarning(1, 'dozen', 'ream', 2);

echo $result['description']; // "1 درزن = 0.04 ريم"
if (!$result['is_logical']) {
    echo "تحذير: " . $result['warning'];
}
// تحذير: ⚠️ تحذر: التحويل من درزن إلى ريم قد لا يكون منطقياً في التطبيقات العملية
```

### أمثلة {#أمثلة-smartunitconverter}

#### مثال 1: استخدام safeConvert مع التحقق
```php
function convertForUser($value, $from, $to) {
    try {
        $result = SmartUnitConverter::safeConvert($value, $from, $to, 2);
        return [
            'success' => true,
            'result' => $result,
            'message' => UnitConverter::describeConversion($value, $from, $to, 2)
        ];
    } catch (InvalidArgumentException $e) {
        return [
            'success' => false,
            'message' => $e->getMessage()
        ];
    }
}

$output = convertForUser(3, 'dozen', 'box');
if ($output['success']) {
    echo $output['message'];
} else {
    echo "عذراً، لا يمكن إجراء هذا التحويل: " . $output['message'];
}
```

#### مثال 2: استخدام convertWithWarning لتقديم توصيات
```php
$from = 'dozen';
$to = 'ream';
$value = 5;

$result = SmartUnitConverter::convertWithWarning($value, $from, $to, 2);

if ($result['is_logical']) {
    echo "النتيجة: {$result['description']}";
} else {
    echo "تنبيه: {$result['warning']}\n";
    echo "لكن يمكنك إجراء التحويل إذا أردت: {$result['description']}\n";
    echo "الوحدات المتوافقة منطقياً: ";
    $compatible = UnitContextValidator::getCompatibleUnits($from);
    $labels = array_column($compatible, 'label', 'unit');
    echo implode(', ', array_slice($labels, 0, 3)) . "...";
}
```

#### مثال 3: دمج مع UnitConverter و UnitContextValidator
```php
class ConversionService
{
    public function convert($value, $from, $to, $mode = 'safe')
    {
        if (!UnitConverter::isUnitSupported($from) || !UnitConverter::isUnitSupported($to)) {
            throw new InvalidArgumentException("وحدة غير مدعومة");
        }
        
        if ($mode === 'safe') {
            return SmartUnitConverter::safeConvert($value, $from, $to);
        } elseif ($mode === 'warning') {
            return SmartUnitConverter::convertWithWarning($value, $from, $to);
        } else {
            return UnitConverter::convert($value, $from, $to);
        }
    }
}
```

---

## الخلاصة

- **`UnitConverter`** هو المحول الأساسي الذي يحتوي على جميع بيانات التحويل ويقوم بالعمليات الحسابية. يوفر دوالاً شاملة للتحويل والاستعلام.
- **`UnitContextValidator`** يضيف طبقة من المنطقية للتحويلات بين وحدات الكميات، مما يمنع التحويلات غير المنطقية ويساعد في بناء تطبيقات أكثر ذكاءً.
- **`SmartUnitConverter`** يقدم واجهة سهلة وآمنة تجمع بين قوة `UnitConverter` ومنطقية `UnitContextValidator`، مع خيارات للتعامل مع التحذيرات بدلاً من الاستثناءات.

باستخدام هذه الكلاسات معاً، يمكن بناء نظام متكامل لتحويل الوحدات يدعم الدقة العالية، المنطقية، والمرونة في التعامل مع مختلف حالات الاستخدام.