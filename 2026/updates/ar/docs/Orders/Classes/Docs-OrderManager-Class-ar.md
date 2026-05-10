# توثيق شامل لكلاس `OrderManager`

**الحزمة:** `Nano.Orders`  
**الكلاس:** `Nano\Orders\Classes\OrderManager`  
**النمط:** Singleton  
**الدور:** مدير عملية الطلب (Checkout) – ينسق جميع الخطوات المطلوبة لإنشاء الطلب، من السلة حتى الدفع، مروراً بالعنوان، الشحن، الكوبونات، الصور، وغيرها.

---

## 📋 مقدمة

يُعدّ `OrderManager` العقل المُدبّر لسير عملية تقديم الطلب في منصة Nano.soft. يعتمد الكلاس على نمط `Singleton` لضمان وجود نسخة واحدة فقط أثناء دورة حياة الطلب، ويستخدم نظام الخطوات (Steps) عبر مجموعة من الـ `Traits` المتخصصة. كل `Trait` يغطي جانباً محدداً من عملية الطلب:

- إدارة السلة والأصناف.
- بيانات العميل والمستخدم.
- المتجر والموقع والتاريخ.
- عنوان الشحن وعنوان الإرسال.
- الحمولة ونوع المركبة.
- طرق الشحن والتكاليف المتوقعة.
- الكوبونات والبقشيش.
- الدفع وبوابة الدفع.
- الصور والمرفقات.

يدعم الكلاس أنماط طلبات متعددة (توصيل، استلام، مشي...) من خلال `OrdersType`، ويوفّر واجهة موحدة للتحقق من صحة كل خطوة (`validate`) وتنفيذها (`set`).

---

## 🧩 جدول الدوال الكامل

ينقسم الجدول إلى قسمين: دوال موجودة مباشرة في `OrderManager`، ودوال قادمة من السمات (Traits) المتصلة به. يُذكر بجانب كل دالة السمة التي تنتمي إليها (إن وجدت).

### ١. دوال النواة (Core Methods) - داخل OrderManager مباشرة

| الدالة | الوصف |
|--------|--------|
| `init()` | تهيئة المدير: تحديد نوع الطلب، تحميل المستخدم، السلة، المتجر. |
| `instanceType($instance, $user)` | تعيين نوع الطلب الحالي (delivery, collection...) وربطه بالمستخدم. |
| `currentInstanceType()` | إرجاع نوع الطلب الحالي. |
| `setOrderModel($order)` / `getOrderModel()` | تعيين أو جلب نموذج الطلب الحالي. |
| `getOrderModelDb($user, $instance)` | جلب نموذج الطلب من قاعدة البيانات. |
| `setOrdersTypeModel($ref_type, $model)` / `getOrdersTypeModel()` | تعيين أو جلب نموذج إعدادات نوع الطلب. |
| `setCart($cart)` / `getCart()` | تعيين أو جلب كائن السلة. |
| `setCartManager($cartManager)` / `getCartManager()` | تعيين أو جلب مدير السلة. |
| `getUser()` / `setUser($user)` / `getUserId()` | إدارة المستخدم المسجّل الدخول. |
| `setCustomer($customer)` / `getCustomerId()` | تعيين العميل وجلب معرّفه. |
| `getOrder()` / `loadOrder()` | تحميل الطلب الحالي (من الجلسة). |
| `getOrderByHash($hash, $customer)` | جلب طلب بواسطة hash. |
| `getDefaultPayment()` / `getDefaultPaymentMethod($order)` | جلب وسيلة الدفع الافتراضية. |
| `getListPaymentMethod($order)` | قائمة وسائل الدفع المتاحة. |
| `processPaymentMethod(&$order, $payment_method, &$data)` | معالجة ربط وسيلة الدفع بالطلب. |
| `getPayment($code)` | جلب وسيلة دفع حسب المعرّف. |
| `getPaymentGateways()` | كل وسائل الدفع النشطة. |
| `findDeliveryAddress($addressId)` | البحث عن عنوان توصيل. |
| `validateCustomer($customer)` | التحقق من صلاحية العميل. |
| `validateDeliveryAddress(array $address)` | التحقق من تغطية منطقة التوصيل للعنوان. |
| `saveOrder($order, array $data)` | حفظ الطلب وإضافة عناصر السلة والإجماليات. |
| `processPayment($order, array $data)` | تنفيذ عملية الدفع عبر البوابة. |
| `applyDepartmentsAttributes($order, $options)` | تطبيق بيانات المتجر على الطلب. |
| `applyRequiredAttributes($order, $options)` | تعبئة السمات الإلزامية (مستخدم، عميل، تاريخ...). |
| `applyCurrentPaymentFee($code)` | تطبيق رسوم وسيلة الدفع على السلة. |
| `byUser(User $user, $instance)` | تحميل نموذج الطلب الخاص بمستخدم معيّن. |
| `validateLocation()` / `validateDefaultLocationOpened()` / `validateEditLocationOpened()` / `validateOrderLocationOpened()` / `validateOrderTime()` | سلسلة فحوصات جاهزية المتجر والتطبيق وأوقات العمل. |
| `getOrderTypeInRequest($defaultValue)` | استخراج `order_type` من الطلب (Header أو Body). |
| `getOrderTypeInRequestHeader($defaultValue)` / `getOrderTypeInRequestBody($defaultValue)` | استخراج نوع الطلب من الهيدر أو البودي. |
| `cleanValueNumber($value)` | إزالة الرموز الحسابية من قيمة نصية. |
| `getExceptRules($is_force)` / `exceptRules($data, $array_except)` | إدارة قواعد الاستثناء (لحذف حقول غير مطلوبة). |
| `clearOrder()` | مسح الطلب من الجلسة وإنهاء عملية الشراء. |
| `setCurrentOrderId($orderId)` / `getCurrentOrderId()` / `clearCurrentOrderId()` / `isCurrentOrderId($orderId)` | إدارة معرّف الطلب الحالي في الجلسة. |
| `setCurrentPaymentCode($code)` / `getCurrentPaymentCode()` | إدارة كود وسيلة الدفع المختارة. |
| `mackPaymentGateway()` | إنشاء كائن بوابة الدفع. |
| `getPaymentGateway($payment_method, $data)` | إنشاء بوابة دفع مع بيانات. |
| `getPaymentGatewayOrder($order, $payment_method, $data)` | بوابة دفع مرتبطة بالطلب. |
| `getPaymentService($gateway, $order, $url, $data)` | إنشاء خدمة تنفيذ الدفع. |

