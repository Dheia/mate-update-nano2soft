# توثيق سلوك `AssociationsModel`

**Namespace:** `Nano\Shop\Behaviors`  
**المساعد:** `Nano\Shop\Classes\AssociationsHelper`

---

## مقدمة

`AssociationsModel` هو **Behavior** (سلوك) مخصص لنماذج تطبيقات NanoSoft (مثل `Product`)، يوفر وظيفة **إدارة الارتباطات بين المنتجات** مثل:

- **المنتجات البديلة** (Alternate)
- **المنتجات المكملة** (Cross‑Sell)
- **الإصدارات الأحدث** (Up‑Sell)

بالإضافة إلى الوظائف الأساسية، يقدم السلوك **مجموعة متطورة من النطاقات (scopes)** تتيح:

- التصفية حسب **نوع العلاقة**، **اتجاه العلاقة** (أب، هدف، كليهما) و**وجود/عدم وجود** الارتباط.
- **شروط إضافية** على جدول الارتباطات (مثل `is_active`, `is_public`, `companys_id`, `departments_id`).
- **تطبيق نطاقات** خاصة بجدول الارتباطات (مثل `isCompany`, `isDepartment`).
- **تخصيص الشروط لكل اتجاه** على حدة (`conditionsPerDirection`, `scopesPerDirection`, `typePerDirection`).
- **التحكم الكامل في منطق الربط** (`AND` / `OR`) بين الشروط الأصلية وبين الاتجاهات المختلفة.

كما يوفر الكلاس المساعد `AssociationsHelper` أدوات جاهزة **لحقن فلاتر الارتباطات** في واجهات الإدارة (مثل قائمة المنتجات) بسهولة ومرونة.

---

## الإعداد والتثبيت

لا يتطلب السلوك تثبيتاً إضافياً، فهو جزء من حزمة `Nano.Shop`. لاستخدامه، تحتاج إلى إضافته كـ Behavior في النموذج المطلوب (مثلاً نموذج `Product`).

#### 1. إضافة السلوك إلى النموذج مباشرة (في ملف النموذج)

```php
class Product extends Model
{
    public $implement = [
        'Nano.Shop.Behaviors.AssociationsModel'
    ];

    // ... باقي تعريفات النموذج
}
```

#### 2. حقن السلوك في نموذج من إضافة أخرى (ديناميكياً)

```php
// في ملف boot الخاصة بالإضافة
public function boot()
{
    \Nano\Shop\Models\Product::extend(function($model) {
        if (!$model->isClassExtendedWith('Nano.Shop.Behaviors.AssociationsModel')) {
            $model->implementClassWith('Nano.Shop.Behaviors.AssociationsModel');
        }
    });
}
```

> **ملاحظة:** السلوك يضيف تلقائياً العلاقات `associations` (hasMany على `product_parent_id`) و `inverseAssociations` (hasMany على `product_target_id`).

---

## هيكل الجدول والعلاقات

السلوك يعتمد على جدول وسيط `nano_shop_product_associations` (موديل `ProductsAssociation`) الذي يحتوي على:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `id` | bigIncrements | المفتاح الأساسي |
| `type` | string(50) | نوع الارتباط: `cross_sell`, `up_sell`, `alternate` |
| `product_parent_id` | bigInteger | المنتج الأب في العلاقة |
| `product_target_id` | bigInteger | المنتج الهدف في العلاقة |
| `is_active` | boolean | هل الارتباط نشط (افتراضي true) |
| `is_public` | boolean | هل الارتباط عام (افتراضي true) |
| `companys_id` | string | معرف الشركة (للتعددية) |
| `departments_id` | string | معرف القسم (للتعددية) |
| `sort_order` | integer | ترتيب العرض |
| `created_at`, `updated_at`, `deleted_at` | timestamp | التواقيت |

**العلاقات المضافة تلقائياً للنموذج:**

- `associations` : `hasMany` – الارتباطات التي يكون النموذج فيها **أب**.
- `inverseAssociations` : `hasMany` – الارتباطات التي يكون النموذج فيها **هدف**.
- `associations_products` : `belongsToMany` – المنتجات المرتبطة كأبناء.
- `inverseAssociations_products` : `belongsToMany` – المنتجات التي ترتبط به كهدف.

