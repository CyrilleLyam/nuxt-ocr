<template>
  <div @dragover.prevent @drop.prevent="onDrop"
    class="flex flex-col items-center justify-center border-2 border-dashed border-blue-500 rounded-xl p-8 cursor-pointer hover:bg-blue-50"
    @click="triggerInput">
    <input type="file" ref="fileInput" class="hidden" accept="image/*" @change="onFileChange" />
    <div v-if="imageUrl">
      <img :src="imageUrl" alt="Uploaded Image" class="max-w-full max-h-96 object-contain rounded-lg shadow" />
    </div>
    <div v-else class="text-center">
      <p class="text-blue-500 text-4xl">📥</p>
      <p class="mt-2 text-gray-700 text-sm">Click or drag & drop an image here</p>
    </div>
    <button class="mt-4 px-6 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 disabled:bg-gray-300"
      :disabled="!imageFile || isScanning" @click="$emit('perform-ocr')">
      {{ isScanning ? 'Scanning...' : 'Extract Text' }}
    </button>
  </div>
</template>

<script>
import { ref, watch } from 'vue'

export default {
  props: {
    imageUrl: String,
    isScanning: Boolean
  },
  emits: ['image-change', 'perform-ocr'],
  setup(props, { emit }) {
    const fileInput = ref(null)
    const imageFile = ref(null)

    const triggerInput = () => fileInput.value.click()

    const onFileChange = (e) => {
      const file = e.target.files[0]
      if (file) {
        emit('image-change', file)
      }
    }

    const onDrop = (e) => {
      const file = e.dataTransfer.files[0]
      if (file) {
        emit('image-change', file)
      }
    }

    return {
      fileInput,
      imageFile,
      triggerInput,
      onFileChange,
      onDrop
    }
  }
}
</script>
