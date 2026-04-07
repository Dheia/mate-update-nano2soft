# توثيق الأمثلة المتقدمة والعملية لكلاس `FileUploadRegistry`

**Namespace:** `Nano\FileUpload\Classes`  
**الهدف:** تقديم أمثلة تطبيقية شاملة لاستخدام `FileUploadRegistry` في سيناريوهات متنوعة، بدءًا من التسجيل الأساسي للنماذج وحقول الرفع وصولًا إلى التحقق المتقدم من الصلاحيات، وإدارة الأحداث، والاستعلام عن القيود، والتكامل مع `FileUploadService`، مع توضيح المدخلات والمخرجات المتوقعة وأفضل الممارسات.

> **ملاحظة:** يغطي هذا المستند الميزات حتى الإصدار 1.0.7، بما في ذلك الإعدادات العامة للتحكم في التحويلات التلقائية، التخزين المتعدد، خطافات الأحداث، وتحسينات الأمان والتخزين المؤقت.

---

## مقدمة

يقدم هذا الوثيقة مجموعة من الأمثلة العملية التي تغطي مختلف حالات استخدام كلاس `FileUploadRegistry`، الذي يمثل **السجل المركزي** لإدارة تعريفات النماذج وحقول رفع الملفات في تطبيقات نانوسوفت. من خلال هذه الأمثلة، ستتعلم كيفية:

- تسجيل النماذج وحقولها مع إعدادات متقدمة (تخزين متعدد، تحجيم تلقائي، علامة مائية).
- تحديد أنواع المستخدمين المسموح لهم (backend/frontend).
- تعيين صلاحيات دقيقة لكل عملية (`add`, `edit`, `delete`, `view`) على مستوى النموذج أو الحقل.
- استخدام دوال التحقق (`can`, `isUserTypeAllowed`, `getFieldConstraints`, `getProcessingOptions`) لضمان الأمان.
- الاستماع إلى الأحداث (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) لتوسيع النظام.
- استخدام التخزين المؤقت (Cache) لتحسين الأداء.
- دمج `FileUploadRegistry` مع `FileUploadService` لرفع الملفات.
- معالجة الأخطاء وتطبيق أفضل الممارسات.

> **ملاحظة:** الأمثلة هنا تتعامل مباشرة مع الكلاس `FileUploadRegistry`. في التطبيقات الحقيقية، غالبًا ما يتم استخدامه عبر `FileUploadService`، ولكن فهم الإعدادات الأساسية يساعد في تخصيص النظام بدقة.

---

## المتطلبات الأساسية

- إضافة `Nano.FileUpload` مثبتة ومهيأة في تطبيقك (الإصدار 1.0.7 أو أحدث).
- الإضافة تعتمد على `Nano.Api` التي توفر `Base64` و `ApiController`.
- معرفة أساسية بمفهوم النماذج (Models) والعلاقات في Laravel / تطبيقات نانوسوفت.
- (اختياري) استخدام `FileUploadUserManager` للتحقق من صلاحيات المستخدمين.

### إعداد الكلاس في التطبيق

```php
use Nano\FileUpload\Classes\FileUploadRegistry;

$registry = FileUploadRegistry::instance();
```

نظرًا لأن الكلاس يتبع نمط `Singleton`، يتم استدعاؤه عبر `instance()`.

---

## 1. تسجيل نموذج مع إعدادات بسيطة

