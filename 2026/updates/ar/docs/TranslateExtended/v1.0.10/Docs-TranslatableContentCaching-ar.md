## توثيق متكامل لسلوك TranslatableContentCaching

### 1. مقدمة

**TranslatableContentCaching** هو سلوك (Behavior) متقدم لنظام NanoSoft System، مصمم لتعزيز وتوسيع وظائف إضافة **RainLab.Translate**. يهدف هذا السلوك إلى تحسين الأداء عبر التخزين المؤقت (Caching) للقيم المترجمة، وتوفير وصول سهل وسريع إلى الترجمات عبر Accessors ديناميكية، وإضافة ميزات متطورة لعرض الترجمات المتعددة في واجهات API.

بينما يوفر سلوك RainLab.Translate الأساسي آلية لتخزين واسترجاع الترجمات، فإن هذا السلوك يبني فوقه طبقة متكاملة تقدم:
- تخزين مؤقت ذكي (Caching) للسمات المترجمة مع إبطاء تلقائي عند التحديث.
- دوال Accessors ديناميكية لأي حقل مترجم، دون الحاجة لكتابة دوال مكررة.
- إمكانية إرجاع جميع ترجمات الحقل الواحد في صيغ متعددة (مناسبة لبناء واجهات API متعددة اللغات).
- دوال مساعدة للحصول على اللغات البديلة للمحتوى مع روابطها.
- دعم كامل لخاصية `translatableUseFallback` من السلوك الأصلي، مع إمكانية التحكم الدقيق في الفولباك.
- التحكم في تمكين/تعطيل الكاش بشكل عام أو لكل استدعاء.
- معالجة الحقول القابلة للتحويل إلى JSON (Jsonable) مع آلية آمنة للتعامل مع الأخطاء.

### 2. الفوائد والميزات الرئيسية

- **تحسين الأداء**: تخزين الترجمات في الكاش يقلل من استعلامات قاعدة البيانات بشكل كبير، خاصة عند تكرار عرض نفس المحتوى المترجم.
- **مرونة عالية**: دعم صيغ متعددة لإخراج الترجمات في API، مما يسمح للمطور باختيار الشكل الأنسب لتطبيقاته (تجميع حسب الحقل، حسب اللغة، أو كليهما).
- **تكامل تام**: يعمل جنباً إلى جنب مع سلوك RainLab.Translate ولا يتعارض معه، بل يوسع وظائفه.
- **توفير الوقت**: Accessors ديناميكية تلغي الحاجة لكتابة دوال getter لكل حقل على حدة.
- **إدارة ذكية للكاش**: مسح تلقائي للكاش عند تحديث أو حذف الموديل، مع إمكانية مسح يدوي عند الحاجة.
- **دعم الروابط البديلة**: دوال مساعدة للحصول على قائمة باللغات المتوفرة للمحتوى الحالي مع روابطها (مفيد لتحسين SEO وتبديل اللغة).
- **تحكم كامل بالفولباك**: إمكانية تعطيل أو تفعيل الفولباك على مستوى الموديل أو داخل الاستدعاءات الفردية.
- **معالجة الحقول JSONable**: دعم فك ترميز الحقول المخزنة بصيغة JSON مع معالجة الأخطاء (try-catch) لمنع تعطل التطبيق.

### 3. طريقة التثبيت والإعداد

#### المتطلبات الأساسية
- NanoSoft System الإصدار 3 أو أعلى (أو الإصدار 2 مع دعم Traits).
- إضافة **RainLab.Translate** مثبتة ومفعلة.
- الموديل المراد ترجمته يجب أن يستخدم سلوك `RainLab.Translate.Behaviors.TranslatableModel`.

#### خطوات التثبيت
1. قم بإنشاء ملف السلوك داخل إضافتك الخاصة، مثلاً:
   `plugins/nano/translateextended/behaviors/TranslatableContentCaching.php`
2. انسخ الكود الكامل للسلوك (آخر إصدار) إلى الملف.
3. في الموديل الذي تريد تعزيزه، أضف السلوك إلى قائمة `$implement`:

