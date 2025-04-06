<script setup>
import { ref, reactive, onMounted, nextTick } from 'vue'
import { GoogleGenerativeAI, HarmCategory, HarmBlockThreshold } from '@google/generative-ai'

// API KEY 填這裡
const apiKey = ''

const modelName = 'gemini-1.5-flash'

// prompt
const systemInstruction = reactive('')

// 基本設定
const generationConfig = reactive({
  temperature: 1,
  topP: 0.95,
  topK: 40,
  maxOutputTokens: 8192,
  responseModalities: [],
  responseMimeType: 'text/plain',
})

// 初始化 AI model
const genAI = new GoogleGenerativeAI(apiKey)
const model = genAI.getGenerativeModel({ model: modelName })

// 建立對話
const chatSession = model.startChat({
  generationConfig,
  history: [
    {
      role: 'user',
      parts: [{ text: 'What is bigger, 9.9 or 9.11?' }],
    },
    {
      role: 'model',
      parts: [
        {
          text: '9.9 is bigger than 9.11.  Think of it as 9.90 vs. 9.11.  The tenths place is larger in 9.9 (9 tenths vs. 1 tenth).\n',
        },
      ],
    },
  ],
})

// 對話歷史
const chatHistory = reactive([])

// 使用者輸入訊息
const userMessage = ref('')

// textarea
const textAreaRef = ref()
const rowHeight = ref(1)

const isLoading = ref(false)

const messageListRef = ref([])

onMounted(async () => await getHistory())

// 取得聊天記錄
async function getHistory() {
  try {
    const response = await chatSession.getHistory()

    for (const message of response) {
      const { role, parts } = message

      const content = parts.reduce(
        (completeText, currentText) => completeText.concat(currentText.text),
        '',
      )

      chatHistory.push({
        role,
        content,
      })
    }
  } catch (error) {
    console.error(error)
  }
}

// 傳送訊息
async function sendMessage(message) {
  if (isLoading.value) return
  if (!message.trim()) return

  isLoading.value = true

  chatHistory.push({
    role: 'user',
    content: message,
  })

  userMessage.value = ''

  nextTick(() => {
    messageListRef.value[`${chatHistory.length - 1}`].scrollIntoView({
      behavior: 'smooth',
      block: 'start',
      inline: 'center',
    })
  })

  try {
    const response = await chatSession.sendMessage(message)
    const result = response.response

    const modelMessage = result.text()

    chatHistory.push({
      role: 'model',
      content: modelMessage,
    })
    nextTick(() => {
      messageListRef.value[`${chatHistory.length - 2}`].scrollIntoView({
        behavior: 'smooth',
        block: 'start',
        inline: 'center',
      })
    })
  } catch (error) {
    console.error(error)
  } finally {
    isLoading.value = false
  }
}

function setRowHeight(event) {
  const rowLimit = 5

  // 處理 Shift + Enter
  if (event.key === 'Enter' && event.shiftKey) {
    // 計算行數
    const lineCount = userMessage.value.split('\n').length

    if (lineCount >= rowLimit) {
      rowHeight.value = rowLimit
      return
    }
  }

  // 設定高度
  nextTick(() => {
    const lineCount = userMessage.value.split('\n').length
    rowHeight.value = Math.min(lineCount, rowLimit)
  })
}
</script>

<template>
  <main>
    <div class="chat-room">
      <!-- <div class="instruction">
        <input type="text" placeholder="System Instructions" />
      </div> -->

      <div class="header">
        <!-- gemini icon -->
        <svg fill="none" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16">
          <path
            d="M16 8.016A8.522 8.522 0 008.016 16h-.032A8.521 8.521 0 000 8.016v-.032A8.521 8.521 0 007.984 0h.032A8.522 8.522 0 0016 7.984v.032z"
            fill="url(#prefix__paint0_radial_980_20147)"
          />
          <defs>
            <radialGradient
              id="prefix__paint0_radial_980_20147"
              cx="0"
              cy="0"
              r="1"
              gradientUnits="userSpaceOnUse"
              gradientTransform="matrix(16.1326 5.4553 -43.70045 129.2322 1.588 6.503)"
            >
              <stop offset=".067" stop-color="#9168C0" />
              <stop offset=".343" stop-color="#5684D1" />
              <stop offset=".672" stop-color="#1BA1E3" />
            </radialGradient>
          </defs>
        </svg>
        <!-- gemini icon -->
        <h1 class="title">Gemini</h1>
      </div>

      <div class="message-area">
        <div
          v-for="(message, messageIdx) in chatHistory"
          ref="messageListRef"
          class="message"
          :key="messageIdx"
          :class="{ own: message.role == 'user' }"
        >
          <span class="bubble">{{ message.content }}</span>
        </div>

        <!-- 載入中圖示 -->
        <div v-if="isLoading" class="message">
          <span class="bubble">
            <span class="loading-dot">
              <span class="dot" v-for="num in 3" :key="num"></span>
            </span>
          </span>
        </div>
      </div>

      <form @submit.prevent="sendMessage(userMessage)" class="footer">
        <!-- @keydown="setRowHeight($event)" -->
        <textarea
          v-model="userMessage"
          @input="setRowHeight($event)"
          :rows="rowHeight"
          ref="textAreaRef"
          type="text"
          placeholder="與 Gemini 聊聊..."
        />
        <button type="submit" :disabled="isLoading">
          <!-- send icon -->
          <svg height="48" viewBox="0 0 48 48" width="48" xmlns="http://www.w3.org/2000/svg">
            <path d="M4.02 42L46 24 4.02 6 4 20l30 4-30 4z" />
            <path d="M0 0h48v48H0z" fill="none" />
          </svg>
        </button>
      </form>
    </div>
  </main>
