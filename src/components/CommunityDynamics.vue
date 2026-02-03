<!--
社区动态管理组件
API配置信息：
- 开发环境：http://localhost:8080/api/v1
- 生产环境：https://api.lingjing.com/v1
- 请求超时时间：30秒

对应API接口：
- GET /api/community/dynamics - 获取社区动态列表（管理端）
- GET /api/community/dynamics/:id - 获取社区动态详情（管理端）
- DELETE /api/community/dynamics/:id - 删除社区动态（管理端）
- PUT /api/community/dynamics/:id/status - 更新社区动态状态（管理端）
- POST /api/community/dynamics/batch/delete - 批量删除社区动态（管理端）
- POST /api/community/dynamics/batch/hide - 批量隐藏社区动态（管理端）
- POST /api/community/dynamics/batch/verify - 批量审核社区动态（管理端）
- GET /api/community/reports - 获取举报列表（管理端）
- PUT /api/community/reports/:id/handle - 处理举报（管理端）
- GET /api/community/analytics - 获取社区数据分析（管理端）
-->
<template>
  <div class="community-container">
    <!-- 头部区域 -->
    <h2 class="community-title">社区动态管理</h2>
    
    <!-- 选项卡导航 -->
    <el-tabs v-model="activeTab" class="community-tabs">
      <!-- 动态列表选项卡 -->
      <el-tab-pane label="动态列表" name="dynamic-list">
        <!-- 操作按钮 -->
        <div class="header-actions">
          <el-button type="danger" @click="batchDelete" :disabled="selectedItems.length === 0">
            <el-icon><Delete /></el-icon>
            批量删除
          </el-button>
          <el-button type="warning" @click="batchHide" :disabled="selectedItems.length === 0">
            <el-icon><View /></el-icon>
            批量隐藏
          </el-button>
          <el-button type="success" @click="batchVerify" :disabled="selectedItems.length === 0">
            <el-icon><Check /></el-icon>
            批量审核
          </el-button>
        </div>

        <!-- 搜索区域 -->
        <el-card class="search-card">
          <div class="search-form">
            <div class="form-row">
              <el-form-item label="发布者">
                <el-input v-model="searchForm.author" placeholder="请输入发布者" clearable />
              </el-form-item>
              <el-form-item label="状态">
                <el-select v-model="searchForm.status" placeholder="请选择状态" clearable>
                  <el-option label="已发布" value="published" />
                  <el-option label="待审核" value="pending" />
                  <el-option label="已隐藏" value="hidden" />
                </el-select>
              </el-form-item>
            </div>
            <div class="form-actions">
              <el-button type="primary" @click="searchDynamics">
                <el-icon><Search /></el-icon>
                搜索
              </el-button>
              <el-button @click="resetSearch">
                <el-icon><RefreshRight /></el-icon>
                重置
              </el-button>
            </div>
          </div>
        </el-card>

        <!-- 动态列表表格 -->
        <el-card class="dynamics-table-card">
          <div class="card-header">
            <span>动态列表</span>
            <el-button size="small" @click="refreshDynamics">
              <el-icon><Refresh /></el-icon>
              刷新
            </el-button>
          </div>
          <div class="table-container">
            <template v-if="showSkeleton">
              <el-skeleton :rows="5" animated />
            </template>
            <template v-else>
              <el-table 
                :data="currentPageDynamics" 
                stripe 
                style="width: 100%"
                @selection-change="handleSelectionChange"
              >
                <el-table-column type="selection" width="55" />
                <el-table-column label="发布者" min-width="150">
                  <template #default="scope">
                    <div class="author-info">
                      <div class="author-avatar">{{ scope.row.authorName.charAt(0) }}</div>
                      <div class="author-name">{{ scope.row.authorName }}</div>
                    </div>
                  </template>
                </el-table-column>
                <el-table-column prop="content" label="内容" min-width="200" show-overflow-tooltip />
                <el-table-column label="状态" min-width="100">
                  <template #default="scope">
                    <el-tag :type="getStatusType(scope.row.status)">
                      {{ getStatusText(scope.row.status) }}
                    </el-tag>
                  </template>
                </el-table-column>
                <el-table-column prop="publishTime" label="发布时间" min-width="160" />
                <el-table-column label="操作" min-width="150" fixed="right">
                  <template #default="scope">
                    <el-button type="primary" link size="small" @click="viewDetail(scope.row)">
                      <el-icon><View /></el-icon>
                      详情
                    </el-button>
                    <el-button type="danger" link size="small" @click="deleteDynamic(scope.row)">
                      <el-icon><Delete /></el-icon>
                      删除
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
            </template>
          </div>

          <!-- 分页 -->
          <div class="pagination-container">
            <div class="pagination-info">
              共 {{ filteredDynamics.length }} 条记录
            </div>
            <el-pagination
              v-model:current-page="currentPage"
              v-model:page-size="pageSize"
              :page-sizes="[10, 20]"
              layout="total, sizes, prev, pager, next"
              :total="filteredDynamics.length"
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
            />
          </div>
        </el-card>
      </el-tab-pane>
      
      <!-- 数据分析选项卡 -->
      <el-tab-pane label="数据分析" name="data-analysis">
        <el-card class="analysis-card">
          <div class="card-header">
            <span>社区数据概览</span>
          </div>
          <div class="analysis-content">
            <el-row :gutter="20">
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ totalDynamics }}</div>
                  <div class="stat-label">动态总数</div>
                </div>
              </el-col>
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ todayDynamics }}</div>
                  <div class="stat-label">今日新增</div>
                </div>
              </el-col>
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ totalInteractions }}</div>
                  <div class="stat-label">总互动数</div>
                </div>
              </el-col>
              <el-col :span="6">
                <div class="stat-card">
                  <div class="stat-value">{{ pendingReviews }}</div>
                  <div class="stat-label">待审核</div>
                </div>
              </el-col>
            </el-row>
          </div>
        </el-card>
      </el-tab-pane>
      
      <!-- 举报管理选项卡 -->
      <el-tab-pane label="举报管理" name="report-management">
        <el-card class="report-card">
          <div class="card-header">
            <span>举报处理</span>
          </div>
          <div class="report-content">
            <el-table :data="reports" stripe style="width: 100%">
              <el-table-column prop="reportTime" label="举报时间" min-width="160" />
              <el-table-column prop="reporterName" label="举报人" min-width="120" />
              <el-table-column prop="reportedContent" label="被举报内容" min-width="200" show-overflow-tooltip />
              <el-table-column prop="reason" label="原因" min-width="150" />
              <el-table-column label="状态" min-width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.status === 'pending' ? 'warning' : 'success'">
                    {{ scope.row.status === 'pending' ? '待处理' : '已处理' }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="操作" min-width="150" fixed="right">
                <template #default="scope">
                  <el-button type="primary" link size="small" @click="handleReport(scope.row)" v-if="scope.row.status === 'pending'">
                    <el-icon><Check /></el-icon>
                    处理
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </div>
        </el-card>
      </el-tab-pane>
    </el-tabs>

    <!-- 动态详情模态框 -->
    <el-dialog
      v-model="detailDialogVisible"
      title="动态详情"
      width="600px"
      :close-on-click-modal="false"
    >
      <div v-if="currentDynamic" class="dynamic-detail">
        <div class="detail-header">
          <div class="author-info">
            <div class="author-avatar">{{ currentDynamic.authorName.charAt(0) }}</div>
            <div class="author-details">
              <div class="author-name">{{ currentDynamic.authorName }}</div>
              <div class="publish-time">{{ currentDynamic.publishTime }}</div>
            </div>
          </div>
        </div>
        <div class="detail-content">
          <p>{{ currentDynamic.content }}</p>
        </div>
        <div class="detail-stats">
          <div class="stat-item">
            <el-icon><Star /></el-icon>
            <span>{{ currentDynamic.likeCount }} 点赞</span>
          </div>
          <div class="stat-item">
            <el-icon><ChatDotRound /></el-icon>
            <span>{{ currentDynamic.commentCount }} 评论</span>
          </div>
        </div>
      </div>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="detailDialogVisible = false">关闭</el-button>
          <el-button type="danger" @click="deleteDynamic(currentDynamic)">删除</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Delete,
  View,
  Check,
  Search,
  RefreshRight,
  Refresh,
  Star,
  ChatDotRound
} from '@element-plus/icons-vue'

