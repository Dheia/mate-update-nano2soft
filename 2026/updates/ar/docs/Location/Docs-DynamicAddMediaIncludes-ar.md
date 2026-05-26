# توثيق Behavior `DynamicAddMediaIncludes` – إضافة `Nano.LocationApi`

## نظرة عامة (Overview)

يُعد `DynamicAddMediaIncludes` Behavior مخصصاً لإضافة علاقات الوسائط المتعددة (Media) بشكل ديناميكي إلى أي Transformer من نوع `Nano\API\Classes\Transformer`، خاصة تلك المتعلقة بالنماذج الجغرافية مثل الدولة (Country)، المدينة/الولاية (State)، والمنطقة/المديرية (Directorate).

يهدف هذا الـ Behavior إلى توفير طريقة موحدة وقابلة لإعادة الاستخدام لتضمين الصور، معارض الصور، الفيديوهات، التسجيلات الصوتية، الملفات العامة، والملفات التمهيدية في استجابات API، مع دعم كامل للتحكم في إرجاع بيانات الميتا (metadata) الخاصة بكل ملف.

**المسار:** `plugins/nano/locationapi/behaviors/DynamicAddMediaIncludes.php`

---

## الغرض والفوائد (Purpose & Benefits)

- **توحيد آلية تضمين الوسائط** عبر جميع Transformers الجغرافية.
- **تقليل ازدواجية الكود** من خلال توفير Behavior واحد يمكن إرفاقه بأي Transformer.
- **دعم ديناميكي للـ includes** بحيث يمكن للمطور طلبها عند الحاجة فقط (`include=image,images`).
- **التحكم في بيانات الميتا** لكل ملف على حدة، مما يقلل حجم الاستجابة غير الضرورية.
- **معالجة آمنة للاستثناءات** وإرجاع قيم افتراضية في حال عدم وجود العلاقة أو حدوث خطأ.
- **التوافق مع معمارية `Nano.API`** واستخدام دوال `image()`, `images()`, `file()`, `files()` الموجودة في الـ Transformer الأساسي.

---

## الهيكل (Structure)

### الـ Namespace والميراث

```php
namespace Nano\LocationApi\Behaviors;

use October\Rain\Extension\ExtensionBase;
use Nano\API\Classes\Transformer;
use League\Fractal\Resource\Primitive;
use League\Fractal\Resource\Item;
use League\Fractal\Resource\Collection;
```

- يمتد الـ Behavior من `ExtensionBase` ليتوافق مع نظام الـ Extendable في October CMS.
- يعتمد على `Transformer` الأساسي لتوفير دوال تحويل الملفات.

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
| :--- | :--- | :--- |
| `$transformer` | `Transformer` | مرجع إلى الـ Transformer الذي يرتبط به الـ Behavior. |
| `$isMateData` | `bool` | (داخلي) يحدد ما إذا كانت بيانات الميتا مطلوبة أم لا. |

### الدوال (Methods)

#### `__construct(Transformer $transformer)`

- **الوصف:** منشئ الـ Behavior. يضيف تلقائياً الـ includes الخاصة بالوسائط إلى `availableIncludes` في الـ Transformer، ويضيفها إلى `defaultIncludes` حسب إعدادات التكوين.
- **المعاملات:**
  - `$transformer`: كائن الـ Transformer المرتبط.
- **التفاصيل:**
  - يتحقق من وجود `availableIncludes` ويضيف `image`, `images`, `videos`, `audios`, `files`, `book_intro` إن لم تكن موجودة.
  - إذا كان إعداد `nano.locationapi::default_media_includes` مُفعّلاً، يضيف هذه الـ includes إلى `defaultIncludes` أيضاً.

#### `shouldReturnMateData(): bool`

