# Advanced Practical Examples for `AssociationsModel` Behavior and `AssociationsHelper`

**Note:** Before starting, ensure that the behavior is added to the `Product` model and that the association tables exist as explained in the basic documentation.

---

## Contents

1. [Setting Up Association Data](#setting-up-association-data)
2. [Basic Queries (Existence/Non-Existence)](#basic-queries-existencenon-existence)
3. [Advanced Queries Using `whereAssociation`](#advanced-queries-using-whereassociation)
   - [Additional Conditions](#additional-conditions)
   - [Applying Scopes to the Association Table](#applying-scopes-to-the-association-table)
   - [Customizing Conditions Per Direction (conditionsPerDirection)](#customizing-conditions-per-direction-conditionsperdirection)
   - [Different Type Per Direction (typePerDirection)](#different-type-per-direction-typeperdirection)
   - [Controlling the Relationship Logic (relationOperator)](#controlling-the-relationship-logic-relationoperator)
   - [Using a Custom Callback](#using-a-custom-callback)
   - [Combining with Other Conditions (AND/OR)](#combining-with-other-conditions-andor)
4. [Negation and Negative Queries](#negation-and-negative-queries)
5. [Examples of Using `AssociationsHelper` in Admin Interfaces](#examples-of-using-associationshelper-in-admin-interfaces)
   - [Injecting All Association Filters](#injecting-all-association-filters)
   - [Injecting Only Specific Filters](#injecting-only-specific-filters)
   - [Removing Specific Filters](#removing-specific-filters)
   - [Using Default Values and Toggle](#using-default-values-and-toggle)
6. [Integrated (Real-World) Scenarios](#integrated-real-world-scenarios)
   - [Displaying Recommendations for a Specific Product (Cross‑Sell)](#displaying-recommendations-for-a-specific-product-cross‑sell)
   - [Displaying Product Alternatives (Alternate)](#displaying-product-alternatives-alternate)
   - [Managing Relationships in the Control Panel (Backend)](#managing-relationships-in-the-control-panel-backend)
   - [Updating or Deleting Associations with Jobs](#updating-or-deleting-associations-with-jobs)

---

## Setting Up Association Data

Let's assume we have the following products:

- **P1**: Laptop (id=1)
- **P2**: Laptop Bag (id=2)
- **P3**: Mouse (id=3)
- **P4**: Alternative Laptop (same specifications) (id=4)
- **P5**: Newer Model Laptop (id=5)

We will create the associations:

```php
$product1 = Product::find(1);
$product2 = Product::find(2);
$product3 = Product::find(3);
$product4 = Product::find(4);
$product5 = Product::find(5);

// P1 has complements: bag (cross_sell)
$product1->associate($product2, 'cross_sell');

// P1 has an alternative: P4 (alternate)
$product1->associate($product4, 'alternate');

// P1 has a newer version: P5 (up_sell)
$product1->associate($product5, 'up_sell');

// P3 (mouse) is a complement to P1 (but we might not want to add it in this example)
// We can create a reverse association: P1 is a target for P2 (if we want)
$product2->associate($product1, 'cross_sell'); // P2 as parent and P1 as target
```

**Note:** Bidirectional associations are not necessary, but they are possible.

---

## Basic Queries (Existence/Non-Existence)

### 1. Products that have alternatives (in any direction)

```php
$products = Product::hasAlternate()->get();
// Result: P1 (has an alternative), and any other product that is an alternative for another (e.g., P4 if it's a target for P1)
```

### 2. Products that are alternatives (as target only)

```php
$alternatives = Product::hasAlternateAsTarget()->get();
// Result: P4 (because it is a target in an alternative relationship)
```

### 3. Products that have no associations at all

```php
$isolated = Product::hasNotAnyAssociationBoth()->get();
// Result: P3 (if it has no associations)
```

### 4. Products that have complements as parent (sold together)

```php
$crossSellParents = Product::hasCrossSellAsParent()->get();
// Result: P1 (has P2 as complement), P2 (has P1 as complement) if we added the reverse relationship
```

---

## Advanced Queries Using `whereAssociation`

### Additional Conditions

We want products that have **active** (`is_active = 1`) associations of type `alternate`.

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => ['is_active' => 1]
])->get();
```

If we also want `is_public` = 1:

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => ['is_active' => 1, 'is_public' => 1]
])->get();
```

**Conditions using `callable`:**

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => function($q) {
        $q->where('is_active', 1)
          ->where('created_at', '>', now()->subMonths(3));
    }
])->get();
```

### Applying Scopes to the Association Table

Scopes like `isCompany` and `isDepartment` are defined in the `ProductsAssociation` model; we can apply them:

```php
// Products that have alternative associations and belong to the current company
$products = Product::whereAssociation([
    'type' => 'alternate',
    'scopes' => ['isCompany']
])->get();

// Passing parameters to the scope
$products = Product::whereAssociation([
    'type' => 'alternate',
    'scopes' => ['isDepartment' => 5]  // departments_id = 5
])->get();
```

### Customizing Conditions Per Direction (conditionsPerDirection)

We want products that have `cross_sell` associations as parent that are **active**, or `cross_sell` associations as target that are **public** (`is_public = 1`).

```php
$products = Product::whereAssociation([
    'direction' => 'both',
    'type' => 'cross_sell',
    'conditionsPerDirection' => [
        'parent' => ['is_active' => 1],
        'target' => ['is_public' => 1]
    ]
])->get();
```

### Different Type Per Direction (typePerDirection)

We want products that have:
- Alternatives (`alternate`) as parent **or**
- Complements (`cross_sell`) as target.

```php
$products = Product::whereAssociation([
    'direction' => ['parent', 'target'],
    'typePerDirection' => [
        'parent' => 'alternate',
        'target' => 'cross_sell'
    ],
    'relationOperator' => 'or' // default, can be omitted
])->get();
```

### Controlling the Relationship Logic (relationOperator)

We want products that have associations **in both directions** at the same time (i.e., the product is a parent for some association **and** a target for another association). This can be useful for finding products that are intermediaries in a recommendation chain.

```php
$products = Product::whereAssociation([
    'direction' => 'both',
    'relationOperator' => 'and',   // AND between directions
    'conditions' => ['is_active' => 1]
])->get();
```

### Using a Custom Callback

We want to customize the relationship query based on direction:

```php
$products = Product::whereAssociation([
    'type' => 'cross_sell',
    'callback' => function($query, $direction) {
        if ($direction === 'parent') {
            $query->where('sort_order', '>', 0);
        } else {
            $query->where('is_public', 1);
        }
    }
])->get();
```

### Combining with Other Conditions (AND/OR)

We want products with price greater than 100 **and** have alternatives.

```php
$products = Product::where('price', '>', 100)
    ->whereAssociation(['type' => 'alternate'])
    ->get();
```

We want products with price less than 50 **or** have no associations.

```php
$products = Product::where('price', '<', 50)
    ->orWhereNotAssociation(['type' => 'alternate', 'direction' => 'both'])
    ->get();
```

---

## Negation and Negative Queries

### Products that do not have `up_sell` associations in any direction

```php
$products = Product::whereNotAssociation(['type' => 'up_sell'])->get();
```

### Products that do not have `alternate` associations as parent

```php
$products = Product::whereNotAssociation([
    'type' => 'alternate',
    'direction' => 'parent'
])->get();
```

### Using `hasNotAssociation`

```php
$products = Product::hasNotAssociation('alternate', 'both');
```

### Products that do not have **active** `cross_sell` associations (negative condition)

```php
$products = Product::whereNotAssociation([
    'type' => 'cross_sell',
    'conditions' => ['is_active' => 1]
])->get();
```

---

## Examples of Using `AssociationsHelper` in Admin Interfaces

### Injecting All Association Filters

In the product controller (e.g., `ShopProductsController`), we add the filters when initializing the list:

```php
use Nano\Shop\Classes\AssociationsHelper;

public function listFilterScopes()
{
    $scopes = parent::listFilterScopes();

    // Add association filters
    if (class_exists(AssociationsHelper::class)) {
        $associationScopes = AssociationsHelper::getAllAssociationFilterScopes();
        $scopes = array_merge($scopes, $associationScopes);
    }

    return $scopes;
}
```

Or they can be injected via the `backend.filter.extendScopes` event:

```php
\Event::listen('backend.filter.extendScopes', function ($filterWidget, $model, $scopes) {
    if ($model instanceof \Nano\Shop\Models\Product) {
        AssociationsHelper::injectAssociationScopes($filterWidget, true);
    }
});
```

### Injecting Only Specific Filters

```php
AssociationsHelper::injectAssociationScopes($filter, 'alternate,cross_sell');
```

Or:

```php
AssociationsHelper::injectAssociationScopes($filter, ['alternate', 'up_sell']);
```

### Removing Specific Filters

```php
AssociationsHelper::removeAssociationScopes($filter, 'any');
```

### Using Default Values and Toggle

When injecting filters, a default value can be specified (e.g., `1` to enable the "any association" filter):

```php
AssociationsHelper::injectAssociationScopes($filter, 'any', 1);
```

**Toggle via User Interface:**  
The `is_toggel_any_association` filter is of type `switch`; its value toggles (`1` → products with associations, `0` → products without associations). This works directly with the built-in scope `isToggelAnyAssociation`.

---

## Integrated (Real-World) Scenarios

### Displaying Recommendations for a Specific Product (Cross‑Sell)

On the product page, we need to display complementary products (linked as parent or target) ordered by `sort_order`.

```php
class Product extends Model
{
    // ... scopes

    public function getCrossSellProducts()
    {
        // Products that are complements to this product (as parent)
        $asParent = $this->associations_products()
            ->where('type', ProductsAssociation::CROSS_SELL)
            ->orderBy('sort_order')
            ->get();

        // Products for which this product is a complement (as target)
        $asTarget = $this->inverseAssociations_products()
            ->where('type', ProductsAssociation::CROSS_SELL)
            ->orderBy('sort_order')
            ->get();

        return $asParent->merge($asTarget)->unique('id');
    }
}
```

### Displaying Product Alternatives (Alternate)

```php
public function getAlternates()
{
    return $this->associations_products()
        ->where('type', ProductsAssociation::ALTERNATE)
        ->get()
        ->merge(
            $this->inverseAssociations_products()
                ->where('type', ProductsAssociation::ALTERNATE)
                ->get()
        )->unique('id');
}
```

### Managing Relationships in the Control Panel (Backend)

Using `AssociationsHelper` with `Filter` to provide quick filters:

```yaml
# config_filter.yaml
scopes:
    # ... other scopes
    has_alternate:
        label: 'Has Alternatives'
        type: checkbox
        scope: hasAlternate
        default: 0
    is_toggel_any_association:
        label: 'Association Status'
        type: switch
        scope: isToggelAnyAssociation
        default: 0
```

In the controller, you can create `listFilterScopes` and merge what `AssociationsHelper` provides with manual definitions.

### Updating or Deleting Associations with Jobs

The behavior provides `associate()` and `dissociate()` functions that use **Jobs** to delay processing. This is useful to avoid long operations in the response.

```php
$product = Product::find(1);
$target = Product::find(2);
$product->associate($target, 'cross_sell'); // sends job
$product->dissociate($target, 'cross_sell'); // sends job
```

If you want immediate execution (without a job), you can use the relationships directly:

```php
$association = new ProductsAssociation([
    'type' => 'cross_sell',
    'product_parent_id' => $product->id,
    'product_target_id' => $target->id,
]);
$association->save();
```

However, using jobs is better to avoid performance issues when creating a large number of associations.

---

## Conclusion

The examples above cover most real-world use cases for the `AssociationsModel` behavior. By combining advanced scopes, additional conditions, and the `AssociationsHelper`, you can build a powerful and flexible product recommendation system with easy-to-use admin interfaces.

**Reminder:**
- All scopes work with `Product` or any model to which the behavior is added.
- Use `scopeWhereAssociation` when you need fine-grained control, and the helper scopes (`hasAlternate`, `hasCrossSell`, ...) for simple usage.
- `AssociationsHelper` simplifies integrating filters into the backend, saving development time.

Feel free to customize the scopes and add new association types according to your project's needs.
