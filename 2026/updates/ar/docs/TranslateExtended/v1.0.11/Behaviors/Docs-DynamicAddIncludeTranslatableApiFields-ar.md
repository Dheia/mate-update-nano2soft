## توثيق متكامل لسلوك DynamicAddIncludeTranslatableApiFields

### 1. مقدمة

**DynamicAddIncludeTranslatableApiFields** هو سلوك (Behavior) خاص بـ **Nano.TranslateExtended**، مصمم لتوسيع نطاق عمل **Transformer** في نظام **Nano.API** (أو أي نظام يستخدم League Fractal) ليدعم تضمين (`include`) الترجمات المتعددة في استجابات API.

المشكلة الأساسية التي يحلها هذا السلوك: في الإصدارات السابقة، لم تكن الترجمات تظهر عند إضافة `translatable_fields` إلى قائمة `include` في طلب API. بالإضافة إلى ذلك، لم تكن هناك طريقة مرنة لتحديد الحقول أو اللغات المطلوبة لكل استدعاء. هذا السلوك يعالج هذه المشاكل تماماً.

يوفر السلوك ثلاث دوال للتضمين:

- `includeTranslatableFields`: المسؤولة عن إرجاع الترجمات الفعلية للحقول، مع دعم تحديد الحقول واللغات والتنسيق والاستبعاد.
- `includeTranslatableAttributes`: تعيد قائمة أسماء الحقول القابلة للترجمة في النموذج (`getTranslatableAttributes`).
- `includeTranslatableDirtyLocales`: (اختيارية، معطلة افتراضياً) تعيد معلومات حول اللغات المتسخة والنسخ الأصلية للتصحيح.

### 2. الهدف والميزات

- **إصلاح مشكلة عدم ظهور الترجمات** عند تضمين `translatable_fields` في API.
- **مرونة كاملة في تحديد الحقول واللغات** عبر:
  - خصائص عامة يمكن تعيينها في الـ Transformer نفسه.
  - ملف الإعدادات `nano.translateextended::translatable_fields`.
  - معاملات query string في طلب API (أولوية عالية).
- **دعم صيغ متعددة للحقول**: مصفوفة، سلسلة مفصولة بفواصل (`title,content`)، أو الكلمات `'all'`/`'*'` لجلب جميع الحقول القابلة للترجمة.
- **دعم استبعاد مفاتيح معينة** من النتيجة (مثل `exclude=password,secret`).
- **إمكانية تجاوز الإعدادات لكل طلب** دون تعديل الكود أو الإعدادات العامة.
- **توافق تام مع معمارية `Nano.API`** (Transformer يمدد `October\Rain\Extension\ExtensionBase`).

### 3. التثبيت والإعداد

#### المتطلبات الأساسية
- إضافة `Nano.TranslateExtended` مثبتة ومفعلة.
- إضافة `Nano.API` (أو أي نظام يستخدم `League\Fractal` مع `Transformer` يمكن تمديده).
- الموديلات التي تريد ترجمتها يجب أن تستخدم `TranslatableContentCaching` (أو على الأقل لديها دالة `getTranslationsInFormat`).

#### إضافة السلوك إلى الـ Transformer

في ملف `Plugin.php` الخاص بإضافتك، أو مباشرة في تعريف الـ Transformer، قم باستدعاء الدالة المساعدة `extendTransformerTranslatableApiFields` المتوفرة في `Plugin` الأساسي للإضافة، أو قم بتمديد الـ Transformer يدوياً:

**طريقة تلقائية (موصى بها):**
```php
// في Plugin.php للإضافة Nano.TranslateExtended (موجود بالفعل)
public function extendTransformer(): void
{
    if(class_exists(\Nano\ShopApi\Transformers\ProductTransformer::class)){
        $this->extendTransformerTranslatableApiFields(\Nano\ShopApi\Transformers\ProductTransformer::class);
    }
    // أضف باقي الـ Transformers التي تحتاج الترجمات
}
```

**طريقة يدوية (في أي إضافة):**
```php
use Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields;

$transformerClass::extend(function($transformer) {
    if (!$transformer->isClassExtendedWith(DynamicAddIncludeTranslatableApiFields::class)) {
        $transformer->extendClassWith(DynamicAddIncludeTranslatableApiFields::class);
    }
});
```

بعد ذلك، سيصبح لديك في الـ Transformer ثلاثة `availableIncludes` جديدة:
- `translatable_fields`
- `translatable_attributes`
- `translatable_dirty_locales` (معلقة، يمكن تفعيلها بإزالة التعليق في المُنشئ)

