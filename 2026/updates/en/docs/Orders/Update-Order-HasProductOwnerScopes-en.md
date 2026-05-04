## 2026-05-02 - 2026-05-03

### Comprehensive Update to Order Filtering Scopes by Product Owner in the `Order` Model

**Plugin:** `Nano.Orders`  
**File:** `HasProductOwnerScopes` trait (`plugins/nano/orders/models/Order/HasProductOwnerScopes.php`)

---

### Development of Advanced Query Scopes for Filtering Orders Based on Product Ownership

The `HasProductOwnerScopes` trait was developed and added to the `Order` model, granting developers the ability to filter orders with high flexibility based on products owned by a specific user. Developers no longer need to write complex subqueries or manually handle nested relationships; instead, they can use ready-made scopes that support:

- Filtering by **product owner** (`user_type` and `user_id` in the `tss_inventory_products` table).
- Specifying the **number of owned products** within the order (using `count` and `countOperator`).
- **Negation** (retrieving orders that do not contain products owned by the owner).
- **Boolean combination** (`AND` / `OR`) to link conditions with other scopes.
- **Additional conditions** on the product itself (price, status...) or on the order item (quantity, actual price...).
- **Adding a computed column** counting owned products in the order, and **sorting** by it.
- **Quick check functions** on a single order object (e.g., `hasAnyProductByOwner`).

All scopes and helper functions were moved to a separate trait to facilitate maintenance and reuse, while adhering to using table name constants to ensure query stability.

---

### 1. Developed Components

| Behavior | Component | Description |
|----------|-----------|-------------|
| `Order` (Model) | `HasProductOwnerScopes` (trait) | Contains all scopes and helper functions for filtering orders by product owner. |

#### Main and Auxiliary Scopes

| Category | Scope | Description |
|----------|-------|-------------|
| **Advanced Scope** | `scopeWhereHasProductsByOwner` | The main scope with a comprehensive options array (`user`, `count`, `boolean`, `not`, `conditions`, `itemConditions`). |
| **Auxiliary Scopes** | `scopeWhereProductOwner` | Quick shortcut: orders containing at least one product owned by the owner. |
| | `scopeHasProductsByOwner` | Filter by a minimum number of owned products. |
| | `scopeDoesntHaveProductsByOwner` | Filter orders that contain no owned product. |
| | `scopeHasProductsByOwnerCount` | Exact (or compared) count of owned products. |
| | `scopeOrWhereHasProductsByOwner` | Link scope with `OR`. |
| **Computed Columns & Sorting** | `scopeWithProductsByOwnerCount` | Add `products_owner_count` column to `SELECT`. |
| | `scopeSortByProductsByOwnerCount` | Sort orders ascending/descending by number of owned products. |
| **Check Functions** | `hasAnyProductByOwner` | Check whether an order contains at least one product owned by the user. |

---

### 2. Code Update Details

#### 2.1 Main Scope `whereHasProductsByOwner`
This scope is designed to be comprehensive and accepts a single options array, unifying the usage interface and eliminating the need for dozens of separate scopes. Supported options:

| Option | Type | Description |
|--------|------|-------------|
| `user` | `Authenticatable\|Model` | The owner user object. |
| `userType` | `string` | User type (e.g., `RainLab\User\Models\User`). |
| `userId` | `int` | User ID. |
| `boolean` | `string` | `'and'` or `'or'` for linking with an outer query. |
| `not` | `bool` | `true` for negation (`doesntHave`). |
| `count` | `int` | Required number of owned products. |
| `countOperator` | `string` | Comparison operator (`>=`, `=`, `<`...). |
| `conditions` | `array\|callable` | Additional conditions on the products table. |
| `itemConditions` | `array\|callable` | Additional conditions on the order items table. |

If no owner data is passed, the scope automatically attempts to fetch the currently logged-in user via `AuthHelpers` or `BackendAuth` or `Auth`. If the owner cannot be determined, the query is returned without applying any filter, protecting against unintended empty results.

