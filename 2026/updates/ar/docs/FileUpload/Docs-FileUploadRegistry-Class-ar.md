## توثيق كلاس `FileUploadRegistry`

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
| `$registeredModels` | `array` | مصفوفة تخزن إعدادات النماذج المسجلة، حيث المفتاح هو اسم الفئة الكامل للنموذج. |

---

### أهم الطرق (Methods)

#### 1. تسجيل النماذج

##### `public function registerModel(string $modelClass, array $config = []): self`
- **الهدف**: تسجيل نموذج واحد مع إعداداته المخصصة.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج (مثال: `Nano\Shop\Models\Product`).
  - `$config`: مصفوفة إعدادات النموذج (انظر بنية الإعدادات أدناه).
- **السلوك**:
  - يتحقق من وجود الفئة (`class_exists`) وإلا يلقي استثناء `ApplicationException`.
  - يدمج الإعدادات الممررة مع الإعدادات الافتراضية (`defaultConfig`).
  - لكل حقل في `fields`، يدمج إعدادات الحقل مع إعدادات النموذج الافتراضية.
  - يخزن الإعدادات النهائية في `$registeredModels` باستخدام `$modelClass` كمفتاح.
  - يطلق حدث `nano.api.fileupload.modelRegistered` بعد التسجيل.
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.

**بنية الإعدادات (`$config`):**
```php
[
    'enabled' => true, // تفعيل/تعطيل النموذج
    'allowed_user_types' => ['backend', 'frontend'], // أنواع المستخدمين المسموح لهم
    'permissions' => [ // صلاحيات عامة للنموذج (تطبق على جميع الحقول ما لم يتم تخصيصها)
        'add'    => null, // يمكن أن تكون صلاحية واحدة (string) أو مصفوفة صلاحيات (array)
        'edit'   => null,
        'delete' => null,
        'view'   => null,
    ],
    'fields' => [ // حقول الرفع
        'image' => [
            'type' => 'image', // file, image, multiple
            'label' => 'الصورة الرئيسية',
            'max_filesize' => 2048, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
            'multiple' => false,
            'max_files' => null, // فقط للحقول المتعددة
            'is_public' => true,
            'use_caption' => true,
            'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
            'allowed_user_types' => ['backend'], // يرث من النموذج إلا إذا تم تخصيصه
            'permissions' => [ // صلاحيات خاصة بالحقل (تتجاوز صلاحيات النموذج)
                'add'    => 'nano.shop.upload_image',
                'delete' => 'nano.shop.delete_image',
            ],
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

##### `public function registerTables(array $tables): void` – (غير موجود في الكلاس الحالي، لكن يمكن إضافته حسب الحاجة)

#### 2. الحصول على النماذج المسجلة

##### `public function getRegisteredModels(): array`
- **الهدف**: إرجاع جميع النماذج المسجلة مع إعداداتها.
- **السلوك**:
  - إذا كانت `$registeredModels` فارغة، يقوم بتحميل النماذج عبر:
    - حدث `nano.api.fileupload.registerModels` (يمكن للإضافات الاستماع له).
    - استدعاء دوال `registerFileUploadFields()` من جميع الإضافات المسجلة.
  - يعيد المصفوفة الكاملة.
- **الإرجاع**: مصفوفة من إعدادات النماذج.

##### `public function isModelRegistered(string $modelClass): bool`
- **الهدف**: التحقق مما إذا كان نموذج معين مسجلاً.
- **الإرجاع**: `true` إذا كان موجوداً، وإلا `false`.

##### `public function getModelConfig(string $modelClass): ?array`
- **الهدف**: الحصول على إعدادات نموذج معين.
- **الإرجاع**: مصفوفة الإعدادات أو `null` إذا لم يكن مسجلاً.

##### `public function getFieldConfig(string $modelClass, string $field): ?array`
- **الهدف**: الحصول على إعدادات حقل معين تابع لنموذج مسجل.
- **الإرجاع**: مصفوفة إعدادات الحقل أو `null` إذا لم يكن موجوداً.

#### 3. التحقق من الصلاحيات

##### `public function isUserTypeAllowed(string $modelClass, string $userType): bool`
- **الهدف**: التحقق مما إذا كان نوع المستخدم المحدد مسموحاً له بالوصول إلى النموذج.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$userType`: نوع المستخدم (`backend`, `frontend`, `guest`).
- **الإرجاع**: `true` إذا كان النوع مسموحاً، وإلا `false`.

