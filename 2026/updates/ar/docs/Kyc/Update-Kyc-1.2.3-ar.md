## 2026-04-21 - 2026-04-22

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.3**

### ملخص التحديثات

بعد إعادة هيكلة منطق تقييم KYC في الإصدار 1.2.2 وإدخال مفهوم "المجموعات البديلة" للوثائق، نقدم في الإصدار 1.2.3 تحسينات جذرية على أداء وسرعة التحقق من حالة KYC. تم نقل دوال التحقق السريع إلى سمة مستقلة `HasValidKycDocuments` مع دعم متقدم للتخزين المؤقت والأقفال الذرية، مما يجعل عمليات الفحص المتكررة (مثل التحقق من وجود هوية أو إثبات عنوان) خفيفة للغاية على قاعدة البيانات ومناسبة للاستخدام في Middleware وPolicies وكل طلب API.

---

### الإصدار 1.2.3 – تحسينات الأداء والتخزين المؤقت لدوال التحقق السريع

#### أهداف الإصدار

- نقل منطق `hasValidKycDocumentByCategory` إلى سمة مستقلة `HasValidKycDocuments` لتحسين قابلية الصيانة.
- دعم **المجموعات البديلة** (`group:primary_identity`, `group:address_verification`) داخل دوال التحقق السريع.
- تنفيذ **تخزين مؤقت متعدد المستويات** مع دعم Cache Tags (لـ Redis/Memcached) ومجموعات مفاتيح احتياطية.
- إضافة **أقفال ذرية (Atomic Locks)** لمنع "تسابق الطلبات" (Cache Stampede) في أوقات الذروة.
- مسح الكاش تلقائياً عند أي تغيير في وثائق المستخدم (إنشاء، تحديث، حذف، اعتماد، رفض).
- توفير دالة مختصرة `hasValidKycDocument()` للتحقق من وجود أي وثيقة صالحة.

#### الميزات الجديدة

##### 1. سمة `HasValidKycDocuments`

تم نقل جميع دوال التحقق السريع إلى سمة جديدة `Nano3\Kyc\Classes\Manager\HasValidKycDocuments` والتي توفر:

| الدالة | الوصف |
| :--- | :--- |
| `hasValidKycDocumentByCategory($owner, $ownerType, $ownerId, $category, $documentType, $options)` | التحقق من وجود وثيقة صالحة ضمن تصنيف وأنواع محددة مع خيارات متقدمة. |
| `hasValidKycDocument($owner, $ownerType, $ownerId)` | دالة مختصرة للتحقق من وجود **أي** وثيقة KYC صالحة للمالك. |
| `clearKycCheckCache($ownerType, $ownerId)` | مسح جميع نتائج الكاش المرتبطة بمالك معين (تُستدعى تلقائياً عند تغيير الوثائق). |

##### 2. دعم المجموعات البديلة في التحقق السريع

يمكن الآن تمرير `group:primary_identity` أو `group:address_verification` ضمن `$documentType`، وسيتم توسيعها تلقائياً إلى قائمة الأنواع الفردية المناسبة. هذا يضمن الاتساق مع منطق `assessKycStatus` الجديد.

**مثال:**
```php
// التحقق من وجود أي هوية أساسية (جواز سفر، هوية وطنية، رخصة قيادة)
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity'
);
```

##### 3. تخزين مؤقت متعدد المستويات

لتقليل الحمل على قاعدة البيانات في الاستدعاءات المتكررة (مثل `include=kyc_status` لكل مستخدم)، تم تطبيق آلية تخزين مؤقت ذكية:

- **Cache Tags (للأنظمة الداعمة)**: يتم تجميع مفاتيح الكاش تحت وسم `owner_{hash}`، مما يسمح بمسحها جميعاً دفعة واحدة عند الحاجة.
- **مجموعة مفاتيح احتياطية (Fallback)**: إذا كان مزود الكاش لا يدعم Tags (مثل `file`)، يتم تخزين المفاتيح في مصفوفة منفصلة (`owner_keys`) تُستخدم للمسح اليدوي.
- **مدة صلاحية افتراضية 5 دقائق** (قابلة للتعديل عبر `cache_ttl`).

##### 4. أقفال ذرية (Atomic Locks)

في حالات التزامن العالي (عدة طلبات لنفس المستخدم في نفس اللحظة)، قد تحاول عدة عمليات إعادة بناء نفس نتيجة الكاش مما يهدر الموارد. تمت إضافة قفل ذري (`Cache::lock`) يضمن أن عملية واحدة فقط هي التي تنفذ الاستعلام الفعلي، بينما تنتظر العمليات الأخرى وتستخدم النتيجة المخزنة.

