<script>
export default {
  data() {
    return {
      title: '',
      author: '',
      description: '',
      price: '',
      coverPreview: '',
      coverFile: null,
      isLoading: false
    }
  },
  methods: {
    getAuthHeader() {
      const user = JSON.parse(localStorage.getItem('user'))
      return { 'Authorization': `Bearer ${user.token}` }
    },
    onFileChange(event) {
      const file = event.target.files && event.target.files[0]
      if (!file) return

      this.coverFile = file
      const reader = new FileReader()
      reader.onload = () => {
        this.coverPreview = reader.result
      }
      reader.readAsDataURL(file)
    },
    removeImage() {
      this.coverFile = null
      this.coverPreview = ''
    },
    async save() {
      if (!this.title || !this.author || !this.price) {
        this.$swal.fire({
          title: 'กรุณากรอกข้อมูลให้ครบ', 
          icon: 'warning',
          confirmButtonText: 'OK',
        });
      }

      this.isLoading = true
      try {
        const formData = new FormData()
        const book = {
          title: this.title,
          author: this.author,
          description: this.description,
          price: Number(this.price)
        }

        formData.append('book', new Blob([JSON.stringify(book)], { type: 'application/json' }))
        if (this.coverFile) {
          formData.append('file', this.coverFile)
        }

        const response = await fetch('http://localhost:8080/api/books', {
          method: 'POST',
          headers: this.getAuthHeader(),
          body: formData
        })

        if (!response.ok) throw new Error('ไม่สามารถบันทึกหนังสือได้')
        this.$swal.fire({
          title: 'บันทึกหนังสือสำเร็จ', 
          icon: 'success',
          confirmButtonText: 'OK',
        }).then(() => {
            this.$router.push('/books');
        });
      } catch (err) {
        console.error(err)
         this.$swal.fire({
          title: 'ไม่สามารถบันทึกหนังสือได้', 
          icon: 'warning',
          confirmButtonText: 'OK',
        });
      } finally {
        this.isLoading = false
      }
    }
  }
}
</script>
<template>
  <div class="max-w-2xl mx-auto p-6 md:p-8 space-y-8 bg-zinc-50 dark:bg-zinc-950 min-h-screen">
    <div class="bg-white dark:bg-zinc-900 rounded-2xl shadow-sm border border-gray-200 dark:border-zinc-800 p-8">
      <h1 class="text-2xl font-bold mb-6 text-zinc-900 dark:text-white flex items-center gap-2">
        <span>📚</span> สร้างหนังสือใหม่
      </h1>

      <form class="space-y-6" @submit.prevent="save">
        <!-- Title -->
        <div class="space-y-2">
          <label class="block font-medium text-zinc-700 dark:text-zinc-300">ชื่อหนังสือ</label>
          <input
            v-model="title"
            type="text"
            placeholder="กรอกชื่อหนังสือ"
            class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-primary-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
          />
        </div>

        <!-- Author -->
        <div class="space-y-2">
          <label class="block font-medium text-zinc-700 dark:text-zinc-300">ผู้เขียน</label>
          <input
            v-model="author"
            type="text"
            placeholder="กรอกชื่อผู้เขียน"
             class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-primary-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
          />
        </div>

        <!-- Description -->
        <div class="space-y-2">
          <label class="block font-medium text-zinc-700 dark:text-zinc-300">รายละเอียด</label>
          <textarea
            v-model="description"
            rows="4"
            placeholder="กรุณากรอกรายละเอียด"
             class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-primary-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
          ></textarea>
        </div>

        <!-- Price -->
        <div class="space-y-2">
          <label class="block font-medium text-zinc-700 dark:text-zinc-300">ราคา</label>
          <div class="relative">
            <input
              v-model="price"
              type="number"
              placeholder="0"
               class="w-full px-4 py-2 bg-white dark:bg-zinc-800 border border-gray-300 dark:border-zinc-700 rounded-xl focus:ring-2 focus:ring-primary-500 focus:border-transparent outline-none transition text-zinc-900 dark:text-white placeholder-zinc-400"
            />
            <span class="absolute right-4 top-2 text-zinc-500">บาท</span>
          </div>
        </div>

        <!-- Cover Upload -->
        <div class="space-y-2">
          <label class="block font-medium text-zinc-700 dark:text-zinc-300">รูปปกหนังสือ</label>
          <label class="flex flex-col items-center justify-center w-full h-32 border-2 border-dashed border-gray-300 dark:border-zinc-700 rounded-xl cursor-pointer hover:bg-gray-50 dark:hover:bg-zinc-800 transition overflow-hidden bg-zinc-50 dark:bg-zinc-900 relative">
             <div v-if="!coverPreview" class="flex flex-col items-center justify-center pt-5 pb-6">
                 <svg class="w-8 h-8 mb-2 text-zinc-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"></path></svg>
                 <p class="mb-2 text-sm text-zinc-500 dark:text-zinc-400"><span class="font-semibold">คลิกเพื่ออัปโหลด</span></p>
             </div>
             <img v-else :src="coverPreview" class="w-full h-full object-cover" />
             <input type="file" class="hidden" accept="image/*" @change="onFileChange" />
          </label>
          <div v-if="coverPreview" class="flex justify-end mt-2">
             <button type="button" @click.prevent="removeImage" class="text-sm text-red-500 hover:text-red-600 font-medium">ลบรูปภาพ</button>
          </div>
        </div>

        <!-- Action Buttons -->
        <div class="flex gap-3 justify-end pt-4 border-t border-gray-100 dark:border-zinc-800">
          <router-link
            to="/books"
            class="px-6 py-2.5 bg-zinc-100 dark:bg-zinc-800 text-zinc-700 dark:text-zinc-300 rounded-xl font-medium hover:bg-zinc-200 dark:hover:bg-zinc-700 transition"
          >
            ยกเลิก
          </router-link>
          <button
            type="submit"
            :disabled="isLoading"
            class="px-6 py-2.5 bg-primary-600 text-white rounded-xl font-medium hover:bg-primary-700 transition shadow-lg shadow-primary-500/20 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <span v-if="!isLoading">บันทึก</span>
            <span v-else>กำลังบันทึก...</span>
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

