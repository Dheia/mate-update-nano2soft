# توثيق الأمثلة المتقدمة والعملية لكلاس `FileUploadUserManager`

**Namespace:** `Nano\FileUpload\Classes`  
**الهدف:** تقديم أمثلة تطبيقية شاملة لاستخدام `FileUploadUserManager` في سيناريوهات متنوعة، بدءًا من الاستخدام الأساسي لجلب المستخدم الحالي وصولًا إلى تخصيص محلل المستخدم ودالة التحقق من الصلاحيات، والتكامل مع `FileUploadRegistry` و `FileUploadService`، مع توضيح المدخلات والمخرجات المتوقعة وأفضل الممارسات.

---

## مقدمة

يقدم هذا الوثيقة مجموعة من الأمثلة العملية التي تغطي مختلف حالات استخدام كلاس `FileUploadUserManager`، الذي يمثل **مدير المستخدمين والصلاحيات** في نظام رفع الملفات في تطبيقات نانوسوفت. هذا الكلاس مسؤول عن:

- جلب المستخدم الحالي من مصادر متعددة (باك إند، فرونت إند، أو أي نظام مستخدمين مخصص).
- تحديد نوع المستخدم (`backend`, `frontend`, `guest`).
- التحقق من صلاحيات المستخدمين بناءً على نوعهم والصلاحيات المطلوبة.

من خلال هذه الأمثلة، ستتعلم كيفية:

- استخدام المحلل الافتراضي للمستخدم (default resolver) لجلب المستخدم الحالي.
- تخصيص محلل المستخدم (`setUserResolver`) ليتناسب مع نظام المصادقة الخاص بك.
- تعيين دالة مخصصة للتحقق من الصلاحيات (`setPermissionChecker`).
- التعامل مع أنواع المستخدمين المختلفة (backend, frontend, guest).
- دمج `FileUploadUserManager` مع `FileUploadRegistry` و `FileUploadService`.
- تتبع المشكلات الشائعة وحلولها.

> **ملاحظة:** الأمثلة هنا تستخدم مباشرة `FileUploadUserManager`، ولكن في التطبيقات الحقيقية غالبًا ما يتم استخدامه عبر `FileUploadRegistry` و `FileUploadService`.

---

## المتطلبات الأساسية

- إضافة `Nano.FileUpload` مثبتة ومهيأة في تطبيقك.
- نظام مصادقة عامل (باك إند أو فرونت إند) يمكن من خلاله الحصول على المستخدم الحالي.
- فهم أساسي لأنواع المستخدمين التي يدعمها التطبيق (backend/frontend).

### إعداد الكلاس في التطبيق

```php
use Nano\FileUpload\Classes\FileUploadUserManager;

$manager = FileUploadUserManager::instance();
```

نظرًا لأن الكلاس يتبع نمط `Singleton`، يتم استدعاؤه عبر `instance()`.

---

## 1. الاستخدام الأساسي (المحلل الافتراضي)

**السيناريو:** جلب المستخدم الحالي باستخدام المحلل الافتراضي، وعرض نوعه ومعرفه.

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();

if ($user) {
    echo "نوع المستخدم: " . $manager->getUserType(); // 'backend' أو 'frontend'
    echo "معرف المستخدم: " . $manager->getId();
    echo "الاسم: " . $user->name ?? $user->full_name ?? 'غير معروف';
} else {
    echo "المستخدم غير مسجل دخول (guest)";
}
```

**المخرجات المتوقعة (مثال):**

```
نوع المستخدم: backend
معرف المستخدم: 5
الاسم: أحمد علي
```

**ملاحظات:**  
- المحلل الافتراضي يحاول استخدام `Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()` أولاً، ثم `BackendAuth::getUser()`، ثم `Auth::getUser()`، ثم `auth()->user()`.
- إذا لم يتم العثور على مستخدم، يعيد `null` ونوع المستخدم يكون `'guest'`.

---

## 2. تخصيص محلل المستخدم (setUserResolver)

**السيناريو:** لديك نظام مستخدمين مخصص يعتمد على جدول `custom_users`، وتريد أن يستخدمه `FileUploadUserManager`.

```php
$manager->setUserResolver(function() {
    // منطق جلب المستخدم من نظامك المخصص
    if (isset($_SESSION['custom_user_id'])) {
        return CustomUser::find($_SESSION['custom_user_id']);
    }
    return null;
});

