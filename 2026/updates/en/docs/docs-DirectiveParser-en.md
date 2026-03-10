## Documentation of `DirectiveParser` Class

This class `DirectiveParser` is designed to parse and apply "directives" to Eloquent queries in Laravel. Its purpose is to enable building dynamic and flexible queries by passing an array containing the method name (e.g., `where`, `whereHas`) and its parameters, while supporting advanced features like replacing dynamic values (from the session or current user) and qualifying columns to avoid ambiguity when using relationships.

---

### Why Do We Need `DirectiveParser`?

In some applications, especially those that build queries based on user input or dynamic rules (such as advanced reporting or filters), it can be useful to represent parts of the query as interpretable arrays. This class interprets these arrays and executes them on the `Builder`, allowing complex queries to be built without writing static code.

---

### Class Structure and Main Methods

#### 1. The `apply` Method
```php
public static function apply(Builder $query, array $directive): Builder
```
A static method to start the application. It creates an instance of the class and calls `applyDirective`. This is a common pattern for providing a simple interface.

#### 2. The `applyDirective` Method
```php
public function applyDirective(Builder $query, array $directive): Builder
```
This is the core method that processes the directive.

**Inputs:**
- `$query`: The current Builder object.
- `$directive`: An array representing the directive. The first element is the Eloquent method name, and the following elements are its parameters.

**Logic:**

- **Extract the method name**: `$method = array_shift($directive);` removes the first element from the array and stores it in `$method`.
- **Handle custom directive classes**: If `$method` contains the word "Directive" and the namespace separator `\` (i.e., it points to a fully qualified class name), it assumes it is a class that implements the `Directive` interface. An instance of this class is created via `app($method)`, and its `apply` method is called. This allows extending the system by adding custom directive classes that implement complex logic.
- **Handle relationship methods (`whereHas`, `orWhereHas`)**: If the method is `whereHas` or `orWhereHas`, it requires special handling because it accepts a Closure as the second parameter.
  - The relationship name (`$relation`) is extracted from the parameters.
  - Then, `$query->$method` is called with the relationship and a Closure that builds a subquery.
  - Inside the Closure, `qualifyDirective` is called to qualify the columns within the sub-directive (prepending the relationship name to unqualified columns), and then the sub-directive is applied to the inner query by calling `applyDirective` again (recursively).
- **General case**: If neither of the above cases apply, it is treated as a normal Eloquent method.
  - The parameters are processed using `parseParameters` to replace any dynamic values (like `session.xxx` or `self.xxx`).
  - If the method exists on the Builder object (`method_exists`), it is called using `$query->$method(...$parameters)`.
  - The result is returned.

---

### The `qualifyDirective` Method
```php
protected function qualifyDirective(array $directive, string $relation): array
```
Its purpose is to avoid ambiguity in column names when using `whereHas`. It prepends the relationship name (as a prefix) to any string element that does not contain a dot (i.e., it is unqualified). Example:
- If the directive is: `['where', 'age', '>', 18]` and the relationship is `posts`, it becomes: `['where', 'posts.age', '>', 18]`.
- This ensures that the column is interpreted in the correct relationship context.

---

### The `parseParameters` Method
```php
protected function parseParameters(array $parameters): array
```
Its purpose is to replace string elements that start with `session.` or `self.` with actual values from the session or the current user.

- **session.xxx**: If the element starts with `session.`, the key (after `session.`) is extracted and `Session::get($sessionKey)` is used to get the value from the session.
- **self.xxx**: If the element starts with `self.`, the key (after `self.`) is extracted and `Auth::user()->{$attributeKey}` is used to get a value from the currently authenticated user.
- **Any other element**: Remains unchanged.

This feature allows building dynamic queries that depend on the request context or user without having to pass values manually.

---

### Illustrative Example

Suppose we have a directive array representing a query to fetch active users who have posts containing the word "laravel" in the title, with an additional condition that the user's age is less than a value stored in the session.

```php
$directives = [
    ['where', 'status', '=', 'active'],
    ['whereHas', 'posts', function ($query) {
        $query->where('title', 'LIKE', '%laravel%');
    }],
    ['where', 'age', '<', 'session.max_age'],
];

$query = User::query();
foreach ($directives as $directive) {
    $query = DirectiveParser::apply($query, $directive);
}
```

Inside `applyDirective`, each directive is processed as follows:
- The first: a normal `where`, parameters are passed through `parseParameters` (no change).
- The second: `whereHas`, the relationship `posts` is extracted, then a Closure is created that calls `qualifyDirective` on the sub-directive (`['where', 'title', 'LIKE', '%laravel%']`), turning it into `['where', 'posts.title', 'LIKE', '%laravel%']` and applying it.
- The third: `where`, `session.max_age` is converted to the actual value from the session via `parseParameters`.

---

### Support for Custom Directives

If we have a custom class like:

```php
namespace App\Directives;

use Nano2\QueryBuilder\Classes\Contracts\Directive;
use Illuminate\Database\Eloquent\Builder;

class ActiveUsersDirective implements Directive
{
    public function apply(Builder $query): Builder
    {
        return $query->where('status', 'active');
    }
}
```

It can be used in directives as follows:

```php
$directive = ['App\Directives\ActiveUsersDirective'];
$query = DirectiveParser::apply($query, $directive);
```

The system will recognize that the first element contains "Directive" and "\", so it will instantiate the class and execute its `apply` method.

---

### Summary

- **Purpose**: Convert directive arrays into dynamic Eloquent queries.
- **Mechanism**: Extract the method name and its parameters, with special handling for `whereHas`, support for replacing values from the session and user, and support for custom directives.
- **Benefits**: 
  - Build flexible queries from structured data (e.g., JSON or user input).
  - Avoid writing static code for every case.
  - Reuse the same directives across different parts of the application.
  - Support extensibility via custom `Directive` classes.

This covers all aspects of the class, from its structure to practical examples.