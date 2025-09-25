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
  <div class="max-w-xl mx-auto p-6 bg-white rounded-xl">
    <h1 class="text-2xl font-bold mb-6 bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">
      📚 สร้างหนังสือใหม่
    </h1>

    <form class="space-y-5" @submit.prevent="save">
      <!-- Title -->
      <div>
        <label class="block text-lg font-medium text-black mb-1">ชื่อหนังสือ</label>
        <input
          v-model="title"
          type="text"
          placeholder="กรอกชื่อหนังสือ"
          class="w-full px-3 py-2 text-black border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <!-- Author -->
      <div>
        <label class="block text-lg font-medium text-black mb-1">ผู้เขียน</label>
        <input
          v-model="author"
          type="text"
          placeholder="กรอกชื่อผู้เขียน"
          class="w-full px-3 py-2 border text-black border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <!-- Description -->
      <div>
        <label class="block text-lg font-medium text-black mb-1">รายละเอียด</label>
        <textarea
          v-model="description"
          type="text"
          placeholder="กรุณากรอกรายละเอียด"
          class="w-full px-3 py-2 text-black border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <!-- Price -->
      <div>
        <label class="block text-lg font-medium text-black mb-1">ราคา</label>
        <input
          v-model="price"
          type="number"
          placeholder="0"
          class="w-full px-3 py-2 text-black border border-gray-300 rounded-lg shadow-sm focus:ring-2 focus:ring-blue-500 focus:outline-none"
        />
      </div>

      <!-- Cover Upload -->
      <div>
        <label class="block text-lg font-medium text-black mb-1">รูปปกหนังสือ</label>
        <input
          type="file"
          accept="image/*"
          @change="onFileChange"
          class="block w-full text-sm text-gray-600 file:mr-4 file:py-2 file:px-4
                 file:rounded-full file:border-0 file:text-sm file:font-semibold
                 file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100"
        />

        <div v-if="coverPreview" class="mt-3 relative">
          <img
            :src="coverPreview"
            alt="preview"
            class="w-full h-56 object-cover rounded-xl border shadow-md"
          />
          <button
            type="button"
            @click.prevent="removeImage"
            class="mt-2 px-3 py-1 bg-red-500 text-white rounded-lg text-sm hover:bg-red-600"
          >
            ลบรูป
          </button>
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="flex gap-2 justify-end">
        <router-link
          to="/books"
          class="px-4 py-2 bg-red-500 rounded-lg text-white hover:bg-red-700"
        >
          ยกเลิก
        </router-link>
        <button
          type="submit"
          :disabled="isLoading"
          class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          <span v-if="!isLoading">บันทึก</span>
          <span v-else>กำลังบันทึก...</span>
        </button>
      </div>
    </form>
  </div>
</template>

