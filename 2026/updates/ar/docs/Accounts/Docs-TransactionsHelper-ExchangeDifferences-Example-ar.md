# أمثلة عملية متقدمة لدوال فروق العملات

يقدم هذا المستند مجموعة من الأمثلة المتقدمة والسيناريوهات الواقعية لاستخدام دوال معالجة فروق العملات `checkExchangeDifferences` و `createExchangeDifferenceEntry`. يجب مطالعة [التوثيق الشامل للدالتين](./Docs-ExchangeDifferences-ar.md) قبل البدء.

---

## السيناريو 1: فحص دوري لجميع القيود غير المرحلة وإنشاء قيود فروق تلقائية

**الهدف:** وظيفة مجدولة (Scheduled Task) تعمل في نهاية كل شهر، تفحص جميع القيود غير المرحلة التي تحتوي على عملات أجنبية، وتُنشئ قيد فروق لكل فرق يتجاوز 0.01.

```php
use Tss\Accounts\Models\TransactionHeader;
use Tss\Accounts\Helpers\TransactionsHelper;

$unrelayedHeaders = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->whereHas('TransactionDetail', function($query) {
        $query->whereColumn('currencys_id', '!=', 'main_currencys_id');
    })
    ->get();

$processedCount = 0;
$entryCount = 0;

foreach ($unrelayedHeaders as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now()->endOfMonth(),
        'threshold' => 0.01,
    ]);

    if ($result['status'] && $result['exchange_entry']) {
        $entryCount++;
        \Log::info("تم إنشاء قيد فروق {$result['exchange_entry']->id} للقيد الأصلي {$header->id}");
    }
    $processedCount++;
}

echo "تم فحص {$processedCount} قيد. تم إنشاء {$entryCount} قيد فروق.";
```

---

## السيناريو 2: استخدام `force_check` لفحص قيود بدون تاريخ ترحيل

**الهدف:** أثناء إعداد النظام، تريد محاكاة فحص الفروقات لقيود جديدة ليس لها تاريخ ترحيل بعد، باستخدام تاريخ مستقبلي مفترض.

```php
$header = TransactionHeader::find(150); // قيد بدون relay_date_at

$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'      => 'none',
    'force_check' => true,
    'relay_date'  => '2026-09-30',
]);

if ($result['data']['has_differences']) {
    $total = $result['data']['total_difference'];
    echo "الفرق الإجمالي المتوقع عند الترحيل: {$total}";
    foreach ($result['data']['details'] as $diff) {
        echo "الحساب: {$diff['accounts_id']}, الفرق: {$diff['difference']}";
    }
}
```

---

## السيناريو 3: إطلاق حدث للمراجعة البشرية للفروقات الكبيرة

**الهدف:** لا نريد إنشاء قيود فروق تلقائياً للمبالغ الكبيرة (> 5000). بدلاً من ذلك، نطلق حدثاً يرسل إشعاراً للمدير المالي للمراجعة.

```php
Event::listen('tss.accounts.exchangeDifference', function ($payload) {
    if (abs($payload['total_difference']) > 5000) {
        // إرسال إشعار للمدير المالي
        $cfo = \Backend\Models\User::where('email', 'cfo@company.com')->first();
        \Mail::to($cfo)->send(new LargeExchangeDifferenceAlert($payload));
    }
});

// ثم فحص القيد
TransactionsHelper::checkExchangeDifferences($header, [
    'action'    => 'event',
    'threshold' => 0.01,
]);
```

---

## السيناريو 4: إنشاء قيد فروق يدوي مُخصص

**الهدف:** لديك تفصيل واحد تسبب في فارق كبير، وتريد إنشاء قيد فروق له مع تخصيص حساب الفروق ونوع الحركة والملاحظات.

```php
$originalHeader = TransactionHeader::find(200);
$originalDetail = $originalHeader->TransactionDetail()->where('currencys_id', 'USD')->first();

// حساب الفرق يدوياً بعد جلب السعر الرسمي
$officialRate = CurrencyHelper::getRate('USD', null, '2026-06-01');
$originalRate = $originalDetail->rate;
$alienAmount = $originalDetail->debit_alien ?: $originalDetail->credit_alien;
$difference = $alienAmount * ($officialRate - $originalRate);

$result = TransactionsHelper::createExchangeDifferenceEntry(
    $originalHeader,
    $originalDetail->toArray(),
    $difference,
    $officialRate,
    [
        'transactions_type' => 10, // نوع مخصص لفروق الصرف
        'notes'             => 'قيد فروق يدوي لأمر الشراء #123',
        'relay_date_at'     => '2026-06-01',
    ]
);

if ($result['status']) {
    echo "تم إنشاء قيد الفروق: " . $result['model']->id;
}
```

