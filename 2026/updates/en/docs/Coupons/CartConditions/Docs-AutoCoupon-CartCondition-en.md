# Comprehensive Documentation for the AutoCoupon CartCondition

## 1. Introduction

`Nano\Coupons\CartConditions\AutoCoupon` is an advanced cart condition (`CartCondition`) within the `Nano.Coupons` plugin (NanoSoft platform). It represents a mechanism for applying **automatic coupons** without any user intervention. Unlike the `Coupon` condition, which requires manual entry of a code, `AutoCoupon` operates silently and invisibly, being automatically activated whenever any item is added to the cart. This occurs for every enabled coupon of type "automatic" (`auto_apply = 1`) that targets either the entire cart (`whole_cart`) or specific items (`menu_items`).

This condition leverages the `nano.cart.added` event that the system listens for inside `Nano\Coupons\Plugin`, automatically adding all applicable automatic coupons to the cart. Then, `AutoCoupon` takes over, verifying the validity of each coupon upon loading (`onLoad`), and subsequently applying the discount either to the whole cart or to specific items.

## 2. Mechanism of Action and Activation

### 2.1 Activation

The condition is registered in `Nano\Coupons\Plugin.php`:

```php
\Nano\Coupons\CartConditions\AutoCoupon::class => [
    'name' => 'auto_coupon',
    'label' => trans('nano.coupons::lang.cart_conditions.auto_coupon.label'),
    'description' => trans('nano.coupons::lang.cart_conditions.auto_coupon.help'),
],
```

The automatic coupon is invoked automatically for every active coupon of type `auto_apply = 1` that does not target only delivery fees. This is done via `bindCouponsEvent` in `Plugin.php`, which listens to `nano.cart.added` and calls `CartManager::applyAutoCouponCondition($code)`.

### 2.2 Condition Lifecycle

1. **Loading (`onLoad`):** Extracts the code from `metaData` and looks up the coupon. Verifies that it is `auto_apply` and does not target delivery only. Executes `validateCoupon` to check all constraints. Upon successful validation, it prepares the list of applicable items (`getApplicableItems`).
2. **Before Apply (`beforeApply`):** Stops application at the cart level if the coupon is for items (`menu_items`) or delivery fee (`delivery_fee`) or if there are errors. In other words, the automatic coupon applies only to the regular cart total (`whole_cart`).
3. **Discount Calculation (`getActions`):** Based on the coupon type and scope, produces the discount value. If it is `menu_items`, the fixed (non-percentage) discount is distributed among the targeted items.
4. **Application to Items (`isApplicableTo`):** In the case of `menu_items`, uses `ActsAsItemable` to apply the discount to each item within the list of eligible items.

## 3. Application Scenarios (Supported Scenarios)

| Scope (`apply_coupon_on`) | Discount Behavior |
|----------------------------|-------------------|
| `whole_cart`               | Applies the discount to the whole cart (items only, excluding delivery). |
| `menu_items`               | Distributes the discount (if fixed) among the items present in the list of products/categories linked to the coupon. |
| `delivery_fee`             | **Not supported in `AutoCoupon`** – rejected in `onLoad` and `beforeApply`. |

**Discount Type:**
- `F` (Fixed): A specific amount off.
- `P` (Percentage): A percentage off.

**Quick Examples:**
- "WELCOME10" coupon automatically grants 10% off to every new user on their first order.
- "BURGERLOVER" coupon gives a 5 riyal discount on all burger items in any order.
- "SUMMERSALE" coupon provides 15% off the entire cart during weekends.

## 4. Features and Benefits

- **Fully Automatic:** The customer does not need to know any code; it works silently.
- **Precise Targeting:** The coupon can be linked to specific categories or products to apply only to them.
- **Time and Store Control:** Like a regular coupon, it respects validity dates, order types, and stores.
- **Integration with the Permission System:** It can be restricted to specific customers or groups.
- **Misuse Protection:** It respects overall and customer-specific usage limits.

## 5. Class Structure and Properties

```php
class AutoCoupon extends CartCondition
{
    use ActsAsItemable;
    use \Nano\Coupons\Traits\HasOrderManager;

    public $removeable = true;
    public $priority = 200;

    protected static $couponModel;
    protected static $applicableItems;
    protected static $hasErrors = false;
}
```

