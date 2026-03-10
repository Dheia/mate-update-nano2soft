## شرح كلاس DirectiveParser

هذا الكلاس `DirectiveParser` مصمم لتحليل وتطبيق "التوجيهات" (Directives) على استعلامات Eloquent في Laravel. الهدف منه هو تمكين بناء استعلامات ديناميكية ومرنة عن طريق تمرير مصفوفة تحتوي على اسم الدالة (مثل `where`، `whereHas`) ومعاملاتها، مع دعم بعض الميزات المتقدمة مثل استبدال القيم الديناميكية (من الجلسة أو المستخدم الحالي) وتأهيل الأعمدة (qualifying columns) لتجنب الغموض عند استخدام العلاقات.

---

### لماذا نحتاج DirectiveParser؟

في بعض التطبيقات، خاصة تلك التي تبني استعلامات بناءً على إدخال المستخدم أو قواعد ديناميكية (مثل بناء تقارير أو فلاتر متقدمة)، قد يكون من المفيد تمثيل أجزاء الاستعلام كمصفوفات قابلة للتفسير. هذا الكلاس يقوم بتفسير هذه المصفوفات وتنفيذها على `Builder`، مما يسمح ببناء استعلامات معقدة دون كتابة كود ثابت.

---

### هيكل الكلاس والدوال الرئيسية

#### 1. دالة `apply`
```php
public static function apply(Builder $query, array $directive): Builder
```
دالة ثابتة (static) لبدء التطبيق. تنشئ كائنًا من الكلاس وتستدعي `applyDirective`. هذا نمط شائع لتوفير واجهة بسيطة.

#### 2. دالة `applyDirective`
```php
public function applyDirective(Builder $query, array $directive): Builder
```
هذه هي الدالة الأساسية التي تقوم بمعالجة التوجيه.

**المدخلات:**
- `$query`: كائن Builder الحالي.
- `$directive`: مصفوفة تمثل التوجيه. العنصر الأول هو اسم الدالة (Eloquent method)، والعناصر التالية هي المعاملات.

**المنطق:**

- **استخراج اسم الدالة**: `$method = array_shift($directive);` يزيل العنصر الأول من المصفوفة ويخزنه في `$method`.
- **التعامل مع التوجيهات المخصصة (Custom Directive Classes)**: إذا كان `$method` يحتوي على كلمة "Directive" ورمز "\" (أي يشير إلى كلاس مؤهل)، يفترض أنه اسم كلاس يطبق واجهة `Directive`. يتم إنشاء كائن من هذا الكلاس عبر `app($method)` واستدعاء `apply` عليه. هذا يسمح بتوسيع النظام بإضافة كلاسات توجيه مخصصة تنفذ منطقًا معقدًا.
- **التعامل مع دوال العلاقات (whereHas, orWhereHas)**: إذا كانت الدالة هي `whereHas` أو `orWhereHas`، فإنها تحتاج إلى معالجة خاصة لأنها تقبل Closure كمعامل ثاني.
  - يتم استخراج اسم العلاقة (`$relation`) من المعاملات.
  - ثم يتم استدعاء `$query->$method` مع تمرير العلاقة و Closure يقوم ببناء استعلام فرعي.
  - داخل الـ Closure، يتم استدعاء `qualifyDirective` لتأهيل الأعمدة داخل التوجيه الفرعي (إضافة اسم العلاقة كمقدمة للأعمدة غير المؤهلة) ثم تطبيق التوجيه الفرعي على الاستعلام الداخلي باستدعاء `applyDirective` مرة أخرى (استدعاء ذاتي).
- **الحالة العامة**: إذا لم تكن الحالتان السابقتان، يتم التعامل معها كدالة Eloquent عادية.
  - يتم تحويل المعاملات باستخدام `parseParameters` لاستبدال أي قيم ديناميكية (مثل `session.xxx` أو `self.xxx`).
  - إذا كانت الدالة موجودة في كائن Builder (`method_exists`)، يتم استدعاؤها باستخدام `$query->$method(...$parameters)`.
  - النتيجة تُعاد.

---

