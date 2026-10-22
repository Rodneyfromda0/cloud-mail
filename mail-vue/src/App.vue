<template>
  <el-config-provider :locale="settingStore.lang === 'zh' ? zhCn : null">
    <router-view />
  </el-config-provider>
</template>
<script setup>
import { useI18n } from "vue-i18n";
import { watch } from "vue";
import {useSettingStore} from "@/store/setting.js";
import {useUiStore} from "@/store/ui.js";
const settingStore = useSettingStore()
const uiStore = useUiStore()
import zhCn from 'element-plus/es/locale/lang/zh-cn';
import('@/icons/index.js')
const { locale } = useI18n()
locale.value = settingStore.lang
watch(() => settingStore.lang, () => locale.value = settingStore.lang)
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
  min-height: 100%;
  background: var(--el-bg-color-page);
}
</style>
