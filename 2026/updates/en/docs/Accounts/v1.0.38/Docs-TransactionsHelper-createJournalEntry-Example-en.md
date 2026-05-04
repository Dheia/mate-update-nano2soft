# Advanced Practical Examples for Using the `createJournalEntry` Function

This document provides a set of advanced examples and real-world scenarios for using the `createJournalEntry` function from the `TransactionsMultiHelper` trait. It assumes familiarity with the basic options documented in the [comprehensive function guide](./Docs-createJournalEntry-en.md).

---

## Example 1: Multi-party Entry with Persons and Dynamic Account Type Checking

**Scenario:** A company wants to record a rent expense (debit: expense account, credit: bank account) linked to a responsible employee, with the condition that the bank account belongs only to the `Bank` module, and the amount is at least 1000.

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-5111010001', // Rent expense account
            'debit' => 5000,
            'notes' => 'Rent for May'
        ],
        [
            'accounts_id' => '2-2-1252010001', // Bank account
            'credit' => 5000,
            'person' => $bankOfficer, // Responsible employee object
        ],
    ],
    'type_header' => BondsDay::class,
    'min_total_amount' => 1000,
    'credit_person_allowed' => true, // Allow linking a person to the credit side (bank)
    'credit_check_account_types' => true,
    'credit_rules_account_types' => [
        ['field' => 'modul_type', 'operator' => '=', 'value' => 'Bank']
    ],
]);

if ($result['status']) {
    echo "Rent expense recorded, entry number: " . $result['model']->id;
}
```

---

## Example 2: Multi-currency Entry with Exchange Difference Handling

**Scenario:** A company records a purchase invoice from a foreign supplier in USD, wants to automatically convert amounts to the local currency (SAR), and automatically create an exchange difference entry upon posting.

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => '2-2-1311010001', // Inventory account
            'debit' => 1000,
            'currencys_id' => 'USD',
            'rate' => 3.75, // Exchange rate at entry time
        ],
        [
            'accounts_id' => '2-2-2211010001', // Foreign supplier account
            'credit' => 1000,
            'currencys_id' => 'USD',
            'rate' => 3.75,
            'person_type' => 'SUPP', // Person type: supplier
            'person_id' => 42,
        ],
    ],
    'main_currencys_id' => 'SAR',
    'currencys_id' => 'USD',
    'auto_convert_currency' => true, // The amount will be stored in USD in alien
    'handle_exchange_differences' => true,
    'relay_date_at' => now()->addDays(30), // Expected posting date
    'exchange_difference_action' => 'entry',
]);

// When posting (calling the checkExchangeDifferences function), an exchange difference entry will be created if the rate changes.
```

---

## Example 3: Automatic Periodic Entry Using `beforeValidate`

**Scenario:** A monthly billing system needs to create a revenue entry on the first day of each month. The base amount can be modified by a hook function.

```php
$baseOptions = [
    'details' => [
        ['accounts_id' => '2-2-1211010001', 'debit' => 0], // Will be modified
        ['accounts_id' => '2-2-4111010001', 'credit' => 0],
    ],
    'type_header' => BondsDay::class,
    'notes' => 'Monthly revenue',
    'header_modul_type_default' => 'recurring',
    'beforeValidate' => function(&$options, &$details) {
        // Fetch the amount from the database or settings
        $monthlyAmount = Setting::get('monthly_subscription_fee', 1000);
        $details[0]['debit'] = $monthlyAmount;
        $details[1]['credit'] = $monthlyAmount;
    },
];

$result = TransactionsHelper::createJournalEntry($baseOptions);
```

---

## Example 4: Balance Check with Dynamic Credit Limit (Callback)

**Scenario:** A customer wants to withdraw an amount from their account (entry: debit: customer, credit: treasury). The withdrawal must not exceed their balance plus a personal credit limit.

```php
$result = TransactionsHelper::createJournalEntry([
    'details' => [
        [
            'accounts_id' => $customerAccountCode,
            'debit' => $requestedAmount,
            'person' => $customer,
        ],
        [
            'accounts_id' => '2-2-1253010001', // Treasury
            'credit' => $requestedAmount,
        ],
    ],
    'debit_check_balance' => true,
    'debit_balance_check_callback' => function($account, $amount, $balance, $personId, $personType, $context) {
        // Fetch the customer's credit limit from their model
        $customer = PersonHelper::getPersonObj($personType, $personId);
        $creditLimit = $customer->credit_limit ?? 0;

        if (($balance + $creditLimit) < $amount) {
            throw new ApplicationException("Balance ({$balance}) + credit limit ({$creditLimit}) insufficient to withdraw {$amount}.");
        }
    },
]);
```

