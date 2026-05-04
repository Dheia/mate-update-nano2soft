# Update 2026-5

**تحديثات شهر خمسة مايو **

## 2026-02-17 – 2026-05-01

**تحديث شامل لـ `Nano.AccountsApi` و `Tss.Accounts` – الإصدار 1.0.38**

### إضافة دعم القيود المحاسبية متعددة الأطراف وفروق العملات والتحقق المتقدم

---

### ملخص التحديثات

في الإصدار **1.0.38**، تم إجراء قفزة نوعية في البنية التحتية المحاسبية للإضافات الأساسية (`Tss.Accounts`) والواجهة البرمجية (`Nano.AccountsApi`) من خلال:

- **إضافة ترايت `TransactionsMultiHelper`** الذي يضم مجموعة متكاملة من الأدوات لإنشاء القيود المحاسبية متعددة الأطراف، مع نظام تحقق شامل وقابل للتخصيص.
- **دعم متكامل لفروق العملات** عبر دوال `checkExchangeDifferences` و`createExchangeDifferenceEntry` و`handleExchangeDifferences`، مما يسمح بإنشاء قيود تسوية فروق الصرف تلقائياً.
- **توسيع خيارات التحكم** لتشمل حدود المبالغ (الإجمالية والخاصة بكل جانب)، فحص الأرصدة، قواعد ديناميكية لفحص أنواع الحسابات، والتحكم في الحقول الإضافية (مثل `modul_type` و `batch_id`).
- **تحديث `TransactionsHelper`** لدمج الترايت الجديد، وإتاحة دالة `createEntry` الموحدة التي تختار تلقائياً بين القيد البسيط ومتعدد الأطراف.
- **تحسين إدارة `paper_id`** مع خيارات التوليد التلقائي والتحقق من التكرار.
- **دوال مساعدة للاختبار والمحاكاة** مثل `testJournalEntry` لتحليل القيد قبل إنشائه.
- **تحديثات طفيفة في `AccountHelper`** لضمان التوافق مع الأنواع الجديدة.

كل هذه التحسينات تؤثر بشكل مباشر على `Nano.AccountsApi` حيث يستفيد منها `BaseDocumentController` وجميع المتحكمات الجديدة (BondsDays، CatchReceiptsV2، ...).

---

### أهداف الإصدار

- **دعم القيود المحاسبية المعقدة** (أكثر من طرفين) عبر API بشكل أصلي وآمن.
- **أتمتة معالجة فروق العملات** الناتجة عن تغير أسعار الصرف بين تاريخ القيد وتاريخ الترحيل.
- **توفير تحكم دقيق في كل تفصيلة** (الشخص، الحساب، الملاحظات، الحقول الإضافية) لكل جانب من جوانب القيد.
- **زيادة موثوقية وسلامة البيانات** من خلال فحوصات متعددة (الرصيد، حدود المبالغ، نوع الحساب، توازن العملات).
- **تسهيل اختبار القيود** قبل حفظها في قاعدة البيانات عبر دالة `testJournalEntry`.
- **الحفاظ على التوافق العكسي** مع الدوال السابقة (`createSingle`، `addPayment`، إلخ).

---

### الميزات الجديدة والتحسينات

#### 1. ترايت `TransactionsMultiHelper` (الجديد)

