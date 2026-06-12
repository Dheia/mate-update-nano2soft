# توثيق سمة `StepPhotos`

**Namespace:** `Nano\Orders\Traits\Steps`  
**الإضافة:** `Nano.Orders`  
**الإصدار:** 2.2.12  

---

## 📋 مقدمة

تُعدّ سمة `StepPhotos` نظاماً متكاملاً لإدارة الصور المرتبطة بالطلب، مدمجاً بالكامل مع `Nano.FileUpload`. صُممت لتوفير آلية مرنة وآمنة لرفع الصور وحذفها والتحقق من صلاحيات المستخدمين (مالك الطلب، الموصل، المدير) بناءً على **حالة الطلب الحالية** وقواعد قابلة للتخصيص لكل حقل.

تستخدم هذه السمة في السيناريو النموذجي لتأجير السلع (عقود الإيجار القصير)، حيث يلتقط المستأجر والمؤجر صوراً للسلعة **قبل التسليم** لتوثيق حالتها، و**بعد التسليم** لإثبات الإرجاع السليم. يتم منع تغيير حالة الطلب إلى `DELIVERY` أو `COMPLETE` إذا كانت الصور المطلوبة غير موجودة، مما يضمن تكاملية العملية.

---

## 📌 الخصائص الرئيسية (Properties)

| الخاصية | النوع | الوصف |
|----------|-------|-------|
| `$cachedPhotoRules` | `array\|null` | تخزين مؤقت لقواعد الصور بعد دمجها من الإعدادات (static، لمشاركتها بين جميع كائنات `OrderManager`). |

---

## 📌 الدوال الرئيسية (Methods)

### 1. الحصول على قواعد الصور

#### `protected function getDefaultPhotoUploadRules(): array`
- **الهدف**: إرجاع القواعد الافتراضية لرفع الصور (مدعومة بالتخزين المؤقت ودعم الإعدادات من `config`).
- **المخرجات**: مصفوفة تحتوي على قواعد لكل حقل من الحقول الأربعة.

**بنية القواعد لكل حقل:**

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `allowed_statuses` | `array` | قائمة رموز الحالات التي يُسمح فيها برفع الصور (مثال: `['NEW', 'PROCESSING']`). |
| `allowed_roles` | `array` | الأدوار المسموحة (`user`, `delivery`, `admin`). |
| `required_on_transition` | `array` | الحالات التي يجب أن تكون الصور موجودة قبل الانتقال إليها. |
| `max_files` | `int` | الحد الأقصى لعدد الصور (للحقول المتعددة). |
| `allowed_types` | `string` | الامتدادات المسموحة (مفصولة بفواصل). |
| `max_size` | `int` | الحد الأقصى لحجم الملف بالكيلوبايت (KB). |
| `description` | `string` | وصف الحقل (للوثائق أو الواجهات). |

**مثال للقيمة المرجعة:**
```php
[
    'user_before_delivery' => [
        'allowed_statuses' => ['NEW', 'PROCESSING'],
        'allowed_roles'    => ['user', 'admin'],
        'required_on_transition' => ['DELIVERY'],
        'max_files'        => 10,
        'allowed_types'    => 'jpg,jpeg,png,gif',
        'max_size'         => 5120,
        'description'      => 'صور المستأجر قبل التسليم',
    ],
    // ... باقي الحقول
]
```

**ملاحظات:**
- يتم دمج القواعد الافتراضية مع `Config::get('nano.orders::photo_rules', [])` إن وُجدت.
- يتم تخزين النتيجة في `self::$cachedPhotoRules` لتجنب إعادة الدمج في نفس الطلب.

---

### 2. رفع الصور

#### `public function uploadOrderPhoto(string $field, mixed $fileData = null, array $options = []): array`
- **الهدف**: رفع صورة واحدة لحقل معين في الطلب.
- **المعاملات**:
  - `$field`: اسم الحقل (`user_before_delivery`, `user_after_delivery`, `delivery_before_delivery`, `delivery_after_delivery`).
  - `$fileData`: كائن `UploadedFile` (من `Input::file()`) أو نص `base64`.
  - `$options`: مصفوفة خيارات إضافية (انظر الجدول أدناه).

**خيارات `$options`:**