##### `public function can(string $modelClass, string $operation, string $userType, $user, ?string $field = null): bool`
- **الهدف**: التحقق مما إذا كان المستخدم يمتلك الصلاحية المطلوبة لعملية معينة (add, edit, delete, view) على النموذج أو حقل معين.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$operation`: العملية (`add`, `edit`, `delete`, `view`).
  - `$userType`: نوع المستخدم (من `FileUploadUserManager`).
  - `$user`: كائن المستخدم الكامل (يُمرر إلى `FileUploadUserManager::checkPermission`).
  - `$field`: (اختياري) اسم الحقل – إذا تم تمريره، يتم التحقق من صلاحيات الحقل أولاً، ثم صلاحيات النموذج.
- **السلوك**:
  - يتحقق أولاً من `isUserTypeAllowed()`.
  - يبحث عن الصلاحيات المطلوبة من الحقل (إن وجد) ثم من النموذج.
  - إذا لم توجد صلاحيات محددة، يُسمح تلقائياً.
  - يستخدم `FileUploadUserManager::checkPermission()` للتحقق الفعلي من الصلاحيات.
- **الإرجاع**: `true` إذا كانت الصلاحية متاحة، وإلا `false`.

#### 4. الحصول على قيود الحقل

##### `public function getFieldConstraints(string $modelClass, string $field): ?array`
- **الهدف**: استخراج قيود الحقل (الحجم الأقصى، الأنواع المسموحة، إلخ) بصيغة مناسبة للتحقق.
- **الإرجاع**: مصفوفة تحتوي على:
```php
[
    'max_filesize' => 2048,
    'allowed_types' => 'jpg,jpeg,png',
    'multiple' => false,
    'max_files' => null,
    'required' => false,
    'is_public' => true,
    'use_caption' => true,
    'thumb_options' => ['mode' => 'crop', 'extension' => 'auto'],
    'type' => 'image',
]
```

#### 5. تحديث إعدادات نموذج مسجل

##### `public function updateModelConfig(string $modelClass, array $config): self`
- **الهدف**: تحديث إعدادات نموذج مسجل مسبقاً.
- **المعاملات**:
  - `$modelClass`: اسم الفئة الكامل للنموذج.
  - `$config`: مصفوفة الإعدادات الجديدة (يتم دمجها مع الإعدادات الحالية باستخدام `array_replace_recursive`).
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.

---

### الأحداث (Events)

| اسم الحدث | الوصف |
|----------|-------|
| `nano.api.fileupload.registerModels` | يُطلق عند الحاجة إلى تحميل النماذج المسجلة، ويتم تمرير كائن `FileUploadRegistry` ليتمكن المطورون من إضافة نماذجهم عبر الاستماع لهذا الحدث. |
| `nano.api.fileupload.modelRegistered` | يُطلق بعد تسجيل نموذج بنجاح، ويتم تمرير `$modelClass` و `$config`. |

---

### أمثلة عملية شاملة

#### مثال 1: تسجيل نموذج من داخل إضافة (Extension)

```php
// في Plugin.php (أو أي كلاس يسجل الإضافة)
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'], // فقط مستخدمي الباك إند
            'permissions' => [
                'add'    => 'nano.shop.product.upload',
                'delete' => 'nano.shop.product.delete',
            ],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                    'required' => false,
                    'permissions' => [
                        'add'    => 'nano.shop.product.upload_image',
                        'delete' => 'nano.shop.product.delete_image',
                    ],
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

#### مثال 3: التحقق من صلاحية قبل رفع ملف