**السيناريو:** تسجيل نموذج `Product` بحقل `image` واحد (صورة رئيسية) مع إعدادات افتراضية.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'image' => [
            'type' => 'image',
            'label' => 'الصورة الرئيسية',
            'max_filesize' => 1024, // KB
            'allowed_types' => 'jpg,jpeg,png',
            'required' => false,
        ],
    ],
]);
```

**المخرجات:** تم إضافة النموذج إلى `$rawDefinitions`. يمكن التحقق بـ:

```php
if ($registry->isModelRegistered(\Nano\Shop\Models\Product::class)) {
    echo "النموذج مسجل بنجاح";
}
```

**ملاحظات:**  
- `type` يمكن أن يكون `image`، `file`، `audio`، `video`، أو `multiple` (للحقول المتعددة).  
- `max_filesize` بالكيلوبايت (KB).  
- `allowed_types` عبارة عن قائمة امتدادات مفصولة بفواصل أو `*` للسماح بكل الأنواع.  
- إذا لم يتم تحديد `label`، سيتم استخدام اسم الحقل.

---

## 2. تسجيل نموذج مع إعدادات متقدمة (تخزين متعدد وتحويل تلقائي للصور)

**السيناريو:** تسجيل نموذج `Product` بحقل `image` مع:
- قرص تخزين مخصص (مثل `s3`).
- تحجيم تلقائي للصورة إلى عرض 800 وارتفاع 600 (وضعية `crop`).
- إضافة علامة مائية تلقائية في الزاوية السفلية اليمنى بنسبة 15%.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'image' => [
            'type' => 'image',
            'storage_disk' => 's3',                   // قرص تخزين مخصص
            'auto_resize' => true,
            'resize_options' => [
                'width' => 800,
                'height' => 600,
                'mode' => 'crop',
            ],
            'auto_watermark' => true,
            'watermark_options' => [
                'position' => 'bottom-right',
                'resize_percentage' => 15,
            ],
        ],
    ],
]);
```

**التحقق من خيارات المعالجة:**

```php
$procOptions = $registry->getProcessingOptions(\Nano\Shop\Models\Product::class, 'image');
// $procOptions['storage_disk'] => 's3'
// $procOptions['auto_resize'] => true
// $procOptions['resize_options']['width'] => 800
```

**ملاحظات:**  
- يجب أن يكون القرص `s3` معرفاً مسبقاً في `config/filesystems.php`.  
- `auto_resize` و `auto_watermark` يعملان فقط إذا كان نوع الحقل `image`.  
- يمكن تعطيل التحويلات التلقائية عالمياً عبر الإعدادات العامة (انظر المثال 4).

---

## 3. تسجيل نموذج مع إعدادات متقدمة (أنواع المستخدمين والصلاحيات)

**السيناريو:** تسجيل نموذج `User` بحقل `avatar`، مع تحديد أن المستخدمين من نوع `frontend` فقط هم المسموح لهم، وكل عملية تحتاج صلاحية محددة.

```php
$registry->registerModel(\RainLab\User\Models\User::class, [
    'allowed_user_types' => ['frontend'], // فقط مستخدمي الواجهة الأمامية
    'permissions' => [
        'add'    => null, // لا حاجة لصلاحية إضافية (يُسمح للجميع)
        'delete' => 'user.delete_avatar',
        'view'   => null,
    ],
    'fields' => [
        'avatar' => [
            'type' => 'image',
            'max_filesize' => 2048,
            'allowed_types' => 'jpg,jpeg,png',
            'permissions' => [
                'add' => 'user.upload_avatar', // تجاوز صلاحية الإضافة العامة
            ],
        ],
    ],
]);
```

**التحقق من الصلاحية:**

```php
$user = \Auth::getUser(); // مستخدم frontend
$userType = $registry->getUserTypeFromModel($user); // يجب أن تعيد 'frontend'

if ($registry->can(\RainLab\User\Models\User::class, 'add', $userType, $user, 'avatar')) {
    echo "مسموح برفع الصورة";
} else {
    echo "غير مسموح";
}
```

**المخرجات:** إذا كان المستخدم يمتلك صلاحية `user.upload_avatar` (في نظام الصلاحيات الخاص بـ frontend)، سيعيد `true`.  
إذا لم يمتلك، سيعيد `false`.

**ملاحظات:**  
- صلاحيات الحقل تتجاوز صلاحيات النموذج.  
- إذا كانت الصلاحية `null`، يتم السماح تلقائيًا بعد التحقق من نوع المستخدم.

---

## 4. استخدام الإعدادات العامة للتحكم في التحويلات التلقائية

**السيناريو:** تعطيل جميع عمليات التحجيم التلقائي على مستوى API بالكامل دون تعديل تعريفات الحقول.

في ملف `.env`:
```ini
NANO_FILE_UPLOAD_DISABLE_AUTO_RESIZE=true
```

في الكود:
```php
if ($registry->isAutoResizeEnabledGlobally()) {
    echo "التحجيم التلقائي مفعل";
} else {
    echo "التحجيم التلقائي معطل عالمياً";
}
```

**ملاحظة:** نفس الشيء ينطبق على `disable_auto_watermark` و `disable_upload`, `disable_delete`, `disable_get`.

---