### 4. الخصائص العامة للتحكم في الباراميترات

يمكنك تعيين قيم افتراضية للباراميترات مباشرة في الـ Transformer نفسه باستخدام الخصائص العامة التالية (جميعها قيمتها الافتراضية `null`):

| الخاصية | الوصف |
|----------|--------|
| `$translatableFieldsConfigFields` | الحقول المطلوبة (مصفوفة أو سلسلة مفصولة بفواصل أو `'all'`/`'*'`). |
| `$translatableFieldsConfigFormat` | تنسيق الإخراج (راجع تنسيقات `TranslatableContentCaching`). |
| `$translatableFieldsConfigLocales` | اللغات المطلوبة (مصفوفة أو سلسلة مفصولة بفواصل). |
| `$translatableFieldsConfigIncludeOriginal` | boolean: تضمين القيمة الأصلية بمفتاح `'x'` أم لا. |
| `$translatableFieldsConfigUseFallback` | boolean: استخدام الفولباك (الرجوع للقيمة الأصلية عند عدم وجود ترجمة). |
| `$translatableFieldsConfigUseCache` | boolean: استخدام الكاش أم لا. |

**مثال:**
```php
class ProductTransformer extends Transformer
{
    public $translatableFieldsConfigFields = 'name,description';
    public $translatableFieldsConfigFormat = 'group_by_field_under_all';
    public $translatableFieldsConfigLocales = ['ar', 'en'];
}
```

### 5. أولوية قراءة القيم

عند استدعاء `includeTranslatableFields`، يتم تحديد قيم الباراميترات حسب الأولوية التالية (من الأعلى إلى الأدنى):

1. **الخاصية العامة** في الـ Transformer (أو في السلوك نفسه، ولكن السلوك يقرأ من الـ Transformer بشكل أساسي).
2. **قيمة من ملف الإعدادات** (`config/nano.translateextended::translatable_fields.{$param}`). يمكنك تعيينها في ملف `config.php` للإضافة.
3. **قيمة من `Input`** (query string) بنمطين:
   - `translatable_fields.{param}` (مثل `translatable_fields.fields=name,content`)
   - مباشرة: `translatableFieldsConfig{Param}` (مثل `translatableFieldsConfigFields=name`).
   - للحقل `fields` فقط: `translated_fields` أو `translatable_fields` (اختصار).
4. **القيمة الافتراضية** الممررة إلى الدالة (في الكود: `'group_by_field_under_all'` للتنسيق، `null` لمعظمها، `true` لـ `includeOriginal`، `false` لـ `useFallback` و `useCache`).

هذا يمنحك تحكماً كاملاً: يمكنك وضع إعدادات عامة في config، ثم تجاوزها في Transformer معين، ثم تجاوزها في كل طلب API على حدة.

### 6. دوال التضمين (includes)

#### أ) `includeTranslatableFields($item)`

الدالة الأساسية التي تُستدعى عند طلب `?include=translatable_fields`. تقوم بما يلي:

1. تسترجع قيم الباراميترات حسب الأولوية المذكورة.
2. تقوم بمعالجة `$fields` و `$locales` (تحويل السلاسل المفصولة بفواصل إلى مصفوفات، وتحويل `'all'`/`'*'` إلى `null` لجلب كل الحقول).
3. تستدعي `$item->transCollectFields()` (إن وجدت) لضمان تحديث قائمة الحقول القابلة للترجمة من الموديل.
4. تستدعي `$item->getTranslationsInFormat()` مع الباراميترات المجهزة.
5. إذا كانت النتيجة تحتوي على مفتاح `'allTranslate'` (من تنسيق `group_by_field_under_all` مثلاً)، تستخرج قيمته.
6. تطبق استبعاد المفاتيح (`exclude`) من `Input` أو الإعدادات.
7. تعيد `Item` resource من Fractal يحتوي على النتيجة (أو مصفوفة فارغة عند الخطأ).

**مثال على الاستجابة (عند التنسيق `group_by_field_under_all`):**
```json
{
    "data": {
        "id": 1,
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

#### ب) `includeTranslatableAttributes($item)`

تعيد مصفوفة بأسماء الحقول القابلة للترجمة في الموديل، عبر استدعاء `$item->getTranslatableAttributes()`.

**مثال:**
```http
GET /api/products/1?include=translatable_attributes
```
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["name", "description", "meta_title"]
    }
}
```

#### ج) `includeTranslatableDirtyLocales($item)`

