# توثيق كلاس `KycDocumentManager` – مدير تكاملات وثائق KYC

## نظرة عامة

`Nano3\Kyc\Classes\KycDocumentManager` هو كلاس Singleton متكامل يعمل كجسر بين وثائق اعرف عميلك (KYC) وأي موديل في برمجيات نانوسوفت (NanoSoft App). يقوم الكلاس بأتمتة عملية ربط الموديلات (مستخدمين، عملاء، شركات...) بوثائق `Document`، مضيفاً لها تلقائياً:

- **سلوك (Behavior)** `KycDocumentModel` الذي يوفّر العلاقة `documents` والنطاقات المتقدمة والإحصائيات.
- **حقول نموذج** (Form Fields) مخصصة في لوحة التحكم الخلفية.
- **أعمدة قوائم** (List Columns) متعددة لعرض إحصائيات الوثائق (عدد الوثائق، المعتمدة، غير المعتمدة، المنتهية، مؤشرات الفئات).
- **فلاتر قوائم** (List Filters) متطورة تشمل فلاتر جماعية (group) مع تبعيات (dependsOn)، وفلاتر تبديل (switch/checkbox) لوجود/عدم وجود وثائق، وفلاتر مدمجة (AND).
- **تكامل استعلامات API والتقارير** (Query Integration) مع معالجة ذكية للمعلمات (comma, pipe, JSON, auto-detect) ومجموعات معقدة.
- **تكامل المحولات (Transformer)** لإضافة includes خاصة بحالة KYC إلى أي Transformer، ودعم التضمين التلقائي والتحميل المبكر.

تم بناء الكلاس على غرار `Nano\Tags\Classes\TagManager` لتحقيق أعلى درجات المرونة والتوافق، مع تخصيص كامل لمتطلبات وثائق الهوية. يسمح الكلاس بتسجيل النماذج بسهولة عن طريق دالة `registerKycDocumentIntegrations` في Plugin، أو باستخدام `registerModel` و `quickRegister`، ثم يتولى الباقي تلقائياً.

---

## أنواع التكاملات

يدعم الكلاس ستة أنواع من التكاملات يمكن تفعيلها أو تعطيلها لكل نموذج مسجّل على حدة:

| الثابت | النوع | الوصف |
|--------|-------|-------|
| `INTEGRATION_BEHAVIOR` | `behavior` | إضافة سلوك `KycDocumentModel` (العلاقة `documents`، النطاقات، الإحصائيات). |
| `INTEGRATION_FORM_FIELD` | `form_field` | إضافة حقل `partial` مخصص في نماذج backend. |
| `INTEGRATION_LIST_COLUMN` | `list_column` | إضافة عدة أعمدة قابلة للتخصيص في قوائم backend. |
| `INTEGRATION_LIST_FILTER` | `list_filter` | إضافة عدة فلاتر متطورة (group, checkbox, switch) مع تبعيات. |
| `INTEGRATION_QUERY` | `query` | فلترة تلقائية في استعلامات API والتقارير مع معالجة ذكية للمعلمات. |
| `INTEGRATION_TRANSFORMER` | `transformer` | حقن سلوك `DynamicAddIncludeKyc` في Transformer الموديل، مع دعم التضمين الافتراضي والتحميل المسبق. |

---

## تسجيل النماذج

### `registerModel(string $modelClass, array $config = []): self`

تسجيل موديل جديد لإدارة وثائق KYC مع إمكانية تخصيص الإعدادات. تقوم الدالة أولاً بتحميل الإعدادات الافتراضية من ملف `config/nano3.kyc/extend.php` إن وجد، ثم تدمجها مع أي إعدادات مخصصة تمرر في `$config`، وتستعمل قيماً صلبة كحل أخير.

**المعاملات**:
- `$modelClass`: اسم الكلاس الكامل للموديل (مثال: `RainLab\User\Models\User`).
- `$config`: مصفوفة إعدادات اختيارية تتجاوز الإعدادات الافتراضية.

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

يمكن لأي Plugin أن يعرّف دالة `registerKycDocumentIntegrations` التي ترجع مصفوفة من النماذج وإعداداتها. سيقوم `KycDocumentManager` بتحميلها تلقائياً عند استدعاء `getRegisteredModels()`.

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

## الإعدادات الافتراضية

### أولاً: ملف الإعدادات المركزي `config/extend.php`

يمكن تخصيص جميع إعدادات التكاملات عبر ملف `plugins/nano3/kyc/config/extend.php`. يحتوي الملف على أربعة أقسام:

- `form_config` – إعدادات حقل النموذج الافتراضي.
- `list_config` – مصفوفة من تعريفات الأعمدة.
- `filter_config` – مصفوفة من تعريفات الفلاتر.
- `query_config` – إعدادات استعلام API والتقارير.

