
Domic is a library for writing complex UIs for html/javascript in
typescript. It puts the accent on **elegance** and **robustness** ;
all the API is geared towards ease of use, clarity and very strict typing,
leveraging typescript's type inferance system as much as possible.

Its philosophy is that the DOM is a nice thing and that standards
are nowadays respected enough by all the big players to actually
do use them. In domic, there is no virtual-dom, no wrappers around
the DOM API, no template system, just pure DOM (with extra sugar).

We deal with Nodes, and we use the methods that they define,
for which [there is ample documentation](https://developer.mozilla.org/en-US/docs/Web/API).

This project focuses mainly on recent browsers (IE is typically not high on the
priority list). It is not geared to make websites. Rather, it was built
with Single Page Applications in the browser and on mobile in mind.

### Use tsx code to directly create document nodes

TSX is used to produce `Element`s.

You can insert `string`, `number`, `Node`, `Observable<string|number|Node>`
or an array of any of those as children.

```tsx
var something = 'something'
var some_class = 'some classes'

// The result of a TSX expression is Element and can thus
// be appended directly to the DOM.
document.body.appendChild(<div class='myclass'>
  This will print {something}
  <p class={some_class}>...</p>
</div>)
```




### Use observables to bind values to the DOM

Domic's Observable type is not RxJS's Observable. Rxjs defines an Observable
simply as something that can be observed and that will notify observers.
In domic, Observable is a value holder ; it has a value at its creation
and will have until it ceases to exist. All operations on them are synchronous.

Like RxJS however, Observables are subscribed to by observers who then are
notified of changes happening inside them.

In this example, `bind` binds an observable to an input ; whenever the user interacts with
it, the observable's value is `set()` to the value. Inversely, whenever
the observable is `set()` from elsewhere, the input updates to reflect
the changes.

```tsx

var o_number = o(1) // Observable<number>
var o_str = o('') // Observable<string>

document.body.appendChild(<div>
  <p>
    The value here will be incremented everytime we
    press the button: {o_number}
    <button $$={click(ev => o_number.add(1))}>Add 1</button>
  </p>

  <p>This will follow whatever's inside the following input: {o_str}</p>
  <input type='text' $$={bind(o_str)}/>

  <p class={o_str}>This paragraph will have classes that are updated
    whenever o_str changes.
  </p>

  <p class='static-class'></p>
</div>)

```

### Use decorators to manipulate nodes easily


At its simplest, a decorator is a function that takes
an `Element` and returns anything. They are one of the "low-level"
ways of working with the DOM through domic.

The `$$` attribute is reserved to domic and is either a decorator
or an array of decorators.

```tsx
document.body.appendChild(<div $$={[
  onmount(node => console.log(node, 'is now in the document')),
  onunmount(node => console.log(node, 'is not part of the document anymore'))
]}>
  <button $$={click(event => d.remove())}>Click me</button>
  <input $$={on('blur', ev => console.log('blurred...'))}/>
</div>)

document.body.appendChild(d)
```

### Use controllers when decorators are not enough

Controllers can be bound to nodes. They offer a few more
functionnalities than plain decorators. They can ;

- find other controllers on the same node
  or on a parent Node **by type** -- and not by name, to avoid
  collisions.
- observe Observables without risking memory leaks.
- easily run code when their node is being created, inserted into
  the document or removed from it.

Use the `ctrl` decorator to bind them.

```tsx

class MyCtrl extends Controller {

  // You can have several @onmount
  @onmount
  someFunction() { // the function name is irrelevant
    console.log(this.node, 'was inserted')
  }

  @onunmount
  someOtherFunction(node: Element) {
    // this.node is null now, to avoid unnecessary reference
    // holding. It is given as an argument though if needed.
    console.log('the node was removed')
  }

  @onrender
  yetAnotherMethod() {
    console.log(this.node, 'was just created')
  }

  callMe() {
    console.log('I was called')
  }

}

class MyOtherCtrl extends Controller {
  @onmount
  helloWorld() {
    // Will print 'I was called' everytime this controller's node
    // is inserted into the document.
    this.getController(Test).callMe()
  }
}

// ctrl() accepts instances or classes, whichever is most useful then.
document.body.appendChild(<div
  $$={ctrl(new MyCtrl, MyOtherCtrl)}
/>)

```


### Use "verbs" in your TSX code for dynamic display.

`DisplayIf` and `Repeat` are the most basic verbs that end up being
used almost everywhere.

There are others, such as `RepeatScroll` which
repeats items in array until there are enough displayed and waits
for the user to scroll to the bottom before adding more.

In general, verbs represent parts of your document that are subject
to appear/disappear or generally move around.

There is no `<Repeat .../>` component ; TSX creates concrete and
static objects, anything else must be "verbs", functions that
return a `Node` (often `Comment`s) but whose mechanics involve adding
or removing stuff around.

This is a design decision to make the distinction between regular,
displayable elements and dynamic code clear.

```tsx

var o_bool = o(true)
var o_arr = o(['hello', 'how', 'are', 'you'])

document.body.appendChild(<div>

  {DisplayIf(o_bool,
    value => <p>
      The condition is truthy and equals
      {o_bool.tf(val => JSON.stringify(val))}
    </p>,
    <p>
      And now it is false. Note that for DisplayIf
      we don't need to use functions.
    </p>
  )}

  <button $$={click(ev => o_bool.toggle())}>Toggle</button>

  {Repeat(o_arr, o_item => <p>{o_item}</p>)}

</div>)

```


### Create "stateless" components

Just like in React, you can create reusable components defined by simple
functions. `children` is **always** a DocumentFragment for maximum
performance when inserting.

```tsx
interface ButtonAttrs extends BasicAttributes {
  click: (ev: MouseEvent) => any
}

function Button(attr: ButtonAttrs, children: DocumentFragment): Element {
  return <button $$={click(attr.click)}>{children}</button>
}

document.body.appendChild(
  <Button click={ev => console.log('clicked')}>Click Me</Button>
)
```

### Or use component classes

Component Classes allow for packing code in a neater way and offer
the added benefit of finding other components and communicate
with them, as they are Controllers themselves with just an added
`attrs` property and `render()` method.

```tsx

interface TestAttrs extends BasicAttributes {
  title: MaybeObservable<title>
}

class Test extends Component {

  attrs: TestAttrs

  o_str = o('')

  sayHello(s: string) {
    this.o_str.set(`hello ${s}`)
  }

  render(children: DocumentFragment): Element {
    return <div>
      <h3>The title: {attrs.title}</h3>
      {this.o_str}
      {children}
    </div>
  }
}

interface Test2Attrs extends BasicAttributes {
  value: string
}

class Test2 extends Component {

  attrs: Test2Attrs

  // This method will be called as soon as
  // Test2 is inserted into the document.
  @onmount
  lets_have_some_fun() {
    this.getController(Test)
      .sayHello(this.attrs.value)
  }

  render(): Element {
    return <div/>
  }

}

document.body.appendChild(
  <Test title='hello domic !'>
    And the children
    <Test2 value='testing stuff'/>
  </Test>
)

```

### ... and that's pretty much it !

There is of course a lot to explore and discover. Observables are
quite full-fledged and powerful and these examples barely graze
everything domic has to offer.

This is however the nick of this library ; there are no other core
concepts. Everything it does and everything that is built on it
derives from these concepts ;

* TSX code creates **Elements** -- basically anything that gets
  rendered visually. For other node types, use
* **Verbs**, to compose dynamically your documents
* Elements can be decorated by **decorators**,
* or they can have **Controllers** bound to them for more functionality.
* **Observables** can be bound to children, to attributes and used
  generally in an MVVM way.
* **Components** help in code reuse and come in two flavors ; "stateless"
  with just a function and with the **Component** class for more
  involved mechanics.

The rest of this "hands-on" tutorial will now focus a little more deeply
into domic's mechanics. The `API` will show everything but with less explanations.