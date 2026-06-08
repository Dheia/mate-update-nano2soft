# دليل استخدام الترجمات المحسّنة في API

## 1. مقدمة

توفر إضافة **Nano.TranslateExtended** آليات متقدمة لتضمين الترجمات في استجابات API الخاصة بك، مع مرونة كاملة في تحديد الحقول واللغات المطلوبة لكل طلب. هذا الدليل يشرح كيفية استخدام `include=translatable_fields` في واجهات API، والتحكم في مخرجات الترجمة عبر معاملات query string، والتعامل مع السيناريوهات المختلفة.

**المتطلبات الأساسية:**
- تثبيت إضافة `Nano.TranslateExtended` الإصدار 1.0.11 أو أحدث.
- تثبيت إضافة `RainLab.Translate` وتكوين اللغات المطلوبة.
- استخدام `Nano.API` أو أي نظام `Transformer` يدعم `League\Fractal` مع إمكانية تمديد الكلاسات.
- الموديلات التي نرغب بترجمتها يجب أن تستخدم سلوك `TranslatableContentCaching` (راجع التوثيق الخاص به).

---

## 2. إعداد الـ Transformer لاستقبال الترجمات

قبل استخدام الترجمات في API، يجب إضافة سلوك `DynamicAddIncludeTranslatableApiFields` إلى الـ `Transformer` المعني.

### الطريقة التلقائية (موصى بها)

في ملف `Plugin.php` الخاص بإضافة `Nano.TranslateExtended`، يتم استدعاء الدالة `extendTransformer()` التي تقوم بتسجيل الـ Transformers المطلوبة تلقائياً. يمكنك إضافة Transformers إضافية بنفس النمط:

```php
// في Plugin.php
public function extendTransformer(): void
{
    if (class_exists(\Nano\ShopApi\Transformers\ProductTransformer::class)) {
        $this->extendTransformerTranslatableApiFields(\Nano\ShopApi\Transformers\ProductTransformer::class);
    }
    // أضف Transformers أخرى هنا
    if (class_exists(\Custom\Api\Transformers\ArticleTransformer::class)) {
        $this->extendTransformerTranslatableApiFields(\Custom\Api\Transformers\ArticleTransformer::class);
    }
}
```

### الطريقة اليدوية

إذا كنت تفضل تمديد Transformer معين يدوياً:

```php
use Nano\TranslateExtended\Behaviors\DynamicAddIncludeTranslatableApiFields;

$transformerClass::extend(function($transformer) {
    if (!$transformer->isClassExtendedWith(DynamicAddIncludeTranslatableApiFields::class)) {
        $transformer->extendClassWith(DynamicAddIncludeTranslatableApiFields::class);
    }
});
```

بعد ذلك، سيصبح لديك في الـ Transformer الخاصية `availableIncludes` التي تتضمن:
- `translatable_fields`
- `translatable_attributes`
- `translatable_dirty_locales` (معطلة افتراضياً، يمكن تفعيلها في المُنشئ)

---

## 3. استخدام `include=translatable_fields` في طلبات API

عند استدعاء أي endpoint يستخدم Transformer مُعداً، يمكنك إضافة `include=translatable_fields` إلى query string، وستظهر الترجمات ضمن مفتاح `translatable_fields` في الاستجابة.

**مثال أساسي:**
```http
GET /api/products/1?include=translatable_fields
```

**استجابة نموذجية (باستخدام التنسيق الافتراضي `group_by_field_under_all`):**
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

---

## 4. معاملات التحكم في الترجمات (Parameters)

يمكنك تجاوز الإعدادات الافتراضية للترجمات (من ملف `config.php` أو من خصائص الـ Transformer) عبر إضافة معاملات إلى `translatable_fields.*` في query string. جميع المعاملات اختيارية.

### 4.1. تحديد الحقول (`fields`)

لجلب ترجمات لحقول محددة فقط (بدلاً من كل الحقول القابلة للترجمة):

```http
GET /api/products/1?include=translatable_fields&translatable_fields.fields=name,description
```

**يمكن أيضاً استخدام الاختصار `translated_fields`:**
```http
GET /api/products/1?include=translatable_fields&translated_fields=name,description
```

**صيغ مدعومة أخرى:**
- `translatable_fields.fields=all` أو `*` لجلب كل الحقول القابلة للترجمة (يتم استدعاء `getTranslatableAttributes()`).
- إرسال القيم كسلسلة مفصولة بفواصل (`name,description,seo_title`).

