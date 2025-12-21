<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-950 text-slate-50 px-4">
    <!-- 标题 -->
    <header class="pt-10 pb-6 text-center">
      <h1 class="text-3xl sm:text-4xl font-bold text-cyan-400">
        Still Alive 软件激活工具
      </h1>
      <p class="mt-3 text-slate-400 text-sm sm:text-base">
        快速激活你的设备
      </p>
    </header>

    <!-- 卡片 -->
    <main class="max-w-md mx-auto">
      <div class="bg-slate-900/70 rounded-2xl border border-slate-800 shadow-xl">

        <!-- STEP 1 -->
        <div v-if="step === 1" class="p-6">
          <h2 class="text-xl font-bold mb-4 text-center">输入卡密</h2>

          <input
            v-model="card"
            @input="validateCard"
            class="w-full px-4 py-4 rounded-xl bg-slate-800 border border-slate-700 text-lg"
            placeholder="12 位卡密"
          />

          <p v-if="cardError" class="mt-2 text-red-400 text-sm">
            {{ cardError }}
          </p>
        </div>

        <!-- STEP 2 -->
        <div v-if="step === 2" class="p-6">
          <h2 class="text-xl font-bold mb-4 text-center">输入设备 ID</h2>

          <input
            v-model="deviceId"
            @input="validateDeviceId"
            class="w-full px-4 py-4 rounded-xl bg-slate-800 border border-slate-700 text-sm break-all"
            placeholder="32 位十六进制设备 ID"
          />

          <p v-if="deviceError" class="mt-2 text-red-400 text-sm">
            {{ deviceError }}
          </p>
        </div>

        <!-- STEP 3 -->
        <div v-if="step === 3" class="p-6">
          <h2 class="text-xl font-bold mb-4 text-center">激活数据</h2>

          <textarea
            v-model="result"
            readonly
            rows="4"
            class="w-full px-4 py-3 rounded-xl bg-slate-800 border border-slate-700 text-sm"
          ></textarea>

          <button
            @click="copyResult"
            class="mt-4 w-full py-4 rounded-xl bg-gradient-to-r from-blue-600 to-emerald-500 text-white font-medium active:scale-95 transition"
          >
            复制激活数据
          </button>

          <p v-if="copied" class="mt-2 text-center text-emerald-400 text-sm">
            已复制到剪贴板
          </p>

          <!-- 再次查询 -->
          <button
            @click="step = 1"
            class="mt-4 w-full py-3 rounded-xl bg-slate-800 text-slate-300"
          >
            重新查询
          </button>
        </div>

        <!-- 底部按钮 -->
        <div
          v-if="step < 3"
          class="flex justify-between gap-4 px-6 py-4 border-t border-slate-800"
        >
          <button
            @click="prev"
            :disabled="step === 1"
            class="flex-1 py-3 rounded-xl bg-slate-800 disabled:opacity-40"
          >
            上一步
          </button>

          <button
            @click="next"
            :disabled="nextDisabled || loading"
            class="flex-1 py-3 rounded-xl bg-gradient-to-r from-blue-600 to-emerald-500 text-white disabled:opacity-40"
          >
            {{ loading ? '处理中…' : '下一步' }}
          </button>
        </div>

      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const step = ref(1)
const card = ref('')
const deviceId = ref('')
const result = ref('')
const loading = ref(false)
const copied = ref(false)

const cardError = ref('')
const deviceError = ref('')

// 校验
const validateCard = () => {
  if (!/^[A-Z0-9]{12}$/.test(card.value)) {
    cardError.value = '卡密格式错误（12 位大写字母或数字）'
    return false
  }
  cardError.value = ''
  return true
}

const validateDeviceId = () => {
  if (!/^[A-Fa-f0-9]{32}$/.test(deviceId.value)) {
    deviceError.value = '设备 ID 格式错误'
    return false
  }
  deviceError.value = ''
  return true
}

const nextDisabled = computed(() => {
  if (step.value === 1) return !validateCard()
  if (step.value === 2) return !validateDeviceId()
  return false
})

// SHA256
const sha256 = async (text: string) => {
  const buf = await crypto.subtle.digest(
    'SHA-256',
    new TextEncoder().encode(text)
  )
  return Array.from(new Uint8Array(buf))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('')
}

// API
const next = async () => {
  if (step.value === 1) {
    step.value++
    return
  }

  if (step.value === 2) {
    loading.value = true

    const hash = await sha256(card.value)

    const res = await fetch('https://api.verify.stillalive.asia', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        mode: 'generate-signature',
        cardText: card.value,
        deviceId: deviceId.value,
        cardHash: hash,
      }),
    })

    const data = await res.json()
    loading.value = false

    if (!data.success) {
      deviceError.value = data.error || '生成失败'
      return
    }

    result.value = data.resultPacket
    step.value = 3
  }
}

const prev = () => {
  if (step.value > 1) step.value--
}

const copyResult = async () => {
  await navigator.clipboard.writeText(result.value)
  copied.value = true
  setTimeout(() => (copied.value = false), 2000)
}
</script>

<style>
input,
textarea {
  outline: none;
}
</style>
