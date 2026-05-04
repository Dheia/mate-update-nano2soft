# Advanced Practical Examples for Exchange Differences Functions

This document provides a set of advanced examples and real-world scenarios for using the exchange difference handling functions `checkExchangeDifferences` and `createExchangeDifferenceEntry`. You should review the [comprehensive documentation of the two functions](./Docs-ExchangeDifferences-en.md) before starting.

---

## Scenario 1: Periodic Check of All Unposted Entries and Automatic Creation of Difference Entries

**Goal:** A scheduled task that runs at the end of each month, checks all unposted entries containing foreign currencies, and creates an exchange difference entry for each difference exceeding 0.01.

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
        \Log::info("Created exchange difference entry {$result['exchange_entry']->id} for original entry {$header->id}");
    }
    $processedCount++;
}

echo "Processed {$processedCount} entries. Created {$entryCount} exchange difference entries.";
```

---

## Scenario 2: Using `force_check` to Check Entries Without a Posting Date

**Goal:** During system setup, you want to simulate difference checking for new entries that do not yet have a posting date, using a hypothetical future date.

```php
$header = TransactionHeader::find(150); // Entry without relay_date_at

$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'      => 'none',
    'force_check' => true,
    'relay_date'  => '2026-09-30',
]);

if ($result['data']['has_differences']) {
    $total = $result['data']['total_difference'];
    echo "Expected total difference at posting: {$total}";
    foreach ($result['data']['details'] as $diff) {
        echo "Account: {$diff['accounts_id']}, Difference: {$diff['difference']}";
    }
}
```

---

## Scenario 3: Firing an Event for Human Review of Large Differences

**Goal:** We do not want to automatically create exchange difference entries for large amounts (> 5000). Instead, we fire an event that sends a notification to the CFO for review.

```php
Event::listen('tss.accounts.exchangeDifference', function ($payload) {
    if (abs($payload['total_difference']) > 5000) {
        // Send notification to CFO
        $cfo = \Backend\Models\User::where('email', 'cfo@company.com')->first();
        \Mail::to($cfo)->send(new LargeExchangeDifferenceAlert($payload));
    }
});

// Then check the entry
TransactionsHelper::checkExchangeDifferences($header, [
    'action'    => 'event',
    'threshold' => 0.01,
]);
```

---

## Scenario 4: Manually Creating a Custom Difference Entry

**Goal:** You have a single detail causing a large difference, and you want to create a custom exchange difference entry with a specific difference account, transaction type, and notes.

```php
$originalHeader = TransactionHeader::find(200);
$originalDetail = $originalHeader->TransactionDetail()->where('currencys_id', 'USD')->first();

// Manually calculate the difference after fetching the official rate
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
        'transactions_type' => 10, // Custom type for exchange differences
        'notes'             => 'Manual exchange difference entry for purchase order #123',
        'relay_date_at'     => '2026-06-01',
    ]
);

if ($result['status']) {
    echo "Created exchange difference entry: " . $result['model']->id;
}
```

---

## Scenario 5: Checking Exchange Differences for a Batch of Entries Using a Custom Scope

**Goal:** Check exchange differences for all entries associated with a specific project (identified by `batch_id`).

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

## Scenario 6: Using `entry_options` to Bypass Amount Limits

**Goal:** A very small exchange difference entry (less than 1.0) is rejected due to the default `min_total_amount` setting. We use `entry_options` to disable this check.

```php
$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'       => 'entry',
    'relay_date'   => now(),
    'entry_options'=> [
        'disabled_rules' => ['amount_limits'],
        'notes'          => 'Exchange difference entry below minimum amount',
    ],
]);
```

---

## Scenario 7: Script to Retroactively Post Difference Entries

**Goal:** After posting all entries of the previous year, you want to create exchange difference entries for any differences that were not handled. You will use `force_check` with a specific posting date (end of the year).

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

## Scenario 8: Building an Interactive Console Command

**Goal:** A command `php artisan accounts:check-exchange` allowing the user to select an entry ID or `all`, and choose the action.

```php
// Inside the handle() method of a console command
$input = $this->ask('Enter entry ID or "all" for all');

