# Update 2026-1

**One-Month Updates**

## 2026-01-01 - 2026-01-04
**Support FileCleanerUnnecessaryFiles**

Development of an integrated and advanced system for cleaning unnecessary files in the Nanosoft system.

### Introduction
An integrated update for the unnecessary file cleaner was launched, representing a qualitative leap in Nanosoft system maintenance tools. This version transforms the traditional maintenance process into an integrated smart platform, combining performance accuracy and ease of use through an enhanced interface that allows for the safe discovery, organization, and deletion of excess files.

The update features an intelligent backup and safe restore system, with an enhanced file search and discovery mechanism. It also introduces a Test Mode to simulate operations before actual execution and integrates advancedly with system permissions and folder structure. The cleaner is designed to ensure automatic protection through proactive checks, improve system performance, free up storage space, and reduce security risks.

This update represents a strategic investment, providing developers and system administrators with effective tools to enhance efficiency and reliability while maintaining system stability and ease of use.

A video is available for clarification.

---

## 2026-01-05 - 2026-01-06
**Update Nano.AccountsApi And Nano.StudentsApi**

- Fixed missing data issues in `api/accounts/students` in API version 2.
- Support for including account data via the `is_include_account` relationship in API version 2.
- Support for including student data via the `is_include_student` relationship in API version 2.
- Updated Nano.AccountsApi documentation in API version 2.

The software module for accounts and student data in the API section has been updated for compatibility with bank requirements, and the code has been improved and updated in many aspects. New relationships have been added within the account balance retrieval section, such as including financial account data or student data, etc., with the ability to easily control all new features and properties from the settings file.
The documentation has been updated with illustrative examples.

**It is now possible to retrieve account data or person data (like a student) when fetching the balance by passing the following variables:**

```json
{
  "is_include_account": "A Boolean value (0 or 1) used if we want to return account data with the balance",
  "is_include_person": "A Boolean value (0 or 1) used if we want to return data of the affected object, such as a student, client, or current user",
  "is_include_student": "Works the same as the previous variable and is more specific for student data"
}
```

**Important Note:** The ability to return data with the account balance is controlled through the following environment settings:

```plaintext
# Allow fetching account data with balance
# null, true, false
NANO_ACCOUNTSAPI_BALANCES_IS_ALLOW_INCLUDE_ACCOUNT = null

# Allow returning person data when fetching their balance
NANO_ACCOUNTSAPI_BALANCES_IS_ALLOW_INCLUDE_PERSON = null

# Allow returning student data when fetching their balance in /api/v1/accounts/balances
NANO_ACCOUNTSAPI_BALANCES_IS_ALLOW_INCLUDE_STUDENT = null

# Allow returning student data when fetching their balance in /api/v1/accounts/students/balances
NANO_ACCOUNTSAPI_STUDENT_BALANCES_IS_ALLOW_INCLUDE_STUDENT = null
```

The following examples have been added to the documentation:

```html
### Example 1.1.2 get Balances student_id = 3 And Include student_data

In the following example, we will include student data in the returned data by passing the parameter is_include_student=1 as follows:

POST http://localhost:8006/api/v1/accounts/students/balances?student_id=3&is_include_student=1
Or
POST http://localhost:8006/api/v1/accounts/students/balances?student_id=3&is_include_person=1

### Example 1.1.3 get Balances student_id = 3 And Include account_data and student_data

In the following example, we will include student data in the returned data by passing the parameter is_include_student=1, and we will also include financial account data as follows:

POST http://localhost:8006/api/v1/accounts/students/balances?student_id=3&is_include_student=1&is_include_account=1
Or
POST http://localhost:8006/api/v1/accounts/students/balances?student_id=3&is_include_person=1&is_include_account=1
```

---

## 2026-01-07
**Support Chat AI In Frontend**

An AI assistant has been added to the frontend interfaces of websites, allowing professional control over the assistant's properties and information.

A video is available for clarification.

---

## 2026-01-06 - 2026-01-07
**Support Secret Includes Relations In Orders Types**

