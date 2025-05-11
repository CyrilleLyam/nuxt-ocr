<template>
  <div>
    <Header />
    <div class="flex flex-col my-16 mx-8 gap-8 md:flex-row md:justify-center md:items-start">
      <div
        @dragover.prevent
        @drop.prevent="handleDrop"
        class="md:w-1/2 flex flex-col items-center justify-center border-2 border-dashed border-blue-100 hover:border-blue-500 rounded-xl p-8 cursor-pointer"
        @click="triggerFileInput"
      >
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
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import { createWorker } from 'tesseract.js'

export default {
  name: 'ImageUploadOCR',
  setup() {
    const fileInput = ref(null)
    const imageUrl = ref(null)
    const ocrText = ref('')
    const isScanning = ref(false)
    let worker = null

    const initWorker = async () => {
      worker = await createWorker()
      await worker.loadLanguage('eng')
      await worker.initialize('eng')
    }

    const terminateWorker = async () => {
      if (worker) {
        await worker.terminate()
        worker = null
      }
    }

    const triggerFileInput = () => fileInput.value?.click()

    const handleFile = (file) => {
      if (!file) return
      imageUrl.value = URL.createObjectURL(file)
    }

    const handleFileChange = (event) => handleFile(event.target.files[0])
    const handleDrop = (event) => handleFile(event.dataTransfer.files[0])

    const performOCR = async () => {
      if (!imageUrl.value || !worker) return
      isScanning.value = true
      ocrText.value = ''
      try {
        const { data: { text } } = await worker.recognize(imageUrl.value)
        ocrText.value = text
      } catch {
        ocrText.value = 'Error recognizing text'
      } finally {
        isScanning.value = false
      }
    }

    watch(imageUrl, () => {
      if (imageUrl.value) performOCR()
    })

    onMounted(() => initWorker())
    onBeforeUnmount(() => terminateWorker())

    return {
      fileInput,
      imageUrl,
      ocrText,
      isScanning,
      triggerFileInput,
      handleFileChange,
      handleDrop
    }
  }
}
</script>
