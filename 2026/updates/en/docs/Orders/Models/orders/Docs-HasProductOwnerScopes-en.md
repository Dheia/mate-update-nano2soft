# Documentation for `HasProductOwnerScopes` Scopes  
**Model** `Nano\Orders\Models\Order`  
**Goal**: Filter orders based on the ownership of products linked to order items.

---

## 📋 Overview
This series of scopes provides flexible and advanced queries on orders based on the **product owner** (`user_type` and `user_id` fields in the `tss_inventory_products` products table). You can specify the target user (or rely on the current user), control the number of owned products, negation, add extra conditions on the product or order item, and even combine filters using boolean `OR`.

---

## 🧩 Main Components

| Scope / Function | Description |
|------------------|-------------|
| `scopeWhereHasProductsByOwner` | **Main scope** – accepts a comprehensive options array. |
| `scopeWhereProductOwner` | Simple shortcut for quick use (at least one product). |
| `scopeHasProductsByOwner` | Filter orders containing a certain number of owned products. |
| `scopeDoesntHaveProductsByOwner` | Filter orders containing no owned product. |
| `scopeHasProductsByOwnerCount` | Exact (or compared) count of owned products within the order. |
| `scopeOrWhereHasProductsByOwner` | Link the filter with `OR` to previous conditions. |
| `scopeWithProductsByOwnerCount` | Add a virtual column with the count of owned products. |
| `scopeSortByProductsByOwnerCount` | Sort orders by the number of owned products ascending/descending. |
| `hasAnyProductByOwner` | Check function on a single `Order` object (without a new query). |
| `applyOwnerToProductQuery` | Helper to apply owner condition on a subquery. |

---

## 📘 Main Scope: `scopeWhereHasProductsByOwner`

### Signature
```php
Order::whereHasProductsByOwner(array $options)
```

### Options Table

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `user` | `Authenticatable\|Model\|null` | `null` | The owner user object. |
| `userType` | `string\|null` | `null` | User type (e.g., `RainLab\User\Models\User`). |
| `userId` | `int\|null` | `null` | User ID. |
| `boolean` | `string` | `'and'` | Link method: `'and'` or `'or'`. |
| `not` | `bool` | `false` | `true` to retrieve orders that **do not** contain the owned product. |
| `count` | `int\|null` | `null` | Required number of owned products (uses `has` instead of `exists`). |
| `countOperator` | `string` | `'>=`' | Comparison operator when using `count` (e.g., `=`, `<=`, `>`). |
| `conditions` | `array\|callable` | `[]` | Additional conditions on the **product table** (`tss_inventory_products`). |
| `itemConditions` | `array\|callable` | `[]` | Additional conditions on the **order items table** (`nano_orders_items`). |

> **Note**: If neither `user` nor `userType`/`userId` is provided, the scope automatically attempts to fetch the currently logged-in user.

---

## 🧪 Illustrative Examples

### Example 1: Orders containing any product owned by the current user
```php
$orders = Order::whereHasProductsByOwner([])->get();
// Or shortened
$orders = Order::hasProductsByOwner()->get();
```

### Example 2: Orders containing a product owned by a specific user
```php
$user = \BackendAuth::getUser();
$orders = Order::whereHasProductsByOwner(['user' => $user])->get();
```

### Example 3: Orders that do not contain any owned product by the user
```php
$orders = Order::doesntHaveProductsByOwner($user)->get();
// Or using explicit negation
$orders = Order::whereHasProductsByOwner([
    'user' => $user,
    'not'  => true,
])->get();
```

### Example 4: Orders containing exactly 3 products owned by a specific user
```php
$orders = Order::hasProductsByOwnerCount($user, 3, '=')->get();
// Or using the main scope
$orders = Order::whereHasProductsByOwner([
    'user'  => $user,
    'count' => 3,
    'countOperator' => '='
])->get();
```

### Example 5: Additional conditions on the product itself
```php
// Orders containing an owned product with price > 100 and status "active"
$orders = Order::whereHasProductsByOwner([
    'user'       => $user,
    'conditions' => [
        'price'  => 100,
        'status' => 'active'
    ]
])->get();
```
Or using a Closure:
```php
$orders = Order::whereHasProductsByOwner([
    'user'       => $user,
    'conditions' => function ($q) {
        $q->where('price', '>', 100)->where('status', 'active');
    }
])->get();
```

### Example 6: Conditions on the order item together
```php
// Quantity > 2 of the owned product
$orders = Order::whereHasProductsByOwner([
    'user'           => $user,
    'itemConditions' => ['quantity' => 2]  // quantity > 2? you'll need a closure
])->get();

// Using closure for itemConditions
$orders = Order::whereHasProductsByOwner([
    'user'           => $user,
    'itemConditions' => function ($q) {
        $q->where('quantity', '>', 2);
    }
])->get();
```

### Example 7: Combining filters with OR
```php
// Orders containing products owned by the user **or** new orders only
$orders = Order::isNewOrders()
    ->orWhereHasProductsByOwner(['user' => $user])
    ->get();
```

### Example 8: Adding a column with owned product count and sorting by it
```php
$orders = Order::withProductsByOwnerCount()
    ->sortByProductsByOwnerCount('desc')
    ->get();

foreach ($orders as $order) {
    echo $order->products_owner_count; // added column
}
```

### Example 9: Checking a specific order object
```php
$order = Order::find(5);
if ($order->hasAnyProductByOwner($user)) {
    // The order contains at least one product owned by the user
}
```

---

## ⚠️ Technical Notes
- **Table constants**: The scopes use `Product::TABLE_NAME` and `(new OrderItem)->getTable()` to ensure no name conflicts.
- **Subqueries**: When using `withProductsByOwnerCount` or `sortByProductsByOwnerCount`, bindings are safely merged via `orderByRaw` with passed bindings.
- **Query performance**: The scopes rely on `whereHas`/`whereDoesntHave` generating `EXISTS` or `COUNT` queries, which are optimized for indexing, but with many conditions, performance monitoring may be needed.
- **No user passed**: If no user identifier is passed (user, userType, userId), the scope attempts to retrieve the currently logged-in user via `AuthHelpers`, `BackendAuth`, or `Auth`. If no user is found, **no filter is applied** (the query is returned as is) to avoid unexpected results.

---

## 🧰 Advanced Scenarios

### Filter orders containing owned products + customer's own orders
```php
$user = \BackendAuth::getUser();
$orders = Order::where('user_id', $user->id)
    ->orWhereHasProductsByOwner(['user' => $user])
    ->get();
```

### Reports: Number of orders containing at least 2 owned products during the current week
```php
$start = Carbon::now()->startOfWeek();
$orders = Order::where('created_at', '>=', $start)
    ->whereHasProductsByOwner([
        'user'  => $user,
        'count' => 2,
        'countOperator' => '>='
    ])
    ->count();
```

---

## 📦 Summary
Using these scopes, filtering orders based on product ownership has become extremely flexible, with full support for negation, counting, sorting, and boolean combination, meeting the needs of advanced reports and dashboards within the NanoSoft system.
