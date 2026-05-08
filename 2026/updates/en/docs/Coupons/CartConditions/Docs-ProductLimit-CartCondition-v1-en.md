# ProductLimit Class Documentation (Cart Condition)

## 1. Introduction

`ProductLimit` is an advanced cart condition (`CartCondition`) within the `Nano.Coupons` plugin, specifically designed to enforce automatic restrictions on products in the shopping cart, without the user needing to enter any coupon code. The condition operates transparently by listening to the `nano.cart.adding` event fired by the cart system (`Nano.Cart`) when attempting to add any item, allowing immediate rejection of the addition if permitted limits are exceeded.

The condition is based on "Product Limit" rules stored in the coupons table (`nano_coupons_coupons`) through the new `product_limit` type, and loads them once per HTTP request with caching (`Cache`) for maximum performance. The actual validation is executed via the independent `ProductLimitValidator` class, which provides high flexibility in handling various scenarios (product, unit, quantity, number of orders, order types, stores, time periods).

## 2. Mechanism of Action and Activation

### 2.1 Automatic Activation

`ProductLimit` requires no manual activation by the end user. It is registered as a cart condition in `Plugin.php` of the `Nano.Coupons` plugin:

```php
\Nano\Coupons\CartConditions\ProductLimit::class => [
    'name' => 'product_limit',
    'label' => 'Product Limit',
    'description' => 'Restrict the quantity of a specific product or its number of orders within a defined period',
],
```

As soon as there is at least one coupon of type `product_limit` with active status (`status = 1`), the condition will automatically load all constraints upon the first cart initialization (`onLoad`) and store them in a static property (`static $activeLimits`) to avoid re-querying during the same request.

### 2.2 Interception Mechanism

When an item is added to the cart, `Nano\Cart\Provider\Cart` fires the `nano.cart.adding` event before the actual addition. `ProductLimit` listens to this event and performs the following:
1. Retrieves the current user.
2. Determines the current order type and store.
3. Calls `ProductLimitValidator::checkOrFail` passing the item data (product, unit, quantity, order type, store).
4. If any limit is exceeded, an `ApplicationException` is thrown, preventing the item from being added while displaying a clear error message.

## 3. Application Scenarios

`ProductLimit` supports two main scenarios, controlled through the coupon settings in the admin panel:

| # | Coupon Setting | Behavior |
|---|----------------|----------|
| 1 | `product_id` = specific product ID, `units_id` optional, `max_quantity` > 0 and/or `max_orders` > 0 | Checks the quantities of the added product and its number of orders within the period, taking into account the order type and store. |
| 2 | `product_id` = empty, `max_orders` > 0 with `order_restriction` and `period_days` specified | Checks the number of orders the user has placed of specific order type(s) within the period, regardless of the product. |

**First Case Details (Specific Product):**
- If `max_quantity` is specified, the total quantities (what is currently in the cart + what was previously purchased within the period + the new quantity) are summed, and the addition is prevented if the limit is exceeded.
- If `max_orders` is specified, the number of previous orders containing this product (or its unit) is counted, and the addition is rejected if the limit is reached.

**Second Case Details (Order Type Only):**
- The product and quantity check is ignored, and only previous orders matching the order type (and store) within the period are counted. For example: "You cannot place more than 3 delivery orders within 7 days."

## 4. Class Features and Benefits

- **Optimized Performance:** Loads constraints once via cache (`Cache::remember`) and static variables, and avoids repetitive `beforeApply` calls.
- **Separation of Logic:** Uses `ProductLimitValidator`, allowing the check to be reused elsewhere (API, CLI).
- **High Flexibility:** Full support for specifying product, unit, quantity, number of orders, period, order types, and stores.
- **Clear User Experience:** Translatable error messages with precise details.
- **Seamless Integration:** Functions as a normal cart condition without affecting other conditions (tax, coupons, gratuity).

## 5. Class Functions

### 5.1 `onLoad()`
Called once when the condition is loaded into the cart. It loads all active `product_limit` constraints and stores them, and registers a listener for the `nano.cart.adding` event.

### 5.2 `getLabel(): string`
Returns the condition label from translation files (`nano.coupons::lang.cart_conditions.product_limit.label`).