### ٢. دوال الخطوات (Traits) – مُلخّصة حسب المجموعة

#### السلة والأصناف (`StepCart`)
| الدالة | الوصف |
|--------|--------|
| `hasRequiredCart()` | هل نوع الطلب يتطلب سلة مشتريات؟ |
| `hasMultiShopInCart($is_cart)` | هل يسمح بمنتجات من عدة متاجر في نفس السلة؟ |
| `validateCartContents()` | التحقق من محتويات السلة (مخزون، خيارات...). |
| `checkCartContents()` | هل محتويات السلة صالحة؟ |
| `setCarts($options)` | حفظ بيانات السلة في الطلب. |
| `setOrderItems($options, $is_delete)` | تحويل عناصر السلة إلى عناصر طلب. |
| `setOrderTotals($options, $is_delete)` | تحويل إجماليات السلة إلى إجماليات الطلب. |
| `getCartTotals()` | حساب إجماليات الطلب الكاملة (يشمل الشحن، البقشيش، الكوبونات...). |

#### العميل والمستخدم (`StepCustomer`)
| الدالة | الوصف |
|--------|--------|
| `hasFirstOrderCustomer($is_allow, $order_type, $user)` | هل هذا أول طلب للعميل؟ |
| `hasAllowFirstOrderCustomer($is_allow)` | هل الخاصية مفعلة؟ |
| `hasFirstOrderCustomerFreeShipping(...)` | هل أول طلب مجاني الشحن؟ |
| `validateCustomerAndUser()` | التحقق من وجود بيانات العميل والمستخدم. |
| `checkCustomerAndUser()` | فحص سريع للصلاحية. |
| `validateCustomerIsBlockedOrders()` | هل العميل محظور من الطلبات؟ |
| `setCustomerAndUser($options)` | تعيين بيانات العميل والمستخدم على الطلب. |

