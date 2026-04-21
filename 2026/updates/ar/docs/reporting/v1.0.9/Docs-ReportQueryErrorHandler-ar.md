## توثيق كلاس `ReportQueryErrorHandler`

**نبذة تعريفية عن حزمة `Nano2.QueryBuilder` ووحدة التقارير `Reporting`**

هذه المجموعة من الكلاسات (`ReportSchemaRegistry`, `Table`, `Relationship`, `Column`، وغيرها) هي جزء من **حزمة `Nano2.QueryBuilder`**، وهي حزمة برمجية متكاملة مقدمة من **شركة نانوسوفت (NanoSoft)**، مصممة خصيصًا لتطبيقات **Laravel** بهدف تسهيل بناء وإدارة الاستعلامات المعقدة والتقارير الديناميكية بطريقة آمنة ومرنة.

تقع جميع كلاسات هذه الوحدة ضمن النطاق (Namespace) التالي:

```
Nano2\QueryBuilder\Classes\Reporting
```

توفر هذه الوحدة أدوات متقدمة لتعريف هيكل البيانات (الجداول، الأعمدة، العلاقات)، والتحقق من صحة الاستعلامات، وتنفيذها بكفاءة، وتصدير النتائج بصيغ متعددة، بالإضافة إلى معالجة الأخطاء بشكل موحد. بفضل هذا التصميم، يمكن للمطورين بناء أنظمة تقارير قابلة للتوسع بسهولة وتلبية احتياجات المستخدمين المتقدمة دون المساس بأداء التطبيق أو سلامته.

### مقدمة
كلاس `ReportQueryErrorHandler` هو المسؤول عن **إدارة وتوحيد معالجة الأخطاء** في نظام التقارير الديناميكي (`Nano2\QueryBuilder\Classes\Reporting`). عند حدوث أي خطأ أثناء تنفيذ التقرير (سواء كان خطأ في التحقق، أو خطأ في قاعدة البيانات، أو انتهاء مهلة، أو مشكلة في التصدير)، يقوم هذا الكلاس بتحويل الاستثناء أو الموقف إلى استجابة منظمة تحتوي على معلومات مفيدة للمستخدم (مع رسائل سهلة الفهم) مع تسجيل كافة التفاصيل التقنية في السجلات (Logs) لتسهيل التصحيح.

---

### الخصائص (Properties)

| الخاصية | النوع | الوصف |
|---------|------|-------|
| `$errorCodes` | `array` | مصفوفة تربط أسماء الأخطاء برموز رقمية فريدة (مثلاً `'VALIDATION_FAILED' => 1001`). تُستخدم لتحديد نوع الخطأ بشكل موحد. |
| `$userFriendlyMessages` | `array` | مصفوفة تربط اسم الخطأ برسالة سهلة للمستخدم (مثلاً `'TABLE_NOT_FOUND' => 'الجدول المحدد غير متاح للتقارير.'`). |

---

### أهم الطرق

#### 1. `handleError(\Throwable $exception, array $context = []): array`
**الهدف**: معالجة أي استثناء عام (من `try-catch`) وتحويله إلى استجابة خطأ منظمة.

**المعاملات**:
- `$exception`: كائن الاستثناء الملتقط.
- `$context`: مصفوفة اختيارية تحتوي على معلومات إضافية عن السياق (مثل تكوين التقرير، المستخدم، إلخ) لتسجيلها.

**الخطوات**:
1. تحديد رمز الخطأ باستخدام `determineErrorCode()`.
2. توليد معرف فريد للخطأ باستخدام `generateErrorId()`.
3. تسجيل الخطأ في السجل مع كافة التفاصيل باستخدام `logError()`.
4. بناء استجابة تحتوي على:
   - `success: false`
   - `error`:
     - `code`: رمز الخطأ النصي (مثل `'TIMEOUT'`).
     - `id`: معرف الخطأ الفريد.
     - `message`: رسالة سهلة للمستخدم.
     - `details`: تفاصيل تقنية محدودة (مثل نوع الاستثناء، الملف، السطر).
     - `suggestions`: قائمة اقتراحات لحل المشكلة.
     - `timestamp`: وقت الخطأ بصيغة ISO.
   - `meta`: معلومات إضافية مثل `request_id`, `user_id`, `company_id`.

