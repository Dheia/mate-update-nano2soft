# Documentation of Advanced and Practical Examples for the `NflowService` Class

**Namespace:** `Nano3\TelecomRecharge\Classes\Services`  
**Objective:** Provide comprehensive practical examples for using `NflowService` in various scenarios, from basic to advanced operations, illustrating inputs, expected outputs, and best practices.

---

## Introduction

This document provides a collection of practical examples covering various use cases for the `NflowService` class, starting from the simplest query and recharge operations to using flexible generic functions, managing Webhooks, validating data, and handling errors.

> **Note:** The examples here interact directly with the `NflowService` class. If you are using the `NflowController`, the response will be standardized according to the structure described in [API Documentation](./Nflow-API-Documentation-en.md). The class itself returns an array containing `success`, `status_code`, `data`, `error`, `message`.

Each example contains:

- **Scenario**: A brief description of the goal.
- **Code**: The code used to execute the operation.
- **Inputs**: Details of the options passed.
- **Expected Outputs**: The format of the result returned by the function, with an explanation of its impact on the system.
- **Notes**: Clarifications about special cases or tips.

---

## Prerequisites

- The `Nano3.TelecomRecharge` add-on installed.
- Valid credentials from [nflow.tech](https://nflow.tech) (username, password, account number, API token).
- Internet connection to access the API.

### Setting up the Class in Laravel / October CMS

Add the credentials to the `.env` file:

```ini
NANO3_TELECOM_RECHARGE_NFLOW_BASE_URL=https://nflow.tech
NANO3_TELECOM_RECHARGE_NFLOW_USERNAME=your_username
NANO3_TELECOM_RECHARGE_NFLOW_PASSWORD=your_password
NANO3_TELECOM_RECHARGE_NFLOW_ACCOUNT_NUMBER=your_account_number
NANO3_TELECOM_RECHARGE_NFLOW_API_TOKEN=your_api_token

NANO3_TELECOM_RECHARGE_NFLOW_LOGIN_ENDPOINT='/rest-api/login'
```

Then create the object:

```php
use Nano3\TelecomRecharge\Classes\Services\NflowService;

$api = new NflowService();
$login = $api->login();
if (!$login['success']) {
    die('Login failed: ' . json_encode($login));
}
```

**Note:** In all examples, we assume that `$api` is a ready object with a successful login.

---

## 1. Basic Operations (Agent and General Transaction Queries)

### 1.1 Query Agent Balance

**Scenario:** Check the current balance in your account.

```php
$balance = $api->getAccountBalance();
if ($balance['success']) {
    echo "Current balance: " . $balance['data']['agentBalance'] . " YER";
} else {
    echo "Error: " . $balance['error'];
}
```

**Expected Outputs (Example):**

```php
[
    'success' => true,
    'status_code' => 200,
    'data' => [
        'status' => true,
        'agentBalance' => 1500.50,
        'message' => 'Agent Balance Query Success',
        'transactionID' => 0
    ]
]
```

### 1.2 Query Status of a Previous Transaction

**Scenario:** Track the status of a previously submitted transaction using your transaction ID.

```php
$status = $api->getOperationStatus('TXN_12345');
if ($status['success']) {
    $data = $status['data'];
    $statusText = $api->getOperationStatusText($data['operationStatus']);
    echo "Transaction status: $statusText\n";
    if ($data['operationStatus'] == 1) {
        echo "Transaction successful. Reference number: " . $data['referenceID'];
    }
}
```

**Expected Outputs (On Success):**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'operationStatus' => 1,
        'mobileNumber' => '777777777',
        'price' => 100,
        'message' => 'Ready',
        'transactionID' => 'TXN_12345',
        'referenceID' => 98765
    ]
]
```

### 1.3 Get List of Agent Deposits

**Scenario:** View deposits made into your account (e.g., adding balance).

```php
$feed = $api->getFeedClientsBalance();
if ($feed['success']) {
    foreach ($feed['data']['data'] as $voucher) {
        echo "Date: {$voucher['Date']} - Amount: {$voucher['Amount']} {$voucher['Currency']}\n";
    }
}
```

---

## 2. Balance Queries for Different Networks

### 2.1 Query Yemen Mobile Balance

**Scenario:** Find out the balance of a Yemen Mobile subscriber line.

```php
$result = $api->queryYemenMobileBalance('777777777');
if ($result['success']) {
    $data = $result['data'];
    echo "Balance: {$data['mobileBalance']} YER\n";
    echo "Line type: {$data['mobileTypeName']}\n";
} else {
    echo "Query failed: " . $result['error'];
}
```

**Expected Outputs (Example):**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'mobileBalance' => 154.15,
        'mobileType' => 1,
        'mobileTypeName' => 'Prepaid',
        'message' => 'Balance Query Success'
    ]
]
```

