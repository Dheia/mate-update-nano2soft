## 2026-06-03 – 2026-06-05

**تحديثات إضافة `Nano.TranslateExtended` – الإصدار 1.0.11**  
**تحسين شامل لاستدعاءات API للترجمات ومعالجة المحتوى القابل للترجمة**

### ملخص التحديثات

شهدت إضافة `Nano.TranslateExtended` تحديثاً رئيسياً في الإصدار **1.0.11** ركز على إصلاح مشكلة عدم ظهور الترجمات عند تضمين `translatable_fields` في استجابات API، وتوسيع مرونة جلب الترجمات عبر دعم تحديد الحقول واللغات بشكل ديناميكي من خلال `Input` والإعدادات والخصائص العامة. كما تم تحسين سلوك `TranslatableContentCaching` ليتيح التحكم البرمجي في الحقول القابلة للترجمة، وتصحيح منطق `fallback` للغة الافتراضية، ودعم فك تشفير JSON مع تسجيل الأخطاء، وتحسين التوافق مع هيكل بيانات `RainLab.Translate`.

---

## Nano.TranslateExtended v1.0.11 – تحسين API الترجمات وإصلاحات جوهرية

### أهداف الإصدار

- **إصلاح مشكلة عدم ظهور الترجمات** عند تضمين `translatable_fields` في استدعاءات API.
- **توفير مرونة كاملة في تحديد الحقول واللغات** المراد جلب ترجماتها عبر `Input` أو الإعدادات أو الخصائص العامة.
- **إضافة `includes` جديدة** مثل `translatable_attributes` و `translatable_dirty_locales` لدعم حالات الاستخدام المتقدمة.
- **تحسين `TranslatableContentCaching`** بإضافة دوال ديناميكية للتحكم بالحقول، وتصحيح منطق `fallback`، ودعم فك تشفير JSON مع تسجيل أخطاء.
- **توحيد واجهة تمرير الحقول** كسلسلة نصية مفصولة بفواصل (`title,content`) أو كلمة `all`/`*`.
- **تعزيز التوافق مع هيكل `RainLab.Translate`** في تخزين بيانات الترجمة.

### الميزات الجديدة والتحسينات

---

#### أولاً: تحديثات `DynamicAddIncludeTranslatableApiFields`

##### 1. دالة `includeTranslatableFields` المحسّنة

- **قراءة باراميترات مرنة** وفق الأولوية: الخاصية العامة ← الإعدادات ← `Input` ← القيمة الافتراضية.
- **دعم تحديد الحقول (`fields`) بأشكال متعددة**:
  - مصفوفة عادية.
  - سلسلة مفصولة بفواصل: `"title,content"`.
  - كلمة `"all"` أو `"*"` لجلب جميع الحقول القابلة للترجمة.
- **دعم تحديد اللغات (`locales`)** بنفس المرونة.
- **استدعاء `$item->transCollectFields()`** قبل استدعاء `getTranslationsInFormat()` لضمان تحديث قائمة الحقول ديناميكياً.
- **معالجة `exclude`** من `Input` أو الإعدادات لاستبعاد مفاتيح معينة من النتيجة.

##### 2. إضافة `includes` جديدة إلى الـ Transformer

- `translatable_attributes`: تعيد قائمة الحقول القابلة للترجمة في النموذج (`getTranslatableAttributes`).
- `translatable_dirty_locales` (معلقة اختيارياً): تعيد معلومات عن اللغات المتسخة والنسخ الأصلية.

##### 3. تحسين دالة `getConfigValue`

- إضافة دعم للقراءة من `Input` عبر المفتاح `translated_fields` كبديل عن `translatable_fields.fields`، لتوفير طريقة أبسط للمطورين.

##### 4. مثال استخدام في API

```json
GET /api/orders/orders/1?include=translatable_fields&translatable_fields.fields=title,content&translatable_fields.locales=en,ar
```

