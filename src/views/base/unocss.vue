<template>
  <CommonPage show-footer title="AI数据分析助手">
    <div class="layout">
      <!-- 会话菜单 -->
      <div class="menu">
        <!-- 添加会话按钮 -->
        <a-button @click="onAddConversation" type="link" class="addBtn">
          <template #icon>
            <PlusOutlined />
          </template>
          新建会话
        </a-button>

        <!-- 会话列表 -->
        <div class="conversation-list">
          <div v-for="item in paginatedConversations" :key="item.key"
            :class="['conversation-item', currentSession === item.key ? 'active' : '']"
            @click="() => selectSession(item.key)">
            <div class="conversation-info">
              <span class="conversation-title">{{ item.label }}</span>
              <div class="conversation-details">
                <span class="update-time">{{ formatDate(item.update_time) }}</span>
              </div>
            </div>
            <a @click.stop="(e) => confirmDeleteSession(item.key, e)" class="delete-link">
              <DeleteOutlined />
            </a>
          </div>
        </div>

        <!-- 分页控制 - 移动到底部 -->
        <div class="pagination-controls" v-if="conversationsItems.length > pageSize" style="text-align: center;">
          <a-pagination v-model:current="currentPage" :total="conversationsItems.length" :pageSize="pageSize"
            size="small" @change="handlePageChange" simple />
        </div>
      </div>

      <div class="chat">
        <!-- 消息列表 -->
        <div class="messages" ref="messagesContainer">
          <template v-if="items.length > 0">
            <div v-for="item in items" :key="item.key" :class="['message', item.role, { loading: item.loading }]">
              <div class="message-wrapper">
                <img v-if="item.role === 'ai'" class="avatar"
                  src="https://mdn.alipayobjects.com/huamei_iwk9zp/afts/img/A*s5sNRo5LjfQAAAAAAAAAAAAADgCCAQ/fmt.webp"
                  alt="AI Avatar" />
                <div class="message-box">
                  <div class="message-content markdown-body">
                    <a-spin v-if="item.loading && !item.workflow" />
                    <div v-if="item.workflow" class="workflow-process">
                      <a-collapse :defaultActiveKey="['1']">
                        <a-collapse-panel key="1">
                          <template #header>
                            <span>
                              <loading-outlined v-if="item.workflow.status === 'running'" spin />
                              <check-circle-outlined v-else-if="item.workflow.status === 'succeeded'"
                                style="color: #52c41a" />
                              <close-circle-outlined v-else-if="item.workflow.status === 'failed'"
                                style="color: #ff4d4f" />
                              <stop-outlined v-else-if="item.workflow.status === 'stopped'" style="color: #faad14" />
                              工作流程展示
                            </span>
                          </template>
                          <div class="workflow-steps">
                            <!-- 主流程步骤 -->
                            <template v-for="step in item.workflow.steps" :key="step.id">
                              <!-- 普通步骤 -->
                              <div v-if="!step.parallel_id" class="workflow-step">
                                <span class="step-icon">
                                  <loading-outlined v-if="step.status === 'running'" spin />
                                  <check-circle-outlined v-else-if="step.status === 'succeeded'" style="color: #52c41a" />
                                  <close-circle-outlined v-else-if="step.status === 'failed'" style="color: #ff4d4f" />
                                  <stop-outlined v-else-if="step.status === 'stopped'" style="color: #faad14" />
                                </span>
                                <span class="step-title">{{ step.title }}</span>
                                <span class="step-time" v-if="step.elapsed_time">{{ step.elapsed_time > 1000 ? (step.elapsed_time / 1000).toFixed(2) + 's' : step.elapsed_time.toFixed(2) + 'ms' }}</span>
                              </div>
                              <!-- 并行分支 -->
                              <div v-else-if="isParallelStartNode(step, item.workflow.steps)" class="workflow-parallel">
                                <div class="parallel-header">并行分支</div>
                                <div class="parallel-branches">
                                  <div v-for="branch in getParallelBranches(step.parallel_id, item.workflow.steps)"
                                    :key="branch.id" class="parallel-branch">
                                    <div class="branch-header">分支 {{ branch.branch_index + 1 }}</div>
                                    <div v-for="branchStep in branch.steps" :key="branchStep.id"
                                      class="workflow-step branch-step">
                                      <span class="step-icon">
                                        <loading-outlined v-if="branchStep.status === 'running'" spin />
                                        <check-circle-outlined v-else-if="branchStep.status === 'succeeded'"
                                          style="color: #52c41a" />
                                        <close-circle-outlined v-else-if="branchStep.status === 'failed'"
                                          style="color: #ff4d4f" />
                                        <stop-outlined v-else-if="branchStep.status === 'stopped'"
                                          style="color: #faad14" />
                                      </span>
                                      <span class="step-title">{{ branchStep.title }}</span>
                                      <span class="step-time" v-if="branchStep.elapsed_time">
                                        {{ branchStep.elapsed_time > 1000 ? (branchStep.elapsed_time / 1000).toFixed(2) + 's' : branchStep.elapsed_time.toFixed(2) + 'ms' }}
                                      </span>
                                    </div>
                                  </div>
                                </div>
                              </div>
                            </template>
                          </div>
                        </a-collapse-panel>
                      </a-collapse>
                    </div>
                    <template v-if="!item.loading || (item.workflow && item.workflow.status !== 'running')">
                      <div>
                        <div class="message-content-inner" v-html="renderMarkdown(item.content)"></div>
                        <div v-if="item.role === 'ai' && !isFirstAIMessage(item)"
                          class="message-actions">
                          <div style="font-size: smaller; color: #454545">此条回答对您是否有帮助？</div>
                          <a @click="likeMessage(item.key)" :class="{ 'active': item.liked }">
                            <HeartTwoTone :twoToneColor="item.liked ? '#eb2f96' : '#000000'" />
                          </a>
                          <a @click="() => dislikeMessage(item.key, item.content)" :class="{ 'active': item.disliked }">
                            <FrownOutlined :style="{ color: item.disliked ? '#1890ff' : '#000000' }" />
                          </a>
                        </div>
                      </div>
                    </template>
                  </div>
                </div>
              </div>
            </div>
          </template>
          <template v-else>
            <!-- 欢迎页面 -->
            <div class="placeholder">
              <a-card class="welcome-card" :bordered="false">
                <template #cover>
                  <div class="welcome-header">
                    <img class="avatar"
                      src="https://mdn.alipayobjects.com/huamei_iwk9zp/afts/img/A*s5sNRo5LjfQAAAAAAAAAAAAADgCCAQ/fmt.webp" />
                    <div class="welcome-info">
                      <h3>欢迎使用 AI 助手</h3>
                      <p>有什么可以帮忙的？</p>
                    </div>
                  </div>
                </template>
              </a-card>

              <!-- 当没有会话时，显示中央输入框 -->
              <div v-if="!currentSession" class="center-input-container">
                <div class="input-wrapper">
                  <a-input v-model:value="content" :disabled="isMessageSending" @pressEnter="() => onSubmit(content)"
                    placeholder="请输入问题..." class="chat-input" />
                  <a-button type="primary" :disabled="isMessageSending" @click="() => onSubmit(content)"
                    class="send-btn">
                    <template #icon>
                      <SendOutlined />
                    </template>
                  </a-button>
                </div>
              </div>
            </div>
          </template>
        </div>

        <div v-if="currentSession" class="sender">
          <div class="input-wrapper">
            <a-input v-model:value="content" :disabled="isMessageSending" @pressEnter="() => onSubmit(content)"
              placeholder="请输入问题..." class="chat-input" />
            <a-button type="primary" :disabled="isMessageSending" @click="() => onSubmit(content)" class="send-btn">
              <template #icon>
                <SendOutlined />
              </template>
            </a-button>
          </div>
        </div>
        <div class="ai-disclaimer">
          AI也可能会犯错。请核查重要信息。
        </div>
        <div v-if="feedbackMessage" class="feedback-message">{{ feedbackMessage }}</div>
      </div>
    </div>
  </CommonPage>