### 4.2. تحديد اللغات (`locales`)

لجلب ترجمات للغات محددة فقط (بدلاً من كل اللغات النشطة):

```http
GET /api/products/1?include=translatable_fields&translatable_fields.locales=ar,en
```

**صيغ مدعومة:**
- سلسلة مفصولة بفواصل (`ar,en,fr`).
- `all` أو `*` (يستخدم كل اللغات النشطة، وهو السلوك الافتراضي عند عدم التحديد).

### 4.3. تنسيق الإخراج (`format`)

يحدد شكل كائن الترجمات المرتجع. القيم المدعومة (راجع تنسيقات `TranslatableContentCaching`):

| القيمة | الوصف |
|--------|-------|
| `group_by_field` | يضيف لكل حقل `{field}_all` كائن بالترجمات `{x, ar, en}`. |
| `array` | `{field}_all` كمصفوفة من الكائنات `{locale, value}`. |
| `both` | كلاهما (`_object` و `_array`). |
| `group_by_locale` | كائن واحد `allTranslate` يجمع حسب اللغة. |
| `group_by_field_under_all` | كائن واحد `allTranslate` يجمع حسب الحقل (الافتراضي). |
| `both_grouped` | يجمع `group_by_field_under_all` مع الحقول الفردية `{field}_all`. |

**مثال:**
```http
GET /api/products/1?include=translatable_fields&translatable_fields.format=group_by_locale
```

**الاستجابة:**
```json
{
    "data": {
        "id": 1,
        "name": "منتج 1",
        "description": "وصف المنتج",
        "translatable_fields": {
            "allTranslate": {
                "x": { "name": "منتج 1", "description": "وصف المنتج" },
                "ar": { "name": "منتج 1 بالعربية", "description": "وصف المنتج بالعربية" },
                "en": { "name": "Product 1", "description": "Product description" }
            }
        }
    }
}
```

### 4.4. تضمين القيمة الأصلية (`includeOriginal`)

التحكم في ظهور القيمة الأصلية (من الحقل في الجدول الرئيسي) تحت مفتاح `'x'`. القيم: `true` / `false` / `1` / `0`.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.includeOriginal=false
```

### 4.5. استخدام الفولباك (`useFallback`)

إذا تم تفعيله (`true`)، فسيتم الرجوع إلى القيمة الأصلية عندما لا توجد ترجمة للغة المطلوبة.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.useFallback=true
```

### 4.6. استخدام الكاش (`useCache`)

تعطيل أو تفعيل استخدام الكاش لهذا الطلب فقط. القيم: `true` / `false` / `1` / `0`.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.useCache=false
```

### 4.7. استبعاد مفاتيح معينة (`exclude`)

قائمة مفصولة بفواصل بمفاتيح (مفاتيح الكائن النهائي) تريد استبعادها من نتيجة الترجمات.

```http
GET /api/products/1?include=translatable_fields&translatable_fields.exclude=allTranslate,internal_notes
```

> **ملاحظة:** `exclude` يتم تطبيقه بعد معالجة النتيجة النهائية، لذلك يمكن استخدامه لإزالة الكائن الجامع `allTranslate` إذا أردت فقط الحقول الفردية (في تنسيق `group_by_field` مثلاً).

---

## 5. أمثلة عملية متكاملة

### مثال 1: جلب ترجمات حقلين فقط (الاسم والوصف) باللغتين العربية والإنجليزية، بتنسيق المجموعة حسب اللغة

```http
GET /api/products/15?include=translatable_fields&translatable_fields.fields=name,description&translatable_fields.locales=ar,en&translatable_fields.format=group_by_locale
```

### مثال 2: جلب كل الترجمات مع تعطيل الكاش واستبعاد الكائن الجامع

```http
GET /api/products/15?include=translatable_fields&translatable_fields.fields=all&translatable_fields.useCache=false&translatable_fields.exclude=allTranslate
```

(هذا سيعيد فقط الحقول الفردية `name_all`, `description_all` بتنسيق `group_by_field` الافتراضي).

### مثال 3: استخدام الاختصار `translated_fields` فقط دون معاملات أخرى

```http
GET /api/products/15?include=translatable_fields&translated_fields=name,description
```

### مثال 4: دمج مع `include=translatable_attributes` للحصول على أسماء الحقول القابلة للترجمة

```http
GET /api/products/15?include=translatable_fields,translatable_attributes&translatable_fields.locales=ar
```

**الاستجابة:**
```json
{
    "data": {
        "id": 15,
        "translatable_attributes": ["name", "description", "meta_title"],
        "translatable_fields": {
            "allTranslate": {
                "name": { "x": "منتج 15", "ar": "منتج 15 بالعربية" },
                "description": { "x": "وصف المنتج", "ar": "وصف المنتج بالعربية" }
            }
        }
    }
}
```

### مثال 5: التحكم في الترجمات من خلال خصائص الـ Transformer مسبقاً (بدون تمرير معاملات)

إذا كنت تريد تحديد قيم افتراضية لكل استدعاءات Transformer معين، يمكن تعيين الخصائص العامة:

```php
class ProductTransformer extends \Nano\API\Classes\Transformer
{
    public $translatableFieldsConfigFields = ['name', 'description'];
    public $translatableFieldsConfigFormat = 'array';
    public $translatableFieldsConfigUseFallback = false;
}
```

ثم مجرد `?include=translatable_fields` كافية لاستخدام هذه الإعدادات.

---

## 6. الحصول على أسماء الحقول القابلة للترجمة فقط

يمكنك استخدام `include=translatable_attributes` دون الحاجة لجلب الترجمات نفسها، وهو مفيد لتوليد واجهات ديناميكية.

```http
GET /api/products/1?include=translatable_attributes
```

**الاستجابة:**
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["name", "description", "meta_title"]
    }
}
```

