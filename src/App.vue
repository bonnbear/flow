<!-- src/App.vue -->
<template>
  <div class="container">
    <h2>基础步骤图 (单一组件 & v-for)</h2>
    <Steps 
      :steps="stepsData" 
      :active="activeStep" 
      finish-status="success" 
      align-center
    />
    <div class="controls">
      <button @click="prevStep" :disabled="activeStep <= 0">上一步</button>
      <button @click="nextStep" :disabled="activeStep >= stepsData.length - 1">下一步</button>
    </div>

    <hr />

    <h2>带时间的步骤图</h2>
    <!-- 使用新的带时间的数据 -->
    <Steps 
      :steps="stepsDataWithTime" 
      :active="1" 
      finish-status="success" 
      align-center
    />

    <hr />

    <h2>垂直步骤图</h2>
    <div style="height: 300px; max-width: 300px;">
      <Steps
        :steps="verticalStepsData"
        :active="1"
        direction="vertical"
        finish-status="success"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Steps from './components/Steps.vue';

const activeStep = ref(1);

// 原始数据
const stepsData = ref([
  { title: '提交订单', description: '订单已提交，等待付款' },
  { title: '付款成功', description: '已完成付款，等待发货' },
  { title: '仓库处理', description: '商品正在打包' },
  { title: '派送中', description: '快递员正在飞速向您赶来' },
  { title: '已签收', description: '感谢您的购买' },
]);

// --- 新增：带时间的数据 ---
// 注意：并非每个步骤都需要有时间，组件会优雅地处理
const stepsDataWithTime = ref([
  { 
    title: '阶段一', 
    description: '需求分析与原型设计',
    startTime: '2024-05-01',
    endTime: '2024-05-10'
  },
  { 
    title: '阶段二', 
    description: 'UI设计与开发',
    startTime: '2024-05-11',
    endTime: '2024-05-25'
  },
  { 
    title: '阶段三', 
    description: '功能测试',
    // 这个步骤只有截止时间
    endTime: '2024-05-30'
  },
  { 
    title: '阶段四', 
    description: '项目上线',
    // 这个步骤没有时间信息
  },
]);


const verticalStepsData = ref([
  { title: '已下单', description: '2025-11-06 14:00:15' },
  { title: '运输中', description: '您的包裹已到达 [深圳中转站]' },
  { title: '派送中', description: '快递员：张三 电话：138****8888' },
  { title: '待签收', description: '请保持电话畅通' },
]);

const nextStep = () => {
  if (activeStep.value < stepsData.value.length - 1) {
    activeStep.value++;
  }
};

const prevStep = () => {
  if (activeStep.value > 0) {
    activeStep.value--;
  }
};
</script>

<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  color: #333;
}
.container {
  padding: 20px;
  max-width: 900px;
  margin: 20px auto;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
}
.controls {
  margin-top: 20px;
  text-align: center;
}
button {
  margin: 0 10px;
  padding: 8px 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #f0f0f0;
  cursor: pointer;
  transition: background-color 0.2s;
}
button:hover:not(:disabled) {
  background-color: #e0e0e0;
}
button:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}
hr {
  margin: 40px 0;
  border: none;
  border-top: 1px solid #eee;
}
</style>
