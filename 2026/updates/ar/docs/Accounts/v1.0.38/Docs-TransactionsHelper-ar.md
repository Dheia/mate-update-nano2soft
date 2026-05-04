# توثيق كلاس `TransactionsHelper` وسمة `TransactionsMultiHelper` – مساعد العمليات المحاسبية

## نظرة عامة

يقع كلاس `TransactionsHelper` في المسار `Tss\Accounts\Helpers\TransactionsHelper`، وهو المسؤول الرئيسي عن تنفيذ كافة العمليات المحاسبية المتعلقة بالقيود والحركات المالية في إضافة `Tss.Accounts`. يعمل الكلاس كطبقة خدمات (Service Layer) شاملة لإنشاء القيود المحاسبية بأنواعها المختلفة (بسيطة ومتعددة الأطراف)، والتحقق من الأرصدة، وإدارة المدفوعات، وتسوية فروق العملات، وغيرها من العمليات المتقدمة.

منذ الإصدار **1.0.38**، تم تعزيز الكلاس بشكل كبير عبر دمج سمة `TransactionsMultiHelper` التي أضافت إمكانيات إنشاء القيود متعددة الأطراف (`createJournalEntry`) مع نظام تحقق متكامل، ودعم فروق العملات، وفحص أنواع الحسابات، والتحكم في الحقول الإضافية لكل جانب من جوانب القيد.

يتميز الكلاس بأن جميع دواله ثابتة (`static`)، مما يسمح باستدعائها مباشرة دون إنشاء كائن. كما يلتزم بإرجاع مصفوفة استجابة موحدة (`result array`) في معظم الدوال، مما يسهل التعامل مع النجاح والفشل بشكل متسق.

---

## هيكل الاستجابة الموحد (Result Array)

الدوال التي تنفذ عمليات كتابة (إنشاء قيود، إضافة دفعات، إلخ) تعيد مصفوفة بالهيكل التالي:

| المفتاح | النوع | الوصف |
| :--- | :--- | :--- |
| `code` | `int` | كود الحالة (`200` للنجاح، `400` للخطأ). |
| `status` | `bool` | حالة العملية (`true` للنجاح، `false` للفشل). |
| `message` | `string` | رسالة وصفية قابلة للترجمة. |
| `error` | `string\|null` | رسالة الخطأ (إن وجدت). |
| `errors` | `array\|null` | مصفوفة أخطاء التحقق (إن وجدت). |
| `model` | `TransactionHeader\|null` | كائن رأس القيد الذي تم إنشاؤه. |
| `data` | `mixed\|null` | بيانات إضافية (حسب السياق). |
| `input_data` | `array` | المعاملات المدخلة بعد المعالجة الأولية. |
| `process_data` | `array` | البيانات المستخدمة فعلياً أثناء تنفيذ العملية. |
| `debug` | `array\|null` | معلومات تصحيح (تظهر فقط في بيئة التطوير عند `app.debug = true`). |

---

## 1. دوال إنشاء القيود الأساسية

### 1.1 `createSingle`

تنشئ قيداً محاسبياً بسيطاً ذا طرفين (مدين ودائن). هذه هي الدالة الأساسية التي تعتمد عليها معظم العمليات المالية مثل إضافة الرصيد الافتتاحي، شحن الرصيد، والمدفوعات.

```php
public static function createSingle($options = []): array
```

#### الخيارات الأساسية

| المفتاح | النوع | الافتراضي | الوصف |
| :--- | :--- | :--- | :--- |
| `companys_id` | `int` | (يُحسب تلقائياً) | معرف الشركة |
| `departments_id` | `int` | (يُحسب تلقائياً) | معرف القسم |
| `cost_centers_id` | `int` | (يُحسب تلقائياً) | معرف مركز التكلفة |
| `periods_id` | `int` | (يُحسب تلقائياً) | معرف الفترة المحاسبية |
| `date_at` | `Carbon/string` | `now()` | تاريخ القيد |
| `transactions_type` | `int` | `4` | نوع الحركة |
| `process_type` | `int/string` | `4` | نوع العملية |
| `patterns_id` | `int` | `4` | معرف النمط |
| `notes` | `string` | `null` | ملاحظات عامة |
| `type_header` | `string` | `null` | نوع رأس القيد (مثل `BondsDay::class`) |
| `mony` | `float` | `0` | **المبلغ (إجباري)** |
| `debit_accounts_id` | `string` | **إجباري** | كود حساب المدين |
| `credit_accounts_id` | `string` | **إجباري** | كود حساب الدائن |
| `debit_person` | `object` | `null` | كائن الشخص المدين |
| `debit_person_id` / `debit_person_type` | `int/string` | `null` | معرف ونوع الشخص المدين |
| `credit_person` | `object` | `null` | كائن الشخص الدائن |
| `credit_person_id` / `credit_person_type` | `int/string` | `null` | معرف ونوع الشخص الدائن |
| `debit_notes` | `string` | `null` | ملاحظات الجانب المدين |
| `credit_notes` | `string` | `null` | ملاحظات الجانب الدائن |
| `default_notes` | `string` | `null` | ملاحظة افتراضية للرأس (إذا لم تُقدم `notes`) |

