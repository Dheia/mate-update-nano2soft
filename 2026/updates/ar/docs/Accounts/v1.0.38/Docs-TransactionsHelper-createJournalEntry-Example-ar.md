# أمثلة عملية متقدمة لاستخدام دالة `createJournalEntry`

هذا المستند يقدم مجموعة من الأمثلة المتقدمة والسيناريوهات الواقعية لاستخدام دالة `createJournalEntry` من السمة `TransactionsMultiHelper`. تفترض الأمثلة familiarity بالخيارات الأساسية الموثقة في [الدليل الشامل للدالة](./Docs-createJournalEntry-ar.md).

---

## المثال 1: قيد متعدد الأطراف مع أشخاص وفحص ديناميكي لنوع الحساب

**السيناريو:** شركة تريد تسجيل مصروف إيجار (المدين: حساب المصروفات، الدائن: حساب البنك) مع ربط القيد بموظف مسؤول، بشرط أن يكون حساب البنك تابعاً لوحدة `Bank` فقط، وألا يقل المبلغ عن 1000.

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-5111010001', // حساب مصروف الإيجار
            'debit' => 5000,
            'notes' => 'إيجار شهر مايو'
        ],
        [
            'accounts_id' => '2-2-1252010001', // حساب البنك
            'credit' => 5000,
            'person' => $bankOfficer, // كائن موظف مسؤول
        ],
    ],
    'type_header' => BondsDay::class,
    'min_total_amount' => 1000,
    'credit_person_allowed' => true, // السماح بربط شخص بالدائن (البنك)
    'credit_check_account_types' => true,
    'credit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Bank']
    ],
]);

if ($result['status']) {
    echo "تم تسجيل مصروف الإيجار، رقم القيد: " . $result['model']->id;
}
```

---

## المثال 2: قيد بعملات متعددة مع معالجة فروق الصرف

**السيناريو:** شركة تسجل فاتورة شراء من مورد أجنبي بالدولار، وتريد تحويل المبالغ تلقائياً للعملة المحلية (SAR)، وإنشاء قيد فروق صرف تلقائياً عند الترحيل.

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-1311010001', // حساب المخزون
            'debit' => 1000,
            'currencys_id' => 'USD',
            'rate' => 3.75, // سعر الصرف وقت القيد
        ],
        [
            'accounts_id' => '2-2-2211010001', // حساب المورد الأجنبي
            'credit' => 1000,
            'currencys_id' => 'USD',
            'rate' => 3.75,
            'person_type' => 'SUPP', // نوع الشخص: مورد
            'person_id' => 42,
        ],
    ],
    'main_currencys_id' => 'SAR',
    'currencys_id' => 'USD',
    'auto_convert_currency' => true, // سيتم تخزين المبلغ بالدولار في alien
    'handle_exchange_differences' => true,
    'relay_date_at' => now()->addDays(30), // تاريخ الترحيل المتوقع
    'exchange_difference_action' => 'entry',
]);

// عند الترحيل (تنفيذ دالة checkExchangeDifferences) سيتم إنشاء قيد فروق إذا تغير السعر.
```

---

## المثال 3: قيد دوري تلقائي باستخدام `beforeValidate`

**السيناريو:** نظام فوترة شهري يحتاج لإنشاء قيد إيرادات في اليوم الأول من كل شهر. المبلغ الأساسي قابل للتعديل بواسطة دالة hook.

```php
$baseOptions = [
    'details' => [
        ['accounts_id' => '2-2-1211010001', 'debit' => 0], // سيتم تعديله
        ['accounts_id' => '2-2-4111010001', 'credit' => 0],
    ],
    'type_header' => BondsDay::class,
    'notes' => 'إيرادات شهرية',
    'header_modul_type_default' => 'recurring',
    'beforeValidate' => function(&$options, &$details) {
        // جلب المبلغ من قاعدة البيانات أو إعدادات
        $monthlyAmount = Setting::get('monthly_subscription_fee', 1000);
        $details[0]['debit'] = $monthlyAmount;
        $details[1]['credit'] = $monthlyAmount;
    },
];

$result = TransactionsHelper::createJournalEntry($baseOptions);
```

