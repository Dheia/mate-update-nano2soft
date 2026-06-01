# Monthly Progress Report of the NanoSoft Software Team  
## May 2026  

### Introduction  
This report presents a professional and accurate summary of the achievements of the development team at NanoSoft Software Company during the period from May 1 to May 31, 2026. This month was marked by unprecedented major releases and updates, including the launch of **5 complete new APIs** (Nano.StudyyearApi, Nano.SchoolApi, Nano.AbsenceApi, Nano.HomeworkApi, Nano.LocationApi), the **development of 4 new payment gateways** (BasPay, CashPay, FloosakPay, and a major update to JaibPay), along with **substantial improvements in the accounting systems** (Tss.Accounts 1.0.38/1.0.39 and Nano.AccountsApi 1.1.0/1.1.1), a **comprehensive restructuring of the orders modules** (Nano.Orders 2.2.10/2.2.11 and Nano.OrdersApi 1.0.20/1.0.21), the **development of an integrated permission management system** (AccessManager in Nano.AuthApi), **advanced enhancements to visit and proposal behaviors** (ProposalModel, VisitModel), and multimedia support in geographic and accounting systems.

These achievements reflect a qualitative leap in the platform's maturity, with full adherence to backward compatibility standards, comprehensive documentation (over 40 documentation files), and advanced practical examples.

---

## Executive Summary  
During May 2026, the team completed **more than 25 major updates**, distributed across eight strategic axes:

1. **Launch of 5 integrated APIs** – covering academic years, schools, attendance, homework, and geographic location, all built according to the latest standards.
2. **Addition of 4 new payment gateways** – BasPay, CashPay (electronic cash payment – OTP), FloosakPay (Floosak Wallet), and a comprehensive update to JaibPay (Jaib Wallet).
3. **Comprehensive development of the accounting system** – support for multi‑party journal entries, exchange rate differences, advanced validation (Tss.Accounts 1.0.38), then multimedia support for all vouchers (1.0.39) with full API integration (Nano.AccountsApi 1.1.0/1.1.1).
4. **Restructuring of the orders system** – adding product owner filtering scopes, a status update function with graduated permissions, new relationships (delivery vehicle, country, city, airports), and expanded report settings.
5. **Launch of a centralised permission management system** – the advanced AccessManager class supporting scopes (company, department, state, own, children) and dynamic student/parent resolution, with advanced filters and centralised configuration.
6. **Development of generic behaviours** – advanced scopes for VisitModel and ProposalModel with statistics and computed columns.
7. **Multimedia support** – adding images and files to countries, states, and directorates (Nano.LocationApi 1.1.0) and to all accounting models (UnifiedMorphClass).
8. **Supporting improvements** – updating Tss.Homework with a complete Feedback module, updating Tss.Student by restructuring getRecords functions and integrating AccessManager and AdvancedQueryHelper, and updating Nano.UserPlus with dynamic account types and new user group columns.

All updates were made while maintaining backward compatibility, adding comprehensive documentation in both Arabic and English, and providing interactive testing interfaces.

---

## Summary Tables of Key Achievements

### 1. Newly Launched Core Systems and Plugins

| System / Plugin | Description | Version / Date | Key Features |
| :--- | :--- | :--- | :--- |
| **Nano.StudyyearApi** | API for managing academic years, semesters, and months | 1.0.0 (6 – 7 May) | 3 controllers (Periods, Semsters, Months), multi‑level permission system, data transformers, support for default values (primary academic year). |
| **Nano.SchoolApi** | API for basic school data (14 resources) | 1.0.0 (8 – 10 May) | Classes, stages, subjects, groups, distributions, fees, documents, grades, per‑resource permissions, comprehensive documentation. |
| **Nano.AbsenceApi** | API for attendance, absence, and roster records | 1.0.0 then 1.1.0 (11 – 12 May then 26 – 27 May) | CRUD for absence records, create/update via API, integration with AccessManager, advanced filters. |
| **Nano.HomeworkApi** | API for managing homework, submissions, and classifications | 1.0.0 (27 – 30 May) | 5 controllers, support for general assignments (`student_id = '*'`), flexible permissions, dynamic student/parent resolver. |
| **Nano.LocationApi (version 1.1.0)** | Multimedia support in geographic location | 1.1.0 (20 – 21 May) | DynamicAddMediaIncludes behavior, embedding images and files in API responses, metadata control. |
| **Nano.AuthApi (versions 1.0.19–1.0.21)** | Development of AccessManager and advanced permissions | 1.0.19 (15-16 May), 1.0.20 (24-25 May), 1.0.21 (25-26 May) | Endpoints for field options, UserGroups controller, AccessManager with dynamic resolver, advanced filters, and fallback support. |

