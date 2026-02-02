# Update 2025-8

**August Updates**

## 2025-8-1 - 2025-8-2 Support Audios and Book Intro in Products

- Support Audios and Book Intro in Products
- Support is_mate_data_audios and is_mate_data_book_intro in ProductManager
- Support Filter is_has_audios and is_has_book_intro in Products
- Support Archive Audios Files
- Support Archive Book Intro Files
- Support Audios and Book Intro in Products API Version 2
- Support is_mate_data_audios and is_mate_data_book_intro in Products API Version 2
- Support Filter is_has_audios and is_has_book_intro in Products API Version 2

Various updates related to product management, where it has become possible to upload audio files for products used in audiobooks or audio selling stores. A feature has been added to allow the upload of an introductory file or a summary of the book. Filters specific to audio files and the book intro file have been added, allowing products to be filtered based on whether they contain audio or not.

An archiving mechanism for book files has been added instead of deleting them, to preserve book sources and protect book files from users. Settings have been added for all previous updates, allowing the archiving feature for books to be activated or not through the following settings:

```
# Archive book intro file instead of deleting it 
NANO_SHOP_PRODUCTS_IS_ARCHIVE_BOOK_INTRO= false
# Archive product files instead of deleting them 
NANO_SHOP_PRODUCTS_IS_ARCHIVE_FILES= false
```

The API for product management has been updated, adding fields for audio files and the book intro file along with their respective filters, and the documentation for them has been written.

To filter products based on the presence of audio files, we use the following parameter:

```json
{
    "is_has_audios": 1
}
```

To filter products based on the presence of a book intro file, we use the following parameter:

```json
{
    "is_has_book_intro": 1
}
```

To return information about audio files and the book intro file, we use the following parameters:

```json
{
    "is_mate_data_book_intro": "Boolean value 0 or 1 to return 
    information about the book intro file, such as description, title, etc.",
    "is_mate_data_audios": "Boolean value 0 or 1 to return 
    information about audio files, such as description, title, etc."
}
```

To clarify the updates, videos and images can be viewed.

## 2023-8-3 Update Reports School In One Subject
Updates in monthly, semi-annual, and final reports where the appearance of these reports has been improved when selecting only one subject. These reports were designed to print all subject grades in a consistent manner, and when selecting only one subject in the report, the appearance of the font was not suitable, so it was modified and the issue was fixed.

## 2025-8-3 Support Protecting Item and Book Files
A mechanism has been added to protect book files in products so that the paths of files are not returned directly in the API. Settings have been made to activate this mechanism. This feature is used in the electronic book store.

### The settings for activating file protection are as follows:
```
# Protect product files so that their paths cannot be viewed 
# The value * can be used to protect files of all types of products 
NANO_SHOPAPI_PRODUCTS_IS_CHECK_FILES_PROTECTED= '*'

# Types of products whose files are to be protected 
# An array of product types whose files need to be protected 
# Types of products can be written or left empty if the value * is set in the previous setting 
NANO_SHOPAPI_PRODUCTS_FILES_PROTECTED_REF_TYPE= null
```
A video can be viewed to understand how to work with this update.

## 2025-8-4 Support Get My Books List In Products API Version 2
An API has been added to fetch the books purchased by the client, allowing control over the validity period of these purchases for one year, for example, depending on the store owner's preference. An API has also been created to display the file content as a web view with specific permissions to protect book files from theft and direct downloads. Videos and images can be viewed for clarification.

### List My Books
To fetch the current user's books, we use the following link:

```
GET /api/v1/shop/books
```

## 2025-8-5 Support Video in Adverts
- Support Video in Adverts
- Support Video to Adverts in API Version 2

Video advertisements have been added to the ads section, allowing a video to be uploaded to the advertisement. The API for advertisements has been updated to support video ads in the system.

## 2025-8-6 - 2025-8-7 Support File Upload Encrypt
A mechanism has been added to protect and encrypt the files uploaded to the system based on user needs, aimed at protecting sensitive files that we want to prevent people from accessing their contents. Even if they download them, they will not be opened because they are encrypted, requiring decryption to view their contents.

## 2025-8-7 Backend User Management Section Update
Updates in the backend user management section where the handling of suspended accounts due to multiple failed login attempts and banned accounts has been improved, allowing for the direct cancellation of any suspended account from the user management interface instead of waiting for the suspension period to end. A video can be viewed for clarification.

