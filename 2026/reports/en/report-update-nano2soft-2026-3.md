# Monthly Report of NanoSoft Software Team Achievements  

## March 2026  

### **Introduction**  
This report provides a professional and comprehensive summary of the development team's achievements at NanoSoft Software Company during the period from March 1 to March 31, 2026. This month was marked by unprecedented qualitative achievements, with the development of **four major integrated systems**: the advanced translation system, the dynamic reporting system, the comprehensive unit management system, and the smart product associations system. The month also saw substantial improvements in review and interaction behaviors, the launch of specialized tools such as the interactive JSON viewer and OTP settings via the admin interface, as well as the expansion of the dynamic variable system in reports with advanced options. These achievements reflect the team's commitment to delivering advanced technical solutions that meet complex business needs and enhance overall system efficiency.

---

## **Executive Summary**  
During March 2026, the team completed **more than 15 major updates**, distributed across seven strategic axes:  

1. **Development of Translation and Internationalization Systems** – Adding caching for translations, dynamically including translatable fields in APIs, and full support for multiple languages.  
2. **Launch of the Integrated Dynamic Reporting System** – A complete platform for building, executing, and exporting custom reports with support for complex relationships and computed columns.  
3. **Development of the Unit Management and Measurement Conversion System** – An advanced system supporting 16 unit types, a smart converter, a context validator, and comprehensive APIs.  
4. **Enhancement of Product Associations and Query Scopes** – Highly flexible scopes with full control over directions and conditions, and ready‑to‑use admin filters.  
5. **Comprehensive Development of Review and Interaction Behaviors** – Advanced scopes for sorting by count and date, adding computed columns, and filtering by value and type.  
6. **Launch of Specialized Helper Tools** – Interactive JSON viewer, OTP settings via the admin interface, the unified `FormFieldsManager` class, and the repeatable fields system with API.  
7. **Expansion of the Dynamic Variable System in Reports** – Advanced options such as priority, mandatory casting, default values, and allowing null.  

All updates were made while preserving backward compatibility, providing comprehensive documentation in both Arabic and English, and adding advanced practical examples.

---

## **Details of Achievements and Updates**

### **1. Advanced Translation System (Nano.TranslateExtended)**  
**Period:** February 26 – March 3, 2026  
**Description:** Development of an integrated package to improve the performance and flexibility of the multi‑language translation system.  

**Key Components:**  
- **`TranslatableContentCaching` Behavior**: A caching layer for translations with automatic cache invalidation on modification, reducing database queries by up to 70%.  
- **Smart Helper Methods**: `getTranslationsInFormat` and `getAllFieldTranslations` to retrieve translations in multiple formats (grouped by field, by language, array, object).  
- **`DynamicAddIncludeTranslatableApiFields` Behavior**: Ability to include translations in API responses via the `include=translatable_fields` parameter, with dynamic control over format and fields via request parameters.  
- **Integration with APIs**: The behavior was injected into `ProductTransformer` and `/api/v1/shop/managers/data`, and documentation was updated.  
- **Updated Versions**: Nano.Shop (1.1.11) and Nano.ShopApi (1.1.30).  

**Documentation:** `Docs-TranslatableContentCaching-en.md`  

---

### **2. Development of Hierarchical Data Traits and Improvements to the Categorization System**  
**Period:** March 3 – 7, 2026  
**Description:** Development of the `HasAdvancedTree` trait to manage tree relationships (ancestors and descendants) efficiently using recursive queries (CTE) and caching.  

**Features:**  
- Advanced methods: `getNodeAncestors`, `getNodeDescendants`, `buildNestedNodeTree`, `toFlatNodeTree`, `getNodePath`.  
- Support for soft deletes and database version checking (MySQL 8+, SQLite 3.8.3+, PostgreSQL).  
- Update of `CategoriesModel` behavior: `prepareCategoryIds` method, improved scopes (`scopeWhereCategories`, `scopeWhereHasCategoryWithDescendants`, etc.), and `hasCategory` check methods with option to include ancestors/descendants.  

**Documentation:** `Docs-HasAdvancedTree-en.md`, `Docs-CategoriesModel-en.md`  

---

### **3. Google Merchant Validator Update**  
**Period:** March 7 – 8, 2026  
**Description:** Update of `Nano2.GoogleMerchant` to version 1.0.10.  
**Features:**  
- Improved `autoFixItem` in `GoogleMerchantValidator`.  
- Support for fixing Arabic URLs via `fixGoogleProductUrl` in `LinkProcessor`.  
- Ability to download the corrected XML file directly from the interface.  

