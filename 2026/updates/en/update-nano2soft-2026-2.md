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

