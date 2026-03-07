# توثيق السمة (Trait) `HasAdvancedTree`

## مقدمة

`HasAdvancedTree` هي سمة (Trait) متقدمة لإدارة البيانات الهرمية (الشجرية) في نماذج Eloquent الخاصة بـ Laravel / OctoberCMS. توفر دوالاً فعالة للتعامل مع الأسلاف (ancestors) والأحفاد (descendants) مع تحسينات أداء كبيرة باستخدام الاستعلامات المتكررة (CTE) والتخزين المؤقت (Caching).

تم تصميم هذه السمة لتعمل بشكل مستقل أو جنباً إلى جنب مع سمات الشجرة الأخرى مثل `SimpleTree` و `NestedTree` دون أي تعارض، وذلك باستخدام أسماء دوال فريدة تبدأ بالبادئة `Node` (مثل `getNodeAncestors`، `getNodeDescendants`، ... إلخ).

---

## المزايا الرئيسية

- **أداء عالي** – تستخدم استعلامات SQL متكررة (CTE) لجلب الأسلاف والأحفاد في استعلام واحد بدلاً من التكرار عبر العلاقات (مدعوم في MySQL 8+، PostgreSQL، SQLite).
- **التخزين المؤقت** – توفر دوالاً لتخزين نتائج الأسلاف والأحفاد لتجنب الاستعلامات المتكررة، مع إمكانية مسح الكاش يدوياً.
- **دعم SoftDeletes** – تتعامل مع السجلات المحذوفة ناعماً بمرونة عبر المعامل `$withTrashed` في معظم الدوال.
- **دوال مساعدة عملية** – مسار العقدة (breadcrumb)، التحقق من العلاقات، بناء الأشجار المتداخلة، قوائم مسطحة، وأكثر.
- **توافق كامل** – يمكن استخدامها مع `SimpleTree` أو `NestedTree` دون تعارض بفضل أسماء الدوال الفريدة.
- **مرونة عالية** – تعمل مع أي نموذج يحتوي على عمود `parent_id` وعلاقات `children` و `parent`.
- **عمق محسوب** – توفر نطاق `scopeWithNodeDepth` لإضافة عمود محسوب للعمق (depth) دون الحاجة إلى عمود مادي في قاعدة البيانات.

---

## المتطلبات

- وجود عمود `parent_id` في جدول النموذج (يمكن تغيير اسمه بتعريف ثابت `PARENT_ID` في النموذج).
- وجود علاقة `children` (hasMany) و `parent` (belongsTo) في النموذج.  
  **ملاحظة:** السمة تقوم بإنشاء هاتين العلاقتين تلقائياً إذا لم تكن موجودة عند تهيئة النموذج (باستخدام `initializeHasAdvancedTree`).
- لا حاجة لوجود أعمدة إضافية مثل `level` أو `depth`، فالسمة تعتمد على الحساب الديناميكي.

---

## التثبيت

1. انسخ ملف `HasAdvancedTree.php` إلى المسار المناسب في مشروعك (مثلاً `nano/tags/traits/`).
2. في النموذج الذي تريد إضافة السمات إليه، استخدم `HasAdvancedTree`:

```php
use Nano\Tags\Traits\HasAdvancedTree;

class Categorie extends Model
{
    use HasAdvancedTree;

    // (اختياري) تغيير اسم عمود parent_id
    const PARENT_ID = 'parent_id';
}
```

3. تأكد من وجود العلاقات التالية في النموذج (السمة تضيفها تلقائياً، لكن يمكنك تعريفها يدوياً أيضاً):

```php
public $hasMany = [
    'children' => [self::class, 'key' => 'parent_id']
];

public $belongsTo = [
    'parent' => [self::class, 'key' => 'parent_id']
];
```

---

## التوافق مع SimpleTree و NestedTree

يمكنك استخدام `HasAdvancedTree` مع `SimpleTree` أو `NestedTree` في نفس النموذج دون مشاكل، لأن جميع دوال `HasAdvancedTree` تحمل البادئة `Node` (مثل `getNodeAncestors`) بينما دوال السمات الأصلية تحمل أسماء عامة (مثل `getParent`). في حالة وجود تعارض في دوال مساعدة (مثل `getParentColumnName`)، يمكنك استخدام `insteadof` لتحديد أي دالة تريد استخدامها:

```php
use October\Rain\Database\Traits\NestedTree;
use Nano\Tags\Traits\HasAdvancedTree;

class Categorie extends Model
{
    use NestedTree, HasAdvancedTree {
        NestedTree::getParentColumnName insteadof HasAdvancedTree;
        NestedTree::getParentId insteadof HasAdvancedTree;
    }
}
```

---

## الدوال المتوفرة

### 1. دوال الأسلاف (Ancestors)

| الدالة | الوصف |
|--------|--------|
| `getNodeAncestors($columns = ['*'], $withTrashed = false)` | يعيد جميع الأسلاف من الأب إلى الجذر. |
| `getNodeAncestorsWithTrashed($columns = ['*'])` | يعيد الأسلاف متضمناً المحذوفين ناعماً. |
| `getNodeSelfAndAncestors($columns = ['*'], $withTrashed = false)` | يعيد العقدة الحالية مع أسلافها. |

### 2. دوال الأحفاد (Descendants)

| الدالة | الوصف |
|--------|--------|
| `getNodeDescendants($columns = ['*'], $withTrashed = false)` | يعيد جميع الأحفاد (بجميع الأعماق) باستخدام CTE إذا كانت قاعدة البيانات تدعمه، وإلا يستخدم الطريقة التكرارية. |
| `getNodeDescendantsWithTrashed($columns = ['*'])` | يعيد الأحفاد متضمناً المحذوفين ناعماً. |
| `getNodeSelfAndDescendants($columns = ['*'], $withTrashed = false)` | يعيد العقدة الحالية مع أحفادها. |
| `getNodeDescendantsUpToDepth(int $depth, array $columns = ['*'], $withTrashed = false)` | يعيد الأحفاد حتى عمق محدد (1 = الأبناء فقط، 2 = الأبناء والأحفاد، ...). |
| `getNodeDescendantsWith($relations, $columns = ['*'], $withTrashed = false)` | يعيد الأحفاد مع تحميل العلاقات المحددة مسبقاً (يتجنب مشكلة N+1). |

### 3. دوال التحقق والخصائص

| الدالة | الوصف |
|--------|--------|
| `isNodeRoot()` | هل العقدة جذر (ليس لها أب)؟ |
| `isNodeLeaf()` | هل العقدة ورقة (ليس لها أبناء)؟ |
| `getNodeDepth($withTrashed = false)` | عمق العقدة (عدد الأسلاف). |
| `getNodeRoot($withTrashed = false)` | يعيد جذر الشجرة لهذه العقدة (أعلى سلف). |
| `getNodeSiblings($withTrashed = false)` | يعيد جميع العقد الشقيقة (باستثناء نفسها). |
| `getNodePath(string $separator = ' > ', string $attribute = 'name', $withTrashed = false): string` | بناء مسار تنقل (breadcrumb) نصي. |
| `isNodeAncestorOf(self $node, $withTrashed = false): bool` | هل العقدة الحالية هي سلف لعقدة أخرى؟ |
| `isNodeDescendantOf(self $node, $withTrashed = false): bool` | هل العقدة الحالية هي نسل لعقدة أخرى؟ |
| `getNodeDescendantsCount($withTrashed = false)` | عدد الأحفاد (بدون تحميلهم). |

### 4. دوال النطاقات (Scopes)

| النطاق | الوصف |
|--------|--------|
| `scopeWhereNodeDescendantOf($query, $id, $withTrashed = false)` | يفلتر العقد التي هي من نسل عقدة معينة (بما فيها العقدة نفسها). |
| `scopeWhereNodeAncestorOf($query, $id, $withTrashed = false)` | يفلتر العقد التي هي أسلاف عقدة معينة (بما فيها العقدة نفسها). |
| `scopeWithNodeDepth($query, $depthColumn = 'depth')` | يضيف عمق كل عقدة إلى النتائج كعمود محسوب (يدعم CTE). هذا العمود غير موجود في قاعدة البيانات بل يتم حسابه ديناميكياً. يمكن تسميته حسب الرغبة (مثلاً `level`). |
| `scopeWhereNodeDepth($query, $operator, $value, $depthColumn = 'depth')` | يفلتر العقد حسب العمق (مثلاً `whereNodeDepth('>', 2)`). |

