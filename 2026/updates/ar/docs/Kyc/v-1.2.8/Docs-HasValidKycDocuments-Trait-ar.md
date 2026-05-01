# توثيق سمة `HasValidKycDocuments` – مدقق KYC السريع مع تخزين مؤقت متقدم

**Namespace:** `Nano3\Kyc\Classes\Manager`  
**الهدف:** توفير دوال سريعة وعالية الأداء للتحقق من وجود وثائق KYC صالحة لمالك معين، مع دعم تخزين مؤقت متعدد المستويات وأقفال ذرية لمنع تضارب الطلبات.

---

## نظرة عامة

سمة `HasValidKycDocuments` هي جزء من كلاس `Manager` في إضافة `Nano3.Kyc`. توفر هذه السمة مجموعة من الدوال المصممة خصيصًا للتحقق **السريع** من حالة KYC (نعم/لا) دون الحاجة إلى تحليل شامل أو حسابات معقدة. تُستخدم هذه الدوال بشكل مثالي في:

- **Middleware** لفحص صلاحيات الوصول.
- **Policies** لتحديد ما إذا كان المستخدم يمكنه تنفيذ إجراء معين.
- **استجابات API** التي تحتاج إلى إشارة سريعة عن حالة KYC (مثل `has_verified_identity: true`).
- **الواجهات الأمامية** لتحديد ما إذا كان يجب عرض مطالبة برفع وثائق.

**الميزات الرئيسية:**

- **أداء فائق**: تستخدم استعلامات محسّنة مع `exists()` و `select()` محدود، وتتوقف عند أول وثيقة صالحة.
- **تخزين مؤقت ذكي**: تدعم Cache Tags (لـ Redis/Memcached) مع آلية احتياطية للمخازن التي لا تدعمها. مدة التخزين الافتراضية 5 دقائق.
- **أقفال ذرية (Atomic Locks)**: تمنع "تسابق الطلبات" (Cache Stampede) عند محاولة عدة عمليات إعادة بناء نفس نتيجة الكاش في وقت واحد.
- **مسح تلقائي للكاش**: يتم مسح الكاش المرتبط بالمالك تلقائيًا عند إنشاء، تحديث، حذف، اعتماد، أو رفض أي وثيقة تخصه (عبر أحداث الموديل).
- **دعم المجموعات البديلة**: يمكن تمرير `group:primary_identity` أو `group:address_verification` للتحقق من وجود أي وثيقة ضمن مجموعة محددة.
- **خيارات مرنة**: التحكم في تضمين الوثائق المنتهية، المعلقة، المرفوضة، اشتراط وجود جميع الأنواع، والحد الأدنى لدرجة التحقق.

---

## الدوال العامة (Public Methods)

### `hasValidKycDocumentByCategory`

التحقق من وجود وثيقة KYC صالحة واحدة على الأقل (أو جميع الأنواع المحددة) ضمن تصنيف وأنواع محددة.

```php
public static function hasValidKycDocumentByCategory(
    $owner,
    ?string $ownerType = null,
    ?int $ownerId = null,
    ?string $category = null,
    $documentType = null,
    array $options = []
): bool
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$owner` | `object\|string` | كائن المالك **أو** نوع المالك. |
| `$ownerType` | `?string` | نوع المالك (مطلوب إذا كان `$owner` ليس كائنًا). |
| `$ownerId` | `?int` | معرف المالك (مطلوب إذا كان `$owner` ليس كائنًا). |
| `$category` | `?string` | تصنيف الوثائق (افتراضي `CATEGORY_PRIMARY_ID`). |
| `$documentType` | `string\|array\|null` | نوع وثيقة محدد، مصفوفة أنواع، أو سلسلة `group:primary_identity` (اختياري). |
| `$options` | `array` | خيارات إضافية (انظر جدول الخيارات أدناه). |

#### خيارات `$options`