**مثال على الاستجابة**:
```json
{
  "success": false,
  "error": {
    "code": "TABLE_NOT_FOUND",
    "id": "ERR_ABC12345_1712345678",
    "message": "الجدول المحدد غير متاح للتقارير.",
    "details": {
      "type": "InvalidArgumentException",
      "file": "ReportQueryConverter.php",
      "line": 123
    },
    "suggestions": [
      "تحقق من اسم الجدول.",
      "تأكد من صلاحيات الوصول."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  },
  "meta": {
    "request_id": "req-xyz-789",
    "user_id": 42,
    "company_id": "comp-123"
  }
}
```

#### 2. `handleValidationError(array $validationResult, array $context = []): array`
**الهدف**: معالجة أخطاء التحقق التي يعيدها `ReportQueryValidator`.

**المعاملات**:
- `$validationResult`: النتيجة من `ReportQueryValidator::validate()` والتي تحتوي على `errors` و `warnings`.
- `$context`: سياق إضافي للتسجيل.

**الخطوات**:
- توليد معرف خطأ.
- تسجيل تحذير في السجل مع الأخطاء والتحذيرات.
- إرجاع استجابة تحتوي على `validation_errors` و `validation_warnings` و `suggestions` المستخرجة عبر `getValidationSuggestions()`.