```php
class Post extends Model
{
    public $implement = [
        '@RainLab.Translate.Behaviors.TranslatableModel',
        'Nano.TranslateExtended.Behaviors.TranslatableContentCaching'
    ];

    // الحقول القابلة للترجمة (خاصة بـ RainLab.Translate)
    public $translatable = ['title', 'content', 'slug'];

    // حقول إضافية نريد تخزينها مؤقتاً (مثل حقول JSON أو علاقات)
    public $translatableCaching = ['images', 'seo'];
}
```

### 4. الخصائص القابلة للتخصيص في الموديل

يمكنك تخصيص سلوك الترجمة عبر إضافة الخصائص التالية إلى الموديل الخاص بك. جميع أسماء الخصائص تبدأ بـ `translatable` لتجنب التعارض مع السلوكيات الأخرى أو خصائص الموديل.

| الخاصية | النوع | الوصف | القيمة الافتراضية |
|---------|-------|-------|-------------------|
| `$translatableCaching` | array | حقول إضافية (غير موجودة في `$translatable`) تريد تخزين ترجماتها مؤقتاً. | `[]` |
| `$translatableApiFields` | array | الحقول التي تريد عرض ترجماتها الكاملة في API (عبر `toArrayWithApiTranslations`). | `[]` |
| `$translatableApiFormat` | string | تنسيق إخراج الترجمات المتعددة (انظر القسم 5). | `'group_by_field'` |
| `$translatableApiGroupKey` | string | اسم المفتاح الجامع في التنسيقات التي تستخدم كائنًا واحدًا. | `'allTranslate'` |
| `$translatableCacheDuration` | int | مدة التخزين المؤقت بالساعات. | `1` |
| `$translatableUseFallback` | bool | يتحكم في استخدام القيمة الأصلية عند عدم وجود ترجمة (يعادل `translatableUseFallback` في السلوك الأصلي). | `true` |
| `$translatableCacheEnabled` | bool | يتحكم في تمكين الكاش بشكل عام. | `true` |

### 5. التنسيقات المدعومة لإخراج الترجمات في API

يوفر السلوك 6 تنسيقات مختلفة للتحكم في شكل البيانات المرتجعة. يمكنك تحديد التنسيق المطلوب عبر الخاصية `$translatableApiFormat`.

#### القيم المدعومة (استخدم النصوص مباشرة)

| القيمة | الوصف |
|--------|-------|
| `'group_by_field'` | لكل حقل، يضيف حقل جديد باسم `[field]_all` يحتوي على كائن بجميع الترجمات. |
| `'array'` | لكل حقل، يضيف حقل `[field]_all` كمصفوفة من الكائنات `{locale, value}`. |
| `'both'` | يضيف كلا التنسيقين السابقين بمفاتيح `[field]_all_object` و `[field]_all_array`. |
| `'group_by_locale'` | يضيف كائنًا واحدًا باسم `allTranslate` (قابل للتخصيص) يجمع الترجمات حسب اللغة. |
| `'group_by_field_under_all'` | يضيف كائن `allTranslate` يجمع الترجمات حسب الحقل. |
| `'both_grouped'` | يضيف كائن `allTranslate` (حسب الحقل) بالإضافة إلى الحقول الفردية `[field]_all`. |

#### أمثلة توضيحية مع المخرجات

لنفترض أن لدينا موديل `Product` يحتوي على الحقول المترجمة `name` و `description`، والقيم كالتالي:
- القيمة الأصلية (x): `"منتج 1"` و `"وصف المنتج"`
- الترجمة العربية (ar): `"منتج 1 بالعربية"` و `"وصف المنتج بالعربية"`
- الترجمة الإنجليزية (en): `"Product 1"` و `"Product description"`

وعند استدعاء `$product->toArrayWithApiTranslations()` نحصل على المخرجات التالية حسب كل تنسيق:

---

