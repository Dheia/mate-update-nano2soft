# توثيق السلوك `CategoriesModel`

## مقدمة

`CategoriesModel` هو سلوك (Behavior) متقدم مصمم لإضافة علاقة تصنيفات متعددة الأشكال (polymorphic many-to-many) إلى أي موديل في OctoberCMS. يتيح هذا السلوك ربط أي نموذج (مثل المنتجات، المقالات، المستخدمين، إلخ) بالتصنيفات المخزنة في موديل `Nano\Tags\Models\Categorie`، مع توفير مجموعة غنية من الدوال والنطاقات للتعامل مع هذه التصنيفات بمرونة وكفاءة.

### الفوائد الرئيسية

- **علاقة تصنيفات موحدة** – يمكن إضافة التصنيفات إلى أي موديل بسهولة.
- **دعم slugs** – يمكن استخدام slugs بدلاً من المعرفات الرقمية في جميع الدوال.
- **مصفوفات مختلطة** – تقبل الدوال خليطاً من الأرقام والنصوص (slugs) في نفس المصفوفة.
- **تصفية هرمية** – إمكانية تضمين الأسلاف والأحفاد عند التصفية (باستخدام `HasAdvancedTree`).
- **أنواع مطابقة متعددة** – دعم `any` (أي من التصنيفات) و `all` (كل التصنيفات).
- **تحسين الأداء** – تخزين مؤقت لنتائج تحضير المعرفات لتجنب الاستعلامات المتكررة.
- **دوال تحقق مساعدة** – التحقق من وجود التصنيفات بطرق مختلفة.
- **ترتيب حسب التصنيفات** – إمكانية ترتيب النتائج حسب اسم التصنيف أو عدد التصنيفات.

---

## متطلبات الاستخدام

1. وجود موديل `Nano\Tags\Models\Categorie` مع تطبيق السمات المناسبة (خاصة `HasAdvancedTree` للاستفادة من الدوال الهرمية).
2. وجود جدول الوسيط (pivot) المحدد في `Categorie::PIVOT_TABLE` (افتراضياً `nano_tags_cateables`).
3. الموديل الذي سيُضاف إليه السلوك يجب أن يكون من نوع Eloquent (أو October\Rain\Database\Model).

---

## إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في OctoberCMS) أو عن طريق إضافته يدوياً عبر `extend()`.

### الطريقة الأولى (في OctoberCMS باستخدام $implement)

```php
class Product extends Model
{
    public $implement = [
        'Nano\Tags\Behaviors\CategoriesModel'
    ];
}
```

### الطريقة الثانية (يدوياً باستخدام extend)

```php
Product::extend(function($model) {
    $model->implement[] = 'Nano\Tags\Behaviors\CategoriesModel';
});
```

بعد إضافة السلوك، يصبح للموديل العلاقة التالية:

```php
$product->pcategories // يعيد مجموعة من التصنيفات المرتبطة (من نوع Categorie)
```

---

## الدوال والنطاقات المتاحة

### 1. دوال تحضير المعرفات (داخلية)

#### `prepareCategoryIds($categories, $is_force_all = false, $useCache = true)`
- **الوصف**: تحويل المدخلات المتنوعة إلى مصفوفة من المعرفات الرقمية.
- **المدخلات**:
  - `$categories`: يمكن أن يكون رقماً، نصاً (id أو slug أو نصاً بفواصل)، مصفوفة مختلطة، كائن `Categorie`، أو `Collection`.
  - `$is_force_all`: تجاهل التحقق من قيمة "الكل".
  - `$useCache`: استخدام التخزين المؤقت.
- **المخرجات**: مصفوفة من الأرقام (معرفات فريدة).

### 2. نطاقات التصفية الأساسية

#### `scopeWhereCategories($query, $categories, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- **الوصف**: النطاق الرئيسي للتصفية حسب التصنيفات.
- **المعاملات**:
  - `$categories`: التصنيفات المطلوبة (يدعم جميع الأنواع).
  - `$matchType`: `'any'` (أي تصنيف) أو `'all'` (كل التصنيفات).
  - `$not`: عكس الشرط (أي لا تحتوي على هذه التصنيفات).
  - `$forceAll`: تجاهل قيمة "الكل".
  - `$useCache`: استخدام التخزين المؤقت.
- **مثال**:
  ```php
  // منتجات تحتوي على أي من التصنيفات 1 أو 5 أو التي slugها "electronics"
  $products = Product::whereCategories([1, 5, 'electronics'])->get();
  
  // منتجات تحتوي على كل من التصنيفات 1 و "clothing" معاً
  $products = Product::whereCategories([1, 'clothing'], 'all')->get();
  ```

#### `scopeWhereAnyCategories($query, $categories, $not = false, $forceAll = false, $useCache = true)`
- اختصار لـ `whereCategories` مع `matchType = 'any'`.

#### `scopeWhereAllCategories($query, $categories, $not = false, $forceAll = false, $useCache = true)`
- اختصار لـ `whereCategories` مع `matchType = 'all'`.

#### `scopeOrWhereCategories($query, $categories, $forceAll = false, $useCache = true)`
- إضافة شرط OR للتصنيفات.

