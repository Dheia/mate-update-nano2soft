## توثيق سلوك `AssociationsModel` والكلاس المساعد `AssociationsHelper`

**Namespace:** `Nano\Shop\Behaviors`  
**Namespace المساعد:** `Nano\Shop\Classes`

---

### مقدمة

`AssociationsModel` هو **Behavior** (سلوك) مخصص لنماذج تطبيقات NanoSoft، يوفر وظيفة **إدارة الارتباطات بين المنتجات** مثل:
- **المنتجات البديلة** (Alternate)
- **المنتجات المكملة** (Cross‑Sell)
- **الإصدارات الأحدث** (Up‑Sell)

بالإضافة إلى ذلك، يتيح السلوك مجموعة واسعة من النطاقات (scopes) للفلترة والبحث عن المنتجات بناءً على هذه الارتباطات. كما يوفر الكلاس المساعد `AssociationsHelper` أدوات جاهزة لحقن فلاتر هذه الارتباطات في واجهات الإدارة (مثل قائمة المنتجات) بسهولة ومرونة.

الهدف الرئيسي هو تمكين المطورين من:
- إنشاء علاقات بين المنتجات (أب ← هدف) من أي نوع.
- الاستعلام بمرونة عن المنتجات التي لها (أو ليس لها) ارتباطات معينة.
- التحكم في اتجاه العلاقة (كأب، كهدف، أو كليهما).
- دمج شروط الارتباط مع شروط أخرى (`and` / `or`).
- إضافة فلاتر جاهزة إلى واجهات الإدارة لتصفية المنتجات حسب حالة الارتباط.

---

### الإعداد والتثبيت

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

يمكنك إضافة السلوك إلى أي نموذج دون تعديل ملفه الأصلي باستخدام دوال التوسيع المتوفرة في إطار NanoSoft:

```php
// في ملف الإضافة الخاص بك
public function boot()
{
    \Nano\Shop\Models\Product::extend(function($model) {
        if (!$model->isClassExtendedWith('Nano.Shop.Behaviors.AssociationsModel')) {
            $model->implementClassWith('Nano.Shop.Behaviors.AssociationsModel');
        }
    });
}
```

> **ملاحظة:** السلوك يعتمد على علاقات معرفة مسبقاً (`hasMany` باسم `associations` و `inverseAssociations`). هذه العلاقات يتم تعريفها تلقائياً داخل البناء (`__construct`) للسلوك، لذا لا تحتاج لإضافتها يدوياً.

---

### هيكل الجدول والعلاقات

السلوك يعتمد على جدول وسيط (pivot) هو `nano_shop_product_associations` الذي يحتوي على:
- `id`
- `product_parent_id` (المنتج الأب)
- `product_target_id` (المنتج الهدف)
- `type` (نوع الارتباط: `cross_sell`, `up_sell`, `alternate`)
- `is_active`, `is_default`, `sort_order` (اختيارية)
- `created_at`, `updated_at`, `deleted_at`

العلاقات المضافة تلقائياً للنموذج:
- `associations` : علاقة `hasMany` للحصول على الارتباطات التي يكون النموذج فيها **أب**.
- `inverseAssociations` : علاقة `hasMany` للحصول على الارتباطات التي يكون النموذج فيها **هدف**.

بالإضافة إلى علاقات `belongsToMany` مساعدة للوصول المباشر إلى المنتجات المرتبطة (إذا احتجت).

---

### النطاقات (Scopes) المتوفرة في السلوك

يوفر السلوك مجموعة شاملة من النطاقات للاستعلام عن المنتجات حسب ارتباطاتها.

#### 1. نطاقات أساسية لأنواع الارتباط (بدون اتجاه محدد)

| النطاق | الوصف |
|--------|-------|
| `scopeHasAlternate($query)` | المنتجات التي لديها ارتباطات بديلة (في أي اتجاه) |
| `scopeHasCrossSell($query)` | المنتجات التي لديها ارتباطات مكملة |
| `scopeHasUpSell($query)` | المنتجات التي لديها إصدارات أحدث |
| `scopeHasNotAlternate($query)` | المنتجات التي ليس لديها ارتباطات بديلة |
| `scopeHasNotCrossSell($query)` | المنتجات التي ليس لديها ارتباطات مكملة |
| `scopeHasNotUpSell($query)` | المنتجات التي ليس لديها إصدارات أحدث |

