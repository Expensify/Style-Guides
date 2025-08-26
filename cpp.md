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

## Querying/changing SQL data

- When querying data from SQL using `DB::read`, for all new code always access the columns by their names (as opposed to by their indexes).

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

- When adding new queries in auth code, always prefer to add the queries to the lib class related to that query, especially if they are writes. 
This allows us to more easily find places where we are updating each of the tables we have and allows us to reuse more code by making it easier to find pieces of code doing the same or similar thing that you need

## Unpacking optionals

When unpacking an optional value, you should use the `.value()` method or `.value_or()` rather than directly dereferencing with `*`

```cpp
const optional<string> myOptional = optional{"Hello world!"};

// Good
const string good = myOptional.value();

// Bad
const string bad = *myOptional;
```

When making assertions about whether or not an optional has a value, use `.has_value()` rather than relying on its implicit truthiness.

```cpp
const optional<string> myOptional = optional{"Hello world!"};

// Bad - implicit conversion of optional to boolean
if (myOptional) {
    doSomething();
}

// Good - clear unpacking of optional
if (myOptional.has_value()) {
    doSomething();
}

// Bad - implicit conversion of optional to boolean
ASSERT_FALSE(myOptional);

// Good - clear unpacking of optional
ASSERT_FALSE(myOptional.has_value());
```

## Prefer early returns

Just like in E/App, we should prefer early returns when it's feasible to do so:

```cpp
void myGoodFunction(const string& name)
{
    // Good, early return
    if (name.empty()) {
        return;
    }
    doSomething(name);
}

void myBadFunction(const string& name)
{
    // Bad, nested functionality
    if (!myStr.empty()) {
        doSomething(name);
    }
}
```

## Using assertions in tests

You should prefer using `EXPECT_` over `ASSERT_` in tests, given that when `EXPECT_` assertion fails the execution of the test doesn't stop. This is useful when working on tests because you don't have to get one working in order to get the next one to run. Moreover, when assertions fail using `EXPECT_` they output shows the assertion that failed with its line number.

You should use `ASSERT_` for assertions that need to pass before the next assertions in the test are run. Said differently, if one assertion depends on another one succeeding, you should use `ASSERT_` on the first one because there's no point in continuing to run the test if the `ASSERT_` fails

```cpp
// Good
ASSERT_TRUE(expression);

// Bad
EXPECT_TRUE(expression);
```

## Casting types

You should use explicit casts over implicit casts. This is because:
* It's clear what type of cast you're doing (the implicit cast can perform any one of the four casts under the hood)
* It's easier to search casts in our codebase with regex
* Benefits from compile-time checking

```cpp
int64_t a = 4;
int64_t b = 3;
double c1;

// Good
c1 = static_cast<double>(a) / static_cast<double>(b);

// Bad
c1 = ((double) a) / ((double) b);
```

## Includes

You should use includes with angle brackets and not with strings.

```cpp
// Good
#include <auth/lib/Account.h>

// Bad
#include "../lib/Account.h"
```


## More on avoiding expensive copying and move semantics

This section outlines common patterns that cause unnecessary JSON copying in C++ and provides solutions to improve performance.

### 1. Use References for Read-Only Operations

When declaring a `JSON::Value` as `const`, it cannot be modified, so we should always use references to avoid unnecessary copies.

#### Examples

**❌ Bad: Creates unnecessary copies**
```cpp
const JSON::Value something = otherThing["someKey"];

for (const JSON::Value item : JSON::ArrayValue(someArray)) {...}

void doSomething(const JSON::Value value) {...}
```

**✅ Good: Uses references**
```cpp
const JSON::Value& something = otherThing["someKey"];

for (const JSON::Value& item : JSON::ArrayValue(someArray)) {...}

void doSomething(const JSON::Value& value) {...}
```

#### Function Return Types Matter

When a function returns a reference like `JSON::Value& SomeClass::getSomething()`:

**❌ Bad: Creates unnecessary copy**
```cpp
const JSON::Value something = someClassInstance.getSomething();
```

**✅ Good: Uses reference**
```cpp
const JSON::Value& something = someClassInstance.getSomething();
```

When a function returns by value like `JSON::Value SomeClass::getSomething()`:

**⚠️ Misleading: Creates temporary object bound to reference**
```cpp
const JSON::Value& something = someClassInstance.getSomething();
```

