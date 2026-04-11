# توثيق كلاس `FileUploadUserManager`

**نبذة تعريفية عن حزمة `Nano.FileUpload` ووحدة إدارة المستخدمين والصلاحيات**

هذه المجموعة من الكلاسات (`FileUploadRegistry`, `FileUploadService`, `FileUploadUserManager` وغيرها) هي جزء من **حزمة `Nano.FileUpload`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **نانوسوفت** بهدف توحيد وإدارة عمليات رفع الملفات عبر واجهات برمجة التطبيقات (API) بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano\FileUpload\Classes
```

توفر هذه الوحدة أدوات متقدمة لتسجيل النماذج (الموديولات) التي تحتوي على حقول رفع ملفات، والتحقق من صلاحيات المستخدمين (بما في ذلك أنواع المستخدمين المختلفة: `backend` و `frontend`)، وإدارة الملفات المرفوعة مؤقتًا قبل حفظ السجل، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة رفع ملفات قابلة للتوسع بسهولة وتلبية احتياجات التطبيقات المتقدمة دون المساس بأمان التطبيق أو أدائه.

---

### مقدمة

كلاس `FileUploadUserManager` هو **مدير المستخدمين والصلاحيات** في نظام رفع الملفات في تطبيقات نانوسوفت. يقوم هذا الكلاس بجلب المستخدم الحالي من مصادر متعددة (باك إند، فرونت إند، أي نموذج مخصص)، ويوفر واجهة موحدة للتحقق من صلاحيات المستخدمين بناءً على نوعهم (backend/frontend) والصلاحيات المطلوبة.

ببساطة، `FileUploadUserManager` هو **الطبقة المسؤولة عن معرفة من هو المستخدم الحالي وما إذا كان يملك الصلاحيات اللازمة لتنفيذ عملية معينة**، مع إمكانية تخصيص دالة حل المستخدم ودالة التحقق من الصلاحيات حسب متطلبات التطبيق.

**ملاحظة الإصدار:** بدءاً من الإصدار 1.1.0، أصبحت عملية `edit` (استبدال الملفات) مدعومة وتستخدم نفس آلية التحقق من الصلاحيات مع تمرير `$operation = 'edit'`.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$userResolver` | `callable|null` | دالة مخصصة لحل المستخدم الحالي (تجاوز المحلل الافتراضي). |
| `$user` | `mixed|null` | المستخدم الحالي بعد حله (مخبأ لتحسين الأداء). |
| `$permissionChecker` | `callable|null` | دالة مخصصة للتحقق من الصلاحيات (تجاوز المنطق الافتراضي). |

---

### أهم الطرق (Methods)

#### 1. إدارة محلل المستخدم

##### `public function setUserResolver(callable $resolver): self`
- **الهدف**: تعيين دالة مخصصة لحل المستخدم الحالي.
- **المعاملات**:
  - `$resolver`: دالة تُستدعى وتعيد كائن المستخدم أو `null`.
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.
- **مثال**:
  ```php
  $manager->setUserResolver(function() {
      return \Auth::user(); // أو أي منطق لجلب المستخدم
  });
  ```

##### `public function setUser($user): self`
- **الهدف**: تعيين المستخدم مباشرة (يتجاوز المحلل).
- **المعاملات**:
  - `$user`: كائن المستخدم أو `null`.
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.

##### `public function getUser()`
- **الهدف**: الحصول على المستخدم الحالي (مع التخزين المؤقت).
- **الإرجاع**: كائن المستخدم أو `null`.
- **السلوك**:
  - إذا كان `$user` مخبأ، يعيده.
  - إذا كان هناك محلل مخصص، يستدعيه ويخبأ النتيجة.
  - وإلا يستدعي المحلل الافتراضي.

##### `public function clearUser(): self`
- **الهدف**: إعادة تعيين المستخدم المخبأ (لإجبار إعادة الحل في المرة القادمة).
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.

#### 2. إدارة دالة التحقق من الصلاحيات

##### `public function setPermissionChecker(callable $checker): self`
- **الهدف**: تعيين دالة مخصصة للتحقق من الصلاحيات.
- **المعاملات**:
  - `$checker`: دالة تستقبل `($user, $operation, $permissions)` وتعيد `bool`.
- **الإرجاع**: الكائن الحالي لتسلسل الاستدعاءات.
- **مثال**:
  ```php
  $manager->setPermissionChecker(function($user, $operation, $permissions) {
      return $user->hasCustomPermission($operation, $permissions);
  });
  ```

