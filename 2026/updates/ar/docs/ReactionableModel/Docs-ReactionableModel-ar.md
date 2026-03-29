# توثيق سلوك `ReactionableModel` 

---

## 1. مقدمة

`ReactionableModel` هو سلوك متقدم يضيف نظام تفاعلات متعدد الأشكال (polymorphic) إلى أي موديل في تطبيقات نانوسوفت. يتيح هذا السلوك للمستخدمين التفاعل مع الكائنات المختلفة (مثل المنتجات، المقالات، التعليقات) بأنواع متعددة من التفاعلات (مثل `like`, `heart`, `wow`, `angry`) عبر حقل `value`. يأتي السلوك مزوداً بمجموعة غنية من النطاقات والدوال المساعدة التي تسهل بناء أنظمة تفاعل متطورة بمرونة وأداء عالٍ.

### الفوائد الرئيسية

- **نظام تفاعلات متعدد الأشكال** – يمكن جعل أي موديل قابلاً للتفاعل.
- **دعم أنواع متعددة** – إمكانية تصنيف التفاعلات عبر حقل `value` (مثل `like`, `heart`, `wow`) مع إمكانية تعيين القيم المسموحة من الإعدادات.
- **نطاقات ترتيب متقدمة** – ترتيب الكائنات حسب عدد التفاعلات، أحدث تفاعل، أو أقدم تفاعل.
- **نطاقات تصفية** – تصفية الكائنات التي تفاعل معها مستخدم معين، أو التي لها تفاعلات، أو التي لا تفاعلات لها.
- **إضافة عمود `is_reacted_by_user`** – يظهر في نتائج الاستعلامات ما إذا كان المستخدم الحالي قد تفاعل مع الكائن، دون الحاجة إلى تحميل العلاقة.
- **واجهة متسقة** – دوال مشابهة لسلوكيات المفضلات والإعجابات (FavoriteableModel, LikeableModel) لتجربة تطوير موحدة.
- **تحسين الأداء** – استخدام استعلامات فرعية (subqueries) فعالة وتجنب تحميل العلاقات بالكامل حيثما أمكن.
- **توافق عكسي** – الاحتفاظ بأسماء النطاقات القديمة (`sortByCountReactionsOld`, `withSortByCountReactions`, `whereHasReaction`، إلخ) لضمان عدم كسر الكود القديم، مع دعم تحسينات إضافية للتحكم في السلوك عند عدم وجود مستخدم.

---

## 2. متطلبات الاستخدام

1. وجود موديل `Nano\Markable\Models\Reaction` (الذي يمثل سجل التفاعل) مع جدول `nano_markable_reactions`.
2. الموديل المراد جعله قابلاً للتفاعل يجب أن يكون من نوع Eloquent (أو `October\Rain\Database\Model` في حالة استخدام OctoberCMS).
3. أن يكون المستخدمون (مثل موديل `User`) قادرين على استخدام نظام التفاعلات (أي أنهم يستوفون واجهة `Authenticatable`).
4. اختياري: إذا كنت ترغب في دعم أنواع متعددة من التفاعلات، قم بتعيين القيم المسموحة عبر الإعدادات (`nano.markable::allowed_values.reaction`).

---

## 3. إضافة السلوك إلى موديل

يمكن إضافة السلوك إلى أي موديل باستخدام خاصية `$implement` (في سياق نانوسوفت). السلوك يتضمن Trait يحمل النطاقات والدوال المساعدة، لذا بعد الإضافة يصبح الموديل قادراً على استخدام جميع الميزات.

### مثال: جعل موديل `Product` قابلاً للتفاعل

```php
<?php namespace Nano\Shop\Models;

use Model;
use Nano\Markable\Behaviors\ReactionableModel;

class Product extends Model
{
    public $implement = [
        ReactionableModel::class
    ];
}
```

**ملاحظة:** إذا كان الموديل يستخدم نظام `ExtensionBase` (كما هو الحال في تطبيقات نانوسوفت)، فإن الإضافة تتم عبر `$implement` كما هو موضح.

