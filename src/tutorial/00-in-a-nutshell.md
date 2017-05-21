
This project focuses mainly on recent browsers (IE is typically not high on the
priority list). It is not geared to make websites. Rather, it was built
with Single Page Applications in the browser and on mobile in mind.

It uses typescript **extensively**. As with any typescript library, it is
perfectly usable by javascript users, but this is a remote concern. Great
care is taken on the typing ; most of the time the classes and functions
were built around the type inferer's way of understanding things.

Domic is all about the DOM. The abstraction level is pretty low ; everything
is done to directly manipulate Nodes.

This has some performance implications ; everyone swears by virtual-dom lately,
which is great ! Domic's stance is just to call a cat a cat and to use
`DocumentFragment` everywhere it can, which even if it is not the fastest
is still pretty efficient.

The payoff is in the fact that the model is actually pretty straightforward.
Integrating third party libraries is very simple, especially the ones that
work directly onto the DOM, and they are many.


### Use tsx code to directly create document nodes


You can insert `string`, `number`, `Node` and `Observable<string|number|Node>` as children.

```tsx
var something = 'something'
document.body.appendChild(<div class='myclass'>
  This will print {something}
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
</div>)

document.body.appendChild(d)
```


### Create "stateless" components easily

> FIXME Use them to

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
