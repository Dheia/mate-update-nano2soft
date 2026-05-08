## 2026-05-02 – 2026-05-08

**Nano.Coupons Plugin Update – Version 2.1.1 – Comprehensive Improvements to the "Product Limit" Condition**

---

### Update Summary

Version **2.1.1** introduces fundamental enhancements to the "Product Limit" feature that was launched in version 2.1.0. The validation logic has been completely restructured through the addition of a new, independent class `ProductLimitValidator`, providing unprecedented flexibility in handling product constraints. The update includes support for advanced scenarios such as limiting the number of orders for specific order types (regardless of product) and detailed multilingual error messages that show when the user can reorder. Significant performance improvements have also been made through caching and preventing redundant database queries.

---

### 1. Release Objectives

- **Separate validation logic** from the cart condition into an independent `ProductLimitValidator` class to facilitate reuse and maintenance.
- **Support new scenarios**: Limiting the number of allowed orders for a specific order type during a period, without the need to restrict a specific product.
- **Improve user experience** through clear and detailed error messages that include the maximum limit, current usage, period, and the date of the next possible order.
- **Significant performance improvements** through caching of constraints (`Cache`) and preventing repetitive expensive database queries.
- **Expand the admin interface** to include product search (`recordfinder`) and support selecting the unit for a specified product.
- **Add new scopes** `scopeOnProductLimit` and `scopeNotOnProductLimit` to the coupon model.

---

### 2. Developed Components

#### 2.1 `ProductLimitValidator` Class (New)

Location: `plugins/nano/coupons/classes/ProductLimitValidator.php`

An independent class specialized in checking product limit constraints. It accepts a user and configuration options (`use_cache`, `cache_ttl`) and provides comprehensive public functions:

| Function | Description |
|----------|-------------|
| `checkOrFail(...)` | Comprehensive check that throws an exception with a detailed error message when limits are exceeded. |
| `check(...)` | Silent check returning `true/false`. |
| `getLimits(...)` | Fetch applicable constraints with multiple strategies (`auto`, `cache`, `fresh_db`, `direct_query`). |
| `getLimitsForProduct(...)` | Shortcut to fetch constraints for a specific product. |
| `checkOrderTypeLimitOrFail(...)` | Check the number of orders for a specific order type. |
| `getOrderTypeLimits(...)` | Fetch constraints for a specific order type. |
| `clearCache()` | Manually clear cache. |
| `setUser(User $user)` | Set the user. |

**Supported Fetch Strategies:**
- `auto`: Automatically uses cache if enabled.
- `cache`: Force loading constraints from cache.
- `fresh_db`: Ignore cache and load directly from the database.
- `direct_query`: Execute a direct query on the coupons table with specified conditions.

**Key Logic Improvements:**

- **First Case (Specific Product):**
  - If `max_quantity > 0`, the total quantity (cart + historical + newly added) is checked.
  - If `max_orders > 0`, the number of orders containing this product (or its unit) is checked.
  - Respects `units_id`, `order_restriction`, `shop_restriction`, and `period_days`.

- **Second Case (No Product – Order Count Limit Only):**
  - If `product_id` is empty, the `max_orders` check is applied based on `order_types`, `shop_ids`, and `period_days`.

#### 2.2 Updates to the `ProductLimit` Cart Condition

Location: `plugins/nano/coupons/cartconditions/ProductLimit.php`

- The class has been simplified to act as a mediator between the cart system (`CartCondition`) and `ProductLimitValidator`.
- The `validateCartItem` function now uses `ProductLimitValidator` directly.
- The `beforeApply` has been temporarily disabled to prevent any additional load during cart total calculation.
- Reliance on cache via `Cache::remember` in `loadAllProductLimits` with `is_expired` stored to avoid serialization issues.

#### 2.3 Updates to the `Coupon` and `Coupons_model` Models

- Added cast properties `'product_id' => 'integer'`, `'units_id' => 'integer'`, `'max_quantity' => 'integer'`, `'max_orders' => 'integer'`, `'period_days' => 'integer'`.
- Added `getProductIdOptions` function to list active products.
- Expanded `getUnitsIdOptions` to dynamically fetch units of a specific product.
- Added scopes `scopeOnProductLimit` and `scopeNotOnProductLimit` to filter coupons.
- In `afterSave`, the cache `nano.coupons.product_limits` is cleared when a `product_limit` coupon is modified/added.

#### 2.4 Admin Interface Updates (`fields.yaml`)

