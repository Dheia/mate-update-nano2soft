## 2026-04-03 - 2026-04-05

**تحديثات إضافة `Nano.FileUpload` – الإصدارات 1.0.2 إلى 1.0.5**

### ملخص التحديثات

منذ الإصدار الأول للإضافة، واصلنا تطوير `Nano.FileUpload` بناءً على ملاحظات المستخدمين والمتطلبات الأمنية. تضمنت التحديثات في الإصدارات 1.0.2 حتى 1.0.5 تحسينات كبيرة في نظام الصلاحيات، وإعدادات افتراضية لأنواع الملفات، وأمان المفاتيح المؤقتة، والتخزين المؤقت، ودعم تعدد اللغات، وتسجيل الأخطاء، ومنع الملفات الخطيرة.

---

## الإصدار 1.0.2 – تحسينات الأمان والصلاحيات والإعدادات الافتراضية

### أهداف الإصدار

- توفير تحكم عام على مستوى API (تعطيل الرفع، الحذف، الجلب) عبر متغيرات البيئة.
- إضافة إعدادات افتراضية لأنواع الملفات (`image`, `audio`, `video`, `file`) لتقليل التكرار في تسجيل النماذج.
- تعزيز أمان مفاتيح الجلسة المؤقتة بتضمين `userType` والتوقيع باستخدام HMAC-SHA256.
- إضافة التحقق من صحة المفتاح المؤقت قبل ربط الملفات.

### الميزات الجديدة

#### 1. التحكم العام في عمليات API

أضفنا إعدادات عامة في `config.php` تسمح بتعطيل عمليات الرفع، الحذف، أو الجلب على مستوى API بالكامل، مع إمكانية التحكم عبر متغيرات البيئة:

```ini
NANO_FILE_UPLOAD_DISABLE_UPLOAD=false
NANO_FILE_UPLOAD_DISABLE_DELETE=false
NANO_FILE_UPLOAD_DISABLE_GET=false
```

أضفنا دوال مساعدة في `FileUploadRegistry`:
- `isUploadEnabledGlobally()`
- `isDeleteEnabledGlobally()`
- `isGetEnabledGlobally()`

وتضمنت دالة `can()` التحقق من هذه الإعدادات قبل أي عملية.

#### 2. الإعدادات الافتراضية لأنواع الملفات

أضفنا قسم `defaults` في `config.php` مع إعدادات لكل نوع من الملفات:
- `image`: الحجم الأقصى، الأنواع المسموحة، استخدام التسمية، خيارات الصور المصغرة.
- `audio`, `video`, `file` مع إعدادات مماثلة.

عند تسجيل حقل، إذا لم يحدد المطور `max_filesize` أو `allowed_types` أو `use_caption`، يتم استخدام الإعدادات الافتراضية تلقائياً حسب نوع الحقل.

#### 3. تعزيز أمان مفاتيح الجلسة المؤقتة

- تم تعديل `generateTempSessionKey()` لتضمين `userType` و `timestamp` في البيانات المشفرة.
- استخدام `hash_hmac('sha256', $data, $secret)` لتوقيع المفتاح، مما يمنع التلاعب أو التخمين.
- إضافة دالة `validateTempSessionKey()` للتحقق من صحة المفتاح وصلاحيته (مدة صلاحية افتراضية ساعة واحدة).
- في دالة `attachTempFiles()`، يتم التحقق من:
  - صحة المفتاح (التوقيع، الانتهاء).
  - تطابق `modelClass` و `field`.
  - تطابق `userId` و `userType` مع المستخدم الحالي.

هذا يمنع أي مستخدم من الوصول إلى ملفات مؤقتة خاصة بمستخدم آخر.

#### 4. تحديث `FileUploadRegistry::can()` لتضمين التحقق العالمي

أصبحت دالة `can()` تتحقق أولاً من الإعدادات العامة لكل عملية (`add`, `delete`, `view`) قبل التحقق من نوع المستخدم والصلاحيات المحددة.

#### 5. دوال مساعدة إضافية

- `getDefaultConfigForType($type)`: لجلب الإعدادات الافتراضية لنوع معين.
- `applyDefaultsToFieldConfig()`: لتطبيق الإعدادات الافتراضية على إعدادات الحقل.

### الفوائد

- إمكانية تعطيل عمليات API مؤقتاً دون تعديل الكود (مثلاً للصيانة).
- تقليل الكود المتكرر عند تسجيل النماذج.
- أمان أعلى للملفات المؤقتة، ومنع الوصول غير المصرح به.

---

## الإصدار 1.0.3 – تحسين أداء ومرونة `FileUploadRegistry`

### أهداف الإصدار

- إعادة هيكلة `FileUploadRegistry` لتحسين الأداء وقابلية التوسع.
- دعم التخزين المؤقت (Cache) لنتائج استعلامات الحقول والقيود.
- توفير واجهة تسجيل مرنة عبر `registerDefinition()` و `registerCallback()`.
- تطبيع تعريفات النماذج والحقول باستخدام دوال متخصصة.

