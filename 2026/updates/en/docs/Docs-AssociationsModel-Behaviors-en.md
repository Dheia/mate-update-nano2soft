## Documentation for `AssociationsModel` Behavior and `AssociationsHelper` Class

**Namespace:** `Nano\Shop\Behaviors`  
**Helper Namespace:** `Nano\Shop\Classes`

---

### Introduction

`AssociationsModel` is a **Behavior** designed for NanoSoft application models, providing **product association management** features such as:
- **Alternate Products** (Alternate)
- **Cross‑Sell Products** (Cross‑Sell)
- **Up‑Sell Products** (Up‑Sell)

In addition, the behavior offers a wide range of scopes for filtering and searching products based on these associations. The helper class `AssociationsHelper` provides ready-to-use tools for injecting these association filters into administration interfaces (such as product lists) with ease and flexibility.

The main objective is to empower developers to:
- Create relationships between products (parent → target) of any type.
- Flexibly query products that have (or do not have) specific associations.
- Control the direction of the relationship (as parent, as target, or both).
- Combine association conditions with other conditions (`and` / `or`).
- Add ready-made filters to `ListController` to filter products by association status.

---

### Setup and Installation

The behavior does not require additional installation; it is part of the `Nano.Shop` package. To use it, you need to add it as a Behavior to the desired model (e.g., `Product` model).

#### 1. Adding the Behavior Directly to the Model (in the model file)

```php
class Product extends Model
{
    public $implement = [
        'Nano.Shop.Behaviors.AssociationsModel'
    ];

    // ... rest of the model definitions
}
```

#### 2. Injecting the Behavior into a Model from Another Plugin (Dynamically)

You can add the behavior to any model without modifying its original file using the extension methods available in the NanoSoft framework:

```php
// In your plugin's boot method
public function boot()
{
    \Nano\Shop\Models\Product::extend(function($model) {
        if (!$model->isClassExtendedWith('Nano.Shop.Behaviors.AssociationsModel')) {
            $model->implementClassWith('Nano.Shop.Behaviors.AssociationsModel');
        }
    });
}
```

> **Note:** The behavior relies on predefined relationships (`hasMany` named `associations` and `inverseAssociations`). These relationships are automatically defined inside the behavior's constructor (`__construct`), so you do not need to add them manually.

---

### Table Structure and Relationships

The behavior uses a pivot table named `nano_shop_product_associations` containing:
- `id`
- `product_parent_id` (parent product)
- `product_target_id` (target product)
- `type` (association type: `cross_sell`, `up_sell`, `alternate`)
- `is_active`, `is_default`, `sort_order` (optional)
- `created_at`, `updated_at`, `deleted_at`

Relationships automatically added to the model:
- `associations` : a `hasMany` relationship to retrieve associations where the model is the **parent**.
- `inverseAssociations` : a `hasMany` relationship to retrieve associations where the model is the **target**.

Additionally, helper `belongsToMany` relationships are defined for direct access to associated products (if needed).

---

### Scopes Available in the Behavior

The behavior provides a comprehensive set of scopes for querying products based on their associations.

#### 1. Basic Scopes for Association Types (Without Specified Direction)

| Scope | Description |
|-------|-------------|
| `scopeHasAlternate($query)` | Products that have alternate associations (in any direction) |
| `scopeHasCrossSell($query)` | Products that have cross‑sell associations |
| `scopeHasUpSell($query)` | Products that have up‑sell associations |
| `scopeHasNotAlternate($query)` | Products that do not have alternate associations |
| `scopeHasNotCrossSell($query)` | Products that do not have cross‑sell associations |
| `scopeHasNotUpSell($query)` | Products that do not have up‑sell associations |

#### 2. Direction‑Specific Scopes

| Scope | Description |
|-------|-------------|
| `scopeHasAlternateAsParent($query)` | Products that have alternates as parent |
| `scopeHasAlternateAsTarget($query)` | Products that are alternates for other products (as target) |
| `scopeHasCrossSellAsParent($query)` | Products that have cross‑sells as parent |
| `scopeHasCrossSellAsTarget($query)` | Products that are cross‑sells for other products |
| `scopeHasUpSellAsParent($query)` | Products that have up‑sells as parent |
| `scopeHasUpSellAsTarget($query)` | Products that are up‑sells for other products |

#### 3. Scopes for Any Association (Existence or Non‑existence)

| Scope | Description |
|-------|-------------|
| `scopeHasAnyAssociationBoth($query)` | Products that have any association (in any direction) |
| `scopeHasNotAnyAssociationBoth($query)` | Products that have no associations |

