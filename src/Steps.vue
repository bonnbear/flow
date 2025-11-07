<script setup>
import { ref, computed, onMounted, onUnmounted, nextTick } from 'vue';

// --- Props (与之前相同) ---
const props = defineProps({
  steps: {
    type: Array,
    default: () => []
  },
  currentStep: {
    type: Number,
    default: 0
  }
});

// --- 内部图标映射 (与之前相同) ---
const iconMap = {
  check: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41L9 16.17z"/></svg>`,
  user: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>`,
  edit: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>`,
  wait: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zm-1-13h2v6h-2zm0 8h2v2h-2z"/></svg>`,
};

// --- Refs for DOM elements ---
// 用于获取滚动容器的 DOM 引用
const scrollContainerRef = ref(null);

// --- State for UI control ---
// 状态：内容是否溢出，决定是否显示滚动按钮
const isOverflowing = ref(false);
// 状态：是否可以向左滚动
const canScrollLeft = ref(false);
// 状态：是否可以向右滚动
const canScrollRight = ref(false);

// --- 方法 ---

/**
 * 根据 type 获取图标的 SVG 字符串 (与之前相同)
 * @param {string} type - 步骤的类型
 */
const getIcon = (type) => {
  return iconMap[type] || '';
};

/**
 * 获取步骤项的 CSS 类 (与之前相同)
 * @param {object} step - 步骤对象
 * @param {number} index - 步骤索引
 */
const getStepClasses = (step, index) => {
  const classes = ['step-item'];
  
  if (index === props.currentStep) {
    classes.push('is-active');
  } else if (index < props.currentStep) {
    classes.push('is-completed');
  }

  if (step.status) {
    classes.push(`status-${step.status}`);
  }

  return classes.join(' ');
};

/**
 * 更新滚动状态，决定按钮是否可点击
 */
const updateScrollState = () => {
  const el = scrollContainerRef.value;
  if (!el) return;

  // 使用一个小的容差值 (1) 来处理可能的浮点数精度问题
  const scrollEnd = Math.ceil(el.scrollLeft) + el.clientWidth + 1 >= el.scrollWidth;

  canScrollLeft.value = el.scrollLeft > 0;
  canScrollRight.value = !scrollEnd;
};

/**
 * 检查内容是否溢出容器
 */
const checkForOverflow = () => {
  // 使用 nextTick 确保 DOM 已经更新
  nextTick(() => {
    const el = scrollContainerRef.value;
    if (!el) return;

    // 如果内容的滚动宽度大于容器的可见宽度，则判断为溢出
    isOverflowing.value = el.scrollWidth > el.clientWidth;
    // 检查完溢出状态后，立即更新一次滚动按钮的状态
    updateScrollState();
  });
};

/**
 * 向左滚动
 */
const scroll = (direction) => {
  const el = scrollContainerRef.value;
  if (!el) return;

  // 每次滚动的距离为容器宽度的 80%，可以根据需要调整
  const scrollAmount = el.clientWidth * 0.8 * direction;
  
  el.scrollBy({
    left: scrollAmount,
    behavior: 'smooth' // 平滑滚动
  });
};


// --- Lifecycle Hooks ---
onMounted(() => {
  // 组件挂载后，检查是否溢出
  checkForOverflow();
  // 添加窗口大小变化的监听器，以便在窗口大小改变时重新检查
  window.addEventListener('resize', checkForOverflow);
});

onUnmounted(() => {
  // 组件卸载时，移除监听器，防止内存泄漏
  window.removeEventListener('resize', checkForOverflow);
});

</script>

<template>
  <!-- 新增：整个组件的最外层包裹容器 -->
  <div class="steps-component-wrapper">
    <!-- 新增：左滚动按钮 -->
    <button
      v-if="isOverflowing"
      class="scroll-btn left"
      :disabled="!canScrollLeft"
      @click="scroll(-1)"
      aria-label="Scroll Left"
    >
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
        <path d="M15.41 7.41L14 6l-6 6 6 6 1.41-1.41L10.83 12z"/>
      </svg>
    </button>

    <!-- 修改：之前的 .steps-container 现在作为滚动视口 -->
    <div
      class="steps-container"
      ref="scrollContainerRef"
      @scroll.passive="updateScrollState"
    >
      <!-- 新增：内容包裹层，用于横向排列所有步骤 -->
      <div class="steps-content">
        <div
          v-for="(step, index) in steps"
          :key="step.id || index"
          :class="getStepClasses(step, index)"
        >
          <div class="step-node">
            <div 
              v-if="getIcon(step.type)" 
              class="step-icon" 
              v-html="getIcon(step.type)"
            ></div>
            <span v-else class="step-number">{{ index + 1 }}</span>
          </div>
          
          <div class="step-title">{{ step.title }}</div>

          <div class="connector-line"></div>
        </div>
      </div>
    </div>
    
    <!-- 新增：右滚动按钮 -->
    <button
      v-if="isOverflowing"
      class="scroll-btn right"
      :disabled="!canScrollRight"
      @click="scroll(1)"
      aria-label="Scroll Right"
    >
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
        <path d="M10 6L8.59 7.41 13.17 12l-4.58 4.59L10 18l6-6z"/>
      </svg>
    </button>
  </div>