if ($input === 'all') {
    $headers = TransactionHeader::with('TransactionDetail')->where('is_relay', false)->get();
} else {
    $headers = TransactionHeader::with('TransactionDetail')->find($input);
    if (!$headers) {
        $this->error('Entry not found');
        return;
    }
    $headers = collect([$headers]);
}

$action = $this->choice('Action:', ['none', 'event', 'entry'], 0);

foreach ($headers as $header) {
    $result = TransactionsHelper::checkExchangeDifferences($header, compact('action'));
    $this->info("Entry {$header->id}: " . ($result['data']['has_differences'] ? 'Has differences' : 'No differences'));
}
```

---

## Scenario 9: Generating a Monthly Exchange Difference Report

**Goal:** Create a detailed report showing all entries with unhandled exchange differences, grouped by currency.

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

echo "Exchange Difference Report for {$month}/{$year}\n";
echo str_repeat('-', 60) . "\n";
foreach ($report as $currency => $data) {
    echo "Currency: {$currency}\n";
    echo "Number of entries: {$data['count']}\n";
    echo "Total difference: " . number_format($data['total_amount'], 2) . "\n";
    echo str_repeat('-', 60) . "\n";
}
echo "Grand total: " . number_format($grandTotal, 2) . "\n";
```

---

## Scenario 10: Using `entry_options` to Link Difference Entries to a Review Batch

**Goal:** When creating exchange difference entries for several original entries, we want to link them all to the same `batch_id` to facilitate tracking the review cycle.

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
            'notes'       => "Exchange difference entry - review batch {$reviewBatchId}",
            'disabled_rules' => ['amount_limits'],
        ],
    ]);
    
    $allResults[] = [
        'header_id'       => $header->id,
        'has_differences' => $result['data']['has_differences'] ?? false,
        'exchange_entry'  => $result['exchange_entry']->id ?? null,
    ];
}

// All difference entries for this batch can now be retrieved
$reviewEntries = TransactionHeader::where('batch_id', $reviewBatchId)->get();
echo "Created {$reviewEntries->count()} exchange difference entries in review batch {$reviewBatchId}";
```

---

## Scenario 11: Handling an Entry with Multiple Foreign Currencies

**Goal:** One entry contains details in USD, EUR, and GBP. We want to create a single difference entry that aggregates all differences.

```php
$header = TransactionHeader::with('TransactionDetail')->find(300);

// Identify the currencies present
$currencies = $header->TransactionDetail
    ->whereColumn('currencys_id', '!=', 'main_currencys_id')
    ->pluck('currencys_id')
    ->unique();

echo "Foreign currencies in the entry: " . $currencies->implode(', ') . "\n";

$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'      => 'entry',
    'relay_date'  => now(),
    'threshold'   => 0.001,
]);