```php
use Nano\FileUpload\Classes\FileUploadRegistry;
use Nano\FileUpload\Classes\FileUploadUserManager;

$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();
$userType = $userManager->getUserType();

$modelClass = 'Nano\Shop\Models\Product';
$field = 'image';

if ($registry->can($modelClass, 'add', $userType, $user, $field)) {
    // مسموح برفع الصورة
} else {
    // غير مسموح
}
```

#### مثال 4: الحصول على قيود حقل معين

```php
$constraints = $registry->getFieldConstraints('Nano\Shop\Models\Product', 'image');
// $constraints['max_filesize'] => 1024
// $constraints['allowed_types'] => 'jpg,jpeg,png'
```

#### مثال 5: تحديث إعدادات نموذج مسجل

```php
$registry->updateModelConfig('Nano\Shop\Models\Product', [
    'permissions' => [
        'add' => 'nano.shop.product.upload_v2', // تغيير صلاحية الإضافة
    ],
]);
```

#### مثال 6: الحصول على جميع أسماء النماذج المسجلة

```php
$modelClasses = array_keys($registry->getRegisteredModels());
// ['Nano\Shop\Models\Product', 'RainLab\User\Models\User', ...]
```

#### مثال 7: التحقق من نوع المستخدم قبل عرض واجهة الرفع

```php
$userType = $userManager->getUserType();
if ($registry->isUserTypeAllowed('Nano\Shop\Models\Product', $userType)) {
    // عرض واجهة رفع الصور
} else {
    // إخفاء الواجهة
}
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `FileUploadService`**: `FileUploadService` يعتمد على `FileUploadRegistry` للحصول على إعدادات النماذج والتحقق من الصلاحيات قبل تنفيذ عمليات الرفع والحذف.
- **مع `FileUploadUserManager`**: يستخدم `FileUploadRegistry` كائن `FileUploadUserManager::instance()` للتحقق الفعلي من الصلاحيات عبر `checkPermission()`.
- **مع `Base64` (من `Nano.Api`)**: ليس تفاعل مباشر، لكن `FileUploadService` يستخدم `Base64` للرفع.

---

### أفضل الممارسات

1. **سجل النماذج في بداية التطبيق**: قم بتسجيل جميع النماذج مرة واحدة عند تهيئة الإضافة (في دالة `boot()` أو `register()`).
2. **استخدم حدث `nano.api.fileupload.registerModels`**: لتوحيد طريقة تسجيل النماذج من الإضافات المختلفة.
3. **حدد أنواع المستخدمين المسموح لهم بدقة**: لا تسمح لجميع الأنواع إذا لم تكن هناك حاجة.
4. **حدد صلاحيات لكل عملية**: استخدم الصلاحيات المناسبة (مثل `add`, `delete`) لتقييد الوصول.
5. **استخدم قيود الحقل (`getFieldConstraints`)**: للتحقق المسبق من حجم الملف ونوعه قبل الرفع.
6. **لا تعتمد على الإعدادات الافتراضية فقط**: قم بتخصيص `max_filesize` و `allowed_types` حسب متطلبات كل حقل.

---

### الخلاصة

`FileUploadRegistry` هو قلب نظام رفع الملفات في تطبيقات نانوسوفت. يقوم بتجميع كل معرفة النظام عن النماذج التي تحتوي على حقول رفع الملفات، ويوفر واجهة برمجية نظيفة وفعالة للوصول إلى هذه المعلومات والتحقق من الصلاحيات. بفضل دعمه لأنواع المستخدمين المختلفة (`backend`، `frontend`) والصلاحيات الدقيقة لكل عملية، يمكن للمطورين بناء أنظمة رفع ملفات قوية وآمنة دون القلق بشأن الأمان أو الأداء. يتكامل هذا الكلاس بشكل وثيق مع `FileUploadService` و `FileUploadUserManager` ليشكل معاً طبقة متكاملة تدير جميع جوانب رفع الملفات في التطبيق.

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)


