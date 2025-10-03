<template>
  <div class="app-container min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-950 text-slate-50">
    <!-- 装饰性背景元素 -->
    <div class="fixed inset-0 -z-10 overflow-hidden">
      <!-- 网格背景 -->
      <div class="absolute inset-0 bg-[radial-gradient(rgba(120,120,120,0.1)_1px,transparent_1px)] bg-[size:30px_30px]"></div>
      <!-- 光晕效果 -->
      <div class="absolute top-1/4 -left-20 w-96 h-96 bg-blue-600 rounded-full filter blur-[120px] opacity-20"></div>
      <div class="absolute bottom-1/3 -right-20 w-96 h-96 bg-emerald-500 rounded-full filter blur-[120px] opacity-20"></div>
    </div>
    
    <!-- 页面标题 -->
    <header class="py-6 px-4 sm:px-6 lg:px-8 text-center">
      <div class="inline-flex items-center justify-center w-12 h-12 rounded-full bg-gradient-to-br from-blue-600 to-emerald-500 mb-4">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="10"></circle>
          <polyline points="12 6 12 12 16 14"></polyline>
        </svg>
      </div>
      <h1 class="text-3xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-blue-400 to-emerald-400">小米手环激活工具</h1>
      <p class="mt-2 text-slate-400 max-w-md mx-auto">快速激活你的设备，享受完整功能体验</p>
    </header>

    <!-- 主内容区 -->
    <main class="max-w-md mx-auto px-4 sm:px-6 lg:px-8 pb-16">
      <!-- 步骤指示器 -->
      <div class="mb-8">
        <div class="flex items-center justify-between">
          <div v-for="(step, index) in 5" :key="index" class="flex flex-col items-center">
            <div 
              class="w-10 h-10 rounded-full flex items-center justify-center mb-2 transition-all duration-500"
              :class="[
                currentStep > index + 1 ? 'bg-gradient-to-br from-blue-600 to-emerald-500' : 
                currentStep === index + 1 ? 'bg-gradient-to-br from-blue-500 to-emerald-400 ring-4 ring-blue-500/20' : 
                'bg-slate-800 text-slate-500'
              ]"
            >
              <span v-if="currentStep > index + 1">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <polyline points="20 6 9 17 4 12"></polyline>
                </svg>
              </span>
              <span v-else>{{ index + 1 }}</span>
            </div>
            <span class="text-xs text-slate-400 hidden sm:block">
              {{ ['卡密', '设备ID', '确认', '结果', '完成'][index] }}
            </span>
          </div>
        </div>
        <!-- 进度线 -->
        <div class="relative mt-[-27px] z-[-1]">
          <div class="h-0.5 bg-slate-800 w-full"></div>
          <div 
            class="absolute top-0 h-0.5 bg-gradient-to-r from-blue-600 to-emerald-500 transition-all duration-500"
            :style="{ width: ((currentStep - 1) / 4) * 100 + '%' }"
          ></div>
        </div>
      </div>

      <!-- 卡片容器 -->
      <div class="bg-slate-900/60 backdrop-blur-md rounded-2xl border border-slate-800 shadow-xl overflow-hidden transition-all duration-500">
        <!-- 步骤内容区 -->
        <div class="p-6 md:p-8">
          <!-- 步骤1：输入卡密 -->
          <div v-if="currentStep === 1" class="step-content">
            <div class="text-center mb-6">
              <h2 class="text-2xl font-bold mb-2">输入卡密</h2>
              <p class="text-slate-400">请输入你购买的12位激活卡密</p>
            </div>
            
            <div class="space-y-4">
              <div>
                <input
                  v-model="key"
                  @input="validateKey"
                  @blur="validateKey"
                  class="w-full px-4 py-3 bg-slate-800/50 border border-slate-700 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all text-slate-50"
                  placeholder="A1B2C3D4E5F6"
                />
                <p v-if="keyError" class="text-red-400 text-sm mt-1 animate-fade-in">{{ keyError }}</p>
              </div>
              
              <a
                href="https://afdian.com/item/751c4cb48ffa11f08f9f5254001e7c00"
                target="_blank"
                rel="noopener noreferrer"
                class="block w-full text-center"
              >
                <div class="mt-6 px-4 py-3 rounded-lg bg-gradient-to-r from-blue-600/20 to-emerald-500/20 border border-blue-500/30 text-center text-blue-400 hover:from-blue-600/30 hover:to-emerald-500/30 transition-all cursor-pointer">
                  还未购买？点击此处获取卡密
                </div>
              </a>
            </div>
          </div>

          <!-- 步骤2：输入设备ID -->
          <div v-if="currentStep === 2" class="step-content">
            <div class="text-center mb-6">
              <h2 class="text-2xl font-bold mb-2">输入设备ID</h2>
              <p class="text-slate-400">扫描手环上的二维码获取32位ID</p>
            </div>
            
            <div class="space-y-4">
              <div>
                <input
                  v-model="deviceId"
                  @input="validateDeviceId"
                  @blur="validateDeviceId"
                  class="w-full px-4 py-3 bg-slate-800/50 border border-slate-700 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all text-slate-50"
                  placeholder="32位的16进制数据"
                />
                <p v-if="deviceIdError" class="text-red-400 text-sm mt-1 animate-fade-in">{{ deviceIdError }}</p>
              </div>
              
              <div class="mt-4 p-3 bg-slate-800/30 rounded-lg border border-slate-700">
                <p class="text-sm text-slate-400 flex items-start gap-2">
                  <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="flex-shrink-0 mt-0.5">
                    <circle cx="12" cy="12" r="10"></circle>
                    <line x1="12" y1="16" x2="12" y2="12"></line>
                    <line x1="12" y1="8" x2="12.01" y2="8"></line>
                  </svg>
                  设备ID可通过小米运动APP扫描手环二维码获取
                </p>
              </div>
            </div>
          </div>

          <!-- 步骤3：确认数据 -->
          <div v-if="currentStep === 3" class="step-content">
            <div class="text-center mb-6">
              <h2 class="text-2xl font-bold mb-2">确认信息</h2>
              <p class="text-slate-400">请检查以下信息是否正确</p>
            </div>
            
            <div class="bg-slate-800/30 rounded-xl p-5 border border-slate-700 mb-6">
              <div class="space-y-4">
                <div>
                  <p class="text-sm text-slate-400 mb-1">卡密</p>
                  <p class="font-medium">{{ key }}</p>
                </div>
                <div>
                  <p class="text-sm text-slate-400 mb-1">设备ID</p>
                  <p class="font-medium break-all">{{ deviceId }}</p>
                </div>
              </div>
            </div>
            
            <div class="p-4 bg-amber-950/30 rounded-lg border border-amber-900/50">
              <p class="text-sm text-amber-400 flex items-start gap-2">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="flex-shrink-0 mt-0.5">
                  <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"></path>
                  <line x1="12" y1="9" x2="12" y2="13"></line>
                  <line x1="12" y1="17" x2="12.01" y2="17"></line>
                </svg>
                请确认设备ID与你要激活的手环一致，信息错误将导致激活失败且无法退款
              </p>
            </div>
          </div>

          <!-- 步骤4：激活数据 -->
          <div v-if="currentStep === 4" class="step-content">
            <div class="text-center mb-6">
              <h2 class="text-2xl font-bold mb-2">激活数据</h2>
              <p class="text-slate-400">复制以下数据用于设备激活</p>
            </div>
            
            <div class="mb-6">
              <div class="flex items-center gap-2">
                <input
                  v-model="activationData"
                  class="flex-1 px-4 py-3 bg-slate-800/50 border border-slate-700 rounded-lg text-slate-50 cursor-not-allowed"
                  readonly
                />
                <button
                  @click="copyActivationData"
                  class="px-3 py-3 bg-slate-800 border border-slate-700 rounded-lg text-slate-50 hover:bg-slate-700 transition-colors"
                  :disabled="!activationData"
                  aria-label="复制激活数据"
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
                class="mt-2 overflow-hidden transition-all duration-300 ease-out"
                :style="{
                  maxHeight: copySuccess ? '2rem' : '0',
                  opacity: copySuccess ? 1 : 0
                }"
              >
                <p class="text-emerald-400 text-sm flex items-center gap-1">
                  <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                    <polyline points="20 6 9 17 4 12"></polyline>
                  </svg>
                  激活数据已复制到剪贴板
                </p>
              </div>
              
              <!-- 复制提示 -->
              <p 
                v-if="showCopyRequiredHint" 
                class="text-red-400 text-sm mt-2 flex items-center gap-1 animate-fade-in-out"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <circle cx="12" cy="12" r="10"></circle>
                  <line x1="12" y1="8" x2="12" y2="12"></line>
                  <line x1="12" y1="16" x2="12.01" y2="16"></line>
                </svg>
                请先复制激活数据再继续
              </p>
            </div>
            
            <div class="p-4 bg-slate-800/30 rounded-lg border border-slate-700">
              <p class="text-sm text-slate-400">
                复制数据后，打开AstroBox并使用对应插件完成激活。详细教程已发送至你的爱发电私信。
              </p>
            </div>
          </div>

          <!-- 步骤5：完成页面 -->
          <div v-if="currentStep === 5" class="step-content text-center">
            <div class="inline-flex items-center justify-center w-16 h-16 rounded-full bg-gradient-to-br from-emerald-500 to-blue-600 mb-6">
              <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            
            <h2 class="text-2xl font-bold mb-2">操作完成</h2>
            <p class="text-slate-400 mb-6">你的设备激活数据已生成</p>
            
            <div class="bg-slate-800/30 rounded-xl p-5 border border-slate-700 mb-6">
              <p class="text-slate-300 mb-4">加入用户交流群，获取更多使用技巧</p>
              <p class="font-medium text-blue-400">群号：在爱发电自动回复中查看</p>
            </div>
            
            <button 
              @click="resetStepper"
              class="w-full py-3 bg-gradient-to-r from-blue-600 to-emerald-500 rounded-lg font-medium hover:opacity-90 transition-all transform hover:scale-[1.02] active:scale-[0.98]"
            >
              激活另一台设备
            </button>
          </div>
        </div>
        
        <!-- 导航按钮 -->
        <div v-if="currentStep < 5" class="px-6 md:px-8 py-4 bg-slate-900/80 border-t border-slate-800 flex justify-between">
          <button
            @click="prevStep"
            class="px-5 py-2.5 bg-slate-800 border border-slate-700 rounded-lg text-slate-300 hover:bg-slate-700 transition-colors"
            :disabled="currentStep === 1"
          >
            上一步
          </button>
          
          <button
            @click="nextStep"
            class="px-5 py-2.5 rounded-lg font-medium transition-all transform hover:scale-105 active:scale-95"
            :class="[
              isNextDisabled ? 'bg-slate-800 text-slate-500 cursor-not-allowed' : 
              'bg-gradient-to-r from-blue-600 to-emerald-500 text-white hover:opacity-90'
            ]"
            :disabled="isNextDisabled || isProcessing"
          >
            <span v-if="!isProcessing">下一步</span>
            <span v-if="isProcessing" class="flex items-center gap-1">
              <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              处理中...
            </span>
          </button>
        </div>
      </div>
      
      <!-- 页脚信息 -->
      <div class="mt-8 text-center text-xs text-slate-500">
        <p>© 2023 小米手环工具 | 本工具非官方产品，仅供学习交流</p>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';