أو باستخدام الاختصار:
```json
GET /api/orders/orders/1?include=translatable_fields&translated_fields=title,content
```

---

#### ثانياً: تحديثات `TranslatableContentCaching`

##### 1. إضافة دوال برمجية للتحكم بالحقول

- `getTransCachedFields()`: إرجاع قائمة الحقول القابلة للترجمة الحالية.
- `setTransCachedFields($fields)`: تعيين القائمة ديناميكياً مع دعم السلسلة النصية المفصولة بفواصل أو `'all'`/`'*'`.

##### 2. تحسين دالة `getTranslated`

- إضافة معامل `$useInBackend` (افتراضي `false`) للسماح بترجمة الحقول حتى في بيئة الباكند عند الحاجة.
- تصحيح منطق `fallback` للغة الافتراضية: عند طلب `$locale` المطابق للغة الافتراضية، يتم تعيين `$fallback = true` لضمان الرجوع إلى القيمة الأصلية إذا لم توجد ترجمة.
- إضافة دالة مساعدة `processTranslatableJsonableValue` مع تسجيل أخطاء JSON في وضع التصحيح.

##### 3. تحسين دالة `getTranslatedAttributes`

- جعل البحث عن الترجمة أكثر توافقاً مع بنية `RainLab.Translate` التي تخزن `locale` داخل مصفوفة `attributes`.
- إضافة خطة احتياطية للبحث عبر الخاصية المباشرة `$item->locale`.

##### 4. تحسين دالة `getTranslationsInFormat`

- دعم تمرير `$fields` كسلسلة نصية مفصولة بفواصل أو `'all'`/`'*'`، مع تحويلها إلى مصفوفة داخل الدالة.
- توحيد المنطق مع باقي أجزاء الإضافة.

##### 5. تحسين تسجيل النماذج ديناميكياً (`registerModel`)

- إضافة دالة ثابتة `isPropertyExists` للتحقق من وجود الخاصية قبل إضافتها.
- تجنب إعادة إضافة الخصائص الموجودة مسبقاً.

##### 6. دالة `isPropertyExists` المساعدة

- دالة ثابتة تتحقق من وجود خاصية في كائن مع دعم `propertyExists` (لتوسعات `October\Rain\Extension\ExtensionBase`).

---

### ثالثاً: تحديث `version.yaml`

تمت إضافة الإصدار `1.0.11` مع وصف موجز للتحديثات:

```yaml
1.0.11:
    - Improved translation API includes with flexible field/locale parameters
    - Added support for passing fields as comma-separated string in translatable_fields include
    - Added new includes: translatable_attributes and translatable_dirty_locales
    - Enhanced TranslatableContentCaching with dynamic field setter/getter
    - Improved fallback handling for default locale in getTranslated method
    - Better compatibility with RainLab.Translate attribute data structure
    - Added error logging for JSON decoding issues in debug mode
    - Optimized cache key generation and invalidation
```

---

### أمثلة تطبيقية

#### 1. استخدام النطاقات الجديدة في الـ API (عبر التضمين)

**طلب جلب ترجمات حقلين فقط مع لغة واحدة:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=title,description&translatable_fields.locales=ar
```

**استخدام الاختصار `translated_fields`:**
```http
GET /api/products/products/1?include=translatable_fields&translated_fields=title
```

**جلب جميع الحقول القابلة للترجمة مع استبعاد حقل معين:**
```http
GET /api/products/products/1?include=translatable_fields&translatable_fields.fields=all&translatable_fields.exclude=seo_keywords
```

#### 2. استخدام `includeTranslatableAttributes` للحصول على أسماء الحقول فقط

```http
GET /api/products/products/1?include=translatable_attributes
```

**الاستجابة:**
```json
{
    "data": {
        "id": 1,
        "translatable_attributes": ["title", "description", "seo_title"]
    }
}
```

#### 3. التحكم البرمجي في `TranslatableContentCaching`

```php
// تعيين الحقول القابلة للترجمة ديناميكياً
$order = Order::find(1);
$order->setTransCachedFields('title,content,notes');

