<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { hc } from 'hono/client'
import type { TodoApiRoutes, UserApiRoutes } from '@nash1111/hono-todo-backend/dist/index'

const baseURL = 'http://localhost:3333/'
const userClient = hc<UserApiRoutes>('http://localhost:3333/')
const todoClient = hc<TodoApiRoutes>(baseURL)

async function getUsers() {
  try {
    const res = await userClient.user.$get()
    const users = await res.json()
    return users
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
    const todos = await res.json()
    return todos
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

onMounted(async () => {
  const users = await getUsers()
  const todos = await getTodos()
  console.log(users)
  console.log(todos)
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
