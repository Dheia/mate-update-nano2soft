# ProductLimitValidator Class Documentation

## 1. Introduction

`ProductLimitValidator` is a central and specialized class within the `Nano.Coupons` plugin (developed by NanoSoft). It acts as an intelligent checking bridge between the product limit settings (`product_limit`) stored in coupons and actual user behavior (cart, previous orders). The class loads active constraints from the database and checks whether the user has exceeded the allowed limits (quantity, number of orders), taking into account:
- A specific product (or its absence for a general check).
- An optional product unit.
- A specified time period (number of days).
- Specific order types (`order_restriction`).
- Specific stores (`shop_restriction`).

The class is designed to be reusable in various contexts (within a `CartCondition`, direct API calls, or background operations), with a strong focus on performance through caching (`Cache`) and optimized database queries.

## 2. Purpose and Benefits

- **Centralized Product Limit Check:** Instead of duplicating checking code in multiple places, the class provides a unified interface.
- **Separation of Concerns:** The verification logic is separated from the `ProductLimit` cart condition, allowing the class to be used independently.
- **High Performance:** Supports multiple fetching strategies (`cache`, `fresh_db`, `direct_query`) with control over the caching duration.
- **Precise Multilingual Error Messages:** When limits are exceeded, it throws an exception with a translatable message including limits, current values, and the date when the next order is possible.
- **Configuration Flexibility:** The class can be configured with options such as `use_cache` and `cache_ttl` to suit different environments.

## 3. Public Functions (Public API)

### 3.1 Constructor `__construct`

```php
public function __construct(User $user = null, array $options = [])
```

**Parameters:**
- `$user` (optional): The user object (`RainLab\User\Models\User`) whose limits are to be checked. If `null`, all checks will fail (return `true`) because constraints cannot be tied to an unknown user.
- `$options` (optional array):
  - `use_cache` (bool): Enable/disable caching for constraints (default `true`).
  - `cache_ttl` (int): Lifetime of constraints in cache in seconds (default 300).

**Example:**

```php
use Nano\Coupons\Classes\ProductLimitValidator;
use RainLab\User\Models\User;

$user = User::find(54);
$validator = new ProductLimitValidator($user);
// With custom options
$validator2 = new ProductLimitValidator($user, ['use_cache' => false]);
```

### 3.2 `checkOrFail`

```php
public function checkOrFail(
    ?int $productId = null,
    ?int $unitId = null,
    int $quantity = 0,
    $defaultOrderTypes = null,
    ?int $defaultShopId = null
): bool
```

Purpose: Comprehensive check for product/unit/quantity and number of orders. If `productId` is `null`, only the number of orders is checked based on the passed order type and store; otherwise, the product is checked in both cases.