(معطلة افتراضياً، تفعيلها يتطلب إزالة التعليق في المُنشئ). تعيد معلومات التصحيح: `getDirtyLocales`، `getTranslatableOriginals('en')`. مفيدة للمطورين عند الحاجة لمراقبة التغييرات في الترجمات.

### 7. تنسيقات الإخراج المدعومة

تعتمد التنسيقات على ما توفره دالة `getTranslationsInFormat` في الموديل (التي تأتي من `TranslatableContentCaching`). يمكنك استخدام أي من الثوابت التالية:

- `'group_by_field'` – لكل حقل كائن `{field}_all`.
- `'array'` – لكل حقل مصفوفة من `{locale, value}`.
- `'both'` – كلاهما بمفاتيح مختلفة.
- `'group_by_locale'` – كائن واحد `allTranslate` (أو المفتاح المخصص) حسب اللغة.
- `'group_by_field_under_all'` – كائن واحد `allTranslate` حسب الحقل.
- `'both_grouped'` – كائن `allTranslate` حسب الحقل مع الحقول الفردية `{field}_all`.

القيمة الافتراضية للتنسيق في هذا السلوك هي `'group_by_field_under_all'` ما لم يتم تجاوزها.

### 8. التحكم عبر معاملات Query String (Input)

عند استدعاء API، يمكنك تمرير المعاملات التالية (ضمن `?`):

| المعامل | الوصف | مثال |
|----------|--------|-------|
| `translatable_fields.fields` | الحقل أو الحقول المطلوبة. | `?translatable_fields.fields=name,description` |
| `translated_fields` | اختصار لـ `fields` فقط. | `?translated_fields=name` |
| `translatable_fields.format` | تنسيق الإخراج. | `?translatable_fields.format=group_by_locale` |
| `translatable_fields.locales` | اللغات المطلوبة. | `?translatable_fields.locales=ar,en` |
| `translatable_fields.includeOriginal` | `true`/`false`/`1`/`0`. | `?translatable_fields.includeOriginal=false` |
| `translatable_fields.useFallback` | `true`/`false`. | `?translatable_fields.useFallback=true` |
| `translatable_fields.useCache` | `true`/`false`. | `?translatable_fields.useCache=false` |
| `translatable_fields.exclude` | مفاتيح لاستبعادها من النتيجة النهائية. | `?translatable_fields.exclude=password,secret` |

**ملاحظة:** يمكنك أيضاً استخدام الصيغة المباشرة `translatableFieldsConfigFields`، لكن الصيغة الموصى بها هي `translatable_fields.*` للتوافق مع توثيق API.

### 9. أمثلة عملية لاستدعاء API

#### مثال 1: جلب ترجمات حقلين فقط مع اللغة العربية
```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar
```

#### مثال 2: جلب كل الترجمات مع استبعاد حقل معين، بدون كاش
```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=all&translatable_fields.useCache=false&translatable_fields.exclude=internal_notes
```

#### مثال 3: استخدام الاختصار `translated_fields` مع تنسيق `group_by_locale`
```http
GET /api/products/1?include=translatable_fields&translated_fields=title&translatable_fields.format=group_by_locale
```

#### مثال 4: جلب أسماء الحقول القابلة للترجمة فقط
```http
GET /api/products/1?include=translatable_attributes
```

### 10. تخصيص السلوك في الـ Transformer

إذا كنت بحاجة إلى قيم افتراضية معينة لجميع استدعاءات Transformer معين، يمكنك تعيين الخصائص العامة:

```php
use Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields;

class MyCustomTransformer extends \Nano\API\Classes\Transformer
{
    public $translatableFieldsConfigFields = ['name', 'description'];
    public $translatableFieldsConfigFormat = 'array';
    public $translatableFieldsConfigUseFallback = false;

    public function __construct()
    {
        // إضافة السلوك يدوياً أو عبر Plugin.php
        $this->extendClassWith(DynamicAddIncludeTranslatableApiFields::class);
    }
}
```

### 11. دالة `getConfigValue` المساعدة

تستخدم داخلياً لتطبيق أولوية القراءة. يمكنك استدعاؤها إذا أردت قراءة باراميتر معين بنفس الأولوية:

```php
protected function getConfigValue($param, $default = null)
```

**المعلمات:**
- `$param`: اسم الباراميتر (`'fields'`, `'format'`, `'locales'`, `'includeOriginal'`, `'useFallback'`, `'useCache'`).
- `$default`: القيمة الافتراضية إذا لم يوجد في أي مصدر.

**قيمة الإرجاع:** القيمة المحددة حسب الأولوية.

**مثال:**
```php
$fields = $this->getConfigValue('fields', null);
```

### 12. دالة `isMethodExists`

