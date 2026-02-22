# Update 2026-2

**Two-month updates**

## 2026-02-01 - 2026-02-02

**Support Nationalities In Nano.Location**

- Created table `nano_location_nationalities`
- `create_nano_location_nationalities_table.php`
- Support `NationalityHelper` Class
- Support `ReligionsHelper` Class
- Support Nationalities Interface in Backend
- Support Auto Seed Nationalities in Interface

The geographic region management software module has been updated. A new table dedicated to nationalities has been added, and nationality management interfaces have been created in the system control panel, including a mechanism to generate data for all nationalities from the interface with a single click. Professional classes for managing nationalities and religions have been developed.

## 2026-2-3

**Support Nationalities in Nano.LocationAPI**

    - Support Nationalities API
    - Support Nationalities API Documentation
    - Support Nationalities Endpoint API and API Documentation

The Nano.LocationAPI module has been updated. The API for handling nationalities has been added, and comprehensive documentation has been written, including several illustrative examples.

```
GET /api/v1/location/nationalities
GET /api/v1/location/nationalities/{id}
GET /api/v1/location/nationalities/activestats
```

## 2026-2-5

** Support Filter By Targets `is_has_targets`, `target_type`, `target_id` in Adverts in API Version 2 **

The Adverts API has been updated. Filters have been added to filter advertisements based on the type of targeted object, making it possible to fetch advertisements for a specific department, category, and so on. The documentation has been updated with several examples for clarification.

** To retrieve advertisements that target specific objects within the system, such as a specific product, store, main category, sub-category, or specific order type, the following parameters are used: **

```json
{
    "is_has_targets": 1,
    "target_type": "Here, pass the type of the object related to the advertisement.",
    "target_id": "Here, pass the identifier of the object related to the advertisement."
}
```

** The types of objects that can be associated with advertisements and can be passed within the `target_type` parameter are as follows: **

```json
{
    "Nano\Orders\Models\OrdersType": "Order Types",
    "Nano\Tags\Models\Type": "Main Categories Or Tag Type",
    "Tss\Basic\Models\Department": "Stores & Branches",
    "Nano\Tags\Models\Categorie": "Categories",
    "Nano\Tags\Models\Tag": "Hashtags",
    "Nano\Shop\Models\Product": "Products"
}
```

** For clarification on handling targeted objects, refer to the following examples: **

Example 1.2.3
Example 1.2.4
Example 1.2.5
Example 1.2.6

## 2026-2-3 - 2026-2-6

**Updated Nano.Location, Nano.LocationApi, Nano.UserPlus, and Nano.AuthApi**

-   Added support for the LocaleHelper class.
-   Added support for the LanguagesHelper class.
-   Updated the Address Behaviors class.
-   Added support for Languages API endpoints and their documentation.
-   Added support for Religions API endpoints and their documentation.
-   Added support for a `belongsTo` relationship with Nationalities in the Frontend User Model.
-   Added dropdown fields for nationality and language in the Backend Interface.

The frontend user management component was updated, the backend user management interfaces were improved, and filters were added to filter users by religion, nationality, or language. The input fields for this data were enhanced, and new relationships were added to the user file specifically for nationality data. The following APIs were created with complete documentation for everything.

The added APIs are as follows:

```
GET /api/v1/location/religions
GET /api/v1/location/religions/{id}
GET /api/v1/location/religions/activelystats

GET /api/v1/location/languages
GET /api/v1/location/languages/{id}
GET /api/v1/location/languages/activelystats
```

## 2026-2-7

**Updated RoutesBrowser API to v3**

We have updated and upgraded the API browsing and testing section to be compatible with the system's third version. Additionally, it is now possible to export a `collection-api.json` file with or without documentation for the purpose of importing the APIs into external tools like Postman and others.

## 2026-2-8

**Updated Map FormWidgets in the Backend**

The input fields for the mapping system in the backend section have been improved and updated.

## 2026-2-9

**Support VisitModel In OrdersType**

