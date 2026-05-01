# Monthly Report of NanoSoft Software Development Team Achievements  
## April 2026  

### **Introduction**  
This report provides a professional and comprehensive summary of the achievements of the development team at NanoSoft Software Company during the period from April 1 to April 30, 2026. This month was distinguished by unprecedented qualitative achievements, including the **launch of three major integrated systems**: `Nano.FileUpload` for centralized file upload management, `Nano3.Kyc` for identity verification (KYC), and `Nano3.TelecomRecharge` for interacting with digital recharge service providers in Yemen. The month also saw the **addition of four new payment gateways** (ThawaniPay, YottaPay, QasemiPay, JaibPay) to the unified payment system, the development of a visual query builder system (`Nano2.QueryBuilder`), and the launch of the second version of the dynamic reporting system with support for predefined reports. These achievements reflect the team's commitment to delivering advanced technical solutions that meet the needs of local and regional markets.

---

## **Executive Summary**  
During April 2026, the team completed **more than 20 major updates**, distributed across seven strategic axes:  

1. **Launch of a Centralized File Upload Management System** – `Nano.FileUpload` with support for permissions, temporary uploads, multi-storage, automatic image conversion, and an integrated API.  
2. **Launch of an Integrated Identity Verification (KYC) System** – `Nano3.Kyc` with support for 28+ document types, dynamic fields, intelligent KYC assessment, and integration with `Nano.FileUpload`.  
3. **Addition of Four New Payment Gateways** – ThawaniPay (Oman), YottaPay/Sabacash (Yemen), QasemiPay (Qasemi Islamic Bank), and JaibPay (Jaib Wallet).  
4. **Launch of a Visual Query Builder System** – `Nano2.QueryBuilder` with a custom `FormWidget` and `QueryBuilderParserHelper` class.  
5. **Launch of the Second Version of the Dynamic Reporting System** – Support for predefined reports, an integrated API, and export to multiple formats.  
6. **Development of a Digital Recharge System** – `Nano3.TelecomRecharge` for integration with the nflow.tech platform (Master Host) in Yemen.  
7. **Continuous Improvements** – Updates to `Nano.TagsApi`, addition of new SMS providers, and security enhancements in `Nano.FileUpload`.  

All updates were made while maintaining backward compatibility, providing comprehensive documentation in both Arabic and English, and adding advanced practical examples.

---

## Summary Tables of Key Achievements

### 1. Major New Systems and Plugins (Completely New)

| System / Plugin | Description | Version / Date | Key Features |
| :--- | :--- | :--- | :--- |
| **Nano.FileUpload** | Centralized file upload management system via API | 1.0.1 → 1.2.8 (March 15 – April 30) | Model registration, granular permissions, temporary upload, multi-storage (S3/FTP/Local), automatic image processing (resize/watermark), events, WebSocket, dangerous file protection, comprehensive testing. |
| **Nano3.Kyc** | Integrated identity verification (KYC) system | 1.0.1 → 1.2.8 (April 10 – 30) | Support for 28+ document types, dynamic fields, intelligent KYC assessment (completion percentage, score, recommendations), file upload integration, verified document protection, alternative groups, advanced caching, full API and backend admin interface. |
| **Nano3.TelecomRecharge** | Unified platform for digital recharge providers (Yemen) | 1.0.0 (March 27 – April 23) | Full integration with nflow.tech (Master Host), methods: balance inquiry, recharge, package activation, game/gift card recharge, Webhook support, comprehensive API + interactive test UI. |
| **Nano2.QueryBuilder** | Visual query builder system | 1.0.0 (March 11 – April 9) | Custom FormWidget (`QueryBuilder`) for building complex conditions, `QueryBuilderParserHelper` class to convert rules to Eloquent queries, plugin support (sortable, filter-description, etc.). |
| **Nano2.QueryBuilder.Reporting (v2)** | Second version of dynamic reporting system | 2.0 (March 10 – April 20) | Support for predefined reports, `DynamicReports` controller with 8 endpoints (run, export to multiple formats, table schema, etc.), pagination support, advanced permissions, improved error handling. |

### 2. Payment Gateways Added (within `Nano.Yepayment`)