---

## المثال 4: فحص رصيد بحد ائتماني ديناميكي (Callback)

**السيناريو:** عميل يريد سحب مبلغ من حسابه (القيد: مدين: العميل، دائن: الخزينة). يجب ألا يتجاوز السحب رصيده مضافاً إليه حد ائتماني شخصي.

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => $customerAccountCode,
            'debit' => $requestedAmount,
            'person' => $customer,
        ],
        [
            'accounts_id' => '2-2-1253010001', // الخزينة
            'credit' => $requestedAmount,
        ],
    ],
    'debit_check_balance' => true,
    'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
        // جلب الحد الائتماني للعميل من نموذجه
        $customer = PersonHelper::getPersonObj($personType, $personId);
        $creditLimit = $customer->credit_limit ?? 0;

        if (($balance + $creditLimit) < $amount) {
            throw new ApplicationException("الرصيد ({$balance}) + الحد الائتماني ({$creditLimit}) غير كافٍ لسحب {$amount}.");
        }
    },
]);
```

---

## المثال 5: إنشاء قيد تسوية تلقائي لفروقات غير متوازنة

**السيناريو:** أثناء ترحيل بيانات من نظام قديم، قد تصل قيود غير متوازنة. نريد محاولة إنشاء قيد تسوية للفروقات الصغيرة تلقائياً.

```php
$options = [
    'details' => [
        ['accounts_id' => '2-2-1211010001', 'debit' => 1500],
        ['accounts_id' => '2-2-4111010001', 'credit' => 1450],
    ],
];

$test = TransactionsHelper::testJournalEntry($options);

if (!$test['analysis']['test_passed']) {
    $difference = $test['analysis']['totals']['total_debit_base'] - $test['analysis']['totals']['total_credit_base'];
    if (abs($difference) < 100) { // تسوية الفروقات الصغيرة فقط
        if ($difference > 0) {
            $options['details'][] = ['accounts_id' => '2-2-9999990001', 'credit' => $difference]; // حساب فروقات
        } else {
            $options['details'][] = ['accounts_id' => '2-2-9999990001', 'debit' => abs($difference)];
        }
        $result = TransactionsHelper::createJournalEntry($options);
        echo "تم إنشاء قيد مع تسوية: " . $result['model']->id;
    } else {
        echo "الفرق كبير جداً ويحتاج مراجعة يدوية.";
    }
}
```

---

## المثال 6: ربط قيد بدُفعة وتتبعها عبر `batch_id`

**السيناريو:** نظام المشتريات ينشئ عدة قيود (استلام بضاعة، ضريبة، خصم) كلها مرتبطة بنفس أمر الشراء. نستخدم `batch_type` و `batch_id` لربطهم.

```php
$purchaseOrderId = 12345;
$batchType = 'PurchaseOrder';

$entries = [];

// 1. قيد استلام المخزون
$entries[] = TransactionsHelper::createEntry([
    'details' => [
        ['accounts_id' => '2-2-1311010001', 'debit' => 10000, 'modul_type' => 'inventory'],
        ['accounts_id' => '2-2-5211010001', 'credit' => 10000, 'modul_type' => 'accrued'],
    ],
    'batch_type' => $batchType,
    'batch_id' => $purchaseOrderId,
]);

// 2. قيد الضريبة
$entries[] = TransactionsHelper::createEntry([
    'details' => [
        ['accounts_id' => '2-2-1311010001', 'debit' => 1500],
        ['accounts_id' => '2-2-2211020001', 'credit' => 1500],
    ],
    'batch_type' => $batchType,
    'batch_id' => $purchaseOrderId,
]);

// ... يمكن استرجاع كل القيود المرتبطة لاحقاً
$poEntries = TransactionHeader::where('batch_type', $batchType)
    ->where('batch_id', $purchaseOrderId)
    ->get();