- **الوصف:** يحدد ما إذا كان سيتم إرجاع بيانات الميتا بشكل عام.
- **آلية العمل:**
  - يتحقق من وجود معامل `is_mate_data_media` في الطلب (Request) ويعيد قيمته.
  - إذا لم يكن موجوداً، يعيد قيمة الإعداد `nano.locationapi::is_mate_data_media` من ملف الإعدادات.
- **الإرجاع:** `true` إذا طلبت بيانات الميتا، وإلا `false`.

#### `getMateDataForInclude(string $includeName): bool`

- **الوصف:** يحدد ما إذا كان سيتم إرجاع بيانات الميتا لكل include على حدة، مع إمكانية تجاوز الإعداد العام.
- **المعاملات:**
  - `$includeName`: اسم الـ include (مثال: `image`, `files`).
- **آلية العمل:**
  - يتحقق من وجود معامل في الطلب بنفس اسم الـ include يحتوي على `mate_data` (مثال: `image[mate_data]=1`).
  - إذا كان موجوداً، يعيد قيمته.
  - وإلا يعيد قيمة `shouldReturnMateData()`.
- **الإرجاع:** `true` إذا طلبت بيانات الميتا لهذا الـ include تحديداً، وإلا `false`.

---

#### دوال الـ Includes الرئيسية

كل دالة من الدوال التالية تمثل include يمكن طلبه عبر معامل `include` في API. جميعها تعيد مورداً من نوع `Primitive` (للمفرد) أو `Collection` (للجماعة).

| الدالة | الـ Include | نوع المورد | العلاقة المتوقعة في النموذج | القيمة الافتراضية عند الخطأ |
| :--- | :--- | :--- | :--- | :--- |
| `includeImage($model)` | `image` | `Primitive` | `attachOne 'image'` | `null` |
| `includeImages($model)` | `images` | `Collection` | `attachMany 'images'` | `[]` |
| `includeVideos($model)` | `videos` | `Collection` | `attachMany 'videos'` | `[]` |
| `includeAudios($model)` | `audios` | `Collection` | `attachMany 'audios'` | `[]` |
| `includeFiles($model)` | `files` | `Collection` | `attachMany 'files'` | `[]` |
| `includeBookIntro($model)` | `book_intro` | `Primitive` | `attachOne 'book_intro'` | `null` |

**آلية عمل كل دالة:**

1. تتحقق من صحة `$model` ووجود الدالة الخاصة بالعلاقة (مثال: `method_exists($model, 'image')`).
2. تستدعي الدالة المناسبة من الـ Transformer الأساسي (`$this->transformer->image()` أو `$this->transformer->images()` أو `$this->transformer->file()` أو `$this->transformer->files()`).
3. تمرر معامل `$isMate` المستخرج من `getMateDataForInclude()`.
4. في حال حدوث أي استثناء (Exception/Throwable)، تسجله (في بيئة التطوير فقط) وتعطي القيمة الافتراضية الآمنة.

---

## التكامل مع Transformers

لاستخدام هذا الـ Behavior في Transformer معين، يتم إرفاقه عبر دالة `extend` كما في المثال التالي:

```php
use Nano\LocationApi\Behaviors\DynamicAddMediaIncludes;

class CountryTransformer extends Transformer
{
    public $implement = [
        DynamicAddMediaIncludes::class,
    ];
    
    // ... باقي الكود
}
```

أو عبر التوسيع الديناميكي داخل `Plugin.php` (كما هو مطبق في `Nano.LocationApi`):

```php
protected function extendLocationApiTransformersWithMedia()
{
    $transformers = [
        \Nano\LocationApi\Transformers\CountryTransformer::class,
        \Nano\LocationApi\Transformers\StateTransformer::class,
        \Nano\LocationApi\Transformers\DirectorateTransformer::class,
    ];

    foreach ($transformers as $transformerClass) {
        $transformerClass::extend(function ($transformer) {
            if (!$transformer->isClassExtendedWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes')) {
                $transformer->extendClassWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes');
            }
        });
    }
}
```

---

## الإعدادات (Configuration)

