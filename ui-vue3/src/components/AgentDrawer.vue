<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one or more
  ~ contributor license agreements.  See the NOTICE file distributed with
  ~ this work for additional information regarding copyright ownership.
  ~ The ASF licenses this file to You under the Apache License, Version 2.0
  ~ (the "License"); you may not use this file except in compliance with
  ~ the License.  You may obtain a copy of the License at
  ~
  ~     http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing, software
  ~ distributed under the License is distributed on an "AS IS" BASIS,
  ~ WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  ~ See the License for the specific language governing permissions and
  ~ limitations under the License.
-->
<template>
  <a-drawer
    v-model:open="localDrawerOpen"
    class="custom-class"
    title="Dubbo Admin AI"
    placement="right"
    :width="600"
  >
    <!-- Prompt Messages Container - Modify the height according to your need -->
    <div class="flex w-full flex-col">
      <!-- Prompt Messages -->
      <div
        class="flex-1 space-y-6 overflow-y-auto rounded-xl bg-white p-4 text-sm leading-6 text-slate-900 sm:text-base sm:leading-7 h-[400px]"
      >
        <template v-if="messages.length === 0">
          <div class="flex flex-col items-center justify-center h-full gap-4">
            <h1 class="text-2xl font-bold">Dubbo Admin AI</h1>
            <p class="text-gray-500">
              我是k8m的AI小助手，你可以问我任何关于kubernetes的问题，我尽量给你提供最准确的答案。
            </p>
            <p class="text-lg text-amber-300 font-medium">✨ 奇思妙想和创新的火花</p>
            <div class="grid grid-cols-2 gap-4 w-full max-w-2xl mt-4">
              <div
                class="border rounded-lg p-4 hover:bg-gray-50 cursor-pointer transition-all duration-200 hover:shadow-md"
                @click="handleSuggestionClick('请给我一个基本的nginx 部署yaml如何配置?')"
              >
                <div class="flex items-center gap-2 mb-2">
                  <div class="text-yellow-500">💡</div>
                  <div class="font-medium">yaml编写</div>
                </div>
                <div class="text-gray-500 text-sm">请给我一个基本的nginx 部署yaml如何配置?</div>
              </div>
              <div
                class="border rounded-lg p-4 hover:bg-gray-50 cursor-pointer transition-all duration-200 hover:shadow-md"
                @click="handleSuggestionClick('请解释下Deploy中的HostNetwork如何配置?')"
              >
                <div class="flex items-center gap-2 mb-2">
                  <div class="text-blue-500">ℹ️</div>
                  <div class="font-medium">网络</div>
                </div>
                <div class="text-gray-500 text-sm">请解释下Deploy中的HostNetwork如何配置?</div>
              </div>
              <div
                class="border rounded-lg p-4 hover:bg-gray-50 cursor-pointer transition-all duration-200 hover:shadow-md"
                @click="handleSuggestionClick('请给我一个基本的nginx 部署yaml, 并部署到集群中')"
              >
                <div class="flex items-center gap-2 mb-2">
                  <div class="text-purple-500">🔔</div>
                  <div class="font-medium">自动应用</div>
                </div>
                <div class="text-gray-500 text-sm">
                  请给我一个基本的nginx 部署yaml, 并部署到集群中
                </div>
              </div>
              <div
                class="border rounded-lg p-4 hover:bg-gray-50 cursor-pointer transition-all duration-200 hover:shadow-md"
                @click="handleSuggestionClick('请给我一个基本的nginx 部署yaml, 并保存为模板')"
              >
                <div class="flex items-center gap-2 mb-2">
                  <div class="text-green-500">✅</div>
                  <div class="font-medium">Yaml模板</div>
                </div>
                <div class="text-gray-500 text-sm">
                  请给我一个基本的nginx 部署yaml, 并保存为模板
                </div>
              </div>
            </div>
          </div>
        </template>

        <template v-else>
          <div
            v-for="msg in messages"
            :key="msg.id"
            :class="{
              'flex items-start': msg.role === 'assistant',
              'flex flex-row-reverse items-start': msg.role === 'user'
            }"
          >
            <img
              v-if="msg.role === 'assistant'"
              class="mr-2 h-8 w-8 rounded-full"
              src="https://dummyimage.com/128x128/fde3cf/f56a00&text=AI"
            />
            <img
              v-else
              class="ml-2 h-8 w-8 rounded-full"
              src="https://dummyimage.com/128x128/87d068/ffffff&text=U"
            />

            <div
              :class="{
                'flex flex-col rounded-xl bg-[#0000000f] text-[#000000e0] p-4 max-w-[480px] break-words':
                  msg.role === 'assistant',
                'flex h-fit rounded-xl bg-[#0000000f] text-[#000000e0] p-4 max-w-[480px] break-words':
                  msg.role === 'user'
              }"
            >
              <template v-if="msg.role === 'assistant'">
                <div v-if="msg.content" class="markdown-body" v-html="md.render(msg.content)"></div>
                <div
                  v-if="isAiThinking && msg === messages[messages.length - 1]"
                  class="mt-2 flex items-center text-gray-400 text-sm"
                >
                  <LoadingOutlined class="mr-2" />
                  <span class="animate-pulse">正在思考...</span>
                </div>
              </template>
              <template v-else>
                <p v-html="msg.content.replace(/\n/g, '<br />')"></p>
              </template>
            </div>
          </div>
        </template>
      </div>
      <!-- Prompt message input -->
      <div class="w-full mt-2">
        <div class="w-full flex flex-row gap-2">
          <a-button
            class="flex items-center"
            style="
              background-image: linear-gradient(
                97deg,
                rgb(242, 249, 254) 0%,
                rgb(247, 243, 255) 100%
              );
            "
            @click="handleNewChat"
          >
            <PlusOutlined />
            新对话
          </a-button>
          <a-button
            class="flex items-center"
            style="
              background-image: linear-gradient(
                97deg,
                rgb(242, 249, 254) 0%,
                rgb(247, 243, 255) 100%
              );
            "
            @click="handleViewHistory"
          >
            <ClockCircleOutlined />
            对话历史
          </a-button>
          <a-button
            class="flex items-center"
            style="
              background-image: linear-gradient(
                97deg,
                rgb(242, 249, 254) 0%,
                rgb(247, 243, 255) 100%
              );
            "
            @click="clearHistory"
          >
            <DeleteOutlined />
            清空历史
          </a-button>
        </div>
        <div class="mt-2">
          <div class="relative">
            <div
              class="flex w-full rounded-lg border border-gray-200 bg-white shadow-lg transition-all duration-200 hover:shadow-xl focus-within:border-blue-400 focus-within:ring-4 focus-within:ring-blue-100"
            >
              <textarea
                id="chat-input"
                v-model="inputMessage"
                @keydown="handleKeyDown"
                rows="1"
                class="block w-full resize-none rounded-lg border-0 bg-transparent px-4 py-3 text-gray-900 placeholder:text-gray-400 focus:outline-none focus:ring-0 sm:text-sm transition-colors duration-200 hover:bg-gray-50"
                placeholder="输入你的问题，Shift + Enter 换行"
                :disabled="isLoading"
                required
              ></textarea>
              <div class="flex items-end gap-2 p-2">
                <a-button
                  shape="circle"
                  type="primary"
                  :loading="isLoading"
                  @click="sendMessage"
                  class="flex h-8 w-8 items-center justify-center"
                >
                  <ArrowUpOutlined v-if="!isLoading" />
                </a-button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </a-drawer>

  <!-- 对话历史Modal -->
  <a-modal v-model:visible="historyModalVisible" title="对话历史" :footer="null" width="600px">
    <div class="max-h-[400px] overflow-y-auto">
      <a-empty v-if="sessions.length === 0" description="暂无对话历史" />
      <a-list v-else>
        <a-list-item
          v-for="session in sessions"
          :key="session.session_id"
          class="cursor-pointer hover:bg-gray-100 rounded p-2"
        >
          <div class="flex justify-between w-full" @click="loadSession(session.session_id)">
            <div>
              <div class="font-medium">
                会话 #{{ session.session_id ? session.session_id.substring(0, 8) : '' }}
              </div>
              <div class="text-gray-500 text-sm">
                {{ session.message_count ? `消息数: ${session.message_count}` : '新会话' }}
              </div>
            </div>
            <div>
              <div class="text-gray-500 text-sm">
                {{ new Date(session.created_at).toLocaleString() }}
              </div>
              <a-button
                type="link"
                size="small"
                @click.stop="deleteSession(session.session_id)"
                danger
              >
                <DeleteOutlined /> 删除
              </a-button>
            </div>
          </div>
        </a-list-item>
      </a-list>
    </div>
  </a-modal>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import {
  ClockCircleOutlined,
  DeleteOutlined,
  PlusOutlined,
  ArrowUpOutlined,
  LoadingOutlined
} from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'
import { aiService } from '@/api/service/ai'
import type { ChatMessage, Session } from '@/api/service/ai'
import MarkdownIt from 'markdown-it'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css' // 使用 GitHub 风格的代码高亮主题

