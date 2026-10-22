<template>
  <el-config-provider :locale="settingStore.lang === 'zh' ? zhCn : null">
    <router-view />
  </el-config-provider>
</template>
<script setup>
import { useI18n } from "vue-i18n";
import { watch } from "vue";
import {useSettingStore} from "@/store/setting.js";
const settingStore = useSettingStore()
import zhCn from 'element-plus/es/locale/lang/zh-cn';
import('@/icons/index.js')
const { locale } = useI18n()
locale.value = settingStore.lang
watch(() => settingStore.lang, () => locale.value = settingStore.lang)
</script>

<style>
:root {
  color-scheme: dark;
  --el-color-primary: #ff5500;
  --el-color-primary-dark-2: #d94700;
  --el-color-primary-light-3: #ff7b3d;
  --el-color-primary-light-5: #ff9b6b;
  --el-color-primary-light-7: #ffbd9e;
  --el-color-primary-light-9: #39241a;
  --el-bg-color: #181614;
  --el-bg-color-page: #181614;
  --el-bg-color-overlay: #211e1b;
  --el-text-color-primary: #f2ebe5;
  --el-text-color-regular: #c1b4aa;
  --el-text-color-secondary: #a99b91;
  --el-border-color: #403731;
  --el-border-color-light: #403731;
  --el-fill-color-blank: #211e1b;
}

html,
body,
#app {
  min-height: 100%;
  background: #181614;
}
</style>
