# Comprehensive Documentation for the Manual Coupon (Coupon CartCondition)

## 1. Introduction

`Nano\Coupons\CartConditions\Coupon` is one of the core cart conditions (`CartCondition`) within the `Nano.Coupons` plugin (NanoSoft platform). This condition represents the mechanism for applying **manual coupons** to the shopping cart, where the customer explicitly enters a coupon code for it to be activated, unlike automatic coupons (`AutoCoupon`). Once the coupon is successfully activated, the condition adds a discount value (a negative value) to the cart total, reducing the final amount due.

The condition is built on the configurable coupon system and supports:
- Two types of discount: percentage (`P`) or fixed amount (`F`).
- Three application scopes: entire cart (`whole_cart`), delivery fees only (`delivery_fee`), or specific items (`menu_items`).
- Multiple constraints: validity date, order type, store, minimum order value, overall and per-customer usage limits, restriction to specific customers or customer groups.

## 2. Mechanism of Action and Activation

### 2.1 Registration and Scope

The class is registered as a cart condition in `Nano\Coupons\Plugin.php`:

```php
\Nano\Coupons\CartConditions\Coupon::class => [
    'name' => 'coupon',
    'label' => trans('nano.coupons::lang.cart_conditions.coupon.label'),
    'description' => trans('nano.coupons::lang.cart_conditions.coupon.help'),
],
```

The condition is **not automatic**; it requires explicit invocation from `CartManager` when the user enters a coupon code. This is typically done through the `applyCouponCondition` function in `CartManager` or via controllers.

### 2.2 Condition Lifecycle

1. **Loading (`onLoad`):** When the condition is loaded into the cart (after calling `applyCondition`), the code is extracted from `metaData` and the corresponding coupon model (`Coupons_model`) is looked up. If not found, or if it is of type "automatic" (`auto_apply`) or "delivery only", loading is rejected. The coupon validity is also checked via `validateCoupon`.
2. **Before Apply (`beforeApply`):** If the coupon is designated for specific items (`appliesOnMenuItems`) or has an error, application stops and no discount is added.
3. **Discount Calculation (`getActions`):** Determines the discount value (percentage or fixed amount) with a negative sign. In the case of `delivery_fee`, the discount is calculated based on the current delivery charge.
4. **Application to Items (`isApplicableTo`):** If the scope is `menu_items`, `ActsAsItemable` is used to apply the discount only to the items specified in the coupon (products/categories).

## 3. Application Scenarios (Supported Scenarios)

| Scope (`apply_coupon_on`) | Discount Behavior |
|----------------------------|-------------------|
| `whole_cart`               | The discount value is deducted from the cart total (excluding delivery). |
| `delivery_fee`             | The discount value is deducted from delivery fees only (items are unaffected). |
| `menu_items`               | The discount is distributed among the specified items (products or categories) proportionally to their subtotal. |

**Discount Type:**
- `F` (Fixed Amount): Deducts a specific monetary value. Example: `-10` means a 10 Riyal discount.
- `P` (Percentage): Deducts a percentage of the amount. Example: `-10%` means 10% discount.

**Quick Examples:**
- "SAVE20" coupon gives 20% off the entire cart for the first 100 users.
- "FREESHIP" coupon provides free shipping (full deduction of delivery cost).
- "PIZZA10" coupon gives a 10 Riyal discount on pizza items only.

## 4. Features and Benefits

- **Configuration Flexibility:** The discount type, application scope, and constraints can be customized directly from the admin panel.
- **Compatibility with Order Types:** The coupon can be restricted to specific order types (e.g., "delivery" only).
- **Usage Control:** Overall and individual usage limits, with the ability to link the coupon to specific customers.
- **Partial Application on Items:** When `menu_items` is selected, the discount is intelligently distributed only among the targeted items, allowing precise offers without affecting the rest of the cart.
- **Minimum Order Check:** The coupon is not applied if the cart value is below the specified threshold.
- **Integration with the Permission System:** The coupon can be linked to customer groups or specific customers.

## 5. Class Structure and Properties

```php
class Coupon extends CartCondition
{
    use ActsAsItemable;
    use \Nano\Coupons\Traits\HasOrderManager;

    public $removeable = true;
    public $priority = 202;

    protected static $couponModel;
    protected static $applicableItems;
    protected static $hasErrors = false;
}
```

