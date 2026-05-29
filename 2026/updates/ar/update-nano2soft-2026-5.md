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

## 2026-03-20 – 2026-05-05

**تحديث إضافة `Nano.AccountsApi` – الإصدار 1.1.0**

### إعادة هيكلة شاملة وإضافة أنواع المستندات المحاسبية المتكاملة

---

### ملخص التحديثات

في الإصدار **1.1.0**، تم تنفيذ قفزة معمارية في إضافة `Nano.AccountsApi`، حيث تم استخراج فئة أساسية مجردة `BaseDocumentController` لإدارة العمليات المشتركة بين مختلف أنواع المستندات المحاسبية. كما تم إضافة دعم كامل لخمسة أنواع جديدة من المستندات المالية (قيود يومية، سندات صرف، إشعارات مدين، إشعارات دائن، تحويلات) مع نماذجها ومحولاتها وإعداداتها الخاصة، بالإضافة إلى نسخة محسّنة من سندات القبض (`CatchReceiptsV2`). وقد تم توسيع ملف الإعدادات والمسارات لدعم هذه الأنواع الجديدة، مع تحسينات متعددة على المتحكمات والمحولات القائمة.

---

### أهداف الإصدار

- **توحيد منطق العمليات** عبر `BaseDocumentController` للتخلص من تكرار الكود وتسهيل إضافة أنواع مستندات جديدة.
- **إضافة أنواع المستندات المحاسبية الشائعة** لتغطية أغلب العمليات المالية اليومية عبر API.
- **توفير إعدادات مرنة وصلاحيات منفصلة** لكل نوع مستند، مدعومة بمتغيرات البيئة (env) لتسهيل التخصيص.
- **تحسين تجربة المطورين** من خلال واجهة برمجة موحدة لعمليات CRUD والتحقق من الصلاحيات.
- **الحفاظ على التوافق العكسي** مع المتحكمات والمحولات القديمة التي لا تزال تعمل كما هي.

---

### الميزات الجديدة والتحسينات

#### 1. فئة أساسية موحدة `BaseDocumentController`

تم إنشاء فئة أساسية مجردة (abstract class) باسم `BaseDocumentController` ترث من `ApiController` وتوفر الوظائف المشتركة التالية:

- **إدارة المستخدم الحالي** والتحقق من صلاحياته (`checkAddPermission`، `checkListPermission`، `checkViewPermission`، `checkUpdatePermission`، `checkDeletePermission`).
- **تجهيز البيانات** عبر دوال مثل `prepareCreateData`، `prepareListOptions`، `validateInput`.
- **تنفيذ العمليات** عبر دوال `performCreate`، `performList`، `performShow`، `performUpdate`، `performDelete` التي تعيد مصفوفة نتائج موحدة.
- **تطبيق المحولات** تلقائياً على مخرجات API.

أصبح بإمكان أي متحكم جديد لمستند مالي أن يرث هذه الفئة ويحدد فقط `$modelClass`، `$transformerClass`، `$configKey`، و`$typeHeader` ليحصل على كامل وظائف CRUD.

#### 2. إضافة أنواع المستندات الجديدة

تمت إضافة خمسة متحكمات جديدة تعتمد على `BaseDocumentController`، لكل منها نموذج (Model) ومحول (Transformer) وإعدادات خاصة:

| المتحكم | النموذج | المحول | النوع (typeHeader) | الوصف |
|---------|--------|--------|-------------------|-------|
| `BondsDays` | `BondsDay` | `BondsDayTransformer` | `BondsDay::class` | قيود يومية متعددة الأطراف مع تحقق من توازن المدين والدائن |
| `PayReceipts` | `PayReceipt` | `PayReceiptTransformer` | `PayReceipt::class` | سندات صرف مع دفع (شيك، نقد) |
| `DebitNotes` | `DebitNote` | `DebitNoteTransformer` | `DebitNote::class` | إشعارات مدين |
| `CreditNotes` | `CreditNote` | `CreditNoteTransformer` | `CreditNote::class` | إشعارات دائن |
| `Transfers` | `Transfer` | `TransferTransformer` | `Transfer::class` | تحويلات بين الحسابات |

كما تم إضافة نسخة محسّنة من سندات القبض:

- `CatchReceiptsV2` (تعتمد على `BaseDocumentController` ونموذج `CatchReceipt`)، مع احتفاظ المتحكم القديم `CatchReceipts` دون تغيير.

**قواعد التحقق الخاصة بكل نوع:**

- **BondsDays:** التحقق من وجود تفاصيل `details` بعدد أطراف ≥ 2، وتوازن المدين والدائن، ومنع المبالغ الصفرية.
- **CatchReceiptsV2 / PayReceipts:** دعم حقول طريقة الدفع (`payment_method`)، رقم الشيك (`cheque_number`)، تاريخه (`cheque_date`)، اسم البنك (`bank_name`)، والمستفيد (`beneficiary`).
- **DebitNotes / CreditNotes:** حقول رقم الفاتورة (`invoice_number`) وسبب الإشعار (`reason`).
- **Transfers:** التحقق من أن الحسابين مختلفان، وحقول رقم المرجع (`reference_number`) والسبب.

#### 3. نظام المحولات (Transformers) الجديد

- تم إنشاء محول `BondsDayTransformer` يرث من `TransactionHeaderTransformer` ويضيف مجموع المدين/الدائن لكل قيد.
- تم إنشاء محولات `CatchReceiptTransformer`، `PayReceiptTransformer`، `DebitNoteTransformer`، `CreditNoteTransformer`، `TransferTransformer` كلها ترث من `TransactionHeaderTransformer` وتضيف حقولها الخاصة (مثل `payment_method`، `cheque_number`...).
- تم تحديث `TransactionHeaderTransformer` ليقبل جميع أنواع النماذج الجديدة عبر union types في توقيع دالة `data()`، مما يسمح باستخدامه كمحول أساسي لأي رأس حركة مالية.

#### 4. توسيع الإعدادات والصلاحيات

تمت إضافة أقسام جديدة في ملف `config.php` لكل نوع مستند، مع دعم متغيرات البيئة (env) للتحكم في:

- **إعدادات التشغيل:** `order_by`، `order_dir`، `transactions_type`، `process_type`، `patterns_id`، `notes`.
- **الصلاحيات المنفصلة:** `is_allow_add`، `is_allow_add_backend`، `is_check_add_permission`، `is_allow_add_frontend` (ومثيلاتها للعرض والتحديث والحذف).
- **إعدادات المحول:** `is_stop_to_array` لكل نوع.

الأقسام المضافة: `bonds_days`، `catch_receipts_v2`، `pay_receipts`، `debit_notes`، `credit_notes`، `transfers`.

#### 5. توسيع مسارات API

أُضيفت مسارات جديدة ضمن مجموعة `oauth-users` في `routes.php`:

```
// قيود يومية
POST  bonds/add → BondsDays@create
POST  bonds/test → BondsDays@test
GET   bonds → BondsDays@index
GET   bonds/{id} → BondsDays@show

// سندات صرف
POST  payreceipts/add → PayReceipts@create
GET   payreceipts → PayReceipts@index
GET   payreceipts/{id} → PayReceipts@show

// إشعارات مدين
POST  debitnotes/add → DebitNotes@create
GET   debitnotes → DebitNotes@index
GET   debitnotes/{id} → DebitNotes@show

// إشعارات دائن
POST  creditnotes/add → CreditNotes@create
GET   creditnotes → CreditNotes@index
GET   creditnotes/{id} → CreditNotes@show

// تحويلات
POST  transfers/add → Transfers@create
GET   transfers → Transfers@index
GET   transfers/{id} → Transfers@show
```

ملاحظة: تم إضافة المسارات كتعليقات احتياطية لعمليات التحديث والحذف (`PUT` و `DELETE`) لتفعيلها مستقبلاً.

#### 6. تحسينات إضافية

- **دالة `show` في `Accounts.php`:** أصبحت تدعم البحث عن الحساب باستخدام `code` بالإضافة إلى `id`، مما يوفر مرونة أكبر للمطورين.
- **دالة `test` في `BondsDays`:** تسمح باختبار القيد اليومي عبر `TransactionsHelper::testJournalEntry` قبل الحفظ، مع صلاحيات خاصة للمستخدمين الخلفيين.
- **`TransactionHeaderTransformer`:** تم توسيعه ليشمل جميع أنواع المستندات المالية، مع الحفاظ على نفس الوظائف الأساسية.

---

### أمثلة على الاستخدام

#### إنشاء قيد يومي متعدد الأطراف

```bash
curl -X POST "https://yourdomain.com/api/v1/accounts/bonds/add" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "date_at": "2026-04-20",
    "notes": "قيد تسوية",
    "details": [
      {"accounts_id": "2-2-1253010001", "debit": 1500},
      {"accounts_id": "2-2-1231010001", "credit": 1000},
      {"accounts_id": "2-2-4111010001", "credit": 500}
    ]
  }'
```

#### جلب قائمة سندات القبض (الإصدار الجديد)

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/catchreceipts" \
  -H "Authorization: Bearer <token>"
```

#### عرض حساب باستخدام الكود

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/accounts/2-2-1231010001" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تسريع التطوير:** يمكن إضافة أي نوع مستند مالي جديد بكتابة بضعة أسطر فقط عبر وراثة `BaseDocumentController`.
- **تقليل الأخطاء:** توحيد منطق التحقق من الصلاحيات وإعداد البيانات يقلل من احتمالية الأخطاء البشرية.
- **مرونة في التخصيص:** كل نوع مستند له إعداداته وصلاحياته المنفصلة، مما يتيح تكييف الإضافة حسب متطلبات كل مشروع.
- **تجربة مستخدم محسنة:** دعم API كامل لجميع العمليات المالية الأساسية، مما يسمح ببناء واجهات أمامية ثرية.
- **حماية معززة:** استخدام نفس نظام الصلاحيات المعمول به في الإضافة الأساسية (`Tss.Accounts`) عبر API.
- **استعداد للمستقبل:** الهيكلية الجديدة تسمح بسهولة إضافة عمليات التحديث والحذف (جاهزة تقنياً، بانتظار تفعيلها).

---

### متطلبات الترقية

1. **تحديث الكود:**
   - استبدال الملفات الأساسية بالإصدارات الجديدة:
     - `APIControllers/BaseDocumentController.php`
     - `APIControllers/BondsDays.php`
     - `APIControllers/PayReceipts.php`
     - `APIControllers/DebitNotes.php`
     - `APIControllers/CreditNotes.php`
     - `APIControllers/Transfers.php`
     - `APIControllers/CatchReceiptsV2.php`
     - `Transformers/BondsDayTransformer.php`
     - `Transformers/CatchReceiptTransformer.php`
     - `Transformers/PayReceiptTransformer.php`
     - `Transformers/DebitNoteTransformer.php`
     - `Transformers/CreditNoteTransformer.php`
     - `Transformers/TransferTransformer.php`
   - تحديث الملفات التالية:
     - `config.php`
     - `routes.php`
     - `TransactionHeaderTransformer.php`
     - `Accounts.php` (تحسين `show`)
2. **لا توجد هجرات قاعدة بيانات جديدة.**
3. **صلاحيات API:**
   - النقاط الجديدة محمية بـ `oauth-users`، لذا يجب أن يمتلك المستخدم توكن صالح، وأن تتوافق صلاحياته مع الإعدادات المحددة لكل نوع.
4. **اختبار الاتصال:**
   - تأكد من أن المتغيرات البيئية (env) للإعدادات الجديدة مضبوطة بشكل صحيح (خاصة `is_allow_add_backend` ونوع الحركة `transactions_type`).
   - جرب إنشاء قيد يومي بسيط باستخدام `bonds/add` وتحقق من الاستجابة.
5. **مسح الكاش:**
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

يمثل الإصدار **1.1.0** نقلة نوعية في إضافة `Nano.AccountsApi`، حيث انتقلت من مجرد مجموعة من المتحكمات الفردية إلى إطار عمل مصغر لإدارة المستندات المالية عبر API. بفضل `BaseDocumentController`، أصبحت إضافة أنواع جديدة عملية سريعة وآمنة، مما يفتح الباب لتوسعات مستقبلية دون تعقيد. التغطية الشاملة لأنواع القيود والسندات والإشعارات والتحويلات تجعل الإضافة مناسبة لمجموعة واسعة من التطبيقات المحاسبية، مع الحفاظ على التوافق مع الإصدارات السابقة.


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

## 2026-05-06 - 2026-05-07

**إضافة `Nano.StudyyearApi` – الإصدار 1.0.0**

### إنشاء واجهة برمجة تطبيقات RESTful لإدارة السنوات والفصول والأشهر الدراسية

---

### ملخص التحديثات

يقدم الإصدار الأول **1.0.0** من إضافة `Nano.StudyyearApi` واجهة برمجة تطبيقات كاملة وموحّدة للتفاعل مع البيانات المُدارة بواسطة إضافة `Tss.Studyyear`. تم بناء هذه الإضافة وفقاً لنمط `Nano.*Api` المعتمد في المشروع، مع توفير نقاط نهاية (endpoints) واضحة ومحمية لاسترجاع السنوات الدراسية (`Periods`) والفصول الدراسية (`Semsters`) والأشهر الدراسية (`Months`). يشمل هذا الإصدار أيضاً نظاماً شاملاً للإعدادات والصلاحيات متعدد المستويات، ومحولات بيانات (Transformers) احترافية، ودعماً كاملاً للترجمة.

---

### أهداف الإصدار

- **توفير API موحد** لجميع كيانات العام الدراسي (السنوات، الفصول، الأشهر).
- **تطبيق نمط معماري متناسق** مع باقي إضافات `Nano.*Api` (مثل `Nano.StudentsApi` و `Nano.AbsenceApi`).
- **تمكين التطبيقات الخارجية والواجهات الأمامية** من الوصول إلى هذه البيانات الحيوية بسهولة وأمان.
- **توفير مرونة عالية في الإعدادات والصلاحيات** لكل مورد (Resource) على حدة.
- **دعم التصفية المتقدمة والبحث** في جميع نقاط النهاية.
- **تسهيل التوسع المستقبلي** لإضافة عمليات الكتابة (إنشاء، تحديث، حذف) دون تغيير جذري في الهيكلية.

---

### الميزات الجديدة والتحسينات

#### 1. وحدات تحكم RESTful كاملة

تم إنشاء ثلاث وحدات تحكم رئيسية (Controllers) تتبع نمط `ApiController` الموحد:

| المتحكم | النموذج المستهدف | الوصف |
|---------|-----------------|-------|
| `Periods` | `Tss\Studyyear\Models\Period` | إدارة السنوات الدراسية |
| `Semsters` | `Tss\Studyyear\Models\Semster` | إدارة الفصول الدراسية (الترم الأول والثاني) |
| `Months` | `Tss\Studyyear\Models\Month` | إدارة الأشهر الدراسية |

**كل متحكم يقدم:**

- **`index()`**: جلب قائمة بالسجلات مع دعم كامل للتصفية والبحث والترتيب والتقسيم إلى صفحات.
- **`show($id)`**: عرض تفاصيل سجل واحد مع كافة العلاقات المرتبطة.
- **`getRecords(array $options)`**: دالة أساسية قابلة للتخصيص تُستخدم داخلياً لبناء استعلامات معقدة.
- **`activelystats()`**: نقطة نهاية لمراقبة آخر تحديث للبيانات (لأغراض التخزين المؤقت).
- **`getLastUpdateAt()`**: إرجاع الطابع الزمني لآخر تحديث.
- **`validationList($user)`**: دالة موحدة للتحقق من صلاحيات الوصول حسب نوع المستخدم (خلفي / أمامي) والإعدادات.

#### 2. نظام صلاحيات محكم وقابل للتخصيص

تم تطبيق نظام صلاحيات متعدد المستويات يسمح للمسؤولين بالتحكم الدقيق في إمكانية الوصول إلى كل نقطة نهاية:

- **على مستوى المستخدم العام**: هل يُسمح بعملية `list` من الأساس؟
- **على مستوى نوع المستخدم**: هل يُسمح لمستخدمي الخلفية (`backend`)؟ هل يُسمح لمستخدمي الواجهة الأمامية (`frontend`)؟
- **على مستوى صلاحيات محددة**: هل يشترط أن يمتلك المستخدم صلاحية معينة (مثل `tss.studyyear.periods.access`)؟

يتم التحكم في هذه الإعدادات عبر ملف `config.php` باستخدام متغيرات البيئة (`env`) مما يسمح بتغيير السلوك دون لمس الكود.

**مثال على إعدادات `periods` في `config.php`:**
```php
'periods' => [
    'is_allow_list'            => env('NANO_STUDYYEARAPI_PERIODS_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_STUDYYEARAPI_PERIODS_IS_ALLOW_LIST_BACKEND', true),
    'is_allow_list_frontend'   => env('NANO_STUDYYEARAPI_PERIODS_IS_ALLOW_LIST_FRONTEND', true),
    'is_check_access_list'     => env('NANO_STUDYYEARAPI_PERIODS_IS_CHECK_ACCESS_LIST', true),
    'is_check_list_permission' => env('NANO_STUDYYEARAPI_PERIODS_IS_CHECK_LIST_PERMISSION', true),
    // ...
],
```

#### 3. محولات بيانات متكاملة (Transformers)

تم إنشاء ثلاثة محولات بيانات لتنسيق استجابات API بشكل احترافي:

- **`PeriodTransformer`**: يعرض حقول السنة الدراسية مع تضمين العلاقات (`company`, `department`).
- **`SemsterTransformer`**: يعرض حقول الفصل الدراسي مع تضمين العلاقات (`company`, `department`, `period`).
- **`MonthTransformer`**: يعرض حقول الشهر الدراسي مع تضمين العلاقات (`company`, `department`, `period`, `semster`).

كل المحولات تدعم:
- **تنسيق البيانات**: تحويل القيم إلى الأنواع الصحيحة (int, string, bool, float, date).
- **دالة `formatDate`**: لتنسيق التواريخ بشكل آمن.
- **تصفية الحقول (`exclude`)**: يمكن للمستخدم تحديد أسماء الحقول التي لا يرغب في استلامها لتقليل حجم البيانات المنقولة.
- **`object_type`**: إضافة نوع الكائن إلى الاستجابة.
- **معالجة الأخطاء**: جميع دوال `include` محاطة بكتل `try-catch` لتجنب فشل الاستعلام بالكامل بسبب علاقة مفقودة.

#### 4. دعم ذكي للقيم الافتراضية

في حالة عدم تمرير معرف السنة الدراسية (`study_year_id`) عند جلب الفصول (`Semsters`) أو الأشهر (`Months`)، تقوم المتحكمات تلقائياً باعتماد **السنة الدراسية الافتراضية** (`Period::getPrimary()`) كمرشح، مما يضمن تجربة سلسة للمطورين دون الحاجة لتحديد السنة صراحةً في كل طلب.

**مثال تطبيقي في `Semsters::getRecords()`:**
```php
if (!$study_year_id) {
    $primary = Period::getPrimary();
    if ($primary) {
        $study_year_id = $primary->id;
    }
}
```

#### 5. دعم التخزين المؤقت (Caching)

تدعم جميع نقاط النهاية `index` التخزين المؤقت عبر إعدادات `nano.api::api_enable_cache`، مما يحسن الأداء بشكل كبير عند تكرار الطلبات. يتم إبطال الكاش تلقائياً عند تحديث أي سجل عبر دالة `getLastUpdateAt()`.

#### 6. تصفية وبحث متقدم

يمكن تمرير العديد من عوامل التصفية (filters) إلى `getRecords` مثل:
- `companys_id` و `departments_id` (مع دعم نطاق `IsCompany` تلقائياً).
- `isActive`، `calendar_type`، `ref_type`، `status`.
- `study_year_id` و `semsters_id` و `month_num` (حسب المورد).
- البحث النصي `q` عبر الاسم والوصف المختصر والكود.

#### 7. رسائل خطأ متعددة اللغات

تم تضمين ملف ترجمة كامل (`lang/en/lang.php`) يحتوي على جميع رسائل النجاح والخطأ والصلاحيات، مما يسهل تعريبها أو ترجمتها إلى أي لغة أخرى.

---

### أمثلة على الاستخدام

#### جلب قائمة السنوات الدراسية النشطة

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/periods?isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### جلب فصول السنة الدراسية الافتراضية مع تضمين السنة والمؤسسة

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/semsters?include=period,company" \
  -H "Authorization: Bearer <token>"
```

#### جلب أشهر فصل دراسي محدد مع ترتيب حسب الشهر

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/months?semsters_id=5&orderBy=month_num&orderDirection=asc" \
  -H "Authorization: Bearer <token>"
```

#### عرض تفاصيل سنة دراسية محددة

```bash
curl -X GET "https://yourdomain.com/api/v1/studyyear/periods/12" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تكامل سلس مع واجهات المستخدم**: يمكن لأي تطبيق Frontend (React, Vue, Angular) جلب بيانات السنوات والفصول والأشهر بسهولة لبناء تقاويم أكاديمية أو نماذج تسجيل.
- **تقليل الحمل على المطورين**: بدلاً من كتابة استعلامات معقدة يدوياً، توفر الإضافة واجهة استعلام غنية ومباشرة.
- **جاهزية للإنتاج**: نظام الصلاحيات والإعدادات المتقدم يسمح بنشر الإضافة مباشرة في بيئات إنتاجية مع تحكم كامل في الوصول.
- **توسع مستقبلي بسيط**: إضافة عمليات `POST` و `PUT` و `DELETE` مستقبلاً لن يتطلب سوى توسيع المتحكمات الحالية دون إعادة هيكلة.
- **أداء عالي**: استخدام `scopeExclude` للتحكم في الأعمدة المسترجعة والتخزين المؤقت يضمن أوقات استجابة سريعة.

---

### متطلبات الترقية

1. **تثبيت الإضافة**: قم بنسخ مجلد `nano/studyyearapi` إلى مسار `plugins/nano/studyyearapi`.
2. **تسجيل الإضافة**: تأكد من تشغيل `php artisan plugin:refresh Nano.StudyyearApi`.
3. **المتغيرات البيئية**: قم بضبط متغيرات `env` حسب الحاجة (مثل تفعيل أو تعطيل الوصول لمستخدمي الواجهة الأمامية).
4. **الصلاحيات**: تأكد من أن الأدوار والمستخدمين في النظام الخلفي يمتلكون الصلاحيات المناسبة (مثلاً `tss.studyyear.periods.access`) إذا تم تفعيل خيار `is_check_list_permission`.
5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **لا توجد هجرات قاعدة بيانات** مطلوبة لهذا الإصدار.

---

### الخاتمة

يمثل الإصدار **1.0.0** من `Nano.StudyyearApi` حجر الأساس للتعامل مع بيانات التقويم الدراسي عبر واجهة برمجة تطبيقات حديثة وآمنة. باتباع أفضل الممارسات والنمط المعماري الموحد لإضافات `Nano.*Api`، تقدم هذه الإضافة أساساً متيناً يمكن البناء عليه لإضافة المزيد من الميزات مثل إنشاء وتعديل السنوات الدراسية، ودمجها مع أنظمة الحضور والغياب أو الجداول الدراسية.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano.StudyyearApi`](./docs/StudyyearApi/Docs-StudyyearApi-ar.md)
- [توثيق إضافة `Nano.StudentsApi`](./docs/StudentsApi/Docs-StudentsApi-ar.md)

## 2026-05-02 – 2026-05-08

**تحديث إضافة `Nano.Coupons` – الإصدار 2.1.1 – تحسينات شاملة على شرط "حد المنتج"**

---

### ملخص التحديثات

يقدم الإصدار **2.1.1** تحسينات جوهرية على ميزة "حد المنتج" التي تم إطلاقها في الإصدار 2.1.0. تم إعادة هيكلة منطق التحقق بالكامل عبر إضافة كلاس جديد ومستقل `ProductLimitValidator`، مما يوفر مرونة غير مسبوقة في التعامل مع قيود المنتج. يشمل التحديث دعم سيناريوهات متقدمة مثل تقييد عدد الطلبات لأنواع طلب محددة (بغض النظر عن المنتج)، ورسائل خطأ تفصيلية متعددة اللغات تُظهر متى يمكن للمستخدم إعادة الطلب، وتحسينات كبيرة في الأداء عبر التخزين المؤقت ومنع الاستعلامات المتكررة.

---

### 1. أهداف الإصدار

- **فصل منطق التحقق** عن شرط السلة إلى كلاس مستقل `ProductLimitValidator` لتسهيل إعادة الاستخدام والصيانة.
- **دعم سيناريوهات جديدة**: تحديد عدد الطلبات المسموحة لنوع طلب معين خلال فترة، بدون الحاجة إلى تقييد منتج محدد.
- **تحسين تجربة المستخدم** عبر رسائل خطأ واضحة ومفصلة تشمل الحد الأقصى، العدد الحالي، الفترة، وتاريخ إمكانية الطلب التالي.
- **تحسين كبير في الأداء** من خلال التخزين المؤقت للقيود (`Cache`) ومنع تكرار استعلامات قاعدة البيانات المكلفة.
- **توسيع واجهة الإدارة** لتشمل البحث عن المنتج (`recordfinder`) ودعم اختيار الوحدة للمنتج المحدد.
- **إضافة نطاقات جديدة** `scopeOnProductLimit` و `scopeNotOnProductLimit` في نموذج الكوبون.

---

### 2. المكونات المطورة

#### 2.1 كلاس `ProductLimitValidator` (جديد)

الموقع: `plugins/nano/coupons/classes/ProductLimitValidator.php`

كلاس مستقل ومتخصص في فحص قيود حد المنتج. يقبل المستخدم وخيارات التهيئة (`use_cache`, `cache_ttl`) ويوفر دوال عامة شاملة:

| الدالة | الوصف |
|--------|-------|
| `checkOrFail(...)` | فحص شامل يرمي استثناءً مع رسالة خطأ مفصلة عند تجاوز الحدود. |
| `check(...)` | فحص صامت يعيد `true/false`. |
| `getLimits(...)` | جلب القيود المطبقة مع استراتيجيات متعددة (`auto`, `cache`, `fresh_db`, `direct_query`). |
| `getLimitsForProduct(...)` | اختصار لجلب قيود منتج محدد. |
| `checkOrderTypeLimitOrFail(...)` | فحص عدد طلبات نوع طلب معين. |
| `getOrderTypeLimits(...)` | جلب القيود الخاصة بنوع طلب معين. |
| `clearCache()` | مسح الكاش يدوياً. |
| `setUser(User $user)` | تعيين المستخدم. |

**استراتيجيات الجلب المدعومة:**
- `auto`: استخدام الكاش تلقائياً إذا كان مفعلاً.
- `cache`: فرض تحميل القيود من الكاش.
- `fresh_db`: تجاهل الكاش وتحميل مباشر من قاعدة البيانات.
- `direct_query`: تنفيذ استعلام مباشر على جدول الكوبونات بالشروط المحددة.

**أبرز التحسينات على المنطق:**

- **الحالة الأولى (منتج محدد):**
  - إذا كان `max_quantity > 0` يتم فحص الكمية الإجمالية (العربة + التاريخية + المضافة حديثاً).
  - إذا كان `max_orders > 0` يتم فحص عدد الطلبات التي تحتوي على هذا المنتج (أو وحدته).
  - احترام `units_id` و `order_restriction` و `shop_restriction` و `period_days`.

- **الحالة الثانية (بدون منتج – تقييد عدد الطلبات فقط):**
  - إذا كان `product_id` فارغاً، يتم تطبيق فحص `max_orders` بناءً على `order_types` و `shop_ids` و `period_days`.

#### 2.2 تحديثات شرط `ProductLimit` في السلة

الموقع: `plugins/nano/coupons/cartconditions/ProductLimit.php`

- تم تبسيط الكلاس ليكون وسيطاً بين نظام السلة (`CartCondition`) و `ProductLimitValidator`.
- دالة `validateCartItem` أصبحت تستخدم `ProductLimitValidator` مباشرة.
- تم إيقاف العمل بـ `beforeApply` مؤقتاً لمنع أي عبء إضافي أثناء حساب إجمالي السلة.
- الاعتماد على الكاش عبر `Cache::remember` في `loadAllProductLimits` مع تخزين `is_expired` لتجنب مشاكل serialization.

#### 2.3 تحديثات نموذج `Coupon` و `Coupons_model`

- إضافة خصائص الكاست `'product_id' => 'integer'`, `'units_id' => 'integer'`, `'max_quantity' => 'integer'`, `'max_orders' => 'integer'`, `'period_days' => 'integer'`.
- إضافة دالة `getProductIdOptions` لجلب قائمة المنتجات النشطة.
- توسيع دالة `getUnitsIdOptions` لجلب وحدات منتج محدد بشكل ديناميكي.
- إضافة النطاقات `scopeOnProductLimit` و `scopeNotOnProductLimit` لفلترة الكوبونات.
- في `afterSave` يتم مسح الكاش `nano.coupons.product_limits` عند تعديل/إضافة كوبون `product_limit`.

#### 2.4 تحديثات واجهة الإدارة (`fields.yaml`)

- تغيير `product_id` من `type: dropdown` إلى `type: recordfinder` للبحث المتقدم عن المنتجات.
- إضافة دعم الترجمة لخصائص `placeholder`, `emptyOption`, `title`, `prompt`.
- تحديث `units_id` ليعمل بـ `dependsOn: [product_id]` لتحديث قائمة الوحدات ديناميكياً.
- إظهار حقول `product_id`, `units_id`, `max_quantity`, `max_orders`, `period_days` فقط عند اختيار `apply_coupon_on = product_limit` عبر `trigger`.

#### 2.5 ملفات الترجمة (`lang/ar/lang.php`)

- مفاتيح جديدة لرسائل حد المنتج مع بارامترات:
  - `max_quantity_exceeded` مع `:max` و `:count`.
  - `max_orders_exceeded` مع `:max`, `:count`, `:days`, `:next_date`.
  - `unlimited` وعناصر تسميات الحقول الجديدة.

#### 2.6 ملف التهيئة (`config.php`)

- إضافة المفتاح `is_support_product_limit` مع القيمة الافتراضية `false` (للتحكم في إظهار النوع في القوائم).

#### 2.7 تحديثات `Plugin.php` (Nano.Coupons)

- تسجيل شرط `ProductLimit` في `registerCartConditions` فقط عند وجود الكلاس.
- تعديل `bindCouponsEvent` لإضافة `notOnProductLimit()` لتجنب تطبيق الكوبونات التلقائية على سلة فيها قيد منتج.

---

### 3. آلية العمل (التدفق الجديد)

1. **الإعداد (مرة واحدة):** ينشئ المسؤول كوبون `product_limit`، ويختار إما منتج معين (ووحدة اختيارية) أو يتركه فارغاً لتقييد عدد الطلبات حسب نوع الطلب.
2. **تحميل القيود:** عند تحميل العربة، يتم تحميل القيود من الكاش (أو قاعدة البيانات) مرة واحدة.
3. **فحص فوري عند إضافة منتج:** `ProductLimit` يستدعي `ProductLimitValidator::checkOrFail` باستخدام الحدث `nano.cart.adding`.
4. **فحص الكمية / عدد الطلبات:**
   - للمنتج: الكمية الإجمالية + الكمية الجديدة ≤ `max_quantity`.
   - للمنتج أو نوع الطلب: عدد الطلبات السابقة < `max_orders`.
5. **رسالة خطأ مفصلة:** "تجاوزت الحد المسموح لعدد الطلبات (الحد الأقصى 3). عدد الطلبات الحالي: 3. الفترة: 7 أيام. يمكنك الطلب مرة أخرى بعد 2026-05-10."

---

### 4. أبرز التحسينات والإنجازات

- **أداء ممتاز:** استخدام `Cache` للقيود مع إمكانية مسحه تلقائياً عند تعديل الكوبون.
- **استعلامات مجمعة:** دوال `getBulkHistoricalQuantities` و `getBulkHistoricalOrderCounts` تجمع البيانات في استعلام واحد.
- **مرونة عالية:**
  - فحص منتج معين أو نوع طلب معين أو كليهما.
  - فحص كمية أو عدد طلبات أو كليهما.
  - إمكانية تحديد `unit_id` اختيارياً.
- **رسائل خطأ احترافية:** تعرض الحد الأقصى، الاستخدام الحالي، الفترة، وتاريخ السماح بإعادة الطلب.
- **دعم كامل للترجمة** (عربي/إنجليزي).
- **فصل الاهتمامات:** `ProductLimitValidator` قابل لإعادة الاستخدام في أي مكان (API، CLI، الخ) بدون الاعتماد على نظام السلة.

---

### 5. المتطلبات والترقية

- **تحديث الملفات:**
  - `plugins/nano/coupons/classes/ProductLimitValidator.php` (جديد)
  - `plugins/nano/coupons/cartconditions/ProductLimit.php`
  - `plugins/nano/coupons/models/Coupon.php`
  - `plugins/nano/coupons/models/Coupons_model.php`
  - `plugins/nano/coupons/models/coupon/fields.yaml`
  - `plugins/nano/coupons/lang/ar/lang.php`
  - `plugins/nano/coupons/Plugin.php`
  - `plugins/nano/coupons/config/config.php`
- **تشغيل الهجرة:** لا توجد هجرات جديدة في هذا الإصدار (الأعمدة أضيفت في 2.1.0).
- **مسح الكاش:** يوصى بتنفيذ `php artisan cache:clear` بعد الترقية لضمان تحميل ملفات الترجمة الجديدة.

---

### 6. ملاحظات إضافية

- يمكن تفعيل/تعطيل ميزة `product_limit` من ملف البيئة `.env` عبر المفتاح `NANO_COUPONS_IS_SUPPORT_PRODUCT_LIMIT`.
- للحصول على أداء أفضل مع عدد كبير من الكوبونات، يُنصح بضبط `cache_ttl` إلى مدة مناسبة (الافتراضي 300 ثانية).
- جميع الاستعلامات الآمنة ضد SQL Injection وتستخدم ربط المعاملات (Parameter Binding).

---

### 7. خطط التطوير المستقبلية

- دعم تحديد عدة منتجات في كوبون `product_limit` واحد.
- تطوير واجهة تقارير لمتابعة استهلاك العملاء لحدود المنتجات.
- إضافة القدرة على تعطيل القيد لفئات عملاء محددة.

---

### 8. الخاتمة

يُكمل الإصدار 2.1.1 بناء نظام "حد المنتج" ويجعله أكثر قوة ومرونة واحترافية. بفضل `ProductLimitValidator` المنفصل، يمكن للمطورين دمج فحوصات الحدود في أي مكان بسهولة، بينما تضمن تحسينات الأداء تجربة سلسة للمستخدم النهائي حتى مع وجود عدد كبير من القيود النشطة.


**الوثائق المرجعية**:

- [توثيق كلاس `ProductLimitValidator`](./docs/Coupons/Classes/Docs-ProductLimitValidator-Class-ar.md)
- [توثيق شرط السلة `ProductLimit`](./docs/Coupons/CartConditions/Docs-ProductLimit-CartCondition-ar.md)
- [توثيق شرط السلة `Coupon`](./docs/Coupons/CartConditions/Docs-Coupon-CartCondition-ar.md)
- [توثيق شرط السلة `AutoCoupon`](./docs/Coupons/CartConditions/Docs-AutoCoupon-CartCondition-ar.md)
- [توثيق شرط السلة `AutoCouponShipping`](./docs/Coupons/CartConditions/Docs-AutoCouponShipping-CartCondition-ar.md)

## 2026-05-06 – 2026-05-08

**تحديثات إضافة `Nano.Orders` – الإصدار 2.2.10**  
**تحديثات إضافة `Nano.OrdersApi` – الإصدار 1.0.20**

### ملخص التحديثات

شهدت إضافة `Nano.Orders` تحديثاً محورياً في الإصدار **2.2.10** ركز على تمكين الفلترة المتقدمة للطلبات بناءً على مالك المنتج، وتطوير آلية احترافية لتحديث حالة الطلب بصلاحيات متدرجة وأدوار متعددة، إلى جانب تحسينات في `OrderHelper` لإدارة المستخدمين والموصلين.  
في المقابل، جاء الإصدار **1.0.20** من `Nano.OrdersApi` ليُتيح واجهة API موحّدة لتحديث حالة الطلب، مستفيداً من الدوال الجديدة في `OrderManager`، مع دعم كامل للخيارات المخصصة والصلاحيات.

---

## Nano.Orders v2.2.10 – فلترة مالك المنتج وتحديث حالة الطلب بصلاحيات متدرجة

### أهداف الإصدار

- **توفير نطاقات استعلام متقدمة** لفلترة الطلبات بناءً على ملكية المنتجات المرتبطة.
- **إنشاء دالة احترافية متكاملة** لتحديث حالة الطلب بأدوار مختلفة (مدير، مالك، موصل) مع قواعد انتقال مرنة وصلاحيات قابلة للتخصيص.
- **إضافة دوال مساعدة** لاستخراج الموصل من المستخدم والعكس.
- **دمج فلترة مالك المنتج** داخل `OrderHelper::getOrdersRecords` لتقارير أكثر ذكاءً.
- **دعم الإعدادات الخارجية** (`config`) للتحكم بقواعد الانتقال والصلاحيات.

### الميزات الجديدة

#### 1. ترايت `HasProductOwnerScopes` في نموذج `Order`

تم إنشاء الترايت في المسار `Nano\Orders\Models\Order\HasProductOwnerScopes` ويضم نطاقات ودوالاً متقدمة لفلترة الطلبات حسب مالك المنتج:

| النطاق / الدالة | الوصف |
|----------------|--------|
| `scopeWhereHasProductsByOwner` | نطاق رئيسي بمصفوفة خيارات شاملة: `user`, `count`, `not`, `boolean`, `conditions`, `itemConditions`. |
| `scopeWhereProductOwner` | اختصار للاستخدام السريع (وجود منتج واحد على الأقل). |
| `scopeHasProductsByOwner` | فلترة حسب عدد أدنى من المنتجات المملوكة. |
| `scopeDoesntHaveProductsByOwner` | فلترة الطلبات التي لا تحتوي أي منتج مملوك. |
| `scopeHasProductsByOwnerCount` | تحديد عدد دقيق (أو بمقارنة) للمنتجات المملوكة. |
| `scopeOrWhereHasProductsByOwner` | ربط النطاق مع `OR`. |
| `scopeWithProductsByOwnerCount` | إضافة عمود محسوب بعدد المنتجات المملوكة. |
| `scopeSortByProductsByOwnerCount` | ترتيب الطلبات حسب عدد المنتجات المملوكة. |
| `hasAnyProductByOwner` | دالة فحص على كائن الطلب الواحد. |
| `applyOwnerToProductQuery` | دالة مساعدة داخلية لتطبيق شرط المالك على استعلام منتج. |

جميع النطاقات تستخدم ثوابت أسماء الجداول (`Product::TABLE_NAME`) لضمان توافق دائم.

**مثال استخدام:**
```php
// طلبات تحتوي منتجين مملوكين للمستخدم الحالي
$orders = Order::whereHasProductsByOwner([
    'count' => 2,
    'countOperator' => '>='
])->get();
```

#### 2. دالة `updateOrderStatusAdvanced` في `StepStatus` ترايت

أُضيفت دالة متكاملة تُلبي جميع متطلبات تحديث حالة الطلب مع صلاحيات متدرجة:

| الميزة | الوصف |
|--------|--------|
| **تحديد الفاعل تلقائياً** | يستخدم المستخدم المسجّل دخوله (مدير باكند، مالك، موصل). |
| **ثلاث حقول للحالة** | `order_states_ref_type` (للإدارة)، `user_status` (للمالك)، `delivery_status` (للموصل). |
| **قواعد انتقال قابلة للتخصيص** | افتراضية + من الإعدادات + خاصة بالدور + مخصصة لكل استدعاء. |
| **منع تعديل المكتمل** | لا يُسمح بتغيير حالة طلب مكتمل (إلا بتجاوز إداري). |
| **مزامنة تلقائية** | عند تساوي `user_status` مع `delivery_status` تُحدّث الحالة الرئيسية. |
| **أسباب الإلغاء** | `because_cancel` (مالك) و `delivery_because_cancel` (موصل). |
| **خيارات تحكم شاملة** | `is_save`, `is_event`, `is_logs`, `skip_permission`, `admin_override`, `custom_message`. |
| **هيكل استجابة موحد** | يحتوي على `input_data`, `process_data`, `model`, `debug`. |

**أمثلة أدوار:**
- **المدير**: يمكنه تغيير أي حقل، مع إمكانية تجاوز قيود الانتقال (`admin_override`).
- **المالك**: يغير `user_status` فقط، ويمكنه إلغاء الطلب مع سبب.
- **الموصل**: يغير `delivery_status`، ويمكنه فقط تحريك `user_status` من `NEW` إلى `DELIVERY`.

#### 3. دوال مساعدة في `OrderHelper`

- `getDeliveryByUser($user)` – استخراج كائن الموصل من المستخدم.
- `getUserByDelivery($delivery)` – استخراج كائن المستخدم من الموصل.

#### 4. دعم فلترة مالك المنتج في `OrderHelper::getOrdersRecords`

أُضيفت الخيارات التالية:
- `is_has_products_by_owner` (تفعيل الفلتر)
- `products_by_owner_*` (جميع خيارات النطاق الرئيسي)
- `is_has_products_by_owner_or_delivery` لدمج الفلتر مع شروط التوصيل بـ `OR`.

#### 5. الإعدادات الجديدة في `config.php`

```php
'nano.orders::manager.edit_status.allowed_transitions'
'nano.orders::manager.edit_status.admin.allowed_transitions'
'nano.orders::manager.edit_status.user.allowed_transitions'
'nano.orders::manager.edit_status.delivery.allowed_transitions'
```

---

## Nano.OrdersApi v1.0.20 – نقطة API لتحديث حالة الطلب

### أهداف الإصدار

- **توفير واجهة API موحدة** لتحديث حالة الطلب من قبل المالك، الموصل، أو المدير.
- **دمج كامل مع `updateOrderStatusAdvanced`** لضمان اتساق الصلاحيات والمنطق.
- **دعم الخيارات المخصصة** عبر الطلب JSON.
- **الحفاظ على هيكل استجابة متسق** مع باقي API.

### الميزات الجديدة

#### 1. نقطة النهاية `POST /api/v1/orders/orders/update-status`

تستقبل الطلب JSON بالحقول التالية (جميعها اختيارية وتعتمد على الدور):
- `order_id` أو `id`
- `order_states_ref_type`
- `user_status`
- `delivery_status`
- `because_cancel`
- `delivery_because_cancel`
- `is_save`, `is_event`, `is_logs`
- `skip_permission`, `admin_override`
- `custom_message`, `custom_error`

#### 2. دالة `updateStatus` في المتحكم `Orders`

تستخرج الطلب، تجهز الخيارات، وتستدعي `OrderManager::updateOrderStatusAdvanced()` مع تمرير جميع المُعامِلات، وتُعيد استجابة موحدة تحتوي على الحالة الجديدة.

**مثال استجابة ناجحة:**
```json
{
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "طلب مكرر"
    }
}
```

---

## ملخص الإصدارات (2.2.10 و 1.0.20)

| الإصدار | أبرز الميزات |
|---------|---------------|
| **Nano.Orders 2.2.10** | `HasProductOwnerScopes` (نطاقات فلترة مالك المنتج)، `updateOrderStatusAdvanced` (تحديث حالة بصلاحيات متدرجة)، دوال `getDeliveryByUser` / `getUserByDelivery`، دعم فلترة المالك في `OrderHelper::getOrdersRecords`، إعدادات مرنة لقواعد الانتقال. |
| **Nano.OrdersApi 1.0.20** | نقطة API `POST orders/update-status`، دمج مع الدالة الاحترافية، دعم جميع الخيارات المخصصة، استجابة موحدة. |

---

### متطلبات الترقية

1. **تحديث الملفات**:
   - إضافة ترايت `HasProductOwnerScopes` في `models/Order/HasProductOwnerScopes.php`.
   - إضافة ترايت `StepStatus` في `traits/steps/StepStatus.php`.
   - تحديث `OrderHelper.php` بالدوال الجديدة ودعم فلترة المالك.
   - تحديث `OrderManager.php` لاستخدام `StepStatus`.
   - تحديث `Orders.php` في `OrdersApi` لإضافة دالة `updateStatus`.
   - تحديث `routes.php` في `OrdersApi` لإضافة المسار الجديد.

2. **لا توجد هجرات جديدة**: الإصداران لا يتطلبان تغييرات في قاعدة البيانات.

3. **ملفات الترجمة**:
   - أضف ملف `lang/ar/manager/update_status.php` في `Nano.Orders` بالمفاتيح المطلوبة.

4. **الإعدادات الاختيارية**:
   - يمكن إضافة قواعد انتقال مخصصة في `config.php` تحت `nano.orders::manager.edit_status`.

5. **اختبار التوافق**:
   - اختبار النطاقات الجديدة مع استعلامات مختلفة.
   - اختبار تحديث الحالة من الأدوار الثلاثة.
   - اختبار API من تطبيقات الموبايل (باستخدام توكن المستخدم المناسب).

---

### الخاتمة

يمثل الإصداران **Nano.Orders 2.2.10** و **Nano.OrdersApi 1.0.20** نقلة نوعية في مرونة نظام الطلبات. فمن جهة، أتاحت نطاقات `HasProductOwnerScopes` فلترة غير مسبوقة للطلبات بناءً على ملكية المنتجات، مما يلبي احتياجات الأسواق متعددة البائعين. ومن جهة أخرى، قدمت دالة `updateOrderStatusAdvanced` ونقطة API المدمجة معها نظاماً موحداً وآمناً لتحديث حالة الطلب من قبل جميع الأطراف (مدير، مالك، موصل) مع إعدادات قابلة للتخصيص بالكامل. الكود نظيف، موثّق، وقابل للتوسع بسهولة.

---

**الوثائق المرجعية**:
- [توثيق `OrderManager` وسماته](./docs/Orders/Classes/Docs-OrderManager-Class-ar.md)
- [توثيق السمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-ar.md)
- [توثيق متقدم للسمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-ar.md)
- [توثيق نطاقات `HasProductOwnerScopes`](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-ar.md)
- [توثيق API الطلبات](./docs/OrdersApi/Docs-OrdersApi-ar.md)

## 2026-05-04 – 2026-05-10

**تحديثات بوابة الدفع JaibPay (محفظة جيب) – مايو 2026**

### تحسينات شاملة وإصلاحات على بوابة دفع JaibPay

ضمن الوحدة البرمجية `Nano.Yepayment`

---

### 1. مقدمة

بعد الانتهاء من الإصدار الأولي لبوابة **JaibPay** (محفظة جيب)، تم تنفيذ سلسلة من التحسينات والإصلاحات الجوهرية لضمان عمل البوابة بشكل احترافي ومتكامل مع نظام نانوسوفت. شملت التحديثات إصلاح دالة المصادقة، تحسين آليات معالجة استجابات API، إضافة دوال اختبار المنفذ ومعالجة الاستجابات، إنشاء دالة استرداد المبالغ، وحل مشكلة حفظ الإعدادات المشفرة في `PaymentGatewaySettings`.

تم اختبار البوابة بشكل فعلي مع منصة Jaib Pay عبر عدة سيناريوهات دفع واسترداد مختلفة، وأثبتت جميعها نجاح التكامل.

---

### 2. المكونات المطورة والمحدثة

#### 2.1. `JaibPay` – كلاس بوابة الدفع الرئيسي

**الدوال الجديدة والمحسّنة:**

| الدالة | الوصف | النوع |
|--------|-------|-------|
| `normalizeApiResponse($response): array` | معالجة استجابة API من Jaib وتحويلها إلى هيكل موحد يحتوي على `success`, `status_code`, `error_code`, `error_message`, `data`, `raw`. تتعامل مع نجاح HTTP 200، أخطاء HTTP 500، وأخطاء JSON. | **جديدة** |
| `looksLikeJson($string): bool` | دالة مساعدة للتحقق مما إذا كان النص يشبه JSON. | **جديدة** |
| `getSettingsByKey($key, $defaultValue = null)` | جلب قيمة إعداد من `PaymentGatewaySettings` بطريقة صحيحة، مع دعم الحقول المشفرة عبر `getEncryptableValue()`. | **جديدة** |
| `testPortConnection(string $url, ?int $port, int $timeout): array` | اختبار ما إذا كان منفذ معين مفتوحاً على خادم. تستخرج المنفذ تلقائياً من الرابط، وتدعم تحليل DNS وقياس زمن الاستجابة. | **جديدة** |
| `refund(string $referenceID, ...): array` | استرداد مبلغ معاملة عبر `POST /api/v1/BuyOnline/RefoundBuy`. تقوم بتغيير حالة الطلب إلى `RefundedState` وتخزين بيانات الاسترداد في `other_data['jaibpay_refund']`. | **جديدة** |
| `getAuthToken()` | **مُحدثة** – تستخدم `normalizeApiResponse` لعرض رسائل خطأ واضحة، مع استخدام `getSettingsByKey` لجلب كلمة المرور المشفرة بشكل صحيح. | **مُحدثة** |
| `executeBuy(...)` | **مُحدثة** – تستخدم `normalizeApiResponse` لعرض رسائل خطأ دقيقة مثل "كود الشراء غير صحيح" أو "تم استخدام الكود مسبقاً". | **مُحدثة** |
| `checkTransactionStatus(...)` | **مُحدثة** – تستخدم `normalizeApiResponse` لتحسين عرض نتائج الاستعلام. | **مُحدثة** |
| `process(PaymentResult $result)` | **مُحدثة** – تعيين `$result->message` من الاستثناء قبل استدعاء `fail()` لضمان ظهور رسالة الخطأ في واجهة المستخدم. | **مُحدثة** |

#### 2.2. `_test_info.htm` – أدوات الاختبار السريع

تم تحديث واجهة الاختبار السريع لتشمل:

| الزر | الوصف |
|------|-------|
| **اختبار المصادقة** | التحقق من صحة بيانات الدخول (username/password/agentCode) |
| **اختبار المنفذ** | فحص اتصال المنفذ 5088 بالخادم البعيد (يستخدم `testPortConnection`) |
| **تنفيذ الدفع** | إنشاء معاملة دفع جديدة |
| **التحقق من الحالة** | الاستعلام عن حالة معاملة |
| **اختبار شامل** | تنفيذ الدفع + استعلام |
| **استرداد المبلغ** | إرسال طلب استرداد باستخدام `referenceID` |

#### 2.3. `routes.php` – نقاط نهاية API

تم إضافة المسارات الجديدة:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/jaibpay/test-port` | GET | اختبار المنفذ (يستخدم `JaibPay::testPortConnection`) |
| `/jaibpay/test-refund` | POST | استرداد مبلغ معاملة |