| الخيار | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `include_expired` | `bool` | `false` | هل الوثائق المنتهية تعتبر صالحة؟ |
| `include_pending` | `bool` | `false` | هل الوثائق قيد الانتظار تعتبر صالحة؟ |
| `include_rejected` | `bool` | `false` | هل الوثائق المرفوضة تعتبر صالحة؟ |
| `require_all_types` | `bool` | `false` | إذا كان `true`، يجب أن تكون **جميع** الأنواع المحددة موجودة وصالحة. |
| `min_verification_score` | `int\|null` | `null` | الحد الأدنى لدرجة التحقق (0-100) المطلوبة. |
| `check_validity` | `bool` | `true` | هل يتم التحقق من صلاحية الوثيقة (تاريخ الانتهاء/العمر الأقصى)؟ |
| `use_cache` | `bool` | `true` | هل يتم استخدام التخزين المؤقت؟ |
| `cache_ttl` | `int` | `300` | مدة صلاحية الكاش بالثواني (5 دقائق). |
| `use_atomic_lock` | `bool` | `true` | استخدام قفل ذري لمنع تضارب الكاش في الطلبات المتزامنة. |

#### القيمة المرجعة

`bool` – `true` إذا تم استيفاء الشروط، `false` في جميع الحالات الأخرى (بما في ذلك الأخطاء).

---

### `hasValidKycDocument`

دالة مختصرة للتحقق من وجود **أي** وثيقة KYC صالحة للمالك (بغض النظر عن التصنيف أو النوع).

```php
public static function hasValidKycDocument(
    $owner,
    ?string $ownerType = null,
    ?int $ownerId = null
): bool
```

تعادل استدعاء `hasValidKycDocumentByCategory` مع `$category = null` و `$documentType = null` والخيارات الافتراضية (`check_validity = true`).

---

## أمثلة عملية متكاملة

### 1. التحقق من وجود هوية أساسية صالحة (جواز سفر أو بطاقة وطنية أو رخصة)

```php
use Nano3\Kyc\Classes\Manager;
use Nano3\Kyc\Classes\DocumentType;

$user = Auth::getUser();

$hasValidId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity'
);

if ($hasValidId) {
    echo "المستخدم لديه هوية أساسية صالحة.";
} else {
    echo "يرجى رفع جواز سفر، بطاقة هوية، أو رخصة قيادة سارية.";
}
```

### 2. التحقق من وجود جواز سفر **فقط** بدرجة تحقق 90+

```php
$hasHighScorePassport = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    DocumentType::TYPE_PASSPORT,
    ['min_verification_score' => 90]
);
```

### 3. التحقق من وجود جواز سفر **و** رخصة قيادة معًا (كلاهما مطلوب)

```php
$hasBoth = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    [DocumentType::TYPE_PASSPORT, DocumentType::TYPE_DRIVERS_LICENSE],
    ['require_all_types' => true]
);
```

### 4. التحقق من وجود أي وثيقة إثبات عنوان (بما فيها الفواتير المعلقة)

```php
$hasAddress = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_ADDRESS,
    null,
    ['include_pending' => true]
);
```

### 5. استخدام الدالة المختصرة للتحقق من وجود أي وثيقة KYC (للحالات البسيطة)

```php
if (Manager::hasValidKycDocument($user)) {
    echo "المستخدم لديه وثيقة KYC واحدة على الأقل.";
}
```

### 6. استخدام في Middleware لحماية مسار يتطلب هوية مثبتة

```php
namespace App\Http\Middleware;

use Closure;
use Auth;
use Nano3\Kyc\Classes\Manager;
use Nano3\Kyc\Classes\DocumentType;

class RequireVerifiedIdentity
{
    public function handle($request, Closure $next)
    {
        $user = Auth::getUser();
        
        if (!$user) {
            return response()->json(['error' => 'Unauthenticated'], 401);
        }

        // التحقق من وجود هوية أساسية صالحة (مع استخدام الكاش تلقائياً)
        if (!Manager::hasValidKycDocumentByCategory(
            $user,
            null,
            null,
            DocumentType::CATEGORY_PRIMARY_ID,
            'group:primary_identity'
        )) {
            return response()->json([
                'error' => 'Identity verification required',
                'message' => 'Please upload a valid passport, national ID, or driver\'s license.'
            ], 403);
        }

        return $next($request);
    }
}
```

