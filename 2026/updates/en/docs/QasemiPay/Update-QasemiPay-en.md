## 2026-04-28 - 2026-04-29

**Yepayment Add-on Update – New QasemiPay Payment Method**

**Update Type:** Major Feature Addition

---

### 1. Overview

This update adds an entirely new payment method called **QasemiPay**, an integrated payment gateway for **Qasemi Islamic Bank** via the **Purchase Code Service API**.  
The gateway is built according to the highest security and flexibility standards, supports two steps to complete payment (create transaction then confirm it), as well as immediate transaction status checking.  
An interactive test interface and integrated development tools have also been provided to ensure ease of integration and testing.

---

### 2. New Features

#### 2.1. Add QasemiPay Payment Gateway

- **Identifier:** `qasemi`
- **Display Name:** Qasemi Islamic Bank
- **Authentication Mechanism:** OAuth 2.0 (grant_type=password)
- **Supported Operations:**
  - Create Purchase Transaction
  - Confirm Transaction using `concurrencyStamp`
  - Query Transaction Status via `refId`
- **Customer Input Fields:**
  - `purchase_code` – Purchase code
  - `mobile_number` – Mobile number
  - `amount` – Amount
  - `currency` – Currency (YER/SAR)
  - `zone` – Zone (optional)
  - `agent` – Agent ID (optional)
  - `branch` – Branch (optional)
  - `notes` – Notes (optional)

#### 2.2. Advanced Settings in Control Panel

- **Base API URL** (production and test modes)
- **Authentication Credentials:** Client ID, Client Secret, Username, Password
- **Scope** – Flexible value, default `purchase_code_service`
- **Default Settings:** Currency, zone, agent, branch
- **Encrypted storage for sensitive data:** `client_secret` and `password`

#### 2.3. Integrated Test UI

- Access path: `/api/v1/yepayment/qasemipay/test-ui`
- **Features:**
  - Auth testing
  - Create test transaction with all fields
  - Confirm transaction using `concurrencyStamp`
  - Query transaction status
  - Full test automating all steps
  - Multi-run test (up to 10 times) to measure stability
  - Instant statistics (request count, success rate, latest logs)
  - Local test logs with export and clear capabilities

#### 2.4. New API Endpoints for Testing and External Integration

| Path | Method | Description |
|------|--------|-------------|
| `/qasemipay/test-auth` | POST | Verify authentication credentials |
| `/qasemipay/test-create-payment` | POST | Create test transaction |
| `/qasemipay/test-confirm-payment` | POST | Confirm a transaction |
| `/qasemipay/test-check-status` | GET | Query transaction status |
| `/qasemipay/test-full-payment` | POST | Full test (create + confirm + query) |
| `/qasemipay/stats` | GET | Gateway usage statistics |

---

### 3. Technical Changes

#### 3.1. Added Classes and Files

```
plugins/nano/yepayment/
├── paymenttypes/
│   └── qasemipay/
│       ├── QasemiPay.php                 ## Main payment gateway class
│       ├── _info.htm                     ## Display information in settings
│       ├── _test_info.htm                ## Quick test tools within settings
│       └── qasemi-ui.htm                 ## Integrated test interface (HTML/JS)
├── views/
│   └── qasemi-ui.htm                     (backup – optional)
└── lang/
    ├── ar/lang.php                       ## Add Arabic translation keys
    └── en/lang.php                       ## Add English translation keys
```

#### 3.2. Modifications to Existing Files

- **`Plugin.php`**  
  Added registration of `QasemiPay` in the `registerPaymentProviders()` function within the section for `allow_yemen_payment` (or `allow_oman_payment` as desired).

- **`routes.php`**  
  Added a full set of `qasemi` endpoints under the `yepayment` group, applying the same `cors`, `api`, and `web` middleware used for other gateways.

#### 3.3. Dependencies

- **Required:** `Nano.MicroCart` version 2.0+
- **Required:** `Nano.Helpers` (for `HttpHelper`)
- **Required:** `Illuminate\Support\Str` (for UUID generation)
- **Recommended:** Updated `nano.yepayment::lang` language files

---

### 4. Upgrade Guide

#### 4.1. For Developers (Refreshing the Package)

```bash
php artisan plugin:refresh Nano.Yepayment
```

Or reinstall the add-on from the internal marketplace.

#### 4.2. For Merchants (Gateway Configuration)

1. Go to **Control Panel → Store Settings → Payment Methods**.
2. Add a new payment method and select **Qasemi Islamic Bank**.
3. Enter the API credentials provided by Qasemi Bank:
   - Client ID / Client Secret
   - Username / Password
   - API URL (preferably use the test URL first)
4. Save the settings.

#### 4.3. Integration Testing

- Use the test interface: `/api/v1/yepayment/qasemipay/test-ui`
- Ensure authentication succeeds, then create a test transaction.
- After verifying all steps work, switch to the production URL.

---

### 5. Compatibility

- **NanoSoft App:** v2.x
- **PHP:** 8.0 / 8.1 / 8.2
- **NanoSoft Add-ons:**  
  - `Nano.MicroCart` (>=2.0)  
  - `Nano.Helpers` (>=1.2)  
  - `Nano.Orders` (>=1.5)
- **Databases:** Supports MySQL, PostgreSQL, SQLite (via Eloquent)

---

### 6. Other Improvements in This Version

- **Language file updates** – Added complete translations for QasemiPay gateway in Arabic and English.
- **Improved token storage security** – Ability to add token caching in the future (not enabled by default, but the structure is prepared).
- **Full documentation** – Added comprehensive documentation file for QasemiPay gateway (`QasemiPay-Documentation.md`).

---

### 7. Developer Notes

- QasemiPay gateway does not rely on redirecting the user to an external page, so `success_url` and `cancel_url` are not used (they can be added later if needed).
- If you wish to enable temporary token caching, add a caching layer inside `getAuthToken()` with a 3600-second TTL.
- All API requests use the unified `HttpHelper`, making it easy to trace errors and extend the gateway.

---

### 8. Bug Fixes

None – this version is dedicated to adding a new feature only.

---

### 9. Related Links

- [QasemiPay API Documentation ](./Docs-QasemiPay-en.md)
- [Purchase Code Service API Documentation – Qasemi Bank (PDF)](Purchase%20Code%20Service%20API%20Documentation%20(Draft).pdf)
- [NanoSoft Payment Gateway Development Guide](https://docs.nano2soft.com/payment-gateways)
- [Technical Support Channel](https://nano2soft.com)

---

**This update prepared by:**  
NanoSoft Development Team – Electronic Payments Department - Qasemi Bank  
**References:** Dheia Ali, Nano2Soft

---