**الخيارات الجديدة في `$options`:**
| الخيار | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `use_cache` | `bool` | `true` | تفعيل/تعطيل التخزين المؤقت. |
| `cache_ttl` | `int` | `300` | مدة صلاحية الكاش بالثواني. |
| `use_atomic_lock` | `bool` | `true` | تفعيل/تعطيل القفل الذري. |

##### 5. مسح الكاش التلقائي عند تغيير الوثائق

تم تعديل موديل `Document` (`afterSave` و `afterDelete`) لاستدعاء `Manager::clearKycCheckCache()` تلقائياً عند أي تغيير يطرأ على وثيقة (إنشاء، تحديث، حذف، اعتماد، رفض). هذا يضمن أن نتائج التحقق السريع تعكس دائماً أحدث حالة للمستخدم دون الحاجة إلى تدخل يدوي.

##### 6. تحسينات أداء الاستعلامات

- استخدام `select()` لتحديد الأعمدة الضرورية فقط (`id`, `document_type`, `status`, `issue_date`, `expiry_date`, `verification_score`).
- استخدام `exists()` بدلاً من `get()` عندما لا تكون هناك حاجة للتحقق من الصلاحية (`check_validity = false` و `require_all_types = false`).
- ترتيب `$requiredTypes` لضمان اتساق مفاتيح الكاش.

---

### أمثلة عملية

#### 1. التحقق من وجود هوية أساسية صالحة (مع استخدام الكاش)

```php
$hasId = Manager::hasValidKycDocumentByCategory(
    $user,
    null,
    null,
    DocumentType::CATEGORY_PRIMARY_ID,
    'group:primary_identity',
    ['use_cache' => true, 'cache_ttl' => 600] // 10 دقائق
);

if ($hasId) {
    // السماح بالوصول
}
```

#### 2. التحقق من وجود أي وثيقة عنوان (بما فيها المعلقة)

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

#### 3. استخدام الدالة المختصرة للتحقق من أي وثيقة

```php
if (Manager::hasValidKycDocument($user)) {
    echo "المستخدم لديه وثيقة KYC واحدة صالحة على الأقل.";
}
```

#### 4. استخدام في Middleware لحماية المسارات

```php
class RequireKycMiddleware
{
    public function handle($request, Closure $next)
    {
        $user = Auth::getUser();
        if (!Manager::hasValidKycDocumentByCategory($user, null, null, DocumentType::CATEGORY_PRIMARY_ID, 'group:primary_identity')) {
            return response()->json(['error' => 'KYC verification required'], 403);
        }
        return $next($request);
    }
}
```

---

### ملخص الإصدارات (1.2.2 – 1.2.3)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.2.2 | إعادة هيكلة منطق التقييم إلى سمة `HasAssessKycStatus`، إضافة المجموعات البديلة للوثائق، تحسين دقة حسابات التقييم. |
| 1.2.3 | نقل دوال التحقق السريع إلى سمة `HasValidKycDocuments`، تخزين مؤقت متعدد المستويات، أقفال ذرية، دعم المجموعات البديلة في التحقق السريع، مسح تلقائي للكاش، تحسينات أداء شاملة. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملفات `Manager.php`، `Document.php`، وإضافة الملفات الجديدة:
     - `classes/Manager/HasValidKycDocuments.php`
     - `classes/Manager/HasAssessKycStatus.php` (إن لم يكن موجوداً)

2. **تشغيل هجرات قاعدة البيانات** (لا توجد تغييرات في هذا الإصدار).

3. **تحديث تعريفات API** (لا توجد تغييرات في نقاط النهاية).

4. **اختبار الأداء**:
   - راقب أداء نقاط النهاية التي تستخدم `include=kyc_status` بعد التفعيل؛ يجب أن تشهد تحسناً ملحوظاً بسبب الكاش.

5. **مسح الكاش يدوياً (اختياري)**:
   - إذا كنت ترغب في تطبيق الكاش فوراً، يمكنك مسح الكاش العام للتطبيق: `php artisan cache:clear`.

---

### الخاتمة

يمثل الإصدار 1.2.3 نقلة نوعية في أداء عمليات التحقق من KYC. بفضل التخزين المؤقت الذكي والأقفال الذرية، أصبحت الدوال التي تُستدعى آلاف المرات يومياً (مثل فحص حالة الهوية) سريعة جداً ولا تثقل كاهل قاعدة البيانات. كما أن مسح الكاش التلقائي يضمن دقة النتائج في جميع الأوقات. هذه التحسينات تجعل إضافة `Nano3.Kyc` جاهزة للاستخدام في المشاريع الضخمة ذات الحركة المرورية العالية دون أي قلق بشأن الأداء.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
