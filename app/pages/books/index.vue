<script>
export default {
  data() {
    return {
      books: [],
      user: null,
      isLoading: false,
      error: null,
      page: 1,
      size: 6,   // จำนวนหนังสือต่อหน้า
      total: 0
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
        this.loadBooks()
      }
    }
  },
  mounted() {
    this.loadBooks()
  },
  methods: {
    getAuthHeader() {
      const user = JSON.parse(localStorage.getItem('user'))
      return { 'Authorization': `Bearer ${user.token}`, 'Content-Type': 'application/json' }
    },

    async loadBooks() {
      this.isLoading = true
      this.error = null
      try {
        const pageIndex = Math.max(0, this.page - 1)
        const response = await fetch(`http://localhost:8080/api/books/me?page=${pageIndex}&size=${this.size}`, {
          headers: this.getAuthHeader()
        })
        if (!response.ok) throw new Error('ไม่มีหนังสือ')
        const data = await response.json()
        this.books = data.books || data
        this.total = data.total || (Array.isArray(data) ? data.length : 0)
      } catch (err) {
        this.error = err.message
        this.$swal.fire({
          title: "ไม่มีหนังสือ",
          icon: "warning",
          confirmButtonText: 'OK',
        })
      } finally {
        this.isLoading = false
      }
    }
  }
}
</script>

<template>
  <div class="max-w-6xl mx-auto p-6">
    <div class="flex items-center justify-between mb-6">
      <h1 class="text-3xl font-bold text-zinc-900 dark:text-white">📚 รายการหนังสือ</h1>
      <router-link
        to="/books/create"
        class="px-4 py-2 bg-primary-600 text-white font-semibold rounded-lg shadow hover:bg-primary-700 transition"
      >
        เพิ่มหนังสือ
      </router-link>
    </div>

    <div v-if="isLoading" class="text-zinc-500 dark:text-zinc-400 text-center py-10">กำลังโหลดข้อมูล...</div>
    <div v-else-if="error" class="text-red-500 text-center py-10">{{ error }}</div>
    <div v-else-if="books.length === 0" class="text-zinc-500 dark:text-zinc-400 text-center py-10">ยังไม่มีรายการหนังสือ</div>

    <div v-else>
      <!-- Books grid -->
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
        <div
          v-for="book in books"
          :key="book.id"
          class="bg-white dark:bg-zinc-900 border border-gray-200 dark:border-zinc-800 rounded-xl overflow-hidden hover:shadow-xl hover:border-primary-500/30 transition duration-300 transform hover:-translate-y-1"
        >
          <div class="h-56 overflow-hidden relative">
            <img
              v-if="book.imageUrl"
              :src="book.imageUrl"
              alt="cover"
              class="w-full h-full object-cover"
            />
            <div v-else class="w-full h-full bg-zinc-100 dark:bg-zinc-800 flex items-center justify-center text-zinc-400 text-lg">
              ไม่มีรูปปก
            </div>
          </div>
          <div class="p-4 flex flex-col gap-2">
            <router-link :to="`/books/${book.id}`" class="text-lg font-bold text-zinc-900 dark:text-white hover:text-primary-600 line-clamp-1">
              {{ book.title }}
            </router-link>
            <div class="text-sm text-zinc-500 dark:text-zinc-400">ผู้เขียน: {{ book.author || '-' }}</div>
            <div class="text-lg font-bold text-primary-600 dark:text-primary-400">ราคา: {{ book.price || 0 }} บาท</div>
            <div class="mt-4 flex gap-2">
              <router-link :to="`/books/${book.id}`"
                class="flex-1 px-3 py-2 border border-gray-300 dark:border-zinc-700 rounded-lg text-zinc-700 dark:text-zinc-300 hover:bg-zinc-50 dark:hover:bg-zinc-800 transition text-center" >
                แก้ไข
              </router-link>
            </div>
          </div>
        </div>
      </div>

      <!-- Pagination -->
      <div class="flex flex-col items-center gap-4 my-6">
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
  </div>
</template>