- Support for the `order_data` relationship in OrdersTypeTransformer.
- Support for the `load_types` relationship in OrdersTypeTransformer.
- Support for the `vehicle_types` relationship in OrdersTypeTransformer.
- Support for the `payment_methods` relationship in OrdersTypeTransformer.
- Support for the `shipping_methods` relationship in OrdersTypeTransformer.
- Support for the `address_type` relationship in OrdersTypeTransformer.
- Support for the `/api/v1/orders/orderstypes/show` endpoint in Nano.OrdersApi.
- Support for retrieving current order types in the Header Request or User Preferences.

The order types section in the API has been updated, and protected relationships have been added to facilitate data retrieval during order completion steps. A new API has been added to return current order type data based on passing the order type in the header or through current user preference data. The documentation has been updated with examples and explanatory notes for the new updates and relationships.

It is now possible to retrieve order type data through the following methods:

```plaintext
GET /api/v1/orders/orderstypes/{id}
GET /api/v1/orders/orderstypes/{ref_type}
GET /api/v1/orders/orderstypes/show
```

The new relationships added are as follows:

#### Include Secret Relation

```json
{
  "order_data": "The relationship for retrieving current incomplete order data according to the specified type",
  "load_types": "The relationship for retrieving supported load types in the passed order type",
  "vehicle_types": "The relationship for retrieving supported vehicle types in the passed order type",
  "address_type": "The relationship for retrieving address type options",
  "payment_methods": "The relationship for retrieving supported payment methods in the passed order type",
  "shipping_methods": "The relationship for retrieving supported shipping methods in the passed order type"
}
```

---

## 2026-01-08 - 2026-01-11
**Development of the Order Types System: Adding Full Control over Sender, Receiver, Invoice, and Identity Data**

Development of the order types system: Adding full control over sender, receiver, invoice, and identity data.

### Update Summary:
A major update was performed on the "Order Types" unit to enable detailed and mandatory management of data fields that customers fill during order creation, through the control panel and Application Programming Interface (API).

### List of Technical Changes:
1. Addition of new control properties in sender (From) data:
   - `is_allow_*` and `is_required_*` fields to control:
     - Company name (`from_company`)
     - First name (`from_firstname`)
     - Last name / Surname (`from_lastname`)
     - Phone number (`from_phone`)
     - Email (`from_email`)

2. Addition of new control properties in receiver (Shipping) data:
   - `is_allow_*` and `is_required_*` fields to control the same set of fields as sender data but for the receiver.

3. Addition of new control properties in billing data:
   - `is_allow_*` and `is_required_*` fields to control the same set of fields as receiver data separately, to allow billing data different from the shipping address.

4. Addition of new control properties in identity (Passport/ID) data:
   - Control fields for the following data:
     - Identity type (`passport_type`)
     - Identity number (`passport_number`)

5. User Interface Update (Control Panel):
   - New settings interfaces were developed within the "Order Types" section, allowing the administrator to easily modify the above properties for each order type.

6. Application Programming Interface (API) Update:
   - The Order Types API was updated to include all new fields and control properties for:
     - `sender_controls` (Options for controlling sender data)
     - `receiver_controls` (Options for controlling receiver data)
     - `billing_controls` (Options for controlling billing data if different from shipping data)
     - `identity_controls` (Options for controlling identity data such as passport or ID card)
   - Backward compatibility was ensured, as these properties can be returned as default values (`null` or `false`) for old orders.

7. Documentation Update:
   - Internal documentation and API documentation were fully updated to include explanations of the new properties, updated data structures, and examples of requests and responses.

The order types section has been updated so that many properties can now be controlled through the control panel, such as receiver, sender, and identity data, etc. Appropriate interfaces for these settings have been created in the order types section, and these properties and settings have been added to the Order Types API, with complete documentation updates.

**New properties have been added to control sender, receiver, and billing data such as sender name, receiver name, etc., as follows:**

#### Sender Data Control Options

