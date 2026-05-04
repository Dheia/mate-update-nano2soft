# توثيق سلوك `KycDocumentModel` ونطاقات الوثائق المساعدة

## نظرة عامة

`Nano3\Kyc\Behaviors\KycDocumentModel` هو سلوك (Behavior) يُضاف إلى أي موديل في برمجيات نانوسوفت (NanoSoft App) لربطه بوثائق اعرف عميلك (KYC) كموديل مالك. يتكون السلوك من:

- **الكلاس الرئيسي** `KycDocumentModel` الذي يعرّف علاقة `documents` (مورفيك) مع موديل `Document`.
- **السمة** `KycDocumentScopesAndHelpers` التي تحتوي على نطاقات متقدمة للاستعلام عن الوثائق، ودوال إحصائية، ودوال فحص مباشرة.

بمجرد إضافة هذا السلوك إلى موديل (مثلاً `RainLab\User\Models\User`)، يصبح بإمكانك:

- استخدام العلاقة `$user->documents` لاستعراض وثائقه.
- استخدام نطاقات قوية مثل `whereHasDocuments` لفلترة النتائج بناءً على خصائص الوثائق (النوع، الفئة، الحالة، التحقق...).
- استخدام دوال إحصائية جاهزة مثل `documents_count` و `verified_documents_count`.
- التحقق من وجود وثائق بأنواع معينة بسرعة عبر دوال مثل `hasVerifiedDocument()`.

السلوك مصمم ليكون متوافقاً بالكامل مع `KycDocumentManager` الذي يقوم بتطبيقه تلقائياً على النماذج المسجلة، ولكن يمكن أيضاً تطبيقه يدوياً باستخدام `extendClassWith`.

---

## العلاقة `documents`

عند تطبيق السلوك، يتم إضافة العلاقة التالية إلى الموديل:

```php
public $morphMany = [
    'documents' => [
        Document::class,
        'name' => 'owner',
    ],
];
```

هذا يعني أن أي موديل يطبق السلوك يصبح مالكاً للوثائق عبر الحقلين `owner_id` و `owner_type` في جدول `nano3_kyc_documents`. يمكن استخدام العلاقة مباشرة:

```php
$user = User::find(1);
foreach ($user->documents as $doc) {
    echo $doc->document_type . ' - ' . $doc->status;
}
```

---

## النطاق المتقدم `scopeWhereHasDocuments`

هذا هو النطاق الرئيسي الذي يوفّر فلترة شاملة لوجود وثائق مع شروط مرنة. يمكن استخدامه مباشرة على أي استعلام.

### التوقيع

```php
public function scopeWhereHasDocuments(
    \October\Rain\Database\Builder $builder,
    array $options = []
): \October\Rain\Database\Builder
```

### الخيارات المدعومة (`$options`)

| المفتاح | النوع | الافتراضي | الوصف |
|---------|------|-----------|-------|
| `type` | `string\|array` | `null` | نوع الوثيقة (مثل `'passport'`) أو مصفوفة `['passport','national_id']`. |
| `category` | `string\|array` | `null` | فئة الوثيقة (مثل `'primary_id'`) أو مصفوفة. |
| `status` | `string\|array` | `null` | حالة الوثيقة (مثل `'verified'`) أو مصفوفة. |
| `has_expiry` | `bool` | `null` | `true` للتي لها تاريخ انتهاء، `false` للتي ليس لها. |
| `is_verified` | `bool` | `null` | `true` للتي تم التحقق منها، `false` للتي لم تتحقق. |
| `conditions` | `array\|callable` | `[]` | شروط إضافية على جدول الوثائق (مصفوفة أو closure). |
| `boolean` | `string` | `'and'` | طريقة ربط الشرط بالاستعلام الأصلي: `'and'` أو `'or'`. |
| `not` | `bool` | `false` | إذا كان `true`، يعكس الشرط (يستخدم `whereDoesntHave`). |
| `count` | `int` | `null` | يحدد عدد الوثائق المطلوبة (يستخدم `has` مع عامل مقارنة). عند تحديده، يتم استخدام `has` بدلاً من `exists`. |
| `countOperator` | `string` | `'>=`' | عامل المقارنة للعدد (`'>=', '=', '<', ...`). |

### طريقة العمل

- النطاق يستخدم `whereHas` (أو `orWhereHas` / `whereDoesntHave` / `orWhereDoesntHave`) على العلاقة `documents` مع إغلاق يضيف الشروط المحددة إلى الاستعلام الفرعي.
- عند تحديد `count`، يتم استخدام `whereHas('documents', ..., $countOperator, $count)` بدلاً من `whereHas` بسيط، مما يسمح بفلترة حسب عدد الوثائق.

### أمثلة على `scopeWhereHasDocuments`

