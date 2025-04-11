<template>
  <CommonPage show-footer title="AI公文写作">


    <!-- 文档类型选择栏 -->
    <div class="doc-type-nav">
      <div v-for="type in docTypes" :key="type.key" :class="['type-item', { active: currentType === type.key }]"
        @click="currentType = type.key">
        <i :class="type.icon"></i>
        <span>{{ type.label }}</span>
      </div>
    </div>

    <!-- 文档子类型选择 -->
    <div class="sub-type-section">
      <div class="sub-type-label">通知类别</div>
      <div class="sub-type-nav">
        <div v-for="subType in subTypes" :key="subType.key"
          :class="['sub-type-item', { active: currentSubType === subType.key }]" @click="currentSubType = subType.key">
          {{ subType.label }}
        </div>
      </div>
    </div>

    <!-- 主要内容区域 -->
    <div class="main-content">

      <!-- 标题输入 -->
      <div class="input-area">
        <div class="input-label">通知标题</div>
        <div class="input-content">
          <input v-model="docTitle" placeholder="关于展开总部青年夜学的通知" class="input-field" required />
        </div>
      </div>

      <!-- 通知提要 -->
      <div class="input-area">
        <div class="input-label">通知提要</div>
        <div class="input-content">
          <textarea v-model="summary" placeholder="请写通知提要例如：
1. 认识推进国企低效用地减量化的重要意义；
2. 国企低效用地减量化的目标和原则；
3. 加强国企低效用地减量化工作的组织领导；
4. 国企低效用地减量化工作具体措施：动员部署，调查摸底；确定目标，制定方案；以点带面，推进实施；完善机制，丰富政策" rows="6" class="input-field textarea"></textarea>
        </div>
      </div>

      <!-- 主送机关 -->
      <div class="input-area">
        <div class="input-label">主送机关</div>
        <div class="input-content">
          <input v-model="mainRecipient" placeholder="请写主送机关例如：各有关市属国企" class="input-field" />
        </div>
      </div>

      <!-- 发文机关 -->
      <div class="input-area">
        <div class="input-label">发文机关</div>
        <div class="input-content">
          <input v-model="issuingOrg" placeholder="请写发文机关例如：XXXXXX委员会" class="input-field" />
        </div>
      </div>

      <!-- 通知缘由 -->
      <div class="input-area">
        <div class="input-label">通知缘由</div>
        <div class="input-content">
          <textarea v-model="reason" placeholder="请写通知缘由例如：落实上海2035城市总体规划，提高土地资源利用质量，推进十四五期间低效建设用地减量化，盘活利用国有企业低效改建设用地"
            rows="4" class="input-field textarea"></textarea>
        </div>
      </div>

      <!-- 批示内容 -->
      <div class="input-area">
        <div class="input-label">批示内容</div>
        <div class="input-content">
          <textarea v-model="instructions"
            placeholder="请写批示内容例如：加强高温作业及高温天气作业劳动保护，有效防范职业性中暑事件的发生，切实保障劳动者身心健康和生命安全，并请各省级卫生健康行政部门及时检查防暑降温工作开展情况。"
            rows="4" class="input-field textarea"></textarea>
        </div>
      </div>

      <!-- 结语 -->
      <div class="input-area">
        <div class="input-label">结语</div>
        <div class="input-content">
          <input v-model="conclusion" placeholder="请写结语例如：各部门要高度重视，切实加强组织实施，确保国企低效用地减量化工作取得实效。特此通知。"
            class="input-field" />
        </div>
      </div>

      <!-- AI模型复选框 -->
      <div class="ai-model-checkbox">
        <input type="checkbox" id="ai-model" v-model="useAIModel" />
        <label for="ai-model">该文由AI生成，仅供参考！</label>
      </div>

      <!-- 生成按钮 -->
      <div class="action-area">
        <button class="generate-btn" @click="generateDocument">
          立即生成
        </button>
      </div>
    </div>
  </CommonPage>
</template>

<script setup>
import { ref } from 'vue'

// 文档类型定义
const docTypes = [
  { key: 'work-report', label: '工作报告', icon: 'i-document' },
  { key: 'speech', label: '讲话稿', icon: 'i-speech' },
  { key: 'heart', label: '心得体会', icon: 'i-heart' },
  { key: 'notice', label: '通知报告', icon: 'i-notice' },
  { key: 'invitation', label: '邀请函', icon: 'i-invitation' },
  { key: 'common', label: '通用公文', icon: 'i-common' }
]

// 子类型定义
const subTypes = [
  { key: 'publish', label: '发布类' },
  { key: 'approve', label: '批转类' },
  { key: 'instruction', label: '指示类' },
  { key: 'forward', label: '转发类' },
  { key: 'matter', label: '事务类' }
]