| Gateway | Region / Entity | Payment Type | Key Features | Version / Date |
| :--- | :--- | :--- | :--- | :--- |
| **ThawaniPay** | Sultanate of Oman (Thawani Pay) | Redirect (hosted) | Create session, Webhook, query, refund, support for test and production environments. | September 17, 2025 – April 2, 2026 |
| **YottaPay (Sabacash)** | Yemen (YottaGate) | Two-step (OTP) | OAuth 2.0, create transaction, confirm with OTP, query, change password, integrated test UI. | April 9 – 25, 2026 |
| **QasemiPay** | Yemen (Qasemi Islamic Bank) | Two-step (concurrencyStamp) | OAuth 2.0 (password grant), create purchase transaction, confirm, query, support for currencies (YER/SAR). | April 28 – 29, 2026 |
| **JaibPay (Jaib Wallet)** | Yemen (Jaib Pay) | Direct | Instant payment without OTP or redirection, token caching in Cache, status query, interactive test UI. | April 29 – 30, 2026 |

### 3. Major Updates to Existing Plugins

| Plugin | Version / Date | Key Updates |
| :--- | :--- | :--- |
| **Nano.TagsApi** | 1.0.10 (April 16 – 18) | New settings in `config.php` (`include`, `exclude`, `per_page`, etc.), advanced filters in `Categories` (`is_has_products`, `is_has_shops`, `is_has_cateables`), `extendQueryBefore/After` events, support for `is_departments` in `Types`. |
| **Nano.SmsNotify** | 1.0.13 (April 18) | Added 5 new SMS providers: `BdBulkSms`, `ReveSms`, `SmsToday`, `TonkraSms`, `ReleansSms`. |
| **Nano.Coupons** | (April 25 – 27) | Added new coupon type `product_limit` with columns (`product_id`, `max_quantity`, `max_orders`, `period_days`) and `ProductLimit` cart condition to immediately reject addition. |

### 4. Key Improvements and Features in `Nano.FileUpload` (by version)

| Version | Key Features |
| :--- | :--- |
| 1.0.2 – 1.0.5 | Global API control, default file type settings, blacklist for dangerous files, custom exception class, full multilingual support. |
| 1.0.6 – 1.0.7 | Multi-storage support (S3/FTP/Local disks), automatic image processing, event hooks, WebSocket, new columns (`hash`, `meta`, `expires_at`). |
| 1.0.8 – 1.2.0 | Comprehensive testing system (`FileUploadPlusTest`), edit operation support for files, integrated permission system (`validate`, `disabled_operations`). |
| 1.2.1 – 1.2.6 | Permission restructuring, unified temporary key logic (Trait `HasFileUploadsMatchTempKey`), new API endpoints, prevent upload before model exists, fix double deletion issue. |

### 5. Key Improvements and Features in `Nano3.Kyc` (by version)

| Version | Key Features |
| :--- | :--- |
| 1.0.1 – 1.0.6 | Create `nano3_kyc_documents` table, define 28+ document types, `Document` model with multiple traits, `Manager` class for operations, full API and backend admin interface. |
| 1.0.7 – 1.1.0 | KYC assessment methods (`assessKycStatus`, `assessKycStatusByCategory`), `DynamicAddIncludeKyc` behavior to inject data into API. |
| 1.1.1 – 1.2.0 | File upload integration (`FileUploadDocuments`), verified document protection, comprehensive testing system (`KycPlusTest`). |
| 1.2.2 | Alternative groups system (`group:primary_identity`, etc.), improved assessment accuracy. |
| 1.2.3 – 1.2.4 | Advanced caching and atomic locks for quick check methods, new API relationships (`kyc_verification_summary`, category checkers). |
| 1.2.5 – 1.2.8 | Verified document protection with flexible settings, endpoints for categories and types, compatibility with `FormFieldsManager`, fix field merging and validation rules. |

---

## **Details of Achievements and Updates**

### **1. Launch of the Centralized File Upload System (Nano.FileUpload)**  
**Period:** March 15 – April 30, 2026  
**Description:** Development of an independent plugin to manage file upload operations via API across all NanoSoft applications, with an integrated structure for model registration, permission validation, and temporary file management.  

