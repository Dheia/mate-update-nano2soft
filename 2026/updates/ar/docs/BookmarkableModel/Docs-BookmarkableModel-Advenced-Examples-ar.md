# أمثلة عملية متقدمة لاستخدام سلوك `BookmarkableModel`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `BookmarkableModel` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 إضافة إشارة مرجعية بأنواع مختلفة

```php
use Nano\AuthApi\Classes\AuthHelpers;
use Nano\Markable\Models\Bookmark;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة إشارة مرجعية من نوع 'favorite'
$bookmark = Bookmark::add($product, $user, 'favorite');

// إضافة إشارة مرجعية من نوع 'read_later'
$bookmark = Bookmark::add($product, $user, 'read_later', ['notes' => 'مقال مهم']);

// إضافة إشارة مرجعية من نوع 'wishlist'
$bookmark = Bookmark::add($product, $user, 'wishlist');

// إضافة إشارة مع ميتاداتا إضافية
$bookmark = Bookmark::add($product, $user, 'favorite', [
    'priority' => 'high',
    'tags' => ['electronics', 'sale']
]);
```

#### 1.2 إزالة إشارة مرجعية

```php
// إزالة الإشارة (أي نوع)
Bookmark::remove($product, $user);

// إزالة إشارة من نوع 'read_later' فقط
Bookmark::remove($product, $user, 'read_later');
```

#### 1.3 التبديل بين الإضافة والإزالة (Toggle)

```php
// إذا كان المنتج في الإشارات، يزيله؛ وإلا يضيفه
$result = Bookmark::toggle($product, $user, 'favorite');

// تبديل إشارة من نوع 'wishlist'
$result = Bookmark::toggle($product, $user, 'wishlist');
```

#### 1.4 الحصول على سجل الإشارة للمستخدم الحالي

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

