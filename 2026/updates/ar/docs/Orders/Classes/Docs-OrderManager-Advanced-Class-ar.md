## 🧱 هيكل الكلاس وتصميمه

`OrderManager` يطبّق نمط الـ **Singleton** (مفرد) لضمان وجود كائن واحد فقط أثناء دورة حياة الطلب.  
يستخدم أيضاً السمة `SessionMaker` للتعامل مع جلسة المستخدم (تخزين واسترجاع بيانات الطلب المؤقتة).  
ينقسم الكلاس إلى عدة سمات (Traits) تمثّل خطوات منفصلة، وكل سمة توفّر دوال للتحقق (`validate`) والتنفيذ (`set`).

### الخصائص الأساسية للكلاس
| الخاصية | النوع | الوصف |
|---------|-------|--------|
| `$sessionKey` | `string` | مفتاح الجلسة المستخدم لحفظ بيانات الطلب (`nano.checkout.order`) |
| `$orderModel` | `Order` | نموذج الطلب الحالي |
| `$ordersTypeModel` | `OrdersType` | إعدادات نوع الطلب (توصيل، استلام...) |
| `$cartManager` | `CartManager` | مدير السلة |
| `$cart` | `Cart` | كائن السلة الحالي |
| `$location` | `Location` | كائن الموقع الجغرافي الحالي |
| `$customer` | `Customer` | العميل المرتبط بالطلب |
| `$user` | `User` | المستخدم المسجّل الدخول |
| `$instance_type` | `string` | نوع الطلب الحالي (مثلاً `delivery`) |

الثوابت:
```php
const DELIVERY = 'delivery';
const COLLECTION = 'collection';
const WALK = 'walk';
const DEFAULT_INSTANCE_TYPE = 'delivery';
```

---

## 🧩 دوال الجلسة (Session)

يمتلك الكلاس مجموعة دوال لإدارة الطلب عبر الجلسة (موروثة من `SessionMaker`):

```php
// تعيين معرّف الطلب الحالي
$manager->setCurrentOrderId(15);

// جلب المعرّف
$manager->getCurrentOrderId();

// فحص ما إذا كان المعرّف الحالي يطابق
$manager->isCurrentOrderId(15); // true/false

// مسح المعرّف
$manager->clearCurrentOrderId();

// إدارة كود الدفع المختار
$manager->setCurrentPaymentCode('paypal');
$manager->getCurrentPaymentCode();
```

---

## 🌐 دوال الموقع والتحقق من الجهوزية

```php
// التحقق من أن التطبيق مفتوح (إعداد check_app_open)
$manager->validateDefaultLocationOpened();

// التحقق من أن المتجر الحالي مفتوح
$manager->validateEditLocationOpened();

// التحقق من أن المتجر المرتبط بالطلب مفتوح
$manager->validateOrderLocationOpened();

// التحقق من أوقات العمل
$manager->validateOrderTime();
```

---

## 🛒 تهيئة الطلب ونوعه

دالة `init()` تقوم بالتسلسل التالي:
1. تستدعي `instanceType()` لتحديد نوع الطلب.
2. تحمّل المستخدم من `Auth`.
3. تحمّل السلة من `CartManager::instance()->instanceType(...)`.
4. تنشئ نموذج الطلب الفارغ أو تجلب الموجود للمستخدم.

```php
$manager = OrderManager::instance()->init();
```

يمكن تغيير نوع الطلب أثناء الجلسة:
```php
$manager->instanceType('booking_hotel', $user);
```

---

## 📦 دالة `setStepDetails` – الخطوة المجمّعة الرئيسية

هذه الدالة تجمع تنفيذ كل الخطوات الأساسية دفعة واحدة:
- السلة (`setCarts`)
- المتجر (`setDepartments`)
- العميل (`setCustomerAndUser`)
- المدينة (`setState`)
- تاريخ الطلب (`setOrderDate`)
- تاريخ الإرجاع (`setReturnDate`)
- ملاحظات العميل (`setCustomerNotes`)
- نوع المركبة (`setVehicleType`)
- الحمولة (`setLoadType`)
- عدد الركاب (`setLoadPeople`)
- القيمة المتوقعة (`setExpectedCartTotal`)
- الحجز (`setBooking`)
- عنوان الشحن (`setShippingAddress`)
- عنوان الفوترة (`setBillingAddress`)
- عنوان الإرسال (`setFromAddress`)
- الصور (`setImage`, `setImage2`, `setImage3`)
- إعادة بناء عناصر الطلب والإجماليات (`setOrderItems`, `setOrderTotals`)

