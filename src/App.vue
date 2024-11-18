<script setup lang="ts">
import { ref } from 'vue'
import Timer from './components/Timer.vue'
import Countdown from './components/Countdown.vue'

const tabs = [
  { id: 'countdown', label: '倒计时' },
  { id: 'stopwatch', label: '秒表' },
]

const activeTab = ref('countdown')
</script>

<template>
  <div class="h-[450px] max-w-2xl mx-auto border border-gray-200 bg-white rounded-xl overflow-hidden">
    <!-- Tab Navigation -->
    <div class="border-b border-gray-200">
      <nav class="flex -mb-px" aria-label="Tabs">
        <button v-for="tab in tabs" :key="tab.id" @click="activeTab = tab.id"
          class="flex-1 px-4 py-4 text-center border-b-2 text-sm font-medium transition-colors" :class="[
            activeTab === tab.id
              ? 'border-blue-500 text-blue-600'
              : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300'
          ]">
          {{ tab.label }}
        </button>
      </nav>
    </div>

    <!-- Tab Panels -->
    <div class="relative">
      <Transition mode="out-in" enter-active-class="transition ease-out duration-200" enter-from-class="opacity-0"
        enter-to-class="opacity-100" leave-active-class="transition ease-in duration-150" leave-from-class="opacity-100"
        leave-to-class="opacity-0">
        <div v-if="activeTab === 'countdown'" key="countdown">
          <Countdown />
        </div>
        <div v-else-if="activeTab === 'stopwatch'" key="stopwatch">
          <Timer />
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
/* empty */
</style>