#### المتجر والموقع والتاريخ (`StepLocation`)
| الدالة | الوصف |
|--------|--------|
| `hasDefaultShop()` / `hasRequiredShop(...)` / `validateDepartments()` / `setDepartments($options)` | إدارة اختيار المتجر وإلزاميته. |
| `validateAppOpen()` / `validateShopOpen()` / `validateOrdersTypeOpen()` / `validateLocationOpen()` | التحقق من جاهزية التطبيق والمتجر ونوع الطلب. |
| `validateOrderDate($is_safe)` / `setOrderDate($options)` / `getOrderDate($is_safe)` | إدارة تاريخ الطلب. |
| `isDateTimeFormat($value)` | التحقق من صيغة التاريخ. |
| `hasFreeShopShipping($is_allow)` / `hasAllowFreeShopShipping(...)` / `getCustomShippingInShop(...)` | الشحن المجاني أو المخصص للمتجر. |
| `hasFirstOrderShopCustomer(...)` / `hasAllowFirstOrderShopCustomer(...)` | أول طلب من متجر معين. |
| `hasAllowStopDelivery(...)` / `hasStopDelivery(...)` / `validateStopDelivery()` | إيقاف التوصيل في متجر معين. |
| `hasAllowIsActiveShop(...)` / `hasIsActiveShop(...)` / `validateIsActiveShop()` | التحقق من نشاط المتجر. |
| `hasExtentOfDelivery(...)` / `validateExtentOfDelivery()` | مدى التوصيل للمتجر. |
| `setAutoNerbyShop()` | التحويل التلقائي لأقرب متجر مفتوح. |

#### تاريخ الإرجاع (`StepReturnDate`)
| `hasAllowReturnDate()` / `validateReturnDate()` / `setReturnDate($options)` | إدارة تاريخ الإرجاع. |

#### الحالة – المدينة (`StepState`)
| `hasAllowState()` / `hasRequiredState()` / `validateState()` / `setState($options)` | إدارة اختيار المدينة/الولاية. |

#### عنوان الشحن (`StepShippingAddress`)
| `hasAllowShippingAddress()` / `hasRequiredShippingAddress()` / `validateShippingAddress()` / `setShippingAddress(...)` | معالجة عنوان الشحن. |

#### عنوان الفوترة (`StepBillingAddress`)
| `hasAllowBillingAddress()` / `hasRequiredBillingAddress()` / `validateBillingAddress()` / `setBillingAddress(...)` | معالجة عنوان الفوترة. |

#### عنوان الإرسال (From) (`StepFromAddress`)
| `hasAllowFromAddress()` / `hasRequiredFromAddress()` / `validateFromAddress()` / `setFromAddress(...)` | معالجة عنوان الانطلاق. |

#### ملاحظات العميل (`StepCustomerNotes`)
| `hasAllowCustomerNotes()` / `validateCustomerNotes()` / `setCustomerNotes($options)` | إدارة ملاحظات العميل. |

#### الإجمالي المتوقع (`StepExpectedCartTotal`)
| `hasAllowExpectedCartTotal()` / `validateExpectedCartTotal()` / `setExpectedCartTotal($options)` | إدارة قيمة الطلب المتوقعة (للطلبات بدون سلة). |

#### نوع المركبة (`StepVehicle`)
| `hasAllowVehicleType()` / `validateVehicleType()` / `setVehicleType($options)` | إدارة نوع المركبة. |

#### الحمولة (`StepLoad`)
| `hasAllowLoadType()` / `hasRequiredLoadType()` / `setLoadType($options)` | إدارة نوع الحمولة وخصائصها (وزن، أبعاد...). |
| `hasAllowLoadPeople()` / `setLoadPeople($options)` | إدارة عدد الركاب. |
| `validateAllLoadType()` / `hasAllowAllOrOneLoadType()` | فحص كل خيارات الحمولة. |

#### الصور (`StepImage`)
| `hasAllowImage()` / `validateImage()` / `setImage($options)` / `setImage2($options)` / `setImage3($options)` | رفع وإدارة صور الطلب. |
| `trans_image($file, ...)` / `trans_images($is_mate_data)` | تحويل الصور لمصفوفة روابط. |
| `setStepUpload($data, ...)` | خطوة مجمعة لرفع الصور. |

#### الحجز (تذاكر، رحلات) (`StepBooking`)
| `hasAllowBooking()` / `validateBooking($is_safe, $data)` / `setBooking($options, $postData)` | إدارة بيانات الحجز. |