##### أ) `'group_by_field'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all": {
        "x": "منتج 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "description_all": {
        "x": "وصف المنتج",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    }
}
```

##### ب) `'array'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all": [
        { "locale": "x", "value": "منتج 1" },
        { "locale": "ar", "value": "منتج 1 بالعربية" },
        { "locale": "en", "value": "Product 1" }
    ],
    "description_all": [
        { "locale": "x", "value": "وصف المنتج" },
        { "locale": "ar", "value": "وصف المنتج بالعربية" },
        { "locale": "en", "value": "Product description" }
    ]
}
```

##### ج) `'both'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all_object": {
        "x": "منتج 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "name_all_array": [
        { "locale": "x", "value": "منتج 1" },
        { "locale": "ar", "value": "منتج 1 بالعربية" },
        { "locale": "en", "value": "Product 1" }
    ],
    "description_all_object": {
        "x": "وصف المنتج",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    },
    "description_all_array": [
        { "locale": "x", "value": "وصف المنتج" },
        { "locale": "ar", "value": "وصف المنتج بالعربية" },
        { "locale": "en", "value": "Product description" }
    ]
}
```

##### د) `'group_by_locale'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "allTranslate": {
        "x": {
            "name": "منتج 1",
            "description": "وصف المنتج"
        },
        "ar": {
            "name": "منتج 1 بالعربية",
            "description": "وصف المنتج بالعربية"
        },
        "en": {
            "name": "Product 1",
            "description": "Product description"
        }
    }
}
```

##### هـ) `'group_by_field_under_all'`
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "allTranslate": {
        "name": {
            "x": "منتج 1",
            "ar": "منتج 1 بالعربية",
            "en": "Product 1"
        },
        "description": {
            "x": "وصف المنتج",
            "ar": "وصف المنتج بالعربية",
            "en": "Product description"
        }
    }
}
```

##### و) `'both_grouped'`
يجمع بين التنسيق `group_by_field_under_all` والحقول الفردية `[field]_all` (بنفس تنسيق `group_by_field`).
```json
{
    "id": 1,
    "name": "منتج 1",
    "description": "وصف المنتج",
    "name_all": {
        "x": "منتج 1",
        "ar": "منتج 1 بالعربية",
        "en": "Product 1"
    },
    "description_all": {
        "x": "وصف المنتج",
        "ar": "وصف المنتج بالعربية",
        "en": "Product description"
    },
    "allTranslate": {
        "name": {
            "x": "منتج 1",
            "ar": "منتج 1 بالعربية",
            "en": "Product 1"
        },
        "description": {
            "x": "وصف المنتج",
            "ar": "وصف المنتج بالعربية",
            "en": "Product description"
        }
    }
}
```

### 6. طريقة الاستخدام الأساسية

#### أ) الوصول إلى القيم المترجمة في القوالب أو الكود

عند إضافة السلوك إلى الموديل، يمكنك الوصول إلى أي حقل مترجم مباشرة باستخدام الخاصية العادية. على سبيل المثال، في قالب Twig:

```twig
{{ post.title }} {# سيعيد القيمة المترجمة حسب اللغة الحالية #}
{{ post.content }}
```

لا تحتاج لاستدعاء أي دوال إضافية؛ السلوك ينشئ Accessors ديناميكية لكل حقل مترجم.

#### ب) الحصول على جميع ترجمات حقل معين

يمكنك استخدام الدالة `getAllFieldTranslations` لجلب جميع ترجمات حقل محدد (بما فيها اللغة الافتراضية) مع إمكانية التحكم في الفولباك واستخدام الكاش:

```php
$allTitles = $post->getAllFieldTranslations(
    'title',          // الحقل
    true,             // includeOriginal (يشمل القيمة الأصلية)
    ['ar', 'en'],     // اللغات المطلوبة
    true,             // useFallback (استخدام الفولباك)
    false             // useCache (عدم استخدام الكاش)
);
// النتيجة: ['x' => 'العنوان الأصلي', 'ar' => 'العنوان بالعربية', 'en' => 'Title in English']
```

#### ج) استخدام API مع تنسيقات متعددة

في المتحكم (Controller) الخاص بك، يمكنك استخدام الدالة `toArrayWithApiTranslations` للحصول على مصفوفة جاهزة للإرجاع كـ JSON مع تمرير باراميترات التحكم:

```php
public function show($id)
{
    $post = Post::find($id);
    return response()->json(
        $post->toArrayWithApiTranslations(
            'ar',        // اللغة
            true,        // useFallback
            false        // useCache (بدون كاش لهذا الاستدعاء)
        )
    );
}
```

إذا أردت تمرير لغة محددة فقط:

```php
$data = $post->toArrayWithApiTranslations('ar'); // يجلب البيانات مع ترجمات API بالعربية
```

#### د) الحصول على اللغات البديلة للمحتوى

يمكن استخدام `getAlternateLocales` للحصول على قائمة باللغات المتوفرة لهذا المحتوى، مع روابطها (إذا وفرت دالة لتوليد الرابط):

```php
$locales = $post->getAlternateLocales(function($locale) {
    return Url::to('post/' . $post->slug . '?lang=' . $locale);
});
```

النتيجة:
```php
[
    ['code' => 'en', 'url' => 'https://example.com/post/my-post?lang=en', 'default' => true],
    ['code' => 'ar', 'url' => 'https://example.com/post/my-post?lang=ar'],
    ['code' => 'fr', 'url' => 'https://example.com/post/my-post?lang=fr'],
]
```

### 7. دوال إضافية مفيدة (محدثة)

| الدالة | الوصف |
|--------|-------|
| `getTranslated($attribute, $default = null, $locale = null, $fallback = null, $useCache = null, $useInBackend = false)` | تجلب قيمة مترجمة لحقل معين. يمكن التحكم في الفولباك واستخدام الكاش وإجبار الترجمة في لوحة التحكم عبر `$useInBackend`. |
| `getTranslatedAttributes($locale, $useCache = null)` | تجلب جميع السمات المترجمة للغة معينة من الكاش (أو مباشرة). |
| `getAllFieldTranslations($field, $includeDefault = true, $locales = null, $useFallback = null, $useCache = null)` | تجلب جميع ترجمات حقل معين لكل اللغات المطلوبة، مع خيارات الفولباك والكاش. |
| `getTranslationsInFormat($fields = null, $format = null, $locales = null, $includeOriginal = true, $useFallback = null, $useCache = null)` | دالة عامة مرنة تتيح استدعاء الترجمات بأي تنسيق مع تحديد الحقول واللغات والتحكم بالفولباك والكاش. |
| `addApiTranslations(&$array, $locale = null, $useFallback = null, $useCache = null)` | تضيف ترجمات API إلى مصفوفة موجودة (تستخدم داخلياً). |
| `toArrayWithApiTranslations($locale = null, $useFallback = null, $useCache = null)` | ترجع مصفوفة الموديل مع إضافة ترجمات API حسب التنسيق المحدد. |
| `invalidateAllCaches()` | تمسح جميع أنواع الكاش لهذا الموديل (تُستدعى تلقائياً عند الحفظ/الحذف). |
| `enableTranslatableCache()` / `disableTranslatableCache()` | لتمكين أو تعطيل الكاش ديناميكياً. |
| `isTranslatableCacheEnabled()` | للتحقق من حالة الكاش. |
| `setUseFallback($useFallback)` / `getUseFallback()` | لضبط أو قراءة حالة الفولباك الحالية. |
| `noFallbackLocale()` / `withFallbackLocale()` | لتعطيل أو تفعيل الفولباك بشكل مؤقت (قابلة للتسلسل). |

### 8. إدارة الكاش المتقدمة

#### إبطال كاش لغة محددة
لا يوفر السلوك دالة عامة مخصصة لإبطال كاش لغة معينة، ولكن يمكنك القيام بذلك يدوياً عبر بناء مفتاح الكاش واستخدام `Cache::forget()`.

يُبنى مفتاح الكاش للسمات المترجمة للغة محددة كالتالي:
```
{ClassName}_{ModelId}_attributes_{locale}
```
ثم يُمرر عبر `md5()`. مثال:

```php
use Cache;

$cacheKey = md5(get_class($post) . '_' . $post->id . '_attributes_ar');
Cache::forget($cacheKey);
```

