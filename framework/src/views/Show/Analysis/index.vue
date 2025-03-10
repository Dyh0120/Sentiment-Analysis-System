<template>
  <div class="chat-container">
    <!-- 聊天消息区域 -->
    <div class="chat-messages" ref="chatMessages">
      <div v-for="(message, index) in messages" :key="index" :class="['message', message.sender]">
        <!-- 头像 -->
        <img v-if="message.sender === 'bot'" src="../../../assets/image.png" alt="Bot Avatar" class="avatar" />
        <div class="message-content">
          <p>{{ message.text }}</p>
          <!-- 加载效果 -->
          <div v-if="message.loading" class="loading">
            <div class="dot"></div>
            <div class="dot"></div>
            <div class="dot"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- 回到顶端按钮 -->
    <button v-if="showScrollToTop" @click="scrollToTop" class="scroll-to-top-btn">
      回到顶部
    </button>

    <!-- 输入框 -->
    <div class="chat-input">
      <textarea
        v-model="userInput"
        :disabled="isBotResponding"
        @keyup.enter="handleButtonClick"
        placeholder="输入你的问题..."
        ref="textarea"
        @input="adjustTextareaHeight"
      ></textarea>
      <button @click="handleButtonClick" :disabled="!userInput.trim() && !isBotResponding">
        {{ isBotResponding ? '停止回答' : '点击发送' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, nextTick, watch } from 'vue';
import axios from 'axios';

// 本地消息数据
const messages = reactive([
  { sender: 'bot', text: '你好！我是你的助手，有什么可以帮你的吗？', loading: false },
]);

// 接收数据
import { inject } from 'vue';
const personalInfo = inject('personalInfo');
// 用户输入
const userInput = ref('');
// 是否机器人正在回答
const isBotResponding = ref(false);
// 聊天消息容器的引用
const chatMessages = ref(null);
// 输入框的引用
const textarea = ref(null);
// 机器人回答的定时器
let botResponseTimeout = null;
// 控制是否显示“回到顶端”按钮
const showScrollToTop = ref(false);

// 监听消息变化，滚动到底部
watch(
  () => messages.length,
  () => {
    scrollToBottom();
  },
  { flush: 'post' } // 确保 DOM 更新后再滚动
);

// 监听聊天区域的滚动，控制回到顶部按钮显示
watch(
  () => chatMessages.value ? chatMessages.value.scrollTop : 0,
  (scrollTop) => {
    const chatMessagesHeight = chatMessages.value.scrollHeight;
    const visibleHeight = chatMessages.value.clientHeight;
    showScrollToTop.value = scrollTop > 100; // 如果滚动条超过100px则显示回到顶部按钮
  }
);

// 处理按钮点击事件
const handleButtonClick = () => {
  if (isBotResponding.value) {
    stopBotResponse(); // 如果是“停止回答”状态，则停止回答
  } else {
    sendMessage(); // 如果是“发送”状态，则发送消息
  }
};

// 发送消息并通过接口获取回答
const sendMessage = async () => {
  if (userInput.value.trim() === '') return;

  // 添加用户消息
  messages.push({ sender: 'user', text: userInput.value, loading: false });
  const userMessage = userInput.value;
  userInput.value = '';

  // 重置输入框高度
  adjustTextareaHeight();

  // 模拟机器人回复延迟
  isBotResponding.value = true;
  messages.push({ sender: 'bot', text: '', loading: true }); // 添加加载效果
  try {
    // 请求接口，传入所需参数
    const response = await axios.post('http://mxt.nat300.top/user/logout/', {
      school: "河北师范大学",      // 根据实际情况修改
      major: "计算机科学与技术",    // 根据实际情况修改
      province: "河北省",          // 根据实际情况修改
      like: "软件工程",             // 根据实际情况修改
      question: userMessage
    });

    // 模拟机器人回答
    const botAnswer = response.data; // 根据接口返回的数据调整
    setTimeout(() => {
      messages.pop(); // 移除加载效果
      messages.push({ sender: 'bot', text: botAnswer, loading: false });
      isBotResponding.value = false;
    }, 1500); // 等待1.5秒后显示回答

  } catch (error) {
    // 如果请求失败
    messages.pop(); // 移除加载效果
    messages.push({ sender: 'bot', text: '抱歉，我无法获取答案。请稍后再试。', loading: false });
    isBotResponding.value = false;
  }
};

// 滚动到底部
const scrollToBottom = () => {
  nextTick(() => {
    if (chatMessages.value) {
      chatMessages.value.scrollTop = chatMessages.value.scrollHeight;
    }
  });
};

// 滚动到顶部
const scrollToTop = () => {
  nextTick(() => {
    if (chatMessages.value) {
      chatMessages.value.scrollTop = 0;
    }
  });
};

// 停止回答
const stopBotResponse = () => {
  if (botResponseTimeout) {
    clearTimeout(botResponseTimeout);
    isBotResponding.value = false;
    messages.pop(); // 移除加载效果
    messages.push({ sender: 'bot', text: '已停止回答。', loading: false });
  }
};

// 动态调整输入框高度
const adjustTextareaHeight = () => {
  if (textarea.value) {
    textarea.value.style.height = 'auto'; // 重置高度
    textarea.value.style.height = `${textarea.value.scrollHeight}px`; // 设置高度为内容高度
  }
};
</script>

<style scoped>
/* 容器样式 */
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 1.5vw;  /* 调小容器的 padding */
  box-sizing: border-box;
  background-color: #f5f5f5;
}

