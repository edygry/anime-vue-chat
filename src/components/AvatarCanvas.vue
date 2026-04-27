<template>
  <div class="avatar-canvas" ref="canvasContainer">
    <canvas ref="canvas"></canvas>
    <div v-if="!isReady" class="loading-overlay">
      <div class="loading-spinner">🔄</div>
      <p>Initializing 3D scene...</p>
    </div>
    <div v-if="error" class="error-overlay">
      <div class="error-icon">🎮</div>
      <h3>3D Avatar Unavailable</h3>
      <p>{{ error }}</p>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'

export default {
  name: 'AvatarCanvas',
  props: {
    isSpeaking: {
      type: Boolean,
      default: false
    }
  },
  emits: ['ready', 'error'],
  setup(props) {
    const canvasContainer = ref(null)
    const canvas = ref(null)
    const isReady = ref(false)
    const error = ref(null)
    
    let scene, camera, renderer, animationId
    let cube, avatarGroup

    onMounted(() => {
      initThreeJS()
    })

    onUnmounted(() => {
      cleanup()
    })

    function initThreeJS() {
      try {
        // Check WebGL support
        const testCanvas = document.createElement('canvas')
        const gl = testCanvas.getContext('webgl') || testCanvas.getContext('webgl2')
        
        if (!gl) {
          error.value = 'WebGL is not supported in your browser'
          emit('error', error.value)
          return
        }

        console.log('WebGL supported!')

        // Create scene
        scene = new THREE.Scene()
        scene.background = new THREE.Color(0x1a1a2e)

        // Create camera
        camera = new THREE.PerspectiveCamera(
          50,
          window.innerWidth / window.innerHeight,
          0.1,
          1000
        )
        camera.position.set(0, 1.5, 4)

        // Create renderer
        renderer = new THREE.WebGLRenderer({
          canvas: canvas.value,
          antialias: true,
          alpha: false
        })
        renderer.setSize(window.innerWidth, window.innerHeight)
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
        renderer.shadowMap.enabled = true

        console.log('Renderer created!')

        // Create avatar group
        avatarGroup = new THREE.Group()
        scene.add(avatarGroup)

        // Create test cube
        const cubeGeometry = new THREE.BoxGeometry(1, 1, 1)
        const cubeMaterial = new THREE.MeshStandardMaterial({
          color: 0xff6b9d,
          emissive: 0xff6b9d,
          emissiveIntensity: 0.3
        })
        cube = new THREE.Mesh(cubeGeometry, cubeMaterial)
        cube.position.set(0, 1.5, 0)
        cube.castShadow = true
        avatarGroup.add(cube)

        // Create body
        const bodyGeometry = new THREE.CapsuleGeometry(0.5, 1.5, 8, 16)
        const bodyMaterial = new THREE.MeshStandardMaterial({
          color: 0xff6b9d,
          emissive: 0xff6b9d,
          emissiveIntensity: 0.3,
          roughness: 0.3,
          metalness: 0.1
        })
        const body = new THREE.Mesh(bodyGeometry, bodyMaterial)
        body.position.set(0, 1.2, 0)
        body.castShadow = true
        avatarGroup.add(body)

        // Create head
        const headGeometry = new THREE.SphereGeometry(0.4, 32, 32)
        const headMaterial = new THREE.MeshStandardMaterial({
          color: 0xffb6c1,
          emissive: 0xffb6c1,
          emissiveIntensity: 0.2,
          roughness: 0.2,
          metalness: 0.05
        })
        const head = new THREE.Mesh(headGeometry, headMaterial)
        head.position.set(0, 2.2, 0)
        head.castShadow = true
        avatarGroup.add(head)

        // Create eyes
        const eyeGeometry = new THREE.SphereGeometry(0.08, 16, 16)
        const eyeMaterial = new THREE.MeshStandardMaterial({
          color: 0x000000,
          emissive: 0x000000,
          emissiveIntensity: 0.5
        })

        const leftEye = new THREE.Mesh(eyeGeometry, eyeMaterial)
        leftEye.position.set(-0.12, 2.25, 0.35)
        avatarGroup.add(leftEye)

        const rightEye = new THREE.Mesh(eyeGeometry, eyeMaterial)
        rightEye.position.set(0.12, 2.25, 0.35)
        avatarGroup.add(rightEye)

        // Create hair
        const hairGeometry = new THREE.SphereGeometry(0.45, 32, 32, 0, Math.PI * 2, 0, Math.PI / 2)
        const hairMaterial = new THREE.MeshStandardMaterial({
          color: 0xff69b4,
          emissive: 0xff69b4,
          emissiveIntensity: 0.2,
          roughness: 0.4,
          metalness: 0.1
        })
        const hair = new THREE.Mesh(hairGeometry, hairMaterial)
        hair.position.set(0, 2.3, -0.1)
        avatarGroup.add(hair)

        // Create ears
        const earGeometry = new THREE.ConeGeometry(0.15, 0.4, 8)
        const earMaterial = new THREE.MeshStandardMaterial({
          color: 0xff69b4,
          emissive: 0xff69b4,
          emissiveIntensity: 0.2
        })

        const leftEar = new THREE.Mesh(earGeometry, earMaterial)
        leftEar.position.set(-0.3, 2.5, 0)
        leftEar.rotation.z = 0.4
        avatarGroup.add(leftEar)

        const rightEar = new THREE.Mesh(earGeometry, earMaterial)
        rightEar.position.set(0.3, 2.5, 0)
        rightEar.rotation.z = -0.4
        avatarGroup.add(rightEar)

        // Create ground
        const groundGeometry = new THREE.PlaneGeometry(10, 10)
        const groundMaterial = new THREE.MeshStandardMaterial({
          color: 0x0a0a0f,
          roughness: 0.8,
          metalness: 0.2
        })
        const ground = new THREE.Mesh(groundGeometry, groundMaterial)
        ground.rotation.x = -Math.PI / 2
        ground.position.y = 0
        ground.receiveShadow = true
        scene.add(ground)

        // Create lights
        const ambientLight = new THREE.AmbientLight(0xffffff, 2.0)
        scene.add(ambientLight)

        const directionalLight = new THREE.DirectionalLight(0xffffff, 3.0)
        directionalLight.position.set(2, 4, 3)
        directionalLight.castShadow = true
        scene.add(directionalLight)

        const backLight = new THREE.DirectionalLight(0xff6b9d, 2.0)
        backLight.position.set(-2, 2, -2)
        scene.add(backLight)

        const rimLight = new THREE.DirectionalLight(0xc44dff, 1.5)
        rimLight.position.set(0, 3, -3)
        scene.add(rimLight)

        console.log('Scene created!')

        // Start animation loop
        isReady.value = true
        emit('ready')
        animate()

        // Handle resize
        window.addEventListener('resize', onWindowResize)

      } catch (e) {
        console.error('Three.js initialization error:', e)
        error.value = 'Failed to initialize 3D scene: ' + e.message
        emit('error', error.value)
      }
    }

    function animate() {
      animationId = requestAnimationFrame(animate)

      const time = Date.now() * 0.001

      // Rotate cube
      if (cube) {
        cube.rotation.x += 0.01
        cube.rotation.y += 0.01
      }

      // Animate avatar
      if (avatarGroup) {
        // Breathing animation
        avatarGroup.position.y = Math.sin(time * 2) * 0.03
        avatarGroup.rotation.y = Math.sin(time * 0.5) * 0.1
      }

      // Render
      if (renderer && scene && camera) {
        renderer.render(scene, camera)
      }
    }

    function onWindowResize() {
      if (!camera || !renderer) return
      
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
    }

    function cleanup() {
      if (animationId) {
        cancelAnimationFrame(animationId)
      }
      if (renderer) {
        renderer.dispose()
      }
      window.removeEventListener('resize', onWindowResize)
    }

    return {
      canvasContainer,
      canvas,
      isReady,
      error
    }
  }
}
</script>

<style scoped>
.avatar-canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  background: #1a1a2e;
}

canvas {
  width: 100%;
  height: 100%;
  display: block;
}

.loading-overlay,
.error-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgba(26, 26, 46, 0.9);
  color: white;
  z-index: 10;
}

.loading-spinner {
  font-size: 48px;
  animation: spin 2s linear infinite;
  margin-bottom: 16px;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.error-icon {
  font-size: 64px;
  margin-bottom: 16px;
}

.error-overlay h3 {
  font-size: 20px;
  margin-bottom: 8px;
}

.error-overlay p {
  color: #888;
  font-size: 14px;
}
</style>
