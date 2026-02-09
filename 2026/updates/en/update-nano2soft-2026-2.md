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
