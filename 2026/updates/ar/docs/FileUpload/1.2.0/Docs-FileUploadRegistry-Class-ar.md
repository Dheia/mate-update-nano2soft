# توثيق كلاس `FileUploadRegistry`

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
        'edit'   => null,   // ⬅️ جديد في الإصدار 1.1.0
        'delete' => null,
        'view'   => null,
    ],
    'fields' => [
        'image' => [
            'type' => 'image', // image, audio, video, file, multiple
            'label' => 'الصورة الرئيسية',
            'max_filesize' => 2048, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'max_files' => null, // فقط للحقول المتعددة
            'is_public' => true,
            'use_caption' => true,
            'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
            'allowed_user_types' => ['backend'],
            'permissions' => [ // صلاحيات خاصة بالحقل (تتجاوز صلاحيات النموذج)
                'add'    => 'nano.shop.upload_image',
                'edit'   => 'nano.shop.edit_image',   // ⬅️ جديد
                'delete' => 'nano.shop.delete_image',
            ],
            // ⬇⬇⬇ الإعدادات المتقدمة (الإصدارات 1.0.6+) ⬇⬇⬇
            'storage_disk'      => null,      // قرص التخزين المخصص (مثل 's3')
            'auto_resize'       => false,     // تفعيل التحجيم التلقائي
            'resize_options'    => [],        // خيارات التحجيم (width, height, mode)
            'auto_watermark'    => false,     // تفعيل العلامة المائية التلقائية
            'watermark_options' => [],        // خيارات العلامة المائية
        ],
    ],
    'defaults' => [ // إعدادات افتراضية للحقول (يمكن تجاوزها لكل حقل)
        'max_filesize' => 2048,
        'allowed_types' => '*',
        'is_public' => true,
        'use_caption' => true,
        'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
    ],
]
```

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

##### `public function isEditEnabledGlobally(): bool` **(جديد في الإصدار 1.1.0)**
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

##### `public function can(string $modelClass, string $operation, string $userType, $user, ?string $field = null): bool`
- **الهدف**: التحقق مما إذا كان المستخدم يمتلك الصلاحية المطلوبة لعملية معينة (add, edit, delete, view) على النموذج أو حقل معين.
- **السلوك**: يتحقق أولاً من الإعدادات العامة (مثل `disable_upload`, `disable_edit`، إلخ)، ثم من وجود النموذج، ثم من نوع المستخدم، ثم من الصلاحيات المحددة في النموذج أو الحقل، ثم يستدعي `FileUploadUserManager::checkPermission`.

#### 4. الحصول على قيود الحقل وخيارات المعالجة المتقدمة

##### `public function getFieldConstraints(string $modelClass, string $field): ?array`
- **الهدف**: استخراج قيود الحقل (الحجم الأقصى، الأنواع المسموحة، إلخ) بصيغة مناسبة للتحقق.
- **الإرجاع**: مصفوفة تحتوي على `max_filesize`, `allowed_types`, `multiple`, `max_files`, `required`, `is_public`, `use_caption`, `thumb_options`, `type`.

##### `public function getProcessingOptions(string $modelClass, string $field): ?array`
- **الهدف**: الحصول على خيارات المعالجة المتقدمة لحقل معين (التخزين، التحجيم، العلامة المائية).
- **الإرجاع**: مصفوفة تحتوي على `storage_disk`, `auto_resize`, `resize_options`, `auto_watermark`, `watermark_options`.

#### 5. إدارة التخزين المؤقت

##### `public function clearModelCache(string $modelClass): void`
- **الهدف**: مسح جميع مفاتيح التخزين المؤقت الخاصة بنموذج معين (مثل `field_config` و `constraints`).

##### `public function clearFieldCache(string $modelClass, string $fieldName): void` **(جديد في الإصدار 1.1.0)**
- **الهدف**: مسح الكاش الخاص بحقل معين في نموذج (يُستخدم داخلياً في `updateFieldConfig`).

##### `public function setCacheEnabled(bool $enabled): void`
- **الهدف**: تفعيل/تعطيل التخزين المؤقت.

##### `public function setCacheTtl(int $ttl): void`
- **الهدف**: تعيين مدة البقاء للذاكرة المؤقتة.

#### 6. تحديث إعدادات نموذج مسجل

##### `public function updateModelConfig(string $modelClass, array $config): self` **(محسّن في الإصدار 1.1.0)**
- **الهدف**: تحديث إعدادات نموذج مسجل مسبقاً (دمج مع الإعدادات الحالية).
- **التحسين**: يقوم الآن بتطبيع النموذج بأكمله باستخدام `normalizeModelDefinition` بعد الدمج، لضمان اكتمال جميع الحقول الافتراضية.

##### `public function updateFieldConfig(string $modelClass, string $fieldName, array $newConfig): self` **(جديد في الإصدار 1.1.0)**
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

#### مثال 1: تسجيل نموذج من داخل إضافة (مع الإعدادات المتقدمة)

```php
// في Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'permissions' => [
                'add'    => 'nano.shop.product.upload',
                'edit'   => 'nano.shop.product.edit',   // جديد
                'delete' => 'nano.shop.product.delete',
            ],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
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
                    'multiple' => true,
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

#### مثال 3: تحديث إعدادات حقل معين (جديد)

```php
$registry = FileUploadRegistry::instance();
// تحديث الحد الأقصى لحجم حقل image إلى 2 ميجابايت
$registry->updateFieldConfig('Nano\Shop\Models\Product', 'image', [
    'max_filesize' => 2048,
]);
// تحديث صلاحية edit للحقل image
$registry->updateFieldConfig('Nano\Shop\Models\Product', 'image', [
    'permissions' => [
        'edit' => 'nano.shop.product.edit_image',
    ],
]);
```