تمت إضافة ترايت `TransactionsMultiHelper` في الملف `Tss\Accounts\Helpers\TransactionsMultiHelper.php`، والذي يحتوي على جميع الدوال المتعلقة بالقيود متعددة الأطراف. تم تصميمه ليكون سهل الاستخدام ومتكاملاً مع `TransactionsHelper`.

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `createJournalEntry` | إنشاء قيد محاسبي متعدد الأطراف (3+ أطراف) مع تحقق كامل ومعالجة العملات وفروق الصرف، مع أحداث `beforeJournalEntry` و `afterJournalEntry`. |
| `createEntry` | دالة موحدة تختار تلقائياً بين `createSingle` (لطرفين) و`createJournalEntry` (لأكثر من طرفين)، ويمكن فرض استخدام `createJournalEntry` عبر الخيار `forceJournal => true`. |
| `validateJournalEntry` | التحقق من صحة بيانات القيد متعدد الأطراف دون إنشائه، وتعيد التفاصيل بعد المعالجة. |
| `checkExchangeDifferences` | فحص قيد موجود لتحديد فروق العملات (بناءً على تاريخ الترحيل وسعر الصرف الرسمي)، مع خيارات للإنشاء (`entry`) أو الإطلاق (`event`) أو الإرجاع فقط (`none`). |
| `createExchangeDifferenceEntry` | إنشاء قيد تسوية لفارق صرف واحد، بربطه بالقيد الأصلي. |
| `handleExchangeDifferences` | معالجة داخلية تُستدعى بعد إنشاء القيد لفحص الفروق واتخاذ الإجراء المحدد. |
| `testJournalEntry` | محاكاة كاملة لإنشاء القيد (بما في ذلك التحقق ومعالجة العملات) مع تحليل مفصل للنتائج ومخالفات القواعد. |
| `applyAccountRules` | تطبيق مجموعة قواعد ديناميكية على حساب للتحقق من نوعه أو خصائصه (مثل `reports = 1` و `modul_type IN ['Boxe','Bank']`). |
| `checkAccountBalance` | فحص رصيد حساب معين قبل تنفيذ الحركة. |
| `processCurrency` | معالجة العملة وسعر الصرف لكل تفصيل، مع دعم التحويل التلقائي (`auto_convert_currency`). |
| `calculateTotals` | حساب المجاميع (الأساسية والأجنبية) للتفاصيل. |
| `validateMultiplicity` | التحقق من قيود عدد الأطراف (الحد الأقصى، تعدد المدين/الدائن). |
| `validateAmountLimits` | التحقق من حدود المبالغ الإجمالية والخاصة بكل جانب. |
| `validateBalance` | تفعيل فحص الرصيد لكل تفصيل حسب الإعدادات. |
| `validatePaperId` / `preparePaperId` | إدارة توليد والتحقق من `paper_id` تلقائياً. |

**ميزات الترايت:**

- **تكامل كامل** مع `TransactionsHelper` بمجرد إضافة `use TransactionsMultiHelper;`.
- **استخدام `Db::transaction`** لضمان سلامة البيانات في `createJournalEntry`.
- **دعم أحداث `beforeJournalEntry` و `afterJournalEntry`**.
- **دعم `beforeValidate` و `afterValidate` كـ hooks**.
- **دعم تعطيل قواعد محددة** عبر `disabled_rules` (مثل `'balance_check'`, `'multiplicity'`).
- **بنية خيارات موحدة** تتضمن جميع الإعدادات الممكنة لرأس القيد والتفاصيل.

#### 2. إعدادات شاملة وقابلة للتخصيص في `createJournalEntry`

يوفر الترايت الجديد تحكماً كاملاً في جميع جوانب القيد المحاسبي من خلال مصفوفة `$options`. فيما يلي أبرز مجموعات الإعدادات المدعومة (انظر ملف `createJournalEntry-config.md` للقائمة الكاملة):