#### 2.4. `PaymentGatewaySettings` – إصلاح حفظ الإعدادات المشفرة

تم تعديل الكلاس `Nano\MicroCart\Models\PaymentGatewaySettings` لإصلاح مشكلة استبدال القيم المشفرة بقيم فارغة عند حفظ الإعدادات:

- **المشكلة:** عند فتح صفحة إعدادات البوابة والضغط على "حفظ" دون إعادة إدخال كلمات المرور (الحقول من نوع `password`)، كانت القيم المشفرة تُستبدل بقيم فارغة.
- **الحل:** تم تجاوز الدالة السحرية `__set` لمنع تعيين قيم فارغة للحقول المشفرة إذا كانت القيمة الحالية غير فارغة، مما يضمن بقاء الإعدادات الحسّاسة دون تغيير.

---

### 3. دورة عمل الدفع بعد التحديث

ظل نمط الدفع **Direct (فوري)** كما هو، ولكن مع تحسينات جوهرية في معالجة الأخطاء وتجربة المستخدم:

1. **المصادقة** – تستخدم `getAuthToken()` المحسّنة التي:
   - تجلب كلمة المرور المشفرة بشكل صحيح عبر `getSettingsByKey()`.
   - تستخدم `normalizeApiResponse` لتقديم رسائل خطأ واضحة (مثل "اسم المستخدم أو كلمة المرور غير صحيح").

2. **تنفيذ الدفع** – تستخدم `executeBuy()` المحسّنة التي:
   - تستخدم `normalizeApiResponse` لاستخراج رسائل الخطأ من استجابة Jaib مباشرة.
   - تعرض أخطاء دقيقة مثل:
     - `"كود الشراء غير صحيح"` (code: 51)
     - `"قد تم استخدام الكود مسبقاً"` (code: -1026)
     - `"رقم الكود غير صحيح"` (code: 51)

3. **معالجة الأخطاء** – في `process()`، يتم تعيين `$result->message` قبل استدعاء `fail()` لضمان ظهور رسالة الخطأ في واجهة المستخدم.

4. **استرداد الأموال** – دالة `refund()` الجديدة التي:
   - ترسل طلب استرداد إلى `/api/v1/BuyOnline/RefoundBuy`.
   - تغير حالة الطلب إلى `RefundedState`.
   - تخزن بيانات الاسترداد في `order->other_data['jaibpay_refund']`.

---

### 4. دوال جديدة مفصلة

#### 4.1. `normalizeApiResponse` – معالجة موحدة لاستجابات API

```php
private function normalizeApiResponse($response): array
```

**الوظيفة:** تستقبل استجابة HTTP (أو استثناء) وتستخرج بشكل موحد:
- `success` – نجاح العملية
- `status_code` – رمز HTTP
- `error_code` – كود الخطأ من Jaib (مثل `51`, `-100`, `-1026`)
- `error_message` – رسالة الخطأ من Jaib (مثل "كود الشراء غير صحيح")
- `data` – بيانات النتيجة (`result` من استجابة Jaib)
- `raw` – الاستجابة الخام كاملة

**الاستخدام:** تُستخدم في جميع دوال API (`getAuthToken`, `executeBuy`, `checkTransactionStatus`, `refund`) لتوحيد معالجة الردود.

#### 4.2. `testPortConnection` – اختبار المنفذ

```php
public static function testPortConnection(string $url, ?int $port = null, int $timeout = 5): array
```

**الوظيفة:** تختبر ما إذا كان منفذ معين مفتوحاً على خادم. تدعم:
- استخراج المنفذ تلقائياً من الرابط (مثل `https://www.api2.e-jaib.com:5088`).
- أولوية المنفذ الممرر Parameter على المنفذ في الرابط.
- استخدام المنفذ الافتراضي حسب scheme (443 لـ HTTPS، 80 لـ HTTP).
- فحص DNS وتحديد ما إذا كانت المشكلة في DNS أم في المنفذ.

**الاستخدام:** تُستخدم في مسار `jaibpay/test-port` وفي `_test_info.htm` لتشخيص مشاكل الاتصال.

#### 4.3. `refund` – استرداد المبلغ

```php
public function refund(string $referenceID, ?string $requestID = null, ?float $amount = null, ?string $currencyCode = null, ?string $notes = null): array
```

**الوظيفة:** تنفذ عملية استرداد كاملة:
- تستخدم `normalizeApiResponse` لمعالجة الرد.
- عند النجاح: تغير `order->payment_state` إلى `RefundedState` وتخزن بيانات الاسترداد في `order->other_data['jaibpay_refund']`.
- عند الفشل: تعرض رسالة الخطأ من Jaib.

---

### 5. إصلاحات الأخطاء

| المشكلة | الوصف | الحل |
|---------|-------|------|
| **حفظ الإعدادات المشفرة بقيم فارغة** | عند حفظ صفحة الإعدادات دون إعادة إدخال كلمة المرور، كانت القيم المشفرة تُستبدل بقيم فارغة. | تجاوز `__set` في `PaymentGatewaySettings` لمنع تعيين قيم فارغة للحقول المشفرة. |
| **رسائل خطأ غير واضحة** | عند فشل الدفع، كانت تظهر رسالة `Server error: POST ... 500 Internal Server Error` بدلاً من رسالة الخطأ الفعلية. | استخدام `normalizeApiResponse` لاستخراج `error.message` من استجابة Jaib. |
| **`$result->message` فارغ عند الفشل** | في `process()`، لم تكن رسالة الخطأ تُمرر إلى `PaymentResult`. | إضافة `$result->message = $e->getMessage();` قبل `$result->fail()`. |
| **جلب كلمة المرور المشفرة** | `PaymentGatewaySettings::get('jaibpay_password')` كان يرجع قيمة فارغة للحقول المشفرة. | استخدام `getSettingsByKey()` مع `getEncryptableValue()`. |
| **عدم ظهور رسالة خطأ `null`** | عند فشل `process()`، كان `$result->message` يبقى `null`. | تحسين `catch` بحيث يتم تعيين الرسالة قبل `fail`. |

---

### 6. اختبارات التكامل الفعلية

تم إجراء اختبارات تكامل فعلية مع منصة Jaib Pay عبر جميع السيناريوهات الممكنة:

#### 6.1. اختبار المصادقة
| السيناريو | النتيجة |
|-----------|---------|
| بيانات دخول صحيحة | ✅ نجاح – إرجاع `accessToken` و `pinApi` |
| بيانات دخول خاطئة | ✅ فشل – رسالة "اسم المستخدم او كلمة المرور غير صحيح" |

#### 6.2. اختبار المنفذ
| السيناريو | النتيجة |
|-----------|---------|
| فحص المنفذ 5088 على `www.api2.e-jaib.com` | ✅ نجاح – تم الاتصال بنجاح |

#### 6.3. اختبار تنفيذ الدفع
| السيناريو | المدخلات | النتيجة |
|-----------|----------|---------|
| **كود صحيح + مبلغ + عملة SAR** | `code`: صحيح، `amount`: 5000، `currency`: SAR | ✅ نجاح – تم الدفع |
| **كود صحيح + مبلغ خاطئ** | `code`: صحيح، `amount`: 6000 (أعلى من الرصيد) | ❌ فشل – رسالة خطأ من Jaib |
| **كود خاطئ + مبلغ صحيح** | `code`: خاطئ، `amount`: 5000 | ❌ فشل – "كود الشراء غير صحيح" |
| **كود صحيح + مبلغ صحيح + عملة SAR** | `code`: صحيح، `amount`: 5000، `currency`: SAR | ✅ نجاح – تم الدفع |
| **كود مستخدم مسبقاً** | `code`: مستخدم | ❌ فشل – "قد تم استخدام الكود مسبقاً" |
| **كود صحيح + مبلغ صحيح + عملة YER** | `code`: صحيح، `amount`: 5000، `currency`: YER | ✅ نجاح – تم الدفع |
| **كود صحيح + مبلغ صحيح + منتهي الصلاحية** | `code`: منتهي | ❌ فشل – رسالة خطأ من Jaib |

#### 6.4. اختبار استرداد المبلغ
| السيناريو | النتيجة |
|-----------|---------|
| استرداد مبلغ معاملة ناجحة | ✅ نجاح – تم الاسترداد |
| استرداد معاملة غير موجودة | ❌ فشل – "رسم مرجع العملية غير صحيح" |
| استرداد بدون `reference_id` | ✅ يتم الجلب تلقائياً من بيانات الطلب |
| محاولة استرداد طلب غير مدفوع | ❌ فشل – "لا يمكن استرداد طلب غير مدفوع" |

---

### 7. ملاحظات للمطورين

- **استخدام `normalizeApiResponse` أصبح إلزامياً** في جميع دوال API لضمان عرض رسائل خطأ دقيقة.
- **دالة `getSettingsByKey`** يجب استخدامها لجلب أي إعداد مشفر بدلاً من `PaymentGatewaySettings::get()` المباشر.
- **اختبار المنفذ** متاح عبر `JaibPay::testPortConnection()` ويمكن استخدامه لتشخيص أي مشكلة اتصال.
- **استرداد المبلغ** يغير حالة الطلب إلى `RefundedState` تلقائياً، ويخزن بيانات الاسترداد تحت `other_data['jaibpay_refund']`.
- **تحسين `PaymentGatewaySettings`** يمنع فقدان الإعدادات المشفرة، ولكن تأكد من مسح الكاش أحياناً (`Cache::forget('system::settings...')`) بعد التعديل.
- **دالة `refund`** تتطلب `referenceID` الخاص بالعملية الأصلية (يمكن الحصول عليه من `order->payment_trans_id` أو `other_data['jaibpay']['reference_id']`).

---

### 8. روابط ذات صلة

- [توثيق JaibPay (دليل المطور)](./docs/JaibPay/Docs-JaibPay-ar.md)
- [وثيقة API الخاصة بـ Jaib Pay – تسجيل الدخول (Login.pdf)](./Login.pdf)
- [وثيقة API الخاصة بـ Jaib Pay – تنفيذ واستعلام الدفع (Jaib Wallet Pay API.pdf)](./Jaib%20Wallet%20Pay%20API.pdf)
- [مجموعة Postman لاختبار API Jaib Pay](./Jaib%20Pay%20API.postman_collection.json)
- [دليل تطوير بوابات الدفع – نانوسوفت](./SKILL.md)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft

## 2026-05-08 - 2026-05-10

**إضافة `Nano.SchoolApi` – الإصدار 1.0.0**

### إنشاء واجهة برمجة تطبيقات RESTful شاملة لإدارة بيانات المدرسة الأساسية

---

### ملخص التحديثات

يقدم الإصدار الأول **1.0.0** من إضافة `Nano.SchoolApi` واجهة برمجة تطبيقات متكاملة للاستعلام عن جميع الكيانات الأساسية التي تديرها إضافة `Tss.School`. تم بناء الإضافة وفقاً لنمط `Nano.*Api` المعتمد في المشروع، مع توفير نقاط نهاية محمية لاسترجاع بيانات الصفوف (`Classes`)، المراحل (`Stages`)، المواد (`Subjects`)، الشعب (`Groups`)، توزيع الشعب على الصفوف (`ClassGroups`)، توزيع المواد على الصفوف (`ClassSubjects`)، توزيع المعلمين على المواد (`TeacherSubjects`)، ربط مراكز التكلفة بالصفوف (`ClassCostCenters`)، أنواع الرسوم (`FeesTypes`)، أنواع الوثائق (`DocumentTypes`)، المدارس الخارجية (`OutSchools`)، وثائق الطلاب (`StudentDocuments`)، رسوم الطلاب (`StudentsFees`) وعلامات آخر سنة (`LastStudentMarks`). يدعم الإصدار نظام صلاحيات متعدد المستويات لكل مورد، ومحولات بيانات احترافية مع دعم كامل للترجمة.

---

### أهداف الإصدار

- **توفير API موحد** لجميع كيانات `Tss.School` الحيوية، مما يفتح البيانات لاستخدامات متعددة.
- **تطبيق نمط معماري متناسق** مع إضافات `Nano.*Api` الأخرى لتسهيل الصيانة والتوسع.
- **تمكين التطبيقات الخارجية والواجهات الأمامية** من الوصول إلى بيانات المدرسة (الصفوف، المواد، الجداول، الرسوم، الوثائق) بسهولة وأمان.
- **توفير تحكم دقيق في صلاحيات الوصول** لكل مورد، مما يسمح بنشر الإضافة في بيئات إنتاجية مع صلاحيات متفاوتة.
- **دعم التصفية المتقدمة والبحث والترتيب** لتلبية احتياجات التقارير والتكاملات.
- **تسهيل بناء تقارير وإحصائيات** تعتمد على هذه البيانات دون الحاجة للوصول المباشر لقاعدة البيانات.

---

### الميزات الجديدة والتحسينات

#### 1. وحدات تحكم RESTful شاملة (14 متحكم)

تم إنشاء 14 وحدة تحكم تغطي جميع الكيانات الأساسية لإضافة `Tss.School`، وكلها تدعم عمليات القراءة (`list` و `show`):

| المتحكم | النموذج المستهدف | الوصف |
|--------|-----------------|-------|
| `Classes` | `TbClass` | الصفوف الدراسية (مع الرسوم والترتيب والمرحلة) |
| `Stages` | `Stage` | المراحل الدراسية |
| `Subjects` | `Subject` | المواد الدراسية |
| `Groups` | `Group` | الشعب الدراسية |
| `ClassGroups` | `ClassGroup` | ربط الشعب بالصفوف وسنة دراسية |
| `ClassSubjects` | `ClassSubject` | توزيع المواد على الصفوف (الوحدات، الدرجات) |
| `TeacherSubjects` | `TeacherSubject` | توزيع المعلمين على المواد والشعب |
| `ClassCostCenters` | `ClassCostCenter` | ربط مراكز التكلفة بالصفوف |
| `FeesTypes` | `FeesType` | أنواع الرسوم الدراسية |
| `DocumentTypes` | `DocumentType` | أنواع الوثائق المطلوبة |
| `OutSchools` | `OutSchool` | المدارس الخارجية |
| `StudentDocuments` | `StudentDocument` | وثائق الطلاب (حالة التسليم) |
| `StudentsFees` | `StudentsFee` | رسوم الطلاب (المبلغ، الخصم) |
| `LastStudentMarks` | `LastStudentMark` | علامات آخر سنة دراسية للطالب |

**كل متحكم يقدم:**

- **`index()`**: جلب قائمة بالسجلات مع تصفية متعددة وبحث وترتيب وتقسيم صفحات.
- **`show($id)`**: عرض تفاصيل سجل واحد مع العلاقات.
- **`getRecords(array $options)`**: دالة أساسية لبناء استعلامات معقدة مع دعم إعدادات افتراضية من `config.php`.
- **`activelystats()`**: نقطة نهاية لمراقبة آخر تحديث (لأغراض التخزين المؤقت).
- **`getLastUpdateAt()`**: إرجاع الطابع الزمني لآخر تحديث.
- **`validationList($user)`**: دالة موحدة للتحقق من صلاحيات الوصول.

#### 2. نظام صلاحيات محكم لكل مورد

تم تطبيق نظام صلاحيات منفصل لكل من الموارد الأربعة عشر، مع تحكم كامل عبر إعدادات `config.php` ومتغيرات البيئة (`env`). كل مورد يمتلك:

- `is_allow_list`: تفعيل/تعطيل عملية الجرد.
- `is_allow_list_backend` / `is_allow_list_frontend`: السماح لمستخدمي الخلفية أو الواجهة الأمامية.
- `is_check_access_list`: هل يجب التحقق من وجود مستخدم صحيح.
- `is_check_list_permission`: هل يجب التحقق من صلاحية محددة (مثل `tss.school.tbclasses.access`).

بالإضافة إلى إعدادات التصفية: `order_by`, `order_dir`, `per_page`, `exclude`.

**مثال من `config.php`:**
```php
'classes' => [
    'is_allow_list'            => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_BACKEND', true),
    'is_allow_list_frontend'   => env('NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND', true),
    'is_check_access_list'     => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_ACCESS_LIST', true),
    'is_check_list_permission' => env('NANO_SCHOOLAPI_CLASSES_IS_CHECK_LIST_PERMISSION', true),
    'order_by'                 => env('NANO_SCHOOLAPI_CLASSES_ORDER_BY', 'sort_order'),
    'order_dir'                => env('NANO_SCHOOLAPI_CLASSES_ORDER_DIR', 'asc'),
    'per_page'                 => env('NANO_SCHOOLAPI_CLASSES_PER_PAGE', 15),
    'exclude'                  => env('NANO_SCHOOLAPI_CLASSES_EXCLUDE', ''),
],
```

#### 3. دعم ذكي للقيم الافتراضية للموارد المعتمدة على السنة الدراسية

العديد من الموارد (مثل `ClassGroups`, `TeacherSubjects`, `ClassCostCenters`, `StudentDocuments`, `StudentsFees`) ترتبط بسنة دراسية (`year_id`). عند عدم تمرير هذه القيمة، تقوم المتحكمات تلقائياً باعتماد **السنة الدراسية الافتراضية** (`Period::getPrimary()`) مما يضمن تجربة سلسة دون أخطاء.

**مثال من `ClassGroups::getRecords()`:**
```php
if (!$year_id) {
    $primary = Period::getPrimary();
    if ($primary) {
        $year_id = $primary->id;
    }
}
```

#### 4. محولات بيانات شاملة (14 محول)

تم إنشاء محول بيانات لكل نموذج، تقوم بتنسيق الاستجابة وتوفير العلاقات المضمنة مع معالجة الأخطاء:

- **`ClassTransformer`**: الصفوف مع `company`, `department`, `stage`.
- **`StageTransformer`**: المراحل مع `company`, `department`.
- **`SubjectTransformer`**: المواد مع `company`, `department`.
- **`GroupTransformer`**: الشعب مع `company`, `department`.
- **`ClassGroupTransformer`**: ربط الشعب مع `company`, `department`, `class`, `group`, `study_year`.
- **`ClassSubjectTransformer`**: توزيع المواد مع `company`, `department`, `class`, `subject`.
- **`TeacherSubjectTransformer`**: توزيع المعلمين مع `company`, `department`, `class`, `group`, `subject`, `teacher`, `study_year`.
- **`ClassCostCenterTransformer`**: ربط مراكز التكلفة مع `company`, `department`, `class`, `cost_center`, `study_year`.
- **`FeesTypeTransformer`**: أنواع الرسوم مع `company`, `department`.
- **`DocumentTypeTransformer`**: أنواع الوثائق مع `company`, `department`.
- **`OutSchoolTransformer`**: المدارس الخارجية مع `company`, `department`.
- **`StudentDocumentTransformer`**: وثائق الطلاب مع `company`, `department`, `student`, `documents`, `study_year`.
- **`StudentsFeeTransformer`**: رسوم الطلاب مع `company`, `department`, `student`, `student_record`, `fees_type`, `study_year`.
- **`LastStudentMarkTransformer`**: علامات آخر سنة مع `company`, `department`, `student`, `class`, `out_school`, `student_record`, `study_year`.

**ميزات المحولات:**
- تنسيق القيم إلى الأنواع الصحيحة (int, string, bool, float, date).
- دالة `formatDate` لتحويل التواريخ بشكل آمن.
- تصفية الحقول (`exclude`) عبر querystring للتحكم في حجم البيانات المرتجعة.
- إضافة `object_type` للتعرف على نوع الكائن.
- كل علاقة محاطة بـ `try-catch` لتجنب فشل الاستعلام الكامل.

#### 5. تسجيل نطاق `scopeExclude` لجميع النماذج

تم تسجيل النطاق الديناميكي `scopeExclude` لجميع النماذج الأربعة عشر داخل `Plugin::boot()`، مما يسمح للعملاء بطلب أعمدة محددة فقط وتقليل البيانات المنقولة عبر الشبكة.

```php
foreach ($models as $model) {
    $model::extend(function($model) {
        $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
            $getTable = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
            return $query->select(array_diff($getTable, (array) $columns));
        });
    });
}
```

#### 6. دعم التخزين المؤقت

جميع نقاط النهاية `index` تدعم التخزين المؤقت عبر `nano.api::api_enable_cache`، مع إبطال الكاش تلقائياً عند تحديث أي سجل.

#### 7. ترجمة شاملة

ملف `lang/en/lang.php` يحتوي على جميع رسائل النجاح والخطأ والصلاحيات لكل مورد، مما يسهل ترجمتها إلى أي لغة.

---

### أمثلة على الاستخدام

#### جلب قائمة الصفوف النشطة لمرحلة معينة

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes?stage_id=2&isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### جلب توزيع المعلمين على المواد لسنة معينة مع تضمين المعلم والمادة

```bash
curl -X GET "https://yourdomain.com/api/v1/school/teacher-subjects?year_id=5&include=teacher,subject" \
  -H "Authorization: Bearer <token>"
```

#### جلب رسوم طالب محدد

```bash
curl -X GET "https://yourdomain.com/api/v1/school/students-fees?student_id=15" \
  -H "Authorization: Bearer <token>"
```

#### البحث عن مادة دراسية بالاسم

```bash
curl -X GET "https://yourdomain.com/api/v1/school/subjects?q=رياضيات" \
  -H "Authorization: Bearer <token>"
```

#### عرض تفاصيل صف دراسي مع المرحلة

```bash
curl -X GET "https://yourdomain.com/api/v1/school/classes/3?include=stage" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تكامل غير مسبوق**: ولأول مرة، يمكن لأي نظام خارجي (تطبيق جوال، موقع أولياء الأمور، لوحات معلومات) الوصول إلى جميع بيانات المدرسة عبر API واحد.
- **تسريع بناء التقارير**: يمكن للمطورين الآن إنشاء تقارير مخصصة عن الصفوف والمواد والرسوم والوثائق باستخدام استعلامات API بسيطة بدلاً من كتابة استعلامات SQL معقدة.
- **تحسين تجربة المستخدم**: الواجهات الأمامية يمكنها جلب القوائم المنسدلة (مثل الصفوف والمراحل والمواد) بشكل ديناميكي من الخادم.
- **جاهزية عالية للإنتاج**: نظام الصلاحيات المتقدم يسمح بالتحكم الدقيق في وصول كل مستخدم، مما يجعل الإضافة مناسبة للنشر في بيئات حقيقية.
- **تصميم قابل للتوسع**: يمكن إضافة عمليات `POST` و `PUT` لأي مورد مستقبلاً بسهولة دون تغيير الهيكل الأساسي.
- **أداء ممتاز**: استخدام `scopeExclude` مع التخزين المؤقت يضمن استجابات سريعة حتى مع آلاف السجلات.

---

### متطلبات الترقية

1. **تثبيت الإضافة**: انسخ مجلد `nano/schoolapi` إلى `plugins/nano/schoolapi`.
2. **تسجيل الإضافة**: شغّل `php artisan plugin:refresh Nano.SchoolApi`.
3. **المتغيرات البيئية**: اضبط متغيرات `env` حسب الحاجة (مثل `NANO_SCHOOLAPI_CLASSES_IS_ALLOW_LIST_FRONTEND`).
4. **الصلاحيات**: تأكد من أن الأدوار تمتلك صلاحيات الوصول المناسبة في النظام الخلفي إذا تم تفعيل `is_check_list_permission`.
5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **لا توجد هجرات قاعدة بيانات** مطلوبة.

---

### الخاتمة

يمثل الإصدار **1.0.0** من `Nano.SchoolApi` نقلة نوعية في إتاحة بيانات المدرسة عبر API. بتغطية 14 كياناً أساسياً، توفر الإضافة قاعدة متينة لبناء أنظمة متكاملة تعتمد على بيانات `Tss.School` دون عناء. التصميم الموحد وقابلية التوسع العالية يجعلانها منصة مثالية لإضافة المزيد من العمليات والتقارير في المستقبل.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano.StudyyearApi`](./docs/StudyyearApi/Docs-StudyyearApi-ar.md)
- [توثيق إضافة `Nano.SchoolApi`](./docs/SchoolApi/Docs-SchoolApi-ar.md)
- [توثيق إضافة `Nano.StudentsApi`](./docs/StudentsApi/Docs-StudentsApi-ar.md)


