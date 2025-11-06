<script setup>
import { ref } from 'vue';
// 1. 引入您的 Steps.vue 组件
import Steps from './Steps.vue'; // 假设 Steps.vue 在同级目录

// 2. 定义步骤图所需的数据
const stepsData = ref([
  { id: 1, title: '已提交', status: 'submitted', type: 'check' },
  { id: 2, title: '审核中', status: 'pending', type: 'wait' },
  { id: 3, title: '待修改', status: 'uncommitted', type: 'edit' },
  { id: 4, title: '待分配', status: 'waiting', type: 'user' },
  { id: 5, title: '已完成', status: 'waiting', type: 'check' } // 最后一个用 'waiting' 状态, 但会根据 currentStep 变灰
]);

// 3. 定义当前激活的步骤 (索引从 0 开始)
const current = ref(2); // 当前在 '待修改' 这一步

// 模拟操作
const nextStep = () => {
  if (current.value < stepsData.value.length - 1) {
    current.value++;
  }
};

const prevStep = () => {
  if (current.value > 0) {
    current.value--;
  }
};
</script>

<template>
  <div class="app-container">
    <h2 style="text-align: center;">动态步骤图组件</h2>
    
    <!-- 4. 使用组件 -->
    <Steps :steps="stepsData" :current-step="current" />

    <div class="controls">
      <button @click="prevStep" :disabled="current === 0">上一步</button>
      <button @click="nextStep" :disabled="current === stepsData.length - 1">下一步</button>
      <p>当前步骤索引: {{ current }} ({{ stepsData[current].title }})</p>
    </div>
  </div>
</template>

<style>
/* 示例 App 的一些简单样式 */
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  padding: 20px;
  background-color: #f4f7f6;
}
.app-container {
  max-width: 900px;
  margin: 40px auto;
  padding: 30px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
}
.controls {
  text-align: center;
  margin-top: 30px;
}
.controls button {
  font-size: 14px;
  padding: 10px 18px;
  margin: 0 10px;
  border: none;
  border-radius: 5px;
  background-color: #409eff;
  color: white;
  cursor: pointer;
  transition: background-color 0.2s;
}
.controls button:hover:not(:disabled) {
  background-color: #66b1ff;
}
.controls button:disabled {
  background-color: #c0c4cc;
  cursor: not-allowed;
}
</style>