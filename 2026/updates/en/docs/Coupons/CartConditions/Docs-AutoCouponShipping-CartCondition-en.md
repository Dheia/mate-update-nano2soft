# Comprehensive Documentation for the AutoCouponShipping Condition (Free / Automatic Shipping Discount)

---

## 1. Introduction

`Nano\Coupons\CartConditions\AutoCouponShipping` is a specialized cart condition within the `Nano.Coupons` plugin (NanoSoft platform). It automatically applies a discount **only on shipping fees** without any user intervention. Unlike the `AutoCoupon` condition, which targets the entire cart or specific items, `AutoCouponShipping` exclusively concerns the shipping charge, making it the ideal tool for "Free Shipping" or "Reduced Shipping" offers that activate automatically when certain conditions are met.

The coupon is automatically linked to coupons of type `delivery_fee` and `auto_apply = 1`, and is loaded by `CartManager` at request time or when automatic coupons are applied (via `bindCouponsEvent`). All validity checks (date, store, order type, minimum order total, usage limits, specific customers) apply here with the same rigor as a manual coupon, providing full control with ease of use.

---

## 2. Mechanism of Action and Activation

### 2.1 Registration as a Cart Condition

The class is registered in `Nano\Coupons\Plugin.php`:

```php
\Nano\Coupons\CartConditions\AutoCouponShipping::class => [
    'name' => 'auto_coupon_shipping',
    'label' => trans('nano.coupons::lang.cart_conditions.auto_coupon_shipping.label'),
    'description' => trans('nano.coupons::lang.cart_conditions.auto_coupon_shipping.help'),
],
```

### 2.2 Interaction with the Cart

When an item is added to the cart or a saved cart is restored, `bindCouponsEvent` in `Plugin.php` calls `CartManager::applyAutoCouponShippingCondition($code)` for each automatic coupon that targets delivery. An `AutoCouponShipping` object is created and loaded into the cart as a condition, where:

1. **Loading (`onLoad`):** Checks for a code, finds the coupon, and verifies it is `auto_apply` and on `delivery_fee`. Executes all validity checks.
2. **Before Apply (`beforeApply`):** Applies the discount only if the coupon is automatic and indeed targets delivery; otherwise, stops.
3. **Discount Calculation (`getActions`):** Calculates the discount based on the actual shipping cost provided by `OrderManager` or the `deliveryCharge` property. If the coupon is fixed and exceeds the shipping cost, the discount is capped at the full shipping charge (resulting in free shipping).
4. **Applying the Discount (`apply`):** The discount is added to the cart total as a negative value that reduces the final amount.

### 2.3 The `inclusive` Flag

Note that `getActions` returns `'inclusive' => true`, meaning the discount is **inclusive** and is not treated as a separate discount outside the total. Instead, the total is recalculated directly after subtracting the shipping, providing a more accurate result in invoice display.

---

## 3. Application Scenarios (Supported Scenarios)

The supported scenarios can be summarized in the following table:

| Coupon Setting | Discount Behavior |
|----------------|-------------------|
| `apply_coupon_on = 'delivery_fee'` and `auto_apply = 1` | **Applied automatically.** |
| `type = 'F'` (Fixed Amount) | Deducts the specified amount, up to the shipping cost (free shipping if the amount ≥ shipping cost). |
| `type = 'P'` (Percentage) | Deducts the percentage of the actual shipping cost. |
| `min_total > 0` | The coupon is not activated if the cart total (excluding shipping) is below the minimum. |
| `order_restriction` specified | Restricted to specified order types. |
| `shop_restriction` specified | Restricted to specific stores. |
| `period_days / validity` | (Present in `product_limit` coupon but regular coupon also has time validity) – respects the validity period. |
| `customer_redemptions / redemptions` | Respects overall and customer usage limits. |

**Examples:**
- "Free automatic delivery on Fridays for all orders over 100 Riyal."
- "50% discount on shipping for new customers for the first 3 delivery orders."
- "Shipping for only 1 Riyal at the Riyadh store throughout the month."

---

## 4. Features and Benefits

- **Silent and Automatic Application:** The customer does not need to enter any code.
- **Focused on Delivery Only:** Does not interfere with item or total cart discounts.
- **Discount Flexibility:** Supports fixed amount and percentage.
- **Automatic Discount Capping:** Ensures the discount does not exceed the shipping charge itself, avoiding negative values.
- **Integration with `OrderManager`:** Retrieves the real shipping cost based on distance or settings.
- **Reuse of Full Coupon Infrastructure:** All constraints on dates, customers, and usage are supported.

---

## 5. Class Structure and Properties

```php
class AutoCouponShipping extends CartCondition
{
    use ActsAsItemable;
    use \Nano\Coupons\Traits\HasOrderManager;

    public $removeable = true;
    public $priority = 350;
    protected $deliveryCharge = 0;

    protected static $couponModel;
    protected static $applicableItems;
    protected static $hasErrors = false;
}
```

- `$priority = 350`: A relatively late priority (after main discounts and before final total).
- `$deliveryCharge`: The current shipping cost upon which the discount is based. Cached during the request and can be set externally.
- `$removeable = true`: The coupon can be removed if the user cancels or shipping conditions change.
- `static $couponModel, $applicableItems, $hasErrors`: The usual static caching mechanism.

---

## 6. Detailed Function Documentation

### 6.1 `getLabel()`

**Purpose:** Describes the condition in display interfaces. Shows the coupon code.

**Example output:** `"Auto Coupon Shipping [FREESHIP]"`.

### 6.2 `getValue()`

Returns `0 - calculatedValue`, i.e., the discount as a negative value.

### 6.3 `getModel()`

Looks up the `coupons_model` using `getByCode` with static caching.

