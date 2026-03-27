# توثيق سلوك `BookmarkableModel`

---

## 1. مقدمة

`BookmarkableModel` هو سلوك متقدم يضيف نظام إشارات مرجعية متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمستخدمين حفظ الكائنات المختلفة (مثل المنتجات، المقالات، الصفحات) في قوائم مخصصة عبر حقل `value` (مثل `favorite`, `read_later`, `wishlist`). يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة توفير مرجعية متطورة بمرونة وأداء عالٍ.

### الفوائد الرئيسية

- **نظام إشارات مرجعية متعدد الأشكال** – يمكن جعل أي موديل قابلاً للإضافة إلى الإشارات المرجعية.
- **دعم أنواع متعددة** – إمكانية تصنيف الإشارات عبر حقل `value` (مثل `favorite`, `read_later`) مع إمكانية تعيين القيم المسموحة من الإعدادات.
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد الإشارات، أحدث إشارة، أو أقدم إشارة.
- **نطاقات تصفية** – تصفية الكائنات التي أضافها مستخدم معين إلى الإشارات، أو التي لها إشارات، أو التي لا إشارات لها.
- **إضافة عمود `is_bookmarked_by_user`** – يظهر في نتائج الاستعلامات ما إذا كان المستخدم الحالي قد أضاف الكائن إلى الإشارات، دون الحاجة إلى تحميل العلاقة.
- **واجهة متسقة** – دوال مشابهة لسلوكيات المفضلات والإعجابات والتفاعلات (FavoriteableModel, LikeableModel, ReactionableModel) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`sortByCountBookmarksOld`, `withSortByCountBookmarks`, `whereHasBookmark`، إلخ) لضمان عدم كسر الكود القديم، مع دعم تحسينات إضافية للتحكم في السلوك عند عدم وجود مستخدم.

---

## 2. متطلبات الاستخدام