## 2026-05-11 - 2026-05-12

**إضافة `Nano.AbsenceApi` – الإصدار 1.0.0**

### إنشاء واجهة برمجة تطبيقات RESTful لإدارة الحضور والغياب والكشوفات وأنواعها

---

### ملخص التحديثات

يقدم الإصدار الأول **1.0.0** من إضافة `Nano.AbsenceApi` واجهة برمجة تطبيقات متكاملة للتفاعل مع سجلات الحضور والغياب وكشوفات التحضير وتصنيفاتها. تم بناء الإضافة وفقاً لنمط `Nano.*Api` المعتمد، مع توفير نقاط نهاية محمية لاسترجاع وإدارة سجلات الغياب (`Absences`) وملفات الكشوفات (`Files`) والتصنيفات (`ClassTypes` و `ClassTypeCompanies`). يدعم هذا الإصدار أيضاً عمليات إنشاء وتحديث سجلات الحضور عبر API، مع نظام صلاحيات متعدد المستويات، ومحولات بيانات احترافية.

---

### أهداف الإصدار

- **توفير API موحد** لجميع كيانات الحضور والغياب (السجلات، الكشوفات، التصنيفات).
- **تطبيق نمط معماري متناسق** مع إضافات `Nano.*Api` الأخرى.
- **تمكين التطبيقات الخارجية** من تسجيل الحضور وتعديله واستعلام التقارير عبر API.
- **توفير تحكم دقيق في الصلاحيات** لكل مورد ولكل عملية (قراءة، إنشاء، تحديث).
- **دعم التصفية المتقدمة** حسب السنة الدراسية والفصل والشهر والصف والشعبة والطالب وحالة الحضور وغيرها.
- **تسهيل التوسع المستقبلي** لإضافة تقارير أو عمليات حذف جماعي.

---

### الميزات الجديدة والتحسينات

#### 1. وحدات تحكم RESTful شاملة

تم إنشاء أربع وحدات تحكم رئيسية تغطي كافة الكيانات:

| المتحكم | النموذج المستهدف | العمليات المدعومة |
|---------|-----------------|-------------------|
| `Absences` | `Tss\Absence\Models\Absence` | قائمة، عرض، **إنشاء، تحديث** |
| `Files` | `Tss\Absence\Models\File` | قائمة، عرض |
| `ClassTypes` | `Tss\Absence\Models\ClassType` | قائمة، عرض |
| `ClassTypeCompanies` | `Tss\Absence\Models\ClassTypeCompany` | قائمة، عرض |

**كل متحكم يدعم:**

- **`index()`**: جلب قائمة مسجلة مع تصفية متعددة وبحث وترتيب وتقسيم صفحات.
- **`show($id)`**: عرض تفاصيل سجل واحد مع العلاقات.
- **`getRecords(array $options)`**: دالة أساسية لبناء استعلامات معقدة.
- **`activelystats()`**: نقطة نهاية لمراقبة آخر تحديث (للتخزين المؤقت).
- **`validationList($user)`**: تحقق موحد من صلاحيات الوصول.

**بالنسبة لـ `Absences` تحديداً:**

- **`store()`**: إنشاء سجل غياب جديد مع تحقق من الصلاحيات والبيانات المدخلة وضبط القيم الافتراضية (الشركة، الفرع، السنة).
- **`update($id)`**: تحديث سجل قائم مع إعادة استخدام نفس منطق الإنشاء (دالة `onStoreUpdateAbsence`).

#### 2. نظام صلاحيات متعدد المستويات وقابل للتخصيص

تم تطبيق نظام صلاحيات منفصل لكل عملية (list, create, update) ولكل نوع مستخدم (backend, frontend). يمكن التحكم عبر ملف `config.php` باستخدام متغيرات البيئة:

```php
'absences' => [
    'is_allow_list'            => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_LIST', true),
    'is_allow_list_backend'    => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_LIST_BACKEND', true),
    // ...
    'is_allow_create'          => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_CREATE', true),
    'is_allow_create_backend'  => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_CREATE_BACKEND', true),
    'is_check_create_permission'=> env('NANO_ABSENCEAPI_ABSENCES_IS_CHECK_CREATE_PERMISSION', true),
    // ...
    'is_allow_update'          => env('NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_UPDATE', true),
    // ...
],
```

بالإضافة إلى ذلك، يمكن ربط الصلاحيات بصلاحيات النظام الخلفي (مثل `tss.absence.absences.add`).

#### 3. دعم ذكي للقيم الافتراضية في سجلات الحضور

عند إنشاء سجل غياب جديد، إذا لم يتم تمرير معرف السنة الدراسية (`year_id`) أو الفرع (`departments_id`) أو الشركة (`companys_id`)، تقوم الدالة تلقائياً باستخدام القيم الافتراضية من النظام (مثل `Period::getPrimary()`)، مما يسهل عملية التسجيل السريع.

**مثال من `Absences::onStoreUpdateAbsence()`:**
```php
if (empty($data['companys_id'])) {
    $data['companys_id'] = \Tss\Basic\Helpers\BasicHelper::getCompanysId(true);
}
if (empty($data['departments_id'])) {
    $data['departments_id'] = \Tss\Basic\Helpers\BasicHelper::getMainDepartmentId(true);
}
if (empty($data['year_id'])) {
    $primary = Period::getPrimary();
    if ($primary) $data['year_id'] = $primary->id;
}
```

#### 4. محولات بيانات متكاملة (Transformers)

تم إنشاء أربعة محولات:

- **`AbsenceTransformer`**: يعرض بيانات سجل الحضور مع تضمين العلاقات (`company`, `department`, `file`, `student`, `record`).
- **`FileTransformer`**: يعرض بيانات ملف الكشف مع تضمين العلاقات (`company`, `department`).
- **`ClassTypeTransformer`**: يعرض التصنيفات الرئيسية مع العلاقات (`company`).
- **`ClassTypeCompanyTransformer`**: يعرض تصنيفات الشركات مع العلاقات (`company`, `department`, `class_type`).

كل المحولات تدعم:
- تنسيق القيم إلى الأنواع الصحيحة.
- دالة `formatDate` للتواريخ.
- تصفية الحقول (`exclude`) لتقليل حجم البيانات.
- `object_type` لمعرفة نوع الكائن.
- معالجة أخطاء العلاقات عبر `try-catch`.

#### 5. استجابة موحدة لعمليات الإنشاء والتحديث

تتبع عمليات `store` و `update` في `Absences` هيكل استجابة موحد ومتناسق مع باقي إضافات Nano. يحتوي الرد على:
- `code`, `status`, `message`
- `error`, `errors`
- `data` (البيانات المحولة عبر Transformer)
- `model` (النموذج الخام)
- `input_data`, `process_data`
- `debug` (في وضع التطوير)

مما يسهل معالجة الأخطاء في الواجهات الأمامية.

#### 6. تصفية وبحث متقدم

يمكن تمرير العديد من عوامل التصفية إلى `getRecords` حسب المورد:

- **`Absences`**: `year_id`, `semster`, `month_num`, `class_id`, `group_id`, `student_id`, `record_id`, `absences_status`, `date_at`, `files_id`, `ref_type_class`, إلخ.
- **`Files`**: `companys_id`, `departments_id`, `ref_type_class`.
- **`ClassTypes`**: `ref_type`.
- **`ClassTypeCompanies`**: `companys_id`, `departments_id`, `ref_type`.

بالإضافة إلى البحث النصي `q` عبر الحقول المنطقية (الاسم، الكود، الملاحظات، إلخ) والترتيب حسب أي عمود.

#### 7. تسجيل نطاق `scopeExclude`

تم تسجيل النطاق الديناميكي `scopeExclude` لجميع النماذج الأربعة (`Absence`, `File`, `ClassType`, `ClassTypeCompany`) مما يسمح للمستخدم باختيار الأعمدة المطلوبة فقط وتقليل البيانات المنقولة.

```php
\Tss\Absence\Models\Absence::extend(function($model) {
    $model->addDynamicMethod('scopeExclude', function($query, $columns) use($model) {
        $getTable = $model->getConnection()->getSchemaBuilder()->getColumnListing($model->getTable());
        return $query->select(array_diff($getTable, (array) $columns));
    });
});
```

#### 8. دعم كامل للترجمة

جميع رسائل النجاح والخطأ والصلاحيات قابلة للترجمة عبر ملف `lang/en/lang.php`، مما يسهل تعريبها أو ترجمتها للغات أخرى.

---

### أمثلة على الاستخدام

#### جلب قائمة سجلات الغياب ليوم محدد مع تضمين الطالب

```bash
curl -X GET "https://yourdomain.com/api/v1/absence/absences?date_at=2026-05-01&include=student" \
  -H "Authorization: Bearer <token>"
```

#### تسجيل غياب جديد لطالب

```bash
curl -X POST "https://yourdomain.com/api/v1/absence/absences" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "student_id": "15",
    "record_id": "8",
    "year_id": "3",
    "class_id": "2",
    "semster": "semster2",
    "month_num": "2",
    "files_id": "5",
    "ref_type_class": "day",
    "absences_status": "absent",
    "absences_bacuse": "مرض",
    "date_at": "2026-05-05"
  }'
```

#### تحديث حالة غياب إلى "حاضر"

```bash
curl -X PUT "https://yourdomain.com/api/v1/absence/absences/42" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "absences_status": "present"
  }'
```

#### جلب كشوفات التحضير اليومية النشطة

```bash
curl -X GET "https://yourdomain.com/api/v1/absence/files?ref_type_class=day&isActive=1" \
  -H "Authorization: Bearer <token>"
```

#### جلب التصنيفات المتاحة للشركة الحالية

```bash
curl -X GET "https://yourdomain.com/api/v1/absence/class-type-companies" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **أتمتة الحضور والغياب**: يمكن للتطبيقات الخارجية (تطبيقات الجوال، أنظمة الإشعارات) تسجيل الغياب مباشرة دون الحاجة لاستخدام لوحة التحكم.
- **تكامل مع التقارير**: يمكن لأدوات إعداد التقارير جلب البيانات الخام عبر API بدلاً من تصدير CSV يدوياً.
- **مرونة عالية في الصلاحيات**: يمكن السماح للمعلمين بتسجيل الغياب عبر API بينما تبقى صلاحيات الحذف محصورة بالمسؤولين.
- **تقليل أخطاء الإدخال**: التحقق من صحة البيانات المدخلة عبر API يمنع إدخال سجلات غير مكتملة.
- **أداء ممتاز**: استخدام `scopeExclude` والتخزين المؤقت يضمن استجابات سريعة حتى مع آلاف السجلات.

---

### متطلبات الترقية

1. **تثبيت الإضافة**: انسخ مجلد `nano/absenceapi` إلى `plugins/nano/absenceapi`.
2. **تسجيل الإضافة**: شغّل `php artisan plugin:refresh Nano.AbsenceApi`.
3. **إعدادات البيئة**: اضبط متغيرات `env` المطلوبة (مثل `NANO_ABSENCEAPI_ABSENCES_IS_ALLOW_CREATE_FRONTEND=true`).
4. **الصلاحيات**: تأكد من أن الأدوار تمتلك صلاحيات مثل `tss.absence.absences.add` و `tss.absence.absences.edit` إذا تم تفعيل `is_check_create_permission` و `is_check_update_permission`.
5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **لا توجد هجرات قاعدة بيانات** مطلوبة.

---

### الخاتمة

يمثل الإصدار **1.0.0** من `Nano.AbsenceApi` نقلة نوعية في إدارة الحضور والغياب عبر API. بفضل التصميم الموحد والقابل للتوسع، يمكن للمطورين الآن بناء تطبيقات متكاملة تعتمد على بيانات الحضور بشكل مباشر وآمن. تغطي الإضافة جميع الاحتياجات الأساسية من استعلام وتسجيل وتحديث، مع أساس متين لإضافة عمليات الحذف والتقارير المتقدمة في الإصدارات المستقبلية.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano.StudyyearApi`](./docs/StudyyearApi/Docs-StudyyearApi-ar.md)
- [توثيق إضافة `Nano.SchoolApi`](./docs/SchoolApi/Docs-SchoolApi-ar.md)
- [توثيق إضافة `Nano.StudentsApi`](./docs/StudentsApi/Docs-StudentsApi-ar.md)
- [توثيق إضافة `Nano.AbsenceApi`](./docs/AbsenceApi/Docs-AbsenceApi-ar.md)

## 2026-05-13 - 2026-05-14

**تحديثات إضافة `Nano.UserPlus` – الإصدار 1.1.7**

### ملخص التحديثات

بعد دعم أنواع حسابات متعددة في الإصدارات السابقة، يأتي الإصدار 1.1.7 ليكمل بناء تجربة إدارة مستخدمين مرنة ومتكاملة. يركز هذا الإصدار على ثلاثة محاور رئيسية:

- **تحسين آلية تعريف أنواع الحسابات** وجعلها قابلة للتحكم الكامل من ملف الإعدادات.
- **التوافق الكامل** مع الإصدارات الحديثة من RainLab.User (التي تستخدم `first_name` و `last_name` بدلاً من `name` و `surname`).
- **إدارة متقدمة لمجموعات المستخدمين** عبر إضافة حقول جديدة (`is_new_user_default`, `is_active`, `sort_order`) إلى جدول `user_groups`، وتوسيع واجهة المستخدم الخلفية لإدارتها بسلاسة.

هذه التحسينات تمنح المطورين ومديري النظام تحكماً أدق في تجربة المستخدمين، وتوحد سلوك المجموعات الافتراضية، وتضمن توافق الإضافة مع أحدث إصدارات المنصة.

---

### الإصدار 1.1.7 – أنواع حسابات مرنة، توافق محسّن، وإدارة متطورة لمجموعات المستخدمين

#### أهداف الإصدار

- **جعل أنواع الحسابات (`ref_type`)** قابلة للتخصيص من ملف الإعدادات دون المساس بالكود.
- **ضمان التوافق** مع تغييرات حقول الأسماء في RainLab.User 2.x عبر معالج أحداث مخصص.
- **توسيع جدول `user_groups`** بأعمدة التحكم في التفعيل والترتيب والتعيين الافتراضي.
- **دمج الحقول الجديدة** في نماذج وقوائم إدارة مجموعات المستخدمين في لوحة التحكم.
- **دعم تعدد اللغات** لجميع التوسيعات الجديدة مع معالجة آمنة للأخطاء.

#### الميزات الجديدة

##### 1. أنواع حسابات قابلة للتخصيص عبر الإعدادات

تمت إعادة هيكلة دالة `getRefTypeList` في موديل `Nano\UserPlus\Models\Preference` لتصبح ديناميكية بالكامل، حيث تعتمد على مفاتيح الإعدادات مثل:
```php
Config::get('nano.userplus::ref_type.department', true)
```
وبذلك يمكن تفعيل أو تعطيل أي نوع حساب (مثل `delivery`, `partner`, `student`, `employee` ...) عبر ملف `config.php` دون الحاجة إلى تعديل الكود البرمجي. كما تم تحديث الدوال المساعدة `getRefTypeOptions` في موديل المستخدم لتغذية القوائم المنسدلة بنفس الخيارات الديناميكية.

##### 2. التوافق مع RainLab.User 2.x

بسبب تغيير حقلي `name` و `surname` إلى `first_name` و `last_name` في الإصدارات الحديثة من RainLab.User، تم إضافة كلاس الحدث `Nano\UserPlus\EventHandlers\RainlabUser2CompatibilityHandler` الذي يقوم بربط وهمي بين الحقول القديمة والجديدة عبر وسائط ديناميكية (`addDynamicMethod`). هذا يضمن استمرار عمل أي كود يعتمد على `name` و `surname` دون أخطاء، مع الحفاظ على البيانات الأصلية في `first_name` و `last_name`.

تم تفعيل هذا المعالج تلقائياً عند وجود عمود `first_name` في جدول `users`.

##### 3. أعمدة جديدة في جدول `user_groups`

تمت إضافة الهجرة `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` التي تضيف الأعمدة التالية إلى جدول `user_groups`:

| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `is_new_user_default` | boolean (default: false) | تحديد ما إذا كانت هذه المجموعة هي الافتراضية للمستخدمين الجدد. |
| `is_active` | boolean (default: true) | تفعيل أو تعطيل المجموعة (لإخفائها من القوائم). |
| `sort_order` | integer (nullable) | ترتيب ظهور المجموعة في القوائم المنسدلة والواجهات. |

##### 4. توسيع نموذج `RainLab\User\Models\UserGroup`

عبر دالة `extendUserGroupModel` في `Plugin.php`، تم:

- إضافة الحقول إلى `$fillable`.
- تعيين قواعد تحقق مناسبة (`boolean`, `integer`).
- ضبط قيم افتراضية ذكية عند إنشاء مجموعة جديدة.
- إضافة دوال مساعدة مثل `getIsActiveOptions` و `getIsNewUserDefaultOptions` لإرجاع خيارات مترجمة.

كل ذلك مع معالجة استثناءات `try-catch` وتسجيل الأخطاء دون تعطيل النظام.

##### 5. تحسين واجهة إدارة مجموعات المستخدمين

تم توسيع متحكم `RainLab\User\Controllers\UserGroups` بطريقتين متكاملتين:

- **أعمدة القائمة (List Columns)**: تم إضافة الأعمدة `is_new_user_default` و `is_active` (بشكل Switch) و `sort_order` إلى قائمة المجموعات، مع إمكانية الفرز والإظهار/الإخفاء.
- **حقول النموذج (Form Fields)**: تم حقن الحقول الثلاثة في نموذج إنشاء/تعديل المجموعة، مع تعليقات مساعدة، وتوزيعها على أقسام `left`/`right` لتنظيم المساحة.

جميع التوسيعات تمت بمراعاة التحقق من السياق (مثلاً أن النموذج ليس فرعياً `isNested`) لضمان عدم التداخل مع النماذج الأخرى.

##### 6. دعم الترجمة الكامل

تمت إضافة مفاتيح ترجمة جديدة في ملف `lang.php` تحت القسم `user_groups` تشمل:

- أسماء الحقول.
- التعليقات المساعدة.
- خيارات القيم (`is_default_yes`, `is_active_yes` …).

كما تم استخدام الدالة `trans()` في جميع المواقع لضمان ظهور النصوص باللغة المناسبة.

---

### أمثلة عملية

#### 1. تخصيص أنواع الحسابات عبر الإعدادات

في ملف `config.php` يمكن تعطيل نوع "موصل" مثلاً:

```php
'ref_type' => [
    'delivery' => false,
    'department' => true,
    // ... باقي الأنواع
]
```

بمجرد حفظ الملف، ستختفي خيارات "عامل توصيل" من قوائم `ref_type` في نماذج المستخدمين.

#### 2. تعيين مجموعة افتراضية للمستخدمين الجدد

بعد التحديث، عند الدخول إلى إدارة المجموعات، يمكن اختيار مجموعة وجعلها "المجموعة الافتراضية للمستخدمين الجدد". يقترن ذلك بحدث يستمع لتسجيل مستخدم جديد ليقوم بربطه بهذه المجموعة تلقائياً (إذا تم تفعيل الخاصية).

#### 3. ترتيب المجموعات في الواجهة الأمامية

باستخدام حقل `sort_order` يمكن ترتيب المجموعات في القوائم المنسدلة، مما يحسن تجربة المستخدم عند التسجيل أو في لوحة التحكم.

---

### ملخص الإصدارات (1.1.6 – 1.1.7)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.1.6 | دعم حقول إضافية للمستخدمين (`phone`, `mobile`, `company`...) وتوسيع الفلاتر. |
| 1.1.7 | أنواع حسابات ديناميكية، معالج توافق `RainlabUser2CompatibilityHandler`، أعمدة `user_groups` الجديدة، وتوسيع واجهة إدارة المجموعات. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `Plugin.php` بالنسخة الجديدة التي تحوي دوال `extendUserGroupModel` و `extendUserGroupsController`.
   - إضافة ملف `eventhandlers/RainlabUser2CompatibilityHandler.php`.
   - تحديث ملف `models/Preference.php` إذا تم تضمين التغييرات الخاصة بالأنواع.
   - تحديث ملف اللغة `lang.php` لإضافة مفاتيح `user_groups`.

2. **تنفيذ الهجرة**:
   - تأكد من تشغيل الهجرة `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` التي تضيف الأعمدة الجديدة إلى جدول `user_groups`. يمكن تنفيذها عبر `php artisan october:up` أو يدوياً.

3. **تحديث الإعدادات** (اختياري):
   - راجع ملف `config.php` لتخصيص أنواع الحسابات المطلوبة (مثلاً `ref_type.delivery` = true/false).

4. **مسح الكاش**:
   - بعد التحديث، قم بتشغيل `php artisan cache:clear` لضمان تحميل ملفات الترجمة والإعدادات الجديدة.

5. **اختبار الوظائف**:
   - تأكد من ظهور الأعمدة الجديدة في قائمة مجموعات المستخدمين.
   - جرب إنشاء مجموعة جديدة وتفعيل خيار "المجموعة الافتراضية" ثم تسجيل مستخدم جديد للتحقق من الارتباط التلقائي.
   - تحقق من أن أنواع الحسابات (`ref_type`) تظهر وفق إعدادات `config.php`.

---

### الخاتمة

مع الإصدار 1.1.7، نكون قد قطعنا شوطاً كبيراً في جعل إضافة `Nano.UserPlus` أكثر مرونة وتكاملاً مع المنصة. إدارة أنواع الحسابات أصبحت ديناميكية تماماً، وتم حل تحدي التوافق مع الإصدارات الحديثة، كما تم تعزيز إدارة مجموعات المستخدمين بأدوات تحكم متقدمة.

هذه التحسينات تفتح الباب لاستخدامات أكثر تخصيصاً، مثل:

- توجيه مستخدمين جدد إلى مجموعات محددة تلقائياً.
- إخفاء مجموعات غير نشطة من الواجهات.
- ترتيب المجموعات حسب الأولوية في استمارات التسجيل.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

---

**الوثائق المرجعية**:
- [توثيق إضافة UserPlus](./docs/UserPlus/Docs-Nano-UserPlus-en.md)
- [توثيق موديل Preference](./docs/UserPlus/Docs-Preference-Model-en.md)
- [توثيق سلوكيات UserModelBehavior](./docs/UserPlus/Docs-UserModelBehavior-en.md)


## 2026-05-15 - 2026-05-16

**تحديثات إضافة `Nano.AuthApi` – الإصدار 1.0.19**

### ملخص التحديثات

بعد توفير إدارة متكاملة لملفات المستخدمين وتسجيل الدخول، يأتي الإصدار 1.0.19 ليعزز تجربة المطورين عبر **توسيع واجهة API بمجموعة من نقاط النهاية الجديدة** التي تتيح:

- **جلب خيارات الحقول الجاهزة** (مثل الجنس، نوع الحساب، اللغة، الجنسية...) بشكل ديناميكي.
- **إدارة قوائم مجموعات المستخدمين** مع دعم فلاتر متقدمة تشمل الحالة الافتراضية والترتيب.
- **دعم كامل للأعمدة الجديدة** (`is_new_user_default`, `is_active`, `sort_order`) في جدول `user_groups`.

يهدف هذا الإصدار إلى تمكين تطبيقات الواجهة الأمامية من بناء نماذج تسجيل وبحث ديناميكية دون الحاجة إلى طلبات متعددة، وتوحيد آلية الوصول إلى خيارات النظام عبر API احترافية.

---

### الإصدار 1.0.19 – خيارات الحقول، نقاط نهاية مجموعات المستخدمين، وتوسعة هيكلية

#### أهداف الإصدار

- **توفير نقاط نهاية لجلب خيارات الحقول** (`/user/options`، `/user/frontend-options`، `/user/backend-options`).
- **إطلاق متحكم `UserGroups`** لإدارة مجموعات المستخدمين عبر API مع فلاتر متوافقة مع الأعمدة الجديدة.
- **تحديث `UserGroupTransformer`** ليشمل الحقول الجديدة (`is_active`، `is_new_user_default`، `sort_order`).
- **تنفيذ هجرة** لإضافة هذه الأعمدة إلى جدول `user_groups`.
- **تحديث ملف الإعدادات** (`config.php`) لإضافة إعدادات خاصة بمجموعات المستخدمين (`user_groups`).

#### الميزات الجديدة

##### 1. نقاط نهاية خيارات الحقول

تم إضافة المتحكم `Profiles` بدوال جديدة لاستخراج جميع الخيارات اللازمة لبناء استمارات المستخدمين:

| المسار | الوصف |
| :--- | :--- |
| `GET /api/v1/user/options` | جلب خيارات عامة (يمكن تحديد الحقول عبر `fields`). |
| `GET /api/v1/user/frontend-options` | نفس الخيارات لكن مخصصة للواجهة الأمامية. |
| `GET /api/v1/user/backend-options` | خيارات مخصصة للوحة التحكم (مستخدمي backend). |

**الدوال المسؤولة:**
- `getOptions($data)`: الدالة الرئيسية التي تستخرج الخيارات بناءً على الحقل `fields` (مثل `gender`, `ref_type`, `language`, `nationality`، `passport_type`...).
- `getAutoptions()`, `getFrontendOptions()`, `getBackendOptions()`: دوال تغليف لتحديد `provider` تلقائياً.
- `getCollectionItem($items)`: تحويل المصفوفات الترابطية إلى مجموعة كائنات `{id, name}` لتنسيق موحد.

**خيارات الحقول المدعومة حاليًا:**

| الحقل | الوصف |
| :--- | :--- |
| `gender_type` | أنواع الجنس الإضافية (إن وجدت) |
| `gender` | خيارات الجنس (ذكر/أنثى) |
| `ref_type` | أنواع الحسابات المتاحة (مستخدم عادي، عامل توصيل...) |
| `language` | قائمة اللغات المدعومة |
| `nationality` | قائمة الجنسيات |
| `passport_type` | أنواع جوازات السفر |
| `marital_status` | الحالة الاجتماعية |
| `fuel_type` | أنواع الوقود |
| `website_type` | أنواع المواقع (website, facebook) |
| `phone_type` | أنواع الهواتف (جوال، منزل، عمل...) |

##### 2. متحكم `UserGroups` لإدارة مجموعات المستخدمين

تم إنشاء `Nano\AuthApi\Controllers\UserGroups` ليوفر:

- **`index()`**: جلب قائمة المجموعات مع دعم الفلاتر (الكود، الحالة، المجموعة الافتراضية، البحث بالنص) والترتيب والتقسيم إلى صفحات.
- **`show($id)`**: جلب تفاصيل مجموعة واحدة.
- **`activelystats()`**: إحصائيات عن آخر تحديث (لأغراض الكاش).

**أبرز مميزات `getRecords()`:**
- دعم `provider` (frontend/backend) لاختيار نموذج المجموعات المناسب.
- فلتر `is_new_user_default` لقصر النتائج على المجموعة الافتراضية للمستخدمين الجدد.
- فلتر `code` و `status` و `q` (بحث في الاسم والوصف والكود).
- دعم خيارات الإرجاع المباشر: `is_collection`, `is_paginator`, `is_query`.
- أحداث `api.list.extendQueryBefore` و `api.list.extendQuery` للسماح للمطورين بتخصيص الاستعلام.
- استخدام `UserGroupTransformer` لتنسيق المخرجات.

##### 3. دعم الأعمدة الجديدة في `user_groups`

تم إضافة الأعمدة التالية عبر هجرة `builder_table_add_is_active_columns_to_frontend_user_groups_table.php`:

| العمود | النوع | الوصف |
| :--- | :--- | :--- |
| `is_new_user_default` | boolean | تعيين المجموعة كافتراضية للمستخدمين الجدد. |
| `is_active` | boolean | تفعيل/تعطيل المجموعة. |
| `sort_order` | integer | ترتيب ظهور المجموعة في القوائم. |

**التحقق من عدم الازدواجية**: تستخدم الشرط `if(!Schema::hasColumn(...))` لتجنب الأخطاء عند تنفيذ التحديث على قواعد بيانات تحتوي الأعمدة مسبقاً.

##### 4. تحديثات في `UserGroupTransformer`

تمت إضافة الحقول الجديدة إلى استجابة JSON لتشمل `is_active`, `is_new_user_default`, `sort_order` إلى جانب الحقول الأساسية (`id`, `name`, `code`, `description`, التواريخ). هذا يضمن توافق جميع نقاط النهاية الخاصة بالمجموعات مع هذه البيانات.

##### 5. إعدادات جديدة في `config.php`

تم إضافة مفتاح `user_groups` إلى ملف `config.php`:

```php
'user_groups' => [
    'is_allow_list' => true,
    'is_allow_list_backend' => true,
    'is_allow_list_frontend' => true,
    'is_check_access_list' => false,
    'is_check_list_permission' => false,
    'order_by' => 'code',
    'order_dir' => 'asc',
    'per_page' => 15,
    'exclude' => '',
],
```

هذه الإعدادات تسمح بالتحكم في سلوك نقاط النهاية ومدى صلاحية الوصول إليها.

---

### أمثلة عملية

#### 1. جلب خيارات الجنس ونوع الحساب

**الطلب:**
```http
GET /api/v1/user/options?fields=gender,ref_type HTTP/1.1
Authorization: Bearer ...
```

**الاستجابة:**
```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب الخيارات بنجاح",
    "data": {
        "gender": [
            {"id": "male", "name": "ذكر"},
            {"id": "female", "name": "أنثى"}
        ],
        "ref_type": [
            {"id": "user", "name": "مستخدم عادى"},
            {"id": "department", "name": "منشأة او شركة"},
            ...
        ]
    }
}
```

#### 2. جلب المجموعات المفعلة فقط مع ترتيب حسب `sort_order`

```http
GET /api/v1/user/groups?is_active=1&orderBy=sort_order&orderDirection=asc HTTP/1.1
```

#### 3. جلب المجموعة الافتراضية للمستخدمين الجدد

```http
GET /api/v1/user/groups?is_new_user_default=1 HTTP/1.1
```

#### 4. جلب تفاصيل مجموعة محددة

```http
GET /api/v1/user/groups/1 HTTP/1.1
```

---

### ملخص الإصدارات (1.0.18 – 1.0.19)

| الإصدار | أبرز الميزات |
| :--- | :--- |
| 1.0.18 | دعم حقول `bio`, `website`, `links` في الملف الشخصي. |
| 1.0.19 | نقاط نهاية لخيارات الحقول (`/user/options`)، متحكم `UserGroups` لإدارة المجموعات، دعم أعمدة `is_new_user_default` و `is_active` و `sort_order`، وتحديث المحولات والإعدادات. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - إضافة المتحكم الجديد `UserGroups.php` في المسار المحدد.
   - إضافة الدوال الجديدة في `Profiles.php` (إن لم تكن موجودة).
   - تحديث `UserGroupTransformer.php` ليتضمن الحقول الجديدة.
   - تحديث ملف `routes.php` لإضافة المسارات الجديدة.

2. **تنفيذ الهجرة**:
   - تشغيل الهجرة `builder_table_add_is_active_columns_to_frontend_user_groups_table.php` لإضافة الأعمدة الجديدة. يمكن استخدام `php artisan october:up` أو ترقية الإضافة من لوحة التحكم.

3. **مراجعة الإعدادات**:
   - تحديث ملف `config.php` لتضمين مفتاح `user_groups` بالإعدادات الافتراضية المناسبة.

4. **اختبار نقاط النهاية الجديدة**:
   - جرب `GET /api/v1/user/options?fields=gender,ref_type,language`.
   - جرب `GET /api/v1/user/groups` مع معلمات مختلفة للفلاتر.

5. **مسح الكاش** (اختياري):
   - `php artisan cache:clear` لضمان تحميل الإعدادات الجديدة.

---

### الخاتمة

مع الإصدار 1.0.19، نكون قد قطعنا خطوة مهمة نحو جعل `Nano.AuthApi` واجهة API شاملة لإدارة المستخدمين ومجموعاتهم. نقاط النهاية الجديدة لتوفير الخيارات تقلل الاعتماد على واجهات منفصلة، بينما توفر `UserGroups` أدوات بحث وتصفية قوية تناسب احتياجات التطبيقات الحديثة.

هذا التكامل يفتح الباب لبناء تجارب تسجيل وتفضيلات ديناميكية بالكامل، ويسهل على المطورين إنشاء لوحات تحكم مرنة تعتمد على API بشكل كامل.

نتطلع إلى ملاحظاتكم ومقترحاتكم لمواصلة تطوير هذه الإضافة الحيوية.

## 2026-05-06 – 2026-05-18

### إضافة وتحديث بوابة دفع BasPay (منصة بس) في نظام المدفوعات بنانوسوفت

ضمن الوحدة البرمجية `Nano.Yepayment`

---

### 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **BasPay** وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع المحلية في اليمن، حيث تُعد **منصة بس (BAS)** إحدى منصات الدفع الإلكتروني التي تتيح قبول المدفوعات عبر واجهة API مباشرة مع توقيع مشفر AES-256-CBC.

بعد الفحص الدقيق لتدفق الدفع في منصة بس، **تم تحديث البوابة** لتعمل وفق نمط **الدفع بخطوتين (Two‑Step Payment)**، وهو النمط الذي يتوافق مع طبيعة عمل المنصة حيث:
1. **الخطوة الأولى (process):** يتم إنشاء معاملة والحصول على `trxToken`، دون خصم المبلغ بعد.
2. **الخطوة الثانية (complete):** بعد أن يكمل العميل الدفع عبر تطبيق بس، يتم التحقق من حالة المعاملة وتحديث الطلب إلى "مدفوع".

هذا التحديث جعل البوابة متوافقة تماماً مع `OrderManager` و `Checkout2` و `PaymentResult`، ويمنع تغيير حالة الطلب إلى مدفوع قبل الأوان.

تم تطوير البوابة وفق أعلى معايير الأمان والجودة، مع تخزين التوكنات في Cache، واستخدام `HttpHelper` الموحد لجميع طلبات API، وتوفير آلية توليد توقيع مطابقة تماماً لمتطلبات منصة بس (SHA256 + AES-256-CBC)، بالإضافة إلى واجهات اختبار متكاملة ونقاط نهاية API مخصصة للمطورين والمسؤولين. كما تم دعم `RedirectHelper` لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى روابط عودة بعد الدفع.

---

### 2. المكونات المطورة

#### 2.1. `BasPay` – كلاس بوابة الدفع الرئيسي

- **المسار:** `Nano\Yepayment\PaymentTypes\BasPay`
- **الوراثة:** يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة:** مسؤول عن المصادقة مع منصة بس عبر OAuth2، تنفيذ الخطوة الأولى (إنشاء معاملة)، والخطوة الثانية (التحقق من الحالة وتأكيد الدفع)، وتوليد التوقيع المشفر.

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | **الخطوة الأولى:** إنشاء معاملة عبر `initiateTransaction`، استلام `trxToken`، وحفظه في الطلب دون تغيير حالة الدفع. تُعيد نجاحاً مع رسالة تأكيد. |
| `complete(PaymentResult $result)` | **الخطوة الثانية:** التحقق من حالة المعاملة عبر `checkTransactionStatusByToken`، وفي حال كانت `SUCCESS` يتم استدعاء `$result->success()` لتحديث الطلب إلى `PaidState`. |
| `checkAndCompletePay(array $options): array` | دالة `static` عامة تُستخدم في مسار `baspay/success` لربط العودة من التطبيق بعملية التأكيد. |
| `getAuthToken(): ?string` | طلب توكن OAuth 2.0 من BAS عبر `POST /api/v1/auth/token` (grant_type: client_credentials)، وتخزينه في Cache لمدة 3500 ثانية. |
| `initiateTransaction($token): array` | إرسال طلب `POST /api/v1/merchant/sdk-payment/initiate-transaction` مع جسم الطلب والتوقيع. |
| `checkTransactionStatusByToken($token, $trxToken): array` | الاستعلام عن حالة المعاملة عبر `POST /api/v1/merchant/sdk-payment/get-transaction-status` (تُستخدم داخلياً). |
| `checkTransactionStatus($trxToken): array` | دالة عامة للاستعلام عن حالة معاملة (للاستخدام الخارجي). |
| `generateSignature($paramsString): string` | توليد توقيع BAS: salt عشوائي (4 أحرف)، SHA256، ثم تشفير AES-256-CBC باستخدام Merchant Key و IV. |
| `encryptAes256Cbc($input, $key, $iv): string` | دالة التشفير AES-256-CBC باستخدام OpenSSL. |
| `parseResponse($response): array` | تحويل استجابة HTTP إلى مصفوفة PHP. |
| `getCommonHeaders(): array` | إرجاع الهيدرات المطلوبة لكل طلب (x-client-id, x-app-id, x-sdk-version, ...). |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL, Client ID, Client Secret, App ID, Merchant Key, IV, العملة الافتراضية). |
| `encryptedSettings(): array` | تحديد الحقول التي تخزن بشكل مشفر (`baspay_client_secret`, `baspay_merchant_key`). |

