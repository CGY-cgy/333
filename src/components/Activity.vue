<!--
活动管理组件
API配置信息：
- 开发环境：http://localhost:8080/api/v1
- 生产环境：https://api.lingjing.com/v1
- 请求超时时间：30秒

对应API接口：
- GET /api/activities - 获取活动列表
- POST /api/activities - 创建活动
- GET /api/activities/:id - 获取活动详情
- PUT /api/activities/:id - 更新活动
- DELETE /api/activities/:id - 删除活动
- POST /api/activities/:id/start - 启动活动
- POST /api/activities/:id/stop - 停止活动
- GET /api/activities/statistics - 获取活动统计数据
- POST /api/activities/import - 导入活动
- GET /api/activities/export - 导出活动
-->
<template>
  <div class="activity-container">
    <!-- 页面标题和操作 -->
    <div class="header-section">
      <h1 class="page-title">活动管理</h1>
    </div>

    <!-- 标签页切换 -->
    <el-tabs v-model="activeTab" class="main-tabs">
      <el-tab-pane label="活动列表" name="list">
        <!-- 顶部操作栏 -->
        <div class="top-actions">
          <el-button type="primary" @click="openCreateDialog">
            <el-icon-plus />
            <span>创建活动</span>
          </el-button>
          <el-button @click="refreshActivities">
            <el-icon-refresh />
            <span>刷新</span>
          </el-button>
          <el-button @click="exportActivities">
            <el-icon-document-download />
            <span>导出</span>
          </el-button>
          <el-button @click="importActivities">
            <el-icon-upload />
            <span>导入</span>
          </el-button>
        </div>

        <!-- 搜索和筛选 -->
        <div class="search-filter">
          <el-input v-model="searchKeyword" placeholder="搜索活动名称" clearable class="search-input">
            <template #prefix>
              <el-icon-search />
            </template>
          </el-input>
          <el-select v-model="activityType" placeholder="活动类型" clearable class="filter-select">
            <el-option label="全部类型" value="" />
            <el-option label="会员折扣" value="member_discount" />
            <el-option label="免费会员" value="free_member" />
            <el-option label="积分翻倍" value="points_double" />
            <el-option label="满减活动" value="满减活动" />
            <el-option label="限时特惠" value="限时特惠" />
          </el-select>
          <el-select v-model="activityStatus" placeholder="活动状态" clearable class="filter-select">
            <el-option label="全部状态" value="" />
            <el-option label="未开始" value="pending" />
            <el-option label="进行中" value="active" />
            <el-option label="已结束" value="ended" />
            <el-option label="已暂停" value="paused" />
          </el-select>
        </div>

        <!-- 活动列表 -->
        <el-table :data="filteredActivities" style="width: 100%" border stripe>
          <el-table-column prop="id" label="活动ID" width="80" />
          <el-table-column prop="name" label="活动名称" />
          <el-table-column prop="type" label="活动类型" width="120">
            <template #default="scope">
              <el-tag :type="getActivityTypeTag(scope.row.type)">
                {{ getActivityTypeName(scope.row.type) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="status" label="活动状态" width="100">
            <template #default="scope">
              <el-tag :type="getActivityStatusTag(scope.row.status)">
                {{ getActivityStatusName(scope.row.status) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="startTime" label="开始时间" width="180" />
          <el-table-column prop="endTime" label="结束时间" width="180" />
          <el-table-column prop="participants" label="参与人数" width="100" />
          <el-table-column prop="createdBy" label="创建人" width="120" />
          <el-table-column label="操作" width="200" fixed="right">
            <template #default="scope">
              <el-button type="primary" size="small" @click="editActivity(scope.row)">
                编辑
              </el-button>
              <el-button type="success" size="small" @click="startActivity(scope.row)" v-if="scope.row.status === 'pending' || scope.row.status === 'paused'">
                启动
              </el-button>
              <el-button type="warning" size="small" @click="pauseActivity(scope.row)" v-else-if="scope.row.status === 'active'">
                暂停
              </el-button>
              <el-button type="danger" size="small" @click="deleteActivity(scope.row.id)">
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>

        <!-- 分页 -->
        <div class="pagination">
          <el-pagination
            v-model:current-page="page.current"
            v-model:page-size="page.size"
            :page-sizes="[10, 20, 50, 100]"
            layout="total, sizes, prev, pager, next, jumper"
            :total="activities.length"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
          />
        </div>
      </el-tab-pane>

      <el-tab-pane label="活动详情" name="detail">
        <el-card shadow="hover" class="detail-card" v-if="selectedActivity">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">{{ selectedActivity.name }}</h3>
              <el-tag :type="getActivityStatusTag(selectedActivity.status)">
                {{ getActivityStatusName(selectedActivity.status) }}
              </el-tag>
            </div>
          </template>
          <div class="activity-detail">
            <div class="detail-section">
              <h4 class="section-title">活动信息</h4>
              <div class="detail-grid">
                <div class="detail-item">
                  <span class="detail-label">活动类型：</span>
                  <span class="detail-value">{{ getActivityTypeName(selectedActivity.type) }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">开始时间：</span>
                  <span class="detail-value">{{ selectedActivity.startTime }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">结束时间：</span>
                  <span class="detail-value">{{ selectedActivity.endTime }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">参与人数：</span>
                  <span class="detail-value">{{ selectedActivity.participants }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">创建时间：</span>
                  <span class="detail-value">{{ selectedActivity.createdAt }}</span>
                </div>
                <div class="detail-item">
                  <span class="detail-label">创建人：</span>
                  <span class="detail-value">{{ selectedActivity.createdBy }}</span>
                </div>
              </div>
            </div>

            <div class="detail-section">
              <h4 class="section-title">活动规则</h4>
              <div class="rule-content">
                <pre>{{ selectedActivity.rules }}</pre>
              </div>
            </div>

            <div class="detail-section">
              <h4 class="section-title">活动统计</h4>
              <div class="stats-grid">
                <div class="stat-item">
                  <div class="stat-value">{{ selectedActivity.statistics?.clicks || 0 }}</div>
                  <div class="stat-label">点击次数</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value">{{ selectedActivity.statistics?.participants || 0 }}</div>
                  <div class="stat-label">参与人数</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value">{{ selectedActivity.statistics?.conversions || 0 }}</div>
                  <div class="stat-label">转化率</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value">{{ selectedActivity.statistics?.revenue || 0 }}</div>
                  <div class="stat-label">活动收入</div>
                </div>
              </div>
            </div>
          </div>
        </el-card>
        <div class="empty-detail" v-else>
          <el-empty description="请选择一个活动查看详情" />
        </div>
      </el-tab-pane>

      <el-tab-pane label="活动统计" name="statistics">
        <el-card shadow="hover" class="stats-card">
          <template #header>
            <div class="card-header-content">
              <h3 class="card-title">活动统计分析</h3>
            </div>
          </template>
          <div class="stats-content">
            <!-- 统计图表 -->
            <div class="chart-section">
              <h4 class="section-title">活动效果分析</h4>
              <div class="chart-container">
                <!-- 这里可以添加Chart.js图表 -->
                <div class="chart-placeholder">
                  <el-icon class="chart-icon"><el-icon-data-analysis /></el-icon>
                  <p>活动效果分析图表</p>
                </div>
              </div>
            </div>

            <!-- 统计数据 -->
            <div class="stats-overview">
              <div class="overview-item">
                <div class="overview-value">{{ totalActivities }}</div>
                <div class="overview-label">总活动数</div>
              </div>
              <div class="overview-item">
                <div class="overview-value">{{ activeActivities }}</div>
                <div class="overview-label">进行中活动</div>
              </div>
              <div class="overview-item">
                <div class="overview-value">{{ totalParticipants }}</div>
                <div class="overview-label">总参与人数</div>
              </div>
              <div class="overview-item">
                <div class="overview-value">{{ totalRevenue }}</div>
                <div class="overview-label">活动总收入</div>
              </div>
            </div>
          </div>
        </el-card>
      </el-tab-pane>
    </el-tabs>

    <!-- 创建/编辑活动对话框 -->
    <el-dialog
      v-model="activityDialogVisible"
      :title="isEditing ? '编辑活动' : '创建活动'"
      width="700px"
      destroy-on-close
    >
      <el-form :model="activityForm" label-width="120px">
        <el-form-item label="活动名称" required>
          <el-input v-model="activityForm.name" placeholder="请输入活动名称" />
        </el-form-item>
        <el-form-item label="活动类型" required>
          <el-select v-model="activityForm.type" placeholder="请选择活动类型">
            <el-option label="会员折扣" value="member_discount" />
            <el-option label="免费会员" value="free_member" />
            <el-option label="积分翻倍" value="points_double" />
            <el-option label="满减活动" value="满减活动" />
            <el-option label="限时特惠" value="限时特惠" />
          </el-select>
        </el-form-item>
        <el-form-item label="活动状态" required>
          <el-select v-model="activityForm.status" placeholder="请选择活动状态">
            <el-option label="未开始" value="pending" />
            <el-option label="进行中" value="active" />
            <el-option label="已结束" value="ended" />
            <el-option label="已暂停" value="paused" />
          </el-select>
        </el-form-item>
        <el-form-item label="开始时间" required>
          <el-date-picker v-model="activityForm.startTime" type="datetime" placeholder="请选择开始时间" style="width: 100%" />
        </el-form-item>
        <el-form-item label="结束时间" required>
          <el-date-picker v-model="activityForm.endTime" type="datetime" placeholder="请选择结束时间" style="width: 100%" />
        </el-form-item>
        <el-form-item label="活动规则" required>
          <el-input v-model="activityForm.rules" type="textarea" :rows="4" placeholder="请输入活动规则" />
        </el-form-item>
        <el-form-item label="活动描述">
          <el-input v-model="activityForm.description" type="textarea" :rows="3" placeholder="请输入活动描述" />
        </el-form-item>
        <el-form-item label="活动图片">
          <el-upload class="activity-upload" action="#" :auto-upload="false">
            <el-button type="primary">
              <el-icon-upload />
              <span>上传图片</span>
            </el-button>
          </el-upload>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="activityDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="saveActivity">确定</el-button>
        </span>
      </template>
    </el-dialog>

    <!-- 导入活动对话框 -->
    <el-dialog
      v-model="importDialogVisible"
      title="导入活动"
      width="500px"
      destroy-on-close
    >
      <el-form label-width="100px">
        <el-form-item label="活动文件">
          <el-upload
            class="upload-demo"
            action="#"
            :auto-upload="false"
            :on-change="handleFileUpload"
            :show-file-list="false"
          >
            <el-button type="primary">
              <el-icon-upload />
              <span>选择文件</span>
            </el-button>
          </el-upload>
          <div v-if="importFile" class="file-info">
            <span>{{ importFile.name }}</span>
            <el-button type="text" @click="clearImportFile">清除</el-button>
          </div>
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="importDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="confirmImport">导入</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import { ElMessage } from 'element-plus'
import {
  Plus,
  Refresh,
  Search,
  Upload,
  Download,
  DataAnalysis
} from '@element-plus/icons-vue'

// 定义组件别名
const ElIconPlus = Plus
const ElIconRefresh = Refresh
const ElIconSearch = Search
const ElIconUpload = Upload
const ElIconDownload = Download
const ElIconDataAnalysis = DataAnalysis

// 响应式数据
const activeTab = ref('list')
const searchKeyword = ref('')
const activityType = ref('')
const activityStatus = ref('')
const page = reactive({ current: 1, size: 10 })
const activityDialogVisible = ref(false)
const importDialogVisible = ref(false)
const isEditing = ref(false)
const importFile = ref(null)

// 活动数据
const activities = ref([
  {
    id: 1,
    name: '会员折扣活动',
    type: 'member_discount',
    status: 'active',
    startTime: '2024-12-01 00:00:00',
    endTime: '2024-12-31 23:59:59',
    participants: 128,
    createdBy: 'admin',
    createdAt: '2024-11-20 10:00:00',
    rules: '会员享受8折优惠，限时一个月',
    statistics: {
      clicks: 520,
      participants: 128,
      conversions: 24.6,
      revenue: 12500
    }
  },
  {
    id: 2,
    name: '免费送会员活动',
    type: 'free_member',
    status: 'pending',
    startTime: '2025-01-01 00:00:00',
    endTime: '2025-01-15 23:59:59',
    participants: 0,
    createdBy: 'admin',
    createdAt: '2024-12-01 14:00:00',
    rules: '新用户注册送7天会员体验',
    statistics: {
      clicks: 0,
      participants: 0,
      conversions: 0,
      revenue: 0
    }
  },
  {
    id: 3,
    name: '积分翻倍活动',
    type: 'points_double',
    status: 'ended',
    startTime: '2024-11-01 00:00:00',
    endTime: '2024-11-30 23:59:59',
    participants: 89,
    createdBy: 'admin',
    createdAt: '2024-10-20 09:00:00',
    rules: '活动期间所有积分翻倍',
    statistics: {
      clicks: 320,
      participants: 89,
      conversions: 27.8,
      revenue: 8900
    }
  }
])

// 选中的活动
const selectedActivity = ref(null)

// 活动表单
const activityForm = reactive({})

// 统计数据
const totalActivities = computed(() => activities.value.length)
const activeActivities = computed(() => activities.value.filter(activity => activity.status === 'active').length)
const totalParticipants = computed(() => activities.value.reduce((sum, activity) => sum + activity.participants, 0))
const totalRevenue = computed(() => activities.value.reduce((sum, activity) => sum + (activity.statistics?.revenue || 0), 0))

// 筛选后的活动列表
const filteredActivities = computed(() => {
  let result = [...activities.value]
  
  // 按关键词搜索
  if (searchKeyword.value) {
    const searchLower = searchKeyword.value.toLowerCase()
    result = result.filter(activity => activity.name.toLowerCase().includes(searchLower))
  }
  
  // 按类型筛选
  if (activityType.value) {
    result = result.filter(activity => activity.type === activityType.value)
  }
  
  // 按状态筛选
  if (activityStatus.value) {
    result = result.filter(activity => activity.status === activityStatus.value)
  }
  
  // 分页
  const start = (page.current - 1) * page.size
  const end = start + page.size
  return result.slice(start, end)
})

// 活动类型名称
const getActivityTypeName = (type) => {
  const typeMap = {
    'member_discount': '会员折扣',
    'free_member': '免费会员',
    'points_double': '积分翻倍',
    '满减活动': '满减活动',
    '限时特惠': '限时特惠'
  }
  return typeMap[type] || type
}

// 活动类型标签
const getActivityTypeTag = (type) => {
  const typeMap = {
    'member_discount': 'primary',
    'free_member': 'success',
    'points_double': 'warning',
    '满减活动': 'info',
    '限时特惠': 'danger'
  }
  return typeMap[type] || 'default'
}

// 活动状态名称
const getActivityStatusName = (status) => {
  const statusMap = {
    'pending': '未开始',
    'active': '进行中',
    'ended': '已结束',
    'paused': '已暂停'
  }
  return statusMap[status] || status
}

// 活动状态标签
const getActivityStatusTag = (status) => {
  const statusMap = {
    'pending': 'info',
    'active': 'success',
    'ended': 'default',
    'paused': 'warning'
  }
  return statusMap[status] || 'default'
}

// 打开创建活动对话框
const openCreateDialog = () => {
  isEditing.value = false
  activityForm.name = ''
  activityForm.type = ''
  activityForm.status = 'pending'
  activityForm.startTime = new Date()
  activityForm.endTime = new Date()
  activityForm.rules = ''
  activityForm.description = ''
  activityDialogVisible.value = true
}

// 编辑活动
const editActivity = (activity) => {
  isEditing.value = true
  Object.assign(activityForm, activity)
  activityDialogVisible.value = true
}

// 保存活动
const saveActivity = () => {
  if (isEditing.value) {
    // 更新活动
    const index = activities.value.findIndex(activity => activity.id === activityForm.id)
    if (index !== -1) {
      activities.value[index] = { ...activityForm }
      ElMessage.success('活动更新成功')
    }
  } else {
    // 创建活动
    const newActivity = {
      ...activityForm,
      id: activities.value.length + 1,
      participants: 0,
      createdBy: 'admin',
      createdAt: new Date().toISOString().split('T')[0] + ' ' + new Date().toTimeString().split(' ')[0],
      statistics: {
        clicks: 0,
        participants: 0,
        conversions: 0,
        revenue: 0
      }
    }
    activities.value.unshift(newActivity)
    ElMessage.success('活动创建成功')
  }
  activityDialogVisible.value = false
}

// 删除活动
const deleteActivity = (id) => {
  const index = activities.value.findIndex(activity => activity.id === id)
  if (index !== -1) {
    activities.value.splice(index, 1)
    ElMessage.success('活动删除成功')
  }
}

// 启动活动
const startActivity = (activity) => {
  activity.status = 'active'
  ElMessage.success('活动已启动')
}

// 暂停活动
const pauseActivity = (activity) => {
  activity.status = 'paused'
  ElMessage.success('活动已暂停')
}

// 刷新活动列表
const refreshActivities = () => {
  ElMessage.success('活动列表已刷新')
}

// 导出活动
const exportActivities = () => {
  ElMessage.success('活动导出成功')
}

// 导入活动
const importActivities = () => {
  importDialogVisible.value = true
}

// 处理文件上传
const handleFileUpload = (file) => {
  importFile.value = file.raw
}

// 清除导入文件
const clearImportFile = () => {
  importFile.value = null
}

// 确认导入
const confirmImport = () => {
  if (!importFile.value) {
    ElMessage.warning('请选择活动文件')
    return
  }
  ElMessage.success('活动导入成功')
  importDialogVisible.value = false
  importFile.value = null
}

// 分页方法
const handleSizeChange = (size) => {
  page.size = size
  page.current = 1
}

const handleCurrentChange = (current) => {
  page.current = current
}
</script>

<style scoped>
.activity-container {
  padding: 0;
  background-color: var(--el-bg-color);
  min-height: 100vh;
}

.header-section {
  margin-bottom: 20px;
}

.page-title {
  font-size: 24px;
  font-weight: 700;
  color: var(--el-text-color-primary);
  margin: 0;
}

.page-subtitle {
  font-size: 14px;
  color: var(--el-text-color-secondary);
  margin: 4px 0 0 0;
}

.main-tabs {
  margin-bottom: 20px;
}

.top-actions {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.search-filter {
  display: flex;
  gap: 16px;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.search-input {
  width: 300px;
}

.filter-select {
  width: 180px;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}

/* 详情页样式 */
.detail-card {
  margin-bottom: 20px;
}

.card-header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  color: var(--el-text-color-primary);
  margin: 0;
}

.activity-detail {
  padding: 20px 0;
}

.detail-section {
  margin-bottom: 30px;
}

.section-title {
  font-size: 14px;
  font-weight: 600;
  color: var(--el-text-color-primary);
  margin: 0 0 16px 0;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--el-border-color);
}

.detail-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 16px;
}

.detail-item {
  display: flex;
  flex-direction: column;
}

.detail-label {
  font-size: 12px;
  color: var(--el-text-color-secondary);
  margin-bottom: 4px;
}

.detail-value {
  font-size: 14px;
  color: var(--el-text-color-primary);
  font-weight: 500;
}

.rule-content {
  background-color: var(--el-bg-color-page);
  padding: 16px;
  border-radius: var(--el-border-radius-base);
}

.rule-content pre {
  margin: 0;
  font-family: 'Courier New', Courier, monospace;
  font-size: 14px;
  color: var(--el-text-color-primary);
  white-space: pre-wrap;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 16px;
}

.stat-item {
  background-color: var(--el-bg-color-page);
  padding: 20px;
  border-radius: var(--el-border-radius-base);
  text-align: center;
  transition: all 0.3s ease;
}

.stat-item:hover {
  transform: translateY(-2px);
  box-shadow: var(--el-box-shadow-light);
}

.stat-value {
  font-size: 24px;
  font-weight: 700;
  color: var(--el-color-primary);
  margin-bottom: 8px;
}

.stat-label {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

.empty-detail {
  padding: 60px 0;
  text-align: center;
}

/* 统计页面样式 */
.stats-card {
  margin-bottom: 20px;
}

.stats-content {
  padding: 20px 0;
}

.chart-section {
  margin-bottom: 30px;
}

.chart-container {
  background-color: var(--el-bg-color-page);
  padding: 40px;
  border-radius: var(--el-border-radius-base);
  min-height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.chart-placeholder {
  text-align: center;
  color: var(--el-text-color-secondary);
}

.chart-icon {
  font-size: 48px;
  margin-bottom: 16px;
  opacity: 0.5;
}

.stats-overview {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
}

.overview-item {
  background-color: var(--el-bg-color-page);
  padding: 24px;
  border-radius: var(--el-border-radius-base);
  text-align: center;
  transition: all 0.3s ease;
}

.overview-item:hover {
  transform: translateY(-2px);
  box-shadow: var(--el-box-shadow-light);
}

.overview-value {
  font-size: 32px;
  font-weight: 700;
  color: var(--el-color-primary);
  margin-bottom: 8px;
}

.overview-label {
  font-size: 14px;
  color: var(--el-text-color-secondary);
}

/* 表单样式 */
.activity-upload {
  margin-top: 8px;
}

.file-info {
  margin-top: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background-color: var(--el-bg-color-page);
  border-radius: var(--el-border-radius-base);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .top-actions {
    flex-direction: column;
    align-items: stretch;
  }
  
  .search-filter {
    flex-direction: column;
    align-items: stretch;
  }
  
  .search-input,
  .filter-select {
    width: 100%;
  }
  
  .detail-grid {
    grid-template-columns: 1fr;
  }
  
  .stats-grid,
  .stats-overview {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .card-header-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}

@media (max-width: 480px) {
  .stats-grid,
  .stats-overview {
    grid-template-columns: 1fr;
  }
  
  .chart-container {
    padding: 20px;
  }
}
</style>