### الميزات الجديدة

#### 1. فصل التعريفات الخام عن الكائنات الجاهزة

- `$rawDefinitions`: تخزين التعريفات كما هي من الإضافات.
- `$builtModels`: تخزين الكائنات الجاهزة بعد البناء (Lazy Loading).
- تحميل التعريفات من الإضافات مرة واحدة فقط (`$loaded` flag).

#### 2. دعم التخزين المؤقت (Cache)

أضفنا قسم `cache` في `config.php`:

```php
'registry' => [
    'cache' => [
        'enabled' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_ENABLED', true),
        'ttl' => env('NANO_FILE_UPLOAD_REGISTRY_CACHE_TTL', 3600),
    ],
],
```

ودوال:
- `getFieldConfig()` و `getFieldConstraints()` تستخدم `Cache::get()` و `Cache::put()`.
- `clearModelCache()` لمسح الكاش عند تحديث تعريف نموذج.
- `setCacheEnabled()`, `setCacheTtl()` للتحكم البرمجي.

#### 3. طرق تسجيل مرنة

- `registerDefinition($modelClass, array $definition)`: تسجيل تعريف خام.
- `registerDefinitions(array $definitions)`: تسجيل مجموعة.
- `registerCallback(callable $callback)`: تسجيل دالة رد تُنفذ عند التحميل (مثل `ReportsManager`).
- بقيت دالة `registerModel()` للتوافق مع الإصدارات السابقة.

#### 4. تطبيع التعريفات

- `normalizeModelDefinition()`: دمج الإعدادات الافتراضية.
- `normalizeFieldDefinition()`: تطبيق الإعدادات الافتراضية حسب نوع الحقل.
- `getDefaultConfigForType()`: جلب الإعدادات من ملف الإعدادات.

#### 5. تحسين دوال الاستعلام

- `isModelRegistered()` تستدعي `getRegisteredModels()` تلقائياً لتحميل التعريفات.
- `getModelConfig()` تستخدم `buildModel()` لبناء النموذج عند الحاجة.

### الفوائد

- أداء أفضل بفضل التخزين المؤقت (تجنب إعادة معالجة التعريفات في كل طلب).
- مرونة أكبر في تسجيل النماذج (عبر تعريفات خام أو دوال رد).
- هيكل كود أكثر نظافة وقابلية للصيانة.

---

## الإصدار 1.0.4 – تعزيز الأمان والتسجيل والاستثناءات

### أهداف الإصدار

- منع رفع الملفات الخطيرة (PHP, JS, HTML, EXE, إلخ) نهائياً حتى لو كانت في `allowed_types`.
- إنشاء قناة سجل منفصلة لتسجيل محاولات الرفع الفاشلة.
- إنشاء كلاس استثناء مخصص `FileUploadException` مع أكواد خطأ فريدة.
- تحسين رسائل الأخطاء وجعلها قابلة للترجمة (أساس للغة).

### الميزات الجديدة

#### 1. القائمة السوداء للملفات الخطيرة (Blacklist)

أضفنا ثابت `BLACKLISTED_EXTENSIONS` في `FileUploadService` يحتوي على امتدادات خطيرة مثل:
`php, js, html, exe, bat, sh, dll, ...`

ودالة `isBlacklistedExtension()` تتحقق من الامتداد قبل أي تحقق آخر. إذا كان الامتداد محظوراً، يتم رفض الملف فوراً مع رسالة مناسبة.

#### 2. تسجيل محاولات الرفع الفاشلة

- أضفنا قناة سجل جديدة `fileupload` في `config/logging.php` (تمت تهيئتها في `Plugin::boot()`).
- دالة `logFailedAttempt()` تسجل التفاصيل التالية:
  - المستخدم (ID, type, IP, User Agent)
  - النموذج والحقل
  - سبب الفشل (حجم، نوع، قائمة سوداء، صلاحية)
  - معلومات الملف (الاسم، الحجم، النوع، الامتداد)
- تُستدعى هذه الدالة عند حدوث أي خطأ في `validateFile()`.

#### 3. كلاس `FileUploadException`

تم إنشاء كلاس استثناء مخصص يرث من `Exception`، ويوفر:
- أكواد أخطاء رقمية (1000-1999 عام، 2000-2999 للملفات، 3000-3999 للصلاحيات، 4000-4999 للمفاتيح المؤقتة).
- توليد كود خطأ نصي فريد (مثل `FILE_UPLOAD_FILE_SIZE_EXCEEDED`).
- إمكانية إضافة سياق (`context`) للخطأ (مثل الحجم المسموح، الامتداد المرفوع).

#### 4. تحديث `FileUploadService::validateFile()` لتشمل:

- التحقق من القائمة السوداء أولاً.
- التحقق من الحجم والأنواع المسموحة (كما كان).
- رمي `FileUploadException` بدلاً من `ApplicationException`.
- استدعاء `logFailedAttempt()` قبل رمي الاستثناء.

