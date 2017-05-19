

```tsx
import {setupMounting, BasicAttributes, click, bind, on} from 'domic'

// This step is necessary, as it sets up domic's internal mechanics.
setupMounting(document)

type TodoType = {value: string, completed: boolean}

var o_todo = o('')
var o_todos = o([] as TodoType[])

interface TodoAttributes extends BasicAttributes {
  model: MaybeObservable<TodoType>
}

function Todo(attrs: TodoAttributes): Element {
  var o_editing = o(false)
  return <li class={ {editing: o_editing} }>
      <div class='view'>
        <input class='toggle' type='checkbox'
          $$={on('change', function (ev) { o_editing.set(this.value) }) }
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
