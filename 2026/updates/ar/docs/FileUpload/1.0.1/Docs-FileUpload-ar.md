# توثيق عام: إدارة رفع الملفات في تطبيقات نانوسوفت (Nano.FileUpload)

**نبذة تعريفية عن حزمة `Nano.FileUpload` ووحدة إدارة رفع الملفات**

هذه المجموعة من الكلاسات (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager` وغيرها) هي جزء من **حزمة `Nano.FileUpload`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **نانوسوفت** بهدف توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano\FileUpload\Classes
```

توفر هذه الوحدة أدوات متقدمة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع ملفات، والتحقق من صلاحيات المستخدمين (بما في ذلك أنواع المستخدمين المختلفة: `backend` و `frontend`)، وإدارة الملفات المرفوعة مؤقتًا قبل حفظ السجل، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة رفع ملفات قابلة للتوسع بسهولة وتلبية احتياجات التطبيقات المتقدمة دون المساس بأمان التطبيق أو أدائه.

---

## 1. مقدمة

**وحدة `Nano.FileUpload`** هي إضافة متخصصة مقدمة من **شركة نانوسوفت (NanoSoft)** لتطبيقات نانوسوفت و Laravel. تهدف هذه الوحدة إلى توفير **نظام متكامل لإدارة رفع الملفات** عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة، مع دعم أنواع المستخدمين المختلفة (backend / frontend) وصلاحيات دقيقة على مستوى كل حقل وعملية.

#### لماذا نحتاج إلى نظام مركزي لرفع الملفات؟
في التطبيقات الإدارية المعقدة، تحتاج العديد من الإضافات (مثل المنتجات، المستخدمين، الطلبات) إلى رفع ملفات (صور، مستندات، إلخ). بدلاً من إعادة كتابة منطق الرفع والتحقق من الصلاحيات في كل إضافة، يوفر هذا النظام **طبقة مركزية** يمكن لأي إضافة التسجيل فيها بسهولة، مما يضمن:
- توحيد عملية الرفع عبر التطبيق.
- إدارة الصلاحيات بأنواع المستخدمين المختلفة.
- إمكانية رفع الملفات قبل حفظ النموذج (باستخدام مفاتيح جلسة مؤقتة).
- معالجة موحدة للأخطاء.
- توفير واجهة API جاهزة للاستخدام.

---

## 2. المكونات الرئيسية للنظام

يتكون النظام من مجموعة من الكلاسات المتخصصة، يمكن تقسيمها إلى عدة طبقات:

### أ. طبقة تسجيل النماذج (Registry Layer)
- **`FileUploadRegistry`**: السجل المركزي الذي يحتوي على تعريفات النماذج وحقول الرفع. يوفر دوال لتسجيل النماذج، الاستعلام عن الإعدادات، والتحقق من الصلاحيات وأنواع المستخدمين.
- **الخصائص الرئيسية**:
  - تسجيل النموذج مع `allowed_user_types` (backend/frontend).
  - تعيين صلاحيات لكل عملية (`add`, `edit`, `delete`, `view`) على مستوى النموذج أو الحقل.
  - دعم الإعدادات الافتراضية (`defaults`) لتقليل التكرار.
  - إطلاق أحداث (`nano.api.fileupload.registerModels`, `nano.api.fileupload.modelRegistered`) لتوسيع النظام.

### ب. طبقة الخدمة (Service Layer)
- **`FileUploadService`**: المسؤول عن تنفيذ عمليات رفع الملفات وحذفها واسترجاعها. يعتمد على `FileUploadRegistry` للوصول إلى الإعدادات والتحقق من الصلاحيات، وعلى `FileUploadUserManager` لإدارة المستخدم الحالي.
- **الوظائف الرئيسية**:
  - رفع ملف واحد أو عدة ملفات (`upload`, `uploadMultiple`).
  - حذف ملف مع التحقق من الصلاحية (`deleteFile`).
  - جلب الملفات المرتبطة بنموذج أو مؤقتة (`getFiles`).
  - توليد مفاتيح جلسة مؤقتة (`generateTempSessionKey`).
  - ربط الملفات المؤقتة بنموذج محفوظ (`attachTempFiles`).
  - التحقق من صحة الملف (حجم، نوع) قبل الرفع (`validateFile`).

### ج. طبقة إدارة المستخدمين والصلاحيات (User & Permission Layer)
- **`FileUploadUserManager`**: مدير المستخدمين والصلاحيات. يوفر واجهة موحدة للحصول على المستخدم الحالي ونوعه (`backend`/`frontend`/`guest`) والتحقق من صلاحياته.
- **الميزات**:
  - محلل افتراضي للمستخدم يدعم `BackendAuth`, `Auth`, `AuthHelpers`.
  - إمكانية تخصيص محلل المستخدم (`setUserResolver`).
  - إمكانية تخصيص دالة التحقق من الصلاحيات (`setPermissionChecker`).
  - دعم دوال `hasAccess` و `hasPermission` لمستخدمي backend، ودالة `canUpload` لمستخدمي frontend.

### د. طبقة واجهة برمجة التطبيقات (API Layer)
- **`FileUploadController`**: متحكم API يقدم نقاط نهاية RESTful لرفع الملفات (فردي/متعدد)، حذف ملف، جلب الملفات. يستخدم `FileUploadService` و `FileUploadRegistry` و `FileUploadUserManager` داخليًا.
- **نقاط النهاية**:
  - `POST /upload` – رفع ملف واحد.
  - `POST /upload-multiple` – رفع عدة ملفات.
  - `DELETE /delete/{id}` – حذف ملف.
  - `GET /files` – جلب الملفات المرتبطة.
- **المصادقة**: OAuth 2.0 عبر middleware `oauth-users`.
- **الاستجابات**: موحدة وفق الهيكل `{code, status, message, data, ...}`.

---

## 3. كيف تعمل هذه الكلاسات معًا؟ (التدفق العام)

لنفترض أن إضافة `Nano.Shop` تريد رفع صورة رئيسية لمنتج جديد. إليك كيف تعمل المكونات معًا:

1. **تسجيل النموذج (مرة واحدة)**:
   - في `Plugin.php` للإضافة، يتم تعريف دالة `registerFileUploadFields` التي تعيد مصفوفة تعريفات النماذج.
   - يقوم `FileUploadRegistry` تلقائيًا بجمع هذه التعريفات عبر `PluginManager` وتسجيلها.

2. **رفع الملف (من خلال API)**:
   - يرسل العميل طلب `POST /upload` مع `model_class` و `field` والملف.
   - يتحقق `FileUploadController` من وجود النموذج والحقل في `FileUploadRegistry`.
   - يستدعي `FileUploadService::upload` مع البيانات.
   - يستخدم `FileUploadService`:
     - `FileUploadRegistry::getFieldConfig` للحصول على إعدادات الحقل.
     - `FileUploadUserManager::getUser` للحصول على المستخدم الحالي.
     - `FileUploadRegistry::can` للتحقق من صلاحية `add`.
     - `Base64::onUpload` (من `Nano.Api`) لرفع الملف الفعلي.
   - إذا تم تمرير `temp_session_key` أو لم يتم تمرير نموذج محفوظ، يتم استخدام مفتاح جلسة مؤقت.
   - يعيد الـ Controller استجابة موحدة تحتوي على `data` (معرف الملف، المسار) و `temp_session_key` إذا وُجد.

3. **ربط الملف بالنموذج (بعد حفظ النموذج)**:
   - بعد إنشاء المنتج وحفظه، يستخدم التطبيق `FileUploadService::attachTempFiles` لنقل الملفات المؤقتة إلى النموذج.

4. **جلب الملفات**:
   - يمكن استدعاء `GET /files` مع `model_class` و `field` و `model_id` لجلب الملفات المرتبطة.

---

## 4. حالات الاستخدام النموذجية

### أ. تسجيل نموذج منتج مع صورة رئيسية ومعرض صور
```php
// في Plugin.php
public function registerFileUploadFields()
{
    return [
        \Nano\Shop\Models\Product::class => [
            'allowed_user_types' => ['backend'],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
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

### ب. رفع صورة لمنتج جديد (باستخدام مفتاح جلسة مؤقت)
```php
// 1. رفع الصورة عبر API
$response = $api->post('/upload', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'image',
    'file_base64' => 'data:image/jpeg;base64,...',
]);
$tempKey = $response['temp_session_key'];

