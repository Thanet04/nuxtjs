<script>
export default {
  data() {
    return {
      user: { username: '', email: '' },
      currentPassword: '',
      newPassword: '',
      confirmNewPassword: '',
      message: '',
      messageType: ''
    }
  },
  async mounted() {
    try {
      const res = await fetch('http://localhost:8080/api/user/me', {
        headers: this.getAuthHeader()
      })
      if (!res.ok) throw new Error('Failed to fetch user data')
      const data = await res.json()
      this.user.username = data.username
      this.user.email = data.email
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
        const res = await fetch('http://localhost:8080/api/user/me', {
          method: 'PUT',
          headers: this.getAuthHeader(),
          body: JSON.stringify({ username: this.user.username, email: this.user.email })
        })
        if (!res.ok) throw new Error('Update failed')
        this.$swal.fire({
          title: 'อัปเดตโปรไฟล์สำเร็จ', 
          text: err.message, 
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
          text: err.message, 
          icon: 'warning',
          confirmButtonText: 'OK',
        });
      }

      try {
        const res = await fetch('http://localhost:8080/api/user/me/change-password', {
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
        this.$swal.fire({
          title: 'เปลี่ยนรหัสผ่านสำเร็จ', 
          text: err.message, 
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

      if (!result.isConfirmed) {
        return;
      }

      try {
        const res = await fetch('http://localhost:8080/api/user/me', {
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
  <div class="max-w-2xl mx-auto  p-6 space-y-8 bg-gray-50 rounded-xl">
    <h1 class="text-3xl text-black font-bold text-center">โปรไฟล์ของคุณ</h1>

    <!-- ข้อความแจ้งเตือน -->
    <p
      v-if="message"
      :class="messageType === 'success' ? 'text-green-600' : 'text-red-500'"
      class="text-center font-medium"
    >
      {{ message }}
    </p>

    <!-- ข้อมูลผู้ใช้ -->
    <section class="bg-white p-6 rounded-xl shadow space-y-4">
      <h2 class="text-xl text-black font-semibold border-b pb-2">ข้อมูลผู้ใช้</h2>

      <!-- Username -->
      <div class="flex flex-col space-y-1">
        <label class="font-medium text-gray-700">Username</label>
        <input
          v-model="user.username"
          type="text"
          class="w-full text-gray-700 px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <!-- Email -->
      <div class="flex flex-col space-y-1">
        <label class="font-medium text-gray-700">Email</label>
        <input
          v-model="user.email"
          type="email"
          class="w-full text-gray-700 px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <button
        @click="saveProfile"
        class="w-full bg-blue-600 text-white py-2 rounded-md font-semibold hover:bg-blue-700 transition cursor-pointer"
      >
        บันทึกข้อมูล
      </button>
    </section>

    <!-- เปลี่ยนรหัสผ่าน -->
    <section class="bg-white p-6 rounded-xl shadow space-y-4">
      <h2 class="text-xl text-black font-semibold border-b pb-2">เปลี่ยนรหัสผ่าน</h2>

      <div class="flex flex-col space-y-1">
        <label class="font-medium text-gray-700">รหัสผ่านปัจจุบัน</label>
        <input
          v-model="currentPassword"
          type="password"
          class="w-full text-gray-700 px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <div class="flex flex-col space-y-1">
        <label class="font-medium text-gray-700">รหัสผ่านใหม่</label>
        <input
          v-model="newPassword"
          type="password"
          class="w-full text-gray-700 px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <div class="flex flex-col space-y-1">
        <label class="font-medium text-gray-700">ยืนยันรหัสผ่านใหม่</label>
        <input
          v-model="confirmNewPassword"
          type="password"
          class="w-full text-gray-700 px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <button
        @click="changePassword"
        class="w-full bg-green-600 text-white py-2 rounded-md font-semibold hover:bg-green-700 transition cursor-pointer"
      >
        เปลี่ยนรหัสผ่าน
      </button>
    </section>

    <!-- ลบบัญชี -->
    <section class="bg-white p-6 rounded-xl shadow space-y-4 text-center">
      <h2 class="text-xl font-semibold border-b pb-2 text-red-600">ลบบัญชี</h2>
      <p class="text-gray-600">การลบบัญชีจะไม่สามารถกู้คืนได้</p>
      <button
        @click="deleteAccount"
        class="w-full bg-red-600 text-white py-2 rounded-md font-semibold hover:bg-red-700 transition cursor-pointer"
      >
        ลบบัญชี
      </button>
    </section>
  </div>
</template>