```json
{
  "is_allow_from_company": "Whether the client is allowed to write the sending company name",
  "is_required_from_company": "Whether the company name is mandatory",
  "is_allow_from_firstname": "Whether writing the sender's name is allowed",
  "is_required_from_firstname": "Whether the sender's name is mandatory",
  "is_allow_from_lastname": "Whether writing the sender's last name is allowed",
  "is_required_from_lastname": "Whether the last name or surname is mandatory",
  "is_allow_from_phone": "Whether writing the sender's phone is allowed",
  "is_required_from_phone": "Whether the sender's phone is mandatory",
  "is_allow_from_email": "Whether writing the sender's email is allowed",
  "is_required_from_email": "Whether the sender's email is mandatory"
}
```

#### Receiver Data Control Options

```json
{
  "is_allow_shipping_company": "Allows writing the receiving company name",
  "is_required_shipping_company": "Whether the receiving company name is mandatory",
  "is_allow_shipping_firstname": "Whether writing the receiver's name is allowed",
  "is_required_shipping_firstname": "Whether the receiver's name is mandatory",
  "is_allow_shipping_lastname": "Whether writing the receiver's last name is allowed",
  "is_required_shipping_lastname": "Whether the receiver's last name is mandatory",
  "is_allow_shipping_phone": "Whether writing the receiver's phone is allowed",
  "is_required_shipping_phone": "Whether the receiver's phone is mandatory",
  "is_allow_shipping_email": "Whether writing the receiver's email is allowed",
  "is_required_shipping_email": "Whether the receiver's email is mandatory"
}
```

#### Billing Data Control Options (if different from shipping data)

```json
{
  "is_allow_billing_company": 1,
  "is_required_billing_company": 1,
  "is_allow_billing_firstname": 1,
  "is_required_billing_firstname": 1,
  "is_allow_billing_lastname": 1,
  "is_required_billing_lastname": 0,
  "is_allow_billing_phone": 0,
  "is_required_billing_phone": 0,
  "is_allow_billing_email": 0,
  "is_required_billing_email": 0
}
```

#### Identity Data Control Options (such as passport or ID card)

```json
{
  "is_allow_passport_type": "Whether the client is allowed to specify the identity type",
  "is_required_passport_type": "Whether the identity type is mandatory",
  "is_allow_passport_number": "Whether the user is allowed to enter the identity number",
  "is_required_passport_number": "Whether the identity number is mandatory"
}
```

A video is available for clarification.

---

## 2026-01-12 - 2026-01-13
**Update Nano.Api And Nano.FollowsApi**

- Updated the `processAllIncludeFormatsUnified` function in Class `FieldSelection`.
- Support for `user_is_follow` and `user_object_follow` relationships in `MorphToTransformer`.
- Support for `user.user_is_follow` and `user.user_object_follow` relationships in `FollowTransformer`.
- Support for `followable.user_is_follow` and `followable.user_object_follow` relationships in `FollowTransformer`.

The API section has been updated, and retrieving nested relationship data has been improved. The software module for the follows system has been updated, and new relationships have been added to facilitate knowing whether the current user follows their followers or not. The documentation has been updated, and the new relationships have been added to the documentation.

**The following relationships are now supported when fetching the current user's followers to determine if the current user follows their followers:**

```json
{
  "user.user_is_follow": "Returns a Boolean value indicating whether the current user follows their follower or not",
  "user.user_object_follow": "The follow relationship object if the current user follows their follower"
}
```

**The following examples can be reviewed for further clarification of the relationships:**

#### Example 6.2.1 Get List My Follows Current User And Include user.user_is_follow

#### Example 6.2.2 Get List Him Follows Current User And Include followable.user_is_follow

---

## 2026-01-14 - 2026-01-18
**Update Nano.Follows And Nano.FollowsApi**