### 6.4 `getApplicableItems($couponModel)`

(Used only if we wanted to apply the condition to specific items – but in the case of `delivery_fee` it is not actually used. Exists for inheritance and compatibility with `ActsAsItemable`).

### 6.5 `onLoad()`

- Checks that the code is non-empty and there are no previous errors.
- Calls `getModel()`, then `validateCoupon()`.
- On failure (e.g., the coupon is not automatic or not for delivery), clears the code and exits.

### 6.6 `beforeApply()`

- Allows application only if the coupon is automatic (`auto_apply`), targets delivery (`appliesOnDelivery`), and there are no errors.
- If the condition is not met, returns `false` and no discount is added.

### 6.7 `getActions()`

**Purpose:** Generate the discount rule.

- If the coupon targets delivery, the value is calculated via `calculateDeliveryDiscount()`.
- Returns an array containing the value with `'inclusive' => true`.

### 6.8 `calculateDeliveryDiscount()`

**Purpose:** Calculate the shipping discount based on the actual shipping charge.

**Steps:**
1. Retrieve the shipping cost from `OrderManager` or from `$this->getDeliveryCharge()`.
2. If the coupon is fixed (`F`):
   - If the discount amount exceeds the shipping cost → discount = shipping cost (free delivery).
   - Otherwise, discount = coupon amount.
3. If percentage (`P`): calculates the percentage of the shipping cost.
4. Store the discount type (`discount_type`) and its value (`discount_value`) in `metaData` for later reference.

**Example:** Shipping 20 Riyal, coupon 15% → discount = 3 Riyal, customer pays 17 Riyal shipping.

### 6.9 `validateCoupon($couponModel)`

A comprehensive check (similar to `AutoCoupon` but ensures the coupon targets delivery):

- Must be `auto_apply`.
- Must be `appliesOnDelivery`.
- Date validity.
- Order type (`order_restriction`) and store (`shop_restriction`).
- Minimum cart amount (`min_total`).
- Overall and specific coupon usage.
- Allowed customers / customer groups.

### 6.10 `isApplicableTo($cartItem)`

(Not actually used in the context of delivery because the discount is on the cart, not the item. Exists for inheritance).

### 6.11 Shipping Charge Management Functions

- `getDeliveryCharge()`: Returns the stored shipping charge, otherwise tries to fetch it from `metaData`.
- `setDeliveryCharge($value)`: Manually sets the shipping charge and stores it in `metaData`.

---

## 7. Practical Examples

### 7.1 Free Automatic Delivery for Orders Over 200 Riyal

**Coupon:**
- Code: `FREE200`
- Automatic: Yes
- Applies to: Delivery fees
- Type: Fixed amount (`F`)
- Discount: `0` (will be capped at shipping cost)
- Minimum order: 200

**Behavior:**
- When the user adds products worth 200+ Riyal and chooses delivery, the condition is automatically added.
- The shipping cost (e.g., 20 Riyal) is fully deducted, resulting in a final total with no shipping fees.
- If the cart is below 200 Riyal, the coupon is not activated.

### 7.2 50% Shipping Discount at Riyadh Store

**Coupon:**
- Code: `RIYADH50`
- Automatic: Yes
- Applies to: Delivery fees
- Type: Percentage (`P`)
- Discount: `50`
- Store: Riyadh (ID=10)
- No minimum

**Behavior:**
- Any delivery order from the Riyadh store will automatically receive a 50% discount on shipping fees.

---

## 8. Integration with NanoSoft Plugins

- **Nano.Cart:** Loaded as a regular cart condition via `CartConditionManager`, and the discount is added to the cart total during `CartConditions::apply`.
- **Nano.Orders:** Heavily relies on `OrderManager` to extract the current shipping cost (`getShippingCost`). Without `OrderManager`, the discount might not work correctly.
- **Nano.Coupons:** Registered in `registerCartConditions`, and activated via `applyAutoCouponShippingCondition` in `CartManager` (called from `bindCouponsEvent`).
- **Nano.Shop / Nano.Tags:** Not directly required (since the condition does not apply to specific items).

---

## 9. Performance Optimizations and Notes

- **Caching Shipping Charge:** To avoid calling `OrderManager` multiple times, the value is cached in `$deliveryCharge` during the request lifecycle.
- **Handling `OrderManager` Errors:** If the shipping cost cannot be retrieved (missing plugin or error), `deliveryCharge` is set to `0`, nullifying the discount.
- **`inclusive` Value:** Note that `'inclusive' => true` means `calculateActionValue` will not add or subtract the value but consider it included. This may need review in financial reporting scenarios, but it works correctly for the final total calculation.
- **Scope of Application:** The coupon does not apply to specific items as in `AutoCoupon`, so `getApplicableItems` and `isApplicableTo` have no real effect.

---

## 10. Conclusion and Recommendations

`AutoCouponShipping` is a powerful condition for applying automatic shipping discounts, enhancing "free shipping" and "reduced shipping" strategies automatically. With full support for various coupon constraints, you can create attractive and profitable shipping offers.

**Recommendations for Administrators:**
- Ensure the `Nano.Orders` plugin is enabled to guarantee accurate shipping calculation.
- Use the coupon with a minimum order requirement to encourage customers to increase cart value.
- Combine it with cart coupons (`AutoCoupon`) to offer comprehensive deals (item discount + free shipping).

**Recommendations for Developers:**
- If the application does not use `Nano.Orders`, you can set `deliveryCharge` directly via `setDeliveryCharge()` before applying the condition.
- The class can be extended to customize shipping calculation behavior (e.g., support for multiple shipping companies).

This condition represents an effective marketing tool that increases customer satisfaction and reduces cart abandonment due to shipping costs.
