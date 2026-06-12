## 2026-06-10 - 2026-06-13

**تحديثات إضافة `Nano.TranslateExtended` – الإصدار 1.0.12**  
**إضافة نظام متكامل لتجاوز الترجمات (Translation Override) مع دعم متعدد المسارات والكاش والإعدادات عبر الواجهة الخلفية ومتغيرات البيئة**

### ملخص التحديثات

يُضيف الإصدار **1.0.12** نظاماً متكاملاً ومرناً لتجاوز أي ترجمة في أي إضافة أو في النظام الأساسي، دون الحاجة إلى تعديل ملفات الترجمة الأصلية. يعتمد النظام على آلية `TranslationOverrideManager` التي تقرأ الترجمات من مجلدات مخصصة، وتخزنها في الكاش، وتدمجها في نظام الترجمة عبر الدالة `set()` من `October\Rain\Translation` لضمان التجاوز الفعال. يتم التحكم في النظام من خلال واجهة إدارة جديدة ضمن إعدادات الإضافة، أو عبر متغيرات البيئة، أو ملف الإعدادات `config.php`.

---

## Nano.TranslateExtended v1.0.12 – نظام تجاوز الترجمات (Translation Override)

### أهداف الإصدار

- **توفير آلية احترافية لتجاوز الترجمات** دون تعديل الملفات الأصلية للإضافات أو النظام.
- **دعم مسارات متعددة لملفات التجاوز** مع إمكانية تحديد الأولوية (آخر مسار له الأسبقية).
- **دمج نظام التجاوز مع إعدادات الإضافة** بحيث يمكن تفعيله / تعطيله وتعديل مدته ومساراته عبر واجهة المستخدم.
- **دعم كامل لمتغيرات البيئة** للتحكم بكل إعداد دون الحاجة إلى قاعدة البيانات.
- **تخزين مؤقت ذكي** لتجنب إعادة قراءة الملفات في كل طلب، مع إمكانية تعطيل الكاش للتطوير.
- **معالجة استثنائية شاملة** وتسجيل الأخطاء في `system.log` لتسهيل التصحيح.
- **توافق كامل مع `October\Rain\Translation::set()`** لضمان استبدال الترجمات بشكل صحيح.

### الميزات الجديدة والتحسينات

---

#### أولاً: `TranslationOverrideManager` – مدير التجاوزات

##### 1. قراءة الملفات من مسارات متعددة

- البحث في المسارات المحددة (مثل `lang/override` و `plugins/nano/translateextended/override/lang`).
- دعم المسارات المطلقة والنسبية (تُحول تلقائياً إلى `base_path()`).
- دمج جميع ملفات PHP الموجودة في المجلد (أي اسم ملف) وفي المجلدات الفرعية.
- تحويل المصفوفات المتداخلة إلى صيغة `dot.notation` (مثلاً `['a' => ['b' => 'c']]` تصبح `['a.b' => 'c']`).

##### 2. دمج الترجمات باستخدام `set()`

- استخدام الدالة `$translator->set($key, $value, $locale)` التي توفرها OctoberCMS لضمان تجاوز الترجمة حتى لو كانت قد حملت مسبقاً.
- دعم المفاتيح العادية (`messages.welcome`) والمفاتيح ذات مساحة الاسم (`nano.plugin::lang.key`).
- تسجيل الأخطاء في حال فشل إضافة أي ترجمة.

##### 3. التخزين المؤقت (Cache)

- تخزين الترجمات المقروءة من الملفات لمدة قابلة للتكوين (افتراضياً 60 دقيقة).
- إمكانية تعطيل الكاش بوضع `override_cache_minutes = 0`.
- مسح الكاش تلقائياً عند حفظ إعدادات الإضافة.
- مسح الكاش يدوياً عبر دالة `clearCache()` أو مع الحدث `locale.changed`.

##### 4. دوال مساعدة عامة

- `loadAll($locale = null)`: تحميل ترجمات كل اللغات المتاحة أو لغة واحدة.
- `loadForLocale($locale)`: تحميل الترجمة للغة معينة ودمجها.
- `reload($locale = null)`: مسح الكاش وإعادة تحميل اللغة.
- `addPath($path)`: إضافة مسار إضافي ديناميكياً.
- `getPaths()` و `isEnabled()` و `getCacheMinutes()` للتشخيص.

##### 5. معالجة استثنائية وتسجيل أخطاء

- كل دالة محاطة بـ `try/catch` مع تسجيل الخطأ في `Log::error` أو `Log::warning`.
- التعامل مع `Throwable` لالتقاط جميع أنواع الأخطاء.
- تسجيل معلومات عند فشل تضمين ملف ترجمة أو عند وجود ملف لا يعيد مصفوفة.

---

#### ثانياً: تحديثات نموذج `Settings`

