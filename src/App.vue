<template>
  <div class="app-container min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-950 text-slate-50">
    <!-- 页面标题 -->
    <header class="py-6 px-4 text-center">
      <h1 class="text-3xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-blue-400 to-emerald-400">
        Still Alive 软件激活工具
      </h1>
      <p class="mt-2 text-slate-400">快速激活你的设备</p>
    </header>

    <main class="max-w-md mx-auto px-4 pb-16">
      <!-- 步骤 -->
      <div class="bg-slate-900/60 rounded-2xl border border-slate-800 shadow-xl overflow-hidden">
        <div class="p-6">

          <!-- STEP 1 -->
          <div v-if="currentStep === 1" class="step-content">
            <h2 class="text-xl font-bold mb-4 text-center">输入卡密</h2>
            <input
              v-model="key"
              @input="validateKey"
              class="w-full px-4 py-3 bg-slate-800 border border-slate-700 rounded-lg"
              placeholder="A1B2C3D4E5F6"
            />
            <p v-if="keyError" class="text-red-400 text-sm mt-2">{{ keyError }}</p>
          </div>

          <!-- STEP 2 -->
          <div v-if="currentStep === 2" class="step-content">
            <h2 class="text-xl font-bold mb-4 text-center">输入设备 ID</h2>
            <input
              v-model="deviceId"
              @input="validateDeviceId"
              class="w-full px-4 py-3 bg-slate-800 border border-slate-700 rounded-lg"
              placeholder="32 位十六进制"
            />
            <p v-if="deviceIdError" class="text-red-400 text-sm mt-2">{{ deviceIdError }}</p>
          </div>

          <!-- STEP 3 -->
          <div v-if="currentStep === 3" class="step-content">
            <h2 class="text-xl font-bold mb-4 text-center">确认信息</h2>
            <p class="mb-2"><span class="text-slate-400">卡密：</span>{{ key }}</p>
            <p><span class="text-slate-400">设备 ID：</span>{{ deviceId }}</p>
          </div>

          <!-- STEP 4 -->
          <div v-if="currentStep === 4" class="step-content">
            <h2 class="text-xl font-bold mb-4 text-center">激活数据</h2>
            <div class="flex gap-2">
              <input
                v-model="activationData"
                readonly
                class="flex-1 px-4 py-3 bg-slate-800 border border-slate-700 rounded-lg"
              />
              <button
                @click="copyActivationData"
                class="px-3 py-3 bg-slate-700 rounded-lg"
              >
                复制
              </button>
            </div>
          </div>

          <!-- STEP 5 -->
          <div v-if="currentStep === 5" class="step-content text-center">
            <h2 class="text-xl font-bold mb-2">完成</h2>
            <p class="text-slate-400">激活数据已生成</p>
            <button
              @click="resetStepper"
              class="mt-6 w-full py-3 bg-gradient-to-r from-blue-600 to-emerald-500 rounded-lg"
            >
              激活另一台
            </button>
          </div>
        </div>

        <!-- 底部按钮 -->
        <div v-if="currentStep < 5" class="flex justify-between px-6 py-4 border-t border-slate-800">
          <button
            @click="prevStep"
            :disabled="currentStep === 1"
            class="px-4 py-2 bg-slate-800 rounded-lg"
          >
            上一步
          </button>

          <button
            @click="nextStep"
            :disabled="isNextDisabled || isProcessing"
            class="px-4 py-2 bg-gradient-to-r from-blue-600 to-emerald-500 rounded-lg"
          >
            {{ isProcessing ? '处理中…' : '下一步' }}
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const key = ref('')
const deviceId = ref('')
const activationData = ref('')
const keyError = ref('')
const deviceIdError = ref('')
const currentStep = ref(1)
const isProcessing = ref(false)
const cardHash = ref('')

// SHA256
const calculateCardHash = async (text: string) => {
  const data = new TextEncoder().encode(text)
  const hash = await crypto.subtle.digest('SHA-256', data)
  return Array.from(new Uint8Array(hash)).map(b => b.toString(16).padStart(2, '0')).join('')
}

// 校验
const validateKey = () => {
  if (!/^[A-Z0-9]{12}$/.test(key.value)) {
    keyError.value = '卡密格式错误'
    return false
  }
  keyError.value = ''
  return true
}

const validateDeviceId = () => {
  if (!/^[A-Fa-f0-9]{32}$/.test(deviceId.value)) {
    deviceIdError.value = '设备 ID 格式错误'
    return false
  }
  deviceIdError.value = ''
  return true
}

// API
const verifyCard = async () => {
  isProcessing.value = true
  cardHash.value = await calculateCardHash(key.value)

  const res = await fetch('https://api.verify.stillalive.asia', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      mode: 'verify-card',
      cardText: key.value,
      cardHash: cardHash.value,
    }),
  })

  const data = await res.json()
  isProcessing.value = false

  if (!data.success) {
    keyError.value = data.error || '验证失败'
    return false
  }
  return true
}

const requestSignature = async () => {
  isProcessing.value = true
  const res = await fetch('https://api.verify.stillalive.asia', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      mode: 'generate-signature',
      cardText: key.value,
      deviceId: deviceId.value,
      cardHash: cardHash.value,
    }),
  })
  const data = await res.json()
  isProcessing.value = false

  if (data.success) {
    activationData.value = data.resultPacket
    return true
  }

  deviceIdError.value = data.error || '生成失败'
  return false
}

// 控制
const isNextDisabled = computed(() => {
  if (currentStep.value === 1) return !validateKey()
  if (currentStep.value === 2) return !validateDeviceId()
  return false
})

const nextStep = async () => {
  if (currentStep.value === 1 && !(await verifyCard())) return
  if (currentStep.value === 2 && !(await requestSignature())) return
  currentStep.value++
}

const prevStep = () => currentStep.value--
const resetStepper = () => {
  key.value = ''
  deviceId.value = ''
  activationData.value = ''
  currentStep.value = 1
}

const copyActivationData = async () => {
  await navigator.clipboard.writeText(activationData.value)
}
</script>

<style>
.step-content {
  transition: all 0.3s ease;
}
</style>
