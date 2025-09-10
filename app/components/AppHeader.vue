<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

  const user = ref(null)
  const router = useRouter()

  onMounted(() => {
    const storedUser = localStorage.getItem('user')
    if (storedUser) {
      user.value = JSON.parse(storedUser)
    }
  })

  function logout() {
    localStorage.removeItem('user')
    user.value = null
    window.alert('Logged out!')
    router.push('/signin') 
  }
</script>

<template>
  <nav class="w-full flex items-center justify-between py-4 px-4">
    <!-- Logo -->
    <div class="flex items-center gap-3">
      <img src="https://github.com/nuxt.png" width="32" height="32" alt="logo" />
      <h1 class="text-2xl font-bold">NuxtJS</h1>
    </div>

    <!-- ถ้า login แล้ว -->
    <div v-if="user" class="flex items-center gap-3">
      <UCollapsible class="flex flex-col gap-2">
        <UAvatar
          :src="user.avatar || 'https://i.pravatar.cc/150?u=' + user.email"
          size="md"
        />
        <UButton color="red" variant="ghost" size="sm" @click="logout">Logout</UButton>
      </UCollapsible>
    </div>

    <div v-else class="flex gap-2">
      <UButton variant="ghost" size="md" to="/signin">Sign in</UButton>
      <UButton variant="ghost" size="md" to="/signup">Sign up</UButton>
    </div>
  </nav>
</template>
