<template>
  <div class="flex justify-center items-center w-full h-full text-white" v-bind="$attrs">
    <div
      :class="`w-full max-w-md p-8 rounded-[2rem] shadow-[0_20px_25px_-5px_rgba(0,0,0,0.1),0_10px_10px_-5px_rgba(0,0,0,0.04)] ${stepCircleContainerClassName}`"
      style="border: 1px solid #222; background-color: #000000; color: #ffffff"
    >
      <div
        :class="`flex items-center justify-center w-full ${stepContainerClassName}`"
        :style="{ marginBottom: isCompleted ? '0' : '2rem' }"
      >
        <template v-for="(_, index) in stepsArray" :key="index + 1">
          <div
            v-if="!renderStepIndicator"
            @click="() => handleStepClick(index + 1)"
            :class="[
              'relative outline-none flex h-8 w-8 items-center justify-center rounded-full font-semibold',
              isCompleted && lockOnComplete ? 'cursor-default' : 'cursor-pointer'
            ]"
            :style="getStepIndicatorStyle(index + 1)"
          >
            <svg
              v-if="getStepStatus(index + 1) === 'complete'"
              class="h-4 w-4 text-black stroke-black"
              fill="none"
              stroke="currentColor"
              :stroke-width="2"
              viewBox="0  0 24 24"
            >
              <Motion
                as="path"
                d="M5 13l4 4L19 7"
                stroke-linecap="round"
                stroke-linejoin="round"
                :initial="{ pathLength: 0, opacity: 0 }"
                :animate="
                  getStepStatus(index + 1) === 'complete'
                    ? { pathLength: 1, opacity: 1 }
                    : { pathLength: 0, opacity: 0 }
                "
              />
            </svg>
            <div v-else-if="getStepStatus(index + 1) === 'active'" class="h-3 w-3 rounded-full bg-black" />
            <span v-else class="text-sm">{{ index + 1 }}</span>
          </div>

          <component
            v-else
            :is="renderStepIndicator"
            :step="index + 1"
            :current-step="currentStep"
            :on-step-click="handleCustomStepClick"
          />

          <div
            v-if="index < totalSteps - 1"
            class="relative ml-2 mr-2 h-0.5 flex-1 overflow-hidden rounded bg-zinc-600"
          >
            <Motion
              as="div"
              class="absolute left-0 top-0 h-full"
              :initial="{ width: 0, backgroundColor: '#52525b' }"
              :animate="
                currentStep > index + 1
                  ? { width: '100%', backgroundColor: '#1246A4' }
                  : { width: 0, backgroundColor: '#52525b' }
              "
              :transition="{ type: 'spring', stiffness: 100, damping: 15, duration: 0.4 }"
            />
          </div>
        </template>
      </div>

      <Motion
        as="div"
        :class="`w-full ${contentClassName}`"
        :style="{
          position: 'relative',
          marginBottom: isCompleted ? '0' : '2rem',
          color: '#ffffff'
        }"
        :animate="{ height: isCompleted ? 0 : `${parentHeight + 1}px` }"
        :transition="{ type: 'spring', stiffness: 200, damping: 25, duration: 0.4 }"
      >
        <AnimatePresence :initial="false" mode="sync" :custom="direction">
          <Motion
            v-if="!isCompleted"
            ref="containerRef"
            as="div"
            :key="currentStep"
            :initial="getStepContentInitial()"
            :animate="{ x: '0%', opacity: 1 }"
            :exit="getStepContentExit()"
            :transition="{ type: 'tween', stiffness: 300, damping: 30, duration: 0.4 }"
            :style="{ position: 'absolute', left: 0, right: 0, top: 0, color: '#ffffff' }"
          >
            <div ref="contentRef" v-if="slots.default && slots.default()[currentStep - 1]">
              <component :is="slots.default()[currentStep - 1]" />
            </div>

            <!-- 第三步的强制阅读提示 -->
            <div
              v-if="currentStep === 3 && showReadWarning"
              class="mt-4 p-3 bg-red-900 border border-red-500 rounded-lg text-red-200 text-sm text-center animate-pulse"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="inline mr-1">
                <circle cx="12" cy="12" r="10"></circle>
                <line x1="12" y1="8" x2="12" y2="12"></line>
                <line x1="12" y1="16" x2="12.01" y2="16"></line>
              </svg>
              请仔细阅读这几句话！
            </div>
          </Motion>
        </AnimatePresence>
      </Motion>

      <div v-if="!isCompleted" :class="`w-full ${footerClassName}`">
        <div :class="`flex w-full ${currentStep !== 1 ? 'justify-between' : 'justify-end'}`">
          <!-- Back Button -->
          <button
            v-if="currentStep !== 1"
            @click="handleBack"
            :disabled="backButtonProps?.disabled"
            :class="backButtonClass"
            v-bind="backButtonProps"
          >
            {{ backButtonText }}
          </button>

          <!-- Next/Complete Button -->
          <button
            v-if="!isLastStep"
            @click="handleNextClick"
            :disabled="currentStep === 3 && countdownSeconds > 0"
            :class="[
              'border-none',
              'transition-all',
              'duration-[350ms]',
              'flex',
              'items-center',
              'justify-center',
              'rounded-full',
              'text-white',
              'font-medium',
              'tracking-tight',
              'px-5',
              'py-2.5',
              'cursor-pointer',
              'hover:bg-[#113671]',
              'disabled:opacity-50',
              'disabled:cursor-not-allowed',
              currentStep === 3 && countdownSeconds > 0 ? 'bg-gray-500' : 'bg-[#1246A4]'
            ]"
          >
            {{
              isLastStep
                ? 'Complete'
                : currentStep === 3 && countdownSeconds > 0
                  ? `下一步 (${countdownSeconds}s)`
                  : nextButtonText
            }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  ref,
  computed,
  useSlots,
  watch,
  onMounted,
  nextTick,
  useTemplateRef,
  type VNode,
  type ButtonHTMLAttributes,
  type Component
} from 'vue';
import { Motion, AnimatePresence } from 'motion-v';