---

## 4. الهيكل الأساسي

بعد إضافة السلوك، يصبح للموديل العلاقة التالية:

```php
$product->reactions() // جميع سجلات التفاعلات (كل الأنواع)
```

كما يتوفر خصائص محسوبة ديناميكية (تُضاف تلقائياً):

- `$product->reactions_count` – عدد التفاعلات (يمكن تمرير `value` عبر الباراميتر).
- `$product->is_reacted()` – هل المستخدم الحالي تفاعل مع الكائن (يمكن تمرير `value`).
- `$product->getReacted()` – إرجاع سجل التفاعل الخاص بالمستخدم الحالي (إن وجد).

---

## 5. الدوال والنطاقات المتاحة

### 5.1 دوال بناء الاستعلامات الفرعية (داخلية)

#### `protected function buildReactionCountSubQuery($value = null)`
- **الوصف**: يبني استعلام فرعي لحساب عدد سجلات التفاعلات (COUNT).
- **المعاملات**:
  - `$value` (string|null): تصفية حسب نوع التفاعل (مثل `like`, `heart`).

#### `protected function buildReactionLatestDateSubQuery($field = 'created_at', $value = null)`
- **الوصف**: يبني استعلام فرعي لأحدث تاريخ تفاعل (MAX).

#### `protected function buildReactionEarliestDateSubQuery(...)` – تستخدم `MIN(field)` للحصول على أقدم تاريخ.

### 5.2 نطاقات عدد التفاعلات (COUNT)

#### `scopeAddCountReactions(Builder $query, $orderDirection = 'DESC', $columnName = 'reactions_count', $value = null)`
- **الوصف**: يضيف عموداً يحتوي على عدد سجلات التفاعلات إلى نتيجة الاستعلام (بدون ترتيب).
- **مثال**:
  ```php
  $products = Product::addCountReactions()->get();
  foreach ($products as $product) {
      echo $product->reactions_count;
  }
  ```

#### `scopeSortByCountReactions(...)` – يرتب النتائج حسب عدد التفاعلات (دون إضافة العمود).

#### `scopeWithCountReactions(...)` – يجمع بين `addCountReactions` و `sortByCountReactions` (إضافة العمود والترتيب معاً).

**مثال**:
```php
$products = Product::withCountReactions('DESC', 'reactions_count', 'heart')->get(); // تفاعلات من نوع heart فقط
```

### 5.3 نطاقات تاريخ التفاعل

#### `scopeAddLatestReaction(...)` – يضيف عموداً بأحدث تاريخ تفاعل (اختيارياً تحديد الحقل عبر `$field`).
#### `scopeSortByLatestReaction(...)` – ترتيب حسب أحدث تاريخ تفاعل.
#### `scopeWithLatestReaction(...)` – إضافة العمود والترتيب معاً.

**مثال**:
```php
$products = Product::withLatestReaction('DESC', 'last_reaction', 'heart')->get();
```

#### `scopeAddEarliestReaction(...)` – يضيف عموداً بأقدم تاريخ تفاعل.
#### `scopeSortByEarliestReaction(...)` – ترتيب حسب أقدم تاريخ تفاعل.
#### `scopeWithEarliestReaction(...)` – إضافة العمود والترتيب معاً.

### 5.4 نطاقات التصفية حسب المستخدم

#### `scopeReactedByUser(Builder $query, $user = null, $value = null)`
- **الوصف**: تصفية الكائنات التي تفاعل معها مستخدم معين (إذا لم يُمرر `$user`، يستخدم المستخدم الحالي).
- **مثال**:
  ```php
  // المنتجات التي تفاعل معها المستخدم الحالي (أي نوع)
  $products = Product::reactedByUser()->get();
  
  // المنتجات التي تفاعل معها مستخدم محدد كنوع 'heart'
  $products = Product::reactedByUser($user, 'heart')->get();
  ```

#### `scopeNotReactedByUser(...)` – تصفية الكائنات التي **لم** يتفاعل معها مستخدم معين.

