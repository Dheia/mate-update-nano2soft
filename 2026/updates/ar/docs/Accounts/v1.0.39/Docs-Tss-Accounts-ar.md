# توثيق إضافة `Tss.Accounts`

**الإصدار:** 1.0.39  
**المؤلف:** Dheia Ali     
**الرخصة:** خاصة بـ NanoSoft  
**آخر تحديث:** 2026-05-30

---

## المحتويات

1. [مقدمة](#مقدمة)
2. [المتطلبات](#المتطلبات)
3. [التثبيت والترقية](#التثبيت-والترقية)
4. [الميزات الرئيسية](#الميزات-الرئيسية)
5. [هيكل قاعدة البيانات](#هيكل-قاعدة-البيانات)
6. [النماذج (Models)](#النماذج-models)
7. [السمات (Traits)](#السمات-traits)
8. [السلوكيات (Behaviors)](#السلوكيات-behaviors)
9. [الواجهة الخلفية (Backend)](#الواجهة-الخلفية-backend)
10. [إعدادات الإضافة](#إعدادات-الإضافة)
11. [الصلاحيات](#الصلاحيات)
12. [قواعد الإشعارات (Notification Rules)](#قواعد-الإشعارات-notification-rules)
13. [تقارير الطباعة (Print Templates)](#تقارير-الطباعة-print-templates)
14. [أدوات تقارير لوحة التحكم (Report Widgets)](#أدوات-تقارير-لوحة-التحكم-report-widgets)
15. [حقول النماذج المخصصة (Form Widgets)](#حقول-النماذج-المخصصة-form-widgets)
16. [علامات Twig المخصصة (Markup Tags)](#علامات-twig-المخصصة-markup-tags)
17. [دوال المساعدة (Helpers)](#دوال-المساعدة-helpers)
18. [الأحداث (Events)](#الأحداث-events)
19. [أمثلة عملية](#أمثلة-عملية)
20. [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
21. [المراجع](#المراجع)

---

## مقدمة

إضافة `Tss.Accounts` هي نظام محاسبي متكامل لأنظمة NanoSoft APP (برمجيات نانوسوفت)، توفر كافة العمليات المحاسبية الأساسية والمتقدمة. تعتمد الإضافة على هيكل قوي يتضمن دليل حسابات هرمي (Chart of Accounts)، مراكز تكلفة، فترات مالية، سندات قبض وصرف، قيود يومية، تحويلات بين حسابات، أشعار مدينة ودائنة، إدارة الأرصدة الافتتاحية، ترحيلات، بالإضافة إلى دعم كامل للوسائط المتعددة (صور، مستندات) على جميع السندات المحاسبية.

تم تصميم الإضافة لتناسب الشركات والمؤسسات والمدارس (تكامل مع `Tss.School` و `Tss.Student`) وتوفر مرونة عالية عبر إعدادات قابلة للتخصيص وصلاحيات دقيقة لكل نوع سند.

---

## المتطلبات

- **NanoSoft APP** الإصدار 3.x أو 2.x (مع بعض القيود).
- **PHP** >= 8.0.
- **الإضافات المطلوبة** (معلنة في `$require`):
  - `Rainlab.Translate`
  - `Rainlab.Location`
  - `Rainlab.User`
  - `Tss.Tools`
  - `Tss.BarcodeGenerator`
  - `Tss.Basic`
  - `Tss.Currency`
- **الإضافات الموصى بها**:
  - `Nano.MicroCart` (لدعم طرق الدفع المحاسبية).
  - `Nano.Orders` (لتكامل الأوامر مع العمولات).
  - `Tss.School` (لدعم حسابات الطلاب وأولياء الأمور).
  - `Tss.Cards` (لدعم سندات `CreditDelivery`).
  - `Nano.AccountsApi` (لتوفير واجهة API محاسبية متكاملة).

---

## التثبيت والترقية

### تثبيت جديد

```bash
php artisan plugin:install Tss.Accounts
php artisan nano:up
```

### الترقية من إصدار سابق

```bash
composer update tss/accounts
php artisan nano:up
php artisan cache:clear
```

### نشر ملف الإعدادات (اختياري)

```bash
php artisan config:publish Tss.Accounts
```

سيتم إنشاء ملف الإعدادات في `config/tss/accounts/config.php`.

---

## الميزات الرئيسية

| الميزة | الوصف |
| :--- | :--- |
| **دليل الحسابات الهرمي (Chart of Accounts)** | حسابات رئيسية وفرعية (main/sub) بترقيم هرمي، يدعم مستويات غير محدودة، مع إمكانية تحديد طبيعة الحساب (مدين/دائن)، وتقارير (الميزانية/الربح والخسارة). |
| **مراكز التكلفة (Cost Centers)** | هيكل هرمي لمراكز التكلفة، يمكن ربطها بالحسابات والفروع. |
| **الفترات المالية (Periods)** | إدارة سنوات مالية بنطاقات تواريخ، والتحقق من صحة تواريخ السندات. |
| **القيود اليومية (BondsDays / Bonds2Days)** | قيود محاسبية متعددة الأطراف مع دعم القيود الدورية والعكسية. |
| **سندات القبض (CatchReceipts)** | تسجيل المقبوضات النقدية أو البنكية من العملاء أو الموردين أو الطلاب. |
| **سندات الدفع (PayReceipts)** | تسجيل المدفوعات النقدية أو البنكية. |
| **سندات التحويل (Transfers)** | تحويل أموال بين حسابات (صندوق/بنك) مع إمكانية إضافة مصاريف بنكية. |
| **أشعار مدينة ودائنة (DebitNotes / CreditNotes)** | تسجيل التعديلات على حسابات العملاء أو الموردين دون الحاجة لحركة نقدية. |
| **المصروفات والإيرادات (Expenses / Incomes)** | تسجيل المصروفات والإيرادات المختلفة. |
| **الترحيلات (Relays)** | ترحيل السندات دفعة واحدة أو فردياً إلى فترات لاحقة. |
| **الرصيد الافتتاحي (OpeningBalances)** | إدخال أرصدة افتتاحية للحسابات في بداية الفترة المالية. |
| **سندات القيد (BindingBonds)** | قيود محاسبية عامة (متعددة الأطراف). |
| **الشركاء (Partners)** | إدارة حسابات الشركاء والعملاء والموردين والموظفين والطلاب مع ربطها بحسابات محاسبية. |
| **الصناديق والبنوك (Boxes / Banks)** | إدارة حسابات الخزائن والحسابات البنكية، وإنشاء حساباتها تلقائياً في دليل الحسابات. |
| **المجموعات والمراجع (Groups / References)** | تصنيف وتجميع الحسابات لأغراض التقارير. |
| **دعم الوسائط المتعددة (Media)** | إرفاق صور ومستندات بجميع أنواع السندات (أضيف في الإصدار 1.0.39). |
| **توحيد الـ Morph Type** | جميع الملفات المرفقة تستخدم `TransactionHeader` كنوع موحد (`UnifiedMorphClass`). |
| **إدارة الأرقام المرجعية (Paper ID)** | دعم التحقق من تكرار الأرقام المرجعية وفق إعدادات مرنة (حسب الدورة المالية أو نوع السند). |
| **ترقيم آلي للسندات (Auto Numbering)** | توليد أرقام تسلسلية للسندات بناءً على إعدادات النوع والفرع. |
| **صلاحيات دقيقة** | صلاحيات منفصلة لكل نوع سند (access, add, edit, delete, print, email, sms, export, search). |
| **قوالب طباعة مخصصة** | دعم قوالب طباعة متعددة لكل نوع سند (مدمج مع نظام تقارير `Tss.Tools`). |
| **إشعارات (Notifications)** | إطلاق أحداث يمكن استخدامها مع `RainLab.Notify` (مثل إضافة رصيد للموصل). |
| **التكامل مع النظام التعليمي** | دعم خاص للطلاب وأولياء الأمور (حسابات الطلاب، الدفعات، شحن الرصيد). |

---

## هيكل قاعدة البيانات

### الجداول الرئيسية التي تنشئها الإضافة

| الجدول | الوصف |
| :--- | :--- |
| `tss_accounts_accounts` | دليل الحسابات (يدعم Nested Tree). |
| `tss_accounts_cost_centers` | مراكز التكلفة (يدعم Nested Tree). |
| `tss_accounts_periods` | الفترات المالية. |
| `tss_accounts_transaction_headers` | رأس الحركات (جميع أنواع السندات). |
| `tss_accounts_transactions_details` | تفاصيل الحركات (مدين/دائن). |
| `tss_accounts_groups` | مجموعات الحسابات. |
| `tss_accounts_groups_accounts` | ربط الحسابات بالمجموعات. |
| `tss_accounts_references` | مراجع الحسابات (لمراجع مختلفة مثل أنواع الحسابات). |
| `tss_accounts_boxes` | الصناديق. |
| `tss_accounts_banks` | البنوك. |
| `tss_accounts_partners` | الشركاء (عملاء، موردون، موظفون، طلاب، إلخ). |
| `tss_accounts_currency_conversions` | تحويلات العملات. |
| `tss_accounts_payments_company` | طرق الدفع المسموحة. |
| `tss_accounts_relays` | (استخدام الجدول نفسه لترحيل السندات). |

> **ملاحظة:** جميع نماذج السندات تستخدم جدول `tss_accounts_transaction_headers` مع عمود `ref_type` لتحديد نوع السند، و `transactions_type` لتحديد نوع الحركة المحاسبية. جدول التفاصيل `tss_accounts_transactions_details` يربط كل سند بحسابه ومدين/دائن.

### الجداول المساعدة من إضافات أخرى

| الجدول | الاستخدام |
| :--- | :--- |
| `tss_basic_departments` | الفروع (المتاجر). |
| `tss_basic_companies` | الشركات. |
| `tss_currency_currencies` | العملات. |
| `tss_basic_employees` | الموظفون. |
| `tss_student_students` / `tss_student_mparents` | الطلاب وأولياء الأمور. |

---

## النماذج (Models)

### 1. `Account` – دليل الحسابات

```php
class Account extends Model
{
    use \October\Rain\Database\Traits\Validation;
    use \October\Rain\Database\Traits\SoftDelete;
    use \October\Rain\Database\Traits\NestedTree;
    use \October\Rain\Database\Traits\Purgeable;
    use \Tss\Accounts\Traits\UnifiedMorphClass;

    public $table = 'tss_accounts_accounts';
    public $fillable = ['code', 'name', 'account_number', 'account_normal', 'reports', 'parent_id', ...];
    public $belongsTo = [
        'parent' => [Account::class],
        'currency' => [Currency::class],
    ];
    public $hasMany = ['children' => [Account::class, 'key' => 'parent_id']];
}
```

**الخصائص الرئيسية:**

- `account_normal`: `debit` (مدين) أو `credit` (دائن).
- `reports`: `mezinah` (ميزانية) أو `arbeh` (ربح وخسارة).
- `main_sub`: `main` (رئيسي) أو `sub` (فرعي).
- يدعم مستويات غير محدودة بفضل `NestedTree`.

### 2. `TransactionHeader` – رأس الحركة (الأصل لجميع السندات)

جميع نماذج السندات (CatchReceipt, PayReceipt, BondsDay, ...) تمتد من هذا النموذج (أو تستخدم نفس الجدول مع `ref_type`). يحتوي على:

```php
public $table = 'tss_accounts_transaction_headers';
public $hasMany = ['TransactionDetail' => [TransactionsDetail::class]];
public $belongsTo = [
    'Period' => [Period::class],
    'Department' => [Department::class],
    'CostCenter' => [CostCenter::class],
    'TransactionsType' => [TransactionsType::class],
    'Currency' => [Currency::class],
    'Employee' => [Employee::class],
];
public $attachOne = ['image', 'book_intro'];
public $attachMany = ['images', 'videos', 'audios', 'files'];
```

### 3. `TransactionsDetail` – تفاصيل الحركة

```php
class TransactionsDetail extends Model
{
    public $belongsTo = [
        'TransactionHeader' => [TransactionHeader::class],
        'Account' => [Account::class],
    ];
    // حقول: debit, credit, person_type, person_id, notes, ...
}
```

### 4. `CatchReceipt` – سند قبض (مثال)

```php
class CatchReceipt extends TransactionHeader
{
    // يستخدم نفس الجدول مع ref_type=2 مثلاً
    // يضيف دوال خيارات خاصة مثل getAccountTypeOptions, getAccountIdOptions, ...
}
```

### 5. `Period` – الفترة المالية

```php
class Period extends Model
{
    public $dates = ['from_date', 'to_date'];
    // دوال مساعدة: isValidDate($date), isOpen() ...
}
```

### 6. `CostCenter` – مركز تكلفة

```php
class CostCenter extends Model
{
    use NestedTree;
}
```

### 7. `Partner` – شريك (عميل، مورد، موظف، طالب)

```php
class Partner extends Model
{
    public $morphTo = ['partnerable']; // علاقة متعددة الأشكال إلى النموذج الحقيقي (Employee, Customer, ...)
    public $belongsTo = ['Account' => [Account::class]];
    public $attachOne = ['image'];
    public $attachMany = ['files', 'images'];
}
```

### 8. `Boxe` / `Bank` – صناديق وبنوك

```php
class Boxe extends Model
{
    public $belongsTo = ['Account' => [Account::class], 'Currency' => [Currency::class]];
    public $attachOne = ['image'];
}
```

### 9. `Group` / `Reference` – مجموعات ومراجع

```php
class Group extends Model
{
    public $belongsToMany = ['Accounts' => [Account::class, 'table' => 'tss_accounts_groups_accounts']];
}
```

---

## السمات (Traits)

| السمة | الوصف |
| :--- | :--- |
| `HasSequenceDocsNumber` | توليد رقم تسلسلي للسندات (docs_number) بناءً على إعدادات كل نوع سند. |
| `OptionsPerson` | توفير دوال خيارات لأنواع الجهات (شخص) وقوائمها المنسدلة في النماذج. |
| `OptionsToPerson` / `OptionsFromPerson` | نسخ للجهات المستخدمة في جانب الدائن/المدين (خاص بالتحويلات). |
| `UnifiedMorphClass` | توحيد الـ morph class للملفات المرفقة إلى `TransactionHeader::class` (أضيف في 1.0.39). |
| `HasTimestamps` (في `Tss.Basic`) | إضافة حقول `created_by`, `updated_by`, `deleted_by` تلقائياً. |
| `BasicScope` (في `Tss.Basic`) | نطاقات أساسية مثل `IsCompany()`, `Departments()`, `IsActive()`. |

---

## السلوكيات (Behaviors)

### `Tss\Accounts\Behaviors\TransactionHeaderController`

(داخل المتحكمات) يوفر دوال مشتركة لعمليات السندات.

### `Tss\Accounts\Behaviors\TransactionHeaderImportExport`

يدعم استيراد/تصدير السندات من/إلى Excel.

---

## الواجهة الخلفية (Backend)

### القوائم الرئيسية

| القائمة | المسار | الوصف |
| :--- | :--- | :--- |
| الحسابات | `/backend/tss/accounts/accounts` | دليل الحسابات (شجرة). |
| سندات القبض | `/backend/tss/accounts/catchreceipts` | إدارة سندات القبض. |
| سندات الدفع | `/backend/tss/accounts/payreceipts` | إدارة سندات الدفع. |
| القيود اليومية | `/backend/tss/accounts/bondsdays` | إدارة القيود اليومية. |
| سندات التحويل | `/backend/tss/accounts/transfers` | إدارة التحويلات بين الحسابات. |
| الأرصدة الافتتاحية | `/backend/tss/accounts/openingbalances` | إدخال الأرصدة الافتتاحية. |
| المصروفات | `/backend/tss/accounts/expenses` | إدارة المصروفات. |
| الإيرادات | `/backend/tss/accounts/incomes` | إدارة الإيرادات. |
| أشعار مدينة | `/backend/tss/accounts/debitnotes` | أشعار مدين. |
| أشعار دائنة | `/backend/tss/accounts/creditnotes` | أشعار دائن. |
| الترحيلات | `/backend/tss/accounts/relays` | ترحيل السندات. |
| مراكز التكلفة | `/backend/tss/accounts/costcenters` | إدارة مراكز التكلفة. |
| الفترات المالية | `/backend/tss/accounts/periods` | إدارة الفترات المالية. |
| الصناديق | `/backend/tss/accounts/boxes` | إدارة الصناديق. |
| البنوك | `/backend/tss/accounts/banks` | إدارة الحسابات البنكية. |
| الشركاء | `/backend/tss/accounts/partners` | إدارة حسابات الشركاء. |
| دليل الحسابات (التهيئة) | `/backend/tss/accounts/configaccounts` | إعدادات الحسابات. |

### قوائم إدارة الوسائط (من الإصدار 1.0.39)

تم إضافة الأعمدة التالية إلى قوائم **جميع السندات والكيانات المحاسبية**:

- عمود `image` (نوع `simpleimage`) لعرض الصورة الرئيسية.
- عمود `images` (نوع `simpleimages`) لمعرض الصور.
- عمود `files` (نوع `partial`) لعرض روابط الملفات المرفقة.

كما تم إضافة تبويب `المرفقات` في نماذج الإدخال يحتوي على حقول رفع الصور والملفات.

---

## إعدادات الإضافة

توفر الإضافة عدة نماذج إعدادات (Settings) يمكن الوصول إليها من `الإعدادات > إعدادات جزء الحسابات`:

| نموذج الإعدادات | الوصف |
| :--- | :--- |
| `AccountsSetting` | الإعدادات العامة لدليل الحسابات (الحد الأقصى للمستوى، حسابات رأس المال والأرباح والخسائر، إلخ). |
| `AccountsReportsSetting` | إعدادات ترويسة التقارير المحاسبية. |
| `CatchReceiptsSetting` | إعدادات سندات القبض (ترقيم المرجع، السماح بتعديل التاريخ، إلزامية الملاحظات، إلخ). |
| `PayReceiptsSetting` | إعدادات سندات الدفع. |
| `BondsDaysSetting` | إعدادات القيود اليومية. |
| `Bonds2DaysSetting` | إعدادات القيود اليومية (الإصدار الثاني). |
| `TransfersSetting` | إعدادات التحويلات بين الحسابات. |
| `CreditNotesSetting` | إعدادات أشعار دائن. |
| `DebitNotesSetting` | إعدادات أشعار مدين. |
| `ExpensesSetting` | إعدادات المصروفات. |

### الإعدادات المتوفرة (أمثلة)

- `is_custome_date`: السماح بتعديل تاريخ السند يدوياً.
- `is_force_notes`: إلزامية كتابة الملاحظات.
- `is_paper_id` / `validat_paper_id`: التحكم في استخدام رقم المرجع والتحقق من تكراره.
- `type_rapet`: خيارات التكرار (يمكن تكرار الرقم عبر فترات أو عبر أنواع سندات أخرى).
- `default_notes`: نص الملاحظة الافتراضي.
- `is_auto_paper_id`: توليد تلقائي للرقم المرجعي بشكل متسلسل.
- `is_custom_employees`: السماح باختيار الموظف المرتبط بالسند.
- `is_relays`: ترحيل السند تلقائياً عند الإنشاء.

---

## الصلاحيات

تم تقسيم الصلاحيات حسب نوع السند، مما يسمح بإدارة دقيقة جداً. مثال لصلاحيات `CatchReceipts`:

| الصلاحية | المفتاح | الوصف |
| :--- | :--- | :--- |
| الوصول | `tss.accounts.catch_receipts.access` | الوصول إلى سندات القبض (خاصة المستخدم). |
| الوصول للكل | `tss.accounts.catch_receipts.access_all` | الوصول إلى كل سندات القبض. |
| إضافة | `tss.accounts.catch_receipts.add` | إضافة سند قبض جديد. |
| تعديل | `tss.accounts.catch_receipts.edit` | تعديل سند قبض. |
| حذف | `tss.accounts.catch_receipts.delete` | حذف سند قبض. |
| طباعة | `tss.accounts.catch_receipts.print` | طباعة سند القبض. |
| إرسال SMS | `tss.accounts.catch_receipts.sms` | إرسال سند القبض عبر رسالة نصية. |
| إرسال بريد | `tss.accounts.catch_receipts.email` | إرسال سند القبض عبر البريد الإلكتروني. |
| بحث | `tss.accounts.catch_receipts.search` | البحث في سندات القبض. |
| تصدير | `tss.accounts.catch_receipts.export` | تصدير بيانات سندات القبض. |

نفس الهيكل ينطبق على `PayReceipts`, `BondsDays`, `Transfers`, `DebitNotes`, `CreditNotes`, `Expenses`, `OpeningBalances`, إلخ.

كما توجد صلاحيات لإدارة دليل الحسابات والمجموعات والمراكز والصناديق والبنوك والشركاء.

---

## قواعد الإشعارات (Notification Rules)

توفر الإضافة الأحداث التالية التي يمكن استخدامها في نظام `RainLab.Notify`:

- `tss.accounts.afterAddMonyDelivery` – بعد إضافة رصيد لموصل.
- `tss.accounts.afterAddOpeningBalanceDelivery` – بعد إضافة رصيد افتتاحي لموصل.
- `tss.accounts.afterBrokerage` – بعد خصم عمولة من موصل.

---

## تقارير الطباعة (Print Templates)

تم تسجيل القوالب التالية في نظام تقارير `Tss.Tools`:

- `OpeningBalances`
- `BondsDays`
- `CatchReceipt`
- `PayReceipt`
- `Expense`
- `Income`
- `BindingBonds`
- `Transfers`
- `DebitNotes`

والتنسيق الرئيسي `ReportAccounts`.

---

## أدوات تقارير لوحة التحكم (Report Widgets)

- `Tss\Accounts\ReportWidgets\WelcomeAccounts` – ترحيب بالحسابات.
- `Tss\Accounts\ReportWidgets\WelcomeConfigAccounts` – ترحيب بإعدادات الحسابات.

---

## حقول النماذج المخصصة (Form Widgets)

- `Tss\Accounts\Formwidgets\Price` – حقل سعر مع دعم العملات.
- `Tss\Accounts\Formwidgets\Repeatertss` – مكرر محسّن.

---

## علامات Twig المخصصة (Markup Tags)

| الدالة | الوصف |
| :--- | :--- |
| `get_old_trans_report($data)` | عرض السندات السابقة لحركة مالية معينة. |
| `get_accounts_person_type_label($person_type)` | عرض نص نوع الجهة (عميل، مورد، إلخ). |
| `get_accounts_person_obj_label($custom_name, $person_type, $person_id)` | عرض اسم الجهة بناءً على نوعها ومعرفها. |
| `get_accounts_person_symbol_name($person_type)` | عرض رمز نوع الجهة. |

---

## دوال المساعدة (Helpers)

### `Tss\Accounts\Helpers\AccountHelper`

دوال للتعامل مع الحسابات، منها:

- `getBalances($accountCode, $options)` – جلب رصيد حساب مع مراعاة الفترة والفرع ومركز التكلفة والعملة والجهة.
- `getAccountsTree()` – الحصول على شجرة الحسابات.
- `getAccountByNumber($number)` – البحث عن حساب برقمه.

### `Tss\Accounts\Helpers\PersonHelper`

- `getPersonTypeLabel($type, $default)` – ترجمة نوع الجهة.
- `getPersonObjName($type, $id)` – الحصول على اسم الجهة الحقيقية (عميل، طالب، إلخ).
- `getPersonSymbolName($type)` – الحصول على رمز الجهة.

### `Tss\Accounts\Helpers\TransactionsHelper`

(موجود في الإصدارات الحديثة، يوفر دوال متقدمة لإنشاء القيود المحاسبية، فحص الأرصدة، فروق العملات، إلخ).

---

## الأحداث (Events)

ترسل الإضافة الأحداث التالية (يمكن الاستماع إليها عبر `Event::listen`):

| اسم الحدث | المعاملات | الوصف |
| :--- | :--- | :--- |
| `tss.accounts.beforeSaveTransaction` | `$header, $details` | قبل حفظ أي حركة مالية (رأس وتفاصيل). |
| `tss.accounts.afterSaveTransaction` | `$header, $details` | بعد حفظ الحركة. |
| `tss.accounts.beforeRelayTransaction` | `$header` | قبل ترحيل سند. |
| `tss.accounts.afterRelayTransaction` | `$header` | بعد ترحيل سند. |
| `tss.accounts.beforeDeleteTransaction` | `$header` | قبل حذف سند. |
| `tss.accounts.afterDeleteTransaction` | `$header` | بعد حذف سند. |

---

## أمثلة عملية

### 1. إنشاء قيد يومي (BondsDay) عبر الكود

```php
$data = [
    'departments_id' => 1,
    'periods_id' => 3,
    'cost_centers_id' => 2,
    'date_at' => Carbon::now(),
    'notes' => 'شراء أثاث',
    'bonds' => [
        ['accounts_id' => '2-3-1211010001', 'process_type' => 'debit', 'mony' => 5000],
        ['accounts_id' => '2-3-1253010001', 'process_type' => 'credit', 'mony' => 5000],
    ]
];
$bond = new BondsDay($data);
$bond->save(); // سيقوم تلقائياً بتوليد رقم السند والتحقق من التوازن
```

### 2. إضافة سند قبض لطالب

```php
$receipt = new CatchReceipt();
$receipt->departments_id = 1;
$receipt->periods_id = 3;
$receipt->account_type = 'current_student'; // أو '24' لطلاب
$receipt->account_id = 1232010001; // رقم حساب الطالب
$receipt->mony = 1000;
$receipt->process_type = 'cach';
$receipt->ac_boxs = '2-2-1253010001'; // حساب الصندوق
$receipt->notes = 'دفعة شهر مارس';
$receipt->save();
```

### 3. جلب رصيد حساب معين

```php
$options = [
    'departments_id' => 1,
    'periods_id' => 3,
    'currencys_id' => 1,
];
$balance = AccountHelper::getBalances('2-2-1232010001', $options);
echo "الرصيد: {$balance}";
```

### 4. إرفاق صورة لسند قبض (الواجهة الخلفية)

في النموذج، سيظهر تبويب `المرفقات` يمكن من خلاله رفع الصورة. بعد الحفظ، يمكن الوصول إليها عبر `$receipt->image`.

### 5. استخدام السمة `UnifiedMorphClass` في نموذج مخصص

```php
use Tss\Accounts\Traits\UnifiedMorphClass;

class MyCustomVoucher extends Model
{
    use UnifiedMorphClass;
    // ...
}
```

---

## استكشاف الأخطاء وإصلاحها

### المشكلة: خطأ "القيد غير متزن"

**السبب:** مجموع المبالغ المدينة لا يساوي مجموع الدائن.

**الحل:** تأكد من أن مصفوفة `bonds` تحتوي على قيدين على الأقل ومجموع `debit` يساوي مجموع `credit`.

### المشكلة: خطأ "التاريخ خارج نطاق الفترة المالية"

**السبب:** تاريخ السند لا يقع ضمن تواريخ الفترة المالية المختارة.

**الحل:** تحقق من `from_date` و `to_date` للفترة، أو قم بتعديل تاريخ السند.

### المشكلة: عدم ظهور أعمدة الصور في القائمة

**السبب:** لم يتم تشغيل هجرات أو تحديث الإضافة إلى 1.0.39.

**الحل:** قم بتشغيل `php artisan nano:up` ومسح الكاش.

### المشكلة: خطأ `Class 'UnifiedMorphClass' not found`

**السبب:** السمة غير موجودة أو لم يتم تضمينها.

**الحل:** تأكد من وجود ملف `traits/UnifiedMorphClass.php` في مساره الصحيح، وأضف `use Tss\Accounts\Traits\UnifiedMorphClass;` في النموذج.

### المشكلة: فشل رفع الملفات

**السبب:** صلاحيات مجلد التخزين أو إعدادات `System\Models\File`.

**الحل:** تأكد من أن `storage/app/uploads` قابل للكتابة، وتحقق من إعدادات `cms.upload_dir`.

---

## الخاتمة

إضافة `Tss.Accounts` هي نظام محاسبي متكامل وقابل للتوسع، يناسب المؤسسات الصغيرة والمتوسطة والمدارس والشركات. بفضل دعمها للعديد من أنواع السندات، والصلاحيات الدقيقة، والإعدادات المرنة، وقوالب الطباعة، والوسائط المتعددة، توفر حلاً شاملاً لإدارة العمليات المالية والمحاسبية ضمن بيئة NanoSoft APP. التحديث الأخير (1.0.39) يضيف دعماً قوياً للملفات والمرفقات، مما يعزز قدرة المستخدمين على توثيق كل عملية محاسبية بالمستندات اللازمة.

---

**المراجع**:

- [توثيق إضافة `Tss.Accounts`](./Docs-Tss-Accounts-ar.md)
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [فهرس دوال `TransactionsHelper`](./Docs-TransactionsHelper-Reference-Function-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
- [توثيق سمة `UnifiedMorphClass`](./Docs-UnifiedMorphClass-ar.md)
- [توثيق إضافة `Nano.AccountsApi`](./docs/AccountsApi/Docs-Nano-AccountsApi-ar.md)
- [توثيق Behavior `DynamicAddMediaIncludes`](./docs/AccountsApi/Docs-DynamicAddMediaIncludes-ar.md)
