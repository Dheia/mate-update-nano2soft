# `AssociationsModel` Behavior Documentation

**Namespace:** `Nano\Shop\Behaviors`  
**Helper:** `Nano\Shop\Classes\AssociationsHelper`

---

## Introduction

`AssociationsModel` is a **Behavior** designed for models in NanoSoft applications (such as `Product`). It provides functionality for **managing associations between products** such as:

- **Alternate Products** (Alternate)
- **Complementary Products** (Cross‑Sell)
- **Newer Versions** (Up‑Sell)

In addition to the basic functions, the behavior offers a **sophisticated set of scopes** enabling:

- Filtering by **relationship type**, **relationship direction** (parent, target, both), and **presence/absence** of the association.
- **Additional conditions** on the association table (e.g., `is_active`, `is_public`, `companys_id`, `departments_id`).
- **Applying scopes** specific to the association table (e.g., `isCompany`, `isDepartment`).
- **Customizing conditions per direction** separately (`conditionsPerDirection`, `scopesPerDirection`, `typePerDirection`).
- **Full control over the relationship logic** (`AND` / `OR`) between the main conditions and between different directions.

It also provides the helper class `AssociationsHelper` with ready-made tools for **injecting association filters** into admin interfaces (like product lists) easily and flexibly.

---

## Setup and Installation

The behavior does not require additional installation; it is part of the `Nano.Shop` package. To use it, you need to add it as a Behavior to the desired model (e.g., the `Product` model).

#### 1. Add the behavior directly to the model (in the model file)

```php
class Product extends Model
{
    public $implement = [
        'Nano.Shop.Behaviors.AssociationsModel'
    ];

    // ... rest of model definitions
}
```

#### 2. Inject the behavior into a model from another extension (dynamically)

```php
// In the extension's boot file
public function boot()
{
    \Nano\Shop\Models\Product::extend(function($model) {
        if (!$model->isClassExtendedWith('Nano.Shop.Behaviors.AssociationsModel')) {
            $model->implementClassWith('Nano.Shop.Behaviors.AssociationsModel');
        }
    });
}
```

> **Note:** The behavior automatically adds the relationships `associations` (hasMany on `product_parent_id`) and `inverseAssociations` (hasMany on `product_target_id`).

---

## Table Structure and Relationships

The behavior relies on an intermediate table `nano_shop_product_associations` (model `ProductsAssociation`) which contains:

| Field | Type | Description |
|-------|------|-------------|
| `id` | bigIncrements | Primary key |
| `type` | string(50) | Association type: `cross_sell`, `up_sell`, `alternate` |
| `product_parent_id` | bigInteger | Parent product in the relationship |
| `product_target_id` | bigInteger | Target product in the relationship |
| `is_active` | boolean | Is the association active (default true) |
| `is_public` | boolean | Is the association public (default true) |
| `companys_id` | string | Company ID (for multi-tenancy) |
| `departments_id` | string | Department ID (for multi-tenancy) |
| `sort_order` | integer | Display order |
| `created_at`, `updated_at`, `deleted_at` | timestamp | Timestamps |

**Relationships automatically added to the model:**

- `associations` : `hasMany` – associations where the model is the **parent**.
- `inverseAssociations` : `hasMany` – associations where the model is the **target**.
- `associations_products` : `belongsToMany` – related products as children.
- `inverseAssociations_products` : `belongsToMany` – products related to it as target.

---

## Available Scopes

The behavior provides multiple scopes ranging from the simplest to the most complex.

### 1. Ready-made scopes for specific types (without options)

| Scope | Description |
|-------|-------------|
| `hasAlternate()` | Products that have alternatives (in any direction) |
| `hasCrossSell()` | Products that have complements |
| `hasUpSell()` | Products that have newer versions |
| `hasNotAlternate()` | Products that do not have alternatives |
| `hasNotCrossSell()` | Products that do not have complements |
| `hasNotUpSell()` | Products that do not have newer versions |

### 2. Direction-based scopes (per type)

| Scope | Description |
|-------|-------------|
| `hasAlternateAsParent()` | Products that have alternatives as parent |
| `hasAlternateAsTarget()` | Products that are alternatives (as target) |
| `hasCrossSellAsParent()` | Products that have complements as parent |
| `hasCrossSellAsTarget()` | Products that are complements (as target) |
| `hasUpSellAsParent()` | Products that have newer versions as parent |
| `hasUpSellAsTarget()` | Products that are newer versions (as target) |

### 3. Scopes for any association (presence/absence)