</template>

<script setup>
import MarkdownIt from 'markdown-it'
import markdownItCharts from 'markdown-it-charts'
import markdownItMultimdTable from 'markdown-it-multimd-table'
import markdownItContainer from 'markdown-it-container'
import markdownItKatex from 'markdown-it-katex'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css'
import 'katex/dist/katex.min.css'
import * as echarts from 'echarts'
import { ref, watch, nextTick, onMounted, computed, onUnmounted } from 'vue'
import {
  PlusOutlined,
  SendOutlined,
  DeleteOutlined,
  FrownOutlined,
  HeartTwoTone,
  LoadingOutlined,
  CheckCircleOutlined,
  CloseCircleOutlined,
  StopOutlined,
} from '@ant-design/icons-vue'
import { message, Modal } from 'ant-design-vue'
import { createChatApiClient } from './api'
import { dictStore } from '@/store/dict'
import { DITC_TYPE } from '@/constants'

// 状态管理
const content = ref('')
const activeKey = ref('0')
const selectedKeys = ref([activeKey.value])
const conversationsItems = ref([])
const feedbackMessage = ref('')
const feedbackKey = ref('')

// 分页相关
const currentPage = ref(1)
const pageSize = ref(10)
const paginatedConversations = computed(() => {
  const totalItems = conversationsItems.value.length
  const totalPages = Math.max(1, Math.ceil(totalItems / pageSize.value))
  
  // 确保当前页不超出范围
  if (currentPage.value > totalPages) {
    currentPage.value = totalPages
  }
  
  const startIndex = (currentPage.value - 1) * pageSize.value
  const endIndex = Math.min(startIndex + pageSize.value, totalItems)
  
  return conversationsItems.value.slice(startIndex, endIndex)
})

