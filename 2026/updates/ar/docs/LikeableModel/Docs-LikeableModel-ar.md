# توثيق سلوك `LikeableModel` 

---

## 1. مقدمة

`LikeableModel` هو سلوك متقدم يضيف نظام إعجابات متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمستخدمين الإعجاب (Like) أو عدم الإعجاب (Dislike) بالكائنات المختلفة (مثل المنتجات، المقالات، التعليقات) مع دعم تمييز أنواع الإعجابات عبر حقل `value`. يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة تفاعل متطورة بمرونة وأداء عالٍ.

### الفوائد الرئيسية

- **نظام إعجابات متعدد الأشكال** – يمكن جعل أي موديل قابلاً للإعجاب.
- **دعم أنواع متعددة** – إمكانية تصنيف الإعجابات عبر حقل `value` (مثل `like`, `dislike`) مع إمكانية تعيين قيم افتراضية من الإعدادات.
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد الإعجابات، أحدث إعجاب، أو أقدم إعجاب.
- **نطاقات تصفية** – تصفية الكائنات التي أعجب بها مستخدم معين، أو التي لها إعجابات، أو التي لا إعجابات لها.
- **إضافة عمود `is_liked_by_user`** – يظهر في نتائج الاستعلامات ما إذا كان المستخدم الحالي قد أعجب بالكائن، دون الحاجة إلى تحميل العلاقة.
- **واجهة متسقة** – دوال مشابهة لسلوكيات المفضلات والتفاعلات الأخرى (FavoriteableModel, ReactionableModel) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`sortByCountLikesOld`, `withSortByCountLikes`, `whereHasLike`، إلخ) لضمان عدم كسر الكود القديم، مع دعم تحسينات إضافية للتحكم في السلوك عند عدم وجود مستخدم.

---

## 2. متطلبات الاستخدام

