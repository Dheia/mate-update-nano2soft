## 2026-05-06 – 2026-05-08

**Updates for `Nano.Orders` Plugin – Version 2.2.10**  
**Updates for `Nano.OrdersApi` Plugin – Version 1.0.20**

### Summary of Updates

The `Nano.Orders` plugin received a pivotal update in version **2.2.10** focusing on enabling advanced filtering of orders by product owner, developing a professional mechanism for updating order status with graduated permissions and multiple roles, along with improvements in `OrderHelper` for managing users and couriers.  
Meanwhile, version **1.0.20** of `Nano.OrdersApi` provided a unified API endpoint for updating order status, leveraging the new functions in `OrderManager`, with full support for custom options and permissions.

---

## Nano.Orders v2.2.10 – Product Owner Filtering and Graduated Permission Order Status Update

### Release Objectives

- **Provide advanced query scopes** for filtering orders based on the ownership of associated products.
- **Create a comprehensive professional function** to update order status with different roles (admin, owner, courier) with flexible transition rules and customizable permissions.
- **Add helper functions** to extract courier from user and vice versa.
- **Integrate product owner filtering** inside `OrderHelper::getOrdersRecords` for smarter reports.
- **Support external configuration** (`config`) to control transition rules and permissions.

### New Features

#### 1. `HasProductOwnerScopes` Trait in the `Order` Model

The trait was created at path `Nano\Orders\Models\Order\HasProductOwnerScopes` and includes advanced scopes and functions for filtering orders by product owner:

| Scope / Function | Description |
|------------------|-------------|
| `scopeWhereHasProductsByOwner` | Main scope with comprehensive options array: `user`, `count`, `not`, `boolean`, `conditions`, `itemConditions`. |
| `scopeWhereProductOwner` | Quick shortcut (at least one product). |
| `scopeHasProductsByOwner` | Filter by a minimum number of owned products. |
| `scopeDoesntHaveProductsByOwner` | Filter orders containing no owned product. |
| `scopeHasProductsByOwnerCount` | Exact (or compared) count of owned products. |
| `scopeOrWhereHasProductsByOwner` | Link scope with `OR`. |
| `scopeWithProductsByOwnerCount` | Add a computed column counting owned products. |
| `scopeSortByProductsByOwnerCount` | Sort orders by number of owned products. |
| `hasAnyProductByOwner` | Check function on a single order object. |
| `applyOwnerToProductQuery` | Internal helper to apply owner condition to a product query. |

All scopes use table name constants (`Product::TABLE_NAME`) to ensure lasting compatibility.

**Usage Example:**
```php
// Orders containing 2 or more products owned by the current user
$orders = Order::whereHasProductsByOwner([
    'count' => 2,
    'countOperator' => '>='
])->get();
```

#### 2. `updateOrderStatusAdvanced` Function in `StepStatus` Trait

An integrated function was added meeting all requirements for updating order status with graduated permissions:

| Feature | Description |
|---------|-------------|
| **Automatic actor detection** | Uses the currently logged-in user (backend admin, owner, courier). |
| **Three status fields** | `order_states_ref_type` (for management), `user_status` (for owner), `delivery_status` (for courier). |
| **Customizable transition rules** | Default + from settings + role-specific + custom per call. |
| **Prevent modifying completed** | Changing a completed order's status is not allowed (except with admin override). |
| **Automatic sync** | When `user_status` equals `delivery_status`, the main status is updated. |
| **Cancellation reasons** | `because_cancel` (owner) and `delivery_because_cancel` (courier). |
| **Comprehensive control options** | `is_save`, `is_event`, `is_logs`, `skip_permission`, `admin_override`, `custom_message`. |
| **Unified response structure** | Contains `input_data`, `process_data`, `model`, `debug`. |

**Role Examples:**
- **Admin**: Can change any field, with the ability to override transition constraints (`admin_override`).
- **Owner**: Changes `user_status` only, and can cancel the order with a reason.
- **Courier**: Changes `delivery_status`, and can only move `user_status` from `NEW` to `DELIVERY`.

#### 3. Helper Functions in `OrderHelper`

- `getDeliveryByUser($user)` – Extracts the courier object from a user.
- `getUserByDelivery($delivery)` – Extracts the user object from a courier.

