<template>
  <div class="vue-office-excel-preview">
    <vue-office-excel
      :src="file"
      class="excel-viewer"
      style="height: 100%;"
      @rendered="onRendered"
      @error="onError"
    />
  </div>
</template>

<script setup lang="ts">
import VueOfficeExcel from '@vue-office/excel'
import '@vue-office/excel/lib/index.css'

defineProps<{
  file: any // ArrayBuffer or string
}>()

const onRendered = () => {
  console.log('Rendering completed')
}

const onError = (e: Error) => {
  console.error('Rendering error', e)
}
</script>

<style scoped>
.vue-office-excel-preview {
  height: 100%;
}
.excel-viewer {
  height: 100%;
}
</style>