#### 2.2. ملفات جزئية (Partials) وإعدادات

| الملف | المسار | الوصف |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/baspay/_info.htm` | عرض إرشادات الإعداد ومعلومات البوابة في لوحة التحكم. |
| `_test_info.htm` | `paymenttypes/baspay/_test_info.htm` | أدوات اختبار سريع داخل صفحة الإعدادات (أزرار المصادقة، إنشاء دفعة، التحقق من الحالة). |
| `baspay-ui.htm` | `views/baspay-ui.htm` | واجهة ويب تفاعلية متكاملة لاختبار جميع وظائف BasPay (يدوي، تلقائي، إحصائيات، سجلات). |

#### 2.3. نقاط نهاية API (`routes.php`)

تم إضافة مجموعة كاملة من المسارات ضمن مجموعة `yepayment` لدعم الاختبار والمراقبة:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/baspay/test-auth` | POST | اختبار المصادقة والحصول على Bearer Token. |
| `/baspay/test-create-payment` | POST | إنشاء معاملة جديدة (الخطوة الأولى). |
| `/baspay/test-check-status` | POST | التحقق من حالة معاملة باستخدام `trx_token`. |
| `/baspay/test-full-payment` | POST | اختبار شامل (إنشاء معاملة + التحقق من الحالة). |
| `/baspay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح). |
| `/baspay/test-ui` | GET | واجهة الاختبار المتكاملة (HTML). |
| `/baspay/success` | GET | نقطة نهاية لتأكيد الدفع بعد عودة المستخدم من تطبيق بس. |
| `/baspay/cancel` | GET | نقطة نهاية اختيارية لإعادة التوجيه عند الإلغاء. |

جميع المسارات (باستثناء success/cancel) محمية بـ `BackendAuth::getUser()` لضمان وصول المسؤولين فقط.

#### 2.4. `Plugin.php` – تسجيل مزود الدفع

تمت إضافة السطر التالي في دالة `registerPaymentProviders()` ضمن قسم `allow_yemen_payment`:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\BasPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\BasPay();
}
```

#### 2.5. ملفات اللغة (Translation)

تم إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php` لكل من الإعدادات ورسائل العموم والأخطاء (مثل `payment_success`, `auth_failed`, `order_already_paid`).

---

### 3. دورة عمل الدفع (Payment Workflow) - نمط الخطوتين

تعمل BasPay وفق تدفق **Two‑Step (خطوتين)**:

1. **المصادقة (Authentication)**  
   يتم استدعاء `getAuthToken()`:
   - التحقق من وجود `access_token` في Cache.
   - إذا لم يكن موجوداً، يتم إرسال طلب `POST /api/v1/auth/token` مع `client_id`، `client_secret`، و `grant_type=client_credentials`.
   - يتم تخزين التوكن في Cache لمدة 3500 ثانية.

2. **الخطوة الأولى – إنشاء المعاملة (`process`)**  
   - التحقق من أن الطلب غير مدفوع مسبقاً.
   - بناء جسم الطلب (`amount`، `currency`، `orderId`، `appId`، `requestTimestamp`).
   - توليد التوقيع عبر `generateSignature()`.
   - إرسال طلب `POST /api/v1/merchant/sdk-payment/initiate-transaction`.
   - استلام `trxToken` وتخزينه في `order.payment_first_trans_id` و `order.other_data['baspay']`.
   - **لا يتم استدعاء `$result->success()`**، ولا يتغير الطلب إلى `PaidState`. بدلاً من ذلك، تُرجع البوابة `successful = true` مع رسالة `confirmation_required`.

3. **الخطوة الثانية – تأكيد الدفع (`complete`)**  
   - تُستدعى من مسار `baspay/success` بعد عودة المستخدم من تطبيق بس.
   - تستخرج `trxToken` من الطلب، وتتواصل مع BAS عبر `POST /api/v1/merchant/sdk-payment/get-transaction-status`.
   - إذا كانت `trxStatus` تساوي `SUCCESS` أو `COMPLETED`:
     - يتم استدعاء `$result->success()` التي **تغير حالة الطلب إلى `PaidState`** وتُشغل الأحداث (مثل `nano.orders.paymentProcessed`).
   - إذا كانت `PENDING`: يتم استدعاء `$result->pending()` وتبقى حالة الطلب معلقة.

4. **روابط العودة الاختيارية**  
   عند تمرير `callback_success_url` أو `callback_error_url`، يتم تخزينها في `other_data`. بعد التأكيد، يمكن للمطور استخدام `RedirectHelper` لإعادة توجيه المستخدم بذكاء (Deep Link أو ويب).

---

### 4. الإعدادات والتهيئة (Configuration)

لتفعيل BasPay، يجب إدخال الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| الإعداد | المفتاح | الوصف | القيمة الافتراضية |
|---------|---------|-------|-------------------|
| رابط API الأساسي | `baspay_url` | عنوان API منصة بس | `https://api.basgate.com` |
| Client ID | `baspay_client_id` | معرف العميل من منصة بس | - |
| Client Secret | `baspay_client_secret` | المفتاح السري (مخزن مشفر) | - |
| App ID | `baspay_app_id` | معرف التطبيق | - |
| Merchant Key | `baspay_merchant_key` | مفتاح التاجر للتوقيع (مخزن مشفر) | - |
| IV | `baspay_iv` | قيمة المتجه الأولي (16 بايت) | `@@@@&&&&####$$$$` |
| العملة الافتراضية | `baspay_default_currency` | العملة المستخدمة إذا لم يحدد الطلب عملة | `YER` |

**ملاحظات أمنية:**
- `baspay_client_secret` و `baspay_merchant_key` يتم تخزينهما مشفرين عبر `encryptedSettings()`.
- لا يتم تسجيل هذه القيم الحساسة في السجلات.

---

### 5. الأمثلة التوضيحية (Usage Examples)

#### 5.1. بدء الدفع – الخطوة الأولى (إنشاء المعاملة)

```php
use Nano\Yepayment\PaymentTypes\BasPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$bas = new BasPay($order, [
    'callback_success_url' => 'myapp://pay/success',   // اختياري
]);
$result = new PaymentResult($bas, $order);
$processResult = $bas->process($result);

if ($processResult->successful) {
    // تم إنشاء المعاملة وحفظ trxToken، لكن الدفع لم يكتمل بعد
    $trxToken = $order->payment_first_trans_id;
    // يجب توجيه المستخدم لتطبيق بس لإكمال الدفع
}
```

#### 5.2. تأكيد الدفع – الخطوة الثانية (التحقق من الحالة)

بعد أن يكمل العميل الدفع في تطبيق بس، يمكن تأكيد العملية كالتالي:

```php
$result = \Nano\Yepayment\PaymentTypes\BasPay::checkAndCompletePay(['order_id' => 200]);
if ($result['success']) {
    // تم تأكيد الدفع وتحديث الطلب إلى PaidState
}
```

#### 5.3. الاستعلام المباشر عن حالة معاملة

```php
$bas = new BasPay();
$status = $bas->checkTransactionStatus('bas_trx_a1b2c3d4...');
if ($status['success']) {
    // $status['data'] يحتوي على تفاصيل المعاملة
}
```

#### 5.4. استخدام واجهة الاختبار المتكاملة

بعد تسجيل الدخول إلى لوحة التحكم كمسؤول، افتح الرابط:
```
https://yourdomain.com/api/v1/yepayment/baspay/test-ui
```

الواجهة تتيح:
- إنشاء معاملة جديدة (Initiate).
- التحقق من الحالة.
- اختبار تلقائي بعدد مرات محدد (حتى 10).
- عرض إحصائيات الاستخدام (عدد الطلبات، نسبة النجاح).
- سجلات محلية للاختبارات.

---

### 6. التعامل مع عمليات إعادة التوجيه (Deeplinks & Callbacks)

تم دمج دعم **اختياري** لروابط إعادة التوجيه باستخدام `RedirectHelper`، لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى فتح شاشة معينة بعد الدفع.

- عند تمرير `callback_success_url` أو `callback_error_url` في بيانات الدفع، يتم تخزينهما في `order.other_data['baspay']`.
- بعد تأكيد الدفع عبر `complete()`، يمكن للمطور توجيه المستخدم إلى `/baspay/success` (مع `order_id`) لتقوم البوابة باستخراج رابط العودة، وتنفيذ إعادة التوجيه الذكي (deeplink أو ويب) مع تمرير بيانات الحالة.
- نفس الآلية تنطبق على `/baspay/cancel`.

---

### 7. القيمة المضافة

- **للمطورين:** نموذج شامل لبوابة دفع من نوع **Two-Step** يستخدم OAuth2 مع توقيع مشفر AES-256-CBC، مما يوضح كيفية بناء بوابات تتطلب تأكيداً خارجياً دون SDK. كما يُظهر التكامل الكامل مع `OrderManager` و `PaymentResult` و `RedirectHelper`.
- **للتجار:** دعم منصة بس المحلية في اليمن، مما يفتح المجال لقبول المدفوعات الإلكترونية بسهولة عبر حساب التاجر في بس.
- **للمستخدمين النهائيين:** تجربة دفع موثوقة تتم على خطوتين: تأكيد الطلب في المتجر، ثم إكمال الدفع في تطبيق بس.

---

### 8. اختبار التكامل (Integration Testing)

تم توفير عدة طبقات للاختبار:

- **بيئة الاختبار:** يمكن استخدام نفس رابط الإنتاج (`https://api.basgate.com`) مع بيانات اعتماد تجريبية من منصة بس. إذا توفرت بيئة اختبار مستقبلية، يمكن إدخال رابطها في حقل `baspay_url`.
- **واجهة الاختبار السريع:** من خلال صفحة إعدادات البوابة (الملف الجزئي `_test_info.htm`) يمكن اختبار المصادقة، إنشاء معاملة، والتحقق من الحالة مباشرة.
- **واجهة الاختبار المتكاملة:** `/api/v1/yepayment/baspay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة بشكل كامل (بما في ذلك محاكاة الخطوتين)، مع إمكانية الاختبار التكراري (Automated Testing).
- **نقاط نهاية API مستقلة:** يمكن للمطورين استخدام أدوات مثل Postman أو cURL للاتصال المباشر بالنقاط مثل `/baspay/test-full-payment`.

**خطوات اختبار سريعة:**
1. تأكد من إعداد Client ID، Client Secret، App ID، Merchant Key، IV في صفحة إعدادات البوابة.
2. افتح `/api/v1/yepayment/baspay/test-ui`.
3. انقر على "اختبار المصادقة" للتحقق من صحة البيانات.
4. أدخل رقم طلب ومبلغ، ثم انقر "إنشاء معاملة جديدة".
5. ستظهر `trxToken` – يمكنك استخدامها في "التحقق من الحالة" لمحاكاة الخطوة الثانية.

---

### 9. ملاحظات للمطورين (Developer Notes)

- **نمط الخطوتين:** `process()` **لا** يُكمل الدفع، بل ينشئ المعاملة فقط. يجب استدعاء `complete()` أو `checkAndCompletePay` لتأكيد الدفع وتحديث الطلب إلى `PaidState`.
- **التخزين المؤقت للتوكن:** يتم تخزين Bearer Token في Cache لمدة 3500 ثانية (أقل من صلاحية التوكن الفعلية المقدّرة بـ 3600 ثانية).
- **التوقيع المشفر:** تم تنفيذ آلية التوقيع يدوياً باستخدام OpenSSL دون الاعتماد على حزمة خارجية، وذلك لضمان التوافق مع البيئة الخاصة بنانوسوفت ومنع التعارضات.
- **استخدام `HttpHelper`:** جميع طلبات API (JSON، Form) تستخدم `HttpHelper` الموحد، مما يسهل تتبع الأخطاء وتوسيع البوابة.
- **دعم `RedirectHelper`:** تم تضمين إمكانية إعادة التوجيه بشكل كامل، مما يجعل البوابة مناسبة لتطبيقات الجوال التي تستخدم Deep Links.
- **قابلية التوسع:** يمكن إضافة دعم Webhook أو استرداد المبالغ مستقبلاً عبر دوال إضافية تستخدم نقاط نهاية BAS المختلفة.

---

### 10. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة وتحديثها فقط (BasPay).

---

### 11. فترة التطوير والاختبار

تم تطوير بوابة BasPay في البداية كدفع مباشر، ثم **تم تحديثها** إلى نمط الخطوتين خلال الفترة من **6 مايو 2026** وحتى **18 مايو 2026**. تضمنت هذه الفترة كتابة الكود الأساسي للكلاس BasPay وتطبيق آلية التوقيع المشفر AES-256-CBC، وإنشاء ملفات الواجهة الجزئية (_info.htm, _test_info.htm) وواجهة الاختبار المتكاملة (baspay-ui.htm)، بالإضافة إلى كتابة نقاط نهاية API اللازمة للاختبار والمراقبة، وتعديل `process()` و`complete()` لتتوافق مع تدفق الخطوتين. تم أيضاً إجراء اختبارات تكامل شاملة للتأكد من صحة عمل البوابة مع بيئة منصة بس، وتم التحقق من توافق التوقيع مع الردود المستلمة.

---

### 12. روابط ذات صلة

- [توثيق BasPay (دليل المطور)](./docs/BasPay/Docs-BasPay-ar.md)
- [وثيقة SKILL-ar.md لإنشاء بوابات الدفع](./docs/BasPay/SKILL-ar.md)
- [دليل منصة بس API](https://basgate.apidog.io)
- [دليل منصة بس API](./docs/BasPay/external/BAS/README-ar.md)
- [مستودع وثائق BasGate على GitHub](https://github.com/basgate/basgate.github.io)
- [حزمة Laravel Payment SDK على GitHub](https://github.com/basgate/laravel-payment-sdk)
- [مستودع BasPaymentFlutter على GitHub](https://github.com/BasPlatform/BasPaymentFlutter.git)
- [حزمة bas_php_sdk على GitHub](https://github.com/basgate/bas_php_sdk)
- [حزمة bas-laravel-sdk على GitHub](https://github.com/basgate/bas-laravel-sdk)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft

## 2026-05-18 - 2026-05-19

### تحديث شامل لنطاقات الاستعلام في سلوك `ProposalModel`

**تطوير كامل لنطاقات (Scopes) الاستعلام والوظائف المساعدة ضمن حزمة `Nano2.Proposals`**

تم تنفيذ حزمة تطويرية متكاملة تهدف إلى تحسين أداء ومرونة الاستعلامات المتعلقة بنظام المقترحات والشكاوى والبلاغات (Proposals/Reports/Complaints) في تطبيقات نانوسوفت. لم يعد المطور بحاجة إلى كتابة استعلامات معقدة أو الاعتماد على `GROUP BY` و `LIMIT` بشكل خاطئ داخل الاستعلامات الفرعية، بل أصبح لديه مجموعة من النطاقات الجاهزة التي تتيح:

- الترتيب حسب **عدد البلاغات** (مع إمكانية تمييز أنواع البلاغات: `proposals`, `reports`, `complaints`).
- الترتيب حسب **أحدث تاريخ بلاغ** (الأحدث).
- الترتيب حسب **أقدم تاريخ بلاغ** (الأقدم).
- إضافة أعمدة محسوبة إلى `SELECT` (مثل `proposals_count`, `latest_proposal_at`) دون التأثير على الترتيب.
- تصفية الكائنات التي أرسل لها المستخدم بلاغاً (كـ `target`).
- تصفية الكائنات التي أرسلت بلاغاً (كـ `user`).
- تصفية الكائنات التي لديها بلاغات (مع حد أدنى للعدد) أو التي لا بلاغات لها.
- إضافة عمود `is_proposed_by_user` (هل أرسل المستخدم بلاغاً للكائن) و `is_proposed_to_user` (هل الكائن هو هدف بلاغ أرسله المستخدم).
- نطاق متقدم (`scopeWhereHasProposalAdvanced`) يدعم الجانبين (`target` و `user`) بمرونة كاملة، مع التحكم في شروط `user_id`/`user_type` و `target_id`/`target_type`، واستخدام `has`/`orHas`/`doesntHave`/`orDoesntHave`، ودعم `withTrashed`/`onlyTrashed` ككلمات مفتاحية خاصة.
- دوال إحصائية متقدمة للحصول على إجمالي البلاغات، توزيعها حسب نوع المستخدم، وقائمة المستخدمين المبلّغين.

تمت إعادة هيكلة النطاقات الموجودة سابقاً وإضافة نطاقات جديدة، مع الاحتفاظ بالتوافق العكسي عبر دوال تغليف موصوفة بـ `@deprecated`. تم نقل جميع النطاقات والدوال المساعدة إلى ترايت مستقل `ProposalScopesAndHelpers` لتسهيل الصيانة وإعادة الاستخدام.

---

### 1. المكونات المطورة

| السلوك | المكون | الوصف |
|--------|--------|-------|
| `ProposalModel` | `ProposalScopesAndHelpers` (trait) | يحتوي على جميع النطاقات المتقدمة والدوال المساعدة لنظام البلاغات. |

#### النطاقات الجديدة والمحسنة

| الفئة | النطاق | الوصف |
|-------|--------|-------|
| **عدد البلاغات** | `scopeAddCountProposals` | إضافة عمود عدد البلاغات إلى `SELECT` (بدون ترتيب). |
| | `scopeSortByCountProposals` | ترتيب النتائج حسب عدد البلاغات (دون إضافة العمود). |
| | `scopeWithCountProposals` | إضافة العمود والترتيب معاً. |
| **أحدث تاريخ بلاغ** | `scopeAddLatestProposal` | إضافة عمود بأحدث تاريخ بلاغ. |
| | `scopeSortByLatestProposal` | ترتيب حسب أحدث تاريخ بلاغ. |
| | `scopeWithLatestProposal` | إضافة العمود والترتيب معاً. |
| **أقدم تاريخ بلاغ** | `scopeAddEarliestProposal` | إضافة عمود بأقدم تاريخ بلاغ. |
| | `scopeSortByEarliestProposal` | ترتيب حسب أقدم تاريخ بلاغ. |
| | `scopeWithEarliestProposal` | إضافة العمود والترتيب معاً. |
| **التصفية حسب المستخدم (كهدف)** | `scopeProposedByUser` | تصفية الكائنات التي أرسل لها المستخدم بلاغاً. |
| | `scopeNotProposedByUser` | تصفية الكائنات التي لم يرسل لها المستخدم بلاغاً. |
| **التصفية حسب المستخدم (كمرسل)** | `scopeProposedToUser` | تصفية الكائنات التي أرسلها المستخدم بلاغاً (أي الكائن هو المرسل). |
| | `scopeNotProposedToUser` | تصفية الكائنات التي لم يرسلها المستخدم بلاغاً. |
| **التصفية حسب وجود بلاغات** | `scopeHasProposals` | تصفية الكائنات التي لديها بلاغات (مع حد أدنى). |
| | `scopeHasNoProposals` | تصفية الكائنات التي لا بلاغات لها. |
| **عمود الحالة للمستخدم** | `scopeWithIsProposedByUser` | إضافة عمود `is_proposed_by_user`. |
| | `scopeWithIsProposedToUser` | إضافة عمود `is_proposed_to_user`. |
| **نطاق متقدم** | `scopeWhereHasProposalAdvanced` | نطاق مرن للتصفية حسب البلاغات مع دعم كامل للجانبين (target/user)، شروط متقدمة، أنواع علاقات متعددة (`has`, `orHas`, `doesntHave`, `orDoesntHave`). |
| **نطاقات خاصة** | `scopeTopProposed` | جلب الكائنات الأكثر بلاغاً (بعدد البلاغات). |

#### الدوال الإحصائية المساعدة

| الدالة | الوصف |
|--------|-------|
| `getTotalProposals` | إجمالي عدد البلاغات (مجموع `COUNT`) مع خيارات التصفية. |
| `getProposalsCountByType` | توزيع البلاغات حسب نوع المستخدم (`user_type`). |
| `getProposersUsers` | قائمة المستخدمين الذين أرسلوا بلاغات للكائن. |

جميع الدوال والنطاقات تدعم خيارات التصفية: `$type` (نوع البلاغ), `$onlyActive`, `$withTrashed`.

---

### 2. تفاصيل التحديثات البرمجية

#### 2.1 إصلاح أخطاء الاستعلامات الفرعية
كانت النطاقات القديمة في السلوك الأصلي تعتمد على استخدام `GROUP BY` و `LIMIT` داخل الاستعلامات الفرعية للحصول على قيم مجمعة (مثل `COUNT`, `MAX`). هذا الأسلوب يؤدي إلى نتائج غير صحيحة وعشوائية.

**النهج الجديد**:
- استخدام دالة تجميعية مباشرة في الاستعلام الفرعي دون `GROUP BY` أو `LIMIT`.
- كتابة اسم الجدول بالكامل أمام كل حقل لتجنب الغموض.
- إضافة شروط إضافية (مثل `type`, `is_active`) عبر `where` داخل الاستعلام الفرعي.

#### 2.2 توحيد واجهة النطاقات
تم توحيد توقيع الدوال لتكون:
- `$orderDirection`: اتجاه الترتيب (افتراضي `DESC`).
- `$columnName`: اسم العمود المضاف (افتراضي مناسب مثل `proposals_count`, `latest_proposal_at`).
- `$type`: تصفية حسب نوع البلاغ (`proposals`, `reports`, `complaints`).
- `$onlyActive`: تصفية حسب `is_active`.
- `$withTrashed`: تضمين السجلات المحذوفة (Soft Deletes).

#### 2.3 إضافة النطاق المتقدم `scopeWhereHasProposalAdvanced`
هذا النطاق هو القلب الجديد للتصفية حسب البلاغات، ويدعم:
- **الجانب المطلوب** (`side`): `target` (بلاغات واردة إلى الكائن) أو `user` (بلاغات صادرة من الكائن).
- **نوع العلاقة** (`type`): `has`, `orHas`, `doesntHave`, `orDoesntHave`.
- **شروط مرنة** (`conditions`): يمكن أن تكون مصفوفة من الحقول والقيم، أو دالة إغلاق، أو مصفوفة تدعم الاستعلامات المتقدمة (مثل `['whereIn', [1,2,3]]`).
- **التحكم في تطبيق شروط المستخدم/الهدف** (`useUserConditions`, `useTargetConditions`): يمكن ضبطها على `true` (تطبيق الكل)، `false` (لا تطبيق)، أو مصفوفة تحتوي على أسماء الحقول المطلوبة (مثل `['user_type']`).
- **التحكم في فرض الشروط عند عدم وجود كيان** (`isForceUser`, `isForceTarget`): إذا كان `true` ولم يوجد المستخدم (أو الهدف) وكانت شروطه مطلوبة، يضاف شرط مستحيل (`whereRaw('1 = 0')`) لعدم إرجاع نتائج. إذا كان `false`، لا يضاف شرط.
- **مصدر الحصول على المستخدم/الهدف** (`userSource`, `targetSource`): يمكن أن يكون `'current_user'` (جلب من AuthHelpers) أو `'current_model'` (استخدام الموديل الحالي). يقرأ من الإعدادات أو يمكن تمريره.
- **دعم `withTrashed` و `onlyTrashed`**: يتم معالجتهما ككلمات مفتاحية خاصة داخل `conditions`، ويُستدعى النطاق المناسب على الاستعلام الفرعي.

#### 2.4 دعم `withTrashed` و `onlyTrashed` ككلمات مفتاحية
في `$conditions`، عند تمرير `'withTrashed' => true` أو `'onlyTrashed' => true`، يتم استدعاء الدالة المناسبة (`withTrashed()` أو `onlyTrashed()`) على الاستعلام الفرعي بدلاً من معاملتها كشرط `where`.

#### 2.5 نقل النطاقات إلى ترايت منفصل
لتسهيل الصيانة وإعادة الاستخدام، تم نقل جميع النطاقات والدوال المساعدة إلى ترايت جديد:  
`Nano2\Proposals\Behaviors\ProposalModel\ProposalScopesAndHelpers`

#### 2.6 التوافق العكسي
تم الاحتفاظ بالنطاقات القديمة كدوال تغليف في الكلاس الرئيسي، مع إضافة تعليق `@deprecated`. كما تم توفير دوال `scopeWhereHasProposal`, `scopeWhereHasProposals`, `scopeWhereHasUserProposal`, `scopeWhereHasUserProposals` التي تُعيد توجيه المستخدمين إلى النطاق المتقدم.

**قائمة الدوال القديمة المدعومة:**
- `scopeSortByCountProposalsOld` → `scopeSortByCountProposals`
- `scopeWithSortByCountProposals` → `scopeWithCountProposals`
- `scopeAddSortByCountProposals` → `scopeAddCountProposals`
- `scopeSortByCreatedAtProposals` → `scopeSortByLatestProposal`
- `scopeAddSortByCreatedAtProposals` → `scopeAddLatestProposal`
- `scopeWithSortByCreatedAtProposals` → `scopeWithLatestProposal`
- `scopeWhereHasProposal` → `scopeWhereHasProposalAdvanced` (جانب target)
- `scopeWhereHasProposals` → `scopeWhereHasProposalAdvanced`
- `scopeWhereHasUserProposal` → `scopeWhereHasProposalAdvanced` (جانب user)
- `scopeWhereHasUserProposals` → `scopeWhereHasProposalAdvanced`

---

### 3. أمثلة تطبيقية

#### 3.1 الترتيب حسب عدد البلاغات
```php
// المنتجات الأكثر بلاغاً (جميع الأنواع)
$products = Product::sortByCountProposals('DESC')->get();

// المنتجات الأكثر بلاغاً من نوع 'complaints'
$products = Product::sortByCountProposals('DESC', 'complaints')->get();

// إضافة عمود عدد البلاغات مع الترتيب
$products = Product::withCountProposals('DESC', 'proposals_count', 'reports')->get();
```

#### 3.2 الترتيب حسب التاريخ
```php
// المنتجات الأحدث بلاغاً (جميع الأنواع)
$products = Product::sortByLatestProposal('DESC')->get();

// المنتجات الأحدث بلاغاً من نوع 'reports'
$products = Product::sortByLatestProposal('DESC', 'latest_report_at', 'reports')->get();

// إضافة عمود آخر تاريخ بلاغ مع الترتيب
$products = Product::withLatestProposal('DESC', 'last_proposal_date')->get();
```

#### 3.3 التصفية حسب المستخدم (جانب target)
```php
// المنتجات التي أرسل لها المستخدم الحالي بلاغاً
$products = Product::proposedByUser()->get();

// المنتجات التي أرسل لها مستخدم محدد بلاغاً من نوع 'complaints'
$user = User::find(1);
$products = Product::proposedByUser($user, 'complaints')->get();

// المنتجات التي لم يرسل لها المستخدم الحالي بلاغاً
$products = Product::notProposedByUser()->get();
```

#### 3.4 التصفية حسب المستخدم (جانب user – الكائن هو المرسل)
```php
// المستخدمون الذين أرسلوا بلاغات (كمرسلين)
$users = User::proposedToUser()->get();

// المستخدمون الذين أرسلوا بلاغات من نوع 'reports'
$users = User::proposedToUser(null, 'reports')->get();
```

#### 3.5 التصفية حسب وجود بلاغات
```php
// المنتجات التي لها أكثر من 5 بلاغات من نوع 'complaints'
$products = Product::hasProposals(5, 'complaints')->get();

// المنتجات التي لا بلاغات لها
$products = Product::hasNoProposals()->get();
```

#### 3.6 إضافة عمود حالة للمستخدم الحالي
```php
$products = Product::withIsProposedByUser()->get();
foreach ($products as $product) {
    echo $product->is_proposed_by_user ? 'أرسلت بلاغاً' : 'لم ترسل بلاغاً';
}
```

#### 3.7 الاستخدام المتقدم للنطاق `scopeWhereHasProposalAdvanced`

```php
// المنتجات التي لديها بلاغات من نوع 'reports' (أي مستخدم)
$products = Product::whereHasProposalAdvanced([
    'conditions' => ['type' => 'reports']
])->get();

// المنتجات التي لديها بلاغات نشطة من نوع 'complaints' للمستخدم الحالي
$products = Product::whereHasProposalAdvanced([
    'side' => 'target',
    'conditions' => ['type' => 'complaints', 'is_active' => true],
])->get();

// المنتجات التي ليس لها بلاغات من نوع 'proposals'
$products = Product::whereHasProposalAdvanced([
    'type' => 'doesntHave',
    'conditions' => ['type' => 'proposals']
])->get();

// استخدام دالة إغلاق لشروط معقدة
$products = Product::whereHasProposalAdvanced([
    'conditions' => function($q) {
        $q->where('type', 'reports')
          ->where('created_at', '>=', Carbon::now()->subWeek());
    }
])->get();

// استخدام whereIn على نوع البلاغ
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => ['whereIn', ['reports', 'complaints']],
        'is_active' => true,
    ]
])->get();

// التحكم في تطبيق شروط المستخدم (جانب target)
$products = Product::whereHasProposalAdvanced([
    'useUserConditions' => false, // لا نطبق user_id/user_type
    'conditions' => ['type' => 'reports']
])->get();

// البحث عن بلاغات من نوع معين مع تضمين المحذوفة
$products = Product::whereHasProposalAdvanced([
    'conditions' => [
        'type' => 'complaints',
        'withTrashed' => true,
    ]
])->get();

// التحكم في مصدر المستخدم عند side=user
$users = User::whereHasProposalAdvanced([
    'side' => 'user',
    'userSource' => 'current_model', // نستخدم الموديل الحالي كمستخدم
])->get();
```

#### 3.8 الدوال الإحصائية
```php
$product = Product::find(1);
echo "إجمالي البلاغات من نوع 'reports': " . $product->getTotalProposals('reports');
print_r($product->getProposalsCountByType('reports')->toArray());
$users = $product->getProposersUsers('reports');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

---

### 4. القيمة المضافة

- **للمطورين**: مجموعة متكاملة من النطاقات الجاهزة توفر الوقت وتقلل الأخطاء. الواجهة الموحدة تسهل التعلم والاستخدام. النطاق المتقدم `scopeWhereHasProposalAdvanced` يلبي جميع حالات الاستخدام المعقدة.
- **للمستخدمين النهائيين**: إمكانية تقديم قوائم مرتبة بذكاء (الأكثر بلاغاً، الأحدث بلاغاً) مما يحسن تجربة المستخدم.
- **للنظام**: أداء أفضل من خلال استعلامات SQL مُحسّنة، والقضاء على الاستعلامات غير الفعالة التي كانت تستخدم `GROUP BY` و `LIMIT` بشكل خاطئ.
- **المرونة**: إمكانية التصفية حسب نوع البلاغ، والنشاط، والمستخدم، والجانب (target/user)، مع التحكم الكامل في شروط المستخدم/الهدف.
- **قابلية التوسع**: إضافة نطاقات جديدة لأي سلوك مستقبلي يتم بنفس النمط، مما يضمن اتساق الكود وسهولة صيانته.

---

### 5. الخاتمة

يمثل هذا التحديث نقلة نوعية في إدارة استعلامات البلاغات والمقترحات والشكاوى داخل تطبيقات نانوسوفت. من خلال إعادة هيكلة النطاقات ونقلها إلى ترايت مستقل، مع إضافة وظائف متقدمة للترتيب والتصفية والإحصائيات، أصبح بإمكان المطورين بناء أنظمة بلاغات متطورة بسهولة وأداء عالٍ. التوافق العكسي يضمن سلاسة الانتقال للمشاريع القائمة، بينما تفتح الإضافات الجديدة آفاقاً واسعة لتجارب مستخدم غنية.

---

**ملاحظة**: لمزيد من التفاصيل حول كل نطاق وطريقة استخدامه، يمكن الرجوع إلى ملف التوثيق الخاص بالسلوك `ProposalModel` أو مراجعة الأمثلة المقدمة أعلاه.

See [docs/ProposalModel/Docs-ProposalModel-ar.md](./docs/ProposalModel/Docs-ProposalModel-ar.md)

See [docs/ProposalModel/Docs-ProposalModel-Advenced-Examples-ar.md](./docs/ProposalModel/Docs-ProposalModel-ar.md)

## 2026-05-19 - 2026-05-20

### تحديث شامل لنطاقات الاستعلام في سلوك `VisitModel`

**تطوير كامل لنطاقات (Scopes) الاستعلام والوظائف المساعدة ضمن حزمة `Nano2.Visitors`**

تم تنفيذ حزمة تطويرية متكاملة تهدف إلى تحسين أداء ومرونة الاستعلامات المتعلقة بنظام الزيارات والمشاهدات (Visits & Views) في تطبيقات نانوسوفت. لم يعد المطور بحاجة إلى كتابة استعلامات معقدة أو الاعتماد على `GROUP BY` و `LIMIT` بشكل خاطئ داخل الاستعلامات الفرعية، بل أصبح لديه مجموعة من النطاقات الجاهزة التي تتيح:

- الترتيب حسب **عدد الزيارات** (عدد السجلات).
- الترتيب حسب **مجموع الزيارات** (حقل `visits`).
- الترتيب حسب **مجموع المشاهدات** (حقل `views`).
- الترتيب حسب **أحدث تاريخ زيارة** (الأحدث).
- الترتيب حسب **أقدم تاريخ زيارة** (الأقدم).
- إضافة أعمدة محسوبة إلى `SELECT` (مثل `visits_count`، `sum_visits`، `latest_visit_at`) دون التأثير على الترتيب.
- تصفية الكائنات التي زارها (أو لم يزرها) مستخدم معين.
- تصفية الكائنات التي لديها (أو ليس لديها) زيارات مع إمكانية تحديد حد أدنى للعدد.
- إضافة عمود `is_visited_by_user` الذي يوضح ما إذا كان المستخدم الحالي قد زار الكائن.
- دوال إحصائية متقدمة للحصول على إجمالي الزيارات، المشاهدات، توزيع الزيارات حسب نوع المستخدم، وقائمة المستخدمين الزائرين.

تمت إعادة هيكلة النطاقات الموجودة سابقاً وإضافة نطاقات جديدة، مع الاحتفاظ بالتوافق العكسي عبر دوال تغليف موصوفة بـ `@deprecated`. تم نقل جميع النطاقات والدوال المساعدة إلى ترايت مستقل `VisitScopesAndHelpers` لتسهيل الصيانة وإعادة الاستخدام.

---

### 1. المكونات المطورة

| السلوك | المكون | الوصف |
|--------|--------|-------|
| `VisitModel` | `VisitScopesAndHelpers` (trait) | يحتوي على جميع النطاقات المتقدمة والدوال المساعدة لنظام الزيارات والمشاهدات. |

#### النطاقات الجديدة والمحسنة

| الفئة | النطاق | الوصف |
|-------|--------|-------|
| **عدد الزيارات** | `scopeAddCountVisits` | إضافة عمود عدد الزيارات إلى `SELECT` (بدون ترتيب). |
| | `scopeSortByCountVisits` | ترتيب النتائج حسب عدد الزيارات (دون إضافة العمود). |
| | `scopeWithCountVisits` | إضافة العمود والترتيب معاً. |
| **مجموع الزيارات** | `scopeAddSumVisits` | إضافة عمود مجموع الزيارات (حقل `visits`). |
| | `scopeSortBySumVisits` | ترتيب حسب مجموع الزيارات. |
| | `scopeWithSumVisits` | إضافة العمود والترتيب معاً. |
| **مجموع المشاهدات** | `scopeAddSumViews` | إضافة عمود مجموع المشاهدات (حقل `views`). |
| | `scopeSortBySumViews` | ترتيب حسب مجموع المشاهدات. |
| | `scopeWithSumViews` | إضافة العمود والترتيب معاً. |
| **أحدث تاريخ زيارة** | `scopeAddLatestVisit` | إضافة عمود بأحدث تاريخ زيارة. |
| | `scopeSortByLatestVisit` | ترتيب حسب أحدث تاريخ زيارة. |
| | `scopeWithLatestVisit` | إضافة العمود والترتيب معاً. |
| **أقدم تاريخ زيارة** | `scopeAddEarliestVisit` | إضافة عمود بأقدم تاريخ زيارة. |
| | `scopeSortByEarliestVisit` | ترتيب حسب أقدم تاريخ زيارة. |
| | `scopeWithEarliestVisit` | إضافة العمود والترتيب معاً. |
| **التصفية حسب المستخدم** | `scopeVisitedByUser` | تصفية الكائنات التي زارها مستخدم معين. |
| | `scopeNotVisitedByUser` | تصفية الكائنات التي لم يزرها مستخدم معين. |
| **التصفية حسب وجود زيارات** | `scopeHasVisits` | تصفية الكائنات التي لديها زيارات (مع حد أدنى). |
| | `scopeHasNoVisits` | تصفية الكائنات التي ليس لديها زيارات. |
| **عمود الحالة للمستخدم** | `scopeWithIsVisitedByUser` | إضافة عمود `is_visited_by_user`. |
| **نطاقات خاصة** | `scopeTopVisited` | جلب الكائنات الأكثر زيارة (عدد السجلات). |
| | `scopeTopSumVisits` | جلب الكائنات الأكثر زيارة (مجموع `visits`). |

