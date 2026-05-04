# توثيق كلاس `KycDocumentManager` – مدير تكاملات وثائق KYC

## نظرة عامة

`Nano3\Kyc\Classes\KycDocumentManager` هو كلاس Singleton متكامل يعمل كجسر بين وثائق اعرف عميلك (KYC) وأي موديل في برمجيات نانوسوفت (NanoSoft App). يقوم الكلاس بأتمتة عملية ربط الموديلات (مستخدمين، عملاء، شركات...) بوثائق `Document`، مضيفاً لها تلقائياً:

- **سلوك (Behavior)** `KycDocumentModel` الذي يوفّر العلاقة `documents` والنطاقات والإحصائيات.
- **حقول نموذج** (Form Fields) مخصصة في لوحة التحكم الخلفية.
- **أعمدة قوائم** (List Columns) لعرض عدد الوثائق.
- **فلاتر قوائم** (List Filters) للتصفية حسب حالة الوثائق.
- **تكامل استعلامات API والتقارير** (Query Integration) مع معالجة ذكية للمعلمات (comma, pipe, JSON, auto-detect).
- **تكامل المحولات (Transformer)** لإضافة includes خاصة بحالة KYC إلى أي Transformer.

تم بناء الكلاس على غرار `Nano\Tags\Classes\TagManager` لتحقيق أعلى درجات المرونة والتوافق، مع تخصيص كامل لمتطلبات وثائق الهوية. يسمح الكلاس بتسجيل النماذج بسهولة عن طريق دالة `registerKycDocumentIntegrations` في Plugin، ثم يتولى الباقي تلقائياً.

---

## أنواع التكاملات

يدعم الكلاس خمسة أنواع من التكاملات يمكن تفعيلها أو تعطيلها لكل نموذج مسجّل على حدة:

| الثابت | النوع | الوصف |
|--------|-------|-------|
| `INTEGRATION_BEHAVIOR` | `behavior` | إضافة سلوك `KycDocumentModel` (العلاقة `documents`، النطاقات، الإحصائيات). |
| `INTEGRATION_FORM_FIELD` | `form_field` | إضافة حقل `partial` مخصص في نماذج backend. |
| `INTEGRATION_LIST_COLUMN` | `list_column` | إضافة عمود `documents_count` في قوائم backend. |
| `INTEGRATION_LIST_FILTER` | `list_filter` | إضافة نطاق تصفية حسب حالة الوثائق في قوائم backend. |
| `INTEGRATION_QUERY` | `query` | فلترة تلقائية في استعلامات API والتقارير مع معالجة ذكية للمعلمات. |
| `INTEGRATION_TRANSFORMER` | `transformer` | حقن سلوك `DynamicAddIncludeKyc` في Transformer الموديل. |

---

## تسجيل النماذج

### `registerModel(string $modelClass, array $config = []): self`

تسجيل موديل جديد لإدارة وثائق KYC مع إمكانية تخصيص الإعدادات. في حالة عدم تحديد إعدادات، تُستخدم القيم الافتراضية الشاملة الموضحة لاحقاً.

**المعاملات**:
- `$modelClass`: اسم الكلاس الكامل للموديل (مثال: `RainLab\User\Models\User`).
- `$config`: مصفوفة إعدادات اختيارية.

**مثال**:
```php
$manager = KycDocumentManager::instance();
$manager->registerModel(\RainLab\User\Models\User::class, [
    'enabled' => true,
    'integrations' => [
        KycDocumentManager::INTEGRATION_QUERY => true,
    ],
]);
```

### `quickRegister(string $modelClass, ?string $controllerClass = null, array $options = []): self`

تسجيل سريع مع خيارات مبسطة.

```php
$manager->quickRegister(User::class, null, [
    'with_form'   => true,
    'with_column' => true,
    'with_filter' => true,
    'tab'         => 'KYC',
]);
```

### التسجيل الآلي من الإضافات

يمكن لأي Plugin أن يعرّف دوال `registerKycDocumentIntegrations` التي ترجع مصفوفة من النماذج وإعداداتها. سيقوم `KycDocumentManager` بتحميلها تلقائياً عند استدعاء `getRegisteredModels()`.

```php
class Plugin extends PluginBase
{
    public function registerKycDocumentIntegrations()
    {
        return [
            \RainLab\User\Models\User::class => [
                'integrations' => [
                    KycDocumentManager::INTEGRATION_QUERY => true,
                ],
            ],
            \Backend\Models\User::class => [
                'integrations' => [
                    KycDocumentManager::INTEGRATION_FORM_FIELD => true,
                ],
            ],
        ];
    }
}
```

