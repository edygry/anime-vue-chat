<template>
  <div class="avatar-upload">
    <button class="upload-btn" @click="triggerUpload">
      <span class="icon">📤</span>
      <span class="label">Upload Avatar</span>
    </button>
    <input
      ref="fileInput"
      type="file"
      accept=".vrm,.glb,.gltf"
      @change="handleFile"
      style="display: none"
    />
  </div>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'AvatarUpload',
  emits: ['upload'],
  setup(props, { emit }) {
    const fileInput = ref(null)

    function triggerUpload() {
      fileInput.value.click()
    }

    function handleFile(event) {
      const file = event.target.files[0]
      if (file) {
        emit('upload', file)
      }
      // Reset input
      event.target.value = ''
    }

    return {
      fileInput,
      triggerUpload,
      handleFile
    }
  }
}
</script>

<style scoped>
.avatar-upload {
  position: absolute;
  bottom: 16px;
  right: 16px;
  z-index: 20;
}

.upload-btn {
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

.upload-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.2);
}

.icon {
  font-size: 18px;
}

.label {
  font-weight: 500;
}

@media (max-width: 768px) {
  .avatar-upload {
    bottom: 12px;
    right: 12px;
  }

  .upload-btn {
    padding: 8px 12px;
    font-size: 12px;
  }

  .icon {
    font-size: 16px;
  }
}
</style>
