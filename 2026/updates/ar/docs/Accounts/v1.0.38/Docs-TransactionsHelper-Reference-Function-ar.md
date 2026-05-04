# فهرس شامل لدوال `TransactionsHelper` و `TransactionsMultiHelper`

يقدم هذا المستند فهرساً كاملاً لجميع الدوال المتاحة في الكلاس `TransactionsHelper` والسمة `TransactionsMultiHelper`، منظمة حسب مجال استخدامها وطبيعتها.

> **ملاحظة:** جميع الدوال ثابتة (`static`) ويمكن استدعاؤها مباشرة عبر `TransactionsHelper::methodName()`.

---

## 1. دوال إنشاء القيود المحاسبية

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `createSingle` | `public` | إنشاء قيد محاسبي بسيط ذو طرفين (مدين ودائن). تعتبر العمود الفقري للعمليات المالية الأساسية. تستخدم داخلياً من معظم دوال المدفوعات والعمليات التجارية. |
| `createEntry` | `public` | دالة موحدة تختار تلقائياً بين `createSingle` (لطرفين) و `createJournalEntry` (لأكثر من طرفين). يمكن فرض استخدام `createJournalEntry` عبر الخيار `forceJournal => true`. |
| `createJournalEntry` | `public` | إنشاء قيد محاسبي متعدد الأطراف مع تحقق شامل، معالجة عملات، فحص أرصدة، وقواعد ديناميكية لفحص أنواع الحسابات، وفروق الصرف. |
| `testJournalEntry` | `public` | محاكاة كاملة لإنشاء قيد متعدد الأطراف دون حفظ في قاعدة البيانات، مع تحليل مفصل للنتائج (مجاميع، عملات، مخالفات، تحذيرات). |

---

## 2. دوال العمليات المالية المتخصصة (من `TransactionsHelper`)

### 2.1 إدارة أرصدة الموصلين والعملاء

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `createMonyDeliverys` | `public` | إضافة رصيد افتتاحي لموصل (Delivery) من الشركة، مع منع التكرار. |
| `createMonyDeliverysFromSyndical` | `public` | شحن رصيد لموصل من قبل وكيل (Syndical/Department)، مع فحص رصيد الوكيل. |
| `createMonyDeliverysFromCompanys` | `public` | شحن رصيد لموصل من قبل الشركة الرسمية (يتطلب صلاحيات مستخدم خلفي). |
| `createDeductingMonyDeliverysFromCompanys` | `public` | خصم مبلغ من حساب موصل (مثلاً عمولة التطبيق)، مع دعم تخصيص الملاحظات والأشخاص. |

### 2.2 دوال العمليات المرتبطة بالطلبات (Orders)

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `createBrokerageFromOrders` | `public` | خصم عمولة التوصيل من الموصل بناءً على الطلب. |
| `createPaymentFromOrders` | `public` | تسجيل قيد دفع قيمة الطلب (من العميل إلى حساب الشركة/الموصل). |
| `createBillShopFromOrders` | `public` | إثبات قيمة مشتريات المتجر (أو الموصل في حالة الدفع أونلاين). |
| `createBrokerageShopFromOrders` | `public` | خصم عمولة التطبيق من المتجر. |
| `createShippingTotalFromOrders` | `public` | تسجيل قيد كلفة التوصيل للموصل (في حالة الدفع أونلاين). |
| `createTipAmountFromOrders` | `public` | تسجيل قيد الإكرامية (بقشيش) للموصل. |
| `createPaymentFromOrdersOld` | `public` | (قديمة) نسخة سابقة من دالة تسجيل قيد دفع قيمة الطلب. |
| `createPaymentMonyDeliverysFromCompanys` | `public` | (قديمة) محاولة سابقة لتسجيل قيد دفع متكامل للطلب. |

### 2.3 دوال المدفوعات والتحقق

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `addPayment` | `public` | إضافة دفعة مالية من طرف ثالث (مثل البنك) إلى حساب معين (مثلاً حساب طالب)، مع تحكم كامل بالأطراف الدائنة والمدينة عبر الإعدادات. |
| `checkAllowPay` | `public` | التحقق من كفاية رصيد حساب معين (يجب أن يكون دائناً) لتنفيذ عملية شراء أو سداد. |
| `checkTransactions` | `public` | البحث عن قيد (رأس حركة) محدد والتأكد من وجوده. تستخدم للتحقق قبل التحديث أو الإلغاء. |

---

