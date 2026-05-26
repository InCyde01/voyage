<template>
  <div class="journey-input">
    <div class="earth-background">
      <canvas ref="canvas"></canvas>
    </div>
    
    <div class="content-overlay">
      <div class="input-card">
        <h1>What is your journey?</h1>
        
        <div class="form-section">
          <textarea 
            v-model="story" 
            placeholder="Tell us about your travels... Where did you go? What did you do? What did you experience?"
            class="story-input"
          ></textarea>
        </div>

        <div class="api-keys-section">
          <h3>API Keys (for AI Processing)</h3>
          
          <div class="api-key-input">
            <label>Gemini API Key</label>
            <input 
              v-model="apiKeys.gemini" 
              type="password" 
              placeholder="sk-..."
              @blur="validateKey('gemini')"
            />
            <span :class="['status', keyStatus.gemini]">
              {{ keyStatus.gemini === 'valid' ? '✓' : keyStatus.gemini === 'invalid' ? '✗' : '?' }}
            </span>
          </div>

          <div class="api-key-input">
            <label>Claude API Key</label>
            <input 
              v-model="apiKeys.claude" 
              type="password" 
              placeholder="sk-ant-..."
              @blur="validateKey('claude')"
            />
            <span :class="['status', keyStatus.claude]">
              {{ keyStatus.claude === 'valid' ? '✓' : keyStatus.claude === 'invalid' ? '✗' : '?' }}
            </span>
          </div>

          <div class="api-key-input">
            <label>OpenAI API Key</label>
            <input 
              v-model="apiKeys.openai" 
              type="password" 
              placeholder="sk-..."
              @blur="validateKey('openai')"
            />
            <span :class="['status', keyStatus.openai]">
              {{ keyStatus.openai === 'valid' ? '✓' : keyStatus.openai === 'invalid' ? '✗' : '?' }}
            </span>
          </div>
        </div>

        <button 
          @click="submitJourney" 
          class="submit-btn"
          :disabled="!story.trim() || !hasValidKey"
        >
          Visualize My Journey
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import * as THREE from 'three'

const emit = defineEmits(['submit'])

const canvas = ref(null)
const story = ref('')
const apiKeys = ref({
  gemini: '',
  claude: '',
  openai: ''
})
const keyStatus = ref({
  gemini: 'unchecked',
  claude: 'unchecked',
  openai: 'unchecked'
})

const hasValidKey = computed(() => {
  return Object.values(keyStatus.value).includes('valid')
})

onMounted(() => {
  if (canvas.value) {
    initializeEarth()
  }
})

const initializeEarth = () => {
  const width = window.innerWidth
  const height = window.innerHeight

  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000)
  const renderer = new THREE.WebGLRenderer({ canvas: canvas.value, alpha: true, antialias: true })
  
  renderer.setSize(width, height)
  renderer.setPixelRatio(window.devicePixelRatio)

  const geometry = new THREE.SphereGeometry(2, 64, 64)
  const texture = createEarthTexture()
  const material = new THREE.MeshPhongMaterial({ map: texture })
  const earth = new THREE.Mesh(geometry, material)
  scene.add(earth)

  const light = new THREE.DirectionalLight(0xffffff, 0.8)
  light.position.set(5, 3, 5)
  scene.add(light)

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.4)
  scene.add(ambientLight)

  camera.position.z = 5

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
  scene.add(stars)

  const animate = () => {
    requestAnimationFrame(animate)
    earth.rotation.y += 0.001
    renderer.render(scene, camera)
  }

  animate()

  window.addEventListener('resize', () => {
    const newWidth = window.innerWidth
    const newHeight = window.innerHeight
    camera.aspect = newWidth / newHeight
    camera.updateProjectionMatrix()
    renderer.setSize(newWidth, newHeight)
  })
}

