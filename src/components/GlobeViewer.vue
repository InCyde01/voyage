<template>
  <div class="globe-viewer">
    <div class="canvas-container" ref="containerRef"></div>
    
    <div class="controls">
      <div class="playback-controls">
        <button @click="goBack" class="control-btn" title="Go Back">⟨</button>
        <button @click="togglePlay" class="control-btn" :class="{ playing: isPlaying }">
          {{ isPlaying ? '⏸' : '▶' }}
        </button>
        <button @click="goForward" class="control-btn" title="Go Forward">⟩</button>
      </div>

      <div class="timeline-scrubber">
        <input 
          v-model.number="currentStep" 
          type="range" 
          :min="0" 
          :max="totalSteps - 1" 
          class="scrubber"
        />
        <span class="step-counter">{{ currentStep + 1 }} / {{ totalSteps }}</span>
      </div>

      <div class="speed-control">
        <label>Speed:</label>
        <input 
          v-model.number="speed" 
          type="range" 
          min="0.5" 
          max="2" 
          step="0.25" 
          class="speed-slider"
        />
        <span>{{ speed }}x</span>
      </div>
    </div>

    <div class="location-info" v-if="currentLocation">
      <h3>{{ currentLocation.name }}</h3>
      <p>{{ currentLocation.description }}</p>
    </div>

    <button @click="zoomOut" class="zoom-btn">Show All Locations</button>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'

const props = defineProps({
  journeyData: {
    type: Object,
    required: true
  }
})

const containerRef = ref(null)
const scene = ref(null)
const camera = ref(null)
const renderer = ref(null)
const earth = ref(null)

const isPlaying = ref(false)
const currentStep = ref(0)
const totalSteps = ref(5)
const speed = ref(1)

const currentLocation = reactive({
  name: 'Starting Point',
  description: 'Your journey begins here'
})

let animationFrameId = null

onMounted(() => {
  initializeGlobe()
  startAnimation()
})

onUnmounted(() => {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId)
  }
  if (renderer.value) {
    renderer.value.dispose()
  }
})

const initializeGlobe = () => {
  const width = window.innerWidth
  const height = window.innerHeight

  scene.value = new THREE.Scene()
  camera.value = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000)
  renderer.value = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  
  renderer.value.setSize(width, height)
  renderer.value.setPixelRatio(window.devicePixelRatio)
  containerRef.value.appendChild(renderer.value.domElement)

  const geometry = new THREE.SphereGeometry(2, 64, 64)
  const texture = createDetailedEarthTexture()
  const material = new THREE.MeshPhongMaterial({ map: texture })
  earth.value = new THREE.Mesh(geometry, material)
  scene.value.add(earth.value)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(5, 3, 5)
  scene.value.add(directionalLight)

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.value.add(ambientLight)

  const starsGeometry = new THREE.BufferGeometry()
  const starsMaterial = new THREE.PointsMaterial({ color: 0xffffff, size: 0.02 })
  const starsVertices = []
  for (let i = 0; i < 1000; i++) {
    const x = (Math.random() - 0.5) * 200
    const y = (Math.random() - 0.5) * 200
    const z = (Math.random() - 0.5) * 200
    starsVertices.push(x, y, z)
  }
  starsGeometry.setAttribute('position', new THREE.BufferAttribute(new Float32Array(starsVertices), 3))
  const stars = new THREE.Points(starsGeometry, starsMaterial)
  scene.value.add(stars)

  camera.value.position.z = 5

  const animate = () => {
    animationFrameId = requestAnimationFrame(animate)
    
    if (earth.value) {
      earth.value.rotation.y += 0.0002
    }

    renderer.value.render(scene.value, camera.value)
  }

  animate()

  window.addEventListener('resize', () => {
    const newWidth = window.innerWidth
    const newHeight = window.innerHeight
    camera.value.aspect = newWidth / newHeight
    camera.value.updateProjectionMatrix()
    renderer.value.setSize(newWidth, newHeight)
  })
}

