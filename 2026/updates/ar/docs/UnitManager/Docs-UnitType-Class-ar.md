# توثيق كلاسات نظام الوحدات

## 📋 جدول المحتويات

1. [UnitType - تعريف أنواع الوحدات](#unittype---تعريف-أنواع-الوحدات)
   - [الثوابت](#الثوابت)
   - [الدوال الرئيسية](#الدوال-الرئيسية-لـ-unittype)
   - [أمثلة](#أمثلة-unittype)
2. [Unit - كائن الوحدة](#unit---كائن-الوحدة)
   - [الخصائص](#الخصائص)
   - [الدوال الرئيسية](#الدوال-الرئيسية-لـ-unit)
   - [أمثلة](#أمثلة-unit)
3. [UnitFactory - مصنع الوحدات](#unitfactory---مصنع-الوحدات)
   - [الدوال الرئيسية](#الدوال-الرئيسية-لـ-unitfactory)
   - [أمثلة](#أمثلة-unitfactory)

---

## UnitType - تعريف أنواع الوحدات

**Namespace:** `Tss\Inventory\Classes\Units`

### مقدمة
كلاس ثابت (static) يُستخدم لتعريف وتصنيف أنواع الوحدات المدعومة في النظام. يوفر ثوابت لأنواع الوحدات الرئيسية والفرعية، بالإضافة إلى دوال مساعدة للتحقق من الأنواع والحصول على تسمياتها المترجمة.

### الثوابت

| الثابت | القيمة | الوصف |
|--------|--------|-------|
| `LENGTH` | `'length'` | الطول |
| `WEIGHT` | `'weight'` | الوزن |
| `VOLUME` | `'volume'` | الحجم |
| `AREA` | `'area'` | المساحة |
| `TEMPERATURE` | `'temperature'` | درجة الحرارة |
| `TIME` | `'time'` | الزمن |
| `SPEED` | `'speed'` | السرعة |
| `PRESSURE` | `'pressure'` | الضغط |
| `ENERGY` | `'energy'` | الطاقة |
| `POWER` | `'power'` | القدرة |
| `DATA` | `'data'` | البيانات |
| `ANGLE` | `'angle'` | الزوايا |
| `QUANTITY` | `'quantity'` | الكميات (النوع الرئيسي) |
| `QUANTITY_GENERAL` | `'quantity_general'` | كميات عامة |
| `QUANTITY_PACKAGING` | `'quantity_packaging'` | كميات التعبئة |
| `QUANTITY_PAPER` | `'quantity_paper'` | كميات الورق |
| `QUANTITY_PAIRS` | `'quantity_pairs'` | كميات الأزواج |
| `QUANTITY_LARGE` | `'quantity_large'` | كميات كبيرة |
| `CUSTOM` | `'custom'` | وحدات مخصصة |

### الدوال الرئيسية لـ UnitType

#### `getMainTypesWithLabels($lang = null): array`
ترجع الأنواع الرئيسية مع تسمياتها.

- **المعاملات:**
  - `$lang` : اللغة (`'ar'` أو `'en'`). إذا لم تُمرر، تُستخدم اللغة الحالية.
- **الإرجاع:** مصفوفة ترابطية `[type => label]`.

**مثال:**
```php
$types = UnitType::getMainTypesWithLabels('ar');
/*
[
    'length' => 'الطول',
    'weight' => 'الوزن',
    'volume' => 'الحجم',
    ...
]
*/
```

#### `getAllTypesWithLabels($lang = null): array`
ترجع جميع الأنواع (بما في ذلك الفرعية) مع تسمياتها.

#### `getQuantitySubtypesWithLabels($lang = null): array`
ترجع الأنواع الفرعية للكميات فقط.

#### `getTypesWithLabels(array $types, $lang = null): array`
ترجع تسميات لأنواع محددة.

- **المعاملات:**
  - `$types` : مصفوفة من أسماء الأنواع.
- **الإرجاع:** مصفوفة ترابطية `[type => label]` للأنواع الموجودة فقط.

#### `getCategorizedTypes($lang = null): array`
ترجع الأنواع مصنفة حسب الفئة (رئيسية، كميات، فرعية، مخصصة).

**مثال:**
```php
$categorized = UnitType::getCategorizedTypes('ar');
/*
[
    'main' => ['length' => 'الطول', ...],
    'quantity' => ['quantity' => 'الكميات'],
    'quantity_subtypes' => ['quantity_general' => 'كميات عامة', ...],
    'custom' => ['custom' => 'وحدات مخصصة'],
]
*/
```

#### `isValid(string $type): bool`
التحقق مما إذا كان النوع معرفًا (أحد الثوابت).

- **الإرجاع:** `true` إذا كان النوع صالحًا.

#### `isQuantitySubtype(string $type): bool`
التحقق مما إذا كان النوع فرعيًا للكميات (يبدأ بـ `quantity_`).

#### `isCustomType(string $type): bool`
التحقق مما إذا كان النوع مخصصًا (يساوي `custom` أو يبدأ بـ `custom_`).

#### `getMainType(string $type): string`
إرجاع النوع الرئيسي للنوع المُعطى.

- بالنسبة للأنواع الفرعية للكميات، تُرجع `QUANTITY`.
- للأنواع المخصصة، تُرجع `CUSTOM`.
- وإلا تُرجع النوع نفسه.

#### `getArabicLabel(string $type): string`
إرجاع التسمية العربية الافتراضية للنوع (بدون ترجمة من الملفات).

#### `getTransLabel(string $type, $lang = null): string`
إرجاع التسمية المترجمة للنوع حسب اللغة. تبحث في ملفات الترجمة أولاً، ثم تعود للتسمية العربية أو الإنجليزية.

#### `getEnglishLabel(string $type): string`
إرجاع التسمية الإنجليزية الافتراضية.

#### `getAllTypes(): array`
ترجع قائمة بجميع الأنواع (قيم الثوابت).

#### `getMainTypes(): array`
ترجع قائمة بالأنواع الرئيسية فقط.

#### `getSelectOptions(string $selectedType = '', bool $includeEmpty = true, bool $onlyMainTypes = false): string`
توليد خيارات HTML `<select>` للأنواع.

- **المعاملات:**
  - `$selectedType` : النوع المحدد مسبقًا (يُضاف له `selected`).
  - `$includeEmpty` : إضافة خيار فارغ في البداية.
  - `$onlyMainTypes` : عرض الأنواع الرئيسية فقط (إذا `false`، تعرض الكل بما في ذلك الفرعية).
- **الإرجاع:** كود HTML للخيارات.

#### `getQuantitySubtypesSelectOptions(string $selectedType = '', bool $includeEmpty = true): string`
توليد خيارات HTML للأنواع الفرعية للكميات فقط.

#### `filterByPrefix(string $prefix, bool $withLabels = true, $lang = null)`
ترجع الأنواع التي تبدأ ببادئة معينة.

- **المعاملات:**
  - `$prefix` : البادئة (مثل `'quantity_'`).
  - `$withLabels` : إذا `true`، ترجع مصفوفة ترابطية `[type => label]`، وإلا ترجع قائمة عادية.
- **الإرجاع:** مصفوفة.

### أمثلة UnitType

```php
// 1. التحقق من النوع
if (UnitType::isValid('weight')) {
    echo "نوع صالح";
}

// 2. الحصول على التسمية المترجمة
echo UnitType::getTransLabel('length', 'ar'); // "الطول"

// 3. معرفة النوع الرئيسي لفرعي
$main = UnitType::getMainType('quantity_packaging'); // "quantity"

// 4. إنشاء قائمة منسدلة للأنواع الرئيسية
echo '<select name="type">';
echo UnitType::getSelectOptions('weight', true, true);
echo '</select>';
```

---

## Unit - كائن الوحدة

**Namespace:** `Tss\Inventory\Classes\Units`

### مقدمة
يمثل هذا الكائن وحدة قياس واحدة بخصائصها (الرمز، النوع، التسمية، السياق) ويوفر دوالاً للتحويل والمقارنة والاستعلام. يُنشأ عادةً عبر `UnitFactory`.

### الخصائص

| الخاصية | النوع | الوصف |
|---------|-------|-------|
| `symbol` | `string` | رمز الوحدة (مثل `'kg'`) |
| `type` | `string` | نوع الوحدة (أحد ثوابت `UnitType`) |
| `label` | `string` | التسمية العربية للوحدة |
| `context` | `string\|null` | السياق (للوحدات الكمية: `general`, `packaging`, ...) |

جميع الخصائص للقراءة فقط (`readonly`).

### الدوال الرئيسية لـ Unit

#### `__construct(string $symbol, string $type, string $label, ?string $context = null)`
يُنشئ كائن وحدة جديد. يستدعي داخلياً `UnitNameHelper::getUnitInfo` لتخزين معلومات إضافية.

#### `convertTo(float $value, string $toUnit, int $precision = 6): float`
يحول قيمة من هذه الوحدة إلى وحدة أخرى.

- **المعاملات:**
  - `$value` : القيمة المراد تحويلها.
  - `$toUnit` : رمز الوحدة الهدف.
  - `$precision` : عدد المنازل العشرية (افتراضي 6).
- **الإرجاع:** القيمة المحولة.
- **الاستثناءات:** يرمي `InvalidArgumentException` إذا كانت الوحدة الهدف غير مدعومة أو التحويل غير ممكن.

**مثال:**
```php
$kg = UnitFactory::createFromString('kg');
$grams = $kg->convertTo(5, 'g', 2); // 5000.00
```

#### `convertToBase(float $value, int $precision = 6): float`
يحول القيمة إلى الوحدة الأساسية لهذا النوع.

- **الإرجاع:** القيمة بالوحدة الأساسية.
- **الاستثناءات:** يرمي `InvalidArgumentException` إذا لم توجد وحدة أساسية للنوع.

**مثال:**
```php
$dozen = UnitFactory::createFromString('dozen');
$pieces = $dozen->convertToBase(2, 2); // 24.00 (لأن الأساسية pcs)
```

#### `isCompatibleWith(string $otherUnit): bool`
التحقق مما إذا كانت هذه الوحدة متوافقة مع وحدة أخرى (أي يمكن التحويل بينهما منطقيًا).

- **المعاملات:**
  - `$otherUnit` : رمز الوحدة الأخرى.
- **الإرجاع:** `true` إذا كانت الوحدتان من نفس النوع الرئيسي.

**مثال:**
```php
$meter = UnitFactory::createFromString('m');
if ($meter->isCompatibleWith('cm')) {
    echo "يمكن التحويل";
}
```

#### `getConversionFactor(): float`
إرجاع عامل التحويل لهذه الوحدة نسبة إلى الوحدة الأساسية لنوعها.

**مثال:**
```php
$km = UnitFactory::createFromString('km');
echo $km->getConversionFactor(); // 1000.0
```

#### `isBaseUnit(): bool`
التحقق مما إذا كانت هذه الوحدة هي الوحدة الأساسية لنوعها.

#### `getEnglishLabel(): string`
إرجاع التسمية الإنجليزية للوحدة.

#### `getInfo(): array`
إرجاع معلومات كاملة عن الوحدة (باستدعاء `UnitNameHelper::getUnitInfo`).

#### `getContext(): ?string`
إرجاع السياق (للوحدات الكمية).

#### `getCompatibleUnits(): array`
إرجاع مصفوفة من رموز الوحدات المتوافقة (من نفس النوع) مع تسمياتها.

- **الإرجاع:** مصفوفة ترابطية `[symbol => label]`.

#### `canConvertTo(string $toUnit): bool`
التحقق مما إذا كان يمكن التحويل إلى وحدة معينة (دون تنفيذ التحويل الفعلي).

#### `formatValue(float $value, int $decimals = 2): string`
تنسيق القيمة مع تسمية الوحدة بالعربية.

- **الإرجاع:** نص منسق مثل `"5.00 كيلوجرام"`.

#### `__toString(): string`
تمثيل نصي للوحدة بالصيغة `"symbol (label)"`.

**مثال:**
```php
echo $kg; // "kg (كيلوجرام)"
```

#### `equals(Unit $other): bool`
مقارنة هذه الوحدة مع وحدة أخرى (حسب الرمز والنوع).

#### `copy(): self`
إنشاء نسخة جديدة من الوحدة.

### أمثلة Unit

```php
// إنشاء كائن وحدة عبر المصنع
$meter = UnitFactory::createFromString('m');

// تحويل 100 متر إلى سنتيمتر
$cm = $meter->convertTo(100, 'cm'); // 10000

// التحقق من التوافق
if ($meter->isCompatibleWith('km')) {
    echo "نعم";
}

// الحصول على معلومات كاملة
$info = $meter->getInfo();
echo $info['base_unit']; // 'm'

// تنسيق قيمة
echo $meter->formatValue(3.5, 2); // "3.50 متر"

// استخدام الوحدة في عملية حسابية
$lengthInKm = $meter->convertToBase(5000) / 1000; // 5 كم
```

---

## UnitFactory - مصنع الوحدات

**Namespace:** `Tss\Inventory\Classes\Units`

### مقدمة
كلاس مسؤول عن إنشاء كائنات `Unit` بطرق مختلفة: من رمز الوحدة، من النوع، أو من السياق. يضمن أن الكائنات التي ينشئها صالحة (تتحقق من صحة الرمز).

### الدوال الرئيسية لـ UnitFactory

#### `createFromString(string $unitString): Unit`
ينشئ كائن `Unit` من رمز الوحدة.

- **المعاملات:**
  - `$unitString` : رمز الوحدة (مثل `'kg'`, `'m'`, `'dozen'`).
- **الإرجاع:** كائن `Unit`.
- **الاستثناءات:** يرمي `InvalidArgumentException` إذا كان الرمز غير مدعوم.

**مثال:**
```php
$kg = UnitFactory::createFromString('kg');
echo $kg->label; // "كيلوجرام"
```

#### `createUnitsByType(string $type): array`
ينشئ مصفوفة من كائنات `Unit` لجميع الوحدات المنتمية لنوع معين.

- **المعاملات:**
  - `$type` : نوع الوحدة (أحد ثوابت `UnitType`).
- **الإرجاع:** مصفوفة من كائنات `Unit`.

**مثال:**
```php
$weightUnits = UnitFactory::createUnitsByType(UnitType::WEIGHT);
foreach ($weightUnits as $unit) {
    echo $unit->symbol . ' - ' . $unit->label . "\n";
}
```

#### `createAllUnits(): array`
ينشئ مصفوفة من كائنات `Unit` لجميع الوحدات المدعومة في النظام.

#### `createBaseUnits(): array`
ينشئ مصفوفة من كائنات `Unit` للوحدات الأساسية لكل نوع (مثل `m` للطول، `kg` للوزن). المصفوفة ترابطية `[type => Unit]`.

**مثال:**
```php
$baseUnits = UnitFactory::createBaseUnits();
$baseLength = $baseUnits[UnitType::LENGTH]; // وحدة المتر
```

#### `createUnitByContext(string $context): ?Unit`
ينشئ وحدة أساسية لسياق معين من سياقات الكميات.

- **المعاملات:**
  - `$context` : السياق (`'general'`, `'packaging'`, `'paper'`, `'pairs'`, `'large'`).
- **الإرجاع:** كائن `Unit` أو `null` إذا كان السياق غير معروف.

**السياقات المدعومة:**
- `'general'` ← `pcs` (حبة)
- `'packaging'` ← `dozen` (درزن)
- `'paper'` ← `ream` (ريم)
- `'pairs'` ← `pair` (زوج)
- `'large'` ← `hundred` (مئة)

**مثال:**
```php
$packagingUnit = UnitFactory::createUnitByContext('packaging');
echo $packagingUnit->symbol; // "dozen"
```

#### `createCompatibleUnits(string $unitSymbol): array`
ينشئ مصفوفة من كائنات `Unit` للوحدات المتوافقة مع وحدة معينة (من نفس النوع، باستثناء الوحدة نفسها).

- **المعاملات:**
  - `$unitSymbol` : رمز الوحدة.
- **الإرجاع:** مصفوفة من كائنات `Unit`.

**مثال:**
```php
$compatible = UnitFactory::createCompatibleUnits('m');
foreach ($compatible as $unit) {
    echo $unit->symbol . ' '; // cm, km, mm, ...
}
```

### أمثلة UnitFactory

```php
// 1. إنشاء وحدة من الرمز
try {
    $unit = UnitFactory::createFromString('xyz');
} catch (InvalidArgumentException $e) {
    echo "وحدة غير مدعومة";
}

// 2. إنشاء جميع وحدات الطول
$lengthUnits = UnitFactory::createUnitsByType(UnitType::LENGTH);
echo count($lengthUnits); // عدد وحدات الطول

// 3. إنشاء وحدة من السياق
$pairUnit = UnitFactory::createUnitByContext('pairs');
echo $pairUnit->label; // "زوج"

// 4. إنشاء الوحدات المتوافقة مع الكيلوجرام
$kg = UnitFactory::createFromString('kg');
$compatible = UnitFactory::createCompatibleUnits($kg->symbol);
```

---

## أفضل الممارسات

1. **استخدم `UnitFactory` لإنشاء كائنات الوحدة** بدلاً من إنشائها يدويًا باستخدام `new Unit(...)`، لأن المصنع يتحقق من صحة الرمز.
2. **عند الحاجة إلى تحويلات متعددة**، أنشئ كائن `Unit` مرة واحدة وأعد استخدامه بدلاً من إنشاء كائن جديد لكل عملية تحويل.
3. **للحصول على معلومات مفصلة عن وحدة**، استخدم `getInfo()` بدلاً من الاعتماد على الخصائص العامة فقط.
4. **قبل التحويل**، يمكنك استخدام `canConvertTo()` للتحقق من إمكانية التحويل دون رمي استثناء.
5. **استخدم `isCompatibleWith()`** عندما تريد التحقق من التوافق المنطقي (نفس النوع الرئيسي) قبل إجراء التحويل.

---

بهذا نكون قد غطينا جميع الجوانب الأساسية للكلاسات الثلاثة. لمزيد من التفاصيل، يُرجى الرجوع إلى الكود المصدري والتعليقات المضمنة.