const key = ref('');
const deviceId = ref('');
const activationData = ref('');
const keyError = ref('');
const deviceIdError = ref('');
const currentStep = ref(1);
const copySuccess = ref(false);
const showCopyRequiredHint = ref(false);
const isProcessing = ref(false);

// 存储卡密哈希值
const cardHash = ref<string>('');

// 设备锁定状态管理
interface DeviceLockState {
  failCount: number;
  lockedUntil: number | null;
  lastUsedCard: string | null;
}

const deviceLockMap = new Map<string, DeviceLockState>();

// 加载和保存锁定状态
const loadDeviceLockState = () => {
  try {
    const savedState = localStorage.getItem('deviceLockMap');
    if (savedState) {
      const parsedState = JSON.parse(savedState);
      Object.keys(parsedState).forEach(deviceId => {
        deviceLockMap.set(deviceId, parsedState[deviceId]);
      });
    }
  } catch (e) {
    console.error('Failed to load device lock state:', e);
  }
};

const saveDeviceLockState = () => {
  try {
    const stateToSave: Record<string, DeviceLockState> = {};
    deviceLockMap.forEach((value, key) => {
      stateToSave[key] = value;
    });
    localStorage.setItem('deviceLockMap', JSON.stringify(stateToSave));
  } catch (e) {
    console.error('Failed to save device lock state:', e);
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

// 计算卡密哈希
const calculateCardHash = async (text: string): Promise<string> => {
  const encoder = new TextEncoder();
  const data = encoder.encode(text);
  const hashBuffer = await crypto.subtle.digest('SHA-256', data);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
};

// 格式化剩余时间
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

// 倒计时管理
const countdownTimer = ref<number | null>(null);
const remainingTime = ref<number>(0);

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
              state.lockedUntil = null;
              saveDeviceLockState();
            } else {
              remainingTime.value = remaining;
            }
          } else {
            clearInterval(countdownTimer.value!);
            countdownTimer.value = null;
            remainingTime.value = 0;
          }
        }, 100);
      }
    } else {
      if (countdownTimer.value) {
        clearInterval(countdownTimer.value);
        countdownTimer.value = null;
      }
      remainingTime.value = 0;
      state.lockedUntil = null;
      saveDeviceLockState();
    }
  } else {
    if (countdownTimer.value) {
      clearInterval(countdownTimer.value);
      countdownTimer.value = null;
    }
    remainingTime.value = 0;
  }
};