#### **Version 1.0.1 – Initial Launch**  
**Main Components:**  
- **`FileUploadRegistry`**: Central registry for registering models and upload fields with settings (max size, allowed types).  
- **`FileUploadService`**: Service layer for executing upload, delete, and retrieve operations.  
- **`FileUploadUserManager`**: Management of current user and permission checks (backend/frontend/guest).  
- **`FileUploadController`**: RESTful endpoints protected by OAuth.  

**Main Endpoints:**  
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/upload` | POST | Upload a single file (multipart or base64) |
| `/upload-multiple` | POST | Upload multiple files |
| `/delete/{id}` | DELETE | Delete a file |
| `/files` | GET | Retrieve associated files |

#### **Versions 1.0.2 – 1.0.5 (Security and Permissions Improvements)**  
- **Global API control**: Settings to disable upload/delete/get via environment variables.  
- **Default file type settings**: `image`, `audio`, `video`, `file`.  
- **Enhanced temporary key security**: HMAC-SHA256 and prevention of unauthorized access.  
- **Blacklist for dangerous files**: Block PHP, JS, EXE, etc.  
- **Custom exception class**: `FileUploadException` with unique error codes.  
- **Full multilingual support**: API messages in Arabic and English.  

#### **Versions 1.0.6 – 1.0.7 (Multi-Storage and Automatic Processing)**  
- **Multi-storage support**: S3, FTP, Local disks with `disk` column in `system_files`.  
- **Automatic image processing**: Auto-resize and auto-watermark with global settings.  
- **Event hooks**: `beforeUpload`, `afterUpload`, `beforeDelete`, `afterDelete`.  
- **WebSocket notifications**: Optional for real-time notifications.  
- **New columns**: `hash` (SHA256), `meta` (image dimensions), `expires_at` (temporary file expiration).  

#### **Versions 1.0.8 – 1.2.0 (Testing System and Edit Operations)**  
- **Comprehensive testing system**: `FileUploadTest` and `FileUploadPlusTest` with unified output.  
- **Edit operation support**: Replace files in `attachOne` relationships with custom permissions.  
- **Permission validation system**: `validate`, `validateGlobal`, `validateModel`, `validateField` methods.  
- **`disabled_operations` property**: Disable specific operations at model/field level.  

#### **Version 1.2.1 – Permission Restructuring**  
- Added specialized validation methods that throw appropriate exceptions.  
- Improved `attachTempFiles` with unified response structure.  
- Changed HTTP code for `ERR_PERMISSION_DENIED` from 403 to 422.  

#### **Version 1.2.2 – Unified Temporary Key Logic**  
- Created trait `HasFileUploadsMatchTempKey` with `validateAndMatchTempKey` method.  
- Support for strict mode, grace period, and caching.  

#### **Version 1.2.3 – New API Endpoints**  
| Endpoint | Description |
|----------|-------------|
| `GET /models` | List of registered models |
| `GET /model/config` | Settings for a specific model |
| `GET /field/config` | Settings for a specific field |
| `GET /field/constraints` | Field constraints |
| `POST /permissions/check` | Integrated permission check |
| `POST /temp-key/validate` | Validate a temporary key |

#### **Version 1.2.4 – Prevent Upload Before Model Exists**  
- Option `allow_upload_only_when_model_exists` to prevent file uploads for fields that require the model to exist first.  

#### **Version 1.2.5 – API Route Restructuring**  
- Converted `model_class` and `field` parameters from route parameters to query parameters.  
- New clean routes: `/model/config?model_class=...` instead of `/models/{modelClass}`.  

#### **Version 1.2.6 – Fix Double Deletion Issue**  
- Fixed a bug that caused new files to be deleted when uploaded to `AttachOne` relationships.  

**Reference Documentation:** 8 documentation files under `docs/FileUpload/`.

---

### **2. Launch of the Identity Verification System (Nano3.Kyc)**  
**Period:** April 10 – 30, 2026  
**Description:** Development of an integrated identity verification (KYC) system supporting over 28 document types, with dynamic fields, intelligent assessment, and API/backend admin interfaces.  

#### **Versions 1.0.1 – 1.0.6 (Initial Launch)**  
**Main Components:**  
- **`nano3_kyc_documents` table**: Comprehensive columns (documents, dates, relationships, hard copy, JSON data).  
- **`DocumentType` class**: Definition of 28+ document types (ID, address proof, corporate docs, UBO) with properties and dynamic fields.  
- **`Document` model**: Traits `HasScopesModel`, `HasDefault`, `HasOwnerScopes`, `HasRecordsOptions`.  
- **`Manager` class**: Methods `createDocument`, `updateDocument`, `verifyDocument`, `rejectDocument`, `checkDuplicateDocument`.  
- **API Controller**: RESTful endpoints (`/documents`, `/documents/{id}/verify`, `/document-types`, `/document-fields/{type}`).  
- **Backend Controller**: Integrated lists, forms, and filters.  

#### **Versions 1.0.7 – 1.1.0 (KYC Assessment)**  
- **`assessKycStatus` method**: Comprehensive KYC status assessment for a given owner (completion percentage, score, missing documents, recommendations).  
- **`assessKycStatusByCategory` method**: Assessment by specific document category (`primary_id`, `address`).  
- **`DynamicAddIncludeKyc` behavior**: Inject KYC data into API responses via `include=kyc_status,kyc_documents`.  
- **Support for user types**: `backend` and `frontend` as individuals.  

#### **Versions 1.1.1 – 1.2.0 (File Upload and Testing System)**  
- **`FileUploadDocuments` trait**: Integrate file upload (`document_front`, `document_back`, `files`) into API.  
- **Protection of verified documents**: Prevent changing document files after `is_verified = true`.  
- **Comprehensive testing system**: `KycPlusTest` with `GET /tests` endpoint.  

#### **Version 1.2.1 – API Route Restructuring**  
- New routes: `POST /documents/verify/{id?}` instead of `POST /documents/{id}/verify`.  

#### **Version 1.2.2 – Alternative Document Groups**  
- **Alternative groups system**: `group:primary_identity`, `group:address_verification`.  
- **`HasAssessKycStatus` trait**: Separate assessment logic into an independent trait.  
- Improved accuracy of `completion_percentage` and `overall_score`.  

#### **Version 1.2.3 – Performance Improvements for Quick Checks**  
- **`HasValidKycDocuments` trait**: Methods `hasValidKycDocumentByCategory`, `hasValidKycDocument`.  
- **Multi-level caching**: Cache tags and atomic locks to prevent cache stampede.  
- **Automatic cache clearing** when documents change.  

#### **Version 1.2.4 – Improvements to `DynamicAddIncludeKyc`**  
- **New relationships**: `kyc_verification_summary`, `is_verifier_primary_id`, `is_verifier_address`, etc.  
- **Support for `expiring_within_days`** to detect documents expiring soon.  

#### **Version 1.2.5 – Protection of Verified Documents**  
- System to prevent modification of verified documents with flexible settings (`protect_verified`).  
- Automatic calculation of `verification_score` upon verification.  
- Prevent duplicate document type for the same owner.  

#### **Version 1.2.6 – Endpoints for Categories and Types**  
- `GET /document-categories-list` and `GET /document-types-list` with advanced include options.  

#### **Version 1.2.7 – Compatibility with `FormFieldsManager`**  
- Convert field definitions to `FormFieldsManager` format with automatic conversion methods.  
- Improved `/document-fields/{type}` endpoint with resolved options.  

#### **Version 1.2.8 – Fix Field Merging and Validation Rules**  
- Fundamental fix for merging algorithms, support field deletion (`null`).  
- Intelligent merging of validation rules to prevent accumulation.  

**Reference Documentation:** 8 documentation files under `docs/Kyc/`.

---

### **3. Addition of ThawaniPay Payment Gateway (Oman)**  
**Period:** September 17, 2025 – April 2, 2026  
**Description:** Development of a new payment method to support the Thawani Pay gateway in the Sultanate of Oman, the first electronic payment gateway licensed by the Central Bank of Oman (CBO).  

**Developed Components:**  
- **`ThawaniPay` class**: Create payment sessions, handle webhooks, query status, and process refunds.  
- **`Http` class**: cURL communications with proxy support.  
- **`RedirectHelper` class**: Smart redirection supporting deeplinks and JSON.  
- **API Endpoints**: `/thawani/success`, `/thawani/cancel`, `/thawani/retrieve`, `/thawani/test`.  

**Payment Workflow:**  
1. Create a payment session via `POST /checkout/session`.  
2. Redirect user to `https://checkout.thawani.om/pay/{session_id}`.  
3. Handle webhook via `/thawani/success` and update order status.  