## 2025-8-3 - 2025-8-8 Support Nano2.Ciphersweet Encrypts Version v1
The first version of the software module for encrypting and protecting files in the system has been launched, designed to work smoothly with various other components of the system. It can be used to encrypt the contents of files in several parts of the system and in the matter of file exchange between the site and the application. It has been used in the book store to encrypt the contents of books purchased by users with a unique key for each user, allowing them to read the content of the file in the application. This module also handles files stored locally or on other providers like S3, Google Drive, Dropbox, etc.

An API has been created for generating keys and testing the validity of keys as follows: To generate an encryption key through the API, we use the following link:

```
POST https://localhost:8006/api/v1/ciphersweet/encrypts/create
```

```json
{
  "code": 200,
  "status": true,
  "message": "Operation completed successfully",
  "error": "",
  "data": {
    "process_type": "create",
    "cipher": "AES-256-CBC",
    "encrypt_key": "base64:lrEae8L/pqzJ9ziG/mXGfwrcKtoc8nHck5qMb1ohhLA="
  }
}
```

To check if the encryption key is correct, we use the following link:

```
POST https://localhost:8006/api/v1/ciphersweet/encrypts/create?process_type=check&encrypt_key=base64:lrEae8L/pqzJ9ziG/mXGfwrcKtoc8nHck5qMb1ohhLA=
```

```json
{
  "code": 200,
  "status": true,
  "message": "Operation completed successfully",
  "error": "",
  "data": {
    "encrypt_key_type1": "base64",
    "process_type": "check",
    "cipher": "AES-256-CBC",
    "encrypt_key": "base64:lrEae8L/pqzJ9ziG/mXGfwrcKtoc8nHck5qMb1ohhLA=",
    "encrypt_key_type": "base64"
  }
}
```

## 2025-8-8 Support LabsMobile SMS Channel
- Update SMS Channel Model
- Support LabsMobile SMS Channel

The SMS service provider management section has been updated, and a new service provider affiliated with LabsMobile has been added.

## 2025-8-9 Support View and Download Book Files
An API has been created for browsing or downloading the books that the user has purchased from the store, allowing this section to read books in the form of a web view or download them to the application in an encrypted form and then open them only in the application. Both methods are secure.

To read the book file and browse it, we use the following link with the product ID and file ID:

**Important Note:** The user must be logged into the system.

If there is a specific period for using the book, the user will not be able to view the book after this period until they purchase it again.

```
GET /api/v1/shop/books/view
```

To download the book, we use the following link with the product ID, file ID, and encryption key:

```
GET /api/v1/shop/books/download
```

The required parameters are as follows:

```json
{
  "product_id": "Product ID",
  "file_id": "Electronic file ID",
  "encrypt_key": "Encryption key used when downloading the file"
}
```

## 2025-8-10
- Support getOtherItems and Config similar_items in Products API Version 2
- Support Get other_items or similar items based on authors, classrooms, subjects_id, or publishers in Products API Version 2

The part for fetching similar or matching products has been updated, allowing the criteria for fetching similar products to be defined based on classifications, hashtags, names, authors, classrooms, subjects, or publishers in the case of a book. These criteria can be defined while fetching data in the API or through the configuration file to apply these criteria automatically according to the nature of the store.

### The Same Products List
/shop/sameproducts/{id?}

The parameters for determining the criteria for fetching similar or matching products are as follows:

```json
{
  "is_support_authors": 0,
  "is_support_classrooms": 0,
  "is_support_subjects": 0,
  "is_support_publishers": 0,
  "is_support_categories": 1,
  "is_support_tags": 1,
  "is_support_tags_type": 1,
  "is_support_name": 1
}
```

Settings for controlling these criteria can also be configured instead of passing them in a request through the following settings in the system:

```
# Fetch similar or matching products based on classifications 
NANO_SHOPAPI_SIMILAR_ITEMS_IS_CATEGORIES= true
# Fetch similar or matching products based on hashtags
NANO_SHOPAPI_SIMILAR_ITEMS_IS_TAGS= true
# Fetch similar or matching products based on the main category of the product 
NANO_SHOPAPI_SIMILAR_ITEMS_IS_TAGS_TYPE= true
# Fetch similar or matching products based on similarity in name and description 
NANO_SHOPAPI_SIMILAR_ITEMS_IS_NAME= true
# Fetch similar or matching products based on authors 
NANO_SHOPAPI_SIMILAR_ITEMS_IS_AUTHORS= false
# Fetch similar or matching products based on classrooms  
NANO_SHOPAPI_SIMILAR_ITEMS_IS_CLASSROOMS= false
# Fetch similar or matching products based on subjects 
NANO_SHOPAPI_SIMILAR_ITEMS_IS_SUBJECTS= false
# Fetch similar or matching products based on publishers 
NANO_SHOPAPI_SIMILAR_ITEMS_IS_PUBLISHERS= false
```

