<template>
  <div class="demo-container">
    <h1>🎓 状态管理演示</h1>
    
    <div class="control-panel">
      <h2>控制面板（手动改变状态）</h2>
      <div class="controls">
        <label>
          <input type="checkbox" v-model="gameStarted" />
          gameStarted = {{ gameStarted }}
        </label>
        <label>
          <input type="checkbox" v-model="gameOver" />
          gameOver = {{ gameOver }}
        </label>
        <label>
          <input type="checkbox" v-model="isPaused" />
          isPaused = {{ isPaused }}
        </label>
      </div>
      
      <div class="action-buttons">
        <button @click="simulateStart">模拟：开始游戏</button>
        <button @click="simulateEnd">模拟：游戏结束</button>
        <button @click="simulateRestart">模拟：重新开始</button>
        <button @click="simulatePause">模拟：暂停/继续</button>
      </div>
    </div>

    <div class="preview-area">
      <h2>当前显示的内容</h2>
      
      <div class="game-preview">
        <!-- 开始页面 -->
        <div v-if="!gameStarted" class="page start-page">
          <h3>🎮 开始页面</h3>
          <p>条件: !gameStarted ({{ !gameStarted }})</p>
          <div class="code-block">
            v-if="!gameStarted"
          </div>
        </div>

        <!-- 暂停页面 -->
        <div v-if="isPaused && gameStarted && !gameOver" class="page pause-page">
          <h3>⏸️ 暂停页面</h3>
          <p>条件: isPaused && gameStarted && !gameOver</p>
          <div class="code-block">
            v-if="isPaused && gameStarted && !gameOver"
          </div>
        </div>

        <!-- 游戏结束页面 -->
        <div v-if="gameOver" class="page end-page">
          <h3>🏁 结束页面</h3>
          <p>条件: gameOver ({{ gameOver }})</p>
          <div class="code-block">
            v-if="gameOver"
          </div>
        </div>

        <!-- 游戏画布（一直存在） -->
        <div class="canvas-placeholder">
          <h3>🎨 游戏画布</h3>
          <p>无条件，一直存在</p>
          <div class="code-block">
            &lt;canvas&gt; (无 v-if)
          </div>
          <div v-if="gameStarted && !gameOver && !isPaused" class="status-active">
            ✅ 游戏正在运行
          </div>
          <div v-else class="status-inactive">
            ⏹️ 游戏未运行（被遮挡或暂停）
          </div>
        </div>
      </div>

      <div class="explanation">
        <h3>📝 当前状态说明</h3>
        <div v-if="!gameStarted">
          <strong>状态：等待开始</strong>
          <p>显示开始页面，画布在后面但被遮挡</p>
        </div>
        <div v-else-if="gameOver">
          <strong>状态：游戏结束</strong>
          <p>显示结束页面，画布在后面但被遮挡</p>
        </div>
        <div v-else-if="isPaused">
          <strong>状态：游戏暂停</strong>
          <p>显示暂停页面，游戏循环已停止</p>
        </div>
        <div v-else>
          <strong>状态：游戏进行中</strong>
          <p>只显示画布，游戏循环正在运行</p>
        </div>
      </div>
    </div>

    <div class="code-explanation">
      <h2>💻 代码工作原理</h2>
      <div class="code-section">
        <h3>1. 定义状态变量</h3>
        <pre><code>const gameStarted = ref(false)
const gameOver = ref(false)
const isPaused = ref(false)</code></pre>
      </div>

      <div class="code-section">
        <h3>2. 在 template 中使用条件</h3>
        <pre><code>&lt;div v-if="!gameStarted"&gt;开始页面&lt;/div&gt;
&lt;div v-if="gameOver"&gt;结束页面&lt;/div&gt;
&lt;canvas&gt;画布&lt;/canvas&gt;</code></pre>
      </div>

      <div class="code-section">
        <h3>3. 改变状态触发重新渲染</h3>
        <pre><code>const startGame = () => {
  gameStarted.value = true  // ← 改变状态
  // Vue 自动检测到变化，重新渲染
}</code></pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const gameStarted = ref(false)
const gameOver = ref(false)
const isPaused = ref(false)

const simulateStart = () => {
  gameStarted.value = true
  gameOver.value = false
  isPaused.value = false
}

const simulateEnd = () => {
  gameOver.value = true
}

const simulateRestart = () => {
  gameStarted.value = true
  gameOver.value = false
  isPaused.value = false
}

const simulatePause = () => {
  if (gameStarted.value && !gameOver.value) {
    isPaused.value = !isPaused.value
  }
}
</script>

<style scoped>
.demo-container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
}

h2 {
  color: #555;
  border-bottom: 2px solid #4ecca3;
  padding-bottom: 10px;
  margin-top: 30px;
}

.control-panel {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 30px;
}

.controls {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.controls label {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.1em;
  font-family: 'Courier New', monospace;
  padding: 10px;
  background: white;
  border-radius: 5px;
}

.controls input[type="checkbox"] {
  width: 20px;
  height: 20px;
  cursor: pointer;
}

.action-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.action-buttons button {
  padding: 10px 20px;
  font-size: 1em;
  background: #4ecca3;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.2s;
}

.action-buttons button:hover {
  background: #45b393;
  transform: translateY(-2px);
}

.preview-area {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.game-preview {
  position: relative;
  min-height: 400px;
  background: #1a1a2e;
  border-radius: 10px;
  padding: 20px;
  margin: 20px 0;
}

.page {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  padding: 20px;
  animation: fadeIn 0.3s;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.start-page {
  background: rgba(78, 204, 163, 0.9);
  z-index: 3;
}

.pause-page {
  background: rgba(255, 193, 7, 0.9);
  z-index: 2;
}

.end-page {
  background: rgba(255, 107, 107, 0.9);
  z-index: 3;
}

.page h3 {
  color: white;
  font-size: 2em;
  margin: 0 0 10px 0;
}

.page p {
  color: white;
  font-size: 1.2em;
  margin: 10px 0;
}

.canvas-placeholder {
  background: #16213e;
  border-radius: 10px;
  padding: 40px;
  text-align: center;
  color: white;
}

.canvas-placeholder h3 {
  margin: 0 0 10px 0;
}

.code-block {
  background: rgba(0, 0, 0, 0.3);
  padding: 10px 20px;
  border-radius: 5px;
  font-family: 'Courier New', monospace;
  color: #ffd700;
  margin-top: 10px;
}

.status-active {
  margin-top: 20px;
  padding: 10px;
  background: rgba(78, 204, 163, 0.3);
  border: 2px solid #4ecca3;
  border-radius: 5px;
  font-weight: bold;
}

.status-inactive {
  margin-top: 20px;
  padding: 10px;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 5px;
}

.explanation {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
  margin-top: 20px;
  border-left: 4px solid #4ecca3;
}

.explanation strong {
  color: #4ecca3;
  font-size: 1.3em;
}

.explanation p {
  margin: 10px 0 0 0;
  color: #666;
}

.code-explanation {
  margin-top: 30px;
}

.code-section {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 10px;
  margin-bottom: 20px;
}

.code-section h3 {
  color: #333;
  margin-top: 0;
}

.code-section pre {
  background: #1a1a2e;
  color: #4ecca3;
  padding: 15px;
  border-radius: 5px;
  overflow-x: auto;
  margin: 10px 0 0 0;
}

.code-section code {
  font-family: 'Courier New', monospace;
  font-size: 0.95em;
}
</style>
