# Monthly Achievement Report of NanoSoft Software Team  
## February 2026  

### **Introduction**  
This report provides a professional and accurate summary of the development team's achievements at NanoSoft Software Company during the period from February 1 to February 28, 2026. This month witnessed the launch and update of a wide range of software modules, focusing on enhancing security, expanding API capabilities, and improving the experience for developers and end users. Achievements were diverse, ranging from managing nationalities and languages, developing filtering tools, to launching specialized packages such as **Nano3.Legal** and **Nano3.Redactor**.

---

## **Executive Summary**  
During February 2026, the team completed **19 major updates** distributed across several strategic axes:  
1. **Expanding the scope of geographical and demographic data** by supporting nationalities, languages, and religions and managing them through unified API interfaces.  
2. **Improving filtering and search tools** in adverts, products, tags, and categories, while providing a professional filter manager.  
3. **Launching advanced security modules** such as **Nano3.Redactor** for sensitive data redaction, along with updates to **GDPR** and consent management.  
4. **Developing legal and administrative systems** through the launch of the integrated **Nano3.Legal** for legal documents.  
5. **Enhancing developer interfaces** by updating RouteBrowser, providing a unified form fields manager (**FormFieldsManager**), and comprehensive documentation for all APIs.  
All updates were made while maintaining backward compatibility and adding numerous illustrative examples.

---

## **Achievements and Updates Details**

### **1. Support for Nationalities, Languages, and Religions**  
**Period:** February 1 – 6, 2026  
**Description:** The **Nano.Location**, **Nano.UserPlus**, and **Nano.AuthApi** modules were updated to fully support the management of nationalities, languages, and religions.  
**Key Features:**  
- Creation of the `nano_location_nationalities` table with helper classes (`NationalityHelper`, `ReligionsHelper`).  
- Addition of nationality management interfaces in the control panel with an Auto Seed feature.  
- Development of new API endpoints:  
  - `GET /api/v1/location/nationalities` and its variants.  
  - `GET /api/v1/location/religions` and its variants.  
  - `GET /api/v1/location/languages` and its variants.  
- Added a `belongsTo` relationship for nationality in the frontend user model, and dropdown fields for nationality and language in the admin interfaces.  
- Comprehensive documentation update with practical examples.

---

### **2. Adverts System Development**  
**Period:** February 5, 2026  
**Description:** Updated **Nano.Adverts API v2** to support filtering by targeted objects.  
**Features:**  
- Added parameters `is_has_targets`, `target_type`, `target_id`.  
- Support for multiple object types: Order Types, Main Types, Stores/Departments, Categories, Tags, Products.  
- Comprehensive documentation with illustrative examples.

---

### **3. RoutesBrowser Updated to Version 3**  
**Period:** February 7, 2026  
**Description:** Upgraded the API browsing and testing tool to align with version 3, with the ability to export a `collection-api.json` file for import into tools like Postman.

---

### **4. Support for Visit Tracking in Orders Types (OrdersType)**  
**Period:** February 9, 2026  
**Description:** Updated **Nano.OrdersType** and **Nano.OrdersApi** to support visit tracking at the Orders Type level.  
**Additions:**  
- New environment settings: `NANO2_VISITORS_IS_SUPPORT_ORDERS_TYPES_SHOW` and `NANO2_VISITORS_IS_SUPPORT_ORDERS_TYPES_LIST`.  
- Ability to retrieve the visit count for each order type via an updated API.

---

### **5. Launch of Nano3.Packer Tools v2 (Code Encryption & Production Packaging)**  
**Period:** October 25, 2025 – February 10, 2026  
**Description:** Developed an integrated module for code processing and encryption, enabling the packaging and encryption of complete software modules in multiple ways (via a compressed file or by selecting a module within the system). It aims to streamline code preparation for production and protect intellectual property.

---

### **6. Tags System (Nano.Tags) and Filter Management Updates**  
#### **6.1. TagManager Development and tags Relationships**  
**Period:** February 11, 2026  
**Description:** Added **TagManager Class** to flexibly link any object or interface with tags, and supported `tags` relationships in `Types` and `Categories`.  
- Added `tagsId` filtering in `types` and `categories` endpoints in API v2.  
- Comprehensive documentation of the new mechanism.

#### **6.2. Launch of AvailableFilterManager**  
**Period:** February 23 – 25, 2026  
**Description:** Created the **AvailableFilterManager Class** (version 1.0.16) to uniformly manage filter data (loading, normalization, validation, merging, sorting, dynamic options).  
- Updated **Nano.TagsApi v2** by adding the endpoint `api/v1/tags/availablefilters` with support for `filter_id`.  
- Improved `children_count` and `sub_children_count` relationships in **CategorieTransformer**.

#### **6.3. Enhancement of step_fields in Types**  
**Period:** February 25 – 27, 2026  
**Description:** Modified the way product addition fields are returned to be an includable relationship (`include=step_fields`), with the setting `NANO_TAGSAPI_TYPES_IS_STOP_CONFIG_DATA_STEP_FIELDS` to stop returning them within `config_data`.  
- Updated documentation and examples.

---