| Scope | Description |
|-------|-------------|
| `hasAnyAssociationBoth()` | Products that have any association (in any direction) |
| `hasNotAnyAssociationBoth()` | Products that have no associations |

### 4. The advanced `scopeWhereAssociation`

This is the core scope that controls all functionalities. It can be called directly or via the helper scopes. Syntax:

```php
public function scopeWhereAssociation($query, $options = [], $boolean = 'and', $not = false)
```

**`$options` parameter (array) details:**

| Option | Type | Description |
|--------|------|-------------|
| `type` | string\|array\|null | Relationship type (e.g., `'alternate'`) or array (`['alternate', 'cross_sell']`). |
| `direction` | string\|array | Direction: `'parent'`, `'target'`, `'both'` or array of directions. (default `'both'`) |
| `has` | bool | `true` = existence (whereHas), `false` = non-existence (whereDoesntHave). (default `true`) |
| `boolean` | string | `'and'` or `'or'` to combine this condition with the main query. (default `'and'`) |
| `relationOperator` | string\|null | `'and'` or `'or'` to combine multiple directions. If not specified: `'or'` if `has = true`, and `'and'` if `has = false`. |
| `callback` | callable\|null | Function (relationship query, direction name) for additional customization per direction. |
| `callbacks` | array | Array per direction: `['parent' => callable, 'target' => callable]`. |
| `typePerDirection` | array | Customize relationship type per direction: `['parent' => 'alternate', 'target' => 'cross_sell']`. |
| `conditions` | array\|callable\|null | Additional conditions on the relationship table (e.g., `['is_active' => 1]`). |
| `scopes` | array | Scopes to apply to the relationship query (e.g., `['isCompany', 'isDepartment' => [1]]`). |
| `conditionsPerDirection` | array | Conditions specific to each direction: `['parent' => ['is_active' => 1], 'target' => ['is_public' => 1]]`. |
| `scopesPerDirection` | array | Scopes specific to each direction: `['parent' => ['isCompany'], 'target' => ['isDepartment' => [1]]]`. |

**Parameter `$boolean`** – determines how to combine this scope with any previous conditions in the query.

**Parameter `$not`** – if `true`, it flips `has` (i.e., becomes `has = false`) unless `has` is explicitly passed in the options.

### 5. Helper Scopes

To simplify usage, the behavior provides functions that wrap `scopeWhereAssociation`:

| Scope | Description |
|-------|-------------|
| `scopeOrWhereAssociation($query, $options, $not = false)` | Combine condition with `OR` to the main query |
| `scopeWhereNotAssociation($query, $options, $boolean = 'and')` | Negative condition (doesntHave) |
| `scopeOrWhereNotAssociation($query, $options)` | Negative condition with `OR` |
| `scopeHasAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | Existence of a specific type (shortcut) |
| `scopeHasNotAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | Non-existence of a specific type |
| `scopeHasAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | Existence of any association (any type) |
| `scopeHasNotAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | Non-existence of any association |

> **Note:** The helper functions above do not directly support advanced options (`conditions`, `scopes`, `typePerDirection`, ...). If you need these options, use `scopeWhereAssociation` directly.

### 6. Toggle Scopes (for interface filters)

| Scope | Description |
|-------|-------------|
| `scopeIsToggelAnyAssociation($query, $value)` | `1` → products with any association; `0` → products without associations |
| `scopeIsToggelAnyAlternate($query, $value)` | Same logic for alternatives |
| `scopeIsToggelAnyCrossSell($query, $value)` | For complements |
| `scopeIsToggelAnyUpSell($query, $value)` | For newer versions |

---

## Practical Examples of Advanced Scopes

### Example 1: Products that have **active** alternative associations (`is_active = 1`) in any direction

```php
Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => ['is_active' => 1]
])->get();
```

### Example 2: Products that have alternative associations **as parent** only, with condition `is_public = 1`

```php
Product::whereAssociation([
    'type' => 'alternate',
    'direction' => 'parent',
    'conditions' => ['is_public' => 1]
])->get();
```

### Example 3: Products that **do not have** any `cross_sell` association in any direction, but may be related by other types

```php
Product::whereAssociation([
    'type' => 'cross_sell',
    'has' => false,
    'direction' => 'both'
])->get();
```

### Example 4: Products that have an `alternate` association **as parent** **or** an `up_sell` association **as target** (using `typePerDirection`)