---

## الإعدادات الافتراضية الكاملة

عند تسجيل نموذج بدون إعدادات مخصصة، يعتمد الكلاس على الإعدادات الافتراضية التالية (يمكن تخصيص أي جزء منها):

### `form_config` – إعدادات حقل النموذج

| المفتاح | الافتراضي | الوصف |
|---------|-----------|-------|
| `field_name` | `'documents'` | اسم الحقل المضاف. |
| `label` | `'nano3.kyc::lang.documents.form.documents_label'` | تسمية الحقل. |
| `comment` | `'nano3.kyc::lang.documents.form.documents_comment'` | تعليق توضيحي. |
| `tab` | `'nano3.kyc::lang.plugin.tab'` | علامة التبويب التي سيظهر فيها الحقل. |
| `type` | `'partial'` | نوع الحقل (partial لقالب مخصص). |
| `path` | `'~/plugins/nano3/kyc/partials/_documents_relation.htm'` | مسار القالب الجزئي. |
| `span` | `'storm'` | عرض الحقل. |
| `cssClass` | `'col-sm-12'` | كلاس CSS للحقل. |
| `conditions` | `null` | شروط لإظهار الحقل (مصفوفة أو callable). |
| `trigger` | `[]` | إعدادات trigger (action, field, condition). |
| `depends_on` | `[]` | مصفوفة dependsOn. |

### `list_config` – إعدادات عمود القائمة

| المفتاح | الافتراضي | الوصف |
|---------|-----------|-------|
| `column_name` | `'documents_count'` | اسم العمود. |
| `label` | `'nano3.kyc::lang.documents.list.documents_count'` | تسمية العمود. |
| `type` | `'text'` | نوع العمود. |
| `searchable` | `false` | إمكانية البحث. |
| `sortable` | `false` | إمكانية الترتيب. |

### `filter_config` – إعدادات فلتر القائمة

| المفتاح | الافتراضي | الوصف |
|---------|-----------|-------|
| `scope_name` | `'kyc_documents_status'` | اسم نطاق الفلتر. |
| `label` | `'nano3.kyc::lang.documents.filter.status_label'` | تسمية الفلتر. |
| `type` | `'group'` | نوع الفلتر. |
| `options` | `[verified, pending, rejected, expired]` | خيارات الفلتر (مصفوفة). |
| `conditions` | `null` | شروط لإظهار الفلتر. |
| `depends_on` | `[]` | تبعية الفلتر. |

### `query_config` – إعدادات استعلام API والتقارير

| المفتاح | الافتراضي | الوصف |
|---------|-----------|-------|
| `param_keys` | `['kyc_status', 'kycStatus', 'document_status', 'documentStatus']` | مفاتيح الباراميتر التي يتم البحث عنها في `$params`. |
| `wildcard_values` | `['*', 'all', 'ALL']` | القيم التي تعني "تخطي الفلتر". |
| `scope_method` | `'whereHasDocuments'` | اسم النطاق الذي سيتم استدعاؤه على الاستعلام. |
| `logic_operator` | `'AND'` | العامل المنطقي الافتراضي. |
| `multiple_value_handler` | `['comma', 'array']` | معالجات القيم المتعددة (مصفوفة أو `'auto'`). |
| `value_processors` | `[verified, pending, rejected, expired, any, none]` | معالجات مخصصة للكلمات المفتاحية (انظر الشرح أدناه). |
| `advanced_features` | `[debug_mode => false...]` | خيارات متقدمة (تفعيل debug، تعطيل معالجات معينة...). |

#### معالجات القيم (value_processors)

| المفتاح | السلوك |
|---------|--------|
| `verified` | يبحث عن وثائق معتمدة (`hasVerifiedDocuments`) |
| `pending` | يبحث عن وثائق بحالة `pending` |
| `rejected` | وثائق مرفوضة |
| `expired` | وثائق منتهية الصلاحية |
| `any` | أي وثيقة موجودة (`has('documents')`) |
| `none` | لا توجد وثائق (`doesntHave('documents')`) |

### `transformer_config` – إعدادات Transformer

| المفتاح | الافتراضي | الوصف |
|---------|-----------|-------|
| `include_relation` | `'kyc_status'` | اسم include المضاف. |
| `include_method` | `'includeKycStatus'` | اسم الدالة في الـ Transformer. |
| `default_include` | `false` | هل يتم تضمينه تلقائياً في الـ Response؟ |
| `eager_load` | `false` | هل يتم تحميل العلاقة `documents` تلقائياً في الـ Transformer؟ |
| `additional_fields` | `[documents_count => callback]` | حقول إضافية تولد ديناميكياً (مثل عدد الوثائق). |

