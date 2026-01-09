<template>
  <div class="mountain-picker">
    <!-- 可爱的云朵装饰 -->
    <div class="clouds">
      <div class="cloud cloud1">☁️</div>
      <div class="cloud cloud2">☁️</div>
      <div class="cloud cloud3">☁️</div>
    </div>

    <h1 class="title">
      <span class="title-emoji">🏔️</span>
      <span class="title-text">周末爬山抽签器</span>
      <span class="title-emoji">🎈</span>
    </h1>
    
    <!-- 主抽签区域 -->
    <div class="display-area" :class="{ spinning: isSpinning }">
      <div class="stars" v-if="selectedMountain && !isSpinning">
        <span class="star">⭐</span>
        <span class="star">⭐</span>
        <span class="star">⭐</span>
      </div>
      
      <!-- 显示多个山名 -->
      <div class="mountains-carousel">
        <div 
          v-for="(mountain, index) in visibleMountains" 
          :key="index"
          class="mountain-name"
          :class="{
            'is-current': index === 2,
            'is-prev': index === 1,
            'is-next': index === 3,
            'is-far': index === 0 || index === 4,
            spinning: isSpinning
          }"
        >
          {{ mountain }}
        </div>
      </div>
      
      <div class="mountain-emoji" v-if="!isSpinning && selectedMountain">
        🎉🎊🎉
      </div>
    </div>

    <!-- 控制按钮 -->
    <div class="controls">
      <button 
        @click="start" 
        :disabled="isSpinning"
        class="btn btn-start"
      >
        <span class="btn-icon">🚀</span>
        <span class="btn-text">{{ selectedMountain && !isConfirmed ? '再抽一次' : '开始' }}</span>
      </button>
      <button 
        @click="stop" 
        :disabled="!isSpinning"
        class="btn btn-stop"
      >
        <span class="btn-icon">🛑</span>
        <span class="btn-text">停止</span>
      </button>
      <button 
        @click="confirmChoice" 
        :disabled="!selectedMountain || isSpinning || isConfirmed"
        class="btn btn-confirm"
        v-if="selectedMountain && !isConfirmed"
      >
        <span class="btn-icon">✅</span>
        <span class="btn-text">就这个了！</span>
      </button>
    </div>

    <!-- 结果信息卡片 -->
    <div class="info-card" v-if="selectedMountain">
      <div class="info-header">
        <span class="trophy">🏆</span>
        <span class="info-title">本周目标</span>
        <span class="trophy">🏆</span>
      </div>
      <div class="info-content">
        <h2 class="mountain-result">{{ selectedMountain.name }}</h2>
        <div class="info-details">
          <div class="info-item">
            <span class="info-emoji">📍</span>
            <span>{{ selectedMountain.location }}</span>
          </div>
          <div class="info-item">
            <span class="info-emoji">⛰️</span>
            <span>{{ selectedMountain.elevation }}米</span>
          </div>
          <div class="info-item">
            <span class="info-emoji">💪</span>
            <span>{{ selectedMountain.difficulty }}</span>
          </div>
        </div>
      </div>
      <div class="encouragement">加油！我们一起去探险吧！🎒</div>
    </div>

    <!-- 历史记录 -->
    <div class="history-card" v-if="history.length > 0">
      <h3 class="history-title">
        <span>🎯</span> 最终决定 <span>✨</span>
      </h3>
      <ul class="history-list">
        <li v-for="(item, index) in history" :key="index" class="history-item">
          <span class="history-badge">{{ index + 1 }}</span>
          <span class="history-date">{{ item.date }}</span>
          <span class="history-name">{{ item.name }}</span>
          <span class="history-icon">🏔️</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import Papa from 'papaparse'
import mountainsCSV from '../assets/mountains.csv?raw'

const mountains = ref([])
const currentMountain = ref('点击开始')
const isSpinning = ref(false)
const selectedMountain = ref(null)
const isConfirmed = ref(false)
const history = ref([])
const currentIndex = ref(0)  // 改为响应式变量

let spinInterval = null
let speed = 50

// 计算可见的山名列表（当前山名及其前后的山）
const visibleMountains = computed(() => {
  if (mountains.value.length === 0) {
    return ['点击开始', '', '', '', '']
  }
  
  const total = mountains.value.length
  const result = []
  
  // 显示5个山名：前2个、当前、后2个
  for (let i = -2; i <= 2; i++) {
    let index = (currentIndex.value + i + total) % total
    result.push(mountains.value[index]?.name || '')
  }
  
  return result
})