مثال استدعاء:
```php
$result = $manager->setStepDetails([
    'departments_id'   => 3,
    'state_id'         => 9,
    'order_date'       => '2026-05-03 20:00:00',
    'shipping_address_id' => 25,
    'customer_notes'   => 'الرجاء الاتصال قبل التوصيل',
    'vehicle_type_ref_type' => 'car_small',
    'load_people'      => 2,
    'image'            => $base64Image,
]);
```

---

## 🚚 الشحن والمسافات – تفاصيل متقدمة

### حساب المسافة
```php
// المسافة بين عنوان الشحن وعنوان الإرسال (بالمتر)
$meters = $manager->getCalcDistanceMetersOrder();
```

### جلب طريقة الشحن المناسبة
```php
// طريقة الشحن المثلى حسب الطلب
$shipping = $manager->getShippingMethodForOrder();

// أو مع خيارات مخصصة
$shipping = $manager->getShippingMethodForOrder($order, [
    'stateId' => 5,
    'countryId' => 120,
]);
```

### طرق الشحن الخاصة
```php
// طريقة الشحن الثابتة (FLAG_TYPE_FIXED)
$fixed = $manager->getFixedShippingMethod($order);

// أدنى سعر شحن متاح
$minMethod = $manager->getMinShippingMethod($order);

// أقصى مسافة مسموحة للتوصيل
$maxDistMethod = $manager->getMaxDistanceShippingMethod($order);
```

### فحص القيود على الشحن
```php
// التحقق من عدم تجاوز المسافة القصوى
try {
    $manager->validateMaxDistanceShipping();
} catch (ApplicationException $e) {
    echo $e->getMessage(); // "المسافة تتجاوز الحد المسموح (15 كم)"
}

// بنفس الطريقة للحد الأدنى للمسافة
$manager->validateMinDistanceShipping();
```

### الشحن المجاني والمخصص
```php
// هل المتجر يوفّر شحن مجاني؟
if ($manager->hasFreeShopShipping()) {
    // الشحن مجاني
}

// الحصول على قيمة الشحن المخصص (إن وجد)
$customShipping = $manager->getCustomShippingInShop();

// التحقق من أن هذا أول طلب للعميل في هذا المتجر
if ($manager->hasFirstOrderShopCustomer()) {
    // تطبيق خصم أو إجراء خاص
}

// تحويل الطلب تلقائياً لأقرب متجر مفتوح (إذا كان خارج النطاق)
if ($manager->setAutoNerbyShop()) {
    echo 'تم تحويل الطلب إلى متجر أقرب';
}
```

---

## 💰 الحدود الدنيا والعليا لقيمة السلة

```php
// هل تفعيل فحص الحدود نشط؟
if ($manager->hasAllowMinMaxCartTotal()) {
    // التحقق من القيم
    $manager->validateMinCartTotalShipping(); // الحد الأدنى
    $manager->validateMaxCartTotalShipping(); // الحد الأعلى
}
```

---

## 🏷️ الكوبونات والبقشيش

```php
// تطبيق كوبون
$result = $manager->setCoupon([
    'coupon_code' => 'SALE20',
]);
if (!$result['status']) {
    echo $result['error'];
}

// إضافة بقشيش
$manager->setTip([
    'tip_amount' => 5.00, // أو نسبة مئوية
    'amountType' => 'amount', // 'amount' أو 'percent'
]);
```

### كوبونات الشحن التلقائية
عند حساب الشحن (`setShiping`)، تُستدعى `bindAutoCouponShippingEvent()` تلقائياً للبحث عن كوبونات شحن مؤتمتة وتطبيقها.

---

## 💳 الدفع وبوابة الدفع

### إعداد وسيلة الدفع
```php
$manager->setPaymentMethodId(['payment_method_id' => 3]);
```

### تنفيذ الدفع
```php
$result = $manager->setPay([
    'payment_method_id' => 3,
    'card_number'       => '4111111111111111',
    'card_expiry_month' => 12,
    'card_expiry_year'  => 2028,
    'card_cvv'          => '123',
]);

if ($result['code'] == 300) {
    // إعادة توجيه لصفحة الدفع الخارجية
    header('Location: ' . $result['redirectUrl']);
} elseif ($result['code'] == 200) {
    echo 'تم الدفع بنجاح';
}
```

### التعامل مع بوابة الدفع مباشرة
```php
// كائن البوابة
$gateway = $manager->mackPaymentGateway();

// بوابة مخصصة مع بيانات
$gateway = $manager->getPaymentGateway($paymentMethod, $data);

// خدمة تنفيذ الدفع
$service = $manager->getPaymentService($gateway, $order, $url);
$redirector = $service->process();
```