// 初始化 markdown 解析器
const md: MarkdownIt = new MarkdownIt({
  html: true, // 启用 HTML 标签
  breaks: true, // 转换 \n 为 <br>
  linkify: true, // 自动转换 URL 为链接
  typographer: true, // 启用一些语言中性的替换 + 引号美化
  highlight: function (str: string, lang: string): string {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return (
          '<pre class="code-block"><code class="hljs language-' +
          lang +
          '">' +
          hljs.highlight(str, { language: lang, ignoreIllegals: true }).value +
          '</code></pre>'
        )
      } catch (error) {
        console.warn('Failed to highlight code block:', error)
      }
    }
    return (
      '<pre class="code-block"><code class="hljs">' + md.utils.escapeHtml(str) + '</code></pre>'
    )
  }
})

// 定义本地响应式变量
const localDrawerOpen = ref(false)
const messages = ref<ChatMessage[]>([])
const inputMessage = ref('')
const isLoading = ref(false)
const isAiThinking = ref(false) // AI是否正在思考（用于显示思考中的动画）
const currentSessionId = ref('')
const sessions = ref<Session[]>([])
const historyModalVisible = ref(false)

// 使用Vue 3的<script setup>编译器宏定义props和emits
const props = defineProps({
  agentDrawerOpen: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['update:agentDrawerOpen'])

// 初始化本地变量
localDrawerOpen.value = props.agentDrawerOpen

// 监听 prop 变化同步到本地变量
watch(
  () => props.agentDrawerOpen,
  (newVal) => {
    localDrawerOpen.value = newVal
  }
)

// 监听本地变量变化并触发事件通知父组件
watch(localDrawerOpen, (newVal) => {
  emit('update:agentDrawerOpen', newVal)
})

// 创建新会话
async function createNewSession() {
  try {
    const sessionId = await aiService.createSession()
    currentSessionId.value = sessionId
    messages.value = []
    return sessionId
  } catch (error) {
    message.error('创建会话失败')
    console.error('创建会话失败:', error)
    return ''
  }
}

// 获取会话列表
async function fetchSessions() {
  try {
    sessions.value = await aiService.getSessions()
  } catch (error) {
    console.error('获取会话列表失败:', error)
  }
}

// 获取特定会话信息
async function fetchSessionInfo(sessionId: string) {
  try {
    const response = await aiService.getSessionInfo(sessionId)
    return response.data
  } catch (error) {
    console.error('获取会话信息失败:', error)
    return null
  }
}

// 删除会话
async function deleteSession(sessionId: string) {
  try {
    await aiService.deleteSession(sessionId)
    if (currentSessionId.value === sessionId) {
      currentSessionId.value = ''
      messages.value = []
    }
    await fetchSessions()
    message.success('会话已删除')
  } catch (error) {
    message.error('删除会话失败')
    console.error('删除会话失败:', error)
  }
}

// 发送消息并接收流式响应
async function sendMessage() {
  if (!inputMessage.value.trim() || isLoading.value) return

  // 如果没有当前会话ID，先创建一个新会话
  if (!currentSessionId.value) {
    const sessionId = await createNewSession()
    if (!sessionId) {
      message.error('无法创建会话，请稍后再试')
      return
    }
  }

  const userMessage: ChatMessage = {
    id: Date.now().toString(),
    content: inputMessage.value,
    role: 'user',
    timestamp: Date.now()
  }

  messages.value.push(userMessage)

  const aiMessage: ChatMessage = {
    id: (Date.now() + 1).toString(),
    content: '',
    role: 'assistant',
    timestamp: Date.now() + 1
  }

  messages.value.push(aiMessage)
  isLoading.value = true
  inputMessage.value = ''

  try {
    // 发送消息并获取流式响应
    const stream = await aiService.sendChatMessage(userMessage.content, currentSessionId.value)

    // 处理SSE流
    const reader = stream.getReader()
    if (!reader) throw new Error('无法读取响应流')

    const decoder = new TextDecoder()
    let partialChunk = ''
    let hasError = false

    // eslint-disable-next-line no-constant-condition
    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      // 解码并处理数据
      const chunk = decoder.decode(value, { stream: true })
      partialChunk += chunk

      // 处理可能的多行数据
      let lines = partialChunk.split('\n')

      // 如果最后一行不完整，保存到partialChunk中
      if (!chunk.endsWith('\n')) {
        partialChunk = lines.pop() || ''
      } else {
        partialChunk = ''
      }

      // 处理每一对事件和数据
      for (let i = 0; i < lines.length - 1; i++) {
        const line = lines[i]
        const nextLine = lines[i + 1]

        if (line.trim() === '') continue

        // 解析事件类型和数据
        if (line.startsWith('event: ')) {
          const eventType = line.substring(7).trim()

          // 确保下一行是数据行
          if (!nextLine.startsWith('data: ')) continue

          // 跳过已处理的数据行
          i++

          const dataStr = nextLine.substring(6)
          try {
            const data = JSON.parse(dataStr)

            // 根据事件类型处理数据
            switch (eventType) {
              case 'message_start':
                console.log('开始新的消息')
                aiMessage.content = ''
                isAiThinking.value = true
                break

              case 'content_block_start':
                console.log('开始新的内容块')
                break

              case 'content_block_delta':
                if (data.delta?.type === 'text_delta' && data.delta?.text) {
                  // 将新文本添加到消息内容中
                  aiMessage.content += data.delta.text
                  // 创建新的消息数组以触发响应式更新
                  const updatedMessages = [...messages.value]
                  // 更新最后一条消息
                  updatedMessages[updatedMessages.length - 1] = { ...aiMessage }
                  // 更新消息列表
                  messages.value = updatedMessages
                }
                break

              case 'content_block_stop':
                console.log('内容块结束')
                break

              case 'message_delta':
                // 处理消息更新，比如处理建议的动作等
                if (data.delta?.suggested_actions) {
                  console.log('收到建议动作:', data.delta.suggested_actions)
                }
                break

              case 'message_stop':
                console.log('消息结束')
                // 标记AI不再思考
                isAiThinking.value = false
                break

              case 'error':
                hasError = true
                isAiThinking.value = false
                console.error('SSE 流错误:', data.error)
                if (data.error?.message) {
                  message.error(`发生错误: ${data.error.message}`)
                  aiMessage.content = `错误: ${data.error.message}`
                }
                break

              default:
                console.log(`收到事件: ${eventType}`, data)
            }
          } catch (e) {
            console.error(`解析 ${eventType} 事件数据失败:`, e, dataStr)
          }
        }
      }
    }

    // 如果处理过程中出现错误，添加错误提示
    if (hasError && !aiMessage.content) {
      aiMessage.content = '抱歉，处理消息时发生错误，请稍后再试。'
    }
  } catch (error) {
    message.error('发送消息失败')
    console.error('发送消息失败:', error)
    // 更新AI消息为错误信息
    aiMessage.content = '抱歉，发生了错误，请稍后再试。'
  } finally {
    isLoading.value = false
    isAiThinking.value = false
  }
}

