# Comprehensive Documentation for `OrderManager` Class

**Package:** `Nano.Orders`  
**Class:** `Nano\Orders\Classes\OrderManager`  
**Pattern:** Singleton  
**Role:** Manages the order process (Checkout) – coordinates all steps required to create an order, from cart to payment, passing through address, shipping, coupons, images, and more.

---

## 📋 Introduction

`OrderManager` is the orchestrating brain for the order submission process on the Nano.soft platform. The class uses the `Singleton` pattern to ensure only one instance exists during the order lifecycle, and employs a steps system via a set of specialized `Traits`. Each `Trait` covers a specific aspect of the order process:

- Cart and items management.
- Customer and user data.
- Store, location, and date.
- Shipping address and from address.
- Load and vehicle type.
- Shipping methods and expected costs.
- Coupons and tips.
- Payment and payment gateway.
- Images and attachments.

The class supports multiple order types (delivery, pickup, walk...) through `OrdersType`, providing a unified interface for validating each step (`validate`) and executing it (`set`).

---

## 🧩 Complete Function Table

The table is divided into two sections: functions directly in `OrderManager`, and functions from its connected traits. The source trait is mentioned beside each function (if applicable).

### 1. Core Methods – Inside OrderManager Directly

| Function | Description |
|----------|-------------|
| `init()` | Initialize manager: determine order type, load user, cart, store. |
| `instanceType($instance, $user)` | Set current order type (delivery, collection...) and link it to user. |
| `currentInstanceType()` | Return current order type. |
| `setOrderModel($order)` / `getOrderModel()` | Set or get current order model. |
| `getOrderModelDb($user, $instance)` | Fetch order model from database. |
| `setOrdersTypeModel($ref_type, $model)` / `getOrdersTypeModel()` | Set or get order type settings model. |
| `setCart($cart)` / `getCart()` | Set or get cart object. |
| `setCartManager($cartManager)` / `getCartManager()` | Set or get cart manager. |
| `getUser()` / `setUser($user)` / `getUserId()` | Manage logged-in user. |
| `setCustomer($customer)` / `getCustomerId()` | Set customer and get their ID. |
| `getOrder()` / `loadOrder()` | Load current order (from session). |
| `getOrderByHash($hash, $customer)` | Get order by hash. |
| `getDefaultPayment()` / `getDefaultPaymentMethod($order)` | Get default payment method. |
| `getListPaymentMethod($order)` | Available payment methods list. |
| `processPaymentMethod(&$order, $payment_method, &$data)` | Process linking payment method to order. |
| `getPayment($code)` | Get payment method by code. |
| `getPaymentGateways()` | All active payment methods. |
| `findDeliveryAddress($addressId)` | Search for a delivery address. |
| `validateCustomer($customer)` | Validate customer eligibility. |
| `validateDeliveryAddress(array $address)` | Check delivery area coverage for address. |
| `saveOrder($order, array $data)` | Save order and add cart items and totals. |
| `processPayment($order, array $data)` | Execute payment via gateway. |
| `applyDepartmentsAttributes($order, $options)` | Apply store data to order. |
| `applyRequiredAttributes($order, $options)` | Fill mandatory attributes (user, customer, date...). |
| `applyCurrentPaymentFee($code)` | Apply payment method fee to cart. |
| `byUser(User $user, $instance)` | Load order model for a specific user. |
| `validateLocation()` / `validateDefaultLocationOpened()` / `validateEditLocationOpened()` / `validateOrderLocationOpened()` / `validateOrderTime()` | Series of checks for store/app readiness and working hours. |
| `getOrderTypeInRequest($defaultValue)` | Extract `order_type` from request (Header or Body). |
| `getOrderTypeInRequestHeader($defaultValue)` / `getOrderTypeInRequestBody($defaultValue)` | Extract order type from header or body. |
| `cleanValueNumber($value)` | Remove arithmetic symbols from a numeric string. |
| `getExceptRules($is_force)` / `exceptRules($data, $array_except)` | Manage exception rules (to remove non-required fields). |
| `clearOrder()` | Clear order from session and end checkout. |
| `setCurrentOrderId($orderId)` / `getCurrentOrderId()` / `clearCurrentOrderId()` / `isCurrentOrderId($orderId)` | Manage current order ID in session. |
| `setCurrentPaymentCode($code)` / `getCurrentPaymentCode()` | Manage selected payment method code. |
| `mackPaymentGateway()` | Create payment gateway object. |
| `getPaymentGateway($payment_method, $data)` | Create payment gateway with data. |
| `getPaymentGatewayOrder($order, $payment_method, $data)` | Payment gateway linked to order. |
| `getPaymentService($gateway, $order, $url, $data)` | Create payment execution service. |