## 2025-8-11 Update Docs Adverts API and Explain the Relationship Targets
The advertising management section has been updated, where fixes were made for some issues, and the method of handling advertisement directives and events has been explained and clarified. 
Please refer to the documentation and watch the video to understand how to deal with advertisements.

## 2025-8-12 - 2025-8-13 Support Nano3.PdfToImage
- Show Info book pages in Shop API
- Download book pages as images in Shop API

A software module has been launched for processing PDF files and converting the pages of the file into images, allowing the desired page to be fetched as an image and enabling the user to know the number of pages in the file and other tasks related to PDF files. An API has been created to test and use this module separately. 
- The part for fetching information about book pages in the book store has been updated.
- An API has been created to fetch a page from the book as an image to use this part for securely browsing books as images to prevent copying. 
- The documentation for the previous parts has been completed with several examples. 
Videos can be viewed to clarify these updates for developers.

### Show Info Book Pages
**To fetch information about the number of pages in the book, which will be used when loading the pages of the book as images, we use the following link:**
```
GET http://localhost:8006/api/v1/shop/books/infopage?product_id=4242&file_id=4684&page=1
```

### Download Book Pages as Images
**To download pages of the book as images, we use the following link with the product ID, file ID, and the page number to be downloaded as an image:**
```
GET http://localhost:8006/api/v1/shop/books/downloadtoimage?product_id=4242&file_id=4684&page=1
```

## 2025-8-14 Support is_add_column_expiration_at and expiration_at Column In Products API Version 2
A field has been added for the expiration date of book purchase subscriptions, allowing control over whether to display this field or not in the settings, noting that the default value is not to display.

```
# Show the expiration date field in the purchased book 
NANO_SHOPAPI_BOOKS_IS_ADD_COLUMN_EXPIRATION_AT= false
```

**When the previous option is activated, a new field named expiration_at will be returned within the data of the books purchased by the user, indicating the expiration date of the purchase or subscription for the book.**

**The affected API for this update is:**
```
GET /api/v1/shop/books
```

## 2025-8-14 - 2025-8-15 Updated the Products Management Interface
The product management interface has been updated to support viewing and browsing book, video, and audio files directly instead of downloading them for user convenience. 
Videos and images can be viewed for clarification.

## 2025-8-15 Update Report Student Info and Report Seatings
Updates in the student data reports and seating numbers reports, fixing some issues in these reports.

## 2025-8-16 Support Charging or Payment Cards System
The full version of the charging or payment card system has been launched, where this module has been developed over a long period intermittently. This module has been professionally designed to work with various other system modules independently or integratively. This module includes the following: 
- Packages of charging or payment cards, allowing different packages for cards at different prices and balances.
- Payment batches for card printing can be managed independently to facilitate the management and tracking of card printing in the system.
- Charging and payment cards can be generated in bulk or individually according to the packages and linked to the batches automatically. 
- Printing cards and controlling card formats and colors completely.
- Detailed reports about the cards with various filters based on usage, balance, etc.
- A complete permissions system at all levels of card properties in terms of printed and used cards, etc.
- A payment gateway through charging or payment cards or gift cards, linking them easily in the API and applications. 

All the above points have been professionally completed and have been tested preliminarily and will be tested effectively in a bookstore. Videos and images can be viewed to clarify how this module works.

## 2025-8-17 - 2025-8-18 Support Allow Test Mode All Process to Dev In VerifyCode
The OTP section has been updated, allowing the test mode to be activated in the development phase for all verification processes, to train developers. The system simulates sending the verification code as if the process has been completed, using the following code as the verification code 11111 or as set in the settings. To activate the test mode for everyone, the following settings must be activated:

```
# Allow activating test mode for all operations; this option is used in development mode
VERIFYCODE_ALLOW_TEST_ALL= false

# Activate test mode for all mobile verification code sending processes for frontend users
VERIFYCODE_FRONTEND_MOBILE_ALLOW_TEST_ALL= false

# Activate test mode for all mobile verification code sending processes for frontend users in single-step registration
VERIFYCODE_FRONTEND_CHECK_LOGIN_ALLOW_TEST_ALL= false

VERIFYCODE_FRONTEND_EMAIL_ALLOW_TEST_ALL= false

VERIFYCODE_BACKEND_MOBILE_ALLOW_TEST_ALL= false
VERIFYCODE_BACKEND_CHECK_LOGIN_ALLOW_TEST_ALL= false
VERIFYCODE_BACKEND_EMAIL_ALLOW_TEST_ALL= false
```