// API基础URL配置
// const API_BASE_URL = process.env.NODE_ENV === 'development' 
//   ? 'http://localhost:8080/api/v1' 
//   : 'https://api.lingjing.com/v1'

// 请求超时时间：30秒
// const API_TIMEOUT = 30000

// 活动标签
const activeTab = ref('dynamic-list')

/**
 * 动态数据
 * 对应API: GET /community/posts
 * 数据结构：
 * - id: 动态ID
 * - authorId: 作者ID
 * - authorName: 作者名称
 * - content: 动态内容
 * - status: 状态 (published/pending/hidden)
 * - publishTime: 发布时间
 * - likeCount: 点赞数
 * - commentCount: 评论数
 */
const dynamics = ref([
  { id: '200001', authorId: '100001', authorName: '李晓明', content: '今天天气真好，适合户外运动！分享一张跑步的照片。', status: 'published', publishTime: '2024-12-27 09:30:15', likeCount: 45, commentCount: 12 },
  { id: '200002', authorId: '100002', authorName: '王芳', content: '求助：最近睡眠质量不好，大家有什么建议吗？', status: 'published', publishTime: '2024-12-27 08:45:22', likeCount: 23, commentCount: 34 },
  { id: '200003', authorId: '100003', authorName: '张伟', content: '分享一个养生小知识：每天喝温水对身体好处多多', status: 'pending', publishTime: '2024-12-27 10:15:33', likeCount: 0, commentCount: 0 },
  { id: '200004', authorId: '100004', authorName: '刘洋', content: '今天学习了新的瑜伽动作，感觉身体轻松了很多！', status: 'published', publishTime: '2024-12-27 07:20:45', likeCount: 89, commentCount: 23 },
  { id: '200005', authorId: '100005', authorName: '陈静', content: '春季养生食谱分享，大家快来试试吧！', status: 'published', publishTime: '2024-12-26 18:30:10', likeCount: 156, commentCount: 45 }
])

