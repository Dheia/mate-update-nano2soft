### **الميزات الإضافية (FormFieldsHelperTrait)**

تم إضافة مجموعة من الدوال المساعدة الجديدة عبر الـ Trait `FormFieldsHelperTrait` لتوسيع إمكانيات الكلاس `FormFieldsManager`. هذه الدوال تغطي مجالات متعددة مثل التحقق من القيم، توليد المخرجات، التصفية المتقدمة، والبحث. فيما يلي شرح مفصل لكل مجموعة مع أمثلة.

---

#### 1. **التحقق من القيم (Validation Simulation)**

##### `validateFieldValue(array $field, $value)`
محاكاة التحقق من قيمة معينة ضد قواعد التحقق الخاصة بحقل. تعيد مصفوفة تحتوي على `valid` (boolean) و `errors` (مصفوفة رسائل الخطأ).

**مثال:**
```php
$field = [
    'validation' => [
        ['rule' => 'required'],
        ['rule' => 'email', 'messages' => ['ar' => 'البريد الإلكتروني غير صحيح']]
    ]
];
$result = $manager->validateFieldValue($field, 'test@example.com');
// $result: ['valid' => true, 'errors' => []]

$result = $manager->validateFieldValue($field, 'invalid-email');
// $result: ['valid' => false, 'errors' => ['البريد الإلكتروني غير صحيح']]
```

##### `getValidationRulesFlattened(array $field)`
تحويل قواعد التحقق الخاصة بحقل إلى صيغة Laravel النصية (مثل `'required|email|min:5'`).

**مثال:**
```php
$field = [
    'validation' => [
        ['rule' => 'required'],
        ['rule' => 'min', 'value' => 5],
        ['rule' => 'email']
    ]
];
$rules = $manager->getValidationRulesFlattened($field);
// $rules: 'required|min:5|email'
```

---

#### 2. **توليد المخرجات**

##### `toHtml(array $field, $lang = null)`
توليد كود HTML تمهيدي بسيط للحقل. يأخذ في الاعتبار اللغة المحددة للنصوص (مثل `label`, `placeholder`).

**مثال:**
```php
$field = [
    'id' => 'username',
    'name' => 'username',
    'type' => 'text',
    'label' => ['ar' => 'اسم المستخدم', 'en' => 'Username'],
    'placeholder' => ['ar' => 'أدخل اسم المستخدم', 'en' => 'Enter username'],
    'required' => true
];
echo $manager->toHtml($field, 'ar');
// النتيجة: <div class="form-group"><label for="username">اسم المستخدم</label><input type="text" id="username" name="username" value="" placeholder="أدخل اسم المستخدم" required ></div>
```

##### `toJsonSchema(array $fields, array $options = [])`
تحويل مجموعة الحقول إلى هيكل JSON Schema مخصص، مع إمكانية إضافة عنوان ووصف عام.

**مثال:**
```php
$schema = $manager->toJsonSchema($fields, [
    'title' => 'نموذج التسجيل',
    'description' => 'حقول نموذج التسجيل'
]);
// النتيجة: مصفوفة تمثل JSON Schema
```

---

#### 3. **التصفية حسب خصائص محددة**

##### `getFieldsByDataSourceType(array $fields, $type)`
جلب الحقول التي تستخدم مصدر بيانات محدد (`api`, `database`, `static`, `function`).

**مثال:**
```php
$apiFields = $manager->getFieldsByDataSourceType($fields, 'api');
```

##### `getFieldsWithValidation(array $fields)`, `getFieldsWithoutValidation(array $fields)`
الحقول التي تحتوي/لا تحتوي على قواعد تحقق.

##### `getFieldsByReadOnly(array $fields, $readOnly = true)`, `getFieldsByDisabled`, `getFieldsByHidden`
الحقول حسب حالة `readOnly`, `disabled`, `hidden`.

##### `getFieldsByTabGroup(array $fields)`
تجميع الحقول في مصفوفة ترابطية حيث المفتاح هو اسم التبويب (tab).

