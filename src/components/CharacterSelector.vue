<template>
  <div class="character-selector">
    <button class="selector-btn" @click="togglePanel">
      <span class="icon">👤</span>
      <span class="label">{{ currentCharacter.name }}</span>
    </button>

    <div v-if="showPanel" class="panel">
      <div class="panel-header">
        <h3>Choose Character</h3>
        <button class="close-btn" @click="togglePanel">×</button>
      </div>

      <div class="character-list">
        <div
          v-for="char in characters"
          :key="char.file"
          class="character-item"
          :class="{ active: currentCharacter.file === char.file }"
          @click="selectCharacter(char)"
        >
          <div class="character-avatar">
            <span class="emoji">{{ char.emoji }}</span>
          </div>
          <div class="character-info">
            <div class="character-name">{{ char.name }}</div>
            <div class="character-type">{{ char.type }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'CharacterSelector',
  props: {
    currentCharacter: {
      type: Object,
      default: () => ({
        name: 'Rose',
        type: 'Human Girl • Calm',
        file: '/avatars/rose.vrm',
        emoji: '🌹'
      })
    }
  },
  emits: ['select'],
  setup(props, { emit }) {
    const showPanel = ref(false)

    const characters = [
      {
        name: 'Rose',
        type: 'Human Girl • Calm',
        file: '/avatars/rose.vrm',
        emoji: '🌹'
      },
      {
        name: 'Rabbit',
        type: 'Rabbit Girl • Energetic',
        file: '/avatars/rabbit.vrm',
        emoji: '🐰'
      },
      {
        name: 'Pyre',
        type: 'Sorcerer • Mysterious',
        file: '/avatars/pyre-sorcerer.vrm',
        emoji: '🔥'
      },
      {
        name: 'Cool Pan',
        type: 'Chef • Friendly',
        file: '/avatars/cool-pan.vrm',
        emoji: '🍳'
      }
    ]

    function togglePanel() {
      showPanel.value = !showPanel.value
    }

    function selectCharacter(character) {
      emit('select', character)
      showPanel.value = false
    }

    return {
      showPanel,
      characters,
      togglePanel,
      selectCharacter
    }
  }
}
</script>

<style scoped>
.character-selector {
  position: absolute;
  bottom: 16px;
  left: 16px;
  z-index: 20;
  pointer-events: auto;
}

.selector-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  color: white;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.selector-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.2);
}

.icon {
  font-size: 18px;
}

.label {
  font-weight: 500;
}

.panel {
  position: absolute;
  bottom: 60px;
  left: 0;
  width: 280px;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  overflow: hidden;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.panel-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: white;
  margin: 0;
}

.close-btn {
  background: none;
  border: none;
  color: #888;
  font-size: 24px;
  cursor: pointer;
  padding: 0;
  line-height: 1;
}

.close-btn:hover {
  color: white;
}

.character-list {
  padding: 8px;
}

.character-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.character-item:hover {
  background: rgba(255, 255, 255, 0.05);
}

.character-item.active {
  background: rgba(255, 255, 255, 0.1);
}

.character-avatar {
  width: 40px;
  height: 40px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.emoji {
  font-size: 24px;
}

.character-info {
  flex: 1;
}

.character-name {
  font-size: 14px;
  font-weight: 600;
  color: white;
  margin-bottom: 2px;
}

.character-type {
  font-size: 12px;
  color: #888;
}
</style>
