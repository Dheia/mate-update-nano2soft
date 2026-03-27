## أمثلة عملية متقدمة لاستخدام سلوك `FollowableModel`

يقدم هذا القسم مجموعة من الأمثلة المتكاملة التي توضح كيفية استخدام سلوك `FollowableModel` في سيناريوهات حقيقية، بدءاً من العمليات الأساسية وصولاً إلى الاستعلامات المعقدة والإحصائيات المتقدمة.

---

### 1. العمليات الأساسية

#### 1.1 إضافة متابعة جديدة مع بيانات مخصصة

```php
use Nano\Follows\Models\Follow;
use Nano\AuthApi\Classes\AuthHelpers;

$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إضافة متابعة عادية (مقبولة افتراضياً)
$follow = $product->addFollow([], $user);

// إضافة متابعة مع تعطيل القبول (متابعة معلقة)
$follow = $product->addFollow(['is_accepted' => false], $user);

// إضافة متابعة مع محتوى إضافي (مثلاً سبب المتابعة)
$follow = $product->addFollow([
    'content' => 'متابعة بغرض الحصول على عروض خاصة',
    'is_accepted' => true,
], $user);
```

#### 1.2 التبديل بين المتابعة والإلغاء (Toggle)

```php
$user = AuthHelpers::getCurrentUser();
$product = Product::find(1);

// إذا كان المستخدم يتابع، يقوم بحذف المتابعة
// إذا كان لا يتابع، يقوم بإضافتها
$product->toggleFollow([], $user);
```

#### 1.3 تحديث حالة متابعة موجودة

```php
$user = AuthHelpers::getCurrentUser();
$follow = $product->getUserFollow($user);

if ($follow) {
    // قبول المتابعة بعد أن كانت معلقة
    $follow = $product->updateFollow($follow->id, ['is_accepted' => true, 'accepted_at' => now()]);
}
```

#### 1.4 حذف متابعة (Soft Delete)

```php
$user = AuthHelpers::getCurrentUser();
$follow = $product->getUserFollow($user);

if ($follow) {
    $product->deleteFollow($follow->id);
}
```

---

### 2. التصفية حسب المستخدمين

#### 2.1 جلب جميع المنتجات التي يتابعها المستخدم الحالي

```php
$myFollowedProducts = Product::followedByUser()->get();

// مع تضمين المتابعات غير المقبولة
$myFollowedProducts = Product::followedByUser(null, false)->get();

// مع تضمين المتابعات المحذوفة (Soft Deletes)
$myFollowedProducts = Product::followedByUser(null, true, null, true)->get();
```

#### 2.2 جلب المنتجات التي لا يتابعها المستخدم الحالي (لاقتراحها)

```php
$suggestedProducts = Product::notFollowedByUser()->paginate(20);
```

#### 2.3 استخدام النطاق القديم `whereHasFollow` (للتوافق العكسي)

```php
$products = Product::whereHasFollow($user, true)->get();
```

#### 2.4 جلب المنتجات التي يتابعها مستخدم معين مع تحميل تفاصيل المتابعة

```php
$user = User::find(2);
$products = Product::followedByUser($user)
    ->with(['follows' => function($q) use ($user) {
        $q->where('user_id', $user->id);
    }])
    ->get();
```

---

### 3. الترتيب والتصنيف المتقدم

#### 3.1 ترتيب المنتجات حسب عدد المتابعين (الأكثر متابعة)

```php
// باستخدام النطاق المباشر
$topProducts = Product::sortByCountFollows('DESC', 'follows_count', true)->take(10)->get();

// باستخدام الاختصار topFollowed
$topProducts = Product::topFollowed(10)->get();

// ترتيب حسب عدد المتابعات النشطة فقط (بما فيها غير المقبولة)
$topActive = Product::sortByCountFollows('DESC', 'active_follows_count', null, true)->get();
```

#### 3.2 إضافة عمود عدد المتابعات إلى النتائج (بدون ترتيب)

```php
$products = Product::addCountFollows()->get();
foreach ($products as $product) {
    echo $product->name . ' - متابعين: ' . $product->follows_count;
}
```

#### 3.3 ترتيب حسب أحدث متابعة (آخر مستخدم قام بالمتابعة)

```php
// ترتيب حسب تاريخ إنشاء آخر متابعة (الأحدث أولاً)
$products = Product::sortByLatestFollow('DESC')->get();

// استخدام حقل accepted_at بدلاً من created_at
$products = Product::sortByLatestFollow('DESC', 'latest_follow_at', true, null, false, 'accepted_at')->get();
```

#### 3.4 إضافة عمود آخر تاريخ متابعة مع الترتيب

