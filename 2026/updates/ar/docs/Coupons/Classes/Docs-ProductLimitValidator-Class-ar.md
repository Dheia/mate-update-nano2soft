# توثيق كلاس ProductLimitValidator

## 1. مقدمة

`ProductLimitValidator` هو كلاس مركزي ومتخصص ضمن إضافة `Nano.Coupons` (من تطوير نانوسوفت)، يعمل كجسر فحص ذكي بين إعدادات "حد المنتج" (`product_limit`) المُخزّنة في الكوبونات، وسلوك المستخدم الفعلي (السلة، الطلبات السابقة). يقوم الكلاس بتحميل القيود النشطة من قاعدة البيانات، والتحقق من تجاوز المستخدم للحدود المسموحة (كمية، عدد طلبات) مع مراعاة:
- منتج معيَّن (أو غيابه لفحص عام).
- وحدة منتج اختيارية.
- فترة زمنية محدّدة (عدد أيام).
- أنواع طلبات معينة (`order_restriction`).
- متاجر معينة (`shop_restriction`).

صُمم الكلاس ليكون قابلاً لإعادة الاستخدام في سياقات مختلفة (داخل شرط السلة `CartCondition`، أو استدعاء مباشر من الـ API، أو عمليات الخلفية)، مع تركيز كبير على الأداء عبر التخزين المؤقت (`Cache`) واستعلامات قاعدة البيانات المُحسَّنة.

## 2. الغرض والفوائد

- **مركزية فحص حدود المنتج:** بدلاً من تكرار كود الفحص في أماكن متعددة، يوفر الكلاس واجهة موحدة.
- **فصل المسؤوليات:** تم فصل منطق التحقق عن شرط السلة `ProductLimit` (CartCondition)، مما يتيح استخدام الكلاس بشكل مستقل.
- **أداء عالي:** دعم استراتيجيات جلب متعددة (`cache`, `fresh_db`, `direct_query`) مع إمكانية التحكم في مدة التخزين المؤقت.
- **رسائل خطأ دقيقة ومتعددة اللغات:** عند انتهاك الحدود، يرمي استثناءً برسالة قابلة للترجمة تشمل الحدود والقيم الحالية وتاريخ إمكانية الطلب التالي.
- **مرونة في الإعدادات:** يمكن تهيئة الكلاس بخيارات مثل `use_cache` و `cache_ttl` لتناسب بيئات مختلفة.

## 3. الدوال العامة (Public API)

### 3.1 الباني `__construct`

```php
public function __construct(User $user = null, array $options = [])
```

**الباراميترات:**
- `$user` (اختياري): كائن المستخدم (`RainLab\User\Models\User`) المطلوب فحص حدوده. إذا كان `null`، ستفشل كل عمليات الفحص (ترجع `true`) لأنه لا يمكن ربط القيود بمستخدم غير معروف.
- `$options` (مصفوفة اختيارية):
  - `use_cache` (bool): تفعيل/تعطيل التخزين المؤقت للقيود (افتراضي `true`).
  - `cache_ttl` (int): مدة بقاء القيود في الكاش بالثواني (افتراضي 300).

**مثال:**

```php
use Nano\Coupons\Classes\ProductLimitValidator;
use RainLab\User\Models\User;

$user = User::find(54);
$validator = new ProductLimitValidator($user);
// مع خيارات مخصصة
$validator2 = new ProductLimitValidator($user, ['use_cache' => false]);
```

### 3.2 `checkOrFail`

```php
public function checkOrFail(
    ?int $productId = null,
    ?int $unitId = null,
    int $quantity = 0,
    $defaultOrderTypes = null,
    ?int $defaultShopId = null
): bool
```

الغرض: فحص شامل للمنتج/الوحدة/الكمية وعدد الطلبات. إذا كان `productId` يساوي `null`، يتم فحص عدد الطلبات فقط بناءً على نوع الطلب والمتجر الممررين، وإلا يتم فحص المنتج حسب الحالتين.