// 会话管理
const currentSession = ref('')
const loading = ref(false)
const isMessageSending = ref(false)
const currentStreamController = ref(null)

// 消息列表
const messages = ref([])
const items = ref([])
const messagesContainer = ref(null)

// 工作流相关状态
const currentWorkflow = ref(null)
const workflowSteps = ref([])

// 初始化 API 客户端
const store = dictStore()
const { apiKey, apiBaseUrl } = computed(() => {
  const dict = store.getDictByType(DITC_TYPE.AI_DATA_ANALYSIS_BOT_KEY) || []
  return {
    apiKey: dict.find(item => item.label === 'aiDataAnalysisBot')?.value || '',
    apiBaseUrl: dict.find(item => item.label === 'aiDataAnalysisBotUrl')?.value || ''
  }
}).value

const chatApi = createChatApiClient(apiBaseUrl, apiKey)

// 创建 markdown-it 实例并配置插件
const md = new MarkdownIt({
  html: true,
  breaks: true,
  linkify: true,
  highlight: function (str, lang) {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return '<pre class="hljs"><code>' +
          hljs.highlight(str, { language: lang, ignoreIllegals: true }).value +
          '</code></pre>'
      } catch (__) { }
    }
    return '<pre class="hljs"><code>' + md.utils.escapeHtml(str) + '</code></pre>'
  }
})
  .use(markdownItKatex, {
    throwOnError: false,
    errorColor: '#cc0000'
  })
  .use(markdownItCharts, {
    echarts: echarts,
    useCache: true,
    container: {
      style: 'min-height: 400px; width: 100%; margin: 1em 0;'
    }
  })
  .use(markdownItMultimdTable, {
    multiline: true,
    rowspan: true,
    headerless: true
  })
  .use(markdownItContainer, 'info', {
    validate: function (params) {
      return params.trim().match(/^info\s+(.*)$/)
    },
    render: function (tokens, idx) {
      if (tokens[idx].nesting === 1) {
        const m = tokens[idx].info.trim().match(/^info\s+(.*)$/)
        return '<div class="info-container"><p class="info-title">' + md.utils.escapeHtml(m[1]) + '</p>\n'
      } else {
        return '</div>\n'
      }
    }
  })
  .use(markdownItContainer, 'warning', {
    validate: function (params) {
      return params.trim().match(/^warning\s+(.*)$/)
    },
    render: function (tokens, idx) {
      if (tokens[idx].nesting === 1) {
        const m = tokens[idx].info.trim().match(/^warning\s+(.*)$/)
        return '<div class="warning-container"><p class="warning-title">' + md.utils.escapeHtml(m[1]) + '</p>\n'
      } else {
        return '</div>\n'
      }
    }
  })

