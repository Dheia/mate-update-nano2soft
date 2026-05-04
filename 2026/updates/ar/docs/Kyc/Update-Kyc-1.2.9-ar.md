## 2026-05-01 - 2026-05-02

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.9**

### ملخص التحديثات

يُمثل الإصدار 1.2.9 نقلة نوعية في طريقة ربط وثائق KYC بالموديلات المختلفة داخل برمجيات نانوسوفت (NanoSoft App). بعد أن ركزت الإصدارات السابقة على إدارة الوثائق نفسها (إنشاء، تحديث، تحقق، تقييم)، يأتي هذا الإصدار ليُقدّم **نظاماً متكاملاً لإدارة تكاملات KYC** مع أي موديل (مستخدمين، عملاء، منشآت، موظفين...).

بدلاً من كتابة كود مكرر لكل موديل، يوفر الإصدار الجديد:

- **سلوك `KycDocumentModel`** جاهز يُضاف إلى أي موديل ليمنحه علاقة `documents` ونطاقات فلترة متقدمة وإحصائيات جاهزة.
- **كلاس `KycDocumentManager`** (Singleton) لإدارة مركزية لجميع التكاملات (سلوك، نماذج، قوائم، فلاتر، استعلامات API، محولات).
- **معالج `KycIntegrationHandler`** لربط الأحداث تلقائياً وتطبيق التكاملات دون تدخل يدوي.
- **معالجة ذكية لمعاملات API** تدعم صيغ متعددة (comma, pipe, JSON, auto-detect) ومجموعات معقدة بمنطق `AND`/`OR`.

كل ذلك مع توثيق شامل ومفصل بالعربية والإنجليزية.

---

### الإصدار 1.2.9 – نظام تكاملات KYC المتكامل

#### أهداف الإصدار

- **توفير سلوك موحد** (`KycDocumentModel`) يضيف علاقة الوثائق ونطاقات الفلترة والإحصائيات إلى أي موديل.
- **إنشاء كلاس إدارة مركزي** (`KycDocumentManager`) لتسجيل النماذج وإدارة جميع أنواع التكاملات بشكل تصريحي.
- **أتمتة التطبيق** عبر `KycIntegrationHandler` الذي يربط الأحداث تلقائياً.
- **دعم فلترة API متقدمة** مع معالجات متعددة للقيم ومجموعات معقدة ومنطق `AND`/`OR`.
- **توثيق احترافي** يغطي كل الكلاسات الجديدة مع أمثلة عملية متنوعة.

#### الميزات الجديدة

##### 1. سلوك `KycDocumentModel` مع السمة `KycDocumentScopesAndHelpers`

تم إنشاء سلوك جديد `Nano3\Kyc\Behaviors\KycDocumentModel` يمكن إضافته إلى أي موديل. يقوم السلوك تلقائياً بـ:

- تعريف علاقة `documents` (morphMany) مع موديل `Document` عبر حقل `owner`.
- توفير نطاقات متقدمة:
  - `scopeWhereHasDocuments` – فلترة حسب أي خصائص للوثائق (نوع، فئة، حالة، تحقق، تاريخ انتهاء...) مع دعم `count` و `boolean` و `not`.
  - `scopeHasVerifiedDocuments`، `scopeHasDocumentType`، `scopeHasDocumentCategory`، `scopeHasDocumentStatus` – نطاقات مختصرة للاستخدام الشائع.
  - `scopeAddDocumentsCount` – إضافة عمود عدد الوثائق إلى نتائج الاستعلام.
  - `scopeSortByDocumentsCount` – ترتيب النتائج حسب عدد الوثائق.
- دوال إحصائية (Accessors):
  - `getDocumentsCountAttribute` – إجمالي عدد الوثائق.
  - `getVerifiedDocumentsCountAttribute` – عدد الوثائق المعتمدة.
  - `getPendingDocumentsCountAttribute` – عدد الوثائق المعلقة.
- دوال فحص سريعة:
  - `hasAnyDocument()`، `hasVerifiedDocument()`، `hasDocumentOfType($type)`.

**مثال للاستخدام المباشر بعد تطبيق السلوك:**