Support VisitModel In OrdersType
    - Update Nano2.Visitors Support extendOrdersTypeModel And Config
    - Update Nano.OrdersApi
    - Development of the Order Types System Adding Behaviors VisitModel In OrdersType Models And OrdersTypeTransformer
    - Support Visitors In Order Types
    - Support Visit Event Api In Order Types

The Order Types section has been updated to support the feature of recording views and visits at the order type level, allowing the tracking of visit counts for each order type. The API for this section has been updated, along with the related documentation.
To enable or disable this feature, the following settings have been added:

```
# Allow recording visits when accessing the record in the order type
NANO2_VISITORS_IS_SUPPORT_ORDERS_TYPES_SHOW= true
# Allow recording visits when fetching records in the order types list
NANO2_VISITORS_IS_SUPPORT_ORDERS_TYPES_LIST= true
```

## 2025-10-25 - 2026-2-10

**Support Nano3.Packer Tools Version V2**

A comprehensive and professionally developed software module has been created for processing, encrypting, and preparing code for the production stage. Through this module, an entire software package can be encrypted in several ways—either via a compressed file or by specifying the specific module within the system to be encrypted.

## 2026-02-11

**Update Nano.Tags And Nano.TagsApi**

    - Support TagManager Class
    - Support TagIntegrationHandler EventHandlers Class
    - Support DynamicAddIncludeTags Transformer Class
    - Support tags Relation In Types
    - Support tags Relation In Categories
    - Support Tags Relation And tagsId Filter in api/v1/tags/types API Version 2.
    - Support Tags Relation And tagsId Filter in api/v1/tags/categories API Version 2.
    - Update Docs Api Version 2.

The software module for managing public hashtags has been updated. The hashtag management mechanism, as well as the injection of objects, interfaces, queries, and the API, have been professionally improved through a class named `TagManager`. This class flexibly associates any object or interface with hashtags while providing control over each object's properties (such as interfaces, etc.). Complete documentation has been created for interacting with the new mechanism, and the API along with its documentation has been updated. Additionally, it is now possible to filter public types and public categories by hashtags.

See [docs/Docs-TagManager-en.md](2026/updates/en/docs/Docs-TagManager-en.md)


### Example 1.1.5 get List of Types with include=tags and where tagsId=43,54
### Example 1.1.12 get List of Categories with include=tags and where tagsId=1,2

**In the following example, we will fetch categories by hashtags, including the hashtag relation as follows:**

```
GET http://localhost:8006/api/v1/tags/categories?tagsId=1,2&include=tags
```

**The parameters passed are as follows:**

```json
{
  "tagsId": "1,2",
  "include": tags,
}
```

## 2026-02-08 – 2026-02-12

**Support Google Merchant Validator Version 2**

    - Support ValidationLibrary Class
    - Support GoogleMerchantFields Class
    - Support GoogleMerchantValidator Class
    - Support Interface Google Merchant Validator

A new software module has been created for professionally validating the content of Google Merchant feed items, verifying all rules for each item in the file.

Public link to view the interface of this module:

https://account.now-ye.com/nano2/googlemerchant/validator

---

## 2026-02-12

**Update GDPR**

The software module for cookies and obtaining consent from the browser or site visitor has been updated. An experimental API has been added for this part.

The endpoints added in the API are as follows:

```json

POST /api/v1/gdpr/cookie/accept-defaults
Accept default cookies (e.g., cookieBanner->onAccept)

GET api/v1/gdpr/cookie/check/{code}/{level?}
Check the validity of a specific cookie
GET api/v1/gdpr/cookie/consent
Get the current user consent status
POST api/v1/gdpr/cookie/decline-all
Decline all cookies (e.g., cookieBanner->onDecline)
POST api/v1/gdpr/cookie/reset
Reset all consents
GET api/v1/gdpr/cookie/settings
Get all cookie groups and settings
GET api/v1/gdpr/cookie/status
Get the general consent status
POST api/v1/gdpr/cookie/update
Update cookie preferences (e.g., CookieManager)
GET api/v1/gdpr/data-retention/policies
Get all data retention policies
```