// 2. إنشاء المنتج وحفظه
$product = new Product();
$product->name = 'منتج جديد';
$product->save();

// 3. ربط الصورة بالمنتج (يتم من الخادم)
FileUploadService::instance()->attachTempFiles($product, 'image', $tempKey);
```

### ج. رفع عدة صور لمعرض منتج موجود
```php
$response = $api->post('/upload-multiple', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'files' => [$file1, $file2],
]);
```

### د. جلب صور معرض منتج مع صور مصغرة
```php
$files = $api->get('/files', [
    'model_class' => 'Nano\Shop\Models\Product',
    'field' => 'gallery',
    'model_id' => 10,
    'with_thumbs' => true,
    'thumb_sizes' => [
        'small' => [150, 150, 'crop'],
        'medium' => [300, 300, 'crop'],
    ],
]);
```

---

## 5. الفوائد والمزايا

- **الأمان**:
  - التحقق من صلاحيات المستخدم (backend/frontend) قبل كل عملية.
  - دعم أنواع المستخدمين المختلفة، مع إمكانية تخصيص التحقق.
  - استخدام `FileUploadUserManager` الموحد للحصول على المستخدم الحالي.
  - التحقق من حجم ونوع الملف قبل الرفع عبر `validateFile`.
  - استخدام `Base64::onUpload` الآمن (من `Nano.Api`).

- **المرونة**:
  - تسجيل أي نموذج مع إعدادات مخصصة لكل حقل.
  - إمكانية تحديد `allowed_user_types` لكل نموذج أو حقل.
  - دعم الصلاحيات لكل عملية (`add`, `edit`, `delete`, `view`) على مستوى النموذج أو الحقل.
  - إمكانية تخصيص محلل المستخدم ودالة التحقق من الصلاحيات.
  - دعم رفع الملفات بصيغ متعددة (multipart, base64).
  - دعم الرفع المؤقت للنماذج غير المحفوظة.

- **الأداء**:
  - استخدام مفاتيح جلسة مؤقتة لتجنب حفظ الملفات بشكل غير مرتبط.
  - إمكانية جلب الملفات مع صور مصغرة مخصصة.
  - تخزين إعدادات النماذج في الذاكرة بعد التحميل.

- **قابلية التوسع**:
  - يمكن تسجيل نماذج من أي إضافة عبر دالة `registerFileUploadFields` أو حدث `nano.api.fileupload.registerModels`.
  - إمكانية توسيع المنطق عبر الأحداث (`modelRegistered`).
  - فصل واضح بين طبقات التسجيل، الخدمة، المستخدمين، وواجهة API.

- **تجربة المطور**:
  - واجهة برمجية بسيطة وموحدة.
  - توثيق شامل باللغة العربية.
  - دعم عمليات الرفع الفردية والمتعددة.
  - إدارة الملفات المؤقتة تلقائيًا.

---

## 6. مثال عملي متكامل (كود كامل)

```php
<?php
use Nano\FileUpload\Classes\FileUploadService;
use Nano\FileUpload\Classes\FileUploadRegistry;
use Nano\FileUpload\Classes\FileUploadUserManager;
use Nano\Shop\Models\Product;

