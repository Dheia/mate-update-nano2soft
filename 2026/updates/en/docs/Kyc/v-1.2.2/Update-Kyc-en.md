## 2026-04-10 – 2026-04-18

**Official Launch of the `Nano3.Kyc` Plugin – A Smart and Integrated Identity Verification (KYC) System for NanoSoft App**

---

### 🚀 Introducing the Package: Why `Nano3.Kyc`?

In the fast‑paced world of digital business, verifying customer and supplier identity (KYC) has become a fundamental requirement for regulatory compliance and building trust. Whether you run an e‑commerce platform, a digital financial institution, or an online marketplace, the need for a flexible and secure system to collect and manage identity documents is indispensable.

Enter the **`Nano3.Kyc`** plugin – the integrated solution from **Nano2Soft**, designed specifically for the **NanoSoft App** environment. It is not just a tool for storing document images; it is a **smart platform for managing the entire identity verification lifecycle**. From defining the required document types (with over 28 ready‑to‑use types) to managing dynamic fields, validity checking, physical copy tracking, and providing complete APIs and backend admin interfaces.

`Nano3.Kyc` is built on the principles of **flexibility and security**, allowing it to adapt to any business model or changing regulatory requirements without complex code modifications.

---

### 🎯 Why `Nano3.Kyc` Was Developed (The Need and the Problem)

Before `Nano3.Kyc`, developers at NanoSoft faced recurring challenges when implementing KYC systems:

| Challenge | `Nano3.Kyc` Solution |
| :--- | :--- |
| **Lack of a unified structure** | The plugin provides a flexible data model (`Document`) designed to accommodate **any document type**, with indexed columns for common fields and a JSON field for specialised fields. |
| **Repeatedly writing validation code** | The `DocumentType` class provides a centralised definition for each document type with ready‑to‑use validation rules (file size, type, expiry date, etc.). |
| **Difficulty managing physical documents** | The plugin includes a complete system for tracking **physical copies received and returned** (`is_physical_submitted`, `physical_received_at`, ...). |
| **Checking document validity over time** | Built‑in functions to check document validity (`isDocumentAcceptable`, `isDocumentExpiring`) that automatically alert when a document expires. |
| **Need for custom admin interfaces** | The plugin provides a **ready‑made backend interface** (lists, forms, filters) and a **complete RESTful API** for integration with mobile apps. |
| **Multiple user types and permissions** | The plugin supports polymorphic relations (`owner`, `verifier`, `subject`), allowing documents to be linked to their owner (individual/corporate) and the verifier, with fine‑grained permissions. |

In short, `Nano3.Kyc` was born to end fragmentation and provide **one centralised solution** for all KYC requirements in NanoSoft projects.

---

### 💎 Core Components and Capabilities of the Package

The `Nano3.Kyc` package consists of several integrated components that work in harmony:

| Component | Function |
| :--- | :--- |
| **`DocumentType`** | **The brain of the plugin**. A central class that defines over **28 ready‑to‑use document types** (passport, ID, bill, business license, etc.) distributed across 6 categories. Contains field schemas and validation rules for each type, and supports loading settings from a `config` file. |
| **`Manager`** | **The operations coordinator**. Provides a unified API for creating, updating, deleting, approving, and rejecting documents. Supports transactions, events, and a testing mode (`is_test_create`). |
| **`Document` (Model)** | **The data model**. The `nano3_kyc_documents` table is designed as the backbone for storing all documents. Uses a rich set of traits to provide search scopes, dropdown options, caching, and advanced filters (`getRecords`). |
| **`Documents` (API Controller)** | **The integration gateway**. Provides OAuth‑protected RESTful endpoints for external applications to interact with the system. All responses follow the unified `Nano.API` structure. |
| **`Documents` (Backend Controller)** | **The administrative user interface**. Provides complete management lists and forms inside the NanoSoft dashboard, with support for advanced filters, search, and sorting. |
| **`DocumentTransformer`** | **The data formatter**. Responsible for transforming model data into JSON compatible with the API, with support for including relations and formatting fields. |
| **Translation files (`lang.php`)** | **Localised user experience**. All texts and notifications are translatable (Arabic is fully provided), ensuring a smooth experience for users. |

---

### ✨ Key Features of the First Release (1.0.0)

This launch represents the culmination of intensive development work and includes a rich set of features:

#### 1. Centralised Definition of Over 28 Document Types

- **6 main categories**: Primary ID, Secondary ID, Proof of Address, Corporate Documents, Ultimate Beneficial Owner (UBO), Additional Verification.
- **Advanced properties per type**: whether it expires (`has_expiry`), maximum age in days (`max_age_days`), trust weight (`verification_weight`), allowed MIME types (`allowed_mime_types`), allowed entities (`allowed_for_entity`), and more.
- **Fully customisable**: any property can be modified, or new types can be added entirely, via the `config/nano3/kyc/document_types.php` file.

#### 2. Dynamic and Intelligent Fields

- **Automatic form generation**: the `getFieldsSchema()` function returns the complete field schema for any document type, making it easy to build dynamic input interfaces.
- **Hybrid storage**: common indexed fields (e.g. `document_number`, `full_name`, `expiry_date`) are stored in separate columns for performance, while specialised fields (e.g. `passport_type`, `religion`) are stored in a single JSON field (`fields_data`).
- **Automatic data mapping**: `mapInputToDocumentData` and `mapDocumentToOutput` simplify working with this hybrid structure.

#### 3. Complete Document Lifecycle

- **Multiple statuses**: `pending`, `verified`, `rejected`, `expired`.
- **Verification operations**: `verifyDocument` updates the status and records the verifier, verification date, and trust score.
- **Rejection with reason**: ability to reject a document and record the reason in `metadata`.

#### 4. Advanced Hard Copy Tracking

- **Precise tracking**: dedicated fields to record when a physical copy is received (`physical_received_at`), returned (`physical_returned_at`), and its type (original/copy).
- **Notes**: `physical_notes` field to document any additional information.

#### 5. Rich RESTful API

- **Comprehensive endpoints**: create, read, update, delete, verify, reject, restore, and statistics.
- **Advanced filtering**: parameters like `document_type`, `status`, `owner_id`, `issue_date` can be passed to filter results.
- **Unified formatting**: all responses follow a uniform structure that is easy to process.

#### 6. Easy‑to‑Use Backend Interface

- **Customisable document list**: sortable and searchable columns, quick filters, and bulk actions.
- **Organised input form**: divided into logical tabs (Basic, Owner, Verification, Physical Copy, Files, Publishing).
- **Smart dropdowns**: dependent dropdowns (e.g. `owner_id` depends on `owner_type`).

---

### 🔧 Looking to the Future

This launch is just the beginning. Nano2Soft is committed to continuing the development of `Nano3.Kyc` to include:

- **Optical Character Recognition (OCR)**: automatic data extraction from document images.
- **Automated document verification**: integration with external services to verify document authenticity.
- **Analytics dashboards**: advanced statistics on document statuses and verification processes.
- **Customisable document templates**: ability to add custom fields via the user interface without programming.

---

### 📚 Reference Documentation

- [General Plugin Documentation](./docs/Kyc/Docs-Nano3-Kyc-en.md)
- [DocumentType Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [Manager Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [Document Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)

---

**Nano3.Kyc** – Developed with love by the **Nano2Soft** team 💙