- Support for checking user preference `is_required_accepted_follows` when adding a follow.
- Support for the `api/v1/follows/follows/toggle` endpoint in API version 2.
- Support for validating relationships when they are included in API version 2.
- Support for the `is_followable_follows_user` relationship in API version 2.
- Support for `is_current_user_follows_followable` and `is_current_user_follows_user` relationships in API version 2.
- Support for `is_followable_follows_current_user` and `is_user_follows_current_user` relationships in API version 2.

The follows system section has been updated, and its API has been updated where various relationships have been added, and the Toggle function has been supported in the API. Documentation for the updates has been written with several illustrative examples for all scenarios.

### Toggle Follows

**This part performs all previous parts of adding, modifying, confirming, and deleting follows with a single function:**

```plaintext
DELETE /api/v1/follows/follows/toggle
```

#### Example 4.2.1 Toggle Add
#### Example 4.2.2 Toggle Delete
#### Example 4.2.3 Toggle Update
##### Example 4.2.3.1 Toggle Update is_accepted=1
##### Example 4.2.3.2 Toggle Update is_accepted=0

**Clarification of the added relationships:**

##### followable relation
**It is the relationship for the object affected by the operation, i.e., the object that was followed or that is being followed.**

##### user relation
**It is the relationship for the object that performed the operation, i.e., the object that initiated the follow.**

##### Other Relation

**The following relationships are now supported when fetching the current user's followers to determine if the current user follows their followers:**

```json
{
  "user.user_is_follow": "Returns a Boolean value indicating whether the current user follows their follower or not",
  "user.user_object_follow": "The follow relationship object if the current user follows their follower"
}
```

**New relationships have been added that can be used if the current user views another user's followers:**

```json
{
  "is_followable_follows_user": "Indicates whether the object in the follow record follows the user in the follow record",
  "is_current_user_follows_followable": "This relationship indicates whether the current user follows the object in the follow record",
  "is_current_user_follows_user": "This relationship indicates whether the current user follows the user in the relationship record",
  "is_followable_follows_current_user": "Indicates whether the object in the follow record follows the current user or not",
  "is_user_follows_current_user": "Indicates whether the user in the follow record follows the current user"
}
```

**Places to use the relationships are as follows:**

```json
{
  "is_current_user_follows_followable": "When fetching a list of people followed by a specific user and we want to know if the current user follows these people or not",
  "is_current_user_follows_user": "When fetching a list of a specific user's followers and we want to know if the current user follows these people or not",
  "is_followable_follows_current_user": "When fetching a list of people followed by a specific user and we want to know if these people follow the current user or not",
  "is_user_follows_current_user": "When fetching a list of a specific user's followers and we want to know if these people follow the current user or not"
}
```

**The added illustrative examples for dealing with the follows system in a specific profile are as follows:**

#### Example 6.2.4 get List Has Follows Other Object And include=user

In the following example, we will fetch followers of a specific object as follows:

```plaintext
GET http://localhost:8006/api/v1/follows/follows?isUser=0&him=0&my=0&object_type=Backend\Models\User&object_id=24&include=user
```

#### Example 6.2.5 get List Him Follows Other Object And include=followable

In the following example, we will fetch a list of people followed by a specific object as follows:

```plaintext
GET http://localhost:8006/api/v1/follows/follows?isUser=0&him=0&my=0&user_type=Backend\Models\User&user_id=24&include=followable
```

#### Example 6.2.6 get List Has Follows Other Object And include=is_current_user_follows_user,is_user_follows_current_user

In the following example, we will fetch followers of a specific object while checking if the current user follows these people and checking if these people follow the current user as follows:

```plaintext
GET http://localhost:8006/api/v1/follows/follows?isUser=0&him=0&my=0&object_type=Backend\Models\User&object_id=24&include=user,is_current_user_follows_user,is_user_follows_current_user
```

#### Example 6.2.7 get List Him Follows Other Object And include=is_current_user_follows_followable,is_followable_follows_current_user

In the following example, we will fetch a list of people followed by a specific object while checking if the current user follows these people and if these people follow the current user as follows:

```plaintext
GET http://localhost:8006/api/v1/follows/follows?isUser=0&him=0&my=0&user_type=Backend\Models\User&user_id=24&include=followable,is_current_user_follows_followable,is_followable_follows_current_user
```

