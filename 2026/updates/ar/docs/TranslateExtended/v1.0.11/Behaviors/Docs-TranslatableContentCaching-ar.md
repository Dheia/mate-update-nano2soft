## توثيق متكامل لسلوك TranslatableContentCaching (الإصدار المُحسَّن)

### 1. مقدمة

**TranslatableContentCaching** هو سلوك (Behavior) متقدم لنظام OctoberCMS / WinterCMS، مصمم لتعزيز وتوسيع وظائف إضافة **RainLab.Translate**. يهدف هذا السلوك إلى تحسين الأداء عبر التخزين المؤقت (Caching) للقيم المترجمة، وتوفير وصول سهل وسريع إلى الترجمات عبر Accessors ديناميكية، وإضافة ميزات متطورة لعرض الترجمات المتعددة في واجهات API.

بينما يوفر سلوك RainLab.Translate الأساسي آلية لتخزين واسترجاع الترجمات، فإن هذا السلوك يبني فوقه طبقة متكاملة تقدم:

- تخزين مؤقت ذكي (Caching) للسمات المترجمة مع إبطاء تلقائي عند التحديث.
- دوال Accessors ديناميكية لأي حقل مترجم، دون الحاجة لكتابة دوال مكررة.
- إمكانية إرجاع جميع ترجمات الحقل الواحد في صيغ متعددة (مناسبة لبناء واجهات API متعددة اللغات).
- دوال مساعدة للحصول على اللغات البديلة للمحتوى مع روابطها.
- دعم كامل لخاصية `translatableUseFallback` من السلوك الأصلي، مع إمكانية التحكم الدقيق في الفولباك.
- التحكم في تمكين/تعطيل الكاش بشكل عام أو لكل استدعاء.
- معالجة الحقول القابلة للتحويل إلى JSON (Jsonable) مع آلية آمنة للتعامل مع الأخطاء.
- إمكانية تعيين الحقول القابلة للترجمة ديناميكيًا بعد تهيئة السلوك.
- دعم تمرير أسماء الحقول كسلسلة نصية مفصولة بفواصل (`'title,content'`) أو كلمة `'all'`/`'*'` في جميع الدوال التي تقبل مصفوفة حقول.

### 2. الفوائد والميزات الرئيسية

- **تحسين الأداء**: تخزين الترجمات في الكاش يقلل من استعلامات قاعدة البيانات بشكل كبير، خاصة عند تكرار عرض نفس المحتوى المترجم.
- **مرونة عالية**: دعم صيغ متعددة لإخراج الترجمات في API، مما يسمح للمطور باختيار الشكل الأنسب لتطبيقاته (تجميع حسب الحقل، حسب اللغة، أو كليهما).
- **تكامل تام**: يعمل جنباً إلى جنب مع سلوك RainLab.Translate ولا يتعارض معه، بل يوسع وظائفه.
- **توفير الوقت**: Accessors ديناميكية تلغي الحاجة لكتابة دوال getter لكل حقل على حدة.
- **إدارة ذكية للكاش**: مسح تلقائي للكاش عند تحديث أو حذف الموديل، مع إمكانية مسح يدوي عند الحاجة.
- **دعم الروابط البديلة**: دوال مساعدة للحصول على قائمة باللغات المتوفرة للمحتوى الحالي مع روابطها (مفيد لتحسين SEO وتبديل اللغة).
- **تحكم كامل بالفولباك**: إمكانية تعطيل أو تفعيل الفولباك على مستوى الموديل أو داخل الاستدعاءات الفردية.
- **معالجة الحقول JSONable**: دعم فك ترميز الحقول المخزنة بصيغة JSON مع معالجة الأخطاء (try-catch) لمنع تعطل التطبيق، مع تسجيل الأخطاء في وضع التصحيح.
- **تحكم ديناميكي بالحقول القابلة للترجمة**: إمكانية تعديل قائمة الحقول (`transCachedFields`) بعد تهيئة السلوك باستخدام `getTransCachedFields` و `setTransCachedFields`.
- **تمرير مرن للحقول**: قبول الحقول كسلسلة مفصولة بفواصل (`'title,content'`) أو كمصفوفة أو كقيمة `'all'`/`'*'` في دوال مثل `setTransCachedFields` و `getTranslationsInFormat`.

### 3. طريقة التثبيت والإعداد

