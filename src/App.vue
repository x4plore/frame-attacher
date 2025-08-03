<template>
  <div :class="['container', darkMode ? 'dark' : 'light']">
    <div class="mode-toggle">
      <button
        class="mode-btn"
        @click="toggleDarkMode"
        :aria-label="darkMode ? 'Switch to light mode' : 'Switch to dark mode'"
      >
        ☀️
      </button>
    </div>
    <h1>🖼️ Frame Attacher</h1>
    <div class="upload-controls">
      <label class="file-label">
        <span>Upload Your Profile Picture</span>
        <input type="file" accept="image/*" @change="onMainImageUpload" />
      </label>
      <label class="file-label">
        <span>Upload Your Frame</span>
        <input type="file" accept="image/*" @change="onFrameImageUpload" />
      </label>
    </div>
    <div class="sliders">
      <label>
        <span>Zoom</span>
        <input type="range" min="0.5" max="2" step="0.01" v-model="zoom" />
      </label>
      <label>
        <span>Rotation</span>
        <input type="range" min="0" max="360" step="1" v-model="rotation" />
      </label>
    </div>
    <div class="preview-box">
      <img
        v-if="mainImage"
        :src="mainImage"
        :style="{
          position: 'absolute',
          width: '100%',
          height: '100%',
          objectFit: 'contain',
          transform: `scale(${zoom}) rotate(${rotation}deg)`,
          transition: 'transform 0.2s',
        }"
        alt="Main"
      />
      <img
        v-if="frameImage"
        :src="frameImage"
        :style="{
          position: 'absolute',
          width: '100%',
          height: '100%',
          objectFit: 'fill',
          pointerEvents: 'none',
          zIndex: 2,
        }"
        alt="Frame"
      />
    </div>
    <button class="save-btn" @click="saveImage" :disabled="!mainImage || !frameImage">
      💾 Save Image
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const mainImage = ref(null)
const frameImage = ref(null)
const zoom = ref(1)
const rotation = ref(0)
const darkMode = ref(false)

function toggleDarkMode() {
  darkMode.value = !darkMode.value
}

onMounted(() => {
  // Detect system preference
  if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
    darkMode.value = true
  } else {
    darkMode.value = false
  }
  // Listen for changes in system preference
  window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', (e) => {
    darkMode.value = e.matches
  })
})

watch(darkMode, (val) => {
  document.body.style.background = val ? '#181a20' : '#fff'
})

function onMainImageUpload(e) {
  const file = e.target.files[0]
  if (file) mainImage.value = URL.createObjectURL(file)
}

function onFrameImageUpload(e) {
  const file = e.target.files[0]
  if (file) frameImage.value = URL.createObjectURL(file)
}

function saveImage() {
  const canvasSize = 800
  const canvas = document.createElement('canvas')
  canvas.width = canvasSize
  canvas.height = canvasSize
  const ctx = canvas.getContext('2d')

  const mainImg = new Image()
  const frameImg = new Image()
  let loaded = 0

  mainImg.crossOrigin = 'anonymous'
  frameImg.crossOrigin = 'anonymous'

  function tryDraw() {
    loaded++
    if ((!mainImage.value && loaded === 1) || (mainImage.value && loaded === 2)) {
      ctx.clearRect(0, 0, canvasSize, canvasSize)

      if (mainImage.value) {
        const iw = mainImg.naturalWidth
        const ih = mainImg.naturalHeight
        const aspectRatio = iw / ih

        // Determine how to scale the image into the square preview box
        let drawWidth, drawHeight
        if (aspectRatio > 1) {
          // Landscape image
          drawWidth = canvasSize * zoom.value
          drawHeight = (canvasSize / aspectRatio) * zoom.value
        } else {
          // Portrait image
          drawHeight = canvasSize * zoom.value
          drawWidth = canvasSize * aspectRatio * zoom.value
        }

        ctx.save()
        ctx.translate(canvasSize / 2, canvasSize / 2)
        ctx.rotate((rotation.value * Math.PI) / 180)
        ctx.drawImage(mainImg, -drawWidth / 2, -drawHeight / 2, drawWidth, drawHeight)
        ctx.restore()
      }

      if (frameImage.value) {
        ctx.drawImage(frameImg, 0, 0, canvasSize, canvasSize)
      }

      const link = document.createElement('a')
      link.download = 'framed-cropped.png'
      link.href = canvas.toDataURL('image/png')
      link.click()
    }
  }

  if (mainImage.value) {
    mainImg.onload = tryDraw
    mainImg.src = mainImage.value
  } else {
    loaded = 1
  }

  if (frameImage.value) {
    frameImg.onload = tryDraw
    frameImg.src = frameImage.value
  }
}
</script>

