# توثيق إضافة Nano.TranslateExtended

## 1. مقدمة

**Nano.TranslateExtended** هي إضافة متقدمة لنظام **OctoberCMS / WinterCMS**، تهدف إلى توسيع وتعزيز وظائف إضافة **RainLab.Translate** الشهيرة. تقدم الإضافة مجموعة من الميزات التي تحسن تجربة المستخدم والمطور في المواقع متعددة اللغات، مع التركيز على:

- كشف لغة المتصفح وتحديد اللغة تلقائياً.
- تحكم مرن ببادئات اللغة في روابط URL.
- تخزين مؤقت (Caching) ذكي للقيم المترجمة لتحسين الأداء.
- دعم متقدم لإخراج الترجمات في واجهات API (بما في ذلك تضمين `translatable_fields` مع إمكانية تحديد الحقول واللغات).
- أدوات مساعدة للحصول على اللغات البديلة للمحتوى ومعالجة الحقول القابلة للتحويل إلى JSON.

تم بناء الإضافة لتكون متوافقة تماماً مع `RainLab.Translate` وتعمل كمكمل له، دون تعارض أو حاجة لتعديل الكود الأصلي للإضافة.

---

## 2. المتطلبات الأساسية

- OctoberCMS أو WinterCMS الإصدار 2.x أو أعلى.
- إضافة **RainLab.Translate** مثبتة ومفعلة.
- PHP 7.2+ (يفضل 7.4 أو 8.x).

---

## 3. التثبيت

### عبر سوق الإضافات (Marketplace)
1. افتح لوحة التحكم (Backend) → الإعدادات → تحديث النظام.
2. ابحث عن `Nano.TranslateExtended`.
3. انقر تثبيت.

### عبر سطر الأوامر
```bash
php artisan plugin:install Nano.TranslateExtended
```

