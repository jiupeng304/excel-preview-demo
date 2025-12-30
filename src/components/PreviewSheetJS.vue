<template>
  <div class="sheetjs-preview">
    <div v-if="loading">Loading...</div>
    <el-tabs v-if="workbook" v-model="activeSheetName" @tab-click="handleTabClick">
      <el-tab-pane
        v-for="sheetName in sheetNames"
        :key="sheetName"
        :label="sheetName"
        :name="sheetName"
      >
        <!-- Use a simple div if table is too heavy, but el-table is better for demo -->
      </el-tab-pane>
    </el-tabs>
    <el-table
      ref="tableRef"
      v-if="tableData.length"
      :data="tableData"
      border
      style="width: 100%; flex: 1; overflow: hidden;"
      height="100%"
    >
      <el-table-column
        v-for="header in tableHeaders"
        :key="header"
        :prop="header"
        :label="header"
        width="150"
      />
    </el-table>
     <div v-else-if="!loading && !workbook">Please upload a file</div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import * as XLSX from 'xlsx'

const props = defineProps<{
  file: ArrayBuffer | null
}>()

const workbook = ref<XLSX.WorkBook | null>(null)
const sheetNames = ref<string[]>([])
const activeSheetName = ref('')
const tableData = ref<any[]>([])
const tableHeaders = ref<string[]>([])
const loading = ref(false)

const processFile = () => {
  if (!props.file) return
  loading.value = true
  try {
    const wb = XLSX.read(props.file, { type: 'array' })
    workbook.value = wb
    sheetNames.value = wb.SheetNames
    if (sheetNames.value.length > 0) {
      activeSheetName.value = sheetNames.value[0] || ''
      loadSheet(activeSheetName.value)
    }
  } catch (err) {
    console.error(err)
  } finally {
    loading.value = false
  }
}

const tableRef = ref()

const loadSheet = async (sheetName: string) => {
  if (!workbook.value || !sheetName) return
  const ws = workbook.value.Sheets[sheetName]
  if (!ws) return
  const data = XLSX.utils.sheet_to_json(ws, { header: 1 }) as any[][]
  if (data && data.length > 0) {
    tableHeaders.value = data[0]?.map((h: any) => String(h)) || []
    // Transform rest of rows to object
    const rows = data.slice(1).map(row => {
      const obj: any = {}
      tableHeaders.value.forEach((header, index) => {
        obj[header] = row[index]
      })
      return obj
    })
    tableData.value = rows
    
    // Force layout update next tick
    setTimeout(() => {
      if (tableRef.value) {
        tableRef.value.doLayout()
      }
    }, 0)
  } else {
    tableHeaders.value = []
    tableData.value = []
  }
}

const handleTabClick = (tab: any) => {
  loadSheet(tab.props.name)
}

watch(() => props.file, processFile)
onMounted(() => {
  if (props.file) processFile()
})
</script>

<style scoped>
.sheetjs-preview {
  height: 100%;
  display: flex;
  flex-direction: column;
}
</style>
