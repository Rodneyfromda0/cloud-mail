<template>
  <el-container class="layout">
    <el-aside
        class="aside"
        :class="uiStore.asideShow ? 'aside-show' : 'el-aside-hide'">
      <Aside />
    </el-aside>
    <div
        :class="(uiStore.asideShow && isMobile)? 'overlay-show':'overlay-hide'"
        @click="uiStore.asideShow = false"
    ></div>
    <el-container class="main-container">
      <el-main>
        <el-header>
            <Header />
        </el-header>
        <Main />
      </el-main>
    </el-container>
  </el-container>
  <writer ref="writerRef" />
</template>
<script setup>
import Aside from '@/layout/aside/index.vue'
import Header from '@/layout/header/index.vue'
import Main from '@/layout/main/index.vue'
import { ref, onMounted, onBeforeUnmount } from 'vue'
import {useUiStore} from "@/store/ui.js";
import {hasPerm} from "@/perm/perm.js";
import writer from '@/layout/write/index.vue'

const uiStore = useUiStore();
const writerRef = ref({})
const isMobile = ref(window.innerWidth < 1025)
function handleComposeShortcut(event) {
  const target = event.target
  const isTyping = target instanceof HTMLElement
    && (target.isContentEditable || target.matches('input, textarea, select, [contenteditable="true"]'))
  if (event.key.toLowerCase() !== 'c' || event.altKey || event.ctrlKey || event.metaKey || isTyping) return
  if (!hasPerm('email:send')) return
  event.preventDefault()
  uiStore.writerRef?.open()
}
const handleResize = () => {
  const nextIsMobile = window.innerWidth < 1025
  if (nextIsMobile === isMobile.value) return
  isMobile.value = nextIsMobile
  uiStore.asideShow = !nextIsMobile
}

onMounted(() => {
  uiStore.writerRef = writerRef

  window.addEventListener('resize', handleResize)
  window.addEventListener('keydown', handleComposeShortcut)
  handleResize()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
  window.removeEventListener('keydown', handleComposeShortcut)
})
</script>
<style lang="scss" scoped>
.el-aside-hide {
  position: fixed;
  left: 0;
  height: 100%;
  z-index: 100;
  transform: translateX(-100%);
  transition: transform 220ms ease;
}

.aside-show {
  -webkit-box-shadow: var(--aside-right-border);
  box-shadow: var(--aside-right-border);
  transform: translateX(0);
  transition: transform 220ms ease;
  z-index: 101;
  @media (max-width: 1025px) {
    position: fixed;
    top: 0;
    left: 0;
    z-index: 101;
    height: 100%;
    background: var(--el-bg-color);
  }
}

.el-aside {
  width: auto;
  transition: all 100ms ease;
}

.layout {
  height: 100vh;
  height: 100svh;
  height: 100svh;
  min-height: 0;
  position: fixed;
  width: 100%;
  top: 0;
  left: 0;
  overflow: hidden;
}

.main-container {
  min-height: 0;
  height: 100%;
  background: var(--el-bg-color);
  overflow: hidden;
  -webkit-overflow-scrolling: touch;
}

.el-main {
  min-height: 0;
  height: 100%;
  padding: 0;
  display: flex;
  flex-direction: column;
}

.el-header {
  flex: 0 0 60px;
  background: var(--el-bg-color);
  border-bottom: solid 1px var(--el-border-color);
  padding: 0;
  @media (max-width: 767px) {
    height: 50px;
    flex-basis: 50px;
  }
}

.overlay-show {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.4);
  z-index: 99;
  transition: all 0.3s;
}

.overlay-hide {
  display: flex;
  pointer-events: none;
  opacity: 0;
}
</style>
