PHP Coding Standard
=====================

## Table of Contents

1. [Introduction](#introduction)
1. [Overview](#overview)
1. [Files](#files)
1. [Namespaces](#namespaces)
1. [Class](#class)
	1. [Constants](#constants)
   	1. [Properties](#properties)
   	1. [Methods](#methods)
1. [Control Structures](#control-structures)
1. [Closures](#closures)
1. [Variables](#variables)
1. [SQL](#sql)
1. [HTML](#html)
1. [PHPDocs](#phpdocs)
1. [Blank Lines](#blank-lines)
1. [Checking types](#checking-types)

## Introduction

Our PHP style guide is based on [[PSR-1]] and [[PSR-2]].
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [[RFC 2119]].

[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt
[PSR-0]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md
[PSR-1]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md
[PSR-2]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md
[PSR-4]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md

**[⬆ back to top](#table-of-contents)**

## Overview

- Code MUST use 4 spaces for indenting, not tabs.

> N.b.: Using only spaces, and not mixing spaces with tabs, helps to avoid problems with diffs, patches, history, and
> annotations. The use of spaces also makes it easy to insert fine-grained sub-indentation for inter-line alignment.

- There MUST NOT be a hard limit on line length; the soft limit MUST be 120 characters; lines SHOULD be 80 characters or
less.

- Files MUST use only `<?php` and `<?=` tags.

- Files MUST use only UTF-8 without BOM for PHP code.

- Files SHOULD *either* declare symbols (classes, functions, constants, etc.) *or* cause side-effects (e.g. generate
output, change .ini settings, etc.) but SHOULD NOT do both.

- PHP [keywords] MUST be in lower case.

- Method names and variables use the lowerCamelCase style.

- When calling/referencing classes and methods, their respective case MUST be respected.

[keywords]: http://php.net/manual/en/reserved.keywords.php

**[⬆ back to top](#table-of-contents)**

## Files

### PHP Tags

- The opening tag `<?php` at the top of a file MUST be followed by a blank line.

- PHP code MUST use the long `<?php ?>` tags or the short-echo `<?= ?>` tags; it MUST NOT use the other tag variations.

- The closing `?>` tag MUST be omitted from files containing only PHP.

**[⬆ back to top](#table-of-contents)**


### Character Encoding

- PHP code MUST use only UTF-8 without BOM.
- All PHP files MUST use the Unix LF (linefeed) line ending.
- All PHP files MUST end with a single blank line.

**[⬆ back to top](#table-of-contents)**


### Token alignment

Aligning tokens should be avoided as it rarely aids in readability and often produces inconsistencies and larger diffs
when updating the code.

```php
// Good
$foo = [
    'foo' => 'bar',
    'foobar' => 'foobar',
    'foobarbaz' => 'foobarbaz',
];

// Bad
$foo = [
    'foo'       => 'bar',
    'foobar'    => 'foobar',
    'foobarbaz' => 'foobarbaz',
];
```

**[⬆ back to top](#table-of-contents)**

### Side Effects

A file SHOULD declare new symbols (classes, functions, constants, etc.) and cause no other side effects, or it SHOULD
execute logic with side effects, but SHOULD NOT do both.

The phrase "side effects" means execution of logic not directly related to declaring classes, functions, constants,
etc., *merely from including the file*.

"Side effects" include but are not limited to: generating output, explicit use of `require` or `include`, connecting to
external services, modifying ini settings, emitting errors or exceptions, modifying global or static variables, reading
from or writing to a file, and so on.

The following is an example of a file with both declarations and side effects; i.e, an example of what to avoid:

```php
<?php

// side effect: change ini settings
ini_set('error_reporting', E_ALL);

// side effect: loads a file
include "file.php";

// side effect: generates output
echo "<html>\n";

// declaration
function foo()
{
    // function body
}
```

The following example is of a file that contains declarations without side effects; i.e., an example of what to emulate:

```php
<?php

// declaration
function foo()
{
    // function body
}

// conditional declaration is *not* a side effect
if (! function_exists('bar')) {
    function bar()
    {
        // function body
    }
}
```

**[⬆ back to top](#table-of-contents)**


## Namespaces


- Namespaces  MUST follow [[PSR-4]].

- When present, there MUST be one blank line after the `namespace` declaration.

- When present, all `use` declarations MUST go after the `namespace` declaration.

- There MUST be one `use` keyword per declaration.

- There MUST be one blank line after the `use` block.

- When inside a namespaced class, always declare all the `use`d classes that are outside the namespace. This means not
using the `\` at the beginning of a class name and make all the dependencies explicit.

For example:

```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

// ... additional PHP code ...

```
**[⬆ back to top](#table-of-contents)**

## Class

- The term "class" refers to all classes, interfaces, and traits.
- Classes without a namespace MUST follow [[PSR-0]] : the class name is mapping the directory, using underscores has
directory seperator.
- Class names MUST be declared in `StudlyCaps`.
- Visibility MUST be declared on all properties and methods; `abstract` and  `final` MUST be declared before the
visibility; `static` MUST be declared after the visibility.
- Opening braces for classes MUST go on the next line, and closing braces MUST go on the next line after the body.
- Opening braces for methods MUST go on the next line, and closing braces MUST go on the next line after the body.
- When working with class names, for example in code [like this](https://github.com/Expensify/Web-Expensify/blob/88d1907100d204efb33658335632b3b8f8321b38/lib/Expensiworks/Job.php#L434), you MUST use the `ClassName::class` syntax and never refer to the classes as strings. 
- You SHOULD prefer the use of `instanceof` over `is_a`

**[⬆ back to top](#table-of-contents)**

### Constants

- Class constants MUST be declared in all upper case with underscore separators.
For example:

```php
<?php

namespace Vendor\Model;

class Foo
{
    const VERSION = '1.0';
    const DATE_APPROVED = '2012-06-01';
}
```

**[⬆ back to top](#table-of-contents)**

### Properties

- Visibility MUST be declared on all properties.
- The `var` keyword MUST NOT be used to declare a property.
- There MUST NOT be more than one property declared per statement.
- Property names SHOULD NOT be prefixed with a single underscore to indicate protected or private visibility.
- Property MUST be explicitely defined in the root of the class
- The default value of a property MUST be set explicitly.
- A property declaration looks like the following.
- PHPDoc with the type definition MUST be added on properties

```php
<?php

// Good
class ClassName
{
    /**
     * @var int
     */
    public $foo = 5;
}

// Bad
class ClassName
{
    public function run()
    {
    	$this->foo = 5;
    }
}

```

**[⬆ back to top](#table-of-contents)**

### Methods

- Magic methods MUST NOT be used
- Visibility MUST be declared on all methods.
- Method names SHOULD NOT be prefixed with a single underscore to indicate protected or private visibility.
- Method names MUST NOT be declared with a space after the method name.
- The opening brace MUST go on its own line
- The closing brace MUST go on the next line following the body.
- There MUST NOT be a space after the opening parenthesis
- There MUST NOT be a space before the closing parenthesis.

A method declaration looks like the following. Note the placement of parentheses, commas, spaces, and braces:

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function fooBarBaz($arg1, &$arg2, $arg3 = [])
    {
        // method body
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Method Arguments

- In the argument list, there MUST NOT be a space before each comma, and there MUST be one space after each comma.
- Method arguments with default values MUST go at the end of the argument list.

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function foo($arg1, &$arg2, $arg3 = [])
    {
        // method body
    }
}
```

- Argument lists MAY be split across multiple lines, where each subsequent line is indented once. When doing so, the
first item in the list MUST be on the next line, and there MUST be only one argument per line. When the argument list is
split across multiple lines, the closing parenthesis and opening brace MUST be placed together on their own line with
one space between them.

```php
<?php

namespace Vendor\Package;

class ClassName
{
    public function aVeryLongMethodName(
        ClassTypeHint $arg1,
        &$arg2,
        array $arg3 = []
    ) {
        // method body
    }
}
```

### Type declarations and return type declarations

[Type declarations](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration) and
[return type declarations](http://php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration)
must be used on all new code whenever possible. This includes type hinting for classes, arrays and scalar types too
(int, float, string and bool).

We prefer the [strict](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict)
variation of type hinting, this means that when you add a new file, the php tag must look like this:

```php
<?php

declare(strict_types=1);
```

Example:
```php
<?php

declare(strict_types=1);

function doSomething(int $anInt, float $aFloat, string $aString, bool $aBool, array $anArray, SomeClass $aClass): int
{
...
}
```

This means 3 things:
- If you pass a variable of the wrong type, an error will be raised (luckily phan will tell you too).
- If the function returns a value of the wrong type, an error will be raised.
- This affects to **ALL** methods/functions calls in the file, this includes built-in PHP functions (ie: strpos, date,
et al).

Adding type and return declarations and the `strict_types` directive to existing methods should be done with **extreme**
care and preferrably adding unit tests to it.

**[⬆ back to top](#table-of-contents)**

### Extends Implements Traits

- The `extends` and `implements` keywords MUST be declared on the same line as the class name.
- The opening brace for the class MUST go on its own line
- The closing brace for the class MUST go on the next line after the body.

```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements \ArrayAccess, \Countable
{
    // constants, properties, methods
}
```

Lists of `implements` MAY be split across multiple lines, where each subsequent line is indented once. When doing so,
the first item in the list MUST be on the next line, and there MUST be only one interface per line.

```php
<?php

namespace Vendor\Package;

use FooClass;
use BarClass as Bar;
use OtherVendor\OtherPackage\BazClass;

class ClassName extends ParentClass implements
    \ArrayAccess,
    \Countable,
    \Serializable
{
    // constants, properties, methods
}
```

- Traits MUST be define immediately after the class name

```PHP
class Yop extends Lait
{
	use MyAwesomeTrait;

	public function __construct()
	{

	}
}
```

**[⬆ back to top](#table-of-contents)**

### abstract final static

- When present, the `abstract` and `final` declarations MUST precede the visibility declaration.
- When present, the `static` declaration MUST come after the visibility declaration.

```php
<?php

namespace Vendor\Package;

abstract class ClassName
{
    protected static $foo;

    abstract protected function zim();

    final public static function bar()
    {
        // method body
    }
}
```

**[⬆ back to top](#table-of-contents)**

### Method and Function Calls

When making a method or function call, there MUST NOT be a space between the method or function name and the opening
parenthesis, there MUST NOT be a space after the opening parenthesis, and there MUST NOT be a space before the closing
parenthesis. In the argument list, there MUST NOT be a space before each comma, and there MUST be one space after each
comma.

```php
<?php

bar();
$foo->bar($arg1);
Foo::bar($arg2, $arg3);
```

Argument lists MAY be split across multiple lines, where each subsequent line is indented once. When doing so, the first
item in the list MUST be on the next line, and there MUST be only one argument per line.

```php
<?php

$foo->bar(
    $longArgument,
    $longerArgument,
    $muchLongerArgument
);
```

**[⬆ back to top](#table-of-contents)**

### Avoid Multiple Method Return Types

When defining method signatures, you SHOULD prefer a single return type
over multiple return types. This includes prefering non-nullable types
to nullable types. If the value your method expects to return is not
found, you SHOULD prefer to return the "zero value" for the type (e.g.
`0`, `''`, `[]`) instead of `null`.

This is because if you return more than one type of data, then you need
to add checks to exclude one type of data before passing the return
value to methods that do not support both types. For example, if you
return `int|string`, then you need to add an `if (is_int($value))` or
`if (is_string($value))` to the caller before passing `$value` to another
method that just takes a string. 

There are a few notable exceptions to this:
* If you are modifying an existing method, you MAY keep a nullable or
  complex return type if the caller checks the return value and that
code cannot be easily changed. (i.e. You should avoid a large unrelated
refactor just to change the return type of an existing method.)
* You MAY use a nullable return type if the "zero value" for your
  return type is a valid return value.
* You MAY use a nullable return type if your return type is an object
  and you cannot safely instantiate a "blank" object (e.g. for tests)

For example:

```php
// Good
function getDomainFromTitle(string $title): string
{
    $parts = array_reverse(explode(' ', $title));
    foreach ($parts as $part) {
        if (filter_var($part, FILTER_VALIDATE_DOMAIN, FILTER_FLAG_HOSTNAME)) {
            return $part;
        }
    }
    return '';
}

// Bad
function getDomainFromTitle(string $title): ?string
{
    $parts = array_reverse(explode(' ', $title));
    foreach ($parts as $part) {
        if (filter_var($part, FILTER_VALIDATE_DOMAIN, FILTER_FLAG_HOSTNAME)) {
            return $part;
        }
    }
    return null;
}

// OK
function getUserScore(): ?int
{
    // User's current score could be zero, so `0` is a valid return value.
    if ($this->gameStarted) {
        return $this->score;
    }

    // Return `null` to indicate that game has not started, so user has no score.
    return null;
}
```

**[⬆ back to top](#table-of-contents)**

## Control Structures


The general style rules for control structures are as follows:

- There MUST be one space after the control structure keyword
- There MUST NOT be a space after the opening parenthesis
- There MUST NOT be a space before the closing parenthesis
- There MUST be one space between the closing parenthesis and the opening brace
- The structure body MUST be indented once
- The closing brace MUST be on the next line after the body
- There MUST NOT be variable assignment inside control structures

The body of each structure MUST be enclosed by braces. This standardizes how the structures look, and reduces the
likelihood of introducing errors as new lines get added to the body.

**[⬆ back to top](#table-of-contents)**

### if elseif else

An `if` structure looks like the following. Note the placement of parentheses, spaces, and braces; and that `else` and
`elseif` are on the same line as the closing brace from the earlier body.

```php
<?php

if ($expr1) {
    // if body
} elseif ($expr2) {
    // elseif body
} else {
    // else body;
}
```

The keyword `elseif` SHOULD be used instead of `else if` so that all control keywords look like single words.

- Strict equality testing (`===`) SHOULD be used as often as you can

**[⬆ back to top](#table-of-contents)**

### switch

A `switch` structure looks like the following. Note the placement of parentheses, spaces, and braces.

- The `case` statement MUST be indented once from `switch`, and the `break` keyword (or other terminating keyword) MUST
be indented at the same level as the `case` body.
- There MUST be a comment such as `// no break` when fall-through is intentional in a non-empty `case` body.

```php
<?php

switch ($expr) {
    case 0:
        echo 'First case, with a break';
        break;
    case 1:
        echo 'Second case, which falls through';
        // no break
    case 2:
    case 3:
    case 4:
        echo 'Third case, return instead of break';
        return;
    default:
        echo 'Default case';
        break;
}
```

**[⬆ back to top](#table-of-contents)**

### while

A `while` statement looks like the following. Note the placement of parentheses, spaces, and braces.

```php
<?php

while ($expr) {
    // structure body
}
```

Similarly, a `do while` statement looks like the following. Note the placement of parentheses, spaces, and braces.

```php
<?php

do {
    // structure body;
} while ($expr);
```

**[⬆ back to top](#table-of-contents)**

### for

A `for` statement looks like the following. Note the placement of parentheses, spaces, and braces.

```php
<?php

for ($i = 0; $i < 10; $i++) {
    // for body
}
```

**[⬆ back to top](#table-of-contents)**

### foreach

A `foreach` statement looks like the following. Note the placement of parentheses, spaces, and braces.

```php
<?php

foreach ($iterable as $key => $value) {
    // foreach body
}
```

Don't pass variables by reference in a foreach loop to avoid the strange behavior described [here](https://www.php.net/manual/en/control-structures.foreach.php),
which can cause bugs that are tricky to diagnose. Instead, pass variables by value and aggregate results in a separate variable.

**[⬆ back to top](#table-of-contents)**


### try catch

A `try catch` block looks like the following. Note the placement of parentheses, spaces, and braces.

```php
<?php

try {
    // try body
} catch (FirstExceptionType $e) {
    // catch body
} catch (OtherExceptionType $e) {
    // catch body
}
```

**[⬆ back to top](#table-of-contents)**

### Variable Assignment

Variables MUST NOT be assigned inside of control structures. It is a common error to do `if ($a = 1)` when the intended
is `if ($a == 1)`. To avoid this error, assign the variable outside of the control structure.

```php
// Good
$foo = setValueToOne();
if ($foo) {
    // ...
}

// Bad
if ($foo = setValueToOne()) {
    // ...
}
```

**[⬆ back to top](#table-of-contents)**

## Closures

- Closures MUST be declared with a space after the `function` keyword, and a space before and after the `use` keyword.
- The opening brace MUST go on the same line, and the closing brace MUST go on the next line following the body.
- There MUST NOT be a space after the opening parenthesis of the argument list or variable list, and there MUST NOT be a
space before the closing parenthesis of the argument list or variable list.
- In the argument list and variable list, there MUST NOT be a space before each comma, and there MUST be one space after
each comma.
- Closure arguments with default values MUST go at the end of the argument list.
- A closure declaration looks like the following. Note the placement of parentheses, commas, spaces, and braces:

```php
<?php

$closureWithArgs = function ($arg1, $arg2) {
    // body
};

$closureWithArgsAndVars = function ($arg1, $arg2) use ($var1, $var2) {
    // body
};
```

Argument lists and variable lists MAY be split across multiple lines, where each subsequent line is indented once. When
doing so, the first item in the list MUST be on the next line, and there MUST be only one argument or variable per line.

When the ending list (whether or arguments or variables) is split across multiple lines, the closing parenthesis and
opening brace MUST be placed together on their own line with one space between them.

The following are examples of closures with and without argument lists and variable lists split across multiple lines.

```php
<?php

$longArgs_noVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) {
   // body
};

$noArgs_longVars = function () use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // body
};

$longArgs_longVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // body
};

$longArgs_shortVars = function (
    $longArgument,
    $longerArgument,
    $muchLongerArgument
) use ($var1) {
   // body
};

$shortArgs_longVars = function ($arg) use (
    $longVar1,
    $longerVar2,
    $muchLongerVar3
) {
   // body
};
```

Note that the formatting rules also apply when the closure is used directly in a function or method call as an argument.

```php
<?php

$foo->bar(
    $arg1,
    function ($arg2) use ($var1) {
        // body
    },
    $arg3
);
```

When the closure only has one expression, prefer the use of arrow functions.

```php
<?php

// Preferred
$nums = array_map(fn ($n) => $n * $factor, [1, 2, 3, 4]);

// Not preferred
$nums = array_map(function($n) {
  return $n * $factor;
}, [1, 2, 3, 4]);
```

**[⬆ back to top](#table-of-contents)**


## Variables

- Variable assignement MUST have a space before and after the equal sign

```PHP
$test = 5;
```


### Null

- MUST be lower case
- Use mosty to indicate that something is not instanciated or wrong

### Bool

- Must be lower case

### String

- Always use single quotes `'` if no variables or apostrophes are needed.
- For strings that require apostrophes, we SHOULD prefer enclosure in double quotes over escaping. That makes it easier to search for the string in code.

```php
// Good
$myString = "Alice's car isn't safe to drive home from the shop because it has a cracked fuel tank AND a leaky fuel line."

// Bad
$myString = 'Alice\'s car isn\'t safe to drive home from the shop because it has a cracked fuel tank AND a leaky fuel line.'
```

- If variables are needed, prefer interpolation over concatenation as long as it doesn't hurt readability.
- Complex variable interpolation can be done as follows:

```php
$myString = "$bob is a part of {$policy->getID()}";
```

- When using heredocs, keep the indentation level of the function where the string is defined:
```php
// Bad
function something() {
    $str = <<<STR
        Bad
STR;
}

// Good
function something() {
    $str = <<<STR
        Good
    STR;
}
```

### Number

- PHPDoc can use int or integer

### Array

- Array are not a scalar type, and MUST be type hinted
- Use short notation to create arrays

```php
// Good
$n = [];

// Bad
$b = array();
```

- When building a list that is going to grow in the future, align the items and keep a trailing comma

```PHP
$myList = [
    'a',
    'b',
    'c',
];
```

#### Accessing arrays

- Usage of the internal helper mfunction `ArrayUtils::get()` is deprecated
- Use the [null coalescing operator](http://php.net/manual/de/migration70.new-features.php#migration70.new-features.null-coalesce-op)
to access values in an array (single- or multi-dimensional) while also providing a default fallback

```php
$array = [
    'foo' => [
        'bar' => 'value',
    ],
];

// Deprecated
$value = ArrayUtils::get($array, ['foo', 'bar'], 'default');

// Good
$value = $array['foo']['bar'] ?? 'default';
```

**[⬆ back to top](#table-of-contents)**

## SQL

- Every dynamic parameter of a query MUST be escape before being used, even if it is a constant.
- SQL keywords MUST be UPPER CASE.
- Simple query can be written on one line
- Complex query MUST be exploded on multiple line

```PHP
$query = "SELECT
            name,
	    age,
	    COUNT(*) AS nb
	  FROM receipts r
	  JOIN fun f ON f.id = r.fID
	  WHERE
	    (
		f.id > 5 AND
		f.id < 42
	    ) OR
	    name = 'fred'
	  ORDER BY name DESC;"

```

**[⬆ back to top](#table-of-contents)**

## HTML

- For complex HTML pages, you MUST use a templating engine
- The PHP short tag `<?=` can be used to print $variable
- When adding control structures in templates (if, for, etc) the markup inside them must be indented the same as code
would be.

Good:
```
<p>Some html</p>
<% if (something) { %>
  <p>Yes</p>
<% } else { %>
  <p>No</p>
<% } %>
```
Bad:
```
<p>Some html</p>
<% if (something) { %>
<p>Yes</p>
<% } else { %>
<p>No</p>
<% } %>
```

**[⬆ back to top](#table-of-contents)**

## PHPDocs

- PHP type hinting *must* be used for parameters and return values when on PHP7.
- PHPDocs *should* be avoided if they don't provide any additional useful information.

```php
// Good
public function __construct(Report $report)
{
    $this->report = $report;
}

// Bad
/**
 * Constructor.
 *
 * @param Report $report
 */
public function __construct(Report $report)
{
    $this->report = $report;
}
```

- PHPDocs *must* document all exceptions being thrown only in the first level (read: only exceptions which are being
thrown directly by the method itself) using `@throws` tag

```php
// Good
/**
 * @throws ExpError
 */
public function __construct(Report $report)
{
    throw new ExpError('47A45CA6-72D1-49CF-9031-AF29D58507DF', "Parameter has an invalid value");
}

// Bad
// Do not add this @throws tag just because somewhere down the call tree that given exception might be thrown
/**
 * @throws \GuzzleHttp\Exception\GuzzleException
 */
public function __construct(Report $report)
{
    $intercom = new IntercomClient();
    $intercom->tagUsersByEmail('Tag', ['florian@expensify.com']);
}
```

- When function is deprecated in favor of an other one, the `@deprecated` tag *must* be used
- The `@deprecated` *must* be followed by the replacement function name

```PHP
/**
 * @deprecated Authentication::validateSessionOrRedirect
 */
function ValidateSession()
{
    Log::deprecated("ValidateSession: use Authentication::validateSessionOrRedirect");
    Authentication::validateSessionOrRedirect();
}
```

- Type of the parameters *must* be written before the variable name
- Scalar types *must* be lower case
- The description of a function *must* explain what the function is doing from an external point of view. If you need to
explain how the function is working, add a comment inside the body of the function
- A blank line *must* be added between the description and the list of parameter

```PHP
    /**
     * Sets the currency that a report should use.
     *
     * @param string $currency
     */
    public function setReportCurrency($currency)
    {
        $this->_load();
        $this->_values['outputCurrency'] = $currency;
    }
```

## Blank Lines

- Comments *should* be preceded by a blank line (Unless preceded by a `{`)
- Blocks of code that are not self-explanatory *should* be consolidated and preceded by a useful comment.
- Unnecessary blank lines without comments *should* be avoided. If you add one, that suggests the following block of
code is functionally different and thus can benefit from a comment

```php
// Good
// This is a comment summarizing a block of code
thisIsSomeCode();
thisIsSomeCode();
thisIsSomeCode();

// Blank lines separate blocks of code; each block of code has a comment therefore there should be a comment after each
// blank line.  If it doesn't seem appropriate to have a comment, then it means it probably isn't appropriate to split
// into a new block of code, and thus the blank line should be removed.
moreCode();

// Bad
// This is a comment summarizing a block of code
thisIsSomeCode();
thisIsSomeCode();

thisIsSomeCode();

moreCode();
```

## Checking types

When you have to check the type of a variable, prefer using `instanceof` instead of `is_a` or `get_class`. This is
because is better to reference a class than a string that's a class name.

```php
// Bad
is_a($var, 'Transaction');

// Bad
get_class($var) === 'Transaction';

// Good
$var instanceof Transaction
```

**[⬆ back to top](#table-of-contents)**
