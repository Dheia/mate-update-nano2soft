# توثيق متقدم ومكمّل لـ `TransactionsHelper` و `TransactionsMultiHelper`

## 1. الهيكل الداخلي وخريطة الدوال

لفهم الكلاس بعمق، من الضروري معرفة العلاقة بين الدوال العامة والخاصة وكيفية تدفق البيانات بينها.

### 1.1. الهيكل العام
- **`TransactionsHelper`**: الكلاس الرئيسي. يحتوي على دوال العمليات التجارية (`createSingle`, `addPayment`, ...) ودوال الاستعلام (`getTransactionHeader`, ...). يستخدم السمة `TransactionsMultiHelper`.
- **`TransactionsMultiHelper`**: سمة (Trait) توفر:
    - **الدوال العامة:** `createJournalEntry`, `createEntry`, `validateJournalEntry`, `testJournalEntry`, `checkExchangeDifferences`.
    - **الدوال المساعدة (protected static):** مسؤولة عن الخطوات الفردية في عملية إنشاء القيد متعدد الأطراف.

### 1.2. خريطة تدفق `createJournalEntry`
يوضح هذا المخطط تسلسل استدعاء الدوال عند تنفيذ `createJournalEntry`:
1.  `createJournalEntry(options)`
2.  `mergeDefaultCompanyValues(options)` (دمج القيم الافتراضية)
3.  `preparePaperId(options)` / `validatePaperIdRequired(options)` / `validatePaperId(options)`
4.  `prepareDateAt(options)` / `validateDateAtRequired(options)`
5.  `validatePeriod(options)`
6.  `fireHook('beforeValidate', options)`
7.  `validateJournalEntry(options)` -> داخلياً تستدعي `validateJournalDetails` بدورها تستدعي `validateSingleDetail`
8.  `fireHook('afterValidate', options, details)`
9.  `calculateTotals(details)`
10. `prepareHeaderOptions(options, headerPerson)`
11. **إنشاء وحفظ** الرأس والتفاصيل.
12. `handleExchangeDifferences(header, details, options)`

---

## 2. نظام القيم الافتراضية (`mergeDefaultCompanyValues`)

الدالة `mergeDefaultCompanyValues` هي مفتاح المرونة في `createJournalEntry`. تقوم بدمج `$options` المُمررة مع مصفوفة ضخمة من القيم الافتراضية. فهم هذه القيم هو المفتاح لاستخدام الدالة بشكل صحيح.

```php
$defaults = [
    'process_type'        => 4,
    'auto_generate_custom_name' => true,
    // ... جميع المفاتيح الأخرى
];
```
**نقطة هامة:** القيم الافتراضية للمفاتيح مثل `debit_account_required` هي `true`، مما يعني أن الدالة تفرض وجود حساب مدين ودائن في كل تفصيل **ما لم يتم تعطيل ذلك صراحةً** عبر الخيارات. هذا يختلف عن `createSingle` الذي يسمح بمرونة أكبر في الصيغة.

---

## 3. آلية العمل الداخلية لدوال "FromOrders"

هذه الدوال (`createPaymentFromOrders`, `createBillShopFromOrders`, إلخ) تتبع نمطاً موحداً، وفهم هذا النمط يسهل تتبع الأخطاء وتخصيصها.

1.  **التحقق من صحة كائن `Order`**: التحقق من أن الطلب في الحالة الصحيحة (مكتمل)، وأن القيد لم يتم إنشاؤه مسبقاً (باستخدام `TransactionHeader::where(...)->first()`).
2.  **حساب المبلغ**: يتم تحديد المبلغ (`$mony`) من خصائص الطلب (`order_total`, `cart_total`, `shipping_total`، إلخ).
3.  **إعداد خيارات `createSingle`**:
    - يتم جلب إعدادات العملية (مثل `transactions_type`, `debit_accounts_id`) من ملف `Config` (مثال: `tss.accounts::create_brokerage_shop_from_orders`).
    - يتم تعيين قيم `modul_type` (عادةً `'order'`) و `extend_id` (معرف الطلب) و `batch_type/id` لتسهيل تتبع القيد لاحقاً.
    - يتم تحديد "الشخص" (المدين أو الدائن) بناءً على سياق العملية (العميل، الموصل، المتجر).