#### الدوال المساعدة الجديدة

| الدالة | الوصف |
|--------|-------|
| `getTotalVisits` | إجمالي عدد الزيارات (مجموع `visits`) مع خيارات التصفية. |
| `getTotalViews` | إجمالي عدد المشاهدات (مجموع `views`) مع خيارات التصفية. |
| `getVisitsCountByType` | توزيع الزيارات حسب نوع المستخدم (`user_type`). |
| `getVisitorsUsers` | قائمة المستخدمين الذين زاروا الكائن. |

جميع هذه الدوال تقبل خيارات التصفية: `$onlyActive`, `$withTrashed`, `$type`, `$processType`, `$eventType`.

---

### 2. تفاصيل التحديثات البرمجية

#### 2.1 إصلاح أخطاء الاستعلامات الفرعية
كانت النطاقات القديمة في السلوك الأصلي تعتمد على استخدام `GROUP BY` و `LIMIT` داخل الاستعلامات الفرعية للحصول على قيم مجمعة (مثل `COUNT`, `SUM`, `MAX`). هذا الأسلوب يؤدي إلى نتائج غير صحيحة وعشوائية، لأن `GROUP BY` ينشئ عدة مجموعات ثم `LIMIT 1` يختار أول مجموعة فقط.

**النهج الجديد**:
- استخدام دالة تجميعية مباشرة في الاستعلام الفرعي دون `GROUP BY` أو `LIMIT`. الدالة التجميعية في استعلام فرعي مرتبط تعمل تلقائياً على جميع الصفوف المطابقة لشرط الربط.
- كتابة اسم الجدول بالكامل أمام كل حقل لتجنب الغموض.
- إضافة شروط إضافية (مثل `type`, `processType`, `eventType`) عبر `where` داخل الاستعلام الفرعي.

مثال على الاستعلام الصحيح لترتيب حسب عدد الزيارات:

```php
$subQuery = Visit::selectRaw('COUNT(' . $table . '.id)')
    ->whereColumn($table . '.visitable_id', $this->model->getTable() . '.id')
    ->where($table . '.visitable_type', $this->model->getMorphClass())
    ->where(...);
```

#### 2.2 توحيد واجهة النطاقات
تم توحيد توقيع الدوال لتكون:
- `$orderDirection`: اتجاه الترتيب (افتراضي `DESC`).
- `$columnName`: اسم العمود المضاف (افتراضي مناسب مثل `visits_count`, `sum_visits`, `latest_visit_at`).
- `$onlyActive`: تصفية حسب `is_active`.
- `$withTrashed`: تضمين السجلات المحذوفة (Soft Deletes).
- `$type`, `$processType`, `$eventType`: خيارات إضافية للتصفية حسب نوع الزيارة ونوع العملية ونوع الحدث.
- في نطاقات التاريخ، تم إضافة باراميتر `$field` لتحديد حقل التاريخ المستخدم (`created_at`, `updated_at`, `last_visit_at`, `date_at`).

#### 2.3 دعم `withTrashed` (الزيارات المحذوفة)
تم إضافة باراميتر `$withTrashed` في جميع النطاقات والدوال الإحصائية. عند تمرير `true`، يتم تضمين سجلات الزيارة المحذوفة (Soft Deletes) في الاستعلام الفرعي. يتطلب ذلك أن يكون موديل `Visit` مفعّلاً للـ `SoftDeletes`.

#### 2.4 نقل النطاقات إلى ترايت منفصل
لتسهيل الصيانة وإعادة الاستخدام، تم نقل جميع النطاقات والدوال المساعدة إلى ترايت جديد:  
`Nano2\Visitors\Behaviors\VisitModel\VisitScopesAndHelpers`

ثم تم استخدام هذا التراث داخل السلوك `VisitModel` عبر `use`.

#### 2.5 التوافق العكسي
تم الاحتفاظ بالنطاقات القديمة كدوال تغليف في الكلاس الرئيسي، مع إضافة تعليق `@deprecated` لتوجيه المطورين إلى استخدام البدائل الجديدة. كما تم تعديل دوال `scopeWhereHasVisit` و `scopeWhereHasVisits` لتحافظ على السلوك الأصلي (عند عدم وجود مستخدم، لا تعيد أي نتائج) مع إضافة معامل `$isForceUser` للتحكم في هذا السلوك.

**قائمة الدوال القديمة المدعومة:**
- `scopeSortByCountVisitsOld` → `scopeSortByCountVisits`
- `scopeWithSortByCountVisits` → `scopeWithCountVisits`
- `scopeSortBySumVisitsOld` → `scopeSortBySumVisits`
- `scopeWithSortBySumVisits` → `scopeWithSumVisits`
- `scopeSortBySumViewsOld` → `scopeSortBySumViews`
- `scopeWithSortBySumViews` → `scopeWithSumViews`
- `scopeWhereHasVisit` → `scopeVisitedByUser` (مع تحسينات)
- `scopeWhereHasVisits` → `scopeVisitedByUser`

---

### 3. أمثلة تطبيقية

#### 3.1 الترتيب حسب عدد الزيارات
```php
// المنتجات الأكثر زيارة (عدد السجلات)
$products = Product::sortByCountVisits('DESC', 'visits_count')->take(10)->get();

// إضافة عمود عدد الزيارات
$products = Product::addCountVisits()->get();
foreach ($products as $product) {
    echo $product->visits_count;
}
```

#### 3.2 الترتيب حسب مجموع الزيارات (حقل visits)
```php
$products = Product::sortBySumVisits('DESC', 'sum_visits')->get();
```

#### 3.3 إضافة عمود `is_visited_by_user` للمستخدم الحالي
```php
$products = Product::withIsVisitedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->is_visited_by_user ? 'زُرتَه' : 'لم تزره';
}
```

#### 3.4 تصفية المنتجات التي زارها المستخدم الحالي
```php
$visitedProducts = Product::visitedByUser()->get();
```

#### 3.5 التصفية حسب نوع الزيارة والعملية
```php
// المنتجات التي زارها المستخدم الحالي من نوع "add" (دخول) في حدث "contest"
$products = Product::visitedByUser(null, null, false, 'add', 'in', 'contest')->get();
```

#### 3.6 الإحصائيات
```php
$product = Product::find(1);
echo "إجمالي الزيارات: " . $product->getTotalVisits(true); // نشطة فقط
echo "توزيع الزوار حسب النوع: ";
print_r($product->getVisitsCountByType(true)->toArray());
$visitors = $product->getVisitorsUsers(true);
```

#### 3.7 الجمع بين النطاقات المتقدمة
```php
// المنتجات التي لها أكثر من 5 زيارات نشطة، مرتبة حسب أحدث زيارة
$products = Product::hasVisits(5, true)
    ->sortByLatestVisit('DESC', 'latest_visit_at')
    ->get();
```

---

### 4. القيمة المضافة

- **للمطورين**: مجموعة متكاملة من النطاقات الجاهزة توفر الوقت وتقلل الأخطاء. لم يعد المطور بحاجة لكتابة استعلامات فرعية معقدة أو القلق بشأن صحة النتائج. الواجهة الموحدة تسهل التعلم والاستخدام عبر مختلف السلوكيات.
- **للمستخدمين النهائيين**: إمكانية تقديم قوائم مرتبة بذكاء (الأكثر زيارة، الأحدث نشاطاً) مما يحسن تجربة المستخدم ويزيد من فعالية التطبيقات.
- **للنظام**: أداء أفضل من خلال استعلامات SQL مُحسّنة، والقضاء على الاستعلامات غير الفعالة التي كانت تستخدم `GROUP BY` و `LIMIT` بشكل خاطئ.
- **المرونة**: إمكانية التصفية حسب النوع (`type`) والعملية (`processType`) والحدث (`eventType`) تسمح ببناء أنظمة معقدة مثل تتبع الإحصائيات التفصيلية للزيارات والمشاهدات.
- **قابلية التوسع**: إضافة نطاقات جديدة لأي سلوك مستقبلي يتم بنفس النمط، مما يضمن اتساق الكود وسهولة صيانته.

---

### 5. الخاتمة

يمثل هذا التحديث نقلة نوعية في إدارة استعلامات الزيارات والمشاهدات داخل تطبيقات نانوسوفت. من خلال إعادة هيكلة النطاقات ونقلها إلى ترايت مستقل، مع إضافة وظائف متقدمة للترتيب والتصفية والإحصائيات، أصبح بإمكان المطورين بناء أنظمة تتبع زيارة متطورة بسهولة وأداء عالٍ. التوافق العكسي يضمن سلاسة الانتقال للمشاريع القائمة، بينما تفتح الإضافات الجديدة آفاقاً واسعة لتجارب مستخدم غنية.

---

**ملاحظة**: لمزيد من التفاصيل حول كل نطاق وطريقة استخدامه، يمكن الرجوع إلى ملف التوثيق الخاص بالسلوك `VisitModel` أو مراجعة الأمثلة المقدمة أعلاه.

See [docs/VisitModel/Docs-VisitModel-ar.md](./docs/VisitModel/Docs-VisitModel-ar.md)

See [docs/VisitModel/Docs-VisitModel-Advenced-Examples-ar.md](./docs/VisitModel/Docs-VisitModel-Advenced-Examples-ar.md)

## 2026-05-20 - 2026-05-21

### تحديثات إضافة `Nano.Location` وإضافة `Nano.LocationApi`

### الإصدار 1.1.0 – دعم الوسائط المتعددة (Media) في النماذج الجغرافية وواجهة API

**الإصدار:** 1.1.0 (لـ Nano.LocationApi) و 1.0.16 (لـ Nano.Location)

---

#### ملخص التحديثات

يأتي هذا الإصدار ليعزز إضافة `Nano.Location` بإضافة دعم كامل للوسائط المتعددة (صور، فيديوهات، ملفات صوتية، مستندات) إلى نماذج **الدول (Country)**، **المدن/الولايات (State)**، و **المناطق/المديريات (Directorate)**. يشمل التحديث كلاً من الواجهة الخلفية (Backend) حيث أضيفت الحقول والأعمدة في قوائم الإدارة والنماذج، وواجهة برمجة التطبيقات (API) حيث أُضيف Behavior مخصص يسمح بتضمين هذه الوسائط ديناميكياً في استجابات الـ API عبر Transformers الخاصة بالموقع الجغرافي (`CountryTransformer`، `StateTransformer`، `DirectorateTransformer`).

تم تصميم الحل ليكون **قابلاً للتوسع**، **متوافقاً مع معمارية `Nano.API`**، ومتبعاً لنفس النمط المستخدم في `ProductTransformer` و `DynamicAddIncludeKyc` لضمان الاتساق وسهولة الصيانة.

---

### الإصدار 1.1.0 – تفاصيل التحديث

#### أهداف الإصدار

- **إضافة علاقات الميديا** إلى نماذج الموقع الجغرافي (Country، State، Directorate) باستخدام `attachOne` و `attachMany`.
- **توسيع قوائم الإدارة** (Backend Lists) لعرض الصور الرئيسية ومعارض الصور بشكل مرئي.
- **إضافة حقول رفع الملفات** في نماذج الإدارة (Backend Forms) مع دعم الصور، الفيديو، الصوت، والملفات العامة.
- **إنشاء Behavior جديد** `DynamicAddMediaIncludes` داخل `Nano.LocationApi` لإضافة includes ديناميكية إلى Transformers API.
- **ربط Behavior المذكور** مع `CountryTransformer`، `StateTransformer`، `DirectorateTransformer` لتمكين طلب الوسائط عبر API بنفس آلية `include`.
- **دعم إرجاع بيانات الميتا** (معلومات الملف التفصيلية) اختيارياً عبر المعامل `is_mate_data_media` أو عبر الخيارات المحددة لكل include.
- **ضمان التوافق** مع الأنظمة الحالية وعدم كسر أي من نقاط النهاية (Endpoints) الحالية.

---

#### الميزات الجديدة

##### 1. دعم الوسائط المتعددة في نماذج الموقع الجغرافي (Models)

تم توسيع كل من النماذج التالية بإضافة العلاقات الآتية (ديناميكياً عبر `Plugin.php` دون تعديل النماذج الأصلية):

- `RainLab\Location\Models\Country`
- `RainLab\Location\Models\State`
- `Nano\Location\Models\State`
- `Nano\Location\Models\Directorate`

**العلاقات المضافة:**

```php
public $attachOne = [
    'image'      => \System\Models\File::class,
    'book_intro' => \System\Models\File::class,
];

public $attachMany = [
    'images' => \System\Models\File::class,
    'videos' => \System\Models\File::class,
    'audios' => \System\Models\File::class,
    'files'  => \System\Models\File::class,
];
```

###### الجدول التالي يوضح استخدام كل علاقة:

| العلاقة | النوع | الوصف | الاستخدام النموذجي |
| :--- | :--- | :--- | :--- |
| `image` | `attachOne` | صورة واحدة (شعار، علم) | الصورة الرئيسية للدولة أو المدينة |
| `images` | `attachMany` | معرض صور متعدد | صور متعددة للمعالم السياحية |
| `videos` | `attachMany` | فيديوهات متعددة | فيديوهات تعريفية |
| `audios` | `attachMany` | ملفات صوتية متعددة | تسجيلات صوتية للتراث |
| `files` | `attachMany` | ملفات عامة متعددة | مستندات PDF، ملفات إضافية |
| `book_intro` | `attachOne` | ملف تمهيدي واحد | كتيب تعريف أو ملف PDF واحد |

##### 2. تحسين الواجهة الخلفية (Backend)

###### أ. إضافة أعمدة في قوائم الإدارة (List Columns)

تمت إضافة الأعمدة التالية إلى قوائم `Country`، `State`، `Directorate` عبر حدث `backend.list.extendColumns`:

| العمود | النوع | مرئي افتراضياً | الوصف |
| :--- | :--- | :--- | :--- |
| `image` | `simpleimage` | نعم | عرض الصورة الرئيسية كمصغرة |
| `images` | `simpleimages` | لا | عرض أيقونة تشير إلى وجود معرض صور |
| `book_intro` | `partial` (رابط) | لا | رابط لتنزيل الملف التمهيدي |
| `files` | `partial` (رابط) | لا | رابط لتنزيل الملفات العامة |
| `videos` | `partial` (رابط) | لا | رابط لتنزيل الفيديوهات |
| `audios` | `partial` (رابط) | لا | رابط لتنزيل التسجيلات الصوتية |

**مثال لعمود `image` في `columns.yaml` (ديناميكياً):**

```php
'image' => [
    'label' => Lang::get('nano.location::lang.public.models.image'),
    'type' => 'simpleimage',
    'searchable' => false,
    'sortable' => false,
    'invisible' => false,
    'cssClass' => 'nolink',
],
```

###### ب. إضافة حقول في نماذج الإدارة (Form Fields)

تمت إضافة تبويب جديد باسم `Media` (أو `الوسائط` حسب اللغة) يحتوي على الحقول التالية عبر حدث `backend.form.extendFields`:

| الحقل | النوع | الخيارات | التبويب |
| :--- | :--- | :--- | :--- |
| `image` | `fileupload` | mode: image, imageWidth: 120, imageHeight: 120, useCaption: true | Media |
| `images` | `fileupload` | mode: image, multiple: true | Media |
| `videos` | `fileupload` | mode: file, fileTypes: mp4,avi,mov,mkv,webm, maxFiles: 5, maxFilesize: 15 | Media |
| `audios` | `fileupload` | mode: file, fileTypes: mp3,wav,wma,m4a,ogg, maxFiles: 5, maxFilesize: 15 | Media |
| `book_intro` | `fileupload` | mode: file, useCaption: true | Media |
| `files` | `fileupload` | mode: file, multiple: true | Media |

**مثال لكود إضافة الحقول في `Plugin.php`:**

```php
$widget->addTabFields([
    'image' => [
        'label' => Lang::get('nano.location::lang.public.models.image'),
        'type' => 'fileupload',
        'mode' => 'image',
        'imageWidth' => 120,
        'imageHeight' => 120,
        'useCaption' => true,
        'span' => 'auto',
        'tab' => $tab,
    ],
    // ... باقي الحقول
]);
```

##### 3. Behavior جديد: `DynamicAddMediaIncludes` (في إضافة `Nano.LocationApi`)

تم إنشاء Behavior ضمن الـ namespace `Nano\LocationApi\Behaviors\DynamicAddMediaIncludes` لتوفير الدوال الخاصة بـ includes الخاصة بالوسائط بشكل موحد وقابل لإعادة الاستخدام. هذا الـ Behavior يتبع نفس نمط `DynamicAddIncludeKyc` المستخدم في إضافة `Nano3.Kyc`.

###### أ. هيكل الـ Behavior

```php
<?php namespace Nano\LocationApi\Behaviors;

use October\Rain\Extension\ExtensionBase;
use Nano\API\Classes\Transformer;
use League\Fractal\Resource\Primitive;
use League\Fractal\Resource\Collection;

class DynamicAddMediaIncludes extends ExtensionBase
{
    protected Transformer $transformer;
    
    public function __construct(Transformer $transformer)
    {
        $this->transformer = $transformer;
        
        // إضافة includes إلى availableIncludes
        $mediaIncludes = ['image', 'images', 'videos', 'audios', 'files', 'book_intro'];
        foreach ($mediaIncludes as $include) {
            if (!in_array($include, $this->transformer->availableIncludes, true)) {
                $this->transformer->availableIncludes[] = $include;
            }
        }
    }
    
    public function includeImage($model) { /* ... */ }
    public function includeImages($model) { /* ... */ }
    public function includeVideos($model) { /* ... */ }
    public function includeAudios($model) { /* ... */ }
    public function includeFiles($model) { /* ... */ }
    public function includeBookIntro($model) { /* ... */ }
}
```

###### ب. الميزات الرئيسية للـ Behavior:

- **إضافة تلقائية للـ includes**: يضيف الـ Behavior تلقائياً `image`, `images`, `videos`, `audios`, `files`, `book_intro` إلى `availableIncludes` في الـ Transformer.
- **دعم التحكم في بيانات الميتا**: يمكن التحكم في إرجاع بيانات الميتا لكل include على حدة عبر request parameters:
  - عام: `?is_mate_data_media=1`
  - خاص: `?image[mate_data]=1&files[mate_data]=1`
- **التكامل مع دوال الـ Transformer الأساسية**: يستخدم الـ Behavior دوال `image()`، `images()`، `file()`، `files()` الموجودة في `Nano\API\Classes\Transformer` لضمان تنسيق موحد.
- **معالجة آمنة للاستثناءات**: في حالة حدوث خطأ (مثل عدم وجود العلاقة في النموذج)، يتم إرجاع قيم افتراضية (`null` للـ attachOne، `[]` للـ attachMany) بدلاً من إلقاء استثناء، مما يضمن عدم تعطل استجابة الـ API.

###### ج. دوال الـ Behavior بالتفصيل:

| الدالة | نوع المورد | القيمة الافتراضية عند الخطأ | الوصف |
| :--- | :--- | :--- | :--- |
| `includeImage` | `Primitive` | `null` | إرجاع بيانات الصورة الرئيسية |
| `includeImages` | `Collection` | `[]` | إرجاع قائمة ببيانات معرض الصور |
| `includeVideos` | `Collection` | `[]` | إرجاع قائمة ببيانات الفيديوهات |
| `includeAudios` | `Collection` | `[]` | إرجاع قائمة ببيانات التسجيلات الصوتية |
| `includeFiles` | `Collection` | `[]` | إرجاع قائمة ببيانات الملفات العامة |
| `includeBookIntro` | `Primitive` | `null` | إرجاع بيانات الملف التمهيدي |

##### 4. توسيع Transformers الخاصة بالموقع الجغرافي

تم توسيع الـ Transformers التالية لتشمل الـ Behavior الجديد:

- `Nano\LocationApi\Transformers\CountryTransformer`
- `Nano\LocationApi\Transformers\StateTransformer`
- `Nano\LocationApi\Transformers\DirectorateTransformer`

تم ذلك عبر دالة `extendTransformerDynamicAddMediaIncludes` داخل `Plugin.php` الخاصة بـ `Nano.LocationApi`:

```php
protected function extendTransformerDynamicAddMediaIncludes($className): void
{
    try {
        $className::extend(function ($transformer) {
            if (!$transformer->isClassExtendedWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes')) {
                $transformer->extendClassWith('Nano\LocationApi\Behaviors\DynamicAddMediaIncludes');
            }
        });
    } catch (\Exception | \Throwable $e) {
        Log::error("Error extending transformer: " . $e->getMessage());
    }
}
```

##### 5. دعم الترجمة (Multi-language)

تمت إضافة مفاتيح الترجمة الجديدة في ملفات اللغة العربية والإنجليزية داخل إضافة `Nano.Location`:

**الملف:** `plugins/nano/location/lang/ar/lang.php`

```php
'models' => [
    'media_tab' => 'الوسائط',
    'image' => 'الصورة الرئيسية',
    'images' => 'معرض الصور',
    'book_intro' => 'ملف تمهيدي',
    'files' => 'الملفات المرفقة',
    'videos' => 'الفيديوهات',
    'audios' => 'التسجيلات الصوتية',
    'image_comment' => 'اختر صورة واحدة',
    'images_comment' => 'اختر عدة صور',
    'book_intro_comment' => 'رفع ملف PDF أو مستند',
    'files_comment' => 'رفع ملفات متنوعة',
    'videos_comment' => 'رفع فيديوهات (mp4, avi, mov, mpg, mpeg, mkv, webm)',
    'audios_comment' => 'رفع ملفات صوتية (mp3, wav, wma, m4a, ogg)',
],
```

**الملف:** `plugins/nano/location/lang/en/lang.php`

```php
'models' => [
    'media_tab' => 'Media',
    'image' => 'Main Image',
    'images' => 'Gallery Images',
    'book_intro' => 'Introductory File',
    'files' => 'Attached Files',
    'videos' => 'Videos',
    'audios' => 'Audio Recordings',
    'image_comment' => 'Choose a single image',
    'images_comment' => 'Choose multiple images',
    'book_intro_comment' => 'Upload a PDF or document',
    'files_comment' => 'Upload various files',
    'videos_comment' => 'Upload videos (mp4, avi, mov, mpg, mpeg, mkv, webm)',
    'audios_comment' => 'Upload audio files (mp3, wav, wma, m4a, ogg)',
],
```

---

#### أمثلة عملية

##### 1. استخدام API لاسترجاع دولة مع صورها وفيديوهات

**الطلب:**
```http
GET /api/v1/countries/1?include=image,images,videos&is_mate_data_media=1
```

**الاستجابة (JSON):**
```json
{
  "data": {
    "id": 1,
    "name": "Yemen",
    "code": "YE",
    "image": {
      "original": "https://domain.com/storage/app/media/flags/ye.png",
      "small": "https://domain.com/storage/app/media/flags/ye_small.png",
      "thumb": "https://domain.com/storage/app/media/flags/ye_thumb.png",
      "mate_data": {
        "id": 101,
        "title": "Flag of Yemen",
        "description": "Official flag",
        "file_name": "ye.png",
        "file_size": 45678,
        "extension": "png",
        "mime_type": "image/png"
      }
    },
    "images": [
      {
        "original": "https://domain.com/storage/app/media/gallery/sanaa.jpg",
        "small": "https://domain.com/storage/app/media/gallery/sanaa_small.jpg",
        "mate_data": { ... }
      }
    ],
    "videos": [
      {
        "file_name": "yemen_intro.mp4",
        "path": "https://domain.com/storage/app/media/videos/yemen_intro.mp4",
        "mate_data": { ... }
      }
    ]
  }
}
```

##### 2. طلب بيانات مدينة مع تضمين الملف التمهيدي فقط

**الطلب:**
```http
GET /api/v1/states/5?include=book_intro
```

**الاستجابة (مقتطف):**
```json
{
  "data": {
    "id": 5,
    "name": "Sana'a",
    "country_id": 1,
    "book_intro": {
      "file_name": "sanaa_guide.pdf",
      "path": "https://domain.com/storage/app/media/guides/sanaa_guide.pdf",
      "file_size": 2048000
    }
  }
}
```

##### 3. طلب بيانات مديرية مع تضمين الصور والملفات مع بيانات الميتا المخصصة لكل include

**الطلب:**
```http
GET /api/v1/directorates/10?include=images,files&images[mate_data]=1&files[mate_data]=0
```

في هذا المثال، سيتم إرجاع بيانات الميتا للصور فقط وليس للملفات.

##### 4. الواجهة الخلفية: رفع صورة رئيسية لدولة

1. انتقل إلى `الموقع الجغرافي > الدول`.
2. اختر دولة من القائمة أو أنشئ دولة جديدة.
3. في تبويب `الوسائط` (Media)، ارفع صورة في حقل `الصورة الرئيسية`.
4. احفظ التغييرات.
5. ستظهر الصورة المصغرة في قائمة الدول.

##### 5. استخدام الـ Behavior في Transformer مخصص

إذا كان لديك Transformer مخصص ترغب في إضافة دعم الوسائط له، يمكنك ببساطة إضافة الـ Behavior:

```php
use Nano\LocationApi\Behaviors\DynamicAddMediaIncludes;

class MyCustomTransformer extends Transformer
{
    public $implement = [
        DynamicAddMediaIncludes::class,
    ];
    
    // ... باقي الكود
}
```

---

#### التحسينات والإصلاحات

| المجال | التحسين |
| :--- | :--- |
| **معالجة الاستثناءات** | في حالة عدم وجود علاقة ميديا في النموذج (مثل `image` غير معرفة)، يتم إرجاع `null` أو `[]` بدلاً من إلقاء خطأ. |
| **دعم جزئي للميتا** | يمكن طلب بيانات الميتا لكل include على حدة (`image[mate_data]=1`) أو بشكل عام (`is_mate_data_media=1`). |
| **توافق عكسي كامل** | جميع الـ Transformers الحالية لم تتأثر، ولا يتم تضمين الوسائط إلا إذا طلبها العميل صراحة (`include=image,...`). |
| **تحسين أداء التخزين المؤقت** | يتم التعامل مع الوسائط كأي علاقة أخرى في نظام Fractal، مما يسمح بتخزينها مؤقتاً بشكل ذكي. |
| **توسع سهل** | يمكن إضافة علاقات وسائط جديدة مستقبلاً ببساطة عن طريق إضافة الدوال المناسبة في الـ Behavior وإضافتها إلى `availableIncludes`. |
| **توافق مع `System\Models\File`** | يتم استخدام نظام الملفات القياسي في October CMS، مما يضمن توافق الملفات مع جميع ميزات `System\Models\File` (مثل التصغير، التحكم في الوصول، إلخ). |

---

#### متطلبات الترقية

##### 1. تحديث كود الإضافة `Nano.Location`

- **استبدال** ملف `plugins/nano/location/Plugin.php` بالنسخة الجديدة (التي تحتوي على دوال `extendCountryWithMedia`، `extendRainLabStateWithMedia`، إلخ، واستدعائها في `boot()`).
- **إضافة** مفاتيح الترجمة الجديدة إلى `plugins/nano/location/lang/ar/lang.php` و `plugins/nano/location/lang/en/lang.php`.
- **إنشاء** ملفات الـ partial لعرض الروابط في الأعمدة (اختياري، حيث تم إخفاء الأعمدة غير `image` افتراضياً). إذا أردت استخدامها، قم بإنشاء المجلد `plugins/nano/location/models/country/` وأضف الملفات التالية:
  - `_link_book_intro.htm`
  - `_link_files.htm`
  - `_link_videos.htm`
  - `_link_audios.htm`

**مثال لمحتوى `_link_book_intro.htm`:**
```twig
{% if model.book_intro %}
    <a href="{{ model.book_intro.path }}" target="_blank">تحميل</a>
{% endif %}
```

##### 2. تحديث كود الإضافة `Nano.LocationApi`

- **استبدال** ملف `plugins/nano/locationapi/Plugin.php` بالنسخة الجديدة (التي تحتوي على دالة `extendTransformerDynamicAddMediaIncludes` واستدعائها في `boot()`).
- **إضافة** مجلد `behaviors` في `plugins/nano/locationapi/` ووضع ملف `DynamicAddMediaIncludes.php` فيه.
- **تحديث** ملف `version.yaml` الخاص بـ `Nano.LocationApi` لإضافة الإصدار `1.1.0` كما هو موضح في قسم التحديثات.

##### 3. تحديث ملفات `version.yaml`

**لـ `Nano.Location` (`plugins/nano/location/updates/version.yaml`):**
```yaml
1.0.16:
    - 'Add media relations (image, images, videos, audios, files, book_intro) to Country, State, and Directorate models'
    - 'Add media fields and list columns to backend forms and lists for Country, State, Directorate'
    - 'Add translations for media fields (ar/en)'
```

**لـ `Nano.LocationApi` (`plugins/nano/locationapi/updates/version.yaml`):**
```yaml
1.1.0:
    - 'Add DynamicAddMediaIncludes behavior for CountryTransformer, StateTransformer, DirectorateTransformer'
    - 'Support dynamic media includes (image, images, videos, audios, files, book_intro) in Location API'
    - 'Add is_mate_data_media parameter to control returning file metadata in API responses'
    - 'Extend availableIncludes automatically for media relations'
```

##### 4. تشغيل أوامر التحديث

```bash
# مسح الكاش
php artisan cache:clear
php artisan config:clear
```

##### 5. التحقق من صلاحيات رفع الملفات

تأكد من أن مجلد `storage/app/uploads` قابل للكتابة بواسطة خادم الويب:

```bash
chmod -R 775 storage/app/uploads
```

##### 6. اختبار الوظائف

- **اختبار الواجهة الخلفية:** انتقل إلى `الموقع الجغرافي > الدول`، جرب رفع صورة لدولة وتأكد من ظهورها في القائمة.
- **اختبار API:** استخدم `curl` أو Postman لاختبار نقاط النهاية التالية:

```bash
# جلب دولة مع الصورة الرئيسية
curl -X GET "https://yourdomain.com/api/v1/countries/1?include=image"

# جلب مدينة مع معرض الصور والملفات
curl -X GET "https://yourdomain.com/api/v1/states/5?include=images,files&is_mate_data_media=1"
```

---

#### التوافق مع الإصدارات السابقة

- **لا توجد تغييرات جوهرية** في هيكل قاعدة البيانات، فقط تمت إضافة علاقات `attachOne` و `attachMany` ديناميكياً.
- **لا يتم تضمين الوسائط تلقائياً** في استجابات API الحالية، لذلك لن تتأثر التطبيقات الحالية التي تستخدم API.
- **جميع دوال الـ API الحالية** تعمل كما هي دون تعديل.
- **الواجهة الخلفية الحالية** لم تتأثر، وتمت إضافة تبويب جديد فقط دون إزالة أي حقول موجودة.

---

#### الخاتمة

يمثل الإصدار 1.1.0 من إضافة `Nano.Location` و `Nano.LocationApi` نقلة نوعية في دعم الوسائط المتعددة للنماذج الجغرافية. بفضل البنية المعيارية المستندة إلى Behaviors، أصبح من السهل الآن إضافة وإدارة الصور والفيديوهات والملفات للدول والمدن والمديريات من خلال واجهة إدارة بديهية، واسترجاعها عبر API بنفس طريقة استرجاع أي علاقة أخرى.

---

**الوثائق المرجعية:**
- [توثيق إضافة Nano.Location](./docs/Location/Docs-Nano-Location-ar.md)
- [توثيق Behavior DynamicAddMediaIncludes](./docs/Location/Docs-DynamicAddMediaIncludes-ar.md)
- [توثيق واجهة برمجة التطبيقات (API) للموقع الجغرافي](./docs/Location/Docs-LocationAPI-ar.md)


## 2026-05-20 – 2026-05-22

**تحديث بوابة الدفع CashPay (الدفع النقدي الإلكتروني – OTP) في نظام المدفوعات بنانوسوفت**

### إضافة بوابة دفع CashPay (الدفع النقدي الإلكتروني – OTP)

ضمن الوحدة البرمجية `Nano.Yepayment`

---

## 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **CashPay** وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع الإلكترونية التي تعتمد على تأكيد الدفع عبر رمز OTP (One‑Time Password)، حيث تُعد **CashPay** إحدى منصات الدفع التي تتيح قبول المدفوعات عبر واجهة API مباشرة مع تشفير AES‑256‑CBC لكلمة المرور وآلية تبادل آمنة.

تعمل CashPay وفق نمط **الدفع بخطوتين (Two‑Step Payment) باستخدام OTP**، وهو النمط الذي يتوافق مع طبيعة عمل المنصة حيث:
1. **الخطوة الأولى (process):** يتم إنشاء معاملة (InitPayment) والحصول على `TransactionRef`، دون خصم المبلغ بعد. في هذه المرحلة يُطلب من العميل إدخال رمز OTP الذي سيصله لاحقاً.
2. **الخطوة الثانية (complete):** بعد أن يدخل العميل رمز OTP، يتم إرسال طلب تأكيد الدفع (ConfirmPayment) إلى CashPay. إذا كان الرمز صحيحاً، يتم خصم المبلغ وتحديث الطلب إلى "مدفوع".

هذا التصميم جعل البوابة متوافقة تماماً مع `OrderManager` و `Checkout2` و `PaymentResult`، ويمنع تغيير حالة الطلب إلى مدفوع قبل إدخال OTP الصحيح.

تم تطوير البوابة وفق أعلى معايير الأمان والجودة، مع استخدام `HttpHelper` الموحد لجميع طلبات API، وتشفير كلمة المرور في الهيدر باستخدام AES‑256‑CBC، وتوفير واجهات اختبار متكاملة ونقاط نهاية API مخصصة للمطورين والمسؤولين. كما تم دعم `RedirectHelper` لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى روابط عودة بعد الدفع.

---

## 2. المكونات المطورة

### 2.1. `CashPay` – كلاس بوابة الدفع الرئيسي

- **المسار:** `Nano\Yepayment\PaymentTypes\CashPay`
- **الوراثة:** يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة:** مسؤول عن المصادقة عبر الهيدر المشفر (`encPassword`)، تنفيذ الخطوة الأولى (InitPayment)، والخطوة الثانية (ConfirmPayment)، والاستعلام عن حالة المعاملة (OperationStatus)، وتغيير كلمة المرور (ChangePass).

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | **الخطوة الأولى:** التحقق من وجود معاملة سابقة، إنشاء معاملة جديدة عبر `InitPayment`، استلام `TransactionRef`، حفظه في الطلب دون تغيير حالة الدفع. تُعيد نجاحاً مع رسالة "أدخل OTP". |
| `complete(PaymentResult $result)` | **الخطوة الثانية:** التحقق من `TransactionRef` و `OTP`، إرسال طلب `ConfirmPayment`، وعند النجاح استدعاء `$result->success()` لتحديث الطلب إلى `PaidState`. |
| `operationStatus(string $requestId, string $type): array` | الاستعلام عن حالة معاملة باستخدام `RequestID` (نوع `InitOP` للمعاملة الأولية، `PayOP` للتأكيد). |
| `changePassword(string $newPassword): array` | تغيير كلمة المرور الخاصة بالتاجر لدى CashPay (عند انتهاء صلاحيتها أو الحاجة إلى تحديثها). |
| `getAuthToken(): ?string` | (غير مستخدم في CashPay لأن المصادقة تعتمد على `encPassword` في الهيدر. لكن الدالة موجودة للتوافق). |
| `initPayment(): array` | بناء طلب `InitPayment` وإرساله إلى `/api/CashPay/InitPayment`، مع إرجاع `TransactionRef`. |
| `confirmPayment(string $transactionRef, string $otp): array` | بناء طلب `ConfirmPayment` مع `TRCode = md5(TransactionRef + OTP)` وإرساله إلى `/api/CashPay/ConfirmPayment`. |
| `encryptAes256Cbc(string $data, string $key): string` | تشفير نص (كلمة المرور) باستخدام AES‑256‑CBC للحصول على `encPassword`. |
| `getEncryptedPassword(): string` | تشفير كلمة المرور المخزنة باستخدام مفتاح التشفير من الإعدادات. |
| `buildHeaders(): array` | بناء هيدرات الطلبات (`encPassword`, `unixtimestamp`, `Content-Type`). |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL, Username, Password, Encryption Key, SpId, العملة الافتراضية). |
| `encryptedSettings(): array` | تحديد الحقول التي تخزن بشكل مشفر (`cashpay_password`, `cashpay_encryption_key`). |
| `getSupportedCurrencies(): array` | قائمة العملات المدعومة مع أرقامها (YER=2, USD=1, SAR=3). |
| `getCurrencyId($currency): ?int` | تحويل رمز العملة أو رقمها إلى الرقم الصحيح المدعوم. |
| `isCurrencySupported($currency): bool` | التحقق من دعم عملة معينة. |

### 2.2. ملفات جزئية (Partials) وإعدادات