---

## النطاقات (Scopes) المتوفرة

يوفر السلوك نطاقات متعددة تبدأ من الأبسط إلى الأكثر تعقيداً.

### 1. نطاقات جاهزة لأنواع محددة (بدون خيارات)

| النطاق | الوصف |
|--------|-------|
| `hasAlternate()` | المنتجات التي لها بدائل (في أي اتجاه) |
| `hasCrossSell()` | المنتجات التي لها مكملات |
| `hasUpSell()` | المنتجات التي لها إصدارات أحدث |
| `hasNotAlternate()` | المنتجات التي ليس لها بدائل |
| `hasNotCrossSell()` | المنتجات التي ليس لها مكملات |
| `hasNotUpSell()` | المنتجات التي ليس لها إصدارات أحدث |

### 2. نطاقات حسب الاتجاه (لكل نوع)

| النطاق | الوصف |
|--------|-------|
| `hasAlternateAsParent()` | المنتجات التي لها بدائل كأب |
| `hasAlternateAsTarget()` | المنتجات التي هي بدائل (كهدف) |
| `hasCrossSellAsParent()` | المنتجات التي لها مكملات كأب |
| `hasCrossSellAsTarget()` | المنتجات التي هي مكملات (كهدف) |
| `hasUpSellAsParent()` | المنتجات التي لها إصدارات أحدث كأب |
| `hasUpSellAsTarget()` | المنتجات التي هي إصدارات أحدث (كهدف) |

### 3. نطاقات لأي ارتباط (وجود/عدم وجود)

| النطاق | الوصف |
|--------|-------|
| `hasAnyAssociationBoth()` | المنتجات التي لها أي ارتباط (في أي اتجاه) |
| `hasNotAnyAssociationBoth()` | المنتجات التي ليس لها أي ارتباط |

### 4. النطاق المتقدم `scopeWhereAssociation`

هذا هو النطاق الأساسي الذي يتحكم في كل الوظائف. يمكن استدعاؤه مباشرة أو عبر النطاقات المساعدة. الصيغة:

```php
public function scopeWhereAssociation($query, $options = [], $boolean = 'and', $not = false)
```

**معامل `$options` (مصفوفة) تفصيلاً:**

| الخيار | النوع | الوصف |
|--------|-------|-------|
| `type` | string\|array\|null | نوع العلاقة (مثل `'alternate'`) أو مجموعة (`['alternate', 'cross_sell']`). |
| `direction` | string\|array | الاتجاه: `'parent'`, `'target'`, `'both'` أو مصفوفة اتجاهات. (افتراضي `'both'`) |
| `has` | bool | `true` = وجود الارتباط (whereHas), `false` = عدم وجود (whereDoesntHave). (افتراضي `true`) |
| `boolean` | string | `'and'` أو `'or'` لربط هذا الشرط مع الاستعلام الأصلي. (افتراضي `'and'`) |
| `relationOperator` | string\|null | `'and'` أو `'or'` للربط بين الاتجاهات المتعددة. إن لم يحدد: `'or'` إذا `has = true`، و `'and'` إذا `has = false`. |
| `callback` | callable\|null | دالة (استعلام العلاقة, اسم الاتجاه) لتخصيص إضافي لكل اتجاه. |
| `callbacks` | array | مصفوفة لكل اتجاه: `['parent' => callable, 'target' => callable]`. |
| `typePerDirection` | array | تخصيص نوع العلاقة لكل اتجاه: `['parent' => 'alternate', 'target' => 'cross_sell']`. |
| `conditions` | array\|callable\|null | شروط إضافية على جدول العلاقة (مثل `['is_active' => 1]`). |
| `scopes` | array | نطاقات لتطبيقها على استعلام العلاقة (مثل `['isCompany', 'isDepartment' => [1]]`). |
| `conditionsPerDirection` | array | شروط خاصة بكل اتجاه: `['parent' => ['is_active' => 1], 'target' => ['is_public' => 1]]`. |
| `scopesPerDirection` | array | نطاقات خاصة بكل اتجاه: `['parent' => ['isCompany'], 'target' => ['isDepartment' => [1]]]`. |

**معامل `$boolean`** – يحدد كيفية دمج هذا النطاق مع أي شروط سابقة في الاستعلام.