---

## السيناريو 5: فحص فروق عملات على دفعة قيود باستخدام نطاق مخصص

**الهدف:** فحص فروق العملات لجميع القيود المرتبطة بمشروع معين (محدد بـ `batch_id`).

```php
$batchId = 'PROJ-567';
$headers = TransactionHeader::with('TransactionDetail')
    ->where('batch_id', $batchId)
    ->where('transactions_type', 4)
    ->get();

$summary = [
    'total_headers' => $headers->count(),
    'with_differences' => 0,
    'entries_created' => 0,
    'total_amount' => 0,
];

foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now(),
        'threshold' => 0.001,
    ]);

    if ($result['data']['has_differences']) {
        $summary['with_differences']++;
        $summary['total_amount'] += $result['data']['total_difference'];
    }
    if ($result['exchange_entry']) {
        $summary['entries_created']++;
    }
}

print_r($summary);
```

---

## السيناريو 6: دمج خيارات `entry_options` لتجاوز حدود المبالغ

**الهدف:** قيد فروق صغير جداً (أقل من 1.0) يتم رفضه بسبب إعدادات `min_total_amount` الافتراضية. نستخدم `entry_options` لتعطيل هذا الفحص.

```php
$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'       => 'entry',
    'relay_date'   => now(),
    'entry_options'=> [
        'disabled_rules' => ['amount_limits'],
        'notes'          => 'قيد فروق أقل من الحد الأدنى',
    ],
]);
```

---

## السيناريو 7: برنامج نصي لترحيل الفروقات بأثر رجعي

**الهدف:** بعد ترحيل جميع قيود السنة الماضية، تريد إنشاء قيود فروق لأي فروقات لم تُعالج. ستستخدم `force_check` مع تاريخ ترحيل محدد (نهاية السنة).

```php
$historicalHeaders = TransactionHeader::with('TransactionDetail')
    ->whereYear('date_at', 2025)
    ->where('is_relay', true)
    ->get();

$relayDate = '2025-12-31';

foreach ($historicalHeaders as $header) {
    TransactionsHelper::checkExchangeDifferences($header, [
        'action'      => 'entry',
        'force_check' => true,
        'relay_date'  => $relayDate,
        'threshold'   => 0.01,
    ]);
}
```

---

## السيناريو 8: بناء أمر كونسول (Console Command) تفاعلي

**الهدف:** أمر `php artisan accounts:check-exchange` يسمح للمستخدم باختيار معرّف قيد أو `all`، واختيار الإجراء.

```php
// داخل handle() في أمر كونسول
$input = $this->ask('أدخل معرف القيد أو "all" للجميع');

if ($input === 'all') {
    $headers = TransactionHeader::with('TransactionDetail')->where('is_relay', false)->get();
} else {
    $headers = TransactionHeader::with('TransactionDetail')->find($input);
    if (!$headers) {
        $this->error('القيد غير موجود');
        return;
    }
    $headers = collect([$headers]);
}

$action = $this->choice('الإجراء:', ['none', 'event', 'entry'], 0);

foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, compact('action'));
    $this->info("القيد {$header->id}: " . ($result['data']['has_differences'] ? 'يوجد فروق' : 'لا يوجد فروق'));
}
```

---

## السيناريو 9: بناء تقرير شهري بفروقات العملات

**الهدف:** إنشاء تقرير مفصل يعرض جميع القيود التي تحتوي على فروقات عملات غير معالجة، مع تجميع حسب العملة.