**مثال:**
```php
$grouped = $manager->getFieldsByTabGroup($fields);
foreach ($grouped as $tab => $tabFields) {
    echo "تبويب: $tab";
    // عرض الحقول
}
```

##### `groupFieldsByType(array $fields)`
تجميع الحقول حسب النوع (`type`).

##### `getFieldsWithChangeHandler(array $fields)`
الحقول التي تحتوي على `changeHandler`.

##### `getFieldsWithOptions(array $fields)`, `getFieldsWithoutOptions(array $fields)`
جلب الحقول من نوع القوائم (select, dropdown, ...) التي تحتوي على خيارات (ثابتة أو ديناميكية) أو تلك التي لا تحتوي على خيارات.

---

#### 4. **البحث والاستعلام**

##### `hasFieldWithId(array $fields, $id)`, `hasFieldWithName(array $fields, $name)`
التحقق من وجود حقل بمعرف أو اسم معين.

##### `findByName(array $fields, $name)`
البحث عن حقل بواسطة `name`.

##### `findBy(array $fields, $key, $value)`
بحث عام عن حقل بمفتاح وقيمة معينة.

**مثال:**
```php
$field = $manager->findBy($fields, 'type', 'email');
```

##### `filterByCallback(array $fields, callable $callback)`
تصفية مخصصة باستخدام callback.

**مثال:**
```php
$filtered = $manager->filterByCallback($fields, function($field) {
    return strpos($field['id'], 'address') !== false;
});
```

---

#### 5. **تحويل البيانات ومعالجتها**

##### `extractFieldValues(array $fields, $key)`
استخراج قيم مفتاح معين من جميع الحقول (مثل استخراج كل `label`).

**مثال:**
```php
$labels = $manager->extractFieldValues($fields, 'label');
```

##### `mapFieldsToAssoc(array $fields, $key = 'id')`
تحويل مصفوفة الحقول إلى مصفوفة ترابطية باستخدام `id` أو `name` كمفتاح.

**مثال:**
```php
$assoc = $manager->mapFieldsToAssoc($fields, 'name');
$field = $assoc['email']; // الوصول المباشر
```

##### `sortFieldsBy(array $fields, $key, $direction = 'asc')`
ترتيب الحقول حسب أي مفتاح (مثل `id`, `name`, `sort_order`).

##### `sliceFields(array $fields, $offset, $length = null)`
استخراج شريحة من الحقول.

##### `paginateFields(array $fields, $perPage = 10, $page = 1)`
تقسيم الحقول إلى صفحات مع بيانات وصفية.

**مثال:**
```php
$page2 = $manager->paginateFields($fields, 5, 2);
// يحتوي على: data, total, per_page, current_page, last_page
```

##### `localizeField(array &$field, $lang)`
تحويل جميع الحقول متعددة اللغات في سجل حقل واحد إلى نص بسيط بلغة محددة (تعديل بالمرجع).

##### `localizeFields(array &$fields, $lang)`
تطبيق `localizeField` على جميع الحقول.

##### `stripMultilingualFields(array &$fields, $lang)`
مرادف لـ `localizeFields`، يزيل الهيكل متعدد اللغات.

##### `toArraySimple(array $fields, $lang = null)`
تحويل الحقول إلى صيغة مبسطة (بدون بنية اللغات) مناسبة للـ API.

**مثال:**
```php
$simple = $manager->toArraySimple($fields, 'en');
// جميع الحقول أصبحت تحتوي على نصوص إنجليزية مباشرة
```

---

### **ختاماً**

تمثل هذه الإضافات نقلة نوعية في مرونة الكلاس `FormFieldsManager`، حيث تتيح للمطورين التعامل مع حقول النماذج بطرق أكثر تقدماً: التحقق الآلي، توليد المخرجات، التصفية الذكية، والتحويلات المتنوعة. يُنصح باستخدام هذه الدوال لتقليل التكرار وزيادة قابلية الصيانة في المشاريع التي تعتمد heavily على حقول النماذج.