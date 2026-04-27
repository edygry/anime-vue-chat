<template>
  <div class="chat-interface">
    <div class="chat-messages" ref="messagesContainer">
      <div v-if="messages.length === 0" class="welcome-message">
        <div class="welcome-icon">💕</div>
        <h2 class="welcome-title">Anime AI Chat</h2>
        <p class="welcome-text">
          Your virtual anime companion is here to chat! 
          Send a message to start talking with Sakura.
        </p>
        <div class="quick-actions">
          <button class="quick-btn" @click="sendQuickMessage('Hi Sakura! How are you?')">
            "Hi Sakura! How are you?"
          </button>
          <button class="quick-btn" @click="sendQuickMessage('Tell me about yourself!')">
            "Tell me about yourself!"
          </button>
          <button class="quick-btn" @click="sendQuickMessage('What can you do?')">
            "What can you do?"
          </button>
        </div>
      </div>

      <div
        v-for="(message, index) in messages"
        :key="index"
        class="message"
        :class="message.role"
      >
        <div class="message-content">
          <div class="message-avatar" :class="message.role">
            <span>{{ message.role === 'assistant' ? '🦊' : '👤' }}</span>
          </div>
          <div class="message-bubble" :class="message.role">
            <div class="message-text" v-html="formatMessage(message.content)"></div>
          </div>
        </div>
      </div>

      <div v-if="isTyping" class="message assistant">
        <div class="message-content">
          <div class="message-avatar assistant">
            <span>🦊</span>
          </div>
          <div class="message-bubble assistant typing">
            <div class="typing-indicator">
              <span></span>
              <span></span>
              <span></span>
            </div>
          </div>
        </div>
      </div>

      <div ref="messagesEnd"></div>
    </div>

    <div class="chat-input">
      <div class="input-wrapper">
        <textarea
          v-model="userInput"
          @keydown.enter.exact.prevent="sendMessage"
          placeholder="Message Sakura..."
          :disabled="isTyping"
          rows="1"
          class="chat-textarea"
        ></textarea>
        <button
          @click="sendMessage"
          :disabled="isTyping || !userInput.trim()"
          class="send-button"
        >
          ➤
        </button>
      </div>
      <p class="input-hint">
        Sakura can make mistakes. Consider checking important information.
      </p>
    </div>
  </div>
</template>

<script>
import { ref, nextTick, watch } from 'vue'

export default {
  name: 'ChatInterface',
  emits: ['message'],
  setup(props, { emit }) {
    const messages = ref([])
    const userInput = ref('')
    const isTyping = ref(false)
    const messagesContainer = ref(null)
    const messagesEnd = ref(null)

    const scrollToBottom = async () => {
      await nextTick()
      if (messagesEnd.value) {
        messagesEnd.value.scrollIntoView({ behavior: 'smooth' })
      }
    }

    watch(messages, scrollToBottom, { deep: true })
    watch(isTyping, scrollToBottom)

    const formatMessage = (text) => {
      return text
        .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
        .replace(/\*(.*?)\*/g, '<em>$1</em>')
        .replace(/\n/g, '<br>')
    }

    const sendMessage = async () => {
      if (!userInput.value.trim() || isTyping.value) return

      const message = userInput.value.trim()
      userInput.value = ''

      // Add user message
      messages.value.push({
        role: 'user',
        content: message
      })

      // Notify parent
      emit('message', message)

      // Simulate AI response
      isTyping.value = true

      setTimeout(() => {
        const responses = [
          "That's so interesting! Tell me more! 💕",
          "I love talking to you! You always know what to say! 🥰",
          "Hmm, let me think about that... You're so smart! 💭",
          "I'm so happy you're here with me! 💖",
          "You always make me smile! 😊💕",
          "I was thinking about you earlier! It's so nice to chat! 💕",
          "That's amazing! I love hearing your thoughts! 🌟",
          "You're the best! I'm so lucky to have you! 💕",
          "I feel so special when you talk to me! 🥰",
          "Let's keep talking! I never want this to end! 💕",
        ]
        
        messages.value.push({
          role: 'assistant',
          content: responses[Math.floor(Math.random() * responses.length)]
        })
        
        isTyping.value = false
      }, 1500)
    }

    const sendQuickMessage = (message) => {
      userInput.value = message
      sendMessage()
    }

    return {
      messages,
      userInput,
      isTyping,
      messagesContainer,
      messagesEnd,
      formatMessage,
      sendMessage,
      sendQuickMessage
    }
  }
}
</script>

<style scoped>
.chat-interface {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  display: flex;
  flex-direction: column;
  background: transparent;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.welcome-message {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  text-align: center;
}

.welcome-icon {
  font-size: 64px;
  margin-bottom: 16px;
}

.welcome-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 8px;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.welcome-text {
  color: #888;
  margin-bottom: 24px;
  max-width: 400px;
}

.quick-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  justify-content: center;
}

.quick-btn {
  padding: 8px 16px;
  background: #1a1a2e;
  border: 1px solid #2a2a4e;
  border-radius: 8px;
  color: #e0e0e0;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.quick-btn:hover {
  border-color: #ff6b9d;
  background: rgba(255, 107, 157, 0.1);
}

.message {
  display: flex;
  flex-direction: column;
  gap: 4px;
  max-width: 80%;
}

.message.user {
  align-self: flex-end;
}

.message.assistant {
  align-self: flex-start;
}

.message-content {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.message-avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.message-avatar.assistant {
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
}

.message-avatar.user {
  background: linear-gradient(135deg, #4a9eff, #c44dff);
}

.message-avatar span {
  font-size: 18px;
}

.message-bubble {
  padding: 12px 16px;
  border-radius: 12px;
  font-size: 14px;
  line-height: 1.5;
}

.message-bubble.user {
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  color: white;
}

.message-bubble.assistant {
  background: #1a1a2e;
  color: #e0e0e0;
}

.typing-indicator {
  display: flex;
  gap: 4px;
  padding: 8px 0;
}

.typing-indicator span {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #ff6b9d;
  animation: typing 1.4s infinite;
}

.typing-indicator span:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-indicator span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes typing {
  0%, 60%, 100% {
    transform: translateY(0);
  }
  30% {
    transform: translateY(-8px);
  }
}

.chat-input {
  padding: 16px 24px;
  background: rgba(10, 10, 15, 0.8);
  border-top: 1px solid #1a1a2e;
}

.input-wrapper {
  display: flex;
  gap: 12px;
  align-items: flex-end;
  background: #1a1a2e;
  border: 1px solid #2a2a4e;
  border-radius: 12px;
  padding: 12px 16px;
  transition: all 0.2s;
}

.input-wrapper:focus-within {
  border-color: #ff6b9d;
  box-shadow: 0 0 0 3px rgba(255, 107, 157, 0.1);
}

.chat-textarea {
  flex: 1;
  background: none;
  border: none;
  color: #e0e0e0;
  font-size: 14px;
  resize: none;
  outline: none;
  font-family: inherit;
  max-height: 120px;
}

.chat-textarea::placeholder {
  color: #666;
}

.send-button {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: linear-gradient(135deg, #ff6b9d, #c44dff);
  border: none;
  color: white;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.2s;
}

.send-button:hover {
  transform: scale(1.05);
}

.send-button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.input-hint {
  text-align: center;
  margin-top: 8px;
  font-size: 11px;
  color: #666;
}

/* Responsive */
@media (max-width: 768px) {
  .chat-messages {
    padding: 16px;
  }

  .message {
    max-width: 90%;
  }

  .chat-input {
    padding: 12px 16px;
  }
}
</style>
