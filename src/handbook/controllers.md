
Controllers are classes whose object can be bound to a `Node`.

They are warned whenever said `Node` is inserted into or removed from the Document through
`onmount` and `onunmount` mechanics.

When their `Node` is mounted, they have methods to find other controllers
instances on their `Node` or on their parents. They also provide the `observe()` method,
useful for Observables to observe their value without fearing memory leaks.


### Defining a controller

To create a Controller intended to use with nodes created with tsx,
simply extend the controller class.

You can access `this.node` as an HTMLElement **when it is mounted**. `this.node`
is `null` when unmounted (This is to help javascript with garbage collecting).

If you are planning of assigning the controller to base nodes (not just
HTMLElement), use the BaseController instead. It changes nothing functionnaly ;
`this.node` is just typed as `Node`.

To run code when the node is created, mounted or unmounted, just use the `onrender`,
`onmount` and `onunmount` decorators. All of these decorators take the node as
their first parameter.

```tsx
class MyCtrl extends Controller {

  // Controllers
  constructor(public obs: Observable<string>) { }

  @onrender
  // The method name is irrelevant.
  doStuff() {
    this.observe(obs, value => {
      console.log('my obs has value', value)
    })
  }

  @onmount
  anotherMethod(node: Node) {
    console.log(node, 'is now in the document')
  }

  @onmount
  // There can be as many @onmount as needed. Use this to
  // separate code by theme.
  myMethod(node: Node) {
    // do something !
  }

  @onunmount
  leftTheDocument(node: Node) {
    console.log(node, 'is no longer part of the document')
  }
}
```


### Assigning a controller to a Node

When using tsx, use the `ctrl` decorator to bind a controller to a Node.
You can give it the class or an object instance. However, if the controller
needs arguments, you must give an initialized object.

```tsx
// Both are a valid way of assigning them.
<div $$={ctrl(MyCtrl, MyCtrl2, ...)}/>
<div $$={ctrl(new MyCtrl(arg1, arg2, ...))}/>
```

There is no limit on the number of controllers present on a Node.


### Communicating with other Controllers

When mounted, a controller may find another controller instance and then
manipulate it.

The `getController()` method takes a class as parameter and looks from the
node and upwards to get the object instance.

This method may fail and throw an exception if the controller was not found.

Use `getControllerIfExists(...)` if you want a null value if not found instead
of an exception.

```tsx

class MyCtrl2 {
  @onmount
  findMyCtrl() {
    var mc = this.getController(MyCtrl)
    mc.doSomething(...)
  }
}

```


### Observing an observable in a controller


The `observe()` method should be called *only once*, as it will add callbacks
to `onmount` and `onunmount`.

The best place to call them is thus in an `onrender` callback since it will be
called only once, or in the constructor.


### The Component class

The `Component` class is a subclass of Controller. It thus behaves the same way, with
the following differences ;

* It must define an `attrs` property with the correct attribute type. You can use
  `this.attrs` afterwards with the attributes in them.
* It must define a `render(children: DocumentFragment): Element` method that creates
  its base node
* There is no need to bind it to a node, since it will be bound to the one returned
  by `render()`.

You may override the constructor, but you cannot change its signature as it is instanciated
by `D()` and not by you.

```tsx
interface WidgetAttributes extends BasicAttributes {
  obs: MaybeObservable<string>
}

class Widget extends Component {

  // This is what tells tsx to check for attribute typing.
  attrs: WidgetAttributes

  render(children: DocumentFragment): Element {

    this.observe(this.attrs.obs, value {
      // the render() method is a good place to do the observe()
      // but you can still use @onrender, or even use the constructor if
      // you wish.
      console.log(value)
    })

    return <div class='test'>{children}</div>
  }
}

<Widget obs='hello'>Content</Widget>
```


### The "stateless" Component

Alternatively, you can write function components. They exist to declare components
tersely, but the functions themselves are not controllers.

```tsx
function MyCmp(attrs: BasicAttributes, children: DocumentFragment): Element {
  return <div>{children}</div>
}

<MyCmp>...</MyCmp>
```