#### 3. الحصول على معلومات المستخدم

##### `public function getUserType(): ?string`
- **الهدف**: تحديد نوع المستخدم الحالي.
- **الإرجاع**: `'backend'`, `'frontend'`, `'guest'`, أو `null` إذا لم يُعرف.
- **السلوك**:
  - يتحقق من نوع الكائن: إذا كان `Backend\Models\User` يعيد `'backend'`.
  - إذا كان `RainLab\User\Models\User` يعيد `'frontend'`.
  - إذا كان المستخدم غير موجود يعيد `'guest'`.
  - أي نموذج آخر يعتبر `'frontend'` افتراضيًا.

##### `public function getId()`
- **الهدف**: الحصول على معرف المستخدم الحالي.
- **الإرجاع**: معرف المستخدم (`int`, `string`, `uuid`) أو `null`.
- **السلوك**:
  - يستخدم `getKey()` إذا كان موجودًا.
  - وإلا يبحث عن `id` أو `uuid`.

#### 4. التحقق من الصلاحيات

##### `public function checkPermission($operation, $permissions = null, $user = null): bool`
- **الهدف**: التحقق مما إذا كان المستخدم يمتلك الصلاحية المطلوبة.
- **المعاملات**:
  - `$operation`: العملية (`add`, `edit`, `delete`, `view`) – قد لا تُستخدم مباشرة في كل الحالات، ولكنها تمرر إلى الدالة المخصصة إن وجدت.
  - `$permissions`: صلاحية واحدة (string) أو مصفوفة صلاحيات (array) – تُستخدم أساسًا لمستخدمي backend.
  - `$user`: (اختياري) كائن المستخدم، إذا لم يُمرر يستخدم المستخدم الحالي.
- **الإرجاع**: `true` إذا كان مسموحًا، وإلا `false`.
- **السلوك**:
  - يحصل على نوع المستخدم من الكائن.
  - إذا كان المستخدم من نوع **backend** وتم تمرير صلاحيات، يستخدم `checkBackendPermissions()` للتحقق باستخدام `hasAccess` أو `hasPermission`.
  - إذا كان المستخدم من نوع **frontend** أو لم تمرر صلاحيات، يتحقق:
    - إذا كان هناك `permissionChecker` مخصص، يستخدمه.
    - وإلا إذا كان للمستخدم دالة `canUpload`، يستدعيها.
    - وإلا يعيد `true` (السماح الافتراضي لمستخدمي frontend).

##### `protected function checkBackendPermissions($user, $permissions): bool`
- **الهدف**: التحقق من صلاحيات مستخدم backend باستخدام دوال `hasAccess` أو `hasPermission`.
- **المعاملات**:
  - `$user`: كائن المستخدم (من نوع backend).
  - `$permissions`: صلاحية واحدة أو مصفوفة.
- **الإرجاع**: `true` إذا كان المستخدم يمتلك أيًا من الصلاحيات المطلوبة، وإلا `false`.

#### 5. المحلل الافتراضي

##### `protected function defaultUserResolver()`
- **الهدف**: المحلل الافتراضي للمستخدم الحالي.
- **السلوك**:
  - يحاول أولاً استخدام `Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()` إذا كان متاحًا (من إضافة `Nano.Api`).
  - ثم يحاول `BackendAuth::getUser()`.
  - ثم `Auth::getUser()`.
  - ثم `auth()->user()`.
- **الإرجاع**: كائن المستخدم أو `null`.

##### `public static function defaultResolver(): callable`
- **الهدف**: إنشاء دالة محلل افتراضي (مفيد للاختبارات).
- **الإرجاع**: دالة (callable) تعيد نتيجة `defaultUserResolver()`.

---

### الأحداث (Events)

هذا الكلاس لا يطلق أحداثًا بشكل مباشر، ولكنه يُستخدم في سياق `FileUploadRegistry` و `FileUploadService` التي قد تطلق أحداثًا خاصة بها (مثل `beforeUpload`, `afterEditDelete`).

---

### أمثلة عملية شاملة

#### مثال 1: استخدام المحلل الافتراضي (جلب مستخدم backend)

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();
if ($user) {
    echo "نوع المستخدم: " . $manager->getUserType(); // 'backend'
    echo "المعرف: " . $manager->getId();
    echo "الاسم: " . $user->full_name;
}
```

#### مثال 2: تعيين محلل مخصص لمستخدمي frontend

```php
$manager->setUserResolver(function() {
    return Auth::user(); // مستخدم RainLab.User
});

