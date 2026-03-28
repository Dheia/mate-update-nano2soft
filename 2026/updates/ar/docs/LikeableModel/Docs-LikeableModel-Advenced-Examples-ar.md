# أمثلة عملية متقدمة لاستخدام سلوك `LikeableModel`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `LikeableModel` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 إضافة إعجاب وعدم إعجاب

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Like;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة إعجاب (النوع الافتراضي 'like')
$like = Like::add($product, $user);

// إضافة عدم إعجاب (نوع 'dislike')
$dislike = Like::add($product, $user, 'dislike');

// إضافة إعجاب مع بيانات ميتاداتا
$like = Like::add($product, $user, 'like', ['reason' => 'جودة عالية']);
```

#### 1.2 إزالة إعجاب أو عدم إعجاب

```php
// إزالة الإعجاب (النوع الافتراضي)
Like::remove($product, $user);

// إزالة عدم الإعجاب فقط
Like::remove($product, $user, 'dislike');
```

#### 1.3 التبديل بين الإعجاب وعدم الإعجاب (Toggle)

```php
// إذا كان المستخدم معجباً، يزيل الإعجاب؛ وإلا يضيف إعجاباً
$result = Like::toggle($product, $user, 'like');

// تبديل حالة عدم الإعجاب
$result = Like::toggle($product, $user, 'dislike');
```

#### 1.4 الحصول على سجل الإعجاب للمستخدم الحالي

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$like = $product->getLiked($user);
if ($like) {
    echo "نوع الإعجاب: " . $like->value; // like أو dislike
    echo "بيانات إضافية: " . $like->metadata['reason'] ?? '';
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب جميع المنتجات التي أعجب بها المستخدم الحالي

```php
$myLikedProducts = Product::likedByUser()->get();

// فقط الإعجابات من نوع 'like'
$myLikes = Product::likedByUser(null, 'like')->get();