```php
$month = 5;
$year = 2026;

$headers = TransactionHeader::with('TransactionDetail')
    ->whereYear('date_at', $year)
    ->whereMonth('date_at', $month)
    ->get();

$report = [];
$grandTotal = 0;

foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'      => 'none',
        'force_check' => true,
        'relay_date'  => now(),
    ]);
    
    if ($result['data']['has_differences']) {
        foreach ($result['data']['details'] as $diff) {
            $currency = $diff['currencys_id'];
            if (!isset($report[$currency])) {
                $report[$currency] = [
                    'count'        => 0,
                    'total_amount' => 0,
                    'entries'      => [],
                ];
            }
            $report[$currency]['count']++;
            $report[$currency]['total_amount'] += $diff['difference'];
            $report[$currency]['entries'][] = [
                'header_id'    => $header->id,
                'detail_id'    => $diff['detail_id'],
                'accounts_id'  => $diff['accounts_id'],
                'difference'   => $diff['difference'],
                'rate_change'  => $diff['rate_difference'],
            ];
            $grandTotal += $diff['difference'];
        }
    }
}

echo "تقرير فروقات العملات لشهر {$month}/{$year}\n";
echo str_repeat('-', 60) . "\n";
foreach ($report as $currency => $data) {
    echo "العملة: {$currency}\n";
    echo "عدد القيود: {$data['count']}\n";
    echo "إجمالي الفروق: " . number_format($data['total_amount'], 2) . "\n";
    echo str_repeat('-', 60) . "\n";
}
echo "المجموع الكلي: " . number_format($grandTotal, 2) . "\n";
```

---

## السيناريو 10: استخدام `entry_options` لربط قيد الفروق بدُفعة مراجعة

**الهدف:** عند إنشاء قيود فروق لعدة قيود أصلية، نريد ربطها جميعاً بنفس `batch_id` لتسهيل تتبع دورة المراجعة.

```php
$reviewBatchId = 'REVIEW-' . now()->format('Ymd-His');
$allResults = [];

$headersNeedingReview = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->whereHas('TransactionDetail', function($q) {
        $q->whereColumn('currencys_id', '!=', 'main_currencys_id');
    })
    ->get();

foreach ($headersNeedingReview as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now(),
        'threshold' => 0.01,
        'entry_options' => [
            'batch_type'  => 'exchange_review',
            'batch_id'    => $reviewBatchId,
            'notes'       => "قيد فروق - دفعة مراجعة {$reviewBatchId}",
            'disabled_rules' => ['amount_limits'],
        ],
    ]);
    
    $allResults[] = [
        'header_id'       => $header->id,
        'has_differences' => $result['data']['has_differences'] ?? false,
        'exchange_entry'  => $result['exchange_entry']->id ?? null,
    ];
}

// يمكن الآن استرجاع كل قيود الفروق لهذه الدفعة
$reviewEntries = TransactionHeader::where('batch_id', $reviewBatchId)->get();
echo "تم إنشاء {$reviewEntries->count()} قيد فروق في دفعة المراجعة {$reviewBatchId}";
```

---

## السيناريو 11: التعامل مع قيد يحتوي على أكثر من عملة أجنبية

**الهدف:** قيد واحد يحتوي على تفاصيل بعملات USD و EUR و GBP. نريد إنشاء قيد فروق يجمع كل الفروقات مرة واحدة.

```php
$header = TransactionHeader::with('TransactionDetail')->find(300);

// التحقق من العملات الموجودة
$currencies = $header->TransactionDetail
    ->whereColumn('currencys_id', '!=', 'main_currencys_id')
    ->pluck('currencys_id')
    ->unique();

echo "العملات الأجنبية في القيد: " . $currencies->implode(', ') . "\n";

$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'      => 'entry',
    'relay_date'  => now(),
    'threshold'   => 0.001,
]);

if ($result['data']['has_differences']) {
    echo "إجمالي الفروقات: {$result['data']['total_difference']}\n";
    echo "عدد التفاصيل المتأثرة: " . count($result['data']['details']) . "\n";
    
    // عرض تفاصيل كل فرق
    foreach ($result['data']['details'] as $index => $diff) {
        echo ($index + 1) . ". الحساب: {$diff['accounts_id']} | ";
        echo "العملة: {$diff['currencys_id']} | ";
        echo "السعر الأصلي: {$diff['original_rate']} | ";
        echo "السعر الرسمي: {$diff['official_rate']} | ";
        echo "الفرق: {$diff['difference']}\n";
    }
    
    if ($result['exchange_entry']) {
        echo "تم إنشاء قيد فروق جامع رقم: {$result['exchange_entry']->id}\n";
    }
}
```