### **7. Google Merchant Validator v2 Development**  
**Period:** February 12 – 19, 2026  
**Description:** Launched a specialized module to validate Google Merchant feed content, with a public interface at:  
`https://account.now-ye.com/nano2/googlemerchant/validator`  
**Additions:**  
- Classes `ValidationLibrary`, `GoogleMerchantFields`, `GoogleMerchantValidator`.  
- New validators: `DescriptiveTextValidator` (multilingual descriptive text), `NoPromotionalTextValidator` (promotional words), `LinkProcessor` (link processing).  
- Performance improvements and multilingual support.

---

### **8. GDPR and Cookie Management Module Update**  
**Period:** February 12, 2026  
**Description:** Added an experimental API for handling cookie consents.  
**New Endpoints:**  
- Accept/decline default cookies.  
- Check cookie validity.  
- Retrieve and update consent status.  
- Reset consents.  
- Fetch data retention policies.  
- All endpoints documented.

---

### **9. Launch of Nano3.Legal (Legal Documents)**  
**Period:** February 6 – 15, 2026  
**Description:** Developed an integrated module for managing legal documents (Privacy Policy, Terms of Service, Return Policy, Shipping Policy, Cookie Policy, User Agreement).  
**Features:**  
- Advanced classes for managing document types and classifications.  
- Multiple API endpoints:  
  - `GET /api/v1/legal/documents` and its variants.  
  - `GET /api/v1/legal/consents` and its variants.  
  - `GET /api/v1/legal/support/ref-types/{id?}` and `support/types/{id?}`.  
- Full documentation with classifications for document types.

---

### **10. Orders Requests and Products Updates**  
#### **10.1. OrdersRequests and OrdersLists API v2**  
**Period:** February 15, 2026  
**Description:** Added new filters: `is_user_or_delivery_user_id`, `companys_id`, `departments_id`. Supported backend user permissions.

#### **10.2. Product Options Support**  
**Period:** February 16 – 18, 2026  
**Description:** Updated **Nano.Shop** and **Nano.ShopApi** by adding filters:  
- `is_has_product_options` (filter products that have additional options).  
- `is_has_product_options_value` (filter based on existence of option values).  
- `product_options_values` (filter by a specific option value).  
- Documentation with practical examples.

---

### **11. Launch of Nano3.Redactor (Sensitive Data Redaction Tool)**  
**Period:** February 20 – 23, 2026  
**Description:** An advanced tool for automatically detecting and redacting sensitive data (passwords, API tokens, email, credit card numbers, etc.) before logging or displaying.  
**Adopted Strategies:**  
- Safe and blocked keys (including Wildcard).  
- Text patterns (email, SSN, credit card).  
- Entropy-based redaction (Shannon Entropy) to detect random strings.  
- Redaction based on object size or long text.  
**Profiles:** default, strict, performance.  
**Benefits:** privacy protection, developer time savings, improved log quality, file scanning tool (Artisan Command) to search for sensitive data.  
- Easy integration via Facade or container service.

---

### **12. Addition of FormFieldsManager and Available Fields API**  
**Period:** February 24 – 28, 2026  
**Description:** Developed a unified class **FormFieldsManager** (in **Nano.Tags**) to manage form fields from multiple sources (array, JSON, config file) with multilingual support (`ar`, `en`, `fr`).  
**Features:**  
- Field normalization, validation, merging, sorting, searching, and dynamic options management (from API, database, custom PHP functions).  
- Added helper Trait `SettingsFormFields` to facilitate reuse.  
- Updated configuration file `config/nano/tags/form_fields.php` with ready-made field examples.  
- Added a new endpoint in **Nano.TagsApi v2**: `api/v1/tags/availableformfields` with support for `filter_id`.  
- Comprehensive documentation of supported field types and request/response examples.

---

### **13. Other Updates**  
- **Map fields improvement** in the backend (February 8).  
- **Notes system (Tss.Notes) update** to support all versions (mentioned in January report, compatibility ensured).  

---

## **Conclusion and Recommendations**  
### **Achieved Successes:**  
1. **Strategic Expansion:** Launching two new modules, **Nano3.Legal** and **Nano3.Redactor**, covering important legal and security needs.  
2. **Improved Developer Experience:** Unifying filter and field management via `AvailableFilterManager` and `FormFieldsManager` reduces redundancy and accelerates development.  
3. **Greater API Flexibility:** Adding advanced filters in adverts, products, and tags enables building smarter applications.  
4. **Enhanced Documentation:** Providing examples and comprehensive documentation for each update facilitates adoption by other teams.  
5. **Enhanced Security:** Tools like Redactor and GDPR protect sensitive data and ensure compliance with standards.  

### **Recommendations for the Next Phase:**  
1. **Internal Team Training:** Conduct workshops on using `FormFieldsManager` and `AvailableFilterManager` to spread the benefits.  
2. **Redactor Performance Review:** After integrating it into projects, measure its impact on system performance and refine strategies if necessary.  
3. **Expand Visual Documentation:** Produce short videos explaining new features, especially for complex modules like Nano3.Legal.  
4. **Continue Migrating Settings to Admin Interfaces:** As done in January, to facilitate control without needing to modify files.  
5. **Prepare an Upgrade Plan for Existing Projects** to leverage improved APIs and new filters.  

---

**Report Prepared by:** Technical Documentation Team  
**Release Date:** February 28, 2026  
**Classification:** Internal Achievements Report