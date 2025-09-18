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
    // โหลด userId จาก localStorage
    const storedUser = JSON.parse(localStorage.getItem('user') || '{}')
    this.userId = storedUser.userid || null

    if (!this.userId) {
      alert('กรุณาเข้าสู่ระบบ')
      return
    }

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
    async loadOrders() {
      this.isLoading = true
      try {
        const res = await fetch(`http://localhost:8080/api/users/${this.userId}/orders`)
        if (!res.ok) throw new Error('โหลดคำสั่งซื้อไม่สำเร็จ')
        const data = await res.json()

        // Map จาก order ตาม structure ใหม่
        this.items = data.map(o => ({
          id: o.id,
          title: o.title || 'ไม่มีชื่อหนังสือ',
          author: o.author || '-',
          price: o.price || 0,
          quantity: o.quantity || 1
        }))
      } catch (err) {
        console.error(err)
        alert('โหลดคำสั่งซื้อไม่สำเร็จ')
      } finally {
        this.isLoading = false
      }
    },
    async increase(item) {
      item.quantity++
      await this.updateOrder(item)
    },
    async decrease(item) {
      if (item.quantity > 1) {
        item.quantity--
        await this.updateOrder(item)
      }
    },
    async removeItem(index) {
      const item = this.items[index]
      try {
        const res = await fetch(`http://localhost:8080/api/users/${this.userId}/orders/${item.id}`, {
          method: 'DELETE'
        })
        if (!res.ok) throw new Error('ลบไม่สำเร็จ')
        this.items.splice(index, 1)
      } catch (err) {
        console.error(err)
        alert(err.message)
      }
    },
    async clearCart() {
      for (const item of [...this.items]) {
        try {
          await fetch(`http://localhost:8080/api/users/${this.userId}/orders/${item.id}`, {
            method: 'DELETE'
          })
        } catch (err) {
          console.error(err)
        }
      }
      this.items = []
    },
    async updateOrder(item) {
      try {
        await fetch(`http://localhost:8080/api/users/${this.userId}/orders/${item.id}`, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            title: item.title,
            quantity: item.quantity,
            price: item.price,
            user: { id: this.userId }
          })
        })
      } catch (err) {
        console.error('อัปเดตคำสั่งซื้อไม่สำเร็จ', err)
      }
    }
  }
}
</script>

<template>
  <div class="max-w-4xl mx-auto p-4 h-full">
    <h1 class="text-2xl font-bold mb-4">🛒 ตะกร้าสินค้า</h1>

    <div v-if="isLoading" class="text-center text-gray-500">กำลังโหลด...</div>

    <div v-else-if="items.length === 0" class="text-gray-600 text-center py-8">
      ยังไม่มีสินค้าในตะกร้า
    </div>

    <div v-else class="space-y-4">
      <div
        v-for="(item, index) in items"
        :key="item.id"
        class="flex items-center justify-between border p-3 rounded-lg shadow-sm"
      >
        <div>
          <div class="font-medium">{{ item.title }}</div>
          <div class="text-sm text-gray-500">
            โดย {{ item.author }} | ราคา: {{ item.price }} บาท
          </div>
        </div>

        <div class="flex items-center gap-2">
          <button @click="decrease(item)" class="px-3 py-1 text-lg text-black bg-white rounded-lg cursor-pointer">-</button>
          <span class="px-2">{{ item.quantity }}</span>
          <button @click="increase(item)" class="px-3 py-1 text-lg text-black bg-white rounded-lg cursor-pointer">+</button>
          <button
            @click="removeItem(index)"
            class="px-3 py-1 bg-red-500 text-white rounded-lg cursor-pointer"
          >
            ลบ
          </button>
        </div>
      </div>

      <div class="flex items-center justify-between border-t pt-4">
        <div class="text-lg">รวม: <b>{{ totalPrice }}</b> บาท</div>
        <div class="flex gap-2">
          <button
            @click="clearCart"
            class="px-4 py-2 bg-red-500 rounded-lg hover:bg-red-700 cursor-pointer"
          >
            ล้างตะกร้า
          </button>
          <button class="px-4 py-2 bg-green-600 text-white rounded-lg cursor-pointer">
            ชำระเงิน
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