// 动态倒计时文本
const dynamicCountdownText = computed(() => {
  if (remainingTime.value <= 0) return '';
  return formatTimeRemaining(remainingTime.value);
});

// 锁定错误信息
const getLockErrorMessage = (step: 1 | 2): string => {
  const state = getDeviceLockState();
  if (state.lockedUntil === null) return '';

  const timeStr = dynamicCountdownText.value || formatTimeRemaining(state.lockedUntil - Date.now());
  return `操作被锁定，请等待 ${timeStr} 后再试`;
};

// 验证卡密
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
      const state = getDeviceLockState();
      state.failCount = 0;
      state.lockedUntil = null;
      state.lastUsedCard = cardText;
      saveDeviceLockState();
      return true;
    }

    if (data.code === 403 && data.isCardInvalid) {
      const state = getDeviceLockState();
      state.failCount++;
      const lockTime = Math.min(300, 60 * Math.pow(2, state.failCount - 1));
      state.lockedUntil = Date.now() + lockTime * 1000;
      keyError.value = '';
      updateCountdown();
      saveDeviceLockState();
      return false;
    }

    if (data.code === 409 && data.isCardUsed) {
      const state = getDeviceLockState();
      state.lockedUntil = Date.now() + 30 * 1000;
      keyError.value = '';
      updateCountdown();
      saveDeviceLockState();
      return false;
    }

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