#### المتطلبات الأساسية
- OctoberCMS أو WinterCMS الإصدار 2 أو أعلى.
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
| `getTranslated($attribute, $default = null, $locale = null, $fallback = null, $useCache = null, $useInBackend = false)` | تجلب قيمة مترجمة لحقل معين. يمكن التحكم في الفولباك واستخدام الكاش وإجبار الترجمة في لوحة التحكم عبر `$useInBackend`. تم تحسينها لتعيين `$fallback = true` تلقائياً عند طلب اللغة الافتراضية لضمان الرجوع إلى القيمة الأصلية. |
| `getTranslatedAttributes($locale, $useCache = null)` | تجلب جميع السمات المترجمة للغة معينة من الكاش (أو مباشرة). تم تحسين التوافق مع هيكل `RainLab.Translate` حيث تبحث في `$item->attributes['locale']` أولاً ثم في `$item->locale`. |
| `getAllFieldTranslations($field, $includeDefault = true, $locales = null, $useFallback = null, $useCache = null)` | تجلب جميع ترجمات حقل معين لكل اللغات المطلوبة، مع خيارات الفولباك والكاش. |
| `getTranslationsInFormat($fields = null, $format = null, $locales = null, $includeOriginal = true, $useFallback = null, $useCache = null)` | دالة عامة مرنة تتيح استدعاء الترجمات بأي تنسيق مع تحديد الحقول واللغات والتحكم بالفولباك والكاش. **تم تحسينها لدعم تمرير الحقول كسلسلة نصية مفصولة بفواصل (`'title,content'`) أو كلمة `'all'`/`'*'`.** |
| `addApiTranslations(&$array, $locale = null, $useFallback = null, $useCache = null)` | تضيف ترجمات API إلى مصفوفة موجودة (تستخدم داخلياً). |
| `toArrayWithApiTranslations($locale = null, $useFallback = null, $useCache = null)` | ترجع مصفوفة الموديل مع إضافة ترجمات API حسب التنسيق المحدد. |
| `invalidateAllCaches()` | تمسح جميع أنواع الكاش لهذا الموديل (تُستدعى تلقائياً عند الحفظ/الحذف). |
| `enableTranslatableCache()` / `disableTranslatableCache()` | لتمكين أو تعطيل الكاش ديناميكياً. |
| `isTranslatableCacheEnabled()` | للتحقق من حالة الكاش. |
| `setUseFallback($useFallback)` / `getUseFallback()` | لضبط أو قراءة حالة الفولباك الحالية. |
| `noFallbackLocale()` / `withFallbackLocale()` | لتعطيل أو تفعيل الفولباك بشكل مؤقت (قابلة للتسلسل). |
| `getTransCachedFields()` | إرجاع قائمة الحقول القابلة للترجمة الحالية (المدمجة من `$translatable` و `$translatableCaching`). |
| `setTransCachedFields($fields = [])` | تعيين قائمة الحقول القابلة للترجمة ديناميكياً. تقبل مصفوفة أو سلسلة مفصولة بفواصل أو `'all'`/`'*'`. |
| `transCollectFields()` | تقوم بجمع الحقول القابلة للترجمة من الموديل (تُستدعى تلقائياً في المُنشئ، ولكن يمكن استدعاؤها يدوياً بعد تغيير الخصائص ديناميكياً). |
| `processTranslatableJsonableValue($attribute, $value)` | دالة مساعدة لفك ترميز JSON للحقول القابلة للتحويل إلى JSON مع تسجيل الأخطاء في وضع التصحيح. |

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

### 10. التحكم الديناميكي بالحقول القابلة للترجمة

في بعض السيناريوهات، قد تحتاج إلى تعديل قائمة الحقول القابلة للترجمة بعد تهيئة السلوك (مثلاً بناءً على سياق معين). يوفر السلوك دالتين للتعامل مع هذا:

#### `getTransCachedFields()`
إرجاع المصفوفة الحالية للحقول القابلة للترجمة (التي تم جمعها من `$translatable` و `$translatableCaching`).

#### `setTransCachedFields($fields)`
تعيين قائمة جديدة للحقول القابلة للترجمة. تقبل الدالة:
- مصفوفة من أسماء الحقول.
- سلسلة نصية مفصولة بفواصل (مثل `'title,content'`).
- القيمة `'all'` أو `'*'` لجلب جميع الحقول القابلة للترجمة تلقائياً (باستدعاء `transCollectFields`).
- قيمة فارغة (ستقوم باستدعاء `transCollectFields` أيضاً).

**مثال:**
```php
$product = Product::find(1);
// تعيين حقول محددة ديناميكياً
$product->setTransCachedFields('name,description,specs');
// الآن سيتم التعامل مع هذه الحقول فقط في عمليات الترجمة
$translations = $product->getTranslationsInFormat();
```

### 11. تحسينات معالجة JSON والحقول القابلة للتحويل

تم إضافة دالة مساعدة `processTranslatableJsonableValue` التي تقوم بفك ترميز القيم المخزنة بصيغة JSON إذا كان الحقل معرفاً كـ `jsonable` في الموديل (عبر `$jsonable` property أو دالة `isJsonable`). في حالة وجود خطأ في JSON (مثل تنسيق غير صالح)، يتم تسجيل الخطأ في سجل التتبع (`trace_log`) فقط عند تفعيل وضع التصحيح (`app.debug = true`) دون أن يتسبب في تعطل التطبيق.

هذا يضمن التعامل الآمن مع البيانات المخزنة بصيغة JSON مثل `images` أو `specs` أو أي حقول مرنة.

### 12. الدالة `getTranslationsInFormat` (محدثة)