</template>

<style scoped>
/* --- 新增样式 --- */

/* 新增：最外层包裹容器 */
.steps-component-wrapper {
  position: relative;
  width: 100%;
  padding: 0 40px; /* 为按钮留出空间 */
  box-sizing: border-box;
}

/* 新增：滚动按钮通用样式 */
.scroll-btn {
  position: absolute;
  top: 0;
  bottom: 0;
  margin: auto 0;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.9);
  border: 1px solid #e0e0e0;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  transition: opacity 0.3s, box-shadow 0.3s;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.scroll-btn:hover {
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}
.scroll-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
  box-shadow: none;
}
.scroll-btn svg {
  width: 24px;
  height: 24px;
  fill: #606266;
}
.scroll-btn.left {
  left: 0px;
}
.scroll-btn.right {
  right: 0px;
}

/* 新增：实际承载步骤的内容区 */
.steps-content {
  display: inline-flex; /* 使用 inline-flex 让其宽度由内容决定 */
  align-items: flex-start;
  padding: 20px 0;
  min-width: 100%; /* 确保在内容不足时也能撑满容器 */
}

/* --- 修改后的样式 --- */

/* 修改：步骤图容器，现在作为滚动视口 */
.steps-container {
  display: flex; /* 保持 flex 布局 */
  width: 100%;
  overflow-x: auto; /* 允许横向滚动 */
  -ms-overflow-style: none;  /* IE and Edge */
  scrollbar-width: none;  /* Firefox */
}
/* 隐藏 Chrome, Safari 和 Opera 的滚动条 */
.steps-container::-webkit-scrollbar {
  display: none;
}

/* 修改：每个步骤项 */
.step-item {
  /* flex: 1; -> 移除 flex: 1，不再让其均分宽度 */
  flex-shrink: 0; /* 防止步骤项被压缩 */
  min-width: 120px; /* 设置一个最小宽度，确保内容不会挤在一起 */
  padding: 0 10px; /* 增加一些内边距 */
  box-sizing: border-box;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  color: #c0c4cc;
}

/* --- 未改变的样式 --- */

.step-node {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: 2px solid #c0c4cc;
  background-color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 16px;
  z-index: 1;
  transition: all 0.3s ease;
  flex-shrink: 0; /* 防止节点自身被压缩 */
}

.step-icon {
  width: 18px;
  height: 18px;
  color: inherit;
}
.step-icon :deep(svg) {
  width: 100%;
  height: 100%;
  fill: currentColor;
}

.step-title {
  margin-top: 8px;
  font-size: 14px;
  white-space: nowrap;
}

.connector-line {
  position: absolute;
  top: 15px;
  left: 50%;
  right: -50%;
  height: 2px;
  background-color: #c0c4cc;
  z-index: 0;
  transition: background-color 0.3s ease;
}

.step-item:first-child .connector-line {
  /* 对于第一个元素，我们只需要它右半部分的连接线 */
  left: 50%;
  right: -50%;
}

.step-item:last-child .connector-line {
  display: none;
}

.step-item.is-completed,
.step-item.status-submitted {
  color: #409eff;
}
.step-item.is-completed .step-node,
.step-item.status-submitted .step-node {
  border-color: #409eff;
  color: #409eff;
}
.step-item.is-completed .connector-line,
.step-item.status-submitted .connector-line {
  background-color: #409eff;
}
.step-item.is-completed .step-icon,
.step-item.status-submitted .step-icon {
  color: #409eff;
}

.step-item.is-active {
  color: #303133;
  font-weight: 500;
}
.step-item.is-active .step-node {
  border-color: #409eff;
  background-color: #409eff;
  color: #fff;
}
.step-item.is-active .step-icon {
  color: #fff;
}

.step-item.status-pending {
  color: #e6a23c;
}
.step-item.status-pending .step-node {
  border-color: #e6a23c;
  color: #e6a23c;
}
.step-item.status-pending .step-icon {
  color: #e6a23c;
}

.step-item.status-uncommitted {
  color: #f56c6c;
}
.step-item.status-uncommitted .step-node {
  border-color: #f56c6c;
  color: #f56c6c;
}
.step-item.status-uncommitted .step-icon {
  color: #f56c6c;
}
</style>