## 5. تسجيل حقل متعدد (multiple) مع قيود إضافية

**السيناريو:** إضافة معرض صور `gallery` يسمح برفع حتى 10 صور، كل صورة بحجم أقصى 1 ميجابايت، بصيغ jpg/png فقط.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'gallery' => [
            'type' => 'multiple',
            'label' => 'معرض الصور',
            'max_filesize' => 1024,
            'allowed_types' => 'jpg,jpeg,png',
            'max_files' => 10,
            'multiple' => true,
            'required' => false,
        ],
    ],
]);
```

**الاستعلام عن القيود:**

```php
$constraints = $registry->getFieldConstraints(\Nano\Shop\Models\Product::class, 'gallery');
// $constraints['max_files'] == 10
// $constraints['multiple'] == true
```

---

## 6. استخدام حدث `nano.api.fileupload.registerModels` لتسجيل النماذج من إضافات مختلفة

**السيناريو:** تسجيل نماذج متعددة من خلال مستمع حدث، مما يسمح بتجميع التسجيلات في مكان واحد.

```php
Event::listen('nano.api.fileupload.registerModels', function ($registry) {
    $registry->registerModel(\Nano\Shop\Models\Product::class, [
        'allowed_user_types' => ['backend'],
        'fields' => [
            'image' => ['type' => 'image', 'max_filesize' => 1024],
            'gallery' => ['type' => 'multiple', 'multiple' => true, 'max_files' => 10],
        ],
    ]);

    $registry->registerModel(\RainLab\User\Models\User::class, [
        'allowed_user_types' => ['frontend'],
        'fields' => [
            'avatar' => ['type' => 'image', 'max_filesize' => 2048],
        ],
    ]);
});
```

**ملاحظات:**  
- هذا الحدث يُطلق تلقائيًا بواسطة `FileUploadRegistry` عند استدعاء `getRegisteredModels()`.  
- يمكن وضعه في `boot()` الخاص بالإضافة الرئيسية.

---

## 7. الاستماع لحدث `nano.api.fileupload.modelRegistered` للقيام بإجراءات إضافية بعد التسجيل

**السيناريو:** تسجيل نموذج ثم تسجيل مسار API ديناميكي بناءً على الحقول.

```php
Event::listen('nano.api.fileupload.modelRegistered', function ($modelClass, $config) {
    // يمكن إضافة مسار مخصص للحصول على ملفات هذا النموذج
    \Route::get('/api/custom/' . class_basename($modelClass) . '/files', function () use ($modelClass) {
        return $registry->getFiles($modelClass, 'image');
    });
});
```

---

## 8. التحقق من نوع المستخدم (isUserTypeAllowed) والإعدادات العامة

**السيناريو:** عرض قائمة بالحقول فقط إذا كان المستخدم من النوع المسموح وكان الرفع مفعلاً عالمياً.

```php
$userType = $userManager->getUserType(); // backend, frontend, guest
$modelClass = \Nano\Shop\Models\Product::class;

if ($registry->isUserTypeAllowed($modelClass, $userType) && $registry->isUploadEnabledGlobally()) {
    // عرض واجهة رفع الملفات
    $fields = $registry->getModelConfig($modelClass)['fields'];
    foreach ($fields as $field => $config) {
        echo "حقل: {$field} - نوع: {$config['type']}\n";
    }
} else {
    echo "غير مسموح بالوصول أو الرفع معطل";
}
```

---

## 9. استخدام `can` للتحقق من صلاحية الحذف مع الإعدادات العامة

**السيناريو:** التحقق مما إذا كان المستخدم الحالي يمتلك صلاحية حذف ملف من حقل معين، مع مراعاة الإعدادات العامة.

```php
$user = \BackendAuth::getUser(); // مستخدم backend
$userType = $registry->getUserType(); // 'backend'
$modelClass = \Nano\Shop\Models\Product::class;
$field = 'image';