تستخدم للتحقق الآمن من وجود دالة في الكائن، مع دعم `methodExists` الذي قد يكون موجوداً في بعض التوسعات (مثل `October\Rain\Extension\ExtensionBase`).

### 13. ملاحظات مهمة

- **اعتماد على `TranslatableContentCaching`**: لكي يعمل `includeTranslatableFields` بشكل صحيح، يجب أن يكون الموديل المراد ترجمته قد أضاف سلوك `TranslatableContentCaching` (أو على الأقل أن يكون لديه دالة `getTranslationsInFormat`). في حالة عدم وجودها، سيعيد السلوك مصفوفة فارغة.
- **أداء الكاش**: يمكن تفعيل الكاش لتحسين الأداء، لكن انتبه إلى أن تحديثات الترجمات قد لا تظهر فوراً إذا كان الكاش مفعلاً. استخدم `useCache=false` مؤقتاً أو قم بمسح الكاش بعد التحديث.
- **الاستبعاد (`exclude`)**: يتم تطبيقه على النتيجة النهائية بعد كل المعالجة، لذا يمكنك استبعاد مفاتيح مثل `'allTranslate'` إذا أردت فقط الحقول الفردية.
- **الترجمة في الـ Backend**: السلوك لا يترجم تلقائياً في بيئة الباكند، ولكن إذا كان الموديل يستخدم `TranslatableContentCaching` مع تفعيل `useInBackend`، فستعمل الترجمة.
- **التوافق**: يعمل هذا السلوك مع أي `Transformer` يمتد من `October\Rain\Extension\ExtensionBase` أو `Nano\API\Classes\Transformer`. إذا كنت تستخدم Fractal مباشرة بدون تمديد `ExtensionBase`، قد تحتاج إلى تعديل بسيط.

### 14. استكشاف الأخطاء وإصلاحها

#### المشكلة: الترجمات لا تظهر رغم إضافة `include=translatable_fields`
- تأكد من أن الموديل المستخدم يطبق `TranslatableContentCaching` (أو لديه دالة `getTranslationsInFormat`).
- تأكد من أن الـ Transformer قد تم تمديده بهذا السلوك (راجع طريقة `extendTransformer` في `Plugin.php`).
- تحقق من سجلات الخطأ (`storage/logs/laravel.log`) حيث يقوم السلوك بتسجيل الأخطاء في وضع التصحيح.

#### المشكلة: يظهر خطأ `Method getTranslationsInFormat does not exist`
- هذا يعني أن الموديل لا يملك الدالة المطلوبة. أضف `TranslatableContentCaching` إلى `$implement` في الموديل أو قم بتسجيله باستخدام `TranslatableContentCaching::registerModel()`.

#### المشكلة: لا يتم تطبيق بعض الباراميترات من الـ Input
- تأكد من كتابة أسماء الباراميترات بشكل صحيح (مثل `translatable_fields.fields` وليس `translatable_fields.field`).
- جرب استخدام الاختصار `translated_fields` لتحديد الحقول.
- تأكد من أن القيم المرسلة لا تحتوي على مسافات زائدة.

### 15. مثال كامل لاستخدام السلوك في مشروع

**ملف `Plugin.php` (تسجيل الـ Transformer):**
```php
public function boot()
{
    // تسجيل السلوك لـ ProductTransformer
    if (class_exists(\Nano\ShopApi\Transformers\ProductTransformer::class)) {
        \Nano\ShopApi\Transformers\ProductTransformer::extend(function($transformer) {
            if (!$transformer->isClassExtendedWith(\Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields::class)) {
                $transformer->extendClassWith(\Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields::class);
            }
        });
    }
}
```

**استدعاء API (عرض منتج مع ترجمات محددة):**
```http
GET https://example.com/api/products/15?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar,en&translatable_fields.format=both_grouped
```

**الاستجابة المتوقعة:**
```json
{
    "data": {
        "id": 15,
        "name": "منتج عادي",
        "description": "وصف عادي",
        "name_all": {
            "x": "منتج عادي",
            "ar": "منتج عادي بالعربية",
            "en": "Normal Product"
        },
        "description_all": {
            "x": "وصف عادي",
            "ar": "وصف عادي بالعربية",
            "en": "Normal description"
        },
        "allTranslate": {
            "name": {
                "x": "منتج عادي",
                "ar": "منتج عادي بالعربية",
                "en": "Normal Product"
            },
            "description": {
                "x": "وصف عادي",
                "ar": "وصف عادي بالعربية",
                "en": "Normal description"
            }
        }
    }
}
```