// 搜索表单
const searchForm = ref({
  author: '',
  status: ''
})

// 分页数据
const currentPage = ref(1)
const pageSize = ref(10)

// 优化性能：减少初始加载的数据量
const showSkeleton = ref(true)

// 选择的项
const selectedItems = ref([])

// 详情模态框
const detailDialogVisible = ref(false)
const currentDynamic = ref(null)

/**
 * 分析数据
 * 对应API: 待后端提供
 * 数据结构：
 * - totalDynamics: 动态总数
 * - todayDynamics: 今日新增
 * - totalInteractions: 总互动数
 * - pendingReviews: 待审核数量
 */
const totalDynamics = ref(1580)
const todayDynamics = ref(45)
const totalInteractions = ref(12500)
const pendingReviews = ref(12)

/**
 * 举报数据
 * 对应API: 待后端提供
 * 数据结构：
 * - id: 举报ID
 * - reportTime: 举报时间
 * - reporterName: 举报人
 * - reportedContent: 被举报内容
 * - reason: 举报原因
 * - status: 状态 (pending/resolved)
 */
const reports = ref([
  { id: 'R001', reportTime: '2024-12-27 09:15:22', reporterName: '赵强', reportedContent: '这个动态包含不当内容', reason: '广告推销', status: 'pending' },
  { id: 'R002', reportTime: '2024-12-27 08:45:10', reporterName: '孙丽', reportedContent: '虚假信息', reason: '虚假信息', status: 'pending' },
  { id: 'R003', reportTime: '2024-12-26 16:20:33', reporterName: '周杰', reportedContent: '这个动态侵犯了我的隐私', reason: '侵权举报', status: 'resolved' }
])

/**
 * 过滤后的动态列表
 * 对应API: GET /community/posts (带搜索参数)
 * 搜索参数：
 * - author: 发布者名称
 * - status: 状态
 */
const filteredDynamics = computed(() => {
  const { author, status } = searchForm.value
  return dynamics.value.filter(dynamic => {
    const matchAuthor = !author || dynamic.authorName.includes(author)
    const matchStatus = !status || dynamic.status === status
    return matchAuthor && matchStatus
  })
})

/**
 * 当前页动态数据
 * 对应API: GET /community/posts (带分页参数)
 * 分页参数：
 * - page: 当前页码
 * - pageSize: 每页数量
 */
