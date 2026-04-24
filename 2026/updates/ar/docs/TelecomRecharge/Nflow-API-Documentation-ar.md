# توثيق واجهة برمجة التطبيقات (API) – خدمات الدفع الإلكتروني (Nflow Integration)

**الإصدار:** v1  
**المسار الأساسي:** `https://[domain]/api/v1/telecomrecharge/nflow`  
**Or:** `http://localhost:8006/api/v1/telecomrecharge/nflow`  

**المصادقة:** OAuth 2.0 (يُمرر توكن الوصول في الهيدر `Authorization: Bearer <token>`)

---

## نظرة عامة

هذه الوثيقة تشرح جميع نقاط النهاية المتاحة للتفاعل مع منصة **nflow.tech** (ماستر هوست) عبر واجهة برمجة التطبيقات (API) المُدمجة في إضافة `Nano3.TelecomRecharge`. تغطي الخدمات:

- إدارة حساب الوكيل (رصيد، إيداعات، حالة عمليات)
- استعلام أرصدة شركات الاتصالات اليمنية (يمن موبايل، ADSL، هاتف أرضي، Yemen 4G، عدن نت)
- شحن رصيد مباشر لجميع الشبكات المدعومة
- تفعيل باقات الإنترنت والخدمات
- شحن الألعاب وبطاقات الهدايا (بوبجي، يويو شات، وغيرها)

جميع الطلبات تحتاج إلى **توكن OAuth صالح** (يُضاف في الهيدر) وتستخدم طريقة `POST` أو `GET` حسب الحاجة. تستجيب جميع النقاط بتنسيق JSON موحد.

---

## هيكل الاستجابة الموحد

جميع نقاط النهاية تعيد استجابة بنفس الهيكل الأساسي:

```json
{
    "code": 200,
    "status": true,
    "message": "نص الرسالة",
    "error": null,
    "errors": [],
    "data": {},
    "meta": {},
    "input_data": {},
    "process_data": {},
    "debug": {}
}
```

### حقول الاستجابة

| الحقل | النوع | الوصف |
|-------|-------|-------|
| `code` | integer | كود HTTP (200 للنجاح، 400، 401، 402، 403، 404، 500 للأخطاء) |
| `status` | boolean | `true` للنجاح، `false` للفشل |
| `message` | string | رسالة وصفية مناسبة للمستخدم |
| `error` | string | رسالة الخطأ الأساسية (في حالة الفشل) |
| `errors` | array | مصفوفة من رسائل الأخطاء التفصيلية (مثل أخطاء التحقق) |
| `data` | mixed | بيانات الاستجابة الرئيسية (مصفوفة أو كائن) |
| `meta` | mixed | بيانات إضافية (مثل معلومات التصفح، الوقت المنقضي) |
| `input_data` | mixed | البيانات التي أرسلها المستخدم (للتتبع) |
| `process_data` | mixed | بيانات المعالجة الداخلية (مثل نتيجة الخدمة) |
| `debug` | mixed | معلومات التصحيح (تظهر فقط عند تفعيل `app.debug`) |

---

## المصادقة

يتم إرسال توكن الوصول في هيدر `Authorization` بالصيغة:

```
Authorization: Bearer {access_token}
```

يُحصل على هذا التوكن من خلال نظام OAuth الخاص بالتطبيق. (إذا كنت تستخدم October CMS مع Laravel OAuth، يتم إضافته تلقائيًا بعد تسجيل الدخول).

---

## نقاط النهاية (Endpoints)

### 1. العمليات العامة (Account Operations)

#### 1.1 استعلام رصيد الوكيل

**GET** `/account-balance`

**الوصف:** استرجاع الرصيد الحالي لحساب الوكيل.