| الملف | المسار | الوصف |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/cashpay/_info.htm` | عرض إرشادات الإعداد ومعلومات البوابة في لوحة التحكم. |
| `_test_info.htm` | `paymenttypes/cashpay/_test_info.htm` | أدوات اختبار سريع داخل صفحة الإعدادات (أزرار المصادقة، إنشاء معاملة، تأكيد، استعلام، تغيير كلمة المرور، اختبار شامل). |
| `cashpay-ui.htm` | `views/cashpay-ui.htm` | واجهة ويب تفاعلية متكاملة لاختبار جميع وظائف CashPay (يدوي، تلقائي، إحصائيات، سجلات). |

### 2.3. نقاط نهاية API (`routes.php`)

تم إضافة مجموعة كاملة من المسارات ضمن مجموعة `yepayment` لدعم الاختبار والمراقبة:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/cashpay/test-auth` | POST | اختبار صحة الإعدادات (Username, Password, Encryption Key, SpId). |
| `/cashpay/test-create-payment` | POST | إنشاء معاملة جديدة (InitPayment). |
| `/cashpay/test-confirm-payment` | POST | تأكيد الدفع (ConfirmPayment) باستخدام OTP. |
| `/cashpay/test-check-status` | POST | الاستعلام عن حالة معاملة (OperationStatus). |
| `/cashpay/test-change-password` | POST | تغيير كلمة المرور (ChangePass). |
| `/cashpay/test-full-payment` | POST | اختبار شامل (InitPayment + ConfirmPayment). |
| `/cashpay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح). |
| `/cashpay/test-ui` | GET | واجهة الاختبار المتكاملة (HTML). |
| `/cashpay/success` | GET | نقطة نهاية لإعادة التوجيه عند نجاح الدفع (تستخدم `RedirectHelper`). |
| `/cashpay/cancel` | GET | نقطة نهاية لإعادة التوجيه عند الإلغاء. |

جميع المسارات (باستثناء success/cancel) محمية بـ `BackendAuth::getUser()` لضمان وصول المسؤولين فقط.

### 2.4. `Plugin.php` – تسجيل مزود الدفع

تمت إضافة السطر التالي في دالة `registerPaymentProviders()` ضمن قسم `allow_yemen_payment`:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\CashPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\CashPay();
}
```

### 2.5. ملفات اللغة (Translation)

تم إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php` لكل من الإعدادات ورسائل العموم والأخطاء (مثل `payment_success`, `auth_success`, `confirmation_required`, `order_already_paid`).

---

## 3. دورة عمل الدفع (Payment Workflow) – نمط الخطوتين (OTP)

تعمل CashPay وفق تدفق **Two‑Step (خطوتين) مع OTP**:

1. **المصادقة (Authentication)**  
   لا توجد جلسة توكن منفصلة. يتم إرسال كل طلب مع هيدر `encPassword` وهو كلمة مرور التاجر مشفرة بـ AES‑256‑CBC باستخدام مفتاح التشفير المقدم.  
   تُستخدم أيضاً `unixtimestamp` (بالمللي ثانية) لمنع إعادة الهجمات.

2. **الخطوة الأولى – إنشاء المعاملة (`process`)**  
   - التحقق من أن الطلب غير مدفوع مسبقاً.
   - التحقق من وجود معاملة سابقة مرتبطة بالطلب عبر `payment_first_trans_id` أو `other_data['cashpay']['transaction_ref']`:
     - إذا وُجدت، يتم استدعاء `operationStatus()` للاستعلام عن حالتها.
     - إذا كانت الحالة `success` → يتم استدعاء `$result->success()` (الطلب مدفوع مسبقاً).
     - إذا كانت الحالة `pending` → يتم إرجاع رسالة "أدخل OTP" دون إنشاء معاملة جديدة.
   - **لا توجد معاملة سابقة أو الحالة غير صالحة:**  
     - بناء جسم الطلب (`RequestID`, `UserName`, `SpId`, `MDToken`, `TargetMSISDN`, `CustomerCashPayCode`, `Amount`, `CurrencyId`, `Desc`).
     - إرسال طلب `POST /api/CashPay/InitPayment`.
     - استلام `TransactionRef` وتخزينه في `order.payment_first_trans_id` و `order.payment_trans_id`.
     - تخزين البيانات الإضافية في `order.other_data['cashpay']` (المبلغ، العملة، رقم الجوال، رمز الدفع، تاريخ الإنشاء، روابط العودة).
     - **لا يتم استدعاء `$result->success()`**، ولا يتغير الطلب إلى `PaidState`. بدلاً من ذلك، تُرجع البوابة `successful = true` مع رسالة `confirmation_required`.
     - تسجيل `PaymentLog` عبر `$result->logSuccessfulPayment()`.

3. **الخطوة الثانية – تأكيد الدفع (`complete`)**  
   - تُستدعى بعد أن يدخل العميل OTP (يُمرر في `$this->data['otp']`).
   - تُستخرج `TransactionRef` من الطلب.
   - تُحسب `TRCode = md5(TransactionRef . OTP)`.
   - يُرسل طلب `POST /api/CashPay/ConfirmPayment` مع `TransactionRef` و `TRCode`.
   - إذا كان `ResultCode == 1`:
     - يتم استدعاء `$result->success()` التي **تغير حالة الطلب إلى `PaidState`** وتُشغل الأحداث (مثل `nano.orders.paymentProcessed`) وتُفرغ السلة.
   - إذا كان `ResultCode == 6022`: يتم إرجاع خطأ "يجب تغيير كلمة المرور".
   - أي رمز آخر يُعتبر فشلاً.

4. **روابط العودة الاختيارية**  
   عند تمرير `callback_success_url` أو `callback_error_url` في بيانات الدفع، يتم تخزينها في `other_data`. بعد التأكيد، يمكن للمطور استخدام `RedirectHelper` لإعادة التوجيه بذكاء (Deep Link أو ويب).

---

## 4. الإعدادات والتهيئة (Configuration)

لتفعيل CashPay، يجب إدخال الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| الإعداد | المفتاح | الوصف | القيمة الافتراضية |
|---------|---------|-------|-------------------|
| رابط API الأساسي | `cashpay_url` | عنوان API لبيئة الإنتاج | `https://api.cash-pay.com` |
| رابط API للاختبار | `cashpay_test_url` | عنوان API لبيئة UAT (اختياري) | `https://test-api.cash-pay.com` |
| اسم المستخدم | `cashpay_username` | اسم مستخدم التاجر | - |
| كلمة المرور | `cashpay_password` | كلمة مرور التاجر (تُخزن مشفرة) | - |
| مفتاح التشفير (AES‑256‑CBC) | `cashpay_encryption_key` | المفتاح المستخدم لتشفير كلمة المرور في الهيدر | - |
| رمز العميل (SpId) | `cashpay_sp_id` | رمز مزود الخدمة المقدم من CashPay | - |
| العملة الافتراضية | `cashpay_default_currency_id` | العملة الافتراضية (تظهر كخيارات منسدلة) | `YER` |

**ملاحظات أمنية:**
- `cashpay_password` و `cashpay_encryption_key` يتم تخزينهما مشفرين عبر `encryptedSettings()`.
- لا يتم تسجيل هذه القيم الحساسة في السجلات.

---

## 5. الأمثلة التوضيحية (Usage Examples)

### 5.1. بدء الدفع – الخطوة الأولى (InitPayment)

```php
use Nano\Yepayment\PaymentTypes\CashPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$cash = new CashPay($order, [
    'target_msisdn' => '771234567',
    'customer_cash_pay_code' => 555,
    'amount' => 100,
    'currency' => 'YER',
    'desc' => 'دفع الطلب #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($cash, $order);
$processResult = $cash->process($result);

if ($processResult->successful) {
    // تم إنشاء المعاملة وحفظ TransactionRef، ينتظر النظام إدخال OTP
    $transactionRef = $order->payment_first_trans_id;
    // إظهار حقل إدخال OTP للمستخدم
}
```

### 5.2. تأكيد الدفع – الخطوة الثانية (ConfirmPayment)

بعد أن يدخل المستخدم OTP، يمكن استدعاء `complete()` مباشرة:

```php
$cash = new CashPay($order, ['otp' => '1234']);
$result = new PaymentResult($cash, $order);
$completeResult = $cash->complete($result);

if ($completeResult->successful) {
    // تم تأكيد الدفع وتحديث الطلب إلى PaidState
}
```

أو باستخدام دالة مساعدة ثابتة (عبر مسار `success`):

```php
$result = \Nano\Yepayment\PaymentTypes\CashPay::checkAndCompletePay(['order_id' => 200, 'otp' => '1234']);
if ($result['success']) {
    // تم الدفع بنجاح
}
```

### 5.3. الاستعلام المباشر عن حالة معاملة

```php
$cash = new CashPay();
$status = $cash->operationStatus('1234567890', 'InitOP');
if ($status['success']) {
    echo "الحالة: " . $status['status']; // success, pending, failed
}
```

### 5.4. تغيير كلمة المرور (ChangePass)

عند استلام رمز خطأ 6022، يجب تغيير كلمة المرور:

```php
$cash = new CashPay();
$result = $cash->changePassword('NewStrongPass123');
if ($result['success']) {
    // تم تغيير كلمة المرور بنجاح، يجب تحديث الإعدادات في لوحة التحكم أيضاً
}
```

### 5.5. استخدام واجهة الاختبار المتكاملة

بعد تسجيل الدخول إلى لوحة التحكم كمسؤول، افتح الرابط:
```
https://yourdomain.com/api/v1/yepayment/cashpay/test-ui
```

الواجهة تتيح:
- اختبار المصادقة (التحقق من صحة الإعدادات).
- إنشاء معاملة (InitPayment) وعرض `TransactionRef`.
- تأكيد الدفع (ConfirmPayment) باستخدام OTP.
- الاستعلام عن الحالة (OperationStatus) باستخدام `RequestID`.
- تغيير كلمة المرور (ChangePass).
- اختبار شامل (إنشاء + تأكيد) في خطوة واحدة.
- اختبار تلقائي بعدد مرات محدد (حتى 5 مرات).
- إحصائيات الاستخدام (عدد الطلبات، نسبة النجاح).
- سجلات محلية للاختبارات.

---

## 6. التعامل مع رموز الأخطاء وأكواد الاستجابة

| كود | المعنى | الإجراء المناسب |
|-----|--------|------------------|
| 1 | نجاح العملية | متابعة التدفق الطبيعي. |
| 35 | بيانات الإدخال غير صالحة | التحقق من الحقول المرسلة. |
| 6022 | كلمة المرور منتهية الصلاحية أو غير صالحة | استدعاء `changePassword()` لتغيير كلمة المرور. |
| 6025 | رصيد العميل غير كافٍ | إبلاغ العميل بشحن الرصيد. |
| 9999 | خطأ داخلي عام | إعادة المحاولة أو الاتصال بالدعم. |

> **ملاحظة:** أي رمز خطأ آخر يتم عرضه كما هو من CashPay عبر `ResultMessage`.

---

## 7. اختبار التكامل (Integration Testing)

تم توفير عدة طبقات للاختبار:

- **بيئة الاختبار:** يمكن استخدام رابط UAT المقدم من CashPay (يُحدد في حقل `cashpay_test_url`).
- **واجهة الاختبار السريع:** من خلال صفحة إعدادات البوابة (الملف الجزئي `_test_info.htm`) يمكن اختبار المصادقة، إنشاء معاملة، تأكيد، استعلام، وتغيير كلمة المرور.
- **واجهة الاختبار المتكاملة:** `/api/v1/yepayment/cashpay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة بشكل كامل (يدوي، تلقائي، إحصائيات).
- **نقاط نهاية API مستقلة:** يمكن للمطورين استخدام أدوات مثل Postman أو cURL للاتصال المباشر بالنقاط مثل `/cashpay/test-create-payment`.

**خطوات اختبار سريعة:**
1. تأكد من إعداد Username, Password, Encryption Key, SpId في صفحة إعدادات البوابة.
2. افتح `/api/v1/yepayment/cashpay/test-ui`.
3. انقر على "اختبار المصادقة" للتحقق من صحة البيانات.
4. أدخل رقم طلب، رقم جوال، رمز الدفع، المبلغ، ثم انقر "إنشاء معاملة".
5. سيظهر `TransactionRef` – أدخل OTP الصحيح (يُرسل إلى الجوال) في حقل OTP وانقر "تأكيد الدفع".
6. ستتغير حالة الطلب إلى مدفوع.

---

## 8. ملاحظات للمطورين (Developer Notes)

- **نمط الخطوتين:** `process()` **لا** يُكمل الدفع، بل ينشئ المعاملة فقط. يجب استدعاء `complete()` مع OTP صحيح لتأكيد الدفع وتحديث الطلب إلى `PaidState`.
- **تشفير كلمة المرور:** يتم تشفير كلمة المرور في هيدر كل طلب باستخدام `encryptAes256Cbc()` ومفتاح التشفير المخصص. تأكد من أن المفتاح مطابق لما هو معطى من CashPay.
- **تغيير كلمة المرور:** عند ظهور رمز خطأ `6022`، يجب استدعاء `changePassword()` فوراً (يمكن إظهار رسالة للمسؤول أو توجيهه لتغيير كلمة المرور في لوحة التحكم).
- **التحقق من المعاملة السابقة:** في `process()`، يتم التحقق أولاً من وجود معاملة سابقة مرتبطة بالطلب. إذا كانت حالتها `success` يتم إكمال الطلب فوراً، مما يمنع إنشاء معاملات مكررة.
- **استخدام `HttpHelper`:** جميع طلبات API تستخدم `HttpHelper` الموحد، مما يسهل تتبع الأخطاء وتوسيع البوابة.
- **دعم العملات:** تم تضمين دوال `getSupportedCurrencies()`, `getCurrencyId()`, `isCurrencySupported()` للتحقق من العملات المدعومة وتحويل الرموز إلى الأرقام الصحيحة.
- **دعم `RedirectHelper`:** تم تضمين إمكانية إعادة التوجيه بشكل كامل، مما يجعل البوابة مناسبة لتطبيقات الجوال التي تستخدم Deep Links.
- **قابلية التوسع:** يمكن إضافة دعم Webhook أو استرداد المبالغ مستقبلاً عبر دوال إضافية تستخدم نقاط نهاية CashPay المختلفة.

---

## 9. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة (CashPay).

---

## 10. فترة التطوير والاختبار

تم تطوير بوابة CashPay خلال الفترة من **15 مايو 2026** وحتى **20 مايو 2026**. تضمنت هذه الفترة:
- كتابة الكود الأساسي لكلاس `CashPay` (يمتد من `PaymentProvider`).
- تنفيذ دوال `process()`, `complete()`, `initPayment()`, `confirmPayment()`, `operationStatus()`, `changePassword()`.
- إضافة التحقق من المعاملة السابقة في `process()` لمنع التكرار.
- تنفيذ آلية تشفير كلمة المرور باستخدام AES‑256‑CBC.
- إنشاء ملفات الواجهة الجزئية (`_info.htm`, `_test_info.htm`) وواجهة الاختبار المتكاملة (`cashpay-ui.htm`).
- كتابة نقاط نهاية API اللازمة للاختبار والمراقبة في `routes.php` (10 نقاط نهاية).
- إضافة مفاتيح الترجمة في ملفات اللغة.
- تسجيل البوابة في `Plugin.php`.
- إجراء اختبارات تكامل شاملة للتحقق من صحة العمل مع بيئة CashPay (محاكاة باستخدام Postman و UI).

---

## 11. روابط ذات صلة

- [توثيق CashPay (دليل المطور)](./docs/CashPay/Docs-CashPay-ar.md)
- [وثيقة SKILL-ar.md لإنشاء بوابات الدفع](./docs/CashPay/SKILL-ar.md)
- [وثيقة Cash-Pay API](./docs/CashPay/external/Cash-Pay%20API%20Doc2.pdf)
- [كلاس CashPay.php](./Nano/Yepayment/PaymentTypes/CashPay.php)
- [ملف routes.php الخاص بـ Nano.Yepayment](./routes.php)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft

## 2026-05-18 – 2026-05-24

**إضافة وتحديث بوابة دفع FloosakPay (محفظة فلوسك) ضمن الوحدة البرمجية `Nano.Yepayment`**

---

## 1. مقدمة

تم تطوير طريقة دفع جديدة باسم **FloosakPay** وإضافتها إلى نظام المدفوعات الموحد في نانوسوفت ضمن الوحدة `Nano.Yepayment`. تأتي هذه الإضافة استجابة للحاجة إلى دعم بوابات الدفع عبر المحافظ الرقمية في اليمن، حيث تُعد **محفظة فلوسك (Floosak Wallet)** أحد الحلول المالية الرقمية الرائدة التي تتيح للمستخدمين إجراء المدفوعات الإلكترونية بسهولة وأمان.

تعتمد FloosakPay على واجهة برمجة تطبيقات **Floosak Wallet API** (الإصدار v1)، والتي تتيح:
- مصادقة التاجر عبر `/api/v1/auth/login` والحصول على `Bearer token`.
- إنشاء عملية شراء معلقة (`pending purchase`) عبر `/api/v1/merchant/p2mcl`.
- تأكيد الدفع باستخدام رمز OTP عبر `/api/v1/merchant/p2mcl/confirm`.
- الاستعلام عن حالة المعاملة عبر `/api/v1/merchant/check-status`.
- استرداد المبلغ (refund) عبر `/api/v1/merchant/p2mcl/refund`.

تم تصميم البوابة وفق **نمط الدفع من خطوتين (Two‑Step) مع تأكيد OTP**، حيث:
1. **الخطوة الأولى (`process`):** يتم إنشاء معاملة معلقة (`Send`)، وإرسال OTP إلى جوال العميل، وتُعاد بيانات المعاملة (`purchase_id`). **لا يتغير** الطلب إلى حالة مدفوع.
2. **الخطوة الثانية (`complete`):** بعد إدخال العميل للرمز، يتم تأكيد الدفع باستخدام OTP. إذا كان الرمز صحيحاً، يتم تحديث الطلب إلى `PaidState` وتفريغ السلة.

هذا التصميم يضمن التوافق الكامل مع `OrderManager` و `Checkout2` و `PaymentResult`، ويمنع تغيير حالة الطلب إلى مدفوع قبل تأكيد الرمز الصحيح من العميل.

تم تطوير البوابة وفق أعلى معايير الأمان والجودة، مع تخزين التوكنات في Cache، واستخدام `HttpHelper` الموحد لجميع طلبات API، وتوفير واجهات اختبار متكاملة ونقاط نهاية API مخصصة للمطورين والمسؤولين. كما تم دعم `RedirectHelper` لضمان التوافق مع تطبيقات الجوال التي تحتاج إلى روابط عودة بعد الدفع، وآلية `Idempotency` عبر `request_id` لمنع تكرار المعاملات، وتسجيل `PaymentLog` في المرحلة الأولى لتتبع المحاولات غير المكتملة.

---

## 2. المكونات المطورة

### 2.1. `FloosakPay` – كلاس بوابة الدفع الرئيسي

- **المسار:** `Nano\Yepayment\PaymentTypes\FloosakPay`
- **الوراثة:** يمتد من `Nano\MicroCart\Classes\Payments\PaymentProvider`
- **الوظيفة:** مسؤول عن المصادقة عبر `/api/v1/auth/login`، إنشاء معاملة معلقة (Send)، تأكيد الدفع (Confirm)، الاستعلام عن الحالة (Check Status)، واسترداد المبلغ (Refund).

**الدوال الرئيسية:**

| الدالة | الوصف |
|--------|-------|
| `process(PaymentResult $result)` | **الخطوة الأولى:** التحقق من بيانات الإدخال (`target_phone`, `amount`, `purpose`)، التحقق من وجود معاملة سابقة (Idempotency)، الحصول على توكن، إرسال طلب Send، حفظ `request_id` و `purchase_id` في الطلب، تسجيل `PaymentLog` بحالة `initiated`، ثم إرجاع `successful = true` مع رسالة تطلب OTP (**لا تستدعي `$result->success()`**). |
| `complete(PaymentResult $result)` | **الخطوة الثانية:** استخراج `purchase_id`، الحصول على توكن جديد، إرسال طلب Confirm مع OTP، إذا نجحت وكانت الحالة `Completed` يتم استدعاء `$result->success()` لتحديث الطلب إلى `PaidState`. |
| `reverse(string $refNo, string $reason = ''): array` | استرداد مبلغ معاملة مكتملة عبر `/api/v1/merchant/p2mcl/refund`. عند النجاح، تغيير حالة الطلب إلى `RefundedState` وتسجيل بيانات الاسترداد في `other_data`. |
| `checkTransactionStatus(string $requestId): array` | الاستعلام عن حالة معاملة باستخدام `request_id` عبر `/api/v1/merchant/check-status`. تُستخدم في منع التكرار وأدوات الاختبار. |
| `getTransactionStatusForOrder(string $orderId): array` | دالة مساعدة لاستعلام الحالة باستخدام `order_id` مباشرة. |
| `getAuthToken(bool $useCache = true): ?string` | الحصول على `Bearer token` عبر `POST /api/v1/auth/login` (مع `x-channel: merchant`). يتم تخزين التوكن في Cache لمدة 3500 ثانية. |
| `sendPayment(string $token): array` | إرسال طلب `POST /api/v1/merchant/p2mcl` لإنشاء معاملة معلقة. تولد `request_id` كـ UUID. تعيد `purchase_id`, `reference_id`, `fee`, `gross`. |
| `confirmPayment(string $token, int $purchaseId, string $otp): array` | إرسال طلب `POST /api/v1/merchant/p2mcl/confirm` لتأكيد الدفع. تتحقق من أن `status.en == "Completed"`. |
| `refundPayment(string $token, int $transactionId, string $requestId, float $amount, string $reason): array` | إرسال طلب `POST /api/v1/merchant/p2mcl/refund`. |
| `settings(): array` | تعريف حقول الإعدادات في لوحة التحكم (URL, phone, password, source_wallet_id, default_purpose). |
| `encryptedSettings(): array` | تحديد الحقول التي تخزن بشكل مشفر (`floosakpay_password`). |
| `defineValidationRules(): array` | قواعد التحقق من `target_phone`, `amount`, `purpose`. |
| `handleSpecificErrors(array $responseData, string $defaultMessage): array` | استخراج رسائل الخطأ من استجابة Floosak (تدعم `errors` و `message`). |

### 2.2. آلية Idempotency (منع تكرار المعاملات)

- في `process()`، يتم البحث عن معاملة سابقة عبر `payment_first_trans_id` أو `other_data['floosakpay']['request_id']`.
- إذا وُجدت، يتم استدعاء `checkTransactionStatus()`:
  - إذا كانت الحالة `Completed` → يتم استدعاء `$result->success()` مباشرة (الطلب مدفوع مسبقاً).
  - إذا كانت الحالة `Pending` → يتم إعلام المستخدم بانتظار OTP دون إنشاء معاملة جديدة.
  - إذا كانت الحالة غير معروفة أو فاشلة → يتم إنشاء معاملة جديدة.
- هذا يضمن عدم إنشاء معاملات مكررة عند إعادة محاولة المستخدم لعملية الدفع.

### 2.3. تسجيل `PaymentLog` في المرحلة الأولى

- بعد إنشاء المعاملة بنجاح، يتم تسجيل `PaymentLog` يدوياً عبر:
  ```php
  $paymentLog = $result->logSuccessfulPayment($sendResult, null, 'initiated');
  $this->order->payment_id = $paymentLog->id;
  $this->order->save();
  ```
- هذا يسمح بتتبع محاولات الدفع حتى لو لم يكمل العميل إدخال OTP، مما يساعد في تحليل معدل الإكمال.

### 2.4. ملفات جزئية (Partials) وإعدادات

| الملف | المسار | الوصف |
|-------|--------|-------|
| `_info.htm` | `paymenttypes/floosakpay/_info.htm` | عرض إرشادات الإعداد ومعلومات البوابة في لوحة التحكم. |
| `_test_info.htm` | `paymenttypes/floosakpay/_test_info.htm` | أدوات اختبار سريع داخل صفحة الإعدادات (أزرار المصادقة، إنشاء معاملة، تأكيد OTP، استعلام، استرداد). |
| `floosakpay-ui.htm` | `views/floosakpay-ui.htm` | واجهة ويب تفاعلية متكاملة لاختبار جميع وظائف FloosakPay (يدوي، تلقائي، إحصائيات، سجلات). |

### 2.5. نقاط نهاية API (`routes.php`)

تم إضافة مجموعة كاملة من المسارات ضمن مجموعة `yepayment` لدعم الاختبار والمراقبة:

| المسار | الطريقة | الوصف |
|--------|---------|-------|
| `/floosakpay/test-auth` | POST | اختبار المصادقة والحصول على Bearer Token. |
| `/floosakpay/test-create-payment` | POST | إنشاء معاملة معلقة (محاكاة `process`). |
| `/floosakpay/test-confirm-payment` | POST | تأكيد الدفع باستخدام OTP (محاكاة `complete`). |
| `/floosakpay/test-check-status` | GET | الاستعلام عن حالة معاملة باستخدام `request_id` أو `order_id`. |
| `/floosakpay/test-refund` | POST | استرداد مبلغ معاملة مكتملة. |
| `/floosakpay/test-full-payment` | POST | اختبار شامل (إنشاء + تأكيد + استعلام) في خطوة واحدة. |
| `/floosakpay/stats` | GET | إحصائيات استخدام البوابة (عدد الطلبات، نسبة النجاح، السجلات الحديثة). |
| `/floosakpay/logs` | GET | سجلات المعاملات من قاعدة البيانات (للمسؤول). |
| `/floosakpay/test-ui` | GET | واجهة الاختبار المتكاملة (HTML). |
| `/floosakpay/success` | GET | نقطة نهاية لتأكيد الدفع بعد إدخال OTP (تستخدم `checkAndCompletePay`). |
| `/floosakpay/cancel` | GET | نقطة نهاية اختيارية لإعادة التوجيه عند الإلغاء. |

جميع المسارات (باستثناء success/cancel) محمية بـ `BackendAuth::getUser()` لضمان وصول المسؤولين فقط.

### 2.6. `Plugin.php` – تسجيل مزود الدفع

تمت إضافة السطر التالي في دالة `registerPaymentProviders()` ضمن قسم `allow_yemen_payment`:

```php
if (class_exists(\Nano\Yepayment\PaymentTypes\FloosakPay::class)) {
    $providers[] = new \Nano\Yepayment\PaymentTypes\FloosakPay();
}
```

### 2.7. ملفات اللغة (Translation)

تم إضافة مفاتيح الترجمة في `lang/ar/lang.php` و `lang/en/lang.php` لكل من الإعدادات ورسائل العموم والأخطاء (مثل `auth_success`, `confirmation_required`, `payment_success`, `order_already_paid`, `auth_failed`, `missing_credentials`, `missing_wallet_id`).

---

## 3. دورة عمل الدفع (Payment Workflow) – نمط Two‑Step (OTP)

تعمل FloosakPay وفق تدفق **Two‑Step** مع تأكيد OTP:

### 3.1. المصادقة (Authentication)

يتم استدعاء `getAuthToken()`:
- التحقق من وجود `access_token` في Cache (صلاحية 3500 ثانية).
- إذا لم يكن موجوداً، يتم إرسال طلب `POST /api/v1/auth/login` مع `phone` و `password` و `x-channel: merchant`.
- يتم تخزين التوكن في Cache لمدة أقل بقليل من صلاحيته الفعلية.

### 3.2. الخطوة الأولى – إنشاء معاملة معلقة (`process`)

- التحقق من صحة البيانات (`target_phone`, `amount`, `purpose`) باستخدام `Validator`.
- التأكد من أن الطلب ليس مدفوعاً مسبقاً (`payment_state != PaidState`).
- **التحقق من وجود معاملة سابقة (Idempotency):**
  - البحث عن `request_id` في `payment_first_trans_id` أو `other_data['floosakpay']['request_id']`.
  - إذا وُجد، استدعاء `checkTransactionStatus()`:
    - إذا `Completed` → استدعاء `$result->success()` (الطلب مدفوع مسبقاً).
    - إذا `Pending` → إرجاع رسالة تطلب OTP بدون إنشاء معاملة جديدة.
    - إذا غير ذلك → متابعة الإنشاء.
- **لا توجد معاملة سابقة أو الحالة غير صالحة:**
  - الحصول على `access_token`.
  - إرسال طلب `POST /api/v1/merchant/p2mcl` مع:
    - `source_wallet_id` (من الإعدادات)
    - `request_id` (UUID فريد)
    - `target_phone`, `amount`, `purpose`
  - استلام `purchase_id`, `reference_id`, `fee`, `gross`.
  - حفظ `request_id` في `order.payment_first_trans_id`.
  - حفظ `purchase_id` في `order.payment_trans_id`.
  - تخزين البيانات الإضافية في `order.other_data['floosakpay']` (الرقم، المبلغ، الغرض، روابط العودة، إلخ).
  - **لا يتم استدعاء `$result->success()`**، ولا يتغير الطلب إلى `PaidState`.
  - تسجيل `PaymentLog` بحالة `'initiated'`.
  - إرجاع `$result->successful = true` مع رسالة "يرجى إدخال رمز التأكيد".

### 3.3. الخطوة الثانية – تأكيد الدفع (`complete`)

- تُستدعى من مسار `floosakpay/success` بعد أن يدخل العميل OTP.
- استخراج `purchase_id` من `order.payment_trans_id`.
- الحصول على `access_token` جديد (قد يكون منتهياً).
- إرسال طلب `POST /api/v1/merchant/p2mcl/confirm` مع `otp` و `purchase_id`.
- تحليل الاستجابة:
  - إذا كان `is_success == true` و `status.en == "Completed"`:
    - يتم استدعاء `$result->success()` التي:
      - تغير `payment_state` إلى `PaidState`.
      - تضع `processed = true`.
      - تُخطر النظام بالأحداث المرتبطة (إرسال إيميل، تحديث المخزون، تفريغ السلة).
  - إذا كان `status.en == "Pending"` → إرجاع رسالة بأن الرمز غير صحيح أو المعاملة لا تزال معلقة.
- أي حالة أخرى تعتبر فشلاً.

### 3.4. روابط العودة الاختيارية (Callbacks)

عند تمرير `callback_success_url` أو `callback_error_url` في بيانات الدفع، يتم تخزينها في `other_data`. بعد التأكيد، يمكن للمطور استخدام `RedirectHelper` في مسار `success` لإعادة توجيه المستخدم بذكاء (Deep Link أو ويب) مع تمرير نتيجة الدفع.

---

## 4. الإعدادات والتهيئة (Configuration)

لتفعيل FloosakPay، يجب إدخال الإعدادات التالية في واجهة إعدادات بوابة الدفع في نظام نانوسوفت (`Nano\MicroCart\Models\PaymentGatewaySettings`):

| الإعداد | المفتاح | الوصف | القيمة الافتراضية |
|---------|---------|-------|-------------------|
| رابط API الأساسي | `floosakpay_url` | عنوان API لبيئة الإنتاج | `https://staging.fintech-expert.net` |
| رابط API للاختبار | `floosakpay_test_url` | عنوان API لبيئة الاختبار (اختياري) | `https://staging.fintech-expert.net` |
| رقم هاتف التاجر | `floosakpay_phone` | رقم الجوال المستخدم في المصادقة (مع رمز الدولة) | - |
| كلمة المرور | `floosakpay_password` | كلمة مرور حساب التاجر (تخزن مشفرة) | - |
| معرف المحفظة المصدر | `floosakpay_source_wallet_id` | معرف المحفظة التي سيتم السحب منها | - |
| الغرض الافتراضي | `floosakpay_default_purpose` | نص يُرسل كوصف للمعاملة | `دفع الطلب` |

**ملاحظات أمنية:**
- `floosakpay_password` يتم تخزينه مشفراً عبر `encryptedSettings()`.
- لا يتم تسجيل هذه القيم الحساسة في السجلات.

---

## 5. الأمثلة التوضيحية (Usage Examples)

### 5.1. بدء الدفع – الخطوة الأولى (إنشاء معاملة معلقة)

```php
use Nano\Yepayment\PaymentTypes\FloosakPay;
use Nano\MicroCart\Classes\Payments\PaymentResult;

$order = Order::find(200);
$floosak = new FloosakPay($order, [
    'target_phone' => '967771234567',
    'amount'       => 100,
    'purpose'      => 'دفع الطلب #200',
    'callback_success_url' => 'myapp://pay/success',
]);
$result = new PaymentResult($floosak, $order);
$processResult = $floosak->process($result);

if ($processResult->successful && !$processResult->redirect) {
    // عرض واجهة إدخال OTP للمستخدم
    $purchaseId = $order->payment_trans_id;
    $requestId  = $order->payment_first_trans_id;
}
```

### 5.2. تأكيد الدفع – الخطوة الثانية (إدخال OTP)

بعد أن يدخل العميل الرمز، يمكن تأكيد العملية كالتالي:

```php
$floosak = new FloosakPay($order, ['otp' => '123456']);
$result = new PaymentResult($floosak, $order);
$completeResult = $floosak->complete($result);

if ($completeResult->successful) {
    // تم تأكيد الدفع وتحديث الطلب إلى PaidState
}
```

أو باستخدام الدالة الثابتة `checkAndCompletePay`:

```php
$result = \Nano\Yepayment\PaymentTypes\FloosakPay::checkAndCompletePay(['order_id' => 200, 'otp' => '123456']);
if ($result['success']) {
    // نجاح
}
```

### 5.3. الاستعلام المباشر عن حالة معاملة

```php
$floosak = new FloosakPay();
$status = $floosak->checkTransactionStatus('550e8400-...');
if ($status['success'] && $status['completed']) {
    echo "المعاملة مكتملة";
}
```

### 5.4. استرداد مبلغ معاملة

```php
$order = Order::find(200);
$floosak = new FloosakPay($order);
$refundResult = $floosak->reverse($order->payment_first_trans_id, 'استرداد بناءً على طلب العميل');
if ($refundResult['success']) {
    // تم الاسترداد، حالة الطلب أصبحت RefundedState
}
```

### 5.5. استخدام واجهة الاختبار المتكاملة

بعد تسجيل الدخول إلى لوحة التحكم كمسؤول، افتح الرابط:
```
https://yourdomain.com/api/v1/yepayment/floosakpay/test-ui
```

الواجهة تتيح:
- اختبار المصادقة (التحقق من صحة phone, password, source_wallet_id).
- إنشاء معاملة معلقة (محاكاة `process`) وعرض `purchase_id` و `request_id`.
- تأكيد الدفع باستخدام OTP (يدوياً أو افتراضي `123456` في بيئة الاختبار).
- الاستعلام عن حالة معاملة.
- استرداد المبلغ.
- اختبار شامل (إنشاء + تأكيد + استعلام) في خطوة واحدة.
- اختبار تلقائي بعدد مرات محدد (حتى 5 مرات).
- إحصائيات الاستخدام (عدد الطلبات، نسبة النجاح، السجلات الحديثة).
- سجلات محلية للاختبارات (تخزن في `localStorage`).

---

## 6. التعامل مع بيئة الاختبار (Sandbox)

- في بيئة الاختبار، يتم تعيين `x-channel: merchant` كما في الإنتاج.
- قيمة OTP ثابتة: **`123456`** لأغراض الاختبار (لا يتم إرسال رسالة SMS حقيقية).
- يمكن استخدام رابط الاختبار `https://staging.fintech-expert.net` (يُحدد في حقل `floosakpay_test_url`).
- تأكد من أن `source_wallet_id` صالح في بيئة الاختبار.

---

## 7. القيمة المضافة

- **للمطورين:** نموذج متكامل لبوابة دفع من نوع **Two‑Step** باستخدام OTP، مع آلية Idempotency، وتخزين Cache للتوكنات، وتسجيل PaymentLog في المرحلة الأولى. يُظهر كيفية التعامل مع واجهات API التي تتطلب `x-channel` وجسم JSON قياسي.
- **للتجار:** دعم محفظة فلوسك كطريقة دفع رقمية، مما يتيح للعملاء الدفع مباشرة من محافظهم الإلكترونية دون الحاجة إلى بطاقات ائتمان.
- **للمستخدمين النهائيين:** تجربة دفع سلسة عبر OTP، حيث يتلقى العميل رمزاً على هاتفه ويؤكده داخل التطبيق، مما يعزز الأمان.

---

## 8. اختبار التكامل (Integration Testing)

تم توفير عدة طبقات للاختبار:

- **بيئة الاختبار:** يمكن استخدام رابط `staging.fintech-expert.net` (يُحدد في حقل `floosakpay_test_url`). OTP ثابت `123456`.
- **واجهة الاختبار السريع:** من خلال صفحة إعدادات البوابة (الملف الجزئي `_test_info.htm`) يمكن اختبار المصادقة، إنشاء معاملة، تأكيد، واستعلام مباشرة.
- **واجهة الاختبار المتكاملة:** `/api/v1/yepayment/floosakpay/test-ui` توفر جميع الأدوات اللازمة لاختبار البوابة بشكل كامل (يدوي، تلقائي، إحصائيات، سجلات).
- **نقاط نهاية API مستقلة:** يمكن للمطورين استخدام أدوات مثل Postman أو cURL للاتصال المباشر بالنقاط مثل `/floosakpay/test-create-payment` و `/floosakpay/test-confirm-payment`.

