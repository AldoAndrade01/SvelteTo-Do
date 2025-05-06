<script>
  import { onMount } from 'svelte';

  let newTask = '';
  let tasks = [];
  let filter = 'all'; // 'all', 'active', 'completed'
  let activeSection = 'tareas'; // 'tareas' o 'notas'
  let selectedTask = null;
  let taskNote = '';

  // Cargar tareas desde localStorage al iniciar
  onMount(() => {
    const saved = localStorage.getItem('svelte-todo-tasks');
    tasks = saved ? JSON.parse(saved) : [];
  });

  // Guardar tareas en localStorage cuando cambian
  $: localStorage.setItem('svelte-todo-tasks', JSON.stringify(tasks));

  function addTask() {
    const text = newTask.trim();
    if (text) {
      tasks = [...tasks, { id: Date.now(), text, done: false, note: '' }];
      newTask = '';
    }
  }

  function removeTask(id) {
    tasks = tasks.filter(t => t.id !== id);
    if (selectedTask && selectedTask.id === id) {
      selectedTask = null;
      taskNote = '';
    }
  }

  function toggleDone(id) {
    tasks = tasks.map(task => 
      task.id === id 
        ? { ...task, done: !task.done } 
        : task
    );
    
    // Forzar actualización reactiva
    tasks = [...tasks];
    
    // Guardar en localStorage
    localStorage.setItem('svelte-todo-tasks', JSON.stringify(tasks));
  }

  function selectTask(task) {
    selectedTask = task;
    taskNote = task.note || '';
  }

  function saveNote() {
    if (selectedTask) {
      tasks = tasks.map(t =>
        t.id === selectedTask.id ? { ...t, note: taskNote } : t
      );
      // Actualizar referencia de selectedTask
      selectedTask = tasks.find(t => t.id === selectedTask.id);
    }
  }

  function updateTaskText(id, newText) {
    const text = newText.trim();
    if (text) {
      tasks = tasks.map(t => t.id === id ? { ...t, text } : t);
      if (selectedTask && selectedTask.id === id) {
        selectedTask.text = text;
      }
    }
  }

  function clearCompleted() {
    tasks = tasks.filter(t => !t.done);
    if (selectedTask && selectedTask.done) {
      selectedTask = null;
      taskNote = '';
    }
  }

  $: filteredTasks =
    filter === 'all'
      ? tasks
      : filter === 'active'
      ? tasks.filter(t => !t.done)
      : tasks.filter(t => t.done);

  $: tasksLeft = tasks.filter(t => !t.done).length;
</script>

<main>
  <header class="app-header">
    <div class="header-content">
      <h1 class="animated-title">Svelte Todo List 📌</h1>
      <nav class="main-nav">
        <button class:active={activeSection === 'tareas'} on:click={() => activeSection = 'tareas'}>Tareas</button>
        <button class:active={activeSection === 'notas'} on:click={() => activeSection = 'notas'}>Notas</button>
      </nav>
    </div>
  </header>

  <div class="app-container">
    {#if activeSection === 'tareas'}
      <div class="section-container">
        <h2>Gestión de Tareas</h2>
        <div class="input-container">
          <input
            type="text"
            bind:value={newTask}
            placeholder="Nueva tarea..."
            on:keydown={(e) => e.key === 'Enter' && addTask()}
          />
          <button class="add-btn" on:click={addTask}>Agregar</button>
        </div>

        <div class="filters">
          <button class:active={filter === 'all'} on:click={() => filter = 'all'}>Todas</button>
          <button class:active={filter === 'active'} on:click={() => filter = 'active'}>Activas</button>
          <button class:active={filter === 'completed'} on:click={() => filter = 'completed'}>Completadas</button>
        </div>

        <ul>
          {#each filteredTasks as task (task.id)}
            <li class:completed={task.done}>
              <div class="task-content">
                <input
                  type="checkbox"
                  bind:checked={task.done}
                  on:change={() => toggleDone(task.id)}
                />
                <button 
                  class="complete-btn" 
                  on:click={() => toggleDone(task.id)}
                >
                  {task.done ? '✓' : '○'}
                </button>
                <span>{task.text}</span>
              </div>
              <button class="delete-btn" on:click={() => removeTask(task.id)}>Eliminar</button>
            </li>
          {/each}

          {#if filteredTasks.length === 0}
            <p class="empty-message">
              No hay tareas {filter === 'all' ? '' : filter === 'active' ? 'activas' : 'completadas'}
            </p>
          {/if}
        </ul>

        <div class="todo-footer">
          <span>{tasksLeft} {tasksLeft === 1 ? 'tarea pendiente' : 'tareas pendientes'}</span>
          <button class="clear-btn" on:click={clearCompleted}>Limpiar completadas</button>
        </div>
      </div>
    {:else}
      <div class="section-container">
        <h2>Notas de Tareas</h2>
        <div class="notes-layout">
          <div class="tasks-list">
            <h3>Selecciona una tarea</h3>
            <ul>
              {#each tasks as task (task.id)}
                <li
                  class:selected={selectedTask && selectedTask.id === task.id}
                  class:completed={task.done}
                  on:click={() => selectTask(task)}
                  on:keydown={(e) => (e.key === 'Enter' || e.key === ' ') && selectTask(task)}
                  role="button"
                  tabindex="0"
                >
                  <div class="task-content">
                    <input
                      type="checkbox"
                      bind:checked={task.done}
                      on:change={() => toggleDone(task.id)}
                    />
                    <button 
                      class="complete-btn" 
                      on:click={() => toggleDone(task.id)}
                    >
                      {task.done ? '✓' : '○'}
                    </button>
                    <span>{task.text}</span>
                  </div>
                  {#if task.note}
                    <div class="note-indicator">📝</div>
                  {/if}
                </li>
              {/each}

              {#if tasks.length === 0}
                <p class="empty-message">No hay tareas disponibles</p>
              {/if}
            </ul>
          </div>

          <div class="note-editor">
            {#if selectedTask}
              <div class="edit-task">
                <h3>Editar tarea</h3>
                <input
                  type="text"
                  bind:value={selectedTask.text}
                  on:input={(e) => updateTaskText(selectedTask.id, e.target.value)}
                  placeholder="Texto de la tarea"
                />
              </div>
              <div class="note-area">
                <h3>Nota para: {selectedTask.text}</h3>
                <textarea
                  bind:value={taskNote}
                  placeholder="Escribe una nota para esta tarea..."
                  rows="6"
                ></textarea>
                <button class="save-btn" on:click={saveNote}>Guardar nota</button>
              </div>
            {:else}
              <p class="select-message">Selecciona una tarea para agregar o editar notas</p>
            {/if}
          </div>
        </div>
      </div>
    {/if}
  </div>

  <footer class="app-footer">
    <div class="footer-content">
      <p>© {new Date().getFullYear()} Svelte Todo List - Aplicación de gestión de tareas</p>
    </div>
  </footer>
</main>
