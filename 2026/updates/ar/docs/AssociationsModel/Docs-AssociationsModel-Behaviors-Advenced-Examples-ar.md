# أمثلة عملية متقدمة لسلوك `AssociationsModel` والمساعد `AssociationsHelper`

**ملاحظة:** قبل البدء، تأكد من أن السلوك مضاف لنموذج `Product` وأن جداول الارتباطات موجودة كما هو موضح في التوثيق الأساسي.

---

## المحتويات

1. [إعداد بيانات الارتباطات](#إعداد-بيانات-الارتباطات)
2. [الاستعلامات الأساسية (وجود/عدم وجود)](#الاستعلامات-الأساسية-وجودعدم-وجود)
3. [الاستعلامات المتقدمة باستخدام `whereAssociation`](#الاستعلامات-المتقدمة-باستخدام-whereassociation)
   - [شروط إضافية (conditions)](#شروط-إضافية-conditions)
   - [تطبيق نطاقات (scopes) على جدول الارتباطات](#تطبيق-نطاقات-scopes-على-جدول-الارتباطات)
   - [تخصيص الشروط لكل اتجاه (conditionsPerDirection)](#تخصيص-الشروط-لكل-اتجاه-conditionsperdirection)
   - [نوع مختلف لكل اتجاه (typePerDirection)](#نوع-مختلف-لكل-اتجاه-typeperdirection)
   - [التحكم في منطق الربط (relationOperator)](#التحكم-في-منطق-الربط-relationoperator)
   - [استخدام callback مخصص](#استخدام-callback-مخصص)
   - [الدمج مع شروط أخرى (AND/OR)](#الدمج-مع-شروط-أخرى-andor)
4. [النفي والاستعلامات السلبية](#النفي-والاستعلامات-السلبية)
5. [أمثلة على استخدام `AssociationsHelper` في واجهات الإدارة](#أمثلة-على-استخدام-associationshelper-في-واجهات-الإدارة)
   - [حقن جميع فلاتر الارتباطات](#حقن-جميع-فلاتر-الارتباطات)
   - [حقن فلاتر محددة فقط](#حقن-فلاتر-محددة-فقط)
   - [إزالة فلاتر معينة](#إزالة-فلاتر-معينة)
   - [استخدام القيم الافتراضية والتبديل](#استخدام-القيم-الافتراضية-والتبديل)
6. [سيناريوهات متكاملة (واقعية)](#سيناريوهات-متكاملة-واقعية)
   - [عرض توصيات لمنتج معين (Cross‑Sell)](#عرض-توصيات-لمنتج-معين-cross‑sell)
   - [عرض بدائل للمنتج (Alternate)](#عرض-بدائل-للمنتج-alternate)
   - [إدارة العلاقات في لوحة التحكم (Backend)](#إدارة-العلاقات-في-لوحة-التحكم-backend)
   - [تحديث أو حذف ارتباطات بالجوبات (Jobs)](#تحديث-أو-حذف-ارتباطات-بالجوبات-jobs)

---

## إعداد بيانات الارتباطات

لنفترض أن لدينا المنتجات التالية:

- **P1**: لاب توب (id=1)
- **P2**: حقيبة لاب توب (id=2)
- **P3**: ماوس (id=3)
- **P4**: لاب توب بديل (نفس المواصفات) (id=4)
- **P5**: لاب توب أحدث موديل (id=5)

سننشئ الارتباطات:

```php
$product1 = Product::find(1);
$product2 = Product::find(2);
$product3 = Product::find(3);
$product4 = Product::find(4);
$product5 = Product::find(5);

// P1 لديه مكملات: حقيبة (cross_sell)
$product1->associate($product2, 'cross_sell');

// P1 لديه بديل: P4 (alternate)
$product1->associate($product4, 'alternate');

// P1 لديه إصدار أحدث: P5 (up_sell)
$product1->associate($product5, 'up_sell');

// P3 (ماوس) هو مكمل لـ P1 (لكن قد لا نريد إضافته في هذا المثال)
// يمكننا إنشاء ارتباط عكسي: P1 هو هدف لـ P2 (إذا أردنا ذلك)
$product2->associate($product1, 'cross_sell'); // P2 كأب و P1 هدف
```

**ملاحظة:** الارتباطات ثنائية الاتجاه ليست ضرورية، لكنها ممكنة.

---

## الاستعلامات الأساسية (وجود/عدم وجود)

### 1. المنتجات التي لها بدائل (في أي اتجاه)

```php
$products = Product::hasAlternate()->get();
// النتيجة: P1 (لديه بديل), وأي منتج آخر هو بديل لآخر (مثل P4 إذا كان هدفاً لـ P1)
```

### 2. المنتجات التي هي بدائل (كهدف فقط)

```php
$alternatives = Product::hasAlternateAsTarget()->get();
// النتيجة: P4 (لأنه هدف في علاقة بديلة)
```

### 3. المنتجات التي ليس لها أي ارتباط على الإطلاق

```php
$isolated = Product::hasNotAnyAssociationBoth()->get();
// النتيجة: P3 (إذا لم يكن له أي ارتباط)
```

### 4. المنتجات التي لها مكملات كأب (تبيع معاً)

```php
$crossSellParents = Product::hasCrossSellAsParent()->get();
// النتيجة: P1 (لديه P2 كمكمل), P2 (لديه P1 كمكمل) إذا قمنا بإضافة العلاقة العكسية
```

---

## الاستعلامات المتقدمة باستخدام `whereAssociation`

### شروط إضافية (conditions)

نريد المنتجات التي لها ارتباطات **نشطة** (`is_active = 1`) من نوع `alternate`.

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => ['is_active' => 1]
])->get();
```

إذا أردنا التأكد من أن `is_public` أيضاً = 1:

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => ['is_active' => 1, 'is_public' => 1]
])->get();
```

**شروط باستخدام `callable`:**

```php
$products = Product::whereAssociation([
    'type' => 'alternate',
    'conditions' => function($q) {
        $q->where('is_active', 1)
          ->where('created_at', '>', now()->subMonths(3));
    }
])->get();
```

### تطبيق نطاقات (scopes) على جدول الارتباطات

نطاقات مثل `isCompany` و `isDepartment` معرفة في موديل `ProductsAssociation`، يمكننا تطبيقها:

```php
// المنتجات التي لها ارتباطات بديلة وتنتمي للشركة الحالية
$products = Product::whereAssociation([
    'type' => 'alternate',
    'scopes' => ['isCompany']
])->get();

// تمرير معاملات للنطاق
$products = Product::whereAssociation([
    'type' => 'alternate',
    'scopes' => ['isDepartment' => 5]  // departments_id = 5
])->get();
```

### تخصيص الشروط لكل اتجاه (conditionsPerDirection)

نريد المنتجات التي لها ارتباطات من نوع `cross_sell` كأب **نشطة**، أو ارتباطات من نوع `cross_sell` كهدف **عامة** (`is_public = 1`).

```php
$products = Product::whereAssociation([
    'direction' => 'both',
    'type' => 'cross_sell',
    'conditionsPerDirection' => [
        'parent' => ['is_active' => 1],
        'target' => ['is_public' => 1]
    ]
])->get();
```

### نوع مختلف لكل اتجاه (typePerDirection)

نريد المنتجات التي لها:
- بدائل (`alternate`) كأب **أو**
- مكملات (`cross_sell`) كهدف.

```php
$products = Product::whereAssociation([
    'direction' => ['parent', 'target'],
    'typePerDirection' => [
        'parent' => 'alternate',
        'target' => 'cross_sell'
    ],
    'relationOperator' => 'or' // افتراضي، يمكن حذفه
])->get();
```

### التحكم في منطق الربط (relationOperator)

نريد المنتجات التي لها ارتباطات **في كلا الاتجاهين** بنفس الوقت (أي أن المنتج هو أب لارتباط ما **و** هدف لارتباط آخر).  
هذا يمكن أن يكون مفيداً لمعرفة المنتجات التي هي وسيطة في سلسلة توصيات.

```php
$products = Product::whereAssociation([
    'direction' => 'both',
    'relationOperator' => 'and',   // AND بين الاتجاهين
    'conditions' => ['is_active' => 1]
])->get();
```

### استخدام callback مخصص

نريد تخصيص استعلام العلاقة حسب الاتجاه:

```php
$products = Product::whereAssociation([
    'type' => 'cross_sell',
    'callback' => function($query, $direction) {
        if ($direction === 'parent') {
            $query->where('sort_order', '>', 0);
        } else {
            $query->where('is_public', 1);
        }
    }
])->get();
```

### الدمج مع شروط أخرى (AND/OR)

نريد المنتجات التي يزيد سعرها عن 100 **و** لها بدائل.

```php
$products = Product::where('price', '>', 100)
    ->whereAssociation(['type' => 'alternate'])
    ->get();
```

نريد المنتجات التي يقل سعرها عن 50 **أو** ليس لها أي ارتباط.

```php
$products = Product::where('price', '<', 50)
    ->orWhereNotAssociation(['type' => 'alternate', 'direction' => 'both'])
    ->get();
```

---

## النفي والاستعلامات السلبية

### المنتجات التي ليس لها ارتباطات من نوع `up_sell` في أي اتجاه

```php
$products = Product::whereNotAssociation(['type' => 'up_sell'])->get();
```

### المنتجات التي ليس لها ارتباطات من نوع `alternate` كأب

```php
$products = Product::whereNotAssociation([
    'type' => 'alternate',
    'direction' => 'parent'
])->get();
```

### استخدام `hasNotAssociation`

```php
$products = Product::hasNotAssociation('alternate', 'both');
```

### المنتجات التي ليس لها ارتباطات **نشطة** من نوع `cross_sell` (شرط سلبي)

```php
$products = Product::whereNotAssociation([
    'type' => 'cross_sell',
    'conditions' => ['is_active' => 1]
])->get();
```

---

## أمثلة على استخدام `AssociationsHelper` في واجهات الإدارة

### حقن جميع فلاتر الارتباطات

في متحكم المنتجات (مثلاً `ShopProductsController`)، نضيف الفلاتر عند تهيئة القائمة:

```php
use Nano\Shop\Classes\AssociationsHelper;

public function listFilterScopes()
{
    $scopes = parent::listFilterScopes();

    // إضافة فلاتر الارتباطات
    if (class_exists(AssociationsHelper::class)) {
        $associationScopes = AssociationsHelper::getAllAssociationFilterScopes();
        $scopes = array_merge($scopes, $associationScopes);
    }

    return $scopes;
}
```

أو يمكن حقنها عبر الحدث `backend.filter.extendScopes`:

```php
\Event::listen('backend.filter.extendScopes', function ($filterWidget, $model, $scopes) {
    if ($model instanceof \Nano\Shop\Models\Product) {
        AssociationsHelper::injectAssociationScopes($filterWidget, true);
    }
});
```

### حقن فلاتر محددة فقط

```php
AssociationsHelper::injectAssociationScopes($filter, 'alternate,cross_sell');
```

أو:

```php
AssociationsHelper::injectAssociationScopes($filter, ['alternate', 'up_sell']);
```

### إزالة فلاتر معينة

```php
AssociationsHelper::removeAssociationScopes($filter, 'any');
```

### استخدام القيم الافتراضية والتبديل

عند حقن الفلاتر، يمكن تحديد قيمة افتراضية (مثلاً `1` لتشغيل فلتر "أي ارتباط"):

```php
AssociationsHelper::injectAssociationScopes($filter, 'any', 1);
```

**التبديل عبر واجهة المستخدم:**  
فلتر `is_toggel_any_association` من نوع `switch`، يعكس قيمته (`1` → منتجات لها ارتباط، `0` → منتجات بدون ارتباط). هذا يعمل مباشرة مع النطاق المضمن `isToggelAnyAssociation`.

---

## سيناريوهات متكاملة (واقعية)

### عرض توصيات لمنتج معين (Cross‑Sell)

نحتاج في صفحة المنتج عرض المنتجات المكملة (التي تم ربطها كأب أو هدف) مع ترتيب حسب `sort_order`.

```php
class Product extends Model
{
    // ... النطاقات

    public function getCrossSellProducts()
    {
        // المنتجات التي هي مكملات لهذا المنتج (كأب)
        $asParent = $this->associations_products()
            ->where('type', ProductsAssociation::CROSS_SELL)
            ->orderBy('sort_order')
            ->get();

        // المنتجات التي هذا المنتج هو مكمل لها (كهدف)
        $asTarget = $this->inverseAssociations_products()
            ->where('type', ProductsAssociation::CROSS_SELL)
            ->orderBy('sort_order')
            ->get();

        return $asParent->merge($asTarget)->unique('id');
    }
}
```

### عرض بدائل للمنتج (Alternate)

```php
public function getAlternates()
{
    return $this->associations_products()
        ->where('type', ProductsAssociation::ALTERNATE)
        ->get()
        ->merge(
            $this->inverseAssociations_products()
                ->where('type', ProductsAssociation::ALTERNATE)
                ->get()
        )->unique('id');
}
```

### إدارة العلاقات في لوحة التحكم (Backend)

استخدام `AssociationsHelper` مع `Filter` لتوفير فلاتر سريعة:

```yaml
# config_filter.yaml
scopes:
    # ... نطاقات أخرى
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

في ملف التحكم، يمكنك إنشاء `listFilterScopes` ودمج ما توفره `AssociationsHelper` مع التعريفات اليدوية.

### تحديث أو حذف ارتباطات بالجوبات (Jobs)

السلوك يوفر دوال `associate()` و `dissociate()` التي تستخدم **الجوبات** (Jobs) لتأخير المعالجة. هذا مفيد لتجنب العمليات الطويلة في الاستجابة.

```php
$product = Product::find(1);
$target = Product::find(2);
$product->associate($target, 'cross_sell'); // يرسل job
$product->dissociate($target, 'cross_sell'); // يرسل job
```

إذا أردت التنفيذ الفوري (بدون جوب)، يمكنك استخدام العلاقات مباشرة:

```php
$association = new ProductsAssociation([
    'type' => 'cross_sell',
    'product_parent_id' => $product->id,
    'product_target_id' => $target->id,
]);
$association->save();
```

لكن استخدام الجوبات أفضل لتجنب مشاكل الأداء عند إنشاء عدد كبير من الارتباطات.

---

## خاتمة

الأمثلة أعلاه تغطي معظم حالات الاستخدام الواقعية لسلوك `AssociationsModel`. من خلال الجمع بين النطاقات المتقدمة، الشروط الإضافية، والمساعد `AssociationsHelper`، يمكنك بناء نظام توصيات منتجات قوي ومرن مع واجهات إدارة سهلة الاستخدام.

**تذكير:**  
- جميع النطاقات تعمل مع `Product` أو أي نموذج مضاف إليه السلوك.  
- استخدم `scopeWhereAssociation` عندما تحتاج إلى تحكم دقيق، والنطاقات المساعدة (`hasAlternate`, `hasCrossSell`, ...) للاستخدام البسيط.  
- `AssociationsHelper` يسهل دمج الفلاتر في الواجهة الخلفية، مما يوفر وقت التطوير.

لا تتردد في تخصيص النطاقات وإضافة أنواع ارتباطات جديدة حسب احتياجات مشروعك.