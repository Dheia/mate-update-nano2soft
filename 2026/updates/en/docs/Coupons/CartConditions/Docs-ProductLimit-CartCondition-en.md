# Comprehensive Documentation for the ProductLimit Condition (CartCondition)

## 1. Introduction

`ProductLimit` is an advanced cart condition (`CartCondition`) within the `Nano.Coupons` plugin (NanoSoft platform). This condition differs from traditional conditions (such as discounts and taxes) in that it **does not add any monetary value to the cart**; rather, it acts as a security guard that automatically prevents the addition of products that exceed limits preset by the store management. It relies on coupons of the new type `product_limit` stored in the `nano_coupons_coupons` table.

The condition is based on "Product Limit" rules stored in the coupons table (`nano_coupons_coupons`) through the new `product_limit` type, and loads them once per HTTP request with caching (`Cache`) for maximum performance. The actual validation is performed via the independent `ProductLimitValidator` class, which provides high flexibility in handling various scenarios (product, unit, quantity, number of orders, order types, stores, time periods).

Once there is at least one active coupon of this type, all constraints are loaded once at the beginning of each HTTP request and cached (`Cache`). Then, the condition listens to the `nano.cart.adding` event that fires when attempting to add any item to the cart, to immediately decide whether to accept or reject the addition based on the defined rules.

## 2. Mechanism of Action and Activation

### 2.1 Registration and Loading

The class is registered as a cart condition inside `Nano\Coupons\Plugin.php` via `registerCartConditions`:

```php
\Nano\Coupons\CartConditions\ProductLimit::class => [
    'name' => 'product_limit',
    'label' => 'Product Limit',
    'description' => 'Restrict the quantity of a specific product or its number of orders within a defined period',
],
```

During cart building (via `CartManager` → `CartConditionManager` → `loadRegisteredConditions`), an instance of `ProductLimit` is created and `onLoad()` is called, which:

1. Loads all active `product_limit` coupons (status = 1, not expired) from the database or cache.
2. Stores them in the static property `self::$activeLimits`.
3. Registers a listener for the `nano.cart.adding` event only once (`self::$listenerRegistered`).

### 2.2 Condition Lifecycle

1. **Initialization (`onLoad`):** Load constraints and register listeners.
2. **Early Interception (`nano.cart.adding`):** When any item is added, `validateCartItem` is called, which uses `ProductLimitValidator` to check limits. If exceeded, an exception is thrown preventing the addition.
3. **Total Calculation (`beforeApply`):** Currently disabled for performance reasons (can be enabled for an additional check at the payment stage).

### 2.3 Interception Mechanism

When an item is added to the cart, `Nano\Cart\Provider\Cart` fires the `nano.cart.adding` event before the actual addition. `ProductLimit` listens to this event and performs the following:
1. Gets the current user.
2. Determines the current order type and store.
3. Calls `ProductLimitValidator::checkOrFail` passing the item's data (product, unit, quantity, order type, store).
4. If any limit is exceeded, an `ApplicationException` is thrown, preventing the item from being added while displaying a clear error message.

## 3. Application Scenarios (Supported Scenarios)

Flexibility is the essence of `ProductLimit`'s design. The same coupon can cover multiple cases according to the data entered in the admin panel:

### 3.1 Restricting Quantity of a Specific Product

| Coupon Setting | Behavior |
|----------------|----------|
| `product_id` = product ID | Only this product is checked. |
| `units_id` = unit ID (or empty) | If a unit is specified, the product is checked with that specific unit; otherwise, all units together. |
| `max_quantity` > 0 | The current cart quantity + historical quantity (previous orders within the period) + new quantity are summed. If it exceeds `max_quantity`, it is rejected. |
| `max_orders` > 0 (optional) | Checks how many previous orders contained this product (and unit if specified). If it reaches `max_orders`, the new order is rejected. |
| `period_days` > 0 | Filters previous orders that occurred within the last `X` days. If 0 = all history. |
| `order_restriction` = [...] | The check is only applied to orders of these types (e.g., "delivery"). Empty = all types. |
| `shop_restriction` = [...] | The check is only applied to orders from these stores. Empty = all stores. |

### 3.2 Restricting the Number of Orders by Order Type Only (Without Specifying a Product)

| Coupon Setting | Behavior |
|----------------|----------|
| `product_id` = empty | **No specific product is checked.** Only the number of orders is verified. |
| `max_orders` > 0 | The number of orders the user has made of the specified order type(s) within the period. |
| `period_days` > 0 | Same period concept as above. |
| `order_restriction` = [...] | Determines the targeted order types (e.g., "delivery"). **Must be specified**; otherwise, the constraint will not apply. |
| `shop_restriction` = [...] | Optional to specify stores. |

### 3.3 Quick Examples

- "You cannot purchase more than 10 kg of product 123 (unit 5) within 30 days."
- "You cannot place more than 3 delivery orders within 7 days at any store."
- "Product 55 cannot be ordered more than once per week from the main store."