**الباراميترات:**
- `$productId` (`int|null`): معرّف المنتج. إذا كان `null`، فسيتم تجاهل فحص المنتج والكمية والتركيز على عدد الطلبات.
- `$unitId` (`int|null`): معرّف الوحدة (اختياري). يستخدم فقط مع `$productId`.
- `$quantity` (`int`): الكمية المطلوب إضافتها للعربة.
- `$defaultOrderTypes` (`string|array|null`): نوع/أنواع الطلب الحالية (تُستخدم إذا لم تحدد القيود `order_restriction`).
- `$defaultShopId` (`int|null`): المتجر الحالي (يُستخدم إذا لم تحدد القيود `shop_ids`).

**الإرجاع:** `true` إذا لم يتم تجاوز أي حد.

**الاستثناءات:** يرمي `ApplicationException` عند تجاوز الكمية أو عدد الطلبات.

**مثال:**

```php
// فحص إضافة 3 قطع من المنتج 3918 لمتجر 10
$validator->checkOrFail(3918, null, 3, 'delivery', 10);
```

### 3.3 `check`

```php
public function check(
    ?int $productId = null,
    ?int $unitId = null,
    int $quantity = 0,
    $orderTypes = null,
    ?int $shopId = null
): bool
```

الغرض: مثل `checkOrFail` لكن بصمت (لا يرمي استثناءات)، بل يعيد `false` عند التجاوز.

**الباراميترات:** نفس `checkOrFail` (بدون إمكانية تجاوز القيم).

**مثال:**

```php
if(!$validator->check(3918, 5, 1, 'delivery', 10)){
    echo 'لا يمكن إضافة هذا المنتج';
}
```

### 3.4 `getLimits`

```php
public function getLimits(
    ?int $productId = null,
    ?int $unitId = null,
    $orderTypes = null,
    ?int $shopId = null,
    string $strategy = 'auto'
): array
```

الغرض: جلب مصفوفة القيود المطبقة بناءً على المعايير، دون إجراء فحص للكميات.

**الباراميترات:**
- `$productId`, `$unitId`, `$orderTypes`, `$shopId` – للتصفية.
- `$strategy` (`string`): استراتيجية الجلب.
  - `auto`: استخدام الكاش إذا مفعّل، وإلا استعلام مباشر.
  - `cache`: إجبار التحميل من الكاش.
  - `fresh_db`: تجاهل الكاش وجلب مباشر من القاعدة.
  - `direct_query`: استعلام مباشر بالشروط المحددة فقط (لا يحمل كل القيود).

**مثال:**

```php
// جلب كل القيود الخاصة بنوع الطلب delivery
$limits = $validator->getLimits(null, null, 'delivery', null, 'fresh_db');
```

### 3.5 `getLimitsForProduct`

```php
public function getLimitsForProduct(
    int $productId,
    ?int $unitId = null,
    ?string $orderType = null,
    ?int $shopId = null,
    string $strategy = 'auto'
): array
```

اختصار مفيد لـ `getLimits` عندما يكون لديك منتج محدد.

### 3.6 `checkOrderTypeLimitOrFail`

```php
public function checkOrderTypeLimitOrFail(
    $orderTypes,
    int $maxOrders,
    int $periodDays,
    ?int $shopId = null
): bool
```

الغرض: فحص عدد الطلبات لنوع طلب معين بغض النظر عن المنتج. تستخدم داخلياً `checkOrFail` بتمرير `productId=null`.

**مثال:**

```php
// هل تجاوز المستخدم 3 طلبات من نوع delivery خلال 7 أيام؟
$validator->checkOrderTypeLimitOrFail('delivery', 3, 7, 1);
```

### 3.7 `getOrderTypeLimits`

```php
public function getOrderTypeLimits(
    $orderTypes,
    ?int $shopId = null,
    string $strategy = 'auto'
): array
```

الغرض: جلب القيود التي تطابق نوع الطلب والمتجر فقط (بدون منتج).

### 3.8 `loadActiveLimits`

```php
public function loadActiveLimits(): array
```

تحميل كافة قيود `product_limit` النشطة من الكاش (أو قاعدة البيانات حسب الإعدادات).

### 3.9 `clearCache`