**Configuration:**  
```ini
THAWANI_ENABLED=true
THAWANI_MODE=test   # test / live
THAWANI_TEST_API_KEY=...
THAWANI_TEST_SECRET_KEY=...
THAWANI_TEST_PUBLISHABLE_KEY=...
```

---

### **4. Addition of YottaPay Payment Gateway (Sabacash – Yemen)**  
**Period:** April 9 – 25, 2026  
**Description:** Development of a new payment method to support the YottaGate (Sabacash) gateway in Yemen, using a two-step process (create transaction then confirm with OTP).  

**Developed Components:**  
- **`YottaPay` class**: Create transactions, confirm with OTP, query status, change password.  
- **OAuth 2.0 support**: Request token via `/login`.  
- **API Endpoints**: `/yottapay/test-auth`, `/yottapay/test-create-payment`, `/yottapay/test-confirm-payment`, `/yottapay/test-ui`.  

**Payment Workflow:**  
1. Create transaction via `POST /onLinePayment` and store `adjustment.id`.  
2. OTP sent to user.  
3. Confirm payment via `PATCH /onLinePayment` with `id` and `otp`.  

**Configuration:**  
```ini
YOTTAPAY_ENABLED=true
YOTTAPAY_URL=https://api.sabacash.com:49901
YOTTAPAY_USERNAME=...
YOTTAPAY_PASSWORD=...
```

