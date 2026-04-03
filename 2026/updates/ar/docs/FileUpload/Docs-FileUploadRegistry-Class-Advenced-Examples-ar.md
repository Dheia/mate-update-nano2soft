# توثيق الأمثلة المتقدمة والعملية لكلاس `FileUploadRegistry`

**Namespace:** `Nano\FileUpload\Classes`  
**الهدف:** تقديم أمثلة تطبيقية شاملة لاستخدام `FileUploadRegistry` في سيناريوهات متنوعة، بدءًا من التسجيل الأساسي للنماذج وحقول الرفع وصولًا إلى التحقق المتقدم من الصلاحيات، وإدارة الأحداث، والاستعلام عن القيود، والتكامل مع `FileUploadService`، مع توضيح المدخلات والمخرجات المتوقعة وأفضل الممارسات.

---

## مقدمة

يقدم هذا الوثيقة مجموعة من الأمثلة العملية التي تغطي مختلف حالات استخدام كلاس `FileUploadRegistry`، الذي يمثل **السجل المركزي** لإدارة تعريفات النماذج وحقول رفع الملفات في تطبيقات نانوسوفت. من خلال هذه الأمثلة، ستتعلم كيفية:

- تسجيل النماذج وحقولها مع إعدادات متقدمة.
- تحديد أنواع المستخدمين المسموح لهم (backend/frontend).
- تعيين صلاحيات دقيقة لكل عملية (add, edit, delete, view) على مستوى النموذج أو الحقل.
- استخدام دوال التحقق (`can`, `isUserTypeAllowed`, `getFieldConstraints`) لضمان الأمان.
- الاستماع إلى الأحداث (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) لتوسيع النظام.
- دمج `FileUploadRegistry` مع `FileUploadService` لرفع الملفات.
- معالجة الأخطاء وتطبيق أفضل الممارسات.

> **ملاحظة:** الأمثلة هنا تتعامل مباشرة مع الكلاس `FileUploadRegistry`. في التطبيقات الحقيقية، غالبًا ما يتم استخدامه عبر `FileUploadService`، ولكن فهم الإعدادات الأساسية يساعد في تخصيص النظام بدقة.

---

## المتطلبات الأساسية

- إضافة `Nano.FileUpload` مثبتة في تطبيقك.
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

**المخرجات:** تم إضافة النموذج إلى `$registeredModels`. يمكن التحقق بـ:

```php
if ($registry->isModelRegistered(\Nano\Shop\Models\Product::class)) {
    echo "النموذج مسجل بنجاح";
}
```

**ملاحظات:**  
- `type` يمكن أن يكون `image`، `file`، أو `multiple` (للحقول المتعددة).  
- `max_filesize` بالميغابايت (KB).  
- `allowed_types` عبارة عن قائمة امتدادات مفصولة بفواصل أو `*` للسماح بكل الأنواع.  
- إذا لم يتم تحديد `label`، سيتم استخدام اسم الحقل.

---

## 2. تسجيل نموذج مع إعدادات متقدمة (أنواع المستخدمين والصلاحيات)

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

## 3. تسجيل حقل متعدد (multiple) مع قيود إضافية

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

## 4. استخدام حدث `nano.api.fileupload.registerModels` لتسجيل النماذج من إضافات مختلفة

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

## 5. الاستماع لحدث `nano.api.fileupload.modelRegistered` للقيام بإجراءات إضافية بعد التسجيل

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

## 6. التحقق من نوع المستخدم (isUserTypeAllowed)

**السيناريو:** عرض قائمة بالحقول فقط إذا كان المستخدم من النوع المسموح.

```php
$userType = $userManager->getUserType(); // backend, frontend, guest
$modelClass = \Nano\Shop\Models\Product::class;

if ($registry->isUserTypeAllowed($modelClass, $userType)) {
    // عرض واجهة رفع الملفات
    $fields = $registry->getModelConfig($modelClass)['fields'];
    foreach ($fields as $field => $config) {
        echo "حقل: {$field} - نوع: {$config['type']}\n";
    }
} else {
    echo "غير مسموح بالوصول";
}
```

---

## 7. استخدام `can` للتحقق من صلاحية الحذف

**السيناريو:** التحقق مما إذا كان المستخدم الحالي يمتلك صلاحية حذف ملف من حقل معين.

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

**المخرجات:** إذا كان المستخدم لديه صلاحية `delete` (على مستوى النموذج أو الحقل) سيعيد `true`.

---

## 8. الحصول على قيود الحقل والتحقق منها قبل الرفع

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

## 9. دمج `FileUploadRegistry` مع `FileUploadService` في عملية رفع كاملة

