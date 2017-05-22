
This project focuses mainly on recent browsers (IE is typically not high on the
priority list). It is not geared to make websites. Rather, it was built
with Single Page Applications in the browser and on mobile in mind.

It uses typescript **extensively**. As with any typescript library, it is
perfectly usable by javascript users. Javascript usage is however
not a focus. Great care is taken on the typing ; most of the time the classes and functions
were built around the type inferer's way of understanding things and
are designed to really help the developper in highlighting errors
as soon as possible.

Domic is all about the DOM. The abstraction level is pretty low ; everything
is done to help directly playing with document Nodes.

### Use tsx code to directly create document nodes

TSX is used to produce `Element`s.

You can insert `string`, `number`, `Node`, `Observable<string|number|Node>`
or an array of any of those as children.

```tsx
var something = 'something'
var some_class = 'some classes'
document.body.appendChild(<div class='myclass'>
  This will print {something}
  <p class={some_class}>...</p>
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


### Use observables to bind values to the DOM

`bind()` and `click()` are just examples of the many useful decorators
shipped with domic.

`bind` binds an observable to an input ; whenever the user interacts with
it, the observable's value is `set()` to the value. Inversely, whenever
the observable is `set()` from elsewhere, the input updates to reflect
the changes.

The `MaybeObservable<T> = T | Observable<T>` is used everywhere for
attributes for maximum flexibility.

```tsx

var o_number = o(1)
var o_str = o('')

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
with them.

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
  rendered visually. For other types, use
* **Verbs**, to compose dynamically your documents
* These Elements can be decorated by **decorators**
* **Observables** can be bound to children, to attributes and used
  generally in an MVVM way.
* **Components** help in code reuse and come in two flavors ; "stateless"
  with just a function and with the **Component** class for more
  involved mechanics.

The rest of this "hands-on" tutorial will now focus a little more deeply
into domic's mechanics. The `API` will show everything but with less explanations.