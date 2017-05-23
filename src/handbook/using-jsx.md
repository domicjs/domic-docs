
### Insertable

Domic defines the `Insertable` type, which defines what can be
inserted as TSX children. The definition is this :

```tsx
export type InsertableSingle = MaybeObservable<string|number|Node|null|undefined>
export type Insertable = InsertableSingle | InsertableSingle[]
```

It means that you can insert as children anything that is a string, a number,
a Node, null or undefined (which render nothing), an Observable of any of
them, or an array of any of them plus Observables.

```tsx

var _ = <div>
  {12 /* valid */}
  {'hello' /* valid */}
  { document.createComment('!') /* valid */ }
  {o('hello') /* valid */}
  {new Date() /* Invalid */}
</div>

```

Basically, if you want to insert anything else you will have to transform
your variables into one of those types.

### Mounting

To achieve its magic, domic relies on the [`MutationObserver` API](https://developer.mozilla.org/en/docs/Web/API/MutationObserver).

Whenever a Node is inserted or removed from the document, domic runs
the `_mount` (or `_unmount`) function defined in `mounting.ts`, which
purpose is for every Node that was inserted or removed __and their children__
to find their controllers and run their corresponding `onmount` and `onunmount`
functions.

Most notably this process ensures that observing an Observable starts
whenever a Node is inserted into the document
and ends whenever it leaves it. This is because the `observe()` decorator
and `Controller#observe()` method internally use the `onmount` / `onunmount`
mechanic.

This is why all projects must call somewhere ;

```tsx
import {setupMounting} from 'domic'
setupMounting(document)
```

Calling the mounting directly on `document` is not necessary ; you can
just call it on the body or even on any node inside it. This would then
restrict the calling of `onmount` and `onunmount` to the nodes subsequently
added or removed from it.

However, calling it directly on `document` could allow you to do that

```tsx
import {setupMounting} from 'domic'
setupMounting(document)

var o_title = o('my title')
document.head.appendChild(<title>{o_title}</title>)
// it is now possible to change the page title by doing
// o_title.set('something') !
```

### Attributes handling

When creating an intrinsic element (in lowercase), all attributes
will be added to the Node in the end. If their values were observables,
they will be updated into the Document whenever they change.

Domic defines some basic attributes for now, but the list is somewhat
incomplete as of now. **It will be made accurate in time**.

On components however, aside from `class`, `style` and `id`, who behave
a little differently than the rest, any declared attribute that is not
forwarded by the component definition is lost. It is the component's
designer job to correctly pass them on (the compiler will throw errors
anyways to inform the user about this problem).

### Special Attributes

Some tsx attributes are handled separately from the rest ; they
are `class`, `style` and `id`.

When declaring a component, its render method can very well return
a call to another component, which may itself do the same, until
at some point an intrinsic element will be called.

To avoid the hassle for components to have to deal with `class`, `id`
or `style` forwarding, domic simply takes them out at the beginning
to apply them directly on the final Element.

A direct consequence of this is that you cannot use `id`, `class` or
`style` as attribute names for your components as they are reserved.

```tsx

function Comp2(attrs: BasicAttributes, children: DocumentFragment): Element {
  return <div class='cls1'>{children}</div>
}

function Comp1(attrs: BasicAttributes, children: DocumentFragment): Element {
  return <Comp2 class='cls2'>{children}</Comp2>
}

var t = <Comp1 id='myid' class='cls3'>Content</Comp1>
// Gives <div class='cls1 cls2 cls3'>Content</div> in the end.
```

### Playing with class

On top of being handled separately, the class attribute also understands
more than just strings. It can receive an `Observable<string>`, an object
which keys are classes and values are `MaybeObservable<boolean>` which
will apply the class when they are true, or an array of all the above.

```tsx

var o_cls = o('someclass')
var o_bool = o(false)
var test = <div class={['base', o_cls, {test: o_bool, test2: true}]}/>
// <div class='base someclass test2'/>
o_bool.set(true)
// <div class='base someclass test test2'/>
o_cls.set('someotherclass')
// <div class='base someotherclass test test2'/>
```

This behaviour comes in really handy when defining widgets that are
meant to be reused. This way they can use their own CSS classes and
the developper using them can add more rules if they wish to without
having to redefine base styles.

```tsx
interface ButtonAttributes extends BasicAttributes {
  click: (ev: MouseEvent) => any
}

function Button(attrs: ButtonAttributes, children: DocumentFragment): Element {
  return <button
    class='domic-material--button'
    type='button'
    $$={click(attrs.click)}
  >
    {children}
  </button>
}

var b = <Button class='my-btn'>Click Me</Button>
// This button will have both classes in the end
// <button class='my-btn domic-material-button' type='button'>Click Me</button>

```

### Playing with style

Style can be either a `MaybeObservable<string>` or an object which keys
are valid [`HTMLElement.style`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)
keys, and which values are `MaybeObservable<...>` of the correct type.

The `string` value is just like regular HTML.

```tsx
var o_size = o('12px')
var test = <div style={ {fontSize: o_size, height: '14px'} }/>
```

> It is not possible to mix the two, unlike with class. A string will
> always destroy whatever is set with an object.

When designing widget libraries, always make sure to use the object
style definition rather than strings to avoid conflicts. Better yet,
use CSS classes :)