---

## السيناريو 12: مراقبة فروق العملات اليومية مع نظام إنذار

**الهدف:** مهمة يومية تفحص فروق العملات، وإذا تجاوزت قيمة معينة، تُرسل إنذاراً للمدير المالي **دون** إنشاء قيود تلقائياً (إجراء Event فقط)، ثم يقوم المدير بمراجعة وإنشاء القيود من لوحة التحكم.

```php
// في مهمة يومية
$today = now()->toDateString();

$headers = TransactionHeader::with('TransactionDetail')
    ->whereDate('created_at', $today)
    ->get();

$alerts = [];

foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'      => 'event',
        'relay_date'  => now(),
    ]);
    
    if (($result['data']['total_difference'] ?? 0) > 10000) {
        $alerts[] = [
            'header_id'   => $header->id,
            'total_diff'  => $result['data']['total_difference'],
            'details'     => $result['data']['details'],
        ];
    }
}

if (!empty($alerts)) {
    // إرسال بريد إنذار للمدير المالي
    \Mail::to('cfo@company.com')->send(new ExchangeAlertsMail($alerts));
    
    \Log::warning('تم اكتشاف فروق عملات كبيرة', [
        'count'   => count($alerts),
        'headers' => array_column($alerts, 'header_id'),
    ]);
}
```

**مستمع الحدث (Listener):**

```php
Event::listen('tss.accounts.exchangeDifference', function ($payload) {
    $entry = ExchangeDifferenceLog::create([
        'header_id'       => $payload['header']->id,
        'total_difference'=> $payload['total_difference'],
        'differences'     => $payload['differences'],
        'status'          => 'pending_review',
    ]);
    
    // يمكن للمحاسب مراجعة هذه السجلات من لوحة التحكم
});
```

---

## السيناريو 13: سكربت ترحيل دفعة فروقات مع تتبع الحالة

**الهدف:** معالج يمر على سجلات `ExchangeDifferenceLog` المعلقة، وينشئ القيود بعد موافقة المراجع.

```php
$pendingLogs = ExchangeDifferenceLog::with('header.TransactionDetail')
    ->where('status', 'approved')
    ->whereNull('exchange_entry_id')
    ->get();

foreach ($pendingLogs as $log) {
    $header = $log->header;
    
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'      => 'entry',
        'relay_date'  => $log->review_date ?? now(),
        'entry_options'=> [
            'notes' => "قيد فروق معالج - سجل {$log->id}",
        ],
    ]);
    
    if ($result['exchange_entry']) {
        $log->update([
            'exchange_entry_id' => $result['exchange_entry']->id,
            'status'            => 'processed',
            'processed_at'      => now(),
        ]);
    } elseif (!$result['status']) {
        $log->update([
            'status'      => 'failed',
            'error_notes' => $result['error'],
        ]);
    }
}
```

---

## السيناريو 14: محاكاة وحدة اختبار (Unit Test) لفروق العملات

**الهدف:** اختبار أن `checkExchangeDifferences` يكتشف الفروقات بشكل صحيح باستخدام بيانات وهمية.

```php
public function testExchangeDifferencesDetection()
{
    // إنشاء قيد وهمي مع تفاصيل
    $header = TransactionHeader::factory()
        ->has(TransactionDetail::factory()->count(2)->sequence(
            ['debit' => 1000, 'credit' => 0, 'currencys_id' => 'USD', 'main_currencys_id' => 'SAR', 'rate' => 3.75, 'debit_alien' => 1000],
            ['debit' => 0, 'credit' => 3750, 'currencys_id' => 'SAR', 'main_currencys_id' => 'SAR', 'rate' => 1],
        ))
        ->create(['date_at' => '2026-01-01']);
    
    // محاكاة سعر صرف مختلف في تاريخ الترحيل
    Currency::shouldReceive('getRate')
        ->with('USD', null, \Mockery::any())
        ->andReturn(3.80); // ارتفع السعر
    
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'      => 'none',
        'relay_date'  => '2026-03-01',
    ]);
    
    $this->assertTrue($result['data']['has_differences']);
    $this->assertEquals(50, $result['data']['total_difference']); // (3.80 - 3.75) * 1000
}
```

---