// 加载 CSV 数据
onMounted(() => {
  Papa.parse(mountainsCSV, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
      mountains.value = results.data
      console.log('加载了', mountains.value.length, '座山')
    }
  })

  // 加载历史记录
  const saved = localStorage.getItem('hikingHistory')
  if (saved) {
    history.value = JSON.parse(saved)
  }
})

// 开始旋转
function start() {
  if (mountains.value.length === 0) {
    alert('山的数据还未加载完成')
    return
  }

  isSpinning.value = true
  selectedMountain.value = null
  isConfirmed.value = false
  speed = 50

  spinInterval = setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % mountains.value.length
    currentMountain.value = mountains.value[currentIndex.value].name
  }, speed)
}

// 停止旋转（立即停止）
function stop() {
  if (!isSpinning.value) return

  // 清除快速旋转
  clearInterval(spinInterval)
  
  // 立即停止
  isSpinning.value = false
  selectedMountain.value = mountains.value[currentIndex.value]
}


// 确认选择并保存到历史记录
function confirmChoice() {
  if (!selectedMountain.value || isConfirmed.value) return
  
  isConfirmed.value = true
  
  // 保存到历史记录
  const record = {
    name: selectedMountain.value.name,
    date: new Date().toLocaleDateString('zh-CN')
  }
  history.value.unshift(record)
  
  // 只保留最近10条
  if (history.value.length > 10) {
    history.value = history.value.slice(0, 10)
  }
  
  localStorage.setItem('hikingHistory', JSON.stringify(history.value))
}
</script>

<style scoped>
.mountain-picker {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem 1rem;
  text-align: center;
  position: relative;
  min-height: 100vh;
}

/* 云朵动画 */
.clouds {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 200px;
  pointer-events: none;
  overflow: hidden;
}

.cloud {
  position: absolute;
  font-size: 3rem;
  animation: float 20s infinite ease-in-out;
  opacity: 0.6;
}

.cloud1 {
  top: 20px;
  left: -50px;
  animation-delay: 0s;
}

.cloud2 {
  top: 80px;
  right: -50px;
  animation-delay: 7s;
  font-size: 2.5rem;
}

.cloud3 {
  top: 40px;
  left: 30%;
  animation-delay: 14s;
  font-size: 2rem;
}

@keyframes float {
  0%, 100% {
    transform: translateX(0) translateY(0);
  }
  25% {
    transform: translateX(100vw) translateY(-20px);
  }
  50% {
    transform: translateX(100vw) translateY(20px);
  }
  75% {
    transform: translateX(100vw) translateY(-10px);
  }
}

/* 标题样式 */
.title {
  font-size: 2.8rem;
  margin: 3rem 0 2rem;
  color: #ff6b9d;
  font-weight: 900;
  text-shadow: 3px 3px 0 #ffd93d, 6px 6px 0 #6bcf7f;
  animation: bounce 2s infinite;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.title-emoji {
  font-size: 3.5rem;
  animation: rotate 3s infinite ease-in-out;
  display: inline-block;
}

@keyframes rotate {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-15deg); }
  75% { transform: rotate(15deg); }
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

/* 主显示区域 */
.display-area {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  border-radius: 30px;
  padding: 4rem 2rem;
  margin-bottom: 2rem;
  box-shadow: 0 15px 40px rgba(102, 126, 234, 0.4),
              0 0 0 8px #fff,
              0 0 0 12px #ffd93d;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
}

.display-area::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(45deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  transform: rotate(45deg);
  animation: shine 3s infinite;
}

@keyframes shine {
  0% { transform: translateX(-100%) translateY(-100%) rotate(45deg); }
  100% { transform: translateX(100%) translateY(100%) rotate(45deg); }
}

.display-area.spinning {
  animation: shake 0.3s infinite;
}

@keyframes shake {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-2deg); }
  75% { transform: rotate(2deg); }
}

.stars {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 2rem;
  animation: twinkle 1s infinite;
}

.star {
  display: inline-block;
  animation: spin 2s infinite linear;
}

.star:nth-child(2) {
  animation-delay: 0.3s;
}

.star:nth-child(3) {
  animation-delay: 0.6s;
}