$user = $manager->getUser();
echo "نوع المستخدم: " . $manager->getUserType(); // 'frontend'
```

#### مثال 3: التحقق من صلاحية مستخدم backend (عملية `add`)

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();

if ($manager->checkPermission('add', 'nano.shop.upload_image', $user)) {
    echo "مسموح برفع الصورة";
} else {
    echo "غير مسموح";
}
```

#### مثال 4: التحقق من صلاحية `edit` لاستبدال ملف (جديد في الإصدار 1.1.0)

```php
$manager = FileUploadUserManager::instance();
$user = $manager->getUser();
$hasEditPermission = $manager->checkPermission('edit', 'nano.shop.edit_image', $user);
if ($hasEditPermission) {
    echo "مسموح باستبدال الصورة";
}
```

#### مثال 5: تعيين دالة مخصصة للتحقق من الصلاحيات (مثلاً لمستخدمي frontend)

```php
$manager->setPermissionChecker(function($user, $operation, $permissions) {
    // منطق مخصص: مثلاً التحقق من جدول صلاحيات مخصص
    return $user->hasCustomPermission($operation);
});

$allowed = $manager->checkPermission('add', null, $user); // لا نمرر صلاحيات backend
```

#### مثال 6: استخدام `canUpload` في نموذج المستخدم (frontend)

إذا كان نموذج المستخدم يحتوي على دالة `canUpload`، يمكن استخدامها:

```php
class User extends Model {
    public function canUpload($operation, $permissions) {
        return $this->hasRole('editor') || $this->id == 1;
    }
}

// في مكان آخر
$manager = FileUploadUserManager::instance();
if ($manager->checkPermission('add', null, $user)) {
    // سيتم استدعاء canUpload تلقائيًا
}
```

#### مثال 7: دمج مع `FileUploadRegistry` للتحقق من صلاحية نموذج معين (بما في ذلك `edit`)

```php
$registry = FileUploadRegistry::instance();
$userManager = FileUploadUserManager::instance();

$modelClass = Product::class;
$field = 'image';
$user = $userManager->getUser();
$userType = $userManager->getUserType();

if ($registry->can($modelClass, 'edit', $userType, $user, $field)) {
    // مسموح باستبدال الصورة
}
```

#### مثال 8: إعادة تعيين المستخدم بعد تغيير الجلسة

```php
$manager->clearUser();
$newUser = $manager->getUser(); // يعيد حل المستخدم من جديد
```

---

### التفاعل مع الكلاسات الأخرى

- **مع `FileUploadRegistry`**: يستخدم `FileUploadRegistry` كائن `FileUploadUserManager` للتحقق من الصلاحيات في دالة `can()` (لعمليات `add`, `edit`, `delete`, `view`).
- **مع `FileUploadService`**: يستخدم `FileUploadService` كائن `FileUploadUserManager` للحصول على المستخدم الحالي ونوعه (`getUser()`, `getUserType()`)، ويدعو `checkPermission()` عند الحاجة.
- **مع `Base64` (المنقول إلى داخل الإضافة)**: ليس تفاعل مباشر، لكن `FileUploadService` يستخدم `Base64` للرفع.

---

### أفضل الممارسات

1. **استخدم المحلل الافتراضي ما لم يكن لديك متطلبات خاصة**: يوفر المحلل الافتراضي دعمًا شاملاً لمستخدمي backend و frontend.
2. **خصص المحلل فقط إذا كنت بحاجة إلى منطق مختلف**: مثلاً إذا كان لديك نظام مستخدمين مخصص غير المدعوم افتراضيًا.
3. **استخدم `setPermissionChecker` لتوحيد منطق التحقق من الصلاحيات لجميع المستخدمين**: هذا مفيد إذا كان نظام الصلاحيات لديك مخصصًا بالكامل.
4. **لا تعتمد على السماح الافتراضي لمستخدمي frontend**: إذا كنت بحاجة إلى تقييد، قم بتعيين دالة تحقق مخصصة أو استخدم `canUpload` في نموذج المستخدم.
5. **اختبر صلاحيات المستخدمين جيدًا**: تأكد من أن المستخدمين غير المصرح لهم لا يمكنهم تنفيذ عمليات رفع أو حذف أو استبدال.
6. **استخدم `clearUser()` بعد تغيير بيانات المستخدم أو تسجيل الخروج**: لتجنب الاحتفاظ ببيانات قديمة.
7. **لعمليات `edit` (استبدال الملفات)**، تأكد من أن المستخدم لديه الصلاحية المناسبة (`edit`) وأن `disable_edit` غير مفعّل.

