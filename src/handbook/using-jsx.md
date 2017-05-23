
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

### Special Attributes

> Talk about class, style and id.

### Decorators

> decorators


### The `d` function

The `d` function powers the node creation of domic. It is defined
in `src/domic.ts` and adds itself to `window.D` if nothing was defined
there for practicity.

It follows the signature of `React.createElement` since this is what tsx
transforms the tsx constructs into.