#### الشحن والتكاليف المتوقعة (`StepShipingAndExpected`)
| الدالة | الوصف |
|--------|--------|
| `hasAllowShiping()` / `hasRequiredShiping()` | هل الشحن مطلوب؟ |
| `validateShiping()` | التحقق من بيانات الشحن. |
| `getCalcDistanceMetersOrder($order)` | حساب المسافة بالمتر. |
| `getCalcTotalOrder($order)` | حساب إجمالي الطلب. |
| `getShippingMethodForOrder($order, $options)` | تحديد أنسب طريقة شحن. |
| `getListShippingMethod($order, $options)` | قائمة طرق الشحن المطابقة. |
| `getFixedShippingMethod($order, $options)` | طريقة الشحن الثابتة. |
| `getMinShippingMethod($order, $options)` / `getMaxShippingMethod(...)` | أدنى/أعلى سعر شحن. |
| `getMaxDistanceShippingMethod($order, $options)` / `getMinDistanceShippingMethod(...)` | أقصى/أدنى مسافة مسموحة. |
| `checkMaxDistanceShipping($options)` / `validateMaxDistanceShipping($options)` | فحص تجاوز المسافة القصوى. |
| `checkMinDistanceShipping($options)` / `validateMinDistanceShipping($options)` | فحص المسافة الدنيا. |
| `getProcessShippingMethod($shipping_method, $order, $distance)` | احتساب تكلفة الشحن النهائية. |
| `processShippingMethod(&$order, $shipping_method, &$data, $is_output)` | تطبيق طريقة الشحن على الطلب. |
| `setShiping($options)` | تعيين طريقة الشحن والتكلفة. |
| `hasAllowExpectedShiping()` / `setExpectedShipping($options)` | إدارة التكلفة المتوقعة للشحن. |
| `bindAutoCouponShippingEvent()` | تطبيق كوبونات الشحن التلقائية. |

#### الكوبون والبقشيش (`StepCouponAndTip`)
| `hasAllowCoupon()` / `setCoupon($options)` / `validateCoupon()` | إدارة كوبونات الخصم. |
| `hasAllowTip()` / `setTip($options)` / `validateTip()` | إدارة البقشيش. |
| `hasAllowCouponShiping()` | هل يسمح بكوبون شحن؟ |

#### الدفع (`StepPay`)
| `hasAllowPay()` / `validatePay()` / `setPay($options)` | إدارة عملية الدفع. |
| `setPaymentMethodId($options)` | تعيين وسيلة الدفع. |
| `getPaymentMethodObj($payment_method_id)` | جلب كائن وسيلة الدفع. |
| `getPaymentMethodForOrder($order)` | وسيلة الدفع المرتبطة بالطلب. |
| `validatePaymentMethodId()` | التحقق من وجود وسيلة دفع. |

#### حدود السلة الدنيا/العليا (`StepShipingMinMaxCartTotal`)
| `hasAllowMinMaxCartTotal(...)` / `hasAllowMinOrMaxCartTotal(...)` | تفعيل فحص حدود السلة. |
| `checkMinCartTotalShipping($options)` / `validateMinCartTotalShipping($options)` | فحص الحد الأدنى لقيمة السلة. |
| `checkMaxCartTotalShipping($options)` / `validateMaxCartTotalShipping($options)` | فحص الحد الأعلى. |
| `validateMinMaxCartTotal($is_allow)` | فحص كلا الحدين. |

#### دوال مجمّعة للخطوات
| الدالة | السمة | الوصف |
|--------|--------|--------|
| `validateStepDetails()` | `StepDetails` | التحقق من كل تفاصيل الطلب (السلة، العميل، المتجر، العنوان...). |
| `setStepDetails($data, ...)` | `StepDetails` | تنفيذ كل تفاصيل الطلب دفعة واحدة. |
| `validateStepShiping()` | `StepShiping` | التحقق من الشحن والتوصيل. |
| `setStepShiping($data, ...)` | `StepShiping` | تعيين الشحن والتوصيل. |
| `validateStepCoupons()` | `StepCoupons` | التحقق من الكوبون والبقشيش. |
| `setStepCoupons($data, ...)` | `StepCoupons` | تعيين الكوبون والبقشيش. |
| `validateStepPayments()` | `StepPayments` | التحقق من الدفع. |
| `setStepPayments($data, ...)` | `StepPayments` | تنفيذ الدفع. |

---

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

