# توثيق كلاس `PersonHelper` – مساعد الأشخاص والجهات

## نظرة عامة

يقع كلاس `PersonHelper` في المسار `Tss\Accounts\Helpers\PersonHelper`، وهو المكون الأساسي المسؤول عن إدارة كيانات "الأشخاص" (Persons) أو "الجهات" (Entities) داخل إضافة `Tss.Accounts`. في النظام المحاسبي، يمكن ربط أي حركة مالية (مدين أو دائن) بـ "شخص" لتمثيل العميل، المورد، الموظف، الطالب، المتجر، أو أي كيان آخر له علاقة مالية. هذا الكلاس هو الجسر الذي يربط النظام المحاسبي بمختلف نماذج النظام الأخرى عبر علاقات Morph.

تشمل مسؤوليات الكلاس ما يلي:

- **تحويل أسماء الكلاسات إلى قيم Morph والعكس**: دوال مثل `getMorphClassNano` و `getMorphedModelNano` لتسهيل التعامل مع علاقات Eloquent polymorphic.
- **توفير قوائم اختيارات (Options) واجهات المستخدم**: دوال مثل `getPersonTypeLabelOptions` و `getPersonSymbolLabelOptions` لتوليد مصفوفات جاهزة لقوائم الاختيار في النماذج والتقارير.
- **جلب كائنات الأشخاص**: دالة `getPersonObj` لجلب كائن الشخص الفعلي (مثلاً كائن `Employee` أو `Student`) بناءً على `person_type` و `person_id`.
- **استخراج أسماء وبيانات الاتصال**: دوال `getPersonObjName` و `getPersonObjArray` لاستخراج الاسم ورقم الجوال والبريد الإلكتروني من أي كائن شخص بطريقة موحدة.
- **إدارة الرموز المختصرة (Symbols)**: نظام لربط رموز مختصرة (مثل `'TTYP'`، `'EMPL'`، `'STUD'`) بكل نوع شخص، مما يسهل التخزين والعرض في واجهات المستخدم.

يتميز هذا الكلاس بأن جميع دواله ثابتة (`static`)، مما يسمح باستدعائها بسهولة من أي مكان في التطبيق دون الحاجة إلى إنشاء كائن. كما يعتمد على التخزين المؤقت الداخلي (`$listPersonTypeLabel` و `$listPersonSymbolLabel`) لتحسين الأداء عند تكرار استدعاء نفس القوائم.

---

## هيكل البيانات الأساسي: أنواع الأشخاص المدعومة

يدعم الكلاس 15 نوعاً من الأشخاص بشكل افتراضي، يمكن تفعيلها أو تعطيلها عبر ملف الإعدادات (`tss.accounts::person_type.*`). لكل نوع رمز (Symbol) واسم وصفي (Label) يستخدمان في جميع أنحاء النظام.

| القيمة (Morph Class) | الكلاس (Class) | الرمز | الوصف |
| :--- | :--- | :--- | :--- |
| `TTYP` | `Nano\Tags\Models\Type` | `TTYP` | أنواع الوسوم |
| `TAGS` | `Nano\Tags\Models\Tag` | `TAGS` | الوسوم |
| `TCAT` | `Nano\Tags\Models\Categorie` | `TCAT` | فئات الوسوم |
| `DEPT` | `Tss\Basic\Models\Department` | `DEPT` | الأقسام / الفروع |
| `PROD` | `Nano\Shop\Models\Product` | `PROD` | المنتجات |
| `EMPL` | `Tss\Basic\Models\Employee` | `EMPL` | الموظفون |
| `CUST` | `Tss\SalesAndMarketing\Models\Customer` | `CUST` | العملاء |
| `FUSE` | `RainLab\User\Models\User` | `FUSE` | مستخدمو الواجهة الأمامية |
| `BUSE` | `Backend\Models\User` | `BUSE` | مستخدمو لوحة التحكم |
| `DELI` | `Nano\Deliverys\Models\Delivery` | `DELI` | موصلو الطلبات |
| `SUPP` | `Tss\Purchasing\Models\Suppler` | `SUPP` | الموردون |
| `SYND` | `Tss\Purchasing\Models\Syndical` | `SYND` | الوكلاء / النقابات |
| `STUD` | `Tss\Student\Models\Student` | `STUD` | الطلاب |
| `MPAR` | `Tss\Student\Models\Mparent` | `MPAR` | أولياء الأمور |
| `PART` | `Tss\Accounts\Models\Partner` | `PART` | الشركاء التجاريون |