$user = $manager->getUser();
if ($user) {
    echo "تم جلب المستخدم المخصص بنجاح";
}
```

**ملاحظات:**  
- بعد تعيين محلل مخصص، يتم تجاهل المحلل الافتراضي.
- يمكنك أيضًا تعيين المستخدم مباشرة باستخدام `setUser($user)` لتجاوز المحلل بالكامل.

---

## 3. تعيين المستخدم مباشرة (setUser)

**السيناريو:** لديك المستخدم بالفعل (مثلاً من طلب API) وتريد تمريره إلى `FileUploadUserManager` دون استخدام المحلل.

```php
$user = CustomUser::find(10);
$manager->setUser($user);

echo "معرف المستخدم: " . $manager->getId(); // 10
```

**ملاحظات:**  
- `setUser` يتجاوز المحلل والمحلل المخصص، ويستخدم المستخدم المحدد مباشرة.
- يمكن استخدام `clearUser()` لإعادة تعيين المستخدم المخبأ والعودة إلى المحلل.

---

## 4. إعادة تعيين المستخدم المخبأ (clearUser)

**السيناريو:** بعد تسجيل خروج المستخدم، تريد إجبار `FileUploadUserManager` على إعادة حل المستخدم.

```php
// افترض أن المستخدم كان مسجلاً
$user = $manager->getUser(); // يعيد المستخدم القديم

// بعد تسجيل الخروج
$manager->clearUser();

$user = $manager->getUser(); // يعيد المستخدم الجديد (null أو مستخدم آخر)
```

---

## 5. التحقق من صلاحيات مستخدم backend (checkPermission)

**السيناريو:** مستخدم من نوع backend يريد رفع صورة، نتحقق مما إذا كان يملك صلاحية `nano.shop.upload_image`.

```php
$user = BackendAuth::getUser(); // مستخدم backend
$hasPermission = $manager->checkPermission('add', 'nano.shop.upload_image', $user);

if ($hasPermission) {
    echo "مسموح برفع الصورة";
} else {
    echo "غير مسموح";
}
```

**المخرجات:**  
- إذا كان المستخدم يمتلك الصلاحية (عبر `hasAccess` أو `hasPermission`)، يعيد `true`.  
- وإلا يعيد `false`.

**ملاحظات:**  
- الدالة تستدعي `checkBackendPermissions` داخليًا لمستخدمي backend.
- يمكن تمرير مصفوفة صلاحيات بدلاً من واحدة: `['nano.shop.upload', 'nano.shop.manage']`.

---

## 6. تعيين دالة مخصصة للتحقق من الصلاحيات (setPermissionChecker)

**السيناريو:** لديك نظام صلاحيات مخصص لمستخدمي frontend يعتمد على جدول `user_roles`.

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    // منطق مخصص: مثلاً تحقق من دور المستخدم
    if ($user->hasRole('editor')) {
        return true;
    }
    return false;
});

$user = Auth::getUser(); // مستخدم frontend
$allowed = $manager->checkPermission('add', null, $user); // لا نمرر صلاحيات backend
if ($allowed) {
    echo "مسموح";
}
```

**ملاحظات:**  
- دالة التحقق المخصصة تُستدعى لكل من backend و frontend إذا كانت موجودة، وتتجاوز المنطق الافتراضي.
- `$permissions` قد تكون `null` إذا لم تمرر صلاحيات (كما في حالة frontend).

---

## 7. استخدام دالة `canUpload` في نموذج المستخدم (frontend)

**السيناريو:** نموذج المستخدم يحتوي على دالة `canUpload` تُستخدم للتحقق من الصلاحيات.

```php
class User extends Model {
    public function canUpload($operation, $permissions) {
        // منطق مخصص: مثلاً التحقق من جدول صلاحيات
        return $this->hasPermission('upload') || $operation === 'view';
    }
}

// في مكان آخر
$manager = FileUploadUserManager::instance();
$user = Auth::getUser();

if ($manager->checkPermission('add', null, $user)) {
    // سيتم استدعاء $user->canUpload('add', null)
    echo "مسموح";
}
```

**ملاحظات:**  
- يتم استدعاء `canUpload` فقط إذا كان المستخدم من نوع frontend ولم يتم تعيين `permissionChecker` مخصص.
- إذا كان للمستخدم دالة `canUpload`، تعيد نتيجتها، وإلا تعيد `true` (السماح الافتراضي).

---

## 8. دمج مع `FileUploadRegistry` للتحقق من صلاحية حقل معين