#### 4. Product Owner Filtering Support in `OrderHelper::getOrdersRecords`

The following options were added:
- `is_has_products_by_owner` (enable filter)
- `products_by_owner_*` (all options of the main scope)
- `is_has_products_by_owner_or_delivery` to combine the filter with delivery conditions using `OR`.

#### 5. New Settings in `config.php`

```php
'nano.orders::manager.edit_status.allowed_transitions'
'nano.orders::manager.edit_status.admin.allowed_transitions'
'nano.orders::manager.edit_status.user.allowed_transitions'
'nano.orders::manager.edit_status.delivery.allowed_transitions'
```

---

## Nano.OrdersApi v1.0.20 – API Endpoint for Updating Order Status

### Release Objectives

- **Provide a unified API interface** for updating order status by the owner, courier, or admin.
- **Full integration with `updateOrderStatusAdvanced`** to ensure consistent permissions and logic.
- **Support custom options** via request JSON.
- **Maintain a consistent response structure** with the rest of the API.

### New Features

#### 1. Endpoint `POST /api/v1/orders/orders/update-status`

Accepts a JSON request with the following fields (all optional and depend on the role):
- `order_id` or `id`
- `order_states_ref_type`
- `user_status`
- `delivery_status`
- `because_cancel`
- `delivery_because_cancel`
- `is_save`, `is_event`, `is_logs`
- `skip_permission`, `admin_override`
- `custom_message`, `custom_error`

#### 2. `updateStatus` Function in `Orders` Controller

Extracts the order, prepares options, calls `OrderManager::updateOrderStatusAdvanced()` with all parameters, and returns a unified response containing the new status.

**Example successful response:**
```json
{
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "Duplicate order"
    }
}
```

---

## Version Summary (2.2.10 and 1.0.20)

| Version | Key Features |
|---------|--------------|
| **Nano.Orders 2.2.10** | `HasProductOwnerScopes` (product owner filtering scopes), `updateOrderStatusAdvanced` (graduated permission status update), `getDeliveryByUser` / `getUserByDelivery` functions, product owner filtering support in `OrderHelper::getOrdersRecords`, flexible transition rules configuration. |
| **Nano.OrdersApi 1.0.20** | API endpoint `POST orders/update-status`, integration with the professional function, support for all custom options, unified response. |

---

### Upgrade Requirements

1. **Update Files**:
   - Add `HasProductOwnerScopes` trait in `models/Order/HasProductOwnerScopes.php`.
   - Add `StepStatus` trait in `traits/steps/StepStatus.php`.
   - Update `OrderHelper.php` with new functions and product owner filter support.
   - Update `OrderManager.php` to use `StepStatus`.
   - Update `Orders.php` in `OrdersApi` to add `updateStatus` function.
   - Update `routes.php` in `OrdersApi` to add the new route.

2. **No New Migrations**: The two versions require no database changes.

3. **Translation Files**:
   - Add `lang/ar/manager/update_status.php` file in `Nano.Orders` with the required keys.

4. **Optional Settings**:
   - Custom transition rules can be added in `config.php` under `nano.orders::manager.edit_status`.

5. **Compatibility Testing**:
   - Test the new scopes with different queries.
   - Test status updates from the three roles.
   - Test the API from mobile applications (using the appropriate user token).

---

### Conclusion

Versions **Nano.Orders 2.2.10** and **Nano.OrdersApi 1.0.20** represent a qualitative leap in the flexibility of the orders system. On one hand, the `HasProductOwnerScopes` scopes provided unprecedented filtering of orders based on product ownership, meeting the needs of multi-vendor marketplaces. On the other hand, the `updateOrderStatusAdvanced` function and the integrated API endpoint delivered a unified and secure system for updating order status by all parties (admin, owner, courier) with fully customizable settings. The code is clean, documented, and easily extensible.

---

**Reference Documentation**:
- [`OrderManager` and its traits Documentation](./docs/Orders/Classes/Docs-OrderManager-Class-en.md)
- [`StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-en.md)
- [Advanced `StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-en.md)
- [`HasProductOwnerScopes` Scopes Documentation](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-en.md)
- [Orders API Documentation](./docs/OrdersApi/Docs-OrdersApi-en.md)