##### 1. إضافة حقول جديدة في `fields.yaml`

- `enable_overrides` (switch): تفعيل / تعطيل نظام التجاوز.
- `override_cache_minutes` (number): مدة التخزين المؤقت (بالدقائق).
- `override_paths` (textarea): قائمة المسارات (كل مسار في سطر جديد).

##### 2. دوال مساعدة جديدة في `Settings`

- `getWithEnvPriority($key, $envVarName, $default)`: استرجاع القيمة وفق الأولوية (env > database > config).
- `getOverridePaths()`: إرجاع مصفوفة المسارات بعد تحويل النسبي إلى مطلق ودعم سلاسل متعددة الأسطر.
- تحسين معالجة القيم المنطقية من `env` (مثل `false` و `true` و `null`).

##### 3. تحسين التوافق مع PHP 7.4+

- استبدال `str_starts_with()` بـ `strpos() === 0` لضمان العمل على الإصدارات الأقدم.

---

#### ثالثاً: إعدادات `config.php` الجديدة

أضيفت المفاتيح التالية إلى ملف `config.php` داخل الإضافة:

```php
'enable_overrides'          => env('NANO_TRANSLATE_ENABLE_OVERRIDES', true),
'override_paths'            => [
    base_path('lang/override'),
    plugins_path('nano/translateextended/override/lang'),
],
'override_cache_minutes'    => env('NANO_TRANSLATE_CACHE_MINUTES', 60),
'forceDefaultLocale'        => env('NANO_TRANSLATE_FORCE_LOCALE', null),
'prefixDefaultLocale'       => env('NANO_TRANSLATE_PREFIX_LOCALE', true),
'disableLocalePrefixRoutes' => env('NANO_TRANSLATE_DISABLE_PREFIX_ROUTES', false),
```

---

#### رابعاً: متغيرات البيئة (`.env`) المدعومة

| المتغير | الغرض | مثال |
|---------|-------|------|
| `NANO_TRANSLATE_ENABLE_OVERRIDES` | تفعيل / تعطيل النظام | `true` / `false` |
| `NANO_TRANSLATE_CACHE_MINUTES` | مدة الكاش بالدقائق | `120` |
| `NANO_TRANSLATE_OVERRIDE_PATHS` | مسارات التجاوز (سطر واحد، مفصولة بفواصل أو أسطر) | `lang/override,plugins/nano/translateextended/override/lang` |
| `NANO_TRANSLATE_FORCE_LOCALE` | فرض لغة ثابتة على الموقع | `ar` |
| `NANO_TRANSLATE_PREFIX_LOCALE` | إضافة بادئة اللغة للروابط (حتى للغة الافتراضية) | `true` / `false` |
| `NANO_TRANSLATE_DISABLE_PREFIX_ROUTES` | تعطيل توجيه اللغة عبر الرابط بشكل كامل | `true` / `false` |

---

#### خامساً: ربط الأحداث (Event Listeners)

- **`backend.form.beforeSave`**: مسح كاش التجاوزات عند حفظ إعدادات الإضافة.
- **`locale.changed`**: إعادة تحميل ترجمات اللغة الجديدة عند تغيير اللغة.
- (اختياري) **`rainlab.translate.beforeResolveLocale`** و **`LocaleMiddleware::$prefixDefaultLocale`** تم دعمهما في `applyLocaleSettings` لتطبيق `forceDefaultLocale` و `prefixDefaultLocale`.

---

#### سادساً: أمثلة ملفات التجاوز

**ملف `lang/override/ar/custom.php`**

```php
<?php return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'إدارة متجر آخر',
    'tss.inventory::lang.products.form.use_case_options.new' => 'جديد',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'ممتاز',
];
```

**ملف `lang/override/en/custom.php`**

```php
<?php return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Another Store',
    'tss.inventory::lang.products.form.use_case_options.new' => 'New',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
];
```

---

### كيفية الاستخدام (للمطورين)

#### 1. تفعيل النظام

- من لوحة التحكم: `الإعدادات → Translate Extended → تفعيل تجاوز الترجمات`.
- أو عبر `.env`: `NANO_TRANSLATE_ENABLE_OVERRIDES=true`.

#### 2. إنشاء ملفات التجاوز

- أنشئ مجلد `lang/override/{locale}/` في جذر المشروع.
- ضع أي ملف `.php` يعيد مصفوفة المفاتيح والقيم المطلوب تجاوزها.

#### 3. مسح الكاش

- احفظ إعدادات الإضافة (يتم تلقائياً).
- أو نفذ أمر `php artisan cache:clear`.
- أو استخدم دالة `$manager->clearCache()` برمجياً.

#### 4. استخدام البرمجة

```php
$manager = app('translateextended.translationOverride');
$manager->addPath(base_path('my/extra/path'));
$manager->reload('ar');
```