// 清空历史消息
function clearHistory() {
  messages.value = []
  message.success('历史记录已清空')
}

// 处理新对话按钮点击
async function handleNewChat() {
  await createNewSession()
  message.success('已创建新对话')
}

// 处理对话历史按钮点击
async function handleViewHistory() {
  await fetchSessions()
  historyModalVisible.value = true
}

// 处理键盘事件
function handleKeyDown(event: KeyboardEvent) {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    sendMessage()
  }
}

// 处理建议问题的点击
function handleSuggestionClick(suggestion: string) {
  inputMessage.value = suggestion
  // 自动聚焦到输入框
  setTimeout(() => {
    const inputElement = document.getElementById('chat-input')
    if (inputElement) {
      inputElement.focus()
    }
  }, 0)
}

onMounted(() => {
  // 移除自动创建会话的逻辑，改为用户主动点击创建
})

// 加载选定的会话
async function loadSession(sessionId: string) {
  try {
    isLoading.value = true
    const sessionInfo = await fetchSessionInfo(sessionId)
    if (sessionInfo) {
      currentSessionId.value = sessionId
      messages.value = sessionInfo.messages || []
      historyModalVisible.value = false
    }
  } catch (error) {
    console.error('加载会话失败:', error)
    message.error('加载会话失败')
  } finally {
    isLoading.value = false
  }
}
</script>