// 滚动到最底部的函数
const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

// 初始化 ECharts 图表
const initCharts = () => {
  const chartContainers = document.querySelectorAll('.echarts')
  chartContainers.forEach((container, index) => {
    // 如果已经初始化过，跳过
    if (container._echarts_instance_) {
      return
    }

    const dataElement = document.querySelectorAll('.echarts-data')[index]
    if (dataElement) {
      try {
        // 先设置容器高度
        container.style.height = '400px'
        container.style.width = '100%'

        const chartData = JSON.parse(dataElement.textContent)
        // 确保容器有尺寸后再初始化
        const chart = echarts.init(container)
        chart.setOption(chartData)

        // 添加响应式调整
        const resizeHandler = () => {
          chart.resize()
        }
        window.addEventListener('resize', resizeHandler)

        // 清理之前的事件监听器
        const cleanup = () => {
          window.removeEventListener('resize', resizeHandler)
          chart.dispose()
        }

        // 存储清理函数，以便在组件卸载时调用
        container._cleanup = cleanup
      } catch (error) {
        console.error('图表初始化失败:', error)
      }
    }
  })
}

// 在组件卸载时清理
onUnmounted(() => {
  const chartContainers = document.querySelectorAll('.echarts')
  chartContainers.forEach(container => {
    if (container._cleanup) {
      container._cleanup()
    }
  })
})

// 修改 renderMarkdown 函数
const renderMarkdown = (text) => {
  if (!text) return ''
  try {
    const rendered = md.render(text)
    // 在下一个 tick 初始化图表
    nextTick(() => {
      initCharts()
    })
    return rendered
  } catch (error) {
    console.error('Markdown 渲染错误:', error)
    return text
  }
}

// 判断是否为第一条 AI 消息
const isFirstAIMessage = (message) => {
  const aiMessages = messages.value.filter(m => m.role === 'ai')
  return aiMessages.length > 0 && aiMessages[0].key === message.key
}

// 不喜欢消息
const dislikeMessage = async (key, content) => {
  const messageIndex = messages.value.findIndex(m => m.key === key)
  if (messageIndex !== -1 && messages.value[messageIndex].id) {
    try {
      // 确定要发送的反馈类型
      let feedbackType = 'dislike';

      // 如果已经点踩，再次点击则取消点踩
      if (messages.value[messageIndex].disliked) {
        feedbackType = null;
      }

      // 使用真实的消息ID而不是前端生成的key
      await chatApi.sendFeedback(messages.value[messageIndex].id, feedbackType, 'abc-123')

      // 更新本地状态
      if (feedbackType === null) {
        // 取消点踩
        messages.value[messageIndex].disliked = false;
      } else {
        // 设置点踩，取消点赞
        messages.value[messageIndex].disliked = true;
        messages.value[messageIndex].liked = false;
      }

      messages.value = [...messages.value]

      // 显示反馈消息
      feedbackMessage.value = '感谢您的反馈！'
      feedbackKey.value = key

      // 3秒后清除反馈消息
      setTimeout(() => {
        feedbackMessage.value = ''
      }, 3000)

    } catch (error) {
      console.error('发送反馈失败:', error)
      message.error('发送反馈失败')
    }
  } else {
    console.error('找不到消息ID，无法发送反馈')
    message.error('无法发送反馈，消息ID不存在')
  }
}

