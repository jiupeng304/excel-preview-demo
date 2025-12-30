<template>
  <div class="vue-office-excel-preview" ref="containerRef">
    <vue-office-excel
      v-if="shouldRender"
      :src="file"
      class="excel-viewer"
      style="height: 100%;"
      @rendered="onRendered"
      @error="onError"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import VueOfficeExcel from '@vue-office/excel'
import '@vue-office/excel/lib/index.css'

defineProps<{
  file: any // ArrayBuffer or string
}>()

const shouldRender = ref(false)
const containerRef = ref<HTMLElement | null>(null)

const onRendered = () => {
  console.log('Rendering completed')
}

const onError = (e: Error) => {
  console.error('Rendering error', e)
}

onMounted(() => {
  if (containerRef.value) {
    const observer = new ResizeObserver((entries) => {
      for (const entry of entries) {
        if (entry.contentRect.height > 0 && entry.contentRect.width > 0) {
           if (!shouldRender.value) {
             shouldRender.value = true
           }
        }
      }
    })
    observer.observe(containerRef.value)
  }
})
</script>

<style scoped>
.vue-office-excel-preview {
  height: 100%;
}
.excel-viewer {
  height: 100%;
}
</style>
