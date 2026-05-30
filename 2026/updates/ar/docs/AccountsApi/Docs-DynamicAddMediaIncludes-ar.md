# توثيق Behavior `DynamicAddMediaIncludes`

**الإصدار:** 1.1.1 (إضافة `Nano.AccountsApi`)  
**المسار:** `plugins/nano/accountsapi/behaviors/DynamicAddMediaIncludes.php`  
**المؤلف:** فريق تطوير نانوسوفت (NanoSoft Team)  
**الرخصة:** خاصة بـ NanoSoft  
**آخر تحديث:** 2026-05-30

---

## المحتويات

1. [مقدمة](#مقدمة)
2. [الغرض من السلوك](#الغرض-من-السلوك)
3. [المتطلبات والاعتماديات](#المتطلبات-والاعتماديات)
4. [الهيكل والخصائص](#الهيكل-والخصائص)
5. [دوال الـ Includes المقدمة](#دوال-الـ-includes-المقدمة)
6. [آلية التحكم في بيانات الميتا](#آلية-التحكم-في-بيانات-الميتا)
7. [آلية استخراج الإعدادات الخاصة بـ Transformer](#آلية-استخراج-الإعدادات-الخاصة-بـ-transformer)
8. [التكامل مع Transformers في `Nano.AccountsApi`](#التكامل-مع-transformers-في-nanoaccountsapi)
9. [الإعدادات (Configuration)](#الإعدادات-configuration)
10. [أمثلة عملية](#أمثلة-عملية)
11. [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
12. [المراجع](#المراجع)

---

## مقدمة

`DynamicAddMediaIncludes` هو Behavior مخصص لإضافة `includes` ديناميكية خاصة بالوسائط المتعددة (الصور، الملفات، الفيديوهات، التسجيلات الصوتية، الملفات التمهيدية) إلى أي Transformer في إضافة `Nano.AccountsApi`. يهدف هذا السلوك إلى توحيد طريقة تضمين بيانات الملفات المرتبطة بالنماذج المحاسبية (مثل سندات القبض، القيود اليومية، الحسابات، الشركاء، إلخ) في استجابات الـ API، مع توفير مرونة كاملة للتحكم في إرجاع بيانات الميتا (metadata) لكل نوع ملف على حدة.

يعتمد السلوك على دوال مساعدة من `Nano\Api\Helpers\ApiHelper` (مثل `image()`, `images()`, `files()`) لتنسيق البيانات بشكل موحد مع باقي إضافات نانوسوفت (مثل `Nano.LocationApi`). كما أنه يدعم قراءة الإعدادات من `config.php` ومتغيرات البيئة، بالإضافة إلى إمكانية تجاوزها عبر معاملات الطلب (request parameters).

---

## الغرض من السلوك

- **إضافة includes الوسائط إلى Transformers الحسابات** دون الحاجة إلى تعديل كل Transformer يدوياً.
- **توحيد تنسيق استجابات الميديا** عبر جميع الـ Transformers (نفس تنسيق `ProductTransformer` في `Nano.ShopApi`).
- **التحكم الدقيق في بيانات الميتا** لكل include على حدة (عام، خاص بـ Transformer، أو عبر الطلب).
- **تحسين أداء API** بعدم إرجاع بيانات الميتا إلا عند طلبها صراحة.
- **سهولة التوسع** بإضافة أنواع وسائط جديدة (مثل `videos`, `audios`) بمجرد تفعيل العلاقات في النماذج وإلغاء تعليق الدوال في السلوك.

---

## المتطلبات والاعتماديات

- **إضافة `Nano.API`** – لتوفير دوال `ApiHelper` والهيكل الأساسي للـ Transformers.
- **إضافة `Tss.Accounts` (الإصدار 1.0.39 أو أحدث)** – لتوفير علاقات `attachOne` و `attachMany` للنماذج المحاسبية.
- **نظام `System\Models\File`** – المدمج في NanoSoft APP.
- **PHP 8.0 أو أحدث**.

---

## الهيكل والخصائص

الـ Behavior موجود في `Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes` ويمتد من `October\Rain\Extension\ExtensionBase`.

### الخصائص الرئيسية

| الخاصية | النوع | الوصف |
| :--- | :--- | :--- |
| `$transformer` | `Transformer` | مرجع إلى الـ Transformer المرتبط بالسلوك. |
| `$isMateData` | `bool` | (داخلية) تخزين مؤقت لإعدادات الميتا العامة. |

### المنشئ (`__construct`)

- يتحقق من وجود `availableIncludes` ويضيف `image`, `images`, `files` إليها (ويمكن تفعيل `videos`, `audios`, `book_intro` بتعديل المصفوفة).
- يضيف بعضها إلى `defaultIncludes` حسب إعداد `default_media_includes` (من الكونفيغ أو متغير البيئة).
- يستدعي `getDefaultIncludeForConfig` للحصول على الإعدادات الخاصة بـ Transformer إن وُجدت.

### دوال مساعدة داخلية

| الدالة | الوصف |
| :--- | :--- |
| `shouldReturnMateData()` | تحديد ما إذا كانت بيانات الميتا مطلوبة بشكل عام (من الطلب أو الإعدادات). |
| `getMateDataForInclude(string $includeName)` | تحديد ما إذا كانت بيانات الميتا مطلوبة لـ include معين (يتجاوز الإعداد العام). |
| `getDefaultIncludeForConfig($class, $defaultValue, $prefixConfig, $suffixConfig)` | استخراج الإعدادات الخاصة بـ Transformer بناءً على اسم الكلاس. |
| `pascalToSnake(string $input)` | تحويل اسم Transformer من `PascalCase` إلى `snake_case`. |

---

## دوال الـ Includes المقدمة

كل دالة تمثل include يمكن طلبه عبر معامل `include` في API. جميعها تعيد مورداً من نوع `Primitive` (للمفرد) أو `Collection` (للجماعة) من Fractal.

| الدالة | الـ Include | نوع المورد | العلاقة المتوقعة في النموذج | القيمة الافتراضية عند الخطأ |
| :--- | :--- | :--- | :--- | :--- |
| `includeImage($model)` | `image` | `Primitive` | `attachOne 'image'` | `null` |
| `includeImages($model)` | `images` | `Primitive` (تعيد مصفوفة) | `attachMany 'images'` | `[]` |
| `includeFiles($model)` | `files` | `Primitive` (تعيد مصفوفة) | `attachMany 'files'` | `[]` |
| `includeVideos($model)` | `videos` | `Primitive` | `attachMany 'videos'` (معطل افتراضياً) | `[]` |
| `includeAudios($model)` | `audios` | `Primitive` | `attachMany 'audios'` (معطل) | `[]` |
| `includeBookIntro($model)` | `book_intro` | `Primitive` | `attachOne 'book_intro'` (معطل) | `null` |

**ملاحظة:** دوال `includeImages`, `includeFiles`, `includeVideos`, `includeAudios` تُعيد `Primitive` يحتوي على مصفوفة، بدلاً من `Collection`. هذا يضمن تنسيقاً موحداً مع دوال `ApiHelper` التي تعيد مصفوفة جاهزة.

### آلية عمل الدوال

1. تتحقق من صحة `$model` ووجود الدالة الخاصة بالعلاقة (`method_exists($model, 'image')` أو `isset($model->images)`).
2. تستدعي الدالة المساعدة من `ApiHelper` (`image()`, `images()`, `files()`) مع تمرير معامل `$isMate` المستخرج من `getMateDataForInclude()`.
3. في حال حدوث استثناء، تسجله (في بيئة التطوير فقط) وتعطي القيمة الافتراضية الآمنة (`null` أو `[]`).
4. تُعيد النتيجة ضمن `Primitive`.

**سبب استخدام `ApiHelper` بدلاً من دوال الـ Transformer الأصلية:**  
لضمان تنسيق موحد للبيانات (مثل `original`, `small`, `medium`, `thumb` للصور) وللاستفادة من دعم resize وغيرها من الميزات الموجودة في `ApiHelper`.

---

## آلية التحكم في بيانات الميتا

بيانات الميتا تتضمن معلومات الملف مثل `id`, `title`, `description`, `file_name`, `file_size`, `extension`, `content_type`, `path`, `created_at`, `updated_at`. يمكن التحكم في إرجاعها عبر ثلاث طبقات (الأولى لها الأولوية):

### 1. معامل الطلب الخاص بـ include

عن طريق إرسال معامل بنفس اسم الـ include يحتوي على `mate_data`:

```http
GET /api/v1/accounts/catchreceipts/1?include=image,files&image[mate_data]=1
```

### 2. معامل الطلب العام `is_mate_data_media`

```http
GET /api/v1/accounts/catchreceipts/1?include=image&is_mate_data_media=1
```

### 3. الإعدادات الخاصة بـ Transformer (عبر `config.php`)

مثال:

```php
'catch_receipts_v2' => [
    'is_mate_data_image' => true,
    'is_mate_data_files' => false,
],
```

### 4. الإعداد العام `is_mate_data_media` في `config.php` أو متغير البيئة

```php
'is_mate_data_media' => env('NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA', false),
```

**ترتيب الأولوية:**  
الخاص بـ include > العام في الطلب > الخاص بـ Transformer > العام في الإعدادات.

---

## آلية استخراج الإعدادات الخاصة بـ Transformer

تستخدم الدالة `getDefaultIncludeForConfig` لتوليد اسم الإعداد بناءً على اسم الـ Transformer. الخطوات:

1. استخراج اسم الكلاس الأساسي (مثال: `CatchReceiptTransformer`).
2. إزالة لاحقة `Transformer` (تصبح `CatchReceipt`).
3. تحويل الاسم إلى `snake_case` (تصبح `catch_receipt`).
4. محاولة جلب الإعداد من `config('nano.accountsapi::config.' . $name . '.include')` أو `... . 'is_mate_data_image'`.
5. تجربة إزالة أو إضافة حرف `s` في نهاية الاسم (للتمييز بين المفرد والجمع).
6. تجربة إزالة الشرطات السفلية (`_`).
7. إذا لم يتم العثور على إعداد، تُرجع القيمة الافتراضية.

**مثال:**  
Transformer باسم `CatchReceiptTransformer` → يبحث في:
- `nano.accountsapi::config.catch_receipt.include`
- `nano.accountsapi::config.catch_receipt.is_mate_data_image`
- ثم `nano.accountsapi::config.catch_receipts.include` (بإضافة `s`)
- ثم `nano.accountsapi::config.catchreceipt.include` (بدون شرطات)
- إلخ.

هذا يسمح بتخصيص دقيق لكل نوع سند.

---

## التكامل مع Transformers في `Nano.AccountsApi`

في ملف `Plugin.php` الخاص بـ `Nano.AccountsApi`، توجد دالة `extendAccountsApiTransformersWithMedia()` التي تقوم بالتالي:

```php
protected function extendAccountsApiTransformersWithMedia()
{
    $transformers = [
        \Nano\AccountsApi\Transformers\AccountTransformer::class,
        \Nano\AccountsApi\Transformers\ReferenceTransformer::class,
        // ... جميع Transformers الأخرى
    ];

    foreach ($transformers as $transformerClass) {
        $transformerClass::extend(function ($transformer) {
            if (!$transformer->isClassExtendedWith('Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes')) {
                $transformer->extendClassWith('Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes');
            }
        });
    }
}
```

هذا يضمن أن جميع Transformers التالية تمتلك الـ Behavior وتدعم `image`, `images`, `files` في `availableIncludes`:

- `AccountTransformer`
- `ReferenceTransformer`
- `GroupTransformer`
- `CostCenterTransformer`
- `PeriodTransformer`
- `BoxeTransformer`
- `BankTransformer`
- `PartnerTransformer`
- `TransactionHeaderTransformer`
- `TransactionsDetailTransformer`
- `BondsDayTransformer`
- `CatchReceiptTransformer`
- `PayReceiptTransformer`
- `CreditNoteTransformer`
- `DebitNoteTransformer`
- `TransferTransformer`
- `ExpenseTransformer`
- `IncomeTransformer`
- `OpeningBalanceTransformer`
- `BindingBondTransformer`
- `RelayTransformer`
- `Bonds2DayTransformer`
- `CreditDeliveryTransformer`
- وأيضاً الـ Transformer الأساسي (`\Nano\AccountsApi\Transformers\Transformer`).

---

## الإعدادات (Configuration)

يمكن تخصيص السلوك عبر ملف الإعدادات `config/nano/accountsapi/config.php` (بعد نشره باستخدام `php artisan config:publish Nano.AccountsApi`).

### الإعدادات العامة

| المفتاح | النوع | القيمة الافتراضية | الوصف |
| :--- | :--- | :--- | :--- |
| `default_media_includes` | `string\|array\|null` | `null` | قائمة بـ `includes` التي تضاف إلى `defaultIncludes`. أمثلة: `'image,files'` أو `['image', 'files']`. |
| `is_mate_data_media` | `bool` | `false` | إرجاع بيانات الميتا للميديا بشكل عام. |

### الإعدادات الخاصة بـ Transformer (مثال لـ `catch_receipts_v2`)

يمكن إضافة أي من المفاتيح التالية تحت اسم الـ Transformer المحول إلى `snake_case`:

```php
'catch_receipts_v2' => [
    // إعدادات أخرى (order_by, transactions_type, is_allow_add...)
    'include' => ['image', 'files'],   // لتجاوز default_media_includes لهذا النوع
    'is_mate_data_image' => true,      // تفعيل الميتا للصورة
    'is_mate_data_files' => false,     // تعطيل الميتا للملفات
],
```

### متغيرات البيئة (`.env`)

يمكن استخدام متغيرات البيئة لتجاوز أي من الإعدادات:

```
NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES=image,files
NANO_ACCOUNTSAPI_IS_MATE_DATA_MEDIA=true
NANO_ACCOUNTSAPI_CATCH_RECEIPTS_IS_MATE_DATA_IMAGE=false
```

---

## أمثلة عملية

### 1. طلب حساب مع الصورة الرئيسية فقط (بدون ميتا)

```http
GET /api/v1/accounts/accounts/2-2-1231010001?include=image
```

**الرد:**

```json
{
  "data": {
    "id": 2,
    "code": "2-2-1231010001",
    "name": "عملاء",
    "image": {
      "original": "https://domain.com/storage/accounts/customer.jpg",
      "small": "https://domain.com/storage/accounts/customer_small.jpg"
    }
  }
}
```

### 2. طلب سند قبض مع الصورة والملفات، مع بيانات الميتا للصورة فقط

```http
GET /api/v1/accounts/catchreceipts/1?include=image,files&image[mate_data]=1
```

**الرد:**

```json
{
  "data": {
    "id": 1,
    "code": "CR-001",
    "image": {
      "original": "https://domain.com/receipts/1.jpg",
      "mate_data": {
        "id": 101,
        "file_name": "receipt.jpg",
        "file_size": 204800
      }
    },
    "files": [
      {
        "file_name": "invoice.pdf",
        "path": "https://domain.com/receipts/invoice.pdf"
      }
    ]
  }
}
```

### 3. تعيين `default_media_includes` لجلب الصورة والملفات تلقائياً لكل الطلبات

في `.env`:

```
NANO_ACCOUNTSAPI_DEFAULT_MEDIA_INCLUDES=image,files
```

ثم أي طلب لـ `/accounts/catchreceipts` سيعيد الصورة والملفات دون الحاجة لكتابة `include`.

### 4. تعطيل الميتا للصورة في Transformer معين فقط

في `config/nano/accountsapi/config.php`:

```php
'catch_receipts_v2' => [
    'is_mate_data_image' => false,
],
```

حتى إذا أرسل العميل `is_mate_data_media=1`، لن تُعاد بيانات الميتا للصورة في سندات القبض.

### 5. إضافة include مخصص (مثل `videos`) في Transformer معين

1. تفعيل العلاقة `videos` في النموذج (تأكد من وجود `attachMany 'videos'`).
2. في `DynamicAddMediaIncludes.php`، أضف `'videos'` إلى مصفوفة `$mediaIncludes` في المنشئ.
3. أضف الدالة `includeVideos` (موجودة بالفعل ولكنها معلقة).
4. أعد تشغيل الكاش.

---

## استكشاف الأخطاء وإصلاحها

| المشكلة | السبب المحتمل | الحل |
| :--- | :--- | :--- |
| `include=image` لا يعيد بيانات الصورة | العلاقة `image` غير موجودة في النموذج | تأكد من أن النموذج يحتوي على `$attachOne['image']` أو تمت إضافته ديناميكياً. |
| خطأ `Call to undefined method ApiHelper::image()` | إضافة `Nano.API` غير مثبتة أو قديمة | قم بتثبيت `Nano.API` والتأكد من وجود الدوال المساعدة. |
| بيانات الميتا لا تظهر رغم `is_mate_data_media=1` | ربما تم تعطيلها خاص بـ Transformer | تحقق من إعدادات `is_mate_data_image` أو `is_mate_data_files` في الكونفيغ. |
| `include=images` يعيد مصفوفة فارغة | لا توجد صور مرفقة | تأكد من رفع صور عبر الواجهة الخلفية أو API. |
| ظهور خطأ `Class 'Nano\AccountsApi\Behaviors\DynamicAddMediaIncludes' not found` | السلوك غير موجود في المسار الصحيح | تأكد من وجود الملف في `plugins/nano/accountsapi/behaviors/DynamicAddMediaIncludes.php`. |
| `getDefaultIncludeForConfig` لا يعيد الإعدادات المتوقعة | اسم الـ Transformer لا يتطابق مع المفتاح في الكونفيغ | تحقق من تحويل الاسم إلى `snake_case` وجرب إضافة مفتاح بالاسم الكامل. |

---

## الخاتمة

يُعد `DynamicAddMediaIncludes` Behavior أداة قوية ومرنة لتوحيد إدارة الوسائط المتعددة في واجهة برمجة التطبيقات المحاسبية. بفضل تصميمه المعياري واعتماده على دوال `ApiHelper`، فإنه يضمن اتساق التنسيق مع بقية إضافات نانوسوفت (`Nano.LocationApi`, `Nano.ShopApi`). كما أن دعمه للتحكم متعدد المستويات (عام، خاص بـ Transformer، عبر الطلب) يمنح المطورين تحكماً دقيقاً في استجابات الـ API، مما يحسن الأداء ويقلل حجم البيانات غير الضرورية.

للاستفادة القصوى من هذا السلوك، يُوصى بقراءة توثيق إضافة `Tss.Accounts` (خاصة قسم الوسائط المتعددة) وتوثيق `Nano.API` لفهم دوال `ApiHelper` بشكل أعمق.