```php
$products = Product::withLatestFollow('DESC', 'last_follow_date')->get();
foreach ($products as $product) {
    echo $product->name . ' - آخر متابعة: ' . $product->last_follow_date;
}
```

#### 3.5 ترتيب حسب أقدم متابعة (أول من تابع)

```php
$products = Product::sortByEarliestFollow('ASC')->get();
```

---

### 4. الإحصائيات والتقارير

#### 4.1 إجمالي عدد متابعي منتج معين

```php
$product = Product::find(1);

// كل المتابعين
$total = $product->getTotalFollowers();

// المتابعين المقبولين فقط
$totalAccepted = $product->getTotalFollowers(true);

// المتابعين المقبولين والنشطين فقط
$totalActiveAccepted = $product->getTotalFollowers(true, true);
```

#### 4.2 توزيع المتابعين حسب نوع المستخدم

```php
$counts = $product->getFollowersCountByType(true, true);
// ['RainLab\User\Models\User' => 42, 'App\Models\Admin' => 5]
```

#### 4.3 جلب قائمة بأسماء المستخدمين الذين يتابعون منتجاً معيناً

```php
$users = $product->getFollowersUsers(true, true);
foreach ($users as $user) {
    echo $user->name . "\n";
}
```

#### 4.4 إحصائيات المتابعات في نطاق زمني (باستخدام النطاقات مع where)

```php
use Carbon\Carbon;

$lastWeek = Carbon::now()->subWeek();

// المنتجات التي تمت متابعتها خلال الأسبوع الماضي
$products = Product::whereHas('follows', function($q) use ($lastWeek) {
    $q->where('created_at', '>=', $lastWeek);
})->get();
```

---

### 5. الجمع بين النطاقات لاستعلامات معقدة

#### 5.1 المنتجات الأكثر متابعة (مقبولة) والتي يتابعها المستخدم الحالي أيضاً

```php
$user = AuthHelpers::getCurrentUser();

$products = Product::topFollowed(20)
    ->followedByUser($user)
    ->withIsFollowedByUser()
    ->get();

foreach ($products as $product) {
    echo $product->name . ' (متابعين: ' . $product->follows_count . ') - ';
    echo $product->is_followed_by_user ? 'أنت تتابعه' : 'لا تتابعه';
}
```

#### 5.2 المنتجات التي لها أكثر من 3 متابعين مقبولين، مرتبة حسب آخر متابعة (مع إضافة العمود)

```php
$products = Product::hasFollowers(3, true)
    ->withLatestFollow('DESC', 'last_follow_date')
    ->get();
```

#### 5.3 المنتجات النشطة التي يتابعها المستخدم الحالي، مع عدد المتابعين

```php
$products = Product::active()  // افتراض وجود نطاق active في الموديل
    ->followedByUser()
    ->withCountFollows('DESC', 'follows_count', true)
    ->get();
```

#### 5.4 الحصول على المنتجات التي يتابعها المستخدم الحالي، مع إضافة عمود الحالة للمستخدم نفسه (سيكون 1 دائماً)

```php
$products = Product::followedByUser()
    ->withIsFollowedByUser()
    ->get();
// كل المنتجات ستحوي is_followed_by_user = 1
```

#### 5.5 المنتجات التي لها متابعون مقبولون، مع استبعاد المنتجات التي لا متابعين لها، وترتيبها حسب عدد المتابعين

```php
$products = Product::hasFollowers(1, true)
    ->sortByCountFollows('DESC', 'follows_count', true)
    ->get();
```

---

### 6. استخدام الدوال المساعدة في القوالب (مثال Twig/Blade)

#### 6.1 عرض زر المتابعة/إلغاء المتابعة

```php
@php
$product = Product::find(1);
$isFollowed = $product->isUserFollow();
$followCount = $product->getTotalFollowers(true);
@endphp

<button class="follow-btn" data-id="{{ $product->id }}">
    @if($isFollowed)
        إلغاء المتابعة
    @else
        متابعة
    @endif
</button>
<span class="followers-count">{{ $followCount }} متابع</span>
```

#### 6.2 عرض قائمة المتابعين في صفحة المنتج

```php
$followers = $product->getFollowersUsers(true, true);
?>

<div class="followers-list">
    <h4>أبرز المتابعين</h4>
    @foreach($followers->take(5) as $user)
        <div class="follower">
            <img src="{{ $user->avatar }}" width="40">
            {{ $user->name }}
        </div>
    @endforeach
</div>
```

---

### 7. أمثلة على واجهات API (JSON)

#### 7.1 إرجاع تفاصيل المنتج مع معلومات المتابعة

```php
public function show($id)
{
    $product = Product::withIsFollowedByUser()
        ->withCountFollows('DESC', 'follows_count', true)
        ->find($id);
    
    return response()->json($product);
}
```

