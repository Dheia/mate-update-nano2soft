## 📘 دليل استخدام تجاوز الترجمات (Translation Override Guide)

**الإصدار:** 1.0.12  
**الإضافة:** Nano.TranslateExtended  
**الغرض:** تخصيص أي ترجمة في النظام دون تعديل الملفات الأصلية للإضافات أو النظام الأساسي.

---

### 1. ما هو تجاوز الترجمات؟

تجاوز الترجمات هو آلية تتيح لك استبدال أي نص مترجم (مثل أزرار، عناوين، رسائل) يأتي من إضافة معينة أو من النظام نفسه، وذلك عن طريق وضع ملفات ترجمة مخصصة في مجلدات محددة. هذه الملفات لا تتأثر بتحديثات الإضافات، مما يحمي تخصيصاتك.

**مثال:**  
إذا كانت إضافة `nano.sitemanager` تعرض زرًا مكتوبًا عليه "إدارة المتجر" وتريد تغييره إلى "إدارة الفرع"، يمكنك عمل ذلك عبر ملف تجاوز دون لمس ملفات الإضافة الأصلية.

---

### 2. متطلبات التشغيل

- OctoberCMS أو WinterCMS الإصدار 2.0+.
- إضافة `RainLab.Translate` مثبتة ومفعّلة.
- إضافة `Nano.TranslateExtended` مثبتة ومفعّلة (الإصدار 1.0.12 أو أحدث).
- صلاحية الوصول إلى لوحة التحكم لتعديل إعدادات الإضافة (اختياري، يمكن استخدام `.env` بدلاً من ذلك).

---

### 3. تفعيل النظام

#### الطريقة الأولى: عبر لوحة التحكم (موصى بها للمواقع الإنتاجية)

1. اذهب إلى **الإعدادات** ← **Translate Extended** (تجدها ضمن فئة RainLab.Translate).
2. في علامة التبويب العامة، ابحث عن قسم **تجاوز الترجمات**.
3. فعّل الخيار **تفعيل تجاوز الترجمات**.
4. حدد **مدة التخزين المؤقت** (بالدقائق). القيمة 60 مناسبة للمواقع العادية. إذا كنت في بيئة تطوير، يمكنك وضع 0 لتعطيل الكاش.
5. في حقل **مسارات التجاوز**، اترك المسارات الافتراضية (`lang/override` و `plugins/nano/translateextended/override/lang`) أو أضف مسارًا إضافيًا خاصًا بك (كل مسار في سطر جديد).
6. احفظ الإعدادات.

#### الطريقة الثانية: عبر متغيرات البيئة (`.env`)

أضف الأسطر التالية إلى ملف `.env` في جذر المشروع:

```ini
NANO_TRANSLATE_ENABLE_OVERRIDES=true
NANO_TRANSLATE_CACHE_MINUTES=60
NANO_TRANSLATE_OVERRIDE_PATHS="lang/override,plugins/nano/translateextended/override/lang"
```

> **ملاحظة:** متغيرات البيئة لها الأولوية القصوى على أي إعدادات في قاعدة البيانات أو ملف `config.php`.

#### الطريقة الثالثة: عبر ملف `config.php`

إذا كنت لا تريد استخدام قاعدة البيانات ولا `.env`، يمكنك تعديل ملف الإعدادات الخاص بالإضافة:

`plugins/nano/translateextended/config/config.php`

```php
'enable_overrides' => true,
'override_cache_minutes' => 60,
'override_paths' => [
    base_path('lang/override'),
    plugins_path('nano/translateextended/override/lang'),
],
```

> **ملاحظة:** الإعدادات في قاعدة البيانات لها الأولوية على ملف `config.php` إذا كانت موجودة.

---

### 4. إنشاء ملفات التجاوز

#### 4.1. هيكل المجلدات

افتراضيًا، يبحث النظام في مسارين:

- **مسار عام للمشروع:** `lang/override/{locale}/`
- **مسار داخل الإضافة:** `plugins/nano/translateextended/override/lang/{locale}/`

يمكنك استخدام أي منهما. يفضل استخدام المسار العام لتجميع كل التخصيصات في مكان واحد.

#### 4.2. إنشاء الملفات

لنفترض أنك تريد تجاوز ترجمات للغة العربية (`ar`) والإنجليزية (`en`).

**الخطوات:**

1. أنشئ المجلدات:
   ```bash
   mkdir -p lang/override/ar
   mkdir -p lang/override/en
   ```

2. داخل كل مجلد، أنشئ ملفًا باسم `.php` (يمكنك تسميته كيف شئت، مثل `custom.php` أو `overrides.php`).

3. اكتب محتوى الملف كمصفوفة PHP.

