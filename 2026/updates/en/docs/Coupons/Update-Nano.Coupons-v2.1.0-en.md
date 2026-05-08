## 2026-04-25 – 2026-04-27

**`Nano.Coupons` Add-on Update – Adding the "Product Limit" Feature and Order Quantity Restriction System**

---

### Summary of Updates

A completely new feature has been developed and released in the `Nano.Coupons` add-on called **"Product Limit"**, which allows administrators to create automatic restriction rules on specific product quantities or the number of times they can be ordered within a given period. The rule works instantly: when a customer tries to add a violating product to the cart, the addition is rejected with a clear error message, and verification is performed again when attempting to complete the order.

The feature is built on the existing coupon structure and the cart conditions system in `Nano.Cart`, and has been extended by adding a new type `product_limit` alongside the existing types (`whole_cart`, `delivery_fee`, `menu_items`). An integrated management interface has been provided via new fields in the coupon form, with support for translation and field visibility based on the selected type. The system respects all other constraints such as coupon validity period, order type, and the specified store.

---

### 1. Objectives of the Updates

- **Enable the creation of automatic product restriction rules** without the need for custom code or manual developer intervention.
- **Prevent exceeding maximum quantities** of products or units within a specified time period, or exceeding the number of orders.
- **Seamless integration with the cart system** via the `ProductLimit` condition that listens to the `nano.cart.adding` event and prevents immediate addition.
- **Leverage the existing coupon model** to store rules, allowing reuse of date validity, order types, and store permissions.
- **Easy management interface** using a `recordfinder` field for product selection and numeric options for limits.
- **Complete documentation** of code changes and translation keys used.

---

### 2. Developed Components

#### 2.1 `Coupon` Model / `Coupons_model`

- **Added the new type `product_limit`** to the `getApplyCouponOnOptions` function.
- **Added new properties** to the `$casts` array to support integer values.
- **Added new columns** to the database via a migration:
  - `product_id` (int) – product ID
  - `units_id` (int, nullable) – unit ID (optional)
  - `max_quantity` (int, default 0) – maximum quantity
  - `max_orders` (int, default 0) – maximum number of orders
  - `period_days` (int, default 0) – period in days
- **Provided dropdown option functions** (`getProductIdOptions`, `getUnitsIdOptions`) to support the admin interface.

#### 2.2 Cart Condition `ProductLimit`

File: `plugins/nano/coupons/cartconditions/ProductLimit.php`

- **New class** extending `CartCondition` that registers itself automatically.
- **Loads active restrictions** from all active `product_limit` coupons once on first use.
- **Listens to the `nano.cart.adding` event** to immediately check any item being added to the cart and reject it if it violates the rules.
- **Additional check in `beforeApply`** to verify the entire cart before completing the order (handles the case of restoring a saved cart).
- **Respects additional constraints**: order type (`order_restriction`), store (`shop_restriction`), coupon expiry date, and general/individual usage limits.
- **Precise queries** to aggregate historical quantities and past order counts from the `nano_orders_orders` and `nano_orders_items` tables, filtering by status (excluding `CART` and `CANCELLED`).
- **Translation support** via `Lang::get` keys for messages, with variable substitution.
- **Safety handling** to avoid duplicate listener registration and to prevent errors when `OrderManager` is absent.

#### 2.3 Admin Interface (fields.yaml)

- **Added new fields**: `product_id`, `units_id`, `max_quantity`, `max_orders`, `period_days`.
- **Used `trigger`** to show these fields only when `apply_coupon_on = product_limit`.
- **`product_id` field** of type `recordfinder` with `useRelation: false` and `modelClass` to search for products.
- **Translation support** for all field properties: `label`, `commentAbove`, `placeholder`, `emptyOption`, `title`, `prompt`.

#### 2.4 Language Files

- **Added Arabic and English translation keys** for error messages (`max_quantity_exceeded`, `max_orders_exceeded`) and for field labels in the admin interface.
- **Extended `apply_coupon_on_options` keys** to include `product_limit`.