### 5.3 `beforeApply()`
(Currently disabled for performance reasons) It used to perform a comprehensive check of the cart before calculating the total. It can be enabled in the future with greater control.

### 5.4 `loadAllProductLimits(): array`
Loads all active coupons of type `product_limit` from cache or the database, filtering out expired coupons. Returns an array of constraints in a unified format.

### 5.5 `validateCartItem(CartItem $cartItem)`
The main entry point for immediate checking when an item is added. It creates a `ProductLimitValidator` object and calls `checkOrFail` with the item data.

**Parameters:**
- `$cartItem`: The `CartItem` object the user is trying to add.

**Exceptions:** `ApplicationException` if limits are exceeded.

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

### 5.6 `isLimitApplicable(array $limit, string $currentOrderType, ?int $currentShopId): bool`
Checks that the given constraint applies to the current order type and store.

### 5.7 `getCurrentOrderType(): string`
Fetches the current order type from `OrderManager` or returns `'delivery'` as a default when unavailable.

### 5.8 `getCurrentShopId(): ?int`
Fetches the current store ID from `OrderManager` or `null`.

### 5.9 `getCartQuantity(int $productId, ?int $unitId): int`
Calculates the total quantity of a specific product/unit in the current cart.

### 5.10 `getBulkHistoricalQuantities($user, array $limits): array`
A single aggregated query to fetch historical quantities (previous purchases) for all products/units existing in the constraints.

### 5.11 `getBulkHistoricalOrderCounts($user, array $limits): array`
An aggregated query to fetch the number of distinct historical orders for each product/unit.

## 6. Practical Examples

### 6.1 Creating a "Product Limit" Coupon in the Admin Panel

1. Go to "Offers and Coupons" > "Coupons".
2. Create a new coupon.
3. From the "Coupon applies on" field, choose "Product Limit".
4. New fields will appear: product (optional), unit, maximum quantity, maximum orders, duration in days.
5. Specify the required values and save.

### 6.2 Example of Intercepting Addition of a Product Exceeding Quantity

We have a coupon preventing the purchase of more than 5 units of product 3918 within 30 days. When attempting to add the 6th unit, the user will receive the following response:

```json
{
  "code": 400,
  "status": false,
  "message": "You have exceeded the allowed quantity limit (maximum 5). Current quantity: 5."
}
```

### 6.3 Example of Limiting the Number of Orders by Order Type

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

## 7. Integration with NanoSoft Plugins

- **`Nano.Cart`:** Integrates via the `CartCondition` interface and listens for the `nano.cart.adding` event fired by `Cart::add()`. The condition is loaded by `CartConditionManager` when building the cart.
- **`Nano.Orders`:** Previous orders are queried from `nano_orders_orders` and `nano_orders_items` with status filtering (excluding `CART` and `CANCELLED`), and the current order type and store are fetched from `OrderManager`.
- **`Nano.Shop`:** Products are fetched via `Nano\Shop\Models\Product`, and units via `Tss\Inventory\Models\Unit` and `ProductsUnit`.
- **`Nano.Coupons`:** Constraint settings are stored in the `nano_coupons_coupons` table using the fields added in version 2.1.0.

## 8. Conclusion and Recommendations

`ProductLimit` represents a qualitative leap in managing product and order limits within NanoSoft applications, providing automatic and effective protection that prevents customers from exceeding the allowed quantities or number of orders. We recommend the following:

- **For Administrators:** Use coupons of type `product_limit` to set precise sales policies that prevent exploitation or stock depletion.
- **For Developers:** You can leverage `ProductLimitValidator` to check limits in any custom context (e.g., when submitting an order via an external API) without needing the cart.
- **For Performance:** Keep cache enabled (`useCache = true`) with an appropriate duration (default 300 seconds), and ensure indexes exist on the columns used in queries (`nano_orders_orders.user_id`, `order_states_ref_type`, `created_at` and `nano_orders_items.products_id`, `units_id`).
- **For Expansion:** `beforeApply` can be enabled later after improving the mechanism to prevent redundant checks, adding an extra layer of protection at order completion.

Thanks to this condition, your online stores can operate more efficiently and flexibly, while ensuring a fair and clear user experience.