@keyframes twinkle {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* 山名轮播容器 */
.mountains-carousel {
  position: relative;
  min-height: 8rem;
  display: flex;
  align-items: center;
  justify-content: center;
  perspective: 1000px;
}

.mountain-name {
  position: absolute;
  font-size: 2rem;
  font-weight: 900;
  color: rgba(255, 255, 255, 0.3);
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  z-index: 1;
  letter-spacing: 0.05em;
  transition: all 0.3s ease;
  white-space: nowrap;
}

/* 当前显示的山名 - 最明显 */
.mountain-name.is-current {
  font-size: 3.5rem;
  color: #fff;
  text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.3),
               0 0 20px rgba(255, 255, 255, 0.5);
  z-index: 5;
  transform: translateY(0) scale(1);
}

/* 前一个和后一个 - 稍小 */
.mountain-name.is-prev {
  font-size: 2.5rem;
  color: rgba(255, 255, 255, 0.6);
  z-index: 3;
  transform: translateY(-2.5rem) scale(0.8);
}

.mountain-name.is-next {
  font-size: 2.5rem;
  color: rgba(255, 255, 255, 0.6);
  z-index: 3;
  transform: translateY(2.5rem) scale(0.8);
}

/* 更远的山名 - 更小更透明 */
.mountain-name.is-far {
  font-size: 1.8rem;
  color: rgba(255, 255, 255, 0.2);
  z-index: 1;
}

.mountain-name.is-far:first-child {
  transform: translateY(-4.5rem) scale(0.6);
}

.mountain-name.is-far:last-child {
  transform: translateY(4.5rem) scale(0.6);
}

/* 旋转时的动画 */
.mountain-name.is-current.spinning {
  animation: pulse 0.3s infinite, colorChange 0.5s infinite;
}

@keyframes pulse {
  0%, 100% { transform: translateY(0) scale(1); }
  50% { transform: translateY(0) scale(1.1); }
}

