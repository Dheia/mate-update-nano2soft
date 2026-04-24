# Documentation of the `NflowService` Class

**Version:** 1.0  

**Namespace:** `Nano3\TelecomRecharge\Classes\Services`  
**Objective:** Integration with the nflow.tech electronic payment gateway to manage digital recharge operations, query balances of Yemeni telecommunications companies, activate packages, and recharge games and gift cards.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Requirements and Setup](#requirements-and-setup)
   - 2.1 [Register an Account on nflow.tech](#21-register-an-account-on-nflowtech)
   - 2.2 [Add Data to the Environment File](#22-add-data-to-the-environment-file)
   - 2.3 [Using the Class Without Laravel](#23-using-the-class-without-laravel)
   - 2.4 [Login Before Any Operation](#24-login-before-any-operation)
3. [Response Structure](#response-structure)
4. [General Operations (Global Operations)](#general-operations-global-operations)
5. [Flexible Generic Functions](#flexible-generic-functions)
   - 5.1 [`executeService`](#51-executeservice)
   - 5.2 [`queryBalance`](#52-querybalance)
   - 5.3 [`chargeBalanceGeneric`](#53-chargebalancegeneric)
   - 5.4 [`activateOfferGeneric`](#54-activateoffergeneric)
6. [Network-Specific Functions (Network-Specific Shortcuts)](#network-specific-functions-network-specific-shortcuts)
   - 6.1 [Balance Queries](#61-balance-queries)
   - 6.2 [Original Payment Functions (For Compatibility)](#62-original-payment-functions-for-compatibility)
7. [Data and Helper Functions](#data-and-helper-functions)
8. [Webhook Support](#webhook-support)
9. [Connection Test](#connection-test)
10. [Error Handling](#error-handling)
11. [Practical Examples](#practical-examples)
12. [Summary](#summary)

---

## Introduction

The `NflowService` class is an integrated tool designed to enable **integration with the nflow.tech platform** (Master Host) to manage all digital recharge services. The class provides a unified interface to perform operations such as:

- Querying agent balance.
- Querying transaction status.
- Querying customer balances (agent deposits).
- Fetching game and gift card categories.
- Querying balances of telecommunications companies (Yemen Mobile, ADSL, Land Phone, Yemen 4G, Aden Net).
- Direct balance recharge for any network (Yemen Mobile, Sabafon, YOU, Y Telecom, ADSL, Land Phone, Yemen 4G, Sabafon South, Aden Net).
- Activating internet packages and services (Yemen Mobile, YOU, Sabafon, Y Telecom, Yemen 4G, Sabafon South).
- Recharging games and gift cards (PUBG, Yoyo Chat, etc.).

The class features:

- **Flexible generic functions** to execute any service using network and service numbers.
- **Shortcut functions** for each network to simplify usage.
- **Professional error handling** with Webhook support for status updates.
- **Helper functions** to validate mobile numbers, format them, and retrieve lists of networks and services.
- **Full support for Laravel Environment** via environment variables.

---

## Requirements and Setup

### 2.1 Register an Account on nflow.tech

To obtain credentials, you must register on the official website:  
[https://nflow.tech](https://nflow.tech)

After registration, you can obtain:
- **UserName**
- **Password**
- **AccountNumber**
- **API Token**

> **Security Note:** Be careful not to share this data with any unauthorized person, and always use HTTPS.

### 2.2 Add Data to the Environment File (.env)

Add the following lines to your application's `.env` file (if using Laravel or October CMS):

```ini
# YemenRecharge By Nflow API Configuration
NANO3_TELECOM_RECHARGE_NFLOW_BASE_URL=https://nflow.tech
NANO3_TELECOM_RECHARGE_NFLOW_USERNAME=your_username_here
NANO3_TELECOM_RECHARGE_NFLOW_PASSWORD=your_password_here
NANO3_TELECOM_RECHARGE_NFLOW_ACCOUNT_NUMBER=your_account_number_here
NANO3_TELECOM_RECHARGE_NFLOW_API_TOKEN=your_api_token_here
```

### 2.3 Using the Class Without Laravel (Or with Custom Settings)

Data can be passed directly when creating the object:

```php
$api = new NflowService(
    'https://nflow.tech',
    'your_username',
    'your_password',
    'your_account_number',
    'your_api_token'
);
```

### 2.4 Login Before Any Operation

`login()` must be executed first to obtain an `accessToken`:

```php
$login = $api->login();
if (!$login['success']) {
    die('Login failed: ' . json_encode($login));
}
```

---

## Response Structure

All functions return an array with the same structure:

```php
[
    'success'     => bool,      // Request success (HTTP 200 and data)
    'status_code' => int,       // HTTP code
    'data'        => mixed,     // Received data (array or text)
    'error'       => string,    // Error message (if any)
    'message'     => mixed,     // Original message from API
]
```

**Example of a successful response:**

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

**Example of a failed response:**

```php
[
    'success' => false,
    'status_code' => 402,
    'error' => 'Insufficient balance',
    'message' => ['status' => false, 'message' => 'Insufficient balance']
]
```

---

## General Operations (Global Operations)

These functions pertain to the agent account and use `NetworkNumber = 0`.

| Function | Description | Example Request |
|----------|-------------|-----------------|
| `getAccountBalance()` | Query agent balance | `$api->getAccountBalance();` |
| `getOperationStatus($transactionId)` | Query status of a previous transaction | `$api->getOperationStatus('TXN_123');` |
| `getFeedClientsBalance()` | View agent deposits | `$api->getFeedClientsBalance();` |
| `getGamesAndGiftCardsCategories()` | Fetch game and gift card categories | `$api->getGamesAndGiftCardsCategories();` |

**Example:**

```php
$balance = $api->getAccountBalance();
if ($balance['success']) {
    echo "Current balance: " . $balance['data']['agentBalance'] . " YER";
}
```

---

## Flexible Generic Functions

These functions allow executing any service by specifying the network number and service number.

### 5.1 `executeService`

```php
public function executeService($networkNumber, $serviceNumber, $additionalFields = [], $transactionId = null)
```

Execute a custom service with any network and service number.  
**Parameters:**
- `$networkNumber`: Network number (e.g., 1 for Yemen Mobile)
- `$serviceNumber`: Service number (e.g., 101 for Yemen Mobile balance inquiry)
- `$additionalFields`: Array of additional fields (e.g., `MobileNumber`, `Amount`, `OfferCode`)
- `$transactionId`: Transaction ID (auto-generated if not passed)

**Example:**

```php
$result = $api->executeService(3, 301, [
    'MobileNumber' => '777777777',
    'Amount' => 100
], 'TXN_123');
```

### 5.2 `queryBalance`

```php
public function queryBalance($networkNumber, $mobileNumber, $transactionId = null)
```

Query balance for any network supporting the service (determines service number automatically).  
**Parameters:**
- `$networkNumber`: Network number (1,5,6,7,13)
- `$mobileNumber`: Mobile number or line
- `$transactionId`: Transaction ID (optional)

**Example:**

```php
$result = $api->queryBalance(1, '777777777');
```

### 5.3 `chargeBalanceGeneric`

```php
public function chargeBalanceGeneric($networkNumber, $mobileNumber, $amount, $transactionId = null, $webHookUrl = null, $webHookCode = null)
```

Recharge balance for any network supporting the service (determines service number automatically).  
**Parameters:**
- `$networkNumber`: Network number
- `$mobileNumber`: Mobile number
- `$amount`: Amount (for networks using units, this represents the number of units)
- `$transactionId`: Transaction ID (auto-generated)
- `$webHookUrl`: WebHook URL to receive status updates (optional)
- `$webHookCode`: Security code for the WebHook (optional)

**Example:**

```php
$result = $api->chargeBalanceGeneric(2, '777777777', 50, 'TXN_456');
```

### 5.4 `activateOfferGeneric`

```php
public function activateOfferGeneric($networkNumber, $mobileNumber, $offerCode, $transactionId = null, $webHookUrl = null, $webHookCode = null)
```

Activate a package for any network supporting the service (determines service number automatically).  
**Parameters:**
- `$networkNumber`: Network number
- `$mobileNumber`: Mobile number
- `$offerCode`: Package code
- `$transactionId`: Transaction ID (auto-generated)
- `$webHookUrl`: WebHook URL (optional)
- `$webHookCode`: WebHook code (optional)

**Example:**

```php
$result = $api->activateOfferGeneric(2, '777777777', 'PREWhatsApp', 'TXN_789');
```

---

## Network-Specific Functions (Network-Specific Shortcuts)

### 6.1 Balance Queries

| Function | Network | Example |
|----------|---------|---------|
| `queryYemenMobileBalance($mobileNumber, $transactionId)` | Yemen Mobile | `$api->queryYemenMobileBalance('777777777');` |
| `queryYemenMobileLoan($mobileNumber, $transactionId)` | Yemen Mobile (Loan) | `$api->queryYemenMobileLoan('777777777');` |
| `queryYemenMobileOffers($mobileNumber, $transactionId)` | Yemen Mobile (Packages) | `$api->queryYemenMobileOffers('777777777');` |
| `queryAdslBalance($mobileNumber, $transactionId)` | ADSL | `$api->queryAdslBalance('12345678');` |
| `queryLandPhoneBalance($mobileNumber, $transactionId)` | Land Phone | `$api->queryLandPhoneBalance('012345678');` |
| `queryYemen4GBalance($mobileNumber, $transactionId)` | Yemen 4G | `$api->queryYemen4GBalance('777777777');` |
| `queryAdenNetBalance($mobileNumber, $transactionId)` | Aden Net | `$api->queryAdenNetBalance('770000000');` |

**Example:**

```php
$result = $api->queryYemenMobileBalance('777777777');
if ($result['success']) {
    echo "Customer balance: " . $result['data']['mobileBalance'];
    echo "Line type: " . $result['data']['mobileTypeName'];
}
```

### 6.2 Original Payment Functions (For Compatibility)

| Function | Description |
|----------|-------------|
| `chargeBalance($networkNumber, $serviceNumber, $mobileNumber, $amount, $transactionId, $webHookUrl, $webHookCode)` | Recharge balance, specifying the service number manually. |
| `activateOffer($networkNumber, $serviceNumber, $mobileNumber, $offerCode, $transactionId, $webHookUrl, $webHookCode)` | Activate a package, specifying the service number manually. |
| `chargeGameOrGiftCard($linkCode, $fields, $quantity, $transactionId, $mobileNumber, $webHookUrl, $webHookCode)` | Recharge a game or gift card. |

**Example:**

```php
// Recharge Sabafon balance (service 301)
$result = $api->chargeBalance(3, 301, '777777777', 100, 'TXN_123');

// Activate YOU Telecom package (service 202)
$result = $api->activateOffer(2, 202, '777777777', 'PREWhatsApp', 'TXN_456');

// Recharge PUBG game (60 UC)
$fields = ['PlayerID' => '123456789'];
$result = $api->chargeGameOrGiftCard('pubg_60', $fields, 1, 'TXN_789');
```

---

## Data and Helper Functions

These functions help get information about networks and services and validate data.

| Function | Description |
|----------|-------------|
| `getNetworksList()` | Returns an array of network numbers and names. |
| `getNetworkName($networkNumber)` | Gets the network name from its number. |
| `isValidNetwork($networkNumber)` | Validates the network number. |
| `getAllServices()` | Returns a full list of services for each network. |
| `getServiceName($networkNumber, $serviceNumber)` | Gets the service description from network and service numbers. |
| `getServicesForNetwork($networkNumber)` | Gets the list of services for a specific network. |
| `isValidService($networkNumber, $serviceNumber)` | Validates the service number for a given network. |
| `getBillBalanceServiceNumber($networkNumber)` | Balance recharge service number for the network. |
| `getQueryBalanceServiceNumber($networkNumber)` | Balance inquiry service number for the network. |
| `getQueryOffersServiceNumber($networkNumber)` | Package inquiry service number for the network. |
| `supportsBalanceQuery($networkNumber)` | Does the network support balance inquiry? |
| `supportsBalanceBill($networkNumber)` | Does the network support direct balance recharge? |
| `supportsOffers($networkNumber)` | Does the network support packages? |
| `isValidMobileNumber($mobileNumber, $networkNumber)` | Validates mobile number according to the network (optional). |
| `formatMobileNumber($mobileNumber)` | Formats the number (removes leading zeros). |
| `getOperationStatusText($operationStatus)` | Converts operation status code to Arabic text. |

**Examples:**

```php
// List networks
$networks = $api->getNetworksList();
print_r($networks);

// Check network support
if ($api->supportsBalanceQuery(1)) {
    echo "Yemen Mobile supports balance inquiry";
}

// Validate mobile number
if ($api->isValidMobileNumber('777777777', 1)) {
    echo "Valid number for Yemen Mobile";
}

// Format number
$formatted = $api->formatMobileNumber('0 77 7777777'); // -> 777777777
```

---

## Webhook Support

When performing payment operations (recharge, packages, games), you can optionally pass `WebHookURL` and `WebHookCode`. The platform will send transaction status updates to this URL using the GET method with the following parameters:

| Parameter | Description |
|-----------|-------------|
| `OperationStatus` | 1: Success, 0: Failure |
| `WebHookCode` | The code you sent earlier |
| `TransactionID` | Transaction ID from your system |
| `ReferenceID` | Transaction ID in the nflow.tech system |
| `price` | The amount deducted from your balance |
| `message` | Explanatory message |

**Example for receiving Webhook in Laravel:**

```php
Route::get('/webhook/yepayment', function (Request $request) {
    $status = $request->get('OperationStatus');
    $txnId = $request->get('TransactionID');
    // ... save status in the database
});
```

**Sending the request with Webhook:**

```php
$result = $api->chargeBalanceGeneric(
    1, '777777777', 100, 'TXN_123',
    'https://your-site.com/webhook/yepayment',
    'SECURE_CODE'
);
```

---

## Connection Test

To ensure data validity and ability to connect to the server:

```php
$test = $api->testConnection();
if ($test['success']) {
    echo "Connection successful. Token exists: " . ($test['token'] ? 'Yes' : 'No');
} else {
    echo "Connection failed: " . $test['error'];
}
```

---

## Error Handling

All functions return an array with `success`, and errors can be handled as follows:

```php
$result = $api->queryYemenMobileBalance('777777777');
if ($result['success']) {
    // Success
} else {
    switch ($result['status_code']) {
        case 401:
            echo "Authentication failed, check credentials.";
            break;
        case 402:
            echo "Insufficient balance.";
            break;
        case 404:
            echo "Service or number not found.";
            break;
        case 423:
            echo "Service temporarily locked.";
            break;
        default:
            echo "Error: " . ($result['error'] ?? 'Unknown');
    }
}
```

---

## Practical Examples

### Example 1: Recharge Yemen Mobile Balance

```php
$txnId = 'TXN_' . uniqid();
$result = $api->chargeBalanceGeneric(1, '777777777', 500, $txnId);
if ($result['success']) {
    echo "Balance recharged successfully. Reference: " . $result['data']['referenceID'];
}
```

### Example 2: Activate YOU Telecom Package

```php
$result = $api->activateOfferGeneric(2, '777777777', 'PREWhatsApp', 'TXN_123');
```

### Example 3: Query ADSL Balance

```php
$result = $api->queryAdslBalance('12345678');
if ($result['success']) {
    echo "Remaining balance: " . $result['data']['mobileBalance'];
}
```

### Example 4: Recharge PUBG Game

```php
$fields = ['PlayerID' => '123456789'];
$result = $api->chargeGameOrGiftCard('pubg_60', $fields, 1, 'TXN_456');
```

---

## Summary

The `NflowService` class is an integrated tool for interacting with nflow.tech services, and provides:

- **A unified interface** for general operations, queries, and payments.
- **Flexible generic functions** `executeService` to execute any service with network and service numbers.
- **Shortcut functions** for each network to simplify daily usage.
- **Full Webhook support** for automated status updates.
- **Helper functions** for data validation and retrieving network/service information.
- **Professional error handling** with HTTP codes and appropriate messages.

Whether you are a developer needing to integrate these services into your application, or an administrator performing batch updates, `NflowService` provides the necessary tools securely and efficiently.

---

**Note:** For more details on the original API interface, refer to [https://nflow.tech/api-documentation](https://nflow.tech/api-documentation).  
For advanced examples and practical scenarios, refer to [Docs-NflowService-Advanced-Examples-en.md](./Docs-NflowService-Advanced-Examples-en.md).

---