</template>

<style lang="scss" scoped>
main {
  background-color: snow;
  min-height: 100vh;
  padding: 10%;

  min-width: 640px;
}

.chat-room {
  --border-color: #eef2f4;

  margin: 0 auto;
  min-height: 40vh;
  max-height: 80vh;

  position: relative;
  transition: box-shadow 0.65s ease-in-out;

  border-radius: 12px;
  overflow: hidden;
  background-color: white;

  display: flex;
  flex-direction: column;

  font-family: 'Noto Sans TC';
  color: #273346;

  box-shadow:
    rgba(0, 0, 0, 0.25) 0px 54px 55px,
    rgba(0, 0, 0, 0.12) 0px -12px 30px,
    rgba(0, 0, 0, 0.12) 0px 4px 6px,
    rgba(0, 0, 0, 0.17) 0px 12px 13px,
    rgba(0, 0, 0, 0.09) 0px -3px 5px;

  &:focus-within {
    box-shadow: rgba(17, 12, 46, 0.15) 0px 48px 100px 0px;
  }

  .instruction {
    border-bottom: 1px solid var(--border-color);
    padding: 0px 20px;
    height: 60px;

    input {
      outline: initial;
      height: 100%;
      width: 100%;
      font-weight: 600;
      font-size: 15px;
    }
  }

  .header {
    z-index: 2;
    width: 100%;

    padding: 20px;
    position: absolute;

    display: flex;

    top: 0px;
    left: 0px;

    background: linear-gradient(
      to bottom,
      rgba(255, 255, 255, 1) 0%,
      rgba(255, 255, 255, 1) 78%,
      rgba(255, 255, 255, 0) 100%
    );

    *:nth-child(n + 2) {
      margin-left: 5px;
    }

    .title {
      font-size: 18px;
      font-weight: 600;
    }

    svg {
      width: 20px;
    }
  }

  .message-area {
    flex-grow: 1;
    padding: 80px 20px 0px;
    overflow: scroll;
  }

  .message {
    display: flex;
    margin-bottom: 20px;
    /* 設定滾動時的上方間距 */
    scroll-margin-top: 100px;

    .bubble {
      max-width: 80%;
      border-radius: 20px 20px 20px 0px;
      padding: 10px;
      background-color: #f1f2f6;
      word-break: break-all;

      animation: popup 0.6s ease forwards;
    }

    &.own {
      flex-direction: row-reverse;
      .bubble {
        border-radius: 20px 20px 0px 20px;
        background-color: #0086ff;
        color: white;
      }
    }

    &:has(.loading-dot) {
      justify-content: center;

      .bubble {
        border-radius: 20px;
      }
    }

    @keyframes popup {
      0% {
        transform: translateY(20px) scale(0.8);
        opacity: 0;
      }
      100% {
        transform: translateY(0) scale(1);
        opacity: 1;
      }
    }
  }

  .loading-dot {
    display: flex;
    align-items: center;
    justify-content: center;

    .dot {
      width: 6px;
      height: 6px;
      margin: 0 2px;
      border-radius: 50%;

      animation: bounce 1.4s infinite ease-in-out both;

      &:nth-child(1) {
        background-color: #9177c7;
        background: linear-gradient(135deg, #9177c7, #847fcb);

        animation-delay: -0.32s;
      }

      &:nth-child(2) {
        background-color: #6d8ed4;
        background: linear-gradient(135deg, #7987cf, #6d8ed4);
        animation-delay: -0.16s;
      }

      &:nth-child(3) {
        background-color: #4796e3;
        background: linear-gradient(135deg, #4ba5e0, #4796e3);
      }
    }

    @keyframes bounce {
      0%,
      80%,
      100% {
        transform: scale(0);
      }
      40% {
        transform: scale(1);
      }
    }
  }

  .footer {
    border-top: 1px solid var(--border-color);
    padding: 10px 20px;

    display: flex;
    justify-content: center;

    textarea {
      border-radius: 6px;
      padding: 12px;
      flex-grow: 1;

      overflow-y: auto;
      resize: none;

      background-color: #f8f8fa;
      outline: initial;
    }

    button {
      padding: 5px;

      &:disabled {
        cursor: not-allowed;
        svg {
          fill: lightgray;
        }
      }
    }

    svg {
      width: 32px;
      height: 32px;

      fill: #0086ff;
      transition: fill 0.35s ease-out;
    }
  }

  /* 整個滾動條的寬度 */
  ::-webkit-scrollbar {
    width: 8px;
    height: 8px;
  }

  ::-webkit-scrollbar-track {
    background: transparent;
  }

  ::-webkit-scrollbar-thumb {
    background: rgba(0, 0, 0, 0.3);
    border-radius: 4px;
    transition: background 0.3s;
  }

  ::-webkit-scrollbar-thumb:hover {
    background: rgba(0, 0, 0, 0.5);
  }
}
</style>
