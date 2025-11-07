<template>
  <div class="collapsible-card">
    <div class="card-header">
      <button class="toggle-btn" @click="toggle">
        <span class="arrow" :class="{ expanded: isExpanded }">▼</span>
      </button>
      <h3 class="card-title">{{ title }}</h3>
    </div>
    
    <div 
      ref="cardBodyRef"
      class="card-body" 
      :class="{ collapsed: !isExpanded, expanded: isExpanded }"
      @scroll="handleScroll"
    >
      <div v-if="isInitialLoading" class="loading-indicator">
        <span class="spinner"></span> 加载数据中...
      </div>

      <div v-else-if="items.length === 0" class="empty-state">
        暂无项目数据
      </div>

      <template v-else>
        <div class="progress-item" v-for="item in items" :key="item.id">
          <div class="item-label">{{ item.label }}</div>
          
          <div class="progress-track">
            <div class="progress-line">
              <div 
                class="progress-line-fill"
                :style="{ width: getProgressWidth(item.currentStep, item.nodes.length) + '%' }"
              ></div>
            </div>
            
            <div class="progress-nodes">
              <div 
                class="node" 
                v-for="(node, index) in item.nodes" 
                :key="index"
              >
                <div 
                  class="node-circle"
                  :class="getNodeStatus(index, item.currentStep)"
                ></div>
                <span 
                  class="node-label"
                  :class="getNodeStatus(index, item.currentStep)"
                >{{ node }}</span>
              </div>
            </div>
          </div>
          
          <div class="status-indicator">
            <component 
              :is="statusConfig[item.status]?.icon" 
              class="status-icon" 
              :style="{ color: statusConfig[item.status]?.color }"
            />
            <span class="status-text" :class="item.status">{{ item.statusText }}</span>
          </div>
        </div>

        <div v-if="isLoadingMore" class="loading-more-indicator">
          <span class="spinner"></span> 正在加载更多...
        </div>

        <div v-if="!hasMoreData && !isLoadingMore && items.length > 0" class="no-more-data">
          --- 没有更多数据了 ---
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
// import { ref } from 'vue'; // 修改: 引入 onMounted
import { ref, onMounted } from 'vue';

const title = ref('季度專案進度');
// const items = ref([...]); // 修改: 初始為空數組
const items = ref([]);
const isExpanded = ref(true);

const statusConfig = {
  running: { icon: 'Loading', color: '#4CAF50' },
  completed: { icon: 'CircleCheck', color: '#2196F3' },
  pending: { icon: 'Clock', color: '#FF9800' },
  paused: { icon: 'VideoPause', color: '#9E9E9E' }
};

const toggle = () => {
  isExpanded.value = !isExpanded.value;
};

const getNodeStatus = (nodeIndex, currentStep) => {
  if (nodeIndex < currentStep) {
    return 'completed';
  } else if (nodeIndex === currentStep) {
    return 'active';
  }
  return 'pending';
};

const getProgressWidth = (currentStep, totalSteps) => {
  if (totalSteps <= 1) return 0; // 防止除以 0
  return (currentStep / (totalSteps - 1)) * 100;
};


// --- 以下為新增的邏輯 ---

// --- 状态 Ref ---
const cardBodyRef = ref(null);      // 對 .card-body 的引用
const isInitialLoading = ref(true); // 是否正在進行初始加載
const isLoadingMore = ref(false);  // 是否正在加載更多
const hasMoreData = ref(true);      // 是否還有更多數據可加載
let page = 1;                       // 當前頁碼
const PAGE_SIZE = 4;                // 每次加載的項目數

