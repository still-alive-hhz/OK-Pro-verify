<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-950 text-slate-50">
    <!-- 页面标题 -->
    <header class="py-6 text-center">
      <h1 class="text-3xl font-bold text-cyan-400">
        Still Alive 软件激活工具
      </h1>
      <p class="mt-2 text-slate-400">快速激活你的设备</p>
    </header>

    <!-- 主内容 -->
    <main class="max-w-md mx-auto px-4 pb-16">
      <div class="bg-slate-900/70 rounded-2xl border border-slate-800 shadow-xl overflow-hidden">

        <!-- STEP 1 -->
        <div v-if="currentStep === 1" class="p-6">
          <h2 class="text-xl font-bold mb-2 text-center">输入卡密</h2>
          <p class="text-slate-400 text-sm mb-4 text-center">
            请输入你购买的 12 位激活码
          </p>

          <input
            v-model="key"
            @input="validateKey"
            class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-700"
            placeholder="A1B2C3D4E5F6"
          />
          <p v-if="keyError" class="mt-2 text-red-400 text-sm">
            {{ keyError }}
          </p>
        </div>

        <!-- STEP 2 -->
        <div v-if="currentStep === 2" class="p-6">
          <h2 class="text-xl font-bold mb-2 text-center">输入设备 ID</h2>
          <p class="text-slate-400 text-sm mb-4 text-center">
            32 位十六进制设备 ID
          </p>

          <input
            v-model="deviceId"
            @input="validateDeviceId"
            class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-700 text-sm"
            placeholder="32位十六进制字符串"
          />
          <p v-if="deviceIdError" class="mt-2 text-red-400 text-sm">
            {{ deviceIdError }}
          </p>
        </div>

        <!-- STEP 3 -->
        <div v-if="currentStep === 3" class="p-6">
          <h2 class="text-xl font-bold mb-4 text-center">激活数据</h2>

          <textarea
            readonly
            rows="4"
            v-model="activationData"
            class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-700 text-sm"
          />

          <button
            class="mt-4 w-full py-3 rounded-lg bg-gradient-to-r from-blue-600 to-emerald-500 text-white"
            @click="copyActivationData"
          >
            复制激活数据
          </button>

          <p v-if="copySuccess" class="mt-2 text-center text-emerald-400 text-sm">
            已复制到剪贴板
          </p>

          <button
            class="mt-4 w-full py-2 rounded-lg bg-slate-800 text-slate-300"
            @click="resetStepper"
          >
            激活另一台设备
          </button>
        </div>

        <!-- 底部按钮 -->
        <div
          v-if="currentStep < 3"
          class="px-6 py-4 border-t border-slate-800 flex justify-between gap-4"
        >
          <button
            @click="prevStep"
            :disabled="currentStep === 1"
            class="flex-1 py-2 rounded-lg bg-slate-800 disabled:opacity-40"
          >
            上一步
          </button>

          <button
            @click="nextStep"
            :disabled="isNextDisabled"
            class="flex-1 py-2 rounded-lg bg-gradient-to-r from-blue-600 to-emerald-500 text-white disabled:opacity-40"
          >
            {{ isProcessing ? '处理中…' : '下一步' }}
          </button>
        </div>

      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const key = ref('')
const deviceId = ref('')
const activationData = ref('')
const cardHash = ref('')

const keyError = ref('')
const deviceIdError = ref('')

const currentStep = ref(1)
const isProcessing = ref(false)
const copySuccess = ref(false)

/* ===== 工具函数 ===== */
const calculateCardHash = async (text: string): Promise<string> => {
  const buf = await crypto.subtle.digest(
    'SHA-256',
    new TextEncoder().encode(text)
  )
  return Array.from(new Uint8Array(buf))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('')
}

/* ===== 校验 ===== */
const validateKey = () => {
  if (!/^[A-Z0-9]{12}$/.test(key.value)) {
    keyError.value = '卡密格式不正确，应为12位大写字母或数字'
    return false
  }
  keyError.value = ''
  return true
}

const validateDeviceId = () => {
  if (!/^[A-Fa-f0-9]{32}$/.test(deviceId.value)) {
    deviceIdError.value = '设备ID格式不正确'
    return false
  }
  deviceIdError.value = ''
  return true
}

/* ===== 核心：验证卡密真实性 ===== */
const verifyCard = async (cardText: string, hash: string) => {
  try {
    isProcessing.value = true
    const res = await fetch('https://api.verify.stillalive.asia', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        mode: 'verify-card',
        cardText,
        cardHash: hash,
      }),
    })
    const data = await res.json()

    if (res.ok && data.success) return true
    if (data.isCardUsed) {
      keyError.value = '该卡密已使用，可重新生成激活数据'
      return true
    }
    if (data.isCardInvalid) {
      keyError.value = '卡密无效'
      return false
    }

    keyError.value = data.error || '卡密验证失败'
    return false
  } catch {
    keyError.value = '网络错误'
    return false
  } finally {
    isProcessing.value = false
  }
}

/* ===== 请求激活数据 ===== */
const requestSignature = async () => {
  try {
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

    if (res.ok && data.success) {
      activationData.value = data.resultPacket
      return true
    }

    deviceIdError.value = data.error || '生成失败'
    return false
  } finally {
    isProcessing.value = false
  }
}

/* ===== 步骤控制 ===== */
const isNextDisabled = computed(() => {
  if (isProcessing.value) return true
  if (currentStep.value === 1) return !validateKey()
  if (currentStep.value === 2) return !validateDeviceId()
  return false
})

const nextStep = async () => {
  if (currentStep.value === 1) {
    const hash = await calculateCardHash(key.value)
    cardHash.value = hash
    const ok = await verifyCard(key.value, hash)
    if (!ok) return
    currentStep.value = 2
    return
  }

  if (currentStep.value === 2) {
    const ok = await requestSignature()
    if (!ok) return
    currentStep.value = 3
  }
}

const prevStep = () => {
  if (currentStep.value > 1) currentStep.value--
}

const copyActivationData = async () => {
  await navigator.clipboard.writeText(activationData.value)
  copySuccess.value = true
  setTimeout(() => (copySuccess.value = false), 2000)
}

const resetStepper = () => {
  key.value = ''
  deviceId.value = ''
  activationData.value = ''
  cardHash.value = ''
  keyError.value = ''
  deviceIdError.value = ''
  currentStep.value = 1
}

onMounted(() => {
  const params = new URLSearchParams(location.search)
  const id = params.get('deviceId')
  if (id) deviceId.value = id
})
</script>

<style>
* {
  box-sizing: border-box;
}
</style>
