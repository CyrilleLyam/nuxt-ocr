<template>
  <div class="bg-gray-100 p-4 rounded-xl shadow text-sm whitespace-pre-wrap border border-slate-300">
    <h3 class="font-semibold mb-4 text-blue-500">OCR Result:</h3>
    <div v-if="isScanning" class="text-gray-500 italic">Scanning image... please wait.</div>
    <div v-else>{{ ocrText || 'No image uploaded yet' }}</div>
  </div>
</template>

<script>
export default {
  props: {
    ocrText: String,
    isScanning: Boolean
  }
}
</script>