// 请求签名
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

// 下一步按钮是否禁用
const isNextDisabled = computed(() => {
  if (isProcessing.value) return true;

  if (currentStep.value === 1) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;
    return !key.value || !validateKey() || isLocked;
  } else if (currentStep.value === 2) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;
    return !deviceId.value || !validateDeviceId() || isLocked;
  } else if (currentStep.value === 4) {
    return !copySuccess.value;
  }
  
  return false;
});

// 步骤导航
const prevStep = () => {
  if (currentStep.value > 1) {
    currentStep.value--;
    // 添加步骤切换动画
    animateStepChange();
  }
};

const nextStep = async () => {
  if (isNextDisabled.value) return;

  if (currentStep.value === 1) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;
    
    if (isLocked) {
      keyError.value = getLockErrorMessage(1);
      return;
    }

    if (!validateKey()) return;

    const calculatedHash = await calculateCardHash(key.value);
    cardHash.value = calculatedHash;
    const success = await verifyCard(key.value, calculatedHash);

    if (!success) {
      const state = getDeviceLockState();
      if (state.lockedUntil !== null && Date.now() < state.lockedUntil) {
        keyError.value = getLockErrorMessage(1);
      }
      return;
    }
  } else if (currentStep.value === 2) {
    const state = getDeviceLockState();
    const isLocked = state.lockedUntil !== null && Date.now() < state.lockedUntil;
    
    if (isLocked) {
      deviceIdError.value = getLockErrorMessage(2);
      return;
    }

    if (!validateDeviceId()) return;

    const resultPacket = await requestSignature(key.value, deviceId.value);
    if (!resultPacket) {
      const state = getDeviceLockState();
      if (state.lockedUntil !== null && Date.now() < state.lockedUntil) {
        deviceIdError.value = getLockErrorMessage(2);
      }
      return;
    }
  } else if (currentStep.value === 4) {
    if (!copySuccess.value) {
      showCopyRequiredHint.value = true;
      return;
    }
  }

  if (currentStep.value < 5) {
    currentStep.value++;
    // 添加步骤切换动画
    animateStepChange();
  }
};