أما بالنسبة لكاش الحقل المحدد (من `getAllFieldTranslations`)، فإن النمط أصبح أكثر تفصيلاً ليشمل معلومات الفولباك واللغات المطلوبة:
```
{ClassName}_{ModelId}_field_{fieldName}_{localesString}_fb{fallback}
```
حيث `localesString` هي ربط رموز اللغات بشرطة سفلية، و`fb` تشير إلى حالة الفولباك (`default`، `1`، `0`). مثال لكاش يحتوي على اللغات `ar_en` مع تفعيل الفولباك:
```php
$cacheKey = md5(get_class($post) . '_' . $post->id . '_field_title_ar_en_fb1');
Cache::forget($cacheKey);
```
إذا كنت تريد مسح جميع أشكال الكاش لحقل معين، يمكنك مسح المفتاح الأساسي دون إضافات:
```php
$cacheKeyBase = md5(get_class($post) . '_' . $post->id . '_field_title');
Cache::forget($cacheKeyBase); // هذا قد لا يمسح جميع المتغيرات، لذا يفضل مسح المفاتيح بدقة.
```

لتسهيل الأمر، يمكنك استدعاء `invalidateAllCaches()` الذي يمسح كل الكاش المرتبط بهذا الموديل.

### 9. مثال متكامل لموديل مع جميع الإعدادات

```php
<?php namespace Acme\Blog\Models;

use Model;

class Product extends Model
{
    public $implement = [
        '@RainLab.Translate.Behaviors.TranslatableModel',
        'Nano.TranslateExtended.Behaviors.TranslatableContentCaching'
    ];

    public $translatable = [
        'name',
        'description',
        'meta_title',
        'meta_description'
    ];

    // حقول إضافية للتخزين المؤقت (مثل JSON)
    public $translatableCaching = [
        'images',
        'specs'
    ];

    // حقول API التي نريد ترجماتها الكاملة
    public $translatableApiFields = [
        'name',
        'description',
        'meta_title',
        'meta_description'
    ];

    // تنسيق API
    public $translatableApiFormat = 'group_by_field_under_all';

    // مفتاح مخصص للكائن الجامع
    public $translatableApiGroupKey = 'translations';

    // مدة الكاش (ساعتان)
    public $translatableCacheDuration = 2;

    // تعطيل الفولباك (اختياري)
    public $translatableUseFallback = false;

    // تمكين الكاش (افتراضي true)
    public $translatableCacheEnabled = true;

    // ... باقي تعريفات الموديل (جداول، علاقات، إلخ)
}
```

### 10. ملاحظات مهمة

- **الترجمة في لوحة التحكم**: يتم تعطيل الترجمة تلقائياً في واجهة الإدارة (backend) لتجنب التداخل أثناء التحرير، إلا إذا تم تمرير `$useInBackend = true`.
- **التوافق مع الإضافات الأخرى**: السلوك لا يتعارض مع أي إضافات أخرى، ويمكن استخدامه مع أي موديل يستخدم RainLab.Translate. جميع الخصائص والدوال الداخلية مسبوقة بـ `trans` لضمان عدم التعارض.
- **تفرد مفتاح الكاش**: يتم إنشاء مفتاح الكاش بناءً على اسم الكلاس ورقم الموديل ونوع الكاش، مما يضمن عدم حدوث تعارض بين موديلات مختلفة.
- **تحديث الكاش التلقائي**: عند حفظ أو حذف الموديل، يتم مسح جميع مفاتيح الكاش المرتبطة به تلقائياً لضمان تحديث البيانات.
- **معالجة الحقول JSONable**: يتم فك ترميز JSON تلقائياً مع معالجة الأخطاء (try-catch)، مما يمنع تعطل التطبيق في حالة وجود JSON غير صالح.

### 11. الدالة `getTranslationsInFormat` (محدثة)

تم تحسين الدالة العامة لتستقبل باراميترات إضافية للتحكم في الفولباك والكاش.

#### التوقيع:
```php
public function getTranslationsInFormat(
    $fields = null,
    $format = null,
    $locales = null,
    $includeOriginal = true,
    $useFallback = null,
    $useCache = null
)
```

