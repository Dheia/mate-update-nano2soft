## Explanation of the Function's Workflow Step by Step

The `updateOrderStatusAdvanced` function goes through several sequential stages ensuring security, flexibility, and compatibility with different scenarios. Below is a precise breakdown of each step.

---

### 🔹 Step 1: Initialize Response Structure

A `$result` array is created with the unified structure (`code`, `status`, `message`, `error`, `errors`, `model`, `data`) along with default message variables and `input_data` and `process_data` arrays.

---

### 🔹 Step 2: Prepare Settings (Hierarchical Merge)

Settings are read from multiple levels, with higher priority given to the top:

1. **Default values** (hardcoded within the function).
2. **General settings** from `config('nano.orders::manager.edit_status')` like `is_save`, `is_event`, `allowed_transitions`.
3. **Direct call options** passed by the developer in the `$options` array.

A helper `$mergeConfig` is used to compare values in order: direct options ← general settings ← default value.

All internal variables (like `$actor`, `$order`, `$newOrderState`, `$isSave`, ...) are extracted from options or settings or defaults.

---

### 🔹 Step 3: Identify the Actor

If the `actor` user object is not explicitly passed, the function searches for a currently logged-in user in the following order:
1. `\Nano\AuthApi\Classes\AuthHelpers::getCurrentUser()`
2. `\BackendAuth::getUser()`
3. `\Auth::getUser()`

This ensures the function works automatically without needing to pass the user every time.

---

### 🔹 Step 4: Role Classification

After identifying the actor, they are classified into one of the roles:

- **Admin**: if the object is of type `Backend\Models\User`.
- **Owner**: if it is `RainLab\User\Models\User` and its ID equals `order->user_id`.
- **Courier**: if it is `RainLab\User\Models\User` and its ID equals `order->delivery_user_id`.
- If the user is both owner and courier, they are treated as **Owner** only.
- If the `skip_permission` option is enabled, the actor is treated as an admin with full permissions.

---

### 🔹 Step 5: Prevent Modifying Completed Orders

If the order's main status is `COMPLETE`, any change is rejected unless the actor is an **Admin** with `admin_override` enabled.

---

### 🔹 Step 6: Build Allowed Transition Rules (Hierarchical)

The final allowed transitions array is built by merging the following layers:

1. **Basic rules** (Hardcoded):
   ```
   CART       → [NEW, CANCELLED]
   NEW        → [PROCESSING, DELIVERY, CANCELLED]
   PROCESSING → [DELIVERY, CANCELLED]
   DELIVERY   → [COMPLETE, CANCELLED]
   ```

2. **General settings**: key `nano.orders::manager.edit_status.allowed_transitions` in the `config` file.

3. **Role-specific settings**: key `nano.orders::manager.edit_status.{role}.allowed_transitions` (where `role` is `admin`, `user`, or `delivery`).

4. **Custom callable**: if the option `role_allowed_transitions` exists and is callable, it is invoked with `(role, currentTransitions)`, and the result is merged.

5. **Direct options**: key `allowed_transitions` in `$options`.

Merging uses `array_replace_recursive` to preserve the multi-level structure.

---

### 🔹 Step 7: Transition Validation Closure

An internal closure `$isValidTransition` is created that:
- If the new value is empty → allows (no change).
- If actor is admin and `admin_override` is on → always allows.
- Otherwise, checks if the new status exists within the allowed values for the current status.

---

### 🔹 Step 8: Apply Changes by Role

The role is inspected, and its specific logic is executed:

#### A. Admin
- Can change the main status `order_states_ref_type`.
- If the new status is `COMPLETE` or `CANCELLED`, it is immediately copied to `user_status` and `delivery_status`.
- If no main status is passed, can change `user_status` or `delivery_status` separately.
- Can record cancellation reasons (`because_cancel` and `delivery_because_cancel`).

#### B. Owner
- **Forbidden** to change the main status or `delivery_status`.
- Can only change `user_status`.
- If they cancel (`CANCELLED`) and provide a reason, it records in `because_cancel`.
- After modification, if `user_status` matches `delivery_status`, the main status is updated automatically.

#### C. Courier
- **Forbidden** to change the main status.
- Can change `delivery_status`.
- Can change `user_status` **only** from `NEW` to `DELIVERY` (start delivery).
- If canceling (`CANCELLED`) and provides a reason, records in `delivery_because_cancel`.
- After modification, if `user_status` and `delivery_status` are equal, the main status is updated automatically.

#### D. Other Roles
- Any actor not matching the above roles is rejected.

---

### 🔹 Step 9: Save, Logs, and Events

If `is_save` is on and an actual change occurred in the order:

1. **Auto-update dates**:
   - If `order_states_ref_type` changes to `DELIVERY` → `shipped_at` and `delivery_at` are set if not already.
   - If it becomes `COMPLETE` or `CANCELLED` → `end_date` is set.

2. **Save the order** within a database transaction to ensure integrity.

3. **Log status history** (if `is_logs` is on and the class `Nano2\StatusHistory\Classes\Manager` exists):
   - Each of the three status fields (`order_states_ref_type`, `user_status`, `delivery_status`) change is logged separately, linked to the actor.

4. **Fire events** (if `is_event` is on):
   - `nano.orders.newOrderStatus` when main status changes.
   - `nano.orders.newUserOrderStatus` when `user_status` changes.
   - `nano.orders.newDeliveryOrderStatus` when `delivery_status` changes.

---

### 🔹 Step 10: Prepare Output

`$output_process_data` is filled with the final order status and placed in `$result['data']` and `$result['model']`. `$output_input_data` is placed in `$result['input_data']`.

---

### 🔹 Step 11: Error Handling

Any exception is caught in the `catch (\Throwable $e)` block:

- The transaction is rolled back if active.
- Fields `code`, `status`, `message`, `error` are populated.
- In debug environment (`debug = true`), trace details (`debug`) are added.

---

### 🔹 Step 12: Final Return

The function returns the `$result` array regardless of the outcome.

**Reference Documentation**:
- [`OrderManager` and its Traits Documentation](./docs/Orders/Classes/Docs-OrderManager-Class-en.md)
- [`StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-en.md)
- [Advanced `StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-en.md)