### 2.2 Query ADSL Balance (Landline Internet)

**Scenario:** Find out the remaining data from an ADSL package.

```php
$result = $api->queryAdslBalance('12345678');
if ($result['success']) {
    $data = $result['data'];
    echo "Remaining balance: {$data['mobileBalance']}\n";
    echo "Package expiry date: {$data['expiredDate']}\n";
}
```

**Expected Outputs:**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'mobileBalance' => '44.18 GB',
        'expiredDate' => '21/10/2024',
        'offerAmount' => '12600',
        'message' => 'Balance Query Success'
    ]
]
```

### 2.3 Query Land Phone Balance

```php
$result = $api->queryLandPhoneBalance('012345678');
if ($result['success']) {
    echo "Balance: {$result['data']['mobileBalance']} YER";
}
```

### 2.4 Query Yemen 4G Balance

```php
$result = $api->queryYemen4GBalance('777777777');
if ($result['success']) {
    $data = $result['data'];
    echo "Balance: {$data['mobileBalance']}\n";
    echo "Expiry date: {$data['expiredDate']}\n";
}
```

### 2.5 Query Aden Net Balance

```php
$result = $api->queryAdenNetBalance('770000000');
if ($result['success']) {
    echo "Balance: {$result['data']['mobileBalance']} YER";
}
```

### 2.6 Query Yemen Mobile Loan

**Scenario:** Check if the customer is eligible for a loan (advance).

```php
$result = $api->queryYemenMobileLoan('777777777');
if ($result['success']) {
    $data = $result['data'];
    echo "Loan status: {$data['loanStatusString']}\n";
}
```

**Expected Outputs:**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'loanStatus' => false,
        'loanStatusString' => 'Number not loaned',
        'message' => 'Balance Query Success'
    ]
]
```

### 2.7 Query Yemen Mobile Packages

**Scenario:** Get the list of packages subscribed to by the customer.

```php
$result = $api->queryYemenMobileOffers('777777777');
if ($result['success']) {
    foreach ($result['data']['data'] as $offer) {
        echo "Package: {$offer['offerName']} (from {$offer['offerStartDate']} to {$offer['offerEndDate']})\n";
    }
}
```

---

## 3. Recharge Operations (Adding Balance)

### 3.1 Recharge Yemen Mobile Balance Using the Generic Function `chargeBalanceGeneric`

**Scenario:** Add 500 YER to a Yemen Mobile number.

```php
$txnId = 'TXN_' . uniqid();
$result = $api->chargeBalanceGeneric(1, '777777777', 500, $txnId);
if ($result['success']) {
    echo "Balance recharged successfully.\n";
    echo "Remaining balance in your account: " . $result['data']['agentBalance'];
    echo "Reference number: " . $result['data']['referenceID'];
} else {
    echo "Recharge failed: " . $result['error'];
}
```

**Expected Outputs:**

```php
[
    'success' => true,
    'data' => [
        'status' => true,
        'operationStatus' => 1,
        'agentBalance' => 1500.50,
        'price' => 500,
        'message' => 'Payment success',
        'transactionID' => 'TXN_xxx',
        'referenceID' => 123456
    ]
]
```

### 3.2 Recharge Sabafon Balance (Units)

**Scenario:** Recharge 5 units to a Sabafon number (multiplied by unit price in the system).

```php
$result = $api->chargeBalanceGeneric(3, '777777777', 5, 'TXN_' . uniqid());
```

**Notes:**  
- Networks (2) YOU, (3) Sabafon, (4) Y Telecom use the `Amount` value as the number of units, not a direct monetary amount.

### 3.3 Recharge ADSL Balance with 1000 YER

```php
$result = $api->chargeBalanceGeneric(5, '12345678', 1000, 'TXN_' . uniqid());
```

### 3.4 Recharge Land Phone Balance

```php
$result = $api->chargeBalanceGeneric(6, '012345678', 500, 'TXN_' . uniqid());
```

### 3.5 Recharge Balance Using the Original `chargeBalance` Function (To Manually Specify Service Number)

**Scenario:** Recharge Yemen Mobile balance (ServiceNumber = 103) while passing a WebHook.

```php
$result = $api->chargeBalance(
    1, 103, '777777777', 200, 'TXN_789',
    'https://your-site.com/webhook', 'SECURE_CODE'
);
```

---

## 4. Activating Packages (Offers)

### 4.1 Activate a Yemen Mobile Package (Using Generic Function)

**Scenario:** Activate a specific package for a Yemen Mobile number using the package code.

```php
$result = $api->activateOfferGeneric(1, '777777777', 'OFFER_CODE', 'TXN_' . uniqid());
if ($result['success']) {
    echo "Package activated successfully.";
}
```

