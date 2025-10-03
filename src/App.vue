<template>
  <div class="app-container w-full min-h-screen relative">
    <!-- 全屏背景 -->
    <div class="fixed inset-0 -z-10 overflow-hidden bg-gradient-to-br from-black to-gray-900">
      <DotGrid
        :dot-size="4"
        :gap="25"
        base-color="#6C7C74"
        class-name="w-full h-full opacity-60"
        :animate-dots="true"
      />
    </div>
    
    <!-- 主内容区域 - 横向布局 -->
    <div class="relative z-10 w-full min-h-screen flex justify-center items-center p-4 py-12">
      <div class="w-[90%] max-w-[1000px] flex justify-center items-center">
        <Stepper
          :initial-step="1"
          :on-step-change="handleStepChange"
          :on-final-step-completed="handleFinalStepCompleted"
          :next-button-props="{ class: nextButtonClass, onClick: handleNextButtonClick }"
          :lock-on-complete="false"
          :on-next-step="handleNextClick"
          back-button-text="上一步"
          next-button-text="下一步"
          step-circle-container-class-name="
            w-full max-w-md 
            p-4 
            rounded-[2rem] 
            shadow-[0_20px_25px_-5px_rgba(0,0,0,0.3),0_10px_10px_-5px_rgba(0,0,0,0.2)] 
            border border-gray-800 
            bg-gradient-to-br from-gray-900 to-black 
            text-white
            backdrop-blur-sm
          "
          step-container-class-name="flex items-center justify-center w-full mb-8"
          content-class-name="w-full"
          footer-class-name="w-full"
        >
          <!-- 步骤1：输入卡密 -->
          <div class="py-4 px-6 text-center w-full step-content-transition">
            <div class="flex items-center justify-center gap-2 mb-3">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-[#4079ff]">
                <rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
                <path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
              </svg>
              <h2 class="text-2xl font-bold mb-3 text-white">输入卡密</h2>
            </div>
            <p class="text-gray-300 mb-6">购买后会自动返回一个12位的卡密</p>
            
            <input
              v-model="key"
              @input="validateKey"
              @blur="validateKey"
              class="mt-3 px-4 py-3 border border-gray-700 rounded-lg w-full max-w-[300px] bg-gray-900/50 text-white focus:outline-none focus:border-[#1246A4] focus:ring-2 focus:ring-[#1246A4]/30 transition-all duration-300 hover:border-gray-500"
              placeholder="A1B2C3D4E5F6"
            />

            <div class="mt-4 flex justify-center">
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
                    hover:bg-black/20
                    transition-all duration-300
                    cursor-pointer
                  "
                />
              </a>
            </div>

            <p v-if="keyError" class="text-red-400 text-sm mt-2 mb-2 animate-fade-in">{{ keyError }}</p>
          </div>

          <!-- 步骤2：输入设备ID -->
          <div class="p-6 text-center w-full step-content-transition">
            <div class="flex items-center justify-center gap-2 mb-3">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-[#4079ff]">
                <rect x="4" y="4" width="16" height="16" rx="2" ry="2"></rect>
                <rect x="9" y="9" width="6" height="6"></rect>
                <line x1="9" y1="2" x2="9" y2="4"></line>
                <line x1="15" y1="2" x2="15" y2="4"></line>
                <line x1="9" y1="20" x2="9" y2="22"></line>
                <line x1="15" y1="20" x2="15" y2="22"></line>
                <line x1="20" y1="9" x2="22" y2="9"></line>
                <line x1="20" y1="14" x2="22" y2="14"></line>
                <line x1="2" y1="9" x2="4" y2="9"></line>
                <line x1="2" y1="14" x2="4" y2="14"></line>
              </svg>
              <h2 class="text-2xl font-bold mb-4 text-white">输入设备ID</h2>
            </div>
            <p class="text-gray-300 mb-6">手环上的二维码扫描出来的数据</p>
            <input
              v-model="deviceId"
              @input="validateDeviceId"
              @blur="validateDeviceId"
              class="mt-4 px-4 py-3 border border-gray-700 rounded-lg w-full max-w-[300px] bg-gray-900/50 text-white focus:outline-none focus:border-[#1246A4] focus:ring-2 focus:ring-[#1246A4]/30 transition-all duration-300 hover:border-gray-500"
              placeholder="32位的16进制数据"
            />
            <p v-if="deviceIdError" class="text-red-400 text-sm mt-2 animate-fade-in">{{ deviceIdError }}</p>
          </div>

          <!-- 步骤3：确认数据 -->
          <div class="py-4 px-6 text-center w-full step-content-transition">
            <div class="flex items-center justify-center gap-2 mb-3">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-[#40ffaa]">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
              <h2 class="text-2xl font-bold mb-4 text-white">确认你的数据无误</h2>
            </div>
            
            <div class="bg-gray-900/30 rounded-lg p-4 max-w-md mx-auto mb-6 border border-gray-800">
              <p class="text-gray-300 mb-3"><span class="text-gray-400">卡密：</span>{{ key || '未输入' }}</p>
              <p class="text-gray-300"><span class="text-gray-400">设备ID：</span>{{ deviceId || '未输入' }}</p>
            </div>
            
            <p class="text-gray-300 mb-2">请使用你要激活的设备的二维码</p>
            <p class="text-gray-300 mb-2">如果你因为填错数据导致激活失败</p>
            <p class="text-gray-300 mb-2">本产品概不负责，你需要重新购买</p>
          </div>

          <!-- 步骤4：激活数据 -->
          <div class="p-6 text-center w-full step-content-transition">
            <div class="flex items-center justify-center gap-2 mb-3">
              <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="text-[#40ffaa]">
                <path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"></path>
                <rect x="8" y="2" width="8" height="4" rx="1" ry="1"></rect>
                <path d="M9 14l2 2 4-4"></path>
              </svg>
              <h2 class="text-2xl font-bold mb-4 text-white">你的激活数据</h2>
            </div>
            <p class="text-gray-300 mb-2">复制数据后前往AstroBox使用插件激活</p>
            <p class="text-gray-300 mb-6">操作教程会在购买后在爱发电私信给你</p>
            
            <div class="mt-4 flex items-center justify-center gap-2 max-w-md mx-auto bg-gray-900/30 rounded-lg p-2 border border-gray-800">
              <input
                v-model="activationData"
                class="flex-1 px-4 py-3 border border-gray-700 rounded-lg bg-gray-900/50 text-white cursor-not-allowed text-center min-w-0"
                readonly
                style="height: 40px; box-sizing: border-box;"
              />

              <button
                @click="copyActivationData"
                class="w-10 h-10 border border-gray-700 rounded-lg bg-gray-900/50 text-white hover:bg-[#1246A4] hover:border-[#1246A4] transition-all duration-300 focus:outline-none focus:ring-2 focus:ring-[#1246A4]/50 flex items-center justify-center disabled:opacity-50 disabled:cursor-not-allowed"
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
              class="overflow-hidden transition-all duration-300 ease-out"
              :style="{
                maxHeight: copySuccess ? '1.5rem' : '0',
                marginTop: copySuccess ? '0.5rem' : '0',
                opacity: copySuccess ? 1 : 0
              }"
            >
              <p class="text-green-400 text-sm leading-none flex items-center justify-center gap-1">
                <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="20 6 9 17 4 12"></polyline>
                </svg>
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

          <!-- 步骤5：完成页面 -->
          <div class="py-4 px-6 text-center w-full step-content-transition">
            <div class="mb-6 inline-flex items-center justify-center w-16 h-16 rounded-full bg-gradient-to-br from-[#40ffaa] to-[#4079ff] text-black">
              <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            
            <h2 class="text-2xl font-bold mb-4 text-white">老乡！来钓鱼</h2>
            <p class="text-gray-300 mb-6">你可以加Q群，群号在自动回复里</p>
            
            <button 
              @click="resetStepper"
              class="px-6 py-2.5 bg-gradient-to-r from-[#1246A4] to-[#1E5BC8] text-white rounded-lg hover:opacity-90 transition-all duration-300 transform hover:scale-105 active:scale-95 focus:outline-none focus:ring-2 focus:ring-[#1246A4]/50"
            >
              重新激活
            </button>
          </div>
        </Stepper>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, inject } from 'vue';
import DotGrid from './components/DotGrid.vue';
import Stepper from './components/Stepper.vue';
import GradientText from "./components/GradientText.vue";

const key = ref('');
const deviceId = ref('');
const activationData = ref('');
const keyError = ref('');
const deviceIdError = ref('');

// 存储第一步计算出的卡密哈希值
const cardHash = ref<string>('');

const currentStep = ref(1);
const copySuccess = ref(false);
const showCopyRequiredHint = ref(false);
const isProcessing = ref(false); // 防止重复提交

// 设备失败计数器和锁定状态（基于 deviceId）
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

// 倒计时状态管理
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

// 响应式倒计时文本（根据 remainingTime 动态生成）
const dynamicCountdownText = computed(() => {
  if (remainingTime.value <= 0) return '';
  return formatTimeRemaining(remainingTime.value);
});

// 计算当前步骤的锁定提示（动态倒计时 + 静态提示）
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
    // 显示加载动画
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
      // 成功：重置设备状态
      const state = getDeviceLockState();
      state.failCount = 0;
      state.lockedUntil = null;
      state.lastUsedCard = cardText;
      saveDeviceLockState(); // 保存状态
      return true;
    }

    // 失败：处理不同错误类型
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
  let baseClass = 'border-none transition-all duration-350 flex items-center justify-center rounded-full font-medium tracking-tight px-6 py-2.5 transform hover:scale-105 active:scale-95 ';

  if (currentStep.value === 1) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;

    if (!key.value || isProcessing.value || isLocked) {
      baseClass += 'bg-gray-700 text-gray-300 cursor-not-allowed';
    } else {
      baseClass += validateKey()
        ? 'bg-gradient-to-r from-[#1246A4] to-[#1E5BC8] text-white hover:opacity-90 focus:outline-none focus:ring-2 focus:ring-[#1246A4]/50 cursor-pointer'
        : 'bg-gray-700 text-gray-300 cursor-not-allowed';
    }
  } else if (currentStep.value === 2) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;

    if (!deviceId.value || isProcessing.value || isLocked) {
      baseClass += 'bg-gray-700 text-gray-300 cursor-not-allowed';
    } else {
      baseClass += validateDeviceId()
        ? 'bg-gradient-to-r from-[#1246A4] to-[#1E5BC8] text-white hover:opacity-90 focus:outline-none focus:ring-2 focus:ring-[#1246A4]/50 cursor-pointer'
        : 'bg-gray-700 text-gray-300 cursor-not-allowed';
    }
  } else if (currentStep.value === 4) {
    if (!copySuccess.value) {
      baseClass += 'bg-gray-700 text-gray-300 cursor-not-allowed';
    } else {
      baseClass += 'bg-gradient-to-r from-[#1246A4] to-[#1E5BC8] text-white hover:opacity-90 focus:outline-none focus:ring-2 focus:ring-[#1246A4]/50 cursor-pointer';
    }
  } else {
    baseClass += 'bg-gradient-to-r from-[#1246A4] to-[#1E5BC8] text-white hover:opacity-90 focus:outline-none focus:ring-2 focus:ring-[#1246A4]/50 cursor-pointer';
  }

  return baseClass;
});