| المفتاح | النوع | الوصف |
|---------|-------|-------|
| `order` | `Order\|null` | نموذج الطلب (يُستخدم تلقائياً إذا لم يُمرّر). |
| `user` | `mixed` | كائن المستخدم القائم بالرفع (يُجلب تلقائياً إذا لم يُمرّر). |
| `skip_permission` | `bool` | تجاوز التحقق من الصلاحية (افتراضي `false`). |
| `title` | `string\|null` | عنوان الصورة (يُحفظ في `title`). |
| `description` | `string\|null` | وصف الصورة. |
| `is_public` | `bool` | هل الصورة عامة (افتراضي `false`). |
| `auto_resize` | `bool` | تفعيل التحجيم التلقائي. |
| `resize_options` | `array` | إعدادات التحجيم (العرض، الارتفاع، الوضع). |
| `auto_watermark` | `bool` | تفعيل العلامة المائية. |
| `watermark_options` | `array` | إعدادات العلامة المائية. |
| `custom_message` | `string\|null` | رسالة نجاح مخصصة. |
| `custom_error` | `string\|null` | رسالة خطأ مخصصة. |
| `allowed_statuses` | `array\|null` | تجاوز قائمة الحالات المسموحة. |
| `allowed_roles` | `array\|null` | تجاوز قائمة الأدوار المسموحة. |
| `required_on_transition` | `array\|null` | تجاوز قائمة الحالات المطلوبة. |

**خطوات التنفيذ:**

1. **التحقق من صحة الحقل** – يتحقق من أن `$field` واحد من الحقول المسموحة.
2. **جلب بيانات الملف** – إذا لم يُمرّر `$fileData`، يحاول جلبها من `Input::file($field)`.
3. **التحقق من وجود الملف** – يرفض إذا كان `null`.
4. **تحديد الفاعل** – يستخدم `resolveActor()` إذا لم يُمرّر `user`.
5. **التحقق من الصلاحية** – يستدعي `validatePhotoUploadPermission` (ما لم يكن `skip_permission = true`).
6. **تحضير خيارات الرفع** – يجهز `$uploadOptions` لـ `FileUploadService::upload`.
7. **رفع الملف** – يستدعي خدمة رفع الملفات.
8. **معالجة النتيجة** – إذا نجح، يستخرج `$newFile` من النتيجة ويطلق حدث `nano.orders.photo.uploaded`.
9. **إرجاع الاستجابة** – مصفوفة موحدة تحتوي على `code`, `status`, `message`, `data`, `process_data`.

**هيكل الاستجابة (نجاح):**
```php
[
    'code' => 200,
    'status' => true,
    'message' => 'تم رفع الصورة بنجاح',
    'data' => [
        'id' => 123,
        'path' => 'https://.../image.jpg',
        'thumb' => 'https://.../thumb.jpg'
    ],
    'process_data' => [ ... ]
]
```

---

#### `public function uploadOrderPhotos(string $field, array $filesData = [], array $options = []): array`
- **الهدف**: رفع عدة صور دفعة واحدة للحقل المتعدد.
- **المعاملات**:
  - `$field`: اسم الحقل.
  - `$filesData`: مصفوفة من كائنات `UploadedFile` أو سلاسل `base64`.
  - `$options`: نفس خيارات `uploadOrderPhoto`.
- **السلوك**:
  - إذا لم تُمرّر `$filesData`، يحاول جلبها من `Input::file($field)`.
  - يتكرر على كل ملف ويستدعي `uploadOrderPhoto` مع `skip_permission = true` (لأن الصلاحية فُحصت مرة واحدة فقط).
  - يجمع النتائج ويحسب عدد النجاحات.
  - إذا نجحت جميع الملفات، `code = 200`؛ إذا نجح بعضها فقط، `code = 207 (Partial Content)`.
- **الإرجاع**: مصفوفة تحتوي على `data` (نتائج كل ملف)، `errors` (أخطاء كل ملف فاشل)، `process_data.success_count`.

---

### 3. حذف الصور

#### `public function deleteOrderPhoto(string $field, int $fileId, array $options = []): array`
- **الهدف**: حذف صورة موجودة من حقل معين.
- **المعاملات**:
  - `$field`: اسم الحقل.
  - `$fileId`: معرف الصورة المراد حذفها.
  - `$options`: يحتوي على `order`, `user`, `skip_permission`.