### 2. Payment Gateways Added and Updated (within Nano.Yepayment)

| Gateway | Region / Provider | Payment Type | Key Features | Version / Date |
| :--- | :--- | :--- | :--- | :--- |
| **BasPay** | Yemen (Bas platform) | Two‑Step | OAuth2, AES‑256‑CBC encrypted signature, create then confirm transaction, integrated test UI. | 6 – 18 May |
| **CashPay** | Yemen (Electronic Cash Payment – OTP) | Two‑Step with OTP | Password encryption in header, InitPayment, ConfirmPayment, password change, test UI. | 20 – 22 May |
| **FloosakPay** | Yemen (Floosak Wallet) | Two‑Step with OTP | OAuth2, idempotency, PaymentLog in the first stage, refund, test UI. | 18 – 24 May |
| **JaibPay (comprehensive update)** | Yemen (Jaib Wallet) | Direct | Fixed authentication, refund function, improved response handling, fixed encrypted settings, test UI. | 4 – 10 May |

### 3. Major Updates to Existing Plugins

| Plugin | Version / Date | Key Updates |
| :--- | :--- | :--- |
| **Tss.Accounts** | 1.0.38 (17 Feb – 1 May), then 1.0.39 (29 – 31 May) | 1.0.38: TransactionsMultiHelper trait (multi‑party entries, exchange differences, advanced validation). 1.0.39: multimedia support for all accounting models, unified MorphClass (UnifiedMorphClass). |
| **Nano.AccountsApi** | 1.1.0 (20 Mar – 5 May), then 1.1.1 (29 – 31 May) | 1.1.0: BaseDocumentController, added 5 new document types (BondsDays, PayReceipts, DebitNotes, CreditNotes, Transfers) and CatchReceiptsV2. 1.1.1: DynamicAddMediaIncludes behavior for media embedding in API. |
| **Nano.Orders** | 2.2.10 (6 – 8 May), then 2.2.11 (26 – 29 May) | 2.2.10: HasProductOwnerScopes (filter by product owner), updateOrderStatusAdvanced (graduated permissions). 2.2.11: new relationships (delivery_vehicle_type, delivery_car, country, state, departuredestination, arrivaldestination), expanded order report settings. |
| **Nano.OrdersApi** | 1.0.20 (6 – 8 May), then 1.0.21 (26 – 29 May) | 1.0.20: update‑status API endpoint. 1.0.21: support for new relationships in OrderTransformer. |
| **Nano.UserPlus** | 1.1.7 (13 – 14 May) | Dynamic account types, compatibility with RainLab.User 2.x (first_name/last_name), new user_groups columns (is_new_user_default, is_active, sort_order), expanded group management interface. |
| **Tss.Student** | 1.0.13 (24 – 26 May) | Restructured getRecords functions, added StudentRecordsHelper, integrated AccessManager and AdvancedQueryHelper, linking functions between user and student/parent. |
| **Tss.Homework** | 1.0.9 (27 – 28 May) | Added complete Feedback module (submissions and evaluations) with model, admin interface, permissions, and sidebar menu. |
| **Nano.Coupons** | 2.1.1 (2 – 8 May) | Restructured "product limit" condition in ProductLimitValidator class, support for advanced scenarios (limiting order count without a specific product), performance improvement (Cache). |
| **Nano.Location** | 1.0.16 (20 – 21 May) | Added media relationships (attachOne/attachMany) for country, state, directorate, and extended the backend with media fields and columns. |

