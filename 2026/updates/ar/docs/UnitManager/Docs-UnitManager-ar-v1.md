# توثيق نظام إدارة وتحويل الوحدات الشامل

## 📋 جدول المحتويات

1. [مقدمة عامة](#مقدمة-عامة)
2. [نظرة عامة على البنية](#نظرة-عامة-على-البنية)
3. [الكلاسات الرئيسية](#الكلاسات-الرئيسية)
   - [UnitType - تعريف أنواع الوحدات](#unittype---تعريف-أنواع-الوحدات)
   - [UnitNameHelper - مساعد أسماء الوحدات](#unitnamehelper---مساعد-أسماء-الوحدات)
   - [UnitConverter - محول الوحدات](#unitconverter---محول-الوحدات)
   - [UnitHelper - مساعد الوحدات](#unithelper---مساعد-الوحدات)
   - [UnitContextValidator - مدقق سياق الوحدات](#unitcontextvalidator---مدقق-سياق-الوحدات)
   - [SmartUnitConverter - محول ذكي](#smartunitconverter---محول-ذكي)
   - [UnitsInformation - معلومات الوحدات](#unitsinformation---معلومات-الوحدات)
   - [ConversionForm - نموذج التحويل](#conversionform---نموذج-التحويل)
   - [UnitFactory - مصنع الوحدات](#unitfactory---مصنع-الحدات)
   - [Unit - كائن الوحدة](#unit---كائن-الوحدة)
   - [UnitCreator - منشئ الوحدات في قاعدة البيانات](#unitcreator---منشئ-الوحدات-في-قاعدة-البيانات)
   - [UnitManager - مدير الوحدات الموحد](#unitmanager---مدير-الوحدات-الموحد)
4. [النماذج (Models) المحدثة](#النماذج-models-المحدثة)
5. [أمثلة متكاملة](#أمثلة-متكاملة)
6. [أفضل الممارسات ومعالجة الأخطاء](#أفضل-الممارسات-ومعالجة-الأخطاء)
7. [الخلاصة](#الخلاصة)

---

## مقدمة عامة

نظام إدارة وتحويل الوحدات الشامل هو حزمة PHP متكاملة ضمن `Tss.Inventory` تهدف إلى توفير حل كامل ومرن للتعامل مع جميع أنواع الوحدات القياسية والتجارية. يدعم النظام أكثر من 16 نوعًا رئيسيًا من الوحدات (الطول، الوزن، الحجم، المساحة، درجة الحرارة، الوقت، السرعة، الضغط، الطاقة، القدرة، البيانات، الزوايا، الكميات بمختلف سياقاتها) ويوفر أدوات للتحويل، البحث، التحقق، والربط مع المنتجات.

تم إعادة هيكلة النظام بالكامل ليشمل:
- **كلاسات مساعدة (Helpers)** للتعامل مع أسماء الوحدات وتحويلاتها.
- **مدقق سياق** لمنع التحويلات غير المنطقية بين وحدات الكميات المختلفة.
- **مصنع وحدات (Factory)** لإنشاء كائنات وحدة.
- **منشئ وحدات (Creator)** لإنشاء الوحدات في قاعدة البيانات وربطها بالمنتجات.
- **مدير موحد (Manager)** يقدم واجهة برمجية مبسطة لجميع العمليات.

الهدف هو تمكين المطورين من:
- تحويل القيم بين الوحدات بدقة عالية.
- الحصول على معلومات كاملة عن أي وحدة.
- إنشاء وإدارة الوحدات في قاعدة البيانات.
- ربط الوحدات بالمنتجات والأسعار.
- التحقق من منطقية التحويلات في التطبيقات العملية.

---

## نظرة عامة على البنية

```
Tss\Inventory\Classes\
│
├── UnitManager.php                      # مدير الوحدات الموحد (واجهة برمجية رئيسية)
│
└── Units\                                # كلاسات نظام التحويل
    ├── UnitType.php                      # تعريف أنواع الوحدات
    ├── UnitNameHelper.php                 # أسماء وتصنيفات الوحدات
    ├── UnitConverter.php                  # محول الوحدات الأساسي
    ├── UnitHelper.php                      # دوال مساعدة عامة
    ├── UnitContextValidator.php            # مدقق سياق الكميات
    ├── SmartUnitConverter.php              # محول ذكي مع تحذيرات
    ├── UnitsInformation.php                # عرض معلومات الوحدات (HTML)
    ├── ConversionForm.php                  # نماذج تحويل (للاستخدام التجريبي)
    ├── UnitFactory.php                      # مصنع كائنات الوحدة
    ├── Unit.php                             # كائن الوحدة
    └── UnitCreator.php                      # إنشاء الوحدات في قاعدة البيانات
```

---

## الكلاسات الرئيسية

### UnitType - تعريف أنواع الوحدات

كلاس ثابت يحتوي على ثوابت لتعريف أنواع الوحدات المختلفة، مع دوال مساعدة للتحقق والترجمة.

#### الثوابت الأساسية

```php
UnitType::LENGTH;          // 'length'   - الطول
UnitType::WEIGHT;          // 'weight'   - الوزن
UnitType::VOLUME;          // 'volume'   - الحجم
UnitType::AREA;            // 'area'     - المساحة
UnitType::TEMPERATURE;     // 'temperature' - درجة الحرارة
UnitType::TIME;            // 'time'     - الزمن
UnitType::SPEED;           // 'speed'    - السرعة
UnitType::PRESSURE;        // 'pressure' - الضغط
UnitType::ENERGY;          // 'energy'   - الطاقة
UnitType::POWER;           // 'power'    - القدرة
UnitType::DATA;            // 'data'     - البيانات
UnitType::ANGLE;           // 'angle'    - الزوايا
UnitType::QUANTITY;        // 'quantity' - الكميات (النوع الرئيسي)

// الأنواع الفرعية للكميات
UnitType::QUANTITY_GENERAL;    // 'quantity_general'   - كميات عامة
UnitType::QUANTITY_PACKAGING;  // 'quantity_packaging' - كميات التعبئة
UnitType::QUANTITY_PAPER;      // 'quantity_paper'     - كميات الورق
UnitType::QUANTITY_PAIRS;      // 'quantity_pairs'     - كميات الأزواج
UnitType::QUANTITY_LARGE;      // 'quantity_large'     - كميات كبيرة
UnitType::CUSTOM;              // 'custom'             - وحدات مخصصة
```

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `getMainTypesWithLabels($lang)` | إرجاع الأنواع الرئيسية مع تسمياتها (`['length'=>'الطول', ...]`) |
| `getAllTypesWithLabels($lang)` | إرجاع جميع الأنواع (بما في ذلك الفرعية) مع تسمياتها |
| `getQuantitySubtypesWithLabels($lang)` | إرجاع الأنواع الفرعية للكميات فقط |
| `getTransLabel($type, $lang)` | الحصول على التسمية المترجمة لنوع معين |
| `isValid($type)` | التحقق مما إذا كان النوع معرفًا |
| `isQuantitySubtype($type)` | التحقق مما إذا كان النوع فرعيًا للكميات |
| `isCustomType($type)` | التحقق مما إذا كان النوع مخصصًا |
| `getMainType($type)` | إرجاع النوع الرئيسي (للأنواع الفرعية) |
| `getAllTypes()` | إرجاع قائمة بجميع الأنواع |
| `getMainTypes()` | إرجاع قائمة بالأنواع الرئيسية فقط |
| `getSelectOptions($selected, $includeEmpty, $onlyMainTypes)` | إنشاء خيارات HTML `<select>` للأنواع |

#### أمثلة

```php
// الحصول على تسمية نوع
$label = UnitType::getTransLabel('length', 'ar'); // 'الطول'

// التحقق من النوع
if (UnitType::isValid('weight')) { ... }

// التحقق من النوع الفرعي
if (UnitType::isQuantitySubtype('quantity_packaging')) { ... }

// إنشاء خيارات select
echo UnitType::getSelectOptions('length', true, false);
```

---

### UnitNameHelper - مساعد أسماء الوحدات

يوفر هذا الكلاس وصولاً مركزياً إلى معلومات الوحدات: أسمائها، رموزها، أنواعها، سياقاتها، والبحث فيها.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `getAllUnitsWithLabels($lang)` | إرجاع جميع الوحدات مع تسمياتها (`['mm'=>'مليمتر', ...]`) |
| `getUnitsByTypeWithLabels($type, $lang, $isBaseUnit)` | إرجاع وحدات نوع معين، مع إمكانية تصفية الأساسية فقط |
| `getUnitLabel($unit, $lang)` | الحصول على تسمية وحدة محددة |
| `getEnglishLabel($unit)` | الحصول على التسمية الإنجليزية للوحدة |
| `getCategorizedUnits($lang)` | إرجاع الوحدات مصنفة حسب النوع (لهيكلة القوائم) |
| `getSelectOptions($type, $selected, $includeEmpty, $lang)` | إنشاء خيارات HTML `<select>` لوحدات نوع معين |
| `getGroupedSelectOptions($selected, $includeEmpty, $lang)` | إنشاء خيارات `<select>` مصنفة حسب النوع |
| `searchUnits($searchText, $withLabels, $lang)` | بحث بسيط عن وحدات تحتوي على النص |
| `searchUnitAdvanced($searchText, $type, $lang)` | بحث متقدم يُرجع معلومات أول وحدة مطابقة (حسب الرمز، الاسم العربي، الإنجليزي) |
| `getUnitType($unit)` | إرجاع نوع الوحدة |
| `isValidUnit($unit)` | التحقق من صحة رمز الوحدة |
| `getSimilarUnits($unit, $lang)` | إرجاع وحدات من نفس النوع |
| `getBaseUnits($lang)` | إرجاع الوحدات الأساسية لكل نوع (`['length'=>['unit'=>'m','label'=>'متر']]`) |
| `getUnitInfo($unit, $lang)` | إرجاع معلومات أساسية عن الوحدة (رمز، تسمية، نوع، ...) |
| `getUnitFullInfo($unit, $lang)` | إرجاع معلومات كاملة عن الوحدة (بما في ذلك `unit_id`, `unit_name_ar`, `unit_name_en`, `type_id`, `type_name_ar`, `type_name_en`, `is_base_unit`, `base_unit`, `context`, `is_metric`, `is_imperial`, `display_name`, `full_name`) |
| `isBaseUnitForItsType($unit)` | التحقق مما إذا كانت الوحدة أساسية لنوعها |
| `getBaseUnitByType($type)` | إرجاع رمز الوحدة الأساسية لنوع معين |

#### أمثلة

```php
// الحصول على تسمية وحدة
echo UnitNameHelper::getUnitLabel('kg', 'ar'); // "كيلوجرام"

// بحث متقدم عن وحدة
$info = UnitNameHelper::searchUnitAdvanced('كيلو');
if ($info) {
    echo $info['unit_name_ar']; // "كيلوجرام"
}

// الحصول على معلومات كاملة
$full = UnitNameHelper::getUnitFullInfo('dozen', 'ar');
/*
[
    'unit_id' => 'dozen',
    'unit_name_ar' => 'درزن',
    'unit_name_en' => 'Dozen',
    'type_id' => 'quantity_packaging',
    'type_name_ar' => 'كميات التعبئة',
    'is_base_unit' => true,
    'base_unit' => 'dozen',
    'context' => 'packaging',
    ...
]
*/

// إنشاء خيارات select مصنفة
echo UnitNameHelper::getGroupedSelectOptions('kg', true, 'ar');
```

---

### UnitConverter - محول الوحدات

قلب نظام التحويل، يحتوي على بيانات التحويل لجميع الوحدات ويقوم بإجراء التحويلات الرياضية.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `convert($value, $from, $to, $precision, $strictContext)` | تحويل قيمة من وحدة إلى أخرى. `$strictContext` يفعّل التحقق الصارم من السياق لوحدات الكميات. |
| `convertBatch(array $values, $from, $to, $precision)` | تحويل مجموعة من القيم دفعة واحدة. |
| `describeConversion($value, $from, $to, $precision)` | إرجاع وصف نصي للتحويل (مثال: "1 متر = 100 سنتيمتر"). |
| `isUnitSupported($unit)` | التحقق مما إذا كانت الوحدة مدعومة. |
| `getUnitType($unit)` | إرجاع نوع الوحدة. |
| `getArabicLabel($unit)` | إرجاع التسمية العربية (اختصار لـ `UnitNameHelper::getUnitLabel`). |
| `getUnitsByType($type, $withLabels, $lang)` | إرجاع وحدات نوع معين (مع أو بدون تسميات). |
| `getAllUnitsGrouped($groupByMainType, $lang)` | إرجاع جميع الوحدات مصنفة حسب النوع الرئيسي أو الكامل. |
| `getUnitInfo($unit, $lang)` | إرجاع معلومات أساسية عن الوحدة (بما في ذلك عامل التحويل). |
| `canConvert($from, $to)` | التحقق من إمكانية التحويل بين وحدتين (بدون استثناء). |
| `getSupportedUnits($type, $includeSubtypes)` | إرجاع قائمة بجميع رموز الوحدات المدعومة، أو لنوع معين مع تضمين الأنواع الفرعية. |
| `getSupportedUnitsWithLabels($type, $includeSubtypes, $lang)` | مثل السابقة لكن مع التسميات. |
| `getSupportedConversionTypes($includeSubtypes)` | إرجاع قائمة بأنواع التحويل المدعومة. |
| `getSupportedConversionTypesWithLabels($includeSubtypes, $lang)` | مثل السابقة مع التسميات. |
| `isTypeSupported($type)` | التحقق مما إذا كان نوع الوحدة مدعومًا. |
| `getConversionFactor($unit, $type)` | إرجاع عامل التحويل للوحدة نسبة إلى الوحدة الأساسية للنوع. |

#### أمثلة

```php
// تحويل بسيط
$result = UnitConverter::convert(5, 'kg', 'lb', 4); // 11.0231

// تحويل درجة حرارة
$result = UnitConverter::convert(100, 'c', 'f', 2); // 212.00

// تحويل وحدات كميات
$result = UnitConverter::convert(1, 'dozen', 'pcs'); // 12

// التحقق من إمكانية التحويل
if (UnitConverter::canConvert('dozen', 'box')) { ... }

// الحصول على جميع وحدات الوزن مع تسميات
$weightUnits = UnitConverter::getSupportedUnitsWithLabels('weight', false, 'ar');
```

---

### UnitHelper - مساعد الوحدات

دوال مساعدة عامة لتسهيل التعامل مع الوحدات في التطبيقات.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `safeConvert($value, $from, $to, $precision, $validateLogic)` | تحويل آمن مع معالجة الأخطاء وإمكانية التحقق من المنطقية (يرجع `null` في حالة الفشل). |
| `formatWithUnit($value, $unit, $decimals, $lang)` | تنسيق القيمة مع تسمية الوحدة (مثال: "100.00 كيلوجرام"). |
| `getBestDisplayUnit($value, $currentUnit, $preferredUnits, $lang)` | البحث عن أفضل وحدة لعرض قيمة (بحيث تكون القيمة بين 1 و 1000). |
| `generateSelectOptions($type, $selected, $includeEmpty, $lang)` | توليد خيارات `<select>` لوحدات نوع معين. |
| `generateGroupedSelectOptions($selected, $includeEmpty, $lang)` | توليد خيارات `<select>` مصنفة حسب النوع. |
| `getRentalTimeUnits($lang, $is_key)` | إرجاع وحدات الزمن المستخدمة في التأجير (ساعة، يوم، أسبوع، شهر). |
| `getConversionFactor($unit, $defaultValue)` | إرجاع عامل التحويل للوحدة (مع قيمة افتراضية في حالة الخطأ). |
| `getConversionFactorFormatted($unit, $defaultValue, $decimals, $isFormatDecimal)` | إرجاع عامل التحويل منسقًا (بدون أصفار زائدة). |
| `formatDecimal($value, $decimals)` | تنسيق رقم عشري بدون أصفار زائدة. |

#### أمثلة

```php
// تحويل آمن
$result = UnitHelper::safeConvert(1, 'dozen', 'box', 2, true); // 0.50

// تنسيق قيمة
echo UnitHelper::formatWithUnit(2.5, 'kg', 3, 'ar'); // "2.500 كيلوجرام"

// الحصول على أفضل وحدة للعرض
$best = UnitHelper::getBestDisplayUnit(1500, 'm', ['km','m'], 'ar'); // 'km'

// الحصول على وحدات التأجير
$rentalUnits = UnitHelper::getRentalTimeUnits('ar'); // ['h'=>'ساعة', 'day'=>'يوم', ...]

// تنسيق رقم عشري
echo UnitHelper::formatDecimal(12.3000, 4); // "12.3"
```

---

### UnitContextValidator - مدقق سياق الوحدات

مسؤول عن التحقق من منطقية تحويل وحدات الكميات بناءً على سياقها (تعبئة، ورق، أزواج، كميات كبيرة).

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `isConversionLogical($from, $to)` | التحقق مما إذا كان التحويل بين وحدتين منطقياً (ضمن نفس السياق أو عبر `pcs`). |
| `getConversionReason($from, $to)` | إرجاع سبب المنطقية أو عدمها (نص). |
| `getCompatibleUnits($unit)` | إرجاع قائمة بالوحدات المتوافقة منطقياً مع وحدة معينة (كل منها مصفوفة تحتوي على `unit` و `label`). |

#### القواعد

- `pcs` (الحبة) هي وحدة عالمية يمكن تحويلها من وإلى أي وحدة كمية.
- القواعد المحددة في `CONVERSION_RULES` تحدد العلاقات المنطقية بين الوحدات (مثال: `dozen` ↔ `box`).
- إذا لم توجد قاعدة، يتم السماح إذا كان السياق متطابقاً.

#### أمثلة

```php
// التحقق من المنطقية
if (UnitContextValidator::isConversionLogical('dozen', 'box')) {
    // مسموح (كلاهما تعبئة)
}

if (!UnitContextValidator::isConversionLogical('dozen', 'ream')) {
    // غير منطقي (تعبئة إلى ورق)
}

// الحصول على الوحدات المتوافقة
$compatible = UnitContextValidator::getCompatibleUnits('dozen');
```

---

### SmartUnitConverter - محول ذكي

يوفر تحويلات مع تحذيرات وتحقق صارم من المنطقية.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `safeConvert($value, $from, $to, $precision)` | تحويل آمن مع التحقق من المنطقية؛ يرمي استثناء إذا كان التحويل غير منطقي. |
| `convertWithWarning($value, $from, $to, $precision)` | تحويل مع إرجاع مصفوفة تحتوي على القيمة، العلم `is_logical`، وتحذير إذا لم يكن منطقياً. |

#### أمثلة

```php
try {
    $result = SmartUnitConverter::safeConvert(1, 'dozen', 'box', 2);
    // النتيجة 0.50
} catch (InvalidArgumentException $e) {
    echo $e->getMessage();
}

// تحويل مع تحذير
$result = SmartUnitConverter::convertWithWarning(1, 'dozen', 'ream', 2);
if (!$result['is_logical']) {
    echo $result['warning']; // تحذير: التحويل غير منطقي
}
```

---

### UnitsInformation - معلومات الوحدات

كلاس لعرض معلومات الوحدات بتنسيق HTML منسق (لأغراض التوثيق والعرض التجريبي).

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `displayAllUnitsInfo()` | عرض جميع الوحدات في صفحة HTML كاملة مع إحصائيات وبطاقات (باستخدام Bootstrap). |
| `displayTypeInfo($typeKey)` | عرض وحدات نوع معين فقط. |
| `displayAllUnitsInfoBootstrap()` | نفس `displayAllUnitsInfo` (اسم بديل). |
| `displayTypeInfoBootstrap($typeKey)` | نفس `displayTypeInfo`. |

#### مثال

```php
echo UnitsInformation::displayAllUnitsInfo();
```

---

### ConversionForm - نموذج التحويل

ينشئ نماذج HTML تفاعلية لاختبار تحويل الوحدات.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `renderAdvancedForm()` | نموذج متقدم مع خيارات (نوع التحويل، وحدات، قيمة، وضع صارم، دقة). |
| `renderForm()` | نموذج بسيط. |
| `testQuantityConversions()` | دالة اختبار لتجربة تحويلات الكميات (تطبع النتائج في سطر الأوامر). |

#### مثال

```php
$form = new ConversionForm();
echo $form->renderAdvancedForm();
```

---

### UnitFactory - مصنع الوحدات

ينشئ كائنات `Unit` من رموز الوحدات أو الأنواع.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `createFromString($unitString)` | إنشاء كائن `Unit` من رمز الوحدة (مثل 'm'). |
| `createUnitsByType($type)` | إنشاء مصفوفة من كائنات `Unit` لنوع معين. |
| `createAllUnits()` | إنشاء مصفوفة من كائنات `Unit` لجميع الوحدات. |
| `createBaseUnits()` | إنشاء مصفوفة من كائنات `Unit` للوحدات الأساسية لكل نوع (مرتبطة بالنوع). |
| `createUnitByContext($context)` | إنشاء وحدة أساسية لسياق معين (مثلاً `packaging` → `dozen`). |
| `createCompatibleUnits($unitSymbol)` | إنشاء مصفوفة من كائنات `Unit` المتوافقة مع وحدة معينة. |

#### أمثلة

```php
$meter = UnitFactory::createFromString('m');
echo $meter->convertTo(100, 'cm'); // 10000

$lengthUnits = UnitFactory::createUnitsByType('length');
foreach ($lengthUnits as $unit) {
    echo $unit->symbol . ' - ' . $unit->label . "\n";
}
```

---

### Unit - كائن الوحدة

يمثل وحدة قياس واحدة بخصائصها وعملياتها.

#### الخصائص

| الخاصية | الوصف |
|---------|-------|
| `symbol` | رمز الوحدة (مثل 'kg') |
| `type` | نوع الوحدة (مثل 'weight') |
| `label` | التسمية العربية (مثل 'كيلوجرام') |
| `context` | السياق (للوحدات الكمية) |

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `convertTo($value, $toUnit, $precision)` | تحويل قيمة إلى وحدة أخرى. |
| `convertToBase($value, $precision)` | تحويل إلى الوحدة الأساسية للنوع. |
| `isCompatibleWith($otherUnit)` | التحقق من التوافق مع وحدة أخرى (نفس النوع الرئيسي). |
| `getConversionFactor()` | إرجاع عامل التحويل للوحدة. |
| `isBaseUnit()` | التحقق مما إذا كانت الوحدة أساسية. |
| `getEnglishLabel()` | إرجاع التسمية الإنجليزية. |
| `getInfo()` | إرجاع معلومات كاملة عن الوحدة (باستخدام `UnitNameHelper`). |
| `getContext()` | إرجاع السياق. |
| `getCompatibleUnits()` | إرجاع وحدات متوافقة (من نفس النوع). |
| `canConvertTo($toUnit)` | التحقق من إمكانية التحويل إلى وحدة أخرى. |
| `formatValue($value, $decimals)` | تنسيق قيمة مع الوحدة (مثال: "2.50 كيلوجرام"). |
| `__toString()` | تمثيل نصي للوحدة (مثال: "kg (كيلوجرام)"). |
| `equals(Unit $other)` | مقارنة مع وحدة أخرى. |
| `copy()` | إنشاء نسخة جديدة. |

#### أمثلة

```php
$kg = UnitFactory::createFromString('kg');
echo $kg->convertTo(5, 'g'); // 5000
echo $kg->formatValue(2.5, 2); // "2.50 كيلوجرام"
if ($kg->canConvertTo('lb')) { ... }
```

---

### UnitCreator - منشئ الوحدات في قاعدة البيانات

كلاس متخصص لإنشاء وتحديث الوحدات وربطها بالمنتجات والأسعار في قاعدة البيانات.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `createUnit(array $options, $is_test_create)` | إنشاء وحدة جديدة في قاعدة البيانات. |
| `createUnitFromSymbol(array $options, $is_test_create)` | إنشاء وحدة من رمز معروف (مثل 'kg') مع إكمال البيانات تلقائياً. |
| `createStandardUnitsForType(array $options, $is_test_create)` | إنشاء جميع الوحدات القياسية لنوع معين (مثل كل وحدات الطول). |
| `createAllStandardUnits(array $options, $is_test_create)` | إنشاء جميع الوحدات القياسية لكل الأنواع. |
| `attachUnitToProduct(array $options, $is_test_create)` | ربط وحدة بمنتج (إنشاء سجل في `products_units`). |
| `createProductUnitPrice(array $options, $is_test_create)` | إنشاء سعر لوحدة منتج (في `products_prices_units`). |
| `createProductUnitsStructure(array $options, $is_test_create)` | إنشاء هيكل كامل من الوحدات والأسعار لمنتج معين. |
| `createUnitConversion(array $options, $is_test_create)` | إنشاء علاقة تحويل بين وحدتين (للاستخدام المستقبلي). |
| `importUnitsFromSymbols(array $options, $is_test_create)` | استيراد وحدات من قائمة رموز. |
| `createStandardQuantityUnits(array $options, $is_test_create)` | إنشاء وحدات الكميات القياسية (درزن، ريم، زوج، ...). |
| `firstOrCreateFromSymbol(array $options, $is_test_create)` | البحث عن وحدة برمز معين، وإنشاؤها إذا لم توجد. |

#### هيكل `$options` الشائع

- `companys_id`, `departments_id`: لتحديد الشركة والقسم.
- `name`: اسم الوحدة.
- `unit_symbol`: رمز الوحدة.
- `type`: نوع الوحدة (من `UnitType`).
- `conversion_factor`, `value`: عامل التحويل (عادة نفس القيمة).
- `is_base_unit`: هل هي الوحدة الأساسية؟
- `is_default`: هل هي الافتراضية؟
- `is_active`: حالة التفعيل.

#### أمثلة

```php
// إنشاء وحدة جديدة
$result = UnitCreator::createUnit([
    'name' => 'درزن',
    'unit_symbol' => 'dozen',
    'type' => UnitType::QUANTITY_PACKAGING,
    'is_base_unit' => true,
    'conversion_factor' => 12,
]);

if ($result['status']) {
    $unit = $result['model'];
}

// ربط وحدة بمنتج
$result = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => 5,
    'is_main' => true,
    'conversion_factor' => 1,
]);

// إنشاء سعر لوحدة منتج
$result = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => 5,
    'prices_id' => 1,
    'value' => 100.50,
    'currencys_id' => 1,
]);
```

---

### UnitManager - مدير الوحدات الموحد

واجهة برمجية موحدة تجمع معظم وظائف النظام وتوفر دوال ثابتة سهلة الاستخدام.

#### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `createUnit($options, $is_test_create)` | إنشاء وحدة جديدة (يعتمد على `UnitCreator::createUnit`). |
| `checkUnit($options)` | التحقق من صحة وحدة (رمزها، رصيدها، مستخدمها) - يستخدم للتحقق من كروت الشحن مثلاً. |
| `getQueryDate($records, $options)` | تطبيق شروط تاريخية على استعلام (دالة مساعدة). |
| `scopeWhereField($query, $field, $value, $is_or, $table, $is_not, $is_force)` | نطاق مساعد لتطبيق شروط على الحقول مع دعم `*` و `all`. |
| `printReports($options)` | طباعة تقارير الوحدات (باستخدام نظام التقارير). |
| `getAllUnitsGrouped($groupByMainType, $lang)` | الحصول على جميع الوحدات مصنفة (تمرير إلى `UnitConverter::getAllUnitsGrouped`). |
| `getUnitsForType($type, $includeLabels, $lang)` | الحصول على وحدات نوع معين (تمرير إلى `UnitConverter::getUnitsForType`). |
| `getRentalTimeUnits($lang, $is_key)` | الحصول على وحدات الزمن المستخدمة في التأجير. |
| `formatDecimal($value, $decimals)` | تنسيق رقم عشري بدون أصفار زائدة. |
| `generateDigital($length)` | توليد رقم عشوائي بطول معين (لأكواد الكروت). |
| `getGuid()` | توليد UUID عشوائي. |
| `getVoucherCode()` | توليد رمز قسيمة مكون من 13 رقمًا. |
| `humanUuid($options)` | توليد UUID صديق للإنسان (بصيغة `YYYYMMDD-HHMM-SSMM-MMMM...`). |
| `nanoUid($options)` | توليد معرف فريد بالنانوثانية. |

#### أمثلة

```php
use Tss\Inventory\Classes\UnitManager;

// إنشاء وحدة
$result = UnitManager::createUnit([
    'name' => 'كيلوجرام',
    'unit_symbol' => 'kg',
    'type' => UnitType::WEIGHT,
    'is_base_unit' => true,
]);

// التحقق من كرت شحن
$check = UnitManager::checkUnit(['unit_symbol' => 'CARD123']);

// الحصول على وحدات التأجير
$rentalUnits = UnitManager::getRentalTimeUnits('ar');

// طباعة تقرير
UnitManager::printReports($options);

// تنسيق رقم
echo UnitManager::formatDecimal(12.3000, 4); // "12.3"
```

---

## النماذج (Models) المحدثة

### `Tss\Inventory\Models\Unit`

تمت إضافة أعمدة جديدة لدعم النظام:

- `unit_symbol` (string, index)
- `conversion_factor` (decimal, 20,10)
- `value` (decimal, 20,10)
- `is_base_unit` (boolean)
- `is_public` (boolean)
- `is_published` (boolean)
- `published_at` (timestamp)
- `unpublished_at` (timestamp)

كما أضيفت Traits لتوفير وظائف متقدمة:
- `HasBaseUnit`: التعامل مع الوحدة الأساسية للنوع.
- `HasDefault`: جعل وحدة افتراضية.
- `HasRecordsOptions`: دالة `getRecords` لاسترجاع السجلات بمرونة.
- `ListObjects`: تخزين مؤقت للكائنات.
- `ListOptions`: قوائم الخيارات (للواجهات).
- `FieldsOptions`: خيارات الحقول (للواجهات).
- `HasScopesModel`: نطاقات متقدمة للاستعلام.

### `Tss\Inventory\Models\ProductsUnit`

- إضافة عمود `unit_symbol` لتسريع الاستعلامات.
- إضافة عمود `type` للتصنيف.
- تحسين التحقق من التكرار.

### `Tss\Inventory\Models\ProductsPricesUnit`

- إضافة عمود `unit_symbol`.
- تحسين دوال `makePrimary` و `makeIsActive` لتحديث السجلات بشكل صحيح.

---

## أمثلة متكاملة

### مثال 1: إنشاء وحدة جديدة وربطها بمنتج

```php
use Tss\Inventory\Classes\UnitManager;
use Tss\Inventory\Classes\Units\UnitCreator;

// 1. إنشاء وحدة
$unitResult = UnitManager::createUnit([
    'name' => 'درزن',
    'unit_symbol' => 'dozen',
    'type' => UnitType::QUANTITY_PACKAGING,
    'is_base_unit' => true,
    'conversion_factor' => 12,
]);

if (!$unitResult['status']) {
    throw new Exception($unitResult['error']);
}

$unit = $unitResult['model'];

// 2. ربط الوحدة بمنتج (products_id = 123)
$attachResult = UnitCreator::attachUnitToProduct([
    'products_id' => 123,
    'units_id' => $unit->id,
    'is_main' => false,
    'conversion_factor' => 12,
]);

if (!$attachResult['status']) {
    throw new Exception($attachResult['error']);
}

// 3. إنشاء سعر لهذه الوحدة
$priceResult = UnitCreator::createProductUnitPrice([
    'products_id' => 123,
    'units_id' => $unit->id,
    'prices_id' => 1,
    'value' => 50.00,
    'currencys_id' => 1,
    'is_default' => false,
]);

echo "تم إنشاء الوحدة وربطها بالمنتج بنجاح.";
```

### مثال 2: استخدام نظام التحويل في عملية بيع

```php
use Tss\Inventory\Classes\Units\UnitConverter;
use Tss\Inventory\Classes\Units\UnitHelper;

// المنتج يُباع بالدرزن، لكن العميل يريد 50 حبة
$quantityInDozen = UnitConverter::convert(50, 'pcs', 'dozen', 2); // 4.17 درزن

// عرض السعر للعميل بالوحدة المطلوبة
$pricePerPiece = 5; // سعر الحبة
$totalPrice = $quantityInDozen * $pricePerPiece * 12; // أو بطريقة أخرى

// تنسيق العرض
echo UnitHelper::formatWithUnit($quantityInDozen, 'dozen', 2, 'ar'); // "4.17 درزن"
```

### مثال 3: البحث عن وحدة وعرض معلوماتها

```php
use Tss\Inventory\Classes\Units\UnitNameHelper;

$searchTerm = 'كيلو';
$info = UnitNameHelper::searchUnitAdvanced($searchTerm, null, 'ar');

if ($info) {
    echo "الوحدة: {$info['unit_name_ar']} ({$info['unit_id']})\n";
    echo "النوع: {$info['type_name_ar']}\n";
    echo "هل هي أساسية؟ " . ($info['is_base_unit'] ? 'نعم' : 'لا') . "\n";
    if ($info['context']) {
        echo "السياق: {$info['context']}\n";
    }
}
```

### مثال 4: إنشاء جميع الوحدات القياسية لنوع الوزن

```php
use Tss\Inventory\Classes\Units\UnitCreator;

$result = UnitCreator::createStandardUnitsForType([
    'type' => UnitType::WEIGHT,
    'companys_id' => 1,
    'departments_id' => 2,
]);

if ($result['status']) {
    echo "تم إنشاء " . count($result['data']) . " وحدة بنجاح.\n";
    foreach ($result['data'] as $unit) {
        echo "- {$unit->name} ({$unit->unit_symbol})\n";
    }
}
```

---

## أفضل الممارسات ومعالجة الأخطاء

1. **استخدم `SmartUnitConverter::safeConvert`** عندما تحتاج إلى ضمان منطقية التحويل في سياقات الأعمال (مثل تحويل درزن إلى ريم غير منطقي).
2. **استخدم `UnitNameHelper::isValidUnit`** قبل استخدام أي رمز وحدة للتأكد من أنه مدعوم.
3. **عند إنشاء الوحدات في قاعدة البيانات**، تأكد من تمرير `companys_id` و `departments_id` المناسبين.
4. **للحصول على معلومات كاملة عن وحدة**، استخدم `UnitNameHelper::getUnitFullInfo` بدلاً من `getUnitInfo` إذا كنت بحاجة إلى بيانات مفصلة.
5. **عرض الوحدات في واجهات المستخدم**، استخدم `UnitHelper::generateGroupedSelectOptions` للحصول على قائمة منسدلة مصنفة.
6. **التعامل مع الأخطاء**:
   - معظم دوال `UnitCreator` تعيد مصفوفة تحتوي على `status` و `error` و `message`.
   - دوال التحويل قد ترمي استثناءات من نوع `InvalidArgumentException`، لذا استخدم `try-catch`.

```php
try {
    $result = SmartUnitConverter::safeConvert($value, $from, $to);
} catch (InvalidArgumentException $e) {
    // تحويل غير منطقي أو وحدة غير مدعومة
    log_error($e->getMessage());
    return back()->withError($e->getMessage());
}
```

---

## الخلاصة

نظام إدارة وتحويل الوحدات الشامل يوفر حلاً متكاملاً ومرناً لجميع احتياجات التعامل مع الوحدات في تطبيقات TSS Inventory. بفضل التصميم النمطي والكلاسات المتخصصة، يمكن للمطورين:

- تحويل القيم بدقة بين أي وحدتين مدعومتين.
- الحصول على معلومات تفصيلية عن الوحدات.
- إنشاء وإدارة الوحدات في قاعدة البيانات بسهولة.
- ربط الوحدات بالمنتجات والأسعار.
- التحقق من منطقية التحويلات في سياقات الأعمال.
- تقديم واجهات مستخدم غنية (قوائم منسدلة، معلومات منسقة، نماذج تحويل).

النظام قابل للتوسع بسهولة: يمكن إضافة وحدات جديدة بتحديث `UnitConverter::CONVERSION_DATA`، وإضافة قواعد منطقية جديدة في `UnitContextValidator::CONVERSION_RULES`، ودعم لغات جديدة عبر ملفات الترجمة.

---

بهذا نكون قد غطينا جميع جوانب النظام مع أمثلة عملية لكل جزء. لمزيد من التفاصيل، يُرجى الرجوع إلى كود المصدر والتعليقات المضمنة.