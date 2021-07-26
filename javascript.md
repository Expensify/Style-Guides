# JavaScript Coding Standards

For almost all of our code style rules, refer to the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript).

When writing ES6 or React code (supported in files with a `.jsx` extension), please also refer to the
[Airbnb React/JSX Style Guide](https://github.com/airbnb/javascript/tree/master/react).

There are a few things that we have customized for our tastes which will take presidence over Airbnb's guide.

## Functions
- Always wrap the function expression for immediately-invoked function expressions (IIFE) in parens:

    ```javascript
    // Bad
    (function () {
        console.log('Welcome to the Internet. Please follow me.');
    }());

    // Good
    (function () {
        console.log('Welcome to the Internet. Please follow me.');
    })();
    ```

## Whitespace
- Use soft tabs set to 4 spaces.

    ```javascript
    // Bad
    function () {
    ∙∙const name;
    }

    // Bad
    function () {
    ∙const name;
    }

    // Good
    function () {
    ∙∙∙∙const name;
    }
    ```

- Place 1 space before the function keyword and the opening paren for anonymous functions. This does not count for named
functions.

    ```javascript
    // Bad
    function() {
        ...
    }

    // Bad
    function getValue (element) {
        ...
    }

    // Good
    function∙() {
        ...
    }

    // Good
    function getValue(element) {
        ...
    }
    ```

- Do not add spaces inside curly braces.

    ```javascript
    // Bad
    const foo = { clark: 'kent' };

    // Good
    const foo = {clark: 'kent'};
    ```
- Aligning tokens should be avoided as it rarely aids in readability and often produces inconsistencies and larger diffs
when updating the code.

    ```javascript
    // Good
    const foo = {
        foo: 'bar',
        foobar: 'foobar',
        foobarbaz: 'foobarbaz',
    };

    // Bad
    const foo = {
        foo      : 'bar',
        foobar   : 'foobar',
        foobarbaz: 'foobarbaz',
    };
    ```

## Naming Conventions
- When you have an event handler, do not prefix it with "on" or "handle". The method should be named for what it does,
not what it handles. This promotes code reuse by minimizing assumptions that a method must be called in a certain
fashion (eg. only as an event handler).
- One exception for allowing the prefix of "on" is when it is used for callback `props` of a React component. Using it
in this way helps to distinguish callbacks from public component methods.

    ```javascript
    // Bad
    var onSubmitClick = function () {
        // Validate form items and submit form
    };
    $('.submit').click(onSubmitClick);

    // Good
    var validateAndSubmit = function () {
        // Validate form items and submit form
    };
    $('.submit').click(validateAndSubmit);
    ```

- When saving a reference to `this` use `self`.

    ```javascript
    // Bad
    function () {
        var _this = this;
        return function () {
            console.log(_this);
        };
    }

    // Bad
    function () {
        var that = this;
        return function () {
            console.log(that);
        };
    }

    // Good
    function () {
        var self = this;
        return function () {
            console.log(self);
        };
    }
    ```

## JSDocs
- Avoid docs that don't add any additional information.
- Always document parameters and return values.
- Method descriptions are optional and should be added when it's not obvious what the purpose of the method is.
- Use uppercase when referring to JS primitive values (e.g. `Boolean` not `bool`, `Number` not `int`, etc)

```
// Bad
/**
 * Populates the shortcut modal
 * @param {bool} shouldShowAdvancedShortcuts whether to show advanced shortcuts
 */
function populateShortcutModal(shouldShowAdvancedShortcuts) {
}

// Good
/**
 * @param {Boolean} shouldShowAdvancedShortcuts
 */
function populateShortcutModal(shouldShowAdvancedShortcuts) {
}
```

## Destructuring
JavaScript destructuring is convenient and fun, but we should avoid using it in situations where it reduces code
clarity. Here are some general guidelines on destructuring.

**General Guidelines**

- Avoid object destructuring for a single variable that you only use *once*. It's clearer to use dot notation for
accessing a single variable.

```javascript
// Bad
const {data} = event.data;

// Good
const {name, accountID, email} = data;
```

**React Components**

- Avoid destructuring props and state at the *same time*. It makes the source of a given variable unclear.
- Avoid destructuring either props or state when there are other variables declared in a render block. This helps us
quickly know which variables are from props, state, or declared inside the render.
- Use parameter destructuring for stateless function components when there are no additional variable declarations in
the render.

```javascript
// Bad
render() {
	const {userData} = this.props;
	const {firstName, lastName} = this.state;
	...
}

// Good
const UserInfo = ({name, email}) => (
	<div>
		<p>Name: {name}</p>
		<p>Email: {email}</p>
	</div>
);
```

# JavaScript bits of no-ledge

## Module Pattern

When you hear "module pattern" in the context of JavaScript think singleton pattern. The way singletons or modules are
usually implemented in JS is by using a plain JS object. Typically we set variables and functions as properties of the
object so you won't see prototype anywhere in a module. Variables are prefixed with `_` since they're usually "private"
to the module and if they're made available to the outside world it's usually through getters and setters
The [PubSub](https://github.com/Expensify/Mobile-Expensify/blob/c778eba328ea7de0f323bd1db99695576b9925aa/app/libs/pubSub.js)
library is a perfect example of this pattern in use.

## Immediately Invoked Function Expression (IIFE)

The name is confusing and the syntax may feel strange if you've never seen it before but this is a very simple concept
and what it allows you to do is declare a function and immediately call it, as the name suggests.

In Expensify this looks like this

```js
(function () {
    // function body
})();
```

and a perfect example of this is `setTimeout()` (on mobile):

```js
this.setTimeout = (function () {
    var id = 0;
    return function (fun, milliseconds) {
        yaplCall('yaplSetTimeout', fun, milliseconds);
        return ++id;
    };
})();
```

We only know of one **good reason** to use IIFEs and that is to **take advantage of closures**.

What the code above does is define a function that returns a closure and immediately invoke the function. You can think
of a closure like a box with two things in it: a function and a context where the context is just a collection of
variables that are available to the function. In the case above the IIFE defines (and invokes) a function that returns
a closure: the inner function + the variable `id`.

Closures are used in JS to emulate private variables. In JS there's no such thing as private variables (surprise!) but
by leveraging closures you can emulate private variables and this is illustrated in the `setTimeout` example above where
the variable id is inaccessible from outside of that function.

Note that we **do not recommend using IIFEs to emulate private variables** in Expensify's code bases because the
convention of prefixing private variables with an underscore serves the same purpose and allows you to access the
variables from outside the function **for debugging purposes**: think about accessing these from the Safari / Chrome
console while debugging our iOS or web app.

For the specific case of `setTimeout()`, creating a module seemed like overkill for this and we need `setTimeout()` to
return a number but we don't think we're ever gonna need to inspect the value of `id`.

Also note that IIFEs make no sense in a modular JS environment since you export only what you want to expose and
everything else remains private to the module.

Last, we'd advise against using it for dependency injection unless you're trying not to pollute scope since you can
accomplish the same thing with clearer syntax. Consider the following example

```js
// Nothing exciting here, just defining two different app renderers
var ConsoleRenderer = {
    render: function (app) {
        console.log(app.name);
    }
};

var GrowlRenderer = {
    render: function (app) {
        yaplShowGrowl(app.name);
    }
};

// This illustrates the benefits of dependency injection
// as this is the only thing you'd need to change to use a different renderer
var appRenderer = ConsoleRenderer;
// var appRenderer = GrowlRenderer;

// The IIFE way
(function (renderer) {
    var app = {
        name: 'Carlos'
    };

    renderer.render(app);
})(appRenderer);

// The alternative with clearer syntax
// Note that in this case renderApplication is added to the current scope
// (which may well be the global scope)
function renderApplication(renderer) {
    var app = {
        name: 'Carlos'
    };

    renderer.render(app);
}

renderApplication(appRenderer);
```

## Publisher / Subscriber

This one isn't specific to JS but we think it's worth mentioning because

1. We use it heavily on web and mobile.
2. We should be using it more on mobile.
3. We want new hires to understand it.

The publisher / subscriber pattern helps decouple objects from one another by changing the paradigm a bit from having
objects directly call one another (thus coupling one to the public API of the other) to having objects publish (send /
emit) events or messages while others subscribe (react) to them and using an event bus to send and receive these
messages, which in our case is the `PubSub` library.

This is very useful when you have an event or action that is of interest to different parts of the application that are
logically split and in different areas of the code. An example of this would be an event that causes different parts of
the UI owned by different parts of the code to be updated.

On mobile for instance we publish the event `Event.REPORT_LIST_FETCHED` when we load reports from the servers to which
both the expenses page and the reports page subscribe to and redraw the cells in the expenses list (we show the report
name on the expenses list for reported expenses) and reports lists.

Note that if an object is publishing an event to which only that object is subcribed to, it's better to remove the event
entirely and just call the handler directly. Said another way, there's no benefit to having an object adopt the
publisher/subscriber pattern to communicate only with itself.

## Functions as first class citizens

Probably one of the most important concepts (and features!) **to understand** of JS is that functions are first-class
citizens, which is a fancy way of saying that functions can be assigned to variables, passed as function parameters
and returned from functions.

Something we see often is people creating unnecessary anonymous functions and passing them as parameters. Here's an
example of what we mean:

```js
API.BankAccount_Create({...})
  .done(function () {
    return deferred.resolve();
  }).unhandled(function () {
    return deferred.reject();
  });
```

Notice that `deferred.resolve` is a function and the `()` is just an operator to invoke that function but the important
part here is that **`deferred.resolve` is a function**.

So what happens in the example above when the API request succeeds is

```
invoke done callbacks
invoke anonymous function
invoke deferred.resolve
```

Now a similar way of doing this is

```js
API.BankAccount_Create({...})
  .done(deferred.resolve)
  .unhandled(deferred.reject);
```

And the call stack for the successful case would be

```
invoke done callbacks
invoke deferred.resolve
```

While the two solutions are semantically equivalent (given that the wrapping anonymous functions have the same
_contract_ than the ones that aren't wrapped) they are logically different as one introduces a new anonymous function
that in this particular case is unnecessary. This is unlikely to improve performance or affect the code in any way but
we think it's important to understand the difference between the two implementations.

This isn't limited to functions that take no parameters, for example, imagine `API.GetReportSummary` returns a promise
that resolves with a report name and status as two separate strings, that means that the function passed to the done
callback takes two parameters (name, status), so

```js
// This
API.GetReportSummary({reportID: 123})
    .done(function (name, status) {
        logReportSummary(name, status);
    });

// Is (semantically) equivalent to
API.GetReportSummary({reportID: 123})
    .done(logReportSummary);

function logReportSummary(name, status) {
    console.log('Report Name:', name, 'Report Status:', status);
}
```

There are times however where a function that is passed as a parameter to another function depends on its context (ie,
what `this` is bound to), and in that case it's important that `this` is set to the right thing. In the following
example, the last two implementations are (semantically) equivalent since `.bind` returns a function and in the second
example we're just creating an anonymous function on the fly.

```js
// Wrong
deletingExpense.done(this.fetchItems);

// Right
deletingExpense.done(this.fetchItems.bind(this));

// Right and more performant (keep reading)
var self = this;
deletingExpense.done(function () {
    self.fetchItems();
});
```

Note that using Function's `bind` to set the scope of `this` is extremely non-performant since it iterates through every
single property of the object being bound to, which takes a lot of processing especially with large objects. Thus, using
`self` has a very good performance benefit and thus is our preferred convention. A side tangent here is that with ES6,
you can use arrow functions which removes the need for either of these, since arrow functions have a scope dead-zone
with no `this` (using `this` will always reference `this` in the parent scope). With web, you can use any ES6 code if
you give the file a **.jsx** extension in the **/site** directory.


## Named vs Default Exports in ES6 - When to use what?

ES6 provides two ways to export a module from a file: `named export` and `default export`. Which variation to use
depends on how the module will be used, the general idea is:

##### First determine if we only need one conceptual thing from the module, if yes, having one default export is the
way to go.

```
// export
const SomeComponent = () => { ... }
export default SomeComponent;

// import
import DefaultComponent from './DefaultExport';

```
_SideNote: One cool thing about default exports is you can name it whatever you like when importing which helps give
context to how they are being used in the current scope. However, you shouldn't really change the name unless there is
an obvious need to. Here for example we decided to call it `DefaultComponent` when importing._

In our own codebase take an example of the [StepProgressBar](https://github.com/Expensify/expensify-common/blob/e3b7ee4b111a3a370a1427ad904485df4e65a472/lib/components/StepProgressBar.js)
component in `expensify-common`. It only needs to export one single stateless react component. Hence we have used a
default export for the same. `export default StepProgressBar;`

#####  If we want to expose/ breakdown a module's functionality into different parts which may or may-not be used, named exports is the way to go.

```
// exports from ./MyComponent.js file
export const NamedComponent1 = () => { .. }
export const NamedComponent2 = () => { .. }

// importing a single named export
import { NamedComponent1 } from './MyComponent';

// importing multiple named exports
import { NamedComponent1, NamedComponent2 } from './MyComponent';

// giving a named import a different name by using 'as':
import { NamedComponent1 as NewComponent } from './MyComponent';

```

Let's take the same example of [StepProgressBar](https://github.com/Expensify/expensify-common/blob/e3b7ee4b111a3a370a1427ad904485df4e65a472/lib/components/StepProgressBar.js).
You will notice that it has an import statement `import {UI} from '../CONST';`. This is importing a named export called _UI_ from `CONST`. Now if we look at
[CONST](https://github.com/Expensify/expensify-common/blob/e3b7ee4b111a3a370a1427ad904485df4e65a472/lib/CONST.jsx) in
`expensify-common`, we can see it exports _UI_ like so:

```
export const UI = {
    ICON: {
        DELETE: 'trashcan',
        CAR: 'car',
        ...
    },
    ...
```

The advantage of having these named exports in [CONST](https://github.com/Expensify/expensify-common/blob/e3b7ee4b111a3a370a1427ad904485df4e65a472/lib/CONST.jsx)
is that some components like [StepProgressBar](https://github.com/Expensify/expensify-common/blob/e3b7ee4b111a3a370a1427ad904485df4e65a472/lib/components/StepProgressBar.js)
need to only use parts of it and by having named exports we avoid loading unneccessary parts of the library in the
component that doesn't need all of it.

#####  If some file needs all of the module's functionality and some don't need all, then at this point we can do both,
have one default export and multiple named exports. (all big libraries like lodash etc use this pattern.)

```

// exporting from ./MyComponent.js file
export const NamedComponent1 = () => { .. }
export const NamedComponent2 = () => { .. }
export default {
    NamedComponent1,
    NamedComponent2
}

// importing multiple named exports
import { NamedComponent1, NamedComponent2 } from './MyComponent';

// importing default export
import DefaultComponent from './MyComponent';

// using NamedComponent1 from DefaultComponent
const sometest = DefaultComponent.NamedComponent1();

```

## Classes and constructors

#### Class syntax
Using the class syntax is preferred wherever appropriate. Airbnb has clear
[guidelines](https://github.com/airbnb/javascript#classes--constructors) in their JS styleguide which promotes using the
_class_ syntax. Dont manipulate the `prototype` directly. The class syntax is generally considered more concise and
easier to understand.

#### Constructor
Classes have a default constructor if one is not specified. No need to write a constructor function that is empty or
just delegates to a parent class.

```
// bad
class Jedi {
  constructor() {}

  getName() {
    return this.name;
  }
}

// bad
class Rey extends Jedi {
  constructor(...args) {
    super(...args);
  }
}

// good
class Rey extends Jedi {
  constructor(...args) {
    super(...args);
    this.name = 'Rey';
  }
}
```