- **السلوك**:
  - يتحقق من وجود الطلب والملف.
  - إذا لم يُتجاوز الفحص (`skip_permission = false`)، يستدعي `validatePhotoUploadPermission` (للتأكد من أن المستخدم له صلاحية الحذف).
  - يحذف الملف عبر `$file->delete()`.
  - يطلق حدث `nano.orders.photo.deleted`.
- **الإرجاع**: مصفوفة تحتوي على `code`, `status`, `message`, `data.deleted_file_id`.

---

### 4. التحقق من الصلاحية

#### `public function validatePhotoUploadPermission(string $field, mixed $user, Order $order, ?string $newStatus = null, array $options = []): void`
- **الهدف**: التحقق من أن المستخدم مخوّل برفع صورة في الحقل المعطى.
- **الاستثناءات**: ترمي `ApplicationException` في حال الفشل مع رسالة مناسبة.

**خطوات التحقق:**

1. **جلب القواعد** – من `$options['rules']` أو القواعد الافتراضية.
2. **استخراج قواعد الحقل** – إذا لم توجد قواعد، يُسمح بالرفع (تخطي).
3. **تحديد دور المستخدم** – عبر `getUserRoleForOrder` (يعيد `user`, `delivery`, `admin`, أو `null`).
4. **التحقق من الدور المسموح** – إذا لم يكن الدور في `allowed_roles` وليس `admin`، يرفض.
5. **التحقق من الحالة المسموحة** – إذا كانت القائمة غير فارغة والحالة الحالية للطلب غير موجودة فيها، يرفض.
6. **التحقق من الحد الأقصى** – إذا كان `max_files` محدّداً وعدد الصور الحالي يساويه أو يتجاوزه، يرفض.
7. **التحقق من الحالة الجديدة** – إذا تم تمرير `$newStatus` وكان موجوداً في `required_on_transition` وعدد الصور `0`، يرفض.

**أمثلة على رسائل الخطأ:**
- `لا تملك صلاحية رفع الصور لهذا الطلب.`
- `المستخدم من نوع user غير مسموح له برفع الصور في حقل delivery_before_delivery.`
- `لا يمكن رفع الصور في حقل user_before_delivery في الحالة الحالية (COMPLETE).`
- `تم الوصول إلى الحد الأقصى لعدد الصور (10) في حقل user_before_delivery.`
- `يجب رفع الصور في حقل user_before_delivery قبل تغيير الحالة إلى DELIVERY.`

---

### 5. دوال الفحص السريع

#### `public function canUploadPhoto(string $field, mixed $user, Order $order): bool`
- **الهدف**: التحقق من إمكانية رفع الصورة (بدون استثناء).
- **الإرجاع**: `true` إذا كان مسموحاً، `false` خلاف ذلك.

#### `public function canUploadPhotoToField(string $field, mixed $user, Order $order, ?string $newStatus = null): bool`
- **الهدف**: نفس ما سبق، مع دعم تمرير الحالة الجديدة (للتحقق من متطلبات الانتقال).
- **الإرجاع**: `bool`.

---

### 6. دوال متطلبات تغيير الحالة

#### `public function enforcePhotoRequirementsOnStatusChange(string $oldStatus, string $newStatus, Order $order, mixed $actor = null, array $options = []): void`
- **الهدف**: تُستدعى قبل تغيير حالة الطلب للتأكد من أن الصور المطلوبة موجودة.
- **الاستثناءات**: ترمي `ApplicationException` إذا كان أحد الحقول المطلوبة للانتقال إلى `$newStatus` لا يحتوي على صور.
- **السلوك**:
  - يستخدم القواعد من `$options['rules']` أو القواعد الافتراضية.
  - لكل حقل، إذا كانت `$newStatus` موجودة في `required_on_transition` وعدد الصور `0`، يرمي استثناء.
  - إذا تم تمرير `$actor`، يتحقق أيضاً من أن دوره مسموح برفع هذا الحقل (اختياري، يُستخدم للأمان الإضافي).

