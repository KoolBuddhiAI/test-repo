<script>
  import { onMount, tick } from 'svelte'

  const FILTERS = ['all', 'active', 'completed']

  function loadStorage(key, fallback) {
    try {
      const stored = localStorage.getItem(key)
      return stored ? JSON.parse(stored) : fallback
    } catch {
      return fallback
    }
  }

  let todos = $state(loadStorage('todos', []))
  let filter = $state(loadStorage('filter', 'all'))
  let inputValue = $state('')
  let editingId = $state(null)
  let editValue = $state('')

  let inputEl
  let editEl

  $effect(() => {
    localStorage.setItem('todos', JSON.stringify(todos))
  })

  $effect(() => {
    localStorage.setItem('filter', JSON.stringify(filter))
  })

  onMount(() => {
    inputEl?.focus()
  })

  let filtered = $derived(todos.filter(t => {
    if (filter === 'active') return !t.completed
    if (filter === 'completed') return t.completed
    return true
  }))

  let activeCount = $derived(todos.filter(t => !t.completed).length)
  let completedCount = $derived(todos.filter(t => t.completed).length)
  let allDone = $derived(todos.length > 0 && todos.every(t => t.completed))

  function addTodo(e) {
    e.preventDefault()
    const text = inputValue.trim()
    if (!text) return
    todos = [...todos, { id: Date.now(), text, completed: false }]
    inputValue = ''
  }

  function toggleTodo(id) {
    todos = todos.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
  }

  function deleteTodo(id) {
    todos = todos.filter(t => t.id !== id)
  }

  async function startEdit(todo) {
    editingId = todo.id
    editValue = todo.text
    await tick()
    editEl?.focus()
  }

  function saveEdit(id) {
    const text = editValue.trim()
    if (text) {
      todos = todos.map(t => t.id === id ? { ...t, text } : t)
    }
    editingId = null
  }

  function handleEditKey(e, id) {
    if (e.key === 'Enter') saveEdit(id)
    if (e.key === 'Escape') editingId = null
  }

  function clearCompleted() {
    todos = todos.filter(t => !t.completed)
  }

  function toggleAll() {
    todos = todos.map(t => ({ ...t, completed: !allDone }))
  }
</script>

<div class="app">
  <div class="container">
    <header class="header">
      <h1>todos</h1>
    </header>

    <div class="card">
      <form class="input-row" onsubmit={addTodo}>
        {#if todos.length > 0}
          <button
            type="button"
            class="toggle-all {allDone ? 'active' : ''}"
            onclick={toggleAll}
            aria-label="Toggle all todos"
          >
            &#8964;
          </button>
        {/if}
        <input
          bind:this={inputEl}
          class="todo-input"
          type="text"
          placeholder="What needs to be done?"
          bind:value={inputValue}
        />
      </form>

      {#if filtered.length > 0}
        <ul class="todo-list">
          {#each filtered as todo (todo.id)}
            <li class="todo-item {todo.completed ? 'completed' : ''}">
              <input
                type="checkbox"
                class="todo-check"
                checked={todo.completed}
                onchange={() => toggleTodo(todo.id)}
                id="todo-{todo.id}"
              />
              <label for="todo-{todo.id}" class="check-label"></label>

              {#if editingId === todo.id}
                <input
                  bind:this={editEl}
                  class="edit-input"
                  bind:value={editValue}
                  onblur={() => saveEdit(todo.id)}
                  onkeydown={e => handleEditKey(e, todo.id)}
                />
              {:else}
                <span
                  class="todo-text"
                  ondblclick={() => startEdit(todo)}
                >
                  {todo.text}
                </span>
              {/if}

              <button
                class="delete-btn"
                onclick={() => deleteTodo(todo.id)}
                aria-label="Delete todo"
              >
                &times;
              </button>
            </li>
          {/each}
        </ul>
      {/if}

      {#if todos.length === 0}
        <div class="empty-state">
          <p>No todos yet. Add one above!</p>
        </div>
      {/if}

      {#if todos.length > 0}
        <footer class="footer">
          <span class="count">
            {activeCount} {activeCount === 1 ? 'item' : 'items'} left
          </span>
          <div class="filters">
            {#each FILTERS as f}
              <button
                class="filter-btn {filter === f ? 'active' : ''}"
                onclick={() => filter = f}
              >
                {f.charAt(0).toUpperCase() + f.slice(1)}
              </button>
            {/each}
          </div>
          {#if completedCount > 0}
            <button class="clear-btn" onclick={clearCompleted}>
              Clear completed
            </button>
          {/if}
        </footer>
      {/if}
    </div>
  </div>
</div>
