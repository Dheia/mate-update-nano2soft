## 2026-03-20 – 2026-05-05

**تحديث إضافة `Nano.AccountsApi` – الإصدار 1.1.0**

### إعادة هيكلة شاملة وإضافة أنواع المستندات المحاسبية المتكاملة

---

### ملخص التحديثات

في الإصدار **1.1.0**، تم تنفيذ قفزة معمارية في إضافة `Nano.AccountsApi`، حيث تم استخراج فئة أساسية مجردة `BaseDocumentController` لإدارة العمليات المشتركة بين مختلف أنواع المستندات المحاسبية. كما تم إضافة دعم كامل لخمسة أنواع جديدة من المستندات المالية (قيود يومية، سندات صرف، إشعارات مدين، إشعارات دائن، تحويلات) مع نماذجها ومحولاتها وإعداداتها الخاصة، بالإضافة إلى نسخة محسّنة من سندات القبض (`CatchReceiptsV2`). وقد تم توسيع ملف الإعدادات والمسارات لدعم هذه الأنواع الجديدة، مع تحسينات متعددة على المتحكمات والمحولات القائمة.

---

### أهداف الإصدار

- **توحيد منطق العمليات** عبر `BaseDocumentController` للتخلص من تكرار الكود وتسهيل إضافة أنواع مستندات جديدة.
- **إضافة أنواع المستندات المحاسبية الشائعة** لتغطية أغلب العمليات المالية اليومية عبر API.
- **توفير إعدادات مرنة وصلاحيات منفصلة** لكل نوع مستند، مدعومة بمتغيرات البيئة (env) لتسهيل التخصيص.
- **تحسين تجربة المطورين** من خلال واجهة برمجة موحدة لعمليات CRUD والتحقق من الصلاحيات.
- **الحفاظ على التوافق العكسي** مع المتحكمات والمحولات القديمة التي لا تزال تعمل كما هي.

---

### الميزات الجديدة والتحسينات

#### 1. فئة أساسية موحدة `BaseDocumentController`

تم إنشاء فئة أساسية مجردة (abstract class) باسم `BaseDocumentController` ترث من `ApiController` وتوفر الوظائف المشتركة التالية:

- **إدارة المستخدم الحالي** والتحقق من صلاحياته (`checkAddPermission`، `checkListPermission`، `checkViewPermission`، `checkUpdatePermission`، `checkDeletePermission`).
- **تجهيز البيانات** عبر دوال مثل `prepareCreateData`، `prepareListOptions`، `validateInput`.
- **تنفيذ العمليات** عبر دوال `performCreate`، `performList`، `performShow`، `performUpdate`، `performDelete` التي تعيد مصفوفة نتائج موحدة.
- **تطبيق المحولات** تلقائياً على مخرجات API.

أصبح بإمكان أي متحكم جديد لمستند مالي أن يرث هذه الفئة ويحدد فقط `$modelClass`، `$transformerClass`، `$configKey`، و`$typeHeader` ليحصل على كامل وظائف CRUD.

#### 2. إضافة أنواع المستندات الجديدة

تمت إضافة خمسة متحكمات جديدة تعتمد على `BaseDocumentController`، لكل منها نموذج (Model) ومحول (Transformer) وإعدادات خاصة:

| المتحكم | النموذج | المحول | النوع (typeHeader) | الوصف |
|---------|--------|--------|-------------------|-------|
| `BondsDays` | `BondsDay` | `BondsDayTransformer` | `BondsDay::class` | قيود يومية متعددة الأطراف مع تحقق من توازن المدين والدائن |
| `PayReceipts` | `PayReceipt` | `PayReceiptTransformer` | `PayReceipt::class` | سندات صرف مع دفع (شيك، نقد) |
| `DebitNotes` | `DebitNote` | `DebitNoteTransformer` | `DebitNote::class` | إشعارات مدين |
| `CreditNotes` | `CreditNote` | `CreditNoteTransformer` | `CreditNote::class` | إشعارات دائن |
| `Transfers` | `Transfer` | `TransferTransformer` | `Transfer::class` | تحويلات بين الحسابات |

كما تم إضافة نسخة محسّنة من سندات القبض:

- `CatchReceiptsV2` (تعتمد على `BaseDocumentController` ونموذج `CatchReceipt`)، مع احتفاظ المتحكم القديم `CatchReceipts` دون تغيير.