interface StepperProps {
  children?: VNode[];
  initialStep?: number;
  onStepChange?: (step: number) => void;
  onFinalStepCompleted?: () => void;
  stepCircleContainerClassName?: string;
  stepContainerClassName?: string;
  contentClassName?: string;
  footerClassName?: string;
  backButtonProps?: ButtonHTMLAttributes;
  nextButtonProps?: ButtonHTMLAttributes;
  backButtonText?: string;
  nextButtonText?: string;
  disableStepIndicators?: boolean;
  renderStepIndicator?: Component;
  lockOnComplete?: boolean;
  onNextStep?: (currentStep: number) => boolean | Promise<boolean>; //  支持异步验证
}

const props = withDefaults(defineProps<StepperProps>(), {
  initialStep: 1,
  onStepChange: () => {},
  onFinalStepCompleted: () => {},
  stepCircleContainerClassName: '',
  stepContainerClassName: '',
  contentClassName: '',
  footerClassName: '',
  backButtonProps: () => ({}),
  nextButtonProps: () => ({}),
  backButtonText: 'Back',
  nextButtonText: 'Continue',
  disableStepIndicators: true,
  renderStepIndicator: undefined,
  lockOnComplete: false,
  onNextStep: () => true // 默认允许通过
});

const slots = useSlots();
const currentStep = ref(props.initialStep);
const direction = ref(1);
const isCompleted = ref(false);
const parentHeight = ref(0);
const containerRef = useTemplateRef<HTMLDivElement>('containerRef');
const contentRef = useTemplateRef<HTMLDivElement>('contentRef');

const stepsArray = computed(() => slots.default?.() || []);
const totalSteps = computed(() => stepsArray.value.length);
const isLastStep = computed(() => currentStep.value === totalSteps.value);

// 倒计时相关
const countdownSeconds = ref(8);
const countdownTimer = ref<number | null>(null);
const showReadWarning = ref(false); // 显示“请仔细阅读”提示

const backButtonClass = computed(() => {
  return [
    'border-none',
    'bg-[#444C6C]',
    'transition-all',
    'duration-[350ms]',
    'flex',
    'items-center',
    'justify-center',
    'rounded-full',
    'text-black',
    'font-medium',
    'px-5',
    'py-2.5',
    'cursor-pointer',
    'hover:bg-[#113671]',
    'disabled:opacity-50',
    'disabled:cursor-not-allowed',
    currentStep.value === 1 ? 'opacity-50' : '',
    currentStep.value === 1 ? 'cursor-not-allowed' : ''
  ].filter(Boolean).join(' ');
});