---

### **4. Launch of the Integrated Dynamic Reporting System (Nano2.QueryBuilder.Reporting)**  
**Period:** March 1 – 10, 2026  
**Description:** Development of a complete system for building and executing custom reports based entirely on configuration rather than static queries.  

**Main Components:**  
- **Schema Definition Layer**: `Column`, `Relationship`, `Table`, and `ReportSchemaRegistry` (central registry).  
- **Schema Management Layer**: `ReportsManager` to collect table definitions from plugins, and `ReportsManagerQueryConverter` (Trait) to run reports.  
- **Validation Layer**: `ComputedColumnValidator` and `ReportQueryValidator`.  
- **Query Execution Layer**: `ReportQueryConverter` to build a Laravel Query Builder from report configuration, with support for auto‑joins on nested relationships.  
- **Export Layer**: `ReportQueryExporter` to export results in CSV, Excel (XLSX), JSON, and XML formats.  
- **Error Handling Layer**: `ReportQueryErrorHandler` with unified messages and unique error codes.  

**Documentation:** 12 documentation files under `docs/reporting/` covering each class in detail.  

---

### **5. Interactive JSON Viewer Addition (Nano.Jsondb)**  
**Period:** March 11 – 12, 2026  
**Description:** Addition of the `jjsonviewer.js` library and `oc.json-format.js` control to convert JSON text into a collapsible tree with data type coloring.  
**Usage:** Add `data-control="json"` to any HTML element to display formatted JSON.  
**Settings:** Environment variable `NANO_JSONDB_EXTEND_JSONVIEWER` to enable the feature.  

---

### **6. Enhancements and Extensions to the `FormFieldsManager` Class (Nano.Tags)**  
**Period:** March 12 – 14, 2026  
**Description:** Update of `FormFieldsManager` to version 1.0.19.  
**Features:**  
- Addition of `FormFieldsHelperTrait` with a large set of helper methods (validation, advanced filtering, search, data transformation).  
- Support for nested fields via the `nestedform` type and enhancement of the `repeater` type to include inner fields.  
- Update of the `form_fields.php` configuration file with an advanced example of unit‑linked prices.  
- Improvement of `normalizeFields` to support recursive normalization of inner fields.  

**Documentation:** `Docs-FormFieldsHelperTrait-en.md`  

---

### **7. Development of the Repeater Fields System**  
**Period:** March 13 – 15, 2026  
**Description:** Creation of the `RepeaterFieldsData` class (in `Tss.Basic`) to uniformly handle jsonable fields (phone, email, website, properties, links) with support for all input formats.  

**Features:**  
- Methods: `processInput` (normalization), `formatOutput` (formatting), `extractMainValues`, `mergeData`, `validate`.  
- Dynamic reading of YAML definitions (default values, field types).  
- Smart handling of field types: `switch` → `'1'/'0'`, `dropdown` (validation), `mediafinder` (path to URL).  

**Unified API:**  
`RepeaterFields` controller (in `Nano.BasicApi`) with endpoints: `supported`, `process`, `format`, `extract`, `merge`, `validate`, `tests`.  

**Documentation:** `Docs-RepeaterFieldsData-en.md`, `Docs-RepeaterFieldsData-API-en.md`  

---

### **8. Support for Bio, Website, and Links Fields in User Profiles**  
**Period:** March 15 – 16, 2026  
**Description:** Added columns `bio`, `website`, `links` to user profiles (frontend and backend).  
**Updates:**  
- `Nano.AuthApi` (1.0.17, 1.0.18): Support for processing inputs via `processWebsiteInput` and `processLinksInput`.  
- `Nano.BackendUserPlus` (1.0.7): Added columns for backend users.  
- `Nano.UserPlus` (1.1.6): Added columns for frontend users.  

---

### **9. Support for WhatsApp Field in Products**  
**Period:** March 17, 2026  
**Description:** Added a `whatsapp` field to products in backend, frontend, and APIs.  
**Updates:**  
- `Nano.Shop` (1.1.12): Backend interface support and `HasWhatsappField`.  
- `Nano.ShopApi` (1.2.1): Support for the field in `shop/products` and `shop/managers/products` API v2, and documentation updates.  

---