#### `public function hasRequiredPhotosForStatus(string $newStatus, Order $order, ?string $role = null): bool`
- **الهدف**: فحص سريع (بدون استثناء) ما إذا كانت جميع الصور المطلوبة للحالة الجديدة موجودة.
- **الإرجاع**: `true` إذا كانت موجودة، `false` خلاف ذلك.
- **الاستخدام**: لواجهات المستخدم (إظهار/إخفاء زر تغيير الحالة).

---

### 7. جلب روابط الصور

#### `public function getOrderPhotoUrls(string $field, array $thumbSizes = [], bool $includeModel = false): ?array`
- **الهدف**: إرجاع روابط الصور لحقل معين مع صور مصغرة بأحجام مخصصة.
- **المعاملات**:
  - `$field`: اسم الحقل.
  - `$thumbSizes`: مصفوفة بأسماء الأحجام وأبعادها (مثال: `['thumb' => [150,150,'crop']]`).
  - `$includeModel`: هل تضمين كائن الملف الكامل في النتيجة.
- **الإرجاع**:
  - إذا كان الحقل متعدد (`attachMany`): مصفوفة من الصور.
  - إذا كان فردياً (`attachOne`): صورة واحدة.
  - إذا لم توجد صور: `null`.

**مثال النتيجة:**
```php
[
    'id' => 456,
    'title' => 'صورة قبل التسليم',
    'path' => 'https://.../original.jpg',
    'thumb' => 'https://.../thumb.jpg',
    'medium' => 'https://.../medium.jpg',
    'size' => 12345,
    'content_type' => 'image/jpeg',
    'created_at' => '2026-06-05 10:00:00',
]
```

#### `public function formatPhotoFile($file, array $thumbSizes = [], bool $includeModel = false): array`
- **الهدف**: تنسيق ملف واحد إلى مصفوفة تحتوي على البيانات الأساسية والصور المصغرة.
- **الاستخدام الداخلي**: تُستدعى بواسطة `getOrderPhotoUrls`.

---

### 8. دوال مساعدة داخلية

#### `protected function getUserRoleForOrder(mixed $user, Order $order): ?string`
- **الهدف**: تحديد دور المستخدم بالنسبة للطلب (مدير، مالك، موصل، أو `null`).
- **المنطق**:
  - إذا كان `$user` من نوع `Backend\User` → `'admin'`.
  - إذا كان من نوع `RainLab\User\Models\User`:
    - إذا `user->id == $order->user_id` → `'user'`.
    - إذا `user->id == $order->delivery_user_id` → `'delivery'`.
  - خلاف ذلك → `null`.

#### `protected function resolveActor(): mixed`
- **الهدف**: جلب المستخدم الحالي (بنفس منطق `updateOrderStatusAdvanced`).
- **الترتيب**:
  1. `\Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()`
  2. `\BackendAuth::getUser()`
  3. `\Auth::getUser()`

#### `protected function isPhotoDuplicate(Order $order, string $field, $uploadedFile): bool`
- **الهدف**: فحص ما إذا كان هناك ملف بنفس المحتوى (`hash`) موجود مسبقاً في الحقل.
- **ملاحظة**: لا تُستخدم حالياً بشكل افتراضي، ولكن يمكن تفعيلها عبر `$options['check_duplicate'] = true`.

---

### 9. الأحداث (Events)

| اسم الحدث | وقت الإطلاق | المعاملات |
|-----------|-------------|------------|
| `nano.orders.photo.uploaded` | بعد رفع الصورة بنجاح | `['order' => $order, 'field' => $field, 'file' => $newFile, 'actor' => $actor]` |
| `nano.orders.photo.deleted` | بعد حذف الصورة بنجاح | `['order' => $order, 'field' => $field, 'file_id' => $fileId, 'actor' => $actor]` |

---

## 🧪 أمثلة عملية

### مثال 1: رفع صورة واحدة (مالك الطلب)

```php
$orderManager = OrderManager::instance();
$order = Order::find(123);
$orderManager->setOrderModel($order);

$uploadedFile = Input::file('user_photo');
$result = $orderManager->uploadOrderPhoto('user_before_delivery', $uploadedFile, [
    'title' => 'صورة المستأجر قبل التسليم',
    'description' => 'تم الرفع بواسطة المالك',
]);

if ($result['status']) {
    echo "تم الرفع، معرف الصورة: " . $result['data']['id'];
} else {
    echo "فشل الرفع: " . $result['error'];
}
```

