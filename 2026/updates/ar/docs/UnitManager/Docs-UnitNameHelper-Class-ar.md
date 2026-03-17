# توثيق الكلاس `UnitNameHelper`

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 جدول المحتويات

1. [مقدمة](#مقدمة)
2. [الثوابت](#الثوابت)
3. [الدوال الرئيسية](#الدوال-الرئيسية)
   - [getAllUnitsWithLabels](#getallunitswithlabels)
   - [getAllUnitsFilterWithLabels](#getallunitsfilterwithlabels)
   - [getUnitsByTypeWithLabels](#getunitsbytypewithlabels)
   - [getUnitLabel](#getunitlabel)
   - [getEnglishLabel](#getenglishlabel)
   - [getCategorizedUnits](#getcategorizedunits)
   - [getSelectOptions](#getselectoptions)
   - [getGroupedSelectOptions](#getgroupedselectoptions)
   - [searchUnits](#searchunits)
   - [searchUnitAdvanced](#searchunitadvanced)
   - [getUnitType](#getunittype)
   - [isValidUnit](#isvalidunit)
   - [getSimilarUnits](#getsimilarunits)
   - [getUnitsByContext](#getunitsbycontext)
   - [getBaseUnits](#getbaseunits)
   - [getBaseUnitByType](#getbaseunitbytype)
   - [getBaseUnitInfoByType](#getbaseunitinfobytype)
   - [getBaseUnitWithLabel](#getbaseunitwithlabel)
   - [getAllBaseUnitsForMainTypes](#getallbaseunitsformaintypes)
   - [getBaseUnitsForQuantitySubtypes](#getbaseunitsforquantitysubtypes)
   - [isBaseUnitForItsType](#isbaseunitforitstype)
   - [filterBaseUnits](#filterbaseunits)
   - [convertToBaseUnit](#converttobaseunit)
   - [getBaseUnitsList](#getbaseunitslist)
   - [getUnitInfo](#getunitinfo)
   - [getUnitFullInfo](#getunitfullinfo)
4. [دوال مساعدة داخلية](#دوال-مساعدة-داخلية)
5. [أمثلة شاملة](#أمثلة-شاملة)

---

## مقدمة

كلاس `UnitNameHelper` هو المسؤول المركزي عن إدارة أسماء وتسميات الوحدات في نظام `Tss.Inventory`. يوفر واجهة برمجية موحدة للوصول إلى معلومات الوحدات المدعومة، مثل الحصول على التسميات العربية والإنجليزية، البحث عن الوحدات، تصنيفها حسب النوع، واستخراج المعلومات التفصيلية (الرمز، النوع، السياق، كونه وحدة أساسية، ...). يعتمد هذا الكلاس على بيانات داخلية (مصفوفات) ويدعم الترجمة عبر ملفات اللغة، مما يجعله أداة أساسية لبناء واجهات المستخدم والتحقق من صحة المدخلات.

---

## الثوابت

| الثابت | القيمة | الوصف |
|--------|--------|-------|
| `LOCALIZATION_KEY` | `'tss.inventory::lang.units.'` | مفتاح ملف الترجمة الخاص بالوحدات |

---

## الدوال الرئيسية

### `getAllUnitsWithLabels`

```php
public static function getAllUnitsWithLabels($lang = null): array
```

**الوصف:**  
إرجاع قائمة بجميع الوحدات المدعومة في النظام مع تسمياتها حسب اللغة المطلوبة.

**المعاملات:**
- `$lang` : (اختياري) رمز اللغة (`'ar'` أو `'en'`). إذا لم يُمرر، تُستخدم اللغة الحالية للتطبيق.

**الإرجاع:**  
مصفوفة ترابطية `[رمز_الوحدة => التسمية]`.

**مثال:**
```php
$allUnits = UnitNameHelper::getAllUnitsWithLabels('ar');
// ['mm' => 'مليمتر', 'cm' => 'سنتيمتر', 'm' => 'متر', ...]
```

---

### `getAllUnitsFilterWithLabels`

```php
public static function getAllUnitsFilterWithLabels($isBaseUnit = null, $lang = null): array
```

**الوصف:**  
إرجاع قائمة بجميع الوحدات مع إمكانية تصفيتها حسب كونها وحدة أساسية أم لا.

**المعاملات:**
- `$isBaseUnit` : (اختياري) إذا كان `true`، تُرجع الوحدات الأساسية فقط. إذا كان `false`، تُرجع غير الأساسية. إذا كان `null`، تُرجع الكل.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية `[رمز_الوحدة => التسمية]` بعد التصفية.

**مثال:**
```php
$baseUnits = UnitNameHelper::getAllUnitsFilterWithLabels(true, 'ar');
// فقط الوحدات الأساسية (مثل 'm', 'kg', 'pcs')
```

---

### `getUnitsByTypeWithLabels`

```php
public static function getUnitsByTypeWithLabels(string $type, $lang = null, $isBaseUnit = null): array
```

**الوصف:**  
إرجاع قائمة بالوحدات التي تنتمي إلى نوع معين (مثل `length`, `weight`)، مع إمكانية تصفية الأساسية.

**المعاملات:**
- `$type` : نوع الوحدة (أحد ثوابت `UnitType`).
- `$lang` : اللغة.
- `$isBaseUnit` : (اختياري) فلترة حسب الأساسية.

**الإرجاع:**  
مصفوفة ترابطية `[رمز_الوحدة => التسمية]`.

**مثال:**
```php
$lengthUnits = UnitNameHelper::getUnitsByTypeWithLabels(UnitType::LENGTH, 'ar');
/*
[
    'mm' => 'مليمتر',
    'cm' => 'سنتيمتر',
    'm' => 'متر',
    'km' => 'كيلومتر',
    'in' => 'بوصة',
    'ft' => 'قدم',
    'yd' => 'يارد',
    'mile' => 'ميل',
    'nmi' => 'ميل بحري'
]
*/
```

---

### `getUnitLabel`

```php
public static function getUnitLabel(string $unit, $lang = null): string
```

**الوصف:**  
الحصول على التسمية (حسب اللغة) لرمز وحدة معين.

**المعاملات:**
- `$unit` : رمز الوحدة (مثل `'kg'`).
- `$lang` : اللغة.

**الإرجاع:**  
تسمية الوحدة، أو رمزها نفسه إذا لم توجد التسمية.

**مثال:**
```php
echo UnitNameHelper::getUnitLabel('kg', 'ar'); // "كيلوجرام"
echo UnitNameHelper::getUnitLabel('kg', 'en'); // "Kilogram"
```

---

### `getEnglishLabel`

```php
public static function getEnglishLabel(string $unit): string
```

**الوصف:**  
إرجاع التسمية الإنجليزية للوحدة (من القائمة الداخلية، بغض النظر عن الترجمة).

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
التسمية الإنجليزية.

**مثال:**
```php
echo UnitNameHelper::getEnglishLabel('dozen'); // "Dozen"
```

---

### `getCategorizedUnits`

```php
public static function getCategorizedUnits($lang = null): array
```

**الوصف:**  
إرجاع جميع الوحدات مصنفة حسب النوع، مع معلومات إضافية مثل `label` (تسمية النوع) و `has_subtypes` (للكميات).

**المعاملات:**
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية معقدة، حيث المفتاح هو نوع الوحدة، والقيمة مصفوفة تحتوي على:
- `label` : تسمية النوع.
- `units` : مصفوفة `[الرمز => التسمية]` لوحدات ذلك النوع.
- `has_subtypes` : `bool` (خاص بالكميات).
- `subtypes` : (إن وجد) مصفوفة بأنواع الكميات الفرعية.

**مثال:**
```php
$categorized = UnitNameHelper::getCategorizedUnits('ar');
foreach ($categorized as $type => $info) {
    echo $info['label'] . ":\n";
    foreach ($info['units'] as $symbol => $label) {
        echo "  $symbol : $label\n";
    }
}
```

---

### `getSelectOptions`

```php
public static function getSelectOptions(?string $type = null, string $selectedUnit = '', bool $includeEmpty = true, $lang = null): string
```

**الوصف:**  
توليد كود HTML `<option>` لعمل قائمة منسدلة (select) تحتوي على وحدات من نوع معين (أو كل الوحدات).

**المعاملات:**
- `$type` : نوع الوحدات (اختياري). إذا كان `null`، تُعرض كل الوحدات.
- `$selectedUnit` : رمز الوحدة التي يجب أن تكون محددة مسبقاً (يُضاف لها `selected`).
- `$includeEmpty` : إضافة خيار فارغ في البداية.
- `$lang` : اللغة.

**الإرجاع:**  
نص HTML للخيارات.

**مثال:**
```php
echo '<select name="unit">';
echo UnitNameHelper::getSelectOptions('weight', 'kg', true, 'ar');
echo '</select>';
// الناتج: <option value="">-- اختر وحدة --</option>
//         <option value="mg">مليجرام</option>
//         <option value="g">جرام</option>
//         <option value="kg" selected>كيلوجرام</option>
//         ...
```

---

### `getGroupedSelectOptions`

```php
public static function getGroupedSelectOptions(string $selectedUnit = '', bool $includeEmpty = true, $lang = null): string
```

**الوصف:**  
توليد خيارات `<select>` مصنفة حسب النوع باستخدام `<optgroup>`.

**المعاملات:**
- `$selectedUnit` : الوحدة المحددة.
- `$includeEmpty` : إضافة خيار فارغ.
- `$lang` : اللغة.

**الإرجاع:**  
نص HTML للخيارات مع تجميع حسب النوع.

**مثال:**
```php
echo '<select name="unit">';
echo UnitNameHelper::getGroupedSelectOptions('kg', true, 'ar');
echo '</select>';
// الناتج: <option value="">-- اختر وحدة --</option>
//         <optgroup label="الطول">
//             <option value="mm">مليمتر</option> ...
//         </optgroup>
//         <optgroup label="الوزن">
//             <option value="mg">مليجرام</option>
//             <option value="g">جرام</option>
//             <option value="kg" selected>كيلوجرام</option> ...
//         </optgroup>
```

---

### `searchUnits`

```php
public static function searchUnits(string $searchText, bool $withLabels = true, $lang = null)
```

**الوصف:**  
بحث بسيط عن الوحدات التي تحتوي على النص المطلوب في الرمز أو التسمية العربية أو الإنجليزية.

**المعاملات:**
- `$searchText` : نص البحث.
- `$withLabels` : إذا كان `true`، تُرجع مصفوفة ترابطية `[الرمز => التسمية]`، وإلا تُرجع قائمة بالرموز فقط.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة (حسب `$withLabels`).

**مثال:**
```php
$results = UnitNameHelper::searchUnits('كيلو', true, 'ar');
// ['kg' => 'كيلوجرام', 'km' => 'كيلومتر', 'kcal' => 'كيلوسعرة', ...]
```

---

### `searchUnitAdvanced`

```php
public static function searchUnitAdvanced(string $searchText, ?string $type = null, $lang = null): ?array
```

**الوصف:**  
بحث متقدم يُرجع معلومات كاملة عن أول وحدة تطابق النص (حسب ترتيب أولوية: المطابقة التامة للرمز، ثم الاسم العربي، ثم الإنجليزي، ثم البحث الجزئي).

**المعاملات:**
- `$searchText` : نص البحث.
- `$type` : (اختياري) تحديد نوع الوحدة للبحث ضمن نطاق معين.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة تحتوي على معلومات كاملة عن الوحدة (كما في `getUnitFullInfo`) أو `null` إذا لم يُعثر على شيء.

**مثال:**
```php
$info = UnitNameHelper::searchUnitAdvanced('كيلو');
if ($info) {
    echo $info['unit_name_ar']; // "كيلوجرام" (أول وحدة تطابق)
}
```

---

### `getUnitType`

```php
public static function getUnitType(string $unit): ?string
```

**الوصف:**  
الحصول على نوع الوحدة (مثل `'length'`).

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
نوع الوحدة كسلسلة نصية، أو `null` إذا كانت غير معروفة.

**مثال:**
```php
$type = UnitNameHelper::getUnitType('kg'); // 'weight'
```

---

### `isValidUnit`

```php
public static function isValidUnit(string $unit): bool
```

**الوصف:**  
التحقق مما إذا كان رمز الوحدة معروفاً ومدعوماً في النظام.

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
`true` إذا كانت الوحدة موجودة، وإلا `false`.

**مثال:**
```php
if (UnitNameHelper::isValidUnit('kg')) {
    echo "الوحدة مدعومة";
}
```

---

### `getSimilarUnits`

```php
public static function getSimilarUnits(string $unit, $lang = null): array
```

**الوصف:**  
إرجاع قائمة بالوحدات المشابهة (من نفس النوع) لوحدة معينة.

**المعاملات:**
- `$unit` : رمز الوحدة.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية `[الرمز => التسمية]` للوحدات من نفس النوع (بما في ذلك الوحدة نفسها؟ - الكود يعيد الكل، لكن يفضل استبعاد نفسها).

**مثال:**
```php
$similar = UnitNameHelper::getSimilarUnits('m', 'ar');
// ['mm' => 'مليمتر', 'cm' => 'سنتيمتر', 'm' => 'متر', 'km' => 'كيلومتر', ...]
```

---

### `getUnitsByContext`

```php
public static function getUnitsByContext(string $context, bool $withLabels = true, $lang = null)
```

**الوصف:**  
إرجاع وحدات تنتمي إلى سياق معين (للوحدات الكمية فقط).

**المعاملات:**
- `$context` : السياق (`'general'`, `'packaging'`, `'paper'`, `'pairs'`, `'large'`).
- `$withLabels` : إذا كان `true`، تُرجع مصفوفة ترابطية؛ وإلا قائمة رموز.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة حسب `$withLabels`.

**مثال:**
```php
$packagingUnits = UnitNameHelper::getUnitsByContext('packaging', true, 'ar');
// ['dozen' => 'درزن', 'half_dozen' => 'نصف درزن', 'box' => 'علبة', ...]
```

---

### `getBaseUnits`

```php
public static function getBaseUnits($lang = null): array
```

**الوصف:**  
إرجاع الوحدات الأساسية لكل نوع (مثل `m` للطول، `kg` للوزن) مع تسمياتها.

**المعاملات:**
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية `[النوع => ['unit' => الرمز, 'label' => التسمية]]`.

**مثال:**
```php
$baseUnits = UnitNameHelper::getBaseUnits('ar');
// [
//     'length' => ['unit' => 'm', 'label' => 'متر'],
//     'weight' => ['unit' => 'kg', 'label' => 'كيلوجرام'],
//     ...
// ]
```

---

### `getBaseUnitByType`

```php
public static function getBaseUnitByType(string $type): ?string
```

**الوصف:**  
الحصول على رمز الوحدة الأساسية لنوع معين.

**المعاملات:**
- `$type` : نوع الوحدة.

**الإرجاع:**  
رمز الوحدة الأساسية أو `null` إذا لم يوجد.

**مثال:**
```php
$base = UnitNameHelper::getBaseUnitByType(UnitType::LENGTH); // 'm'
```

---

### `getBaseUnitInfoByType`

```php
public static function getBaseUnitInfoByType(string $type, $lang = null): ?array
```

**الوصف:**  
الحصول على معلومات كاملة عن الوحدة الأساسية لنوع معين.

**المعاملات:**
- `$type` : نوع الوحدة.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة تحتوي على `symbol`, `label`, `english_label`, `type`, `type_label` أو `null`.

---

### `getBaseUnitWithLabel`

```php
public static function getBaseUnitWithLabel(string $type, $lang = null): array
```

**الوصف:**  
إرجاع مصفوفة `[الرمز => التسمية]` للوحدة الأساسية لنوع معين.

**المعاملات:**
- `$type` : نوع الوحدة.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة تحتوي على عنصر واحد (أو فارغة).

**مثال:**
```php
$base = UnitNameHelper::getBaseUnitWithLabel(UnitType::LENGTH, 'ar'); // ['m' => 'متر']
```

---

### `getAllBaseUnitsForMainTypes`

```php
public static function getAllBaseUnitsForMainTypes($lang = null): array
```

**الوصف:**  
إرجاع جميع الوحدات الأساسية للأنواع الرئيسية فقط (بدون الأنواع الفرعية).

**المعاملات:**
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية `[النوع => ['symbol' => الرمز, 'label' => التسمية, 'type_label' => تسمية النوع]]`.

---

### `getBaseUnitsForQuantitySubtypes`

```php
public static function getBaseUnitsForQuantitySubtypes($lang = null): array
```

**الوصف:**  
إرجاع الوحدات الأساسية للأنواع الفرعية للكميات فقط.

**المعاملات:**
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية مشابهة للسابقة.

---

### `isBaseUnitForItsType`

```php
public static function isBaseUnitForItsType(string $unit): bool
```

**الوصف:**  
التحقق مما إذا كانت الوحدة هي الوحدة الأساسية لنوعها.

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
`true` إذا كانت أساسية لنوعها.

**مثال:**
```php
UnitNameHelper::isBaseUnitForItsType('m');   // true
UnitNameHelper::isBaseUnitForItsType('cm');  // false
```

---

### `filterBaseUnits`

```php
public static function filterBaseUnits(array $units): array
```

**الوصف:**  
تصفية قائمة من رموز الوحدات لاستبقاء الأساسية فقط.

**المعاملات:**
- `$units` : مصفوفة من رموز الوحدات.

**الإرجاع:**  
مصفوفة تحتوي فقط على الرموز الأساسية.

**مثال:**
```php
$filtered = UnitNameHelper::filterBaseUnits(['m', 'cm', 'kg']); // ['m', 'kg']
```

---

### `convertToBaseUnit`

```php
public static function convertToBaseUnit(string $unit): string
```

**الوصف:**  
تحويل رمز وحدة إلى الوحدة الأساسية لنوعها. إذا لم يتم العثور على نوع، يُعاد الرمز نفسه.

**المعاملات:**
- `$unit` : رمز الوحدة.

**الإرجاع:**  
رمز الوحدة الأساسية.

**مثال:**
```php
$base = UnitNameHelper::convertToBaseUnit('cm'); // 'm'
```

---

### `getBaseUnitsList`

```php
public static function getBaseUnitsList($lang = null): array
```

**الوصف:**  
إرجاع قائمة بالوحدات الأساسية (مرة واحدة لكل وحدة) مع معلوماتها الكاملة.

**المعاملات:**
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة ترابطية `[الرمز => معلومات كاملة]`.

---

### `getUnitInfo`

```php
public static function getUnitInfo(string $unit, $lang = null): ?array
```

**الوصف:**  
الحصول على معلومات أساسية عن الوحدة (الرمز، التسمية، النوع، كونه أساسية، السياق، عامل التحويل).

**المعاملات:**
- `$unit` : رمز الوحدة.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة تحتوي على `symbol`, `label`, `english_label`, `type`, `type_label`, `is_base_unit`, `context`, `conversion_factor`, `conversion_factor_formatted` أو `null`.

**مثال:**
```php
$info = UnitNameHelper::getUnitInfo('dozen', 'ar');
/*
[
    'symbol' => 'dozen',
    'label' => 'درزن',
    'english_label' => 'Dozen',
    'type' => 'quantity_packaging',
    'type_label' => 'كميات التعبئة',
    'is_base_unit' => true,
    'context' => 'packaging',
    'conversion_factor' => 12.0,
    'conversion_factor_formatted' => '12'
]
*/
```

---

### `getUnitFullInfo`

```php
public static function getUnitFullInfo(string $unit, $lang = null): ?array
```

**الوصف:**  
الحصول على معلومات كاملة ومفصلة عن الوحدة، بما في ذلك أسماء الأنواع، النظام المتري/الإمبراطوري، وأسماء العرض.

**المعاملات:**
- `$unit` : رمز الوحدة.
- `$lang` : اللغة.

**الإرجاع:**  
مصفوفة تحتوي على:
- `unit_id` : رمز الوحدة.
- `unit_name_ar`, `unit_name_en` : الأسماء.
- `type_id`, `type_name_ar`, `type_name_en` : معلومات النوع.
- `is_base_unit`, `base_unit`, `context`.
- `is_metric`, `is_imperial`.
- `display_name`, `full_name`.

**مثال:**
```php
$full = UnitNameHelper::getUnitFullInfo('kg', 'ar');
/*
[
    'unit_id' => 'kg',
    'unit_name_ar' => 'كيلوجرام',
    'unit_name_en' => 'Kilogram',
    'type_id' => 'weight',
    'type_name_ar' => 'الوزن',
    'type_name_en' => 'Weight',
    'is_base_unit' => true,
    'base_unit' => 'kg',
    'context' => null,
    'is_metric' => true,
    'is_imperial' => false,
    'display_name' => 'كيلوجرام',
    'full_name' => 'كيلوجرام (Kilogram)'
]
*/
```

---

## دوال مساعدة داخلية

- `getTransLabel(string $unit, string $defaultLabel, $lang)`: تحصل على التسمية المترجمة من ملفات اللغة، وإلا ترجع التسمية الافتراضية.
- `getUnitContext(string $unit)`: تحدد سياق الوحدة (للوحدات الكمية) بناءً على قائمة داخلية.
- `isMetricUnit(string $unit)`, `isImperialUnit(string $unit)`: تتحقق من النظام القياسي للوحدة.

---

## أمثلة شاملة

### مثال 1: بناء قائمة منسدلة مصنفة لواجهة مستخدم
```php
$html = '<form><select name="unit">';
$html .= UnitNameHelper::getGroupedSelectOptions('kg', true, 'ar');
$html .= '</select></form>';
echo $html;
```

### مثال 2: البحث عن وحدة وعرض معلوماتها
```php
$search = 'درزن';
$info = UnitNameHelper::searchUnitAdvanced($search, null, 'ar');
if ($info) {
    echo "الوحدة: {$info['unit_name_ar']} ({$info['unit_id']})\n";
    echo "النوع: {$info['type_name_ar']}\n";
    echo "هل هي أساسية؟ " . ($info['is_base_unit'] ? 'نعم' : 'لا') . "\n";
    echo "السياق: " . ($info['context'] ?? 'لا يوجد') . "\n";
}
```

### مثال 3: الحصول على جميع وحدات الوزن الأساسية
```php
$weightBaseUnits = UnitNameHelper::getUnitsByTypeWithLabels(UnitType::WEIGHT, 'ar', true);
// ['kg' => 'كيلوجرام'] (إذا كان الكيلوجرام هو الأساسي)
```

### مثال 4: التحقق من صحة رمز وحدة قبل الاستخدام
```php
$unitCode = 'xyz';
if (!UnitNameHelper::isValidUnit($unitCode)) {
    throw new Exception("الوحدة غير مدعومة");
}
```

### مثال 5: الحصول على معلومات كاملة لوحدة وعرضها
```php
$unit = 'dozen';
$info = UnitNameHelper::getUnitFullInfo($unit, 'ar');
if ($info) {
    echo $info['full_name'] . "\n"; // "درزن (Dozen)"
    echo "النوع: {$info['type_name_ar']}\n";
    echo "النظام: " . ($info['is_metric'] ? 'متري' : 'إمبراطوري') . "\n";
}
```

---

بهذا نكون قد غطينا جميع دوال الكلاس `UnitNameHelper` مع شرح وافٍ وأمثلة عملية. هذا الكلاس يُعد المصدر الأساسي لكل ما يتعلق بأسماء وتصنيفات الوحدات في النظام.