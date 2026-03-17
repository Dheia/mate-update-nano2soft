# توثيق الكلاس `UnitHelper`

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 جدول المحتويات

1. [مقدمة](#مقدمة)
2. [الدوال الرئيسية](#الدوال-الرئيسية)
   - [safeConvert](#safeconvert)
   - [formatWithUnit](#formatwithunit)
   - [getBestDisplayUnit](#getbestdisplayunit)
   - [generateSelectOptions](#generateselectoptions)
   - [generateGroupedSelectOptions](#generategroupedselectoptions)
   - [getRentalTimeUnits](#getrentaltimeunits)
   - [getConversionFactor](#getconversionfactor)
   - [getConversionFactorFormatted](#getconversionfactorformatted)
   - [formatDecimal](#formatdecimal)
3. [أمثلة شاملة](#أمثلة-شاملة)

---

## مقدمة

كلاس `UnitHelper` هو كلاس مساعد (Helper) يوفر دوالاً عامة لتسهيل التعامل مع الوحدات في التطبيقات. يقدم هذا الكلاس دوالاً للتحويل الآمن، تنسيق القيم مع الوحدات، اختيار أفضل وحدة للعرض، توليد خيارات القوائم المنسدلة (Select) بشكل مبسط، والتعامل مع وحدات التأجير. يعتمد الكلاس على الكلاسات الأخرى (`UnitNameHelper`, `UnitConverter`, `UnitContextValidator`) لتقديم وظائف متكاملة.

---

## الدوال الرئيسية

### `safeConvert`

```php
public static function safeConvert(
    float $value,
    string $from,
    string $to,
    int $precision = 6,
    bool $validateLogic = true
): ?float
```

**الوصف:**  
تحويل قيمة من وحدة إلى أخرى مع معالجة الأخطاء وإمكانية التحقق من منطقية التحويل. لا يرمي استثناءات، بل يعيد `null` في حالة الفشل.

**المعاملات:**
- `$value` : القيمة المراد تحويلها.
- `$from` : رمز الوحدة المصدر.
- `$to` : رمز الوحدة الهدف.
- `$precision` : عدد المنازل العشرية (افتراضي 6).
- `$validateLogic` : إذا كان `true`، يتم التحقق من منطقية التحويل باستخدام `UnitContextValidator::isConversionLogical`. إذا كان التحويل غير منطقي، تُعاد `null`.

**الإرجاع:**  
القيمة المحولة (float) أو `null` في حالة حدوث خطأ (وحدة غير مدعومة، تحويل غير ممكن، أو تحويل غير منطقي).

**مثال:**
```php
$result = UnitHelper::safeConvert(5, 'kg', 'lb', 2, true);
if ($result !== null) {
    echo "النتيجة: $result";
} else {
    echo "لا يمكن إجراء التحويل";
}
```

---

### `formatWithUnit`

```php
public static function formatWithUnit(float $value, string $unit, int $decimals = 2, $lang = 'ar'): string
```

**الوصف:**  
تنسيق قيمة رقمية مع تسمية الوحدة المناسبة (حسب اللغة) للحصول على نص جاهز للعرض.

**المعاملات:**
- `$value` : القيمة الرقمية.
- `$unit` : رمز الوحدة.
- `$decimals` : عدد الخانات العشرية (افتراضي 2).
- `$lang` : اللغة (`'ar'` أو `'en'`).

**الإرجاع:**  
سلسلة نصية مثل `"100.00 كيلوجرام"` أو `"2.50 Meter"`.

**مثال:**
```php
echo UnitHelper::formatWithUnit(2.5, 'kg', 3, 'ar'); // "2.500 كيلوجرام"
echo UnitHelper::formatWithUnit(100, 'cm', 0, 'en'); // "100 Centimeter"
```

---

### `getBestDisplayUnit`

```php
public static function getBestDisplayUnit(float $value, string $currentUnit, array $preferredUnits = [], $lang = 'ar'): string
```

**الوصف:**  
البحث عن أفضل وحدة لعرض قيمة معينة، بحيث تكون القيمة المحولة بين 1 و 1000. مفيد لاختيار الوحدة المناسبة (مثلاً عرض 1500 متر كـ 1.5 كم).

**المعاملات:**
- `$value` : القيمة بوحدة `$currentUnit`.
- `$currentUnit` : رمز الوحدة الحالية.
- `$preferredUnits` : مصفوفة من رموز الوحدات المفضلة (اختياري). إذا أعطيت، يقتصر البحث عليها.
- `$lang` : اللغة (تستخدم فقط لجلب معلومات الوحدة، لكن النتيجة هي رمز الوحدة).

**الإرجاع:**  
رمز الوحدة الأفضل للعرض. إذا لم يُعثر على وحدة مناسبة، تُعاد `$currentUnit`.

**مثال:**
```php
$best = UnitHelper::getBestDisplayUnit(1500, 'm', ['km', 'm'], 'ar');
echo $best; // 'km'
```

---

### `generateSelectOptions`

```php
public static function generateSelectOptions(string $type, string $selected = '', bool $includeEmptyOption = true, $lang = 'ar'): string
```

**الوصف:**  
توليد كود HTML `<option>` لقائمة منسدلة تحتوي على وحدات من نوع معين.

**المعاملات:**
- `$type` : نوع الوحدة (أحد ثوابت `UnitType`).
- `$selected` : رمز الوحدة التي يجب أن تكون محددة مسبقاً (يُضاف لها `selected`).
- `$includeEmptyOption` : إضافة خيار فارغ (`<option value="">-- اختر وحدة --</option>`) في البداية.
- `$lang` : اللغة.

**الإرجاع:**  
نص HTML يحتوي على عناصر `<option>`.

**مثال:**
```php
echo '<select name="weight_unit">';
echo UnitHelper::generateSelectOptions(UnitType::WEIGHT, 'kg', true, 'ar');
echo '</select>';
```

---

### `generateGroupedSelectOptions`

```php
public static function generateGroupedSelectOptions(string $selected = '', bool $includeEmptyOption = true, $lang = 'ar'): string
```

**الوصف:**  
توليد خيارات `<select>` مصنفة حسب النوع باستخدام `<optgroup>`، مناسبة لعرض جميع الوحدات بطريقة منظمة.

**المعاملات:**
- `$selected` : رمز الوحدة المحددة.
- `$includeEmptyOption` : إضافة خيار فارغ.
- `$lang` : اللغة.

**الإرجاع:**  
نص HTML مع مجموعات `<optgroup>`.

**مثال:**
```php
echo '<select name="unit">';
echo UnitHelper::generateGroupedSelectOptions('kg', true, 'ar');
echo '</select>';
```

---

### `getRentalTimeUnits`

```php
public static function getRentalTimeUnits($lang = 'ar', $is_key = false): array
```

**الوصف:**  
إرجاع وحدات الزمن المستخدمة في سياق التأجير (ساعة، يوم، أسبوع، شهر). هذه الوحدات هي الأكثر شيوعاً في أنظمة التأجير.

**المعاملات:**
- `$lang` : اللغة.
- `$is_key` : إذا كان `true`، تُرجع فقط قائمة الرموز (`['h','day','week','month']`). إذا كان `false`، تُرجع مصفوفة ترابطية `[الرمز => التسمية]`.

**الإرجاع:**  
مصفوفة حسب `$is_key`.

**مثال:**
```php
$units = UnitHelper::getRentalTimeUnits('ar');
// ['h' => 'ساعة', 'day' => 'يوم', 'week' => 'أسبوع', 'month' => 'شهر']

$keys = UnitHelper::getRentalTimeUnits('ar', true);
// ['h', 'day', 'week', 'month']
```

---

### `getConversionFactor`

```php
public static function getConversionFactor(string $unit, $defaultValue = 0.0): float
```

**الوصف:**  
الحصول على عامل التحويل للوحدة (بالنسبة للوحدة الأساسية لنوعها). في حالة حدوث خطأ، تُعاد القيمة الافتراضية.

**المعاملات:**
- `$unit` : رمز الوحدة.
- `$defaultValue` : القيمة الافتراضية (افتراضي 0.0).

**الإرجاع:**  
عامل التحويل كـ float.

**مثال:**
```php
$factor = UnitHelper::getConversionFactor('km'); // 1000.0
$factor = UnitHelper::getConversionFactor('xyz', 1); // 1.0 (لأن الوحدة غير معروفة)
```

---

### `getConversionFactorFormatted`

```php
public static function getConversionFactorFormatted(string $unit, $defaultValue = 0.0, int $decimals = 10, $isFormatDecimal = true): string
```

**الوصف:**  
الحصول على عامل التحويل منسقاً كسلسلة نصية، مع إزالة الأصفار الزائدة (اختياري). مناسبة لعرضه في حقول الإدخال.

**المعاملات:**
- `$unit` : رمز الوحدة.
- `$defaultValue` : القيمة الافتراضية.
- `$decimals` : عدد الخانات العشرية (افتراضي 10).
- `$isFormatDecimal` : إذا كان `true`، يتم تطبيق `formatDecimal` على الناتج.

**الإرجاع:**  
سلسلة نصية تمثل عامل التحويل.

**مثال:**
```php
echo UnitHelper::getConversionFactorFormatted('kg'); // "1"
echo UnitHelper::getConversionFactorFormatted('lb', 0, 6); // "0.453592"
```

---

### `formatDecimal`

```php
public static function formatDecimal($value, int $decimals = 12): string
```

**الوصف:**  
تنسيق رقم عشري بحيث يتم إزالة الأصفار الزائدة من اليمين، وإزالة النقطة العشرية إذا لم تبقى خانات. مفيد لعرض الأرقام بشكل نظيف.

**المعاملات:**
- `$value` : الرقم (float أو string).
- `$decimals` : أقصى عدد من الخانات العشرية المسموح بها (افتراضي 12).

**الإرجاع:**  
سلسلة نصية بدون أصفار زائدة.

**مثال:**
```php
echo UnitHelper::formatDecimal(12.3000, 4); // "12.3"
echo UnitHelper::formatDecimal(5.0, 2);      // "5"
echo UnitHelper::formatDecimal(3.14159, 3);  // "3.142"
```

---

## أمثلة شاملة

### مثال 1: تحويل آمن وعرض النتيجة
```php
$value = 100;
$from = 'cm';
$to = 'm';

$result = UnitHelper::safeConvert($value, $from, $to, 2, true);
if ($result !== null) {
    echo UnitHelper::formatWithUnit($result, $to, 2, 'ar');
} else {
    echo "لا يمكن تحويل $value $from إلى $to";
}
```

### مثال 2: اختيار أفضل وحدة لعرض الطول
```php
$lengthInMeters = 3456; // متر
$bestUnit = UnitHelper::getBestDisplayUnit($lengthInMeters, 'm', ['km', 'm'], 'ar');
$converted = UnitConverter::convert($lengthInMeters, 'm', $bestUnit, 2);
echo "أفضل عرض: " . UnitHelper::formatWithUnit($converted, $bestUnit, 2, 'ar');
// الناتج مثلاً: "3.46 كم"
```

### مثال 3: توليد قائمة منسدلة لوحدات التأجير
```php
$rentalUnits = UnitHelper::getRentalTimeUnits('ar');
echo '<select name="rental_period">';
foreach ($rentalUnits as $unit => $label) {
    echo "<option value=\"$unit\">$label</option>";
}
echo '</select>';
```

### مثال 4: استخدام formatDecimal في حقول الإدخال
```php
$conversionFactor = UnitHelper::getConversionFactor('kg'); // 1.0
echo '<input type="text" value="' . UnitHelper::formatDecimal($conversionFactor, 5) . '">';
// <input type="text" value="1">
```

---

بهذا نكون قد غطينا جميع دوال الكلاس `UnitHelper` مع شرح وافٍ وأمثلة عملية. يساعد هذا الكلاس في تبسيط العمليات اليومية المتعلقة بالوحدات، خاصة في واجهات المستخدم والتحويلات الآمنة.