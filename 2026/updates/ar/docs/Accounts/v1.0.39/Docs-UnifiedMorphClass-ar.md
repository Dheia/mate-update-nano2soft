# توثيق سمة `UnifiedMorphClass`

**الإصدار:** 1.0.39 (إضافة `Tss.Accounts`)  
**المؤلف:** فريق تطوير TSS (Tss Team)  
**الرخصة:** خاصة بـ NanoSoft  
**آخر تحديث:** 2026-05-30

---

## المحتويات

1. [مقدمة](#مقدمة)
2. [الغرض من السمة](#الغرض-من-السمة)
3. [التثبيت والاعتماديات](#التثبيت-والاعتماديات)
4. [التركيب والاستخدام](#التركيب-والاستخدام)
5. [آلية العمل](#آلية-العمل)
6. [التكامل مع `System\Models\File`](#التكامل-مع-systemmodelsfile)
7. [التوسيع الديناميكي عبر `Plugin.php`](#التوسيع-الديناميكي-عبر-pluginphp)
8. [الفوائد والمزايا](#الفوائد-والمزايا)
9. [أمثلة عملية](#أمثلة-عملية)
10. [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
11. [المراجع](#المراجع)

---

## مقدمة

`UnifiedMorphClass` هي سمة (Trait) صُممت خصيصاً لإضافة `Tss.Accounts` بهدف توحيد اسم النوع polymorphic (morph class) لجميع نماذج السندات المحاسبية التي قد ترتبط بها ملفات من خلال علاقات `attachOne` أو `attachMany` مع `System\Models\File`. هذه السمة تضمن أن جميع الملفات المرفقة بالسندات (سندات قبض، صرف، قيود يومية، تحويلات، إلخ) تُخزَّن في جدول `system_files` بنفس قيمة `attachment_type`، وهي `Tss\Accounts\Models\TransactionHeader`. هذا التوحيد يبسط استعلامات البحث عن الملفات المرتبطة بالسندات، ويمنع ازدواجية أنواع المرفقات، ويُسهل عمليات التصفية والإدارة عبر الواجهات الخلفية وواجهات API.

---

## الغرض من السمة

- **توحيد `attachment_type`:** جعل جميع ملفات السندات تستخدم نفس الكلاس المرجعي (`TransactionHeader`)، بغض النظر عن النوع الحقيقي للسند (CatchReceipt, PayReceipt, BondsDay, إلخ).
- **تبسيط استعلامات الملفات:** عند الحاجة لجلب جميع الملفات المرتبطة بأي سند محاسبي، يمكن الاستعلام مباشرة على `attachment_type = TransactionHeader::class`.
- **منع تعقيد الـ polymorphic relations:** تجنب أن تتوزع الملفات على عدة `attachment_type` مختلفة مما يصعب تتبعها وإدارتها.
- **دعم `extendMorphMapTransactionHeader`:** توفير أساس متين للدالة الموسعة التي تعيد توجيه `morphMap` إلى `TransactionHeader`.

---

## التثبيت والاعتماديات

هذه السمة جزء من إضافة `Tss.Accounts` (الإصدار 1.0.39 أو أحدث). لا تحتاج إلى تثبيت منفصل. لمجرد استخدامها، تأكد من:

- تثبيت `Tss.Accounts` عبر `composer` أو سوق الإضافات.
- تحديث الإضافة إلى الإصدار 1.0.39 على الأقل.
- تشغيل هجرات قاعدة البيانات (إذا كانت هناك تحديثات مرتبطة).

ليست هناك حاجة إلى أي اعتماديات إضافية غير `System\Models\File` الموجود في NanoSoft APP.

---

## التركيب والاستخدام

### طريقة الاستخدام الأساسية

يمكنك تطبيق السمة على أي نموذج ترغب في توحيد `morphClass` له:

```php
<?php namespace Tss\Accounts\Models;

use Model;
use Tss\Accounts\Traits\UnifiedMorphClass;

class MyCustomVoucher extends Model
{
    use UnifiedMorphClass;
    
    // بقية تعريف النموذج
}
```

### الاستخدام في نماذج `Tss.Accounts` القياسية

جميع النماذج التالية تم توسيعها ديناميكياً (دون الحاجة لتعديل ملفاتها الأصلية) باستخدام `Plugin::extend`:

- `TransactionHeader`
- `BindingBond`
- `OpeningBalance`
- `Bonds2Day`
- `BondsDay`
- `CatchReceipt`
- `PayReceipt`
- `CreditNote`
- `DebitNote`
- `Expense`
- `Income`
- `Relay`
- `Transfer`
- `CreditDelivery` (من إضافة `Tss.Cards`)

لذلك، عند استخدام أي من هذه النماذج، تكون السمة قد طبقت تلقائياً.

### التحقق من تطبيق السمة

```php
$receipt = CatchReceipt::find(1);
echo $receipt->getMorphClass(); // يطبع: Tss\Accounts\Models\TransactionHeader
```

---

## آلية العمل

السمة تحتوي على دالة واحدة فقط، وهي إعادة تعريف `getMorphClass()`:

```php
<?php namespace Tss\Accounts\Traits;

trait UnifiedMorphClass
{
    /**
     * إرجاع اسم polymorphic موحد لجميع سندات الحسابات.
     *
     * @return string
     */
    public function getMorphClass()
    {
        return \Tss\Accounts\Models\TransactionHeader::class;
    }
}
```

عند استخدام هذه السمة في نموذج، فإن أي علاقة `morphTo`, `morphMany`, `morphOne`, أو `morphToMany` ستعتمد على القيمة المُرجعة من `getMorphClass()` لتحديد النوع.

**مثال توضيحي:**

بدون السمة، نموذج `CatchReceipt` يعيد `CatchReceipt::class` كـ `morphClass`. وعند رفع ملف عبر `$receipt->image = $file`، يُخزن `attachment_type` بقيمة `CatchReceipt`. هذا يؤدي إلى توزيع الملفات تحت عدة أنواع.

باستخدام السمة، `CatchReceipt` يعيد `TransactionHeader::class`، وبالتالي يُخزن `attachment_type` موحداً. والنتيجة أن جميع الملفات المرفقة بأي نوع سند تجد نفس `attachment_type`.

### التكامل مع `extendMorphMapTransactionHeader`

في `Plugin.php` لإضافة `Tss.Accounts`، توجد دالة `extendMorphMapTransactionHeader` التي تعيد توجيه الـ `morphMap` لجميع هذه النماذج إلى `TransactionHeader`. تضمن هذه الدالة أن أي عملية بحث تستخدم `TransactionHeader` ستجلب الملفات المرتبطة بأي سند، حتى لو كان النموذج الأصلي مختلفاً.

---

## التكامل مع `System\Models\File`

عند رفع ملف إلى نموذج يستخدم `UnifiedMorphClass`، يقوم `System\Models\File` تلقائياً بتسجيل `attachment_type` باعتباره القيمة المُرجعة من `getMorphClass()` للموديل الأصلي. هذا السلوك يأتي من NanoSoft APP ولا يحتاج إلى تعديل.

**مثال:**  
إذا قمت برفع صورة إلى `CatchReceipt` عبر `$receipt->image = $uploadedFile`، فإن `System\Models\File` سيخزن `attachment_type = Tss\Accounts\Models\TransactionHeader`. وبالتالي يمكنك استرجاع جميع ملفات السندات بغض النظر عن نوعها باستخدام:

```php
$files = File::where('attachment_type', TransactionHeader::class)->get();
```

---

## التوسيع الديناميكي عبر `Plugin.php`

تستخدم إضافة `Tss.Accounts` ميزة التوسيع الديناميكي لتطبيق السمة على جميع النماذج المذكورة دون الحاجة إلى تعديل ملفات النماذج الأصلية. الكود التالي موجود في `Plugin.php`:

```php
public function extendMorphMapTransactionHeader()
{
    $models = [
        TransactionHeader::class,
        BindingBond::class,
        OpeningBalance::class,
        Bonds2Day::class,
        BondsDay::class,
        CatchReceipt::class,
        PayReceipt::class,
        CreditNote::class,
        DebitNote::class,
        Expense::class,
        Income::class,
        Relay::class,
        Transfer::class,
        CreditDelivery::class,
    ];

    foreach ($models as $modelClass) {
        $modelClass::extend(function ($model) {
            $model->addDynamicMethod('getMorphClass', function () {
                return TransactionHeader::class;
            });
        });
    }
}
```

بهذه الطريقة، السمة مطبقة دون أن ترث النماذج السمة مباشرة، مما يحافظ على نقاوة الكود الأصلي.

---

## الفوائد والمزايا

| الفائدة | الوصف |
| :--- | :--- |
| **تبسيط استعلامات الملفات** | يمكن جلب جميع ملفات السندات (قبض، صرف، قيود) باستعلام واحد على `attachment_type = TransactionHeader::class`. |
| **توافق مع `Media Manager`** | تسهل إدارة الملفات عبر واجهات الإدارة التي تعتمد على `attachment_type`. |
| **تقليل تعقيد `morphMap`** | ليس هناك حاجة لتسجيل كل نوع سند في `morphMap`؛ يكفي تسجيل `TransactionHeader`. |
| **تسهيل عمليات الترحيل** | عند ترحيل السندات، تظل الملفات مرتبطة بالسند الأصلي دون فقدان العلاقة. |
| **دعم `extendMorphMapTransactionHeader`** | يوفر أساساً متيناً للدالة التي تحافظ على التوافق عند استعلامات `File` عبر `withTrashed` وغيره. |
| **لا حاجة لتعديل النماذج** | يتم التطبيق ديناميكياً، مما يسهل التحديثات المستقبلية. |

---

## أمثلة عملية

### 1. رفع ملف إلى سند قبض (CatchReceipt) والتحقق من `attachment_type`

```php
$receipt = CatchReceipt::find(1);
$file = new \System\Models\File();
$file->data = Input::file('document');
$file->save();

$receipt->files()->add($file);

// الملف الآن له attachment_type = TransactionHeader::class
echo $file->attachment_type; // Tss\Accounts\Models\TransactionHeader
```

### 2. جلب جميع ملفات سندات القبض والدفع والقيود اليومية

```php
use Tss\Accounts\Models\TransactionHeader;
use System\Models\File;

$allAccountFiles = File::where('attachment_type', TransactionHeader::class)->get();
```

### 3. استخدام `morphTo` في نموذج مخصص للحصول على السند الأصلي

```php
class CustomLog extends Model
{
    public $morphTo = ['transaction'];
}

$log = CustomLog::find(1);
$transaction = $log->transaction; // سيكون من النوع TransactionHeader (وليس CatchReceipt)
```

### 4. إضافة السمة يدوياً لنموذج جديد

```php
<?php namespace MyPlugin\Models;

use Model;
use Tss\Accounts\Traits\UnifiedMorphClass;

class CustomVoucher extends Model
{
    use UnifiedMorphClass;
    
    public $attachMany = [
        'files' => \System\Models\File::class
    ];
}
```

---

## استكشاف الأخطاء وإصلاحها

| المشكلة | السبب المحتمل | الحل |
| :--- | :--- | :--- |
| `Class 'UnifiedMorphClass' not found` | السمة غير موجودة أو لم يتم تضمينها | تأكد من تحديث `Tss.Accounts` إلى 1.0.39، وأن الملف موجود في `traits/UnifiedMorphClass.php`. |
| `getMorphClass()` لا يعيد الاسم الموحد | السمة لم تطبق على النموذج | تحقق من أنك استخدمت `use UnifiedMorphClass;` أو أن التوسيع الديناميكي يعمل. |
| الملفات المرفقة تخزن `attachment_type` القديم (مثلاً `CatchReceipt`) | السمة لم تُفعّل قبل رفع الملف | تأكد من ترتيب تحميل الإضافات ومسح الكاش (`php artisan cache:clear`). |
| تعارض مع `morphMap` مخصص آخر | إضافة أخرى تعيد تعريف `morphMap` | راجع توافق الإضافات، وقد تحتاج إلى ترتيب `$require` في `Plugin.php`. |
| خطأ `Call to undefined method getMorphClass` | السمة لم تطبق | أضف السمة يدوياً أو استخدم التوسيع الديناميكي. |

---

## الخاتمة

سمة `UnifiedMorphClass` هي حجر الزاوية لتوحيد إدارة الملفات المرفقة في النظام المحاسبي `Tss.Accounts`. بفضل هذه السمة، يمكن للمطورين الاعتماد على `TransactionHeader` كنوع واحد لجميع الملفات المرتبطة بالسندات، مما يبسط الاستعلامات ويحسن أداء النظام ويعزز قابلية الصيانة. السمة مطبقة ديناميكياً على جميع نماذج السندات الرئيسية، ويمكن استخدامها أيضاً في أي نموذج مخصص يرغب في مشاركة نفس الفضاء (namespace) من الملفات.

---

**المراجع**:
- [توثيق إضافة `Tss.Accounts` (الإصدار 1.0.39)](./Update-Accounts-v1.0.39-ar.md)
- [توثيق Behavior `DynamicAddMediaIncludes` (لـ API)](./docs/AccountsApi/Docs-DynamicAddMediaIncludes-ar.md)
- [دليل العلاقات متعددة الأشكال (Polymorphic Relations)](https://laravel.com/docs/11.x/eloquent-relationships#polymorphic-relations)