<script>
export default {
  data() {
    return {
      email: '',
      password: '',
      loading: false,
      message: ''
    }
  },
  methods: {
    async forgotPassword() {
      this.loading = true;
      this.message = '';

      try {
        const response = await fetch('http://localhost:8080/api/user/reset-password', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ email: this.email ,password: this.password})
        })

        const message = await response.text() 

        if (response.ok) {
          this.message = message 
          this.$router.push('/signin')
        } else {
          this.message = message || 'Something went wrong. Please try again.'
        }

      } catch (error) {
        this.message = 'Something went wrong. Please try again.'
        console.error(error)
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<template>
  <div class="h-full bg-gradient-to-br from-blue-50 via-indigo-50 to-purple-50 flex items-center justify-center p-4">
    <div class="w-full max-w-md">
      <div class="bg-white/80 backdrop-blur-sm rounded-2xl shadow-xl border border-white/20 p-8 sm:p-10">
        
        <!-- Header -->
        <div class="text-center mb-8">
          <h1 class="text-3xl font-bold bg-gradient-to-r from-gray-900 to-gray-600 bg-clip-text text-transparent">
            Forgot Password
          </h1>
          <p class="text-gray-500 mt-2">Enter your email and new password</p>
        </div>

        <!-- Form -->
        <form @submit.prevent="forgotPassword">
          <!-- Email -->
          <div class="mb-6">
            <label for="email" class="block text-sm font-medium text-gray-700">Email</label>
            <input 
              type="email" 
              id="email" 
              v-model="email"
              class="w-full px-4 py-2 text-black border border-gray-300 rounded-md focus:outline-none focus:border-blue-500" 
              placeholder="Enter your email" 
              required
            >
          </div>

          <!-- New Password -->
          <div class="mb-6">
            <label for="password" class="block text-sm font-medium text-gray-700">New Password</label>
            <input 
              type="password" 
              id="password" 
              v-model="password"
              class="w-full px-4 py-2 text-black border border-gray-300 rounded-md focus:outline-none focus:border-blue-500" 
              placeholder="Enter new password" 
              required
            >
          </div>

          <!-- Submit Button -->
          <button 
            type="submit" 
            class="w-full py-2 px-4 bg-blue-600 text-white rounded-md font-semibold hover:bg-blue-700 transition disabled:opacity-50"
            :disabled="loading"
          >
            <span v-if="loading">Resetting...</span>
            <span v-else>Reset Password</span>
          </button>
        </form>

        <!-- Message -->
        <p v-if="message" class="text-center mt-4 text-sm text-gray-600">{{ message }}</p>

        <!-- Back to Sign In -->
        <div class="text-center mt-6">
            <p class="text-sm text-gray-600">
                Already have an account?
                <NuxtLink class="font-medium text-blue-600 hover:text-blue-500 transition-colors duration-200" to="/signin">
                    Sign in
                </NuxtLink>
            </p>
        </div>
      </div>
    </div>
  </div>
</template>
