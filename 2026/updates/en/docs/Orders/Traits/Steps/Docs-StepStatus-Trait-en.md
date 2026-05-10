# Documentation for `updateOrderStatusAdvanced` Function

**Class:** `Nano\Orders\Traits\Steps\StepStatus`  
**Plugin:** `Nano.Orders`  
**Version:** 2.2.10  

---

## 📋 Introduction

The `updateOrderStatusAdvanced` function is the cornerstone of order status management within the Nano.soft platform. It was designed to provide a unified and secure mechanism allowing the three parties (system admin, order owner, and order courier) to update the order status according to graduated permissions and strict logical constraints. The function relies on a hierarchical configuration system (default → general settings → role-specific settings → direct call options) giving developers maximum flexibility in customizing its behavior without compromising security.

---

## 📌 Function Signature

```php
public function updateOrderStatusAdvanced(array $options = []): array
```

### Parameters

| # | Parameter | Type | Required | Description |
|---|-----------|------|----------|-------------|
| 1 | `$options` | `array` | No | Options array controlling the function's behavior (see table below). |

---

## ⚙️ Complete Options

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `actor` | `User\|BackendUser\|null` | `null` | The user performing the update. If not passed, the function tries to fetch from the current session (`AuthHelpers` ← `BackendAuth` ← `Auth`). |
| `order` | `Order\|null` | `null` | The order model to be modified. If not passed, the current order model from `OrderManager` is used. |
| `order_states_ref_type` | `string\|null` | `null` | New main status (e.g., `COMPLETE`, `CANCELLED`). **For admin only**. |
| `user_status` | `string\|null` | `null` | Order status from the client/owner side. |
| `delivery_status` | `string\|null` | `null` | Order status from the courier side. |
| `because_cancel` | `string\|null` | `null` | Cancellation reason provided by the owner. |
| `delivery_because_cancel` | `string\|null` | `null` | Cancellation reason provided by the courier. |
| `is_save` | `bool` | `true` | Save changes to the database? (`false` useful for testing). |
| `is_event` | `bool` | `true` | Fire system events (`Event::fire`) such as `nano.orders.newOrderStatus`? |
| `is_logs` | `bool` | `true` | Log status history via `Nano2\StatusHistory` (if present)? |
| `skip_permission` | `bool` | `false` | Bypass all permission and role checks (gives full admin privileges). |
| `admin_override` | `bool` | `false` | Allows admin to override transition constraints (e.g., revert `DELIVERY` back to `PROCESSING`). |
| `custom_message` | `string\|null` | `null` | Custom success message used instead of the default translation message. |
| `custom_error` | `string\|null` | `null` | Custom default error message used on failure. |
| `allowed_transitions` | `array\|null` | `null` | Custom transition rules for this call only. Merged on top of all other settings. |
| `role_allowed_transitions` | `callable\|null` | `null` | A callback to generate custom transition rules based on role. Receives `($role, $currentTransitions)` and returns a transitions array. |

---

## 🧰 Function Workflow (Step by Step)

### 1. Merge Default Settings
The function merges default values (shown above) with any values from `config` files under the path `nano.orders::manager.edit_status`, then finally the direct options passed in `$options` (direct options have the highest priority).

### 2. Identify the Actor
If `actor` not explicitly passed, the function searches for a logged-in user:
1. `AuthHelpers::getCurrentUser()`
2. `BackendAuth::getUser()`
3. `Auth::getUser()`

### 3. Role Classification
- **Admin** (`admin`): Object of `Backend\Models\User`.
- **Owner** (`user`): Frontend user (`RainLab\User\Models\User`) whose ID matches `order->user_id`.
- **Courier** (`delivery`): Frontend user whose ID matches `order->delivery_user_id`.
- If `skip_permission = true`, everyone is treated as admin.

### 4. Prevent Modifying Completed Orders
If `order_states_ref_type = COMPLETE`, any changes are rejected unless the actor is an admin with `admin_override = true`.