const createDetailedEarthTexture = () => {
  const canvas = document.createElement('canvas')
  canvas.width = 2048
  canvas.height = 1024

  const ctx = canvas.getContext('2d')
  
  const gradient = ctx.createLinearGradient(0, 0, 0, canvas.height)
  gradient.addColorStop(0, '#1a4d7a')
  gradient.addColorStop(1, '#0d1f3c')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  ctx.fillStyle = '#2d3d2d'
  
  ctx.beginPath()
  ctx.ellipse(150, 250, 120, 100, 0.2, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.beginPath()
  ctx.ellipse(200, 450, 70, 120, 0.1, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.beginPath()
  ctx.ellipse(480, 200, 100, 80, 0, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.beginPath()
  ctx.ellipse(600, 400, 120, 150, 0, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.beginPath()
  ctx.ellipse(900, 250, 200, 150, 0, 0, Math.PI * 2)
  ctx.fill()
  
  ctx.beginPath()
  ctx.ellipse(1200, 480, 60, 50, 0, 0, Math.PI * 2)
  ctx.fill()

  ctx.strokeStyle = '#ffffff'
  ctx.lineWidth = 2
  ctx.stroke()

  return new THREE.CanvasTexture(canvas)
}

const startAnimation = () => {
  const locations = [
    { name: 'Start: New York', description: 'The Big Apple - where the journey begins' },
    { name: 'London', description: 'Historic capital of the United Kingdom' },
    { name: 'Paris', description: 'The City of Light and romance' },
    { name: 'Tokyo', description: 'Vibrant metropolis of Japan' },
    { name: 'Sydney', description: 'Beautiful harbor city in Australia' }
  ]
  
  totalSteps.value = locations.length
}

const togglePlay = () => {
  isPlaying.value = !isPlaying.value
  if (isPlaying.value) {
    playAnimation()
  }
}

const playAnimation = () => {
  if (!isPlaying.value) return
  
  currentStep.value += 1
  if (currentStep.value >= totalSteps.value) {
    isPlaying.value = false
    return
  }

  setTimeout(() => {
    playAnimation()
  }, 2000 / speed.value)
}

const goBack = () => {
  if (currentStep.value > 0) {
    currentStep.value--
    isPlaying.value = false
  }
}

const goForward = () => {
  if (currentStep.value < totalSteps.value - 1) {
    currentStep.value++
    isPlaying.value = false
  }
}

const zoomOut = () => {
  alert('Showing all locations - animation zoom out would happen here')
}
</script>

<style scoped>
.globe-viewer {
  position: relative;
  width: 100%;
  height: 100%;
}

.canvas-container {
  width: 100%;
  height: 100%;
}

.controls {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(26, 31, 58, 0.9);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  backdrop-filter: blur(10px);
  z-index: 100;
  min-width: 400px;
}

.playback-controls {
  display: flex;
  gap: 12px;
  justify-content: center;
}

.control-btn {
  width: 44px;
  height: 44px;
  background: var(--primary);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.control-btn:hover {
  background: var(--primary-dark);
  transform: scale(1.05);
}

.control-btn.playing {
  background: var(--accent);
}

.timeline-scrubber {
  display: flex;
  gap: 12px;
  align-items: center;
}

.scrubber {
  flex: 1;
  height: 6px;
  border-radius: 3px;
  background: var(--surface);
  outline: none;
  -webkit-appearance: none;
  appearance: none;
  cursor: pointer;
}

.scrubber::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: var(--primary);
  cursor: pointer;
}

.scrubber::-moz-range-thumb {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: var(--primary);
  cursor: pointer;
  border: none;
}

.step-counter {
  font-size: 12px;
  color: var(--text-secondary);
  min-width: 50px;
  text-align: right;
}

.speed-control {
  display: flex;
  gap: 12px;
  align-items: center;
  justify-content: center;
}

.speed-control label {
  font-size: 12px;
  color: var(--text-secondary);
}

.speed-slider {
  width: 120px;
  height: 4px;
  border-radius: 2px;
  background: var(--surface);
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.speed-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: var(--primary);
  cursor: pointer;
}

.speed-slider::-moz-range-thumb {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: var(--primary);
  cursor: pointer;
  border: none;
}

.speed-control span {
  font-size: 12px;
  color: var(--text-secondary);
  min-width: 30px;
}

.location-info {
  position: absolute;
  top: 40px;
  left: 40px;
  background: rgba(26, 31, 58, 0.9);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
  max-width: 400px;
  backdrop-filter: blur(10px);
  z-index: 100;
}

.location-info h3 {
  font-size: 18px;
  margin-bottom: 8px;
  color: var(--primary);
}

.location-info p {
  font-size: 14px;
  color: var(--text-secondary);
}

.zoom-btn {
  position: absolute;
  top: 40px;
  right: 40px;
  padding: 12px 24px;
  background: linear-gradient(135deg, var(--primary) 0%, var(--accent) 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  z-index: 100;
}

.zoom-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(99, 102, 241, 0.4);
}
</style>