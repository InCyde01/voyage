<template>
  <div class="loading-container">
    <div class="loading-content">
      <div class="thinking-animation">
        <div class="loading-dot"></div>
        <div class="loading-dot"></div>
        <div class="loading-dot"></div>
      </div>
      <h2>Processing Your Journey...</h2>
      <p>Extracting locations and generating your story visualization</p>
    </div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.loading-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--background);
}

.loading-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px;
}

.thinking-animation {
  display: flex;
  gap: 12px;
  align-items: center;
}

.loading-dot {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--primary) 0%, var(--accent) 100%);
  animation: thinking-pulse 1.4s infinite;
}

.loading-dot:nth-child(2) {
  animation-delay: 0.2s;
}

.loading-dot:nth-child(3) {
  animation-delay: 0.4s;
}

h2 {
  font-size: 28px;
  font-weight: 700;
  background: linear-gradient(135deg, var(--primary) 0%, var(--accent) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

p {
  color: var(--text-secondary);
  font-size: 14px;
}

@keyframes thinking-pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.3);
    opacity: 0.6;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style>