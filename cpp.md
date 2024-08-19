# C++

Our main style in C++ is to follow every style rule in PHP. This document only describes styles specific to C++.
If you can't find something in here, it means you need to refer to our [PHP](php.md) style.

The only notable difference with the PHP style is that concatenation operators *must* be surrounded by spaces.

## Documentation

- Docs for functions, params and return values *should* be omitted, unless they add useful additional information (eg,
`@param reportID` in the code below is useless because because both the type and the name of the param are in the
signature).
- Docs for methods *must* be in the header files.
- Each parameter *must* be documented using the following format: `* @param paramName description`. The description is
required, otherwise it's just redundant information.
- Return values *must* be documented using the following format: `* @return description`. Similarly, the description is
required, otherwise the docs are pointless.
- All exceptions thrown by a method *must* be documented using the following format: `* @throws InvalidJSONException
[description]`. The description is required if it is not obvious why this exception can be thrown.

Thus, a doc block for a method *must* look like this:

```cpp
// Good
/**
 * Gets a reportNameValuePair's value
 *
 * @param name The name of the reportNameValuePair
 * @return the rNVP's value
 */
static string getValue(SQLite& db, uint64_t reportID, const string& name);

// Bad
/**
 * Gets a reportNameValuePair's value
 *
 * @param db
 * @param reportID
 * @param name     The name of the reportNameValuePair
 * @return the rNVP's value
 */
static string getValue(SQLite& db, uint64_t reportID, const string& name);

// Good
static bool isDeleted(SQLite& db, const int64_t receiptID);

// Bad (unnecessary)
/**
 * Checks if a receipt is deleted
 *
 * @param db
 * @param receiptID
 * @return whether or not the receipt is deleted
 */
static bool isDeleted(SQLite& db, const int64_t receiptID);
```

## Namespace

The `using namespace` statement is *only* allowed on the `std` namespace. That means `std::` does *not have to* be
prefixed everywhere.

## How to define constants / Use of `inline` declaration
Define the constant variable in the header file without the `inline` keyword. There may be cases where using the `inline` specifier is needed but these should be rare and used sparingly. If you're adding `inline` "just so we can define and set the variable in the header file" you are holding it wrong. 

The reason to follow standard cpp practice of declaring the variable in the header file and assigning it in the source file (.cpp) is to avoid unnecessary recompilation if the varible changes. You can see the discussion here https://github.com/Expensify/Auth/pull/10803#discussion_r1640322275


## Pointers and references

The `&` and `*` *must* stick to the type, like this: `const string& var`, `Command* cmd`.

When using these on variables, they must stick to the variable name.

## When to Use References

In general, use references to complex datatypes (including strings!) everywhere that you can. This avoids expensive
memory allocations and copy operations. This is widely done already for strings passed to methods, with `const string&`
being a very commonly used data type. In fact `const` parameters should almost always be passed as references, unless
they may be deleted or changed by another thread while the function they've been passed to is running.

This extends beyond method parameters, though. Another common practice is to do something like:

```
SQResult result;
db.read("SELECT some data FROM some table...");

const string& particularField = result.rows[0][1];
```

This uses a reference to assign a convenient name to an existing piece of data. The compiler can actually optimize this
entire line away, so this adds no performance overhead whatsoever. On the other hand:

```
SQResult result;
db.read("SELECT some data FROM some table...");

string particularField = result.rows[0][1];
```

This doesn't assign a name to `result.rows[0][1]` but rather, duplicates the data stored there. This requires a memory
allocation and a copy operation to initialize the new string, which can be expensive, particularly if the string is
large. This is generally to be avoided, unless you specifically need two copies of them (for instance, because you're
going to modify one but need to keep the original, or because one of them might get deleted outside of your control).

There are, of course, specific reasons to either use, or not use references. For instance, consider the following
function:

```
void getEmailName(string& email) {
    size_t offset = email.find_first_of('@');
    email.resize(offset);
}
```

This takes an existing email address, and chops off everything after (and including) the `@`. This is fast, it requires
no copy nor initialization, but it *requires* passing `email` by reference, so you can operate on the original object,
not a copy.

On the other hand, you're likely to want to keep the whole email and also want the username, in which case, you might
write:

```
string getEmailName(string email) {
    size_t offset = email.find_first_of('@');
    email.resize(offset);
    return email;
}
```

This passes a copy of `email` to `getEmailName`, modifies the copy, and returns the modified value, thus preserving the
original for the caller (this is not the best way to do this particular operation, but illustrates a case for *not*
using a reference parameter). Which of the above functions you might write depends on what you're trying to accomplish,
but in general, use references for large objects if you can, for performance reasons. On the other hand, when passing
around primitives, like integers, there's no performance benefit to using references, because a reference itself takes
up the same amount of memory as the original datatype. In this case, use a reference if you need to modify the original,
or use a copy if you don't.


### On `const` references to temporary objects:

The C++ standard allows this slightly weird behavior:

```
string myFunctionThatMakesAString()
{
    return string("abc123");
}

const string& whatDoesThisReferenceReferTo = myFunctionThatMakesAString();
```

Even though this looks like it refers to an object that should have been destroyed when `myFunctionThatMakesAString`
returned, the compiler allows this, and extends the lifetime of this object to the life of the reference that it's
assigned to. If you're wondering why the compiler does that, it's because old compilers used this as an optimization
trick to avoid copying the return value from `myFunctionThatMakesAString`. However, new compilers don't need to do this,
because they use a technique called "return value optimization" to do the same thing without the weird "reference to
nothing".

Please don't use this mechanism in new code, just create a plain non-reference variable if you're creating a temporary
object in this manner. The compiler will handle the optimization for you.


## const

`const` *should* be used to achieve specific objectives, generally to avoid extraneous copies. A key scenario would be
when passing an object by pointer or reference that shouldn't be modified, rather than by copy. Example:

```cpp
string foo(const string& dontChangeThis)
{
    return dontChangeThis + "bar";
}
```

In this case, the const reference in the parameter to `foo()` assures the caller that this function won't modify the
object being passed in.

Generally it is advised that you prefer to use `const` when creating variables unless you need to modify them, see [https://isocpp.org/wiki/faq/const-correctness](https://isocpp.org/wiki/faq/const-correctness) for more info.

## Default Params vs Function Overloading
When deciding whether to give a method a default param or to instead implement function overloading you should use default params when the types for the method will be the same and a default is easily set. If you need to allow or use different types then using function overloading should be preferred.

Ex:
**Bad** Function Overloading
```cpp
const string myMethod(const string& myParam);

const string myMethod(const string& myParam, const string& anotherParam);
```

Instead in the above case we should prefer a default param like so:

```cpp
const string myMethod(const string& myParam, const string& anotherParam = "");
```

**Good** Function Overloading
```cpp
string quotedString(int i)
{
    return "\"" + to_string(i) + "\"";
}
string quotedString(string s)
{
    return "\"" + s "\"";
}
```

## Querying SQL data

When querying data from SQL using `DB::read`, for all new code always access the columns by their names (as opposed to by their indexes).

Ex:
**Bad**
```cpp
SQResult result;
db.read("SELECT name, value FROM nameValuePairs;", result);
auto name = result[0][0];
auto value = result[0][1];
```

Ex:
**Good**
```
SQResult result;
db.read("SELECT name, value FROM nameValuePairs;", result);
auto name = result[0]["name"];
auto value = result[0]["value"];
```
