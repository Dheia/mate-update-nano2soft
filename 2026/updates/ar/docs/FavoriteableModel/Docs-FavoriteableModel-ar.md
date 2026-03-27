# توثيق سلوك `FavoriteableModel` 

---

## 1. مقدمة

`FavoriteableModel` هو سلوك متقدم يضيف نظام مفضلات متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمستخدمين إضافة كائنات (مثل المنتجات، المقالات، المستخدمين) إلى قائمة المفضلات لديهم، مع دعم إمكانية تمييز أنواع مختلفة من المفضلات (مثل `favorite`, `like`, `bookmark`) عبر حقل `value`. يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة تفضيل متطورة بمرونة وأداء عالٍ.

### الفوائد الرئيسية

- **نظام مفضلات متعدد الأشكال** – يمكن جعل أي موديل قابلاً للإضافة إلى المفضلة.
- **دعم أنواع متعددة** – إمكانية تصنيف المفضلات عبر حقل `value` (مثل `favorite`, `like`, `bookmark`).
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد المفضلات، أحدث تفضيل، أو أقدم تفضيل.
- **نطاقات تصفية** – تصفية الكائنات التي أضافها مستخدم معين إلى المفضلة، أو التي لها مفضلات، أو التي لا مفضلات لها.
- **إضافة عمود `is_favorited_by_user`** – يظهر في نتائج الاستعلامات ما إذا كان المستخدم الحالي قد أضاف الكائن إلى المفضلة، دون الحاجة إلى تحميل العلاقة.
- **واجهة متسقة** – دوال مشابهة لسلوكيات المتابعة والتقييمات والزيارات (FollowableModel, ReviewRateable, VisitModel) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`sortByCountFavoritesOld`, `withSortByCountFavorites`, `whereHasFavorite`، إلخ) لضمان عدم كسر الكود القديم، مع دعم تحسينات إضافية للتحكم في السلوك عند عدم وجود مستخدم.

---

## 2. متطلبات الاستخدام

