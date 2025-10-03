<template>
  <div class="app-container w-full min-h-screen relative">
    <!-- 全屏背景 -->
    <div class="fixed inset-0 -z-10 overflow-hidden bg-black">
      <DotGrid
        :dot-size="4"
        :gap="25"
        base-color="#6C7C74"
        class-name="w-full h-full"
      />
    </div>
    
    <!-- 主内容区域 - 横向布局 -->
    <div class="relative z-10 w-full min-h-screen flex justify-center items-center p-4">
      <div class="w-[90%] max-w-[1000px] flex justify-center items-center">
        <Stepper
          :initial-step="1"
          :on-step-change="handleStepChange"
          :on-final-step-completed="handleFinalStepCompleted"
          :next-button-props="{ class: nextButtonClass }"
          :lock-on-complete="false"
          :on-next-step="handleNextClick"
          back-button-text="上一步"
          next-button-text="下一步"
          step-circle-container-class-name="
    w-full max-w-md 
    p-4 
    rounded-[2rem] 
    shadow-[0_20px_25px_-5px_rgba(0,0,0,0.1),0_10px_10px_-5px_rgba(0,0,0,0.04)] 
    border border-gray-800 
    bg-black 
    text-white
  "
          step-container-class-name="flex items-center justify-center w-full mb-8"
          content-class-name="w-full"
          footer-class-name="w-full"
        >
          <div class="py-4 px-6 text-center w-full">
    <h2 class="text-2xl font-bold mb-3 text-white">输入卡密</h2>
    <p class="text-gray-200 mb-3">购买后会自动返回一个12位的卡密</p>
    
    <input
      v-model="key"
      class="mt-3 px-4 py-3 border border-gray-300 rounded-lg w-full max-w-[300px] bg-black/10 text-white focus:outline-none focus:border-[#1246A4] focus:ring-2 focus:ring-[#1246A4]/30"
      placeholder="A1B2C3D4E5F6"
    />

    <div class="mt-1 flex justify-center">
      <a
        href="https://afdian.com/item/751c4cb48ffa11f08f9f5254001e7c00"
        target="_blank"
        rel="noopener noreferrer"
      >
        <GradientText
          text="还未购买？点我购买，限时特价"
          :colors="['#40ffaa', '#4079ff','#40ffaa', '#4079ff', '#40ffaa']"
          :animation-speed="5"
          :show-border="false"
          class-name="
            w-full max-w-[300px]
            px-4 py-3
            rounded-lg
            bg-black/5
            text-center
          "
        />
      </a>
    </div>

    <p v-if="keyError" class="text-red-500 text-sm mt-1 mb-2">{{ keyError }}</p>
  </div>

          <div class="p-6 text-center w-full">
            <h2 class="text-2xl font-bold mb-4 text-white">输入设备ID</h2>
            <p class="text-gray-200 mb-4">手环上的二维码扫描出来的数据</p>
            <input
              v-model="deviceId"
              class="mt-4 px-4 py-3 border border-gray-300 rounded-lg w-full max-w-[300px] bg-black/10 text-white focus:outline-none focus:border-[#1246A4] focus:ring-2 focus:ring-[#1246A4]/30"
              placeholder="32位的16进制数据"
            />
            <p v-if="deviceIdError" class="text-red-500 text-sm mt-2">{{ deviceIdError }}</p>
          </div>

          <div class="py-4 px-6 text-center w-full">
            <h2 class="text-2xl font-bold mb-4 text-white">确认你的数据无误</h2>
            <p class="text-gray-200 mb-4">请使用你要激活的设备的二维码</p>
            <p class="text-gray-200 mb-4">如果你因为填错数据导致激活失败</p>
            <p class="text-gray-200 mb-4">本产品概不负责，你需要重新购买</p>
          </div>

          <div class="p-6 text-center w-full">
            <h2 class="text-2xl font-bold mb-4 text-white">你的激活数据</h2>
            <p class="text-gray-200 mb-4">复制数据后前往AstroBox使用插件激活</p>
            <p class="text-gray-200 mb-4">操作教程会在购买后在爱发电私信给你</p>
            <div class="mt-4 flex items-center justify-center gap-2 max-w-md mx-auto">
              <input
                v-model="activationData"
                class="flex-1 px-4 py-3 border border-gray-300 rounded-lg bg-black/30 text-white cursor-not-allowed text-center min-w-0"
                readonly
                style="height: 40px; box-sizing: border-box;"
              />

              <button
                @click="copyActivationData"
                class="w-10 h-10 border border-gray-300 rounded-lg bg-black/30 text-white hover:bg-black/40 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-[#1246A4]/50 flex items-center justify-center disabled:opacity-50 disabled:cursor-not-allowed"
                :disabled="!activationData"
                aria-label="复制激活数据"
                style="box-sizing: border-box;"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="18"
                  height="18"
                  viewBox="0 0 24 24"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <rect x="9" y="9" width="13" height="13" rx="2" ry="2"></rect>
                  <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"></path>
                </svg>
              </button>
            </div>

            <!-- 复制成功提示 -->
            <div 
              class="overflow-hidden transition-[max-height] duration-300 ease-out"
              :style="{
                maxHeight: copySuccess ? '1.5rem' : '0',
                marginTop: copySuccess ? '0.5rem' : '0'
              }"
            >
              <p class="text-green-400 text-sm leading-none">
                已复制！
              </p>
            </div>

            <!-- 仅在点击下一步且未复制时显示强制提示 -->
            <p 
              v-if="showCopyRequiredHint" 
              class="text-red-400 text-sm mt-2 flex items-center justify-center gap-1 animate-fade-in-out"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <circle cx="12" cy="12" r="10"></circle>
                <line x1="12" y1="8" x2="12" y2="12"></line>
                <line x1="12" y1="16" x2="12.01" y2="16"></line>
              </svg>
              请先点击“复制”按钮，再继续
            </p>
          </div>

          <div class="py-4 px-6 text-center w-full">
            <h2 class="text-2xl font-bold mb-4 text-white">老乡！来钓鱼</h2>
            <p class="text-gray-200 mb-4">你可以加Q群，群号在自动回复里</p>
          </div>
        </Stepper>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';
