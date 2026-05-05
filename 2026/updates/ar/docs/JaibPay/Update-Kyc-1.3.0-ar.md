## 2026-05-03 - 2026-05-04

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.3.0**

### ملخص التحديثات

يأتي الإصدار 1.3.0 ليُكمِل بناء نظام تكاملات KYC الذي بدأ في 1.2.9، وينقله إلى مرحلة **جاهزية إنتاجية كاملة** مع دعم واسع لإنشاء فلاتر متقدمة في لوحة التحكم، وأعمدة متعددة قابلة للتخصيص، ونطاقات Toggle احترافية، وإدارة مركزية للإعدادات عبر ملفات Config.

يركز هذا الإصدار على:

- **نطاقات Toggle متطورة** (تبديل) مستوحاة من سلوك `AssociationsModel`، تدعم الفلاتر من نوع `switch` و`checkbox`.
- **فلاتر جماعية متعددة** (group filters) مع دعم `dependsOn` لبناء فلاتر متصلة (مثلاً تصنيف الوثيقة ← نوع الوثيقة).
- **أعمدة قوائم متعددة** (document counts, is_verifier_*) مع خيارات OctoberCMS الكاملة.
- **مركزية الإعدادات الافتراضية** عبر ملف `config/extend.php`، مما يسمح بتخصيص الأعمدة والفلاتر والإعدادات بدون تعديل الكود البرمجي.
- **تسجيل تلقائي لنماذج المستخدمين** (Frontend و Backend) مع التكاملات الأساسية.
- **تحسين تكامل Transformer** ليدعم `default_include` و `eager_load` للوثائق.

كل ذلك مع توسيع ملفات الترجمة لتشمل جميع التسميات الجديدة، وتحسينات على معالجة الاستعلامات المتقدمة.

---

### الإصدار 1.3.0 – النضج الكامل لتكاملات وثائق KYC

#### أهداف الإصدار

- **توسيع نطاقات السلوك `KycDocumentModel`** بنطاقات Toggle متوافقة مع آلية فلاتر OctoberCMS.
- **دعم فلاتر متعددة ومتطورة** في لوحة التحكم (group, checkbox, switch) مع خيارات ديناميكية وتبعيات بينها.
- **مركزية إعدادات التكاملات** في ملف `config/extend.php` لتسهيل التخصيص والإدارة.
- **دعم أعمدة متعددة** في القوائم (عدد الوثائق بأنواعها، مؤشرات الفئات).
- **تحسين تكامل المحولات** وربط النماذج الشائعة تلقائياً.

#### الميزات الجديدة والتحسينات

##### 1. نطاقات Toggle متقدمة في `KycDocumentModel`

تمت إضافة مجموعة من النطاقات التي تدعم فلاتر التبديل (`switch` و `checkbox`) في أكتوبر، مما يسمح بإنشاء تجارب فلترة سلسة:

| النطاق | النوع | الوصف |
| :--- | :--- | :--- |
| `scopeIsToggelAnyKycDocuments` | switch | وجود أي وثيقة (تبديل بين "لديه وثائق" و "ليس لديه وثائق") |
| `scopeIsToggelKycDocumentsVerified` | checkbox | وجود وثائق معتمدة |
| `scopeIsToggelKycDocumentsNotVerified` | checkbox | وجود وثائق غير معتمدة |
| `scopeIsToggelKycDocumentsPending` | checkbox | وجود وثائق معلقة |
| `scopeIsToggelKycDocumentsExpired` | checkbox | وجود وثائق منتهية |
| `scopeIsToggelKycDocumentsByType` | أي | فلترة حسب نوع الوثيقة |
| `scopeIsToggelKycDocumentsByCategory` | أي | فلترة حسب فئة الوثيقة |
| `scopeIsToggelKycDocumentsVerifiedAndCategory` | checkbox | شرط مزدوج: وثيقة معتمدة **و** ضمن فئة محددة (مثلاً `primary_id`) |
| `scopeIsToggelKycDocumentsVerifiedAndType` | checkbox | شرط مزدوج: وثيقة معتمدة **و** من نوع محدد (مثلاً `passport`) |

جميع هذه النطاقات تدعم آلية التبديل الذكي: عند الضغط على الفلتر، تنعكس القيمة تلقائياً (من 1 إلى 0) بفضل دالة `post('scopeName')`.

**مثال لإعدادات الفلتر في الملف:**

```yaml
is_toggel_any_kyc_documents:
    label: 'حالة الوثائق'
    type: switch
    scope: isToggelAnyKycDocuments
    default: 0

is_toggel_kyc_documents_verified_and_category_primary_id:
    label: 'هوية أساسية معتمدة'
    type: checkbox
    scope: isToggelKycDocumentsVerifiedAndCategoryPrimaryId
    default: 0
```

##### 2. فلاتر جماعية مع تبعيات (Group Filters with `dependsOn`)

