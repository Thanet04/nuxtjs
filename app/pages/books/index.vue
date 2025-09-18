<script>
export default {
  data() {
    return {
      books: [],
      userId: null,
      isLoading: false,
      error: null
    }
  },
  mounted() {
    const storedUser = JSON.parse(localStorage.getItem('user') || '{}')
    this.userId = storedUser.userid || null
    this.loadBooks()
  },
  methods: {
    async loadBooks() {
      this.isLoading = true
      this.error = null
      try {
        const response = await fetch('http://localhost:8080/api/books')
        if (!response.ok) throw new Error('ไม่สามารถดึงข้อมูลหนังสือได้')
        this.books = await response.json()
      } catch (err) {
        console.error(err)
        this.error = err.message
      } finally {
        this.isLoading = false
      }
    },
    async addToCart(book) {
      if (!this.userId) {
        alert('กรุณาเข้าสู่ระบบก่อนเพิ่มลงตะกร้า')
        return
      }
      try {
        const orderDTO = {
          bookId: book.id,
          title: book.title,
          quantity: 1,
          price: book.price || 0
        }

        const res = await fetch(`http://localhost:8080/api/users/${this.userId}/orders`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(orderDTO)
        })

        if (!res.ok) throw new Error('ไม่สามารถเพิ่มลงตะกร้าได้')

        const data = await res.json()
        console.log('Order created:', data)
        alert('เพิ่มลงตะกร้าเรียบร้อยแล้ว!')
      } catch (err) {
        console.error(err)
        alert(err.message)
      }
    }
  }
}
</script>

<template>
  <div class="max-w-6xl mx-auto p-6">
    <div class="flex items-center justify-between mb-6">
      <h1 class="text-3xl font-bold text-white">📚 รายการหนังสือ</h1>
      <router-link
        to="/books/create"
        class="px-4 py-2 bg-green-500 text-white font-semibold rounded-lg shadow hover:bg-green-600 transition"
      >
        เพิ่มหนังสือ
      </router-link>
    </div>

    <div v-if="isLoading" class="text-white text-center py-10">กำลังโหลดข้อมูล...</div>
    <div v-else-if="error" class="text-red-500 text-center py-10">{{ error }}</div>
    <div v-else-if="books.length === 0" class="text-white text-center py-10">ยังไม่มีรายการหนังสือ</div>

    <div v-else class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
      <div
        v-for="book in books"
        :key="book.id"
        class="bg-white rounded-xl shadow-lg overflow-hidden hover:shadow-2xl transition transform hover:-translate-y-1"
      >
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
          <div class="text-sm font-medium text-gray-700">ราคา: {{ book.price || 0 }} บาท</div>
          <div class="mt-4 flex gap-2">
            <button
              @click="addToCart(book)"
              class="flex-1 px-3 py-2 bg-blue-500 text-white font-medium rounded-lg hover:bg-blue-600 transition"
            >
              ใส่ตะกร้า
            </button>
            <router-link :to="`/books/${book.id}`"
              class="flex-1 px-3 py-2 border border-gray-300 rounded-lg text-gray-700 hover:bg-gray-100 transition text-center" >
              แก้ไข
            </router-link>

          </div>
        </div>
      </div>
    </div>
  </div>
</template>