import DotGrid from './components/DotGrid.vue';
import Stepper from './components/Stepper.vue';
import GradientText from "./components/GradientText.vue";

const key = ref('');
const deviceId = ref('');
const activationData = ref('');
const keyError = ref('');
const deviceIdError = ref('');

// ✅ 新增：用于存储第一步计算出的卡密哈希值
const cardHash = ref<string>('');

const currentStep = ref(1);
const copySuccess = ref(false);
const showCopyRequiredHint = ref(false);
const isProcessing = ref(false); // 防止重复提交

// ✅ 新增：设备失败计数器和锁定状态（基于 deviceId）
interface DeviceLockState {
  failCount: number;       // 失败次数（仅对无效卡密累加）
  lockedUntil: number | null; // 锁定结束时间戳（毫秒）
  lastUsedCard: string | null; // 上次尝试的卡密（用于判断是否切换）
}

// 每个设备独立状态
const deviceLockMap = new Map<string, DeviceLockState>();

// 从本地存储加载锁定状态
const loadDeviceLockState = () => {
  try {
    const savedState = localStorage.getItem('deviceLockMap');
    if (savedState) {
      const parsedState = JSON.parse(savedState);
      // 恢复每个设备的锁定状态
      Object.keys(parsedState).forEach(deviceId => {
        deviceLockMap.set(deviceId, parsedState[deviceId]);
      });
    }
  } catch (e) {
    console.error('Failed to load device lock state from localStorage:', e);
  }
};

// 保存锁定状态到本地存储
const saveDeviceLockState = () => {
  try {
    const stateToSave: Record<string, DeviceLockState> = {};
    deviceLockMap.forEach((value, key) => {
      stateToSave[key] = value;
    });
    localStorage.setItem('deviceLockMap', JSON.stringify(stateToSave));
  } catch (e) {
    console.error('Failed to save device lock state to localStorage:', e);
  }
};

const getDeviceLockState = (): DeviceLockState => {
  const id = deviceId.value;
  if (!deviceLockMap.has(id)) {
    deviceLockMap.set(id, {
      failCount: 0,
      lockedUntil: null,
      lastUsedCard: null
    });
  }
  return deviceLockMap.get(id)!;
};

// 计算卡密的 SHA-256 哈希值
const calculateCardHash = async (text: string): Promise<string> => {
  const encoder = new TextEncoder();
  const data = encoder.encode(text);
  const hashBuffer = await crypto.subtle.digest('SHA-256', data);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
};