1. وجود موديل `Nano\Markable\Models\Bookmark` (الذي يمثل سجل الإشارة المرجعية) مع جدول `nano_markable_bookmarks`.
2. الموديل المراد جعله قابلاً للإضافة إلى الإشارات المرجعية يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام الإشارات (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: إذا كنت ترغب في دعم أنواع متعددة من الإشارات، قم بتعيين القيم المسموحة عبر الإعدادات (`nano.markable::allowed_values.bookmark`).

---

## 3. إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في سياق نانوسوفت). السلوك يتضمن Trait يحمل النطاقات والدوال المساعدة، لذا بعد الإضافة يصبح الموديل قادراً على استخدام جميع الميزات.

### مثال: جعل موديل `Product` قابلاً للإضافة إلى الإشارات المرجعية

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\BookmarkableModel;

class Product extends Model
{
    public $implement = [
        BookmarkableModel::class
    ];
}
```

**ملاحظة:** إذا كان الموديل يستخدم نظام `ExtensionBase` (كما هو الحال في تطبيقات نانوسوفت)، فإن الإضافة تتم عبر `$implement` كما هو موضح.

---

## 4. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقة التالية:

```php
$product->bookmarks() // جميع سجلات الإشارات المرجعية (كل الأنواع)
```

كما يتوفر خصائص محسوبة ديناميكية (تُضاف تلقائياً):

- `$product->bookmarks_count` – عدد الإشارات (يمكن تمرير `value` عبر الباراميتر).
- `$product->is_bookmarked()` – هل المستخدم الحالي أضاف الكائن إلى الإشارات (يمكن تمرير `value`).
- `$product->getBookmarked()` – إرجاع سجل الإشارة الخاص بالمستخدم الحالي (إن وجد).

---

## 5. الدوال والنطاقات المتاحة

### 5.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildBookmarkCountSubQuery($value = null)`
- **الوصف**: يبني استعلام فرعي لحساب عدد سجلات الإشارات (COUNT).
- **المعاملات**:
  - `$value` (string|null): تصفية حسب نوع الإشارة (مثل `favorite`, `read_later`).

#### `protected function buildBookmarkLatestDateSubQuery($field = 'created_at', $value = null)`
- **الوصف**: يبني استعلام فرعي لأحدث تاريخ إشارة (MAX).

#### `protected function buildBookmarkEarliestDateSubQuery(...)` – تستخدم `MIN(field)` للحصول على أقدم تاريخ.

### 5.2 نطاقات عدد الإشارات (COUNT)

#### `scopeAddCountBookmarks(Builder $query, $orderDirection = 'DESC', $columnName = 'bookmarks_count', $value = null)`
- **الوصف**: يضيف عموداً يحتوي على عدد سجلات الإشارات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountBookmarks()->get();
  foreach ($products as $product) {
      echo $product->bookmarks_count;
  }
  ```

#### `scopeSortByCountBookmarks(...)` – يرتب النتائج حسب عدد الإشارات (دون إضافة العمود).

#### `scopeWithCountBookmarks(...)` – يجمع بين `addCountBookmarks` و `sortByCountBookmarks` (إضافة العمود والترتيب معاً).

**مثال**:
```php
$products = Product::withCountBookmarks('DESC', 'bookmarks_count', 'favorite')->get(); // إشارات من نوع favorite فقط
```

### 5.3 نطاقات تاريخ الإشارة

#### `scopeAddLatestBookmark(...)` – يضيف عموداً بأحدث تاريخ إشارة (اختيارياً تحديد الحقل عبر `$field`).
#### `scopeSortByLatestBookmark(...)` – ترتيب حسب أحدث تاريخ إشارة.
#### `scopeWithLatestBookmark(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::withLatestBookmark('DESC', 'last_bookmark', 'favorite')->get();
```

#### `scopeAddEarliestBookmark(...)` – يضيف عموداً بأقدم تاريخ إشارة.
#### `scopeSortByEarliestBookmark(...)` – ترتيب حسب أقدم تاريخ إشارة.
#### `scopeWithEarliestBookmark(...)` – إضافة العمود والترتيب معاً.

### 5.4 نطاقات التصفية حسب المستخدم

#### `scopeBookmarkedByUser(Builder $query, $user = null, $value = null)`
- **الوصف**: تصفية الكائنات التي أضافها مستخدم معين إلى الإشارات المرجعية (إذا لم يُمرر `$user`، يستخدم المستخدم الحالي).
- **مثال**:
  ```php
  // المنتجات التي أضافها المستخدم الحالي إلى الإشارات (أي نوع)
  $products = Product::bookmarkedByUser()->get();
  
  // المنتجات التي أضافها مستخدم محدد كنوع 'read_later'
  $products = Product::bookmarkedByUser($user, 'read_later')->get();
  ```

#### `scopeNotBookmarkedByUser(...)` – تصفية الكائنات التي **لم** يضفها مستخدم معين إلى الإشارات.

### 5.5 نطاقات التصفية حسب وجود إشارات

#### `scopeHasBookmarks(Builder $query, $minCount = 1, $value = null)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من الإشارات (مع خيارات التصفية حسب `value`).
- **مثال**:
  ```php
  // منتجات لها 5 إشارات من نوع 'favorite' على الأقل
  $products = Product::hasBookmarks(5, 'favorite')->get();
  ```

#### `scopeHasNoBookmarks(...)` – تصفية الكائنات التي لا إشارات لها.

### 5.6 نطاق إضافة عمود حالة الإشارة للمستخدم الحالي

#### `scopeWithIsBookmarkedByUser(Builder $query, $user = null, $columnName = 'is_bookmarked_by_user', $value = null)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد قد أضاف الكائن إلى الإشارات المرجعية.
- **مثال**:
  ```php
  $products = Product::withIsBookmarkedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_bookmarked_by_user) {
          echo "هذا المنتج في إشاراتك";
      }
  }
  ```

### 5.7 نطاقات خاصة

#### `scopeTopBookmarked(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null)`
- **الوصف**: يجلب الكائنات الأكثر إشارة (حسب عدد الإشارات) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topBookmarked(10, 'DESC', 'favorite')->get();
  ```

### 5.8 دوال إحصائية مساعدة

#### `getTotalBookmarks($value = null)`
- **الوصف**: إجمالي عدد الإشارات المرجعية (مجموع `COUNT`) حسب خيارات التصفية.
- **المخرجات**: عدد صحيح.

#### `getBookmarksCountByType($value = null)`
- **الوصف**: توزيع الإشارات حسب نوع المستخدم (`user_type`).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.

#### `getBookmarkersUsers($value = null)`
- **الوصف**: قائمة المستخدمين الذين أضافوا الكائن إلى الإشارات المرجعية (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` من كائنات المستخدمين.

---

## 6. أمثلة عملية شاملة

### مثال 1: إضافة إشارة مرجعية والتحقق

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Bookmark;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة إشارة مرجعية من نوع 'favorite'
$bookmark = Bookmark::add($product, $user, 'favorite');

// إضافة إشارة مرجعية من نوع 'read_later' مع ميتاداتا
$bookmark = Bookmark::add($product, $user, 'read_later', ['notes' => 'اقرأ هذا لاحقاً']);

// التحقق مما إذا كان المستخدم قد أضاف المنتج إلى الإشارات (أي نوع)
if ($product->isBookmarked()) {
    echo "المنتج في إشاراتك";
}

// التحقق من نوع معين
if ($product->isBookmarked(null, 'read_later')) {
    echo "المنتج في قائمة القراءة لاحقاً";
}
```

### مثال 2: إزالة إشارة مرجعية

```php
// إزالة الإشارة (أي نوع)
Bookmark::remove($product, $user);

// إزالة إشارة من نوع 'favorite' فقط
Bookmark::remove($product, $user, 'favorite');
```

### مثال 3: ترتيب المنتجات حسب عدد الإشارات

```php
$topProducts = Product::topBookmarked(10)->get();
```

### مثال 4: إضافة عمود `is_bookmarked_by_user` في قائمة المنتجات

```php
$products = Product::withIsBookmarkedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_bookmarked_by_user ? 'مفضل' : 'غير مفضل');
}
```

### مثال 5: تصفية المنتجات التي أضافها المستخدم الحالي إلى الإشارات

```php
$myBookmarks = Product::bookmarkedByUser()->get();
```

### مثال 6: الإحصائيات

```php
$product = Product::find(1);
echo "عدد الإشارات من نوع 'favorite': " . $product->getTotalBookmarks('favorite');
print_r($product->getBookmarksCountByType('favorite')->toArray());
$users = $product->getBookmarkersUsers('favorite');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين النطاقات المتقدمة