## 2026-2-6 - 2026-2-15

**Support Nano3.Legal**

A new integrated software module for managing legal documents and records has been launched. The module includes all legal documents such as the privacy policy, terms of use, refund and payment policy, and others. Professional classes have been created to manage document types and classifications, and document templates have been automatically generated according to international specifications. Its API part has been created completely and professionally, with its documentation completed, and test functions have been created to test the software module.

The API endpoints are as follows:
```
GET api/v1/legal/support/ref-types/{id?}
GET api/v1/legal/support/types/{id?}
GET api/v1/legal/documents
GET api/v1/legal/documents/activelystats
GET api/v1/legal/documents/{id}
GET api/v1/legal/consents
GET api/v1/legal/consents/activelystats
GET api/v1/legal/consents/{id}
```

The document types are classified as follows:
```json
{
        "ecommerce": "E-commerce",
        "rental": "Rental and Applications",
        "contract": "Contracts and Orders",
        "user": "Users and Registration",
        "employment": "Employment and Work",
        "security": "Security and Protection",
        "general": "General",
        "industry": "Industry Specific"
}
```

The main document identifiers are as follows:
```json
{
        "privacy_policy": "Privacy Policy",
        "terms_of_service": "Terms of Service",
        "return_policy": "Return and Exchange Policy",
        "shipping_policy": "Shipping and Delivery Policy",
        "cookie_policy": "Cookie Policy",
        "user_agreement": "User Agreement"
}
```

See [docs/docs-DocumentType-en.md](2026/updates/en/docs/docs-DocumentType-en.md)

## 2026-2-15

**Update OrdersRequests And OrdersLists Api Version 2**

    - Update OrdersRequests And OrdersLists Api Version 2
    - Support Filter is_user_or_delivery_user_id In OrdersLists And OrdersRequests Api Version 2
    - Support Permission Backend User In get OrdersRequests Api Version 2
    - Support Filter companys_id And departments_id In get OrdersRequests Api Version 2
The section for managing delivery request offers in the control panel and its related API has been updated.

## 2026-2-16 - 2026-2-18

**Support Filter: is_has_product_options, is_has_product_options_value, and product_options_values**

  -- Update Nano.Shop Product Models  
  -- Update Nano.ShopApi Products ApiController  
  -- Update Nano.ShopApi Products Api Docs  

The product management module has been updated with new filters to allow filtering products based on whether they have additional options or option values. It is now possible to filter by product options and their attributes. The API part has been updated along with the documentation, including illustrative examples.

#### Example 3.3.2: Get Products Filtered by is_has_product_options or is_has_product_options_value

**You can filter products by whether they have additional options or not, and also by whether they have additional option values or not, using the following filters:**

```json
{
  "is_has_product_options": "To fetch products based on whether they have options. Default value is all.",
  "is_has_product_options_value": "To fetch products based on whether they have option values. Default value is all."
}
```

**To fetch products that have additional options, pass the following parameter:**
```json
{
  "is_has_product_options": 1
}
```

**To fetch products that do not have additional options, pass the following parameter:**
```json
{
  "is_has_product_options": 0
}
```

**To fetch products that have additional options and option values, pass the following parameter:**
```json
{
  "is_has_product_options_value": 1
}
```

**To fetch products that do not have additional options and option values, pass the following parameter:**
```json
{
  "is_has_product_options_value": 0
}
```

**In the following example, we will fetch only products that have additional option values, using this filter:**

```
GET http://localhost:8006/api/v1/shop/products?is_has_product_options_value=1
```

#### Example 3.3.3: Get Products Filtered by product_options_values=Black

**You can filter products by whether they have a specific additional option value as follows:**

```json
{
  "product_options_values": "The option value you want to filter products by.",
  "products_options_id": "The ID of the specific option (optional, leave empty to search across all options)."
}
```

**To fetch products where one of their additional option values equals "Black", pass the following parameter:**

```json
{
  "product_options_values": "Black"
}
```