### 7. استخدام مع خيارات مخصصة للكاش (مدة أطول)

```php
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    null,
    ['use_cache' => true, 'cache_ttl' => 3600] // ساعة كاملة
);
```

### 8. تعطيل التحقق من الصلاحية (للأغراض الإحصائية أو التاريخية)

```php
// حتى لو كانت الوثيقة منتهية الصلاحية، تعتبر موجودة
$hasAnyPassport = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    DocumentType::TYPE_PASSPORT,
    ['check_validity' => false, 'include_expired' => true]
);
```

---

## آلية التخزين المؤقت والأقفال الذرية

### كيف يعمل الكاش؟

1. عند استدعاء الدالة لأول مرة، يتم إنشاء مفتاح كاش فريد بناءً على جميع المعاملات (المالك، التصنيف، الأنواع، الخيارات).
2. إذا كان `use_cache = true`، يتم البحث عن المفتاح في الكاش. إذا وجدت نتيجة، تُعاد فوراً دون استعلام قاعدة البيانات.
3. إذا لم توجد نتيجة، يتم تنفيذ الاستعلام الفعلي، ثم تخزين النتيجة في الكاش.
4. يتم أيضًا تسجيل المفتاح في مجموعة خاصة بالمالك (أو استخدام Cache Tags) لتسهيل مسح جميع مفاتيحه لاحقًا.

### الأقفال الذرية (Atomic Locks)

في السيناريوهات ذات التزامن العالي (مثل تحديث صفحة رئيسية تعرض بيانات آلاف المستخدمين مع حالة KYC)، قد تحاول عدة عمليات متزامنة حساب نفس النتيجة غير الموجودة في الكاش بعد. هذا يؤدي إلى "تسابق الطلبات" (Cache Stampede) وإهدار موارد قاعدة البيانات.

تستخدم السمة `Cache::lock()` لضمان أن **عملية واحدة فقط** هي التي تنفذ الاستعلام الفعلي، بينما تنتظر العمليات الأخرى (حتى 5 ثوانٍ) ثم تستخدم النتيجة المخزنة حديثًا. هذا يحمي قاعدة البيانات من الضغط غير الضروري.

### مسح الكاش التلقائي

تم ربط موديل `Document` بالسمة: في `afterSave` و `afterDelete`، يتم استدعاء `Manager::clearKycCheckCache($ownerType, $ownerId)` تلقائيًا. هذا يضمن أن أي تغيير في وثائق المستخدم (رفع جديد، تحديث، حذف، اعتماد، رفض) يؤدي إلى مسح جميع نتائج الكاش المرتبطة به، مما يحافظ على دقة البيانات.

---

## أفضل الممارسات

1. **استخدم `hasValidKycDocumentByCategory` في Middleware وPolicies**: هي مثالية للتحقق من الشروط قبل السماح بالوصول إلى الموارد.
2. **لا تستخدمها لإنشاء تقارير مفصلة**: هذه الدوال مصممة لإجابات نعم/لا سريعة. للتحليلات الشاملة، استخدم `assessKycStatus`.
3. **استفد من المجموعات البديلة**: استخدم `'group:primary_identity'` بدلاً من سرد جميع الأنواع يدويًا، خاصة مع إمكانية تغيير قوائم المجموعات مستقبلًا.
4. **اترك إعدادات الكاش الافتراضية**: 5 دقائق توفر توازنًا ممتازًا بين حداثة البيانات والأداء. يمكن زيادتها للتطبيقات التي لا تتغير فيها الوثائق كثيرًا.
5. **لا تقم بتعطيل `use_atomic_lock` إلا إذا كنت متأكدًا**: في بيئات الإنتاج ذات الحركة العالية، القفل الذري يحمي قاعدة البيانات بشكل كبير.
6. **تأكد من أن `Document` يمسح الكاش**: تم تضمين ذلك افتراضيًا في الموديل. لا تقم بإزالته إلا إذا كنت تدير الكاش يدويًا بطريقة أخرى.

---

## التوثيق الإضافي

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق موديل `Document`](./Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
