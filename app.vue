<template>
  <div>
    <div
      class="sticky top-0 z-30 flex h-16 w-full justify-center bg-opacity-90 backdrop-blur bg-base-100 text-base-content border-b border-slate-800/10">
      <nav class="navbar px-2 w-full flex items-center">
        <a href="/" aria-current="page" aria-label="Homepage" class="font-title inline-flex text-lg">
          <span class="capitalize">Nuxt</span>
          <span class="uppercase font-bold text-blue-500">OCR</span>
        </a>
      </nav>
    </div>

    <div class="flex flex-col my-16 mx-8 gap-8 md:flex-row md:justify-center md:items-start">
      <div @dragover.prevent @drop.prevent="handleDrop"
        class="md:w-1/2 flex flex-col items-center justify-center border-2 border-dashed border-blue-100 hover:border-blue-500 rounded-xl p-8 cursor-pointer"
        @click="triggerFileInput">
        <input type="file" ref="fileInput" class="hidden" accept="image/*" @change="handleFileChange" />
        <div v-if="imageUrl">
          <img :src="imageUrl" alt="Uploaded Image" class="max-w-full max-h-96 object-contain rounded-lg shadow" />
        </div>
        <div v-else class="text-center">
          <p class="text-blue-500 text-4xl">📥</p>
          <p class="mt-2 text-gray-700 text-sm">Click or drag & drop an image here</p>
        </div>
      </div>

      <div class="md:w-1/2 bg-gray-100 p-4 rounded-md shadow text-sm whitespace-pre-wrap border border-slate-300">
        <h3 class="font-semibold mb-4 text-blue-500">OCR Result:</h3>
        <div v-if="isScanning" class="text-gray-500 italic">Scanning image... please wait.</div>
        <div v-else>{{ ocrText }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue'
import Tesseract from 'tesseract.js'

export default {
  name: 'ImageUploadOCR',
  setup() {
    const fileInput = ref(null)
    const imageUrl = ref(null)
    const imageFile = ref(null)
    const ocrText = ref('')
    const isScanning = ref(false)

    const triggerFileInput = () => {
      fileInput.value.click()
    }

    const handleFileChange = (event) => {
      const file = event.target.files[0]
      if (file) {
        imageUrl.value = URL.createObjectURL(file)
        imageFile.value = file
      }
    }

    const handleDrop = (event) => {
      const file = event.dataTransfer.files[0]
      if (file) {
        imageUrl.value = URL.createObjectURL(file)
        imageFile.value = file
      }
    }

    const performOCR = async () => {
      if (!imageFile.value) return
      isScanning.value = true
      ocrText.value = ''
      try {
        const result = await Tesseract.recognize(imageFile.value, 'eng', {
          logger: m => console.log(m)
        })
        ocrText.value = result.data.text
      } catch (err) {
        ocrText.value = 'Error recognizing text'
      } finally {
        isScanning.value = false
      }
    }

    watch(imageFile, () => {
      if (imageFile.value) performOCR()
    })

    return {
      fileInput,
      imageUrl,
      imageFile,
      ocrText,
      isScanning,
      triggerFileInput,
      handleFileChange,
      handleDrop
    }
  }
}
</script>