const createEarthTexture = () => {
  const canvas = document.createElement('canvas')
  canvas.width = 2048
  canvas.height = 1024

  const ctx = canvas.getContext('2d')
  
  ctx.fillStyle = '#1e3a8a'
  ctx.fillRect(0, 0, canvas.width, canvas.height)

  ctx.fillStyle = '#1f2937'
  
  ctx.fillRect(50, 200, 200, 150)
  ctx.fillRect(120, 350, 100, 180)
  ctx.fillRect(400, 150, 150, 100)
  ctx.fillRect(500, 300, 200, 250)
  ctx.fillRect(750, 200, 400, 200)
  ctx.fillRect(1150, 450, 100, 80)

  ctx.strokeStyle = '#ffffff'
  ctx.lineWidth = 2
  ctx.strokeRect(50, 200, 200, 150)
  ctx.strokeRect(120, 350, 100, 180)
  ctx.strokeRect(400, 150, 150, 100)
  ctx.strokeRect(500, 300, 200, 250)
  ctx.strokeRect(750, 200, 400, 200)
  ctx.strokeRect(1150, 450, 100, 80)

  return new THREE.CanvasTexture(canvas)
}

const validateKey = async (provider) => {
  const key = apiKeys.value[provider]
  if (!key) {
    keyStatus.value[provider] = 'unchecked'
    return
  }

  keyStatus.value[provider] = 'checking'
  setTimeout(() => {
    keyStatus.value[provider] = key.length > 10 ? 'valid' : 'invalid'
  }, 500)
}

const submitJourney = () => {
  if (!story.value.trim() || !hasValidKey.value) {
    alert('Please enter a story and provide at least one valid API key')
    return
  }

  emit('submit', {
    story: story.value,
    apiKeys: apiKeys.value,
    timestamp: new Date()
  })
}
</script>

<style scoped>
.journey-input {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.earth-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

canvas {
  display: block;
  width: 100%;
  height: 100%;
}

.content-overlay {
  position: relative;
  z-index: 10;
  width: 90%;
  max-width: 600px;
}

.input-card {
  background: rgba(26, 31, 58, 0.95);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 40px;
  backdrop-filter: blur(10px);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
}

h1 {
  font-size: 32px;
  font-weight: 700;
  margin-bottom: 30px;
  text-align: center;
  background: linear-gradient(135deg, var(--primary) 0%, var(--accent) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.form-section {
  margin-bottom: 30px;
}

.story-input {
  width: 100%;
  height: 200px;
  padding: 16px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 8px;
  color: var(--text-primary);
  font-family: inherit;
  font-size: 14px;
  resize: none;
  transition: border-color 0.2s;
}

.story-input:focus {
  outline: none;
  border-color: var(--primary);
}

.story-input::placeholder {
  color: var(--text-secondary);
}

.api-keys-section {
  margin-bottom: 30px;
  padding-bottom: 30px;
  border-bottom: 1px solid var(--border);
}

.api-keys-section h3 {
  font-size: 14px;
  font-weight: 600;
  text-transform: uppercase;
  color: var(--text-secondary);
  margin-bottom: 16px;
}

.api-key-input {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.api-key-input label {
  font-size: 12px;
  font-weight: 600;
  color: var(--text-secondary);
  min-width: 120px;
}

.api-key-input input {
  flex: 1;
  padding: 8px 12px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 6px;
  color: var(--text-primary);
  font-size: 12px;
  transition: border-color 0.2s;
}

.api-key-input input:focus {
  outline: none;
  border-color: var(--primary);
}

.status {
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  border-radius: 50%;
  font-size: 12px;
}

.status.valid {
  background: var(--success);
  color: #000;
}

.status.invalid {
  background: var(--error);
  color: #fff;
}

.status.checking {
  background: var(--primary);
  color: #fff;
  animation: spin 1s linear infinite;
}

.status.unchecked {
  background: var(--surface-light);
  color: var(--text-secondary);
}

.submit-btn {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, var(--primary) 0%, var(--accent) 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.submit-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(99, 102, 241, 0.4);
}

.submit-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>