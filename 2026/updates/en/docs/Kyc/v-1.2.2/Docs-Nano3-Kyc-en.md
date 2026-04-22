# Nano3.Kyc Plugin Documentation

## Overview

**Nano3.Kyc** is an integrated software solution for managing Know Your Customer (KYC) verification processes within the NanoSoft App platform. The add-on provides a flexible and secure system for collecting personal and business identity documents, verifying them, and tracking their status according to best regulatory compliance practices.

The add-on was developed by **Nano2Soft** to meet the needs of financial institutions, e-commerce platforms, and digital services that require user and customer identity verification.

### Plugin Goals

- **Centralized document management**: Unify the storage and management of all types of KYC documents.
- **High flexibility**: Support unlimited document types with customizable dynamic fields.
- **Verification and approval**: Provide a complete document lifecycle (pending → verified → rejected).
- **Seamless integration**: Ready-to-use API for integration with mobile and web applications.
- **Compliance and security**: Track expiration dates, hard copies, and audit logs.

---

## Plugin Structure and Components

The add-on is designed according to NanoSoft App standards and the `Nano3` structure, consisting of the following main components:

```
plugins/nano3/kyc/
├── classes/
│   ├── Manager.php              # Operations manager (create, update, delete, verify)
│   └── DocumentType.php         # Document type definitions and properties
├── config/
│   └── document_types.php       # Document type customization file
├── controllers/
│   └── Documents.php            # Backend controller
├── models/
│   └── Document.php             # Main document model
│   └── document/                # Model-related traits
├── updates/
│   ├── version.yaml             # Version and update log
│   └── create_table_nano3_kyc_documents.php
├── lang/
│   └── ar/
│       └── lang.php             # Arabic translation file
├── apicontrollers/
│   └── Documents.php            # RESTful API controller
├── transformers/
│   └── DocumentTransformer.php  # API data transformer
└── routes.php                   # API routes
```

### Core Components

| Component | Description |
| :--- | :--- |
| `Manager` | The class responsible for all document operations (create, update, delete, verify, reject) with support for testing (`is_test_create`), duplicate checking, and events. |
| `DocumentType` | A central class defining over 28 document types, their properties, fields, validation rules, and trust weights. Supports loading settings from `config` files. |
| `Document` | Eloquent model representing the `nano3_kyc_documents` table. Uses several traits to add search scopes, list options, and caching. |
| `Documents (APIController)` | API controller providing RESTful endpoints for document management and verification. |
| `Documents (Controller)` | Backend controller for managing documents from the NanoSoft App control panel. |

---

## Supported Document Types

The add-on supports **28+ document types** distributed across **6 main categories**, with the ability to easily add new types via the configuration file.

| Category | Types |
| :--- | :--- |
| **Primary Identity** | Passport, National ID Card, Driver's License, Residence Permit, Refugee Document |
| **Secondary Identity** | Voter Card, Military ID, Government ID |
| **Proof of Address** | Utility Bill, Bank Statement, Tax Document, Rental Agreement, Mortgage Statement |
| **Corporate Documents** | Certificate of Incorporation, Articles of Association, Commercial License, Tax Registration, Shareholder Register, Board Resolution, Financial Statements, Corporate Address Proof |
| **Ultimate Beneficial Owner** | UBO Declaration, Ownership Structure, Trust Deed |
| **Additional Verification** | Selfie (Photo), Liveness Check, Video Verification, Signature Form |

---

## Key Features

### 1. Flexible Dynamic Fields

Each document type has its own set of fields (e.g., `full_name`, `passport_type`, `religion`, `trustee_name`). Common and indexed fields are stored in separate columns, while specialized fields are stored in a single JSON column (`fields_data`). This ensures:

- **High performance** in search and filtering.
- **Absolute flexibility** to accommodate any future document type without altering the table structure.

**Example of a dynamic field definition:**
```php
'passport_type' => [
    'label'      => 'passport_type',
    'type'       => 'select',
    'required'   => false,
    'options'    => ['P' => 'Personal', 'D' => 'Diplomatic', 'O' => 'Official'],
    'order'      => 350,
],
```

### 2. Document Lifecycle and Verification

