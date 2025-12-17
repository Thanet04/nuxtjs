<script>
export default {
  data() {
    return {
      items: [],
      isLoading: false,
      userId: null
    }
  },
  async mounted() {
    await this.loadOrders()
  },
  computed: {
    totalPrice() {
      return this.items.reduce(
        (sum, item) => sum + (item.price || 0) * (item.quantity || 1),
        0
      )
    }
  },
  methods: {
  getAuthHeader() {
    const user = JSON.parse(localStorage.getItem('user'))
    return { 'Authorization': `Bearer ${user.token}`, 'Content-Type': 'application/json' }
  },

  async loadOrders() {
    this.isLoading = true
      try {
        const res = await fetch(`http://localhost:8080/api/orders`, {
          headers: this.getAuthHeader()
        })
        if (!res.ok) throw new Error('โหลดคำสั่งซื้อไม่สำเร็จ')
        const data = await res.json()

        this.items = data.map(o => ({
          id: o.id,
          title: o.title || 'ไม่มีชื่อหนังสือ',
          author: o.author || '-',
          price: o.price || 0,
          quantity: o.quantity || 1
        }))
      } catch (err) {
        console.error(err)
        this.$swal.fire({
          title: 'โหลดคำสั่งซื้อไม่สำเร็จ', 
          icon: 'warning',
          confirmButtonText: 'OK',
        });
      } finally {
        this.isLoading = false
      }
    },

    async removeItem(index) {
      const item = this.items[index]
      const result = await this.$swal.fire({
        title: 'คุณแน่ใจหรือไม่?',
        text: `ต้องการลบ "${item.title}" ออกจากตะกร้าใช่ไหม?`,
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: 'ตกลง',
        cancelButtonText: 'ยกเลิก',
      })

      if (!result.isConfirmed) return 

      try {
        const res = await fetch(`http://localhost:8080/api/orders/${item.id}`, {
          method: 'DELETE',
          headers: this.getAuthHeader()
        })
        if (!res.ok) throw new Error('ลบไม่สำเร็จ')

        this.items.splice(index, 1)

        window.dispatchEvent(new Event('cart-updated'))

        this.$swal.fire({
          title: 'ลบสินค้าสำเร็จ',
          icon: 'success',
          confirmButtonText: 'OK',
        })
      } catch (err) {
        console.error(err)
        this.$swal.fire({
          title: 'ลบไม่สำเร็จ',
          icon: 'error',
          confirmButtonText: 'OK',
        })
      }
    },

    async clearCart() {
      const result = await this.$swal.fire({
        title: 'คุณแน่ใจหรือไม่?',
        text: 'ต้องการล้างตะกร้าทั้งหมดหรือไม่?',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: 'ใช่',
        cancelButtonText: 'ยกเลิก',
      })

      if (!result.isConfirmed) return

      try {
        for (const item of [...this.items]) {
          await fetch(`http://localhost:8080/api/orders/${item.id}`, {
            method: 'DELETE',
            headers: this.getAuthHeader()
          })
        }
        this.items = []

        window.dispatchEvent(new Event('cart-updated'))
        this.$swal.fire({
          title: 'ล้างตะกร้าเรียบร้อย',
          icon: 'success',
          confirmButtonText: 'OK',
        })
      } catch (err) {
        console.error(err)
        this.$swal.fire({
          title: 'เกิดข้อผิดพลาดในการล้างตะกร้า',
          icon: 'error',
          confirmButtonText: 'OK',
        })
      }
    },


    async updateOrder(item) {
      try {
        await fetch(`http://localhost:8080/api/orders/${item.id}`, {
          method: 'PUT',
          headers: this.getAuthHeader(),
          body: JSON.stringify({
            title: item.title,
            quantity: item.quantity,
            price: item.price
          })
        })
      } catch (err) {
        this.$swal.fire({
          title: 'อัปเดตคำสั่งซื้อไม่สำเร็จ', 
          icon: 'error',
          confirmButtonText: 'OK',
        });
      }
    },

    checkout(){
      this.$router.push('/checkout')
    }
  }
}
</script>

