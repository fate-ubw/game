<template>
  <div class="snake-game">
    <div class="game-header">
      <h1>🐍 贪吃蛇游戏</h1>
      <div class="score-board">
        <div class="score">得分: {{ score }}</div>
        <div class="high-score">最高分: {{ highScore }}</div>
      </div>
    </div>

    <div class="game-container">
      <div v-if="!gameStarted" class="start-screen">
        <h2>准备开始</h2>
        <p>使用方向键控制蛇的移动</p>
        <button @click="startGame" class="start-button">开始游戏</button>
        <div class="difficulty-selector">
          <label>难度:</label>
          <select v-model="difficulty" @change="updateSpeed">
            <option value="easy">简单</option>
            <option value="medium">中等</option>
            <option value="hard">困难</option>
          </select>
        </div>
      </div>

      <div v-if="gameOver" class="game-over">
        <h2>游戏结束!</h2>
        <p>最终得分: {{ score }}</p>
        <button @click="restartGame" class="restart-button">重新开始</button>
      </div>

      <canvas
        ref="gameCanvas"
        :width="canvasWidth"
        :height="canvasHeight"
        @click="handleCanvasClick"
      ></canvas>
    </div>

    <div class="game-controls">
      <div class="control-buttons">
        <button @click="moveUp" class="control-btn">↑</button>
        <div class="control-row">
          <button @click="moveLeft" class="control-btn">←</button>
          <button @click="moveDown" class="control-btn">↓</button>
          <button @click="moveRight" class="control-btn">→</button>
        </div>
      </div>
      <button @click="togglePause" class="pause-button" v-if="gameStarted && !gameOver">
        {{ isPaused ? '继续' : '暂停' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

const gameCanvas = ref(null)
const canvasWidth = 600
const canvasHeight = 600
const gridSize = 20
const tileCount = canvasWidth / gridSize

const snake = ref([{ x: 10, y: 10 }])
const food = ref({ x: 15, y: 15 })
const direction = ref({ x: 0, y: 0 })
const nextDirection = ref({ x: 0, y: 0 })
const score = ref(0)
const highScore = ref(0)
const gameStarted = ref(false)
const gameOver = ref(false)
const isPaused = ref(false)
const difficulty = ref('medium')
const gameSpeed = ref(100)

let gameLoop = null
let ctx = null

const speedMap = {
  easy: 150,
  medium: 100,
  hard: 60
}

onMounted(() => {
  const canvas = gameCanvas.value
  ctx = canvas.getContext('2d')
  
  const savedHighScore = localStorage.getItem('snakeHighScore')
  if (savedHighScore) {
    highScore.value = parseInt(savedHighScore)
  }

  window.addEventListener('keydown', handleKeyPress)
  drawGame()
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyPress)
  if (gameLoop) {
    clearInterval(gameLoop)
  }
})

const updateSpeed = () => {
  gameSpeed.value = speedMap[difficulty.value]
  if (gameLoop) {
    clearInterval(gameLoop)
    gameLoop = setInterval(update, gameSpeed.value)
  }
}

const startGame = () => {
  gameStarted.value = true
  gameOver.value = false
  snake.value = [{ x: 10, y: 10 }]
  direction.value = { x: 1, y: 0 }
  nextDirection.value = { x: 1, y: 0 }
  score.value = 0
  isPaused.value = false
  generateFood()
  gameSpeed.value = speedMap[difficulty.value]
  gameLoop = setInterval(update, gameSpeed.value)
}

const restartGame = () => {
  startGame()
}

const togglePause = () => {
  isPaused.value = !isPaused.value
}

const handleKeyPress = (event) => {
  if (!gameStarted.value || gameOver.value || isPaused.value) return

  const key = event.key
  
  if (key === 'ArrowUp' && direction.value.y === 0) {
    nextDirection.value = { x: 0, y: -1 }
  } else if (key === 'ArrowDown' && direction.value.y === 0) {
    nextDirection.value = { x: 0, y: 1 }
  } else if (key === 'ArrowLeft' && direction.value.x === 0) {
    nextDirection.value = { x: -1, y: 0 }
  } else if (key === 'ArrowRight' && direction.value.x === 0) {
    nextDirection.value = { x: 1, y: 0 }
  } else if (key === ' ') {
    event.preventDefault()
    togglePause()
  }
}

const moveUp = () => {
  if (direction.value.y === 0) {
    nextDirection.value = { x: 0, y: -1 }
  }
}

const moveDown = () => {
  if (direction.value.y === 0) {
    nextDirection.value = { x: 0, y: 1 }
  }
}

const moveLeft = () => {
  if (direction.value.x === 0) {
    nextDirection.value = { x: -1, y: 0 }
  }
}

const moveRight = () => {
  if (direction.value.x === 0) {
    nextDirection.value = { x: 1, y: 0 }
  }
}

const handleCanvasClick = () => {
  if (!gameStarted.value) {
    startGame()
  }
}

const update = () => {
  if (isPaused.value) return

  direction.value = nextDirection.value

  const head = { ...snake.value[0] }
  head.x += direction.value.x
  head.y += direction.value.y

  if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
    endGame()
    return
  }

  for (let i = 0; i < snake.value.length; i++) {
    if (head.x === snake.value[i].x && head.y === snake.value[i].y) {
      endGame()
      return
    }
  }

  snake.value.unshift(head)

  if (head.x === food.value.x && head.y === food.value.y) {
    score.value += 10
    if (score.value > highScore.value) {
      highScore.value = score.value
      localStorage.setItem('snakeHighScore', highScore.value.toString())
    }
    generateFood()
  } else {
    snake.value.pop()
  }

  drawGame()
}