### 4. Development of Generic Behaviors

| Behavior | Version / Date | Key New Scopes and Functions |
| :--- | :--- | :--- |
| **ProposalModel (Proposals)** | 19 – 20 May | Scopes: count of proposals, latest/oldest proposal, filtering by user (target/sender), added is_proposed_by_user column, advanced scope scopeWhereHasProposalAdvanced, statistical functions. |
| **VisitModel (Visits)** | 20 – 21 May | Scopes: count of visits, sum of visits, sum of views, latest/oldest visit, filtering by user, is_visited_by_user column, statistical functions (getTotalVisits, getVisitorsUsers). |

---

## Detailed Achievements and Updates

### 1. Launch of Integrated APIs

#### 1.1. Nano.StudyyearApi (version 1.0.0)  
**Period:** 6 – 7 May  
**Description:** Creation of a RESTful API for managing academic years, semesters, and months, based on the Tss.Studyyear plugin.  
**Controllers:**  
- `Periods` (academic years)  
- `Semsters` (semesters)  
- `Months` (academic months)  
**Features:**  
- Per‑resource permissions (backend/frontend) via config.php.  
- Data transformers with relationship embedding.  
- Support for default values (default academic year via Period::getPrimary()).  
- Support for caching and text search.  
**Endpoints:**  
- `GET /api/v1/studyyear/periods` , `GET /api/v1/studyyear/semsters` , `GET /api/v1/studyyear/months`  
- `GET /api/v1/studyyear/periods/{id}` and their counterparts.

#### 1.2. Nano.SchoolApi (version 1.0.0)  
**Period:** 8 – 10 May  
**Description:** Comprehensive API for all entities of the Tss.School plugin (14 resources).  
**Controllers:**  
`Classes`, `Stages`, `Subjects`, `Groups`, `ClassGroups`, `ClassSubjects`, `TeacherSubjects`, `ClassCostCenters`, `FeesTypes`, `DocumentTypes`, `OutSchools`, `StudentDocuments`, `StudentsFees`, `LastStudentMarks`.  
**Features:**  
- Separate permissions for each resource.  
- Support for default academic year for resources associated with it.  
- Dynamic scopeExclude scope.  
- Comprehensive documentation with examples.

#### 1.3. Nano.AbsenceApi (version 1.0.0 then 1.1.0)  
**Period:** 11 – 12 May (version 1.0.0), then 26 – 27 May (version 1.1.0)  
**Description:** API for managing attendance, absence, and roster records.  
**Controllers:** `Absences`, `Files`, `ClassTypes`, `ClassTypeCompanies`.  
**Version 1.0.0 features:**  
- Support for store and update operations on absence records.  
- Permission checks for each operation.  
- Smart default values (companys_id, departments_id, year_id).  
**Version 1.1.0 features:**  
- Complete restructuring according to Nano-Api-SKILL.md standards.  
- Use of AccessManager::checkByResource and checkWithFallback.  
- Support for advanced filters (is_or, is_not, is_force, is_or_null).  
- Dynamic student/parent resolver (frontend_resolver).  
- Unified getRecords function with events and caching.

#### 1.4. Nano.HomeworkApi (version 1.0.0)  
**Period:** 27 – 30 May  
**Description:** Fully integrated API for managing homework, submissions, and classifications.  
**Controllers:** `Homeworks`, `Feedbacks`, `Categories`, `ClassTypes`, `ClassTypeCompanies`.  
**Features:**  
- Support for general assignments for whole classes (`student_id = '*'`) or whole groups (`group_id = '*'`).  
- Multi‑level permissions (backend/frontend/guest) with centralised settings.  
- Dynamic resolver for `student_id` and `record_id` for students and parents.  
- Unified getRecords function with advanced filters and events.  
- Data transformers (HomeworkTransformer, FeedbackTransformer, etc.).  
- Full CRUD operations for homeworks, and create/update for Feedback.