## السيناريو 15: منع ازدواجية قيود الفروق عند التشغيل المتكرر

**الهدف:** التأكد من أن نفس القيد لا ينشئ قيد فروق مكرر إذا تم تشغيل المهمة أكثر من مرة.

```php
$headers = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->whereDoesntHave('TransactionDetail', function($query) {
        // استبعاد القيود التي تم إنشاء قيود فروق لها مسبقاً
        $query->where('modul_type', 'exchange_difference');
    })
    ->get();

foreach ($headers as $header) {
    // التحقق المزدوج: هل يوجد قيد فروق مرتبط بهذا القيد بالفعل؟
    $existingExchange = TransactionHeader::where('extend_id', $header->id)
        ->where('modul_type', 'exchange_difference')
        ->exists();
    
    if ($existingExchange) {
        continue; // تخطي، تمت معالجته مسبقاً
    }
    
    TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now(),
    ]);
}
```

---

## السيناريو 16: تكامل مع إغلاق الفترة المحاسبية

**الهدف:** عند إغلاق فترة محاسبية، يتم ترحيل جميع القيود أولاً، ثم فحص وإنشاء فروق العملات للفترة بالكامل.

```php
public function closePeriod($periodId)
{
    $period = Period::find($periodId);
    
    \DB::transaction(function() use ($period) {
        // 1. ترحيل جميع قيود الفترة
        TransactionHeader::where('periods_id', $periodId)
            ->where('is_relay', false)
            ->update([
                'is_relay'      => true,
                'relay_date_at' => now(),
            ]);
        
        // 2. فحص فروق العملات للفترة
        $headers = TransactionHeader::with('TransactionDetail')
            ->where('periods_id', $periodId)
            ->get();
        
        $exchangeSummary = [];
        foreach ($headers as $header) {
            $result = TransactionsHelper::checkExchangeDifferences($header, [
                'action'    => 'entry',
                'relay_date'=> $period->end_date,
                'entry_options' => [
                    'periods_id'    => $periodId,
                    'notes'         => "قيد فروق - إغلاق فترة {$period->name}",
                ],
            ]);
            
            if ($result['exchange_entry']) {
                $exchangeSummary[] = $result['exchange_entry']->id;
            }
        }
        
        // 3. تحديث حالة الفترة
        $period->update([
            'is_open'              => false,
            'closed_at'            => now(),
            'exchange_entries_ids' => $exchangeSummary,
        ]);
    });
}
```

---

## السيناريو 17: بناء واجهة API لفحص فروق العملات

**الهدف:** توفير نقطة API للمستخدمين تسمح بفحص فروقات قيد معين وإرجاع النتيجة.

```php
// في متحكم API
public function checkExchange($id)
{
    $header = TransactionHeader::with('TransactionDetail')->find($id);
    
    if (!$header) {
        return response()->json(['error' => 'القيد غير موجود'], 404);
    }
    
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action' => request('action', 'none'),
        'relay_date' => request('relay_date'),
        'threshold'  => request('threshold', 0.001),
    ]);
    
    if (!$result['status']) {
        return response()->json($result, 400);
    }
    
    // تجهيز استجابة API منسقة
    return response()->json([
        'header_id'        => $header->id,
        'exchange_entry'   => $result['exchange_entry'] ? $result['exchange_entry']->id : null,
        'has_differences'  => $result['data']['has_differences'],
        'total_difference' => $result['data']['total_difference'],
        'details'          => $result['data']['details'],
    ]);
}
```

**استدعاء API:**

```bash
curl -X POST "https://api.example.com/accounts/exchange/check/123" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"action": "entry", "relay_date": "2026-06-30"}'
```

---

### السيناريو 18: إعداد أمر كونسول لفحص الفروقات لفترة محاسبية محددة

**الهدف:** أمر `php artisan accounts:exchange-check {period_id}` يقوم بفحص فروقات جميع القيود في فترة محاسبية معينة وخيار لإنشاء القيود أو عرضها فقط.

