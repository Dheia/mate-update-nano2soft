# توثيق سلوك `ReviewRateable` 

---

## 1. مقدمة

`ReviewRateable` هو سلوك متقدم يضيف نظام تقييم متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمستخدمين إضافة تقييمات (نجوم) مع مراجعات نصية للكائنات المختلفة (مثل المنتجات، المقالات، الخدمات) مع دعم شامل لخيارات القبول (approved)، النشاط (is_active)، الإيجابية/السالبة (is_positive)، والحذف الناعم (soft delete). يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة تقييم متطورة بمرونة وأداء عالٍ.

### الفوائد الرئيسية

- **نظام تقييم متعدد الأشكال** – يمكن جعل أي موديل قابلاً للتقييم.
- **مرونة في حالات التقييم** – دعم حقول `rating` (القيمة الرقمية)، `approved` (الموافقة)، `is_active` (النشاط)، `is_positive` (الإيجابية/السالبة) لتمثيل حالات متعددة.
- **دعم التقييمات المحذوفة (Soft Deletes)** – إمكانية تضمين التقييمات المحذوفة في الاستعلامات حسب الحاجة.
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد التقييمات، متوسط التقييم، أحدث تقييم، أو أقدم تقييم، مع خيارات تصفية القبول والنشاط والإيجابية.
- **نطاقات تصفية** – تصفية الكائنات التي قيمها مستخدم معين، أو التي لها تقييمات، أو التي لا تقييمات لها.
- **إضافة عمود `is_rated_by_user`** – يظهر في نتائج الاستعلامات ما إذا كان المستخدم الحالي قد قيم الكائن، دون الحاجة إلى تحميل العلاقة.
- **واجهة متسقة** – دوال مشابهة لسلوكي المتابعة والزيارات (FollowableModel و VisitModel) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`sortByRating`, `withRating`, `sortByCreatedAtReviews`، إلخ) لضمان عدم كسر الكود القديم.

---

## 2. متطلبات الاستخدام

