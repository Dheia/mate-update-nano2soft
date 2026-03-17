# توثيق الكلاس `UnitsInformation`

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 جدول المحتويات

1. [مقدمة](#مقدمة)
2. [الدوال العامة (Public Static Methods)](#الدوال-العامة-public-static-methods)
   - [displayAllUnitsInfoV1](#displayallunitsinfov1)
   - [displayAllUnitsInfoBootstrap](#displayallunitsinfobootstrap)
   - [displayAllUnitsInfo](#displayallunitsinfo)
   - [displayTypeInfoBootstrap](#displaytypeinfobootstrap)
   - [displayTypeInfo](#displaytypeinfo)
3. [الدوال الخاصة (Private Static Methods)](#الدوال-الخاصة-private-static-methods)
4. [أمثلة شاملة](#أمثلة-شاملة)

---

## مقدمة

كلاس `UnitsInformation` هو كلاس مسؤول عن توليد صفحات HTML لعرض معلومات شاملة عن جميع الوحدات المدعومة في النظام، مع إحصائيات، بطاقات تعريفية لكل وحدة، وأمثلة عملية للتحويل. الهدف منه هو توفير واجهة توثيقية وتجريبية للمطورين والمستخدمين للاطلاع على إمكانيات نظام الوحدات. يتوفر إصداران من العرض: إصدار بتصميم مخصص (V1) وإصدار متوافق مع Bootstrap (الافتراضي).

---

## الدوال العامة (Public Static Methods)

### `displayAllUnitsInfoV1`

```php
public static function displayAllUnitsInfoV1(): string
```

**الوصف:**  
توليد صفحة HTML كاملة بتصميم مخصص (غير معتمد على Bootstrap) تعرض جميع الوحدات مصنفة حسب النوع، مع إحصائيات، مربع بحث، وبطاقات تحتوي على معلومات الوحدة وأمثلة تحويل.

**الإرجاع:**  
نص HTML للصفحة الكاملة.

**مثال:**
```php
echo UnitsInformation::displayAllUnitsInfoV1();
```

---

### `displayAllUnitsInfoBootstrap`

```php
public static function displayAllUnitsInfoBootstrap(): string
```

**الوصف:**  
توليد صفحة HTML بتصميم Bootstrap تعرض جميع الوحدات مصنفة، مع إحصائيات، مربع بحث، وبطاقات قابلة للطي تحتوي على معلومات وأمثلة. هذه النسخة تستخدم Bootstrap 5 وتدعم التجاوب مع الأجهزة المختلفة.

**الإرجاع:**  
نص HTML للصفحة.

**مثال:**
```php
echo UnitsInformation::displayAllUnitsInfoBootstrap();
```

---

### `displayAllUnitsInfo`

```php
public static function displayAllUnitsInfo(): string
```

**الوصف:**  
دالة متوافقة مع الإصدارات السابقة، تقوم باستدعاء `displayAllUnitsInfoBootstrap()`.

**الإرجاع:**  
نص HTML.

**مثال:**
```php
echo UnitsInformation::displayAllUnitsInfo();
```

---

### `displayTypeInfoBootstrap`

```php
public static function displayTypeInfoBootstrap(string $typeKey): string
```

**الوصف:**  
عرض معلومات عن نوع معين من الوحدات فقط، مع بطاقات مبسطة لكل وحدة.

**المعاملات:**
- `$typeKey` : مفتاح النوع (مثل `'length'`).

**الإرجاع:**  
نص HTML يحتوي على بطاقات وحدات ذلك النوع.

**مثال:**
```php
echo UnitsInformation::displayTypeInfoBootstrap('weight');
```

---

### `displayTypeInfo`

```php
public static function displayTypeInfo(string $typeKey): string
```

**الوصف:**  
دالة متوافقة مع الإصدارات السابقة، تستدعي `displayTypeInfoBootstrap`.

**المعاملات:**
- `$typeKey` : مفتاح النوع.

**الإرجاع:**  
نص HTML.

**مثال:**
```php
echo UnitsInformation::displayTypeInfo('length');
```

---

## الدوال الخاصة (Private Static Methods)

هذه الدوال تستخدم داخلياً ولا يُقصد استدعاؤها مباشرة، ولكن نذكرها للتوضيح:

- `generateStatistics()`: توليد إحصائيات (عدد الأنواع والوحدات).
- `generateSearchBox()`: توليد مربع البحث.
- `generateAllUnitsInfo()`: توليد كل أقسام الوحدات (للمظهر المخصص).
- `generateUnitCard()`: توليد بطاقة وحدة للمظهر المخصص.
- `getUnitIcon()`: إرجاع الأيقونة المناسبة للوحدة.
- `generateConversionExample()`: توليد مثال عشوائي للتحويل.
- `getContextLabel()`: الحصول على تسمية السياق.
- `generateBootstrapStatistics()`: إحصائيات Bootstrap.
- `generateBootstrapSearchBox()`: مربع بحث Bootstrap.
- `generateBootstrapAllUnitsInfo()`: توليد أقسام الوحدات Bootstrap.
- `generateBootstrapUnitCard()`: بطاقة وحدة Bootstrap.
- `getTypeIcon()`: أيقونة النوع.
- `getContextBadge()`: بادج السياق.
- `getSearchScript()`: سكريبت البحث.
- `generateSimpleUnitCard()`: بطاقة مبسطة.

---

## أمثلة شاملة

### مثال 1: عرض صفحة الوحدات الكاملة في متحكم

```php
public function index()
{
    return UnitsInformation::displayAllUnitsInfo();
}
```

### مثال 2: عرض وحدات الوزن فقط داخل قالب موجود

```php
<div class="container">
    <h2>وحدات الوزن</h2>
    <?= UnitsInformation::displayTypeInfo('weight') ?>
</div>
```

### مثال 3: دمج مع Bootstrap مخصص

```php
<!DOCTYPE html>
<html>
<head>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <?= UnitsInformation::displayAllUnitsInfoBootstrap() ?>
</body>
</html>
```

---

## ملاحظات

- يتطلب العرض استخدام مكتبة Font Awesome للأيقونات.
- نسخة Bootstrap تحتاج إلى تضمين Bootstrap JS إذا أردت عمل الأزرار القابلة للطي (collapse).
- جميع النصوص والتسميات باللغة العربية.

--- 

بهذا نكون قد وثقنا الكلاس `UnitsInformation` بشكل شامل.