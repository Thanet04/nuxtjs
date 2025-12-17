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
      const res = await fetch('http://localhost:8080/api/user/me', {
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
        const res = await fetch('http://localhost:8080/api/user/me', {
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
  <div class="max-w-3xl mx-auto p-6 md:p-8 space-y-8 bg-zinc-50 dark:bg-zinc-950 min-h-screen">
    <div class="text-center">
      <h1 class="text-3xl font-bold text-zinc-900 dark:text-white">โปรไฟล์ของคุณ</h1>
      <p class="text-zinc-500 dark:text-zinc-400 mt-2">จัดการข้อมูลส่วนตัวและรหัสผ่าน</p>
    </div>

    <!-- ข้อความแจ้งเตือน -->
    <p
      v-if="message"
      :class="messageType === 'success' ? 'text-primary-600 dark:text-primary-400' : 'text-red-500'"
      class="text-center font-medium bg-white dark:bg-zinc-900 py-2 rounded-lg shadow-sm border border-gray-100 dark:border-zinc-800"
    >
      {{ message }}
    </p>

    <!-- ข้อมูลผู้ใช้ -->
    <section class="bg-white dark:bg-zinc-900 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-zinc-800 space-y-6">
      <div class="flex items-center gap-3 border-b border-gray-100 dark:border-zinc-800 pb-4">
        <div class="p-2 bg-primary-100 dark:bg-primary-900/30 rounded-lg text-primary-600 dark:text-primary-400">
           <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path></svg>
        </div>
        <h2 class="text-xl font-semibold text-zinc-900 dark:text-white">ข้อมูลผู้ใช้</h2>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <!-- Full Name -->
        <div class="flex flex-col space-y-2">
          <label class="font-medium text-zinc-700 dark:text-zinc-300">Full Name</label>
          <input
            v-model="user.fullName"
            type="text"
            class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-primary-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
          />
        </div>

        <!-- Username -->
        <div class="flex flex-col space-y-2">
          <label class="font-medium text-zinc-700 dark:text-zinc-300">Username</label>
          <input
            v-model="user.username"
            type="text"
            class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-primary-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
          />
        </div>

        <!-- Email -->
        <div class="flex flex-col space-y-2 md:col-span-2">
          <label class="font-medium text-zinc-700 dark:text-zinc-300">Email</label>
          <input
            v-model="user.email"
            type="email"
            class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-primary-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
          />
        </div>
      </div>

      <button
        @click="saveProfile"
        class="w-full bg-primary-600 text-white py-2.5 rounded-xl font-semibold hover:bg-primary-700 transition shadow-lg shadow-primary-500/20"
      >
        บันทึกข้อมูล
      </button>
    </section>

    <!-- เปลี่ยนรหัสผ่าน -->
    <section class="bg-white dark:bg-zinc-900 p-6 rounded-2xl shadow-sm border border-gray-200 dark:border-zinc-800 space-y-6">
      <div class="flex items-center gap-3 border-b border-gray-100 dark:border-zinc-800 pb-4">
        <div class="p-2 bg-blue-100 dark:bg-blue-900/30 rounded-lg text-blue-600 dark:text-blue-400">
           <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"></path></svg>
        </div>
        <h2 class="text-xl font-semibold text-zinc-900 dark:text-white">เปลี่ยนรหัสผ่าน</h2>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="flex flex-col space-y-2">
          <label class="font-medium text-zinc-700 dark:text-zinc-300">รหัสผ่านปัจจุบัน</label>
          <input
            v-model="currentPassword"
            type="password"
            class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white"
          />
        </div>

        <div class="flex flex-col space-y-2">
          <label class="font-medium text-zinc-700 dark:text-zinc-300">รหัสผ่านใหม่</label>
          <input
            v-model="newPassword"
            type="password"
             class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white"
          />
        </div>

        <div class="flex flex-col space-y-2 md:col-span-2">
          <label class="font-medium text-zinc-700 dark:text-zinc-300">ยืนยันรหัสผ่านใหม่</label>
          <input
            v-model="confirmNewPassword"
            type="password"
             class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white"
          />
        </div>
      </div>

      <button
        @click="changePassword"
        class="w-full bg-blue-600 text-white py-2.5 rounded-xl font-semibold hover:bg-blue-700 transition shadow-lg shadow-blue-500/20"
      >
        เปลี่ยนรหัสผ่าน
      </button>
    </section>

    <!-- ลบบัญชี -->
    <section class="bg-white dark:bg-zinc-900 p-6 rounded-2xl shadow-sm border border-red-100 dark:border-red-900/30 text-center space-y-4">
      <h2 class="text-xl font-semibold text-red-600 dark:text-red-500">ลบบัญชี</h2>
      <p class="text-zinc-600 dark:text-zinc-400">การลบบัญชีจะไม่สามารถกู้คืนได้ ข้อมูลทั้งหมดจะถูกลบถาวร</p>
      <button
        @click="deleteAccount"
        class="w-full bg-red-500/10 text-red-600 dark:text-red-400 py-2.5 rounded-xl font-semibold hover:bg-red-500 hover:text-white transition"
      >
        ลบบัญชี
      </button>
    </section>
  </div>
</template>