- `$removeable = true`: The coupon can be removed from the cart by the user.
- `$priority = 202`: Application priority (before `ProductLimit` and after `auto_coupon`).
- `$couponModel`: The current coupon model (cached in memory for the session).
- `$applicableItems`: A list of item IDs the coupon applies to (in the case of `menu_items`).
- `$hasErrors`: A flag indicating an error with the coupon to prevent retries.

## 6. Detailed Function Documentation

### 6.1 `getLabel()`

**Purpose:** Returns the condition label with the coupon code.

```php
public function getLabel(): string
```

**Example output:** `"Coupon [SAVE20]"` (according to the translation file).

### 6.2 `getValue()`

**Purpose:** Returns the actual discount value (negative total).

```php
public function getValue(): float
```
Internally, `calculatedValue` is computed as a positive discount amount, so it returns `0 - $this->calculatedValue`.

### 6.3 `getModel()`

**Purpose:** Retrieves the coupon model from the database based on the code stored in `metaData`.

```php
public function getModel(): ?Coupons_model
```

**Behavior:**
- If no code exists, returns `null`.
- Uses static caching (`static::$couponModel`) to avoid repeated queries.
- If the code changes during the session, the query is re-executed.

**Exceptions:** Does not throw an exception, but returns `null` on failure.

### 6.4 `getApplicableItems($couponModel)`

**Purpose:** Builds a list of items (products) to which the coupon applies and stores them in `static::$applicableItems`.

```php
public function getApplicableItems($couponModel): \Illuminate\Support\Collection
```

**Parameters:** `$couponModel` the coupon model.

**Logic:**
1. Collects `menu_id` from the direct `menus` relation.
2. For each associated category (`categories`), adds all products belonging to that category.

**Example:** If the coupon is linked to the "Drinks" category and the "Burger" product, the list will include all drinks + the burger.

### 6.5 `onLoad()`

**Purpose:** Loads the coupon and validates it when added to the cart.

```php
public function onLoad(): void
```

**Details:**
- If the code is empty or there is a previous error (`$hasErrors`), exits immediately.
- Calls `getModel()` to find the coupon.
- If not found, throws an "Invalid coupon" exception.
- Calls `validateCoupon()` to check the conditions.
- Calls `getApplicableItems()` to prepare the items list.
- On any error, empties `metaData['code']` (cancelling the coupon).

### 6.6 `beforeApply()`

**Purpose:** Prevents applying the coupon if it is designated for items only or if there are errors.

```php
public function beforeApply(): bool
```