4.  **استدعاء `createSingle`**: يتم تمرير جميع الخيارات المجهزة إلى `createSingle`.
5.  **معالجة النتيجة وإطلاق الأحداث**: في حالة النجاح، يتم تحديث الطلب (مثلاً تعيين `is_trans_brokerage = true`) وإطلاق حدث مثل `tss.accounts.afterBrokerage`.

---

## 4. نظام معالجة العملات وفروق الصرف (تفصيلي)

### 4.1. تدفق معالجة العملة في `validateSingleDetail` (دالة `processCurrency`)
1.  **تحديد العملة الرئيسية**: تُؤخذ من `$context['main_currencys_id']`.
2.  **تحديد عملة التفصيل**: تُؤخذ من `$detail['currencys_id']`، وإذا لم تكن موجودة، تُستخدم عملة الرأس `$context['currencys_id']`، ثم العملة الرئيسية.
3.  **تحديد سعر الصرف (`$rate`)**: يُؤخذ من `$detail['rate']`، أو `$context['rate']`، أو `1`.
4.  **التحويل (إذا كان `auto_convert_currency = true` والعملة أجنبية)**:
    - تُعتبر القيم المدخلة في `debit` و `credit` أجنبية.
    - تُنسخ القيم الأصلية إلى `debit_alien` و `credit_alien`.
    - تُحوّل القيم الأساسية إلى العملة الرئيسية: `$debit = $debit * $rate`.
5.  **حالة عدم التفعيل (`auto_convert_currency = false`)**:
    - القيم المدخلة في `debit` و `credit` تُعتبر بالعملة الرئيسية أساساً.
    - إذا كانت العملة أجنبية، يُفترض أن `beforeSave` في الموديل سيتولى التحويل (بافتراض أن المبلغ المُدخل هو الأجنبي). هذا قد يسبب ارتباكاً، ولذلك يُنصح باستخدام `auto_convert_currency = true` للتحكم الصريح.

### 4.2. آلية `handleExchangeDifferences`
- **شرط التشغيل:** يجب أن يكون الخيار `handle_exchange_differences = true` (افتراضياً `false`).
- **شرط الفحص:** وجود `relay_date_at` (تاريخ ترحيل) في الخيارات أو في كائن القيد (`$header->relay_date_at`)، وأن يكون مختلفاً عن `date_at` (تاريخ القيد).
- **آلية العمل:** لكل تفصيل بعملة غير رئيسية، يتم جلب سعر الصرف الرسمي في `relay_date`. إذا اختلف عن `rate` المستخدم في التفصيل، يتم حساب الفرق المالي. بناءً على `exchange_difference_action`:
    - `'entry'`: إنشاء قيد فروق.
    - `'event'`: إطلاق حدث للمعالجة الخارجية.

---

## 5. توسع في الخيارات المتقدمة

### 5.1. استخدام `disabled_rules` بفعالية
يمكن تمرير مصفوفة `disabled_rules` لتعطيل فحوصات معينة في حالات خاصة. المعرفات النصية المدعومة حالياً:
- `'balance_check'`: تعطيل فحص الرصيد.
- `'multiplicity'`: تعطيل فحص عدد الأطراف.
- `'amount_limits'`: تعطيل فحص حدود المبالغ (الإجمالية والجانبية).

**مثال:** إنشاء قيد تسوية لا يتطلب توازناً في عدد الأطراف أو فحص رصيد.
```php
TransactionsHelper::createEntry([
    'details' => [...],
    'disabled_rules' => ['balance_check', 'multiplicity'],
]);
```

### 5.2. نظام Hooks (`beforeValidate` و `afterValidate`)
يمكن تمرير دوال مجهولة (Closures) للتحكم في المنطق قبل أو بعد التحقق من صحة القيد.

**مثال: تعديل مبلغ التفاصيل تلقائياً قبل التحقق.**
```php
TransactionsHelper::createJournalEntry([
    'details' => [...],
    'beforeValidate' => function(&$options, &$details) {
        // إضافة 10% ضريبة على كل مبلغ مدين
        foreach ($details as &$detail) {
            if ($detail['debit'] > 0) {
                $detail['debit'] *= 1.10;
            }
        }
    },
]);
```

---

## 6. سيناريوهات استخدام متقدمة (غير مغطاة سابقاً)

### 6.1. برمجة `Batch` لإنشاء قيود دورية
باستخدام `createJournalEntry`، يمكن بناء نظام للقيود الدورية بسهولة.