### 2. Step Functions (Traits) – Summarized by Group

#### Cart and Items (`StepCart`)
| Function | Description |
|----------|-------------|
| `hasRequiredCart()` | Does the order type require a shopping cart? |
| `hasMultiShopInCart($is_cart)` | Allow products from multiple stores in the same cart? |
| `validateCartContents()` | Validate cart contents (stock, options...). |
| `checkCartContents()` | Are cart contents valid? |
| `setCarts($options)` | Save cart data in order. |
| `setOrderItems($options, $is_delete)` | Convert cart items to order items. |
| `setOrderTotals($options, $is_delete)` | Convert cart totals to order totals. |
| `getCartTotals()` | Calculate full order totals (includes shipping, tip, coupons...). |

#### Customer and User (`StepCustomer`)
| Function | Description |
|----------|-------------|
| `hasFirstOrderCustomer($is_allow, $order_type, $user)` | Is this the customer's first order? |
| `hasAllowFirstOrderCustomer($is_allow)` | Is the feature enabled? |
| `hasFirstOrderCustomerFreeShipping(...)` | Is first order free shipping? |
| `validateCustomerAndUser()` | Check customer and user data existence. |
| `checkCustomerAndUser()` | Quick validity check. |
| `validateCustomerIsBlockedOrders()` | Is the customer blocked from orders? |
| `setCustomerAndUser($options)` | Set customer and user data on order. |

#### Store, Location, and Date (`StepLocation`)
| Function | Description |
|----------|-------------|
| `hasDefaultShop()` / `hasRequiredShop(...)` / `validateDepartments()` / `setDepartments($options)` | Manage store selection and its requirement. |
| `validateAppOpen()` / `validateShopOpen()` / `validateOrdersTypeOpen()` / `validateLocationOpen()` | Check app, store, and order type readiness. |
| `validateOrderDate($is_safe)` / `setOrderDate($options)` / `getOrderDate($is_safe)` | Manage order date. |
| `isDateTimeFormat($value)` | Check date format. |
| `hasFreeShopShipping($is_allow)` / `hasAllowFreeShopShipping(...)` / `getCustomShippingInShop(...)` | Free or custom store shipping. |
| `hasFirstOrderShopCustomer(...)` / `hasAllowFirstOrderShopCustomer(...)` | First order from a specific store. |
| `hasAllowStopDelivery(...)` / `hasStopDelivery(...)` / `validateStopDelivery()` | Stop delivery in a specific store. |
| `hasAllowIsActiveShop(...)` / `hasIsActiveShop(...)` / `validateIsActiveShop()` | Check store activity. |
| `hasExtentOfDelivery(...)` / `validateExtentOfDelivery()` | Delivery range for the store. |
| `setAutoNerbyShop()` | Auto redirect to nearest open store. |

#### Return Date (`StepReturnDate`)
| `hasAllowReturnDate()` / `validateReturnDate()` / `setReturnDate($options)` | Manage return date. |

#### State – City (`StepState`)
| `hasAllowState()` / `hasRequiredState()` / `validateState()` / `setState($options)` | Manage city/state selection. |

#### Shipping Address (`StepShippingAddress`)
| `hasAllowShippingAddress()` / `hasRequiredShippingAddress()` / `validateShippingAddress()` / `setShippingAddress(...)` | Process shipping address. |

#### Billing Address (`StepBillingAddress`)
| `hasAllowBillingAddress()` / `hasRequiredBillingAddress()` / `validateBillingAddress()` / `setBillingAddress(...)` | Process billing address. |

#### From Address (`StepFromAddress`)
| `hasAllowFromAddress()` / `hasRequiredFromAddress()` / `validateFromAddress()` / `setFromAddress(...)` | Process from (origin) address. |

#### Customer Notes (`StepCustomerNotes`)
| `hasAllowCustomerNotes()` / `validateCustomerNotes()` / `setCustomerNotes($options)` | Manage customer notes. |

#### Expected Total (`StepExpectedCartTotal`)
| `hasAllowExpectedCartTotal()` / `validateExpectedCartTotal()` / `setExpectedCartTotal($options)` | Manage expected order value (for orders without cart). |

#### Vehicle Type (`StepVehicle`)
| `hasAllowVehicleType()` / `validateVehicleType()` / `setVehicleType($options)` | Manage vehicle type. |