Documents pass through specific stages:
- **`pending`**: Awaiting verification (default status upon creation).
- **`verified`**: Verified and approved.
- **`rejected`**: Rejected with an optional reason.
- **`expired`**: Expired (automatically updated when the expiry date passes).

A verifier (e.g., an employee or system) can approve a document via `Manager::verifyDocument()`, which records:
- Verification date (`verified_at`)
- Verifier ID (`verifier_id` / `verifier_type`)
- Trust score (`verification_score`)

### 3. Hard Copy Tracking

The add-on supports tracking hard copy documents received from the customer (e.g., original passport). Dedicated fields:

| Field | Description |
| :--- | :--- |
| `is_physical_submitted` | Has a hard copy been submitted? |
| `physical_copy_type` | Copy type (`original` or `copy`) |
| `physical_received_at` | Date the hard copy was received |
| `physical_returned_at` | Date it was returned to the owner |
| `physical_notes` | Notes about the hard copy |

### 4. Integrated API

All operations are available via a RESTful API, allowing integration with mobile and web applications.

**Example requests:**

```http
GET /api/v1/kyc/documents?document_type=passport&status=pending
POST /api/v1/kyc/documents/{id}/verify
POST /api/v1/kyc/documents
Content-Type: multipart/form-data

{
  "document_type": "national_id",
  "owner_id": 123,
  "owner_type": "RainLab\\User\\Models\\User",
  "fields[full_name]": "Ahmed Mohammed",
  "fields[document_number]": "1234567890",
  "files[document_front]": (binary)
}
```

**Sample response:**
```json
{
  "code": 200,
  "status": true,
  "message": "Document created successfully.",
  "data": {
    "id": 1,
    "document_type": "national_id",
    "status": "pending",
    ...
  }
}
```

### 5. Comprehensive Backend UI

The add-on provides full management lists and forms within the NanoSoft App control panel:

- **Documents list**: With search, filtering, and sorting capabilities on all fields.
- **Create/Edit form**: Divided into tabs (Basic, Owner, Verification, Hard Copy, Files, Publishing).
- **Advanced filters**: Via `config_filter.yaml` including document type, status, issue date, owner, etc.
- **Batch actions**: Verify, reject, delete, export.

---

## System Requirements

| Requirement | Version |
| :--- | :--- |
| PHP | >= 8.0 |
| NanoSoft App | 2.x or 3.x |
| `Nano.API` add-on | ^1.0 |
| `Tss.Basic` add-on | ^1.0 |

---

## Installation and Setup

### 1. Install the Plugin

Copy the add-on folder to `plugins/nano3/kyc`, then run the update command:

```bash
php artisan october:up
```

The `nano3_kyc_documents` table will be created automatically.

### 2. (Optional) Publish Configuration File

To customize document type properties, publish the configuration file:

```bash
php artisan config:publish nano3.kyc
```

This will create `config/nano3/kyc/document_types.php`, which you can modify as needed.

### 3. Set Permissions

After installation, go to **Settings → Administrators → Permissions** and grant appropriate permissions to roles (e.g., `documents.access`, `documents.verify`).

---

## Practical Examples for Developers

### Creating a Document Using `Manager`

```php
use Nano3\Kyc\Classes\Manager;

$result = Manager::createDocument([
    'document_type' => 'passport',
    'owner'         => $user, // user object
    'fields'        => [
        'document_number' => 'P12345678',
        'full_name'       => 'John Doe',
        'issue_date'      => '2020-01-01',
        'expiry_date'     => '2030-01-01',
        'country_of_issue'=> 'US',
        'nationality'     => 'US',
    ],
    'files' => [
        'document_front' => Input::file('front_image')
    ],
    'is_stop_duplicate' => true, // prevent duplicate document number for the same owner
]);

if ($result['status']) {
    $document = $result['model'];
    echo "Document created with ID: " . $document->id;
} else {
    echo "Error: " . $result['error'];
}
```

### Fetching Document Type Fields to Build a Dynamic Form

