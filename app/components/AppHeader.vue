<script>
export default {
  data() {
    return {
      user: null,
      showMenu: false
    }
  },
  mounted() {
    this.loadUser();

    window.addEventListener('user-updated', this.loadUser);
  },
  beforeUnmount() {
    window.removeEventListener('user-updated', this.loadUser);
  },
  methods: {
    loadUser(){
      const storedUser = localStorage.getItem('user')
      if (storedUser) {
        this.user = JSON.parse(storedUser)
      }
    },
    toggleMenu() {
      this.showMenu = !this.showMenu
      console.log(this.showMenu)
    },
    logout() {
      localStorage.removeItem('user')
      this.user = null
      window.alert('Logged out!')
      this.$router.push('/signin')
    }
  }
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
    <div v-if="user" class="relative flex items-center gap-3">
      <UAvatar :src="user.avatar || 'https://i.pravatar.cc/150?u=' + user.email"
        size="md" class="cursor-pointer" @click="toggleMenu" />

      <!-- Dropdown Menu -->
      <div v-if="showMenu" class="absolute top-full right-0 mt-2 flex flex-col gap-2 bg-white text-black p-2 rounded shadow-lg" >
        <UButton class="hover:bg-gray-200 cursor-pointer" color="blue" variant="ghost" size="md">Profile</UButton>
        <UButton class="hover:bg-gray-200 cursor-pointer" color="red" variant="ghost" size="md" @click="logout">Logout</UButton>
      </div>
    </div>

    <div v-else class="flex gap-2">
      <UButton variant="ghost" size="md" to="/signin">Sign in</UButton>
      <UButton variant="ghost" size="md" to="/signup">Sign up</UButton>
    </div>
  </nav>
</template>