بالإضافة إلى العديد من الخيارات الأخرى للتحكم في الحقول الإضافية (`extend_id`, `modul_type`, `ref_type`...)، والعملات (`currencys_id`, `rate`) والحالة (`status`, `is_active`).

#### مثال

```php
$result = TransactionsHelper::createSingle([
    'mony'               => 1500,
    'debit_accounts_id'  => '2-2-1253010001', // الخزينة
    'credit_accounts_id' => '2-2-1231010001', // العميل
    'notes'              => 'إيداع نقدي',
]);

if ($result['status']) {
    echo "تم إنشاء القيد بنجاح، رقم الحركة: " . $result['model']->id;
}
```

---

### 1.2 `createEntry` (من `TransactionsMultiHelper`)

دالة موحدة لإنشاء القيود، تختار تلقائياً بين `createSingle` (إذا كان عدد الأطراف ≤ 2) و `createJournalEntry` (إذا كان عدد الأطراف > 2). يمكن فرض استخدام `createJournalEntry` عبر الخيار `forceJournal => true`.

```php
public static function createEntry($options = [], $is_test_create = false): array
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$options` | `array` | خيارات القيد، يجب أن تحتوي على `details` إذا كان متعدد الأطراف. |
| `$is_test_create` | `bool` | إذا كان `true`، يتم التراجع عن العملية (لأغراض الاختبار). |

#### مثال

```php
// قيد متعدد الأطراف (3 أطراف)
$result = TransactionsHelper::createEntry([
    'date_at' => '2026-05-18',
    'notes'   => 'قيد تسوية',
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 5000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 3000],
        ['accounts_id' => '2-2-4111010001', 'credit' => 2000],
    ],
]);
```

---

### 1.3 `createJournalEntry` (من `TransactionsMultiHelper`)

إنشاء قيد محاسبي متعدد الأطراف (أكثر من طرفين) مع تحقق متقدم ومعالجة للعملات وفروق الصرف.

```php
public static function createJournalEntry($options = [], $is_test_create = false): array
```

تدعم هذه الدالة **مجموعة هائلة من الخيارات** للتحكم في كل جانب من جوانب القيد. فيما يلي ملخص بأهم المجموعات، والقائمة الكاملة موجودة في ملف [createJournalEntry-config.md](./createJournalEntry-config.md).

#### أبرز مجموعات الخيارات

- **حقول الرأس الأساسية**: `companys_id`, `departments_id`, `date_at`, `transactions_type`, `process_type`, `patterns_id`, `notes`, `type_header`...
- **الشخص على مستوى الرأس**: `header_person_id`, `header_person_type`, `header_person`.
- **قيود عدد الأطراف**: `max_debit_entries`, `max_credit_entries`, `allow_multiple_debit`, `allow_multiple_credit`.
- **التحكم في الشخص لكل جانب**: `debit_person_allowed`, `debit_person_required`, `debit_person_allowed_types` (ونظائرها للدائن).
- **التحكم في الحساب لكل جانب**: `debit_account_allowed`, `debit_account_required`, `debit_default_account`.
- **التحكم في الملاحظات لكل جانب**: `debit_notes_allowed`, `debit_notes_required`, `debit_default_notes`.
- **حدود المبالغ**: إجمالي (`min_total_amount`, `max_total_amount`) وحسب الجانب (`min_debit_amount`, `max_credit_amount`...).
- **فحص الرصيد**: `debit_check_balance` مع دعم دوال callback مخصصة (`debit_balance_check_callback`).
- **فحص نوع الحساب بقواعد ديناميكية**: `debit_check_account_types` و `debit_rules_account_types`.
- **التحكم في الحقول الإضافية**: `header_extend_id_allowed`, `debit_batch_id_required`... (تغطي 7 حقول).
- **العملات**: `auto_convert_currency`, `handle_exchange_differences`, `exchange_difference_action`.
- **إدارة `paper_id`**: `is_paper_id`, `paper_id_allowed`, `paper_id_auto`, `paper_id_check_duplicate`...
- **أخرى**: `disabled_rules`, `beforeValidate`, `afterValidate`.

