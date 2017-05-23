
Observables are at the very core of Domic and are the foundation
of its dynamic nature. They basically are a box holding a value
that alerts the developper when it changes.

**They are completely synchronous, and their value can always be accessed at all times.**

Special care has been taken to make sure all of their methods
are completely type-checked, even those
that transform their types or access subproperties or array
items.

Note that this is **not** a change detection library. You *must* use
the `set()` method if you want changes to be acted upon.

### Creating an Observable

When creating an observable, you always must give a value. Most of the
time, this value will be enough for typescript to guess what kind of
Observable you're dealing with.

```tsx
var ob = new Observable(4) // Observable<number>
var ob2 = new Observable('hello') // Observable<string>
// As a shorthand, use the o() function
var ob3 = o(false) // Observable<boolean>
var ob4 = o([] as string[]) // Observable<string[]>
```

### Getting and setting its value

Unlike other libraries such as RxJS, domic's observables always
have a value that you can get with the `get()` method.

To set its value, simply use the `set()` method.

```tsx
var a = new Observable(3)
var b = a.get() // 3
a.set(42)
var c = a.get() // 42
```

Observables are generically typed to prevent a wrong data flow in your application.

Domic is built with typescript's strict flags. `null` and `undefined`
are thus checked if you opt-in the strict mode as well.

```tsx
var a = o(1) // a is Observable<number>
a.set('hello') // Error !
a.set(null) // Error !
a.set(4) // OK

var b = o(1 as number | null) // b is Observable<number|null>
b.set(null) // OK</textarea>
```

### Observing value change

At its core, the Observable class provides an `addObserver()` method which takes a callback
as its first argument that will be called with the new value. This method
returns a function that can be used to "unwatch" the changes.

By default, adding an observer **immediately calls it**. See the API to
prevent this behaviour.

> The observing facilities are pretty much opt-in. Anything that happens outside
> a `set()` will not be reported to the observers.

```tsx
var o_str = o('hello')

// prints 'hello'
var unreg = o_str.addObserver(newvalue =>
  console.log(newvalue)
)

// prints 'bye'
o_str.set('bye')

unreg()

// doesn't print now we're unregistered.
o_str.set('hello again')
```

> Using `addObserver()` has another caveat ; as with any Observer pattern
> library, not cleaning up the observers when the observable is not used anymore
> can lead to memory leaks.

To avoid problems, it is recommended to use the `observe` decorator or the
`observe()` method of the `Controller` (or `Component`) class.

These will make it so `addObserver()` is called whenever the
Node is inserted into the document, and the unregister function
whenever it is removed. You can add and remove the Node as much
as you want, it will still work. This is accomplished thanks
to the mounting mechanism.

Note that there is also an `observe()` method on domic-app's `Service` class
which purpose is more or less the same.

```tsx
setupMounting(document.body)

var o_str = o('hello')

// nothing is printed now
var d = <div $$={observe(o_str, value => console.log(value))}>
  ...
</div>

// still nothing is printed
o_str.set('bye')

// doesn't print right now, but will print 'bye' once the mouting
// is executed.
document.body.appendChild(d)

// ... at some point later
document.body.removeChild(d)
// ... and a little later
o_str.set('bye') // will not print anything.
```


### Using them with TS

Observables can be used as children of tsx code. Any change to them
and their value will automatically be updated into the DOM.

Note however that for this to work, <em>mounting</em> needs to be setup
as per the instructions in the previous chapter.

```tsx
var o_content = o('some content')
document.appendChild(<h3>{o_content}</h3>)

// At some point later, it becomes <h3>some other content</h3>
// in the DOM
o_content.set('some other content')
```


Similarily, node attributes can also be observables and updated
whenever one changes.

This example shows the `class` attribute being linked to an observable.
While this will work as intended, read the chapter about special attributes, as `class` is handled differently to allow for
a lot of flexibility.

```tsx
var o_class = o('myclass')
document.appendChild(<h3 class={o_class}>My Title</h3>)

// At some point later...
o_class.set('myotherclass')
```

### The MaybeObservable typ

We often want to be able to use code using both observables or
regular values. Domic provides a type which helps programmers
with this.

Most of the domic functions, methods and components make extensive
use of it.