#### `scopeWhereNotCategories($query, $categories, $forceAll = false, $useCache = true)`
- تصفية النتائج التي لا تحتوي على التصنيفات المحددة.

### 3. نطاقات التصفية بالـ slug

#### `scopeWhereCategorySlug($query, $slug, $matchType = 'any', $not = false, $forceAll = false)`
- تصفية حسب slug التصنيف.

#### `scopeOrWhereCategorySlug($query, $slug, $forceAll = false)`
- شرط OR حسب slug.

### 4. نطاقات التصفية النشطة

#### `scopeWhereCategoriesActive($query, $categories, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- تصفية حسب التصنيفات النشطة فقط (`is_active = true`).

### 5. نطاقات التصفية الهرمية (باستخدام HasAdvancedTree)

#### `scopeWhereHasCategoryWithDescendants($query, $categoryId, $includeDescendants = true, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- تضمين جميع أحفاد التصنيف المحدد.
- مثال: `Product::whereHasCategoryWithDescendants('electronics')->get()` يجلب كل المنتجات في تصنيف "electronics" أو أي تصنيف فرعي تحته.

#### `scopeWhereHasCategoryWithAncestors($query, $categoryId, $includeAncestors = true, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- تضمين جميع أسلاف التصنيف المحدد.

#### `scopeWhereHasCategoryUpToDepth($query, $categoryId, $depth, $matchType = 'any', $not = false, $forceAll = false, $useCache = true)`
- تضمين الأحفاد حتى عمق معين (1 = الأبناء فقط، 2 = الأبناء والأحفاد، ...).

### 6. نطاقات الترتيب

#### `scopeOrderByCategoryName($query, $direction = 'asc')`
- ترتيب النتائج حسب اسم التصنيف المرتبط (أول تصنيف).

#### `scopeOrderByCategoriesCount($query, $direction = 'asc')`
- ترتيب النتائج حسب عدد التصنيفات المرتبطة.

### 7. دوال الخصائص المحسوبة

#### `getIsCategoriesAttribute()`
- **الوصف**: تحقق مما إذا كان الموديل يحتوي على أي تصنيفات.
- **الاستخدام**: `$product->is_categories` (يعيد true/false).

#### `getCategoriesCountAttribute()`
- **الوصف**: عدد التصنيفات المرتبطة.
- **الاستخدام**: `$product->categories_count`.

#### `getPcategoriesCountAttribute()`
- مرادف للدالة السابقة.

### 8. دوال مساعدة لجلب التصنيفات

#### `getCategories($relations = [], $activeOnly = false)`
- **الوصف**: جلب جميع التصنيفات المرتبطة.
- **المعاملات**:
  - `$relations`: علاقات لتحميلها مسبقاً.
  - `$activeOnly`: جلب التصنيفات النشطة فقط.
- **المخرجات**: `Collection` من `Categorie`.
- **مثال**: `$product->getCategories(['parent'])` يجلب التصنيفات مع تحميل الأب.

#### `getCategoriesHierarchy($activeOnly = false)`
- **الوصف**: جلب التصنيفات على شكل شجرة متداخلة (مع تعبئة `children`).
- **المخرجات**: `Collection` من الجذور مع `children` ممتلئة.

#### `getCategoriesFlatList($labelColumn = 'name', $keyColumn = 'id', $indent = '&nbsp;&nbsp;&nbsp;', $activeOnly = false)`
- **الوصف**: جلب قائمة مسطحة مع مسافات بادئة (مناسبة للقوائم المنسدلة).
- **المخرجات**: مصفوفة `[id => مسافة + name]`.

#### `getCategoryIds()`
- **الوصف**: جلب مصفوفة من معرفات التصنيفات المرتبطة.

### 9. دوال التحقق

#### `hasCategory($categoryId, $relationType = 'self', $forceAll = false, $useCache = true)`
- **الوصف**: التحقق من وجود تصنيف معين أو علاقاته.
- **`$relationType`**: 
  - `'self'`: التصنيف نفسه.
  - `'descendants'`: التصنيف أو أحفاده.
  - `'ancestors'`: التصنيف أو أسلافه.
  - `'both'`: التصنيف أو أسلافه أو أحفاده.
- **مثال**: `$product->hasCategory('electronics', 'descendants')`.

#### `hasAnyCategory(array $categoryIds)`
- التحقق من وجود أي من التصنيفات المحددة.

#### `hasAllCategories(array $categoryIds)`
- التحقق من وجود كل التصنيفات المحددة.

---

## أمثلة عملية شاملة

### مثال 1: تصنيف المنتجات

لنفترض أن لدينا موديل `Product` مضاف إليه السلوك. نريد:

