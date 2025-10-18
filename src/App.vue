<template>
  <div class="app-container min-h-screen bg-gradient-to-br from-slate-950 via-slate-900 to-slate-950 text-slate-50">
    <!-- 背景装饰 -->
    <div class="fixed inset-0 -z-10 overflow-hidden">
      <div class="absolute inset-0 bg-[radial-gradient(rgba(120,120,120,0.1)_1px,transparent_1px)] bg-[size:30px_30px]"></div>
      <div class="absolute top-1/4 -left-20 w-96 h-96 bg-blue-600 rounded-full filter blur-[120px] opacity-20"></div>
      <div class="absolute bottom-1/3 -right-20 w-96 h-96 bg-emerald-500 rounded-full filter blur-[120px] opacity-20"></div>
    </div>

    <!-- 标题 -->
    <header class="py-6 px-4 sm:px-6 lg:px-8 text-center">
      <div class="inline-flex items-center justify-center w-12 h-12 rounded-full bg-gradient-to-br from-blue-600 to-emerald-500 mb-4">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="10"></circle>
          <polyline points="12 6 12 12 16 14"></polyline>
        </svg>
      </div>
      <h1 class="text-3xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-blue-400 to-emerald-400">Still Alive软件激活工具</h1>
      <p class="mt-2 text-slate-400 max-w-md mx-auto">快速激活你的设备，享受完整功能体验</p>
    </header>

    <!-- 主内容 -->
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
        <div class="p-6 md:p-8">
          
          <!-- 步骤1：输入卡密 -->
          <div v-if="currentStep === 1" class="step-content">
            <div class="text-center mb-6">
              <h2 class="text-2xl font-bold mb-2">输入卡密</h2>
              <p class="text-slate-400">请输入你购买的12位激活卡密</p>
            </div>
            <div class="space-y-4">
              <input
                v-model="key"
                @input="validateKey()"
                class="w-full px-4 py-3 bg-slate-800/50 border border-slate-700 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all text-slate-50"
                placeholder="A1B2C3D4E5F6"
              />
              <p v-if="keyError" class="text-red-400 text-sm animate-fade-in">{{ keyError }}</p>
            </div>
            <a href="https://afdian.com/item/521b2056a07011f087c55254001e7c00" target="_blank" rel="noopener noreferrer" class="block w-full mt-6">
              <div class="px-4 py-3 rounded-lg bg-gradient-to-r from-blue-600/20 to-emerald-500/20 border border-blue-500/30 text-center text-blue-400 hover:from-blue-600/30 hover:to-emerald-500/30 transition-all cursor-pointer">
                还未购买？点击此处获取卡密
              </div>
            </a>
          </div>

          <!-- 步骤2：输入设备ID -->
          <div v-if="currentStep === 2" class="step-content">
            <div class="text-center mb-6">
              <h2 class="text-2xl font-bold mb-2">输入设备ID</h2>
              <p class="text-slate-400">扫描手环上的二维码获取32位ID</p>
            </div>
            <div class="space-y-4">
              <input
                v-model="deviceId"
                @input="validateDeviceId()"
                class="w-full px-4 py-3 bg-slate-800/50 border border-slate-700 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all text-slate-50"
                placeholder="32位16进制ID"
              />
              <p v-if="deviceIdError" class="text-red-400 text-sm animate-fade-in">{{ deviceIdError }}</p>
            </div>
          </div>

          <!-- 步骤3：确认 -->
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
          </div>

          <!-- 步骤4：激活数据 -->
          <div v-if="currentStep === 4" class="step-content">
            <div class="text-center mb-6">
              <h2 class="text-2xl font-bold mb-2">激活数据</h2>
              <p class="text-slate-400">复制以下数据用于设备激活</p>
            </div>
            <div class="mb-6 flex gap-2">
              <input
                v-model="activationData"
                class="flex-1 px-4 py-3 bg-slate-800/50 border border-slate-700 rounded-lg text-slate-50 cursor-not-allowed"
                readonly
              />
              <button @click="copyActivationData" class="px-3 py-3 bg-slate-800 border border-slate-700 rounded-lg text-slate-50 hover:bg-slate-700 transition-colors" :disabled="!activationData || copySuccess">
                {{ copySuccess ? '已复制' : '复制' }}
              </button>
            </div>
            <p v-if="showCopyRequiredHint" class="text-red-400 text-sm animate-fade-in-out">请先复制激活数据才能进行下一步</p>
          </div>

          <!-- 步骤5：完成 -->
          <div v-if="currentStep === 5" class="step-content text-center">
            <h2 class="text-2xl font-bold mb-2">完成</h2>
            <p class="text-slate-400">激活流程已完成，祝你使用愉快！</p>
            <div class="mt-4 py-4 px-6 bg-slate-800/40 rounded-lg border border-slate-700 text-slate-50">
              🎉 你的手环已成功激活！
            </div>
          </div>

          <!-- 下一步按钮 -->
          <div class="mt-6 flex justify-end">
            <button
              @click="nextStep"
              :disabled="isNextDisabled"
              class="px-6 py-3 rounded-lg bg-gradient-to-r from-blue-600 to-emerald-500 hover:from-blue-700 hover:to-emerald-600 disabled:opacity-40 transition-all flex items-center gap-2"
            >
              <span>下一步</span>
              <svg v-if="isProcessing" class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"></path>
              </svg>
            </button>
          </div>

        </div>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