**مثال الطلب:**

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/account-balance" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم استعلام الرصيد بنجاح",
    "error": null,
    "errors": null,
    "data": {
        "status": true,
        "agentBalance": 1500.50,
        "message": "Agent Balance Query Success",
        "transactionID": 0
    },
    "input_data": {},
    "process_data": {
        "balance_result": { ... }
    }
}
```

---

#### 1.2 استعلام حالة عملية سابقة

**GET** `/operation-status?transactionId={transactionId}`

**الوصف:** استعلام حالة معاملة تم إرسالها مسبقًا.

| المعامل | النوع | مطلوب | الوصف |
|---------|-------|-------|-------|
| transactionId | string | نعم | معرف المعاملة الخاص بك (كما أرسلته في الطلب الأصلي) |

**مثال الطلب:**

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/operation-status?transactionId=TXN_123" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم استعلام حالة المعاملة بنجاح",
    "data": {
        "status": true,
        "operationStatus": 1,
        "mobileNumber": "777777777",
        "price": 100,
        "message": "جاهزة",
        "transactionID": "TXN_123",
        "referenceID": 98765
    },
    "input_data": {
        "transactionId": "TXN_123"
    }
}
```

**رموز الحالة:**  
- `operationStatus = 1` → نجاح  
- `operationStatus = 0` → فشل  
- `operationStatus = -1` → قيد الانتظار

---

#### 1.3 استعلام إيداعات الوكيل

**GET** `/feed-clients-balance`

**الوصف:** عرض قائمة الإيداعات التي أضيفت إلى حساب الوكيل.

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب إيداعات الوكيل بنجاح",
    "data": {
        "status": true,
        "data": [
            {
                "Date": "2024-08-03",
                "Amount": "70000.00000",
                "Currency": "ريال يمني",
                "Notes": "Notes"
            }
        ],
        "message": "Query Success"
    }
}
```

---

#### 1.4 جلب فئات الألعاب وبطاقات الهدايا

**GET** `/games-categories`

**الوصف:** الحصول على قائمة الألعاب والبطاقات المتاحة مع أسعارها والحقول المطلوبة.

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب فئات الألعاب والبطاقات بنجاح",
    "data": {
        "status": true,
        "data": [
            {
                "TheNumber": 1,
                "ServiceName": "بـوبـجـي",
                "CategoryName": "بوبجي ( 60 شدة )",
                "LinkCode": "pubg_60",
                "LocalPrice": 467.61,
                "LocalCurrencyName": "ريال يمني",
                "RequiredFields": [
                    { "FieldCode": "PlayerID", "FieldName": "رقم ايدي - اللاعب" }
                ]
            }
        ],
        "message": "Success"
    }
}
```

---

### 2. الدوال العامة (المرنة)

#### 2.1 تنفيذ خدمة مخصصة

**POST** `/execute`

**الوصف:** تنفيذ أي خدمة بتحديد رقم الشبكة ورقم الخدمة يدويًا (أقصى مرونة).

**بيانات الطلب (JSON):**

| الحقل | النوع | مطلوب | الوصف |
|-------|-------|-------|-------|
| networkNumber | int | نعم | رقم الشبكة (مثال: 1 ليمن موبايل) |
| serviceNumber | int | نعم | رقم الخدمة (مثال: 101 لاستعلام رصيد يمن موبايل) |
| additionalFields | object | لا | حقول إضافية مثل `MobileNumber`, `Amount`, `OfferCode` |
| transactionId | string | لا | معرف معاملة (يُولَّد تلقائياً إذا لم يُمرر) |

**مثال الطلب:**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/execute" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 1,
    "serviceNumber": 102,
    "additionalFields": {
        "MobileNumber": "777777777"
    },
    "transactionId": "TXN_123"
  }'
```

**استجابة ناجحة (200):**  
تعتمد على الخدمة، لكنها تتضمن عادةً `operationStatus` و `agentBalance` و `price`.

---

#### 2.2 استعلام رصيد أي شبكة

**GET** `/balance/query?networkNumber={networkNumber}&mobileNumber={mobileNumber}&transactionId={transactionId}`

**الوصف:** استعلام رصيد رقم في أي شبكة تدعم خدمة الاستعلام.

| المعامل | النوع | مطلوب | الوصف |
|---------|-------|-------|-------|
| networkNumber | int | نعم | رقم الشبكة (1,5,6,7,13) |
| mobileNumber | string | نعم | رقم الجوال أو الخط |
| transactionId | string | لا | معرف معاملة (يُولَّد تلقائياً) |

**مثال الطلب:**

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/balance/query?networkNumber=1&mobileNumber=777777777" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم استعلام الرصيد بنجاح",
    "data": {
        "status": true,
        "mobileBalance": 154.15,
        "mobileTypeName": "دفع مسبق",
        "message": "Balance Query Success"
    },
    "input_data": {
        "networkNumber": "1",
        "mobileNumber": "777777777"
    }
}
```