```php
// إنشاء منتج وربطه بتصنيفات
$product = Product::find(1);
$product->pcategories()->attach([1, 2, 3]); // ربط بمعرفات
$product->pcategories()->attach(Categorie::where('slug', 'electronics')->first()); // ربط بكائن

// جلب جميع منتجات التصنيف "electronics"
$products = Product::whereCategorySlug('electronics')->get();

// جلب منتجات التصنيف "electronics" أو أي تصنيف فرعي
$products = Product::whereHasCategoryWithDescendants('electronics')->get();

// جلب منتجات تحتوي على كل من "electronics" و "sale" معاً
$products = Product::whereAllCategories(['electronics', 'sale'])->get();

// جلب منتجات لا تحتوي على التصنيف "out-of-stock"
$products = Product::whereNotCategories('out-of-stock')->get();
```

### مثال 2: استخدام المصفوفات المختلطة

```php
// نص بفواصل
$products = Product::whereCategories("1,5,electronics,clothing")->get();

// مصفوفة مختلطة
$products = Product::whereCategories([1, 'electronics', 7, 'sale'])->get();
// الدالة ستستخرج المعرفات 1,7 وتبحث عن slugs "electronics" و "sale" وتحولها إلى معرفات

// استخدام forceAll لتجاهل الكلمة "all"
$products = Product::whereCategories(['all', 'electronics'], 'any', false, true)->get();
// forceAll = true يعني أن "all" تعامل كـ slug وليس كقيمة خاصة
```

### مثال 3: التصفية الهرمية

```php
// كل المنتجات في تصنيف "إلكترونيات" أو أي تصنيف فرعي
$products = Product::whereHasCategoryWithDescendants('electronics')->get();

// كل المنتجات في تصنيف "هواتف" أو أي تصنيف أعلى (أسلاف)
$products = Product::whereHasCategoryWithAncestors('phones')->get();

// كل المنتجات في تصنيف "آيفون" أو أبنائه حتى عمق 2
$products = Product::whereHasCategoryUpToDepth('iphone', 2)->get();
```

### مثال 4: الترتيب والخصائص المحسوبة

```php
// ترتيب المنتجات حسب اسم التصنيف
$products = Product::orderByCategoryName()->get();

// ترتيب حسب عدد التصنيفات
$products = Product::orderByCategoriesCount('desc')->get();

// استخدام الخصائص المحسوبة
foreach ($products as $product) {
    echo $product->name . ' - تصنيفات: ' . $product->categories_count . "\n";
    if ($product->is_categories) {
        echo "له تصنيفات\n";
    }
}
```

### مثال 5: جلب التصنيفات بطرق مختلفة

```php
$product = Product::find(1);

// جلب التصنيفات مع تحميل العلاقات
$categories = $product->getCategories(['parent', 'children']);

// جلب التصنيفات على شكل شجرة
$tree = $product->getCategoriesHierarchy();
foreach ($tree as $root) {
    echo $root->name . "\n";
    foreach ($root->children as $child) {
        echo " - " . $child->name . "\n";
    }
}

// جلب قائمة منسدلة
$list = $product->getCategoriesFlatList('name', 'id', '--');
// الناتج: [1 => 'إلكترونيات', 2 => '--هواتف', 3 => '----آيفون', ...]
```

### مثال 6: التحقق المتقدم

```php
$product = Product::find(1);

if ($product->hasCategory('electronics')) {
    echo "المنتج في تصنيف electronics";
}

if ($product->hasCategory('electronics', 'descendants')) {
    echo "المنتج في electronics أو أي تصنيف فرعي";
}

if ($product->hasAnyCategory([1, 5, 7])) {
    echo "المنتج له واحد على الأقل من هذه التصنيفات";
}

if ($product->hasAllCategories([2, 3])) {
    echo "المنتج له كل هذين التصنيفين";
}
```

---

## تحسين الأداء والتخزين المؤقت

السلوك يستخدم التخزين المؤقت افتراضياً عند تحضير المعرفات (`prepareCategoryIds`) لتجنب الاستعلامات المتكررة عن slugs. يمكن التحكم في ذلك عبر المعامل `$useCache`.

- مدة التخزين المؤقت: ساعة واحدة (قابلة للتعديل بتغيير الخاصية `$cacheTtl`).
- يتم إنشاء مفتاح فريد بناءً على المدخلات.

يمكن تعطيل التخزين المؤقت مؤقتاً:

```php
$products = Product::whereCategories($categories, 'any', false, false, false); // آخر false يعطل cache
```

---

## التوافق مع الإصدارات السابقة

تم الاحتفاظ بالدوال القديمة (`scopeWhereHasCategory`, `scopeWhereHasCategorie`, `scopeWhereHasPcategorie`) للتوافق مع الكود القديم. تستدعي هذه الدوال النطاق الجديد `whereAnyCategories` مع القيم الافتراضية.

يُنصح باستخدام الدوال الجديدة (`whereCategories`, `whereAnyCategories`, `whereAllCategories`) في التطوير الجديد.

---

## الخلاصة

`CategoriesModel` هو أداة قوية ومرنة لإدارة التصنيفات في أي موديل. بفضل دعمه للـ slugs، المصفوفات المختلطة، التصفية الهرمية، والتخزين المؤقت، يمكن للمطورين بناء أنظمة تصنيف متقدمة بسهولة وكفاءة. نأمل أن يكون هذا التوثيق مفيداً لاستكشاف كامل إمكانيات السلوك.