// 格式化剩余时间：超过60秒显示为 "X分Y秒"，否则显示 "Z秒"
const formatTimeRemaining = (remainingMs: number): string => {
  if (remainingMs <= 0) return '0秒';

  const totalSeconds = Math.ceil(remainingMs / 1000);
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;

  if (minutes > 0) {
    return `${minutes}分${seconds > 0 ? seconds + '秒' : ''}`;
  } else {
    return `${seconds}秒`;
  }
};

// ✅ 倒计时状态管理
const countdownTimer = ref<number | null>(null); // 定时器句柄
const remainingTime = ref<number>(0); // 当前剩余时间（毫秒），用于动态更新

// 启动倒计时：监听当前是否处于锁定状态
const updateCountdown = () => {
  const state = getDeviceLockState();

  if (state.lockedUntil !== null) {
    const now = Date.now();
    const remaining = state.lockedUntil - now;

    if (remaining > 0) {
      remainingTime.value = remaining;
      if (!countdownTimer.value) {
        countdownTimer.value = window.setInterval(() => {
          const now = Date.now();
          // 修复：添加对 state.lockedUntil 可能为 null 的检查
          if (state.lockedUntil !== null) {
            const remaining = state.lockedUntil - now;
            if (remaining <= 0) {
              clearInterval(countdownTimer.value!);
              countdownTimer.value = null;
              remainingTime.value = 0;
              // 保存更新后的状态
              saveDeviceLockState();
            } else {
              remainingTime.value = remaining;
            }
          } else {
            // 如果 lockedUntil 变为 null，则清除定时器
            clearInterval(countdownTimer.value!);
            countdownTimer.value = null;
            remainingTime.value = 0;
          }
        }, 100); // 每100ms刷新一次，保证流畅
      }
    } else {
      // 已过期，停止定时器
      if (countdownTimer.value) {
        clearInterval(countdownTimer.value);
        countdownTimer.value = null;
      }
      remainingTime.value = 0;
      // 清除过期的锁定状态
      state.lockedUntil = null;
      saveDeviceLockState();
    }
  } else {
    // 未锁定，停止定时器
    if (countdownTimer.value) {
      clearInterval(countdownTimer.value);
      countdownTimer.value = null;
    }
    remainingTime.value = 0;
  }
};

// ✅ 响应式倒计时文本（根据 remainingTime 动态生成）
const dynamicCountdownText = computed(() => {
  if (remainingTime.value <= 0) return '';
  return formatTimeRemaining(remainingTime.value);
});

// ✅ 计算当前步骤的锁定提示（动态倒计时 + 静态提示）
const getLockErrorMessage = (step: 1 | 2): string => {
  const state = getDeviceLockState();
  if (state.lockedUntil === null) return '';

  const timeStr = dynamicCountdownText.value || formatTimeRemaining(state.lockedUntil - Date.now());
  if (step === 1) {
    return `操作被锁定，请等待 ${timeStr}后`;
  } else {
    return `操作被锁定，请等待 ${timeStr}后`;
  }
};

// 向 Worker 发送请求（卡密验证阶段）→ mode: verify-card
const verifyCard = async (cardText: string, cardHashValue: string): Promise<boolean> => {
  try {
    isProcessing.value = true;
    const res = await fetch('https://verify.cheongszesuen.workers.dev/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        mode: 'verify-card',
        cardText,
        cardHash: cardHashValue,
      }),
    });

    const data = await res.json();

    if (res.ok && data.success) {
      // ✅ 成功：重置设备状态
      const state = getDeviceLockState();
      state.failCount = 0;
      state.lockedUntil = null;
      state.lastUsedCard = cardText;
      saveDeviceLockState(); // 保存状态
      return true;
    }

    // ❌ 失败：处理不同错误类型
    if (data.code === 403 && data.isCardInvalid) {
      // 卡密无效：指数退避锁定
      const state = getDeviceLockState();
      state.failCount++;
      const lockTime = Math.min(300, 60 * Math.pow(2, state.failCount - 1)); // 最大5分钟
      state.lockedUntil = Date.now() + lockTime * 1000;

      keyError.value = ''; // 清空静态错误
      updateCountdown();   // 启动倒计时
      saveDeviceLockState(); // 保存状态
      return false;
    }

    if (data.code === 409 && data.isCardUsed) {
      // 卡密已用：固定30秒锁定
      const state = getDeviceLockState();
      state.lockedUntil = Date.now() + 30 * 1000;

      keyError.value = ''; // 清空静态错误
      updateCountdown();   // 启动倒计时
      saveDeviceLockState(); // 保存状态
      return false;
    }

    // 其他错误
    keyError.value = data.error || '卡密验证失败，请重试';
    return false;

  } catch (err) {
    keyError.value = '网络错误，请检查连接';
    console.error('Verify card error:', err);
    return false;
  } finally {
    isProcessing.value = false;
  }
};