### 5.5 نطاقات التصفية حسب وجود تفاعلات

#### `scopeHasReactions(Builder $query, $minCount = 1, $value = null)`
- **الوصف**: تصفية الكائنات التي لديها على الأقل `$minCount` من التفاعلات (مع خيارات التصفية حسب `value`).
- **مثال**:
  ```php
  // منتجات لها 5 تفاعلات من نوع 'heart' على الأقل
  $products = Product::hasReactions(5, 'heart')->get();
  ```

#### `scopeHasNoReactions(...)` – تصفية الكائنات التي لا تفاعلات لها.

### 5.6 نطاق إضافة عمود حالة التفاعل للمستخدم الحالي

#### `scopeWithIsReactedByUser(Builder $query, $user = null, $columnName = 'is_reacted_by_user', $value = null)`
- **الوصف**: يضيف عموداً منطقياً (`1` أو `0`) إلى نتيجة الاستعلام يبين ما إذا كان المستخدم المحدد قد تفاعل مع الكائن.
- **مثال**:
  ```php
  $products = Product::withIsReactedByUser()->get();
  foreach ($products as $product) {
      if ($product->is_reacted_by_user) {
          echo "لقد تفاعلت مع هذا المنتج";
      }
  }
  ```

### 5.7 نطاقات خاصة

#### `scopeTopReacted(Builder $query, $limit = 10, $orderDirection = 'DESC', $value = null)`
- **الوصف**: يجلب الكائنات الأكثر تفاعلاً (حسب عدد التفاعلات) مع تحديد الحد.
- **مثال**:
  ```php
  $topProducts = Product::topReacted(10, 'DESC', 'heart')->get();
  ```

### 5.8 دوال إحصائية مساعدة

#### `getTotalReactions($value = null)`
- **الوصف**: إجمالي عدد التفاعلات (مجموع `COUNT`) حسب خيارات التصفية.
- **المخرجات**: عدد صحيح.

#### `getReactionsCountByType($value = null)`
- **الوصف**: توزيع التفاعلات حسب نوع المستخدم (`user_type`).
- **المخرجات**: `Collection` حيث المفتاح هو `user_type` والقيمة هي العدد.

#### `getReactorsUsers($value = null)`
- **الوصف**: قائمة المستخدمين الذين تفاعلوا مع الكائن (مع تحميل العلاقة `user`).
- **المخرجات**: `Collection` من كائنات المستخدمين.

---

## 6. أمثلة عملية شاملة

### مثال 1: إضافة تفاعل والتحقق

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Reaction;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة تفاعل من نوع 'heart'
$reaction = Reaction::add($product, $user, 'heart');

// إضافة تفاعل من نوع 'wow'
$reaction = Reaction::add($product, $user, 'wow', ['notes' => 'مذهل!']);

// التحقق مما إذا كان المستخدم قد تفاعل بالمنتج (أي نوع)
if ($product->isReacted()) {
    echo "لقد تفاعلت مع هذا المنتج";
}

// التحقق مما إذا كان التفاعل من نوع 'heart'
if ($product->isReacted(null, 'heart')) {
    echo "لقد أضفت قلباً لهذا المنتج";
}
```

### مثال 2: إزالة تفاعل

```php
// إزالة التفاعل (أي نوع)
Reaction::remove($product, $user);

// إزالة تفاعل من نوع 'heart' فقط
Reaction::remove($product, $user, 'heart');
```

### مثال 3: ترتيب المنتجات حسب عدد التفاعلات

```php
$topProducts = Product::topReacted(10)->get();
```

### مثال 4: إضافة عمود `is_reacted_by_user` في قائمة المنتجات

```php
$products = Product::withIsReactedByUser()->paginate(20);
foreach ($products as $product) {
    echo $product->name . ' - ' . ($product->is_reacted_by_user ? 'تفاعلت' : 'لم تتفاعل');
}
```

### مثال 5: تصفية المنتجات التي تفاعل معها المستخدم الحالي

```php
$myReactedProducts = Product::reactedByUser()->get();
```

### مثال 6: الإحصائيات

```php
$product = Product::find(1);
echo "إجمالي التفاعلات من نوع 'heart': " . $product->getTotalReactions('heart');
print_r($product->getReactionsCountByType('heart')->toArray());
$users = $product->getReactorsUsers('heart');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