---

## تطبيق التكاملات آلياً

### `applyBehaviorIntegration($modelClass)`

يضيف سلوك `KycDocumentModel` للموديل تلقائياً عند حدث `eloquent.booting: *`. هذا السلوك يعرّف:

- علاقة `documents` (morphMany) إلى موديل `Document` عبر حقل `owner`.
- نطاقات قوية مثل `scopeWhereHasDocuments` و `scopeHasVerifiedDocuments` و `scopeAddDocumentsCount`.
- دوال إحصائية مثل `getDocumentsCountAttribute` و `hasVerifiedDocument()`.

### `applyFormFieldIntegration($modelClass)`

يستمع للحدث `backend.form.extendFieldsBefore` ويضيف حقل `partial` إلى علامة التبويب المحددة، مع دعم `conditions` و `trigger` و `depends_on`.

### `applyListColumnIntegration($modelClass)`

يضيف عمود `documents_count` إلى قوائم backend.

### `applyListFilterIntegration($modelClass)`

يضيف فلتر `kyc_documents_status` بخيارات الحالات إلى قوائم backend، مع دعم `conditions` و `depends_on`.

### `applyQueryIntegration($modelClass)`

أهم تكامل، حيث يستمع للأحداث `api.list.extendQuery` و `reports.list.extendQuery` وحدث `EVENT_EXTEND_QUERY`، ويقوم بفلترة الاستعلامات تلقائياً بناءً على المعاملات الواردة. يستخدم منطقاً متقدماً لمعالجة القيم كما سيأتي.

### `applyTransformerIntegration($modelClass)`

يحقن سلوك `DynamicAddIncludeKyc` في Transformer الموديل، مما يضيف includes مثل `kyc_status` و `kyc_documents` و `kyc_verification_summary` وغيرها.

---

## تكامل الاستعلام المتقدم (Query Integration)

يتولى الكلاس استخراج قيمة الفلترة من `$params` باستخدام `extractDocumentParams`، ثم يطبق أحد المسارات التالية:

1. **معالج قيم (Processor)** – إذا كانت القيمة تطابق أحد `value_processors` (مثل `verified`, `any`, `none`).
2. **مجموعات معقدة (Complex)** – إذا كانت القيمة تحتوي على `|` (pipe)، يتم تقسيمها إلى مجموعات تتعامل مع بعضها بـ OR.
3. **قيم بسيطة (Simple Values)** – تعالج باستخدام `processMultipleValues` مع دعم comma, pipe, JSON, array, auto-detect.

### `extractDocumentParams(array $params, array $config): ?array`

- يبحث عن أول مفتاح من `param_keys` موجود في `$params`.
- إذا كانت القيمة wildcard (`*`, `all`, `ALL`) أو فارغة، يرجع `null` (لا يتم تطبيق الفلتر).
- يتحقق من المعالجات (processors).
- إذا كانت القيمة تحتوي على `|` و `complex_handling_enabled`، يقسمها إلى مجموعات.
- وإلا، يعالج القيمة باستخدام `processMultipleValues`.

### `processMultipleValues($value, $handler)`

يدعم المعالجات:

- `'comma'`: `"verified,pending"` ← `['_values' => ['verified','pending'], '_logic' => 'AND']`
- `'pipe'`: `"verified|pending"` ← `['_values' => ['verified','pending'], '_logic' => 'OR']`
- `'json'`: `'["verified","pending"]'` ← نفس تأثير comma
- `'array'`: قيمة واحدة.
- `'auto'`: يكشف تلقائياً حسب الصيغة.

يمكن تمرير مصفوفة معالجات (مثل `['comma', 'array']`) ليجرب كل واحد على التوالي ويدمج النتائج.

### `applyAdvancedQueryLogic($query, $extracted, array $qc, string $modelClass)`

الدالة المركزية التي تُطبّق المنطق المستخرج على الاستعلام:

1. إذا كان `_processor`، يُنفذ المعالج.
2. إذا كان `_complex`، يُطبق `applyComplexQuery` (مجموعات مرتبطة بـ OR).
3. إذا كان `_values`، يُطبق `applyQueryGroup` مع مراعاة العامل المنطقي (`AND`/`OR`).

### `applyComplexQuery($query, $complexData, $scopeMethod, $advanced)`

تطبق مجموعات الفلتر مع بعضها باستخدام `OR`. مثال:

```
kyc_status = verified,pending|rejected
```
**التفسير**: المستخدمون الذين لديهم (verified أو pending) أو (rejected).

