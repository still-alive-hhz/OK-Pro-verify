import { ref, computed, onMounted } from 'vue'

/* ================== 基础状态 ================== */
const key = ref('')
const deviceId = ref('')
const activationData = ref('')
const cardHash = ref('')

const keyError = ref('')
const deviceIdError = ref('')

const currentStep = ref(1)
const isProcessing = ref(false)
const copySuccess = ref(false)
const showCopyRequiredHint = ref(false)

/* ================== 工具函数 ================== */
const calculateCardHash = async (text: string): Promise<string> => {
  const buf = await crypto.subtle.digest(
    'SHA-256',
    new TextEncoder().encode(text)
  )
  return Array.from(new Uint8Array(buf))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('')
}

/* ================== 校验 ================== */
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
    deviceIdError.value = '设备ID格式不正确，应为32位十六进制'
    return false
  }
  deviceIdError.value = ''
  return true
}

/* ================== 核心：卡密真实性校验 ================== */
const verifyCard = async (
  cardText: string,
  hash: string
): Promise<boolean> => {
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

    // ✅ 正常卡密
    if (res.ok && data.success) return true

    // ⚠️ 已使用卡密：允许继续
    if (data.isCardUsed) {
      keyError.value = '该卡密已使用，可重新生成激活数据'
      return true
    }

    // ❌ 无效卡密
    if (data.isCardInvalid) {
      keyError.value = '卡密无效，请检查后重试'
      return false
    }

    keyError.value = data.error || '卡密验证失败'
    return false
  } catch (e) {
    keyError.value = '网络错误，请稍后再试'
    return false
  } finally {
    isProcessing.value = false
  }
}

/* ================== 请求激活数据 ================== */
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

    deviceIdError.value = data.error || '生成激活数据失败'
    return false
  } catch (e) {
    deviceIdError.value = '网络错误，请稍后再试'
    return false
  } finally {
    isProcessing.value = false
  }
}

/* ================== 步骤控制 ================== */
const isNextDisabled = computed(() => {
  if (isProcessing.value) return true
  if (currentStep.value === 1) return !validateKey()
  if (currentStep.value === 2) return !validateDeviceId()
  return false
})

const nextStep = async () => {
  if (isNextDisabled.value) return

  // STEP 1 → 校验卡密真实性
  if (currentStep.value === 1) {
    const hash = await calculateCardHash(key.value)
    cardHash.value = hash

    const ok = await verifyCard(key.value, hash)
    if (!ok) return

    currentStep.value++
    return
  }

  // STEP 2 → 请求激活数据
  if (currentStep.value === 2) {
    const ok = await requestSignature()
    if (!ok) return

    currentStep.value++
    return
  }

  // STEP 3 → 展示结果
  if (currentStep.value < 5) {
    currentStep.value++
  }
}

const prevStep = () => {
  if (currentStep.value > 1) currentStep.value--
}

/* ================== 复制 ================== */
const copyActivationData = async () => {
  if (!activationData.value) return
  await navigator.clipboard.writeText(activationData.value)
  copySuccess.value = true
  showCopyRequiredHint.value = false
  setTimeout(() => (copySuccess.value = false), 3000)
}

/* ================== 重置 ================== */
const resetStepper = () => {
  key.value = ''
  deviceId.value = ''
  activationData.value = ''
  cardHash.value = ''
  keyError.value = ''
  deviceIdError.value = ''
  currentStep.value = 1
}

/* ================== URL 自动填充设备ID ================== */
onMounted(() => {
  const params = new URLSearchParams(location.search)
  const id = params.get('deviceId')
  if (id) deviceId.value = id
})