#### 5. تحديث `FileUploadController`:

- دالة `handleException()` تتعرف على `FileUploadException` وتستخرج `errorCode`.
- دالة `errorResponse()` تقبل معامل `$errorCode` وتضيفه إلى الاستجابة.

### الفوائد

- أمان أعلى بمنع الملفات الخطيرة حتى لو تم تكوين `allowed_types` بشكل خاطئ.
- إمكانية تتبع الهجمات أو المشاكل من خلال سجل منفصل.
- توحيد التعامل مع الأخطاء وتوفير أكواد فريدة لتسهيل التعامل من الواجهات الأمامية.

---

## الإصدار 1.0.5 – دعم تعدد اللغات الكامل (Multilingual Support)

### أهداف الإصدار

- جعل جميع رسائل API (نجاح، خطأ) قابلة للترجمة إلى العربية والإنجليزية.
- إضافة مفاتيح ترجمة لكل عملية (`upload`, `uploadMultiple`, `delete`, `getFiles`).
- تحديث المتحكم والخدمة لاستخدام `trans()` بدلاً من النصوص الثابتة.

### الميزات الجديدة

#### 1. إضافة مفاتيح الترجمة إلى `lang.php`

أضفنا قسماً جديداً `public.helpers.upload` يحتوي على مفاتيح مثل:
- `msg_upload_success`
- `msg_upload_multiple_success`
- `msg_delete_success`
- `msg_get_success`
- `msg_permission_denied`
- `msg_validation_failed`
- `msg_upload_disabled`, `msg_delete_disabled`, `msg_get_disabled`
- وغيرها من رسائل الأخطاء الشائعة.

كما أضفنا مفاتيح في قسم `errors` للرسائل التفصيلية مثل `file_size_exceeded`, `file_type_not_allowed`, `file_type_blacklisted`.

#### 2. تحديث `FileUploadController`

- دوال `successResponse()` و `errorResponse()` تستخدم `trans()` مع القيم الافتراضية.
- دوال `upload()`, `uploadMultiple()`, `delete()`, `getFiles()` تستخدم `trans()` في كل رسالة (نجاح أو فشل).
- دالة `getSafeErrorMessage()` تستخدم `trans()` للرسائل العامة.
- في `uploadMultiple()`، يتم تمرير `success_count` و `total` إلى دالة الترجمة.

#### 3. تحديث `FileUploadService`

- استبدال جميع رسائل `ApplicationException` الثابتة بـ `trans()` باستخدام المفاتيح المناسبة.
- في `validateFile()`, يتم استخدام `trans()` في رسائل `FileUploadException` مع تمرير المعاملات (مثل `max_size`, `actual_size`, `extension`, `allowed`).

#### 4. دعم اللغتين العربية والإنجليزية

- ملفات الترجمة موجودة في `lang/ar/lang.php` و `lang/en/lang.php`.
- يمكن للمستخدم تبديل اللغة عبر `app.locale`.

### الفوائد

- تجربة مستخدم محسنة للمتحدثين بالعربية والإنجليزية.
- سهولة إضافة لغات جديدة مستقبلاً.
- توحيد جميع الرسائل في ملفات الترجمة، مما يسهل الصيانة والتعديل.

---

## ملخص الإصدارات (1.0.2 – 1.0.5)

| الإصدار | أبرز الميزات |
|---------|---------------|
| 1.0.2 | تحكم عام في API، إعدادات افتراضية لأنواع الملفات، تعزيز أمان المفاتيح المؤقتة. |
| 1.0.3 | إعادة هيكلة `FileUploadRegistry`، دعم التخزين المؤقت، تسجيل مرن عبر تعريفات ودوال رد. |
| 1.0.4 | قائمة سوداء للملفات الخطيرة، تسجيل محاولات الرفع الفاشلة، كلاس استثناء مخصص. |
| 1.0.5 | دعم كامل لتعدد اللغات (عربي/إنجليزي) لجميع رسائل API. |

---

## الخاتمة

بفضل هذه التحديثات، أصبحت إضافة `Nano.FileUpload` أكثر أماناً، أداءً، ومرونة. توفر الآن:
- تحكم دقيق في الصلاحيات على مستويات متعددة.
- إعدادات افتراضية ذكية تقلل من تكرار الكود.
- أمان عالٍ للملفات المؤقتة ولمنع الملفات الخطيرة.
- تسجيل شامل للأخطاء لمراقبة المحاولات الضارة.
- واجهة API موحدة مع رسائل قابلة للترجمة.

هذه التحسينات تجعل الإضافة جاهزة للاستخدام في أكبر المشاريع، مع إمكانية التوسع مستقبلاً.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/FileUpload/Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./docs/FileUpload/Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./docs/FileUpload/Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./docs/FileUpload/Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/FileUpload/Docs-API-Documentation-ar.md)
