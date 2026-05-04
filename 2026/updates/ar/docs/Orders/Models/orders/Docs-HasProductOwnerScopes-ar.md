# توثيق نطاقات `HasProductOwnerScopes`  
**موديل** `Nano\Orders\Models\Order`  
**الهدف**: فلترة الطلبات بناءً على ملكية المنتجات المرتبطة بعناصر الطلب.

---

## 📋 نظرة عامة
تتيح لك هذه السلسلة من النطاقات (Scopes) استعلامات مرنة ومتقدمة على الطلبات بناءً على **مالك المنتج** (حقل `user_type` و `user_id` في جدول المنتجات `tss_inventory_products`). يمكنك تحديد المستخدم المستهدف (أو الاعتماد على المستخدم الحالي)، والتحكم بعدد المنتجات المملوكة، والنفي، وإضافة شروط إضافية على المنتج أو عنصر الطلب، بل ودمج الفلاتر باستخدام `OR` البوليني.

---

## 🧩 المكونات الرئيسية

| النطاق / الدالة | الوصف |
|----------------|--------|
| `scopeWhereHasProductsByOwner` | **النطاق الرئيسي** – يقبل مصفوفة خيارات شاملة. |
| `scopeWhereProductOwner` | اختصار بسيط للاستخدام السريع (وجود منتج واحد على الأقل). |
| `scopeHasProductsByOwner` | فلترة الطلبات التي تحتوي عددًا معينًا من المنتجات المملوكة. |
| `scopeDoesntHaveProductsByOwner` | فلترة الطلبات التي لا تحتوي أي منتج مملوك. |
| `scopeHasProductsByOwnerCount` | تحديد عدد دقيق (أو مقارن) للمنتجات المملوكة داخل الطلب. |
| `scopeOrWhereHasProductsByOwner` | ربط الفلتر بـ `OR` مع الشروط السابقة. |
| `scopeWithProductsByOwnerCount` | إضافة عمود افتراضي بعدد المنتجات المملوكة. |
| `scopeSortByProductsByOwnerCount` | ترتيب الطلبات حسب عدد المنتجات المملوكة تصاعديًا/تنازليًا. |
| `hasAnyProductByOwner` | دالة فحص على كائن `Order` مفرد (دون استعلام جديد). |
| `applyOwnerToProductQuery` | دالة مساعدة لتطبيق شرط المالك على استعلام فرعي. |

---

## 📘 النطاق الرئيسي: `scopeWhereHasProductsByOwner`

### البناء
```php
Order::whereHasProductsByOwner(array $options)
```

### جدول الخيارات

| المفتاح | النوع | الافتراضي | الوصف |
|--------|-------|-----------|--------|
| `user` | `Authenticatable\|Model\|null` | `null` | كائن المستخدم المالك. |
| `userType` | `string\|null` | `null` | نوع المستخدم (مثلاً `RainLab\User\Models\User`). |
| `userId` | `int\|null` | `null` | معرّف المستخدم. |
| `boolean` | `string` | `'and'` | طريقة الربط: `'and'` أو `'or'`. |
| `not` | `bool` | `false` | `true` لجلب الطلبات التي **لا** تحتوي المنتج المملوك. |
| `count` | `int\|null` | `null` | عدد المنتجات المملوكة المطلوب (يستخدم `has` بدلاً من `exists`). |
| `countOperator` | `string` | `'>=`' | عامل المقارنة عند استخدام `count` (مثل `=`, `<=`, `>`) |
| `conditions` | `array\|callable` | `[]` | شروط إضافية على **جدول المنتج** (`tss_inventory_products`). |
| `itemConditions` | `array\|callable` | `[]` | شروط إضافية على **جدول عناصر الطلب** (`nano_orders_items`). |

> **ملاحظة**: إذا لم تُمرّر `user` ولا `userType`/`userId`، يُحاول النطاق جلب المستخدم المسجّل حاليًا تلقائيًا.

---

## 🧪 أمثلة توضيحية

### المثال 1: طلبات تحتوي على أي منتج يملكه المستخدم الحالي
```php
$orders = Order::whereHasProductsByOwner([])->get();
// أو بشكل مختصر
$orders = Order::hasProductsByOwner()->get();
```

### المثال 2: طلبات تحتوي منتجًا مملوكًا لمستخدم محدد
```php
$user = \BackendAuth::getUser();
$orders = Order::whereHasProductsByOwner(['user' => $user])->get();
```

### المثال 3: طلبات لا تحتوي أي منتج مملوك للمستخدم
```php
$orders = Order::doesntHaveProductsByOwner($user)->get();
// أو باستخدام النفي الصريح
$orders = Order::whereHasProductsByOwner([
    'user' => $user,
    'not'  => true,
])->get();
```