**✅ Good: Clearly shows copy intention**
```cpp
const JSON::Value something = someClassInstance.getSomething();
```

#### Important Note

These cases are easy to spot when declaring variables as `const` because the content cannot be manipulated. When not dealing with `const`, only copy the value (omit the `&`) when you will be manipulating the object and don't want to modify the source.

**Related PR:** [Example implementation](https://github.com/Expensify/Auth/pull/16628/files)

### 2. Avoid Initializer List Constructor for Large Objects

The `JSON::Value(initializer_list)` constructor always creates copies. For large objects (transactions, reportActions, reports, personalDetails, participants etc.), use assignment with `move()` instead.

#### Examples

**❌ Bad: Forces copies**
```cpp
return JSON::Value({
    {"structuredReportsForOnyx", move(structuredReportsForOnyx)},
    {"structuredReportsNVPsForOnyx", move(structuredReportsNVPsForOnyx)},
    {"lastMessageData", move(lastMessageData)}
});
```

**✅ Good: Avoids copies**
```cpp
JSON::Value returnValue(JSON::OBJECT);
returnValue["structuredReportsForOnyx"] = move(structuredReportsForOnyx);
returnValue["structuredReportsNVPsForOnyx"] = move(structuredReportsNVPsForOnyx);
returnValue["lastMessageData"] = move(lastMessageData);
return returnValue;
```

**Related PR:** [Example implementation](https://github.com/Expensify/Auth/pull/14365)

### 3. Use Move Semantics and Helper Functions

When data is no longer needed after an operation, use `move()` to transfer ownership and avoid copies. Combine with `JSON::Value` helper functions for cleaner code.

#### Examples

The following example assumes `participants` is not used after creating and queueing the onyx updates.

**❌ Bad: Multiple unnecessary copies**
```cpp
const JSON::Value onyxUpdate = OnyxUpdates::createOnyxUpdate(
    OnyxUpdates::METHOD_MERGE,
    OnyxUpdates::COLLECTION_REPORT + to_string(reportID),
    // We use the JSON::Value(initializer_list<pair<const string, Value>> initializerList) copying the `participants`
    JSON::Value(map<string, JSON::Value>{{OnyxUpdates::KEY_REPORT_PARTICIPANTS, participants}})
);
// The initializer_list for list<JSON::Value> is copying `onyxUpdate`
const JSON::Value onyxUpdates = JSON::Value(list<JSON::Value>{onyxUpdate});
// Uses the queueOnyxUpdates overload that receives the updates as `const JSON::Value& onyxUpdates` which will result in a copy
AuthCommand::currentCommand->queueOnyxUpdates(OnyxUpdates::channelTypes::REPORT, to_string(reportID), onyxUpdates);
```

**✅ Good: Zero unnecessary copies**
```cpp
JSON::Value onyxUpdate = OnyxUpdates::createOnyxUpdate(
    OnyxUpdates::METHOD_MERGE,
    OnyxUpdates::COLLECTION_REPORT + to_string(reportID),
    // `move(participants)` to avoid copy and avoid the initializer_list constructor by using the helper function `JSON::Value::singleEntryObject`
    JSON::Value::singleEntryObject(OnyxUpdates::KEY_REPORT_PARTICIPANTS, move(participants))
);
// `move(onyxUpdate)` to avoid copy and the initializer_list constructor by using helper function `JSON::Value::singleItemArray`
JSON::Value onyxUpdates = JSON::Value::singleItemArray(move(onyxUpdate));
// `move(OnyxUpdates)` to avoid copy and use the overload `JSON::Value&& onyxUpdates`
AuthCommand::currentCommand->queueOnyxUpdates(OnyxUpdates::channelTypes::REPORT, to_string(reportID), move(onyxUpdates));
```

#### Available Move-Enabled Functions

These are common functions that have overloads that accept rvalue references to avoid copying:

```cpp
void AuthCommand::queueOnyxUpdates(const OnyxUpdates::channelTypes channelType, const string& channelID, JSON::Value&& updates);
void queueOnyxUpdatesForChannels(const OnyxUpdates::channelTypes channelType, set<string> channelIDs, JSON::Value&& updates, const bool disableSquash = false);
void Value::merge(Value&& v);
void Value::mergeDeep(Value&& v);
static Value singleItemArray(Value&& value);
static Value singleEntryObject(string&& key, Value&& value);
Value(map<string, Value>&& values);
Value& operator=(Value&& v);
```

**⚠️ Important Warning About Move Semantics**

**Never use an object after it has been moved.** Moved objects are in a "valid but unspecified state" and can cause unexpected behavior.

> Unless otherwise specified, all standard library objects that have been moved from are placed in a "valid but unspecified state", meaning the object's class invariants hold (so functions without preconditions, such as the assignment operator, can be safely used on the object after it was moved from).

**Related PR:** [Example implementation](https://github.com/Expensify/Auth/pull/14820)

### 4. Handle Ternary Operations with References Carefully

Assigning ternary operator results to references can cause copies if one side creates a temporary object.

#### Examples

**❌ Bad: Creates copy of `policy["customLists"]` if it exists**
```cpp
const JSON::Value& customLists = policy.hasMember("customLists") ? policy["customLists"] : JSON::Value(JSON::ARRAY);
```

**✅ Good: Uses static instance instead of temporary value**
```cpp
const JSON::Value& customLists = policy.hasMember("customLists") ? policy["customLists"] : JSON::Utils::EMPTY_ARRAY;
```

**✅ Good: Alternative using helper method**
```cpp
const JSON::Value& customLists = policy.getMemberWithDefault("customLists", JSON::Utils::EMPTY_ARRAY);
```

### 5. Optimize Functions Returning by Value

Functions returning by value always create copies, even when assigned to reference variables.

#### The Problem

Consider this function:
```cpp
JSON::Value BankAccountItem::getAdditionalData() const;
```

**❌ Bad: Creates copy even with reference declaration**
```cpp
const JSON::Value& additionalData = bankAccountItem.getAdditionalData();
```

#### Incomplete Solution: Create Reference Overload

Add an overload that returns by reference for read-only access:
```cpp
const JSON::Value& BankAccountItem::getAdditionalData() const;
```

#### ⚠️ Warning: Avoid Dangling References

Returning by reference can be dangerous because it could be used on a temporary object creating a "dangling reference". Dangling references are very evil because they can cause segmentation faults if used just like a pointer pointing to a deleted object.

**❌ Dangerous: Creates dangling reference**
```cpp
// additionalData would be a dangling reference
const JSON::Value& additionalData = auth.bankAccount.get(db, existingBankAccountID).getAdditionalData();
```

#### Safe Solution: Use r-value/l-value Overloads

Define different overloads for temporary and persistent objects:

```cpp
// For temporary objects (r-values): move the value out
JSON::Value BankAccountItem::getAdditionalData() &&;

// For persistent objects (l-values): return reference to avoid copy
const JSON::Value& BankAccountItem::getAdditionalData() const&;
```

With these overloads, the dangerous code above will call the correct overload and avoid dangling references.

**Related PR:** [Example implementation](https://github.com/Expensify/Auth/pull/15772/files#r2157875973) (see overloads added in JSON::Value)

### 6. Optimize Transformer Functions

These are functions that take parameters, transform the data, and return it in a different shape.

#### Case 1: Parameter Passed by Value

```cpp
JSON::Value formatSomething(JSON::Value something);
```

**Improvements:**
1. **If `something` won't be used after the call:** Use `JSON::Value&& something` to enable move semantics
2. **If only parts are needed:** Consider using `const JSON::Value& something` and copy only what's needed

#### Case 2: Parameter Passed by Reference

```cpp
JSON::Value formatSomething(const JSON::Value& something);
```

**Improvement:** If `something` won't be used after the call, use `JSON::Value&& something`:
```cpp
JSON::Value formatSomething(JSON::Value&& something);

// Usage
JSON::Value y = ...;
JSON::Value x = formatSomething(move(y));
// y should not be used after this point
```

#### Case 3: Function Modifies and Returns

**❌ Confusing design:**
```cpp
JSON::Value formatSomething(JSON::Value& something);
```

**✅ Better alternatives:**
```cpp
void formatSomething(JSON::Value& something);           // Modify in-place
JSON::Value formatSomething(JSON::Value&& something);   // Transform and return
```

**Related PRs:**
- [Example 1](https://github.com/Expensify/Auth/pull/16032)
- [Example 2](https://github.com/Expensify/Auth/pull/16416)