---

### **5. Addition of QasemiPay Payment Gateway (Qasemi Islamic Bank)**  
**Period:** April 28 – 29, 2026  
**Description:** Development of a new payment method to integrate with the Purchase Code Service API of Qasemi Islamic Bank.  

**Developed Components:**  
- **`QasemiPay` class**: Create purchase transactions, confirm with `concurrencyStamp`, query status.  
- **OAuth 2.0 support**: Request token via `grant_type=password`.  
- **API Endpoints**: `/qasemipay/test-auth`, `/qasemipay/test-create-payment`, `/qasemipay/test-confirm-payment`, `/qasemipay/test-ui`.  

**Input Fields:** `purchase_code`, `mobile_number`, `amount`, `currency` (YER/SAR).  

**Configuration:**  
```ini
QASEMIPAY_ENABLED=true
QASEMIPAY_CLIENT_ID=...
QASEMIPAY_CLIENT_SECRET=...
QASEMIPAY_USERNAME=...
QASEMIPAY_PASSWORD=...
```

---

### **6. Addition of JaibPay Payment Gateway (Jaib Wallet – Yemen)**  
**Period:** April 29 – 30, 2026  
**Description:** Development of a new payment method to support the Jaib Pay (Jaib Wallet) gateway in Yemen, using a direct payment system (no OTP or redirection).  

**Developed Components:**  
- **`JaibPay` class**: Execute direct payment via `ExeBuy`, query status via `CheckProgress`.  
- **Token caching in Cache**: `accessToken` and `pinApi` for 86000 seconds.  
- **API Endpoints**: `/jaibpay/test-auth`, `/jaibpay/test-create-payment`, `/jaibpay/test-check-status`, `/jaibpay/test-ui`.  

**Payment Workflow:**  
1. Authenticate via `POST /api/v1/TokenAuth/LogAPI`.  
2. Execute payment directly via `POST /api/v1/BuyOnline/ExeBuy` with `code`, `mobile`, `amount`.  

**Configuration:**  
```ini
JAIBPAY_ENABLED=true
JAIBPAY_URL=https://www.api2.e-jaib.com:5088
JAIBPAY_USERNAME=...
JAIBPAY_PASSWORD=...
JAIBPAY_AGENTCODE=10004
```