// فقط عدم الإعجاب (dislike)
$myDislikes = Product::likedByUser(null, 'dislike')->get();
```

#### 2.2 جلب المنتجات التي لم يعجب بها المستخدم الحالي (لاقتراحها)

```php
$suggestedProducts = Product::notLikedByUser()->paginate(20);
```

#### 2.3 جلب المنتجات التي أعجب بها مستخدم معين مع تحميل تفاصيل الإعجاب

```php
$user = User::find(2);
$products = Product::likedByUser($user)
    ->with(['likes' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 استخدام الدوال القديمة `whereHasLike` مع التحكم في السلوك عند عدم وجود مستخدم

```php
// السلوك الافتراضي (لا يعيد نتائج عند عدم وجود مستخدم)
$products = Product::whereHasLike(null, 'like')->get();

// تعطيل القسر (يعيد جميع المنتجات عند عدم وجود مستخدم)
$products = Product::whereHasLike(null, 'like', false)->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد الإعجابات (الأكثر إعجاباً)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountLikes('DESC', 'likes_count')->take(10)->get();

// باستخدام الاختصار topLiked
$topProducts = Product::topLiked(10)->get();

// ترتيب حسب عدد الإعجابات من نوع 'like' فقط
$topLiked = Product::sortByCountLikes('DESC', 'likes_count', 'like')->get();

// ترتيب حسب عدد عدم الإعجاب (dislike)
$topDisliked = Product::sortByCountLikes('DESC', 'dislikes_count', 'dislike')->get();
```

#### 3.2 إضافة عمود عدد الإعجابات إلى النتائج (بدون ترتيب)

```php
$products = Product::addCountLikes()->get();
foreach ($products as $product) {
    echo $product->name . ' - إعجابات: ' . $product->likes_count;
}
```

#### 3.3 ترتيب حسب أحدث إعجاب (آخر مستخدم أعجب بالمنتج)

```php
// ترتيب حسب تاريخ إنشاء آخر إعجاب (الأحدث أولاً)
$products = Product::sortByLatestLike('DESC')->get();

// استخدام حقل updated_at بدلاً من created_at
$products = Product::sortByLatestLike('DESC', 'latest_like_at', null, 'updated_at')->get();
```

#### 3.4 إضافة عمود آخر تاريخ إعجاب مع الترتيب

```php
$products = Product::withLatestLike('DESC', 'last_like_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - آخر إعجاب: ' . $product->last_like_date;
}
```

#### 3.5 ترتيب حسب أقدم إعجاب (أول من أعجب)

```php
$products = Product::sortByEarliestLike('ASC')->get();
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي عدد الإعجابات لمنتج معين

```php
$product = Product::find(1);

// كل الإعجابات (بما فيها like و dislike)
$total = $product->getTotalLikes();

// الإعجابات من نوع 'like' فقط
$totalLikes = $product->getTotalLikes('like');

// عدم الإعجاب
$totalDislikes = $product->getTotalLikes('dislike');
```

#### 4.2 توزيع الإعجابات حسب نوع المستخدم

```php
$counts = $product->getLikesCountByType('like');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// توزيع عدم الإعجاب
$dislikeCounts = $product->getLikesCountByType('dislike');
```

#### 4.3 جلب قائمة بأسماء المستخدمين الذين أعجبوا بمنتج معين

```php
$users = $product->getLikersUsers('like');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 إحصائيات الإعجابات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي أعجب بها المستخدمون خلال الأسبوع الماضي
$products = Product::whereHas('likes', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأكثر إعجاباً والتي أعجب بها المستخدم الحالي أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topLiked(20)
    ->likedByUser($user)
    ->withIsLikedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (عدد الإعجابات: ' . $product->likes_count . ') - ';
    echo $product->is_liked_by_user ? 'أعجبك' : 'لم يعجبك';
}
```

#### 5.2 المنتجات التي لها أكثر من 5 إعجابات من نوع 'like'، مرتبة حسب أحدث إعجاب

```php
$products = Product::hasLikes(5, 'like')
    ->withLatestLike('DESC', 'last_like_date')
    ->get();
```

#### 5.3 المنتجات النشطة التي أعجب بها المستخدم الحالي، مع عدد الإعجابات

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->likedByUser()
    ->withCountLikes('DESC', 'likes_count')
    ->get();
```

#### 5.4 المنتجات التي أعجب بها المستخدم الحالي مع إضافة عمود الحالة (سيكون 1 دائماً)

```php
$products = Product::likedByUser()
    ->withIsLikedByUser()
    ->get();
// كل المنتجات ستحوي is_liked_by_user = 1
```

#### 5.5 المنتجات التي لها إعجابات من نوع 'like' فقط، واستبعاد المنتجات التي لا إعجابات لها

```php
$products = Product::hasLikes(1, 'like')
    ->sortByCountLikes('DESC', 'likes_count', 'like')
    ->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض زر الإعجاب/عدم الإعجاب

```php
@php
$product = Product::find(1);
$isLiked = $product->isLiked(null, 'like');
$isDisliked = $product->isLiked(null, 'dislike');
$likeCount = $product->getTotalLikes('like');
$dislikeCount = $product->getTotalLikes('dislike');
@endphp

<button class="like-btn" data-id="{{ $product->id }}" data-value="like">
    @if($isLiked)
        ❤️ إلغاء الإعجاب
    @else
        🤍 إعجاب
    @endif
</button>
<button class="dislike-btn" data-id="{{ $product->id }}" data-value="dislike">
    @if($isDisliked)
        👎 إلغاء عدم الإعجاب
    @else
        👍 عدم الإعجاب
    @endif
</button>
<span class="likes-count">{{ $likeCount }} إعجاب</span>
<span class="dislikes-count">{{ $dislikeCount }} عدم إعجاب</span>
```

#### 6.2 عرض قائمة المعجبين في صفحة المنتج

```php
$likers = $product->getLikersUsers('like');
?>

<div class="likers-list">
    <h4>أبرز المعجبين</h4>
    @foreach($likers->take(5) as $user)
        <div class="liker">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع معلومات الإعجابات

```php
public function show($id)
{
    $product = Product::withIsLikedByUser()
        ->withCountLikes('DESC', 'likes_count')
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "likes_count": 245,
    "is_liked_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأكثر إعجاباً مع إضافة عمود الإعجاب للمستخدم

```php
public function topLiked()
{
    $products = Product::topLiked(10)
        ->withIsLikedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لإضافة أو إزالة إعجاب

```php
public function toggleLike($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'like');
    
    $result = Like::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_liked' => $product->isLiked($user, $value),
        'likes_count' => $product->getTotalLikes($value)
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::sortByCountLikesOld('DESC')->get();

// جديد
$products = Product::sortByCountLikes('DESC')->get();
```

```php
// قديم
$products = Product::withSortByCountLikes('DESC', 'like')->get();

// جديد
$products = Product::withCountLikes('DESC', 'likes_count', 'like')->get();
```

```php
// قديم
$products = Product::sortByCreatedAtLikes('DESC', 'like')->get();

// جديد
$products = Product::sortByLatestLike('DESC', 'latest_like_at', 'like')->get();
```

```php
// قديم
$products = Product::whereHasLike($user, 'like')->get();

// جديد
$products = Product::likedByUser($user, 'like')->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام أنواع متعددة من الإعجابات

```php
// تعيين الأنواع المسموحة في الإعدادات
// config/nano/markable.php
return [
    'allowed_values' => [
        'like' => ['like', 'dislike', 'love', 'angry'],
    ],
];

// الاستعلام عن منتجات تم الإعجاب بها كنوع 'love'
$loveProducts = Product::likedByUser(null, 'love')->get();
```

#### 9.2 الحصول على عدد الإعجابات لكل نوع

```php
$product = Product::find(1);

$likesByType = [
    'like' => $product->getTotalLikes('like'),
    'dislike' => $product->getTotalLikes('dislike'),
    'love' => $product->getTotalLikes('love'),
];
```

#### 9.3 استخدام النطاق `hasLikes` مع حد أدنى متغير من التحكم

```php
$minCount = request('min_likes', 5);
$products = Product::hasLikes($minCount, 'like')->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountLikes('DESC', 'my_likes_column', 'like')->get();
echo $products->first()->my_likes_column;
```

#### 9.5 الجمع بين `likedByUser` و `notLikedByUser` في استعلام واحد (باستخدام union)

```php
$liked = Product::likedByUser()->get();
$notLiked = Product::notLikedByUser()->get();
$all = $liked->merge($notLiked);
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح منتجات للمستخدم بناءً على إعجابات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي (افترض وجود نظام متابعة)
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي أعجب بها هؤلاء المستخدمون (نوع like)
$suggestedProducts = Product::whereHas('likes', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('value', 'like');
})->whereNotIn('id', $user->likedBy()->pluck('id')) // استبعاد ما أعجب به المستخدم
  ->withCountLikes('DESC', 'likes_count')
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات الإعجابات الجديدة (عند إعجاب مستخدم بمنتج)

```php
// بعد إضافة الإعجاب
$like = Like::add($product, $user, 'like');

// إرسال إشعار لمالك المنتج
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewLikeNotification($product, $user));
}
```

#### 10.3 عرض عدد الإعجابات في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withCountLikes('DESC', 'likes_count')
    ->paginate(20);
```

#### 10.4 تحليل سلوك المستخدمين (الأكثر تفاعلاً عبر الإعجابات)

```php
// المستخدمون الذين أعجبوا بأكثر من 10 منتجات
$activeUsers = User::has('likes', '>=', 10)->get();

// المنتجات الأكثر إعجاباً من قبل المستخدمين النشطين
$topProducts = Product::whereHas('likes', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'))
      ->where('value', 'like');
})->withCountLikes('DESC', 'likes_count')
  ->limit(10)
  ->get();
```

#### 10.5 إنشاء نظام تقييم مركب (إعجاب/عدم إعجاب)

```php
// حساب نسبة الإعجاب لمنتج
$product = Product::find(1);
$likes = $product->getTotalLikes('like');
$dislikes = $product->getTotalLikes('dislike');
$total = $likes + $dislikes;
$ratio = $total > 0 ? round(($likes / $total) * 100, 1) : 0;

echo "نسبة الإعجاب: {$ratio}%";
```

---

### الخلاصة

يوفر سلوك `LikeableModel` مجموعة غنية من الأدوات التي تمكنك من بناء نظام إعجابات متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل سلوك المستخدمين، وتحسين أداء التطبيقات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.