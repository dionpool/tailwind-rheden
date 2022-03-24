<script setup>
document.title = 'ToDo Project'

import '../assets/js/bootstrap.bundle.min';
import { ref, computed, watchEffect } from 'vue'

const STORAGE_KEY = 'vue-todo'

const filters = {
  all: (todos) => todos,
  active: (todos) => todos.filter((todo) => !todo.completed),
  completed: (todos) => todos.filter((todo) => todo.completed)
}

// State
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
const visibility = ref('all')
const editedTodo = ref()

// Derived state
const filteredTodos = computed(() => filters[visibility.value](todos.value))
const remaining = computed(() => filters.active(todos.value).length)

// Handle routing
window.addEventListener('hashchange', onHashChange)
onHashChange()

// Persist state
watchEffect(() => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
})

// Add to do item
function addTodo(e) {
  const value = e.target.value.trim()
  if (value) {
    todos.value.push({
      id: Date.now(),
      title: value,
      completed: false
    })
    e.target.value = ''
  }
}

// Remove to do item
function removeTodo(todo) {
  todos.value.splice(todos.value.indexOf(todo), 1)
}

// Edit to do item
let beforeEditCache = ''
function editTodo(todo) {
  beforeEditCache = todo.title
  editedTodo.value = todo
}

// Cancel edit of to do item
function cancelEdit(todo) {
  editedTodo.value = null
  todo.title = beforeEditCache
}

// Update edited to do item
function doneEdit(todo) {
  if (editedTodo.value) {
    editedTodo.value = null
    todo.title = todo.title.trim()
    if (!todo.title) removeTodo()
  }
}

// Remove all completed to do items
function removeCompleted() {
  todos.value = filters.active(todos.value)
}

// Hash change route
function onHashChange() {
  const route = window.location.hash.replace(/#\/?/, '')
  if (filters[route]) {
    visibility.value = route
  } else {
    window.location.hash = ''
    visibility.value = 'all'
  }
}
</script>

<style>
  @import '../assets/css/bootstrap.min.css';
  @import 'https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css';
  @import '../assets/css/style.css';
</style>

<template>
  <div class="container">
    <!-- Header -->
    <header class="header">
      <h1 class="mt-5 mb-5">To Do</h1>
      <input class="form-control-lg w-100 shadow-sm border-0" autofocus placeholder="Wat moet er gedaan worden?" @keyup.enter="addTodo">
    </header>
    <!-- No items found -->
    <div class="mt-5">
      <h5 v-if="!todos.length">Er zijn geen taken gevonden!</h5>
    </div>
    <!-- To do -->
    <div class="mt-5 me-0" v-show="todos.length">
      <ul class="list-inline d-flex justify-content-end">
        <li class="list-inline-item"><a href="#/all" :class="{ selected: visibility === 'all' }">Alles</a></li>
        <li class="list-inline-item"><a href="#/active" :class="{ selected: visibility === 'active' }">Actief</a></li>
        <li class="list-inline-item"><a href="#/completed" :class="{ selected: visibility === 'completed' }">Voltooid</a></li>
      </ul>
      <ul class="list-group">
        <li class="list-group-item shadow-sm border-0 rounded-1 my-2" v-for="todo in filteredTodos" :key="todo.id" :class="{ completed: todo.completed, editing: todo === editedTodo }">
          <div class="form-check">
            <input type="checkbox" class="form-check-input" v-model="todo.completed">
            <label class="form-check-label strikethrough" @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button class="btn btn-close btn-sm float-end" @click="removeTodo(todo)"></button>
          </div>
          <input type="text" class="form-control mt-1" v-if="todo === editedTodo" v-model="todo.title" @vnode-mounted="({ el }) => el.focus"  @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.escape="cancelEdit(todo)">
        </li>
      </ul>
    </div>
    <!-- Footer -->
    <footer class="footer d-flex justify-content-between mt-3">
      <span v-show="todos.length">
        <strong>{{ remaining }}</strong>
        <span>{{ remaining === 1 ? '&nbsp;taak' : '&nbsp;taken' }}</span>
      </span>
      <a href="#" @click="removeCompleted" v-show="todos.length > remaining">Voltooid legen</a>
    </footer>
  </div>
</template>