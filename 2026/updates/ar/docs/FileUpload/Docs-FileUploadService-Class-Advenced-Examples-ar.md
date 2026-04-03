# توثيق الأمثلة المتقدمة والعملية لكلاس `FileUploadService`

**Namespace:** `Nano\FileUpload\Classes`  
**الهدف:** تقديم أمثلة تطبيقية شاملة لاستخدام `FileUploadService` في سيناريوهات متنوعة، بدءًا من العمليات الأساسية لرفع الملفات وصولًا إلى التعامل مع الملفات المؤقتة، والربط بالنماذج، والتحقق من الصلاحيات والقيود، مع توضيح المدخلات والمخرجات المتوقعة وأفضل الممارسات.

---

## مقدمة

يقدم هذا الوثيقة مجموعة من الأمثلة العملية التي تغطي مختلف حالات استخدام كلاس `FileUploadService`، الذي يمثل **طبقة الخدمة** المسؤولة عن تنفيذ عمليات رفع الملفات وحذفها واسترجاعها في تطبيقات نانوسوفت. يعتمد هذا الكلاس على `FileUploadRegistry` للوصول إلى إعدادات النماذج والتحقق من الصلاحيات، وعلى `FileUploadUserManager` لإدارة المستخدم الحالي والتحقق من صلاحياته.

من خلال هذه الأمثلة، ستتعلم كيفية:

- رفع ملف واحد أو عدة ملفات باستخدام بيانات `UploadedFile` أو base64.
- استخدام مفاتيح الجلسة المؤقتة (`temp_session_key`) لربط الملفات بالنماذج غير المحفوظة.
- ربط الملفات المؤقتة بنموذج بعد حفظه.
- جلب الملفات المرتبطة بنموذج مع صور مصغرة.
- حذف ملفات مع التحقق من الصلاحيات.
- التحقق من صحة الملف (حجم، نوع) قبل الرفع.
- التعامل مع الأخطاء المختلفة (صلاحيات، قيود، أخطاء الخادم).
- دمج الخدمة مع واجهات API في تطبيقات نانوسوفت.

> **ملاحظة:** الأمثلة هنا تستخدم مباشرة `FileUploadService`، ولكن في التطبيقات الحقيقية غالبًا ما يتم استدعاؤها عبر `FileUploadController` أو من داخل كود الإضافة.

---

## المتطلبات الأساسية

- إضافة `Nano.FileUpload` مثبتة ومهيأة في تطبيقك.
- الإضافة تعتمد على `Nano.Api` التي توفر `Base64` و `ApiController`.
- تم تسجيل النماذج وحقول الرفع في `FileUploadRegistry` كما هو موضح في [توثيق الأمثلة المتقدمة لكلاس FileUploadRegistry](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md).
- معرفة أساسية بكائنات `UploadedFile` وكيفية الحصول عليها من الطلبات (مثلاً عبر `$request->file('file')`).

### إعداد الكلاس في التطبيق

```php
use Nano\FileUpload\Classes\FileUploadService;

$service = FileUploadService::instance();
```

نظرًا لأن الكلاس يتبع نمط `Singleton`، يتم استدعاؤه عبر `instance()`.

---

## 1. رفع ملف واحد باستخدام بيانات UploadedFile (ربط مباشر بنموذج موجود)

**السيناريو:** منتج موجود بالفعل، نريد رفع صورة رئيسية له وربطها مباشرة.

```php
$product = Product::find(10);
$uploadedFile = $request->file('image'); // كائن UploadedFile

$result = $service->upload(Product::class, 'image', $uploadedFile, [
    'model' => $product, // الربط مباشر
    'title' => 'صورة المنتج',
    'description' => 'وصف الصورة',
]);

if ($result['status']) {
    echo "تم رفع الصورة بنجاح. معرف الملف: " . $result['data']['id'];
    echo "المسار: " . $result['data']['path'];
} else {
    echo "فشل الرفع: " . $result['error'];
}
```

