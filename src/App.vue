<template>
  <div class="app">
    <!-- 3D Avatar Background -->
    <AvatarCanvas
      :isSpeaking="isSpeaking"
      :avatarUrl="currentAvatar"
      @ready="onAvatarReady"
      @error="onAvatarError"
      @avatarLoaded="onAvatarLoaded"
    />

    <!-- Chat Interface -->
    <ChatInterface @message="onMessage" :isTyping="isTyping" />

    <!-- Character Info Overlay -->
    <div class="character-info">
      <div class="info-card">
        <div class="info-content">
          <span class="character-emoji">{{ currentCharacter.emoji }}</span>
          <div class="character-details">
            <h2 class="character-name">{{ currentCharacter.name }}</h2>
            <p class="character-type">{{ currentCharacter.type }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Character Selector -->
    <CharacterSelector
      :currentCharacter="currentCharacter"
      @select="onCharacterSelect"
    />

    <!-- Avatar Upload -->
    <AvatarUpload @upload="onAvatarUpload" />
  </div>
</template>

<script>
import { ref } from 'vue'
import AvatarCanvas from './components/AvatarCanvas.vue'
import ChatInterface from './components/ChatInterface.vue'
import CharacterSelector from './components/CharacterSelector.vue'
import AvatarUpload from './components/AvatarUpload.vue'

export default {
  name: 'App',
  components: {
    AvatarCanvas,
    ChatInterface,
    CharacterSelector,
    AvatarUpload
  },
  setup() {
    const isTyping = ref(false)
    const isSpeaking = ref(false)
    const avatarReady = ref(false)
    const currentAvatar = ref('/avatars/rose.vrm')
    const currentCharacter = ref({
      name: 'Rose',
      type: 'Human Girl • Calm',
      file: '/avatars/rose.vrm',
      emoji: '🌹'
    })

    const onAvatarReady = () => {
      console.log('Avatar ready!')
      avatarReady.value = true
    }

    const onAvatarError = (error) => {
      console.error('Avatar error:', error)
    }

    const onAvatarLoaded = (data) => {
      console.log('Avatar loaded:', data)
    }

    const onMessage = (message) => {
      isTyping.value = true
      // Simulate typing delay
      setTimeout(() => {
        isTyping.value = false
      }, 1500)
    }

    const onCharacterSelect = (character) => {
      currentCharacter.value = character
      currentAvatar.value = character.file
    }

    const onAvatarUpload = (file) => {
      console.log('Avatar uploaded:', file.name)
      // The AvatarCanvas component will handle the upload
      // We just need to trigger it through a ref or event
    }

    return {
      isTyping,
      isSpeaking,
      avatarReady,
      currentAvatar,
      currentCharacter,
      onAvatarReady,
      onAvatarError,
      onAvatarLoaded,
      onMessage,
      onCharacterSelect,
      onAvatarUpload
    }
  }
}
</script>

<style>
/* Global styles */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
  background: #0a0a0f;
  color: #e0e0e0;
  min-height: 100vh;
  overflow: hidden;
}

#app {
  width: 100vw;
  height: 100vh;
  position: relative;
}

.app {
  width: 100%;
  height: 100%;
  position: relative;
}

/* Character Info Overlay */
.character-info {
  position: absolute;
  top: 16px;
  left: 16px;
  z-index: 20;
}

.info-card {
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(10px);
  border-radius: 12px;
  padding: 12px 16px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.info-content {
  display: flex;
  align-items: center;
  gap: 12px;
}

.character-emoji {
  font-size: 32px;
}

.character-details {
  display: flex;
  flex-direction: column;
}

.character-name {
  font-size: 16px;
  font-weight: bold;
  color: white;
  margin-bottom: 2px;
}

.character-type {
  font-size: 12px;
  color: #888;
}

/* Responsive */
@media (max-width: 768px) {
  .character-info {
    top: 12px;
    left: 12px;
  }

  .info-card {
    padding: 8px 12px;
  }

  .character-emoji {
    font-size: 24px;
  }

  .character-name {
    font-size: 14px;
  }

  .character-type {
    font-size: 10px;
  }
}
</style>
