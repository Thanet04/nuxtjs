<script>
import { ref, computed, onMounted } from 'vue'
import Swal from 'sweetalert2'

export default {
  name: 'CheckoutPage',
  data() {
    return {
      items: [],
      name: '',
      address: '',
      phone: ''
    }
  },
  computed: {
    totalPrice() {
      return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
    }
  },
  methods: {
    async loadCart() {
      try {
        const userRaw = localStorage.getItem('user')
        if (!userRaw) return
        const token = JSON.parse(userRaw).token

        const res = await fetch('http://localhost:8080/api/orders', {
          headers: { 'Authorization': `Bearer ${token}` }
        })
        if (!res.ok) throw new Error('ไม่สามารถโหลดคำสั่งซื้อได้')
        const data = await res.json()
        this.items = data
      } catch (err) {
        console.error(err)
      }
    },

    async checkout() {
      if (!this.name || !this.address || !this.phone) {
        Swal.fire({
          icon: 'warning',
          title: 'กรุณากรอกข้อมูลให้ครบถ้วน'
        })
        return
      }

      const result = await Swal.fire({
        title: `ยอดรวม ${this.totalPrice} บาท`,
        text: "คุณต้องการยืนยันการชำระเงินหรือไม่?",
        icon: "question",
        showCancelButton: true,
        confirmButtonText: "ยืนยัน",
        cancelButtonText: "ยกเลิก"
      })

      if (result.isConfirmed) {
        try {
          const userRaw = localStorage.getItem('user')
          if (!userRaw) throw new Error('กรุณาเข้าสู่ระบบก่อนชำระเงิน')
          const token = JSON.parse(userRaw).token

          const res = await fetch('http://localhost:8080/api/orders/checkout', {
            method: 'POST',
            headers: { 'Authorization': `Bearer ${token}` }
          })
          if (!res.ok) throw new Error('ชำระเงินไม่สำเร็จ')

          const message = await res.text()

          Swal.fire({
            icon: 'success',
            title: 'ชำระเงินสำเร็จ!',
            text: message
          })

          // เคลียร์ตะกร้า
          this.items = []
          this.name = ''
          this.address = ''
          this.phone = ''

          window.dispatchEvent(new Event('cart-updated'))
        } catch (err) {
          Swal.fire({
            icon: 'error',
            title: 'เกิดข้อผิดพลาด',
            text: err.message
          })
        }
      }
    }
  },
  mounted() {
    this.loadCart()
  }
}
</script>
<template>
  <div class="max-w-5xl mx-auto py-10 px-4">
    <h1 class="text-2xl font-bold mb-6">🛒 ชำระเงิน</h1>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <!-- ข้อมูลผู้ซื้อ -->
      <div class="md:col-span-2 text-black bg-white p-6 rounded-2xl shadow">
        <h2 class="text-xl font-bold mb-4">ข้อมูลการจัดส่ง</h2>
        <form class="space-y-4">
          <div>
            <label class="block text-sm text-md mb-1">ชื่อ-นามสกุล</label>
            <input v-model="name" type="text" class="w-full border rounded-lg p-2" />
          </div>
          <div>
            <label class="block text-sm text-md mb-1">ที่อยู่</label>
            <textarea v-model="address" rows="3" class="w-full border rounded-lg p-2"></textarea>
          </div>
          <div>
            <label class="block text-sm text-md mb-1">เบอร์โทรศัพท์</label>
            <input v-model="phone" maxlength="10" type="text" class="w-full border rounded-lg p-2" />
          </div>
        </form>
      </div>

      <!-- สรุปคำสั่งซื้อ -->
      <div class="bg-white text-black p-6 rounded-2xl shadow h-fit">
        <h2 class="text-lg font-semibold mb-4">คำสั่งซื้อของคุณ</h2>
        <ul class="divide-y">
          <li v-for="item in items" :key="item.id" class="flex justify-between py-2">
            <div>
              <p class="text-md">{{ item.title }}</p>
              <p class="text-sm text-gray-500">x{{ item.quantity }}</p>
            </div>
            <p>{{ item.price * item.quantity }} ฿</p>
          </li>
        </ul>

        <div class="flex justify-between font-bold text-lg mt-4">
          <span>ยอดรวม</span>
          <span>{{ totalPrice }} ฿</span>
        </div>

        <button 
          @click="checkout"
          class="w-full mt-6 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition">
          ยืนยันชำระเงิน
        </button>
      </div>
    </div>
  </div>
</template>