---

### **7. Launch of the Visual Query Builder System (Nano2.QueryBuilder)**  
**Period:** March 11 – April 9, 2026  
**Description:** Development of an interactive system to build complex query conditions without writing SQL, based on the `jQuery QueryBuilder` library.  

**Developed Components:**  
- **`QueryBuilder` (FormWidget)**: UI element for defining filters and operators.  
- **`QueryBuilderParserHelper`**: Convert rules to Eloquent queries.  

**Key Properties:**  
| Property | Description |
|----------|-------------|
| `filters` | Definition of available filters (text, number, date, dropdowns) |
| `allow_groups` | Maximum number of nested groups |
| `plugins` | Plugins (sortable, filter-description, bt-tooltip-errors) |
| `isShowParseSqlBtn` | Button to translate rules to SQL |

**Usage Example:**  
```php
$rules = [
    'condition' => 'AND',
    'rules' => [
        ['id' => 'price', 'operator' => 'greater', 'value' => 100],
        ['id' => 'category', 'operator' => 'equal', 'value' => 5]
    ]
];
$query = Product::query();
$finalQuery = QueryBuilderParserHelper::parse($query, $rules);
```

**Reference Documentation:** 2 documentation files under `docs/querybuilder/`.

---

### **8. Launch of the Second Version of the Dynamic Reporting System (Reporting API v2)**  
**Period:** March 10 – April 20, 2026  
**Description:** Development of an integrated system for predefined reports with a comprehensive API.  

**Developed Components:**  
- **`HasStoredReports` trait**: Register and manage predefined reports.  
- **`DynamicReports` controller**: 8 endpoints (list, details, run, export, schema, validate, estimate).  
- **Improvements to `ReportQueryConverter`**: Support dynamic company field (`company_key`).  
- **`ReportUserManager` class**: Manage users and permissions.  

