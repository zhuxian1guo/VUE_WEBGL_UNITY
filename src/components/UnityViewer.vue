<script setup>
import { ref, onBeforeUnmount, onMounted, nextTick } from 'vue'
import UnityWebgl from 'unity-webgl'
import UnityVue from 'unity-webgl/vue'

const props = defineProps({
  loaderUrl: { type: String, default: '/unity-build/Build/Build/WEBGL.loader.js' },
  dataUrl: { type: String, default: '/unity-build/Build/Build/WEBGL.data.unityweb' },
  frameworkUrl: { type: String, default: '/unity-build/Build/Build/WEBGL.framework.js.unityweb' },
  codeUrl: { type: String, default: '/unity-build/Build/Build/WEBGL.wasm.unityweb' },
  streamingAssetsUrl: { type: String, default: '/unity-build/Build/Build/StreamingAssets' },
  width: { type: [String, Number], default: '100%' },
  height: { type: [String, Number], default: '100%' },
  companyName: { type: String, default: 'DefaultCompany' },
  productName: { type: String, default: 'My project (3)' },
  productVersion: { type: String, default: '0.1' },
})

const emit = defineEmits(['mounted', 'progress', 'error', 'unmounted', 'logMessage', 'pageFlag'])

const loading = ref(true)
const progress = ref(0)
const errorMsg = ref('')
const viewerRef = ref(null)

const unity = new UnityWebgl({
  loaderUrl: props.loaderUrl,
  dataUrl: props.dataUrl,
  frameworkUrl: props.frameworkUrl,
  codeUrl: props.codeUrl,
  streamingAssetsUrl: props.streamingAssetsUrl,
  companyName: props.companyName,
  productName: props.productName,
  productVersion: props.productVersion,
})

function resizeCanvas() {
  const canvas = unity._unity?.Module?.canvas
  if (!canvas || !viewerRef.value) return
  const { clientWidth, clientHeight } = viewerRef.value
  const dpr = window.devicePixelRatio || 1
  canvas.width = clientWidth * dpr
  canvas.height = clientHeight * dpr
  canvas.style.width = clientWidth + 'px'
  canvas.style.height = clientHeight + 'px'
}

unity.on('mounted', () => {
  loading.value = false
  emit('mounted')
  nextTick(() => resizeCanvas())

  window.LogMessage = (id) => {
    emit('logMessage', id)
  }

  window.pageFlag = (id) => {
    emit('pageFlag', id)
  }
})

unity.on('progress', (p) => {
  progress.value = Math.round(p * 100)
  emit('progress', progress.value)
})

unity.on('error', (err) => {
  errorMsg.value = typeof err === 'string' ? err : err.message
  loading.value = false
  emit('error', errorMsg.value)
})

unity.on('unmounted', () => {
  emit('unmounted')
})

let resizeObserver = null

onMounted(() => {
  if (viewerRef.value) {
    resizeObserver = new ResizeObserver(() => resizeCanvas())
    resizeObserver.observe(viewerRef.value)
  }
})

function sendMessage(objectName, methodName, value) {
  unity.sendMessage(objectName, methodName, value)
}

function setFullscreen(enabled) {
  unity.setFullscreen(enabled)
}

function unload() {
  return unity.unload()
}

defineExpose({ sendMessage, setFullscreen, unload, unity })

onBeforeUnmount(() => {
  resizeObserver?.disconnect()
  delete window.LogMessage
  delete window.pageFlag
  unity.unsafe_unload()
})
</script>

<template>
  <div ref="viewerRef" class="unity-viewer">
    <div v-if="loading" class="unity-loading">
      <div class="loading-bar">
        <div class="loading-fill" :style="{ width: progress + '%' }"></div>
      </div>
      <p class="loading-text">{{ errorMsg ? errorMsg : `Loading... ${progress}%` }}</p>
    </div>
    <UnityVue :unity="unity" :width="width" :height="height" tabindex="-1" />
  </div>
</template>

<style scoped>
.unity-viewer {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}

.unity-viewer canvas {
  display: block;
  width: 100% !important;
  height: 100% !important;
}

.unity-loading {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #231f20;
  z-index: 10;
}

.loading-bar {
  width: 300px;
  height: 6px;
  background: #333;
  border-radius: 3px;
  overflow: hidden;
}

.loading-fill {
  height: 100%;
  background: #42b883;
  transition: width 0.3s ease;
}

.loading-text {
  margin-top: 12px;
  color: #ccc;
  font-size: 14px;
}
</style>