#### Load (`StepLoad`)
| `hasAllowLoadType()` / `hasRequiredLoadType()` / `setLoadType($options)` | Manage load type and properties (weight, dimensions...). |
| `hasAllowLoadPeople()` / `setLoadPeople($options)` | Manage passenger count. |
| `validateAllLoadType()` / `hasAllowAllOrOneLoadType()` | Check all load options. |

#### Images (`StepImage`)
| `hasAllowImage()` / `validateImage()` / `setImage($options)` / `setImage2($options)` / `setImage3($options)` | Upload and manage order images. |
| `trans_image($file, ...)` / `trans_images($is_mate_data)` | Convert images to array of links. |
| `setStepUpload($data, ...)` | Combined step to upload images. |

#### Booking (Tickets, Trips) (`StepBooking`)
| `hasAllowBooking()` / `validateBooking($is_safe, $data)` / `setBooking($options, $postData)` | Manage booking data. |

#### Shipping and Expected Costs (`StepShipingAndExpected`)
| Function | Description |
|----------|-------------|
| `hasAllowShiping()` / `hasRequiredShiping()` | Is shipping required? |
| `validateShiping()` | Validate shipping data. |
| `getCalcDistanceMetersOrder($order)` | Calculate distance in meters. |
| `getCalcTotalOrder($order)` | Calculate order total. |
| `getShippingMethodForOrder($order, $options)` | Determine best shipping method. |
| `getListShippingMethod($order, $options)` | Matching shipping methods list. |
| `getFixedShippingMethod($order, $options)` | Fixed shipping method. |
| `getMinShippingMethod($order, $options)` / `getMaxShippingMethod(...)` | Min/Max shipping price. |
| `getMaxDistanceShippingMethod($order, $options)` / `getMinDistanceShippingMethod(...)` | Max/Min allowed distance. |
| `checkMaxDistanceShipping($options)` / `validateMaxDistanceShipping($options)` | Check exceeding max distance. |
| `checkMinDistanceShipping($options)` / `validateMinDistanceShipping($options)` | Check minimum distance. |
| `getProcessShippingMethod($shipping_method, $order, $distance)` | Calculate final shipping cost. |
| `processShippingMethod(&$order, $shipping_method, &$data, $is_output)` | Apply shipping method to order. |
| `setShiping($options)` | Set shipping method and cost. |
| `hasAllowExpectedShiping()` / `setExpectedShipping($options)` | Manage expected shipping cost. |
| `bindAutoCouponShippingEvent()` | Apply automatic shipping coupons. |

#### Coupon and Tip (`StepCouponAndTip`)
| `hasAllowCoupon()` / `setCoupon($options)` / `validateCoupon()` | Manage discount coupons. |
| `hasAllowTip()` / `setTip($options)` / `validateTip()` | Manage tip. |
| `hasAllowCouponShiping()` | Allow shipping coupon? |

#### Payment (`StepPay`)
| `hasAllowPay()` / `validatePay()` / `setPay($options)` | Manage payment process. |
| `setPaymentMethodId($options)` | Set payment method. |
| `getPaymentMethodObj($payment_method_id)` | Get payment method object. |
| `getPaymentMethodForOrder($order)` | Payment method linked to order. |
| `validatePaymentMethodId()` | Verify existence of payment method. |

#### Cart Total Min/Max Limits (`StepShipingMinMaxCartTotal`)
| `hasAllowMinMaxCartTotal(...)` / `hasAllowMinOrMaxCartTotal(...)` | Activate cart total limit check. |
| `checkMinCartTotalShipping($options)` / `validateMinCartTotalShipping($options)` | Check minimum cart total. |
| `checkMaxCartTotalShipping($options)` / `validateMaxCartTotalShipping($options)` | Check maximum cart total. |
| `validateMinMaxCartTotal($is_allow)` | Check both limits. |

#### Combined Step Functions
| Function | Trait | Description |
|----------|-------|-------------|
| `validateStepDetails()` | `StepDetails` | Validate all order details (cart, customer, store, address...). |
| `setStepDetails($data, ...)` | `StepDetails` | Execute all order details at once. |
| `validateStepShiping()` | `StepShiping` | Validate shipping and delivery. |
| `setStepShiping($data, ...)` | `StepShiping` | Set shipping and delivery. |
| `validateStepCoupons()` | `StepCoupons` | Validate coupon and tip. |
| `setStepCoupons($data, ...)` | `StepCoupons` | Set coupon and tip. |
| `validateStepPayments()` | `StepPayments` | Validate payment. |
| `setStepPayments($data, ...)` | `StepPayments` | Execute payment. |