**قواعد التحقق الخاصة بكل نوع:**

- **BondsDays:** التحقق من وجود تفاصيل `details` بعدد أطراف ≥ 2، وتوازن المدين والدائن، ومنع المبالغ الصفرية.
- **CatchReceiptsV2 / PayReceipts:** دعم حقول طريقة الدفع (`payment_method`)، رقم الشيك (`cheque_number`)، تاريخه (`cheque_date`)، اسم البنك (`bank_name`)، والمستفيد (`beneficiary`).
- **DebitNotes / CreditNotes:** حقول رقم الفاتورة (`invoice_number`) وسبب الإشعار (`reason`).
- **Transfers:** التحقق من أن الحسابين مختلفان، وحقول رقم المرجع (`reference_number`) والسبب.

#### 3. نظام المحولات (Transformers) الجديد

- تم إنشاء محول `BondsDayTransformer` يرث من `TransactionHeaderTransformer` ويضيف مجموع المدين/الدائن لكل قيد.
- تم إنشاء محولات `CatchReceiptTransformer`، `PayReceiptTransformer`، `DebitNoteTransformer`، `CreditNoteTransformer`، `TransferTransformer` كلها ترث من `TransactionHeaderTransformer` وتضيف حقولها الخاصة (مثل `payment_method`، `cheque_number`...).
- تم تحديث `TransactionHeaderTransformer` ليقبل جميع أنواع النماذج الجديدة عبر union types في توقيع دالة `data()`، مما يسمح باستخدامه كمحول أساسي لأي رأس حركة مالية.

#### 4. توسيع الإعدادات والصلاحيات

تمت إضافة أقسام جديدة في ملف `config.php` لكل نوع مستند، مع دعم متغيرات البيئة (env) للتحكم في:

- **إعدادات التشغيل:** `order_by`، `order_dir`، `transactions_type`، `process_type`، `patterns_id`، `notes`.
- **الصلاحيات المنفصلة:** `is_allow_add`، `is_allow_add_backend`، `is_check_add_permission`، `is_allow_add_frontend` (ومثيلاتها للعرض والتحديث والحذف).
- **إعدادات المحول:** `is_stop_to_array` لكل نوع.

الأقسام المضافة: `bonds_days`، `catch_receipts_v2`، `pay_receipts`، `debit_notes`، `credit_notes`، `transfers`.

#### 5. توسيع مسارات API

أُضيفت مسارات جديدة ضمن مجموعة `oauth-users` في `routes.php`:

```
// قيود يومية
POST  bonds/add → BondsDays@create
POST  bonds/test → BondsDays@test
GET   bonds → BondsDays@index
GET   bonds/{id} → BondsDays@show

// سندات صرف
POST  payreceipts/add → PayReceipts@create
GET   payreceipts → PayReceipts@index
GET   payreceipts/{id} → PayReceipts@show

// إشعارات مدين
POST  debitnotes/add → DebitNotes@create
GET   debitnotes → DebitNotes@index
GET   debitnotes/{id} → DebitNotes@show

// إشعارات دائن
POST  creditnotes/add → CreditNotes@create
GET   creditnotes → CreditNotes@index
GET   creditnotes/{id} → CreditNotes@show

// تحويلات
POST  transfers/add → Transfers@create
GET   transfers → Transfers@index
GET   transfers/{id} → Transfers@show
```

ملاحظة: تم إضافة المسارات كتعليقات احتياطية لعمليات التحديث والحذف (`PUT` و `DELETE`) لتفعيلها مستقبلاً.

#### 6. تحسينات إضافية

- **دالة `show` في `Accounts.php`:** أصبحت تدعم البحث عن الحساب باستخدام `code` بالإضافة إلى `id`، مما يوفر مرونة أكبر للمطورين.
- **دالة `test` في `BondsDays`:** تسمح باختبار القيد اليومي عبر `TransactionsHelper::testJournalEntry` قبل الحفظ، مع صلاحيات خاصة للمستخدمين الخلفيين.
- **`TransactionHeaderTransformer`:** تم توسيعه ليشمل جميع أنواع المستندات المالية، مع الحفاظ على نفس الوظائف الأساسية.

---

### أمثلة على الاستخدام

#### إنشاء قيد يومي متعدد الأطراف

