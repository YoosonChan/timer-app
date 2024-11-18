<script setup lang="ts">
import { ref, onUnmounted, computed, onMounted } from 'vue'
import {
  PlayIcon,
  PauseIcon,
  ArrowPathIcon,
  PlusIcon,
  MinusIcon,
} from '@heroicons/vue/24/solid'

const ADJUST_SECONDS = 30
const presetTimes = [
  { label: '5分钟', seconds: 5 * 60 },
  { label: '10分钟', seconds: 10 * 60 },
  { label: '15分钟', seconds: 15 * 60 },
]

const initialTime = ref(presetTimes[0].seconds)
const currentTime = ref(initialTime.value)
const isRunning = ref(false)
const isOvertime = computed(() => currentTime.value < 0)

// 计算超时轮数（每30秒一轮）
const overtimeRounds = ref(0)

// 计数点相关计算
const MAX_POINTS_PER_ROW = 3
const MAX_ROWS = 3
const MAX_POINTS = MAX_POINTS_PER_ROW * MAX_ROWS
const POINT_SPACING = 20
const ROW_SPACING = 20
const START_X = 10
const START_Y = 24

const getPointX = (index: number) => {
  const position = (index - 1) % MAX_POINTS
  const col = position % MAX_POINTS_PER_ROW
  const rowStartX = START_X + (250 - (MAX_POINTS_PER_ROW * POINT_SPACING)) / 2
  return rowStartX + col * POINT_SPACING
}

const getPointY = (index: number) => {
  const position = (index - 1) % MAX_POINTS
  const row = Math.floor(position / MAX_POINTS_PER_ROW)
  return START_Y + row * ROW_SPACING
}

const getPointColor = (index: number) => {
  const cyclePosition = Math.floor((index - 1) / MAX_POINTS)
  return cyclePosition === 0 ? '#6b7280' : '#ef4444'
}

// 计算进度（0-1）
const progress = computed(() => {
  if (!isRunning.value) return 1
  if (!isOvertime.value) {
    return currentTime.value / initialTime.value
  } else {
    // 超时后，每30秒一圈
    const overtimeSeconds = Math.abs(currentTime.value)
    const currentRoundSeconds = overtimeSeconds % ADJUST_SECONDS
    return 1 - (currentRoundSeconds / ADJUST_SECONDS)
  }
})

// 计算SVG路径的描述
const progressPath = computed(() => {
  const radius = 120 // SVG圆的半径
  return `M 125,5 
          A ${radius} ${radius} 0 0 1 245,125 
          A ${radius} ${radius} 0 0 1 125,245 
          A ${radius} ${radius} 0 0 1 5,125 
          A ${radius} ${radius} 0 0 1 125,5`
})

let countdownInterval: number | null = null
let startTime: number | null = null

const formatTime = (timeInSeconds: number): string => {
  const absTime = Math.abs(timeInSeconds)
  const minutes = Math.floor(absTime / 60)
  const seconds = absTime % 60
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
}

const startTimer = () => {
  if (!isRunning.value) {
    isRunning.value = true
    startTime = Date.now()
    countdownInterval = setInterval(updateTimer, 100)
  }
}

const pauseTimer = () => {
  isRunning.value = false
  if (countdownInterval) {
    clearInterval(countdownInterval)
    countdownInterval = null
  }
}

const toggleTimer = () => {
  if (isRunning.value) {
    pauseTimer()
  } else {
    startTimer()
  }
}

const resetTimer = () => {
  pauseTimer()
  overtimeRounds.value = 0
  // 重置为上次使用的时间
  const savedTime = localStorage.getItem('lastUsedTime')
  if (savedTime) {
    const time = parseInt(savedTime)
    currentTime.value = time
    initialTime.value = time
  } else {
    currentTime.value = presetTimes[0].seconds
    initialTime.value = presetTimes[0].seconds
  }
}

const setTime = (seconds: number) => {
  pauseTimer()
  initialTime.value = seconds
  currentTime.value = seconds
  // 保存到本地存储
  localStorage.setItem('lastUsedTime', seconds.toString())
}

const adjustTime = (seconds: number) => {
  if (!isRunning.value) {
    const newTime = Math.max(0, currentTime.value + seconds)
    currentTime.value = newTime
    initialTime.value = newTime
    // 保存到本地存储
    localStorage.setItem('lastUsedTime', newTime.toString())
  }
}