**Logic:**
- If `appliesOnMenuItems()` or `$hasErrors`, returns `false` (no discount is added at the cart level, instead it's applied at the item level via `ActsAsItemable`).

### 6.7 `getActions()`

**Purpose:** Returns the discount action to be applied to the cart.

```php
public function getActions(): array
```

**Logic:**
- The discount value comes from `discountWithOperand()`: if fixed `-10`, if percentage `-10%`.
- If the scope is `delivery_fee`, the value is calculated via `calculateDeliveryDiscount()`.
- If the scope is `menu_items` and the discount is fixed (not percentage), it is distributed among the relevant items (`calculateApportionment`).

**Example return:**
```php
[['value' => '-10%']]
```

### 6.8 `validateCoupon($couponModel)`

**Purpose:** Comprehensive validation of the coupon against all constraints.

```php
protected function validateCoupon(Coupons_model $couponModel): void
```

**Checks in order:**
1. The coupon must not be automatic (`auto_apply = true`).
2. It must not be designated for delivery only (`appliesOnDelivery`).
3. Date check: `isExpired()`.
4. Current order type matches `order_restriction` constraints.
5. Current store (`shop_restriction`) check.
6. Minimum order total check (`hasMinimumOrderTotal`).
7. Overall usage exhausted check.
8. Customer-specific usage exhausted check.
9. The customer is among the allowed customers/groups.

**Exceptions:** `ApplicationException` with an appropriate message for the first violation.

### 6.9 `isApplicableTo($cartItem)`

**Purpose:** Determines whether the discount is applied to a specific item (used only when `menu_items`).

```php
public static function isApplicableTo($cartItem): bool
```

**Conditions:**
- Must have a `$couponModel` and no errors.
- The coupon must be `appliesOnMenuItems()`.
- The item's ID must exist in `$applicableItems`.

### 6.10 Discount Calculation Functions

#### `calculateDeliveryDiscount()`

**Purpose:** Calculate the shipping discount based on the actual shipping cost.

**Logic:**
- Retrieves the shipping cost from `OrderManager`.
- If the coupon is fixed (`F`) and the discount exceeds the shipping cost, the discount is capped at the shipping cost.
- If percentage, calculates the percentage of the shipping cost.

#### `calculateApportionment($value)`

**Purpose:** Distribute a fixed discount among applicable items according to their proportion of the total of targeted items.

**Formula:** `(subtotalWithoutConditions of targeted items / total subtotalWithoutConditions of all targeted items) * discount value`.

### 6.11 Helper Functions in `hasOrderManager`

The class uses `HasOrderManager` to conveniently access `OrderManager` and obtain `orderType`, `orderDateTime`, `shopId`, `subtotal`... etc.

## 7. Comprehensive Practical Examples

### 7.1 Creating a 10% Off Entire Cart Coupon

In the admin panel:
- Code: `SUMMER10`
- Type: Percentage (`P`)
- Discount: `10`
- Applies to: `whole_cart`
- Duration: until 2026-08-31
- Overall usage: 100
- Per-customer usage: 1

**Usage via API:**
```php
$cartManager->applyCouponCondition('SUMMER10');
```

**Result:**
- If the cart total is 200 Riyal, the total after discount becomes `180` Riyal.
- Cart details show: "Coupon [SUMMER10]: -20.00"

### 7.2 Free Shipping Coupon

- Code: `FREESHIP`
- Type: Fixed (`F`)
- Discount: `0` (maximum shipping cost will be calculated)
- Applies to: `delivery_fee`
- No other constraints.

**Usage:**
```php
$cartManager->applyCouponCondition('FREESHIP');
```

**Result:** If shipping fees are 15 Riyal, 15 Riyal is deducted from the cart total (making shipping free).

### 7.3 Coupon on Specific Items

- Code: `BURGER5`
- Type: Fixed (`F`)
- Discount: `5`
- Applies to: `menu_items`
- Associated products: "Beef Burger", "Chicken Burger"

**Usage:**
```php
$cartManager->applyCouponCondition('BURGER5');
```

**Result:** If the cart contains two burgers and a salad, the 5 Riyal discount is distributed over the two burgers only (not the salad). The discount per burger is calculated based on its price.

## 8. Integration with NanoSoft Plugins

- **Nano.Cart:** The condition uses `CartCondition` and is added via `CartManager::applyCondition`. The total after discount is calculated inside `CartConditions::apply`.
- **Nano.Orders:** In `validateCoupon`, `orderDateTime`, `orderType`, `shopId` are obtained from `OrderManager` or default values. Also uses `Cart::subtotal()` to check the minimum order requirement.
- **Nano.Shop:** Items linked to the coupon are fetched from `Nano\Shop\Models\Product`, and categories from `Nano\Tags\Models\Categorie`.
- **Tss.Basic:** Stores are checked via `siteLocation` or `OrderManager`.

## 9. Performance Optimizations and Notes

- **Static Caching:** `$couponModel` and `$applicableItems` are stored in `static` properties to avoid repeated queries within the same request.
- **Error Handling:** When `validateCoupon` fails, the code is cleared from `metaData` so it is not reloaded in subsequent attempts, displaying a warning message (`Flash::warning`) if the coupon is not automatic.
- **Limiting Discount Application on Items:** When `menu_items`, `beforeApply` returns `false`, meaning the discount is not applied twice (once on the cart and once on items).
- **Version Compatibility:** Uses `HasOrderManager` to safely handle the absence of `OrderManager`.

## 10. Conclusion and Recommendations

The `Coupon` condition is the backbone of the manual discount system in your store. Thanks to its flexibility, you can create diverse offers that suit all marketing strategies.

**Recommendations for Administrators:**
- Use `whole_cart` for general discounts on all purchases.
- Use `delivery_fee` to encourage customers to order with free delivery.
- Use `menu_items` to provide discounts on specific categories or products without affecting the rest of the cart.
- Leverage "usage limit" and "minimum order" constraints to achieve specific profitability goals.

**Recommendations for Developers:**
- When invoking a coupon, make sure to use `applyCouponCondition` in `CartManager` and not `applyCondition` directly, to ensure checks are executed.
- If you wish to create custom coupons with complex logic, you can extend this class or use `AutoCoupon` for automatic application.
- Ensure `OrderManager` is present or provide appropriate default values in its absence.

With this condition, your store can provide a smooth and secure discount experience, with full control over usage terms.
