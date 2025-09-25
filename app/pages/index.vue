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
    },
    size() {
      this.page = 1
      this.loadBooks()
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
        this.error = err.message
        this.$swal.fire({
          title: 'เกิดข้อผิดพลาด!',
          text: err.message,
          icon: 'error',
          confirmButtonText: 'OK',
        });
      } finally {
        this.isLoading = false
      }
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
        });
        window.dispatchEvent(new Event('cart-updated'))
      } catch {
        this.$swal.fire({
          title: 'ไม่สามารถเพิ่มลงตะกร้าได้',
          icon: 'warning',
          confirmButtonText: 'OK',
        });
      }
    }
  }
}
</script>

<template>
  <div class="w-full h-full">
    <img src="../assets/img/bannerbook.png" class="w-full h-100 object-cover" alt="Banner Image">
    <div class="w-full flex justify-center my-4">
      <h1 class="text-2xl font-bold">หนังสือ</h1>
    </div>
    
    <div class="w-full grid grid-cols-2 md:grid-cols-4 gap-4 px-4 my-4">
      <div v-for="book in books" :key="book.id"
        class="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-2xl transition transform hover:-translate-y-1" >
        <div class="h-56 overflow-hidden">
          <img
            v-if="book.imageUrl"
            :src="book.imageUrl"
            alt="cover"
            class="w-full h-full object-cover"
          />
          <div v-else class="w-full h-full bg-gray-200 flex items-center justify-center text-gray-400 text-lg">
            ไม่มีรูปปก
          </div>
        </div>
        <div class="p-4 flex flex-col gap-2">
          <router-link :to="`/books/${book.id}`" class="text-lg font-bold text-gray-800 hover:text-blue-600">
            {{ book.title }}
          </router-link>
          <div class="text-sm text-gray-500">ผู้เขียน: {{ book.author || '-' }}</div>
          <div class="text-sm text-gray-500">{{ book.description || '-' }}</div>
          <div class="text-sm font-medium text-gray-700">ราคา: {{ book.price || 0 }} บาท</div>
          <div class="mt-4 flex gap-2">
            <button
              @click="addToCart(book)"
              class="flex-1 px-3 py-2 bg-blue-500 text-white font-medium rounded-lg hover:bg-blue-600 transition"
            >
              ใส่ตะกร้า
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="flex flex-col items-center gap-2 my-4">
      <div class="flex justify-center gap-2">
        <button 
          @click="page = Math.max(1, page - 1)"
          :disabled="page <= 1"
          class="px-4 py-2 bg-blue-500 text-white rounded disabled:opacity-50 disabled:cursor-not-allowed"
        >
          ← Previous
        </button>
        
        <div class="flex items-center gap-1">
          <button 
            v-for="p in pageCount" 
            :key="p"
            @click="page = p"
            :class="[page === p ? 'bg-green-500 text-white' : 'bg-gray-200 text-gray-700 hover:bg-gray-300', 'px-3 py-2 rounded']"
          >
            {{ p }}
          </button>
        </div>
        
        <button 
          @click="page = Math.min(pageCount, page + 1)"
          :disabled="page >= pageCount"
          class="px-4 py-2 bg-blue-500 text-white rounded disabled:opacity-50 disabled:cursor-not-allowed"
        >
          Next →
        </button>
      </div>

      <div class="text-gray-500">
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