**السيناريو:** إنشاء نموذج `Product` جديد، ورفع صورة رئيسية له باستخدام مفتاح جلسة مؤقت.

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
    // الآن إنشاء النموذج (مثلاً: $product = new Product(); $product->save();)
    $product = new \Nano\Shop\Models\Product();
    $product->name = 'منتج جديد';
    $product->save();

    // ربط الملفات المؤقتة بالنموذج المحفوظ
    $service->attachTempFiles($product, $field, $tempKey);

    echo "تم رفع الصورة وربطها بالمنتج";
} else {
    echo "فشل الرفع: " . $uploadResult['error'];
}
```

**ملاحظات:**  
- `attachTempFiles` سيبحث عن الملفات التي تحمل `session_key` = `$tempKey` ويحدّث `attachment_id` و `attachment_type` لربطها بالنموذج.  
- إذا كان الحقل `multiple`، ستضاف جميع الملفات إلى العلاقة.

---

## 10. التعامل مع أخطاء التسجيل (نموذج غير موجود)

**السيناريو:** محاولة تسجيل نموذج غير موجود.

```php
try {
    $registry->registerModel('NonExistentModel', []);
} catch (\ApplicationException $e) {
    echo "خطأ: " . $e->getMessage(); // "النموذج NonExistentModel غير موجود"
}
```

---

## 11. تحديث إعدادات نموذج مسجل

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

## 12. استخدام الإعدادات الافتراضية (defaults) لتقليل التكرار

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

## 13. الحصول على إعدادات النموذج الكامل أو الحقل

**السيناريو:** طباعة جميع إعدادات نموذج معين لأغراض التصحيح.

```php
$config = $registry->getModelConfig(\Nano\Shop\Models\Product::class);
print_r($config);
```

**النتيجة:** مصفوفة تحتوي على `enabled`, `allowed_user_types`, `permissions`, `fields`, `defaults`.

---

## 14. التحقق من صلاحية `view` قبل عرض قائمة الملفات

**السيناريو:** في متحكم API، نتحقق مما إذا كان المستخدم يمتلك صلاحية عرض ملفات نموذج معين.

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

## 15. اختبار صلاحية حقل باستخدام مسار عمود (للتقارير) – لا ينطبق على FileUploadRegistry مباشرة، ولكن يمكن إضافة مثال عن التحقق من صلاحية `view` للحقل نفسه.

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

## 16. أفضل الممارسات

1. **سجل النماذج في وقت مبكر**: استخدم حدث `nano.api.fileupload.registerModels` في `boot()` للإضافة الرئيسية.
2. **استخدم `allowed_user_types` بحكمة**: لا تسمح لجميع الأنواع إذا لم تكن هناك حاجة.
3. **حدد صلاحيات دقيقة لكل عملية**: على الأقل `add` و `delete`، لتجنب رفع أو حذف غير مصرح به.
4. **استخدم `getFieldConstraints` للتحقق المسبق**: قبل رفع الملف، تحقق من الحجم والأنواع لتجنب أخطاء الخادم.
5. **قم بتوثيق إعدادات الحقول**: باستخدام `label` و `description` ليسهل على المطورين الآخرين فهمها.
6. **استخدم الإعدادات الافتراضية (`defaults`)** لتقليل التكرار وتسهيل الصيانة.
7. **لا تعتمد على الصلاحيات العامة فقط**: قم بتخصيص الصلاحيات لكل حقل عند الحاجة.
8. **راقب الأحداث**: `modelRegistered` يمكن استخدامه لتسجيل مسارات API أو تهيئة إضافية.
9. **اختبار الصلاحيات**: تأكد من أن المستخدمين غير المسموح لهم لا يمكنهم رفع الملفات عبر API.
10. **استخدم `FileUploadService` بدلاً من التعامل المباشر مع `FileUploadRegistry` في عمليات الرفع**: لتضمن أن كل التحقق يمر عبر الطبقة الصحيحة.

---

## 17. الأخطاء الشائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `ApplicationException: النموذج ... غير موجود` | محاولة تسجيل نموذج غير موجود. | تأكد من صحة اسم الفئة واستيرادها. |
| عدم ظهور الحقل في الواجهة | `allowed_user_types` لا يشمل نوع المستخدم الحالي. | تحقق من `$userType` أو أضف النوع المناسب. |
| صلاحية `add` لا تعمل | لم يتم تعيين الصلاحية في `permissions` أو `permissions[add]` غير موجود. | حدد الصلاحية المطلوبة أو اجعلها `null` للسماح للجميع. |
| `isModelRegistered` يعيد `false` | تم استدعاء `getRegisteredModels()` بعد تسجيل النموذج؟ | تأكد من أن التسجيل حدث قبل الاستعلام. |
| الملفات المؤقتة لا تُربط | لم يتم تمرير `temp_session_key` الصحيح إلى `attachTempFiles`. | استخدم نفس المفتاح الذي أعادته `upload`. |

---

## 18. دمج مع نظام الصلاحيات في نانوسوفت

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

كلاس `FileUploadRegistry` يمثل العمود الفقري لإدارة رفع الملفات في تطبيقات نانوسوفت. من خلال الأمثلة المذكورة أعلاه، يمكن للمطورين تسجيل أي نموذج مع إعدادات مرنة، والتحكم الدقيق في أنواع المستخدمين والصلاحيات، ودمج النظام مع `FileUploadService` لرفع الملفات بأمان. نوصي باستخدام الإعدادات الافتراضية والاستفادة من الأحداث لتوسيع النظام، والتحقق من الصلاحيات في كل نقطة دخول لضمان أعلى مستويات الأمان.

للاطلاع على تفاصيل هيكل الاستجابة عند استخدام متحكم `FileUploadController`، راجع [توثيق واجهة API](./Docs-API-Documentation-ar.md).

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)