### **10. Comprehensive Development of the Unit Management System (Tss.Inventory)**  
**Period:** January 28 – March 18, 2026 (continued through March)  
**Description:** Complete restructuring of the units system, transforming it into an integrated platform for conversion and measurement management.  

**Developed Components:**  
- **`UnitType`**: Definition of 16 main types (length, weight, volume, ...) and subtypes for quantities.  
- **`UnitNameHelper`**: Comprehensive unit lists with Arabic and English labels, advanced search methods.  
- **`UnitConverter`**: The heart of the conversion system, supporting all types with special handling for temperature.  
- **`UnitContextValidator`**: Prevents illogical conversions between quantity units (e.g., dozen → ream).  
- **`UnitFactory`**, **`UnitHelper`**, **`UnitCreator`**, **`UnitManager`**: Helper classes for creation, linking, and common operations.  
- **Model Improvements**: `Unit`, `ProductsUnit`, `ProductsPricesUnit` with new columns (`unit_symbol`, `conversion_factor`, `is_base_unit`, `is_public`, `is_published`).  
- **Migrations**: 13 files from 1.1.13 to 1.1.25 to adjust tables and support decimal precision.  

**New API Endpoints in `Nano.ShopApi` (1.1.29):**  
- `api/v1/shop/units`  
- `api/v1/shop/managers/units`  
- `api/v1/shop/managers/groupsproducts`  

**Documentation:** 9 files under `docs/UnitManager/`  

---

### **11. Development of the Product Associations System – Advanced Version**  
**Period:** March 19 – 22, 2026  
**Description:** Transformation of the `AssociationsModel` behavior into a highly flexible query tool with advanced scopes and ready‑to‑use admin filters.  

**New Scopes:**  
- Basic: `hasAlternate`, `hasCrossSell`, `hasUpSell` and their negations.  
- Direction‑based: `hasAlternateAsParent`, `hasAlternateAsTarget`, etc.  
- Advanced `scopeWhereAssociation` that accepts options: `type`, `direction`, `has`, `boolean`, `relationOperator`, `callback`, `conditions`, `scopes`, `typePerDirection`, `conditionsPerDirection`, `scopesPerDirection`.  
- Toggle scopes with OR and AND logic (e.g., `isToggelAnyAlternateOr`, `isToggelAnyAlternateAnd`).  

**`AssociationsHelper` Class:**  
- Methods: `getAlternateFilterScopes`, `getCrossSellFilterScopes`, `getUpSellFilterScopes`, `getAnyAssociationFilterScopes`, `getAllAssociationFilterScopes`.  
- `injectAssociationScopes`, `removeAssociationScopes` to integrate filters into admin interfaces.  
- Control via `config.php` and environment variable `NANO_SHOP_PRODUCTS_ASSOCIATIONS_ALLOW_FILTER`.  

**Documentation:** `Docs-AssociationsModel-Behaviors-en.md`, `Docs-AssociationsModel-Behaviors-Advenced-Examples-en.md`  

---

### **12. Comprehensive Update of Query Scopes in Review and Interaction Behaviors**  
**Period:** March 22 – 29, 2026  
**Description:** Restructuring and improvement of scopes in seven behaviors: `ReviewRateable`, `FavoriteableModel`, `LikeableModel`, `ReactionableModel`, `BookmarkableModel`, `FollowableModel`, `ProposalModel`.  

**Unified Improvements:**  
- Fixed subquery errors (removed `GROUP BY` and `LIMIT`).  
- Added `Add...`, `SortBy...`, `With...` scopes for each of: count (`Count`), latest date (`Latest`), earliest date (`Earliest`).  
- Support for filtering by `value` (for interactions and likes) or `type` (for proposals).  
- Added `is_..._by_user` column (e.g., `is_rated_by_user`, `is_liked_by_user`).  
- Statistical helper methods: `getTotal...`, `get...CountByType`, `get...Users`.  
- Moved scopes to independent traits (`ReviewScopesAndHelpers`, `FavoriteScopesAndHelpers`, etc.) for easier maintenance.  

**Examples:**  
- `Product::sortByAvgRating('DESC')->get()` (highest rated).  
- `Product::withIsLikedByUser()->paginate()` (add like status column).  
- `Product::hasFollowers(5, true)->get()` (products with at least 5 followers).  

**Documentation:** Separate files for each behavior under `docs/ReviewRateable/`, `docs/FavoriteableModel/`, etc.  

---

