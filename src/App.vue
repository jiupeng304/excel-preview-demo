<template>
  <div class="app-container">
    <div class="header">
      <h1>Excel Preview Demo</h1>
      <el-upload
        class="upload-demo"
        action="#"
        :auto-upload="false"
        :on-change="handleChange"
        :show-file-list="false"
        accept=".xlsx,.xls"
      >
        <el-button type="primary">Select Excel File</el-button>
        <template #tip>
          <div class="el-upload__tip">
            jpg/png files with a size less than 500kb
          </div>
        </template>
      </el-upload>
      <div v-if="fileName" class="file-info">
        Current File: {{ fileName }}
      </div>
      <a href="./excel-preview.html" target="_blank" style="margin-left: auto;">
        <el-button type="success" plain>跳转静态演示页 (excel-preview.html)</el-button>
      </a>
    </div>

    <div class="preview-area">
      <el-tabs v-model="activeTab" type="card" class="demo-tabs">
        <el-tab-pane label="Vue Office Excel" name="vue-office">
           <PreviewVueOffice v-if="fileArrayBuffer" :file="fileArrayBuffer" />
           <div v-else class="empty-state">Please upload a file</div>
        </el-tab-pane>
        <el-tab-pane label="SheetJS + Table" name="sheetjs">
           <PreviewSheetJS v-if="fileArrayBuffer" :file="fileArrayBuffer" />
           <div v-else class="empty-state">Please upload a file</div>
        </el-tab-pane>
        <el-tab-pane label="Luckysheet" name="luckysheet">
            <PreviewLuckysheet v-if="rawFile" :file="rawFile" />
            <div v-else class="empty-state">Please upload a file</div>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import type { UploadFile } from 'element-plus'
import PreviewVueOffice from './components/PreviewVueOffice.vue'
import PreviewSheetJS from './components/PreviewSheetJS.vue'
import PreviewLuckysheet from './components/PreviewLuckysheet.vue'

const activeTab = ref('vue-office')
const fileName = ref('')
const fileArrayBuffer = ref<ArrayBuffer | null>(null)
const rawFile = ref<File | null>(null)

const handleChange = (uploadFile: UploadFile) => {
  if (uploadFile.raw) {
    fileName.value = uploadFile.name
    rawFile.value = uploadFile.raw
    
    // Read as ArrayBuffer for VueOffice and SheetJS
    const reader = new FileReader()
    reader.onload = (e) => {
      // Use nextTick to allow the UI to update (e.g. filename appearing) before rendering components
      // This helps ensure correct height calculations
      setTimeout(() => {
        if (e.target?.result) {
          fileArrayBuffer.value = e.target.result as ArrayBuffer
        }
      }, 100)
    }
    reader.readAsArrayBuffer(uploadFile.raw)
  }
}
</script>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  padding: 20px;
  box-sizing: border-box;
}

.header {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 20px;
}

.preview-area {
  flex: 1;
  overflow: hidden;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
}

.demo-tabs {
  height: 100%;
  display: flex;
  flex-direction: column;
}

:deep(.el-tabs__content) {
  flex: 1;
  position: relative; /* For luckysheet absolute positioning */
  padding: 0 !important;
  /* overflow: auto;  <-- Removed global auto overflow to check if it helps Luckysheet */
  overflow: hidden; /* Force hidden, let children handle scroll if needed */
}

/* Specific overrides for tab panes might be needed if they rely on this overflow */
:deep(.el-tab-pane) {
    height: 100%;
}


.empty-state {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  color: #909399;
}
</style>