```php
// داخل handle() لأمر كونسول
public function handle()
{
    $periodId = $this->argument('period_id');
    $period = Period::find($periodId);
    
    if (!$period) {
        $this->error('الفترة المحاسبية غير موجودة.');
        return;
    }
    
    $createEntries = $this->option('create');
    $threshold = (float) $this->option('threshold') ?: 0.01;
    
    $headers = TransactionHeader::with('TransactionDetail')
        ->where('periods_id', $periodId)
        ->where('is_relay', true)
        ->get();
    
    $this->info("تم العثور على {$headers->count()} قيد في الفترة.");
    
    $bar = $this->output->createProgressBar($headers->count());
    $summary = [];
    
    foreach ($headers as $header) {
        $result = TransactionsHelper::checkExchangeDifferences($header, [
            'action'      => $createEntries ? 'entry' : 'none',
            'relay_date'  => $period->end_date,
            'threshold'   => $threshold,
            'entry_options'=> [
                'periods_id' => $periodId,
                'notes'      => "قيد فروق - فترة {$period->name}",
            ],
        ]);
        
        if ($result['data']['has_differences']) {
            $summary[] = [
                'header_id'       => $header->id,
                'total_difference'=> $result['data']['total_difference'],
                'exchange_entry'  => $result['exchange_entry']->id ?? 'N/A',
            ];
        }
        
        $bar->advance();
    }
    
    $bar->finish();
    $this->newLine();
    
    // عرض ملخص
    $this->table(
        ['Header ID', 'Total Difference', 'Exchange Entry'],
        $summary
    );
}
```

---

### السيناريو 19: استبعاد حسابات معينة من فروق العملات

**الهدف:** تخصيص `createExchangeDifferenceEntry` لاستبعاد حسابات معينة (مثل حسابات رأس المال أو حسابات الضرائب) من قيود فروق الصرف.

```php
$excludedAccounts = Config::get('tss.accounts::exchange_excluded_accounts', []);

if (in_array($originalDetail['accounts_id'], $excludedAccounts)) {
    \Log::info("تم استبعاد الحساب {$originalDetail['accounts_id']} من فروق الصرف.");
    return ['status' => true, 'model' => null, 'skipped' => true];
}

$result = TransactionsHelper::createExchangeDifferenceEntry(
    $originalHeader,
    $originalDetail,
    $difference,
    $officialRate
);
```

**بديل: استخدام خيارات مخصصة:**

```php
$result = TransactionsHelper::createExchangeDifferenceEntry(
    $originalHeader,
    $originalDetail,
    $difference,
    $officialRate,
    [
        'debit_account_required' => false,
        'exchange_difference_account' => '2-2-8888880001', // حساب فروق مخصص
    ]
);
```

---

### السيناريو 20: إنشاء قيد فروق بعملة أصلية مختلفة عن العملة الرئيسية

**الهدف:** في بعض الحالات، قد نحتاج لإنشاء قيد فروق بعملة غير العملة الرئيسية (مثلاً لتسوية حساب بنكي أجنبي).

```php
$usdAccount = '2-2-125201-USD'; // حساب البنك بالدولار

$details = [
    [
        'accounts_id'    => $exchangeAccount,
        'debit'          => $difference > 0 ? abs($difference) : 0,
        'credit'         => $difference < 0 ? abs($difference) : 0,
        'currencys_id'   => 'USD',
        'rate'           => $officialRate,
        'notes'          => 'تسوية فروق صرف بالعملة الأصلية',
    ],
    [
        'accounts_id'    => $usdAccount,
        'debit'          => $difference < 0 ? abs($difference) : 0,
        'credit'         => $difference > 0 ? abs($difference) : 0,
        'currencys_id'   => 'USD',
        'rate'           => $officialRate,
        'notes'          => 'تسوية بنك الدولار',
    ],
];

$options = [
    'companys_id'       => $originalHeader->companys_id,
    'departments_id'    => $originalHeader->departments_id,
    'transactions_type' => 10,
    'date_at'           => now(),
    'currencys_id'      => 'USD',
    'auto_convert_currency' => true,
    'details'           => $details,
    'modul_type'        => 'exchange_difference',
    'extend_id'         => $originalHeader->id,
];

$result = TransactionsHelper::createJournalEntry($options);
```

---

### السيناريو 21: تصدير فروقات العملات إلى CSV للمراجعة الخارجية

**الهدف:** إنشاء تقرير CSV يحتوي على جميع فروقات العملات غير المعالجة لتقديمها للمراجع الخارجي.