**ملاحظة:** النطاق `scopeWithNodeDepth` يضيف عموداً محسوباً يمثل عمق العقدة (عدد الأسلاف). هذا العمود لا يتعارض مع أي عمود موجود في الجدول مثل `level`؛ إنه قيمة محسوبة أثناء الاستعلام. يمكنك استخدام أي اسم تريده للعمود المحسوب عبر المعامل `$depthColumn`.

### 5. دوال مساعدة عامة (Static)

| الدالة | الوصف |
|--------|--------|
| `buildFlatNodeTree($items, $labelColumn = 'name', $keyColumn = 'id', $indent = '&nbsp;&nbsp;&nbsp;')` | يبني قائمة مسطحة مع مسافات بادئة من أي مجموعة عقد (مفيد للقوائم المنسدلة). |
| `toFlatNodeTree($labelColumn = 'name', $keyColumn = 'id', $indent = '&nbsp;&nbsp;&nbsp;')` | يعيد مصفوفة مسطحة لجميع العقد في الشجرة (مرتبة حسب العمق المحسوب). |
| `buildNestedNodeTree($items, $parentId = null)` | يبني شجرة متداخلة (مع تعبئة `children`) من مجموعة مسطحة من العقد. |
| `getNodeRoots($withTrashed = false)` | يعيد جميع العقد الجذرية. |
| `getNodeHierarchy($withTrashed = false)` | يعيد تمثيلاً كاملاً للشجرة الهرمية (جذور مع أطفال متداخلة). |

### 6. دوال التخزين المؤقت (Caching)

| الدالة | الوصف |
|--------|--------|
| `getCachedNodeAncestors($columns = ['*'])` | يعيد الأسلاف من التخزين المؤقت (مدة التخزين محددة في `$cacheTtl`). |
| `getCachedNodeDescendants($columns = ['*'])` | يعيد الأحفاد من التخزين المؤقت. |
| `clearNodeTreeCache()` | يمسح التخزين المؤقت للعقدة الحالية. |

---

## أمثلة عملية

لنفترض أن لدينا نموذج `Categorie` يحتوي على البيانات التالية (جدول `nano_tags_categories` يحتوي على عمود `level` ولكننا لا نستخدمه في هذه الأمثلة، حيث نعتمد على العمق المحسوب):

| id | name          | parent_id |
|----|---------------|-----------|
| 1  | إلكترونيات    | null      |
| 2  | هواتف         | 1         |
| 3  | آيفون         | 2         |
| 4  | سامسونج       | 2         |
| 5  | أجهزة لوحية   | 1         |
| 6  | آيباد         | 5         |
| 7  | ملابس         | null      |
| 8  | رجالي         | 7         |
| 9  | نسائي         | 7         |

### مثال 1: الحصول على جميع أسلاف فئة "آيفون"

```php
$iphone = Categorie::find(3);
$ancestors = $iphone->getNodeAncestors(['id', 'name']);
```

**المخرجات** (مجموعة من الكائنات):
```
Collection {
    0 => Categorie { id: 2, name: "هواتف" },
    1 => Categorie { id: 1, name: "إلكترونيات" }
}
```

### مثال 2: الحصول على جميع أحفاد فئة "إلكترونيات"

```php
$electronics = Categorie::find(1);
$descendants = $electronics->getNodeDescendants(['id', 'name']);
```

**المخرجات** (مجموعة مسطحة):
```
Collection {
    0 => Categorie { id: 2, name: "هواتف" },
    1 => Categorie { id: 3, name: "آيفون" },
    2 => Categorie { id: 4, name: "سامسونج" },
    3 => Categorie { id: 5, name: "أجهزة لوحية" },
    4 => Categorie { id: 6, name: "آيباد" }
}
```

### مثال 3: الحصول على أحفاد حتى عمق 1 (الأبناء فقط)