**المخرجات المتوقعة (في حالة النجاح):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'تم الرفع بنجاح',
    'data' => [
        'id' => 123,
        'path' => '/storage/app/uploads/public/...',
        'thumb' => '/storage/app/uploads/public/...',
    ],
    'process_data' => [
        'model_class' => 'Nano\Shop\Models\Product',
        'field' => 'image',
        'temp_session_key' => null,
        'user_type' => 'backend',
        // ... بيانات إضافية من Base64::onUpload
    ],
]
```

**ملاحظات:**  
- عندما يتم تمرير `model` موجود ومحفوظ (`$product->exists`)، يتم الربط مباشرة دون استخدام مفتاح جلسة مؤقت.  
- يمكن تخصيص `title` و `description` لإضافتها للملف.

---

## 2. رفع ملف واحد باستخدام base64 (نموذج غير محفوظ)

**السيناريو:** إنشاء منتج جديد، نريد رفع صورته قبل حفظ المنتج، ثم ربطها لاحقًا.

```php
$base64String = 'data:image/jpeg;base64,/9j/4AAQSkZJRg...';

$modelClass = Product::class;
$field = 'image';

// توليد مفتاح جلسة مؤقت
$tempKey = $service->generateTempSessionKey($modelClass, $field);

// رفع الصورة
$result = $service->upload($modelClass, $field, $base64String, [
    'temp_session_key' => $tempKey,
]);

if ($result['status']) {
    // حفظ المنتج
    $product = new Product();
    $product->name = 'منتج جديد';
    $product->save();

    // ربط الصورة بالمنتج
    $service->attachTempFiles($product, $field, $tempKey);

    echo "تم رفع الصورة وربطها بالمنتج رقم {$product->id}";
} else {
    echo "فشل الرفع: " . $result['error'];
}
```

**المخرجات:**  
- `upload` يعيد `temp_session_key` في `$result['temp_session_key']` إذا تم استخدامه.  
- `attachTempFiles` يعيد `true` إذا تم ربط ملفات، وإلا `false`.

**ملاحظات:**  
- `generateTempSessionKey` لا يُستخدم إجباريًا؛ إذا لم تمرر `temp_session_key`، سيقوم `upload` بتوليد واحد تلقائيًا ويعيده في النتيجة.  
- يجب حفظ `temp_session_key` في جلسة المستخدم أو قاعدة بيانات مؤقتة لاستخدامه لاحقًا.

---

## 3. رفع عدة ملفات لحقل متعدد (gallery)

**السيناريو:** رفع مجموعة صور لمعرض منتج موجود.

```php
$product = Product::find(10);
$files = $request->file('gallery'); // مصفوفة من UploadedFile

$result = $service->uploadMultiple(Product::class, 'gallery', $files, [
    'model' => $product, // الربط مباشر
]);

if ($result['status']) {
    echo "تم رفع " . $result['process_data']['success_count'] . " من " . $result['process_data']['total'] . " صور بنجاح.";
    foreach ($result['data'] as $index => $fileResult) {
        if ($fileResult['status']) {
            echo "الصورة $index: معرف {$fileResult['data']['id']}\n";
        } else {
            echo "الصورة $index فشلت: {$fileResult['error']}\n";
        }
    }
} else {
    echo "فشل رفع جميع الصور: " . $result['error'];
}
```

**المخرجات المتوقعة (جزء من `$result`):**

```php
[
    'code' => 200,
    'status' => true,
    'message' => 'تم رفع 2 من 3 ملف بنجاح',
    'data' => [
        0 => ['status' => true, 'data' => ['id' => 101, ...]],
        1 => ['status' => true, 'data' => ['id' => 102, ...]],
        2 => ['status' => false, 'error' => 'نوع الملف غير مسموح'],
    ],
    'process_data' => [
        'success_count' => 2,
        'total' => 3,
    ],
]
```

---

## 4. جلب ملفات حقل معين مع صور مصغرة

**السيناريو:** عرض جميع صور معرض منتج مع صور مصغرة بأحجام مختلفة.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
        'large' => [800, 600, 'auto'],
    ],
]);

if ($result['status']) {
    foreach ($result['data'] as $image) {
        echo "<img src='{$image['small']}' alt='{$image['title']}'>";
        echo "<a href='{$image['large']}'>عرض بحجم كبير</a>";
    }
} else {
    echo "خطأ: " . $result['error'];
}
```