**ملاحظات حول نظام التفعيل (Config):**

- **`getMorphToClassNameOptions`** و **`getPersonTypeLabelOptions`** تراجعان إعدادات `Config` (`tss.accounts::person_type.*`). بعض الأنواع مفعلة افتراضياً (مثل `tags_type`, `department`) بينما البعض الآخر غير مفعل (`employee`, `customer`...).
- **`getMorphToSymbolOptions`** و **`getClassNameToSymbolOptions`** تستخدمان معامل `$stop_check`. إذا كان `$stop_check = true`، يتم تضمين جميع الأنواع بغض النظر عن الإعدادات. هذا مفيد في السيناريوهات التي تحتاج إلى كل الاحتمالات دون الرجوع إلى `Config`.

---

## الدوال العامة (Public API)

### المجموعة الأولى: دوال التحويل بين Morph و Class

#### 1.1 `getMorphClassNano`

تحويل اسم كلاس (أو كائن) إلى قيمة Morph Class الخاصة به. تستخدم للحصول على القيمة المختصرة التي تُخزن في عمود `*_type` في قاعدة البيانات.

```php
public static function getMorphClassNano($class): ?string
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$class` | `string\|object` | اسم الكلاس (مثل `\RainLab\User\Models\User`) أو كائن. |

**القيمة المرجعة:** قيمة Morph (مثل `'FUSE'`، `'EMPL'`...) أو `null` إذا تعذر العثور عليها.

**آلية العمل:**
1.  إذا كان `$class` كائناً، يتم تحويله إلى اسم الكلاس.
2.  يحاول استدعاء `app($className)->getMorphClass()` مباشرة.
3.  إذا لم ينجح (اسم الكلاس ليس قيمة Morph صالحة)، يحاول العثور على الكلاس الكامل من اسم المورف ثم الحصول على قيمة `getMorphClass` من كائن جديد.

**مثال:**
```php
$morph = PersonHelper::getMorphClassNano(\RainLab\User\Models\User::class); // 'FUSE'
$morph = PersonHelper::getMorphClassNano($userObject); // 'FUSE'
```

---

#### 1.2 `getMorphedModelNano`

تحويل قيمة Morph Class (مثل `'FUSE'`) إلى اسم الكلاس الكامل (مثل `'RainLab\User\Models\User'`). العكس المباشر لـ `getMorphClassNano`.

```php
public static function getMorphedModelNano($class): ?string
```

| المعامل | النوع | الوصف |
| :--- | :--- | :--- |
| `$class` | `string` | قيمة Morph Class المختصرة. |

**آلية العمل:**
1.  تحاول استخدام `\Illuminate\Database\Eloquent\Relations\Relation::getMorphedModel`.
2.  إذا فشلت، تحاول استخدام `\October\Rain\Database\Relations\Relation::getMorphedModel` (لنسخ أكتوبر المختلفة).

**مثال:**
```php
$fullClass = PersonHelper::getMorphedModelNano('FUSE'); // 'RainLab\User\Models\User'
```

---

#### 1.3 `getMorphedModelObjectNano`

إنشاء كائن جديد من نوع Morph Class محدد.

```php
public static function getMorphedModelObjectNano($class): ?object
```

**مثال:**
```php
$object = PersonHelper::getMorphedModelObjectNano('FUSE'); // new RainLab\User\Models\User()
```

---

#### 1.4 `getPersonTypeClassName`

دالة تحويل شاملة: تقبل قيمة Morph Class (مثل `'FUSE'`) وتعيد اسم الكلاس الكامل. تستخدم داخلياً `getClassNameToMorphOptions` و `getMorphedModelNano`.

```php
public static function getPersonTypeClassName($person_type): ?string
```

