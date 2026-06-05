<script setup>
import { ref } from 'vue'
import UnityViewer from './components/UnityViewer.vue'

const viewerRef = ref(null)
const isThirdPersonActive = ref(false)
const isPause = ref(false)

function handleMounted() {
  console.log('Unity instance mounted')
}

function handleProgress(p) {
  console.log(`Loading progress: ${p}%`)
}

function handleError(err) {
  console.error('Unity error:', err)
}

function sendToUnity() {
  viewerRef.value?.sendMessage('JSBridge', "JS_BtnClick_Back")
}

function toggleFullscreen() {
  viewerRef.value?.setFullscreen(true)
}

// --- 新增：重置视角函数 ---
function resetView() {
  console.log('Reset view')
  viewerRef.value?.sendMessage('JSBridge', "JS_BtnClick_Reset")
}

function backbtnclick1() {
     viewerRef.value?.sendMessage('JSBridge', "JS_BtnClick_HighLight",1)
}

function backbtnclick2() {
    viewerRef.value?.sendMessage('JSBridge', "JS_BtnClick_HighLight",2)
}

function backbtnc3() {
   viewerRef.value?.sendMessage('JSBridge', "JS_BtnClick_HighLight",3)
}

function handleLogMessage(id) {
  console.log('LogMessage from Unity:', id)
     viewerRef.value?.sendMessage('JSBridge', "JS_Rec_Data",'{"id":"1ba5a3f36def435aaf34d7263216a3ca","dcuId":"535cddaa9d924ebf979bbb71f617f026","name":"2#厂房KTAT楼顶电梯井","commAddr":"115230801951","eqTypeDic":"02","eqTypeDicName":"MY box"}')
}

function LogMessage(id) {
  console.log('LogMessage from Unity:', id)
}


function  pageFlag(flag){
  console.log('FLAG LogMessage from Unity:', flag)
}

// 整合逻辑，确保状态与发送消息同步
function toggleThirdPerson() {
  isThirdPersonActive.value = !isThirdPersonActive.value
  // 注意：根据你的原逻辑，true时发送0，false时发送1
  const param = isThirdPersonActive.value ? 0 : 1
  viewerRef.value?.sendMessage('JSBridge', "AutoMove", param)
}

function togglePause() {
  isPause.value = !isPause.value
  // 注意：根据你的原逻辑，true时发送0，false时发送1
  const param = isPause.value ? 0 : 1
  viewerRef.value?.sendMessage('JSBridge', "PauseMove", param)
}
</script>

<template>
  <div class="app">
    <div class="toolbar">
      <!-- 基础控制组 -->
      <div class="button-group">
        <button @click="sendToUnity" class="btn-primary">Back</button>
        <button @click="resetView" class="btn-primary">重置视角</button> <!-- 添加的按钮 -->
        <button @click="toggleFullscreen" class="btn-primary">全屏</button>
      </div>
      
      <div class="divider"></div>

      <!-- 场景标记组 -->
      <div class="button-group">
        <button @click="backbtnclick1">1</button>
        <button @click="backbtnclick2">2</button>
        <button @click="backbtnc3">3</button>
      </div>

      <div class="divider"></div>

      <!-- iOS 风格开关组 -->
      <div class="switch-group">
        <div class="switch-container">
          <label class="ios-switch">
            <input type="checkbox" :checked="isThirdPersonActive" @change="toggleThirdPerson">
            <span class="slider"></span>
            <span class="label-text">第三人称</span>
          </label>
        </div>

        <div class="switch-container">
          <label class="ios-switch">
            <input type="checkbox" :checked="isPause" @change="togglePause">
            <span class="slider"></span>
            <span class="label-text">暂停移动</span>
          </label>
        </div>
      </div>
    </div>
    
    <div class="unity-container">
      <UnityViewer
        ref="viewerRef"
        @mounted="handleMounted"
        @progress="handleProgress"
        @error="handleError"
        @log-message="handleLogMessage"
        @page-flag="pageFlag"
      />
    </div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  width: 100%;
  height: 100%;
  overflow: hidden;
}
</style>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  background: #1a1a2e;
}

.toolbar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #16213e;
  flex-wrap: wrap;
}

.button-group, .switch-group {
  display: flex;
  gap: 8px;
  align-items: center;
}

.toolbar button {
  padding: 6px 12px;
  border: 1px solid #42b883;
  background: transparent;
  color: #42b883;
  border-radius: 4px;
  cursor: pointer;
  font-size: 13px;
  transition: all 0.2s;
}

.toolbar button:hover {
  background: #42b883;
  color: #1a1a2e;
}

.btn-primary {
  border-color: #42b883;
  color: #42b883;
}

.divider {
  width: 1px;
  height: 24px;
  background: rgba(255, 255, 255, 0.2);
  margin: 0 4px;
}

/* --- iOS Switch 样式 --- */
.switch-container {
  display: flex;
  align-items: center;
}

.ios-switch {
  display: flex;
  align-items: center;
  cursor: pointer;
  user-select: none;
  position: relative;
}

.ios-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: relative;
  display: inline-block;
  width: 40px;
  height: 22px;
  background-color: #ccc;
  border-radius: 22px;
  transition: .4s;
  margin-right: 8px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 2px;
  bottom: 2px;
  background-color: white;
  border-radius: 50%;
  transition: .4s;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

input:checked + .slider {
  background-color: #42b883;
}

input:checked + .slider:before {
  transform: translateX(18px);
}

.label-text {
  color: #e0e0e0;
  font-size: 13px;
  white-space: nowrap;
  margin-right: 10px;
}

.unity-container {
  flex: 1;
  position: relative;
}
</style>