### مثال 2: رفع عدة صور (موصل الطلب)

```php
$files = Input::file('delivery_photos'); // مصفوفة
$result = $orderManager->uploadOrderPhotos('delivery_before_delivery', $files, [
    'user' => $deliveryUser,
]);

if ($result['status']) {
    echo "تم رفع " . $result['process_data']['success_count'] . " من " . $result['process_data']['total'] . " صور.";
}
```

### مثال 3: حذف صورة (مدير)

```php
$result = $orderManager->deleteOrderPhoto('user_after_delivery', 456, [
    'skip_permission' => true, // يتجاوز فحص الصلاحية
]);

if ($result['status']) {
    echo "تم حذف الصورة " . $result['data']['deleted_file_id'];
}
```

### مثال 4: التحقق من الصلاحية قبل عرض واجهة الرفع

```php
$currentUser = Auth::getUser();
$canUpload = $orderManager->canUploadPhoto('user_before_delivery', $currentUser, $order);

if ($canUpload) {
    // عرض نموذج رفع الصورة
} else {
    // إخفاء النموذج أو تعطيله
}
```

### مثال 5: جلب روابط الصور مع صور مصغرة للـ API

```php
$photos = $orderManager->getOrderPhotoUrls('delivery_before_delivery', [
    'thumb' => [100, 100, 'crop'],
    'medium' => [300, 300, 'crop'],
]);

foreach ($photos as $photo) {
    echo "<img src='{$photo['thumb']}' alt='{$photo['title']}'>";
}
```

### مثال 6: منع تغيير الحالة بدون الصور المطلوبة (مدمج في `updateOrderStatusAdvanced`)

```php
// محاولة تغيير الحالة إلى DELIVERY بدون رفع user_before_delivery
try {
    $result = $orderManager->updateOrderStatusAdvanced([
        'order' => $order,
        'order_states_ref_type' => 'DELIVERY',
    ]);
} catch (ApplicationException $e) {
    // رسالة: "يجب رفع الصور في حقل user_before_delivery قبل تغيير الحالة إلى DELIVERY"
}
```

### مثال 7: تجاوز فحص الصور (للمسؤولين)

```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order' => $order,
    'order_states_ref_type' => 'COMPLETE',
    'skip_photo_check' => true,
    'admin_override' => true,
]);
```

---

## 🛠️ التخصيص عبر الإعدادات (Config)

يمكن تخصيص القواعد لكل حقل عن طريق إضافة مصفوفة `photo_rules` في ملف `config/nano/orders.php`:

```php
'photo_rules' => [
    'user_before_delivery' => [
        'allowed_statuses' => ['NEW', 'PROCESSING'],
        'allowed_roles'    => ['user', 'admin'],
        'required_on_transition' => ['DELIVERY'],
        'max_files'        => 15,
        'allowed_types'    => 'jpg,jpeg,png,webp',
        'max_size'         => 10240, // 10 MB
        'description'      => 'صور المستأجر قبل التسليم (مخصص)',
    ],
    // ... باقي الحقول
],
```

عبر متغيرات البيئة في `.env`:

```ini
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_FILES=15
NANO_ORDERS_PHOTO_USER_BEFORE_ALLOWED_TYPES="jpg,jpeg,png,webp"
NANO_ORDERS_PHOTO_USER_BEFORE_MAX_SIZE=10240
```

---

## 🔁 التكامل مع `updateOrderStatusAdvanced`

دالة `updateOrderStatusAdvanced` (في ترايت `StepStatus`) تدعم الآن خيارين جديدين:

| الخيار | النوع | الوصف |
|--------|-------|-------|
| `skip_photo_check` | `bool` | تجاوز فحص الصور المطلوبة (افتراضي `false`). |
| `photo_validation_rules` | `array` | قواعد مخصصة لتجاوز القواعد الافتراضية. |

**آلية العمل:**
1. تحدد الدالة الحالة الجديدة الفعلية (أولوية لـ `order_states_ref_type`، ثم `user_status`/`delivery_status`).
2. تستدعي `enforcePhotoRequirementsOnStatusChange` مع تمرير الحالة الجديدة والمستخدم الفاعل.
3. إذا كانت الصور المطلوبة مفقودة، ترمى استثناء ويمنع التغيير.