---

## Example 5: Creating an Automatic Reconciliation Entry for Imbalances

**Scenario:** During data migration from an old system, unbalanced entries may arrive. We want to automatically create a reconciliation entry for small differences.

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
    if (abs($difference) < 100) { // Reconcile only small differences
        if ($difference > 0) {
            $options['details'][] = ['accounts_id' => '2-2-9999990001', 'credit' => $difference]; // Difference account
        } else {
            $options['details'][] = ['accounts_id' => '2-2-9999990001', 'debit' => abs($difference)];
        }
        $result = TransactionsHelper::createJournalEntry($options);
        echo "Entry created with reconciliation: " . $result['model']->id;
    } else {
        echo "Difference too large, needs manual review.";
    }
}
```

---

## Example 6: Linking Entries to a Batch and Tracking via `batch_id`

**Scenario:** A purchasing system creates several entries (goods receipt, tax, discount) all related to the same purchase order. We use `batch_type` and `batch_id` to link them.

```php
$purchaseOrderId = 12345;
$batchType = 'PurchaseOrder';

$entries = [];

// 1. Goods receipt entry
$entries[] = TransactionsHelper::createEntry([
    'details' => [
        ['accounts_id' => '2-2-1311010001', 'debit' => 10000, 'modul_type' => 'inventory'],
        ['accounts_id' => '2-2-5211010001', 'credit' => 10000, 'modul_type' => 'accrued'],
    ],
    'batch_type' => $batchType,
    'batch_id' => $purchaseOrderId,
]);

// 2. Tax entry
$entries[] = TransactionsHelper::createEntry([
    'details' => [
        ['accounts_id' => '2-2-1311010001', 'debit' => 1500],
        ['accounts_id' => '2-2-2211020001', 'credit' => 1500],
    ],
    'batch_type' => $batchType,
    'batch_id' => $purchaseOrderId,
]);

// ... All related entries can be retrieved later
$poEntries = TransactionHeader::where('batch_type', $batchType)
    ->where('batch_id', $purchaseOrderId)
    ->get();
```

---

## Example 7: Migrating Historical Data with Validations Disabled

**Scenario:** Migrating opening balances from an old system, where accounts may not be active or balances may be temporarily unbalanced.

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
    'debit_account_required' => false, // Accounts may not exist yet
    'notes' => 'Opening balance migrated',
];

try {
    $result = TransactionsHelper::createJournalEntry($options);
} catch (Exception $e) {
    Log::error('Failed to migrate opening balance: ' . $e->getMessage());
}
```

---

## Example 8: Using `afterValidate` for External Auditing

**Scenario:** Before saving any entry exceeding 100,000, a notification must be sent to the CFO.

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

## Example 9: Building a `JournalEntryBuilder` (Fluent API)

To simplify creating complex entries in your project, you can build a lightweight wrapper:

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

// Usage:
$builder = new JournalEntryBuilder();
$result = $builder->setType(BondsDay::class, 4)
    ->setDate(today())
    ->addLine('2-2-5111010001', 5000, 0)
    ->addLine('2-2-1252010001', 0, 5000)
    ->withNotes('Miscellaneous expenses')
    ->withBalanceCheck()
    ->execute();
```

---

## Summary

These advanced examples demonstrate the flexibility and power of the `createJournalEntry` function. By combining the many options, developers can build extremely complex accounting logic that covers the needs of different organizations, while keeping the code clean and maintainable. We always recommend referring to the comprehensive options guide for deeper understanding, and using the testing tools (`testJournalEntry`) before deploying any new logic.

---

## Additional Documentation

**Reference Documentation**:
- [`AccountHelper` Class Documentation](./Docs-AccountHelper-en.md)
- [`PersonHelper` Class Documentation](./Docs-PersonHelper-en.md)
- [`TransactionsHelper` Class Documentation](./Docs-TransactionsHelper-en.md)
- [Advanced `TransactionsHelper` Documentation](./Docs-TransactionsHelper-Advanced-en.md)
- [`TransactionsHelper` Function Index](./Docs-TransactionsHelper-Reference-Function-en.md)
- [Comprehensive `createJournalEntry` Documentation](./Docs-TransactionsHelper-createJournalEntry-en.md)
- [`createJournalEntry` Settings (Comprehensive)](./Docs-createJournalEntry-config-en.md)
- [`createJournalEntry` Settings (Compatible)](./Docs-createJournalEntry-config-v1-en.md)
- [Exchange Differences Functions Documentation](./Docs-TransactionsHelper-ExchangeDifferences-en.md)
- [Practical Examples for Exchange Differences Functions](./Docs-TransactionsHelper-ExchangeDifferences-Example-en.md)