```php
// inside a scheduled command
$templates = RecurringJournalTemplate::where('next_run_at', '<=', now())->get();
foreach ($templates as $template) {
    $options = $template->options; // تحميل options من قالب
    $options['date_at'] = now();

    $result = TransactionsHelper::createJournalEntry($options);

    if ($result['status']) {
        $template->last_run_at = now();
        $template->next_run_at = now()->addDays($template->interval_days);
        $template->save();
    } else {
        Log::error("Failed recurring journal: " . $result['error']);
    }
}
```

### 6.2. بناء مكتبة `JournalEntryBuilder` (نمط Fluent API)
لتسهيل إنشاء القيود المعقدة، يمكن بناء كلاس Builder:

```php
class JournalEntryBuilder
{
    protected $options = [];

    public function setType($typeHeader, $transactionsType) { ... }
    public function setDate($date) { ... }
    public function addDebit($accountId, $amount, $person = null) { ... }
    public function addCredit($accountId, $amount, $person = null) { ... }
    public function setNotes($notes) { ... }
    public function withBalanceCheck() { ... }
    public function execute() { 
        return TransactionsHelper::createEntry($this->options); 
    }
}

// الاستخدام:
$entry = new JournalEntryBuilder();
$entry->setType(BondsDay::class, 1)
      ->setDate(today())
      ->addDebit('2-2-1253010001', 500)
      ->addCredit('2-2-1231010001', 500)
      ->execute();
```

### 6.3. إنشاء قيد "تسوية" تلقائي من قيد غير متوازن
يمكن برمجة دالة تستخدم `testJournalEntry` ثم `afterValidate` لإنشاء قيد تسوية تلقائي للفروقات:

```php
$test = TransactionsHelper::testJournalEntry($options);
if (!$test['analysis']['test_passed']) {
    $difference = $test['analysis']['totals']['total_debit_base'] - $test['analysis']['totals']['total_credit_base'];
    if ($difference > 0) {
        $options['details'][] = ['accounts_id' => '2-2-SUSPENSE_ACCOUNT', 'credit' => $difference];
    } else {
        $options['details'][] = ['accounts_id' => '2-2-SUSPENSE_ACCOUNT', 'debit' => abs($difference)];
    }
    TransactionsHelper::createJournalEntry($options);
}
```

---

## 7. التصحيح والأخطاء الشائعة

### 7.1. كيفية استخدام `process_data` و `input_data`
تحتوي نتيجة الدوال على `input_data` (المدخلات بعد التنظيف الأولي) و `process_data` (البيانات التي استُخدمت فعلاً بعد الدمج مع القيم الافتراضية). عند التصحيح، قارن بينهما لفهم كيف تم تفسير خياراتك.

### 7.2. الأخطاء الشائعة
- **"Call to undefined method"**: ناتجة عن عدم استخدام `use \Tss\Accounts\Helpers\TransactionsMultiHelper;` في إصدار قديم من `TransactionsHelper`. تأكد من تحديث الكلاس.
- **"The given data was invalid" بدون تفاصيل**: قد تكون هناك مشكلة في قواعد التحقق المخصصة. تحقق من سجلات Laravel (`/storage/logs/laravel.log`) أو استخدم `testJournalEntry` لمعرفة المخالفات.
- **فروق العملات لا تُنشأ**: تأكد من تعيين `handle_exchange_differences => true` في الخيارات، وتأكد من أن القيد يحتوي على `relay_date_at` مختلف عن `date_at`.

---

## 8. دليل أفضل الممارسات والنصائح

1.  **اختبار القيد أولاً**: استخدم `testJournalEntry` أو `createJournalEntry($options, true)` قبل الإنشاء الفعلي.
2.  **ضبط الخيارات العامة في ملف `config`**: لتجنب تكرار الإعدادات، ضع القيم الافتراضية لمشروعك في ملف إعدادات (مثلاً `journal_entry_defaults`) وادمجها مع الخيارات قبل الاستدعاء.
3.  **استخدام `type_header` المناسب**: استخدم دائماً `BondsDay::class` أو `CatchReceipt::class` بدلاً من السلاسل النصية للاستفادة من الموديلات المتخصصة.
4.  **تعطيل الفحوصات بحذر**: `disabled_rules` مفيدة في حالات استثنائية، لكن إساءة استخدامها قد تؤدي لبيانات غير متسقة.
5.  **تسجيل الأخطاء**: استخدم `Log::error` مع `$result` في حالات الفشل لتتبع المشاكل في بيئة الإنتاج.



## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./Docs-TransactionsHelper-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
