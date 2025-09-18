<script>
export default {
  computed: {
    isAuthPage() {
      const authPages = ['/signin', '/signup', '/forgot_password'];
      return authPages.includes(this.$route.path);
    }
  }
}
</script>

<template>
  <div class="h-screen flex flex-col">
    <AppHeader v-if="!isAuthPage"/>
    <main class="flex-1">
      <NuxtPage />
    </main>
  </div>
</template>
