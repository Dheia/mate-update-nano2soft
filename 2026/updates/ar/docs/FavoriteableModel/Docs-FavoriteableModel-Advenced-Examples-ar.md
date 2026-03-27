# أمثلة عملية متقدمة لاستخدام سلوك `FavoriteableModel`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `FavoriteableModel` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 إضافة عنصر إلى المفضلة مع أنواع مختلفة

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Favorite;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة المنتج إلى المفضلة (النوع الافتراضي 'favorite')
$favorite = Favorite::add($product, $user);

// إضافة المنتج كمفضلة من نوع 'like'
$like = Favorite::add($product, $user, 'like');

// إضافة المنتج كمفضلة من نوع 'watch_later'
$watchLater = Favorite::add($product, $user, 'watch_later', ['notes' => 'مشاهدة لاحقاً']);

// إضافة منتج مع بيانات ميتاداتا إضافية
$favorite = Favorite::add($product, $user, 'favorite', [
    'notes' => 'منتج رائع',
    'priority' => 'high'
]);
```

#### 1.2 إزالة عنصر من المفضلة

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إزالة المنتج من المفضلة (النوع الافتراضي)
Favorite::remove($product, $user);

// إزالة المنتج من مفضلات نوع 'like' فقط
Favorite::remove($product, $user, 'like');
```

#### 1.3 التبديل بين الإضافة والإزالة (Toggle)

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إذا كان المنتج في المفضلة، يزيله؛ وإلا يضيفه
$result = Favorite::toggle($product, $user, 'favorite', ['notes' => 'تم التبديل']);
```

#### 1.4 الحصول على سجل المفضلة للمستخدم الحالي

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$favorite = $product->getFavorited($user);
if ($favorite) {
    echo "نوع المفضلة: " . $favorite->value;
    echo "ملاحظات: " . $favorite->metadata['notes'] ?? '';
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب جميع المنتجات التي أضافها المستخدم الحالي إلى المفضلة

```php
$myFavorites = Product::favoritedByUser()->get();

// مع تضمين نوع معين فقط
$myLikes = Product::favoritedByUser(null, 'like')->get();