## 2025-8-19 - 2025-8-20 Update Categories Tags in API Version 2
Filters have been added to filter categories based on their association with objects. For example, if we want to fetch categories linked only to products, or categories linked only to stores, or categories linked only to news.

The following explains the linking filters:

```json
{
  "is_has_cateables": "Boolean variable to filter categories based on their association with objects",
  "cateable_type": "To specify the types of associated objects, it carries a string or array value",
  "cateables_count": "Numeric variable to specify the minimum number of linked objects"
}
```

For example, if we want to fetch the main categories linked only to products, with the condition that no single category is linked to less than one product, including the relationship for fetching subcategories, the filters would be as follows:

```json
{
  "isMain": 1,
  "is_has_cateables": 1,
  "cateables_count": 1,
  "cateable_type": "products",
  "include": "children"
}
```
or
```json
{
  "isMain": 1,
  "is_has_cateables": 1,
  "cateables_count": 1,
  "is_has_products": 1,
  "include": "children"
}
```

**See the following examples for further clarification: Example 1.1.3.3 and Example 1.1.3.4.**

## 2025-8-21 - 2025-8-22 Updates and Fixes for Problems in the Store and Categories Management Section
Various updates in the store and categories management section, as well as in the product management section, where some issues in managing and selecting stores have been fixed, and the creation of product types for new stores has been improved to occur automatically when a new store is created. 

The following settings have been added to control the display and visibility of the fields specific to the auction in the API section independently, allowing us to hide these fields in the backend interfaces and show them in the API part or as needed:

```
# Activate auction fields 
NANO_SHOP_ALLOW_HARAJ_FIELDS= false

# Activate user identification fields in the control panel, i.e., the owner of the ad; this property is affected by activating the auction fields
NANO_SHOP_ALLOW_USER_FIELDS= false

# Activate ID and license fields in the auction 
NANO_SHOP_ALLOW_HARAJ_LICENSE_FIELDS= false

# Activate start and end date fields in products and auction 
NANO_SHOP_ALLOW_FROM_TO_DATE_AT= false

# Activate auction fields in the API 
NANO_SHOPAPI_PRODUCTS_ALLOW_HARAJ_FIELDS= false

# Activate ID and license fields in the auction API 
NANO_SHOPAPI_PRODUCTS_ALLOW_HARAJ_LICENSE_FIELDS= false

# Activate start and end date fields in products and auction API 
NANO_SHOPAPI_PRODUCTS_ALLOW_FROM_TO_DATE_AT= false
```

## 2024-8-23 - 2024-8-24 Update Shop Products Config Mobile Fields
The product management section has been updated to allow control over phone number fields in products independently from auction fields, enabling the use of phone number fields in applications that do not require all auction fields. The documentation for fetching product data along with similar and complementary products in the api/v1/shop/similars/{id} section has been updated.

### The settings for controlling phone number fields in the product are as follows:
```
# Activate auction fields 
NANO_SHOP_ALLOW_HARAJ_FIELDS= false
```

You can refer to the following examples in the documentation for fetching product page data api/v1/shop/similars/{id}.

**Example 2.2.1 AND Example 2.2.2**

## 2025-8-25 - 2025-8-26 Various Updates to the Book List and Recharge Cards Section and Support FreeBooks Payment Provider
Various updates in the following sections:
- Update the list of books purchased by the client, where it has become possible to control the order states in which the books appear.
- Update the recharge card management section, where some issues have been resolved and the code has been improved with many fixes.
- A new payment method has been added for free books named FreeBooks.
- The fields specific to payment method details have been added to the order management interface, such as card type, card number, and payment process ID.

The following settings have been added to control order states and payment methods:

```
# Activate the payment method for free books 
NANO_MICROCART_PAYMENT_TYPES_ALLOW_FREE_BOOKS= true

# The default value for the order states in which the book is displayed to the client 
# The value "all" means all states after the purchase process
NANO_SHOPAPI_BOOKS_DEFAULT_ORDER_STATUS= '*'
```

## 2025-8-27 Update Reports Accounts Balance and Reports Accounts Balance Students
Update the reports of account balances and the reports of student account balances, where issues related to filtering the report by sub-balances of a certain main account have been fixed, and the code has been significantly improved.

## 2025-8-27 - 2025-8-31 Support Short URL Version v1
Launching the software module for creating and generating short links, which will be widely used in the reports section and the frontend section. Its importance lies in shortening long links, such as a specific report link, so it can be sent in a text message to clients, with the ability to set specific conditions on the link, such as the number of uses and tracking the number of visits, etc. The documentation for this part has been updated, and illustrative examples have been created. Work on this module has been ongoing intermittently since the beginning of 2025, and it has been continuously updated and tested until the launch date.

