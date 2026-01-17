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
      <div class="bg-slate-900/70 rounded-2xl border border-slate-800 shadow-xl p-6">

        <h2 class="text-xl font-bold mb-2 text-center">
          输入 12 位购买码
        </h2>

        <input
          v-model="key"
          @input="validateKey"
          class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-700 mt-4"
          placeholder="A1B2C3D4E5F6"
        />

        <p v-if="keyError" class="mt-2 text-red-400 text-sm text-center">
          {{ keyError }}
        </p>

        <p v-if="success" class="mt-4 text-emerald-400 text-center">
          ✅ 购买验证通过  
          <br />
          请返回游戏输入 6 位挑战码
        </p>

        <button
          @click="submit"
          :disabled="isProcessing || success || !isValid"
          class="mt-6 w-full py-3 rounded-lg
                 bg-gradient-to-r from-blue-600 to-emerald-500
                 text-white disabled:opacity-40"
        >
          {{ isProcessing ? '验证中…' : '确认购买' }}
        </button>

      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

const key = ref('')
const keyError = ref('')
const isProcessing = ref(false)
const success = ref(false)

/* ===== 工具 ===== */
const sha256 = async (text: string): Promise<string> => {
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
    keyError.value = '格式错误，应为 12 位大写字母或数字'
    return false
  }
  keyError.value = ''
  return true
}

const isValid = computed(() => validateKey())

/* ===== 提交 ===== */
const submit = async () => {
  if (!validateKey()) return

  try {
    isProcessing.value = true
    const hash = await sha256(key.value)

    const res = await fetch('https://api.verify.stillalive.asia', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        mode: 'verify-card',
        cardHash: hash,
      }),
    })

    const data = await res.json()

    if (res.ok && data.success) {
      success.value = true
      return
    }

    if (data.isCardInvalid) {
      keyError.value = '购买码无效，请检查输入'
      return
    }

    keyError.value = data.error || '验证失败'
  } catch {
    keyError.value = '网络错误，请稍后再试'
  } finally {
    isProcessing.value = false
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}
</style>
