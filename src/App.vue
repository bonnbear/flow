<template>
  <div class="container">
    <div class="collapsible-card">
      <div class="card-header" @click="toggle">
        <button class="toggle-btn">
          <span class="arrow" :class="{ expanded: isExpanded }">▼</span>
        </button>
        <h3 class="card-title">{{ title }}</h3>
      </div>

      <div
        class="card-body"
        :class="{ collapsed: !isExpanded, expanded: isExpanded }"
      >
        <!-- 初始加載時的佔位符 -->
        <div v-if="isInitialLoading" class="loading-indicator">
          正在初始化專案列表...
        </div>

        <!-- 專案列表 -->
        <div
          class="progress-item"
          v-for="item in items"
          :key="item.id"
        >
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

        <!-- 滾動加載觸發器和提示 -->
        <div class="load-more-container">
          <!-- 這個 div 是 IntersectionObserver 的觀察目標 -->
          <div ref="loadMoreTrigger"></div>
          <div v-if="isLoadingMore" class="loading-indicator">正在加載更多項目...</div>
          <div v-if="!hasMore && items.length > 0" class="end-indicator">
            沒有更多項目了
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

// --- 原始狀態和設定 ---
const title = ref('季度專案進度 (動態加載)');
const isExpanded = ref(true);
const statusConfig = {
  running: { icon: 'Loading', color: '#4CAF50' },
  completed: { icon: 'CircleCheck', color: '#2196F3' },
  pending: { icon: 'Clock', color: '#FF9800' },
  paused: { icon: 'VideoPause', color: '#9E9E9E' }
};

// --- 新增的狀態用於動態加載 ---
const items = ref([]);
const isInitialLoading = ref(true); // 初始加載狀態
const isLoadingMore = ref(false); // 滾動加載狀態
const hasMore = ref(true); // 是否還有更多數據
const loadMoreTrigger = ref(null); // 用於 IntersectionObserver 的 DOM 元素引用
let projectCounter = 0; // 用於生成唯一專案數據的計數器
let observer = null; // IntersectionObserver 實例

// --- 模擬 API 數據生成 ---
const allNodeSets = [
  ['規劃', '設計', '開發', '測試', '部署'],
  ['需求分析', '原型設計', 'UI/UX 設計', '前端開發', '整合測試'],
  ['效能分析', '重構', '壓力測試', '上線'],
  ['評估', '遷移', '驗證']
];

/**
 * 模擬從後端 API 獲取新專案數據
 * @param {number} limit - 每次獲取的數量
 * @returns {Promise<Array>}
 */
const generateNewItems = (limit) => {
  return new Promise(resolve => {
    setTimeout(() => {
      const newItems = [];
      // 模擬數據總量上限，例如 25 個
      if (projectCounter >= 25) {
        resolve([]); // 返回空陣列表示沒有更多數據
        return;
      }

      for (let i = 0; i < limit; i++) {
        projectCounter++;
        const nodes = allNodeSets[projectCounter % allNodeSets.length];
        const totalSteps = nodes.length;
        // currentStep 可以等於 totalSteps，表示專案已完成所有節點
        const currentStep = Math.floor(Math.random() * (totalSteps + 1));

        let status, statusText;
        if (currentStep === 0) {
          status = 'pending';
          statusText = '待開始';
        } else if (currentStep >= totalSteps) {
          status = 'completed';
          statusText = '已完成';
        } else {
          // 隨機分配 'running' 或 'paused'
          if (Math.random() > 0.3) {
            status = 'running';
            statusText = '進行中';
          } else {
            status = 'paused';
            statusText = '已暫停';
          }
        }

        newItems.push({
          id: `proj-${projectCounter}`,
          label: `動態專案 #${projectCounter}`,
          currentStep: currentStep,
          nodes: nodes,
          status: status,
          statusText: statusText
        });
      }
      resolve(newItems);
    }, 1000); // 模擬 1 秒的網路延遲
  });
};