// --- 模擬的數據庫 ---
const allMockProjects = [
  { id: 'proj-a', label: '核心引擎升級', currentStep: 2, nodes: ['規劃', '設計', '開發', '測試', '部署'], status: 'running', statusText: '進行中' },
  { id: 'proj-b', label: '使用者介面重構', currentStep: 4, nodes: ['需求分析', '原型設計', 'UI/UX 設計', '前端開發', '整合測試'], status: 'completed', statusText: '已完成' },
  { id: 'proj-c', label: '後端 API 優化', currentStep: 1, nodes: ['效能分析', '重構', '壓力測試', '上線'], status: 'pending', statusText: '待開始' },
  { id: 'proj-d', label: '文件系統遷移', currentStep: 0, nodes: ['評估', '遷移', '驗證'], status: 'paused', statusText: '已暫停' },
  // --- 更多模擬數據 ---
  { id: 'proj-e', label: '新功能开发', currentStep: 0, nodes: ['调研', '开发', '上线'], status: 'pending', statusText: '待开始' },
  { id: 'proj-f', label: '移动端适配', currentStep: 1, nodes: ['设计', '开发', '测试'], status: 'running', statusText: '进行中' },
  { id: 'proj-g', label: '数据迁移', currentStep: 3, nodes: ['A', 'B', 'C', 'D'], status: 'completed', statusText: '已完成' },
  { id: 'proj-h', label: '性能调优', currentStep: 1, nodes: ['分析', '优化', '测试'], status: 'paused', statusText: '已暂停' },
  { id: 'proj-i', label: '安全漏洞修复', currentStep: 2, nodes: ['审计', '修复', '验证', '上线'], status: 'running', statusText: '进行中' },
  { id: 'proj-j', label: 'CI/CD 流水线', currentStep: 0, nodes: ['调研', '实施', '推广'], status: 'pending', statusText: '待开始' }
];

// --- 模擬 API 函數 ---
const fetchProjectsAPI = (pageToFetch, pageSize) => {
  console.log(`Fetching page ${pageToFetch}...`);
  return new Promise((resolve) => {
    setTimeout(() => {
      const start = (pageToFetch - 1) * pageSize;
      const end = pageToFetch * pageSize;
      const newItems = allMockProjects.slice(start, end);
      resolve(newItems);
    }, 1200); // 模擬 1.2 秒的網絡延遲
  });
};

// --- 加載更多數據的函數 ---
const loadMore = async () => {
  // 如果正在加載或沒有更多數據了，則直接返回
  if (isLoadingMore.value || !hasMoreData.value) return;

  isLoadingMore.value = true;
  const newItems = await fetchProjectsAPI(page, PAGE_SIZE);
  
  if (newItems.length > 0) {
    items.value = [...items.value, ...newItems];
    page++; // 頁碼 + 1，準備下次加載
  } else {
    hasMoreData.value = false; // 標記為沒有更多數據
  }
  
  isLoadingMore.value = false;
};

// --- 滾動事件處理函數 ---
const handleScroll = () => {
  const el = cardBodyRef.value;
  if (!el) return;
  
  // 檢查是否滾動到底部 (留出 50px 的緩衝區)
  if (el.scrollTop + el.clientHeight >= el.scrollHeight - 50) {
    loadMore();
  }
};

// --- 組件掛載時，加載初始數據 ---
onMounted(async () => {
  isInitialLoading.value = true;
  const initialItems = await fetchProjectsAPI(page, PAGE_SIZE);
  items.value = initialItems;
  page++; // 頁碼 + 1
  isInitialLoading.value = false;
  
  // 檢查初始加載後是否就沒有更多數據了
  if (initialItems.length < PAGE_SIZE) {
    hasMoreData.value = false;
  }
});

</script>

<style scoped>
.collapsible-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  overflow: hidden;
}

.card-header {
  padding: 16px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  align-items: center;
  user-select: none;
  transition: background 0.2s;
  cursor: pointer; /* 讓標頭看起來可以點擊 */
}

.card-header:hover {
  background: #e9ecef;
}