### 🧩 دوال الجلسة (Session)

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

### 🌐 دوال الموقع والتحقق من الجهوزية

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

### 🛒 تهيئة الطلب ونوعه

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

### 📦 دالة `setStepDetails` – الخطوة المجمّعة الرئيسية

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

### 🚚 الشحن والمسافات – تفاصيل متقدمة

#### حساب المسافة
```php
// المسافة بين عنوان الشحن وعنوان الإرسال (بالمتر)
$meters = $manager->getCalcDistanceMetersOrder();
```

#### جلب طريقة الشحن المناسبة
```php
// طريقة الشحن المثلى حسب الطلب
$shipping = $manager->getShippingMethodForOrder();

// أو مع خيارات مخصصة
$shipping = $manager->getShippingMethodForOrder($order, [
    'stateId' => 5,
    'countryId' => 120,
]);
```

#### طرق الشحن الخاصة
```php
// طريقة الشحن الثابتة (FLAG_TYPE_FIXED)
$fixed = $manager->getFixedShippingMethod($order);

// أدنى سعر شحن متاح
$minMethod = $manager->getMinShippingMethod($order);

// أقصى مسافة مسموحة للتوصيل
$maxDistMethod = $manager->getMaxDistanceShippingMethod($order);
```

#### فحص القيود على الشحن
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

#### الشحن المجاني والمخصص
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

### 💰 الحدود الدنيا والعليا لقيمة السلة

```php
// هل تفعيل فحص الحدود نشط؟
if ($manager->hasAllowMinMaxCartTotal()) {
    // التحقق من القيم
    $manager->validateMinCartTotalShipping(); // الحد الأدنى
    $manager->validateMaxCartTotalShipping(); // الحد الأعلى
}
```

### 🏷️ الكوبونات والبقشيش

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

#### كوبونات الشحن التلقائية
عند حساب الشحن (`setShiping`)، تُستدعى `bindAutoCouponShippingEvent()` تلقائياً للبحث عن كوبونات شحن مؤتمتة وتطبيقها.

### 💳 الدفع وبوابة الدفع

#### إعداد وسيلة الدفع
```php
$manager->setPaymentMethodId(['payment_method_id' => 3]);
```

#### تنفيذ الدفع
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

#### التعامل مع بوابة الدفع مباشرة
```php
// كائن البوابة
$gateway = $manager->mackPaymentGateway();

// بوابة مخصصة مع بيانات
$gateway = $manager->getPaymentGateway($paymentMethod, $data);

// خدمة تنفيذ الدفع
$service = $manager->getPaymentService($gateway, $order, $url);
$redirector = $service->process();
```

### 📸 الصور والمرفقات

```php
// رفع صورة من Base64
$manager->setImage(['image' => $base64String]);

// رفع عبر ملف
$manager->setImage(['image_file' => Input::file('photo')]);

// تحويل الصور المرفوعة إلى مصفوفة روابط
$images = $manager->trans_images();
```

### 🔧 نظام الاستثناءات (Except Rules)

يسمح بحذف بعض قواعد التحقق من الإجبارية (مثلاً إلغاء إجبارية بعض الحقول حسب الإعدادات).

```php
// مصفوفة القواعد المستثناة
$exceptions = OrderManager::getExceptRules();

// تطبيق الاستثناءات على مصفوفة قواعد
$rules = ['name' => 'required', 'email' => 'required|email'];
$rules = OrderManager::exceptRules($rules); // قد يزيل 'email' حسب الإعدادات
```

### ⚙️ سيناريوهات استخدام متكاملة

#### 🛵 ١. طلب توصيل من مطعم (السيناريو الكامل)
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

#### 🏨 ٢. حجز فندق (بدون سلة)
```php
$manager = OrderManager::instance()->init('booking_hotel');

$manager->setStepDetails([
    'expected_cart_total' => 500, // قيمة الحجز
    'customer_notes'      => 'تسجيل وصول متأخر',
    'booking_number'      => 'BK123',
    // ... بيانات الحجز الأخرى
]);
```

#### 🚕 ٣. طلب توصيل ركاب (مع عدد الأشخاص)
```php
$manager = OrderManager::instance()->init('ride');

$manager->setStepDetails([
    'load_people' => 3,
    'vehicle_type_ref_type' => 'car_sedan',
    // ...
]);
```