#### 2.5 Migration File

- `add_product_limit_fields.php` to add the five columns to the `nano_coupons_coupons` table, with `Schema::hasColumn` checks to avoid duplication.

---

### 3. How It Works (Flow)

1. **Setup (once)**: The administrator creates a new coupon of type `product_limit`, specifying the product, optionally the unit, maximum quantity, maximum number of orders, period in days, and allowed order types and stores.

2. **Condition activation**: When the cart system loads, `ProductLimit` loads all active coupons of this type and caches them. It registers a listener for the `nano.cart.adding` event.

3. **Adding a product to the cart (immediate check)**: When a user attempts to add a product:
   - The condition checks whether the product and unit match any active rule (considering the current order type and store).
   - It calculates the current quantity in the cart + the historical quantity from past orders + the new quantity.
   - If the total exceeds `max_quantity`, an `ApplicationException` is thrown and an error message is shown to the user.
   - Similarly, if the historical order count has already reached `max_orders`, the addition is rejected.

4. **Order completion (additional check)**: When `beforeApply` is called (calculating the cart total), the same checks are repeated to ensure that the current cart does not contain violating quantities (for example, due to restoring a saved cart).

5. **Error messages**: The user sees clear messages such as "You have exceeded the allowed quantity limit (maximum 5)." or "You have exceeded the allowed order count limit (maximum 3).", with translation support.

---

### 4. Key Achievements and Features

- **Fully automatic**: The rule works immediately after creating an active `product_limit` coupon, without any additional code.
- **Deep integration with the coupon system**: Benefits from date validity, order types, and store permissions without reinventing them.
- **Immediate rejection on addition**: Immediate handling prevents violating products from reaching the cart, rather than waiting until the checkout stage.
- **Performance optimisation**: Restrictions are loaded only once per HTTP request and stored in a static variable.
- **High flexibility**: You can restrict a specific product regardless of its unit, or restrict a specific unit of that product.
- **Multiple rules support**: Several rules for different products can work simultaneously.
- **Safe error handling**: All queries are inside `try-catch` blocks and do not affect cart stability if a query fails.
- **Complete documentation** in the update file with examples of translations and fields.

---

### 5. Requirements and Upgrade

- **Upgrade the add-on**: Replace the following files:
  - `Nano/Coupons/Models/Coupon.php` (or `Coupons_model.php`)
  - `Nano/Coupons/CartConditions/ProductLimit.php` (new)
  - `Nano/Coupons/updates/add_product_limit_fields.php` (new)
  - `Nano/Coupons/models/coupon/fields.yaml`
  - `Nano/Coupons/lang/ar/lang.php` and `lang/en/lang.php`
- **Run the migration**: `php artisan october:up` to apply the new columns.
- **Update existing records**: Does not affect existing coupons (they retain default values for `product_id`, etc.).
- **Compatibility testing**: It is recommended to create a test `product_limit` coupon and try adding a product with a user who has past orders.

---

### 6. Benefits and Added Value

- **For store owners**: Control the quantities of products sold to a single user, preventing abuse or the sale of limited products in large quantities to the same customer.
- **For developers**: A flexible feature that can be customised for any product or unit, leveraging the robust coupon infrastructure without starting from scratch.
- **For end users**: A fair purchasing experience with clear messages when attempting to exceed allowed limits.

---

### 7. Future Development Plans

- Add support for restricting multiple products in a single coupon (instead of a single product).
- Ability to restrict total quantities across multiple products (e.g., “the total number of products from category X should not exceed Y”).
- A reporting interface to monitor customer consumption of product limits.
- Integrate artificial intelligence to suggest optimal limits based on purchasing behaviour.

---

### 8. Conclusion

This update represents an important step towards enabling stores to manage product quantities more intelligently and automatically. By adding `ProductLimit`, it is now possible to implement precise sales policies without the need for code modifications, enhancing the system’s flexibility and power. We look forward to continuing to develop this feature based on your feedback and practical use cases.
