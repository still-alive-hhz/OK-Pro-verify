<template>
  <div class="min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-950 text-slate-50">
    <!-- 标题 -->
    <header class="py-6 text-center">
      <h1 class="text-3xl font-bold text-cyan-400">
        Still Alive 购买验证
      </h1>
      <p class="mt-2 text-slate-400">
        验证成功后请返回游戏继续
      </p>
    </header>

    <main class="max-w-md mx-auto px-4 pb-16">
      <div class="bg-slate-900/70 rounded-2xl border border-slate-800 shadow-xl overflow-hidden">

        <!-- STEP 1：购买码 -->
        <div v-if="step === 1" class="p-6">
          <h2 class="text-xl font-bold mb-4 text-center">
            输入 12 位购买码
          </h2>

          <input
            v-model="card"
            class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-700"
            placeholder="A1B2C3D4E5F6"
            @input="validateCard"
          />

          <p v-if="cardError" class="mt-2 text-red-400 text-sm">
            {{ cardError }}
          </p>

          <button
            class="mt-6 w-full py-3 rounded-lg bg-gradient-to-r from-blue-600 to-emerald-500"
            :disabled="loading"
            @click="submitCard"
          >
            {{ loading ? '验证中…' : '确认购买' }}
          </button>
        </div>

        <!-- STEP 2：手环数字 -->
        <div v-if="step === 2" class="p-6">
          <h2 class="text-xl font-bold mb-2 text-center">
            输入手环显示的数字
          </h2>

          <p class="text-slate-400 text-sm mb-4 text-center">
            范围：0 ～ 4000
          </p>

          <input
            v-model="bandNumber"
            class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-700"
            placeholder="例如：1234"
          />

          <p v-if="bandError" class="mt-2 text-red-400 text-sm">
            {{ bandError }}
          </p>

          <button
            class="mt-6 w-full py-3 rounded-lg bg-gradient-to-r from-blue-600 to-emerald-500"
            @click="generateCode"
          >
            生成挑战码
          </button>

          <div v-if="challengeCode" class="mt-6 text-center">
            <p class="text-slate-400 text-sm mb-1">
              6 位挑战码
            </p>
            <div class="text-3xl font-mono tracking-widest text-emerald-400">
              {{ challengeCode }}
            </div>
            <p class="mt-2 text-slate-400 text-sm">
              请返回游戏输入该数字
            </p>
          </div>
        </div>

      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

/* ================= 状态 ================= */

const step = ref(1)
const loading = ref(false)

const card = ref('')
const cardError = ref('')

const bandNumber = ref('')
const bandError = ref('')
const challengeCode = ref('')

/* ============== 常量（必须一致） ============== */

const API_URL = 'https://api.verify.stillalive.asia'
const CHALLENGE_SECRET = 'still-alive-2026'

/* ============== 工具函数 ============== */

const sha256Hex = async (text: string) => {
  const buf = await crypto.subtle.digest(
    'SHA-256',
    new TextEncoder().encode(text)
  )
  return Array.from(new Uint8Array(buf))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('')
}

/* ============== Step 1：购买码验证 ============== */

const validateCard = () => {
  if (!/^[A-Z0-9]{12}$/.test(card.value)) {
    cardError.value = '购买码格式不正确'
    return false
  }
  cardError.value = ''
  return true
}

const submitCard = async () => {
  if (!validateCard()) return

  loading.value = true
  try {
    const hash = await sha256Hex(card.value)

    const res = await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        mode: 'verify-card',
        cardHash: hash
      })
    })

    const data = await res.json()

    if (res.ok && data.success) {
      step.value = 2
      return
    }

    cardError.value = data.error || '购买码无效'
  } catch {
    cardError.value = '网络错误'
  } finally {
    loading.value = false
  }
}

/* ============== Step 2：生成挑战码 ============== */

const generateCode = async () => {
  const n = Number(bandNumber.value)

  if (!Number.isInteger(n) || n < 0 || n > 4000) {
    bandError.value = '请输入 0 到 4000 之间的整数'
    return
  }

  bandError.value = ''

  const raw = CHALLENGE_SECRET + n
  const hash = await sha256Hex(raw)
  const digits = hash.replace(/\D/g, '')
  challengeCode.value = digits.slice(0, 6)
}
</script>

<style>
* {
  box-sizing: border-box;
}
</style>