```php
// فلترة المستخدمين الذين لديهم وثيقة هوية معتمدة
User::whereHasDocuments([
    'category'    => 'primary_id',
    'is_verified' => true,
])->get();

// جلب مستخدم مع عدد وثائقه
$user = User::find(1);
echo $user->documents_count;          // 5
echo $user->verified_documents_count; // 3

// التحقق السريع
if ($user->hasVerifiedDocument()) {
    // السماح بالعملية
}
```

##### 2. كلاس `KycDocumentManager` – المدير المركزي للتكاملات

تم بناء كلاس `Nano3\Kyc\Classes\KycDocumentManager` كـ Singleton لإدارة جميع تكاملات وثائق KYC بشكل مرن وتصريحي. يدعم **ستة أنواع من التكاملات** يمكن تفعيلها أو تعطيلها لكل نموذج:

| التكامل | الوصف |
| :--- | :--- |
| `INTEGRATION_BEHAVIOR` | تطبيق سلوك `KycDocumentModel` على الموديل. |
| `INTEGRATION_FORM_FIELD` | إضافة حقل `partial` مخصص في نماذج backend. |
| `INTEGRATION_LIST_COLUMN` | إضافة عمود `documents_count` في قوائم backend. |
| `INTEGRATION_LIST_FILTER` | إضافة فلتر حسب حالة الوثائق في قوائم backend. |
| `INTEGRATION_QUERY` | فلترة تلقائية في استعلامات API والتقارير. |
| `INTEGRATION_TRANSFORMER` | حقن `DynamicAddIncludeKyc` في Transformer الموديل. |

**تسجيل نموذج بسيط:**

```php
$manager = KycDocumentManager::instance();
$manager->registerModel(User::class, [
    'integrations' => [
        KycDocumentManager::INTEGRATION_QUERY => true,
    ],
]);
```

**أو تسجيل سريع بخيارات مبسطة:**

```php
$manager->quickRegister(User::class, null, [
    'with_form'   => true,
    'with_filter' => true,
]);
```

##### 3. معالج التكامل الآلي `KycIntegrationHandler`

كلاس `Nano3\Kyc\EventHandlers\KycIntegrationHandler` يقوم بربط جميع الأحداث تلقائياً:

- يستمع لـ `eloquent.booting: *` لتطبيق السلوك على النماذج المسجلة.
- في بيئة backend: يضيف الحقول، الأعمدة، والفلاتر تلقائياً.
- دائماً: يفعّل تكاملات API و Transformer.

يكفي تسجيله مرة واحدة:

```php
Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
```

##### 4. معالجة ذكية لمعاملات API

يدعم `KycDocumentManager` استخراج وتحليل قيم فلترة KYC من طلبات API بطرق متعددة:

- **Wildcards**: القيم `*`، `all`، `ALL` يتم تجاهلها (لا تطبق فلتر).
- **Value Processors**: كلمات مفتاحية مثل `verified`، `pending`، `any`، `none` تُنفذ معالجات مخصصة.
- **Multi-value Handlers**: دعم صيغ `comma`، `pipe`، `json`، `array`، و `auto` للكشف التلقائي.
- **Complex Groups**: القيم التي تحتوي على `|` (pipe) تُقسم إلى مجموعات تتعامل مع بعضها بـ `OR`، وداخل كل مجموعة يمكن استخدام `,` (comma) بمنطق `AND`.

**أمثلة على قيم `kyc_status` المدعومة:**

| القيمة | التفسير |
| :--- | :--- |
| `verified` | يطبق معالج `verified` (وثائق معتمدة). |
| `verified,pending` | وثائق معتمدة أو معلقة (AND). |
| `verified\|pending` | وثائق معتمدة أو معلقة (OR). |
| `verified,pending\|rejected` | (معتمدة ومعلقة) أو (مرفوضة). |

##### 5. دالة `createAdvancedQuery`

لبناء استعلامات جاهزة:

```php
$query = KycDocumentManager::instance()->createAdvancedQuery(
    User::class,
    ['kyc_status' => 'verified|pending'],
    [
        'with_documents'  => true,
        'documents_count' => true,
    ]
);
$users = $query->get();
```

#### التحسينات الإضافية

