<script setup>

import { ref, onMounted, computed, watch } from 'vue'
import draggable from 'vuedraggable'

const todos = ref([])
const name = ref('')

const input_content =ref('')
const input_category = ref(null)

/* const todos_asc = computed (() => todos.value.sort((a,b) => {
  return b.createdAt - a.createdAt
})) */

const addTodo = () => {
    if (input_content.value.trim() === ''  || input_category.value === null ){
    return
    }
    todos.value.push({
      content: input_content.value,
      category: input_category.value,
      done: false,
      createdAt: new Date().getTime()

    })
    input_content.value = ''
    input_category.value = null
}

const removeTodo = todo => {
  todos.value = todos.value.filter (t => t !== todo)
}

const filters = ref(['all', 'active', 'completed']);
const activeFilter = ref('active');

const filterTodo = (type) => {
    activeFilter.value = type;
};

const filteredTodos = computed (() => {
    const sorted = [...todos.value].sort((a, b) => b.createdAt - a.createdAt);

    if (activeFilter.value === 'all') {
      return sorted;
    } else if (activeFilter.value === 'completed') {
      return sorted.filter(todo => todo.done);
    } else if (activeFilter.value === 'active') {
      return sorted.filter(todo =>!todo.done);
    }
    return sorted;
});

const activeCount = computed(() => {
  return todos.value.filter(todo => !todo.done).length;
});

const completedCount = computed(() => {
    return todos.value.filter(todo => todo.done).length;
})

const saveOrder = () => {
  localStorage.setItem('todos', JSON.stringify(todos.value))
}

// Watch for changes and save to localStorage
watch(todos, newVal => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, {deep: true})

watch (name, (newVal) => {
  localStorage.setItem('name', newVal)
})

onMounted(() => {
  name.value = localStorage.getItem('name') || ''
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
  darkMode.value = localStorage.getItem('darkMode') === 'true'
})

/* Dark Mode */ 

const darkMode = ref(false)

watch(darkMode, (val) => {
    document.body.classList.toggle('dark', val)
    localStorage.setItem('darkMode', val)
})



</script>

<template>
    <main class="app">

        <section class="greeting">

            <h2 class=title>
              Welcome Back, <input type="text" placeholder="Name here"
              v-model="name"/>
            </h2>

        <p> You have {{ activeCount }} active tasks </p>
        <p> You have completed {{ completedCount }} tasks far! </p>
        </section>
        
        <section class="create-todo">

          <form @submit.prevent="addTodo"> 
              <h4> What's your goals today? </h4>
              <input type="text" placeholder="e.g. make a video" v-model="input_content"/>


              <h4> Pick a category</h4>

              <div class="options">

                <label>
                  <input type="radio" name="category" value="business" v-model="input_category"/>
                  <span class="bubble business"></span>
                  <div> business</div>
                </label>

                <label>
                  <input type="radio" name="category" value="personal" v-model="input_category"/>
                  <span class="bubble personal"></span>
                  <div> personal</div>
                </label>

              </div>
              
              <input type="submit" value="Add todo">
          </form>
        </section>

        <section class="todo-list">
          <div class="filters"> <!-- Add CSS -->
            <p v-for="(filter, index) in filters" :key="index">
                <span
                  @click="filterTodo(filter)"
                  :class=" { active:filter === activeFilter}"
                  >{{ filter }}</span>
            </p>
            
          </div>
            <div class="list">
              <div v-if="filteredTodos.length === 0" class="empty-state">
              <svg xmlns="http://www.w3.org/2000/svg" width="80" height="80" viewBox="0 0 24 24" fill="none" stroke="#6c757d" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
                <circle cx="12" cy="12" r="10"></circle>
                <line x1="12" y1="8" x2="12" y2="12"></line>
                <line x1="12" y1="16" x2="12.01" y2="16"></line>
              </svg>
              <p>No {{ activeFilter.value }} Tasks Found</p>
              <p> Add a new task to get started!</p>
              
              </div>

              <draggable 
                  v-else
                  :list="filteredTodos"
                  item-key="createdAt"
                  @end="saveOrder"
                >
                <template #item="{ element: todo }">
                <div 
                v-show="(activeFilter === 'all') || 
                        (activeFilter === 'completed' && todo.done) ||
                        (activeFilter === 'active' && !todo.done)"
                    :class="`todo-item ${todo.done && 'done'}`"
                    >
                  <label>
                    <input type="checkbox" v-model="todo.done" />
                    <span :class="`bubble ${todo.category}`"></span>
                  </label>

                  <div class="todo-content">
                    <input type="text" v-model="todo.content"/>
                  </div>

                  <div class="actions">
                    <button class="delete" @click="removeTodo(todo)">Delete</button>
                  </div>

                </div>
                
                </template>
              </draggable>    
            </div>
        </section>
    </main>
</template>