```php
// المستخدمون الذين لديهم وثيقة هوية أساسية معتمدة
User::whereHasDocuments([
    'category'    => 'primary_id',
    'is_verified' => true,
])->get();

// المستخدمون الذين لديهم جواز سفر أو هوية وطنية بحالة pending
User::whereHasDocuments([
    'type'   => ['passport', 'national_id'],
    'status' => 'pending',
])->get();

// المستخدمون الذين ليس لديهم أي وثيقة (لا توجد وثائق)
User::whereHasDocuments(['not' => true])->get();

// المستخدمون الذين لديهم على الأقل وثيقتين تم التحقق منهما
User::whereHasDocuments([
    'is_verified'   => true,
    'count'         => 2,
    'countOperator' => '>=',
])->get();

// ربط الفلتر بـ OR مع استعلام أصلي (على سبيل المثال، شرط آخر)
User::where('is_active', true)
    ->whereHasDocuments(['status' => 'verified'], 'or')
    ->get();

// استخدام closure لشروط مخصصة
User::whereHasDocuments([
    'conditions' => function ($q) {
        $q->where('expiry_date', '>=', Carbon::now()->addMonth());
    }
])->get();
```

---

## النطاقات المساعدة

تم بناء مجموعة من النطاقات المختصرة التي تسهل استخدام `scopeWhereHasDocuments` في الحالات الشائعة:

| النطاق | المعاملات | المكافئ |
|--------|-----------|---------|
| `scopeHasDocuments($query, $count=1, $boolean='and')` | عدد الوثائق وطريقة الربط | `whereHasDocuments(['count' => $count, 'boolean' => $boolean])` |
| `scopeHasVerifiedDocuments($query, $count=1, $boolean='and')` | عدد الوثائق المعتمدة | `whereHasDocuments(['is_verified' => true, 'count' => $count, 'boolean' => $boolean])` |
| `scopeHasDocumentType($query, $type, $boolean='and')` | نوع الوثيقة | `whereHasDocuments(['type' => $type, 'boolean' => $boolean])` |
| `scopeHasDocumentCategory($query, $category, $boolean='and')` | فئة الوثيقة | `whereHasDocuments(['category' => $category, 'boolean' => $boolean])` |
| `scopeHasDocumentStatus($query, $status, $boolean='and')` | حالة الوثيقة | `whereHasDocuments(['status' => $status, 'boolean' => $boolean])` |

**أمثلة:**

```php
// المستخدمون الذين لديهم وثيقة معتمدة واحدة على الأقل
User::hasVerifiedDocuments()->get();

// المستخدمون الذين لديهم وثيقة من فئة العناوين
User::hasDocumentCategory('address')->get();

// المستخدمون الذين لديهم وثيقة بحالة "منتهية" (or)
User::where('role', 'customer')
    ->hasDocumentStatus('expired', 'or')
    ->get();
```

---

## نطاقات عدد الوثائق (Counting Scopes)

تسمح هذه النطاقات بإضافة عدد الوثائق إلى نتائج الاستعلام أو ترتيب النتائج بناءً على هذا العدد.

### `scopeAddDocumentsCount(Builder $query, $columnName = 'documents_count', $options = [])`

تضيف عموداً محسوباً يمثل عدد الوثائق (مع إمكانية تخصيص فلترة العدد عبر `$options`).

**المعاملات:**
- `$columnName`: اسم العمود المضاف إلى SELECT.
- `$options`: مصفوفة اختيارية لفلترة الوثائق المحسوبة (نفس خيارات `whereHasDocuments` بدون `boolean`/`not`)، يمكن استخدام `type`, `category`, `status`, `is_verified`, `is_active`.

### `scopeSortByDocumentsCount(Builder $query, $orderDirection = 'DESC', $options = [])`

ترتيب النتائج تصاعدياً أو تنازلياً بناءً على عدد الوثائق (مع إمكانية فلترة العدد).

**المعاملات:**
- `$orderDirection`: `'ASC'` أو `'DESC'`.
- `$options`: مصفوفة فلترة اختيارية (مثل خيارات `addDocumentsCount`).

### أمثلة

```php
// جلب المستخدمين مع عدد وثائقهم (كل الوثائق)
$users = User::addDocumentsCount('total_docs')->get();
foreach ($users as $user) {
    echo $user->total_docs;
}

// جلب المستخدمين مع عدد الوثائق المعتمدة فقط
User::addDocumentsCount('verified_count', ['is_verified' => true])->get();

// ترتيب المستخدمين تنازلياً حسب عدد الوثائق المعتمدة
User::sortByDocumentsCount('DESC', ['is_verified' => true])->get();

// إضافة العمود والترتيب معاً
User::addDocumentsCount('doc_count')->sortByDocumentsCount()->get();
```

**ملاحظة:** الدالة `buildDocumentCountSubQuery` هي الدالة الداخلية التي تبني الاستعلام الفرعي المستخدم في هذه النطاقات.

---

## دوال إحصائية (Accessors)

