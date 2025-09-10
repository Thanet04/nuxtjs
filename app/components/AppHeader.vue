<script setup>
import { ref } from 'vue'

const user = ref(null)

onMounted(() => {
  user.value = JSON.parse(localStorage.getItem('user'))
})

</script>

<template>
  <nav class="w-full flex items-center justify-between py-4 px-4">
    <div class="flex items-center gap-3">
      <img src="https://github.com/nuxt.png" width="32" height="32" alt="logo" />
      <h1 class="text-2xl font-bold">NuxtJS</h1>
    </div>

    <div v-if="user">
      <UCollapsible class="flex flex-col gap-2 w-48">
        <UAvatar
          :src="user.avatar"
          :chip="{ inset: true }"
        />
      </UCollapsible>
    </div>

    <div v-else class="flex gap-2">
      <UButton variant="ghost" size="md" to="/signin">Sign in</UButton>
      <UButton variant="ghost" size="md" to="/signup">Sign up</UButton>
    </div>
  </nav>
</template>
