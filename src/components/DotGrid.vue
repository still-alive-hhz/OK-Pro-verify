<template>
  <section :class="`flex items-center justify-center h-full w-full relative ${className}`" :style="style">
    <div ref="wrapperRef" class="w-full h-full relative">
      <canvas ref="canvasRef" class="absolute inset-0 w-full h-full" />
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch, nextTick, useTemplateRef } from 'vue';

interface Dot {
  cx: number;
  cy: number;
}

export interface DotGridProps {
  dotSize?: number;
  gap?: number;
  baseColor?: string;
  activeColor?: string;
  className?: string;
  style?: Record<string, string | number>;
}

const props = withDefaults(defineProps<DotGridProps>(), {
  dotSize: 2,
  gap: 30,
  baseColor: '#1a1a1a',
  activeColor: '#27FF64',
  className: '',
  style: () => ({})
});

const wrapperRef = useTemplateRef<HTMLDivElement>('wrapperRef');
const canvasRef = useTemplateRef<HTMLCanvasElement>('canvasRef');
const dots = ref<Dot[]>([]);

const buildGrid = () => {
  const wrap = wrapperRef.value;
  const canvas = canvasRef.value;
  if (!wrap || !canvas) return;

  const { width, height } = wrap.getBoundingClientRect();
  const dpr = window.devicePixelRatio || 1;

  canvas.width = width * dpr;
  canvas.height = height * dpr;
  canvas.style.width = `${width}px`;
  canvas.style.height = `${height}px`;
  const ctx = canvas.getContext('2d');
  if (ctx) ctx.scale(dpr, dpr);

  const cols = Math.floor((width + props.gap) / (props.dotSize + props.gap));
  const rows = Math.floor((height + props.gap) / (props.dotSize + props.gap));
  const cell = props.dotSize + props.gap;

  const gridW = cell * cols - props.gap;
  const gridH = cell * rows - props.gap;

  const extraX = width - gridW;
  const extraY = height - gridH;

  const startX = extraX / 2 + props.dotSize / 2;
  const startY = extraY / 2 + props.dotSize / 2;

  const newDots: Dot[] = [];
  for (let y = 0; y < rows; y++) {
    for (let x = 0; x < cols; x++) {
      const cx = startX + x * cell;
      const cy = startY + y * cell;
      newDots.push({ cx, cy });
    }
  }
  dots.value = newDots;
  
  // 绘制点网格
  draw();
};

let resizeObserver: ResizeObserver | null = null;

const draw = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  if (!ctx) return;
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // 绘制所有点
  ctx.fillStyle = props.baseColor;
  for (const dot of dots.value) {
    ctx.beginPath();
    ctx.arc(dot.cx, dot.cy, props.dotSize / 2, 0, Math.PI * 2);
    ctx.fill();
  }
};

onMounted(async () => {
  await nextTick();
  buildGrid();

  const win = window as Window;
  if (typeof win !== 'undefined' && 'ResizeObserver' in win) {
    resizeObserver = new ResizeObserver(buildGrid);
    if (wrapperRef.value) {
      resizeObserver.observe(wrapperRef.value);
    }
  } else if (typeof win !== 'undefined') {
    win.addEventListener('resize', buildGrid);
  }
});

onUnmounted(() => {
  const win = window as Window;
  if (resizeObserver) {
    resizeObserver.disconnect();
  } else if (typeof win !== 'undefined') {
    win.removeEventListener('resize', buildGrid);
  }
});

watch([() => props.dotSize, () => props.gap, () => props.baseColor], () => {
  buildGrid();
});
</script>