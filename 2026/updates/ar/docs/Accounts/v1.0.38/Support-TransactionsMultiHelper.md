## ملخص التحديثات على `TransactionsHelper` و `TransactionsMultiHelper`

### 1. إضافة دالة موحدة لإنشاء القيود المحاسبية
- **`createEntry`**: دالة موحدة تختار تلقائياً بين:
  - `createSingle` (للقيود ذات الطرفين).
  - `createJournalEntry` (للقيود متعددة الأطراف، أكثر من طرفين).
  - يمكن فرض استخدام `createJournalEntry` عبر الخيار `forceJournal => true`.

### 2. دالة إنشاء قيد محاسبي متعدد الأطراف (`createJournalEntry`)
- إنشاء قيود بعدد غير محدود من الأطراف (مدين/دائن).
- الحفاظ على نفس هيكل النتيجة (`result array`) المستخدم في `createSingle` مع إضافة `input_data` و `process_data`.
- استخدام `Db::transaction` لضمان سلامة البيانات.

### 3. تحسينات التحقق من صحة البيانات
- التحقق من وجود الحساب (`Account::where('code')`) ونشاطه (`is_active`).
- منع الترحيل على حسابات رئيسية (اختياري عبر الإعدادات).
- التحقق من توازن القيد (مجموع المدين = مجموع الدائن مع تسامح 0.001).
- التحقق من صحة الفترة المحاسبية (`Period::isValidDate`) وكونها مفتوحة (`Period::isOpen`).

### 4. قيود على عدد الأطراف لكل جانب
- `max_debit_entries`, `max_credit_entries`: تحديد الحد الأقصى لعدد الأطراف.
- `allow_multiple_debit`, `allow_multiple_credit`: السماح أو منع تعدد الأطراف في جانب معين.

### 5. التحكم في استخدام الشخص (`person`)
- **لكل جانب**:
  - `debit_person_allowed` / `credit_person_allowed`: السماح بربط شخص.
  - `debit_person_required` / `credit_person_required`: إلزامية وجود شخص.
  - `debit_person_allowed_types` / `credit_person_allowed_types`: تحديد أنواع الأشخاص المسموح بها (مصفوفة من أسماء الكلاسات).
- معالجة كائن `person` إذا تم تمريره مباشرةً.

### 6. التحكم في الحساب (`account`)
- **لكل جانب**:
  - `debit_account_allowed` / `credit_account_allowed`: السماح بتحديد حساب.
  - `debit_account_required` / `credit_account_required`: إلزامية تحديد حساب.
  - `debit_default_account` / `credit_default_account`: حساب افتراضي (كود) يُستخدم إذا لم يُحدد.
- التحقق من وجود الحساب ونشاطه بعد تطبيق القواعد.

### 7. التحكم في الملاحظات (`notes`)
- **لكل جانب**:
  - `debit_notes_allowed` / `credit_notes_allowed`: السماح بإدخال ملاحظة.
  - `debit_notes_required` / `credit_notes_required`: إلزامية وجود ملاحظة.
  - `debit_default_notes` / `credit_default_notes`: ملاحظة افتراضية.

### 8. حدود المبالغ
- **إجمالي القيد**:
  - `min_total_amount`, `max_total_amount`, `check_min_total`, `check_max_total`.
- **لكل جانب**:
  - `min_debit_amount`, `max_debit_amount`, `check_debit_min`, `check_debit_max`.
  - `min_credit_amount`, `max_credit_amount`, `check_credit_min`, `check_credit_max`.

### 9. فحص الرصيد (`balance check`)
- **لكل جانب**:
  - `debit_check_balance` / `credit_check_balance`: تفعيل الفحص.
  - `debit_balance_check_callback` / `credit_balance_check_callback`: دالة مخصصة لتجاوز الفحص الافتراضي (تستقبل: `$account`, `$amount`, `$balance`, `$personId`, `$personType`, `$context`).
- الفحص الافتراضي يستخدم `AccountHelper::getBalances` مع مراعاة `person` إذا وُجد.