**المخرجات (مثال لملف واحد):**

```php
[
    'id' => 101,
    'title' => 'صورة المنتج',
    'description' => null,
    'path' => '/storage/app/uploads/public/.../original.jpg',
    'size' => 123456,
    'content_type' => 'image/jpeg',
    'small' => '/storage/app/uploads/public/.../small.jpg',
    'medium' => '/storage/app/uploads/public/.../medium.jpg',
    'large' => '/storage/app/uploads/public/.../large.jpg',
]
```

---

## 5. حذف ملف مع التحقق من الصلاحية

**السيناريو:** مستخدم يريد حذف صورة رئيسية لمنتج، نتحقق من صلاحيته أولاً.

```php
$fileId = 123;
$result = $service->deleteFile($fileId, Product::class, 'image');

if ($result['status']) {
    echo "تم حذف الملف بنجاح";
} else {
    echo "فشل الحذف: " . $result['message'];
}
```

**ملاحظات:**  
- إذا لم يتم تمرير `$modelClass` و `$field`، لن يتم التحقق من الصلاحية.  
- التحقق يستخدم صلاحية `delete` من إعدادات الحقل أو النموذج.

---

## 6. التحقق من صحة الملف قبل الرفع

**السيناريو:** قبل رفع الملف، نتحقق من حجمه ونوعه وفقًا لقيود الحقل المسجل.

```php
$uploadedFile = $request->file('image');

try {
    $service->validateFile(Product::class, 'image', $uploadedFile);
    // الملف صالح
    $result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);
} catch (ApplicationException $e) {
    echo "الملف غير صالح: " . $e->getMessage();
}
```

**الاستثناءات الشائعة:**  
- `حجم الملف يتجاوز الحد المسموح به (2048 KB)`  
- `نوع الملف غير مسموح به. الأنواع المسموحة: jpg,jpeg,png`

---

## 7. التعامل مع أخطاء الصلاحية

**السيناريو:** محاولة رفع ملف دون صلاحية.

```php
$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);

if (!$result['status'] && $result['code'] == 403) {
    echo "لا تملك صلاحية رفع الملفات لهذا الحقل";
}
```

**كود HTTP المستخدمة:**  
- `400`: بيانات غير صالحة أو قيود الحقل.  
- `403`: صلاحية ممنوعة.  
- `404`: النموذج أو الحقل غير مسجل.  
- `500`: خطأ داخلي (يظهر تفاصيل في debug).

---

## 8. رفع ملف باستخدام بيانات base64 من واجهة API

**السيناريو:** واجهة API تستقبل صورة بصيغة base64 وتقوم برفعها وحفظها.

```php
public function uploadBase64(Request $request)
{
    $base64 = $request->input('image_base64');
    $productId = $request->input('product_id');

    $service = FileUploadService::instance();

    $product = Product::find($productId);
    if (!$product) {
        return response()->json(['error' => 'المنتج غير موجود'], 404);
    }

    $result = $service->upload(Product::class, 'image', $base64, ['model' => $product]);

    return response()->json($result);
}
```

---

## 9. رفع ملف مؤقت واستخدامه في عدة نماذج

**السيناريو:** ملف يتم رفعه مبدئيًا (مثل صورة شخصية) ثم ربطه بأكثر من نموذج (مثلاً مستخدم ومقال).