const currentStep = ref(1);
const key = ref('');
const deviceId = ref('');
const activationData = ref('');
const keyError = ref('');
const deviceIdError = ref('');
const isProcessing = ref(false);
const copySuccess = ref(false);
const showCopyRequiredHint = ref(false);

interface DeviceLockState {
  lockedUntil: number | null;
}

function getDeviceLockState(): DeviceLockState {
  return { lockedUntil: null };
}

// 实时校验
function validateKey() {
  keyError.value = key.value.match(/^[A-Z0-9]{12}$/) ? '' : '卡密格式错误，应为12位字母或数字';
}
function validateDeviceId() {
  deviceIdError.value = deviceId.value.match(/^[A-Fa-f0-9]{32}$/) ? '' : '设备ID格式错误，应为32位16进制';
}

// 下一步按钮状态 - 明确返回boolean类型
const isNextDisabled = computed<boolean>(() => {
  if (isProcessing.value) return true;
  
  const state = getDeviceLockState();
  // 明确锁定状态为boolean类型
  const locked: boolean = state.lockedUntil !== null && Date.now() < state.lockedUntil;

  switch (currentStep.value) {
    case 1:
      return !key.value || !!keyError.value || locked;
    case 2:
      return !deviceId.value || !!deviceIdError.value || locked;
    case 4:
      return !copySuccess.value;
    default:
      return false;
  }
});

// 异步模拟
async function calculateCardHash(key: string): Promise<string> { 
  return new Promise<string>(r => setTimeout(() => r('hash_' + key), 500)); 
}
async function verifyCard(key: string, hash: string): Promise<boolean> { 
  return new Promise<boolean>(r => setTimeout(() => r(true), 500)); 
}
async function requestSignature(key: string, deviceId: string): Promise<{data: string}> { 
  return new Promise<{data: string}>(r => setTimeout(() => r({
    data: 'activation_' + key + '_' + deviceId
  }), 500)); 
}

function animateStepChange() {
  // 明确类型为HTMLElement以支持style属性
  const stepContents = document.querySelectorAll<HTMLElement>('.step-content');
  stepContents.forEach(content => {
    if (content) { // 确保元素存在
      content.style.opacity = '0';
      content.style.transform = 'translateY(10px)';
      content.style.transition = 'opacity 0.3s ease, transform 0.3s ease';
      
      setTimeout(() => {
        content.style.opacity = '1';
        content.style.transform = 'translateY(0)';
      }, 50);
    }
  });
}

async function copyActivationData() {
  if (!activationData.value) return;
  try {
    await navigator.clipboard.writeText(activationData.value);
    copySuccess.value = true;
    setTimeout(() => copySuccess.value = false, 2000);
  } catch (err) {
    console.error('复制失败:', err);
  }
}

async function nextStep() {
  if (isNextDisabled.value) return;
  
  try {
    isProcessing.value = true;
    
    switch (currentStep.value) {
      case 1: {
        const hash = await calculateCardHash(key.value);
        const isValid = await verifyCard(key.value, hash);
        if (!isValid) return;
        break;
      }
      case 2: {
        const res = await requestSignature(key.value, deviceId.value);
        activationData.value = res.data;
        break;
      }
      case 4: {
        if (!copySuccess.value) {
          showCopyRequiredHint.value = true;
          setTimeout(() => showCopyRequiredHint.value = false, 2000);
          return;
        }
        break;
      }
    }

    if (currentStep.value < 5) {
      currentStep.value++;
      animateStepChange();
    }
  } catch (e) {
    console.error('操作失败:', e);
  } finally {
    isProcessing.value = false;
  }
}
</script>

<style scoped>
@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
.animate-fade-in {
  animation: fade-in 0.3s ease-out forwards;
}

@keyframes fade-in-out {
  0% { opacity: 0; }
  10% { opacity: 1; }
  90% { opacity: 1; }
  100% { opacity: 0; }
}
.animate-fade-in-out {
  animation: fade-in-out 2s ease-out forwards;
}
</style>