.card-title {
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.toggle-btn {
  background: none;
  border: none;
  color: #666;
  cursor: pointer;
  font-size: 14px;
  display: flex;
  align-items: center;
  transition: color 0.2s;
  margin-right: 12px;
}

.toggle-btn:hover {
  color: #333;
}

.arrow {
  display: inline-block;
  transition: transform 0.3s ease;
}

.arrow.expanded {
  transform: rotate(180deg);
}

.card-body {
  overflow: hidden;
  transition: max-height 0.3s ease, padding 0.3s ease;
}

.card-body.collapsed {
  max-height: 0;
  padding: 0;
}

.card-body.expanded {
  /* 修改: 設置最大高度並允許滾動 */
  max-height: 500px; /* 您可以根據需要調整這個高度 */
  overflow-y: auto;
  padding: 20px;
}

.progress-item {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #f0f0f0;
  min-height: 80px;
}

.progress-item:last-child {
  border-bottom: none;
}

.item-label {
  width: 33.33%;
  font-size: 15px;
  color: #333;
  font-weight: 500;
  padding-right: 20px;
  flex-shrink: 0;
}

.progress-track {
  width: 33.33%;
  position: relative;
  height: 60px;
  display: flex;
  align-items: center;
  padding: 0 10px;
  flex-shrink: 0;
}

.progress-line {
  position: absolute;
  top: 50%;
  left: 10px;
  right: 10px;
  height: 4px;
  background: #e9ecef;
  transform: translateY(-50%);
  border-radius: 2px;
}

.progress-line-fill {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  background: linear-gradient(90deg, #4CAF50, #66BB6A);
  transition: width 0.5s ease;
  border-radius: 2px;
}

.progress-nodes {
  position: relative;
  width: 100%;
  display: flex;
  justify-content: space-between;
  z-index: 1;
}

.node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
}

.node-circle {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: white;
  border: 3px solid #e9ecef;
  transition: all 0.3s ease;
  position: relative;
}

.node-circle.active {
  background: #4CAF50;
  border-color: #4CAF50;
  box-shadow: 0 0 0 4px rgba(76, 175, 80, 0.2);
  animation: pulse-node 2s infinite;
}

.node-circle.completed {
  background: #4CAF50;
  border-color: #4CAF50;
}

.node-circle.completed::after {
  content: '✓';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  font-size: 11px;
  font-weight: bold;
}

@keyframes pulse-node {
  0%, 100% {
    box-shadow: 0 0 0 4px rgba(76, 175, 80, 0.2);
  }
  50% {
    box-shadow: 0 0 0 8px rgba(76, 175, 80, 0);
  }
}

.node-label {
  font-size: 12px;
  color: #999;
  white-space: nowrap;
  transition: color 0.3s ease;
  max-width: 80px;
  overflow: hidden;
  text-overflow: ellipsis;
}

.node-label.active {
  color: #4CAF50;
  font-weight: 600;
}

.node-label.completed {
  color: #666;
}

.status-indicator {
  width: 33.33%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  flex-shrink: 0;
}

.status-icon {
  width: 18px;
  height: 18px;
}

.status-text {
  font-size: 14px;
  font-weight: 500;
}

.status-text.running {
  color: #4CAF50;
}

.status-text.pending {
  color: #FF9800;
}

.status-text.completed {
  color: #2196F3;
}

.status-text.paused {
  color: #9E9E9E;
}

/* --- 新增 CSS 樣式 --- */

.loading-indicator,
.empty-state,
.loading-more-indicator,
.no-more-data {
  padding: 30px;
  text-align: center;
  color: #888;
  font-size: 14px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.no-more-data {
  color: #bbb;
  font-size: 13px;
}

/* 簡易 CSS 加載動畫 */
.spinner {
  width: 18px;
  height: 18px;
  border: 3px solid rgba(0, 0, 0, 0.1);
  border-left-color: #4CAF50; /* 主題色 */
  border-radius: 50%;
  display: inline-block;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}


@media (max-width: 992px) {
  .progress-item {
    flex-direction: column;
    align-items: stretch;
    gap: 15px;
    padding: 16px;
  }

  .item-label,
  .progress-track,
  .status-indicator {
    width: 100%;
  }

  .status-indicator {
    justify-content: flex-start;
  }
}
</style>