---

## 📸 الصور والمرفقات

```php
// رفع صورة من Base64
$manager->setImage(['image' => $base64String]);

// رفع عبر ملف
$manager->setImage(['image_file' => Input::file('photo')]);

// تحويل الصور المرفوعة إلى مصفوفة روابط
$images = $manager->trans_images();
```

---

## 🔧 نظام الاستثناءات (Except Rules)

يسمح بحذف بعض قواعد التحقق من الإجبارية (مثلاً إلغاء إجبارية بعض الحقول حسب الإعدادات).

```php
// مصفوفة القواعد المستثناة
$exceptions = OrderManager::getExceptRules();

// تطبيق الاستثناءات على مصفوفة قواعد
$rules = ['name' => 'required', 'email' => 'required|email'];
$rules = OrderManager::exceptRules($rules); // قد يزيل 'email' حسب الإعدادات
```

---

## ⚙️ سيناريوهات استخدام متكاملة

### 🛵 ١. طلب توصيل من مطعم (السيناريو الكامل)
```php
$manager = OrderManager::instance()->init();

// 1. تحديد المتجر والتاريخ والعنوان
$manager->setStepDetails([
    'departments_id'    => 5,
    'order_date'        => '2026-05-03 19:30:00',
    'shipping_address_id' => 42,
]);

// 2. الشحن
$manager->setStepShiping();

// 3. كوبون(إن وجد)
$manager->setStepCoupons(['coupon_code' => 'NEWUSER']);

// 4. دفع
$payment = $manager->setStepPayments(['payment_method_id' => 1]);
```

### 🏨 ٢. حجز فندق (بدون سلة)
```php
$manager = OrderManager::instance()->init('booking_hotel');

$manager->setStepDetails([
    'expected_cart_total' => 500, // قيمة الحجز
    'customer_notes'      => 'تسجيل وصول متأخر',
    'booking_number'      => 'BK123',
    // ... بيانات الحجز الأخرى
]);
```

### 🚕 ٣. طلب توصيل ركاب (مع عدد الأشخاص)
```php
$manager = OrderManager::instance()->init('ride');

$manager->setStepDetails([
    'load_people' => 3,
    'vehicle_type_ref_type' => 'car_sedan',
    // ...
]);
```

### 🔄 ٤. التحويل التلقائي للمتجر عند الخروج عن النطاق
```php
try {
    $manager->validateExtentOfDelivery();
} catch (ApplicationException $e) {
    // محاولة اقتراح أقرب متجر
    if ($manager->setAutoNerbyShop()) {
        echo 'تم تحويل الطلب إلى متجر بديل';
    }
}
```

---

## 📢 الأحداث المُشغّلة داخل الكلاس

يندمج الكلاس مع نظام الأحداث في NanoSoft App وينطلق عدد من الأحداث:

| الحدث | التوقيت |
|--------|--------|
| `nano.checkout.beforeSaveOrder` | قبل حفظ الطلب |
| `nano.checkout.afterSaveOrder` | بعد حفظ الطلب |
| `nano.checkout.beforePaymentMethod` | قبل ربط وسيلة الدفع |
| `nano.checkout.beforeShippingMethod` | قبل تطبيق طريقة الشحن |
| `nano.checkout.beforePayment` | قبل بدء عملية الدفع |
| `nano.order.beforePaymentProcessed` | قبل وضع علامة "تم الدفع" |
| `nano.order.paymentProcessed` | بعد وضع علامة "تم الدفع" |
| `nano.orders.newOrderStatus` | عند تغيير حالة الطلب (يُستدعى من نموذج الطلب نفسه، لكن OrderManager قد يكون جزءاً من السلسلة) |

---

## 🧪 الوصول إلى الكائنات الداخلية

يمكن الوصول إلى الكائنات المُدارة مباشرة:
```php
$order       = $manager->getOrderModel();
$cartManager = $manager->getCartManager();
$cart        = $manager->getCart();
$user        = $manager->getUser();
```

---

**الوثائق المرجعية**:
- [توثيق `OrderManager` وسماته](./docs/Orders/Classes/Docs-OrderManager-Class-ar.md)
- [توثيق السمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-ar.md)
- [توثيق متقدم للسمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-ar.md)
- [توثيق نطاقات `HasProductOwnerScopes`](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-ar.md)
- [توثيق API الطلبات](./docs/OrdersApi/Docs-OrdersApi-ar.md)