### 5. Build Transition Rules (Hierarchical)
1. **Basic rules** (hardcoded in code):
   ```
   CART       → [NEW, CANCELLED]
   NEW        → [PROCESSING, DELIVERY, CANCELLED]
   PROCESSING → [DELIVERY, CANCELLED]
   DELIVERY   → [COMPLETE, CANCELLED]
   ```
2. Merged with `config('nano.orders::manager.edit_status.allowed_transitions')`.
3. Merged with role-specific settings from `config("nano.orders::manager.edit_status.{admin|user|delivery}.allowed_transitions")`.
4. If `role_allowed_transitions` (callable) exists, it is invoked and merged.
5. Finally merged with `allowed_transitions` from direct call options.

### 6. Apply Changes by Role

#### Admin (`admin`)
- Can change `order_states_ref_type` (complying with transition rules or overriding if `admin_override`).
- Can change `user_status` and `delivery_status` separately.
- Can record cancellation reasons (`because_cancel`, `delivery_because_cancel`).
- When setting the main status to `COMPLETE` or `CANCELLED`, it automatically copies to `user_status` and `delivery_status`.

#### Owner (`user`)
- **Cannot** change `order_states_ref_type` or `delivery_status`.
- Can change `user_status` only (with transition rules).
- If choosing `CANCELLED`, can record `because_cancel`.
- If `user_status` equals `delivery_status` after the change, `order_states_ref_type` is automatically updated.

#### Courier (`delivery`)
- **Cannot** change `order_states_ref_type`.
- Can change `delivery_status` (with transition rules).
- Can change `user_status` **only** from `NEW` to `DELIVERY` (start delivery).
- If choosing `CANCELLED` for `delivery_status`, can record `delivery_because_cancel`.
- Automatic synchronization also applies.

### 7. Save, Events, and Logs
If `is_save = true`:
- Dates are auto-updated (`shipped_at`, `delivery_at`, `end_date`).
- Order is saved within a `DB::transaction`.
- If `is_logs = true`, status changes are logged in `StatusHistory`.
- If `is_event = true`, events are fired:
  - `nano.orders.newOrderStatus`
  - `nano.orders.newUserOrderStatus`
  - `nano.orders.newDeliveryOrderStatus`

---

## 📦 Response Structure

Success:
```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "error": null,
    "errors": null,
    "model": "<Order object>",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "Duplicate order",
        "delivery_because_cancel": null
    },
    "input_data": { ... },
    "process_data": { ... }
}
```

Failure:
```json
{
    "code": 400,
    "status": false,
    "message": "Failed to update order status.",
    "error": "Cannot transition from COMPLETE to CANCELLED.",
    "errors": null,
    "model": null,
    "data": null
}
```

---

## 🧪 Practical Examples

### Example 1: Admin cancels an order (with override)
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'                 => $order,
    'actor'                 => $adminUser,
    'order_states_ref_type' => 'CANCELLED',
    'because_cancel'        => 'Expired',
    'admin_override'        => true,  // allows transition even from COMPLETE
]);
```

### Example 2: Order owner cancels their order
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'         => $order,
    'user_status'   => 'CANCELLED',
    'because_cancel'=> 'Changed my mind',
]);
// If delivery_status is also CANCELLED, order_states_ref_type becomes CANCELLED automatically
```

### Example 3: Courier starts delivery
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'        => $order,
    'user_status'  => 'DELIVERY',     // from NEW -> DELIVERY only
    'actor'        => $courierUser,
]);
```

### Example 4: Courier completes delivery
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'            => $order,
    'delivery_status'  => 'COMPLETE',
    'actor'            => $courierUser,
]);
// If user_status is also COMPLETE, order_states_ref_type becomes COMPLETE
```

### Example 5: Testing without saving (trial)
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'        => $order,
    'user_status'  => 'CANCELLED',
    'is_save'      => false,   // won't save to database
]);
echo $result['data']['user_status']; // "CANCELLED"
```

### Example 6: Customizing transition rules per role via callable
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'                    => $order,
    'user_status'              => 'ON_HOLD',
    'role_allowed_transitions' => function ($role, $current) {
        if ($role === 'user') {
            $current['NEW'][] = 'ON_HOLD';
        }
        return $current;
    },
]);
```

