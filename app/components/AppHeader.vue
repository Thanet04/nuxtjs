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
    // โหลด user และ cart หลัง Vue instance พร้อมแล้ว
    this.loadUser();
    this.loadCartCount();

    // bind method สำหรับ event listener
    this.boundLoadUser = this.loadUser.bind(this);
    this.boundLoadCart = this.loadCartCount.bind(this);
    this.boundOutsideClick = this.onClickOutside.bind(this);

    window.addEventListener('user-updated', this.boundLoadUser);
    window.addEventListener('cart-updated', this.boundLoadCart);
    window.addEventListener('click', this.boundOutsideClick);
  },
  beforeUnmount() {
    window.removeEventListener('user-updated', this.boundLoadUser);
    window.removeEventListener('cart-updated', this.boundLoadCart);
    window.removeEventListener('click', this.boundOutsideClick);
  },
  methods: {
    loadUser() {
      const storedUser = localStorage.getItem('user');
      this.user = storedUser ? JSON.parse(storedUser) : null;
    },

    onClickOutside(event) {
      if (!this.showMenu) return;
      const menu = this.$refs.menuDropdown;
      const avatar = this.$refs.avatarBtn;
      const target = event.target;

      const menuEl = menu && (menu.$el || menu);
      const avatarEl = avatar && (avatar.$el || avatar);

      if (menuEl && menuEl.contains && menuEl.contains(target)) return;
      if (avatarEl && avatarEl.contains && avatarEl.contains(target)) return;

      this.showMenu = false;
    },

    getAuthHeader() {
      if (!this.user?.token) return {};
      return { 
        'Authorization': `Bearer ${this.user.token}`, 
        'Content-Type': 'application/json' 
      };
    },

    async authFetch(url, options = {}) {
      const headers = { ...this.getAuthHeader(), ...(options.headers || {}) };
      const response = await fetch(url, { ...options, headers });
      
      if (response.status === 401) {
        this.ExpiredToken();
        return null;
      }
      return response;
    },

    async loadCartCount() {
      if (!this.user?.token) return;
      try {
        const res = await this.authFetch('http://localhost:8080/api/orders');
        if (!res) return;
        if (!res.ok) throw new Error('ไม่สามารถโหลดคำสั่งซื้อได้');

        const data = await res.json();
        this.cartCount = data.reduce((sum, item) => sum + (item.quantity || 1), 0);
      } catch (err) {
        console.error(err);
        this.cartCount = 0;
      }
    },

    ExpiredToken() {
      localStorage.removeItem('user');
      this.user = null;
      this.cartCount = 0;
      this.$swal.fire({
        icon: 'warning',
        title: 'Session หมดอายุ',
        text: 'กรุณาเข้าสู่ระบบใหม่',
        confirmButtonText: 'OK'
      }).then(() => {
        this.$router.push('/'); // เด้งไปหน้า /
      });
    },

    logout() {
      localStorage.removeItem('user');
      this.user = null;
      this.cartCount = 0;
      this.$swal.fire({
        title: 'Logged out!', 
        icon: 'info',
        confirmButtonText: 'OK',
      }).then(() => {
        this.$router.push('/signin');
      });
    },

    toggleMenu() {
      this.showMenu = !this.showMenu;
    },
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

      <UAvatar ref="avatarBtn" :src="user.avatar || 'https://i.pravatar.cc/150?u=' + user.email"
        size="md" class="cursor-pointer" @click="toggleMenu" />

      <!-- Dropdown Menu -->
      <div v-if="showMenu" ref="menuDropdown" class="absolute top-full right-0 mt-2 flex flex-col gap-2 bg-white text-black p-2 rounded shadow-lg w-24" >
        <UButton class="hover:bg-gray-200 cursor-pointer" color="blue" variant="ghost" size="md" to="/profile">โปรไฟล์</UButton>
        <UButton class="hover:bg-gray-200 cursor-pointer" color="blue" variant="ghost" size="md" to="/books">หนังสือฉัน</UButton>
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