إذا كان الملف موجوداً ويحتوي على قيم، يتم تحميلها تلقائياً عند تسجيل النماذج، ويمكن تجاوزها عبر تمرير قيم مخصصة في `$config`. إذا كان الملف فارغاً، يعود الكلاس إلى قيم صلبة افتراضية.

#### هيكل ملف `extend.php`

```php
return [
    'form_config' => [
        'field_name' => 'documents',
        'label'      => 'nano3.kyc::lang.extend.form.documents_label',
        // ... باقي الإعدادات
    ],
    'list_config' => [
        [
            'column_name' => 'documents_count',
            'label'       => 'nano3.kyc::lang.extend.list.documents_count',
            'type'        => 'number',
            'sortable'    => true,
            'width'       => '10%',
            'align'       => 'center',
        ],
        // ... أعمدة إضافية
    ],
    'filter_config' => [
        [
            'scope_name' => 'kyc_documents_category',
            'label'      => 'nano3.kyc::lang.extend.filter.category_label',
            'type'       => 'group',
            'modelClass' => 'Nano3\Kyc\Models\Document',
            'options'    => 'getDocumentCategoryFilterOptions',
            'dependsOn'  => [],
        ],
        // ... فلاتر إضافية
    ],
    'query_config' => [
        'param_keys' => ['kyc_status', 'kycStatus', 'document_status'],
        // ... باقي الإعدادات
    ],
];
```

#### تفصيل إعدادات `list_config` (الأعمدة)

يمكن تعريف عدة أعمدة بمصفوفة، وكل عمود يدعم جميع خيارات OctoberCMS القياسية:

| الخيار | الوصف |
|--------|-------|
| `column_name` | اسم العمود (مطلوب). |
| `label` | التسمية المعروضة. |
| `type` | نوع العمود (text, number, switch, ...). |
| `searchable` | إمكانية البحث (true/false). |
| `sortable` | إمكانية الترتيب (true/false). |
| `invisible` | إخفاء العمود افتراضياً. |
| `width` | عرض العمود (مثلاً "10%"). |
| `align` | محاذاة المحتوى (left, right, center). |
| `clickable` | تفعيل النقر على العمود. |
| `select` | استعلام SQL مخصص للقيمة. |
| `valueFrom` / `displayFrom` | تخصيص مصدر القيمة. |
| `relation` / `relationCount` | عرض عدد علاقة. |
| `cssClass` / `headCssClass` | كلاسات CSS. |
| `permissions` | صلاحيات مطلوبة لرؤية العمود. |

**الأعمدة الافتراضية**:
- `documents_count` – إجمالي عدد الوثائق.
- `verified_documents_count` – عدد الوثائق المعتمدة.
- `not_verified_documents_count` – عدد الوثائق غير المعتمدة.
- `expired_verified_documents_count` – عدد الوثائق المنتهية (المعتمدة).
- `is_verifier_primary_id`، `is_verifier_secondary_id`، ... – مؤشرات (switch) لوجود وثائق من كل فئة.

#### تفصيل إعدادات `filter_config` (الفلاتر)

يمكن تعريف عدة فلاتر بمصفوفة، وكل فلتر يدعم جميع خيارات OctoberCMS القياسية:

| الخيار | الوصف |
|--------|-------|
| `scope_name` | اسم نطاق الفلتر (مطلوب). |
| `label` | التسمية المعروضة. |
| `type` | نوع الفلتر (group, checkbox, switch، ...). |
| `scope` | اسم نطاق الاستعلام في الموديل. |
| `options` | خيارات الفلتر (مصفوفة ثابتة أو اسم دالة في الموديل). |
| `modelClass` | كلاس الموديل المستخدم لجلب الخيارات الديناميكية (اختياري). |
| `default` | القيمة الافتراضية للفلتر. |
| `dependsOn` | مصفوفة من أسماء فلاتر أخرى يعتمد عليها هذا الفلتر. |
| `conditions` | شروط إظهار الفلتر (مصفوفة أو callable). |
| `emptyOption` | تسمية الخيار الفارغ. |
| `permissions` | صلاحيات مطلوبة لرؤية الفلتر. |