```bash
curl -X POST "https://yourdomain.com/api/v1/accounts/bonds/add" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "date_at": "2026-04-20",
    "notes": "قيد تسوية",
    "details": [
      {"accounts_id": "2-2-1253010001", "debit": 1500},
      {"accounts_id": "2-2-1231010001", "credit": 1000},
      {"accounts_id": "2-2-4111010001", "credit": 500}
    ]
  }'
```

#### جلب قائمة سندات القبض (الإصدار الجديد)

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/catchreceipts" \
  -H "Authorization: Bearer <token>"
```

#### عرض حساب باستخدام الكود

```bash
curl -X GET "https://yourdomain.com/api/v1/accounts/accounts/2-2-1231010001" \
  -H "Authorization: Bearer <token>"
```

---

### الفوائد والقيمة المضافة

- **تسريع التطوير:** يمكن إضافة أي نوع مستند مالي جديد بكتابة بضعة أسطر فقط عبر وراثة `BaseDocumentController`.
- **تقليل الأخطاء:** توحيد منطق التحقق من الصلاحيات وإعداد البيانات يقلل من احتمالية الأخطاء البشرية.
- **مرونة في التخصيص:** كل نوع مستند له إعداداته وصلاحياته المنفصلة، مما يتيح تكييف الإضافة حسب متطلبات كل مشروع.
- **تجربة مستخدم محسنة:** دعم API كامل لجميع العمليات المالية الأساسية، مما يسمح ببناء واجهات أمامية ثرية.
- **حماية معززة:** استخدام نفس نظام الصلاحيات المعمول به في الإضافة الأساسية (`Tss.Accounts`) عبر API.
- **استعداد للمستقبل:** الهيكلية الجديدة تسمح بسهولة إضافة عمليات التحديث والحذف (جاهزة تقنياً، بانتظار تفعيلها).

---

### متطلبات الترقية

1. **تحديث الكود:**
   - استبدال الملفات الأساسية بالإصدارات الجديدة:
     - `APIControllers/BaseDocumentController.php`
     - `APIControllers/BondsDays.php`
     - `APIControllers/PayReceipts.php`
     - `APIControllers/DebitNotes.php`
     - `APIControllers/CreditNotes.php`
     - `APIControllers/Transfers.php`
     - `APIControllers/CatchReceiptsV2.php`
     - `Transformers/BondsDayTransformer.php`
     - `Transformers/CatchReceiptTransformer.php`
     - `Transformers/PayReceiptTransformer.php`
     - `Transformers/DebitNoteTransformer.php`
     - `Transformers/CreditNoteTransformer.php`
     - `Transformers/TransferTransformer.php`
   - تحديث الملفات التالية:
     - `config.php`
     - `routes.php`
     - `TransactionHeaderTransformer.php`
     - `Accounts.php` (تحسين `show`)
2. **لا توجد هجرات قاعدة بيانات جديدة.**
3. **صلاحيات API:**
   - النقاط الجديدة محمية بـ `oauth-users`، لذا يجب أن يمتلك المستخدم توكن صالح، وأن تتوافق صلاحياته مع الإعدادات المحددة لكل نوع.
4. **اختبار الاتصال:**
   - تأكد من أن المتغيرات البيئية (env) للإعدادات الجديدة مضبوطة بشكل صحيح (خاصة `is_allow_add_backend` ونوع الحركة `transactions_type`).
   - جرب إنشاء قيد يومي بسيط باستخدام `bonds/add` وتحقق من الاستجابة.
5. **مسح الكاش:**
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

يمثل الإصدار **1.1.0** نقلة نوعية في إضافة `Nano.AccountsApi`، حيث انتقلت من مجرد مجموعة من المتحكمات الفردية إلى إطار عمل مصغر لإدارة المستندات المالية عبر API. بفضل `BaseDocumentController`، أصبحت إضافة أنواع جديدة عملية سريعة وآمنة، مما يفتح الباب لتوسعات مستقبلية دون تعقيد. التغطية الشاملة لأنواع القيود والسندات والإشعارات والتحويلات تجعل الإضافة مناسبة لمجموعة واسعة من التطبيقات المحاسبية، مع الحفاظ على التوافق مع الإصدارات السابقة.


### التوثيق الإضافي

**الوثائق المرجعية**:
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