```php
public function clearCache(): void
```

مسح الكاش الخاص بالقيود (يُستدعى عادة عند حفظ كوبون جديد).

### 3.10 `setUser`

```php
public function setUser(User $user): self
```

تعيين المستخدم الحالي (يُستخدم إذا تغيَّر بعد إنشاء الكائن).

## 4. أمثلة عملية شاملة

### 4.1 منع إضافة منتج تجاوز الحد المسموح

```php
use Nano\Coupons\Classes\ProductLimitValidator;
use RainLab\User\Facades\Auth;

$user = Auth::getUser();
$validator = new ProductLimitValidator($user);

try {
    $validator->checkOrFail(
        $cartItem->id,          
        $cartItem->units_id,   
        $cartItem->qty,       
        $this->getCurrentOrderType(),
        $this->getCurrentShopId()
    );
    // السماح بالإضافة
} catch (\ApplicationException $e) {
    // عرض رسالة الخطأ
    return $e->getMessage();
}
```

### 4.2 جلب قيود منتج معين وعرضها

```php
$limits = $validator->getLimitsForProduct(3918);
foreach($limits as $limit){
    echo "الحد الأقصى للكمية: " . ($limit['max_quantity'] ?: 'غير محدد');
    echo "الحد الأقصى للطلبات: " . ($limit['max_orders'] ?: 'غير محدد');
}
```

### 4.3 فحص عدد طلبات نوع delivery في متجر معين

```php
$allowed = $validator->check(null, null, 0, 'delivery', 10);
if(!$allowed){
    // إجراء منع إتمام الطلب
}
```

## 5. ملاحظات حول الأداء والتصميم

- **الكاش:** يُستخدم بشكل افتراضي لتجنب تحميل جميع القيود من قاعدة البيانات في كل طلب. يتم مسح الكاش تلقائياً عند حفظ أي كوبون من نوع `product_limit` (في `afterSave` داخل `Coupons_model`).
- **الاستعلامات المجمعة:** دوال `getBulkHistoricalQuantities` و `getBulkHistoricalOrderCounts` تجمع بيانات جميع المنتجات المطلوبة في استعلام واحد، مما يقلل الضغط على قاعدة البيانات.
- **الفصل بين السيناريوهات:** تم فصل الحالتين (منتج محدد / بدون منتج) بشكل واضح داخل `checkOrFail` لتسهيل القراءة والصيانة.

## 6. التكامل مع إضافات نانوسوفت

- **`Nano.Cart`:** يُستخدم `ProductLimitValidator` داخل شرط `ProductLimit` الذي يستمع للحدث `nano.cart.adding` ويمنع إضافة الصنف فورياً.
- **`Nano.Orders`:** يتم الاستعلام عن الطلبات السابقة للمستخدم من جدول `nano_orders_orders` وعناصرها من `nano_orders_items`، مع استبعاد الطلبات الملغية وتلك في حالة السلة.
- **`Nano.Shop`:** يتم ربط القيود بالمنتج عبر `product_id` واختيارياً `units_id` باستخدام دوال جلب خيارات الوحدة من `Tss.Inventory`.

## 7. الخلاصة والتوصيات

- استخدم `checkOrFail` للفحص الفوري عند إضافة منتج، حيث يرمي استثناء برسالة واضحة يمكن عرضها للمستخدم.
- استخدم `check` للفحص الصامت عندما لا ترغب في التعامل مع الاستثناءات.
- استفد من استراتيجية `'direct_query'` عندما تريد فحص منتج واحد بدون تحميل جميع القيود من الكاش.
- قم بمسح الكاش (`clearCache`) يدوياً فقط في حالات نادرة؛ النظام يقوم بمسحه تلقائياً عند حفظ كوبون `product_limit`.
- لتخصيص مدة التخزين المؤقت، مرر `cache_ttl` عند إنشاء الكائن.

باستخدام `ProductLimitValidator`، يمكنك بناء منطق قوي ومرن للتحقق من حدود المنتجات في تطبيقك، مع ضمان أداء ممتاز وتجربة مستخدم سلسة.