// 格式化日期
const formatDate = (timestamp) => {
  if (!timestamp) return '';

  // 处理数字时间戳
  let date;
  if (typeof timestamp === 'number' || !isNaN(Number(timestamp))) {
    // 如果是秒级时间戳，转换为毫秒级
    const numTimestamp = Number(timestamp);
    // 判断是秒级还是毫秒级时间戳（通常秒级时间戳长度为10位）
    date = new Date(numTimestamp < 10000000000 ? numTimestamp * 1000 : numTimestamp);
  } else {
    // 处理日期字符串
    date = new Date(timestamp);
  }

  // 检查日期是否有效
  if (isNaN(date.getTime())) {
    console.error('无效的日期格式:', timestamp);
    return '';
  }

  const now = new Date();
  const diffMs = now - date;
  const diffSec = Math.floor(diffMs / 1000);
  const diffMin = Math.floor(diffSec / 60);
  const diffHour = Math.floor(diffMin / 60);
  const diffDay = Math.floor(diffHour / 24);

  // 今天内的时间显示为"HH:MM"
  if (diffDay === 0) {
    return date.toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit' });
  }

  // 昨天的显示为"昨天"
  if (diffDay === 1) {
    return '昨天';
  }

  // 一周内的显示为星期几
  if (diffDay < 7) {
    const weekdays = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
    return weekdays[date.getDay()];
  }

  // 超过一周的显示为"YYYY-MM-DD"
  return date.toLocaleDateString('zh-CN', { year: 'numeric', month: '2-digit', day: '2-digit' }).replace(/\//g, '-');
}

// 处理分页变化
const handlePageChange = (page) => {
  currentPage.value = page
}

// 处理工作流事件
const handleWorkflowEvent = (data) => {
  // 使用 workflow_run_id 来查找对应的消息
  const messageIndex = messages.value.findIndex(m => 
    (m.workflow && m.workflow.id === data.workflow_run_id) || 
    (!m.workflow && m.loading)
  )
  
  if (messageIndex === -1) return

  if (!messages.value[messageIndex].workflow) {
    messages.value[messageIndex].workflow = {
      id: data.workflow_run_id,
      status: 'running',
      steps: [],
      branches: new Map() // 添加分支信息存储
    }
  }

  const workflow = messages.value[messageIndex].workflow  
  console.log('Workflow event:', data.event, data)  // 添加日志

  switch (data.event) {
    case 'workflow_started':
      workflow.status = 'running'
      workflow.id = data.workflow_run_id
      messages.value[messageIndex].loading = true
      break

    case 'parallel_branch_started':
      if (!workflow.branches.has(data.data.parallel_id)) {
        workflow.branches.set(data.data.parallel_id, new Set())
      }
      workflow.branches.get(data.data.parallel_id).add(data.data.parallel_branch_id)
      break

    case 'node_started':
      workflow.steps.push({
        id: data.data.id,
        title: data.data.title,
        status: 'running',
        node_type: data.data.node_type,
        start_time: data.data.created_at,
        parallel_id: data.data.parallel_id,
        parallel_start_node_id: data.data.parallel_start_node_id,
        parent_parallel_id: data.data.parent_parallel_id,
        parent_parallel_start_node_id: data.data.parent_parallel_start_node_id
      })
      break

    case 'node_finished':
      const stepIndex = workflow.steps.findIndex(s => s.id === data.data.id)
      if (stepIndex !== -1) {
        workflow.steps[stepIndex] = {
          ...workflow.steps[stepIndex],
          status: data.data.status,
          elapsed_time: data.data.elapsed_time * 1000,
          error: data.data.error
        }
      }
      break

    case 'parallel_branch_finished':
      // 可以在这里处理分支完成的逻辑
      break

    case 'workflow_finished':
      workflow.status = data.data.status
      workflow.elapsed_time = data.data.elapsed_time
      workflow.error = data.data.error
      if (data.data.status === 'succeeded') {
        messages.value[messageIndex].loading = false
      }
      break
  }

  messages.value = [...messages.value]
  items.value = [...messages.value]
}

// 判断是否为并行分支的起始节点
const isParallelStartNode = (step, allSteps) => {
  if (!step.parallel_id) return false
  
  // 检查是否是该并行分支中的第一个节点
  const parallelSteps = allSteps.filter(s => s.parallel_id === step.parallel_id)
  const firstStepInParallel = parallelSteps.reduce((first, current) => 
    (!first || current.index < first.index) ? current : first
  , null)
  
  return firstStepInParallel && firstStepInParallel.id === step.id
}

// 获取特定并行分支 ID 下的所有分支
const getParallelBranches = (parallelId, steps) => {
  if (!parallelId) return []

  // 获取属于这个并行分支的所有步骤
  const parallelSteps = steps.filter(step => step.parallel_id === parallelId)
  
  // 获取所有唯一的分支ID
  const branchIds = [...new Set(parallelSteps.map(step => step.parallel_start_node_id))]
  
  // 按照分支分组并排序
  return branchIds.map((branchId, index) => {
    const branchSteps = parallelSteps.filter(step => step.parallel_start_node_id === branchId)
    return {
      id: branchId,
      branch_index: index,
      steps: branchSteps.sort((a, b) => {
        // 首先按照索引排序
        if (a.index !== b.index) {
          return a.index - b.index
        }
        // 如果索引相同，按照创建时间排序
        return (a.start_time || 0) - (b.start_time || 0)
      })
    }
  })
}

// 修改 sendStreamingChatMessage 中的 onMessage 处理
const onSubmit = async (nextContent) => {
  if (!nextContent || isMessageSending.value) return

  isMessageSending.value = true

  // 获取当前会话中的消息数量作为索引
  const currentMessageIndex = messages.value.length

  // 添加用户消息
  const newMessage = {
    key: `msg_${currentMessageIndex + 1}`,
    content: nextContent,
    role: 'local'
  }
  messages.value.push(newMessage)

  // 添加 AI 消息（带 loading 状态）
  const aiMessage = {
    key: `msg_${currentMessageIndex + 2}`,
    role: 'ai',
    content: '',
    loading: true,
    workflow: null // 初始化workflow为null
  }
  messages.value.push(aiMessage)
  
  // 同步更新 items 数组
  items.value = [...messages.value]

  // 清空输入框内容 - 确保在添加消息后立即清空
  content.value = ''

  // 滚动到最底部
  setTimeout(scrollToBottom)

  try {
    const controller = await chatApi.sendStreamingChatMessage(
      {
        query: nextContent,
        user: 'abc-123',
        conversation_id: currentSession.value || '',
        auto_generate_name: !currentSession.value
      },
      async (data) => {
        console.log('Received event:', data.event, data)  // 添加日志
        
        if ([
          'workflow_started',
          'node_started',
          'node_finished',
          'workflow_finished',
          'parallel_branch_started',
          'parallel_branch_finished'
        ].includes(data.event)) {
          handleWorkflowEvent(data)
          return
        }

        // 处理消息事件
        if (data.event === 'message') {
          const messageIndex = messages.value.findIndex(m => m.loading || m.key === `msg_${currentMessageIndex + 2}`)
          if (messageIndex !== -1) {
            const currentContent = messages.value[messageIndex].content || '';
            const newContent = currentContent + (data.answer || '');
            
            messages.value[messageIndex] = {
              ...messages.value[messageIndex],
              content: newContent,
              // 移除对 workflow 的依赖，单独处理消息的 loading 状态
              loading: false,
              id: data.id || messages.value[messageIndex].id
            }
            items.value = [...messages.value]
            nextTick(() => {
              scrollToBottom()
            })
          }
        } else if (data.event === 'message_end') {
          // 如果是新会话，保存会话ID并自动重命名
          if (!currentSession.value && data.conversation_id) {
            currentSession.value = data.conversation_id
            // 使用第一条用户消息作为会话名称的一部分
            const truncatedQuery = nextContent.length > 20 ? nextContent.substring(0, 20) + '...' : nextContent
            await chatApi.renameConversation(data.conversation_id, {
              name: truncatedQuery,
              user: 'abc-123'
            })
            // 刷新会话列表以显示新名称
            await fetchConversations()
          }
        }
      },
      (error) => {
        console.error('Chat error:', error)
        message.error('发送消息失败，请重试')

        // 更新错误状态
        const messageIndex = messages.value.findIndex(m => m.loading)
        if (messageIndex !== -1) {
          messages.value[messageIndex] = {
            ...messages.value[messageIndex],
            content: '发送消息失败，请重试',
            loading: false
          }
          items.value = [...messages.value]
        }
      },
      () => {
        isMessageSending.value = false
      }
    )

    // 保存控制器以便可以中断请求
    currentStreamController.value = controller

  } catch (error) {
    console.error('Chat request failed:', error)
    isMessageSending.value = false

    // 显示错误消息
    const messageIndex = messages.value.findIndex(m => m.loading)
    if (messageIndex !== -1) {
      messages.value[messageIndex] = {
        ...messages.value[messageIndex],
        content: '发送消息失败，请重试',
        loading: false
      }
      items.value = [...messages.value]
    }
  }
}

// 获取会话列表
const fetchConversations = async () => {
  try {
    const response = await chatApi.getConversations('abc-123')
    if (response.data) {
      conversationsItems.value = response.data.data.map(conv => ({
        key: conv.id,
        label: conv.name,
        update_time: conv.updated_at
      }))
    }
  } catch (error) {
    console.error('获取会话列表失败:', error)
    message.error('获取会话列表失败')
  }
}

// 选择会话
const selectSession = async (sessionId) => {
  // 先设置会话ID，确保输入框显示
  currentSession.value = sessionId
  console.log('选择会话:', sessionId)

  try {
    const response = await chatApi.getMessages(sessionId, 'abc-123')
    console.log('API响应数据:', response.data) // 调试日志

    if (response.data && Array.isArray(response.data.data)) {
      // 创建一个新的消息数组
      const processedMessages = [];

      // 处理每条API返回的消息
      response.data.data.forEach((msg, index) => {
        console.log('处理消息:', msg) // 调试日志

        // 如果有查询，添加用户消息
        if (msg.query) {
          processedMessages.push({
            key: `msg_user_${index}`,
            content: msg.query,
            role: 'local',
            loading: false
          });
        }

        // 如果有回答，添加AI消息
        if (msg.answer) {
          processedMessages.push({
            key: `msg_ai_${index}`,
            content: msg.answer,
            role: 'ai',
            reference: msg.reference || null,
            liked: msg.feedback === 'like',
            disliked: msg.feedback === 'dislike',
            loading: false,
            id: msg.id // 保存真实的消息ID
          });
        }

        // 如果既没有query也没有answer，但有content，根据role添加消息
        if (!msg.query && !msg.answer && msg.content) {
          processedMessages.push({
            key: `msg_${index}`,
            content: msg.content,
            role: msg.role === 'user' ? 'local' : 'ai',
            reference: msg.reference || null,
            liked: msg.feedback === 'like',
            disliked: msg.feedback === 'dislike',
            loading: false
          });
        }
      });

      console.log('处理后的消息:', processedMessages) // 调试日志

      // 更新消息数组
      messages.value = processedMessages;

      // 同步更新 items 数组
      items.value = [...messages.value];

      // 确保DOM更新后滚动到底部
      nextTick(() => {
        console.log('DOM更新后，准备滚动到底部') // 调试日志
        scrollToBottom()
      })
    } else {
      console.error('API响应格式不符合预期:', response.data)
      message.error('获取会话消息失败：响应格式错误')
    }
  } catch (error) {
    console.error('获取会话消息失败:', error)
    message.error('获取会话消息失败')
    // 即使获取消息失败，也保持会话ID不变，确保输入框显示
  }
}

// 添加新会话
const onAddConversation = async () => {
  // 不再调用API创建会话，只是清空当前会话状态
  currentSession.value = ''
  messages.value = []
  items.value = []

  // 确保滚动到底部以显示中央输入框
  nextTick(() => {
    scrollToBottom()
  })
}

// 确认删除会话
const confirmDeleteSession = (sessionId) => {
  Modal.confirm({
    title: '确认删除会话',
    content: '您确定要删除这个会话吗？',
    onOk: () => deleteSession(sessionId),
    onCancel() {
      console.log('取消删除')
    },
  })
}

// 删除会话
const deleteSession = async (sessionId) => {
  try {
    await chatApi.deleteConversation(sessionId, 'abc-123')

    // 从会话列表中移除
    conversationsItems.value = conversationsItems.value.filter(item => item.key !== sessionId)

    // 如果删除的是当前会话，清空消息列表
    if (currentSession.value === sessionId) {
      messages.value = []
      items.value = []
      currentSession.value = ''
    }

    message.success('会话已成功删除')
  } catch (error) {
    console.error('删除会话失败:', error)
    message.error('删除会话失败，请重试')
  }
}

// 点赞功能
const likeMessage = async (key) => {
  const messageIndex = messages.value.findIndex(m => m.key === key)
  if (messageIndex !== -1 && messages.value[messageIndex].id) {
    try {
      // 确定要发送的反馈类型
      let feedbackType = 'like';

      // 如果已经点赞，再次点击则取消点赞
      if (messages.value[messageIndex].liked) {
        feedbackType = null;
      }

      // 使用真实的消息ID而不是前端生成的key
      await chatApi.sendFeedback(messages.value[messageIndex].id, feedbackType, 'abc-123')

      // 更新本地状态
      if (feedbackType === null) {
        // 取消点赞
        messages.value[messageIndex].liked = false;
      } else {
        // 设置点赞，取消点踩
        messages.value[messageIndex].liked = true;
        messages.value[messageIndex].disliked = false;
      }

      messages.value = [...messages.value]

      // 显示反馈消息
      feedbackMessage.value = '感谢您的反馈！'
      feedbackKey.value = key

      // 3秒后清除反馈消息
      setTimeout(() => {
        feedbackMessage.value = ''
      }, 3000)

    } catch (error) {
      console.error('发送反馈失败:', error)
      message.error('发送反馈失败')
    }
  } else {
    console.error('找不到消息ID，无法发送反馈')
    message.error('无法发送反馈，消息ID不存在')
  }
}

onMounted(async () => {
  await store.loadDictData()
  await fetchConversations()
})

// 监听 activeKey 变化
watch(activeKey, (newKey) => {
  selectedKeys.value = [newKey]
  messages.value = []
})

</script>
<style src="@/assets/app.css"></style>
<style>
.workflow-parallel {
  margin: 8px 0;
  border: 1px solid #f0f0f0;
  border-radius: 4px;
  padding: 8px;
}

.parallel-header {
  font-weight: 500;
  margin-bottom: 8px;
  color: #1890ff;
  font-size: 13px;
  display: flex;
  align-items: center;
  height: 20px;
}

.parallel-header::before {
  content: '';
  display: inline-block;
  width: 2px;
  height: 12px;
  background-color: #1890ff;
  margin-right: 6px;
  border-radius: 1px;
}

.parallel-branches {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.parallel-branch {
  background: #fafafa;
  padding: 6px 8px;
  border-radius: 4px;
  border-left: 2px solid #1890ff;
}

.branch-header {
  font-weight: 500;
  margin-bottom: 4px;
  color: #666;
  font-size: 12px;
  display: flex;
  align-items: center;
  height: 18px;
}

.branch-step {
  margin-left: 4px;
  border: 1px solid #f0f0f0;
}

.workflow-step {
  display: flex;
  align-items: center;
  margin: 4px 0;
  padding: 4px 8px;
  background: #fff;
  border-radius: 4px;
  min-height: 28px;
}

.step-icon {
  margin-right: 6px;
  display: flex;
  align-items: center;
}

.step-title {
  flex: 1;
  color: #333;
  font-size: 12px;
  line-height: 1.2;
}

.step-time {
  color: #999;
  font-size: 11px;
  margin-left: 6px;
  white-space: nowrap;
}

</style>