1. وجود موديل `Nano\Markable\Models\Like` (الذي يمثل سجل الإعجاب) مع جدول `nano_markable_likes`.
2. الموديل المراد جعله قابلاً للإعجاب يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام الإعجابات (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: إذا كنت ترغب في دعم أنواع متعددة من الإعجابات (`like` / `dislike`)، قم بتعيين القيم المسموحة عبر الإعدادات (`nano.markable::allowed_values.like`).

---

## 3. إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في سياق نانوسوفت). السلوك يتضمن Trait يحمل النطاقات والدوال المساعدة، لذا بعد الإضافة يصبح الموديل قادراً على استخدام جميع الميزات.

### مثال: جعل موديل `Product` قابلاً للإعجاب

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\LikeableModel;

class Product extends Model
{
    public $implement = [
        LikeableModel::class
    ];
}
```

**ملاحظة:** إذا كان الموديل يستخدم نظام `ExtensionBase` (كما هو الحال في تطبيقات نانوسوفت)، فإن الإضافة تتم عبر `$implement` كما هو موضح.

---

## 4. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقات التالية:

```php
$product->likes()      // جميع سجلات الإعجابات (كل الأنواع)
$product->my_likes()   // الإعجابات من نوع 'like' فقط
$product->my_dislikes() // الإعجابات من نوع 'dislike' فقط
```

كما يتوفر خصائص محسوبة ديناميكية (تُضاف تلقائياً):

- `$product->likes_count` – عدد الإعجابات (يمكن تمرير `value` عبر الباراميتر).
- `$product->is_liked()` – هل المستخدم الحالي أعجب بالكائن (قيمة افتراضية `like`).
- `$product->getLiked()` – إرجاع سجل الإعجاب الخاص بالمستخدم الحالي (إن وجد).

---

## 5. الدوال والنطاقات المتاحة

### 5.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildLikeCountSubQuery($value = null)`
- **الوصف**: يبني استعلام فرعي لحساب عدد سجلات الإعجابات (COUNT).
- **المعاملات**:
  - `$value` (string|null): تصفية حسب نوع الإعجاب (مثل `like` أو `dislike`).

#### `protected function buildLikeLatestDateSubQuery($field = 'created_at', $value = null)`
- **الوصف**: يبني استعلام فرعي لأحدث تاريخ إعجاب (MAX).

#### `protected function buildLikeEarliestDateSubQuery(...)` – تستخدم `MIN(field)` للحصول على أقدم تاريخ.

### 5.2 نطاقات عدد الإعجابات (COUNT)

#### `scopeAddCountLikes(Builder $query, $orderDirection = 'DESC', $columnName = 'likes_count', $value = null)`
- **الوصف**: يضيف عموداً يحتوي على عدد سجلات الإعجابات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountLikes()->get();
  foreach ($products as $product) {
      echo $product->likes_count;
  }
  ```

#### `scopeSortByCountLikes(...)` – يرتب النتائج حسب عدد الإعجابات (دون إضافة العمود). إذا لم يُمرر `$value`، يتم استخدام القيمة الافتراضية من `Like::getDefaultValue()`.

#### `scopeWithCountLikes(...)` – يجمع بين `addCountLikes` و `sortByCountLikes` (إضافة العمود والترتيب معاً).

**مثال**:
```php
$products = Product::withCountLikes('DESC', 'likes_count', 'like')->get(); // إعجابات من نوع like فقط
```

### 5.3 نطاقات تاريخ الإعجاب

#### `scopeAddLatestLike(...)` – يضيف عموداً بأحدث تاريخ إعجاب (اختيارياً تحديد الحقل عبر `$field`).
#### `scopeSortByLatestLike(...)` – ترتيب حسب أحدث تاريخ إعجاب.
#### `scopeWithLatestLike(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::withLatestLike('DESC', 'last_like', 'like')->get();
```

#### `scopeAddEarliestLike(...)` – يضيف عموداً بأقدم تاريخ إعجاب.
#### `scopeSortByEarliestLike(...)` – ترتيب حسب أقدم تاريخ إعجاب.
#### `scopeWithEarliestLike(...)` – إضافة العمود والترتيب معاً.

### 5.4 نطاقات التصفية حسب المستخدم

#### `scopeLikedByUser(Builder $query, $user = null, $value = null)`
- **الوصف**: تصفية الكائنات التي أعجب بها مستخدم معين (إذا لم يُمرر `$user`، يستخدم المستخدم الحالي).
- **مثال**:
  ```php
  // المنتجات التي أعجب بها المستخدم الحالي (إعجابات من نوع like افتراضياً)
  $products = Product::likedByUser()->get();
  
  // المنتجات التي أعجب بها مستخدم محدد كنوع 'like'
  $products = Product::likedByUser($user, 'like')->get();
  ```

#### `scopeNotLikedByUser(...)` – تصفية الكائنات التي **لم** يعجب بها مستخدم معين.

### 5.5 نطاقات التصفية حسب وجود إعجابات

#### `scopeHasLikes(Builder $query, $minCount = 1, $value = null)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من الإعجابات (مع خيارات التصفية حسب `value`).
- **مثال**:
  ```php
  // منتجات لها 5 إعجابات من نوع 'like' على الأقل
  $products = Product::hasLikes(5, 'like')->get();
  ```

#### `scopeHasNoLikes(...)` – تصفية الكائنات التي لا إعجابات لها.

### 5.6 نطاق إضافة عمود حالة الإعجاب للمستخدم الحالي

#### `scopeWithIsLikedByUser(Builder $query, $user = null, $columnName = 'is_liked_by_user', $value = null)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد قد أعجب بالكائن.
- **مثال**:
  ```php
  $products = Product::withIsLikedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_liked_by_user) {
          echo "هذا المنتج أعجبك";
      }
  }
  ```

### 5.7 نطاقات خاصة

#### `scopeTopLiked(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null)`
- **الوصف**: يجلب الكائنات الأكثر إعجاباً (حسب عدد الإعجابات) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topLiked(10, 'DESC', 'like')->get();
  ```

### 5.8 دوال إحصائية مساعدة

#### `getTotalLikes($value = null)`
- **الوصف**: إجمالي عدد الإعجابات (مجموع `COUNT`) حسب خيارات التصفية.
- **المخرجات**: عدد صحيح.

#### `getLikesCountByType($value = null)`
- **الوصف**: توزيع الإعجابات حسب نوع المستخدم (`user_type`).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.

#### `getLikersUsers($value = null)`
- **الوصف**: قائمة المستخدمين الذين أعجبوا بالكائن (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` من كائنات المستخدمين.

---

## 6. أمثلة عملية شاملة

### مثال 1: إضافة إعجاب والتحقق

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Like;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة إعجاب (النوع الافتراضي 'like')
$like = Like::add($product, $user);

// إضافة عدم إعجاب (نوع 'dislike')
$dislike = Like::add($product, $user, 'dislike');

// التحقق مما إذا كان المستخدم قد أعجب بالمنتج
if ($product->isLiked()) {
    echo "المنتج أعجبك";
}

// الحصول على عدد الإعجابات
echo $product->likes_count; // كل الإعجابات
echo $product->likes_count('like'); // إعجابات من نوع like فقط
```

### مثال 2: إزالة إعجاب

```php
// إزالة الإعجاب (النوع الافتراضي)
Like::remove($product, $user);

// إزالة عدم الإعجاب فقط
Like::remove($product, $user, 'dislike');
```

### مثال 3: ترتيب المنتجات حسب عدد الإعجابات

```php
$topProducts = Product::topLiked(10)->get();
```

### مثال 4: إضافة عمود `is_liked_by_user` في قائمة المنتجات

```php
$products = Product::withIsLikedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_liked_by_user ? 'أعجبك' : 'لم يعجبك');
}
```

### مثال 5: تصفية المنتجات التي أعجب بها المستخدم الحالي

```php
$myLikedProducts = Product::likedByUser()->get();
```

### مثال 6: الإحصائيات

```php
$product = Product::find(1);
echo "عدد الإعجابات: " . $product->getTotalLikes('like');
echo "عدد عدم الإعجاب: " . $product->getTotalLikes('dislike');
print_r($product->getLikesCountByType('like')->toArray());
$users = $product->getLikersUsers('like');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين النطاقات المتقدمة