const currentPageDynamics = computed(() => {
  const startIndex = (currentPage.value - 1) * pageSize.value
  const endIndex = startIndex + pageSize.value
  return filteredDynamics.value.slice(startIndex, endIndex)
})

/**
 * 搜索动态
 * 对应API: GET /api/community/dynamics (带搜索参数)
 * 搜索参数：
 * - author: 发布者名称
 * - status: 状态
 * - page: 当前页码
 * - pageSize: 每页数量
 */
const searchDynamics = () => {
  currentPage.value = 1
  // TODO: 调用API搜索动态
  // 示例：axios.get(`${API_BASE_URL}/api/community/dynamics`, { params: { ...searchForm.value, page: currentPage.value, pageSize: pageSize.value } })
  ElMessage.success('搜索完成')
}

/**
 * 重置搜索
 */
const resetSearch = () => {
  searchForm.value = {
    author: '',
    status: ''
  }
  currentPage.value = 1
  ElMessage.success('搜索条件已重置')
}

/**
 * 查看详情
 * 对应API: GET /api/community/dynamics/:id
 * @param {Object} dynamic - 动态对象
 */
const viewDetail = (dynamic) => {
  // TODO: 调用API获取动态详情
  // 示例：axios.get(`${API_BASE_URL}/api/community/dynamics/${dynamic.id}`)
  currentDynamic.value = dynamic
  detailDialogVisible.value = true
}

/**
 * 删除动态
 * 对应API: DELETE /api/community/dynamics/:id
 * @param {Object} dynamic - 动态对象
 */
const deleteDynamic = (dynamic) => {
  ElMessageBox.confirm('确定要删除这条动态吗？', '警告', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    // TODO: 调用API删除动态
    // 示例：axios.delete(`${API_BASE_URL}/api/community/dynamics/${dynamic.id}`)
    const index = dynamics.value.findIndex(d => d.id === dynamic.id)
    if (index !== -1) {
      dynamics.value.splice(index, 1)
      ElMessage.success('动态已删除')
    }
  }).catch(() => {})
}

/**
 * 批量删除
 * 对应API: POST /api/community/dynamics/batch/delete
 */
const batchDelete = () => {
  if (selectedItems.value.length === 0) {
    ElMessage.warning('请选择要删除的动态')
    return
  }
  
  ElMessageBox.confirm(`确定要删除选中的 ${selectedItems.value.length} 条动态吗？`, '警告', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    const selectedIds = selectedItems.value.map(item => item.id)
    // TODO: 调用API批量删除动态
    // 示例：axios.post(`${API_BASE_URL}/api/community/dynamics/batch/delete`, { ids: selectedIds })
    dynamics.value = dynamics.value.filter(d => !selectedIds.includes(d.id))
    selectedItems.value = []
    ElMessage.success(`已删除 ${selectedIds.length} 条动态`)
  }).catch(() => {})
}

/**
 * 批量隐藏
 * 对应API: POST /api/community/dynamics/batch/hide
 */
const batchHide = () => {
  // TODO: 调用API批量隐藏动态
  // 示例：axios.post(`${API_BASE_URL}/api/community/dynamics/batch/hide`, { ids: selectedItems.value.map(item => item.id) })
  selectedItems.value.forEach(item => {
    item.status = 'hidden'
  })
  ElMessage.success(`已隐藏 ${selectedItems.value.length} 条动态`)
  selectedItems.value = []
}

/**
 * 批量审核
 * 对应API: POST /api/community/dynamics/batch/verify
 */
const batchVerify = () => {
  // TODO: 调用API批量审核动态
  // 示例：axios.post(`${API_BASE_URL}/api/community/dynamics/batch/verify`, { ids: selectedItems.value.map(item => item.id) })
  selectedItems.value.forEach(item => {
    if (item.status === 'pending') {
      item.status = 'published'
    }
  })
  ElMessage.success(`已审核 ${selectedItems.value.length} 条动态`)
  selectedItems.value = []
}

/**
 * 刷新动态列表
 * 对应API: GET /api/community/dynamics
 */