```php
$headers = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->get();

$rows = [];
$rows[] = ['معرف القيد', 'تاريخ القيد', 'الحساب', 'العملة', 'السعر الأصلي', 'السعر الرسمي', 'الفرق', 'الحالة'];

foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'      => 'none',
        'force_check' => true,
        'relay_date'  => now(),
    ]);
    
    if ($result['data']['has_differences']) {
        foreach ($result['data']['details'] as $diff) {
            $rows[] = [
                $header->id,
                $header->date_at->toDateString(),
                $diff['accounts_id'],
                $diff['currencys_id'],
                number_format($diff['original_rate'], 6),
                number_format($diff['official_rate'], 6),
                number_format($diff['difference'], 2),
                $result['exchange_entry'] ? 'تمت المعالجة' : 'غير معالج',
            ];
        }
    }
}

$filename = 'exchange_differences_' . now()->format('Ymd_His') . '.csv';
$handle = fopen(storage_path("exports/{$filename}"), 'w');
foreach ($rows as $row) {
    fputcsv($handle, $row);
}
fclose($handle);

return response()->download(storage_path("exports/{$filename}"));
```

---

### السيناريو 22: تخصيص وقت الفحص الديناميكي بناءً على وقت الترحيل

**الهدف:** نظام يسمح بفحص الفروقات في أي وقت معين (مثلاً وقت إغلاق السوق) بدلاً من منتصف الليل.

```php
// في إعدادات Config أو Setting
$exchangeCheckTime = Setting::get('exchange_check_time', '15:00:00'); // 3 عصراً

$checkDateTime = Carbon::parse($header->relay_date_at->toDateString() . ' ' . $exchangeCheckTime);

$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'      => 'entry',
    'relay_date'  => $checkDateTime,
    'threshold'   => 0.001,
]);
```

---

### السيناريو 23: مراقبة أداء فحص الفروقات (Benchmarking)

**الهدف:** قياس أداء عملية فحص الفروقات لعدد كبير من القيود لتحسين الأداء.

```php
$start = microtime(true);
$processedCount = 0;
$differencesCount = 0;

$headers = TransactionHeader::with('TransactionDetail')
    ->whereYear('date_at', 2026)
    ->take(500)
    ->get();

foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action' => 'none',
        'relay_date' => now(),
    ]);
    
    $processedCount++;
    if ($result['data']['has_differences']) {
        $differencesCount++;
    }
}

$end = microtime(true);
$executionTime = round($end - $start, 2);

\Log::info("فحص فروقات العملات", [
    'total_processed'   => $processedCount,
    'with_differences'  => $differencesCount,
    'execution_time'    => "{$executionTime} ثانية",
    'avg_per_header'    => round($executionTime / max($processedCount, 1), 4) . ' ثانية',
]);
```

---

### السيناريو 24: معالجة فروقات العملات المجمعة حسب العملة

**الهدف:** بدلاً من إنشاء قيد فروق لكل قيد على حدة، تجميع الفروقات حسب العملة وإنشاء قيد واحد لكل عملة.

```php
$headers = TransactionHeader::with('TransactionDetail')
    ->where('periods_id', $periodId)
    ->get();

$aggregated = [];

// تجميع الفروقات
foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action' => 'none',
        'relay_date' => $period->end_date,
    ]);
    
    if ($result['data']['has_differences']) {
        foreach ($result['data']['details'] as $diff) {
            $currency = $diff['currencys_id'];
            $account = $diff['accounts_id'];
            
            if (!isset($aggregated[$currency])) {
                $aggregated[$currency] = [
                    'total_difference' => 0,
                    'accounts' => [],
                ];
            }
            
            $aggregated[$currency]['total_difference'] += $diff['difference'];
            
            if (!isset($aggregated[$currency]['accounts'][$account])) {
                $aggregated[$currency]['accounts'][$account] = 0;
            }
            $aggregated[$currency]['accounts'][$account] += $diff['difference'];
        }
    }
}

// إنشاء قيود فروق مجمعة
foreach ($aggregated as $currency => $currencyData) {
    if (abs($currencyData['total_difference']) > 0.01) {
        $details = [];
        
        // الطرف الأول: حساب فروق الصرف
        $details[] = [
            'accounts_id' => $exchangeAccount,
            'debit'       => $currencyData['total_difference'] > 0 ? abs($currencyData['total_difference']) : 0,
            'credit'      => $currencyData['total_difference'] < 0 ? abs($currencyData['total_difference']) : 0,
            'notes'       => "فروق مجمعة لعملة {$currency}",
        ];
        
        // أطراف الحسابات الأصلية
        foreach ($currencyData['accounts'] as $account => $amount) {
            $details[] = [
                'accounts_id' => $account,
                'debit'       => $amount < 0 ? abs($amount) : 0,
                'credit'      => $amount > 0 ? abs($amount) : 0,
                'notes'       => "فروق مجمعة لعملة {$currency}",
            ];
        }
        
        $entryResult = TransactionsHelper::createJournalEntry([
            'details'           => $details,
            'date_at'           => $period->end_date,
            'notes'             => "قيد فروق مجمع لعملة {$currency} - فترة {$period->name}",
            'transactions_type' => 10,
            'modul_type'        => 'exchange_difference_aggregated',
        ]);
        
        \Log::info("تم إنشاء قيد فروق مجمع لعملة {$currency}", [
            'entry_id' => $entryResult['model']->id ?? 'فشل',
        ]);
    }
}
```