#### 4. Advanced Scope Using `scopeWhereAssociation`

This is the most powerful scope, allowing an array of detailed options:

```php
scopeWhereAssociation($query, $options = [], $boolean = 'and', $not = false)
```

**Options (array):**
- `type` : `string|array|null` – association type (alternate, cross_sell, up_sell) or an array of types.
- `direction` : `string|array|null` – direction: `'parent'`, `'target'`, `'both'` (default both) or an array of directions.
- `callback` : `callable|null` – an additional callback to customize the relation query.
- `has` : `bool` – `true` for existence (whereHas), `false` for negation (whereDoesntHave); usually inferred from `$not`.

**Additional Parameters:**
- `$boolean` : `'and'` or `'or'` to combine this condition with previous ones.
- `$not` : `true` to negate the condition (whereDoesntHave).

#### 5. Helper Scopes for OR and NOT

| Scope | Description |
|-------|-------------|
| `scopeOrWhereAssociation($query, $options = [], $not = false)` | `orWhere` condition using the same options |
| `scopeWhereNotAssociation($query, $options = [], $boolean = 'and')` | Negation condition (doesntHave) |
| `scopeOrWhereNotAssociation($query, $options = [])` | `orWhere` condition with negation |

#### 6. General Convenience Scopes

| Scope | Description |
|-------|-------------|
| `scopeHasAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | Has associations of a specific type |
| `scopeHasNotAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | Does not have associations of a specific type |
| `scopeHasAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | Has any association (any type) |
| `scopeHasNotAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | Does not have any association |

#### 7. Toggle Scopes (Specifically for Interface Filters)

| Scope | Description |
|-------|-------------|
| `scopeIsToggelAnyAssociation($query, $value)` | Toggle: if `1` returns products with any association, if `0` returns products without any association |
| `scopeIsToggelAnyAlternate($query, $value)` | Toggle for alternates |
| `scopeIsToggelAnyCrossSell($query, $value)` | Toggle for cross‑sells |
| `scopeIsToggelAnyUpSell($query, $value)` | Toggle for up‑sells |

---

### Practical Examples of Using Scopes

#### 1. Get All Products That Have Alternates

```php
$products = Product::hasAlternate()->get();
```

#### 2. Get Products That Are Alternates (as Target)

```php
$products = Product::hasAlternateAsTarget()->get();
```

#### 3. Products That Have Cross‑Sell Associations **as Parent** Only

```php
$products = Product::hasCrossSellAsParent()->get();
```

#### 4. Products That Have No Associations at All

```php
$products = Product::hasNotAnyAssociationBoth()->get();
```

#### 5. Using `whereAssociation` to Find Products That Have Alternates or Cross‑Sells

```php
$products = Product::whereAssociation([
    'type' => ['alternate', 'cross_sell'],
    'direction' => 'both'
])->get();
```

#### 6. Combining Multiple Conditions with `orWhereAssociation`

```php
$products = Product::where('is_active', true)
    ->orWhereAssociation(['type' => 'up_sell', 'direction' => 'parent'])
    ->get();
```

#### 7. Using a Callback to Customize Relation Conditions (e.g., Only Active Associations)

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'direction' => 'both',
    'callback' => function ($relationQuery) {
        $relationQuery->where('is_active', true);
    }
])->get();
```

#### 8. Finding Products That Do Not Have Alternates (Using `hasNotAssociation`)

```php
$products = Product::hasNotAssociation('alternate')->get();
```

#### 9. Using `whereNotAssociation` for Negation with a Specific Direction

```php
$products = Product::whereNotAssociation([
    'type' => 'cross_sell',
    'direction' => 'parent'
])->get();
```

#### 10. Combining Multiple Conditions with Negation and OR

```php
$products = Product::where('price', '>', 100)
    ->orWhereNotAssociation(['type' => 'up_sell', 'direction' => 'target'])
    ->get();
```

#### 11. Using Toggle Scopes in Programmatic Queries

```php
// If you want products that have any association
$products = Product::isToggelAnyAssociation(1)->get();