---

## 🧱 Class Structure and Design

`OrderManager` implements the **Singleton** pattern to ensure only one instance exists during the order lifecycle.  
It also uses the `SessionMaker` trait to handle the user session (storing and retrieving temporary order data).  
The class is divided into several traits representing separate steps, each providing functions for validation (`validate`) and execution (`set`).

### Core Class Properties
| Property | Type | Description |
|----------|------|-------------|
| `$sessionKey` | `string` | Session key used to store order data (`nano.checkout.order`) |
| `$orderModel` | `Order` | Current order model |
| `$ordersTypeModel` | `OrdersType` | Order type settings (delivery, pickup...) |
| `$cartManager` | `CartManager` | Cart manager |
| `$cart` | `Cart` | Current cart object |
| `$location` | `Location` | Current geographic location object |
| `$customer` | `Customer` | Customer associated with the order |
| `$user` | `User` | Logged-in user |
| `$instance_type` | `string` | Current order type (e.g., `delivery`) |

Constants:
```php
const DELIVERY = 'delivery';
const COLLECTION = 'collection';
const WALK = 'walk';
const DEFAULT_INSTANCE_TYPE = 'delivery';
```

### 🧩 Session Functions

The class has a set of functions for managing the order via session (inherited from `SessionMaker`):

```php
// Set current order ID
$manager->setCurrentOrderId(15);

// Get the ID
$manager->getCurrentOrderId();

// Check if the current ID matches
$manager->isCurrentOrderId(15); // true/false

// Clear the ID
$manager->clearCurrentOrderId();

// Manage selected payment code
$manager->setCurrentPaymentCode('paypal');
$manager->getCurrentPaymentCode();
```

### 🌐 Location and Readiness Check Functions

```php
// Check if the app is open (check_app_open setting)
$manager->validateDefaultLocationOpened();

// Check if the current store is open
$manager->validateEditLocationOpened();

// Check if the store associated with the order is open
$manager->validateOrderLocationOpened();

// Check working hours
$manager->validateOrderTime();
```

### 🛒 Order Initialization and Type

The `init()` function performs the following sequence:
1. Calls `instanceType()` to determine the order type.
2. Loads the user from `Auth`.
3. Loads the cart from `CartManager::instance()->instanceType(...)`.
4. Creates an empty order model or retrieves the existing one for the user.

```php
$manager = OrderManager::instance()->init();
```

The order type can be changed during the session:
```php
$manager->instanceType('booking_hotel', $user);
```

### 📦 `setStepDetails` Function – Main Combined Step

This function executes all basic steps at once:
- Cart (`setCarts`)
- Store (`setDepartments`)
- Customer (`setCustomerAndUser`)
- City (`setState`)
- Order date (`setOrderDate`)
- Return date (`setReturnDate`)
- Customer notes (`setCustomerNotes`)
- Vehicle type (`setVehicleType`)
- Load (`setLoadType`)
- Passenger count (`setLoadPeople`)
- Expected value (`setExpectedCartTotal`)
- Booking (`setBooking`)
- Shipping address (`setShippingAddress`)
- Billing address (`setBillingAddress`)
- From address (`setFromAddress`)
- Images (`setImage`, `setImage2`, `setImage3`)
- Rebuild order items and totals (`setOrderItems`, `setOrderTotals`)

Example call:
```php
$result = $manager->setStepDetails([
    'departments_id'   => 3,
    'state_id'         => 9,
    'order_date'       => '2026-05-03 20:00:00',
    'shipping_address_id' => 25,
    'customer_notes'   => 'Please call before delivery',
    'vehicle_type_ref_type' => 'car_small',
    'load_people'      => 2,
    'image'            => $base64Image,
]);
```

### 🚚 Shipping and Distances – Advanced Details

#### Distance Calculation
```php
// Distance between shipping address and from address (in meters)
$meters = $manager->getCalcDistanceMetersOrder();
```

#### Getting the Appropriate Shipping Method
```php
// Optimal shipping method for the order
$shipping = $manager->getShippingMethodForOrder();

// With custom options
$shipping = $manager->getShippingMethodForOrder($order, [
    'stateId' => 5,
    'countryId' => 120,
]);
```

#### Special Shipping Methods
```php
// Fixed shipping method (FLAG_TYPE_FIXED)
$fixed = $manager->getFixedShippingMethod($order);

// Minimum shipping price available
$minMethod = $manager->getMinShippingMethod($order);

// Maximum allowed delivery distance
$maxDistMethod = $manager->getMaxDistanceShippingMethod($order);
```