- **حقول الرأس الأساسية:** `companys_id`, `departments_id`, `date_at`, `transactions_type`, إلخ.
- **الشخص على مستوى الرأس:** `header_person_id`, `header_person_type`, `header_person`.
- **قيود عدد الأطراف:** `max_debit_entries`, `max_credit_entries`, `allow_multiple_debit`, `allow_multiple_credit`.
- **التحكم في الشخص لكل جانب:** `debit_person_allowed`, `debit_person_required`, `debit_person_allowed_types` (ونظائرها للدائن).
- **التحكم في الحساب لكل جانب:** `debit_account_allowed`, `debit_account_required`, `debit_default_account`.
- **التحكم في الملاحظات لكل جانب:** `debit_notes_allowed`, `debit_notes_required`, `debit_default_notes`.
- **حدود المبالغ:** `min_total_amount`, `max_total_amount`، `check_debit_min`، `min_debit_amount`، إلخ.
- **فحص الرصيد:** `debit_check_balance` و `debit_balance_check_callback` (مع دعم دوال مخصصة).
- **فحص نوع الحساب:** `debit_check_account_types` و `debit_rules_account_types` (قواعد ديناميكية تدعم `AND/OR` ومشغلات مثل `=`, `IN`, `LIKE`, `REGEXP`).
- **التحكم في الحقول الإضافية:** `header_extend_id_allowed`، `debit_batch_id_required`، `credit_modul_type_default`، إلخ. (تغطي الحقول: `extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`).
- **إعدادات `paper_id`:** `is_paper_id`, `paper_id_allowed`, `paper_id_auto`, `paper_id_required`, `paper_id_check_duplicate`, `paper_id_duplicate_scope`.
- **إعدادات التاريخ:** `is_custome_date`, `date_at_auto`, `date_at_required`.
- **العملات وفروق الصرف:** `auto_convert_currency`, `handle_exchange_differences`, `exchange_difference_action` (`'entry'`, `'event'`, `'none'`).
- **Hooks:** `beforeValidate`, `afterValidate`.
- **تعطيل القواعد:** `disabled_rules`.

#### 3. تكامل `TransactionsMultiHelper` مع `TransactionsHelper`

- تمت إضافة السطر `use \Tss\Accounts\Helpers\TransactionsMultiHelper;` داخل كلاس `TransactionsHelper`، مما يجعل جميع دوال الترايت الجديد متاحة كجزء من الكلاس.
- تم تعديل بسيط في `createSingle` لتهيئة `$result` كمصفوفة فارغة (`$result = [];`) لضمان عدم حدوث أخطاء في السيناريوهات المختلفة.

#### 4. دعم فروق العملات (Exchange Differences)

تمت إضافة آليات متكاملة للتعامل مع فروق العملات الناتجة عن تغير سعر الصرف بين تاريخ القيد وتاريخ الترحيل:

- **`checkExchangeDifferences`**: تستقبل معرف القيد أو كائنه، وتفحص كل تفصيل بعملة غير رئيسية، وتقارن سعره الأصلي مع السعر الرسمي في تاريخ الترحيل.
- **`createExchangeDifferenceEntry`**: تنشئ قيداً محاسبياً لتسوية الفرق (مُدين/دائن) بين حساب فروق الصرف والحساب الأصلي.
- **`handleExchangeDifferences`**: تُستدعى تلقائياً بعد نجاح `createJournalEntry` إذا كان الخيار `handle_exchange_differences = true`.
- **خيارات التحكم:** `threshold` (العتبة الدنيا للفرق)، `action` (`'entry'`، `'event'`، `'none'`)، `force_check`، `relay_date`، `exchange_difference_account`.

#### 5. تحسينات `AccountHelper`

على الرغم من أن التغييرات طفيفة، إلا أن `AccountHelper::getObjHeaderFromType` أصبحت تدعم الآن أسماء الكلاسات (class names) بالإضافة إلى السلاسل النصية للأنواع (مثل `'Transfer'` و `Transfer::class`)، وهو ما يعزز التوافق مع المتحكمات الجديدة.

#### 6. تأثير التحديثات على `Nano.AccountsApi`

- **`BaseDocumentController`**: يستخدم الآن `TransactionsHelper::createEntry` (التي تم جلبها من الترايت) بدلاً من `createSingle` مباشرة، مما يسمح بإنشاء قيود بأي عدد من الأطراف.
- **`BondsDays`**: يستفيد بشكل كامل من `createJournalEntry` عبر `createEntry`، مع تفعيل التحقق من توازن القيد ومنع ترحيل الحسابات الرئيسية.
- **جميع المتحكمات** (`CatchReceiptsV2`, `PayReceipts`, etc.) يمكنها الآن قبول تفاصيل متعددة وتمرير خيارات متقدمة.
- **دالة `test` في `BondsDays`** تستخدم `TransactionsHelper::testJournalEntry` لاختبار القيد قبل حفظه.