**مثال على الاستجابة**:
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_FAILED",
    "id": "ERR_DEF45678_1712345678",
    "message": "Query validation failed",
    "validation_errors": [
      "Column 'xyz' does not exist in table 'orders'"
    ],
    "validation_warnings": [
      "Many conditions may impact performance"
    ],
    "suggestions": [
      "تحقق من وجود الأعمدة المحددة.",
      "قلل عدد الشروط لتحسين الأداء."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  }
}
```

#### 3. `handleTimeoutError(float $executionTime, array $context = []): array`
**الهدف**: معالجة أخطاء انتهاء المهلة (Timeout) بشكل خاص.

**المعاملات**:
- `$executionTime`: وقت التنفيذ الفعلي (بالملي ثانية) قبل حدوث المهلة.
- `$context`: سياق إضافي.

**الاستجابة**:
- تحتوي على `execution_time` واقتراحات محددة للتعامل مع المهلة.

**مثال**:
```json
{
  "success": false,
  "error": {
    "code": "TIMEOUT",
    "id": "ERR_GHI78901_1712345678",
    "message": "Query execution timed out",
    "execution_time": 30500,
    "suggestions": [
      "قلل عدد الأعمدة المحددة.",
      "أضف فلاتر أكثر تحديداً.",
      "زد الحد المسموح به (limit)."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  }
}
```

#### 4. `handleExportError(\Throwable $exception, string $format, array $context = []): array`
**الهدف**: معالجة الأخطاء التي تحدث أثناء تصدير النتائج (باستخدام `ReportQueryExporter`).

**المعاملات**:
- `$exception`: الاستثناء الملتقط.
- `$format`: صيغة التصدير المطلوبة (csv, excel, json, pdf, xml).
- `$context`: سياق إضافي.

**الاستجابة**:
- تحتوي على `format` واقتراحات محددة للتصدير.

**مثال**:
```json
{
  "success": false,
  "error": {
    "code": "EXPORT_FAILED",
    "id": "ERR_JKL01234_1712345678",
    "message": "Export to pdf format failed",
    "format": "pdf",
    "suggestions": [
      "جرب صيغة تصدير أخرى.",
      "قلل كمية البيانات المصدرة.",
      "تحقق من مساحة التخزين."
    ],
    "timestamp": "2025-04-06T10:30:00Z"
  }
}
```

#### 5. `formatErrorForOutput(array $error, string $outputType = 'json'): mixed`
**الهدف**: تنسيق الخطأ لمخرجات مختلفة (JSON، HTML، نص عادي). حاليًا يدعم JSON بشكل افتراضي، مع وجود دوال مساعدة `formatErrorAsHtml` و `formatErrorAsText` (يمكن توسيعها لاحقاً).

**المعاملات**:
- `$error`: مصفوفة الخطأ (كما تم إرجاعها من الدوال السابقة).
- `$outputType`: نوع المخرجات (`'json'`, `'html'`, `'text'`).

**الاستخدام**: مفيد إذا كان النظام يدعم استجابات متعددة (مثل واجهة برمجية تتعامل مع طلبات Ajax قد تطلب HTML لعرض الخطأ مباشرة).

---

### الدوال المساعدة (Protected)

#### `determineErrorCode(\Throwable $exception): string`
- يحلل رسالة الاستثناء ليحدد رمز الخطأ الأنسب (مثل `TABLE_NOT_FOUND`, `COLUMN_NOT_FOUND`, `PERMISSION_DENIED`, `TIMEOUT`, `MEMORY_LIMIT`, `CONNECTION_ERROR`, `VALIDATION_FAILED`، أو `QUERY_EXECUTION_FAILED` كافتراضي).

#### `getUserFriendlyMessage(string $errorCode): string`
- يُرجع الرسالة المناسبة للمستخدم من `$userFriendlyMessages`، أو رسالة افتراضية إذا لم يجد.

#### `getErrorDetails(\Throwable $exception, string $errorCode): array`
- يُرجع تفاصيل تقنية محدودة (نوع الاستثناء، اسم الملف، رقم السطر). يضيف تفاصيل إضافية حسب رمز الخطأ (مثل `timeout_limit`، `memory_limit`، `database`).

#### `getErrorSuggestions(string $errorCode): array`
- يُرجع قائمة اقتراحات لحل المشكلة بناءً على رمز الخطأ.

#### `getValidationSuggestions(array $errors): array`
- يحلل أخطاء التحقق وينشئ اقتراحات محددة بناءً على محتوى كل خطأ (مثل وجود كلمة "table"، "column"، "join"، "condition").

#### `logError(\Throwable $exception, string $errorCode, string $errorId, array $context): void`
- يسجل الخطأ في سجل الأخطاء (`Log::error`) مع كافة التفاصيل (الاستثناء، السياق، المستخدم، الشركة، الطلب، IP، User-Agent).

#### `generateErrorId(): string`
- يُنشئ معرفاً فريداً للخطأ بصيغة `ERR_` + 8 أحرف عشوائية + `_` + timestamp. مثال: `ERR_AB12CD34_1712345678`.

#### `isRecoverable(string $errorCode): bool`
- يحدد ما إذا كان الخطأ قابلاً للاسترداد (أي يمكن إعادة المحاولة لاحقاً). الأخطاء القابلة للاسترداد: `TIMEOUT`, `MEMORY_LIMIT`, `RATE_LIMIT_EXCEEDED`, `CONNECTION_ERROR`.

#### `getRetrySuggestions(string $errorCode): array`
- يُرجع اقتراحات إعادة المحاولة (مدة الانتظار، رسالة) للأخطاء القابلة للاسترداد.

---

### أمثلة عملية على استخدام الكلاس

#### السيناريو 1: استخدام `handleError` في كتلة `try-catch`
```php
use Nano2\QueryBuilder\Classes\Reporting\ReportQueryErrorHandler;

class ReportController extends Controller
{
    public function generateReport(Request $request, ReportQueryErrorHandler $errorHandler)
    {
        try {
            // تنفيذ التقرير...
            $converter = new ReportQueryConverter($registry, $queryConfig);
            $result = $converter->execute();
            return response()->json($result);
        } catch (\Throwable $e) {
            // تجميع السياق للمساعدة في التصحيح
            $context = [
                'query_config' => $queryConfig,
                'user_id' => auth()->id(),
                'company_id' => session('company'),
            ];
            
            $errorResponse = $errorHandler->handleError($e, $context);
            return response()->json($errorResponse, 500);
        }
    }
}
```

**سيناريو خطأ (جدول غير موجود):**
- الاستثناء: `new \InvalidArgumentException("Table 'non_existent' not found")`.
- الاستجابة الناتجة (كما في المثال السابق) برمز `TABLE_NOT_FOUND` ورسالة مناسبة.

#### السيناريو 2: استخدام `handleValidationError` بعد التحقق
```php
$validator = new ReportQueryValidator($registry);
$validation = $validator->validate($queryConfig);

if (!$validation['valid']) {
    $errorResponse = $errorHandler->handleValidationError($validation, ['query_config' => $queryConfig]);
    return response()->json($errorResponse, 422);
}
```

#### السيناريو 3: معالجة مهلة التنفيذ
```php
$startTime = microtime(true);
// ... تنفيذ استعلام طويل ...
$executionTime = microtime(true) - $startTime;

if ($executionTime > 30) { // افترض أن الحد الأقصى 30 ثانية
    $errorResponse = $errorHandler->handleTimeoutError($executionTime * 1000, ['query_config' => $queryConfig]);
    return response()->json($errorResponse, 500);
}
```

#### السيناريو 4: معالجة خطأ التصدير
```php
try {
    $exporter = new ReportQueryExporter($data, $columns, $metadata, $tableName);
    $exportResult = $exporter->export($format);
    return response()->json($exportResult);
} catch (\Throwable $e) {
    $errorResponse = $errorHandler->handleExportError($e, $format, ['table' => $tableName]);
    return response()->json($errorResponse, 500);
}
```

#### السيناريو 5: الحصول على اقتراحات إعادة المحاولة
```php
if ($errorHandler->isRecoverable($errorCode)) {
    $retryInfo = $errorHandler->getRetrySuggestions($errorCode);
    // $retryInfo = ['wait_time' => 30, 'message' => 'Try again in 30 seconds...']
    // يمكن إضافتها إلى الاستجابة أو عرضها للمستخدم.
}
```

---

### قائمة أكواد الأخطاء (Error Codes)

| الرمز | المعنى | رسالة المستخدم (مترجمة) |
|-------|--------|--------------------------|
| `VALIDATION_FAILED` | فشل التحقق من التكوين | تكوين التقرير يحتوي على أخطاء. يرجى المراجعة. |
| `TABLE_NOT_FOUND` | الجدول غير موجود | الجدول المحدد غير متاح للتقارير. |
| `COLUMN_NOT_FOUND` | عمود غير موجود | عمود واحد أو أكثر غير متاح. |
| `PERMISSION_DENIED` | صلاحيات غير كافية | لا تملك صلاحية الوصول لهذه البيانات. |
| `QUERY_EXECUTION_FAILED` | فشل تنفيذ الاستعلام | تعذر تنفيذ الاستعلام. يرجى التبسيط. |
| `EXPORT_FAILED` | فشل التصدير | تعذر إتمام التصدير. حاول مجدداً. |
| `TIMEOUT` | انتهاء المهلة | استغرق الاستعلام وقتاً طويلاً. حاول تقليل نطاق البيانات. |
| `MEMORY_LIMIT` | تجاوز حد الذاكرة | الاستعلام يستهلك ذاكرة كبيرة. حاول تحديد النتائج. |
| `INVALID_CONFIGURATION` | تكوين غير صالح | تكوين التقرير غير صالح. |
| `SCHEMA_ERROR` | خطأ في المخطط | حدث خطأ في مخطط البيانات. |
| `CONNECTION_ERROR` | خطأ في الاتصال | تعذر الاتصال بقاعدة البيانات. |
| `RATE_LIMIT_EXCEEDED` | تجاوز حد الطلبات | عدد كبير من الطلبات. انتظر قليلاً. |

---

### ملاحظات مهمة
- الكلاس يركز على **توحيد الاستجابات** وتوفير رسائل آمنة للمستخدم مع إخفاء التفاصيل التقنية الحساسة (مثل أسماء الملفات الكاملة، تتبع الاستدعاءات الطويلة) ولكن يسجلها في السجلات.
- دالة `logError` تسجل كمية كبيرة من المعلومات (بما في `trace`) مما يساعد في التصحيح.
- يمكن توسيع الكلاس بإضافة صيغ مخرجات جديدة (مثل XML) عبر تعديل `formatErrorForOutput` وإضافة دوال مساعدة.
- `getValidationSuggestions` تحلل الأخطاء النصية لتقديم اقتراحات مخصصة، مما يحسن تجربة المستخدم.

---

### الخلاصة
`ReportQueryErrorHandler` هو أداة أساسية لبناء نظام تقارير متين وسهل الاستخدام. من خلال توحيد معالجة الأخطاء، يضمن أن المستخدمين يتلقون رسائل واضحة ومفيدة، بينما يحتفظ المطورون بكل التفاصيل اللازمة لتصحيح الأخطاء. هذا يساهم في تحسين استقرار النظام وتجربة المستخدم النهائية.