هذه الدوال تُضاف إلى الموديل الموسع لتوفير إحصائيات سريعة:

| الدالة | الوصف |
|--------|-------|
| `getDocumentsCountAttribute()` | إجمالي عدد الوثائق المرتبطة بالكيان. |
| `getVerifiedDocumentsCountAttribute()` | عدد الوثائق المعتمدة. |
| `getPendingDocumentsCountAttribute()` | عدد الوثائق المعلقة. |

**استخدام:**

```php
$user = User::find(1);
echo $user->documents_count;              // 5
echo $user->verified_documents_count;     // 3
echo $user->pending_documents_count;      // 1
```

---

## دوال فحص (Checkers)

دوال بسيطة تعيد قيم منطقية للتحقق من وجود وثائق معينة:

| الدالة | الوصف |
|--------|-------|
| `hasAnyDocument(): bool` | هل يمتلك الكيان أي وثيقة؟ |
| `hasVerifiedDocument(): bool` | هل يمتلك وثيقة معتمدة واحدة على الأقل؟ |
| `hasDocumentOfType(string $type): bool` | هل يمتلك وثيقة من نوع محدد (مثل `'passport'`)؟ |
| `getDocuments($options = [])` | جلب الوثائق المرتبطة بالكيان (مغلف لـ `$this->model->documents()->get()`). |

**أمثلة:**

```php
$user = User::find(1);

if ($user->hasAnyDocument()) {
    // عرض وثائقه
}

if ($user->hasVerifiedDocument()) {
    // السماح بالشراء
}

if ($user->hasDocumentOfType('passport')) {
    // جواز السفر موجود
}

// جلب جميع الوثائق
$docs = $user->getDocuments();
```

---

## أمثلة عملية متقدمة

### 1. فلترة المستخدمين حسب حالة وتاريخ انتهاء الوثيقة

```php
$expiringUsers = User::whereHasDocuments([
    'status'         => 'verified',
    'has_expiry'     => true,
    'conditions'     => function ($q) {
        $q->where('expiry_date', '>=', now())
          ->where('expiry_date', '<=', now()->addDays(30));
    }
])->get();
```

### 2. تقرير بأفضل 10 مستخدمين بعدد الوثائق المعتمدة

```php
$topUsers = User::addDocumentsCount('verified_docs', ['is_verified' => true])
    ->sortByDocumentsCount('DESC', ['is_verified' => true])
    ->limit(10)
    ->get();
```

### 3. التحقق من استيفاء متطلبات KYC قبل إجراء عملية

```php
$user = Auth::getUser();
if ($user->hasVerifiedDocument()) {
    // السماح بالعملية
} else {
    throw new \Exception('يجب تقديم وثيقة هوية معتمدة.');
}
```

### 4. استخدام النطاق مع علاقات أخرى

```php
// المدن التي لديها مستخدمون بوثائق غير معتمدة
City::whereHas('users', function ($q) {
    $q->whereHasDocuments(['is_verified' => false]);
})->get();
```

---

## ملاحظات هامة

- **الاعتماد على `Document`**: النطاقات والدوال تفترض وجود موديل `Document` وجدول `nano3_kyc_documents`. تأكد من تثبيت إضافة `Nano3.Kyc` وتشغيل الهجرات.
- **الأداء**: استخدام `whereHas` مع عدد كبير من الوثائق يمكن أن يكون مكلفاً. يُنصح باستخدام الفهارس المناسبة (مثل `owner_id`, `owner_type`, `document_type`, `status`) والتي تم إنشاؤها بواسطة الهجرة.
- **التوسع**: يمكنك إضافة نطاقات إضافية إلى الموديل الموسع دون تعديل الكلاس الأصلي.
- **التوافق**: السلوك مصمم للعمل مع OctoberCMS Builder. جميع النطاقات تقبل `Builder` وتعيد `Builder` مما يسمح بتسلسل الاستعلامات.

---

## الخلاصة

سلوك `KycDocumentModel` والسمة `KycDocumentScopesAndHelpers` يوفران طبقة قوية وسهلة لإدارة وثائق KYC لأي موديل. من خلال النطاق المتقدم `whereHasDocuments` والنطاقات المساعدة، يمكنك بناء استعلامات معقدة وفلترة متطورة بسهولة. دوال الإحصاء والفحص تجعل التحقق من حالة الوثائق أمراً بسيطاً ومباشراً، مما يسرّع تطوير تطبيقات متطلبات الامتثال في منصات نانوسوفت.

## التوثيق الإضافي

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./Docs-Document-Model-ar.md)
- [توثيق سلوك `DynamicAddIncludeKyc`](./Docs-DynamicAddIncludeKyc-Behaviors-ar.md)
- [توثيق كلاس `KycDocumentManager`](./Docs-KycDocumentManager-Class-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./Docs-API-Documentation-ar.md)