// If you want products that have no association
$products = Product::isToggelAnyAssociation(0)->get();
```

#### 12. Getting Products That Have Alternates and Ordering by Price

```php
$products = Product::hasAlternate()->orderBy('price', 'desc')->get();
```

#### 13. Using `scopeHasAssociation` with a Callback

```php
$products = Product::hasAssociation('cross_sell', 'both', function($q) {
    $q->where('created_at', '>', now()->subDays(30));
})->get();
```

---

### Helper Class `AssociationsHelper`

The `AssociationsHelper` class is located at `Nano\Shop\Classes\AssociationsHelper` and provides static methods for managing association filters in administration interfaces (such as `ListController`). Its purpose is to simplify the process of adding or removing product filters based on association types.

#### 1. Methods Returning Filter Arrays

| Method | Description |
|--------|-------------|
| `getAlternateFilterScopes($defaultValue = 0, $is_force = false)` | Array of alternate filters (checkboxes + toggle) |
| `getCrossSellFilterScopes($defaultValue = 0, $is_force = false)` | Cross‑sell filters |
| `getUpSellFilterScopes($defaultValue = 0, $is_force = false)` | Up‑sell filters |
| `getAnyAssociationFilterScopes($defaultValue = 0, $is_force = false)` | "Any association" filters (existence / non‑existence) |
| `getAllAssociationFilterScopes($defaultValue = 0, $is_force = false)` | All of the above filters combined |

#### 2. Methods for Injecting Filters into the Filter Widget

```php
injectAssociationScopes($filter, $types = null, $defaultValue = 0, $is_force = false)
```

- `$filter` : The filter object (usually from `extendListFilterScopes`).
- `$types` : Specifies which filter types to add. Can be:
  - `null` or `false` : adds nothing.
  - `true` : adds all filters (`any`, `alternate`, `cross_sell`, `up_sell`).
  - A string like `'any,alternate'` : adds the mentioned types.
  - An array like `['alternate', 'up_sell']`.
- `$defaultValue` : Default value for filters (0 means not active).
- `$is_force` : For future use.

#### 3. Methods for Removing Filters

```php
removeAssociationScopes($filter, $types = null)
```

- Removes the specified filters (`$types` follows the same logic as in `injectAssociationScopes`).

#### 4. Method to Check if a Type is Enabled

```php
isTypeEnabled($type)
```

- Relies on configuration settings (`nano.shop::products.associations.allow_filter`) to determine if a specific type is enabled.

---

### Using `AssociationsHelper` in Plugins

```php
use Nano\Shop\Classes\AssociationsHelper;

// In the boot method of your plugin
public function boot()
{
    // You can inject filters into the product controller
    \ShopProductsController::extendListFilterScopes(function($filter) {
        $allowFilter = \Config::get('nano.shop::products.associations.allow_filter', false);
        if ($allowFilter && class_exists(AssociationsHelper::class)) {
            AssociationsHelper::injectAssociationScopes($filter, $allowFilter);
        }

        // Example: Remove specific filters based on other settings
        if (!\Config::get('nano.shop::allow_alternate_filter', true)) {
            AssociationsHelper::removeAssociationScopes($filter, 'alternate');
        }
    });
}
```

### Example Configuration in config.php

```php
// config.php
'products' => [
    'associations' => [
        // Can be true (all filters), false (no filters), or a string like 'any,alternate'
        'allow_filter' => env('NANO_SHOP_PRODUCTS_ASSOCIATIONS_ALLOW_FILTER', false),
    ],
],
```

---

### Putting It All Together: A Complete Example

Suppose you have a `Products` controller and you want to display products that have alternates, with the ability to toggle based on any association.

**In the Controller:**
```php
$query = Product::query();

// Apply a user filter
if ($hasAlternate = input('has_alternate')) {
    $query->hasAlternate();
}

if ($toggleAny = input('toggle_any')) {
    $query->isToggelAnyAssociation($toggleAny);
}

$products = $query->paginate(20);
```

**In the filter configuration file (e.g., `config_filter.yaml`):**
```yaml
scopes:
    has_alternate:
        label: 'Has Alternates'
        type: checkbox
        scope: hasAlternate
        default: 0
    is_toggel_any_association:
        label: 'Association Status'
        type: switch
        scope: isToggelAnyAssociation
        default: 0
```

---

### Summary

- **`AssociationsModel`** is a powerful and flexible behavior for managing and querying product associations.
- It provides multiple scopes suitable for most use cases, from simple to complex.
- The `whereAssociation` scope offers full control through an options array.
- **`AssociationsHelper`** simplifies the integration of these association filters into administration interfaces.
- The design is easily extensible (adding new association types).

Using these tools, developers can build comprehensive e‑commerce systems with intelligent product association management, enhancing user experience through recommendations (cross‑sell, up‑sell) and providing suitable alternatives.