```php
// المنتجات التي لها أكثر من 5 إشارات من نوع 'read_later'، مرتبة حسب أحدث إشارة
$products = Product::hasBookmarks(5, 'read_later')
    ->sortByLatestBookmark('DESC', 'latest_read_later_at')
    ->get();
```

---

## 7. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة، تم توفير دوال تغليف في الكلاس الرئيسي. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

| الدالة القديمة | الدالة الجديدة |
|----------------|----------------|
| `scopeSortByCountBookmarksOld` | `scopeSortByCountBookmarks` |
| `scopeWithSortByCountBookmarks` | `scopeWithCountBookmarks` |
| `scopeAddSortByCountBookmarks` | `scopeAddCountBookmarks` |
| `scopeSortByCreatedAtBookmarks` | `scopeSortByLatestBookmark` |
| `scopeAddSortByCreatedAtBookmarks` | `scopeAddLatestBookmark` |
| `scopeWithSortByCreatedAtBookmarks` | `scopeWithLatestBookmark` |
| `scopeWhereHasBookmark` | `scopeBookmarkedByUser` (مع تحسينات) |
| `scopeWhereHasBookmarks` | `scopeBookmarkedByUser` |
| `scopeWhereHasUserBookmark` | `scopeBookmarkedByUser` |
| `scopeWhereHasUserBookmarks` | `scopeBookmarkedByUser` |

**ملاحظة حول الدوال القديمة `whereHasBookmark` ونظيراتها:** تم تحسينها لدعم معامل `$isForceUser` للتحكم في سلوك الاستعلام عند عدم وجود مستخدم، مع قراءة القيمة الافتراضية من الإعدادات (`nano.markable::bookmarkable.scope.where_has_bookmark.is_force_user`). السلوك الافتراضي (`true`) يضيف شرطاً مستحيلاً لعدم إرجاع أي نتائج (يتوافق مع السلوك الأصلي).

---

## 8. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكن الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountBookmarks` أو `sortByCountBookmarks`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsBookmarkedByUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة:** إذا أضفت `SoftDeletes` إلى موديل `Bookmark` في المستقبل، يمكن تعديل النطاقات لدعم `withTrashed` بسهولة.

---

## 9. الخاتمة

`BookmarkableModel` هو حل متكامل وقوي لإدارة نظام الإشارات المرجعية في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة توفير مرجعية متطورة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.