يمكن تخصيص سلوك الـ Behavior عبر ملف إعدادات الإضافة `config/nano/locationapi/config.php` (يُنشر باستخدام `php artisan config:publish Nano.LocationApi`).

| المفتاح | النوع | القيمة الافتراضية | الوصف |
| :--- | :--- | :--- | :--- |
| `is_mate_data_media` | `bool` | `false` | إرجاع بيانات الميتا افتراضياً عند عدم طلبها صراحة. |
| `default_media_includes` | `array|false` | `false` | إذا كانت مصفوفة، تضاف هذه الـ includes إلى `defaultIncludes` في الـ Transformer. مثال: `['image', 'images']`. |

**مثال لملف الإعدادات:**

```php
<?php

return [
    'is_mate_data_media' => false,
    'default_media_includes' => ['image'], // تضمين الصورة الرئيسية افتراضياً
];
```

---

## أمثلة الاستخدام (Usage Examples)

### 1. طلب دولة مع تضمين الصورة الرئيسية فقط

**الطلب:**
```http
GET /api/v1/countries/1?include=image
```

**الاستجابة:**
```json
{
  "data": {
    "id": 1,
    "name": "Yemen",
    "image": {
      "original": "https://domain.com/storage/flags/ye.png",
      "small": "https://domain.com/storage/flags/ye_small.png"
    }
  }
}
```

### 2. طلب مدينة مع تضمين معرض الصور وبيانات الميتا العامة

**الطلب:**
```http
GET /api/v1/states/5?include=images&is_mate_data_media=1
```

**الاستجابة:**
```json
{
  "data": {
    "id": 5,
    "name": "Sana'a",
    "images": [
      {
        "original": "https://domain.com/storage/gallery/1.jpg",
        "mate_data": {
          "id": 101,
          "title": "Old City",
          "file_name": "1.jpg",
          "file_size": 204800
        }
      }
    ]
  }
}
```

### 3. طلب مديرية مع تضمين فيديوهات وملفات، مع طلب ميتا للملفات فقط

**الطلب:**
```http
GET /api/v1/directorates/10?include=videos,files&files[mate_data]=1
```

**الاستجابة (مقتطف):**
```json
{
  "data": {
    "id": 10,
    "name": "Al-Sabeen",
    "videos": [
      {
        "file_name": "intro.mp4",
        "path": "https://domain.com/storage/videos/intro.mp4"
      }
    ],
    "files": [
      {
        "file_name": "report.pdf",
        "path": "https://domain.com/storage/files/report.pdf",
        "mate_data": {
          "id": 202,
          "title": "Annual Report",
          "file_size": 1048576
        }
      }
    ]
  }
}
```

### 4. طلب دولة مع تضمين جميع الوسائط (استخدام متعدد)

**الطلب:**
```http
GET /api/v1/countries/1?include=image,images,videos,audios,files,book_intro&is_mate_data_media=1
```

---

## التعامل مع الأخطاء (Error Handling)

- **التحقق من وجود الدالة:** قبل استدعاء أي علاقة، يتم التحقق من وجود الدالة في النموذج (`method_exists`). إذا لم تكن موجودة، يتم إرجاع القيمة الافتراضية مباشرة دون محاولة الوصول.
- **معالجة الاستثناءات:** جميع دوال الـ includes مغلفة بـ `try-catch`. في حالة حدوث أي استثناء، يتم تسجيله (في بيئة التطوير فقط) باستخدام `trace_log`، ثم يتم إرجاع القيمة الافتراضية الآمنة.
- **تسجيل الأخطاء:** يتم تسجيل الأخطاء في سجل التتبع (trace log) عند تفعيل `app.debug = true`، مما يساعد في التصحيح أثناء التطوير دون التأثير على المستخدم النهائي.

**مثال على خطأ مسجل:**
```
DynamicAddMediaIncludes::includeImage - Call to undefined method App\Models\Country::image()
```

---

