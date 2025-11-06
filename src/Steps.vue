<script setup>
import { computed } from 'vue';

// 定义组件的 props
const props = defineProps({
  // 步骤数据数组
  // 示例: [{ id: 1, title: '步骤1', status: 'submitted', type: 'check' }, ...]
  steps: {
    type: Array,
    default: () => []
  },
  // 当前激活的步骤索引
  currentStep: {
    type: Number,
    default: 0
  }
});

// 内部图标映射
// 您可以在这里添加更多
const iconMap = {
  check: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41L9 16.17z"/></svg>`,
  user: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/></svg>`,
  edit: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z"/></svg>`,
  wait: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zm-1-13h2v6h-2zm0 8h2v2h-2z"/></svg>`,
};

/**
 * 根据 type 获取图标的 SVG 字符串
 * @param {string} type - 步骤的类型
 */
const getIcon = (type) => {
  return iconMap[type] || '';
};

/**
 * 获取步骤项的 CSS 类
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

  // 根据传入的 status 添加特定状态类
  if (step.status) {
    classes.push(`status-${step.status}`); // e.g., 'status-submitted', 'status-pending'
  }

  return classes.join(' ');
};

</script>

<template>
  <div class="steps-container">
    <!-- 使用 v-for 循环渲染步骤 -->
    <div
      v-for="(step, index) in steps"
      :key="step.id || index"
      :class="getStepClasses(step, index)"
    >
      <!-- 步骤节点 (图标或数字) -->
      <div class="step-node">
        <!-- 优先显示 type 对应的图标 -->
        <div 
          v-if="getIcon(step.type)" 
          class="step-icon" 
          v-html="getIcon(step.type)"
        ></div>
        <!-- 如果没有图标, 显示步骤数字 -->
        <span v-else class="step-number">{{ index + 1 }}</span>
      </div>
      
      <!-- 步骤标题 -->
      <div class="step-title">{{ step.title }}</div>

      <!-- 步骤间的连接线 -->
      <div class="connector-line"></div>
    </div>
  </div>
</template>

<style scoped>
/* 整个步骤图容器 */
.steps-container {
  display: flex;
  width: 100%;
  padding: 20px 0;
  overflow: hidden; /* 隐藏最后一个的多余线条 */
}

/* 每个步骤项 */
.step-item {
  flex: 1; /* 均分宽度 */
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  color: #c0c4cc; /* 默认颜色 (未激活) */
}

/* 步骤节点 (圆圈) */
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
  z-index: 1; /* 覆盖在连接线之上 */
  transition: all 0.3s ease;
}

/* 步骤图标 */
.step-icon {
  width: 18px;
  height: 18px;
  color: inherit; /* 继承父级 .step-node 的颜色 */
}
/* v-html 注入的 svg 默认没有大小 */
.step-icon :deep(svg) {
  width: 100%;
  height: 100%;
  fill: currentColor; /* 使用 CSS 的 color 属性填充 */
}

/* 步骤标题 */
.step-title {
  margin-top: 8px;
  font-size: 14px;
  white-space: nowrap; /* 防止标题换行 */
}

/* 连接线 */
.connector-line {
  position: absolute;
  top: 15px; /* (32px / 2) - 1px (线高) */
  left: 50%;
  right: -50%;
  height: 2px;
  background-color: #c0c4cc; /* 默认线条颜色 */
  z-index: 0;
  transition: background-color 0.3s ease;
}

/* 隐藏最后一个步骤项的连接线 */
.step-item:last-child .connector-line {
  display: none;
}

/* --- 状态样式 --- */

/* 已完成/已提交 状态 */
.step-item.is-completed,
.step-item.status-submitted {
  color: #409eff; /* 主题蓝 */
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
/* 已完成状态下图标颜色 */
.step-item.is-completed .step-icon,
.step-item.status-submitted .step-icon {
  color: #409eff;
}

/* 激活状态 (当前步骤) */
.step-item.is-active {
  color: #303133; /* 激活的文字深色 */
  font-weight: 500;
}
.step-item.is-active .step-node {
  border-color: #409eff;
  background-color: #409eff;
  color: #fff; /* 激活的图标/数字为白色 */
}
.step-item.is-active .step-icon {
  color: #fff;
}


/* 待提交 (pending) 状态 - 示例 (黄色) */
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

/* 未提交 (uncommitted) 状态 - 示例 (红色) */
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