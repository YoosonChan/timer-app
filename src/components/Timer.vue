<script setup lang="ts">
import { ref, onUnmounted } from 'vue'
import {
  PlayIcon,
  PauseIcon,
  ArrowPathIcon,
  ClockIcon,
} from '@heroicons/vue/24/solid'

const time = ref(0)
const milliseconds = ref(0)
const isRunning = ref(false)
const showMilliseconds = ref(false)
let timerInterval: number | null = null

const formatTime = (timeInSeconds: number, ms: number): string => {
  const hours = Math.floor(timeInSeconds / 3600)
  const minutes = Math.floor((timeInSeconds % 3600) / 60)
  const seconds = timeInSeconds % 60
  const baseTime = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
  return baseTime
}

const startTimer = () => {
  if (!isRunning.value) {
    isRunning.value = true
    const startTime = Date.now() - (time.value * 1000 + milliseconds.value * 10)
    timerInterval = setInterval(() => {
      const elapsed = Date.now() - startTime
      time.value = Math.floor(elapsed / 1000)
      milliseconds.value = Math.floor((elapsed % 1000) / 10)
    }, 10)
  }
}

const pauseTimer = () => {
  isRunning.value = false
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
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
  time.value = 0
  milliseconds.value = 0
}

const toggleMilliseconds = () => {
  showMilliseconds.value = !showMilliseconds.value
}

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval)
  }
})
</script>

<template>
  <div class="flex flex-col items-center justify-center p-8 space-y-6">
    <div class="text-6xl font-mono font-bold text-gray-800 flex items-baseline">
      {{ formatTime(time, milliseconds) }}
      <span v-if="showMilliseconds" class="text-4xl ml-1 text-gray-600">
        .{{ milliseconds.toString().padStart(2, '0') }}
      </span>
    </div>
    <div class="flex space-x-4">
      <button
        @click="toggleTimer"
        class="p-3 text-white bg-blue-600 rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-colors"
        :title="isRunning ? '暂停' : '开始'"
      >
        <PlayIcon v-if="!isRunning" class="w-6 h-6" />
        <PauseIcon v-else class="w-6 h-6" />
      </button>
      <button
        @click="toggleMilliseconds"
        class="p-3 text-gray-600 border border-gray-300 rounded-lg hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-gray-300 focus:ring-offset-2 transition-colors group"
        :title="showMilliseconds ? '隐藏毫秒' : '显示毫秒'"
      >
        <ClockIcon class="w-6 h-6 group-hover:text-gray-900 transition-colors" />
      </button>
      <button
        @click="resetTimer"
        class="p-3 text-white bg-red-600 rounded-lg hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 transition-colors"
        title="重置"
      >
        <ArrowPathIcon class="w-6 h-6" />
      </button>
    </div>
  </div>
</template>