---

### أمثلة على الاستخدام

#### إنشاء قيد متعدد الأطراف مع تحديد حدود المبالغ وفحص نوع الحساب

```php
$result = TransactionsHelper::createEntry([
    'date_at' => '2026-05-18',
    'notes'   => 'قييد تسوية',
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 3000],
        ['accounts_id' => '2-2-4111010001', 'credit' => 2000],
    ],
    'min_total_amount' => 1000,
    'max_total_amount' => 10000,
    'debit_check_account_types' => true,
    'debit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
    ],
]);
```

#### اختبار قيد قبل إنشائه

```php
$test = TransactionsHelper::testJournalEntry([
    'details' => [...],
]);
if ($test['analysis']['test_passed']) {
    echo "✅ القيد صالح وجاهز للحفظ.";
} else {
    print_r($test['analysis']['rule_violations']);
}
```

#### فحص فروق العملات لقيد موجود وإنشاء قيد تسوية

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'threshold' => 0.01,
    'action'    => 'entry',
    'relay_date' => Carbon::parse('2026-05-20'),
]);
if ($result['exchange_entry']) {
    echo "تم إنشاء قيد فروق رقم: " . $result['exchange_entry']->id;
}
```

---

### الفوائد والقيمة المضافة

- **مرونة غير مسبوقة**: يمكن للمطورين الآن إنشاء قيود بأي عدد من الأطراف، مع تحكم دقيق في كل جانب، مما يفتح المجال لتطبيقات محاسبية متطورة.
- **أمان ودقة**: فحوصات الرصيد ونوع الحساب وحدود المبالغ تقلل من احتمالية الأخطاء المحاسبية.
- **أتمتة العمليات**: معالجة فروق العملات تتم تلقائياً دون تدخل يدوي، مما يوفر الوقت ويمنع الأخطاء.
- **سهولة التطوير**: بفضل الدوال المساعدة مثل `testJournalEntry`، يمكن للمطورين اختبار القيود قبل إرسالها.
- **توافق عكسي كامل**: جميع الدوال السابقة ما زالت تعمل بنفس التوقيع، ويمكن استخدام الميزات الجديدة بشكل تدريجي.
- **تحسين أداء `Nano.AccountsApi`**: المتحكمات الجديدة أصبحت قادرة على التعامل مع السيناريوهات المعقدة التي تتطلبها الأنظمة المالية الحديثة.

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال الملفات التالية في `Tss.Accounts`:
     - `helpers/TransactionsHelper.php`
     - إضافة الملف الجديد `helpers/TransactionsMultiHelper.php`
     - (اختياري) `helpers/AccountHelper.php` إذا كان هناك حاجة للتحديثات الطفيفة.
   - تحديث `Nano.AccountsApi` إلى الإصدار الذي يستخدم `createEntry` (عادة ما يكون `BaseDocumentController` والمتحكمات المبنية عليه – إذا لم تكن قد حدثت من قبل، فستحتاج إلى الإصدار 1.1.0 أولاً أو دمج الميزات معاً).

2. **لا توجد هجرات قاعدة بيانات جديدة**.
3. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
4. **اختبار الوظائف**:
   - جرب إنشاء قيد متعدد الأطراف باستخدام `bonds/add`.
   - استخدم `bonds/test` لاختبار صحة القيد.
   - تأكد من أن فروق العملات تُعالج بشكل صحيح إذا كنت تستخدم عملات متعددة.

---

### ملخص الإصدارات

| الإصدار | أبرز الميزات والإصلاحات |
| :--- | :--- |
| 1.1.0 | إضافة `BaseDocumentController` وأنواع المستندات الجديدة (BondsDays, PayReceipts, DebitNotes, CreditNotes, Transfers, CatchReceiptsV2). |
| 1.0.38 | إضافة `TransactionsMultiHelper`: قيود متعددة الأطراف، فروق عملات، فحوصات متقدمة (الرصيد، نوع الحساب، حدود المبالغ)، إعدادات شاملة، دالة `testJournalEntry`. |

---

### الخاتمة

يمثل الإصدار 1.0.38 نقلة نوعية في قوة ومرونة النظام المحاسبي في منصة نانوسوفت. مع `TransactionsMultiHelper`، أصبح بإمكان المطورين إنشاء قيود محاسبية معقدة بسهولة وأمان، مع تغطية كاملة لحالات الاستخدام الواقعية مثل فروق العملات والتحقق من الحسابات. هذه الميزات تُترجم مباشرة إلى `Nano.AccountsApi`، مما يوفر واجهة برمجة غنية وقوية للتطبيقات المالية.

نشكركم على دعمكم المستمر، ونتطلع إلى ملاحظاتكم لمزيد من التطوير.

---

### التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./docs/Accounts/Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./docs/Accounts/Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./docs/Accounts/Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./docs/Accounts/Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./docs/Accounts/Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./docs/Accounts/Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./docs/Accounts/Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./docs/Accounts/Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./docs/Accounts/Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)


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

## 2026-05-02 - 2026-05-03


### تحديث شامل لنطاقات فلترة الطلبات حسب مالك المنتج في موديل `Order`


**الإضافة:** `Nano.Orders`  
**الملف:** `HasProductOwnerScopes` trait (`plugins/nano/orders/models/Order/HasProductOwnerScopes.php`)

---

### تطوير نطاقات استعلام متقدمة لفلترة الطلبات بناءً على ملكية المنتجات

تم تطوير ترايت `HasProductOwnerScopes` وإضافته إلى موديل `Order`، ليمنح المطورين القدرة على فلترة الطلبات بمرونة عالية اعتماداً على المنتجات المملوكة لمستخدم معين. لم يعد المطور بحاجة لكتابة استعلامات فرعية معقدة أو التعامل مع علاقات متداخلة يدوياً، بل أصبح بإمكانه استخدام نطاقات جاهزة تدعم:

- التصفية حسب **مالك المنتج** (`user_type` و `user_id` في جدول `tss_inventory_products`).
- تحديد **عدد المنتجات المملوكة** داخل الطلب (باستخدام `count` و `countOperator`).
- **النفي** (جلب الطلبات التي لا تحتوي منتجات للمالك).
- **الدمج البوليني** (`AND` / `OR`) لربط الشروط مع نطاقات أخرى.
- **شروط إضافية** على المنتج نفسه (السعر، الحالة...) أو على عنصر الطلب (الكمية، السعر الفعلي...).
- **إضافة عمود محسوب** بعدد المنتجات المملوكة في الطلب، و**الترتيب** بناءً عليه.
- **دوال فحص سريعة** على كائن الطلب الواحد (مثلاً `hasAnyProductByOwner`).

تم نقل جميع النطاقات والدوال المساعدة إلى ترايت منفصل لتسهيل الصيانة وإعادة الاستخدام، مع الالتزام باستخدام ثوابت أسماء الجداول لضمان استقرار الاستعلامات.

---

### 1. المكونات المطورة

| السلوك | المكون | الوصف |
|--------|--------|-------|
| `Order` (Model) | `HasProductOwnerScopes` (trait) | يحتوي على جميع النطاقات والدوال المساعدة لفلترة الطلبات حسب مالك المنتج. |

#### النطاقات الرئيسية والمساعدة

| الفئة | النطاق | الوصف |
|-------|--------|-------|
| **نطاق متقدم** | `scopeWhereHasProductsByOwner` | النطاق الرئيسي بمصفوفة خيارات شاملة (user, count, boolean, not, conditions, itemConditions). |
| **نطاقات مساعدة** | `scopeWhereProductOwner` | اختصار سريع: طلبات تحتوي منتجاً واحداً على الأقل للمالك. |
| | `scopeHasProductsByOwner` | فلترة حسب وجود عدد أدنى من المنتجات المملوكة. |
| | `scopeDoesntHaveProductsByOwner` | فلترة الطلبات التي لا تحتوي أي منتج مملوك. |
| | `scopeHasProductsByOwnerCount` | تحديد عدد دقيق (أو بمقارنة) للمنتجات المملوكة. |
| | `scopeOrWhereHasProductsByOwner` | ربط النطاق مع `OR`. |
| **أعمدة محسوبة وترتيب** | `scopeWithProductsByOwnerCount` | إضافة عمود `products_owner_count` إلى `SELECT`. |
| | `scopeSortByProductsByOwnerCount` | ترتيب الطلبات تصاعدياً/تنازلياً حسب عدد المنتجات المملوكة. |
| **دوال الفحص** | `hasAnyProductByOwner` | فحص ما إذا كان الطلب يحتوي على منتج واحد على الأقل يملكه المستخدم. |

---

### 2. تفاصيل التحديثات البرمجية

#### 2.1 النطاق الرئيسي `whereHasProductsByOwner`
تم تصميم هذا النطاق ليكون شاملاً ويقبل مصفوفة خيارات واحدة، مما يوحد واجهة الاستخدام ويغني عن عشرات النطاقات المنفصلة. الخيارات المدعومة:

| الخيار | النوع | الوصف |
|--------|-------|-------|
| `user` | `Authenticatable\|Model` | كائن المستخدم المالك. |
| `userType` | `string` | نوع المستخدم (مثلاً `RainLab\User\Models\User`). |
| `userId` | `int` | معرّف المستخدم. |
| `boolean` | `string` | `'and'` أو `'or'` للربط باستعلام خارجي. |
| `not` | `bool` | `true` للنفي (`doesntHave`). |
| `count` | `int` | عدد المنتجات المملوكة المطلوب. |
| `countOperator` | `string` | عامل المقارنة (`>=`, `=`, `<`...). |
| `conditions` | `array\|callable` | شروط إضافية على جدول المنتجات. |
| `itemConditions` | `array\|callable` | شروط إضافية على جدول عناصر الطلب. |

في حال عدم تمرير بيانات المالك، يحاول النطاق تلقائياً جلب المستخدم المسجّل حالياً عبر `AuthHelpers` أو `BackendAuth` أو `Auth`. إذا تعذر تحديد المالك، يُعيد الاستعلام بدون تطبيق أي فلتر، مما يحمي من نتائج فارغة غير مقصودة.

#### 2.2 النطاقات المساعدة
النطاقات المساعدة (`HasProductsByOwner`، `DoesntHaveProductsByOwner`، `HasProductsByOwnerCount`، `OrWhereHasProductsByOwner`) مجرد أغلفة تستدعي النطاق الرئيسي مع ضبط الخيارات المناسبة. هذا يضمن عدم تكرار الكود ويجعل الصيانة مركزية.

#### 2.3 أعمدة محسوبة وترتيب
- `withProductsByOwnerCount`: يضيف عموداً محسوباً بعدد المنتجات المملوكة داخل الطلب باستخدام `selectSub`.
- `sortByProductsByOwnerCount`: يرتب النتائج بناءً على ذلك العمود باستخدام `orderByRaw` مع تمرير bindings بشكل آمن، لتجنب الأخطاء الناتجة عن استخدام `DB::raw` دون bindings.

#### 2.4 دوال الفحص
- `hasAnyProductByOwner($user)`: دالة سريعة على كائن `Order` للتحقق من وجود منتج مملوك واحد على الأقل للمستخدم (أو المستخدم الحالي تلقائياً).

#### 2.5 استخدام ثوابت الجداول
لتجنب الأخطاء عند تغيير أسماء الجداول، تم استخدام:
- `\Nano\Shop\Models\Product::TABLE_NAME` للإشارة إلى جدول المنتجات.
- `(new \Nano\Orders\Models\OrderItem)->getTable()` للإشارة إلى جدول عناصر الطلب.

كما تم تضمين اسم الجدول قبل كل حقل في الشروط لتجنب الغموض في الاستعلامات التي تنضم لأكثر من جدول.

---

### 3. أمثلة تطبيقية

#### 3.1 طلبات تحتوي على أي منتج للمستخدم الحالي
```php
$orders = Order::hasProductsByOwner()->get();
```

#### 3.2 طلبات لا تحتوي أي منتج لمستخدم معين
```php
$user = \BackendAuth::getUser();
$orders = Order::doesntHaveProductsByOwner($user)->get();
```

#### 3.3 طلبات تحتوي بالضبط 3 منتجات مملوكة مع شرط إضافي على المنتج (السعر > 50)
```php
$orders = Order::whereHasProductsByOwner([
    'user'       => $user,
    'count'      => 3,
    'countOperator' => '=',
    'conditions' => function ($q) {
        $q->where('price', '>', 50);
    }
])->get();
```

#### 3.4 طلبات تحتوي منتجاً مملوكاً مع كمية أكبر من 2 في الطلب
```php
$orders = Order::whereHasProductsByOwner([
    'user'           => $user,
    'itemConditions' => function ($q) {
        $q->where('quantity', '>', 2);
    }
])->get();
```

#### 3.5 دمج مع حالة الطلب باستخدام OR
```php
$orders = Order::isNewOrders()
    ->orWhereHasProductsByOwner(['user' => $user])
    ->get();
