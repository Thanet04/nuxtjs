<script>
export default {
  data() {
    return {
      user: { username: '', email: '', fullName: '' },
      currentPassword: '',
      newPassword: '',
      confirmNewPassword: '',
      message: '',
      messageType: ''
    }
  },
  async mounted() {
    try {
      const res = await fetch('https://book-production-e730.up.railway.app/api/user/me', {
        headers: this.getAuthHeader()
      })
      if (!res.ok) throw new Error('Failed to fetch user data')
      const data = await res.json()
      this.user.username = data.username
      this.user.email = data.email
      this.user.fullName = data.fullname || ''
    } catch (err) {
      console.error(err)
      this.$swal.fire({
        title: 'ไม่สามารถโหลดข้อมูลผู้ใช้ได้', 
        text: err.message, 
        icon: 'error',
        confirmButtonText: 'OK',
      });
    }
  },
  methods: {
    getAuthHeader() {
      const storedUser = JSON.parse(localStorage.getItem('user') || '{}')
      return { 'Authorization': `Bearer ${storedUser.token}`, 'Content-Type': 'application/json' }
    },

    async saveProfile() {
      try {
        const res = await fetch('https://book-production-e730.up.railway.app/api/user/me', {
          method: 'PUT',
          headers: this.getAuthHeader(),
          body: JSON.stringify({
            username: this.user.username,
            email: this.user.email,
            fullName: this.user.fullName
          })
        })
        if (!res.ok) throw new Error('Update failed')
        this.$swal.fire({
          title: 'อัปเดตโปรไฟล์สำเร็จ', 
          icon: 'success',
          confirmButtonText: 'OK',
        });
      } catch (err) {
        console.error(err)
        this.$swal.fire({
            title: 'เกิดข้อผิดพลาดในการอัปเดต', 
            text: err.message, 
            icon: 'error',
            confirmButtonText: 'OK',
          });
      }
    },

    async changePassword() {
      if (this.newPassword !== this.confirmNewPassword) {
        this.$swal.fire({
          title: 'รหัสผ่านใหม่ไม่ตรงกัน', 
          icon: 'warning',
          confirmButtonText: 'OK',
        });
        return
      }

      try {
        const res = await fetch('https://book-production-e730.up.railway.app/api/user/me/change-password', {
          method: 'PUT',
          headers: this.getAuthHeader(),
          body: JSON.stringify({
            currentPassword: this.currentPassword,
            newPassword: this.newPassword
          })
        })
        if (!res.ok) throw new Error('Change password failed')

        this.currentPassword = ''
        this.newPassword = ''
        this.confirmNewPassword = ''
        this.message = 'เปลี่ยนรหัสผ่านสำเร็จ'
        this.messageType = 'success'
        this.$swal.fire({
          title: 'เปลี่ยนรหัสผ่านสำเร็จ', 
          icon: 'success',
          confirmButtonText: 'OK',
        });
      } catch (err) {
        console.error(err)
        this.$swal.fire({
          title: 'เกิดข้อผิดพลาดในการเปลี่ยนรหัสผ่าน', 
          text: err.message, 
          icon: 'error',
          confirmButtonText: 'OK',
        });
      }
    },

    async deleteAccount() {
      const result = await this.$swal.fire({
        title: 'คุณแน่ใจว่าต้องการลบบัญชี?', 
        text: 'การลบบัญชีจะไม่สามารถกู้คืนได้!', 
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: 'ตกลง', 
        cancelButtonText: 'ยกเลิก',
        reverseButtons: true, 
      });

      if (!result.isConfirmed) return;

      try {
        const res = await fetch('https://book-production-e730.up.railway.app/api/user/me', {
          method: 'DELETE',
          headers: this.getAuthHeader(),
        });
        if (!res.ok) throw new Error('Delete failed');
        localStorage.removeItem('user');

        this.$swal.fire({
          title: 'บัญชีถูกลบแล้ว', 
          icon: 'success',
          confirmButtonText: 'OK',
        }).then(() => {
          this.$router.push('/signup');
        });

      } catch (err) {
        console.error(err);
        this.$swal.fire({
          title: 'เกิดข้อผิดพลาดในการลบบัญชี', 
          text: err.message,
          icon: 'error',
          confirmButtonText: 'OK',
        });
      }
    }
  }
}
</script>

