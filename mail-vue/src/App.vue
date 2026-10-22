<template>
  <el-config-provider>
    <router-view />
  </el-config-provider>
</template>
<script setup>
import { useI18n } from "vue-i18n";
import { watch } from "vue";
import {useUiStore} from "@/store/ui.js";
const uiStore = useUiStore()
import('@/icons/index.js')
const { locale } = useI18n()
locale.value = 'en'
watch(() => uiStore.dark, (dark) => {
  document.documentElement.classList.toggle('dark', dark)
}, {immediate: true})
</script>

<style>
:root {
  --el-color-primary: #ff5500;
  --el-color-primary-dark-2: #d94700;
  --el-color-primary-light-3: #ff7b3d;
  --el-color-primary-light-5: #ff9b6b;
  --el-color-primary-light-7: #ffbd9e;
  --el-color-primary-light-9: #fff1e8;
}

html,
body,
#app {
  width: 100%;
  height: 100%;
  min-height: 100%;
  overflow: hidden;
  background: var(--el-bg-color-page);
}
</style>