**Parameters:**
- `$productId` (`int|null`): Product identifier. If `null`, product and quantity checks are ignored and focus shifts to the number of orders.
- `$unitId` (`int|null`): Unit identifier (optional). Used only with `$productId`.
- `$quantity` (`int`): Quantity to be added to the cart.
- `$defaultOrderTypes` (`string|array|null`): Current order type(s) (used if constraints don't specify `order_restriction`).
- `$defaultShopId` (`int|null`): Current store (used if constraints don't specify `shop_ids`).

**Returns:** `true` if no limit is exceeded.

**Exceptions:** Throws `ApplicationException` when quantity or order count is exceeded.

**Example:**

```php
// Check adding 3 units of product 3918 to store 10
$validator->checkOrFail(3918, null, 3, 'delivery', 10);
```

### 3.3 `check`

```php
public function check(
    ?int $productId = null,
    ?int $unitId = null,
    int $quantity = 0,
    $orderTypes = null,
    ?int $shopId = null
): bool
```

Purpose: Like `checkOrFail` but silently (does not throw exceptions), returns `false` when exceeded.

**Parameters:** Same as `checkOrFail` (without the ability to override values).

**Example:**

```php
if(!$validator->check(3918, 5, 1, 'delivery', 10)){
    echo 'Cannot add this product';
}
```

### 3.4 `getLimits`

```php
public function getLimits(
    ?int $productId = null,
    ?int $unitId = null,
    $orderTypes = null,
    ?int $shopId = null,
    string $strategy = 'auto'
): array
```

Purpose: Fetches the array of applicable constraints based on criteria, without performing a quantity check.

**Parameters:**
- `$productId`, `$unitId`, `$orderTypes`, `$shopId` – for filtering.
- `$strategy` (`string`): Fetch strategy.
  - `auto`: Use cache if enabled, otherwise direct query.
  - `cache`: Force loading from cache.
  - `fresh_db`: Ignore cache and fetch directly from database.
  - `direct_query`: Direct query with the specified conditions only (does not load all constraints).

**Example:**

```php
// Fetch all constraints for order type delivery
$limits = $validator->getLimits(null, null, 'delivery', null, 'fresh_db');
```

### 3.5 `getLimitsForProduct`

```php
public function getLimitsForProduct(
    int $productId,
    ?int $unitId = null,
    ?string $orderType = null,
    ?int $shopId = null,
    string $strategy = 'auto'
): array
```

A useful shortcut to `getLimits` when you have a specific product.

### 3.6 `checkOrderTypeLimitOrFail`

```php
public function checkOrderTypeLimitOrFail(
    $orderTypes,
    int $maxOrders,
    int $periodDays,
    ?int $shopId = null
): bool
```

Purpose: Check the number of orders for a specific order type regardless of product. Internally uses `checkOrFail` by passing `productId=null`.

**Example:**

```php
// Has the user exceeded 3 orders of type delivery within 7 days?
$validator->checkOrderTypeLimitOrFail('delivery', 3, 7, 1);
```

### 3.7 `getOrderTypeLimits`

```php
public function getOrderTypeLimits(
    $orderTypes,
    ?int $shopId = null,
    string $strategy = 'auto'
): array
```

Purpose: Fetch constraints that match the order type and store only (without product).

### 3.8 `loadActiveLimits`

```php
public function loadActiveLimits(): array
```

Loads all active `product_limit` constraints from cache (or database depending on settings).

### 3.9 `clearCache`

```php
public function clearCache(): void
```

Clears the constraints cache (typically called when a new coupon is saved).

### 3.10 `setUser`

```php
public function setUser(User $user): self
```

Sets the current user (used if changed after object creation).

## 4. Comprehensive Practical Examples

### 4.1 Preventing Addition of a Product That Exceeds the Allowed Limit

```php
use Nano\Coupons\Classes\ProductLimitValidator;
use RainLab\User\Facades\Auth;

$user = Auth::getUser();
$validator = new ProductLimitValidator($user);

try {
    $validator->checkOrFail(
        $cartItem->id,          
        $cartItem->units_id,   
        $cartItem->qty,       
        $this->getCurrentOrderType(),
        $this->getCurrentShopId()
    );
    // Allow addition
} catch (\ApplicationException $e) {
    // Display error message
    return $e->getMessage();
}
```

### 4.2 Fetching Constraints for a Specific Product and Displaying Them

```php
$limits = $validator->getLimitsForProduct(3918);
foreach($limits as $limit){
    echo "Max quantity: " . ($limit['max_quantity'] ?: 'Not specified');
    echo "Max orders: " . ($limit['max_orders'] ?: 'Not specified');
}
```

### 4.3 Checking the Number of Delivery Orders in a Specific Store

```php
$allowed = $validator->check(null, null, 0, 'delivery', 10);
if(!$allowed){
    // Prevent order completion
}
```

## 5. Performance and Design Notes

- **Cache:** Used by default to avoid loading all constraints from the database on every request. Cache is automatically cleared when any `product_limit` coupon is saved (in `afterSave` inside `Coupons_model`).
- **Bulk Queries:** Functions `getBulkHistoricalQuantities` and `getBulkHistoricalOrderCounts` aggregate data for all required products in a single query, reducing database load.
- **Separation of Scenarios:** The two cases (specific product / no product) are clearly separated within `checkOrFail` for readability and maintenance.

## 6. Integration with NanoSoft Plugins

- **`Nano.Cart`:** `ProductLimitValidator` is used inside the `ProductLimit` condition that listens for the `nano.cart.adding` event and prevents item addition immediately.
- **`Nano.Orders`:** Past orders are queried from the `nano_orders_orders` table and their items from `nano_orders_items`, excluding cancelled orders and those in cart state.
- **`Nano.Shop`:** Constraints are linked to products via `product_id` and optionally `units_id` using unit option fetching functions from `Tss.Inventory`.

## 7. Conclusion and Recommendations

- Use `checkOrFail` for immediate checks when adding a product, as it throws an exception with a clear message that can be displayed to the user.
- Use `check` for silent checks when you don't want to handle exceptions.
- Take advantage of the `'direct_query'` strategy when you want to check a single product without loading all constraints from cache.
- Manually clear cache (`clearCache`) only in rare cases; the system automatically clears it when a `product_limit` coupon is saved.
- To customize caching duration, pass `cache_ttl` when creating the object.

Using `ProductLimitValidator`, you can build robust and flexible logic for checking product limits in your application, ensuring excellent performance and a smooth user experience.

