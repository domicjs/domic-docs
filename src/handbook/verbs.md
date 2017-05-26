
Verbs are not very different from the component logic. They are
generally just `Comment` nodes to which some controllers are bound
that perform "unusual" operations.

In contrast, Components are elements that are clearly _visible_ in
the document.

They could perfectly have been created by tsx as well (in fact, at the
beginning of designing this library, they were), but a design choice
was taken to handle them differently for a clearer visual distinction.

In fact, this documentation uses a custom typescript highlighter to
make function calls in tsx code starting with upper case characters
a different color for maximum visibility.

```tsx
<div class='myclass'>
  {DisplayIf(o_my_condition, () => <F>
    <h3>My condition is true</h3>
    So we'll repeat some items.
    {Repeat(o_my_array, o_item =>
      <Item model={o_item}>an item</Item>
    )}
  </F>)}
</div>
```

The available verbs won't be described here (see instead the API).

All of them are defined in the following manner ;

```tsx
class SomeVerbCtrl extends BaseController {
  /* ... */
}

function SomeVerb(/* arguments */): Node {
  var node = document.createComment('  someverb  ')
  var svc = new SomeVerbCtrl(/* other arguments */)
  svc.bindToNode(node)
  return node
}

// later on...
<span>{SomeVerb(arg1, arg2)}</span>
```