يدعم الإصدار الآن فلاتر متعددة في نفس القائمة، مترابطة عبر `dependsOn`. مثلاً:

- **فلتر تصنيف الوثيقة** (`kyc_documents_category`) – يختار المستخدم فئة (primary_id, address، ...).
- **فلتر نوع الوثيقة** (`kyc_documents_type`) – يعتمد على الفئة المختارة، ويعرض أنواع الوثائق المنتمية لتلك الفئة فقط.

تم تحقيق ذلك عبر:

- دوال `getDocumentCategoryFilterOptions` و `getDocumentTypeFilterOptions` في موديل `Document`.
- دعم كامل لخيار `dependsOn` في تعريفات الفلاتر بملف `config/extend.php`.

مثال من ملف الإعدادات:

```php
[
    'scope_name' => 'kyc_documents_category',
    'label'      => 'تصنيف الوثيقة',
    'type'       => 'group',
    'modelClass' => 'Nano3\Kyc\Models\Document',
    'options'    => 'getDocumentCategoryFilterOptions',
    'dependsOn'  => [],
],
[
    'scope_name' => 'kyc_documents_type',
    'label'      => 'نوع الوثيقة',
    'type'       => 'group',
    'modelClass' => 'Nano3\Kyc\Models\Document',
    'options'    => 'getDocumentTypeFilterOptions',
    'dependsOn'  => ['kyc_documents_category'],
],
```

##### 3. أعمدة قوائم متعددة ومخصصة

أضفنا مجموعة أعمدة افتراضية جاهزة في `config/extend.php`، يمكن استخدامها مباشرة أو تخصيصها:

- `documents_count` – إجمالي عدد الوثائق.
- `verified_documents_count` – عدد الوثائق المعتمدة.
- `not_verified_documents_count` – عدد الوثائق غير المعتمدة.
- `expired_verified_documents_count` – عدد الوثائق المنتهية (المعتمدة).
- `is_verifier_primary_id`، `is_verifier_secondary_id`، ... – مؤشرات (switch) لوجود وثائق من كل فئة.

كل عمود يدعم جميع خيارات أكتوبر القياسية (label, type, sortable, width, align، ...). لضمان ظهور القيم الإحصائية، تم إضافة نطاقات عد جديدة:

```php
public function scopeAddVerifiedDocumentsCount($query, $columnName = 'verified_documents_count')
public function scopeAddPendingDocumentsCount($query, $columnName = 'pending_documents_count')
public function scopeAddRejectedDocumentsCount($query, $columnName = 'rejected_documents_count')
public function scopeAddExpiredDocumentsCount($query, $columnName = 'expired_documents_count')
```

ولكي تعمل هذه الأعمدة، يجب تطبيق النطاقات على استعلام القائمة، ويقوم `KycDocumentManager` بذلك تلقائياً عبر التكاملات.

##### 4. مركزية الإعدادات الافتراضية عبر `config/extend.php`

بدلاً من كتابة إعدادات الأعمدة والفلاتر بشكل مكرر، تم نقل جميع الإعدادات الافتراضية إلى ملف `plugins/nano3/kyc/config/extend.php`، والذي يحتوي على:

- `form_config` – إعدادات حقل النموذج الافتراضي.
- `list_config` – مصفوفة تعريفات الأعمدة الافتراضية.
- `filter_config` – مصفوفة تعريفات الفلاتر الافتراضية.
- `query_config` – إعدادات استعلام API الافتراضية.

عند تسجيل أي نموذج، يتم تحميل هذه الإعدادات تلقائياً من الملف (إن وُجد)، مع إمكانية تجاوزها عبر الكود. هذا يسمح بتخصيص شامل لكل موديل دون تعديل الملفات الأساسية.

##### 5. تحسين تكامل Transformer

تم تحسين دالة `applyTransformerIntegration` لتدعم:

- `default_include` – إضافة `kyc_status` إلى قائمة `defaultIncludes` مما يجعله يظهر تلقائياً في كل استجابة API.
- `eager_load` – تحميل علاقة `documents` تلقائياً مع الاستعلام الرئيسي لتحسين الأداء.
- دعم `additional_fields` – إضافة حقول مخصصة إلى الـ Transformer يمكن تضمينها عند الطلب.

##### 6. تسجيل تلقائي لمستخدمي الواجهة الأمامية والخلفية

تمت إضافة دالة `registerKycDocumentIntegrations` في الـ Plugin الرئيسي لتسجيل كل من `RainLab\User\Models\User` و `Backend\Models\User` مع التكاملات التالية كإعداد افتراضي:

```php
'integrations' => [
    KycDocumentManager::INTEGRATION_BEHAVIOR    => true,  // السلوك
    KycDocumentManager::INTEGRATION_LIST_COLUMN => true,  // أعمدة القائمة
    KycDocumentManager::INTEGRATION_LIST_FILTER => true,  // فلاتر القائمة
    KycDocumentManager::INTEGRATION_FORM_FIELD  => false, // يمكن تفعيله حسب الحاجة
    KycDocumentManager::INTEGRATION_QUERY       => false,
    KycDocumentManager::INTEGRATION_TRANSFORMER  => false,
],
```

هذا يعني أن قوائم المستخدمين في لوحة التحكم ستعرض تلقائياً أعمدة الوثائق وفلاترها بمجرد تفعيل الإضافة.

---

### أمثلة عملية

#### 1. استخدام نطاقات Toggle الجديدة

```php
// المستخدمون الذين لديهم وثائق معتمدة
User::isToggelKycDocumentsVerified(1)->get();

// المستخدمون الذين ليس لديهم وثائق منتهية (toggle بإرسال 0)
User::isToggelKycDocumentsExpired(0)->get();

// المستخدمون الذين لديهم هوية أساسية معتمدة (شرط مزدوج)
User::isToggelKycDocumentsVerifiedAndCategory(1, 'primary_id')->get();
```

#### 2. إعداد ملف `config_filter.yaml` مع الفلاتر الجديدة

```yaml
scopes:
    kyc_documents_category:
        label: 'تصنيف الوثيقة'
        type: group
        modelClass: Nano3\Kyc\Models\Document
        options: getDocumentCategoryFilterOptions
        scope: kycDocumentsCategory

    kyc_documents_type:
        label: 'نوع الوثيقة'
        type: group
        modelClass: Nano3\Kyc\Models\Document
        options: getDocumentTypeFilterOptions
        scope: kycDocumentsType
        dependsOn: kyc_documents_category

    is_toggel_any_kyc_documents:
        label: 'وثائق (أي)'
        type: switch
        scope: isToggelAnyKycDocuments
        default: 0

    is_toggel_kyc_documents_verified:
        label: 'معتمدة'
        type: checkbox
        scope: isToggelKycDocumentsVerified
        default: 0
```

#### 3. تخصيص الأعمدة الافتراضية لمشروع معين

في حال أردت إضافة عمود جديد أو تغيير ترتيب الأعمدة، ما عليك سوى نشر ملف الإعدادات وتعديله:

```bash
php artisan config:publish nano3.kyc
```

ثم تحرير `config/extend.php` في مشروعك.

---

### ملخص الإصدارات (1.2.9 – 1.3.0)

| الإصدار | أبرز الميزات والإصلاحات |
| :--- | :--- |
| 1.2.9 | نظام تكاملات KYC المتكامل: سلوك `KycDocumentModel`، كلاس `KycDocumentManager`، معالج `KycIntegrationHandler`، معالجة ذكية لمعاملات API، توثيق شامل. |
| 1.3.0 | **نضج التكاملات**: نطاقات Toggle متقدمة، فلاتر جماعية مع تبعيات، أعمدة قوائم متعددة، مركزية الإعدادات، تسجيل تلقائي لنماذج المستخدمين، تحسين Transformer. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال الملفات التالية بالنسخ الجديدة:
     - `behaviors/KycDocumentModel.php`
     - `behaviors/KycDocumentModel/KycDocumentScopesAndHelpers.php`
     - `classes/KycDocumentManager.php`
     - `eventhandlers/KycIntegrationHandler.php`
     - `Plugin.php` (لإضافة دالة `registerKycDocumentIntegrations`)
   - إضافة ملف `config/extend.php` إن لم يكن موجوداً.
   - تحديث ملفات الترجمة `lang/ar/lang.php` و `lang/en/lang.php` بالمفاتيح الجديدة في قسم `extend`.

2. **تسجيل النماذج الإضافية**:
   - إذا كنت تستخدم نماذج مخصصة، يمكنك إضافتها إلى دالة `registerKycDocumentIntegrations` في الـ Plugin الخاص بك.

3. **اختبار الفلاتر والأعمدة**:
   - تحقق من ظهور الأعمدة الجديدة في قوائم المستخدمين.
   - جرب فلاتر التصنيف والنوع والـ switch للتأكد من عملها.
   - تأكد من أن الفلاتر المترابطة (category ← type) تعمل بشكل صحيح.

4. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

الإصدار 1.3.0 هو تتويج لسلسلة تطويرات جعلت من إضافة `Nano3.Kyc` أداة متكاملة ومرنة لإدارة وثائق الهوية في برمجيات نانوسوفت. بفضل النطاقات المتطورة، تكاملات لوحة التحكم، ومركزية الإعدادات، يمكن للمطورين الآن بناء أنظمة KYC معقدة بسرعة وسهولة، مع تجربة مستخدم راقية في لوحة التحكم.

نشكركم على دعمكم المستمر، ونرحب بملاحظاتكم لمواصلة تحسين الإضافة.

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