**Notes:**  
- The package code (`OfferCode`) must be obtained from the system (often known or received from package inquiry).
- For the Yemen Mobile network, the package activation service is `104` (Re New Yemen Mobile Offer), but the generic function uses `107` for inquiry, not activation. According to the documentation, service `104` is `Re New Yemen Mobile Offer` (package renewal), and `107` is `Query Yemen Mobile Offers`. If you want to activate a new package, you might need to use `109` (New Yemen Mobile Offer). Therefore, in the `activateOfferGeneric` function, we assumed the package service number is `202` for YOU network, `303` for Sabafon, `402` for Y Telecom, `704` for Yemen 4G, and `1202` for Sabafon South. For Yemen Mobile, we use `104` in the dedicated function. It is recommended to use the shortcut functions if available.

### 4.2 Activate a Yemen Mobile Package Using the Dedicated Function

```php
$result = $api->activateOffer(1, 104, '777777777', 'OFFER_CODE', 'TXN_' . uniqid());
```

### 4.3 Activate a YOU Telecom Package

```php
$result = $api->activateOfferGeneric(2, '777777777', 'PREWhatsApp', 'TXN_' . uniqid());
```

### 4.4 Activate a Sabafon Package

```php
$result = $api->activateOfferGeneric(3, '777777777', 'SABAFON_OFFER', 'TXN_' . uniqid());
```

### 4.5 Activate a Sabafon South Package

```php
$result = $api->activateOfferGeneric(12, '777777777', 'SOUTH_OFFER', 'TXN_' . uniqid());
```

---

## 5. Recharging Games and Gift Cards

### 5.1 Fetch Available Game Categories

**Scenario:** Get a list of supported games with their prices and required data.

```php
$games = $api->getGamesAndGiftCardsCategories();
if ($games['success']) {
    foreach ($games['data']['data'] as $game) {
        echo "Game: {$game['ServiceName']} - {$game['CategoryName']}\n";
        echo "Price: {$game['LocalPrice']} {$game['LocalCurrencyName']}\n";
        echo "Min: {$game['MinQuantity']} - Max: {$game['MaxQuantity']}\n";
        echo "Required fields: ";
        foreach ($game['RequiredFields'] as $field) {
            echo "{$field['FieldName']} ({$field['FieldCode']}) ";
        }
        echo "\n\n";
    }
}
```

### 5.2 Recharge PUBG (60 UC)

**Scenario:** Recharge 60 UC for PUBG using `PlayerID`.

```php
$fields = ['PlayerID' => '1234567890'];
$result = $api->chargeGameOrGiftCard('pubg_60', $fields, 1, 'TXN_' . uniqid());
if ($result['success']) {
    echo "Game recharged successfully.";
}
```

### 5.3 Recharge Yoyo Chat (Variable Quantity)

**Scenario:** Recharge 500 coins for Yoyo Chat (allows free quantity).

```php
$fields = ['PlayerID' => 'user@example.com']; // Or any required identifier
$result = $api->chargeGameOrGiftCard('YoyoChat', $fields, 500, 'TXN_' . uniqid());
```

---

## 6. Using Flexible Generic Functions (`executeService`)

This function allows executing any service with specific network and service numbers, providing complete flexibility.

### 6.1 Execute a Service Not Covered by a Shortcut Function (e.g., Remove Yemen Mobile Package)

**Scenario:** Remove a specific package from a customer number.

```php
$result = $api->executeService(1, 108, [
    'MobileNumber' => '777777777',
    'OfferCode' => 'OFFER_ID'
], 'TXN_' . uniqid());
```

### 6.2 Execute a Balance Recharge Service with WebHook Using `executeService`

```php
$result = $api->executeService(3, 301, [
    'MobileNumber' => '777777777',
    'Amount' => 100,
    'WebHookURL' => 'https://your-site.com/webhook',
    'WebHookCode' => 'SECURE'
], 'TXN_' . uniqid());
```

### 6.3 Execute a Specific Balance Inquiry Service (e.g., ADSL Balance Query) Without Using the Shortcut Function

```php
$result = $api->executeService(5, 501, ['MobileNumber' => '12345678'], 'TXN_' . uniqid());
```

---

## 7. Webhook Support

### 7.1 Send Request with WebHook

**Scenario:** Recharge balance and notify your system of status updates via WebHook.

```php
$webhookUrl = 'https://your-app.com/api/yepayment/webhook';
$webhookCode = 'SECURE_' . uniqid();

$result = $api->chargeBalanceGeneric(1, '777777777', 100, 'TXN_' . uniqid(), $webhookUrl, $webhookCode);
```

### 7.2 Receiving WebHook in Laravel (Route Example)

```php
Route::get('/api/yepayment/webhook', function (Request $request) {
    $operationStatus = $request->get('OperationStatus'); // 1 success, 0 failure
    $txnId = $request->get('TransactionID');
    $referenceId = $request->get('ReferenceID');
    $price = $request->get('price');
    $message = $request->get('message');

    // Update transaction status in the database
    // ...
});
```