**الفلاتر الافتراضية**:
1. **فلتر تصنيف الوثيقة** (`kyc_documents_category`) – group، خياراته من `DocumentType::getCategories()`.
2. **فلتر نوع الوثيقة** (`kyc_documents_type`) – group، يعتمد على الفئة المختارة، خياراته من `getDocumentTypeFilterOptions`.
3. **فلتر حالة الوثيقة** (`kyc_documents_status`) – group، خيارات ثابتة (معتمدة، معلقة، مرفوضة، منتهية).
4. **فلتر تبديل "أي وثائق"** (`is_toggel_any_kyc_documents`) – switch.
5. **فلتر "وثائق معتمدة"** (`is_toggel_kyc_documents_verified`) – checkbox.
6. **فلتر "وثائق منتهية"** (`is_toggel_kyc_documents_expired`) – checkbox.
7. **فلتر "هوية أساسية معتمدة"** (`is_toggel_kyc_documents_verified_and_category_primary_id`) – checkbox.

#### تفصيل إعدادات `query_config` (استعلامات API)

| المفتاح | الافتراضي | الوصف |
|---------|-----------|-------|
| `param_keys` | `['kyc_status', 'kycStatus', 'document_status', 'documentStatus']` | مفاتيح الباراميتر المستخدمة في الطلبات. |
| `wildcard_values` | `['*', 'all', 'ALL']` | القيم التي تعني "تخطي الفلتر". |
| `scope_method` | `'whereHasDocuments'` | اسم النطاق الذي سيتم استدعاؤه على الاستعلام. |
| `logic_operator` | `'AND'` | العامل المنطقي الافتراضي. |
| `multiple_value_handler` | `['comma', 'array']` | معالجات القيم المتعددة (مصفوفة أو `'auto'`). |
| `value_processors` | `[verified, pending, rejected, expired, any, none]` | معالجات مخصصة للكلمات المفتاحية. |
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

### ثانياً: إعدادات `transformer_config`

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
- نطاقات قوية مثل `scopeWhereHasDocuments`، `scopeHasVerifiedDocuments`، `scopeAddDocumentsCount`، ونطاقات Toggle (تبديل) مثل `scopeIsToggelAnyKycDocuments` و `scopeIsToggelKycDocumentsVerified` وغيرها.
- دوال إحصائية مثل `getDocumentsCountAttribute` و `hasVerifiedDocument()`.

### `applyFormFieldIntegration($modelClass)`

يستمع للحدث `backend.form.extendFieldsBefore` ويضيف حقل `partial` إلى علامة التبويب المحددة، مع دعم `conditions` و `trigger` و `dependsOn`.

### `applyListColumnIntegration($modelClass)`

يضيف عدة أعمدة (حسب تعريفات `list_config`) إلى قوائم backend. يستخدم دالة `normalizeColumnConfig` لتطبيع الخيارات والتأكد من توافقها مع معايير OctoberCMS.

### `applyListFilterIntegration($modelClass)`

يضيف عدة فلاتر (حسب تعريفات `filter_config`) إلى قوائم backend، مع دعم `conditions` و `dependsOn` وخيارات ديناميكية. يستخدم دالة `normalizeFilterConfig` لتطبيع الخيارات.

### `applyQueryIntegration($modelClass)`

أهم تكامل، حيث يستمع للأحداث `api.list.extendQuery` و `reports.list.extendQuery` وحدث `EVENT_EXTEND_QUERY`، ويقوم بفلترة الاستعلامات تلقائياً بناءً على المعاملات الواردة. يستخدم منطقاً متقدماً لمعالجة القيم.

### `applyTransformerIntegration($modelClass)`

يحقن سلوك `DynamicAddIncludeKyc` في Transformer الموديل، ويضيف دعم `default_include` (التضمين التلقائي) و `eager_load` (التحميل المسبق) و `additional_fields`.

---

## تكامل الاستعلام المتقدم (Query Integration)

### استخراج وتحليل المعاملات

- يبحث عن أول مفتاح من `param_keys` موجود في `$params`.
- إذا كانت القيمة wildcard، يرجع `null` (لا فلتر).
- يتحقق من معالجات القيم (value_processors).
- إذا كانت القيمة تحتوي على `|`، يقسمها إلى مجموعات (complex).
- وإلا، يعالج باستخدام `processMultipleValues` التي تدعم comma, pipe, JSON, array, auto-detect.

### المعالجات المنطقية

- `applyAdvancedQueryLogic`: تطبق processor أو complex أو simple values.
- `applyComplexQuery`: مجموعات مرتبطة بـ OR.
- `applyQueryGroup`: مجموعة واحدة مع دعم OR/AND داخلي.

### دالة `createAdvancedQuery`

لبناء استعلامات KYC جاهزة مع دعم eager load، عد الوثائق، وترتيبها.

---

## معالج التكامل الآلي `KycIntegrationHandler`

يقوم بربط الأحداث تلقائياً: تطبيق السلوك عند `eloquent.booting: *`، وإضافة الحقول/الأعمدة/الفلاتر في backend، وتفعيل تكاملات API و Transformer. يكفي تسجيله في `Plugin.php`:

```php
Event::subscribe(\Nano3\Kyc\EventHandlers\KycIntegrationHandler::class);
```

---

## دوال مساعدة عامة

| الدالة | الوصف |
|--------|-------|
| `normalizeColumnConfig(array $config)` | تطبيع إعدادات العمود لتتوافق مع OctoberCMS. |
| `normalizeFilterConfig(array $config, $model)` | تطبيع إعدادات الفلتر لتتوافق مع OctoberCMS. |
| `checkConditions($model, $conditions)` | تقييم شروط إظهار حقل/فلتر. |
| `checkValueIsNotAll($value)` | التحقق من أن القيمة ليست wildcard. |
| `scopeWhereField(...)` | نطاق مرن لشروط WHERE. |
| `parseComplexFilter($str, $sep)` | تحليل سلسلة معقدة إلى مجموعات. |
| `applyLogicOperator(...)` | تطبيق شروط متعددة مع AND/OR. |

---

## أمثلة عملية شاملة (مُحدّثة)

### 1. تفعيل التكامل الكامل للمستخدمين عبر ملف الإعدادات

بمجرد وجود ملف `extend.php` بالإعدادات الافتراضية، وتسجيل النماذج في `registerKycDocumentIntegrations`:

```php
// Plugin.php
public function registerKycDocumentIntegrations()
{
    return [
        \RainLab\User\Models\User::class => [
            'controller'   => \RainLab\User\Controllers\Users::class,
            'transformer'  => \Nano\AuthApi\Transformers\UserTransformer::class,
            'integrations' => [
                KycDocumentManager::INTEGRATION_BEHAVIOR    => true,
                KycDocumentManager::INTEGRATION_LIST_COLUMN => true,
                KycDocumentManager::INTEGRATION_LIST_FILTER => true,
            ],
        ],
    ];
}
```

ستظهر الأعمدة والفلاتر الافتراضية تلقائياً في قائمة المستخدمين.

### 2. استخدام الفلاتر المترابطة (Category => Type)

تم تكوين فلتر `kyc_documents_type` ليعتمد على `kyc_documents_category`. عند تحديد الفئة، تتغير خيارات النوع ديناميكياً.

### 3. استخدام نطاقات Toggle الجديدة

```php
// المستخدمون الذين لديهم هوية أساسية معتمدة (شرط مزدوج)
User::where(function($q) {
    $q->hasVerifiedDocuments()
      ->whereHasDocuments(['category' => 'primary_id']);
})->get();

// أو باستخدام النطاق المختصر
User::isToggelKycDocumentsVerifiedAndCategory(1, 'primary_id')->get();
```

### 4. تخصيص الأعمدة والفلاتر لمشروع معين

انسخ ملف `extend.php` إلى مشروعك عبر `php artisan config:publish nano3.kyc`، ثم عدل الأعمدة والفلاتر حسب الحاجة.

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
| `getQueryConfig($class)` | جلب إعدادات استعلام API لنموذج. |
| `getTransformerConfig($class)` | جلب إعدادات Transformer لنموذج. |
| `getRegisteredModelClasses()` | قائمة بأسماء كلاسات النماذج المسجلة. |
| `hasRegisteredModels()` | هل هناك أي نموذج مسجل؟ |
| `getModelsWithIntegration($type)` | النماذج التي تم تفعيل تكامل معين لها. |
| `clearRegisteredModels()` | مسح جميع التسجيلات (للاستخدام في الاختبارات). |

---

## ملاحظات هامة

- **الإعدادات المركزية**: يفضل تخصيص الأعمدة والفلاتر عبر ملف `config/extend.php` بدلاً من تعديل الكود، لتسهيل الصيانة والترقية.
- **الأداء**: الأعمدة الإحصائية (مثل `verified_documents_count`) تحتاج إلى تطبيق النطاقات المناسبة على استعلام القائمة. يقوم `KycDocumentManager` بذلك تلقائياً عند تفعيل تكامل `list_column`.
- **الأمان**: الكلاس لا يقوم بالتحقق من صلاحيات المستخدم. يُفترض أن يتم ذلك في طبقة المتحكم (Controller).

---

## الخاتمة

`KycDocumentManager` هو الحل الشامل والمرن لربط وثائق KYC بأي موديل في نظامك. بفضل تصميمه المعتمد على الإعدادات المركزية، ودعمه لأعمدة وفلاتر متعددة، ونطاقات Toggle الذكية، يمكنك بناء تجارب إدارة KYC متطورة في لوحة التحكم وواجهات API ببضعة أسطر من الكود.

---

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