```php
$descendantsDepth1 = $electronics->getNodeDescendantsUpToDepth(1, ['id', 'name']);
```

**المخرجات**:
```
Collection {
    0 => Categorie { id: 2, name: "هواتف" },
    1 => Categorie { id: 5, name: "أجهزة لوحية" }
}
```

### مثال 4: مسار العقدة (Breadcrumb) لفئة "آيباد"

```php
$ipad = Categorie::find(6);
$path = $ipad->getNodePath(' / ', 'name');
```

**المخرجات**:
```
"إلكترونيات / أجهزة لوحية / آيباد"
```

### مثال 5: التحقق من العلاقات

```php
$electronics = Categorie::find(1);
$iphone = Categorie::find(3);

if ($electronics->isNodeAncestorOf($iphone)) {
    echo "إلكترونيات هي سلف لآيفون";
}
// الناتج: إلكترونيات هي سلف لآيفون

if ($iphone->isNodeDescendantOf($electronics)) {
    echo "آيفون هو نسل لإلكترونيات";
}
// الناتج: آيفون هو نسل لإلكترونيات
```

### مثال 6: الحصول على العقد الشقيقة

```php
$iphone = Categorie::find(3);
$siblings = $iphone->getNodeSiblings(['id', 'name']);
```

**المخرجات**:
```
Collection {
    0 => Categorie { id: 4, name: "سامسونج" }
}
```

### مثال 7: استخدام النطاقات

```php
// جميع الفئات التي هي من نسل "إلكترونيات"
$descendantCategories = Categorie::whereNodeDescendantOf(1)->get();

// جميع الفئات التي عمقها أكبر من 1 (أي ليست جذوراً ولا أبناء مباشرين)
$deepCategories = Categorie::whereNodeDepth('>', 1)->get();
```

### مثال 8: استخدام `scopeWithNodeDepth` للحصول على العمق المحسوب

```php
$categoriesWithDepth = Categorie::withNodeDepth('calculated_depth')->get();
foreach ($categoriesWithDepth as $cat) {
    echo $cat->name . ' - العمق: ' . $cat->calculated_depth . "\n";
}
```

**المخرجات** (مثال):
```
إلكترونيات - العمق: 0
هواتف - العمق: 1
آيفون - العمق: 2
سامسونج - العمق: 2
أجهزة لوحية - العمق: 1
آيباد - العمق: 2
ملابس - العمق: 0
رجالي - العمق: 1
نسائي - العمق: 1
```

**ملاحظة:** العمود `calculated_depth` غير موجود في قاعدة البيانات، بل تم إنشاؤه ديناميكياً بواسطة الاستعلام المتكرر.

### مثال 9: بناء شجرة متداخلة

```php
$allCategories = Categorie::all();
$nestedTree = Categorie::buildNestedNodeTree($allCategories);

foreach ($nestedTree as $root) {
    echo $root->name . "\n";
    foreach ($root->children as $child) {
        echo "  " . $child->name . "\n";
    }
}
```

**المخرجات**:
```
إلكترونيات
  هواتف
  أجهزة لوحية
ملابس
  رجالي
  نسائي
```

### مثال 10: الحصول على قائمة مسطحة مع مسافات بادئة (للقوائم المنسدلة)

```php
$flatList = Categorie::toFlatNodeTree('name', 'id', '--');
```

**المخرجات**:
```php
[
    1 => 'إلكترونيات',
    2 => '--هواتف',
    3 => '----آيفون',
    4 => '----سامسونج',
    5 => '--أجهزة لوحية',
    6 => '----آيباد',
    7 => 'ملابس',
    8 => '--رجالي',
    9 => '--نسائي'
]
```

### مثال 11: استخدام التخزين المؤقت

```php
// المرة الأولى: يتم تنفيذ الاستعلام والتخزين
$ancestors = $iphone->getCachedNodeAncestors();

// المرة الثانية: يتم جلبها من الكاش مباشرة (أسرع)
$ancestorsAgain = $iphone->getCachedNodeAncestors();

// مسح الكاش لهذه العقدة
$iphone->clearNodeTreeCache();
```

