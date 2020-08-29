# The Application Instance

## Creating an Instance

Every Vue application starts by creating a new **application instance** with the `createApp` function:

```js
Vue.createApp(/* options */)
```

After the instance is created, we can _mount_ it, passing a container to `mount` method. For example, if we want to mount a Vue application on `<div id="app"></div>`, we should pass `#app`:

```js
Vue.createApp(/* options */).mount('#app')
```

Although not strictly associated with the [MVVM pattern](https://en.wikipedia.org/wiki/Model_View_ViewModel), Vue's design was partly inspired by it. As a convention, we often use the variable `vm` (short for ViewModel) to refer to our instance.

When you create an instance, you pass in an **options object**. The majority of this guide describes how you can use these options to create your desired behavior. For reference, you can also browse the full list of options in the [API reference](../api/options-data.html).

A Vue application consists of a **root instance** created with `createApp`, optionally organized into a tree of nested, reusable components. For example, a `todo` app's component tree might look like this:

```
Root Instance
└─ TodoList
   ├─ TodoItem
   │  ├─ DeleteTodoButton
   │  └─ EditTodoButton
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```

We'll talk about [the component system](component-basics.html) in detail later. For now, just know that all Vue components are also instances, and so accept the same options object.

## Data and Methods

When an instance is created, it adds all the properties found in its `data` to [Vue's **reactivity system**](reactivity.html). When the values of those properties change, the view will "react", updating to match the new values.

```js
// Our data object
const data = { a: 1 }

// The object is added to the root instance
const vm = Vue.createApp({
  data() {
    return data
  }
}).mount('#app')

// Getting the property on the instance
// returns the one from the original data
vm.a === data.a // => true

// Setting the property on the instance
// also affects the original data
vm.a = 2
data.a // => 2
```

When this data changes, the view will re-render. It should be noted that properties in `data` are only **reactive** if they existed when the instance was created. That means if you add a new property, like:

```js
vm.b = 'hi'
```

Then changes to `b` will not trigger any view updates. If you know you'll need a property later, but it starts out empty or non-existent, you'll need to set some initial value. For example:

```js
data() {
  return {
    newTodoText: '',
    visitCount: 0,
    hideCompletedTodos: false,
    todos: [],
    error: null
  }
}
```

The only exception to this being the use of `Object.freeze()`, which prevents existing properties from being changed, which also means the reactivity system can't _track_ changes.

```js
const obj = {
  foo: 'bar'
}

Object.freeze(obj)

const vm = Vue.createApp({
  data() {
    return obj
  }
}).mount('#app')
```

```html
<div id="app">
  <p>{{ foo }}</p>
  <!-- this will no longer update `foo`! -->
  <button v-on:click="foo = 'baz'">Change it</button>
</div>
```

In addition to data properties, instances expose a number of useful instance properties and methods. These are prefixed with `$` to differentiate them from user-defined properties. For example:

```js
const vm = Vue.createApp({
  data() {
    return {
      a: 1
    }
  }
}).mount('#example')

vm.$data.a // => 1
```

In the future, you can consult the [API reference](../api/instance-properties.html) for a full list of instance properties and methods.

## Instance Lifecycle Hooks

Each instance goes through a series of initialization steps when it's created - for example, it needs to set up data observation, compile the template, mount the instance to the DOM, and update the DOM when data changes. Along the way, it also runs functions called **lifecycle hooks**, giving users the opportunity to add their own code at specific stages.

For example, the [created](../api/options-lifecycle-hooks.html#created) hook can be used to run code after an instance is created:

```js
Vue.createApp({
  data() {
    return {
      a: 1
    }
  },
  created() {
    // `this` points to the vm instance
    console.log('a is: ' + this.a) // => "a is: 1"
  }
})
```

There are also other hooks which will be called at different stages of the instance's lifecycle, such as [mounted](../api/options-lifecycle-hooks.html#mounted), [updated](../api/options-lifecycle-hooks.html#updated), and [unmounted](../api/options-lifecycle-hooks.html#unmounted). All lifecycle hooks are called with their `this` context pointing to the current active instance invoking it.

::: tip
Don't use [arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) on an options property or callback, such as `created: () => console.log(this.a)` or `vm.$watch('a', newValue => this.myMethod())`. Since an arrow function doesn't have a `this`, `this` will be treated as any other variable and lexically looked up through parent scopes until found, often resulting in errors such as `Uncaught TypeError: Cannot read property of undefined` or `Uncaught TypeError: this.myMethod is not a function`.
:::

## Lifecycle Diagram

Below is a diagram for the instance lifecycle. You don't need to fully understand everything going on right now, but as you learn and build more, it will be a useful reference.

<img src="/images/lifecycle.png" width="840" height="auto" style="margin: 0px auto; display: block; max-width: 100%;" loading="lazy" alt="Instance lifecycle hooks">