**خطوات اختبار سريعة:**
1. تأكد من إعداد `phone`, `password`, `source_wallet_id` في صفحة إعدادات البوابة.
2. افتح `/api/v1/yepayment/floosakpay/test-ui`.
3. انقر على "اختبار المصادقة" للتحقق من صحة البيانات.
4. أدخل رقم طلب ورقم جوال العميل ومبلغ، ثم انقر "إنشاء معاملة".
5. سيظهر `purchase_id` ورسالة "يرجى إدخال OTP". استخدم OTP `123456`.
6. انقر "تأكيد الدفع" – يجب أن يعود بنجاح وتتغير حالة الطلب إلى مدفوع.
7. جرب "الاستعلام عن الحالة" و "استرداد المبلغ".

---

## 9. ملاحظات للمطورين (Developer Notes)

- **نمط Two‑Step:** `process()` **لا** يُكمل الدفع، بل ينشئ معاملة معلقة ويعيد رسالة تطلب OTP. يجب استدعاء `complete()` (أو `checkAndCompletePay`) بعد إدخال العميل للرمز لتأكيد الدفع وتحديث الطلب إلى `PaidState`.
- **Idempotency:** يتم التحقق من وجود معاملة سابقة في بداية `process()` لمنع تكرار الإنشاء. هذا مهم جداً عند إعادة تحميل صفحة الدفع أو الضغط على زر "ادفع" عدة مرات.
- **التخزين المؤقت للتوكن:** يتم تخزين Bearer Token في Cache لمدة 3500 ثانية (أقل من صلاحية التوكن الفعلية البالغة 3600 ثانية) لتجنب استخدام توكن منتهي.
- **استخدام `HttpHelper`:** جميع طلبات API تستخدم `HttpHelper::sendJson()` مع الهيدرز المناسبة (`x-channel: merchant`).
- **تسجيل `PaymentLog`:** يتم تسجيل محاولة الدفع في المرحلة الأولى (حالة `initiated`) حتى لو لم يكمل العميل OTP، مما يساعد في تتبع وتحليل معدل الإكمال.
- **دعم `RedirectHelper`:** تم تضمين إمكانية إعادة التوجيه بشكل كامل، مما يجعل البوابة مناسبة لتطبيقات الجوال التي تستخدم Deep Links.
- **اختبار OTP:** في بيئة الاختبار، OTP ثابت `123456`. لا يتم إرسال رسالة حقيقية.
- **قابلية التوسع:** يمكن إضافة دعم Webhook أو واجهات إضافية مستقبلاً باستخدام نقاط النهاية المتاحة.

---

## 10. إصلاحات الأخطاء (Bug Fixes)

لا يوجد – هذا الإصدار مخصص لإضافة ميزة جديدة (FloosakPay).

---

## 11. فترة التطوير والاختبار

تم تطوير بوابة FloosakPay خلال الفترة من **18 مايو 2026** وحتى **24 مايو 2026**. تضمنت هذه الفترة:
- كتابة الكود الأساسي لكلاس `FloosakPay` (يمتد من `PaymentProvider`).
- تنفيذ دوال `process()`, `complete()`, `reverse()`, `getAuthToken()`, `sendPayment()`, `confirmPayment()`, `checkTransactionStatus()`.
- إضافة آلية Idempotency (التحقق من معاملة سابقة).
- إضافة تسجيل `PaymentLog` في `process()` بحالة `initiated`.
- إنشاء ملفات الواجهة الجزئية (`_info.htm`, `_test_info.htm`) وواجهة الاختبار المتكاملة (`floosakpay-ui.htm`).
- كتابة نقاط نهاية API اللازمة للاختبار والمراقبة في `routes.php` (11 نقطة نهاية).
- إضافة مفاتيح الترجمة في ملفات اللغة.
- تسجيل البوابة في `Plugin.php`.
- إجراء اختبارات تكامل شاملة للتحقق من صحة العمل مع بيئة Floosak (محاكاة باستخدام Postman و UI).

---

## 12. روابط ذات صلة