**حالات الخطأ:**  
- إذا كانت الشبكة لا تدعم استعلام الرصيد → كود 400 مع رسالة "هذه الشبكة لا تدعم خدمة استعلام الرصيد".

---

#### 2.3 شحن رصيد أي شبكة

**POST** `/balance/charge`

**الوصف:** شحن رصيد لرقم في أي شبكة تدعم خدمة الشحن المباشر.

**بيانات الطلب (JSON):**

| الحقل | النوع | مطلوب | الوصف |
|-------|-------|-------|-------|
| networkNumber | int | نعم | رقم الشبكة |
| mobileNumber | string | نعم | رقم الجوال أو الخط |
| amount | float | نعم | المبلغ (للشبكات التي تستخدم وحدات، يُمثل عدد الوحدات) |
| transactionId | string | لا | معرف معاملة (يُولَّد تلقائياً) |
| webHookUrl | string | لا | رابط WebHook لتلقي تحديثات الحالة |
| webHookCode | string | لا | كود أمان للـ WebHook |

**مثال الطلب:**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/balance/charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 1,
    "mobileNumber": "777777777",
    "amount": 500,
    "transactionId": "TXN_456",
    "webHookUrl": "https://example.com/webhook"
  }'
```

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تمت عملية الشحن بنجاح",
    "data": {
        "status": true,
        "operationStatus": 1,
        "agentBalance": 1500.50,
        "price": 500,
        "transactionID": "TXN_456",
        "referenceID": 123456
    },
    "input_data": { ... },
    "process_data": { "charge_result": { ... } }
}
```

---

#### 2.4 تفعيل باقة أي شبكة

**POST** `/offer/activate`

**الوصف:** تفعيل باقة إنترنت أو خدمة لرقم في أي شبكة تدعم الباقات.

**بيانات الطلب (JSON):**

| الحقل | النوع | مطلوب | الوصف |
|-------|-------|-------|-------|
| networkNumber | int | نعم | رقم الشبكة |
| mobileNumber | string | نعم | رقم الجوال |
| offerCode | string | نعم | كود الباقة (مثل `PREWhatsApp`) |
| transactionId | string | لا | معرف معاملة |
| webHookUrl | string | لا | رابط WebHook |
| webHookCode | string | لا | كود أمان |

**مثال الطلب:**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/offer/activate" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 2,
    "mobileNumber": "777777777",
    "offerCode": "PREWhatsApp",
    "transactionId": "TXN_789"
  }'
```

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم تفعيل الباقة بنجاح",
    "data": {
        "status": true,
        "operationStatus": 1,
        "agentBalance": 1490.00,
        "price": 10.50,
        "transactionID": "TXN_789",
        "referenceID": 78901
    }
}
```

---

### 3. استعلامات الرصيد للشبكات (اختصارات)

| الطريقة | المسار | الوصف | مثال |
|---------|--------|-------|------|
| GET | `/yemen-mobile/balance?mobileNumber={num}` | رصيد يمن موبايل | `?mobileNumber=777777777` |
| GET | `/yemen-mobile/loan?mobileNumber={num}` | قرض يمن موبايل | |
| GET | `/yemen-mobile/offers?mobileNumber={num}` | باقات يمن موبايل | |
| GET | `/adsl/balance?mobileNumber={num}` | رصيد ADSL | |
| GET | `/land-phone/balance?mobileNumber={num}` | رصيد هاتف أرضي | |
| GET | `/yemen-4g/balance?mobileNumber={num}` | رصيد Yemen 4G | |
| GET | `/aden-net/balance?mobileNumber={num}` | رصيد عدن نت | |