const refreshDynamics = () => {
  // TODO: 调用API刷新动态列表
  // 示例：axios.get(`${API_BASE_URL}/api/community/dynamics`)
  ElMessage.success('动态列表已刷新')
}

/**
 * 处理选择变化
 * @param {Array} val - 选择的项
 */
const handleSelectionChange = (val) => {
  selectedItems.value = val
}

/**
 * 分页大小变化
 * @param {Number} newSize - 新的分页大小
 */
const handleSizeChange = (newSize) => {
  pageSize.value = newSize
  currentPage.value = 1
}

/**
 * 当前页码变化
 * @param {Number} newPage - 新的页码
 */
const handleCurrentChange = (newPage) => {
  currentPage.value = newPage
}

/**
 * 获取状态文本
 * @param {String} status - 状态值
 * @returns {String} 状态文本
 */
const getStatusText = (status) => {
  const map = {
    published: '已发布',
    pending: '待审核',
    hidden: '已隐藏'
  }
  return map[status] || status
}

/**
 * 获取状态类型
 * @param {String} status - 状态值
 * @returns {String} 状态类型
 */
const getStatusType = (status) => {
  const map = {
    published: 'success',
    pending: 'warning',
    hidden: 'info'
  }
  return map[status] || 'info'
}

/**
 * 处理举报
 * 对应API: PUT /api/community/reports/:id/handle
 * @param {Object} report - 举报对象
 */
const handleReport = (report) => {
  // TODO: 调用API处理举报
  // 示例：axios.put(`${API_BASE_URL}/api/community/reports/${report.id}/handle`)
  report.status = 'resolved'
  ElMessage.success('举报已处理')
}

// 生命周期
onMounted(() => {
  // 模拟数据加载，减少延迟时间以提升用户体验
  // TODO: 调用API获取动态列表
  // 示例：axios.get(`${API_BASE_URL}/api/community/dynamics`)
  setTimeout(() => {
    showSkeleton.value = false
  }, 100)
})
</script>

<style scoped>
.community-container {
  padding: 0;
}

.community-title {
  font-size: 24px;
  font-weight: 700;
  color: var(--el-text-color-primary);
  margin: 0 0 20px 0;
}

.community-tabs {
  margin-bottom: 20px;
}

.header-actions {
  display: flex;
  gap: 16px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.search-card {
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 16px;
  font-weight: 600;
  color: var(--el-text-color-primary);
  margin-bottom: 16px;
}

.search-form {
  width: 100%;
}

.form-row {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.form-actions {
  display: flex;
  gap: 16px;
}

.dynamics-table-card {
  margin-bottom: 20px;
}

.table-container {
  margin-bottom: 20px;
  overflow-x: auto;
}

.pagination-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 20px;
  border-top: 1px solid var(--el-border-color);
}

.pagination-info {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

/* 作者信息 */
.author-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.author-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: 600;
  font-size: 14px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.author-name {
  font-size: 14px;
  font-weight: 500;
  color: var(--el-text-color-primary);
}

.publish-time {
  font-size: 12px;
  color: var(--el-text-color-tertiary);
}

/* 详情页 */
.dynamic-detail {
  padding: 10px 0;
}

.detail-header {
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 1px solid var(--el-border-color);
}

.author-details {
  display: flex;
  flex-direction: column;
}

.detail-content {
  margin-bottom: 20px;
}

.detail-content p {
  font-size: 15px;
  line-height: 1.6;
  color: var(--el-text-color-primary);
}

.detail-stats {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

/* 分析页面 */
.analysis-card {
  margin-bottom: 20px;
}

.analysis-content {
  padding: 20px 0;
}

.stat-card {
  text-align: center;
  padding: 20px;
  border-radius: var(--el-border-radius);
  box-shadow: var(--el-box-shadow-light);
}

.stat-label {
  font-size: 12px;
  color: var(--el-text-color-tertiary);
  margin-bottom: 8px;
}

.stat-value {
  font-size: 24px;
  font-weight: 600;
  color: var(--el-text-color-primary);
}

/* 举报管理页面 */
.report-card {
  margin-bottom: 20px;
}

.report-content {
  padding: 20px 0;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header-actions {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
  
  .form-row {
    flex-direction: column;
    gap: 16px;
  }
  
  .card-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }
  
  .pagination-container {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }
}
</style>