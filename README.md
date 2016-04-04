React Firebase
==============

React bindings for [Firebase](https://www.firebase.com).

## Installation

React Firebase requires **React 0.14 or later.**

```
npm install --save react-firebase
```

## Usage

### `<Provider firebase>`

Makes a Firebase refence available to the `connect()` calls in the component hierarchy below. Normally, you can’t use `connect()` without wrapping the root component in `<Provider>`.

If you *really* need to, you can manually pass `firebase` as a prop to every `connect()`ed component, but we only recommend to do this for stubbing `firebase` in unit tests, or in non-fully-React codebases. Normally, you should just use `<Provider>`.

#### Props

* `firebase` (*[Firebase](https://www.firebase.com/docs/web/api/firebase/constructor.html)*): A Firebase reference.
* `children` (*ReactElement*) The root of your component hierarchy.

#### Example

##### Vanilla React

```js
const firebase = new Firebase('https://my-firebase.firebaseio.com');

ReactDOM.render(
  <Provider firebase={firebase}>
    <MyRootComponent />
  </Provider>,
  rootEl
)
```

### `connect([mapPropsToSubscriptions], [mapFirebaseToProps], [mergeProps], [options])`

Connects a React component to a Firebase refence.

It does not modify the component class passed to it.  
Instead, it *returns* a new, connected component class, for you to use.

#### Arguments

* [`mapPropsToSubscriptions(props): subscriptions`] \(*Function*): If specified, the component will subscribe to Firebase `change` events. Its result must be a plain object, and it will be merged into the component’s props. Each value must either a path to a location in the firebase or a function with the signature `createQuery(firebase): [Query](https://www.firebase.com/docs/web/api/query)`.

* [`mapFirebaseToProps(firebase, [ownProps]): actionProps`] \(*Function*): If specified, its result must be a plain object where each value is assumed to be a function that performs modifications to the Firebase. If you omit it, the default implementation just injects `firebase` into your component’s props.

* [`mergeProps(stateProps, actionProps, ownProps): props`] \(*Function*): If specified, it is passed the result of `mapPropsToSubscriptions()`, `mapFirebaseToProps()`, and the parent `props`. The plain object you return from it will be passed as props to the wrapped component. You may specify this function to select a slice of the state based on props, or to bind action creators to a particular variable from props. If you omit it, `Object.assign({}, ownProps, stateProps, actionProps)` is used by default.

* [`options`] *(Object)* If specified, further customizes the behavior of the connector.
  * [`pure = true`] *(Boolean)*: If true, implements `shouldComponentUpdate` and shallowly compares the result of `mergeProps`, preventing unnecessary updates, assuming that the component is a “pure” component and does not rely on any input or state other than its props and subscriptions. *Defaults to `true`.*

#### Returns

A React component class that injects subscriptions and actions into your component according to the specified options.

##### Static Properties

* `WrappedComponent` *(Component)*: The original component class passed to `connect()`.

#### Remarks

* It needs to be invoked two times. The first time with its arguments described above, and a second time, with the component: `connect(mapPropsToSubscriptions, mapFirebaseToProps, mergeProps)(MyComponent)`.

* It does not modify the passed React component. It returns a new, connected component, that you should use instead.

#### Examples

> Runnable examples can be found in the [examples folder](examples/).

##### Inject `firebase` and `todos`

  > Note: The value of `todos` is analogous to https://my-firebase.firebaseio.com/todos.

```js
function mapPropsToSubscriptions() {
  return { todos: 'todos' }
}

export default connect(mapPropsToSubscriptions)(TodoApp)
```

#####  Inject `todos` and a function that adds a new todo (`addTodo`)

```js
function mapPropsToSubscriptions() {
  return { todos: 'todos' }
}

function mapFirebaseToProps(firebase) {
  return {
    addTodo: function(todo) {
      firebase.child('todos').push(todo)
    }
  }
}

export default connect(mapPropsToSubscriptions, mapFirebaseToProps)(TodoApp)
```

#####  Inject `todos`, `completedTodos` and a function that completes a todo (`completeTodo`)

```js
function mapPropsToSubscriptions() {
  return {
    todos: 'todos',
    completedTodos: function(firebase) {
      return firebase.child('todos').orderByChild('completed').equalTo(true)
    }
  }
}

function mapFirebaseToProps(firebase) {
  return {
    completeTodo: function(id) {
      firebase.child('todos').child(id).child('completed').set(true)
    }
  }
}

export default connect(mapPropsToSubscriptions, mapFirebaseToProps)(TodoApp)
```

## License

MIT

## Acknowledgements

[`react-redux`](https://github.com/reactjs/react-redux) which this library is heavily inspired by.