```

#### 3.6 إضافة عمود عدد المنتجات المملوكة والترتيب به تنازلياً
```php
$orders = Order::withProductsByOwnerCount()
    ->sortByProductsByOwnerCount('desc')
    ->get();

foreach ($orders as $order) {
    echo $order->products_owner_count;
}
```

#### 3.7 فحص طلب مفرد
```php
$order = Order::find(10);
if ($order->hasAnyProductByOwner($user)) {
    // معالجة
}
```

---

### 4. القيمة المضافة

- **للمطورين**: واجهة موحدة ومرنة تغطي جميع حالات فلترة الطلبات بناءً على ملكية المنتجات، دون الحاجة لكتابة استعلامات فرعية معقدة. الخيارات المتقدمة مثل `count`، `not`، `conditions`، و `itemConditions` تفتح آفاقاً واسعة للتقارير ولوحات التحكم.
- **للتطبيق**: أداء محسّن باستخدام `whereHas` و `whereHas` مع `count`، واستعلامات فرعية آمنة للـ bindings. الاعتماد على ثوابت الجداول يمنع الأخطاء عند تغيير البنية.
- **للمستخدمين النهائيين**: إمكانية بناء قوائم ذكية مثل "طلبات تحتوي منتجاتي" أو "طلبات من عملائي (عبر منتجاتي)" مما يحسن تجربة المستخدم في أنظمة السوق متعدد البائعين.
- **المرونة**: الدمج البوليني (`OR`) يسمح بمزج الشروط بسهولة مع نطاقات أخرى (حالة الطلب، نوعه، تاريخه...).
- **قابلية التوسع**: يمكن إضافة خيارات جديدة إلى النطاق الرئيسي بسهولة، أو اشتقاق نطاقات مساعدة إضافية بنفس النمط.

---

### 5. الخاتمة

يمثل هذا التحديث نقلة نوعية في إدارة استعلامات الطلبات المرتبطة بملكية المنتجات ضمن إضافة `Nano.Orders`. من خلال ترايت مستقل ونطاقات متقدمة، أصبح بإمكان المطورين بناء أنظمة فلترة معقدة بسهولة وأداء عالٍ، مع الحفاظ على توافق كامل مع البنية الحالية. التوثيق المقدم يضمن سرعة الاستيعاب والتطبيق.

---

**ملاحظة**: للحصول على تفاصيل تقنية أعمق، راجع ملف `HasProductOwnerScopes.php` والتعليقات المرفقة مع النطاقات.

**الوثائق المرجعية**:
- [Trait HasProductOwnerScopes Documentation](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-ar.md)