**مثال لملف `lang/override/ar/custom.php`**:
```php
<?php

return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'إدارة متجر آخر',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'ممتاز',
    'tss.inventory::lang.products.form.use_case_options.good' => 'جيد',
];
```

**مثال لملف `lang/override/en/custom.php`**:
```php
<?php

return [
    'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'Manage Another Store',
    'tss.inventory::lang.products.form.use_case_options.excellent' => 'Excellent',
    'tss.inventory::lang.products.form.use_case_options.good' => 'Good',
];
```

#### 4.3. تنسيق المفاتيح

- **مفتاح عادي:** `'messages.welcome'` (بدون `::`) – يخص الترجمة العامة في مجلد `lang/xx/`.
- **مفتاح بمساحة اسم:** `'nano.sitemanager::lang.site_switcher.btn.manage_shop'` – يخص إضافة معينة (`nano.sitemanager`)، ملف `lang`، المجموعة `site_switcher`، والمفتاح `btn.manage_shop`.

> **هام:** استخدم نفس المفتاح الذي تستخدمه في `Lang::get()` تمامًا.

#### 4.4. استخدام المصفوفات المتداخلة (اختياري)

يمكنك تنظيم الترجمات هرميًا:

```php
<?php

return [
    'nano' => [
        'sitemanager' => [
            'lang' => [
                'site_switcher' => [
                    'btn' => [
                        'manage_shop' => 'إدارة متجر آخر'
                    ]
                ]
            ]
        ]
    ]
];
```

النظام يقوم بتسليط المصفوفة إلى `dot.notation` تلقائيًا.

---

### 5. كيفية عمل التجاوز (من الداخل)

بمجرد تحميل الصفحة، يقوم `TranslationOverrideManager` بما يلي:

1. يقرأ الإعدادات (env → db → config).
2. يبحث في كل المسارات المحددة عن ملفات `.php` داخل مجلد اللغة الحالية.
3. يسطح المصفوفات المتداخلة إلى مفاتيح بصيغة `dot.notation`.
4. يدمج كل التجاوزات في مترجم October عبر `$translator->set($key, $value, $locale)`.
5. يخزن النتيجة في الكاش للمدة المحددة.
6. عند أي استدعاء لـ `Lang::get($key)`، يعيد المترجم القيمة المتجاوزة بدلاً من الأصلية.

---

### 6. إدارة الكاش (Cache)

#### 6.1. متى يتم مسح الكاش تلقائيًا؟

- عند حفظ إعدادات الإضافة (من خلال صفحة الإعدادات).
- عند تغيير اللغة (حدث `locale.changed`).

#### 6.2. مسح الكاش يدويًا

**أمر Artisan:**
```bash
php artisan cache:clear
```
> هذا يمسح كل كاش التطبيق، وليس فقط ترجمات التجاوز.

**من الكود:**
```php
$manager = app('translateextended.translationOverride');
$manager->clearCache();        // يمسح كل اللغات
$manager->clearCache('ar');    // يمسح العربية فقط
```

#### 6.3. تعطيل الكاش مؤقتًا (للتطوير)

في `.env`:
```ini
NANO_TRANSLATE_CACHE_MINUTES=0
```
أو في الإعدادات: ضع `override_cache_minutes = 0`.

---

### 7. أمثلة عملية لتجاوز الترجمات

#### مثال 1: تغيير نص زر "إدارة المتجر" في إضافة SiteManager

**الترجمة الأصلية:** `nano.sitemanager::lang.site_switcher.btn.manage_shop` ← قيمة افتراضية "إدارة المتجر".

**نريد تغييرها إلى:** "إدارة الفرع".

**الخطوات:**

1. أنشئ ملف `lang/override/ar/custom.php`.
2. أضف:
   ```php
   <?php return [
       'nano.sitemanager::lang.site_switcher.btn.manage_shop' => 'إدارة الفرع',
   ];
   ```
3. امسح الكاش (إذا كان مفعلاً).
4. افتح الصفحة التي يظهر فيها الزر – سترى النص الجديد.

#### مثال 2: إضافة خيار جديد في قائمة منسدلة (use_case_options) في إضافة Inventory

**المفتاح:** `tss.inventory::lang.products.form.use_case_options.excellent` (غير موجود في الأصل، لكننا نضيفه).

**الملف:**
```php
'tss.inventory::lang.products.form.use_case_options.excellent' => 'ممتاز',
```

ثم في الكود (أو القالب) يمكنك استخدام:
```php
Lang::get('tss.inventory::lang.products.form.use_case_options.excellent')
```
وسيظهر "ممتاز".

#### مثال 3: تجاوز ترجمة عامة في النظام (بدون مساحة اسم)

