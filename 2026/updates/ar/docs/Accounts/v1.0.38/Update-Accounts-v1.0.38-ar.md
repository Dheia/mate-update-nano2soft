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
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
