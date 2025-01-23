<script setup lang="ts">
import { hc } from 'hono/client'
import { ref, onMounted } from 'vue'
import type { UserApiRoutes } from '@nash1111/hono-todo-backend/dist/index'
import type { HonoBase } from 'hono/hono-base'
import type { userApiRoutes } from '@nash1111/hono-todo-backend/dist/routes/user'
const userClient = hc<UserApiRoutes>('http://localhost:3333/')

type ExtractUserGetOutput<T> =
  T extends HonoBase<any, infer R, any>
    ? R['/user'] extends { $get: { output: infer O } }
      ? O
      : never
    : never
export type UserGetOutput = ExtractUserGetOutput<typeof userApiRoutes>

const users = ref<UserGetOutput>([])

onMounted(async () => {
  const res = await userClient.user.$get()
  users.value = await res.json()
  console.log(users.value)
})
</script>

<template>
  <div>
    <h1>User Management</h1>
    <ul>
      <li v-for="user in users" :key="user.id">
        ID: {{ user.id }} / {{ user.name }} ({{ user.email }})
      </li>
    </ul>
  </div>
</template>