#### المعاملات:
- `$fields`: مصفوفة من أسماء الحقل. إذا كانت `null`، يتم استخدام كل الحقول المترجمة (من `$translatableCaching` و `$translatable`).
- `$format`: التنسيق المطلوب. يمكن أن يكون إحدى القيم: `'group_by_field'`, `'array'`, `'both'`, `'group_by_locale'`, `'group_by_field_under_all'`, `'both_grouped'`. إذا كانت `null`، يتم استخدام التنسيق الافتراضي للموديل (`$translatableApiFormat`).
- `$locales`: مصفوفة من أكواد اللغات (مثل `['ar', 'en']`). إذا كانت `null`، يتم استخدام كل اللغات النشطة.
- `$includeOriginal`: منطقي، إذا كان `true` يتم تضمين القيمة الأصلية بمفتاح `'x'`.
- `$useFallback`: منطقي، يتحكم في استخدام الفولباك (`null` لاستخدام الإعداد العام).
- `$useCache`: منطقي، يتحكم في استخدام الكاش (`null` لاستخدام الإعداد العام).

#### قيمة الإرجاع:
مصفوفة تحتوي على الترجمات بالتنسيق المطلوب.

#### أمثلة على الاستخدام:

```php
// استخدام كل الإعدادات الافتراضية (كل الحقول، التنسيق الافتراضي، كل اللغات)
$translations = $product->getTranslationsInFormat();

// تحديد حقل واحد وتنسيق group_by_field مع تعطيل الفولباك
$translations = $product->getTranslationsInFormat(
    ['name'],
    'group_by_field',
    null,
    true,
    false,   // useFallback
    true     // useCache
);

// تحديد حقول متعددة وتنسيق group_by_locale مع لغات محددة، بدون كاش
$translations = $product->getTranslationsInFormat(
    ['name', 'description'],
    'group_by_locale',
    ['ar', 'en'],
    true,    // includeOriginal
    true,    // useFallback
    false    // useCache
);

// الحصول على ترجمات كل الحقول بتنسيق array بدون القيمة الأصلية
$translations = $product->getTranslationsInFormat(null, 'array', null, false);
```

#### مثال على الناتج (للاستدعاء بـ `['name', 'description'], 'group_by_field_under_all', ['ar', 'en']` مع `includeOriginal = true`):
```php
[
    'allTranslate' => [
        'name' => [
            'x' => 'منتج 1',
            'ar' => 'منتج 1 بالعربية',
            'en' => 'Product 1'
        ],
        'description' => [
            'x' => 'وصف المنتج',
            'ar' => 'وصف المنتج بالعربية',
            'en' => 'Product description'
        ]
    ]
]
```

### 12. أمثلة إضافية على التحكم في الفولباك والكاش

#### أ) استدعاء دالة `getTranslated` مع تعطيل الفولباك مؤقتاً

```php
$value = $product->getTranslated('name', null, 'ar', false); // لا يستخدم الفولباك حتى لو كان مفعلاً
```

#### ب) تعطيل الكاش في استدعاء معين

```php
$attributes = $product->getTranslatedAttributes('en', false); // يجلب من قاعدة البيانات مباشرة
```

#### ج) استخدام `getAllFieldTranslations` مع تحديد الفولباك والكاش

```php
$allNames = $product->getAllFieldTranslations(
    'name',
    true,
    ['ar', 'en', 'fr'],
    true,   // useFallback
    false   // useCache (بدون كاش)
);
```

#### د) التحقق من حالة الكاش والفولباك ديناميكياً

```php
// تعطيل الكاش مؤقتاً لكل الاستدعاءات اللاحقة لهذا الموديل
$product->disableTranslatableCache();

// تفعيل الفولباك
$product->withFallbackLocale();

// استدعاء دالة ستتأثر بهذه الإعدادات
$data = $product->toArrayWithApiTranslations();
```

---

بهذا نكون قد وثقنا جميع جوانب السلوك `TranslatableContentCaching` مع آخر التحديثات. يمكنك الآن استخدامه بثقة في مشاريع NanoSoft System متعددة اللغات.