---

## 2026-01-19 - 2026-01-20
**Support image and images in Options And OptionsValue And ProductsOptions And ProductsOptionsValue**

- Support for `image` and `images` in Options, OptionsValue, ProductsOptions, and ProductsOptionsValue.
- Support for Filter Has ProductsOptions, ProductsOptionsValue, etc., in the Backend Interface.

The software module for managing products has been updated, where the product management interface has been improved, and special filters for filtering products based on additional fields and properties have been added. The ability to add images for each field and additional property has been supported, and the affected API has been updated.
The filters added to the products are as follows:

```json
{
  "is_has_product_options": "Has additional properties",
  "is_not_has_product_options": "Does not have additional properties",
  "is_has_product_options_value": "Has additional property values",
  "is_not_has_product_options_value": "Does not have additional property values",
  "is_toggel_any_product_options": "Has any additional properties or property values",
  "is_has_any_product_options": "Has any additional properties or property values",
  "is_not_has_any_product_options": "Does not have any property values or additional properties"
}
```

A video is available for clarification.

---

## 2026-01-21
**Support new and returning customer orders report statistics**

Updates in statistical and graphical reports where the control panel section has been improved so that the panel is initialized with various reports instead of appearing empty for the first time, with the ability to revert to the panel's default settings. The following reports have been added:
- Reports for customers making their first purchase.
- Report for customers who returned to make a purchase.

A video is available for clarification.

---

## 2026-01-22 - 2026-01-23
**Update Tss.Notes And Update formwidgets Notes**

- Updated for compatibility with versions v2.x, v3.x, and v4.x.
- Added support for dark mode for all V4.x color presets.

The software module for the general notes system has been updated, supporting all system versions from the first to the fourth version V4.
Dark mode formats have been supported for all versions, and the code has been updated to comply with the requirements of each version.

A video is available for clarification.

---

## 2026-01-24
**Support currencys_id_symbol Column In Products API Version 2**

**Nano.ShopApi Version 1.1.28:**
- Support for the `currencys_id_symbol` field in Products API version 2.
- Support for `is_set_coupon` in the Step coupon in Checkout2 API version 2.
- Support for including relationships (`country`, `state`, `directorate`) in AddressTransformer API.

The API for products and product units has been updated, where a new field named `currencys_id_symbol` has been added to the returned data, specifically for the currency symbol alongside the existing currency name, without the need to include the relationship for currency data. A new parameter for the coupon entry step named `is_set_coupon` has been added, used if we want to enter the coupon before entering shipping data and basic data in the order completion steps.
The API for addresses has been updated to support relationships for country, city, and region.

A video is available for clarification.

---

## 2026-01-15 - 2026-01-27
**Support Units Conversion System v2.0: Smart Context Validation & Multi-language Support**

### Summary of Comprehensive Unit Conversion System Updates

#### Overview
An integrated unit conversion system has been developed, supporting 16 main types of units including length, weight, volume, area, temperature, time, speed, pressure, energy, power, data, angles, and quantities of various types.

#### Main Components

1. **UnitType** - Definition of unit types:
   - Management of unit type constants (16 main types + 5 sub-types for quantities).
   - Functions to get translated labels (Arabic/English).
   - Validation of types and retrieval of main types.

2. **UnitNameHelper** - Unit names helper:
   - Centralized translation hub for units in multiple languages.
   - Functions for searching, categorizing, and obtaining unit information.
   - Support for regular and categorized HTML select options.

3. **UnitConverter** - Core converter:
   - Performs conversions between all supported units.
   - Supports decimal precision (up to 8 decimal places).
   - Batch conversions for efficiency.
   - Validation of unit compatibility.

4. **UnitContextValidator** - Context validator:
   - Validates the logic of converting quantity units.
   - Specific rules for conversion between different contexts.
   - Prevention of illogical conversions (e.g., dozen to ream).