### المثال 4: طلبات تحتوي بالضبط 3 منتجات مملوكة لمستخدم معين
```php
$orders = Order::hasProductsByOwnerCount($user, 3, '=')->get();
// أو باستخدام النطاق الرئيسي
$orders = Order::whereHasProductsByOwner([
    'user'  => $user,
    'count' => 3,
    'countOperator' => '='
])->get();
```

### المثال 5: شروط إضافية على المنتج نفسه
```php
// طلبات تحتوي منتجًا مملوكًا سعره > 100 وحالته "active"
$orders = Order::whereHasProductsByOwner([
    'user'       => $user,
    'conditions' => [
        'price'  => 100,
        'status' => 'active'
    ]
])->get();
```
أو باستخدام Closure:
```php
$orders = Order::whereHasProductsByOwner([
    'user'       => $user,
    'conditions' => function ($q) {
        $q->where('price', '>', 100)->where('status', 'active');
    }
])->get();
```

### المثال 6: شروط على عنصر الطلب معًا
```php
// الكمية > 2 من المنتج المملوك
$orders = Order::whereHasProductsByOwner([
    'user'           => $user,
    'itemConditions' => ['quantity' => 2]  // quantity > 2 ? ستحتاج closure
])->get();

// استخدام closure لـ itemConditions
$orders = Order::whereHasProductsByOwner([
    'user'           => $user,
    'itemConditions' => function ($q) {
        $q->where('quantity', '>', 2);
    }
])->get();
```

### المثال 7: دمج الفلاتر بـ OR
```php
// طلبات تحتوي منتجات مملوكة للمستخدم **أو** الطلبات الجديدة فقط
$orders = Order::isNewOrders()
    ->orWhereHasProductsByOwner(['user' => $user])
    ->get();
```

### المثال 8: إضافة عمود بعدد المنتجات المملوكة والترتيب به
```php
$orders = Order::withProductsByOwnerCount()
    ->sortByProductsByOwnerCount('desc')
    ->get();

foreach ($orders as $order) {
    echo $order->products_owner_count; // العمود المضاف
}
```

### المثال 9: فحص كائن طلب محدد
```php
$order = Order::find(5);
if ($order->hasAnyProductByOwner($user)) {
    // الطلب يحتوي على منتج واحد على الأقل يملكه المستخدم
}
```

---

## ⚠️ تنبيهات فنية
- **ثوابت الجداول**: النطاقات تستخدم `Product::TABLE_NAME` و `(new OrderItem)->getTable()` لضمان عدم تضارب الأسماء.
- **الاستعلامات الفرعية (Subqueries)**: عند استخدام `withProductsByOwnerCount` أو `sortByProductsByOwnerCount`، تُدمج bindings بشكل آمن عبر `orderByRaw` مع تمرير bindings.
- **أداء الاستعلام**: النطاقات تعتمد على `whereHas`/`whereDoesntHave` مما يولد استعلامات `EXISTS` أو `COUNT` وهي محسّنة للفهرسة، لكن مع الشروط الكثيرة قد تحتاج لمراقبة الأداء.
- **عدم تمرير مستخدم**: إذا لم يتم تمرير أي وسيط (user, userType, userId)، يحاول النطاق جلب المستخدم المسجل حالياً عبر `AuthHelpers` أو `BackendAuth` أو `Auth`. إذا لم يجد أي مستخدم، **لا يُطبّق أي فلتر** (يُعيد الاستعلام كما هو) تجنباً لنتائج غير متوقعة.

---

## 🧰 السيناريوهات المتقدمة

### تصفية طلبات تحتوي منتجات مملوكة + طلبات العميل نفسه
```php
$user = \BackendAuth::getUser();
$orders = Order::where('user_id', $user->id)
    ->orWhereHasProductsByOwner(['user' => $user])
    ->get();
```

### تقارير: عدد الطلبات التي تحتوي منتجين مملوكين على الأقل خلال الأسبوع الحالي
```php
$start = Carbon::now()->startOfWeek();
$orders = Order::where('created_at', '>=', $start)
    ->whereHasProductsByOwner([
        'user'  => $user,
        'count' => 2,
        'countOperator' => '>='
    ])
    ->count();
```

---

## 📦 خلاصة
باستخدام هذه النطاقات، أصبحت فلترة الطلبات بناءً على ملكية المنتجات أمرًا فائق المرونة، مع دعم كامل للنفي والعد والترتيب والدمج البوليني، مما يلبي احتياجات التقارير ولوحات التحكم المتقدمة داخل نظام نانوسوفت .