---

## ❗ معالجة الأخطاء (الأخطاء الشائعة)

| الخطأ | السبب | الحل |
|-------|-------|------|
| `Invalid field name.` | تم تمرير حقل غير مسموح (ليس من الحقول الأربعة). | استخدم أحد الحقول: `user_before_delivery`, `user_after_delivery`, `delivery_before_delivery`, `delivery_after_delivery`. |
| `الملف مطلوب.` | لم يتم تمرير بيانات الملف أو الملف فارغ. | تأكد من وجود `file` في الطلب (كـ `UploadedFile`) أو `file_base64`. |
| `لا تملك صلاحية رفع الصور لهذا الطلب.` | المستخدم غير مرتبط بالطلب وليس مديراً. | تحقق من أن المستخدم هو المالك أو الموصل أو مدير. |
| `المستخدم من نوع user غير مسموح له برفع الصور في حقل ...` | دور المستخدم غير موجود في `allowed_roles` لهذا الحقل. | راجع إعدادات `photo_rules` للحقل المعني. |
| `لا يمكن رفع الصور في حقل ... في الحالة الحالية (COMPLETE).` | الطلب في حالة لا تسمح بالرفع لهذا الحقل. | غيّر حالة الطلب أولاً أو عدّل `allowed_statuses` في الإعدادات. |
| `تم الوصول إلى الحد الأقصى لعدد الصور (10) في حقل ...` | تم رفع العدد الأقصى من الصور بالفعل. | احذف بعض الصور أولاً أو زِد `max_files`. |
| `يجب رفع الصور في حقل ... قبل تغيير الحالة إلى DELIVERY.` | محاولة تغيير الحالة إلى `DELIVERY` أو `COMPLETE` بدون الصور المطلوبة. | ارفع الصور أولاً، أو استخدم `skip_photo_check = true`. |

---

## ✅ أفضل الممارسات

1. **استخدم `getOrderPhotoUrls` في الـ API** لعرض الصور مع صور مصغرة، بدلاً من إعادة المسار الأصلي فقط.
2. **لا تثق بإدخال المستخدم** – الدالة تتحقق من صلاحية الحقل (`allowedFields`) وقواعد الصلاحية تلقائياً.
3. **للمسؤولين فقط استخدم `skip_photo_check = true`** – لتجنب تجاوز القواعد عن طريق الخطأ.
4. **اضبط `max_files` و `max_size` في الإعدادات** لتتناسب مع خطة التخزين وخادمك.
5. **استخدم الأحداث (`nano.orders.photo.uploaded`, `nano.orders.photo.deleted`)** لإرسال إشعارات WebSocket أو تسجيل النشاطات.
6. **اختبر جميع الأدوار** (مالك، موصل، مدير) للتأكد من أن صلاحيات الرفع مضبوطة بشكل صحيح.

---

## 🔚 الخاتمة

سمة `StepPhotos` هي إضافة قوية ومرنة لنظام `Nano.Orders`، تتيح إدارة الصور المرتبطة بالطلب مع تحكم كامل في الصلاحيات بناءً على حالة الطلب ودور المستخدم. بفضل التكامل مع `Nano.FileUpload`، أصبح الرفع آمناً وموحداً، مع دعم الملفات العادية و `base64`. دوال الفحص السريع (`canUploadPhoto`, `hasRequiredPhotosForStatus`) تمكن المطورين من بناء واجهات مستخدم ذكية، في حين أن خيار `skip_photo_check` في `updateOrderStatusAdvanced` يوفر مرونة إضافية للمسؤولين عند الحاجة. التوثيق المفصل والأمثلة العملية تجعل من السهل دمج هذه السمة في أي تطبيق.

---

**الوثائق المرجعية:**
- [توثيق `OrderManager` وسماته](./docs/Orders/Classes/Docs-OrderManager-Class-ar.md)
- [توثيق `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-ar.md)
- [توثيق متقدم لـ `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-ar.md)
- [توثيق `Nano.FileUpload`](./docs/FileUpload/Docs-FileUpload-ar.md)