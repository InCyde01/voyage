<template>
  <div class="app-container">
    <JourneyInput v-if="currentScreen === 'input'" @submit="handleJourneySubmit" />
    <LoadingAnimation v-else-if="currentScreen === 'loading'" />
    <GlobeViewer v-else-if="currentScreen === 'globe'" :journey-data="journeyData" />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import JourneyInput from './components/JourneyInput.vue'
import LoadingAnimation from './components/LoadingAnimation.vue'
import GlobeViewer from './components/GlobeViewer.vue'

const currentScreen = ref('input')
const journeyData = ref(null)

const handleJourneySubmit = async (data) => {
  currentScreen.value = 'loading'
  // Simulate processing time
  setTimeout(() => {
    journeyData.value = data
    currentScreen.value = 'globe'
  }, 3000)
}
</script>

<style scoped>
.app-container {
  width: 100%;
  height: 100%;
  background: var(--background);
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>