#### 2.2 Auxiliary Scopes
The auxiliary scopes (`HasProductsByOwner`, `DoesntHaveProductsByOwner`, `HasProductsByOwnerCount`, `OrWhereHasProductsByOwner`) are mere wrappers that call the main scope with the appropriate options. This ensures no code duplication and centralizes maintenance.

#### 2.3 Computed Columns and Sorting
- `withProductsByOwnerCount`: Adds a computed column counting owned products in the order using `selectSub`.
- `sortByProductsByOwnerCount`: Sorts results by that column using `orderByRaw` with safe binding passing to avoid errors from using `DB::raw` without bindings.

#### 2.4 Check Functions
- `hasAnyProductByOwner($user)`: Quick function on an `Order` object to check for at least one product owned by the user (or the current user automatically).

#### 2.5 Using Table Constants
To avoid errors when table names change:
- `\Nano\Shop\Models\Product::TABLE_NAME` is used to refer to the products table.
- `(new \Nano\Orders\Models\OrderItem)->getTable()` is used for the order items table.

Also, the table name is prefixed to each field in conditions to avoid ambiguity in queries joining more than one table.

---

### 3. Practical Examples

#### 3.1 Orders containing any product owned by the current user
```php
$orders = Order::hasProductsByOwner()->get();
```

#### 3.2 Orders not containing any product owned by a specific user
```php
$user = \BackendAuth::getUser();
$orders = Order::doesntHaveProductsByOwner($user)->get();
```

#### 3.3 Orders containing exactly 3 owned products with an additional condition on the product (price > 50)
```php
$orders = Order::whereHasProductsByOwner([
    'user'       => $user,
    'count'      => 3,
    'countOperator' => '=',
    'conditions' => function ($q) {
        $q->where('price', '>', 50);
    }
])->get();
```

#### 3.4 Orders containing an owned product with quantity greater than 2 in the order
```php
$orders = Order::whereHasProductsByOwner([
    'user'           => $user,
    'itemConditions' => function ($q) {
        $q->where('quantity', '>', 2);
    }
])->get();
```

#### 3.5 Combining with order status using OR
```php
$orders = Order::isNewOrders()
    ->orWhereHasProductsByOwner(['user' => $user])
    ->get();
```

#### 3.6 Adding the owned products count column and sorting descending
```php
$orders = Order::withProductsByOwnerCount()
    ->sortByProductsByOwnerCount('desc')
    ->get();

foreach ($orders as $order) {
    echo $order->products_owner_count;
}
```

#### 3.7 Checking a single order
```php
$order = Order::find(10);
if ($order->hasAnyProductByOwner($user)) {
    // Processing
}
```

---

### 4. Added Value

- **For developers**: A unified and flexible interface covering all order filtering scenarios based on product ownership, without needing complex subqueries. Advanced options like `count`, `not`, `conditions`, and `itemConditions` open broad possibilities for reports and dashboards.
- **For the application**: Improved performance using `whereHas` and `whereHas` with `count`, and secure subquery bindings. Reliance on table constants prevents errors when the structure changes.
- **For end users**: The ability to build smart lists such as "Orders containing my products" or "Orders from my customers (via my products)" enhances the user experience in multi-vendor marketplace systems.
- **Flexibility**: Boolean combination (`OR`) allows easily mixing conditions with other scopes (order status, type, date...).
- **Extensibility**: New options can be easily added to the main scope, or additional auxiliary scopes can be derived in the same pattern.

---

### 5. Conclusion

This update represents a qualitative leap in managing order queries related to product ownership within the `Nano.Orders` plugin. Through an independent trait and advanced scopes, developers can now build complex filtering systems easily and with high performance, while maintaining full compatibility with the existing structure. The provided documentation ensures quick comprehension and application.

---

**Note**: For deeper technical details, refer to the `HasProductOwnerScopes.php` file and the comments attached to the scopes.

**Reference Documentation**:
- [Trait HasProductOwnerScopes Documentation](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-en.md)

---