<style>
.markdown-body {
  font-family:
    -apple-system,
    BlinkMacSystemFont,
    Segoe UI,
    Helvetica,
    Arial,
    sans-serif;
  font-size: 14px;
  line-height: 1.6;
  word-wrap: break-word;
  width: 100%;
  max-width: 100%;
  overflow-x: auto;
  color: #24292e;
}

.markdown-body > *:first-child {
  margin-top: 0 !important;
}

.markdown-body > *:last-child {
  margin-bottom: 0 !important;
}

.markdown-body .code-block {
  margin: 0;
  padding: 16px;
  overflow: auto;
  font-size: 85%;
  line-height: 1.45;
  background-color: #f6f8fa;
  border-radius: 6px;
}

.markdown-body .hljs {
  background: transparent;
  padding: 0;
}

.markdown-body a {
  color: #0366d6;
  text-decoration: none;
}

.markdown-body a:hover {
  text-decoration: underline;
}

.markdown-body hr {
  height: 0.25em;
  padding: 0;
  margin: 24px 0;
  background-color: #e1e4e8;
  border: 0;
}

.markdown-body blockquote {
  padding: 0 1em;
  color: #6a737d;
  border-left: 0.25em solid #dfe2e5;
  margin: 0 0 16px 0;
}

.markdown-body ul,
.markdown-body ol {
  padding-left: 2em;
  margin-top: 0;
  margin-bottom: 16px;
}