#### 2. نطاقات حسب الاتجاه

| النطاق | الوصف |
|--------|-------|
| `scopeHasAlternateAsParent($query)` | المنتجات التي لديها بدائل كأب |
| `scopeHasAlternateAsTarget($query)` | المنتجات التي هي بدائل لمنتجات أخرى (كهدف) |
| `scopeHasCrossSellAsParent($query)` | المنتجات التي لديها مكملات كأب |
| `scopeHasCrossSellAsTarget($query)` | المنتجات التي هي مكملات لمنتجات أخرى |
| `scopeHasUpSellAsParent($query)` | المنتجات التي لديها إصدارات أحدث كأب |
| `scopeHasUpSellAsTarget($query)` | المنتجات التي هي إصدارات أحدث لمنتجات أخرى |

#### 3. نطاقات لأي ارتباط (وجود أو عدم وجود)

| النطاق | الوصف |
|--------|-------|
| `scopeHasAnyAssociationBoth($query)` | المنتجات التي لها أي ارتباط (في أي اتجاه) |
| `scopeHasNotAnyAssociationBoth($query)` | المنتجات التي ليس لها أي ارتباط |

#### 4. نطاقات متقدمة باستخدام `scopeWhereAssociation`

هذا هو النطاق الأقوى الذي يسمح بتمرير مصفوفة خيارات مفصلة:

```php
scopeWhereAssociation($query, $options = [], $boolean = 'and', $not = false)
```

**الخيارات (مصفوفة):**
- `type` : `string|array|null` – نوع العلاقة (alternate, cross_sell, up_sell) أو مجموعة منها.
- `direction` : `string|array|null` – الاتجاه: `'parent'`، `'target'`، `'both'` (افتراضي both) أو مصفوفة اتجاهات.
- `callback` : `callable|null` – دالة إضافية لتخصيص استعلام العلاقة.
- `has` : `bool` – `true` للوجود، `false` للنفي (عادة ما يُستنتج من `$not`).

**المعلمات الإضافية:**
- `$boolean` : `'and'` أو `'or'` لدمج هذا الشرط مع شروط سابقة.
- `$not` : `true` لتحويل الشرط إلى نفي (doesntHave).

#### 5. نطاقات مساعدة لـ OR و NOT

| النطاق | الوصف |
|--------|-------|
| `scopeOrWhereAssociation($query, $options = [], $not = false)` | شرط `orWhere` باستخدام نفس الخيارات |
| `scopeWhereNotAssociation($query, $options = [], $boolean = 'and')` | شرط نفي (doesntHave) |
| `scopeOrWhereNotAssociation($query, $options = [])` | شرط `orWhere` مع نفي |

#### 6. نطاقات عامة لسهولة الاستخدام

| النطاق | الوصف |
|--------|-------|
| `scopeHasAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | وجود ارتباط من نوع معين |
| `scopeHasNotAssociation($query, $type, $direction = 'both', $callback = null, $boolean = 'and')` | عدم وجود ارتباط من نوع معين |
| `scopeHasAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | وجود أي ارتباط (أي نوع) |
| `scopeHasNotAnyAssociation($query, $direction = 'both', $callback = null, $boolean = 'and')` | عدم وجود أي ارتباط |

#### 7. نطاقات Toggle (خاصة بفلاتر الواجهة)

| النطاق | الوصف |
|--------|-------|
| `scopeIsToggelAnyAssociation($query, $value)` | تبديل: إذا `1` يرجع المنتجات ذات أي ارتباط، إذا `0` يرجع المنتجات بدون ارتباط |
| `scopeIsToggelAnyAlternate($query, $value)` | تبديل للبدائل |
| `scopeIsToggelAnyCrossSell($query, $value)` | تبديل للمكملات |
| `scopeIsToggelAnyUpSell($query, $value)` | تبديل للإصدارات الأحدث |

---

### أمثلة عملية على استخدام النطاقات