$bookmark = $product->getBookmarked($user);
if ($bookmark) {
    echo "نوع الإشارة: " . $bookmark->value; // favorite, read_later, wishlist
    echo "ملاحظات: " . $bookmark->metadata['notes'] ?? '';
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب جميع المنتجات التي أضافها المستخدم الحالي إلى الإشارات

```php
// جميع الإشارات (أي نوع)
$myBookmarks = Product::bookmarkedByUser()->get();

// فقط الإشارات من نوع 'favorite'
$myFavorites = Product::bookmarkedByUser(null, 'favorite')->get();

// فقط الإشارات من نوع 'read_later'
$myReadLater = Product::bookmarkedByUser(null, 'read_later')->get();
```

#### 2.2 جلب المنتجات التي لم يضفها المستخدم الحالي إلى الإشارات (لاقتراحها)

```php
$suggestedProducts = Product::notBookmarkedByUser()->paginate(20);
```

#### 2.3 جلب المنتجات التي أضافها مستخدم معين إلى الإشارات مع تحميل تفاصيل الإشارة

```php
$user = User::find(2);
$products = Product::bookmarkedByUser($user)
    ->with(['bookmarks' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

#### 2.4 استخدام الدوال القديمة `whereHasBookmark` مع التحكم في السلوك عند عدم وجود مستخدم

```php
// السلوك الافتراضي (لا يعيد نتائج عند عدم وجود مستخدم)
$products = Product::whereHasBookmark(null, 'favorite')->get();

// تعطيل القسر (يعيد جميع المنتجات عند عدم وجود مستخدم)
$products = Product::whereHasBookmark(null, 'favorite', false)->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد الإشارات (الأكثر إشارة)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountBookmarks('DESC', 'bookmarks_count')->take(10)->get();

// باستخدام الاختصار topBookmarked
$topProducts = Product::topBookmarked(10)->get();

// ترتيب حسب عدد الإشارات من نوع 'favorite' فقط
$topFavorites = Product::sortByCountBookmarks('DESC', 'favorites_count', 'favorite')->get();

// ترتيب حسب عدد الإشارات من نوع 'read_later' فقط
$topReadLater = Product::sortByCountBookmarks('DESC', 'read_later_count', 'read_later')->get();
```

#### 3.2 إضافة عمود عدد الإشارات إلى النتائج (بدون ترتيب)

```php
$products = Product::addCountBookmarks()->get();
foreach ($products as $product) {
    echo $product->name . ' - عدد الإشارات: ' . $product->bookmarks_count;
}
```

#### 3.3 ترتيب حسب أحدث إشارة (آخر مستخدم أضاف المنتج إلى الإشارات)

```php
// ترتيب حسب تاريخ إنشاء آخر إشارة (الأحدث أولاً)
$products = Product::sortByLatestBookmark('DESC')->get();

// استخدام حقل updated_at بدلاً من created_at
$products = Product::sortByLatestBookmark('DESC', 'latest_bookmark_at', null, 'updated_at')->get();
```

#### 3.4 إضافة عمود آخر تاريخ إشارة مع الترتيب

```php
$products = Product::withLatestBookmark('DESC', 'last_bookmark_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - آخر إشارة: ' . $product->last_bookmark_date;
}
```

#### 3.5 ترتيب حسب أقدم إشارة (أول من أضاف)

```php
$products = Product::sortByEarliestBookmark('ASC')->get();
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي عدد الإشارات لمنتج معين

```php
$product = Product::find(1);

// كل الإشارات (جميع الأنواع)
$total = $product->getTotalBookmarks();

// الإشارات من نوع 'favorite' فقط
$totalFavorites = $product->getTotalBookmarks('favorite');

// الإشارات من نوع 'read_later' فقط
$totalReadLater = $product->getTotalBookmarks('read_later');
```

#### 4.2 توزيع الإشارات حسب نوع المستخدم

```php
$counts = $product->getBookmarksCountByType('favorite');
// ['RainLab\User\Models\User' => 42, 'Backend\Models\User' => 5]

// توزيع إشارات 'read_later'
$readLaterCounts = $product->getBookmarksCountByType('read_later');
```

#### 4.3 جلب قائمة بأسماء المستخدمين الذين أضافوا منتجاً معيناً إلى الإشارات

```php
$users = $product->getBookmarkersUsers('favorite');
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 إحصائيات الإشارات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي أضيفت إلى الإشارات خلال الأسبوع الماضي
$products = Product::whereHas('bookmarks', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأكثر إشارة والتي أضافها المستخدم الحالي أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topBookmarked(20)
    ->bookmarkedByUser($user)
    ->withIsBookmarkedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (عدد الإشارات: ' . $product->bookmarks_count . ') - ';
    echo $product->is_bookmarked_by_user ? 'أضفتها' : 'لم تضفها';
}
```

#### 5.2 المنتجات التي لها أكثر من 5 إشارات من نوع 'favorite'، مرتبة حسب أحدث إشارة

```php
$products = Product::hasBookmarks(5, 'favorite')
    ->withLatestBookmark('DESC', 'last_favorite_date')
    ->get();
```

#### 5.3 المنتجات النشطة التي أضافها المستخدم الحالي إلى الإشارات، مع عدد الإشارات

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->bookmarkedByUser()
    ->withCountBookmarks('DESC', 'bookmarks_count')
    ->get();
```

#### 5.4 المنتجات التي أضافها المستخدم الحالي إلى الإشارات مع إضافة عمود الحالة (سيكون 1 دائماً)

```php
$products = Product::bookmarkedByUser()
    ->withIsBookmarkedByUser()
    ->get();
// كل المنتجات ستحوي is_bookmarked_by_user = 1
```

#### 5.5 المنتجات التي لها إشارات من نوع 'read_later' فقط، واستبعاد المنتجات التي لا إشارات لها

```php
$products = Product::hasBookmarks(1, 'read_later')
    ->sortByCountBookmarks('DESC', 'read_later_count', 'read_later')
    ->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض أزرار إشارات متعددة

```php
@php
$product = Product::find(1);
$isFavorite = $product->isBookmarked(null, 'favorite');
$isReadLater = $product->isBookmarked(null, 'read_later');
$favoriteCount = $product->getTotalBookmarks('favorite');
$readLaterCount = $product->getTotalBookmarks('read_later');
@endphp

<button class="bookmark-btn" data-type="favorite" data-id="{{ $product->id }}">
    @if($isFavorite)
        ⭐ إزالة من المفضلة
    @else
        ☆ إضافة إلى المفضلة
    @endif
</button>
<button class="bookmark-btn" data-type="read_later" data-id="{{ $product->id }}">
    @if($isReadLater)
        📖 إزالة من القراءة لاحقاً
    @else
        📚 قراءة لاحقاً
    @endif
</button>
<span class="favorite-count">{{ $favoriteCount }}</span>
<span class="read-later-count">{{ $readLaterCount }}</span>
```

#### 6.2 عرض قائمة المفضلين في صفحة المنتج

```php
$bookmarkers = $product->getBookmarkersUsers('favorite');
?>

<div class="bookmarkers-list">
    <h4>أبرز المفضلين</h4>
    @foreach($bookmarkers->take(5) as $user)
        <div class="bookmarker">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع معلومات الإشارات

```php
public function show($id)
{
    $product = Product::withIsBookmarkedByUser()
        ->withCountBookmarks('DESC', 'bookmarks_count')
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "bookmarks_count": 245,
    "is_bookmarked_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأكثر إشارة مع إضافة عمود الإشارة للمستخدم

```php
public function topBookmarked()
{
    $products = Product::topBookmarked(10)
        ->withIsBookmarkedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لإضافة أو إزالة إشارة مرجعية

```php
public function toggleBookmark($productId, Request $request)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    $value = $request->input('value', 'favorite');
    
    $result = Bookmark::toggle($product, $user, $value);
    
    return response()->json([
        'success' => true,
        'is_bookmarked' => $product->isBookmarked($user, $value),
        'bookmarks_count' => $product->getTotalBookmarks($value)
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::sortByCountBookmarksOld('DESC')->get();

// جديد
$products = Product::sortByCountBookmarks('DESC')->get();
```

```php
// قديم
$products = Product::withSortByCountBookmarks('DESC', 'favorite')->get();

// جديد
$products = Product::withCountBookmarks('DESC', 'bookmarks_count', 'favorite')->get();
```

```php
// قديم
$products = Product::sortByCreatedAtBookmarks('DESC', 'favorite')->get();

// جديد
$products = Product::sortByLatestBookmark('DESC', 'latest_bookmark_at', 'favorite')->get();
```

```php
// قديم
$products = Product::whereHasBookmark($user, 'favorite')->get();

// جديد
$products = Product::bookmarkedByUser($user, 'favorite')->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام أنواع متعددة من الإشارات المرجعية

```php
// تعيين الأنواع المسموحة في الإعدادات
// config/nano/markable.php
return [
    'allowed_values' => [
        'bookmark' => ['favorite', 'read_later', 'wishlist', 'archive'],
    ],
];

// الاستعلام عن منتجات تم إضافتها كنوع 'wishlist'
$wishlist = Product::bookmarkedByUser(null, 'wishlist')->get();
```

#### 9.2 الحصول على عدد الإشارات لكل نوع

```php
$product = Product::find(1);

$bookmarksByType = [
    'favorite' => $product->getTotalBookmarks('favorite'),
    'read_later' => $product->getTotalBookmarks('read_later'),
    'wishlist' => $product->getTotalBookmarks('wishlist'),
];
```

#### 9.3 استخدام النطاق `hasBookmarks` مع حد أدنى متغير من التحكم

```php
$minCount = request('min_bookmarks', 5);
$products = Product::hasBookmarks($minCount, 'favorite')->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountBookmarks('DESC', 'my_favorites_column', 'favorite')->get();
echo $products->first()->my_favorites_column;
```

#### 9.5 الجمع بين `bookmarkedByUser` و `notBookmarkedByUser` في استعلام واحد (باستخدام union)

```php
$bookmarked = Product::bookmarkedByUser()->get();
$notBookmarked = Product::notBookmarkedByUser()->get();
$all = $bookmarked->merge($notBookmarked);
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح منتجات للمستخدم بناءً على إشارات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي (افترض وجود نظام متابعة)
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي أضافها هؤلاء المستخدمون إلى الإشارات (نوع favorite)
$suggestedProducts = Product::whereHas('bookmarks', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers)
      ->where('value', 'favorite');
})->whereNotIn('id', $user->bookmarkedBy()->pluck('id')) // استبعاد ما أضافه المستخدم
  ->withCountBookmarks('DESC', 'favorites_count', 'favorite')
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات الإشارات الجديدة (عند إضافة مستخدم منتجاً إلى الإشارات)

```php
// بعد إضافة الإشارة
$bookmark = Bookmark::add($product, $user, 'favorite');

// إرسال إشعار لمالك المنتج
if ($product->owner_id != $user->id) {
    Notification::send($product->owner, new NewBookmarkNotification($product, $user));
}
```

#### 10.3 عرض عدد الإشارات في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withCountBookmarks('DESC', 'bookmarks_count')
    ->paginate(20);
```

#### 10.4 تحليل سلوك المستخدمين (الأكثر تفاعلاً عبر الإشارات)

```php
// المستخدمون الذين أضافوا أكثر من 10 منتجات إلى الإشارات
$activeUsers = User::has('bookmarks', '>=', 10)->get();

// المنتجات الأكثر إشارة من قبل المستخدمين النشطين (نوع favorite)
$topProducts = Product::whereHas('bookmarks', function($q) use ($activeUsers) {
    $q->whereIn('user_id', $activeUsers->pluck('id'))
      ->where('value', 'favorite');
})->withCountBookmarks('DESC', 'favorites_count', 'favorite')
  ->limit(10)
  ->get();
```

#### 10.5 إنشاء قوائم مخصصة (مثل قائمة "أتمنى شراؤها")

```php
// إضافة منتج إلى قائمة "أتمنى شراؤها"
$product = Product::find(1);
Bookmark::add($product, $user, 'wishlist', [
    'priority' => 'high',
    'expected_price' => 500
]);

// جلب قائمة "أتمنى شراؤها"
$wishlist = Product::bookmarkedByUser(null, 'wishlist')
    ->with(['bookmarks' => function($q) use ($user) {
        $q->where('value', 'wishlist')
          ->where('user_id', $user->id);
    }])
    ->get();

// عرض قائمة الأمنيات مع الأولويات
foreach ($wishlist as $item) {
    $bookmark = $item->bookmarks->first();
    echo $item->name . ' - أولوية: ' . ($bookmark->metadata['priority'] ?? 'عادي');
}
```

---

### الخلاصة

يوفر سلوك `BookmarkableModel` مجموعة غنية من الأدوات التي تمكنك من بناء نظام إشارات مرجعية متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل سلوك المستخدمين، وتحسين أداء التطبيقات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.