if ($result['data']['has_differences']) {
    echo "Total difference: {$result['data']['total_difference']}\n";
    echo "Number of affected details: " . count($result['data']['details']) . "\n";
    
    // Display details of each difference
    foreach ($result['data']['details'] as $index => $diff) {
        echo ($index + 1) . ". Account: {$diff['accounts_id']} | ";
        echo "Currency: {$diff['currencys_id']} | ";
        echo "Original rate: {$diff['original_rate']} | ";
        echo "Official rate: {$diff['official_rate']} | ";
        echo "Difference: {$diff['difference']}\n";
    }
    
    if ($result['exchange_entry']) {
        echo "Created aggregate exchange difference entry number: {$result['exchange_entry']->id}\n";
    }
}
```

---

## Scenario 12: Daily Monitoring of Exchange Differences with an Alert System

**Goal:** A daily task checks exchange differences, and if a value exceeds a certain threshold, sends an alert to the CFO **without** automatically creating entries (Event action only), then the CFO reviews and creates entries from the control panel.

```php
// In a daily scheduled task
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
    // Send alert email to CFO
    \Mail::to('cfo@company.com')->send(new ExchangeAlertsMail($alerts));
    
    \Log::warning('Large exchange differences detected', [
        'count'   => count($alerts),
        'headers' => array_column($alerts, 'header_id'),
    ]);
}
```

**Event Listener:**

```php
Event::listen('tss.accounts.exchangeDifference', function ($payload) {
    $entry = ExchangeDifferenceLog::create([
        'header_id'       => $payload['header']->id,
        'total_difference'=> $payload['total_difference'],
        'differences'     => $payload['differences'],
        'status'          => 'pending_review',
    ]);
    
    // Accountants can review these records from the control panel
});
```

---

## Scenario 13: Batch Processing Script for Pending Difference Logs

**Goal:** A processor goes through pending `ExchangeDifferenceLog` records and creates the entries after a reviewer approves them.

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
            'notes' => "Exchange difference entry processed - log {$log->id}",
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

## Scenario 14: Unit Test for Exchange Difference Detection

**Goal:** Test that `checkExchangeDifferences` correctly detects differences using mock data.

```php
public function testExchangeDifferencesDetection()
{
    // Create a mock entry with details
    $header = TransactionHeader::factory()
        ->has(TransactionDetail::factory()->count(2)->sequence(
            ['debit' => 1000, 'credit' => 0, 'currencys_id' => 'USD', 'main_currencys_id' => 'SAR', 'rate' => 3.75, 'debit_alien' => 1000],
            ['debit' => 0, 'credit' => 3750, 'currencys_id' => 'SAR', 'main_currencys_id' => 'SAR', 'rate' => 1],
        ))
        ->create(['date_at' => '2026-01-01']);
    
    // Mock a different exchange rate at the posting date
    Currency::shouldReceive('getRate')
        ->with('USD', null, \Mockery::any())
        ->andReturn(3.80); // Rate increased
    
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action'      => 'none',
        'relay_date'  => '2026-03-01',
    ]);
    
    $this->assertTrue($result['data']['has_differences']);
    $this->assertEquals(50, $result['data']['total_difference']); // (3.80 - 3.75) * 1000
}
```

---

## Scenario 15: Preventing Duplicate Difference Entries When Running Repeatedly

**Goal:** Ensure the same entry does not create duplicate exchange difference entries if the task is run more than once.

```php
$headers = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->whereDoesntHave('TransactionDetail', function($query) {
        // Exclude entries that have already had difference entries created
        $query->where('modul_type', 'exchange_difference');
    })
    ->get();