**مثال استجابة (يمن موبايل):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم الاستعلام بنجاح",
    "data": {
        "status": true,
        "mobileBalance": 154.15,
        "mobileTypeName": "دفع مسبق",
        "message": "Balance Query Success"
    },
    "input_data": {
        "mobileNumber": "777777777"
    }
}
```

---

### 4. شحن الرصيد (اختصارات)

#### 4.1 شحن يمن موبايل

**POST** `/yemen-mobile/charge`

**بيانات الطلب (JSON):**

| الحقل | النوع | مطلوب | الوصف |
|-------|-------|-------|-------|
| mobileNumber | string | نعم | رقم الجوال |
| amount | float | نعم | المبلغ |
| transactionId | string | نعم | معرف معاملة |
| webHookUrl | string | لا | رابط WebHook |
| webHookCode | string | لا | كود WebHook |

**استجابة:** مثل `balance/charge`.

---

### 5. تفعيل الباقات (اختصارات)

| الطريقة | المسار | الوصف | مثال للطلب |
|---------|--------|-------|------------|
| POST | `/you/offer/activate` | تفعيل باقة YOU Telecom | `{"mobileNumber":"777777777","offerCode":"PREWhatsApp","transactionId":"..."}` |
| POST | `/sabafon/offer/activate` | تفعيل باقة سبافون | `{"mobileNumber":"777777777","offerCode":"SAB_OFFER","transactionId":"..."}` |

---

### 6. الألعاب وبطاقات الهدايا

#### 6.1 شحن لعبة أو بطاقة

**POST** `/game-charge`

**بيانات الطلب (JSON):**

| الحقل | النوع | مطلوب | الوصف |
|-------|-------|-------|-------|
| linkCode | string | نعم | كود اللعبة (من فئات الألعاب) |
| fields | object | نعم | الحقول المطلوبة (مثل `PlayerID`) |
| quantity | int | نعم | الكمية (للألعاب التي تسمح بكمية حرة) |
| transactionId | string | نعم | معرف معاملة |
| mobileNumber | string | لا | رقم الجوال إن لزم |
| webHookUrl | string | لا | رابط WebHook |
| webHookCode | string | لا | كود WebHook |

**مثال الطلب (بوبجي):**

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/game-charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "linkCode": "pubg_60",
    "fields": {"PlayerID": "123456789"},
    "quantity": 1,
    "transactionId": "TXN_123"
  }'
```

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم شحن اللعبة أو البطاقة بنجاح",
    "data": {
        "status": true,
        "operationStatus": 1,
        "agentBalance": 1440.00,
        "price": 467.61,
        "transactionID": "TXN_123",
        "referenceID": 98765
    }
}
```

---

### 7. دوال البيانات والمساعدة

#### 7.1 الحصول على قائمة الشبكات

**GET** `/networks`

**الوصف:** إرجاع مصفوفة بأرقام الشبكات وأسمائها.

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم جلب قائمة الشبكات بنجاح",
    "data": {
        "0": "Global",
        "1": "Yemen Mobile",
        "2": "YOU Telecom",
        "3": "Sabafon",
        "4": "Y Telecom",
        "5": "ADSL",
        "6": "Land Phone",
        "7": "Yemen 4G",
        "8": "Games And Gift Cards",
        "9": "Games And Gift Cards",
        "12": "Sabafon South",
        "13": "Aden Net"
    }
}
```

---

#### 7.2 الحصول على قائمة الخدمات

**GET** `/services`

**الوصف:** إرجاع قائمة كاملة بالخدمات لكل شبكة (كما في جدول Services Data).

**استجابة ناجحة (200):**  
تُعيد مصفوفة شبيهة بـ `getAllServices()` في الكلاس.

---

#### 7.3 اختبار الاتصال

**GET** `/test-connection`

**الوصف:** اختبار صحة بيانات الاعتماد وإمكانية الاتصال بـ nflow.tech.

**استجابة ناجحة (200):**

```json
{
    "code": 200,
    "status": true,
    "message": "تم اختبار الاتصال",
    "data": {
        "success": true,
        "message": "تم الاتصال بنجاح",
        "token": true
    }
}
```

---

## رموز الاستجابة (كود HTTP)