if ($registry->can($modelClass, 'delete', $userType, $user, $field)) {
    // عرض زر الحذف
} else {
    // إخفاء زر الحذف
}
```

**المخرجات:** إذا كان المستخدم لديه صلاحية `delete` (على مستوى النموذج أو الحقل) وكان الحذف مفعلاً عالمياً (`disable_delete = false`)، سيعيد `true`.

---

## 10. الحصول على قيود الحقل والتحقق منها قبل الرفع

**السيناريو:** في واجهة الرفع، نعرض للمستخدم الحد الأقصى للحجم والأنواع المسموحة، ونتحقق منها قبل إرسال الطلب.

```php
$constraints = $registry->getFieldConstraints(\Nano\Shop\Models\Product::class, 'image');
if ($constraints['max_filesize']) {
    echo "الحد الأقصى للحجم: {$constraints['max_filesize']} KB\n";
}
if ($constraints['allowed_types'] != '*') {
    echo "الأنواع المسموحة: {$constraints['allowed_types']}\n";
}
if ($constraints['required']) {
    echo "هذا الحقل إلزامي\n";
}
```

---

## 11. استخدام التخزين المؤقت (Cache) لتحسين الأداء

**السيناريو:** تفعيل التخزين المؤقت يدوياً (على الرغم من أن الإعدادات الافتراضية مفعلة) وتعيين مدة صلاحية أطول.

```php
$registry->setCacheEnabled(true);
$registry->setCacheTtl(7200); // ساعتين

// بعد تحديث تعريف نموذج، يمكن مسح الكاش الخاص به
$registry->clearModelCache(\Nano\Shop\Models\Product::class);
```

**ملاحظة:** يمكن التحكم في الكاش عالمياً عبر متغيرات البيئة:
```ini
NANO_FILE_UPLOAD_REGISTRY_CACHE_ENABLED=true
NANO_FILE_UPLOAD_REGISTRY_CACHE_TTL=3600
```

---

## 12. دمج `FileUploadRegistry` مع `FileUploadService` في عملية رفع كاملة باستخدام الميزات المتقدمة

**السيناريو:** إنشاء نموذج `Product` جديد، ورفع صورة رئيسية له مع تطبيق التحجيم التلقائي والعلامة المائية كما هو محدد في التسجيل.

```php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadRegistry;

$registry = FileUploadRegistry::instance();
$service = FileUploadService::instance();

$modelClass = \Nano\Shop\Models\Product::class;
$field = 'image';

// التحقق من وجود النموذج والصلاحية
if (!$registry->isModelRegistered($modelClass)) {
    die('النموذج غير مسجل');
}

// إنشاء مفتاح جلسة مؤقت
$tempKey = $service->generateTempSessionKey($modelClass, $field);

// رفع الصورة (بافتراض أن $uploadedFile هو كائن UploadedFile)
$uploadResult = $service->upload($modelClass, $field, $uploadedFile, [
    'temp_session_key' => $tempKey
]);