1. وجود موديل `Nano\Reviews\Models\Review` (الذي يمثل سجل التقييم) مع جدول `nano_reviews_reviews`.
2. الموديل المراد جعله قابلاً للتقييم يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام التقييم (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: استخدام `SoftDeletes` في موديل `Review` للاستفادة من خيار `withTrashed`.

---

## 3. إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في سياق نانوسوفت). السلوك يتضمن Trait يحمل النطاقات والدوال المساعدة، لذا بعد الإضافة يصبح الموديل قادراً على استخدام جميع الميزات.

### مثال: جعل موديل `Product` قابلاً للتقييم

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Reviews\Behaviors\ReviewRateable;

class Product extends Model
{
    public $implement = [
        ReviewRateable::class
    ];
}
```

**ملاحظة:** إذا كان الموديل يستخدم نظام `ExtensionBase` (كما هو الحال في تطبيقات نانوسوفت)، فإن الإضافة تتم عبر `$implement` كما هو موضح.

---

## 4. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقة التالية:

```php
$product->ratings() // يعيد علاقة morphMany تربط سجلات Review بهذا الموديل
```

كما يتوفر خصائص محسوبة لسهولة الوصول إلى إحصائيات التقييمات:

```php
$product->ratings_count   // عدد سجلات التقييمات
$product->avg_rating      // متوسط التقييم (في حالة إضافة العمود)
$product->is_rated_by_user // هل قام المستخدم الحالي بالتقييم
```

---

## 5. الدوال والنطاقات المتاحة

### 5.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildReviewCountSubQuery($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: يبني استعلام فرعي لحساب عدد سجلات التقييمات (COUNT).
- **المعاملات**:
  - `$onlyActive` (bool|null): `true` للتقييمات النشطة فقط، `false` للتقييمات غير النشطة فقط، `null` للكل.
  - `$withTrashed` (bool): تضمين التقييمات المحذوفة.
  - `$onlyApproved` (bool|null): `true` للتقييمات المعتمدة فقط، `false` للغير معتمدة، `null` للكل.
  - `$isPositive` (bool|null): `true` للتقييمات الإيجابية فقط، `false` للسالبة، `null` للكل.
  - `$ratingValue` (int|null): تصفية حسب قيمة التقييم (مثلاً 5).

#### `protected function buildReviewAvgSubQuery(...)` – نفس المعاملات، تستخدم `AVG(rating)` مع إمكانية التقريب.

#### `protected function buildReviewLatestDateSubQuery($field = 'created_at', ...)` – تستخدم `MAX(field)` للحصول على أحدث تاريخ.

#### `protected function buildReviewEarliestDateSubQuery(...)` – تستخدم `MIN(field)` للحصول على أقدم تاريخ.

### 5.2 نطاقات عدد التقييمات (COUNT)

#### `scopeAddCountReviews(Builder $query, $orderDirection = 'DESC', $columnName = 'reviews_count', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: يضيف عموداً يحتوي على عدد سجلات التقييمات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountReviews()->get();
  foreach ($products as $product) {
      echo $product->reviews_count;
  }
  ```

#### `scopeSortByCountReviews(...)` – يرتب النتائج حسب عدد التقييمات (دون إضافة العمود).

#### `scopeWithCountReviews(...)` – يجمع بين `addCountReviews` و `sortByCountReviews` (إضافة العمود والترتيب معاً).

**مثال**:
```php
$products = Product::withCountReviews('DESC', 'reviews_count', true)->get(); // التقييمات النشطة فقط
```

### 5.3 نطاقات متوسط التقييم (AVG)

#### `scopeAddAvgRating(Builder $query, $orderDirection = 'DESC', $columnName = 'avg_rating', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null, $round = 2)`
- **الوصف**: يضيف عموداً بمتوسط التقييم (مقرباً إلى عدد منازل عشرية).
- **مثال**:
  ```php
  $products = Product::addAvgRating()->get();
  echo $products->first()->avg_rating;
  ```

#### `scopeSortByAvgRating(...)` – ترتيب حسب متوسط التقييم.
#### `scopeWithAvgRating(...)` – إضافة العمود والترتيب معاً.

### 5.4 نطاقات أحدث تاريخ تقييم

#### `scopeAddLatestReview(...)` – يضيف عموداً بأحدث تاريخ تقييم (اختيارياً تحديد الحقل عبر `$field`).
#### `scopeSortByLatestReview(...)` – ترتيب حسب أحدث تاريخ تقييم.
#### `scopeWithLatestReview(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::withLatestReview('DESC', 'last_review', true, false, true, null, null, 'created_at')->get();
```

### 5.5 نطاقات أقدم تاريخ تقييم

#### `scopeAddEarliestReview(...)` – يضيف عموداً بأقدم تاريخ تقييم.
#### `scopeSortByEarliestReview(...)` – ترتيب حسب أقدم تاريخ تقييم.
#### `scopeWithEarliestReview(...)` – إضافة العمود والترتيب معاً.

### 5.6 نطاقات التصفية حسب المستخدم

#### `scopeRatedByUser(Builder $query, $user = null, $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: تصفية الكائنات التي قيمها مستخدم معين (إذا لم يُمرر `$user`، يستخدم المستخدم الحالي).
- **مثال**:
  ```php
  // المنتجات التي قيمها المستخدم الحالي
  $products = Product::ratedByUser()->get();
  
  // المنتجات التي قيمها مستخدم محدد بتقييم إيجابي فقط
  $products = Product::ratedByUser($user, null, false, null, true)->get();
  ```

#### `scopeNotRatedByUser(...)` – تصفية الكائنات التي **لم** يقيمها مستخدم معين.

### 5.7 نطاقات التصفية حسب وجود تقييمات

#### `scopeHasRatings(Builder $query, $minCount = 1, $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من التقييمات (مع خيارات التصفية).
- **مثال**:
  ```php
  // منتجات لها 5 تقييمات معتمدة على الأقل
  $products = Product::hasRatings(5, true, false, true)->get();
  ```

#### `scopeHasNoRatings(...)` – تصفية الكائنات التي لا تقييمات لها.

### 5.8 نطاق إضافة عمود حالة التقييم للمستخدم الحالي

#### `scopeWithIsRatedByUser(Builder $query, $user = null, $columnName = 'is_rated_by_user', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد قد قيم الكائن.
- **مثال**:
  ```php
  $products = Product::withIsRatedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_rated_by_user) {
          echo "لقد قيمت هذا المنتج";
      }
  }
  ```

### 5.9 نطاقات خاصة

#### `scopeTopRated(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null, $round = 2)`
- **الوصف**: يجلب الكائنات الأعلى تقييماً (حسب متوسط التقييم) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topRated(10, 'DESC', true, false, true)->get();
  ```

#### `scopeTopRatedByCount(Builder $query, $limit = 10, $orderDirection = 'DESC', $onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: يجلب الكائنات الأكثر تقييماً (حسب عدد التقييمات) مع تحديد الحد.

### 5.10 دوال إحصائية مساعدة

#### `getTotalRatings($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: إجمالي عدد التقييمات (مجموع `rating`) حسب خيارات التصفية.
- **المخرجات**: عدد صحيح.

#### `getAverageRating($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null, $round = null)`
- **الوصف**: متوسط التقييم مع خيارات التصفية.

#### `getRatingsCountByType($onlyActive = null, $withTrashed = false, $onlyApproved = null, $isPositive = null, $ratingValue = null)`
- **الوصف**: توزيع التقييمات حسب نوع المستخدم (`user_type`).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.

#### `getRatersUsers(...)`
- **الوصف**: قائمة المستخدمين الذين قيموا الكائن (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` من كائنات المستخدمين.

### 5.11 دوال أساسية في السلوك (غير موجودة في الـ Trait)

#### `ratingdBy()`
- **الوصف**: يعيد مجموعة المستخدمين الذين قيموا الكائن الحالي.
- **المخرجات**: `Collection` من المستخدمين (مع تحميل `user`).

#### `isRating($user = null)`
- **الوصف**: يتحقق مما إذا كان المستخدم (أو المستخدم الحالي) قد قيم الكائن.
- **المخرجات**: `bool`.

#### `getRating($user = null)`
- **الوصف**: يعيد سجل التقييم الخاص بالمستخدم (إن وجد).
- **المخرجات**: كائن `Review` أو `null`.

#### `addRating($data, $user, $parent = null): Review`
- **الوصف**: إضافة تقييم جديد للكائن. يمكن تمرير `$data` كـ `array` تحتوي على `rating`, `title`, `content`, `approved`, `is_positive` إلخ.
- **المخرجات**: كائن `Review` المُنشأ.

#### `toggleRating($data, $user = null)`
- **الوصف**: إذا كان المستخدم قد قيم، يقوم بتحديث التقييم؛ وإلا يضيف تقييماً جديداً.
- **المخرجات**: كائن `Review`.

#### `updateRating($id, $data): Review`
- **الوصف**: تحديث بيانات تقييم موجود.

#### `deleteRating($id): ?bool`
- **الوصف**: حذف تقييم معين (Soft delete).

#### `countRating()`
- **الوصف**: عدد التقييمات (اختيارياً المعتمدة فقط).

#### `averageRating($round = null, $onlyApproved = false): Collection`
- **الوصف**: متوسط التقييم (دالة مساعدة قديمة، يُفضل استخدام `getAverageRating`).

---

## 6. أمثلة عملية شاملة

### مثال 1: إضافة تقييم وتحديثه

```php
$user = Auth::getUser();
$product = Product::find(1);

// إضافة تقييم جديد (5 نجوم، مراجعة نصية)
$review = $product->addRating([
    'rating' => 5,
    'title' => 'منتج رائع',
    'content' => 'جودة عالية وسعر مناسب',
    'approved' => true,
    'is_positive' => true,
], $user);

// تحديث التقييم
$product->updateRating($review->id, ['rating' => 4, 'content' => 'تحديث بعد الاستخدام']);

// التبديل بين التقييم والإلغاء (إذا كان هناك تقييم، يحدثه؛ وإلا يضيف)
$product->toggleRating(['rating' => 5], $user);
```

### مثال 2: ترتيب المنتجات حسب متوسط التقييم

```php
$topProducts = Product::topRated(10, 'DESC', true, false, true)->get(); // التقييمات المعتمدة فقط

// إضافة عمود متوسط التقييم مع الترتيب
$products = Product::withAvgRating('DESC', 'avg_rating', null, false, true)->get();
```

### مثال 3: إضافة عمود `is_rated_by_user` في قائمة المنتجات

```php
$products = Product::withIsRatedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_rated_by_user ? 'قيمته' : 'لم تقيمه');
}
```

### مثال 4: تصفية المنتجات التي قيمها المستخدم الحالي

```php
$myRatedProducts = Product::ratedByUser()->get();
```

### مثال 5: استخدام التصفية حسب التقييمات الإيجابية فقط

```php
$products = Product::hasRatings(1, true, false, null, true)->get(); // على الأقل تقييم إيجابي واحد
```

### مثال 6: إحصائيات التقييمات لمنتج معين

```php
$product = Product::find(1);

echo "إجمالي التقييمات: " . $product->getTotalRatings(true, false, true);
echo "متوسط التقييم: " . $product->getAverageRating(true, false, true);
echo "توزيع التقييمات حسب نوع المستخدم: ";
print_r($product->getRatingsCountByType(true, false, true)->toArray());

$users = $product->getRatersUsers(true, false, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين النطاقات المتقدمة

```php
// المنتجات التي لها أكثر من 5 تقييمات معتمدة، مرتبة حسب متوسط التقييم
$products = Product::hasRatings(5, true, false, true)
    ->sortByAvgRating('DESC', 'avg_rating')
    ->get();
```

---

## 7. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة، تم توفير دوال تغليف في الكلاس الرئيسي. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

| الدالة القديمة | الدالة الجديدة |
|----------------|----------------|
| `scopeSortByRating` | `scopeSortByAvgRating` |
| `scopeWithRating` | `scopeWithAvgRating` |
| `scopeAddSortByRating` | `scopeAddAvgRating` |
| `scopeWithSortByCountReviews` | `scopeWithCountReviews` |
| `scopeSortByCountReviewsOld` | `scopeSortByCountReviews` |
| `scopeSortByCreatedAtReviews` | `scopeSortByLatestReview` |
| `scopeAddSortByCreatedAtReviews` | `scopeAddLatestReview` |
| `scopeWithSortByCreatedAtReviews` | `scopeWithLatestReview` |

**ملاحظة:** النطاق `scopeSortByCountReviews` تم الاحتفاظ بنفس الاسم ولكن مع توقيع جديد يدعم باراميترات إضافية اختيارية، مما لا يسبب مشكلة للاستدعاءات القديمة.

---

## 8. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكن الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountReviews` أو `sortByCountReviews`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsRatedByUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة حول `withTrashed`:** عند استخدام `withTrashed = true` في أي من النطاقات أو الدوال الإحصائية، تأكد من أن موديل `Nano\Reviews\Models\Review` يستخدم الترايت `SoftDeletes`؛ وإلا ستظهر أخطاء في الاستعلام.

---

## 9. الخاتمة

`ReviewRateable` هو حل متكامل وقوي لإدارة نظام التقييمات في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة تقييم متقدمة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.