<style scoped>
body {
  transition: background 0.2s;
}

.container {
  max-width: 480px;
  margin: 40px auto;
  border-radius: 18px;
  padding: 2rem 2rem 1.5rem 2rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  transition:
    background 0.2s,
    color 0.2s;
  box-shadow: 0 4px 32px 0 rgba(0, 0, 0, 0.1);
}
.container.dark {
  background: #181a20;
  color: #f1f5f9;
}
.container.light {
  background: #fff;
  color: #2d3748;
}

.mode-toggle {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1.2rem;
  align-self: flex-end;
}
.mode-btn {
  background: none;
  border: 1px solid #4a5568;
  color: inherit;
  border-radius: 6px;
  padding: 0.3rem 1.1rem;
  font-size: 1rem;
  cursor: pointer;
  transition:
    background 0.2s,
    border 0.2s;
  opacity: 0.7;
}
.mode-btn.active,
.mode-btn:hover {
  background: #4a5568;
  color: #fff;
  opacity: 1;
  border-color: #4a5568;
}
.container.dark .mode-btn.active,
.container.dark .mode-btn:hover {
  background: #2d3748;
  color: #fff;
  border-color: #2d3748;
}

h1 {
  font-weight: 700;
  font-size: 2rem;
  margin-bottom: 1.5rem;
  letter-spacing: 1px;
}

.upload-controls {
  display: flex;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.file-label {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #f7fafc;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 0.75rem 1rem;
  cursor: pointer;
  font-size: 0.98rem;
  color: #4a5568;
  transition:
    background 0.2s,
    color 0.2s,
    border 0.2s;
}
.file-label:hover {
  background: #e2e8f0;
}
.file-label input[type='file'] {
  display: none;
}
.container.dark .file-label {
  background: #23272f;
  border: 1px solid #353945;
  color: #f1f5f9;
}
.container.dark .file-label:hover {
  background: #353945;
}

.sliders {
  display: flex;
  gap: 2rem;
  margin-bottom: 1.5rem;
  width: 100%;
  justify-content: center;
}
.sliders label {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-size: 0.95rem;
}
.sliders input[type='range'] {
  width: 120px;
  margin-top: 0.5rem;
}

.preview-box {
  position: relative;
  width: 400px;
  height: 400px;
  border-radius: 12px;
  border: 2px solid #e2e8f0;
  background: #f7fafc;
  overflow: hidden;
  margin-bottom: 1.5rem;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.06);
  display: flex;
  align-items: center;
  justify-content: center;
  transition:
    background 0.2s,
    border 0.2s;
}
.container.dark .preview-box {
  background: #23272f;
  border: 2px solid #353945;
}

.save-btn {
  background: linear-gradient(90deg, #667eea 0%, #5a67d8 100%);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 0.75rem 2rem;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition:
    background 0.2s,
    box-shadow 0.2s;
  box-shadow: 0 2px 8px 0 rgba(102, 126, 234, 0.1);
}
.save-btn:disabled {
  background: #cbd5e1;
  cursor: not-allowed;
  color: #a0aec0;
  box-shadow: none;
}
.save-btn:not(:disabled):hover {
  background: linear-gradient(90deg, #5a67d8 0%, #667eea 100%);
  box-shadow: 0 4px 16px 0 rgba(102, 126, 234, 0.18);
}
</style>
