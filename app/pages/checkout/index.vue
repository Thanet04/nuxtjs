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
  <div class="max-w-5xl mx-auto py-10 px-4 md:px-6 min-h-screen bg-zinc-50 dark:bg-zinc-950">
    <h1 class="text-3xl font-bold mb-8 text-zinc-900 dark:text-white flex items-center gap-2">
      <span>💳</span> ชำระเงิน
    </h1>

    <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
      <!-- ข้อมูลผู้ซื้อ -->
      <div class="lg:col-span-2 space-y-6">
        <div class="bg-white dark:bg-zinc-900 p-6 md:p-8 rounded-2xl shadow-sm border border-gray-200 dark:border-zinc-800">
          <div class="flex items-center gap-3 border-b border-gray-100 dark:border-zinc-800 pb-4 mb-6">
             <div class="p-2 bg-blue-50 dark:bg-blue-900/20 rounded-lg text-blue-600 dark:text-blue-400">
               <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
             </div>
             <h2 class="text-xl font-bold text-zinc-900 dark:text-white">ข้อมูลการจัดส่ง</h2>
          </div>
          
          <form class="space-y-5">
            <div>
              <label class="block text-sm font-medium text-zinc-700 dark:text-zinc-300 mb-2">ชื่อ-นามสกุล</label>
              <input 
                v-model="name" 
                type="text" 
                class="w-full px-4 py-2.5 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
                placeholder="กรอกชื่อ-นามสกุล"
              />
            </div>
            <div>
              <label class="block text-sm font-medium text-zinc-700 dark:text-zinc-300 mb-2">ที่อยู่จัดส่ง</label>
              <textarea 
                v-model="address" 
                rows="3" 
                class="w-full px-4 py-2.5 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
                placeholder="กรอกที่อยู่จัดส่ง"
              ></textarea>
            </div>
            <div>
              <label class="block text-sm font-medium text-zinc-700 dark:text-zinc-300 mb-2">เบอร์โทรศัพท์</label>
              <input 
                v-model="phone" 
                maxlength="10" 
                type="tel" 
                class="w-full px-4 py-2.5 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
                placeholder="08xxxxxxxx"
              />
            </div>
          </form>
        </div>
        
        <router-link to="/cart" class="inline-flex items-center text-zinc-500 hover:text-zinc-800 dark:hover:text-zinc-200 transition font-medium">
          <svg class="w-5 h-5 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"></path></svg>
          ย้อนกลับไปตะกร้าสินค้า
        </router-link>
      </div>

      <!-- สรุปคำสั่งซื้อ -->
      <div class="lg:col-span-1">
        <div class="bg-white dark:bg-zinc-900 p-6 md:p-8 rounded-2xl shadow-sm border border-gray-200 dark:border-zinc-800 sticky top-24">
          <h2 class="text-lg font-bold mb-6 text-zinc-900 dark:text-white border-b border-gray-100 dark:border-zinc-800 pb-4">สรุปคำสั่งซื้อ</h2>
          <div class="space-y-4 mb-6">
            <div v-for="item in items" :key="item.id" class="flex justify-between items-start text-sm">
              <div class="pr-4">
                <p class="font-medium text-zinc-800 dark:text-zinc-200 line-clamp-2">{{ item.title }}</p>
                <p class="text-zinc-500 dark:text-zinc-500 mt-1">จำนวน: {{ item.quantity }}</p>
              </div>
              <p class="font-semibold text-zinc-900 dark:text-white whitespace-nowrap">{{ item.price * item.quantity }} ฿</p>
            </div>
          </div>
          
          <div class="border-t border-gray-100 dark:border-zinc-800 pt-4 mb-6">
            <div class="flex justify-between items-center bg-zinc-50 dark:bg-zinc-800/50 p-4 rounded-xl">
              <span class="text-zinc-600 dark:text-zinc-400 font-medium">ยอดรวมสุทธิ</span>
              <span class="text-xl font-bold text-blue-600 dark:text-blue-400">{{ totalPrice }} ฿</span>
            </div>
          </div>

          <button 
            @click="checkout"
            class="w-full bg-blue-600 hover:bg-blue-700 text-white py-3 px-4 rounded-xl font-bold transition shadow-lg shadow-blue-500/20 flex items-center justify-center gap-2 group"
          >
            <span>ยืนยันการชำระเงิน</span>
            <svg class="w-5 h-5 group-hover:translate-x-1 transition" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3"></path></svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
