<script>
export default {
  data() {
    return {
      books: [],
      user: null,
      isLoading: false,
      error: null
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
    async loadBooks(){
      this.isLoading = true
      this.error = null
      try {
        const response = await fetch('http://localhost:8080/api/books', {
        })
        if (!response.ok) throw new Error('ไม่มีหนังสือ')
        this.books = await response.json()
      } catch (err) {
        console.error(err)
        this.$swal.fire({
          title: 'Error!',
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

        const data = await res.json()
        console.log('Order created:', data)
        this.$swal.fire({
          title: 'เพิ่มลงตะกร้าเรียบร้อยแล้ว!',
          icon: 'success',
          confirmButtonText: 'Cool',
        });
        window.dispatchEvent(new Event('cart-updated'))
      } catch (err) {
        console.error(err)
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
  </div>
</template>
  