**معامل `$not`** – إذا كان `true`، يتم قلب `has` (أي يصبح `has = false`) ما لم يتم تمرير `has` صراحة في الخيارات.

### 5. النطاقات المساعدة

لتسهيل الاستخدام، يوفر السلوك دوال تغلف `scopeWhereAssociation`:

| النطاق | الوصف |
|--------|-------|
| `scopeOrWhereAssociation($query, $options, $not = false)` | دمج الشرط بـ `OR` مع الاستعلام الأصلي |
| `scopeWhereNotAssociation($query, $options, $boolean = 'and')` | شرط نفي (doesntHave) |
| `scopeOrWhereNotAssociation($query, $options)` | شرط نفي مع `OR` |
| `scopeHasAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | وجود ارتباط من نوع معين (اختصار) |
| `scopeHasNotAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | عدم وجود ارتباط من نوع معين |
| `scopeHasAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | وجود أي ارتباط (أي نوع) |
| `scopeHasNotAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | عدم وجود أي ارتباط |

> **ملاحظة:** الدوال المساعدة أعلاه لا تدعم الخيارات المتقدمة (`conditions`, `scopes`, `typePerDirection`, ...) مباشرة. إذا كنت بحاجة لهذه الخيارات، استخدم `scopeWhereAssociation` مباشرة.

### 6. نطاقات Toggle (خاصة بفلاتر الواجهة)

| النطاق | الوصف |
|--------|-------|
| `scopeIsToggelAnyAssociation($query, $value)` | `1` → منتجات لها أي ارتباط؛ `0` → منتجات بدون ارتباط |
| `scopeIsToggelAnyAlternate($query, $value)` | نفس المنطق للبدائل |
| `scopeIsToggelAnyCrossSell($query, $value)` | للمكملات |
| `scopeIsToggelAnyUpSell($query, $value)` | للإصدارات الأحدث |

---

## أمثلة عملية على النطاقات المتقدمة

### مثال 1: المنتجات التي لها ارتباطات بديلة **نشطة** (`is_active = 1`) في أي اتجاه

```php
Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => ['is_active' => 1]
])->get();
```

### مثال 2: المنتجات التي لها ارتباطات بديلة **كأب** فقط، مع شرط `is_public = 1`

```php
Product::whereAssociation([
    'type' => 'alternate',
    'direction' => 'parent',
    'conditions' => ['is_public' => 1]
])->get();
```

### مثال 3: المنتجات التي **ليس لها** أي ارتباط من نوع `cross_sell` في أي اتجاه، ولكنها قد تكون مرتبطة بأنواع أخرى

```php
Product::whereAssociation([
    'type' => 'cross_sell',
    'has' => false,
    'direction' => 'both'
])->get();
```

### مثال 4: المنتجات التي لها ارتباط من نوع `alternate` **كأب** **أو** ارتباط من نوع `up_sell` **كهدف** (باستخدام `typePerDirection`)

```php
Product::whereAssociation([
    'direction' => ['parent', 'target'],
    'typePerDirection' => [
        'parent' => 'alternate',
        'target' => 'up_sell'
    ],
    'relationOperator' => 'or'   // افتراضي أصلاً لأن has = true
])->get();
```

### مثال 5: المنتجات التي لها ارتباط في كلا الاتجاهين بنفس الوقت (AND بين الاتجاهين)

```php
Product::whereAssociation([
    'direction' => 'both',
    'relationOperator' => 'and',  // فرض AND
    'conditions' => ['is_active' => 1]
])->get();
```

### مثال 6: استخدام `scopes` على جدول العلاقة (مثل `isCompany` و `isDepartment`)

```php
Product::whereAssociation([
    'type' => 'alternate',
    'scopes' => ['isCompany', 'isDepartment' => [1]]
])->get();
```

### مثال 7: شروط خاصة بكل اتجاه باستخدام `conditionsPerDirection`

```php
Product::whereAssociation([
    'direction' => 'both',
    'conditionsPerDirection' => [
        'parent' => ['is_active' => 1],
        'target' => ['is_public' => 1]
    ]
])->get();
```

### مثال 8: دمج `scopesPerDirection` مع `typePerDirection`