## التوسع والتخصيص (Extensibility)

### إضافة Include جديد

يمكن إضافة include جديد (مثل `documents`) باتباع الخطوات التالية:

1. التأكد من وجود العلاقة (`attachMany 'documents'`) في النموذج.
2. إضافة دالة جديدة في الـ Behavior:

```php
public function includeDocuments($model)
{
    try {
        if (!$model || !method_exists($model, 'documents')) {
            return $this->collection([]);
        }
        $files = $model->documents;
        $isMate = $this->getMateDataForInclude('documents');
        $result = $this->transformer->files($files, $isMate);
        return $this->collection($result);
    } catch (Exception|Throwable $e) {
        $this->logError(__FUNCTION__, $e);
        return $this->collection([]);
    }
}
```

3. إضافة `'documents'` إلى مصفوفة `$mediaIncludes` في المنشئ.

### تجاوز السلوك في Transformer مخصص

يمكنك تجاوز أي من دوال الـ includes في Transformer الخاص بك إذا احتجت إلى تنسيق مختلف:

```php
class MyCountryTransformer extends CountryTransformer
{
    public function includeImage($model)
    {
        // تنسيق مخصص للصورة
        $image = $model->image;
        return $this->primitive([
            'url' => $image ? $image->path : null,
            'caption' => $image ? $image->title : null,
        ]);
    }
}
```

---

## الاعتماديات (Dependencies)

- **Nano.API:** يوفر الـ Transformer الأساسي (`Nano\API\Classes\Transformer`) والدوال المساعدة (`image()`, `images()`, `file()`, `files()`).
- **Nano.Location:** يوفر النماذج الجغرافية التي تحتوي على علاقات الوسائط المضافة.
- **System\Models\File:** نظام إدارة الملفات الأساسي في October CMS.

---

## الاختبار (Testing)

لاختبار الـ Behavior بعد إضافته إلى Transformer معين، يمكن استخدام الكود التالي في بيئة الاختبار:

```php
use Nano\LocationApi\Transformers\CountryTransformer;
use Nano\LocationApi\Behaviors\DynamicAddMediaIncludes;

// إنشاء Transformer مع الـ Behavior
$transformer = new CountryTransformer();

// محاكاة طلب API مع include معين
Input::merge(['include' => 'image', 'is_mate_data_media' => 1]);

// الحصول على نموذج دولة يحتوي على صورة
$country = Country::find(1);

// استدعاء دالة include مباشرة
$behavior = new DynamicAddMediaIncludes($transformer);
$result = $behavior->includeImage($country);

// التحقق من النتيجة
assert($result instanceof Primitive);
assert(!is_null($result->getData()));
```

---

## ملخص (Summary)

يُعد `DynamicAddMediaIncludes` Behavior قوياً ومرناً يسهل دمج الوسائط المتعددة في واجهات برمجة تطبيقات الموقع الجغرافي. بفضل تصميمه المعياري ومعالجته الآمنة للاستثناءات، يمكن الاعتماد عليه في الإنتاج مع ضمان استجابات مستقرة حتى في حالات عدم اكتمال النماذج. يتبع نفس النمط المعماري المستخدم في `DynamicAddIncludeKyc` من إضافة `Nano3.Kyc`، مما يضمن تناسقاً في الشيفرة البرمجية عبر مشاريع نانوسوفت.

---

**المراجع:**
- [توثيق إضافة Nano.LocationApi](./Docs-Nano-LocationApi-ar.md)
- [توثيق إضافة Nano.Location](./Docs-Nano-Location-ar.md)
- [نظام الـ Extendable في October CMS](https://docs.octobercms.com/3.x/extend/classes/extendable.html)
- [توثيق `System\Models\File`](https://docs.octobercms.com/3.x/api/system/file.html)

---

**تاريخ آخر تحديث:** 2025-04-30  
**الإصدار:** 1.1.0  
**المؤلف:** Nano2Soft Team