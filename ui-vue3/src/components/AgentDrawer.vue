<!--
  ~ Licensed to the Apache        <template v-if="messages.length === 0">
          <div class="flex items-center justify-center h-full">
            <p class="text-gray-500">
              <template v-if="!currentSessionId"> 请点击"新对话"开始与AI对话 </template>
              <template v-else>
                <span v-if="sessions.length === 0">开始提问吧！</span>
                <span v-else>当前会话暂无消息，快来提问吧！</span>
              </template>
            </p>
          </div>
        </template>oundation (ASF) under one or more
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
    title="问AI"
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
          <div class="flex items-center justify-center h-full">
            <p class="text-gray-500">
              <template v-if="!currentSessionId"> 尚未创建对话，请点击“新对话”开始。 </template>
              <template v-else>
                <span v-if="sessions.length === 0">已创建新会话，快来提问吧！</span>
                <span v-else>当前会话暂无消息，快来提问吧！</span>
              </template>
            </p>
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
                'flex rounded-xl bg-[#0000000f] text-[#000000e0] p-4 sm:max-w-md md:max-w-2xl':
                  msg.role === 'assistant',
                'flex min-h-[85px] rounded-xl bg-[#0000000f] text-[#000000e0] p-4 sm:min-h-0 sm:max-w-md md:max-w-2xl':
                  msg.role === 'user'
              }"
            >
              <p v-if="msg.content" v-html="msg.content.replace(/\n/g, '<br />')"></p>
              <p v-else-if="isLoading && msg.role === 'assistant'" class="flex items-center">
                <LoadingOutlined class="mr-2" /> 正在思考...
              </p>
            </div>
          </div>
        </template>
      </div>
      <!-- Prompt message input -->
      <div class="w-full mt-2">
        <div class="w-full flex flex-row gap-2">
          <a-button class="flex items-center" @click="handleNewChat">
            <PlusOutlined />
            新对话
          </a-button>
          <a-button class="flex items-center" @click="handleViewHistory">
            <ClockCircleOutlined />
            对话历史
          </a-button>
          <a-button class="flex items-center" @click="clearHistory">
            <DeleteOutlined />
            清空历史
          </a-button>
        </div>
        <div class="mt-2">
          <div class="relative">
            <textarea
              id="chat-input"
              v-model="inputMessage"
              @keydown="handleKeyDown"
              rows="2"
              class="block w-full resize-none rounded-xl border-none bg-slate-200 p-4 pl-10 pr-20 text-sm text-slate-900 focus:outline-none focus:ring-2 focus:ring-blue-600 sm:text-base"
              placeholder="输入你的问题..."
              :disabled="isLoading"
              required
            ></textarea>
            <a-button
              shape="circle"
              type="primary"
              :loading="isLoading"
              @click="sendMessage"
              class="absolute top-0 right-0 -translate-x-1/2 translate-y-1/2 flex items-center justify-center"
            >
              <ArrowUpOutlined v-if="!isLoading" />
            </a-button>
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
import axios from 'axios'

import { aiService } from '@/api/service/ai'
import type { ChatMessage, Session } from '@/api/service/ai'

// 定义本地响应式变量
const localDrawerOpen = ref(false)
const messages = ref<ChatMessage[]>([])
const inputMessage = ref('')
const isLoading = ref(false)
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

    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      // 解码并处理数据
      const chunk = decoder.decode(value, { stream: true })
      partialChunk += chunk

      // 处理可能的多行数据
      const lines = partialChunk.split('\n')
      partialChunk = lines.pop() || ''

      for (const line of lines) {
        if (line.trim() === '') continue

        // 解析事件类型和数据
        if (line.startsWith('event: ')) {
          const eventType = line.substring(7).trim()
          // 获取下一行的数据
          const nextLine = lines.find((l, index) => lines.indexOf(line) + 1 === index)
          if (!nextLine || !nextLine.startsWith('data: ')) continue

          const dataStr = nextLine.substring(6)
          try {
            const data = JSON.parse(dataStr)

            // 根据事件类型处理数据
            switch (eventType) {
              case 'message_start':
                console.log('开始新的消息')
                aiMessage.content = ''
                break

              case 'content_block_start':
                console.log('开始新的内容块')
                break

              case 'content_block_delta':
                if (data.delta?.type === 'text_delta' && data.delta?.text) {
                  aiMessage.content += data.delta.text
                  // 强制更新视图
                  messages.value = [...messages.value]
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
                // 可以在这里处理最终的清理工作
                break

              case 'error':
                hasError = true
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

<style></style>
