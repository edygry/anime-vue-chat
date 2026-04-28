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
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { VRMLoaderPlugin } from '@pixiv/three-vrm'

export default {
  name: 'AvatarCanvas',
  props: {
    isSpeaking: {
      type: Boolean,
      default: false
    },
    avatarUrl: {
      type: String,
      default: '/avatars/rose.vrm'
    }
  },
  emits: ['ready', 'error', 'avatarLoaded'],
  setup(props, { emit }) {
    const canvasContainer = ref(null)
    const canvas = ref(null)
    const isReady = ref(false)
    const error = ref(null)
    
    let scene, camera, renderer, animationId
    let currentVRM = null
    let clock = new THREE.Clock()

    onMounted(() => {
      initThreeJS()
    })

    onUnmounted(() => {
      cleanup()
    })

    // Watch for avatar URL changes
    watch(() => props.avatarUrl, (newUrl) => {
      if (newUrl && scene) {
        loadAvatar(newUrl)
      }
    })

    async function initThreeJS() {
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
        camera.position.set(0, 1.0, 3)

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

        console.log('Scene created!')

        // Load default avatar
        await loadAvatar(props.avatarUrl)

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

    async function loadAvatar(url) {
      try {
        // Dispose current VRM if exists
        if (currentVRM) {
          if (currentVRM.scene) {
            scene.remove(currentVRM.scene)
          }
          currentVRM = null
        }

        // Create loader
        const loader = new GLTFLoader()
        loader.register((parser) => {
          return new VRMLoaderPlugin(parser)
        })

        // Load VRM
        const gltf = await loader.loadAsync(url)
        const vrm = gltf.userData.vrm

        if (!vrm) {
          throw new Error('No VRM data found in file')
        }

        currentVRM = vrm
        scene.add(vrm.scene)

        // Reset VRM position
        vrm.scene.position.set(0, 0, 0)

        console.log('VRM loaded:', url)
        emit('avatarLoaded', {
          url,
          meta: vrm.meta
        })

      } catch (e) {
        console.error('Failed to load avatar:', e)
        // Fallback to geometric avatar
        createFallbackAvatar()
      }
    }

    function createFallbackAvatar() {
      // Create a simple geometric avatar as fallback
      const group = new THREE.Group()

      // Body
      const bodyGeometry = new THREE.CapsuleGeometry(0.3, 1.0, 8, 16)
      const bodyMaterial = new THREE.MeshStandardMaterial({
        color: 0xff6b9d,
        emissive: 0xff6b9d,
        emissiveIntensity: 0.3
      })
      const body = new THREE.Mesh(bodyGeometry, bodyMaterial)
      body.position.set(0, 1.0, 0)
      group.add(body)

      // Head
      const headGeometry = new THREE.SphereGeometry(0.25, 32, 32)
      const headMaterial = new THREE.MeshStandardMaterial({
        color: 0xffb6c1,
        emissive: 0xffb6c1,
        emissiveIntensity: 0.2
      })
      const head = new THREE.Mesh(headGeometry, headMaterial)
      head.position.set(0, 1.8, 0)
      group.add(head)

      scene.add(group)
      currentVRM = { 
        scene: group,
        update: function() {} // Dummy update method
      }
    }

    function animate() {
      animationId = requestAnimationFrame(animate)

      const delta = clock.getDelta()
      const time = clock.elapsedTime

      // Update VRM animation
      if (currentVRM) {
        // Call VRM update if it exists (VRM v2/v3 API)
        if (currentVRM.update && typeof currentVRM.update === 'function') {
          currentVRM.update(delta)
        }

        // Idle animation - body sway
        if (currentVRM.scene) {
          currentVRM.scene.rotation.y = Math.sin(time * 0.5) * 0.05
        }

        // Breathing animation on chest bone
        if (currentVRM.humanoid) {
          try {
            const chest = currentVRM.humanoid.getNormalizedBoneTransform('chest')
            if (chest) {
              const breathe = Math.sin(time * 2) * 0.02
              chest.scale.set(1 + breathe, 1 + breathe, 1 + breathe)
            }

            const head = currentVRM.humanoid.getNormalizedBoneTransform('head')
            if (head) {
              const bob = Math.sin(time * 3) * 0.02
              head.rotation.x = bob
            }
          } catch (e) {
            // Ignore bone transform errors
          }
        }
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

    // Expose methods for parent components
    function switchAvatar(url) {
      loadAvatar(url)
    }

    function uploadAvatar(file) {
      const url = URL.createObjectURL(file)
      loadAvatar(url)
    }

    return {
      canvasContainer,
      canvas,
      isReady,
      error,
      switchAvatar,
      uploadAvatar
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
  pointer-events: none;
}

canvas {
  width: 100%;
  height: 100%;
  display: block;
  pointer-events: auto;
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