#### مثال 4: التحقق من صلاحية قبل رفع ملف (مع مراعاة الإعدادات العامة)

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();
$userType = $userManager->getUserType();
$modelClass = 'Nano\Shop\Models\Product';
$field = 'image';

if ($registry->can($modelClass, 'add', $userType, $user, $field)) {
    // مسموح برفع الصورة
} else {
    // غير مسموح (قد يكون بسبب الإعدادات العامة أو الصلاحيات)
}
```

#### مثال 5: الحصول على قيود حقل معين

```php
$constraints = $registry->getFieldConstraints('Nano\Shop\Models\Product', 'image');
// $constraints['max_filesize'] => 1024
// $constraints['allowed_types'] => 'jpg,jpeg,png'
```

#### مثال 6: الحصول على خيارات المعالجة المتقدمة

```php
$procOptions = $registry->getProcessingOptions('Nano\Shop\Models\Product', 'image');
// $procOptions['auto_resize'] => true
// $procOptions['storage_disk'] => 's3'
```

#### مثال 7: تحديث إعدادات نموذج مسجل (مثل تغيير صلاحية الإضافة)

```php
$registry->updateModelConfig('Nano\Shop\Models\Product', [
    'permissions' => [
        'add' => 'nano.shop.product.upload_v2',
    ],
]);
```

#### مثال 8: التحقق من نوع المستخدم قبل عرض واجهة الرفع (مع الإعدادات العامة)

```php
$userType = $userManager->getUserType();
if ($registry->isUserTypeAllowed('Nano\Shop\Models\Product', $userType)) {
    // عرض واجهة رفع الصور
} else {
    // إخفاء الواجهة
}
```

#### مثال 9: تعطيل التحجيم التلقائي أو التعديل عالمياً عبر متغيرات البيئة

```ini
NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
NANO_FILE_UPLOAD_DISABLE_EDIT=true
```

ثم في الكود:
```php
if ($registry->isAutoResizeEnabledGlobally()) {
    // التحجيم التلقائي مفعل
} else {
    // معطل
}

if ($registry->isEditEnabledGlobally()) {
    // عملية استبدال الملفات مفعلة
} else {
    // معطلة
}
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `FileUploadService`**: `FileUploadService` يعتمد على `FileUploadRegistry` للحصول على إعدادات النماذج (`getFieldConfig`, `getProcessingOptions`) والتحقق من الصلاحيات (`can`)، بما في ذلك عملية `edit` الجديدة.
- **مع `FileUploadUserManager`**: يستخدم `FileUploadRegistry` كائن `FileUploadUserManager::instance()` للتحقق الفعلي من الصلاحيات عبر `checkPermission()`.
- **مع `Base64` (المنقول إلى داخل الإضافة)**: ليس تفاعل مباشر، لكن `FileUploadService` يستخدم `Base64` للرفع.

---

### أفضل الممارسات

1. **سجل النماذج في بداية التطبيق**: قم بتسجيل جميع النماذج مرة واحدة عند تهيئة الإضافة (في دالة `boot()` أو `register()`).
2. **استخدم حدث `nano.api.fileupload.registerModels`**: لتوحيد طريقة تسجيل النماذج من الإضافات المختلفة.
3. **حدد أنواع المستخدمين المسموح لهم بدقة**: لا تسمح لجميع الأنواع إذا لم تكن هناك حاجة.
4. **حدد صلاحيات لكل عملية**: استخدم الصلاحيات المناسبة (مثل `add`, `edit`, `delete`, `view`) لتقييد الوصول.
5. **استخدم قيود الحقل (`getFieldConstraints`)**: للتحقق المسبق من حجم الملف ونوعه قبل الرفع.
6. **استخدم خيارات المعالجة المتقدمة (`getProcessingOptions`)** لتطبيق التحجيم التلقائي والعلامة المائية.
7. **فعّل التخزين المؤقت في بيئة الإنتاج**: لتحسين الأداء (الإعدادات الافتراضية مفعّلة).
8. **استخدم الإعدادات العامة (`disable_upload`, `disable_edit`, `disable_auto_resize`)** كإجراء طارئ لتعطيل العمليات مؤقتاً.
9. **استخدم `updateFieldConfig` لتعديل حقل واحد فقط** بدلاً من إعادة تسجيل النموذج بأكمله.

---

### الخلاصة

`FileUploadRegistry` هو قلب نظام رفع الملفات في تطبيقات نانوسوفت. يقوم بتجميع كل معرفة النظام عن النماذج التي تحتوي على حقول رفع الملفات، ويوفر واجهة برمجية نظيفة وفعالة للوصول إلى هذه المعلومات والتحقق من الصلاحيات. بفضل دعمه لأنواع المستخدمين المختلفة (`backend`، `frontend`) والصلاحيات الدقيقة لكل عملية (بما في ذلك `edit`)، والإعدادات العامة للتحكم في API والتحويلات التلقائية، والتخزين المؤقت، ودوال التحديث المرنة (`updateModelConfig`, `updateFieldConfig`)، يمكن للمطورين بناء أنظمة رفع ملفات قوية وآمنة دون القلق بشأن الأمان أو الأداء. يتكامل هذا الكلاس بشكل وثيق مع `FileUploadService` و `FileUploadUserManager` ليشكل معاً طبقة متكاملة تدير جميع جوانب رفع الملفات في التطبيق.

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