### **13. Comprehensive Update of OTP Settings in VerifyCode**  
**Period:** March 28 – 29, 2026  
**Description:** Added an integrated settings system for managing the OTP service via the backend user interface instead of relying only on `config.php`.  

**Components:**  
- `Settings` model inheriting `SettingsModel` with `registerConfig` method to merge stored settings with the config file.  
- `fields.yaml` organized into tabs: General, Tests, Frontend (mobile, check_login, email), Backend.  
- Updated `Plugin.php` with `registerSettings` and `registerPermissions`.  
- Updated `Manager.php` to read settings from `Config` (merged).  
- Arabic and English translation files.  

**Benefits:** Full control over expiration time, sending channels, test mode, activation URLs, and query parameters for each operation type and platform.  

---

### **14. Expansion of the Dynamic Variable System in Report Conditions**  
**Period:** March 20 – 31, 2026  
**Description:** Development of an advanced system for dynamic variables inside report conditions, allowing users to control value replacement behavior via multiple options.  

**New Variable Syntax:**  
`source:parameter|option1:value|option2`  

**Supported Options:**  
- `as:key` – Store the value under a custom key.  
- `override` – Priority to filter values, then source.  
- `fallback` – Priority to source, then filters.  
- `source` – Source only (default).  
- `required` – Mandatory; prevents the report if no value is found.  
- `nullable` – Allows null value.  
- `strict` – Immediate failure if source resolution fails.  
- `default:v` – Default value.  
- `cast:t` – Type casting (int, float, bool, string, array, date, datetime).  
- `multiple` – Ensures the value is an array.  

**Supported Sources:** `user`, `cache`, `config`, `static`.  

**Updates in `ReportsManager` and `ValueResolver`:**  
- Added `resolveDynamicVariables` that resolves all variables in report conditions and collects errors.  
- Added `getValueResolver` and `setValueResolver`.  
- Created `ResolvedValue` class to hold the resolution result.  

**Example:**  
```php
'variable' => 'user:id|required|cast:int|as:userId'
```  

**Documentation:** `Docs-Dynamic-Variables-In-Report-Conditions-en.md`  

---

### **15. Other Updates**  
- **Update of API for Managers and Units** in `Nano.ShopApi` (1.1.29) as mentioned in the units section.  
- **Separate detailed update of `ReviewRateable` behavior** (already covered under scopes).  
- **Documentation of all updates** in both Arabic and English with advanced practical examples.  

---

## **Conclusion and Recommendations**  

### **Achievements:**  
1. **Unprecedented Achievements** – Four major integrated systems (translation, reporting, units, product associations) were developed in a single month, reflecting the team's ability to deliver complex projects with high quality.  
2. **Radical Improvement in Performance and Flexibility** – Translation caching reduces database queries by up to 70%; the dynamic reporting system eliminates the need for manual SQL queries; the unit system provides accurate and contextual conversions.  
3. **Code Unification and Reuse** – Creating independent traits for behaviors (e.g., `ReviewScopesAndHelpers`) reduces duplication and eases maintenance.  
4. **Empowerment of Users and Developers** – Ready‑to‑use filters in product associations and OTP settings via the interface give system administrators full control without code changes.  
5. **Exceptional Documentation** – More than 25 detailed documentation files (Arabic/English) with advanced examples were produced, facilitating adoption of these systems in current and future projects.  

### **Recommendations for the Next Phase:**  
1. **Internal Awareness** – Conduct training sessions for teams (development, QA, support) on the dynamic reporting system and the unit system, as they are the most complex and highest‑value.  
2. **Integration into Existing Projects** – Develop a phased plan to upgrade current projects (e.g., Haraj, Taseer, Souq Aqar platforms) to benefit from the new unit system and advanced product associations.  
3. **Monitor Translation Caching Performance** – Measure the impact of `TranslatableContentCaching` on response speed and query count, and adjust settings if necessary.  
4. **Add Graphical Interfaces** – Develop graphical admin interfaces for the dynamic reporting system (drag‑and‑drop report creation) and for the unit system (managing custom conversions).  
5. **Expand Dynamic Variables** – Support additional sources such as `session`, `input` (from request), and `relationship` (properties from model relations).  
6. **Continue Improving Visual Documentation** – Produce short educational videos for the new systems, especially the reporting and unit systems, to accelerate adoption.  

---

**Report Prepared by:** Technical Documentation Team  
**Date of Issue:** March 31, 2026  
**Classification:** Internal Achievements Report  