تم تحسين الدالة العامة لتستقبل باراميترات إضافية للتحكم في الفولباك والكاش، وتدعم الآن تمرير الحقول كسلسلة نصية مفصولة بفواصل.

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
- `$fields`: مصفوفة من أسماء الحقل، أو سلسلة مفصولة بفواصل (`'title,content'`)، أو `'all'`/`'*'`، أو `null` (يستخدم كل الحقول المترجمة من `transCachedFields`).
- `$format`: التنسيق المطلوب. إذا كانت `null`، يتم استخدام التنسيق الافتراضي للموديل (`$translatableApiFormat`).
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

// تحديد حقل واحد بتنسيق group_by_field مع تعطيل الفولباك
$translations = $product->getTranslationsInFormat(
    'name',
    'group_by_field',
    null,
    true,
    false,   // useFallback
    true     // useCache
);

// تحديد حقول متعددة كسلسلة نصية، تنسيق group_by_locale مع لغات محددة، بدون كاش
$translations = $product->getTranslationsInFormat(
    'name,description',
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

### 13. أمثلة إضافية على التحكم في الفولباك والكاش

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

#### هـ) فرض الترجمة في بيئة الباكند (لوحة التحكم)

عادةً، يتم تعطيل الترجمة تلقائياً في لوحة التحكم لتجنب التداخل أثناء التحرير. لكن يمكنك تجاوز هذا السلوك باستخدام المعامل `$useInBackend = true`:

```php
// حتى لو كان الكود يُنفذ في backend، سيتم إرجاع القيمة المترجمة
$value = $product->getTranslated('title', null, 'ar', true, true, true);
```

### 14. ملاحظات مهمة

- **الترجمة في لوحة التحكم**: يتم تعطيل الترجمة تلقائياً في واجهة الإدارة (backend) لتجنب التداخل أثناء التحرير، إلا إذا تم تمرير `$useInBackend = true`.
- **التوافق مع الإضافات الأخرى**: السلوك لا يتعارض مع أي إضافات أخرى، ويمكن استخدامه مع أي موديل يستخدم RainLab.Translate. جميع الخصائص والدوال الداخلية مسبوقة بـ `trans` لضمان عدم التعارض.
- **تفرد مفتاح الكاش**: يتم إنشاء مفتاح الكاش بناءً على اسم الكلاس ورقم الموديل ونوع الكاش، مما يضمن عدم حدوث تعارض بين موديلات مختلفة.
- **تحديث الكاش التلقائي**: عند حفظ أو حذف الموديل، يتم مسح جميع مفاتيح الكاش المرتبطة به تلقائياً لضمان تحديث البيانات.
- **معالجة الحقول JSONable**: يتم فك ترميز JSON تلقائياً مع معالجة الأخطاء (try-catch)، مما يمنع تعطل التطبيق في حالة وجود JSON غير صالح.
- **تحسين الفولباك للغة الافتراضية**: تم تعديل دالة `getTranslated` لتعيين `$fallback = true` تلقائياً عندما تكون اللغة المطلوبة هي اللغة الافتراضية، مما يضمن الرجوع إلى القيمة الأصلية المخزنة في `attributes` إذا لم توجد ترجمة. هذا يعالج سلوكاً غير متوقع في الإصدارات السابقة.
- **توافق هيكل البيانات مع RainLab.Translate**: تم تحسين `getTranslatedAttributes` للبحث عن الترجمة باستخدام `$item->attributes['locale']` أولاً (الطريقة التي يخزن بها RainLab.Translate اللغة) ثم `$item->locale` كخطة احتياطية. هذا يضمن التوافق مع الإصدارات المختلفة من الإضافة.

### 15. تسجيل الموديلات ديناميكياً (دالة `registerModel`)

يوفر السلوك دالة ثابتة `registerModel` تسمح بتسجيل أي موديل لاستخدام هذا السلوك دون الحاجة إلى تعديل الكود الأصلي للموديل. يمكن استدعاؤها من دالة `boot` في ملف `Plugin.php` الخاص بإضافتك.

#### التوقيع:
```php
public static function registerModel($modelClass, array $dynamicProperties = [])
```

#### المعاملات:
- `$modelClass`: اسم الكلاس المؤهل بالكامل للموديل.
- `$dynamicProperties`: مصفوفة من الخصائص الديناميكية التي تريد إضافتها إلى الموديل (مثل `translatableApiFields`, `translatableApiFormat`، إلخ).

#### مثال:
```php
// في Plugin.php
public function boot()
{
    \Nano\TranslateExtended\Behaviors\TranslatableContentCaching::registerModel(
        \Tss\Inventory\Models\Product::class,
        [
            'translatableApiFormat' => 'group_by_field_under_all',
            'translatableApiGroupKey' => 'translations',
            'translatableCacheDuration' => 2
        ]
    );
}
```

**التحسينات في الإصدار الحالي:**
- تتحقق الدالة من أن السلوك لم يُضف مسبقاً للموديل (`$model->isClassExtendedWith($behaviorClass)`) قبل محاولة إضافته.
- تتحقق من وجود الخصائص الديناميكية قبل إضافتها باستخدام دالة `isPropertyExists` المساعدة، مما يمنع إعادة تعريف الخصائص الموجودة مسبقاً.