```

---

## المثال 7: ترحيل بيانات تاريخية مع تعطيل فحوصات

**السيناريو:** ترحيل أرصدة افتتاحية من نظام قديم، حيث قد لا تكون الحسابات نشطة أو الأرصدة غير متوازنة مؤقتاً.

```php
$options = [
    'details' => [
        ['accounts_id' => '2-2-1211010001', 'debit' => 50000],
        ['accounts_id' => '2-2-3111010001', 'credit' => 50000],
    ],
    'type_header' => OpeningBalance::class,
    'is_custome_date' => true,
    'date_at' => '2025-01-01',
    'disabled_rules' => ['balance_check', 'multiplicity'],
    'debit_account_required' => false, // قد لا تكون الحسابات موجودة بعد
    'notes' => 'رصيد افتتاحي مرحّل',
];

try {
    $result = TransactionsHelper::createJournalEntry($options);
} catch (Exception $e) {
    Log::error('فشل ترحيل رصيد افتتاحي: ' . $e->getMessage());
}
```

---

## المثال 8: استخدام `afterValidate` لتدقيق خارجي

**السيناريو:** قبل حفظ أي قيد يتجاوز 100,000، يجب إرسال إشعار إلى المدير المالي.

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        ['accounts_id' => '2-2-1211010001', 'debit' => 150000],
        ['accounts_id' => '2-2-4111010001', 'credit' => 150000],
    ],
    'afterValidate' => function($options, $details) {
        $total = 0;
        foreach ($details as $d) {
            $total += $d['debit'];
        }
        if ($total > 100000) {
            Notification::send(CFO::first(), new LargeJournalEntryCreated($total));
        }
    },
]);
```

---

## المثال 9: بناء `JournalEntryBuilder` (Fluent API)

لتبسيط إنشاء القيود المعقدة في مشروعك، يمكنك بناء غلاف (Wrapper) خفيف:

```php
class JournalEntryBuilder
{
    private array $details = [];
    private array $options = [];

    public function setType(string $typeHeader, int $transactionsType): self
    {
        $this->options['type_header'] = $typeHeader;
        $this->options['transactions_type'] = $transactionsType;
        return $this;
    }

    public function setDate($date): self
    {
        $this->options['date_at'] = $date;
        return $this;
    }

    public function addLine(string $accountCode, float $debit, float $credit, $person = null, array $extra = []): self
    {
        $line = array_merge($extra, [
            'accounts_id' => $accountCode,
            'debit' => $debit,
            'credit' => $credit,
        ]);
        if ($person) {
            $line['person'] = $person;
        }
        $this->details[] = $line;
        return $this;
    }

    public function withNotes(string $notes): self
    {
        $this->options['notes'] = $notes;
        return $this;
    }

    public function requirePersonOnDebit(): self
    {
        $this->options['debit_person_required'] = true;
        return $this;
    }

    public function withBalanceCheck(callable $callback = null): self
    {
        $this->options['debit_check_balance'] = true;
        if ($callback) {
            $this->options['debit_balance_check_callback'] = $callback;
        }
        return $this;
    }

    public function execute(): array
    {
        $this->options['details'] = $this->details;
        return TransactionsHelper::createEntry($this->options);
    }
}

// استخدام:
$builder = new JournalEntryBuilder();
$result = $builder->setType(BondsDay::class, 4)
    ->setDate(today())
    ->addLine('2-2-5111010001', 5000, 0)
    ->addLine('2-2-1252010001', 0, 5000)
    ->withNotes('مصاريف متنوعة')
    ->withBalanceCheck()
    ->execute();
```

---

## خلاصة

توضح هذه الأمثلة العملاقة مرونة وقوة دالة `createJournalEntry`. من خلال الجمع بين الخيارات العديدة، يمكن للمطورين بناء منطق محاسبي معقد للغاية يغطي احتياجات المؤسسات المختلفة، مع الحفاظ على كود نظيف وسهل الصيانة. ننصح دائماً بالرجوع إلى الدليل الشامل للخيارات لفهم أعمق، واستخدام أدوات الاختبار (`testJournalEntry`) قبل نشر أي منطق جديد.



## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
