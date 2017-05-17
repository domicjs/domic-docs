
Observables are at the very core of Domic and are the foundation
of its dynamic nature. They basically are a box holding a value
that alerts the developper when it changes.

**They are completely synchronous, and their value can always be accessed at all times.**

Special care has been taken to make sure all of their methods
are completely type-checked, even those
that transform their types or access subproperties or array
items.

As with any Observer pattern implementation, domic's Observables
are potentially subject to memory leaks when not paying attention.
To avoid them, always use the `Controller#observe()`
method or the `observe()` decorator to actually observe
a value. These functions are leak-proof with the following gotcha ;
the observing is tied to the presence of the underlying Node in
the DOM.


### Creating an Observable

<div class='row'><div>
When creating an observable, you always must give a value. Most of the
time, this value will be enough for typescript to guess what kind of
Observable you're dealing with.

While you can use `new Observable(...)` to create an observable,
the `o(/* value */)` function shorthand is actually preferred.
</div>

```typescript
var ob = new Observable(4) // Observable<number>
var ob2 = new Observable('hello') // Observable<string>
// As a shorthand, use the o() function
var ob3 = o(false) // Observable<boolean>
var ob4 = o([] as string[]) // Observable<string[]>
```
</div>

### Getting and setting its value

<div class='row'><div>
Unlike other libraries such as RxJS, domic's observables always
have a value that you can get with the <code>get()</code> method.

To set its value, simply use the <code>set()</code> method.
</div>

```typescript
var a = new Observable(3)
var b = a.get() // 3
a.set(42)
var c = a.get() // 42
```
</div>

<div class='row'><div>
Observables are generically typed to prevent a wrong data flow in your application.

Domic is built with typescript's strict flags. <code>null</code> and <code>undefined</code>
are thus checked if you opt-in the strict mode as well.
</div>

```typescript
var a = o(1) // a is Observable<number>
a.set('hello') // Error !
a.set(null) // Error !
a.set(4) // OK

var b = o(1 as number | null) // b is Observable<number|null>
b.set(null) // OK</textarea>
```
</div>

### Observing value changes

FIXME: talk about the fact that there is an underlying notify()
method and as such changes may not always be what we expect of them.

CAUTION: careful with the underlying value, we can lose updates
if we're using two different Observables on the same underlying value.

Memory leaks !

### Using it with TSX

<div class='row'><div>
Observables can be used as children of tsx code. Any change to them
and their value will automatically be updated into the DOM.

Note however that for this to work, <em>mounting</em> needs to be setup
as per the instructions in the previous chapter.
</div>

```typescript
var o_content = o('some content')
document.appendChild(<h3>{o_content}</h3>)

// At some point later, it becomes <h3>some other content</h3>
// in the DOM
o_content.set('some other content')
```
</div>


<div class='row'><div>
Similarily, node attributes can also be observables and updated
whenever one changes.

This example shows the class attribute being linked to an observable.
While this will work as intended, read the chapter about special attributes,
as the behaviour for class is actually more complex than just using strings.
</div>

```typescript
var o_class = o('myclass')
document.appendChild(<h3 class={o_class}>My Title</h3>)

// At some point later...
o_class.set('myotherclass')
```
</div>

### The MaybeObservable type

### Playing with properties

### Working with arrays

Observables can do a lot more than what was shown here and will be talked about some more in a later chapter.