### Decorators

Decorators are function that take an `Element` as parameter and
return anything (the return value is ignored).

They are always applied on the resulting `Node`, which means that
applying a decorator on a component will apply it on the `Node` that
it ultimately returned.

Some functions are called decorators (like `click` or `onmount`) when
they actually are functions that return `Decorator`s but need
an argument for it to be useful.

To apply a decorator to the node, use the `$$` attribute which
is of type `Decorator | Decorator[]`.

Domic exports a few convenient decorators out of the box. You can
check the API to see their exact definitions

- `click` to setup listening to click events. It is a convenience
  function that only takes the callback with the `MouseEvent` as
  parameter, the same could be achieved with `on('click', cbk)`
- `clickfix` which is used as-is (it takes no argument) to remove
  the 300ms latency on iOS
- `on` which takes an event type and a callback, which will setup
  the `addEventListener` in a terser way
- `onmount` called whenever the node enters the document
- `onunmount` whenever the node leaves
- `onrender` called only once when `d()` creates the element
- `ctrl` to bind a controller to the Node
- `bind` to bind an Observable to an `<input>`
- `scrollable` to make an Element scrollable, with some work-arounds
  for iOS.

```tsx
var test = <div $$={[
  onmount(node => console.log(node, 'was mounted')),
  scrollable
]}>
  A decorator can be applied on any element or Component
  <Test $$={[clickfix, click(ev => console.log('clicked'))]}/>
</div>
```

Consult the API to see how each of them expects to be called.


### The Fragment Component

In domic, any tsx code sends back an `Element`. `Fragment` is the only
exception to this rule ; it sends back a `DocumentFragment`, which
is **not** an `Element`. It is just there for convenience ;

As convenient as it is, it should only be used for Observable transforms
or to be generally put into verbs which can handle `DocumentFragment`
pretending to be single values.

> Never, ever, use a fragment as a return value of a component ! Classes, style,
> ids and such can **not** be applied on them.

```tsx
import {Fragment as F} from 'domic'

var o_will_change = o(1)
var test = <div>
  {o_will_change.tf(value => <F>
    I have exactly {value} apple{value === 1 ? '' : 's'}
  </F>)}
</div>
```

### The `d` function

The `d` function powers the node creation of domic. It is defined
in `src/domic.ts` and adds itself to `window.D` if nothing was defined.

It follows the signature of `React.createElement` since this is what tsx
transforms the tsx constructs into.

```tsx
var test = <div class='cls'>some text</div>
// ... translates into this if jsxFactory is "D"...
var test = D('div', {class: 'cls'}, 'some text')
```

It is the function that is responsible for actually creating the elements
with `document.createElement()`, instantiate the Components if it wasn't
an instrinsic tag, call the decorators on the node, do the
magic with `style`, `class` and `id` and call the `onrender` decorators
of controllers.

### SVG

SVG Elements are correctly declared by domic as being in the correct
XML namespace. You can thus use SVG tags safely.