**السيناريو:** قبل رفع ملف لحقل معين، نستخدم `FileUploadRegistry::can` الذي يعتمد بدوره على `FileUploadUserManager`.

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';
$user = $userManager->getUser();
$userType = $userManager->getUserType();

if ($registry->can($modelClass, 'add', $userType, $user, $field)) {
    // مسموح بالرفع
}
```

**ملاحظات:**  
- `FileUploadRegistry::can` يستدعي `FileUploadUserManager::checkPermission` مع صلاحيات الحقل أو النموذج.
- هذا هو الاستخدام الشائع في التطبيقات الحقيقية.

---

## 9. الحصول على نوع المستخدم (getUserType)

**السيناريو:** عرض واجهة مختلفة لمستخدمي backend و frontend.

```php
$userType = $manager->getUserType();

switch ($userType) {
    case 'backend':
        echo "مرحبًا مدير النظام";
        break;
    case 'frontend':
        echo "مرحبًا زائرنا العزيز";
        break;
    default:
        echo "الرجاء تسجيل الدخول";
}
```

**المخرجات:**  
- `'backend'` لمستخدمي النظام.  
- `'frontend'` لمستخدمي الواجهة الأمامية.  
- `'guest'` للمستخدم غير المسجل.

---

## 10. التعامل مع المستخدم غير المسجل (guest)

**السيناريو:** لا يسمح للمستخدمين غير المسجلين برفع الملفات.

```php
$user = $manager->getUser();
if (!$user) {
    die("الرجاء تسجيل الدخول لرفع الملفات");
}
```

---

## 11. دمج مع `FileUploadService` في عملية رفع كاملة

**السيناريو:** رفع ملف مع التحقق من الصلاحيات باستخدام `FileUploadService` الذي يستخدم `FileUploadUserManager` داخليًا.

```php
$service = FileUploadService::instance();
$result = $service->upload(Product::class, 'image', $uploadedFile, ['model' => $product]);

if (!$result['status'] && $result['code'] == 403) {
    echo "غير مسموح برفع الملف: " . $result['error'];
}
```

**ملاحظات:**  
- `FileUploadService` يستدعي `FileUploadRegistry::can` الذي بدوره يستخدم `FileUploadUserManager`.
- أي خطأ صلاحية يظهر مع كود 403.

---

## 12. اختبار دالة `checkPermission` مع أنواع مختلفة من الصلاحيات

**السيناريو:** اختبار صلاحية واحدة ومصفوفة صلاحيات.

```php
$user = BackendAuth::getUser();

// صلاحية واحدة
$allowed = $manager->checkPermission('add', 'nano.shop.upload', $user);

// مصفوفة صلاحيات (تحقق من أي واحدة)
$allowed = $manager->checkPermission('add', ['nano.shop.upload', 'nano.shop.manage'], $user);
```

**السلوك:**  
- في حالة المصفوفة، تعيد `true` إذا كان المستخدم يمتلك أيًا من الصلاحيات.

---

## 13. تخصيص دالة حل المستخدم بناءً على التوكن (JWT)

**السيناريو:** واجهة API تستخدم JWT، وتحتاج إلى استخراج المستخدم من التوكن.

```php
$manager->setUserResolver(function() {
    $token = request()->bearerToken();
    if ($token) {
        return User::where('api_token', $token)->first();
    }
    return null;
});

$user = $manager->getUser();
```

---

## 14. تخصيص دالة التحقق من الصلاحيات لتكون صارمة (مثل Laravel Gates)

**السيناريو:** استخدام Gates في Laravel للتحقق من الصلاحيات.

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    if (Gate::allows($operation, $user)) {
        return true;
    }
    return false;
});
```

---

## 15. معالجة أخطاء الصلاحيات في واجهة API

**السيناريو:** في متحكم API، نعيد استجابة مناسبة إذا كان المستخدم غير مصرح له.

```php
public function upload(Request $request)
{
    $user = FileUploadUserManager::instance()->getUser();
    if (!$user) {
        return response()->json(['error' => 'غير مصرح'], 401);
    }

    $userType = FileUploadUserManager::instance()->getUserType();
    $allowed = FileUploadRegistry::instance()->can(
        Product::class, 'add', $userType, $user, 'image'
    );

    if (!$allowed) {
        return response()->json(['error' => 'لا تملك صلاحية'], 403);
    }

    // ... رفع الملف
}
```

---

## 16. أفضل الممارسات

