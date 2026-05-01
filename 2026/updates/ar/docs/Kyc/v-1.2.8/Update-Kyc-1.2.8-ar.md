## 2026-04-30

**تحديثات إضافة `Nano3.Kyc` – الإصدار 1.2.8**

### ملخص التحديثات

يأتي الإصدار 1.2.8 كإصلاح شامل لآلية دمج الحقول وتوليد قواعد التحقق في كلاس `DocumentType`. رغم النجاح الذي حققه الإصدار 1.2.7 في توحيد بنية الحقول مع `FormFieldsManager`، إلا أن الاختبارات العملية كشفت عن مشكلات في طريقة دمج الحقول الافتراضية مع التجاوزات (overrides) ومع إعدادات ملف `document_types.php`. أدت هذه المشكلات إلى ظهور حقول غير مرغوب فيها في بعض أنواع الوثائق (مثل ظهور حقول الشركة في جواز السفر)، وتكرار بعض الحقول، ووجود قواعد تحقق خاطئة أو مشوهة.

يركز هذا الإصدار على **إصلاح جذري لخوارزميات الدمج** لضمان:

- حذف الحقول غير المرغوب فيها نهائياً عند تحديدها بقيمة `null` في التجاوزات.
- دمج قواعد التحقق (`validation`) بذكاء مع إمكانية استبدال القواعد المتكررة بدلاً من تراكمها.
- استبدال الحقول بالكامل عند تعريفها في ملف الإعدادات (`document_types.php`) بدلاً من دمجها مع الحقول الافتراضية.
- توليد قواعد تحقق (`rules`) صحيحة ومتوافقة تماماً مع صيغة Laravel، جاهزة للاستخدام المباشر في `Validator::make`.

كل ذلك مع الحفاظ على التوافق التام مع الإصدارات السابقة وعدم كسر أي واجهات برمجية (API).

---

### الإصدار 1.2.8 – إصلاح دمج الحقول وتوليد قواعد التحقق

#### أهداف الإصدار

- **إصلاح دالة `mergeFieldsWithOverrides`** لدعم حذف الحقول (`null`) ودمج قواعد التحقق بشكل صحيح.
- **إضافة دالة `mergeValidation`** لاستبدال القواعد المكررة بدلاً من تراكمها، مما يحل مشكلة القيم المشوهة في `validation`.
- **إصلاح دالة `loadConfig`** لجعل ملف الإعدادات هو المصدر الوحيد للحقول عند تعريفها، مما يمنع تسرب الحقول الافتراضية إلى الأنواع المخصصة.
- **إزالة التكرارات** في الحقول (مثل `religion` و `passport_type`).
- **تصحيح قواعد التحقق** في جميع أنواع الوثائق لتعكس القيم الصحيحة (مثل `max:50` بدلاً من `max:100` أو قيم غير مقصودة).

#### التحسينات والإصلاحات

##### 1. إعادة كتابة دالة `mergeFieldsWithOverrides`

تم استبدال التنفيذ السابق الذي كان يعتمد على `array_merge` و `unset` (الذي لم يكن يحذف الحقول بشكل صحيح في بعض الحالات) بتنفيذ جديد يعتمد على **الفهرسة بالمفتاح `id`**:

```php
protected static function mergeFieldsWithOverrides(array $baseFields, array $extraFields = [], array $overrides = []): array
{
    // تحويل baseFields إلى associative بمفتاح id
    $fieldsAssoc = [];
    foreach ($baseFields as $field) {
        $id = $field['id'] ?? null;
        if ($id) {
            $fieldsAssoc[$id] = $field;
        }
    }

    // extraFields تستبدل الحقل بالكامل إذا كان id موجوداً
    foreach ($extraFields as $extraField) {
        $id = $extraField['id'] ?? null;
        if ($id) {
            $fieldsAssoc[$id] = $extraField;
        } else {
            $fieldsAssoc[] = $extraField;
        }
    }

    // overrides تطبق التعديلات أو الحذف (null)
    foreach ($overrides as $fieldId => $override) {
        if ($override === null) {
            unset($fieldsAssoc[$fieldId]);   // حذف نهائي
            continue;
        }

        if (isset($fieldsAssoc[$fieldId])) {
            $base = $fieldsAssoc[$fieldId];
            foreach ($override as $key => $value) {
                if ($key === 'validation') {
                    $base['validation'] = self::mergeValidation($base['validation'] ?? [], $value);
                } else {
                    $base[$key] = $value;
                }
            }
            $fieldsAssoc[$fieldId] = $base;
        } else {
            $fieldsAssoc[$fieldId] = $override;
        }
    }

    $fields = array_values($fieldsAssoc);
    usort($fields, fn($a, $b) => ($a['sort_order'] ?? 999) <=> ($b['sort_order'] ?? 999));
    return $fields;
}
```