// مع تضمين نوعين مختلفين (باستخدام orWhereHas)
$myFavorites = Product::whereHas('favorites', function($q) use ($user) {
    $q->where('user_id', $user->id)
      ->whereIn('value', ['favorite', 'like']);
})->get();
```

#### 2.2 جلب المنتجات التي لم يضفها المستخدم الحالي إلى المفضلة (لاقتراحها)

```php
$suggestedProducts = Product::notFavoritedByUser()->paginate(20);
```

#### 2.3 جلب المنتجات التي أضافها مستخدم معين إلى المفضلة مع تحميل تفاصيل المفضلة

```php
$user = User::find(2);
$products = Product::favoritedByUser($user)
    ->with(['favorites' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 استخدام الدوال القديمة `whereHasFavorite` مع التحكم في السلوك عند عدم وجود مستخدم

```php
// السلوك الافتراضي (لا يعيد نتائج عند عدم وجود مستخدم)
$products = Product::whereHasFavorite(null, 'like')->get();

// تعطيل القسر (يعيد جميع المنتجات عند عدم وجود مستخدم)
$products = Product::whereHasFavorite(null, 'like', false)->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد المفضلات (الأكثر تفضيلاً)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountFavorites('DESC', 'favorites_count')->take(10)->get();

// باستخدام الاختصار topFavorited
$topProducts = Product::topFavorited(10)->get();

// ترتيب حسب عدد المفضلات من نوع 'like' فقط
$topLiked = Product::sortByCountFavorites('DESC', 'likes_count', 'like')->get();
```

#### 3.2 إضافة عمود عدد المفضلات إلى النتائج (بدون ترتيب)

```php
$products = Product::addCountFavorites()->get();
foreach ($products as $product) {
    echo $product->name . ' - عدد المفضلات: ' . $product->favorites_count;
}
```

#### 3.3 ترتيب حسب أحدث تفضيل (آخر مستخدم أضاف إلى المفضلة)

```php
// ترتيب حسب تاريخ إنشاء آخر تفضيل (الأحدث أولاً)
$products = Product::sortByLatestFavorite('DESC')->get();

// استخدام حقل updated_at بدلاً من created_at
$products = Product::sortByLatestFavorite('DESC', 'latest_favorite_at', null, null, null, 'updated_at')->get();
```

#### 3.4 إضافة عمود آخر تاريخ تفضيل مع الترتيب

```php
$products = Product::withLatestFavorite('DESC', 'last_favorite_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - آخر تفضيل: ' . $product->last_favorite_date;
}
```

#### 3.5 ترتيب حسب أقدم تفضيل (أول من أضاف إلى المفضلة)

```php
$products = Product::sortByEarliestFavorite('ASC')->get();
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي عدد المفضلات لمنتج معين

```php
$product = Product::find(1);

// كل المفضلات
$total = $product->getTotalFavorites();

// المفضلات من نوع 'like' فقط
$totalLikes = $product->getTotalFavorites('like');
```

#### 4.2 توزيع المفضلات حسب نوع المستخدم

```php
$counts = $product->getFavoritesCountByType();
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// توزيع المفضلات من نوع 'like' فقط
$likesByType = $product->getFavoritesCountByType('like');
```

#### 4.3 جلب قائمة بأسماء المستخدمين الذين أضافوا منتجاً معيناً إلى المفضلة

```php
$users = $product->getFavoritersUsers();
foreach ($users as $user) {
    echo $user->name . "\n";
}

// فقط المستخدمين الذين أضافوه كنوع 'like'
$likers = $product->getFavoritersUsers('like');
```

#### 4.4 إحصائيات المفضلات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي أضيفت إلى المفضلة خلال الأسبوع الماضي
$products = Product::whereHas('favorites', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأكثر تفضيلاً والتي أضافها المستخدم الحالي أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topFavorited(20)
    ->favoritedByUser($user)
    ->withIsFavoritedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (عدد المفضلات: ' . $product->favorites_count . ') - ';
    echo $product->is_favorited_by_user ? 'أنت تفضله' : 'لا تفضله';
}
```

#### 5.2 المنتجات التي لها أكثر من 5 مفضلات من نوع 'like'، مرتبة حسب آخر تفضيل

```php
$products = Product::hasFavorites(5, 'like')
    ->withLatestFavorite('DESC', 'last_like_date')
    ->get();
```

#### 5.3 المنتجات النشطة التي أضافها المستخدم الحالي إلى المفضلة، مع عدد المفضلات

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->favoritedByUser()
    ->withCountFavorites('DESC', 'favorites_count')
    ->get();
```

#### 5.4 المنتجات التي أضافها المستخدم الحالي إلى المفضلة مع إضافة عمود الحالة (سيكون 1 دائماً)

```php
$products = Product::favoritedByUser()
    ->withIsFavoritedByUser()
    ->get();
// كل المنتجات ستحوي is_favorited_by_user = 1
```

#### 5.5 المنتجات التي لها مفضلات من نوع 'favorite' فقط، واستبعاد المنتجات التي لا مفضلات لها

```php
$products = Product::hasFavorites(1, 'favorite')
    ->sortByCountFavorites('DESC', 'favorites_count', 'favorite')
    ->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض زر الإضافة/الإزالة من المفضلة

```php
@php
$product = Product::find(1);
$isFavorited = $product->isFavorited();
$favoriteCount = $product->getTotalFavorites();
@endphp

<button class="favorite-btn" data-id="{{ $product->id }}" data-value="favorite">
    @if($isFavorited)
        إزالة من المفضلة
    @else
        إضافة إلى المفضلة
    @endif
</button>
<span class="favorites-count">{{ $favoriteCount }} مفضلة</span>
```

#### 6.2 عرض قائمة المفضلين في صفحة المنتج

```php
$favoriters = $product->getFavoritersUsers();
?>

<div class="favoriters-list">
    <h4>أبرز المفضلين</h4>
    @foreach($favoriters->take(5) as $user)
        <div class="favoriter">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

#### 6.3 عرض أزرار لأنواع متعددة من المفضلات

```php
@php
$product = Product::find(1);
$isFavorite = $product->isFavorited(null, 'favorite');
$isLike = $product->isFavorited(null, 'like');
@endphp

<button class="favorite-btn" data-type="favorite">
    {{ $isFavorite ? '❤️' : '🤍' }} مفضلة
</button>
<button class="like-btn" data-type="like">
    {{ $isLike ? '👍' : '👎' }} إعجاب
</button>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع معلومات المفضلة

```php
public function show($id)
{
    $product = Product::withIsFavoritedByUser()
        ->withCountFavorites('DESC', 'favorites_count')
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "favorites_count": 245,
    "is_favorited_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأكثر تفضيلاً مع إضافة عمود المفضلة للمستخدم

```php
public function topFavorited()
{
    $products = Product::topFavorited(10)
        ->withIsFavoritedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لإضافة أو إزالة عنصر من المفضلة

```php
public function toggleFavorite($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'favorite');
    
    $result = Favorite::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_favorited' => $product->isFavorited($user, $value),
        'favorites_count' => $product->getTotalFavorites($value)
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::sortByCountFavoritesOld('DESC')->get();

// جديد
$products = Product::sortByCountFavorites('DESC')->get();
```

```php
// قديم
$products = Product::withSortByCountFavorites('DESC')->get();

// جديد
$products = Product::withCountFavorites('DESC')->get();
```

```php
// قديم
$products = Product::sortByCreatedAtFavorites('DESC')->get();

// جديد
$products = Product::sortByLatestFavorite('DESC')->get();
```

```php
// قديم
$products = Product::whereHasFavorite($user, 'like')->get();

// جديد
$products = Product::favoritedByUser($user, 'like')->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام أنواع متعددة من المفضلات في نفس النموذج

```php
// تعيين الأنواع المسموحة في الإعدادات
// config/nano/markable.php
return [
    'allowed_values' => [
        'favorite' => ['favorite', 'like', 'watch_later', 'saved'],
    ],
];

// الاستعلام عن منتجات تم تفضيلها كنوع 'watch_later'
$watchLater = Product::favoritedByUser(null, 'watch_later')->get();
```

#### 9.2 الحصول على عدد المفضلات لكل نوع

```php
$product = Product::find(1);

$favoritesByType = [
    'favorite' => $product->getTotalFavorites('favorite'),
    'like' => $product->getTotalFavorites('like'),
    'watch_later' => $product->getTotalFavorites('watch_later'),
];
```

#### 9.3 استخدام النطاق `hasFavorites` مع حد أدنى متغير من التحكم

```php
$minCount = request('min_favorites', 5);
$products = Product::hasFavorites($minCount, 'like')->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountFavorites('DESC', 'my_favorites_column', 'like')->get();
echo $products->first()->my_favorites_column;
```

#### 9.5 الجمع بين `favoritedByUser` و `notFavoritedByUser` في استعلام واحد (باستخدام union)

```php
$favorited = Product::favoritedByUser()->get();
$notFavorited = Product::notFavoritedByUser()->get();
$all = $favorited->merge($notFavorited);
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح منتجات للمستخدم بناءً على مفضلات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي (افترض وجود نظام متابعة)
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي أضافها هؤلاء المستخدمون إلى المفضلة
$suggestedProducts = Product::whereHas('favorites', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers);
})->whereNotIn('id', $user->favoritedBy()->pluck('id')) // استبعاد ما يفضله المستخدم
  ->withCountFavorites('DESC', 'favorites_count')
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات المفضلات الجديدة (عند إضافة منتج إلى المفضلة)

```php
// بعد إضافة المنتج إلى المفضلة
$favorite = Favorite::add($product, $user, 'favorite');

// إرسال إشعار لمالك المنتج
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewFavoriteNotification($product, $user));
}
```

#### 10.3 عرض عدد المفضلات في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withCountFavorites('DESC', 'favorites_count')
    ->paginate(20);
```

#### 10.4 تحليل سلوك المستخدمين (الأكثر تفاعلاً عبر المفضلات)

```php
// المستخدمون الذين أضافوا أكثر من 10 منتجات إلى المفضلة
$activeUsers = User::has('favorites', '>=', 10)->get();

// المنتجات الأكثر تفضيلاً من قبل المستخدمين النشطين
$topProducts = Product::whereHas('favorites', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'));
})->withCountFavorites('DESC', 'favorites_count')
  ->limit(10)
  ->get();
```

#### 10.5 إنشاء قوائم مفضلات مخصصة (مثل قائمة "أقرأ لاحقاً")

```php
// إضافة مقال إلى قائمة "أقرأ لاحقاً"
$article = Article::find(1);
Favorite::add($article, $user, 'read_later', ['notes' => 'مقال مهم']);

// جلب قائمة "أقرأ لاحقاً"
$readLater = Article::favoritedByUser(null, 'read_later')->get();
```

---

### الخلاصة

يوفر سلوك `FavoriteableModel` مجموعة غنية من الأدوات التي تمكنك من بناء نظام مفضلات متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل سلوك المستخدمين، وتحسين أداء التطبيقات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.