#### 1.5. Nano.LocationApi (version 1.1.0)  
**Period:** 20 – 21 May  
**Description:** Added multimedia support to the Location API.  
**Features:**  
- `DynamicAddMediaIncludes` behavior adds dynamic includes (image, images, files, videos, audios, book_intro) to CountryTransformer, StateTransformer, DirectorateTransformer.  
- Metadata control via global `is_mate_data_media` or per‑include parameters.  
- Added media columns and fields in the backend for Country, State, Directorate.  
- Use of `UnifiedMorphClass` trait to unify polymorphic type for files (optional, applied in accounting).

---

### 2. New and Updated Payment Gateways

#### 2.1. BasPay (Bas platform)  
**Period:** 6 – 18 May  
**Description:** Payment gateway for Yemen operating in two‑step mode with AES‑256‑CBC encrypted signature.  
**Components:** BasPay class extending PaymentProvider, process (create transaction), complete (confirm), getAuthToken (OAuth2), generateSignature (AES‑256‑CBC), checkTransactionStatus.  
**Test endpoints:** `/baspay/test-auth`, `/baspay/test-create-payment`, `/baspay/test-ui`.  
**Settings:** baspay_url, client_id, client_secret, app_id, merchant_key, iv.

#### 2.2. CashPay (Electronic Cash Payment – OTP)  
**Period:** 20 – 22 May  
**Description:** OTP‑based gateway, two‑step mode with password encryption in header.  
**Components:** CashPay class, process (InitPayment), complete (ConfirmPayment), operationStatus, changePassword, encryptAes256Cbc.  
**Test endpoints:** `/cashpay/test-auth`, `/cashpay/test-create-payment`, `/cashpay/test-confirm-payment`, `/cashpay/test-ui`.  
**Settings:** cashpay_url, username, password (encrypted), encryption_key, sp_id.

#### 2.3. FloosakPay (Floosak Wallet)  
**Period:** 18 – 24 May  
**Description:** Yemen payment gateway, two‑step mode with OTP, idempotency, token caching.  
**Components:** FloosakPay class, process (sendPayment), complete (confirmPayment), reverse (refund), getAuthToken, checkTransactionStatus.  
**Features:**  
- Check for existing transaction to prevent duplication.  
- PaymentLog with 'initiated' status in first stage.  
- Integrated test UI.  
**Test endpoints:** `/floosakpay/test-auth`, `/floosakpay/test-create-payment`, `/floosakpay/test-confirm-payment`, `/floosakpay/test-ui`.

#### 2.4. JaibPay (comprehensive update)  
**Period:** 4 – 10 May  
**Description:** Major improvements to the JaibPay gateway (Jaib Wallet) launched in April.  
**Fixes and improvements:**  
- normalizeApiResponse function to unify response handling.  
- Fixed encrypted settings storage in PaymentGatewaySettings (overriding __set).  
- Added refund function.  
- testPortConnection function.  
- Improved error messages (e.g., "Purchase code is incorrect").  
- Updated test UI.

---

### 3. Advanced Accounting Updates

#### 3.1. Tss.Accounts version 1.0.38  
**Period:** 17 February – 1 May (included in May report because it completed in May)  
**Key features:**  
- `TransactionsMultiHelper` trait for creating multi‑party journal entries (3+ parties).  
- Exchange difference functions: `checkExchangeDifferences`, `createExchangeDifferenceEntry`, `handleExchangeDifferences`.  
- `testJournalEntry` function to test an entry before creation.  
- Comprehensive configuration controls (amount limits, balance checking, dynamic account type rules, paper_id management).  
- Integration with `TransactionsHelper` to make trait functions available.