- **دوال مساعدة عامة**: `checkValueIsNotAll`، `scopeWhereField`، `parseComplexFilter`، `applyLogicOperator`.
- **أحداث مخصصة**: `nano3.kyc.modelRegistered`، `nano3.kyc.beforeQueryProcessing`، `nano3.kyc.afterQueryProcessing`، وغيرها.
- **توثيق شامل**: تم إنشاء ملفات توثيق كاملة بالعربية والإنجليزية لكل من `KycDocumentManager` و `KycDocumentModel` مع أمثلة عملية متقدمة.

---

### أمثلة عملية

#### 1. تسجيل موديل العميل مع كافة التكاملات

```php
KycDocumentManager::instance()->registerModel(Customer::class, [
    'controller' => CustomersController::class,
    'integrations' => [
        KycDocumentManager::INTEGRATION_BEHAVIOR    => true,
        KycDocumentManager::INTEGRATION_FORM_FIELD  => true,
        KycDocumentManager::INTEGRATION_LIST_COLUMN => true,
        KycDocumentManager::INTEGRATION_LIST_FILTER => true,
        KycDocumentManager::INTEGRATION_QUERY       => true,
        KycDocumentManager::INTEGRATION_TRANSFORMER => true,
    ],
    'form_config' => ['tab' => 'KYC'],
]);
```

#### 2. استخدام النطاقات في تقرير مخصص

```php
$expiringUsers = User::whereHasDocuments([
    'status'     => 'verified',
    'has_expiry' => true,
    'conditions' => function ($q) {
        $q->where('expiry_date', '>=', now())
          ->where('expiry_date', '<=', now()->addDays(30));
    }
])->get();
```

#### 3. فلترة API بمعاملات معقدة

```http
GET /api/v1/users?kyc_status=verified,pending|rejected
```

يقوم `KycDocumentManager` تلقائياً بتحليل القيمة وتطبيق الاستعلام المناسب دون أي كود إضافي.

---

### ملخص الإصدارات (1.2.8 – 1.2.9)

| الإصدار | أبرز الميزات والإصلاحات |
| :--- | :--- |
| 1.2.8 | إصلاح جذري لدمج الحقول وقواعد التحقق، حذف الحقول غير المرغوب فيها، دمج قواعد التحقق بذكاء، منع تسرب الحقول الافتراضية. |
| 1.2.9 | **نظام تكاملات KYC المتكامل**: سلوك `KycDocumentModel`، كلاس `KycDocumentManager`، معالج `KycIntegrationHandler`، معالجة ذكية لمعاملات API، توثيق شامل. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - إضافة الملفات الجديدة:
     - `behaviors/KycDocumentModel.php`
     - `behaviors/KycDocumentModel/KycDocumentScopesAndHelpers.php`
     - `classes/KycDocumentManager.php`
     - `eventhandlers/KycIntegrationHandler.php`
   - تسجيل `KycIntegrationHandler` في `Plugin.php`:
     ```php
     Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
     ```

2. **تسجيل النماذج**:
   - أضف دالة `registerKycDocumentIntegrations` في الـ Plugin الخاص بك لتسجيل النماذج التي تريد ربطها بوثائق KYC.

3. **اختبار التكاملات**:
   - تأكد من أن نطاقات `whereHasDocuments` تعمل على النماذج المسجلة.
   - جرب فلترة API باستخدام `kyc_status` بقيم مختلفة.
   - تحقق من ظهور الحقول والفلاتر في لوحة التحكم (إذا تم تفعيلها).

4. **مسح الكاش** (اختياري):
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

الإصدار 1.2.9 هو إصدار محوري في مسار إضافة `Nano3.Kyc`، حيث ينقلها من مجرد أداة لإدارة وثائق KYC إلى **نظام بيئي متكامل** يمكن ربطه بسهولة مع أي موديل في المنصة. بفضل `KycDocumentManager` والسلوك `KycDocumentModel`، يمكن للمطورين تفعيل تكاملات متقدمة ببضعة أسطر من الكود، مع الاستفادة من معالجة ذكية للمعلمات وتوليد تلقائي لعناصر واجهة المستخدم.

نشكركم على متابعتكم المستمرة ونتطلع إلى ملاحظاتكم لتطوير المزيد من الميزات في الإصدارات القادمة.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق كلاس `KycDocumentManager`](./docs/Kyc/Docs-KycDocumentManager-Class-ar.md)
- [توثيق سلوك `KycDocumentModel`](./docs/Kyc/Docs-KycDocumentModel-Behaviors-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./docs/Kyc/Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
