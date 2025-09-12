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
  <div>
    <AppHeader v-if="!isAuthPage"/>
    <NuxtPage />
  </div>
</template>