### 10. فحص نوع الحساب بقواعد ديناميكية
- **لكل جانب**:
  - `debit_check_account_types` / `credit_check_account_types`: تفعيل الفحص.
  - `debit_rules_account_types` / `credit_rules_account_types`: مصفوفة قواعد.
- **صيغة القواعد**:
  ```php
  [
      'operator' => 'AND|OR', // اختياري، افتراضي AND
      'rules' => [
          ['field' => 'account_number', 'operator' => 'LIKE', 'value' => '1253%'],
          ['field' => 'modul_type', 'operator' => 'IN', 'value' => ['Boxe', 'Bank']],
          // ...
      ]
  ]
  ```
- **الحقول المدعومة**: جميع أعمدة جدول `accounts` (مثل `reports`, `final_account_id`, `modul_type`, `ref_type`, `person_type`, `account_number`, `level`, `parent_id`, ...).
- **المشغلات المدعومة**: `=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`, `NOT LIKE`, `IN`, `NOT IN`, `REGEXP`.
- يتم تطبيق القواعد بعد التحقق من وجود الحساب وقبل فحص الرصيد.

### 11. التحكم في الحقول الإضافية (`extend_id`, `modul_type`, `ref_type`, `ref_key_name`, `ref_key_values`, `batch_type`, `batch_id`)
- **على مستوى الرأس**:
  - `header_extend_id_allowed`, `header_extend_id_required`, `header_extend_id_default` (ونظائرها لباقي الحقول).
- **على مستوى تفاصيل كل جانب**:
  - `debit_extend_id_allowed`, `debit_extend_id_required`, `debit_extend_id_default` ... (للمدين).
  - `credit_extend_id_allowed`, `credit_extend_id_required`, `credit_extend_id_default` ... (للدائن).
- يتم تطبيق القواعد (allowed/required/default) بشكل مستقل لكل حقل.

### 12. إضافة أحداث (Events)
- `tss.accounts.beforeJournalEntry`: يُطلق قبل بدء المعاملة، ويستقبل `$options` و `$processedDetails`.
- `tss.accounts.afterJournalEntry`: يُطلق بعد نجاح المعاملة، ويستقبل `$model`, `$options`, `$processedDetails`.

### 13. تحسين التعامل مع الأخطاء والتقارير
- إرجاع `input_data` و `process_data` في نتيجة الدالة لتسهيل التتبع.
- تسجيل الأخطاء في ملف الـ log باستخدام `Log::error`.
- رسائل خطأ مفصلة وقابلة للترجمة مع تحديد رقم التفصيل والجانب.
- دعم `debug` في وضع التطوير.

### 14. تحسين بنية الكود وإعادة الاستخدام
- دوال مساعدة منفصلة:
  - `validateJournalDetails`
  - `validateSingleDetail`
  - `validateSideField` (للحقول البسيطة)
  - `applyAccountRules` (تطبيق قواعد نوع الحساب)
  - `compareAttribute` (مقارنة القيم مع المشغلات)
  - `checkAccountBalance` (فحص الرصيد)
- استخدام `array_merge` لدمج القيم الافتراضية مع الخيارات الممررة.

### 15. تحديث `BaseDocumentController` للاستفادة من الميزات الجديدة
- استخدام `createEntry` بدلاً من `createSingle`.
- تمرير `details` مباشرةً (مع دعم التحويل التلقائي من الصيغة القديمة `debit_accounts_id`/`credit_accounts_id`).
- دمج إعدادات النوع (مثل `bonds_days`) مع الإعدادات الافتراضية العامة (`journal_entry_defaults`).

### 16. إعدادات شاملة قابلة للتكوين
- توفير مصفوفة `journal_entry_defaults` في ملف `config.php` تحتوي على جميع الإعدادات السابقة بقيمها الافتراضية.
- يمكن تخصيصها لكل نوع مستند (مثلاً في `bonds_days`, `catch_receipts`) عبر ملف الإعدادات الخاص بكل نوع.

---

بهذا أصبح النظام قادراً على إنشاء قيود محاسبية معقدة مع تحكم كامل في جميع الجوانب (الصلاحيات، الحسابات، الأشخاص، المبالغ، الرصيد، نوع الحساب، الحقول الإضافية) مع مرونة عالية وإمكانية التوسع.