## 3. دوال الاستعلام وجلب البيانات

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `getTransactionHeader` | `public` | استعلام متقدم عن رؤوس الحركات (`TransactionHeader`) مع فلترة شاملة حسب القسم، الفترة، نوع الحركة، الشخص، الحقول المتقدمة، ونطاقات التاريخ. |
| `seedDocsNumber` | `public` | ترقيم السجلات القديمة في جدول `tss_accounts_transaction_headers` التي لا تحتوي على `docs_number`. |

---

## 4. دوال معالجة فروق العملات (من `TransactionsMultiHelper`)

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `checkExchangeDifferences` | `public` | فحص قيد محاسبي موجود لتحديد فروق العملات بناءً على تاريخ الترحيل، مع خيارات لإنشاء قيد فروق، إطلاق حدث، أو إرجاع المعلومات فقط. |
| `createExchangeDifferenceEntry` | `public` | إنشاء قيد فروق صرف ناتج عن تغير سعر الصرف بين تاريخ القيد وتاريخ الترحيل، مع ربطه بالقيد الأصلي. |
| `handleExchangeDifferences` | `protected` | معالجة داخلية لفروق الصرف تستدعى تلقائياً بعد حفظ القيد إذا كان `handle_exchange_differences = true`. |

---

## 5. دوال التحقق والفحص الداخلي (من `TransactionsMultiHelper`)

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `validateJournalEntry` | `public` | التحقق من صحة بيانات قيد متعدد الأطراف دون إنشائه، تعيد التفاصيل بعد المعالجة والتحقق من التوازن والقيود. |
| `validateJournalDetails` | `protected` | تحقق شامل من جميع تفاصيل القيد (كل عنصر `details`) وإعادة مصفوفة التفاصيل النظيفة. |
| `validateSingleDetail` | `protected` | تحقق من صحة تفصيل واحد (طرف محاسبي) بما فيه الحساب، الشخص، الملاحظات، العملات، والحقول الإضافية. |
| `validateSideField` | `protected` | تحقق من حقل معين (مثل `account` أو `notes`) في جانب معين (مدين/دائن) بناءً على قواعد `allowed`/`required`/`default`. |
| `validateMultiplicity` | `protected` | التحقق من قيود عدد الأطراف (الحد الأقصى، تعدد المدين/الدائن). |
| `validateAmountLimits` | `protected` | التحقق من حدود المبالغ الإجمالية والخاصة بكل جانب. |
| `validateBalance` | `protected` | تفعيل فحص الرصيد لكل تفصيل حسب الإعدادات (مدين/دائن). |
| `checkDetailBalance` | `protected` | فحص رصيد تفصيل واحد باستخدام `checkAccountBalance` أو `callback` مخصص. |
| `validatePeriod` | `protected` | التحقق من صلاحية الفترة المحاسبية (التاريخ ضمن النطاق، الفترة مفتوحة). |
| `validatePaperId` | `protected` | التحقق من عدم تكرار `paper_id` إذا كان مطلوباً (حسب نطاق التكرار). |
| `validatePaperIdRequired` | `protected` | التحقق من إلزامية `paper_id`. |
| `validateDateAtRequired` | `protected` | التحقق من إلزامية `date_at`. |

---

## 6. دوال تجهيز البيانات والبناء (من `TransactionsMultiHelper`)

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `mergeDefaultCompanyValues` | `protected` | دمج جميع الخيارات المُمررة مع القيم الافتراضية الشاملة (الشركة، الفترة، العملة، الحقول الإضافية...). |
| `prepareHeaderOptions` | `protected` | تجهيز مصفوفة خيارات رأس القيد (`TransactionHeader`) لتمريرها إلى `AccountHelper::getObjHeader`. |
| `prepareDetailOptions` | `protected` | تجهيز مصفوفة خيارات تفصيل القيد (`TransactionsDetail`) لتمريرها إلى `AccountHelper::getObjDetail`. |
| `processHeaderPerson` | `protected` | معالجة كائن الشخص على مستوى الرأس وتحويله إلى `id` و `type`. |
| `preparePaperId` | `protected` | تجهيز `paper_id` (توليد تلقائي إذا كان `paper_id_auto = true`، إلغاء القيمة إذا كان `paper_id_allowed = false`). |
| `generatePaperId` | `protected` | توليد `paper_id` بناءً على أقصى قيمة في النطاق المحدد. |
| `getMaxPaperId` | `protected` | الحصول على أكبر `paper_id` مسجل حسب الشروط المحددة. |
| `prepareDateAt` | `protected` | تجهيز `date_at` (تعيين التاريخ الحالي إذا كان `is_custome_date = false`). |