The `o()` function also has a strong link with `MaybeObservable`.
If given an observable, it will simply return it instead of creating
a new one.

```tsx
type MaybeObservable<T> = T | Observable<T>

// The o() function is defined like so:
function o<T>(arg: MaybeObservable<T>): Observable<T> {
  /* ... */
}

var o_str = o('string') // o_str is now Observable<string>
var o_str2 = o(o_str) // o_str2 === o_str,
```

A variable typed as `MaybeObservable` of an `Insertable` subtype (`number`, `string` and `Node`)
can be inserted into TSX code as-is and domic will handle it.

So will the `observe` decorator and the `Controller#observe()` method,
which are in this way similar to `o()`.

As a rule of thumb when designing components, always try to type your attributes
as `MaybeObservable<...>`, and treat them as pure observables in your code.

```tsx
interface Attrs extends BasicAttributes {
  myattr: MaybeObservable<string>
}

function Test(attrs: Attrs, ch: DocumentFragment): Element {
  return <div $$={observe(attrs.myattr, value =>
    /* value is typed as string here */
  )}>
    <h3>My attribute: {attrs.myattr}</h3>
    {children}
  </div>
}

var o_test = o('hello !')

// Both are valid
var t1 = <Test myattr='some_value'>Content</Test>
var t2 = <Test myattr={o_test}>Content</Test>

// ... later on
o_test.set('bye !')
// will update t2 to <h3>My attribute: bye !</h3>
```

### Playing with properties

`Observable` is not limited to basic types such as `number` or `string`.
When using more complex types, there are several ways of interacting
with the underlying object properties.

To directly set or get a property, use the `get()` and `set()` methods,
this time with a property name parameter.

**These methods are type safe**. They rely on the `keyof` operator introduced
to typescript in version `2.1`.

```tsx
var o_test = o({a: 1, b: 2})
o_test.get('a') // 1

// the the 'b' property to 3
o_test.set('b', 3)
o_test.get('b') // 3

// It is of course also possible to do
o_test.get().b // 3
```

> If you don't use `set(...)` when updating a value
> the observers won't be notified.


```tsx
var o_test = o({a: 1, b: 2})
o_test.addObserver(value => console.log(value))
o_test.set('b', 3) // prints "{a: 1, b: 3}"

// prints nothing, even though the value did change.
o_test.get().b = 4
```

The `p()` method creates a new `Observable` that monitors
a parent's property.

It is typed as `PropObservable<OriginalType, PropertyType>` and will act
**exactly** like `Observable<PropertyType>`.

```tsx
type MyType = {a: number, b: string}

// Observable<MyType>
var o_test = o({a: 1, b: 'hello'} as MyType)

// PropObservable<MyType, string>
var o_b = o_test.p('b')

// It is castable
var o_b2: Observable<string> = o_test.p('b')

// And it is perfectly OK to do this
var t = <Test myattr={o_b}>...</Test>
```

PropObservables can be observed just like Observable. In this
case, the observers only see the updates on the property.

When a `set()` is called on a PropObservable, the
parent's observers are called, even though the parent itself
has not changed value (in a `===` sense).

When a `set()` is called on the parent, the `PropObservable`'s
observers are called as well.

```tsx
var o_test = o({a: 1, b: 'hey'})
var o_b = o_test.p('b')

// prints {a: 1, b: 'hey'} and 'hey'
o_test.addObserver(val, => console.log(val))
o_b.addObserver(val => console.log(val))

// prints {a: 1, b: 'ho'} and 'ho'
o_b.set('ho')

// prints {a: 2, b: 'hulo'} and 'hulo'
o_test.set({a: 2, b: 'hulo'})
```


### Working with array

Observables can also work with arrays. They even have a few methods that deal
exclusively with them.

It is possible to create a `PropObservable<T[], T>`, or to set an array item
using `set()` by using numbers as properties.

```tsx
var o_arr = o([1, 2, 3])
o_arr.set(0, 4) // now equals [4, 2, 3]

// This will now watch the second element
var o_second = o_arr.p(1)
// its type is PropObservable<number[], number>

```

Observables can do a lot more than what was shown here and will be talked about some more in a later chapter.