#### 3.2. Nano.AccountsApi version 1.1.0  
**Period:** 20 March – 5 May  
**Features:**  
- `BaseDocumentController` abstract class to unify CRUD logic for financial documents.  
- Added 5 new controllers: `BondsDays` (journal entries), `PayReceipts` (payment receipts), `DebitNotes` (debit notes), `CreditNotes` (credit notes), `Transfers` (transfers).  
- Improved version `CatchReceiptsV2` (catch receipts).  
- Transformers for each type.  
- Expanded config.php settings and routes.

#### 3.3. Tss.Accounts version 1.0.39 and Nano.AccountsApi 1.1.1 (multimedia support)  
**Period:** 29 – 31 May  
**Features:**  
- Added `attachOne` and `attachMany` relationships (image, images, files, videos, audios, book_intro) to all accounting models (TransactionHeader, CatchReceipt, PayReceipt, BondsDay, Transfer, etc.) using the `UnifiedMorphClass` trait to unify the polymorphic type.  
- Backend expansion: columns (image, images, files) in lists, and an "Attachments" tab in forms.  
- `DynamicAddMediaIncludes` behavior in Nano.AccountsApi to embed media in API responses via `include=image,images,files`.  
- Support for `is_mate_data_media` to control metadata return.

---

### 4. Orders System Updates (Nano.Orders & Nano.OrdersApi)

#### 4.1. Nano.Orders version 2.2.10  
**Period:** 6 – 8 May  
**Features:**  
- `HasProductOwnerScopes` trait in Order model:  
  - Advanced filtering scopes by product owner (whereHasProductsByOwner, hasProductsByOwner, doesntHaveProductsByOwner, etc.).  
  - Support for count, not, conditions, itemConditions.  
  - Added products_owner_count column and ordering by it.  
- `updateOrderStatusAdvanced` function (within StepStatus trait):  
  - Order status update with graduated permissions (admin, owner, delivery).  
  - Customisable transition rules via config.  
  - Support for because_cancel, delivery_because_cancel.  
  - Options: is_save, is_event, is_logs, skip_permission, admin_override.  
- Helper functions in OrderHelper: `getDeliveryByUser`, `getUserByDelivery`.  
- Product owner filtering support in `OrderHelper::getOrdersRecords`.

#### 4.2. Nano.OrdersApi version 1.0.20  
**Period:** 6 – 8 May  
**Features:**  
- Endpoint `POST /api/v1/orders/orders/update-status` to update order status.  
- Integration with `updateOrderStatusAdvanced`, supporting all its options.

#### 4.3. Nano.Orders version 2.2.11  
**Period:** 26 – 29 May  
**Features:**  
- New relationships in Order model:  
  - `delivery_vehicle_type` and `delivery_vehicle_ref_type` (delivery vehicle type).  
  - `delivery_car` (delivery vehicle).  
  - `country` and `state`.  
  - `departuredestination` and `arrivaldestination` (departure and arrival airports).  
- Expanded order report settings (OrdersSetting) with new sections: detailed customer data, booking & trip data, load data, etc.  
- Rewritten `ReportsOrders` report template to display all fields in an organised manner with control via settings.

#### 4.4. Nano.OrdersApi version 1.0.21  
**Period:** 26 – 29 May  
**Features:**  
- Added new relationships to `availableIncludes` in OrderTransformer.  
- Separate include methods: `includeDeliveryVehicleType`, `includeDeliveryCar`, `includeCountry`, `includeState`, `includeDeparturedestination`, `includeArrivaldestination`.

---

### 5. Development of Centralised Permission Management System (AccessManager)

#### 5.1. Nano.AuthApi version 1.0.19 (15 – 16 May)  
**Features:**  
- Endpoints to fetch field options:  
  - `GET /api/v1/user/options` with `fields` parameter.  
  - `GET /api/v1/user/frontend-options` and `backend-options`.  
- `UserGroups` controller (list, view) with support for filters (is_active, is_new_user_default, search).  
- Updated UserGroupTransformer to include new columns (is_new_user_default, is_active, sort_order).  
- Migration to add those columns to the user_groups table.

