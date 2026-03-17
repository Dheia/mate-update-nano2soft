# توثيق الكلاس `ConversionForm`

**Namespace:** `Tss\Inventory\Classes\Units`

---

## 📋 جدول المحتويات

1. [مقدمة](#مقدمة)
2. [الدوال الرئيسية](#الدوال-الرئيسية)
   - [getConversionTypes](#getconversiontypes)
   - [getUnitsForSelectedType](#getunitsforselectedtype)
   - [renderAdvancedForm](#renderadvancedform)
   - [renderForm](#renderform)
   - [testQuantityConversions](#testquantityconversions)
3. [دوال داخلية](#دوال-داخلية)
   - [processConversion](#processconversion)
   - [getFormScript](#getformscript)
4. [أمثلة شاملة](#أمثلة-شاملة)

---

## مقدمة

كلاس `ConversionForm` هو كلاس مسؤول عن توليد نماذج HTML تفاعلية لاختبار وتحويل الوحدات. يوفر واجهة مستخدم بسيطة تسمح للمستخدم باختيار نوع الوحدة، الوحدة المصدر، الوحدة الهدف، والقيمة، ثم عرض نتيجة التحويل مع خيارات إضافية مثل الوضع الصارم ودقة النتيجة. هذا الكلاس مخصص أساساً لأغراض العرض التجريبي والاختبار، وليس للاستخدام المباشر في بيئات الإنتاج، حيث يعتمد على `$_POST` بشكل مباشر.

---

## الدوال الرئيسية

### `getConversionTypes`

```php
public function getConversionTypes(bool $includeSubtypes = false): array
```

**الوصف:**  
الحصول على قائمة بأنواع التحويل المتاحة (أنواع الوحدات) مع تسمياتها العربية.

**المعاملات:**
- `$includeSubtypes` : إذا كانت `true`، تُرجع جميع الأنواع بما في ذلك الأنواع الفرعية للكميات. إذا كانت `false`، تُرجع الأنواع الرئيسية فقط.

**الإرجاع:**  
مصفوفة ترابطية `[النوع => التسمية]`.

**مثال:**
```php
$form = new ConversionForm();
$types = $form->getConversionTypes(true);
// ['length' => 'الطول', 'weight' => 'الوزن', 'quantity_packaging' => 'كميات التعبئة', ...]
```

---

### `getUnitsForSelectedType`

```php
public function getUnitsForSelectedType(string $selectedType): array
```

**الوصف:**  
الحصول على قائمة بالوحدات التابعة لنوع معين، مع تسمياتها العربية.

**المعاملات:**
- `$selectedType` : نوع الوحدة (مثل `'length'`).

**الإرجاع:**  
مصفوفة ترابطية `[الرمز => التسمية]`.

**مثال:**
```php
$units = $form->getUnitsForSelectedType('length');
// ['mm' => 'مليمتر', 'cm' => 'سنتيمتر', 'm' => 'متر', ...]
```

---

### `renderAdvancedForm`

```php
public function renderAdvancedForm(): string
```

**الوصف:**  
توليد كود HTML لنموذج تحويل متقدم، يحتوي على:
- قائمة منسدلة لاختيار نوع التحويل (مقسمة إلى مجموعات: الأنواع الرئيسية وأنواع الكميات).
- قائمتان منسدلتان لاختيار الوحدة المصدر والوحدة الهدف.
- حقل إدخال القيمة.
- خيارات إضافية: الوضع الصارم (checkbox) ودقة النتيجة (select).
- زر التحويل.
- منطقة عرض النتيجة (تظهر بعد الإرسال).

**الإرجاع:**  
نص HTML كامل للنموذج.

**مثال:**
```php
$form = new ConversionForm();
echo $form->renderAdvancedForm();
```

**ملاحظات:**
- يعتمد النموذج على `$_POST` لعرض القيم المحددة مسبقاً والنتيجة.
- يتضمن كود JavaScript لتبديل الوحدات عند النقر على السهم، وتحديث النموذج عند تغيير نوع التحويل.

---

### `renderForm`

```php
public function renderForm(): string
```

**الوصف:**  
توليد كود HTML لنموذج تحويل بسيط، يحتوي على:
- قائمة منسدلة لاختيار نوع التحويل (بدون تجميع).
- قائمتان منسدلتان لاختيار الوحدات.
- حقل إدخال القيمة.
- زر التحويل.

**الإرجاع:**  
نص HTML للنموذج البسيط.

**مثال:**
```php
$form = new ConversionForm();
echo $form->renderForm();
```

---

### `testQuantityConversions`

```php
public static function testQuantityConversions(): void
```

**الوصف:**  
دالة اختبار (static) تقوم بتجربة مجموعة من تحويلات الكميات المعرفة مسبقاً، وتطبع النتائج في سطر الأوامر (أو مخرجات HTTP). مفيدة للتحقق من صحة بيانات التحويل.

**الاختبارات تشمل:**
- dozen → pcs
- gross → dozen
- box → dozen
- tray → box
- pair → half_dozen
- hundred → thousand
- million → thousand
- carton → dozen
- pallet → pcs
- ream → pcs
- pack → dozen
وغيرها.

**مثال:**
```php
ConversionForm::testQuantityConversions();
// سيتم طباعة نتائج الاختبار على الشاشة.
```

---

## دوال داخلية

### `processConversion`

```php
private function processConversion(): string
```

**الوصف:**  
معالجة بيانات النموذج بعد الإرسال. تقوم بالتحقق من المدخلات، وإجراء التحويل باستخدام `UnitConverter` (مع مراعاة الوضع الصارم عبر `UnitContextValidator`)، وإرجاع كود HTML لعرض النتيجة أو رسالة الخطأ.

**الإرجاع:**  
نص HTML يحتوي على نتيجة التحويل (نجاح أو خطأ).

### `getFormScript`

```php
private function getFormScript(): string
```

**الوصف:**  
إرجاع كود JavaScript اللازم للنموذج المتقدم، والذي يقوم بما يلي:
- إرسال النموذج تلقائياً عند تغيير نوع التحويل.
- تبديل قيم الوحدات عند النقر على السهم بين القائمتين.
- تحويل القيم السالبة إلى موجبة تلقائياً.

**الإرجاع:**  
نص JavaScript داخل وسم `<script>`.

---

## أمثلة شاملة

### مثال 1: تضمين النموذج المتقدم في صفحة ويب

```php
<?php
use Tss\Inventory\Classes\Units\ConversionForm;
?>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>محول الوحدات</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
</head>
<body>
    <div class="container mt-5">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header bg-primary text-white">
                        <h4>محول الوحدات الشامل</h4>
                    </div>
                    <div class="card-body">
                        <?php
                        $form = new ConversionForm();
                        echo $form->renderAdvancedForm();
                        ?>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

### مثال 2: استخدام النموذج البسيط في لوحة تحكم

```php
$form = new ConversionForm();
echo $form->renderForm();
```

### مثال 3: تشغيل اختبارات الكميات

```php
// يمكن استدعاؤها من أي مكان، مثل سطر الأوامر أو صفحة اختبار
ConversionForm::testQuantityConversions();
```

---

## ملاحظات هامة

- هذا الكلاس مخصص للعرض التجريبي والاختبار، ويعتمد بشكل كبير على `$_POST` مما يجعله غير مناسب للاستخدام في بيئات الإنتاج دون معالجة أمان إضافية.
- النماذج تستخدم Bootstrap 5 و Font Awesome للتصميم، لذا يجب تضمين هذه المكتبات في الصفحة.
- الدالة `testQuantityConversions` تطبع النتائج مباشرة، وهي مفيدة للمطورين للتحقق من صحة بيانات التحويل.

---