**مثال:**
```php
$class = PersonHelper::getPersonTypeClassName('FUSE'); // 'RainLab\User\Models\User'
$class = PersonHelper::getPersonTypeClassName('EMPL'); // 'Tss\Basic\Models\Employee'
```

---

### المجموعة الثانية: دوال الخيارات والقوائم (Options)

#### 2.1 `getPersonTypeLabelOptions`

توليد قائمة خيارات بأسماء أنواع الأشخاص القابلة للترجمة، للاستخدام في القوائم المنسدلة.

```php
public static function getPersonTypeLabelOptions(): array
```

**القيمة المرجعة:** مصفوفة على الشكل `['FUSE' => 'مستخدم أمامي', 'EMPL' => 'موظف', ...]`.

تستخدم تخزيناً مؤقتاً داخلياً (`$listPersonTypeLabel`).

**مثال:**
```php
$options = PersonHelper::getPersonTypeLabelOptions();
// [
//     ""    => "اختر نوع الشخص",
//     "TTYP" => "أنواع الوسوم",
//     "DEPT" => "الأقسام",
//     "EMPL" => "الموظفون",
//     ...
// ]
```

---

#### 2.2 `getPersonSymbolLabelOptions`

توليد قائمة بالرموز المختصرة (مثل `'TTYP'`، `'EMPL'`).

```php
public static function getPersonSymbolLabelOptions(): array
```

**القيمة المرجعة:** مصفوفة على الشكل `['FUSE' => 'FUSE', 'EMPL' => 'EMPL', ...]`.

**مثال:**
```php
$symbols = PersonHelper::getPersonSymbolLabelOptions();
// ["TTYP" => "TTYP", "DEPT" => "DEPT", "EMPL" => "EMPL", ...]
```

---

#### 2.3 `getMorphToClassNameOptions`

مصفوفة عكسية: مفتاح = قيمة Morph Class، قيمة = اسم الكلاس الكامل.

```php
public static function getMorphToClassNameOptions(): array
```

---

#### 2.4 `getClassNameToMorphOptions`

عكس السابقة: مفتاح = اسم الكلاس الكامل، قيمة = قيمة Morph Class.

```php
public static function getClassNameToMorphOptions(): array
```

---

#### 2.5 `getMorphToSymbolOptions`

مصفوفة تربط Morph Class بالرمز المختصر (معامل `$stop_check` يتحكم في تجاهل إعدادات التفعيل).

```php
public static function getMorphToSymbolOptions($stop_check = true): array
```

#### 2.6 `getClassNameToSymbolOptions`

نفس السابقة ولكن المفتاح هو اسم الكلاس الكامل.

```php
public static function getClassNameToSymbolOptions($stop_check = true): array
```

---

### المجموعة الثالثة: جلب كائنات الأشخاص وبياناتهم

#### 3.1 `getPersonObj`

جلب كائن الشخص الفعلي بناءً على نوعه ومعرفه.

```php
public static function getPersonObj($person_type = null, $person_id = null, $auto_fill = true): ?object
```

- إذا لم يُمرر `$person_id`، يتم إرجاع كائن جديد فارغ.

**مثال:**
```php
$user = PersonHelper::getPersonObj('FUSE', 15);
$newEmployee = PersonHelper::getPersonObj('EMPL'); // كائن فارغ
```

---

#### 3.2 `getPersonObjArray`

استخراج بيانات الشخص (الاسم، الجوال، البريد الإلكتروني) كمصفوفة أو قيمة مفردة. تبحث عن الاسم في `name`, `full_name`, `first_name`, `username`, `login` بالتسلسل.

```php
public static function getPersonObjArray($person_type = null, $person_id = null, $filed = 'all', $auto_fill = true): mixed
```

**أمثلة:**
```php
$data = PersonHelper::getPersonObjArray('FUSE', 15);
// ['name' => 'أحمد', 'mobile' => '0123456789', 'email' => 'a@b.com']

$name = PersonHelper::getPersonObjArray($userObject, null, 'name');
```

---

#### 3.3 `getPersonObjLabel`