#### 5.2. Nano.AuthApi version 1.0.20 (24 – 25 May)  
**Features:**  
- `AccessManager` class (Singleton) for centralised permission and access management.  
**Core functions:**  
  - `check($operation, $config, $user)` returns an array (allowed, scope, message, applied_scope_details).  
  - Scope application functions: `applyAccessScope`, `applyCompanyScope`, `applyDepartmentScope`, `applyStateScope`, `applyCreatedByScope`, `applyChildrenScope`.  
  - Helper functions: `canAccessAllCompanies`, `canAccessAllDepartments`, `canAccessAllStates`.  
  - Support for user types: backend, frontend, guest.  
  - Support for scopes: all, own, created_by, company, department, state, children.  
  - Integration with `BasicHelper::checkAccessAllCompanys` and others.

#### 5.3. Nano.AuthApi version 1.0.21 (25 – 26 May)  
**Advanced features:**  
- Dynamic student/parent resolver (`resolveDynamicFrontendOptions`) automatically populates `student_id` and `record_id` based on `ref_type` and config‑defined rules.  
- Advanced filters control (`advanced_filters`): specify which fields allow `is_or`, `is_not`, `is_or_null` per user type.  
- Simplified methods: `checkByResource`, `getOperationByConfig`, `checkWithFallback`.  
- Centralised permission configuration loading from config.php with default merging.  
- Fallback support between operations (e.g., show falls back to list).

---

### 6. Development of Generic Behaviors (ProposalModel and VisitModel)

#### 6.1. ProposalModel (Proposals and Reports) – 19 – 20 May  
**New scopes:**  
- `scopeAddCountProposals` / `scopeSortByCountProposals` / `scopeWithCountProposals`.  
- `scopeAddLatestProposal` / `scopeSortByLatestProposal` / `scopeWithLatestProposal` (and oldest).  
- `scopeProposedByUser` / `scopeNotProposedByUser` (object as target).  
- `scopeProposedToUser` / `scopeNotProposedToUser` (object as sender).  
- `scopeHasProposals` / `scopeHasNoProposals`.  
- `scopeWithIsProposedByUser` / `scopeWithIsProposedToUser`.  
- `scopeWhereHasProposalAdvanced` (advanced scope supporting side, type, conditions, useUserConditions, withTrashed).  
- Statistical functions: `getTotalProposals`, `getProposalsCountByType`, `getProposersUsers`.

#### 6.2. VisitModel (Visits and Views) – 20 – 21 May  
**New scopes:**  
- `scopeAddCountVisits` / `scopeSortByCountVisits` / `scopeWithCountVisits`.  
- `scopeAddSumVisits` / `scopeSortBySumVisits` / `scopeWithSumVisits`.  
- `scopeAddSumViews` / `scopeSortBySumViews` / `scopeWithSumViews`.  
- `scopeAddLatestVisit` / `scopeSortByLatestVisit` / `scopeWithLatestVisit` (and oldest).  
- `scopeVisitedByUser` / `scopeNotVisitedByUser`.  
- `scopeHasVisits` / `scopeHasNoVisits`.  
- `scopeWithIsVisitedByUser`.  
- Statistical functions: `getTotalVisits`, `getTotalViews`, `getVisitsCountByType`, `getVisitorsUsers`.  
- Support for `withTrashed` to include soft‑deleted visits.

---

### 7. Supporting Updates and Other Improvements

#### 7.1. Tss.Student (version 1.0.13) – 24 – 26 May  
**Features:**  
- Restructured `getRecords` functions in the new `StudentRecordsHelper` class.  
- `getParentRecords` (for parents) and `getStudentRecordRecords` (for academic records).  
- Integrated `AccessManager` to automatically apply permission scopes.  
- Integrated `AdvancedQueryHelper` for dynamic filtering with support for is_or, is_not, is_force, is_or_null.  
- Linking functions: `getStudentByUser`, `getUserByStudent`, `getMparentByUser`, `getUserByMparent`, `getStudentsByMparent`, `getStudentsByUser`, `getCurrentStudentRecord`.  
- Support for caching, events, and unified response structure.

