<template>
  <div class="app">
    <h2>业务流程步骤</h2>
    <CustomSteps :steps="stepsData" />
    
    <div class="controls">
      <button @click="changeStatus">模拟状态变化</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import CustomSteps from './CustomSteps.vue'

const stepsData = ref([
  {
    type: 'submitted',
    title: '提交申请',
    description: '2024-01-10 10:30'
  },
  {
    type: 'processing',
    title: '审核中',
    description: '预计2个工作日'
  },
  {
    type: 'pending',
    title: '待批准',
    description: '等待主管审批'
  },
  {
    type: 'unsubmitted',
    title: '待完成',
    description: '最后环节'
  }
])

const changeStatus = () => {
  // 模拟状态变化
  stepsData.value = [
    {
      type: 'success',
      title: '提交申请',
      description: '已完成'
    },
    {
      type: 'success',
      title: '审核通过',
      description: '2024-01-11 15:20'
    },
    {
      type: 'processing',
      title: '批准中',
      description: '进行中...'
    },
    {
      type: 'pending',
      title: '待完成',
      description: '等待处理'
    }
  ]
}
</script>

<style scoped>
.app {
  max-width: 1000px;
  margin: 0 auto;
  padding: 40px 20px;
}

h2 {
  text-align: center;
  margin-bottom: 40px;
  color: #333;
}

.controls {
  margin-top: 40px;
  text-align: center;
}

button {
  padding: 10px 24px;
  background: #1890ff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

button:hover {
  background: #40a9ff;
}
</style>