---

### الأخطاء الشائعة وحلولها

| الخطأ | السبب | الحل |
|-------|-------|------|
| `getUser()` يعيد `null` رغم أن المستخدم مسجل دخول | المحلل الافتراضي لم يعثر على المستخدم (ربما بسبب اختلاف نظام المصادقة). | قم بتعيين محلل مخصص باستخدام `setUserResolver`. |
| `checkPermission` يعيد `true` دائمًا لمستخدمي frontend | لم يتم تعيين دالة تحقق مخصصة، والسماح الافتراضي هو `true`. | استخدم `setPermissionChecker` لتنفيذ منطق التحقق الخاص بك. |
| صلاحيات backend لا تعمل | المستخدم من نوع backend ولكن دالة `hasAccess` أو `hasPermission` غير موجودة. | تأكد من أن كائن المستخدم يحتوي على إحدى هذه الدوال (في نظام نانوسوفت عادةً تكون موجودة). |
| `getUserType` يعيد `'frontend'` لنموذج backend | النموذج لا يتطابق مع أي من الأنواع المعروفة. | أضف التحقق المناسب في دالة `getUserTypeFromModel` إذا كنت تستخدم نموذجًا مخصصًا. |
| المستخدم يتغير في الجلسة ولكن `getUser()` لا يعيد القيمة الجديدة | التخزين المؤقت يحتفظ بالقيمة القديمة. | استخدم `clearUser()` بعد تغيير المستخدم. |
| `checkPermission` يعيد `false` رغم أن المستخدم عنده صلاحية `edit` | العملية `'edit'` غير معرفة في نظام الصلاحيات أو `disable_edit` مفعّل عالمياً. | تأكد من أن صلاحية `edit` مضبوطة في `FileUploadRegistry` وأن `disable_edit = false`. |

---

### دمج مع نظام الصلاحيات في نانوسوفت

في تطبيقات نانوسوفت، يتم عادةً استخدام `FileUploadUserManager` بشكل غير مباشر عبر `FileUploadService`. ومع ذلك، يمكن تخصيصه ليتناسب مع أي نظام صلاحيات:

- لمستخدمي **backend**: يعتمد على `hasAccess` أو `hasPermission` الموجودين في نموذج `Backend\Models\User`.
- لمستخدمي **frontend**: يمكن تخصيص دالة `canUpload` في نموذج المستخدم، أو تعيين `permissionChecker` مخصص.

**ملاحظة:** بدءاً من الإصدار 1.1.0، أصبحت عملية `edit` (استبدال الملفات) مدعومة وتستخدم نفس آلية التحقق من الصلاحيات مع تمرير `$operation = 'edit'`.

---

### الخلاصة

كلاس `FileUploadUserManager` هو العمود الفقري لإدارة هوية المستخدم وصلاحياته في نظام رفع الملفات في تطبيقات نانوسوفت. يوفر واجهة مرنة للحصول على المستخدم الحالي والتحقق من صلاحياته، مع دعم لأنواع المستخدمين المختلفة (backend/frontend) وإمكانية تخصيص محلل المستخدم ودالة التحقق. بفضل هذا التصميم، يمكن للمطورين دمج النظام بسهولة مع أي آلية مصادقة وصلاحيات موجودة في تطبيقاتهم. كما يدعم الإصدارات الحديثة (1.1.0+) عملية `edit` بشكل كامل، مما يتيح التحكم في صلاحيات استبدال الملفات.

للاطلاع على كيفية استخدام هذا الكلاس في سياق أوسع، راجع [توثيق كلاس FileUploadRegistry](./Docs-FileUploadRegistry-Class-ar.md) و [توثيق كلاس FileUploadService](./Docs-FileUploadService-Class-ar.md).

---

## التوثيق الإضافي

- [التوثيق العام للإضافة](./Docs-FileUpload-ar.md)
- [توثيق كلاس `FileUploadRegistry`](./Docs-FileUploadRegistry-Class-ar.md)
- [توثيق كلاس `FileUploadService`](./Docs-FileUploadService-Class-ar.md)
- [أمثلة متقدمة لكلاس `FileUploadUserManager`](./Docs-FileUploadUserManager-Class-Advenced-Examples-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)