**النتائج**:
- حذف الحقول المحددة بقيمة `null` في `overrides` يعمل الآن بشكل موثوق.
- دمج الخصائص يتم بشكل دقيق مع معالجة خاصة لـ `validation`.

##### 2. دالة `mergeValidation` لدمج قواعد التحقق بذكاء

تمت إضافة دالة مساعدة جديدة تقوم بفهرسة القواعد الحالية باستخدام `rule` كمفتاح، ثم تستبدل القاعدة إذا تكرر اسمها، مما يمنع تراكم القواعد المكررة وظهور قيم مشوهة (مثل `rule: "required", value: 100`).

```php
protected static function mergeValidation(array $baseValidation, array $newValidation): array
{
    $indexed = [];
    foreach ($baseValidation as $rule) {
        if (isset($rule['rule'])) {
            $indexed[$rule['rule']] = $rule;
        } else {
            $indexed[] = $rule;
        }
    }

    foreach ($newValidation as $rule) {
        if (isset($rule['rule'])) {
            $indexed[$rule['rule']] = $rule;   // استبدال
        } else {
            $indexed[] = $rule;
        }
    }

    return array_values($indexed);
}
```

##### 3. إصلاح دالة `loadConfig` – استبدال الحقول عند تعريفها في ملف الإعدادات

في السابق، كان ملف `document_types.php` يدمج حقوله مع الحقول الافتراضية، مما يؤدي إلى ظهور حقول غير مرغوب فيها. الآن، إذا تم تعريف المفتاح `fields` في ملف الإعدادات، فإنه **يستبدل** الحقول الافتراضية بالكامل:

```php
protected static function loadConfig(): array
{
    // ...
    foreach ($fileConfig as $type => $settings) {
        if (isset($merged[$type])) {
            // إذا كان الملف يحتوي على 'fields'، نستبدل الحقول بالكامل
            if (isset($settings['fields'])) {
                $merged[$type]['fields'] = $settings['fields'];
                unset($settings['fields']);
            }
            $merged[$type] = array_replace_recursive($merged[$type], $settings);
        } else {
            $merged[$type] = $settings;
        }
    }
    // ...
}
```

##### 4. إزالة تكرار الحقول

تم حذف الحقل `religion` من `getDefaultFields` لأنه يُضاف يدويًا عند الحاجة عبر `extraFields` أو `overrides`. كما تم ضبط `overrides` لمنع ظهور حقول مكررة مثل `passport_type`.

##### 5. تصحيح قواعد التحقق في جميع الأنواع

بفضل `mergeValidation` الجديدة، أصبحت قواعد التحقق تعكس القيم الصحيحة المحددة في `overrides` أو ملف الإعدادات. على سبيل المثال:

- `document_number` → `['required', 'max:50']` (بدلاً من القيم المشوهة السابقة).
- `issue_date` → `['required', 'date', 'before_or_equal:today']` (بدون `value` إضافية).
- `country_of_issue` → `['required']` (بدون قواعد `date` الخاطئة).

---

### أمثلة عملية

#### 1. حقل `signature_specimen` (نموذج التوقيع) – قبل وبعد

**قبل الإصلاح (الإصدار 1.2.7)**:
كانت استجابة `GET /document-types-list/signature_specimen` تُرجع **جميع الحقول الافتراضية** (مثل `issue_date`، `address_line1`، `company_name`، إلخ) بالإضافة إلى حقل `signature_image`.

**بعد الإصلاح (الإصدار 1.2.8)**:
الاستجابة تقتصر على الحقول المعرفة في `document_types.php` فقط:

```json
{
    "fields": [
        {
            "id": "signature_image",
            "type": "fileupload",
            "label": { "ar": "نموذج التوقيع", "en": "Signature Specimen" },
            "required": true,
            "validation": [
                { "rule": "required" },
                { "rule": "mimes", "value": "jpg,jpeg,png" },
                { "rule": "max", "value": 2048 }
            ]
        }
    ],
    "rules": {
        "signature_image": ["required", "file", "mimes:jpg,jpeg,png", "max:2048"]
    }
}
```

#### 2. حقل `passport` – قواعد تحقق نظيفة

بعد الإصلاحات، أصبحت قواعد التحقق لحقل `passport` كما يلي (مقتطف):

```json
"rules": {
    "document_front": ["required", "file", "mimes:jpg,jpeg,png,pdf", "max:10240"],
    "document_number": ["required", "max:50"],
    "full_name": ["required", "max:100"],
    "issue_date": ["required", "date", "before_or_equal:today"],
    "expiry_date": ["required", "date", "after_or_equal:today"],
    "country_of_issue": ["required"],
    "nationality": ["required"]
}
```

لاحظ اختفاء القواعد الخاطئة مثل `date:today` المكررة أو `before_or_equal:today - 16 years` على `country_of_issue`.

---

### ملخص الإصدارات (1.2.7 – 1.2.8)

| الإصدار | أبرز الميزات والإصلاحات |
| :--- | :--- |
| 1.2.7 | توافق كامل مع `FormFieldsManager`، تحويل تلقائي للحقول القديمة، إثراء API بالخيارات المحلولة. |
| 1.2.8 | **إصلاح جذري لدمج الحقول وقواعد التحقق**، حذف الحقول غير المرغوب فيها (`null`)، دمج قواعد التحقق بذكاء، منع تسرب الحقول الافتراضية، إزالة التكرارات، تصحيح جميع قواعد التحقق. |

---

### متطلبات الترقية

1. **تحديث الكود**:
   - استبدال ملف `classes/DocumentType.php` بالنسخة الجديدة (يتضمن الدوال المحسنة `mergeFieldsWithOverrides` و `mergeValidation` و `loadConfig`).
   - التأكد من تحديث `getDefaultFields` بإزالة حقل `religion` إن وُجد.

2. **اختبار نقاط النهاية**:
   - جرب `GET /api/v1/kyc/document-types-list/{type}?include_fields=1&include_rules=1` وتأكد من أن الحقول المعروضة تطابق المتوقع.
   - تحقق من عمليات `POST /documents` و `PUT /documents/{id}` للتأكد من أن قواعد التحقق تعمل بشكل صحيح.

3. **مسح الكاش**:
   ```bash
   php artisan cache:clear
   php artisan config:clear
   ```

---

### الخاتمة

الإصدار 1.2.8 هو إصدار استقرار مهم يعالج مشكلات أساسية في آلية دمج الحقول التي ظهرت بعد الانتقال إلى `FormFieldsManager`. بفضل هذه الإصلاحات، أصبح بإمكان المطورين الاعتماد على تعريفات الحقول في ملف الإعدادات بثقة تامة، وستعمل قواعد التحقق في `Manager` كما هو متوقع دون أخطاء.

نشكركم على ملاحظاتكم القيمة التي ساعدت في اكتشاف هذه المشكلات ومعالجتها. نلتزم بمواصلة تحسين إضافة KYC لتكون قوية ومرنة وسهلة الاستخدام.

---

**الوثائق المرجعية**:
- [التوثيق العام للإضافة](./docs/Kyc/Docs-Nano3-Kyc-ar.md)
- [توثيق كلاس `DocumentType`](./docs/Kyc/Docs-DocumentType-Class-ar.md)
- [توثيق كلاس `Manager`](./docs/Kyc/Docs-Manager-Class-ar.md)
- [توثيق السمة `HasAssessKycStatus`](./docs/Kyc/Docs-HasAssessKycStatus-Trait-ar.md)
- [توثيق سمة `HasValidKycDocuments`](./docs/Kyc/Docs-HasValidKycDocuments-Trait-ar.md)
- [توثيق موديل `Document`](./docs/Kyc/Docs-Document-Model-ar.md)
- [توثيق واجهة برمجة التطبيقات (API)](./docs/Kyc/Docs-API-Documentation-ar.md)