#### 🔄 ٤. التحويل التلقائي للمتجر عند الخروج عن النطاق
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

### 📢 الأحداث المُشغّلة داخل الكلاس

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

### 🧪 الوصول إلى الكائنات الداخلية

يمكن الوصول إلى الكائنات المُدارة مباشرة:
```php
$order       = $manager->getOrderModel();
$cartManager = $manager->getCartManager();
$cart        = $manager->getCart();
$user        = $manager->getUser();
```

---

## 📘 أمثلة توضيحية

### ١. إنشاء مدير الطلب وبدء عملية الشراء
```php
$orderManager = OrderManager::instance();
$orderManager->init(); // يهيئ الجلسة، السلة، المستخدم، نوع الطلب
```

### ٢. تعيين المتجر والعميل
```php
$orderManager->setStepDetails([
    'departments_id' => 12,
    'state_id'       => 5,
    'order_date'     => '2026-05-03 20:30:00',
    'shipping_address_id' => 101,
]);
```

### ٣. احتساب الشحن
```php
$orderManager->setStepShiping([
    'shipping_method_id' => 7,
]);
```

### ٤. تطبيق كوبون
```php
$orderManager->setStepCoupons([
    'coupon_code' => 'RAMADAN10',
]);
```

### ٥. تنفيذ الدفع
```php
$paymentResult = $orderManager->setStepPayments([
    'payment_method_id' => 2,
    'card_number'       => '4111111111111111',
    'card_expiry_month' => 12,
    'card_expiry_year'  => 2028,
    'card_cvv'          => 123,
]);
```

### ٦. فحص المسافة القصوى للتوصيل
```php
$check = $orderManager->checkMaxDistanceShipping();
if (!$check['status']) {
    throw new ApplicationException($check['error']);
}
```

### ٧. الحصول على قائمة طرق الشحن المتاحة
```php
$methods = $orderManager->getListShippingMethod();
foreach ($methods as $method) {
    echo $method->name . ' - ' . $method->price;
}
```

---

## 🧰 سيناريوهات استخدام متقدمة

### 🛵 تطبيق توصيل (Delivery)
- يتحقق من جاهزية التطبيق والمطعم.
- يفرض عنوان الشحن وطريقة الدفع.
- يحسب الشحن بناءً على المسافة ويطبّق كوبونات الشحن التلقائية.

### 🛍️ طلب بدون سلة (خدمات)
- يسمح بإدخال `expected_cart_total`.
- لا يتطلب `CartManager`.
- يعتمد على `hasAllowExpectedCartTotal()`.

### 🧾 طلب حجز (تذاكر طيران)
- يستدعي `setBooking()` لحقول مثل `flight_number`, `passport_number`.

### 🔄 إعادة تعيين المتجر تلقائياً (Auto Nerby Shop)
- في حالة الخروج عن نطاق التوصيل، يمكن استدعاء `setAutoNerbyShop()` لتحويل الطلب لأقرب فرع مفتوح.

### 🚫 منع العميل المحظور
- تنفيذ `validateCustomerIsBlockedOrders()` قبل أي خطوة.

---

## 🔚 خاتمة

يُشكّل `OrderManager` العمود الفقري لعملية الطلب في منصة Nano.soft. تصميمه القائم على الخطوات (Steps) يجعل من السهل تخصيص سير الطلب لكل نوع من أنواع الخدمات، مع إمكانية إضافة خطوات جديدة أو تعطيل القائم منها عبر إعدادات `OrdersType`. الدوال المجمّعة (`setStepDetails`, `setStepShiping`, `setStepCoupons`, `setStepPayments`) توفّر واجهة بسيطة للمطورين، بينما الدوال الفردية تمنحهم تحكّماً كاملاً عند الحاجة. التوثيق أعلاه يُغطّي كامل الوظائف المتاحة ويساعد على فهم أعمق للكلاس واستخدامه بكفاءة.

**الوثائق المرجعية**:
- [توثيق السمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-ar.md)
- [توثيق متقدم للسمة `StepStatus`](./docs/Orders/Traits/Steps/Docs-StepStatus-Trait-Advenced-ar.md)
- [توثيق نطاقات `HasProductOwnerScopes`](./docs/Orders/Models/orders/Docs-HasProductOwnerScopes-ar.md)
- [توثيق API الطلبات](./docs/OrdersApi/Docs-OrdersApi-ar.md)
