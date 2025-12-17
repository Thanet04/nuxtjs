<script>
export default {
  data() {
    return {
      books: [],
      user: null,
      isLoading: false,
      error: null,
      page: 1,
      size: 12,
      total: 0,
      search: '' 
    }
  },
  computed: {
    pageCount() {
      return Math.ceil(this.total / this.size)
    },
    startItem() {
      if (this.total === 0) return 0
      return (this.page - 1) * this.size + 1
    },
    endItem() {
      return Math.min(this.page * this.size, this.total)
    }
  },
  watch: {
    page(newPage, oldPage) {
      if (newPage && newPage !== oldPage) {
        if (this.search) {
          this.searchBooks()
        } else {
          this.loadBooks()
        }
      }
    },
    size() {
      this.page = 1
      if (this.search) {
        this.searchBooks()
      } else {
        this.loadBooks()
      }
    }
  },
  mounted() {
    this.loadUser()
    this.loadBooks()
  },
  methods: {
    getAuthHeader() {
      const userRaw = localStorage.getItem('user')
      if (!userRaw) return {}
      const user = JSON.parse(userRaw)
      return { 'Authorization': `Bearer ${user.token}`, 'Content-Type': 'application/json' }
    },
    loadUser() {
      const storedUser = localStorage.getItem('user')
      if (storedUser) {
        this.user = JSON.parse(storedUser)
      }
    },
    async loadBooks() {
      this.isLoading = true
      this.error = null
      try {
        const pageIndex = Math.max(0, (this.page || 1) - 1)
        const url = `http://localhost:8080/api/books?page=${pageIndex}&size=${this.size}`
        const response = await fetch(url, { method: 'GET' })
        if (!response.ok) throw new Error('ไม่มีหนังสือ')
        const data = await response.json()
        this.books = data.books || data
        this.total = data.total || (Array.isArray(data) ? data.length : 0)
      } catch (err) {
        this.books = []
        this.total = 0
        this.error = err.message
      } finally {
        this.isLoading = false
      }
    },
    async searchBooks() {
      if (!this.search) {
        this.page = 1
        this.loadBooks()
        return
      }
      this.isLoading = true
      this.error = null
      try {
        const pageIndex = Math.max(0, (this.page || 1) - 1)
        const url = `http://localhost:8080/api/books/search/title?title=${encodeURIComponent(this.search)}&page=${pageIndex}&size=${this.size}`
        const response = await fetch(url, { method: 'GET' })
        if (!response.ok) throw new Error('ไม่พบหนังสือที่ค้นหา')
        const data = await response.json()
        this.books = (data.books && data.books.length > 0) ? data.books : []
        this.total = data.total || (Array.isArray(data.books) ? data.books.length : 0)
      } catch (err) {
        this.books = []
        this.total = 0
        this.error = err.message
      } finally {
        this.isLoading = false
      }
    },
    clearSearch() {
      this.search = ''
      this.page = 1
      this.loadBooks()
    },
    async addToCart(book) {
      if (!this.user) {
        this.$swal.fire({
          title: 'กรุณาเข้าสู่ระบบก่อนเพิ่มลงตะกร้า',
          icon: 'warning',
          confirmButtonText: 'OK',
        })
        return
      }
      try {
        const orderDTO = {
          bookId: book.id,
          title: book.title,
          description: book.description,
          quantity: 1,
          price: book.price || 0
        }
        const res = await fetch('http://localhost:8080/api/orders', {
          method: 'POST',
          headers: this.getAuthHeader(),
          body: JSON.stringify(orderDTO)
        })
        if (!res.ok) throw new Error('ไม่สามารถเพิ่มลงตะกร้าได้')
        this.$swal.fire({
          title: 'เพิ่มลงตะกร้าเรียบร้อยแล้ว!',
          icon: 'success',
          confirmButtonText: 'OK',
        })
        window.dispatchEvent(new Event('cart-updated'))
      } catch {
        this.$swal.fire({
          title: 'ไม่สามารถเพิ่มลงตะกร้าได้',
          icon: 'warning',
          confirmButtonText: 'OK',
        })
      }
    }
  }
}
</script>

