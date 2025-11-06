<template>
  <div class="flow-chart-container">
    <div class="toolbar">
      <button @click="addNode" class="btn">新增節點</button>
      <button @click="addEdge" class="btn">連接節點</button>
      <button @click="resetFlow" class="btn btn-danger">重置</button>
      <button @click="fitView" class="btn">適應視窗</button>
    </div>
    
    <VueFlow
      v-model:nodes="nodes"
      v-model:edges="edges"
      :default-zoom="1"
      :min-zoom="0.2"
      :max-zoom="4"
      @nodes-change="onNodesChange"
      @edges-change="onEdgesChange"
      @connect="onConnect"
      class="vue-flow"
    >
      <Background pattern-color="#aaa" :gap="16" />
      <Controls />
      <MiniMap />
      
      <template #node-custom="{ data }">
        <div class="custom-node">
          <div class="node-header" :style="{ backgroundColor: data.color }">
            {{ data.label }}
          </div>
          <div class="node-body">
            <div v-if="data.description" class="node-description">
              {{ data.description }}
            </div>
            <div class="node-handles">
              <Handle type="target" :position="Position.Left" />
              <Handle type="source" :position="Position.Right" />
            </div>
          </div>
        </div>
      </template>
    </VueFlow>
    
    <div class="info-panel">
      <h3>流程圖資訊</h3>
      <p>節點數量: {{ nodes.length }}</p>
      <p>連線數量: {{ edges.length }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { VueFlow, useVueFlow, Handle, Position, MarkerType } from '@vue-flow/core'
import { Background } from '@vue-flow/background'
import { Controls } from '@vue-flow/controls'
import { MiniMap } from '@vue-flow/minimap'

// 使用 VueFlow 的組合式 API
const { fitView: vueFlowFitView } = useVueFlow()

// 節點和邊的響應式數據
const nodes = ref([
  {
    id: '1',
    type: 'custom',
    position: { x: 100, y: 100 },
    data: { 
      label: '開始', 
      color: '#4CAF50',
      description: '流程起點'
    }
  },
  {
    id: '2',
    type: 'custom',
    position: { x: 400, y: 100 },
    data: { 
      label: '處理數據', 
      color: '#2196F3',
      description: '執行主要業務邏輯'
    }
  },
  {
    id: '3',
    type: 'custom',
    position: { x: 700, y: 100 },
    data: { 
      label: '決策', 
      color: '#FF9800',
      description: '判斷條件'
    }
  },
  {
    id: '4',
    type: 'custom',
    position: { x: 550, y: 300 },
    data: { 
      label: '成功', 
      color: '#4CAF50',
      description: '流程完成'
    }
  },
  {
    id: '5',
    type: 'custom',
    position: { x: 850, y: 300 },
    data: { 
      label: '失敗', 
      color: '#F44336',
      description: '錯誤處理'
    }
  }
])

const edges = ref([
  {
    id: 'e1-2',
    source: '1',
    target: '2',
    type: 'smoothstep',
    animated: true,
    markerEnd: MarkerType.ArrowClosed
  },
  {
    id: 'e2-3',
    source: '2',
    target: '3',
    type: 'smoothstep',
    animated: true,
    markerEnd: MarkerType.ArrowClosed
  },
  {
    id: 'e3-4',
    source: '3',
    target: '4',
    label: '是',
    type: 'smoothstep',
    animated: true,
    markerEnd: MarkerType.ArrowClosed
  },
  {
    id: 'e3-5',
    source: '3',
    target: '5',
    label: '否',
    type: 'smoothstep',
    animated: true,
    markerEnd: MarkerType.ArrowClosed
  }
])

let nodeIdCounter = 6

// 新增節點
const addNode = () => {
  const newNode = {
    id: `${nodeIdCounter++}`,
    type: 'custom',
    position: { x: Math.random() * 500 + 100, y: Math.random() * 400 + 100 },
    data: { 
      label: `新節點 ${nodeIdCounter - 1}`, 
      color: '#9C27B0',
      description: '點擊編輯內容'
    }
  }
  nodes.value.push(newNode)
}

// 新增連線（連接最後兩個節點）
const addEdge = () => {
  if (nodes.value.length < 2) {
    alert('至少需要兩個節點才能建立連線！')
    return
  }
  
  const lastTwo = nodes.value.slice(-2)
  const newEdge = {
    id: `e${lastTwo[0].id}-${lastTwo[1].id}`,
    source: lastTwo[0].id,
    target: lastTwo[1].id,
    type: 'smoothstep',
    animated: true,
    markerEnd: MarkerType.ArrowClosed
  }
  
  // 檢查是否已存在相同的連線
  const exists = edges.value.some(e => 
    e.source === newEdge.source && e.target === newEdge.target
  )
  
  if (!exists) {
    edges.value.push(newEdge)
  } else {
    alert('這兩個節點之間已經存在連線！')
  }
}

// 重置流程圖
const resetFlow = () => {
  if (confirm('確定要重置流程圖嗎？所有更改將會丟失。')) {
    nodes.value = [
      {
        id: '1',
        type: 'custom',
        position: { x: 250, y: 100 },
        data: { 
          label: '開始', 
          color: '#4CAF50',
          description: '流程起點'
        }
      }
    ]
    edges.value = []
    nodeIdCounter = 2
  }
}

// 適應視窗
const fitView = () => {
  vueFlowFitView({ padding: 0.2, duration: 300 })
}

// 節點變化處理
const onNodesChange = (changes) => {
  console.log('節點變化:', changes)
}

// 邊變化處理
const onEdgesChange = (changes) => {
  console.log('連線變化:', changes)
}

// 連接處理
const onConnect = (params) => {
  const newEdge = {
    ...params,
    type: 'smoothstep',
    animated: true,
    markerEnd: MarkerType.ArrowClosed
  }
  edges.value.push(newEdge)
}

onMounted(() => {
  // 組件掛載後適應視窗
  setTimeout(() => {
    fitView()
  }, 100)
})
</script>

<style scoped>
.flow-chart-container {
  width: 100%;
  height: 100vh;
  position: relative;
  background: #f5f5f5;
}

.toolbar {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 10;
  display: flex;
  gap: 10px;
  background: white;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  background: #2196F3;
  color: white;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s;
}

.btn:hover {
  background: #1976D2;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.btn-danger {
  background: #F44336;
}

.btn-danger:hover {
  background: #D32F2F;
}

.vue-flow {
  height: 100%;
}

.custom-node {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  min-width: 180px;
  overflow: hidden;
  transition: all 0.3s;
}

.custom-node:hover {
  box-shadow: 0 4px 16px rgba(0,0,0,0.25);
  transform: scale(1.05);
}

.node-header {
  padding: 12px;
  color: white;
  font-weight: bold;
  text-align: center;
  font-size: 14px;
}

.node-body {
  padding: 12px;
  position: relative;
}

.node-description {
  font-size: 12px;
  color: #666;
  margin-bottom: 8px;
}

.node-handles {
  position: relative;
}

.info-panel {
  position: absolute;
  bottom: 20px;
  right: 20px;
  background: white;
  padding: 15px 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  z-index: 10;
}

.info-panel h3 {
  margin: 0 0 10px 0;
  font-size: 16px;
  color: #333;
}

.info-panel p {
  margin: 5px 0;
  font-size: 14px;
  color: #666;
}
</style>

<style>
/* Vue Flow 全域樣式 */
@import '@vue-flow/core/dist/style.css';
@import '@vue-flow/core/dist/theme-default.css';
@import '@vue-flow/controls/dist/style.css';
@import '@vue-flow/minimap/dist/style.css';
@import '@vue-flow/background/dist/style.css';

.vue-flow__handle {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #555;
  border: 2px solid white;
}

.vue-flow__handle:hover {
  background: #2196F3;
  transform: scale(1.2);
}

.vue-flow__edge-path {
  stroke-width: 2;
}

.vue-flow__edge.animated path {
  stroke-dasharray: 5;
  animation: dashdraw 0.5s linear infinite;
}

@keyframes dashdraw {
  from {
    stroke-dashoffset: 10;
  }
}

.vue-flow__minimap {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.vue-flow__controls {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.vue-flow__controls-button {
  border: none;
  background: white;
  transition: all 0.2s;
}

.vue-flow__controls-button:hover {
  background: #f0f0f0;
}
</style>
