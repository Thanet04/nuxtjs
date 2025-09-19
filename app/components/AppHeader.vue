<script>
export default {
  data() {
    return {
      user: null,
      showMenu: false,
      cartCount: 0
    }
  },
  mounted() {
    this.loadUser();
    this.loadCartCount();

    window.addEventListener('user-updated', this.loadUser);
    window.addEventListener('cart-updated', this.loadCartCount);
  },
  beforeUnmount() {
    window.removeEventListener('user-updated', this.loadUser);
    window.removeEventListener('cart-updated', this.loadCartCount);
  },
  methods: {
    // คืนค่า header สำหรับ fetch
    getAuthHeader() {
      const user = JSON.parse(localStorage.getItem('user'));
      if (!user?.token) return {};
      return { 
        'Authorization': `Bearer ${user.token}`, 
        'Content-Type': 'application/json' 
      };
    },

    loadUser() {
      const storedUser = localStorage.getItem('user');
      if (storedUser) {
        this.user = JSON.parse(storedUser);
      }
    },

    async loadCartCount() {
      if (!this.user?.token) return;
      try {
        const res = await fetch(`http://localhost:8080/api/orders`, {
          headers: this.getAuthHeader()
        });
        if (!res.ok) throw new Error('ไม่สามารถโหลดคำสั่งซื้อได้');
        const data = await res.json();
        this.cartCount = data.reduce((sum, item) => sum + (item.quantity || 1), 0);
      } catch (err) {
        console.error(err);
        this.cartCount = 0;
      }
    },

    toggleMenu() {
      this.showMenu = !this.showMenu;
      console.log(this.showMenu);
    },

    logout() {
      localStorage.removeItem('user');
      this.user = null;
      this.$swal.fire({
        title: 'Logged out!', 
        icon: 'info',
        confirmButtonText: 'OK',
      }).then(() => {
        this.$router.push('/signin');
      });
    }

  }
}
</script>
<template>
  <nav class="w-full flex items-center justify-between py-4 px-4">
    <!-- Logo -->
    <div class="flex items-center gap-3">
      <img src="https://github.com/nuxt.png" width="32" height="32" alt="logo" />
      <NuxtLink to="/" class="text-2xl font-bold cursor-pointer">NuxtJS</NuxtLink>
    </div>

    <!-- ถ้า login แล้ว -->
    <div v-if="user" class="relative flex items-center gap-3">
      <!-- Cart Button -->
      <UButton to="/cart" variant="ghost" size="md" class="relative">
        🛒
        <span v-if="cartCount > 0" class="absolute -top-1 -right-1 bg-red-500 text-white text-xs rounded-full px-1 min-w-[18px] text-center">{{ cartCount }}</span>
      </UButton>

      <UAvatar :src="user.avatar || 'https://i.pravatar.cc/150?u=' + user.email"
        size="md" class="cursor-pointer" @click="toggleMenu" />

      <!-- Dropdown Menu -->
      <div v-if="showMenu" class="absolute top-full right-0 mt-2 flex flex-col gap-2 bg-white text-black p-2 rounded shadow-lg" >
        <UButton class="hover:bg-gray-200 cursor-pointer" color="blue" variant="ghost" size="md" to="/profile">Profile</UButton>
        <UButton class="hover:bg-gray-200 cursor-pointer" color="blue" variant="ghost" size="md" to="/books">Books</UButton>
        <UButton class="hover:bg-gray-200 cursor-pointer" color="red" variant="ghost" size="md" @click="logout">Logout</UButton>
      </div>
    </div>

    <div v-else class="flex gap-2">
      <UButton to="/cart" variant="ghost" size="md" class="relative">
        🛒
        <span v-if="cartCount > 0" class="absolute -top-1 -right-1 bg-red-500 text-white text-xs rounded-full px-1 min-w-[18px] text-center">{{ cartCount }}</span>
      </UButton>
      <UButton variant="ghost" size="md" to="/signin">Sign in</UButton>
      <UButton variant="ghost" size="md" to="/signup">Sign up</UButton>
    </div>
  </nav>
</template>
