# Update Order Status – API Reference

Change the order status with graduated permissions (owner, delivery person, or system administrator). This endpoint relies on the professional function `updateOrderStatusAdvanced` inside `OrderManager`.

---

## Endpoint

```
POST /api/v1/orders/orders/update-status
```

---

## Permissions & Roles

| Role | Allowed Fields | Notes |
|------|----------------|-------|
| **System Administrator** (`Backend\Models\User`) | `order_states_ref_type`, `user_status`, `delivery_status`, `because_cancel`, `delivery_because_cancel` | Can bypass transition rules using `admin_override: true`. When setting the main status to `COMPLETE` or `CANCELLED`, it is automatically copied to `user_status` and `delivery_status`. |
| **Order Owner** (`user_id`) | `user_status`, `because_cancel` | Cannot change `order_states_ref_type` or `delivery_status`. If `user_status` equals `delivery_status` after the update, the main status is automatically updated. |
| **Delivery Person** (`delivery_user_id`) | `delivery_status`, `delivery_because_cancel`, and `user_status` only from `NEW` → `DELIVERY` | The courier can start delivery by changing the customer status from `NEW` to `DELIVERY`. The same automatic synchronisation applies here. |

> If no user is passed, the endpoint uses the currently logged‑in user (admin, frontend user, or courier).

---

## Request Body (JSON)

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `id` or `order_id` | `integer` | **Yes** | Order ID |
| `order_states_ref_type` | `string` | No | New main status: `NEW`, `PROCESSING`, `DELIVERY`, `CANCELLED`, `COMPLETE` |
| `user_status` | `string` | No | Order status from the customer side |
| `delivery_status` | `string` | No | Order status from the courier side |
| `because_cancel` | `string` | No | Cancellation reason from the owner |
| `delivery_because_cancel` | `string` | No | Cancellation reason from the courier |
| `is_save` | `boolean` | No | Save changes to the database (default `true`). Set to `false` to test without saving. |
| `is_event` | `boolean` | No | Fire events such as `nano.orders.newOrderStatus` (default `true`). |
| `is_logs` | `boolean` | No | Log status history in `StatusHistory` (default `true`). |
| `skip_permission` | `boolean` | No | Bypass all permission checks (treats the user as admin). |
| `admin_override` | `boolean` | No | For admin only: override status transition constraints (e.g., revert status). |
| `custom_message` | `string` | No | Custom success message. |
| `custom_error` | `string` | No | Custom error message. |
| `allowed_transitions` | `object` | No | Custom transition rules for this order, merged on top of the general rules. |
| `role_allowed_transitions` | `callable` (only via code) | No | Function to generate transition rules based on role. |

---

## Response Structure

### Success (200)

```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "error": null,
    "errors": null,
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

### Error (400 / 401)

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to update order status.",
    "error": "Cannot transition from COMPLETE to NEW.",
    "errors": null,
    "data": [],
    "input_data": { ... },
    "process_data": []
}
```

---

## Examples

### 1. Admin cancels an order with override

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <admin_token>
Content-Type: application/json

{
    "order_id": 125,
    "order_states_ref_type": "CANCELLED",
    "because_cancel": "Violates terms",
    "admin_override": true
}
```

#### Response

```html
Status: 200 OK
```

```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "error": null,
    "errors": null,
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

### 2. Order owner cancels his own order

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <user_token>
Content-Type: application/json

{
    "order_id": 125,
    "user_status": "CANCELLED",
    "because_cancel": "Changed my mind"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "CANCELLED",
        "delivery_status": "CANCELLED",
        "because_cancel": "Changed my mind",
        "delivery_because_cancel": null
    }
}
```

> The main status was automatically updated because `user_status` became equal to `delivery_status` after the change.

---

### 3. Delivery person starts delivery

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "user_status": "DELIVERY"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "DELIVERY",
        "user_status": "DELIVERY",
        "delivery_status": "COMPLETE",
        "because_cancel": null,
        "delivery_because_cancel": null
    }
}
```

---

### 4. Delivery person completes delivery

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "COMPLETE"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "COMPLETE",
        "user_status": "COMPLETE",
        "delivery_status": "COMPLETE",
        "because_cancel": null,
        "delivery_because_cancel": null
    }
}
```

---

### 5. Unauthorised attempt (owner tries to change courier status)

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <user_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "COMPLETE"
}
```

#### Response

```html
Status: 400 Bad Request
```

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to update order status.",
    "error": "You are not allowed to change the courier status.",
    "errors": null,
    "data": []
}
```

---

### 6. Illegal transition (courier tries to revert status)

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "NEW"
}
```

#### Response

```json
{
    "code": 400,
    "status": false,
    "message": "Failed to update order status.",
    "error": "Cannot transition from DELIVERY to NEW.",
    "errors": null,
    "data": []
}
```

---

### 7. Cancellation with reason from courier

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <courier_token>
Content-Type: application/json

{
    "order_id": 125,
    "delivery_status": "CANCELLED",
    "delivery_because_cancel": "Customer not responding"
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "CANCELLED",
        "user_status": "NEW",
        "delivery_status": "CANCELLED",
        "because_cancel": null,
        "delivery_because_cancel": "Customer not responding"
    }
}
```

---

### 8. Dry‑run (without saving)

```bash
POST /api/v1/orders/orders/update-status
Authorization: Bearer <admin_token>
Content-Type: application/json

{
    "order_id": 125,
    "order_states_ref_type": "COMPLETE",
    "is_save": false
}
```

#### Response

```json
{
    "code": 200,
    "status": true,
    "message": "Order status updated successfully.",
    "data": {
        "order_id": 125,
        "order_states_ref_type": "COMPLETE",
        "user_status": "COMPLETE",
        "delivery_status": "COMPLETE",
        "because_cancel": null,
        "delivery_because_cancel": null
    }
}
```

> The changes were not actually saved to the database.

---

### 9. User not logged in

```bash
POST /api/v1/orders/orders/update-status
Content-Type: application/json

{
    "order_id": 125,
    "user_status": "CANCELLED"
}
```

#### Response

```html
Status: 401 Unauthorized
```

```json
{
    "code": 401,
    "status": false,
    "message": "Failed to update order status.",
    "error": "User Not found",
    "errors": null,
    "data": []
}
```

---

## Important Notes

- All text fields for statuses (`order_states_ref_type`, `user_status`, `delivery_status`) accept the values shown above.
- Default transition rules prevent going backwards (e.g., from `DELIVERY` to `NEW`), but an admin can override them using `admin_override: true`.
- When `is_save: false` is enabled, you can use the endpoint to test the logic without actual effect.
- Events (`nano.orders.newOrderStatus` etc.) are fired automatically unless disabled with `is_event: false`.
- History logging (`StatusHistory`) is performed by default and can be disabled with `is_logs: false`.
- Transition rules can be customised via the configuration `nano.orders::manager.edit_status.allowed_transitions` as well as by role.
- The response structure includes `input_data` (received input) and `process_data` (post‑processing data) to facilitate tracking.