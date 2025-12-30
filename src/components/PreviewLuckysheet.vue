<template>
  <div class="luckysheet-preview">
    <div id="luckysheet" style="margin:0px;padding:0px;position:absolute;width:100%;left:0px;top:0px;bottom:0px;outline: none;"></div>
  </div>
</template>

<script setup lang="ts">
import { watch, onMounted, onBeforeUnmount } from 'vue'
// @ts-ignore
import LuckyExcel from 'luckyexcel'

const props = defineProps<{
  file: File | null
}>()

const initLuckysheet = () => {
    if(!props.file) return;
    
    LuckyExcel.transformExcelToLucky(props.file, function(exportJson: any, _luckysheetfile: any){
        if(exportJson.sheets==null || exportJson.sheets.length==0){
            alert("Failed to read the content of the excel file, currently does not support xls files!");
            return;
        }
        
        // @ts-ignore
        if(window.luckysheet) {
             // @ts-ignore
            window.luckysheet.destroy();
             // @ts-ignore
            window.luckysheet.create({
                container: 'luckysheet', // container id
                showinfobar: false,
                data:exportJson.sheets,
                title:exportJson.info.name,
                userInfo:exportJson.info.name.creator
            });
        }
    });
}

watch(() => props.file, () => {
    initLuckysheet();
})

onMounted(() => {
    if(props.file) {
        initLuckysheet();
    }
})

onBeforeUnmount(() => {
    // @ts-ignore
    if(window.luckysheet) {
         // @ts-ignore
        window.luckysheet.destroy();
    }
})
</script>

<style scoped>
.luckysheet-preview {
  position: absolute; /* Changed from relative to absolute to fill the tab pane */
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