```php
// الخطوة 1: رفع الصورة بمفتاح مؤقت
$tempKey = $service->generateTempSessionKey(User::class, 'avatar');
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['temp_session_key' => $tempKey]);

if (!$result['status']) die('فشل الرفع');

// الخطوة 2: حفظ المستخدم والمقال وربط نفس الصورة
$user = new User();
$user->name = 'أحمد';
$user->save();

$article = new Article();
$article->title = 'مقال جديد';
$article->save();

// ربط الصورة بالمستخدم
$service->attachTempFiles($user, 'avatar', $tempKey);

// ربط الصورة نفسها بالمقال (إذا كان المقال له حقل image)
$service->attachTempFiles($article, 'image', $tempKey);

// ملاحظة: attachTempFiles يبحث عن الملفات التي تحمل المفتاح، وسيربط جميع الملفات الموجودة
// إذا أردت ربط نفس الملف بكلا النموذجين، سيتم تكرار الرابط (يجب التأكد من أن الملفات مستقلة)
```

**ملاحظة:** `attachTempFiles` يبحث عن جميع الملفات التي تحمل `session_key` المطلوب، لذلك إذا كان هناك عدة ملفات بنفس المفتاح، ستربط جميعها. يمكن استخدام مفاتيح منفصلة لكل عملية إذا أردت تمييزها.

---

## 10. استخدام `validateFile` مع بيانات base64 في واجهة API

**السيناريو:** قبل رفع الصورة عبر base64، نتحقق من صحتها.

```php
$base64 = $request->input('image_base64');

try {
    $service->validateFile(Product::class, 'image', $base64);
    // الملف صالح
    $result = $service->upload(Product::class, 'image', $base64, ['model' => $product]);
} catch (ApplicationException $e) {
    return response()->json(['error' => $e->getMessage()], 400);
}
```

---

## 11. الحصول على ملفات مؤقتة (غير مرتبطة بنموذج)

**السيناريو:** بعد رفع الملفات باستخدام مفتاح جلسة مؤقت، نريد عرضها قبل حفظ النموذج.

```php
$tempKey = $request->input('temp_session_key');
$result = $service->getFiles(Product::class, 'gallery', null, ['temp_session_key' => $tempKey]);

if ($result['status']) {
    echo "عدد الملفات المؤقتة: " . count($result['data']);
    foreach ($result['data'] as $file) {
        echo "<img src='{$file['path']}'>";
    }
}
```

---

## 12. التعامل مع أخطاء الحذف (ملف غير موجود)

```php
$result = $service->deleteFile(999999);

if (!$result['status']) {
    switch ($result['code']) {
        case 404:
            echo "الملف غير موجود";
            break;
        default:
            echo "خطأ: " . $result['message'];
    }
}
```

---

## 13. دمج `FileUploadService` مع `FileUploadUserManager` لتخصيص صلاحيات frontend

**السيناريو:** مستخدم frontend يريد رفع صورة شخصية، نستخدم `FileUploadUserManager` للتحقق من صلاحية مخصصة.

```php
$userManager = FileUploadUserManager::instance();
$user = $userManager->getUser();

if (!$user) {
    return response()->json(['error' => 'غير مصرح'], 401);
}

// افتراض أن نموذج المستخدم مسجل في Registry مع صلاحية add للـ avatar
$result = $service->upload(User::class, 'avatar', $uploadedFile, ['model' => $user]);
```

---

## 14. إضافة صور مصغرة مخصصة عند جلب الملفات

**السيناريو:** نريد أحجامًا مخصصة للصور المصغرة تختلف عن الأحجام الافتراضية.

```php
$result = $service->getFiles(Product::class, 'gallery', 10, [
    'with_thumbs' => true,
    'thumb_sizes' => [
        'icon' => [50, 50, 'crop'],
        'preview' => [200, 150, 'auto'],
        'full' => [800, 600, 'auto'],
    ],
]);

// النتيجة تحتوي على المفاتيح icon, preview, full لكل ملف.
```

---

## 15. رفع ملفات متعددة باستخدام بيانات base64 من طلب واحد

**السيناريو:** واجهة API تستقبل مصفوفة من سلاسل base64 وترفعها كلها.

```php
$base64Array = $request->input('images'); // ['data:image/...', 'data:image/...', ...]

$result = $service->uploadMultiple(Product::class, 'gallery', $base64Array, [
    'model' => $product,
]);

if ($result['status']) {
    echo "تم رفع " . $result['process_data']['success_count'] . " صور.";
}
```

