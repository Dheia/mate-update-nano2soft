## 🧱 Class Structure and Design

`OrderManager` implements the **Singleton** pattern to ensure only one instance exists during the request lifecycle.  
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

---

## 🧩 Session Functions

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

---

## 🌐 Location and Readiness Check Functions

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

---

## 🛒 Order Initialization and Type

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

---

## 📦 `setStepDetails` Function – Main Combined Step

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

---

## 🚚 Shipping and Distances – Advanced Details

### Distance Calculation
```php
// Distance between shipping address and from address (in meters)
$meters = $manager->getCalcDistanceMetersOrder();
```

### Getting the Appropriate Shipping Method
```php
// Optimal shipping method for the order
$shipping = $manager->getShippingMethodForOrder();

// With custom options
$shipping = $manager->getShippingMethodForOrder($order, [
    'stateId' => 5,
    'countryId' => 120,
]);
```

### Special Shipping Methods
```php
// Fixed shipping method (FLAG_TYPE_FIXED)
$fixed = $manager->getFixedShippingMethod($order);

// Minimum shipping price available
$minMethod = $manager->getMinShippingMethod($order);

// Maximum allowed delivery distance
$maxDistMethod = $manager->getMaxDistanceShippingMethod($order);
```

### Checking Shipping Restrictions
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

### Free and Custom Shipping
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

---

## 💰 Cart Total Minimum and Maximum Limits

```php
// Is the limit check active?
if ($manager->hasAllowMinMaxCartTotal()) {
    // Check values
    $manager->validateMinCartTotalShipping(); // minimum
    $manager->validateMaxCartTotalShipping(); // maximum
}
```

---

## 🏷️ Coupons and Tips

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

### Automatic Shipping Coupons
When calculating shipping (`setShiping`), `bindAutoCouponShippingEvent()` is called automatically to search for automated shipping coupons and apply them.

---

## 💳 Payment and Payment Gateway

### Setting Payment Method
```php
$manager->setPaymentMethodId(['payment_method_id' => 3]);
```

### Processing Payment
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

### Interacting with Payment Gateway Directly
```php
// Gateway object
$gateway = $manager->mackPaymentGateway();

// Custom gateway with data
$gateway = $manager->getPaymentGateway($paymentMethod, $data);

// Payment processing service
$service = $manager->getPaymentService($gateway, $order, $url);
$redirector = $service->process();
```

---

## 📸 Images and Attachments

```php
// Upload image from Base64
$manager->setImage(['image' => $base64String]);

// Upload via file
$manager->setImage(['image_file' => Input::file('photo')]);

// Convert uploaded images to an array of links
$images = $manager->trans_images();
```

---

## 🔧 Exception Rules System

Allows deleting some validation rules from being mandatory (e.g., making certain fields non-required based on settings).

```php
// Array of excepted rules
$exceptions = OrderManager::getExceptRules();

// Apply exceptions to a rules array
$rules = ['name' => 'required', 'email' => 'required|email'];
$rules = OrderManager::exceptRules($rules); // might remove 'email' based on settings
```

---

## ⚙️ Integrated Use Scenarios

### 🛵 1. Restaurant Delivery Order (Full Scenario)
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

### 🏨 2. Hotel Booking (Without Cart)
```php
$manager = OrderManager::instance()->init('booking_hotel');

$manager->setStepDetails([
    'expected_cart_total' => 500, // Booking value
    'customer_notes'      => 'Late check-in',
    'booking_number'      => 'BK123',
    // ... other booking data
]);
```

### 🚕 3. Ride Order (With Passenger Count)
```php
$manager = OrderManager::instance()->init('ride');

$manager->setStepDetails([
    'load_people' => 3,
    'vehicle_type_ref_type' => 'car_sedan',
    // ...
]);
```

### 🔄 4. Automatic Store Redirection When Out of Range
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

---

## 📢 Events Triggered Inside the Class

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

---

## 🧪 Accessing Internal Objects

Managed objects can be accessed directly:
```php
$order       = $manager->getOrderModel();
$cartManager = $manager->getCartManager();
$cart        = $manager->getCart();
$user        = $manager->getUser();
```

**Reference Documentation**:
- [`OrderManager` and its traits Documentation](./docs/Orders/Classes/Docs-OrderManager-Class-en.md)
- [`StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-en.md)
- [Advanced `StepStatus` Trait Documentation](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-en.md)