#### Checking Shipping Restrictions
```php
// Check not to exceed the maximum distance
try {
    $manager->validateMaxDistanceShipping();
} catch (ApplicationException $e) {
    echo $e->getMessage(); // "Distance exceeds allowed limit (15 km)"
}

// Same for minimum distance limit
$manager->validateMinDistanceShipping();
```

#### Free and Custom Shipping
```php
// Does the store offer free shipping?
if ($manager->hasFreeShopShipping()) {
    // Free shipping
}

// Get custom shipping value (if any)
$customShipping = $manager->getCustomShippingInShop();

// Check if this is the customer's first order in this store
if ($manager->hasFirstOrderShopCustomer()) {
    // Apply discount or special procedure
}

// Automatically redirect order to the nearest open store (if out of range)
if ($manager->setAutoNerbyShop()) {
    echo 'Order redirected to a closer store';
}
```

### 💰 Cart Total Minimum and Maximum Limits

```php
// Is the limit check active?
if ($manager->hasAllowMinMaxCartTotal()) {
    // Check values
    $manager->validateMinCartTotalShipping(); // minimum
    $manager->validateMaxCartTotalShipping(); // maximum
}
```

### 🏷️ Coupons and Tips

```php
// Apply coupon
$result = $manager->setCoupon([
    'coupon_code' => 'SALE20',
]);
if (!$result['status']) {
    echo $result['error'];
}

// Add tip
$manager->setTip([
    'tip_amount' => 5.00, // or percentage
    'amountType' => 'amount', // 'amount' or 'percent'
]);
```

#### Automatic Shipping Coupons
When calculating shipping (`setShiping`), `bindAutoCouponShippingEvent()` is called automatically to search for automated shipping coupons and apply them.

### 💳 Payment and Payment Gateway

#### Setting Payment Method
```php
$manager->setPaymentMethodId(['payment_method_id' => 3]);
```

#### Processing Payment
```php
$result = $manager->setPay([
    'payment_method_id' => 3,
    'card_number'       => '4111111111111111',
    'card_expiry_month' => 12,
    'card_expiry_year'  => 2028,
    'card_cvv'          => '123',
]);

if ($result['code'] == 300) {
    // Redirect to external payment page
    header('Location: ' . $result['redirectUrl']);
} elseif ($result['code'] == 200) {
    echo 'Payment successful';
}
```

#### Interacting with Payment Gateway Directly
```php
// Gateway object
$gateway = $manager->mackPaymentGateway();

// Custom gateway with data
$gateway = $manager->getPaymentGateway($paymentMethod, $data);

// Payment processing service
$service = $manager->getPaymentService($gateway, $order, $url);
$redirector = $service->process();
```

### 📸 Images and Attachments

```php
// Upload image from Base64
$manager->setImage(['image' => $base64String]);

// Upload via file
$manager->setImage(['image_file' => Input::file('photo')]);

// Convert uploaded images to an array of links
$images = $manager->trans_images();
```

### 🔧 Exception Rules System

Allows deleting some validation rules from being mandatory (e.g., making certain fields non-required based on settings).

```php
// Array of excepted rules
$exceptions = OrderManager::getExceptRules();

// Apply exceptions to a rules array
$rules = ['name' => 'required', 'email' => 'required|email'];
$rules = OrderManager::exceptRules($rules); // might remove 'email' based on settings
```

### ⚙️ Integrated Use Scenarios

#### 🛵 1. Restaurant Delivery Order (Full Scenario)
```php
$manager = OrderManager::instance()->init();

// 1. Set store, date, and address
$manager->setStepDetails([
    'departments_id'    => 5,
    'order_date'        => '2026-05-03 19:30:00',
    'shipping_address_id' => 42,
]);

// 2. Shipping
$manager->setStepShiping();

// 3. Coupon (if any)
$manager->setStepCoupons(['coupon_code' => 'NEWUSER']);

// 4. Payment
$payment = $manager->setStepPayments(['payment_method_id' => 1]);
```

#### 🏨 2. Hotel Booking (Without Cart)
```php
$manager = OrderManager::instance()->init('booking_hotel');

$manager->setStepDetails([
    'expected_cart_total' => 500, // Booking value
    'customer_notes'      => 'Late check-in',
    'booking_number'      => 'BK123',
    // ... other booking data
]);
```