---

## 16. أفضل الممارسات

1. **استخدم مفاتيح جلسة مؤقتة للنماذج الجديدة**: لتجنب فقدان الملفات إذا لم يتم حفظ النموذج.
2. **تحقق من صحة الملف قبل الرفع باستخدام `validateFile`**: لتقديم رسائل خطأ واضحة.
3. **سجل معرفات المعاملات (transaction IDs)** في قاعدة البيانات لتتبع عمليات الرفع.
4. **استخدم `uploadMultiple` للحقول المتعددة** بدلاً من استدعاء `upload` في حلقة، لأن `uploadMultiple` يجمع الأخطاء ويقدم إحصاءات.
5. **لا تعتمد على `temp_session_key` فقط**: بعد حفظ النموذج، استخدم `attachTempFiles` لنقل الملكية.
6. **تعامل مع الأخطاء حسب الكود**: 400، 403، 404، 500 لتقديم استجابات مناسبة.
7. **فعّل `app.debug` في بيئة التطوير** للحصول على تفاصيل الأخطاء الكاملة.
8. **استخدم `with_thumbs` فقط عند الحاجة** لتقليل حجم الاستجابة وزيادة الأداء.
9. **احتفظ بمفاتيح الجلسة المؤقتة في الجلسة (session) أو قاعدة بيانات مؤقتة** لتجنب فقدانها.
10. **اختبر الصلاحيات جيدًا**، تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم رفع أو حذف الملفات.

---

## 17. الأخطاء الشائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `ApplicationException: الحقل ... غير مسجل للنموذج ...` | الحقل غير مسجل في `FileUploadRegistry`. | أضف الحقل في التسجيل. |
| `لا تملك صلاحية رفع الملفات لهذا الحقل` | المستخدم لا يمتلك الصلاحية `add` للحقل. | تحقق من إعدادات `permissions` في Registry أو صلاحيات المستخدم. |
| `بيانات الملف غير صالحة` | البيانات الممررة ليست `UploadedFile` ولا base64 صالح. | تأكد من صيغة البيانات. |
| `الملف غير موجود` (عند الحذف) | معرف الملف غير موجود. | تأكد من صحة المعرف. |
| `حجم الملف يتجاوز الحد المسموح به` | حجم الملف أكبر من `max_filesize`. | قم بضغط الملف أو زيادة `max_filesize`. |
| `نوع الملف غير مسموح به` | امتداد الملف غير مدرج في `allowed_types`. | استخدم نوع مسموح أو عدل `allowed_types`. |

---

## 18. دمج مع نظام الصلاحيات المخصص

في تطبيقات نانوسوفت، يمكن تخصيص دالة التحقق من الصلاحيات لمستخدمي frontend عبر `FileUploadUserManager`:

```php
FileUploadUserManager::instance()->setPermissionChecker(function ($user, $operation, $permissions) {
    // منطق مخصص: مثلاً التحقق من جدول صلاحيات مخصص
    if ($user->hasRole('editor')) {
        return true;
    }
    return $user->hasPermission($permissions);
});
```

---

## الخلاصة

كلاس `FileUploadService` يقدم واجهة قوية وموحدة لإدارة رفع الملفات في تطبيقات نانوسوفت. من خلال الأمثلة المذكورة أعلاه، يمكن للمطورين تنفيذ أي سيناريو متعلق برفع الملفات: من الرفع البسيط مع الربط المباشر إلى الرفع المؤقت مع النماذج غير المحفوظة، والتحقق من الصلاحيات والقيود، وجلب الملفات مع صور مصغرة، وحذفها بأمان. نوصي باستخدام هذه الخدمة كطبقة وحيدة للتعامل مع الملفات في جميع أنحاء التطبيق لضمان الاتساق والأمان.

للاطلاع على تفاصيل هيكل الاستجابة عند استخدام متحكم `FileUploadController`، راجع [توثيق واجهة API](./Docs-API-Documentation-ar.md).

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)