---

## 7. معلومات التصحيح (Debugging)

إذا تم تفعيل وضع التصحيح (`config('app.debug') = true`) وحدث خطأ أثناء جلب الترجمات، ستحتوي استجابة `translatable_fields` على المفتاحين `error` و `debug` مع تفاصيل الخطأ. هذا يساعد في تحديد المشاكل دون تعطيل API بالكامل.

**مثال لاستجابة خطأ:**
```json
{
    "data": {
        "id": 1,
        "translatable_fields": {
            "error": "Call to undefined method getTranslationsInFormat",
            "debug": {
                "line": 105,
                "file": "...",
                "trace": [...]
            }
        }
    }
}
```

---

## 8. التوافق مع إصدارات RainLab.Translate

يتم التعامل مع هيكل بيانات `RainLab.Translate` بمرونة: يبحث السلوك عن `locale` في `attributes` أولاً (الطريقة الحديثة)، ثم في الخاصية المباشرة `locale` كخطة احتياطية. هذا يضمن التوافق مع الإصدارات المختلفة من الإضافة.

---

## 9. نصائح الأداء

- **استخدم الكاش**: في الإنتاج، يوصى بتفعيل `useCache` (افتراضياً `false` في السلوك، ولكن يمكن تفعيله في الإعدادات العامة). سيقلل ذلك استعلامات قاعدة البيانات بشكل كبير.
- **حدد الحقول واللغات المطلوبة فقط**: كلما قل عدد الحقول واللغات، كانت الاستجابة أسرع.
- **استبعد المفاتيح غير الضرورية** باستخدام `exclude` إذا كنت لا تحتاج إلى الكائن الجامع أو بعض الحقول المحسوبة.

---

## 10. الخلاصة

أصبحت إدارة الترجمات في API مع `Nano.TranslateExtended` عملية مرنة وقوية. يمكنك الآن:

- تضمين الترجمات في أي استجابة API باستخدام `include=translatable_fields`.
- التحكم الدقيق في الحقول واللغات والتنسيق عبر query string.
- الحصول على أسماء الحقول القابلة للترجمة بشكل منفصل.
- تجاوز الإعدادات العامة لكل طلب على حدة.
- الاستفادة من الكاش لتحسين الأداء.

استخدم هذا الدليل كمرجع سريع لبناء واجهات API متعددة اللغات بكل احترافية.

---

**لمزيد من التفاصيل التقنية، راجع:**

- [توثيق الإضافة `Nano.TranslateExtended`](./Docs-Nano.TranslateExtended-ar.md)
- [توثيق سلوك TranslatableContentCaching](./Behaviors/Docs-TranslatableContentCaching-ar.md)
- [توثيق سلوك DynamicAddIncludeTranslatableApiFields](./Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-ar.md)