5. **SmartUnitConverter** - Smart converter:
   - Safe conversion with automatic error handling.
   - Returns warnings for suspicious conversions.
   - Suitable for applications requiring high reliability.

6. **UnitFactory and Unit** - Factory pattern:
   - Creates reusable Unit objects.
   - Encapsulates unit properties and functions.
   - Facilitates testing and expansion.

7. **Display Interfaces**:
   - **UnitsInformation**: Displays unit information in HTML format.
   - **ConversionForm**: Creates ready-made conversion forms.
   - Bootstrap support for responsive design.

#### Technical Features

✅ **Comprehensive Support:**
- 16 main unit types.
- 80+ standard units.
- Temperature support (C, F, K).
- Quantity units with logical contexts.

✅ **Accuracy and Reliability:**
- High conversion accuracy (8 decimal places).
- Input validation.
- Comprehensive error handling.
- Smart conversions with logic validation.

✅ **Ease of Use:**
- Clear and unified programming interface.
- Multilingual support (Arabic/English).
- Comprehensive illustrative examples.
- Detailed documentation for each function.

✅ **Scalability:**
- Modular design that facilitates adding new units.
- Support for adding additional languages.
- Factory Pattern for object creation.
- Clear separation between components.

#### Usage Examples

##### Basic Conversion
```php
$result = UnitConverter::convert(1, 'm', 'cm', 2); // 100.00
```

##### Safe Conversion
```php
$result = SmartUnitConverter::safeConvert(1, 'dozen', 'box', 2);
```

##### Getting Labels
```php
$label = UnitNameHelper::getUnitLabel('m', 'ar'); // 'meter'
```

##### Creating User Interface
```php
$form = new ConversionForm();
echo $form->renderAdvancedForm();
```

#### Recommended Best Practices
1. **Use SmartUnitConverter** for conversions in production applications.
2. **Use UnitContextValidator** to validate the logic of quantity conversions.
3. **Use UnitFactory** to create Unit objects instead of direct creation.
4. **Avoid direct conversions** between different unit types.
5. **Use error handling** around all conversion calls.

#### Suggested Future Updates
1. Adding more specialized units (medical, scientific, etc.).
2. Supporting complex conversions (compound units).
3. Integration with current inventory management systems.

---

**Note**: The system is ready for integration with inventory management, accounting, and scientific and engineering applications. It is designed to be lightweight, fast-performing, and easy to maintain.

```html
Link to the general unit system interfaces:
https://account.now-ye.com/nano/helpers/units
```

A video is available for clarification.

---

## 2026-01-26 - 2026-01-27
**Support Nano.Api Settings Interface In Backend**

The API system section has been updated, and control interfaces for the settings of this section have been fully supported through the backend control panel, whereas previously the settings for this section were controlled only through files.

A video is available for clarification.

---

## 2026-01-28 - 2026-01-29
**Support Nano.AuthApi Settings Interface In Backend**

The authentication section in the API has been updated, and control interfaces for the settings of this section have been fully supported through the backend control panel, whereas previously the settings for this section were controlled only through files.

A video is available for clarification.

---

## 2026-01-29 - 2026-01-31
**Update Nano UserPlus and BackendUserPlus And AuthApi**

- Added columns `business_number`, `vat_number`, `language`, and `nationality` to the frontend user profile.
  - `builder_table_add_business_columns_to_frontend_users_table.php`
- Added columns `business_number`, `vat_number`, `language`, and `nationality` to the backend user profile.
  - `builder_table_add_business_columns_to_backend_users_table.php`
- Support for columns `business_number`, `vat_number`, `language`, and `nationality` in the user profile in AuthApi.

The user management section has been updated, and new fields have been added specifically for commercial registration number, tax number, languages, and nationality. New types have been supported for frontend user accounts, where it is now possible to determine the account type when creating the account: whether it is an individual account, establishment account, delivery agent account, freelance work account, etc. User interfaces in the control panel section have been updated to support the added fields, and filters have been created for filtering users based on the added fields, along with various other improvements.

A video is available for clarification.