const updateTimer = () => {
  if (startTime && isRunning.value) {
    const now = Date.now()
    const elapsed = Math.floor((now - startTime) / 1000)
    const newTime = initialTime.value - elapsed
    currentTime.value = newTime // 允许负值以显示超时时间

    if (newTime <= 0) {
      const overtimeElapsed = Math.abs(newTime)
      const newOvertimeRounds = Math.floor(overtimeElapsed / ADJUST_SECONDS) + 1

      // 检查是否超过最大计数点数（包括红色点）
      if (newOvertimeRounds > MAX_POINTS * 2) {
        pauseTimer()
        resetTimer()
        return
      }

      overtimeRounds.value = newOvertimeRounds
    }
  }
}

// 键盘事件处理
const handleKeydown = (event: KeyboardEvent) => {
  // 防止在输入框中触发
  if (event.target instanceof HTMLInputElement || event.target instanceof HTMLTextAreaElement) {
    return
  }

  switch (event.key) {
    case ' ': // 空格键
      event.preventDefault() // 防止页面滚动
      if (currentTime.value !== 0) {
        toggleTimer()
      }
      break
    case 'Enter': // 回车键
      event.preventDefault()
      if (!(!isRunning.value && currentTime.value === initialTime.value)) {
        resetTimer()
      }
      break
    case '+': // 加号键
    case '=': // 等号键（未按Shift的加号）
      event.preventDefault()
      if (!isRunning.value) {
        adjustTime(ADJUST_SECONDS)
      }
      break
    case '-': // 减号键
      event.preventDefault()
      if (!isRunning.value && currentTime.value !== 0) {
        adjustTime(-ADJUST_SECONDS)
      }
      break
  }
}

onMounted(() => {
  const savedTime = localStorage.getItem('lastUsedTime')
  if (savedTime) {
    const time = parseInt(savedTime)
    currentTime.value = time
    initialTime.value = time
  }
  // 添加键盘事件监听
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  if (countdownInterval) {
    clearInterval(countdownInterval)
  }
  // 移除键盘事件监听
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <div class="flex flex-col items-center justify-center p-8 space-y-6">
    <div class="flex space-x-4">
      <button v-for="preset in presetTimes" :key="preset.seconds" @click="setTime(preset.seconds)"
        class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-md hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed"
        :class="{ 'ring-2 ring-blue-500 ring-offset-2': currentTime === preset.seconds }" :disabled="isRunning">
        {{ preset.label }}
      </button>
    </div>
    <div class="flex items-center space-x-4">
      <button @click="adjustTime(-ADJUST_SECONDS)"
        class="p-2 text-gray-600 hover:text-gray-900 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed rounded-lg transition-colors"
        :disabled="isRunning || currentTime === 0" title="减少30秒">
        <MinusIcon class="w-6 h-6" />
      </button>
      <div class="relative w-48 h-48 flex items-center justify-center">
        <!-- 进度边框和计数点 -->
        <svg class="absolute inset-0 w-full h-full transition-all duration-300" viewBox="0 0 250 250">
          <!-- 主进度圆环 -->
          <g class="-rotate-90 origin-center">
            <path stroke-linecap="round" :d="progressPath" fill="none" :stroke="isOvertime ? '#ef4444' : '#3b82f6'"
              stroke-width="4" class="transition-all duration-300" :style="{
                strokeDasharray: 2 * Math.PI * 120,
                strokeDashoffset: 2 * Math.PI * 120 * (1 - progress),
              }" />
          </g>
          <!-- 超时计数点 -->
          <g v-if="overtimeRounds > 0">
            <circle v-for="i in overtimeRounds" :key="i" :cx="getPointX(i)" :cy="getPointY(i)" r="4"
              :fill="getPointColor(i)" class="transition-all duration-300" />
          </g>
        </svg>
        <!-- 时间显示 -->
        <div class="text-5xl font-mono font-bold text-center text-gray-800">
          {{ formatTime(currentTime) }}
        </div>
      </div>
      <button @click="adjustTime(ADJUST_SECONDS)"
        class="p-2 text-gray-600 hover:text-gray-900 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:opacity-50 disabled:cursor-not-allowed rounded-lg transition-colors"
        :disabled="isRunning" title="增加30秒">
        <PlusIcon class="w-6 h-6" />
      </button>
    </div>
    <div class="flex space-x-4">
      <button @click="toggleTimer"
        class="p-3 text-white bg-blue-600 rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
        :title="isRunning ? '暂停' : '开始'" :disabled="currentTime === 0">
        <PlayIcon v-if="!isRunning" class="w-6 h-6" />
        <PauseIcon v-else class="w-6 h-6" />
      </button>
      <button @click="resetTimer"
        class="p-3 text-white bg-red-600 rounded-lg hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
        title="重置" :disabled="!isRunning && currentTime === initialTime">
        <ArrowPathIcon class="w-6 h-6" />
      </button>
    </div>
  </div>
</template>