// 向 Worker 发送请求（获取签名阶段）→ mode: generate-signature
const requestSignature = async (cardText: string, deviceIdValue: string): Promise<string | null> => {
  try {
    isProcessing.value = true;
    const res = await fetch('https://verify.cheongszesuen.workers.dev/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        mode: 'generate-signature',
        cardText,
        deviceId: deviceIdValue,
        cardHash: cardHash.value,
      }),
    });

    const data = await res.json();

    if (res.ok && data.success) {
      activationData.value = data.resultPacket;
      return data.resultPacket;
    } else {
      deviceIdError.value = data.error || '签名生成失败';
      console.error('Signature generation failed:', data);
      return null;
    }
  } catch (err) {
    deviceIdError.value = '网络错误，请检查连接';
    console.error('Request signature error:', err);
    return null;
  } finally {
    isProcessing.value = false;
  }
};

// 下一步按钮样式类（增强第4步逻辑）
const nextButtonClass = computed(() => {
  let baseClass = 'border-none transition-all duration-[350ms] flex items-center justify-center rounded-full font-medium tracking-tight px-3.5 py-1.5 ';

  if (currentStep.value === 1) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;

    if (!key.value || isProcessing.value || isLocked) {
      baseClass += 'bg-gray-400 text-gray-200 cursor-not-allowed';
    } else {
      baseClass += validateKey()
        ? 'bg-[#1246A4] text-white hover:bg-[#113671] cursor-pointer'
        : 'bg-gray-400 text-gray-200 cursor-not-allowed';
    }
  } else if (currentStep.value === 2) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;

    if (!deviceId.value || isProcessing.value || isLocked) {
      baseClass += 'bg-gray-400 text-gray-200 cursor-not-allowed';
    } else {
      baseClass += validateDeviceId()
        ? 'bg-[#1246A4] text-white hover:bg-[#113671] cursor-pointer'
        : 'bg-gray-400 text-gray-200 cursor-not-allowed';
    }
  } else if (currentStep.value === 4) {
    if (!copySuccess.value) {
      baseClass += 'bg-gray-400 text-gray-200 cursor-not-allowed';
    } else {
      baseClass += 'bg-[#1246A4] text-white hover:bg-[#113671] cursor-pointer';
    }
  } else {
    baseClass += 'bg-[#1246A4] text-white hover:bg-[#113671] cursor-pointer';
  }

  return baseClass;
});

// 步骤切换时清空错误 & 停止倒计时
const handleStepChange = (step: number) => {
  currentStep.value = step;
  console.log('Step changed to:', step);

  if (step !== 1) keyError.value = '';
  if (step !== 2) deviceIdError.value = '';

  if (step === 4) {
    showCopyRequiredHint.value = false;
  }

  // ✅ 切换步骤时强制停止倒计时
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value);
    countdownTimer.value = null;
  }
  remainingTime.value = 0;
};

