## توثيق كلاس `FileUploadRegistry` – الإصدار 1.2.3

**نبذة تعريفية عن حزمة `Nano.FileUpload` ووحدة إدارة رفع الملفات**

هذه المجموعة من الكلاسات (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager` وغيرها) هي جزء من **حزمة `Nano.FileUpload`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **نانوسوفت** بهدف توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano\FileUpload\Classes
```

توفر هذه الوحدة أدوات متقدمة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع ملفات، والتحقق من صلاحيات المستخدمين (بما في ذلك أنواع المستخدمين المختلفة: `backend` و `frontend`)، وإدارة الملفات المرفوعة مؤقتًا قبل حفظ السجل، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة رفع ملفات قابلة للتوسع بسهولة وتلبية احتياجات التطبيقات المتقدمة دون المساس بأمان التطبيق أو أدائه.

---

### مقدمة

كلاس `FileUploadRegistry` هو **السجل المركزي (Central Registry)** لنظام رفع الملفات في تطبيقات نانوسوفت. يقوم هذا الكلاس بإدارة جميع النماذج (الموديولات) المسجلة التي تحتوي على حقول رفع ملفات، ويوفر واجهة موحدة للاستعلام عن إعدادات هذه النماذج والحقول، والتحقق من صلاحيات المستخدمين وأنواعهم قبل تنفيذ العمليات.

ببساطة، `FileUploadRegistry` هو **المكان الذي تعيش فيه جميع تعريفات النماذج وحقول الرفع**، وهو الوسيلة التي تستخدمها بقية مكونات النظام (مثل `FileUploadService`) للوصول إلى معلومات الإعدادات والصلاحيات.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$rawDefinitions` | `array` | التعريفات الخام للنماذج (من الإضافات والـ callbacks). |
| `$builtModels` | `array` | الكائنات الجاهزة للنماذج بعد البناء (تستخدم للتخزين المؤقت). |
| `$loaded` | `bool` | هل تم تحميل التعريفات من الإضافات؟ |
| `$cacheEnabled` | `bool` | تفعيل/تعطيل التخزين المؤقت لنتائج الاستعلامات. |
| `$cacheTtl` | `int` | مدة بقاء الذاكرة المؤقتة بالثواني. |

---

### أهم الطرق (Methods)

#### 1. تسجيل النماذج

##### `public function registerModel(string $modelClass, array $config = []): self`
- **الهدف**: تسجيل نموذج واحد مع إعداداته المخصصة (واجهة مبسطة، تستدعي `registerDefinition`).
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج (مثال: `Nano\Shop\Models\Product`).
  - `$config`: مصفوفة إعدادات النموذج (انظر بنية الإعدادات أدناه).
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.

##### `public function registerDefinition(string $modelClass, array $definition): void`
- **الهدف**: تسجيل تعريف نموذج خام (يُستخدم داخلياً ويمكن استخدامه مباشرة).
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$definition`: تعريف النموذج (نفس بنية `$config` في `registerModel`).
- **السلوك**: يطبق التطبيع (`normalizeModelDefinition`) ويخزن في `$rawDefinitions`، ويمسح الكاش.

##### `public function registerDefinitions(array $definitions): void`
- **الهدف**: تسجيل مجموعة تعريفات دفعة واحدة.

##### `public static function registerCallback(callable $callback): void`
- **الهدف**: تسجيل دالة رد تُنفذ عند تحميل التعريفات (مفيد للإضافات التي لا تستخدم `registerFileUploadFields`).

**بنية الإعدادات (`$config`):**
```php
[
    'enabled' => true,
    'allowed_user_types' => ['backend', 'frontend'],
    'permissions' => [
        'add'    => null,
        'edit'   => null,
        'delete' => null,
        'view'   => null,
    ],
    'disabled_operations' => [], // ⬅️ تعطيل عمليات معينة على مستوى النموذج (جديد في 1.2.1)
    'fields' => [
        'image' => [
            'type' => 'image', // image, audio, video, file, multiple
            'label' => 'الصورة الرئيسية',
            'max_filesize' => 2048, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'max_files' => null,
            'is_public' => true,
            'use_caption' => true,
            'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
            'allowed_user_types' => ['backend'],
            'permissions' => [ // صلاحيات خاصة بالحقل (تتجاوز صلاحيات النموذج)
                'add'    => 'nano.shop.upload_image',
                'edit'   => 'nano.shop.edit_image',
                'delete' => 'nano.shop.delete_image',
            ],
            'disabled_operations' => [], // ⬅️ تعطيل عمليات على مستوى الحقل (جديد في 1.2.1)
            'storage_disk'      => null,
            'auto_resize'       => false,
            'resize_options'    => [],
            'auto_watermark'    => false,
            'watermark_options' => [],
        ],
    ],
    'defaults' => [ // إعدادات افتراضية للحقول
        'max_filesize' => 2048,
        'allowed_types' => '*',
        'is_public' => true,
        'use_caption' => true,
        'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
    ],
]
```

> **ملاحظة حول `disabled_operations`**: يمكن تعطيل عمليات محددة (`add`, `edit`, `delete`, `view`) بشكل مستقل. إذا تم تعطيل عملية على مستوى النموذج، فسيتم منعها لجميع حقوله ما لم يتم تجاوزها على مستوى الحقل. هذه الخاصية مفيدة لتعطيل الرفع مؤقتاً دون تغيير الصلاحيات.

#### 2. الحصول على النماذج المسجلة

##### `public function getRegisteredModels(): array`
- **الهدف**: إرجاع جميع النماذج المسجلة (مع تحميل التعريفات أولاً).

##### `public function isModelRegistered(string $modelClass): bool`
- **الهدف**: التحقق مما إذا كان نموذج معين مسجلاً (يحمّل التعريفات تلقائياً).

##### `public function getModelConfig(string $modelClass): ?array`
- **الهدف**: الحصول على إعدادات نموذج كامل.

##### `public function getFieldConfig(string $modelClass, string $field): ?array`
- **الهدف**: الحصول على إعدادات حقل معين (يدعم التخزين المؤقت).

#### 3. التحقق من الصلاحيات والإعدادات العامة

##### `public function isUploadEnabledGlobally(): bool`
- **الهدف**: التحقق مما إذا كان الرفع مفعلاً عالمياً (من إعدادات `general.disable_upload`).

##### `public function isEditEnabledGlobally(): bool`
- **الهدف**: التحقق مما إذا كان التعديل (استبدال الملفات) مفعلاً عالمياً (من إعدادات `general.disable_edit`).

##### `public function isDeleteEnabledGlobally(): bool`
- **الهدف**: التحقق مما إذا كان الحذف مفعلاً عالمياً.

##### `public function isGetEnabledGlobally(): bool`
- **الهدف**: التحقق مما إذا كان الجلب مفعلاً عالمياً.

##### `public function isAutoResizeEnabledGlobally(): bool`
- **الهدف**: التحقق مما إذا كان التحجيم التلقائي مفعلاً عالمياً (من إعدادات `general.disable_auto_resize`).

##### `public function isAutoWatermarkEnabledGlobally(): bool`
- **الهدف**: التحقق مما إذا كانت العلامة المائية التلقائية مفعلة عالمياً.

##### `public function isUserTypeAllowed(string $modelClass, string $userType): bool`
- **الهدف**: التحقق مما إذا كان نوع المستخدم المحدد مسموحاً له بالوصول إلى النموذج.

#### 4. دوال التحقق المتكاملة (جديدة في الإصدار 1.2.1)

تمت إضافة مجموعة من الدوال التي تفصل مستويات التحقق وتسمح بالتحكم الدقيق في الصلاحيات:

##### `public function canGlobal(string $operation): bool`
- **الهدف**: التحقق مما إذا كانت عملية معينة مفعلة عالمياً (دون التحقق من النموذج أو المستخدم).
- **المعاملات**:
  - `$operation`: العملية (`add`, `edit`, `delete`, `view`).
- **الإرجاع**: `true` إذا كانت العملية مفعلة في الإعدادات العامة، وإلا `false`.

##### `public function validateGlobal(string $operation): void`
- **الهدف**: مثل `canGlobal` ولكن ترمي `FileUploadException` إذا كانت العملية معطلة.
- **الاستثناءات**: `FileUploadException` مع الكود المناسب (`ERR_UPLOAD_DISABLED_GLOBALLY`, `ERR_EDIT_DISABLED_GLOBALLY`, إلخ).

##### `public function canModel(string $modelClass, string $operation): bool`
- **الهدف**: التحقق مما إذا كانت عملية معينة مفعلة على مستوى النموذج (من خلال `disabled_operations`).
- **المعاملات**:
  - `$modelClass`: اسم النموذج.
  - `$operation`: العملية.
- **الإرجاع**: `true` إذا كانت العملية غير معطلة في النموذج، وإلا `false`.

##### `public function validateModel(string $modelClass, string $operation): void`
- **الهدف**: مثل `canModel` ولكن ترمي `FileUploadException` إذا كانت العملية معطلة (مع كود `ERR_MODEL_OPERATION_DISABLED`).

##### `public function canField(string $modelClass, ?string $field, string $operation): bool`
- **الهدف**: التحقق مما إذا كانت عملية معينة مفعلة على مستوى حقل معين (من خلال `disabled_operations` الخاصة بالحقل).
- **المعاملات**:
  - `$modelClass`: اسم النموذج.
  - `$field`: اسم الحقل (إذا كان `null`، تعيد `false`).
  - `$operation`: العملية.
- **الإرجاع**: `true` إذا كانت العملية غير معطلة في الحقل، وإلا `false`.

##### `public function validateField(string $modelClass, ?string $field, string $operation): void`
- **الهدف**: مثل `canField` ولكن ترمي `FileUploadException` إذا كانت العملية معطلة أو الحقل غير موجود.

##### `public function can(string $modelClass, string $operation, string $userType, $user, ?string $field = null, ?bool $isForceField = false): bool`
- **الهدف**: التحقق من صلاحية عملية معينة على كامل السلسلة (عالمي ← موديول ← حقل ← نوع المستخدم ← الصلاحيات المحددة).
- **المعاملات**:
  - `$modelClass`: اسم النموذج.
  - `$operation`: العملية (`add`, `edit`, `delete`, `view`).
  - `$userType`: نوع المستخدم (`backend`, `frontend`, `guest`).
  - `$user`: كائن المستخدم الكامل (للتحقق من الصلاحيات المحددة).
  - `$field`: اسم الحقل (اختياري).
  - `$isForceField`: إذا كان `true`، يتم التحقق من الحقل حتى لو كان `null` (يستخدم في سياقات معينة).
- **الإرجاع**: `true` إذا تم اجتياز جميع المستويات، وإلا `false`.

##### `public function validate(string $modelClass, string $operation, string $userType, $user, ?string $field = null, ?bool $isForceField = false): void`
- **الهدف**: مثل `can` ولكن ترمي `FileUploadException` عند أول فشل، مما يبسط كتابة الكود في طبقة الخدمة.
- **الاستثناءات**: ترمي `FileUploadException` مع الكود المناسب حسب مستوى الفشل.

##### `public function isModelOperationEnabled(string $modelClass, string $operation): bool`
- **الهدف**: التحقق مما إذا كانت العملية مفعلة على مستوى النموذج (دالة مساعدة تستخدمها `canModel`).
- **الإرجاع**: `true` إذا لم يتم تعطيل العملية في `disabled_operations` للنموذج، وإلا `false`.

##### `public function isFieldOperationEnabled(string $modelClass, ?string $field, string $operation): bool`
- **الهدف**: التحقق مما إذا كانت العملية مفعلة على مستوى الحقل (دالة مساعدة تستخدمها `canField`).
- **الإرجاع**: `true` إذا لم يتم تعطيل العملية في `disabled_operations` للحقل، وإلا `false`.

#### 5. الحصول على قيود الحقل وخيارات المعالجة المتقدمة

##### `public function getFieldConstraints(string $modelClass, string $field): ?array`
- **الهدف**: استخراج قيود الحقل (الحجم الأقصى، الأنواع المسموحة، إلخ) بصيغة مناسبة للتحقق.
- **الإرجاع**: مصفوفة تحتوي على `max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`.

##### `public function getProcessingOptions(string $modelClass, string $field): ?array`
- **الهدف**: الحصول على خيارات المعالجة المتقدمة لحقل معين (التخزين، التحجيم، العلامة المائية).
- **الإرجاع**: مصفوفة تحتوي على `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.

#### 6. إدارة التخزين المؤقت

##### `public function clearModelCache(string $modelClass): void`
- **الهدف**: مسح جميع مفاتيح التخزين المؤقت الخاصة بنموذج معين (مثل `field_config` و `constraints`).

##### `public function clearFieldCache(string $modelClass, string $fieldName): void`
- **الهدف**: مسح الكاش الخاص بحقل معين في نموذج (يُستخدم داخلياً في `updateFieldConfig`).

##### `public function setCacheEnabled(bool $enabled): void`
- **الهدف**: تفعيل/تعطيل التخزين المؤقت.

##### `public function setCacheTtl(int $ttl): void`
- **الهدف**: تعيين مدة البقاء للذاكرة المؤقتة.

##### `public function geCacheEnabled(): bool`
- **الهدف**: إرجاع حالة تفعيل التخزين المؤقت (ملاحظة: هناك خطأ مطبعي في اسم الدالة، يجب أن تكون `getCacheEnabled` ولكنها موجودة كما هي للتوافق).

#### 7. تحديث إعدادات نموذج مسجل

##### `public function updateModelConfig(string $modelClass, array $config): self`
- **الهدف**: تحديث إعدادات نموذج مسجل مسبقاً (دمج مع الإعدادات الحالية). يقوم الآن بتطبيع النموذج بأكمله باستخدام `normalizeModelDefinition` بعد الدمج.

##### `public function updateFieldConfig(string $modelClass, string $fieldName, array $newConfig): self`
- **الهدف**: تحديث إعدادات حقل معين دون الحاجة لإعادة تسجيل النموذج بأكمله.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$fieldName`: اسم الحقل المراد تحديثه.
  - `$newConfig`: الإعدادات الجديدة (يتم دمجها مع الإعدادات الحالية).
- **السلوك**:
  1. يتحقق من وجود النموذج والحقل.
  2. يدمج الإعدادات الجديدة مع الحالية باستخدام `array_replace_recursive`.
  3. يطبق التطبيع عبر `normalizeFieldDefinition`.
  4. يحدّث التعريف الخام (`rawDefinitions`).
  5. يعيد تعيين النموذج المبني (`builtModels`).
  6. يمسح الكاش الخاص بهذا الحقل فقط عبر `clearFieldCache`.
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.
- **الاستثناءات**: يرمي `ApplicationException` إذا كان النموذج أو الحقل غير مسجل.

---

### الأحداث (Events)

| اسم الحدث | الوصف |
|----------|-------|
| `nano.api.fileupload.registerModels` | يُطلق عند الحاجة إلى تحميل النماذج المسجلة، ويتم تمرير كائن `FileUploadRegistry` ليتمكن المطورون من إضافة نماذجهم عبر الاستماع لهذا الحدث. |
| `nano.api.fileupload.modelRegistered` | يُطلق بعد تسجيل نموذج بنجاح، ويتم تمرير `$modelClass` و `$config`. |

---

### أمثلة عملية شاملة

#### مثال 1: تسجيل نموذج من داخل إضافة (مع الإعدادات المتقدمة و `disabled_operations`)

```php
// في Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'disabled_operations' => ['delete'], // تعطيل الحذف على مستوى المنتج
            'permissions' => [
                'add'    => 'nano.shop.product.upload',
                'edit'   => 'nano.shop.product.edit',
                'delete' => 'nano.shop.product.delete',
            ],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'disabled_operations' => ['edit'], // تعطيل استبدال الصورة فقط
                    'auto_resize' => true,
                    'resize_options' => ['width' => 800, 'height' => 600, 'mode' => 'crop'],
                    'auto_watermark' => true,
                    'storage_disk' => 's3',
                ],
                'gallery' => [
                    'type' => 'multiple',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'max_files' => 10,
                ],
            ],
        ],
    ];
}
```

#### مثال 2: تسجيل نموذج عبر حدث

```php
Event::listen('nano.api.fileupload.registerModels', function ($registry) {
    $registry->registerModel(\RainLab\User\Models\User::class, [
        'allowed_user_types' => ['frontend'],
        'fields' => [
            'avatar' => [
                'type' => 'image',
                'max_filesize' => 2048,
                'allowed_types' => 'jpg,jpeg,png',
            ],
        ],
    ]);
});
```

#### مثال 3: التحقق من الصلاحية باستخدام `validate` (بدلاً من `can`)

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();
$userType = $userManager->getUserType();

try {
    $registry->validate('Nano\Shop\Models\Product', 'edit', $userType, $user, 'image');
    // مسموح باستبدال الصورة
} catch (FileUploadException $e) {
    // غير مسموح - رسالة الخطأ في $e->getMessage()
}
```

#### مثال 4: التحقق من تفعيل عملية التعديل عالمياً

```php
if ($registry->canGlobal('edit')) {
    // عملية التعديل مفعلة عالمياً
} else {
    // معطلة عبر disable_edit = true
}
```

#### مثال 5: التحقق من تفعيل عملية الرفع على مستوى النموذج

```php
if ($registry->canModel('Nano\Shop\Models\Product', 'add')) {
    // الرفع مفعل على مستوى المنتج (لم يتم تعطيله في disabled_operations)
}
```

#### مثال 6: تحديث إعدادات حقل معين (تغيير الحد الأقصى للحجم)

```php
$registry->updateFieldConfig('Nano\Shop\Models\Product', 'image', [
    'max_filesize' => 2048, // رفع الحد إلى 2 ميجابايت
]);
```

#### مثال 7: تعطيل عملية الحذف مؤقتاً على مستوى النموذج دون تغيير الصلاحيات

```php
$registry->updateModelConfig('Nano\Shop\Models\Product', [
    'disabled_operations' => ['delete'],
]);
```

#### مثال 8: الحصول على قيود حقل معين لعرضها في الواجهة الأمامية

```php
$constraints = $registry->getFieldConstraints('Nano\Shop\Models\Product', 'image');
// يمكن تمرير $constraints['max_filesize'] و $constraints['allowed_types'] إلى JavaScript
```

#### مثال 9: التحقق من نوع المستخدم قبل عرض واجهة الرفع

```php
$userType = $userManager->getUserType();
if ($registry->isUserTypeAllowed('Nano\Shop\Models\Product', $userType)) {
    // عرض واجهة رفع الصور
} else {
    // إخفاء الواجهة
}
```

#### مثال 10: استخدام دالة `can` مع تمرير الحقل فقط (بدون صلاحيات إضافية)

```php
// في حالة عدم وجود صلاحيات محددة في النموذج أو الحقل، تعيد true
if ($registry->can('Nano\Shop\Models\Product', 'add', $userType, $user, 'image')) {
    // مسموح
}
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `FileUploadService`**: `FileUploadService` يعتمد على `FileUploadRegistry` للحصول على إعدادات النماذج (`getFieldConfig`, `getProcessingOptions`) والتحقق من الصلاحيات (`can` و `validate`).
- **مع `FileUploadUserManager`**: يستخدم `FileUploadRegistry` كائن `FileUploadUserManager::instance()` للتحقق الفعلي من الصلاحيات عبر `checkPermission()`.
- **مع `Base64`**: ليس تفاعل مباشر، لكن `FileUploadService` يستخدم `Base64` للرفع.

---

### أفضل الممارسات

1. **سجل النماذج في بداية التطبيق**: قم بتسجيل جميع النماذج مرة واحدة عند تهيئة الإضافة (في دالة `boot()` أو `register()`).
2. **استخدم حدث `nano.api.fileupload.registerModels`**: لتوحيد طريقة تسجيل النماذج من الإضافات المختلفة.
3. **حدد أنواع المستخدمين المسموح لهم بدقة**: لا تسمح لجميع الأنواع إذا لم تكن هناك حاجة.
4. **حدد صلاحيات لكل عملية**: استخدم الصلاحيات المناسبة (مثل `add`, `edit`, `delete`, `view`) لتقييد الوصول.
5. **استخدم `disabled_operations` لتعطيل عمليات مؤقتاً**: أفضل من تعديل الصلاحيات أو إزالة التسجيل.
6. **استخدم قيود الحقل (`getFieldConstraints`)**: للتحقق المسبق من حجم الملف ونوعه قبل الرفع.
7. **استخدم خيارات المعالجة المتقدمة (`getProcessingOptions`)** لتطبيق التحجيم التلقائي والعلامة المائية.
8. **فعّل التخزين المؤقت في بيئة الإنتاج**: لتحسين الأداء (الإعدادات الافتراضية مفعّلة).
9. **استخدم الإعدادات العامة (`disable_upload`, `disable_edit`, `disable_auto_resize`)** كإجراء طارئ لتعطيل العمليات مؤقتاً.
10. **استخدم `updateFieldConfig` لتعديل حقل واحد فقط** بدلاً من إعادة تسجيل النموذج بأكمله.
11. **استخدم `validate` بدلاً من `can` في طبقة الخدمة**: لأنها ترمي استثناءات موحدة وتقلل من الكود المتكرر.

---

### الخلاصة

`FileUploadRegistry` هو قلب نظام رفع الملفات في تطبيقات نانوسوفت. يقوم بتجميع كل معرفة النظام عن النماذج التي تحتوي على حقول رفع الملفات، ويوفر واجهة برمجية نظيفة وفعالة للوصول إلى هذه المعلومات والتحقق من الصلاحيات. بفضل دعمه لأنواع المستخدمين المختلفة (`backend`، `frontend`) والصلاحيات الدقيقة لكل عملية (بما في ذلك `edit`)، والإعدادات العامة للتحكم في API والتحويلات التلقائية، والتخزين المؤقت، ودوال التحديث المرنة (`updateModelConfig`, `updateFieldConfig`)، ودوال التحقق المتكاملة (`canGlobal`, `validate`, `canModel`, `canField`)، يمكن للمطورين بناء أنظمة رفع ملفات قوية وآمنة دون القلق بشأن الأمان أو الأداء. يتكامل هذا الكلاس بشكل وثيق مع `FileUploadService` و `FileUploadUserManager` ليشكل معاً طبقة متكاملة تدير جميع جوانب رفع الملفات في التطبيق.

---

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
