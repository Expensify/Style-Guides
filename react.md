# React specific styles

## The Two Rules
There are two rules to abide by when building a React component. These rules:

- Promote code re-use because there are no side-effects (ie. no extra dependencies and nothing done that you don't know
about)
- Make it easier to debug
- Make it easier to maintain
- Prevent bugs

### Rule 1. #DataFlowsDown
_Any_ data that is displayed in a component must come from its props.

#### Things to do:
- Make API requests in the top level controller
- Pass data to a component in its props
- Document all props as strictly as possible in the component's propTypes

#### Things to avoid:
- Making API requests inside a component to get data. Instead, make the API request in the controller and then pass data
to the component through its props.
- Loading data inside a component using global methods like `Policy.getCurrent()`. Pass the policy as a prop instead.

### Rule 2. #InteractionsFlowUp
_Any_ UI interaction that needs to trigger business logic (ie. saving a form value) must be passed up the component
chain via prop callbacks to communicate with the parent component and ultimately the controller.

#### Things to avoid:
- Communicating with something that is not the parent component (eg. API, global libs)
- Putting business logic in a component.

The business logic is the job of the controller. In React only apps there will ideally be high-level components
designated as page controllers in charge of the business logic.

**Note on using PubSub:** PubSub is allowed in some extreme cases. These should be the exception, not the rule.
Including and probably limited to:

- Global libs like the site loader, error messages, alerts, growls
- When two components _need_ to communicate with each other and the chain of nested components is more than 3-4
components deep in each path. This is an exception SOLEY because it can be fairly tedious to drill props down like this
in deep ways. However, please never resort to this as your plan A.

When using PubSub, try to use the `withPubSub` higher order component to make it easy to subscribe and unsubscribe to
events in a React component.

#### Things to do:
- Pass callback handlers to a component's props so that the parent knows when the child component has done something.
(eg. if the child component is a form, then pass a prop called `onSubmit` which is triggered when the form is submitted,
and pass all the form values to that method)
- If an action in a component will be accompanied with a re-rendering of the view, it's best to let the parent handle
this by passing updated props back down to the child (see below about controlled and uncontrolled components)