<template>
  <div class="max-w-4xl mx-auto p-4 md:p-6 h-full min-h-screen">
    <h1 class="text-3xl font-bold mb-6 text-zinc-900 dark:text-white flex items-center gap-2">
      <span>🛒</span> ตะกร้าสินค้า
    </h1>

    <div v-if="isLoading" class="text-center text-zinc-500 dark:text-zinc-400 py-10">กำลังโหลด...</div>

    <div v-else-if="items.length === 0" class="text-zinc-500 dark:text-zinc-400 text-center py-16 bg-white dark:bg-zinc-900 rounded-2xl shadow-sm border border-gray-200 dark:border-zinc-800">
      <div class="text-6xl mb-4">🛍️</div>
      <p class="text-lg">ยังไม่มีสินค้าในตะกร้า</p>
      <router-link to="/books" class="mt-4 inline-block text-primary-600 hover:text-primary-700 font-medium">เลือกซื้อหนังสือ</router-link>
    </div>

    <div v-else class="space-y-6">
      <div class="space-y-4">
        <div
          v-for="(item, index) in items"
          :key="item.id"
          class="flex flex-col sm:flex-row items-center justify-between border border-gray-200 dark:border-zinc-800 p-4 rounded-2xl bg-white dark:bg-zinc-900 shadow-sm hover:shadow-md transition-shadow"
        >
          <div class="flex-1 text-center sm:text-left mb-4 sm:mb-0">
            <div class="text-lg font-bold text-zinc-900 dark:text-white">{{ item.title }}</div>
            <div class="text-sm text-zinc-500 dark:text-zinc-400">
              โดย {{ item.author }} | <span class="text-primary-600 dark:text-primary-400 font-medium">ราคา: {{ item.price }} บาท</span>
            </div>
          </div>

          <div class="flex items-center gap-3">
            <div class="flex items-center border border-gray-200 dark:border-zinc-700 rounded-lg bg-zinc-50 dark:bg-zinc-800 overflow-hidden">
               <button @click="decrease(item)" class="px-3 py-1 text-lg text-zinc-600 dark:text-zinc-300 hover:bg-zinc-200 dark:hover:bg-zinc-700 transition">-</button>
               <span class="px-3 font-medium text-zinc-900 dark:text-white">{{ item.quantity }}</span>
               <button @click="increase(item)" class="px-3 py-1 text-lg text-zinc-600 dark:text-zinc-300 hover:bg-zinc-200 dark:hover:bg-zinc-700 transition">+</button>
            </div>
            
            <button
              @click="removeItem(index)"
              class="p-2 bg-red-50 text-red-500 rounded-lg hover:bg-red-100 dark:bg-red-900/20 dark:text-red-400 dark:hover:bg-red-900/40 transition"
              title="ลบสินค้า"
            >
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
            </button>
          </div>
        </div>
      </div>

      <div class="bg-white dark:bg-zinc-900 rounded-2xl shadow-sm border border-gray-200 dark:border-zinc-800 p-6">
        <div class="flex flex-col sm:flex-row items-center justify-between gap-4">
          <div class="text-xl font-bold text-zinc-900 dark:text-white">
            รวมทั้งหมด: <span class="text-primary-600 dark:text-primary-400">{{ totalPrice }}</span> <span class="text-sm font-normal text-zinc-500">บาท</span>
          </div>
          <div class="flex gap-3 w-full sm:w-auto">
            <button
              @click="clearCart"
              class="flex-1 sm:flex-none px-6 py-2.5 bg-red-50 text-red-600 dark:bg-red-900/20 dark:text-red-400 rounded-xl font-medium hover:bg-red-100 dark:hover:bg-red-900/40 transition"
            >
              ล้างตะกร้า
            </button>
            <button 
              @click="checkout" 
              class="flex-1 sm:flex-none px-6 py-2.5 bg-green-600 text-white rounded-xl font-medium hover:bg-green-700 transition shadow-lg shadow-green-500/20"
            >
              ชำระเงิน
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
