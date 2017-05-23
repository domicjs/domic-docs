
### Insertable

> string, number, Node, Observable of all the above.

### Mounting

> mounting

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