#### 1. الحصول على جميع المنتجات التي لها بدائل

```php
$products = Product::hasAlternate()->get();
```

#### 2. الحصول على المنتجات التي هي بدائل (كهدف)

```php
$products = Product::hasAlternateAsTarget()->get();
```

#### 3. المنتجات التي لها ارتباطات مكملة **كأب** فقط

```php
$products = Product::hasCrossSellAsParent()->get();
```

#### 4. المنتجات التي ليس لها أي ارتباط على الإطلاق

```php
$products = Product::hasNotAnyAssociationBoth()->get();
```

#### 5. استخدام `whereAssociation` للبحث عن منتجات لها بدائل أو مكملات

```php
$products = Product::whereAssociation([
    'type' => ['alternate', 'cross_sell'],
    'direction' => 'both'
])->get();
```

#### 6. الجمع بين شروط متعددة مع `orWhereAssociation`

```php
$products = Product::where('is_active', true)
    ->orWhereAssociation(['type' => 'up_sell', 'direction' => 'parent'])
    ->get();
```

#### 7. استخدام callback لتخصيص شروط العلاقة (مثلاً الارتباطات النشطة فقط)

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'direction' => 'both',
    'callback' => function ($relationQuery) {
        $relationQuery->where('is_active', true);
    }
])->get();
```

#### 8. البحث عن المنتجات التي ليس لها بدائل (باستخدام `hasNotAssociation`)

```php
$products = Product::hasNotAssociation('alternate')->get();
```

#### 9. استخدام `whereNotAssociation` للنفي مع اتجاه محدد

```php
$products = Product::whereNotAssociation([
    'type' => 'cross_sell',
    'direction' => 'parent'
])->get();
```

#### 10. دمج شروط متعددة مع نفي وجمعها بـ OR

```php
$products = Product::where('price', '>', 100)
    ->orWhereNotAssociation(['type' => 'up_sell', 'direction' => 'target'])
    ->get();
```

#### 11. استخدام نطاقات toggle في الاستعلامات البرمجية

```php
// إذا أردت المنتجات التي لها أي ارتباط
$products = Product::isToggelAnyAssociation(1)->get();

// إذا أردت المنتجات التي ليس لها أي ارتباط
$products = Product::isToggelAnyAssociation(0)->get();
```

#### 12. الحصول على المنتجات التي لها بدائل مع ترتيب حسب السعر

```php
$products = Product::hasAlternate()->orderBy('price', 'desc')->get();
```

#### 13. استخدام `scopeHasAssociation` مع callback

```php
$products = Product::hasAssociation('cross_sell', 'both', function($q) {
    $q->where('created_at', '>', now()->subDays(30));
})->get();
```

---

### الكلاس المساعد `AssociationsHelper`

الكلاس `AssociationsHelper` موجود في `Nano\Shop\Classes\AssociationsHelper` ويوفر دوال ثابتة لإدارة فلاتر الارتباطات في واجهات الإدارة (مثل `ListController`). الهدف منه هو تبسيط عملية إضافة أو إزالة فلاتر المنتجات حسب أنواع الارتباطات.

#### 1. دوال إرجاع مصفوفات الفلاتر

| الدالة | الوصف |
|--------|-------|
| `getAlternateFilterScopes($defaultValue = 0, $is_force = false)` | مصفوفة فلاتر البدائل (checkboxes + toggle) |
| `getCrossSellFilterScopes($defaultValue = 0, $is_force = false)` | فلاتر المكملات |
| `getUpSellFilterScopes($defaultValue = 0, $is_force = false)` | فلاتر الإصدارات الأحدث |
| `getAnyAssociationFilterScopes($defaultValue = 0, $is_force = false)` | فلاتر "أي ارتباط" (وجود / عدم وجود) |
| `getAllAssociationFilterScopes($defaultValue = 0, $is_force = false)` | جميع الفلاتر السابقة مجتمعة |

#### 2. دوال حقن الفلاتر في أداة الفلتر

```php
injectAssociationScopes($filter, $types = null, $defaultValue = 0, $is_force = false)
```

- `$filter` : كائن الفلتر (عادة ما يكون من نوع `Nano\Backend\Widgets\Filter`).
- `$types` : يحدد أي أنواع الفلاتر تريد إضافتها. يمكن أن يكون:
  - `null` أو `false` : لا يضف شيئاً.
  - `true` : يضف جميع الفلاتر (`any`, `alternate`, `cross_sell`, `up_sell`).
  - نص مثل `'any,alternate'` : يضف الأنواع المذكورة.
  - مصفوفة مثل `['alternate', 'up_sell']`.
- `$defaultValue` : القيمة الافتراضية للفلاتر (0 يعني غير مفعل).
- `$is_force` : للاستخدام المستقبلي.

#### 3. دوال إزالة الفلاتر

```php
removeAssociationScopes($filter, $types = null)
```

- تحذف الفلاتر المحددة (`$types` بنفس منطق `injectAssociationScopes`).

#### 4. دالة التحقق من التفعيل

```php
isTypeEnabled($type)
```

- تعتمد على إعدادات config (`nano.shop::products.associations.allow_filter`) لتقرير إذا كان نوع معين مفعلاً.

---

### استخدام `AssociationsHelper` في الإضافات

```php
use Nano\Shop\Classes\AssociationsHelper;