#### 7.2. Tss.Homework (version 1.0.9) – 27 – 28 May  
**Features:**  
- Added complete `Feedback` module (submissions and evaluations):  
  - Feedback model with relationships (homework, student, student_record, class, group, subject, teacher).  
  - Admin interface (Feedbacks controller, lists, input form with tabs).  
  - Custom permissions (access_all, access, add, edit, delete).  
  - Sidebar menu under the main homework menu.  
  - Full translations.  
- No new migrations (the tss_homework_feedbacks table already existed since 1.0.7).

#### 7.3. Nano.UserPlus (version 1.1.7) – 13 – 14 May  
**Features:**  
- Dynamic account types (`ref_type`) via config.php (ref_type.delivery, ref_type.department, etc.).  
- Compatibility handler for RainLab.User 2.x (RainlabUser2CompatibilityHandler) to map first_name/last_name with name/surname.  
- New columns for the user_groups table: `is_new_user_default` (default for new users), `is_active`, `sort_order`.  
- Expanded user group management interface (list columns and form fields).  
- Translations for the new keys.

#### 7.4. Nano.Coupons (version 2.1.1) – 2 – 8 May  
**Features:**  
- Independent `ProductLimitValidator` class for checking product limit constraints.  
- Support for new scenarios: limiting the number of orders for a specific order type (without a specific product).  
- Performance improvement via caching of constraints.  
- Detailed error messages with parameters (limit, current count, period, next allowed order date).  
- Admin interface update: product_id as recordfinder, units_id depends on product_id.  
- Scopes `scopeOnProductLimit` and `scopeNotOnProductLimit`.

---

## Conclusion and Recommendations

### Achievements  
1. **Unprecedented quantity and quality** – 5 complete APIs, 4 payment gateways, and major updates to accounting, orders, and permissions systems, demonstrating the team's ability to deliver high‑quality work in record time.  
2. **Architectural unification** – Adopting `Nano-Api-SKILL.md` standards in all new APIs (AccessManager, AdvancedQueryHelper, unified getRecords structure) ensures ease of maintenance and extensibility.  
3. **Coverage of new sectors** – Support for multiple local Yemeni payment gateways (BasPay, CashPay, FloosakPay) opens new markets; previous support for ThawaniPay completes regional coverage.  
4. **Major security improvement** – The launch of AccessManager provides fine‑grained control over permissions and access scopes, complementing the KYC document protection (April) and sensitive data redaction (February).  
5. **Exceptional documentation** – Over 40 documentation files (Arabic/English) were produced during May, with advanced and comprehensive practical examples.  
6. **High flexibility** – Ability to customise advanced filters (is_or, is_not) and permissions (AccessManager) solely through config.php files, without code modification.

### Recommendations for the next phase  
1. **Internal and external awareness** – Organise workshops and presentations for teams (development, QA, technical support) on the new APIs, AccessManager, and the new payment gateways, as these systems represent a qualitative leap in the platform.  
2. **Integration into existing projects** – Develop a phased plan to upgrade projects (Haraj, Taysseer, Souq Aqar, the educational platform) to benefit from the schools, attendance, and homework APIs, as well as the central file upload system and KYC.  
3. **Monitor performance of the new payment gateways** – Track success rates and response times for BasPay, CashPay, FloosakPay in production, and produce periodic reports.  
4. **Complete multimedia support** – Activate the `videos`, `audios`, `book_intro` includes in DynamicAddMediaIncludes for accounting and location, and add the appropriate backend interfaces.  
5. **Continue improving visual documentation** – Produce short tutorial videos (2‑5 minutes) for each new API and for AccessManager to accelerate adoption by external developers.  
6. **Prepare a performance testing plan** – Especially for advanced getRecords queries with filters, events, and caching, to ensure query efficiency when handling thousands of records.

---

**Report prepared by:** Technical Documentation Team  
**Release date:** 31 May 2026  
**Classification:** Internal progress report