<template>
  <div class="w-full h-full pb-10">
    <div class="relative w-full h-64 md:h-80 lg:h-96 bg-gray-900 overflow-hidden">
      <!-- Gradient overlay instead of just image -->
      <img src="../assets/img/bannerbook.png" class="w-full h-full object-cover opacity-60" alt="Banner Image">
      <div class="absolute inset-0 bg-gradient-to-t from-zinc-950 to-transparent"></div>
      <div class="absolute bottom-0 left-0 w-full p-8 flex justify-center">
         <h1 class="text-4xl md:text-5xl font-bold text-white tracking-tight drop-shadow-md">Explore Our Collection</h1>
      </div>
    </div>
    
    <div class="w-full flex justify-center my-8">
      <h2 class="text-3xl font-bold text-zinc-900 dark:text-white">Books</h2>
    </div>

    <!-- Search bar -->
    <div class="flex justify-center my-6 px-4 gap-3 max-w-4xl mx-auto">
      <input
        v-model="search"
        @keyup.enter="searchBooks"
        type="text"
        placeholder="ค้นหาหนังสือ..."
        class="border border-gray-300 dark:border-zinc-700 bg-white dark:bg-zinc-900 text-zinc-900 dark:text-white rounded-xl px-4 py-2 w-full focus:outline-none focus:ring-2 focus:ring-primary-500 transition"
      />
      <button
        @click="searchBooks"
        class="px-6 py-2 bg-primary-600 text-white font-medium rounded-xl hover:bg-primary-700 transition shadow-sm"
      >
        ค้นหา
      </button>
      <button
        @click="clearSearch"
        class="px-6 py-2 bg-zinc-200 dark:bg-zinc-800 text-zinc-700 dark:text-gray-300 font-medium rounded-xl hover:bg-zinc-300 dark:hover:bg-zinc-700 transition"
      >
        ล้าง
      </button>
    </div>

    <!-- Books grid -->
    <div class="w-full grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 gap-6 px-4 md:px-8 my-8">
      <div v-for="book in books" :key="book.id"
        class="bg-white dark:bg-zinc-900 border border-gray-200 dark:border-zinc-800 rounded-2xl overflow-hidden hover:shadow-xl hover:border-primary-500/30 transition duration-300 group" >
        <div class="h-64 overflow-hidden relative">
          <img
            v-if="book.imageUrl"
            :src="book.imageUrl"
            alt="cover"
            class="w-full h-full object-cover group-hover:scale-105 transition duration-500"
          />
          <div v-else class="w-full h-full bg-zinc-100 dark:bg-zinc-800 flex items-center justify-center text-zinc-400 text-lg">
            ไม่มีรูปปก
          </div>
        </div>
        <div class="p-4 flex flex-col gap-2">
          <router-link :to="`/books/${book.id}`" class="text-lg font-bold text-zinc-900 dark:text-white hover:text-primary-500 line-clamp-1" :title="book.title">
            {{ book.title }}
          </router-link>
          <div class="text-sm text-zinc-500 dark:text-zinc-400 line-clamp-1">ผู้เขียน: {{ book.author || '-' }}</div>
          <div class="text-sm text-zinc-500 dark:text-zinc-400 line-clamp-2 h-10">{{ book.description || '-' }}</div>
          <div class="text-lg font-bold text-primary-600 dark:text-primary-400 mt-1">{{ book.price || 0 }} บาท</div>
          <div class="mt-3">
            <button
              @click="addToCart(book)"
              class="w-full py-2 bg-zinc-900 dark:bg-white text-white dark:text-zinc-900 font-medium rounded-lg hover:bg-primary-600 dark:hover:bg-primary-400 dark:hover:text-white transition-colors"
            >
              ใส่ตะกร้า
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Pagination -->
    <div class="flex flex-col items-center gap-4 my-8">
      <div class="flex justify-center gap-2">
        <button 
          @click="page = Math.max(1, page - 1)"
          :disabled="page <= 1"
          class="px-4 py-2 bg-white dark:bg-zinc-900 border border-gray-200 dark:border-zinc-800 text-zinc-700 dark:text-zinc-300 rounded-lg hover:bg-gray-50 dark:hover:bg-zinc-800 disabled:opacity-50 disabled:cursor-not-allowed transition"
        >
          ← Previous
        </button>
        
        <div class="flex items-center gap-1">
          <button 
            v-for="p in pageCount" 
            :key="p"
            @click="page = p"
            :class="[page === p ? 'bg-primary-600 text-white border-primary-600' : 'bg-white dark:bg-zinc-900 text-zinc-700 dark:text-zinc-300 border-gray-200 dark:border-zinc-800 hover:bg-gray-50 dark:hover:bg-zinc-800', 'px-3 py-2 rounded-lg border transition']"
          >
            {{ p }}
          </button>
        </div>
        
        <button 
          @click="page = Math.min(pageCount, page + 1)"
          :disabled="page >= pageCount"
          class="px-4 py-2 bg-white dark:bg-zinc-900 border border-gray-200 dark:border-zinc-800 text-zinc-700 dark:text-zinc-300 rounded-lg hover:bg-gray-50 dark:hover:bg-zinc-800 disabled:opacity-50 disabled:cursor-not-allowed transition"
        >
          Next →
        </button>
      </div>

      <div class="text-zinc-500 dark:text-zinc-400 text-sm">
        <template v-if="total > 0">
          แสดง {{ startItem }}–{{ endItem }} จาก {{ total }} รายการ
        </template>
        <template v-else>
          ไม่พบรายการ
        </template>
      </div>
    </div>
  </div>
</template>