- [توثيق FloosakPay (دليل المطور)](./docs/FloosakPay/Docs-FloosakPay-ar.md)
- [وثيقة SKILL-v2.5 لإنشاء بوابات الدفع](./docs/FloosakPay/SKILL-v2.5.md)
- [وثيقة API الرسمية من Floosak](./docs/FloosakPay/external/Merchant_Wallet_Integration_API_Documentation_RefStyle.pdf)
- [مجموعة Postman لاختبار FloosakPay](./docs/FloosakPay/external/Marchant%20Last%20Version.json)
- [كلاس FloosakPay.php](./Nano/Yepayment/PaymentTypes/FloosakPay.php)
- [ملف routes.php الخاص بـ Nano.Yepayment (قسم FloosakPay)](./routes.php)
- [قناة الدعم الفني](https://nano2soft.com)

---

**تم إعداد هذا التحديث بواسطة:**  
فريق تطوير نانوسوفت – قسم المدفوعات الإلكترونية  
**المراجع:** Dheia Ali, Nano2Soft

## 2026-05-24 - 2026-05-25

**تحديث إضافة `Nano.AuthApi` – الإصدار 1.0.20**

### إضافة كلاس `AccessManager` لإدارة الصلاحيات والوصول بشكل مركزي ومتقدم

---

### ملخص التحديثات

يقدم الإصدار **1.0.20** من إضافة `Nano.AuthApi` كلاساً جديداً هو `AccessManager`، وهو مسؤول عن إدارة صلاحيات المستخدمين (Backend، Frontend، ضيف) والتحكم في نطاق الوصول (Access Scopes) بطريقة موحدة وقابلة للتوسع. يدمج الكلاس دوال مساعدة من `BasicHelper` (مثل `checkAccessAllCompanys`) ويوفر واجهة بسيطة لتطبيق قيود الوصول على استعلامات Eloquent.

يهدف هذا التحديث إلى توحيد منطق الصلاحيات عبر جميع إضافات النظام، وتقليل الكود المكرر، ودعم سيناريوهات متقدمة مثل الوصول حسب الشركة أو القسم أو الولاية، أو عرض السجلات التي أنشأها المستخدم فقط، أو عرض أبناء ولي الأمر.

---

### أهداف الإصدار

- **توفير كلاس مركزي** لإدارة الصلاحيات بدلاً من توزيع المنطق في متحكمات متعددة.
- **دعم أنواع المستخدمين الثلاثة:** Backend (Nano Soft App)، Frontend (RainLab.User أو أي نموذج مخصص)، Guest.
- **دعم نطاقات وصول متعددة (Scopes):**
  - `all`: الوصول إلى كل السجلات.
  - `own`: السجلات التي أنشأها المستخدم (عبر `created_by`).
  - `created_by`: نفس `own`.
  - `company`: الوصول إلى سجلات شركة محددة فقط (مع إمكانية الوصول إلى كل الشركات للمستخدمين المخولين).
  - `department`: الوصول إلى سجلات قسم معين فقط.
  - `state`: الوصول إلى سجلات ولاية معينة فقط.
  - `children`: لمستخدمي Frontend من نوع ولي أمر، لعرض سجلات أبنائه.
- **توفير دوال لتطبيق النطاق على Query Builder** (`applyAccessScope`, `applyCompanyScope`, `applyDepartmentScope`, `applyStateScope`, `applyCreatedByScope`, `applyUserScope`, `applyChildrenScope`).
- **دعم تخصيص أسماء الحقول** (مثل `companys_id`, `departments_id`, `state_id`, `created_by`) عبر خيارات مرنة.
- **دعم التحقق من الصلاحيات (Permissions)** لمستخدمي Backend باستخدام `hasAccess` أو `hasPermission`.
- **دعم تصفية أنواع المستخدمين Frontend** عبر `ref_type` (مثل `student`, `parent`).
- **إرجاع معلومات مفصلة** عن نتيجة التحقق (السماح، النطاق، الرسالة، تفاصيل النطاق المطبق).

---

### الميزات الجديدة والتحسينات

#### 1. كلاس `AccessManager` – الوظائف الأساسية

- **`setUser($user) / getUser()`**: تعيين أو جلب المستخدم الحالي (يدعم الجلب التلقائي من `AuthHelpers`, `BackendAuth`, `Auth`).
- **`getUserType($user)`**: يحدد نوع المستخدم (`backend`, `frontend`, `guest`).
- **`getUserRefType($user)`, `getUserRefId($user)`**: يستخرج `ref_type` و `ref_id` (للمستخدمين المرتبطين بنماذج مثل الطلاب).
- **`getUserCompanyId($user)`, `getUserDepartmentId($user)`**: يستخرج معرف الشركة والقسم من المستخدم (إذا كان يمتلك الخصائص).
- **`check(string $operation, array $config, $user)`**: دالة التحقق الأساسية. تعيد مصفوفة تحتوي على `allowed`, `scope`, `message`, `user_type`, `user`, `config`, `applied_scope_details`.

#### 2. دعم نطاقات متقدمة لـ Backend

يمكن تخصيص سلوك Backend عبر خيارات `backend` في مصفوفة `$config`:

```php
'backend' => [
    'allow' => true,                         // هل مسموح لمستخدمي Backend
    'check_permission' => true,              // هل نتحقق من الصلاحيات
    'permissions' => ['tss.example.access'], // الصلاحيات المطلوبة
    'access_scope' => 'company',             // النطاق المطلوب
    'company_field' => 'companys_id',        // حقل الشركة في الجدول
    'department_field' => 'departments_id',
    'state_field' => 'state_id',
    'created_by_field' => 'created_by',
    'company_id' => 2,                       // رقم شركة محدد (اختياري)
    'department_id' => 5,
    'state_id' => 1,
],
```

يدعم `access_scope` القيم: `all`, `own`, `created_by`, `company`, `department`, `state`.

عند استخدام نطاق `company` أو `department` أو `state`، يتحقق `AccessManager` أولاً مما إذا كان المستخدم يملك صلاحية الوصول إلى كل الشركات/الأقسام/الولايات (عبر `canAccessAllCompanies()` وغيرها). إذا كان يملك، يصبح النطاق الفعلي `all`. وإلا، يستخدم المعرف المرتبط بالمستخدم أو المعرف الممرر في الإعدادات.

#### 3. دعم Frontend مع تصفية `ref_type` ونطاق `children`

```php
'frontend' => [
    'allow' => true,
    'allowed_ref_types' => ['student', 'parent'], // الأنواع المسموحة
    'access_scope' => 'own',                     // 'own' أو 'children'
    'children_relation' => 'students',           // اسم العلاقة لجلب الأبناء
],
```

إذا كان النطاق `children`، يجب تعريف `children_relation` (مثل `students` في نموذج ولي الأمر). ثم يمكن استخدام `applyChildrenScope()` لتطبيق الفلترة.

#### 4. دوال تطبيق النطاق على Query Builder

- **`applyAccessScope($query, $accessResult, $options)`**: دالة عامة تستخدم نتيجة `check()` لتطبيق النطاق المناسب.
- **`applyCompanyScope($query, $companyId, $field)`**, **`applyDepartmentScope`**, **`applyStateScope`**, **`applyCreatedByScope`**, **`applyUserScope`**.
- **`applyChildrenScope($query, $parent, $studentIdField, $relation)`**: تطبق فلترة لجلب سجلات أبناء ولي أمر معين.

**مثال:**

```php
$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed']) {
    $query = Student::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access, [
        'company_field' => 'companys_id',
        'department_field' => 'departments_id',
        'state_field' => 'state_id',
        'created_by_field' => 'created_by',
    ]);
    // إذا كان scope = children
    if ($access['scope'] === 'children') {
        $parent = Mparent::byUser($user)->first();
        $query = AccessManager::instance()->applyChildrenScope($query, $parent, 'student_id', 'students');
    }
}
```

#### 5. دوال مساعدة للتحقق من صلاحيات الشركات والأقسام

- **`canAccessAllCompanies($user)`**: تعيد `true` إذا كان المستخدم يمكنه الوصول إلى كل الشركات (يستخدم `BasicHelper::checkAccessAllCompanys`).
- **`canAccessAllDepartments($user)`**
- **`canAccessAllStates($user)`**
- **`canAccessDepartment($departmentId, $user)`**

هذه الدوال تبسط المنطق وتضمن الاتساق مع بقية النظام.

#### 6. دالة `getPermissionArray`

تعيد مصفوفة مشابهة لتلك المستخدمة في `OrderHelper::getArrayDataAfterPermission`، مما يسهل التكامل مع الكود القديم.

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
// ['allowed'=>true, 'scope'=>'department', 'check_departments'=>true, 'departments_id'=>5, ...]
```

#### 7. دعم متكامل للـ Guest

يمكن السماح للمستخدمين غير المسجلين بالوصول إلى بعض العمليات عبر إعدادات `guest`:

```php
'guest' => [
    'allow' => true,
    'access_scope' => 'none',
],
```

---

### أمثلة على الاستخدام

#### 1. التحقق من صلاحية الوصول إلى قائمة الطلاب لمستخدم Frontend من نوع ولي أمر

```php
$config = [
    'is_allow' => true,
    'frontend' => [
        'allow' => true,
        'allowed_ref_types' => ['parent'],
        'access_scope' => 'children',
        'children_relation' => 'students',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
if (!$access['allowed']) {
    throw new ApplicationException($access['message']);
}
// $access['scope'] = 'children'
```

#### 2. تطبيق نطاق `company` لمستخدم Backend

```php
$config = [
    'backend' => [
        'allow' => true,
        'check_permission' => true,
        'permissions' => ['tss.student.students.access'],
        'access_scope' => 'company',
        'company_field' => 'companys_id',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
if ($access['allowed']) {
    $query = Student::query();
    $query = AccessManager::instance()->applyAccessScope($query, $access);
    // سيتم إضافة شرط companys_id = (معرف الشركة المرتبطة بالمستخدم أو المحدد في config)
}
```

#### 3. الحصول على مصفوفة إعدادات الصلاحيات لاستخدامها في دوال `listExtendQuery`

```php
$perms = AccessManager::instance()->getPermissionArray('list', $config);
if ($perms['allowed']) {
    if ($perms['check_departments']) {
        $query->where('departments_id', $perms['departments_id']);
    }
}
```

---

### التوافق مع الإصدارات السابقة

- **لم يتم تغيير أي دوال قديمة** في `Nano.AuthApi`. تمت إضافة كلاس `AccessManager` فقط.
- يمكن الاستمرار في استخدام دوال `AuthHelpers` القديمة (`getCurrentUser`, `isBackendUser`, إلخ) دون أي تأثير.
- لا توجد تغييرات في قاعدة البيانات أو الإعدادات.

---

### متطلبات الترقية (من أي إصدار سابق إلى 1.0.20)

1. **تحديث الكود**:
   - استبدال ملف `classes/AccessManager.php` بالنسخة الجديدة (المرفقة).
   - (اختياري) تحديث `Plugin.php` إذا كنت تريد تسجيل `AccessManager` كـ Singleton أو توفير alias، لكن غير مطلوب.

2. **لا توجد هجرات قاعدة بيانات** جديدة.

3. **لا توجد تغييرات في الإعدادات** – يظل ملف `config.php` كما هو.

4. **تحديث ملف الإصدارات `version.yaml`**:
   - أضف السطر `1.0.20:` كما هو موضح في بداية هذا المستند.

5. **تنفيذ `php artisan plugin:refresh Nano.AuthApi`** (اختياري) لتسجيل الإصدار الجديد.

6. **اختبار الوظائف**:
   - اختبر `AccessManager::check()` مع سيناريوهات مختلفة (Backend، Frontend، Guest).
   - تأكد من أن `applyAccessScope` تضيف شروط `where` صحيحة.
   - تحقق من دوال `canAccessAllCompanies` وغيرها.

---

### الفوائد والقيمة المضافة

- **توحيد منطق الصلاحيات** عبر جميع إضافات API (`Nano.StudentsApi`, `Nano.AbsenceApi`, ...)، مما يقلل تكرار الكود ويوحد السلوك.
- **تحكم دقيق في الوصول** بناءً على الشركة/القسم/الولاية، مما يتيح بناء أنظمة متعددة الفروع (Multi‑tenancy) بسهولة.
- **دعم طبيعي لنطاق `children`** لولاة الأمور، مما يسهل بناء تطبيقات مدرسية.
- **سهولة التوسع** بإضافة نطاقات جديدة (مثل `team`, `project`) دون تعديل الكلاس الأساسي.
- **تحسين الأمان** من خلال الفصل الواضح بين منطق الصلاحيات ومنطق الأعمال.

---

### الخاتمة

يمثل الإصدار **1.0.20** من `Nano.AuthApi` خطوة مهمة نحو توحيد إدارة الصلاحيات في نظام Nano البيئي. بفضل `AccessManager`، أصبح بإمكان المطورين تطبيق قيود وصول معقدة بسهولة، مع الحفاظ على مرونة التخصيص وقابلية التوسع. نوصي باستخدام هذا الكلاس في جميع إضافات API المستقبلية لضمان اتساق الأمان وسهولة الصيانة.

---

**الوثائق المرجعية**:
- [توثيق كلاس `AccessManager`](./docs/AuthApi/Docs-AccessManager-ar.md)
- [توثيق كلاس `AuthHelpers`](./docs/AuthApi/Docs-AuthHelpers-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md)

## 2026-05-24 – 2026-05-26

**تحديث إضافة `Tss.Student` – الإصدار 1.0.13**

### إعادة هيكلة دوال `getRecords`، إضافة مساعدات متخصصة، ودعم `AccessManager` و `AdvancedQueryHelper`

---

### ملخص التحديثات

يقدم الإصدار **1.0.13** من إضافة `Tss.Student` نقلة نوعية في طريقة جلب بيانات الطلاب وأولياء الأمور وسجلاتهم الدراسية (`StudentRecord`) عبر API ونظام Backend. تم إعادة هيكلة وتوحيد دوال `getRecords` في كلاس `StudentRecordsHelper` جديد، مع إضافة دوال مماثلة لأولياء الأمور (`getParentRecords`) وسجلات الطلاب (`getStudentRecordRecords`). كما تم دمج تقنيات متقدمة مثل `AccessManager` للتحكم المرن في الصلاحيات و `AdvancedQueryHelper` للتصفية الديناميكية.

يهدف هذا الإصدار إلى توفير واجهة موحدة وآمنة وقابلة للتوسع لاستعلامات الطلاب وأولياء الأمور، مع تحسين الأداء من خلال التخزين المؤقت المدمج، ودعم الفلاتر المتقدمة (`is_or`, `is_not`, `is_force`, `is_or_null`) والبحث النصي الذكي.

---

### أهداف الإصدار

- **توحيد منطق جلب السجلات** في كلاس مساعد واحد (`StudentRecordsHelper`) يمكن استخدامه من متحكمات API و Backend.
- **إنشاء دوال متخصصة** لـ:
  - الطلاب (`getRecords`)
  - أولياء الأمور (`getParentRecords`)
  - سجلات الطالب الدراسية (`getStudentRecordRecords`)
- **دمج `AccessManager`** لإدارة الصلاحيات ونطاقات الوصول (`all`, `own`, `created_by`, `company`, `department`, `state`, `children`).
- **الاستفادة من `AdvancedQueryHelper`** لتنفيذ الفلاتر الديناميكية مع دعم المنطق المتقدم (`is_or`, `is_not`, `is_force`, `is_or_null`).
- **توفير هيكل استجابة موحد** لجميع دوال `getRecords` يشمل `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`.
- **دعم التخزين المؤقت (caching)** مع مفاتيح فريدة وعلامات (tags) وإمكانية فرض التحديث.
- **دعم الأحداث** (`event_before_name`, `event_after_name`, `event_list_name`) مع إمكانية تعطيلها.
- **إضافة دوال ربط بين المستخدم والطالب/ولي الأمر** (مثل `getStudentByUser`, `getUserByStudent`, `getMparentByUser`, `getUserByMparent`).
- **إضافة دوال مساعدة خاصة بـ `Mparent`** مثل `getStudentIdsByMparent`, `getStudentsByMparent`, `getStudentsByUser`.
- **إضافة دوال مساعدة لـ `StudentRecord`** مثل `getCurrentStudentRecord`, `getStudentIdFromRecord`.

---

### الميزات الجديدة والتحسينات

#### 1. كلاس `StudentRecordsHelper` – مساعد موحد لجلب السجلات

تم إنشاء كلاس جديد في المسار `Tss\Student\Helpers\StudentRecordsHelper` يحتوي على دالة عامة `getRecords` يمكن استخدامها لأي نموذج، بالإضافة إلى دوال متخصصة للطلاب وأولياء الأمور وسجلات الطالب.

**الخصائص الرئيسية لدالة `getRecords` العامة:**

- **دمج الخيارات الافتراضية** بشكل آمن (مع التحقق من وجود المفاتيح باستخدام `array_key_exists`).
- **استخدام `AdvancedQueryManager::scopeWhereField`** لتطبيق الفلاتر مع دعم `is_or`, `is_not`, `is_force`, `is_or_null`.
- **دعم البحث النصي المتقدم (`q`)** مع خيارات `is_advanced_search` و `is_search_weights`.
- **دعم `with_count`** لجلب عدد العلاقات المرتبطة.
- **دعم `group_by`** (كمصفوفة أو نص) و `having`.
- **دعم التخزين المؤقت** عبر `cache_enabled`, `cache_key`, `cache_ttl`, `cache_tags`, `cache_force_refresh`.
- **دعم الأحداث** (`fire_before_event`, `fire_after_event`) مع أسماء أحداث قابلة للتخصيص.
- **دعم `is_to_sql`** لتسجيل استعلام SQL في `trace_log` لأغراض التصحيح.
- **دعم `withTrashed` و `onlyTrashed`** للعمل مع السجلات المحذوفة ناعماً.
- **دعم خيارات الإخراج المختلفة**: `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`.

#### 2. دالة `getRecords` في `StudentHelper` (لجلب الطلاب)

تم تحديث الدالة الحالية `StudentHelper::getRecords` لتصبح واجهة مباشرة تستدعي `StudentRecordsHelper::getRecords` مع تمرير نموذج `Student`. هذا يضمن التوافق العكسي مع الكود القديم مع الاستفادة من الميزات الجديدة.

**مثال الاستخدام:**

```php
$result = StudentHelper::getRecords([
    'gender' => 'male',
    'is_active' => true,
    'q' => 'محمد',
    'orderBy' => 'full_name',
    'per_page' => 20,
    'cache_enabled' => true,
]);
```

#### 3. دالة `getParentRecords` – جلب أولياء الأمور

تم إضافة دالة جديدة `StudentHelper::getParentRecords` تستدعي `StudentRecordsHelper` مع نموذج `Mparent`. تدعم جميع خيارات التصفية المتوفرة في `getRecords` بالإضافة إلى فلاتر خاصة بـ `Mparent`:

- `students_id`: فلترة أولياء الأمور الذين لديهم أبناء بأرقام محددة.
- `has_students` / `is_has_students`: فلترة حسب وجود أبناء (أو عدم وجودهم).
- `has_students_active` / `is_has_students_active`: فلترة حسب وجود أبناء نشطين.
- `is_has_students_count`: فلترة حسب عدد الأبناء (يدعم مقارنات مثل `['>', 2]` أو `">=3"`).

**مثال الاستخدام:**

```php
$result = StudentHelper::getParentRecords([
    'has_students' => true,
    'is_has_students_count' => ['>=', 2],
    'orderBy' => 'full_name',
]);
```

#### 4. دالة `getStudentRecordRecords` – جلب سجلات الطالب الدراسية

تم إضافة دالة `StudentHelper::getStudentRecordRecords` مخصصة لنموذج `Tss\School\Models\StudentRecord`. تدعم الفلاتر التالية (بالإضافة إلى الفلاتر العامة):

- `student_id`, `year_id`, `class_id`, `group_id`.
- `status_class`, `status_mony`, `semster`, `currencys_id`, `periods_id`, `cost_centers_id`.
- `is_bus`, `is_discount`, `is_registration_fees`.
- فلاتر رقمية مع مقارنات لـ `study_fees`, `registration_fees`, `discount`, `bus_fees` (مثل `['>', 1000]`).
- فلاتر التاريخ (`field_date`, `date_at`, `to_date_at`, `created_at_from`, `created_at_to`, `updated_at_from`, `updated_at_to`).

**مثال الاستخدام:**

```php
$result = StudentHelper::getStudentRecordRecords([
    'year_id' => 3,
    'status_class' => 'continue',
    'study_fees' => ['>', 5000],
    'with_count' => ['student', 'class'],
]);
```

#### 5. دعم `AccessManager` للصلاحيات

تم دمج `AccessManager` في دوال `getRecords` بحيث يمكن تمرير نتيجة التحقق من الصلاحيات (`access_result`) وتطبيق النطاقات تلقائياً على الاستعلام.

**مثال في متحكم API:**

```php
$config = [
    'is_allow' => true,
    'backend' => [
        'allow' => true,
        'check_permission' => true,
        'permissions' => ['tss.student.students.access'],
        'access_scope' => 'all',
    ],
    'frontend' => [
        'allow' => true,
        'allowed_ref_types' => ['student', 'parent'],
        'access_scope' => 'own',
    ],
];
$access = AccessManager::instance()->check('list', $config, $user);
$options['access_result'] = $access;
$result = StudentHelper::getRecords($options);
```

ثم داخل `StudentRecordsHelper::getRecords` يتم استدعاء `AccessManager::instance()->applyAccessScope()` لتطبيق شروط الصلاحية تلقائياً.

#### 6. دوال الربط بين المستخدم والطالب/ولي الأمر

تمت إضافة الدوال التالية في `StudentHelper` (كانت موجودة جزئياً وتم تحسينها):

- `getStudentByUser($user)`: جلب كائن الطالب المرتبط بمستخدم معين.
- `getUserByStudent($student)`: جلب كائن المستخدم المرتبط بطالب.
- `getMparentByUser($user)`: جلب كائن ولي الأمر المرتبط بمستخدم.
- `getUserByMparent($mparent)`: جلب كائن المستخدم المرتبط بولي أمر.
- `getStudentIdsByMparent($mparent, $useCache)`: جلب مصفوفة بأرقام الطلاب التابعين لولي أمر.
- `getStudentsByMparent($mparent, array $options)`: جلب كائنات الطلاب التابعين لولي أمر مع دعم خيارات `getRecords`.
- `getStudentsByUser($user, array $options)`: جلب الطلاب المرتبطين بمستخدم (يدعم الطالب نفسه أو أبناء ولي الأمر).
- `getStudentIdFromRecord($record)`: استخراج معرف الطالب من كائن `StudentRecord` أو معرف السجل.
- `getCurrentStudentRecord($student, array $options)`: جلب سجل الطالب الحالي (عادةً بحالة `continue`) مع دعم خيارات السنة والصف.

**مثال استخدام `getStudentsByUser`:**

```php
$students = StudentHelper::getStudentsByUser(null, ['is_active' => true]);
// إذا كان المستخدم الحالي ولي أمر -> يعيد أبناءه
// إذا كان طالباً -> يعيد بياناته فقط
```

#### 7. تحسين معالجة الفلاتر المتقدمة (AdvancedQueryHelper)

تم تطبيق `AdvancedQueryManager::scopeWhereField` على جميع الفلاتر، مما يسمح بتمرير قيم متعددة مفصولة بفواصل، واستخدام المنطق `is_or`, `is_not`, `is_force`, `is_or_null`. هذا متاح الآن لجميع الحقول القابلة للتصفية في الطلاب وأولياء الأمور وسجلات الطالب.

**مثال:**

```php
$options = [
    'gender' => 'male,female',           // القيم مفصولة بفواصل
    'is_or_gender' => true,              // OR بين القيم
    'is_not_gender' => false,
    'is_force_gender' => true,
    'is_or_null_gender' => false,        // تضمين السجلات التي gender = null
];
```

#### 8. هيكل استجابة موحد

جميع دوال `getRecords` (لطلاب، أولياء الأمور، سجلات الطالب) تعيد الآن مصفوفة موحدة بالصيغة التالية:

```php
[
    'code'    => 200,
    'status'  => true,
    'message' => '...',
    'data'    => $data,          // Builder, Collection, Paginator, أو مصفوفة محولة
    'error'   => null,
    'errors'  => null,
    'input_data'   => $input,    // الخيارات الأصلية
    'process_data' => $processed, // الخيارات بعد الدمج
    'debug'   => [...]            // فقط عند APP_DEBUG=true
]
```

هذا يسهل التعامل مع النتائج في طبقات API ويوفر معلومات تتبع للتطوير.

#### 9. دعم التخزين المؤقت

تمت إضافة خيارات كاملة للتخزين المؤقت:

- `cache_enabled`: تفعيل الكاش (افتراضي `false`، يمكن تفعيله عبر الإعدادات).
- `cache_key`: مفتاح مخصص (يُولّد تلقائياً إذا لم يُقدم).
- `cache_ttl`: مدة الصلاحية بالدقائق (افتراضي `60`).
- `cache_tags`: علامات للكاش (تستخدم مع Redis مثلاً).
- `cache_force_refresh`: تجاهل الكاش وجلب بيانات جديدة.

#### 10. دعم الأحداث

تم إطلاق حدثين قابلين للتخصيص:

- `event_before_name` (افتراضي `api.list.extendQueryBefore`) – قبل تطبيق الفلاتر.
- `event_after_name` (افتراضي `api.list.extendQuery`) – بعد تطبيق الفلاتر وقبل التنفيذ.
- `event_list_name` (افتراضي `StudentsApi.ApiControllers.Students.List`) – بعد الحصول على `Paginator` وقبل التحويل (Transformer).

يمكن تعطيل الأحداث عبر `fire_before_event` و `fire_after_event`.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. جلب الطلاب النشطين مع فلترة المحافظة وترتيب حسب الاسم

```php
$result = StudentHelper::getRecords([
    'is_active' => true,
    'state_id' => '5',
    'orderBy' => 'full_name',
    'orderDirection' => 'asc',
]);
```

#### 2. جلب أولياء الأمور الذين لديهم أكثر من طفلين

```php
$result = StudentHelper::getParentRecords([
    'is_has_students_count' => ['>', 2],
    'with_count' => ['Student'],
]);
```

#### 3. جلب سجلات الطلاب لسنة دراسية معينة ورسوم دراسة أكبر من 5000

```php
$result = StudentHelper::getStudentRecordRecords([
    'year_id' => 3,
    'study_fees' => ['>', 5000],
    'orderBy' => 'study_fees',
    'orderDirection' => 'desc',
]);
```

#### 4. استخدام `getCurrentStudentRecord` لطالب معين

```php
$student = Student::find(100);
$currentRecord = StudentHelper::getCurrentStudentRecord($student, [
    'year_id' => 3,   // إذا لم يُمرر، يُستخدم العام الافتراضي
    'status_class' => 'continue',
]);
if ($currentRecord) {
    echo "الصف: " . $currentRecord->class_id;
}
```

#### 5. استخدام `getStudentsByUser` في متحكم API

```php
public function myChildren()
{
    $user = AuthHelpers::getCurrentUser();
    $students = StudentHelper::getStudentsByUser($user, ['is_active' => true]);
    return $this->respondWithCollection($students, new StudentTransformer);
}
```

---

### التوافق مع الإصدارات السابقة

- **جميع الدوال القديمة في `StudentHelper` بقيت كما هي** (مثل `getStudentRecord`, `getResultRecord`, `setUpStudentRecord`, دوال التواريخ). لم يتم حذف أو تعديل أي دالة قديمة.
- **الدالة `StudentHelper::getRecords` القديمة** أصبحت تستخدم `StudentRecordsHelper::getRecords` بشكل شفاف، مع الحفاظ على نفس بنية المخرجات السابقة (مصفوفة تحتوي على `code`, `status`, `message`, `data`, `error`, `errors`, `input_data`, `process_data`, `debug`).
- **أي كود كان يستخدم `StudentHelper::getRecords` مباشرة سيستمر في العمل** دون تعديل.
- **تم إضافة دوال جديدة فقط** (`getParentRecords`, `getStudentRecordRecords`, دوال الربط الإضافية). لا توجد إزالة أو تعديلات جذرية.

---

### متطلبات الترقية (من 1.0.12 إلى 1.0.13)

1. **تحديث الكود**:
   - استبدال ملف `helpers/StudentHelper.php` بالنسخة الجديدة (التي تستدعي `StudentRecordsHelper` وتحتوي على الدوال الجديدة).
   - إضافة ملف `helpers/StudentRecordsHelper.php` الجديد.
   - (اختياري) إضافة ملف `helpers/ParentRecordsHelper.php` و `StudentRecordRecordsHelper.php` إذا رغبت في فصل الدوال، لكن تم دمجها في `StudentHelper` لتقليل الاعتماديات.

2. **تحديث ملف الإصدارات `version.yaml`**:
   أضف الإصدار الجديد كما يلي:
   ```yaml
   1.0.13:
       - 'Improve StudentHelper and add new helpers with advanced features'
       - 'Add StudentRecordsHelper, ParentRecordsHelper, StudentRecordRecordsHelper'
       - 'Support AccessManager for permissions and AdvancedQueryHelper for dynamic filtering'
       - 'Add unified getRecords, getParentRecords, getStudentRecordRecords methods'
       - 'Enhance caching, event hooks, and sorting scopes'
   ```

3. **تنفيذ الهجرات**: لا توجد هجرات قاعدة بيانات جديدة.

4. **تحديث الإعدادات (اختياري)**:
   يمكن إضافة مفاتيح جديدة في `config.php` لـ `nano.studentsapi::students.cache_enabled` وغيرها، لكن هذا خارج نطاق إضافة `Tss.Student`. يوصى بتحديث إعدادات `Nano.StudentsApi` للاستفادة من التخزين المؤقت.

5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

6. **اختبار الوظائف**:
   - تأكد من أن دوال `getRecords`, `getParentRecords`, `getStudentRecordRecords` تعيد النتائج المتوقعة.
   - اختبر الفلاتر المتقدمة (`is_or`, `is_not`, إلخ) مع قيم متعددة.
   - تأكد من أن التخزين المؤقت يعمل عند تفعيله.
   - تحقق من أن الأحداث (events) تُطلق في الوقت المناسب.

---

### الفوائد والقيمة المضافة

- **توحيد معالجة الاستعلامات** بين الطلاب وأولياء الأمور وسجلات الطالب، مما يقلل تكرار الكود ويوحد السلوك.
- **مرونة عالية في التصفية** بفضل `AdvancedQueryHelper`، مما يسمح للعملاء ببناء استعلامات معقدة بسهولة.
- **أداء أفضل** عبر التخزين المؤقت المدمج وإمكانية استبعاد الأعمدة غير الضرورية.
- **أمان محسّن** عبر دمج `AccessManager` وتطبيق نطاقات الوصول تلقائياً، مما يمنع تسرب بيانات غير مصرح بها.
- **توافق عكسي كامل**، مما يسمح بترقية الإضافة دون الحاجة لتعديل التطبيقات الحالية.
- **قابلية التوسع** من خلال الأحداث التي تسمح لإضافات أخرى بتعديل الاستعلام ديناميكياً.

---

### الخاتمة

يمثل الإصدار **1.0.13** من إضافة `Tss.Student` قفزة نوعية في قدرات الاستعلام عن بيانات الطلاب وأولياء الأمور. من خلال إعادة هيكلة دوال `getRecords`، وإضافة مساعدات متخصصة، ودمج `AccessManager` و `AdvancedQueryHelper`، أصبح النظام قادراً على تقديم واجهة موحدة وآمنة وعالية الأداء. جميع الدوال الجديدة تحافظ على التوافق العكسي، مما يضمن عملية ترقية سلسة للمشاريع القائمة. هذه التحسينات تفتح الباب لتطبيقات أكثر تفاعلية وتقارير متقدمة تعتمد على API نظيف وقوي.

---

**الوثائق المرجعية**:
- [التوثيق العام لإضافة Tss.Student](./docs/Student/Docs-Student-ar.md)
- [توثيق كلاس StudentHelper](./docs/Student/Docs-StudentHelper-Class-ar.md)
- [توثيق كلاس StudentRecordsHelper](./docs/Student/Docs-StudentRecordsHelper-Class-ar.md)
- [توثيق دوال AccessManager](./docs/AuthApi/Docs-AccessManager-ar.md)
- [توثيق AdvancedQueryHelper](./docs/querybuilder/Docs-AdvancedQueryHelper-ar.md)

## 2026-05-25 - 2026-05-26

**تحديث إضافة `Nano.AuthApi` – الإصدار 1.0.21**

### إضافة ميزات متقدمة إلى `AccessManager`: محلل ديناميكي للطلاب وأولياء الأمور، فلاتر متقدمة، تحميل مركزي للإعدادات، دوال مبسطة، ودعم الرجوع.

---

### ملخص التحديثات

يأتي الإصدار **1.0.21** من إضافة `Nano.AuthApi` بعدد من التحسينات الجوهرية على كلاس `AccessManager`، الذي أصبح الآن مسؤولاً ليس فقط عن إدارة الصلاحيات، بل أيضاً عن معالجة سيناريوهات متقدمة مثل:

- **المحلل الديناميكي للطلاب وأولياء الأمور (`resolveDynamicFrontendOptions`)** – يقوم تلقائياً بتعبئة `student_id` و `record_id` بناءً على نوع المستخدم وقواعد مرنة من `config.php`.
- **التحكم في الفلاتر المتقدمة (`advanced_filters`)** – يسمح بتحديد الحقول التي يمكن استخدام `is_or`, `is_not`, `is_or_null` عليها، مع قواعد حسب نوع المستخدم و `ref_type`.
- **تحميل مركزي لإعدادات الصلاحيات** – دوال `getOperationByConfig`, `checkByResource` تقرأ الإعدادات مباشرة من `config.php` دون الحاجة لبناء مصفوفات يدوية.
- **دعم الرجوع (Fallback)** – دوال `getOperationConfigWithFallback`, `checkWithFallback` تسمح بعمليات مثل `show` بالاستفادة من إعدادات `list`.
- **دوال مبسطة للوصول إلى الإعدادات** – `getOperationBySimpleKey`, `getAdvancedFiltersConfig` تسهل الاستخدام اليومي.

هذه التحسينات تجعل `AccessManager` أداة متكاملة لإدارة الصلاحيات، الفلاتر، ومعالجة متطلبات الطلاب وأولياء الأمور في أي API.

---

### أهداف الإصدار

- **تمكين تطبيقات المدارس** من التعامل بسهولة مع حالات الطلاب وأولياء الأمور عبر `frontend_resolver`.
- **توفير تحكم دقيق في الفلاتر** لضمان أمان البيانات ومنع تمرير فلاتر خطيرة من مستخدمين غير مصرح لهم.
- **مركزية إعدادات الصلاحيات** في ملف `config.php` لتقليل الكود المكرر وتسهيل الصيانة.
- **تبسيط استخدام `AccessManager`** عبر دوال مختصرة (مثل `checkByResource` بدلاً من تمرير 3 معاملات).
- **دعم الرجوع تلقائياً** لعمليات مثل `show` دون الحاجة إلى كتابة إعدادات كاملة.

---

### الميزات الجديدة والتحسينات

#### 1. المحلل الديناميكي للطلاب وأولياء الأمور (`resolveDynamicFrontendOptions`)

تمت إضافة مجموعة دوال للتعامل مع سيناريوهات الطلاب وأولياء الأمور:

- **`resolveDynamicFrontendOptions($user, $options, $query, $resourceKey)`** – دالة رئيسية تقوم بتحليل `ref_type` للمستخدم وتطبيق قواعد محددة في `frontend_resolver` من `config.php`.
- **`getFrontendResolverConfig($resourceKey)`** – تحميل إعدادات المحلل من ملف الإعدادات.
- **`ensureOptionsKeys($options, $keys)`** – التأكد من وجود مفاتيح معينة في مصفوفة الخيارات لتجنب أخطاء `undefined index`.

**مثال على إعدادات `frontend_resolver` في `config.php`:**

```php
'frontend_resolver' => [
    'enabled' => true,
    'apply_for_backend' => false,
    'apply_for_frontend' => true,
    'ref_type_rules' => [
        'student' => [
            'fields' => [
                'student_id' => ['source' => 'user', 'auto_fill' => true, 'force' => true],
                'record_id'  => ['source' => 'current_record', 'auto_fill_if_missing' => true, 'force' => true],
            ],
        ],
        'parent' => [
            'fields' => [
                'student_id' => ['source' => 'request', 'required' => true, 'validate_ownership' => true, 'force' => true],
                'record_id'  => ['source' => 'request_or_current', 'fallback_field' => 'student_id', 'force' => true],
            ],
        ],
    ],
],
```

عند استدعاء `resolveDynamicFrontendOptions`، سيقوم `AccessManager` تلقائياً بتعبئة `student_id` و `record_id` بناءً على هذه القواعد.

#### 2. التحكم في الفلاتر المتقدمة (`advanced_filters`)

تمت إضافة دوال للتحقق مما إذا كان المستخدم مسموحاً له باستخدام مفاتيح `is_or_*`, `is_not_*`, `is_or_null_*`:

- **`isAdvancedFilterAllowed($user, $field, $operationConfig)`** – تتحقق من السماح بناءً على `advanced_filters` في إعدادات العملية.
- **`filterAdvancedOptions($user, $options, $operationConfig)`** – تزيل أي مفاتيح غير مسموحة من `$options`.
- **`applyAdvancedFilterConstraints($user, $options, $operationConfig)`** – دالة شاملة (مرادف لـ `filterAdvancedOptions`).

**مثال على إعدادات `advanced_filters`:**

```php
'advanced_filters' => [
    'enabled' => true,
    'default_for_backend' => true,
    'default_for_frontend' => false,
    'field_specific_rules' => [
        'student_id' => [
            'backend' => true,
            'frontend' => ['*' => false, 'parent' => true],
        ],
    ],
],
```

#### 3. تحميل مركزي لإعدادات الصلاحيات من `config.php`

تمت إضافة دوال لقراءة الإعدادات وتطبيعها:

- **`getDefaultOperationConfig()`** – تعيد الإعدادات الافتراضية القياسية.
- **`normalizeOperationConfig($config)`** – تدمج الإعدادات المقدمة مع الافتراضية.
- **`arrayMergeRecursiveDistinct($array1, $array2)`** – دمج مصفوفات متكرر مع استبدال القيم وليس دمجها.
- **`getOperationConfig($plugin, $resource, $operation, $overrideConfig)`** – تجلب الإعدادات من `config` باستخدام ثلاثة معاملات.
- **`checkWithConfig($operation, $plugin, $resource, $user, $overrideConfig)`** – تجمع بين الجلب والتحقق.

#### 4. دوال مبسطة للوصول إلى الإعدادات (Single Key)

لتسهيل الاستخدام، تمت إضافة دوال تستقبل مفتاحاً واحداً بدلاً من ثلاثة:

- **`getOperationByConfig($resourceKey, $overrideConfig)`** – مفتاح مثل `'nano.absenceapi::absences.list'`.
- **`checkByResource($resourceKey, $user, $overrideConfig)`** – تستخرج العملية تلقائياً من آخر جزء من المفتاح.
- **`getAdvancedFiltersConfig($resourceKey, $overrideConfig)`** – تعيد فقط مصفوفة `advanced_filters` للمورد.
- **`getOperationBySimpleKey($simpleKey, $overrideConfig)`** – تختصر كتابة `'nano.absenceapi::'` (مثال: `'absences.list'`).

#### 5. دعم الرجوع (Fallback) بين العمليات

تساعد هذه الدوال في سيناريوهات مثل `show` التي يمكنها استخدام إعدادات `list` إذا لم تكن إعدادات `show` مكتملة:

- **`getOperationConfigWithFallback($resourceKey, $fallbackResourceKey, $overrideConfig)`**
- **`checkWithFallback($resourceKey, $fallbackResourceKey, $user, $overrideConfig)`**

إذا كان `is_allow` موجوداً في الإعدادات الأساسية، يُستخدم مباشرة. وإلا، يتم دمج الإعدادات الاحتياطية مع الأساسية.

#### 6. دوال مساعدة إضافية لسيناريوهات الطلاب / أولياء الأمور

- **`applyFrontendStudentParentFilters($query, $user, $options, $studentIdColumn, $recordIdColumn, $tablePrefix, $resolveIfNeeded)`** – تطبق الفلاتر المستنتجة مباشرة على الاستعلام.
- **`getStudentIdForFrontendUser($user, $options)`**, **`getRecordIdForFrontendUser($user, $options)`** – دوال مختصرة لجلب القيم المحلولة.
- **`validateParentStudentRelation($user, $studentId, $recordId)`** – تتحقق من أن الطالب يخص ولي الأمر.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. التحقق من صلاحية `list` باستخدام `checkByResource`

```php
$access = AccessManager::checkByResource('nano.absenceapi::absences.list', $user);
if (!$access['allowed']) {
    return $this->errorUnauthorized($access['message']);
}
```

#### 2. استخدام `checkWithFallback` لعملية `show`

```php
$access = AccessManager::checkWithFallback(
    'nano.absenceapi::absences.show',
    'nano.absenceapi::absences.list',
    $user
);
```

#### 3. تطبيق الفلاتر المتقدمة وإزالة غير المسموحة

```php
$advConfig = AccessManager::getAdvancedFiltersConfig('nano.absenceapi::absences.list');
$options = AccessManager::instance()->filterAdvancedOptions($user, $options, ['advanced_filters' => $advConfig]);
```

#### 4. تطبيق المحلل الديناميكي لحل `student_id` و `record_id`

```php
$options = AccessManager::instance()->resolveDynamicFrontendOptions(
    $user, $options, null, 'nano.absenceapi::absences.list'
);
// ثم في getRecords()، يمكن تطبيق الفلاتر المستنتجة
if (!empty($options['student_id']) && $options['is_force_student_id']) {
    $query->where('student_id', $options['student_id']);
}
```

#### 5. تطبيق `applyAccessScope` مع إمكانية تعطيل نطاقات

```php
$query = AccessManager::instance()->applyAccessScope($query, $access, [
    'company_field'    => 'companys_id',
    'department_field' => 'departments_id',
    // 'state_field' omitted -> no state filter
    'created_by_field' => 'created_by',
]);
```

---

### التوافق مع الإصدارات السابقة

- **جميع الدوال القديمة** (مثل `check`, `applyAccessScope`, `canAccessAllCompanies`) لا تزال تعمل كما هي.
- **الكلاس لم يفقد أي وظيفة** – تمت الإضافة فقط، لم يتم الحذف.
- **لا توجد هجرات قاعدة بيانات** مطلوبة.
- **يمكن الترقية مباشرة** دون تعديل الكود الحالي.

---

### متطلبات الترقية (من 1.0.20 إلى 1.0.21)

1. **تحديث الكود**:
   - استبدال ملف `AccessManager.php` بالنسخة الجديدة (الموجودة في المرفقات).
   - لا توجد تغييرات أخرى في الإضافة.

2. **تحديث ملف `version.yaml`**:
   - أضف الإصدار `1.0.21` كما هو موضح أعلاه.

3. **تحديث ملفات `config.php` لإضافات API** (اختياري):
   - يمكن إضافة أقسام `advanced_filters` و `frontend_resolver` إلى إعدادات الموارد للاستفادة من الميزات الجديدة.

4. **تنفيذ `php artisan plugin:refresh Nano.AuthApi`** (اختياري).

5. **اختبار الميزات الجديدة**:
   - جرب `checkByResource` بدلاً من `check` التقليدية.
   - جرب `checkWithFallback` لـ `show` endpoint.
   - جرب `filterAdvancedOptions` لتنظيف خيارات الطلب.
   - جرب `resolveDynamicFrontendOptions` في متحكم يستخدم الطلاب وأولياء الأمور.

---

### الفوائد والقيمة المضافة

- **تقليل الكود المكرر** في متحكمات API بنسبة تصل إلى 70%.
- **مركزية التحكم** في الصلاحيات والفلاتر، مما يسهل تغيير السلوك عبر ملفات الإعدادات فقط.
- **أمان أعلى** بفضل التحكم الدقيق في الفلاتر المتقدمة ومنع المستخدمين غير المخولين من استخدامها.
- **دعم السيناريوهات المدرسية** (طلاب وأولياء أمور) بشكل طبيعي ومباشر.
- **سهولة الصيانة** حيث أن أي تعديل في منطق الصلاحيات يتم في مكان واحد.

---

### الخاتمة

الإصدار **1.0.21** من `Nano.AuthApi` يضع `AccessManager` كأداة شاملة ومتقدمة لإدارة الصلاحيات والفلاتر ومعالجة العلاقات بين المستخدمين والكيانات (مثل الطلاب وأولياء الأمور). بفضل الدوال الجديدة، أصبح بناء API آمن ومرن أسهل بكثير، ويتماشى تماماً مع دليل `Nano-Api-SKILL.md`.

---

**الوثائق المرجعية**:
- [توثيق كلاس `AccessManager`](./docs/AuthApi/Docs-AccessManager-ar.md)
-
- [توثيق الصلاحيات `AccessManager-Permission`](./docs/AuthApi/Docs-AccessManager-Permission-ar.md)
- [امثلة عامة `AccessManager-Example`](./docs/AuthApi/Docs-AccessManager-Example-ar.md)
- [مثال عملى شامل  `AccessManager-Example-Api`](./docs/AuthApi/Docs-AccessManager-Example-Api-ar.md)
- [توثيق كلاس `AuthHelpers`](./docs/AuthApi/Docs-AuthHelpers-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL)](./docs/mcp/Nano-Api-SKILL.md
- [تحديث إضافة `Nano.AbsenceApi` (نموذج تطبيقي)](./docs/AbsenceApi/Update-AbsenceApi-v1.1.0-ar.md)

## 2026-05-26 - 2026-05-27

**تحديث إضافة `Nano.AbsenceApi` – الإصدار 1.1.0**

### إعادة هيكلة كاملة للمتحكمات وفقاً لدليل `Nano-Api-SKILL.md` وتكامل `AccessManager`

---

### ملخص التحديثات

يقدم الإصدار **1.1.0** من إضافة `Nano.AbsenceApi` نقلة نوعية في جودة الكود وأمانه وتوافقه مع معايير Nano API الحديثة. تم إعادة كتابة جميع المتحكمات (`Absences`, `Files`, `ClassTypes`, `ClassTypeCompanies`) بالكامل لتصبح متوافقة مع دليل `Nano-Api-SKILL.md`، مع الاستفادة الكاملة من `AccessManager` للتحكم المركزي بالصلاحيات، `AdvancedQueryHelper` للتصفية الديناميكية، ونظام تخزين مؤقت وأحداث موحد.

**أبرز التغييرات:**
- توحيد هيكل جميع المتحكمات (Index, Show, Store, Update, getRecords).
- استخدام `AccessManager::checkByResource` و `checkWithFallback` للتحقق من الصلاحيات بناءً على إعدادات `config.php`.
- إضافة دعم الفلاتر المتقدمة (`is_or`, `is_not`, `is_force`, `is_or_null`) مع تكوين مرن لكل نوع مستخدم.
- توحيد دالة `getRecords` لدعم خيارات الإخراج المتعددة (`is_query`, `is_first`, `is_collection`, `is_paginator`، إلخ) والأحداث (`extendQueryBefore`, `extendQuery`) والتخزين المؤقت.
- إضافة `frontend_resolver` الديناميكي لحل `student_id` و `record_id` تلقائياً لمستخدمي الطلاب وأولياء الأمور.
- تطبيق `applyAccessScope` مع إمكانية تعطيل نطاقات محددة.
- تحسين الأمان عبر دمج صلاحيات `show` مع `list` باستخدام `checkWithFallback`.
- توسيع ملف `config.php` ليشمل إعدادات `permission`, `advanced_filters`, `frontend_resolver` لكل مورد.

---

### أهداف الإصدار

- **توحيد معايير التطوير** بين جميع متحكمات الـ API في النظام البيئي Nano.
- **تبسيط إدارة الصلاحيات** عبر مركزية الإعدادات في `config.php` واستخدام دوال `AccessManager`.
- **تحسين الأمان** من خلال الفلاتر المتقدمة المقيدة والتحقق من الصلاحيات لكل عملية.
- **زيادة المرونة** عبر دعم خيارات إخراج متعددة وأحداث قابلة للتوسع.
- **تسهيل الصيانة** بإعادة هيكلة الكود بحيث يكون متسقاً وقابلاً للقراءة.

---

### الميزات الجديدة والتحسينات

#### 1. توحيد الصلاحيات باستخدام `AccessManager`

- **قبل الإصدار 1.1.0**: كان كل متحكم يبني مصفوفة الصلاحيات يدوياً داخل الكود، مما أدى إلى تكرار المنطق وصعوبة الصيانة.
- **بعد التحديث**: يتم تعريف جميع إعدادات الصلاحيات في `config.php` لكل مورد وعملية (`list`, `show`, `create`, `update`). يتم تحميلها تلقائياً عبر `AccessManager::checkByResource`.

**مثال من `Absences`:**

```php
$access = AccessManager::checkByResource('nano.absenceapi::absences.list', $user);
```

#### 2. دعم الفلاتر المتقدمة (`advanced_filters`)

تمت إضافة مصفوفة `advanced_filters` داخل إعدادات كل عملية، مما يسمح بالتحكم في:

- الحقول التي يمكن استخدام `is_or`, `is_not`, `is_or_null` عليها.
- قيم افتراضية لكل نوع مستخدم (backend/frontend).
- قواعد خاصة بكل حقل وحتى بكل `ref_type` لمستخدمي frontend.

**مثال من `config.php`:**

```php
'advanced_filters' => [
    'field_specific_rules' => [
        'student_id' => [
            'frontend' => [
                '*' => false,
                'parent' => true,
            ],
        ],
    ],
],
```

#### 3. المحلل الديناميكي للطلاب وأولياء الأمور (`frontend_resolver`)

تم إضافة مقطع `frontend_resolver` الذي يقوم تلقائياً بـ:

- تحديد ما إذا كان المستخدم طالباً (ref_type = student) ثم تعيين `student_id` من المستخدم وجلب `record_id` الحالي إذا لزم الأمر.
- لولي الأمر، التأكد من توفير `student_id` أو `record_id`، والتحقق من ملكية الطالب، ثم ملء القيم المفقودة.

**مثال التطبيق في المتحكم:**

```php
$options = AccessManager::instance()->resolveDynamicFrontendOptions(
    $user, $options, null, 'nano.absenceapi::absences.list'
);
```

#### 4. توحيد دالة `getRecords`

أصبحت دالة `getRecords` متطابقة في جميع المتحكمات وتدعم:

- **فلاتر متقدمة:** عبر `AdvancedQueryManager::scopeWhereField` لكل حقل مع خيارات `is_or`, `is_not`, `is_force`, `is_or_null`.
- **أحداث:** `api.list.extendQueryBefore` و `api.list.extendQuery` لتمكين التوسع الخارجي.
- **تخزين مؤقت:** اختياري عبر `cached()`.
- **خيارات إخراج متعددة:** `is_query`, `is_first`, `is_model`, `is_collection`, `is_paginator`, `is_to_sql`.
- **`with_count`, `group_by`, `having`, `take_limet`, `withTrashed`, `onlyTrashed`.**

#### 5. تطبيق النطاق (`applyAccessScope`) مع إمكانية التعطيل

أصبح من الممكن تطبيق نطاقات الشركة والقسم والولاية ومنشئ السجل تلقائياً، مع إمكانية تعطيل نطاقات معينة ببساطة عن طريق حذف المفتاح المقابل:

```php
$query = AccessManager::instance()->applyAccessScope($query, $access, [
    'company_field'    => 'companys_id',
    'department_field' => 'departments_id',
    // 'state_field' omitted -> no state filter
    'created_by_field' => 'created_by',
]);
```

#### 6. تحسين دالة `show` باستخدام `checkWithFallback`

لتجنب تكرار إعدادات الصلاحيات، تستخدم `show` الآن `checkWithFallback` لاستخدام إعدادات `list` إذا لم توجد إعدادات خاصة بـ `show`:

```php
$access = AccessManager::checkWithFallback(
    'nano.absenceapi::absences.show',
    'nano.absenceapi::absences.list',
    $user
);
```

#### 7. تحديث كامل للمتحكمات الأخرى

- **`Files`**: أضيفت لها نفس بنية `getRecords` مع دعم فلاتر `companys_id`, `departments_id`, `ref_type_class`، وتطبيق `applyAccessScope` للشركة والقسم.
- **`ClassTypes`**: تم تحديثه ليدعم الفلاتر الأساسية (`ref_type`, `is_active`) وخيارات الإخراج المتعددة.
- **`ClassTypeCompanies`**: تم تحديثه ليدعم فلاتر `companys_id`, `departments_id`, `ref_type`، وتطبيق `applyAccessScope` على الشركة والقسم.

---

### أمثلة عملية على الاستخدام الجديد

#### 1. جلب قائمة الغيابات لولي أمر مع فلترة متقدمة

```bash
curl -X GET "https://yourdomain.com/api/v1/absence/absences?student_id=100&is_or_student_id=false&absences_status=absent&include=student"
```

#### 2. إنشاء سجل غياب جديد عبر API

```bash
curl -X POST "https://yourdomain.com/api/v1/absence/absences" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"student_id": 100, "absences_status": "absent", "date_at": "2026-05-25"}'
```

#### 3. استخدام `is_to_sql` لتصحيح الأخطاء

```php
$result = StudentHelper::getStudentRecordRecords([
    'year_id' => 3,
    'is_to_sql' => true,
]);
// ستتم طباعة استعلام SQL في trace_log
```

---

### توافق الإصدارات السابقة

- **جميع نقاط النهاية الحالية** (routes) لم تتغير، لذا لن يتأثر أي تطبيق يستهلك الـ API.
- **المخرجات** لا تزال متوافقة مع الهيكل السابق (مع إضافة `input_data`, `process_data`, `debug` فقط عند الحاجة).
- **لا توجد هجرات قاعدة بيانات** جديدة.
- **التكوين الجديد** اختياري: يمكنك الاستمرار في استخدام الإعدادات القديمة (دون `advanced_filters` أو `frontend_resolver`) وستعمل الإضافة بالسلوك الافتراضي.

---

### متطلبات الترقية (من 1.0.1 إلى 1.1.0)

1. **تحديث الكود**:
   - استبدال جميع ملفات المتحكمات (`Absences.php`, `Files.php`, `ClassTypes.php`, `ClassTypeCompanies.php`) بالنسخ الجديدة.
   - استبدال ملف `config.php` بالإصدار الجديد الذي يحتوي على أقسام `permission`, `advanced_filters`, `frontend_resolver`.

2. **تحديث ملف `version.yaml`**:
   - أضف الإصدار `1.1.0` كما هو موضح في بداية هذا المستند.

3. **تنفيذ الهجرات** (لا توجد).

4. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

5. **اختبار الوظائف**:
   - تحقق من أن جميع نقاط النهاية (`absences`, `files`, `class-types`, `class-type-companies`) تعمل كما هو متوقع.
   - اختبر صلاحيات وصول مختلفة (backend, frontend student, frontend parent, guest).
   - تحقق من أن الفلاتر المتقدمة (`is_or`, `is_not`) تعمل فقط للحقول والأنواع المسموحة.

---

### الفوائد والقيمة المضافة

- **أمان محسّن**: يتم تطبيق نطاقات الصلاحيات تلقائياً ولا يمكن تجاوزها بسهولة.
- **مرونة عالية**: يمكن تكوين سلوك كل عملية بشكل مستقل دون تعديل الكود.
- **قابلية التوسع**: بفضل الأحداث، يمكن للإضافات الأخرى تعديل الاستعلامات ديناميكياً.
- **أداء أفضل**: مع دعم التخزين المؤقت واستبعاد الأعمدة غير الضرورية.
- **تجربة مطور أفضل**: جميع المتحكمات تتبع نفس النمط، مما يسهل تعلم وصيانة الكود.

---

### الخاتمة

يمثل الإصدار **1.1.0** من `Nano.AbsenceApi` إنجازاً كبيراً نحو توحيد معايير API في نظام Nano البيئي. بفضل إعادة الهيكلة الكاملة وتكامل `AccessManager`، أصبحت الإضافة أكثر أماناً وقابلية للصيانة والتوسع. جميع المتحكمات الآن متوافقة مع دليل `Nano-Api-SKILL.md` وتقدم ميزات متقدمة مثل الفلاتر المتقدمة والمحلل الديناميكي والأحداث.

نوصي جميع المطورين بالترقية إلى هذا الإصدار والاستفادة من الإعدادات المركزية في `config.php` لتخصيص سلوك API حسب احتياجاتهم.

---

**الوثائق المرجعية**:
- [توثيق إضافة `Nano.StudyyearApi`](./docs/StudyyearApi/Docs-StudyyearApi-ar.md)
- [توثيق إضافة `Nano.SchoolApi`](./docs/SchoolApi/Docs-SchoolApi-ar.md)
- [توثيق إضافة `Nano.StudentsApi`](./docs/StudentsApi/Docs-StudentsApi-ar.md)
- [توثيق إضافة `Nano.AbsenceApi`](./docs/AbsenceApi/Docs-AbsenceApi-ar.md)
- [دليل تطوير إضافات API (Nano-Api-SKILL.md)](./Nano-Api-SKILL.md)
- [توثيق كلاس `AccessManager`](../AuthApi/Docs-AccessManager-ar.md)
- [توثيق كلاس `AdvancedQueryHelper`](../../QueryBuilder/Docs-AdvancedQueryHelper.md)
- [توثيق كلاس `StudentRecordsHelper` (Tss.Student)](../../Student/Helpers/Docs-StudentRecordsHelper-Class-ar.md)

## 2026-05-27 - 2026-05-28

**تحديث إضافة `Tss.Homework` – الإصدار 1.0.9**

### إضافة وحدة `Feedback` (التسليمات والتقييمات) بشكل كامل

---

### ملخص التحديثات

يقدم الإصدار **1.0.9** من إضافة `Tss.Homework` وحدة **`Feedback`** المتكاملة، والتي كانت موجودة هيكلياً منذ الإصدار 1.0.7 (جدول `tss_homework_feedbacks`) ولكنها أصبحت الآن مدعومة بالكامل في لوحة التحكم:

- **نموذج (`Feedback`)** مع جميع العلاقات والنطاقات (Scopes) والأحداث.
- **واجهة إدارة متكاملة** (Controller, Model, Config) ضمن `Backend`.
- **قائمة (`columns.yaml`)** لعرض التسليمات في جدول.
- **نموذج إدخال (`fields.yaml`)** متكامل مع تبويبات ومكونات (منسدلات، محرر، رفع ملفات).
- **صلاحيات مخصصة** (`permissions`) لإدارة التسليمات.
- **قائمة جانبية** (`menus`) تحت قائمة `homework` الرئيسية.
- **ترجمة كاملة** (`lang/ar/lang.php` و `lang/en/lang.php`) لكل العناصر.
- **علاقات مع الواجبات (`homeworks`)** والطلاب والصفوف والمواد والمعلمين.

يأتي هذا التحديث لتكملة نظام الواجبات المنزلية، حيث يسمح للمعلمين بتسجيل تسليمات الطلاب وتقييمهم، وللمشرفين بمتابعة الأداء، ولاستخدامها لاحقاً عبر الـ API.

---

### أهداف الإصدار

- **إكمال منظومة الواجبات المنزلية** بربط التسليمات والتقييمات بالواجبات.
- **تمكين المعلمين من رصد درجات الطلاب** لكل واجب عبر واجهة خلفية سهلة.
- **توفير تقارير أساسية** عن حالة التسليم (مستلم، درجة، تقييم، ملاحظات).
- **دعم العلاقات المتشابكة** مع الصفوف والمجموعات والمواد والسجلات الدراسية.
- **التحضير لاستخدام الـ API** (`Nano.HomeworkApi`) الذي يعتمد على هذا النموذج.

---

### الميزات الجديدة والتحسينات

#### 1. نموذج `Feedback` كامل المواصفات (`models/Feedback.php`)

تم إنشاء النموذج وفقاً لأفضل ممارسات OctoberCMS:

- **Table**: `tss_homework_feedbacks` (موجود منذ 1.0.7).
- **Traits**: `Validation`, `SoftDelete`, `Sortable`.
- **Behaviors**: `TranslatableModel`, `BasicModel`, `BasicScope`.
- **حقوق JSONable**: `keywords`, `config_data`, `properties`, `other_data`, `ref_key_data`.
- **علاقات `belongsTo`**: مع `Homework`, `Student`, `StudentRecord`, `ClassTypeCompany`, `TbClass`, `Group`, `Subject`, `Employee` (teacher, educator), وغيرها.
- **علاقات الملفات**: `attachOne` (image), `attachMany` (images, files).
- **نطاقات مخصصة**: `scopeRefTypeClass`, `scopeStatusHomework`, `scopeEvaluation`, `scopeIsRecive`.
- **أحداث قبل/بعد الحفظ**: إنشاء تلقائي للـ `code`، تعيين `ref_key_name_class` و `ref_key_values_class` بناءً على `ref_type_class`، تعيين `published_at` افتراضياً.

#### 2. واجهة الإدارة المتكاملة

##### **قائمة التسليمات (`feedback/columns.yaml`)**
- أعمدة أساسية: `id`, `code`, `name`, `homeworks_id` (مع رابط للواجب), `student_name`, `class`, `subject`, `evaluation`, `status_homework`, `degree`, `is_recive`, `is_active`, `created_at`.
- دعم البحث والفرز والترشيح.

##### **نموذج الإدخال (`feedback/fields.yaml`)**
- **تبويبات**:
  - `basic`: البيانات الأساسية (الواجب، الطالب، السجل الدراسي، الصف، الشعبة، المادة، المعلم، الترم، الشهر، إلخ).
  - `evaluation`: التقييم، الحالة، درجة، نسبة الإنجاز، استلام.
  - `notes`: ملاحظات المعلم، المشرف، ولي الأمر، والملاحظة التلقائية.
  - `details`: وصف قصير، وصف منسق، الحل (solation).
  - `media`: صورة رئيسية، معرض صور، ملفات.
  - `seo`: عناوين ووصف SEO.
  - `properties`: خصائص إضافية (Repeater).
  - `options`: خيارات النشر، الحالة، الترتيب.
- **حقول منسدلة** تعتمد على بعضها (dependsOn) مثل: `class_id`, `group_id`, `student_id`, `subject_id`, `teacher_id`.
- **مكونات**: `fileupload`, `richeditor`, `taglist`, `datepicker`, `repeater`.

#### 3. صلاحيات جديدة (`permissions`)

تم إضافة قسم `feedbacks` في `registerPermissions()`:

```php
'feedbacks' => [
    'title' => 'صلاحيات التسليمات والتقييمات',
    'access_all' => 'صلاحية الوصول إلى كافة التسليمات',
    'access' => 'صلاحية الوصول إلى التسليمات',
    'add' => 'صلاحية إضافة تسليمة',
    'edit' => 'صلاحية تعديل التسليمة',
    'delete' => 'صلاحية حذف التسليمة',
],
```

#### 4. قائمة جانبية (`menus`)

أضيف عنصر `feedbacks` كقائمة فرعية تحت `homework`:

```php
'feedbacks' => [
    'label' => 'التسليمات والتقييمات',
    'url' => Backend::url('tss/homework/feedbacks'),
    'icon' => 'icon-check-square-o',
    'permissions' => ['tss.homework.feedbacks.*'],
    'order' => 4,
],
```

#### 5. الترجمة الكاملة (`lang/ar/lang.php` و `lang/en/lang.php`)

تم إضافة ترجمات لكل العناصر:
- `feedbacks` → `tab`, `form`, `list`, `filter`, `pages`.
- قيم الخيارات: `evaluation_options`, `status_homework_options`, `option_month_num`, إلخ.
- رسائل الأخطاء والنجاح (للإصدارات القادمة قد تستخدمها API).

#### 6. دعم جلب البيانات عبر `getStudentIdOptions`, `getClassIdOptions`, إلخ

تم تنفيذ دوال مساعدة في النموذج لتوليد قوائم منسدلة ديناميكية مع مراعاة:
- نطاق الشركة (`IsCompany`).
- الاعتماد على الخيارات المحددة مسبقاً (مثل `departments_id`، `class_id`، `year_id`).
- إظهار البيانات بشكل هرمي (للصفوف والفروع).

#### 7. تكامل مع `Tss\Homework\Traits\Homeworkble`

تم استخدام التريت `Homeworkble` الذي يضيف نطاقاً عالمياً `scopeHomeworkble` لتصفية السجلات حسب `ref_type_class` (في حالة `Feedback`، يمكن استخدامه لتمييز الأنواع). ولكن بحسب التصميم الحالي، `Feedback` لا يستخدم `Homeworkble` افتراضياً لأن `feedback` ليس نوعاً من وسائط `homeworkble`، لكنه ترك للاستخدام المستقبلي.

---

### أمثلة على الاستخدام في لوحة التحكم

#### 1. إضافة تسليمة جديدة

- اذهب إلى `الواجبات > التسليمات والتقييمات`.
- اضغط **"إضافة جديد"**.
- اختر **الواجب** من القائمة المنبثقة (يعتمد على القسم والفرع ونوع التصنيف).
- اختر **الطالب** (يتم تحديث القائمة تلقائياً حسب الصف والشعبة والسنة).
- أدخل **الدرجة** (مثلاً `9.5`).
- اختر **التقييم** (`ممتاز`, `جيد`, `متوسط`, `ضعيف`).
- اختر **حالة التسليم** (`ناجح`, `راسب`).
- ارفع ملفات الحل إن وجدت.
- احفظ.

#### 2. تحديث درجة تسليمة من القائمة

- من قائمة التسليمات، انقر على الدرجة المراد تعديلها مباشرة (إذا كان العمود قابل للتعديل) أو افتح النموذج.
- غير القيمة واحفظ.

#### 3. تصفية التسليمات حسب الواجب

- استخدم فلتر `homeworks_id` في أعلى القائمة لعرض تسليمات واجب معين فقط.

---

### التغييرات التقنية التفصيلية

#### ملفات جديدة

| المسار | الوصف |
|--------|-------|
| `models/Feedback.php` | نموذج `Feedback` الكامل. |
| `models/feedback/columns.yaml` | تعريف أعمدة القائمة. |
| `models/feedback/fields.yaml` | تعريف حقول النموذج. |
| `controllers/Feedbacks.php` | متحكم `Backend` لإدارة التسليمات. |
| `controllers/feedbacks/config_list.yaml` | تكوين القائمة. |
| `controllers/feedbacks/config_form.yaml` | تكوين النموذج. |
| `controllers/feedbacks/config_reorder.yaml` | تكوين إعادة الترتيب (اختياري). |
| `lang/ar/lang.php` | تحديث بإضافة قسم `feedbacks`. |
| `lang/en/lang.php` | تحديث بإضافة قسم `feedbacks`. |

#### تعديلات على الملفات الموجودة

- **`Plugin.php`**:
  - أضيف `registerPermissions()` لقسم `feedbacks`.
  - أضيف `bootBackendNavigation()` لإضافة عنصر القائمة الجانبية.
- **`version.yaml`**:
  - أضيف الإصدار `1.0.9` مع وصف `Support Feedbacks`.

#### لا توجد تغييرات في قاعدة البيانات

- جدول `tss_homework_feedbacks` موجود بالفعل منذ الإصدار 1.0.7، لذا لا يلزم تشغيل هجرات إضافية.

---

### متطلبات الترقية (من 1.0.8 إلى 1.0.9)

1. **تحديث الكود**: استبدال جميع الملفات المذكورة أعلاه.
2. **تحديث `version.yaml`**:
   ```yaml
   1.0.9:
     - 'Support Feedbacks'
   ```
3. **تنفيذ هجرات**:
   ```bash
   php artisan plugin:refresh Tss.Homework
   ```
   (سيتم تحديث الإصدار فقط، لا توجد هجرات جديدة).
4. **إعادة تعيين الأذونات** (اختياري): يمكنك تعيين الصلاحيات الجديدة للمجموعات المناسبة في `الإعدادات > إدارة المستخدمين > مجموعات`.
5. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```
6. **التحقق من القائمة الجانبية**: يجب أن يظهر عنصر `التسليمات والتقييمات` تحت قائمة `الواجبات`.

---

### الفوائد والقيمة المضافة

- **متكامل مع نظام الواجبات**: يربط كل تسليمة بواجب محدد.
- **سهولة إدخال البيانات**: حقول منسدلة ذكية تعتمد على بعضها تقلل الأخطاء.
- **تقارير فورية**: يمكن للمعلم والمشرف الاطلاع على حالة كل طالب.
- **جاهز للـ API**: النموذج جاهز لاستخدامه من قبل `Nano.HomeworkApi` (متحكم `Feedbacks`).
- **دعم متعدد اللغات**: جاهز لترجمة الواجهة حسب لغة المستخدم.
- **مرونة التوسع**: يمكن إضافة حقول أو علاقات جديدة مستقبلاً.

---

### الخاتمة

يعد الإصدار **1.0.9** من إضافة `Tss.Homework` إضافة مهمة تكمل منظومة الواجبات المنزلية، حيث يوفر واجهة إدارة متكاملة للتسليمات والتقييمات. مع هذا التحديث، أصبح لدى المدارس والمراكز التعليمية أداة قوية لرصد أداء الطلاب في الواجبات اليومية والشهرية، مع إمكانية التوسع لاحقاً باستخدام الـ API المخصص (`Nano.HomeworkApi`).

نوصي جميع مستخدمي الإضافة بالترقية للاستفادة من هذه الميزات، مع التأكد من تعيين الصلاحيات المناسبة للمعلمين والمشرفين.

---

**الوثائق المرتبطة**:
- [دليل مستخدم `Tss.Homework` (قيد الإعداد)](./docs/User-Guide-ar.md)
- [توثيق `Nano.HomeworkApi`](./docs/HomeworkApi/Docs-HomeworkApi-ar.md)
- [تحديث `Nano.HomeworkApi` الإصدار 1.0.0](./docs/HomeworkApi/Update-HomeworkApi-v1.0.1-ar.md)