```php
Product::whereAssociation([
    'direction' => ['parent', 'target'],
    'typePerDirection' => [
        'parent' => 'alternate',
        'target' => 'up_sell'
    ],
    'relationOperator' => 'or'   // default because has = true
])->get();
```

### Example 5: Products that have associations in both directions at the same time (AND between directions)

```php
Product::whereAssociation([
    'direction' => 'both',
    'relationOperator' => 'and',  // Force AND
    'conditions' => ['is_active' => 1]
])->get();
```

### Example 6: Using `scopes` on the relationship table (e.g., `isCompany` and `isDepartment`)

```php
Product::whereAssociation([
    'type' => 'alternate',
    'scopes' => ['isCompany', 'isDepartment' => [1]]
])->get();
```

### Example 7: Conditions specific to each direction using `conditionsPerDirection`

```php
Product::whereAssociation([
    'direction' => 'both',
    'conditionsPerDirection' => [
        'parent' => ['is_active' => 1],
        'target' => ['is_public' => 1]
    ]
])->get();
```

### Example 8: Combining `scopesPerDirection` with `typePerDirection`

```php
Product::whereAssociation([
    'direction' => 'both',
    'typePerDirection' => [
        'parent' => 'alternate',
        'target' => 'cross_sell'
    ],
    'scopesPerDirection' => [
        'parent' => ['isCompany'],
        'target' => ['isDepartment' => [1]]
    ]
])->get();
```

### Example 9: Using a custom `callback` to modify the relationship query

```php
Product::whereAssociation([
    'type' => 'alternate',
    'callback' => function($query, $direction) {
        $query->where('created_at', '>', now()->subDays(30));
    }
])->get();
```

### Example 10: Using `orWhereAssociation` with options

```php
Product::where('price', '>', 100)
    ->orWhereAssociation([
        'type' => 'alternate',
        'conditions' => ['is_active' => 1]
    ])
    ->get();
```

### Example 11: Using `whereNotAssociation` to negate associations with conditions

```php
Product::whereNotAssociation([
    'type' => 'cross_sell',
    'direction' => 'both',
    'conditions' => ['is_public' => 1]
])->get();
```

---

## The Helper Class `AssociationsHelper`

Located in `Nano\Shop\Classes\AssociationsHelper`, it simplifies managing association filters in admin interfaces (`ListController` / `Filter`).

### Main Functions

| Function | Description |
|----------|-------------|
| `getAlternateFilterScopes($defaultValue = 0, $is_force = false)` | Returns array of definition for alternative filters (checkbox + toggle) |
| `getCrossSellFilterScopes(...)` | Same for complements |
| `getUpSellFilterScopes(...)` | For newer versions |
| `getAnyAssociationFilterScopes(...)` | For "any association" filter (switch) |
| `getAllAssociationFilterScopes(...)` | Combines all the above filters |
| `injectAssociationScopes($filter, $types = null, $defaultValue = 0, $is_force = false)` | Injects specified filters into the filter object |
| `removeAssociationScopes($filter, $types = null)` | Removes specific filters |
| `isTypeEnabled($type)` | Checks if a specific type is enabled via settings |

**Parameter `$types` in `injectAssociationScopes` and `removeAssociationScopes`:**
- `null` or `false`: adds nothing.
- `true`: adds all types (`any`, `alternate`, `cross_sell`, `up_sell`).
- String like `'any,alternate'`: adds the mentioned types.
- Array: `['alternate', 'up_sell']`.

### Example of using the helper in a controller

```php
use Nano\Shop\Classes\AssociationsHelper;

public function boot()
{
    \ShopProductsController::extendListFilterScopes(function($filter) {
        $allowFilter = \Config::get('nano.shop::products.associations.allow_filter', false);
        if ($allowFilter && class_exists(AssociationsHelper::class)) {
            AssociationsHelper::injectAssociationScopes($filter, $allowFilter);
        }
    });
}
```

---

## Summary

- **`AssociationsModel`** is an integrated behavior for managing product associations (alternatives, complements, newer versions).
- It provides multiple scopes ranging from simple (`hasAlternate()`) to advanced ones that allow passing additional conditions (`conditions`, `scopes`) and customizing each direction (`typePerDirection`, `conditionsPerDirection`).
- The core scope `whereAssociation` controls all aspects: relationship type, direction, presence/absence, conditions on the association table, applying scopes, and defining the relationship logic (`AND`/`OR`).
- **`AssociationsHelper`** simplifies integrating these filters into admin interfaces, saving development time and ensuring consistency of experience.

These tools make building product recommendation systems and managing associations in e-commerce stores a flexible and maintainable process.