.markdown-body img {
  max-width: 100%;
  box-sizing: content-box;
  background-color: #fff;
  border-radius: 3px;
}

.markdown-body pre {
  background-color: #f6f8fa;
  border-radius: 6px;
  padding: 16px;
  overflow-x: auto;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.markdown-body code {
  background-color: rgba(175, 184, 193, 0.2);
  border-radius: 6px;
  padding: 0.2em 0.4em;
  font-family:
    ui-monospace,
    SFMono-Regular,
    SF Mono,
    Menlo,
    Consolas,
    Liberation Mono,
    monospace;
}

.markdown-body pre code {
  background-color: transparent;
  padding: 0;
}

.markdown-body h1,
.markdown-body h2,
.markdown-body h3,
.markdown-body h4,
.markdown-body h5,
.markdown-body h6 {
  margin-top: 24px;
  margin-bottom: 16px;
  font-weight: 600;
  line-height: 1.25;
}

.markdown-body h1 {
  font-size: 2em;
}
.markdown-body h2 {
  font-size: 1.5em;
}
.markdown-body h3 {
  font-size: 1.25em;
}
.markdown-body h4 {
  font-size: 1em;
}

.markdown-body ul,
.markdown-body ol {
  padding-left: 2em;
  margin-top: 0;
  margin-bottom: 16px;
}

.markdown-body blockquote {
  padding: 0 1em;
  color: #57606a;
  border-left: 0.25em solid #d0d7de;
  margin: 0 0 16px;
}

.markdown-body table {
  display: block;
  width: 100%;
  width: max-content;
  max-width: 100%;
  overflow: auto;
  margin-top: 0;
  margin-bottom: 16px;
  border-spacing: 0;
  border-collapse: collapse;
}

.markdown-body table th,
.markdown-body table td {
  padding: 6px 13px;
  border: 1px solid #d0d7de;
}

.markdown-body table tr {
  background-color: #ffffff;
  border-top: 1px solid #d0d7de;
}

.markdown-body table tr:nth-child(2n) {
  background-color: #f6f8fa;
}

.markdown-body img {
  max-width: 100%;
  box-sizing: content-box;
  background-color: #ffffff;
}

.markdown-body p {
  margin-top: 0;
  margin-bottom: 16px;
}
</style>