### Example 7: Disable events and logs (silent operation)
```php
$result = $orderManager->updateOrderStatusAdvanced([
    'order'    => $order,
    'user_status' => 'CANCELLED',
    'is_event' => false,
    'is_logs'  => false,
]);
```

---

## 🌐 Using the Function via API

### Endpoint
```
POST /api/v1/orders/orders/update-status
```

### Permissions
- **Order owner** (`user_id`) – can update `user_status` only.
- **Order courier** (`delivery_user_id`) – can update `delivery_status` and start delivery (`user_status` from `NEW` to `DELIVERY`).
- **Dashboard user** (`Backend\Models\User`) – full permissions.

### Request Example (Owner cancels)
```bash
curl -X POST https://yourdomain.com/api/v1/orders/orders/update-status \
  -H "Authorization: Bearer {user_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 125,
    "user_status": "CANCELLED",
    "because_cancel": "Wrong order"
  }'
```

### Request Example (Courier completes delivery)
```bash
curl -X POST https://yourdomain.com/api/v1/orders/orders/update-status \
  -H "Authorization: Bearer {courier_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 125,
    "delivery_status": "COMPLETE"
  }'
```

### Request Example (Admin cancels with override)
```bash
curl -X POST https://yourdomain.com/api/v1/orders/orders/update-status \
  -H "Authorization: Bearer {admin_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": 125,
    "order_states_ref_type": "CANCELLED",
    "admin_override": true,
    "because_cancel": "Violates terms"
  }'
```

### Example Response
```json
{
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "Violates terms",
        "delivery_because_cancel": null
    }
}
```

---

## ❗ Error Handling

| Error | Cause |
|-------|-------|
| `Cannot modify a completed order.` | `order_states_ref_type` equals `COMPLETE` and no `admin_override`. |
| `Cannot transition from X to Y.` | Transition not found in the allowed transition rules. |
| `You do not have permission to change the main order status.` | Owner or courier attempting to change `order_states_ref_type`. |
| `You do not have permission to change the courier status.` | Owner attempting to change `delivery_status`. |
| `You can only change customer status from NEW to DELIVERY.` | Courier trying to change `user_status` to anything other than those two. |
| `You do not have permission to modify this order.` | User not linked to the order and not an admin. |

---

## 🛠️ Customization via Config

Transition rules and permissions per role can be customized in the `config.php` file:

```php
'nano.orders::manager' => [
    'edit_status' => [
        // General transitions (merge with basic rules)
        'allowed_transitions' => [
            'PROCESSING' => ['DELIVERY', 'CANCELLED', 'ON_HOLD'],
        ],

        // Admin-specific transitions
        'admin' => [
            'allowed_transitions' => [
                'DELIVERY' => ['COMPLETE', 'CANCELLED', 'PROCESSING'], // allow rollback
            ],
        ],

        // Owner-specific transitions
        'user' => [
            'allowed_transitions' => [
                'NEW' => ['CANCELLED'],
            ],
        ],

        // Courier-specific transitions
        'delivery' => [
            'allowed_transitions' => [
                'NEW' => ['DELIVERY'],
            ],
        ],
    ],
],
```

---

## ✅ Best Practices

- Use `is_save = false` during testing.
- Pass `actor` explicitly in background operations (Cron, Jobs) to avoid relying on the session.
- Utilize `role_allowed_transitions` to add custom statuses without modifying core code.
- Ensure translation keys (`nano.orders::lang.manager.update_status.*`) have clear messages.
- Use `admin_override` cautiously, and prefer recording a reason in `admin_notes`.

---

## 🔚 Conclusion

The `updateOrderStatusAdvanced` function covers all needs related to updating order status in a multi-party system. Thanks to its hierarchical configuration design and wide range of options, it can be adapted to any business scenario while maintaining high security and efficiency. Integration with the API via the unified endpoint makes it ready for immediate use in mobile applications and dashboards.

**Reference Documentation**:
- [`OrderManager` and its Traits Documentation](./docs/Orders/Classes/Docs-OrderManager-Class-en.md)
- [`StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-en.md)
- [Advanced `StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-en.md)