## 4. Features and Benefits

- **Fully Automatic:** The customer does not need to enter a code; it works silently.
- **High Performance:** Constraints are loaded only once and cached (`Cache`). Bulk historical queries are used.
- **Extreme Flexibility:** Supports constraints on product, unit, quantity, number of orders, period, order type, and store.
- **Separation of Concerns:** Relies on the `ProductLimitValidator` class, which can be reused anywhere (API, CLI).
- **Clear User Experience:** Error messages in Arabic (or any language) with information that helps the customer understand the reason and when they can try again.
- **System Integration:** Works as part of the coupon system without conflicting with discounts or taxes.

## 5. Class Structure and Properties

```php
class ProductLimit extends CartCondition
{
    public $name = 'product_limit';
    public $priority = 250;
    public $removeable = false;

    protected static $activeLimits = null;
    protected static $isLoadActiveLimits = null;
    protected static $listenerRegistered = false;
    protected static $beforeApplyChecked = false;
}
```

- `$name`: The unique identifier of the condition within the cart.
- `$priority`: Application priority (250, after coupons and before tax).
- `$removeable = false`: It cannot be manually removed from the cart (permanent constraint).
- `$activeLimits`: Array of loaded constraints (static across all instances).
- `$listenerRegistered`: Ensures the listener is registered once.
- `$beforeApplyChecked`: Prevents re-checking in `beforeApply`.

## 6. Detailed Function Documentation

### 6.1 `onLoad()`

**Purpose:** Initial setup, load constraints, register event listener.

**Signature:**
```php
public function onLoad(): void
```

**Details:**
- Called once when the condition is loaded in `Cart::loadCondition()`.
- Uses `static::$isLoadActiveLimits` to avoid reloading in the same request.
- Constraints are loaded via `loadAllProductLimits()`.
- Registers a listener for `nano.cart.adding` to call `validateCartItem`.

### 6.2 `getLabel()`

**Purpose:** Returns the condition label for use in display interfaces.

```php
public function getLabel(): string
```
**Example returned value:** `"Product Limit"` (according to the translation file).

### 6.3 `beforeApply()`

**Purpose:** Check the cart before calculating the total. Currently disabled.

```php
public function beforeApply(): bool
```
There is previous code whose execution is stopped (`return true` at the beginning) that used to check all constraints against cart contents. It can be re-enabled in future versions after improving the de-duplication mechanism.

### 6.4 `loadAllProductLimits()`

**Purpose:** Load all active and non-expired `product_limit` coupons.

```php
protected function loadAllProductLimits(): array
```

**Returns:** An array where each element represents a constraint in the form:

```php
[
    'product_id'   => (int) $coupon->product_id,
    'units_id'     => $coupon->units_id ?: null,
    'max_quantity' => (int) $coupon->max_quantity,
    'max_orders'   => (int) $coupon->max_orders,
    'period_days'  => (int) $coupon->period_days,
    'order_types'  => $coupon->order_restriction ?? [],
    'shop_ids'     => $coupon->shop_restriction ?? [],
    'is_expired'   => $coupon->isExpired($now),
]
```

It is stored in `Cache::remember` for 300 seconds.

### 6.5 `validateCartItem(CartItem $cartItem)`

**Purpose:** Check an item when added to the cart and prevent addition if limits are violated.

```php
protected function validateCartItem(CartItem $cartItem): void
```

**Parameters:**
- `$cartItem`: A `CartItem` object containing `id` (product), `units_id`, `qty` (requested quantity).

**Exceptions:** `ApplicationException` with a translated message from `Lang::get()`.

**Internal call example:**
```php
$validator = new \Nano\Coupons\Classes\ProductLimitValidator($user);
$validator->checkOrFail(
    $cartItem->id,
    $cartItem->units_id,
    $cartItem->qty,
    $this->getCurrentOrderType(),
    $this->getCurrentShopId()
);
```

### 6.6 `isLimitApplicable(array $limit, string $currentOrderType, ?int $currentShopId): bool`

Checks that the given constraint applies to the current order type and store.

**Parameters:**
- `$limit`: A single element from `$activeLimits`.
- `$currentOrderType`: Current order type.
- `$currentShopId`: Store ID (may be null).

**Example:**
```php
if ($this->isLimitApplicable($limit, 'delivery', 10)) { ... }
```

### 6.7 `getCurrentOrderType(): string`

Fetches the current order type. It tries to get it from `OrderManager` (the `Nano.Orders` plugin) and returns `'delivery'` on failure.

### 6.8 `getCurrentShopId(): ?int`

Fetches the current store ID from `OrderManager`, or `null` if unavailable.

### 6.9 `getCartQuantity(int $productId, ?int $unitId): int`

Calculates the total quantity of a specific product/unit inside the current user's cart.

### 6.10 `getBulkHistoricalQuantities($user, array $limits): array`

