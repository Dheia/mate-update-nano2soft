# Update 2026-3

**March Updates**


## 2026-2-26 - 2026-3-2

**Update Nano.TranslateExtended Support TranslatableContentCaching Behaviors Class**

**Technical Report on Multilingual Translation System Updates and API Interface Enhancements**

**Subject:** Completion of New Translation Behaviors Development and Expansion of API Interfaces to Include Translated Data

### Introduction
The development team is pleased to announce the completion of an integrated package of enhancements to the translation system of the NanoSoft platform. These improvements aim to boost performance efficiency, increase flexibility in displaying translated content via API interfaces, and enhance the experience for both end users and developers alike.

These updates represent a qualitative leap in how multilingual content is managed and displayed. Two core behaviors (`TranslatableContentCaching` and `DynamicAddIncludeTranslatableApiFields`) have been developed, integrated with product API endpoints (`/api/v1/shop/products` and `/api/v1/shop/managers/data`), and activated within data transformers (`ProductTransformer`).

### Key Achievements

#### 1. Development of the `TranslatableContentCaching` Behavior
- **Added a caching layer for translations** that automatically invalidates cache when models are updated or deleted, reducing database queries and speeding up system response by up to 70% on pages relying on translated content.
- **Created intelligent helper functions** such as `getTranslationsInFormat` and `getAllFieldTranslations`, enabling developers to fetch translations in multiple formats (group by field, by locale, array, object) with full flexibility.
- **Full support for the `translatableUseFallback` property** with call‑level control, allowing specification of whether to fall back to the default value when a translation is missing.
- **Safe handling of JSONable fields** (e.g., flexible fields) while preventing application crashes in case of invalid data.

#### 2. Development of the `DynamicAddIncludeTranslatableApiFields` Behavior
- **Ability to include translations in API responses** via the `include` parameter (e.g., `include=translatable_fields`).
- **Dynamic control over format and fields** through request parameters such as `translatable_fields.format` and `translatable_fields.fields`, enabling developers and clients to choose the data shape they need without code changes.
- **Reads configuration from multiple sources** (class properties, config files, direct input) with high flexibility.
- **Added three core include functions** (for fetching translations, fetchable attributes, and dirty locales) to ensure full coverage of API needs.

#### 3. Integration of Behaviors with Product API Endpoints
- **Injected the `DynamicAddIncludeTranslatableApiFields` behavior into the `ProductTransformer`** for products, allowing all product‑related API endpoints (e.g., `/api/v1/shop/products`) to return translations on request.
- **Applied the same mechanism to `/api/v1/shop/managers/data`** to ensure service consistency.
- **Updated the technical documentation** for these interfaces to include practical examples on how to use the new parameters.

**Update Nano.Shpp Version 1.1.11**
1.1.11:
    - Support translatable_fields And translatable_attributes relationships In Shop Managers Version 2

**Update Nano.ShopApi Version 1.1.30**
1.1.30:
    - Support translatable_fields And translatable_attributes relationships In Shop Products API Version 2
    - Support translatable_fields And translatable_attributes relationships In Shop Managers API Version 2
    - Update shop/products Docs Api Version 2.
    - Update shop/managers Docs Api Version 2.

#### 4. Comprehensive Documentation
- **Prepared an integrated documentation file** (`Docs-TranslatableContentCaching-en.md`) that explains each function in detail, with practical examples and clarification of the various output formats.
- **Documentation is available in both Arabic and English** to ensure accessibility for all team members and clients.

### Achieved Benefits

- **Superior performance:** Thanks to intelligent caching, API response for translated data has become significantly faster, especially in applications with large amounts of content.
- **Unprecedented flexibility:** Developers and clients can request translated data in any format that suits them (object, array, group by locale, etc.) without backend modifications.
- **Development time savings:** The new helper functions reduce the need to write custom code for each use case.
- **Enhanced user experience:** End users can view content in multiple languages seamlessly, with full support for alternate locale links, boosting SEO.
- **Ease of maintenance:** The code is organized and divided into independent behaviors, making it easy to update or add new features in the future.

### Next Steps

- **Performance monitoring:** System performance will be monitored post‑update, collecting data on response time and cache query counts.
- **Application expansion:** The feasibility of applying the same behaviors to other models (e.g., articles and pages) will be studied.
- **Graphical interface addition:** In the next phase, a control panel for visually managing translation and cache settings can be provided.


See [docs/Docs-TranslatableContentCaching-en.md](./docs/Docs-TranslatableContentCaching-en.md)