1. **استخدم المحلل الافتراضي ما لم يكن لديك متطلبات خاصة**: يوفر دعمًا شاملاً لأنظمة المصادقة الشائعة في نانوسوفت.
2. **لا تعتمد على السماح الافتراضي لمستخدمي frontend**: إذا كنت بحاجة إلى تقييد، قم بتعيين دالة تحقق مخصصة أو استخدم `canUpload` في نموذج المستخدم.
3. **استخدم `setPermissionChecker` لتوحيد منطق التحقق عبر التطبيق**: مفيد إذا كان لديك نظام صلاحيات مركزي (مثل Roles & Permissions).
4. **قم بإعادة تعيين المستخدم المخبأ (`clearUser()`) بعد تسجيل الخروج أو تغيير المستخدم**: لتجنب الاحتفاظ ببيانات قديمة.
5. **اختبر صلاحيات المستخدمين جيدًا**: تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم تنفيذ عمليات رفع أو حذف.
6. **استخدم `getUserType()` لتمييز تجربة المستخدم**: مثل عرض واجهة مختلفة لمستخدمي backend و frontend.
7. **لا تمرر صلاحيات backend لمستخدمي frontend**: في `checkPermission`، إذا كان المستخدم frontend، اترك `$permissions` فارغًا واستخدم دالة التحقق المخصصة.

---

## 17. الأخطاء الشائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `getUser()` يعيد `null` رغم أن المستخدم مسجل | المحلل الافتراضي لم يعثر على المستخدم. | استخدم `setUserResolver` لتحديد طريقة جلب المستخدم الخاصة بك. |
| `checkPermission` يعيد `true` دائمًا لمستخدمي frontend | لم يتم تعيين `permissionChecker`، والسماح الافتراضي هو `true`. | قم بتعيين `setPermissionChecker` لتنفيذ منطق التحقق الخاص بك. |
| صلاحيات backend لا تعمل | المستخدم من نوع backend ولكن دالة `hasAccess` أو `hasPermission` غير موجودة. | تأكد من أن كائن المستخدم يحتوي على إحدى هذه الدوال (في أنظمة نانوسوفت عادةً تكون موجودة). |
| `getUserType` يعيد `'frontend'` لنموذج backend | النموذج لا يتطابق مع أي من الأنواع المعروفة. | إذا كنت تستخدم نموذجًا مخصصًا، قم بتوسيع دالة `getUserTypeFromModel` أو استخدم `setUserResolver` لتحديد النوع. |
| المستخدم يتغير في الجلسة ولكن `getUser()` لا يعيد القيمة الجديدة | التخزين المؤقت يحتفظ بالقيمة القديمة. | استخدم `clearUser()` بعد تغيير المستخدم. |

---

## 18. دمج مع نظام الصلاحيات في نانوسوفت

في تطبيقات نانوسوفت، يتم عادةً استخدام `FileUploadUserManager` بشكل غير مباشر عبر `FileUploadService`. ومع ذلك، يمكن تخصيصه بسهولة:

- **لمستخدمي backend**: يعتمد على `hasAccess` أو `hasPermission` الموجودين في نموذج `Backend\Models\User`.
- **لمستخدمي frontend**: يمكن تخصيص دالة `canUpload` في نموذج المستخدم، أو تعيين `permissionChecker` مخصص يستخدم `Auth` أو أي نظام صلاحيات آخر.

---

## الخلاصة

كلاس `FileUploadUserManager` هو أداة مرنة وقوية لإدارة المستخدمين والصلاحيات في نظام رفع الملفات في تطبيقات نانوسوفت. من خلال الأمثلة المذكورة أعلاه، يمكن للمطورين:

- جلب المستخدم الحالي من أي مصدر بسهولة.
- التحقق من صلاحيات المستخدمين حسب نوعهم (backend/frontend).
- تخصيص منطق جلب المستخدم والتحقق من الصلاحيات ليتناسب مع أي نظام مصادقة أو صلاحيات موجود.

نوصي بالاستفادة من هذه الأمثلة لبناء أنظمة رفع ملفات آمنة وقابلة للتوسع.

للاطلاع على تفاصيل أكثر حول كيفية دمج هذا الكلاس مع `FileUploadRegistry` و `FileUploadService`، راجع [توثيق كلاس FileUploadRegistry](./Docs-FileUploadRegistry-Class-ar.md) و [توثيق كلاس FileUploadService](./Docs-FileUploadService-Class-ar.md).

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [توثيق كلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)