---

### الترقية من الإصدارات السابقة

1. تحديث ملفات الإضافة عبر `php artisan plugin:refresh Nano.TranslateExtended` أو نسخ الملفات يدوياً.
2. تشغيل `php artisan cache:clear` لمسح الكاش القديم.
3. (اختياري) إضافة متغيرات البيئة الجديدة إلى `.env`.
4. الدخول إلى صفحة إعدادات الإضافة وضبط القيم المطلوبة.

> **ملاحظة:** لا توجد تغييرات في هيكل قاعدة البيانات، الترقية آمنة ولا تؤثر على البيانات الحالية.

---

### معالجة الأخطاء الشائعة

| المشكلة | الحل |
|---------|------|
| الترجمات لا تتجاوز رغم وجود الملفات | تأكد من تفعيل `enable_overrides` ومن صحة المسارات. امسح الكاش. |
| خطأ `Too few arguments to function ...` | حدث `locale.changed` يحتاج إلى `$oldLocale = null` (تم إصلاحه في هذا الإصدار). |
| تظهر الترجمة القديمة بعد التعديل مباشرة | امسح الكاش أو استخدم `$manager->reload()`. |
| استخدام `str_starts_with` يسبب خطأ في PHP 7.4 | تم استبدالها بـ `strpos() === 0`. |

---

### ملخص الإصدار (1.0.12)

| الإصدار | أبرز الميزات |
|---------|---------------|
| **Nano.TranslateExtended 1.0.12** | نظام متكامل لتجاوز الترجمات، دعم مسارات متعددة، كاش قابل للتكوين، إعدادات عبر واجهة المستخدم ومتغيرات البيئة، توافق كامل مع `October\Rain\Translation::set()`، معالجة استثنائية وتسجيل أخطاء. |

---

### متطلبات الترقية

1. **تحديث الملفات**:
   - `Plugin.php`
   - `classes/TranslationOverrideManager.php`
   - `models/Settings.php`
   - `config/config.php`
   - `fields.yaml`
   - `version.yaml`

2. **لا توجد هجرات قاعدة بيانات جديدة** – فقط إضافة حقول إعدادات اختيارية.

3. **الاعتماديات**:
   - `RainLab.Translate` (أي إصدار حديث)
   - `OctoberCMS` ≥ 2.0 أو `WinterCMS` ≥ 1.2

4. **إعدادات إضافية موصى بها في `.env`**:
   ```
   NANO_TRANSLATE_ENABLE_OVERRIDES=true
   NANO_TRANSLATE_CACHE_MINUTES=60
   NANO_TRANSLATE_OVERRIDE_PATHS="lang/override,plugins/nano/translateextended/override/lang"
   ```

5. **اختبار التوافق**:
   - إنشاء ملف تجاوز بسيط والتحقق من ظهور الترجمة الجديدة.
   - اختبار تغيير اللغة (`locale.changed`) وتحميل الترجمات تلقائياً.
   - اختبار تفعيل / تعطيل النظام وحفظ الإعدادات.

---

### الخاتمة

يمثل الإصدار **1.0.12** نقلة نوعية في إمكانيات الإضافة، حيث يمنح المطورين القدرة على تعديل أي ترجمة في النظام دون لمس الملفات الأصلية. النظام مصمم ليكون مرناً، عالي الأداء، وسهل الاستخدام، مع دعم كامل لأدوات OctoberCMS المعتادة (الإعدادات، الأحداث، الكاش). يمكن استخدامه لتخصيص واجهات المستخدم، تصحيح ترجمات خاطئة، أو حتى إنشاء ترجمات خاصة بالعميل دون التعارض مع تحديثات الإضافات.

---

**الوثائق المرجعية**:
- [توثيق الإضافة `Nano.TranslateExtended`](./docs/TranslateExtended/Docs-Nano.TranslateExtended-ar.md)
- [توثيق سلوك `DynamicAddIncludeTranslatableApiFields`](./docs/TranslateExtended/Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-ar.md)
- [توثيق سلوك `TranslatableContentCaching`](./docs/TranslateExtended/Behaviors/Docs-TranslatableContentCaching-ar.md)
- [دليل استخدام الترجمات المحسّنة في API](./docs/TranslateExtended/Docs-API-Translations-Guide-ar.md)
- [دليل استخدام تجاوز الترجمات](./docs/TranslateExtended/Docs-Translation-Override-Guide-ar.md)
- [توثيق كلاس `TranslationOverrideManager`](./docs/TranslateExtended/Classes/Docs-TranslationOverrideManager-Class-ar.md)
- [ملف الإعدادات `config.php`](./plugins/nano/translateextended/config/config.php)
- [نموذج الإعدادات `Settings`](./plugins/nano/translateextended/models/Settings.php)