// في دالة boot الخاصة بالإضافة
public function boot()
{
    // يمكنك حقن الفلاتر في متحكم المنتجات
    \ShopProductsController::extendListFilterScopes(function($filter) {
        $allowFilter = \Config::get('nano.shop::products.associations.allow_filter', false);
        if ($allowFilter && class_exists(AssociationsHelper::class)) {
            AssociationsHelper::injectAssociationScopes($filter, $allowFilter);
        }

        // مثال: إزالة فلاتر معينة حسب إعدادات أخرى
        if (!\Config::get('nano.shop::allow_alternate_filter', true)) {
            AssociationsHelper::removeAssociationScopes($filter, 'alternate');
        }
    });
}
```

### مثال لإعدادات config.php

```php
// config.php
'products' => [
    'associations' => [
        // يمكن أن يكون true (كل الفلاتر)، false (بدون فلاتر)، أو نص مثل 'any,alternate'
        'allow_filter' => env('NANO_SHOP_PRODUCTS_ASSOCIATIONS_ALLOW_FILTER', false),
    ],
],
```

---

### دمج كل شيء معاً: مثال متكامل

لنفترض أن لديك متحكم `Products` وتريد عرض المنتجات التي لها بدائل، مع إمكانية التبديل بين العرض حسب وجود أي ارتباط.

**في المتحكم:**
```php
$query = Product::query();

// تطبيق فلتر من المستخدم
if ($hasAlternate = input('has_alternate')) {
    $query->hasAlternate();
}

if ($toggleAny = input('toggle_any')) {
    $query->isToggelAnyAssociation($toggleAny);
}

$products = $query->paginate(20);
```

**في ملف الفلاتر (مثلاً `config_filter.yaml`):**
```yaml
scopes:
    has_alternate:
        label: 'لديه بدائل'
        type: checkbox
        scope: hasAlternate
        default: 0
    is_toggel_any_association:
        label: 'حالة الارتباط'
        type: switch
        scope: isToggelAnyAssociation
        default: 0
```

---

### الخلاصة

- **`AssociationsModel`** هو سلوك قوي ومرن لإدارة واستعلام الارتباطات بين المنتجات.
- يوفر نطاقات متعددة تناسب معظم حالات الاستخدام، من البسيط إلى المعقد.
- النطاق `whereAssociation` يعطي تحكماً كاملاً عبر مصفوفة خيارات.
- **`AssociationsHelper`** يسهل دمج فلاتر هذه الارتباطات في واجهات الإدارة بسرعة.
- التصميم يدعم التوسع بسهولة (إضافة أنواع جديدة من الارتباطات).

باستخدام هذه الأدوات، يمكن للمطورين بناء أنظمة متاجر إلكترونية متكاملة مع إدارة ذكية للارتباطات بين المنتجات، وتحسين تجربة المستخدم من خلال التوصيات (cross‑sell, up‑sell) وتوفير بدائل مناسبة.