const getStepStatus = (step: number) => {
  if (isCompleted.value || currentStep.value > step) return 'complete';
  if (currentStep.value === step) return 'active';
  return 'inactive';
};

const getStepIndicatorStyle = (step: number) => {
  const status = getStepStatus(step);
  switch (status) {
    case 'active':
    case 'complete':
      return { backgroundColor: '#1246A4', color: '#fff' };
    default:
      return { backgroundColor: '#222', color: '#a3a3a3' };
  }
};

const getStepContentInitial = () => ({
  x: direction.value >= 0 ? '-100%' : '100%',
  opacity: 0
});

const getStepContentExit = () => ({
  x: direction.value >= 0 ? '50%' : '-50%',
  opacity: 0
});

const handleStepClick = (step: number) => {
  if (isCompleted.value && props.lockOnComplete) return;
  if (!props.disableStepIndicators) {
    direction.value = step > currentStep.value ? 1 : -1;
    updateStep(step);
  }
};

const handleCustomStepClick = (clicked: number) => {
  if (isCompleted.value && props.lockOnComplete) return;
  if (clicked !== currentStep.value && !props.disableStepIndicators) {
    direction.value = clicked > currentStep.value ? 1 : -1;
    updateStep(clicked);
  }
};

const measureHeight = () => {
  nextTick(() => {
    if (contentRef.value) {
      const height = contentRef.value.offsetHeight;
      if (height > 0 && height !== parentHeight.value) {
        parentHeight.value = height;
      }
    }
  });
};

const updateStep = (newStep: number) => {
  if (newStep >= 1 && newStep <= totalSteps.value) {
    currentStep.value = newStep;
  }
};

const handleBack = () => {
  direction.value = -1;
  updateStep(currentStep.value - 1);
};

//  新增：统一验证函数，支持 await
const validateStep = async (step: number): Promise<boolean> => {
  if (!props.onNextStep) return true;
  return await props.onNextStep(step);
};

const handleNextClick = async () => {
  // 第3步：倒计时拦截逻辑
  if (currentStep.value === 3) {
    if (countdownSeconds.value > 0) {
      if (countdownSeconds.value <= 2) {
        showReadWarning.value = true;
        setTimeout(() => {
          showReadWarning.value = false;
        }, 3000);
      }
      return; // 阻止跳转
    }
  }

  // 执行自定义验证（支持异步）
  const shouldProceed = await validateStep(currentStep.value);
  if (!shouldProceed) return;

  // 正常流程：跳转或完成
  if (isLastStep.value) {
    handleComplete();
  } else {
    handleNext();
  }
};

const handleNext = () => {
  direction.value = 1;
  updateStep(currentStep.value + 1);
};

const handleComplete = () => {
  isCompleted.value = true;
  props.onFinalStepCompleted?.();
};

watch(currentStep, (newStep, oldStep) => {
  props.onStepChange?.(newStep);

  // 清除旧倒计时
  if (countdownTimer.value) {
    clearInterval(countdownTimer.value);
    countdownTimer.value = null;
    countdownSeconds.value = 8; // 重置为 8 秒
    showReadWarning.value = false;
  }

  if (newStep === 3) {
    // 启动 8 秒倒计时
    countdownSeconds.value = 8;
    countdownTimer.value = window.setInterval(() => {
      countdownSeconds.value--;
      if (countdownSeconds.value <= 0) {
        clearInterval(countdownTimer.value!);
        countdownTimer.value = null;
      }
    }, 1000);
  }

  if (newStep !== oldStep && !isCompleted.value) {
    nextTick(measureHeight);
  } else if (!props.lockOnComplete && isCompleted.value) {
    isCompleted.value = false;
    nextTick(measureHeight);
  }
});

onMounted(() => {
  if (props.initialStep !== 1) {
    currentStep.value = props.initialStep;
  }
  measureHeight();
});
</script>

<style scoped>
/* 确保所有文本默认为白色 */
div, span, button {
  color: white;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}
.animate-pulse {
  animation: pulse 1.5s infinite;
}
</style>