<template>
  <div>
    <div class="navigation">
      <button 
        @click="currentView = 'game'" 
        :class="{ active: currentView === 'game' }"
      >
        🎮 玩游戏
      </button>
      <button 
        @click="currentView = 'demo'" 
        :class="{ active: currentView === 'demo' }"
      >
        🎓 学习状态管理
      </button>
    </div>

    <SnakeGame v-if="currentView === 'game'" />
    <StateDemo v-if="currentView === 'demo'" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import SnakeGame from './SnakeGame.vue'
import StateDemo from './StateDemo.vue'

const currentView = ref('game')
</script>

<style scoped>
.navigation {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
  z-index: 1000;
  background: rgba(255, 255, 255, 0.95);
  padding: 10px;
  border-radius: 50px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
}

.navigation button {
  padding: 10px 20px;
  font-size: 1em;
  border: 2px solid transparent;
  border-radius: 25px;
  background: transparent;
  cursor: pointer;
  transition: all 0.3s;
  font-weight: 500;
  color: #333;
}

.navigation button:hover {
  background: rgba(78, 204, 163, 0.1);
  border-color: #4ecca3;
}

.navigation button.active {
  background: #4ecca3;
  color: white;
  border-color: #4ecca3;
}
</style>