// --- 核心加載邏輯 ---
const loadItems = async () => {
  // 防止重複加載
  if (isLoadingMore.value || !hasMore.value) return;

  // 根據是否為初次加載設置不同狀態
  if (items.value.length === 0) {
    isInitialLoading.value = true;
  } else {
    isLoadingMore.value = true;
  }

  try {
    const newItems = await generateNewItems(4); // 每次加載 4 個
    if (newItems.length > 0) {
      items.value.push(...newItems);
    } else {
      hasMore.value = false; // 沒有新數據了
    }
  } catch (error) {
    console.error("加載專案失敗:", error);
  } finally {
    isInitialLoading.value = false;
    isLoadingMore.value = false;
  }
};

// --- 組件生命週期鉤子 ---
onMounted(() => {
  // 1. 立即加載初始數據
  loadItems();

  // 2. 設置 IntersectionObserver
  const options = {
    root: document.querySelector('.card-body'), // 在 card-body 內部滾動
    rootMargin: '0px',
    threshold: 1.0 // 當觸發器完全可見時觸發
  };

  observer = new IntersectionObserver((entries) => {
    const entry = entries[0];
    if (entry.isIntersecting && hasMore.value) {
      loadItems(); // 觸發加載更多
    }
  }, options);

  if (loadMoreTrigger.value) {
    observer.observe(loadMoreTrigger.value);
  }
});

onUnmounted(() => {
  // 組件銷毀時，斷開觀察，防止內存洩漏
  if (observer) {
    observer.disconnect();
  }
});


// --- 原始輔助函式 ---
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

// 優化後的進度條寬度計算
const getProgressWidth = (currentStep, totalSteps) => {
  if (totalSteps <= 1) {
    return currentStep > 0 ? 100 : 0;
  }
  // 當 currentStep 大於或等於最後一個節點的索引時，進度為 100%
  const progress = (currentStep / (totalSteps - 1)) * 100;
  return Math.min(progress, 100); // 確保寬度不超過 100%
};

</script>

<style scoped>
/* 整體頁面容器 */
.container {
  padding: 20px;
  background-color: #f0f2f5;
  min-height: 100vh;
}

.collapsible-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  margin: 20px auto;
  max-width: 900px;
  overflow: hidden;
}

.card-header {
  padding: 16px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  align-items: center;
  user-select: none;
  cursor: pointer;
  transition: background 0.2s;
}

.card-header:hover {
  background: #e9ecef;
}

.card-title {
  font-size: 18px;
  font-weight: 600;
  color: #333;
  margin: 0;
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
  padding: 0;
}

.toggle-btn:hover {
  color: #333;
}

.arrow {
  display: inline-block;
  transition: transform 0.3s ease;
}

.arrow.expanded {
  transform: rotate(90deg); /* 箭頭改為向右，符合展開方向 */
}

.card-body {
  overflow: hidden;
  transition: max-height 0.4s ease-in-out, padding 0.4s ease-in-out;
}

.card-body.collapsed {
  max-height: 0;
  padding-top: 0;
  padding-bottom: 0;
}

.card-body.expanded {
  /* 設置固定最大高度和滾動條，以觸發滾動加載 */
  max-height: 500px; 
  overflow-y: auto;
  padding: 0; /* 移除內邊距，讓 progress-item 撐開 */
}

.progress-item {
  display: flex;
  flex-direction: row;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #f0f0f0;
  min-height: 80px;
}

.progress-item:last-of-type {
  border-bottom: none;
}

.item-label {
  width: 25%;
  font-size: 15px;
  color: #333;
  font-weight: 500;
  padding-right: 20px;
  flex-shrink: 0;
}

.progress-track {
  width: 50%;
  position: relative;
  height: 60px;
  display: flex;
  align-items: center;
  padding: 0 10px;
  flex-grow: 1;
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
  width: 25%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  flex-shrink: 0;
  padding-left: 20px;
}

.status-icon {
  width: 18px;
  height: 18px;
}

.status-text {
  font-size: 14px;
  font-weight: 500;
}

.status-text.running { color: #4CAF50; }
.status-text.pending { color: #FF9800; }
.status-text.completed { color: #2196F3; }
.status-text.paused { color: #9E9E9E; }

/* 新增的加載提示樣式 */
.load-more-container {
  padding: 20px;
  text-align: center;
  color: #888;
}
.loading-indicator, .end-indicator {
  font-size: 14px;
  padding: 15px;
}

/* 響應式設計 */
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
  
  .item-label { padding-right: 0; }
  .progress-track { padding: 0; }
  .status-indicator { justify-content: flex-start; padding-left: 0; }
}
</style>