// 步骤切换时清空错误 & 停止倒计时
const handleStepChange = (step: number) => {
  // 添加步骤切换动画类
  const stepContents = document.querySelectorAll('.step-content-transition');
  stepContents.forEach(el => {
    el.classList.add('step-transition-out');
    setTimeout(() => {
      el.classList.remove('step-transition-out');
    }, 300);
  });
  
  currentStep.value = step;
  console.log('Step changed to:', step);

  if (step !== 1) keyError.value = '';
  if (step !== 2) deviceIdError.value = '';

  if (step === 4) {
    showCopyRequiredHint.value = false;
  }

  // 切换步骤时强制停止倒计时
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value);
    countdownTimer.value = null;
  }
  remainingTime.value = 0;
};

// 处理下一步按钮点击（仅用于动画反馈）
const handleNextButtonClick = () => {
  if (isProcessing.value) return;
  
  const nextButton = document.querySelector('.next-button');
  if (nextButton) {
    nextButton.classList.add('button-press');
    setTimeout(() => {
      nextButton.classList.remove('button-press');
    }, 200);
  }
};

const handleNextClick = async (step: number): Promise<boolean> => {
  if (step === 1) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;

    if (isLocked) {
      // 显示动态倒计时
      keyError.value = getLockErrorMessage(1);
      return false;
    }

    if (!key.value) {
      keyError.value = '请输入卡密';
      return false;
    }
    if (!validateKey()) return false;

    const calculatedHash = await calculateCardHash(key.value);
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

// 重置步骤器
const resetStepper = () => {
  // 假设Stepper组件有一个重置方法，通过inject获取
  const stepperReset = inject<() => void>('stepperReset');
  if (stepperReset) {
    stepperReset();
  }
  
  // 重置表单数据
  key.value = '';
  deviceId.value = '';
  activationData.value = '';
  currentStep.value = 1;
  copySuccess.value = false;
};

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
    
    // 复制成功动画反馈
    const copyButton = document.querySelector('[aria-label="复制激活数据"]');
    if (copyButton) {
      copyButton.classList.add('copy-success');
      setTimeout(() => {
        copyButton.classList.remove('copy-success');
      }, 1000);
    }
    
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
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

.app-container {
  width: 100%;
  min-height: 100vh;
  position: relative;
}

/* 步骤内容过渡动画 */
.step-content-transition {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  opacity: 1;
  transform: translateY(0);
}

.step-transition-out {
  opacity: 0;
  transform: translateY(10px);
}

/* 按钮动画效果 */
.button-press {
  transform: scale(0.95) !important;
  box-shadow: 0 0 0 3px rgba(18, 70, 164, 0.2) !important;
}

/* 复制按钮成功效果 */
.copy-success {
  background-color: #10b981 !important;
  border-color: #10b981 !important;
}

/* 动画效果 */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-5px); }
  to { opacity: 1; transform: translateY(0); }
}

.animate-fade-in {
  animation: fadeIn 0.4s ease-out forwards;
}

@keyframes fadeInOut {
  0% { opacity: 0; transform: translateY(-5px); }
  20% { opacity: 1; transform: translateY(0); }
  80% { opacity: 1; transform: translateY(0); }
  100% { opacity: 0; transform: translateY(-5px); }
}

.animate-fade-in-out {
  animation: fadeInOut 1.2s ease-out infinite;
}

/* 响应式调整 */
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
  
  h2.text-2xl {
    font-size: 1.5rem;
  }
}
</style>
