<script>
  import { onMount } from 'svelte';
  
  let newTask = "";
  let tasks = [];
  let filter = 'all'; // 'all', 'active', 'completed'
  let activeSection = 'tareas'; // 'tareas' o 'notas'
  let selectedTask = null; // Para la sección de notas
  let taskNote = ""; // Para almacenar la nota de una tarea

  // Cargar tareas desde localStorage al iniciar
  onMount(() => {
    const savedTasks = localStorage.getItem('svelte-todo-tasks');
    if (savedTasks) {
      tasks = JSON.parse(savedTasks);
    }
  });

  // Guardar tareas en localStorage cuando cambian
  $: {
    if (tasks) {
      localStorage.setItem('svelte-todo-tasks', JSON.stringify(tasks));
    }
  }

  const addTask = () => {
    if (newTask.trim() !== "") {
      tasks = [...tasks, { id: Date.now(), text: newTask, done: false, note: "" }];
      newTask = "";
    }
  };

  const removeTask = (index) => {
    tasks = tasks.filter((_, i) => i !== index);
  };

  const toggleDone = (task) => {
    task.done = !task.done;
    tasks = [...tasks]; // Forzar actualización reactiva
  };
  
  const selectTask = (task) => {
    selectedTask = task;
    taskNote = task.note || "";
  };
  
  const saveNote = () => {
    if (selectedTask) {
      selectedTask.note = taskNote;
      tasks = [...tasks]; // Forzar actualización reactiva
    }
  };
  
  const updateTaskText = (task, newText) => {
    if (newText && newText.trim() !== "") {
      task.text = newText;
      tasks = [...tasks]; // Forzar actualización reactiva
    }
  };

  const clearCompleted = () => {
    tasks = tasks.filter(task => !task.done);
  };

  $: filteredTasks = filter === 'all' 
    ? tasks 
    : filter === 'active' 
      ? tasks.filter(task => !task.done) 
      : tasks.filter(task => task.done);

  $: tasksLeft = tasks.filter(task => !task.done).length;
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
            on:keydown={(e) => e.key === "Enter" && addTask()}
          />
          <button class="add-btn" on:click={addTask}>Agregar</button>
        </div>

        <div class="filters">
          <button class:active={filter === 'all'} on:click={() => filter = 'all'}>Todas</button>
          <button class:active={filter === 'active'} on:click={() => filter = 'active'}>Activas</button>
          <button class:active={filter === 'completed'} on:click={() => filter = 'completed'}>Completadas</button>
        </div>

        <ul>
          {#each filteredTasks as task, index}
            <li class:completed={task.done}>
              <div class="task-content">
                <input type="checkbox" bind:checked={task.done} on:change={() => toggleDone(task)} />
                <span>
                  {task.text}
                </span>
              </div>
              <button class="delete-btn" on:click={() => removeTask(index)}>Eliminar</button>
            </li>
          {/each}
          
          {#if filteredTasks.length === 0}
            <p class="empty-message">No hay tareas {filter === 'all' ? '' : filter === 'active' ? 'activas' : 'completadas'}</p>
          {/if}
        </ul>

        <div class="todo-footer">
          <span>{tasksLeft} {tasksLeft === 1 ? 'tarea pendiente' : 'tareas pendientes'}</span>
          <button class="clear-btn" on:click={clearCompleted}>Limpiar completadas</button>
        </div>
      </div>
    {:else if activeSection === 'notas'}
      <div class="section-container">
        <h2>Notas de Tareas</h2>
        
        <div class="notes-layout">
          <div class="tasks-list">
            <h3>Selecciona una tarea</h3>
            <ul>
              {#each tasks as task, index}
                <li class:selected={selectedTask === task} class:completed={task.done} on:click={() => selectTask(task)}>
                  <div class="task-content">
                    <input type="checkbox" bind:checked={task.done} on:change={() => toggleDone(task)} />
                    <span>
                      {task.text}
                    </span>
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
                  value={selectedTask.text} 
                  on:input={(e) => updateTaskText(selectedTask, e.target.value)}
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

<style>
  :global(body) {
    background-color: #1e1e1e;
    color: #e0e0e0;
    margin: 0;
    padding: 0;
    min-height: 100vh;
    font-family: Arial, sans-serif;
  }

  main {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
    width: 100%;
    margin: 0;
    padding: 0;
  }

  .app-header {
    background-color: #222;
    padding: 15px 0;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
    width: 100%;
  }
  
  .header-content {
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 20px;
  }
  
  .main-nav button {
    background-color: transparent;
    color: #ccc;
    border: 1px solid #555;
    padding: 8px 16px;
    margin-left: 10px;
    border-radius: 4px;
    font-size: 1rem;
    transition: all 0.2s;
  }
  
  .main-nav button:hover {
    border-color: #ff8c00;
    color: #ff8c00;
  }
  
  .main-nav button.active {
    background-color: #ff8c00;
    border-color: #ff8c00;
    color: #fff;
  }
  
  .app-container {
    flex: 1;
    max-width: 800px;
    width: 100%;
    margin: 30px auto;
    background-color: #2d2d2d;
    border-radius: 10px;
    padding: 25px;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  }
  
  .section-container {
    width: 100%;
  }

  h1 {
    color: #ff8c00;
    margin: 0;
    font-size: 1.8rem;
  }
  
  h2 {
    color: #ff8c00;
    margin-top: 0;
    margin-bottom: 20px;
    font-size: 1.5rem;
    text-align: center;
  }
  
  h3 {
    color: #e0e0e0;
    margin-top: 0;
    margin-bottom: 15px;
    font-size: 1.2rem;
  }

  .input-container {
    display: flex;
    margin-bottom: 20px;
  }

  input[type="text"] {
    flex: 1;
    padding: 12px;
    border: none;
    background-color: #3d3d3d;
    color: #e0e0e0;
    border-radius: 5px 0 0 5px;
    font-size: 1rem;
  }

  input[type="text"]::placeholder {
    color: #888;
  }

  .add-btn {
    background-color: #ff8c00;
    color: #fff;
    border: none;
    padding: 12px 20px;
    border-radius: 0 5px 5px 0;
    font-weight: bold;
    transition: background-color 0.2s;
  }

  .add-btn:hover {
    background-color: #ff9d2f;
  }

  .filters {
    display: flex;
    justify-content: center;
    margin-bottom: 15px;
    gap: 10px;
  }

  .filters button {
    background-color: transparent;
    border: 1px solid #555;
    color: #ccc;
    padding: 6px 12px;
    border-radius: 4px;
    font-size: 0.9rem;
    transition: all 0.2s;
  }

  .filters button:hover {
    border-color: #ff8c00;
    color: #ff8c00;
  }

  .filters button.active {
    background-color: #ff8c00;
    border-color: #ff8c00;
    color: #fff;
  }

  ul {
    list-style: none;
    padding: 0;
    margin-bottom: 20px;
  }

  li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 12px 15px;
    background-color: #3d3d3d;
    margin-bottom: 8px;
    border-radius: 5px;
    transition: background-color 0.2s;
  }

  li:hover {
    background-color: #444;
  }

  li.completed {
    opacity: 0.7;
  }

  li.completed span {
    text-decoration: line-through;
    color: #888;
  }

  .task-content {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  input[type="checkbox"] {
    width: 18px;
    height: 18px;
    accent-color: #ff8c00;
  }

  .delete-btn {
    background-color: transparent;
    color: #ff5252;
    border: none;
    padding: 5px 10px;
    border-radius: 4px;
    opacity: 0.7;
    transition: opacity 0.2s;
  }

  .delete-btn:hover {
    opacity: 1;
  }

  .empty-message {
    text-align: center;
    color: #888;
    font-style: italic;
    padding: 15px 0;
  }

  .todo-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #888;
    font-size: 0.9rem;
  }

  .clear-btn {
    background-color: transparent;
    color: #ff8c00;
    border: none;
    padding: 5px 10px;
    font-size: 0.9rem;
    opacity: 0.8;
    transition: opacity 0.2s;
  }

  .clear-btn:hover {
    opacity: 1;
  }
  
  /* Estilos para la sección de notas */
  .notes-layout {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }
  
  .tasks-list {
    background-color: #333;
    padding: 15px;
    border-radius: 5px;
  }
  
  .note-editor {
    background-color: #333;
    padding: 15px;
    border-radius: 5px;
  }
  
  .tasks-list li {
    cursor: pointer;
  }
  
  .tasks-list li.selected {
    background-color: #4a4a4a;
    border-left: 3px solid #ff8c00;
  }
  
  .note-indicator {
    margin-left: 10px;
    color: #ff8c00;
  }
  
  .note-area textarea {
    width: 100%;
    background-color: #3d3d3d;
    color: #e0e0e0;
    border: 1px solid #555;
    border-radius: 5px;
    padding: 10px;
    font-family: inherit;
    resize: vertical;
    margin-bottom: 10px;
  }
  
  .save-btn {
    background-color: #ff8c00;
    color: #fff;
    border: none;
    padding: 8px 16px;
    border-radius: 5px;
    font-weight: bold;
    transition: background-color 0.2s;
  }
  
  .save-btn:hover {
    background-color: #ff9d2f;
  }
  
  .select-message {
    color: #888;
    font-style: italic;
    text-align: center;
    padding: 20px 0;
  }
  
  .edit-task {
    margin-bottom: 20px;
  }
  
  .edit-task input {
    width: 100%;
    padding: 10px;
    background-color: #3d3d3d;
    color: #e0e0e0;
    border: 1px solid #555;
    border-radius: 5px;
    font-size: 1rem;
    margin-bottom: 10px;
  }
  
  /* Estilos para el footer */
  .app-footer {
    background-color: #222;
    padding: 15px 0;
    margin-top: 30px;
    width: 100%;
  }
  
  .footer-content {
    max-width: 1200px;
    margin: 0 auto;
    text-align: center;
    color: #888;
    font-size: 0.9rem;
    padding: 0 20px;
  }
  
  /* Animación para el título */
  .animated-title {
    display: inline-block;
    color: #ff8c00;
    margin: 0;
    font-size: 1.8rem;
    animation: titlePulse 2s infinite alternate;
    transition: transform 0.3s ease;
  }
  
  .animated-title:hover {
    transform: scale(1.05);
  }
  
  @keyframes titlePulse {
    from {
      text-shadow: 0 0 5px rgba(255, 140, 0, 0.2);
    }
    to {
      text-shadow: 0 0 15px rgba(255, 140, 0, 0.6);
    }
  }
  
  /* Responsive design */
  @media (max-width: 768px) {
    .notes-layout {
      grid-template-columns: 1fr;
    }
    
    .header-content {
      flex-direction: column;
      gap: 15px;
    }
    
    h1 {
      text-align: center;
    }
  }
</style>