**Purpose:** A single aggregated query to fetch the quantities of all relevant products/units historically (previous orders) related to the current constraints, filtered by statuses, duration, types, and stores.

**Returns:** `[productId][unitKey] = quantity`

### 6.11 `getBulkHistoricalOrderCounts($user, array $limits): array`

A single aggregated query to fetch the number of distinct orders for each product/unit.

**Returns:** `[productId][unitKey] = number of orders`

## 7. Comprehensive Practical Examples

### 7.1 Creating a "Product Limit" Coupon in the Admin Panel

1. Go to "Offers and Coupons" > "Coupons".
2. Create a new coupon.
3. From the "Coupon applies on" field, choose "Product Limit".
4. New fields will appear: product (optional), unit, maximum quantity, maximum orders, duration in days.
5. Specify the required values and save.

### 7.2 Creating a Coupon to Restrict a Specific Product

In the control panel (coupons):

- **Code:** `LIMIT-001`
- **Applies to:** `Product Limit`
- **Product:** "Basmati Rice" (ID=100)
- **Unit:** "5 kg bag" (ID=5)
- **Maximum quantity:** 20
- **Maximum orders:** 0
- **Duration:** 30 days
- **Order type:** `delivery`, `collection`
- **Store:** `Main Branch` (ID=1)

**Result:** Any user trying to add the 21st unit of this product (5 kg bag) within 30 days will receive an error message.

### 7.3 Example of Limiting the Number of Orders by Order Type

A coupon preventing more than 3 orders of type "delivery" within 7 days at store 10:

```
product_id: null
max_orders: 3
period_days: 7
order_restriction: ["delivery"]
shop_restriction: [10]
```

Any attempt to add any product to a "delivery" cart after the user has completed 3 previous orders will be rejected with the message:

> You have exceeded the allowed number of orders (maximum 3). Current order count: 3. Period: 7 days. You can order again after 2026-05-14.

### 7.4 API Invocation Code

Assuming a `CartController`, the class can be used as follows:

```php
use Nano\Coupons\Classes\ProductLimitValidator;

$user = Auth::getUser();
$validator = new ProductLimitValidator($user);

try {
    $validator->checkOrFail(
        $request->product_id,
        $request->units_id,
        $request->quantity,
        $orderType,
        $shopId
    );
    // Continue to add product
} catch (\ApplicationException $e) {
    return response()->json(['error' => $e->getMessage()], 400);
}
```

## 8. Integration with NanoSoft Plugins

- **Nano.Cart:** The condition is loaded via `CartConditionManager` and listens for the `nano.cart.adding` event.
- **Nano.Orders:** Previous orders are queried from `nano_orders_orders` and `nano_orders_items`, excluding `CART` and `CANCELLED`. The current order type and store are taken from `OrderManager`.
- **Nano.Shop:** Constraints are linked to products via `product_id` from the `Product` model.
- **Tss.Inventory:** Units via `Unit` and `ProductsUnit`.
- **Nano.Coupons:** Constraint settings are stored in `nano_coupons_coupons` with the `product_limit` type. Cache is automatically cleared when the coupon is saved via `afterSave`.

## 9. Applied Performance Improvements

- **Loading Constraints Once:** `static $isLoadActiveLimits` prevents reloading the same data from the database during the same request.
- **Cache:** `Cache::remember` with a 5‑minute lifetime reduces database hits.
- **Preventing `beforeApply` Redundancy:** `return true` at the beginning of the function protects against heavy redundant calculations.
- **Bulk Queries:** `getBulkHistoricalQuantities` and `getBulkHistoricalOrderCounts` collect data for all products in one request instead of N requests.
- **Early Interception:** The check is done at `nano.cart.adding`, not only at the payment stage, preventing the accumulation of violating items in the cart.

## 10. Conclusion and Recommendations

`ProductLimit` is the cornerstone for enforcing fair selling policies and preventing the exploitation of offers or the purchase of unauthorized quantities. Thanks to its deep integration with the NanoSoft ecosystem, it can be easily enabled without any additional code.

**Recommendations for Developers:**
- Use `ProductLimitValidator` to check limits in custom scenarios (e.g., when submitting an order via an external API).
- Keep caching enabled (`use_cache = true`) for optimal performance.
- Ensure database indexes exist on `user_id`, `order_states_ref_type`, `created_at` in the orders table, and `products_id`, `units_id`, `orders_id` in the items table.
- The `beforeApply` can be re-enabled in the future after improving the de‑duplication mechanism (using the already present `static $beforeApplyChecked`).

**For Store Owners:**
- Use `product_limit` to set smart purchasing limits that prevent a single customer from depleting stock.
- You can create multiple `product_limit` coupons for different products, and they will all work together without conflict.
- The clear error messages will help your customers understand the constraints without frustration.

In summary, `ProductLimit` adds an advanced and secure control layer to your store, while maintaining excellent performance and a premium user experience.