if ($uploadResult['status']) {
    // إنشاء المنتج وحفظه
    $product = new \Nano\Shop\Models\Product();
    $product->name = 'منتج جديد';
    $product->save();

    // ربط الملفات المؤقتة بالنموذج المحفوظ
    $service->attachTempFiles($product, $field, $tempKey);

    echo "تم رفع الصورة وربطها بالمنتج";
} else {
    echo "فشل الرفع: " . $uploadResult['error'] . " (كود: " . $uploadResult['error_code'] . ")";
}
```

**ملاحظات:**  
- `attachTempFiles` سيبحث عن الملفات التي تحمل `session_key` = `$tempKey` ويحدّث `attachment_id` و `attachment_type` لربطها بالنموذج.  
- إذا كان الحقل `multiple`، ستضاف جميع الملفات إلى العلاقة.

---

## 13. التعامل مع أخطاء التسجيل (نموذج غير موجود)

**السيناريو:** محاولة تسجيل نموذج غير موجود.

```php
try {
    $registry->registerModel('NonExistentModel', []);
} catch (\ApplicationException $e) {
    echo "خطأ: " . $e->getMessage(); // "النموذج NonExistentModel غير موجود"
}
```

---

## 14. تحديث إعدادات نموذج مسجل (مثل تغيير صلاحية الإضافة)

**السيناريو:** بعد تسجيل النموذج، نريد تغيير صلاحية الإضافة للحقل `image` من `null` إلى صلاحية محددة.

```php
$registry->updateModelConfig(\Nano\Shop\Models\Product::class, [
    'fields' => [
        'image' => [
            'permissions' => [
                'add' => 'nano.shop.upload_image_v2',
            ],
        ],
    ],
]);
```

الآن عند استدعاء `can` للحقل `image`، ستُستخدم الصلاحية الجديدة.

---

## 15. استخدام الإعدادات الافتراضية (defaults) لتقليل التكرار

**السيناريو:** العديد من الحقول تشترك في نفس الإعدادات (حجم 2 ميجابايت، أنواع الصور، عامة). يمكن تحديد `defaults` ثم تجاوز الحقول الفردية.

```php
$registry->registerModel(\Nano\Shop\Models\Product::class, [
    'defaults' => [
        'max_filesize' => 2048,
        'allowed_types' => 'jpg,jpeg,png',
        'is_public' => true,
    ],
    'fields' => [
        'image' => [
            'type' => 'image',
            'required' => true,
        ],
        'gallery' => [
            'type' => 'multiple',
            'max_files' => 10,
            'multiple' => true,
        ],
    ],
]);
```

في `image`، سيتم استخدام `max_filesize=2048` و `allowed_types=...`، لكن `required` يُضبط إلى `true`.  
في `gallery`، سيتم استخدام نفس الإعدادات الافتراضية مع إضافة `max_files` و `multiple`.

---

## 16. الحصول على إعدادات النموذج الكامل أو الحقل

**السيناريو:** طباعة جميع إعدادات نموذج معين لأغراض التصحيح.

```php
$config = $registry->getModelConfig(\Nano\Shop\Models\Product::class);
print_r($config);
```

**النتيجة:** مصفوفة تحتوي على `enabled`, `allowed_user_types`, `permissions`, `fields`, `defaults`.

---

## 17. التحقق من صلاحية `view` قبل عرض قائمة الملفات (مع مراعاة الإعدادات العامة)

**السيناريو:** في متحكم API، نتحقق مما إذا كان المستخدم يمتلك صلاحية عرض ملفات نموذج معين، وإذا كان الجلب مفعلاً عالمياً.

```php
public function getFiles(Request $request)
{
    $modelClass = $request->input('model_class');
    $field = $request->input('field');
    $user = \Auth::getUser();
    $userType = $this->userManager->getUserType();

    $registry = FileUploadRegistry::instance();
    if (!$registry->can($modelClass, 'view', $userType, $user, $field)) {
        return response()->json(['error' => 'غير مسموح'], 403);
    }

    // جلب الملفات...
}
```

---

## 18. اختبار صلاحية حقل باستخدام مسار عمود (للتقارير) – لا ينطبق على FileUploadRegistry مباشرة، ولكن يمكن إضافة مثال عن التحقق من صلاحية `view` للحقل نفسه.

**السيناريو:** التحقق من أن المستخدم يمتلك صلاحية رؤية حقل معين قبل عرضه في واجهة المستخدم.

```php
$user = \Auth::getUser();
$userType = 'frontend';
$modelClass = \RainLab\User\Models\User::class;
$field = 'avatar';