**التدفق**:
```php
$query->where(function ($q) {
    $q->whereHasDocuments(['status' => ['verified','pending']]);  // المجموعة الأولى
    $q->orWhere(function ($orQ) {
        $orQ->whereHasDocuments(['status' => ['rejected']]);      // المجموعة الثانية
    });
});
```

### `applyQueryGroup($query, $group, $scopeMethod)`

تطبق مجموعة واحدة مع دعم منطق `OR` داخلي. إذا كانت المجموعة تحتوي على عدة قيم ومحدد منطق `OR`، سيتم توليد استعلام `orWhere` لكل قيمة.

---

## تكامل Transformer

يضيف سلوك `DynamicAddIncludeKyc` الذي يسجل includes مثل `kyc_status`, `kyc_documents`, `kyc_verification_summary`, وغيرها.

يدعم:

- `default_include`: إدراج `kyc_status` تلقائياً في كل Response.
- `eager_load`: تحميل علاقة `documents` مسبقاً.
- `additional_fields`: إضافة حقول ديناميكية مثل `documents_count` تظهر في Response الـ API.

---

## معالج التكامل الآلي `KycIntegrationHandler`

`Nano3\Kyc\EventHandlers\KycIntegrationHandler` هو الكلاس المسؤول عن ربط الأحداث وتفعيل جميع التكاملات تلقائياً عند بدء التشغيل. يكفي تسجيله مرة واحدة في `Plugin.php`:

```php
Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
```

سيقوم المعالج بـ:

- تحميل جميع النماذج المسجلة.
- الاستماع لحدث `eloquent.booting: *` لتطبيق السلوك.
- في backend: تطبيق تكاملات النموذج والقائمة والفلتر.
- دائماً: تطبيق تكاملات الاستعلام والمحولات.

---

## دوال مساعدة عامة

| الدالة | الوصف |
|--------|-------|
| `checkValueIsNotAll($value)` | التحقق من أن القيمة ليست wildcard (`*`, `all`). |
| `scopeWhereField($query, $field, $value, …)` | نطاق مرن لتطبيق شروط `WHERE` مع دعم `is_or`, `is_not`, `is_force`. |
| `parseComplexFilter($str, $sep)` | تحليل سلسلة معقدة إلى مجموعات `_values`. |
| `applyLogicOperator($query, $conditions, $operator, $callback)` | تطبيق شروط متعددة مع عامل منطقي (`AND`/`OR`). |
| `isJson($str)` | فحص صحة JSON. |
| `checkConditions($model, $conditions)` | تقييم شروط إظهار حقول/فلاتر (callable أو مصفوفة). |
| `handleDebug($msg)` | تسجيل أخطاء فقط في وضع التطوير. |

---

## أمثلة عملية شاملة

### 1. تسجيل موديل المستخدمين وتفعيل استعلامات API فقط

```php
// في Plugin.php
public function registerKycDocumentIntegrations()
{
    return [
        \RainLab\User\Models\User::class => [
            'integrations' => [
                KycDocumentManager::INTEGRATION_QUERY => true,
            ],
        ],
    ];
}
```

الآن عند استدعاء `GET /api/users?kyc_status=verified` يتم فلترة المستخدمين الذين لديهم وثائق معتمدة تلقائياً.

### 2. تسجيل مع تكامل كامل للوحة التحكم

```php
$manager->registerModel(\Backend\Models\User::class, [
    'controller' => \Backend\Controllers\Users::class,
    'integrations' => [
        KycDocumentManager::INTEGRATION_BEHAVIOR    => true,
        KycDocumentManager::INTEGRATION_FORM_FIELD  => true,
        KycDocumentManager::INTEGRATION_LIST_COLUMN => true,
        KycDocumentManager::INTEGRATION_LIST_FILTER => true,
        KycDocumentManager::INTEGRATION_QUERY       => true,
    ],
    'form_config' => ['tab' => 'KYC Documents'],
]);
```

### 3. استخدام `createAdvancedQuery` في تقرير مخصص

```php
$query = KycDocumentManager::instance()->createAdvancedQuery(
    User::class,
    ['kyc_status' => 'verified|pending'],
    [
        'with_documents'    => true,
        'documents_count'   => true,
        'documents_sort'    => ['column' => 'created_at', 'direction' => 'desc'],
        'conditions'        => function ($q) {
            $q->where('is_active', true);
        }
    ]
);

$users = $query->get();

foreach ($users as $user) {
    echo $user->name . ' has ' . $user->documents_count . ' documents.';
    foreach ($user->documents as $doc) {
        echo ' - ' . $doc->document_type . ' (' . $doc->status . ')';
    }
}
```