- `$removeable`: The automatic coupon can be removed from the cart (e.g., if the user cancels it or eligible products are removed).
- `$priority = 200`: Higher priority than the manual coupon (202) and the product limit (250).
- `$couponModel`, `$applicableItems`, `$hasErrors`: Operate with the same static caching mechanism.

## 6. Detailed Function Documentation

### 6.1 `getLabel()`

Returns the condition label including the coupon code.

### 6.2 `getValue()`

Returns `0 - calculatedValue` (a negative value representing the discount).

### 6.3 `getModel()`

Looks up the coupon model using `coupons_model::getByCode` with static caching to avoid repeated queries.

### 6.4 `getApplicableItems($couponModel)`

Collects eligible items from the `menus` and `categories` relations in the model. Used only when `apply_coupon_on = 'menu_items'`.

### 6.5 `onLoad()`

- Checks for a code and that there are no previous errors.
- Finds the coupon and ensures it is `auto_apply` and not `delivery_fee`.
- Executes `validateCoupon`, and if successful, prepares the items list.
- On failure, clears the code from `metaData` to disable the coupon.

### 6.6 `beforeApply()`

Returns `false` if the coupon targets items or delivery or has an error, meaning the condition does not add a discount at the total cart level, leaving application to the item level via `ActsAsItemable`.

### 6.7 `getActions()`

Produces the discount rules:
- In the case of `delivery_fee` (used in `AutoCouponShipping` only) – not allowed here.
- If `menu_items` and the discount is fixed, distributes it (`calculateApportionment`).
- Otherwise, returns the discount value as is (percentage or fixed).

### 6.8 `validateCoupon($couponModel)`

Checks in order:
1. Must have `auto_apply = true`.
2. Must not be for delivery only (`appliesOnDelivery`).
3. Date validity, order type, store.
4. Minimum order total.
5. Overall and customer-specific usage limits.
6. Allowed customers/groups.

### 6.9 `isApplicableTo($cartItem)`

Determines whether a cart item is eligible for the discount (when `menu_items`). Relies on `$applicableItems`.

### 6.10 Helper Functions (`calculateApportionment`)

Distributes the fixed discount value among eligible items according to each item's percentage share of the total of eligible items.

## 7. Comprehensive Practical Examples

### 7.1 Automatic 10% Welcome Coupon

- Code: `WELCOME`
- Automatic: Yes
- Type: Percentage 10%
- Applies to: Whole cart
- First order only: Enabled

**Result:** When the first product is added, a 10% discount on the cart value is automatically applied.

### 7.2 Automatic Category Discount Coupon

- Code: `DRINKS5`
- Automatic: Yes
- Type: Fixed 5 Riyal
- Applies to: Specific items (Drinks category)

**Result:** Whenever a drink is added to the list, 5 Riyal is deducted from the total of drinks (the amount is distributed proportionally if there are multiple drinks).

## 8. Integration with NanoSoft Plugins

- **Nano.Cart:** Uses `CartCondition` and is added via `CartManager::applyAutoCouponCondition`.
- **Nano.Orders:** Leverages `OrderManager` for checking `orderType`, `shopId`, and date.
- **Nano.Shop/Nano.Tags:** For loading associated products and categories.
- **Nano.Coupons:** Cache is cleared when a coupon is saved, and constraints are reloaded automatically.

## 9. Performance Optimizations and Notes

- Static caching of the model and items list reduces queries.
- Errors prevent re-attempting to apply the coupon in the same session.
- The `nano.cart.added` event applies automatic coupons instantly when a product is added, providing a seamless experience.

## 10. Conclusion and Recommendations

`AutoCoupon` is a powerful tool for applying automatic discounts without user friction. You can use it to create welcome offers, discounts on specific categories, or seasonal marketing campaigns that operate silently.

**Recommendations for Administrators:**
- Use the automatic coupon cautiously with `has_first_order` to avoid excessive discounts.
- Ensure `max_orders` or `max_quantity` is set if the offer is limited.
- Combine it with `ProductLimit` to prevent repeat exploitation of the same product.

**Recommendations for Developers:**
- Note that `AutoCoupon` does not support `delivery_fee`; use `AutoCouponShipping` for that purpose.
- You can customize the loading behavior by overriding `onLoad` in an extended class.

`AutoCoupon` is a cornerstone of the intelligent marketing system in your online store.

