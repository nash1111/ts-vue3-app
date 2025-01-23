<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { hc } from 'hono/client'
import type { TodoApiRoutes, UserApiRoutes } from '@nash1111/hono-todo-backend/dist/index'

interface User {
  id: number
  name: string
  email: string
  password: string
  createdAt: string
  updatedAt: string
}

interface Todo {
  id: number
  title: string
  completed: boolean
  userId: number
}

const baseURL = 'http://localhost:3333/'
const userClient = hc<UserApiRoutes>('http://localhost:3333/')
const todoClient = hc<TodoApiRoutes>(baseURL)

const users = ref<User[]>([])
const todos = ref<Todo[]>([])

async function getUsers() {
  try {
    const res = await userClient.user.$get()
    const data = await res.json()
    users.value = data
  } catch (error) {
    console.error(error)
  }
}

async function createUser() {
  try {
    const res = await userClient.user.$post({
      json: {
        name: 'Test User',
        email: 'test@example.com',
        password: 'testpass',
      },
    })
    if (res.ok) {
      const createdUser = await res.json()
      console.log('User created:', createdUser)
      await getUsers()
    } else {
      const errorData = await res.json()
      console.error('Error creating user:', errorData)
    }
  } catch (error) {
    console.error(error)
  }
}

async function getTodos() {
  try {
    const res = await todoClient.todo.$get()
    const data = await res.json()
    todos.value = data
  } catch (error) {
    console.error(error)
  }
}

async function createTodo() {
  try {
    const res = await todoClient.todo.$post({
      json: {
        title: 'Sample Todo',
        completed: false,
        userId: 1,
      },
    })

    if (res.ok) {
      const createdTodo = await res.json()
      console.log('Todo created:', createdTodo)
      await getTodos()
    } else {
      const errorData = await res.json()
      console.error('Error creating todo:', errorData)
    }
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  getUsers()
  getTodos()
})
</script>

<template>
  <div>
    <h1>ユーザー管理</h1>
    <button @click="createUser">新規ユーザー作成</button>
    <ul>
      <li v-for="user in users" :key="user.id">
        ID: {{ user.id }} / {{ user.name }} ({{ user.email }})
      </li>
    </ul>

    <hr />

    <h1>ToDo管理</h1>
    <button @click="createTodo">新規ToDo作成</button>
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        ID: {{ todo.id }} / {{ todo.title }} / Done? {{ todo.completed }}
      </li>
    </ul>
  </div>
</template>