افترض أن النظام يستخدم `'backend::lang.form.save'` وتريد تغييرها إلى "حفظ التغييرات".

```php
'backend::lang.form.save' => 'حفظ التغييرات',
```
(مع مساحة اسم `backend`، المفتاح كامل).

---

### 8. استكشاف الأخطاء وإصلاحها

| المشكلة | الحل |
|--------|------|
| **التجاوز لا يعمل رغم وضع الملفات** | 1. تأكد من تفعيل `enable_overrides`. 2. تحقق من صحة المسار (هل المجلد `lang/override/ar` موجود؟). 3. امسح الكاش (`php artisan cache:clear`). 4. راجع `storage/logs/system.log` بحثًا عن أخطاء. |
| **ظهور الترجمة القديمة بعد تعديل الملف** | بسبب الكاش. إما انتظر حتى انتهاء المدة أو امسح الكاش يدويًا. |
| **الخطأ `Too few arguments to function ...`** | حدث `locale.changed` في الإصدارات السابقة كان يمرر وسيطين. الإصدار 1.0.12 قد أصلح ذلك. تأكد من تحديث `Plugin.php`. |
| **الملف لا يُقرأ رغم وجوده** | تأكد من أن امتداد الملف `.php` وأنه لا يحتوي على أخطاء نحوية (Syntax Error). جرب تشغيله مباشرة `php -l custom.php`. |
| **التجاوز يعمل في الواجهة الأمامية لكن ليس في الخلفية (Backend)** | في الخلفية، قد يتم تعطيل الترجمة مؤقتًا لتسهيل التحرير. هذه طبيعية. إذا أردت فرض الترجمة في الباكند، يمكنك استخدام `$manager->loadForLocale($locale)` مباشرة. |
| **تظهر قيمة `null` أو سلسلة فارغة** | تأكد من أن المفتاح صحيح تمامًا (حساس لحالة الأحرف). جرب طباعة `Lang::get('your.key')` لترى ما إذا كان يعيد المفتاح نفسه (يعني غير موجود) أو قيمة فارغة. |

---

### 9. نصائح متقدمة

#### 9.1. إضافة مسار إضافي ديناميكيًا

في ملف `Plugin.php` أو في أي مكان بعد التهيئة:

```php
use Nano\TranslateExtended\Classes\TranslationOverrideManager;

$manager = app(TranslationOverrideManager::class);
$manager->addPath(base_path('my/custom/overrides'));
```

#### 9.2. إعادة تحميل الترجمات بعد تغيير الملفات دون انتظار الكاش

```php
$manager->reload('ar');
```

#### 9.3. استخدام التجاوز مع ترجمات JSON (مثل الحقول القابلة للترجمة في API)

يمكنك تجاوز قيم الحقول التي تُرجع عبر API بنفس الطريقة، لأن `Lang::get` يُستخدم في الـ Transformers أيضًا.

#### 9.4. دمج التجاوز مع `TranslatableContentCaching`

إذا كنت تستخدم سلوك `TranslatableContentCaching` للحصول على ترجمات متعددة في API، فإن التجاوز يعمل أيضًا على تلك القيم، لأن الدوال الداخلية تستخدم `Lang::get`.

---

### 10. الخلاصة

نظام تجاوز الترجمات في `Nano.TranslateExtended` يمثل حلاً أنيقًا وقويًا لتخصيص نصوص واجهات المستخدم دون المساس بملفات الإضافات. بفضل مرونته (دعم مسارات متعددة، كاش، إعدادات عبر UI و env)، يمكنك تطبيقه بسهولة في أي مشروع، سواء كان صغيرًا أو كبيرًا، مع ضمان احتفاظ تخصيصاتك بعد تحديث الإضافات.

**الخطوات السريعة للبدء:**

1. فعّل النظام في الإعدادات أو `.env`.
2. أنشئ مجلد `lang/override/{locale}/`.
3. ضع ملفًا `.php` يعيد مصفوفة المفاتيح والقيم المطلوب تجاوزها.
4. امسح الكاش.
5. استمتع بالترجمات الجديدة.

---

**المراجع:**

- [توثيق الإضافة `Nano.TranslateExtended`](./Docs-Nano.TranslateExtended-ar.md)
- [توثيق سلوك `DynamicAddIncludeTranslatableApiFields`](./Behaviors/Docs-DynamicAddIncludeTranslatableApiFields-ar.md)
- [توثيق سلوك `TranslatableContentCaching`](./Behaviors/Docs-TranslatableContentCaching-ar.md)
- [دليل استخدام الترجمات المحسّنة في API](./Docs-API-Translations-Guide-ar.md)
- [توثيق كلاس `TranslationOverrideManager`](./Classes/Docs-TranslationOverrideManager-Class-ar.md)