<template>
  <div class="max-w-2xl mx-auto p-8 space-y-8 bg-gray-50 rounded-2xl shadow-lg">
    <h1 class="text-3xl font-bold text-center text-gray-800">โปรไฟล์ของคุณ</h1>

    <!-- ข้อความแจ้งเตือน -->
    <p
      v-if="message"
      :class="messageType === 'success' ? 'text-green-600' : 'text-red-500'"
      class="text-center font-medium"
    >
      {{ message }}
    </p>

    <!-- ข้อมูลผู้ใช้ -->
    <section class="bg-white text-black p-6 rounded-xl shadow-md space-y-4">
      <h2 class="text-xl font-semibold border-b pb-2 text-gray-800">ข้อมูลผู้ใช้</h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <!-- Full Name -->
        <div class="flex flex-col space-y-1">
          <label class="font-medium text-gray-700">Full Name</label>
          <input
            v-model="user.fullName"
            type="text"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
        </div>

        <!-- Username -->
        <div class="flex flex-col space-y-1">
          <label class="font-medium text-gray-700">Username</label>
          <input
            v-model="user.username"
            type="text"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
        </div>

        <!-- Email -->
        <div class="flex flex-col space-y-1 md:col-span-2">
          <label class="font-medium text-gray-700">Email</label>
          <input
            v-model="user.email"
            type="email"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
          />
        </div>
      </div>

      <button
        @click="saveProfile"
        class="w-full bg-blue-600 text-white py-2 rounded-md font-semibold hover:bg-blue-700 transition"
      >
        บันทึกข้อมูล
      </button>
    </section>

    <!-- เปลี่ยนรหัสผ่าน -->
    <section class="bg-white text-black p-6 rounded-xl shadow-md space-y-4">
      <h2 class="text-xl font-semibold border-b pb-2 text-gray-800">เปลี่ยนรหัสผ่าน</h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="flex flex-col space-y-1">
          <label class="font-medium text-gray-700">รหัสผ่านปัจจุบัน</label>
          <input
            v-model="currentPassword"
            type="password"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-green-500 focus:outline-none"
          />
        </div>

        <div class="flex flex-col space-y-1">
          <label class="font-medium text-gray-700">รหัสผ่านใหม่</label>
          <input
            v-model="newPassword"
            type="password"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-green-500 focus:outline-none"
          />
        </div>

        <div class="flex flex-col space-y-1 md:col-span-2">
          <label class="font-medium text-gray-700">ยืนยันรหัสผ่านใหม่</label>
          <input
            v-model="confirmNewPassword"
            type="password"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-green-500 focus:outline-none"
          />
        </div>
      </div>

      <button
        @click="changePassword"
        class="w-full bg-green-600 text-white py-2 rounded-md font-semibold hover:bg-green-700 transition"
      >
        เปลี่ยนรหัสผ่าน
      </button>
    </section>

    <!-- ลบบัญชี -->
    <section class="bg-white p-6 rounded-xl shadow-md space-y-4 text-center">
      <h2 class="text-xl font-semibold border-b pb-2 text-red-600">ลบบัญชี</h2>
      <p class="text-gray-600">การลบบัญชีจะไม่สามารถกู้คืนได้</p>
      <button
        @click="deleteAccount"
        class="w-full bg-red-600 text-white py-2 rounded-md font-semibold hover:bg-red-700 transition"
      >
        ลบบัญชี
      </button>
    </section>
  </div>
</template>