### دالة `qualifyDirective`
```php
protected function qualifyDirective(array $directive, string $relation): array
```
الغرض منها: تجنب الغموض في أسماء الأعمدة عند استخدام `whereHas`. تقوم بإضافة اسم العلاقة كمقدمة (prefix) لأي عنصر نصي لا يحتوي على نقطة (أي غير مؤهل بالفعل). مثال:
- إذا كان التوجيه: `['where', 'age', '>', 18]` والعلاقة `posts`، ستصبح: `['where', 'posts.age', '>', 18]`.
- هذا يضمن أن العمود يُفسر في سياق العلاقة الصحيحة.

---

### دالة `parseParameters`
```php
protected function parseParameters(array $parameters): array
```
الغرض منها: استبدال العناصر النصية التي تبدأ بـ `session.` أو `self.` بقيم حقيقية من الجلسة أو المستخدم الحالي.

- **session.xxx**: إذا كان العنصر يبدأ بـ `session.`، يتم استخراج المفتاح (بعد `session.`) واستخدام `Session::get($sessionKey)` للحصول على القيمة من الجلسة.
- **self.xxx**: إذا كان العنصر يبدأ بـ `self.`، يتم استخراج المفتاح (بعد `self.`) واستخدام `Auth::user()->{$attributeKey}` للحصول على قيمة من المستخدم المصادق عليه حاليًا.
- **أي عنصر آخر**: يبقى كما هو.

هذه الميزة تسمح ببناء استعلامات ديناميكية تعتمد على سياق الطلب أو المستخدم دون الحاجة لتمرير القيم يدويًا.

---

### مثال توضيحي

لنفترض أن لدينا مصفوفة توجيه تمثل استعلامًا لجلب المستخدمين النشطين الذين لديهم منشورات بكلمة "laravel" في العنوان، مع شرط أن يكون عمر المستخدم أقل من قيمة محفوظة في الجلسة.

```php
$directives = [
    ['where', 'status', '=', 'active'],
    ['whereHas', 'posts', function ($query) {
        $query->where('title', 'LIKE', '%laravel%');
    }],
    ['where', 'age', '<', 'session.max_age'],
];

$query = User::query();
foreach ($directives as $directive) {
    $query = DirectiveParser::apply($query, $directive);
}
```

داخل `applyDirective`، سيتم معالجة كل توجيه:
- الأول: `where` عادي، يتم تحويل المعاملات (`parseParameters`) ولكن لا يوجد تغيير.
- الثاني: `whereHas`، يتم استخراج العلاقة `posts`، ثم إنشاء Closure يستدعي `qualifyDirective` على التوجيه الفرعي (`['where', 'title', 'LIKE', '%laravel%']`) ليصبح `['where', 'posts.title', 'LIKE', '%laravel%']` ثم تطبيقه.
- الثالث: `where`، يتم تحويل `session.max_age` إلى القيمة الفعلية من الجلسة عبر `parseParameters`.

---

### دعم التوجيهات المخصصة

إذا كان لدينا كلاس مخصص مثل:

```php
namespace App\Directives;

use Nano2\QueryBuilder\Classes\Contracts\Directive;
use Illuminate\Database\Eloquent\Builder;

class ActiveUsersDirective implements Directive
{
    public function apply(Builder $query): Builder
    {
        return $query->where('status', 'active');
    }
}
```

يمكن استخدامه في التوجيهات كالتالي:

```php
$directive = ['App\Directives\ActiveUsersDirective'];
$query = DirectiveParser::apply($query, $directive);
```

سيتم التعرف على أن العنصر الأول يحتوي على "Directive" و "\"، فيتم إنشاء الكلاس وتنفيذ `apply` عليه.

---

### ملخص

- **الهدف**: تحويل مصفوفات التوجيه إلى استعلامات Eloquent ديناميكية.
- **آلية العمل**: استخراج اسم الدالة ومعاملاتها، مع معالجة خاصة لـ `whereHas`، ودعم استبدال القيم من الجلسة والمستخدم، ودعم التوجيهات المخصصة.
- **الفوائد**: 
  - بناء استعلامات مرنة من بيانات مهيكلة (مثل JSON أو إدخال المستخدم).
  - تجنب كتابة كود ثابت لكل حالة.
  - إعادة استخدام نفس التوجيهات عبر أجزاء مختلفة من التطبيق.
  - دعم التوسع عبر كلاسات Directive المخصصة.

بهذا نكون قد غطينا كل جوانب الكلاس، من البنية إلى الأمثلة التطبيقية.