const generateFood = () => {
  let newFood
  let isOnSnake
  
  do {
    isOnSnake = false
    newFood = {
      x: Math.floor(Math.random() * tileCount),
      y: Math.floor(Math.random() * tileCount)
    }
    
    for (let segment of snake.value) {
      if (segment.x === newFood.x && segment.y === newFood.y) {
        isOnSnake = true
        break
      }
    }
  } while (isOnSnake)
  
  food.value = newFood
}

const drawGame = () => {
  if (!ctx) return

  ctx.fillStyle = '#1a1a2e'
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)

  ctx.strokeStyle = '#16213e'
  ctx.lineWidth = 1
  for (let i = 0; i <= tileCount; i++) {
    ctx.beginPath()
    ctx.moveTo(i * gridSize, 0)
    ctx.lineTo(i * gridSize, canvasHeight)
    ctx.stroke()
    
    ctx.beginPath()
    ctx.moveTo(0, i * gridSize)
    ctx.lineTo(canvasWidth, i * gridSize)
    ctx.stroke()
  }

  snake.value.forEach((segment, index) => {
    if (index === 0) {
      ctx.fillStyle = '#4ecca3'
      ctx.shadowBlur = 10
      ctx.shadowColor = '#4ecca3'
    } else {
      ctx.fillStyle = '#45b393'
      ctx.shadowBlur = 5
      ctx.shadowColor = '#45b393'
    }
    
    ctx.fillRect(
      segment.x * gridSize + 1,
      segment.y * gridSize + 1,
      gridSize - 2,
      gridSize - 2
    )
    ctx.shadowBlur = 0
  })

  ctx.fillStyle = '#ff6b6b'
  ctx.shadowBlur = 15
  ctx.shadowColor = '#ff6b6b'
  ctx.beginPath()
  ctx.arc(
    food.value.x * gridSize + gridSize / 2,
    food.value.y * gridSize + gridSize / 2,
    gridSize / 2 - 2,
    0,
    Math.PI * 2
  )
  ctx.fill()
  ctx.shadowBlur = 0
}

const endGame = () => {
  gameOver.value = true
  clearInterval(gameLoop)
}
</script>

<style scoped>
.snake-game {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.game-header {
  text-align: center;
  margin-bottom: 20px;
  color: white;
}

.game-header h1 {
  font-size: 3em;
  margin: 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.score-board {
  display: flex;
  gap: 30px;
  justify-content: center;
  margin-top: 10px;
  font-size: 1.5em;
  font-weight: bold;
}

.game-container {
  position: relative;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

canvas {
  display: block;
  background: #1a1a2e;
}

.start-screen,
.game-over {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(26, 26, 46, 0.95);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  z-index: 10;
}

.start-screen h2,
.game-over h2 {
  font-size: 3em;
  margin: 0 0 20px 0;
  color: #4ecca3;
}

.start-screen p,
.game-over p {
  font-size: 1.2em;
  margin: 10px 0;
  color: #ddd;
}

.start-button,
.restart-button,
.pause-button {
  margin-top: 20px;
  padding: 15px 40px;
  font-size: 1.2em;
  font-weight: bold;
  color: white;
  background: linear-gradient(135deg, #4ecca3 0%, #45b393 100%);
  border: none;
  border-radius: 50px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 4px 15px rgba(78, 204, 163, 0.4);
}

.start-button:hover,
.restart-button:hover,
.pause-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(78, 204, 163, 0.6);
}

.start-button:active,
.restart-button:active,
.pause-button:active {
  transform: translateY(0);
}

.difficulty-selector {
  margin-top: 20px;
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.1em;
}

.difficulty-selector label {
  color: #ddd;
}

.difficulty-selector select {
  padding: 8px 15px;
  font-size: 1em;
  border: 2px solid #4ecca3;
  border-radius: 5px;
  background: #16213e;
  color: white;
  cursor: pointer;
  outline: none;
}

.difficulty-selector select:hover {
  background: #1a2847;
}

.game-controls {
  margin-top: 30px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.control-buttons {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.control-row {
  display: flex;
  gap: 10px;
}

.control-btn {
  width: 60px;
  height: 60px;
  font-size: 1.5em;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  color: white;
  cursor: pointer;
  transition: all 0.2s;
  backdrop-filter: blur(10px);
}

.control-btn:hover {
  background: rgba(255, 255, 255, 0.2);
  border-color: rgba(255, 255, 255, 0.5);
  transform: scale(1.05);
}

.control-btn:active {
  transform: scale(0.95);
}

.pause-button {
  padding: 10px 30px;
  font-size: 1em;
}

@media (max-width: 768px) {
  .game-header h1 {
    font-size: 2em;
  }

  .score-board {
    font-size: 1.2em;
    gap: 20px;
  }

  canvas {
    width: 100%;
    height: auto;
  }

  .control-btn {
    width: 50px;
    height: 50px;
    font-size: 1.2em;
  }
}
</style>