### مثال 12: جلب الأحفاد مع تحميل العلاقات

لنفترض أن فئة `Categorie` لديها علاقة `products` (منتجات مرتبطة بها). نريد جلب جميع أحفاد فئة معينة مع تحميل منتجاتهم:

```php
$electronics = Categorie::find(1);
$descendantsWithProducts = $electronics->getNodeDescendantsWith('products', ['id', 'name']);
```

هذا يقوم بجلب جميع الأحفاد أولاً (معرفاتهم فقط)، ثم في استعلام واحد يقوم بتحميلهم مع علاقة `products`، مما يتجنب مشكلة N+1.

### مثال 13: الحصول على جميع العقد الجذرية

```php
$roots = Categorie::getNodeRoots();
foreach ($roots as $root) {
    echo $root->name; // إلكترونيات، ملابس
}
```

### مثال 14: الحصول على تمثيل كامل للشجرة (hierarchy)

```php
$hierarchy = Categorie::getNodeHierarchy();
// يعيد مجموعة من الجذور مع children ممتلئة لجميع المستويات
```

---

## ملاحظات هامة حول العمق (Depth)

- السمة لا تستخدم عمود `level` الموجود في جدول `nano_tags_categories` (إذا كان موجوداً) لحساب العمق. بدلاً من ذلك، تعتمد على العلاقات وتحسب العمق ديناميكياً.
- نطاق `scopeWithNodeDepth` يضيف عموداً محسوباً (calculated column) إلى النتائج، يمكن تسميته حسب الرغبة (مثلاً `depth` أو `level`). هذا العمود لا يتعارض مع أي عمود موجود في الجدول.
- إذا كان جدولك يحتوي بالفعل على عمود `level`، يمكنك استخدامه كأي عمود عادي، لكن السمة لا تقرأه أو تكتبه تلقائياً. يمكنك تحديثه يدوياً إذا احتجت لذلك.
- الدوال مثل `getNodeDepth` تعتمد على العلاقات (عدد الأسلاف) وليس على عمود `level`.

---

## ملاحظات عامة

- السمة لا تتطلب وجود أي أعمدة إضافية غير `parent_id`. تعتمد فقط على العلاقات `parent` و `children`.
- عند استخدام `getNodeDescendants` مع مجموعات كبيرة جداً، يُفضل استخدام `getNodeDescendantsCount` أولاً لتقدير الحجم، أو استخدام `getNodeDescendantsUpToDepth` إذا كنت تحتاج عمقاً محدداً.
- دوال CTE (مثل `getNodeDescendantsUsingCte`) تعمل فقط مع قواعد البيانات التي تدعم الاستعلامات المتكررة (MySQL 8+، PostgreSQL، SQLite). في حالة عدم الدعم، تتحول السمة تلقائياً إلى الطريقة التكرارية.
- عند تفعيل `withTrashed = true`، يتم استخدام الطريقة التكرارية دائماً لضمان دقة النتائج مع السجلات المحذوفة، لأن دمج soft deletes مع CTE معقد.
- السمة تقوم تلقائياً بإنشاء العلاقات `children` و `parent` إذا لم تكن موجودة عند تهيئة النموذج (في `initializeHasAdvancedTree`). هذا مفيد في OctoberCMS.
- يمكن تغيير مدة التخزين المؤقت بتعديل الخاصية `$cacheTtl` (بالدقائق) في النموذج الذي يستخدم السمة.
- جميع الدوال التي تتعامل مع الأسلاف والأحفاد تدعم تحديد الأعمدة المطلوبة (`$columns`) لتقليل البيانات المحملة.

---

## خاتمة

`HasAdvancedTree` هي إضافة قوية لأي نموذج يحتاج إلى إدارة بيانات هرمية في Laravel أو OctoberCMS. بفضل أدائها العالي ومرونتها وتوافقها مع السمات الأخرى، يمكنها تسهيل الكثير من المهام الشائعة مثل بناء القوائم المتداخلة، عرض مسارات التنقل، والاستعلامات المعقدة على الشجرة. نأمل أن يكون هذا التوثيق مفيداً لك في استخدام السمة بكفاءة.