```php
// المنتجات التي لها أكثر من 5 إعجابات من نوع 'like'، مرتبة حسب أحدث إعجاب
$products = Product::hasLikes(5, 'like')
    ->sortByLatestLike('DESC', 'latest_like_at')
    ->get();
```

---

## 7. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة، تم توفير دوال تغليف في الكلاس الرئيسي. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

| الدالة القديمة | الدالة الجديدة |
|----------------|----------------|
| `scopeSortByCountLikesOld` | `scopeSortByCountLikes` |
| `scopeWithSortByCountLikes` | `scopeWithCountLikes` |
| `scopeSortByCreatedAtLikes` | `scopeSortByLatestLike` |
| `scopeAddSortByCreatedAtLikes` | `scopeAddLatestLike` |
| `scopeWithSortByCreatedAtLikes` | `scopeWithLatestLike` |
| `scopeWhereHasLike` | `scopeLikedByUser` (مع تحسينات) |
| `scopeWhereHasLikes` | `scopeLikedByUser` |
| `scopeWhereHasUserLike` | `scopeLikedByUser` |
| `scopeWhereHasUserLikes` | `scopeLikedByUser` |

**ملاحظة حول الدوال القديمة `whereHasLike` ونظيراتها:** تم تحسينها لدعم معامل `$isForceUser` للتحكم في سلوك الاستعلام عند عدم وجود مستخدم، مع قراءة القيمة الافتراضية من الإعدادات (`nano.markable::likeable.scope.where_has_like.is_force_user`). السلوك الافتراضي (`true`) يضيف شرطاً مستحيلاً لعدم إرجاع أي نتائج (يتوافق مع السلوك الأصلي).

---

## 8. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكن الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountLikes` أو `sortByCountLikes`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsLikedByUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة:** إذا أضفت `SoftDeletes` إلى موديل `Like` في المستقبل، يمكن تعديل النطاقات لدعم `withTrashed` بسهولة.

---

## 9. الخاتمة

`LikeableModel` هو حل متكامل وقوي لإدارة نظام الإعجابات في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة تفاعل متطورة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.