
### Foreword

> FIXME maybe we should do a "teaser" page somewhere that just
> showcases domic's main strength.

> FIXME Move the TodoMVC example somewhere else.

Domic applications are pure javascript. The base html markup you need is
exclusively what you would put in `<head>` and an empty body, as all the
DOM Nodes will be created by domic.

For now, `require()` is the recommanded way to go, along with packers like
[webpack](https://webpack.github.io/) or [browserify](http://browserify.org/).
All the examples use webpack and npm.

In most examples, it will be assumed that the different objects/functions/classes
that are being used were previously imported ;

```tsx
import {Controller, o, Observable /*, ... */} from 'domic'
```

### Setting up tsconfig.json

For your project to compile correctly and output the correct code, you will
need your tsconfig.json to contain at least the following (the comments
might not work in the json files) ;

```tsx
{
  "compilerOptions": {
    // generally useful for maximum compatibility
    "target": "es5",

    // useful if you're writing a library
    "declaration": true,

    // Use modules through npm
    "module": "commonjs",
    "moduleResolution": "node",

    // the following allow for real, nazi checking, which
    // is a must in all serious projects.
    "strictNullChecks": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,

    // compile <tags> for use with a library that behaves like
    // React, which is domic's case.
    "jsx": "react",

    // transform all JSX calls to use D(...) instead of
    // React.createElement(). D is exported globally by domic if
    // nothing existed.
    "jsxFactory": "D",

    // we use decorators in components
    "experimentalDecorators": true,

    // if you're using domic, chances are that you're going
    // to use the DOM :)
    "lib": ["es6", "dom"]
    },
  "files": [
    "... your files go here"
  ]
}
```

You can use `d` instead of `D` for `"jsxFactory"`, especially if you have
a global variable of the same name. However, you will have to `import {d} from 'domic'`
in every. single. file. Your choice.

Most of our project however use a more consistent tsconfig.json, along the lines of

A good tsconfig.json is all you should need to start with domic. For webpack
examples, you might want to check [domic-seed](https://github.com/domicjs/domic-seed).

> Create domic-seed

For a lot of examples, check the [domic-example](https://github.com/domicjs/domic-examples) which
has several examples of all the things you can do with it.

> Create domic-examples


```tsx
import {setupMounting, BasicAttributes, click, bind, on} from 'domic'

// This step is necessary, as it sets up domic's internal mechanics.
setupMounting(document)

type TodoType = {value: string, completed?: boolean, editing?: boolean}

var o_todo = o('')
var o_todos = o([] as TodoType[])

interface TodoAttributes extends BasicAttributes {
  model: MaybeObservable<TodoType>
}

function Todo(attrs: TodoAttributes): Element {
  var o_model = o(attrs.model)
  return <li class={ {editing: o_model.p('editing')} }>
      <div class='view'>
        <input class='toggle' type='checkbox'
          $$={on('change', function (ev) { o_model.set('editing', this.value) }) }
        />
        <label>{attrs.model.p('value')}</label>
        <button class='destroy' $$={click(e =>
          o_todos.remove(attrs.model.get())
        )}/>
      </div>
    </li>
}

document.appendChild(<section class='todoapp'>
  <header class='header'>
    <h1>todos</h1>
    <input class='new-todo' placeholder='What needs to be done ?'
      autofocus $$={bind(o_todo)}
    />
  </header>
  <section class='main'>
    <input class='toggle-all' type='checkbox'/>
    <ul class='todo-list'>
      {/* This will do the job of watching the o_todos array
        and display all of the todos.
      */}
      {Repeat(o_todos, o_todo => <Todo model={o_todo}/>)}
    </ul>
  </section>
</section>)
```