### مثال 7: الجمع بين النطاقات المتقدمة

```php
// المنتجات التي لها أكثر من 5 تفاعلات من نوع 'heart'، مرتبة حسب أحدث تفاعل
$products = Product::hasReactions(5, 'heart')
    ->sortByLatestReaction('DESC', 'latest_heart_at')
    ->get();
```

---

## 7. دوال التوافق العكسي

للحفاظ على الكود القديم الذي يستخدم أسماء النطاقات السابقة، تم توفير دوال تغليف في الكلاس الرئيسي. هذه الدوال موصوفة بـ `@deprecated` وينصح باستخدام البدائل الجديدة.

**قائمة الدوال القديمة المدعومة:**

| الدالة القديمة | الدالة الجديدة |
|----------------|----------------|
| `scopeSortByCountReactionsOld` | `scopeSortByCountReactions` |
| `scopeWithSortByCountReactions` | `scopeWithCountReactions` |
| `scopeSortByCreatedAtReactions` | `scopeSortByLatestReaction` |
| `scopeAddSortByCreatedAtReactions` | `scopeAddLatestReaction` |
| `scopeWithSortByCreatedAtReactions` | `scopeWithLatestReaction` |
| `scopeWhereHasReaction` | `scopeReactedByUser` (مع تحسينات) |
| `scopeWhereHasReactions` | `scopeReactedByUser` |
| `scopeWhereHasUserReaction` | `scopeReactedByUser` |
| `scopeWhereHasUserReactions` | `scopeReactedByUser` |

**ملاحظة حول الدوال القديمة `whereHasReaction` ونظيراتها:** تم تحسينها لدعم معامل `$isForceUser` للتحكم في سلوك الاستعلام عند عدم وجود مستخدم، مع قراءة القيمة الافتراضية من الإعدادات (`nano.markable::reactionable.scope.where_has_reaction.is_force_user`). السلوك الافتراضي (`true`) يضيف شرطاً مستحيلاً لعدم إرجاع أي نتائج (يتوافق مع السلوك الأصلي).

---

## 8. تحسين الأداء والتخزين المؤقت

السلوك يعتمد على استعلامات فرعية (subqueries) محسنة لتجنب تحميل العلاقات بالكامل. لا يوجد نظام تخزين مؤقت مدمج بشكل افتراضي، لكن يمكن الاعتماد على آلية التخزين المؤقت للموديلات نفسها أو استخدام ذاكرة التخزين المؤقت للتطبيق إذا احتجت لتكرار استعلامات ثقيلة.

- عند استخدام `withCountReactions` أو `sortByCountReactions`، يتم تنفيذ استعلام فرعي واحد فقط.
- استخدام `withIsReactedByUser` يضيف استعلاماً فرعياً لكل صف، لذا يُنصح باستخدامه بحذر في القوائم الكبيرة، لكنه عادةً ما يكون مقبولاً بسبب فاعلية استعلامات SQL.

**ملاحظة:** إذا أضفت `SoftDeletes` إلى موديل `Reaction` في المستقبل، يمكن تعديل النطاقات لدعم `withTrashed` بسهولة.

---

## 9. الخاتمة

`ReactionableModel` هو حل متكامل وقوي لإدارة نظام التفاعلات في تطبيقات نانوسوفت. بفضل تنوع النطاقات والدوال المساعدة، يمكن للمطورين بناء أنظمة تفاعل متطورة بسهولة مع الاحتفاظ بمرونة عالية وقابلية للتوسع. التوافق العكسي يضمن انتقالاً سلساً من الإصدارات السابقة. نأمل أن يكون هذا التوثيق مرجعاً شاملاً لجميع إمكانيات السلوك.