const handleNextClick = async (step: number): Promise<boolean> => {
  if (step === 1) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;

    if (isLocked) {
      // ✅ 显示动态倒计时
      keyError.value = getLockErrorMessage(1);
      return false;
    }

    if (!key.value) {
      keyError.value = '请输入卡密';
      return false;
    }
    if (!validateKey()) return false;

    const calculatedHash = await calculateCardHash(key.value); // ✅ 现在函数存在，不会报错！
    cardHash.value = calculatedHash;
    const success = await verifyCard(key.value, calculatedHash);

    if (!success) {
      // 确保在验证失败时也显示锁定信息
      const state = getDeviceLockState();
      if (state.lockedUntil !== null && Date.now() < state.lockedUntil) {
        keyError.value = getLockErrorMessage(1);
      }
      return false;
    }
    return true;
  } else if (step === 2) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;

    if (isLocked) {
      deviceIdError.value = getLockErrorMessage(2);
      return false;
    }

    if (!deviceId.value) {
      deviceIdError.value = '请输入设备ID';
      return false;
    }
    if (!validateDeviceId()) return false;

    const resultPacket = await requestSignature(key.value, deviceId.value);
    if (!resultPacket) {
      // 确保在验证失败时也显示锁定信息
      const state = getDeviceLockState();
      if (state.lockedUntil !== null && Date.now() < state.lockedUntil) {
        deviceIdError.value = getLockErrorMessage(2);
      }
      return false;
    }

    activationData.value = resultPacket;
    return true;
  } else if (step === 4) {
    if (!copySuccess.value) {
      showCopyRequiredHint.value = true;
      return false;
    }
    return true;
  }

  return true;
};

const handleFinalStepCompleted = () => {
  console.log('Stepper completed!');
};

// // 开发环境自动填充测试数据
// if (import.meta.env.DEV) {
//   key.value = 'A1B2C3D4E5F6';
//   deviceId.value = 'd4cd0dabcf4caa22ad92fab40844c786';
//   activationData.value = `${deviceId.value}:abc123...`; // 模拟签名
//   currentStep.value = 3;
//   console.log('✅ [DEV] 测试数据已自动填充：', activationData.value);
// }

// 在组件挂载时检查URL参数
onMounted(() => {
  const deviceIdFromUrl = getDeviceIdFromUrl();
  if (deviceIdFromUrl) {
    deviceId.value = deviceIdFromUrl;
  }

  // 加载本地存储的锁定状态
  loadDeviceLockState();
  
  // 初始化倒计时监听
  updateCountdown();
});

// 组件销毁时清理定时器
onUnmounted(() => {
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value);
  }
});

const copyActivationData = async () => {
  if (!activationData.value) return;

  try {
    await navigator.clipboard.writeText(activationData.value);
    copySuccess.value = true;
    showCopyRequiredHint.value = false;
    setTimeout(() => {
      copySuccess.value = false;
    }, 3000);
  } catch (err) {
    console.error('Failed to copy text: ', err);
    alert('复制失败，请手动选择文本并复制');
  }
};

// 验证卡密格式 (12位大写字母和数字组合)
const validateKey = () => {
  const keyRegex = /^[A-Z0-9]{12}$/;
  if (!key.value) {
    keyError.value = '请输入卡密';
    return false;
  } else if (!keyRegex.test(key.value)) {
    keyError.value = '卡密格式不正确，应为12位大写字母或数字';
    return false;
  } else {
    keyError.value = '';
    return true;
  }
};

// 验证设备ID格式 (32位十六进制)
const validateDeviceId = () => {
  const deviceIdRegex = /^[A-Fa-f0-9]{32}$/;
  if (!deviceId.value) {
    deviceIdError.value = '请输入设备ID';
    return false;
  } else if (!deviceIdRegex.test(deviceId.value)) {
    deviceIdError.value = '设备ID格式不正确，应为32位十六进制数据';
    return false;
  } else {
    deviceIdError.value = '';
    return true;
  }
};

// 从URL中提取设备ID参数
const getDeviceIdFromUrl = () => {
  const urlParams = new URLSearchParams(window.location.search);
  return urlParams.get('deviceId');
};
</script>

<style>
html, body, #app {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

.app-container {
  width: 100%;
  min-height: 100vh;
  position: relative;
}

div, span, button {
  color: blue;
}

@media (max-width: 768px) {
  .stepper-wrapper {
    width: 95%;
  }
  
  .stepper-panel {
    min-width: unset;
    max-width: 100%;
    padding: 1.5rem;
  }
  
  .stepper-indicator {
    gap: 0.5rem;
  }
  
  .step-content {
    padding: 1rem;
  }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-5px); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-fade-in {
  animation: fadeIn 0.6s ease-out;
}

@keyframes fadeInOut {
  0% { opacity: 0; transform: translateY(-5px); }
  20% { opacity: 1; transform: translateY(0); }
  80% { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-5px); }
}

.animate-fade-in-out {
  animation: fadeInOut 1.2s ease-out;
}
</style>