// 1. تسجيل النموذج (في Plugin.php)
public function registerFileUploadFields()
{
    return [
        Product::class => [
            'allowed_user_types' => ['backend'],
            'fields' => [
                'image' => [
                    'type' => 'image',
                    'max_filesize' => 1024,
                    'allowed_types' => 'jpg,jpeg,png',
                ],
            ],
        ],
    ];
}

// 2. في متحكم API
public function createProduct(Request $request)
{
    $base64Image = $request->input('image_base64');
    $productName = $request->input('name');

    // رفع الصورة مؤقتًا
    $service = FileUploadService::instance();
    $tempKey = $service->generateTempSessionKey(Product::class, 'image');
    $uploadResult = $service->upload(Product::class, 'image', $base64Image, [
        'temp_session_key' => $tempKey,
    ]);

    if (!$uploadResult['status']) {
        return response()->json(['error' => $uploadResult['error']], 400);
    }

    // إنشاء المنتج
    $product = new Product();
    $product->name = $productName;
    $product->save();

    // ربط الصورة بالمنتج
    $service->attachTempFiles($product, 'image', $tempKey);

    return response()->json(['product_id' => $product->id]);
}
```

---

## 7. الاعتماديات والمتطلبات

- **PHP** (>= 8.0).
- **Laravel / OctoberCMS** (أي إطار عمل متوافق مع `System\Models\File`).
- **إضافة `Nano.Api`** (لتوفير `Base64` و `ApiController`).
- **Composer** لتثبيت الحزمة.

---

## 8. الخاتمة

تمثل كلاسات `Nano.FileUpload` حلاً متكاملاً واحترافياً لإدارة رفع الملفات في تطبيقات نانوسوفت. بفضل التصميم الطبقي (Registry, Service, UserManager, API)، يمكن للمطورين بناء أنظمة رفع ملفات قوية وآمنة بسرعة وسهولة. مع إضافة `FileUploadService`، أصبح النظام أكثر تنظيماً وسهولة في الاستخدام، حيث يوفر نقطة دخول واحدة لتنفيذ عمليات الرفع والحذف والاسترجاع، مع دعم خيارات متقدمة مثل الرفع المؤقت، الصور المصغرة، والتحقق من الصلاحيات بأنواع المستخدمين المختلفة. هذا التصميم يضمن قابلية التوسع والصيانة، ويلبي احتياجات التطبيقات المتقدمة دون المساس بأمان النظام أو أدائه.

---

## 9. التوثيق الإضافي

للحصول على تفاصيل أكثر عن كل كلاس، يُرجى مراجعة المستندات التالية:

- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadService`](./Docs-FileUploadService-Class-Advenced-Examples-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