#### مثال متقدم

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 2500],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1500],
        ['accounts_id' => '2-2-4111010001', 'credit' => 1000],
    ],
    'min_total_amount'       => 1000,
    'max_total_amount'       => 10000,
    'debit_check_account_types' => true,
    'debit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Boxe']
    ],
    'handle_exchange_differences' => true,
]);
```

---

## 2. دوال العمليات المالية الخاصة

### 2.1 `createMonyDeliverys`

إضافة رصيد افتتاحي لموصل (Delivery) من الشركة. تستخدم في أنظمة التوصيل عند تفعيل حسابات الموصلين.

```php
public static function createMonyDeliverys($user, $mony = null): array
```

- `$user`: كائن المستخدم (أو كائن `Delivery`).
- `$mony`: المبلغ (إذا لم يُحدد، يُقرأ من الإعدادات `tss.accounts::create_mony_deliverys.mony`).

يتم جلب إعدادات الحسابات المدينة والدائنة ونوع الحركة من ملف الإعدادات (`tss.accounts::create_mony_deliverys`). تتحقق الدالة من عدم وجود رصيد افتتاحي سابق للموصل.

---

### 2.2 `createMonyDeliverysFromSyndical`

شحن رصيد لموصل من قبل وكيل (Syndical/Department). تتحقق من رصيد الوكيل قبل الخصم.

```php
public static function createMonyDeliverysFromSyndical($user, $department, $mony, $currencys_id = null, $rate = null): array
```

- `$department`: كائن القسم (الوكيل) الذي سيتم الخصم من حسابه.
- `$mony`: المبلغ (يخضع لحدود `min_mony` و `max_mony` من الإعدادات).
- `$currencys_id`, `$rate`: العملة وسعر الصرف.

---

### 2.3 `createMonyDeliverysFromCompanys`

شحن رصيد لموصل من قبل الشركة الرسمية (يتطلب صلاحيات مستخدم خلفي).

```php
public static function createMonyDeliverysFromCompanys($user, $department, $mony, $currencys_id = null, $rate = null, $other_options = []): array
```

- `$other_options`: مصفوفة خيارات إضافية مثل `periods_id`, `notes`, `date_at`, `paper_id`...

---

### 2.4 `createDeductingMonyDeliverysFromCompanys`

خصم مبلغ من حساب موصل (مثلاً عمولة التطبيق).

```php
public static function createDeductingMonyDeliverysFromCompanys($user, $mony, $currencys_id = null, $rate = null, $other_options = []): array
```

تدعم خيارات إضافية لتخصيص الملاحظات والأشخاص (`person_notes`, `debit_notes`, `credit_notes`...).

---

### 2.5 `addPayment`

إضافة دفعة مالية من طرف ثالث (مثل البنك) إلى حساب معين (مثلاً حساب طالب). تُستخدم بكثرة في أنظمة المدارس.

```php
public static function addPayment($options = []): array
```

تعتمد سلوكها بشكل كبير على الإعدادات في `tss.accounts::add_payment`، والتي تسمح بالتحكم في أي جانب يظهر فيه الشخص، وأي الحسابات تُستخدم.

#### مثال (دفعة بنكية لطالب)

```php
$result = TransactionsHelper::addPayment([
    'mony'              => 500,
    'credit_accounts_id'=> '2-2-1232010001', // حساب الطالب
    'credit_person'     => $student,
    'notes'             => 'دفعة عن طريق البنك',
]);
```

---

### 2.6 دوال العمليات المرتبطة بالطلبات (Orders)

مجموعة دوال متخصصة لتسجيل القيود المحاسبية المرتبطة بدورة حياة الطلب في نظام التجارة الإلكترونية:

| الدالة | الوصف |
| :--- | :--- |
| `createBrokerageFromOrders` | خصم عمولة التوصيل من الموصل. |
| `createPaymentFromOrders` | تسجيل قيد دفع قيمة الطلب (من العميل إلى حساب الشركة/الموصل). |
| `createBillShopFromOrders` | إثبات قيمة مشتريات المتجر (أو الموصل في حالة الدفع أونلاين). |
| `createBrokerageShopFromOrders` | خصم عمولة التطبيق من المتجر. |
| `createShippingTotalFromOrders` | تسجيل قيد كلفة التوصيل للموصل (في حالة الدفع أونلاين). |
| `createTipAmountFromOrders` | تسجيل قيد الإكرامية (بقشيش) للموصل. |
| `createPaymentMonyDeliverysFromCompanys` | (قديمة) محاولة سابقة لتسجيل قيد دفع متكامل للطلب. |

جميع هذه الدوال تستقبل كائن `Order` وتقوم بحساب المبالغ المناسبة (بناءً على `cart_total`, `shipping_total`, `order_total`...) وتمريرها إلى `createSingle` أو `createDeductingMonyDeliverysFromCompanys`. كما تتحقق من عدم وجود قيد مسبق لنفس العملية (لمنع التكرار).

---

## 3. دوال التحقق والاستعلام

### 3.1 `checkAllowPay`

التحقق من كفاية رصيد حساب معين لتنفيذ عملية شراء (أو أي عملية تتطلب رصيداً دائناً).

```php
public static function checkAllowPay($options = []): array
```

| الخيار | الوصف |
| :--- | :--- |
| `total` | المبلغ الإجمالي المطلوب проверка. |
| `code` | كود الحساب المراد فحصه. |
| (نفس خيارات `getBalances`) | للفلترة (قسم، فترة، عملة...). |

ترمي استثناء إذا كان الرصيد غير كافٍ.

#### مثال

```php
TransactionsHelper::checkAllowPay([
    'total' => 250,
    'code'  => '2-2-1231010001',
]);
// إذا لم يتم رمي استثناء، فالرصيد كافٍ.
```

---

### 3.2 `checkTransactions`

البحث عن قيد (رأس حركة) محدد والتأكد من وجوده.

```php
public static function checkTransactions($options = []): array
```

تستخدم بشكل رئيسي للتحقق من وجود قيد قبل إجراء عمليات التحديث أو الإلغاء. تعيد المصفوفة الموحدة مع `model` (كائن القيد) و `transactions_id`.

---

### 3.3 `getTransactionHeader`

استعلام متقدم عن رؤوس الحركات (`TransactionHeader`) مع فلترة شاملة.

```php
public static function getTransactionHeader($options = [], $isException = false): array
```

تدعم مجموعة كبيرة من خيارات الفلترة والترتيب والتجميع (`group_by`) والبحث حسب:

- معرف القيد (`id`, `is_force_id`)
- القسم، الفترة، نوع الحركة، العملية، نوع المستند الورقي.
- الحقول المتقدمة: `extend_id`, `modul_type`, `ref_type`, `batch_type`, `batch_id`
- البحث عبر التفاصيل (`accounts_id`, `person_id`, `person_type`, `currencys_id`)
- تصفية حسب نوع المستخدم (`users_ref_type`)
- نطاقات التاريخ (باستخدام `AccountHelper::getQueryDate`)
- خيارات الإرجاع: `is_query`, `is_first`, `is_collection`, `is_paginator`
- ترتيب خاص حسب عدد الإعجابات، الزيارات، إلخ (نفس سلوك `AccountsApi`)

#### مثال

```php
$result = TransactionsHelper::getTransactionHeader([
    'transactions_type' => [1,2], // أنواع محددة
    'departments_id'    => 3,
    'person_id'         => 42,
    'person_type'       => 'RainLab\User\Models\User',
    'is_paginator'      => true,
    'per_page'          => 20,
]);
$paginator = $result['data'];
```

---

## 4. دوال الترایت المتقدمة (`TransactionsMultiHelper`)

### 4.1 `validateJournalEntry`

التحقق من صحة بيانات قيد متعدد الأطراف دون إنشائه. تعيد التفاصيل بعد المعالجة.

```php
public static function validateJournalEntry($options = []): array
```

تتحقق من:
- وجود التفاصيل (`details`).
- توازن القيد (مجموع المدين = مجموع الدائن).
- توازن العملات الأجنبية (إذا كانت مختلطة).
- قيود عدد الأطراف (`multiplicity`).
- حدود المبالغ.
- فحص الأرصدة.

#### مثال

```php
$validation = TransactionsHelper::validateJournalEntry([
    'details' => [...],
]);
// إذا لم يرمي استثناء، فالقيد صالح.
```

---

### 4.2 `testJournalEntry`

محاكاة كاملة لإنشاء قيد متعدد الأطراف مع تحليل مفصل (بدون حفظ في قاعدة البيانات).

```php
public static function testJournalEntry($options = []): array
```

تعيد المصفوفة الموحدة مع إضافة مفتاح `analysis` الذي يحتوي على:
- `header_prepared`, `details_prepared`
- `totals`, `currency_analysis`
- `rule_violations`, `warnings`
- `test_passed` (هل اجتاز الاختبار؟)

#### مثال

```php
$test = TransactionsHelper::testJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1253010001', 'debit' => 1000],
        ['accounts_id' => '2-2-1231010001', 'credit' => 1000],
    ],
]);
print_r($test['analysis']);
```

---

### 4.3 `checkExchangeDifferences`

فحص قيد محاسبي موجود لتحديد ما إذا كان يحتاج إلى قيد فروق عملات، مع إمكانية إنشاء القيد تلقائياً.

```php
public static function checkExchangeDifferences($input, $options = []): array
```

| المعامل | الوصف |
| :--- | :--- |
| `$input` | معرف القيد (int) أو كائن `TransactionHeader`. |
| `$options` | `threshold`, `action` (`'entry'`, `'event'`, `'none'`), `force_check`, `relay_date`, `exchange_difference_account`. |

#### مثال

```php
$result = TransactionsHelper::checkExchangeDifferences(123, [
    'threshold' => 0.01,
    'action'    => 'entry',
]);
if ($result['exchange_entry']) {
    echo "قيد الفروق: " . $result['exchange_entry']->id;
}
```

---

### 4.4 `createExchangeDifferenceEntry`

إنشاء قيد فروق صرف ناتج عن تغير سعر الصرف بين تاريخ القيد وتاريخ الترحيل. عادة ما تُستدعى داخلياً.

```php
public static function createExchangeDifferenceEntry($originalHeader, $originalDetail, $difference, $officialRate, $originalOptions = []): array
```

---

### 4.5 دوال مساعدة داخلية (مختارة)

| الدالة | الوصف |
| :--- | :--- |
| `applyAccountRules` | تطبيق مجموعة قواعد على حساب للتحقق من نوعه (تدعم `AND`/`OR` ومشغلات متعددة). |
| `compareAttribute` | مقارنة قيمة مع عامل تشغيل (`=`, `LIKE`, `IN`, `REGEXP`...). |
| `validateSingleDetail` | تحقق شامل من تفصيل واحد (حساب، شخص، ملاحظات، حقول إضافية). |
| `processCurrency` | معالجة العملة والـ rate لكل تفصيل، مع دعم التحويل التلقائي. |
| `checkAccountBalance` | فحص رصيد حساب باستخدام `AccountHelper::getBalances`. |
| `mergeDefaultCompanyValues` | دمج القيم الافتراضية للشركة والفترة والعملات. |
| `prepareHeaderOptions` / `prepareDetailOptions` | تجهيز مصفوفات خيارات الرأس والتفصيل. |
| `validateMultiplicity` / `validateAmountLimits` / `validateBalance` | فحوصات القيود. |
| `preparePaperId` / `validatePaperId` / `generatePaperId` | إدارة `paper_id` (توليد تلقائي، تحقق من التكرار). |
| `prepareDateAt` / `validateDateAtRequired` | إدارة تاريخ القيد. |
| `fireHook` | تنفيذ hooks (`beforeValidate`, `afterValidate`). |

---

## 5. دالة ترقيم المستندات `seedDocsNumber`

تقوم بترقيم السجلات القديمة في جدول `tss_accounts_transaction_headers` التي لا تحتوي على `docs_number`.

```php
public static function seedDocsNumber($options = []): array
```

---

## 6. الأحداث (Events)

الكلاس والترايت يطلقان الأحداث التالية، مما يسمح بربط منطق مخصص:

| الحدث | المعاملات | الوصف |
| :--- | :--- | :--- |
| `tss.accounts.afterAddOpeningBalanceDelivery` | `$options, $header, $user` | بعد إضافة رصيد افتتاحي لموصل. |
| `tss.accounts.afterAddMonyDelivery` | `$options, $header, $user` | بعد شحن رصيد لموصل من الشركة. |
| `tss.accounts.afterBrokerage` | `$order, $state, $options` | بعد خصم عمولة من موصل أو متجر. |
| `tss.accounts.beforeJournalEntry` | `$options, $details` | قبل إنشاء قيد متعدد الأطراف. |
| `tss.accounts.afterJournalEntry` | `$header, $options, $details` | بعد إنشاء قيد متعدد الأطراف. |
| `tss.accounts.exchangeDifference` | `$header, $detail, $difference, $officialRate, $options` | عند اكتشاف فروق صرف (إذا كان `action='event'`). |
| `tss.accounts.beforeValidate` / `tss.accounts.afterValidate` | `$options, $details` | من hooks الترايت. |

---

## 7. الاعتماديات (Dependencies)

- `Tss\Accounts\Helpers\AccountHelper`
- `Tss\Accounts\Helpers\PersonHelper`
- `Tss\Accounts\Models\*` (TransactionHeader, TransactionsDetail, Account, Period, Currency, CostCenter...)
- `Tss\Basic\Helpers\BasicHelper`
- `Tss\Currency\Facades\Currency` (للوصول إلى أسعار الصرف)
- `Illuminate\Support\Facades\DB` (للمعاملات والاستعلامات الخام)
- `Illuminate\Support\Facades\Event`
- `Carbon\Carbon`
- `Config` (لقراءة الإعدادات)

---

## 8. ملاحظات هامة

1. **التوافق العكسي**: جميع الدوال السابقة للإصدار 1.0.38 لا تزال تعمل بنفس التوقيع. تمت إضافة `use TransactionsMultiHelper;` فقط، مع تهيئة مصفوفة النتيجة (`$result = [];`) لتفادي أي مشاكل.
2. **دعم الاختبارات**: الدوال `createJournalEntry` و `createEntry` تدعم وضع `is_test_create` الذي يلغي العملية بعد التحقق، مما يسهل اختبارات الوحدة.
3. **صلاحيات المستخدم**: لا تقوم هذه الدوال بالتحقق من الصلاحيات مباشرة، بل يُفترض أن يتم ذلك في الطبقات الأعلى (المتحكمات). بعض الدوال مثل `createMonyDeliverysFromCompanys` تقوم ببعض الفحوصات (مثل `hasAccess`).
4. **إدارة `paper_id`**: الترايت يدعم الآن خيارات متقدمة للتحكم في رقم المستند الورقي، راجع `preparePaperId` والخيارات المرتبطة.
5. **فحص نوع الحساب**: الميزة الجديدة (`debit_rules_account_types`/`credit_rules_account_types`) تسمح بمنع القيد على حسابات لا تنتمي لأنواع معينة (مثلاً "الخزينة فقط"). راجع أمثلة الاستخدام في ملف `createJournalEntry-config.md`.

---

## 9. أمثلة متكاملة

### سيناريو متجر إلكتروني – تسجيل قيد دفع كامل عند اكتمال الطلب

```php
$order = Order::find(100);