### 4. تفعيل Transformer مع default_include

```php
$manager->registerModel(User::class, [
    'transformer' => \Nano\AuthApi\Transformers\UserTransformer::class,
    'transformer_config' => [
        'default_include' => true,  // kyc_status يظهر تلقائياً
        'eager_load'      => true,   // تحميل documents مسبقاً
        'additional_fields' => [
            'verified_documents_count' => function ($model) {
                return $model->documents()->where('is_verified', true)->count();
            }
        ]
    ],
]);
```

ثم استدعاء `GET /api/users/1` سيعيد تلقائياً `kyc_status` و `verified_documents_count`.

### 5. استخدام النطاقات مباشرة في الكود

بعد إضافة السلوك، يمكن استخدام النطاقات مباشرة:

```php
// المستخدمون الذين لديهم وثائق هوية أساسية معتمدة
User::whereHasDocuments([
    'category' => 'primary_id',
    'is_verified' => true
])->get();

// فرز المستخدمين حسب عدد وثائقهم
User::addDocumentsCount('total_docs')
    ->sortByDocumentsCount('DESC')
    ->get();

// فحص مستخدم معين
$user = User::find(1);
if ($user->hasVerifiedDocument()) {
    echo "لديه وثائق معتمدة: " . $user->verified_documents_count;
}
```

---

## الأحداث المخصصة

| الحدث | المعاملات | الوصف |
|-------|-----------|-------|
| `nano3.kyc.modelRegistered` | `$modelClass, $config` | عند تسجيل نموذج جديد. |
| `nano3.kyc.beforeApplyIntegrations` | `$modelClass, $config` | قبل تطبيق التكاملات. |
| `nano3.kyc.afterApplyIntegrations` | `$modelClass, $config` | بعد تطبيق التكاملات. |
| `nano3.kyc.beforeQueryProcessing` | `$rawValue, $params, $config` | قبل معالجة قيمة الفلتر. |
| `nano3.kyc.afterQueryProcessing` | `$result, $params, $config` | بعد معالجة قيمة الفلتر. |
| `nano3.kyc.extendQuery` | `$query, $params, $context` | لتوسيع الاستعلامات من الخارج. |

---

## إدارة النماذج المسجلة

| الدالة | الوصف |
|--------|-------|
| `getRegisteredModels()` | تحميل جميع النماذج المسجلة (من الذاكرة أو الإضافات). |
| `isModelRegistered($class)` | هل النموذج مسجل بالفعل؟ |
| `getModelConfig($class)` | جلب إعدادات نموذج. |
| `getRegisteredModelClasses()` | قائمة بأسماء كلاسات النماذج المسجلة. |
| `hasRegisteredModels()` | هل هناك أي نموذج مسجل؟ |
| `getModelsWithIntegration($type)` | النماذج التي تم تفعيل تكامل معين لها. |
| `clearRegisteredModels()` | مسح جميع التسجيلات (للاستخدام في الاختبارات). |

---

## ملاحظات هامة

- **الكاش**: الكلاس لا يستخدم تخزيناً مؤقتاً خاصاً به، لكنه يستمع لأحداث الموديل (`eloquent.booting: *`) لتطبيق السلوك. عند تغيير الإعدادات في وقت التشغيل، قد تحتاج إلى إعادة تحميل الصفحة أو مسح الكاش العام.
- **الاعتماد على `DocumentType`**: الإعدادات الافتراضية لخيارات الفلتر (statuses) مبنية على القيم المدعومة من `DocumentType`. أي توسيع في أنواع الحالات يجب أن ينعكس هنا.
- **الأمان**: الكلاس لا يقوم بالتحقق من صلاحيات المستخدم. يتم افتراض أن المتحكم (Controller) يقوم بذلك قبل استدعاء الدوال.

---

## الخاتمة

`KycDocumentManager` هو الحل الشامل والمرن لربط وثائق KYC بأي موديل في نظامك. بفضل تصميمه المستوحى من `TagManager`، يوفر تجربة موحدة وقوية تتيح لك تفعيل تكاملات متعددة ببضعة أسطر من الكود. سواء كنت تبني API، لوحة تحكم، أو تقارير، ستجد في هذا الكلاس الأدوات اللازمة لإدارة حالة KYC بكل سهولة واحترافية.

## التوثيق الإضافي

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./Docs-Document-Model-ar.md)
- [توثيق سلوك `KycDocumentModel`](./Docs-KycDocumentModel-Behaviors-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
