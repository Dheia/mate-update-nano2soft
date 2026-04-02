## Documentation of Dynamic Variables in Report Conditions
**Advanced Version – `Nano2.QueryBuilder.Reporting` System**

---

### 1. Introduction

In the world of dynamic reports, the values used in conditions are rarely static. These values often depend on the execution context, such as the identity of the current user, application settings, temporarily stored values, or even the results of computational operations. To meet these needs, the reporting system provides a **Dynamic Variables** mechanism that allows defining intelligent value sources within report conditions, with full control over their behavior.

It is no longer limited to simple text replacement; you can now:
- Specify **search priority** (do we take the value from user input first or from the source?).
- Enforce **required** fields or **allow null values** (nullable).
- Apply **type casting** to ensure query integrity.
- Provide **default values** in case the source fails.
- Perform **strict** verification of the source's existence itself.
- Easily handle **arrays** (multiple).

This documentation explains in detail how to use this advanced feature, with practical examples covering all possible cases.

---

### 2. Dynamic Variable Structure

The dynamic variable is written in the `variable` key within a condition using the following unified format:

```
source:parameter|option1:value|option2|option3
```

- **Source**: Specifies the source of the value (see section 3).
- **Parameter**: Additional information for the source (e.g., `id` for user, or a cache key).
- **Options**: A list of options separated by `|`, which can be:
  - **Simple** (without value): such as `required`, `strict`, `nullable`.
  - **With value**: such as `as:userId`, `default:100`, `cast:int`.