---

### السيناريو 25: إعداد شاشة مراجعة فروق العملات في لوحة التحكم

**الهدف:** إنشاء Widget في لوحة تحكم Nano2Soft App لعرض ملخص فروقات العملات غير المعالجة.

```php
// داخل Dashboard Widget
public function render()
{
    $unprocessedHeaders = TransactionHeader::with('TransactionDetail')
        ->where('is_relay', false)
        ->whereHas('TransactionDetail', function($q) {
            $q->whereColumn('currencys_id', '!=', 'main_currencys_id');
        })
        ->take(10)
        ->get();
    
    $differencesData = [];
    $totalAmount = 0;
    
    foreach ($unprocessedHeaders as $header) {
        $result = TransactionsHelper::checkExchangeDifferences($header, [
            'action'      => 'none',
            'force_check' => true,
            'relay_date'  => now(),
        ]);
        
        if ($result['data']['has_differences']) {
            $differencesData[] = [
                'header'      => $header,
                'total_diff'  => $result['data']['total_difference'],
                'details'     => $result['data']['details'],
            ];
            $totalAmount += $result['data']['total_difference'];
        }
    }
    
    return View::make('widgets.exchange_differences', [
        'differences'  => $differencesData,
        'totalAmount'  => $totalAmount,
        'count'        => count($differencesData),
    ]);
}
```

---

### السيناريو 26: اختبار شامل لفروق العملات (Integration Test)

**الهدف:** اختبار تكاملي يتحقق من أن دورة حياة فروق العملات تعمل بشكل كامل.

```php
public function testFullExchangeDifferenceLifecycle()
{
    // 1. إنشاء قيد بعملة أجنبية
    $createResult = TransactionsHelper::createJournalEntry([
        'details' => [
            ['accounts_id' => '2-2-1253010001', 'debit' => 1000, 'currencys_id' => 'USD', 'rate' => 3.75],
            ['accounts_id' => '2-2-1231010001', 'credit' => 1000, 'currencys_id' => 'USD', 'rate' => 3.75],
        ],
        'main_currencys_id' => 'SAR',
        'currencys_id' => 'USD',
        'auto_convert_currency' => true,
    ]);
    
    $this->assertTrue($createResult['status']);
    $header = $createResult['model'];
    
    // 2. محاكاة سعر صرف مختلف في الترحيل
    Currency::shouldReceive('getRate')->andReturn(3.80);
    
    // 3. فحص الفروقات
    $checkResult = TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now()->addDays(30),
    ]);
    
    // 4. التحقق من إنشاء قيد الفروق
    $this->assertTrue($checkResult['status']);
    $this->assertNotNull($checkResult['exchange_entry']);
    $this->assertEquals(50, $checkResult['data']['total_difference']);
    
    // 5. التحقق من توازن قيد الفروق
    $exchangeEntry = $checkResult['exchange_entry'];
    $exchangeDetails = $exchangeEntry->TransactionDetail;
    $totalDebit = $exchangeDetails->sum('debit');
    $totalCredit = $exchangeDetails->sum('credit');
    $this->assertEquals($totalDebit, $totalCredit);
}
```


## التوثيق الإضافي

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