// 响应式状态
const currentType = ref('notice')
const currentSubType = ref('publish')
const docTitle = ref('')
const summary = ref('')
const mainRecipient = ref('')
const issuingOrg = ref('')
const reason = ref('')
const instructions = ref('')
const conclusion = ref('')
const useAIModel = ref(true)

// 生成文档方法
const generateDocument = () => {
  console.log('Generating document with:', {
    type: currentType.value,
    subType: currentSubType.value,
    title: docTitle.value,
    summary: summary.value,
    mainRecipient: mainRecipient.value,
    issuingOrg: issuingOrg.value,
    reason: reason.value,
    instructions: instructions.value,
    conclusion: conclusion.value,
    useAIModel: useAIModel.value
  })
  // 这里可以添加实际的生成逻辑或API调用
}
</script>

<style scoped>
.document-writing {
  padding: 20px;
  min-height: 100vh;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.header {
  text-align: center;
  margin-bottom: 20px;
}

.main-title {
  color: #18A058;
  font-size: 28px;
  font-weight: bold;
  margin-bottom: 8px;
}

.subtitle {
  color: #36ad6a;
  font-size: 16px;
}

.doc-type-nav {
  display: flex;
  justify-content: space-between;
  background: white;
  border-radius: 30px;
  padding: 10px;
  margin-bottom: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.type-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  cursor: pointer;
  border-radius: 20px;
  transition: all 0.3s;
  flex: 1;
  justify-content: center;
}

.type-item i {
  width: 20px;
  height: 20px;
  display: inline-block;
}

.type-item.active {
  color: #18A058;
  font-weight: 500;
  position: relative;
}

.type-item.active::after {
  content: '';
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 30px;
  height: 3px;
  background: #18A058;
  border-radius: 3px;
}

.sub-type-section {
  margin-bottom: 20px;
}

.sub-type-label {
  color: #333;
  font-size: 14px;
  margin-bottom: 10px;
}

.sub-type-nav {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.sub-type-item {
  padding: 6px 16px;
  border: 1px solid #ddd;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 14px;
}

.sub-type-item:hover {
  border-color: #18A058;
  color: #18A058;
}

.sub-type-item.active {
  background: #18A058;
  color: white;
  border-color: #18A058;
}

.main-content {
  background: white;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.input-area {
  margin-bottom: 20px;
}

.input-label {
  color: #333;
  font-size: 14px;
  margin-bottom: 8px;
}

.input-field {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #e8e8e8;
  border-radius: 4px;
  font-size: 14px;
  transition: all 0.3s;
}

.textarea {
  resize: none;
  min-height: 80px;
}

.input-field:focus {
  outline: none;
  border-color: #18A058;
  box-shadow: 0 0 0 2px rgba(24, 160, 88, 0.1);
}

.ai-model-checkbox {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 20px 0;
  color: #18A058;
}

.action-area {
  text-align: center;
  margin-top: 20px;
}

.generate-btn {
  padding: 12px 40px;
  font-size: 16px;
  background: #18A058;
  color: white;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s;
}

.generate-btn:hover {
  background: #0c7a43;
  box-shadow: 0 4px 12px rgba(24, 160, 88, 0.3);
}

/* 模拟图标 */
.i-document,
.i-speech,
.i-heart,
.i-notice,
.i-invitation,
.i-common {
  display: inline-block;
  width: 18px;
  height: 18px;
  background-color: #18A058;
  mask-size: contain;
  mask-repeat: no-repeat;
  mask-position: center;
  -webkit-mask-size: contain;
  -webkit-mask-repeat: no-repeat;
  -webkit-mask-position: center;
}

.i-document {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z'/%3E%3Cpolyline points='14 2 14 8 20 8'/%3E%3Cline x1='16' y1='13' x2='8' y2='13'/%3E%3Cline x1='16' y1='17' x2='8' y2='17'/%3E%3Cpolyline points='10 9 9 9 8 9'/%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z'/%3E%3Cpolyline points='14 2 14 8 20 8'/%3E%3Cline x1='16' y1='13' x2='8' y2='13'/%3E%3Cline x1='16' y1='17' x2='8' y2='17'/%3E%3Cpolyline points='10 9 9 9 8 9'/%3E%3C/svg%3E");
}

.i-speech {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z'/%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z'/%3E%3C/svg%3E");
}

.i-heart {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z'/%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z'/%3E%3C/svg%3E");
}

.i-notice {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9'/%3E%3Cpath d='M13.73 21a2 2 0 0 1-3.46 0'/%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9'/%3E%3Cpath d='M13.73 21a2 2 0 0 1-3.46 0'/%3E%3C/svg%3E");
}

.i-invitation {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z'/%3E%3Cpolyline points='22,6 12,13 2,6'/%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z'/%3E%3Cpolyline points='22,6 12,13 2,6'/%3E%3C/svg%3E");
}

.i-common {
  mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z'/%3E%3C/svg%3E");
  -webkit-mask-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='24' height='24' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z'/%3E%3C/svg%3E");
}
</style>