| الكود | المعنى |
|-------|--------|
| 200 | نجاح العملية |
| 400 | خطأ في الطلب (بيانات ناقصة أو غير صحيحة) |
| 401 | فشل المصادقة (توكن غير صالح أو منتهي الصلاحية) |
| 402 | رصيد غير كافٍ |
| 403 | غير مصرح (الرقم محظور أو صلاحية غير كافية) |
| 404 | الخدمة أو الرقم غير موجود |
| 406 | تنسيق غير صحيح (رقم هاتف خاطئ، مثلاً) |
| 416 | القيمة خارج النطاق المسموح |
| 423 | الخدمة مقفلة أو غير متاحة مؤقتًا |
| 429 | تجاوز الحد المسموح من الطلبات |
| 500 | خطأ داخلي في الخادم |

جميع الاستجابات تحتوي على حقل `status` (true/false) وحقل `message` وصفياً.

---

## معالجة الأخطاء

في حالة حدوث خطأ، تعيد النقطة استجابة بهيكل موحد:

```json
{
    "code": 400,
    "status": false,
    "message": "بيانات غير صالحة",
    "error": "خطأ في التحقق",
    "errors": [
        "networkNumber مطلوب"
    ],
    "input_data": { ... },
    "debug": { ... } // يظهر فقط عند تفعيل app.debug
}
```

**أمثلة على أخطاء شائعة:**

| الخطأ | كود | رسالة |
|-------|-----|-------|
| بيانات ناقصة | 400 | "networkNumber و mobileNumber مطلوبان" |
| فشل المصادقة | 401 | "فشل المصادقة، تأكد من بيانات الاعتماد" |
| رصيد غير كافٍ | 402 | "رصيد غير كافٍ" |
| الرقم محظور | 403 | "الرقم محظور" |
| خدمة غير موجودة | 404 | "الخدمة أو الرقم غير موجود" |
| تنسيق رقم خاطئ | 406 | "صيغة رقم الهاتف غير صحيحة" |
| خدمة مقفلة | 423 | "الخدمة مقفلة مؤقتًا" |
| حد الطلبات | 429 | "تجاوز الحد المسموح من الطلبات" |

---

## ملاحظات

- **معرف المعاملة (transactionId):** يوصى بأن يكون فريدًا لكل طلب لتجنب التكرار. يمكن توليده بـ `uniqid('TXN_', true)`.
- **Webhook:** عند تمرير `webHookUrl`، سترسل المنصة تحديثات الحالة (GET) إلى هذا الرابط مع معاملات `OperationStatus`, `TransactionID`, `ReferenceID`, `price`, `message`.
- **شبكات الوحدات:** بالنسبة لشبكات (2) YOU, (3) Sabafon, (4) Y Telecom, (12) Sabafon South، قيمة `amount` تمثل عدد الوحدات، وليس المبلغ النقدي. يتم ضربها في سعر الوحدة المحدد في النظام.
- **توافق OAuth:** جميع نقاط النهاية محمية بـ `oauth-users`، لذا يجب أن يكون المستخدم مسجل دخوله ولديه الصلاحيات المناسبة.
- **التصحيح (debug):** في بيئة الإنتاج (`app.debug = false`)، لا تظهر حقول `debug` ولا تفاصيل حساسة في رسائل الخطأ. في بيئة التطوير، تظهر معلومات إضافية للمساعدة في التصحيح.

---

## أمثلة باستخدام cURL

### استعلام رصيد يمن موبايل

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/yemen-mobile/balance?mobileNumber=777777777" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### شحن رصيد

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/balance/charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "networkNumber": 1,
    "mobileNumber": "777777777",
    "amount": 100,
    "transactionId": "TXN_123"
  }'
```

### شحن لعبة

```bash
curl -X POST "http://localhost:8006/api/v1/telecomrecharge/nflow/game-charge" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "linkCode": "pubg_60",
    "fields": {"PlayerID": "123456789"},
    "quantity": 1,
    "transactionId": "TXN_456"
  }'
```

### استعلام حالة معاملة

```bash
curl -X GET "http://localhost:8006/api/v1/telecomrecharge/nflow/operation-status?transactionId=TXN_123" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

**ملاحظة:** هذا التوثيق مبني على الإصدار v1 من واجهة API. أي تحديثات مستقبلية سيتم الإعلان عنها في سجل التغييرات.