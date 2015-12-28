# ES5 / ES6 / TypeScript flavored examples

Here are some examples show you how you can make the state of a simple todo application observable,
using either plain objects or classes.
Mobservable ships with TypeScript typings in the package (supported in TypeScript 1.6 and higher).
So `import * as mobservable from "mobservable"` gives access to the strongly typed api without further imports.

## Exposed TypeScript types

The following types can be imported from the `"mobservable"` package to assist you with using strong types (the types will still be infered if needed, even if you don't import them).
The full definitions can be found [here](https://github.com/mweststrate/mobservable/blob/master/src/interfaces.ts)

* ObservableMap
* Lambda (`() => void`).
* IObservable
* IObservableValue
* IObservableArray
* IArrayChange
* IArraySplice
* IObjectChange

## Creating objects

### ES5 / ES6 / TS: Making plain objects observable

```javascript
var todoStore = mobservable.observable({
    todos: [{
          title: 'Find a clean mug',
          completed: true
    }],
    completedCount: function() {
        return this.todos.filter(function(todo) {
            return todo.completed;
        }).length;
    }
});
```

### ES5 constructor functions

If `TodoStore` is a constructor function that is typically invoked using the `new` keyword,
`extendObservable` can be used to add observable properties to the object during creation:

```javascript
function TodoStore() {
    mobservable.extendObservable(this, {
        todos: [{
            title: 'Find a clean mug',
            completed: true
        }],
        completedCount: function() {
            return this.todos.filter(function(todo) {
                return todo.completed;
            }).length;
        }
    });
}
```

### ES6 / TypeScript classes

When using ES6 or TypeScript it is recommended to use the `@observable` decorator to make observable properties or derivations.
Note that getters should be used to define observable functions on a class.This is to keep the type of a derived value consistent between ES5 / ES6 and TypeScript:
Use `store.completedCount` to obtain a derived value; not `store.completedCount()`.
In contrast, `@observable someFunction() {}` will just create an observable reference to `someFunction`, but `someFunction` itself won't become reactive.


```javascript
class TodoStore {
    @observable todos = todos: [{
        title: 'Find a clean mug',
        completed: true
    }]

    @observable get completedCount() {
        return this.todos.filter((todo) => todo.completed).length;
    }
}
```

## React components

In combination with `@observer` decorator from the `mobservable-react` package:

## Components in ES5

```javascript
var MyComponent = observer(React.createClass({
    render: function() {
        return <button onClick={this.onButtonClick}>Hi</button>
    },
    onButtonClick: function(e) {
        // bound function
    }
});
```

If stateless component functions are used:

```javascript
var MyOtherComponent = observer(function(props) {
    return <div>{props.user.name{/div};
});
```

## Components in ES6 / TypeScript

```javascript
@observer class MyComponent extends React.Component {
    render() {
        // Warning: don't use {this.onButtonClick.bind(this)} or {() => this.onButtonClick} !
        return <button onClick={this.onButtonClick}>Hi</button>
    }

    onButtonClick = (e) => {
        // bound function
    }
}
```

If stateless component functions are used:

```javascript
const MyOtherComponent = observer(props => // ..or with destructuring: ({user}) => ..
    <div>{props.user.name{/div}
);
```

Please note that `onButtonClick` is not just a class method, but an instance field which get a bound function.
This is the most convenient and the fastest way to create bound event handlers.
If `{this.onButtonClick.bind(this)}` or `{(e) => this.onButtonClick(e)}` was used instead in the rendering, each render invocation would create a new closures.
That is not only slightly slower in itself, 
but it will also cause the `button` (or any other component) to be always re-rendered because you are effectively passing a new event handler to the button each time `MyComponent` is rendered.  