// 步骤切换动画
const animateStepChange = () => {
  const stepContents = document.querySelectorAll('.step-content');
  stepContents.forEach(el => {
    el.classList.add('opacity-0', 'translate-y-4');
    setTimeout(() => {
      el.classList.remove('opacity-0', 'translate-y-4');
    }, 200);
  });
};

// 重置步骤
const resetStepper = () => {
  key.value = '';
  deviceId.value = '';
  activationData.value = '';
  currentStep.value = 1;
  copySuccess.value = false;
  keyError.value = '';
  deviceIdError.value = '';
};

// 复制激活数据
const copyActivationData = async () => {
  if (!activationData.value) return;

  try {
    await navigator.clipboard.writeText(activationData.value);
    copySuccess.value = true;
    showCopyRequiredHint.value = false;
    
    // 复制按钮动画反馈
    const copyButton = document.querySelector('[aria-label="复制激活数据"]');
    if (copyButton) {
      copyButton.classList.add('bg-emerald-500/20', 'border-emerald-500/50');
      setTimeout(() => {
        copyButton.classList.remove('bg-emerald-500/20', 'border-emerald-500/50');
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

// 验证卡密格式
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

// 验证设备ID格式
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

// 从URL获取设备ID
const getDeviceIdFromUrl = () => {
  const urlParams = new URLSearchParams(window.location.search);
  return urlParams.get('deviceId');
};

// 初始化
onMounted(() => {
  const deviceIdFromUrl = getDeviceIdFromUrl();
  if (deviceIdFromUrl) {
    deviceId.value = deviceIdFromUrl;
  }

  loadDeviceLockState();
  updateCountdown();
  
  // 初始化步骤内容动画类
  const style = document.createElement('style');
  style.textContent = `
    .step-content {
      transition: all 0.3s ease-out;
    }
    .animate-fade-in {
      animation: fadeIn 0.4s ease-out forwards;
    }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }
    .animate-fade-in-out {
      animation: fadeInOut 1.2s ease-out infinite;
    }
    @keyframes fadeInOut {
      0%, 100% { opacity: 0; transform: translateY(5px); }
      20%, 80% { opacity: 1; transform: translateY(0); }
    }
  `;
  document.head.appendChild(style);
});

onUnmounted(() => {
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value);
  }
});
</script>

<style>
/* 基础样式重置 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  line-height: 1.5;
}

/* 自定义滚动条 */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

::-webkit-scrollbar-track {
  background: #1e293b;
}

::-webkit-scrollbar-thumb {
  background: #475569;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #64748b;
}
</style>