```php
use Nano3\Kyc\Classes\DocumentType;

$fields = DocumentType::getFieldsSchema('drivers_license');

foreach ($fields as $fieldCode => $fieldDef) {
    echo "<label>" . DocumentType::getFieldLabel('drivers_license', $fieldCode) . "</label>";
    if ($fieldDef['type'] === 'text') {
        echo "<input type='text' name='fields[$fieldCode]' />";
    }
    // ... other types
}
```

### Checking Document Validity

```php
$document = Document::find(1);
if (DocumentType::hasExpiry($document->document_type)) {
    if (!DocumentType::isDocumentAcceptable($document->document_type, $document->expiry_date)) {
        // Document has expired
    }
}
```

---

## `nano3_kyc_documents` Table Structure

| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | bigint | Auto-increment ID |
| `document_type` | string(50) | Document type code (e.g., `passport`) |
| `document_category` | string(50) | Document category (e.g., `primary_id`) |
| `code` / `barcode` | string(100) | Reference codes |
| `document_number` | string(100) | Document number (indexed) |
| `issue_date` / `expiry_date` | date | Issue and expiry dates |
| `full_name` | string(200) | Full name (indexed) |
| `company_name` | string(200) | Company name (indexed) |
| `owner_id` / `owner_type` | polymorphic | Owner (individual/company) |
| `verifier_id` / `verifier_type` | polymorphic | Verifier |
| `subject_id` / `subject_type` | polymorphic | Associated entity (order/transaction) |
| `is_verified` | boolean | Verification status |
| `verified_at` | timestamp | Verification date |
| `verification_score` | tinyint | Trust score (0-100) |
| `status` | string(50) | Status (`pending`, `verified`, `rejected`, `expired`) |
| `fields_data` | json | All dynamic fields specific to the document type |
| `metadata` / `other_data` / `config_data` | json | Additional flexible data |
| `is_physical_submitted` | boolean | Hard copy submitted |
| `physical_copy_type` | string(20) | `original` or `copy` |
| `physical_received_at` / `physical_returned_at` | timestamp | Receive and return dates |
| `companys_id` / `departments_id` | string(190) | Multi-company and multi-branch support |
| `timestamps` / `softDeletes` | - | Timestamp tracking and soft delete |

---

## Customization via Configuration File

Any default document type properties can be overridden via `config/nano3/kyc/document_types.php`. New values are automatically merged with the defaults.

**Example: Customizing passport**
```php
// config/nano3/kyc/document_types.php
return [
    'passport' => [
        'max_file_size_kb' => 5120, // 5MB instead of 10MB
        'requires_selfie_match' => false,
        'fields' => [
            'expiry_date' => ['required' => true],
            'religion'    => ['required' => true], // make religion required
        ],
    ],
];
```

---

## Events

Developers can attach listeners to the following events:

| Event | Description |
| :--- | :--- |
| `nano3.kyc.document.created` | When a new document is created |
| `nano3.kyc.document.updated` | When a document is updated |
| `nano3.kyc.document.deleted` | When a document is deleted (Soft Delete) |
| `nano3.kyc.document.restored` | When a deleted document is restored |
| `nano3.kyc.document.verified` | When a document is verified |
| `nano3.kyc.document.rejected` | When a document is rejected |

---

## Dependencies and Required Plugins

- **Nano.API**: Provides the foundational API structure (controllers, transformers, caching).
- **Tss.Basic**: Provides helper functions for companies and departments (`BasicHelper`).

---

## License and Support

This add-on is proprietary property of **Nano2Soft**. All rights reserved © 2025.

For inquiries and technical support:
- Website: [https://nano2soft.com](https://nano2soft.com)
- Email: support@nano2soft.com

## Additional Documentation

**Reference Documentation**:
- [`DocumentType` Class Documentation](./docs/Kyc/Docs-DocumentType-Class-en.md)
- [`Manager` Class Documentation](./docs/Kyc/Docs-Manager-Class-en.md)
- [`HasAssessKycStatus` Trait Documentation](./docs/Kyc/Docs-HasAssessKycStatus-Trait-en.md)
- [`Document` Model Documentation](./docs/Kyc/Docs-Document-Model-en.md)
- [API Documentation](./docs/Kyc/Docs-API-Documentation-en.md)