### يدوياً
1. قم بتنزيل الكود المصدري من [GitHub](https://github.com/nano/wn-translate-extended).
2. فك الضغط في مجلد `plugins/nano/translateextended`.
3. قم بتشغيل `php artisan plugin:refresh Nano.TranslateExtended`.

بعد التثبيت، تأكد من أن إضافة `RainLab.Translate` مفعلة وأن لديك لغات محددة في `rainlab.translate::locales`.

---

## 4. الإعدادات

### 4.1. إعدادات لوحة التحكم (Backend Settings)

بعد التثبيت، ستجد صفحة إعدادات جديدة ضمن **RainLab.Translate** → **Translate Extended** (أو من القائمة الجانبية: الإعدادات → Translate Extended).

الحقول المتاحة:

| الحقل | الوصف | الافتراضي |
|-------|-------|------------|
| **Browser Language Detection** | تفعيل كشف لغة المتصفح وتحديد اللغة تلقائياً. | `true` |
| **Prefer User Session** | إذا تم التفعيل، تكون اللغة المخزنة في جلسة المستخدم لها أولوية على اللغة المكتشفة من المتصفح. | `true` |
| **Query Parameter** | اسم معامل الاستعلام (مثل `lang`) الذي يمكن استخدامه لتحديد اللغة. | (فارغ) |
| **Header** | اسم رأس HTTP الذي يحمل اللغة (مثل `Accept-Language`). | (فارغ) |
| **Route Prefixing** | تفعيل إضافة بادئة اللغة في روابط URL (مثل `/en/about`). | `true` |
| **Homepage Redirect** | إعادة توجيه الصفحة الرئيسية (`/`) إلى `/locale` (مثل `/ar`). | `true` |
| **Force Prefix** | إجبار إضافة بادئة اللغة لجميع طلبات GET التي لا تحتوي عليها. | `true` |

> **ملاحظة:** يمكن تجاوز هذه الإعدادات عبر متغيرات البيئة (`.env`) أو مباشرة في ملف الإعدادات (انظر القسم 4.2).

### 4.2. إعدادات ملف التهيئة (config.php)

يقع ملف الإعدادات في `plugins/nano/translateextended/config.php`. يمكن نشره إلى مجلد `config` عبر الأمر:
```bash
php artisan vendor:publish --provider="Nano\TranslateExtended\Plugin" --tag="config"
```

أبرز الإعدادات:

```php
return [
    // فرض اللغة الافتراضية (تجاوز الكشف التلقائي)
    'forceDefaultLocale' => env('NANO_TRANSLATE_FORCE_LOCALE', null),

    // إضافة بادئة اللغة الافتراضية أم لا
    'prefixDefaultLocale' => env('NANO_TRANSLATE_PREFIX_LOCALE', true),

    // مدة تخزين الكاش المؤقت للترجمات (بالدقائق)
    'cacheTimeout' => env('NANO_TRANSLATE_CACHE_TIMEOUT', 1440),

    // تعطيل بادئات اللغة للمسارات تلقائياً (قد يُستخدم في واجهات API)
    'disableLocalePrefixRoutes' => env('NANO_TRANSLATE_DISABLE_PREFIX_ROUTES', false),

    // إعدادات خاصة بواجهة API (للسلوك DynamicAddIncludeTranslatableApiFields)
    'translatable_fields' => [
        'fields'          => env('NANO_TRANSLATE_API_FIELDS', null),
        'format'          => env('NANO_TRANSLATE_API_FORMAT', 'group_by_field_under_all'),
        'locales'         => env('NANO_TRANSLATE_API_LOCALES', null),
        'includeOriginal' => env('NANO_TRANSLATE_API_INCLUDE_ORIGINAL', null),
        'useFallback'     => env('NANO_TRANSLATE_API_USE_FALLBACK', null),
        'useCache'        => env('NANO_TRANSLATE_API_USE_CACHE', null),
        'exclude'         => env('NANO_TRANSLATE_API_EXCLUDE', ''),
    ],
];
```

---

## 5. الميزات الرئيسية

### 5.1. كشف اللغة المتقدم (Browser Language Detection)

- عند زيارة المستخدم لأول مرة، تقوم الإضافة بقراءة اللغة المفضلة من المتصفح (عبر رأس `Accept-Language` أو معامل استعلام مخصص أو رأس مخصص).
- يتم تعيين اللغة تلقائياً للمستخدم، مع إمكانية حفظها في الجلسة (إذا تم تفعيل `Prefer User Session`).
- يمكن التحكم في الأولويات عبر الإعدادات.

### 5.2. إدارة بادئات المسارات (Route Prefixing)

- تضاف بادئة اللغة إلى جميع روابط الموقع (مثل `/ar/contact`).
- إعادة توجيه الصفحة الرئيسية تلقائياً إلى اللغة النشطة (إذا تم تفعيل `Homepage Redirect`).
- فرض البادئة على جميع طلبات GET التي لا تحتوي عليها (`Force Prefix`) لضمان اتساق الروابط.

> **مثال:**  
> إذا كان المستخدم يتصفح `https://example.com/about` وكانت اللغة الافتراضية هي `en`، فسيتم إعادة توجيهه إلى `https://example.com/en/about`.

### 5.3. التخزين المؤقت للترجمات (TranslatableContentCaching)

**سلوك `TranslatableContentCaching`** يوفر:

- **تخزين مؤقت للسمات المترجمة** لكل موديل ولكل لغة على حدة.
- **مسح تلقائي للكاش** عند حفظ أو حذف الموديل.
- **دوال ديناميكية** للوصول إلى القيم المترجمة مباشرة عبر الخاصيات (مثل `$model->title`).
- **إمكانية تعطيل الكاش** بشكل عام أو لكل استدعاء.
- **دعم الحقول القابلة للتحويل إلى JSON** مع معالجة آمنة للأخطاء.

#### إضافة السلوك إلى الموديل

```php
class Product extends Model
{
    public $implement = [
        '@RainLab.Translate.Behaviors.TranslatableModel',
        'Nano.TranslateExtended.Behaviors.TranslatableContentCaching'
    ];

    // حقول قابلة للترجمة (من RainLab.Translate)
    public $translatable = ['name', 'description'];

    // حقول إضافية للتخزين المؤقت (مثل JSON)
    public $translatableCaching = ['images', 'specs'];

    // حقول سيتم عرض ترجماتها الكاملة في API (انظر القسم 5.4)
    public $translatableApiFields = ['name', 'description'];
}
```

#### دوال إضافية في الموديل

- `getTranslated($attribute, $default, $locale, $fallback, $useCache, $useInBackend)`
- `getAllFieldTranslations($field, $includeDefault, $locales, $useFallback, $useCache)`
- `getTranslationsInFormat($fields, $format, $locales, $includeOriginal, $useFallback, $useCache)`
- `toArrayWithApiTranslations($locale, $useFallback, $useCache)`
- `invalidateAllCaches()`

#### تنسيقات إخراج الترجمات

| التنسيق | الوصف |
|---------|-------|
| `group_by_field` | يضيف لكل حقل `field_all` كائن بالترجمات `{x, ar, en}`. |
| `array` | `field_all` كمصفوفة من `{locale, value}`. |
| `both` | كلاهما (`_object` و `_array`). |
| `group_by_locale` | كائن واحد `allTranslate` يجمع حسب اللغة. |
| `group_by_field_under_all` | كائن `allTranslate` يجمع حسب الحقل. |
| `both_grouped` | يجمع `group_by_field_under_all` مع الحقول الفردية. |

### 5.4. تضمين الترجمات في واجهات API (DynamicAddIncludeTranslatableApiFields)

**سلوك `DynamicAddIncludeTranslatableApiFields`** يضيف إمكانية تضمين (`include`) الترجمات في استجابات API عبر المعامل `?include=translatable_fields`. يعمل مع أي Transformer يستخدم `Nano\API\Classes\Transformer` (أو أي Transformer يمكن تمديده بـ `October\Rain\Extension\ExtensionBase`).

#### الميزات

- تلقائياً يضيف `availableIncludes` جديدة: `translatable_fields` و `translatable_attributes`.
- **مرونة كاملة في تحديد الحقول واللغات** عبر:
  - خصائص عامة في الـ Transformer.
  - ملف الإعدادات `nano.translateextended::translatable_fields`.
  - معاملات query string (أولوية أعلى).
- **دعم الاستبعاد** `?translatable_fields.exclude=key1,key2`.
- **التوافق** مع جميع دوال `TranslatableContentCaching` (`getTranslationsInFormat`).

#### الاستخدام

**في الـ Plugin.php** (تسجيل الـ Transformer):
```php
public function boot()
{
    \Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields::extendTransformer(
        \Nano\ShopApi\Transformers\ProductTransformer::class
    );
}
```

**في طلب API**:
```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar,en
```

**الاستجابة** (بتنسيق `group_by_field_under_all`):
```json
{
    "data": {
        "id": 1,
        "name": "منتج 1",
        "description": "وصف المنتج",
        "translatable_fields": {
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
    }
}
```

### 5.5. الأدوات المساعدة الأخرى

- **ExtendedLocalePicker** – مكون (Component) يمكن إضافته للقوالب لعرض قائمة باللغات المتاحة مع روابطها.
- **ControllerTrait** – تريت لتسهيل الحصول على كائن `Cms\Classes\Controller` داخل أي كلاس.
- **TranslatableContentObjectTrait** – تريت بديل (قديم) يوفر دوال مشابهة لـ `TranslatableContentCaching`، يُنصح باستخدام السلوك الجديد بدلاً منه.

---

## 6. أمثلة عملية

### 6.1. عرض الترجمات في القالب (Twig)

بعد إضافة `TranslatableContentCaching` للموديل، يمكنك الوصول للقيم المترجمة مباشرة:

```twig
<h1>{{ product.name }}</h1> {# حسب اللغة الحالية #}
<p>{{ product.description }}</p>
```

### 6.2. الحصول على جميع ترجمات حقل معين

```php
$allNames = $product->getAllFieldTranslations('name', true, ['ar', 'en']);
// النتيجة: ['x' => 'منتج 1', 'ar' => 'منتج 1 بالعربية', 'en' => 'Product 1']
```

### 6.3. استدعاء واجهة API مع ترجمات مخصصة

```http
GET https://example.com/api/products/5?include=translatable_fields&translatable_fields.format=both_grouped&translatable_fields.useCache=false
```

### 6.4. تعطيل الكاش مؤقتاً في الكود

```php
$product->disableTranslatableCache();
$data = $product->toArrayWithApiTranslations();
$product->enableTranslatableCache();
```

---

## 7. التخصيص والتوسيع

### 7.1. إضافة ترايت أو سلوك جديد

يمكنك تمديد `TranslatableContentCaching` عبر إضافة دوال جديدة أو تعديل السلوك باستخدام `extendClassWith`.

### 7.2. استبدال دالة التوجيه (routes.php)

تم تضمين ملف `routes.php` الذي يعالج جميع منطق إعادة التوجيه والبادئات. يمكنك تعطيله بالكامل عن طريق إزالة الملف أو تعديله حسب حاجتك.

### 7.3. متغيرات البيئة (.env)

```ini
NANO_TRANSLATE_FORCE_LOCALE=ar
NANO_TRANSLATE_PREFIX_LOCALE=true
NANO_TRANSLATE_CACHE_TIMEOUT=2880
NANO_TRANSLATE_DISABLE_PREFIX_ROUTES=false
NANO_TRANSLATE_API_FORMAT=array
NANO_TRANSLATE_API_FIELDS=name,description
NANO_TRANSLATE_API_EXCLUDE=internal_notes
```

---

## 8. استكشاف الأخطاء وإصلاحها

| المشكلة | الحل المحتمل |
|---------|---------------|
| الترجمات لا تظهر في API بالرغم من `include=translatable_fields` | تأكد من أن الموديل يستخدم `TranslatableContentCaching`، وأن الـ Transformer مُمدد بالسلوك `DynamicAddIncludeTranslatableApiFields`. تحقق من سجلات الخطأ. |
| لا يتم إعادة توجيه الصفحة الرئيسية | تأكد من تفعيل `Homepage Redirect` في الإعدادات، ومن عدم وجود تعارض مع إضافات أخرى تغير المسارات. |
| الكاش لا يحدث بعد تعديل الترجمة | إذا تم تعديل الترجمة مباشرة في قاعدة البيانات، قد تحتاج إلى استدعاء `$model->invalidateAllCaches()` يدوياً. |
| خطأ `Method getTranslationsInFormat does not exist` | الموديل لا يطبق `TranslatableContentCaching`. أضف السلوك أو استخدم التريت البديل. |
| لا يتم كشف لغة المتصفح | تأكد من تفعيل `Browser Language Detection`، وأنه لم يتم تعطيله عبر متغيرات البيئة. |

---

## 9. الإصدارات والتحديثات

الإصدار الحالي: **1.0.11**

آخر التحديثات (من `version.yaml`):
- تحسين استدعاءات API للترجمات بدعم مرن للحقول واللغات.
- إضافة `includes` جديدة: `translatable_attributes` و `translatable_dirty_locales`.
- تحسين `TranslatableContentCaching` بدوال ديناميكية (`getTransCachedFields`, `setTransCachedFields`).
- معالجة أفضل للفولباك في اللغة الافتراضية.
- توافق أفضل مع هيكل بيانات `RainLab.Translate`.
- تسجيل أخطاء JSON في وضع التصحيح.

لمعرفة المزيد عن التحديثات لكل إصدار، راجع `version.yaml` داخل الإضافة.

---

## 10. الخاتمة

**Nano.TranslateExtended** تقدم حلاً متكاملاً لإدارة المحتوى متعدد اللغات في مشاريع OctoberCMS/WinterCMS. من خلال تكاملها السلس مع `RainLab.Translate`، توفر الإضافة:

- كشف ذكي للغة المستخدم.
- تحكماً كاملاً بالمسارات والبادئات.
- تخزيناً مؤقتاً عالي الأداء للترجمات.
- واجهات API مرنة لعرض الترجمات مع تحديد الحقول واللغات المطلوبة.
- أدوات مساعدة للمطورين لتسريع التطوير.

سواء كنت تدير مدونة صغيرة أو متجراً إلكترونياً ضخماً متعدد اللغات، فإن هذه الإضافة ستساعدك على تحقيق تجربة مستخدم سلسة وقابلة للتوسع.

---

**الوثائق المرجعية**:
- [توثيق سلوك `DynamicAddIncludeTranslatableApiFields`](./Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-ar.md)
- [توثيق سلوك `TranslatableContentCaching`](./Behaviors/Docs-TranslatableContentCaching-ar.md)
- [دليل استخدام الترجمات المحسّنة في API](./Docs-API-Translations-Guide-ar.md)
- [دليل تجاوز الترجمات](../Docs-Translation-Override-Guide-ar.md)
- [توثيق كلاس `TranslationOverrideManager`](./Classes/Docs-TranslationOverrideManager-Class-ar.md)

**روابط مفيدة:**
- [Winter.Translate Documentation](https://github.com/wintercms/wn-translate-plugin)
- [WinterCMS Documentation](https://wintercms.com/docs/)
- [RainLab.Translate Documentation](https://github.com/rainlab/translate-plugin)
- [OctoberCMS Documentation](https://docs.octobercms.com)