```php
Product::whereAssociation([
    'direction' => 'both',
    'typePerDirection' => [
        'parent' => 'alternate',
        'target' => 'cross_sell'
    ],
    'scopesPerDirection' => [
        'parent' => ['isCompany'],
        'target' => ['isDepartment' => [1]]
    ]
])->get();
```

### مثال 9: استخدام `callback` مخصص لتعديل استعلام العلاقة

```php
Product::whereAssociation([
    'type' => 'alternate',
    'callback' => function($query, $direction) {
        $query->where('created_at', '>', now()->subDays(30));
    }
])->get();
```

### مثال 10: استخدام `orWhereAssociation` مع خيارات

```php
Product::where('price', '>', 100)
    ->orWhereAssociation([
        'type' => 'alternate',
        'conditions' => ['is_active' => 1]
    ])
    ->get();
```

### مثال 11: استخدام `whereNotAssociation` لنفي الارتباطات مع شروط

```php
Product::whereNotAssociation([
    'type' => 'cross_sell',
    'direction' => 'both',
    'conditions' => ['is_public' => 1]
])->get();
```

---

## الكلاس المساعد `AssociationsHelper`

يوجد في `Nano\Shop\Classes\AssociationsHelper` ويسهل إدارة فلاتر الارتباطات في واجهات الإدارة (`ListController` / `Filter`).

### الدوال الرئيسية

| الدالة | الوصف |
|--------|-------|
| `getAlternateFilterScopes($defaultValue = 0, $is_force = false)` | إرجاع مصفوفة تعريفات فلاتر البدائل (checkbox + toggle) |
| `getCrossSellFilterScopes(...)` | نفس الشيء للمكملات |
| `getUpSellFilterScopes(...)` | للإصدارات الأحدث |
| `getAnyAssociationFilterScopes(...)` | لفلتر "أي ارتباط" (switch) |
| `getAllAssociationFilterScopes(...)` | يجمع جميع الفلاتر السابقة |
| `injectAssociationScopes($filter, $types = null, $defaultValue = 0, $is_force = false)` | حقن الفلاتر المحددة في كائن الفلتر |
| `removeAssociationScopes($filter, $types = null)` | إزالة فلاتر معينة |
| `isTypeEnabled($type)` | التحقق من تفعيل نوع معين عبر الإعدادات |

**معامل `$types` في `injectAssociationScopes` و `removeAssociationScopes`:**  
- `null` أو `false`: لا يضيف شيئاً.
- `true`: يضيف جميع الأنواع (`any`, `alternate`, `cross_sell`, `up_sell`).
- نص مثل `'any,alternate'`: يضيف الأنواع المذكورة.
- مصفوفة: `['alternate', 'up_sell']`.

### مثال على استخدام المساعد في متحكم

```php
use Nano\Shop\Classes\AssociationsHelper;

public function boot()
{
    \ShopProductsController::extendListFilterScopes(function($filter) {
        $allowFilter = \Config::get('nano.shop::products.associations.allow_filter', false);
        if ($allowFilter && class_exists(AssociationsHelper::class)) {
            AssociationsHelper::injectAssociationScopes($filter, $allowFilter);
        }
    });
}
```

---

## ملخص

- **`AssociationsModel`** هو سلوك متكامل لإدارة ارتباطات المنتجات (بدائل، مكملات، إصدارات أحدث).
- يوفر نطاقات متعددة تبدأ من البسيطة (`hasAlternate()`) إلى المتقدمة التي تسمح بتمرير شروط إضافية (`conditions`, `scopes`) وتخصيص كل اتجاه (`typePerDirection`, `conditionsPerDirection`).
- النطاق الأساسي `whereAssociation` يتحكم في جميع الجوانب: نوع العلاقة، الاتجاه، وجود/عدم وجود، شروط على جدول الارتباطات، تطبيق نطاقات، وتحديد منطق الربط (`AND`/`OR`).
- **`AssociationsHelper`** يسهل دمج هذه الفلاتر في واجهات الإدارة، مما يوفر وقت التطوير ويضمن تناسق التجربة.

هذه الأدوات تجعل بناء أنظمة توصيات المنتجات وإدارة الارتباطات في المتاجر الإلكترونية عملية مرنة وسهلة الصيانة.