if ($registry->can($modelClass, 'view', $userType, $user, $field)) {
    echo '<img src="' . $user->avatar->getPath() . '">';
} else {
    echo 'لا تظهر الصورة';
}
```

---

## 19. أفضل الممارسات للإصدارات الحديثة (1.0.7)

1. **سجل النماذج في وقت مبكر**: استخدم حدث `nano.api.fileupload.registerModels` في `boot()` للإضافة الرئيسية.
2. **استخدم `allowed_user_types` بحكمة**: لا تسمح لجميع الأنواع إذا لم تكن هناك حاجة.
3. **حدد صلاحيات دقيقة لكل عملية**: على الأقل `add` و `delete`، لتجنب رفع أو حذف غير مصرح به.
4. **استخدم `getFieldConstraints` للتحقق المسبق**: قبل رفع الملف، تحقق من الحجم والأنواع لتجنب أخطاء الخادم.
5. **قم بتوثيق إعدادات الحقول**: باستخدام `label` و `description` ليسهل على المطورين الآخرين فهمها.
6. **استخدم الإعدادات الافتراضية (`defaults`)** لتقليل التكرار وتسهيل الصيانة.
7. **لا تعتمد على الصلاحيات العامة فقط**: قم بتخصيص الصلاحيات لكل حقل عند الحاجة.
8. **راقب الأحداث**: `modelRegistered` يمكن استخدامه لتسجيل مسارات API أو تهيئة إضافية.
9. **اختبار الصلاحيات**: تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم رفع الملفات عبر API.
10. **استخدم `FileUploadService` بدلاً من التعامل المباشر مع `FileUploadRegistry` في عمليات الرفع**: لتضمن أن كل التحقق يمر عبر الطبقة الصحيحة.
11. **استخدم الإعدادات العامة (`disable_upload`, `disable_auto_resize`) كإجراء طارئ** لتعطيل العمليات مؤقتاً دون تعديل الكود.
12. **فعّل التخزين المؤقت في بيئة الإنتاج** (`cache.enabled = true`) لتحسين الأداء، مع TTL مناسب.
13. **استخدم `storage_disk` فقط عند الحاجة**، ويفضل استخدام القرص الافتراضي لتبسيط الإدارة.
14. **حدد `auto_resize` و `auto_watermark` على مستوى الحقل** لتجنب المعالجة غير الضرورية للملفات غير الصورية.

---

## 20. الأخطاء الشائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `ApplicationException: النموذج ... غير موجود` | محاولة تسجيل نموذج غير موجود. | تأكد من صحة اسم الفئة واستيرادها. |
| عدم ظهور الحقل في الواجهة | `allowed_user_types` لا يشمل نوع المستخدم الحالي. | تحقق من `$userType` أو أضف النوع المناسب. |
| صلاحية `add` لا تعمل | لم يتم تعيين الصلاحية في `permissions` أو `permissions[add]` غير موجود. | حدد الصلاحية المطلوبة أو اجعلها `null` للسماح للجميع. |
| `isModelRegistered` يعيد `false` | تم استدعاء `getRegisteredModels()` بعد تسجيل النموذج؟ | تأكد من أن التسجيل حدث قبل الاستعلام (الدالة تحمّل التعريفات تلقائياً الآن). |
| الملفات المؤقتة لا تُربط | لم يتم تمرير `temp_session_key` الصحيح إلى `attachTempFiles`. | استخدم نفس المفتاح الذي أعادته `upload`. |
| `Auto-resize failed` | عدم وجود مكتبة GD أو Imagick أو مسار الملف غير صالح. | تأكد من تثبيت GD أو Imagick وتحقق من صلاحيات الملف. |
| `Auto-watermark failed` | إضافة `Nano2.Watermark` غير مثبتة أو مسار الشعار غير صحيح. | ثبّت الإضافة وتأكد من وجود ملف الشعار في المسار المحدد. |
| `Storage disk 's3' not found` | لم يتم تكوين قرص `s3` في `config/filesystems.php`. | أضف إعدادات القرص المناسبة. |
| `التحجيم التلقائي لا يعمل رغم تفعيله` | `disable_auto_resize` مضبوط على `true` في الإعدادات العامة. | تحقق من متغير البيئة أو قم بتعطيله. |

---

## 21. دمج مع نظام الصلاحيات في نانوسوفت

في تطبيقات نانوسوفت، يتم استخدام `FileUploadUserManager` للتحقق من الصلاحيات. يمكنك تعيين دالة مخصصة للتحقق في حالة المستخدمين من نوع `frontend`:

```php
FileUploadUserManager::instance()->setPermissionChecker(function ($user, $operation, $permissions) {
    // منطق مخصص للتحقق من صلاحيات المستخدمين frontend
    // مثلاً: التحقق من جدول الصلاحيات المخصص
    return $user->hasCustomPermission($operation, $permissions);
});
```

---

## الخلاصة

كلاس `FileUploadRegistry` يمثل العمود الفقري لإدارة رفع الملفات في تطبيقات نانوسوفت. من خلال الأمثلة المذكورة أعلاه، يمكن للمطورين تسجيل أي نموذج مع إعدادات مرنة (بما في ذلك التخزين المتعدد والتحويل التلقائي)، والتحكم الدقيق في أنواع المستخدمين والصلاحيات، والاستفادة من التخزين المؤقت والأحداث، ودمج النظام مع `FileUploadService` لرفع الملفات بأمان. نوصي باستخدام الإعدادات الافتراضية والاستفادة من الأحداث لتوسيع النظام، والتحقق من الصلاحيات في كل نقطة دخول لضمان أعلى مستويات الأمان.

للاطلاع على تفاصيل هيكل الاستجابة عند استخدام متحكم `FileUploadController`، راجع [توثيق واجهة API](./Docs-API-Documentation-ar.md).

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