// 1. قيد دفع قيمة الطلب (من العميل إلى حساب الشركة أو الموصل حسب طريقة الدفع)
$paymentResult = TransactionsHelper::createPaymentFromOrders($order);

// 2. إذا كان المتجر، قيد إثبات قيمة المشتريات للمتجر
$billResult = TransactionsHelper::createBillShopFromOrders($order);

// 3. إذا كان المتجر وعليه عمولة، قيد خصم عمولة التطبيق
$brokerageResult = TransactionsHelper::createBrokerageShopFromOrders($order);

// 4. إذا كانت طريقة الدفع أونلاين، قيد كلفة التوصيل للموصل
$shippingResult = TransactionsHelper::createShippingTotalFromOrders($order);
```

### فحص فروق العملات لجميع القيود غير المرحلة

```php
$unrelayedHeaders = TransactionHeader::where('is_relay', false)->get();
foreach ($unrelayedHeaders as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, ['action' => 'entry']);
    if ($result['exchange_entry']) {
        Log::info("تم إنشاء قيد فروق للقيد رقم {$header->id}");
    }
}
```

---

## الخاتمة

يمثل كلاس `TransactionsHelper` مع سمة `TransactionsMultiHelper` العمود الفقري لجميع العمليات المحاسبية في منصة نانوسوفت. بفضل الدوال الموحدة، والتحقق الشامل، والدعم المتقدم للعملات وفروق الصرف، يمكن للمطورين بناء أنظمة مالية معقدة بسهولة وأمان. نأمل أن يكون هذا التوثيق دليلاً شاملاً يساعدك في تحقيق أقصى استفادة من هذه الأدوات القوية.


## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