**New Endpoints:**  
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/dynamic-reports` | GET | List of predefined reports |
| `/dynamic-reports/run/{reportId}` | POST | Run a predefined report |
| `/dynamic-reports/execute` | POST | Run a dynamic report |
| `/dynamic-reports/export` | POST | Export results (CSV, Excel, JSON, XML) |
| `/dynamic-reports/schema/{tableName}` | GET | Table schema |

**Reference Documentation:** Comprehensive documentation under `docs/reporting/`.

---

### **9. Launch of the Digital Recharge System (Nano3.TelecomRecharge)**  
**Period:** March 27 – April 23, 2026  
**Description:** Development of a unified platform for interacting with digital recharge service providers in Yemen, with an initial release dedicated to the nflow.tech platform (Master Host).  

**Developed Components:**  
- **`NflowService` class**: Execute all API operations (agent balance, recharge, activate packages, game/gift card recharge).  
- **Specialized traits**: `NflowEndpoints`, `NflowExtendedEndpoints`, `NflowDataEndpoints`.  
- **`NflowController`**: 20+ OAuth-protected endpoints.  

**Main Methods:**  
| Method | Description |
|--------|-------------|
| `getAccountBalance()` | Agent balance |
| `queryBalance($networkNumber, $mobileNumber)` | Query balance for any network |
| `chargeBalanceGeneric($networkNumber, $mobileNumber, $amount)` | Recharge balance for any network |
| `activateOfferGeneric($networkNumber, $mobileNumber, $offerCode)` | Activate a package |
| `getNetworksList()` | List of available networks |

**Shortcut Endpoints:**  
- `GET /yemen-mobile/balance` – Yemen Mobile balance  
- `POST /yemen-mobile/charge` – Yemen Mobile recharge  
- `POST /you/offer/activate` – Activate YOU package  

**Reference Documentation:** 3 documentation files under `docs/TelecomRecharge/`.

---

### **10. Updates to Nano.TagsApi (Version 1.0.10)**  
**Period:** April 16 – 18, 2026  
**Description:** Comprehensive improvements to settings and controllers with support for advanced filters in `Categories`.  

**New Features:**  
- **New settings in `config.php`**: `include`, `exclude`, `per_page`, `is_public` for each section.  
- **Advanced filters in `Categories`**: `is_has_products`, `is_has_shops`, `is_has_cateables`.  
- **New events**: `api.list.extendQueryBefore` and `api.list.extendQuery`.  
- **Support for `is_departments` in `Types`**.  
- **Improved `availablefilters` and `formfields` methods** with `input_data` and `process_data`.  

**New Environment Variables:**  
```ini
NANO_TAGSAPI_CATEGORIES_IS_HAS_PRODUCTS=true
NANO_TAGSAPI_CATEGORIES_IS_HAS_SHOP=true
NANO_TAGSAPI_TYPES_IS_DEPARTMENTS=true
NANO_TAGSAPI_CATEGORIES_PER_PAGE=30
```

---

### **11. Addition of New SMS Providers to Nano.SmsNotify (Version 1.0.13)**  
**Period:** April 18, 2026  
**Description:** Added five new SMS providers to cover the needs of additional regions.  

**New Providers:**  
| Provider | Region | Feature |
|----------|--------|---------|
| `BdBulkSms` | Bangladesh | GreenWeb |
| `ReveSms` | Multiple | Authentication with API Key + Secret Key |
| `SmsToday` | Multiple | Simple endpoint |
| `TonkraSms` | Multiple | Bearer token |
| `ReleansSms` | Global | Bearer token + Sender ID |

---

### **12. Update to Nano.Coupons – "Product Limit" Feature**  
**Period:** April 25 – 27, 2026  
**Description:** Added a new coupon type `product_limit` to restrict product quantities or the number of times a product can be ordered within a period.  

**New Features:**  
- **New columns**: `product_id`, `units_id`, `max_quantity`, `max_orders`, `period_days`.  
- **`ProductLimit` cart condition**: Immediately rejects adding a violating product to the cart.  
- **Respects additional constraints**: Order type, shop, expiration date.  

**Usage Example:**  
```php
// product_limit coupon with max_quantity=5, period_days=30
// Prevents the user from buying more than 5 units of a specific product within 30 days
```

---

## **Conclusion and Recommendations**  

### **Achievements:**  
1. **Unprecedented achievements** – Three major integrated systems (file upload, KYC, digital recharge) and four new payment gateways were launched in one month, reflecting the team's ability to deliver complex projects with high quality.  
2. **Coverage of new markets** – Supporting local payment gateways in Oman (ThawaniPay) and Yemen (YottaPay, QasemiPay, JaibPay) opens new horizons for regional expansion.  
3. **Significant security improvements** – Verified document protection in KYC, blacklist for dangerous files in FileUpload, and atomic cache locks.  
4. **Flexibility and extensibility** – FileUpload system supports multi-storage and automatic image conversion, TelecomRecharge is ready to add new providers.  
5. **Exceptional documentation** – More than 30 detailed documentation files (Arabic/English) with advanced examples, facilitating adoption of these systems in current and future projects.  

### **Recommendations for the Next Phase:**  
1. **Internal awareness** – Conduct training sessions for teams (development, QA, support) on `Nano.FileUpload`, `Nano3.Kyc`, and `Nano3.TelecomRecharge`, as they are the most complex and highest-value systems.  
2. **Integrate systems into existing projects** – Develop a phased plan to upgrade existing projects (Haraj platforms, Taysir, Aqar market) to benefit from the KYC system and the centralized file upload system.  
3. **Monitor performance of new gateways** – Track payment success rates and response times for ThawaniPay, YottaPay, QasemiPay, and JaibPay, adjusting settings as necessary.  
4. **Add graphical interfaces** – Develop graphical admin interfaces for the dynamic reporting system (drag-and-drop report creation) and the KYC system (document type management).  
5. **Expand TelecomRecharge system** – Add support for new providers (Sadad, YouGotaGift) and develop an admin interface to monitor transactions.  
6. **Continue improving visual documentation** – Produce short educational videos for new systems, especially `Nano.FileUpload` and `Nano3.Kyc`, to accelerate adoption.  

---

**Prepared by:** Technical Documentation Team  
**Release Date:** April 30, 2026  
**Classification:** Internal Achievements Report