@keyframes colorChange {
  0% { color: #fff; }
  25% { color: #ffd93d; }
  50% { color: #6bcf7f; }
  75% { color: #ff6b9d; }
  100% { color: #fff; }
}

.mountain-emoji {
  margin-top: 1rem;
  font-size: 2.5rem;
  animation: celebration 0.5s ease-in-out;
}

@keyframes celebration {
  0% { transform: scale(0); opacity: 0; }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); opacity: 1; }
}

/* 控制按钮 */
.controls {
  display: flex;
  gap: 2rem;
  justify-content: center;
  margin-bottom: 3rem;
  flex-wrap: wrap;
}

.btn {
  padding: 1.5rem 3rem;
  font-size: 1.5rem;
  font-weight: 900;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 8px 0 rgba(0, 0, 0, 0.2),
              0 12px 20px rgba(0, 0, 0, 0.15);
  position: relative;
  display: flex;
  align-items: center;
  gap: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.btn:active:not(:disabled) {
  transform: translateY(4px);
  box-shadow: 0 4px 0 rgba(0, 0, 0, 0.2),
              0 6px 10px rgba(0, 0, 0, 0.15);
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  filter: grayscale(50%);
}

.btn-icon {
  font-size: 2rem;
  animation: wiggle 1s infinite;
}

@keyframes wiggle {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-10deg); }
  75% { transform: rotate(10deg); }
}

.btn:hover:not(:disabled) .btn-icon {
  animation: wiggle 0.3s infinite;
}

.btn-start {
  background: linear-gradient(135deg, #6bcf7f 0%, #4caf50 100%);
  color: white;
}

.btn-start:hover:not(:disabled) {
  background: linear-gradient(135deg, #5bb56f 0%, #3d9640 100%);
  transform: translateY(-4px);
  box-shadow: 0 12px 0 rgba(0, 0, 0, 0.2),
              0 16px 25px rgba(76, 175, 80, 0.4);
}

.btn-stop {
  background: linear-gradient(135deg, #ff6b9d 0%, #e74c3c 100%);
  color: white;
}

.btn-stop:hover:not(:disabled) {
  background: linear-gradient(135deg, #ff5a8d 0%, #c0392b 100%);
  transform: translateY(-4px);
  box-shadow: 0 12px 0 rgba(0, 0, 0, 0.2),
              0 16px 25px rgba(231, 76, 60, 0.4);
}

.btn-confirm {
  background: linear-gradient(135deg, #ffd93d 0%, #ffaa00 100%);
  color: #333;
  animation: glow 2s infinite;
}

.btn-confirm:hover:not(:disabled) {
  background: linear-gradient(135deg, #ffc93d 0%, #ff9900 100%);
  transform: translateY(-4px);
  box-shadow: 0 12px 0 rgba(0, 0, 0, 0.2),
              0 16px 25px rgba(255, 217, 61, 0.6);
}

@keyframes glow {
  0%, 100% {
    box-shadow: 0 8px 0 rgba(0, 0, 0, 0.2),
                0 12px 20px rgba(0, 0, 0, 0.15),
                0 0 20px rgba(255, 217, 61, 0.5);
  }
  50% {
    box-shadow: 0 8px 0 rgba(0, 0, 0, 0.2),
                0 12px 20px rgba(0, 0, 0, 0.15),
                0 0 40px rgba(255, 217, 61, 0.8);
  }
}

/* 信息卡片 */
.info-card {
  background: linear-gradient(135deg, #fff9e6 0%, #ffe4f5 100%);
  padding: 2rem;
  border-radius: 25px;
  margin-bottom: 2rem;
  box-shadow: 0 10px 30px rgba(255, 107, 157, 0.2),
              0 0 0 5px #fff,
              0 0 0 8px #ffd93d;
  animation: slideIn 0.5s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.info-header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.trophy {
  font-size: 2rem;
  animation: swing 1s infinite ease-in-out;
}

.trophy:nth-child(3) {
  animation-delay: 0.5s;
}

@keyframes swing {
  0%, 100% { transform: rotate(0deg); }
  25% { transform: rotate(-15deg); }
  75% { transform: rotate(15deg); }
}

.info-title {
  font-size: 1.8rem;
  font-weight: 900;
  color: #ff6b9d;
  text-shadow: 2px 2px 0 #ffd93d;
}

.mountain-result {
  font-size: 3rem;
  font-weight: 900;
  color: #667eea;
  margin: 1rem 0;
  text-shadow: 3px 3px 0 #ffd93d;
}

.info-details {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  flex-wrap: wrap;
  margin: 1.5rem 0;
}

.info-item {
  background: white;
  padding: 0.8rem 1.5rem;
  border-radius: 20px;
  font-size: 1.2rem;
  font-weight: 700;
  color: #667eea;
  box-shadow: 0 4px 10px rgba(102, 126, 234, 0.2);
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.info-emoji {
  font-size: 1.5rem;
}

.encouragement {
  margin-top: 1.5rem;
  font-size: 1.4rem;
  font-weight: 700;
  color: #6bcf7f;
  animation: bounce 1s infinite;
}

/* 历史记录 */
.history-card {
  background: linear-gradient(135deg, #e0f7fa 0%, #e1bee7 100%);
  padding: 2rem;
  border-radius: 25px;
  box-shadow: 0 10px 30px rgba(102, 126, 234, 0.2),
              0 0 0 5px #fff,
              0 0 0 8px #6bcf7f;
  animation: slideIn 0.6s ease-out;
}

.history-title {
  font-size: 2rem;
  font-weight: 900;
  color: #667eea;
  margin: 0 0 1.5rem 0;
  text-shadow: 2px 2px 0 #ffd93d;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.history-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.history-item {
  background: white;
  padding: 1rem 1.5rem;
  margin-bottom: 0.8rem;
  border-radius: 15px;
  display: flex;
  align-items: center;
  gap: 1rem;
  font-weight: 700;
  color: #667eea;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.history-item:hover {
  transform: translateX(10px);
  box-shadow: 0 6px 15px rgba(102, 126, 234, 0.3);
}

.history-badge {
  background: linear-gradient(135deg, #ff6b9d, #ffd93d);
  color: white;
  width: 2rem;
  height: 2rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  font-weight: 900;
  flex-shrink: 0;
}

.history-date {
  color: #999;
  font-size: 0.9rem;
  flex-shrink: 0;
}

.history-name {
  flex: 1;
  text-align: left;
  font-size: 1.1rem;
}

.history-icon {
  font-size: 1.5rem;
  flex-shrink: 0;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .title {
    font-size: 2rem;
  }
  
  .mountain-name {
    font-size: 2.5rem;
  }
  
  .btn {
    padding: 1.2rem 2rem;
    font-size: 1.2rem;
  }
  
  .controls {
    gap: 1rem;
  }
  
  .mountain-result {
    font-size: 2rem;
  }
  
  .info-details {
    flex-direction: column;
    gap: 0.8rem;
  }
}
</style>