1. وجود موديل `Nano\Markable\Models\Favorite` (الذي يمثل سجل المفضلة) مع جدول `nano_markable_favorites`.
2. الموديل المراد جعله قابلاً للإضافة إلى المفضلة يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام المفضلة (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: إذا كنت ترغب في دعم أنواع متعددة من المفضلات، قم بتعيين القيم المسموحة عبر الإعدادات (`nano.markable::allowed_values.favorite`).

---

## 3. إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في سياق نانوسوفت). السلوك يتضمن Trait يحمل النطاقات والدوال المساعدة، لذا بعد الإضافة يصبح الموديل قادراً على استخدام جميع الميزات.

### مثال: جعل موديل `Product` قابلاً للإضافة إلى المفضلة

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\FavoriteableModel;

class Product extends Model
{
    public $implement = [
        FavoriteableModel::class
    ];
}
```

**ملاحظة:** إذا كان الموديل يستخدم نظام `ExtensionBase` (كما هو الحال في تطبيقات نانوسوفت)، فإن الإضافة تتم عبر `$implement` كما هو موضح.

---

## 4. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقة التالية:

```php
$product->favorites() // يعيد علاقة morphMany تربط سجلات Favorite بهذا الموديل
```

كما يتوفر خصائص محسوبة ديناميكية (تُضاف تلقائياً):

- `$product->favorites_count` – عدد المفضلات لهذا الكائن.
- `$product->is_favorited()` – هل المستخدم الحالي أضاف هذا الكائن إلى المفضلة.
- `$product->getFavorited()` – إرجاع سجل المفضلة الخاص بالمستخدم الحالي (إن وجد).

---

## 5. الدوال والنطاقات المتاحة

### 5.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildFavoriteCountSubQuery($onlyActive = null, $withTrashed = false, $value = null)`
- **الوصف**: يبني استعلام فرعي لحساب عدد سجلات المفضلات (COUNT).
- **المعاملات**:
  - `$value` (string|null): تصفية حسب نوع المفضلة (مثل `favorite` أو `like`).

#### `protected function buildFavoriteLatestDateSubQuery($field = 'created_at', $onlyActive = null, $withTrashed = false, $value = null)`
- **الوصف**: يبني استعلام فرعي لأحدث تاريخ تفضيل (MAX).

#### `protected function buildFavoriteEarliestDateSubQuery(...)` – تستخدم `MIN(field)` للحصول على أقدم تاريخ.

### 5.2 نطاقات عدد المفضلات (COUNT)

#### `scopeAddCountFavorites(Builder $query, $orderDirection = 'DESC', $columnName = 'favorites_count', $value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يضيف عموداً يحتوي على عدد سجلات المفضلات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountFavorites()->get();
  foreach ($products as $product) {
      echo $product->favorites_count;
  }
  ```

#### `scopeSortByCountFavorites(...)` – يرتب النتائج حسب عدد المفضلات (دون إضافة العمود).

#### `scopeWithCountFavorites(...)` – يجمع بين `addCountFavorites` و `sortByCountFavorites` (إضافة العمود والترتيب معاً).

**مثال**:
```php
$products = Product::withCountFavorites('DESC', 'favorites_count', 'like')->get(); // فقط مفضلات من نوع like
```

### 5.3 نطاقات تاريخ المفضلة

#### `scopeAddLatestFavorite(...)` – يضيف عموداً بأحدث تاريخ تفضيل (اختيارياً تحديد الحقل عبر `$field`).
#### `scopeSortByLatestFavorite(...)` – ترتيب حسب أحدث تاريخ تفضيل.
#### `scopeWithLatestFavorite(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::withLatestFavorite('DESC', 'last_favorite', 'favorite')->get();
```

#### `scopeAddEarliestFavorite(...)` – يضيف عموداً بأقدم تاريخ تفضيل.
#### `scopeSortByEarliestFavorite(...)` – ترتيب حسب أقدم تاريخ تفضيل.
#### `scopeWithEarliestFavorite(...)` – إضافة العمود والترتيب معاً.

### 5.4 نطاقات التصفية حسب المستخدم

#### `scopeFavoritedByUser(Builder $query, $user = null, $value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي أضافها مستخدم معين إلى المفضلة (إذا لم يُمرر `$user`، يستخدم المستخدم الحالي).
- **مثال**:
  ```php
  // المنتجات التي أضافها المستخدم الحالي إلى المفضلة
  $products = Product::favoritedByUser()->get();
  
  // المنتجات التي أضافها مستخدم محدد من نوع 'like'
  $products = Product::favoritedByUser($user, 'like')->get();
  ```

#### `scopeNotFavoritedByUser(...)` – تصفية الكائنات التي **لم** يضفها مستخدم معين إلى المفضلة.

### 5.5 نطاقات التصفية حسب وجود مفضلات

#### `scopeHasFavorites(Builder $query, $minCount = 1, $value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من المفضلات (مع خيارات التصفية حسب `value`).
- **مثال**:
  ```php
  // منتجات لها 5 مفضلات من نوع 'favorite' على الأقل
  $products = Product::hasFavorites(5, 'favorite')->get();
  ```

#### `scopeHasNoFavorites(...)` – تصفية الكائنات التي لا مفضلات لها.

### 5.6 نطاق إضافة عمود حالة المفضلة للمستخدم الحالي

#### `scopeWithIsFavoritedByUser(Builder $query, $user = null, $columnName = 'is_favorited_by_user', $value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد قد أضاف الكائن إلى المفضلة.
- **مثال**:
  ```php
  $products = Product::withIsFavoritedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_favorited_by_user) {
          echo "هذا المنتج في مفضلتك";
      }
  }
  ```

### 5.7 نطاقات خاصة

#### `scopeTopFavorited(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: يجلب الكائنات الأكثر تفضيلاً (حسب عدد المفضلات) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topFavorited(10, 'DESC', 'like')->get();
  ```

### 5.8 دوال إحصائية مساعدة

#### `getTotalFavorites($value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: إجمالي عدد المفضلات (مجموع `COUNT`) حسب خيارات التصفية.
- **المخرجات**: عدد صحيح.

#### `getFavoritesCountByType($value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: توزيع المفضلات حسب نوع المستخدم (`user_type`).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.

#### `getFavoritersUsers($value = null, $onlyActive = null, $withTrashed = false)`
- **الوصف**: قائمة المستخدمين الذين أضافوا الكائن إلى المفضلة (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` من كائنات المستخدمين.

---

## 6. أمثلة عملية شاملة

### مثال 1: إضافة عنصر إلى المفضلة والتحقق

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Favorite;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة المنتج إلى المفضلة (النوع الافتراضي 'favorite')
$favorite = Favorite::add($product, $user);

// إضافة المنتج كمفضلة من نوع 'like'
$like = Favorite::add($product, $user, 'like');

// التحقق مما إذا كان المستخدم قد أضاف المنتج إلى المفضلة
if ($product->isFavorited()) {
    echo "المنتج في قائمة المفضلة";
}
```

### مثال 2: إزالة عنصر من المفضلة

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إزالة المنتج من المفضلة (النوع الافتراضي)
Favorite::remove($product, $user);

// إزالة المنتج من مفضلات نوع 'like'
Favorite::remove($product, $user, 'like');
```

### مثال 3: ترتيب المنتجات حسب عدد المفضلات

```php
$topProducts = Product::topFavorited(10)->get();
```

### مثال 4: إضافة عمود `is_favorited_by_user` في قائمة المنتجات

```php
$products = Product::withIsFavoritedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_favorited_by_user ? 'مفضل' : 'غير مفضل');
}
```

### مثال 5: تصفية المنتجات التي أضافها المستخدم الحالي إلى المفضلة

```php
$myFavorites = Product::favoritedByUser()->get();
```

### مثال 6: الإحصائيات

```php
$product = Product::find(1);
echo "عدد المفضلات: " . $product->getTotalFavorites();
print_r($product->getFavoritesCountByType()->toArray());
$users = $product->getFavoritersUsers();
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين النطاقات المتقدمة

```php
// المنتجات التي لها أكثر من 5 مفضلات من نوع 'like'، مرتبة حسب أحدث تفضيل
$products = Product::hasFavorites(5, 'like')
    ->sortByLatestFavorite('DESC', 'latest_favorite_at')
    ->get();
```

---

## 7. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة، تم توفير دوال تغليف في الكلاس الرئيسي. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

| الدالة القديمة | الدالة الجديدة |
|----------------|----------------|
| `scopeSortByCountFavoritesOld` | `scopeSortByCountFavorites` |
| `scopeWithSortByCountFavorites` | `scopeWithCountFavorites` |
| `scopeAddSortByCountFavorites` | `scopeAddCountFavorites` |
| `scopeSortByCreatedAtFavorites` | `scopeSortByLatestFavorite` |
| `scopeAddSortByCreatedAtFavorites` | `scopeAddLatestFavorite` |
| `scopeWithSortByCreatedAtFavorites` | `scopeWithLatestFavorite` |
| `scopeWhereHasFavorite` | `scopeFavoritedByUser` (مع تحسينات) |
| `scopeWhereHasFavorites` | `scopeFavoritedByUser` |
| `scopeWhereHasUserFavorite` | `scopeFavoritedByUser` |
| `scopeWhereHasUserFavorites` | `scopeFavoritedByUser` |

**ملاحظة حول الدوال القديمة `whereHasFavorite` ونظيراتها:** تم تحسينها لدعم معامل `$isForceUser` للتحكم في سلوك الاستعلام عند عدم وجود مستخدم، مع قراءة القيمة الافتراضية من الإعدادات (`nano.markable::favoriteable.scope.where_has_favorite.is_force_user`). السلوك الافتراضي (`true`) يضيف شرطاً مستحيلاً لعدم إرجاع أي نتائج (يتوافق مع السلوك الأصلي).

---

## 8. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكن الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountFavorites` أو `sortByCountFavorites`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsFavoritedByUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة حول `withTrashed`:** إذا أضفت `SoftDeletes` إلى موديل `Favorite` في المستقبل، يمكن تفعيل خيار `$withTrashed` في النطاقات لتضمين السجلات المحذوفة.

---

## 9. الخاتمة

`FavoriteableModel` هو حل متكامل وقوي لإدارة نظام المفضلات في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة تفضيل متطورة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.