النتيجة:
```json
{
    "id": 1,
    "name": "هاتف ذكي",
    "follows_count": 245,
    "is_followed_by_user": 1
}
```

#### 7.2 قائمة المنتجات الأكثر متابعة مع إضافة عمود المتابعة للمستخدم

```php
public function topFollowed()
{
    $products = Product::topFollowed(10)
        ->withIsFollowedByUser()
        ->get(['id', 'name', 'price']);
    
    return response()->json($products);
}
```

#### 7.3 API لإضافة أو إلغاء متابعة

```php
public function toggleFollow($productId)
{
    $product = Product::findOrFail($productId);
    $user = AuthHelpers::getCurrentUser();
    
    $result = $product->toggleFollow([], $user);
    
    return response()->json([
        'success' => true,
        'is_followed' => $product->isUserFollow($user),
        'followers_count' => $product->getTotalFollowers(true)
    ]);
}
```

---

### 8. التوافق العكسي: استخدام النطاقات القديمة

إذا كنت تعتمد على كود قديم يستخدم النطاقات القديمة، يمكنك الاستمرار في استخدامها، لكن يُنصح بالترقية تدريجياً.

```php
// قديم
$products = Product::addSortByCountFollows('DESC', 'follows_count', true)->get();

// جديد
$products = Product::addCountFollows('DESC', 'follows_count', true)->get();
```

```php
// قديم
$products = Product::sortByLatestAtFollows('DESC', 'latest_follow_at', true, null, false, 'created_at')->get();

// جديد
$products = Product::sortByLatestFollow('DESC', 'latest_follow_at', true, null, false, 'created_at')->get();
```

---

### 9. نصائح وحيل متقدمة

#### 9.1 استخدام `withTrashed` في الإحصائيات لتضمين المتابعات المحذوفة

```php
$totalWithDeleted = $product->getTotalFollowers(null, null, true);
```

#### 9.2 الحصول على عدد المتابعين غير المقبولين فقط

```php
$pending = $product->getTotalFollowers(false);
```

#### 9.3 استخدام النطاق `hasFollowers` مع حد أدنى متغير من التحكم

```php
$minCount = request('min_followers', 5);
$products = Product::hasFollowers($minCount, true)->get();
```

#### 9.4 تخصيص اسم العمود في النطاقات

```php
$products = Product::withCountFollows('DESC', 'my_custom_column', true)->get();
echo $products->first()->my_custom_column;
```

#### 9.5 الجمع بين `followedByUser` و `notFollowedByUser` في استعلام واحد (باستخدام union)

```php
$followed = Product::followedByUser()->get();
$notFollowed = Product::notFollowedByUser()->get();
$all = $followed->merge($notFollowed);
```

---

### 10. حالات استخدام متقدمة في نظام اجتماعي

#### 10.1 اقتراح أصدقاء للمستخدم بناءً على متابعات أصدقائه

```php
$user = AuthHelpers::getCurrentUser();

// المستخدمون الذين يتابعهم المستخدم الحالي
$followedUsers = $user->followedBy()->pluck('id');

// المنتجات التي يتابعها هؤلاء المستخدمون
$suggestedProducts = Product::whereHas('follows', function($q) use ($followedUsers) {
    $q->whereIn('user_id', $followedUsers);
})->whereNotIn('id', $user->followedBy()->pluck('id')) // استبعاد ما يتابعه المستخدم
  ->withCountFollows('DESC', 'follows_count', true)
  ->limit(10)
  ->get();
```

#### 10.2 إشعارات المتابعات الجديدة (عند قبول متابعة)

```php
// بعد قبول المتابعة
$follow = $product->updateFollow($followId, ['is_accepted' => true, 'accepted_at' => now()]);

if ($follow->wasChanged('is_accepted')) {
    // إرسال إشعار للمستخدم
    Notification::send($follow->user, new FollowAcceptedNotification($product));
}
```

#### 10.3 عرض عدد المتابعين في قائمة المنتجات مع التحميل البطيء

```php
$products = Product::withCountFollows('DESC', 'follows_count', true)
    ->paginate(20);
```

---

### الخلاصة

يوفر سلوك `FollowableModel` مجموعة غنية من الأدوات التي تمكنك من بناء نظام متابعة متكامل بسهولة. من خلال الأمثلة المقدمة، يمكنك تطبيق هذه الميزات في مشاريعك لإنشاء تجارب مستخدم تفاعلية، وتحليل سلوك المستخدمين، وتحسين أداء التطبيقات. استخدم النطاقات المتقدمة والدوال الإحصائية لتقليل التعقيد وزيادة الإنتاجية.