---

## 8. Data Validation Using Helper Functions

### 8.1 Validate Mobile Number According to Network

**Scenario:** Ensure the entered number is valid for Yemen Mobile before sending the request.

```php
$mobile = '777777777';
if ($api->isValidMobileNumber($mobile, 1)) {
    echo "Number is valid for Yemen Mobile.\n";
} else {
    echo "Number is invalid.\n";
}
```

### 8.2 Format Mobile Number (Remove Leading Zeros)

```php
$raw = '0 77 7777777';
$formatted = $api->formatMobileNumber($raw); // 777777777
```

### 8.3 Get Network Description

```php
$networkName = $api->getNetworkName(1); // "Yemen Mobile"
```

### 8.4 Check if Network Supports a Specific Service

```php
if ($api->supportsBalanceQuery(1)) {
    $api->queryBalance(1, '777777777');
}
```

### 8.5 Get List of Services for a Specific Network

```php
$services = $api->getServicesForNetwork(1);
foreach ($services as $serviceNumber => $description) {
    echo "$serviceNumber: $description\n";
}
```

---

## 9. Advanced Error Handling

### 9.1 Handle Specific Errors Based on HTTP Code

**Scenario:** Handle insufficient balance.

```php
$result = $api->chargeBalanceGeneric(1, '777777777', 1000, 'TXN_' . uniqid());
if (!$result['success']) {
    switch ($result['status_code']) {
        case 402:
            echo "Insufficient balance. Please top up your account.";
            break;
        case 401:
            echo "Authentication failed. Check credentials.";
            break;
        case 404:
            echo "Number or service not found.";
            break;
        case 423:
            echo "Service temporarily locked.";
            break;
        default:
            echo "Error: " . $result['error'];
    }
}
```

### 9.2 Using Test Mode (Simulate Change Without Saving)

Unlike `PriceHelper`, `NflowService` does not have a built-in `previewUpdate` function, but a preview can be achieved by executing an operation using a dummy transaction and not relying on the result. **Alternative**: You can use `getOperationStatus` after the operation to confirm success.

---

## 10. Best Practices

1. **Use Flexible Generic Functions** (`executeService`, `queryBalance`, `chargeBalanceGeneric`, `activateOfferGeneric`) to reduce duplicate code.
2. **Log all transactions** in the local database (transaction ID, network, amount, status) to track operations.
3. **Use WebHook** to get instant updates instead of periodic queries.
4. **Validate the number** before sending the request to avoid 406 errors.
5. **Monitor agent balance** before large operations to avoid 402 errors.
6. **Keep `transactionId` unique** (e.g., `TXN_` + uniqid()) to avoid duplicate transactions.
7. **In development environment, use test data** to avoid consuming balance.
8. **Log errors** using `Log::error` in Laravel to track issues.

---

## 11. Connection Test and Configuration Validation

**Scenario:** Ensure credentials are correct and connection is possible.

```php
$test = $api->testConnection();
if ($test['success']) {
    echo "Connection successful. Token exists: " . ($test['token'] ? 'Yes' : 'No');
} else {
    echo "Connection failed: " . $test['error'];
}
```

---

## 12. Integrating `NflowService` with Local Tracking System (Example)

**Scenario:** Save each recharge transaction in a local database.

```php
use Nano3\TelecomRecharge\Classes\Services\NflowService;

class NflowController extends Controller
{
    public function charge(Request $request)
    {
        $api = new NflowService();
        $login = $api->login();
        if (!$login['success']) {
            return response()->json(['error' => 'Login failed'], 500);
        }

        $txnId = 'TXN_' . uniqid();
        $result = $api->chargeBalanceGeneric(
            $request->network,
            $request->mobile,
            $request->amount,
            $txnId,
            route('webhook.yepayment')
        );

        // Save in local database
        Transaction::create([
            'transaction_id' => $txnId,
            'network' => $request->network,
            'mobile' => $request->mobile,
            'amount' => $request->amount,
            'status' => $result['success'] ? 'pending' : 'failed',
            'response' => json_encode($result)
        ]);

        return response()->json($result);
    }
}
```

---

## Conclusion

The `NflowService` class provides an integrated set of tools for interacting with nflow.tech services, through flexible generic functions and easy-to-use shortcut functions. Using the examples mentioned above, developers can implement any scenario related to balance recharge, balance inquiry, package activation, or game recharging, while considering security and reliability. We recommend using the generic functions (`executeService`, `queryBalance`, `chargeBalanceGeneric`, `activateOfferGeneric`) for maximum flexibility, and leveraging helper functions for data validation.

For the unified response structure when using the `NflowController`, refer to [API Interface Documentation](./Nflow-API-Documentation-en.md).

---