دالة مغلفة لـ `getPersonObjArray` لتحديد حقل واحد (`name`, `mobile`, `email`).

```php
public static function getPersonObjLabel($person_type = null, $person_id = null, $filed = 'name', $auto_fill = false): mixed
```

#### 3.4 `getPersonObjName`

اختصار لجلب الاسم فقط. تستخدم بكثرة في `TransactionsHelper`.

```php
public static function getPersonObjName($person_type = null, $person_id = null, $auto_fill = false): mixed
```

**مثال:**
```php
$name = PersonHelper::getPersonObjName('STUD', 120); // 'محمد علي'
```

---

### المجموعة الرابعة: دوال الرموز المختصرة

#### 4.1 `getPersonSymbolName`

جلب الرمز المختصر (Symbol) لنوع شخص معين.

```php
public static function getPersonSymbolName($class_name, $defaultValue = null, $stop_check = true): ?string
```

#### 4.2 `getPersonTypeLabel`

جلب الاسم الوصفي القابل للترجمة لنوع شخص معين.

```php
public static function getPersonTypeLabel($class_name, $defaultValue = null): ?string
```

---

## التخزين المؤقت الداخلي

يستخدم الكلاس خاصيتين ثابتتين لتحسين الأداء:

```php
public static $listPersonTypeLabel = null;
public static $listPersonSymbolLabel = null;
```

عند أول استدعاء لدوال `getPersonTypeLabelOptions` أو `getPersonSymbolLabelOptions`، يتم بناء المصفوفة وتخزينها. الاستدعاءات اللاحقة تعيد القيمة المخزنة. لتحديث القوائم بعد تغيير الإعدادات خلال نفس الطلب:

```php
PersonHelper::$listPersonTypeLabel = null;
PersonHelper::$listPersonSymbolLabel = null;
```

---

## سيناريوهات الاستخدام

### السيناريو 1: بناء قائمة منسدلة لاختيار نوع الشخص

```php
$options = PersonHelper::getPersonTypeLabelOptions();
echo Form::select('person_type', $options, null, ['class' => 'form-control']);
```

### السيناريو 2: عرض اسم الطرف المرتبط بقيد محاسبي

```php
$detail = TransactionsDetail::find(100);
$name = PersonHelper::getPersonObjName($detail->person_type, $detail->person_id);
echo "الطرف: " . $name;
```

### السيناريو 3: التحقق من صلاحية `person_type` مدخل

```php
$validTypes = PersonHelper::getPersonTypeLabelOptions();
if (!array_key_exists($request->input('person_type'), $validTypes)) {
    throw new ApplicationException('نوع الشخص غير صالح.');
}
```

### السيناريو 4: إضافة نوع شخص جديد (للمطورين)

1.  أضف النوع إلى جميع دوال الخيارات الست (`getMorphToClassNameOptions`, `getMorphToSymbolOptions`, `getMorphToSymbolOptions`, `getClassNameToSymbolOptions`, `getPersonTypeLabelOptions`, `getPersonSymbolLabelOptions`).
2.  أضف مفتاح التفعيل في `Config` (`tss.accounts::person_type.new_type`).
3.  تأكد من أن النموذج المقابل يدعم `MorphMany`/`MorphTo`.

---

## الاعتماديات

- `Illuminate\Database\Eloquent\Relations\Relation` و `October\Rain\Database\Relations\Relation`
- `Tss\Accounts\Models\*` (Period, Account, Reference, Currency, TransactionHeader, TransactionsDetail)
- `Config` (لإعدادات `tss.accounts::person_type.*`)
- `Lang` / `trans` (للترجمة)

---

## الخاتمة

يمثل كلاس `PersonHelper` حجر الزاوية في إدارة الكيانات المرتبطة بالحركات المالية. من خلال نظامه الموحد للتحويل بين أنواع Morph، والخيارات الجاهزة للواجهات، وآلية استخراج البيانات الذكية، يبسط الكلاس ربط أي كيان في النظام (مستخدم، طالب، موظف، متجر) بالعمليات المحاسبية. هذا التوثيق يوفر دليلاً شاملاً لاستخدام وتوسيع الكلاس.


## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