foreach ($headers as $header) {
    // Double-check: does a difference entry already exist for this header?
    $existingExchange = TransactionHeader::where('extend_id', $header->id)
        ->where('modul_type', 'exchange_difference')
        ->exists();
    
    if ($existingExchange) {
        continue; // Skip, already processed
    }
    
    TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now(),
    ]);
}
```

---

## Scenario 16: Integration with Period Closing

**Goal:** When closing an accounting period, first post all entries, then check and create exchange differences for the entire period.

```php
public function closePeriod($periodId)
{
    $period = Period::find($periodId);
    
    \DB::transaction(function() use ($period) {
        // 1. Post all entries of the period
        TransactionHeader::where('periods_id', $periodId)
            ->where('is_relay', false)
            ->update([
                'is_relay'      => true,
                'relay_date_at' => now(),
            ]);
        
        // 2. Check exchange differences for the period
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
                    'notes'         => "Exchange difference entry - closing period {$period->name}",
                ],
            ]);
            
            if ($result['exchange_entry']) {
                $exchangeSummary[] = $result['exchange_entry']->id;
            }
        }
        
        // 3. Update period status
        $period->update([
            'is_open'              => false,
            'closed_at'            => now(),
            'exchange_entries_ids' => $exchangeSummary,
        ]);
    });
}
```

---

## Scenario 17: Building an API Endpoint to Check Exchange Differences

**Goal:** Provide an API point for users to check differences for a specific entry and return the result.

```php
// In an API controller
public function checkExchange($id)
{
    $header = TransactionHeader::with('TransactionDetail')->find($id);
    
    if (!$header) {
        return response()->json(['error' => 'Entry not found'], 404);
    }
    
    $result = TransactionsHelper::checkExchangeDifferences($header, [
        'action' => request('action', 'none'),
        'relay_date' => request('relay_date'),
        'threshold'  => request('threshold', 0.001),
    ]);
    
    if (!$result['status']) {
        return response()->json($result, 400);
    }
    
    // Format API response
    return response()->json([
        'header_id'        => $header->id,
        'exchange_entry'   => $result['exchange_entry'] ? $result['exchange_entry']->id : null,
        'has_differences'  => $result['data']['has_differences'],
        'total_difference' => $result['data']['total_difference'],
        'details'          => $result['data']['details'],
    ]);
}
```

**API call:**

```bash
curl -X POST "https://api.example.com/accounts/exchange/check/123" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"action": "entry", "relay_date": "2026-06-30"}'
```

---

## Scenario 18: Setting Up a Console Command to Check Differences for a Specific Period

**Goal:** A command `php artisan accounts:exchange-check {period_id}` that checks differences for all entries in a given accounting period, with an option to create entries or just display them.

```php
// Inside the handle() method of a console command
public function handle()
{
    $periodId = $this->argument('period_id');
    $period = Period::find($periodId);
    
    if (!$period) {
        $this->error('Accounting period not found.');
        return;
    }
    
    $createEntries = $this->option('create');
    $threshold = (float) $this->option('threshold') ?: 0.01;
    
    $headers = TransactionHeader::with('TransactionDetail')
        ->where('periods_id', $periodId)
        ->where('is_relay', true)
        ->get();
    
    $this->info("Found {$headers->count()} entries in the period.");
    
    $bar = $this->output->createProgressBar($headers->count());
    $summary = [];
    
    foreach ($headers as $header) {
        $result = TransactionsHelper::checkExchangeDifferences($header, [
            'action'      => $createEntries ? 'entry' : 'none',
            'relay_date'  => $period->end_date,
            'threshold'   => $threshold,
            'entry_options'=> [
                'periods_id' => $periodId,
                'notes'      => "Exchange difference entry - period {$period->name}",
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
    
    // Display summary
    $this->table(
        ['Header ID', 'Total Difference', 'Exchange Entry'],
        $summary
    );
}
```

---

## Scenario 19: Excluding Certain Accounts from Exchange Differences

**Goal:** Customize `createExchangeDifferenceEntry` to exclude specific accounts (e.g., capital or tax accounts) from exchange difference entries.

```php
$excludedAccounts = Config::get('tss.accounts::exchange_excluded_accounts', []);

if (in_array($originalDetail['accounts_id'], $excludedAccounts)) {
    \Log::info("Account {$originalDetail['accounts_id']} excluded from exchange differences.");
    return ['status' => true, 'model' => null, 'skipped' => true];
}

$result = TransactionsHelper::createExchangeDifferenceEntry(
    $originalHeader,
    $originalDetail,
    $difference,
    $officialRate
);
```

**Alternative: using custom options**

```php
$result = TransactionsHelper::createExchangeDifferenceEntry(
    $originalHeader,
    $originalDetail,
    $difference,
    $officialRate,
    [
        'debit_account_required' => false,
        'exchange_difference_account' => '2-2-8888880001', // Custom difference account
    ]
);
```

---

## Scenario 20: Creating a Difference Entry in a Different Base Currency

**Goal:** In some cases, you may need to create an exchange difference entry in a currency other than the main currency (e.g., to settle a foreign bank account).

```php
$usdAccount = '2-2-125201-USD'; // USD bank account

$details = [
    [
        'accounts_id'    => $exchangeAccount,
        'debit'          => $difference > 0 ? abs($difference) : 0,
        'credit'         => $difference < 0 ? abs($difference) : 0,
        'currencys_id'   => 'USD',
        'rate'           => $officialRate,
        'notes'          => 'Exchange difference settlement in original currency',
    ],
    [
        'accounts_id'    => $usdAccount,
        'debit'          => $difference < 0 ? abs($difference) : 0,
        'credit'         => $difference > 0 ? abs($difference) : 0,
        'currencys_id'   => 'USD',
        'rate'           => $officialRate,
        'notes'          => 'USD bank account settlement',
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

## Scenario 21: Exporting Exchange Differences to CSV for External Audit

**Goal:** Create a CSV report containing all unhandled exchange differences to submit to an external auditor.

```php
$headers = TransactionHeader::with('TransactionDetail')
    ->where('is_relay', false)
    ->get();

$rows = [];
$rows[] = ['Entry ID', 'Entry Date', 'Account', 'Currency', 'Original Rate', 'Official Rate', 'Difference', 'Status'];

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
                $result['exchange_entry'] ? 'Processed' : 'Unprocessed',
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

## Scenario 22: Dynamic Check Time Based on Posting Time

**Goal:** A system that allows checking differences at a specific time (e.g., market closing time) instead of midnight.

```php
// In Config or Settings
$exchangeCheckTime = Setting::get('exchange_check_time', '15:00:00'); // 3:00 PM

$checkDateTime = Carbon::parse($header->relay_date_at->toDateString() . ' ' . $exchangeCheckTime);

$result = TransactionsHelper::checkExchangeDifferences($header, [
    'action'      => 'entry',
    'relay_date'  => $checkDateTime,
    'threshold'   => 0.001,
]);
```

---

## Scenario 23: Benchmarking Exchange Difference Checking Performance

**Goal:** Measure the performance of the difference checking process for a large number of entries to optimize performance.

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

\Log::info("Exchange difference checking benchmark", [
    'total_processed'   => $processedCount,
    'with_differences'  => $differencesCount,
    'execution_time'    => "{$executionTime} seconds",
    'avg_per_header'    => round($executionTime / max($processedCount, 1), 4) . ' seconds',
]);
```

---

## Scenario 24: Aggregated Exchange Difference Processing by Currency

**Goal:** Instead of creating a separate difference entry for each original entry, aggregate differences by currency and create one entry per currency.

```php
$headers = TransactionHeader::with('TransactionDetail')
    ->where('periods_id', $periodId)
    ->get();

$aggregated = [];

// Aggregate differences
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

// Create aggregated exchange difference entries
foreach ($aggregated as $currency => $currencyData) {
    if (abs($currencyData['total_difference']) > 0.01) {
        $details = [];
        
        // First party: exchange difference account
        $details[] = [
            'accounts_id' => $exchangeAccount,
            'debit'       => $currencyData['total_difference'] > 0 ? abs($currencyData['total_difference']) : 0,
            'credit'      => $currencyData['total_difference'] < 0 ? abs($currencyData['total_difference']) : 0,
            'notes'       => "Aggregated difference for currency {$currency}",
        ];
        
        // Original account parties
        foreach ($currencyData['accounts'] as $account => $amount) {
            $details[] = [
                'accounts_id' => $account,
                'debit'       => $amount < 0 ? abs($amount) : 0,
                'credit'      => $amount > 0 ? abs($amount) : 0,
                'notes'       => "Aggregated difference for currency {$currency}",
            ];
        }
        
        $entryResult = TransactionsHelper::createJournalEntry([
            'details'           => $details,
            'date_at'           => $period->end_date,
            'notes'             => "Aggregated exchange difference entry for currency {$currency} - period {$period->name}",
            'transactions_type' => 10,
            'modul_type'        => 'exchange_difference_aggregated',
        ]);
        
        \Log::info("Created aggregated exchange difference entry for currency {$currency}", [
            'entry_id' => $entryResult['model']->id ?? 'failed',
        ]);
    }
}
```

---

## Scenario 25: Setting Up a Dashboard Widget for Exchange Differences Review

**Goal:** Create a Widget in the Nano2Soft App control panel to display a summary of unprocessed exchange differences.

```php
// Inside Dashboard Widget
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

## Scenario 26: Integration Test for Full Exchange Difference Lifecycle

**Goal:** An integration test that verifies the entire exchange difference lifecycle works completely.

```php
public function testFullExchangeDifferenceLifecycle()
{
    // 1. Create an entry with foreign currency
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
    
    // 2. Mock a different exchange rate at posting
    Currency::shouldReceive('getRate')->andReturn(3.80);
    
    // 3. Check differences
    $checkResult = TransactionsHelper::checkExchangeDifferences($header, [
        'action'    => 'entry',
        'relay_date'=> now()->addDays(30),
    ]);
    
    // 4. Verify a difference entry was created
    $this->assertTrue($checkResult['status']);
    $this->assertNotNull($checkResult['exchange_entry']);
    $this->assertEquals(50, $checkResult['data']['total_difference']);
    
    // 5. Verify the difference entry is balanced
    $exchangeEntry = $checkResult['exchange_entry'];
    $exchangeDetails = $exchangeEntry->TransactionDetail;
    $totalDebit = $exchangeDetails->sum('debit');
    $totalCredit = $exchangeDetails->sum('credit');
    $this->assertEquals($totalDebit, $totalCredit);
}
```

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [Advanced Practical Examples for `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
