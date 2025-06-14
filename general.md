Language Agnostic Coding Standard
=====================

## Table of Contents
1. [Variables](#variables)
1. [Acronyms](#acronyms)
1. [Abbreviations](#abbreviations)
1. [Simplicity](#simplicity)

## Variables
- Variables should always start lowercase and be camelCase (caveat see #Acryonyms)
- Use of full words within names is RECOMMENDED; avoid abbreviations that create ambiguity (e.g. Bad: msg Good: message)

## Acronyms
- Acronyms - should always be UPPERCASE unless it is the beginning of a variable name in which it should follow Variable
styles and be lowercase:

```PHP
   // Good
   public function setUDFs($udfs)
   {
     ...
   }

   // Bad
   public function setUdfs($udfs)
   {
     ...
   }

   // Good
   $udf = 'a';
   $myNVP = null;

   // Bad
   $NVP = 'asdf';
   $yourNvps = '1234';
```

## Abbreviations
- Don't use abbreviations if you can avoid it
- When referencing an ID, you should make "ID" all caps

```php
   public function setReportID($id)
   {
      ...
   }
```

```js
   var policyID = 1234;
```

## Simplicity
As an engineer, you should strive to keep things simple and relevant.

> Don't engineer something in the hope it will be useful for someone someday. Engineer something so that it's useful for
you today.

- Build things that you know will need to be refactored when/if more functionality is desired in the future.
- Don't add functionality that is not in use RIGHT NOW.
- Don't engineer things that you think will have value later.
- Don't abstract anything to be reusable unless it's being actively reused.
- Use positive boolean variables names. Eg: instead of `$shouldIgnoreTriggers` use `$includeTriggers` and negate it when
using the variable.
- Don't use a class when a function will do.
- Do use comments to explain _why_ your code is there; your naming should explain _what_ it does.

```php
// Bad
class Foo
{
   public function __construct($data)
   {
      ...
   }

   public function doSomethingWithBar()
   {
      ...
   }
}
$Foo = new Foo('bar');
$Foo->doSomethingWithBar();

// Good
function doSomethingWithData($data)
{
   ...
}
doSomethingWithData('bar');
```

- Don't add getters and setters when referencing properties has no side effects

```php
// Bad
class Foo
{
   private $bar;

   public function __construct($bar)
   {
      $this->setBar($bar);
   }

   public function setBar($val)
   {
      $this->bar = $val;
   }

   public function getBar()
   {
      return $this->bar;
   }
}
$Foo = new Foo('bar');
echo $Foo->getBar();


// Good
class Foo
{
   // Public var that we don't care if someone has access to, so no getters or setters
   public $bar;

   // Private var that would cause side effects if someone could access it
   private $state;

   public function __construct($bar)
   {
      $this->bar = $bar;
      $this->state = 'initialized';
   }

   public function getState()
   {
      return $this->state;
   }
}
$Foo = new Foo('bar');
echo $Foo->bar;
echo $Foo->getState();
```

**[back to top](#table-of-contents)**