- Changed `product_id` from `type: dropdown` to `type: recordfinder` for advanced product search.
- Added translation support for `placeholder`, `emptyOption`, `title`, `prompt` properties.
- Updated `units_id` to work with `dependsOn: [product_id]` to dynamically update the unit list.
- Show fields `product_id`, `units_id`, `max_quantity`, `max_orders`, `period_days` only when `apply_coupon_on = product_limit` is selected via `trigger`.

#### 2.5 Translation Files (`lang/ar/lang.php`)

- New keys for product limit messages with parameters:
  - `max_quantity_exceeded` with `:max` and `:count`.
  - `max_orders_exceeded` with `:max`, `:count`, `:days`, `:next_date`.
  - `unlimited` and new field label items.

#### 2.6 Configuration File (`config.php`)

- Added key `is_support_product_limit` with default value `false` (to control showing the type in lists).

#### 2.7 Updates to `Plugin.php` (Nano.Coupons)

- Registered the `ProductLimit` condition in `registerCartConditions` only when the class exists.
- Modified `bindCouponsEvent` to add `notOnProductLimit()` to avoid applying automatic coupons to a cart that has a product limit.

---

### 3. Workflow (New Flow)

1. **Setup (Once):** The admin creates a `product_limit` coupon, selecting either a specific product (and optional unit) or leaving it empty to restrict the number of orders by order type.
2. **Loading Constraints:** When the cart loads, constraints are loaded from cache (or database) once.
3. **Immediate Check on Product Addition:** `ProductLimit` calls `ProductLimitValidator::checkOrFail` using the `nano.cart.adding` event.
4. **Quantity / Order Count Check:**
   - For product: total quantity + new quantity ≤ `max_quantity`.
   - For product or order type: previous order count < `max_orders`.
5. **Detailed Error Message:** "You have exceeded the allowed number of orders (maximum 3). Current order count: 3. Period: 7 days. You can order again after 2026-05-10."

---

### 4. Key Enhancements and Achievements

- **Excellent Performance:** Using `Cache` for constraints with automatic clearing when the coupon is modified.
- **Bulk Queries:** `getBulkHistoricalQuantities` and `getBulkHistoricalOrderCounts` aggregate data in a single query.
- **High Flexibility:**
  - Check a specific product, a specific order type, or both.
  - Check quantity, number of orders, or both.
  - Optionally specify `unit_id`.
- **Professional Error Messages:** Display the maximum limit, current usage, period, and the date when reordering is allowed.
- **Full Translation Support** (Arabic/English).
- **Separation of Concerns:** `ProductLimitValidator` is reusable anywhere (API, CLI, etc.) without depending on the cart system.

---

### 5. Requirements and Upgrade

- **File Updates:**
  - `plugins/nano/coupons/classes/ProductLimitValidator.php` (new)
  - `plugins/nano/coupons/cartconditions/ProductLimit.php`
  - `plugins/nano/coupons/models/Coupon.php`
  - `plugins/nano/coupons/models/Coupons_model.php`
  - `plugins/nano/coupons/models/coupon/fields.yaml`
  - `plugins/nano/coupons/lang/ar/lang.php`
  - `plugins/nano/coupons/Plugin.php`
  - `plugins/nano/coupons/config/config.php`
- **Run Migration:** No new migrations in this version (columns were added in 2.1.0).
- **Clear Cache:** It is recommended to run `php artisan cache:clear` after upgrading to ensure loading of new translation files.

---

### 6. Additional Notes

- The `product_limit` feature can be enabled/disabled from the `.env` file via the `NANO_COUPONS_IS_SUPPORT_PRODUCT_LIMIT` key.
- For better performance with a large number of coupons, adjust `cache_ttl` to an appropriate duration (default 300 seconds).
- All queries are safe against SQL Injection and use parameter binding.

---

### 7. Future Development Plans

- Support specifying multiple products in a single `product_limit` coupon.
- Develop a reporting interface to track customer consumption of product limits.
- Add the ability to disable the constraint for specific customer groups.

---

### 8. Conclusion

Version 2.1.1 completes the "Product Limit" system and makes it more powerful, flexible, and professional. Thanks to the separate `ProductLimitValidator`, developers can easily integrate limit checks anywhere, while performance improvements ensure a smooth end-user experience even with a large number of active constraints.

**Reference Documentation**:

- [`ProductLimitValidator` Class Documentation](./docs/Coupons/Classes/Docs-ProductLimitValidator-Class-en.md)
- [`ProductLimit` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-ProductLimit-CartCondition-en.md)
- [`Coupon` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-Coupon-CartCondition-en.md)
- [`AutoCoupon` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-AutoCoupon-CartCondition-en.md)
- [`AutoCouponShipping` Cart Condition Documentation](./docs/Coupons/CartConditions/Docs-AutoCouponShipping-CartCondition-en.md)