---

## 7. دوال معالجة العملات والحسابات (من `TransactionsMultiHelper`)

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `processCurrency` | `protected` | معالجة العملة وسعر الصرف لكل تفصيل، مع دعم التحويل التلقائي (`auto_convert_currency`). |
| `calculateTotals` | `protected` | حساب مجاميع المدين والدائن الأساسية والأجنبية، وتحديد ما إذا كانت العملات مختلطة. |
| `applyAccountRules` | `protected` | تطبيق مجموعة قواعد ديناميكية على حساب للتحقق من نوعه أو خصائصه (مثل `modul_type = 'Boxe'`). |
| `compareAttribute` | `protected` | مقارنة قيمة خاصية بحساب مع عامل تشغيل (`=`, `LIKE`, `IN`, `REGEXP`...). |
| `checkAccountBalance` | `protected` | فحص رصيد حساب معين باستخدام `AccountHelper::getBalances`. |

---

## 8. دوال مساعدة عامة (من `TransactionsMultiHelper`)

| الدالة | النوع | الوصف |
| :--- | :--- | :--- |
| `handleException` | `protected` | معالجة موحدة للاستثناءات وإرجاع مصفوفة النتيجة مع تفاصيل التصحيح في وضع `debug`. |
| `fireHook` | `protected` | تنفيذ دوال `beforeValidate`/`afterValidate` المخصصة وإطلاق أحداث النظام. |
| `isUseDefaultCompany` | `public` | التحقق مما إذا كان إعداد `use_default_company` مفعلاً. |
| `getCheckCompanysId` | `public` | الحصول على معرف الشركة المناسب (مع مراعاة إعدادات `use_default_company`). |
| `getCheckDepartmentsId` | `public` | الحصول على معرف القسم المناسب. |

---

## 9. ملخص سريع للاستخدامات الشائعة

| السيناريو | الدوال المستخدمة |
| :--- | :--- |
| إنشاء قيد بسيط (طرفين) | `createSingle` أو `createEntry` |
| إنشاء قيد معقد (أكثر من طرفين) | `createEntry` أو `createJournalEntry` |
| اختبار قيد قبل حفظه | `testJournalEntry` |
| إضافة دفعة طالب عبر البنك | `addPayment` |
| شحن رصيد موصل | `createMonyDeliverys` / `createMonyDeliverysFromCompanys` |
| معالجة دورة حياة طلب (دفع، توصيل، عمولة) | `createPaymentFromOrders`, `createBillShopFromOrders`, `createBrokerageShopFromOrders`... |
| فحص فروق عملات قيد | `checkExchangeDifferences` |
| إنشاء قيد فروق صرف | `createExchangeDifferenceEntry` |
| البحث عن قيود بفلاتر متقدمة | `getTransactionHeader` |
| التحقق من رصيد قبل عملية شراء | `checkAllowPay` |

---

هذا الفهرس يغطي جميع الدوال المتاحة في الكلاس والسمة، مع وصف دقيق وواضح لكل منها. يمكن الرجوع إلى التوثيق التفصيلي لكل دالة للحصول على معلومات أعمق حول الخيارات والاستخدامات المتقدمة.


## التوثيق الإضافي

**الوثائق المرجعية**:
- [توثيق كلاس `AccountHelper`](./Docs-AccountHelper-ar.md)
- [توثيق كلاس `PersonHelper`](./Docs-PersonHelper-ar.md)
- [توثيق كلاس `TransactionsHelper`](./Docs-TransactionsHelper-ar.md)
- [توثيق متقدم لـ `TransactionsHelper`](./Docs-TransactionsHelper-Advanced-ar.md)
- [توثيق شامل لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-ar.md)
- [أمثلة عملية متقدمة لدالة `createJournalEntry`](./Docs-TransactionsHelper-createJournalEntry-Example-ar.md)
- [إعدادات `createJournalEntry` (شاملة)](./Docs-createJournalEntry-config-ar.md)
- [إعدادات `createJournalEntry` (متوافقة)](./Docs-createJournalEntry-config-v1-ar.md)
- [توثيق دوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-ar.md)
- [أمثلة عملية لدوال فروق العملات](./Docs-TransactionsHelper-ExchangeDifferences-Example-ar.md)