// الحصول على الترجمات بتنسيق مخصص
$translations = $order->getTranslationsInFormat(
    fields: 'title',
    format: TranslatableContentCaching::FORMAT_GROUP_BY_FIELD_UNDER_ALL,
    locales: ['en', 'ar'],
    includeOriginal: true,
    useFallback: false
);
```

#### 4. تفعيل الترجمة في الباكند (حالات خاصة)

```php
// فرض الترجمة حتى في بيئة الباكند
$value = $model->getTranslated('title', null, 'ar', true, true, true);
```

---

## ملخص الإصدار (1.0.11)

| الإصدار | أبرز الميزات |
|---------|---------------|
| **Nano.TranslateExtended 1.0.11** | إصلاح مشكلة `translatable_fields` في API، دعم تحديد الحقول واللغات بشكل مرن، إضافة `translatable_attributes` و `translatable_dirty_locales`، تحسين `TranslatableContentCaching` بدوال ديناميكية ومنطق `fallback` للغة الافتراضية، توافق أفضل مع `RainLab.Translate`، تسجيل أخطاء JSON. |

---

### متطلبات الترقية

1. **تحديث الملفات**:
   - `DynamicAddIncludeTranslatableApiFields.php`
   - `TranslatableContentCaching.php`
   - `version.yaml`

2. **لا توجد هجرات جديدة**: الإصدار لا يتطلب تغييرات في قاعدة البيانات.

3. **الاعتماديات**:
   - `RainLab.Translate` (الإصدار الأحدث)
   - `OctoberCMS` ≥ 2.0 أو `WinterCMS`

4. **إعدادات اختيارية**:
   - يمكن إضافة إعدادات مخصصة في ملف `config.php` للإضافة تحت مفتاح `nano.translateextended::translatable_fields` لتحديد قيم افتراضية لـ `fields`, `format`, `locales`, `includeOriginal`, `useFallback`, `useCache`, `exclude`.

5. **اختبار التوافق**:
   - اختبار استدعاء API مع `include=translatable_fields` ومختلف الباراميترات.
   - اختبار النماذج التي تستخدم `TranslatableContentCaching` والتحقق من عمل `getTranslated` في الواجهة والخلفية.
   - اختبار فك تشفير JSON للحقول القابلة للترجمة من نوع JSON.

---

### الخاتمة

يمثل الإصدار **Nano.TranslateExtended 1.0.11** نقلة نوعية في كيفية التعامل مع الترجمات عبر API وفي الكود البرمجي. لم يعد المطور بحاجة إلى كتابة استعلامات معقدة أو تمديد الـ Transformer يدوياً، بل أصبح بإمكانه استخدام `translatable_fields` مع تحديد الحقول واللغات المطلوبة مباشرة عبر الـ `Input`، والحصول على الترجمات بتنسيق نظيف ومرن. كما تم تصحيح العديد من المشاكل السلوكية في `TranslatableContentCaching` لضمان أن الترجمات تعمل كما هو متوقع في جميع السيناريوهات، بما فيها اللغة الافتراضية والرجوع إلى القيم الأصلية. الكود موثق بالكامل، ويتضمن تحسينات في الأداء والتوافق.

---

**الوثائق المرجعية**:
- [توثيق الإضافة `Nano.TranslateExtended`](./docs/TranslateExtended/Behaviors/Docs-Nano.TranslateExtended-ar.md)
- [توثيق سلوك `DynamicAddIncludeTranslatableApiFields`](./docs/TranslateExtended/Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-ar.md)
- [توثيق سلوك `TranslatableContentCaching`](./docs/TranslateExtended/Behaviors/Docs-TranslatableContentCaching-ar.md)
- [دليل استخدام الترجمات المحسّنة في API](./docs/TranslateExtended/Docs-API-Translations-Guide-ar.md)