## Method Naming and Code Documentation
* Prop callbacks should be named for what has happened, not for what is going to happen. Components should never assume
anything about how they will be used (that's the job of whatever is implementing it).

```javascript
// BAD
    propTypes: {
        // A callback to call when we want to save the form
        onSaveForm: React.PropTypes.func.isRequired,
    }
```

```javascript
// GOOD
    propTypes: {
        // A callback to call when the form has been submitted
        onFormSubmitted: React.PropTypes.func.isRequired,
    }
```

* Avoid public methods on components.

```javascript
// BAD
class MyComponent extends React.Component {

    /**
     * Refresh the data in our component by calling `MyComponent.refreshData()`
     *
     * @public
     * @params {Object[]} newData
     */
    refreshData(newData) {
        this.setState({data: newData});
    }
}

ReactDOM.render(<MyComponent ref={el => this.component = el}>, $('#container)[0]);
fetchData().done(newData => this.component.refreshData(newData));
```

```javascript
// GOOD
class MyComponent extends React.Component {
    static get propTypes {
        // The data that will be displayed in our component
        data: React.PropTypes.array.isRequired,
    }
}

function renderComponent(data = []) {
    ReactDOM.render(<MyComponent data={data}>, $('#container)[0]);
}
fetchData().done(newData => renderComponent(newData));
```

* Do not use underscores when naming private methods.
* Do not add method documentation for the built-in [lifecycle methods](https://facebook.github.io/react/docs/component-specs.html)
of a component.
* Add descriptions to all propTypes using a single line comment above the definition. No need to document the types, but
add some context for each property so that other developers understand the intended use.

```javascript
// Bad
    propTypes: {
      currency: React.PropTypes.string.isRequired,
      amount: React.PropTypes.number.isRequired,
      isIgnored: React.PropTypes.bool.isRequired
    }

// Bad
    propTypes: {
        /**
         * The currency that the reward is in
         */
        currency: React.PropTypes.string.isRequired,
        /**
         * The amount of reward
         */
        amount: React.PropTypes.number.isRequired,
        /**
         * If the reward has been ignored or not
         */
        isIgnored: React.PropTypes.bool.isRequired
    }
```

```javascript
// Good
    propTypes: {
        // The currency that the reward is in
        currency: React.PropTypes.string.isRequired,

        // The amount of reward
        amount: React.PropTypes.number.isRequired,

        // If the reward has been ignored or not
        isIgnored: React.PropTypes.bool.isRequired
    }
```

## Inline Ternarys
* Use inline ternary statements when rendering optional pieces of templates. Notice the white space and formatting of
the ternary.

```javascript
// Bad
{
    render() {
        const optionalTitle = this.props.title ? <div className="title">{this.props.title}</div> : null;
        return (
            <div>
                {optionalTitle}
                <div className="body">This is the body</div>
            </div>
        );
    }
}
```

```javascript
// Good
{
    render() {
        return (
            <div>
                {this.props.title ?
                    <div className="title">{this.props.title}</div>
                : null}
                <div className="body">This is the body</div>
            </div>
        );
    }
}
```

```javascript
// Good
{
    render() {
        return (
            <div>
                {this.props.title ?
                    <div className="title">{this.props.title}</div>
                :
                    <div className="title">Default Title</div>
                }
                <div className="body">This is the body</body>
            </div>
        );
    }
}
```

```javascript
// Best
{
    render() {
        return (
            <div>
                {this.props.title && <div className="title">{this.props.title}</div>}
                {!this.props.title && <div className="title">Default Title</div>}

                <div className="body">This is the body</body>
            </div>
        );
    }
}

```

### Important Note:

In React Native, one **must not** attempt to falsey-check a string for an inline ternary. Even if it's in curly braces,
React Native will try to render it as a `<Text>` node and most likely throw an error about trying to render text outside
of a `<Text>` component.

```javascript
// Bad! This will cause a breaking an error on native platforms
{
    render() {
        return (
            <View>
                {this.props.title ?
                    <View style={styles.title}>{this.props.title}</View>
                : null}
                <View style={styles.body}>This is the body</View>
            </View>
        );
    }
}

// Good
{
    render() {
        return (
            <View>
                {!_.isEmpty(this.props.title) ?
                    <View style={styles.title}>{this.props.title}</View>
                : null}
                <View style={styles.body}>This is the body</View>
            </View>
        );
    }
}
```

## Stateless component style

When writing a stateless component you must:
1. Define `propTypes` and `defaultProps` at the top of the file in variables called `propTypes` and `defaultProps`.
2. Assign those variables to the component at the bottom of the file.
3. ALWAYS add a `displayName` property and give it the same value as the name of the component (this is so it appears
properly in the React dev tools).
4. When writing a stateless component in non-modularized code (eg. `web-expensify/site`, or anytime you are adding
something to `React.v` or `React.c`), you must wrap the definition of the component in an
[IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) to remove as much from the global scope as you can.

Good example:

```javascript
/* global React, PersonalDetail */

React.c.AvatarPersonalDetail = (() => {
    const propTypes = {
        // ID to provide in case we are creating more than one
        id: window.PropTypes.string,

        // The text to display for the label
        labelText: window.PropTypes.string,

        // PersonalDetail object to pull values from
        personalDetail: window.PropTypes.instanceOf(PersonalDetail)
    };
    const defaultProps = {
        id: '',
        labelText: '',
        personalDetail: null,
    };

    const AvatarPersonalDetail = props => (
        <li className="notPersonal marginTop">
            <label htmlFor={props.id}>{props.labelText}</label>
            <span className="marginRight5 gravatar size-input-lg round">
                <img alt={props.personalDetail.getDisplayName()} src={props.personalDetail.getAvatarUrl()} />
            </span>
            <span id={props.id} className="marginRight lineHeightInput-lg inlineBlock vAlignTop">
                {props.personalDetail.getDisplayName(true)}
            </span>
        </li>
    );

    AvatarPersonalDetail.propTypes = propTypes;
    AvatarPersonalDetail.defaultProps = defaultProps;
    AvatarPersonalDetail.displayName = 'AvatarPersonalDetail';

    return AvatarPersonalDetail;
})();
```

## Stateless components vs Pure Components vs Class based components vs Render Props - When to use what?

*_1. Stateless components: Used when you don't need to maintain state or use lifecycle methods._*

In many cases we create components that do not need to have a state, lifecycle hooks or any internal variables. In other
words just a dumb component that takes props and renders something presentational. But often times we write them as
class based components which come with a lot of cruft ( for e.g., it has a state, lifecycle hooks and it is a javascript
class which means that React creates instances of it ) that is unnecessary in this situation.

A quote from the React documentation:

> These components must not retain internal state, do not have backing instances, and do not have the component
lifecycle methods. They are pure functional transforms of their input, with zero boilerplate. However, you may still
specify `.propTypes` and `.defaultProps` by setting them as properties on the function, just as you would set them on an
ES6 class.

 - Here is an [example](https://github.com/Expensify/expensify-common/blob/e3b7ee4b111a3a370a1427ad904485df4e65a472/lib/components/StepProgressBar.js#L20)
from our codebase of a stateless component.

```
const StepProgressBar = ({steps, currentStep}) => {
    const currentStepIndex = Math.max(0, _.findIndex(steps, step => step.id === currentStep));
    return (
        <div id="js_steps_progress" className="progress-wrapper">
...

```

*_2. Pure components: Use to improve performance where a component does not need to be rendered too often._*

*IF YOU ARE NOT SURE ABOUT USING React.PureComponent, USE React.Component INSTEAD.* It's very important that you
understand the differences.

By default, a plain React.Component has `shouldComponentUpdate` set to always return true. This is good because it means
React errs on the side of always updating the component in case there's any new data to show. However, it is bad because
it means React might trigger unnecessary re-renders.

React.PureComponent has a default implementation of `shouldComponentUpdate` that does a shallow comparison of props and
state to determine if it should re-render or not.

Read the [React Docs](https://reactjs.org/docs/react-api.html#reactpurecomponent) to understand this more.

```
// Internal react code for pure component looks like this w.r.t shouldComponentUpdate

if (type.prototype && type.prototype.isPureReactComponent) {
    shouldUpdate = !shallowEqual(oldProps, props) ||
                   !shallowEqual(oldState, state);
}
```

> A common pitfall when converting from Component to PureComponent is to forget that the children need to re-render too.
> As with all React - if the parent doesn't re-render the children won't either. So if you have a PureComponent with
> children, those children can only update if the parent's state or props are shallowly different (causing the parent
> to re-render). You can only have a PureComponent parent if you know none of the children should re-render if the
> parent doesn't re-render.

*_3. Class based components: Use it when you need to maintain state and use lifecycle methods._*

Always extend from `React.Component`.

If the component needs some data which cannot be passed as a prop, use class components to get the data. Another rule of
thumb is you should always fetch data which involves API calls, any ajax calls in the lifecycle methods. Never do this
in render!

*_4. Render Props: Use for Cross-Cutting Concerns and Code Reuse_*

Let's say you have two separate components that render two different things, but you want them both to be based on the
same state. You can pass a `render` property to a component to accomplish this. You can read more about how this works
and when to use it in the [React Docs](https://reactjs.org/docs/render-props.html).

## Do not use hooks in React components

*Exception: If a third party library is being implemented and the only interfact they provide is via hooks, then it's OK to use hooks.*

Hooks have been avoided because we don't want to adopt new React features until they are well established and have clear best practices. We were bitten by this previously when we were an early adopter of React mixins. After a couple of years, React decided that mixins had many downfalls and they moved away from using them which left us with a lot of technical debt.

They are also avoided so that we can maintain consistent code styles everywhere and it doesn't come as a surprise on how something should be built. This decreases the barrier of entry to our codebase because you don't need to be an expert in React in order to work on it.

The conversation about using hooks is not final, but as for now, we choose not to have them in our code.

## Do not use inheritance for other React components
```
// Bad
// Extends from our custom components
class AComponent extends CComponent, React.Component { ... }
class BComponent extends CComponent, React.Component { ... }

// Good
const AEnhancedComponent = higherOrderComponent(CComponent);

// Good
// Only extends from React's component, not from our own components
class AComponent extends React.Component { ... }
class BComposedComponent extends React.Component
{

    ...
    render() {
        return (
            <AComponent {...props}>
                <div>
                    {this.state.whatever}
                </div>
            </AComponent>
        )
    }
    ...
}
```

## Composition vs Inheritance

From React's documentation:

> Props and composition give you all the flexibility you need to customize a componentts look and behavior in an
explicit and safe way. Remember that components may accept arbitrary props, including primitive values, React elements,
or functions. If you want to reuse non-UI functionality between components, we suggest extracting it into a separate
JavaScript module. The components may import it and use that function, object, or a class, without extending it.

Use *[Higher order components](https://reactjs.org/docs/higher-order-components.html)* if you find a use case where you
need inheritance.

## Use Refs Appropriately

React's documentation explains refs in [detail](https://reactjs.org/docs/refs-and-the-dom.html). It's important to
understand when to use them and how to use them to avoid bugs and hard to maintain code.

A common mistake with refs is using them to pass data back to a parent component higher up the chain. In most cases, you
can try [lifting state up](https://reactjs.org/docs/lifting-state-up.html) to solve this. 

There are several ways to use and declare refs and we prefer the [callback method](https://reactjs.org/docs/refs-and-the-dom.html#callback-refs). 

## jQuery in React

There are a few cases where jQuery should be used in a React component. There are also times when you should not use
jQuery at all.

#### Things to do:

1. Do use jQuery for global event listeners e.g. on `document` or `window` (sparingly). jQuery offers some conveniences
around global event binding that are more useful and less verbose than their native JS equivalents such as
[namespaced events](https://api.jquery.com/on/). 
1. Do use jQuery in a component that requires [jQuery UI](https://jqueryui.com/) or Bootstrap plugins e.g. modals, date
pickers, tooltips, etc. These all necessarily have dependencies on the global version of jQuery.
1. Do use jQuery when you need to modify or interact with some content that has been set via
[`dangerouslySetInnerHTML`](https://reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml). Where possible, try to
find a better way to do whatever you are trying to do. But if we _must_ use `dangerouslySetInnerHTML` then React has no
way to interact with the DOM at that point. jQuery is likely our best option.
1. Do use jQuery to trigger global animations like page scrolls.

#### Things to avoid:

1. Do not target an HTML element with a jQuery `id` or `class` selector that a React component knows about. Use a `ref`
instead.
1. Do not access HTML elements that exist outside of a React component's immediate control or child view heirarchy
(e.g. to access a `<div>` rendered by a parent component from a child component). Move logic into whatever component
owns the element you are trying to access and/or use [`React.forwardRef`](https://reactjs.org/docs/forwarding-refs.html)
if you need to access an element owned by a child.
1. Do not call `addClass`, `toggleClass`, or `removeClass` on a React component's HTML elements. Use the
[`classnames` library](https://github.com/JedWatson/classnames) instead and set or unset the class based on props or
state.
1. Do not add or remove HTML attributes to rendered elements. If you want an attribute to optionally not appear simply
default it to `undefined` in those cases.
1. Do not use jQuery to attach event listeners to DOM elements that React knows about. Use the built-in events instead
e.g. [`onChange` event](https://reactjs.org/docs/dom-elements.html#onchange).
1. Do not import another instance of jQuery into a modular JS file if that file has a jQuery UI dependency. This can
overwrite `window.$` with an instance of jQuery that is missing the various plugins and dependencies it needs.
1. Avoid using jQuery to perform animations that are easily replaced by CSS. Animations can be triggered by adding and
removing classes or by using [React Transition Group](https://reactcommunity.org/react-transition-group/).
1. Do not get or set an uncontrolled input's value via `$.val()`. Use the input's `ref` instead e.g.
`this.input.value = newValue`.

## Styling

Never put CSS in a react file. Always use classes and put the styles in CSS files.

```
// Bad
<input style={{height: '25px'}} />

// Good
<input className="tallInput" />
```