**In the following example, we will fetch products that have additional options and whose option value is "Black":**

```
GET http://localhost:8006/api/v1/shop/products?product_options_values=Black
```
or
```
GET http://localhost:8006/api/v1/shop/products?product_options_values=Black&products_options_id=4,8
```

## 2026-2-18 - 2026-2-19

**Update Google Merchant Validator Version 2**

The Google Merchant file validation module has been updated with new validation rules, significant code improvements, and the addition of new helper classes to enhance the validation of descriptive text professionally. It allows control over the settings of the relevant class and also supports validating descriptive text in multiple languages. A separate class has been created for professional and flexible validation of promotional text, which also supports multiple languages. Many other code improvements have been made.

Public link to view the interface of this module:

https://account.now-ye.com/nano2/googlemerchant/validator


## 2026-2-20 - 2026-2-23

**Nano3.Redactor – Sensitive Data Redaction Tool for Nanosoft Applications**

### Overview

**Nano3.Redactor** is an advanced plugin designed to automatically detect and redact (hide) sensitive data within applications built on the **Nanosoft Software** platform. The plugin analyzes various data types (strings, arrays, objects) and replaces private information such as passwords, API keys, email addresses, credit card numbers, and others with a safe placeholder (e.g., `[REDACTED]`) before logging, displaying, or exporting them.

### How It Works?

The plugin relies on a system of **specialized strategies** that are executed in a defined priority order. Each strategy is responsible for a specific type of sensitive data:

- **Safe Keys Strategy** – Preserves non-sensitive keys (e.g., `id`, `created_at`).
- **Blocked Keys Strategy** – Redacts well-known keys such as `password`, `token`.
- **Regex Patterns Strategy** – Searches for patterns like email addresses, Social Security numbers, credit card numbers.
- **Shannon Entropy Strategy** – Detects high-entropy random strings such as API keys and JWT tokens.
- **Large Object Strategy** – Redacts arrays or objects that exceed a certain size limit.
- **Large String Strategy** – Redacts strings that exceed a certain length limit.

These strategies can be customized through multiple **profiles**, allowing different redaction behaviors depending on the context: a default profile (balanced), a strict profile (for highly sensitive data), and a performance profile (fast with minimal redaction). The plugin also supports **wildcard patterns** for flexibly specifying blocked keys.

### Key Benefits

1. **Data Privacy Protection** – Prevents sensitive information leakage through logs or API responses, reducing the risk of breaches and ensuring compliance with privacy standards such as GDPR and PCI DSS.
2. **Developer Time Savings** – No need to write manual redaction code for every field; configuring the plugin once is enough.
3. **High Flexibility** – Strategies can be customized, custom strategies can be added, and profiles can be created to suit each environment (development, production, auditing).
4. **Improved Log Quality** – With `CustomLogTap`, log context is automatically redacted, keeping logs useful for analysis without exposing sensitive data.
5. **File Scanning Tool** – The plugin provides an Artisan command to scan files and directories for sensitive data, helping with security reviews and vulnerability detection before deployment.
6. **Easy Integration** – Can be used via a simple Facade, through the service container, or by direct instantiation, and works with any data type (arrays, Eloquent models, plain objects).
7. **Comprehensive Language and Encoding Support** – Works with Arabic text and various encodings, and supports entropy calculation to detect random strings even in non-English text.

### Typical Use Cases

- **Logging** – Redact sensitive user data before writing it to the system log.
- **Data Export** – Create backups or reports containing de-identified data.
- **API Responses** – Hide private fields when returning debug information or user data.
- **Security Audit** – Use the file scanning tool to search for forgotten API keys or passwords in the codebase.
- **PCI Compliance** – Automatically redact credit card numbers before storing or logging them.

### In a Nutshell

**Nano3.Redactor** is the ideal plugin for any project running on the **Nanosoft Software** platform that requires a high level of security and privacy without complexity. It gives you peace of mind that your sensitive data will never appear in logs or unauthorized outputs, while maintaining ease of use and full flexibility.