#### Illustrative Example:
```php
'variable' => 'user:id|required|cast:int|as:userId'
```
**Analysis**:
- Source: `user` with parameter `id` (i.e., the current user's ID).
- Options:
  - `required`: Mandatory – if no value is obtained, the report is prevented.
  - `cast:int`: Convert the value to an integer.
  - `as:userId`: Store the value in `$filterValues['userId']` instead of the original key.

---

### 3. Supported Sources

#### 3.1 `user` – Current User Data
Retrieved from `ReportUserManager`. Available parameters:
- `id`: User ID.
- `company_id`: Company ID (if available).
- `user_type`: User type (e.g., `Backend\Models\User` or the value returned by `getMorphClass`).
- `name`: Full name.
- `email`: Email address.
- `mobile`: Phone number.
- Any other property present on the user object (e.g., `role_id`, `branch_id`).

**Example**: `user:company_id|required`

#### 3.2 `cache` – Values Stored in Cache
Uses `Cache::get($key)` where `$key` is the specified parameter.

**Example**: `cache:reports.start_date|default:2025-01-01|cast:date`

#### 3.3 `config` – Application Configuration
Uses `config($path)` where `$path` is the configuration path (e.g., `app.timezone`).

**Example**: `config:app.default_company|required|cast:int`

#### 3.4 `static` – Calling Static Methods
Supports the formats:
- `ClassName::method`
- `ClassName@method`
- `function_name` (regular function)

The function is called without parameters and must be callable at report execution time.

**Example**: `static:App\Helpers\DateHelper::getFirstDayOfMonth|override`

---

### 4. Advanced Options – Detailed Explanation

#### 4.1 Specifying the Storage Key – `as:key`
By default, the resolved value is stored in the `$filterValues` array with the same original string (e.g., `'user:id'`). This key might be too long or unsuitable for subsequent processing. Using `as`, you can set a custom key.

**Example**:
```php
'variable' => 'user:id|as:userId'
```
Result: The value is stored in `$filterValues['userId']`.

#### 4.2 Search Priority – `override`, `fallback`, `source`
These options determine how to merge the value coming from the source with the values sent by the user via `filters`.

- **`override`**: Priority is given to the value sent by the user. If a value exists in `filters` with the specified key (considering `as`), it is used directly. Otherwise, the source is resolved.
- **`fallback`**: Tries to resolve the source first. If it fails, it looks in `filters`.
- **`source`**: (Default behavior) Tries to resolve the source only. If it fails, it does not look in filters. It can be written explicitly for clarity.

**Comparison Example**:
```php
// Priority to user
'variable' => 'user:email|override|as:userEmail'

// Priority to source, if it fails then user
'variable' => 'user:email|fallback|as:userEmail'

// Source only (default)
'variable' => 'user:email|source|as:userEmail'  // or 'user:email|as:userEmail'
```

#### 4.3 Required and Validation – `required`, `nullable`, `strict`
- **`required`**: If no value is obtained (neither from source nor from filters according to priority), report execution is prevented and a clear error is returned.
- **`nullable`**: Allows the value to be `null`. If no value exists, `null` is passed to the condition (instead of ignoring the condition). Useful for conditions that handle empty values like `is_null`.
- **`strict`**: If the resolution of the source itself fails (even if the value is optional and not required), the report is prevented. Used when the existence of the source is a prerequisite for the report's integrity.

**Examples**:
```php
// Required – a user must exist
'variable' => 'user:id|required'

// Allows null if the value does not exist
'variable' => 'config:app.optional_setting|nullable'

// Immediate failure if the user does not have a company_id (even if the condition is not mandatory)
'variable' => 'user:company_id|strict'
```

#### 4.4 Default Value – `default:value`
If no value is obtained from any source (after applying search priorities), this value is used as a fallback.

**Example**:
```php
'variable' => 'cache:reports.limit|default:1000|cast:int'
```
If the key is not found in the cache, `1000` is used.

#### 4.5 Type Casting – `cast:type`
Ensures that the resolved value is of the required type before being used in the query. Supported types:

| Type | Description |
|------|-------------|
| `int`, `integer` | Cast to integer |
| `float`, `double` | Cast to float |
| `bool`, `boolean` | Cast to boolean (supports `true`/`false`, `1`/`0`, `"1"`/`"0"`) |
| `string` | Cast to string |
| `array` | Cast to array (if not already, it is wrapped in an array) |
| `date` | Validate format `Y-m-d` |
| `datetime` | Validate format `Y-m-d H:i:s` |

**Examples**:
```php
'variable' => 'user:id|cast:int'
'variable' => 'static:DateHelper::getToday|cast:date'
'variable' => 'config:app.debug_mode|cast:bool'
```

#### 4.6 Handling Arrays – `multiple`
If the source returns an array (or you expect a multi-value), use `multiple`. This ensures that the final value is an array, and if it is not, it is converted to an array with a single element. Useful with comparison operators like `in`.

**Example**:
```php
'variable' => 'static:App\StatusHelper::getActiveStatuses|multiple|cast:array|as:allowedStatuses'
```
Assuming the function returns an array of active statuses, the value will be an array and stored in `allowedStatuses`.

---

### 5. Workflow (How Variables are Resolved)

When a report is executed (via `runStoredReport` or `runReport`), the system goes through the following steps:

1. **Extract conditions** from `query_config`.
2. **Pass each condition** to the `resolveDynamicVariables` function inside `ReportsManager`.
3. **For each condition containing a `variable`**:
   - `ValueResolver::resolve($variable, $filterValues)` is called.
   - `ValueResolver` parses the string (source + options) and resolves the value.
   - It returns a `ResolvedValue` object containing the value, success status, error, and an indicator of whether the value came from filters.
4. **Collect errors**: If resolution fails (e.g., `required` with no value), the error is added to an error array.
5. **Update `$filterValues`**: If resolution succeeds and the value was not taken from filters, it is added to `$filterValues` with the appropriate key (considering `as`).
6. **If errors exist**: The report is prevented and an error response is returned.
7. **Otherwise**: The report proceeds using the updated `$filterValues` (which contains the resolved values in addition to user-provided values).

---

### 6. Advanced Practical Examples

#### 6.1 Report Showing Only Current User's Data
```php
'my_orders' => [
    'report_id' => 'my_orders',
    'name'      => 'My Orders',
    'query_config' => [
        'table' => ['name' => 'orders'],
        'columns' => [['name' => '*']],
        'conditions' => [
            [
                'field'    => ['name' => 'user_id'],
                'operator' => ['value' => '='],
                'variable' => 'user:id|required|as:userId'
            ]
        ]
    ]
]
```
- **Behavior**: A user must be logged in; otherwise, the report fails. Orders are automatically filtered by the current user's ID.

#### 6.2 Sales Report with Default Date and Override Capability
```php
'sales_report' => [
    'report_id' => 'sales_report',
    'name'      => 'Sales Report',
    'query_config' => [
        'table' => ['name' => 'order_items'],
        'columns' => [
            ['name' => 'SUM(amount)', 'alias' => 'total']
        ],
        'conditions' => [
            [
                'field'    => ['name' => 'created_at'],
                'operator' => ['value' => '>='],
                'variable' => 'cache:sales.start_date|override|default:2026-01-01|cast:date|as:startDate'
            ],
            [
                'field'    => ['name' => 'created_at'],
                'operator' => ['value' => '<='],
                'variable' => 'static:DateHelper::getToday|required|cast:date|as:endDate'
            ]
        ]
    ],
    'filters' => [
        'startDate' => ['type' => 'date', 'required' => false]
    ]
]
```
- **Behavior**:
  - Start date: tries from cache, can be overridden via `filters['startDate']`, otherwise uses `2026-01-01`.
  - End date: mandatory from a helper function that returns today's date.

#### 6.3 Report with Multi-Value Filter from a Static Function
```php
'products_by_status' => [
    'report_id' => 'products_by_status',
    'name'      => 'Products by Status',
    'query_config' => [
        'table' => ['name' => 'products'],
        'columns' => [['name' => '*']],
        'conditions' => [
            [
                'field'    => ['name' => 'status'],
                'operator' => ['value' => 'in'],
                'variable' => 'static:App\ProductStatus::getActive|multiple|cast:array|as:activeStatuses'
            ]
        ]
    ]
]
```
- **Behavior**: The `getActive` function returns an array of active statuses (e.g., `['active', 'pending']`). It is used with the `in` operator.

#### 6.4 Strict Report Requiring Company Existence
```php
'company_report' => [
    'report_id' => 'company_report',
    'name'      => 'Company Report',
    'query_config' => [
        'table' => ['name' => 'transactions'],
        'columns' => [['name' => '*']],
        'conditions' => [
            [
                'field'    => ['name' => 'company_id'],
                'operator' => ['value' => '='],
                'variable' => 'user:company_id|strict|as:companyId'
            ]
        ]
    ]
]
```
- **Behavior**: If the current user does not have a `company_id` (i.e., does not belong to a company), the report fails immediately even if the condition is not mandatory.

---

## 7. Table of Available Options in the Dynamic Variable (`variable`)

The following table shows all options that can be used within a `variable` string, explaining the function of each option with a simple example.

| Option | Function | Example | Description |
|--------|----------|---------|-------------|
| `as:key` | Store value with a custom key | `as:userId` | Stores the resolved value in the `$filterValues` array using the key `key` instead of the variable's original string. |
| `override` | Priority to filters | `override` | If a value exists in `filters` with the same key (considering `as`), it is used directly. Otherwise, the source is resolved. |
| `fallback` | Priority to source then filters | `fallback` | Tries to resolve the source first. If it fails, it looks in `filters` for the value. |
| `source` | Source only (default behavior) | `source` | Tries to resolve the source only. If it fails, it does not look in filters. Can be omitted as it is the default. |
| `required` | Mandatory | `required` | If no value is obtained (neither from source nor from filters according to priority), the report is prevented and an error is returned. |
| `nullable` | Allow null value | `nullable` | Allows the resolved value to be `null` if no value exists. `null` is passed to the condition. |
| `strict` | Immediate failure if source fails | `strict` | If source resolution fails (even if not `required`), the report is prevented. Used to verify the existence of the source itself. |
| `default:value` | Default value | `default:100` | If no value is obtained from any source, this value is used as a fallback. |
| `cast:type` | Type casting | `cast:int` | Casts the value to the specified type (`int`, `float`, `bool`, `string`, `array`, `date`, `datetime`). Ensures query integrity. |
| `multiple` | Ensure the value is an array | `multiple` | Ensures the final value is an array. If it is not an array, it is converted to an array with a single element. Useful with `in` operators. |

### Important Notes:
- Multiple options can be combined by separating them with the `|` symbol; the order does not affect the result.
- The options `override`, `fallback`, and `source` are mutually exclusive; use only one of them. If none are specified, the default behavior is `source`.
- The `as` option is typically used with any of the other options to specify the key under which the value will be stored in `$filterValues`.
- When using `cast:date` or `cast:datetime`, the format is validated; if invalid, the report fails.
- The `multiple` option is very useful when using comparison operators that require an array, such as `in` and `not_in`.

### 8. Best Practices and Tips

1. **Always use `as`** – It gives you meaningful keys in `filters` and makes it easier to track values in subsequent code.
2. **Prefer `required` over `strict` in most cases** – `required` expresses the need for the value itself, while `strict` expresses the need for the source. Choose the most appropriate.
3. **For dates, use `cast:date` or `cast:datetime`** – Ensures the format is valid before it reaches the database.
4. **When using `cache`, specify a `default`** – Protects the report from the key expiring unexpectedly.
5. **Avoid overusing options** – Keep each variable as simple as possible. Excessive complexity makes maintenance difficult.
6. **Test reports with different scenarios** – Logged-in user, not logged-in, default values, values from filters.

---

### 9. Conclusion

You can now build smart, context-aware reports with precise control over variable behavior. The Dynamic Variables system in `Nano2.QueryBuilder.Reporting` provides you with the necessary tools to make your reports more secure and flexible, reducing the need to write custom code for each case. Using advanced options, you can easily cover 99% of real-world business scenarios.

**Always remember**: The true power of this mechanism lies in combining multiple sources and intelligent options while maintaining clarity and readability of the conditions.