#### 🚕 3. Ride Order (With Passenger Count)
```php
$manager = OrderManager::instance()->init('ride');

$manager->setStepDetails([
    'load_people' => 3,
    'vehicle_type_ref_type' => 'car_sedan',
    // ...
]);
```

#### 🔄 4. Automatic Store Redirection When Out of Range
```php
try {
    $manager->validateExtentOfDelivery();
} catch (ApplicationException $e) {
    // Attempt to suggest nearest store
    if ($manager->setAutoNerbyShop()) {
        echo 'Order redirected to alternate store';
    }
}
```

### 📢 Events Triggered Inside the Class

The class integrates with the NanoSoft App event system, and several events are fired:

| Event | Timing |
|-------|--------|
| `nano.checkout.beforeSaveOrder` | Before saving the order |
| `nano.checkout.afterSaveOrder` | After saving the order |
| `nano.checkout.beforePaymentMethod` | Before linking payment method |
| `nano.checkout.beforeShippingMethod` | Before applying shipping method |
| `nano.checkout.beforePayment` | Before starting payment process |
| `nano.order.beforePaymentProcessed` | Before marking "paid" |
| `nano.order.paymentProcessed` | After marking "paid" |
| `nano.orders.newOrderStatus` | When order status changes (fired from the order model itself, but OrderManager may be part of the chain) |

### 🧪 Accessing Internal Objects

Managed objects can be accessed directly:
```php
$order       = $manager->getOrderModel();
$cartManager = $manager->getCartManager();
$cart        = $manager->getCart();
$user        = $manager->getUser();
```

---

## 📘 Illustrative Examples

### 1. Create Order Manager and Start Checkout
```php
$orderManager = OrderManager::instance();
$orderManager->init(); // Initializes session, cart, user, order type
```

### 2. Set Store and Customer
```php
$orderManager->setStepDetails([
    'departments_id' => 12,
    'state_id'       => 5,
    'order_date'     => '2026-05-03 20:30:00',
    'shipping_address_id' => 101,
]);
```

### 3. Calculate Shipping
```php
$orderManager->setStepShiping([
    'shipping_method_id' => 7,
]);
```

### 4. Apply Coupon
```php
$orderManager->setStepCoupons([
    'coupon_code' => 'RAMADAN10',
]);
```

### 5. Process Payment
```php
$paymentResult = $orderManager->setStepPayments([
    'payment_method_id' => 2,
    'card_number'       => '4111111111111111',
    'card_expiry_month' => 12,
    'card_expiry_year'  => 2028,
    'card_cvv'          => 123,
]);
```

### 6. Check Maximum Delivery Distance
```php
$check = $orderManager->checkMaxDistanceShipping();
if (!$check['status']) {
    throw new ApplicationException($check['error']);
}
```

### 7. Get Available Shipping Methods List
```php
$methods = $orderManager->getListShippingMethod();
foreach ($methods as $method) {
    echo $method->name . ' - ' . $method->price;
}
```

---

## 🧰 Advanced Use Scenarios

### 🛵 Delivery Application
- Checks app and restaurant readiness.
- Enforces shipping address and payment method.
- Calculates shipping based on distance and applies automatic shipping coupons.

### 🛍️ Order Without Cart (Services)
- Allows entering `expected_cart_total`.
- Does not require `CartManager`.
- Relies on `hasAllowExpectedCartTotal()`.

### 🧾 Booking Order (Flight Tickets)
- Calls `setBooking()` for fields like `flight_number`, `passport_number`.

### 🔄 Auto Reset Store (Auto Nerby Shop)
- In case of out-of-delivery-range, `setAutoNerbyShop()` can redirect to the nearest branch.

### 🚫 Blocked Customer Prevention
- Execute `validateCustomerIsBlockedOrders()` before any step.

---

## 🔚 Conclusion

`OrderManager` is the backbone of the order process on the Nano.soft platform. Its step-based design makes it easy to customize the order flow for each service type, with the ability to add new steps or disable existing ones via `OrdersType` settings. The combined functions (`setStepDetails`, `setStepShiping`, `setStepCoupons`, `setStepPayments`) provide a simple interface for developers, while individual functions give them full control when needed. The documentation above covers all available functionality and helps in deeper understanding and efficient use of the class.

**Reference Documentation**:
- [`StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-en.md)
- [Advanced `StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-en.md)
- [`HasProductOwnerScopes` Scopes Documentation](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-en.md)
- [Orders API Documentation](./docs/OrdersApi/Docs-OrdersApi-en.md)