/* 聊天消息区域 */
.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 1vw;
}

/* 消息样式 */
.message {
  display: flex;
  align-items: flex-start;
  margin-bottom: 0.8vw;  /* 调小消息之间的间距 */
}

.message.bot {
  justify-content: flex-start;
}

.message.user {
  justify-content: flex-end;
  align-items: flex-end;
}

/* 头像样式 */
.avatar {
  width: 5vw;  /* 调小头像大小 */
  height: 5vw;  /* 调小头像大小 */
  border-radius: 50%;
  margin-right: 1vw;  /* 调小头像与消息内容之间的间距 */
}

/* 消息内容样式 */
.message-content {
  max-width: 65%;  /* 调小最大宽度 */
  padding: 0.7vw;  /* 调小内边距 */
  border-radius: 0.5vw;  /* 调小圆角 */
  background-color: #fff;
  position: relative;
}

.message.bot .message-content {
  background-color: #e0e0e0;
}

.message.user .message-content {
  background-color: #005C97;
  color: #fff;
}

/* 消息文本样式 */
.message-content p {
  margin: 0;
  padding: 0;
  word-wrap: break-word;
  white-space: normal;
  font-size: 1.1rem;  /* 调小字体大小 */
}

/* 加载效果样式 */
.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 0.8vw;
}

.dot {
  width: 1vw;  /* 调小圆点的大小 */
  height: 1vw;  /* 调小圆点的大小 */
  margin: 0 0.4vw;
  background-color: #999;
  border-radius: 50%;
  animation: bounce 1.4s infinite ease-in-out;
}

.dot:nth-child(2) {
  animation-delay: 0.2s;
}

.dot:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes bounce {
  0%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-1vw);
  }
}

/* 回到顶部按钮 */
.scroll-to-top-btn {
  position: fixed;
  bottom: 15px;  /* 调小按钮与底部的距离 */
  right: 15px;   /* 调小按钮与右侧的距离 */
  background-color: #000;
  color: #fff;
  border: none;
  padding: 0.6vw 1.2vw;  /* 调小按钮的内边距 */
  border-radius: 1.5vw;  /* 调小圆角 */
  font-size: 1.4rem;  /* 调小字体 */
  cursor: pointer;
}

.scroll-to-top-btn:hover {
  background-color: #333;
}

/* 输入框样式 */
.chat-input {
  display: flex;
  align-items: flex-end;
  padding: 0.8vw;  /* 调小内边距 */
  background-color: #fff;
  border-top: 1px solid #ddd;
}

.chat-input textarea {
  flex: 1;
  padding: 0.8vw;  /* 调小内边距 */
  border: 1px solid #ddd;
  border-radius: 1vw;  /* 调小圆角 */
  font-size: 1.3rem;  /* 调小字体大小 */
  resize: none;
  overflow-y: auto;
  max-height: 15vh;  /* 调小最大高度 */
  white-space: pre-wrap;
  outline: none;
}

.chat-input button {
  margin-left: 1vw;
  padding: 0.8vw 1.5vw;  /* 调小按钮的内边距 */
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 1vw;  /* 调小圆角 */
  font-size: 1rem;  /* 调小字体大小 */
  cursor: pointer;
}

/* 响应式字体大小调整 (桌面端) */
@media (min-width: 768px) {
  .chat-container {
    padding: 1vw;  /* 调小容器的 padding */
  }

  .chat-messages {
    padding: 0.8vw;
  }

  .message {
    margin-bottom: 0.6vw;  /* 更加紧凑 */
  }

  /* 调整消息内容的字体大小和 padding */
  .message-content {
    padding: 0.6vw;
    max-width: 60%;  /* 增加消息内容的宽度 */
  }

  /* 消息文本 */
  .message-content p {
    font-size: 1rem;  /* 调小字体 */
  }

  /* 头像大小 */
  .avatar {
    width: 3vw;
    height: 3vw;
  }

  /* 加载圆点 */
  .dot {
    width: 1vw;
    height: 1vw;
  }

  /* 输入框 */
  .chat-input {
    padding: 0.6vw;
  }

  .chat-input textarea {
    font-size: 1rem;  /* 调小字体 */
    padding: 0.6vw;
  }

  .chat-input button {
    padding: 0.6vw 0.8vw;
    font-size: 0.8rem;
  }

  /* 回到顶部按钮 */
  .scroll-to-top-btn {
    font-size: 1.1rem;
    padding: 0.5vw 1.1vw;
  }
}

</style>
