<!-- src/components/Steps.vue -->
<template>
  <div class="my-steps" :class="[
    `my-steps--${direction}`,
    simple && 'my-steps--simple'
  ]">
    <div
      v-for="(step, index) in steps"
      :key="index"
      class="my-step"
      :style="getStepStyle(index)"
      :class="getStepClass(index)"
    >
      <!-- 头部：包含图标和线条 -->
      <div class="my-step__head" :class="`is-${getStepStatus(index)}`">
        <div class="my-step__line"></div>
        <div class="my-step__icon" :class="`is-${step.icon ? 'icon' : 'text'}`">
          <span v-if="getStepStatus(index) === 'success' || getStepStatus(index) === 'finish'" class="my-step__icon-inner icon-check">
            ✔
          </span>
          <span v-else class="my-step__icon-inner">{{ index + 1 }}</span>
        </div>
      </div>
      <!-- 主体：包含时间、标题和描述 -->
      <div class="my-step__main">
        
        <!-- ==================== 新增：时间显示区域 ==================== -->
        <div 
          v-if="step.startTime || step.endTime" 
          class="my-step__times"
          :class="`is-${getStepStatus(index)}`"
        >
          <div v-if="step.startTime" class="my-step__time-item">
            开始: {{ step.startTime }}
          </div>
          <div v-if="step.endTime" class="my-step__time-item">
            截止: {{ step.endTime }}
          </div>
        </div>
        <!-- ========================================================== -->

        <div class="my-step__title" :class="`is-${getStepStatus(index)}`">
          {{ step.title }}
        </div>
        <div v-if="simple" class="my-step__arrow"></div>
        <div v-else class="my-step__description" :class="`is-${getStepStatus(index)}`">
          {{ step.description }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  steps: {
    type: Array,
    required: true,
    default: () => [],
  },
  active: { type: Number, default: 0 },
  direction: { type: String, default: 'horizontal', validator: val => ['horizontal', 'vertical'].includes(val) },
  space: { type: [Number, String], default: '' },
  alignCenter: { type: Boolean, default: false },
  simple: { type: Boolean, default: false },
  finishStatus: { type: String, default: 'finish', validator: val => ['wait', 'process', 'finish', 'error', 'success'].includes(val) },
  processStatus: { type: String, default: 'process', validator: val => ['wait', 'process', 'finish', 'error', 'success'].includes(val) },
});

const isVertical = computed(() => props.direction === 'vertical');
const isCenter = computed(() => props.alignCenter && !isVertical.value);

const getStepStatus = (index) => {
  if (props.steps[index]?.status) {
    return props.steps[index].status;
  }
  if (index < props.active) {
    return props.finishStatus;
  }
  if (index === props.active) {
    return props.processStatus;
  }
  return 'wait';
};

const getStepClass = (index) => {
  const isLast = index === props.steps.length - 1;
  return [
    `is-${props.direction}`,
    props.simple && 'is-simple',
    isLast && !props.space && !props.simple && 'is-flex',
    isCenter.value && !isVertical.value && !props.simple && 'is-center',
  ];
};

const getStepStyle = (index) => {
  const style = {};
  const space = props.space;
  
  if (space) {
    const spaceValue = /^\d+$/.test(space) ? `${space}px` : space;
    style.flexBasis = spaceValue;
  } else {
    style.flex = 1;
  }
  
  const isLast = index === props.steps.length - 1;
  if (isLast && space) {
    style.flexBasis = 'auto';
    style.flexShrink = 0;
  } else if (isLast && !space) {
    style.flexShrink = 0;
  }

  if (isVertical.value && isLast) {
      style.paddingBottom = '0px';
  }

  return style;
};
</script>

<style scoped>
/* 容器样式 */
.my-steps {
  display: flex;
  width: 100%;
}
.my-steps--vertical {
  flex-direction: column;
  height: 100%;
}

/* 单个步骤项的基础样式 */
.my-step {
  position: relative;
}

/* 水平布局 */
.my-step.is-horizontal {
  display: inline-block;
}
.my-step.is-horizontal .my-step__line {
  height: 2px;
  top: 11px;
  left: 0;
  right: 0;
}

/* 垂直布局 */
.my-step.is-vertical {
  display: flex;
  padding-bottom: 10px;
}
.my-step.is-vertical .my-step__head {
  flex: 0 0 24px;
  width: 24px;
}
.my-step.is-vertical .my-step__line {
  width: 2px;
  top: 24px;
  bottom: 0;
  left: 11px;
}
.my-step.is-vertical .my-step__main {
  padding-left: 10px;
  text-align: left; /* 垂直模式下，所有内容左对齐 */
}
.my-step.is-vertical .my-step__title {
  padding-bottom: 5px;
  line-height: 24px;
}
.my-step:last-child .my-step__line {
  display: none;
}

/* 头部 */
.my-step__head {
  position: relative;
  border-color: #c0c4cc;
}
.my-step__head.is-process {
  color: #409eff;
  border-color: #409eff;
}
.my-step__head.is-finish,
.my-step__head.is-success {
  color: #67c23a;
  border-color: #67c23a;
}
.my-step__head.is-wait {
  color: #c0c4cc;
  border-color: #c0c4cc;
}
.my-step__head.is-error {
  color: #f56c6c;
  border-color: #f56c6c;
}

/* 连接线 */
.my-step__line {
  position: absolute;
  background-color: #c0c4cc;
}
.my-step__head.is-finish .my-step__line,
.my-step__head.is-success .my-step__line {
    background-color: #67c23a;
}

/* 图标 */
.my-step__icon {
  position: relative;
  z-index: 1;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  width: 24px;
  height: 24px;
  font-size: 14px;
  box-sizing: border-box;
  background: #fff;
  border: 2px solid;
  border-color: inherit;
  border-radius: 50%;
  transition: 0.15s ease-out;
}
.my-step__icon.is-text {
  color: inherit;
}
.my-step__icon-inner {
  font-weight: bold;
  user-select: none;
}
.my-step__icon-inner.icon-check {
  font-size: 16px;
  font-weight: bold;
}

/* 主体内容 */
.my-step__main {
  /* 调整 margin-top 为正值，给时间区域留出空间 */
  margin-top: 8px;
  text-align: center;
  white-space: normal;
}

/* ==================== 新增：时间区域样式 ==================== */
.my-step__times {
  font-size: 12px;
  line-height: 1.5;
  color: #c0c4cc; /* 默认灰色 */
  margin-bottom: 4px; /* 与下方标题的间距 */
}
.my-step__times.is-process,
.my-step__times.is-finish,
.my-step__times.is-success {
  color: #606266; /* 激活和完成状态颜色加深 */
}
.my-step__time-item {
  /* 每个时间项独占一行 */
  display: block;
}
/* ========================================================== */

.my-step__title {
  font-size: 16px;
  line-height: 1.2; /* 调整行高，使其更紧凑 */
  color: #c0c4cc;
}
.my-step__title.is-process {
  font-weight: bold;
  color: #303133;
}
.my-step__title.is-finish,
.my-step__title.is-success {
  color: #303133;
}
.my-step__description {
  padding: 0 10px;
  margin-top: 4px; /* 与标题的间距 */
  font-size: 13px;
  line-height: 1.4;
  color: #c0c4cc;
}
.my-step__description.is-process,
.my-step__description.is-finish,
.my-step__description.is-success {
  color: #606266;
}

/* 居中对齐 */
.my-step.is-center .my-step__main {
  text-align: center;
}
</style>
