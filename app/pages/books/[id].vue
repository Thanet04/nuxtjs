<script lang="ts">
export default {
  data() {
    return {
      book: null,
      coverFile: null,
      coverPreview: '',
      notFound: false,
      isLoading: false
    }
  },
  mounted() {
    this.loadBook()
  },
  methods: {
    async loadBook() {
      const id = this.$route.params.id
      try {
        const res = await fetch(`http://localhost:8080/api/books/${id}`)
        if (!res.ok) {
          this.notFound = true
          return
        }
        const data = await res.json()
        this.book = data
        this.coverPreview = data.imageUrl || ''
      } catch (err) {
        console.error(err)
        this.notFound = true
      }
    },
    onFileChange(event) {
      const file = event.target.files && event.target.files[0]
      if (!file) return
      this.coverFile = file
      const reader = new FileReader()
      reader.onload = () => {
        this.coverPreview = String(reader.result || '')
      }
      reader.readAsDataURL(file)
    },
    async save() {
      if (!this.book.title || !this.book.author || !this.book.price) {
        alert('กรุณากรอกข้อมูลให้ครบถ้วน')
        return
      }
      this.isLoading = true
      try {
        const formData = new FormData()
        const bookData = {
          title: this.book.title,
          author: this.book.author,
          price: Number(this.book.price)
        }
        formData.append('book', new Blob([JSON.stringify(bookData)], { type: 'application/json' }))
        if (this.coverFile) formData.append('file', this.coverFile)

        const res = await fetch(`http://localhost:8080/api/books/${this.book.id}`, {
          method: 'PUT',
          body: formData
        })

        if (!res.ok) throw new Error('ไม่สามารถบันทึกได้')

        alert('อัปเดตหนังสือสำเร็จ!')
        this.$router.push('/books')
      } catch (err) {
        console.error(err)
        alert(err.message)
      } finally {
        this.isLoading = false
      }
    }
  }
}
</script>

<template>
  <div class="max-w-xl mx-auto p-4 h-full">
    <div v-if="notFound">ไม่พบหนังสือ</div>
    <div v-else-if="!book">กำลังโหลด...</div>
    <div v-else class="space-y-4">
      <h1 class="text-2xl font-bold">แก้ไขหนังสือ</h1>

      <div>
        <label class="block mb-1 font-medium">ชื่อหนังสือ</label>
        <input v-model="book.title" type="text" class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500" />
      </div>

      <div>
        <label class="block mb-1 font-medium">ผู้เขียน</label>
        <input v-model="book.author" type="text" class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500" />
      </div>

      <div>
        <label class="block mb-1 font-medium">ราคา</label>
        <input v-model="book.price" type="number" class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500" />
      </div>

      <div>
        <label class="block mb-1 font-medium">รูปปกหนังสือ</label>
        <input type="file" accept="image/*" @change="onFileChange" class="block w-full text-sm mb-2" />
        <img v-if="coverPreview" :src="coverPreview" alt="cover" class="w-full h-48 object-cover rounded border" />
      </div>

      <div class="flex justify-end  gap-2">
        <router-link to="/books"
          class="px-4 py-2 bg-red-500 rounded-lg text-white hover:bg-red-700" >
          ยกเลิก
        </router-link>
        <button @click="save" :disabled="isLoading" class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50">
          {{ isLoading ? 'กำลังบันทึก...' : 'บันทึก' }}
        </button>
      </div>
    </div>
  </div>
</template>
